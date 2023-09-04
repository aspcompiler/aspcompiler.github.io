---
title: "Is Rust a Good Language for Machine Learning?"
date: 2023-09-04T08:09:34-07:00
draft: false
---

# Introduction

Rust has been known for its performance and memory/thread safety. However, it takes an ecosystem to be successful. So far, Rust has found
a lot of success in network programming and bit coin mining. Rust has an excellent support for `async` making it a great language any time
we need distributed systems.

For the reason, Rust has found a lot of success in vector databases.

# Vector Databases

There are several start-up companies that are building vector databases using Rust: [Pinecone](https://www.pinecone.io/blog/rust-rewrite/),
[Qdrant](https://github.com/qdrant/qdrant) and [Fennel.ai](https://fennel.ai/blog/vector-search-in-200-lines-of-rust/).

## Const generics

Rust has an interesting feature called [const generics](https://doc.rust-lang.org/reference/items/generics.html) that is uniquely suitable for vector databases. Const generics are generic arguments that range over constant values, rather than types or lifetimes. This allows, for instance, types to be parameterized by integers. The follow are some examples of const generics:

```rust
// Examples where const generic parameters can be used.

// Used in the signature of the item itself.
fn foo<const N: usize>(arr: [i32; N]) {
    // Used as a type within a function body.
    let x: [i32; N];
    // Used as an expression.
    println!("{}", N * 2);
}

// Used as a field of a struct.
struct Foo<const N: usize>([i32; N]);

impl<const N: usize> Foo<N> {
    // Used as an associated constant.
    const CONST: usize = N * 4;
}

trait Trait {
    type Output;
}

impl<const N: usize> Trait for Foo<N> {
    // Used as an associated type.
    type Output = [i32; N];
}
```

# Frameworks

Rust has been growing in machine learning frameworks as well, for example, Hugging Face [candle](https://github.com/huggingface/candle) and [burn-rs](https://github.com/burn-rs/burn). Each of these frameworks can only run some models. For example, for `candle`, [candle-whisper](https://huggingface.co/spaces/lmz/candle-whisper) and [candle-llama2](https://huggingface.co/spaces/lmz/candle-llama2) and for `burn`, [whisper-burn](https://github.com/Gadersd/whisper-burn) and [llama2-burn](https://github.com/Gadersd/llama2-burn).

These frameworks can work in spacial cases where frameworks like `PyTorch` are too large. For example, Candle's core goal is to make `serverless inference` possible.

This frameworks gave proof that Rust is a viable language for machine learning. They also built-up an ecosystem of Rust libraries that could be very useful for other Rust projects, for example, [tch-rs](https://github.com/LaurentMazare/tch-rs). Hugging Face also has [several tools developed in Rust](https://github.com/huggingface?q=&type=all&language=rust&sort=).

# References

I have only touched a small part of machine learning in Rust. For more comprehensive reviews, please check out the following references:
* [Are we learning yet](https://www.arewelearningyet.com/)
* [Awesome-Rust-MachineLearning](Awesome-Rust-MachineLearning)
