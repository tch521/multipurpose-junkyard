---
layout: default
title: Climatics Research Problem
permalink: /climatics-research-problem
---

# Estimating High-Resolution Weather Metrics from Climate Model Data

## Definitions

- **Variable:** A simple weather observation such as daily maximum temperature or daily total precipitation.
- **Metric:** A derived measure based on variables, such as "the average number of days per year where the maximum temperature exceeds 40°C" or "the average number of days per year with more than 10mm of rainfall." 
More complex metrics include the annual average number of consecutive wet days (>1mm), heatwaves intensity/frequency (using BoM definition), and Standard Precipitation Index (SPI) statistics.

## Introduction

The goal of this research is to develop a method for producing continuous graphs of weather metrics over time, from 1950 to 2100, at a given point in Australia. 
This involves integrating two datasets of gridded weather data: the Bureau of Meteorology (BoM) Australian Gridded Climate Dataset (AGCD) and the CMIP6 climate model data. 
The AGCD has a spatial resolution of 0.05 degrees and is derived from interpolating weather observations from 1950 to present. 
The CMIP6 dataset, on the other hand, simulates the climate under various emissions scenarios from 2015 to 2100 and has a spatial resolution of approximately 0.25 degrees. 
This spatial relationship is illustrated below.

![Diagram showing the relationship between a CMIP6 "pixel"and the 25 corresponding AGCD "pixels"](./images/c2a_pixel_comparison.svg)

The nature of CMIP6 climate models means that while there is data for 2015 onward, this data does not match the observed day-to-day weather. 
It is only attempting to represent the general climate. 
That is to say the CMIP6 climate models allow us to peer into the future, albeit with extremely blurry vision.
This picture of the future is blurred not just spatially, but also temporally.
To explain by way of example: while a CMIP6 temperature at a given point on a given day in 2015 should not be expected to match what was observed, the average temperature for a region over the 2015 to 2024 period should match what was observed. 

There are existing efforts to "downscale" CMIP6 data using physics-based approaches, which a complex and computationally intensive. 
In theory, they should produce a version of the CMIP6 data that is almost the same spatial resolution as the AGCD data.
One might therefore expect this downscaled data to have a reasonably good agreement with current observations.
However, in our testing, we have not found this to be the case, as illustrated in the image below for a number of Australian cities.

![Charts showing days over 35°C over time for Australian cities](./images/TX_g35.png)

Notice the discontinuity where the AGCD data ends and the CMIP6 data begins.
We see similar discrepancies for other metrics:
- [CDD](./images/CDD.png) - Consecutive Dry (<1mm) Days
- [CWD](./images/CWD.png) - Consecutive Wet (>1mm) Days
- [RD_g0](./images/RD_g0.png) - Rainfall Days >0mm
- [RD_g1](./images/RD_g1.png) - Rainfall Days >1mm
- [RD_g5](./images/RD_g5.png) - Rainfall Days >5mm
- [RD_g10](./images/RD_g10.png) - Rainfall Days >10mm
- [RD_g20](./images/RD_g20.png) - Rainfall Days >20mm
- [RD_g50](./images/RD_g50.png) - Rainfall Days >50mm
- [RD_p95](./images/RD_p95.png) - Rainfall Days 95th Percentile
- [RD_p99](./images/RD_p99.png) - Rainfall Days 99th Percentile
- [TN_g20](./images/TN_g20.png) - Days with Min Temp >20°C
- [TN_g25](./images/TN_g25.png) - Days with Min Temp >25°C
- [TN_l0](./images/TN_l0.png) - Days with Min Temp <0°C
- [TN_l5](./images/TN_l5.png) - Days with Min Temp <5°C
- [TN_p01](./images/TN_p01.png) - Daily Min Temp 1st Percentile
- [TN_p99](./images/TN_p99.png) - Daily Min Temp 99th Percentile
- [TX_g35](./images/TX_g35.png) - Days with Max Temp >35°C
- [TX_g40](./images/TX_g40.png) - Days with Max Temp >40°C
- [TX_l0](./images/TX_l0.png) - Days with Max Temp <0°C
- [TX_l10](./images/TX_l10.png) - Days with Max Temp <10°C
- [TX_l15](./images/TX_l15.png) - Days with Max Temp <15°C
- [TX_p01](./images/TX_p01.png) - Daily Max Temp 1st Percentile
- [TX_p99](./images/TX_p99.png) - Daily Max Temp 99th Percentile

*Why not just "drag" the CMIP6 metrics up or down in value until they match the AGCD data at the transition point?*
The Darwin chart above illustrates how there would still be an obvious discontinuity in the trend of the data.

*Why not just normalise the underlying CMIP6 variable so that its mean and standard deviation match the AGCD data?*
This would not help with the kinds of higher order, extremes-focused metrics we're trying to analyse (like heatwaves or consecutive wet days).

*Why do this at all if it's not really what the CMIP6 data was intended for?*
Partially because of TCFD regulations, there is a growing demand for this kind of data and there are already some companies supplying some of it.
We have found this data to be highly dubious, often with little to no explanation of how it was derived and no quantified uncertainties.
We simply believe we can do better.

> ## Problem Statement
> In order to produce continuous graphs of weather metrics from 1950 to 2100, we need to develop a model to translate the low resolution, "blurry" CMIP6 data into something that statistically aligns with the high-resolution AGCD data.

## Model Design

How do we go about solving this problem in a practical way?
Do we fit a model for every AGCD pixel and every CMIP6 climate model?
What kind of model should they be? Would some machine learning be useful?
(In principle we should not have to fit different models for the different CMIP6 emmissions scenarios since they all have the same starting point.)
Some other more specific questions follow.

### *Do we translate CMIP6 variables and then derive metrics from those results, or do we translate metrics directly?*

The advantage of the variable-based approach is that there would be a lot less model fitting to do, as there are at least 10 but ideally 30 or more metrics we would like to calculate for a given variable like daily maximum temperature.
A variable-based model would also mean that metrics values would be guaranteed to be consistent with one another, i.e. the number of days over 40°C would always be less than the number of days over 35°C. 
However some metrics are very complex, and so metric-based models may be more likely to satisfy our research goal.

### *If doing a variable-based model, do we enforce that the mean of the downscaled data equals the original CMIP6 data?*

![Diagram showing the relationship between a CMIP6 "pixel" and the 25 corresponding AGCD "pixels"](./images/c2a_pixel_comparison.svg)

In the image above, we could say we've derived 25 AGCD-aligned values for a variable (say temperature) based on a single CMIP6 value.
The question is, should the mean of these 25 values equal that CMIP6 value?
Traditional downscaling techniques would say "definitely yes", but enforcing this means a much more intensive model fitting procedure.

### *Should we consider the surrounding CMIP6 pixels?*

![Diagram showing how we might include the surrounding pixels in each model](./images/c2a_surrounding_pixels.svg)

The idea is that while a given point may be entirely within a single CMIP6 pixel, seeing the spatial distribution of data may improve the estimate, not unlike Kriging.

### *What other data should factor into the model?*

![Diagram showing other data that we might include in the models](./images/c2a_other_data.svg)

The image above is a not-to-scale representation of some other useful data that we could incorporate into our models to improve the estimates.
Elevation is known to have a particularly strong correlation with temperature so it is not hard to imagine how it could be useful.
Land use is more subtle, but may help to factor in urban-heat-island effects.

### *Should there be a random noise component?*

Random noise may help to better replicate certain statistical behaviours that are fundamentally missing from the low resolution CMIP6 data.
It may be particularly useful for precipitation.

## Conclusion

This document is just a starting point for discussions and there are many, many open questions about how to approach this problem.

> The CMIP6 models allow us to peek at a few different versions of the future, but the catch is that we don't have our glasses on - the images we see are blurry, as if we forgot to put on our glasses.
> While we cannot be 100% certain, we should be able to make a reasonable estimate of what we're looking at based on context and our past experience.