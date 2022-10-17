---
layout: post
title: "Using ctags and cscope in Vim"
---
My philosophy has always been to maximize productivity with minimal effort. I think this is why I prefer Vim over Emacs :smile:.
Find a tool that works, stick with it, that's what I do. I haven't distro hopped in 10 years and I've been using vim as my text 
editor and tcsh as my login shell for more than 10 years. If it ain't broken, don't fix it!

My job has me working on dinosaur code bases where running `grep` to find stuff isn't exactly the most practical thing to do. 
Some of the tools I've integrated into my environment over the years are `ctags` and `cscope`, which, while requiring me
to learn a couple of new tricks, do make the *archaeologia cōdicis* phase go a lot faster.

So let's see how `ctags` and `cscope` work with Vim.

ctags
-----

You need to have some sort of `ctags` package installed. E.g.

```sh
pkg_add ectags
pkg_add universal-ctags
apt install exuberant-ctags
...
```

*Note: Once a package is installed, the actual binary name may vary. Replace `ctags` in the shell invocations below with the actual binary name of the thing you have installed.*

Vim used to ship with exuberant ctags, which has somewhat decent C++ support, plus a whole bunch of features related to local symbols, who calls what, etc.
If exuberant ctags isn't available, there's universal ctags, which is mostly CLI compatible (though it might complain that some `--c-kinds` are not available). The much older UNIX-y `ctags` is still floating around, that one doesn't do much. It can still be useful if it's all you got; it supports basically no flags.

For a lot of people, running `ctags .` is enough. But I like having a giant database for an entire source tree (sometimes for multiple source trees if
I'm working on a tool that uses some library). I invoke it like this:

```sh
# go to command
ctags --extra=+fq --fields=+amnStail --c-kinds=+cdefglpx --c++-kinds=+lpx -f tags -R .

# or, for a more exuberant version:
# --tag-relative=yes  <-- files in tags are relative to tags file and not to cwd
# --fields: enable everything I know of
# --extra: extra stuff for C++
# --c&c++-kinds: enable everything I know of
ctags -f tags --tag-relative=yes --fields=+a+f+i+k+l+m+n+s+S+z+t --extra=+f+q --c-kinds=+c+d+e+f+g+l+m+n+p+s+t+u+v+x --c++-kinds=+c+d+e+f+g+l+m+n+p+s+t+u+v+x -R .
```

I remember going through the exuberant ctags man page and picking everything that I might ever find interesting. I may have enabled all flags, I don't remember.

Vim
---

Vim has neat `ctags` integration, and I use almost none of it.

To add tags, set your `'tags'`: `set tags=tags`. Note, this is stored as a string and possibly looked up relative to `cwd`. This feature is meant to support having a 
bunch of ctags databases and only loading the subset for the current folder (or whatever). You can have absolute paths.

I prefer to have a giant database with an entire source tree, so I tend to `set tags=~/Projects/thing/tags,~/Projects/otherlib/tags`. 

The magical command you need to remember is `:ts` (tag-select). There's a shortcut `g]` which takes the string under the cursor and passes it to `:ts`. It then
brings up a menu with all the matches.

And if you weren't aware, `^O` and `^I` are Vim's *back* and *forward* buttons (like a web browser). I recall ctags support in Vim having a concept of a tag stack , but that's
one of the many features I don't use :smile:. I just stick with ol' reliable `^O`. Just go see `:help tags` and `:help tagsrch.txt`, there's a lot of functionality there...

cscope
------

sometimes ctags isn't enough; for example, there are many projects that provide helpful macros to help devs define or use stuff, and that confuses C parsers.

Again, you need `cscope` installed, e.g.

```sh
pkg_add cscope
apt install cscope
```

`cscope` comes in and provides its own search tools, and the features I mostly use are "who calls this", "where is this text string", "where is this regex", "where is this macro defined".

```sh
cscope -bqR
cscope -bqR -s../include -s../drv          # additional include dirs
cscope -bqR -u                             # force update the database
```

`cscope` isn't as neatly integrated into Vim, but it's good enough. `cscope` actually runs as a background process and Vim drives it.

Use `:cs add cscope.out` to enable.

Use `:cs` to show the help.

Then, you can use `:cs f g symbol`, `:cs f d who_calls_this`, `:cs f t text` and so on.

If you want to pass a complicated and long symbol name, and you don't want to type it (I usually don't), you can use Vim registers to hold it. 
Interactively, yank the thing in a register (e.g. `"qye` or visual select + `"qy` etc), then type `:cs f t ^Rq`. You can automate e.g. in visual mode
with `:vremap` or related functions.

I don't believe `cscope` works well with C++, but since I mostly use it as a complementary text or grep tool to `ctags`, it works well enough :smile:.

Conclusion
----------

Vim is a powerful IDE, but we knew that already. Especially when you pair `ctags` and `cscope` with the somewhat recent `:Termdebug` feature which makes Vim be a front-end to `gdb`; now, while debugging, if you stumble over a macro like `LIBPATH`, you can jump to it to see what it was compiled as :smile:.

The thing is, you can use these tools without Vim. `cscope` in particular has a menu drive text interface (one of the `b` or `q` flags disables this). You can use them
with any text editor / IDE.

The provide a massive amount of value on large or old or complicated projects/products. And they are considerably faster than running `grep` repeatedly, or trying
to guess where everything should be. There's just the initial setup time that can be annoying, but it usually finishes by the time you grab some coffee; and if the code base is large enough, `ctags` should have more than enough time to finish while the product builds :smile:. Keep these tools handy.

There is at least one more tool, GNU GLOBAL, but I haven't had fun with that one, so I never use it. Visual Studio Code also tries to do similar things to `ctags`
and `cscope`, and it mostly works, but who wants to run a text editor that eats up as much RAM as Chrome does? And I'm willing to wager [someone wrote a cool clang-\* thing that does the same thing as these two tools](https://duckduckgo.com/?q=clang+based+ctags), but [I'm too lazy to check](https://stackoverflow.com/a/48414520) (or to change my ways).
