# Ecochange Tutorial

This repository contains my work with the **ecochange** R package, developed as part of my internship project.  
The goal is to explore ecosystem change analysis by following the official ecochange tutorial and documenting the process in R Markdown.

## Contents
- `ecochange_tutorial.Rmd` → the main R Markdown file with code, explanations, and analysis  
- Additional data or figures generated during the tutorial  

## What I did
- Installed and explored the **ecochange** package  
- Downloaded and processed environmental datasets (e.g., Global Forest Change, Global Surface Water, MODIS vegetation indices)  
- Used functions such as:
  - `getGADM()` → to obtain administrative units  
  - `listGP()` → to list available environmental datasets  
  - `getrsp()` → to download specific layers (e.g., water occurrence)  
  - `echanges()` → to represent ecosystem changes  
  - `sampleIndicator()` → to compute biodiversity indicators on grid cells  
- Produced plots and visual outputs included in the R Markdown/HTML report  

## Purpose
The repository serves as a record of my progress in learning and applying the **ecochange** package.  
It will also be the starting point for further analysis during my internship.  

## How to use
1. Clone or download the repository  
2. Open the `.Rmd` file in RStudio  
3. Run the code chunks step by step, or knit the file to reproduce the results  

---

📌 Internship started: **September 8, 2025**  

## Step 1 – Load Packages

```r
library(ecochange)
library(raster)
library(viridis)
```

## Step 2 - Download Administrative Boundaries
The getGADM() function provides the names of administrative units. By default, it shows level 2 administrative units in the country of Colombia but those settings can be changed through the parameters level and country
```{r}
getGADM()
```

## Step 3 – List Available Environmental Products

List the products that can be downloaded with ecochange
It shows you all the environmental datasets (ERSP) that ecochange can download, for example: Global Forest Change (Hansen et al.), Global Surface Water (Pekel et al.), and MODIS vegetation indices.
```{r}
listGP()
```

## Step 4 – Download a Specific Dataset
Download a layer containing water occurrence for the municipality of Chimichagua

```{r}
waterocc=getrsp("Chimichagua", lyrs=c("occurrence"))
waterocc

treecover=getrsp("SantoDomingo", lyrs = c("treecover2000"))
treecover
```

## Step 5 - Plot the object
The object is downloaded by default to a temporary folder.
Users can define a different path using the argument path()

```{r}
treecover <- plot(raster(treecover), axes = T,
                  main = ' Tree Cover (Santo Domingo)')
dev.off()

wateroc <- plot(raster(waterocc), axes = T,
                main = 'Occurrence (Colombia)')
dev.off()

#In this step, I use the `viridis` package to apply a color palette to the water occurrence raster.  
This improves visualization and highlights the intensity of water presence in the selected area

waterviridis <- plot(raster(waterocc), col = viridis::viridis(100), 
                     main = 'Water occurrence (Chimichagua)')
dev.off()
```

<img width="1161" height="628" alt="treecoversantodomingo1" src="https://github.com/user-attachments/assets/166328fb-e735-450e-b030-242e5821abb6" />

<img width="1167" height="683" alt="occuranceColombia" src="https://github.com/user-attachments/assets/8eea47e9-5121-422f-900d-f62cf319c7c8" />


<img width="1200" height="800" alt="water_occurrence_chimichagua" src="https://github.com/user-attachments/assets/ba16b242-11e1-45ad-afbf-a3bfe13b1aec" />


## Step 6 - Downloading, resizing, reprojecting and integrating datasets

The function rsp2ebv() automates the downloading, reprojection and cropping of the dataset of interest to a given polygon. It can inherit arguments in the previous getrsp(). 
The function will detect whether the file was already downloaded to avoid download it again.

```{r}
wo=rsp2ebv('Chimichagua', lyrs = c('occurrence'), mc.cores = detectCores())

plot(wo, main = 'Occurrence (Municipality of Chimichagua)')

dev.off()

# Check that the projection is different to the file that was originally downloaded
crs(raster(waterocc[1]))
crs(wo[[1]])

#The rsp2ebv() can also be implemented to download and integrate several datasets simultaneously.

ebv <- rsp2ebv('Chimichagua', lyrs = c('treecover2000','lossyear'), mc.cores = detectCores())
plot(ebv, main = 'Forest cover and loss (Chimichagua)')

dev.off()
```
<img width="1156" height="683" alt="occuranceMunicipalityChimichagua" src="https://github.com/user-attachments/assets/ce46cb65-c35a-4dde-aed7-accfd7fb236d" />
<img width="1156" height="683" alt="forestcoverandloss" src="https://github.com/user-attachments/assets/c6aa636b-cb5f-4845-bcc3-623f388f9a1f" />


## Step 7 - Deriving ecological changes
The echanges() function generates RasterStack representations of ecosystem change by masking values from a target ecosystem variable with corresponding change data.
It is designed to work with datasets previously prepared using rsp2ebv(), from which it can also inherit arguments and intermediate data structures. 
By default, the function interprets the first raster layer as the target variable and the last one as the change raster. If this ordering does not apply, the user must specify the correct layers through the eco and change arguments, which accept regular expressions matching layer names (for example, “tree” for treecover2000 and “loss” for lossyear). 
Additional arguments, such as eco_range and change_vals, control which pixel values are processed in both the target and change rasters. 
The result of echanges() is a RasterStack whose number of layers depends on the values given in change_vals; for instance, specifying three years of deforestation produces three raster layers representing those events.
Let’s calculate forest cover area for the years 2005, 2010, and 2020, assuming a definition of forest as those areas with canopy cover between 95 and 100%

```{r}
forExt <- echanges(ebv, eco = 'tree', eco_range=c(95,100), echanges = 'loss', change_vals = c(5,10,20), binary_output=TRUE, mc.cores = detectCores())
plot(forExt, main = 'Changes in forest cover')
dev.off()

It is possible to produce instead, a map with cumulative deforested areas by setting the argument get_unaffected=FALSE.
defExt <- echanges(ebv, eco = 'tree', eco_range=c(95,100), echanges = 'loss', change_vals = c(5,10,20), binary_output=TRUE, mc.cores = detectCores(),
                   get_unaffected=FALSE)
plot(defExt, main = 'Changes in forest loss')
dev.off()

The function also allows to calculate, for instance, the distribution of tree cover among deforested pixels by changing the argument binary_output = FALSE.

defExtTC <- echanges(ebv, eco = 'tree', eco_range=c(95,100), echanges = 'loss', change_vals = c(5,10,20), binary_output=FALSE, mc.cores = detectCores(), get_unaffected=FALSE)
plot(defExtTC, main = 'Changes in deforested pixels')

dev.off()
```
<img width="1156" height="683" alt="forestcoverandloss" src="https://github.com/user-attachments/assets/7c435c00-1c39-451e-a402-ed92ea1e8a75" />
<img width="1156" height="683" alt="change in forest cover" src="https://github.com/user-attachments/assets/ae3ffe4a-dd71-4c4f-a3ad-d8adf27f17a8" />
<img width="1156" height="683" alt="changesindeforestedpixels" src="https://github.com/user-attachments/assets/deae0b9a-8598-4674-a510-8f17d2ef7f13" />


