## Convertir un archivo netcdf a geojson

Las librerías previamente instaladas deben ser:

``` r
library("terra")
library("geojsonio")
```

Abrimos un archivo netcdf (en este caso será
“WRFDETAR_01H_20240101_12_000.nc”) y será transformado a un polígono y
escrito a un archivo shp:

``` r
nc <- rast("WRFDETAR_01H_20240101_12_000.nc")

temp <- as.polygons(nc[[2]])

writeVector(temp, "temp.shp", overwrite = TRUE)
```

Finalmente, se utiliza la función file_to_geojson para crear el archivo
geojson:

``` r
s <- file_to_geojson("temp.shp", method = "local", output = ".")
```

    ## Success! File is at file24e546b51aa63.geojson
