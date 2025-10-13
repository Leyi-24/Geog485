# Lab one instructions: An Image is Worth at Least a Thousand Words

### Summary

A useful tip when working with remote sensing data is to consider beginning your workflow by visualizing your data in **true color**. This can serve as reference prior to processing so that you can toggle the changes you make on and off and make semi-informed interpretations, at least at the beginning.

There is SO much remote sensing data that is freely available but the go-to data source is often **Landsat** because of its historical record. It is also relatively easy to download Landsat imagery with different levels of processing (for example, surface reflectance vs top-of-atmosphere reflectance products vs analysis ready data), which can take a lot of steps out of the scientific process for us. Nonetheless the surface reflectance product from Landsat has been used in many studies, here is a good [review paper](https://www.sciencedirect.com/science/article/pii/S0034425722003054).

As for data, I’ll provide USGS’s [EarthExplorer](https://earthexplorer.usgs.gov/) for your reference, which is a simple, web-based archive of satellite and aerial imagery and indices from various sources. **You will need to create an account to download data.** You are not limited to EarthExplorer; there are other sources like The National Map, ArcGIS online, and Google Earth Engine (GEE). We will introduce GEE in the next lab when we need more data.

Inside EarthExplorer, you can filter your data search spatially, temporally, and by cloud cover. Try to get logged on, drop a pin on or near Mount St Helens and define a date range and cloud cover percentage (lower cloud cover is great).

> [Check out this demo] for steps to download and visualize!

![True Color Composite of Landsat 8](https://github.com/Leyi-24/Geog485/blob/main/Lab1/images/tcc-ss.jpeg)

*Figure 1: True Color Composite of Landsat 8 (RED, GREEN, BLUE)*

A review of [Landsat 8 Bands and Band Combinations](https://gisgeography.com/landsat-8-bands-combinations/)

---

### Software and tools

You will want to get familiar with the GIS software and your file structure if you are not already. I typically start a file structure like this:
rs_485/
─ lab_one/

├── input_data/ <– raw images, shapefiles

├── output_data/ <– processed imagery, CSVs

├── scripts/ <– useful in future labs

├── venv/ <– useful in future labs

└── writing/

*Note that data in future labs may be uploaded to the R drive.*

This lab relies on **QGIS** which is an open-source platform that we will use for most of our visualization and image manipulation. However, examples of different approaches techniques using alternative software R, GEE and Python are provided in future labs.

---

## Deliverables

You are required to submit the following:

* A **PDF document** documenting the steps you took to create a true color and/or false image composites from raw satellite imagery.
    * This can be as detailed as you want, but you must obtain imagery from **5 distinct satellite or aerial sources**, display and manipulate each image in QGIS (documenting your methods).
* **Screenshot of each image with a brief description.**
* **Review of advantages and disadvantages** and which datasets might be useful as we continue to study the region.
* A **brief reflection** on the types of imagery you would use for specific studies.

![False Color Composite of Landsat 8](https://github.com/Leyi-24/Geog485/blob/main/Lab1/images/fcc-ss.jpeg)

*Figure 2: False Color Composite of Landsat 8 (SWIR2, NIR, RED)*

A reference of [Sentinel 2 Bands and Band Combinations](https://gisgeography.com/sentinel-2-bands-combinations/) might be useful for your 5 satellite imagery assignment.

---

## Check out this demo

[https://youtu.be/u-FCX4RjsxM](https://youtu.be/u-FCX4RjsxM)

## Lab 1-b Using Satellite Imagery Data on Google Earth Engine

This lab will introduce you data accessing and preparation through Google Earth Engine.

You will:

* **Authenticate and access Google Earth Engine** API.
* Familiarize yourself with **two GEE workflows** using different programming languages (JavaScript in Code Editor and Python in Google Colab).
* **Set up the environment on Google Colab** and **Set up the GEE API** connection.
* Perform **Data access and data preparation** through programming (filtering, compositing).
* Learn some **key indices and their false color combinations**, e.g.:
    * **NDVI** (Normalized Difference Vegetation Index) for vegetation.
    * **NDWI** (Normalized Difference Water Index) for water and moisture.
    * **NDBI** (Normalized Difference Built-up Index) for urban areas and buildings.
 
Launch the Lab: Google Colab Notebook

Click the link below to begin your lab. To save your changes, please go to File → Save a copy in Drive within the Colab notebook.

[**Lab 1-b **](https://colab.research.google.com/drive/1PpSRoedfAyyxjImcGEHd9YJ5S-LXnxgY?usp=sharing)

### Sentinel-2 Spectral Bands Overview

| GEE Band ID | Band Name | Wavelength Center ($\mu m$) | Resolution (meters) | Primary Use |
| :--- | :--- | :---: | :---: | :--- |
| **B1** | Coastal Aerosol | 0.443 | 60 m | Atmospheric correction, coastal studies |
| **B2** | Blue | 0.490 | **10 m** | Water penetration, soil discrimination |
| **B3** | Green | 0.560 | **10 m** | Vegetation, clear water (for **NDWI**) |
| **B4** | Red | 0.665 | **10 m** | Vegetation absorption (for **NDVI**), urban areas |
| **B5** | Red Edge 1 | 0.705 | **20 m** | Vegetation health, Red Edge indices |
| **B6** | Red Edge 2 | 0.740 | **20 m** | Vegetation health, Red Edge indices |
| **B7** | Red Edge 3 | 0.783 | **20 m** | Vegetation health, Red Edge indices |
| **B8** | Near Infrared (NIR) | 0.842 | **10 m** | **Biomass content** (for **NDVI, NDWI**) |
| **B8A** | Narrow NIR | 0.865 | **20 m** | Vegetation monitoring, atmospheric correction |
| **B9** | Water Vapor | 0.945 | 60 m | Atmospheric water vapor detection |
| **B10** | SWIR - Cirrus | 1.375 | 60 m | Cirrus cloud detection |
| **B11** | Short-Wave Infrared (SWIR 1) | 1.610 | **20 m** | **Moisture content** (for **NDBI, NDMI**) |
| **B12** | Short-Wave Infrared (SWIR 2) | 2.190 | **20 m** | Geology, mineral mapping |

[Sentinel 2 Bands and Band Combinations](https://gisgeography.com/sentinel-2-bands-combinations/)

## Remote Sensing Indices and False Color Composites
False Color Composites (FCCs) are fundamental to visual interpretation and analysis in remote sensing. Different band combinations enhance specific land cover characteristics, while Normalized Difference Indices (NDIs) are key for quantifying these features.

Below are several crucial indices and their corresponding False Color Composites for different application purposes.

### Crucial Indices and Their False Color Composites

| Application Purpose | Key Index (NDI) | Normalized Formula | Common Band Composite (R-G-B) | Description / Enhanced Features |
| :--- | :--- | :--- | :--- | :--- |
| **Vegetation Health / Biomass** | **NDVI** (Normalized Difference Vegetation Index) | $\frac{(NIR - RED)}{(NIR + RED)}$ | **NIR - RED - GREEN** (e.g., Sentinel-2 **B8-B4-B3**) | **Healthy vegetation** is highlighted in **bright red**. Used for monitoring growth and density. |
| **Water Bodies / Moisture** | **NDWI** (Normalized Difference Water Index) | $\frac{(GREEN - NIR)}{(GREEN + NIR)}$ | **NIR - SWIR - BLUE** (e.g., Sentinel-2 **B8-B11-B2**) | **Open water bodies** are displayed in deep blue or black. Used for monitoring water extent and moisture. |
| **Urban / Built-up Areas** | **NDBI** (Normalized Difference Built-up Index) | $\frac{(SWIR - NIR)}{(SWIR + NIR)}$ | **SWIR - NIR - BLUE** (e.g., Sentinel-2 **B11-B8-B2**) | **Urban areas and concrete** are highlighted in **magenta/cyan**. Used for urban sprawl analysis. |
| **Soil/Vegetation Background** | **SAVI** (Soil Adjusted Vegetation Index) | $\frac{(NIR - RED)}{(NIR + RED + L)} \cdot (1 + L)$ | Similar to NDVI (NIR-RED-GREEN) | A modified NDVI that minimizes the influence of **soil background** brightness ($L$ is the soil adjustment factor). |

**Hints for lab assignment**: You are supposed to provide interpretations for the values range you set (the min and max parameters in visualization code you used for NDVI, NDWI, etc.) and the land cover (water, snow, clouds, rock, bare ground, or non-vegetated urban areas, bare soil, built-up areas, or senescing (dying) plants, forest ...) 
