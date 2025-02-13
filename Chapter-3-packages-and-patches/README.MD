# Chapter 3: Packages and Patches

## Chapter 3.1 Introduction

We need to build a **sources** folder inside **$LFS**, which we will use to store packages for the LFS project.

### Make the sources folder
```bash
mkdir -v $LFS/sources
```

### Make the folder writable and sticky
**Sticky** means only the owner has permission to delete files, in case multiple users have access to the folder.

```bash
chmod -v a+wt $LFS/sources
```

### Package Downloading using wget command

```bash
wget --input-file=wget-list-sysv --continue --directory-prefix=$LFS/sources
```

You can get the `wget-list-sysv` from the repository itself or from the [LFS](https://www.linuxfromscratch.org/lfs/view/stable/chapter03/introduction.html) book page.

### Verifying the packages
To verify the packages, use the following command:
```bash
pushd $LFS/sources
  md5sum -c md5sums
popd
```
You can get the **md5sums** from the repo itself or the LFS book page.

### Screenshot of my md5sums result
![md5sums output](screenshots/md5.png)

I got two warnings, so I installed those packages from [here](https://lfs.gnlug.org/pub/lfs/lfs-packages/10.0/).

You can use the `grep` command to see which packages are causing issues. For example:
```bash
md5sum -c md5sums | grep Failed
```

```bash
wget https://lfs.gnlug.org/pub/lfs/lfs-packages/10.0/expat-2.2.9.tar.xz --directory=$LFS/sources
wget https://lfs.gnlug.org/pub/lfs/lfs-packages/10.0/perl-5.32.0.tar.xz --directory=$LFS/sources
```

If you are using a non-root user, the packages will be owned by the user. To avoid any future conflicts, use the following command to change the ownership to root:
```bash
chown root:root $LFS/sources/*
```

## Chapter 3.2 All Packages

We have already completed this step in the introduction part. Here, all the packages are listed that we need to install, with their links. We already installed them in Chapter 3.1.

## Chapter 3.3 Needed Patches

We have already completed this step in the introduction part. Here, all the packages are listed that we need to install, with their links. We already installed them in Chapter 3.1.

