[![DOI](https://zenodo.org/badge/209853926.svg)](https://zenodo.org/badge/latestdoi/209853926) ![C/C++ CI](https://github.com/pkestene/ppkMHD/workflows/C/C++%20CI/badge.svg)

# ppkMHD

## What is it ?

ppkMHD stands for Performance Portable Kokkos for Magneto-HydroDynamics (MHD) solvers.

Here a small list of numerical schemes implementations:

- second order MUSCL-HANCOCK scheme for hydro and MHD
- high-order MOOD (hydro only)
- high-order Spectral Difference Method schemes: hydro only

All scheme are available in 2D and 3D using Kokkos+MPI implementation.

## Dependencies

* [Kokkos](https://github.com/kokkos/kokkos) library will be built by ppkMHD using the same flags (architecture, optimization, ...).
* [CMake](https://cmake.org/) with version >= 3.X (3.X is chosen to meet Kokkos own requirement for CMake; i.e. it might increase in the future)

Current application is configured with kokkos library as a git submodule. So you'll need to run the following git commands right after cloning ppkMHD:

```shell
git submodule init
git submodule update
```

Kokkos is built with the same flags as the main application.

## Build

A few example builds, with minimal configuration options.

### Build without MPI / With Kokkos-openmp

* Create a build directory, configure and make

```shell
mkdir build; cd build
cmake -DUSE_MPI=OFF -DKokkos_ENABLE_OPENMP=ON -DKokkos_ENABLE_HWLOC=OFF -DUSE_SDM=ON ..
make -j 4
```

Add variable CXX on the cmake command line to change the compiler (clang++, icpc, pgcc, ....).

### Build without MPI / With Kokkos-openmp for Intel KNL

* Create a build directory, configure and make

```shell
export CXX=icpc
mkdir build; cd build
cmake -DUSE_MPI=OFF -DKokkos_ARCH_KNL=ON -DKokkos_ENABLE_OPENMP=ON ..
make -j 4
```

### Build without MPI / With Kokkos-cuda

To be able to build with CUDA backend, you need to use nvcc_wrapper located in
kokkos source (external/kokkos/bin/nvcc_wrapper).

* Create a build directory, configure and make

```shell
mkdir build; cd build
export CXX=/path/to/nvcc_wrapper
cmake -DUSE_MPI=OFF -DKokkos_ENABLE_CUDA=ON -DKokkos_ARCH_PASCAL61=ON ..
cmake -DUSE_MPI=OFF -DKokkos_ENABLE_CUDA=ON -DKokkos_ARCH_PASCAL61=ON -DUSE_SDM=ON ..
cmake -DUSE_MPI=OFF -DKokkos_ENABLE_CUDA=ON -DKokkos_ARCH_PASCAL61=ON -DUSE_SDM=ON -DUSE_MOOD=ON ..
make -j 4
```

`nvcc_wrapper` is a compiler wrapper arroud NVIDIA `nvcc`. It is available from Kokkos sources: `external/kokkos/bin/nvcc_wrapper`. Any Kokkos application target NVIDIA GPUs must be built with `nvcc_wrapper`.

### Build with MPI / With Kokkos-cuda

Please make sure to use a CUDA-aware MPI implementation (OpenMPI or MVAPICH2) built with the proper flags for activating CUDA support.

It may happen that eventhough your MPI implementation is actually cuda-aware, cmake find_package macro for MPI does not detect it to be cuda aware. In that case, you can enforce cuda awareness by turning option USE_MPI_CUDA_AWARE_ENFORCED to ON.

You don't need to use mpi compiler wrapper mpicxx, cmake *should* be able to correctly populate MPI_CXX_INCLUDE_PATH, MPI_CXX_LIBRARIES which are passed to all final targets.

* Create a build directory, configure and make

```shell
mkdir build; cd build
export CXX=/path/to/nvcc_wrapper
cmake -DUSE_MPI=ON -DKokkos_ENABLE_CUDA=ON -DKokkos_ARCH_MAXWELL50=ON -DUSE_SDM=ON ..
make -j 4
```

Example command line to run the application (1 GPU used per MPI task)

```shell
mpirun -np 4 ./ppkMHD ./test_implode_2D_mpi.ini
```

### Additionnal requirements

In order to activate building SDM schemes, use Cmake option `-DUSE_SDM=ON`

The MOOD numerical scheme require some linear algebra (QR decomposition) on the host (not device). This is done using a Blas/Lapack implementation using the C language interface named Lapacke.

Please note that Atlas doesn't provide Lapackage.
Currently (March 2017), on Ubuntu 16.04, package libatlas-dev is not compatible with package Lapacke (generate errors at link time). So please either Netlib or OpenBLAS implementation.

If you want to enforce the use of OpenBLAS, just use a recent CMake (>=3.6) and add "-DBLA_VENDOR" on the cmake command line. This will tell the cmake system (through the call to find_package(BLAS) ) to only look for OpenBLAS implementation.

On a recent Ubuntu, if atlas is not installed, but OpenBLAS is, you don't need to have a bleeding edge CMake, current cmake will find OpenBLAS.

### Developping with vim or emacs and semantic completion/navigation from ccls

Make sure to have CMake variable CMAKE_EXPORT_COMPILE_COMMANDS set to ON, it will generate a file named _compile_commands.json_.
Then you can symlink the generated file in the top level source directory.

Please visit :
* [ccls](https://github.com/MaskRay/ccls)
* [editor configuration for using ccls](https://github.com/MaskRay/ccls/wiki/Editor-Configuration)
* [project setup for using ccls](https://github.com/MaskRay/ccls/wiki/Project-Setup)

## See also

* [ppkMHD Wiki](https://github.com/pkestene/ppkMHD/wiki)
* [Implementing Spectral Difference Methods (SDM) for Compressible Euler flow simulations using performance portable library kokkos](https://www.researchgate.net/publication/326400645_Implementing_Spectral_Difference_Methods_SDM_for_Compressible_Euler_flow_simulations_using_performance_portable_library_kokkos)
* [Implementing Spectral Difference Methods (SDM) for Compressible Euler flow simulations using performance portable library kokkos (astrosim 2018)](https://www.researchgate.net/publication/328175816_Implementing_Spectral_Difference_Methods_SDM_for_Compressible_Euler_flow_simulations_using_performance_portable_library_kokkos)
