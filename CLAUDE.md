# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Athena++ is a C++ astrophysical magnetohydrodynamics (MHD) code with adaptive mesh refinement (AMR) capabilities. It simulates various physics including hydrodynamics, MHD, general relativity, radiation transport, cosmic rays, and chemistry.

## Common Development Commands

### Building the Code

1. **Configure the build** (from repository root):
   ```bash
   python configure.py --prob=<problem_name> [options]
   ```
   Common options:
   - `-b`: Enable magnetic fields (MHD)
   - `-mpi`: Enable MPI parallelization
   - `-omp`: Enable OpenMP parallelization
   - `-hdf5`: Enable HDF5 output (requires HDF5 library)
   - `--hdf5_path=<path>`: Path to HDF5 libraries
   - `-g`: Enable general relativity
   - `-s`: Enable special relativity
   - `-debug`: Enable debug flags (-g -O0)
   - `--flux=<solver>`: Choose Riemann solver (hlle, hllc, hlld, roe, llf)
   - `--coord=<system>`: Coordinate system (cartesian, cylindrical, spherical_polar)

2. **Compile**:
   ```bash
   make
   ```
   The executable will be created at `bin/athena`

3. **Clean build**:
   ```bash
   make clean
   ```

### Running Tests

1. **Run all regression tests**:
   ```bash
   cd tst/regression
   python run_tests.py
   ```

2. **Run specific test suite**:
   ```bash
   cd tst/regression
   python run_tests.py hydro/
   ```

3. **Run individual test**:
   ```bash
   cd tst/regression
   python run_tests.py hydro/hydro_linwave
   ```

4. **Run with MPI**:
   ```bash
   cd tst/regression
   python run_tests.py --mpirun=mpirun --mpirun_opts="-n 4"
   ```

### Running Simulations

```bash
bin/athena -i <input_file> [-r <restart_file>] [-d <output_dir>]
```

Input files are located in `inputs/` subdirectories organized by physics module.

### Code Style and Linting

The project uses cpplint for code style checking:
- Line length limit: 90 characters
- Header files use `.hpp` extension
- Follow existing indentation and formatting patterns

## Code Architecture

### Core Components

- **src/mesh/**: Mesh and MeshBlock classes, AMR infrastructure
- **src/hydro/**: Hydrodynamics module and Riemann solvers
- **src/field/**: Magnetic field evolution (for MHD)
- **src/eos/**: Equations of state
- **src/coordinates/**: Coordinate systems (Cartesian, cylindrical, spherical, GR metrics)
- **src/bvals/**: Boundary conditions and communication
- **src/pgen/**: Problem generators (initial conditions)
- **src/outputs/**: Output formats (VTK, HDF5, etc.)
- **src/task_list/**: Task-based execution framework

### Key Classes

- **Mesh**: Top-level container managing all MeshBlocks
- **MeshBlock**: Fundamental unit of the mesh, contains all physics data
- **Hydro**: Hydrodynamics variables and methods
- **Field**: Magnetic field variables and methods (when MHD enabled)
- **Coordinates**: Metric and coordinate transformation functions
- **TimeIntegrator**: Manages time stepping and task execution

### Problem Generators

Problem generators define initial conditions and are located in `src/pgen/`. To add a new problem:
1. Create a new file `src/pgen/my_problem.cpp`
2. Implement the `MeshBlock::ProblemGenerator()` function
3. Configure with `--prob=my_problem`

### Physics Modules

Modules are enabled at compile time through configure.py:
- Hydrodynamics (always enabled)
- MHD (`-b` flag)
- Special/General Relativity (`-s`/`-g` flags)
- Radiation (`-nr_radiation` or `-implicit_radiation`)
- Cosmic Rays (`-cr` flag)
- Chemistry (`--chemistry=<network>`)
- Self-gravity (`--grav=fft` or `--grav=mg`)

### Boundary Conditions

Standard boundary conditions include:
- reflecting
- outflow
- periodic
- user (custom, defined in problem generator)
- polar (for spherical coordinates)
- shearing_box (for orbital advection)

### Development Workflow

1. Fork the repository
2. Create a feature branch for your changes
3. Follow the code style guidelines (CPPLINT compatible)
4. Add regression tests for new functionality
5. Ensure all tests pass before submitting PR
6. Reference any related issues in commit messages

### Testing Guidelines

Tests are Python scripts in `tst/regression/scripts/tests/` that must define:
- `prepare()`: Configure and compile
- `run()`: Execute the test
- `analyze()`: Verify results

Use utilities from `scripts/utils/athena.py` for compilation and execution.

## Important Notes

- The code requires C++11 or later
- MPI and OpenMP can be used together for hybrid parallelization
- HDF5 output requires HDF5 library to be installed
- FFTW library required for FFT-based self-gravity
- Code coverage analysis available with `--coverage` flag in tests