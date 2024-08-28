# Image Classification Part

## Image Classification
### 1. Requirements

torch>=1.7.0; torchvision>=0.8.0; pyyaml; [apex-amp](https://github.com/NVIDIA/apex) (if you want to use fp16); [timm](https://github.com/rwightman/pytorch-image-models) (`pip install git+https://github.com/rwightman/pytorch-image-models.git@9d6aad44f8fd32e89e5cca503efe3ada5071cc2a`)

data prepare: ImageNet with the following folder structure, you can extract ImageNet by this [script](https://gist.github.com/BIGBALLON/8a71d225eff18d88e469e6ea9b39cef4).

```
│imagenet/
├──train/
│  ├── n01440764
│  │   ├── n01440764_10026.JPEG
│  │   ├── n01440764_10027.JPEG
│  │   ├── ......
│  ├── ......
├──val/
│  ├── n01440764
│  │   ├── ILSVRC2012_val_00000293.JPEG
│  │   ├── ILSVRC2012_val_00002138.JPEG
│  │   ├── ......
│  ├── ......
```



### 2. VITA Models (polattnformer_s12 from POTTER is compared as baseline)

| Model    |  #Params | Image resolution | #MACs* | Top1 Acc| Download | Log |
| :---     |   :---:    |  :---: |  :---: |  :---:  |  :---:  | :---:  |
| poolattnformer_s12  |    12M     |   224  |  1.8G |  79.0  | [Refer to POTTER] |[Refer to POTTER] |
| VITA |   12M     |   224 | 1.8G | 85.4  | [here](https://drive.google.com/file/d/1aBJBxOX3zpCJ11rfAhanXIIL2DJk-HwR/view?usp=sharing)  |[log](https://drive.google.com/file/d/16nrzye1JDKRcJ--yCKjKuI-ENN-qlyim/view?usp=sharing) |




### 3. Validation

To evaluate our VITA models, run:

```bash
MODEL=poolattnformer_s12 # poolattnformer_{s12, s24, s36, m36, m48}
python3 validate.py /path/to/imagenet  --model $MODEL -b 128 \
  --pretrained # or --checkpoint /path/to/checkpoint --alt_embed_dim_pooling
```
You can also use arg `--no-alt-embed-dim-pooling` which evaluates the baseline model.


### 4. Train
We show how to train VITA on 8 GPUs. The relation between learning rate and batch size is lr=bs/1024*2e-3.
For convenience, assuming the batch size is 1024, then the learning rate is set as 2e-3 


```bash
MODEL=poolattnformer_s12 # poolattnformer_{s12, s24, s36, m36, m48}
DROP_PATH=0.1 # drop path rates [0.1, 0.1, 0.2, 0.3, 0.4] responding to model [s12, s24, s36, m36, m48]
CUDA_VISIBLE_DEVICES=0,1,2,3,4,5,6,7 ./distributed_train.sh 8 /path/to/imagenet \
  --model $MODEL -b 128 --lr 2e-3 --drop-path $DROP_PATH --apex-amp
```


## Acknowledgment
Our implementation is mainly based on the following codebases. We gratefully thank the authors for their wonderful works.

[POTTER](https://github.com/zczcwh/POTTER/tree/main),[PoolFormer](https://github.com/sail-sg/poolformer), [pytorch-image-models](https://github.com/rwightman/pytorch-image-models), [mmdetection](https://github.com/open-mmlab/mmdetection), [mmsegmentation](https://github.com/open-mmlab/mmsegmentation).


