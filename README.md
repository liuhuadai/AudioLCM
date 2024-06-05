# AudioLCM: Text-to-Audio Generation with Latent Consistency Models

#### Huadai Liu, Rongjie Huang, Yang Liu, Hengyuan Cao, Jialei Wang, Xize Cheng, Siqi Zheng, Zhou Zhao

PyTorch Implementation of [AudioLCM]: a efficient and high-quality text-to-audio generation with latent consistency model.

[![arXiv](https://img.shields.io/badge/arXiv-Paper-<COLOR>.svg)](https://arxiv.org/abs/2406.00356v1)
[![Hugging Face](https://img.shields.io/badge/%F0%9F%A4%97%20Hugging%20Face-blue)](https://huggingface.co/spaces/AIGC-Audio/AudioLCM)
[![GitHub Stars](https://img.shields.io/github/stars/liuhuadai/AudioLCM?style=social)](https://github.com/liuhuadai/AudioLCM)

We provide our implementation and pretrained models as open source in this repository.

Visit our [demo page](https://audiolcm.github.io/) for audio samples.

[AudioLCM HuggingFace Space](https://huggingface.co/spaces/AIGC-Audio/AudioLCM) 

## News
<!-- - Jan, 2023: **[Make-An-Audio](https://arxiv.org/abs/2207.06389)** submitted to arxiv. -->
- June, 2024: **[AudioLCM]** released in Githu and HuggingFace. 

## Quick Started
We provide an example of how you can generate high-fidelity samples quickly using AudioLCM.

To try on your own dataset, simply clone this repo in your local machine provided with NVIDIA GPU + CUDA cuDNN and follow the below instructions.


### Support Datasets and Pretrained Models

Simply download the weights from [Huggingface](https://huggingface.co/liuhuadai/AudioLCM).
<!-- Download bert-base-uncased weights from [Hugging Face](https://huggingface.co/google-bert/bert-base-uncased). Down load t5-v1_1-large weights from [Hugging Face](https://huggingface.co/google/t5-v1_1-large). Download CLAP weights from [Hugging Face](https://huggingface.co/microsoft/msclap/blob/main/CLAP_weights_2022.pth).  -->

```
Download:
    audiolcm.ckpt and put it into ./ckpts  
    BigVGAN vocoder and put it into ./vocoder/logs/bigvnat16k93.5w  
    t5-v1_1-large and put it into ./ldm/modules/encoders/CLAP
    bert-base-uncased and put it into ./ldm/modules/encoders/CLAP
    CLAP_weights_2022.pth and put it into ./wav_evaluation/useful_ckpts/CLAP
```
<!-- The directory structure should be:
```
useful_ckpts/
├── bigvgan
│   ├── args.yml
│   └── best_netG.pt
├── CLAP
│   ├── config.yml
│   └── CLAP_weights_2022.pth
└── maa1_full.ckpt
``` -->


### Dependencies
See requirements in `requirement.txt`:

## Inference with pretrained model
```bash
python scripts/txt2audio_for_lcm.py  --ddim_steps 2 -b configs/audiolcm.yaml --sample_rate 16000 --vocoder-ckpt  vocoder/logs/bigvnat16k93.5w --outdir results --test-dataset audiocaps  -r ckpt/audiolcm.ckpt
```
# Train
## dataset preparation
We can't provide the dataset download link for copyright issues. We provide the process code to generate melspec.  
Before training, we need to construct the dataset information into a tsv file, which includes name (id for each audio), dataset (which dataset the audio belongs to), audio_path (the path of .wav file),caption (the caption of the audio) ,mel_path (the processed melspec file path of each audio). We provide a tsv file of audiocaps test set: ./audiocaps_test_16000_struct.tsv as a sample.
### generate the melspec file of audio
Assume you have already got a tsv file to link each caption to its audio_path, which mean the tsv_file have "name","audio_path","dataset" and "caption" columns in it.
To get the melspec of audio, run the following command, which will save mels in ./processed
```bash
python ldm/data/preprocess/mel_spec.py --tsv_path tmp.tsv
```
Add the duration into the tsv file
```bash
python ldm/data/preprocess/add_duration.py
```
## Train variational autoencoder
Assume we have processed several datasets, and save the .tsv files in data/*.tsv . Replace **data.params.spec_dir_path** with the **data**(the directory that contain tsvs) in the config file. Then we can train VAE with the following command. If you don't have 8 gpus in your machine, you can replace --gpus 0,1,...,gpu_nums
```bash
python main.py --base configs/train/vae.yaml -t --gpus 0,1,2,3,4,5,6,7
```
The training result will be save in ./logs/
## train latent diffsuion
After Trainning VAE, replace model.params.first_stage_config.params.ckpt_path with your trained VAE checkpoint path in the config file.
Run the following command to train Diffusion model
```bash
python main.py --base configs/autoencoder1d.yaml -t  --gpus 0,1,2,3,4,5,6,7
```
The training result will be save in ./logs/
# Evaluation
## generate audiocaps samples
```bash
python scripts/txt2audio_for_lcm.py  --ddim_steps 2 -b configs/audiolcm.yaml --sample_rate 16000 --vocoder-ckpt  vocoder/logs/bigvnat16k93.5w --outdir results --test-dataset audiocaps  -r ckpt/audiolcm.ckpt
```

## calculate FD,FAD,IS,KL
install [audioldm_eval](https://github.com/haoheliu/audioldm_eval) by
```bash
git clone git@github.com:haoheliu/audioldm_eval.git
```
Then test with:
```bash
python scripts/test.py --pred_wavsdir {the directory that saves the audios you generated} --gt_wavsdir {the directory that saves audiocaps test set waves}
```
## calculate Clap_score
```bash
python wav_evaluation/cal_clap_score.py --tsv_path {the directory that saves the audios you generated}/result.tsv
```


## Acknowledgements
This implementation uses parts of the code from the following Github repos:
[Make-An-Audio](https://github.com/Text-to-Audio/Make-An-Audio)
[CLAP](https://github.com/LAION-AI/CLAP),
[Stable Diffusion](https://github.com/CompVis/stable-diffusion),
as described in our code.

## Citations ##
If you find this code useful in your research, please consider citing:
```bibtex
@misc{liu2024audiolcm,
      title={AudioLCM: Text-to-Audio Generation with Latent Consistency Models}, 
      author={Huadai Liu and Rongjie Huang and Yang Liu and Hengyuan Cao and Jialei Wang and Xize Cheng and Siqi Zheng and Zhou Zhao},
      year={2024},
      eprint={2406.00356},
      archivePrefix={arXiv},
      primaryClass={eess.AS}
}
```

# Disclaimer ##
Any organization or individual is prohibited from using any technology mentioned in this paper to generate someone's speech without his/her consent, including but not limited to government leaders, political figures, and celebrities. If you do not comply with this item, you could be in violation of copyright laws.
