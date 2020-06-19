---
description: How to access resources outside of the WebAssembly sandbox
---

# Access system resources

## This page is outdated. Please [visit here for the most up-to-date content](https://www.secondstate.io/articles/why-webassembly-server/).

The WebAssembly VM provides a sandbox to ensure application safety. However, this sandbox is also a very limited "computer" that has no concept of file system, network, or even a clock or timer. That is very limiting for the Rust programs running inside WebAssembly.

In the Second State VM \(SSVM\), our innovation is a set of standard and proprietary \(still open source\) extensions that allow WebAssembly bytecode applications to access system and external resources. Read on!

{% page-ref page="wasi.md" %}

{% page-ref page="the-storage-interface.md" %}

{% page-ref page="the-inference-hardware-interface.md" %}



