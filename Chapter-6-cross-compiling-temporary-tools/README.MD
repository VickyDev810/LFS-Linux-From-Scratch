# Chapter 6: Cross-Compiling Temporary Tools

## 6.1 Introduction

This chapter covers the cross-compilation of essential utilities using the newly built cross-toolchain. These utilities are installed in their final location but cannot be used yet. The system still relies on the host’s tools for basic tasks; however, the installed libraries will be used during linking.

These utilities will become usable in the next chapter after entering the **“chroot”** environment. Before that, all the packages in this chapter must be built, meaning we still depend on the host system.

⚠ **Warning:** Improperly setting up **LFS** or building as the root user may render your system unusable. All steps in this chapter must be executed as the **lfs** user, using the environment settings from [Section 4.4: Setting Up the Environment](#).

---

## 6.2 M4-1.4.19

**The M4 package contains a macro processor.**

- **Approximate build time:** 0.1 SBU  
- **Required disk space:** 31 MB  

### 🔧 Preparing M4 for Compilation
```bash
./configure --prefix=/usr   \
            --host=$LFS_TGT \
            --build=$(build-aux/config.guess)
```

### 🏗️ Compiling the Package
```bash
make
```

### 📦 Installing the Package
```bash
make DESTDIR=$LFS install
```

---

## 6.3 Ncurses-6.5

**The Ncurses package provides libraries for terminal-independent character screen handling.**

- **Approximate build time:** 0.4 SBU  
- **Required disk space:** 53 MB  

### 🔍 Pre-Configuration Step
Ensure that **gawk** is used instead of **mawk**:
```bash
sed -i s/mawk// configure
```
This replaces occurrences of `mawk` with `gawk` in the configuration file.

### 🏗️ Building the `tic` Program on the Host
```bash
mkdir build
pushd build
    ../configure
    make -C include
    make -C progs tic
popd
```

### 🔧 Preparing Ncurses for Compilation
```bash
./configure --prefix=/usr                 \
            --host=$LFS_TGT               \
            --build=$(./config.guess)     \
            --mandir=/usr/share/man       \
            --with-manpage-format=normal  \
            --with-shared                 \
            --without-normal              \
            --with-cxx-shared             \
            --without-debug               \
            --without-ada                 \
            --disable-stripping
```

### 📌 Explanation of Configuration Options
- **`--with-manpage-format=normal`**: Prevents Ncurses from installing compressed manual pages.
- **`--with-shared`**: Builds and installs shared C libraries.
- **`--without-normal`**: Prevents the installation of static C libraries.
- **`--with-cxx-shared`**: Builds and installs shared C++ libraries while avoiding static C++ libraries.
- **`--without-debug`**: Skips building debug libraries.
- **`--without-ada`**: Ensures Ada support is not built.
- **`--disable-stripping`**: Prevents host tools from stripping binaries, avoiding cross-compilation failures.

### 🏗️ Compiling the Package
```bash
make
```

### 📦 Installing the Package
```bash
make DESTDIR=$LFS TIC_PATH=$(pwd)/build/progs/tic install
ln -sv libncursesw.so $LFS/usr/lib/libncurses.so
sed -e 's/^#if.*XOPEN.*$/#if 1/' -i $LFS/usr/include/curses.h
```

### 🔍 Explanation of Installation Commands
- **`TIC_PATH=$(pwd)/build/progs/tic`**: Specifies the newly built `tic` program for terminal database creation.
- **`ln -sv libncursesw.so $LFS/usr/lib/libncurses.so`**: Creates a symbolic link, making `libncursesw.so` a replacement for `libncurses.so`.
- **`sed -e 's/^#if.*XOPEN.*$/#if 1/' ...`**: Modifies `curses.h` to always use the wide-character data structures, ensuring compatibility with `libncursesw.so`.

---

## 6.4. Bash-5.2.32

**The bash package contains the Bourne-Again Shell.**

**Approximate Build Time:** 0.2 SBU  
**Required Disk Space:** 67 MB  

### 🔧 Preparing Bash for compilation:
```bash
./configure --prefix=/usr                      \
            --build=$(sh support/config.guess) \
            --host=$LFS_TGT                    \
            --without-bash-malloc              \
            bash_cv_strtold_broken=no
```

### 🔍 Explanation of Installation Commands
- **`--without-bash-malloc`**: This option turns off the use of Bash's memory allocation (malloc) function which is known to cause segmentation faults.

### 🏗️ Compiling the package:
```bash
make
```

### 📦 Installing the package:
```bash
make DESTDIR=$LFS install
```

Make a link for the programs that use sh for a shell.
```bash
ln -sv bash $LFS/bin/sh
```

---

## 6.5. Coreutils-9.5

**The coreutils package contains the basic utility programs needed by every operating system.**  
**Approximate build time**: 0.3 SBU  
**Required disk space**: 175 MB  

### 🔧 Preparing Coreutils for compilation:
```bash
./configure --prefix=/usr                     \
            --host=$LFS_TGT                   \
            --build=$(build-aux/config.guess) \
            --enable-install-program=hostname \
            --enable-no-install-program=kill,uptime
```

### 🔍 Explanation of Installation Commands
- `--enable-install-program=hostname:` This enables the **hostname** binary to be built and installed - it is disabled by default but is required by the Perl test suite.

### 🏗️ Compiling the package:
```bash
make
```

### 📦 Installing the package:
```bash
make DESTDIR=$LFS install
```

Move programs to their final expected locations:
```bash
mv -v $LFS/usr/bin/chroot              $LFS/usr/sbin
mkdir -pv $LFS/usr/share/man/man8
mv -v $LFS/usr/share/man/man1/chroot.1 $LFS/usr/share/man/man8/chroot.8
sed -i 's/"1"/"8"/'                    $LFS/usr/share/man/man8/chroot.8
```

---

## 6.6. Diffutils-3.10

**The diffutils package contains programs that show the differences between files or directories.**  
**Approximate build time**: 0.1 SBU  
**Required disk space**: 29 MB  

### 🔧 Preparing Diffutils for compilation:
```bash
./configure --prefix=/usr   \
            --host=$LFS_TGT \
            --build=$(./build-aux/config.guess)
```

### 🏗️ Compiling the package:
```bash
make
```

### 📦 Installing the package:
```bash
make DESTDIR=$LFS install
```

---

## 6.7. File-5.45

**The File package contains a utility for determining the type of a given file or files.**  
**Approximate build time**: 0.1 SBU  
**Required disk space**: 37 MB  

### 🔧 Preparing File for compilation:
```bash
./configure --prefix=/usr --host=$LFS_TGT --build=$(./config.guess)
```

### 🏗️ Compiling the package:
```bash
make FILE_COMPILE=$(pwd)/build/src/file
```

### 📦 Installing the package:
```bash
make DESTDIR=$LFS install
```

Remove the libtool archive file:
```bash
rm -v $LFS/usr/lib/libmagic.la
```

## 6.8. Findutils-4.10.0

**The Findutils package contains programs to find files. Programs are provided to search through all the files in a directory tree and to create, maintain, and search a database (often faster than the recursive find, but unreliable unless the database has been updated recently). Findutils also supplies the xargs program, which can be used to run a specified command on each file selected by a search.**

- **Approximate build time:** 0.2 SBU  
- **Required disk space:** 48 MB  

### 🔧 Preparing Findutils for Compilation
```bash
 ./configure --prefix=/usr                   \
            --localstatedir=/var/lib/locate \
            --host=$LFS_TGT                 \
            --build=$(build-aux/config.guess)
```

### 🏗️ Compiling the Package
```bash
make
```

### 📦 Installing the Package
```bash
make DESTDIR=$LFS install
```

---

## 6.9. Gawk-5.3.0

**The gawk package contains programs for manipulating text files.**

- **Approximate build time:** 0.1 SBU  
- **Required disk space:** 47 MB  

### 🔧 Removing Unneeded Files
```bash
sed -i 's/extras//' Makefile.in
```

### 🔧 Preparing Gawk for Compilation
```bash
./configure --prefix=/usr   \
            --host=$LFS_TGT \
            --build=$(build-aux/config.guess)
```

### 🏗️ Compiling the Package
```bash
make
```

### 📦 Installing the Package
```bash
make DESTDIR=$LFS install
```

---

## 6.10. Grep-3.11

**The Grep package contains programs for searching through the content of files.**

- **Approximate build time:** 0.1 SBU  
- **Required disk space:** 27 MB  

### 🔧 Preparing Grep for Compilation
```bash
./configure --prefix=/usr   \
            --host=$LFS_TGT \
            --build=$(./build-aux/config.guess)
```

### 🏗️ Compiling the Package
```bash
make
```

### 📦 Installing the Package
```bash
make DESTDIR=$LFS install
```

---

## 6.11. Gzip-1.13

**The Gzip package contains programs for compressing and decompressing files.**

**Approximate build time:** 0.1 SBU
**Required disk space:** 11 MB

### 🔧 Preparing Gzip for compilation:
```bash
./configure --prefix=/usr --host=$LFS_TGT
```

### 🏗️ Compiling the Package:
```bash
make
```
### 📦 Installing the Package:
```bash
make DESTDIR=$LFS install
```

## 6.12. Make-4.4.1
**The Make package contains a program for controlling the generation of executables and other non-source files of a package from source files.**
**Approximate build time:** less than 0.1 SBU
**Required disk space:** 15 MB

### 🔧 Preparing Make for compilation:
```bash
./configure --prefix=/usr   \
            --without-guile \
            --host=$LFS_TGT \
            --build=$(build-aux/config.guess)
```
### 🔍 Explanation of Installation Commands
- `--without-guile`: Although we are cross-compiling, configure tries to use guile from the build host if it finds it. This makes compilation fail, so this switch prevents using it.

### 🏗️ Compiling the package:
```bash
make
```
### 📦 Installing the package:
```bash
make DESTDIR=$LFS install
```

## 6.13. Patch-2.7.6
**The patch package contains a program for modifying or creating files by applying a "patch" file typically created by the `diff` program**

**Approximate build time:** 0.1 SBU
**Required disk space:** 12 MB

### 🔧 Preparing patch for compilation:
```bash
./configure --prefix=/usr   \
            --host=$LFS_TGT \
            --build=$(build-aux/config.guess)
```

### 🏗️ Compiling the package:
```bash
make
```
### 📦 Installing the package:
```bash
make DESTDIR=$LFS install
```

## 6.14. Sed-4.9
**The sed package contains a stream editor**

**Approximiate build time:** 0.1 SBU
**Required disk space:** 21 MB

### 🔧 Preparing Sed for compilation:
```bash
./configure --prefix=/usr   \
            --host=$LFS_TGT \
            --build=$(./build-aux/config.guess)
```
### 🏗️ Compiling the package:
```bash
make
```
### 📦 Installing the package:
```bash
make DESTDIR=$LFS install
```

## 6.15. Tar-1.35
**The Tar package provides the ability to create tar archives as well as perform various other kinds of archive manipulation. Tar can be used on previously created archives to extract files, to store additional files, or to update or list files which were already stored.**

**Approximiate build time:** 0.1 SBU
**Required disk space:** 42 MB

### 🔧 Preparing Tar for compilation:
```bash
./configure --prefix=/usr                     \
            --host=$LFS_TGT                   \
            --build=$(build-aux/config.guess)
```
### 🏗️ Compiling the package:
```bash
make
```
### 📦 Installing the package:
```bash
make DESTDIR=$LFS install
```

## 6.16. Xz-5.6.2
**The Xz package contains programs for compressing and decompressing files. It provides capabilities for the lzma and the newer xz compression formats. Compressing text files with xz yields a better compression percentage than with the traditional gzip or bzip2 commands.**

**Approximiate build time:** 0.1 SBU
**Required disk space:** 20 MB

### 🔧 Preparing Xz for compilation:
```bash
./configure --prefix=/usr                     \
            --host=$LFS_TGT                   \
            --build=$(build-aux/config.guess) \
            --disable-static                  \
            --docdir=/usr/share/doc/xz-5.6.2
```
### 🏗️ Compiling the package:
```bash
make
```
### 📦 Installing the package:
```bash
make DESTDIR=$LFS install
```

Remove the libtool archive file because it is harmful for cross compilation:
```bash
rm -v $LFS/usr/lib/liblzma.la
```

## 6.17. Binutils-2.43.1 - Pass 2
**The Binutils package contains a linker, an assembler, and other tools for handling object files.**

**Approximiate build time:** 0.4 SBU
**Required disk space:** 549 MB


Binutils building system relies on an shipped libtool copy to link against internal static libraries, but the libiberty and zlib copies shipped in the package do not use libtool. This inconsistency may cause produced binaries mistakenly linked against libraries from the host distro. Work around this issue:

```bash
sed '6009s/$add_dir//' -i ltmain.sh
```
Create a seperate build directory again:
```bash
mkdir -v build
cd build
```
### 🔧 Preparing Binutils for compilation:
```bash
../configure                   \
    --prefix=/usr              \
    --build=$(../config.guess) \
    --host=$LFS_TGT            \
    --disable-nls              \
    --enable-shared            \
    --enable-gprofng=no        \
    --disable-werror           \
    --enable-64-bit-bfd        \
    --enable-new-dtags         \
    --enable-default-hash-style=gnu
```
### 🔍 Explanation of Installation Commands
- `--enable-shared` Builds **libbfd** as a shared library.
- `--enable-64-bit-bfd` Enables 64-bit support (on hosts with smaller word sizes). This may not be needed on 64-bit systems, but it does no harm.

### 🏗️ Compiling the package:
```bash
make
```
### 📦 Installing the package:
```bash
make DESTDIR=$LFS install
```
Remove the libtool archive file because it is harmful for cross compilation and remove unnecessary static libraries:
```bash
rm -v $LFS/usr/lib/lib{bfd,ctf,ctf-nobfd,opcodes,sframe}.{a,la}
```

## 6.18. GCC-14.2.0-Pass 2
**The GCC package contains the GNU Compiler Collection, which includes the C and C++ compilers.**

**Approxiamiate build time:** 4.2 SBU
**Required disk space:** 5.5 GB

As in the first build of GCC, the GMP, MPFR, and MPC packages are required. Unpack the tarballs and move them into the required directories:

```bash
tar -xf ../mpfr-4.2.1.tar.xz
mv -v mpfr-4.2.1 mpfr
tar -xf ../gmp-6.3.0.tar.xz
mv -v gmp-6.3.0 gmp
tar -xf ../mpc-1.3.1.tar.gz
mv -v mpc-1.3.1 mpc
```

If building on x86_64, change the default directory name for 64-bit libraries to “lib”:
```bash
case $(uname -m) in
  x86_64)
    sed -e '/m64=/s/lib64/lib/' \
        -i.orig gcc/config/i386/t-linux64
  ;;
esac
```
Override the building rule of libgcc and libstdc++ headers, to allow building these libraries with POSIX threads support:
```bash
sed '/thread_header =/s/@.*@/gthr-posix.h/' \
    -i libgcc/Makefile.in libstdc++-v3/include/Makefile.in
```
Create a separate build directory again:
```bash
mkdir -v build
cd build
```
Before starting to build GCC, remember to unset any environment variables that override the default optimization flags.

### 🔧 Now Preparing GCC for compilation:
```bash
../configure                                       \
    --build=$(../config.guess)                     \
    --host=$LFS_TGT                                \
    --target=$LFS_TGT                              \
    LDFLAGS_FOR_TARGET=-L$PWD/$LFS_TGT/libgcc      \
    --prefix=/usr                                  \
    --with-build-sysroot=$LFS                      \
    --enable-default-pie                           \
    --enable-default-ssp                           \
    --disable-nls                                  \
    --disable-multilib                             \
    --disable-libatomic                            \
    --disable-libgomp                              \
    --disable-libquadmath                          \
    --disable-libsanitizer                         \
    --disable-libssp                               \
    --disable-libvtv                               \
    --enable-languages=c,c++
```

### 🔍 Explanation of Installation Commands

- `--with-build-sysroot=$LFS` Normally, using `--host` ensures that a cross-compiler is used for building GCC, and that compiler knows that it has to look for headers and libraries in `$LFS`. But the build system for GCC uses other tools, which are not aware of this location. This switch is needed so those tools will find the needed files in `$LFS`, and not on the host.
- `--target=$LFS_TGT` We are cross-compiling GCC, so it's impossible to build target libraries (`libgcc` and `libstdc++`) with the GCC binaries compiled in this pass—those binaries won't run on the host. The GCC build system will attempt to use the host's C and C++ compilers as a workaround by default. Building the GCC target libraries with a different version of GCC is not supported, so using the host's compilers may cause the build to fail. This parameter ensures the libraries are built by GCC pass 1.
- `LDFLAGS_FOR_TARGET=...` Allow `libstdc++` to use the `libgcc` being built in this pass, instead of the previous version built in gcc-pass1. The previous version cannot properly support C++ exception handling because it was built without libc support
- `--disable-libsanitizer` Disable GCC sanitizer runtime libraries. They are not needed for the temporary installation. In gcc-pass1 it was implied by `--disable-libstdcxx`, and now we can explicitly pass it.

### 🏗️ Compiling the package:
```bash
make
```
### 📦 Installing the package:
```bash
make DESTDIR=$LFS install
```

As a finishing touch, create a utility symlink. Many programs and scripts run cc instead of gcc, which is used to keep programs generic and therefore usable on all kinds of UNIX systems where the GNU C compiler is not always installed. Running cc leaves the system administrator free to decide which C compiler to install:

```bash
ln -sv gcc $LFS/usr/bin/cc
```

---
