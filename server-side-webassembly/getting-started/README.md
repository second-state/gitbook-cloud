---
description: High-performance Rust functions in Node.js
---

# Getting started

There are great use cases for [WebAssembly on the server-side](../why/), especially for AI, blockchain, and big data applications. In this tutorial, I will show you how to incorporate WebAssembly functions, written in Rust, into Node.js applications on the server. This approach combines Rust's _**performance**_, WebAssembly's _**security and portability**_, and JavaScript's _**ease-of-use**_. A typical Rust + Node.js hybrid app works like this.

* The host application is a Node.js web application written in JavaScript. It makes WebAssembly function calls.
* The WebAssembly bytecode program is written in Rust. It runs inside the SSVM, and is called from the Node.js web application.

{% hint style="success" %}
The source code of the tutorial is [here](https://github.com/second-state/wasm-learning/tree/master/nodejs/hello). If you just want to try it out, you can [fork this repository](https://github.com/second-state/ssvm-nodejs-starter/fork) and [use the VSCode IDE to open it](the-no-software-approach.md).
{% endhint %}

## Prerequisites

Since we are building Rust functions to run in Node.js, make sure that you have [Rust](https://www.rust-lang.org/tools/install) and [Node.js](https://nodejs.org/en/download/package-manager/) installed on your computer.

## **Setup**

> We use the Second State Virtual Machine \(SSVM\) , an open source WebAssembly runtime [optimized for server-side applications](../performance.md), together with Node.js.

The [ssvm](https://www.npmjs.com/package/ssvm) and [ssvmup](https://www.npmjs.com/package/ssvmup) npm modules install the [Second State Virtual Machine \(SSVM\)](https://github.com/second-state/ssvm) into Node.js as a native addon, and provides the necessary compiler tools. [Learn more](the-ssvmup-tool.md) about the [ssvmup](https://github.com/second-state/ssvmup) tool.

```text
# Install ssvmup toolchain
$ npm install -g ssvmup # Append --unsafe-perm if permission denied

# Install the nodejs addon for SSVM
$ npm install ssvm
```

## **WebAssembly program in Rust**

In this example, our Rust program appends the input string after “hello”. Below is the content of the Rust program [`src/lib.rs`](https://github.com/second-state/ssvm-nodejs-starter/blob/master/src/lib.rs). You can define multiple external functions in this library file, and all of them will be available to the host JavaScript app via WebAssembly. Just remember to annotate each function with `#[wasm_bindgen]` so that [ssvmup](https://github.com/second-state/ssvmup) knows to generate the correct JavaScript to Rust interface for it when you build it.

```text
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub fn say(s: String) -> String {
  let r = String::from("hello ");
  return r + &s;
}
```

Next, you can compile the Rust source code into WebAssembly bytecode and generate the accompanying JavaScript module for the Node.js host environment.

```text
$ ssvmup build
```

The result are files in the `pkg/` directory. the `.wasm` file is the WebAssembly bytecode program, and the `.js` files are for the JavaScript module.

## **The Node.js host application**

Next, go to the `node` folder and examine the JavaScript program [`app.js`](https://github.com/second-state/ssvm-nodejs-starter/blob/master/node/app.js). With the generated `hello_lib.js` module, it is very easy to write JavaScript to call WebAssembly functions. Below is the node application `app.js`. It simply imports the `say()` function from the generated module. The node application takes the `name` parameter from incoming an HTTP GET request, and responds with “hello `name`”.

```text
const { say } = require('../pkg/hello_lib.js');

const http = require('http');
const url = require('url');
const hostname = '127.0.0.1';
const port = 8080;

const server = http.createServer((req, res) => {
  const queryObject = url.parse(req.url,true).query;
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end(say(queryObject['name']));
});

server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});
```

Start the Node.js application server as follows.

```text
$ node app.js
Server running at http://127.0.0.1:8080/
```

Then, you can test it.

```text
$ curl http://127.0.0.1:8080/?name=Wasm
hello Wasm
```

## **More complex examples**

Besides passing string values between Rust and JavaScript, the ssvmup tool supports the following data types.

* Rust call parameters can be any combo of `i32`, `String`, `&str`, `Vec<u8>`, and `&[u8]`
* Return value can be `i32` or `String` or `Vec<u8>`
* For complex data types, such as structs, you could use JSON strings to pass data. 

{% hint style="info" %}
With JSON support, you can [call Rust functions with any number of input parameters and return any number of return values of any type](../pass-any-argument-and-return-any-value.md).
{% endhint %}

The Rust program [`src/lib.rs`](https://github.com/second-state/wasm-learning/blob/master/nodejs/functions/src/lib.rs) in the [functions example](https://github.com/second-state/wasm-learning/tree/master/nodejs/functions) demonstrates  how to pass in call arguments in various supported types, and return values.

```text
#[wasm_bindgen]
pub fn obfusticate(s: String) -> String {
  (&s).chars().map(|c| {
    match c {
      'A' ..= 'M' | 'a' ..= 'm' => ((c as u8) + 13) as char,
      'N' ..= 'Z' | 'n' ..= 'z' => ((c as u8) - 13) as char,
      _ => c
    }
  }).collect()
}

#[wasm_bindgen]
pub fn lowest_common_denominator(a: i32, b: i32) -> i32 {
  let r = lcm(a, b);
  return r;
}

#[wasm_bindgen]
pub fn sha3_digest(v: Vec<u8>) -> Vec<u8> {
  return Sha3_256::digest(&v).as_slice().to_vec();
}

#[wasm_bindgen]
pub fn keccak_digest(s: &[u8]) -> Vec<u8> {
  return Keccak256::digest(s).as_slice().to_vec();
}
```

Perhaps the most interesting is the `create_line()` function. It takes two JSON strings, each representing a `Point` struct, and returns a JSON string representing a `Line` struct. Notice that both the `Point` and `Line` structs are annotated with `Serialize` and `Deserialize` so that the Rust compiler automatically generates necessary code to support their conversion to and from JSON strings.

```text
use wasm_bindgen::prelude::*;
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
struct Point {
  x: f32, 
  y: f32
}

#[derive(Serialize, Deserialize, Debug)]
struct Line {
  points: Vec<Point>,
  valid: bool,
  length: f32,
  desc: String
}

#[wasm_bindgen]
pub fn create_line (p1: &str, p2: &str, desc: &str) -> String {
  let point1: Point = serde_json::from_str(p1).unwrap();
  let point2: Point = serde_json::from_str(p2).unwrap();
  let length = ((point1.x - point2.x) * (point1.x - point2.x) + (point1.y - point2.y) * (point1.y - point2.y)).sqrt();

  let valid = if length == 0.0 { false } else { true };
  let line = Line { points: vec![point1, point2], valid: valid, length: length, desc: desc.to_string() };
  return serde_json::to_string(&line).unwrap();
}

#[wasm_bindgen]
pub fn say(s: &str) -> String {
  let r = String::from("hello ");
  return r + s;
}
```

Next, let's examine the JavaScript program [`app.js`](https://github.com/second-state/wasm-learning/blob/master/nodejs/functions/node/app.js). It shows how to call the Rust functions. As you can see `String` and `&str` are simply strings in JavaScript, `i32` are numbers, and `Vec<u8>` or `&[8]` are JavaScript `Uint8Array`. JavaScript objects need to go through `JSON.stringify()` or `JSON.parse()` before being passed into or returned from Rust functions.

```text
const { say, obfusticate, lowest_common_denominator, sha3_digest, keccak_digest, create_line } = require('./functions_lib.js');

var util = require('util');
const encoder = new util.TextEncoder();
console.hex = (d) => console.log((Object(d).buffer instanceof ArrayBuffer ? new Uint8Array(d.buffer) : typeof d === 'string' ? (new util.TextEncoder('utf-8')).encode(d) : new Uint8ClampedArray(d)).reduce((p, c, i, a) => p + (i % 16 === 0 ? i.toString(16).padStart(6, 0) + '  ' : ' ') + c.toString(16).padStart(2, 0) + (i === a.length - 1 || i % 16 === 15 ?  ' '.repeat((15 - i % 16) * 3) + Array.from(a).splice(i - i % 16, 16).reduce((r, v) => r + (v > 31 && v < 127 || v > 159 ? String.fromCharCode(v) : '.'), '  ') + '\n' : ''), ''));

console.log( say("SSVM") );
console.log( obfusticate("A quick brown fox jumps over the lazy dog") );
console.log( lowest_common_denominator(123, 2) );
console.hex( sha3_digest(encoder.encode("This is an important message")) );
console.hex( keccak_digest(encoder.encode("This is an important message")) );

var p1 = {x:1.5, y:3.8};
var p2 = {x:2.5, y:5.8};
var line = JSON.parse(create_line(JSON.stringify(p1), JSON.stringify(p2), "A thin red line"));
console.log( line );
```

After running ssvmup to build the Rust library, running `app.js` in Node.js environment produces the following output.

```text
$ ssvmup build
... Building the wasm file and JS shim file in pkg/ ...

$ node app.js
hello SSVM
N dhvpx oebja sbk whzcf bire gur ynml qbt
246
000000  57 1b e7 d1 bd 69 fb 31 9f 0a d3 fa 0f 9f 9a b5  W.çÑ½iû1..Óú...µ
000010  2b da 1a 8d 38 c7 19 2d 3c 0a 14 a3 36 d3 c3 cb  +Ú..8Ç.-<..£6ÓÃË

000000  7e c2 f1 c8 97 74 e3 21 d8 63 9f 16 6b 03 b1 a9  ~ÂñÈ.tã!Øc..k.±©
000010  d8 bf 72 9c ae c1 20 9f f6 e4 f5 85 34 4b 37 1b  Ø¿r.®Á .öäõ.4K7.

{ points: [ { x: 1.5, y: 3.8 }, { x: 2.5, y: 5.8 } ],
  valid: true,
  length: 2.2360682,
  desc: 'A thin red line' }
```

## **What’s next?**

Now we have seen a very simple example to call a Rust function from JavaScript in a Node.js application. [In the next article,](../pass-any-argument-and-return-any-value.md) we will discuss how to pass arbitrary arguments from a JavaScript program to Rust.

