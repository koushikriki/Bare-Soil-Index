# Bare-Soil-Index
// Import an uploaded feature collection
var west = ee.FeatureCollection("users/dynamicriki1999/west");
Map.addLayer(west, {color: 'red'}, "West Tripura");
Map.centerObject(west, 10);


// Import sentinal 2 image collection

var s2 = ee.ImageCollection("COPERNICUS/S2_SR_HARMONIZED");
print(s2.size());

// filter image collection by location, date, metadata
var filteredS2 = s2.filterBounds(west)
                   .filterDate("2020-01-01","2020-12-31")
                   .filterMetadata("CLOUDY_PIXEL_PERCENTAGE", "less_than", 10);
print(filteredS2.size());
print(filteredS2);

// sort cloud free image by asseding

var cloudFreeImage = filteredS2.sort("CLOUDY_PIXEL_PERCENTAGE")
                               .first();
print(cloudFreeImage);

var thirdImage = ee.Image("COPERNICUS/S2_SR_HARMONIZED/20200117T043121_20200117T043313_T46QCM");


Map.addLayer(thirdImage, {}, "Cloud Free Sentinal 2 Image");


 // Create median composite
var composite = filteredS2.median();
print(composite);
 var rgbVis = {
   min: 161,
   max: 1275,
   bands:["B4", "B3", "B2"]
 };
Map.addLayer(composite, rgbVis, "Median Composite Image");

// clip image with the region of interest
var clippedImage = composite.clip(west)
                             .select("B.*");
print(clippedImage);
Map.addLayer(clippedImage, rgbVis, "Clipped Image");

//Calculatr Bare SOIL index BSI = ((Red+SWIR) - (NIR+Blue)) / ((Red+SWIR) + (NIR+Blue))
var bsi = clippedImage.expression({
  expression: "((RED+SWIR) - (NIR+BLUE)) / ((RED+SWIR) + (NIR+BLUE))", 
  map: {
     "RED": clippedImage.select("B4").multiply(0.0001),
     "SWIR": clippedImage.select("B11").multiply(0.0001),
     "NIR": clippedImage.select("B8").multiply(0.0001),
     "BLUE": clippedImage.select("B2").multiply(0.0001),
  }
}).rename("BSI");
var bsiVis = {
  min: -0.21357506080035377,
  max:  0.10121749153519195,
  palette: ['#ffffd4','#fed98e','#fe9929','#cc4c02']
};
Map.addLayer(bsi,  bsiVis, "BSI IMAGE");

  
  






