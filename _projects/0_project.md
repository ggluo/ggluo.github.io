---
layout: page
title: "Self-diffusion Driven Blind Imaging"
description: CVPR 2026, Computational imaging, Blind image deblur, Self-diffusion, Inverse problem, Optical aberrations
img: https://arxiv.org/html/2510.27439v2/x1.png
importance: 4
category: work
permalink: /projects/DeblurSDI
github: https://github.com/ggluo/DeblurSDI
---


**`TLDR`** We propose DeblurSDI, a training-free and hyperparameter-insensitive framework that leverages self-diffusion to jointly recover sharp images and Point Spread Functions (PSFs). It introduces explicit modeling of optical aberrations via Zernike polynomials and demonstrates state-of-the-art performance on both optical aberration correction and motion deblurring tasks. The full paper is available [here](https://arxiv.org/abs/2510.27439v2) and the code is available [here](https://github.com/ggluo/DeblurSDI).


## Introduction

Optical imaging systems are inherently imperfect due to diffraction limits, lens manufacturing tolerances, assembly misalignment, and other physical constraints. Additionally, unavoidable camera shake and object motion introduce non-ideal degradations during acquisition. These aberrations and motion-induced variations are typically unknown, difficult to measure, and costly to model or calibrate in practice.

Blind inverse problems offer a promising direction by jointly estimating both the latent image and the unknown degradation kernel. However, existing approaches often suffer from convergence instability, limited prior expressiveness, and sensitivity to hyperparameters. Motivated by recent advances in self-diffusion, we propose **DeblurSDI**, a zero-shot, self-supervised blind imaging framework that requires no pre-training.

## Key Contributions

This paper makes three key contributions:

1. **DeblurSDI Framework**: We propose a training-free and self-supervised framework for blind image recovery that integrates self-diffusion to jointly estimate both the sharp image and the PSF without external priors.

2. **Physical Degradation Modeling**: We construct evaluation datasets that include optical-aberration PSFs modeled via Zernike polynomials and motion-blur kernels, enabling comprehensive assessment under challenging conditions.

3. **State-of-the-Art Performance**: We demonstrate that DeblurSDI achieves consistently superior performance and robustness compared to state-of-the-art blind deblurring methods, particularly in stability and PSF estimation.

## Method

### 3.1 Modeling Optical Aberrations

To simulate realistic optical degradations, we adopt the standard wavefront–pupil–PSF formulation widely used in computational optics. Optical aberrations are represented through a weighted sum of **Zernike polynomials**, which form an orthonormal basis on the unit disk and accurately model low- and high-order aberration modes such as defocus, astigmatism, coma, spherical aberration, trefoil, and quadrafoil.

Let $$(\rho,\theta)$$ denote polar coordinates on the normalized pupil domain. The Zernike polynomial of radial order $$n$$ and azimuthal order $$m$$ is defined as:

$$
Z_n^m(\rho,\theta) = \begin{cases}
R_n^{|m|}(\rho)\cos(m\theta), & m \geq 0, \\
R_n^{|m|}(\rho)\sin(|m|\theta), & m < 0,
\end{cases}
$$

where the radial term $$R_n^m(\rho)$$ is given by:

$$
R_n^m(\rho) = \sum_{k=0}^{\frac{n-m}{2}} (-1)^k \frac{(n-k)!}{k! \left(\tfrac{n+m}{2}-k\right)! \left(\tfrac{n-m}{2}-k\right)!} \rho^{\,n-2k}.
$$

A wavefront corrupted by multiple aberrations can therefore be written as:

$$
W(\rho,\theta) = \sum_{(n,m)\in\mathcal{A}} a_{n,m} Z_n^m(\rho,\theta),
$$

where each coefficient $$a_{n,m}$$ controls the severity of the corresponding aberration mode. In our simulations, we include all Zernike modes up to order $$n=4$$, which are sufficient to reproduce a wide range of realistic aberration patterns observed in lens-based imaging systems.

### 3.2 Blind Imaging via Self-Diffusion

We extend the self-diffusion framework to address blind optical correction. The forward model for blind image deblurring is given by:

$$
\mathbf{y} = \mathbf{x}_{\text{true}} \circledast \mathbf{k} + n,
$$

where $$\circledast$$ denotes convolution, the sharp image $$\mathbf{x}_{\text{true}} \in \mathbb{R}^{H \times W \times C}$$ and the PSF $$\mathbf{k} \in \mathbb{R}^{K \times K \times 1}$$ are both unknown.

We employ two dedicated, randomly initialized networks: an **image denoiser $$D_\theta$$** to restore $$\mathbf{x}_{\text{true}}$$, and a **PSF generator $$G_\phi$$** to produce $$\mathbf{k}$$. Our method simulates a reverse diffusion process over $$T$$ discrete time steps, starting with random noise for both the image estimate, $$\mathbf{x}_T$$, and the PSF estimate, $$\mathbf{z}_T$$.

At each time step $$t \in \{T,T-1,\ldots,1\}$$, the current estimates are perturbed with scheduled noise:

$$
\hat{\mathbf{x}}_t = \mathbf{x}_t + \sigma_t \cdot \epsilon_x, \quad \text{and} \quad \hat{\mathbf{z}}_t = \mathbf{z}_t + \sigma'_t \cdot \epsilon_z,
$$

where $$\epsilon_x \sim \mathcal{N}(0,\mathbf{I})$$ and $$\epsilon_z \sim \mathcal{N}(0,\mathbf{I})$$. The noise schedule is $$\sigma_t = \sqrt{1-\bar{\alpha}_t}$$, where $$\bar{\alpha}_t = \prod_{i=0}^t (1-\beta_i)$$ and $$\beta_t = \beta_{\text{end}} + \frac{t}{T-1}(\beta_{\text{start}} - \beta_{\text{end}})$$, and $$\sigma'_t = \mu\sigma_t$$, with $$\mu$$ as an adjustable hyperparameter.

The both networks are jointly optimized within an inner loop by minimizing the following objective:

$$
\mathcal{L}_t(\theta,\phi) = \|(D_\theta(\hat{\mathbf{x}}_t) \circledast G_\phi(\hat{\mathbf{z}}_t)) - \mathbf{y}\|_2^2 + \lambda_k R(G_\phi(\hat{\mathbf{z}}_t))
$$

where $$R(\cdot)$$ is an L1-norm regularization to enforce sparsity on the generated PSF, which is known to be sparse in practice.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="https://arxiv.org/html/2510.27439v2/x1.png" title="PSFs for Individual and Combined Aberrations" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    PSFs for individual and combined aberrations (defocus, coma, astigmatism, spherical, coma+spherical, defocus+coma). Upper row: wavefronts. Lower row: PSFs.
</div>

## Results

### 4.1 Visualization of Iterative Process

As shown in the evolution figures, the scheduled injection of noise effectively "shakes" the optimization out of local minima, allowing it to explore a broader solution space before converging on a high-fidelity reconstruction.
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/projects/DeblurSDI/evo.png" title="Evolution of Image and Kernel Estimates" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Evolution of image and kernel estimates during DeblurSDI's reverse diffusion process. The "up-down-up" curve in the PSNR graph highlights how noise scheduling prevents premature convergence to local optima.
</div>

### 4.2 Optical Aberration Correction

We evaluated DeblurSDI on six simulated optical aberrations (defocus, coma, astigmatism, spherical, coma+spherical, defocus+coma) against state-of-the-art methods. The quantitative results demonstrate DeblurSDI's superior performance:

**Table 1: Optical Aberration Correction Performance (PSNR/SSIM)**

| Method | Levin Dataset | Kohler Dataset | FFHQ Dataset |
|--------|---------------|----------------|--------------|
| Phase-Only | 15.52/0.3722 | 27.37/0.7889 | 26.31/0.7703 |
| FFT-ReLU Deblur | 19.57/0.5655 | 29.89/0.8358 | 23.21/0.6942 |
| SelfDeblur | 18.13/0.4706 | 20.76/0.5409 | 19.65/0.5591 |
| FastDiffusionEM | 18.68/0.5085 | 19.83/0.5242 | 17.90/0.4508 |
| **DeblurSDI (Ours)** | **28.36/0.8598** | **32.07/0.9061** | **33.00/0.9343** |

<br>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="https://arxiv.org/html/2510.27439v2/x7.png" title="Optical Aberration Correction Results" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Reconstruction results of DeblurSDI on optical aberration situations. Our method consistently recovers accurate PSFs and sharp images across different aberration types.
</div>

### 4.3 Motion Deblurring Performance

We conducted extensive experiments on four benchmark datasets (Levin, Cho, Kohler, and FFHQ) to evaluate motion deblurring performance. DeblurSDI demonstrates remarkable robustness across different kernel sizes:

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="https://arxiv.org/html/2510.27439v2/x6.png" title="Robustness to Kernel Size Variations" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Performance and stability comparison across different kernel sizes. DeblurSDI (blue) consistently achieves the highest scores and demonstrates remarkable stability, with its performance remaining largely unaffected by changes in kernel size.
</div>

**Table 2: Motion Deblurring Performance (PSNR/SSIM)**

| Method | Levin Dataset | Cho Dataset | Kohler Dataset | FFHQ Dataset |
|--------|---------------|-------------|----------------|--------------|
| Phase-Only | 20.68/0.6061 | 19.89/0.6746 | 28.23/0.8092 | 25.80/0.7904 |
| FFT-ReLU Deblur | 15.56/0.3845 | 18.73/0.6546 | 25.33/0.7140 | 21.71/0.6579 |
| SelfDeblur | 25.06/0.7301 | 20.37/0.6844 | 21.97/0.5995 | 19.82/0.5563 |
| FastDiffusionEM | 16.55/0.4005 | 15.39/0.4687 | 18.85/0.4813 | 15.59/0.3592 |
| **DeblurSDI (Ours)** | **31.85/0.7911** | **28.73/0.8859** | **29.17/0.7653** | **33.90/0.9064** |

### 4.4 Qualitative Results

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="https://arxiv.org/html/2510.27439v2/x10.png" title="FFHQ Dataset Results" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Deblurring results on the FFHQ dataset. DeblurSDI maintains structural integrity and recovers sharp details even under heavy motion blur, while accurately estimating the blur kernel.
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="https://arxiv.org/html/2510.27439v2/x9.png" title="Cho Dataset Results" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Deblurring results on the Cho dataset. Our method consistently outperforms other approaches in recovering fine details and accurate kernel structures.
</div>

## Conclusion

Through solving blind inverse problems to recover images under optical aberrations and motion blur, we investigate a novel and robust solution for image deblurring. The plausible performance and robustness of our proposed method are brought about by the noise schedule, which is a key component of the self-diffusion framework. We astutely observed this characteristic and extended it to solving extremely unstable joint optimization inverse problems, greatly addressing the problem of unstable solutions or collapse into trivial solutions.

While our method requires longer runtime compared to others due to simultaneous constraints on both the image and kernel with neural networks and hierarchical noise scheduling, this investment is worthwhile because we significantly improve the realism of recovered images across the continuous spectrum, especially high-frequency details.

## Citation

If you find our work useful, please cite our CVPR 2026 paper:

```bibtex
@inproceedings{yang2025selfdiffusion,
  title={Self-diffusion Driven Blind Imaging},
  author={Yang, Yanlong and Luo, Guanxiong},
  booktitle={Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR)},
  year={2026}
}
```

### Resources

- **Paper**: [arXiv:2510.27439v2](https://arxiv.org/abs/2510.27439v2)
- **Code**: [GitHub Repository](https://github.com/ggluo/DeblurSDI)