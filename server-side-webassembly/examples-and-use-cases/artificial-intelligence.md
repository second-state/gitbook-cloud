---
description: Image recognition using Tensorflow
---

# Artificial intelligence

## This page is outdated. Please Please visit here to see the use case of [Rust function in AI](https://www.secondstate.io/articles/artificial-intelligence/).

This example shows how to write a Rust function for image recognition, and then offer this function as AI-as-a-Service.

{% embed url="https://www.youtube.com/watch?v=Ce2am-ugQhg" %}

Using machine learning libraries in Rust, such as the [Tract](https://github.com/snipsco/tract) crate which supports both Tensorflow and ONNX inference model, we can write AI-as-a-Service functions in Node.js. The functions could take AI models and input data, and return inference results, such as recognized objects on an input image, through a web service.

{% hint style="success" %}
The example project source code is [here](https://github.com/second-state/rust-wasm-ai-demo).
{% endhint %}

The following Rust function does the inference.

* The `infer()` function takes raw bytes for an already-trained Tensorflow model from ImageNet, and an input image.
* The `infer_impl()` function resizes the image, applies the model to it, and returns the top matched label and probability. The label indicates an object the ImageNet model has been trained to recognize.

```text
use wasm_bindgen::prelude::*;
use tract_tensorflow::prelude::*;
use std::io::Cursor;

#[wasm_bindgen]
pub fn infer(model_data: &[u8], image_data: &[u8]) -> String {
    let res: (f32, u32) = infer_impl (model_data, image_data, 224, 224).unwrap();
    return serde_json::to_string(&res).unwrap();
}

fn infer_impl (model_data: &[u8], image_data: &[u8], image_height: usize, image_width: usize) -> TractResult<(f32, u32)> {
    // load the model
    let mut model_data_mut = Cursor::new(model_data);
    let mut model = tract_tensorflow::tensorflow().model_for_read(&mut model_data_mut)?;
    model.set_input_fact(0, InferenceFact::dt_shape(f32::datum_type(), tvec!(1, image_height, image_width, 3)))?;
    // optimize the model and get an execution plan
    let model = model.into_optimized()?;
    let plan = SimplePlan::new(&model)?;
    
    // open image, resize it and make a Tensor out of it
    let image = image::load_from_memory(image_data).unwrap().to_rgb();
    let resized = image::imageops::resize(&image, image_height as u32, image_width as u32, ::image::imageops::FilterType::Triangle);
    let image: Tensor = tract_ndarray::Array4::from_shape_fn((1, image_height, image_width, 3), |(_, y, x, c)| {
        resized[(x as _, y as _)][c] as f32 / 255.0
    })
    .into();
    
    // run the plan on the input
    let result = plan.run(tvec!(image))?;
    
    // find and display the max value with its index
    let best = result[0]
        .to_array_view::<f32>()?
        .iter()
        .cloned()
        .zip(1..)
        .max_by(|a, b| a.0.partial_cmp(&b.0).unwrap());
    match best {
        Some(t) => Ok(t),
        None => Ok((0.0, 0)),
    }
}
```

The Javascript function reads the model and image files, and calls the Rust function.

```text
const { infer } = require('../pkg/csdn_ai_demo_lib.js');

const fs = require('fs');
var data_model = fs.readFileSync("mobilenet_v2_1.4_224_frozen.pb");
var data_img_cat = fs.readFileSync("cat.png");
var data_img_hopper = fs.readFileSync("grace_hopper.jpg");

var result = JSON.parse( infer(data_model, data_img_hopper) );
console.log("Detected object id " + result[1] + " with probability " + result[0]);

var result = JSON.parse( infer(data_model, data_img_cat) );
console.log("Detected object id " + result[1] + " with probability " + result[0]);
```

Next, build it with `ssvmup`, and then run the Javascript file in Node.js.

```text
$ ssvmup build
$ cd node
$ node app.js
Detected object id 654 with probability 0.3256046
Detected object id 284 with probability 0.27039126
```

You can look up the output detected object ID from the [imagenet\_slim\_labels.txt](https://github.com/second-state/rust-wasm-ai-demo/blob/master/node/imagenet_slim_labels.txt) file from ImageNet.

```text
... ...
284 tiger cat
... ...
654 military uniform
... ...
```

Now, it should be easy for you to turn this example into a Node.js-based web service so that users can send in images and detect objects!

