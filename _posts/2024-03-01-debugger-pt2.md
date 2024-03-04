# Making a Debugger: Pt 2

### A Program to Test

If we're going to be debugging, we need a program to debug. So let's start off with a simple sample program. I wrote mine below, it it quite simple, simply printing out it's own name and the first argument provided to this. Really, this test program can be anything, but I chose to do this just so I can make sure I am properly launching the program. For those following along with the repo, see [631de8f](https://github.com/bababuck/CRiscvUserDebugger/commit/631de8fa2ba0774e5298279d1c5f60ce393ac788).

```c
#include <stdio.h>

int main(int argc, char **argv) {
  printf("Program=%s Argument[0]=%s\n", argv[0], argv[1]);
}
```

### Launching a Child Process

In order to debug a program, we have to be able to launch it from our debugger. To do this, we will use the system calls (syscalls) `fork` and `execv`. You can learn more about these by typing `man syscall_name` into `Terminal` (if on Mac, otherwise whatever shell you have on your system).

`fork` creates a new child process that is identical to the "caller" process with a few caveats, with the relevant one being that the child process has a unique process ID (as well as task ID as well learn more about when getting our hands dirty with `mach`. With `fork`, the program counter (which tells us where in our program we are currently at) will be identical, so we'll have as a result two processes executing the same code. Thus, we'll need to differentiate between the two processes so we can tell one to execute our debugger code and the other to start the test code. The simplest method for this is to look at the return value of the `fork` call. `0` will be returned to the child process, and the process ID of the child process will be returned to the parent process. Using this information, we can begin to write the first bit of code for our debugger. Note that `fork` returns a value less than 0 if it fails, so we should check for that too. See [673a961](https://github.com/bababuck/CRiscvUserDebugger/commit/673a9613c9a7a9ec359ac90fdf06547b9606a43e).

```c
int main(int argc, char** argv) {
  int child_pid = fork();
  if (child_pid > 0) {
    // Debugger
  } else if (child_pid == 0) {
    // Child Process
  } else {
    perror("fork() failed");
    exit(1);
  }
  return 0;
}
```

Once we've forked, we next need to launch our test program. `execv` comes from the `exec` family of functions, which replace the current process image with a new process image; this allows us to start execution of a different program. `execv` in particular specifies how we will provide the new program to the OS, which is `execv(const char *path, char *const argv[]);`. We will call this new `launch_child()` from the `// Child Process` seciton of `main()`, the modifcation is show below, see [23e3263](https://github.com/bababuck/CRiscvUserDebugger/commit/23e3263ada24347603489a74f74e0e77cf0518cd):

```c
  } else if (child_pid == 0) {
    // Child Process
    int child_argc = argc - 1;
    char** child_argv = argv + 1;
    launch_child(child_argc, child_argv);
  } else {
```

For `launch_child()`, we need to pass the arguments of the program we want to test. Namely, we need to just ignore the first argument that was passed to the debugger (the name of our debugger). So for calling our debugger on a test program with arguments A and B, we would write `./debugger test_program A B`, for the child process we just need `test_program A B`.

```c
void launch_child(int argc, char** argv) {
  char *user_program = argv[0];
  execv(user_program, argv);
}
```

Now if we run our debugger (the paths and name are specific to my directory structure, see my repo if you want to match), we can see that it successfully launches our test program before exiting.

```console
personal@Ryans-MacBook-Pro CRiscvUserDebugger % ./build/crudbg test/simple_print.exe A B
personal@Ryans-MacBook-Pro CRiscvUserDebugger % Program=test/simple_print.exe Argument[0]=A
```

Notice that the Shell prompt appears before the output from our test program; this isn't a bug, just our debugger returning before the child process prints its output. The Shell requests more input after the debugger returns, it doesn't not wait for all decendant processes to finish (see my [Github](https://github.com/bababuck/simple_shell) for a simple Shell implementation). In fact, it is more than likely the oblvious to the existance of other decendent processes.

In the next section, we'll get introduced to `ptrace`; see [Part 3](https://bababuck.github.io/blog/2024/03/02/debugger-pt2.html) to continue.