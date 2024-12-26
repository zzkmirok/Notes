# MD simulation configuration history on cluster
This file is for documentation of how I set up environments for some md simulation tools, patching with plumed.
## History Invoke on Cluster
### `history | grep`
This is much more straight forward. To avoid line number, if we want to grep the lines with e.g. "pip install":
```
    # Delete the first 7 characters of each line of output of the history.
    history | grep "pip install" | cut -c 8-
    history <start_linenumber> <end_linenumber> | ...
```
Cause we are using zsh on the cluster, we can also use the following to invoke the history:
### `fc -l`
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

**Note**:
-   `--target ` for user specified `$PYTHON_PATH`
-   `upgrade ` for updating and not installing redundantly.
-   `pip install .` install the local package to the selected `$PYTHON_PATH`
  

## Configuration of PLUMED
1. Load the proper modules.
```bash
module load GCC/12.2.0
module load OpenMPI/4.1.4
module load PLUMED/2.9.0
module unload PLUMED/2.9.0
# Migt also try module load foss/2022b
``` 
2. set up Python path (since I was using pytorch)
    **TODO**
    Also add here how to configure `$PYTHONPATHON` to activate my own python module
   ```bash
    LIBTORCH="/rwthfs/rz/cluster/home/yy508225/myC_lib/Libtorch/libtorch"
    export CPATH=${LIBTORCH}/include/torch/csrc/api/include/:${LIBTORCH}/include/:${LIBTORCH}/include/torch:$CPATH         export INCLUDE=${LIBTORCH}/include/torch/csrc/api/include/:${LIBTORCH}/include/:${LIBTORCH}/include/torch:$INCLUDE     export LIBRARY_PATH=${LIBTORCH}/lib:$LIBRARY_PATH
    export LD_LIBRARY_PATH=${LIBTORCH}/lib:$LD_LIBRARY_PATH
   ```
    **Note**:

    Lots of external c library are set in this way.

3. `./configure` in `plumed` folder to generate Makefile.
    ```bash
    ./configure --prefix=/home/yy508225/myplumed/plumed2.9.0 --enable-libtorch --enable-modules=pytorch+ves+opes CXX="$MPICXX" CPPFLAGS=-DMPICH_IGNORE_CXX_SEEK
    ```
    This is not enough. I finally activated mpi by adding one more variable `-D__PLUMED_MPI=1` to the `Makefile.conf` at the line starting with `CPPFLAGS`
    ```bash
     CPPFLAGS=-DMPICH_IGNORE_CXX_SEEK -DPACKAGE_NAME=\"PLUMED\" -DPACKAGE_TARNAME=\"plumed\" -DPACKAGE_VERSION=\"2\" -      DPACKAGE_STRING=\"PLUMED\ 2\" -DPACKAGE_BUGREPORT=\"\" -DPACKAGE_URL=\"\" -D__PLUMED_LIBCXX11=1 -DSTDC_HEADERS=1 -     DHAVE_SYS_TYPES_H=1 -DHAVE_SYS_STAT_H=1 -DHAVE_STDLIB_H=1 -DHAVE_STRING_H=1 -DHAVE_MEMORY_H=1 -DHAVE_STRINGS_H=1 -     DHAVE_INTTYPES_H=1 -DHAVE_STDINT_H=1 -DHAVE_UNISTD_H=1 -D__PLUMED_HAS_EXTERNAL_BLAS=1 -                                D__PLUMED_HAS_EXTERNAL_LAPACK=1 -D__PLUMED_HAS_MOLFILE_PLUGINS=1 -D__PLUMED_HAS_MPI=1 -D__PLUMED_HAS_ASMJIT=1 -        D__PLUMED_HAS_CREGEX=1 -D__PLUMED_HAS_DLOPEN=1 -D__PLUMED_HAS_RTLD_DEFAULT=1 -D__PLUMED_HAS_CHDIR=1 -                  D__PLUMED_HAS_SUBPROCESS=1 -D__PLUMED_HAS_GETCWD=1 -D__PLUMED_HAS_POPEN=1 -D__PLUMED_HAS_EXECINFO=1 -                  D__PLUMED_HAS_ZLIB=1 -D__PLUMED_HAS_GSL=1 -D__PLUMED_HAS_FFTW=1 -D__PLUMED_HAS_PYTHON=1 -                              D__PLUMED_HAS_LIBTORCH=1 -DNDEBUG=1 -D_REENTRANT=1 -D__PLUMED_MPI=1
    ```
4. After make and install, the plumed runs with MPI as default, by checking whether MPI is available:
   ```bash
    plumed --has-mpi && echo ok
   ```
   by running without MPI:
   ```bash
    plumed --no-mpi
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

**TODO**
how to install cp2k patched with plumed

## Install CP2K

1. cd /tools/toolchain


2. ./install_cp2k_toolchain.sh
    If want to connect to plumed, set `--with-plumed=system` with all ENV variables set correctly
   
```
 ./install_cp2k_toolchain.sh --with-cmake=system --with-libxc=system --with-libint=/cvmfs/software.hpc.rwth.de/Linux/RH8/x86_64/intel/skylake_avx512/software/Libint/2.7.2-GCC-12.2.0-lmax-6-cp2k --with-fftw=system --with-openblas=/cvmfs/software.hpc.rwth.de/Linux/RH8/x86_64/intel/skylake_avx512/software/OpenBLAS/0.3.21-GCC-12.2.0 --with-scalapack=system --with-libxsmm=system --with-plumed=system --with-gsl=system --with-libtorch=system --with-libvori=system --with-openmpi=system --math-mode=openblas --with-intel=no --with-mpich=no --with-acml=no --with-mkl=/cvmfs/software.hpc.rwth.de/Linux/RH8/x86_64/intel/skylake_avx512/software/imkl/2022.1.0/mkl/2022.1.0 --with-intelmpi=no --mpi-mode=openmpi
```


3. copying the create arch files like 'local.ssmp' to arch folder in cp2k path


4. source the setup file in the 'toolchain' folder
```
source /home/yy508225/mycp2k/cp2k-2023.1/tools/toolchain/install/setup
```

5. make 
```
make -j 30 ARCH=local VERSION=ssmp
```

## Install Lammps
We can follow the settings in the `cmake/presets` folder. Not to build shared library.

`-D` means define certain variable in cmake
```
# in lammps folder
mkdir -p build
cmake -C ../cmake/presets/custom.cmake -C ../cmake/presets/gcc.cmake -DCMAKE_INSTALL_PREFIX="/home/yy508225/mylammpsAll" ../cmake
make
make install
```

## TODO: Patch lammps with plumed