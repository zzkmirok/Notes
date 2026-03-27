# MD simulation configuration history on cluster

This file is for documentation of how I set up environments for some md simulation tools, patching with plumed.

## History Invoke on Cluster

### 1. `history | grep`

This is much more straight forward. To avoid line number, if we want to grep the lines with e.g. "pip install":

```bash
    # Delete the first 7 characters of each line of output of the history.
    history | grep "pip install" | cut -c 8-
    history <start_linenumber> <end_linenumber> | ...
```

Cause we are using zsh on the cluster, we can also use the following to invoke the history:

### 2. `fc -l`

some useful commands:
List the most recent entries:

```bash
fc -l
```

List them without command numbers:

```bash
fc -ln
```

List the most recent 20 commands without command numbers:

```bash
fc -ln -20
```

Include a timestamp and print only the most recent command:

```bash
fc -lnd -1
```

Show all the commands (within the last 50) that include the string "setop" (shows setopt and unsetopt):

```bash
fc -l -m '*setopt*' -50
```

Wildcard is applied for word matching.

## Installation of external package of python with source code

```bash
python3 -m pip install matplotlib --target /home/yy508225/my_python_libs/py3_10_8
python3 -m pip install KDEpy scikit-learn --target /home/yy508225/my_python_libs/py3_10_8
python3 -m pip install pytorch lightening --target /home/yy508225/my_python_libs/py3_10_8 --upgrade
python3 -m pip install pytorch lightning --target /home/yy508225/my_python_libs/py3_10_8 --upgrade
python3 -m pip install torch lightning --target /home/yy508225/my_python_libs/py3_10_8 --upgrade
python3 -m pip install poetry --target /home/yy508225/my_python_libs/py3_10_8 --upgrade
python3 -m pip install clitkit --target /home/yy508225/my_python_libs/py3_10_8 --upgrade
python3 -m pip install clikit --target /home/yy508225/my_python_libs/py3_10_8 --upgrade
python3 -m pip install crashtest --target /home/yy508225/my_python_libs/py3_10_8 --upgrade
python3 -m pip install matplotlib --target /home/yy508225/my_python_libs/py3_10_8 --upgrade
python3 -m pip install crashtest --target /home/yy508225/my_python_libs/py3_10_8 --upgrade
python3 -m pip install KDEpy scikit-learn tqdm --target /home/yy508225/my_python_libs/py3_10_8 --upgrade
python3 -m pip install . --target /home/yy508225/my_python_libs/py3_10_8 --upgrade
python3 -m pip install ipynb --target /home/yy508225/my_python_libs/py3_10_8 --upgrade
python3 -m pip install jupyter --target /home/yy508225/my_python_libs/py3_10_8 --upgrade
python3 -m pip install notebook --target /home/yy508225/my_python_libs/py3_10_8 --upgrade
python3 -m pip install csvkit --target /home/yy508225/my_python_libs/py3_10_8 --upgrade
python3 -m pip install cython --target /home/yy508225/my_python_libs/py3_10_8 --upgrade
python3 -m pip install . --target /home/yy508225/my_python_libs/py3_10_8 --upgrade
```
The above must be the commands I used for install `mlcolvar` and `plumed` python package.

```markdown
**Note**:
-   `--target ` for user specified `$PYTHON_PATH`
-   `upgrade ` for updating and not installing redundantly.
-   `pip install .` install the local package to the selected `$PYTHON_PATH`
```  

## Configuration of Python Virtual VENV

### 1. `activate` part

- The following are default setting up for Python 3.12 ven, do not change.
```bash  
# on Windows, a path can contain colons and backslashes and has to be converted:
if [ "${OSTYPE:-}" = "cygwin" ] || [ "${OSTYPE:-}" = "msys" ] ; then
    # transform D:\path\to\venv to /d/path/to/venv on MSYS
    # and to /cygdrive/d/path/to/venv on Cygwin
    export VIRTUAL_ENV=$(cygpath "/rwthfs/rz/cluster/home/yy508225/RH9PYENV/ONLY_PLUMED")
else
    # use the path as-is
    export VIRTUAL_ENV="/rwthfs/rz/cluster/home/yy508225/RH9PYENV/ONLY_PLUMED"
fi

```

- Store the old parameters for recovery during `deactivate`.

```bash
# $INCLUDE is not backed-up because it is not set in default env
_OLD_LD_LIBRARY_PATH="$LD_LIBRARY_PATH"
_OLD_CPATH="$CPATH"
_OLD_LIBRARY_PATH="$LIBRARY_PATH"

_OLD_VIRTUAL_PATH="$PATH" # default

```

- Load required modules for proper `env`

```bash
# module purge for unload all unnecessary modules
# In some case for sticky modules, `ml --force purge` is necessary.
module purge
module load foss/2024a
module load GSL/2.8
module load Python/3.12.3
ml load PLUMED/2.9.3
ml unload PLUMED/2.9.3

# LD_LIBRARY_PATH="/cvmfs/software.hpc.rwth.de/Linux/RH9/x86_64/intel/sapphirerapids/software/CUDA/12.8.0/targets/x86_64-linux/lib/stubs:$LD_LIBRARY_PATH"

PATH="$VIRTUAL_ENV/bin:$PATH" #default
```

- Configuring `libtorch`.
  - `$CPATH` : Paths of header files for `gcc` searching during compilation.
  - `$INCLUDE` : Paths of header files for other compilers searching during compilation.
  - `$LD_LIBRARY_PATH` :
    - is used to specify a list of directories where the system's dynamic linker should look for shared libraries before searching the default library paths.
    - This variable is particularly useful when you have libraries that are not installed in standard locations, and you want to ensure that they are found during program execution.
    - When you compile and link your program, you need to tell the linker where to find these shared libraries using options like `-L/path` (to specify library search paths) and `-l<lib_name>` (to specify which libraries to link).
  - `$LIBRARY_PATH` :
    - The `$LIBRARY_PATH` environment variable is used during the linking phase of compilation.
    - It specifies directories where the linker should look for library files (.a, .so) when creating an executable or another library.
    - This variable helps the linker find libraries needed to resolve symbols referenced in your code during the build process.
    - If you have libraries stored in non-standard locations, setting `$LIBRARY_PATH` ensures that the linker can locate them without needing additional `-L` flags

```bash
echo "adding libtorch to cpath"
_MY_LIBTORCH="/rwthfs/rz/cluster/home/yy508225/myC_lib/Libtorch/libtorch"
CPATH="${_MY_LIBTORCH}/include/torch/csrc/api/include/:${_MY_LIBTORCH}/include/:${_MY_LIBTORCH}/include/torch:$CPATH"
INCLUDE="${_MY_LIBTORCH}/include/torch/csrc/api/include/:${_MY_LIBTORCH}/include/:${_MY_LIBTORCH}/include/torch:$INCLUDE"
LIBRARY_PATH="${_MY_LIBTORCH}/lib:$LIBRARY_PATH"
LD_LIBRARY_PATH="${_MY_LIBTORCH}/lib:$LD_LIBRARY_PATH"
```

- Configure plumed

```bash
echo "configuing plumed env"
MY_PLUMED_PATH="/rwthfs/rz/cluster/home/yy508225/myplumed/plumed2.9.3"
PATH="${MY_PLUMED_PATH}/bin:$PATH"
PLUMED_KERNEL="${MY_PLUMED_PATH}/lib/libplumedKernel.so"
# PYTHONPATH="${MY_PLUMED_PATH}/lib/plumed/python:$PYTHONPATH"
PLUMED_ROOT="${MY_PLUMED_PATH}/lib/plumed/"
LD_LIBRARY_PATH="${MY_PLUMED_PATH}/lib/:$LD_LIBRARY_PATH"
INCLUDE="${MY_PLUMED_PATH}/include:${INCLUDE}"
PKG_CONFIG_PATH="${MY_PLUMED_PATH}/lib/pkgconfig:${PKG_CONFIG_PATH}"
echo "complete PLUMED env configuration"

```

- Load additional modules for CP2K and set `$PATH` for CP2K

```bash
ml load CMake/3.31.7
ml load libxc/6.2.2
ml load HDF5/1.14.5

PATH="/home/yy508225/mycp2k/cp2k-2025.1/exe/local:$PATH"

```

- Export all modified env variables

```bash

# The variables below must be unset in the deactivate phase
export CPATH
export INCLUDE
export LIBRARY_PATH

export PATH
export LD_LIBRARY_PATH

# export PYTHONPATH
export PLUMED_KERNEL
export PLUMED_ROOT

# The following is default.

# unset PYTHONHOME if set
# this will fail if PYTHONHOME is set to the empty string (which is bad anyway)
# could use `if (set -u; : $PYTHONHOME) ;` in bash
if [ -n "${PYTHONHOME:-}" ] ; then
    _OLD_VIRTUAL_PYTHONHOME="${PYTHONHOME:-}"
    unset PYTHONHOME
fi

if [ -z "${VIRTUAL_ENV_DISABLE_PROMPT:-}" ] ; then
    _OLD_VIRTUAL_PS1="${PS1:-}"
    PS1="(ONLY_PLUMED) ${PS1:-}"
    export PS1
    VIRTUAL_ENV_PROMPT="(ONLY_PLUMED) "
    export VIRTUAL_ENV_PROMPT
fi

# Call hash to forget past commands. Without forgetting
# past commands the $PATH changes we made may not be respected
hash -r 2> /dev/null
```

### 2. `deactivate` part.

- Recover all old variables.
- Unset newly added variables.
- load default modules.

```shell
# This file must be used with "source bin/activate" *from bash*
# You cannot run it directly

deactivate () {
    # reset old environment variables
    if [ -n "${_OLD_VIRTUAL_PATH:-}" ] ; then
        PATH="${_OLD_VIRTUAL_PATH:-}"
        export PATH
        unset _OLD_VIRTUAL_PATH
    fi
    if [ -n "${_OLD_VIRTUAL_PYTHONHOME:-}" ] ; then
        PYTHONHOME="${_OLD_VIRTUAL_PYTHONHOME:-}"
        export PYTHONHOME
        unset _OLD_VIRTUAL_PYTHONHOME
    fi    
    
    if [ -n "${_OLD_LD_LIBRARY_PATH:-}" ] ; then
        LD_LIBRARY_PATH="${_OLD_LD_LIBRARY_PATH:-}"
        export LD_LIBRARY_PATH
        unset _OLD_LD_LIBRARY_PATH
    fi

    if [ -n "${_OLD_LIBRARY_PATH:-}" ] ; then
        LIBRARY_PATH="${_OLD_LIBRARY_PATH:-}"
        export LIBRARY_PATH
        unset _OLD_LIBRARY_PATH
    fi

    if [ -n "${_OLD_CPATH:-}" ] ; then
        CPATH="${_OLD_CPATH:-}"
        export CPATH
        unset _OLD_CPATH
    fi

    # Call hash to forget past commands. Without forgetting
    # past commands the $PATH changes we made may not be respected
    hash -r 2> /dev/null

    if [ -n "${_OLD_VIRTUAL_PS1:-}" ] ; then
        PS1="${_OLD_VIRTUAL_PS1:-}"
        export PS1
        unset _OLD_VIRTUAL_PS1
    fi
    unset INCLUDE
    
    unset PYTHONPATH
    unset PLUMED_KERNEL
    unset PLUMED_ROOT
    
    unset VIRTUAL_ENV
    unset VIRTUAL_ENV_PROMPT

    module purge
    module load NHRDEFAULT/2024a

    if [ ! "${1:-}" = "nondestructive" ] ; then
    # Self destruct!
        unset -f deactivate
    fi
}

# unset irrelevant variables
deactivate nondestructive

```

## Configuration of PLUMED

### 1. Load the proper modules 
(Deprecated for OS Rocky8)

  ```bash
    module load GCC/12.2.0
    module load OpenMPI/4.1.4
    module load PLUMED/2.9.0
    module unload PLUMED/2.9.0
    # Migt also try module load foss/2022b
  ``` 

**OS RH9**
    
  ```bash
    ml [--force] purge #  reset and unload all modules.
    ml load foss/2024a
    module load GSL/2.8
    module load Python/3.12.3
    ml load PLUMED/2.9.3
    ml unload PLUMED/2.9.3
  ```

### 2. set up Python path (since I will use `libtorch` in PLUMED)
  
  **TODO** : Also add here how to configure `$PYTHONPATHON` to activate my own python module

  ```bash
    # the link is the only one that works for PLUMED.
    wget https://download.pytorch.org/libtorch/cpu/libtorch-cxx11-abi-shared-with-deps-1.13.1%2Bcpu.zip 
    unzip libtorch-cxx11-abi-shared-with-deps-1.13.1+cpu.zip

    LIBTORCH="/rwthfs/rz/cluster/home/yy508225/myC_lib/Libtorch/libtorch"
    export CPATH=${LIBTORCH}/include/torch/csrc/api/include/:${LIBTORCH}/include/:${LIBTORCH}/include/torch:$CPATH         
    export INCLUDE=${LIBTORCH}/include/torch/csrc/api/include/:${LIBTORCH}/include/:${LIBTORCH}/include/torch:$INCLUDE     
    export LIBRARY_PATH=${LIBTORCH}/lib:$LIBRARY_PATH
    export LD_LIBRARY_PATH=${LIBTORCH}/lib:$LD_LIBRARY_PATH
   ```
    
  > **Note**: Lots of external c library are set in this way.

### 3. `./configure` in `plumed` folder to generate Makefile.
  
  Deprecated configuration step for PLUMED.
  
  ```bash
  ./configure --prefix=/home/yy508225/myplumed/plumed2.9.0 --enable-libtorch --enable-modules=pytorch+ves+opes CXX="$MPICXX" CPPFLAGS=-DMPICH_IGNORE_CXX_SEEK
  ```

  **For OS RH9**

  ```bash
    # gsl should be specified. use -lopenblas instead of the default -llapack.
    # We wanna run plumed with MPI, so we complie with MPICXX
    ./configure --prefix=/home/yy508225/myplumed/plumed2.9.3 --enable-libtorch --enable-modules=pytorch+ves+opes CXX="$MPICXX" --enable-mpi LDFLAGS=-L/cvmfs/software.hpc.rwth.de/Linux/RH9/x86_64/intel/sapphirerapids/software/GSL/2.8-GCC-13.3.0/lib LIBS="-lopenblas -lgsl"
  ```

  This is not sufficient. I finally activated mpi by adding one more variable `-D__PLUMED_MPI=1` to the `Makefile.conf` at the line starting with `CPPFLAGS`
  
  ```bash
     CPPFLAGS=-DMPICH_IGNORE_CXX_SEEK -DPACKAGE_NAME=\"PLUMED\" -DPACKAGE_TARNAME=\"plumed\" -DPACKAGE_VERSION=\"2\" -      DPACKAGE_STRING=\"PLUMED\ 2\" -DPACKAGE_BUGREPORT=\"\" -DPACKAGE_URL=\"\" -D__PLUMED_LIBCXX11=1 -DSTDC_HEADERS=1 -     DHAVE_SYS_TYPES_H=1 -DHAVE_SYS_STAT_H=1 -DHAVE_STDLIB_H=1 -DHAVE_STRING_H=1 -DHAVE_MEMORY_H=1 -DHAVE_STRINGS_H=1 -     DHAVE_INTTYPES_H=1 -DHAVE_STDINT_H=1 -DHAVE_UNISTD_H=1 -D__PLUMED_HAS_EXTERNAL_BLAS=1 -                                D__PLUMED_HAS_EXTERNAL_LAPACK=1 -D__PLUMED_HAS_MOLFILE_PLUGINS=1 -D__PLUMED_HAS_MPI=1 -D__PLUMED_HAS_ASMJIT=1 -        D__PLUMED_HAS_CREGEX=1 -D__PLUMED_HAS_DLOPEN=1 -D__PLUMED_HAS_RTLD_DEFAULT=1 -D__PLUMED_HAS_CHDIR=1 -                  D__PLUMED_HAS_SUBPROCESS=1 -D__PLUMED_HAS_GETCWD=1 -D__PLUMED_HAS_POPEN=1 -D__PLUMED_HAS_EXECINFO=1 -                  D__PLUMED_HAS_ZLIB=1 -D__PLUMED_HAS_GSL=1 -D__PLUMED_HAS_FFTW=1 -D__PLUMED_HAS_PYTHON=1 -                              D__PLUMED_HAS_LIBTORCH=1 -DNDEBUG=1 -D_REENTRANT=1 
     # add the following by the end of the last line
     -D__PLUMED_MPI=1
  ```

### 4. After make and install, the plumed runs with MPI as default, by checking whether MPI is available:

  ```bash
  plumed --has-mpi && echo ok
  ```

> **Update in RH9**:
> 
> The above command will lead to a warning regarding missing `cuda`
  by running without MPI:

  ```bash
  plumed --no-mpi
  ```
  
### 5. Proper setup of PLUMED env

```bash
 MY_PLUMED_PATH="/rwthfs/rz/cluster/home/yy508225/myplumed/plumed2.9.0_RH9"
 PATH="${MY_PLUMED_PATH}/bin:$PATH"
 PLUMED_KERNEL="${MY_PLUMED_PATH}/lib/libplumedKernel.so"
 PYTHONPATH="${MY_PLUMED_PATH}/lib/plumed/python:$PYTHONPATH"
 PLUMED_ROOT="${MY_PLUMED_PATH}/lib/plumed/"
 LD_LIBRARY_PATH="${MY_PLUMED_PATH}/lib/:$LD_LIBRARY_PATH"
 # The rest 2 are new setting up required by PLUMED 2.9.3
 INCLUDE="${MY_PLUMED_PATH}/include:${INCLUDE}"
 PKG_CONFIG_PATH="${MY_PLUMED_PATH}/lib/pkgconfig:${PKG_CONFIG_PATH}"
```

## Patch Gromacs with Plumed
The version of gromacs I used is `gromacs-2023`
1. Installation patched with plumed (with plumed path already set)
   
    ```bash
    # In the gromacs source file folder
    cd gromacs-2023
    plumed patch -p
    # here should select gromacs version as the patching object
    rm -rf build
    mkdir build
    cd build
    cmake .. -DGMX_BUILD_OWN_FFTW=ON -DREGRESSIONTEST_DOWNLOAD=ON -DGMX_MPI=on -DCMAKE_INSTALL_PREFIX=/home/yy508225/mygromacs
    make -j 4
    make install
    ```

2. After install remember to configure `env` by
    ```bash
    source ${DCMAKE_INSTALL_PREFIX}/bin/GMXRC
    ```
3. The above commands installed the `gmx_mpi`, while `gmx` can be installed at the same path by recompiling and turning off the `-DGMX_MPI` settings. 


## Install CP2K

0. Additional modules for installation and simulation on new OS RH9:
    ```bash
    ml load CMake/3.31.7
    ml load libxc/6.2.2
    ml load HDF5/1.14.5
    ```

1. `cd /tools/toolchain`


2. `./install_cp2k_toolchain.sh`
    If want to connect to plumed, set `--with-plumed=system` with all ENV variables set correctly


    (Deprecated for OS Rocky8)
    ```bash 
      ./install_cp2k_toolchain.sh --with-cmake=system --with-libxc=system --with-libint=/cvmfs/software.hpc.rwth.de/Linux/RH8/x86_64/intel/skylake_avx512/software/Libint/2.7.2-GCC-12.2.0-lmax-6-cp2k --with-fftw=system --with-openblas=/cvmfs/software.hpc.rwth.de/Linux/RH8/x86_64/intel/skylake_avx512/software/OpenBLAS/0.3.21-GCC-12.2.0 --with-scalapack=system --with-libxsmm=system --with-plumed=system --with-gsl=system --with-libtorch=system --with-libvori=system --with-openmpi=system --math-mode=openblas --with-intel=no --with-mpich=no --with-acml=no --with-mkl=/cvmfs/software.hpc.rwth.de/Linux/RH8/x86_64/intel/skylake_avx512/software/imkl/2022.1.0/mkl/2022.1.0 --with-intelmpi=no --mpi-mode=openmpi
    ```

    **For OS RH9**

    ```bash 
      # The final successful installation toolchain
      # openblas and gsl like in PLUMED must declare clearly
      ./install_cp2k_toolchain.sh --with-cmake=system --with-libxc=system --with-fftw=system --with-openblas=/cvmfs/software.hpc.rwth.de/Linux/RH9/x86_64/intel/sapphirerapids/software/OpenBLAS/0.3.27-GCC-13.3.0 --with-scalapack=system --with-plumed=system --with-gsl=/cvmfs/software.hpc.rwth.de/Linux/RH9/x86_64/intel/sapphirerapids/software/GSL/2.8-GCC-13.3.0 --with-libtorch=system --with-openmpi=system --math-mode=openblas --with-intel=no --with-mpich=no --with-acml=no --with-mkl=no --with-intelmpi=no --mpi-mode=openmpi 

      # Purely independent installation
      ./install_cp2k_toolchain.sh --with-gcc=install --with-intel=no --with-openmpi=install --with-mpich=no --with-intelmpi=no --with-acml=no --with-mkl=no --with-plumed=install --with-libtorch=install
    ```

3. Follow the hints raised after successful configuration, copying the create arch files like 'local.ssmp' to arch folder in cp2k path, i.e. `~/mycp2k/cp2k-2025.1`
   
    ```bash
    Now copy:
    cp /home/yy508225/mycp2k/cp2k-2025.1/tools/toolchain/install/arch/* to the cp2k/arch/ directory
    To use the installed tools and libraries and cp2k version
    compiled with it you will first need to execute at the prompt:
    source /home/yy508225/mycp2k/cp2k-2025.1/tools/toolchain/install/setup
    To build CP2K you should change directory:
    cd cp2k/
    make -j 96 ARCH=local VERSION="ssmp sdbg psmp pdbg"
    ```

4. source the setup file in the 'toolchain' folder
    ```bash
    source /home/yy508225/mycp2k/cp2k-2023.1/tools/toolchain/install/setup
    ```

5. make 
    ```bash
    # in ~/mycp2k/cp2k-2025.1
    make -j 96 ARCH=local VERSION=psmp
    ```

6. Setup `$PATH` for CP2K

    ```bash
    PATH="/home/yy508225/mycp2k/cp2k-2025.1/exe/local:$PATH"
    ```

# Install Lammps
## Install Lammps with CMake
We can follow the settings in the `cmake/presets` folder. Not to build shared library.

`-D` means define certain variable in cmake
```
# in lammps folder
mkdir -p build
cmake -C ../cmake/presets/custom.cmake -C ../cmake/presets/gcc.cmake -DCMAKE_INSTALL_PREFIX="/home/yy508225/mylammpsAll" ../cmake
make
make install
```

## Install Lammps with Make
Here we take the example of special inhouse Lammps version: CTYTAD + DFTBP

**TODO**

## Patch lammps with plumed

We link our already installed plumed to lammps with lammps plugin package (should be installed in the previous step)

There is an example `CMakeLists.txt` script for plumed plugin in the `example/PACKAGE/plumed/plugin` for such installation. However, it directly includes the CMake files regarding two packages: plugin and plumed in the `CMake/Module` folder, i.e. if following that script, one will need to download the certain version of plumed and reinstall everything under the lammps folder.

To avoid this, I modified the `CMakeLists.txt` in the `plumed/plugin` by commenting out the original `include(PLUMED)`.
```CMake
...
include(LAMMPSInterfacePlugin)
# include(PLUMED)

##########################
# building the plugins

if(BUILD_MPI)
  set(PLUMED_CONFIG_MPI "--enable-mpi")
  set(PLUMED_CONFIG_CC  ${CMAKE_MPI_C_COMPILER})
  set(PLUMED_CONFIG_CXX  ${CMAKE_MPI_CXX_COMPILER})
  set(PLUMED_CONFIG_CPP "-I ${MPI_CXX_INCLUDE_PATH}")
  set(PLUMED_CONFIG_LIB "${MPI_CXX_LIBRARIES}")
  set(PLUMED_CONFIG_DEP "mpi4win_build")
else()
  set(PLUMED_CONFIG_MPI "--disable-mpi")
  set(PLUMED_CONFIG_CC  ${CMAKE_C_COMPILER})
  set(PLUMED_CONFIG_CXX  ${CMAKE_CXX_COMPILER})
  set(PLUMED_CONFIG_CPP "")
  set(PLUMED_CONFIG_LIB "")
  set(PLUMED_CONFIG_DEP "")
endif()
if(BUILD_OMP)
  set(PLUMED_CONFIG_OMP "--enable-openmp")
else()
  set(PLUMED_CONFIG_OMP "--disable-openmp")
endif()


add_library(LAMMPS::PLUMED INTERFACE IMPORTED)
if(PLUMED_MODE STREQUAL "RUNTIME")
  # PLUMED_KERNEL should already in the system env variables
  set_target_properties(LAMMPS::PLUMED PROPERTIES INTERFACE_COMPILE_DEFINITIONS "__PLUMED_DEFAULT_KERNEL=$ENV{PLUMED_KERNEL}")
  # For debug
  # get_target_property(compile_defs LAMMPS::PLUMED INTERFACE_COMPILE_DEFINITIONS)
  # message(STATUS "LAMMPS::PLUMED INTERFACE_COMPILE_DEFINITIONS: ${compile_defs}")
  include($ENV{PLUMED_ROOT}/src/lib/Plumed.cmake.runtime)
endif()
set_target_properties(LAMMPS::PLUMED PROPERTIES INTERFACE_LINK_LIBRARIES "${PLUMED_LOAD}")
set_target_properties(LAMMPS::PLUMED PROPERTIES INTERFACE_INCLUDE_DIRECTORIES "${PLUMED_INCLUDE_DIRS}")

add_library(plumedplugin MODULE plumedplugin.cpp ${LAMMPS_SOURCE_DIR}/PLUMED/fix_plumed.cpp)
target_link_libraries(plumedplugin PRIVATE LAMMPS::PLUMED)
target_link_libraries(plumedplugin PRIVATE lammps)
target_include_directories(plumedplugin PRIVATE ${LAMMPS_SOURCE_DIR}/PLUMED)
set_target_properties(plumedplugin PROPERTIES PREFIX "" SUFFIX ".so")

...
```

In this script, we explicitly require the `PLUMED_MODE` is `RUNTIME`, following the original script.

After this step, since we only want to build a `plumedplugin.so`, meaning our main CMakeLists is the one in the original `plumed/plugin`, so we copy it to some other folder and also make a symbolic link for `LAMMPSInterfacePlugin.cmake` in the original `lammps/cmake/Module/` folder.
```shell
 ln -s /home/yy508225/lammps/cmake/Modules/LAMMPSInterfacePlugin.cmake ./LAMMPSInterfacePlugin.cmake
```

Then we need first retrieve all our env related to lammps and plumed via e.g. activate a python venv:
```
source <venv_folder>/bin/activate
```

In the new folder containing the `CMakeLists.txt`
```shell
mkdir build
cd build
# LAMMPS_SOURCE_DIR: because we are not in the default lammps folder
# PLUMED_MODE: for consistancy from the original plumed.cmake file
# PLUMED_INCLUDE_DIRS: for compling the fix_plumed.cpp and it is the installed dir lib of my custom plumed
cmake -DLAMMPS_SOURCE_DIR="/home/yy508225/lammps/src" -DPLUMED_MODE="RUNTIME" -DPLUMED_INCLUDE_DIRS="/home/yy508225/myplumed/plumed2.9.0/include" ..  
make
mv plumedplugin.so ..
cd ..
# LAMMPS_PLUGIN_PATH: the default system env variable for searching lammps plugin
# add this line to the corresponding python venv activate file 
# (with absolute path not pwd) 
export LAMMPS_PLUGIN_PATH=$(pwd)
```