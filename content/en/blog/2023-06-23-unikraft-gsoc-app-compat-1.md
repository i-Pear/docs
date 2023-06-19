+++
title = "GSoC'23: Expanding Binary Compatibility Mode (Part 1)"
date = "2023-06-20T00:00:00+01:00"
author = "Tianyi Liu"
tags = ["GSoC'23", "Binary Compatibility", "Performance Optimization"]
+++

<img width="100px" src="https://summerofcode.withgoogle.com/assets/media/gsoc-2023-badge.svg" align="right" />

# GSoC'23: Expanding Binary Compatibility Mode

## Objectives

Unikraft supports various forms of workloads.
It can be linked together with user-space programs in source code or load binary files using [`app-elfloader`](https://github.com/unikraft/app-elfloader/).
The app-elfloader supports statically linked programs as well as programs that use dynamic libraries.

There is no doubt that compiling native images from source code can provide the maximum potential for compilation optimization and deliver the highest performance.
However, for dynamically linked programs, the current approach involves providing the native `glibc.so` to the program.
When the program calls glibc functions, it triggers a system interrupt and switches to kernel mode.
The `syscall-shim` layer can intercept these interrupts and forward them to the corresponding system call handlers.
We understand that the user-space to kernel-space transition is for isolation and security purposes, but in the context of Unikernels, it loses its significance.
Therefore, we can employ various methods to bypass system interrupts and allow applications to directly access kernel functions, achieving performance similar to native images.

### Summary of Objectives

The project aims to achieve the following outcomes:

- A. Generate an adapter dynamic library during the compilation of app-elfloader.
Users can preload this library using LD_PRELOAD, allowing the application to bypass system interrupts and directly access kernel functions.
The goal is to provide performance that is as close as possible to a native image.

- B. Provide a VDSO image compiled with the kernel and pass it to the application through the loader, enabling fast handling of time-related system calls.
Both statically and dynamically linked programs can benefit from this.

- C. Introduce a regression testing CI/CD suite to detect whether new pull requests break existing libc compatibility.

- D. Utilize the remaining time to analyze the compatibility shortcomings of Unikraft and fix compatibility issues in at least two modules.

## Current Progress

A. Preload Library: 
Currently, we have successfully generated a preload library for the libc wrapper targeting system calls.
Disregarding TLS considerations, this approach has allowed the Redis Benchmark with dynamic libraries, to achieve performance close to that of a native image.

B. VDSO:
TBD.

C. CI/CD:
The regression tests can now be executed on GitHub Actions.

## Next steps

A. Preload Library: 
- Through experimentation, it has been discovered that the switching between user-space and kernel-space TLS incurs significant overhead.
Therefore, our next step is to explore the use of `FSGSBASE` to reduce these costs.

- The preload library needs to be automatically generated during the compilation of app-elfloader, which requires the implementation of some automation scripts.

- Currently, only the libc wrapper for system calls has been replaced, while other functionalities provided by libc still rely on system interrupts.
To address this, it is necessary to replace the portion of functionality provided by glibc with a modified version of Musl.

B. VDSO:
- Rapidly implementing a VDSO library to accelerate time-related system calls.

C. CI/CD:
- Find a server to host the necessary data for CI/CD and officially incorporate CI/CD into Unikraft's workflow.

## About Me

I'm Tianyi Liu, a first-year graduate student at Nanjing University.
My research interests include system software, compilers, and program analysis, etc.
