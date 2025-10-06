---
layout: post
title: International Soil Moisture Network (ISMN) Data Processing Pipeline
description: This repository provides an efficient pipeline to batch process, filter, and standardize soil moisture observation data from the International Soil Moisture Network (ISMN) into daily analysis-ready CSV files, including detailed site metadata.
date: 2025-9-16
event_time:
event_date:
category:
author: jiangtao
image: '/images/posts/ismn_data_processing.jpg'
video_embed:
tags: [Tutorial]
featured: false
toc: false
---

[Full code](https://zenodo.org/records/17127983)

## 🚀 Key Features

* ✅ **Automated Directory Scanning and Indexing**:

  * Recursive scanning of the ISMN root directory (organized by network → station → file).
  * Automatic caching of site metadata to accelerate future processing (`./tmp/info.csv`).

* ✅ **Advanced Multi-dimensional Filtering**:

  * Filter sites by continent, network (e.g., USCRN).
  * Depth-based filtering supporting three thresholds (`0.05`, `0.1`, `0.2` m ...).

* ✅ **Robust Data Quality Control**:

  * Observations outside the range `[0,1]` are flagged as missing (`NaN`).
  * Aggregates hourly data to daily averages.

* ✅ **Flexible Data Export**:

  * Annual CSV outputs (`soil_moisture.csv`) organized by year (rows: sites, columns: daily values).
  * Detailed site coordinates (`crd.csv`) and metadata (`info.csv`) included.

---

## 📏 Soil Depth Threshold Selection

The pipeline supports selecting soil moisture observations at three standard depth thresholds:

* **0.05 m**: selects data measured from surface to a depth of approximately 0.075 m.
* **0.1 m**: selects data measured between depths approximately 0.075 m to 0.150 m.
* **0.2 m**: selects data measured between depths approximately 0.150 m to 0.250 m.

These standard thresholds are set as:

```python
if depth_threshold == 0.05: 
    depth_bool = (depth_end <= 0.075) & (depth_str >= 0)
elif depth_threshold == 0.1:
    depth_bool = (depth_end <= 0.150) & (depth_str >= 0.075 + eps)
elif depth_threshold == 0.2:
    depth_bool = (depth_end <= 0.250) & (depth_str >= 0.15 + eps)
else:
    raise ValueError(f"You can add your custom depth_threshold {depth_threshold}")
```

To use different depth layers, modify this logic according to your requirements. Note that the current implementation does **not interpolate** between different depth layers. If interpolation between depth layers is needed, you can add the corresponding functionality based on your analysis needs.

---


## ⚙️ Environment Setup

### Python

**Python ≥ 3.9** recommended (**3.12** preferred).

### Dependencies

Create and activate an environment, then install dependencies:

```bash
conda create -n rainflow-env python=3.12 -y
conda activate rainflow-env

pip install numpy pandas tqdm
```

---

## 📥 How to Download ISMN Data

1. Log in at [https://ismn.earth/en/](https://ismn.earth/en/)
2. Go to **Data Access**
3. Select the networks (or choose all)
4. Select the desired date range (e.g., 2015-01-01 to 2025-08-31)
5. Click **Download**
6. Choose **“Variables stored in separate files (Header+values formatted) (zipped)”** and select **Gap filling**
7. Click **Download**
8. Unzip into a directory, e.g.: `path_directory/ISMN_20150101_20250831`

---

## 📂 Expected Input Layout (example)

```
ISMN_20150101_20250831/
  <NETWORK_A>/
    <STATION_ID_1>/
      *.txt ...
    <STATION_ID_2>/
      *.txt ...
  <NETWORK_B>/
    ...
```

---

## 💻 Quickstart (Command Line)

**Single network (USCRN):**

```bash
python extract_international_soil_moisture_network.py \
  --input-dir path_directory/ISMN_20150101_20250831 \
  --output-dir ./formatted_ISMN \
  --start-date 2015-01-01 --end-date 2020-12-31 \
  --depth-threshold 0.05 \
  --network USCRN
```

**Multiple networks (USCRN, COSMOS):**

```bash
python extract_international_soil_moisture_network.py \
  --input-dir path_directory/ISMN_20150101_20250831 \
  --output-dir ./formatted_ISMN \
  --start-date 2015-01-01 --end-date 2020-12-31 \
  --depth-threshold 0.05 \
  --network USCRN COSMOS
```

**One or more continents (Europe and Asia):**

```bash
python extract_international_soil_moisture_network.py \
  --input-dir path_directory/ISMN_20150101_20250831 \
  --output-dir ./formatted_ISMN \
  --start-date 2015-01-01 --end-date 2020-12-31 \
  --depth-threshold 0.05 \
  --continent Europe Asia
```

**All continents (processed separately in one run):**

```bash
python extract_international_soil_moisture_network.py \
  --input-dir path_directory/ISMN_20150101_20250831 \
  --output-dir ./formatted_ISMN \
  --start-date 2015-01-01 --end-date 2020-12-31 \
  --depth-threshold 0.05 \
  --continent ALL
```

**Global extraction (all continents merged):**

```bash
python extract_international_soil_moisture_network.py \
  --input-dir path_directory/ISMN_20150101_20250831 \
  --output-dir ./formatted_ISMN \
  --start-date 2015-01-01 --end-date 2020-03-31 \
  --depth-threshold 0.05 \
  --global
```

---

## 🌍 Continent Keys

Use the following exact keys with `--continent`:
`Africa`, `Asia`, `Australia`, `Europe`, `North_America`, `Oceania`, `South_America`, or `ALL`.

---


## 📦 Output Structure

For each selection (per network, per continent, or `GLOBAL`), the script writes outputs under the specified `--output-dir` in a dedicated subfolder.

Typical contents:

* **Annual data**: one CSV per year named `soil_moisture.csv` with

  * Rows: stations
  * Columns: daily aggregated soil moisture values
* **Coordinates**: `crd.csv` (station coordinates and IDs)
* **Metadata**: `info.csv` (site‑level attributes)

> **Missing values: -9999**

Example tree (for a network run):

```
formatted_ISMN/
  USCRN/
    2015/soil_moisture.csv
    2016/soil_moisture.csv
    ...
    crd.csv
    info.csv
```

---


## 📜 Citation

If you use this pipeline in your research, please cite:

> Liu, J., Rahmani, F., Lawson, K., & Shen, C. (2022). A multiscale deep learning model for soil moisture integrating satellite and in situ data. Geophysical Research Letters, 49(7), e2021GL096847. [https://doi.org/10.1029/2021GL096847](https://doi.org/10.1029/2021GL096847)

> Liu, J., Hughes, D., Rahmani, F., Lawson, K., & Shen, C. (2023). Evaluating a global soil moisture dataset from a multitask model (GSM3 v1.0) with potential applications for crop threats. Geoscientific Model Development, 16(5), 1553–1567. [https://doi.org/10.5194/gmd-16-1553-2023](https://doi.org/10.5194/gmd-16-1553-2023)

---

## 🙌 Acknowledgments

* [International Soil Moisture Network (ISMN)](https://ismn.earth/en/)
* [ChatGPT](https://chat.openai.com/) for assistance in refining documentation.

---

## 📄 License

This software is licensed under **CC BY‑NC 4.0**.
Academic and research use is explicitly permitted.
