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

s <- file_to_geojson("temp.shp", method = "local", output = ".")
```

    ## Success! File is at file24c12647970d4.geojson
