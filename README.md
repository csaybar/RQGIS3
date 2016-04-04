<!-- README.md is generated from README.Rmd. Please edit that file -->
<!-- C:\OSGeo4W64\bin\python-qgis -> opens Python!! -->
RQGIS
=====

RQGIS establishes an interface between R and QGIS, i.e. it allows the user to access QGIS functionalities from within R. It achieves this by using the QGIS API via the command line. This provides the user with an extensive suite of GIS functions, since QGIS combines the functionalities of various GIS - QGIS, SAGA GIS, GRASS GIS and GDAL (Sextante). The main advantages of RQGIS are:

1.  It provides access to QGIS functionalities. Thereby, it calls Python from the command line (QGIS API) but R users can stay in their programming environment of choice without having to touch Python.
2.  It offers a broad suite of geoalgorithms making it possible to solve virtually any GIS problem.
3.  R users can just use one package (RQGIS) instead of using RSAGA and spgrass to access SAGA and GRASS functions. This, however, does not mean that RSAGA and spgrass are obsolete since both packages offer various other advantages. For instance, RSAGA provides many user-friendly and ready-to-use GIS functions such as `rsaga.slope.asp.curv` and `multi.focal.function`. Besides, both RSAGA and spgrass let you specify specific GIS function arguments while retrieving automatically the default values for unspecified GIS function arguments. So far, this is not possible using RQGIS, i.e. you need to specify each single function argument. This can be tedious and we hope that we will take care of this in future releases.

Installation
============

Before installing RQGIS, download the latest OSGeo4W from <http://trac.osgeo.org/osgeo4w/>, and make sure that following components will be installed on your system using the advanced settings during the installation process:

-   gdal
-   grass
-   msys
-   Python27
-   qgis
-   Qt4
-   saga

After the installation of OSGeo4W these programs should be found in `../OSGeo4W64/apps`. Soon, we will also provide you with a detailed OSGeo4W installation manual.

You can install the latest RQGIS development version from Github with

``` r
if (packageVersion("devtools") < 1.6) {
    install.packages("devtools")    
    }
if (!"lazyeval" %in% installed.packages()[, "Package"]) {
devtools::install_github("hadley/lazyeval")  
}
devtools::install_github("jannes-m/RQGIS")
```

**Please note that RQGIS is still a beta version and under active development.** Therefore, it is likely that major changes will occur in the near future. If you detect any bugs, let us know or, even better, commit a pull request.

Usage
=====

Subsequently, we will show you a typical workflow of how to use RQGIS. Let's start with a very simple example and assume that we simply wanted to add coordinates to a spatial object. Using the raster package, we download administrative areas of Germany. Secondly, we save the resulting SpatialObject as a shapefile in a temporary folder.

``` r
# attach packages
library("raster")
library("rgdal")

# define path to a temporary folder
dir_tmp <- tempdir()
# download German administrative areas
ger <- getData(name = "GADM", country = "DEU", level = 2)
# save ger as a shapefile
writeOGR(ger, dir_tmp, "ger", driver = "ESRI Shapefile")
```

Now that we have a shapefile, we can move on to using RQGIS. First of all, we need to find out how the function in QGIS is called which adds coordinates to the attribute table of a shapefile. To do so, we use `find_algorithms`. We suspect that the function we are looking for contains the word *add* and *coordinate*. Please note, that `find_algorithms` needs to know where OSGeo4W is located on your system. Under Windows `find_root` will look for OSGeo4W in `C:`, `C:/Program Files` and `C:/Program Files (x86)`. If you are using Linux or a Mac or if you have installed OSGeo4W in another location, you need to specify the path to your OSGeo4W-installation yourself. Note also, that most other RQGIS functions require the path to your OSGeo4W-installation such as `get_usage`, `get_options` and `run_qgis`.

``` r
# attach RQGIS
library("RQGIS")

# look for a function that contains the words "add" and "coordinate"
find_algorithms(search_term = "add coordinate", 
                osgeo4w_root = find_root())
#> [1] "Add coordinates to points---------------------------->saga:addcoordinatestopoints"
#> [2] ""
```

This gives us a function named `saga:addcoordinatestopoints`. Subsequently, we would like to know how we can use it, i.e. which function parameters we need to specify.

``` r
get_usage(algorithm_name = "saga:addcoordinatestopoints", intern = TRUE)
#> [1] "ALGORITHM: Add coordinates to points"
#> [2] "\tINPUT <ParameterVector>"            
#> [3] "\tOUTPUT <OutputVector>"              
#> [4] ""                                    
#> [5] ""                                    
#> [6] ""
```

All the function expects is a parameter called INPUT, i.e. the path to a shapefile whose attribute table we wish to extend with coordinates, and a parameter called OUTPUT, i.e. the path to the output shapefile. `run_qgis` expects exactly these function parameters as a list.

``` r

library("rgdal")
# construct a list with our function parameters
params <- list(
  # path to the input shapefile
  INPUT = paste(dir_tmp, "ger.shp", sep = "\\"),
  # path to the output shapefile
  OUTPUT = paste(dir_tmp, "ger_coords.shp", sep = "\\"))
run_qgis(algorithm = "saga:addcoordinatestopoints", 
         params = params)

# load the shapefile QGIS has created for us
ger_coords <- readOGR(dsn = dir_tmp, layer = "ger_coords", verbose = FALSE)

# let's have a look at the output
head(ger_coords@data, 2)
#>   OBJECTID ID_0 ISO  NAME_0 ID_1            NAME_1 ID_2          NAME_2
#> 0        1   86 DEU Germany    1 Baden-Württemberg    1 Alb-Donau-Kreis
#> 1        2   86 DEU Germany    1 Baden-Württemberg    2       Böblingen
#>     HASC_2 CCN_2 CCA_2    TYPE_2 ENGTYPE_2 NL_NAME_2 VARNAME_2        X
#> 0 DE.BW.AD    NA 08425 Landkreis  District      <NA>      <NA> 9.948325
#> 1 DE.BW.BL    NA 08115 Landkreis  District      <NA>      <NA> 8.938024
#>          Y
#> 0 48.63110
#> 1 48.86639
```

Excellent! QGIS added the coordinates to the attribute table of our shapefile in columns x and y using SAGA. Of course, this is a very simple example. We could have achieved the same using `sp::coordinates`. To harness the real power of integrating R with a GIS, we will present a second, more complex example. Yet to come in the form of a vignette...

TO DO:
======

-   OSGeo4w installation guide/manual with screenshots
-   processing.runalg -&gt; user has to provide each argument and cannot call single arguments, find out if there is a more user-friendly way
-   Check if run\_qgis in fact is able to run all QGIS, SAGA, GRASS, GDAL functions (Sextante). It could be a problem that one needs to specify function arguments as characters.
-   Take care of the error message: ERROR 1: Can't load requested DLL: C:4~1\_FileGDB.dll 193: %1 ist keine zulässige Win32-Anwendung.
-   Write find\_root for Linux and Apple
-   Write html-vignette, i.e. present a more complex QGIS example
-   find out if SAGA and GRASS can be located somewhere else on the system, i.e. if they can be located outside of C:/OSGeo4W64
-   look for SAGA, GRASS GIS, GDAL, sys, etc. (and find out if we need sys and if we need to specify the paths to the GIS in QGIS, I don't think so)
-   you should write a `check_apps`-function testing if all necessary components were installed (saga, rgdal, grass, qgis, Python, msys, ...)
