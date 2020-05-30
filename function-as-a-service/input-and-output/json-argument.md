---
description: Use JSON strings as function arguments
---

# JSON argument

## Strongly typed data structures
Rust is able to map a JSON string to a Rust object using what is known as a [strongly typed data structure](https://docs.serde.rs/serde_json/#parsing-json-as-strongly-typed-data-structures). The following Rust code defines the `Person` data before compile time.

```rust
struct Person {
    name: String,
    age: u8,
    phones: Vec<String>,
}
```
The Rust code can create a `Person` object, in a strongly typed fashion, using the following syntax.
```rust
let p: Person = serde_json::from_str(data)?;
```
In the case above, the JSON string used to create the object would be as follows.
```rust
{
	"name": "John Doe",
	"age": 43,
	"phones": [
		"+44 1234567",
		"+44 2345678"
	]
}
```
This strongly typed approach may be perfect if the data is defined up front and not likely to change.
This strongly typed approach is also perfectly suitable for simple flat data (which is not nested).

If the JSON string being presented to the Rust application is likely to have extra or missing fields (in some situations) or is complex in nature (contains several nested layers) then please consider mapping the JSON string to a Rust object using an untyped data structure.

## Untyped data structures

Instead of writing complex nested Structs before compile time, you could use [serde_json's generic Value type](https://docs.serde.rs/serde_json/value/enum.Value.html) as demonstrated in the following code. This approach allows for maximum flexiblility.
```rust
use serde_json;
use serde_json::{Value};

#[no_mangle]
fn process(s: &str){
    let json_as_object: Value = serde_json::from_str(s).unwrap();
}
```
The above approach allows the string to be parsed, regardless of its structural complexity. Once parsed, you can simply access each of the keys and values using syntax like the following.
```rust
json_as_object["outer_object"]["middle_nested"]["inner_nested"]["value_to_use_in_app"]
```

## Callback_url argument
It is possible for a function to initiate further processing via the use of a callback. This functionality is optional, but very useful and worth learning about.

**Important note:**
Before we begin with an example, please note that it is important to adhere to the following request structure when creating your own callback object in your Rust / Wasm code.
```
{
    "method": "POST",
    "hostname": "rpc.ssvm.secondstate.io",
    "port": 8081,
    "path": "/api/run/1/my_function",
    "headers": {
      "Content-Type": "application/json"
    },
    "maxRedirects": 20
}
```
This is the standard request format which Javascript/Nodejs uses. It will be passed straight into a request programatically and therefore the structure and the key:value entries must conform to the standard [outlined here](https://nodejs.org/api/https.html#https_https_request_options_callback).

Please note that over and above the example we have just used ... there **must be an additional `callback` object which wraps the standard request format**. Here is the complete example of what your callback argument should look like; note the `{"callback": {}}` wrapper.
```Javascript
{
	"callback": {
		"method": "POST",
		"hostname": "rpc.ssvm.secondstate.io",
		"port": 8081,
		"path": "/api/run/1/my_function",
		"headers": {
			"Content-Type": "application/json"
		},
		"maxRedirects": 20
	}
}
```

### Callback example
In some cases the results of a specific function's output may be used for another function's input. This function as a service infrastructure allows a callback_url argument to be passed into a function, along with the function's other arguments.  

Consider the following Rust code
```rust
use serde_json;
use serde_json::json;
use serde_json::Value;

fn process<'a>(_callback_data: &'a str, _function_data:  &'a str) -> &'a str {
    let callback_data_as_object: Value = serde_json::from_str(_callback_data).unwrap();
    let function_data_as_object: Value = serde_json::from_str(_function_data).unwrap();
    let answer: u64 = function_data_as_object["left_value"].as_u64().unwrap() + function_data_as_object["right_value"].as_u64().unwrap();
    println!("Answer: {:?}", answer);
    let response_with_callback = json!({
    "callback_data": _callback_data,
    "function_data": {"answer": answer}
    });
    &response_with_callback.to_string()
}
fn main() {
    let callback_data_to_use = json!({
        "method": "POST",
        "hostname": "rpc.ssvm.secondstate.io",
        "port": 8081,
        "path": "/api/run/1/my_function",
        "headers": {
          "Content-Type": "application/json"
        },
      "maxRedirects": 20
    });

    let function_data_to_use = json!({
        "left_value": 1,
        "right_value": 1,
    });

    let returned_string = process(
        &callback_data_to_use.to_string(),
        &function_data_to_use.to_string(),
    );
}

```

