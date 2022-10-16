---
layout: post
title: "Fixing fbpanel's battery plugin on OpenBSD"
---

Recently, I set up [fbpanel](aanatoly.github.io/fbpanel/) as my Window Manager's panel thing. When I tried enabling the battery plugin, I got told it couldn't be loaded. After some snooping around (mainly in `ports/x11/fbpanel/patches`), I saw that it is not built or shipped by the port.

So... let's try to build it.

```sh
cd /usr/ports/x11/fbpanel
make prepare
make configure
```

Now let's tweak `/usr/ports/x11/fbpanel/patches/patch-plugins_Makefile` to enable it building the battery plugin. It rather amounts to adding `battery` to its `SUBDIRS`.

```sh
make build
```

TODO

I still wanted a battery indicator because I would very much want to know when my laptop is about to kick the bucket. fbpanel has the `genmon` plugin to arbitrarily run any command, let's do that.

```csh
#!/usr/bin/env tcsh
set APM_A=`apm -a`
set APM_L=`apm -l`
if( $APM_A == 1 ) then
    echo ' (+) '$APM_L'%'
else
    echo ' (-) '$APM_L'%'
endif
```

And let's get it running by tweaking fbpanel's config:

```
Plugin {
    type = genmon
    config {
        Command = ~/script.csh
        PollingTime = 10
        TextColor = black
    }
}
```

TODO image

This is nice, but it wouldn't help me fix fbpanel's plugin. So let's read the docs: `apm(8)` which points to `apm(4)` (the driver), which lists some `ioctl(2)`s to get the same information.

Cool! Let's write a [C program](https://github.com/alzwded/obsdBatteryInfo). Now that's something I can use to replace that Linux-y code with something more native!

TODO writing battery code

TODO missing meter plugin

TODO generating a patch for the patches
