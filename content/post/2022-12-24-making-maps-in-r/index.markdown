---
title: "Making maps in R"
author: "Tejendra Pratap Singh"
date: "2022-12-24T12:00:00-07:00"
output: html_document
bibliography: references.bib
csl: journal-of-environmental-economics-and-management.csl
draft: true
---

I recently shifted from using the `raster` package for spatial data to using the incredibly versatile and fast `terra` package. Since, `tmap` does not provide direct integration with `terra` objects, I wrote this post to show how `terra` allows for plotting nice graphics using it’s inbuilt functions.

I am going to create a map of continental United States with the spatial distribution of annual `\(PM2.5\)` concentration for 2019. I use `\(PM2.5\)` concentration data from [Atmospheric Composition Analysis Group](https://sites.wustl.edu/acag/datasets/surface-pm2-5/) (Donkelaar et al., 2021).

We begin by loading our required packages.

``` r
rm(list=ls())
gc()

#load required packages
require(data.table)
require(here)
require(pbapply) 
library(terra)

# check root directory
here::here()
```

I use the United States Census Bureau shapefile which can be downloaded from [here](https://www2.census.gov/geo/tiger/GENZ2018/shp/cb_2018_us_state_500k.zip). I now import the shapefile.

``` r
us.shp <- terra::vect(x = here::here('static', 'data', 'shapefile', 'cb_2018_us_state_500k', 'cb_2018_us_state_500k.shp'))
print(us.shp)
```

    ##  class       : SpatVector 
    ##  geometry    : polygons 
    ##  dimensions  : 56, 9  (geometries, attributes)
    ##  extent      : -179.1489, 179.7785, -14.5487, 71.36516  (xmin, xmax, ymin, ymax)
    ##  source      : cb_2018_us_state_500k.shp
    ##  coord. ref. : lon/lat NAD83 (EPSG:4269) 
    ##  names       : STATEFP  STATENS    AFFGEOID GEOID STUSPS           NAME  LSAD
    ##  type        :   <chr>    <chr>       <chr> <chr>  <chr>          <chr> <chr>
    ##  values      :      28 01779790 0400000US28    28     MS    Mississippi    00
    ##                     37 01027616 0400000US37    37     NC North Carolina    00
    ##                     40 01102857 0400000US40    40     OK       Oklahoma    00
    ##       ALAND     AWATER
    ##       <int>      <int>
    ##  1274435193 -368047538
    ##  1369604480  581169507
    ##  1569266587 -920379299

I subset the shapefile by removing the states that are not part of continental United States.

``` r
us.shp <- us.shp[us.shp$NAME %in% us.shp$NAME[!us.shp$NAME %in% c("Puerto Rico", "American Samoa", "United States Virgin Islands", "Hawaii", "Guam", "Commonwealth of the Northern Mariana Islands", "Alaska")],]
us.shp <- terra::project(x = us.shp, y = 'EPSG:5070')
```

I now import the `\(PM2.5\)` concentration data for 2019.

``` r
poll.dt <- terra::rast(x = here::here('static', 'data', 'pollution', 'V5GL03.HybridPM25.NorthAmerica.201901-201912.nc'),
                       subds = c('GWRPM25'),
                       lyrs = 1L)
```

I now subset the `\(PM2.5\)` concentration raster layer for the continental United States.

``` r
poll.dt <- terra::project(x = poll.dt,
                          y = 'EPSG:5070')
poll.dt <- terra::crop(x = poll.dt,
                       y = us.shp)
poll.dt <- terra::mask(x = poll.dt,
                       mask = us.shp)
```

I now plot the map.

``` r
terra::plot(x = poll.dt, type = 'continuous', pax = list(labels = FALSE, tick = FALSE), maxcell = 300000000000000000)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-6-1.png" width="672" />

#### References

<div id="refs" class="references csl-bib-body hanging-indent">

<div id="ref-vandonkelaaretal2021" class="csl-entry">

Donkelaar, A. van, Hammer, M.S., Bindle, L., Brauer, M., Brook, J.R., Garay, M.J., Hsu, N.C., Kalashnikova, O.V., Kahn, R.A., Lee, C., Levy, R.C., Lyapustin, A., Sayer, A.M., Martin, R.V., 2021. Monthly global estimates of fine particulate matter and their uncertainty. Environmental Science & Technology 55, 15287–15300. <https://doi.org/10.1021/acs.est.1c05309>

</div>

</div>
