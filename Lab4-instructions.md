# Lab 4: Converting to GeoJSON

## TGIS 503, Fall 2022, Dr. Emma Slager

### Introduction

In this lab, you'll practice converting data from Shapefile to GeoJSON so that it can be displayed in a web map and learn two ways to simplify GeoJSON data to improve map performance. Additionally, you'll gain experience with a new visualization method and Leaflet plugin that allows you to interactively cluster point data.

### Part 1: Converting from Shapefile to GeoJSON

From Canvas, download the Zip folder named '2020_Census_Tracts.' This folder contains a shapefile of Census tracts in Pierce County, with basic demographic and housing information for the Census tracts in its attribute table. The data originally came from the Pierce County GIS data portal and is currently projected using the NAD 1983 HARN State Plane Washington South projected coordinate system. 

Before you can display the Census tract data in a web map, you first need to get it into useable shape. At minimum this means converting it to GeoJSON in an appropriate projection. Here, we will also simplify and shrink the data to improve map performance and load times. 

Follow these basic steps, consulting your lecture notes and any help forums or guides you might find by searching the web as needed: 

1. Extract (decompress) the Zip folder so that you can load the data into ArcMap or ArcPro (or QGIS if you prefer).
2. In ArcMap or ArcPro, reproject the shapefile so that it is in the **WGS84 Geographic Coordinate System.** Remember that there are Projected Coordinate Systems that use the WGS84 datum, but we need our data to be unprojected so that the coordinate information for each feature is readable to Leaflet as lat/long coordinate pairs, rather than as projected coordinate pairs. 
3. Still in ArcMap or ArcPro, use the 'Features To JSON' tool to export your reprojected shapefile as a GeoJSON. Be sure to check the box to save the output as a GeoJSON. Do <u>not</u> check the box to save the output as a formatted JSON, as this will greatly increase the file size. Save the file in a location you can find it again. 
4. Validate your file by uploading it (open > file) to [geojson.io](https://geojson.io/). If the Pierce County census tracts appear over the basemap, you've successfully converted your file--congrats! If the layer doesn't appear, you've made a mistake and should consult your notes and try again. 

If you've successfully converted and validated your file, you could go ahead and load it into Leaflet. However, we're going to try another way of converting the file and compare the results. 

Mapshaper is an online converter and editor for map data. It's a piece of JavaScript software originally developed by [Matthew Bloch](https://github.com/mbloch), graphics editor at the New York Times. 

Visit [Mapshaper.org](https://mapshaper.org/) and follow these basic steps, consulting your lecture notes and any help forums or guides you might find by searching the web as needed: 

1. Upload your reprojected (the one using WGS84) shapefile of Pierce County Census tracts to the site, using drag and drop or the `select` button. Note: you must upload the .shp file *and all of its supporting files* (.dbf, .shx, .prj, etc.), not just the .shp. 
2. Leave the options at their defaults and hit `import`
3. Use the `Simplify` button to simplify the geometry of the file. Consult the [Mapshaper documentation](https://github.com/mbloch/mapshaper/wiki) as needed to understand the options here. You should simplify the geometry to at least the level of 10% of vertices remaining but can go even farther if you wish. 
4. Export your simplified file to GeoJSON and save the file in a location you can find it again. Give it a name that differentiates it from the GeoJSON file you produced with ArcGIS. 
5. Validate your file using [geojson.io](https://geojson.io/) as you did with your first GeoJSON. If it doesn't appear, figure out what you did wrong and correct your mistakes. 

In your write up, answer the following questions: 

* Using your file manager (e.g. Windows File Explorer or MacOS Finder), find the the size of the two GeoJSON files you've just created. What are their sizes? Based only on this information, which do you think is better to use in your web map and why? 
* Describe which method (ArcMap export or Mapshaper web converter) you prefer for converting Shapefiles to GeoJSON, and justify your answer. (There is no correct answer here; I just want you to reflect on which workflow <u>you</u> prefer.) Your answer doesn't need to be longer than a brief paragraph. 

### Part 2: Simplifying attribute data and clustering points

In this part of the lab, you'll convert another shapefile into a GeoJSON that can be displayed in a Leaflet Map. This time, instead of simplifying geometry, you'll simplify the layer's attribute information as a way of shrinking the data file. Then, you'll learn how to implement a Leaflet plugin called Leaflet.markercluster to interactively cluster markers on your map in order to declutter dense point data. 

From Canvas, download the Zip folder named 'RECREATION_BikeRacks'. This contains a point layer of all the bike racks in Cambridge, Massachusetts, as of June 2022. The data is currently projected, using the State Plane Massachusetts Mainland projected coordinate system. 

Unzip the data and reproject it into the WGS 84 geographic coordinate system. Before you convert to GeoJSON, simplify and shrink the data by deleting unnecessary attribute data, using the following steps: 

* Open the attribute table and glance through the fields. Some of the fields are helpful, but some provide very little useful data or contain mostly null values, and others are artifacts leftover from  data collection that are no longer needed. 
* In the attribute table, remove unnecessary fields by right clicking on the field names and selecting 'Delete'. Remove every field <u>**except**</u> Capacity and Status (additionally, FID and the Shape constitute the geometry data of each feature and cannot be deleted). Note: you should *delete* the fields and not merely turn them off. 
* Now, export to GeoJSON using whichever method you prefer (online conversion tool or ArcMap export tool).
* Validate your file using [geojson.io](https://geojson.io/). If it doesn't appear, figure out what you did wrong and correct your mistakes. 

Next we'll create a map using the newly converted bike rack data. Set up a new folder and GitHub repository to hold your files, and create an index.html file using the skills you've learned in previous labs. You may copy a starter file from an earlier lab to set this up or create it from scratch. 

Add links to the Leaflet CSS and JS libraries, create a map div, and initialize a map with basemap tiles of your choice. Though you could give the map div any id you want, name it `mymap` to make it easier to follow my instructions below. Set your initial extent and zoom level appropriately for a map of Cambridge, MA (hint: you may need to do some Googling to find a good starting lat, long).

Add your bike rack data using the jQuery method you learned in Lab 2. Make your bike rack data clickable with some popups by changing your code as follows: 

```javascript
    L.geoJson(data, {
        pointToLayer: function(feature, latlng){
            var marker = L.marker(latlng);
            marker.bindPopup("hello!");
            return marker;
        }
    }).addTo(mymap);
```

On your own, change the code above so that instead of creating popups that say "hello!", your popups provide information on the capacity and status of each feature (Hint: recall that the syntax for accessing a property stored in the GeoJSON is `feature.properties.PropertyName` where PropertyName is replaced with the name of the particular property you want to display.)

This is a functional map, but the point markers are very dense and cluttered. Next, we'll use the [Leaflet.markercluster plugin](https://github.com/Leaflet/Leaflet.markercluster) to cluster our markers in order to declutter things. Follow these steps:

* First, add links in the `<head>` of your HTML to the Leaflet.markercluster CSS and JS libraries, using a CDN: 

  ```html
  <!-- Leaflet.markercluster CSS links -->
  <link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster@1.4.1/dist/MarkerCluster.css"/>
  <link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster@1.4.1/dist/MarkerCluster.Default.css"/>
  <!-- Leaflet.markercluster JS link -->
  <script src="https://unpkg.com/leaflet.markercluster@1.4.1/dist/leaflet.markercluster.js"></script>
  ```

* Next in the JS code, assign the GeoJSON layer a variable name; `bikeRacks` would be a logical choice for the name. The layer options that create the markers remain the same, but we need this variable name to use the GeoJSON layer later: 

  ```javascript <pre>
  var bikeRacks = L.geoJson(data, {
  		pointToLayer: function(feature, latlng){
              var marker = L.marker(latlng);
              marker.bindPopup("Capacity: " + feature.properties.Capacity + "<br>Status: " + feature.properties.Status);
              return marker;
  		}
  }).addTo(mymap);
  ```

* Additionally, remove the .`addTo()` method that immediately adds the GeoJSON layer the map: 

  ```javascript
  var bikeRacks = L.geoJson(data, {
  		...
          }
  });
  ```

* Now, we use methods from the Leaflet.clustermarker library to add the markers back to the map, this time clustered based on proximity. Immediately after the `});` that closes the bikeRacks variable but before the `});` that closes the `$getJSON` function, add the following:

  ```javascript
  var clusters = L.markerClusterGroup();
  clusters.addLayer(bikeRacks);
  mymap.addLayer(clusters);
  ```

  A couple of notes about the above: First, `bikeRacks` is the variable name you assigned to the L.geoJson layer above. If you chose a different name for the variable that holds your GeoJSON layer, be sure to use the variable name you chose. Second, observe that each line of code does the following: 

  * the first line creates a cluster group and names it `clusters`
  * the second line uses the `bikeRacks` variable to add the bike rack layer to the cluster group
  * the third line adds the cluster group (and thus the bike rack layer) to map. 

Save your changes, preview them in the browser, and trouble shoot any problems that may arise. 

Notice that when you hover over any of the cluster group icons, a blue polygon appears showing the coverage area of the markers in the cluster group. This is an option that is turned on by default by the Leaflet.clustermarker library, but I personally find it distracting. If you set the `showCoverageonHover` [option](https://github.com/Leaflet/Leaflet.markercluster#options) (link is to the plugin documentation) to `false`, you can disable the blue polygon. Using the plugin documentation, set this option to false so that no blue polygons appear on hover. 

Make your web page and map into a final version: add a title and some brief explanatory text that explains what the map shows, adjust the size of the map canvas, and include attribution for the bike rack data (original source is here: https://www.cambridgema.gov/GIS/gisdatadictionary/Recreation/RECREATION_BikeRacks). Make any CSS adjustments you wish to make for the page design. 

OPTIONAL BONUS CHALLENGE (points awarded on a case-by-case basis): change the options of L.marker to display the unclustered points with a custom image icon--perhaps [something like this](https://commons.wikimedia.org/wiki/File:Biciclet%C3%A1rio2.svg). Some resources for helping you do this: 

* The [Leaflet custom icon tutorial](https://leafletjs.com/examples/custom-icons/)
* The "Add some style" section of [this Maptime tutorial](https://maptimeboston.github.io/leaflet-intro/)

When you're satisfied with your map, upload your files to your GitHub repository, enable Pages, and submit the link to your work on Canvas. 

### What to submit

For Part 1, please submit a write-up with the answers to the questions in Part 1 and submit the simplified GeoJSON you created of the Pierce County Census tracts. 

For Part 2, please submit a link to your map. 

Note: grading will be easier for me if you submit your files separately, rather than as a .zip folder, so please no .zips! 

