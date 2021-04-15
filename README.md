# DensePose-Install
The instructions for installing DensePose. Install before checking that the fuction of [Detectron2-DensePose](https://github.com/facebookresearch/detectron2/tree/master/projects/DensePose) wheather meets your demand. Because the process of installing Detectron2 is way much easier. 
# DensePose安装

*此为DensePose旧版本安装,新版本安装请直接参照[Detectron2](https://github.com/facebookresearch/detectron2)安装*



## 安装环境(Environment)：

Anaconda（Python 3.6）

CUDA:10.0 with cuDNN 7.6 [CUDA:9.0也支持]

Pytorch 1.1.0



## 安装依赖项(Install Dependencies)：

1.通过conda创建python 3.6环境(build python3.6 environment via conda) 

```
$ conda create -y -n your_env_name python=3.6
```

为了后续安装指导方便，设立一些环境变量(For convenience in the following instructions, we set some environment variables accordingly:)

~~~
$ export CONDA_ENV_PATH=/path/to/anaconda3/envs/your_env_name
$ export CONDA_PKGS_PATH=/path/to/anaconda3/envs/your_env_name/lib/pythonx.x/site-packages
~~~

2.依次安装对应需要的Python依赖包(install the following python packages under the created environment:)

~~~
conda install -y numpy setuptools cffi typing pyyaml=3.13 mkl=2019.1 mkl-include=2019.1
~~~

~~~
conda install -y cython matplotlib
~~~

~~~
conda install -y pydot future networkx
~~~

~~~
conda install -y opencv mock scipy h5py
~~~

~~~
pip install chumpy
~~~



## COCO API

~~~
# COCOAPI=/path/to/clone/cocoapi
$ git clone https://github.com/cocodataset/cocoapi.git $COCOAPI
$ cd $COCOAPI/PythonAPI
# Install into global site-packages
$ make install
~~~



## PyTorch & Caffe2

Pytorch版本最好为1.1.0，新版本可能会出问题，cudatoolkit版本请参照对应系统版本

~~~
conda install pytorch==1.1.0 torchvision==0.3.0 cudatoolkit=10.0 -c pytorch
~~~

因为版本原因，对应安装protobuf，版本过新与过久都会引起后续编译问题

~~~
conda install protobuf=3.6.1
~~~

### Caffe2安装验证

~~~
# To check if Caffe2 build was successful
$ python -c 'from caffe2.python import core' 2>/dev/null && echo "Success" || echo "Failure"

# To check if Caffe2 GPU build was successful
# This must print a number > 0 in order to use Detectron
$ python -c 'from caffe2.python import workspace; print(workspace.NumCudaDevices())'
~~~

为了后续安装指导方便，设立一些环境变量

~~~
$ export TORCH_PATH=$CONDA_PKGS_PATH/torch
$ export CAFFE2_INCLUDE_PATH=$TORCH_PATH/include/caffe2
~~~



## GCC编译器安装

为了保证Caffe2的编译，过新的gcc编译器编译后的结果会存在一些问题，例如

~~~
OSError: /path/to/densepose/build/libcaffe2_detectron_custom_ops_gpu.so: undefined symbol: _ZN6caffe219CPUOperatorRegistryB5cxx11Ev
~~~

在此建议安装已经成功测试过的gcc4.9.2版本

### 安装GCC

下载[GCC-4.9.2](https://ftp.gnu.org/gnu/gcc/gcc-4.9.2/) 并放在一个合适的地方，并依照以下步骤进行安装

1.解压

~~~
$tar -zxvf gcc-4.9.2.tar.gz
~~~

2.下载依赖项

~~~
$ cd gcc-4.9.2
$ ./contrib/download_prerequisites
$ cd ..
~~~

3.建立编译文件夹并编译安装

~~~
$ mkdir gcc-build
$ cd gcc-build
$ ../gcc-4.9.2/configure --prefix=/path/to/gcc_install --enable-checking=release --enable-languages=c,c++ --disable-multilib
$ make -j && make install
$ cd /path/to/gcc_install/bin/
$ ln -s gcc cc
$ cp -r /path/to/gcc_install/include/c++ $CONDA_ENV_PATH/include
~~~

4.添加环境变量

 ~~~
$ export PATH=/path/to/gcc_install/bin:$PATH
$ export LD_LIBRARY_PATH=/path/to/install/gcc/lib/:/path/to/gcc_install/lib64:$LD_LIBRARY_PATH
$ source ~/.bashrc
 ~~~

5.检查版本

~~~
$ gcc -v
~~~



## DensePose安装

1. 先克隆修改适配Python3.6环境的DensePose项目

   ~~~
   $ git clone https://github.com/Johnqczhang/DensePose $DENSEPOSE
   ~~~

2. 设置并建立Python模块

   ~~~
   $ cd $DENSEPOSE
   $ python setup.py develop
   ~~~

3. 验证Detectron tests通过 

   ~~~
   $ python $DENSEPOSE/detectron/tests/test_spatial_narrow_as_op.py
   ~~~

   通过结果：

   ~~~
   [E init_intrinsics_check.cc:43] CPU feature avx is present on your machine, but the Caffe2 binary is not compiled with it. It means you may not get the full speed of your CPU.
   [E init_intrinsics_check.cc:43] CPU feature avx2 is present on your machine, but the Caffe2 binary is not compiled with it. It means you may not get the full speed of your CPU.
   [E init_intrinsics_check.cc:43] CPU feature fma is present on your machine, but the Caffe2 binary is not compiled with it. It means you may not get the full speed of your CPU.
   Found Detectron ops lib: /root/anaconda3/envs/Densepose/lib/python3.6/site-packages/torch/lib/libcaffe2_detectron_ops_gpu.so
   ...
   ----------------------------------------------------------------------
   Ran 3 tests in 3.263s
   
   OK
   ~~~

   

4. 对DensePose里的CMakeList进行编辑，下载[CMakeList.txt](https://github.com/Johnqczhang/densepose_installation/blob/master/CMakeLists.txt) , 并将文件的路径对应修改

    样例：

   ~~~
   cmake_minimum_required(VERSION 2.8.12 FATAL_ERROR)

   # set caffe2 cmake path manually
   set(Caffe2_DIR "/root/anaconda3/envs/Densepose/lib/python3.6/site-packages/torch/share/cmake/Caffe2/")
   # set cuDNN path
   set(CUDNN_INCLUDE_DIR "/usr/local/cuda-10.0/targets/x86_64-linux/include")
   set(CUDNN_LIBRARY "/usr/local/cuda-10.0/targets/x86_64-linux/lib/libcudnn.so")
   include_directories("/root/anaconda3/envs/Densepose/include/")
   # add static protobuf library
   add_library(libprotobuf STATIC IMPORTED)
   set(PROTOBUF_LIB "/root/anaconda3/envs/Densepose/lib/libprotobuf.a")
   set_property(TARGET libprotobuf PROPERTY IMPORTED_LOCATION "${PROTOBUF_LIB}")

   # Find the Caffe2 package.
   # Caffe2 exports the required targets, so find_package should work for
   # the standard Caffe2 installation. If you encounter problems with finding
   # the Caffe2 package, make sure you have run `make install` when installing
   # Caffe2 (`make install` populates your share/cmake/Caffe2).
   find_package(Caffe2 REQUIRED)
   include_directories(${CAFFE2_INCLUDE_DIRS})

   if (${CAFFE2_VERSION} VERSION_LESS 0.8.2)
     # Pre-0.8.2 caffe2 does not have proper interface libraries set up, so we
     # will rely on the old path.
     message(WARNING
         "You are using an older version of Caffe2 (version " ${CAFFE2_VERSION}
         "). Please consider moving to a newer version.")
     include(cmake/legacy/legacymake.cmake)
     return()
   endif()

   # Add compiler flags.
   set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c11")
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -O2 -fPIC -Wno-narrowing")

   # Print configuration summary.
   include(cmake/Summary.cmake)
   detectron_print_config_summary()

   # Collect custom ops sources.
   file(GLOB CUSTOM_OPS_CPU_SRCS ${CMAKE_CURRENT_SOURCE_DIR}/detectron/ops/*.cc)
   file(GLOB CUSTOM_OPS_GPU_SRCS ${CMAKE_CURRENT_SOURCE_DIR}/detectron/ops/*.cu)

   # Install custom CPU ops lib.
   add_library(
       caffe2_detectron_custom_ops SHARED
       ${CUSTOM_OPS_CPU_SRCS})

   target_include_directories(
       caffe2_detectron_custom_ops PRIVATE
       ${CAFFE2_INCLUDE_DIRS})

   target_link_libraries(caffe2_detectron_custom_ops caffe2_library libprotobuf)
   install(TARGETS caffe2_detectron_custom_ops DESTINATION lib)

   # Install custom GPU ops lib, if gpu is present.
   if (CAFFE2_USE_CUDA OR CAFFE2_FOUND_CUDA)
     # Additional -I prefix is required for CMake versions before commit (< 3.7):
     # https://github.com/Kitware/CMake/commit/7ded655f7ba82ea72a82d0555449f2df5ef38594
     list(APPEND CUDA_INCLUDE_DIRS -I${CAFFE2_INCLUDE_DIRS})
     CUDA_ADD_LIBRARY(
         caffe2_detectron_custom_ops_gpu SHARED
         ${CUSTOM_OPS_CPU_SRCS}
         ${CUSTOM_OPS_GPU_SRCS})

     target_link_libraries(caffe2_detectron_custom_ops_gpu caffe2_gpu_library libprotobuf)
     install(TARGETS caffe2_detectron_custom_ops_gpu DESTINATION lib)
   endif()

   ~~~

5. 因为一些math、threadpool库的丢失原因，而pytorch新版本源码的math与threadpool与旧版不兼容，故推荐下载旧版本的[math](https://github.com/AndyVerne/DensePose-Install/tree/main/math)和[threadpool](https://github.com/AndyVerne/DensePose-Install/tree/main/threadpool)放于/path/to/CAFFE2_INCLUDE_PATH/utils/下

   

6. 编译

   ~~~
   $ cd $DENSEPOSE/build
   $ cmake .. && make
   ~~~

7. 因为gcc4.9.2版本过低，cmake .. && make可能遇见的**问题** 

   ~~~
   cmake: /path/to/gcc_install/lib64/libstdc++.so.6: version `CXXABI_1.3.9' not found (required by cmake)
   cmake: /path/to/gcc_install/lib64/libstdc++.so.6: version `GLIBCXX_3.4.21' not found (required by cmake)
   cmake: /path/to/gcc_install/lib64/libstdc++.so.6: version `GLIBCXX_3.4.21' not found (required by /usr/lib/x86_64-linux-gnu/libjsoncpp.so.1)
   ~~~

   查看动态库(CXXABI,GLIBCXX)

   ~~~
   strings /path/to/gcc_install/lib64/libstdc++.so.6 | grep CXXABI
   ~~~

   gcc 4.9.2的版本libstdc++.so.6过老，导致库缺少，

   这里推荐/path/to/anaconda/lib/libstdc++.so.6替换/path/to/gcc_install/lib64/libstdc++.so.6

8. 检查custom operator tests是否通过：

   ~~~
   $ python $DENSEPOSE/detectron/tests/test_zero_even_op.py
   ~~~

   成功编译后会有以下结果：

   ~~~
   $ python detectron/tests/test_zero_even_op.py
   [E init_intrinsics_check.cc:43] CPU feature avx is present on your machine, but the Caffe2 binary is not compiled with it. It means you may not get the full speed of your CPU.
   [E init_intrinsics_check.cc:43] CPU feature avx2 is present on your machine, but the Caffe2 binary is not compiled with it. It means you may not get the full speed of your CPU.
   [E init_intrinsics_check.cc:43] CPU feature fma is present on your machine, but the Caffe2 binary is not compiled with it. It means you may not get the full speed of your CPU.
   ............
   ----------------------------------------------------------------------
   Ran 12 tests in 3.155s
   
   OK
   ~~~

   

## Fetch DensePose data.

Get necessary files to run, train and evaluate DensePose.

```
cd $DENSEPOSE/DensePoseData
bash get_densepose_uv.sh
```

For training, download the DensePose-COCO dataset:

```
bash get_DensePose_COCO.sh
```

For evaluation, get the necessary files:

```
bash get_eval_data.sh
```

## Setting-up the COCO dataset.

Create a symlink for the COCO dataset in your `datasets/data` folder.

```
ln -s /path/to/coco $DENSEPOSE/detectron/datasets/data/coco
```

Create symlinks for the DensePose-COCO annotations

```
ln -s $DENSEPOSE/DensePoseData/DensePose_COCO/densepose_coco_2014_minival.json $DENSEPOSE/detectron/datasets/data/coco/annotations/
ln -s $DENSEPOSE/DensePoseData/DensePose_COCO/densepose_coco_2014_train.json $DENSEPOSE/detectron/datasets/data/coco/annotations/
ln -s $DENSEPOSE/DensePoseData/DensePose_COCO/densepose_coco_2014_valminusminival.json $DENSEPOSE/detectron/datasets/data/coco/annotations/
```

Your local COCO dataset copy at `/path/to/coco` should have the following directory structure:

```
coco
|_ coco_train2014
|  |_ <im-1-name>.jpg
|  |_ ...
|  |_ <im-N-name>.jpg
|_ coco_val2014
|_ ...
|_ annotations
   |_ instances_train2014.json
   |_ ...
```

