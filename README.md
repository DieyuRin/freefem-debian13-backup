**English** | [简体中文](README.zh-CN.md)

---

# FreeFEM 4.17 · Debian 13 (trixie) · amd64 package backup

Backup `.deb` of FreeFEM 4.17 (with PETSc / SLEPc / MUMPS / MPICH) for **Debian 13 (trixie) / amd64**.

> The software itself is [FreeFEM](https://freefem.org), licensed under **LGPL-3.0**; upstream source: <https://github.com/FreeFem/FreeFem-sources>. This repository exists solely as a binary backup.

## 1. Origin

FreeFEM upstream publishes binaries only for **Ubuntu** (22.04 / 24.04 / 26.04), **Windows** and **macOS** — there is **no official `.deb` for Debian 13**, and the freefem++ shipped in Debian's own repositories is far too old (3.x/4.9 era, lacking modern features such as the PETSc plugin used by the scripts in this project).

The key fact: the official FreeFEM docker image **`freefem/freefem:v4.17-docker-test`** happens to be built on **Debian 13 (trixie)** — so an official, complete FreeFEM 4.17 compiled for Debian 13 (with PETSc + SLEPc + MUMPS + MPICH) does exist, it is just wrapped inside a docker image instead of being released as a `.deb`.

This package is that **native FreeFEM 4.17 extracted from the official image** — the "de-containerized" official Debian 13 build of 4.17.

## 2. What we did

1. **Extract**: copied the whole `/usr/local` install tree from the official image `freefem/freefem:v4.17-docker-test` (FreeFem++ executables, the `ff++/4.17` plugins & idp macros, and PETSc/SLEPc/MUMPS/MPICH under `ff-petsc`) onto a Debian 13 host.
2. **System dependencies**: installed `libhdf5-310`; installed OpenBLAS and switched the system `libblas.so.3` / `liblapack.so.3` to OpenBLAS (PETSc needs OpenBLAS' `dgemmt_` symbol; this mirrors the image's own configuration).
3. **Packaged**: turned the install tree into a standard `.deb` (xz-compressed, ~365 MB) with a built-in `postinst` that automatically points BLAS/LAPACK at OpenBLAS on installation, so the result works out of the box.

## 3. Why we did it

- **Debian 13 users cannot obtain an official 4.17 `.deb`** — the most direct reason;
- building the full version from source (PETSc from scratch) takes **hours** and many steps;
- extracting from the official Debian 13 image takes **minutes** and yields the official native build, behaving identically to the official Ubuntu release;
- packaging it as a `.deb` makes it easy to **back up, share, install offline, and manage via dpkg** (install / remove / status).

## 4. How to use

**Requirements**: Debian 13 "trixie", x86_64 / amd64. Other distros or Debian 12 and older are not guaranteed to work (different library versioning).

**Install**

```bash
# Online (resolves dependencies automatically)
sudo dpkg -i freefem_4.17-2_amd64.deb
sudo apt-get -f install

# Fully offline — also need the two dependency packages:
#   libhdf5-310, libopenblas0-pthread
sudo dpkg -i libhdf5-310_*.deb libopenblas0-pthread_*.deb freefem_4.17-2_amd64.deb
```

**Run**

```bash
# Scripts using PETSc (e.g. load "PETSc") need the MPI build:
ff-mpirun -n 8 your-script.edp     # -n = number of processes

# Plain scripts, serial:
FreeFem++-nw your-script.edp
```

Everything installs to `/usr/local/bin`, with command-line usage identical to the official Ubuntu build. FreeFEM's idp macros (`macro_ddm.idp`, `ffddm.idp`, …) and all plugins are included.


**GUI (interactive plotting)**

FreeFEM's graphical output (plot windows via ffglut) requires `freeglut3-dev`, which is declared as a dependency of this package — online installs pull it in automatically; for fully offline installs, also carry the `freeglut3-dev` package.
**Package contents**: `FreeFem++` / `FreeFem++-mpi` / `FreeFem++-nw`, `ff-mpirun`, `bamg`, `cvmsh2`, `ff-c++`, etc.; `/usr/local/lib/ff++/4.17` (208 plugin `.so`, 28 idp macros, 327 headers); `/usr/local/ff-petsc` (PETSc 3.25 + SLEPc + MUMPS + MPICH).

**Checksum**

```
SHA-256 (freefem_4.17-2_amd64.deb) = 2c81580e94c79187f6d1bed32c92019f08f677bdfe32b079e822d22cd394c1b9
```

**Uninstall**

```bash
sudo dpkg -r freefem
```
