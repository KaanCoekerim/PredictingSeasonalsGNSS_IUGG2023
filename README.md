# PredictingSeasonalsGNSS_IUGG2023
Authors: **Kaan Cökerim<sup>1</sup>, Jonathan Bedford<sup>1</sup> & Henryk Dobslaw<sup>2</sup>**\
Contact: kaan.coekerim@rub.de

<sup>1</sup>Ruhr-University Bochum, Institute for Geology, Mineralogy and Geophysics, Bochum, Germany\
<sup>2</sup>GFZ German Research Centre for Geosciences, Earth System Modelling, Potsdam, Germany

## Introduction
Seasonal oscillations in GNSS time series are a major source of noise for the interpretation of tectonic signals and are difficult to remove with established methods such as regression models, especially with regards to inter-annual wavelength variations of the seasonal signal. Non-tidal fluid loading is shown to be a main cause of the seasonality in tectonic geodesy, though the exact interactions remain to be determined.

Here, we show the capabilities in predicting and removing the seasonal signals by using global non-tidal fluid data and attempt to shed some light into the interaction between non-tidal fluid loading and seasonal noise of GNSS time series.

## GNSS Data
We retrieve 4-year-long, vertical GNSS residual coordinate displacement time series for 134 stations in South America from [Nevada Geodetic Laboratory (NGL)](http://geodesy.unr.edu/index.php) over the time period from 2010-2022. 

## Methods
### Trajectory Model
We decompose the raw GNSS time series into tectonic, seasonal and residual contributions using the [GrAtSiD](https://github.com/TectonicGeodesy-RUB/Gratsid) trajactory modelling algorithm.

### Machine Learning Model
#### Pre-processing
As part of our inputs, we use non-tidal loading data from [ESMGFZ](http://esmdata.gfz-potsdam.de:8080/repository/) data products consisting of atmospheric, oceanic and hydrological loading. We prepare for all stations, waveforms of all loading components. We also add the respective detrended GNSS time series of each station as further input feature.

To regularize and stabilize the training, we found it helpful to add two features representing the trigonometric encoding of the day-of-year of each sample (i.e. for each sample $\sin\left(\frac{2\pi}{T} doy\right)$ and $\cos\left(\frac{2\pi}{T} doy\right)$ with $T=365.25$). We also provide or models with geographic coordinate information of the station each time series was recorded at. The coordinates are also trigonometrical encoded, where the latitude is represented by its sine and the longitude is encoded as sine and cosine to avoid non-uniqueness.

To avoid temporal information leakage in the learning process, we separate the data set into a training segment before Jan 1st 2020 and a test segment after Jan 1st 2020. We further extract from the training set the period after Jan 1st 2018 as validation data to be used as guidance during training. We require all waveforms in the training data set to have at least 2 years worth of recorded data with a maximum of 30% gaps relative to the whole time period covered by the time series. In the test data set we also require 70% time series completes but only 1 year of data in order not to eliminate to many time series from this data set. 

We use z-score normalization to standardize all input features and the targets separately. For the prediction, we use the sample at prediction time and information from a 14-days time window into past time steps. The geographic station coordinate and day-of-year information remain as static feature. Thus, in the case of deep learning models, we provide the static features as auxiliary inputs that are concatenated with the main time series processing in the model before entering the dense output network.

#### Model Struckture
- 1 [**XGB**](https://xgboost.readthedocs.io/en/stable/#): Gradient Boosted Regression Tree
- 2 **CNN**: Convolutional Neural Network 

