---
description: >-
  Use JavaScript modules to access the file system, network, databases, and
  other system resources
---

# Access JavaScript from Rust

## This page is outdated. Please [visit here for the most up-to-date content](https://www.secondstate.io/articles/why-webassembly-server/).

In this tutorial, we will show you how to use the [`nodejs-helper`](https://crates.io/crates/nodejs-helper) crate to call Node.js functions from Rust code. Rust functions can now access the file system, network, database, and other system resources from within the WebAssembly container.

{% hint style="info" %}
It is important to note that a better way for Rust programs to access system resources is through the [WebAssembly WASI extension](../server-side-webassembly/enterprise-webassembly/wasi.md), as well as numerous [host extensions provided by the SSVM](../server-side-webassembly/enterprise-webassembly/).
{% endhint %}

{% hint style="success" %}
The source code of the tutorial is [here](https://github.com/second-state/wasm-learning/tree/master/nodejs/nodejs_example).
{% endhint %}

Let's see an example Rust function that gets the system time and prints to the standard output console, all from within a WebAssembly container.

```text
#[wasm_bindgen]
pub fn utc_now() {
  let now: String = nodejs_helper::date::utc_string();
  nodejs_helper::console::log("UTC time: ");
  nodejs_helper::console::log(&now);
}
```

## Prerequisite

You must have Node.js installed with the following packages.

```text
$ npm i ssvm sync-request better-sqlite3
$ npm i -g wasm-pack
```

In your Rust application, add the following dependency.

```text
[dependencies]
wasm-bindgen = "=0.2.61"
nodejs-helper = "0.0.3"
```

## Building and running

Use the following command to build your Rust application for WebAssembly.

```text
$ wasm-pack build --target nodejs
```

Now, let's look some concrete examples from the [example project](https://github.com/second-state/wasm-learning/tree/master/nodejs/nodejs_example).

## Example: system time and console

The Rust functions to access the system time and console resources are as follows.

```text
#[wasm_bindgen]
pub fn show_now() {
  nodejs_helper::console::log("Timestamp now: ");
  nodejs_helper::console::log(&nodejs_helper::date::timestamp());
}

#[wasm_bindgen]
pub fn utc_now() {
  nodejs_helper::console::log("UTC time: ");
  nodejs_helper::console::log(&nodejs_helper::date::utc_string());
}

#[wasm_bindgen]
pub fn my_time(tz: &str) {
  nodejs_helper::console::log(tz);
  nodejs_helper::console::log(&nodejs_helper::date::format_date("en-US", "long", "numeric", "long", "numeric", tz, "short"));
}
```

The JavaScript code that loads the WebAssembly container and runs the above Rust functions is as follows.

```text
const { show_now, utc_now, my_time } = require('../pkg/nodejs_example.js');

show_now();
utc_now();
my_time("America/Chicago");
```

Running the Javascript in Node.js shows the following.

```text
$ node date.js
Timestamp now:
1588013800826
UTC time:
Mon, 27 Apr 2020 18:56:40 GMT
America/Chicago
Monday, April 27, 2020, CDT
```

## Example: Sqlite database access

The Rust functions to create, update, and query a Sqlite database on the local file system are as follows.

```text
#[derive(Serialize, Deserialize)]
pub struct User {
  pub id: u32,
  pub full_name: String,
  pub created: String,
}

#[wasm_bindgen]
pub fn create_sqlite(path: &str) {
  let sql_create = "
CREATE TABLE users (
  id INTEGER PRIMARY KEY NOT NULL,
  full_name TEXT NOT NULL,
  created DATE NOT NULL
);";
  let sql_insert = "
INSERT INTO users
VALUES
(1, 'Bob McFett', '32-01-01'),
(2, 'Angus Vader', '02-03-04'),
(3, 'Imperator Colin', '01-01-01');";

  nodejs_helper::sqlite3::create(path);
  nodejs_helper::sqlite3::update(path, sql_create);
  nodejs_helper::sqlite3::update(path, sql_insert);
}

#[wasm_bindgen]
pub fn query_sqlite(path: &str) {
  let sql_query = "SELECT * FROM users;";
  let rows: String = nodejs_helper::sqlite3::query(path, sql_query);
  let users: Vec<User> = serde_json::from_str(&rows).unwrap();
  for user in users.into_iter() {
    nodejs_helper::console::log(&(user.id.to_string() + " : " + &user.full_name));
  }
}
```

The JavaScript code that loads the WebAssembly container and runs the above Rust functions is as follows.

```text
const { create_sqlite, query_sqlite } = require('../pkg/nodejs_example.js');

create_sqlite("test.sqlite");
query_sqlite("test.sqlite");
```

Running the Javascript in Node.js shows the following.

```text
$ node db.js
1 : Bob McFett
2 : Angus Vader
3 : Imperator Colin
```

## Example: HTTP network access

The Rust functions to access web services via HTTP/HTTPS and then save content on the local file system are as follows.

```text
#[wasm_bindgen]
pub fn fetch(url: &str) {
  let content = nodejs_helper::request::fetch_as_string(url);
  nodejs_helper::console::log(url);
  nodejs_helper::console::log(&content);
}

#[wasm_bindgen]
pub fn download(url: &str, path: &str) {
  let content = nodejs_helper::request::fetch(url);
  nodejs_helper::fs::write_file_sync(path, &content);
}
```

The JavaScript code that loads the WebAssembly container and runs the above Rust functions is as follows.

```text
const { fetch, download } = require('../pkg/nodejs_example.js');

fetch("https://raw.githubusercontent.com/second-state/nodejs-helper/master/LICENSE");
download("https://www.secondstate.io/", "test.html");
```

Running the Javascript in Node.js shows the following.

```text
$ node http.js
https://raw.githubusercontent.com/second-state/nodejs-helper/master/LICENSE
MIT License

Copyright (c) 2020 Second State

Permission is hereby granted, free of charge, to any person obtaining a copy
... ...
```

## Example: File system access and performance profiler

The Rust functions in this section read an image file from the local file system, resize it, and write back to the file system. It also uses the Javascript console tool to measure the time spent on each task.

```text
#[derive(Serialize, Deserialize)]
#[derive(Copy, Clone, Debug)]
pub struct Dimension {
  pub width: u32,
  pub height: u32,
}

#[derive(Serialize, Deserialize)]
pub struct Picture {
  pub dim: Dimension,
  pub raw: Vec<u8>,
}

#[wasm_bindgen]
pub fn resize_file(input: &str) {
  // Use JSON to pass multiple call arguments
  let p: (Dimension, String, String) = serde_json::from_str(input).unwrap();

  nodejs_helper::console::time("Resize file");
  let raw = nodejs_helper::fs::read_file_sync(&p.1);
  nodejs_helper::console::time_log("Resize file", "Done reading");
  let src = Picture {
    dim: p.0,
    raw: raw,
  };
  let target = resize_impl(&src);
  nodejs_helper::console::time_log("Resize file", "Done resizing");

  nodejs_helper::fs::write_file_sync(&p.2, &target.raw);
  nodejs_helper::console::time_log("Resize file", "Done writing");
  nodejs_helper::console::time_end("Resize file");
}

pub fn resize_impl(src: &Picture) -> Picture {
  // ... use the img crate to resize ...
}
```

The JavaScript code that loads the WebAssembly container and runs the above Rust functions is as follows.

```text
const { resize_file } = require('../pkg/nodejs_example.js');

const dim = {
    width: 100,
    height: 100
};

resize_file(JSON.stringify([dim, 'cat.png', `test.png`]));
```

Running the Javascript in Node.js shows the following.

```text
$ node image.js
Resize file: 5.603ms Done reading
Resize file: 1506.694ms Done resizing
Resize file: 1507.634ms Done writing
Resize file: 1507.977ms
```

That's it for now. The [`nodejs-helper`](https://crates.io/crates/nodejs-helper) crate is still a work-in-progress. We aim to eventually provide Rust APIs for all common system functions here. You are welcome to [fork and add to it](https://github.com/second-state/nodejs-helper).

