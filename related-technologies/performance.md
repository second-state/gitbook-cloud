---
description: Performance benchmark for the SSVM against other WebAssembly runtimes
---

# SSVM Performance

We use the Second State Virtual Machine \(SSVM\) , an open source WebAssembly runtime optimized for server-side applications, together with Node.js. The SSVM provides not only a WebAssembly runtime in Node.js, but also a compiler toolchain for Rust and JavaScript.

> While Node.js comes with a default WebAssmebly runtime inside its V8 JavaScript engine, V8 is not designed to handle the performance and complex integration requirements of server-side applications. Compared with V8, the server-optimized SSVM is more performant, integrates better with JavaScript, provides access to external enterprise resources, and supports finely grained metering.

#### Performance benchmark

{% hint style="info" %}
This benchmark runs on Intel\(R\) Xeon\(R\) CPU E5-2673 v4 @ 2.30GHz, Linux 5.3.0-1016-azure. The unit of the following numbers are in seconds.
{% endhint %}

|  | SSVM❤️ | Lucet | WAVM | V8 | docker+native |
| :--- | :--- | :--- | :--- | :--- | :--- |
| nop 0 | 0.003👍 | 0.002👍 | 0.024 | 0.056 | 0.849 |
| cat-sync 0 | 0.007👍 | 0.573 | 0.029👍 | 0.06 | 0.826 |
| nbody-c 50000000 | 3.716👍 | 4.611 | 3.753 | 3.408👍 | 4.128 |
| nbody-cpp 50000000 | 3.759👍 | 4.705 | 3.741👍 | 3.962 | 3.944 |
| fannkuch-redux-c 12 | 28.06👍 | 53.104 | 28.477 | 29.285 | 24.459👍 |
| mandelbrot-c 15000 | 10.347👍 | 28.97 | 12.072👍 | 18.062 | 16.05 |
| binary-trees-c 18 | 1.328👍 | 2.91 | 1.612👍 | 2.002 | 17.191 |

