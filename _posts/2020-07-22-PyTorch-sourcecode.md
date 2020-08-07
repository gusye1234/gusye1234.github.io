---
layout: post
title: PyTorch
subtitle: A reading note for pytorch source code 
thumbnail-img: /assets/img/pytorch-logo.png
cover-img: /assets/img/pytorch-logo-dark.png
share-img: /assets/img/pytorch-logo-dark.png
tags: [ note, pytorch, code]
comments: true
---

*Save words to introduce what is `pytorch`, more information in [here]( https://github.com/pytorch/pytorch/tree/release/1.5)*

## Preparation

1. Download the code 

go to the [repo](https://github.com/pytorch/pytorch/tree/release/1.5) of `pytorch` , choose a branch, I chose ``master/1.7`. And 

```shell
git clone git@github.com:pytorch/pytorch.git
```

2. find a great editor

Highly recommend vscode under our scenario: Reading source code . Since no modification required, we actually want our editor to be fast and code-navigation-supported.

3. Have a look at this [sildes](http://blog.ezyang.com/2019/05/pytorch-internals/), get some abstracts from the developer in NYC
4. [internals 1](https://pytorch.org/blog/a-tour-of-pytorch-internals-1/), [internals 2](https://pytorch.org/blog/a-tour-of-pytorch-internals-2/), [summary 1](https://www.miracleyoo.com/2019/12/11/Pytorch-Core-Code-Research/), [summary 2](http://blog.christianperone.com/2018/03/pytorch-internal-architecture-tour/)



## Tasks

1. C/CPP
   1. ATen `aten/src/Aten`, A tensor library, including the defination of data structure, cpu layout, gpu layout...
   2. TH  `aten/src/TH`, 
   3. THC  `aten/src/THC`
   4. THCUNN  `aten/src/THCUNN`
2. dependencies: 
   1. BLAS, linear algebra library, defining interface. *MKL, OMP* 
   2. …
3. python
   * high level api, we will start from here
4. Consider CPU only



## Get Started

#### 0x00 — init and `csrc`

*While building, the process changes facing different platform. There are `APPLE`, `MSVC`. Since I'm using macOS,  Will focus on the `APPLE=True`*

Look at the `setup.py`, binding src files from `csrc` fold to module `_C`

```python
C = Extension("torch._C",
                  libraries=main_libraries, # torch_python
                  sources=main_sources, # torch/csrc/stub.c
                  language='c',
                  extra_compile_args=main_compile_args + extra_compile_args,
                  include_dirs=[],
                  library_dirs=library_dirs,
                  extra_link_args=extra_link_args + main_link_args + [make_relative_rpath('lib')])
    extensions.append(C)
```

*After a quick look into python api  `nn`, `_C` actually contains many operation. We start from here*

The building order is:

1. `CMakeLists.txt` in the top level.
2. `CMakeLists.txt` in the `c10`, `caffe2`, `modules`...
3. `CMakeLists.txt` in the `caffe2` add the `torch` fold: `  add_subdirectory(../torch torch)`
4. (Why the hell `torch_python` is clearified in `caffe2`)

### **torch_python**, or `libtorch_python.dylib`

This shared library is being produced by the `torch/CMakeList.txt`. 

We further look into the dependencies libs of `torch_python` (Only for Macos)

```cmake
set(TORCH_PYTHON_LINK_LIBRARIES
    torch_library
    shm
    fmt::fmt-header-only)
# ...
list(APPEND TORCH_PYTHON_LINK_LIBRARIES c10d)
# list(APPEND <list> [<element> ...]), Appends elements to the list.
```

How about the headers?

```
set(TORCH_PYTHON_INCLUDE_DIRECTORIES
    ${PYTHON_INCLUDE_DIR}

    ${TORCH_ROOT}
    ${TORCH_ROOT}/aten/src
    ${TORCH_ROOT}/aten/src/TH

    ${CMAKE_BINARY_DIR}
    ${CMAKE_BINARY_DIR}/aten/src
    ${CMAKE_BINARY_DIR}/caffe2/aten/src
    ${CMAKE_BINARY_DIR}/third_party
    ${CMAKE_BINARY_DIR}/third_party/onnx

    ${TORCH_ROOT}/third_party/gloo
    ${TORCH_ROOT}/third_party/onnx
    ${TORCH_ROOT}/third_party/tensorpipe
    ${pybind11_INCLUDE_DIRS}

    ${TORCH_SRC_DIR}/csrc
    ${TORCH_SRC_DIR}/csrc/api/include
    ${TORCH_SRC_DIR}/lib
    )
list(APPEND TORCH_PYTHON_INCLUDE_DIRECTORIES ${LIBSHM_SRCDIR})
```

Ok, Let's get those slowly

* Headers

  * `TORCH_ROOT` is the top level fold. `CMAKE_BINARY_DIR` is the `build` , `TORCH_SRC_DIR` is the `torch`

  * `pybind11_INCLUDE_DIRS`, pybind11 is a library to binding CPP and Python. `set(pybind11_INCLUDE_DIRS ${CMAKE_CURRENT_LIST_DIR}/../third_party/pybind11/include)` is already included in `third_party`

  * `LIBSHM_SRCDIR`: 

    ```
    set(LIBSHM_SUBDIR libshm)
    set(LIBSHM_SRCDIR ${TORCH_SRC_DIR}/lib/${LIBSHM_SUBDIR})
    add_subdirectory(${LIBSHM_SRCDIR})
    ```

* shared libs

  * `torch_library` seems like is `torch`. Cause in the `CMakeLists.txt` of `caffe2`, we have `caffe2_interface_library(torch torch_library)`, and `caffe2_interface_library` is a marco

    ```
    # cmake/utils.cmake
    # What caffe2_interface_library does is create an interface library
    # that indirectly depends on the real library, but sets up the link
    # arguments so that you get all of the extra link settings you need.
    # The result is not a "real" library, and so we have to manually
    # copy over necessary properties from the original target.
    ```

    Anyway, skip it. We claim `torch_library` is just like `torch`😊. And `target_link_libraries(torch PUBLIC torch_cpu_library)`. Seems like `torch_cpu` is just the `torch_cpu_library`. So `torch_cpu`

    ```cmake
    set(Caffe2_CPU_SRCS ${Caffe2_CPU_SRCS_NON_AVX} ${Caffe2_CPU_SRCS_AVX} ${Caffe2_CPU_SRCS_AVX2})
    add_library(torch_cpu ${Caffe2_CPU_SRCS})
    ...
    target_link_libraries(torch_cpu PUBLIC c10)
    target_link_libraries(torch_cpu PUBLIC ${Caffe2_PUBLIC_DEPENDENCY_LIBS})
    target_link_libraries(torch_cpu PRIVATE ${Caffe2_DEPENDENCY_LIBS})
    target_link_libraries(torch_cpu PRIVATE ${Caffe2_DEPENDENCY_WHOLE_LINK_LIBS})
    target_include_directories(torch_cpu INTERFACE $<INSTALL_INTERFACE:include>)
    target_include_directories(torch_cpu PRIVATE ${Caffe2_CPU_INCLUDE})
    target_include_directories(torch_cpu SYSTEM PRIVATE "${Caffe2_DEPENDENCY_INCLUDE}")
    
    ```

    Bascially, we now track down the bottom

    ```cmake
    list(APPEND Caffe2_CPU_SRCS ${ATen_CPU_SRCS})
    ...
    foreach(input_filename ${Caffe2_CPU_SRCS})
      if(${input_filename} MATCHES "AVX\\.cpp")
        list(APPEND Caffe2_CPU_SRCS_AVX ${input_filename})
      elseif(${input_filename} MATCHES "AVX2\\.cpp")
        list(APPEND Caffe2_CPU_SRCS_AVX2 ${input_filename})
      else()
        list(APPEND Caffe2_CPU_SRCS_NON_AVX ${input_filename})
      endif()
    endforeach(input_filename)
    ...
    # aten/src/ATen/CMakeLists.txt 
    set(all_cpu_cpp ${base_cpp} ${ATen_CORE_SRCS} ${native_cpp} ${native_sparse_cpp} ${native_quantized_cpp} ${native_mkl_cpp} ${native_mkldnn_cpp} ${native_xnnpack} ${generated_cpp} ${core_generated_cpp} ${ATen_CPU_SRCS} ${ATen_QUANTIZED_SRCS} ${cpu_kernel_cpp})
    if(AT_MKL_ENABLED)
      set(all_cpu_cpp ${all_cpu_cpp} ${mkl_cpp})
    endif()
    if(AT_MKLDNN_ENABLED)
      set(all_cpu_cpp ${all_cpu_cpp} ${mkldnn_cpp})
    endif()
    if(USE_VULKAN)
      set(all_cpu_cpp ${all_cpu_cpp} ${native_vulkan_cpp} ${vulkan_generated_cpp})
    else()
      set(all_cpu_cpp ${all_cpu_cpp} ${native_vulkan_stub_cpp})
    endif()
    ...
    set(ATen_CPU_SRCS ${all_cpu_cpp})
    ```

    I won't expand all those *_cpp variable means, or maybe later

  * `shm`, located in `torch/lib/libshm`. Won't go further

  * `c10d`, located in `torch/lib/c10d`. (**Confused**)

Finally, be clear about the `torch_python`, we already have a clue to find the source cpp codes.

### csrc

>  C++ files composing the PyTorch library. Files in this directory tree are a mix of Python binding code, and C++ heavy lifting.

* jit - Compiler and frontend for TorchScript JIT frontend. 
* autograd - Implementation of reverse-mode automatic differentiation.
* api - The PyTorch C++ frontend.
* distributed -  Distributed training support for PyTorch.

Above have their indiviual README.

It all starts at `torch/csrc/stub.c`

```
PyMODINIT_FUNC PyInit__C(void)
{
  return initModule();
}
```



  

