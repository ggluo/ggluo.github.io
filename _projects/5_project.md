---
layout: page
title: Generalized deep learning-based proximal gradient descent for MR reconstruction
description: generative model, PixelCNN, Bayesian estimation, MR image reconstruction, optimization
img: assets/img/projects/aime/aime_2023.png
importance: 5
github: https://github.com/ggluo/proximator-net
related_publications: Luo_AIME_2023
---

Before employing deep learning in accelerated MRI reconstruction, conventional methods for parallel MR imaging are based on the numerical pseudo-inversion of ill-posed
MRI encoding matrix, which could be prone to reconstruction error at poor conditioning.
The encoding matrix comprises the k-space under-sampling scheme, coil
sensitivities, Fourier transform. The traditional reconstruction involves some gradient
descent methods for minimizing the cost function of the k-space fidelity and the regularization term

With the fast growth of machine learning, the supervised learning have been applied
to MRI reconstruction. Those methods MRI encoding matrices were fully included in the neural network models. These models were trained with predetermined
encoding matrices and corresponding under-sampling artifacts. After training, imaging
configurations, including the k-space under-sampling schemes and coil sensitivities,
associated encoding matrices, must also be unchanged or changed only within predetermined sampling patterns, during the validation and application, which could be
cumbersome or to some extent impractical for the potential clinical use.

To tackle this design challenge, we unroll proximal gradient descent steps into a
network and call it Proximator-Net. The proposed method was adapted
from proximal gradient descent. This study’s objective was to develop a flexible and
practical deep learning-based MRI reconstruction method and implement and validate
the proposed method in an experimental setting regarding changeable k-space undersampling schemes.