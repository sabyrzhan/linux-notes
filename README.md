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

## `/etc/factab`
It is used to mount filesystems at boot time. The file contains list of filesystems 
with boot options.

## Special-purpose filesystems
Other than standard `ext2/ext3/ext4/fat` filesystems, there are filesystems that are mounted and 
not mapped to physical disk devices. Those filesystems can represent info about processes, can be 
mapped to RAM, etc. Some of them are:
1. `/proc` - the directory is mounted and stores every process information. It also provides info about 
   CPU, RAM, disk and etc.
2. `/sys` - sysfs. Provides disk storage information.
3. `tmpfs` - mounted on `/run` and other locations. It is used to create tmp storage in RAM and swap spaces.
4. `sqashfs` - RO filesystem used to mount compressed files. One example is Ubuntu snap package manager that mounts 
   installed packages.
5. `overlay` - used to merge multiple directories to be used as single entity. Most often containers use this FS
   to provide volumes to containers.

## Swap memory
Swap memory is used to temporarily store the inactive processes in non-ram storage. On Linux, swap memory can be
created either in partition or swap file.
1. to create swap in a partition:
   1. execute `mkswap <partition uuid>`
   2. specify partition in `/etc/fstab`
   3. execute `swapon` to enable swap
2. to create swap in regular file
   1. create file with the size you prefer, for example with `dd` command:
      1. `dd if=/dev/zero of=swap_file bs=1024k count=num_mb`
   2. execute `mkswap <file_name>`
   3. specify file in `/etc/fstab`
   4. execute `swapon` to enable swap

# Boot loading
Overall boot process steps look following:
1. BIOS/UEFI loads boot loader to memory and runs it
2. Boot loader finds kernel image from disk, loads to memory and starts it
3. Kernal initializes devices and loads device drivers
4. Kernel mounts root file system
5. Kernel starts `init` process with `ID=1`. `init` starts `user space`
6. `init` starts other user related initializations and processes
7. at the end `init` prompts `login` to user to enter the system

If your OS uses systemd for booting, you can view kernel boot logs using 
`journalctl -k`. By using `journalctl -b` it is possible to view old logs too.

To boot the kernel some parameters are passed in order to boot properly. For example,
```bash
BOOT_IMAGE=/boot/vmlinuz-5.11.0-27-generic root=/dev/mapper/vgubuntu-root ro quiet splash
(this line can also be viewed from /proc/cmdline)
```
Here we can see that bootloader using image in `ro`(read-only) mode. This is required to run `fsck`
to validate filesystem integrity and that no changes will be made. 

## Boot loaders
Boot loader's task is to search for kernel somewhere from disk, load it to memory and 
run with parameters. But as we know only kernel can initialize devices and load the drivers.
However, thanks to disk internal firmware BIOS/UEFI can access the disk using 
LBA (Logical Block Access) interface. Even though its performance is very low compared to
device driver, but it is enough to read kernel image and load to RAM.

The list of known boot loaders are GRUB, LILO (ELILO that supports UEFI), SYSLINUX, LOADLIN etc.
