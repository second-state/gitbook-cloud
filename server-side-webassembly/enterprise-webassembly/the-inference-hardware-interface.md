---
description: Use AI hardware to accelerate inference operations in WebAssembly
---

# The inference interface

One of the key benefits of Rust is that it is close to the hardware and can fully take advantage of new hardware features. That is most interesting in the area of Artificial Intelligence \(AI\), where almost all major cloud players now have their own customized silicon chips for AI inference.

![](../../.gitbook/assets/screen-shot-2020-05-20-at-1.21.30-am.png)

The SSVM inference interface enables Rust applications to directly drive natively compiled ONNX and Tensorflow models on those new hardware. We do that through Rust and WebAssembly code instead of C++ native code for better safety, portability, and manageability.

