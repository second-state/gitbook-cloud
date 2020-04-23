---
description: >-
  Seamless integration between the high performance Rust and easy-to-use
  JavaScript
---

# Rust and JavaScript

The role of WebAssembly is not \(yet\) to replace JavaScript, but to enhance JavaScript by taking over computationally intensive tasks. Similarly, it is not \(yet\) to replace native applications or libraries, but to provide a safe sandbox for native and user submitted code. In server side Node.js applications, that translates to functions written in Rust that can be called from a JavaScript host program.

{% hint style="info" %}
Before you start, make sure that you have gone through the [previous tutorial to set up a Node.js application in Rust](../getting-started/). You should have [`ssvmup`](https://www.npmjs.com/package/ssvmup) and [`ssvm`](https://www.npmjs.com/package/ssvm) NPM modules installed through the last tutorial.
{% endhint %}

In the next several articles, we will show more examples on how the Rust program in WebAssembly / SSVM can exchange data with the Node.js JavaScript host app. 

{% page-ref page="supported-types.md" %}

{% page-ref page="pass-any-argument-and-return-any-value.md" %}

{% page-ref page="call-javascript-functions-from-rust.md" %}

Later in this series, we will show more complex use cases. The JavaScript host app will delegate computationally intensive tasks such as cryptography, machine learning, and artificial intelligence to Rust modules inside the SSVM WebAssembly runtime.

