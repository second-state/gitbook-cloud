---
description: Use RSA public key algorithms to encrypt and decrypt
---

# Encryption and decryption

## This page is outdated. Please visit here to see the use case of [Rust function in encryption and decryption](https://www.secondstate.io/articles/encryption-and-decryption/).

One of the frequently performed computing tasks is public key encryption and decryption. Rust and C++ code vastly outperforms JavaScript code in these tasks. In this tutorial, let's use [pure Rust implementation of the RSA algorithm](https://crates.io/crates/rsa) as an example to show how to perform public key encryption and decryption in a Node.js web service.

{% hint style="success" %}
The example project source code is [here](https://github.com/second-state/rust-wasm-ai-demo).
{% endhint %}

The following Rust functions perform the encryption and decryption tasks.

* The `generate_key_pair()` function creates a random public / private key pair of specified length. The generated `RSAKeyPair` is serialized into a JSON string and returned to the JavaScript caller.
* The `encrypt()` function takes a `RSAPublicKey` in serialized JSON format, and a byte array message. It encrypts the message and returns the result as a byte array.
* The `decrypt()` function takes a `RSAPrivateKey` in serialized JSON format, and an encrypted byte array message. It decrypts the message and returns the result as a byte array.

```text
use wasm_bindgen::prelude::*;
use rsa::{PublicKey, RSAPublicKey, RSAPrivateKey, PaddingScheme};
use rand::rngs::OsRng;
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize)]
struct RSAKeyPair {
  rsa_private_key: RSAPrivateKey,
  rsa_public_key: RSAPublicKey
}


#[wasm_bindgen]
pub fn generate_key_pair (bits: i32) -> String {
  let mut rng = OsRng;
  let private_key = RSAPrivateKey::new(&mut rng, bits as usize).expect("failed to generate a key");
  let public_key = private_key.to_public_key();
  let key_pair = RSAKeyPair {rsa_private_key: private_key, rsa_public_key: public_key};
  return serde_json::to_string(&key_pair).unwrap();
}

#[wasm_bindgen]
pub fn decrypt (pk: &str, data: &[u8]) -> Vec<u8> {
  let private_key: RSAPrivateKey = serde_json::from_str(pk).unwrap();
  return private_key.decrypt(PaddingScheme::PKCS1v15, data).expect("failed to decrypt");
}

#[wasm_bindgen]
pub fn encrypt (pk: &str, data: &[u8]) -> Vec<u8> {
  let mut rng = OsRng;
  let public_key: RSAPublicKey = serde_json::from_str(pk).unwrap();
  return public_key.encrypt(&mut rng, PaddingScheme::PKCS1v15, data).expect("failed to encrypt");
}
```

The Javascript host application calls the Rust functions as follows. It first calls the Rust `generate_key_pair()` function to generate the key pair, and then saves the generated public and private keys respectively. It then uses the public key to encrypt a string message, and then use the private key to decrypt that message. The keys are serialized into JSON strings before passing to the Rust functions.

```text
const { generate_key_pair, encrypt, decrypt } = require('../pkg/rsa_example_lib.js');

var kp = JSON.parse(generate_key_pair(2048));
var public_key = kp['rsa_public_key'];
var private_key = kp['rsa_private_key'];

var msg = "The Times 03/Jan/2009 Chancellor on brink of second bailout for banks";
var enc_data = encrypt(JSON.stringify(public_key), encoder.encode(msg));
var dec_data = decrypt(JSON.stringify(private_key), enc_data);
console.log(decoder.decode(dec_data));
```

The RSA example is simple but provides substantial performance benefits when you run many public key encryption and decryption operations.

For a more complex example of public key encryption and decryption, please see our [Recrypt-as-a-Service](https://github.com/second-state/recrypt-as-a-service) repo. It is a scalable approach for individuals to control access to private data without shared secrets or storing secrets \(e.g., private keys\) on a centralized service. As a part of the workflow, the web service needs to perform large amounts of proxy encryption using public keys. Rust and WebAssembly are ideally suited for this.

