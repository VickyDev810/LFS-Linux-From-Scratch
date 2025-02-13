
# Chapter 2 Preparing The Host System

## Day 1 Continued

## Chapter 2.2
### Install Required Packages

**You can use shared folders for virtual machines to copy data if you are running minimal versions of Linux.**

For Arch-based systems, use the **pacman** package manager to install the required packages:

```bash
sudo pacman -Syu --needed bash binutils bison coreutils diffutils findutils gawk gcc grep gzip linux-headers m4 make patch perl python sed tar texinfo xz
```

For Debian-based systems, install the necessary packages using:

```bash
sudo apt update && sudo apt install -y bash binutils bison coreutils diffutils findutils gawk gcc g++ grep gzip linux-headers-$(uname -r) m4 make patch perl python3 sed tar texinfo xz-utils
```

For more details on the requirements, refer to the official [Linux From Scratch Requirements Page](https://www.linuxfromscratch.org/lfs/view/stable/chapter02/hostreqs.html).

### Verifying Installed Packages

You can verify if the required packages are installed by running the `verify.sh` script. You can find the script [here](https://www.linuxfromscratch.org/lfs/view/stable/chapter02/hostreqs.html).

After running the script, the output should look similar to this:

![Verification Screenshot](screenshots/verify.png)

### Next Steps

Now that the host system is prepared, we will proceed to set up Linux From Scratch (LFS).

# Day 2

## Chapter 2.4: Creating an LFS Partition  

🔹 **Minimum Storage Requirements:**  
- **10GB** (Basic LFS Build)  
- **30GB** (If LFS will be the primary Linux system)  
- **I have used 23.6GB of storage**  

If you have already created an LFS partition, you can skip this step.  

### 🔹 Checking Available Disks  
Use the following command to check the available disks:  

```bash
lsblk
```

### 🔹 Creating a New Partition  
Use the `fdisk` tool to create a partition:  

```bash
fdisk /dev/sdX
```

Alternatively, you can use tools like **cfdisk** or **GParted** for partitioning.  

🔹 **Note:** If you previously mounted your LFS partition at `/lfs`, unmount it using:  

```bash
umount /lfs
```

---

## Chapter 2.5: Formatting the LFS Partition  

Specify a filesystem for your LFS partition. You can use **ext3** or **ext4** (recommended).  

Use the following command:  

```bash
mkfs -v -t ext4 /dev/sdX  # Replace "sdX" with your LFS partition name
```

---

## Chapter 2.6: Setting the LFS Environment Variable  

Set the **LFS** environment variable:  

```bash
export LFS=/mnt/lfs
```

Verify the environment variable:  

```bash
echo $LFS
```

---

## Chapter 2.7: Mounting the New Partition  

Since I previously unmounted `/lfs`, I will now remount it.  

### 🔹 Mounting a Single Partition  

```bash
mkdir -pv $LFS
mount -v -t ext4 /dev/sdX $LFS
```

### 🔹 Mounting Multiple Partitions  

```bash
mkdir -pv $LFS
mount -v -t ext4 /dev/sdX $LFS
mkdir -v $LFS/home
mount -v -t ext4 /dev/sdY $LFS/home
```

📌 **For reference, check the following screenshot of my disk structure:**  
![Disk Structure](screenshots/disk_str.png)  

