
**Course:** Sistemas Operativos II  
**Student:** Joan Daniel Rivas Andrade  

## Overview

This project implements a Linux kernel module named `sysinfo_chardev` that creates the character device `/dev/sysinfo`.

The module extends the functionality developed in Workshop 3. In the original version, reading from the device with:

```bash
cat /dev/sysinfo
````

returned general system information such as:

* Kernel release
* Jiffies
* Number of CPUs

In this updated version, the device also supports commands written from user space, allowing the user to choose which information is returned.

## Supported Commands

The device accepts the following commands:

* `all` → show all information
* `kernel` → show only the kernel release
* `cpu` → show only the number of CPUs
* `time` → show only jiffies

## How the Module Works

The module is implemented as a **character device driver** using:

* `register_chrdev()`
* a `file_operations` structure
* `open()`
* `read()`
* `write()`

### Behavior

* The `write()` function receives a command from user space.
* The command is validated and stored as the current operating mode.
* The `read()` function checks the selected mode and returns the corresponding system information.

## Why a Character Device?

A character device is appropriate for `/dev/sysinfo` because it provides information as a stream of bytes and does not represent a storage device.

This module is only used to:

* receive simple commands from user space
* return kernel-generated information through read operations

A block device would not be appropriate because block devices are designed for storage hardware such as disks, where data is managed in fixed-size blocks.

## Project Files

```text
sysinfo_project/
├── sysinfo_chardev.c
├── Makefile
├── README.md
├── screenshots/
└── report.pdf
```

## Requirements

Before building the module, make sure your system has:

* Linux kernel headers for the current running kernel
* `gcc`
* `make`

You can verify your environment with:

```bash
uname -r
gcc --version
make --version
```

## Build Instructions

Compile the module with:

```bash
make
```

This generates:

```bash
sysinfo_chardev.ko
```

## Load the Module

Insert the module into the kernel:

```bash
sudo insmod sysinfo_chardev.ko
```

Check that it was loaded correctly:

```bash
lsmod | grep sysinfo_chardev
dmesg | tail -20
cat /proc/devices | grep sysinfo
```

## Create the Device Node

After loading the module, obtain the assigned major number with:

```bash
cat /proc/devices | grep sysinfo
```

Then create the device manually:

```bash
sudo mknod /dev/sysinfo c <MAJOR> 0
sudo chmod 666 /dev/sysinfo
ls -l /dev/sysinfo
```

Replace `<MAJOR>` with the value assigned by the kernel.

### Example

```bash
sudo mknod /dev/sysinfo c 234 0
sudo chmod 666 /dev/sysinfo
ls -l /dev/sysinfo
```

## Testing

### Default output

```bash
cat /dev/sysinfo
```

### Show all information

```bash
echo all | sudo tee /dev/sysinfo > /dev/null
cat /dev/sysinfo
```

### Show only kernel release

```bash
echo kernel | sudo tee /dev/sysinfo > /dev/null
cat /dev/sysinfo
```

### Show only CPU count

```bash
echo cpu | sudo tee /dev/sysinfo > /dev/null
cat /dev/sysinfo
```

### Show only jiffies

```bash
echo time | sudo tee /dev/sysinfo > /dev/null
cat /dev/sysinfo
```

### Test invalid command

```bash
echo hola | sudo tee /dev/sysinfo > /dev/null
dmesg | tail -20
```

## Unload the Module

To remove the module from the kernel:

```bash
sudo rmmod sysinfo_chardev
```

## Important Notes

* The module uses `register_chrdev(0, ...)`, so the kernel assigns a **dynamic major number**.
* If the module is unloaded and loaded again, the major number may change.
* In that case, `/dev/sysinfo` must be recreated using the new major number.
* Invalid commands are rejected with `-EINVAL`.

## Example Workflow

```bash
make
sudo insmod sysinfo_chardev.ko
cat /proc/devices | grep sysinfo
sudo mknod /dev/sysinfo c <MAJOR> 0
sudo chmod 666 /dev/sysinfo

cat /dev/sysinfo
echo all | sudo tee /dev/sysinfo > /dev/null
cat /dev/sysinfo

echo kernel | sudo tee /dev/sysinfo > /dev/null
cat /dev/sysinfo

echo cpu | sudo tee /dev/sysinfo > /dev/null
cat /dev/sysinfo

echo time | sudo tee /dev/sysinfo > /dev/null
cat /dev/sysinfo

echo hola | sudo tee /dev/sysinfo > /dev/null
dmesg | tail -20

sudo rmmod sysinfo_chardev
```


