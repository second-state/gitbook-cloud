---
description: Access high performance storage and databases from a Rust API
---

# The storage interface

The [SSVM storage interface](https://github.com/second-state/specs/blob/master/storage_interface.md) provides a Rust API that allows programs to persist arbitrary  data into a key value store. The data store is configured and started by the SSVM and hence is transparent to the Rust application. Rust developers can view this as an abstract storage space for application data. 

{% hint style="info" %}
The storage interface provides much higher performance and data throughput than using WASI calls to access database on the file system or via network. It is also much easier to work with than most database APIs in Rust.
{% endhint %}

### Dependency

You will need to include the [rust\_storage\_interface\_library](https://crates.io/crates/rust_storage_interface_library) crate in your `Cargo.toml`. In many cases, you will also need [serde](https://serde.rs/) dependency for storing and loading structs. Add dependency for [serialize\_deserialize\_u8\_i32](https://crates.io/crates/serialize_deserialize_u8_i32) if you wish to store `Vec<u8>` or `&[u8]` byte arrays.

```text
[dependencies]
rust_storage_interface_library = "^0.1"
serde = { version = "^1.0", features = ["derive"] }
serialize_deserialize_u8_i32 = "^0.1"
```

In your Rust code, do this.

```text
use serialize_deserialize_u8_i32::s_d_u8_i32;
use rust_storage_interface_library::ssvm_storage;
```

### Store and load primitive types

With a simple API call, you can store and load Rust data of primitive types.

```text
// store boolean
let boolean1: bool = true;
let storage_key: i32 = ssvm_storage::store::store(boolean1);
// load boolean
let boolean2: bool = ssvm_storage::load::load_as_bool(storage_key);

// store char
let char1: char = 'a';
let storage_key: i32 = ssvm_storage::store::store(char1);
// load char
let char2: char = ssvm_storage::load::load_as_char(storage_key);

// store i64
let i641: i64 = 1234567890;
let storage_key: i64 = ssvm_storage::store::store(i641);
// load i64
let i642: i64 = ssvm_storage::load::load_as_i64(storage_key);

// store f64
let f641: f64 = 3.1415926536;
let storage_key: i64 = ssvm_storage::store::store(f641);
// load i64
let f642: f64 = ssvm_storage::load::load_as_f64(storage_key);
```

### Store and load string data

Strings are similarly easy. With string capabilities, you can store and load complex JSON structures.

```text
// store a string
let my_string = String::from("A string to store");
let storage_key: i32 = ssvm_storage::store::store(my_string);
// load a string
let my_loaded_string = ssvm_storage::load::load_as_string(storage_key);
```

### Store and load structs data

With serde, it is easy to store a struct. Notice that, when you load the data, you will need to pass in an "empty" struct of the same type in order for the the compiler to know the returned struct type.

```text
// Define the struct
#[derive(Serialize, Deserialize, PartialEq, Debug, Default)]
struct TestStruct {
    a_vec: Vec<u8>,
    a_i32: i32,
    a_u8: u8,
    a_bool: bool,
}

// Store the struct
let test_struct1 = TestStruct {
    a_vec: vec![134, 122, 131],
    a_i32: 4,
    a_u8: 4,
    a_bool: true,
};
let storage_key: i32 = ssvm_storage::store::store(test_struct1);

// Load the struct
// Instantiate the struct, just using the default (no actual data necessary, this is just a placeholder to pass in so the call will return your data as the correct type)
let struct_skeleton = TestStruct::default();
let my_loaded_struct: TestStruct = ssvm_storage::load::load_as_struct(struct_skeleton, storage_key);
```

### Store and load binary data

Often times, it is easier just to store data in binary format as a byte array. Think file content, images, videos, and binary serialized structs etc. With the storage interface, we do that by packing and unpacking the byte array into `i32` arrays.

```text
// Store byte array Vec<u8>
let bytes : Vec<u8> = vec![134, 122, 131, 111];
let bytes_in_i32: Vec<i32> = s_d_u8_i32::serialize_u8_to_i32(bytes);
let storage_key: i32 = ssvm_storage::store::store_as_i32_vector(bytes_in_i32);

// Load Vec<u8>
let bytes_in_i32_1: Vec<i32> = ssvm_storage::load::load_as_i32_vector(storage_key);
let mut bytes_1: Vec<u8> = s_d_u8_i32::deserialize_i32_to_u8(bytes_in_i32_1);
```

For more examples and how it works under the hood, please review the [spec document](https://github.com/second-state/specs/blob/master/storage_interface.md).

