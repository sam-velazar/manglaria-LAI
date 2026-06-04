
# 🌿 LAI–fAPAR Pipeline using PlanetScope Imagery and Neural Networks 🌿

![R Version](https://img.shields.io/badge/R-4.5.x-blue.svg)
![Python](https://img.shields.io/badge/Python-3.9-yellow.svg)
![Conda Env](https://img.shields.io/badge/Conda-r--tensorflow-orange.svg)
![License](https://img.shields.io/badge/License-MIT-green.svg)
![Status](https://img.shields.io/badge/Pipeline-Operational-success.svg)

---

##  Overview

Este flujo de trabajo deriva:

- **Leaf Area Index (LAI)** derived from PlanetScope imagery using a pre-trained neural network.  
- **Fraction of Absorbed Photosynthetically Active Radiation (fAPAR)** estimated through the Beer–Lambert transfer function.

---

##  Workflow

```text
PlanetScope Image
│
▼
Spectral Bands
│
▼
Vegetation Indices (NDVI, GNDVI, NDRE, CIRE, NDWI)
│
▼
Feature Stack
│
▼
Scaling (center_vals.rds + scale_vals.rds)
│
▼
Neural Network Model (model_LAI.keras)
│
▼
LAI Raster
│
▼
Beer-Lambert Function
│
▼
fAPAR Raster

```

##  Software Requirements

- **R** (tested with R 4.5.x)
- **Main packages:**
  - terra
  - reticulate
  - tensorflow
  - keras3
  - parallel


##  Directory Structure

```text
project/
├── Models/
│   └── model_LAI.keras
├── Parameters/
│   ├── center_vals.rds
│   └── scale_vals.rds
├── Inputs/
│   └── composite.tif
├── Outputs/
│   ├── LAI.tif
│   └── fAPAR.tif
└── Scripts/
    └── run_pipeline.R
```
## Part I – PlanetScope Preprocessing

```r
library(terra)

planet_raw <- rast("Inputs/composite.tif")
mangrove <- vect("AOI/mangrove.shp")

planet_crop <- crop(planet_raw, mangrove)
planet <- mask(planet_crop, mangrove)

names(planet) <- c(
  "Coastal_Blue","Blue","Green_I","Green",
  "Yellow","Red","RE","NIR"
)

green <- planet[["Green"]] / 10000
red   <- planet[["Red"]] / 10000
nir   <- planet[["NIR"]] / 10000
re    <- planet[["RE"]] / 10000

ndvi  <- (nir - red) / (nir + red)
gndvi <- (nir - green) / (nir + green)
ndre  <- (nir - re) / (nir + re)
cire  <- (nir / re) - 1
ndwi  <- (green - nir) / (green + nir)

final_stack <- c(red, green, nir, re, ndvi, gndvi, ndre, cire, ndwi)
names(final_stack) <- c("RED","GREEN","NIR","RE","NDVI","GNDVI","NDRE","CIRE","NDWI")

writeRaster(final_stack,"Outputs/features.tif",overwrite=TRUE)
```

## Part II – LAI Prediction

Conda Environment Setup
To run the pipeline, an Anaconda environment named **r-tensorflow** with Python 3.9 and the required packages installed is needed:

```r
library(reticulate)

# Create the environment
conda_create("r-tensorflow", python_version = "3.9")

# Install packages
conda_install("r-tensorflow", packages = c("tensorflow", "keras", "numpy"), pip = TRUE)
```

Once the environment **r-tensorflow** has been created, linked, and activated with the required libraries:

```r
library(terra)
library(reticulate)
library(tensorflow)
library(keras3)

use_condaenv("r-tensorflow", required = TRUE)

model <- load_model("Models/model_LAI.keras")
center_vals <- readRDS("Parameters/center_vals.rds")
scale_vals  <- readRDS("Parameters/scale_vals.rds")

pred_fun <- function(model, data){
  valid <- complete.cases(data)
  output <- rep(NA, nrow(data))
  if(any(valid)){
    data_scaled <- scale(data[valid,,drop=FALSE],
                         center=center_vals,
                         scale=scale_vals)
    output[valid] <- as.vector(predict(model, data_scaled))
  }
  output
}

features <- rast("Outputs/features.tif")

lai <- terra::predict(
  features, model, fun=pred_fun,
  filename="Outputs/LAI.tif", overwrite=TRUE,
  wopt=list(datatype="FLT4S", gdal=c("COMPRESS=LZW"))
)

```

## Part III – fAPAR Estimation

Beer–Lambert Transfer Function:

$fAPAR = 99.01974 \cdot (1 - e^{-0.72874 \cdot LAI})$


```r
fapar_fun <- function(x){
  x[x <= 0] <- NA
  y <- 99.01974 * (1 - exp(-0.72874 * x))
  y[y < 0] <- 0
  y[y > 100] <- 100
  return(y)
}

app(
  lai, fun=fapar_fun,
  filename="Outputs/fAPAR.tif", overwrite=TRUE,
  cores=parallel::detectCores()-2,
  wopt=list(datatype="FLT4S", gdal=c("COMPRESS=LZW"))
)
```

## Outputs 
| Product | Units |
| :--- | :--- |
| LAI | m² m⁻² |
| fAPAR | % |

## Pipeline Summary Table

| Step | Input(s) | Process | Output(s) |
| :--- | :--- | :--- | :--- |
| PlanetScope Preproc | composite.tif, AOI/mangrove.shp | Crop + Mask + Band naming | Preprocessed raster |
| Vegetation Indices | Spectral bands | NDVI, GNDVI, NDRE, CIRE, NDWI | Feature stack |
| Scaling | Feature stack, center_vals.rds, scale_vals.rds | Standardization | Scaled features |
| Neural Network | Scaled features, model_LAI.keras | Prediction with pre-trained NN | LAI raster |
| Beer–Lambert Func | LAI raster | Transfer function application | fAPAR raster |


*© 2026 **Velázquez-Salazar S.** — Comisión Nacional para el Conocimiento y Uso de la Biodiversidad*  
*Proyecto **ManglarIA** — NON COMERCIAL USE 
