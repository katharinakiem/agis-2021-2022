---
title: 'Structural diversity metrics from Lidar data '
author: ''
date: '2022-01-07'
slug: []
categories:
  - aGIS
tags:
  - task
subtitle: ''
description: ''
image: ''
toc: yes
output:
  blogdown::html_page:
    keep_md: yes
weight: 100
---




## Introduction

To measure structural diversity of forests can help predict forest ecosystem functions such as microclimate or hydrology functions. Since forests structures are highly complex, dynamic and on multiple scales it is difficult to measure those structures accurately. One approach would be to use Lidar data because it provides threedimensional data of an area of interest.  
Some metrics calculated with Lidar data will be presented down below and the question if it is possible to predict microclimate parameters like temperature or humidity from those metrics will be answered.  

LaRue and Wagner et al (2020) suggest using metrics from the following categories to measure structural diversity of forests: 
- height eg mean canopy height, mean outer canopy height, maximum canopy height
- cover and openness eg deep gaps, cover fraction
- heterogeneity #(external and internal) 
- vegetation area eg vegetation area index

Some indices from each category will be shown examplarily.

## Data and Methods



### Load data 
The used data is a section of the forest of the University of Marburg clos to Caldern. 

```r
lasfile <- list.files(envrmt$path_l_raw,
                  pattern = glob2rx("*.las"),
                  full.names = TRUE)
las <- lidR::readLAS(lasfile[1])
summary(las)
```

```
## class        : LAS (v1.3 format 1)
## memory       : 2 Gb 
## extent       : 477500, 478217.5, 5631730, 5632500 (xmin, xmax, ymin, ymax)
## coord. ref.  : NA 
## area         : 0.55 kunits²
## points       : 26.35 million points
## density      : 47.7 points/units²
## File signature:           LASF 
## File source ID:           0 
## Global encoding:
##  - GPS Time Type: GPS Week Time 
##  - Synthetic Return Numbers: no 
##  - Well Know Text: CRS is GeoTIFF 
##  - Aggregate Model: false 
## Project ID - GUID:        00000000-0000-0000-0000-000000000000 
## Version:                  1.3
## System identifier:         
## Generating software:      rlas R package 
## File creation d/y:        0/2018
## header size:              235 
## Offset to point data:     235 
## Num. var. length record:  0 
## Point data format:        1 
## Point data record length: 28 
## Num. of point records:    26353974 
## Num. of points by return: 10055794 7269852 5128257 2678650 937508 
## Scale factor X Y Z:       0.001 0.001 0.001 
## Offset X Y Z:             4e+05 5e+06 0 
## min X Y Z:                477500 5631730 232.798 
## max X Y Z:                478217.5 5632500 371.824 
## Variable Length Records (VLR):  void
## Extended Variable Length Records (EVLR):  void
```

```r
plot(las, bg = "green", color = "Z",backend="rgl")
plot(las, map= TRUE)
```

### Indices 

1. Height:  
Height metrics contain information about vertical patterns of the forest vegetation.  

To get the height metrics a Canopy height model is calculated at first. 

```r
# Calculate canopy height model with grid_canopy function of lidR package 
# Points-to-raster algorithm with a resolution of 1 meter
# source: (https://rdrr.io/cran/lidR/man/grid_canopy.html)

# Load already nornmalized data and calculate chm 
mof100_ctg_chm <-readRDS(file.path(envrmt$path_level1, "mof100_ctg_chm.rds"))

chm = grid_canopy(mof100_ctg_chm, res = 1, dsmtin())
```

```
## Warning in getProjectionRef(x, OVERRIDE_PROJ_DATUM_WITH_TOWGS84 = OVERRIDE_PROJ_DATUM_WITH_TOWGS84, : PROJ/GDAL PROJ string degradation in workflow
##  repeated warnings suppressed
##  Discarded datum European_Terrestrial_Reference_System_1989 in Proj4 definition: +proj=utm +zone=32 +ellps=GRS80 +towgs84=0,0,0,0,0,0,0 +units=m +no_defs
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-3-1.png" width="672" />

```
## Chunk 1 of 12 (8.3%): state <U+26A0>
## Chunk 2 of 12 (16.7%): state <U+2713>
## Chunk 3 of 12 (25%): state <U+2713>
## Chunk 4 of 12 (33.3%): state <U+2713>
## Chunk 5 of 12 (41.7%): state <U+2713>
## Chunk 6 of 12 (50%): state <U+2713>
## Chunk 7 of 12 (58.3%): state <U+2713>
## Chunk 8 of 12 (66.7%): state <U+2713>
## Chunk 9 of 12 (75%): state <U+2713>
## Chunk 10 of 12 (83.3%): state <U+2713>
## Chunk 11 of 12 (91.7%): state <U+2713>
## Chunk 12 of 12 (100%): state <U+2713>
```

```r
col <- height.colors(50)
plot(chm, col = col, main = "Mean canopy height within 1 m² cells")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-3-2.png" width="672" />

As one can see in the plot, the mean canopy height in the research area varies from 0 to 40.    
Further height metrics can be obtained from the CHM Rasterlayer which was built using the grid_canopy function:  

Mean outer canopy height (MOCH)

```r
mean.max.canopy.ht <- mean(chm@data@values, na.rm = TRUE) 
```

Maximum canopy height

```r
max.canopy.ht <- max(chm@data@values, na.rm=TRUE)
```

```
## Warning in max(chm@data@values, na.rm = TRUE): kein nicht-fehlendes Argument für
## max; gebe -Inf zurück
```


2. Cover and Openness:  
Cover and Openness metrics "describe openness of the outer surface of the canopy as a two dimensional (x, y) horizontal plane" (Atkins et al 2018).

These metrics can also be derived from the CHM:  

Deep Gaps and Deep Gap Fraction 

```r
#number of cells in raster (also area in m2)
cells <- length(chm@data@values) 
chm.0 <- chm
chm.0[is.na(chm.0)] <- 0 #replace NAs with zeros in CHM
#create variable for the number of deep gaps, 1 m^2 canopy gaps
zeros <- which(chm.0@data@values == 0) 
deepgaps <- length(zeros) #number of deep gaps
#deep gap fraction, the number of deep gaps in the chm relative 
#to total number of chm pixels
deepgap.fraction <- deepgaps/cells 
```


## Results

- table with metrics 

## Discussion
Take your research question and your results and discuss the results with respect to your approach

## References

- LaRue & Wager et al: Compatibility of Aerial and Terrestrial LiDAR for Quantifying Forest Structural Diversity https://doi.org/10.3390/rs12091407
- Atkins et al: Quantifying vegetation and canopy structural complexity from terrestrial LiDAR data using the forestr r package https://doi.org/10.1111/2041-210X.13061
- grid canopy_dunction / lidr package https://rdrr.io/cran/lidR/man/grid_canopy.html
- Code was used from the script structural metrics: https://datacommons.cyverse.org/browse/iplant/home/shared/NEON_workshop/tutorials/r_scripts/structural-diversity-discrete-return.r


