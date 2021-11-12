# ua-geog330


```
// Sentinel-2 Image for floods in Omaha in 2019
var s2_omaha = ee.Image('COPERNICUS/S2/20190316T171039_20190316T171921_T14TQL')

var s2_viz = {opacity:1,
              bands:["B12","B8A","B4"],
              min:547.0221369809726,
              max:2790.664937848959,
              gamma:1}
Map.addLayer(s2_omaha, s2_viz, "Sentinel-2 - Omaha, MO")
```
