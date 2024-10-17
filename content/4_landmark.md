---
title: Landmark Identification
nav: Landmark Identification
gallery: true
---

If these last two methods were unsuccessful and you are working with a photograph that contains landmarks like banks, libraries, statues or drinking fountains, you might want to use a resource named [Overpass Turbo](https://overpass-turbo.eu/). This is an interactive web based tool designed to create and run queries on the [OpenStreetMap](https://wiki.openstreetmap.org/wiki/Main_Page), a collaborative, volunteer generated geographic database that has been tagging these types of landmarks for over 20 years. 

<div class="symbol-container">
    <p class="symbol">&#10042;</p>
</div>

In addition to volunteer tagging, the database also pulls from infrastructural public datasets and aerial imagery. Overpass Turbo runs on the [Overpass API query language](https://wiki.openstreetmap.org/wiki/Overpass_API/Language_Guide) to search for an [extremely wide array of different built and natural environmental features](https://wiki.openstreetmap.org/wiki/Map_features). For example, using the code on the left side of the screen we are now seeing all of the fast food “amenities” in Moscow. [Click here to explore.](https://overpass-turbo.eu/map.html?Q=%5Bout%3Ajson%5D%5Btimeout%3A25%5D%3B%0A%2F%2F+define+the+area+for+Moscow%2C+ID%0Aarea%5Bname%3D%22Moscow%22%5D-%3E.searchArea%3B%0A%2F%2F+search+for+all+nodes%2C+ways%2C+and+relations+with+amenity%3Dfast_food+within+the+area%0A%28%0A++node%5B%22amenity%22%3D%22fast_food%22%5D%28area.searchArea%29%3B%0A++way%5B%22amenity%22%3D%22fast_food%22%5D%28area.searchArea%29%3B%0A++relation%5B%22amenity%22%3D%22fast_food%22%5D%28area.searchArea%29%3B%0A%29%3B%0A%2F%2F+output+results%0Aout+body%3B%0A%3E%3B%0Aout+skel+qt%3B%0A)

{% include gallery-figure.html img="geo_30.jpg" width="100%" alt="Map of Moscow with a coding area to the left and various nodes mostly along the major roads" caption="Overpass Turbo map of Moscow Fast Food locations" %}

One thing to consider when geolocating with these types of tools is whether you have enough points of reference to clearly identify a location. In triangulation you have (you guessed it) three points of reference that you can use to measure angles and distances between landmarks to identify an area. These points of reference can be landmarks or distinctive elements of the skyline. 

{% include gallery-figure.html img="geo_31.jpg" width="100%" alt="Group of people outside at a table eating near tents and automobiles." caption="Photograph PG070-04-03 from Unidentified Family Album, 1920-1939, Courtesy U of I Special Collections." %}

Take this photo from the scrapbook collection for instance. Because this photograph is located in the scrapbook between two others we can identify as Yellowstone National Park, there’s a good chance it is as well. 

**Using the following code**, we can search this area for the water tower we see on the left.


```
[out:json][timeout:25];

// Define the bounding box for the area around Yellowstone National Park
(
  node["man_made"="water_tower"](44.0030,-111.2045,45.0030,-109.2045);
);

// Output the results
out body;
>;
out skel qt;
```
{% include gallery-figure.html img="geo_32.jpg" width="100%" alt="Overpass Turbo map with a single node denoting the water tower in Yosemite National Park" caption="Resulting Overpass Turbo map with a single node denoting the water tower in Yosemite National Park" %}

{% include gallery-figure.html img="geo_33.jpg" width="75%" alt="Google Earth rendering of the water tower from the perspective of the photograph." caption="Google Earth rendering of the water tower from the perspective of the photograph." %}

**While we find that there is a water tower** located within an area that would likely have had the power line we see running across the block in the background, there aren’t enough landmarks or clearly visible landforms in the background of the image to orient us in space. 

{% include gallery-figure.html img="geo_34.jpg" width="100%" alt="Street scene with a bank, stone monument, street lights and mountain in the background highlighted for reference" caption="First National Bank with Mullan Monument, n.d. Courtesy U of I Special Collections." %}

In contrast, this photograph would be an example of an image that is ideal for triangulation, setting aside the fact that we actually know this is located in [Mullan, Idaho](https://www.lib.uidaho.edu/digital/tabor/items/tabor1552.html). We have a monument in the road, a series of street lamps, a structure that resembles a 19th century bank and a landform in the background with a fairly distinct shape. 

Using the [OpenStreetMap language](https://wiki.openstreetmap.org/wiki/Map_features) again, we can search for these elements within the state of Idaho:

```
[out:json][timeout:25];
// Define the area of Idaho using its boundary
area["admin_level"="4"]["name"="Idaho"]->.searchArea;

// Query for street lamps
(
  node["highway"="street_lamp"](area.searchArea);
  way["highway"="street_lamp"](area.searchArea);
  relation["highway"="street_lamp"](area.searchArea);
);

// Query for monuments
(
  node["historic"="monument"](area.searchArea);
  way["historic"="monument"](area.searchArea);
  relation["historic"="monument"](area.searchArea);
);

// Query for banks
(
  node["amenity"="bank"](area.searchArea);
  way["amenity"="bank"](area.searchArea);
  relation["amenity"="bank"](area.searchArea);
);

// Output the results
out body;
>;
out skel qt;
```

{% include gallery-figure.html img="geo_35.jpg" width="100%" alt="Overpass Turbo map with many nodes denoting all of the banks in Idaho." caption="Overpass Turbo map with many nodes denoting all of the banks in Idaho." %}

**The map this generates is indicative of some of the shortcomings OpenStreetMaps has** as a collaborative, volunteer driven platform: just because there is a tag for a type of landmark doesn’t mean that there will be consistent data for it. While there are **3439** banks identified here, we have no monuments or street lamps and even the bank in Mullan isn’t tagged. While Overpass Turbo is excellent at identifying more densely populated areas, it is generally not as strong in rural areas and there may be significant gaps in data for many landmarks.