# 🔬 TF-Nuclear-Translocation-Analysis

**Custom Python workflows for post-processing time-lapse cell tracking data and quantifying Dot6–GFP nuclear translocation in live yeast cells.**

This repository provides Python scripts for refining single-cell trajectories and quantitatively measuring stress-induced nuclear translocation of the Dot6–GFP transcription factor from time-lapse microscopy data. The scripts operate **after** image acquisition, segmentation, and tracking, which are assumed to be performed in Fiji/ImageJ with Trackmate-Cellpose.

---

## 📌 Scope and Purpose

* Post-process time-lapse single-cell tracking data
* Remove unreliable or incomplete cell trajectories
* Quantify nuclear localization using an intensity-based metric
* Compute a final **Dot6–GFP Nuclear Translocation Index** for strain-level comparison

---

## 🛠 Prerequisites

These scripts assume that the following preprocessing steps have already been completed, producing **time-aligned image stacks** with dimensions *(T, Y, X)*.

### External Image Processing Workflow

1. **Image Acquisition**

   * Time-lapse Z-stack live-cell microscopy
   * Channels: GFP fluorescence (Dot6–GFP) and Brightfield

2. **Segmentation**

   * Brightfield images segmented using **Cellpose (Cytoplasm 2.0)**
   * Approximate cell diameter: **5 μm**

3. **Tracking**

   * Segmentation masks imported into **Fiji/ImageJ**
   * Cell tracking performed with **TrackMate (with TrackMate-Cellpose)**
   * Tracker: *Simple LAP*
   * Max linking distance: **5 μm**
   * No splitting or merging enabled

The outputs of this stage should include:

* Intensity images (GFP)
* Corresponding label images with consistent cell IDs over time

---

## ⚙️ Analysis Workflow Overview

The analysis consists of **two sequential Python scripts**, each operating on the outputs of the previous step.

### Part I — Label Filtering

**Script:** `PartI_label_filtering.ipynb`

Purpose:

* Remove cell labels that:

  * Touch the image border
  * Appear or disappear during the time-lapse

Outcome:

* A filtered set of cell labels representing **complete, uninterrupted single-cell trajectories** suitable for quantitative analysis.

---

### Part II — Nuclearization Ratio Calculation and Indexing

**Script:** `nuclearization_ratio_calculation.ipynb`

Purpose:

* Quantify Dot6–GFP nuclear localization per cell and per time point
* Remove outlier cells based on intensity and ratio statistics
* Compute a final nuclear translocation index for population-level comparison

#### Nuclearization Ratio Calculation

For each cell at each time point (t), a nuclear signal score is computed as:

Mean intensity of brightest 5% of cell pixels / Median pixel intensity of the cell

* The **brightest 5% of pixels** are assumed to represent the nuclear-localized signal
* The **median cell intensity** provides a robust cytoplasmic baseline

---

#### Outlier Removal Criteria

Cells are excluded if **any** of the following conditions are met:

* Time-averaged nuclearization ratio is outside the **5th–95th percentile** of the population
* Time-averaged **mean fluorescence intensity** exceeds the **90th percentile**
  *(typically indicating debris, segmentation errors, or abnormally bright artifacts)*

---

#### Nuclear Translocation Index

* A **nuclear-localization event** is defined using a ratio threshold (default: **1.65**)
* This value corresponds to the **mean unstressed nuclearization ratio** in the reference condition
* The final index summarizes the frequency and magnitude of nuclear translocation events for quantitative strain comparison

> **Note:** Calculations exclude the first two time points (T1 and T2), corresponding to pre-stress baseline measurements.

---

## ▶️ Usage

### 1. Configure Input and Output Paths

Edit the script to define your data locations:

```python
image_directory = r'D:\Your\Path\To\Intensity\Images'
label_directory = r'D:\Your\Path\To\Label\Images'
output_directory = r'D:\Your\Path\To\Output\Results'
```

---

### 2. Adjust Key Parameters

```python
intensity_threshold_percentile = 90  # Mean intensity outlier threshold
nuclearization_threshold = 1.65      # Threshold for nuclear localization event
```

Modify these parameters if applying the pipeline to different reporters, imaging conditions, or experimental designs.

---

## 📊 Output

The pipeline generates:

* Filtered label sets with reliable trajectories
* Per-cell, per-time-point nuclearization ratios
* Summary statistics and final **Dot6–GFP Nuclear Translocation Index** values

These outputs are suitable for downstream statistical analysis and figure generation.

---

## 🧬 Intended Use

This repository is intended for researchers analyzing **live-cell transcription factor dynamics**, particularly in yeast, and can be adapted to other nuclear translocation assays with minimal modification.

---

## 📄 License and Citation

If you use or adapt this workflow in published work, please cite appropriately and acknowledge the original authors.

Contributions and issues are welcome.
