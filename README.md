# Singularity on Apple Silicon (M4) MacOS

Author: Mira Sohn

Date: September 5, 2026

[Singularity](https://docs.sylabs.io/guides/3.5/user-guide/index.html) container 
is not supported on Apple Silicon as of today. One workaround is to use 
a Linux virtualizer. The current workflow demonstrates how I set it up 
on my MacBook M4 with macOS Tahoe (v26.6.2).

## Install required tools

The following three main tools are required:

- [Homebrew](https://brew.sh/)
- [Qemu](https://www.qemu.org/)
- [Lima](https://lima-vm.io/)

Briefly, `brew` is a package manager used to install `qemu` and `lima`. Follow
the instructions in [Homebrew](https://brew.sh/) to install `brew`. 

```bash
$ brew --version
Homebrew 6.0.22
```

Once the `brew` command is available, install `qemu` and `lima` as demonstrated
below:

```bash
$ qemu-img --version
qemu-img version 11.1.1

$ lima --version
limactl version 2.2.0
```

## Initialize a new virtual machine (VM) 

You have two options to start your VM. 

- Option A: Initialize a Linux VM and install Singularity in it afterwards. 
- Option B: Initialize a Linux VM where Singularity has been already installed.

Here, we chose Option B, as demonstrated below:

```bash
$ limactl start template://apptainer --vm-type=vz --rosetta --cpus 4 --disk 100 --memory 8
```

The `--cpus`, `--disk`, and `--memory` parameters are optional but helpful when building 
a new Singularity container, which varies in using temporary disk space. Also, **this step 
is required only once for a new VM. You are good to directly execute this VM next time.**

Note that Singularity is now [Apptainer](https://apptainer.org/). Visit the following
resources to follow up on new changes.

- [Apptainer GitHub](https://github.com/apptainer/apptainer)
- [Apptainer Documentation](https://apptainer.org/docs/user/main/index.html)


# Execute the initialized VM


```bash
$ limactl shell apptainer
```
