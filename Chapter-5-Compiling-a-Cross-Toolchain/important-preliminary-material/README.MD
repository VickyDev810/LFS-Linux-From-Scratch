# 📌 Important Preliminary Material

This section follows **Chapter 4** in the table of contents and serves as an introduction to building a cross-toolchain.

---

## 📖 Introduction

### ⚠️ Important

No matter how eager you are to build the LFS system, at this stage, you **must** understand all the commands and avoid blindly pasting code.

Use the **tee** utility to track your typing and the output of commands. It sends terminal output to a file, making debugging easier.

---

## 🛠 Toolchain Technical Notes

This section provides the rationale and technical details behind the overall build method.

**You can omit this section at first and refer to it later for a deeper understanding.**

The primary goal of **Chapter 5** and **Chapter 6** is to create a temporary environment containing a set of reliable tools that are isolated from the host system. We achieve this by using the **chroot** command to create an isolated environment for building the target LFS system.

This section also explains the cross-compilation process.

### 🖥 Key Terminology

- **The Build** - The machine where we build programs, also known as the "host."
- **The Host** - The machine/system where the built programs will run.
- **The Target** - Used only for compilers. It is the machine the compiler produces code for.

💡 **Native Compilers** are cross-compilers that produce code for the machine they are running on.

🔗 Learn more on the official **[LFS Book](https://www.linuxfromscratch.org/lfs/view/stable/partintro/toolchaintechnotes.html)**

---

## 🔧 Implementation of Cross-Compilation for LFS

### 🔑 Key Takeaways

- All cross-compiled packages in the LFS book use an **autoconf-based** build system. This system accepts system types in the format **cpu-vendor-kernel-os**, known as the **system triplet**.
- Why "Triplet"? Originally, the kernel-os field was a single "system" field. While most systems treat kernel-os the same, for cases like Android, you cannot run an executable server without an emulation layer.
- You can check your system triplet using:
  ```bash
  ./config.guess
  ```
  or
  ```bash
  gcc -dumpmachine
  ```
- Be aware of the **dynamic linker** (also called the dynamic loader). It is provided by the **glibc** package and finds, loads, and prepares shared libraries needed by a program.
- Check your dynamic linker with:
  ```bash
  readelf -l <binary_name> | grep interpreter
  ```

### 📌 Cross-Compilation Process in LFS

To simulate a cross-compilation environment in LFS, we will:

1. Use **LFS_TGT** in the system triplet to represent "lfs."
2. Use the **--with-sysroot** option while building the cross-linker and cross-compiler to locate necessary host files.

#### 📋 Compilation Stages

| Stage | Build | Host | Target | Action |
|-------|-------|------|--------|------------------------------------------------|
| 1     | PC    | PC   | LFS    | Build cross-compiler (cc1) using cc-pc on PC. |
| 2     | PC    | LFS  | LFS    | Build compiler (cc-lfs) using cc1 on PC.      |
| 3     | LFS   | LFS  | LFS    | Rebuild and test cc-lfs using cc-lfs on LFS.  |

📍 **Note:**
- "On PC" means commands are run on a standard Linux distribution.
- "On LFS" means commands are executed in the **chrooted** environment.

For a deeper understanding, refer to the **[LFS Book](https://www.linuxfromscratch.org/lfs/view/stable/partintro/toolchaintechnotes.html)**, which explains the role of different compilers and why **cross-compiler cc-lfs** should not be used on the host system.

---

## 📂 Other Procedural Details

The **Cross-Compiler** will be installed in a separate `$LFS/tools` directory since it is not part of the final system.

This section outlines the sequence in which tools for the LFS system will be installed and how they will be stored.

---

## ⚙️ General Compilation Instructions

### ⚠️ Warning

🚨 **Do not mix content from different LFS versions or repositories, as it may cause subtle build failures.**

### 📌 Key Points to Remember

- Some packages require patches. LFS provides patches, but they might not need immediate installation since some packages are pre-patched before compilation.
- Warnings during the build process are **normal** and can be ignored.
- Ensure that your LFS environment is correctly set up:
  ```bash
  echo $LFS
  ```
  It should display **/mnt/lfs**.

### 🛠 System Environment Requirements

Ensure the following symbolic links exist:

- **bash** is the active shell.
- **sh** is a symbolic link to **bash**.
- **/usr/bin/awk** is a symbolic link to **gawk**.
- **/usr/bin/yacc** is a symbolic link to **bison**, or a script executing bison.

---

## 📑 Summary of the Build Process
### ❗Important

1️⃣ Place all sources and patches in a directory accessible from the chroot environment, e.g., `/mnt/lfs/sources/`. *(Already done if you followed the repo setup.)*

2️⃣ Change to the `/mnt/lfs/sources/` directory:
   ```bash
   cd /mnt/lfs/sources/
   ```

3️⃣ For each package:

- Extract the package using `tar`. *(Use only `tar`; avoid `cp -R` as it can modify timestamps and cause build failures.)*
```bash
tar -xvf packageName.tar.gz 
```
- Navigate into the extracted directory.
``` bash
cd packageName
```
- Follow build instructions.
- Return to the `/mnt/lfs/sources/` directory after the build.
```bash
cd $LFS/sources
```
- Delete the extracted source directory unless instructed otherwise.
```bash
rm -r packageName 
rm -rf packageName ## permanently delete the directory
```
---

✅ **By following this structured approach, you ensure a smooth and successful LFS build!**

