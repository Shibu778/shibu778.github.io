---
title: Installing Quantum Espresso from Source
date: 2025-02-24 00:48:00
categories: [Compuational Materials Science]
tags: [Compiling, Quantum Espresso, Ubuntu]     # TAG names should always be lowercase
author: Shibu
description: Installing Quantum Espresso from Source Code.
toc: true # default is true
pin: false
comments: true
---


# Compiling Quantum Espresso from Source
You can directly install quantum espresso in your wsl or linux system by using the following command.

```bash
sudo apt-get install quantum-espresso
```

However, this way of installing can result in buffer overflow error sometime. One way to resolve this error is
to build the quantum espresso package. That is what we are going to do in this tutorial.

Follow the following step to compile quantum espresso package from source when you have the admin priviledges.


### Step 1: Installing required packages
 
```bash
sudo apt-get install build-essential                                                                                   
sudo apt-get install gcc gfortran                                                                                      
sudo apt-get install mpi                                                                                               
sudo apt-get install libomp-dev                                                                                        
sudo apt-get install openmpi-bin libopenmpi-dev                                                                        
sudo apt-get install libscalapack-openmpi-dev
sudo apt-get install libscalapack-mpich-dev
sudo apt-get install libscalapack-mpi-dev
sudo apt-get install git
```

### Step 2: Install FFTW package
 
```bash
sudo apt-get install libfftw3-dev libfftw3-doc
```

### Step 3: Get quantum espresso from Github

```bash
wget https://github.com/QEF/q-e/archive/qe-7.2.tar.gz
```

### Step 4: Decompress the `*.tar.gz` file.

```bash
tar -zxvf qe-7.2.tar.gz
```

### Step 5: Go into the folder
```bash
cd q-e-qe-7.2/
```

### Step 6: Configure
```bash
./configure --enable-openmp --with-scalapack=openmpi
```
 
### Step 7: Do `make all`. It will take some time so wait. Make sure you do not get any error at the end.
 
```bash
make all
```
 
### Step 8: Go to bin directory

```bash
cd bin
```
 
### Step 9: Do `ls` and check whether following files are present.
 
```bash
ls
```
 
You should see following. Check whether `pw.x` is present or not.
 
```bash
<somepath>/q-e-qe-7.2/bin$ ls
alpha2f.x             epsilon.x         kcwpp_sh.x            ph.x        pw2critic.x           sumpdos.x
average.x             ev.x              kpoints.x             phcg.x      pw2gw.x               turbo_davidson.x
band_interpolation.x  fermi_proj.x      lambda.x              plan_avg.x  pw2wannier90.x        turbo_eels.x
bands.x               fermi_velocity.x  ld1.x                 plotband.x  pw4gww.x              turbo_lanczos.x
bse_main.x            fqha.x            manycp.x              plotproj.x  pwcond.x              turbo_magnon.x
cell2ibrav.x          fs.x              manypw.x              plotrho.x   pwi2xsf.x             turbo_spectrum.x
cp.x                  gww.x             matdyn.x              pmw.x       q2qstar.x             wannier90.x
cppp.x                gww_fit.x         molecularnexafs.x     postahc.x   q2r.x                 wannier_ham.x
d3hess.x              head.x            molecularpdos.x       pp.x        rism1d.x              wannier_plot.x
dist.x                hp.x              neb.x                 ppacf.x     scan_ibrav.x          wfck2r.x
dos.x                 ibrav2cell.x      open_grid.x           pprism.x    simple.x              wfdd.x
dvscf_q2r.x           initial_state.x   oscdft_et.x           projwfc.x   simple_bse.x          xspectra.x
dynmat.x              kcw.x             oscdft_pp.x           pw.x        simple_ip.x
epa.x                 kcwpp_interp.x    path_interpolation.x  pw2bgw.x    spectra_correction.x

```

### Step 10: Copy you `bin` path. It will look like the following.
```bash
<some_path>/q-e-qe-7.2/bin
```

### Step 11: Now open `~/.bashrc` and go to the last of the file. You can use `shift+g` to go to the last line.

```bash
vi ~/.bashrc
```
 
Paste the following line to the file. Replace <your_path_to_qe_bin> with your bin path.
```bash
export PATH="<your_path_to_qe_bin>:$PATH"
```

It adds the path to the library of paths so that you can access pw.x file inside the bin directory from anywhere in your system.
 
### Step 12: Do source `~/.bashrc` to make the changes to take effect.
```bash
source ~/.bashrc
```

### Step 13: Now go to your calculation directory and try running `pw.x`.
```bash
pw.x < si.scf.in > si.scf.out
```


## Reference
1. [https://vladgladkikh.wordpress.com/2020/10/13/quantum-espresso-on-the-windows-subsystem-for-linux-2-wsl-2/](https://vladgladkikh.wordpress.com/2020/10/13/quantum-espresso-on-the-windows-subsystem-for-linux-2-wsl-2/)
