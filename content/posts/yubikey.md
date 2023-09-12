---
title: "Yubikey"
date: 2023-07-29T15:17:59-07:00
draft: false
---
So picked up a yubikey, and am digging into how linux handles these
sorts of things exactly. It looks like the linux-pam module delegates
to pam-u2f, which appears to be written by Yubico themselves.
<!--more-->

On a separate note, you can jury rig the vscode's Intellisense system
to use `compile_commands.json`. To my pleasant surprise, the gnome
build system, using `meson`, for at least some of their projects will
automatically generate `compile_commands.json`.

GNOME reminds me of something that has always made me somewhat sad.
One of the cool things about the smalltalk environment was how you
could inspect anything and get the source code to it, and even start
mucking around with it. I think it'd be great, and a certain amount
of fun, to figure out how far you could recreate this experience on
top of GNOME.

If I managed to get it working, I'd totall demo it to Mark. :)
