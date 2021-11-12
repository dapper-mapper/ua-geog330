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

This lab uses data from Sentinel-2A/2B MultiSpectral Instrument (MSI) Level-1C. The Sentinel-2 platform is comprised of two sensors, Sentinel-2A and Sentinel-2B. Sentinel-2A was launched on 23 June 2015, while Sentinel-2B satellite was launched on 7 March 2017. Therefore, the cadence of Sentinel-2 imagery after March 2017 is greater with images available every 3-5 days with higher-frequencies closer to the poles (see [Li & Roy, 2017](https://www.mdpi.com/2072-4292/9/9/902/htm). For a reminder of the spectral bands included within Sentinel-2 imagery, follow [this link](https://hatarilabs.com/ih-en/how-many-spectral-bands-have-the-sentinel-2-images). We will use [Google Earth Engine Code Editor](https://code.earthengine.google.com/) to access, visualize, analyze, and classify Sentinel-2 imagery. No data downloads are necessary for this lab as ALL the data is hosted in the cloud as part of Google Earth Engine's data catalog.  


## Part 1 - Supervised Classification of flooding surrounding Omaha, NE on March 16th, 2019
Historic flooding in the Midwest has caused unprecedented damage and led nearly $1.3-billion (USD) in damage in areas surrounding Omaha. To assess the damage, the Federal Emergency Management Agency (FEMA) has contracted you to develop a map of inundated area based on available satellite imagery. They have provided you no data to do this exercise and have asked for a rapid response. The methodology you decide includes:

1. A supervised classification of Sentinel-2 imagery given its availability and medium resolution
2. To create training and validation data you decide to sample an the [Joint Research Centre's (JRC) Global Surface Water dataset](https://storage.googleapis.com/global-surface-water/downloads_ancillary/DataUsersGuidev2.pdf), an existing surface water dataset ([Pekel et al, 2016](https://www.nature.com/articles/nature20584)). 

To realize this methodology, execute the following steps.

### Step 1. Open Google Earth Engine's Code Editor
Open Google Earth Engine's (GEE) Code Editor where we will access, analyze and classify Sentinel-2 imagery

https://code.earthengine.google.com/

If you are new to the code editor, the image below provides an overview of the different user-interface with each portion of the image described below
![image](https://user-images.githubusercontent.com/13280304/141536045-c217813a-2d1e-465f-a4f9-f7a20c3cda24.png)

1. Editor Panel
The Editor Panel is where you write and edit your Javascript code

2. Right Panel
- Console tab for printing output.
- Inspector tab for querying map results.
- Tasks tab for managing long running tasks.

3. Left Panel
- Scripts tab for managing your programming scripts.
- Docs tab for accessing documentation of Earth Engine objects and methods, as well as a few specific to the Code Editor application
- Assets tab for managing assets that you upload.

4. Interactive Map
- For visualizing map layer output

5. Search Bar 
- for finding datasets and places of interest

6. Help Menu
- User guide reference documentation
- Help forum Google group for discussing Earth Engine
- Shortcuts Keyboard shortcuts for the Code Editor
F- eature Tour overview of the Code Editor
- Feedback for sending feedback on the Code Editor
- Suggest a dataset

### Step 2. View Sentinel-2 Imagery for Omaha, NE on March 16th, 2019
Sentinel-2 passed over Omaha, NE on March 16th, 2019 following a major flood event and was able to capture a portion of the flood extent. To visualize this Sentinel-2 image copy the Javascript code below (copy whole cell with button in upper right) and paste into GEE's code editor. 

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

![image](https://user-images.githubusercontent.com/13280304/141536838-b0925512-3803-4ee0-8e45-46511528fd51.png)



## Part 2 - Application of your RF Classifier to flooding in Queensland, Australia on 
