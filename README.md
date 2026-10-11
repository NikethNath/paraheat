# paraheat

**A 2D heat-diffusion solver in C, written three ways: serial, OpenMP and MPI.** It's a small, complete HPC exercise:
- a 5-point stencil
- domain decomposition across MPI ranks, with **non-blocking halo exchange** that overlaps communication with computation
- validation against the **exact analytical solution**
- a measured **strong and weak scaling** study

> **Status: in active development.** This README describes the planned design. Error and scaling numbers are published only from real runs.

## The problem
`∂u/∂t = α∇²u` on the unit square, with zero boundaries and `u₀ = sin(πx)·sin(πy)`. The exact solution `u = e^(−2π²αt)·sin(πx)·sin(πy)` lets every run report its true error. The scheme is second-order in space, so the error should fall about 4× each time the grid spacing halves.

## How it's parallelised
```
OpenMP:  rows of the grid split across threads (parallel for, static schedule)

MPI:     P ranks in a 2D Cartesian grid (MPI_Cart_create)
         ┌──────┬──────┐   each rank owns a block + 1-cell ghost border
         │ r0   │ r1   │   per step: Irecv 4 halos → Isend 4 edges (MPI_Type_vector for columns)
         ├──────┼──────┤            → update interior while messages are in flight
         │ r2   │ r3   │            → Waitall → update the border cells
         └──────┴──────┘
```
One stencil function is shared by all three versions, and they produce **bit-identical** results, which the tests check.

## Planned usage
```bash
make
./heat_serial -n 1024 -steps 2000
OMP_NUM_THREADS=8 ./heat_omp -n 1024 -steps 2000
mpirun -np 4 ./heat_mpi -n 1024 -steps 2000
bench/scaling.sh   # strong + weak scaling → docs/results/
```

## Stack
C11, OpenMP, MPI (Open MPI), GNU Make, Bash, Python/matplotlib for plots, GitHub Actions.

## License
MIT
