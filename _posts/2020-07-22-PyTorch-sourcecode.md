---
layout: post
title: PyTorch
subtitle: A reading note for pytorch source code 
thumbnail-img: /assets/img/pytorch-logo.png
cover-img: /assets/img/pytorch-logo-dark.png
share-img: /assets/img/pytorch-logo-dark.png
tags: [ note ]
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



## Get Started

*This order is only personal preference* 

#### 0x00 - CMake

(*Just want to be able to understand what the fuck is going on in `CMakeLists.txt`*)

refer to this [blog](../2020-07-23-fast-dive-into-cmake) 