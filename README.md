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
the object is downloaded by default to a temporary folder.
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









