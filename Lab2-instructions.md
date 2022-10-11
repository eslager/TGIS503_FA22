# Lab 2: Getting started with Leaflet

## Why Leaflet?

[Leaflet](https://leafletjs.com/) is one of many different frameworks that exist for making interactive web maps. I like it because it is free and open-source, is lightweight and flexible, and can be easily combined with other tools to expand its capabilities. Also, you can build Leaflet maps without signing up for any kind of account, which makes it an ideal framework for new learners.

Lab 2 will walk you through setting up a Leaflet map, adding data in a few different ways, and making interactive map markers with clickable pop-ups. 

## A basic Leaflet map

<iframe src="https://ejslgr.github.io/Leaflet-Intro/code-samples/basic.html" style="width:100%; height:500px;"></iframe> [View this example on its own](https://ejslgr.github.io/Leaflet-Intro/code-samples/basic.html)

The map above is a very simple Leaflet map. You can pan, zoom, and click it, but it only requires a few lines of code that include:

1. An HTML page with `<head>` and `<body>` elements
2. A link to the Leaflet CSS styles
3. A link to the Leaflet JavaScript library
4. A `<div>` element to hold the map
5. A height style specified for the map div
6. A bit of JavaScript to create the map in the div

View the code below with comments explaining what each part does:

```html
<html>
<head>
	<title>Intro to Leaflet</title>
 	<!-- the Leaflet CSS -->
	<link rel="stylesheet" href="https://unpkg.com/leaflet@1.5.1/dist/leaflet.css"/>
	<!-- the Leaflet JavaScript library -->
	<script src="https://unpkg.com/leaflet@1.5.1/dist/leaflet.js"></script>
</head>

<body>
  	<!-- code where we create and name the container that holds the map -->
	<div id="map" style="height: 95%"></div>
  	<!-- code where we build the map and its functionality -->
	<script>
		var mymap = L.map('map').setView([47.258, -122.465], 12);
		var basemap = L.tileLayer('https://stamen-tiles-{s}.a.ssl.fastly.net/terrain/{z}/{x}/{y}{r}.png', {
			attribution: 'Map tiles by <a href="http://stamen.com">Stamen Design</a>, <a href="http://creativecommons.org/licenses/by/3.0">CC BY 3.0</a> &mdash; Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
		}).addTo(mymap);
		var point = L.marker([47.258, -122.465]).addTo(mymap);
			point.bindPopup("<b>Hello world!</b><br>I am a popup.");
  	</script>
</body>
</html>
```

Let's take a closer look at the JavaScript code that creates the map. Inside the <script> tags, we have code that:

1. Creates the map variable (named "`mymap`")
2. Uses `L.map()` to initialize the map object, passing it the ID of the div where we want the map to go
3. Uses the `setView()` method to center the map on Tacoma (latitude 47.258 and longitude -122.465) and set the zoom level to 12
4. Uses `L.tileLayer()` to create the base layer of map tiles, specifying a URL template for the tile images. In this case, we're using tiles map designed by a company called Stamen, but we could use tiles from [many different sources](http://leaflet-extras.github.io/leaflet-providers/preview/), including Open Street Map, ESRI, or Carto. {z}/{x}/{y} is a template that Leaflet uses to find tiles at the correct zoom, x, and y coordinates. The code inside the `attribution` option sets the attribution text that appears in the bottom right corner of the map.
5. Uses the `addTo()` method to add the base tile layer to the map
6. Uses `L.marker()`to create a point marker at lat/long 42.258, -122.465 and `addTo()` to add the point marker to the map
7. Uses `.bindPopup` to create a popup that appears when the point marker is clicked. We use HTML `("<b>Hello world!</b><br>I am a popup.")` to provide the content of the popup.

## Try it yourself

To complete the rest of Lab 2, copy the code from this [starter file](https://github.com/ejslgr/Leaflet-Intro/blob/master/code-samples/earthquakes-starter.html), paste it into a new HTML document in Atom, and follow along to make the changes yourself. Keep your files organized: create a new folder for your Lab 2 files and save this file with the name `index.html`.

You'll note that the starter file is very similar to the basic Leaflet map we looked at above, except that we've removed the marker and changed the center and zoom level. Next, we'll add some data.

## Working with GeoJSON data

Adding data with the `L.marker()` method is simple, but it can be somewhat inconvenient. If we were mapping hundreds of points, we would have to manually type in the lat/long pairs for every point into our code. No thank you! Can't we just add a Shapefile?

Shapefiles, as you no doubt know, are the default vector data format when working with ArcGIS. With web mapping, however, the standard data format is GeoJSON. Like other formats for geospatial data, GeoJSON stores information about geographic features and their non-spatial attributes (e.g. a line indicating a street and the name of the street). It is based on JavaScript Object Notation, which means it will be more familiar to web developers than GIS professionals, but it's fairly easy to work with and understand.

Instead of storing data in tables, GeoJSON stores data in "key: value pairs." These are both machine readable and human readable. Here's an example:

```javascript
{ "type": "FeatureCollection",
  "features": [
    { "type": "Feature",
      "geometry": {"type": "Point", "coordinates": [102.0, 0.5]},
      "properties": {"name": "Example Point"}
      },
      
    { "type": "Feature",
      "geometry": {
        "type": "LineString",
        "coordinates": [
          [102.0, 0.0], [103.0, 1.0], [104.0, 0.0], [105.0, 1.0]
          ]},
      "properties": {
        "name": "example line",
        "number of vertices": 4
        }
      },
      
    { "type": "Feature",
       "geometry": {
         "type": "Polygon",
         "coordinates": [
           [ [100.0, 0.0], [101.0, 0.0], [101.0, 1.0],
             [100.0, 1.0], [100.0, 0.0] ]
           ]},
       "properties": {
         "name": "example polygon",
         "number of vertices": 5}
         }
    ]
  }
```

In this file, we have a collection of features. Each feature has a geometry and properties. The geometry describes the geographic feature. For instance, the first feature is a point located at lat: 0.5 and lon: 102.0. The properties are the non-spatial attributes. In this case, each feature has a name, and the line and polygon features also have a property that lists of the number of vertices in the shape.

If you want to know more about GeoJSON, a good place to start is its [Wikipedia page](https://en.wikipedia.org/wiki/GeoJSON). Note that many open data portals make data available to download in the GeoJSON format, but it's also possible to convert data in other formats (like Shapefiles, CSVs, KMLs, etc.) into GeoJSON with various tools. We'll cover data conversions in a future lab. 

## Adding GeoJSON data to our Leaflet map

To our map, we're going to add two different data layers using two different methods: 

1. a live GeoJSON feed of all the earthquakes (as point data) that occurred in the past day. USGS maintains numerous earthquake feeds, and you can see a summary of the information it makes available about these quakes here: https://earthquake.usgs.gov/earthquakes/feed/v1.0/geojson.php. 
2. a static GeoJSON of global active faults (as line data). This data set is maintained by the Global Earthquake Model Foundation, and you can find it here: https://github.com/cossatot/gem-global-active-faults

### Adding live data with jQuery

First, let's add the earthquake data. The earthquake data is a live feed, updated every minute, and thus we cannot download the data or we would lose its real-time usefulness. Instead, we will include it in our map by pointing to the URL of the feed. To add data in this way, we'll use something called jQuery to load the data from the feed into the map. 

USGS publishes numerous earthquake feeds. The feed we will choose for this visualization is all earthquakes of every magnitude that occurred in the last 24 hours: https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_day.geojson. If you click on this link, you'll recognize that what you're looking at is a GeoJSON file: a collection of features that have a specified geometry and numerous properties or non-spatial attributes.

In Atom, follow the steps below to add the lines of code as indicated, then view your changes in a web browser:

1. In the `head` of the HTML file, after the line of code where you add the leaflet.js script, insert the following:

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.5.1/jquery.min.js"></script>
```

This is a link to the jQuery library. jQuery is a common JavaScript library that makes it easy to make changes to a web page. In this case, we'll use it to load our GeoJSON file.

2. In the `body` of the HTML file, after the code where you initialize the map but before the `</script>` tag closes, add the following:

```javascript
// load earthquake GeoJSON from an external file and add it to the map as a layer
$.getJSON("https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_day.geojson",function(data){
    L.geoJson(data).addTo(mymap);
});
```

`$.getJSON` loads the file located at the URL that is specified. Then, `L.geoJson()` creates a vector layer from the GeoJSON and adds it to the map with `.addTo()`.

3. Save your work and open it in your web browser and it should look like this:

<iframe src="https://ejslgr.github.io/Leaflet-Intro/code-samples/earthquakes-ex1.html" style="width:100%; height:500px;"></iframe> [View this example on its own](https://ejslgr.github.io/Leaflet-Intro/code-samples/earthquakes-ex1.html)

### Adding static data

To add static data to our map, we could use the exact same process as we used with the live data: that is, we could use jQuery to load the data from a URL or a relative path name. However, when we're using static data, we can download the data and edit it, which gives us the ability--if we prefer--to use another method that does not use jQuery. I am teaching you both methods here so that when you make maps on your own later in the quarter, you can chose which method you prefer. But remember, you only get to choose when you're using static data that you can download and edit directly. Otherwise: use jQuery. Use these steps: 

1. Download the GeoJSON of the fault lines: 

   * From the Canvas assignment page, download the faultLines.geojson file and save it in your Lab 2 directory. I suggest saving it in a folder called 'data' that you put in the same folder that holds your Lab 2 index.html file. 
   * Note: I downloaded this file from [this location](https://github.com/cossatot/gem-global-active-faults), but I edited it to simplify the shape geometry and shrink it in size so that it is easier to work with. All credit for this data goes to the GEM Foundation, who makes the data available under a CC-SA 4.0 license.

2. Edit the file to change it from a GeoJSON file to a JavaScript file:

   * Add the following to the very beginning of the file: 

     ``` var faults = ```

     This creates a variable named 'faults' and sets the value of that variable equal to the GeoJSON data you just downloaded. 

   * File > Save As... and save as faults.js, making sure to put it in a location you can find again (such as the same 'data' folder where you saved your original GeoJSON file). It's very important to get that file type right--make sure it is a .js file, not a .geojson file. 

3. In the `<body>` of your `index.html` file, under the line of code where you create your map div (read the HTML comments if you need help finding this), add the following:

   ```html
   <!-- link that loads the fault line data from an external file -->
   <script type="text/javascript" src="data/faults.js"></script>
   ```

   *Note that if your `faults.js` file is not enclosed in a folder named `data`, you will have to edit that `src` file path. 

   This will load the data from the GeoJSON, much like `$.getJSON("https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_day.geojson"` did in the method above, but without us having to use jQuery. Next, we take that data and create a layer from it. 

4. In the JavaScript area of your code, after the section of code where you add the earthquake data as a layer, add the following: 

   ```javascript
   // add the fault line data to the map as a layer
   L.geoJson(faults).addTo(mymap);
   ```

   Here, `faults` is the name of the variable we created in the `faults.js` file to hold all of the fault line GeoJSON data. We use the `L.geoJson()` method to create a layer from that data, then use the `.addTo()` method to add the layer to the map. 

5. Save your work, and refresh it in your web browser. It should look like this: 

   <iframe src="https://ejslgr.github.io/Leaflet-Intro/code-samples/earthquakes-ex6.html" style="width:100%; height:500px;"></iframe> [View this example on its own](https://ejslgr.github.io/Leaflet-Intro/code-samples/earthquakes-ex6.html)

## Adding interactivity: clickable earthquake markers

If we hover over the markers, we should see the cursor change from the panning hand to the pointing finger, suggesting that we can click on the markers. However, clicking doesn't seem to do anything. Let's change that and make it so that when we click the markers, we get a pop-up window that tells us the location and magnitude of the earthquake and get a link we can click for more information. Thankfully each of these things (location, magnitude, and further info link) are available as properties in the earthquake GeoJSON.

Change the code that creates the GeoJSON layer as follows, adding the `pointToLayer` option to the `L.geoJson()` method before we add the layer to the map:

```javascript
$.getJSON("https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_day.geojson",function(data){
    L.geoJson(data, {
        pointToLayer: function(feature, latlng){
            var marker = L.marker(latlng);
            marker.bindPopup("hello!");
            return marker;
        }
    }).addTo(mymap);
});
```

`pointToLayer` is an option built into the `L.geoJson()` method that Leaflet uses to determine how to convert a point feature into a map layer. `pointToLayer` always accepts two arguments, the GeoJSON `feature` and `latlng`, which indicates the location of the feature. We then return the features as some kind of Leaflet layer, in this case, a marker layer, specified by `L.marker()`. Each marker appears at the given feature's specified lat,lng location.

Now, when we click on each earthquake marker, we should get a pop-up that reads "hello!" Save and try it out to make sure your code works as expected. 

Next, let's make the pop-up text a little more helpful. Change the `marker.bindPopup` line of code as follows:

```javascript
marker.bindPopup("Location: " + feature.properties.place + "<br>Magnitude: " + feature.properties.mag + "<br><a href =" + feature.properties.url + "target = '_blank'>More info</a>");
```

This code uses HTML to set the content of the marker pop-up. It selects information from GeoJSON using the `feature.properties. ` notation to display the 'place,' 'mag,' and 'url' properties for the selected feature. We can reference what these properties are in the [GeoJSON metadata](https://earthquake.usgs.gov/earthquakes/feed/v1.0/geojson.php). Save and test again. 

## Removing indications of interactivity where it's not there

Hover over one of the fault lines and notice that the panning hand icon switches to a pointer icon. This suggests that the fault lines are also clickable; however, when we click on them, nothing happens. This is expected, since we only made the markers interactive in the prior step. 

To avoid confusing your map user, it's important to remove any indications of interactivity where the map does not actually enable interaction. This is actually quite easy. When we create a layer in Leaflet using the L.geoJson method, we can specify *options* for that layer. These options might be things like styling to change the color of the layer, or the function we used to add pop-ups to the earthquake points. Another option turns of interactivity for the layer so that that pointer finger doesn't appear when the user hovers over a feature in the layer. 

To achieve this, change the section of the code that adds the fault line data to the map as a layer so that reads as follows: 

```javascript
		L.geoJson(faults, {
			interactive: false
		}).addTo(mymap);
```

Options go inside the curly brackets { } that we added here. 'Interactive' is the specific option we have set in this case, and it can take boolean values, meaning that it can be set to either 'true' or 'false.' The default value for the interactive option is true, so we've manually set it to false in order to disable the hover-over pointer. You can read more about this in the [Leaflet documentation](https://leafletjs.com/reference.html#interactive-layer), and in future labs, you'll get more practice reading and using documentation. 

## Adding a title and explanatory text

Finally, let's practice good cartography and add some vital map elements. We'll add the title and explanatory text outside of the map and in the web page itself. At the very top of the `<body>` of your html document, above the code where you create the div that holds the map, add the following line of code:

```html
<h1>Earthquakes that have occurred in the past 24 hours</h1>
```

When you save and preview the changes, you might now have to scroll down a bit to see both the header and the full map. Let's shrink the size of the map div so that this isn't the case. Change the `"height"` attribute of the map div to 80vh instead of 9vh (recall that vh stands for "view height" and is basically another way to write %):

```html
	<div id="map" style="height: 80vh"></div>
```

Sans-serif fonts are generally easier to read on digital screens than serified fonts, but our text defaults to the serified font Times New Roman. Let's change that with some CSS styling. In the `<head>` of the HTML document, beneath the lines of code where we add the Leaflet and JQuery links, add the following:

```html
	<!--CSS styles-->
	<style>
        body {
            font-family: sans-serif;
        }
	</style>
```

Next, let's add some explanatory text. In the `<body>` of the HTML document, below the code that creates the map div but above the opening `<script>` tag, add the following:

```html
<p>This map depicts all earthquakes that have occurred in the past 24 hours. Data comes from the <a href="https://earthquake.usgs.gov/earthquakes/feed/v1.0/geojson.php">USGS Live Earthquake Feed</a> and is updated every minute. Additionally, the map shows active faults using data from the <a href = "https://github.com/cossatot/gem-global-active-faults">GEM Foundation's Global Active Faults Database</a>. Click on an earthquake to find out more information about the event.</p>
<p>The Pacific Northwest lies along the Cascadia fault. Tectonic activity along this fault could cause a catastrophic earthquake that would produce major damage throughout our region. Learn about how researchers at the University of Washington are modelling earthquake risk and preparing for disaster response at the <a href="https://hazards.uw.edu/geology/m9/">M9 Project</a>.</p>
```

The `<p>` tag is an HTML element that we use to enclose paragraphs. The `<a>` tag (a for 'anchor') allows us to link to other URLs, which we use here to link to our data source and an external website where users can learn more about earthquake preparedness research in the PNW.

Having text span the entire width of the browser window makes for very short, wide paragraphs. Let's limit the width and center the main content on our page with a little extra CSS styling. Back in the `<head>` inside the `<style>` tags, update the CSS as follows:

```css
body {
	font-family: sans-serif;
	max-width: 900px;
	margin: auto;
}
```

Now, our map and the explanatory text will not exceed a width of 900px, no matter how wide the browser screen is. `margin: auto;` also centers the content by automatically adding margins of equal width on either side of the main body content. We could do a whole lot more with CSS to make our page appear more polished and professional, but this is enough for now. Feel free to take this further with more CSS styling if you wish!

Ordinarily, we might want to add a legend to a thematic map of this sort. However, adding a legend with HTML and JavaScript code is a somewhat complicated task, so we'll save that for next week. Plus, the clickable pop-ups fulfill some of the explanatory function that a legend might otherwise provide, and the map's interactivity lets us to get away with breaking some of the rules of more conventional cartography. This isn't always the case with interactive maps, but this one is simple enough that it works here. 

## Self-check comprehension questions

You do not need to answer these questions in your lab submission, but I've included them here so that you can test your comprehension of the concepts and skills presented here. I try to include these with most labs and expect you to complete them. If you can't figure out the answers or have any questions about them, please ask in class, as these are fundamental things you need to understand in order to become a good web mapper. 

1. Look at [the raw GeoJSON](https://raw.githubusercontent.com/cossatot/gem-global-active-faults/master/geojson/gem_active_faults.geojson) for the fault lines data. What attribute properties are included for each feature? If you were to include a pop-up for the fault lines layer, which of these properties would make the most sense to use as a label for each feature? 

   * hint: try installing and using the [JSON viewer extension](https://chrome.google.com/webstore/detail/json-viewer/gbmdgpbipfallnflgajpaliibnhdgobh?hl=en-US) in the Chrome browser if you are having a hard time reading the file in your web browser: 

2. Examine [the documentation for the pop-up object](https://leafletjs.com/reference.html#popup) in Leaflet, specifically at the 'Options' list (it's OK if this looks intimidating or hard to read right now! You'll get more comfortable with it as we go along). What would happen in terms of map interactivity if you set the 'keepInView' option to 'true' instead of its default value of 'false'? 

3. In this lab, we added CSS styling with which of the following methods? 

   a) in-line styles

   b) an embedded style sheet

   c) an external style sheet

## Submission

Upload your files to a GitHub repository as we went over in class, and enable Pages for the repository. Submit just the URL for your GitHub Pages site in the assignment dropbox on Canvas. The URL for this will look something like: username.github.io/repository_name.