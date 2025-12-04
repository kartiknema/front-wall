**Process** is a program in execution, an entity to which system resources like CPU, memory are allocated. A new process is created via the fork() system call, which is a near-exact copy of the parent process, although these processes may share the pages containing the program code (or text), however they have separate copies of data (stack and heap) so that changes made by one don’t affect the other.

- The text segment is read only, so no issues of a process modifying some instructions and adversely affecting other processes sharing those pages.

**Threads**: Linux uses light weight processes to support multithreaded applications, essentially a light weight process or LWP represents a thread. Two LWPs can share some resources like the address space, open files and so on (however they have separate eip, heap and stack), whenever one of them modifies a shared resource, the other LWP can see the change reflected. Each lightweight process can be scheduled independently by the kernel.
- **Lightweight Process** refers to a thread.
- A LWP can be created using the clone() system call, not the fork() system call.
- A LWP shares resources with its parent process, and with other LWPs, these resources include: Code (text) segment, open file descriptors, data segment.

The important point is Linux does not have a separate thread abstraction, instead it uses the same struct_task structure for both processes and threads (lwp).

Thread Group: A thread group is basically a set of lightweight processes that together implement the multithreaded application, and respond as a whole (i.e. as the process) to some system calls like getpid(), kill() and exit().
