# Singularity on Apple Silicon (M4) macOS

Author: Mira Sohn

Date: September 5, 2026

[Singularity](https://docs.sylabs.io/guides/3.5/user-guide/index.html) containers
are not supported on Apple Silicon yet. One workaround is to use 
a Linux virtualizer. This workflow demonstrates how I set it up 
on my MacBook M4 with macOS Tahoe (v26.6.2).

## Install required tools

The following three main tools are required:

- [Homebrew](https://brew.sh/)
- [QEMU](https://www.qemu.org/)
- [Lima](https://lima-vm.io/)

Briefly, `brew` is a package manager used to install `qemu` and `lima`. Follow
the instructions on the [Homebrew](https://brew.sh/) website to install `brew`. 

```bash
$ brew --version
Homebrew 6.0.22
```

Once the `brew` command is available, install `qemu` and `lima` as demonstrated
below:

```bash
$ brew install qemu lima
```

Confirm that both packages are installed.

```bash
$ qemu-img --version
qemu-img version 11.1.1

$ lima --version
limactl version 2.2.0
```

## Initialize a new virtual machine (VM) 

You have two options for starting your VM.

- Option A: Initialize a Linux VM and install Singularity in it afterward.
- Option B: Initialize a Linux VM where Singularity has already been installed.

Here, we choose Option B, as demonstrated below:

```bash
$ limactl start template://apptainer \
    --vm-type=vz \
    --rosetta \
    --mount-writable \
    --cpus 4 \
    --disk 100 \
    --memory 8
```

The `--cpus`, `--disk`, and `--memory` parameters are optional but helpful when building 
a new Singularity container or running a script, both of which can require varying amounts of
temporary disk space. The `--mount-writable` parameter is configured to enable
read/write access when creating a new Singularity container. Also, **this step is 
required only once for a new VM. You can directly execute this VM next time.**

Note that Singularity is now [Apptainer](https://apptainer.org/). Visit the following
resources to learn about recent changes.

- [Apptainer GitHub](https://github.com/apptainer/apptainer)
- [Apptainer Documentation](https://apptainer.org/docs/user/main/index.html)


## Execute the initialized VM

Once you are ready to execute the VM, call `limactl shell <VM_name>`, as demonstrated below:


```bash
$ limactl shell apptainer
```

When you are in the VM, both the `singularity` and `apptainer` commands are available.

```bash
# In VM:
$ singularity --version
apptainer version 1.5.3
$ apptainer --version
apptainer version 1.5.3
```

Either command works in the same way, as `singularity` is redirected to `apptainer`. 

## Build a new Singularity container

This section demonstrates how to build a new Singularity container where
[R](https://cran.r-project.org) is installed. This demonstration uses
the following two files: `requirements.txt` and `demo.def`.

### Prepare recipe files

#### `requirements.txt`

This file provides a recipe for the conda packages to be installed.

```bash
$ cat requirements.txt
r-base
```

#### `demo.def`

A Singularity definition file (`.def`) is a recipe used to build a custom container image.
To learn about individual sections, visit
[Definition Files](https://docs.sylabs.io/guides/3.7/user-guide/definition_files.html).

```bash
$ cat demo.def
Bootstrap: docker
From: condaforge/miniforge3

%environment
    export BASH_ENV=/opt/etc/bashrc
    source /opt/etc/bashrc

%files
    requirements.txt
%post
    ENVNAME=rbase

    # Ensure locales packages installed
    apt-get -y update
    apt-get install -y locales-all
    rm -rf /var/lib/apt/lists/*

    # Create target dir for non-root-accessible bashrc
    mkdir -p /opt/etc

    # Replace these demo packages with the packages you want to add.
    conda create -n "${ENVNAME}" -f requirements.txt

    conda clean -afy

    # Finalize bashrc file
    echo "#! /bin/bash\n\n# script to activate the conda environment" > ~/.bashrc
    conda init bash
    echo "echo \"Activating ${ENVNAME}\"" >> ~/.bashrc
    echo "\nconda activate ${ENVNAME}" >> ~/.bashrc

    # Copy bashrc to non-root-accessible location
    cp ~/.bashrc /opt/etc/bashrc

%runscript
    exec /bin/bash "$@"
```

### Build a container

Now you're ready to build a new container.

```bash
$ singularity build demo.sif demo.def
```
The Singularity Image File (`.sif`) stores the complete container file system, metadata, 
and cryptographic signatures. Note that the image file is immutable. If you would like to 
add more packages, add the conda package names to the `requirements.txt` file and build a new
container.

## Run the Singularity container

Confirm that R is available in your Singularity container:

```bash
$ singularity shell demo.sif
Activating rbase
(rbase) R --version
R version 4.6.1 (2026-06-24) -- "Happy Hop"
Copyright (C) 2026 The R Foundation for Statistical Computing
Platform: aarch64-conda-linux-gnu

R is free software and comes with ABSOLUTELY NO WARRANTY.
You are welcome to redistribute it under the terms of the
GNU General Public License versions 2 or 3.
For more information about these matters see
https://www.gnu.org/licenses/.
```
