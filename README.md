# Deep-Learning-Driven-Breast-Cancer-Detection
Deep Learning-Driven Breast Cancer Detection using breast histopathology images dataset

## A Diagnostic Pipeline from Histopathology Patch Classification to Whole-Slide Patient Prediction

## 📌 Project Overview

This project investigates the use of deep learning to detect **invasive
ductal carcinoma (IDC)** in breast cancer histopathology images.

The study uses the **Breast Histopathology Images dataset**, containing
**277,524 RGB patches** extracted from whole-slide images (WSIs). Each
patch is classified as IDC-negative or IDC-positive.

The broader goal is to develop a reproducible computational pathology
pipeline that can progress from **patch-level classification toward
whole-slide and patient-level prediction**.

## 🎯 Research Objectives

The project investigates whether a deep learning pipeline can:

1.  Classify 50 × 50 histopathology patches as IDC-positive or
    IDC-negative.
2.  Generalize to patients not represented in training.
3.  Compare convolutional and attention-based approaches.
4.  Assess the use of Vision Transformers on small histopathology
    patches.
5.  Identify model failure modes that may be hidden by conventional
    training metrics.
6.  Provide a foundation for future whole-slide patient-level
    prediction.

## 📊 Dataset

  Characteristic              Description
  --------------------------- -----------------
  Total patches               277,524
  Patch size                  50 × 50 pixels
  Image channels              RGB
  IDC-negative                198,738 (71.6%)
  IDC-positive                78,786 (28.4%)
  Original WSIs               162
  Patient-level directories   279
  Magnification               40×

The small patch size makes the task challenging because each image
contains limited tissue context.

## 🔬 Methodology

### Patient-Stratified Splitting

The data were split at the **patient level rather than the patch
level**. This prevents patches from the same patient appearing in both
training and validation sets and reduces patient-specific information
leakage.

The split produced:

-   **217,034 training patches**
-   **60,490 validation patches**

Positive-class rates were approximately 28.6% and 27.6%, respectively.

### Preprocessing and Augmentation

Images were:

-   Resized from 50 × 50 to 224 × 224 pixels.
-   Rescaled to \[0, 1\].
-   Augmented during training using rotations, shifts, shear, zoom, and
    horizontal/vertical flips.

Macenko stain normalization was planned but could not be executed
because the required dependency was unavailable in the project
environment.

## 🤖 Models

### 1. Baseline CNN

A CNN trained from scratch with three convolutional blocks, max pooling,
a 128-unit dense layer, dropout, and a sigmoid output.

### 2. EfficientNetB0

An ImageNet-pretrained EfficientNetB0 with a frozen convolutional
backbone and a trainable classification head.

### 3. Vision Transformer (ViT-Base/16)

An ImageNet-21k-pretrained ViT-Base/16 fine-tuned end-to-end. The model
uses 16 × 16 image patches, 196 tokens, 12 transformer layers, 12
attention heads, and a hidden size of 768.

## ⚖️ Class Imbalance

The dataset is approximately 2.5:1 in favour of IDC-negative patches.

Two approaches were explored:

-   **Focal loss** for the CNN and EfficientNetB0.
-   **Majority-class undersampling** for the ViT.

The ViT training data were balanced by reducing the negative class to
78,786 patches, producing 157,572 patches before the final
training/validation allocation.

## 📏 Evaluation Metrics

Because of class imbalance and the clinical importance of different
error types, the study evaluated:

-   Accuracy
-   Precision
-   Recall / Sensitivity
-   Specificity
-   F1 score
-   Balanced accuracy
-   Matthews correlation coefficient (MCC)
-   Confusion matrix
-   Reliability diagram

Raw accuracy was not treated as the sole measure of performance.

# 📈 Results

## Vision Transformer Performance

The ViT-Base/16 was evaluated on **34,176 patches from held-out
patients**.

  Metric                   ViT-Base/16
  ---------------------- -------------
  Accuracy                      0.8416
  Precision                     0.8474
  Recall / Sensitivity          0.8244
  Specificity                   0.8582
  F1 Score                      0.8357
  Balanced Accuracy             0.8413
  MCC                           0.6832

Key reported results:

-   **F1 = 0.836**
-   **Balanced Accuracy = 0.841**
-   **MCC = 0.683**

## 🔎 Failure Analysis

A major finding was the silent collapse of both the baseline CNN and
EfficientNetB0.

Both models converged to predicting **IDC-positive for every input**.
Their recall remained 1.000, while precision and accuracy reflected the
positive-class base rate and AUC-ROC was 0.500.

The cause was an incorrectly implemented focal loss. The modulating
factor was applied to the predicted probability rather than the
probability of the **true class**.

This allowed the optimizer to reduce the loss toward zero even when
negative examples were classified incorrectly.

### Key lesson

> A decreasing training loss is not, by itself, evidence that a model is
> learning useful representations.

For imbalanced medical classification, multiple metrics should be
monitored from the beginning of training.

## 🧠 Interpretation

The report does **not** establish that the ViT outperforms CNNs or
EfficientNetB0. The convolutional experiments were invalid because of
the focal-loss implementation error.

Therefore, the ViT result should be interpreted as a **successful
working experiment**, not as proof of a controlled three-model
architectural comparison.

## 🏥 Clinical and Computational Relevance

Automated detection of suspicious tumour regions could potentially
support digital pathology workflows by helping identify areas of
interest within very large whole-slide images.

The present study operates primarily at the **patch level**. The
intended progression is:

**Histopathology image → Patch classification → Slide-level aggregation
→ Patient-level prediction**

The current work provides a foundation for that pipeline but does not
yet provide validated patient-level diagnostic predictions.

# ⚠️ Limitations

1.  **CNN/EfficientNet failure:** The convolutional experiments were
    affected by the incorrect focal-loss implementation.
2.  **No slide-level aggregation:** Whole-slide heatmaps and
    patient-level metrics were not completed.
3.  **Stain normalization:** Macenko normalization could not be
    executed.
4.  **Undersampling:** A substantial number of negative patches were
    discarded for the ViT experiment.
5.  **Calibration:** The reliability diagram indicated under-confidence,
    so scores should be calibrated before being interpreted as
    probabilities.
6.  **External validation:** Further testing on independent datasets and
    institutions would be needed before clinical deployment.

# 🚀 Future Work

### 1. Correct the Focal Loss

Rerun the CNN and EfficientNetB0 experiments with the published
focal-loss formulation.

### 2. Complete Model Comparison

After obtaining valid results, use appropriate statistical comparisons
such as McNemar's test and bootstrapped confidence intervals.

### 3. Whole-Slide Reconstruction

Use patch coordinates to reconstruct spatial tumour probability maps and
whole-slide heatmaps.

### 4. Multiple Instance Learning

Investigate attention-based Multiple Instance Learning for aggregating
patch information into patient-level predictions.

### 5. Explainable AI

Use methods such as Grad-CAM to identify tissue regions contributing to
model predictions.

### 6. Stain Normalization

Evaluate whether Macenko normalization improves generalization across
slides.

### 7. Probability Calibration

Investigate temperature scaling or related methods before interpreting
model scores as probabilities.

# 🛠️ Methods and Technologies

-   Python
-   Deep Learning
-   Convolutional Neural Networks
-   EfficientNet
-   Vision Transformers
-   Transfer Learning
-   Image Augmentation
-   Patient-Level Data Splitting
-   Class-Imbalance Handling
-   Focal Loss
-   Undersampling
-   Histopathology Image Analysis
-   Model Evaluation
-   Failure Analysis
-   Reliability Analysis


# 📁 Suggested Repository Structure

``` text
breast-cancer-deep-learning/
├── README.md
├── data/
│   └── README.md
├── notebooks/
│   ├── data_preprocessing.ipynb
│   ├── model_training.ipynb
│   └── model_evaluation.ipynb
├── src/
│   ├── preprocessing/
│   ├── models/
│   ├── training/
│   └── evaluation/
├── results/
│   ├── confusion_matrix/
│   ├── reliability_diagram/
│   └── metrics/
├── figures/
└── requirements.txt
```


# 📌 Summary

This project developed a patient-stratified deep learning pipeline for
IDC detection using **277,524 histopathology patches**.

The working Vision Transformer experiment achieved:

-   **F1 Score: 0.836**
-   **Balanced Accuracy: 0.841**
-   **MCC: 0.683**

The project also documented an important negative result: an incorrectly
implemented focal loss caused the CNN and EfficientNetB0 experiments to
silently collapse into single-class predictions.

Overall, the project demonstrates the importance of:

-   Preventing patient-level data leakage
-   Correctly implementing loss functions
-   Using class-imbalance-aware metrics
-   Monitoring multiple metrics during training
-   Transparently reporting failed experiments
-   Avoiding unsupported claims when experimental conditions differ

The next stage is to correct the convolutional experiments and extend
the pipeline from **patch-level classification toward whole-slide and
patient-level prediction**.
