
# fasterize

Fast polygon-to-raster conversion, burn polygon shapes and/or values
into pixels.

<!-- badges: start -->

[![R-CMD-check](https://github.com/hypertidy/fasterize/actions/workflows/R-CMD-check.yaml/badge.svg)](https://github.com/hypertidy/fasterize/actions/workflows/R-CMD-check.yaml)
[![Project Status: Active – The project has reached a stable, usable
state and is being actively
developed.](http://www.repostatus.org/badges/latest/active.svg)](http://www.repostatus.org/#active)
[![MIT Licensed - Copyright 2016 EcoHealth
Alliance](https://img.shields.io/badge/license-MIT-blue.svg)](https://badges.mit-license.org/)
[![CRAN
status](https://www.r-pkg.org/badges/version/fasterize)](https://CRAN.R-project.org/package=fasterize)
[![CRAN RStudio mirror
downloads](https://cranlogs.r-pkg.org/badges/fasterize)](https://www.r-pkg.org/pkg/fasterize)
[![Codecov test
coverage](https://codecov.io/gh/hypertidy/fasterize/graph/badge.svg)](https://app.codecov.io/gh/hypertidy/fasterize)
<!-- badges: end -->

**fasterize** is a high-performance replacement for the `rasterize()`
function in the [**raster**](https://cran.r-project.org/package=raster)
package.

Functionality is currently limited to rasterizing polygons in
[**sf**](https://cran.r-project.org/package=sf)-type data frames.

## Installation

Install the current version of **fasterize** from CRAN:

``` r
install.packages('fasterize')
```

Install the development version of **fasterize** with
[**devtools**](https://cran.r-project.org/package=devtools):

``` r
devtools::install_github("hypertidy/fasterize")
```

**fasterize** uses [**Rcpp**](https://cran.r-project.org/package=Rcpp)
and thus requires a compile toolchain to install from source. Testing
(and for normal use of sf objects) requires
[**sf**](https://cran.r-project.org/package=sf), which requires GDAL,
GEOS, and PROJ to be installed.

## Usage

The main function, `fasterize()`, takes the same inputs as
`raster::rasterize()` but currently has fewer options and is is limited
to rasterizing polygons.

A `raster()` and `plot()` methods for rasters are re-exported from the
[raster package](https://cran.r-project.org/package=raster).

``` r
library(raster)
library(fasterize)
library(wk)
library(fasterize)
p123 <- c(paste0("POLYGON ((-180 -20, -140 55, 10 0, -140 -60, -180 -20),", 
                  "(-150 -20, -100 -10, -110 20, -150 -20))"), 
             "POLYGON ((-10 0, 140 60, 160 0, 140 -55, -10 0))", 
             "POLYGON ((-125 0, 0 60, 40 5, 15 -45, -125 0))")
pols <- data.frame(value = seq_along(p123), geometry = wk::as_wkt(p123))
ex <- as.numeric(wk_bbox(pols))[c(1, 3, 2, 4)]
r <- raster::raster(raster::extent(ex), res = 1)
r <- fasterize(pols, r, field = "value", fun="sum")
plot(r)
```

![](vignettes/readme-example-1-1.png)<!-- -->

## Performance

Let’s compare `fasterize()` to `terra::rasterize()`:

``` r
pols_t <- terra::vect(p123)
```

    #> dialect must be 'OGRSQL' or 'SQLITE', default value '' means 'OGRSQL'

``` r
pols_t$value <- 1:3
#pols_r <-  as(pols_t, "Spatial")
tr <- terra::rast(r)

bench <- microbenchmark::microbenchmark(
 # rasterize = r <- raster::rasterize(pols_r, r, field = "value", fun="sum"),
  terrarize = tr <- terra::rasterize(pols_t, tr, field = "value", fun = "sum"),
  fasterize = f <- fasterize(pols, r, field = "value", fun="sum"),
  unit = "ms"
)

print(bench, digits = 3)
```

    #> Unit: milliseconds
    #>       expr   min    lq   mean median     uq  max neval cld
    #>  terrarize 9.464 9.761 11.306 10.022 10.375 82.0   100  a 
    #>  fasterize 0.647 0.684  0.839  0.845  0.971  1.1   100   b

How does `fasterize()` do on a large set of polygons? Here I download
the IUCN shapefile for the ranges of all terrestrial mammals and
generate a 1/6 degree world map of mammalian biodiversity by rasterizing
all the layers.

(this doesn’t work anymore because the source data is gone, left as a
record 2024-09-25).

``` r
if(!dir.exists("Mammals_Terrestrial")) {
  download.file(
    "https://s3.amazonaws.com/hp3-shapefiles/Mammals_Terrestrial.zip",
    destfile = "Mammals_Terrestrial.zip") # <-- 383 MB
  unzip("Mammals_Terrestrial.zip", exdir = ".")
  unlink("Mammals_Terrestrial.zip")
}
```

``` r
mammal_shapes <- st_read("Mammals_Terrestrial")
mammal_raster <- raster(mammal_shapes, res = 1/6)
bench2 <- microbenchmark::microbenchmark(
  mammals = mammal_raster <- fasterize(mammal_shapes, mammal_raster, fun="sum"),
  times=20, unit = "s")
print(bench2, digits=3)
par(mar=c(0,0.5,0,0.5))
plot(mammal_raster, axes=FALSE, box=FALSE)
```

    #> Unit: seconds
    #>     expr   min    lq  mean median    uq   max neval
    #>  mammals 0.847 0.857 0.883  0.886 0.894 0.963    20

![](vignettes/readme-so-damn-fast-1.png)

## About

**fasterize** was developed openly at [EcoHealth
Alliance](https://www.ecohealthalliance.org/) under the USAID PREDICT
project by Noam Ross. The repository for hosting fasterize was taken
over by Michael Sumner in December 2022, and was later migrated from
Github ‘ecohealthalliance/fasterize’ to
<https://github.com/hypertidy/fasterize> in March 2025.

Please note that this project is released with a [Contributor Code of
Conduct](CODE_OF_CONDUCT.md). By participating in this project you agree
to abide by its terms.

[![https://www.ecohealthalliance.org/](vignettes/eha-footer.png)](https://www.ecohealthalliance.org/)
[![https://ohi.vetmed.ucdavis.edu/programs-projects/predict-project](vignettes/predictfooter.png)](https://ohi.vetmed.ucdavis.edu/programs-projects/predict-project)
