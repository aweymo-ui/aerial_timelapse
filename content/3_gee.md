---
title: Google Earth Engine
nav: Google Earth Engine
gallery: true
---

While I found the learning curve for this resource steep, its adaptability, variety of datasets, and convenient export settings make it an excellent choice for geolocation. Like the USGS Earth Explorer, you’ll need to sign a waiver confirming that you’ll use the resource for non-profit purposes, but you should gain near-immediate access after signing.

The interface resembles a text editor familiar to those with coding experience. The left pane displays the scripts you’ve written, along with examples of other scripts you can automatically generate to aid your learning of the programmatic language. You’ll also find the Google Earth Engine documentation here for specific functionalities. The center pane is where you write and run your code, while the right pane features a console that functions as a terminal, displaying error messages and printing data after script execution. The task pane lists the scripts you’ve run along with their status.

The intention of the code is to create a time lapse over Moscow, ID, covering the longest possible time span.

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

  To begin, we are defining our perimeter geographically around the 83843 area code. Next we are going to define the dataset we want to pull from to create this visualization. While Google Earth Engine contains over 900 datasets, for a beginner like me, I was unable to combine datasets to increase the span of time in any given area. I believe this is due to the datasets having slightly different metadata and contrasting color outputs so, for this script, we are only pulling from the National Agriculture Imagery Program dataset, which goes back to about 1980 and in line 13 you can see I am requesting the earliest possible image of this defined area. 

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

Moving  to line 28, another complication that I encountered was that not all of the satellite photographs that capture part of the area necessarily cover all of it, so some of the early iterations of this script had little slivers of the terrain that were unhelpful in understanding geographic change over time. To solve this I am asking in code for just the largest image for each year to balance getting the best coverage with the best yield of images. Starting around line 41, we are asking in code to assemble these images chronologically from earliest to most recent and compile them in a video format and export them in red, green and blue color formatting. 

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
**In a perfect world** I was hoping to create a code that would automatically overlay the date information of each photograph over its image, so the viewer would have a better frame of reference, but the best I was able to do was have this date information print when the script is run in the console. This information can then be fairly easily added over the frames using video editing software like Adobe Premiere
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

Finally, one of the convenient things about working with Google Earth Engine is that you can export videos, images and interactive maps you generate directly into your Drive. The exported video files can be converted into GIF files and loaded onto project sites like we see below. 