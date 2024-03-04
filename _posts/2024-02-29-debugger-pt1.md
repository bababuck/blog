# Making a Debugger: Pt 1

This will be a series on a recent (ongoing) project of mine on writing a debugger.

### What is a Debugger

A debugger is a software tool for developers to find and understand bugs their programs. It allows for inspection of the internal state of a program while it is running or after it has crashed. Some general features of a debugger:

- Breakpoints: Points in the code where execution will pause, allowing the programmer to inspect the state of the program at that moment.

- Stepping: Debuggers typically allow stepping through code one line or one instruction at a time, enabling programmers to trace the execution flow and identify problems.

- Variable inspection: Programmers can examine the values of variables at different points in the code execution to understand how data is being manipulated.

- Call stack inspection: Debuggers provide information about the sequence of function calls leading up to the current point in the code.

- Watchpoints: Halt program execution when the value of the watched variable changes or meets a specified condition.

- Memory inspection: Allows for examining the contents of memory at specific address.

### Why Writing a Debugger

The main motivation with this project is to provide an extremely lightweight debugger that can be used with minimal dependences. However, at the end of the day, this is mainly a learning experience for myself.

### Background

This project started out with the goal of creating a Linux debugger for RISCV systems, though the architecture only matters for a few small parts (which I will highlight when they arise). However, I was on an internation plane flight without internet when I started this project, and as I rudely found out, MacOS doesn't fully support the `ptrace()` system call, which as we'll see, is integral for implementing most of the debugging features we'll want. Fortunately, MacOS does still support debugging (GDB works on MacOS) so there must be a way to implement the functionality we need. Unfortunately, the documentation for that part of the kernel isn't great, so it was a bit of a slog to get much of this working. I will explain every syscall that I use here so you don't have to worry about it! For Linux users, I will hopefully get around to adding Linux support to this guide eventually.

### Getting Started

All you need for this tutorial is a Mac of some sort with administrative access and a C/C++ compiler, as well as a text editor. I personally use emacs, but any editor will work just fine. I will skip over some of the logistical details in this guide (such as compiling, linking, file setup etc.), so if you want to see exactly what I did feel free to follow along with my [git repository](https://github.com/bababuck/CRiscvUserDebugger). Don't let the name throw you off, this project did start of with RISCV Linux in mind, but there was a change in objective for reasons to be explained later.

Continue to Part 2 to begin writing your debugger!