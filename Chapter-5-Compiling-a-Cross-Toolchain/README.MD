# **Chapter 5: Compiling a Cross-Toolchain**  

## ⛔ **Before We Start**  

Refer to the **[Important Preliminary Material](https://github.com/VickyDev810/LFS-Linux-From-Scratch/tree/CACT/Chapter-5-Compiling-a-Cross-Toolchain/important-preliminary-material#-important-preliminary-material).**  

---  

## 5.1 Introduction  

The programs compiled in this chapter will be stored in `$LFS/tools` to keep them separate from the files installed in later chapters.  

---  

## 5.2 Binutils-2.43.1 - Pass 1  

The **Binutils** package contains a linker, an assembler, and other tools for handling object files.  

- **Approximate Build Time:** 1 SBU  
- **Required Disk Space:** 677 MB  

Follow the steps mentioned in **[General Compilation Instructions](https://github.com/VickyDev810/LFS-Linux-From-Scratch/tree/CACT/Chapter-5-Compiling-a-Cross-Toolchain/important-preliminary-material#-summary-of-the-build-process)** before proceeding.  

### 📌 Preparing for Compilation  

Assuming you are inside the **binutils** package directory, create a build directory and enter it:  

```bash
mkdir -v build
cd build
```  

📍 **Note:** To measure the build time for this package, use the following command:  

```bash
time { ../configure ... && make && make install; }
```  

### 🛠️ Configuring Binutils  

Run the following command to prepare **Binutils** for compilation:  

```bash
../configure --prefix=$LFS/tools \
             --with-sysroot=$LFS \
             --target=$LFS_TGT   \
             --disable-nls       \
             --enable-gprofng=no \
             --disable-werror    \
             --enable-new-dtags  \
             --enable-default-hash-style=gnu
```  

### 🔍 Explanation of Configure Options  

- `--prefix=$LFS/tools`: Installs **Binutils** programs in the `$LFS/tools` directory.  
- `--with-sysroot=$LFS`: Directs the build system to look in `$LFS` for target system libraries.  
- `--target=$LFS_TGT`: Adjusts Binutils' build system for cross-linking.  
- `--disable-nls`: Disables internationalization (i18n), which is unnecessary for temporary tools.  
- `--enable-gprofng=no`: Disables **gprofng**, as it is not needed.  
- `--disable-werror`: Prevents the build from stopping due to warnings from the host compiler.  
- `--enable-new-dtags`: Uses **runpath** instead of **rpath** for embedding library search paths.  
- `--enable-default-hash-style=gnu`: Uses the GNU-style hash table exclusively to optimize performance and save disk space.  

---  

### 🚀 Compiling and Installing Binutils  

Compile the package:  

```bash
make
```  

Install the package:  

```bash
make install
```  

### ✅ Expected Output (Time Command)  

If you used the `time` command, you should see an output similar to:  

![Time Command Output](screenshots/BinUtils.png)  

---

## 5.3 GCC-14.2.0 - Pass 1  
**The GCC package contains the GNU compiler collection, which includes the C and C++ compilers.**

- **Approximate Build Time:** 3.2 SBU  
- **Required Disk Space:** 4.9 GB  

Follow the **General Compilation Instructions** before executing the following steps:  

### 📌 Preparing for Compilation  

1. Ensure you are inside the **GCC** package directory.  
2. Extract and prepare the necessary dependencies:  

```bash
tar -xf ../mpfr-4.2.1.tar.xz
mv -v mpfr-4.2.1 mpfr
tar -xf ../gmp-6.3.0.tar.xz
mv -v gmp-6.3.0 gmp
tar -xf ../mpc-1.3.1.tar.gz
mv -v mpc-1.3.1 mpc
```  

3. On **x86_64** hosts, set the default directory name for **64-bit** libraries to `lib`:  

```bash
case $(uname -m) in
  x86_64)
    sed -e '/m64=/s/lib64/lib/' \
        -i.orig gcc/config/i386/t-linux64
 ;;
esac
```  

4. Create a build directory and navigate to it:  

```bash
mkdir -v build
cd build
```  

### 🛠️ Configuring GCC  

Run the following command to prepare **GCC** for compilation:  

```bash
../configure                  \
    --target=$LFS_TGT         \
    --prefix=$LFS/tools       \
    --with-glibc-version=2.40 \
    --with-sysroot=$LFS       \
    --with-newlib             \
    --without-headers         \
    --enable-default-pie      \
    --enable-default-ssp      \
    --disable-nls             \
    --disable-shared          \
    --disable-multilib        \
    --disable-threads         \
    --disable-libatomic       \
    --disable-libgomp         \
    --disable-libquadmath     \
    --disable-libssp          \
    --disable-libvtv          \
    --disable-libstdcxx       \
    --enable-languages=c,c++
```  

### 🔍 Explanation of Configure Options  

- `--with-glibc-version=2.40`: Specifies the **Glibc** version for the target system.  
- `--with-newlib`: Ensures **inhibit_libc** is defined to prevent compiling code that requires libc.  
- `--without-headers`: Prevents **GCC** from looking for system headers (since they are not yet available).  
- `--enable-default-pie` & `--enable-default-ssp`: Enables **security hardening features**.  
- `--disable-shared`: Forces static linking of internal **GCC** libraries (since **Glibc** is not yet installed).  
- `--disable-multilib`: Disables multilib support on **x86_64** (harmless for **x86**).  
- `--disable-threads` and related options: Disables threading, **libatomic**, **libgomp**, **libquadmath**, **libssp**, **libvtv**, and **libstdcxx** (since they may fail to compile during cross-compilation).  
- `--enable-languages=c,c++`: Builds only the **C** and **C++** compilers (others are unnecessary at this stage).  

---

### 🚀 Compiling and Installing GCC  

Compile the package:  

```bash
make
```  

Install the package:  

```bash
make install
```  

### 📌 Creating a Full Version of `limits.h`  

After installing **GCC**, an internal system header (`limits.h`) is created, but it does not include the extended features of the system header (`$LFS/usr/include/limits.h`), which does not exist at this stage.  

To create a full version of `limits.h`, run the following command **from the GCC build directory**:  

```bash
cd ..
cat gcc/limitx.h gcc/glimits.h gcc/limity.h > \
  `dirname $($LFS_TGT-gcc -print-libgcc-file-name)`/include/limits.h
```  

---

## 5.4. Linux-6.10.5 API Headers

**The Linux API Headers (in `linux-6.10.5.tar.xz`) expose the kernel's API for use by Glibc.**

- **Approximate Build Time:** less than 0.1 SBU  
- **Required Disk Space:** 1.6 GB  

### 5.4.1. Installation of Linux API Headers

The Linux kernel exposes an Application Programming Interface (API) for the system's C library (Glibc in LFS) to utilize. This is done by sanitizing various C header files shipped in the Linux kernel source tarball.

Execute the following commands after following the General Compilation Instructions inside the **linux-6.10.5** directory:

1. Ensure no stale files are embedded in the package:
   ```bash
   make mrproper
   ```

2. Extract the user-visible kernel headers from the source. The recommended make target `headers_install` requires `rsync`, which may not be available. Instead, headers are placed in `./usr` and then copied to the appropriate location:
   ```bash
   make headers
   find usr/include -type f ! -name '*.h' -delete
   cp -rv usr/include $LFS/usr
   ```

### 5.4.2. Contents of Linux API Headers

**Installed headers:**
- `/usr/include/asm/*.h`, `/usr/include/asm-generic/*.h`, `/usr/include/drm/*.h`, `/usr/include/linux/*.h`, `/usr/include/misc/*.h`, `/usr/include/mtd/*.h`, `/usr/include/rdma/*.h`, `/usr/include/scsi/*.h`, `/usr/include/sound/*.h`, `/usr/include/video/*.h`, and `/usr/include/xen/*.h`

**Installed directories:**
- `/usr/include/asm`, `/usr/include/asm-generic`, `/usr/include/drm`, `/usr/include/linux`, `/usr/include/misc`, `/usr/include/mtd`, `/usr/include/rdma`, `/usr/include/scsi`, `/usr/include/sound`, `/usr/include/video`, and `/usr/include/xen`

**Short Description:**

| Header File Path                      | Description                          |
|---------------------------------------|--------------------------------------|
| `/usr/include/asm/*.h`                | The Linux API ASM Headers            |
| `/usr/include/asm-generic/*.h`        | The Linux API ASM Generic Headers    |
| `/usr/include/drm/*.h`                | The Linux API DRM Headers            |
| `/usr/include/linux/*.h`              | The Linux API Linux Headers          |
| `/usr/include/misc/*.h`               | The Linux API Misc Headers           |
| `/usr/include/mtd/*.h`                | The Linux API MTD Headers            |
| `/usr/include/rdma/*.h`               | The Linux API RDMA Headers           |
| `/usr/include/scsi/*.h`               | The Linux API SCSI Headers           |
| `/usr/include/sound/*.h`              | The Linux API Sound Headers          |
| `/usr/include/video/*.h`              | The Linux API Video Headers          |
| `/usr/include/xen/*.h`                | The Linux API Xen Headers            |

---

## 5.5. Glibc-2.40

**The Glibc package contains the main C library, providing the basic routines for memory allocation, file operations, string handling, and more.**

- **Approximate Build Time:** 1.3 SBU  
- **Required Disk Space:** 828 MB  

### 5.5.1. Installation of Glibc

1. First, create a symbolic link for LSB compliance. Additionally, for x86_64, create a compatibility symbolic link required for the dynamic library loader:
   ```bash
   case $(uname -m) in
       i?86)   ln -sfv ld-linux.so.2 $LFS/lib/ld-lsb.so.3 ;;
       x86_64) ln -sfv ../lib/ld-linux-x86-64.so.2 $LFS/lib64
               ln -sfv ../lib/ld-linux-x86-64.so.2 $LFS/lib64/ld-lsb-x86-64.so.3 ;;
   esac
   ```

2. ✏️ **Note:** The above command is correct. The `ln` command has several syntactic versions, so check `info coreutils ln` and [`ln(1)`](https://man.archlinux.org/man/ln.1) if you face issues.

3. Apply a patch to make Glibc programs store their runtime data in FHS-compliant locations:
   ```bash
   patch -Np1 -i ../glibc-2.40-fhs-1.patch
   ```

4. Create a dedicated build directory for Glibc:
   ```bash
   mkdir -v build
   cd build
   ```

5. Ensure that `ldconfig` and `sln` are installed in `/usr/sbin`:
   ```bash
   ls /usr/sbin/ldconfig
   ls /usr/sbin/sln
   ```

6. Run the following command to set the correct location:
   ```bash
   echo "rootsbindir=/usr/sbin" > configparms
   ```

7. Prepare Glibc for compilation:
   ```bash
   ../configure                             \
         --prefix=/usr                      \
         --host=$LFS_TGT                    \
         --build=$(../scripts/config.guess) \
         --enable-kernel=4.19               \
         --with-headers=$LFS/usr/include    \
         --disable-nscd                     \
         libc_cv_slibdir=/usr/lib
   ```

### 🔍 Explanation of Configure Options

- `--host=$LFS_TGT, --build=$(../scripts/config.guess)`: Configures Glibc for cross-compilation using the cross-linker and cross-compiler in `$LFS/tools`.
- `--enable-kernel=4.19`: Compiles Glibc with support for Linux kernel 4.19 and later.
- `--with-headers=$LFS/usr/include`: Compiles Glibc against the headers recently installed to `$LFS/usr/include`.
- `--disable-nscd`: Disables building the name service daemon (no longer used).
- `libc_cv_slibdir=/usr/lib`: Ensures that the library is installed in `/usr/lib` (not `/lib64` on 64-bit systems).

---

1. During this step, you might encounter the following warning:
   ![Warning](screenshots/warning.png)
   The missing or incompatible **msgfmt** program is generally harmless. It is part of the Gettext package, which the host distribution should provide.

2. ✏️ **Note:** If you encounter issues building with parallel make, rerun with the following option:
   ```bash
   make -ji
   ```

3. Compile the package:
   ```bash
   make
   ```

4. ⛔ **Warning:** If LFS is not set up correctly, and you are building as root, this command will install the newly built Glibc to your host system, potentially making it unusable. Ensure the environment is correctly set and avoid building as root.

5. Install Glibc:
   ```bash
   make DESTDIR=$LFS install
   ```

6. Fix a hard-coded path to the executable loader in the `ldd` script:
   ```bash
   sed '/RTLDLIST=/s@/usr@@g' -i $LFS/usr/bin/ldd
   ```

7. ❗ **Caution:** It's important to stop and check if basic functions like compiling and linking are working as expected:
   ```bash
   echo 'int main(){}' | $LFS_TGT-gcc -xc -
   readelf -l a.out | grep ld-linux
   ```

Expected output:
```bash
[Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
```

For 32-bit machines, it will look like:
```bash
[Requesting program interpreter: /lib/ld-linux.so.2]
```

If the output is incorrect or absent, something is wrong.

8. Clean up the test file:
   ```bash
   rm -v a.out
   ```

---

## 5.6. Libstdc++ from GCC-14.2.0

**Libstdc++ is the standard C++ library, needed to compile C++ code. Part of GCC is written in C++, but we deferred its installation during the GCC pass1 because it depends on Glibc.**

- **Approximate Build Time:** 0.2 SBU  
- **Required Disk Space:** 1.2 GB  

### 5.6.1. Installation of Target Libstdc++

✏️ **Note:** Libstdc++ is part of the GCC sources. You should first unpack the GCC tarball and enter the `gcc-14.2.0` directory.

1. Create a separate build directory for Libstdc++:
   ```bash
   mkdir -v build
   cd build
   ```

2. Prepare Libstdc++ for compilation:
   ```bash
   ../libstdc++-v3/configure           \
       --host=$LFS_TGT                 \
       --build=$(../config.guess)      \
       --prefix=/usr                   \
       --disable-multilib              \
       --disable-nls                   \
       --disable-libstdcxx-pch         \
       --with-gxx-include-dir=/tools/$LFS_TGT/include/c++/14.2.0
   ```

### 🔍 Explanation of Configure Options

- `--host=`: Specifies using the cross-compiler.
- `--disable-libstdcxx-pch`: Prevents the installation of precompiled include files.
- `--with-gxx-include-dir=`: Specifies the directory for include files.

3. Compile Libstdc++:
   ```bash
   make
   ```

4. Install Libstdc++:
   ```bash
   make DESTDIR=$LFS install
   ```

5. Remove libtool archive files to avoid issues with cross-compilation:
   ```bash
   rm -v $LFS/usr/lib/lib{stdc++{,exp,fs},supc++}.la
   ```

---

Next: **Chapter-6: Cross Compiling Temporary Tools**.

--- 
