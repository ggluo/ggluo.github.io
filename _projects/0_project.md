---
layout: post
title: Hands-on tutorials for spreco
description: spreco, generative pretraining, image priors
img: assets/img/projects/image_priors/image_priors.png
importance: 5
category: work
related_posts: false
giscus_comments: true
tags: spreco tutorials
toc:
  sidebar: right
---


## Overview 
This post will give a brief introduction to the usage of [spreco](https://github.com/mrirecon/spreco). Spreco is a tool for MR researchers to train generative priors for image reconstruction, developed with TensorFlow. It is recommended to use it on a Linux-based system. The same to most workflows in deep learning, we can use it by following steps below.

1. The preparation of data
2. Training of generative priors
3. Export and inference
4. Feedback

## Preparation of data
Before embarking on any ML projects, the first thing come up is where to get data for training. Except collecting your own data, specifically in the field of medical imaging, there so many datasets are made publicly available for researchers. I found a list on this github [repository](https://github.com/sfikas/medical-imaging-datasets).

Then many questions follow up. How large is the dataset? Is it necessary to do preprocessing before using it? Do I have enough computation resources to work on it? From my own experiences, the preprocessing step could be time-consuming, and it is always beneficial to parallelize your tasks. If the tasks are cpu-based, the python library [multiprocessing](https://docs.python.org/3/library/multiprocessing.html#module-multiprocessing) is a good option. If the tasks are gpu-based, one possible trick is to have a small snippet for one task and then use shell script to parallelize multiple snippets for tasks over the available gpus. In the end, make sure to store processed files in an organized way.By doing this, it would be easier to define a dataloader for training. When your data is heavy and it takes long to load, here is one data loading pipeline, [Tensorpack DataFlow](https://github.com/tensorpack/dataflow), optimized for speed. To prepare data for spreco after finishing the preprocessing step, you need to create a list of training files specifying where the files are. Shell commands `ls` and `find` are so efficient to do so.
```text
./dataset/abide_2/train/abide_1000000.npz
./dataset/abide_2/train/abide_1000001.npz
./dataset/abide_2/train/abide_1000002.npz
./dataset/abide_2/train/abide_1000003.npz
```

## Training of generative priors
I assume nowadays most labs are equipped with workstations that have multiple gpus, keeping track of hyperparameters needs a good deal of patience and monitoring the training is important. Spreco provides a workable solution for these things with one yaml config file. 

### Configuration file

```yml
# parameters for the model
model: 'PIXELCNN'
batch_size: 6
input_shape: [256, 256, 2]
nr_resnet: 3
nr_filters: 128
nr_logistic_mix: 10
data_chns: 'CPLX'
dropout_rate: 0.5
itg_interval: 255.0
rlt: 1
layer_norm: False
conditional: False

# learning rate schedule
lr_warm_up_steps: 1000
lr_start: 0.0001
lr_min: 0.0001
lr_max: 0.0001
lr_max_decay_steps: 2000

max_keep: 100
max_epochs: 500
save_interval: 50     # take a snapshot every 50 epochs
saved_name: pixelcnn
log_folder: ./logs/   # where to save the snapshots of the model

num_prepare: 10
print_loss: True
train_list: train_list 
test_list:  test_list

nr_gpu: 4
gpu_id: '0,1,2,3'
```

### Script for training

After the configuration file is done, you need to write a script to start training that includes the read of configuration file, the initialization of the dataloader, and the start of the trainer. The below is an example.

```python

from spreco.common import utils
from spreco.trainer import trainer
from spreco.dataflow.base import RNGDataFlow
from spreco.dataflow.common import BatchData
from spreco.dataflow.parallel_map import MultiThreadMapData

import os
import numpy as np
import argparse

class cfl_pipe(RNGDataFlow):

    def __init__(self, files, shuffle):
        self._size   = len(files)
        self.files   = files
        self.shuffle = shuffle
    
    def __len__(self):
        return len(self.files)
    
    def __iter__(self):
        idxs = np.arange(self._size)
        if self.shuffle:
            self.rng.shuffle(idxs)
        
        for idx in idxs:
            fname = self.files[idx]
            yield fname

def main(config_path):

        config = utils.load_config(config_path)
        train_files = utils.read_filelist(config['train_list'])
        test_files = utils.read_filelist(config['test_list'])

    def load_file(x):
        """
        x     ---> file path
        imgs  ---> normalized images with shape (batch_size, x, y, 2) 
        """
        path, ext = os.path.splitext(x)
        imgs = np.squeeze(utils.readcfl(path))
        imgs = imgs / np.max(np.abs(imgs), axis=(1,2), keepdims=True)
        imgs = utils.cplx2float(imgs)
        return imgs

    def map_f(x):
        d = aug_load_file(x)
        return {"inputs": d}
    
    nr_elm = config['batch_size']*config['nr_gpu']

    d1 = cfl_pipe(train_files, True)
    d1 = MultiThreadMapData(d1, num_thread=config['num_thread'], map_func=map_f,  buffer_size=nr_elm*10, strict=True)
    train_pipe = BatchData(d1, nr_elm, use_list=False)

    d2 = cfl_pipe(test_files, True)
    d2 = MultiThreadMapData(d2, num_thread=config['num_thread'], map_func=map_f,  buffer_size=nr_elm*10, strict=True)
    test_pipe = BatchData(d2, nr_elm, use_list=False)

    go = trainer(train_pipe, test_pipe, config)
    utils.log_to(os.path.join(go.log_path, 'train_files'), train_files)
    utils.log_to(os.path.join(go.log_path, 'test_files'), test_files)
    utils.log_to(os.path.join(go.log_path, 'train_info'), [utils.get_timestamp() + ", the training is starting"])
    go.train()
    utils.log_to(os.path.join(go.log_path, 'train_info'), [utils.get_timestamp() + ", the training is ending"])
    utils.color_print('TRAINING FINISHED at ' + utils.get_timestamp())

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description='')
    parser.add_argument('--config', metavar='path', default='config file for training', help='')

    args = parser.parse_args()
    main(args.config)
```
## Export and inference
For every model in spreco, you can use the `exporter` module to export the trained models. It exports the neural network with default inputs and outputs. You can load the exported graph and inference it with [BART](https://github.com/mrirecon/bart) toolbox. It is possible to customize the inputs and outputs. The below is an example for handling 2D MR images. Click here for [3D volumes](https://github.com/ggluo/image-priors/blob/release/scripts/recon/create_graph.py).

```python
from spreco.exporter import exporter
from spreco.common import utils
import tensorflow.compat.v1 as tf
tf.disable_eager_execution()

import numpy as np

e = exporter("./PixelCNN/cplx_large", "pixelcnn", default_out=False, path="./PixelCNN/exported", name="cplx_large", gpu_id='0')
x = tf.placeholder(tf.float32, shape=[1, 256, 256, 2], name="input_0")


logits = e.model.eval(x)
loss   = e.model.loss_func(x, logits) / np.log(2.0) / np.prod([1, 256, 256, 2])

e.export([x], [loss], attach_gradients=True)
```

We prepare a complete [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/ggluo/image-priors/blob/release/misc/demo_image_priors_colab.ipynb) to demonstrate how to use the trained generative priors when using BART for image reconstruction. 

Once you have gone through the whole workflow, you can start to collect feedback and iterate models.