---
title: Evaporation soil model calibration
subtitle: Gonzalo M. Díaz
tags: [R,leaflet,Jekyll, html, maps]
leafletmap: true
always_allow_html: yes
last_modified_at: 2024-11-10
output: 
  md_document:
    variant: gfm
    preserve_yaml: true
---

Post for calibration of soil model evaporation output

## Evaporation soil model calibration

First, the aws.wrfsmn library should be open:

``` r
library(aws.wrfsmn)
```

The example data to use will be ‘eva’ and can be visualize:

``` r
head(eva)
```

    ##        Dates evapo_obs OUT_PREC OUT_EVAP OUT_RUNOFF OUT_BASEFLOW
    ## 1 2015-01-01       7.5    2.829   1.6771     0.3699       0.0017
    ## 2 2015-01-02      10.5    0.065   1.2188     0.0000       0.0017
    ## 3 2015-01-03       6.8    0.000   1.4352     0.0000       0.0017
    ## 4 2015-01-04       6.7    1.107   1.3349     0.1247       0.0017
    ## 5 2015-01-05       6.6    0.165   1.8848     0.0000       0.0017
    ## 6 2015-01-06      11.1    0.000   1.7053     0.0000       0.0017
    ##   OUT_SOIL_MOIST_lyr_1 OUT_EVAP_CANOP OUT_SURF_TEMP
    ## 1              25.2455         0.0000       29.8297
    ## 2              24.8014         0.2505       22.5682
    ## 3              24.2769         0.0000       25.3413
    ## 4              24.5836         0.0000       27.9340
    ## 5              24.0370         0.3505       29.7361
    ## 6              23.4706         0.0000       31.6070
