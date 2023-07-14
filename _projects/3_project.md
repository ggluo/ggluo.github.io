---
layout: page
title: Using generative image priors for image reconstruction with BART
description: MR image reconstruction, model deployment, TensorFlow C API, TensorFlow computation graph
img: assets/img/projects/bart_tf/bart_tf.png
importance: 5
category: work
github: https://github.com/mrirecon/bart
colab: https://colab.research.google.com/github/mrirecon/bart-workshop/blob/master/ismrm2021/bart_tensorflow/bart_tf.ipynb
related_publications: Luo__2021_a, Blumenthal_Magn.Reson.Med._2023
---
<div style="float: right; margin-left: 1rem; margin-bottom: 0rem">
{% include figure.html path="assets/img/projects/bart_tf/bart_tf.png" width="400" title="overview" class="img-fluid rounded z-depth-1" %}
<div class="caption_post">
    Figure 1. Overview of the proposed method.
</div>
</div>

**`Abstract`** 
Advanced reconstruction algorithms based on deep learning have recently drawn a
lot of interest as they tend to outperform state-of-the-art methods. BART [1] is a versatile framework for image reconstruction. In this work, we demonstrate how neural
networks trained and tested with TensorFlow [5] can be integrated into BART. As an
example, we discuss non-Cartesian parallel imaging using the SENSE model regularized by a log-likelihood image prior. The image prior is based on an autoregressive
generative network pixel-cnn++ [4]. Furthermore, we validated the reconstruction
pipeline using radial brain scans.


**`Method`**

The left figure shows the evolution of the probability density function (PDF) (dashed curves) and the ground truth PDF (solid curves) at five selected pixels over iterations. The middle figure shows the evolution of magnitude image. The right figure shows the convergence of metrics.

<div style="margin-bottom: 0rem">
<div style="margin-bottom: -0.5rem">
{% include video.html path="assets/img/projects/bart_tf/movie.mp4" class="img-fluid rounded z-depth-1" controls=true autoplay=true %}
</div>
<div class="caption_post" style="margin-bottom: 1rem">
    Figure 3. Visualization of the iterations.
</div>
</div>



**`Conclusion`** Trained neural networks can be incorporated into established MRI reconstruction
routines within the BART toolbox. In this work, we demonstrate
training of the prior and implementation of reconstruction pipelines as a proof of concept.
