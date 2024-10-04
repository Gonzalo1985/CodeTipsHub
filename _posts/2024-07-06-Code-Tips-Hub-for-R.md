---
title: Transformation from netcdf to geojson
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

This is a post for the management of netcdf data and its transformation to geojson


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

<iframe src="../assets/mapa.html" width="600" height="400">
</iframe>
