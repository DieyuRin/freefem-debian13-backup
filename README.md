# FreeFEM 4.17 · Debian 13 (trixie) · amd64 安装包备份

FreeFEM 4.17 `.deb` 安装包备份（含 PETSc / SLEPc / MUMPS / MPICH），面向 **Debian 13 (trixie) / amd64**。

> 软件本体为 [FreeFEM](https://freefem.org)，遵循 **LGPL-3.0** 开源许可；上游源码见 <https://github.com/FreeFem/FreeFem-sources>。本仓库仅作二进制备份用途。

---

## 简体中文

### 一、这个东西的来源

FreeFEM 官方只发布 **Ubuntu**（22.04 / 24.04 / 26.04）、**Windows** 和 **macOS** 的二进制，**没有面向 Debian 13 的官方 `.deb`**；而 Debian 自带仓库里的 freefem++ 版本停留在很老的 3.x/4.9 时代，缺少本仓库脚本依赖的 PETSc 插件等现代功能。

关键事实：FreeFEM 官方的 docker 镜像 **`freefem/freefem:v4.17-docker-test`** 恰好是基于 **Debian 13 (trixie)** 构建的——也就是说官方其实存在一套为 Debian 13 编译的完整 4.17（含 PETSc+SLEPc+MUMPS+MPICH），只是被封装成了 docker 镜像而没有单独发布 `.deb`。

本安装包就是**从该官方镜像中提取出来的原生 FreeFEM 4.17**，即官方 Debian 13 版 4.17 的"去容器化"版本。

### 二、我们做了什么

1. **提取**：从官方镜像 `freefem/freefem:v4.17-docker-test` 将完整的 `/usr/local` 安装树（FreeFem++ 系列可执行文件、`ff++/4.17` 插件与 idp 宏、`ff-petsc` 下的 PETSc/SLEPc/MUMPS/MPICH）拷贝到 Debian 13 宿主机。
2. **补齐系统依赖**：安装 `libhdf5-310`；安装 OpenBLAS 并将系统 `libblas.so.3` / `liblapack.so.3` 切到 OpenBLAS（PETSc 依赖 OpenBLAS 的 `dgemmt_` 符号，镜像原始配置即是如此）。
3. **端到端验证**：
   - 普通版与 MPI 版均正常启动运行；
   - `ff-mpirun -n 10` 下 PETSc 插件加载、MUMPS 求解、网格 DDM 分区全部正常，跑通了 Marangoni 伪弧长延拓求解器；
   - 同一 Poisson 算例与服务器上的官方 Ubuntu `.deb` 版结果一致到 ~1e-16（机器精度）；
   - `idp`(28)、`include`(327)、插件 `.so`(208) 与官方镜像**逐文件 MD5 一致**。
4. **打包**：将安装树打成标准 `.deb`（xz 压缩，约 365 MB），并内置 `postinst`——新机器安装时自动把 BLAS/LAPACK 指向 OpenBLAS，装完即可直接使用。

### 三、为什么这么做

- **Debian 13 用户拿不到官方 4.17 的 `.deb`**，这是本包存在的最直接原因；
- 若从源码编译完整版（PETSc 从源码构建），需要**数小时**且步骤繁琐；
- 从官方 Debian 13 镜像提取只需**几分钟**，得到的是官方原生构建，行为与官方 Ubuntu 版一致；
- 打成 `.deb` 便于**备份、分发、离线安装、统一用 dpkg 管理（安装/卸载/查状态）**。

### 四、这个东西怎么用

**适用环境**：Debian 13 "trixie"，x86_64 / amd64 架构。其它发行版或 Debian 12 以下不保证可用（依赖库版本体系不同）。

**安装**

```bash
# 方法一：在线安装（自动补装依赖）
sudo dpkg -i freefem_4.17-1_amd64.deb
sudo apt-get -f install

# 方法二：完全离线
# 需同时准备好两个依赖包：libhdf5-310、libopenblas0-pthread
sudo dpkg -i libhdf5-310_*.deb libopenblas0-pthread_*.deb freefem_4.17-1_amd64.deb
```

**运行**

```bash
# 含 PETSc 的脚本（如 load "PETSc"）用 MPI 版：
ff-mpirun -n 8 你的脚本.edp      # -n 为进程数

# 普通脚本用串行版：
FreeFem++-nw 你的脚本.edp
```

安装后即位于 `/usr/local/bin`，与官方 Ubuntu 版的命令行用法完全一致。FreeFEM 自带 idp 宏（如 `macro_ddm.idp`、`ffddm.idp`）与全部插件均随包安装。

**包内包含**：`FreeFem++` / `FreeFem++-mpi` / `FreeFem++-nw`、`ff-mpirun`、`bamg`、`cvmsh2`、`ff-c++` 等可执行文件；`/usr/local/lib/ff++/4.17`（208 个插件 .so、28 个 idp 宏、327 个头文件）；`/usr/local/ff-petsc`（PETSc 3.25 + SLEPc + MUMPS + MPICH）。

**校验**

```
SHA-256 (freefem_4.17-1_amd64.deb) = 5076ec9b000e5671ff648ad5b71ce9ae0c30b98d0e5a72bb993f9bbf4b6c0371
```

**卸载**

```bash
sudo dpkg -r freefem
```

---

## English

### 1. Origin

FreeFEM upstream publishes binaries only for **Ubuntu** (22.04 / 24.04 / 26.04), **Windows** and **macOS** — there is **no official `.deb` for Debian 13**, and the freefem++ shipped in Debian's own repositories is far too old (3.x/4.9 era, lacking modern features such as the PETSc plugin used by the scripts in this project).

The key fact: the official FreeFEM docker image **`freefem/freefem:v4.17-docker-test`** happens to be built on **Debian 13 (trixie)** — so an official, complete FreeFEM 4.17 compiled for Debian 13 (with PETSc + SLEPc + MUMPS + MPICH) does exist, it is just wrapped inside a docker image instead of being released as a `.deb`.

This package is that **native FreeFEM 4.17 extracted from the official image** — the "de-containerized" official Debian 13 build of 4.17.

### 2. What we did

1. **Extract**: copied the whole `/usr/local` install tree from the official image `freefem/freefem:v4.17-docker-test` (FreeFem++ executables, the `ff++/4.17` plugins & idp macros, and PETSc/SLEPc/MUMPS/MPICH under `ff-petsc`) onto a Debian 13 host.
2. **System dependencies**: installed `libhdf5-310`; installed OpenBLAS and switched the system `libblas.so.3` / `liblapack.so.3` to OpenBLAS (PETSc needs OpenBLAS' `dgemmt_` symbol; this mirrors the image's own configuration).
3. **End-to-end verification**:
   - serial and MPI binaries both start and run;
   - under `ff-mpirun -n 10`: PETSc plugin loads, MUMPS solves, DDM mesh partitioning all work — a Marangoni pseudo-arclength continuation solver was run successfully;
   - the same Poisson benchmark matches the official Ubuntu `.deb` build on the server to ~1e-16 (machine precision);
   - `idp` (28), `include` (327) and plugin `.so` files (208) are MD5-identical to the official image, file by file.
4. **Packaged**: turned the install tree into a standard `.deb` (xz-compressed, ~365 MB) with a built-in `postinst` that automatically points BLAS/LAPACK at OpenBLAS on installation, so the result works out of the box.

### 3. Why we did it

- **Debian 13 users cannot obtain an official 4.17 `.deb`** — the most direct reason;
- building the full version from source (PETSc from scratch) takes **hours** and many steps;
- extracting from the official Debian 13 image takes **minutes** and yields the official native build, behaving identically to the official Ubuntu release;
- packaging it as a `.deb` makes it easy to **back up, share, install offline, and manage via dpkg** (install / remove / status).

### 4. How to use

**Requirements**: Debian 13 "trixie", x86_64 / amd64. Other distros or Debian 12 and older are not guaranteed to work (different library versioning).

**Install**

```bash
# Online (resolves dependencies automatically)
sudo dpkg -i freefem_4.17-1_amd64.deb
sudo apt-get -f install

# Fully offline — also need the two dependency packages:
#   libhdf5-310, libopenblas0-pthread
sudo dpkg -i libhdf5-310_*.deb libopenblas0-pthread_*.deb freefem_4.17-1_amd64.deb
```

**Run**

```bash
# Scripts using PETSc (e.g. load "PETSc") need the MPI build:
ff-mpirun -n 8 your-script.edp     # -n = number of processes

# Plain scripts, serial:
FreeFem++-nw your-script.edp
```

Everything installs to `/usr/local/bin`, with command-line usage identical to the official Ubuntu build. FreeFEM's idp macros (`macro_ddm.idp`, `ffddm.idp`, …) and all plugins are included.

**Package contents**: `FreeFem++` / `FreeFem++-mpi` / `FreeFem++-nw`, `ff-mpirun`, `bamg`, `cvmsh2`, `ff-c++`, etc.; `/usr/local/lib/ff++/4.17` (208 plugin `.so`, 28 idp macros, 327 headers); `/usr/local/ff-petsc` (PETSc 3.25 + SLEPc + MUMPS + MPICH).

**Checksum**

```
SHA-256 (freefem_4.17-1_amd64.deb) = 5076ec9b000e5671ff648ad5b71ce9ae0c30b98d0e5a72bb993f9bbf4b6c0371
```

**Uninstall**

```bash
sudo dpkg -r freefem
```

---

*The software itself is [FreeFEM](https://freefem.org), licensed under LGPL-3.0. Upstream source: <https://github.com/FreeFem/FreeFem-sources>.*
