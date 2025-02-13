# Chapter 4: Final Preparation

## 4.1 Introduction

In this chapter, we will complete the final preparations for the project. This includes:

- Setting up **directories** in `$LFS` for installing temporary tools.
- Adding an **unprivileged user** for building the system.
- Creating an appropriate **build environment** for the user.

---

## 4.2 Creating a Limited Directory Layout in the LFS Filesystem

Run the following commands as **root** to create the necessary directories:

### Create essential directories:
```bash
mkdir -pv $LFS/{etc,var} $LFS/usr/{bin,lib,sbin}  
```
> This command creates directories like `$LFS/etc`, `$LFS/var`, `$LFS/usr/bin`, `$LFS/usr/lib`, etc.

### Create symbolic links for essential directories:
```bash
for i in bin lib sbin; do
  ln -sv usr/$i $LFS/$i  
done
```
> The `ln` command creates links.  
> The `-s` option ensures **symbolic links** instead of hard links.

### Ensure compatibility for 64-bit systems:
```bash
case $(uname -m) in
  x86_64) mkdir -pv $LFS/lib64 ;;
esac
```
> This checks if the system is **64-bit** and, if true, creates the `$LFS/lib64` directory.

### Create a directory for the cross-compiler (used in Chapter 6):
```bash
mkdir -pv $LFS/tools
```

---

## 4.3 Adding the Linux User

### Create a new user group and user:
```bash
groupadd lfs
useradd -s /bin/bash -g lfs -m -k /dev/null lfs    
```
> Breakdown of the options used:
> - `-s /bin/bash` → Sets Bash as the default shell.
> - `-g lfs` → Adds the user to the `lfs` group.
> - `-m` → Creates a home directory for the user.
> - `-k /dev/null` → Prevents copying files from a skeleton directory.

### Set a password for the new user:
```bash
passwd lfs
```

### Grant the `lfs` user full permissions for `$LFS` directories:
```bash
chown -v lfs $LFS/{usr{,/*},lib,var,etc,bin,sbin,tools}
case $(uname -m) in
  x86_64) chown -v lfs $LFS/lib64 ;;
esac
```

### Switch to the new user:
```bash
su - lfs
```

## 4.4 Setting Up the Environment

### Logging in as the `lfs` User
To begin, log in as the `lfs` user using the following command:

```bash
su - lfs
```

### Creating a New `.bash_profile`
Create a new `~/.bash_profile` file:

```bash
cat > ~/.bash_profile << "EOF"
exec env -i HOME=$HOME TERM=$TERM PS1='\u:\w\$ ' /bin/bash
EOF
```

#### Explanation:
- The `exec env -i` command replaces the running shell, which reads its data from `/etc/profile` (containing host-specific settings and environment variables), with a new shell that starts with a completely empty environment.
- Only `HOME`, `TERM`, and `PS1` variables are retained to ensure a **clean environment** for the build process.

---

### Creating the `.bashrc` File
Now, create the `~/.bashrc` file:

```bash
cat > ~/.bashrc << "EOF"
set +h
umask 022
LFS=/mnt/lfs
LC_ALL=POSIX
LFS_TGT=$(uname -m)-lfs-linux-gnu
PATH=/usr/bin
if [ ! -L /bin ]; then PATH=/bin:$PATH; fi
PATH=$LFS/tools/bin:$PATH
CONFIG_SITE=$LFS/usr/share/config.site
export LFS LC_ALL LFS_TGT PATH CONFIG_SITE
EOF
```

#### Explanation:
- **`set +h`**: Disables Bash’s hash function to ensure that newly compiled tools in `$LFS/tools/bin` are used instead of older versions.
- **`umask 022`**: Ensures that newly created files are only writable by their owner but readable and executable by others.
- **`LFS=/mnt/lfs`**: Sets the mount point for the LFS system.
- **`LC_ALL=POSIX`**: Forces standard, predictable program behavior by disabling localization.
- **`LFS_TGT=$(uname -m)-lfs-linux-gnu`**: Sets a specific machine description to ensure compatibility when building the cross-compiler.
- **`PATH=/usr/bin`**: Some distributions merge `/bin` and `/usr/bin`. If this is the case, the standard `PATH` should be set to `/usr/bin`.
- **`if [ ! -L /bin ]; then PATH=/bin:$PATH; fi`**: If `/bin` is not a symbolic link, it must be added to `PATH`.
- **`PATH=$LFS/tools/bin:$PATH`**: Ensures the cross-compiler installed in Chapter 5 is used instead of the host system’s compiler.
- **`CONFIG_SITE=$LFS/usr/share/config.site`**: Prevents contamination from host system configuration files.
- **`export`**: Makes the defined variables available in sub-shells.

---

## Important: Adjusting System Configuration as `root`

As the **root** user, run the following command:

```bash
[ ! -e /etc/bash.bashrc ] || mv -v /etc/bash.bashrc /etc/bash.bashrc.NOUSE
```

#### Why?
- This renames `/etc/bash.bashrc` (if it exists) to `bashrc.NOUSE`.
- Prevents potential **interference** from the host system’s Bash configuration during critical package builds.

---

## Enabling Parallel Compilation
To speed up the build process, we will use **parallel make**, which tells the `make` program how many processors to use.

### Checking Available Processors
To find the number of logical processors:
```bash
lscpu
```

The number of build jobs is typically **equal to the number of available logical processors**. For example:
- An **Intel Core i9-13900K** has **8 P-cores (performance cores)** and **16 E-cores (efficiency cores)**.
- Each **P-core** can run **two threads**, resulting in **32 logical cores**.
- To maximize efficiency, we set `make` to use **32 jobs**:

```bash
make -j32
```

If you are using a Virtual Machine and have allocated **6 logical processors**, set `make` accordingly:

```bash
make -j6
```

Alternatively, you can **automate** this by exporting the `MAKEFLAGS` environment variable:

```bash
export MAKEFLAGS=-j$(nproc)  # Automatically detects available processors
```

To make this setting persistent, append it to `~/.bashrc`:

```bash
cat >> ~/.bashrc << "EOF"
export MAKEFLAGS=-j$(nproc)
EOF
```

---

## Applying the New Environment
To force the shell to use the newly configured environment, run:

```bash
source ~/.bash_profile
```

This ensures that all settings take effect immediately.

One way to check if all thing worked is to view the environment variables of bash using command
```bash
env
```

---

**SS of my env variables is provided below for refernce**
![LFS User Env](screenshots/lfs_env.png)
---

Now, your environment is fully set up for building **Linux From Scratch (LFS)**. 🚀


## 4.5 ABOUT SBUs

The **Standard Build Unit** (SBU) is a measure used to estimate how long it will take for a machine to install and compile a package.

Key Intakes

🔹 In chapter 5 we will be compiling our first package that is binutils.
The time it takes to compile using one core is what we will refer to as the Standard Build Unit or SBU.<br>
🔹 Controlling the clock speed in the newer motherboards can be done with powerprofilesct1 set performance but we can't use it now during the building of LFS. But it can be available on your host distro.

You can read more about it [here](https://www.linuxfromscratch.org/lfs/view/stable/chapter04/aboutsbus.html)

## 4.6 About the test suites

Many packages now provide a test suite that can be used as a sanity check to verify that everything compiled correctly.

You can read more about it [here](https://www.linuxfromscratch.org/lfs/view/stable/chapter04/abouttestsuites.html)

----
NEXT STEPS- **Building the LFS Cross Toolchain and Temporary Tools**😊

