## Código

Como convertir un netcdf a un geojson

``` r
library("terra")
```

    ## terra 1.7.65

``` r
library("geojsonio")
```

    ## Registered S3 method overwritten by 'geojsonsf':
    ##   method        from   
    ##   print.geojson geojson

    ## 
    ## Attaching package: 'geojsonio'

    ## The following object is masked from 'package:base':
    ## 
    ##     pretty

``` r
nc <- rast("WRFDETAR_01H_20240101_12_000.nc")

temp <- as.polygons(nc[[2]])

writeVector(temp, "temp.shp", overwrite = TRUE)
```

    ## Warning in x@cpp$write(filename, layer, filetype, insert[1], overwrite[1], :
    ## GDAL Message 6: dataset temp.shp does not support layer creation option
    ## OVERWRITE

``` r
s <- file_to_geojson("temp.shp", method = "local", output = ".")
```

    ## Success! File is at file24ccd5bb2f9c1.geojson
