---
description: Give the function a context
---

# Context

## This page is outdated. Please [visit here for the most up-to-date content](https://www.secondstate.io/articles/why-webassembly-server/).

The first argument of any function must be a `&str`. You can use it to pass a `context` value to every function call. For example, we can set a `context` string for wasm ID `123`, which is the hello world wasm example we just created. The context specifies that the function should say hello in emoji.

```text
curl --location --request PUT 'https://rpc.ssvm.secondstate.io:8081/api/state/123' \
--header 'Content-Type: text/plain' \
--data-raw 'emoji'
```

The Rust function now looks like this.

```text
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub fn say(context: &str, s: &str) -> String {
  if context == "emoji" {
    let r = String::from("👋 ");
    return r + &s;
  } else {
    let r = String::from("hello ");
    return r + &s;
  }
}
```

The next time you run it, it says hello based on the context.

```text
$ curl --location --request POST 'https://rpc.ssvm.secondstate.io:8081/api/run/123/say' \
--header 'Content-Type: text/plain' \
--data-raw 'Second State FaaS'
```

The response is as follows.

```text
👋 Second State FaaS
```

We have just used a simple string as the context. For more complex use cases, the context could be a JSON object. You can set the state to the entire JSON string. The function is responsible for parsing the JSON string argument and makes sense of it.

You can also store the Second State VM storage ID in the stateful context. That enables stateful functions that can update and maintain its internal states across execution runs.  We will cover that use case in a later tutorial.



