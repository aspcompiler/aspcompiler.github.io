---
title: "A First Look of PyO3"
date: 2023-07-29T22:33:22-07:00
draft: false
---

# Introduction

[PyO3](https://pyo3.rs/main/) is a Rust library for building Python bindings. In this post, we will
try to build a simple Python module with a few features using PyO3. For exhaustive documentation,
please refer to the [PyO3 user guide](https://pyo3.rs/main/).

# Setup

To learn how to setup PyO3, please follow the [PyO3 user guide](https://pyo3.rs/main/). I have
save my code in [this repository](https://github.com/aspcompiler/pyo3-test). Please follow the
README there to run my code.

# A Simple Python Module

Let us implement a single Python function in Rust.

```rust
use pyo3::prelude::*;

#[pyfunction]
fn sum_as_string(a: usize, b: usize) -> PyResult<String> {
    Ok((a + b).to_string())
}
```

We can call the function with the following Python code.

```python
import pyo3_test

print(pyo3_test.sum_as_string(3, 5))
```

The output is `8`.

# A Python Class

Next, we will implement a Python function that returns a Python class in Rust.

```rust
#[pyclass]
struct MyClass {
    #[pyo3(get, set)]
    num: i32,
}

#[pyfunction]
fn return_myclass() -> Py<MyClass> {
    Python::with_gil(|py| -> Py<MyClass> {
        Py::new(py, MyClass { num: 1 }).unwrap()
    })
}
```

Note that we use Py::new() to allocate the class in Python GIL memory.

We can use the class in Python.

```python
print(pyo3_test.return_myclass().num)
```

The output is `1`.

# A Python Iterator

Next, we will implement something very Pythonic: an iterator.

```rust
#[pyclass]
struct PyClassIter {
    count: usize,
}

#[pymethods]
impl PyClassIter {
    #[new]
    pub fn new() -> Self {
        PyClassIter { count: 0 }
    }

    fn __iter__(slf: PyRef<'_, Self>) -> PyRef<'_, Self> {
        slf
    }

    fn __next__(&mut self) -> Option<usize> {
        if self.count < 5 {
            self.count += 1;
            // Given an instance `counter`, First five `next(counter)` calls yield 1, 2, 3, 4, 5.
            Some(self.count)
        } else {
            None
        }
    }
}
```

We can use the iterator in Python.

```python
for i in pyo3_test.PyClassIter():
    print(i)
```

The output is `1`, `2`, `3`, `4`, `5`.

# A Python Async Iterator

Next, we want to have even more fun: an async iterator.

```rust
#[pyclass]
struct PyAsyncIter {
    count: Arc<Mutex<usize>>,
}

#[pymethods]
impl PyAsyncIter {
    #[new]
    pub fn new() -> Self {
        PyAsyncIter { count: Arc::new(Mutex::new(0)) }
    }

    fn __aiter__(slf: PyRef<'_, Self>) -> PyRef<'_, Self> {
        slf
    }

    fn __anext__(slf: PyRefMut<'_, Self>) -> PyResult<Option<PyObject>> {
        let count = slf.count.clone();
        let fut = pyo3_asyncio::tokio::future_into_py(slf.py(), async move {
            let mut count = count.lock().unwrap();
            if *count < 5 {
                *count += 1;
                Ok(Python::with_gil(|py| count.into_py(py)))
            } else {
                Err(PyStopAsyncIteration::new_err("stream exhausted"))
            }    
        })?;
        Ok(Some(fut.into()))
    }
}
```

There is a bit of boilerplate code. Hope [PyO3 0.20](https://github.com/PyO3/pyo3/issues/3246) will
make it easier.

We can use the async iterator in Python.

```python
async for i in pyo3_test.PyAsyncIter():
    print(i)
```

The output is `1`, `2`, `3`, `4`, `5`.

# Summary

PyO3 is a very powerful library for building Python bindings. We can build very Pythonic modules
in Rust.

Finally, we will implement a Python module in Rust.
