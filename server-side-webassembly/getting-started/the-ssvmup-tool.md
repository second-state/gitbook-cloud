---
description: The compiler toolchain for Rust functions in JavaScript
---

# The ssvmup tool

Throughout our examples, we make extensive use of the [ssvmup](https://github.com/second-state/ssvmup) tool. It is inspired by the wasm-pack project but is optimized for server-side applications. Specifically, it supports the [SSVM](https://github.com/second-state/ssvm) WebAssembly virtual machine and Deno host runtime.

The [ssvmup](https://github.com/second-state/ssvmup) uses `wasm-bindgen` to automatically generate the “glue” code between JavaScript and Rust source code so that they can communicate using their native data types. Without it, the function arguments and return values would be limited to very simple types \(i.e., 32-bit integers\) supported natively by WebAssembly. For example, [strings or arrays would not be possible](https://medium.com/wasm/strings-in-webassembly-wasm-57a05c1ea333) without [ssvmup](https://github.com/second-state/ssvmup) and `wasm-bindgen`.

The easiest way to install [ssvmup](https://github.com/second-state/ssvmup) is through [NPM](https://www.npmjs.com/package/ssvmup).

```text
$ npm install -g ssvmup # Append --unsafe-perm if permission denied
```

You could also install [ssvmup](https://github.com/second-state/ssvmup) as a standalone tool for runtimes such as Deno. You need to have Rust installed before running the command below.

```text
$ curl https://raw.githubusercontent.com/second-state/ssvmup/master/installer/init.sh -sSf | sh
```

Next, learn how to use ssvmup to build Rust functions for Node.js and Deno applications.

