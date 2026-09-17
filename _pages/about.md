---
layout: about
title: About
permalink: /
subtitle: ""

profile:
  align: right
  image: photo_ski_2.jpg
  image_circular: false # crops the image to make it circular
  address:   >  #<p>office: ggluo@github</p><p>email: luoguan5@gmail.com</p><p>Göttingen, Germany</p>

news: false  # includes a list of news items
latest_posts: false  # includes a list of the newest posts
selected_papers: false # includes a list of papers marked as "selected={true}"
social: true  # includes social icons at the bottom of the page
---

I work on generative models for imaging: diffusion models, autoregressive models, and the inverse problems they solve. I am most interested in cases where the measurements are incomplete. Reconstructing an image from fewer samples, restoring one when the blur is unknown, or generating a sequence whose frames have to stay consistent with each other. Most of my work sits between the mathematics of those problems and the systems needed to run them.

Since June 2026 I have been a Senior Algorithm Engineer as a contractor at Apple in Munich, working on computational imaging. Before that I was a postdoctoral research fellow at the Luxembourg Institute of Health, and before that a research scientist at University Medical Center Göttingen, where I also completed my PhD.

A full list is on my [publications page](/publications/), but a few are worth singling out. Autoregressive Image Diffusion (NeurIPS 2024) combines causal attention with diffusion so that the model generates image sequences rather than single images, which cuts down the hallucinations standard diffusion models produce when data is heavily undersampled. Self-diffusion (NeurIPS 2025) solves inverse problems with no training set and no pretrained model at all, and its follow-up DeblurSDI (CVPR 2026) handles the blind case where the blur kernel is unknown too. Earlier, my Bayesian reconstruction work in Magnetic Resonance in Medicine produced pixel-wise uncertainty maps alongside the reconstruction, so that a reader can see which parts of an image to trust.

I also spend a lot of time on the engineering side. I have written CUDA kernels from scratch, implemented tensor parallelism and ZeRO-1 to get a clear picture of what distributed training actually costs, and recently rebuilt a 27B open-weights language model from scratch in MLX, first in Python and then in C++, including a fused Metal kernel and a speculative decoder.

I did my PhD at the University of Göttingen with Prof. Dr. Martin Uecker, and before that an M.Phil at the University of Hong Kong and a B.Eng at Xi'an Jiaotong University. I review for NeurIPS, ICML, ICLR, CVPR, AISTATS, TMLR and several IEEE and medical imaging journals.

I am open to research and engineering roles in generative AI, computational imaging and ML systems. My [CV](/assets/pdf/CV.pdf) has the details.

Outside work I play soccer and tennis, and I take photographs.
