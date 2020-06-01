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

This strongly typed approach may be perfect if the data is defined up front and not likely to change. This strongly typed approach is also perfectly suitable for simple flat data \(which is not nested\).

If the JSON string being presented to the Rust application is likely to have extra or missing fields \(in some situations\) or is complex in nature \(contains several nested layers\) then please consider mapping the JSON string to a Rust object using an untyped data structure.

## Untyped data structures

Instead of writing complex nested Structs before compile time, you could use [serde\_json's generic Value type](https://docs.serde.rs/serde_json/value/enum.Value.html) as demonstrated in the following code. This approach allows for maximum flexiblility.

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

## Callback\_url argument

It is possible for a function to initiate further processing via the use of a callback. This functionality is optional, but very useful and worth learning about.

**Important notes:**

* Please ensure that the data for the callback object adheres to the standard shown below \(1\)
* Please ensure that the data for the callback object is wrapped in a single `callback` wrapper as shown below \(2\)
* Please ensure that the `callback` object is at the top level as shown below \(3\)

\(1\) Before we begin with an example, please note that it is important to adhere to the following request structure when creating your own callback object in your Rust / Wasm code.

```javascript
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

\(2\) Please note that over and above the example we have just used ... there **must be an additional `callback` object which wraps the standard request format**. Here is the complete example of what your callback argument should look like; note the `{"callback": {}}` wrapper.

```javascript
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

\(3\) Please note that the `callback` is a standalone object which is always at the top level \(along side other top level objects as required\)

```javascript
{
    "function": {
        "name": "new template name"
    },
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

Below is an example of a single HTTP POST request which can perform two different tasks by calling a second endpoint, using the data derived from the calculations at the first endpoint. Specifically, we find the average of two temperatures, and then send that result off for conversion from Celsius to Fahrenheit.

Consider the following Rust / Wasm code (`my_first_function`)

```rust
use serde_json;
use serde_json::json;
use serde_json::Value;

#[no_mangle]
fn my_first_function(_function_data: &str) -> String {
    let function_data_as_object: Value = serde_json::from_str(_function_data).unwrap();
    let answer: f64 = (function_data_as_object["first_function_input"]["left_temperature"]
        .as_f64()
        .unwrap()
        + function_data_as_object["first_function_input"]["right_temperature"]
            .as_f64()
            .unwrap())
        / 2.0;
    println!("average_temperature_as_celsius: {:?}", answer);
    let response_with_callback = json!({
    "callback": function_data_as_object["callback"],
    "first_function_output": {"average_temperature_as_celsius": answer}
    });
    response_with_callback.to_owned().to_string()
}
```
The above `my_first_function` can be called using the following JSON string input (notice the `callback` to `my_other_function` at `wasm_id` `2` that is being passed in as input)
```javascript
{
	"callback": {
		"method": "POST",
		"hostname": "rpc.ssvm.secondstate.io",
		"port": 8081,
		"path": "/api/run/2/my_other_function",
		"headers": {
			"Content-Type": "application/json"
		},
		"maxRedirects": 20
	},
	"first_function_input": {
		"left_temperature": 35,
		"right_temperature": 38
	}
}
```
The following is an example of the inputs and outputs that are derived as the entire request/response process unfolds.

A caller now issues the following HTTP request to `my_first_function` (the wasm executable at `wasm_id` `1`)
```bash
curl --location --request POST 'https://rpc.ssvm.secondstate.io:8081/api/run/1/my_first_function' \
--header 'Content-Type: application/json' \
--data-raw '{
	"callback": {
		"method": "POST",
		"hostname": "rpc.ssvm.secondstate.io",
		"port": 8081,
		"path": "/api/run/2/my_other_function",
		"headers": {
			"Content-Type": "application/json"
		},
		"maxRedirects": 20
	},
    "first_function_input": {
        "left_temperature": 35,
        "right_temperature": 38
    }
}'
```
As we can see in the Rust source code above, `my_first_function` finds the `average_temperature_as_celsius` (given the `left_temperature` and the `right_temperature` of the `first_function_input`) and in addition returns the `callback`. The result of the execution of `my_first_function` looks like this.
```Javascript
{
	"callback": {
		"headers": {
			"Content-Type": "application/json"
		},
		"hostname": "rpc.ssvm.secondstate.io",
		"maxRedirects": 20,
		"method": "POST",
		"path": "/api/run/2/my_other_function",
		"port": 8081
	},
	"first_function_output": {
		"average_temperature_as_celsius": 36.5
	}
}
```
You may be wondering why we passed the `callback` into `my_first_function` as an input (part of the JSON string), only to see it returned here, right? 

The reason for this is because it is not wise to hard code a callback (into your source code) because most of the time the callback's data would have some private credentials in it i.e. an API key or a password etc. It is safer to only pass the callback to `rpc.ssvm.secondstate.io` via HTTPS and let the callback execute securely inside the `POST` request.

Here is the latter part of the execution (the callback). Consider the following function `my_other_function` at `wasm_id` `2` which converts the `average_temperature_as_celsius` to fahrenheit.

```Rust
#[no_mangle]
fn my_other_function(_function_data:  &str) -> String {
    let function_data_as_object: Value = serde_json::from_str(_function_data).unwrap();
    let new_answer = function_data_as_object["first_function_output"]["average_temperature_as_celsius"].as_f64().unwrap() * (9.0/5.0) + 32.0;
    let response = json!({
    "other_function_output": {"average_temperature_as_fahrenheit": new_answer}
    });
    response.to_owned().to_string()
}
```
We have demonstrated here that each Wasm executable does return data, these multihop function executions (which use the callback object as input to the first callable function) facilitate all of the work to be handled internally. For example, if the caller issued the following HTTP POST request, they would simply get back one response which shows the `average_temperature_as_fahrenheit`. All of the other inner workings are not visible to the caller. 

```
curl --location --request POST 'https://rpc.ssvm.secondstate.io:8081/api/run/1/my_first_function' \
--header 'Content-Type: application/json' \
--data-raw '{
	"callback": {
		"method": "POST",
		"hostname": "rpc.ssvm.secondstate.io",
		"port": 8081,
		"path": "/api/run/2/my_other_function",
		"headers": {
			"Content-Type": "application/json"
		},
		"maxRedirects": 20
	},
    "first_function_input": {
        "left_temperature": 35,
        "right_temperature": 38
    }
}'
```

The above request, produces the following result.

```Javascript
{"other_function_output":{"average_temperature_as_fahrenheit":97.7}}
```


