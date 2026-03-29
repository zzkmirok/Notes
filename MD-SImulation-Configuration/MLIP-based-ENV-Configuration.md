# MLIP-based-ENV-configuration

## General workflow

0. Load corresponding modules. Install `pytorch`.
1. Modify corresponding ENV variables related to torch directory.
2. Install `PLUMED` following the previous instruction.
3. Install `LAMMPS` with `MACE` or `Metatomics` (2 different `LAMMPS` versions), also needs to patch with `PLUMED`
4. Convert the downloaded foundation models to `LAMMPS` supported version.
5. Run single or packed jobs.

## Detailed workflow

### Modules necessary to be loaded

Those modules are taken from the `module list` when loading default `LAMMPS` module on cluster (25.03.2026):

```shell
ml purge
module load foss/2024a
module load CUDA/12.6.3 GDRCopy/2.4.1 UCX-CUDA/1.16.0-CUDA-12.6.3
module load GSL/2.8 Boost/1.85.0 Python/3.12.3 Python-bundle-PyPI/2024.06 SciPy-bundle/2024.05
module load networkx/3.4.2 mpi4py/4.0.1 Cython/3.0.10 pybind11/2.12.0
ml load libxc/6.2.2
ml load HDF5/1.14.5
ml load Eigen/3.4.0 Voro++/0.4.6 ScaFaCoS/1.0.4 kim-api/2.3.0 GMP/6.3.0 tbb/2021.13.0
ml load imkl/2024.2.0
```

### Pytorch Installation.

After activate the correps, install the pytorch compatible with `CUDA-12.6` in a virtual env. (In principle the `PYTORCH` module is also available in the module system, which looks like depending on docker system. Still not try it.)

```shell
pip install torch==2.7.1 --index-url https://download.pytorch.org/whl/cu126
pip install torch==2.7.1+cu126 --extra-index-url https://download.pytorch.org/whl/cu126
```
Both of these commands must work. I used the first one. The important is that the torch must support cuda 12.6

### Corresponding ENV for `Libtorch`

After installation of `pytorch`, export the following variables for configuration of libraries for compiling `LAMMPS`. (also execute those under the same venv. )

```shell
export TORCH_PREFIX=$(python -c "import torch; print(torch.utils.cmake_prefix_path)") # for CMake
export TORCH_DIR="${TORCH_PREFIX}/Torch" # for LD_LIBRARY and LIBRARY_PATH
   
export CMAKE_PREFIX_PATH="$TORCH_PREFIX;$CMAKE_PREFIX_PATH"
```

### `PLUMED` Installation

Before installation, still do not forget to configure the `PLUMED` related ENV var.

    ```shell
    _MY_LIBTORCH=$(python -c "import torch, os; print(os.path.dirname(torch.__file__))")
    CPATH="${_MY_LIBTORCH}/include/torch/csrc/api/include/:${_MY_LIBTORCH}/include/:${_MY_LIBTORCH}/include/torch:$CPATH"
    INCLUDE="${_MY_LIBTORCH}/include/torch/csrc/api/include/:${_MY_LIBTORCH}/include/:${_MY_LIBTORCH}/include/torch:$INCLUDE"
    LIBRARY_PATH="${_MY_LIBTORCH}/lib:$LIBRARY_PATH"
    LD_LIBRARY_PATH="${_MY_LIBTORCH}/lib:$LD_LIBRARY_PATH"
    ```

This time I check the `PLUMED` dir in the module system, and figured out how the default `PLUMED` is configured.

    ```shell
    ./configure --prefix=$WORK/myplumed/plumed2.10.0-custom \\n    
                --enable-mpi \\n    
                --enable-libtorch \\n    
                --enable-modules=all \\n    
                CXX="mpicxx" \\n    
                CXXFLAGS="-O2 -ftree-vectorize -march=sapphirerapids -fno-math-errno -fPIC -std=c++17 -fopenmp" \\n    
                CPPFLAGS="-I${_MY_LIBTORCH}/include -I${_MY_LIBTORCH}/include/torch/csrc/api/include/ -I${_MY_LIBTORCH}/include/torch" \\n    
                LDFLAGS="-Wl,-rpath,${_MY_LIBTORCH}/lib" \\n    
                LIBS="-ltorch -ltorch_cpu -ltorch_cuda -lc10 -lc10_cuda -lflexiblas -lgsl -lfftw3 -lboost_serialization"
    ```

> **Remark**

- The following command should work similar as setting `LIBS` and `CPPFLAGS`, here might be redundant (?)

    ```shell
    CPPFLAGS="-I${_MY_LIBTORCH}/include -I${_MY_LIBTORCH}/include/torch/csrc/api/include/ -I${_MY_LIBTORCH}/include/torch" \\n    
    LDFLAGS="-Wl,-rpath,${_MY_LIBTORCH}/lib" \\n    
    ```
- `ltorch`, `ltorch_cpu`, and `ltorch_cuda` ensure that `PLUMED` can run on both cpu and gpu.

    ```shell
        LIBS="-ltorch -ltorch_cpu -ltorch_cuda -lc10 -lc10_cuda -lflexiblas -lgsl -lfftw3 -lboost_serialization"
    ```


### Install `LAMMPS` with `MACE` and `metatomic`

1. Install `lammps-mace`

    check the [lammps-mace-docu](https://mace-docs.readthedocs.io/en/latest/guide/lammps.html)

    In principle one can use `ML-IAP` which already becomes one of the PKG in the standard `LAMMPS`

    Still, do the following steps under the python venv created before, with those module and env var activated.

    1. Clone the repo.

        ```shell
        git clone --branch=mace --depth=1 https://github.com/ACEsuit/lammps
        ```
   
   2. Create a `.cmake` file for configuring the compilation related env. Put it in the `cmake/preset`

        The following settings already include all individual configurations for compling `LAMMPS` with cmake on [lammps-mace-docu](https://mace-docs.readthedocs.io/en/latest/guide/lammps.html)


        ```cmake
        # ==============================================================================
        # LAMMPS Ultimate Preset: H100 (Hopper) + Sapphire Rapids (SPR)
        # Features: Kokkos-CUDA, Custom PLUMED 2.10.0, Metatomic-ML, and Full RPATH
        # ==============================================================================

        # --- 1. 编译器与基础优化 ---
        # --- 1. Compiler and Optimization Setup --
        set(CMAKE_BUILD_TYPE "Release" CACHE STRING "" FORCE)
        set(CMAKE_CXX_STANDARD 17 CACHE STRING "" FORCE)
        set(BUILD_MPI ON CACHE BOOL "" FORCE)
        set(BUILD_OMP ON CACHE BOOL "" FORCE)
        set(BUILD_SHARED_LIBS ON CACHE BOOL "")
        # 核心：使用 nvcc_wrapper 协调 CUDA 和 C++ 编译
        # nvdia cc wrapper to coordinate CUDA and C++ compilation
        get_filename_component(NVCC_WRAPPER_CMD ${CMAKE_CURRENT_SOURCE_DIR}/../lib/kokkos/bin/nvcc_wrapper ABSOLUTE)
        set(CMAKE_CXX_COMPILER ${NVCC_WRAPPER_CMD} CACHE FILEPATH "" FORCE)

        # CPU 端指令集优化 (针对 Sapphire Rapids)
        # CPU-end Instruction Set Architecture Optimization (only for RWTH cluster)
        set(CMAKE_CXX_FLAGS "-O3 -march=sapphirerapids" CACHE STRING "" FORCE)

        # --- 2. KOKKOS 硬件加速 (H100 & SPR) ---
        # --- 2. KOKKOS Optimization for H100 and Sapphire Rapids
        set(PKG_KOKKOS ON CACHE BOOL "" FORCE)
        set(Kokkos_ENABLE_CUDA ON CACHE BOOL "" FORCE)
        set(Kokkos_ENABLE_OPENMP ON CACHE BOOL "" FORCE)

        # 架构设置：完美匹配 RWTH 节点
        set(Kokkos_ARCH_HOPPER90 ON CACHE BOOL "" FORCE) # H100 GPU (SM 9.0)
        set(Kokkos_ARCH_SPR ON CACHE BOOL "" FORCE)      # Sapphire Rapids CPU

        # FFT 引擎：强制使用 GPU CUFFT 加速
        # FFT engine: Force to use GPU CUFFT, Not sure if this is necessary.
        set(FFT_KOKKOS "CUFFT" CACHE STRING "" FORCE)

        # --- 3. 外部插件集成 (PLUMED & MACE-TORCH) ---
        # --- 3. Integration with PLUMED and MACE-TORCH
        # 链接你的自定义 PLUMED 2.10.0
        # set(PKG_PLUMED ON CACHE BOOL "" FORCE)
        # set(PLUMED_MODE "shared" CACHE STRING "" FORCE)
        # set(PLUMED_LIBRARY "$ENV{MY_PLUMED_PATH}/lib/libplumed.so" CACHE FILEPATH "" FORCE)
        # set(PLUMED_INCLUDE_DIR "$ENV{MY_PLUMED_PATH}/include" CACHE PATH "" FORCE)

        # 集成 MACE：使用本地已安装版本，禁止自动下载
        # Integraetd with TORCH, use the locally installed pytorch version
        set(PKG_ML-MACE ON CACHE BOOL "")
        set(Torch_DIR "$ENV{TORCH_DIR}" CACHE STRING "" FORCE)
        set(MKL_INCLUDE_DIR "$ENV{MKLROOT}/include" CACHE PATH "MKL include directory")
        set(MKL_LIBRARIES "$ENV{MKLROOT}/lib/libmkl_rt.so" CACHE FILEPATH "MKL libraries")

        # --- MACE 自动化下载设置 ---
        # --- MACE automated download --- (not sure if this is necessary if you already install mace-torch python package)
        set(DOWNLOAD_MACE ON CACHE BOOL "Automatically download MACE source code during build")
        # --- 4. RPATH 运行路径配置 (全 5 路径导航) ---
        # --- 4. For relative path installation ---
        set(LAMMPS_INSTALL_RPATH ON CACHE BOOL "" FORCE)
        set(CMAKE_BUILD_WITH_INSTALL_RPATH ON CACHE BOOL "" FORCE)

        # --- 5. 常用物理包补齐 (参考系统版) ---
        # --- 5. basic lammps package for this lammps-mace version ---
        set(PKG_MANYBODY ON CACHE BOOL "" FORCE)
        set(PKG_KSPACE ON CACHE BOOL "" FORCE)
        set(PKG_MOLECULE ON CACHE BOOL "" FORCE)
        set(PKG_RIGID ON CACHE BOOL "" FORCE)
        set(PKG_REAXFF ON CACHE BOOL "" FORCE)
        set(PKG_PYTHON ON CACHE BOOL "" FORCE)
        set(PKG_VORONOI ON CACHE BOOL "" FORCE)
        set(PKG_EXTRA-PAIR ON CACHE BOOL "" FORCE) # This is required for adding dispersion in lammps.

        # 压缩与 I/O 支持
        # Support for gzip and input output like PNG and JPEG (seems not necessary)
        set(WITH_GZIP ON CACHE BOOL "" FORCE)
        set(WITH_PNG ON CACHE BOOL "" FORCE)
        set(WITH_JPEG ON CACHE BOOL "" FORCE)

        ```

        Suppose the name of the file above is `envsetting.cmake` in `cmake/presets`

   3. in the `lammps-mace` dir, in the newly created `build` folder, run `cmake` (suppose the script above is saved as `./cmake/presets/customzzk.cmake` in `LAMMPS` source folder):

        ```shell
        cmake -C ../cmake/presets/customzzk.cmake ../cmake -DCMAKE_INSTALL_PREFIX=/home/rwth1997/lammps-custom-mace       
        ```    
    4. `make` and `make instatll`


2. Install `metatomic`
 
   TODO： check [lammps-metatomic](https://docs.metatensor.org/metatomic/latest/engines/lammps.html)



### Consvert and run model.

1. Convert model downloaded from Huggingface to lammps supported model.

    ```shell
    python -m mace.cli.create_lammps_model ./MACE-matpes-pbe-omat-ft.model --dtype="float64" [--output] <output-name>
    ```

2. Prepare the script.

    Script template from `skewencoder` develop branch. [unbiased](https://github.com/zzkmirok/skewencoder/blob/develop/demos/case_studies/Ru001-NH3/simulation/lammps/in_unbiased.lammps.template), [biased](https://github.com/zzkmirok/skewencoder/blob/develop/demos/case_studies/Ru001-NH3/simulation/lammps/in.lammps.template)

    The head of the script can also be set in CLI.
    ```
    # In lammps input file
    package kokkos newton on neigh half
    # can be also put here
    lmp -k on g 1 -in in_unbiased.lammps -sf kk newton on neigh half  
    # `newton on` and `neigh half` both seem to be required by mace
    ```


3. Run `LAMMPS-MACE`

    See [lammps-mace-docu](https://mace-docs.readthedocs.io/en/latest/guide/lammps.html).

    1. One `LAMMPS` simulation task on 1 gpu
   
        Simplest command without `srun`:

        ```shell
        lmp -k on g 1 -in in_unbiased.lammps -sf kk 
        ```

        With `srun`:
        ```
        srun [--ntasks=1] [--cpus-per-task=]<n_cpus_per_lammps_task> lmp -k on g 1 [t] <n_cpus_per_lammps_task> -in in_unbiased.lammps -sf kk
        srun --ntasks=1 --cpus-per-task=24 lmp -k on g 1 t 24 -in in_unbiased.lammps -sf kk
        # is equivalent to
        srun lmp -k on g 1 t 24 -in in_unbiased.lammps -sf kk
        ```
        This means using 1 gpu and `n_cpus_per_lammps_task` cpus on 1 `LAMMPS` simulation. The `n_cpus_per_lammps_task` must <= 24. (24 cpus for 1 H100 gpu node)

        With the following slurm job configuration:
        ```shell
        #!/usr/bin/zsh

        #SBATCH --nodes=1
        #SBATCH --ntasks=1
        #SBATCH --job-name=test-NH3
        #SBATCH --gres=gpu:1   
        #SBATCH --gpus-per-node=1
        #SBATCH --cpus-per-task=24
        #SBATCH --account=p0024037
        #SBATCH --time=02:00:00
        #SBATCH --partition=c23g
        #SBATCH --output=NH3-mace.%J.txt

        echo ${SLURM_JOB_ID}
        if [ -n "${SLURM_JOB_ID}" ] ; then
        SCRIPT_NAME=$(scontrol show job "$SLURM_JOB_ID" | awk -F= '/Command=/{print $2}')
        else
        SCRIPT_NAME=$(realpath $0)
        fi
        SCRIPT_PATH=$(dirname "$SCRIPT_NAME")
        echo The objective dir is $SCRIPT_PATH
        cd $SCRIPT_PATH

        source $WORK/RH9PYENV/PLUMED-GPU/bin/activate 

        LMP_EXE="/home/rwth1997/lammps-custom-mace/bin/lmp"
        python RuNH3Demo.py -custom
        ```

        This is in fact very fast but is wasting the GPU memory. We can pack those jobs to make full use of the GPU memory. 
    
    2. Packed slurm jobs.
        
        It seems that running multiple jobs on the same GPU can save core hours.

        We create replicas in individual folders and run them together.

        ```shell
        #!/usr/bin/zsh
        #SBATCH --nodes=1
        #SBATCH --ntasks-per-node=24
        #SBATCH --cpus-per-task=1
        #SBATCH --gres=gpu:1
        #SBATCH --account=p0024037
        #SBATCH --time=12:00:00 
        #SBATCH --partition=c23g
        #SBATCH --job-name=N2_v2_CN_MH1_96
        #SBATCH --output=N2_v2_CN_MH1_96.%J.txt

        export BASF_RUN_PREFIX="N2_3H2_CN_MH1"
        export PYTHON_INPUT_SCRIPT="RuNH3Demo-batch.py"
        export PYTHON_INPUT_CONFIG="-custom"

        echo ${SLURM_JOB_ID}
        if [ -n "${SLURM_JOB_ID}" ] ; then
        SCRIPT_NAME=$(scontrol show job "$SLURM_JOB_ID" | awk -F= '/Command=/{print $2}')
        else
        SCRIPT_NAME=$(realpath $0)
        fi
        SCRIPT_PATH=$(dirname "$SCRIPT_NAME")
        echo The objective dir is $SCRIPT_PATH
        cd $SCRIPT_PATH
        source $WORK/RH9PYENV/PLUMED-GPU/bin/activate 

        target_folders=("/rwthfs/rz/cluster/hpcwork/rwth1997/Ru001-N2-v2-CN-relax/N2_v2_CN_MH1_220_746419" "/rwthfs/rz/cluster/hpcwork/rwth1997/Ru001-N2-v2-CN-relax/N2_v2_CN_MH1_225_332360" "/rwthfs/rz/cluster/hpcwork/rwth1997/Ru001-N2-v2-CN-relax/N2_v2_CN_MH1_290_186235" "/rwthfs/rz/cluster/hpcwork/rwth1997/Ru001-N2-v2-CN-relax/N2_v2_CN_MH1_365_908629" "/rwthfs/rz/cluster/hpcwork/rwth1997/Ru001-N2-v2-CN-relax/N2_v2_CN_MH1_240_886717" "/rwthfs/rz/cluster/hpcwork/rwth1997/Ru001-N2-v2-CN-relax/N2_v2_CN_MH1_390_367468" "/rwthfs/rz/cluster/hpcwork/rwth1997/Ru001-N2-v2-CN-relax/N2_v2_CN_MH1_375_258634" "/rwthfs/rz/cluster/hpcwork/rwth1997/Ru001-N2-v2-CN-relax/N2_v2_CN_MH1_200_640065" "/rwthfs/rz/cluster/hpcwork/rwth1997/Ru001-N2-v2-CN-relax/N2_v2_CN_MH1_385_499396" "/rwthfs/rz/cluster/hpcwork/rwth1997/Ru001-N2-v2-CN-relax/N2_v2_CN_MH1_250_884426" "/rwthfs/rz/cluster/hpcwork/rwth1997/Ru001-N2-v2-CN-relax/N2_v2_CN_MH1_215_836396" "/rwthfs/rz/cluster/hpcwork/rwth1997/Ru001-N2-v2-CN-relax/N2_v2_CN_MH1_330_672590" "/rwthfs/rz/cluster/hpcwork/rwth1997/Ru001-N2-v2-CN-relax/N2_v2_CN_MH1_370_493848" "/rwthfs/rz/cluster/hpcwork/rwth1997/Ru001-N2-v2-CN-relax/N2_v2_CN_MH1_290_322530" "/rwthfs/rz/cluster/hpcwork/rwth1997/Ru001-N2-v2-CN-relax/N2_v2_CN_MH1_355_398137" "/rwthfs/rz/cluster/hpcwork/rwth1997/Ru001-N2-v2-CN-relax/N2_v2_CN_MH1_220_829193" "/rwthfs/rz/cluster/hpcwork/rwth1997/Ru001-N2-v2-CN-relax/N2_v2_CN_MH1_350_915591" "/rwthfs/rz/cluster/hpcwork/rwth1997/Ru001-N2-v2-CN-relax/N2_v2_CN_MH1_270_194378" "/rwthfs/rz/cluster/hpcwork/rwth1997/Ru001-N2-v2-CN-relax/N2_v2_CN_MH1_325_364255" "/rwthfs/rz/cluster/hpcwork/rwth1997/Ru001-N2-v2-CN-relax/N2_v2_CN_MH1_235_840464" "/rwthfs/rz/cluster/hpcwork/rwth1997/Ru001-N2-v2-CN-relax/N2_v2_CN_MH1_265_638921" "/rwthfs/rz/cluster/hpcwork/rwth1997/Ru001-N2-v2-CN-relax/N2_v2_CN_MH1_345_886216" "/rwthfs/rz/cluster/hpcwork/rwth1997/Ru001-N2-v2-CN-relax/N2_v2_CN_MH1_245_516338" "/rwthfs/rz/cluster/hpcwork/rwth1997/Ru001-N2-v2-CN-relax/N2_v2_CN_MH1_390_143060")
        total_tasks=${#target_folders[@]}

        echo "Found $total_tasks folders to process."
        count=0
        for folder in "${target_folders[@]}"; do
            ((count++))

            echo "[$count/$total_tasks] Starting task in $folder"
            
            (
                cd "$folder" || exit
                python "$PYTHON_INPUT_SCRIPT" "$PYTHON_INPUT_CONFIG" > "SimuLog.txt" 2>&1
            ) &

        done

        echo "All $total_tasks simulations are started."
        wait
        echo "All tasks in this batch are done!"
        unset BASF_RUN_PREFIX
        unset PYTHON_INPUT_SCRIPT
        unset PYTHON_INPUT_CONFIG
        ``` 
    The above is an example showing how to execute `LAMMPS` in different folder parallelly.
    
    **Attention**

   - The product of the following 2 variables must be 24.

        ```shell
        
        #SBATCH --ntasks-per-node=24
        #SBATCH --cpus-per-task=1

        ```
    - `--cpus-per-tasks` must correspond to the `LAMMPS` task value, i.e. The value after `t`. 

        ```shell
        # The following 2 settings are suggested by `LAMMPS-KKOKOS`,But I find them in fact make the computation slower. 
        # export OMP_PLACES="threads"
        # export OMP_PROC_BIND="spread"
        srun --exact --ntasks=1 --cpus-per-task=1 --overlap lmp -k on g 1 t 1 -in in_unbiased.lammps -sf kk
        ```