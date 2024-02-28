---
layout: post
title: "How Bindings Work with I/O in TensorRT"
date: 2023-03-08 12:31 +0100
tag: 
    - c++, tensorRT
---

In TensorRT, a "binding" refers to the association between a tensor in the computational graph (network) and a specific memory buffer that holds the data for that tensor during inference. Bindings are used to specify how input and output tensors of a neural network are mapped to memory buffers where the input data is provided and where the output data is retrieved after inference.

Here's a basic overview of how bindings work in TensorRT from the docs from [TensorRT engine](https://docs.nvidia.com/deeplearning/tensorrt/api/python_api/infer/Core/Engine.html?highlight=binding#tensorrt.ICudaEngine) :

1. **Input Bindings**: Input bindings specify where the input data for the neural network will be located during inference. Typically, this involves specifying memory buffers where the input data is stored, such as CPU or GPU memory.

2. **Output Bindings**: Output bindings specify where the output data produced by the neural network during inference will be placed. Similar to input bindings, output bindings specify memory buffers where the output data will be stored, such as CPU or GPU memory.

3. **Binding Indices**: Each input and output tensor in the network is associated with a unique binding index. These binding indices are used to identify which input or output tensor corresponds to which memory buffer during inference.

4. **Binding Shapes**: Along with specifying the memory location for input and output tensors, bindings also specify the shape (dimensions) of each tensor. This ensures that the memory buffers provided for input and output data have the correct size to accommodate the tensors produced by the network.

5. **Binding Data Types**: Bindings also specify the data type of each tensor, such as float32, int8, etc. This ensures that the memory buffers provided for input and output data have the correct data type to match the tensors produced by the network.

