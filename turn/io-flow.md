# How I/O works and related concepts

The following flow illustrates the sequence of steps which take place when a user issues a “read” system call.

## Userspace -> Library Call
The application calls: read(fd, buf, sizeof(buf)), which is a glibc wrapper around the actual “read” system call. Internally this wrapper uses the syscall instruction to invoke the system call, specifying the system call number for read (__NR_read).

## Transition the processor to kernel mode
As part of the syscall instruction the CPU switches from user to kernel mode, as part of this transition the process registers are saved, so that they can be restored later and the kernel updates the stack pointer so that it points to the kernel mode stack for the process.

- This highlights that userspace and kernel space have separate stacks, this should be expected since each of them run in their own memory spaces, and kernel code cannot share stack with user mode because of security reasons.
- When a process, after performing a system call, starts executing kernel code then it switches to the kernel stack (from the user stack).
- The kernel stack is much smaller than the user stack.
- This also indicates that kernel stacks are maintained at a per-thread level, this is equivalent to how stacks work in the userspace.

## Kernel system call entry point
Based on the system call number, the kernel checks the syscall table to find the appropriate handler for this system call, it finds it and it is called: “sys_read”. Control is transferred to this handler.

## sys_read and VFS
- This is the actual implementation for the system call, since it must be compatible with different filesystems, hence it uses a VFS layer and dispatches to the appropriate “read” implementation.
- sys_read internally calls vfs_read, which is a VFS method
- VFS provides an interface which different filesystems must implement.
- Finding the correct handler: The file OFD (open file description or struct file) includes a struct of function pointers, called file_operations which maps the functions like “read”, “write” to their implementations. Each filesystem populates this struct to point to their implementations.
- In this way, sys_read is not hardcoded to any filesystem specific implementation.

## Actual IO implementation
Behind the VFS layer sit actual physical disk filesystems (or virtual filesystems) that perform the actual IO operations. The filesystem, for example ext4 translates the “read” request into a block IO request.
- If data is already present in page cache, the kernel satisfies the request immediately, i.e. no disk access.
- If required data is not present in page cache, then it must first be brought there, to do that the kernel issues a block IO request to the block layer.

## Block Layer
The block layer sits b/w VFS and device drivers, it receives block IO requests from the VFS. The block layer queues the request to the appropriate device driver. In addition, it performs other functionalities too, like determining the order in which IO requests should be issued.

## Process Sleep
Since the data is not readily available, hence disk I/O operation is needed, since disk I/O is expensive hence the process can temporarily be put to sleep and other processes can be scheduled.
- The kernel changes the process state from TASK_RUNNING to TASK_INTERUPPTIBLE.
- Context switches to a different process.

## Disk IO and Interrupts
The process which issued the “read” system call has been put to sleep, it will be woken up only when the data is ready. However, how does the kernel know that the disk IO has completed?
- When the disk finishes reading it triggers a hardware interrupt.
- The interrupt handler in the disk device driver runs, and marks the I/O completed.
- The required data is populated in the kernel’s page cache.

## Waking up the process
Once the driver marks the IO request as completed, the block layer wakes up the sleeping process, in other words the process states changes back to TASK_RUNNING.

## Copy data to userspace
Once data is available in the page cache, it is copied back to userspace using safe functions like: copy_to_user().

## Completion
Once the system call returns, the processor switches back to user mode, the read() library wrapper populates errno if needed and then returns to the caller.
