---
title: "Small Binaries in Zig"
date: 2023-09-14T15:02:09-07:00
---
So [https://ziglang.org/](Zig) can create some pretty small binaries!

So I was doing a ctf last week and found that I needed to upload
some binaries to a target machine with minimal tooling via a somewhat
cumbersome mechanism.

Thus I was very interested in minimizing the size of the uploaded binary.
<!--more-->

If you attempt to build a C program that is completely statically linked,
you get a rather large 870k program. If you're willing to use the
target machine's libc, then you can reduce the size of the binary to
33k.

However, `zig` is able to create standalone executables which are a mere
8.7k in size.  Just compile the following with `zig build-exe example1.zig -OReleaseSmall`.

```zig
const std = @import("std");
pub fn main() void {
  std.io.getStdOut().writeAll("Hello, World!\n") catch unreachable;
}
```

But can one do even better? Well, yes. :)

```zig
const std = @import("std");
const linux = std.os.linux;

pub export fn _start() noreturn {
    const s = "Hello World!\n";
    _ = linux.write(1, s, s.len);
    _ = linux.exit(0);
}
```

Let us marvel for a mometn at the clean, minimal generated code:

```asm
00000000002011c4 <.text>:
  2011c4:       6a 01                   push   $0x1
  2011c6:       58                      pop    %rax
  2011c7:       6a 0d                   push   $0xd
  2011c9:       5a                      pop    %rdx
  2011ca:       be 58 01 20 00          mov    $0x200158,%esi
  2011cf:       48 89 c7                mov    %rax,%rdi
  2011d2:       0f 05                   syscall
  2011d4:       6a 3c                   push   $0x3c
  2011d6:       58                      pop    %rax
  2011d7:       31 ff                   xor    %edi,%edi
  2011d9:       0f 05                   syscall
```


But there's a slight catch with the second example, if you attempt to use the stack, for example with

```zig
const std = @import("std");

const linux = std.os.linux;

pub export fn _start() noreturn {
    var tv = linux.timespec{
        .tv_sec = 1,
        .tv_nsec = 0
    };
    _ = linux.nanosleep(&tv, &tv);
    linux.exit(0);
}
```

this will segfault. 

`gdb` reveals the following issue:

```
─────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:64 ────
     0x2011bc                  lea    rdi, [rsp-0x18]
     0x2011c1                  movabs rax, 0x23
     0x2011cb                  mov    rsi, rdi
 →   0x2011ce                  vmovaps XMMWORD PTR [rdi], xmm0
     0x2011d2                  syscall
     0x2011d4                  movabs rax, 0x3c
     0x2011de                  xor    edi, edi
     0x2011e0                  syscall
     0x2011e2                  imul   r13, QWORD PTR [rsi+0x6b], 0x203a7265
─────────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "badzig", stopped 0x2011ce in ?? (), reason: SIGSEGV
───────────────────────────────────────────────────────────────────────────────────────────────────────────── trace ────
[#0] 0x2011ce → vmovaps XMMWORD PTR [rdi], xmm0
```

Ah, we've seen this before; stack alignment with the `vmovaps` instruction!

Turns out the normal zig `_start` command does a little extra work to align the stack. A quick fix is simply to
force a raw push at the begining to realign the stack. Once that is done, zig should keep things aligned.

```zig
const std = @import("std");

const linux = std.os.linux;

pub export fn _start() noreturn {
    asm volatile (
        \\ push %rbp
        :
        :
    );
    main();
    linux.exit(0);
}
fn main() void {
    var tv = linux.timespec{
        .tv_sec = 1,
        .tv_nsec = 0
    };
    _ = linux.nanosleep(&tv, &tv);
}
```

Note, some of this seems to be in flux. I had previously come across this post:

https://zserge.com/posts/zig-the-small-language/

Where they say to use `callConv(.Naked)` which unfortunately no longer works as
of July (people on the support discord were also suprised by this).

Anyhow, `zig` seems to be a nice language that may work as a more ergonomic
replacement for assembly. :)