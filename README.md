# **Maximum-Suppression-Regularization (MaxSup)**

**Max Suppression (MaxSup)** retains the desired regularization effect of Label Smoothing (LS) while preserving the intra-class variation in the feature space. This boosts performance on classification and downstream tasks such as **linear transfer** and **image segmentation**.

---

## **Table of Contents**

1. [Improved Feature Representation for Better Transferability](#improved-feature-representation-for-better-transferability)  
   1.1 [Qualitative Evaluation](#qualitative-evaluation)  
   1.2 [Quantitative Evaluation](#quantitative-evaluation)  
2. [Train Vision Transformer with MaxSup](#train-vision-transformer-with-maxsup)  
   2.1 [Cache Feature for Faster Data Loading (Optional)](#cache-feature-for-faster-data-loading-optional)  
   2.2 [Prepare the Data and Annotation for the Cache Feature](#prepare-the-data-and-annotation-for-the-cache-feature)  
3. [Pretrained Weights](#pretrained-weights)  
4. [Training ConvNets with MaxSup](#training-convnets-with-maxsup)  
5. [Visualization of Logit Characteristics](#visualization-of-logit-characteristics)  

---

## **Improved Feature Representation for Better Transferability**

### **Qualitative Evaluation**

<p align="center">
   <img src="Improved_Feature.png" alt="drawing" width="1100"/>
</p>
<p align="center">
   <b>Figure 1:</b> MaxSup mitigates the reduced intra-class variation in Label Smoothing while preserving inter-class separability. Additionally, in Grad-CAM analysis, MaxSup highlights class-discriminative regions more effectively than Label Smoothing.
</p>

<p align="center">
   <img src="gradcam.png" alt="drawing" width="650"/>
</p>
<p align="center">
   <b>Figure 2:</b> We visualize the class activation map using GradCAM (Selvaraju et al., 2019) from Deit-Small models trained with MaxSup (2nd row), Label Smoothing (3rd row) and Baseline (4th row). The first row are original images. The results show that MaxSup training with MaxSup can reduce the distraction by non-target class, whereas Label Smoothing increases the model’s vulnerability to interference, causing the model partially or completely focusing on incorrect objects, due to the loss of richer information of individual samples.
</p>

### **Quantitative Evaluation**

| Methods                | Intra-Class Variation (Train) | Intra-Class Variation (Validation) | Inter-Class Separability (Train) | Inter-Class Separability (Validation) |
|------------------------|-------------------------------|------------------------------------|----------------------------------|----------------------------------------|
| **Baseline**           | 0.3114                        | 0.3313                             | 0.4025                           | 0.4451                                 |
| **Label Smoothing**    | 0.2632                        | 0.2543                             | 0.4690                           | 0.4611                                 |
| **Online Label Smoothing** | 0.2707                        | 0.2820                             | 0.5943                           | 0.5708                                 |
| **Zipf's Label Smoothing** | 0.2611                        | 0.2932                             | 0.5522                           | 0.4790                                 |
| **MaxSup**             | **0.2926**                    | **0.2998**                         | 0.5188                           | 0.4972                                 |

**Table 1:** Quantitative measures of feature representations for inter-class separability (indicating classification performance) and intra-class variation (indicating transferability), computed using ResNet-50 trained on ImageNet-1K. Although all methods reduce intra-class variation compared to the baseline, MaxSup exhibits the least reduction.

| Methods                | Linear Transfer Val. Acc |
|------------------------|--------------------------|
| **Baseline**           | 0.8143                  |
| **Label Smoothing**    | 0.7458                  |
| **MaxSup**             | **0.8102**              |

**Table 2:** The linear transfer performance of different methods, evaluated using multinomial logistic regression with l2 regularization on CIFAR-10. Despite improving ImageNet accuracy, Label Smoothing notably degrades transfer performance.

---

## **Train Vision Transformer with MaxSup**

We adopt [Deit](https://github.com/facebookresearch/deit) as the baseline model, and MaxSup is included in the `train_one_epoch` function of `engine.py`.

```bash
cd Deit
python train_with_MaxSup.sh
```

### **Cache Feature for Faster Data Loading (Optional)**

To accelerate the data loading procedure, we additionally implemented a feature which caches the compressed ImageNet dataset as a Zip file in RAM (adapted from [Swin-Transformer](https://github.com/microsoft/Swin-Transformer)). It significantly reduces data loading time with slow I/O speed and sufficient RAM, e.g., on a cluster in our case. It is activated by additionally providing `--cache` as an argument, as shown in the bash script.

### **Prepare the Data and Annotation for the Cache Feature**

1. **ZIP Archives**  
   Please run the following commands in the terminal to create the compressed files for the train and validation sets respectively:
   ```bash
   cd data/ImageNet
   zip -r train.zip train
   zip -r val.zip val
   ```

2. **Mapping Files**  
   Please download the `train_map.txt` and `val_map.txt` in the releases and put them under the same directory:
   ```
   data/ImageNet/
   ├── train_map.txt      # Training image paths and labels
   ├── val_map.txt        # Validation image paths and labels
   ├── train.zip          # Training images (compressed)
   └── val.zip            # Validation images (compressed)
   ```

   - **Training Map File (train_map.txt)**  
     - **Format**: `<class_folder>/<image_filename>\t<class_label>`
     - **Example entries**:
       ```
       ImageNet/train/n03146219/n03146219_8050.JPEG    0
       ImageNet/train/n03146219/n03146219_12728.JPEG   0
       ImageNet/train/n03146219/n03146219_9736.JPEG    0
       ImageNet/train/n03146219/n03146219_22069.JPEG   0
       ImageNet/train/n03146219/n03146219_5467.JPEG    0
       ```

   - **Validation Map File (val_map.txt)**  
     - **Format**: `<image_filename>\t<class_label>`
     - **Example entries**:
       ```
       ILSVRC2012_val_00000001.JPEG    65
       ILSVRC2012_val_00000002.JPEG    970
       ILSVRC2012_val_00000003.JPEG    230
       ```

   You should make sure:
   - Paths include the class folder structure.
   - Labels are zero-based integers.

---

## **Pretrained Weights**

Please find the pretrained weights as well as the training log in the releases **"checkpoint_deit"**.

---

## **Training ConvNets with MaxSup**

- The image classification results in the main paper refer to `Conv/ffcv` folder. See `README.md` there.  
- The additional image classification results in the appendix refer to `Conv/common_resnet`. See `README.md` there.


<p align="center">
  <img src="logit.png" alt="Logit Analysis" width="300"/>
</p>
