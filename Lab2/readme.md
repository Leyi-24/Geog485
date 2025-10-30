# Lab2 Classification

## Overview
1.  You will download Landsat imagery on EarthExplorer of the Mt St Helens area, before and after the 1980 eruption to answer a brief research question of something that interests you (for example: how long did it take for vegetation recovery after the eruption?). The research question will guide the image dates and number of images you download. It’s been almost 45 years, so using Landsat you should be able and ask a pretty interesting question about this. If visible bands are not available you can download any raw bands that are available. 
2.  You will work image by image, and your task is to classify pixels in the image to match a classification schema of interest (i.e., landcover or forest type).
3.  You will be using your image interpretation skills to generate training data.

At this point you can navigate to EarthExplorer, download your Landsat images and then read on.

# Classification Methods Overview
**Image Classification Basics:** Image classification is the process of categorizing and labeling pixels or groups of pixels in satellite or aerial images based on their spectral values. This process allows us to analyze and interpret complex patterns and information contained within the images, transforming raw data into more meaningful information that can be used for various applications.

**Purpose of image classification in GIS and remote sensing**: Image classification plays a critical role in GIS and remote sensing, as it helps in extracting valuable information from the remotely sensed data. This information can be utilized for various purposes, such as land use and land cover mapping, urban planning, agriculture monitoring, natural resource management, and environmental studies, among others. By categorizing the pixels into different classes, image classification simplifies the data and makes it easier for users to analyze and understand spatial patterns, trends, and relationships.

**modeling and prediction step** in the machine learning (ML) pipeline:
1.  **Data Preprocessing:**
    * Cleaning and correcting raw data (e.g., Radiometric/Atmospheric Correction).
    * Preparing the study area (e.g., Clipping the image).
    * **Feature Extraction** (e.g., calculating spectral indices like **NDVI**).

2.  **Training Data Generation:**
    * (For Supervised ML) The analyst collects and labels representative **training data** (e.g., creating Regions of Interest or ROIs).

3.  **Model Training/Clustering:**
    * **Supervised:** The classification algorithm **learns** the relationship between the input features (pixel values) and the desired output labels (land cover classes) using the training data.
    * **Unsupervised:** The clustering algorithm groups the pixels based purely on spectral similarity.

4.  **Prediction (Classification):**
    * The trained model or the clustering logic is applied to the **entire image**.
    * The goal is to assign a discrete **class label** (e.g., "Forest," "Water") to every pixel or object.

5.  **Model Evaluation:**
    * The output classified map is assessed for **accuracy** (e.g., using a Confusion Matrix and Kappa coefficient) to determine the model's performance and reliability.

DIFFERENT CLASSIFICATION METHODS

1. **Part one:** Unsupervised classification.

* The tool you are going to run in QGIS will basically group pixels that have similar spectral characteristics to classes or clusters **without** any additional information from the user (you).
* **For example:** pixels that have really high NIR values could be grouped as one cluster because maybe these pixels have vegetation in them.

2. **Part two:** Supervised classification.

* The tool will take additional information (training data) and *then* it will look at the spectral characteristics of each pixel and group into classes accordingly. You are going to be the source of this training data.
* Supervised image classification is a method where the user provides a set of labeled training samples for each class of interest. The classifier algorithms use these training samples to learn the characteristics of each class and then apply this knowledge to classify the entire image into the specified categories. This method relies on the user’s expertise and understanding of the study area and typically results in higher accuracy compared to unsupervised classification.
   * **Key components of supervised classification:**
      * **Training samples:** These are representative examples of each class of interest, selected by the user based on their knowledge of the study area. The quality and quantity of training samples directly affect the classification accuracy.
      * **Classifier algorithms:** These are machine learning algorithms that learn from the provided training samples and generalize this knowledge to classify the entire image, including:
         * **Maximum likelihood classifier:** A statistical-based algorithm that assumes each class follows a Gaussian distribution and assigns pixels to the class with the highest likelihood based on their spectral values.
         * **Support vector machines:** A machine learning algorithm that finds the optimal hyperplane to separate different classes by maximizing the margin between them.
         * **Decision trees:** A hierarchical approach that splits the data into subsets based on specific rules or conditions at each node of the tree until the pixel is assigned to a class.   

3. **Deep Learning**(in the comming labs): Uses advanced neural networks (like Convolutional Neural Networks, or CNNs) to classify data. These models automatically learn complex features from the raw data, often leading to very high accuracy.
   
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

* You need to first go to "**Plugins**" - Manage and Install Plugins - to search **SAGA**
* Install "**Processing Saga NextGen Provider**" and its patch "**Dissect and dissolve overlaps(Saga NextGen)**"
  
  
* Search for **K-means clustering for grids** in your **toolbox window**
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

# Part Two: Supervised classification

## Step1: Install Plugin Dzetsaka

Plugins -> Python Console -> At the bottom of your screen type this and press enter:

import pip
pip.main(['install', 'scikit-learn'])

## Step2: Train a model

### Create a Point Geometry

* **Option 1**: You can search the toolbox for a tool called random points in extent
   * 10 points minimum per class (so 50 points if you are going for 5 classes)
   * input extent: calculate from layer and use the shapefile from step 2 in the unsupervised classification section.
   * You can set a mimimum distance in between points. This is up to you.
   * Right-click the points layer and select attribute table
   * You will need a value column and you will need to adjust the value of each point in your table depending on where it lies within a true or false color image. For this you can reference your **key**! You should also add a class column that will be the label (i.e., forest, grass, urban).

* **Option 2**: You can create a new points shapefile and manually place your points

   * Layer -> Create Layer -> New Shapefile layer -> recall the way you created the clip frame for the unsupervised classification section except geometry type will be points
   * <img width="602" height="665" alt="1" src="https://github.com/user-attachments/assets/1431a65c-8707-4c91-be8e-669b8ea6f365" />

   * **OK**
   * Change the points symbology so it is a bright color and then start adding points
   * Now **toggle editing** and add points using your true or false color as a reference
   * Each point you add, you have the option to set the value or id (for this you will refer to your key and assess what you see visually in true or false color). Essentially this is just a number you assign it 1-5 based on what class it falls in when you look at an image.
   * Right-click the layer after your save your changes and turn off editing and select attribute table to see how this looks. Feel free to add more columns to match your key. You should also add a class column that will be the label (i.e., forest, grass, urban).
   * <img width="602" height="543" alt="2" src="https://github.com/user-attachments/assets/cc39c9de-ffcd-4fae-b944-19e8e1526ffa" />

* **Considerations**:
   * The first option is random so you might get some points that fall within vegetation pixels and way more that fall within water pixels so there is a bias.
   * The second option is tedious but you have full control over placement. Shoot for at least 10 points per class, so 50 points in total if you are trying to classify the image into 5 distinct classes.
   * ![train_pts_overlay](https://github.com/user-attachments/assets/77bd4dbc-f3ee-40fc-9314-e38690fca9ec)

### Train algorithm
   * dzetsaka -> Classification tool -> Train algorithm.
   * ![dzetsaka_train_params](https://github.com/user-attachments/assets/cd97a8fd-3b2c-48c3-9ea0-075a848d0de2)

## Step3: Predictions from a trained model

   * Once you have saved your models for each band you are ready to make predictions (I saved three models for bands 6, 5, and 4 from a Landsat 1 image)
   * Open the processing toolbox –> dzetsaka
   * Classification tool
   * Predict model
   * Input raster: Landsat B4 clipped to start
   * Model learned is my B4 model from the above step
   * Output raster is saved in output data folder as a tif
   * Confidence raster is saved in output data folder as a tif with a unique name
   * RUN
   * Change layer symbology to match your key
   * ![model_prediction_params](https://github.com/user-attachments/assets/2c1dcb90-c566-4e45-abd4-4c9dd5022c3c)


# Deliverables for both Part 1 and Part 2
You need to upload a PDF answering below questions:
   * What is your research question?
   * Did the unsupervised or supervised classification perform better and what makes you say so?
   * What is the K-Means clustering algorithm doing? Check out google or the QGIS documentation for the tool
   * What is the K nearest neighbors algorithm doing?
   * What kinds of biases could be introduced in the training process?
   * How was your training data distributed spatially? Did you favor a particular area of the image?

Insert screenshots of the unsupervised result, and your supervised results with brief figure captions.
