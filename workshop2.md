# ARCH 5115 - QGIS Workshop, Day 2
<https://kgjenkins.github.io/arch-5115-guyana/>

Workshop 2022-02-01 by Keith Jenkins, GIS Librarian at Mann Library

For help after this workshop, contact me at kgj2@cornell.edu  
Or set up a Zoom appointment at <https://guides.library.cornell.edu/gis/help>

All the data and documentation for this workshop can be downloaded from:  
<https://github.com/kgjenkins/arch-5115-guyana/archive/main.zip>


## Elevation raster data

Various scales of elevation data is available for different parts of the world.  Some places like Ithaca, NY may have 1- or 2-meter resolution data available from a local lidar survey.  But for much of the world, the best available is 30-meter SRTM data, collected as part of the Shuttle Radar Topography Mission in 2000.

Due to the large size of these elevation datasets, downloads are divided into smaller tiles.  SRTM data is available in 1x1 degree tiles.  There are various ways of accessing this data, including the "SRTM Downloader" plugin for QGIS, which makes it fairly easy to download all the tiles needed for a given area.  Downloading the data requires registering for a free NASA EarthData login, but data tiles have already been downloaded for you using <http://dwtkns.com/srtm30m/> and merged to a single GeoTIFF (.tif) file for each resolution.

* Drag the srtm30m_guyana.tif file onto your project

By default, raster data is displayed with a gray gradient, with higher values brighter.  We can change the style to bring out more details in the data.

* In the Layer Styling panel, change from "Singleband gray" to "Singleband pseudocolor"
* Try changing the color ramp to "Turbo"

Notice the values assigned to each color.  You will probably want to adjust those values to better represent sea level.  Try the following, and notice how the map changes at each step:

* Double-click the value for light blue and set it to 0
* Set the green value to 1
* Set the orange value to 100

Zoom into an area along the coast.  With those settings, any value 0 or less will be bluish, and anything 1 or higher will be green.  Notice that there are some land areas near the shore that are actually below sea level, according to this dataset.

To better see subtle elevation differences in relatively flat areas like the coast, we can set the color scale to dynamically change based on the values currently visible on the map:

* Expand the "Min / max Value Settings" section of the styling panel
* Select "Min / max"
* Set the "Statistics extent" to "Updated canvas"


## Value Tool plugin

Although we could use the "Identify" tool to click pixels and see their values, the "Value Tool" plugin will allow you to instantly show the values from raster layers without even having to click.

* Plugins > Manage and Install Plugins...
* Search all plugins for "value tool" and install it

The Value Tool will appear on your toolbar as a green circle with white "i".

* Click the Value Tool button to open the panel
* Check the box to enable the tool
* Move your cursor across the map to view the raster values

Try checking the elevations for various parts of the country.  Note that the elevations are in meters, and are rounded to the nearest meter.

Due to the nature of the SRTM dataset, with heights being measured from the space shuttle, there are limits to the precision of the measurements.  Also, the dataset may show higher elevations for areas with tall trees or buildings.


## Elevation hillshade

Even though we can see the differences between the highest and lowest parts of the country, the area along the coast looks rather flat and devoid of detail.  Let's use a hillshade to made it look more 3-dimensional.

* In the Layers Panel, right-click the elevation layer > Duplicate
* Right-click "elevation copy" > Rename to "hillshade"
* Move the "hillshade" layer just above "elevation" and check the box to display the hillshade layer
* Open the styling panel be sure that you are styling the "hillshade" layer
* Change "Singleband pseudocolor" to "Hillshade"
* Set the Z Factor to 0.00001, which is roughly the ratio between the horizontal units (degrees) and vertical units (meters)
* Set the blending mode to "multiply" (which will let us also see the colorized version underneath)

* Try turning the dial to adjust the angle of the "sun"

Notice that the highlands in the western regions are fairly distinct, but it more difficult to see the textures near the coast.  We can exaggerate the hillshade effect by increasing the Z Factor:

* Set the Z Factor te 0.0001 (one less zero)

Also notice that if you zoom in too far, you see individual pixel edges.  To avoid this effect:

* Set the resampling "Zoomed in" to "Bilinear" which will smooth out the pixels


## Generating contour lines

There is a "contours" display style for rasters, but is just a visual effect.  If we want to export 3D contour lines for use in a program like Rhino or AutoCAD, we will need to create a vector contour layer.

* Open the processing toolbox (gear icon, or in the Processing menu)
* Search for "contour" and open the "Contour" tool under GDAL > Raster extraction
* Set the following parameters:
  * Input layer = "elevation-1m"
  * Interval between contour lines = 2m (this value should usually not be any less than the pixel size)
  * Check the box to "Produce 3D vector" (if you don't see it, expand the Advanced Parameters section)
* Keep the other options and click "Run"

To export the 3D contours to DXF, right-click the layer > Export > Save features as ... AutoCAD DXF or else export your whole project to DXF.


# Georeferencing historical maps

Georeferencing is the process of adding location information to an image so that it can be displayed in the correct location on a map, with the proper scale and rotation.  The process involves defining a set of control points that can be identified on both the image and a basemap.

Add a satellite basemap and use its CRS:

* Web web > QuickMapServices > Google > Google Satellite
* Right-click the layer name > Layer CRS > Set Project CRS from Layer

Open and configure the Georeferencer:

* Raster menu > Georeferencer
* File > Open Raster...
  * data/mapscans/G 5254 G4 G46 1846 P6 2006-jpeg75.tif
* Settings menu > Transformation settings...
  * Transformation type = Polynomial 1
  * Resampling method = Linear
  * Target SRS = EPSG:3857 (this is the CRS used by most QuickMapServices basemaps)
  * Compression = LZW (for a smaller output file)
  * Save GCP points (in case you want to adjust things later)

Find a spot that you can find on both the scanned map and the Google basemap, and add a ground control point:

* Select the "Add Point" tool
* Click a point on the scanned map
* In the dialog, click "From Map Canvas"
* Click the corresponding point on the Google basemap
* Repeat for several points scattered across the map.

More points generally leads to better georeferencing, but as you go, keep your eye on the "Residual" values in the table (which appear after 3 or more points have been added, depending on the transformation type) and try to keep these values as low as possible.  If one of your points has a residual much higher than the others, then you may want to delete that point and re-do it.  (Right-click the table row > delete)

When you think you have enough good points:
* Click the green triangle button to "Start Georeferencing"

It may take a moment to warp the image to fit the control points and output the file.  When done, it should display in the main QGIS window.
