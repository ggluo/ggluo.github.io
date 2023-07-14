---
layout: page
title: Generative pretrained image priors for MRI reconstruction
description: generative pretraining, image priors, diffusion models, inverse problem, MR image reconstruction, proximal operator, optimization
img: assets/img/projects/image_priors/image_priors.png
importance: 5
category: work
github: https://github.com/ggluo/image-priors
related_publications: Luo__2022a, Luo__2021_b, Luo__2021_a, Luo_Magn.Reson.Med._2023
---

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
use existing image data bases for training.


You can also put regular text between your rows of images.
Say you wanted to write a little bit about your project before you posted the rest of the images.
You describe how you toiled, sweated, *bled* for your project, and then... you reveal its glory in the next row of images.


<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.html path="assets/img/6.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm-4 mt-3 mt-md-0">
        {% include figure.html path="assets/img/11.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    You can also have artistically styled 2/3 + 1/3 images, like these.
</div>
