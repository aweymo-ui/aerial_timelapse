---
title: Google Earth Engine
nav: Google Earth Engine
gallery: true
---

Like the USGS Earth Explorer, you'll need to sign a waiver confirming that you'll use the resource for non-profit purposes, but you should gain near-immediate access after signing. 

The interface resembles a text editor: the left pane displays the scripts you've written, along with documentation and examples of code you can plug in to test the platform. The center pane is where you write and run your code, while the right pane features a console that functions as a terminal, displaying error messages and printing data after running the script. The task pane lists the scripts you've run along with their status.

<div class="symbol-container">
    <p class="symbol">&#10042;</p>
</div>

Here's the code I developed to create a time lapse over Moscow, covering the longest possible time span.

```
// Define the center of the 83843 area code in Moscow, Idaho
var moscowCenter = ee.Geometry.Point([-117.0028, 46.7326]); // Approximate center of 83843

// Define a square boundary around Moscow, covering an area of interest (e.g., 5 square miles ~ 12.95 square km)
var moscowBoundary = moscowCenter.buffer(1810).bounds(); // 1810 meters radius approximates a 5 square mile area

// Load the ImageCollection and filter by the defined area
var collection = ee.ImageCollection('USDA/NAIP/DOQQ')
  .filterBounds(moscowBoundary)
  .sort('system:time_start');
  ```

  To start, we define our geographic perimeter around the Moscow area code. Next, we will specify the dataset to use for creating this visualization. Here we are focusing solely on the National Agriculture Imagery Program dataset, which goes back to around 1980 and we are doing this due to the many errors I encountered previously attempting to chain together datasets to increase the time frame.

  ```
// Get the earliest image
var earliestImage = collection.first();
var earliestDate;

if (earliestImage) {
  earliestDate = ee.Date(earliestImage.get('system:time_start')).format('YYYY-MM-dd').getInfo();
  print('Earliest Available Date:', earliestDate);

  // Define the updated date range
  var startDate = earliestDate; // Use earliest date found
  var endDate = '2024-06-01'; // End date

  // Filter the ImageCollection with the updated date range
  collection = collection.filterDate(startDate, endDate);

  // Function to get the image with the largest area for each year
  var getLargestImagePerYear = function(year) {
    var yearlyCollection = collection.filter(ee.Filter.calendarRange(year, year, 'year'));
    
    // Compute the intersection of each image with the boundary and calculate area
    var largestImage = yearlyCollection.map(function(image) {
      var intersection = image.geometry().intersection(moscowBoundary, 1); // Added error margin of 1 meter
      return image.set({'intersectionArea': intersection.area()});
    }).sort('intersectionArea', false).first(); // Sort by area and get the largest

    return largestImage;
  };

  // Get the years from the collection
  var years = ee.List(collection.aggregate_array('system:time_start'))
    .map(function(date) {
      return ee.Date(date).get('year');
    }).distinct();

  // Map over years to get the largest image per year
  var largestImagesCollection = ee.ImageCollection(years.map(getLargestImagePerYear))
    .sort('system:time_start'); // Ensure chronological order
```

Moving ahead, I encountered another complication: not all satellite photographs that capture parts of the area offer complete coverage, leaving distracting gaps in some of the earlier iterations of the code. To address this, I adjusted to request only the largest image for each year, ensuring optimal coverage and a better yield of images. A little farther down in the script, we are assembling these images chronologically, from earliest to most recent, and compiling them into a video form for export in red, green, and blue (RGB) color formatting. 

```
 // Function to prepare images for video export and add date metadata
  var prepareForExport = function(image) {
    var date = ee.Date(image.get('system:time_start')).format('YYYY-MM-dd');
    
    // Visualize the image with appropriate bands and scale
    var visualImage = image.visualize({
      bands: ['R', 'G', 'B'],
      min: 0,
      max: 255
    });
    
    // Annotate the image with the date (add metadata)
    return visualImage.set({'date': date});
  };

  // Apply the function to the largest images collection
  var annotatedCollection = largestImagesCollection.map(prepareForExport);

  // Print dates of all images to the console in chronological order
  annotatedCollection.aggregate_array('date').evaluate(function(dates) {
    print('Image Dates:', dates);
  }, function(error) {
    print('Error:', error);
  });
```
Ideally, I wanted to create code that would automatically overlay the date information onto each photograph, providing viewers with a better frame of reference. However, the best I could achieve was to have this metadata "printed" in the console when the script runs. 

<div class="symbol-container">
    <p class="symbol">&#10042;</p>
</div>


This data can then be easily added to the frames using video editing software like Adobe Premiere.
```
  // Define video export parameters with increased display time
  var videoParams = {
    region: moscowBoundary,
    dimensions: 900, // Adjust as needed
    crs: 'EPSG:3857',
    framesPerSecond: 0.5, // Increase time on screen to 2 seconds per image
    description: 'Moscow_TimeLapse',
    folder: 'EarthEngine'
  };

  // Export the video
  Export.video.toDrive({
    collection: annotatedCollection,
    description: videoParams.description,
    folder: videoParams.folder,
    dimensions: videoParams.dimensions,
    framesPerSecond: videoParams.framesPerSecond,
    crs: videoParams.crs,
    region: videoParams.region
  });

} else {
  print('No images found within the specified boundary.');
}
```

**Finally**, one of the convenient things about working with Google Earth Engine is that you can export videos, images and interactive maps you create directly into your Google Drive account. These exported video files can be converted into GIFs and loaded onto project sites like we see on the following page.

<div class="symbol-container">
    <p class="symbol">&#10042;</p>
</div>