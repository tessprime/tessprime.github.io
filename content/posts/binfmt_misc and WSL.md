---
title: "`binfmt_misc` and WSL"
date: 2023-12-11T17:10:42-08:00
draft: false
---

So if you work with `wsl` on windows, you've likely learned that you can
execute, from the wsl terminal, windows executables. This is surprising!

If wsl is just a linux kernel running in a virtual machine, how does it
know how to deal with windows executables? Also, once it *does* know,
how does it signal to the host operating system that it should handle
the file?

Well, there is a mechanism called `binfmt_misc` that allows you to hook
into the OS `execve` functionality essentially. When the process loader
first looks at a file, it at the first several bytes of the file for:

1. Shebang `#!`
2. Elf Magic `\x7fELF`
3. Custom Magic.

Custom magic? Well, if you look in `/proc/sys/fs/binfmt_misc/` on your
linux machine, you'll see (at least) two files `status` and `register`.
In a WSL box you'll also see `WSLInterop` whose contents are

```
enabled
interpreter /init
flags: PF
offset 0
magic 4d5a
```

Where `45da` is hex for `MZ`, ye olde DOS marker of a PE (portable executable) format.

Hey that's interesting! On my linux machine, I found a number of interesting
things in it. Specifically, python .pyc files are supported, and more surprisingly,
qemu-arm as well. Looks like my linux x86-64 machine can run arm 'directly'!

Okay, so that answers the question "How does linux know to do something
special when it runs into a `MZ` file?". Now, what exactly is that something?

We can clearly see that linux is not just emulating the process via wine, becuase
if we spawn a few powershell instances in tmux, we see that these are 100% real
windows processes.

![Spawned Procs](/images/wsl_spawned_procs.png)

Interestingly, the interpreter for `MZ` is /init, which is the first binary
spawned when linux boots. And we can see this from the linux side by running
`ps`:

```
froeb      923  0.0  0.0   2328  1612 pts/29   S+   17:27   0:00 /init /mnt/c/Windows/System32/WindowsPowerShell/v1.0/powershell.exe powershell.exe
```

So now we ask a new question: `init` is just a process (run by a regular user no less!). So how does *it* talk to windows?

We can imagine a few possibilities:

1. The WSL linux kernel treats windows as an external device, and writes to special memory accordingly
via the regular device driver mechanisms.
2. Windows is a "networked machine" and WSL communictates to it 
3. Magic Assembly instructions that cause the hypervisor to intervene, presumably doing
ring checks. So the process would need to execute a special syscall.

`1` seems the most reasonable. `2` seems a tad scary and makes one worry about network reflection attacks.

So, to figure out what `init` at least is doing, we run `strace explorer.exe` and see what it's doing. Is it making
magic syscalls? We see that it doing the following:

```strace
 socket(AF_VSOCK, SOCK_STREAM|SOCK_CLOEXEC, 0) = 3
bind(3, {sa_family=AF_VSOCK, svm_cid=VMADDR_CID_ANY, svm_port=VMADDR_PORT_ANY, svm_flags=0}, 16) = 0
getsockname(3, {sa_family=AF_VSOCK, svm_cid=VMADDR_CID_ANY, svm_port=0x35d7636, svm_flags=0}, [16]) = 0
listen(3, 4)                            = 0
access("/run/WSL/30948_interop", F_OK)  = 0
socket(AF_UNIX, SOCK_STREAM|SOCK_CLOEXEC, 0) = 4
connect(4, {sa_family=AF_UNIX, sun_path="/run/WSL/30948_interop"}, 110) = 0
write(4, "\7\0\0\0=\1\0\0006v]\3\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\09\0\0\0Q\0\0\0\36\1\0\0\2\0\0\0\226\0\0\0\0\0\0\0\0C:\\Windows\\explorer.exe\0\\\\wsl.localhost\\Ubuntu-18.04\\home\\froeb\\projects\\cwgreene.github.com\0WSLENV=WT_SESSION:WT_PROFILE_ID:\0WT_SESSION=56f6065c-5d5c-4284-9a17-536ab64be312\0WT_PROFILE_ID={c6eaf9f4-32a7-5fdc-b5cf-066e8a4b1e40}\0\0\0explorer.exe"..., 317) = 317
poll([{fd=3, events=POLLIN}], 1, 10000) = 1 ([{fd=3, revents=POLLIN}])
accept4(3, {sa_family=AF_VSOCK, svm_cid=VMADDR_CID_HOST, svm_port=0x360a8b5f, svm_flags=0}, [16], SOCK_CLOEXEC) = 5
poll([{fd=3, events=POLLIN}], 1, 10000) = 1 ([{fd=3, revents=POLLIN}])
accept4(3, {sa_family=AF_VSOCK, svm_cid=VMADDR_CID_HOST, svm_port=0x360a8b60, svm_flags=0}, [16], SOCK_CLOEXEC) = 6
poll([{fd=3, events=POLLIN}], 1, 10000) = 1 ([{fd=3, revents=POLLIN}])
accept4(3, {sa_family=AF_VSOCK, svm_cid=VMADDR_CID_HOST, svm_port=0x360a8b61, svm_flags=0}, [16], SOCK_CLOEXEC) = 7
poll([{fd=3, events=POLLIN}], 1, 10000) = 1 ([{fd=3, revents=POLLIN}])
accept4(3, {sa_family=AF_VSOCK, svm_cid=VMADDR_CID_HOST, svm_port=0x360a8b62, svm_flags=0}, [16], SOCK_CLOEXEC) = 8
close(3)                                = 0
```

So it's creating a socket of type `AF_VSOCK`. Running `man vsock` reveals:

```
       The  VSOCK  address family facilitates communication between virtual machines and the host
       they are running on.  This address family is used by guest agents and hypervisor  services
       that  need  a  communications  channel  that  is  independent  of  virtual machine network
       configuration.
```

Well that seems to answer a good chunk of the question. From the user process (init) perspective,
windows is just a unix domain socket that it can read and hear back from. Also, this is a generic
linux solution to talking to a host operating system.

So current speculation of how this is working:

1. Linux kernel has a memory buffer which it uses to communicate with host operating system.
This is a completely generic solution and Linux figures this out the same way it would
with any other peripheral. It knows nothing about what's on the other end. Microsoft could
have baked in more self-awareness, but very likely chose to make it a user space problem (yay!).
2. `wsl.exe` reads and writes to this buffer from the windows side. Going to need to dig
into how microsoft's hyper-v interfaces work.
3. There exists a protocol between the two.
4. /init knows the protocol, and that's the part that microsoft provides.

Alright, that's good enough for now.
