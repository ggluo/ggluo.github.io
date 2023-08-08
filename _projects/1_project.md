---
layout: page
title: Generative Image Priors for MRI Reconstruction Trained from Magnitude-Only Images
description: generative pretraining, image priors, diffusion models, inverse problem, MR image reconstruction, proximal operator, optimization
img: assets/img/projects/image_priors/image_priors.png
importance: 5
category: work
github: https://github.com/ggluo/image-priors
colab: https://colab.research.google.com/github/ggluo/image-priors/blob/release/misc/demo_image_priors_colab.ipynb
related_publications: Luo__2022a, Luo__2021_b, Luo__2021_a, luo2023generative
---
<div style="float: right; margin-left: 1rem; margin-bottom: 0rem">
{% include figure.html path="assets/img/projects/image_priors/image_priors.png" width="400" title="overview" class="img-fluid rounded z-depth-1" %}
<div class="caption_post">
    Figure 1. Overview of the proposed method.
</div>
</div>

**`Abstract`** In this work, we present a workflow to construct generic and robust generative image priors from magnitude-only images. The priors can then be used for regularization in reconstruction to improve image quality. The workflow begins with the preparation of training datasets from magnitude-only MR images. This dataset is then augmented with phase information and used to train generative priors of complex images. Finally, trained priors are evaluated using both linear and nonlinear reconstruction for compressed sensing parallel imaging with various undersampling schemes. The results of our experiments demonstrate that priors trained on complex images outperform priors trained only on magnitude images. Additionally, a prior trained on a larger dataset exhibits higher robustness. Finally, we show that the generative priors are superior to L$$^\mathrm{1}$$-wavelet regularization for compressed sensing parallel imaging with high undersampling.

**`TLDR`** These findings stress the importance of incorporating phase information and leveraging large datasets to raise the performance and reliability of the generative priors for MRI reconstruction. Phase augmentation makes it possible to use existing image databases for training.

We leverage a diffusion model trained on a small dataset (1000 images) of complex images to augment a much larger dataset (~80k images) for which the phase
information of the image is not available.

<div class="col-sm mt-3 mt-md-0">
{% include figure.html path="assets/img/projects/image_priors/phase_aug.png" title="MMSE vs MAP" class="img-fluid rounded z-depth-1" %}
<div class="caption_post" style="margin-bottom: 1.15rem">
    Human brain images of different quality are shown (rows). On the left, the original magnitude-only images are compared to the magnitude of a corresponding image generated using phase augmentation. On the right, the phase maps of three different  generated images are shown.
</div>
</div>

In total, we trained six priors in this work with a small phase-preserved dataset and phase-augmented images using the full ABIDE dataset with 1206 volumes. The information about computational resources used to train the six different priors is listed in below table.
<div class="col-sm mt-3 mt-md-0">
{% include figure.html path="assets/img/projects/image_priors/priors_table.png" title="MMSE vs MAP" class="img-fluid rounded z-depth-1" %}
</div>

 Comparison of images reconstructed using PICS using the priors $$\texttt{P}_\mathrm{SM},~\texttt{P}_\mathrm{LM},~\texttt{P}_\mathrm{SC},~\texttt{P}_\mathrm{LC},~\texttt{D}_\mathrm{SC}$$ in comparison to an $$\ell_1$$-wavelet reconstruction and a reference. The top two rows (1D) present the results for 5-fold acceleration along phase-encoding direction with 30 calibration lines. The bottom two rows (Poisson) show the results using a Poisson-disc acquisition of 8.2x-undersampling. PSNR and SSIM values are shown in white text.

<div class="col-sm mt-3 mt-md-0">
{% include figure.html path="assets/img/projects/image_priors/pics.png" title="MMSE vs MAP" class="img-fluid rounded z-depth-1" %}
</div>

**`Conclusion`**This work focuses on how to extract prior knowledge from existing
magnitude-only image datasets using phase augmentation with generative models.
The extracted prior knowledge is then applied as regularization in image
reconstruction. The effectiveness of this approach in improving image quality
is systematically evaluated  across different settings. Our findings stress the
importance of incorporating phase information and leveraging large datasets to
raise performance and reliability of generative priors for MRI reconstruction.