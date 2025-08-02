---
title: Installing QE and VASP (GPU version)
date: 2025-08-02
categories: [Computational Materials Science]
tags: [compiling, quantum espresso, vasp]
author: Shibu
description: Comprehensive guide to installing Quantum Espresso and VASP (GPU version) on Linux with NVIDIA HPC SDK.
toc: true
pin: false
comments: true
---

# Comprehensive Tutorial: Installing Quantum Espresso and VASP (GPU Version) on Linux

This tutorial provides a step-by-step guide to installing Quantum Espresso (QE) and VASP (GPU version) on a Linux system using the NVIDIA HPC SDK. The instructions are based on the following versions:

- **NVIDIA HPC-SDK:** 23.5
- **Quantum Espresso:** qe-7.4.1 (only `pw.x`)
- **VASP:** 6.4.2

> **Note:** For Quantum Espresso, only `pw.x` was tested. The same method should work for other QE subpackages.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Step 1: Install NVIDIA HPC SDK](#step-1-install-nvidia-hpc-sdk)
3. [Step 2: Install Quantum Espresso (GPU)](#step-2-install-quantum-espresso-gpu)
4. [Step 3: Install FFTW Library](#step-3-install-fftw-library)
5. [Step 4: Install VASP (GPU)](#step-4-install-vasp-gpu)
6. [Troubleshooting & Tips](#troubleshooting--tips)

---

## Prerequisites

- A Linux system (tested on CentOS 7)
- Sudo/admin is not required
- Compatible NVIDIA GPU and drivers
- Basic familiarity with the terminal

---

## Step 1: Install NVIDIA HPC SDK

Download and install the NVIDIA HPC SDK, which provides compilers and libraries required for GPU-accelerated scientific computing.

### Download and Extract

```bash
wget https://developer.download.nvidia.com/hpc-sdk/23.5/nvhpc_2023_235_Linux_x86_64_cuda_multi.tar.gz
tar xpzf nvhpc_2023_235_Linux_x86_64_cuda_multi.tar.gz
./nvhpc_2023_235_Linux_x86_64_cuda_multi/install
```

### Installation Notes

- Choose an installation directory (e.g., `/home/youruser/nvidia/hpc_sdk`).
- After installation, the SDK will be in `/home/youruser/nvidia/hpc_sdk/Linux_x86_64/23.5`.

### Set Environment Variables

Add the following to your `~/.bashrc` or source them in your shell:

```bash
export PATH=/home/youruser/nvidia/hpc_sdk/Linux_x86_64/23.5/compilers/bin:$PATH
export MANPATH=$MANPATH:/home/youruser/nvidia/hpc_sdk/Linux_x86_64/23.5/compilers/man
export PATH=/home/youruser/nvidia/hpc_sdk/Linux_x86_64/23.5/comm_libs/mpi/bin:$PATH
export MANPATH=$MANPATH:/home/youruser/nvidia/hpc_sdk/Linux_x86_64/23.5/comm_libs/mpi/man
```

Or, if you use environment modules:

```bash
module load /home/youruser/nvidia/hpc_sdk/modulefiles/nvhpc/23.5
```

---

## Step 2: Install Quantum Espresso (GPU)

### Clone the QE Repository

```bash
git clone -b 'qe-7.4.1' --single-branch --depth 1 https://gitlab.com/QEF/q-e.git QEdir
cd QEdir
git submodule init
git submodule update
```

### Prepare the Environment

- Ensure Intel compiler paths (e.g., `ifort`, `mpiifort`) are **not** active:
  ```bash
  which ifort
  # Should return: no ifort in (<paths>)
  ```

- Check CUDA compute capability and driver version:
  ```bash
  nvaccelinfo | grep -e 'Target' -e 'Driver'
  # Note the CUDA Driver Version and Default Target (e.g., cc70)
  ```

### Configure and Build QE

Create a script (or run commands manually):

```bash
#!/bin/bash
root_path=/home/youruser
cuda_runtime=12.1 # cuda driver version
cc=70 # compute capacity

module load /home/youruser/nvidia/hpc_sdk/modulefiles/nvhpc/23.5

./configure --with-cuda=$root_path/nvidia/hpc_sdk/Linux_x86_64/23.5/cuda/$cuda_runtime \
            --with-cuda-cc=$cc --with-cuda-runtime=$cuda_runtime --enable-openmp

make -j 8 pw  # Builds pw.x using 8 cores
# make -j 8 all  # To build all QE executables
```

### Running QE (`pw.x`)

Example PBS script:

```bash
#!/bin/sh
#PBS -N test
#PBS -q batch
#PBS -l nodes=1:ppn=1
#PBS -l walltime=72:00:00

cd $PBS_O_WORKDIR
module load /home/youruser/nvidia/hpc_sdk/modulefiles/nvhpc/23.5

binpath=/home/youruser/Code/gpu/QEdir/bin

export OMP_NUM_THREADS=1
mpirun -n 4 $binpath/pw.x -i pw.in > pw.out
```

---

If you do not have to install VASP stop here.

## Step 3: Install FFTW Library

FFTW is required for VASP.

```bash
cd /home/youruser/Code
wget https://fftw.org/fftw-3.3.10.tar.gz
tar xvf fftw-3.3.10.tar.gz
cd fftw-3.3.10
```

Build and install FFTW:

```bash
mkdir build
module load /home/youruser/nvidia/hpc_sdk/modulefiles/nvhpc/23.5
./configure --prefix=/home/youruser/Code/fftw-3.3.10/build
make
make check
make install
make installcheck
```

---

## Step 4: Install VASP (GPU)

### Prepare VASP Source

```bash
cd /home/youruser/Code
tar xvf vasp.6.4.2.tgz
cd vasp.6.4.2
cp arch/makefile.include.linux_nv makefile.include
```

### Edit `makefile.include`

- Set the FFTW path:
  ```makefile
  FFTW ?= /home/youruser/Code/fftw-3.3.10/build # TOBECHANGED
  ```
- Ensure `NVROOT` is set correctly:
  ```makefile
  NVROOT = $(shell which nvfortran | awk -F /compilers/bin/nvfortran '{ print $$1 }')
  ```
- Other paths and flags can usually remain as defaults unless your setup differs.

### Build VASP

```bash
make all
```

If successful, you will find `vasp_std`, `vasp_gam`, and `vasp_ncl` in the `bin` directory.

---

## Troubleshooting & Tips

- **Compiler Conflicts:** Ensure only NVIDIA compilers are in your `PATH` during build.
- **Module System:** If you use `module load`, ensure the correct modulefiles are loaded for each step.
- **CUDA Version:** Match the CUDA version in your system and in the HPC SDK.
- **Permissions:** Run commands as your user unless root is required.
- **Documentation:** Refer to official [NVIDIA HPC SDK Docs](https://docs.nvidia.com/hpc-sdk/), [Quantum Espresso Docs](https://www.quantum-espresso.org/) and [VASP Wiki](https://www.vasp.at/wiki/index.php/Installing_VASP.6.X.X).

---

## Conclusion

You now have a working installation of Quantum Espresso and VASP (GPU version) using the NVIDIA HPC SDK. This setup enables high-performance, GPU-accelerated simulations for computational materials science.




