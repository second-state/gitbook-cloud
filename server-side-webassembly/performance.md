---
description: Performance benchmark for the SSVM against other WebAssembly runtimes
---

# SSVM Performance

The Second State Virtual Machine \([SSVM](https://github.com/second-state/ssvm)\)  is an open source WebAssembly runtime optimized for server-side applications. The SSVM provides not only a WebAssembly runtime in Node.js, but also a compiler toolchain [ssvmup](https://github.com/second-state/ssvmup) for Rust and JavaScript.

#### Performance benchmarks

{% hint style="info" %}
The benchmark scores are in seconds. The smaller the better. The 👍emoji marks the two best performing runtimes for each benchmark. The docker+native runtime is a simple Ubuntu Docker on an Ubuntu host.
{% endhint %}

|  | [SSVM](https://github.com/second-state/SSVM)❤️ | [Lucet](https://github.com/bytecodealliance/lucet) / [wasmtime](https://github.com/bytecodealliance/wasmtime) | [WAVM](https://github.com/WAVM/WAVM) | [V8](https://github.com/v8/v8) | docker+native |
| :--- | :--- | :--- | :--- | :--- | :--- |
| nop 0 | 0.003👍 | 0.002👍 | 0.024 | 0.056 | 0.849 |
| cat-sync 0 | 0.007👍 | 0.573 | 0.029👍 | 0.06 | 0.826 |
| nbody-c 50M | 3.716👍 | 4.611 | 3.753 | 3.408👍 | 4.128 |
| nbody-cpp 50M | 3.759👍 | 4.705 | 3.741👍 | 3.962 | 3.944 |
| fannkuch-redux-c 12 | 28.06👍 | 53.104 | 28.477 | 29.285 | 24.459👍 |
| mandelbrot-c 15K | 10.347👍 | 28.97 | 12.072👍 | 18.062 | 16.05 |
| binary-trees-c 18 | 1.328👍 | 2.91 | 1.612👍 | 2.002 | 17.191 |

