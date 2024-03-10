# Making a Debugger: Pt 3

### Debugger Class

One thing you'll note as you follow along is that I do mix together C and Cpp quite liberally. I enjoy the object oriented programming model that Cpp allows for, but in cases that that is not required, I tend to stick with C for it's simplicity. In this case, we need Cpp to create our debugger class, which will be extended to implement all the debugging functionality we desire. To keep our `main` function simple, we'll make a new function that will be in charge of creating a starting up our debugger. We'll pass along both the child process ID as well as the name of the program which we are testing. The former is necessary for getting information about the child process from the OS, the latter will be for making more useful printouts to the user. See commit [cd84433](https://github.com/bababuck/CRiscvUserDebugger/commit/cd844334335d17bc1a4a51f4731511f3ef8d5180).

```c
   } else if (user_pid > 0) {
     // Debugger
    launch_debugger(child_pid, argv[1]);
   } else {
```

```c
void launch_debugger(int child_pid, char *program_name) {
  debugger_t debugger(child_pid, program_name);
  debugger.run();
}
```

### Wait for Launch

When we create our debugger class, we want to wait for the child program to be ready to be debugger. I we can do this via the `waitpid` system call, using the given `child_pid` that was passed to it. `waitpid` forces the calling process to wait for the specified process to be stopped (for various reasons). I call `waitpid` within the constructor. Commit [3b1a081](https://github.com/bababuck/CRiscvUserDebugger/commit/3b1a08158354d76563f8cabf37849e98daa74684).

```c
debugger_t::debugger_t(int child_pid, char *program_name): child_pid(child_pid), program_name(program_name) {
  int status;
  if (waitpid(child_pid, &status, WUNTRACED) == -1)
    perror("waitpid() failed");
}
```

But what exactly are we waiting for? If the child program were to execute, `waitpid` would complete when the child process exited (regardless of success or not). That wouldn't be useful for debugging. We need to have the child program pause itself; in addition, we want our child program to allow itself to be debugged. Processes are not debuggable by default, and for good reason- otherwise, a malicious program could attempt to access or manipulate data in other processes.

Fortunately, Unix provides us both of these utilities in a single system call. `ptrace` is used for everything debug in Unix, depending on the `request` specified in the call. `ptrace` "provides tracing and debugging facilities. It allows one process (the tracing process) to control another (the traced process)". For this first use case, we the child will call `ptrace` with `request=PT_TRACE_ME`, specifying that the current process expects to be traced. If you're a Linux user, you may notice that there is no `PT_TRACE_ME`, instead a `PTRACE_TRACEME` exists. Functionally these are the same, but the provided headers in MacOS use slightly different naming conventions. See commit [e9faeac](https://github.com/bababuck/CRiscvUserDebugger/commit/e9faeac7dadee21406e74e45a35f145546f77c8b).

```c
void launch_child(int argc, char** argv) {
  char *user_program = argv[0];

  if (ptrace(PT_TRACE_ME, 0, 0, 0)) {
    perror("PT_TRACE_ME request failed");
    exit(DBG_SYSCALL_FAILURE);
  }

  execv(user_program, argv);
  exit(DBG_SYSCALL_FAILURE);
}
```

### Continuing Execution

Our child process is now waiting and ready to be debugged, so now lets learn how to resume it. Fortunately, our friend `ptrace` makes this quite simple for us to do! After `ptrace`, we'll again call `waitpid` so that the debugger doesn't do anything until the child process continues. And if you're thinking that this is quite simple, you're not wrong- `ptrace` makes debugging quite simple as long as you're familiar with some other basic Linux utilities. Unfortunately, if you're following along on MacOS, this is where the `ptrace` train stops. `ptrace` on Linux provides lots of other functionality that we would be able to use for the rest of our debugger, bug MacOS only provides a limited version of `ptrace`. We'll have to replicate this other functionality using other MacOS specified system calls. The MacOS comes from a micro-kernel known as Mach, which has it's own additional set of system calls which we will eventuall need to utilize.

```c
void debugger_t::resume_child_process(){
  if (ptrace(PT_CONTINUE, child_pid, (caddr_t) 1, 0)) {
    perror("ptrace() failed");
    exit(DBG_SYSCALL_FAILURE);
  }

  // Wait until process continues
  int status;
  if (waitpid(child_pid, &status, 0) == -1) {
    perror("waitpid() failed");
    exit(DBG_SYSCALL_FAILURE);
  }
}
```

Next, we will want to begin work on our interface for the debugger, an "interactive" debugger. There are plenty of fancy libraries that exist to a nicer interface, but given that we want this to be an lightweight debugger, I chose to implement an admittedly simpler interface myself. The basis for our debugging loop will look something like this. Each iteration, we parse a single command and execute the requested action if valid. For those following along, we've reached [2e10cbc](https://github.com/bababuck/CRiscvUserDebugger/commit/2e10cbcdc20435b281b895b89226eacc265a5152).

```c
void debugger_t::run() {
  std::string command;
  while (1) {
    printf("> "); // Prompt
    std::getline(std::cin, command);
    execute(command);
  }
}

void debugger_t::execute(const std::string &command_line) {
  auto args = parse_command(command_line);
  std::string command = args[0];
  int argc = args.size();

  if (command == "continue" || command == "c") {
    if (argc == 1) {
      resume_child_process();
    } else {
      std::cerr << "No arguments expected with 'continue'" << std::endl;
    }
  } else {
    std::cerr << "Unknown command" << std::endl;
  }
}
```

`parse_command()` looks a little something like this, essentially just splitting up the input:
```c
std::vector<std::string> debugger_t::parse_command(const std::string &line){
  std::vector<std::string> parsed = {};
  std::stringstream ss(line);
  std::string arg;
  while (std::getline(ss, arg,' ')) {
    parsed.push_back(arg);
  }
  return parsed;
}
```

Now armed with this simple interface, we can successfully pause the target program for debugging and resume it! Note that in order to use the `ptrace` utility we need administrative privilege (there is another way I've been told), hence the `sudo`.

```console
personal@Ryans-MacBook-Pro CRiscvUserDebugger % sudo build/crudbg test/simple_print.exe 
> continue
Program=test/simple_print.exe Argument[0]=(null)
```

Next time, we will begin our journey into accessing process state!
