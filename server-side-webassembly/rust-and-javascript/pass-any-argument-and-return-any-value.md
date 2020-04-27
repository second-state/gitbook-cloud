---
description: Call any Rust function from JavaScript via JSON
---

# Passing any argument and return any value

While SSVM only supports a limited number of types as we discussed in the [previous article](supported-types.md), with JSON support, you can call Rust functions with any number of input parameters and return any number of return values of any type. That allows us to take advantage of any Rust libraries and crates in the ecosystem.

{% hint style="success" %}
The source code of the tutorial is [here](https://github.com/second-state/wasm-learning/tree/master/nodejs/json_io). 
{% endhint %}

#### **WebAssembly program in Rust**

In the `cargo` project called `json_io`, edit the `Cargo.toml` file to add a `[lib]` section and a `[dependencies]` section. Besides the `wasm-bindgen` dependency, notice the `serde` and `serde_json` dependencies. They allow us to serialize and deserialize complex Rust types to and from JSON strings, so that the data can be passed to and from JavaScript.

```text
[lib]
name = "json_io_lib"
path = "src/lib.rs"
crate-type =["cdylib"]

[dependencies]
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
wasm-bindgen = "0.2.59"
```

Below is the content of the Rust program `src/lib.rs`. It shows four functions.

* The `circumference()` function takes one floating point number parameter, and returns a floating number value. **Notice** that the floating number type is **not** natively supported in SSVM, but is supported here via JSON.
* The `area()` function takes two floating point numbers \(the width and length of a rectangle\) and returns a floating point number \(the area of the rectangle\).
* The `solve()` function takes three floating point numbers \(parameters of a quadratic equation\), and returns two floating point numbers and a boolean \(the roots or solutions to the equation and whether the equation has real roots\).
* The `draw()` function takes two structs \(`Point`\) and a string, and returns a struct \(`Line`\).

Inside each Rust function, we first deserialize the input JSON string into a tuple, which contains the call arguments of various types. The return values are constructed into a tuple first and then serialized into a JSON string and then returned.

```text
use wasm_bindgen::prelude::*;
use serde::{Serialize, Deserialize};

#[wasm_bindgen]
pub fn circumference(radius: &str) -> String {
  let r: f32 = serde_json::from_str(radius).unwrap();
  let c = 2. * 3.14159 * r;
  return serde_json::to_string(&c).unwrap();
}

#[wasm_bindgen]
pub fn area(sides: &str) -> String {
  let s: (f32, f32) = serde_json::from_str(&sides).unwrap();
  let a = s.0 * s.1;
  return serde_json::to_string(&a).unwrap();
}

#[wasm_bindgen]
pub fn solve(params: &str) -> String {
  let ps: (f32, f32, f32) = serde_json::from_str(&params).unwrap();
  let discriminant: f32 = (ps.1 * ps.1) - (4. * ps.0 * ps.2);
  let mut solution: (f32, f32, bool) = (0., 0., false);
  if discriminant >= 0. {
    solution.0 = (((-1.) * ps.1) + discriminant.sqrt()) / (2. * ps.0);
    solution.1 = (((-1.) * ps.1) - discriminant.sqrt()) / (2. * ps.0);
    solution.2 = true;
  }
  return serde_json::to_string(&solution).unwrap();
}
```

The `draw()` example is the same as the `draw_line()` example from the last article, but the input argument is structured into a single JSON tuple.

```text
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
pub fn draw(points: &str) -> String {
  let ps: (Point, Point, String) = serde_json::from_str(&points).unwrap();
  let length = ((ps.0.x - ps.1.x) * (ps.0.x - ps.1.x) + (ps.0.y - ps.1.y) * (ps.0.y - ps.1.y)).sqrt();

  let valid = if length == 0.0 { false } else { true };
  let line = Line { points: vec![ps.0, ps.1], valid: valid, length: length, desc: ps.2 };
  return serde_json::to_string(&line).unwrap();
}
```

Next, you can compile the Rust source code into WebAssembly bytecode and generate the accompanying JavaScript module for the Node.js host environment.

```text
$ ssvmup build
```

The result are files in the `pkg/` directory. the `.wasm` file is the WebAssembly bytecode program, and the `.js` files are for the JavaScript module.

#### **The Node.js host application**

Next, let’s create a node folder for the Node.js web application. Copy over the generated JavaScript module files.

```text
$ mkdir node
$ cp pkg/* node/
```

Below is the node application `app.js`. It shows how to call the Rust functions. You will first need to construct the call arguments into a JavaScript array \(tuple\), and then pass the serialized JSON string to the Rust function. The Rust return value is deserialized into a tuple of values as well.

```text
const { circumference, area, solve, draw } = require('./json_io_lib.js');

var x = 10.;
console.log( circumference(JSON.stringify(x)) );

var x = [10., 5.];
console.log( area(JSON.stringify(x)) );

var x = [2., 5., -3.];
console.log( solve(JSON.stringify(x)) );

var x = [{x:1.5, y:3.8}, {x:2.5, y:5.8}, "A thin red line"];
console.log( draw(JSON.stringify(x)) );
```

Run `app.js` in Node.js environment as follows.

```text
$ node app.js
62.831802
50.0
[0.5,-3.0,true]
{"points":[{"x":1.5,"y":3.8},{"x":2.5,"y":5.8}],"valid":true,"length":2.2360682,"desc":"A thin red line"}
```

#### **What’s next?**

With JSON support, we can call any Rust function from JavaScript. In the next several tutorials, we will put this into good use. The JavaScript host app will delegate computationally intensive tasks such as cryptography, machine learning, and artificial intelligence to Rust modules inside the SSVM WebAssembly runtime.

