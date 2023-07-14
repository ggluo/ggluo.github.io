---
layout: page
title: Bayesian MRI reconstruction with joint uncertainty estimation using diffusion models
description: sampling posterior, Bayesian inference, diffusion models, uncertainty estimation, inverse problem, MR image reconstruction
img: assets/img/projects/sampling_posterior/sampling_posterior.png
importance: 5
github: https://github.com/mrirecon/spreco
colab: https://colab.research.google.com/github/mrirecon/spreco/blob/main/examples/scripts/demo_recon.ipynb
related_publications: Luo_Magn.Reson.Med._2023, Luo__2022_a
---
<div style="float: right; margin-left: 1rem; margin-bottom: 0rem">
{% include figure.html path="assets/img/projects/sampling_posterior/sampling_posterior.png" width="400" title="overview" class="img-fluid rounded z-depth-1" %}
<div class="caption_post">
    Figure 1. Overview of the proposed method.
</div>
</div>

**`Abstract`** In recent years, the application of deep
learning has made significant advancements in fast MRI
reconstruction, yielding promising results. However,
worries about the uncertainty caused by undersampling
strategies and algorithms have limited their usage in
clinical practice until now. Therefore, the uncertainty
assessment constitutes an important step
for deep learning-based approaches. The uncertainty is twofold:
(1) the uncertainty of weights inside the neural
network; and (2) the uncertainty introduced by the
missing k-space data points. The uncertainty from missing
k-space data points can be addressed in a Bayesian imaging framework.

**`Theory`** The image reconstruction problem is formulated in a Bayesian view and samples
are drawn from the posterior distribution given the k-space using the Markov
chain Monte Carlo (MCMC) method. The minimum mean square error (MMSE)
and maximum a posterior (MAP) estimates are computed. The chains are the reverse of a diffusion process (c.f., Figure 1). Score-based generative models are
used to construct chains and are learned from an image database.

**`MMSE vs MAP`** 200 extended iterations after random exploration (i.e, without noise disturbance) and a deterministic estimate of
      MAP are indicated by solid and dashed lines respectively. (a) PSNR and SSIM over iterations. (b) Variance$_\text{1}$ and
      variance$_\text{2}$ were computed from unextended samples and extended samples respectively. Extended samples converge to $\mathbf{x}_\text{MAP}$.

<div class="col-sm mt-3 mt-md-0">
{% include figure.html path="assets/img/projects/sampling_posterior/map_end.png" title="overview" class="img-fluid rounded z-depth-1" %}
<div class="caption_post" style="margin-bottom: 1.15rem">
    Figure 2. The proposed reconstruction algorithm.
</div>
</div>

**`Highlight`** Reconstructions are $$\ell_1$$-ESPIRiT, XPDNet, $$\mathbf{x}_\text{MMSE}$$
highlighted with confidence interval (CI), $$\mathbf{x}_\text{MMSE}$$
and a fully-sampled coil-combined image (CoilComb). Hallucinations
appear whe using 8-fold acceleration and are highlighted with
CI after thresholding. 
Selected regions of interests are presented in a zoomed view.
<div class="col-sm mt-3 mt-md-0">
{% include figure.html path="assets/img/projects/sampling_posterior/fusion.png" title="overview" class="img-fluid rounded z-depth-1" %}
<div class="caption_post" style="margin-bottom: 1.15rem">
    Figure 2. The proposed reconstruction algorithm.
</div>
</div>


**`Conclusion`** 