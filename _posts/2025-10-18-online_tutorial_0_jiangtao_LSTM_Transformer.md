---
layout: post
title: Building LSTM and Transformer Models for Hydrologic Prediction
description: This tutorial provides a hands-on introduction to applying deep learning models—LSTM and Transformer—to hydrologic prediction using the CAMELS dataset. Participants will learn how to preprocess data, construct and train models, and evaluate their performance on rainfall–runoff tasks. No prior hydrology experience is required—just curiosity about deep learning for Earth science.
date: 2025-10-18
event_time: "10:00 - 11：00 AM"        
event_date: "Oct 18, 2025"
category: agenda
author: jiangtao
image:
video_embed:
tags: [Online Agenda]
featured: false
toc: false
---
## 📘 Overview

In this tutorial, we will explore how to use the **CAMELS (Catchment Attributes and Meteorology for Large-sample Studies)** dataset to build deep learning models for rainfall–runoff prediction.
You will learn how to preprocess hydrologic data, construct and train **LSTM** and **Transformer** models, and evaluate their predictive performance.

To make the tutorial lightweight and runnable on Colab (even without GPU),
we’ll use a **small subset (20 basins)** of CAMELS that has been pre-processed and stored as a NetCDF file.

> 💡 *The full CAMELS dataset can be downloaded and processed using the provided [Python scripts](https://zenodo.org/records/17087240)
> (`01.download_camels.py`, `02.prepare_camels.py`), which takes about 20 minutes.*

---

#### **Resources**
- [Google Colab Notebook](https://drive.google.com/file/d/1KVdT3tZ_Dn7hKa226MU8mOuNKWS1e8L8/view?usp=sharing)
- [LSTM Rainfall–Runoff Modeling Tutorial](https://zenodo.org/records/17216705)  
- [CAMELS Data Processing Pipeline](https://zenodo.org/records/17087240)

#### References  
> Liu, J., Bian, Y., Lawson, K., & Shen, C. (2024). [*Probing the limit of hydrologic predictability with the Transformer network.*](https://doi.org/10.1016/j.jhydrol.2024.131389) Journal of Hydrology, 637, 131389.

> Liu, J., Shen, C., O'Donncha, F., Song, Y., et al. (2025). [*From RNNs to Transformers: Benchmarking deep learning architectures for hydrologic prediction.*](https://doi.org/10.5194/egusphere-2025-1706) EGUsphere, 2025.

---
