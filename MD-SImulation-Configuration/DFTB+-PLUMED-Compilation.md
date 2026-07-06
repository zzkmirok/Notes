# DFTB+ 25.1 + PLUMED Compilation Workflow

Compiling **DFTB+ 25.1** patched with **PLUMED** (metadynamics), MPI-parallel, for
the **Sapphire Rapids CPU nodes** of the RWTH cluster. Reuses the *existing*
custom PLUMED 2.10.0 build (no PLUMED recompilation needed).

> **Status: ✅ built, linked and installed successfully (2026-07-06).**
> Binary: `/rwthfs/rz/cluster/work/yy508225/mydftb25.1/DFTBP25.1INSTALL/bin/dftb+`
> Verified: prints `DFTB+ release 25.1`, MPI initialises, `ldd` shows
> `libplumed.so`/`libplumedKernel.so` (dynamic), `libmpi.so.40` (OpenMPI 5.0.3),
> `libscalapack.so`, `libflexiblas.so.3`, no unresolved libraries.

---

## 1. Target & prerequisites

| Item | Value |
|------|-------|
| DFTB+ version | 25.1 (release tarball, submodules bundled under `external/*/origin`) |
| Source dir | `/rwthfs/rz/cluster/work/yy508225/mydftb25.1/dftbplus-25.1` |
| Install prefix | `/rwthfs/rz/cluster/work/yy508225/mydftb25.1/DFTBP25.1INSTALL` |
| Toolchain | `foss/2024a` → GCC **13.3.0**, OpenMPI **5.0.3** |
| BLAS/LAPACK | FlexiBLAS **3.4.4** (wraps OpenBLAS 0.3.27) |
| ScaLAPACK | **2.2.0-gompi-2024a-fb** |
| CMake | 3.29.3 |
| PLUMED | **2.10.0** custom, `/work/yy508225/myplumed/plumed2.10.0-custom` (reused as-is) |
| Target CPU | Intel Xeon Platinum 8468 (Sapphire Rapids, AVX-512) — login == compute |

### Does PLUMED need recompiling? **No.**
The existing PLUMED was built with the *exact* toolchain DFTB+ uses (GCC 13.3.0 +
OpenMPI 5.0.3 + FlexiBLAS, `-march=sapphirerapids`), is MPI-aware (`has mpi on`),
and ships `libplumed.so` + `libplumed.a` + a working `pkg-config`. DFTB+ links it
dynamically via `-lplumed`; all of PLUMED's own dependencies (torch, gsl, fftw3,
boost_serialization, metatensor …) are recorded as `DT_NEEDED` inside
`libplumed.so` and resolve transitively — so **no `LIBS="-ltorch …"` flags are
needed on the DFTB+ side**. Rebuild PLUMED only if the DFTB+ toolchain (GCC/MPI
ABI) changes.

---

## 2. Detailed workflow

### Step 0 — Activate the environment
The same env used to run PLUMED/LAMMPS. It injects GCC 13.3.0, OpenMPI 5.0.3,
FlexiBLAS, ScaLAPACK and the PLUMED `PKG_CONFIG_PATH`/`LD_LIBRARY_PATH`.

```bash
source /work/yy508225/RH9PYENV/PLUMED-GPU/bin/activate
cd /rwthfs/rz/cluster/work/yy508225/mydftb25.1/dftbplus-25.1
```

### Step 1 — Two pre-build patches

**1a. libMBD version tag.** The bundled libMBD determines its version via
`git describe`, which fails in a tarball (no `.git`). Supply the version-tag file
it looks for:

```bash
echo 'set(VERSION_TAG "0.12.3")' \
  > external/mbd/origin/cmake/libMBDVersionTag.cmake
```
> Only libMBD needs this — the other externals (tblite, s-dftd3, dftd4,
> multicharge, mctc-lib) hardcode `project(... VERSION ...)`. The version string
> is cosmetic (compiled into an `mbd_version` reporting module).

**1b. `WITH_API=OFF` compile bug in `initprogram.F90`.** DFTB+ 25.1 has a fypp
guard bug: the field `isASICallbackEnabled` is only declared under `#:if WITH_API`,
but `initprogram.F90` line ~1701 passes it to `ensureSolverCompatibility()`
**unguarded**. With `WITH_API=OFF` (which we need — see §3) this fails to compile:
`'isasicallbackenabled' is not a member of the 'tcontrol' structure`.

Fix the call site to pass `.false.` when the API is off (matches the codebase's
own inline-fypp idiom, cf. `elsiiface.F90`):

```fortran
! src/dftbp/dftbplus/initprogram.F90  (~line 1701)
!  in the call to ensureSolverCompatibility(...), replace the last argument
!    input%ctrl%isASICallbackEnabled
!  with
    #{if WITH_API}#input%ctrl%isASICallbackEnabled#{else}#.false.#{endif}#
```
> This is a build-config-dependent upstream bug; only appears when `WITH_API=OFF`.
> Report upstream if you update DFTB+.

### Step 2 — Configure
```bash
FC=gfortran CC=gcc CXX=g++ cmake -B _build . \
  -DCMAKE_INSTALL_PREFIX=/rwthfs/rz/cluster/work/yy508225/mydftb25.1/DFTBP25.1INSTALL \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_Fortran_FLAGS="-march=sapphirerapids" \
  -DCMAKE_C_FLAGS="-march=sapphirerapids" \
  -DWITH_MPI=ON -DWITH_OMP=ON \
  -DWITH_PLUMED=ON -DWITH_TRANSPORT=ON -DWITH_MBD=ON \
  -DWITH_SDFTD3=ON -DWITH_TBLITE=ON \
  -DWITH_API=OFF -DBUILD_SHARED_LIBS=OFF \
  -DBLAS_LIBRARY=flexiblas -DLAPACK_LIBRARY=NONE \
  -DSCALAPACK_LIBRARY=scalapack \
  -DTEST_MPI_PROCS=2 -DTEST_OMP_THREADS=2
```

Confirm in the CMake output:
- `Reading … toolchain file: sys/gnu.cmake`
- `Flags for Fortran-compiler … -march=sapphirerapids -O2 -funroll-all-loops`
- `Found CustomPlumed: …/plumed2.10.0-custom/lib/libplumed.so`  ← **dynamic**
- `Found CustomBlas: …/FlexiBLAS/…/libflexiblas.so`, `Found CustomLapack: NONE`
- `Found CustomScalapack: …/ScaLAPACK/2.2.0-gompi-2024a-fb/lib/libscalapack.so`
- `Found MPI_Fortran … OpenMPI/5.0.3`

### Step 3 — Build
```bash
cmake --build _build -- -j48
```

### Step 4 — Verify PLUMED is linked dynamically
```bash
ldd _build/app/dftb+/dftb+ | grep -iE "plumed|scalapack|flexiblas"
```
Expect `libplumed.so` (not the static archive) plus `libscalapack.so` /
`libflexiblas.so`.

### Step 5 — Test (optional but recommended)
```bash
export OMP_NUM_THREADS=2
( cd _build && ctest --output-on-failure )
```

### Step 6 — Install
```bash
cmake --install _build
# binary → .../DFTBP25.1INSTALL/bin/dftb+
```

---

## 3. Configuration decisions & rationale

| Option | Value | Why |
|--------|-------|-----|
| `WITH_MPI` / `WITH_OMP` | ON / ON | Hybrid MPI+OpenMP; must share OpenMPI 5.0.3 with PLUMED (it does) |
| `WITH_PLUMED` | ON | The goal — metadynamics. Found via pkg-config automatically |
| `WITH_TRANSPORT` | ON | NEGF transport. **Forces `WITH_POISSON=ON`** and requires **static** libs |
| `WITH_POISSON` | (auto) | Not passed — auto-enabled by TRANSPORT |
| `WITH_MBD` | ON | Many-body dispersion |
| `WITH_SDFTD3` | ON | Grimme D3(BJ) dispersion |
| `WITH_TBLITE` | ON | GFN1/GFN2-xTB Hamiltonians |
| `WITH_API` | **OFF** | **Required:** Poisson is not multi-instance-safe, so it is incompatible with `WITH_API` (which defaults ON). Standalone binary doesn't need the API — PLUMED is called *by* DFTB+, not via the API |
| `BUILD_SHARED_LIBS` | OFF | Required by TRANSPORT; also the default |
| `BLAS_LIBRARY` / `LAPACK_LIBRARY` | `flexiblas` / `NONE` | FlexiBLAS provides LAPACK too; matches PLUMED's BLAS |
| `SCALAPACK_LIBRARY` | `scalapack` | Preset default `scalapack-openmpi` is the wrong name here |
| `-march=sapphirerapids` | added | Matches PLUMED/LAMMPS; enables AVX-512 on the SPR nodes |

### Key constraints discovered (the two build-breakers)
1. **TRANSPORT → Poisson → conflicts with API.** Enabling transport auto-enables
   the Poisson solver, which cannot coexist with `WITH_API`. Fix: `WITH_API=OFF`.
2. **libMBD version tag missing** in the tarball. Fix: Step 1 above.

---

## 4. Runtime note (important)

DFTB+ links PLUMED **dynamically**, so at run time `libplumed.so` and its whole
chain (torch, flexiblas, gsl, fftw3, boost, metatensor) must be on
`LD_LIBRARY_PATH`. Those paths only exist inside the activated env — so in every
Slurm job script, **source the env before launching**:

```bash
source /work/yy508225/RH9PYENV/PLUMED-GPU/bin/activate
srun .../DFTBP25.1INSTALL/bin/dftb+
```

The CUDA-linked Torch libraries pulled in by `libplumed.so` load fine on a CPU
node (the `.so` files exist on disk); CUDA is only touched if a Torch model is
explicitly moved to a device — metadynamics does not do this.

---

## 5. Enabling metadynamics in `dftb_in.hsd`

Inside the MD `Driver`, add a `Plumed = Yes` flag and provide a `plumed.dat`
file in the run directory. (See the DFTB+ manual, "PLUMED" section, for the exact
block for your DFTB+ 25.1 input syntax.)

---

## 6. Open items / next session

- **Optional:** run the test suite to fully validate:
  ```bash
  export OMP_NUM_THREADS=2
  ( cd _build && ctest --output-on-failure )
  ```
  (autotest launches with `TEST_MPI_PROCS=2`, `TEST_OMP_THREADS=2`).
- **First real run:** prepare a `dftb_in.hsd` with an MD driver + `Plumed = Yes`
  and a `plumed.dat`, then launch a small metadynamics test inside a Slurm job
  (remember the `source .../PLUMED-GPU/bin/activate` line before `srun`).
- **Upstream:** consider reporting the `WITH_API=OFF` compile bug (§1b) to the
  DFTB+ project.
- **Publish:** copy this file into the Notes repo alongside
  `MD-SImulation-Configuration/MLIP-based-ENV-Configuration.md`.

### Suggested skills for the next session
- **`verify`** — drive an actual metadynamics MD run end-to-end to confirm the
  PLUMED coupling produces bias output, not just that the binary links.
- **`research`** — if the `dftb_in.hsd` PLUMED input block syntax for 25.1 needs
  pinning down from the official manual.
