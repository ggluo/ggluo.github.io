---
layout: page
title: "Autogressive image diffusion: generation of image sequences and application in MRI reconstruction"
description: autoregressive, diffusion models, image sequences, inverse problem, MRI reconstruction
img: https://arxiv.org/html/2405.14327v2/x1.png
importance: 4
category: work
permalink: /projects/autoregressive-diffusion
github: https://github.com/mrirecon/aid
#colab: https://colab.research.google.com/github/mrirecon/aid/blob/main/scripts/demo_recon.ipynb
related_publications: luo2024autoregressive, Luo_Magn.Reson.Med._2023
---
<div style="float: right; margin-left: 1rem; margin-bottom: 0rem">
{% include figure.html path="https://arxiv.org/html/2405.14327v2/x1.png" width="400" title="overview" class="img-fluid rounded z-depth-1" %}
<div class="caption_post">
    Figure 1. Overview of the proposed method.
</div>
</div>

**`Abstract`** Magnetic resonance imaging (MRI) is a widely used non-invasive imaging modality. However, a persistent challenge lies in balancing image quality with imaging speed. This trade-off is primarily constrained by k-space measurements, which traverse specific trajectories in the spatial Fourier domain (k-space). These measurements are often undersampled to shorten acquisition times, resulting in image artifacts and compromised quality. Generative models learn image distributions and can be used to reconstruct high-quality images from undersampled k-space data. In this work, we present the autoregressive image diffusion (AID) model for image sequences and use it to sample the posterior for accelerated MRI reconstruction. The algorithm incorporates both undersampled k-space and pre-existing information. Models trained with fastMRI dataset are evaluated comprehensively. The results show that the AID model can robustly generate sequentially coherent image sequences. In 3D and dynamic MRI, the AID can outperform the standard diffusion model and reduce hallucinations, due to the learned inter-image dependencies.

**`TLDR`** In 3D and dynamic MRI, the AID can outperform the standard diffusion model and reduce hallucinations, due to the learned inter-image dependencies.

To test different aspects of the autoregressive diffusion models, we generate the sequence of images using the following two approaches.

**Retrospective and prospective (warm start) sampling**:
This method generates a new sequence of images $$\{\tilde{x}_1, \ldots, \tilde{x}_{N-1}\}$$ based on the given sequence $$\{x_0, \ldots, x_N\}$$. $$\tilde{x}_n$$ is sampled using the reverse process given $$\{x_0, \ldots, x_{n-1}\}$$.

<div class="col-sm mt-3 mt-md-0">
{% include figure.html path="https://arxiv.org/html/2405.14327v2/x2.png" title="MMSE vs MAP" class="img-fluid rounded z-depth-1" %}
</div>

<div class="col-sm mt-3 mt-md-0">
{% include figure.html path="https://arxiv.org/html/2405.14327v2/x3.png" title="MMSE vs MAP" class="img-fluid rounded z-depth-1" %}
<div class="caption_post" style="margin-bottom: 1.15rem">
Top: A sequence of images from dataset is shown in the first row and is used as conditioning to generate retrospective samples that are shown in the second row. Bottom: With the given sequence in the top as a warm start, prospective samples extend it and are shown.
</div>
</div>

**Prospective sampling (cold start)**:
A fixed-length sliding window is initialized with the given sequence $$x_{<n}=\{x_0, \ldots, x_{N-1}\}$$. $$x_N$$ is generated using reverse process with the current window as conditioning. Subsequently, the window is updated by adding the newly generated $$x_N$$ and removing the earliest image $$x_0$$. This autoregressive sampling process is repeated until the stop condition is met. We refer to this process as a warm start. In a cold start, the window is initialized with zeros, and each element $$x_n$$ in it is updated with newly generated images from the beginning to the end, after which the generation is warmed up.
<div class="col-sm mt-3 mt-md-0">
{% include figure.html path="https://arxiv.org/html/2405.14327v2/x4.png" title="MMSE vs MAP" class="img-fluid rounded z-depth-1" %}
</div>

**MRI reconstruction**:
For the visual impression of the improvement by the AID model in reconstruction, we show the reconstructed images in the following figure and more of them in Appendix. The images reconstructed using AID are more visually similar to the reference images than using Guide, even which also provides aliased-free images. Furthermore, it is worth noting that more visually notable hallucinations were introduced by the Guide model than the AID model, which means AID is more trustworthy.
<div class="col-sm mt-3 mt-md-0">
{% include figure.html path="https://arxiv.org/html/2405.14327v2/x9.png" title="MMSE vs MAP" class="img-fluid rounded z-depth-1" %}
<div class="caption_post" style="margin-bottom: 1.15rem">
E: equispaced, R: random. The red lines are autocalibration signal (ACS) and equispaced mask is not shown. Zero-filled images are computed by inverse Fourier transform of the zero-filled k-space data. The hallucinations are pointed with red arrows.
</div>
</div>


**`Conclusion`** The proposed autoregressive image diffusion model offers an approach to generating image sequences, with significant potential as a trustworthy prior in accelerated MRI reconstruction. In various experiments, it outperforms the standard diffusion model in terms of both image quality and robustness by taking the advantage of the prior information on inter-image dependencies.