---
description: Use k-means clustering algorithm to classify data points
---

# Machine learning

## This page is outdated. Please visit here to see the use case of[ Rust function in machine learning](https://www.secondstate.io/articles/machine-learning/).

The lingua franca of machine learning is Python. However, Python relies on C/C++ based native modules to perform the actual computationally intensive tasks of machine learning. It is similar to Node.js relying on C++ to perform computing tasks.

For new machine learning algorithms, developers can choose to implement them in Python for developer productivity or in C++ for runtime efficiency. Now, there is a third choice. Implementing machine learning algorithms in Rust could provide a 25x performance gain over Python as well as safety over C++. In this tutorial, we will demonstrate how to do k-means clustering computation in Rust, and make the function available in Node.js.

{% hint style="success" %}
The example source code for this tutorial is [here](https://github.com/second-state/wasm-learning/tree/master/nodejs/kmeans).
{% endhint %}

The Rust function `fit()` is as follows. It reads content from a CSV data file, and group the points into clusters based on the dimensions for the points and the number of estimated clusters.

```text
use wasm_bindgen::prelude::*;
use ndarray::{Array2};
use std::str::FromStr;

#[wasm_bindgen]
pub fn fit (csv_content: &[u8], dim: i32, num_clusters: i32) -> String {
    let data = read_data(csv_content, dim as usize);
    let (means, _clusters) = rkm::kmeans_lloyd(&data.view(), num_clusters as usize);
    return serde_json::to_string(&means).unwrap();
}

fn read_data(csv_content: &[u8], dim: usize) -> Array2<f32> {
    let mut data_reader = csv::Reader::from_reader(csv_content);
    let mut data: Vec<f32> = Vec::new();
    for record in data_reader.records() {
        for field in record.unwrap().iter() {
            let value = f32::from_str(field);
            data.push(value.unwrap());
        }
    }
    Array2::from_shape_vec((data.len() / dim, dim), data).unwrap()
}
```

The Javascript host application reads the CSV file, and calls the Rust function to perform the computation. The results are returned as a multi-dimensional array for the cluster centers.

```text
const { fit } = require('../pkg/kmeans_lib.js');
const fs = require('fs'); 

var birch3_csv = fs.readFileSync("birch3.data.csv");
var means = JSON.parse( fit(birch3_csv, 2, 100) );
console.log(means);
```

Rust and WebAssembly made it easy to make high performance machine learning algorithms available as web services.

