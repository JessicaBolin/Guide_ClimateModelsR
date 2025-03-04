# Climate Change Metrics

## Introduction

The aim of this tutorial is to provide a worked example of how to calculate climate change metrics using CMIP6 models. The climate-change metrics used in this example are [**Climate Velocity**](https://science.sciencemag.org/content/334/6056/652.abstract) and a [**Relative Climate Exposure**](https://www.researchsquare.com/article/rs-421078/v1) index.

This dataset contains a raster-stack file in format **.grd** of monthly sea surface temperature (tos) for the GFDL-CM4 model under a SSP5-5.8 emission scenario. The model goes from 2015 until 2100.

## Data import

Load the required packages and the data.


```r
# load packages
  library(raster)
  library(VoCC)
  library(stringr)
  library(dplyr)
```

## Climate Velocity (VoCC)

_Climate velocity_ is a vector that describes the speed and direction that a point on a gridded map would need to move to remain static in climate space (e.g., to maintain an isoline of a given variable in a univariate environment) under climate change. From an ecological perspective, _climate velocity_ can be conceptualized as the speed and direction in which a species would need to move to maintain its current climate conditions under climate change. For this reason, _climate velocity_ can be considered to represent the potential exposure to climate change faced by a species if the climate moves beyond the physiological tolerance of a local population. See [Brito-Morales et al. 2018](https://www.sciencedirect.com/science/article/pii/S0169534718300636?casa_token=x8K3zBG-CkwAAAAA:HiDxHChsoKHoonPvIPKOSNQMu3krUEUcMFYOBUYdRLuEhikg2MJ6MdyRKOHenfk3frdXMLjAeyc).

[Install the R package: GitHub Repo](https://github.com/JorGarMol/VoCC)

<img src="images/VoCC_01.png" width="100%" />

To calculate _climate velocity_ the R package `VoCC` provides a comprehensive collection of functions that calculate climate velocity and related metrics from their initial formulation to the latest developments. See [Garcia Molinos et al. 2019](https://besjournals.onlinelibrary.wiley.com/doi/full/10.1111/2041-210X.13295).

<img src="images/VoCC_02.png" width="100%" />


```r
# Load the monthly raster object
  rs <- raster::stack("data/ClimateModelsRasterMonthly/tos/ssp585/tos_Omon_GFDL-CM4_ssp585_2015-2100.grd")
# Establish the time period of interest (if applicable)
  from = 2020
  to = 2100
# Define the time period to estimate climate velocity and subset the original raster
  names.yrs <- paste("X", seq(as.Date(paste(from, "1", "1", sep = "/")), as.Date(paste(to, "12", "1", sep = "/")), by = "month"), sep = "") %>%
    str_replace_all(pattern = "-", replacement = ".")
  rs <- raster::subset(rs, names.yrs)
# If Raster is monthly, get annual mean
  index <- rep(1:nlayers(rs), each = 12, length.out = nlayers(rs))
  rs <- stackApply(x = rs, indices = index, fun = mean)
  
# Calculate VoCC
  # Temporal trend (slope)
    slp <- tempTrend(rs, th = 10)
  # Spatial gradient (gradient)
    grad <- spatGrad(rs, th = 0.0001, projected = FALSE)
  # VoCC local gradient
    vocc <- gVoCC(slp, grad)
    vocc$voccMag[] <- ifelse(is.infinite(vocc$voccMag[]), NA, vocc$voccMag[]) # replace inf with NAs
```

## Relative Climate Exposure (RCE)

The RCE is a metric that we developed to obtain information about the amount of exposure to climate warming that local populations of a species would face relative to its experience of variation in seasonal temperatures [Brito-Morales et al. 2021](https://www.researchsquare.com/article/rs-421078/v1). RCE is calculated as the ratio of the slope of a linear regression of projected mean annual temperatures (°C yr-1) to the current mean seasonal temperature range (°C):


```r
# The current seasonal variation (could be any range)
  from = 2016
  to = 2021
# Getting the years/month to calculate de RCE index
  names.yrs <- paste("X", seq(as.Date(paste(from, "1", "1", sep = "/")), 
                              as.Date(paste(to, "12", "1", sep = "/")), by = "month"), 
                     sep = "") %>%
    str_replace_all(pattern = "-", replacement = ".")
# Read and subset the data
  rs <- readAll(raster::stack("data/ClimateModelsRasterMonthly/tos/ssp585/tos_Omon_GFDL-CM4_ssp585_2015-2100.grd")) %>% 
    subset(names.yrs)
# Annual min and max to estimate the rage to get the RCE
  index <- rep(1:nlayers(rs), each = 12, length.out = nlayers(rs))
  rs_min <- stackApply(x = rs, indices = index, fun = min)
  rs_max <- stackApply(x = rs, indices = index, fun = max)
# Range among the period selected
  rs_range <- rs_max - rs_min
  rs_range_mean <- stackApply(x = rs_range, indices = nlayers(rs_range), fun = mean)
  
# Get the slope
  from = 2020
  to = 2100
  names.yrs <- paste("X", seq(as.Date(paste(from, "1", "1", sep = "/")), as.Date(paste(to, "12", "1", sep = "/")), by = "month"), sep = "") %>%
    str_replace_all(pattern = "-", replacement = ".")
  rs <- raster::stack("data/ClimateModelsRasterMonthly/tos/ssp585/tos_Omon_GFDL-CM4_ssp585_2015-2100.grd") %>% 
    subset(names.yrs)
  index <- rep(1:nlayers(rs), each = 12, length.out = nlayers(rs))
  rs <- stackApply(x = rs, indices = index, fun = mean)
  slp <- ((tempTrend(rs, th = 10))*10)*8 # x10 decadal and x8 for decades (2020-2100)

# Calculate the RCE
  RCE <- abs(slp/rs1_range_mean)
```






