---
layout: post
title: Compiling AMDGPU LLVM IR
date:   2016-06-01 14:34:25
categories: amdgpu llvm ir
tags: featured
image: /assets/article_images/2014-11-30-mediator_features/night-track.JPG
image2: /assets/article_images/2014-11-30-mediator_features/night-track-mobile.JPG
---

AMDGPU LLVM IR, as the name suggests is LLVM Intermediate Representation of kernels that can run on AMD GPUs. Current `master` branch of LLVM ships with AMDGPU compiler infrastructure which can translate LLVM IR to AMD GPU ISA (Yes, the actual Instruction Set Architecture). 

The ISA generated in this post are device functions rather than global kernel functions.

### Installation
```
$ git clone https://github.com/llvm-mirror/llvm.git
$ mkdir build
$ cd build
$ cmake .. -DCMAKE_BUILD_TYPE=Release -DLLVM_TARGETS_TO_BUILD="AMDGPU;X86"
$ make
```

### Usage
After making sure that `llc` is callable, the working of LLVM can be tested using the following IR.

```
; filename = vCopy.ll
define void @vector_copy(i32 addrspace(1)* %in, i32 addrspace(1)* %out) {
    %tid = call i32 @llvm.amdgcn.workitem.id.x()
    %in_ptr = getelementptr i32, i32 addrspace(1)* %in, i32 %tid
    %out_ptr = getelementptr i32, i32 addrspace(1)* %out, i32 %tid
    %in_val = load i32, i32 addrspace(1)* %in_ptr
    store i32 %in_val, i32 addrspace(1)* %out_ptr
    ret void
}

declare i32 @llvm.amdgcn.workitem.id.x()
```

The code can be compiled with

```
llc -march=amdgcn vCopy.ll
```

