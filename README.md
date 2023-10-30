





<img src='imgs/teaser_720.gif' align="right" width=360>

<br><br><br><br>



## Image to Image Translation using Conditional GANs (Pix2PixHD).
- Our landscape Painting to Realistic Image results
- Landscape painting on the left, Generated Images on the Middle, Actual landscape on the Right
<p align='center'>  
   <img src='checkpoints/monet/web/images/epoch192_input_label.jpg' width='250'/>

  <img src='checkpoints/monet/web/images/epoch192_synthesized_image.jpg' width='250'/>
   <img src='checkpoints/monet/web/images/epoch192_real_image.jpg' width='250'/>

   
</p>
<p align='center'>  
   <img src='checkpoints/monet/web/images/epoch186_input_label.jpg' width='250'/>

  <img src='checkpoints/monet/web/images/epoch186_synthesized_image.jpg' width='250'/>
  <img src='checkpoints/monet/web/images/epoch186_real_image.jpg' width='250'/>

</p>
<p align='center'>  
   <img src='checkpoints/monet/web/images/epoch172_input_label.jpg' width='250'/>

  <img src='checkpoints/monet/web/images/epoch172_synthesized_image.jpg' width='250'/>
  <img src='checkpoints/monet/web/images/epoch172_real_image.jpg' width='250'/>

</p>
<p align='center'>  
   <img src='checkpoints/monet/web/images/epoch197_input_label.jpg' width='250'/>

  <img src='checkpoints/monet/web/images/epoch197_synthesized_image.jpg' width='250'/>
  <img src='checkpoints/monet/web/images/epoch197_real_image.jpg' width='250'/>

</p>
## Prerequisites
- Linux or macOS
- Python 2 or 3
- NVIDIA GPU (11G memory or larger) + CUDA cuDNN

## Getting Started
### Installation
- Install PyTorch and dependencies from http://pytorch.org
- Install python libraries [dominate](https://github.com/Knio/dominate).
```bash
pip install dominate
```
- Clone this repo:
```bash
git clone https://github.com/Nisnab/pix2pixHD
cd pix2pixHD
```

### Dataset
- We use the monet styled landscape dataset from kaggle. To train a model on the full dataset, please download it from the [official website](https://www.kaggle.com/shcsteven/paired-landscape-and-monetstylised-image).
After downloading, please put it under the `datasets/monet` folder in the same way the example images are provided.
Folder "train_A" contains actual images of 512*256.
Folder "train_B" contains painting images of 512*256.

###Image Preprocessing
The original training images were reduced to 512*256.You can use "DataResize.ipynb" to resize it as per your needs.

### Training
- I have trained my images as per this configuration. You can do your changes as per your requirement:
```bash
#!./scripts/train_512p.sh
python train.py --label_nc 0 --no_instance --name monet --dataroot /home/nisnab/workspace/pix2pixHD/datasets/monet --save_epoch_freq 50
```
- To view training results, please checkout intermediate results in `./checkpoints/monet/web/index.html`.
- For better visibility, please copy the URL of `index.html` and paste it in `https://htmlpreview.github.io/`
If you have tensorflow installed, you can see tensorboard logs in `./checkpoints/monet/logs` by adding `--tf_log` to the training scripts.

### Multi-GPU training
- Train a model using multiple GPUs (`bash ./scripts/train_512p_multigpu.sh`):
```bash
#!./scripts/train_512p_multigpu.sh
python train.py --name label2city_512p --batchSize 8 --gpu_ids 0,1,2,3,4,5,6,7
```
Note: this is not tested and we trained our model using single GPU only. Please use at your own discretion.

### Training with Automatic Mixed Precision (AMP) for faster speed
- To train with mixed precision support, please first install apex from: https://github.com/NVIDIA/apex
- You can then train the model by adding `--fp16`. For example,
```bash
#!./scripts/train_512p_fp16.sh
python -m torch.distributed.launch train.py --name label2city_512p --fp16
```
In our test case, it trains about 80% faster with AMP on a Volta machine.

### Training at full resolution
- To train the images at full resolution (2048 x 1024) requires a GPU with 24G memory (`bash ./scripts/train_1024p_24G.sh`), or 16G memory if using mixed precision (AMP).
- If only GPUs with 12G memory are available, please use the 12G script (`bash ./scripts/train_1024p_12G.sh`), which will crop the images during training. Performance is not guaranteed using this script.

### Training with your own dataset
- If you want to train with your own dataset, please generate label maps which are one-channel whose pixel values correspond to the object labels (i.e. 0,1,...,N-1, where N is the number of labels). This is because we need to generate one-hot vectors from the label maps. Please also specity `--label_nc N` during both training and testing.
- If your input is not a label map, please just specify `--label_nc 0` which will directly use the RGB colors as input. The folders should then be named `train_A`, `train_B` instead of `train_label`, `train_img`, where the goal is to translate images from A to B.
- If you don't have instance maps or don't want to use them, please specify `--no_instance`.
- The default setting for preprocessing is `scale_width`, which will scale the width of all training images to `opt.loadSize` (1024) while keeping the aspect ratio. If you want a different setting, please change it by using the `--resize_or_crop` option. For example, `scale_width_and_crop` first resizes the image to have width `opt.loadSize` and then does random cropping of size `(opt.fineSize, opt.fineSize)`. `crop` skips the resizing step and only performs random cropping. If you don't want any preprocessing, please specify `none`, which will do nothing other than making sure the image is divisible by 32.

## More Training/Test Details
- Flags: see `options/train_options.py` and `options/base_options.py` for all the training flags; see `options/test_options.py` and `options/base_options.py` for all the test flags.
- Instance map: we take in both label maps and instance maps as input. If you don't want to use instance maps, please specify the flag `--no_instance`.







### Testing
- A few example Cityscapes test images are included in the `datasets` folder.
- Please download the pre-trained landscapes model from [here](https://drive.google.com/drive/folders/1XUpxWIM3TMxlbeuPau91ihj6DiWrtTd6?usp=sharing) (google drive link), and put it under `./checkpoints/monet/`
- Test the model (`bash ./scripts/test_1024p.sh`):
```bash
#!./scripts/test_1024p.sh
python test.py --name label2city_1024p --netG local --ngf 32 --resize_or_crop none
```
The test results will be saved to a html file here: `./results/label2city_1024p/test_latest/index.html`.

More example scripts can be found in the `scripts` directory.

 
## Acknowledgments
This code borrows heavily from [pix2pixHD
](https://github.com/NVIDIA/pix2pixHD).
