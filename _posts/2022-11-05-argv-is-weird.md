---
layout: post
title: "argv[] is weird on OpenBSD, and notes on ps and rc"
---

So recently I was using `rcctl(1)` to change some parameters to some service on OpenBSD, and I noticed it started failing. I'm not going to name drop the service itself, since it runs fine, this is mostly because of a not-very-well documented quirk on OpenBSD. Anyway, the fail was weird, because the daemon would run when invoked from the terminal, so I started digging.

You can `rcctl -d thing start`, and in this case it was spewing out:

```sh
doing rc_check
doing rc_check
doing rc_check
doing rc_check
doing rc_check
doing rc_check
doing rc_check
doing rc_check
doing rc_check
doing rc_check
doing rc_check
doing rc_check
# it gets bored after a while
```

So I started grepping for `rc_check`.  I discovered `/etc/rc.d/rc.subr`, which is a sort of library for rc daemon scripts. `rc_check` looks like this:

```sh
rc_check() {
    pgrep -T "${daemon_rtable}" -q -xf "${pexp}"
}
```

`${daemon_rtable}` was blank in this case, so w/e. `${pexp}` is basically the daemon's name with the flags from `rc.conf.local`. `pgrep -xf` means _match this command line exactly_. (side note -- it keeps track of its PID, so I'm not sure why it's checking with pgrep). Apparently it was failing, but why?

The short answer is because the daemon was running `strtok(3)` on some of its `argv[]` strings, and `ps`/`pgrep` are apparently reading the argument vector from the same memory that the process was.

Allow me to demonstrate with a small C program:

```c
// build with cc argv.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

int main(int argc, char* argv[])
{
    if(argc > 1) {
        strtok(argv[1], ",");
    }
    sleep(5);
    return 0;
}
```

Let's play around:

```sh
cc argv.c || exit 1
./a.out 1 &
PID=$!
sleep 1
ps -ww -p $PID
pgrep -xf './a.out 1' && echo found it || echo lost it
kill -KILL $PID
```

It prints:

```
  PID TT  STAT        TIME COMMAND
88326 pj  S+       0:00.01 ./a.out 1
88326
found it
```

So far so expected.

Let's add some commas in the first argument.

```sh
./a.out 1,2,3 &
PID=$!
sleep 1
ps -ww -p $PID
pgrep -xf './a.out 1,2,3' && echo found it || echo lost it
pgrep -xf './a.out 1' && echo found it without commas || echo your scripts are bad and you should feel bad
kill -KILL $PID
```

This prints:

```
  PID TT  STAT        TIME COMMAND
71520 pj  S+       0:00.01 ./a.out 1
lost it
71520
found it with commas
```

Would you looks at that.

Let's try calling `strtok(3)` on a `strdup`:

```c
// build with cc argv.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

int main(int argc, char* argv[])
{
    if(argc > 1) {
        strtok(strdup(argv[1]), ",");
    }
    sleep(5);
    return 0;
}
```

And run it:

```sh
./a.out 1,2,3 &
PID=$!
sleep 1
ps -ww -p $PID
pgrep -xf './a.out 1,2,3' && echo found it || echo lost it
kill -KILL $PID
```

Which prints

```
  PID TT  STAT        TIME COMMAND
31111 pj  S+       0:00.01 ./a.out 1,2,3
31111
found it
```

Now, what are `ps` and `pgrep` actually doing? On to `/usr/src/bin/ps/print.c`! Here's the relevant snippet:

```c
144         if (!commandonly) {
145             char **argv = NULL;
146 
147             if (kd != NULL) {
148                 argv = kvm_getargv(kd, kp, termwidth);
149                 if ((p = argv) != NULL) {
150                     while (*p) {
151                         if (wantspace) {
152                             putchar(' ');
153                             left--;
154                         }
155                         left -= mbswprint(*p, left, 0);
156                         if (left == 0)
157                             return;
158                         p++;
159                         wantspace = 1;
160                     }
161                 }
162             }
// I'm not going to ballance out the parens
```

It's on line 148. Let's `man kvm_getargv`:

```
kvm_getargv() returns a null-terminated argument vector that corresponds
     to the command line arguments passed to process indicated by p.  Most
     likely, these arguments correspond to the values passed to execve(2) on
     process creation.  This information is, however, deliberately under
     control of the process itself.  Note that the original command name can
     be found, unaltered, in the p_comm field of the process structure
     returned by kvm_getprocs().
```

Well, it sure sounds like yes, if you modify `argv`, you will probably confuse programs calling `kvm_getargv`.

Let's update our test program one more time:

```c
// compile with cc argv.c -lkvm
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/types.h>
#include <sys/param.h>
#include <sys/sysctl.h>
#include <kvm.h>
#include <assert.h>

int main(int argc, char* argv[])
{
    if(argc <= 1) return 1;
    printf("I got %s\n", argv[1]);
    strtok(argv[1], ",");
    kvm_t *kd = kvm_openfiles(NULL, NULL, NULL, KVM_NO_FILES, NULL);
    int cnt = 0;
    struct kinfo_proc *ki = kvm_getprocs(kd, KERN_PROC_PID, (int)getpid(), sizeof(struct kinfo_proc), &cnt);
    char** qargv = kvm_getargv(kd, ki, 132);
    printf("kvm_getargv yielded %s\n", qargv[1]);
    return 0;
}
```

This prints:

```
argv> ./a.out 1,2,3
I got 1,2,3
kvm_getargv yielded 1
```

So the lesson is: don't clobber the contents of `argv` if you want your program to be a daemon run from an rc script on OpenBSD 7.2 on i386.

I have since learned that the standard says that a program can do whatever it wants with `argv[]`, but it is sufficiently vague about the **contents** of the argument vector to allow an implementation to end up in the situation from the post. So, the second lesson is, when drafting a standard, be prepared for people to make the strangest assumptions about undefined behaviour :smile:.
