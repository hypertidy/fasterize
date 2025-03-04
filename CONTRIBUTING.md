# Contributing to the fasterize R package

You want to contribute to **fasterize**? Great! 

Please submit questions, bug reports, and requests in the [issues tracker](https://github.com/hypertidy/fasterize/issues). Please submit bug
reports with a minimal  [reprex](https://www.tidyverse.org/help/#reprex).

If you plan to contribute code, go ahead and fork the repo and submit a pull request. A few notes:

-   This package is released with a [Contributor Code of Conduct](CODE_OF_CONDUCT.md). By participating in this project you agree to abide by its terms.  Why? We want contribution to be enjoyable
and rewarding for everyone!
-   If you have large change, please open an issue first to discuss.
-   I'll generally include contributors as authors in the DESCRIPTION file (with
their permission) for most contributions that go beyond small typos in code or documentation.
-   This package generally uses the [rOpenSci packaging guidelines](https://github.com/ropensci/onboarding/blob/master/packaging_guide.md) for style and structure.
-   Documentation is generated by **roxygen2**. Please write documentation in code files and let it auto-generate documentation files.  We use a recent version so documentation my be [written in markdown](https://cran.r-project.org/web/packages/roxygen2/vignettes/markdown.html)
-   We aim for testing that has high coverage and is robust.  Include tests with
   any major contribution to code. Test your changes the package with [**goodpractice**](https://github.com/MangoTheCat/goodpractice) before
submitting your change.

## Details

- we import raster >= 2.8.3 as that was when sf was supported by raster() https://github.com/rspatial/raster/blame/332bc98bd6edbd7fd64fce2e8c64dea77597dff4/DESCRIPTION

## Roadmap

**fasterize** aims to be limited in scope, lightweight in code and dependencies,
easy to maintain, and fast as hell.  It's limited to operations converting vector
to raster and raster to vector data, in C++, with minimum memory
use or function overhead, dealing with only {sf} and {raster} format. This might change to become more "standard library" like, and just operate on basic inputs with dense output.  Conversion to other vector/raster types and or within-type processing can be handled elsewhere so that we can just have a focussed tool doing a task. 

Things we want to do:


-  Rasterization of lines and points.
-  Conversion of rasters to shapes.  There are half-finished `contourize()` and
   `polygonize()` functions on a branch that might be resurrected.
-  More aggregation functions
-  Rasterization directly to (and from) out-of-memory formats.
-  Expose the C++ API to make the low-level functions available for other uses.
- cell abstraction mode, this is advanced in the package hypertidy/controlledburn on github, this can increase the size of raster used without allocating pixels, provides a very dense indexed version of the polygon or line burn itself, and facilitates separating the pixel allocation from the algorithm itself (so that format, matrix, terra, stars, raster, GeoTIFF, ff becomes inconsequential and optional)

Things we don't want to do:

-  Dependency on GDAL or other libraries that need to be installed out of R
-  Plotting functions
-  Stuff that calls back to R from C++
-  Aggregation functions that require keeping large stacks of raster in memory.
   It's better to just use `raster::calc()` on a RasterStack.

In general, speed and handling of very large objects is favored.  Fanatical tweaking
is encouraged.  Even small operations, like type checking, are implemented in C++ to reduce function overhead. Profile and test your code.  On OSX, Instruments works well for this.  The alpha-stage [**gprofiler**](https://r-prof.github.io/gprofiler/) package does joint R/C(++)
code profiling.
