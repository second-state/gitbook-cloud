---
description: >-
  Access system resources, such as random numbers, file system, and network from
  WebAssembly applications
---

# WASI

## This page is outdated. Please[ visit here for the most up-to-date content](https://www.secondstate.io/articles/wasi-access-system-resources/).

{% hint style="info" %}
If WASM+WASI existed in 2008, we wouldn't have needed to created Docker. That's how important it is. Webassembly on the server is the future of computing. A standardized system interface was the missing link. Let's hope WASI is up to the task! _**-- Solomon Hykes, Co-founder of Docker**_
{% endhint %}

The WebAssembly Systems Interface \(WASI\) is a standard extension for WebAssembly bytecode applications to make operating system calls. It is fully supported in the SSVM. WASI defines a set of function names to perform operating system tasks, such as opening a file. When the WebAssembly VM encounters those function names at runtime, it automatically calls the corresponding operating system standard library function to perform the task and return the result.

In order for WASI to work, we need a compiler toolchain to compile Rust \(or other languages\) standard library functions, such as opening a file, into bytecode that makes the corresponding WASI calls. The [`ssvmup`](https://github.com/second-state/ssvmup) tool uses the `wasm32-wasi` compiler backend for Rust. It supports WASI out of the box.

{% hint style="success" %}
The example source code for this tutorial is [here](https://github.com/second-state/wasm-learning/tree/master/nodejs/wasi).
{% endhint %}

### Get random number

The WebAssembly VM is a pure software construct. It does not have a hardware entropy source for random numbers. That's why WASI defines a function for WebAssembly programs to call its host operating system to get a random seed. As a Rust developer, all you need is to use the popular \(de facto standard\) `rand` and `getrandom` crates. Those crates are written in a way that instructs the `wasm32-wasi` compiler backend to generate the correct WASI calls in the WebAssembly bytecode. The `Cargo.toml` dependencies are as follows.

```text
[dependencies]
rand = "0.7.3"
getrandom = "0.1.14"
wasm-bindgen = "=0.2.61"
```

The Rust code to get random number from WebAssembly is this.

```text
use wasm_bindgen::prelude::*;
use rand::prelude::*;

#[wasm_bindgen]
pub fn get_random_i32() -> i32 {
  let x: i32 = random();
  return x;
}

#[wasm_bindgen]
pub fn get_random_bytes() -> Vec<u8> {
  let mut vec: Vec<u8> = vec![0; 128];
  getrandom::getrandom(&mut vec).unwrap();
  return vec;
}
```

The Javascript code to call the Rust functions from Node.js is as follows.

```text
const { get_random_i32, get_random_bytes } = require('../pkg/wasi_example_lib.js');

console.log( "My random number is: ", get_random_i32() );
console.log( "My random bytes are");
console.hex( get_random_bytes() );
```

Now, let's run this example in SSVM in Node.js.

```text
$ ssvmup build
$ node node/app.js
... ...
```

More to come later for WASI functions to access the file system, console / stdout, time / clock, and network requests.

### Printing and debugging from Rust

The Rust `println!` marco just works in WASI. The statements print to the `STDOUT` of the process that runs the SSVM. In Node.js apps, it is the `STDOUT` on the Node.js server.



