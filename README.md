# Linux notes of the Java developer

# Process management
Process tasks are executed within a time slice which is provided by kernel.
Whenever time slice is up for one process, the kernel provides another slice 
for another process. This is called `multitasking` - the execution that seems 
CPU is executing multiple processes at the same time. The process that CPU 
switches from one process to another is called `context switching`.

Processes access hardware resources using `syscall` - it is used to access 
them through kernel. Because only the kernel has access to hardware 
resources.

Following steps are performed when context switching:
1. The time slice of one process is up by CPU timer
2. CPU turns the current mode from user to kernel and gives the handle to 
   kernel
3. Kernel saves the current state of CPU and memory
4. Kernel searches for the available waiting process from process list
5. Kernel provides memory space and time slice for the selected process
6. Kernel gives command to CPU to execute the given process in its 
   own user space
7. CPU turns mode from kernel mode to user mode 

# Memory management