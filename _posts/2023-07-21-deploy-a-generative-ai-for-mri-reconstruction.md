---
layout: distill
title: Deploy a Generative AI for MRI Reconstruction using BART and TensorRT
permalink: /blog/deploy-ai-models
date: 2023-07-21 16:38 +0200

authors:
  - name: Guanxiong Luo
    url: "https://ggluo.github.io/"
    affiliations:
      name: UMG

tag:
  - deep learning
  - c++
  - computational imaging
  - mri
  - tensorRT

published: false

# Optionally, you can add a table of contents to your post.
# NOTES:
#   - make sure that TOC names match the actual section names
#     for hyperlinks within the post to work correctly.
#   - we may want to automate TOC generation in the future using
#     jekyll-toc plugin (https://github.com/toshimaru/jekyll-toc).

toc:
  - name: Overview 
  - name: Procedure
  - name: Conclusion
  - name: Citation
---


In the realm of medical imaging, Magnetic Resonance Imaging (MRI) stands as a critical diagnostic tool, offering unparalleled insights into the human body's inner workings. However, traditional MRI reconstruction methods often face challenges such as lengthy processing times and susceptibility to artifacts, hindering diagnostic efficiency. To address these limitations, the integration of Generative Artificial Intelligence (AI) presents a transformative solution, revolutionizing MRI reconstruction with unprecedented speed and accuracy.

## Overview

Our mission is to deploy a Generative AI model for MRI reconstruction, leveraging the combined capabilities of BART (Brain Analysis using Reconstruction Toolbox) and TensorRT. This endeavor is guided by the following overview shell script:

```shell
python convert.py folder_to_model model.onnx
bart trt model.onnx kspace reconstruction
bart toimg out reconstruction.png
```

This concise script encapsulates the essence of our deployment strategy, comprising data conversion, inference with TensorRT, and image generation with BART.

## Procedure

### BART

BART serves as the foundation of our MRI reconstruction pipeline, offering a comprehensive suite of tools tailored for neuroimaging tasks. Its versatility and efficiency make it an ideal platform for integrating Generative AI models into MRI reconstruction workflows.

### TensorRT

TensorRT, developed by NVIDIA, empowers our deployment with high-performance deep learning inference optimization. By harnessing the computational power of NVIDIA GPUs, TensorRT accelerates inference speeds, enabling real-time MRI reconstruction in clinical settings.

### Deployment Steps

#### Data Conversion with `python convert.py`

The first step involves converting MRI data from a folder format to the ONNX model format. This prepares the data for inference with our Generative AI model.

#### Inference with `bart trt`

Utilizing the optimized ONNX model, TensorRT performs inference on the MRI data, reconstructing the images with unprecedented speed and efficiency.

#### Image Generation with `bart toimg`

Finally, BART generates high-fidelity MRI images from the reconstructed data, providing clinicians with interpretable visualizations for diagnosis and analysis.

## Conclusion

The deployment of a Generative AI model for MRI reconstruction using BART and TensorRT heralds a new era of diagnostic excellence in medical imaging. By seamlessly integrating advanced AI techniques with robust reconstruction frameworks, we empower healthcare professionals with tools capable of unlocking the full potential of MRI technology, ultimately improving patient outcomes and revolutionizing healthcare delivery.

Stay tuned for further advancements in AI-driven medical imaging, as we continue to push the boundaries of innovation in pursuit of a healthier, more informed world.

## Citation