---
layout: post
title: CMake tutorial
subtitle: keep it easy, simple, and cool
tags: [ note ]
comments: true
---
All the source code can be found [here](https://github.com/ttroy50/cmake-examples)



## 0x00 basic

```
pre-compile -> compile(separate compilation) -> assembly -> link
```

In c++, we can do two things:

* Statement, like `void print()`. 
* Definition, like `void print(){std::cout<<"Hi"<<std::endl;}`

The propose of statement is to state a symbol, which can be function, class, struct…, then the symbol will go into the so-called **Symbol-Tabel**. When we compiling, actually we can "skip" the statement like `void print()`, and put it into **Symbol-Tabel**. Then, when we link different source file, we can look for the definition of this statement. For example

```
# a.cpp
void print();

# b.cpp
void print(){
	std::cout<<"Hi"<<std::endl;
}
```

So , in fact, we can run `clang -c a.cpp`, `clang -c b.cpp`  separately(*or `g++` for linux*). And we will get, `a.o`, `b.o`(`.o` stands for object), noting that `a.o` doesn't have any information about the `print` function right now.

In order to create an Executable, we must link them:

```shell
clang a.o b.o -o main
# or
# clang -undefined dynamic_lookup a.o b.o -o main
```

Then, we get what we want. But, A critical problem about the above Statement, it's realy not so automate, let's say, there are more than 100 cpp files `*.cpp` need to state multiple functions `f1`, `f2`, `f3` and stuffs, whose definitions are in `b.cpp`, do we need to add statement on every one of them? That's why we need header,  which just a file can be automatically unfold by compiler.

```
# header.h
void f1();
void f2();
#....
```

And, the familiar macro command `include` shows up. 

>  There are more rules about how to write header, adding `#define` to avoid being included by a file more than once, ...

## 0x01 make

