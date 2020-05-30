---
description: Use JSON strings as function arguments
---

# JSON argument

## Strongly typed data structures
Rust is able to map a JSON string to a Rust object using what is known as a [strongly typed data structure](https://docs.serde.rs/serde_json/#parsing-json-as-strongly-typed-data-structures). The following Rust code defines the `Person` data before compile time.

```
struct Person {
    name: String,
    age: u8,
    phones: Vec<String>,
}
```
The Rust code can create a `Person` object, in a strongly typed fashion, using the following syntax.
```
let p: Person = serde_json::from_str(data)?;
```
In the case above, the JSON string used to create the object would be as follows.
```
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

## callback_url argument
It is possible for a function to initiate further processing via the use of a callback. This functionality is optional, but very useful and worth learning about.

