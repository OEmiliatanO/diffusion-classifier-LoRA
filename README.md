<div align="center">

<!-- TITLE -->
# **Can LoRA help the diffusion classifier?**

This is **not** the official implementation of the ICCV 2023 paper, 
[Your Diffusion Model is Secretly a Zero-Shot Classifier](https://arxiv.org/abs/2303.16203) by Alexander Li, Mihir Prabhudesai, Shivam Duggal, Ellis Brown, and Deepak Pathak.

</div>

<!-- DESCRIPTION -->
## Abstract

We want to know that can LoRA make the diffusion classifier a dynamic classifier?  
This means, after pre-training, if we want the classifier be able to identify some other objects that is not in the training data, 
the diffusion classifier won't need to be re-trained, just add LoRA. And that means the ability to identify object can be 
modularized. 

## Installation
Create a conda environment with the following command:
```bash
conda env create -f environment.yml
```
If this takes too long, `conda config --set solver libmamba` sets conda to use the `libmamba` solver and could speed up installation.

## Zero-shot Classification with Stable Diffusion

```bash
python eval_prob_adaptive.py --dataset cifar10 --split test --n_trials 1 \
  --to_keep 5 1 --n_samples 50 500 --loss l1 \
  --prompt_path prompts/cifar10_prompts.csv
```
This command reads potential prompts from a csv file and evaluates the epsilon prediction loss for each prompt using Stable Diffusion.
This should work on a variety of GPUs, from as small as a 2080Ti or 3080 to as large as a 3090 or A6000. 
Losses are saved separately for each test image in the log directory. For the command above, the log directory is `data/cifar10/v2-0_1trials_5_1keep_50_500samples_l1`. Accuracy can be computed by running:
```bash
python scripts/print_acc.py data/cifar10/v2-0_1trials_5_1keep_50_500samples_l1
```

Commands to run Diffusion Classifier on each dataset are [here](commands.md). 
If evaluation on your use case is taking too long, there are a few options: 
1. Parallelize evaluation across multiple workers. Try using the `--n_workers` and `--worker_idx` flags.
2. Play around with the evaluation strategy (e.g. `--n_samples` and `--to_keep`).
3. Evaluate on a smaller subset of the dataset. Saving a npy array of test set indices and using the `--subset_path` flag can be useful for this.

### Evaluating on your own dataset
1. Create a csv file with the prompts that you want to evaluate, making sure to match up the correct prompts with the correct class labels. See `scripts/write_cifar10_prompts.py` for an example. Note that you can use multiple prompts per class.
2. Run the command above, changing the `--dataset` and `--prompt_path` flags to match your use case.
3. Play around with the evaluation strategy on a small subset of the dataset to reduce evaluation time.


## Standard ImageNet Classification with Class-conditional Diffusion Models
### Additional installations
Within the `diffusion-classifier` folder, download the DiT repository
```bash
git clone git@github.com:facebookresearch/DiT.git
````

### Running Diffusion Classifier
First, save a consistent set of noise (epsilon) that will be used for all image-class pairs:
```bash
python scripts/save_noise.py --img_size 256
```
Then, compute and save the epsilon-prediction error for each class:
```bash
python eval_prob_dit.py  --dataset imagenet --split test \
  --noise_path noise_256.pt --randomize_noise \
  --batch_size 32 --cls CLS --t_interval 4 --extra dit256 --save_vb
```
For example, for ImageNet, this would need to be run with CLS from 0 to 999. 
This is currently a very expensive process, so we recommend using the `--subset_path` command to evaluate on a smaller subset of the dataset. 
We also plan on releasing an adaptive version that greatly reduces the computation time per test image.

Finally, compute the accuracy using the saved errors:
```bash
python scripts/print_dit_acc.py data/imagenet_dit256 --dataset imagenet
``` 
We show the commands to run DiT on all ImageNet variants [here](commands.md). 

## Compositional Reasoning on Winoground with Stable Diffusion
To run Diffusion Classifier on Winoground:
First, save a consistent set of noise (epsilon) that will be used for all image-caption pairs:
```bash
python scripts/save_noise.py --img_size 512
```
Then, evaluate on Winoground:
```bash
python run_winoground.py --model sd --version 2-0 --t_interval 1 --batch_size 32 --noise_path noise_512.pt --randomize_noise --interpolation bicubic
```
To run CLIP or OpenCLIP baselines:
```bash
python run_winoground.py --model clip --version ViT-L/14
python run_winoground.py --model openclip --version ViT-H-14
```

## Acknowledgment

Thanks for the authors of the origin paper: [Your Diffusion Model is Secretly a Zero-Shot Classifier](https://arxiv.org/abs/2303.16203)

```bibtex
@misc{li2023diffusion,
      title={Your Diffusion Model is Secretly a Zero-Shot Classifier}, 
      author={Alexander C. Li and Mihir Prabhudesai and Shivam Duggal and Ellis Brown and Deepak Pathak},
      year={2023},
      eprint={2303.16203},
      archivePrefix={arXiv},
      primaryClass={cs.LG}
}
```
