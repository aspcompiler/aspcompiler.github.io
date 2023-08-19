---
title: "Practical Rust Ownership"
date: 2023-08-19T10:32:41-07:00
draft: false
---

# Introduction

Rust `ownership` is often listed as [one of the top challenging areas](https://opensource.googleblog.com/2023/06/rust-fact-vs-fiction-5-insights-from-googles-rust-journey-2022.html) of Rust. Contributing to this challenge is that the practical perspective
of the Rust ownership is spread across many chapters the [Rust book](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html). In this post, 
I will attempt to work through Rust ownership in a practical perspective.

I do not intend this post to be an introduction material that cover every details. Instead, I intend this to be a guide that points to various Rust book chapters.

# Stack and Heap

Like many other languages, Rust can allocate memory on a stack or a heap. Like C/C++, and unlike languages with garbage collection like Java or C#,
Rust developers often prefer to allocate memory on a stack. The reason is that stack allocation is faster and deallocation does not cause memory fragmentation. 
The first think to keep in mind is that variable allocated on a stack cannot outlive the function that allocated it.

Question: When we create a variable, how do we know if it is allocated on a stack or a heap?

Answer: It depends on types. Scalar Types are allocated on a stack. Fix-sized Compound Types such as arrays, tuples and structs
are also allocated on a stack if they only have fix-sized members. Growable types such as `String` and `Vec` are allocated on a heap. 
To explicitly allocate on a heap a type that otherwise would be allocated on a stack, we can use [`Box` type](https://doc.rust-lang.org/book/ch15-01-box.html?highlight=box#enabling-recursive-types-with-boxes).

See The Stack and the Heap section in the [Rust book](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#the-stack-and-the-heap). Also see the [Data Types](https://doc.rust-lang.org/book/ch03-02-data-types.html) chapter.

# Ownership

The Rust ownership rule is fairly straight forward: 

* Each value in Rust has an owner.
* There can only be one owner at a time.
* When the owner goes out of scope, the value will be dropped.

Question: when we assign a variable to another variable or when we pass a variable to a function, does the ownership transfer?

Answer: it depends on the type. If the type implements the `Copy` trait, the value will be copied. We sometimes call these types `copy`. Otherwise, the ownership
transfers and we call it `move`.

See the [Ownership](https://doc.rust-lang.org/book/ch04-01-what-is-ownership) chapter for more details.

# References

It would be very inconvenient if we always have to copy or move variables when we pass them to a function. Rust has another mechanism called `reference` or `borrowing`. Reference is like a pointer. Rust has the following rules for references:

* At any given time, you can have either one mutable reference or any number of immutable references.
* References must always be valid.

Question: Does the following code compile?

```rust
fn main() {
    let len = String::from("hello").len();

    println!("The length is {}.", len);
}
```

The answer is yes. During method chaining, Rust can create a temporary ownership without requiring us to explicitly create a owning variable.

See the [References and Borrowing](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html) chapter for more details.

# The Slice Type

Slices let you reference a contiguous sequence of elements in a collection rather than the whole collection. We usually see slices in String or arrays.

Question: If we design a function, how do we decide whether to pass parameter by moving, or borrowing or slicing? How about return type?

Answer:

1. For passing parameter, consider reference first unless moving ownership is the right thing to do (e.g., `setValues`). Use ```mut T` if the function needs to mutate the parameter.
1. If the type supports slicing, consider using slice as the parameter type or return type first.
1. For return type, move ownership if the function creates a new value.
1. If the function receives parameters by reference and returns a reference, check the `lifetime`.

See the [Slices](https://doc.rust-lang.org/book/ch04-03-slices.html) chapter for more details.

# Lifetime

In the previous section, we mentioned `lifetime`. Lifetime is a mechanism to ensure that references are valid as long as we need them to be.

An example of lifetime is the following code:

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

Since the compiler cannot determine whether x or y will be returned, we need to tell the compiler that `x` and `y` needs to be valid. Sometimes, a function
compiles without lifetime annotations. This is called called [lifetime elision](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#lifetime-elision).

# Box, Rc and RefCell

Earlier, we mentioned that `Box` is a type that allocates on a heap. `Box` is a single ownership start pointer. In contrast, `Rc` is a [reference counted 
smart pointer](https://doc.rust-lang.org/book/ch15-04-rc.html). `RefCell` is a mechanism allows you to mutate data even when there are immutable references 
to that data. This is called [interior mutability](https://doc.rust-lang.org/book/ch15-05-interior-mutability.html). The borrowing rules are enforced at runtime.

# Ownership and Threads

`Rc` cannot be shared across threads. To share data across threads, we need to use `Arc` which is an atomic reference count type. `Arc` is often used together
with `Mutex` to synchronize access to data. For example:

```rust
let counter = Arc::new(Mutex::new(0));

// To use
let mut num = counter.lock().unwrap();
*num += 1;
```

See the [Concurrency](https://doc.rust-lang.org/book/ch16-00-concurrency.html) chapter for more details.

Lastly, the type that can be transferred across threads is called `Send`. The type that can be shared across threads is called `Sync`. See the [Send and Sync](https://doc.rust-lang.org/book/ch16-02-message-passing.html#sending-multiple-messages) chapter for more details.

# Final Thoughts

To put the knowledge together, let's ask the following question: should a `struct` owns its members?

```rust
struct User {
    id: u32,
    name: String,
    screen_name: String,
    location: String,
}
```
or

```rust
struct User<'a> {
    id: u32,
    name: &'a str,
    screen_name: &'a str,
    location: &'a str,
}
```

Generally, we prefer to keep everything in the same place. However, the first struct is fairly rigid: we have to construct the struct at once and we cannot move ownership out of the struct. The stepwise construction is needed or if we need to move ownership out of the struct, we need to use `Option`.

The second struct is useful when the data is already owned by a buffer. See zero-copy in [serde](https://serde.rs/lifetimes.html).

# Summary

We took a lap around Rust ownership. We pointed to various chapters in the Rust book. We also discussed some practical considerations when designing a function or a struct.