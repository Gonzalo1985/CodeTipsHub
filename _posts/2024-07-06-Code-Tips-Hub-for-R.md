---
title: Code Tips Hub for R
layout: post
subtitle: Gonzalo M. Díaz
leafletmap: true
always_allow_html: yes
last_modified_at: 2024-07-06
output: 
  md_document:
    variant: gfm
    preserve_yaml: true
---

This is a post for the management of netcdf data and its transformation to geojson, among other processing:


## Convert a netcdf file to geojson

The previously installed libraries must be:

``` r
library("terra")
library("geojsonio")
```

Open several netcdf files and transform it to lat/lon projection and
transform it to polygon. The files are located in the data folder:

``` r
nc <- rast(c("./data/WRFDETAR_01H_20240101_12_000.nc",
             "./data/WRFDETAR_01H_20240101_12_001.nc",
             "./data/WRFDETAR_01H_20240101_12_002.nc",
             "./data/WRFDETAR_01H_20240101_12_003.nc"))

T2.positions <- which(names(nc) == "T2")

T2.nc <- nc[[T2.positions]]

T2.nc.repro <- project(T2.nc, "+proj=longlat +datum=WGS84")

T2.polygons <- lapply(T2.nc, as.polygons)
```

Finally, a shp file is written and the file_to_geojson function is used
to create the geojson file for each time:

``` r
for (i in 1:4){
  writeVector(T2.polygons[[i]], paste0("temps_", i, ".shp"), overwrite = TRUE)

  s <- file_to_geojson(paste0("temps_", i, ".shp"), method = "local",
                       output = paste0("./geojson/temps_", i))
}
```

    ## Success! File is at ./geojson/temps_1.geojson

    ## Success! File is at ./geojson/temps_2.geojson

    ## Success! File is at ./geojson/temps_3.geojson

    ## Success! File is at ./geojson/temps_4.geojson

## View geojson in Leaflet map

The previously installed libraries must be:

``` r
library("sf")
library("leaflet")
```

First the geojson to be graphed is opened:

``` r
T2.geojson <- read_sf("./geojson/temps_1.geojson") 
```

Finally, the geojson variable is graphed, in this case temperature:

``` r
pal <- colorNumeric("viridis", NULL)

mapa <- leaflet(T2.geojson) %>%
  addTiles() %>%
  addPolygons(stroke = FALSE, smoothFactor = 0.8, fillOpacity = 0.7,
              fillColor = ~pal(T2),
              label = ~paste0(T2, ": ", formatC(T2, big.mark = ","))) %>%
  addLegend(pal = pal, values = ~T2, opacity = 1.0)
```

<iframe src="/assets/mapa.html" width="600" height="400">
</iframe>

## Ploting MCD12Q1 MODIS Product

The data is available in data folder. First, open the data with terra
package

``` r
list.pathname <- list.files("./data", pattern = "hdf", full.names = TRUE)

data <- lapply(list.pathname, FUN = rast)
```

Now, creation of mosaic with each hdf.

``` r
data.mosaic <- mosaic(data[[1]], data[[2]], data[[3]], data[[4]], data[[5]], data[[6]], data[[7]], data[[8]], data[[9]])
```

    ## |---------|---------|---------|---------|=========================================                                          

Lastly, ploting of data with no change of projection, the numbers
represent soil type

``` r
plot(data.mosaic$LC_Type1)
```

![](Page01_R_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

If the projection should be in longlat run:

``` r
data.mosaic.repro <- project(data.mosaic$LC_Type1, "+proj=longlat +datum=WGS84", method = "bilinear", progress = FALSE)
```

And plot again:

``` r
plot(data.mosaic.repro)
```

![](Page01_R_files/figure-gfm/unnamed-chunk-11-1.png)<!-- -->
