---
description: How to extend Deno with Rust and WebAssembly functions
---

# Deno, Rust and WebAssembly

## This page is outdated. Please [visit here for the most up-to-date content](https://www.secondstate.io/articles/why-webassembly-server/).

[Deno](https://deno.land/) is a JavaScript / Typescript runtime written in Rust. It is based on Google V8 engine \(same as Node.js\) and created by [Ryan Dahl](https://en.wikipedia.org/wiki/Ryan_Dahl) -- the creator and original developer of Node.js.

**Q: Is Deno going to replace Node.js?** 

A: It could. If it does, it will be one of the coming-to-age events for the Rust language for server-side applications. The same way Twitter made the Ruby language a legitimate choice for web apps almost 15 years ago.  

**Q: One of the great selling point of Deno is that it is written in Rust. Can I use Rust to enhance and extend Deno for my apps?** 

A: Yes. You can.

👉 [Use WebAssembly to run Rust functions in Deno](https://dev.to/lampewebdev/writing-webassembly-in-rust-and-runing-it-in-deno-144j)

👉 [Use the Deno feat native extension](https://github.com/denoland/deno/pull/3372) \(similar to Node.js NAPI\)

**Q: Are server-side WebAssembly runtimes, such as the** [**Second State VM**](https://cloud.secondstate.io/server-side-webassembly/getting-started)**, going to be available for Deno?**

A: Yes, we are working on it!

**Q: Can you run Deno in a serverless environment?**

A: Yes.

* AWS Lambda: [https://github.com/hayd/deno-lambda/tree/master/example](https://github.com/hayd/deno-lambda/tree/master/example)
* Azure Function: [https://deno.land/x/azure\_functions/](https://deno.land/x/azure_functions/)

**Q: Is there a Deno-based serverless environment to run high performance Rust functions as services?**

A: We are working on that!



