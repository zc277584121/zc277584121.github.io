---
layout: post
title:  "Welcome to Jekyll!"
---


## 1.Prepare the dataset

The **[Caltech-UCSD Birds-200-2011 (CUB-200-2011)](http://www.vision.caltech.edu/visipedia/CUB-200-2011.html)** dataset is the most widely-used dataset for fine-grained visual categorization task.
* Number of categories: 200
* Number of images: 11,788
* Annotations per image: 15 Part Locations, 312 Binary Attributes, 1 Bounding Box

And here is a [Benchmarks](https://paperswithcode.com/dataset/cub-200-2011) about this dataset you can refer to.
First, you should download it to your local path from [Images and annotations](https://drive.google.com/file/d/1hbzc_P1FuxMkcabkgn9ZKinBwW683j45/view).
When you downloaded it, you can see this structure.
```
├── 1.jpg
├── 2.jpg
├── 3.jpg
├── README.md
└── data
    ├── CUB_200_2011.tgz
    └── segmentations.tgz
```
Just extract `CUB_200_2011.tgz` and rename folder `images` to `images_orig`.

    cd data
    tar zxvf CUB_200_2011.tgz
    cd CUB_200_2011
    mv images images_orig
    pwd

```
/path/to/your/dataset/Birds-200-2011/data/CUB_200_2011
```
**Use this path as your root_dir.**
