---
layout: pose
title: "feh cheatsheet"
---

I always forget these things and the man page is pretty large.

First off, the configs are `~/.config/feh/buttons` for mouse, and
`~/.config/feh/keys` for keys. Some are the defaults, which I put there
as a sort-of cheatsheet, so let's start with my configs:

buttons:

    pan             1
    toggle_menu     3
    zoom            2
    zoom_in         4
    zoom_out        5
    rotate          C-3

keys:

    toggle_actions          a
    toggle_filenames        d
    toggle_fullscreen       f
    toggle_fixed_geometry   g
    toggle_info             i
    toggle_menu             m
    reload_image            r
    quit                    q
    toggle_aliasing         A
    toggle_auto_zoom        z Z
    flip                    underscore
    mirror                  bar
    zoom_in                 plus Up
    zoom_out                minus Down
    zoom_fit                0 C-z
    orient_3                less
    orient_1                greater
    prev_dir                bracketleft
    next_dir                bracketright

more (defaults):

    close                   x               # close just one window
    jump_random             z               # I've clobbered this...
    jump_first              Home
    jump_last               End
    jump_fwd                page up
    jump_back               page down
    remove                  Delete
    delete                  Ctrl+Delete
    zoom_fit                /               # I've changed this to C-z
    zoom_fill               !

Moving on, useful command line flags:

    -x, --borderless        borderless
    -F, --fullscreen        start fullscreen
    -g, --geometry          like any good X program
    --keep-zoom-vp          keep zoom when scrolling
    -p, --preload           preload
    -r, --recursive         recurse
    -., --scale-down        make images fit window
    -|, --start-at          start from the middle of a large collection
    -f, --filelist          load image paths from file
    -l, --list              ls thing
    -w, --multiwindow       open each image in its own window
    -S, --sort              by name, filename, dirname, mtime
    -t, --thumbnails

As always, see the man page: [feh(1)](https://manpages.org/feh)
