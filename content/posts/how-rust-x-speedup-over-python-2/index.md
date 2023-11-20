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

# Dive into the code

## Rayon

Rayon is an amazing library for parallel processing. It takes just one method change to parallelize the code. In the
example below, we literally just change `chunks_mut` to `par_chunks_mut` and the code is running in parallel.

```rust
    if parallel {
        out.par_chunks_mut(width_in_blocks)
            .enumerate()
            .for_each(|(i, row)| {
                let y = f32s::splat(min_y + dy * (i as f32));
                row.iter_mut().enumerate().for_each(|(j, count)| {
                    let x = xs[j];
                    let z = Complex { real: x, imag: y };
                    *count = MandelbrotIter::new(z).count(iters);
                });
            });
    } else {
        out.chunks_mut(width_in_blocks)
            .enumerate()
            .for_each(|(i, row)| {
                let y = f32s::splat(min_y + dy * (i as f32));
                row.iter_mut().enumerate().for_each(|(j, count)| {
                    let x = xs[j];
                    let z = Complex { real: x, imag: y };
                    *count = MandelbrotIter::new(z).count(iters);
                });
            });
    }
```

## SIMD

SIMD does have a few new concepts. The first is the SIMD vector types, for example:
    
```rust
type u32s = u32x8;
type f32s = f32x8;
type m32s = m32x8;
```

In the above example, each type is a vector of length 8. `m32` is a `mask` type that we will cover later.

To create a vector from a scalar, we use the `splat` method:

```rust
32s::splat(4.0)
```

We can use arithmetic operators on vectors like we do on scalars:

```rust
let xx = x * x;
let yy = y * y;
let sum = xx + yy;
```

Now let us go back to `mask`. `mask` can be used as a vector of boolean. Comparisons between vectors return a `mask`.
we can use the `select` method like `if ... else ...` logic for vectors:

```rust
let mask = sum.le(f32s::splat(4.0));
let count = mask.select(count + 1, count);
```

There are a lot more to SIMD. I recommend the [Rust SIMD book](https://rust-lang.github.io/packed_simd/) for more details.
Although `packed-simd` is getting replaced by [portable-simd](https://github.com/rust-lang/portable-simd). The former is
still has the better documentation.
