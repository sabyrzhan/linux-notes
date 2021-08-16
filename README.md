# Linux notes of the Java developer

# Process management
### Context switching
Process tasks are executed within a time slice which is provided by kernel.
Whenever time slice is up for one process, the kernel provides another slice 
for another process. This is called `multitasking` - the execution that seems 
CPU is executing multiple processes at the same time. The process that CPU 
switches from one process to another is called `context switching`.

### Context switching steps
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

### System calls
Processes access hardware resources using `syscall` - it is used to access 
them through kernel. Because only the kernel has access to hardware 
resources.

Then are 2 most used types of system calls: `fork()` and `exec()`.
They are used to create and execute the processes accordingly.
1. `fork()` - creates a new process by copying the existing one
2. `exec(program)` - executes  the `program` by replacing the current 
  process, which shell.

# Memory management
The memory is used to read data by any process. The data is processed and 
written back by CPU. But the memory space itself is not shared between 
all the processes, even though it can be by some of them. The kernel itself 
must have its own memory space to process its own data. Each user process 
also has its own memory space and can be private or shared. Also it can be 
read-only. The total consumed memory space can be extended by using virtual 
memory. It is also called as `page table`. Page table lets to access data in 
memory by using virtual memory address. This lets to avoid the usage of hardware 
address.

# Disk management
Linux kernel does not directly write the data to persistent storage. Instead it uses 
cache to reuse the resources. Use `sync` command if you want to free up the cache and 
commit the data to storage before unmounting. Because sometimes this helps if you a 
facing issue with unmounting.

## Disk mounting and unmounting
1. To view the mounted device UUIDs execute `blkid` command.
2. To view mounted devices execute `mount` command
3. To mount a device use `mount` command. The format is `mount -t <device> <mount point>`.
   For example, `mount -t /dev/sda /mnt/myhdd`.
4. Use `sync` command to free cache and commit data to storage.
5. To unmount device use `umount <device>`

## Kernel and devices communications
Kernel communication with devices can be divided into 3 layers that goes from top to 
bottom
1. Block device interface (/dev/sda, /dev/sr0, etc) - provides interface to device
   specific driviers (SATA, CD/ROM, Flash etc). These drivers converts API messages to 
   SCSI messages to talk to SCSI devices and vice versa.
2. SCSI Sybsystem - abstraction layer that privides communication between 
   Kernel API and Hardware using SCSI protocol.
3. Hardware layer - drivers here send SCSI messages to hardware and vice versa. 