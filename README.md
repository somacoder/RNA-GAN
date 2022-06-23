# Synthetic whole-slide imaging tile generation with gene expression profiles infused deep generative models

**Francisco Carrillo-Perez<sup>1,2</sup>, Marija Pizurica<sup>1,3</sup>, Michael G. Ozawa<sup>4</sup>, Hannes Vogel<sup>4</sup>, Robert B. West<sup>4</sup>, Christina S. Kong<sup>4</sup>, Jeanne Shen<sup>4</sup> and Olivier Gevaert<sup>1,5</sup>**

**<sup>1</sup> Stanford Center for Biomedical Informatics Research (BMIR), Stanford University, School of Medicine**

**<sup>2</sup> Department of Architecture and Computer Technology (ATC), University of Granada**

**<sup>3</sup> Internet technology and Data science Lab (IDLab), Ghent University**

**<sup>4</sup> Department of Pathology, Stanford University, School of Medicine**

**<sup>5</sup> Department of Biomedical Data Science, Stanford University, School of Medicine**

---

## Abstract

The acquisition of multi-modal biological data such as RNA sequencing and whole slide imaging (WSI) for the same sample has increased in recent years, enabling studying human biology from multiple angles. However, despite these emerging multi-modal efforts, for the majority of studies only one modality is typically available due to financial or logistical constraints. Given these difficulties, multi-modal data imputation and multi-modal synthetic data are appealing as a solution for the multi-modal data scarcity problem. Currently, most studies focus on generating a single modality (e.g. WSI), without leveraging the information provided by additional data modalities (e.g. gene expression profiles). In this work, we propose an approach to generate WSI tiles by using deep generative models infused with matched gene expression profiles. First, we train a variational autoencoder (VAE) that learns a latent representation of multi-tissue gene expression profiles, and we show that this model is able to generate realistic synthetic tiles. Then, we use this representation to infuse generative adversarial networks (GAN), generating lung and brain cortex tissue tiles with a new model that we call RNA-GAN. Tiles generated by RNA-GAN were preferred by expert pathologists in comparison to tiles generated using traditional GANs and RNA-GAN needs fewer training epochs to generate high-quality tiles. In addition, RNA-GAN was able to generalize to gene expression profiles outside of the training set and we show that the synthetic tiles can be used to train machine learning models. A web-based quiz is available for users to play a game distinguishing  real and synthetic tiles: [https://rna-gan.stanford.edu](https://rna-gan.stanford.edu) and the code for RNA-GAN is available: [https://github.com/gevaertlab/RNA-GAN](https://github.com/gevaertlab/RNA-GAN)

<img src="imgs/generation.png" alt="generation" width="1500"/>

## Web quiz

A [quiz](https://rna-gan.stanford.edu) is available to get a score on how well fake and real images are detected.

## Checkpoints

Checkpoints for the models can be downloaded [here](https://drive.google.com/drive/folders/1jvWPAaUyFuQYVK88E-hN9ccevljyzBk-?usp=sharing).
## Example usage

### betaVAE

**Training the model**

```bash
python3 betaVAE_training.py --seed 99 --config configs/betavae_tissues.json --log 1 --parallel 0
```
**Compute interpolation vectors**

```bash
python3 betaVAE_interpolation.py --seed 99 config --configs/betavae_tissues.json --log 0 --parallel 0
```
**Interpolating**

```bash
pythion3 betaVAE_sample.py --seed 99 --config configs/betavae_tissues.json --log 0 --parallel 0
```

### Normal GAN and RNA-GAN

**Normal GAN training**

```bash
python3 histopathology_gan.py --seed 99 --config configs/gan_run_brain.json --image_dir gan_generated_images/images_gan_brain --model_dir ./checkpoints/gan_brain --num_epochs 39 --gan_type dcgan --loss_type wgan --num_patches 600

python3 histopathology_gan.py --seed 99 --config configs/gan_run_lung.json --image_dir gan_generated_images/images_gan_lung --model_dir ./checkpoints/gan_lung --num_epochs 91 --gan_type dcgan --loss_type wgan --num_patches 600
```

**RNA-GAN training**

```bash
python3 histopathology_gan.py --seed 99 --config configs/gan_run_brain.json --image_dir gan_generated_images/images_rna-gan_brain --model_dir ./checkpoints/rna-gan_brain --num_epochs 24 --gan_type dcgan --loss_type wganvae --num_patches 600
python3 histopathology_gan.py --seed 99 --config configs/gan_run_lung.json --image_dir gan_generated_images/images_rna-gan_lung --model_dir ./checkpoints/rna-gan_lung --num_epochs 11 --gan_type dcgan --loss_type wganvae --num_patches 600
```

**Compute FID metrics**

```bash
# Real lung vs gan lung
python3 fid.py --checkpoint ./checkpoints/gan_lung.model --config configs/gan_run_lung.json \
        --sample_size 600 --vae --vae_checkpoint checkpoints/betavae_tissues.pt \
        --patient1 GTEX-15RJ7-0625.svs

# Real lung vs rna-gan lung
python3 fid.py --checkpoint ./checkpoints/rna-gan_lung.model --config configs/gan_run_lung.json \
        --sample_size 600 --vae --vae_checkpoint checkpoints/betavae_tissues.pt \
        --patient1 GTEX-15RJ7-0625.svs

# Gan lung vs rna-gan lung
python3 fid.py --checkpoint ./checkpoint/rna-gan_lung.model --checkpoint2 ./checkpoints/gan_lung.model --config configs/gan_run_lung.json \
        --sample_size 600 --vae --vae_checkpoint checkpoints/betavae_tissues.pt \
        --patient1 GTEX-15RJ7-0625.svs

# Gan vs Real brain
python3 fid.py --checkpoint ./checkpoints/gan_lung.model --config configs/gan_run_brain.json \
        --sample_size 600

# Real brain vs rna-gan
python3 fid.py --checkpoint ./checkpoints/rna-gan_lung.model --config configs/gan_run_brain.json \
        --sample_size 600 --vae --vae_checkpoint checkpoints/betavae_tissues.pt \
        --patient1 GTEX-1C6WA-3025.svs

# Gan brain vs gan lung
python3 fid.py --checkpoint ./checkpoints/gan_lung.model --config configs/gan_run_brain.json \
        --sample_size 600 --checkpoint2 ./checkpoints/gan_brain/gan_brain.model

# brain rna-gan vs gan lung
python3 fid.py --checkpoint2 ./checkpoints/gan_lung.model --checkpoint ./checkpoints/rna-gan_brain.model --config configs/gan_run_brain.json \
        --sample_size 600 --vae --vae_checkpoint checkpoints/betavae_tissues.pt \
        --patient1 GTEX-1C6WA-3025.svs

# lung rna-gan vs gan brain
python3 fid.py --checkpoint2 ./checkpoints/gan_brain.model --checkpoint ./checkpoints/rna-gan_lung.model --config configs/gan_run_lung.json \
        --sample_size 600 --vae --vae_checkpoint checkpoints/betavae_tissues.pt \
        --patient1 GTEX-15RJ7-0625.svs

# lung rna-gan vs  brain rna-gan 
python3 fid.py --checkpoint2 ./checkpoints/rna-gan_brain.model --checkpoint ./checkpoints/rna-gan_lung.model --config configs/gan_run_lung.json \
        --config2 configs/gan_run_brain.json --sample_size 600 --vae --vae_checkpoint checkpoints/betavae_tissues.pt \
        --patient1 GTEX-15RJ7-0625.svs --patient2 GTEX-1C6WA-3025.svs

#############

# Real brain vs gan brain
python3 fid.py --checkpoint ./checkpoints/gan_brain.model --config configs/gan_run_brain.json \
        --sample_size 600 \
        --patient1 GTEX-1C6WA-3025.svs

# Real brain vs rna-gan brain
python3 fid.py --checkpoint ./checkpoints/rna-gan_brain.model --config configs/gan_run_brain.json \
        --sample_size 600 --vae --vae_checkpoint checkpoints/betavae_tissues.pt \
        --patient1 GTEX-1C6WA-3025.svs

# Gan brain vs rna-gan brain
python3 fid.py --checkpoint ./checkpoints/rna-gan_brain.model --checkpoint2 ./checkpoints/gan_brain.model --config configs/gan_run_brain.json \
        --sample_size 600 --vae --vae_checkpoint checkpoints/betavae_tissues.pt \
        --patient1 GTEX-1C6WA-3025.svs

# brain Gan vs Real lung
python3 fid.py --checkpoint ./checkpoints/gan_brain.model --config configs/gan_run_lung.json \
        --sample_size 600

# Real lung vs rna-gan brain
python3 fid.py --checkpoint ./checkpoints/rna-gan_brain.model --config configs/gan_run_lung.json \
        --sample_size 600 --vae --vae_checkpoint checkpoints/betavae_tissues.pt \
        --patient1 GTEX-15RJ7-0625.svs

# lung rna-gan vs  brain rna-gan
python3 fid.py --checkpoint2 ./checkpoints/rna-gan_brain.model --checkpoint ./checkpoints/rna-gan_lung.model --config configs/gan_run_lung.json \
        --config2 configs/gan_run_brain.json --sample_size 600 --vae --vae_checkpoint checkpoints/betavae_tissues.pt \
        --patient1 GTEX-15RJ7-0625.svs --patient2 GTEX-1C6WA-3025.svs

```

**Image generation**

```bash
python3 generate_tissue_images.py --checkpoint ./checkpoints/rna-gan_lung.model --checkpoint2 ./checkpoints/gan_lung.model --config configs/gan_run_lung.json --sample_size 600 --vae --vae_checkpoint checkpoints/betavae_tissues.pt --patient1 GTEX-15RJ7-0625.svs

python3 generate_tissue_images.py --checkpoint ./checkpoints/rna-gan_brain.model --checkpoint2 ./checkpoints/gan_brain.model --config configs/gan_run_brain.json --sample_size 600 --vae --vae_checkpoint checkpoints/betavae_tissues.pt --patient1 GTEX-1C6WA-3025.svs

# From GEO series

python3 generate_tissue_image.py --checkpoint ./checkpoints/rna-gan_lung.model --config configs/gan_run_brain.json --sample_size 600 --vae --vae_checkpoint checkpoints/betavae_tissues.pt --rna_data GSE120795_lung_proteincoding.csv --random_patient

python3 generate_tissue_image.py --checkpoint ./checkpoints/rna-gan_brain.model --config configs/gan_run_brain.json --sample_size 600 --vae --vae_checkpoint checkpoints/betavae_tissues.pt --rna_data GSE120795_brain_proteincoding.csv --random_patient
```

**Training toy classification model**

```bash

# Using real samples
python3 wsi_model.py --patch_data_path images_form/ --csv_path real_toy_example.csv

# Using generated samples
python3 wsi_model.py --patch_data_path images_form/ --csv_path rna-gan_toy_example.csv
```

