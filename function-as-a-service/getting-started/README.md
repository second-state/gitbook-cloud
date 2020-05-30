---
description: Write and deploy Rust functions as web services
---

# Getting started

The Second State FaaS service \(currently in public beta\) enables you to write Rust functions, and make them available as RESTful web services. Key features:

* Each Rust function is a RESTful endpoint
* Input arguments can be supplied via the HTTP request or from another URL
* Return values can be in the HTTP response body or redirected to another URL
* Stateful execution
* Finely-grained resource metering \(based on Opscode\)
* Much faster and lighter compared with Docker
* No-wait code start
* Access to native OS and system features
* Access to customized hardware \(e.g., AI inference chips\)
* Works across multiple clouds

## **Setup**

First, let's install Rust and Node.js on the dev computer. Node.js is needed for our toolchain. If you have already done it, you can skip these steps.

```text
# Prerequisite
$ sudo apt-get update
$ sudo apt install -y build-essential curl wget git vim libboost-all-dev

# Install rust
$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
$ source $HOME/.cargo/env

# Install nvm
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash
# Follow the on-screen instructions to logout and then log in

# Install node
$ nvm install v10.19.0
$ nvm use v10.19.0
```

The `ssvmup` npm module installs the Second State Virtual Machine \(SSVM\) into Node.js as a native `addon`, and provides the necessary compiler tools. Follow the steps below to install Rust and the `ssvmup` tool.

```text
# Install ssvmup toolchain
$ npm install -g ssvmup # Append --unsafe-perm if permission denied

# Install the nodejs addon for SSVM
$ npm install ssvm
```

## **WebAssembly program in Rust**

In this example, our Rust program appends the input string after “hello”. Let’s create a new `cargo` project. Since this program is intended to be called from a host application, not to run as a stand-alone executable, we will create a `hello` project.

```text
$ cargo new --lib hello
$ cd hello
```

Edit the `Cargo.toml` file to add a `[lib]` section. It tells the compiler where to find the source code for the library and how to generate the bytecode output. We also need to add a dependency of `wasm-bindgen` here. It is the utility `ssvmup` uses to generate the JavaScript binding for the Rust WebAssembly program, which is required by the FaaS runtime.

```text
[lib]
name = "hello_lib"
path = "src/lib.rs"
crate-type =["cdylib"]

[dependencies]
wasm-bindgen = "=0.2.61"
```

Below is the content of the Rust program `src/lib.rs`. You can see that it takes two input parameters. Let's not worry about the `context` at this moment. The function parameter `s` comes from the HTTP request when a user calls this function over the web.

```text
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub fn say(context: &str, s: &str) -> String {
  let r = String::from("hello ");
  return r + &s;
}
```

Next, you can compile the Rust source code into WebAssembly bytecode.

```text
$ ssvmup build --nowasi
```

The result are files in the `pkg/` directory. the `.wasm` file is the WebAssembly bytecode program.

## Upload the wasm file to FaaS

Use the following `curl` command to upload the `wasm` file to the FaaS service. In the beta stage, it is all FREE!

```text
$ curl --location --request POST 'https://rpc.ssvm.secondstate.io:8081/api/executables' \
--header 'Content-Type: application/octet-stream' \
--header 'SSVM-Description: say hello' \
--data-binary 'pkg/hello_lib_bg.wasm'
```

It returns an ID for the `wasm` file in the FaaS system.

```text
{"wasm_id":123}
```

## Run the function

Use the following `curl` command to run the `say()` function in the wasm program. The argument `s` for this function call is passed in as a string in the HTTP request body.

```text
$ curl --location --request POST 'https://rpc.ssvm.secondstate.io:8081/api/run/123/say' \
--header 'Content-Type: text/plain' \
--data-raw 'Second State FaaS'
```

The HTTP response body is as follows.

```text
hello Second State FaaS
```

## What's next

In the next article, we will learn how to give the Rust function a persistence context to customize its runtime behavior. That is where the first function parameter `context` comes into play.

