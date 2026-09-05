# FreeFEM 4.17 — Debian 13 (trixie) amd64 backup

**Unofficial backup artifact. Not affiliated with / not endorsed by the FreeFEM project.**

- Software: [FreeFEM](https://freefem.org) v4.17 — finite element PDE solver (LGPL-3.0)
- Package: `freefem_4.17-1_amd64.deb` (contains FreeFem++, FreeFem++-mpi, ff-mpirun, PETSc, SLEPc, MUMPS, MPICH)
- Origin: repackaged from the official `freefem/freefem:v4.17-docker-test` image (Debian 13 base), built for **Debian 13 (trixie) amd64 only**
- Upstream source: https://github.com/FreeFem/FreeFem-sources (LGPL-3.0)

## Install
```bash
sudo dpkg -i freefem_4.17-1_amd64.deb
sudo apt-get -f install   # pulls libhdf5-310 + libopenblas0-pthread
```
Run: `ff-mpirun -n N script.edp` (PETSc) or `FreeFem++-nw script.edp`.
