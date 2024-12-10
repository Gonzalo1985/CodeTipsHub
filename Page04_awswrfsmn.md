---
title: Relative humidity calibration for the WRF model
layout: post
subtitle: Gonzalo M. Díaz
leafletmap: true
always_allow_html: yes
last_modified_at: 2024-12-04
output: 
  md_document:
    variant: gfm
    preserve_yaml: true
---

Post for calibration of relative humidity of WRF model

## Data download

First, the aws.wrfsmn and others libraries should be open:

``` r
library("aws.wrfsmn")
library("dplyr")
library("terra")
library("tibble")
```

Then, some WRF filenames are defined from WRF SMN AWS service

``` r
filenames <- c()
for (i in 1:2)
  {aux.filenames <- get.wrf.files(year = 2023, month = 1, day = i, cycle = 12, time = "01H")
   filenames <- c(filenames, aux.filenames)
  }
print(filenames[1:13])
```

    ##  [1] "DATA/WRF/DET/2023/01/01/12/WRFDETAR_01H_20230101_12_000.nc"
    ##  [2] "DATA/WRF/DET/2023/01/01/12/WRFDETAR_01H_20230101_12_001.nc"
    ##  [3] "DATA/WRF/DET/2023/01/01/12/WRFDETAR_01H_20230101_12_002.nc"
    ##  [4] "DATA/WRF/DET/2023/01/01/12/WRFDETAR_01H_20230101_12_003.nc"
    ##  [5] "DATA/WRF/DET/2023/01/01/12/WRFDETAR_01H_20230101_12_004.nc"
    ##  [6] "DATA/WRF/DET/2023/01/01/12/WRFDETAR_01H_20230101_12_005.nc"
    ##  [7] "DATA/WRF/DET/2023/01/01/12/WRFDETAR_01H_20230101_12_006.nc"
    ##  [8] "DATA/WRF/DET/2023/01/01/12/WRFDETAR_01H_20230101_12_007.nc"
    ##  [9] "DATA/WRF/DET/2023/01/01/12/WRFDETAR_01H_20230101_12_008.nc"
    ## [10] "DATA/WRF/DET/2023/01/01/12/WRFDETAR_01H_20230101_12_009.nc"
    ## [11] "DATA/WRF/DET/2023/01/01/12/WRFDETAR_01H_20230101_12_010.nc"
    ## [12] "DATA/WRF/DET/2023/01/01/12/WRFDETAR_01H_20230101_12_011.nc"
    ## [13] "DATA/WRF/DET/2023/01/01/12/WRFDETAR_01H_20230101_12_012.nc"

The data of WRF model is downloaded from AWS
(<https://registry.opendata.aws/smn-ar-wrf-dataset/>). The documentation
of this dataset can be obtained from:
<https://odp-aws-smn.github.io/documentation_wrf_det/>.

Finally the dataset is downloaded from the ‘filenames’ variable:

``` r
wrf.download(wrf.name = filenames)
```

After this line, the dataset defined in ‘filenames’ will be downloaded.

## Definition of predictors variables

The variables for the adjustment can be any of those found in the WRF
dataset. Here would be:

``` r
variables <- rast("./_includes/WRFDETAR_01H_20230101_12_000.nc") %>% names()
print(variables)
```

    ##  [1] "PP"          "T2"          "PSFC"        "TSLB"        "SMOIS"      
    ##  [6] "ACLWDNB"     "ACLWUPB"     "ACSWDNB"     "HR2"         "dirViento10"
    ## [11] "magViento10"

The definition of each variable can be obtained from:
<https://odp-aws-smn.github.io/documentation_wrf_det/Formato_de_datos/>.
In this case the T2, HR2, SMOIS and magViento10 will be the variables to
use for the calibration of the Relative Humidity (but other combinations
of variables could be a better fit).

The nc files are open with the terra package and separated in each
variable:

``` r
files <- list.files(path = "./_includes/", pattern = "nc", full.names = TRUE)
nc.files <- rast(files)

T2 <- nc.files[[which(names(nc.files) == "T2")]]
HR2 <- nc.files[[which(names(nc.files) == "HR2")]]
SMOIS <- nc.files[[which(names(nc.files) == "SMOIS")]]
magViento10 <- nc.files[[which(names(nc.files) == "magViento10")]]
```

Before extraction of data by location point, the transformation of
coordinate reference should be done:

``` r
T2 <- project(T2, "+proj=longlat +datum=WGS84", method = "bilinear")
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
HR2 <- project(HR2, "+proj=longlat +datum=WGS84", method = "bilinear")
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
SMOIS <- project(SMOIS, "+proj=longlat +datum=WGS84", method = "bilinear")
```

    ## |---------|---------|---------|---------|=========================================                                          

``` r
magViento10 <- project(magViento10, "+proj=longlat +datum=WGS84", method = "bilinear")
```

    ## |---------|---------|---------|---------|=========================================                                          

The location in which will try to calibrate the WRF model will be
Sunchales City, in the center of Argentina, the location of this city
is:

``` r
LON <- -61.53258
LAT <- -30.95686
```

Now, the temporal series of each variable are taken in that location:

``` r
T2.ts <- extract(T2, vect(cbind(LON, LAT)), ID = FALSE)
HR2.ts <- extract(HR2, vect(cbind(LON, LAT)), ID = FALSE)
SMOIS.ts <- extract(SMOIS, vect(cbind(LON, LAT)), ID = FALSE)
magViento.ts <- extract(magViento10, vect(cbind(LON, LAT)), ID = FALSE)
```

Now the data is arranged in a table with the Date information as first
column:

``` r
data.table <- tibble(Date = time(T2),
                     T2.wrf = t(T2.ts),
                     HR2.wrf = t(HR2.ts),
                     SMOIS.wrf = t(SMOIS.ts),
                     magViento.wrf = t(magViento.ts))
data.table
```

    ## # A tibble: 146 × 5
    ##    Date                T2.wrf[,1] HR2.wrf[,1] SMOIS.wrf[,1] magViento.wrf[,1]
    ##    <dttm>                   <dbl>       <dbl>         <dbl>             <dbl>
    ##  1 2023-01-01 12:00:00       24.5        58.2         0.139              1.96
    ##  2 2023-01-01 13:00:00       28.9        35.5         0.139              2.86
    ##  3 2023-01-01 14:00:00       30.8        29.1         0.139              2.81
    ##  4 2023-01-01 15:00:00       31.7        25.8         0.139              1.58
    ##  5 2023-01-01 16:00:00       31.3        30.2         0.139              8.18
    ##  6 2023-01-01 17:00:00       29.4        34.5         0.139              7.26
    ##  7 2023-01-01 18:00:00       26.5        46.7         0.139              9.41
    ##  8 2023-01-01 19:00:00       28.0        41.3         0.139              7.19
    ##  9 2023-01-01 20:00:00       25.3        58.9         0.142              4.29
    ## 10 2023-01-01 21:00:00       26.9        45.9         0.142              2.83
    ## # ℹ 136 more rows

<!-- ## Definition of parameters of the Multiple Linear Regression -->
<!-- The data now will be trained with the 2015-01-01 to 2016-12-31 period using 'multiple.guidance' function with the *predictors.variables* vector: -->
<!-- ```{r} -->
<!-- data <- eva -->
<!-- data.training <- data[1:which(data$Dates == "2016-12-31"),] -->
<!-- ml.model <- multiple.guidance(input.data = data.training, -->
<!--                               predictand = 'evapo_obs', -->
<!--                               predictors = predictors.variables) -->
<!-- ml.model$coefficients -->
<!-- ``` -->
<!-- Now, the parameters can be used to evaluate the model in any dataset. Here it is applied to the same training period: -->
<!-- ```{r} -->
<!-- train.eval <- mg.evaluation(input.data = data.training, predictand = 'evapo_obs', -->
<!--                             predictors = predictors.variables, -->
<!--                             var.model = 'OUT_EVAP', -->
<!--                             lmodel = ml.model) -->
<!-- ``` -->
<!-- The second element of the list has the statistics parameters of the calibration: -->
<!-- ```{r} -->
<!-- train.eval[[2]] -->
<!-- ``` -->
<!-- And the plot can be visualize with 'ploting' function, but first the monthly data is calculated (for better visualization) with 'daily2monthly' function: -->
<!-- ```{r} -->
<!-- ploting(daily2monthly(data = train.eval[[1]])) -->
<!-- ``` -->
<!-- ## Calibration of evaporation soil model output for a verification dataset -->
<!-- The parameters of the previous section now are applied to the data.verification period. This period will be defined from 2017-01-01 to 2017-12-31. Then, the statistics parameters of the calibration in this dataset are shown: -->
<!-- ```{r} -->
<!-- data.verification <- data[which(data$Dates == "2017-01-01"):which(data$Dates == "2017-12-31"),] -->
<!-- verif.eval <- mg.evaluation(input.data = data.verification, predictand = 'evapo_obs', -->
<!--                             predictors = predictors.variables, -->
<!--                             var.model = 'OUT_EVAP', -->
<!--                             lmodel = ml.model) -->
<!-- verif.eval[[2]] -->
<!-- ``` -->
<!-- Finally, the monthly plot of this dataset is displayed below: -->
<!-- ```{r} -->
<!-- ploting(daily2monthly(data = verif.eval[[1]])) -->
<!-- ``` -->
