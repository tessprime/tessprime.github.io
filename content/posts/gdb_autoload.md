---
title: "Gdb autoloading of local .gdbinit files"
date: 2023-10-25T17:27:16-07:00
tags: ["gdb", "notes"]
---
So while figuring out how to attach the vscode to debug the openjdk, I learned the rather
surprising fact that apparently the jvm uses segfaults as a signalling mechanism in
some cases (clearly this should eventually require its own research and blog post).

Anyhow, to handle this, the apparent recommendation for using `gdb` with `java` is to
set the following in your `.gdbinit`:

```none
handle SIGSEGV noprint
handle SIGSEGV nostop
set print thread-events off
```

Now, understandably, I would rather this *not* be the case for most of my projects
(and I'm understandably weary of having it on in openjdk debugging... but that's for
the future!) so I did a quick investigation on how to setup per project `.gdbinit` files.

The way to do it is to turn on local gdbinit files with:

```none
set auto-load local-gdbinit on
```

And then whitelist the desired directories, with something like:

```none
set auto-load safe-path ~/src/openjdk
```

And place the questionable .gdbinit logic in that directory.

However, vscode for some reason doesn't seem to like this (future investigation needed).

To get this working with vscode, add the following to the launch configuration target:

```json
        "postRemoteConnectCommands": [
            {
                "text": "source ${workspaceFolder}/.gdbinit"
            }
```
