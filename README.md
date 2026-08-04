# MV-STD-GAN

Official code, trained models, and partial example data for the manuscript:

> **MV-STD-GAN: Unified Multi-Variable Spatiotemporal Downscaling of Atmospheric Fields for Tropical Cyclone Detection**

## 1. Overview

The Multi-Variable Spatiotemporal Downscaling Generative Adversarial Network
(MV-STD-GAN) is a deep-learning framework for jointly improving the spatial and
temporal resolution of multiple atmospheric variables.

The framework was developed to improve the representation and detection of
tropical cyclones (TCs) in low-resolution reanalysis and climate-model outputs.

MV-STD-GAN jointly reconstructs the following five atmospheric variables:

- Sea level pressure (`SLP`)
- 300-hPa geopotential height (`Z300`)
- 500-hPa geopotential height (`Z500`)
- 10-m zonal wind (`U10`)
- 10-m meridional wind (`V10`)

The primary task is to transform atmospheric fields from approximately
1.0° spatial resolution and 24-hour temporal resolution into fields at
approximately 0.5° spatial resolution and 6-hour temporal resolution.

The model is trained using paired high-resolution and artificially coarsened
low-resolution ERA5 data. The trained ERA5 model is additionally transferred,
without retraining, to coarsened CNRM-CM6-1-HR outputs from the historical,
SSP1-2.6, and SSP5-8.5 experiments.

---

## 2. Main contributions represented in this repository

The repository contains the experimental code used to perform the following
analyses:

1. Comparison of alternative deep-learning spatiotemporal downscaling
   architectures.
2. Comparison with interpolation-based spatial and temporal baseline methods.
3. Comparison of Direct, Indirect, and Separate temporal reconstruction
   strategies.
4. Comparison between independently trained variable-specific models and one
   unified multi-input multi-output model.
5. Evaluation using MSE, CC, PSNR, and SSIM.
6. Permutation-importance analysis of cross-variable contributions.
7. Tropical-cyclone detection using ERA5, model outputs, and IBTrACS.
8. Zero-shot transfer from ERA5 to CNRM-CM6-1-HR.
9. Evaluation under the historical, SSP1-2.6, and SSP5-8.5 experiments.
10. Generation of the figures and tables reported in the manuscript.

---

## 3. Spatiotemporal reconstruction task

Two low-resolution boundary states are used as inputs:

- `T0`: 00:00 UTC
- `T4`: 24:00 UTC

The model reconstructs the corresponding high-resolution sequence:

- `T0`: 00:00 UTC
- `T1`: 06:00 UTC
- `T2`: 12:00 UTC
- `T3`: 18:00 UTC
- `T4`: 24:00 UTC

For the five-variable multi-input multi-output experiment, the low-resolution
input contains 10 channels:

```text
SLP_T0,  SLP_T4,
Z300_T0, Z300_T4,
Z500_T0, Z500_T4,
U10_T0,  U10_T4,
V10_T0,  V10_T4
```

The corresponding high-resolution output contains 25 channels:

```text
SLP_T0,  SLP_T1,  SLP_T2,  SLP_T3,  SLP_T4,
Z300_T0, Z300_T1, Z300_T2, Z300_T3, Z300_T4,
Z500_T0, Z500_T1, Z500_T2, Z500_T3, Z500_T4,
U10_T0,  U10_T1,  U10_T2,  U10_T3,  U10_T4,
V10_T0,  V10_T1,  V10_T2,  V10_T3,  V10_T4
```

For the ERA5 domain used in the manuscript, the representative array shapes
are:

```text
Low-resolution input : (sample, 58, 94, 10)
High-resolution target: (sample, 116, 188, 25)
```

The exact array dimensions may change if a different geographical domain or
grid is used.

---

## 4. Repository structure

```text
MV-STD-GAN/
├── Code/
│   ├── Model construction and training Notebooks
│   ├── Baseline-method Notebooks
│   ├── Temporal-strategy comparison Notebooks
│   ├── ERA5 inference and evaluation Notebooks
│   ├── CNRM-CM6-1-HR transfer Notebooks
│   ├── Tropical-cyclone detection Notebooks
│   └── Figure and table generation Notebooks
│
├── Model/
│   └── Trained models used in the manuscript
│
├── data/
│   └── Partial example data and selected intermediate products
│
├── environment.yml
├── requirements.txt
├── LICENSE
└── README.md
```

The complete ERA5 and CNRM-CM6-1-HR datasets are not included because of their
large data volumes. These datasets can be obtained from their official public
archives.

---

## 5. Data sources

### 5.1 ERA5

The following ERA5 products are required.

#### ERA5 hourly data on single levels

- Mean sea level pressure
- 10-m U-component of wind
- 10-m V-component of wind

#### ERA5 hourly data on pressure levels

- Geopotential at 300 hPa
- Geopotential at 500 hPa

ERA5 data can be obtained from the Copernicus Climate Data Store:

https://cds.climate.copernicus.eu/

The ERA5 experiments in the manuscript use the following configuration:

```text
Domain:
Latitude  = 5°S–53°N
Longitude = 93°E–187°E

Period:
Training = 1980–2007
Testing  = 2008–2014

High-resolution reference:
Approximately 0.5° / 6-hourly

Low-resolution input:
Approximately 1.0° / 24-hourly
```

The default ERA5 filenames used in several Notebooks are:

```text
Mean-sea-level-pressure-1980-2024.nc
Geopotential-300hpa-1980-2024.nc
Geopotential-500hpa-1980-2024.nc
10m-u-component-of-wind-1980-2024.nc
10m-v-component-of-wind-1980-2024.nc
```

Users may use different filenames by editing the variable configuration section
in the corresponding Notebook.

### 5.2 CNRM-CM6-1-HR

CNRM-CM6-1-HR data can be obtained through the Earth System Grid Federation
(ESGF):

https://esgf-node.llnl.gov/search/cmip6/

The repository contains experiments for:

- Historical
- SSP1-2.6
- SSP5-8.5

The required CNRM-CM6-1-HR variables correspond to:

- Sea level pressure
- 300-hPa geopotential height
- 500-hPa geopotential height
- 10-m zonal wind
- 10-m meridional wind

The original CNRM-CM6-1-HR fields are treated as high-resolution references.
They are spatially coarsened and temporally subsampled to construct the
low-resolution model inputs.

The ERA5-trained MV-STD-GAN model is then applied to these low-resolution
CNRM-CM6-1-HR fields without additional training or parameter tuning.

### 5.3 IBTrACS

Observed tropical-cyclone tracks are obtained from the International Best Track
Archive for Climate Stewardship (IBTrACS):

https://www.ncei.noaa.gov/products/international-best-track-archive

IBTrACS is used to evaluate the spatial distribution and identification skill
of tropical cyclones detected from ERA5 and MV-STD-GAN outputs.

---

## 6. Software environment

The experiments were implemented in Python using Jupyter Notebook within an
Anaconda environment.

The deep-learning models were developed using TensorFlow.

The main Python dependencies include:

- `tensorflow`
- `numpy`
- `pandas`
- `xarray`
- `netCDF4`
- `scipy`
- `matplotlib`
- `opencv-python`
- `tqdm`
- `h5py`
- `scikit-learn`

A CUDA-capable GPU is strongly recommended for model training and inference.

Exact package versions are provided in:

```text
environment.yml
requirements.txt
```

### 6.1 Installation using Conda

Clone the repository:

```bash
git clone https://github.com/qq492947833/MV-STD-GAN.git
cd MV-STD-GAN
```

Create the Conda environment:

```bash
conda env create -f environment.yml
conda activate mv-std-gan
```

Start Jupyter Notebook or JupyterLab:

```bash
jupyter notebook
```

or:

```bash
jupyter lab
```

### 6.2 Installation using pip

Alternatively, the Python dependencies can be installed using:

```bash
pip install -r requirements.txt
```

The TensorFlow, CUDA, and cuDNN versions should be mutually compatible.

---

## 7. Path configuration

The original experiments were conducted on a Windows workstation. Therefore,
some Notebooks contain author-specific absolute paths.

Before running a Notebook, locate the path-configuration section near the
beginning of the Notebook and replace the original paths with paths valid on
your computer or server.

Typical path variables include:

```python
DATA_DIR = r"path/to/ERA5_or_CMIP6_data"
MODEL_DIR = r"path/to/trained_models"
RESULT_DIR = r"path/to/output_results"
TMP_DIR = r"path/to/temporary_files"
```

Examples of original author-specific paths that should be replaced include:

```text
H:\ERA5-6hour
E:\Dr_Research\model
E:\Dr_Research\result
```

Users should also check:

- Input NetCDF filenames
- Output directories
- Temporary memory-mapped file directories
- Shapefile paths
- IBTrACS file paths
- Trained-model paths

---

## 8. Notebook guide

The `Code/` directory contains the Jupyter Notebooks used in the study.

Because the complete workflow contains several experiments, the repository is
organized as a collection of research Notebooks rather than as a single-command
software package.

### 8.1 Model development and training

The model-training Notebooks construct the spatial and temporal downscaling
architectures evaluated in the manuscript.

A representative historical filename is:

```text
Auto_ESR_EfficentTemp_GAN_moreoutput-土壤湿度-ssim-不做嵌入-ERA5_5变量_100km_1day.ipynb
```

#### Note on the legacy filename

Some Notebook filenames retain Chinese descriptions or terms inherited from
earlier stages of model development.

In particular, the term `土壤湿度` means `soil moisture`. The initial version
of the model architecture was developed and tested for a soil-moisture
downscaling task. The model code was subsequently adapted to the five
atmospheric variables used in the present manuscript, while the original
Notebook filename was retained for historical continuity and to avoid breaking
existing experiment records.

The presence of `土壤湿度` in the filename does not mean that soil moisture is
used as an input or output variable in the experiments reported in the present
manuscript. The actual variables used by each experiment are defined in the
Notebook configuration and are SLP, Z300, Z500, U10, and V10.

Similarly, some Notebook filenames contain Chinese descriptions because they
were originally created during the development of the project. Their functions
are explained in English in this README.

### 8.2 Temporal reconstruction strategies

```text
MSG_SED_ET_temporal_strategy_mid_metrics.ipynb
```

This Notebook compares three strategies for reconstructing the intermediate
6-hourly states.

#### Direct strategy

T1, T2, and T3 are jointly generated from the two boundary states T0 and T4.

#### Indirect strategy

T2 is first generated from T0 and T4. The predicted T2 is subsequently used
with T0 and T4 to generate T1 and T3.

#### Separate strategy

Three independent models are trained. Each model uses T0 and T4 to predict one
of T1, T2, or T3.

The Notebook reports:

- Five-variable-averaged intermediate-time metrics
- Variable-specific intermediate-time metrics
- Metrics for each variable at T1, T2, and T3

### 8.3 Interpolation-based baseline experiments

Representative baseline Notebooks include:

```text
降尺度baseline-ST-data.ipynb
降尺度baseline-TS-data.ipynb
```

These Notebooks evaluate two conventional spatiotemporal downscaling orders:

- Spatial downscaling followed by temporal downscaling
- Temporal downscaling followed by spatial downscaling

The baseline methods use bilinear interpolation and optical flow.

### 8.4 ERA5 evaluation and result visualization

Representative ERA5 result and visualization Notebooks include:

```text
降尺度结果展示-data.ipynb
```

These Notebooks calculate reconstruction metrics, compare model architectures,
and generate ERA5-based figures and tables.

### 8.5 CNRM-CM6-1-HR zero-shot transfer

Representative CNRM-CM6-1-HR inference Notebooks include:

```text
降尺度结果输出-CMIP-ERA5-mean-std-new.ipynb
降尺度结果输出-CMIP-ERA5-mean-std-new-SSP126.ipynb
降尺度结果输出-CMIP-ERA5-mean-std-new-SSP585.ipynb
降尺度结果输出-CMIP-CMIP-mean-std-new.ipynb
```

These Notebooks:

1. Read CNRM-CM6-1-HR data.
2. Construct low-resolution inputs.
3. Apply standardization parameters derived from the ERA5 training data.
4. Load the ERA5-trained MV-STD-GAN model.
5. Generate high-resolution outputs without retraining.
6. Apply inverse standardization.
7. Calculate reconstruction metrics.
8. Save the downscaled CNRM-CM6-1-HR fields.

### 8.6 Multi-variable and variable-specific comparisons

Representative Notebooks include:

```text
降尺度结果展示-多变量和单变量-CMIP-hist-ERA5-mean-std.ipynb
降尺度结果展示-多变量和单变量-CMIP-ssp126-ERA5-mean-std.ipynb
降尺度结果展示-多变量和单变量-CMIP-ssp585-ERA5-mean-std.ipynb
```

These Notebooks compare:

- Independently trained single-input single-output models
- A unified multi-input multi-output model
- Historical and future-scenario transfer performance

### 8.7 Radar-chart visualization

```text
降尺度雷达图.ipynb
```

This Notebook generates the radar chart comparing model performance across the
historical, SSP1-2.6, and SSP5-8.5 experiments.

### 8.8 Tropical-cyclone detection

The Notebooks whose filenames begin with:

```text
TC_detect_test
```

implement tropical-cyclone detection for different datasets and resolutions.

Representative examples include:

```text
TC_detect_test-6hour-0.25-原始-区域更小-仅用TC-40年-风速阈值筛选.ipynb

TC_detect_test-1day-1.0-Only-point-原始-区域更小-仅用TC-真值结果-CMIP.ipynb

TC_detect_test-6hour-0.5-Only-point-原始-区域更小-仅用TC-模型结果-CMIP.ipynb

TC_detect_test-6hour-0.5-Only-point-原始-区域更小-仅用TC-真值结果-CMIP.ipynb
```

The TC-detection experiments are performed for:

- IBTrACS observations
- ERA5 high-resolution reference fields
- ERA5 low-resolution input fields
- ERA5 MV-STD-GAN outputs
- CNRM-CM6-1-HR high-resolution reference fields
- Coarsened CNRM-CM6-1-HR low-resolution fields
- MV-STD-GAN downscaled CNRM-CM6-1-HR fields

The detected TC track-density fields are compared using:

- Probability of Detection (POD)
- False Alarm Ratio (FAR)
- POD–FAR
- Number of detected TC points
- Spatial pattern correlation coefficient (PCC)

---

## 9. Suggested reproduction workflow

A complete reproduction of the manuscript can be performed using the following
general workflow.

### Step 1: Download the source data

Download:

- ERA5 single-level fields
- ERA5 pressure-level fields
- CNRM-CM6-1-HR historical fields
- CNRM-CM6-1-HR SSP1-2.6 fields
- CNRM-CM6-1-HR SSP5-8.5 fields
- IBTrACS observations

### Step 2: Configure local paths

Modify the path-configuration cells in the required Notebooks.

### Step 3: Construct the ERA5 datasets

Construct paired:

- High-resolution ERA5 targets
- Spatially coarsened and temporally subsampled ERA5 inputs

Preserve the chronological split:

```text
Training: 1980–2007
Testing : 2008–2014
```

### Step 4: Calculate standardization parameters

Calculate the mean and standard deviation using only the ERA5 training data.

Apply the same training-set standardization parameters to:

- ERA5 training data
- ERA5 testing data
- CNRM-CM6-1-HR transfer data

### Step 5: Train or load the models

Users may either:

- Train the candidate architectures from the provided Notebooks, or
- Load the trained models provided in the `Model/` directory

### Step 6: Evaluate candidate architectures

Calculate MSE, CC, PSNR, and SSIM for:

- Deep-learning candidate models
- Bilinear-interpolation and optical-flow baselines

### Step 7: Compare temporal strategies

Run:

```text
MSG_SED_ET_temporal_strategy_mid_metrics.ipynb
```

to compare Direct, Indirect, and Separate temporal reconstruction.

### Step 8: Compare variable-modelling strategies

Compare:

- Five independently trained variable-specific models
- One unified five-variable model

### Step 9: Perform permutation-importance analysis

Randomly permute each input variable across samples while keeping the remaining
input variables unchanged.

Calculate the relative increase in output-variable MSE and normalize the five
input-variable importance values to sum to one for each output variable.

### Step 10: Run ERA5 tropical-cyclone detection

Run the corresponding `TC_detect_test` Notebooks for:

- ERA5 high-resolution fields
- ERA5 low-resolution fields
- MV-STD-GAN outputs
- IBTrACS

### Step 11: Perform CNRM-CM6-1-HR transfer

Apply the ERA5-trained model without retraining to:

- Historical
- SSP1-2.6
- SSP5-8.5

### Step 12: Generate manuscript figures and tables

Run the visualization and analysis Notebooks to reproduce the figures and tables
reported in the manuscript.

---

## 10. Trained models

The `Model/` directory contains the trained models used in the manuscript.

The trained models allow users to perform inference and evaluation without
retraining all candidate architectures.

Before loading a trained model, users should confirm:

1. The TensorFlow version is compatible.
2. The required custom model layers or functions are available.
3. The input-variable order is correct.
4. The input time-step order is correct.
5. The standardization parameters correspond to the ERA5 training data.
6. The input spatial dimensions match those expected by the model.

The principal variable order is:

```text
SLP, Z300, Z500, U10, V10
```

---

## 11. Partial example data

The `data/` directory contains partial example data and selected intermediate
products.

These files are provided to illustrate:

- Data organization
- Expected variable order
- Expected file format
- Selected evaluation procedures
- Selected figure-generation procedures

The `data/` directory does not contain the complete ERA5 or CNRM-CM6-1-HR
archives used in the study.

The complete datasets must be downloaded from their official sources.

---

## 12. Evaluation metrics

The atmospheric-field reconstruction experiments use:

- Mean squared error (`MSE`)
- Correlation coefficient (`CC`)
- Peak signal-to-noise ratio (`PSNR`)
- Structural similarity index measure (`SSIM`)

The TC-detection experiments use:

- Probability of Detection (`POD`)
- False Alarm Ratio (`FAR`)
- `POD − FAR`
- Number of detected TC points (`PN`)
- Spatial pattern correlation coefficient (`PCC`) for TC track-density maps

---

## 13. Reproducibility notes

Users should consider the following points when reproducing the experiments:

1. The ERA5 training and testing periods must be divided chronologically.
2. Standardization parameters must be calculated from the ERA5 training data.
3. The same variable order must be used during training and inference.
4. The same time-step order must be used during training and inference.
5. The same standardization parameters must be used for inverse transformation.
6. Some experiments require substantial system memory and disk storage.
7. Memory-mapped arrays are used in some Notebooks to reduce memory pressure.
8. Model training requires significantly more computational resources than
   inference.
9. Results may vary slightly across TensorFlow, CUDA, cuDNN, and GPU versions.
10. Some Notebooks contain saved outputs from the original experiments.
11. Local path configurations must be modified before execution.
12. Notebook filenames may retain historical Chinese descriptions, but the
    variables used by each experiment are defined inside the Notebook.

---

## 14. Known limitations of the repository

- The complete ERA5 and CNRM-CM6-1-HR datasets are not redistributed because
  of their large sizes.
- The workflow is Notebook-based and is not currently packaged as a standalone
  Python library.
- Some Notebook filenames and comments retain descriptions from earlier stages
  of model development.
- Some Notebooks require users to modify local paths manually.
- Exact numerical reproduction may depend on the TensorFlow, CUDA, cuDNN, and
  hardware environment.
- The CNRM-CM6-1-HR experiment represents a proof-of-concept external transfer
  evaluation rather than a comprehensive multi-model CMIP6 assessment.

---

## 15. Citation

The final bibliographic information will be updated after publication.

When using this repository, please cite the accompanying manuscript:

```bibtex
@article{Ye_MV_STD_GAN,
  title   = {MV-STD-GAN: Unified Multi-Variable Spatiotemporal Downscaling of Atmospheric Fields for Tropical Cyclone Detection},
  author  = {Ye, Yuchen and Yuan, Chaoxia and Qi, Zixuan and Cai, Yanpeng
             and Chen, Anqi and Li, Chuang},
  journal = {Manuscript submitted for publication},
  year    = {2026}
}
```

---

## 16. License

This repository is distributed under the Apache License 2.0.

See the `LICENSE` file for details.

---

## 17. Contact

For questions regarding the model, code, trained models, or data organization,
please contact:

**Yuchen Ye**

Email: 492947833@qq.com
