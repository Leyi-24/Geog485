# Lab2 Classification

## Overview
1.  You will download Landsat imagery on EarthExplorer of the Mt St Helens area, before and after the 1980 eruption to answer a brief research question of something that interests you (for example: how long did it take for vegetation recovery after the eruption?). The research question will guide the image dates and number of images you download. It’s been almost 45 years, so using Landsat you should be able and ask a pretty interesting question about this. If visible bands are not available you can download any raw bands that are available. 
2.  You will work image by image, and your task is to classify pixels in the image to match a classification schema of interest (i.e., landcover or forest type).
3.  You will be using your image interpretation skills to generate training data.

At this point you can navigate to EarthExplorer, download your Landsat images and then read on.
# Classification Methods Overview

**Part one:** Unsupervised classification.

* The tool you are going to run in QGIS will basically group pixels that have similar spectral characteristics to classes or clusters **without** any additional information from the user (you).
* **For example:** pixels that have really high NIR values could be grouped as one cluster because maybe these pixels have vegetation in them.

**Part two:** Supervised classification.

* The tool will take additional information (training data) and *then* it will look at the spectral characteristics of each pixel and group into classes accordingly. You are going to be the source of this training data (more on this below).

# Part One: Unsupervised classification
First, consider: what is it you would like to investigate about the landscape? How will you set up your classification schema? Create a classification key with class, value, and color as columns for reference. You have to freedom over what this looks like and it is purely for reference. Use excel, notepad, or word create one. Here is mine:
| Class | Value | Color |
| :--- | :--- | :--- |
| Healthy Forest | 1 | Green |
| Urban | 2 | Gray |
| Water | 3 | Blue |
| Grass | 4 | Yellow |
| Unhealthy Forest | 5 | Brown |

## Unsupervised Pixel Based Classification

### 1. Prepare the Data:

* Load layers into QGIS. Just use one image date to start
* Create a true color composite or false color composite for reference using the Build Virtual Raster Tool.
    * Raster tab (top of the screen)
    * Miscellaneous
    * Build Virtual Raster
    * Select input layers
    * Set Resolution to highest
    * Place each input file in separate band
    * RUN
* Remember that the virtual output you created does not automatically save.
    * Right-click the virtual layer
    * Export
    * Save in the output data as a tif
    * Be specific with how you name (i.e., `FCC_1979.tif`)

### 2. Define Area of Interest:

* Create a polygon shapefile around Mt. St. Helens to use as a clipping mask.
    * Layer tab (top of the screen)
    * Create layer
    * New shapefile layer
    * Filename is up to you
    * Geometry type: polygon
    * RUN
    * Right-click the shapefile in your layers tab
    * Select “Toggle editing”
    * Add polygon (see screenshot) then Click somewhere on the screen to begin drawing and complete the polygon by right-clicking
    * Right-click and select toggle editing again to turn it off
![1](https://github.com/user-attachments/assets/bc68312f-e2a6-4cbf-b451-533e28c044e1)

### 3. Clip Raster by Mask:

* Clip your images to the shapefile. The shapefile will act as a cookie cutter.
    * Search the processing toolbox for "Clip raster by mask layer". If you don't see the toolbox on the right side of QGIS press `CTRL + ALT + T`.
    * **Input layer:** Landsat raw band raster (you will have to do this for each raw band raster instead of composites because the classification tool will not work for a composite image).
    * **Mask layer:** The shapefile you created
    * **Clipped mask:** Save this in your output data as a TIF (i.e., `Landsat_1979_B5_CLIPPED.tif`)
    * RUN
* Rinse and repeat for the other bands

### 4. K-Means Clustering:

* Search for **K-means clustering for grids** in your toolbox window
* Input the clipped rasters from above. Set the **Number of Clusters** based on your classification key (for example, 5 clusters for 5 land cover classes).
* Adjust the **Iterations** to control how many times the algorithm tries to classify pixels (usually around 10-20 iterations). More on the [tool](https://saga-gis.sourceforge.io/saga_tool_doc/2.2.4/imagery_classification_1.html).
* Run the tool. See the parameters I used:
![2](https://github.com/user-attachments/assets/be41aabf-4951-4180-93d7-8de6d02c4d3f)

### 5. Classify the Output:
* After running the tool, the output will be a classified raster.
![3](https://github.com/user-attachments/assets/8aa7c9de-af8f-4672-b794-d6f15ac5f12e)
* Do a quick export of the clusters layer in your layers pane and save it as a tif. It won’t save if you reload QGIS because it is just a temporary file at the moment.

* Once you save it, load it into your layers and change the symbology to apply a random color palette or directly adjust the colors according to your KEY!

  * Right-click the output clusters layer
  * Properties
  * Symbology
  * Change the render type to paletted/ unique values
  * Press classify
  * Press apply
  * Change the colors to match your key (read below)
*Now, use a true color or false color composite as a reference (toggle your layer on and off) to visually inspect the output and match it with the classification key. Take a screenshot of your classified output after setting the colors as close to your key as possible. Do this to the best of your ability.

See if your output looks any better than this but remember this was unsupervised:
![4](https://github.com/user-attachments/assets/97c7d8aa-cc8f-4073-9e9b-1a8480decd0c)

