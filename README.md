# Evo-ViT: Slow-Fast Token Evolution for Dynamic Vision Transformer

This repository contains the PyTorch code for Evo-ViT. 

This work proposes a slow-fast token evolution approach to accelerate vanilla vision transformers of both flat and deep-narrow structures without additional pre-training and fine-tuning procedures. For details please see [Evo-ViT: Slow-Fast Token Evolution for Dynamic Vision Transformer](https://arxiv.org/abs/2108.01390) by Yifan Xu*, Zhijie Zhang*, Mengdan Zhang, Kekai Sheng, Ke Li, Weiming Dong, Liqing Zhang, Changsheng Xu, and Xing Sun.
![intro](method.png)

Our code is based on [pytorch-image-models](https://github.com/rwightman/pytorch-image-models), [DeiT](https://github.com/facebookresearch/deit), and [LeViT](https://github.com/facebookresearch/LeViT).

## Citation
If you find our work useful in your research, please consider citing:
```
@article{xu2021evo,
  title={Evo-ViT: Slow-Fast Token Evolution for Dynamic Vision Transformer},
  author={Xu, Yifan and Zhang, Zhijie and Zhang, Mengdan and Sheng, Kekai and Li, Ke and Dong, Weiming and Zhang, Liqing and Xu, Changsheng and Sun, Xing},
  journal={arXiv preprint arXiv:2108.01390},
  year={2021}
}
```

# Preparation
Download and extract ImageNet train and val images from http://image-net.org/. The directory structure is the standard layout for the torchvision datasets.ImageFolder, and the training and validation data is expected to be in the train/ folder and val folder respectively.
```
/path/to/imagenet/
  train/
    class1/
      img1.jpeg
    class2/
      img2.jpeg
  val/
    class1/
      img3.jpeg
    class/2
      img4.jpeg
```
All distillation settings are conducted with a teacher model RegNetY-160, which is available at [checkpoint](https://dl.fbaipublicfiles.com/deit/regnety_160-a5fe301d.pth).

Install the requirements by running:
```
pip3 install -r requirements.txt
```
NOTE: all experiments in the paper are conducted under cuda11.0. If necessary, please install the following packages under the environment with CUDA version 11.0:
[torch1.7.0-cu110](https://download.pytorch.org/whl/cu110/torch-1.7.0%2Bcu110-cp36-cp36m-linux_x86_64.whl), 
[torchvision-0.8.1-cu110](https://download.pytorch.org/whl/cu110/torchvision-0.8.1%2Bcu110-cp36-cp36m-linux_x86_64.whl).

# Model Zoo

Comming soon.

<!-- 
We provide our Evo-ViT models pretrained on ImageNet:
| name            | Top-1 Acc (\%) | Throughput (img/s)   | url                 |
| --------------- | -------------- | -------------------- | ------------------- |
| Evo-ViT-T       |  11            |     11               | [Google Drive](xxx) |
| Evo-ViT-S       |  11            |     11               | [Google Drive](xxx) |
| Evo-ViT-B       |  11            |     11               | [Google Drive](xxx) |
| Evo-ViT-B*      |  11            |     11               | [Google Drive](xxx) |
| Evo-LeViT-128S  |  11            |     11               | [Google Drive](xxx) |
| Evo-LeViT-128   |  11            |     11               | [Google Drive](xxx) |
| Evo-LeViT-192   |  11            |     11               | [Google Drive](xxx) |
| Evo-LeViT-256   |  11            |     11               | [Google Drive](xxx) |
| Evo-LeViT-384   |  11            |     11               | [Google Drive](xxx) |
| Evo-LeViT-256*  |  11            |     11               | [Google Drive](xxx) |
| Evo-LeViT-384*  |  11            |     11               | [Google Drive](xxx) |

\* denotes the input image resolution is 384*384. 
 -->

# Usage

## Training with input resolution of 224
To train Evo-ViT  on ImageNet on a single node with 8 gpus for 300 epochs,  run:
<!--  
Evo-ViT-T
```
python3 -m torch.distributed.launch --nproc_per_node=8 --use_env main_deit.py --model evo_deit_tiny_patch16_224 --drop-path 0 --batch-size 256 --data-path /path/to/imagenet --output_dir /path/to/save
```
-->

Evo-ViT-S
```
python3 -m torch.distributed.launch --nproc_per_node=8 --use_env main_deit.py --model evo_deit_small_patch16_224 --batch-size 128 --data-path /path/to/imagenet --output_dir /path/to/save
```

Sometimes loss Nan happens when training DeiT-B, which is described in this [issue](https://github.com/facebookresearch/deit/issues/29). Our solution is to reduce the batch size to 128, load a [warmup checkpoint](xxxxx) (comming soon) trained for 9 epochs, and train Evo-ViT for the remaining 291 epochs. To train Evo-ViT-B  on ImageNet on a single node with 8 gpus for 300 epochs,  run:
```
python3 -m torch.distributed.launch --nproc_per_node=8 --use_env main_deit.py --model evo_deit_base_patch16_224 --batch-size 128 --data-path /path/to/imagenet --output_dir /path/to/save --resume /path/to/warmup_checkpoint.pth
```

To train Evo-LeViT-192  on ImageNet on a single node with 8 gpus for 300 epochs,  run:
```
python3 -m torch.distributed.launch --nproc_per_node=8 --use_env main_levit.py --model EvoLeViT_192 --batch-size 256 --data-path /path/to/imagenet --output_dir /path/to/save
```
The other models of Evo-LeViT are trained with the same command as mentioned above.

## Training with input  resolution of 384

To train Evo-ViT-B*  on ImageNet on 2 nodes with 8 gpus each for 300 epochs, run:
```
python3 -m torch.distributed.launch --nproc_per_node=8 --nnodes=$NODE_SIZE  --node_rank=$NODE_RANK --master_port=$MASTER_PORT --master_addr=$MASTER_ADDR main_deit.py --model evo_deit_base_patch16_384 --input-size 384 --batch-size 64 --data-path /path/to/imagenet --output_dir /path/to/save
```

To train Evo-ViT-S*  on ImageNet on a single node with 8 gpus for 300 epochs,  run:
```
python3 -m torch.distributed.launch --nproc_per_node=8 --use_env main_deit.py --model evo_deit_small_patch16_384 --batch-size 128 --input-size 384 --data-path /path/to/imagenet --output_dir /path/to/save"
```

Evo-ViT-T*  is trained with the same command as Evo-ViT-S*.

To train Evo-LeViT-384*  on ImageNet on a single node with 8 gpus for 300 epochs,  run:

```
python3 -m torch.distributed.launch --nproc_per_node=8 --use_env main_levit.py --model EvoLeViT_384_384 --input-size 384 --batch-size 128 --data-path /path/to/imagenet --output_dir /path/to/save
```

The other models of Evo-LeViT* are trained with the same command of Evo-LeViT-384*.

## Testing inference throughput
To test inference throughput, first modify the model name in line 153 of benchmark.py. Then, run:
```
python3 benchmark.py
```

## Visualization
Comming soon.
