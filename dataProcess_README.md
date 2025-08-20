# Global Ocean Surface Velocity Patches Dataset

## Overview

The data is derived from the Copernicus Marine Service's global ocean reanalysis product and consists of `448x448` pixel patches of eastward (uo) and northward (vo) sea water velocity, extracted daily. Patches with a land presence greater than 80% have been filtered out to ensure data quality and relevance for oceanographic studies.Given the large number of total data, temporarily only 1 year data was processed, containing 53655 patches.

## Data Source

The data is derived from the following E.U. Copernicus Marine Service (CMEMS) product:

*   **Product Name:** Global Ocean Physics Reanalysis
*   **Product ID:** `GLOBAL_MULTIYEAR_PHY_001_030`
*   **Source Dataset ID:** `cmems_mod_glo_phy_my_0.083deg_P1D-m`
*   **Original Variables:** `uo` (Eastward Velocity), `vo` (Northward Velocity)
*   **Original Temporal Coverage:** January 1, 1993 - June 30, 2021
*   **Original Spatial Resolution:** 1/12° (~8 km)
*   **Depth Level:** Surface layer (~0.49m)
*   **Reference:** For full details on the original data, please refer to the [official product page](https://data.marine.copernicus.eu/product/GLOBAL_MULTIYEAR_PHY_001_030/description).

## Diretory Structure
```text
/data_new/uovo_data/
├── raw_data/
│   ├── mask.nc                             # land mask source file
│   ├── uovo_1993-01-01_to_1993-12-31.nc    # uovo source file
│   ├── uovo_1994-01-01_to_1994-12-31.nc
│   └── ...
├── processed_data/
│   └── uovo_1997-01-01_to_1997-12-31.h5    # processed data, temorarily only 1 year processed
├── scripts/
│   ├── dataDownload.py                     # data download script                    
│   ├── check_data.ipynb                    # data check script, including usage
│   ├── dataProcess.py                      # data process script
│   ├── data_distribution.png               # data distribution
│   ├── data_distribution_exponential.png   # data distribution in exponential format
│   ├── uo_vo_processed.png                 # processed data visualization
│   └── uo_vo_raw.png                       # raw data visulization
├── CMEMS-GLO-PUM-001-030 user manual.pdf   # offitial user manual
└── README.md                               # this file
```

## Processing Pipeline

The raw NetCDF data from the Copernicus Marine Service was processed into a consolidated HDF5 file using the following pipeline for each day in the time series:

1.  **Data Extraction:** Daily data for `uo` and `vo` variables and the corresponding static land-sea mask were retrieved. Masked values in the velocity fields, which represent land or areas with no data, were converted to `NaN` (Not a Number).
2.  **Cyclic Padding:** To ensure seamless patch extraction across the 180° longitude line (dateline), the data was padded cyclically along the longitude axis. A padding of 224 pixels was added to each side.
3.  **Sliding Window Patching:** The global data was partitioned into `448x448` pixel patches using a sliding window with a stride of `224` pixels in both latitude and longitude directions.
4.  **Land-based Filtering:** Each patch was checked against the land-sea mask. Patches where the land area constituted more than 80% of the total patch area were discarded.
5.  **Data Stacking and Storage:** For each valid ocean patch, the `uo` and `vo` components were stacked along a channel axis to form an array of shape `(2, 448, 448)`. These arrays, along with mask, longitude, and latitude grids in corresponding position, were then saved into a single HDF5 file.

## HDF5 File Structure

The dataset is stored in the HDF5 (`.h5`) format. The file contains the following datasets, where `N` is the total number of valid patches collected. All datasets use Gzip compression for efficient storage.

***Every day's mask, lat and lon are the same, only save one day static data.Totally 147 patches in a day.***

*   `uovo_data`
    *   **Shape:** `(N, 2, 448, 448)`
    *   **Type:** `np.ndarray`
    *   **DType:** `np.float32`
    *   **Description:** The core velocity data, land is filled with `np.nan`. The channel dimension (axis 1) contains the two velocity components:
        *   `Channel 0`: Eastward velocity (`uo`) in m/s.
        *   `Channel 1`: Northward velocity (`vo`) in m/s.

*   `mask`
    *   **Shape:** `(147, 448, 448)`
    *   **Type:** `np.ndarray`
    *   **DType:** `np.bool`
    *   **Description:** The land-sea mask corresponding to each patch. `True` indicates a masked point (land), and `False` indicates a valid ocean point.

*   `lon`
    *   **Shape:** `(147, 448, 448)`
    *   **Type:** `np.ndarray`
    *   **DType:** `np.float32`
    *   **Description:** A grid of longitude values for each point in the patch. Units are in degrees east.

*   `lat`
    *   **Shape:** `(147, 448, 448)`
    *   **Type:** `np.ndarray`
    *   **DType:** `float32`
    *   **Description:** A grid of latitude values for each point in the patch. Units are in degrees north.

## Data Statistics

The following table provides global statistical information for the `uo` (eastward) and `vo` (northward) velocity components, calculated across all valid (non-land) data points in the dataset.

| Statistic          | `uo` (Eastward Velocity) (m/s) | `vo` (Northward Velocity) (m/s) |
| :----------------- | :----------------------------- | :------------------------------ |
| **Mean**           | `0.0063662025596672046`        | `0.006140309626745585`          |
| **Standard Dev.**  | `0.18706705950830543`          | `0.1470535952269027`            |
| **Min**            | `-1.9061861000955105`          | `-2.2461622953414917`           |
| **25% (Q1)**       | `-0.06897183135151863`         | `-0.0555436871945858`           |
| **50% (Median)**   | `0.003662221133708954`         | `0.004272591322660446`          |
| **75% (Q3)**       | `0.07873775437474251`          | `0.06591998040676117`           |
| **Max**            | `2.633136995136738`            | `2.2211371175944805`            |

Total valid patches(land account less than 80%): 53655，Total patches processed: 62415


