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

**1. Editor Panel**
The Editor Panel is where you write and edit your Javascript code

**2. Right Panel**
- Console tab for printing output.
- Inspector tab for querying map results.
- Tasks tab for managing long running tasks.

**3. Left Panel**
- Scripts tab for managing your programming scripts.
- Docs tab for accessing documentation of Earth Engine objects and methods, as well as a few specific to the Code Editor application
- Assets tab for managing assets that you upload.

**4. Interactive Map**
- For visualizing map layer output

**5. Search Bar** 
- for finding datasets and places of interest

**6. Help Menu**
- User guide reference documentation
- Help forum Google group for discussing Earth Engine
- Shortcuts Keyboard shortcuts for the Code Editor
F- eature Tour overview of the Code Editor
- Feedback for sending feedback on the Code Editor
- Suggest a dataset

### Step 2. View Sentinel-2 Imagery for Omaha, NE on March 16th, 2019
Sentinel-2 passed over Omaha, NE on March 16th, 2019 following a major flood event and was able to capture a portion of the flood extent. To visualize this Sentinel-2 image copy the Javascript code below (copy whole cell with button in upper right) and paste into GEE's code editor. Then zoom into the Omaha, NE area on the map.

```js
// Sentinel-2 Image for floods in Omaha in 2019
var s2_omaha = ee.Image('COPERNICUS/S2/20190316T171039_20190316T171921_T14TQL')

// Visualization parameters
var s2_viz = {bands:["B12","B8A","B4"],
              min:547.0221369809726,
              max:2790.664937848959,
              gamma:1} 
              
// Add to the Map
Map.addLayer(s2_omaha, s2_viz, "Sentinel-2 - Omaha, MO") 
```

The variable `s2_viz` provides details for how to visualize the Sentinel-2 imagery. Here we use an RGB combination of (B12/B8A/B4) or (SWIR/NIR/Red). The SWIR and NIR bands are particulary useful for visualizing water as these wavelenghts are highly absorbed by water. However, to change the visualization hover the mouse of the **Layers** button in the Map view and select the :gear: button.

![image](https://user-images.githubusercontent.com/13280304/141536838-b0925512-3803-4ee0-8e45-46511528fd51.png)

This will open up a menu of visualization paramters where you can select bands to view and different stretches of the image.

![image](https://user-images.githubusercontent.com/13280304/141537208-314aefb4-72d7-4cbf-a6fd-339eb86f71b7.png)


:question:
***Question 1: Create a new visualization of the Sentinel-2 imagery using a different bands combination. Take a screenshot of your new visualization, add to the submission document, and add 1-2 sentences describing the visualization and what you can see with this new visualization.***


### Step 3. Add the JRC Global Surface Water Dataset and Visualize
In order to classify the Sentinel-2 image for Omaha, NE, we need training and validation data. To collect this data we will use an alternative dataset known as the JRC Surface Water dataset. From this dataset we can represent areas of permanent water from which to sample water versus non-water (i.e. land) pixels. Copy and paste the code below into the GEE Code Editor (make sure it is BELOW your previous code), and hit **Run**. 

```js
// JRC Global Surface Water dataset
var jrc = ee.ImageCollection("JRC/GSW1_3/YearlyHistory")

// The YearlyHistory dataset has surface water for every year, let's use 2019
var jrc_2019 = jrc.filter(ee.Filter.eq('year', 2019)).first()

// pixel values of 3 = permanent water in the JRC classification
var jrc_permanent = jrc_2019.eq(3) 
  .unmask() // non-water areas are "masked" from the image, we want to sample these areas too

// Finally, the JRC dataset is global so we "clip" to our satellite imager
var jrc_omaha = jrc_permanent.clip(s2_omaha.geometry())

// Add to the map
Map.addLayer(jrc_omaha, {palette:["white","blue"]}, "JRC Permanent Water")

```
The JRC Global Surface Water dataset was added to the Map. Zoom in and investigate this layer. To investigate actual pixel values, select the **Inspector** tab and then click anywhere on the map to get the value of a pixel. 

![image](https://user-images.githubusercontent.com/13280304/141839208-f4d1e1b9-4318-43d5-8352-8aeb6f59cb9a.png)

After investigating the map, answer **Question 2** using the submission document.


:question:
***Question 2: In the JRC dataset you just prepared, what does a pixel value of 1 represent?***


### Step 4. Sample Training and Validation Data
Now that we have a Sentinel-2 image and a classified map of water (JRC GSW dataset) to stratify our sample, we can go ahead and sample our data. Copy and paste the code below into the GEE Code Editor, as always below the code you have already written, and hit **Run**.

```js
// Create band lists that we can use to subset our data
// to the band we want to use in models
var all_bands = ['B1', 'B2', 'B3', 'B4', 'B5', 'B6', 'B7', 
                 'B8', 'B8A', 'B9', 'B11', 'B12']
var viz_bands = ['B2', 'B3', 'B4']

// Create a dataset that include the Sentinel-2 image for Omaha
// based on bands we care about and the classification map
// we will use to sample our data.
var omaha_input = s2_omaha.select(all_bands) // we'll use all bands in our classification
  .addBands(jrc_omaha) // layer used to stratify sample


// Sample the Omaha, NE image by stratifying the image between
// water and land classes
var seed = 123 // DO NOT CHANGE THIS! Used so everyone gets the same answer for grading
var omaha_sample = omaha_input.stratifiedSample({
  numPoints: 1500,
  classBand: 'waterClass',
  classValues: [0,1],
  classPoints: [750, 750],
  dropNulls: true,
  geometries: true,
  seed: seed
});

print(omaha_sample)
print("# of points in land class", omaha_sample.filter(ee.Filter.eq("waterClass",0)).size())
print("# of points in water class", omaha_sample.filter(ee.Filter.eq("waterClass",1)).size())
Map.addLayer(omaha_sample, {}, "Omaha stratified sample")
```

In the `stratifiedSample` function, there are a few important paramters. First, the `classBand` argument tells function which band in the `omaha_input` variable is to be used for stratifying the data. The `classValues` and `classPoints` arguments then specify how many points to sample per class, in our case the land (0) and water (1) classes. 

After we collected the sample, the data was added to the Map and also printed out in the console. Invesigate the print outs in the **Console** and answer Questions 3 & 4 using the submission document.

![image](https://user-images.githubusercontent.com/13280304/141834304-da933ac7-9313-499b-8b4b-4eca4a46b76e.png)


:question:
***Question 3: How many points were sampled in the land and water classes from the Sentinel-2 dataset?***

***Question 4: Why did we use a stratified sample? What might happen to the values in each class if we took a "random" sample across the Sentinel-2 image?***


### Step 5. Split the sample into training/ validation datasets
Now that we have a sample, we want to further split the sample dataset into a "training" and "validation" set. In short, we will use the training dataset create classifiers that can make prediction of whether a pixel is land or water. Then validation dataset will then be used to assess the skill or accuracy of the trained classifier against new data or data the model/ algorithm has never "seen" before. 

```js
// Split the data into training and validation datasets

// The randomColumn() method will add a column of uniform random
// numbers between 0 and 1 in a column named 'random' by default.
omaha_sample = omaha_sample.randomColumn({seed:seed})

// We want to roughly split the data into 75% training, 25% validation.
// To do this we filter the random column at the split we're aiming for.
var split = 0.75

// We can now filter the "omaha_sample" by the split, with ~75% of the
// data in the training dataset and ~25% in the validation dataset.
var omaha_training = omaha_sample.filter(ee.Filter.lt('random', split))
var omaha_validation = omaha_sample.filter(ee.Filter.gte('random', split))

// Print out the number of points in each dataset.
print("# of points in training dataset", omaha_training.size())
print("# of points in validation dataset", omaha_validation.size())
```

Invesigate the print outs in the **Console** and answer Question 5 using the submission document.

:question:
***Question 5: How many data points are in the training dataset? How many in the validation dataset?***


### Step 6. Train and Evaluate Random Forest (RF) and Support Vector Machine (SVM) Classifiers
All the prior steps were simply data preparation to TRAIN and EVALUTE (VALIDATE) machine learning classifiers. Now that we have this data prepared we can input our training dataset into several classifiers and evaluate its performance. The code below runs a Random Forest (RF) classifier, evaluates its accuracy, and adds a classified image to the map. Copy and paste the code below into the GEE Code Editor and hit **Run**

```js
// Apply the data to a Random Forest (RF) classifier

// Initiate the classifier and its parameters
var omaha_classifier = ee.Classifier.smileRandomForest({
  numberOfTrees:10, 
  seed:seed})

// Train the classifier with the training data
// and using ALL the Sentinel-2 bands
omaha_classifier = omaha_classifier.train({
  features: omaha_training,
  classProperty: 'waterClass',
  inputProperties: viz_bands // we defined this earlier; all S2 bands
  })

// Calculate the accuracy with the validation data by
// first classifying the validation data with the
// classifer we just trained
var omaha_validated = omaha_validation.classify(omaha_classifier)

// Get an error matrix representing expected accuracy.
var omaha_test = omaha_validated.errorMatrix('waterClass', 'classification')
print('Validation overall accuracy: ', omaha_test.accuracy())

// Add the classified data to the map
var omaha_classified = omaha_input.classify(omaha_classifier)
Map.addLayer(omaha_classified.selfMask(), {palette:"red"}, "LULC Classification (Omaha)")
```

The overall accuracy of the Random Forest classifier is 90.9%, an impressive result. But can we do better? Note that when we train the `omaha_classifier`, as input data we only use the visible bands of Sentinel-2 (e.g. Red, Green, Blue). This is specified in the `train()` function with the `inputProperties` parameter. In addition, we only try one classifier while there are several other options. For example, instead of using `ee.Classifier.smileRandomForest()` we could use a classifier using the Support Vector Machines (SVM) algorithm. Search the **Docs** tab for SVM and read the documentation on this classifier.

![image](https://user-images.githubusercontent.com/13280304/141838506-5ea03142-6e91-412e-b566-44d1464133ee.png)

There are several options we can explore here to improve the overall accuracy of our classifier. Toggle the following inputs and fill out the table associated with Question 6 in the submission document.

1. Change the `inputProperties` argument to the `all_bands` variable
2. Convert the Random Forest classifer (`ee.Classifier.smileRandomForest()`) to a SVM classifier (use the default parameters)

:question:
***Question 6: In Table 1, report the accuracy of the four different classifiers you trained represnting combinations of RF/SVM classifiers and all/ visible band inputs. Which classifier has the highest overall accuracy?***



### Step 7. Make Final Flood Map
**IMPORTANT NOTE** From this point forward we will be using your most accurate classification method. Be sure to have the parameters in your code set to this method.

In the previous step, we added an the classified image to the Map referred to as `LULC Classification (Omaha)`. Recall the we 1) acquired a Sentinel-2 image during a flood event 2) sampled a permanent water (e.g. rivers, ponds, lakes) dataset and 3) classified the Sentinel-2 flood image with permanent water dataset. Using your most accurate classification method, investigate the `LULC Classification (Omaha)` image and answer Question 7.

:question:
***Question 7: What types of water does your LULC Classification (Omaha) map represent? Why might this be a problem if you're interested in only FLOOD waters?***

To represent flood water, we can remove "permanent water" from our classification image. To do so, we can simply revert back to the JRC Global Surface Water (GSW) dataset that represent permanent water (e.g. rivers, lakes, ponds) and "mask" out permanet water. Copy and paste the following code into the code editor and hit **Run**

```js
// Remove permanet water
function maskPermanentWater(class_img){
  var jrc_mask = jrc_permanent.neq(1)
  return class_img.updateMask(jrc_mask)
}

var omaha_flood = maskPermanentWater(omaha_classified)
Map.addLayer(omaha_flood.selfMask(), {palette:"lightblue"}, "Flood Map (Omaha)")
```

Toggle on the "Flood Map (Omaha)" layer and answer Question 8.

![image](https://user-images.githubusercontent.com/13280304/141842460-0ced0284-d7f1-42fe-8492-f434ae1f3a4c.png)


:question:
***Question 8: Zoom into a flood affected area (e.g. buildings, agricultural fields) and take a screenshot (use the "Snipping Tool") of your flood map. Paste the screenshot into the submission document.***
```

## Part 2 - Application of your best classifier to flooding in Queensland, Australia
Impressed with your work for FEMA and in Omaha, the municipal government of Rockhampton, Queensland, Australia reached out to you and asked for a flood map representing the extent of flooding in 2017. Since you already created a classifier for flood in Omaha, NE you decide as a quick analysis to apply this classifier to Sentinel-2 imagery for the 2017 floods in Rockhampton, QLD. The steps below walk through how to create a flood map for Rockhmapton and evaluate its accuracy.

### Step 1. View Sentinel-2 Imagery for Rockhampton, QLD on April 8th, 2017
To view the Sentinel-2 satellite image of flooding in Rockhampton, QLD in 2017 copy and paste the code below into the GEE Code Editor and click **Run**.

```js
// ROCKHAMPTON, QUEENSLAND, AUSTRALIA
// Sentinel-2 Image for floods in Rockhampton in 2017
var s2_qld = ee.Image("COPERNICUS/S2/20170408T001211_20170408T001211_T55KHQ")
Map.addLayer(s2_qld, s2_viz, "Sentinel-2 - Queensland, AUS")
```

To view the image, pan to Australia and Zoom into the Rockhampton where you see a satellite image added to the map. Invesigate the image, using different visualizations if you choose, and answer Question 9.

:question:
***Question 9: How does the Sentinel-2 image for Rockhampton, Queesland differ from that of Omaha, NE (hint: think about data noise)? How might this present challenges for flood mapping?***

### Step 2. Create a stratified sample for Rockhampton, QLD
To evaluate the accuracy of our classifier, as before we need to create a sample of data that can be used for validation. To create a stratified sample, copy and paste the code below into the GEE Code Editor and click **Run**.

```js
// Clip the area of the JRC Global Surface Water dataset
// to the same extent as the Sentinel-2 image for Rockhampton, Queensland
var jrc_qld = jrc_permanent.clip(s2_qld.geometry())
var qld_input = s2_qld.select(all_bands) // we'll use all bands in our classification
  .addBands(jrc_qld) // layer used to stratify sample

// Create a region that we want to sample over, this step is so
// we don't sample ocean water along the coast of Rockhampton
var aus = countries.filter(ee.Filter.eq("country_na", "Australia")).first()
var aus_img_geo = s2_qld.geometry().intersection(aus.geometry())

// Get stratified sample
var qld_sample = qld_input.stratifiedSample({
  numPoints: 800,
  classBand: 'waterClass',
  classValues: [0,1],
  classPoints: [400, 400],
  region: aus_img_geo,
  dropNulls: true,
  geometries: true,
  seed: seed
})

// Add to the map
Map.addLayer(qld_sample, {}, "Rockhampton stratified sample")
```

### Step 3. Evaluate the accuracy of your classifier for the 2017 Rockhampton, QLD floods
Using the sample generated in the previous step, we can first classify the sample and then evaluate its accuracy. To do this, copy and paste the code below into the GEE Code Editor and click **Run**.

```js
// Classify the validation data and evaluate the accuracy
var qld_validated = qld_sample.classify(omaha_classifier)
var qld_test = qld_validated.errorMatrix('waterClass', 'classification')

// Accuracy print outs
print("Overall acc., Rockhampton, QLD", qld_test.accuracy())
print("Producers acc., Rockhampton, QLD", qld_test.producersAccuracy())
print("Users acc., Rockhampton, QLD", qld_test.consumersAccuracy())
```

View the print outs in the Console and answer Question 10.

:question:
***Question 10: What is the overall accuracy of your classifier when applied to the 2017 floods in Rockhampton, QLD? What is the producer's and user's accuracy for the water class?***

### Step 4. Create Flood Map for Rockhampton, QLD
Finally, as before we can also make a map of flooded area. To do this, copy and paste the code below into the GEE Code Editor and click **Run**.

```js
// Classify the Sentinel-2 image in Rockhampton and add to the map
var qld_classified = qld_input.classify(omaha_classifier)
Map.addLayer(qld_classified.unmask().selfMask(), {palette:"red"}, "LULC Classification (Rockhampton)")

// Mask permanent water and add flood image to the map
var qld_flood = maskPermanentWater(qld_classified)
Map.addLayer(qld_flood.selfMask().clip(aus_img_geo), {palette:"lightblue"}, "Flood Map (Rockhampton)")
```

Zoom into the Map and view the flood map you created for Rockhampton, QLD. Pay particular attention to areas of misclassification and answer Question 11.

:question:
***Question 11: Identify one systematic (repeated) misclassification in your flood map for Rockhampton, QLD (hint - think of something that looks like water, but isn't water). Why is this misclassification important for flood mapping in the context of urban planning, disaster response, or recovery efforts?***


## Part 3 - Submission
To complete Lab 11 submit the following:

1.	The submission document (available on D2L) renamed as **G330.Lab11.[LastName].docx** with the following:
a.	All the questions answered
b.	The link to your GEE code copy and pasted at the end of the submission document in the indicated space. To do this click the **Get Link** button and copy and paste the URL.

![image](https://user-images.githubusercontent.com/13280304/141849048-852ba6b4-b96c-458c-90f9-760485c49dae.png)

