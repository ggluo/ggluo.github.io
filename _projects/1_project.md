---
layout: page
title: Generative pretrained image priors for MRI reconstruction
description: generative pretraining, image priors, diffusion models, inverse problem, MR image reconstruction, proximal operator, optimization
img: assets/img/projects/image_priors/image_priors.png
importance: 5
category: work
github: https://github.com/ggluo/image-priors
colab: https://colab.research.google.com/github/ggluo/image-priors/blob/release/misc/demo_sampler_colab.ipynb
related_publications: Luo__2022a, Luo__2021_b, Luo__2021_a, Luo_Magn.Reson.Med._2023
---
<div style="float: right; margin-left: 1rem; margin-bottom: 0rem">
{% include figure.html path="assets/img/projects/image_priors/image_priors.png" width="400" title="overview" class="img-fluid rounded z-depth-1" %}
<div class="caption_post">
    Figure 1. Overview of the proposed method.
</div>
</div>

**`This project is ongoing and will be ready soon!`** 

In this work, we present a workflow to train generic and 
robust generative image priors from magnitude images. The priors can then 
be used for regularization in reconstruction to improve image quality.
    The workflow begins with the preparation of 
training datasets from magnitude-only MR images. This dataset is then
augmented with phase information and used to train generative priors
for complex images. Finally, trained priors are evaluated using
both linear and nonlinear reconstruction for compressed sensing
parallel imaging with various undersampling schemes.
    The results of our experiments demonstrate that
priors trained on complex images outperform priors trained only
on magnitude images. Additionally, a prior trained on a larger
dataset exhibits higher robustness. Finally, we show that the
generative priors are superior to L$$^\mathrm{1}$$-wavelet regularization for
compressed sensing parallel imaging with high undersampling.
These findings stress the importance of incorporating phase
information and leveraging large datasets to raise the
performance and reliability of the generative priors for 
MRI reconstruction. Phase augmentation makes it possible to
use existing image databases for training.

