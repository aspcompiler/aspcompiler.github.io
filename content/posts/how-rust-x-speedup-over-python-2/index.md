---
title: "How Rust gets an x speedup over Python - Part 2"
date: 2023-11-19T09:45:56-08:00
draft: false
---

# Introduction

In the [part 1](../how-rust-x-speedup-over-python-1) of this series, we tried to reproduce the speed up over Python
in [part 1 of a Modular blog](https://www.modular.com/blog/how-mojo-gets-a-35-000x-speedup-over-python-part-1), but in `Rust``. We achieved 46x speedup over the base case. In this blog, we will try to reproduce
[part 2](https://www.modular.com/blog/how-mojo-gets-a-35-000x-speedup-over-python-part-2) and 
[part 3](https://www.modular.com/blog/mojo-a-journey-to-68-000x-speedup-over-python-part-3) of the modular blog.

# SIMD

We will first explore the [SIMD instructions](https://en.wikipedia.org/wiki/Single_instruction,_multiple_data). SIMD stands for Single Instruction Multiple Data. SIMD instructions are a type of instructions that can be executed on multiple data in parallel. 

Currently, it is necessary to use Rust nightly to use SIMD instructions. Rust nightly can be installed with the following command:

```bash
rustup toolchain install nightly
```

The nightly toolchain can be invoked with the `+nightly` argument in the `cargo` command. Since we invoke the `cargo`
command indirectly through the `maturin` tool, we instead add a `rust-toolchain.toml` file in the root of the project with the following content:

```toml
[toolchain]
channel = "nightly"
```

`maturin` is the build tool for [PyO3](https://pyo3.rs/v0.20.0/) used to build Rust packages for Python.

We achieved an execution time of 30.2ms which is a 213x single-core speedup over the base case. 

# Parallelization

Part of speedup achieved in the Modular blog is through parallelization. This is not a fair comparison to the single-core
Python baseline. It is worth nothing Python also have parallel processing libraries like [Ray](https://www.ray.io/) or
[Dask](https://www.dask.org/). For the sake of comparison, we tried parallelization in Rust using [Rayon](https://docs.rs/rayon/latest/rayon/).

Using my macPro with 6 cores and 12 hyperthreads, we achieved an execution time of 18.2ms without SIMD, which is a 354x speedup over the base case. With SIMD, we achieved an execution time of 4.8ms, which is a 1342x speedup over the base case.

# Summary

| Method    | Time (s) | Speedup |
| -------- | ------- | ------- |
| Baseline  | 6.44   | 1.00    |
| Numpy vectorization | 6.37 | 1.01 |
| Numpy parallelization | 1.71 | 3.77 |
| Numba | 0.681 | 9.46 |
| Rust | 0.141 | 45.74 |
| Rust SIMD | 0.0302 | 213.24 |
| Rust + Rayon | 0.0182 | 354.40 |
| Rust SIMD + Rayon | 0.0048 | 1342.50 |

We dd not try to match the very impressive speedup in the Modular blog as we did not try on a machine with 88 cores. Nevertheless, we tried our test cases in very accessible hardware and it should be useful in common use cases.

See the [source code](https://github.com/aspcompiler/py-rust-mandel) for the full implementation.