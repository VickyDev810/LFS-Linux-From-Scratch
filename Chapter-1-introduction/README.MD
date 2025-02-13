# Linux From Scratch (LFS) - Day 1: Introduction

## Overview  
Day 1 focuses on setting up the environment for building the Linux From Scratch (LFS) system. This includes preparing the hardware, selecting the Linux distribution, partitioning the disk, and gathering the initial tools.

## Tasks Completed  
1. **Set up the environment for LFS.**  
2. **Configured hardware for building the system.**  
3. **Installed the Arch base system for the build.**  
4. **Created the partition schema.**

## Virtual Machine  
I will be using **VirtualBox** for my Linux host system.

## Hardware Requirements  
- **Recommended:**
  - 4 cores of CPU
  - 8 GB of memory  
- **My Setup:**
  - 6 processors
  - 8 GB of RAM
  - 50 GB of storage

## Linux Distribution Used  
- **Distribution:** archlinux-2025.02.01-x86_64 (install [here](https://archlinux.org/download/))  
- **Virtualization Tool:** VirtualBox  

I chose Arch for its simplicity as a base system for the LFS build process.

## Partition Schema  
A basic partition schema was created, including essential partitions such as `/`, `/home`, `/lfs`, and `swap`.  
You can configure `/lfs` after installing the base system; it's totally up to you, but for my ease, I configured it with the host.

### Recommended schema

| Number | Start (sector) | End (sector) | Size        | Code | Name               |
|--------|----------------|--------------|-------------|------|--------------------|
| 1      | 2048           | 4095         | 1024.0 KiB | EF02 | BIOS boot partition |
| 2      | 4096           | 413695       | 200.0 MiB  | 8300 | /boot              |
| 3      | 413696         | 4607999      | 2.0 GiB    | 8200 | swap               |
| 4      | 4608000        | 46551039     | 20.0 GiB   | 8300 | debian/arch        |

### My schema

| Device     | FSTYPE | FSVER | Size   | FSAVAIL | MOUNTPOINTS   |
|------------|--------|-------|--------|---------|---------------|
| /dev/sda1  | vfat   | FAT32 | 511M   | 511M    | /mnt/boot/efi |
| /dev/sda2  | ext4   | 1.0   | 18.5G  | 18.5G   | /mnt          |
| /dev/sda3  | swap   | 1     | 4G     | N/A     | [SWAP]        |
| /dev/sda4  | ext4   | 1.0   | 23.6G  | 23.6G   | /mnt/lfs      |

## Next Steps  
In the upcoming days, we will begin **Chapter 2: Preparing for the Build**, which involves setting up a user for building, installing necessary host packages, and preparing the build environment.

## Help Resources  
If you encounter any issues, refer to the [LFS Help Page](https://www.linuxfromscratch.org/lfs/view/stable/chapter01/askforhelp.html).
