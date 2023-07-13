---
layout: page
title: Bayesian MRI reconstruction with joint uncertainty estimation using diffusion models
description: sampling posterior, Bayesian inference, diffusion models, uncertainty estimation, inverse problem, MR image reconstruction
img: assets/img/projects/sampling_posterior.png
importance: 5
github: https://github.com/mrirecon/spreco
colab: https://colab.research.google.com/github/mrirecon/spreco/blob/main/examples/scripts/demo_recon.ipynb
---

The application of generative models in MRI reconstruction
is shifting researchers' attention from unrolled reconstruction
networks to probabilistic methods which can be used for
unsupervised medical image reconstruction [1-4].
We formulate the reconstruction problem from the perspective of
Bayesian inference, which enables efficient sampling
from learned posterior probability distributions [1-2]. Different
from conventional deep learning-based MRI reconstruction
techniques, samples are drawn from the posterior distribution
given the measured k-space using the Markov chain Monte Carlo
(MCMC) method. Because the generative model can be learned
from an image database independently of the forward operator,
the same pre-trained models can be applied to k-space acquired
with different sampling schemes or receive coils. Here,
we present the general principles, results related to the
uncertainty of the reconstruction, the transferability of
learned information, and the comparison using data from
the fastMRI challenge.
