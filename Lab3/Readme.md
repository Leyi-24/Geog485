This lab is adapted from the Mapping with Drones class and was originally written by James Lamping and Colin Mast. Here we use a dataset from NOAA and ask slightly different questions about the data.

Lets start by setting up our work space. The things we need to do here are to load the libraries we will use in this analysis and also make sure our working directory is set.
```
# This section of code below loads necessary packages into our R session.
install.packages("pacman") #when you run code again on the same machine you can comment install.packages lines out
install.packages("sf")
install.packages("stars")

# Install Rtools on Root folder. Edit system environment variables with ming64/bin path. Restart R
# https://cran.r-project.org/bin/windows/Rtools/rtools44/rtools.html
```
If you use SSIL PC without admin, you download the Rtools44 installer and execute it.
Change the path to ```C:\Users\<you>\rtools44``` instead of the default ```C:\rtools44```
Then in RStudio Console, run:
```
Sys.setenv(PATH = paste(
  "C:/Users/leyi/rtools44/x86_64-w64-mingw32.static.posix/bin",
  "C:/Users/leyi/rtools44/usr/bin",
  Sys.getenv("PATH"),
  sep = ";"
)) #Change to your username!

# Test
Sys.which("gcc")
Sys.which("make")

#You should now see something like:
                               gcc
"C:\\Users\\leyi\\rtools44\\x86_64-w64-mingw32.static.posix\\bin\\gcc.exe"
# Then:
writeLines('
Sys.setenv(PATH = paste("C:/Users/leyi/rtools44/x86_64-w64-mingw32.static.posix/bin;",
                        "C:/Users/leyi/rtools44/usr/bin;",
                        Sys.getenv("PATH"), sep=""))
', file.path(Sys.getenv("R_USER"), ".Rprofile"), useBytes = TRUE)
```
Now quit RStudio and Restart.
You might need these packages if you cannot plot your point cloud
#https://www.xquartz.org/ #install this and unzip if running on macOS

```
#install.packages("rgl")
#install.packages("stars", type = "source") #an alternative attempt at line 4 of this chunk

pacman::p_load("lidR", "tidyverse", "terra", "tidyterra", "future", 'RCSF', 'gstat') 
library(sf)
library(stars)

# Now we need to set the working directory. This is the filepath where we are going to be working from.
setwd("R:/Geog485_585_12268_FALL2025/Student_Data/Lab3/") # Here you need to first copy the data folder from Class_data to your Student_data folder, then change the PATH.

```

First, lets bring in some lidar data and take a look at it.
```
#Here is the source for the data used in this lab
#https://portal.opentopography.org/lidarDataset?opentopoID=OTLAS.102010.26910.1

# There are two main file types that you will see with lidar data, LAS and LAZ. The LAZ file type is highly compressed, saving a lot of room for storage and transfer. Most processing software can read LAZ files, however they default to LAS when exporting. Be careful about storage space when dealing with lidar data.
# read in an individual LAZ file
las <- readLAS("ot_000119.laz")
# lets take a peak at what that looks like
plot(las)

# Some lidar data also has RGB data included. If so this is how you would visualize it in color.
#plot(las, color = "RGB") 

las # run this code and it will give you a basic summary of the object
```
## SECTION TURN IN

  * Take a screenshot of your point cloud
  * What is the difference between LAS and LAZ?
  * What coordinate reference system is your data in?
  * What is your point density?
  * How much area does your data cover?

* We are going to chunk our data so we can work with smaller files for subsequent steps.
```
# bring in the large lidar file as a LAS Catalog
ctg <- readLAScatalog("ot_000119.laz")
plot(ctg, chunk = TRUE)

# Create a new set of 200 x 200 m. laz files with a 10 meter buffer.
opt_chunk_buffer(ctg) <- 10
opt_chunk_size(ctg) <- 200
plot(ctg, chunk = TRUE)
opt_laz_compression(ctg) <- TRUE # this tells it to be a .laz file instead of a .las file

#set an output folder path (as is, this will write to wherever you setwd)
opt_output_files(ctg) <- "retile_{XLEFT}_{YBOTTOM}" # this sets the folder location and name with the coordinates of the chunk.

# preview the chunk pattern and create new tiles
plot(ctg, chunk = TRUE)
plan(multisession, workers = availableCores() - 1) # create a parallel session. This lets you process more than one at a time
newctg = catalog_retile(ctg)
plot(newctg)
```
# Point classification

One of the most important steps in processing lidar is classifying points to different categories. A lot of freely available aerial lidar already comes classified as part of a completed validated dataset. However, if you are the one producing the lidar from a drone you will need to classify the point cloud yourself. There are many standards when it comes to lidar data and most are controlled by ASPRS. To read more about the classification standards of ASPERS visit the link below. Table 3 contains the standard classification values and meanings set by ASPERS. We will just be focusing on classifying ground points in this lab.

https://www.asprs.org/wp-content/uploads/2010/12/LAS_Specification.pdf
```
# lets plot our data and see if these points are already classified.
plot(las, color = "Classification")

# It likely is, and in this case you will see white points for unclass (0) and blue points for points classified as ground (2). Lets bring the point cloud back in as if there are no ground points classified with the filter funtions in lidR.

#read in one of the tiles that you created
las <- readLAS("retile_722200_4940600.laz", filter = "-change_classification_from_to 2 0")
plot(las, color = "Classification")

# Now we are going to use the ground classification algorithm to classify points as either ground or not. There are a few different methods for this. For details on other methods please look here: https://r-lidar.github.io/lidRbook/gnd.html
las_class <- classify_ground(las, algorithm = pmf(ws = 5, th = 3))

# How does it look? Zoom in move around. Does it look like it performed well? If not, maybe try another classification method from the link above.
plot(las_class, color = "Classification", size = 3, bg = "white")

# Another way to see how well this worked is by plotting a cross section.
plot_crossection <- function(las,
                             p1 = c(min(las@data$X), mean(las@data$Y)),
                             p2 = c(max(las@data$X), mean(las@data$Y)),
                             width = 4, colour_by = NULL)
{
  colour_by <- enquo(colour_by)
  data_clip <- clip_transect(las, p1, p2, width)
  p <- ggplot(data_clip@data, aes(X,Z)) + geom_point(size = .5) + coord_equal() + theme_minimal()
  
  if (!is.null(colour_by))
    p <- p + aes(color = !!colour_by) + labs(color = "")
  
  return(p)
}

plot_crossection(las_class, colour_by = factor(Classification))
```
## SECTION TURN IN

  * Take a screen shot of the cross section you created with the pmf function.
  * How well did it perform?
  * Did you have to use a different classification? If so, what did you end up using?
  * What do you see in your classified point cloud?

## Creating a digital surface model (DSM)

Lets start off by making a DSM. If you remember from lecture a DSM includes all ground and vegetaion points. Its like draping a sheet across the surface of everything that is there.

```
dsm <- rasterize_canopy(las_class, res = 1, p2r(na.fill = tin()))
col <- height.colors(25) # create a color profile
plot(dsm, col = col, main = "Digital Surface Model")

#summary stats
summary(dsm) 
hist(dsm)#repeat this for dtm and chm
```
## Creating a digital terrain model (DTM)

One of the neatest things about lidar data is its ability to model what lies beneath the canopy. Where are old logging roads? Are there artifacts beneath the vegetation? What is the actual elevation of the land? Lets only look at the ground points and make another surface model of the terrain (DTM).

#this one could take a while
dtm <- rasterize_terrain(las_class, algorithm = kriging(k = 40))
plot(dtm, main = "Digital Terrain Model") 
```
# that doesn't look like much... let's make a hillsahde
dtm_prod <- terrain(dtm, v = c("slope", "aspect"), unit = "radians")
dtm_hillshade <- shade(slope = dtm_prod$slope, aspect = dtm_prod$aspect)
plot(dtm_hillshade, col =gray(0:30/30), legend = FALSE, main = "DTM Hillshade")
```
## Creating a canopy height model (CHM)

To look at the height of only the vegetation we need to subtract the height of the ground out of our data. Looking at the DSM you see individual trees, but the height data is a combination of both the ground height and the vegetation height. Lets get rid of the ground and look at the trees!
```
las_norm <- normalize_height(las_class, algorithm = tin())
plot(las_norm, color = "Classification") # notice how the terrain is now completely flat

# now that we have no more terrain elevation. Lets make another surface model. This will be our CHM.
chm <- rasterize_canopy(las_norm, res = 1, pitfree(thresholds = c(0, 10, 20), max_edge = c(0, 1.5)))
plot(chm, col = col, main = "Canopy Height Model")#what are the units
```
## SECTION TURN IN

  * Screenshots of all terrain models and histograms.
  * Report the average values of each model. You can copy summary(dsm) for dtm and chm models.

# Individual tree detection

Now that we have the lidar processed and our terrain models generated, lets do something a bit more advanced. From the terrain data and the unclassified vegetation data there are methods of detecting and modeling every tree on the landscape. This can be useful in so many different ways! It can tell us the density of trees on the landscape, the height distrubution, and even estimate how much biomass there is in the living trees. For this lab we will locate each tree and create spatial data of the canopy area of each tree.

There are many different ways of going about this. For this I chose the fastest, but it might not be the most accurate for this environment. Feel free to try other methods as described here: https://r-lidar.github.io/lidRbook/itd-its.html
```
# locate all the tops of the trees. This does this by looking at a local area maximum height within a set radius.
ttops <- locate_trees(las_norm, lmf(ws = 5))

# plot the tree tops on top of the CHM
plot(chm, col = height.colors(50))
plot(sf::st_geometry(ttops), add = TRUE, pch = 3)

#Lets see what that looks like on top of the normalized point cloud.
x <- plot(las_norm, bg = "white", size = 4)
add_treetops3d(x, ttops)
## Do you think it is oversegmenting trees (as in there are more than one "tree top" per tree)? If so, change the window size (ws) to a larger one and see how that performs.

# Now lets segment the point cloud. To do this we will use the CHM and the tree tops to identify canopies. Then we will segment the point cloud to have individual trees identified.
las_seg <- segment_trees(las_norm, dalponte2016(chm, ttops)) # segment point cloud
plot(las_seg, bg = "white", size = 4, color = "treeID") # visualize trees

# you can now look at individual trees!
tree110 <- filter_poi(las_seg, treeID == 110)
plot(tree110, size = 8, bg = "white")

# from the segmented lidar you can now easily make spatial data that has information about each tree. This is useful in arcpro or QGIS and is the product you will need to make basic figures and maps about the status of those trees.
crowns <- crown_metrics(las_seg, func = .stdmetrics, geom = "convex")
plot(crowns["zq95"], main = "95th percentile of tree height (feet)")
names(crowns) # look at all the data you now have of each tree
```
## Exporting data

All of this can be easily exported for use in other geospatial programs. Lets do that now!
```
# rasters
writeRaster(dsm, "data/output/lidar_dsm.tif", overwrite = TRUE)
writeRaster(dtm, "data/output/lidar_dtm.tif", overwrite = TRUE)
writeRaster(chm, "data/output/lidar_chm.tif", overwrite = TRUE)
writeRaster(dtm_hillshade, "data/output/lidar_hillshade.tif", overwrite = TRUE)

# vectors
writeVector(vect(crowns), "data/output/Crowns/lidar_tree_crowns.shp")
```
## SECTION TURN IN

 * Add your output rasters to a QGIS environment and change symbology. You can add these screenshots to your write up if you want.
 * Additional processing with lasCatalog. No submission required here but worth knowing how to process larger datasets

At the very beginning of this exercise you created a catalog from the large lidar dataset to create smaller chunks that are much more manageable to analyze. In the same way your created these chunks you can analyze these as a whole unit to create complete output products for a whole area. There are many functions that work with the lasCatalog engine in lidR, including terrain modeling, tree segmentation, and classification. Below we are going to just create a DTM on the whole dataset. Remember that the dataset we used for this lab already had ground points classified, so lets just go ahead and run the analysis.

ctg_dtm <- rasterize_terrain(ctg, algorithm = tin()) # calling a catalog into a function runs that function through the lascatalog engine
plot(ctg_dtm, main = "Digital Terrain Model") 
```
# that doesnt look like much... lets make a hillsahde
ctg_dtm_prod <- terrain(ctg_dtm, v = c("slope", "aspect"), unit = "radians")
ctg_dtm_hillshade <- shade(slope = ctg_dtm_prod$slope, aspect = ctg_dtm_prod$aspect)
plot(ctg_dtm_hillshade, col =gray(0:30/30), legend = FALSE, main = "DTM Hillshade")

# export DTM and hillshade
writeRaster(ctg_dtm, "data/output/ctg_DTM.tif", overwrite = TRUE)
writeRaster(ctg_dtm_hillshade, "data/output/ctg_Hillshade.tif", overwrite = TRUE)
```
