---
title: TensorFlow to PyTorch
subtitle: How to evaluate power using Cadence Innovus given VCD files

# Summary for listings and search engines
summary: Converting a model checkpoint from any framework to any framework is a delicate process if you want to achieve the exact same performance.

# Link this post with a project
projects: []

# Date published
date: "2020-06-18T00:00:00Z"

# Date updated
lastmod: "2020-06-18T00:00:00Z"

# Is this an unpublished draft?
draft: false

# Show this page in the Featured widget?
featured: false

# Featured image
# Place an image named `featured.jpg/png` in this page's folder and customize its options here.
image:
  caption: 'Image credit: [**Unsplash**](https://unsplash.com/photos/Oml7GeJM1QY)'
  focal_point: ""
  placement: 1
  preview_only: false

authors:
- admin

tags:
- Synopsys Design Compiler
- Cadence Innovus
- Power Estimation

categories:
- Tutorial
---

Converting a model checkpoint from any framework to any framework is a delicate process if you want to achieve the exact same performance.

I was recently asked to evaluate [my work](https://arxiv.org/abs/2004.09309) on the [MLPerf inference benchmark suite](https://arxiv.org/abs/1911.02549).
So I will guide you through the process and subtleties of migrating a TensorFlow model to PyTorch.
If all you want are MLPerf ResNet-50 and MobileNet-v1 PyTorch checkpoints, you can grab them from my [GitHub repository](https://github.com/gilshm/mlperf-pytorch).

## Using the PB File

A TensorFlow `pb` (protobuf) file comprises a description of the model graph as well as the layer weights and parameters.
The first stage is, therefore, parsing the model pb file.
MLPerf checkpoint files can be download from [here](https://github.com/mlperf/inference/tree/master/vision/classification_and_detection).
Inspired by [this](https://leimao.github.io/blog/Save-Load-Inference-From-TF2-Frozen-Graph/) blog post, I used this class to extract all the necessary data from the pb file:
```python
import tensorflow as tf
import numpy as np


class NeuralNetPB(object):
    def __init__(self, model_filepath):
        self.input = None
        self.graph = None
        self.placeholders = None
        self.layers = None
        self.weights = {}
        self.model_filepath = model_filepath
        self._load_graph()

    def _load_graph(self):
        self.graph = tf.Graph()

        with tf.io.gfile.GFile(self.model_filepath, 'rb') as f:
            graph_def = tf.compat.v1.GraphDef()
            graph_def.ParseFromString(f.read())

        self.placeholders = [n.name for n in graph_def.node if n.op in ('Placeholder')]

        with self.graph.as_default():
            self.input = tf.compat.v1.placeholder(np.float32,
                                                  shape=[None, 224, 224, 3],
                                                  name='input_tensor')
            # Trying the first placeholder as input, might not work.
            tf.import_graph_def(graph_def, {self.placeholders[0]: self.input})

        self.graph.finalize()

        # Get layer names
        self.layers = [op.name for op in self.graph.get_operations()]

        # Get layer weights
        weight_nodes = [n for n in graph_def.node if n.op == 'Const']
        for n in weight_nodes:
            self.weights[n.name] =
                    tf.python.tensor_util.MakeNdarray(n.attr['value'].tensor)

        self.sess = tf.compat.v1.Session(graph=self.graph)

    def test(self, data, output_layer):
        """
        Feed-forward data and get the output from any layer.
        :param data: data to propagate.
        :param output_layer: the output layer (can be any hidden layer for debug).
        :return:
        """
        output_tensor = self.graph.get_tensor_by_name(output_layer)
        output = self.sess.run(output_tensor, feed_dict={self.input: data})
        return output
```

After constructing a NeuralNetPB class, it will hold all weights within its weights attribute.
Now that we have the weight tensors, it's time to map them to the corresponding layers in our PyTorch model.

## Layer Mapping
First, notice we do not have the model definition as a TensorFlow Python file.
However, since we deal with well-known model architectures, layers pretty much match.

> **Tip #1:** Use [Netron](https://lutzroeder.github.io/netron/) to visualize your model graph.
> This will be helpful in understanding the model structure within the `pb` file, as well as mapping the layers from the pb file to PyTorch.

Now, let’s start mapping. I will give a couple of examples from MobileNet-v1:
1. `MobilenetV1/Conv2d_0/weights` (3-3-3-32) --- This is the first convolution layer. We map it to `model.0.1.weight`.
2. `MobilenetV1/Conv2d_0/BatchNorm/gamma` (32) --- This is the first batch normalization layer. gamma corresponds to the layer weights in PyTorch, so we map it to `model.0.2.weight`. The same BatchNorm layer consists a `beta`, `moving_mean`, and `moving_variance` fields, which corresponds to PyTorch `bias`, `running_mean`, and `running_var`, respectively.
3. `MobilenetV1/Conv2d_1_depthwise/depthwise_weights` (3, 3, 32, 1) --- This is the second convolution layer (or the first depthwise convolution layer), and it is mapped to `model.1.weight`.

DNN model structures are usually repetitive, so once you get the idea, you'll be able to write parts in for loops.

Since the `NeuralNetPB` attributes are not PyTorch tensors, cast them with `torch.FloatTensor(...)`.

> **Subtlety #1: Permute.** TensorFlow convolution layers' weight tensors are ordered differently.
> According to TensorFlow [documentation](https://www.tensorflow.org/api_docs/python/tf/nn/conv2d) the weight tensor shape is `[H, W, IN_C, OUT_C]`, whereas PyTorch weight tensor shape is `[OUT_C, IN_C, H, W]`.
> Therefore, rearrange the tensor: `torch.FloatTensor(...).permute(3, 2, 0, 1)`.
> Having said that, the depthwise convolution should be rearranged with `.permute(2, 3, 0, 1)`.

Once you are done here, you should probably save everything with `torch.save(...)`.

## Model Modifications
If you are done with the previous section, it means that the TensorFlow reference model and your PyTorch model is identical from a layer-wise perspective.
However, there are some additional subtleties to pay attention to.

> **Subtlety #2: Params.** Weights are not the only parameters.
> For example, the batch normalization layer consists of an `epsilon` attribute (for numerical stability).
> `epsilon` default value is different in TensorFlow and PyTorch.
> But even if it was equal, TensorFlow ResNet-50 model modifies epsilon, so be aware.
> Another example is Dropout layers, if exist.

> **Subtlety #3: Padding.** The padding argument in TensorFlow consists an option that does not exist in PyTorch: `SAME`.
> `SAME` means that the input feature map will be padded so the output feature map (i.e., the convolution operation result) spatial dimensions will be equal to the input dimensions.
> However, what happens if padding is asymmetric? will TensorFlow pad more on the left or on the right?
> **TensorFlow pads on right, whereas PyTorch pads on the left.**
> The same logic applies vertically, that is, **there may be an extra row of zeros at the bottom, whereas with PyTorch the extra row will come at the top.**

You can check out my slightly modified [ResNet-50](https://github.com/gilshm/mlperf-pytorch/blob/master/models/resnet.py) and [MobileNet-v1](https://github.com/gilshm/mlperf-pytorch/blob/master/models/mobilenet_v1.py) models and see how I modified them accordingly.

## Preprocessing
If you want to reproduce the exact same result as the reference model (in this case, the MLPerf models) then you have to reproduce the exact same preprocessing stages.
The common preprocessing stages when dealing with the ImageNet dataset are (1) resizing to 256; (2) center cropping to 224; (3) normalization to values between [0, 1] (that is, division by 255); and (4) mean and std correction of each of the RGB channels. However, with MLPerf, the std is not normalized, and the bias is normalized differently (see [here](https://github.com/mlperf/inference/blob/master/vision/classification_and_detection/python/dataset.py).

Moreover, even after doing the exact same preprocessing, I found that the same interpolation algorithm is implemented differently in MLPerf, which uses OpenCV, and PyTorch, which uses PIL.
Since MLPerf preprocesses the entire ImageNet validation set and saves it as NumPy arrays, I decided to avoid PyTorch preprocessing and just load the NumPy arrays directly.
At this point I also achieved the exact same performance as MLPerf reports --- 76.456% and 71.676% top-1 accuracies for ResNet-50 and MobileNet-v1, respectively.

## Debugging
So you mapped the weights, and (you think) you have fine-tuned all the parameters and pre-processing stages according to your reference model.
You start inference and nothing happens, 0% accuracy. What's next? there's no easy answer, you'll have to debug it.
What I did is going layer-by-layer and comparing the tensor values.
Things should be almost identical (with some floating-point accepted differences).
Use NeuralNetPB test function, for example, with `nn_pb.test(input, 'import/MobilenetV1/MobilenetV1/Conv2d_1_depthwise/Relu6:0')` you'll get one of the ReLU result.
You should know all layer names by now from the mapping stage.

## Conclusions
Be patient, that's the most important thing.
Migrating any model between frameworks requires attention to all the fine details.
By the way, there are some tools for automatic conversions, but I have no idea if they work properly or not.

You can find my MLPerf ResNet-50 and MobileNet-v1 PyTorch checkpoints and models at [my GitHub repository](https://github.com/gilshm/mlperf-pytorch).

