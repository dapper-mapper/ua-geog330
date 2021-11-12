# Lab 11 - Supervised Classification of Flood Events in Google Earth Engine (GEE)

## Objectives
-	Learn to sample training and validation data and extract reflectance values and indices from satellite imagery
-	Perform supervised classifications with Google Earth Engine and multiple classifiers
-	Classify flooded areas visible in satellite imagery
-	Apply the trained classifier to a new flood region and assess performance
- Consider the limitations of sampling data for training/ validation from other LULC products


## Introduction
Flooding affects more people than any other environmental hazard. Between 2000 to 2019, there was an estimated $651-million (USD) in flood damages across the globe (see [CRED, UNDRR](https://reliefweb.int/report/world/human-cost-disasters-overview-last-20-years-2000-2019)). Remote sensing is an important tool to monitor the extent, frequency, and depth of flooding in order to inform planning, adaptation and diaster response (see [Tellman et al, 2021](https://www.nature.com/articles/s41586-021-03695-w)). In this lab you will use Sentinel-2 imagery to create a supervised classification of a flood event in Omaha, NE in 2019. You will then evaluate how well your method transfers to a new flood event in Queensland, Australia that occurred in 2017. 

There are two basic methods of image classification: **Supervised** and **Unsupervised**. In this lab you will learn how to perform supervised classification by sampling an alternative land-use land cover (LULC) product, in this case representing water, to generate training and validation data. Traditionally, “ground truth” data or “ground control points” (GCPs) are captured either in the field using a Global Positioning System (GPS) or using on-screen digitizing from high resolution satellite imagery or aerial photography. However, with quickly growing data catalogs of land-use land cover (LULC) data an alternative approach is to sample existing datasets. However, there are limitations to such approaches that we will consider in this lab.

Once we collect a series of training data these become the seeds that form the basis from which the pixels in our target image are classified. In effect, we overlay the training data and then identify the pixels overlapping those points. Those pixels form the spectral characteristics for the bounds of our land cover classes. Typically the collection of "ground truth" or GCPs is time consuming, but in this example we can collect training/ validation data rapidly and therefore employ different sampling strategies on-the-fly. In addition, we will withold a portion of the training data to assess the accuracy of sampling strategies and different classifiers. 

This lab uses data from Landsat 5. You’ll notice that the provided data files only contain six bands. We have omitted bands 6 (thermal IR) and band 8 (panchromatic) because they have different spatial resolutions from the 30 meter pixel length of the other bands (TIR: 120m, Panchromatic: 15m). Therefore, layer 4 in this dataset corresponds to NIR (band 4) and layer 6 corresponds to SWIR (band 7). 


## Part 1 - Supervised Classification of flooding surrounding Omaha, NE on March 16th, 2019

#### Step 1. View Sentinel-2 Imagery for Omaha, NE on 
```js
// Sentinel-2 Image for floods in Omaha in 2019
var s2_omaha = ee.Image('COPERNICUS/S2/20190316T171039_20190316T171921_T14TQL')

var s2_viz = {opacity:1,
              bands:["B12","B8A","B4"],
              min:547.0221369809726,
              max:2790.664937848959,
              gamma:1}
Map.addLayer(s2_omaha, s2_viz, "Sentinel-2 - Omaha, MO")
```


## Part 2 - Application of your RF Classifier to flooding in Queensland, Australia on 
