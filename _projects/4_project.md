---
layout: page
title: MRI reconstruction using deep Bayesian estimation
description: generative model, PixelCNN, Bayesian estimation, MR image reconstruction, optimization
img: assets/img/projects/map/overview.png
importance: 5
github: https://github.com/mrirecon/spreco
related_publications: Luo_Magn.Reson.Med._2020
---

<div style="float: right; margin-left: 1rem; margin-bottom: 0rem">
{% include figure.html path="assets/img/projects/map/overview.png" width="350" title="overview" class="img-fluid rounded z-depth-1" %}
<div class="caption_post">
    Overview of the proposed method.
</div>
</div>

**`Abstract`** We developed a deep learning-based Bayesian estimation for MRI
reconstruction. The image reconstruction from
incomplete k-space measurement was obtained by maximizing the posterior probability.
The autoregressive generative model [PixelCNN](https://arxiv.org/abs/1701.05517) was utilized as the image prior
and the k-space data fidelity was enforced by using an equality constraint.
The stochastic back-propagation was utilized to calculate the descent gradient in the
process of maximum a posterior, and a projected sub-gradient method was used to
impose the equality constraint. In contrast to the other deep learning reconstruction
methods, the proposed one used the likelihood of prior as the training loss and the
objective function in reconstruction to improve the image quality.
methods, the proposed one used the likelihood of prior as the training loss and the
objective function in reconstruction to improve the image quality.



<div class="col-sm mt-3 mt-md-0">
{% include figure.html path="assets/img/projects/map/algo.png" title="overview" class="img-fluid rounded z-depth-1" %}
<div class="caption_post" style="margin-bottom: 1.15rem">
    The proposed algorithm.
</div>
</div>



<div style="float: right; margin-left: 1rem; margin-bottom: 0rem">
{% include video.html path="assets/img/projects/map/map.mp4" class="img-fluid rounded z-depth-1" controls=true autoplay=true %}
<div class="caption_post">
    Visualization of the iterations.
</div>
</div>



**`Conclusion`** The Bayesian estimation significantly improved the reconstruction
performance, compared with the conventional L$$^1$$-sparsity prior
in compressed sensing reconstruction tasks. More importantly, the
learnt prior knowledge is generic to many MRI reconstruction scenarios
when changing sampling patterns, coils and so on.