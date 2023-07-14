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
Building on this previous [project](/projects/4_project/), this work aims to deploy the trained model with an MR image reconstruction toolbox, **[BART](https://github.com/mrirecon/bart)**, which is a versatile framework for image reconstruction. As shown in Figure 1, there are two steps to realize this: (a) export the constructed computation graph with TensorFlow; (b) use the graph as regularization in BART.
We validated the reconstruction pipeline using radial brain scans and the [SENSE](https://pubmed.ncbi.nlm.nih.gov/10542355/) model regularized by a log-likelihood image prior.

<div style="float: right; margin-left: 1rem; margin-bottom: 0rem; margin-top: 1rem">
{% include figure.html path="assets/img/projects/bart_tf/spreco.png" width="400" title="overview" class="img-fluid rounded z-depth-1" %}
<div class="caption_post">
    Figure 2. The structure of spreco.
</div>
</div>

**`Method`** We developed a Python library for training generative models, **[spreco](https://github.com/mrirecon/spreco)**, based on TensorFlow, which has the following features:
1. Distributed training 
2. Interruptible training
3. Efficient dataloader for medical images
4. Customizable with a configuration file

And we trained a log-likelihood prior, $$\log p({x};\mathtt{NET}(\hat{\Theta}, {x}))$$[$$^1$$](https://arxiv.org/abs/1701.05517), which is used to impose learned prior knowledge of images in the SENSE model. 
The reconstruction is commonly formulated as the following minimization problem
\begin{equation}
    \hat{x}=\underset{x}{\arg\min}\ \|\mathcal{A}{x}-{y}\|_2^2 + \lambda \log p({x};\mathtt{NET}(\hat{\Theta}, {x})),\nonumber
    \label{eq:1}
\end{equation}
where the first term ensures data consistency between the acquired [k-space](https://en.wikipedia.org/wiki/K-space_(magnetic_resonance_imaging)) data $${y}$$ and the desired image $${x}$$, $$\mathcal{A}$$ is the forward operator. This problem is solved with [FISTA](https://www.ceremade.dauphine.fr/~carlier/FISTA) algorithm that has been implemented in BART. To use the learned log-likelihood prior with BART, we have implemented a [wrapper](https://github.com/mrirecon/bart/commit/8b8d4ed2a727bcbc19a11e9ddd64d46f7e5e21d9) using [TensorFlow C API]([$$^1$$](https://www.tensorflow.org/install/lang_c)) for the initialization, restoration and inference of the exported trained model. Click here [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/mrirecon/bart-workshop/blob/master/ismrm2021/bart_tensorflow/bart_tf.ipynb) and give a quick tryout!

Figure 3 displays the evolution of image during the process of reconstruction.
<div style="margin-bottom: 0rem">
<div style="margin-bottom: -0.5rem">
{% include video.html path="assets/img/projects/bart_tf/evo.mp4" class="img-fluid rounded z-depth-1" controls=true autoplay=true muted%}
</div>
<div class="caption_post" style="margin-bottom: 1rem">
    Figure 3. The left sub-figure shows the evolution of the probability density function (PDF) (dashed curves) and the empirically learned PDF (solid curves) at five selected pixels over iterations. The middle sub-figure shows the evolution of magnitude image. The right sub-figure shows the convergence of metrics.
</div>
</div>


**`Conclusion`** We showcased the seamless integration of a TensorFlow trained model into the existing MRI reconstruction workflows of the BART toolbox. 
This integration allows us to leverage the image reconstruction capabilities offered by BART while harnessing the advantages of rapid prototyping of advanced deep learning models using TensorFlow.
