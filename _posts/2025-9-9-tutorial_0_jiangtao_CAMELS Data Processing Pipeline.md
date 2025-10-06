---
layout: post
title: CAMELS Data Processing Pipeline
description: This repository provides an efficient pipeline to automatically download, prepare, and standardize the CAMELS hydrology dataset into an analysis-ready NetCDF format.
date: 2025-9-9
event_time:
event_date:
category:
author: jiangtao
image: '/images/posts/camels_data_processing.jpg'
video_embed:
tags: [Tutorial]
featured: false
toc: false
---

[Full code](https://zenodo.org/records/17087240)

## 🚀 Key Features

* ✅ **Automated CAMELS downloader**:

  * Downloads basin attributes, meteorological forcing data (including optional extended forcing), and observed streamflow data.
* ✅ **Data preparation pipeline**:

  * Merges static basin attributes.
  * Processes Daymet, Maurer-extended, and NLDAS-extended forcing datasets.
  * Computes **Potential Evapotranspiration (PET)** using the Hargreaves method.
  * Calculates **Runoff**.
  * Generates a standardized NetCDF file (`CAMELS.nc`).
* ✅ **Data normalization and denormalization utilities** to streamline modeling workflows.

---


## ⚙️ Environment Setup

### Python Version

**Python ≥ 3.9** recommended (3.12 preferred)

### Dependencies

Create and activate a conda environment, then install dependencies:

```bash
conda create -n rainflow-env python=3.12 -y
conda activate rainflow-env

pip install numpy pandas xarray tqdm netCDF4
```
**Note**: The `pyeto` library (required for PET calculation) is not available via PyPI. It has been bundled within this repository under `rainflow/data`. Special thanks to the original PyETo authors for their implementation.

---

## Quickstart Guide

### Step 1: Download CAMELS Data

```bash
python 01.download_camels.py --output-dir ./data/CAMELS_raw
# To skip extended forcing datasets, use:
# python 01.download_camels.py --output-dir ./data/CAMELS_raw --no-extended
```

### Step 2: Prepare NetCDF Data

```bash
python 02.prepare_camels.py \
  --input-dir ./data/CAMELS_raw \
  --output-dir ./data/CAMELS_processed \
  --start-date 1980-01-01 \
  --end-date 2014-12-31
```

---

## 📊 Example: Data Loading and Normalization

See `03.load_and_normalize_data.py` for an example:

```python
import os
import pandas as pd
import numpy as np
from types import SimpleNamespace
import rainflow
from rainflow.data.nc_reader import Preprocessor

camles_531_basin_file = os.path.join(os.path.split(rainflow.__file__)[0], "data/camels/531_basin_list.txt")
camels_station_ids = pd.read_csv(camles_531_basin_file, header=None, dtype=str).values[:, 0].tolist()

config_dict = {
    'input_nc_file': './data/CAMELS_processed/CAMELS.nc',
    'train_date_list': ["1980-10-01", "1995-09-30"],
    'val_date_list': ["1980-10-01", "1985-09-30"],
    'test_date_list': ["1995-10-01", "2010-09-30"],
    'time_series_variables': ['daymet_prcp', 'daymet_srad', 'daymet_tmax', 'daymet_tmin', 'daymet_dayl', 'daymet_vp',
                              'daymet_pet'],
    'target_variables': ['Runoff'],
    'static_variables': ['elev_mean', 'slope_mean', 'area_gages2', 'frac_forest', 'lai_max', 'lai_diff', 'gvf_max',
                         'gvf_diff', 'soil_depth_pelletier', 'soil_depth_statsgo', 'soil_porosity', 'soil_conductivity',
                         'max_water_content', 'sand_frac', 'silt_frac', 'clay_frac', 'carbonate_rocks_frac',
                         'geol_permeability', 'p_mean', 'pet_mean', 'aridity', 'frac_snow', 'high_prec_freq',
                         'high_prec_dur', 'low_prec_freq', 'low_prec_dur'],
    'station_ids': camels_station_ids,
    'add_coords': False,
}

config_dataset = SimpleNamespace(**config_dict)

prep = Preprocessor(config_dataset)
# load training data without warmup days and without scaler (to fit a new scaler)
data_dict_train = prep.load_and_process(split="train", warmup_days=0, scaler=None)
norm_x_train = data_dict_train["norm_x"]  # [basins, time, features]
norm_y_train = data_dict_train["norm_y"]  # [basins, time, targets]
norm_c_train = data_dict_train["norm_c"]  # [basins, static_features]
date_range_train = data_dict_train["date_range"]
scaler = data_dict_train["scaler"]
# raw data without normalization
raw_x = data_dict_train["raw_x"]
raw_c = data_dict_train["raw_c"]
raw_y = data_dict_train["raw_y"]

# load test data with the scaler fitted on training data
data_dict_test = prep.load_and_process(split="test", warmup_days=0, scaler=scaler)
norm_x_test = data_dict_test["norm_x"]
norm_y_test = data_dict_test["norm_y"]
norm_c_test = data_dict_test["norm_c"]

# de-normalize target variable
de_norm_y_test = prep.inverse_transform(norm_y_test)
print("Difference in test target (raw vs de-normalized):", np.nanmax(np.abs(data_dict_test["raw_y"] - de_norm_y_test)))
```

---

## 📜 Citation

If you use this pipeline or data in your research, please cite:
```
Liu, J., Bian, Y., Lawson, K., & Shen, C. (2024). Probing the limit of hydrologic predictability with the Transformer network. *Journal of Hydrology*, 637, 131389. https://doi.org/10.1016/j.jhydrol.2024.131389

Liu, J., Shen, C., O'Donncha, F., Song, Y., et al. (2025). From RNNs to Transformers: benchmarking deep learning architectures for hydrologic prediction. *EGUsphere*, 2025, 1-21. https://doi.org/10.5194/egusphere-2025-1706
```

---

## 🙌 Acknowledgments

Thanks to the following resources and tools that provided inspiration and support:

* [CAMELS Dataset](https://ral.ucar.edu/solutions/products/camels)
* [CAMELS Extended Maurer Forcing Data](https://www.hydroshare.org/resource/17c896843cf940339c3c3496d0c1c077/)
* [HydroDL](https://github.com/mhpi/hydroDL)
* [neuralhydrology](https://github.com/neuralhydrology/neuralhydrology)
* [PyETo](https://github.com/woodcrafty/PyETo)
* [ChatGPT](https://chat.openai.com/) for assistance in improving code readability and refining documentation.
* [https://zenodo.org/records/4670268](https://zenodo.org/records/4670268)

---

## 📄 License

This software is licensed under CC BY-NC 4.0.
Academic and research use is explicitly permitted.

