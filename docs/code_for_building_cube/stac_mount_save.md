STAC_mount_save
================
Ty Tuff, ESIIL Data Scientist
2023-10-27

## ‘To Make’ or ‘To Take’ a photo

The distinction between making and taking a photograph lies in the
approach and intent behind the camera. Taking a photo is often a
reactive process, where the photographer captures moments as they
naturally unfold, seizing the spontaneity of life without alteration.
It’s a passive form of photography where the emphasis is on the right
timing and the natural interplay of elements within the frame. On the
other hand, making a photo is a proactive and deliberate act. It is akin
to craftsmanship, where a professional photographer starts with a
concept and utilizes a variety of tools and techniques to stage and
construct the desired scene. They actively manipulate lighting,
composition, and subjects to create a photograph that aligns with their
pre-visualized artistic vision. While both methods use a camera to
produce a photograph, making a photo involves a creation process,
whereas taking a photo is about finding the scene.

David Yarrow is a famous photographer who ‘makes’ his photographs.
![](../assets/stac_mount_save/David-Yarrow-sorrel-sky-gallery-Photographic-Print-Cindys-Shotgun-Wedding.png)
![](../assets/stac_mount_save/bison.png)

## What does it mean to ‘make’ a data cube?

The artistry of Ansel Adams’ photography serves as a compelling analogy
for the meticulous craft of building a data cube from cloud data sources
using tools like STAC and GDAL VSI. Just as Adams would survey the
vastness of a landscape, discerning the interplay of light and shadow
upon the mountains before him, a data architect surveys the expanse of
available data. In this analogy, the raw data are the majestic mountains
and sweeping landscapes waiting to be captured. The STAC collection acts
as the photographer’s deliberate choice of scene, pointing the camera
lens—our data tools—towards the most telling and coherent dataset.

![](../assets/stac_mount_save/Ansel_Adams_datacube.png) Just as Adams’
photographs are more than mere records of a landscape, but rather a
confluence of his vision, technique, and the scene’s natural beauty, so
too is the data cube more than the sum of its parts. It is the artful
synthesis of information, crafted and composed with the skill and intent
of an artist, producing not just a tool for analysis but a harmonized,
data-driven portrait of the world it represents. The builder of the data
cube is, indeed, an artist, and the data cube their masterpiece,
revealing not just data, but a story, a perspective, a landscape sewn
from the raw material of cloud-sourced information.

As Adams would adjust his viewfinder, setting the boundaries of his
photographic frame, the data builder sets the view window, filtering and
transferring relevant data to their own medium, akin to Adams’ film.
This is where the raw data is transformed, organized into the structured
form of a data frame or data cube, a process not unlike the careful
development of a photograph in a darkroom. Here, the data cube creator,
much like Adams with his careful dodging and burning, harmonizes
disparate elements into a cohesive whole, each decision reflecting an
intention and vision for the final product.

``` r
#library(Rcpp)
library(sf)
library(gdalcubes)
library(rstac)
library(gdalUtils)
library(terra)
library(rgdal)
library(reshape2)
library(osmdata)
library(terra)
library(dplyr)
library(stars)
library(ggplot2)
library(colorspace)
library(geos)
library(osmdata)
library(ggthemes)
library(tidyr)
gdalcubes_options(parallel = 8)

sf::sf_extSoftVersion()
```

              GEOS           GDAL         proj.4 GDAL_with_GEOS     USE_PROJ_H 
          "3.11.0"        "3.5.3"        "9.1.0"         "true"         "true" 
              PROJ 
           "9.1.0" 

``` r
gdalcubes_gdal_has_geos()
```

    [1] TRUE

``` r
library(osmdata)
library(dplyr)
library(sf)
library(terra)
library(tidyterra)
library(glue)
library(ggplot2)
library(ggthemes)
library(stars)
library(magrittr)
```

## 1) The Rat through the Snake Problem: Scalability with Cloud Computing

Just like a snake that swallows a rat, traditional computing systems
often struggle to process the large volumes of environmental data —
they’re constrained by their static hardware limitations. Cloud
computing introduces a python-esque capability: massive scalability. By
migrating to the cloud, we essentially make the snake bigger, allowing
it to handle larger “prey.” Scalable computers in the cloud can grow
with the demand, providing the necessary computational power to process
extensive datasets, which is vital in a field where data volumes are
increasing exponentially.

![Raster through a snake](../assets/mouseinsnake.png)

## 2) The Antelope through the Python Problem: Streamlining with GDAL VSI

As we scale up, we encounter a new challenge: trying to pass an antelope
through a python — a metaphor for the next level of complexity in data
processing. The sheer size and complexity of the data can become
overwhelming. This is where GDAL’s Virtual File System (VSI) becomes our
ecological adaptation. VSI allows us to access remote data transparently
and more efficiently. Instead of ingesting the entire “antelope,” VSI
enables the “python” to dynamically access and process only the parts of
the data it needs, when it needs them, much like constriction before
digestion. This selective access minimizes the need for local storage
and expedites the data handling process.

![Antelope through a Python](../assets/antelopeinpython.png)

### Mounting data

A void-filled Digital Elevation Model (DEM) is a comprehensive
topographical representation where any missing data points, known as
voids, have been filled in. These voids can occur due to various
reasons, such as clouds or technical errors during data collection. In a
void-filled DEM, these gaps are interpolated or estimated using the
surrounding data to create a continuous, seamless surface model. This
process enhances the utility and accuracy of the DEM for hydrological
modeling, terrain analysis, and other geographical applications. The
HydroSHEDS website
(https://www.hydrosheds.org/hydrosheds-core-downloads) provides access
to high-quality, void-filled DEM datasets like the
DEM_continuous_CONUS_15s, which users can download and easily integrate
into spatial analysis workflows using tools such as ‘terra’ in R,
allowing for sophisticated environmental and geographical research and
planning.

``` r
# Record start time
a <- Sys.time()  

# Create a string with the file path using glue, then download and read the DEM file as a raster object

DEM_continuous_CONUS_15s <- glue(
  "/vsizip/vsicurl/", #magic remote connection 
  "https://data.hydrosheds.org/file/hydrosheds-v1-dem/hyd_na_dem_15s.zip", #copied link to download location
  "/hyd_na_dem_15s.tif") %>% #path inside zip file
  terra::rast()  

# The 'glue' function constructs the file path string, which is then passed to 'terra::rast()' to read the DEM file into R as a raster layer. '/vsizip/vsicurl/' is a special GDAL virtual file system syntax that allows reading directly from a zipped file on a remote server.

# Record end time and calculate the time difference
b <- Sys.time()  
difftime(b, a) 
```

    Time difference of 0.5191751 secs

``` r
# The resulting raster object is stored in 'DEM_continuous_CONUS_15s', which now contains the void-filled DEM data ready for use

DEM_continuous_CONUS_15s  # Prints out the details of the 'DEM_continuous_CONUS_15s' raster object
```

    class       : SpatRaster 
    dimensions  : 13920, 20640, 1  (nrow, ncol, nlyr)
    resolution  : 0.004166667, 0.004166667  (x, y)
    extent      : -138, -52, 5, 63  (xmin, xmax, ymin, ymax)
    coord. ref. : lon/lat WGS 84 (EPSG:4326) 
    source      : hyd_na_dem_15s.tif 
    name        : Band_1 

``` r
# output is a SpatRaster, which is the object type associated with the 'terra' package. 
```

``` r
# Record start time
a <- Sys.time()

ggplot() +
  geom_spatraster(data=DEM_continuous_CONUS_15s) +
  theme_tufte()
```

![](stac_mount_save_files/figure-gfm/DEM_plot-1.png)

``` r
b <- Sys.time()
difftime(b, a)
```

    Time difference of 1.060889 mins

``` r
# Transform the filtered geometry to EPSG:4326 and store its bounding box
# Record start time
a <- Sys.time()

DEM_continuous_CONUS_15s |>
stars::st_as_stars() |> 
  st_transform("EPSG:4326") |>
  st_bbox() -> bbox_4326


DEM_continuous_CONUS_15s |>
stars::st_as_stars() |> 
  st_transform("EPSG:32618") |>
  st_bbox() -> bbox_32618

b <- Sys.time()
difftime(b, a)
```

    Time difference of 3.909238 mins

``` r
boulder_county <- getbb("boulder, co", format_out="sf_polygon")

boulder_county$multipolygon |> 
  st_transform(crs =4326 ) |>
  st_bbox() -> bbox_4326_boulder

boulder_county$multipolygon |> 
  st_transform(crs =32720 ) |>
  st_bbox() -> bbox_32720_boulder
```

``` r
aoi <- getbb("United States", format_out="sf_polygon")

conus <- aoi$multipolygon |>
  st_crop(bbox_4326)


ggplot(data=conus) +
  geom_sf()
```

![](stac_mount_save_files/figure-gfm/conus_bounding_box-1.png)

``` r
 stac("https://earth-search.aws.element84.com/v1") |>
       get_request()
```

    ###STACCatalog
    - id: earth-search-aws
    - description: A STAC API of public datasets on AWS
    - field(s): stac_version, type, id, title, description, links, conformsTo

``` r
collection_formats()
```

       CHIRPS_v2_0_daily_p05_tif | Image collection format for CHIRPS v 2.0 daily
                                 | global precipitation dataset (0.05 degrees
                                 | resolution) from GeoTIFFs, expects list of .tif
                                 | or .tif.gz files as input. [TAGS: CHIRPS,
                                 | precipitation]
     CHIRPS_v2_0_monthly_p05_tif | Image collection format for CHIRPS v 2.0 monthly
                                 | global precipitation dataset (0.05 degrees
                                 | resolution) from GeoTIFFs, expects list of .tif
                                 | or .tif.gz files as input. [TAGS: CHIRPS,
                                 | precipitation]
               ESA_CCI_SM_ACTIVE | Collection format for ESA CCI soil moisture
                                 | active product (version 4.7) [TAGS: Soil
                                 | Moisture, ESA, CCI]
              ESA_CCI_SM_PASSIVE | Collection format for ESA CCI soil moisture
                                 | passive product (version 4.7) [TAGS: Soil
                                 | Moisture, ESA, CCI]
       GPM_IMERG_3B_DAY_GIS_V06A | Collection format for daily
                                 | IMERG_3B_DAY_GIS_V06A data [TAGS: Precipitation,
                                 | GPM, IMERG]
                         L8_L1TP | Collection format for Landsat 8 Level 1 TP
                                 | product [TAGS: Landsat, USGS, Level 1, NASA]
                           L8_SR | Collection format for Landsat 8 surface
                                 | reflectance product [TAGS: Landsat, USGS, Level
                                 | 2, NASA, surface reflectance]
                           MAXAR | Preliminary collection format for MAXAR open
                                 | data, visual only (under development) [TAGS: ]
                         MxD09GA | Collection format for selected bands from the
                                 | MODIS MxD09GA (Aqua and Terra) product [TAGS:
                                 | MODIS, surface reflectance]
                         MxD10A2 | Collection format for selected bands from the
                                 | MODIS MxD10A2 (Aqua and Terra) v006 Snow Cover
                                 | product [TAGS: MODIS, Snow Cover]
                         MxD11A1 | Collection format for selected bands from the
                                 | MODIS MxD11A2 (Aqua and Terra) v006 Land Surface
                                 | Temperature product [TAGS: MODIS, LST]
                         MxD11A2 | Collection format for selected bands from the
                                 | MODIS MxD11A2 (Aqua and Terra) v006 Land Surface
                                 | Temperature product [TAGS: MODIS, LST]
                         MxD13A2 | Collection format for selected bands from the
                                 | MODIS MxD13A2 (Aqua and Terra) product [TAGS:
                                 | MODIS, VI, NDVI, EVI]
                         MxD13A3 | Collection format for selected bands from the
                                 | MODIS MxD13A3 (Aqua and Terra) product [TAGS:
                                 | MODIS, VI, NDVI, EVI]
                         MxD13Q1 | Collection format for selected bands from the
                                 | MODIS MxD13Q1 (Aqua and Terra) product [TAGS:
                                 | MODIS, VI, NDVI, EVI]
                         MxD14A2 | Collection format for the MODIS MxD14A2 (Aqua
                                 | and Terra) product [TAGS: MODIS, Fire]
    PlanetScope_3B_AnalyticMS_SR | Image collection format for PlanetScope 4-band
                                 | scenes [TAGS: PlanetScope, BOA, Surface
                                 | Reflectance]
                   Sentinel2_L1C | Image collection format for Sentinel 2 Level 1C
                                 | data as downloaded from the Copernicus Open
                                 | Access Hub, expects a list of file paths as
                                 | input. The format works on original ZIP
                                 | compressed as well as uncompressed imagery.
                                 | [TAGS: Sentinel, Copernicus, ESA, TOA]
               Sentinel2_L1C_AWS | Image collection format for Sentinel 2 Level 1C
                                 | data in AWS [TAGS: Sentinel, Copernicus, ESA,
                                 | TOA]
                   Sentinel2_L2A | Image collection format for Sentinel 2 Level 2A
                                 | data as downloaded from the Copernicus Open
                                 | Access Hub, expects a list of file paths as
                                 | input. The format should work on original ZIP
                                 | compressed as well as uncompressed imagery.
                                 | [TAGS: Sentinel, Copernicus, ESA, BOA, Surface
                                 | Reflectance]
             Sentinel2_L2A_THEIA | Image collection format for Sentinel 2 Level 2A
                                 | data as downloaded from Theia. [TAGS: Sentinel,
                                 | ESA, Flat Reflectance, Theia]

![](../assets/stac_mount_save/Ansel_adams_Jackson_hole.png)

``` r
# Record start time
a <- Sys.time()

# Initialize STAC connection
s = stac("https://earth-search.aws.element84.com/v0")


# Search for Sentinel-2 images within specified bounding box and date range
#22 Million items
items = s %>%
    stac_search(collections = "sentinel-s2-l2a-cogs",
                bbox = c(bbox_4326_boulder["xmin"], 
                         bbox_4326_boulder["ymin"],
                         bbox_4326_boulder["xmax"], 
                         bbox_4326_boulder["ymax"]), 
                datetime = "2021-05-15/2021-05-16") %>%
    post_request() %>%
    items_fetch(progress = FALSE)

# Print number of found items
length(items$features)
```

    [1] 1

``` r
# Prepare the assets for analysis
library(gdalcubes)
assets = c("B01", "B02", "B03", "B04", "B05", "B06", 
           "B07", 
           "B08", "B8A", "B09", "B11", "B12", "SCL")
s2_collection = stac_image_collection(items$features, asset_names = assets,
property_filter = function(x) {x[["eo:cloud_cover"]] < 20}) #all images with less than 20% clouds

b <- Sys.time()
difftime(b, a)
```

    Time difference of 0.2137389 secs

``` r
# Display the image collection
s2_collection
```

    Image collection object, referencing 1 images with 13 bands
    Images:
                          name      left      top   bottom     right
    1 S2B_13TDE_20210516_0_L2A -106.1832 40.65079 39.65576 -104.8846
                 datetime        srs
    1 2021-05-16T18:02:54 EPSG:32613

    Bands:
       name offset scale unit nodata image_count
    1   B01      0     1                       1
    2   B02      0     1                       1
    3   B03      0     1                       1
    4   B04      0     1                       1
    5   B05      0     1                       1
    6   B06      0     1                       1
    7   B07      0     1                       1
    8   B08      0     1                       1
    9   B09      0     1                       1
    10  B11      0     1                       1
    11  B12      0     1                       1
    12  B8A      0     1                       1
    13  SCL      0     1                       1

![](../assets/stac_mount_save/View_Ansel-Adams_Camera.png)

``` r
# Record start time
a <- Sys.time()

# Define a specific view on the satellite image collection
v = cube_view(
    srs = "EPSG:32720", #this is harder than expected. 
    dx = 100, 
    dy = 100, 
    dt = "P1M", 
    aggregation = "median", 
    resampling = "near",
    extent = list(
        t0 = "2021-05-15", 
        t1 = "2021-05-16",
        left = bbox_32720_boulder[1], 
        right = bbox_32720_boulder[2],
        top = bbox_32720_boulder[4], 
        bottom = bbox_32720_boulder[3]
    )
)

b <- Sys.time()
difftime(b, a)
```

    Time difference of 0.003129005 secs

``` r
# Display the defined view
v
```

    A data cube view object

    Dimensions:
                    low             high  count pixel_size
    t        2021-05-01       2021-05-31      1        P1M
    y -3103099.52398788 15434400.4760121 185375        100
    x -3178878.98542359 15369521.0145764 185484        100

    SRS: "EPSG:32720"
    Temporal aggregation method: "median"
    Spatial resampling method: "near"

``` r
# Record start time
a <- Sys.time()


x <- s2_collection %>%
    raster_cube(v) %>%
    select_bands(c("B01", "B02", "B03", "B04", 
                   "B05", "B06", "B07", "B08", 
                   "B8A", "B09", "B11", "B12")) %>%
    extract_geom(boulder_county$multipolygon) %>%
    rename(
        "time" = "time",
        "443" = "B01",
        "490" = "B02",
        "560" = "B03",
        "665" = "B04",
        "705" = "B05",
        "740" = "B06",
        "783" = "B07",
        "842" = "B08",
        "865" = "B8A",
        "940" = "B09",
        "1610" = "B11",
        "2190" = "B12"
    )

b <- Sys.time()
difftime(b, a)
```

    Time difference of 1.58433 mins

``` r
x
```

         FID       time   443   490   560   665   705   740   783   842   865   940
    1      1 2021-05-01 11096 10929 10224  9893  9956  9706  9715  9641  9511  8459
    2      1 2021-05-01 11631 11282 10550 10234 10288 10031 10032  9988  9828  9153
    3      1 2021-05-01 11900 11393 10666 10337 10398 10142 10138 10093  9927  9461
    4      1 2021-05-01 11406 10597  9928  9626  9694  9481  9516  9338  9336  8959
    5      1 2021-05-01 11399 10939 10237  9905  9978  9738  9746  9633  9555  8925
    6      1 2021-05-01 11600 11174 10462 10147 10209  9952  9960  9890  9760  9153
    7      1 2021-05-01 11699 11246 10538 10225 10286 10042 10046  9955  9845  9212
    8      1 2021-05-01 11923 11510 10789 10459 10524 10254 10263 10188 10042  9406
    9      1 2021-05-01 12156 11732 10988 10647 10728 10441 10424 10363 10205  9783
    10     1 2021-05-01 11789 11224 10512 10191 10268 10014 10021  9906  9823  9384
    11     1 2021-05-01 11795 11356 10638 10339 10417 10160 10165 10041  9960  9431
    12     1 2021-05-01 11708 11201 10490 10178 10240  9994 10001  9888  9784  9289
    13     1 2021-05-01 11699 11285 10583 10260 10318 10063 10069  9991  9852  9252
    14     1 2021-05-01 11693 11512 10770 10432 10492 10214 10205 10162  9974  9166
    15     1 2021-05-01 12247 11715 10972 10629 10704 10422 10410 10349 10179  9842
    16     1 2021-05-01 12394 11888 11155 10819 10888 10608 10586 10530 10362 10005
    17     1 2021-05-01 11548 10974 10263  9928  9975  9716  9746  9707  9543  9019
    18     1 2021-05-01 11799 11327 10595 10269 10335 10066 10070 10003  9862  9364
    19     1 2021-05-01 11919 11356 10638 10339 10417 10160 10165 10041  9960  9552
    20     1 2021-05-01 11962 11309 10592 10280 10360 10105 10103  9960  9892  9653
    21     1 2021-05-01 11931 11267 10570 10249 10318 10065 10070  9925  9854  9579
    22     1 2021-05-01 11888 11427 10704 10375 10453 10176 10175 10038  9961  9501
    23     1 2021-05-01 12057 11815 11064 10719 10782 10491 10471 10449 10231  9692
    24     1 2021-05-01 12520 12050 11314 10990 11068 10771 10762 10704 10522 10070
    25     1 2021-05-01 12490 11965 11206 10863 10949 10664 10660 10556 10425 10037
    26     1 2021-05-01 12595 12250 11423 11040 11120 10806 10771 10777 10511 10289
    27     1 2021-05-01 11563 11278 10559 10241 10299 10042 10056  9984  9862  8970
    28     1 2021-05-01 11808 11302 10577 10258 10312 10061 10064  9990  9869  9321
    29     1 2021-05-01 11831 11522 10787 10461 10540 10269 10265 10165 10038  9366
    30     1 2021-05-01 12093 11697 10971 10648 10727 10466 10457 10336 10223  9760
    31     1 2021-05-01 12181 11633 10890 10544 10627 10340 10328 10232 10110  9896
    32     1 2021-05-01 12175 11845 11104 10753 10829 10544 10527 10440 10299  9874
    33     1 2021-05-01 12364 11979 11237 10895 10974 10672 10660 10614 10420 10048
    34     1 2021-05-01 12463 11979 11237 10895 10974 10672 10660 10614 10420 10052
    35     1 2021-05-01 12501 11999 11253 10921 10997 10716 10717 10616 10489 10095
    36     1 2021-05-01 12771 12137 11332 10968 11043 10738 10716 10671 10470 10422
    37     1 2021-05-01 13172 12698 11873 11501 11590 11256 11215 11228 10951 10831
    38     1 2021-05-01 13395 12892 12089 11734 11810 11474 11432 11446 11171 11038
    39     1 2021-05-01 10790 10470  9789  9471  9503  9286  9318  9291  9148  8186
    40     1 2021-05-01 11217 11012 10296  9979 10006  9772  9793  9766  9598  8585
    41     1 2021-05-01 11158 11012 10296  9979 10006  9772  9793  9766  9598  8490
    42     1 2021-05-01 11759 11437 10694 10349 10402 10133 10127 10088  9910  9234
    43     1 2021-05-01 12151 11642 10900 10557 10615 10343 10336 10296 10110  9744
    44     1 2021-05-01 12173 11687 10932 10573 10653 10377 10356 10276 10135  9880
    45     1 2021-05-01 12546 11942 11179 10811 10886 10589 10566 10534 10338 10253
    46     1 2021-05-01 12715 12116 11378 11042 11138 10849 10838 10736 10602 10403
    47     1 2021-05-01 12680 12251 11519 11191 11287 10996 10992 10889 10762 10304
    48     1 2021-05-01 12702 12183 11405 11059 11135 10843 10824 10753 10595 10293
    49     1 2021-05-01 12972 12453 11651 11294 11371 11059 11030 11011 10782 10574
    50     1 2021-05-01 13195 12751 11943 11563 11632 11300 11255 11296 11003 10808
    51     1 2021-05-01 13463 12969 12206 11864 11949 11620 11601 11550 11346 11018
    52     1 2021-05-01 13471 12716 11972 11623 11706 11394 11391 11296 11153 10947
    53     1 2021-05-01 13149 12519 11753 11385 11443 11123 11102 11072 10857 10593
    54     1 2021-05-01 10113  9973  9329  9042  9074  8877  8933  8878  8778  7580
    55     1 2021-05-01 10342  9921  9294  9016  9046  8865  8921  8828  8761  7724
    56     1 2021-05-01 10473 10383  9707  9404  9418  9208  9251  9221  9066  7867
    57     1 2021-05-01 10786 10612  9920  9611  9645  9422  9453  9389  9267  8109
    58     1 2021-05-01 11690 11358 10612 10259 10300 10039 10021 10024  9799  9198
    59     1 2021-05-01 12106 11707 10957 10599 10677 10399 10379 10319 10154  9774
    60     1 2021-05-01 12348 11961 11200 10848 10928 10639 10617 10555 10388 10076
    61     1 2021-05-01 12531 12146 11388 11048 11133 10845 10822 10778 10597 10241
    62     1 2021-05-01 12758 12288 11543 11208 11294 10991 10976 10944 10744 10345
    63     1 2021-05-01 12652 12140 11379 11045 11111 10834 10812 10752 10585 10194
    64     1 2021-05-01 12672 12401 11608 11254 11330 11027 10987 10980 10750 10237
    65     1 2021-05-01 12935 12453 11651 11294 11371 11059 11030 11011 10782 10509
    66     1 2021-05-01 13210 12595 11797 11431 11510 11184 11153 11147 10910 10740
    67     1 2021-05-01 13370 12944 12169 11795 11873 11532 11497 11520 11250 10856
    68     1 2021-05-01 13321 12832 12077 11726 11814 11491 11472 11418 11237 10778
    69     1 2021-05-01 13135 12506 11758 11394 11481 11167 11152 11057 10917 10550
    70     1 2021-05-01 12911 12391 11614 11226 11309 10992 10966 10895 10727 10352
    71     1 2021-05-01  9952  9765  9134  8849  8886  8708  8763  8652  8601  7448
    72     1 2021-05-01 10733 10363  9679  9365  9417  9201  9221  9135  9042  8267
    73     1 2021-05-01 11032 10622  9947  9641  9699  9482  9506  9391  9324  8516
    74     1 2021-05-01 11194 11007 10303  9982 10041  9790  9815  9719  9622  8633
    75     1 2021-05-01 11494 11082 10345 10002 10050  9798  9804  9764  9597  8985
    76     1 2021-05-01 12364 11675 10908 10538 10598 10310 10290 10301 10055  9975
    77     1 2021-05-01 12596 11998 11239 10885 10976 10678 10664 10608 10434 10285
    78     1 2021-05-01 12580 12094 11333 10994 11096 10804 10785 10715 10562 10234
    79     1 2021-05-01 12579 12108 11344 11007 11082 10789 10762 10740 10532 10174
    80     1 2021-05-01 12592 12140 11379 11045 11111 10834 10812 10752 10585 10107
    81     1 2021-05-01 12705 12259 11470 11103 11159 10859 10826 10834 10580 10260
    82     1 2021-05-01 12996 12496 11720 11369 11455 11142 11121 11073 10886 10570
    83     1 2021-05-01 13032 12659 11875 11496 11577 11249 11227 11228 10983 10616
    84     1 2021-05-01 13206 12834 12076 11721 11799 11475 11448 11429 11206 10707
    85     1 2021-05-01 13093 12554 11798 11427 11492 11178 11164 11132 10911 10510
    86     1 2021-05-01 13073 12504 11740 11361 11448 11127 11101 11043 10854 10482
    87     1 2021-05-01 12998 12429 11648 11255 11344 11025 10991 10948 10753 10471
    88     1 2021-05-01 12940 12482 11687 11286 11359 11026 10996 10985 10740 10467
    89     1 2021-05-01  9710  9240  8646  8391  8428  8276  8362  8240  8230  7217
    90     1 2021-05-01 10329  9845  9186  8889  8927  8738  8787  8685  8628  7848
    91     1 2021-05-01 10914 10511  9819  9515  9576  9364  9384  9279  9210  8504
    92     1 2021-05-01 11274 10737 10053  9745  9797  9581  9598  9499  9404  8843
    93     1 2021-05-01 11493 10999 10295  9969 10025  9775  9795  9723  9591  8980
    94     1 2021-05-01 11782 11434 10688 10326 10387 10113 10112 10070  9899  9315
    95     1 2021-05-01 12132 11873 11096 10714 10778 10475 10451 10509 10214  9690
    96     1 2021-05-01 12752 12221 11461 11135 11239 10948 10936 10851 10721 10385
    97     1 2021-05-01 12704 12129 11364 11029 11121 10830 10813 10732 10584 10391
    98     1 2021-05-01 12835 12240 11481 11156 11242 10947 10937 10853 10706 10478
    99     1 2021-05-01 12698 12119 11339 10983 11063 10767 10752 10685 10521 10238
    100    1 2021-05-01 12605 12384 11611 11247 11328 11012 10998 10960 10754 10167
    101    1 2021-05-01 12931 12569 11789 11413 11489 11162 11145 11156 10901 10484
    102    1 2021-05-01 13055 12569 11789 11413 11489 11162 11145 11156 10901 10525
    103    1 2021-05-01 13152 12712 11958 11616 11691 11376 11356 11325 11119 10571
    104    1 2021-05-01 12994 12468 11721 11372 11444 11135 11127 11055 10890 10424
    105    1 2021-05-01 12823 12419 11644 11251 11326 10993 10977 10969 10730 10344
    106    1 2021-05-01 12930 12406 11620 11231 11308 10984 10955 10939 10702 10423
    107    1 2021-05-01 12922 12418 11623 11222 11300 10971 10947 10927 10693 10471
    108    1 2021-05-01 10117  9645  9000  8720  8744  8574  8642  8596  8491  7611
    109    1 2021-05-01 10844 10622  9926  9634  9681  9477  9492  9429  9316  8393
    110    1 2021-05-01 11275 10888 10192  9890  9953  9725  9746  9652  9569  8902
    111    1 2021-05-01 11373 11031 10327 10003 10076  9831  9843  9740  9637  9000
    112    1 2021-05-01 11768 11339 10613 10272 10352 10095 10099 10019  9884  9373
    113    1 2021-05-01 11962 11591 10819 10448 10525 10248 10236 10187 10019  9583
    114    1 2021-05-01 12225 11873 11096 10714 10778 10475 10451 10509 10214  9853
    115    1 2021-05-01 12680 12199 11417 11062 11132 10828 10804 10836 10555 10270
    116    1 2021-05-01 12891 12362 11598 11273 11374 11081 11073 10976 10854 10438
    117    1 2021-05-01 12671 12254 11471 11126 11201 10903 10876 10849 10648 10281
    118    1 2021-05-01 12766 12220 11443 11089 11166 10867 10845 10793 10607 10331
    119    1 2021-05-01 12950 12452 11691 11338 11420 11109 11089 11039 10841 10511
    120    1 2021-05-01 12949 12437 11672 11300 11380 11066 11050 11020 10816 10530
    121    1 2021-05-01 12994 12459 11719 11381 11450 11142 11143 11075 10919 10368
    122    1 2021-05-01 12773 12259 11493 11119 11188 10870 10848 10830 10618 10196
    123    1 2021-05-01 12815 12321 11530 11151 11228 10909 10877 10853 10619 10337
    124    1 2021-05-01 12934 12406 11624 11247 11321 10994 10975 10942 10717 10498
    125    1 2021-05-01 10033  9612  9027  8795  8844  8685  8761  8588  8625  7542
    126    1 2021-05-01 10054  9612  9027  8795  8844  8685  8761  8588  8625  7572
    127    1 2021-05-01 10947 10473  9783  9482  9517  9305  9320  9324  9143  8464
    128    1 2021-05-01 11333 10767 10079  9778  9830  9608  9628  9583  9461  8932
    129    1 2021-05-01 11585 11094 10389 10068 10137  9891  9907  9830  9711  9228
    130    1 2021-05-01 11747 11339 10613 10272 10352 10095 10099 10019  9884  9355
    131    1 2021-05-01 12313 11669 10899 10545 10626 10362 10339 10275 10128 10023
    132    1 2021-05-01 12652 12074 11299 10942 11025 10733 10709 10681 10476 10290
    133    1 2021-05-01 12822 12365 11609 11277 11363 11066 11042 11014 10820 10413
    134    1 2021-05-01 12817 12140 11362 11024 11118 10823 10798 10719 10586 10366
    135    1 2021-05-01 12636 12185 11414 11065 11149 10847 10827 10768 10608 10270
    136    1 2021-05-01 12839 12421 11626 11240 11302 10972 10941 11000 10680 10339
    137    1 2021-05-01 12955 12469 11705 11339 11424 11107 11089 11063 10847 10426
    138    1 2021-05-01 12985 12489 11734 11375 11463 11151 11132 11100 10905 10528
    139    1 2021-05-01 12744 12246 11464 11086 11156 10834 10806 10796 10557 10283
    140    1 2021-05-01 12842 12369 11572 11197 11280 10948 10925 10907 10659 10422
    141    1 2021-05-01 12768 12179 11398 11036 11131 10824 10793 10738 10552 10368
    142    1 2021-05-01 12708 12226 11459 11111 11204 10894 10879 10809 10632 10262
    143    1 2021-05-01 10545 10085  9434  9171  9223  9025  9076  8967  8914  8098
    144    1 2021-05-01 10579  9971  9363  9116  9177  8986  9052  8885  8903  8179
    145    1 2021-05-01 10524 10088  9462  9198  9257  9063  9129  8962  8964  8088
    146    1 2021-05-01 10590 10356  9695  9396  9440  9230  9276  9244  9104  8117
    147    1 2021-05-01 11258 10916 10211  9879  9939  9698  9711  9686  9524  8838
    148    1 2021-05-01 11460 11112 10399 10057 10136  9879  9889  9832  9700  9039
    149    1 2021-05-01 11743 11653 10871 10499 10561 10283 10248 10280 10030  9364
    150    1 2021-05-01 12283 12148 11384 11039 11116 10834 10802 10803 10585  9954
    151    1 2021-05-01 12678 12339 11582 11251 11328 11037 11008 10991 10788 10314
    152    1 2021-05-01 12844 12317 11556 11208 11297 10981 10963 10920 10737 10431
    153    1 2021-05-01 12572 12122 11308 10903 10962 10638 10597 10647 10357 10219
    154    1 2021-05-01 13070 12501 11712 11325 11398 11070 11024 11061 10771 10589
    155    1 2021-05-01 12664 12188 11407 11030 11099 10790 10758 10740 10500 10225
    156    1 2021-05-01 12872 12314 11538 11174 11276 10964 10939 10871 10705 10461
    157    1 2021-05-01 12808 12309 11534 11159 11251 10935 10899 10884 10653 10442
    158    1 2021-05-01 12822 12231 11468 11111 11207 10901 10888 10821 10643 10528
    159    1 2021-05-01 12816 12342 11576 11207 11307 10997 10967 10946 10728 10565
    160    1 2021-05-01 10509 10161  9513  9243  9287  9094  9143  9045  8990  8011
    161    1 2021-05-01 10636 10246  9596  9329  9378  9179  9226  9129  9066  8219
    162    1 2021-05-01 10729 10283  9636  9348  9402  9202  9255  9145  9095  8337
    163    1 2021-05-01 10677 10160  9521  9241  9306  9109  9170  9027  9004  8275
    164    1 2021-05-01 10828 10595  9906  9577  9626  9402  9428  9407  9242  8422
    165    1 2021-05-01 11269 10955 10237  9894  9955  9716  9726  9705  9535  8823
    166    1 2021-05-01 12133 11513 10749 10391 10463 10193 10183 10155  9975  9716
    167    1 2021-05-01 12490 12082 11322 10979 11053 10760 10740 10771 10519 10064
    168    1 2021-05-01 12564 12065 11297 10926 10993 10702 10666 10623 10419 10142
    169    1 2021-05-01 12669 12125 11339 10963 11030 10724 10696 10666 10442 10301
    170    1 2021-05-01 12804 12341 11554 11170 11252 10930 10886 10885 10632 10424
    171    1 2021-05-01 12929 12408 11659 11291 11380 11068 11037 11005 10787 10492
    172    1 2021-05-01 12885 12293 11537 11164 11254 10945 10922 10884 10682 10478
    173    1 2021-05-01 12793 12343 11579 11205 11295 10973 10947 10943 10711 10459
    174    1 2021-05-01 12905 12382 11620 11252 11358 11043 11016 10984 10770 10638
    175    1 2021-05-01 12838 12223 11470 11116 11221 10915 10895 10846 10645 10614
    176    1 2021-05-01 12711 12019 11275 10929 11032 10737 10723 10655 10471 10505
    177    1 2021-05-01 12471 11976 11240 10893 10991 10696 10685 10642 10435 10306
    178    1 2021-05-01 10476 10038  9406  9147  9189  9015  9075  8919  8924  7930
    179    1 2021-05-01 10588 10163  9525  9260  9299  9113  9172  9046  9002  8075
    180    1 2021-05-01 10589 10174  9540  9282  9328  9145  9197  9087  9044  8140
    181    1 2021-05-01 10667 10283  9636  9348  9402  9202  9255  9145  9095  8245
    182    1 2021-05-01 10709 10307  9648  9342  9404  9198  9238  9142  9085  8321
    183    1 2021-05-01 10857 10645  9934  9590  9634  9411  9416  9433  9230  8449
    184    1 2021-05-01 11280 11061 10311  9958  9991  9753  9744  9810  9538  8792
    185    1 2021-05-01 11973 11895 11141 10807 10868 10590 10575 10596 10364  9550
    186    1 2021-05-01 12323 11922 11141 10773 10827 10522 10496 10485 10254  9836
    187    1 2021-05-01 12535 12110 11326 10963 11027 10725 10691 10658 10437 10133
    188    1 2021-05-01 12775 12280 11495 11139 11215 10910 10866 10838 10622 10450
    189    1 2021-05-01 12895 12460 11694 11337 11418 11112 11069 11059 10821 10508
    190    1 2021-05-01 12967 12439 11684 11351 11442 11129 11101 11057 10869 10570
    191    1 2021-05-01 12934 12385 11634 11289 11378 11066 11037 10988 10811 10553
    192    1 2021-05-01 12862 12385 11634 11289 11378 11066 11037 10988 10811 10536
    193    1 2021-05-01 12741 12320 11565 11205 11300 10989 10964 10921 10720 10410
    194    1 2021-05-01 12800 12343 11579 11205 11295 10973 10947 10943 10711 10471
    195    1 2021-05-01 12918 12417 11660 11299 11397 11075 11052 11040 10803 10626
    196    1 2021-05-01 12811 12303 11550 11189 11300 10990 10961 10916 10722 10615
    197    1 2021-05-01 12701 12168 11420 11069 11176 10876 10848 10793 10609 10554
    198    1 2021-05-01 12496 11908 11176 10834 10938 10647 10628 10569 10383 10367
    199    1 2021-05-01 12398 11775 11050 10712 10826 10544 10526 10475 10312 10298
    200    1 2021-05-01 10416 10085  9467  9210  9256  9071  9146  9016  8979  7857
    201    1 2021-05-01 10241 10014  9396  9144  9178  9010  9077  8963  8926  7654
    202    1 2021-05-01 10318  9907  9266  8981  9012  8830  8885  8831  8735  7778
    203    1 2021-05-01 10350 10199  9545  9257  9312  9108  9157  9081  8992  7850
    204    1 2021-05-01 10610 10185  9529  9239  9302  9107  9164  9045  8999  8202
    205    1 2021-05-01 10677 10347  9665  9341  9399  9187  9217  9155  9045  8216
    206    1 2021-05-01 11157 10495  9785  9451  9480  9270  9291  9316  9106  8655
    207    1 2021-05-01 11517 11555 10797 10459 10495 10231 10230 10299 10017  8969
    208    1 2021-05-01 12633 12110 11329 10974 11053 10742 10716 10668 10471 10277
    209    1 2021-05-01 12738 12251 11470 11118 11190 10888 10853 10827 10610 10382
    210    1 2021-05-01 12809 12381 11618 11266 11349 11037 11005 10993 10762 10387
    211    1 2021-05-01 12898 12399 11648 11315 11400 11089 11062 11029 10822 10511
    212    1 2021-05-01 12911 12427 11663 11338 11431 11125 11094 11043 10863 10564
    213    1 2021-05-01 12876 12328 11579 11226 11323 11005 10989 10928 10751 10543
    214    1 2021-05-01 12729 12245 11483 11128 11221 10908 10881 10848 10639 10443
    215    1 2021-05-01 12772 12345 11574 11224 11311 10995 10972 10970 10726 10519
    216    1 2021-05-01 12823 12355 11599 11251 11350 11042 11016 10992 10779 10608
    217    1 2021-05-01 12830 12271 11522 11168 11271 10969 10937 10901 10704 10632
    218    1 2021-05-01 12727 12184 11429 11086 11193 10893 10858 10824 10622 10610
    219    1 2021-05-01 12541 11939 11202 10863 10967 10674 10654 10611 10425 10445
    220    1 2021-05-01 12431 11939 11202 10863 10967 10674 10654 10611 10425 10370
    221    1 2021-05-01 12213 11645 10920 10597 10700 10434 10419 10374 10205 10123
    222    1 2021-05-01 12039 11602 10879 10566 10669 10404 10392 10339 10169  9960
    223    1 2021-05-01 10330  9929  9321  9068  9115  8932  9015  8853  8850  7768
    224    1 2021-05-01  9960  9631  9031  8782  8808  8655  8739  8618  8590  7348
    225    1 2021-05-01 10255  9694  9064  8777  8806  8632  8681  8609  8521  7697
    226    1 2021-05-01 10430 10074  9410  9111  9155  8954  8999  8950  8827  7952
    227    1 2021-05-01 10597 10138  9487  9196  9246  9054  9107  9016  8945  8128
    228    1 2021-05-01 10554 10102  9449  9154  9201  9013  9064  8983  8900  8067
    229    1 2021-05-01 10486 10198  9536  9247  9277  9089  9143  9134  8968  7864
    230    1 2021-05-01 10894 10881 10198  9890  9923  9685  9719  9759  9541  8194
    231    1 2021-05-01 11208 10443  9781  9473  9503  9292  9333  9277  9155  8403
    232    1 2021-05-01 12559 12145 11379 11024 11096 10790 10753 10742 10509 10135
    233    1 2021-05-01 12632 12089 11324 10959 11036 10733 10711 10673 10474 10286
    234    1 2021-05-01 12651 12276 11511 11153 11227 10913 10880 10903 10620 10267
    235    1 2021-05-01 12857 12407 11653 11336 11420 11113 11092 11053 10859 10465
    236    1 2021-05-01 12812 12265 11526 11187 11275 10977 10955 10911 10724 10436
    237    1 2021-05-01 12726 12209 11447 11083 11164 10863 10828 10815 10581 10398
    238    1 2021-05-01 12709 12209 11447 11083 11164 10863 10828 10815 10581 10406
    239    1 2021-05-01 12640 12309 11548 11197 11289 10983 10957 10945 10712 10344
    240    1 2021-05-01 12759 12330 11571 11224 11316 11006 10978 10989 10740 10545
    241    1 2021-05-01 12768 12317 11569 11207 11312 11006 10970 10960 10738 10608
    242    1 2021-05-01 12746 12210 11466 11113 11225 10923 10880 10863 10654 10620
    243    1 2021-05-01 12651 12025 11272 10932 11038 10748 10710 10674 10477 10570
    244    1 2021-05-01 12366 11854 11114 10770 10873 10603 10568 10536 10341 10313
    245    1 2021-05-01 12186 11693 10966 10633 10738 10468 10445 10423 10224 10121
    246    1 2021-05-01 12198 11707 10978 10653 10764 10487 10465 10459 10252 10160
    247    1 2021-05-01 12184 11649 10912 10582 10681 10418 10388 10404 10167 10228
    248    1 2021-05-01 12107 11643 10925 10617 10731 10470 10443 10435 10229 10178
    249    1 2021-05-01 10423  9972  9370  9110  9162  8980  9047  8885  8896  7847
    250    1 2021-05-01 10289  9835  9214  8923  8974  8790  8848  8692  8686  7718
    251    1 2021-05-01 10023  9806  9177  8884  8930  8741  8791  8697  8640  7441
    252    1 2021-05-01 10394 10048  9402  9112  9163  8979  9024  8918  8870  7851
    253    1 2021-05-01 10450 10019  9372  9081  9125  8943  9004  8908  8834  7947
    254    1 2021-05-01 10450  9938  9296  9008  9055  8870  8926  8829  8765  7909
    255    1 2021-05-01 10296  9749  9122  8853  8888  8722  8789  8701  8645  7721
    256    1 2021-05-01 10177  9749  9122  8853  8888  8722  8789  8701  8645  7505
    257    1 2021-05-01 10440 10079  9442  9160  9190  9000  9062  9028  8901  7644
    258    1 2021-05-01 10789 10324  9653  9344  9377  9171  9215  9142  9041  7962
    259    1 2021-05-01 11467 10986 10222  9857  9896  9648  9649  9625  9435  8793
    260    1 2021-05-01 12707 12194 11439 11078 11169 10859 10842 10785 10604 10387
    261    1 2021-05-01 12668 12174 11414 11058 11146 10850 10825 10770 10588 10360
    262    1 2021-05-01 12769 12336 11594 11261 11342 11032 11014 11002 10771 10453
    263    1 2021-05-01 12578 12192 11465 11149 11240 10941 10935 10870 10712 10229
    264    1 2021-05-01 12533 12083 11329 10972 11057 10755 10734 10700 10490 10262
    265    1 2021-05-01 12482 11963 11209 10843 10930 10635 10608 10582 10377 10238
    266    1 2021-05-01 12474 12131 11377 11024 11108 10807 10778 10801 10543 10215
    267    1 2021-05-01 12538 12037 11302 10952 11038 10742 10733 10745 10493 10284
    268    1 2021-05-01 12604 12152 11406 11048 11140 10837 10804 10831 10567 10451
    269    1 2021-05-01 12494 11972 11228 10868 10964 10668 10638 10651 10400 10363
    270    1 2021-05-01 12515 11891 11147 10785 10892 10606 10570 10574 10335 10439
    271    1 2021-05-01 12416 11622 10902 10549 10649 10375 10349 10344 10116 10370
    272    1 2021-05-01 12131 11699 10984 10645 10750 10484 10459 10433 10230 10092
    273    1 2021-05-01 12081 11707 10978 10653 10764 10487 10465 10459 10252 10100
    274    1 2021-05-01 12054 11592 10864 10539 10648 10391 10364 10348 10153 10106
    275    1 2021-05-01 12025 11592 10867 10544 10650 10373 10346 10389 10118 10092
    276    1 2021-05-01 11624 11357 10667 10376 10485 10240 10228 10222 10015  9727
    277    1 2021-05-01 11423 11098 10404 10106 10214  9974  9969  9953  9778  9524
    278    1 2021-05-01 10545  9941  9301  9002  9054  8866  8923  8780  8767  8008
    279    1 2021-05-01 10453  9941  9301  9002  9054  8866  8923  8780  8767  7948
    280    1 2021-05-01 10623  9981  9330  9025  9072  8869  8911  8811  8745  8186
    281    1 2021-05-01 10508 10055  9419  9135  9189  9003  9049  8925  8887  8037
    282    1 2021-05-01 10459 10003  9364  9074  9115  8927  8983  8872  8808  7946
    283    1 2021-05-01 10574 10033  9394  9111  9155  8965  9010  8880  8844  8072
    284    1 2021-05-01 10319  9679  9086  8844  8900  8737  8816  8641  8681  7731
    285    1 2021-05-01 10093 10039  9414  9153  9203  9020  9089  8936  8928  7356
    286    1 2021-05-01 10483 10422  9742  9436  9474  9270  9308  9231  9122  7693
    287    1 2021-05-01 11393 10998 10254  9917  9958  9718  9724  9692  9515  8753
    288    1 2021-05-01 11937 11656 10857 10487 10545 10263 10248 10251 10012  9361
    289    1 2021-05-01 12568 12238 11446 11083 11160 10868 10830 10796 10595 10170
    290    1 2021-05-01 12913 12456 11716 11366 11459 11158 11132 11112 10891 10569
    291    1 2021-05-01 12822 12190 11478 11171 11268 10975 10971 10906 10748 10499
    292    1 2021-05-01 12564 11902 11177 10829 10918 10641 10617 10572 10395 10215
    293    1 2021-05-01 12421 11905 11154 10788 10877 10572 10545 10530 10318 10201
    294    1 2021-05-01 12363 11822 11086 10729 10824 10534 10507 10471 10281 10162
    295    1 2021-05-01 12337 11933 11182 10822 10904 10615 10579 10583 10348 10115
    296    1 2021-05-01 12281 11765 11036 10684 10774 10498 10471 10458 10241 10045
    297    1 2021-05-01 12091 11832 11093 10733 10811 10517 10500 10539 10258  9874
    298    1 2021-05-01 12230 11742 11015 10659 10748 10460 10435 10437 10195 10142
    299    1 2021-05-01 12218 11743 11010 10652 10759 10473 10450 10432 10218 10179
    300    1 2021-05-01 12116 11564 10847 10500 10604 10334 10308 10263 10086 10070
    301    1 2021-05-01 11955 11471 10760 10411 10506 10239 10219 10239  9991  9935
    302    1 2021-05-01 11873 11358 10655 10334 10442 10184 10169 10170  9969  9944
    303    1 2021-05-01 11692 11181 10486 10193 10298 10057 10048 10030  9839  9767
    304    1 2021-05-01 11533 10696 10040  9758  9860  9647  9654  9573  9448  9618
    305    1 2021-05-01 10683 10207  9532  9239  9291  9082  9129  9029  8972  8207
    306    1 2021-05-01 10617 10153  9502  9213  9265  9074  9117  9006  8955  8167
    307    1 2021-05-01 10543 10127  9475  9184  9222  9029  9083  8977  8919  8030
    308    1 2021-05-01 10661 10236  9558  9257  9307  9089  9129  9026  8956  8135
    309    1 2021-05-01 10744 10417  9724  9418  9472  9253  9278  9216  9091  8266
    310    1 2021-05-01 10931 10526  9867  9589  9646  9450  9488  9374  9309  8389
    311    1 2021-05-01 10990 10664  9977  9683  9732  9521  9555  9441  9364  8381
    312    1 2021-05-01 11382 10664  9977  9683  9732  9521  9555  9441  9364  8787
    313    1 2021-05-01 11513 10943 10222  9904  9956  9723  9741  9659  9545  8889
    314    1 2021-05-01 11742 11435 10650 10285 10345 10075 10066 10047  9836  9149
    315    1 2021-05-01 12848 12269 11459 11079 11158 10850 10800 10839 10561 10463
    316    1 2021-05-01 12936 12469 11673 11309 11388 11073 11029 11049 10798 10540
    317    1 2021-05-01 12482 12204 11494 11174 11264 10973 10955 10921 10730 10140
    318    1 2021-05-01 12316 11846 11130 10790 10874 10598 10579 10534 10363 10016
    319    1 2021-05-01 12313 11827 11082 10718 10814 10519 10497 10454 10272 10077
    320    1 2021-05-01 12362 11869 11136 10785 10891 10604 10584 10526 10354 10177
    321    1 2021-05-01 12282 11758 11028 10673 10770 10497 10465 10424 10238 10174
    322    1 2021-05-01 12212 11687 10977 10630 10727 10454 10432 10394 10214 10091
    323    1 2021-05-01 12111 11655 10931 10583 10674 10400 10368 10351 10149 10000
    324    1 2021-05-01 12185 11666 10949 10601 10697 10413 10378 10382 10161 10103
    325    1 2021-05-01 12169 11618 10896 10559 10667 10393 10367 10330 10155 10178
    326    1 2021-05-01 12056 11421 10716 10371 10478 10218 10195 10148  9980 10091
    327    1 2021-05-01 11880 11222 10526 10187 10282 10027 10008 10014  9800  9907
    328    1 2021-05-01 11325 11155 10467 10148 10248  9999  9995  9998  9789  9374
    329    1 2021-05-01 11262 11021 10330 10025 10125  9891  9886  9860  9686  9326
    330    1 2021-05-01 11130 10593  9921  9624  9721  9502  9506  9425  9296  9199
    331    1 2021-05-01 10430 10056  9430  9167  9190  9016  9094  8963  8947  7632
    332    1 2021-05-01 10506 10036  9392  9112  9136  8957  9026  8921  8860  7863
    333    1 2021-05-01 10447 10089  9437  9158  9203  9007  9071  8976  8917  7845
    334    1 2021-05-01 10497  9925  9284  9006  9043  8858  8916  8842  8761  7975
    335    1 2021-05-01 10522 10104  9452  9161  9209  9012  9055  8978  8899  8054
    336    1 2021-05-01 10612 10022  9364  9072  9104  8917  8962  8908  8810  8075
    337    1 2021-05-01 10755 10319  9625  9319  9360  9156  9193  9112  9014  8211
    338    1 2021-05-01 10888 10417  9724  9418  9472  9253  9278  9216  9091  8386
    339    1 2021-05-01 10993 10599  9915  9615  9660  9453  9481  9453  9284  8472
    340    1 2021-05-01 11250 10931 10221  9921  9963  9732  9755  9711  9559  8661
    341    1 2021-05-01 11550 11152 10427 10129 10176  9946  9958  9909  9766  8923
    342    1 2021-05-01 11793 11371 10610 10266 10336 10084 10087 10001  9879  9183
    343    1 2021-05-01 12145 11905 11085 10706 10764 10467 10441 10483 10206  9625
    344    1 2021-05-01 12577 12307 11495 11124 11212 10896 10861 10871 10619 10114
    345    1 2021-05-01 12906 12403 11622 11262 11336 11017 10981 10993 10739 10466
    346    1 2021-05-01 12908 12312 11533 11175 11259 10934 10899 10904 10657 10508
    347    1 2021-05-01 12249 11824 11097 10743 10843 10550 10540 10519 10310 10011
    348    1 2021-05-01 12381 11737 11015 10666 10764 10482 10455 10461 10222 10216
    349    1 2021-05-01 12289 11692 10972 10627 10725 10452 10438 10418 10218 10156
    350    1 2021-05-01 12079 11692 10972 10627 10725 10452 10438 10418 10218  9980
    351    1 2021-05-01 12107 11655 10940 10591 10688 10414 10389 10368 10176 10066
    352    1 2021-05-01 12091 11666 10949 10601 10697 10413 10378 10382 10161 10090
    353    1 2021-05-01 12038 11572 10849 10511 10612 10331 10312 10300 10087 10068
    354    1 2021-05-01 11837 11452 10736 10401 10508 10238 10222 10188 10007  9902
    355    1 2021-05-01 11619 11137 10443 10099 10201  9945  9932  9903  9724  9669
    356    1 2021-05-01 11166 10627  9957  9630  9727  9502  9500  9466  9305  9200
    357    1 2021-05-01 11008 10499  9829  9520  9614  9391  9395  9322  9197  9069
    358    1 2021-05-01 10799 10146  9489  9199  9299  9090  9105  8981  8917  8854
    359    1 2021-05-01 10483  9785  9119  8813  8903  8695  8706  8611  8519  8495
    360    1 2021-05-01 11441 11023 10390 10147 10201  9991 10047  9867  9887  8762
    361    1 2021-05-01 10956 10629 10022  9784  9834  9646  9705  9491  9561  8245
    362    1 2021-05-01 10480 10005  9414  9175  9213  9051  9133  8913  8984  7728
    363    1 2021-05-01 10269  9822  9220  8962  8991  8834  8910  8739  8751  7529
    364    1 2021-05-01 10096  9893  9253  8969  8993  8813  8884  8800  8723  7433
    365    1 2021-05-01 10117  9645  9031  8756  8773  8610  8691  8592  8536  7531
    366    1 2021-05-01 10224  9772  9134  8860  8887  8714  8772  8704  8623  7723
    367    1 2021-05-01 10196 10115  9424  9113  9135  8943  8978  8965  8807  7685
    368    1 2021-05-01 10505 10181  9481  9173  9206  8999  9044  9026  8867  7920
    369    1 2021-05-01 10806 10389  9698  9398  9450  9239  9271  9212  9087  8285
    370    1 2021-05-01 10729 10343  9662  9361  9403  9202  9241  9203  9072  8190
    371    1 2021-05-01 10877 10959 10239  9922  9953  9721  9737  9759  9536  8292
    372    1 2021-05-01 11375 10999 10286  9967  9993  9767  9796  9791  9594  8722
    373    1 2021-05-01 11770 11323 10585 10256 10308 10067 10079 10021  9872  9023
    374    1 2021-05-01 11920 11524 10729 10361 10397 10129 10129 10154  9906  9207
    375    1 2021-05-01 12441 12078 11267 10894 10971 10668 10637 10645 10401  9951
    376    1 2021-05-01 12743 12104 11282 10886 10936 10622 10575 10661 10336 10260
    377    1 2021-05-01 12630 12250 11485 11144 11229 10906 10879 10850 10653 10121
    378    1 2021-05-01 12611 12097 11335 10992 11066 10751 10718 10693 10488 10168
    379    1 2021-05-01 12008 11568 10859 10526 10621 10350 10339 10316 10109  9883
    380    1 2021-05-01 11971 11557 10838 10484 10580 10307 10279 10291 10060  9868
    381    1 2021-05-01 11938 11465 10741 10390 10490 10217 10185 10196  9963  9924
    382    1 2021-05-01 11854 11479 10754 10406 10509 10235 10207 10201  9993  9872
    383    1 2021-05-01 11884 11303 10590 10255 10361 10098 10075 10051  9865  9939
    384    1 2021-05-01 11797 11115 10423 10098 10198  9952  9935  9887  9729  9870
    385    1 2021-05-01 11354 10714 10047  9724  9821  9598  9592  9523  9393  9387
    386    1 2021-05-01 10985 10449  9787  9469  9560  9344  9344  9316  9161  9003
    387    1 2021-05-01 10864 10449  9787  9469  9560  9344  9344  9316  9161  8881
    388    1 2021-05-01 10939 10433  9756  9437  9531  9310  9306  9253  9113  8998
    389    1 2021-05-01 10822 10187  9538  9244  9342  9133  9139  9035  8939  8879
    390    1 2021-05-01 10366  9826  9171  8881  8973  8776  8795  8665  8609  8343
    391    1 2021-05-01 10287  9713  9029  8716  8798  8595  8603  8509  8415  8246
    392    1 2021-05-01 10187  9615  8912  8584  8660  8452  8453  8375  8265  8125
    393    1 2021-05-01 11835 11339 10666 10390 10435 10196 10243 10111 10077  9052
    394    1 2021-05-01 11567 10858 10225  9978 10018  9823  9877  9673  9728  8807
    395    1 2021-05-01 11001 10622  9973  9693  9736  9536  9588  9398  9429  8350
    396    1 2021-05-01 10969 10172  9555  9291  9348  9163  9233  9000  9068  8416
    397    1 2021-05-01 10618 10123  9494  9217  9272  9083  9155  8961  8997  8037
    398    1 2021-05-01 10232  9574  8997  8752  8789  8646  8739  8531  8599  7587
    399    1 2021-05-01  9827  9523  8898  8622  8646  8483  8545  8471  8405  7200
    400    1 2021-05-01  9797  9415  8790  8516  8542  8376  8452  8383  8308  7181
    401    1 2021-05-01  9890  9792  9128  8829  8854  8676  8729  8685  8576  7328
    402    1 2021-05-01 10148  9741  9070  8761  8784  8602  8662  8640  8495  7578
    403    1 2021-05-01 10103 10206  9508  9198  9227  9023  9069  9052  8889  7490
    404    1 2021-05-01 10507 10156  9484  9189  9219  9027  9080  9016  8908  7894
    405    1 2021-05-01 10642 10443  9749  9435  9461  9246  9288  9295  9106  7964
    406    1 2021-05-01 11018 10392  9723  9421  9441  9246  9309  9261  9125  8245
    407    1 2021-05-01 10749 10851 10115  9772  9778  9548  9587  9611  9375  7828
    408    1 2021-05-01 11764 11307 10507 10117 10140  9863  9846  9919  9609  9031
    409    1 2021-05-01 12262 11781 10979 10598 10663 10368 10336 10334 10105  9760
    410    1 2021-05-01 12432 11846 11049 10661 10720 10411 10373 10400 10136  9952
    411    1 2021-05-01 12416 11948 11176 10819 10889 10585 10551 10526 10318  9949
    412    1 2021-05-01 11776 11298 10587 10235 10325 10055 10036 10039  9811  9693
    413    1 2021-05-01 11699 11298 10587 10235 10325 10055 10036 10039  9811  9650
    414    1 2021-05-01 11655 11346 10619 10265 10358 10090 10060 10075  9838  9685
    415    1 2021-05-01 11671 11210 10495 10149 10245  9979  9958  9954  9740  9731
    416    1 2021-05-01 11512 11045 10351 10026 10136  9884  9864  9827  9660  9590
    417    1 2021-05-01 11225 10620  9954  9632  9745  9508  9507  9420  9313  9296
    418    1 2021-05-01 10688 10316  9667  9354  9446  9238  9237  9189  9048  8721
    419    1 2021-05-01 10668 10152  9483  9165  9256  9038  9036  8996  8832  8690
    420    1 2021-05-01 10596 10137  9473  9160  9253  9045  9045  8992  8842  8632
    421    1 2021-05-01 10500  9717  9083  8792  8886  8696  8723  8627  8537  8557
    422    1 2021-05-01 10328  9609  8933  8615  8688  8481  8498  8447  8302  8328
    423    1 2021-05-01 10184  9606  8907  8577  8653  8446  8443  8366  8258  8118
    424    1 2021-05-01 10156  9570  8865  8530  8606  8393  8390  8323  8205  8099
    425    1 2021-05-01 10128  9695  8949  8580  8634  8404  8386  8405  8180  8043
    426    1 2021-05-01 11957 11400 10717 10435 10483 10240 10282 10137 10103  9138
    427    1 2021-05-01 11678 11105 10436 10166 10213  9990 10041  9849  9867  8852
    428    1 2021-05-01 11368 10835 10178  9891  9931  9727  9769  9609  9600  8523
    429    1 2021-05-01 11204 10835 10178  9891  9931  9727  9769  9609  9600  8505
    430    1 2021-05-01 11070 10540  9886  9596  9653  9437  9487  9319  9330  8554
    431    1 2021-05-01 10724 10123  9494  9217  9272  9083  9155  8961  8997  8203
    432    1 2021-05-01 10314  9873  9275  9024  9069  8914  8987  8792  8841  7705
    433    1 2021-05-01  9896  9383  8777  8518  8547  8408  8494  8325  8358  7243
    434    1 2021-05-01  9724  9229  8633  8375  8399  8254  8330  8183  8191  7040
    435    1 2021-05-01  9499  9281  8652  8366  8382  8226  8307  8256  8161  6911
    436    1 2021-05-01  9881  9555  8902  8610  8634  8456  8527  8470  8371  7240
    437    1 2021-05-01 10120  9904  9244  8953  8980  8795  8850  8794  8685  7433
    438    1 2021-05-01 10294  9793  9169  8892  8919  8747  8819  8717  8660  7607
    439    1 2021-05-01 10335  9950  9313  9027  9048  8869  8943  8852  8766  7516
    440    1 2021-05-01 10238  9929  9289  8999  9013  8827  8904  8795  8729  7312
    441    1 2021-05-01 10468 10849 10102  9746  9757  9519  9529  9523  9301  7410
    442    1 2021-05-01 11388 10849 10102  9746  9757  9519  9529  9523  9301  8547
    443    1 2021-05-01 12132 11540 10742 10357 10399 10110 10085 10107  9838  9572
    444    1 2021-05-01 12331 11846 11049 10661 10720 10411 10373 10400 10136  9848
    445    1 2021-05-01 12497 11934 11156 10795 10879 10572 10552 10499 10304 10110
    446    1 2021-05-01 11549 11088 10379 10027 10126  9864  9845  9829  9627  9562
    447    1 2021-05-01 11448 10907 10224  9886  9987  9741  9731  9678  9518  9529
    448    1 2021-05-01 11238 10608  9936  9617  9723  9494  9496  9398  9298  9318
    449    1 2021-05-01 10965 10240  9584  9271  9376  9163  9170  9081  8987  9023
    450    1 2021-05-01 10688 10089  9434  9104  9200  8982  8982  8951  8794  8732
    451    1 2021-05-01 10599  9940  9279  8947  9032  8814  8811  8796  8613  8626
    452    1 2021-05-01 10392  9667  9051  8765  8866  8675  8701  8566  8527  8432
    453    1 2021-05-01 10049  9717  9083  8792  8886  8696  8723  8627  8537  8096
    454    1 2021-05-01  9879  9329  8677  8376  8451  8270  8300  8189  8123  7837
    455    1 2021-05-01  9946  9512  8815  8474  8543  8326  8323  8285  8132  7876
    456    1 2021-05-01 10134  9570  8876  8541  8619  8402  8403  8336  8215  8092
    457    1 2021-05-01 10020  9496  8784  8442  8506  8294  8293  8230  8101  7924
    458    1 2021-05-01 10066  9412  8678  8315  8368  8147  8141  8109  7929  7955
    459    1 2021-05-01 10022  9490  8750  8382  8444  8209  8198  8178  7985  7909
    460    1 2021-05-01 12052 11546 10835 10516 10555 10314 10337 10174 10152  9171
    461    1 2021-05-01 12067 11609 10893 10581 10619 10370 10391 10263 10199  9178
    462    1 2021-05-01 12027 11512 10825 10553 10598 10348 10386 10229 10204  9155
    463    1 2021-05-01 11644 11297 10622 10340 10383 10150 10184  9999 10001  8824
    464    1 2021-05-01 11431 10987 10307 10018 10056  9839  9875  9714  9691  8645
    465    1 2021-05-01 11417 10782 10117  9823  9874  9660  9691  9545  9528  8724
    466    1 2021-05-01 11242 10607  9963  9680  9737  9544  9593  9419  9437  8659
    467    1 2021-05-01 10549 10027  9426  9172  9227  9063  9148  8938  9004  7928
    468    1 2021-05-01 10080  9290  8775  8574  8620  8499  8622  8330  8519  7398
    469    1 2021-05-01  9318  9176  8567  8294  8295  8154  8238  8139  8114  6570
    470    1 2021-05-01  9670  9449  8823  8556  8583  8426  8505  8369  8371  6945
    471    1 2021-05-01  9678  9514  8892  8619  8654  8498  8568  8451  8417  7062
    472    1 2021-05-01  9959  9514  8892  8619  8654  8498  8568  8451  8417  7246
    473    1 2021-05-01 10089  9779  9121  8824  8849  8661  8729  8656  8560  7311
    474    1 2021-05-01 10218  9825  9190  8904  8931  8753  8812  8700  8642  7464
    475    1 2021-05-01 10201  9729  9115  8846  8871  8710  8792  8645  8621  7403
    476    1 2021-05-01 10488  9989  9347  9066  9090  8905  8972  8835  8798  7571
    477    1 2021-05-01 10883 10547  9826  9485  9490  9262  9297  9267  9082  7971
    478    1 2021-05-01 11508 11280 10504 10134 10156  9886  9875  9882  9641  8724
    479    1 2021-05-01 11879 11641 10854 10478 10529 10232 10204 10214  9972  9291
    480    1 2021-05-01 11217 10627  9950  9630  9726  9501  9492  9422  9293  9270
    481    1 2021-05-01 10556 10173  9526  9217  9319  9113  9116  9024  8920  8620
    482    1 2021-05-01 10392  9931  9278  8952  9039  8829  8840  8813  8654  8434
    483    1 2021-05-01 10048  9400  8774  8486  8588  8408  8439  8256  8273  8033
    484    1 2021-05-01 10029  9498  8818  8493  8572  8372  8379  8293  8187  7996
    485    1 2021-05-01 10149  9594  8904  8585  8665  8459  8458  8377  8274  8115
    486    1 2021-05-01 10167  9439  8770  8464  8549  8361  8367  8220  8196  8130
    487    1 2021-05-01  9891  9266  8554  8206  8275  8065  8062  7968  7878  7816
    488    1 2021-05-01  9824  9157  8447  8088  8153  7943  7948  7872  7760  7741
    489    1 2021-05-01  9790  9137  8414  8040  8095  7878  7872  7853  7666  7677
    490    1 2021-05-01  9755  9137  8414  8040  8095  7878  7872  7853  7666  7619
    491    1 2021-05-01  9643  9347  8608  8221  8273  8038  8019  8097  7794  7524
    492    1 2021-05-01 11942 11558 10849 10545 10572 10326 10368 10219 10183  8749
    493    1 2021-05-01 11968 11464 10752 10452 10476 10233 10266 10144 10075  8979
    494    1 2021-05-01 11911 11406 10708 10409 10448 10213 10245 10095 10063  8974
    495    1 2021-05-01 11798 11167 10467 10172 10213  9975 10011  9850  9829  8934
    496    1 2021-05-01 11641 11079 10383 10078 10123  9888  9925  9788  9749  8859
    497    1 2021-05-01 11458 10794 10124  9831  9890  9683  9730  9566  9561  8735
    498    1 2021-05-01 11189 10794 10124  9831  9890  9683  9730  9566  9561  8534
    499    1 2021-05-01 10899 10227  9635  9388  9443  9279  9353  9135  9212  8223
    500    1 2021-05-01 10515  9874  9314  9079  9131  8975  9050  8797  8916  7810
    501    1 2021-05-01  9812  9074  8531  8310  8338  8218  8344  8097  8232  7067
    502    1 2021-05-01  9934  9599  8982  8716  8743  8589  8670  8509  8527  7205
    503    1 2021-05-01 10209  9649  9022  8757  8795  8623  8700  8560  8553  7522
    504    1 2021-05-01 10062  9687  9078  8814  8850  8682  8761  8609  8619  7363
    505    1 2021-05-01 10298  9893  9232  8925  8944  8757  8809  8718  8634  7521
    506    1 2021-05-01 10466 10295  9638  9340  9374  9165  9207  9095  9020  7662
    507    1 2021-05-01 10744 10293  9646  9372  9411  9222  9281  9098  9099  7926
    508    1 2021-05-01 10694 10364  9679  9371  9390  9181  9234  9115  9053  7849
    509    1 2021-05-01 11278 10991 10240  9881  9906  9642  9652  9635  9436  8479
    510    1 2021-05-01 11898 11661 10887 10526 10585 10295 10269 10248 10036  9278
    511    1 2021-05-01 10534  9756  9132  8832  8930  8738  8760  8656  8580  8598
    512    1 2021-05-01 10046  9474  8803  8476  8549  8353  8357  8302  8172  7998
    513    1 2021-05-01 10130  9601  8917  8593  8673  8467  8469  8406  8264  8124
    514    1 2021-05-01 10073  9478  8818  8506  8590  8397  8400  8293  8212  8060
    515    1 2021-05-01  9963  9271  8591  8270  8358  8159  8164  8008  7988  7934
    516    1 2021-05-01  9666  9096  8422  8105  8189  7992  8011  7847  7837  7592
    517    1 2021-05-01  9463  8920  8226  7866  7928  7726  7739  7661  7543  7320
    518    1 2021-05-01  9425  8818  8124  7751  7797  7595  7607  7590  7403  7249
    519    1 2021-05-01  9453  9174  8448  8060  8107  7881  7866  7928  7657  7295
    520    1 2021-05-01  9732  9628  8855  8451  8490  8238  8199  8352  7972  7638
    521    1 2021-05-01 10057  9596  8818  8405  8438  8181  8133  8296  7896  7974
    522    1 2021-05-01  8391  7962  7337  7008  7046  6889  6916  6858  6752  6360
    523    1 2021-05-01  8184  7773  7171  6841  6874  6741  6766  6738  6604  6166
    524    1 2021-05-01  8032  7927  7312  6984  7011  6863  6896  6839  6726  5991
    525    1 2021-05-01 11837 11272 10579 10282 10282 10052 10121  9966  9946  8496
    526    1 2021-05-01 11830 11433 10737 10443 10454 10215 10278 10117 10106  8520
    527    1 2021-05-01 11733 11464 10752 10452 10476 10233 10266 10144 10075  8666
    528    1 2021-05-01 11820 11384 10681 10377 10402 10163 10201 10078 10007  8779
    529    1 2021-05-01 11683 11256 10556 10256 10284 10054 10092  9941  9924  8655
    530    1 2021-05-01 11583 11095 10390 10077 10104  9872  9914  9787  9735  8718
    531    1 2021-05-01 11494 10867 10185  9888  9941  9725  9782  9610  9602  8699
    532    1 2021-05-01 11035 10617  9932  9631  9677  9470  9512  9337  9341  8320
    533    1 2021-05-01 10960 10320  9664  9370  9405  9212  9261  9111  9094  8247
    534    1 2021-05-01 10775 10285  9637  9353  9383  9194  9255  9123  9095  8023
    535    1 2021-05-01 10472 10006  9368  9098  9136  8961  9047  8843  8908  7709
    536    1 2021-05-01 10575 10092  9437  9145  9175  8989  9055  8937  8907  7874
    537    1 2021-05-01 10496  9914  9308  9048  9086  8919  9000  8820  8861  7793
    538    1 2021-05-01 10307 10230  9552  9253  9273  9062  9124  9028  8952  7565
    539    1 2021-05-01 10476 10049  9376  9072  9092  8893  8946  8845  8764  7707
    540    1 2021-05-01 11268 10295  9638  9340  9374  9165  9207  9095  9020  8513
    541    1 2021-05-01 11274 10657  9996  9707  9749  9531  9574  9438  9381  8448
    542    1 2021-05-01 11226 10606  9935  9644  9681  9472  9517  9345  9322  8411
    543    1 2021-05-01 11511 11063 10338 10003 10037  9787  9801  9723  9589  8711
    544    1 2021-05-01 12352 11656 10883 10520 10580 10286 10268 10254 10034  9859
    545    1 2021-05-01 12528 11983 11201 10842 10916 10615 10579 10558 10340 10169
    546    1 2021-05-01 12670 12149 11364 11004 11080 10784 10739 10717 10501 10365
    547    1 2021-05-01 10038  9424  8742  8420  8499  8299  8306  8192  8113  8010
    548    1 2021-05-01  9915  9183  8511  8201  8279  8089  8095  7940  7913  7868
    549    1 2021-05-01  9719  8897  8263  7968  8049  7875  7904  7714  7729  7656
    550    1 2021-05-01  9425  8701  8033  7692  7755  7578  7601  7491  7412  7310
    551    1 2021-05-01  9275  8883  8196  7838  7886  7688  7695  7673  7496  7172
    552    1 2021-05-01  9375  8883  8196  7838  7886  7688  7695  7673  7496  7277
    553    1 2021-05-01  9472  9154  8421  8036  8074  7858  7844  7898  7636  7368
    554    1 2021-05-01  9391  9055  8309  7905  7937  7716  7692  7762  7487  7271
    555    1 2021-05-01  9491  9425  8660  8257  8294  8052  8009  8111  7777  7358
    556    1 2021-05-01  8371  7874  7234  6897  6919  6756  6779  6763  6595  6313
    557    1 2021-05-01  8396  7699  7064  6729  6745  6603  6617  6596  6445  6357
    558    1 2021-05-01  8091  7588  6996  6680  6714  6586  6618  6508  6453  6056
    559    1 2021-05-01  8174  7640  7005  6669  6696  6556  6577  6463  6414  6104
    560    1 2021-05-01  8186  7677  7021  6667  6693  6537  6558  6442  6381  6105
    561    1 2021-05-01 11631 11190 10509 10229 10223 10005 10073  9910  9904  8211
    562    1 2021-05-01 11661 11244 10540 10234 10245 10012 10064  9926  9876  8413
    563    1 2021-05-01 11610 11182 10488 10198 10201  9981 10044  9905  9858  8345
    564    1 2021-05-01 11523 11018 10331 10024 10058  9837  9891  9725  9719  8483
    565    1 2021-05-01 11412 10871 10173  9859  9896  9670  9726  9574  9549  8485
    566    1 2021-05-01 11331 10778 10075  9763  9798  9581  9612  9483  9437  8486
    567    1 2021-05-01 11272 10653  9955  9653  9690  9471  9508  9364  9354  8488
    568    1 2021-05-01 10979 10498  9811  9510  9549  9337  9382  9224  9215  8232
    569    1 2021-05-01 10745 10346  9679  9385  9413  9211  9273  9124  9104  7937
    570    1 2021-05-01 10673 10251  9593  9316  9345  9159  9239  9086  9089  7810
    571    1 2021-05-01 10665 10157  9496  9210  9237  9061  9133  8979  8989  7913
    572    1 2021-05-01 10432 10047  9412  9134  9152  8973  9056  8941  8907  7605
    573    1 2021-05-01 10701 10230  9552  9253  9273  9062  9124  9028  8952  7882
    574    1 2021-05-01 11070 10760 10047  9722  9739  9504  9539  9496  9349  8236
    575    1 2021-05-01 11434 10938 10257  9951  9987  9754  9786  9682  9591  8654
    576    1 2021-05-01 11552 11065 10384 10088 10128  9898  9932  9797  9737  8712
    577    1 2021-05-01 11586 11165 10431 10095 10126  9873  9877  9825  9667  8772
    578    1 2021-05-01 11854 11682 10913 10556 10596 10313 10284 10299 10058  9220
    579    1 2021-05-01 12337 12037 11267 10915 10981 10693 10666 10639 10430  9811
    580    1 2021-05-01 12560 12226 11443 11085 11148 10842 10813 10830 10567 10141
    581    1 2021-05-01 12820 12446 11677 11325 11408 11096 11070 11050 10822 10382
    582    1 2021-05-01 12969 12446 11677 11325 11408 11096 11070 11050 10822 10596
    583    1 2021-05-01 12960 12420 11661 11328 11416 11117 11085 11037 10853 10615
    584    1 2021-05-01  9634  8897  8263  7968  8049  7875  7904  7714  7729  7597
    585    1 2021-05-01  9521  8797  8163  7854  7943  7778  7795  7610  7620  7467
    586    1 2021-05-01  9363  8747  8086  7745  7812  7632  7635  7543  7445  7307
    587    1 2021-05-01  9279  8696  8057  7732  7800  7636  7643  7515  7470  7214
    588    1 2021-05-01  9248  8700  8029  7677  7735  7545  7556  7477  7363  7188
    589    1 2021-05-01  9188  8508  7821  7468  7519  7338  7351  7222  7160  7075
    590    1 2021-05-01  9173  8543  7817  7422  7451  7238  7217  7219  7011  7030
    591    1 2021-05-01  8969  8306  7628  7258  7298  7096  7096  7014  6901  6773
    592    1 2021-05-01  8861  8289  7587  7208  7242  7046  7051  6962  6845  6666
    593    1 2021-05-01  7956  7478  6826  6468  6482  6325  6339  6332  6166  5878
    594    1 2021-05-01  8016  7613  6968  6619  6639  6480  6500  6509  6317  5956
    595    1 2021-05-01  7928  7699  7064  6729  6745  6603  6617  6596  6445  5873
    596    1 2021-05-01  7985  7536  6922  6592  6616  6475  6503  6463  6327  5938
    597    1 2021-05-01  8186  7661  7040  6718  6755  6615  6647  6524  6484  6155
    598    1 2021-05-01  8231  7727  7085  6746  6775  6625  6648  6546  6480  6185
    599    1 2021-05-01  8395  7837  7176  6825  6853  6692  6707  6609  6528  6306
    600    1 2021-05-01 11109 10793 10125  9844  9812  9610  9702  9549  9529  7253
    601    1 2021-05-01 11282 10944 10269  9997  9985  9773  9860  9679  9697  7572
    602    1 2021-05-01 11319 10764 10094  9808  9787  9580  9668  9497  9494  7745
    603    1 2021-05-01 11176 11073 10396 10113 10106  9890  9967  9803  9791  7609
    604    1 2021-05-01 11313 10965 10271  9973  9955  9738  9814  9700  9632  7881
    605    1 2021-05-01 11488 11024 10344 10048 10074  9852  9923  9752  9748  8335
    606    1 2021-05-01 11292 10863 10186  9887  9915  9692  9764  9608  9597  8261
    607    1 2021-05-01 11134 10776 10069  9757  9786  9561  9612  9492  9438  8103
    608    1 2021-05-01 11194 10699 10001  9691  9712  9494  9550  9428  9386  8305
    609    1 2021-05-01 11099 10453  9770  9479  9516  9304  9362  9189  9203  8318
    610    1 2021-05-01 10631 10179  9506  9217  9233  9045  9113  8955  8948  7725
    611    1 2021-05-01 10500 10141  9480  9194  9197  9019  9096  8996  8934  7516
    612    1 2021-05-01 10526  9947  9325  9059  9072  8905  8996  8847  8850  7635
    613    1 2021-05-01 10290  9970  9328  9041  9039  8857  8938  8852  8782  7390
    614    1 2021-05-01 10674 10596  9883  9558  9558  9329  9374  9364  9187  7734
    615    1 2021-05-01 11155 10922 10200  9856  9860  9620  9651  9643  9446  8285
    616    1 2021-05-01 11581 11084 10366 10041 10061  9817  9832  9798  9635  8752
    617    1 2021-05-01 11646 11148 10450 10141 10173  9927  9956  9880  9764  8794
    618    1 2021-05-01 11603 11176 10441 10111 10150  9894  9906  9827  9698  8767
    619    1 2021-05-01 12281 11654 10882 10516 10569 10275 10260 10259 10028  9720
    620    1 2021-05-01 12441 11995 11232 10881 10937 10644 10621 10616 10389  9944
    621    1 2021-05-01 12442 12013 11243 10878 10937 10637 10601 10616 10363  9920
    622    1 2021-05-01 12652 12261 11491 11139 11210 10900 10869 10876 10634 10176
    623    1 2021-05-01 12816 12282 11522 11181 11257 10952 10924 10922 10689 10371
    624    1 2021-05-01 12751 12197 11452 11112 11195 10890 10867 10844 10631 10406
    625    1 2021-05-01  9667  8813  8183  7874  7963  7783  7803  7621  7634  7678
    626    1 2021-05-01  9362  8729  8079  7755  7822  7636  7653  7526  7464  7330
    627    1 2021-05-01  9263  8663  8021  7694  7761  7581  7596  7457  7411  7204
    628    1 2021-05-01  9118  8509  7869  7551  7619  7448  7474  7265  7296  7017
    629    1 2021-05-01  8952  8299  7612  7254  7305  7116  7136  6981  6947  6781
    630    1 2021-05-01  9023  8416  7749  7399  7451  7263  7274  7135  7090  6866
    631    1 2021-05-01  9237  8343  7700  7368  7430  7247  7266  7068  7076  7125
    632    1 2021-05-01  9024  8403  7786  7489  7573  7407  7434  7141  7260  6881
    633    1 2021-05-01  8691  7941  7311  6990  7058  6906  6931  6665  6769  6498
    634    1 2021-05-01  7668  7190  6566  6221  6241  6091  6123  6074  5945  5560
    635    1 2021-05-01  7743  7316  6680  6326  6343  6189  6209  6201  6034  5645
    636    1 2021-05-01  7659  7238  6619  6280  6293  6148  6172  6154  6005  5586
    637    1 2021-05-01  7694  7400  6786  6448  6465  6324  6354  6323  6186  5656
    638    1 2021-05-01  7941  7605  6993  6658  6686  6533  6554  6499  6377  5876
    639    1 2021-05-01  8084  7639  7030  6708  6745  6601  6639  6523  6468  6053
    640    1 2021-05-01  8317  7763  7140  6814  6851  6702  6733  6613  6568  6309
    641    1 2021-05-01  8371  7948  7305  6961  6992  6828  6846  6808  6676  6314
    642    1 2021-05-01  8535  7948  7305  6961  6992  6828  6846  6808  6676  6476
    643    1 2021-05-01  8617  8110  7479  7150  7193  7027  7044  6943  6869  6555
    644    1 2021-05-01  8717  8176  7545  7223  7264  7103  7131  7031  6956  6639
    645    1 2021-05-01 10081 10373  9732  9470  9409  9231  9355  9163  9194  5996
    646    1 2021-05-01 10388 10105  9454  9182  9108  8932  9060  8903  8895  6355
    647    1 2021-05-01 10819 10610  9954  9677  9639  9439  9539  9389  9373  6901
    648    1 2021-05-01 11097 10431  9766  9477  9439  9243  9354  9174  9181  7375
    649    1 2021-05-01 10865 10663  9972  9662  9636  9425  9510  9385  9335  7323
    650    1 2021-05-01 11083 10614  9928  9615  9596  9389  9467  9358  9302  7663
    651    1 2021-05-01 11163 10682  9994  9679  9685  9475  9544  9428  9366  7929
    652    1 2021-05-01 11044 10501  9819  9525  9540  9329  9413  9259  9247  7873
    653    1 2021-05-01 10867 10503  9812  9507  9505  9303  9366  9279  9198  7768
    654    1 2021-05-01 10909 10175  9523  9241  9236  9063  9151  9014  8989  7828
    655    1 2021-05-01 10858 10065  9394  9098  9101  8913  8999  8851  8842  7850
    656    1 2021-05-01 10497 10179  9506  9217  9233  9045  9113  8955  8948  7403
    657    1 2021-05-01 10382  9983  9326  9032  9030  8853  8929  8797  8749  7379
    658    1 2021-05-01 10334  9975  9352  9083  9089  8916  9008  8870  8844  7334
    659    1 2021-05-01 10302  9871  9257  8990  8996  8829  8917  8772  8767  7273
    660    1 2021-05-01 10438 10300  9623  9307  9307  9103  9159  9118  8971  7413
    661    1 2021-05-01 11076 10560  9869  9542  9543  9322  9378  9335  9177  8163
    662    1 2021-05-01 11460 11010 10302  9967  9987  9740  9769  9731  9554  8588
    663    1 2021-05-01 11596 11126 10406 10070 10110  9851  9869  9782  9664  8787
    664    1 2021-05-01 11778 11563 10797 10437 10486 10202 10195 10178  9961  9064
    665    1 2021-05-01 12156 11839 11084 10742 10819 10530 10517 10429 10294  9583
    666    1 2021-05-01 12479 11949 11169 10801 10874 10570 10537 10507 10304 10067
    667    1 2021-05-01 12642 12223 11451 11096 11177 10876 10838 10808 10600 10241
    668    1 2021-05-01 12708 12129 11391 11062 11135 10835 10819 10761 10586 10296
    669    1 2021-05-01 12688 12126 11369 11025 11096 10794 10767 10763 10523 10218
    670    1 2021-05-01 12590 12126 11369 11025 11096 10794 10767 10763 10523 10231
    671    1 2021-05-01 12570 12017 11268 10923 11001 10710 10685 10642 10458 10240
    672    1 2021-05-01  9555  8847  8207  7908  7989  7815  7836  7637  7666  7588
    673    1 2021-05-01  9295  8672  8026  7702  7764  7589  7596  7466  7409  7271
    674    1 2021-05-01  9250  8700  8041  7706  7769  7584  7596  7472  7406  7151
    675    1 2021-05-01  9407  8729  8058  7719  7783  7585  7594  7475  7400  7340
    676    1 2021-05-01  9444  9027  8355  8025  8095  7898  7902  7788  7708  7397
    677    1 2021-05-01  9469  8869  8229  7923  7999  7821  7829  7633  7651  7419
    678    1 2021-05-01  9324  8341  7734  7448  7538  7383  7415  7103  7252  7269
    679    1 2021-05-01  9265  8235  7642  7364  7455  7304  7347  7021  7194  7223
    680    1 2021-05-01  8796  7783  7162  6850  6921  6775  6810  6523  6653  6703
    681    1 2021-05-01  8393  7686  7070  6768  6839  6697  6727  6429  6581  6217
    682    1 2021-05-01  7587  7130  6510  6177  6200  6056  6081  5970  5915  5491
    683    1 2021-05-01  7701  7227  6601  6262  6285  6139  6158  6102  5994  5614
    684    1 2021-05-01  7706  7227  6601  6262  6285  6139  6158  6102  5994  5639
    685    1 2021-05-01  7911  7307  6697  6370  6387  6260  6292  6243  6133  5874
    686    1 2021-05-01  8259  7667  7017  6667  6681  6526  6538  6538  6363  6230
    687    1 2021-05-01  8457  7813  7216  6910  6954  6809  6837  6695  6673  6483
    688    1 2021-05-01  8448  7894  7283  6969  7009  6871  6901  6772  6743  6485
    689    1 2021-05-01  8390  7844  7221  6894  6939  6783  6810  6708  6644  6374
    690    1 2021-05-01  8497  7980  7359  7032  7069  6909  6936  6872  6772  6489
    691    1 2021-05-01  8602  8167  7536  7205  7248  7084  7104  7048  6931  6596
    692    1 2021-05-01  9984  9576  8980  8733  8663  8522  8666  8392  8521  6010
    693    1 2021-05-01 10211  9872  9275  9022  9002  8837  8950  8616  8806  6377
    694    1 2021-05-01 10231  9951  9320  9040  8987  8819  8938  8714  8779  6339
    695    1 2021-05-01 10387 10161  9523  9236  9212  9025  9122  8893  8960  6699
    696    1 2021-05-01 10637 10286  9633  9334  9309  9120  9220  9052  9056  6979
    697    1 2021-05-01 10738 10286  9633  9334  9309  9120  9220  9052  9056  7219
    698    1 2021-05-01 10745 10514  9820  9508  9494  9296  9378  9246  9206  7288
    699    1 2021-05-01 10895 10450  9772  9472  9478  9277  9359  9217  9195  7648
    700    1 2021-05-01 10774 10294  9619  9332  9341  9147  9232  9072  9073  7602
    701    1 2021-05-01 10392 10175  9523  9241  9236  9063  9151  9014  8989  7180
    702    1 2021-05-01 10252  9843  9221  8950  8952  8789  8899  8705  8753  7009
    703    1 2021-05-01 10287  9973  9303  8993  8991  8798  8872  8748  8698  7088
    704    1 2021-05-01 10468 10070  9411  9114  9112  8927  8986  8856  8815  7467
    705    1 2021-05-01 10562  9984  9369  9100  9106  8927  9004  8849  8848  7604
    706    1 2021-05-01 10541 10067  9443  9172  9184  8998  9080  8913  8924  7566
    707    1 2021-05-01 10560 10341  9671  9363  9358  9146  9212  9148  9026  7572
    708    1 2021-05-01 10901 10691  9990  9661  9671  9442  9483  9435  9284  7966
    709    1 2021-05-01 11162 11185 10443 10092 10121  9855  9871  9817  9654  8274
    710    1 2021-05-01 11827 11185 10443 10092 10121  9855  9871  9817  9654  9192
    711    1 2021-05-01 12413 11940 11172 10828 10884 10593 10569 10557 10337  9832
    712    1 2021-05-01 12559 12026 11245 10877 10953 10642 10617 10582 10391 10117
    713    1 2021-05-01 12798 12223 11451 11096 11177 10876 10838 10808 10600 10412
    714    1 2021-05-01 12819 12265 11519 11182 11267 10958 10944 10885 10709 10400
    715    1 2021-05-01 12756 12073 11332 10997 11073 10786 10765 10693 10531 10331
    716    1 2021-05-01 12668 12224 11474 11129 11208 10902 10872 10864 10629 10295
    717    1 2021-05-01 12701 12116 11380 11033 11133 10832 10814 10750 10595 10390
    718    1 2021-05-01  9336  8878  8241  7931  8010  7835  7850  7677  7675  7281
    719    1 2021-05-01  9862  8993  8318  7978  8049  7844  7851  7758  7657  7916
    720    1 2021-05-01  9891  9027  8355  8025  8095  7898  7902  7788  7708  7954
    721    1 2021-05-01  9856  9250  8598  8291  8378  8188  8198  8036  8014  7900
    722    1 2021-05-01  9879  8845  8224  7929  8020  7848  7874  7628  7703  7935
    723    1 2021-05-01  9548  8765  8147  7856  7946  7776  7804  7573  7638  7573
    724    1 2021-05-01  9199  8212  7619  7341  7433  7285  7326  7015  7174  7181
    725    1 2021-05-01  8769  8148  7583  7321  7415  7280  7327  7003  7188  6704
    726    1 2021-05-01  8626  7402  6831  6551  6625  6508  6559  6217  6427  6564
    727    1 2021-05-01  7677  7174  6534  6183  6200  6051  6065  5991  5886  5588
    728    1 2021-05-01  7944  7304  6678  6348  6373  6227  6252  6138  6088  5935
    729    1 2021-05-01  7849  7287  6658  6323  6349  6202  6225  6144  6067  5820
    730    1 2021-05-01  7893  7457  6851  6521  6548  6406  6427  6407  6261  5865
    731    1 2021-05-01  7981  7625  7001  6681  6718  6579  6605  6529  6452  5982
    732    1 2021-05-01  8164  7767  7145  6821  6857  6711  6741  6701  6574  6188
    733    1 2021-05-01  8324  7894  7283  6961  7001  6848  6876  6810  6710  6378
    734    1 2021-05-01  8416  8035  7404  7069  7116  6948  6962  6879  6794  6435
    735    1 2021-05-01  8611  8091  7466  7150  7198  7033  7058  6951  6890  6615
    736    1 2021-05-01  8601  8321  7705  7394  7449  7283  7308  7203  7133  6626
    737    1 2021-05-01  8785  8184  7567  7247  7294  7137  7151  7088  6977  6827
    738    1 2021-05-01  8829  8307  7682  7358  7399  7243  7261  7209  7097  6857
    739    1 2021-05-01  9423  9198  8486  8097  8138  7949  7944  8073  7742  7563
    740    1 2021-05-01 10381  9815  9176  8903  8844  8669  8796  8560  8631  6602
    741    1 2021-05-01 10629  9924  9297  9026  8999  8817  8924  8642  8760  7044
    742    1 2021-05-01 10738 10493  9805  9492  9495  9274  9334  9102  9147  7199
    743    1 2021-05-01 11311 10220  9588  9310  9312  9127  9218  8874  9044  8199
    744    1 2021-05-01 11110 10601  9927  9616  9635  9420  9479  9225  9295  7929
    745    1 2021-05-01 10861 10386  9751  9471  9473  9284  9374  9126  9206  7428
    746    1 2021-05-01 10933 10437  9794  9518  9526  9333  9422  9180  9253  7562
    747    1 2021-05-01 10832 10361  9694  9394  9397  9196  9269  9119  9112  7569
    748    1 2021-05-01 10667 10225  9569  9290  9288  9099  9200  9025  9045  7377
    749    1 2021-05-01 10511 10092  9437  9155  9138  8958  9056  8903  8891  7191
    750    1 2021-05-01 10461 10020  9393  9128  9126  8954  9055  8874  8901  7162
    751    1 2021-05-01 10518 10125  9481  9207  9199  9025  9113  8949  8961  7336
    752    1 2021-05-01 10575 10151  9479  9174  9171  8974  9034  8898  8869  7500
    753    1 2021-05-01 10604 10151  9479  9174  9171  8974  9034  8898  8869  7656
    754    1 2021-05-01 10627 10126  9468  9174  9178  8990  9053  8925  8880  7686
    755    1 2021-05-01 10657 10238  9587  9289  9307  9098  9165  9029  8993  7716
    756    1 2021-05-01 10790 10247  9594  9292  9301  9098  9167  9043  8990  7827
    757    1 2021-05-01 11074 10617  9936  9623  9640  9421  9482  9373  9283  8129
    758    1 2021-05-01 12396 11881 11119 10756 10799 10507 10489 10496 10250  9829
    759    1 2021-05-01 12491 12037 11257 10891 10959 10656 10623 10611 10380  9956
    760    1 2021-05-01 12594 12192 11405 11044 11117 10803 10769 10755 10508 10189
    761    1 2021-05-01 12771 12267 11505 11153 11230 10920 10891 10868 10647 10372
    762    1 2021-05-01 12685 12173 11420 11077 11153 10849 10817 10787 10583 10289
    763    1 2021-05-01 12649 12129 11390 11047 11133 10831 10802 10766 10577 10245
    764    1 2021-05-01 12586 12065 11327 10972 11047 10753 10727 10713 10496 10237
    765    1 2021-05-01 12639 12186 11455 11104 11187 10887 10857 10853 10625 10284
    766    1 2021-05-01 10272  9577  8922  8620  8712  8517  8517  8399  8335  8405
    767    1 2021-05-01 10286  9543  8910  8626  8731  8557  8561  8390  8400  8445
    768    1 2021-05-01 10004  9126  8500  8212  8308  8132  8153  7961  7990  8122
    769    1 2021-05-01  9724  9180  8554  8272  8366  8193  8212  8025  8045  7804
    770    1 2021-05-01  9505  8686  8114  7855  7951  7802  7841  7579  7684  7550
    771    1 2021-05-01  9449  8686  8114  7855  7951  7802  7841  7579  7684  7552
    772    1 2021-05-01  9027  7996  7441  7181  7278  7151  7198  6848  7055  7064
    773    1 2021-05-01  8563  8089  7514  7244  7335  7207  7243  6948  7097  6525
    774    1 2021-05-01  8739  7837  7230  6922  6978  6847  6881  6678  6735  6724
    775    1 2021-05-01  8185  7468  6846  6521  6558  6414  6444  6310  6275  6206
    776    1 2021-05-01  8084  7598  7000  6699  6751  6607  6640  6497  6476  6129
    777    1 2021-05-01  7982  7508  6909  6595  6637  6496  6532  6454  6371  6012
    778    1 2021-05-01  7937  7654  7095  6806  6855  6731  6774  6674  6617  5994
    779    1 2021-05-01  8123  7594  7010  6707  6747  6615  6645  6595  6491  6214
    780    1 2021-05-01  8275  7759  7157  6853  6896  6761  6793  6690  6633  6345
    781    1 2021-05-01  8342  7893  7292  6978  7023  6877  6911  6823  6746  6415
    782    1 2021-05-01  8746  8201  7563  7238  7289  7127  7153  7052  6984  6762
    783    1 2021-05-01  8996  8378  7751  7440  7495  7327  7347  7252  7182  7074
    784    1 2021-05-01  8998  8321  7705  7394  7449  7283  7308  7203  7133  7087
    785    1 2021-05-01  8891  8466  7850  7535  7589  7428  7458  7376  7282  6953
    786    1 2021-05-01  9089  8469  7864  7562  7613  7454  7474  7393  7310  7187
    787    1 2021-05-01  9031  8480  7884  7593  7648  7499  7524  7399  7361  7120
    788    1 2021-05-01  8925  8357  7748  7442  7498  7358  7390  7277  7228  7028
    789    1 2021-05-01  8865  8781  8088  7703  7743  7570  7567  7708  7371  7035
    790    1 2021-05-01  9605  9676  8943  8546  8590  8383  8347  8579  8131  7840
    791    1 2021-05-01 10375 10056  9385  9051  9123  8928  8908  9025  8709  8692
    792    1 2021-05-01 11151 10647  9915  9592  9591  9361  9410  9201  9199  7906
    793    1 2021-05-01 11375 10890 10168  9842  9855  9615  9648  9467  9454  8289
    794    1 2021-05-01 11679 11262 10537 10218 10251 10004 10032  9866  9849  8737
    795    1 2021-05-01 11676 11012 10317 10009 10061  9831  9883  9625  9707  8702
    796    1 2021-05-01 11391 10945 10248  9926  9977  9746  9794  9578  9614  8378
    797    1 2021-05-01 11163 10703 10036  9741  9754  9547  9613  9435  9434  7938
    798    1 2021-05-01 11142 10577  9918  9634  9651  9451  9524  9311  9362  7971
    799    1 2021-05-01 11105 10577  9918  9634  9651  9451  9524  9311  9362  7935
    800    1 2021-05-01 10811 10263  9609  9341  9344  9157  9244  9040  9090  7701
    801    1 2021-05-01 10898 10260  9602  9323  9331  9144  9223  9024  9053  7844
    802    1 2021-05-01 10814 10180  9536  9261  9257  9078  9168  8993  9001  7698
    803    1 2021-05-01 10631 10267  9624  9350  9347  9175  9258  9078  9093  7501
    804    1 2021-05-01 10666 10282  9615  9315  9320  9127  9191  9043  9030  7588
    805    1 2021-05-01 10646 10242  9581  9277  9278  9085  9156  9026  8991  7524
    806    1 2021-05-01 10730 10339  9657  9345  9350  9136  9194  9098  9009  7737
    807    1 2021-05-01 10875 10539  9862  9557  9565  9348  9403  9306  9221  7906
    808    1 2021-05-01 11016 10714 10036  9725  9750  9532  9581  9459  9411  8013
    809    1 2021-05-01 11388 11036 10338 10010 10031  9796  9828  9744  9627  8450
    810    1 2021-05-01 11754 11599 10857 10514 10541 10265 10279 10263 10057  8854
    811    1 2021-05-01 12213 11749 10987 10634 10686 10407 10410 10335 10179  9550
    812    1 2021-05-01 12489 11975 11203 10847 10912 10605 10589 10555 10343 10046
    813    1 2021-05-01 12706 12151 11359 10995 11052 10741 10703 10718 10446 10302
    814    1 2021-05-01 12764 12224 11464 11108 11188 10879 10853 10816 10611 10394
    815    1 2021-05-01 12744 12192 11446 11094 11180 10884 10854 10795 10620 10349
    816    1 2021-05-01 12624 12126 11394 11053 11140 10839 10816 10761 10584 10246
    817    1 2021-05-01 12550 12062 11328 10986 11074 10777 10764 10702 10538 10206
    818    1 2021-05-01 12491 11939 11218 10873 10958 10663 10650 10595 10416 10159
    819    1 2021-05-01 12351 11761 11036 10674 10758 10470 10457 10413 10230 10035
    820    1 2021-05-01 10470  9863  9237  8962  9070  8882  8887  8756  8713  8686
    821    1 2021-05-01 10258  9497  8870  8594  8706  8534  8547  8354  8388  8459
    822    1 2021-05-01 10045  9580  8950  8679  8789  8615  8635  8466  8469  8213
    823    1 2021-05-01 10124  9215  8617  8354  8461  8297  8321  8109  8159  8321
    824    1 2021-05-01  9846  9260  8666  8411  8525  8368  8391  8165  8239  8021
    825    1 2021-05-01  9505  8750  8164  7899  8002  7852  7874  7634  7727  7630
    826    1 2021-05-01  9200  8914  8312  8027  8122  7962  7976  7818  7810  7277
    827    1 2021-05-01  9380  8666  8075  7794  7881  7733  7771  7543  7609  7507
    828    1 2021-05-01  9188  8457  7853  7558  7632  7469  7494  7319  7327  7295
    829    1 2021-05-01  9196  8592  8027  7758  7860  7703  7732  7492  7572  7327
    830    1 2021-05-01  8635  7713  7104  6793  6848  6699  6731  6555  6574  6789
    831    1 2021-05-01  8427  7923  7330  7033  7102  6957  6989  6819  6841  6555
    832    1 2021-05-01  8316  7776  7192  6895  6955  6820  6854  6698  6696  6423
    833    1 2021-05-01  8493  8003  7434  7139  7199  7059  7092  6963  6931  6628
    834    1 2021-05-01  8419  7879  7294  7004  7056  6933  6971  6851  6812  6548
    835    1 2021-05-01  8585  8111  7502  7188  7241  7097  7123  7078  6959  6691
    836    1 2021-05-01  8730  8387  7769  7459  7520  7363  7383  7302  7210  6826
    837    1 2021-05-01  8970  8536  7905  7594  7651  7484  7503  7439  7341  7073
    838    1 2021-05-01  9171  8693  8090  7810  7872  7721  7744  7661  7587  7326
    839    1 2021-05-01  9150  8642  8044  7751  7814  7664  7699  7592  7540  7303
    840    1 2021-05-01  9150  8686  8085  7788  7850  7691  7715  7650  7553  7319
    841    1 2021-05-01  9135  8616  8001  7693  7748  7584  7608  7542  7438  7251
    842    1 2021-05-01  9113  8631  8037  7754  7828  7676  7706  7583  7547  7253
    843    1 2021-05-01  8987  8480  7898  7610  7674  7533  7563  7441  7411  7154
    844    1 2021-05-01  9074  8604  8013  7722  7787  7635  7668  7582  7508  7252
    845    1 2021-05-01  9235  8656  8058  7770  7847  7689  7729  7595  7564  7413
    846    1 2021-05-01  8971  8377  7800  7525  7600  7471  7518  7336  7357  7219
    847    1 2021-05-01  9107  8399  7832  7564  7646  7527  7565  7376  7422  7402
    848    1 2021-05-01  8871  8076  7534  7276  7362  7257  7318  7120  7182  7181
    849    1 2021-05-01  8511  8080  7456  7116  7157  7021  7046  7035  6879  6670
    850    1 2021-05-01  8713  8792  8067  7655  7674  7485  7471  7696  7261  6847
    851    1 2021-05-01  9806  9768  9071  8699  8755  8545  8515  8750  8305  8076
    852    1 2021-05-01 10311 10048  9373  9042  9121  8925  8904  9050  8715  8712
    853    1 2021-05-01 10493 10342  9677  9365  9453  9263  9260  9387  9067  8903
    854    1 2021-05-01 10594 10316  9670  9359  9463  9276  9281  9347  9099  9066
    855    1 2021-05-01 11254 10997 10263  9943  9961  9720  9752  9564  9545  7881
    856    1 2021-05-01 11427 11387 10650 10332 10364 10110 10139  9974  9936  8321
    857    1 2021-05-01 11976 11178 10436 10107 10125  9867  9895  9750  9694  9133
    858    1 2021-05-01 11995 11456 10724 10397 10437 10177 10198 10065 10003  9164
    859    1 2021-05-01 11973 11430 10736 10423 10479 10232 10264 10086 10085  9103
    860    1 2021-05-01 11648 10945 10248  9926  9977  9746  9794  9578  9614  8760
    861    1 2021-05-01 11545 10913 10220  9905  9933  9717  9758  9582  9579  8651
    862    1 2021-05-01 11241 10755 10064  9754  9767  9547  9597  9482  9423  8257
    863    1 2021-05-01 11224 10798 10105  9804  9810  9603  9649  9522  9465  8176
    864    1 2021-05-01 11243 10613  9930  9639  9667  9463  9523  9323  9350  8229
    865    1 2021-05-01 11106 10633  9947  9659  9682  9475  9541  9366  9365  8098
    866    1 2021-05-01 10944 10423  9773  9502  9510  9323  9401  9223  9233  7878
    867    1 2021-05-01 10776 10227  9584  9308  9303  9122  9208  9045  9046  7673
    868    1 2021-05-01 10618 10057  9420  9142  9124  8946  9040  8900  8880  7446
    869    1 2021-05-01 10433 10006  9345  9041  9012  8817  8914  8812  8738  7176
    870    1 2021-05-01 10822 10291  9605  9286  9278  9070  9134  9035  8945  7723
    871    1 2021-05-01 11045 10539  9846  9530  9523  9300  9356  9278  9162  7953
    872    1 2021-05-01 11312 10804 10126  9823  9842  9619  9667  9552  9492  8302
    873    1 2021-05-01 11461 11036 10338 10010 10031  9796  9828  9744  9627  8574
    874    1 2021-05-01 11958 11425 10704 10365 10390 10127 10142 10104  9938  9123
    875    1 2021-05-01 12160 11829 11089 10757 10800 10530 10535 10442 10309  9366
    876    1 2021-05-01 12402 11927 11154 10782 10832 10539 10526 10490 10288  9821
    877    1 2021-05-01 12559 12096 11314 10949 11008 10699 10667 10656 10433 10104
    878    1 2021-05-01 12822 12320 11541 11163 11237 10916 10883 10899 10627 10426
    879    1 2021-05-01 12826 12253 11500 11137 11236 10928 10904 10856 10665 10434
    880    1 2021-05-01 12697 12132 11388 11036 11126 10825 10803 10742 10565 10320
    881    1 2021-05-01 12618 12075 11341 10999 11084 10784 10768 10732 10526 10244
    882    1 2021-05-01 12569 11881 11145 10788 10868 10582 10560 10531 10322 10145
    883    1 2021-05-01 12370 11769 11051 10698 10784 10507 10493 10419 10264 10036
    884    1 2021-05-01 12214 11769 11051 10698 10784 10507 10493 10419 10264  9912
    885    1 2021-05-01 10529  9962  9332  9059  9168  8978  8997  8909  8824  8770
    886    1 2021-05-01 10398  9658  9052  8796  8907  8734  8751  8591  8596  8650
    887    1 2021-05-01 10152  9611  8999  8734  8844  8663  8680  8538  8516  8388
    888    1 2021-05-01  9859  9241  8642  8382  8500  8339  8361  8149  8203  8046
    889    1 2021-05-01  9872  8914  8312  8027  8122  7962  7976  7818  7810  8127
    890    1 2021-05-01  9733  8887  8291  8015  8115  7953  7988  7773  7829  7961
    891    1 2021-05-01  9839  9076  8480  8205  8313  8149  8171  7995  8006  8106
    892    1 2021-05-01  9651  8655  8095  7832  7936  7793  7829  7578  7676  7912
    893    1 2021-05-01  9501  8958  8367  8091  8189  8024  8045  7869  7887  7744
    894    1 2021-05-01  9222  8478  7923  7661  7767  7622  7652  7379  7512  7441
    895    1 2021-05-01  9032  8335  7746  7452  7524  7379  7401  7271  7245  7313
    896    1 2021-05-01  8867  8389  7822  7545  7628  7487  7512  7364  7371  7093
    897    1 2021-05-01  8566  8090  7528  7245  7308  7175  7215  7111  7060  6781
    898    1 2021-05-01  8633  8111  7502  7188  7241  7097  7123  7078  6959  6784
    899    1 2021-05-01  8938  8393  7783  7470  7529  7372  7386  7375  7225  7103
    900    1 2021-05-01  9071  8642  8038  7737  7807  7649  7668  7595  7501  7269
    901    1 2021-05-01  9228  8759  8172  7884  7954  7800  7828  7761  7664  7410
    902    1 2021-05-01  9391  8748  8166  7887  7958  7810  7841  7746  7681  7620
    903    1 2021-05-01  9388  8847  8256  7971  8050  7888  7923  7818  7762  7611
    904    1 2021-05-01  9323  8790  8207  7926  7998  7848  7870  7772  7710  7493
    905    1 2021-05-01  9192  8885  8304  8029  8109  7967  7985  7845  7830  7352
    906    1 2021-05-01  9723  8794  8194  7912  7986  7832  7864  7728  7699  7961
    907    1 2021-05-01  9719  9326  8702  8420  8509  8339  8357  8199  8195  7994
    908    1 2021-05-01  9665  9181  8580  8305  8398  8232  8263  8126  8101  7945
    909    1 2021-05-01  9613  9312  8710  8435  8525  8365  8388  8249  8228  7898
    910    1 2021-05-01  9671  9122  8511  8227  8310  8150  8178  8066  8019  8015
    911    1 2021-05-01  9591  8952  8365  8075  8154  7996  8020  7890  7851  7921
    912    1 2021-05-01  9485  8952  8365  8075  8154  7996  8020  7890  7851  7777
    913    1 2021-05-01  9606  8639  8062  7787  7871  7726  7763  7604  7596  7907
    914    1 2021-05-01  9342  8707  8147  7881  7973  7843  7875  7692  7731  7683
    915    1 2021-05-01  8983  8417  7869  7610  7697  7583  7616  7481  7480  7373
    916    1 2021-05-01  8958  8503  7946  7679  7759  7637  7673  7591  7533  7285
    917    1 2021-05-01  9145  8565  7962  7672  7756  7620  7644  7510  7490  7499
    918    1 2021-05-01  8362  8440  7744  7354  7383  7214  7218  7343  7030  6516
    919    1 2021-05-01  8754  8440  7744  7354  7383  7214  7218  7343  7030  6906
    920    1 2021-05-01  9502  9470  8740  8338  8373  8158  8110  8396  7895  7724
    921    1 2021-05-01  9807  9808  9103  8734  8799  8588  8553  8790  8347  8110
    922    1 2021-05-01 10396 10237  9572  9257  9340  9147  9134  9228  8946  8789
    923    1 2021-05-01 10700 10305  9643  9334  9429  9244  9237  9298  9043  9143
    924    1 2021-05-01 10729 10240  9621  9339  9454  9277  9299  9279  9126  9174
    925    1 2021-05-01 10467  9668  9102  8848  8964  8834  8888  8798  8743  8900
    926    1 2021-05-01 11707 11190 10446 10124 10145  9887  9924  9780  9715  8672
    927    1 2021-05-01 11967 11538 10793 10477 10510 10249 10273 10146 10072  9041
    928    1 2021-05-01 12074 11589 10857 10535 10574 10320 10340 10199 10151  9192
    929    1 2021-05-01 12104 11580 10867 10542 10590 10331 10357 10235 10167  9238
    930    1 2021-05-01 12039 11530 10826 10527 10585 10339 10385 10203 10214  9145
    931    1 2021-05-01 11910 11184 10466 10150 10186  9947  9987  9825  9806  9020
    932    1 2021-05-01 11715 11062 10359 10071 10100  9876  9929  9745  9754  8851
    933    1 2021-05-01 11283 10802 10094  9776  9783  9560  9608  9497  9414  8330
    934    1 2021-05-01 11235 10724 10037  9731  9726  9510  9570  9475  9380  8192
    935    1 2021-05-01 11181 10814 10130  9835  9851  9640  9702  9552  9524  8072
    936    1 2021-05-01 11160 10723 10057  9771  9789  9587  9652  9507  9491  8090
    937    1 2021-05-01 10959 10454  9797  9522  9525  9332  9406  9255  9251  7880
    938    1 2021-05-01 10555 10168  9536  9267  9250  9083  9168  8991  9012  7356
    939    1 2021-05-01 10386  9910  9302  9039  9032  8855  8965  8759  8811  7126
    940    1 2021-05-01 10206  9825  9223  8961  8958  8787  8903  8666  8749  6832
    941    1 2021-05-01 10584 10423  9734  9413  9391  9173  9242  9170  9046  7275
    942    1 2021-05-01 11236 10851 10152  9827  9828  9597  9643  9580  9451  8111
    943    1 2021-05-01 11557 11236 10508 10157 10176  9924  9942  9896  9728  8618
    944    1 2021-05-01 11888 11437 10717 10377 10428 10162 10176 10088  9978  9128
    945    1 2021-05-01 12043 11587 10851 10502 10551 10274 10285 10200 10069  9333
    946    1 2021-05-01 12277 11858 11100 10738 10781 10505 10498 10458 10270  9558
    947    1 2021-05-01 12425 12039 11259 10878 10930 10631 10597 10605 10340  9856
    948    1 2021-05-01 12652 12127 11348 10980 11037 10732 10704 10694 10464 10211
    949    1 2021-05-01 12724 12246 11456 11076 11149 10841 10812 10815 10582 10324
    950    1 2021-05-01 12742 12304 11549 11183 11279 10965 10948 10910 10705 10366
    951    1 2021-05-01 12660 12188 11436 11083 11167 10865 10840 10800 10602 10283
    952    1 2021-05-01 12579 12025 11295 10943 11025 10729 10722 10672 10487 10182
    953    1 2021-05-01 12393 11881 11159 10816 10898 10620 10608 10522 10375  9987
    954    1 2021-05-01 12316 11775 11049 10691 10766 10490 10474 10426 10230  9976
    955    1 2021-05-01 12155 11622 10902 10547 10631 10353 10339 10264 10122  9849
    956    1 2021-05-01 12125 11586 10868 10514 10596 10311 10299 10223 10079  9852
    957    1 2021-05-01 10274  9611  8999  8734  8844  8663  8680  8538  8516  8548
    958    1 2021-05-01 10284  9648  9040  8774  8876  8702  8715  8593  8544  8574
    959    1 2021-05-01 10122  9457  8825  8548  8648  8466  8482  8393  8312  8412
    960    1 2021-05-01 10358  9522  8922  8657  8766  8602  8615  8455  8454  8711
    961    1 2021-05-01 10230  9638  9038  8770  8889  8713  8726  8585  8569  8575
    962    1 2021-05-01  9974  9376  8781  8511  8625  8453  8462  8313  8308  8306
    963    1 2021-05-01  9814  9553  8946  8679  8792  8610  8614  8528  8448  8121
    964    1 2021-05-01 10023  9332  8739  8477  8592  8420  8434  8293  8283  8405
    965    1 2021-05-01  9895  9332  8739  8477  8592  8420  8434  8293  8283  8274
    966    1 2021-05-01  9674  8999  8434  8185  8303  8147  8173  7967  8031  8036
    967    1 2021-05-01  9309  9002  8443  8194  8304  8152  8182  7986  8038  7633
    968    1 2021-05-01  9415  8640  8067  7798  7891  7744  7774  7595  7622  7762
    969    1 2021-05-01  8915  8497  7940  7681  7769  7634  7671  7487  7531  7171
    970    1 2021-05-01  9082  8474  7896  7610  7685  7535  7577  7469  7408  7347
    971    1 2021-05-01  9393  8823  8242  7955  8042  7878  7913  7800  7754  7623
    972    1 2021-05-01  9373  8828  8250  7969  8051  7901  7928  7838  7770  7604
    973    1 2021-05-01  9367  9134  8535  8250  8343  8180  8193  8102  8035  7587
    974    1 2021-05-01  9612  9137  8541  8254  8340  8169  8190  8105  8020  7853
    975    1 2021-05-01  9765  9377  8768  8487  8568  8394  8411  8328  8239  8055
    976    1 2021-05-01  9723  9378  8764  8484  8573  8400  8417  8295  8244  7982
    977    1 2021-05-01 10110  9378  8764  8484  8573  8400  8417  8295  8244  8389
    978    1 2021-05-01 10144  9353  8738  8467  8562  8394  8420  8240  8250  8435
    979    1 2021-05-01 10208  9895  9258  8982  9077  8894  8907  8789  8735  8528
    980    1 2021-05-01 10150  9614  9008  8746  8843  8679  8693  8533  8529  8549
    981    1 2021-05-01 10196  9312  8710  8435  8525  8365  8388  8249  8228  8621
    982    1 2021-05-01  9967  9369  8775  8511  8608  8449  8481  8312  8314  8372
    983    1 2021-05-01  9746  9173  8558  8262  8342  8179  8205  8109  8040  8116
    984    1 2021-05-01 10009  9370  8747  8452  8541  8367  8379  8303  8212  8391
    985    1 2021-05-01  9884  9167  8573  8294  8393  8235  8258  8129  8107  8291
    986    1 2021-05-01  9663  9132  8547  8284  8377  8225  8255  8121  8093  8075
    987    1 2021-05-01  9244  8634  8086  7821  7916  7798  7833  7714  7687  7661
    988    1 2021-05-01  9360  8900  8318  8041  8139  7998  8028  7920  7872  7807
    989    1 2021-05-01  9495  8987  8393  8105  8199  8050  8066  7958  7899  7924
    990    1 2021-05-01  9484  9207  8637  8373  8478  8333  8366  8218  8201  7924
    991    1 2021-05-01  8210  7633  7077  6783  6853  6758  6828  6681  6689  6483
    992    1 2021-05-01  8101  7990  7329  6947  6978  6827  6844  6947  6663  6277
    993    1 2021-05-01  8669  8405  7684  7269  7276  7110  7092  7330  6884  6841
    994    1 2021-05-01  9386  9275  8576  8191  8226  8034  8005  8244  7793  7648
    995    1 2021-05-01  9497  9298  8619  8259  8316  8143  8131  8288  7936  7791
    996    1 2021-05-01  9870  9941  9250  8889  8953  8760  8735  8955  8524  8242
    997    1 2021-05-01 10346  9796  9145  8819  8895  8727  8720  8862  8530  8760
    998    1 2021-05-01 10106  9674  9080  8808  8911  8765  8808  8770  8640  8503
    999    1 2021-05-01  9954  9531  8931  8641  8729  8586  8624  8607  8461  8306
    1000   1 2021-05-01  9702  9621  9013  8712  8803  8659  8694  8748  8521  8018
    1001   1 2021-05-01 11790 11437 10724 10431 10448 10215 10251 10081 10073  8502
    1002   1 2021-05-01 11952 11540 10790 10463 10497 10236 10264 10146 10073  8975
    1003   1 2021-05-01 12135 11665 10946 10643 10680 10428 10463 10298 10275  9152
    1004   1 2021-05-01 12172 11612 10902 10601 10637 10399 10433 10261 10251  9248
    1005   1 2021-05-01 12100 11620 10895 10574 10608 10362 10377 10254 10184  9203
    1006   1 2021-05-01 11988 11634 10919 10609 10635 10390 10422 10320 10217  8932
    1007   1 2021-05-01 11865 11391 10679 10379 10426 10188 10239 10061 10061  8860
    1008   1 2021-05-01 11670 11218 10528 10239 10268 10035 10096  9935  9913  8709
    1009   1 2021-05-01 11377 10836 10134  9830  9858  9638  9699  9518  9518  8422
    1010   1 2021-05-01 11196 10583  9910  9618  9626  9428  9500  9312  9329  8160
    1011   1 2021-05-01 10919 10581  9909  9610  9603  9393  9461  9374  9282  7779
    1012   1 2021-05-01 10859 10425  9789  9515  9518  9324  9397  9249  9230  7699
    1013   1 2021-05-01 10826 10284  9642  9369  9368  9177  9266  9073  9100  7688
    1014   1 2021-05-01 10736 10359  9698  9405  9413  9213  9278  9101  9108  7659
    1015   1 2021-05-01 10573 10027  9429  9170  9160  8987  9087  8864  8917  7313
    1016   1 2021-05-01 10303 10095  9433  9126  9091  8903  8998  8892  8830  6897
    1017   1 2021-05-01 10476 10692  9982  9650  9625  9396  9448  9410  9260  7085
    1018   1 2021-05-01 11169 10787 10090  9762  9763  9536  9587  9495  9399  7944
    1019   1 2021-05-01 11919 11220 10499 10154 10173  9922  9954  9894  9755  9033
    1020   1 2021-05-01 12056 11552 10833 10493 10539 10275 10294 10204 10085  9158
    1021   1 2021-05-01 12179 11662 10926 10578 10628 10350 10363 10281 10149  9420
    1022   1 2021-05-01 12263 11776 11032 10672 10724 10446 10448 10389 10216  9588
    1023   1 2021-05-01 12421 11816 11041 10658 10713 10425 10402 10374 10158  9876
    1024   1 2021-05-01 12606 12207 11436 11071 11133 10822 10809 10799 10553 10105
    1025   1 2021-05-01 12770 12288 11532 11155 11228 10916 10886 10901 10631 10319
    1026   1 2021-05-01 12759 12193 11442 11087 11168 10869 10845 10808 10601 10356
    1027   1 2021-05-01 12680 12073 11332 10980 11057 10759 10747 10703 10512 10291
    1028   1 2021-05-01 12503 12005 11273 10930 11007 10719 10702 10647 10469 10065
    1029   1 2021-05-01 12423 11795 11072 10721 10810 10525 10511 10426 10287 10053
    1030   1 2021-05-01 12266 11742 11013 10662 10752 10468 10451 10354 10224  9963
    1031   1 2021-05-01 12248 11689 10964 10606 10694 10408 10391 10312 10165 10010
    1032   1 2021-05-01 12309 11689 10964 10606 10694 10408 10391 10312 10165 10082
    1033   1 2021-05-01 10291  9888  9253  8973  9072  8881  8887  8841  8713  8651
    1034   1 2021-05-01 10481  9892  9274  9005  9112  8924  8922  8849  8750  8852
    1035   1 2021-05-01 10452  9956  9358  9097  9217  9039  9042  8948  8871  8866
    1036   1 2021-05-01 10295  9731  9135  8875  8998  8818  8825  8705  8667  8689
    1037   1 2021-05-01 10233  9728  9128  8871  8981  8798  8809  8737  8655  8651
    1038   1 2021-05-01 10118  9589  8994  8735  8846  8665  8671  8604  8509  8549
    1039   1 2021-05-01 10023  9553  8963  8713  8833  8658  8672  8583  8522  8463
    1040   1 2021-05-01  9891  9404  8822  8574  8689  8524  8550  8412  8399  8323
    1041   1 2021-05-01  9924  9097  8524  8263  8368  8213  8237  8068  8086  8372
    1042   1 2021-05-01  9750  9254  8676  8413  8528  8363  8384  8241  8233  8158
    1043   1 2021-05-01  9690  9017  8460  8209  8317  8160  8191  8032  8051  8047
    1044   1 2021-05-01  9657  8971  8437  8203  8308  8170  8218  8086  8076  8068
    1045   1 2021-05-01  9317  8928  8352  8073  8154  8013  8048  7996  7891  7686
    1046   1 2021-05-01  9372  8928  8352  8073  8154  8013  8048  7996  7891  7684
    1047   1 2021-05-01  9790  9110  8520  8230  8322  8153  8170  8094  8005  8082
    1048   1 2021-05-01 10054  9426  8824  8546  8633  8467  8473  8394  8302  8365
    1049   1 2021-05-01 10049  9435  8830  8550  8641  8476  8491  8389  8342  8354
    1050   1 2021-05-01  9963  9583  8963  8677  8764  8588  8610  8547  8449  8272
    1051   1 2021-05-01 10145  9790  9154  8867  8957  8762  8764  8706  8581  8457
    1052   1 2021-05-01 10496 10018  9389  9102  9203  9001  8994  8960  8812  8890
    1053   1 2021-05-01 10499 10050  9419  9145  9241  9045  9051  9018  8879  8906
    1054   1 2021-05-01 10561 10024  9391  9115  9208  9025  9031  8950  8860  8994
    1055   1 2021-05-01 10556  9985  9367  9091  9186  9007  9020  8954  8846  9027
    1056   1 2021-05-01 10434  9807  9196  8927  9027  8849  8860  8740  8697  8877
    1057   1 2021-05-01 10393  9955  9355  9095  9199  9019  9039  8919  8869  8822
    1058   1 2021-05-01 10449  9691  9084  8809  8912  8741  8754  8633  8603  8910
    1059   1 2021-05-01 10263  9683  9093  8827  8939  8773  8787  8666  8637  8704
    1060   1 2021-05-01  9985  9439  8834  8556  8665  8496  8517  8412  8361  8480
    1061   1 2021-05-01  9863  9428  8810  8525  8620  8453  8471  8395  8306  8332
    1062   1 2021-05-01  9983  9186  8611  8355  8462  8317  8345  8194  8193  8484
    1063   1 2021-05-01  9695  9263  8698  8445  8567  8422  8457  8307  8314  8188
    1064   1 2021-05-01  9962  9320  8710  8419  8513  8351  8371  8301  8195  8489
    1065   1 2021-05-01  9919  9603  9011  8735  8837  8674  8687  8636  8507  8456
    1066   1 2021-05-01  9982  9320  8755  8501  8625  8482  8512  8372  8359  8559
    1067   1 2021-05-01  9745  8753  8239  8017  8151  8041  8096  7823  7957  8276
    1068   1 2021-05-01  8188  7499  6983  6720  6799  6733  6805  6606  6682  6497
    1069   1 2021-05-01  7980  7821  7160  6780  6806  6679  6703  6792  6529  6187
    1070   1 2021-05-01  8293  8029  7355  6963  6989  6853  6859  7010  6672  6423
    1071   1 2021-05-01  8596  8733  8019  7612  7627  7448  7420  7677  7209  6789
    1072   1 2021-05-01  9192  8849  8167  7785  7813  7639  7616  7827  7407  7446
    1073   1 2021-05-01  9597  9397  8711  8344  8400  8220  8209  8381  8003  7882
    1074   1 2021-05-01  9559  9368  8711  8369  8423  8257  8258  8416  8065  7853
    1075   1 2021-05-01  9853  9528  8931  8645  8736  8592  8612  8642  8440  8187
    1076   1 2021-05-01  9980  9570  8945  8637  8721  8569  8590  8608  8408  8297
    1077   1 2021-05-01 10081  9837  9217  8923  9005  8852  8865  8883  8683  8383
    1078   1 2021-05-01 10409 10026  9404  9106  9202  9039  9059  9111  8874  8755
    1079   1 2021-05-01 10759 10319  9695  9414  9526  9365  9377  9386  9205  9189
    1080   1 2021-05-01 12126 11739 11014 10708 10735 10473 10506 10397 10310  9060
    1081   1 2021-05-01 12136 11693 10997 10712 10746 10503 10552 10392 10369  9109
    1082   1 2021-05-01 11994 11535 10817 10505 10532 10300 10340 10202 10150  8912
    1083   1 2021-05-01 11902 11378 10673 10371 10393 10160 10220 10095 10036  8832
    1084   1 2021-05-01 11804 11170 10477 10184 10208  9979 10039  9898  9860  8711
    1085   1 2021-05-01 11417 10921 10239  9944  9965  9740  9801  9661  9627  8338
    1086   1 2021-05-01 11226 10678 10016  9741  9742  9540  9618  9446  9452  8112
    1087   1 2021-05-01 11060 10634  9953  9650  9647  9433  9501  9375  9330  7933
    1088   1 2021-05-01 10998 10628  9975  9678  9686  9479  9539  9380  9352  7918
    1089   1 2021-05-01 11113 10628  9975  9678  9686  9479  9539  9380  9352  8099
    1090   1 2021-05-01 11018 10506  9848  9561  9568  9358  9435  9286  9259  8004
    1091   1 2021-05-01 10960 10359  9698  9405  9413  9213  9278  9101  9108  7958
    1092   1 2021-05-01 10732 10207  9578  9306  9305  9116  9204  9001  9033  7586
    1093   1 2021-05-01 10769 10049  9430  9162  9140  8968  9063  8863  8900  7475
    1094   1 2021-05-01 11069 10597  9908  9580  9572  9353  9404  9307  9213  7857
    1095   1 2021-05-01 11401 11143 10429 10086 10083  9828  9877  9840  9669  8262
    1096   1 2021-05-01 11640 11389 10675 10327 10337 10077 10098 10072  9888  8586
    1097   1 2021-05-01 11933 11570 10829 10471 10502 10233 10250 10205 10031  9060
    1098   1 2021-05-01 12136 11714 10959 10580 10623 10332 10333 10311 10099  9388
    1099   1 2021-05-01 12342 11858 11101 10731 10806 10512 10505 10414 10282  9701
    1100   1 2021-05-01 12490 12105 11330 10949 11018 10711 10685 10653 10445  9986
    1101   1 2021-05-01 12572 12190 11421 11045 11128 10813 10788 10779 10538 10144
    1102   1 2021-05-01 12705 12243 11488 11123 11203 10886 10870 10859 10614 10281
    1103   1 2021-05-01 12704 12226 11484 11127 11211 10902 10888 10851 10635 10304
    1104   1 2021-05-01 12622 12073 11332 10980 11057 10759 10747 10703 10512 10243
    1105   1 2021-05-01 12450 11960 11226 10866 10933 10644 10629 10596 10400 10053
    1106   1 2021-05-01 12367 11893 11154 10796 10872 10584 10564 10514 10331 10038
    1107   1 2021-05-01 12410 11875 11137 10779 10856 10578 10550 10495 10311 10143
    1108   1 2021-05-01 12391 11850 11119 10763 10851 10560 10536 10473 10299 10162
    1109   1 2021-05-01 12363 11829 11108 10760 10844 10571 10542 10483 10299 10155
    1110   1 2021-05-01 10528 10102  9483  9199  9307  9113  9117  9111  8942  8957
    1111   1 2021-05-01 10590 10136  9532  9262  9368  9179  9181  9156  9006  9025
    1112   1 2021-05-01 10553 10102  9503  9244  9364  9173  9181  9123  9012  9002
    1113   1 2021-05-01 10533  9879  9293  9045  9165  8981  8999  8906  8844  9040
    1114   1 2021-05-01 10408 10008  9428  9189  9318  9136  9153  9035  8992  8894
    1115   1 2021-05-01 10208  9739  9166  8932  9058  8894  8914  8790  8762  8701
    1116   1 2021-05-01 10083  9596  9023  8774  8895  8724  8749  8660  8599  8556
    1117   1 2021-05-01 10186  9596  9023  8774  8895  8724  8749  8660  8599  8712
    1118   1 2021-05-01 10184  9507  8924  8668  8782  8614  8630  8500  8480  8669
    1119   1 2021-05-01 10067  9603  9022  8765  8883  8712  8731  8638  8577  8543
    1120   1 2021-05-01 10073  9371  8794  8539  8651  8494  8515  8410  8375  8561
    1121   1 2021-05-01  9668  9230  8684  8452  8560  8421  8468  8355  8330  8139
    1122   1 2021-05-01  9662  9005  8463  8216  8315  8182  8234  8113  8084  8051
    1123   1 2021-05-01  9694  9458  8872  8612  8717  8555  8584  8487  8438  8087
    1124   1 2021-05-01 10048  9670  9055  8770  8866  8672  8680  8639  8506  8447
    1125   1 2021-05-01 10192  9791  9197  8939  9040  8856  8868  8814  8696  8544
    1126   1 2021-05-01 10135  9640  9049  8798  8898  8739  8756  8667  8608  8515
    1127   1 2021-05-01 10012  9850  9235  8969  9073  8905  8915  8805  8765  8386
    1128   1 2021-05-01 10313 10018  9385  9097  9196  8999  8993  8965  8823  8698
    1129   1 2021-05-01 10465 10085  9455  9173  9272  9074  9069  9027  8894  8887
    1130   1 2021-05-01 10588 10168  9539  9265  9367  9172  9172  9095  9001  9053
    1131   1 2021-05-01 10666 10093  9451  9172  9262  9071  9077  9031  8899  9135
    1132   1 2021-05-01 10632 10119  9491  9217  9306  9117  9124  9071  8947  9098
    1133   1 2021-05-01 10570 10067  9449  9172  9261  9078  9079  9024  8913  9028
    1134   1 2021-05-01 10589 10197  9583  9305  9410  9219  9227  9194  9057  9080
    1135   1 2021-05-01 10503 10037  9434  9168  9266  9084  9097  9019  8935  9040
    1136   1 2021-05-01 10524 10031  9424  9171  9284  9113  9123  9001  8969  9038
    1137   1 2021-05-01 10266  9749  9141  8874  8984  8814  8836  8715  8673  8808
    1138   1 2021-05-01 10520  9964  9356  9097  9210  9034  9045  8955  8872  9077
    1139   1 2021-05-01 10261  9621  9056  8813  8939  8786  8803  8697  8653  8822
    1140   1 2021-05-01 10018  9658  9077  8835  8960  8803  8823  8748  8675  8605
    1141   1 2021-05-01 10092  9724  9125  8855  8965  8803  8817  8790  8660  8662
    1142   1 2021-05-01 10180  9608  9019  8757  8872  8713  8732  8679  8570  8819
    1143   1 2021-05-01 10020  9477  8901  8646  8764  8612  8635  8572  8474  8686
    1144   1 2021-05-01  9849  9232  8691  8457  8594  8454  8489  8315  8332  8464
    1145   1 2021-05-01  9589  8882  8387  8185  8318  8210  8268  8012  8138  8228
    1146   1 2021-05-01  9085  8073  7567  7337  7453  7370  7440  7125  7327  7596
    1147   1 2021-05-01  8179  7726  7232  6986  7087  7008  7076  6828  6963  6531
    1148   1 2021-05-01  7784  7412  6850  6542  6601  6524  6593  6507  6454  6069
    1149   1 2021-05-01  7803  7487  6894  6556  6599  6511  6553  6549  6403  6017
    1150   1 2021-05-01  8197  8205  7537  7147  7169  7030  7033  7187  6842  6381
    1151   1 2021-05-01  8774  8321  7663  7286  7316  7175  7186  7311  7005  6967
    1152   1 2021-05-01  8833  8902  8234  7876  7917  7756  7759  7952  7562  7028
    1153   1 2021-05-01  9229  9068  8409  8058  8111  7950  7957  8103  7774  7470
    1154   1 2021-05-01  9752  9514  8873  8543  8618  8456  8463  8522  8277  8020
    1155   1 2021-05-01 10135  9953  9292  8961  9034  8857  8857  8973  8659  8435
    1156   1 2021-05-01 10453 10407  9760  9464  9557  9376  9381  9494  9190  8805
    1157   1 2021-05-01 10827 10471  9844  9587  9707  9545  9555  9522  9397  9250
    1158   1 2021-05-01 10894 10471  9844  9587  9707  9545  9555  9522  9397  9362
    1159   1 2021-05-01 10753 10285  9678  9415  9537  9383  9402  9392  9247  9210
    1160   1 2021-05-01 10178  9987  9413  9158  9272  9140  9183  9176  9034  8657
    1161   1 2021-05-01 11792 11269 10574 10279 10300 10070 10132  9982  9957  8656
    1162   1 2021-05-01 11636 11269 10574 10279 10300 10070 10132  9982  9957  8502
    1163   1 2021-05-01 11606 11129 10441 10146 10166  9940  9999  9846  9826  8504
    1164   1 2021-05-01 11570 11033 10349 10062 10087  9867  9926  9753  9746  8524
    1165   1 2021-05-01 11091 10693 10044  9768  9776  9573  9650  9484  9485  7990
    1166   1 2021-05-01 10984 10548  9891  9602  9600  9401  9473  9335  9304  7863
    1167   1 2021-05-01 11186 10717 10041  9717  9724  9505  9562  9459  9381  8168
    1168   1 2021-05-01 11119 10603  9929  9598  9605  9395  9456  9332  9277  8106
    1169   1 2021-05-01 11065 10573  9891  9563  9587  9366  9415  9287  9226  8106
    1170   1 2021-05-01 10936 10370  9719  9418  9420  9213  9291  9118  9112  7931
    1171   1 2021-05-01 10925 10362  9710  9405  9405  9205  9275  9104  9104  7861
    1172   1 2021-05-01 10799 10678  9983  9644  9636  9406  9464  9380  9265  7571
    1173   1 2021-05-01 11457 11072 10364 10023 10021  9777  9819  9745  9608  8317
    1174   1 2021-05-01 11643 11300 10590 10244 10272 10025 10042  9948  9831  8583
    1175   1 2021-05-01 11860 11300 10590 10244 10272 10025 10042  9948  9831  8902
    1176   1 2021-05-01 11985 11468 10748 10400 10432 10159 10174 10099  9954  9134
    1177   1 2021-05-01 12104 11702 10952 10583 10625 10334 10332 10287 10103  9382
    1178   1 2021-05-01 12522 11997 11236 10868 10926 10628 10621 10576 10392  9899
    1179   1 2021-05-01 12627 12109 11324 10942 11009 10695 10670 10654 10435 10160
    1180   1 2021-05-01 12774 12267 11487 11111 11196 10880 10846 10832 10601 10357
    1181   1 2021-05-01 12741 12243 11488 11123 11203 10886 10870 10859 10614 10354
    1182   1 2021-05-01 12607 12174 11439 11084 11161 10861 10848 10817 10614 10154
    1183   1 2021-05-01 12521 12000 11269 10914 10985 10693 10675 10619 10436 10091
    1184   1 2021-05-01 12470 12052 11302 10942 11015 10716 10688 10667 10451 10128
    1185   1 2021-05-01 12567 11951 11216 10869 10960 10666 10650 10572 10421 10278
    1186   1 2021-05-01 12511 12015 11284 10940 11033 10734 10713 10657 10476 10305
    1187   1 2021-05-01 12469 11923 11202 10856 10941 10661 10639 10574 10402 10259
    1188   1 2021-05-01 12386 11873 11151 10804 10885 10613 10594 10538 10365 10210
    1189   1 2021-05-01 12332 11767 11048 10699 10788 10517 10497 10440 10266 10151
    1190   1 2021-05-01 10642 10179  9566  9288  9398  9213  9218  9206  9040  9150
    1191   1 2021-05-01 10727 10277  9658  9391  9502  9319  9317  9300  9142  9258
    1192   1 2021-05-01 10711 10185  9579  9329  9459  9266  9280  9207  9105  9263
    1193   1 2021-05-01 10723 10008  9428  9189  9318  9136  9153  9035  8992  9317
    1194   1 2021-05-01 10585 10130  9546  9319  9450  9277  9282  9185  9130  9162
    1195   1 2021-05-01 10561  9807  9250  9018  9145  8985  9016  8894  8860  9165
    1196   1 2021-05-01 10379  9877  9324  9093  9220  9066  9095  8999  8944  8950
    1197   1 2021-05-01 10255  9760  9185  8937  9061  8894  8913  8855  8770  8833
    1198   1 2021-05-01 10128  9734  9169  8933  9054  8894  8921  8860  8773  8661
    1199   1 2021-05-01 10080  9436  8901  8677  8798  8655  8691  8587  8546  8617
    1200   1 2021-05-01  9807  9406  8859  8621  8726  8588  8620  8552  8490  8313
    1201   1 2021-05-01 10070  9713  9120  8857  8960  8783  8801  8759  8648  8539
    1202   1 2021-05-01 10391 10029  9426  9170  9282  9094  9106  9044  8948  8854
    1203   1 2021-05-01 10429  9795  9183  8910  9019  8833  8849  8764  8699  8875
    1204   1 2021-05-01 10402 10014  9408  9146  9244  9067  9079  8996  8921  8805
    1205   1 2021-05-01 10275  9858  9265  9021  9123  8954  8970  8879  8806  8680
    1206   1 2021-05-01 10697  9850  9235  8969  9073  8905  8915  8805  8765  9138
    1207   1 2021-05-01 10739 10185  9550  9280  9385  9202  9202  9114  9046  9185
    1208   1 2021-05-01 10702 10192  9549  9263  9361  9161  9150  9109  8978  9159
    1209   1 2021-05-01 10850 10390  9747  9472  9574  9379  9374  9316  9207  9318
    1210   1 2021-05-01 10986 10329  9689  9412  9515  9309  9304  9252  9135  9485
    1211   1 2021-05-01 10911 10489  9859  9591  9699  9493  9488  9453  9311  9404
    1212   1 2021-05-01 10822 10293  9677  9411  9535  9338  9350  9272  9187  9343
    1213   1 2021-05-01 10766 10259  9638  9360  9462  9263  9276  9260  9102  9340
    1214   1 2021-05-01 10578 10201  9596  9342  9449  9253  9262  9245  9086  9203
    1215   1 2021-05-01 10747 10343  9722  9468  9572  9388  9395  9376  9218  9321
    1216   1 2021-05-01 10807 10265  9648  9386  9499  9316  9321  9272  9151  9411
    1217   1 2021-05-01 10704 10265  9648  9386  9499  9316  9321  9272  9151  9307
    1218   1 2021-05-01 10447  9966  9382  9129  9257  9087  9099  9008  8933  9078
    1219   1 2021-05-01 10261  9658  9077  8835  8960  8803  8823  8748  8675  8892
    1220   1 2021-05-01 10472  9871  9284  9040  9157  8997  9014  8936  8864  9111
    1221   1 2021-05-01 10339  9685  9098  8844  8962  8809  8835  8755  8684  9013
    1222   1 2021-05-01 10092  9559  8974  8721  8850  8698  8724  8630  8575  8764
    1223   1 2021-05-01  9889  9198  8663  8434  8559  8433  8461  8327  8328  8546
    1224   1 2021-05-01  9609  8829  8344  8148  8287  8188  8248  7982  8123  8240
    1225   1 2021-05-01  9030  8053  7564  7340  7463  7384  7462  7181  7352  7585
    1226   1 2021-05-01  8467  7863  7359  7118  7226  7154  7234  6960  7114  6872
    1227   1 2021-05-01  8222  7436  6915  6635  6703  6636  6708  6531  6580  6516
    1228   1 2021-05-01  8081  7735  7180  6891  6962  6874  6936  6775  6792  6296
    1229   1 2021-05-01  8149  7951  7353  7026  7078  6973  7006  6978  6850  6353
    1230   1 2021-05-01  8450  8310  7700  7372  7426  7304  7340  7340  7175  6648
    1231   1 2021-05-01  8924  8476  7835  7478  7529  7379  7404  7495  7217  7115
    1232   1 2021-05-01  9255  9068  8409  8058  8111  7950  7957  8103  7774  7492
    1233   1 2021-05-01  9621  9278  8623  8282  8344  8185  8192  8289  8014  7873
    1234   1 2021-05-01  9771  9827  9153  8809  8875  8693  8688  8839  8495  8055
    1235   1 2021-05-01 10403 10165  9513  9193  9273  9083  9075  9259  8879  8799
    1236   1 2021-05-01 10591 10465  9835  9564  9671  9511  9516  9601  9344  9073
    1237   1 2021-05-01 10722 10092  9490  9230  9335  9205  9226  9296  9072  9204
    1238   1 2021-05-01 10288  9825  9250  9001  9109  8994  9046  9037  8918  8766
    1239   1 2021-05-01  9935  8998  8473  8234  8341  8264  8335  8332  8213  8421
    1240   1 2021-05-01  9204  8889  8364  8107  8208  8135  8210  8199  8083  7687
    1241   1 2021-05-01 11590 11151 10455 10156 10162  9938  9997  9870  9817  8490
    1242   1 2021-05-01 11519 11102 10404 10104 10120  9889  9947  9832  9763  8464
    1243   1 2021-05-01 11464 10926 10253  9960  9974  9747  9805  9696  9628  8452
    1244   1 2021-05-01 11218 10586  9938  9656  9660  9457  9535  9369  9359  8143
    1245   1 2021-05-01 10920 10478  9826  9538  9537  9330  9406  9265  9237  7883
    1246   1 2021-05-01 10886 10498  9833  9515  9518  9313  9373  9259  9207  7858
    1247   1 2021-05-01 11014 10634  9934  9596  9609  9392  9433  9331  9246  8002
    1248   1 2021-05-01 11111 10634  9934  9596  9609  9392  9433  9331  9246  8177
    1249   1 2021-05-01 11137 10529  9860  9540  9556  9337  9390  9249  9220  8157
    1250   1 2021-05-01 11135 10579  9912  9587  9597  9383  9435  9285  9258  8102
    1251   1 2021-05-01 11091 10597  9910  9579  9575  9349  9410  9297  9225  7979
    1252   1 2021-05-01 11337 10905 10193  9845  9837  9598  9642  9580  9431  8165
    1253   1 2021-05-01 11554 11358 10637 10291 10301 10042 10067 10001  9847  8424
    1254   1 2021-05-01 12028 11539 10810 10451 10485 10216 10225 10154 10013  9148
    1255   1 2021-05-01 12132 11682 10933 10566 10605 10322 10311 10265 10084  9390
    1256   1 2021-05-01 12213 11820 11046 10655 10705 10405 10379 10379 10144  9554
    1257   1 2021-05-01 12394 12042 11264 10882 10951 10635 10616 10586 10374  9772
    1258   1 2021-05-01 12709 12307 11510 11112 11182 10856 10816 10853 10564 10304
    1259   1 2021-05-01 12898 12400 11641 11278 11368 11057 11033 10987 10799 10514
    1260   1 2021-05-01 12893 12164 11433 11084 11172 10877 10870 10791 10631 10532
    1261   1 2021-05-01 12709 12061 11324 10976 11053 10764 10754 10693 10517 10313
    1262   1 2021-05-01 12495 12023 11280 10926 10995 10701 10684 10642 10450 10098
    1263   1 2021-05-01 12426 12005 11260 10902 10973 10682 10656 10626 10429 10066
    1264   1 2021-05-01 12543 12080 11340 10985 11075 10769 10746 10712 10519 10243
    1265   1 2021-05-01 12545 12033 11292 10937 11022 10724 10694 10666 10461 10330
    1266   1 2021-05-01 12469 11995 11275 10928 11018 10727 10715 10658 10473 10251
    1267   1 2021-05-01 12413 11934 11208 10857 10942 10659 10648 10600 10406 10178
    1268   1 2021-05-01 12377 11864 11134 10795 10884 10618 10596 10538 10363 10188
    1269   1 2021-05-01 10761 10349  9728  9462  9577  9397  9397  9396  9226  9349
    1270   1 2021-05-01 10804 10363  9749  9493  9612  9417  9421  9408  9251  9416
    1271   1 2021-05-01 10798 10387  9777  9520  9645  9458  9458  9459  9296  9423
    1272   1 2021-05-01 10781 10310  9712  9476  9605  9423  9423  9406  9266  9399
    1273   1 2021-05-01 10703 10259  9659  9416  9545  9373  9377  9355  9217  9378
    1274   1 2021-05-01 10640 10116  9545  9306  9434  9260  9273  9247  9108  9278
    1275   1 2021-05-01 10487  9976  9404  9167  9283  9120  9144  9134  8987  9139
    1276   1 2021-05-01 10383  9916  9350  9114  9244  9078  9103  9051  8954  8977
    1277   1 2021-05-01 10285  9887  9324  9089  9207  9050  9074  9019  8924  8874
    1278   1 2021-05-01 10017  9599  9052  8820  8942  8785  8822  8755  8670  8593
    1279   1 2021-05-01  9944  9561  8999  8750  8856  8704  8730  8701  8580  8449
    1280   1 2021-05-01 10262  9814  9220  8970  9071  8898  8917  8875  8765  8749
    1281   1 2021-05-01 10478 10160  9558  9301  9416  9224  9230  9189  9067  8983
    1282   1 2021-05-01 10707 10224  9606  9347  9456  9268  9276  9232  9122  9221
    1283   1 2021-05-01 10692 10116  9526  9279  9390  9214  9231  9140  9070  9180
    1284   1 2021-05-01 10761 10282  9678  9414  9522  9336  9350  9292  9180  9267
    1285   1 2021-05-01 10744 10363  9740  9479  9581  9394  9398  9330  9230  9217
    1286   1 2021-05-01 10954 10499  9872  9614  9724  9522  9533  9466  9341  9417
    1287   1 2021-05-01 11053 10558  9920  9649  9761  9559  9562  9492  9396  9515
    1288   1 2021-05-01 11156 10792 10160  9886  9991  9785  9771  9749  9598  9618
    1289   1 2021-05-01 11175 10680 10046  9772  9887  9669  9665  9645  9490  9689
    1290   1 2021-05-01 11057 10680 10046  9772  9887  9669  9665  9645  9490  9611
    1291   1 2021-05-01 10962 10488  9866  9602  9718  9525  9524  9472  9350  9571
    1292   1 2021-05-01 10858 10439  9797  9517  9626  9426  9435  9416  9258  9465
    1293   1 2021-05-01 10751 10330  9712  9435  9537  9334  9337  9356  9168  9374
    1294   1 2021-05-01 10761 10387  9781  9520  9626  9432  9442  9446  9268  9381
    1295   1 2021-05-01 10812 10408  9777  9507  9612  9420  9415  9430  9235  9439
    1296   1 2021-05-01 10848 10301  9685  9428  9542  9352  9360  9315  9187  9467
    1297   1 2021-05-01 10742 10356  9753  9495  9613  9425  9428  9399  9253  9381
    1298   1 2021-05-01 10746 10095  9506  9255  9378  9209  9218  9175  9052  9418
    1299   1 2021-05-01 10477 10044  9464  9205  9321  9146  9155  9163  8987  9211
    1300   1 2021-05-01 10399  9777  9202  8961  9086  8934  8952  8857  8800  9106
    1301   1 2021-05-01 10155  9745  9181  8928  9049  8891  8898  8848  8739  8834
    1302   1 2021-05-01 10123  9255  8731  8510  8648  8533  8568  8390  8424  8804
    1303   1 2021-05-01  9632  9255  8731  8510  8648  8533  8568  8390  8424  8290
    1304   1 2021-05-01  9046  8452  7981  7788  7929  7845  7916  7627  7801  7621
    1305   1 2021-05-01  8781  8192  7704  7478  7595  7520  7596  7368  7465  7292
    1306   1 2021-05-01  8194  7704  7172  6908  6986  6925  7001  6833  6868  6657
    1307   1 2021-05-01  8400  7837  7305  7036  7105  7028  7097  7002  6960  6717
    1308   1 2021-05-01  8515  8022  7460  7171  7245  7149  7206  7082  7054  6759
    1309   1 2021-05-01  8649  8310  7700  7372  7426  7304  7340  7340  7175  6932
    1310   1 2021-05-01  8859  8688  8067  7739  7803  7671  7699  7717  7529  7100
    1311   1 2021-05-01  9397  9090  8466  8136  8205  8065  8080  8111  7907  7660
    1312   1 2021-05-01  9495  9331  8689  8360  8428  8283  8295  8370  8118  7801
    1313   1 2021-05-01  9848  9847  9181  8840  8909  8728  8722  8894  8533  8145
    1314   1 2021-05-01 10170  9710  9094  8794  8876  8721  8738  8842  8559  8536
    1315   1 2021-05-01 10282  9639  9053  8777  8869  8745  8762  8830  8603  8713
    1316   1 2021-05-01  9510  8903  8385  8141  8232  8163  8226  8209  8089  7907
    1317   1 2021-05-01  9198  9360  8799  8543  8631  8540  8595  8664  8456  7642
    1318   1 2021-05-01  9235  8794  8248  7974  8057  7983  8057  8102  7930  7643
    1319   1 2021-05-01  9097  8687  8181  7940  8048  7987  8069  8024  7949  7531
    1320   1 2021-05-01  9074  8605  8088  7844  7945  7893  7972  7928  7845  7547
    1321   1 2021-05-01  9532  9139  8562  8305  8403  8305  8355  8333  8209  8008
    1322   1 2021-05-01 11496 10917 10214  9912  9916  9699  9769  9644  9580  8368
    1323   1 2021-05-01 11344 11031 10323 10027 10032  9805  9872  9753  9678  8310
    1324   1 2021-05-01 11247 10824 10134  9839  9843  9622  9692  9577  9501  8195
    1325   1 2021-05-01 11082 10662 10000  9716  9710  9510  9583  9439  9407  8046
    1326   1 2021-05-01 10956 10478  9826  9538  9537  9330  9406  9265  9237  7946
    1327   1 2021-05-01 10779 10399  9740  9444  9444  9243  9312  9190  9152  7731
    1328   1 2021-05-01 10956 10560  9863  9531  9536  9316  9365  9277  9186  7962
    1329   1 2021-05-01 11053 10653  9938  9608  9618  9382  9425  9331  9229  8083
    1330   1 2021-05-01 11187 10705 10012  9666  9671  9450  9489  9372  9294  8215
    1331   1 2021-05-01 11337 10844 10156  9824  9833  9605  9646  9509  9456  8330
    1332   1 2021-05-01 11301 10825 10124  9790  9802  9575  9620  9486  9427  8269
    1333   1 2021-05-01 11428 11039 10337  9999 10016  9778  9821  9690  9624  8412
    1334   1 2021-05-01 11570 11419 10684 10324 10336 10066 10090 10042  9877  8455
    1335   1 2021-05-01 11957 11566 10802 10424 10446 10158 10159 10136  9917  8950
    1336   1 2021-05-01 12238 11672 10927 10557 10592 10315 10303 10252 10075  9549
    1337   1 2021-05-01 12335 11803 11040 10655 10713 10416 10403 10347 10158  9751
    1338   1 2021-05-01 12536 12040 11258 10864 10925 10611 10589 10579 10340 10030
    1339   1 2021-05-01 12711 12247 11451 11056 11128 10815 10780 10781 10544 10302
    1340   1 2021-05-01 12988 12470 11705 11335 11411 11106 11072 11050 10834 10595
    1341   1 2021-05-01 12856 12312 11568 11211 11299 11001 10978 10922 10747 10488
    1342   1 2021-05-01 12717 12164 11428 11081 11167 10871 10856 10792 10620 10328
    1343   1 2021-05-01 12575 12010 11290 10959 11043 10761 10748 10656 10506 10199
    1344   1 2021-05-01 12524 11990 11255 10908 10992 10706 10690 10615 10451 10182
    1345   1 2021-05-01 12547 12023 11284 10917 11005 10710 10673 10647 10441 10296
    1346   1 2021-05-01 12519 12034 11304 10955 11046 10763 10732 10680 10506 10296
    1347   1 2021-05-01 12474 11904 11182 10832 10918 10636 10617 10580 10387 10265
    1348   1 2021-05-01 12337 11880 11154 10812 10906 10623 10605 10561 10372 10084
    1349   1 2021-05-01 12310 11828 11098 10749 10836 10561 10543 10535 10307 10085
    1350   1 2021-05-01 10844 10396  9795  9542  9674  9487  9492  9510  9320  9452
    1351   1 2021-05-01 10714 10401  9792  9537  9659  9471  9477  9487  9311  9389
    1352   1 2021-05-01 10762 10367  9755  9505  9632  9444  9450  9471  9283  9425
    1353   1 2021-05-01 10743 10364  9760  9517  9647  9460  9463  9467  9298  9423
    1354   1 2021-05-01 10679 10265  9669  9421  9553  9380  9384  9402  9228  9383
    1355   1 2021-05-01 10470 10147  9561  9305  9435  9254  9268  9268  9093  9119
    1356   1 2021-05-01 10262  9878  9320  9087  9210  9050  9072  9052  8920  8915
    1357   1 2021-05-01 10075  9828  9244  8990  9100  8940  8963  8988  8816  8728
    1358   1 2021-05-01 10165  9698  9128  8868  8980  8823  8856  8856  8698  8750
    1359   1 2021-05-01 10119  9645  9083  8845  8956  8802  8832  8784  8687  8700
    1360   1 2021-05-01 10323  9998  9412  9153  9262  9083  9100  9109  8944  8861
    1361   1 2021-05-01 10508 10181  9571  9310  9429  9239  9252  9239  9086  9055
    1362   1 2021-05-01 10636 10335  9736  9474  9589  9387  9387  9355  9221  9182
    1363   1 2021-05-01 10779 10487  9877  9620  9731  9533  9526  9504  9357  9313
    1364   1 2021-05-01 10981 10350  9741  9481  9594  9407  9411  9368  9248  9524
    1365   1 2021-05-01 11036 10514  9901  9640  9756  9563  9558  9518  9393  9570
    1366   1 2021-05-01 10960 10457  9855  9593  9702  9514  9521  9478  9345  9476
    1367   1 2021-05-01 10999 10624  9994  9717  9824  9615  9618  9603  9428  9500
    1368   1 2021-05-01 11195 10686 10048  9772  9877  9662  9665  9653  9477  9729
    1369   1 2021-05-01 11270 10817 10178  9903 10008  9798  9792  9800  9622  9837
    1370   1 2021-05-01 11252 10790 10149  9876  9987  9784  9776  9761  9613  9844
    1371   1 2021-05-01 11169 10640 10015  9752  9872  9680  9683  9641  9523  9803
    1372   1 2021-05-01 10946 10510  9884  9620  9737  9553  9559  9530  9387  9589
    1373   1 2021-05-01 10957 10524  9897  9630  9748  9552  9560  9556  9377  9615
    1374   1 2021-05-01 10899 10483  9868  9601  9719  9527  9525  9532  9352  9561
    1375   1 2021-05-01 10839 10423  9789  9515  9630  9426  9428  9439  9249  9514
    1376   1 2021-05-01 10844 10504  9861  9581  9702  9485  9472  9505  9293  9484
    1377   1 2021-05-01 10874 10473  9843  9561  9665  9467  9453  9535  9276  9544
    1378   1 2021-05-01 10738 10473  9843  9561  9665  9467  9453  9535  9276  9478
    1379   1 2021-05-01 10642 10234  9641  9392  9510  9326  9331  9321  9158  9390
    1380   1 2021-05-01 10502 10062  9483  9234  9350  9184  9190  9169  9027  9247
    1381   1 2021-05-01 10298  9896  9312  9056  9177  9005  9020  9013  8862  9045
    1382   1 2021-05-01 10223  9851  9255  8982  9098  8928  8948  8983  8787  8977
    1383   1 2021-05-01 10176  9615  9069  8837  8969  8830  8848  8747  8690  8912
    1384   1 2021-05-01  9849  9203  8680  8456  8590  8470  8504  8374  8368  8559
    1385   1 2021-05-01  9385  8470  7978  7759  7879  7790  7861  7655  7727  8008
    1386   1 2021-05-01  8890  8384  7909  7695  7812  7729  7804  7559  7677  7451
    1387   1 2021-05-01  8306  7960  7475  7249  7348  7278  7359  7176  7244  6828
    1388   1 2021-05-01  8402  8084  7570  7320  7410  7327  7395  7269  7258  6831
    1389   1 2021-05-01  8772  8254  7734  7489  7579  7503  7567  7469  7440  7235
    1390   1 2021-05-01  8766  8547  7969  7686  7761  7662  7707  7667  7563  7178
    1391   1 2021-05-01  9036  8735  8179  7914  8005  7905  7950  7847  7807  7389
    1392   1 2021-05-01  9162  8855  8246  7933  8000  7875  7911  7940  7750  7535
    1393   1 2021-05-01  9216  8964  8344  8028  8096  7968  7998  8059  7837  7558
    1394   1 2021-05-01  9632  9507  8863  8536  8612  8466  8482  8570  8302  7981
    1395   1 2021-05-01  9775  9343  8726  8422  8505  8367  8395  8449  8231  8129
    1396   1 2021-05-01  9530  9382  8783  8483  8559  8430  8453  8506  8289  7897
    1397   1 2021-05-01  9525  9054  8509  8242  8325  8217  8258  8219  8104  7846
    1398   1 2021-05-01  9022  8581  8084  7847  7936  7881  7972  7880  7842  7404
    1399   1 2021-05-01  8979  8782  8283  8056  8166  8097  8176  8058  8051  7363
    1400   1 2021-05-01  9106  8687  8196  7974  8091  8033  8113  8002  7985  7535
    1401   1 2021-05-01  9110  9114  8542  8287  8391  8290  8347  8341  8203  7602
    1402   1 2021-05-01  9727  9422  8852  8591  8699  8586  8638  8647  8489  8260
    1403   1 2021-05-01  9926  9735  9127  8868  8969  8838  8869  8871  8715  8469
    1404   1 2021-05-01 10204  9860  9257  8995  9106  8965  8993  8987  8832  8712
    1405   1 2021-05-01 11221 10714 10017  9717  9714  9514  9585  9451  9397  8042
    1406   1 2021-05-01 11240 10685  9998  9698  9701  9488  9561  9429  9373  8163
    1407   1 2021-05-01 10947 10544  9873  9573  9565  9348  9420  9317  9233  7931
    1408   1 2021-05-01 10857 10381  9715  9413  9415  9210  9272  9150  9097  7852
    1409   1 2021-05-01 10717 10247  9606  9316  9312  9115  9195  9068  9016  7666
    1410   1 2021-05-01 10620 10206  9542  9240  9229  9031  9106  8984  8931  7539
    1411   1 2021-05-01 10893 10514  9800  9471  9479  9243  9295  9203  9106  7895
    1412   1 2021-05-01 11093 10769 10051  9716  9724  9489  9525  9432  9334  8104
    1413   1 2021-05-01 11363 10770 10060  9714  9728  9497  9543  9411  9343  8405
    1414   1 2021-05-01 11489 11027 10319  9986  9996  9756  9793  9673  9597  8514
    1415   1 2021-05-01 11653 11040 10343 10023 10048  9809  9849  9692  9654  8670
    1416   1 2021-05-01 11750 11224 10530 10202 10231  9985 10024  9873  9827  8802
    1417   1 2021-05-01 11721 11247 10531 10186 10204  9952  9983  9864  9785  8700
    1418   1 2021-05-01 11881 11282 10527 10157 10182  9907  9921  9836  9697  8950
    1419   1 2021-05-01 11995 11769 10990 10595 10627 10322 10307 10313 10056  9197
    1420   1 2021-05-01 12508 12039 11252 10853 10909 10585 10564 10572 10307  9999
    1421   1 2021-05-01 12764 12355 11568 11175 11249 10923 10889 10905 10634 10295
    1422   1 2021-05-01 12931 12494 11718 11346 11420 11114 11081 11067 10829 10495
    1423   1 2021-05-01 12972 12435 11664 11305 11381 11073 11048 11028 10803 10565
    1424   1 2021-05-01 12779 12247 11500 11146 11236 10932 10911 10863 10676 10410
    1425   1 2021-05-01 12665 12146 11412 11063 11148 10854 10835 10777 10593 10277
    1426   1 2021-05-01 12613 12063 11341 10998 11077 10787 10775 10713 10534 10236
    1427   1 2021-05-01 12555 12063 11341 10998 11077 10787 10775 10713 10534 10165
    1428   1 2021-05-01 12589 12081 11331 10964 11043 10746 10712 10704 10479 10370
    1429   1 2021-05-01 12625 12115 11353 10994 11081 10785 10757 10736 10526 10423
    1430   1 2021-05-01 12525 11975 11239 10900 10991 10704 10671 10620 10452 10359
    1431   1 2021-05-01 12451 12041 11301 10965 11056 10766 10738 10695 10505 10289
    1432   1 2021-05-01 12366 11852 11133 10805 10912 10626 10620 10533 10394 10143
    1433   1 2021-05-01 12313 11828 11098 10745 10832 10546 10531 10536 10292 10114
    1434   1 2021-05-01 10807 10396  9794  9540  9670  9493  9501  9509  9336  9491
    1435   1 2021-05-01 10719 10316  9713  9460  9599  9410  9418  9442  9252  9418
    1436   1 2021-05-01 10727 10320  9715  9458  9597  9410  9418  9435  9245  9417
    1437   1 2021-05-01 10566 10089  9515  9270  9406  9241  9260  9251  9102  9274
    1438   1 2021-05-01 10279  9892  9321  9072  9193  9037  9060  9047  8913  9005
    1439   1 2021-05-01 10382  9923  9366  9131  9260  9090  9116  9082  8964  9039
    1440   1 2021-05-01 10354  9879  9326  9095  9223  9064  9092  9042  8938  9006
    1441   1 2021-05-01 10081  9658  9107  8876  8995  8855  8903  8831  8747  8717
    1442   1 2021-05-01 10115  9658  9107  8876  8995  8855  8903  8831  8747  8697
    1443   1 2021-05-01 10138  9763  9202  8962  9069  8916  8940  8906  8796  8706
    1444   1 2021-05-01 10324  9998  9412  9153  9262  9083  9100  9109  8944  8869
    1445   1 2021-05-01 10602 10158  9565  9303  9420  9244  9254  9255  9090  9186
    1446   1 2021-05-01 10810 10363  9743  9473  9580  9382  9380  9388  9202  9358
    1447   1 2021-05-01 10984 10566  9938  9663  9766  9555  9548  9575  9366  9575
    1448   1 2021-05-01 10976 10612  9989  9720  9829  9619  9600  9626  9427  9556
    1449   1 2021-05-01 11031 10644 10014  9736  9833  9624  9615  9643  9435  9587
    1450   1 2021-05-01 11085 10690 10055  9784  9891  9673  9673  9682  9484  9661
    1451   1 2021-05-01 11156 10784 10148  9873  9982  9770  9757  9767  9575  9733
    1452   1 2021-05-01 11211 10730 10107  9853  9975  9773  9777  9741  9613  9760
    1453   1 2021-05-01 11225 10732 10105  9839  9968  9765  9772  9747  9596  9820
    1454   1 2021-05-01 11173 10673 10052  9793  9912  9720  9726  9701  9561  9809
    1455   1 2021-05-01 11043 10536  9930  9675  9796  9599  9611  9601  9441  9706
    1456   1 2021-05-01 10911 10511  9909  9654  9765  9585  9585  9573  9426  9588
    1457   1 2021-05-01 10923 10524  9897  9630  9748  9552  9560  9556  9377  9586
    1458   1 2021-05-01 10976 10524  9905  9645  9767  9568  9571  9555  9384  9654
    1459   1 2021-05-01 10896 10498  9865  9592  9711  9518  9514  9534  9337  9589
    1460   1 2021-05-01 10968 10582  9937  9653  9763  9560  9546  9585  9361  9612
    1461   1 2021-05-01 10981 10448  9812  9540  9659  9466  9475  9474  9292  9661
    1462   1 2021-05-01 10735 10328  9683  9403  9520  9330  9344  9357  9165  9508
    1463   1 2021-05-01 10648 10228  9634  9366  9474  9291  9277  9316  9097  9422
    1464   1 2021-05-01 10516 10001  9434  9197  9316  9164  9171  9141  9009  9299
    1465   1 2021-05-01 10308  9784  9201  8941  9059  8907  8921  8914  8762  9130
    1466   1 2021-05-01 10097  9666  9087  8820  8931  8772  8794  8823  8631  8871
    1467   1 2021-05-01  9961  9425  8871  8631  8754  8618  8647  8583  8494  8743
    1468   1 2021-05-01  9864  9569  8986  8724  8834  8681  8695  8651  8535  8590
    1469   1 2021-05-01  9850  9049  8523  8292  8421  8302  8336  8152  8195  8496
    1470   1 2021-05-01  9625  8384  7909  7695  7812  7729  7804  7559  7677  8192
    1471   1 2021-05-01  9266  8638  8134  7910  8031  7926  7996  7763  7865  7778
    1472   1 2021-05-01  9434  8446  7935  7701  7802  7707  7765  7596  7636  7953
    1473   1 2021-05-01  9098  8590  8081  7853  7957  7872  7932  7785  7801  7601
    1474   1 2021-05-01  8925  8564  8029  7775  7867  7781  7828  7742  7687  7402
    1475   1 2021-05-01  9405  8980  8420  8155  8249  8139  8177  8086  8029  7811
    1476   1 2021-05-01  9614  8944  8386  8120  8226  8109  8154  8016  8013  8041
    1477   1 2021-05-01  9541  9162  8606  8347  8460  8337  8375  8253  8233  7903
    1478   1 2021-05-01  9405  9090  8518  8249  8344  8224  8281  8231  8135  7776
    1479   1 2021-05-01  9689  9214  8650  8381  8485  8365  8403  8294  8261  8024
    1480   1 2021-05-01  9528  9255  8666  8379  8471  8345  8383  8354  8218  7860
    1481   1 2021-05-01  9719  9697  9095  8819  8925  8779  8804  8750  8637  8021
    1482   1 2021-05-01  9592  9405  8862  8625  8733  8616  8672  8534  8522  7930
    1483   1 2021-05-01  9942  8782  8283  8056  8166  8097  8176  8058  8051  8358
    1484   1 2021-05-01  9772  9154  8627  8398  8516  8433  8499  8363  8371  8211
    1485   1 2021-05-01  9705  9182  8630  8386  8505  8409  8471  8413  8322  8174
    1486   1 2021-05-01  9816  9406  8845  8591  8708  8595  8650  8620  8502  8327
    1487   1 2021-05-01 10081  9670  9073  8800  8898  8767  8805  8856  8652  8605
    1488   1 2021-05-01 10243  9843  9231  8965  9072  8934  8963  8987  8808  8764
    1489   1 2021-05-01 11014 10559  9870  9568  9573  9369  9433  9291  9243  7924
    1490   1 2021-05-01 10970 10463  9770  9468  9471  9265  9331  9187  9151  7922
    1491   1 2021-05-01 10918 10488  9805  9498  9500  9281  9351  9226  9170  7910
    1492   1 2021-05-01 10894 10415  9745  9443  9453  9246  9306  9156  9144  7937
    1493   1 2021-05-01 10669 10182  9541  9253  9257  9071  9144  8980  8969  7676
    1494   1 2021-05-01 10622 10154  9504  9209  9214  9019  9100  8933  8926  7585
    1495   1 2021-05-01 10748 10403  9704  9382  9381  9164  9222  9117  9037  7747
    1496   1 2021-05-01 11042 10696  9989  9656  9660  9432  9472  9386  9285  8059
    1497   1 2021-05-01 11609 11109 10384 10050 10056  9822  9857  9757  9656  8651
    1498   1 2021-05-01 11792 11311 10590 10257 10275 10025 10061  9945  9856  8854
    1499   1 2021-05-01 11848 11342 10642 10310 10339 10084 10118  9985  9901  8925
    1500   1 2021-05-01 11832 11356 10649 10304 10348 10090 10122  9966  9905  8891
    1501   1 2021-05-01 11841 11312 10579 10237 10272 10018 10039  9876  9835  8945
    1502   1 2021-05-01 11937 11620 10823 10416 10448 10143 10124 10139  9879  9211
    1503   1 2021-05-01 12263 12039 11265 10881 10933 10632 10620 10592 10373  9614
    1504   1 2021-05-01 12552 12025 11242 10853 10907 10601 10593 10565 10352 10021
    1505   1 2021-05-01 12777 12301 11514 11113 11178 10848 10825 10857 10567 10262
    1506   1 2021-05-01 12822 12493 11720 11343 11418 11108 11072 11082 10826 10305
    1507   1 2021-05-01 12843 12371 11607 11240 11312 11007 10983 10977 10751 10410
    1508   1 2021-05-01 12761 12318 11546 11178 11253 10945 10923 10922 10676 10387
    1509   1 2021-05-01 12662 12227 11477 11116 11197 10893 10878 10856 10628 10319
    1510   1 2021-05-01 12608 12099 11372 11025 11099 10809 10795 10742 10561 10251
    1511   1 2021-05-01 12559 12108 11348 11002 11082 10797 10786 10731 10551 10243
    1512   1 2021-05-01 12632 12181 11422 11066 11150 10857 10837 10803 10599 10401
    1513   1 2021-05-01 12698 12192 11443 11090 11181 10885 10873 10828 10631 10473
    1514   1 2021-05-01 12662 12155 11405 11062 11161 10870 10845 10801 10605 10486
    1515   1 2021-05-01 12617 12072 11347 11015 11115 10828 10809 10745 10568 10463
    1516   1 2021-05-01 12598 12034 11296 10954 11044 10762 10734 10692 10507 10416
    1517   1 2021-05-01 12510 12091 11338 10987 11075 10788 10756 10771 10516 10319
    1518   1 2021-05-01 10844 10429  9840  9591  9726  9548  9555  9561  9393  9548
    1519   1 2021-05-01 10789 10340  9739  9481  9625  9440  9445  9447  9282  9502
    1520   1 2021-05-01 10775 10340  9739  9481  9625  9440  9445  9447  9282  9468
    1521   1 2021-05-01 10371 10050  9485  9250  9386  9230  9254  9216  9102  9069
    1522   1 2021-05-01 10405  9945  9379  9139  9269  9111  9146  9109  9001  9088
    1523   1 2021-05-01 10476 10024  9456  9215  9340  9171  9198  9177  9048  9179
    1524   1 2021-05-01 10437 10058  9487  9237  9362  9201  9217  9181  9057  9127
    1525   1 2021-05-01 10302  9877  9328  9094  9216  9068  9093  9035  8937  8956
    1526   1 2021-05-01 10349  9897  9346  9108  9228  9069  9098  9044  8950  8979
    1527   1 2021-05-01 10402  9996  9425  9168  9274  9094  9121  9098  8958  9037
    1528   1 2021-05-01 10418 10089  9498  9235  9352  9175  9194  9200  9023  9020
    1529   1 2021-05-01 10543 10361  9736  9462  9580  9387  9389  9388  9210  9122
    1530   1 2021-05-01 11029 10627  9980  9699  9813  9613  9607  9607  9433  9623
    1531   1 2021-05-01 11165 10766 10111  9832  9944  9739  9731  9737  9565  9786
    1532   1 2021-05-01 11175 10780 10132  9850  9957  9748  9744  9733  9570  9773
    1533   1 2021-05-01 11106 10780 10132  9850  9957  9748  9744  9733  9570  9693
    1534   1 2021-05-01 11245 10749 10107  9822  9925  9705  9699  9717  9513  9862
    1535   1 2021-05-01 11207 10814 10185  9907 10020  9801  9802  9815  9616  9839
    1536   1 2021-05-01 11196 10753 10146  9885 10006  9798  9790  9779  9606  9800
    1537   1 2021-05-01 11086 10699 10078  9804  9927  9714  9710  9727  9519  9704
    1538   1 2021-05-01 11082 10686 10060  9797  9925  9728  9742  9711  9569  9745
    1539   1 2021-05-01 10981 10536  9930  9675  9796  9599  9611  9601  9441  9659
    1540   1 2021-05-01 10892 10513  9891  9625  9738  9538  9538  9563  9364  9590
    1541   1 2021-05-01 10923 10551  9932  9659  9770  9578  9564  9589  9386  9634
    1542   1 2021-05-01 10957 10513  9884  9616  9727  9538  9530  9568  9348  9661
    1543   1 2021-05-01 10829 10508  9869  9583  9692  9485  9480  9548  9293  9572
    1544   1 2021-05-01 10875 10451  9806  9517  9629  9409  9395  9456  9202  9595
    1545   1 2021-05-01 10851 10418  9783  9510  9626  9432  9435  9437  9256  9582
    1546   1 2021-05-01 10828 10461  9818  9545  9676  9484  9492  9477  9321  9603
    1547   1 2021-05-01 10693 10343  9704  9408  9526  9324  9323  9365  9150  9462
    1548   1 2021-05-01 10486 10135  9548  9293  9413  9240  9245  9236  9071  9295
    1549   1 2021-05-01 10094  9690  9122  8872  8995  8851  8869  8848  8725  8930
    1550   1 2021-05-01  9993  9533  8949  8679  8789  8640  8672  8703  8516  8809
    1551   1 2021-05-01  9830  9473  8904  8645  8761  8620  8642  8660  8492  8619
    1552   1 2021-05-01  9904  9569  8986  8724  8834  8681  8695  8651  8535  8667
    1553   1 2021-05-01 10154  9704  9127  8871  8990  8837  8846  8825  8686  8873
    1554   1 2021-05-01 10069  9345  8798  8569  8699  8568  8612  8428  8468  8709
    1555   1 2021-05-01  9901  9328  8783  8553  8679  8561  8596  8474  8453  8564
    1556   1 2021-05-01  9536  9009  8474  8235  8348  8233  8286  8193  8136  8158
    1557   1 2021-05-01  9455  9119  8547  8273  8366  8242  8275  8272  8126  8007
    1558   1 2021-05-01  9480  9239  8663  8404  8511  8379  8410  8326  8264  8014
    1559   1 2021-05-01  9766  9510  8928  8668  8781  8636  8672  8606  8519  8307
    1560   1 2021-05-01  9842  9426  8852  8591  8705  8577  8611  8570  8460  8352
    1561   1 2021-05-01  9775  9426  8852  8591  8705  8577  8611  8570  8460  8227
    1562   1 2021-05-01  9639  9217  8664  8418  8524  8408  8459  8369  8315  8125
    1563   1 2021-05-01  9827  9464  8874  8607  8708  8580  8613  8565  8465  8230
    1564   1 2021-05-01 10124  9550  8951  8666  8767  8630  8655  8608  8495  8507
    1565   1 2021-05-01 10232  9697  9095  8819  8925  8779  8804  8750  8637  8610
    1566   1 2021-05-01 10371  9908  9312  9051  9165  9031  9062  8992  8910  8788
    1567   1 2021-05-01 10259  9613  9063  8833  8956  8833  8890  8788  8746  8677
    1568   1 2021-05-01 10251  9833  9259  9016  9133  8999  9045  8980  8894  8745
    1569   1 2021-05-01 10323  9615  9043  8801  8920  8807  8856  8766  8719  8850
    1570   1 2021-05-01 10264  9900  9304  9051  9162  9022  9055  9017  8905  8721
    1571   1 2021-05-01 10257  9869  9277  9022  9141  8998  9042  9005  8897  8767
    1572   1 2021-05-01 10941 10509  9801  9479  9485  9266  9346  9201  9157  7728
    1573   1 2021-05-01 11057 10563  9856  9551  9554  9341  9402  9257  9211  8054
    1574   1 2021-05-01 10936 10454  9773  9484  9487  9290  9368  9187  9184  7920
    1575   1 2021-05-01 10880 10473  9786  9481  9485  9276  9333  9211  9165  7881
    1576   1 2021-05-01 10908 10352  9679  9384  9388  9190  9261  9114  9102  7935
    1577   1 2021-05-01 10788 10238  9597  9308  9322  9122  9195  9021  9037  7840
    1578   1 2021-05-01 10769 10304  9643  9351  9369  9166  9240  9058  9059  7803
    1579   1 2021-05-01 10875 10619  9916  9587  9583  9357  9404  9316  9206  7893
    1580   1 2021-05-01 11213 10985 10267  9939  9943  9703  9743  9649  9534  8215
    1581   1 2021-05-01 11562 11323 10600 10272 10288 10041 10068  9956  9850  8577
    1582   1 2021-05-01 11922 11323 10600 10272 10288 10041 10068  9956  9850  9012
    1583   1 2021-05-01 11972 11415 10704 10367 10398 10137 10167 10048  9956  9059
    1584   1 2021-05-01 11937 11356 10649 10304 10348 10090 10122  9966  9905  9059
    1585   1 2021-05-01 11991 11456 10718 10357 10402 10132 10143 10030  9925  9174
    1586   1 2021-05-01 12327 11609 10824 10423 10464 10174 10174 10121  9940  9670
    1587   1 2021-05-01 12530 12003 11237 10854 10907 10610 10590 10555 10344  9932
    1588   1 2021-05-01 12579 12153 11362 10962 11020 10709 10684 10697 10415 10078
    1589   1 2021-05-01 12666 12215 11451 11076 11136 10833 10813 10799 10558 10166
    1590   1 2021-05-01 12654 12230 11468 11098 11165 10856 10827 10825 10573 10211
    1591   1 2021-05-01 12731 12283 11508 11147 11224 10921 10892 10863 10641 10350
    1592   1 2021-05-01 12778 12277 11523 11169 11248 10942 10912 10885 10661 10445
    1593   1 2021-05-01 12720 12216 11476 11135 11225 10927 10904 10849 10667 10409
    1594   1 2021-05-01 12631 12096 11339 11001 11083 10796 10781 10740 10543 10268
    1595   1 2021-05-01 12626 12172 11414 11061 11150 10862 10840 10808 10592 10345
    1596   1 2021-05-01 12680 12186 11427 11068 11155 10861 10848 10821 10601 10435
    1597   1 2021-05-01 12699 12192 11443 11090 11181 10885 10873 10828 10631 10470
    1598   1 2021-05-01 12701 12232 11476 11122 11214 10923 10897 10874 10658 10524
    1599   1 2021-05-01 12653 12149 11409 11074 11173 10887 10855 10812 10627 10454
    1600   1 2021-05-01 12508 12017 11277 10937 11031 10758 10729 10670 10509 10298
    1601   1 2021-05-01 12490 12055 11298 10942 11024 10740 10711 10729 10491 10263
    1602   1 2021-05-01 12495 11923 11192 10851 10928 10656 10643 10644 10429 10274
    1603   1 2021-05-01 10808 10453  9849  9593  9727  9553  9555  9595  9396  9539
    1604   1 2021-05-01 10831 10422  9824  9571  9708  9527  9537  9578  9368  9549
    1605   1 2021-05-01 10761 10357  9759  9506  9641  9460  9470  9469  9304  9442
    1606   1 2021-05-01 10494  9914  9353  9115  9256  9104  9140  9117  9003  9176
    1607   1 2021-05-01 10259  9916  9343  9091  9215  9063  9087  9118  8936  8985
    1608   1 2021-05-01 10293 10091  9496  9232  9353  9190  9199  9268  9038  9005
    1609   1 2021-05-01 10633 10236  9652  9396  9526  9354  9361  9348  9202  9328
    1610   1 2021-05-01 10723 10226  9645  9393  9521  9350  9353  9306  9185  9382
    1611   1 2021-05-01 10696 10190  9617  9377  9512  9339  9362  9291  9203  9333
    1612   1 2021-05-01 10596 10070  9474  9202  9312  9135  9156  9158  8988  9281
    1613   1 2021-05-01 10618 10070  9474  9202  9312  9135  9156  9158  8988  9267
    1614   1 2021-05-01 10670 10160  9567  9304  9419  9228  9244  9237  9068  9287
    1615   1 2021-05-01 10821 10414  9788  9518  9631  9426  9426  9451  9246  9426
    1616   1 2021-05-01 10990 10613  9982  9711  9827  9627  9626  9618  9445  9618
    1617   1 2021-05-01 11109 10687 10043  9769  9879  9679  9667  9687  9493  9764
    1618   1 2021-05-01 11244 10871 10221  9938 10048  9828  9807  9858  9628  9881
    1619   1 2021-05-01 11200 10837 10176  9887  9990  9771  9748  9839  9552  9861
    1620   1 2021-05-01 11171 10783 10149  9858  9969  9747  9735  9802  9532  9822
    1621   1 2021-05-01 10926 10555  9919  9630  9729  9525  9526  9620  9350  9635
    1622   1 2021-05-01 10974 10635  9992  9687  9775  9545  9522  9668  9316  9621
    1623   1 2021-05-01 10915 10709 10080  9798  9910  9696  9681  9732  9488  9591
    1624   1 2021-05-01 11019 10570  9933  9647  9747  9533  9501  9596  9307  9703
    1625   1 2021-05-01 10979 10522  9893  9611  9717  9504  9498  9569  9319  9660
    1626   1 2021-05-01 10801 10462  9823  9535  9640  9431  9414  9494  9229  9549
    1627   1 2021-05-01 10841 10527  9903  9623  9732  9531  9514  9565  9335  9587
    1628   1 2021-05-01 10831 10436  9809  9535  9647  9451  9442  9481  9257  9582
    1629   1 2021-05-01 10764 10417  9760  9469  9573  9380  9383  9447  9215  9530
    1630   1 2021-05-01 10813 10429  9770  9462  9571  9366  9359  9433  9182  9610
    1631   1 2021-05-01 10902 10480  9839  9557  9672  9471  9460  9491  9281  9689
    1632   1 2021-05-01 10931 10518  9906  9646  9779  9573  9574  9566  9398  9724
    1633   1 2021-05-01 10932 10280  9676  9423  9556  9389  9408  9363  9244  9734
    1634   1 2021-05-01 10302  9895  9348  9121  9262  9114  9145  9063  8994  9167
    1635   1 2021-05-01  9876  9441  8894  8662  8799  8666  8720  8663  8579  8752
    1636   1 2021-05-01  9788  9387  8833  8579  8696  8556  8582  8591  8442  8619
    1637   1 2021-05-01  9848  9463  8906  8662  8791  8650  8686  8622  8538  8663
    1638   1 2021-05-01 10011  9607  9026  8765  8886  8734  8748  8779  8594  8785
    1639   1 2021-05-01 10034  9629  9067  8817  8948  8800  8825  8810  8671  8828
    1640   1 2021-05-01 10041  9650  9089  8848  8973  8832  8851  8814  8701  8830
    1641   1 2021-05-01  9973  9488  8930  8699  8822  8702  8723  8646  8581  8683
    1642   1 2021-05-01  9590  9051  8515  8276  8395  8292  8336  8269  8192  8257
    1643   1 2021-05-01  9434  8945  8380  8100  8203  8085  8126  8159  7975  8030
    1644   1 2021-05-01  9697  9406  8810  8529  8626  8482  8503  8532  8340  8312
    1645   1 2021-05-01  9997  9613  9025  8760  8873  8723  8754  8757  8583  8629
    1646   1 2021-05-01  9924  9432  8876  8624  8739  8610  8648  8612  8503  8503
    1647   1 2021-05-01  9611  9211  8688  8449  8566  8457  8510  8450  8375  8189
    1648   1 2021-05-01  9705  9443  8860  8588  8684  8562  8595  8609  8440  8222
    1649   1 2021-05-01 10057  9788  9192  8922  9028  8877  8914  8888  8752  8504
    1650   1 2021-05-01 10280 10031  9442  9178  9287  9146  9174  9160  9016  8726
    1651   1 2021-05-01 10517 10033  9443  9189  9299  9155  9179  9172  9027  9040
    1652   1 2021-05-01 10439  9996  9429  9189  9307  9179  9216  9176  9071  8956
    1653   1 2021-05-01 10312  9970  9380  9124  9226  9091  9126  9128  8970  8788
    1654   1 2021-05-01 10363  9970  9380  9124  9226  9091  9126  9128  8970  8852
    1655   1 2021-05-01 10135  9927  9347  9108  9220  9098  9138  9089  8995  8663
    1656   1 2021-05-01 10403  9984  9379  9120  9224  9092  9115  9137  8968  8910
    1657   1 2021-05-01 10905 10491  9767  9432  9436  9215  9259  9155  9081  7822
    1658   1 2021-05-01 11065 10492  9820  9524  9546  9343  9405  9223  9233  8101
    1659   1 2021-05-01 10923 10421  9748  9462  9480  9285  9360  9164  9188  7937
    1660   1 2021-05-01 10835 10318  9642  9362  9370  9183  9253  9088  9095  7836
    1661   1 2021-05-01 10722 10318  9642  9362  9370  9183  9253  9088  9095  7759
    1662   1 2021-05-01 10839 10383  9712  9418  9429  9224  9285  9151  9112  7895
    1663   1 2021-05-01 10875 10425  9762  9482  9493  9290  9363  9209  9183  7945
    1664   1 2021-05-01 11088 10554  9856  9533  9537  9322  9379  9250  9182  8130
    1665   1 2021-05-01 11466 10967 10245  9909  9904  9666  9695  9632  9487  8523
    1666   1 2021-05-01 11858 11395 10670 10350 10372 10121 10141 10027  9929  8980
    1667   1 2021-05-01 12006 11491 10777 10447 10475 10217 10235 10109 10019  9112
    1668   1 2021-05-01 12034 11517 10794 10454 10499 10222 10233 10101 10014  9216
    1669   1 2021-05-01 12050 11505 10761 10394 10436 10153 10156 10070  9939  9311
    1670   1 2021-05-01 12112 11663 10909 10532 10577 10279 10270 10223 10047  9332
    1671   1 2021-05-01 12328 12027 11245 10860 10911 10610 10602 10576 10356  9609
    1672   1 2021-05-01 12612 12118 11337 10961 11019 10724 10705 10669 10462 10060
    1673   1 2021-05-01 12613 12201 11441 11067 11123 10824 10792 10783 10525 10102
    1674   1 2021-05-01 12676 12201 11441 11067 11123 10824 10792 10783 10525 10151
    1675   1 2021-05-01 12668 12198 11426 11059 11128 10825 10806 10768 10553 10184
    1676   1 2021-05-01 12790 12313 11535 11169 11251 10941 10914 10881 10654 10399
    1677   1 2021-05-01 12789 12332 11559 11193 11275 10967 10941 10921 10693 10507
    1678   1 2021-05-01 12783 12347 11583 11222 11303 10991 10956 10963 10710 10499
    1679   1 2021-05-01 12579 12120 11384 11039 11127 10841 10826 10759 10593 10275
    1680   1 2021-05-01 12595 12172 11414 11061 11150 10862 10840 10808 10592 10297
    1681   1 2021-05-01 12797 12288 11532 11177 11277 10980 10965 10930 10717 10598
    1682   1 2021-05-01 12773 12281 11525 11174 11275 10979 10957 10923 10717 10598
    1683   1 2021-05-01 12767 12231 11490 11154 11253 10969 10946 10898 10719 10577
    1684   1 2021-05-01 12492 11994 11262 10922 11016 10735 10727 10661 10498 10253
    1685   1 2021-05-01 12431 11849 11142 10811 10903 10632 10622 10540 10405 10219
    1686   1 2021-05-01 12371 11844 11129 10804 10917 10657 10647 10499 10449 10198
    1687   1 2021-05-01 10701 10291  9692  9434  9566  9390  9402  9442  9234  9431
    1688   1 2021-05-01 10618 10167  9569  9303  9430  9260  9275  9305  9105  9341
    1689   1 2021-05-01 10631  9914  9360  9118  9249  9105  9125  9119  8978  9335
    1690   1 2021-05-01  9847  9348  8851  8634  8778  8658  8710  8638  8572  8650
    1691   1 2021-05-01  9884  9549  8982  8717  8841  8701  8745  8840  8603  8649
    1692   1 2021-05-01 10038 10091  9496  9232  9353  9190  9199  9268  9038  8818
    1693   1 2021-05-01 10480 10214  9607  9334  9455  9287  9282  9369  9115  9237
    1694   1 2021-05-01 10777 10400  9793  9526  9646  9461  9456  9485  9288  9487
    1695   1 2021-05-01 10780 10369  9760  9500  9626  9435  9433  9439  9253  9486
    1696   1 2021-05-01 10753 10139  9541  9275  9399  9225  9247  9197  9084  9441
    1697   1 2021-05-01 10651 10288  9665  9384  9489  9306  9303  9338  9136  9316
    1698   1 2021-05-01 10764 10407  9777  9495  9597  9402  9399  9455  9230  9416
    1699   1 2021-05-01 10870 10506  9880  9604  9719  9521  9508  9554  9346  9527
    1700   1 2021-05-01 11010 10624  9991  9722  9843  9652  9642  9627  9469  9682
    1701   1 2021-05-01 11081 10706 10051  9770  9889  9685  9674  9694  9503  9737
    1702   1 2021-05-01 11145 10704 10037  9753  9876  9657  9652  9671  9453  9857
    1703   1 2021-05-01 11104 10675 10025  9745  9866  9653  9655  9631  9465  9800
    1704   1 2021-05-01 11032 10645  9998  9708  9812  9595  9578  9630  9390  9744
    1705   1 2021-05-01 10863 10555  9919  9630  9729  9525  9526  9620  9350  9584
    1706   1 2021-05-01 10593 10294  9656  9357  9442  9242  9253  9376  9093  9395
    1707   1 2021-05-01 10557 10501  9850  9541  9627  9406  9379  9543  9182  9350
    1708   1 2021-05-01 10734 10404  9743  9438  9534  9319  9299  9427  9118  9512
    1709   1 2021-05-01 10778 10444  9805  9501  9595  9383  9367  9477  9187  9571
    1710   1 2021-05-01 10743 10361  9724  9436  9548  9353  9353  9397  9189  9535
    1711   1 2021-05-01 10700 10332  9706  9429  9545  9344  9336  9387  9156  9514
    1712   1 2021-05-01 10700 10273  9656  9386  9513  9325  9327  9312  9166  9508
    1713   1 2021-05-01 10758 10381  9739  9469  9585  9395  9391  9410  9219  9580
    1714   1 2021-05-01 10826 10489  9853  9576  9697  9508  9494  9544  9316  9618
    1715   1 2021-05-01 11002 10618  9986  9707  9833  9631  9621  9665  9448  9812
    1716   1 2021-05-01 11024 10599  9978  9707  9832  9638  9635  9671  9474  9842
    1717   1 2021-05-01 10919 10175  9607  9376  9512  9357  9376  9297  9221  9746
    1718   1 2021-05-01 10490  9895  9348  9121  9262  9114  9145  9063  8994  9351
    1719   1 2021-05-01  9769  9581  9057  8839  8976  8849  8892  8815  8749  8690
    1720   1 2021-05-01  9500  9403  8831  8575  8695  8559  8591  8620  8446  8433
    1721   1 2021-05-01  9999  9653  9063  8798  8903  8751  8763  8822  8601  8831
    1722   1 2021-05-01 10244  9717  9137  8884  9008  8860  8886  8831  8734  9003
    1723   1 2021-05-01 10151  9800  9223  8978  9097  8948  8964  8922  8812  8913
    1724   1 2021-05-01 10090  9539  8979  8742  8867  8737  8769  8712  8627  8864
    1725   1 2021-05-01  9819  9314  8782  8565  8694  8581  8614  8517  8480  8551
    1726   1 2021-05-01  9293  8706  8199  7971  8087  8001  8068  7996  7939  8044
    1727   1 2021-05-01  8996  9183  8601  8322  8421  8283  8317  8371  8153  7778
    1728   1 2021-05-01  9989  9592  9010  8750  8869  8720  8742  8736  8576  8658
    1729   1 2021-05-01 10033  9665  9091  8835  8955  8809  8842  8796  8677  8692
    1730   1 2021-05-01  9882  9445  8906  8666  8779  8667  8700  8642  8556  8495
    1731   1 2021-05-01  9868  9523  8962  8704  8814  8678  8716  8696  8567  8464
    1732   1 2021-05-01 10218  9523  8962  8704  8814  8678  8716  8696  8567  8778
    1733   1 2021-05-01 10390  9865  9263  8987  9092  8939  8961  8995  8792  8911
    1734   1 2021-05-01 10536 10084  9495  9238  9352  9197  9225  9227  9058  9086
    1735   1 2021-05-01 10520 10157  9565  9306  9417  9273  9303  9302  9135  9085
    1736   1 2021-05-01 10377 10086  9499  9231  9341  9204  9235  9261  9071  8903
    1737   1 2021-05-01 10230  9882  9305  9046  9159  9032  9072  9067  8930  8717
    1738   1 2021-05-01 10137  9752  9171  8923  9037  8902  8942  8918  8802  8635
    1739   1 2021-05-01 10064  9780  9187  8935  9039  8911  8946  8982  8798  8558
    1740   1 2021-05-01 10619  9878  9247  8966  8944  8780  8890  8694  8730  7324
    1741   1 2021-05-01 10309  9878  9247  8966  8944  8780  8890  8694  8730  6887
    1742   1 2021-05-01 10752 10199  9510  9203  9189  9000  9074  8919  8898  7555
    1743   1 2021-05-01 10953 10424  9710  9393  9396  9187  9251  9101  9070  7987
    1744   1 2021-05-01 11003 10558  9846  9533  9540  9318  9369  9253  9181  8101
    1745   1 2021-05-01 10995 10530  9839  9535  9540  9335  9398  9264  9210  8064
    1746   1 2021-05-01 10909 10396  9724  9442  9461  9263  9328  9160  9161  7942
    1747   1 2021-05-01 10817 10331  9659  9369  9389  9192  9262  9088  9086  7923
    1748   1 2021-05-01 10820 10413  9741  9460  9466  9274  9343  9201  9165  7896
    1749   1 2021-05-01 10910 10550  9880  9601  9607  9407  9473  9324  9313  7941
    1750   1 2021-05-01 11078 10901 10178  9841  9834  9596  9640  9573  9441  8116
    1751   1 2021-05-01 11838 11522 10793 10460 10474 10196 10210 10165  9999  8935
    1752   1 2021-05-01 12189 11743 11017 10689 10725 10452 10458 10363 10247  9351
    1753   1 2021-05-01 12205 11680 10954 10612 10657 10382 10388 10258 10160  9443
    1754   1 2021-05-01 12306 11680 10954 10612 10657 10382 10388 10258 10160  9571
    1755   1 2021-05-01 12289 11525 10777 10414 10465 10182 10179 10066  9953  9673
    1756   1 2021-05-01 12289 11679 10928 10563 10622 10327 10323 10212 10103  9611
    1757   1 2021-05-01 12366 11857 11102 10733 10785 10491 10482 10409 10247  9666
    1758   1 2021-05-01 12536 12057 11281 10911 10965 10667 10647 10598 10387  9912
    1759   1 2021-05-01 12689 12186 11404 11017 11077 10762 10738 10734 10489 10196
    1760   1 2021-05-01 12693 12248 11490 11123 11191 10881 10859 10820 10611 10214
    1761   1 2021-05-01 12727 12261 11492 11114 11187 10873 10845 10827 10582 10333
    1762   1 2021-05-01 12760 12290 11524 11159 11245 10938 10910 10859 10665 10424
    1763   1 2021-05-01 12766 12263 11486 11125 11211 10905 10868 10853 10636 10463
    1764   1 2021-05-01 12740 12257 11497 11144 11226 10931 10909 10880 10656 10442
    1765   1 2021-05-01 12696 12223 11455 11097 11181 10870 10860 10835 10610 10454
    1766   1 2021-05-01 12694 12340 11587 11232 11318 11022 10994 10991 10751 10440
    1767   1 2021-05-01 12818 12365 11606 11248 11334 11045 11029 11020 10791 10587
    1768   1 2021-05-01 12765 12350 11591 11239 11331 11036 11014 10992 10770 10564
    1769   1 2021-05-01 12525 12114 11371 11042 11129 10852 10843 10808 10619 10236
    1770   1 2021-05-01 12508 12036 11283 10926 11019 10725 10718 10704 10477 10262
    1771   1 2021-05-01 12742 12090 11350 11003 11103 10813 10789 10743 10560 10582
    1772   1 2021-05-01 12804 12261 11538 11212 11326 11042 11023 10942 10799 10644
    1773   1 2021-05-01 10003  9454  8911  8670  8796  8669  8702  8699  8556  8758
    1774   1 2021-05-01  9867  9860  9251  8974  9077  8917  8921  9053  8752  8702
    1775   1 2021-05-01 10355 10094  9468  9175  9284  9098  9101  9205  8931  9104
    1776   1 2021-05-01 10551 10346  9715  9421  9534  9329  9325  9396  9141  9296
    1777   1 2021-05-01 10705 10348  9718  9438  9548  9344  9338  9404  9151  9453
    1778   1 2021-05-01 10760 10258  9648  9378  9497  9313  9321  9337  9164  9475
    1779   1 2021-05-01 10578 10099  9502  9233  9355  9173  9185  9212  9032  9286
    1780   1 2021-05-01 10478 10290  9655  9369  9474  9286  9286  9351  9118  9195
    1781   1 2021-05-01 10877 10443  9796  9504  9612  9411  9400  9481  9216  9531
    1782   1 2021-05-01 11034 10599  9974  9705  9830  9645  9625  9624  9448  9719
    1783   1 2021-05-01 11013 10646  9990  9700  9816  9615  9602  9647  9418  9730
    1784   1 2021-05-01 11137 10755 10097  9804  9918  9694  9673  9725  9492  9887
    1785   1 2021-05-01 11172 10675 10025  9745  9866  9653  9655  9631  9465  9889
    1786   1 2021-05-01 11078 10573  9947  9691  9823  9619  9630  9552  9471  9839
    1787   1 2021-05-01 10970 10161  9564  9322  9450  9277  9308  9217  9164  9744
    1788   1 2021-05-01 10534 10062  9455  9194  9314  9137  9159  9117  9017  9353
    1789   1 2021-05-01 10570 10272  9627  9339  9449  9252  9255  9283  9094  9346
    1790   1 2021-05-01 10815 10425  9791  9513  9624  9411  9399  9432  9217  9598
    1791   1 2021-05-01 10914 10416  9802  9548  9683  9486  9488  9439  9321  9701
    1792   1 2021-05-01 10902 10435  9831  9590  9727  9540  9552  9484  9376  9695
    1793   1 2021-05-01 10811 10317  9713  9456  9582  9404  9412  9377  9249  9628
    1794   1 2021-05-01 10751 10317  9713  9456  9582  9404  9412  9377  9249  9614
    1795   1 2021-05-01 10701 10364  9731  9451  9561  9372  9372  9420  9205  9553
    1796   1 2021-05-01 10786 10380  9746  9463  9572  9384  9381  9474  9209  9608
    1797   1 2021-05-01 10952 10575  9932  9650  9760  9565  9548  9644  9365  9757
    1798   1 2021-05-01 10746 10599  9978  9707  9832  9638  9635  9671  9474  9619
    1799   1 2021-05-01 10692 10362  9769  9510  9635  9449  9453  9500  9296  9599
    1800   1 2021-05-01 10525  9862  9314  9088  9228  9086  9118  9039  8970  9409
    1801   1 2021-05-01 10174  9239  8748  8539  8682  8580  8640  8535  8520  9090
    1802   1 2021-05-01  9315  9188  8605  8327  8417  8289  8307  8442  8172  8259
    1803   1 2021-05-01  9619  9300  8716  8437  8529  8384  8402  8498  8252  8484
    1804   1 2021-05-01 10139  9854  9242  8968  9071  8908  8914  8987  8743  8942
    1805   1 2021-05-01 10230  9903  9307  9056  9167  9010  9016  9044  8851  9017
    1806   1 2021-05-01 10290  9547  9005  8788  8920  8795  8827  8710  8695  9066
    1807   1 2021-05-01  9788  9547  9005  8788  8920  8795  8827  8710  8695  8536
    1808   1 2021-05-01  9635  8927  8443  8239  8371  8286  8342  8206  8219  8400
    1809   1 2021-05-01  9613  9200  8655  8407  8522  8403  8445  8410  8306  8366
    1810   1 2021-05-01  9995  9686  9087  8823  8931  8775  8791  8826  8631  8698
    1811   1 2021-05-01 10227  9665  9091  8835  8955  8809  8842  8796  8677  8878
    1812   1 2021-05-01 10235  9712  9137  8885  9003  8870  8899  8873  8740  8893
    1813   1 2021-05-01 10178  9742  9165  8906  9023  8896  8919  8906  8772  8817
    1814   1 2021-05-01 10269 10066  9469  9196  9310  9162  9178  9205  9024  8879
    1815   1 2021-05-01 10601 10206  9622  9364  9487  9334  9350  9343  9183  9234
    1816   1 2021-05-01 10544 10125  9554  9311  9428  9281  9303  9292  9146  9139
    1817   1 2021-05-01 10363  9910  9345  9099  9210  9089  9129  9109  8958  8947
    1818   1 2021-05-01 10249  9957  9388  9145  9260  9133  9172  9115  9015  8770
    1819   1 2021-05-01 10027  9619  9033  8788  8785  8642  8770  8472  8633  6734
    1820   1 2021-05-01 10366 10153  9439  9100  9046  8839  8916  8924  8734  7025
    1821   1 2021-05-01 10356 10323  9607  9277  9238  9024  9086  9077  8896  7048
    1822   1 2021-05-01 11005 10523  9821  9518  9527  9319  9376  9231  9197  8015
    1823   1 2021-05-01 10995 10578  9893  9591  9597  9384  9440  9312  9273  8085
    1824   1 2021-05-01 11026 10488  9813  9532  9547  9352  9422  9236  9257  8076
    1825   1 2021-05-01 10945 10521  9836  9541  9550  9353  9420  9254  9259  8031
    1826   1 2021-05-01 11010 10443  9757  9452  9464  9262  9318  9189  9142  8043
    1827   1 2021-05-01 11086 10550  9880  9601  9607  9407  9473  9324  9313  8094
    1828   1 2021-05-01 11204 10756 10070  9768  9771  9556  9624  9488  9453  8191
    1829   1 2021-05-01 11652 11481 10730 10375 10376 10103 10112 10112  9891  8719
    1830   1 2021-05-01 12278 11749 11001 10656 10682 10403 10407 10370 10179  9496
    1831   1 2021-05-01 12392 11843 11102 10747 10789 10498 10497 10421 10264  9686
    1832   1 2021-05-01 12482 11953 11194 10830 10884 10588 10571 10507 10334  9787
    1833   1 2021-05-01 12468 11907 11154 10783 10844 10557 10534 10446 10309  9848
    1834   1 2021-05-01 12623 12069 11316 10940 11001 10697 10673 10600 10434 10021
    1835   1 2021-05-01 12592 12001 11245 10864 10924 10633 10598 10519 10345 10072
    1836   1 2021-05-01 12613 12218 11468 11091 11158 10860 10820 10750 10564 10119
    1837   1 2021-05-01 12732 12288 11532 11160 11230 10925 10906 10859 10667 10242
    1838   1 2021-05-01 12764 12352 11591 11217 11291 10991 10974 10936 10728 10289
    1839   1 2021-05-01 12844 12309 11543 11175 11254 10947 10931 10882 10681 10490
    1840   1 2021-05-01 12860 12290 11524 11159 11245 10938 10910 10859 10665 10578
    1841   1 2021-05-01 12858 12350 11590 11247 11343 11046 11016 10935 10786 10578
    1842   1 2021-05-01 12931 12262 11513 11161 11253 10956 10931 10875 10690 10651
    1843   1 2021-05-01 12833 12310 11565 11217 11306 11012 10990 10928 10743 10618
    1844   1 2021-05-01 12744 12274 11528 11186 11274 10979 10960 10926 10730 10481
    1845   1 2021-05-01 12736 12211 11475 11130 11216 10928 10909 10883 10666 10475
    1846   1 2021-05-01 12514 12045 11318 10999 11084 10811 10806 10743 10572 10265
    1847   1 2021-05-01 12425 11981 11242 10901 10987 10713 10712 10666 10482 10146
    1848   1 2021-05-01 12621 12258 11502 11147 11245 10951 10928 10907 10693 10410
    1849   1 2021-05-01 12776 12297 11551 11200 11300 11004 10984 10983 10744 10585
    1850   1 2021-05-01 12581 12022 11288 10944 11033 10746 10733 10712 10487 10380
    1851   1 2021-05-01 12537 11974 11239 10905 11001 10725 10712 10655 10478 10420
    1852   1 2021-05-01  9942  9547  8970  8715  8831  8689  8715  8772  8563  8779
    1853   1 2021-05-01 10181  9817  9208  8931  9034  8866  8874  8965  8709  8985
    1854   1 2021-05-01 10454 10243  9595  9295  9405  9213  9210  9306  9042  9248
    1855   1 2021-05-01 10589 10217  9576  9277  9374  9173  9166  9302  8980  9373
    1856   1 2021-05-01 10541 10176  9577  9320  9429  9243  9248  9277  9071  9277
    1857   1 2021-05-01 10626 10188  9579  9309  9418  9228  9231  9240  9037  9357
    1858   1 2021-05-01 10554 10181  9568  9288  9400  9212  9210  9271  9031  9292
    1859   1 2021-05-01 10562 10264  9626  9331  9433  9225  9211  9313  9016  9288
    1860   1 2021-05-01 10865 10471  9823  9533  9637  9421  9399  9472  9208  9554
    1861   1 2021-05-01 10842 10654  9974  9676  9783  9570  9559  9644  9382  9591
    1862   1 2021-05-01 10996 10671  9995  9681  9785  9547  9531  9660  9345  9758
    1863   1 2021-05-01 11124 10686 10057  9799  9921  9715  9714  9683  9532  9886
    1864   1 2021-05-01 11132 10528  9913  9670  9806  9614  9625  9552  9459  9891
    1865   1 2021-05-01 10902 10375  9781  9552  9694  9512  9523  9389  9380  9684
    1866   1 2021-05-01 10617 10181  9595  9352  9481  9307  9319  9245  9174  9409
    1867   1 2021-05-01 10711 10261  9622  9335  9450  9255  9260  9258  9095  9460
    1868   1 2021-05-01 10861 10459  9799  9500  9612  9410  9402  9444  9227  9623
    1869   1 2021-05-01 10881 10529  9911  9629  9744  9515  9498  9563  9314  9737
    1870   1 2021-05-01 10914 10498  9890  9628  9743  9527  9505  9559  9318  9735
    1871   1 2021-05-01 10867 10402  9788  9535  9659  9473  9491  9460  9325  9701
    1872   1 2021-05-01 10789 10320  9692  9423  9539  9359  9370  9380  9222  9658
    1873   1 2021-05-01 10501 10186  9569  9297  9408  9226  9235  9301  9065  9385
    1874   1 2021-05-01 10514 10330  9694  9409  9512  9320  9319  9441  9138  9368
    1875   1 2021-05-01 10418 10144  9528  9262  9370  9190  9193  9281  9013  9289
    1876   1 2021-05-01 10516 10209  9601  9328  9443  9258  9260  9368  9093  9405
    1877   1 2021-05-01 10535  9933  9344  9082  9197  9029  9038  9121  8865  9450
    1878   1 2021-05-01 10169  9615  9091  8875  9008  8872  8911  8843  8774  9122
    1879   1 2021-05-01 10009  9615  9091  8875  9008  8872  8911  8843  8774  8920
    1880   1 2021-05-01  9654  8941  8439  8224  8355  8263  8317  8206  8202  8519
    1881   1 2021-05-01  9573  9411  8842  8592  8726  8592  8618  8505  8482  8371
    1882   1 2021-05-01 10293  9653  9057  8777  8882  8718  8727  8792  8564  9042
    1883   1 2021-05-01 10282  9893  9297  9032  9147  8983  8996  9017  8837  9069
    1884   1 2021-05-01 10271  9823  9242  8998  9120  8972  8992  8980  8841  9081
    1885   1 2021-05-01 10185  9681  9125  8892  9010  8875  8895  8875  8751  8992
    1886   1 2021-05-01  9932  9475  8920  8676  8794  8669  8700  8655  8560  8724
    1887   1 2021-05-01 10038  9637  9054  8787  8897  8753  8768  8797  8618  8806
    1888   1 2021-05-01 10100  9718  9136  8884  8997  8860  8880  8910  8745  8852
    1889   1 2021-05-01 10073  9906  9297  9030  9145  8992  9014  9056  8854  8838
    1890   1 2021-05-01 10275  9983  9394  9137  9249  9095  9117  9165  8967  9016
    1891   1 2021-05-01 10420 10151  9553  9285  9400  9251  9269  9309  9108  9070
    1892   1 2021-05-01 10588 10172  9577  9312  9423  9273  9293  9350  9136  9218
    1893   1 2021-05-01 10647 10210  9608  9356  9472  9326  9363  9323  9208  9157
    1894   1 2021-05-01 10683 10362  9736  9480  9593  9436  9463  9463  9310  9191
    1895   1 2021-05-01 10663 10257  9656  9415  9530  9381  9416  9374  9264  9200
    1896   1 2021-05-01 10373 10123  9506  9247  9269  9088  9175  8890  9020  7280
    1897   1 2021-05-01 10389  9538  8924  8661  8642  8492  8612  8400  8471  7282
    1898   1 2021-05-01 10195  9818  9173  8885  8854  8681  8772  8640  8624  6938
    1899   1 2021-05-01 10771 10448  9751  9435  9411  9206  9258  9214  9071  7606
    1900   1 2021-05-01 11068 10523  9837  9527  9510  9310  9370  9280  9190  8035
    1901   1 2021-05-01 11006 10576  9925  9649  9648  9448  9521  9376  9364  7878
    1902   1 2021-05-01 10990 10521  9836  9541  9550  9353  9420  9254  9259  7918
    1903   1 2021-05-01 11046 10739 10058  9759  9760  9552  9615  9499  9445  8051
    1904   1 2021-05-01 11478 10878 10222  9948  9964  9749  9817  9655  9642  8444
    1905   1 2021-05-01 11638 11231 10515 10202 10235  9986 10026  9903  9836  8720
    1906   1 2021-05-01 11955 11668 10908 10561 10588 10311 10314 10279 10086  9154
    1907   1 2021-05-01 12243 11885 11138 10794 10825 10544 10545 10490 10322  9472
    1908   1 2021-05-01 12472 12000 11235 10874 10918 10622 10618 10563 10378  9667
    1909   1 2021-05-01 12583 12130 11369 10998 11033 10728 10715 10685 10480  9836
    1910   1 2021-05-01 12667 12057 11300 10932 10991 10697 10684 10598 10455 10001
    1911   1 2021-05-01 12742 12159 11406 11036 11100 10807 10807 10683 10572 10145
    1912   1 2021-05-01 12686 12129 11368 10996 11065 10759 10735 10645 10491 10199
    1913   1 2021-05-01 12937 12390 11634 11262 11337 11033 11001 10924 10741 10485
    1914   1 2021-05-01 12866 12329 11594 11234 11327 11025 11011 10902 10771 10378
    1915   1 2021-05-01 12869 12352 11591 11217 11291 10991 10974 10936 10728 10394
    1916   1 2021-05-01 12867 12478 11700 11317 11386 11081 11048 11060 10803 10460
    1917   1 2021-05-01 12974 12500 11741 11369 11457 11151 11113 11089 10869 10654
    1918   1 2021-05-01 13027 12553 11806 11454 11538 11241 11205 11166 10956 10726
    1919   1 2021-05-01 13013 12406 11653 11309 11409 11116 11096 11019 10857 10757
    1920   1 2021-05-01 12881 12423 11686 11341 11441 11150 11124 11054 10888 10668
    1921   1 2021-05-01 12765 12231 11508 11181 11280 10993 10990 10899 10757 10524
    1922   1 2021-05-01 12683 12218 11480 11140 11228 10934 10917 10889 10676 10467
    1923   1 2021-05-01 12669 12114 11385 11062 11154 10874 10863 10812 10634 10451
    1924   1 2021-05-01 12573 12114 11385 11062 11154 10874 10863 10812 10634 10353
    1925   1 2021-05-01 12582 12073 11336 11006 11093 10824 10811 10752 10582 10348
    1926   1 2021-05-01 12716 12226 11481 11140 11232 10945 10928 10899 10693 10509
    1927   1 2021-05-01 12575 12143 11406 11060 11156 10876 10859 10818 10628 10376
    1928   1 2021-05-01 12464 12022 11288 10944 11033 10746 10733 10712 10487 10282
    1929   1 2021-05-01 12569 12032 11299 10965 11067 10790 10783 10693 10555 10433
    1930   1 2021-05-01 10264  9968  9316  9008  9104  8927  8926  9054  8758  9102
    1931   1 2021-05-01 10385  9972  9298  8972  9055  8857  8844  9042  8665  9220
    1932   1 2021-05-01 10445 10280  9627  9316  9403  9188  9170  9298  8963  9297
    1933   1 2021-05-01 10639 10290  9618  9301  9391  9176  9156  9268  8952  9477
    1934   1 2021-05-01 10646 10251  9607  9303  9413  9200  9181  9239  8984  9442
    1935   1 2021-05-01 10656 10264  9626  9331  9433  9225  9211  9313  9016  9452
    1936   1 2021-05-01 10777 10465  9796  9483  9580  9361  9342  9457  9146  9612
    1937   1 2021-05-01 10854 10448  9802  9526  9634  9424  9403  9442  9215  9654
    1938   1 2021-05-01 10878 10483  9807  9515  9625  9420  9407  9464  9224  9635
    1939   1 2021-05-01 10887 10654 10020  9737  9843  9617  9593  9671  9404  9655
    1940   1 2021-05-01 10988 10537  9921  9648  9772  9565  9556  9562  9378  9773
    1941   1 2021-05-01 10910 10332  9716  9461  9583  9391  9404  9369  9233  9691
    1942   1 2021-05-01 10618 10209  9599  9331  9446  9258  9273  9271  9109  9396
    1943   1 2021-05-01 10655 10435  9787  9493  9596  9400  9385  9430  9193  9441
    1944   1 2021-05-01 10868 10462  9806  9519  9631  9428  9411  9467  9219  9708
    1945   1 2021-05-01 10914 10523  9876  9568  9666  9454  9450  9552  9277  9762
    1946   1 2021-05-01 10830 10444  9829  9547  9659  9446  9437  9504  9265  9702
    1947   1 2021-05-01 10791 10282  9675  9425  9561  9393  9394  9358  9235  9639
    1948   1 2021-05-01 10689 10131  9520  9250  9360  9196  9208  9222  9060  9566
    1949   1 2021-05-01 10514 10131  9520  9250  9360  9196  9208  9222  9060  9394
    1950   1 2021-05-01 10086  9993  9387  9112  9217  9041  9057  9156  8897  9013
    1951   1 2021-05-01 10162  9905  9299  9013  9119  8941  8952  9057  8782  9062
    1952   1 2021-05-01 10164  9943  9338  9063  9174  9005  9014  9104  8843  9103
    1953   1 2021-05-01 10191  9814  9216  8954  9069  8913  8926  8972  8761  9114
    1954   1 2021-05-01 10149  9676  9101  8854  8979  8825  8850  8884  8705  9073
    1955   1 2021-05-01  9950  9595  9003  8742  8862  8707  8730  8791  8578  8879
    1956   1 2021-05-01 10132  9829  9217  8948  9064  8909  8916  8947  8763  9014
    1957   1 2021-05-01 10413 10126  9487  9210  9326  9155  9158  9176  8994  9239
    1958   1 2021-05-01 10517 10038  9437  9171  9285  9122  9126  9144  8960  9273
    1959   1 2021-05-01 10348  9938  9350  9104  9224  9071  9081  9091  8920  9178
    1960   1 2021-05-01 10232  9757  9182  8925  9045  8904  8925  8924  8784  9022
    1961   1 2021-05-01 10111  9541  8987  8746  8857  8739  8770  8787  8629  8944
    1962   1 2021-05-01  9871  9449  8888  8639  8746  8631  8670  8735  8527  8733
    1963   1 2021-05-01 10008  9689  9113  8858  8968  8821  8841  8860  8689  8792
    1964   1 2021-05-01 10061  9731  9151  8895  9009  8866  8886  8921  8739  8843
    1965   1 2021-05-01 10267  9833  9243  8989  9104  8960  8983  9014  8829  9011
    1966   1 2021-05-01 10494 10126  9527  9257  9368  9222  9241  9270  9088  9119
    1967   1 2021-05-01 10485 10303  9687  9420  9532  9380  9391  9437  9233  9091
    1968   1 2021-05-01 10592 10205  9601  9339  9448  9301  9321  9348  9172  9152
    1969   1 2021-05-01 10640 10309  9689  9414  9526  9364  9382  9435  9212  9198
    1970   1 2021-05-01 10742 10290  9683  9430  9546  9390  9417  9430  9264  9264
    1971   1 2021-05-01 11113 10549  9881  9590  9610  9403  9470  9297  9308  8235
    1972   1 2021-05-01 10863 10143  9509  9225  9240  9052  9125  8888  8969  7960
    1973   1 2021-05-01 10473 10064  9442  9161  9166  8976  9052  8839  8908  7382
    1974   1 2021-05-01 10455 10220  9539  9225  9191  8998  9072  8981  8899  7207
    1975   1 2021-05-01 10773 10544  9893  9599  9592  9388  9467  9350  9286  7540
    1976   1 2021-05-01 10893 10535  9890  9611  9596  9403  9486  9364  9326  7620
    1977   1 2021-05-01 11051 10758 10095  9811  9811  9608  9674  9548  9488  7894
    1978   1 2021-05-01 11216 10911 10227  9929  9924  9706  9770  9674  9571  8098
    1979   1 2021-05-01 11405 11029 10341 10048 10047  9820  9882  9784  9695  8339
    1980   1 2021-05-01 12019 11339 10624 10309 10324 10070 10103 10045  9901  9167
    1981   1 2021-05-01 12273 11681 10941 10612 10648 10376 10391 10308 10181  9500
    1982   1 2021-05-01 12349 11849 11112 10782 10821 10544 10554 10472 10336  9604
    1983   1 2021-05-01 12408 11967 11215 10861 10895 10609 10605 10555 10380  9644
    1984   1 2021-05-01 12551 12050 11294 10921 10962 10667 10658 10593 10433  9821
    1985   1 2021-05-01 12726 12262 11498 11118 11171 10863 10848 10805 10601 10140
    1986   1 2021-05-01 12874 12390 11630 11251 11320 11010 10984 10917 10737 10317
    1987   1 2021-05-01 12940 12494 11722 11339 11412 11101 11074 11028 10817 10464
    1988   1 2021-05-01 13033 12421 11685 11325 11412 11103 11085 10990 10834 10554
    1989   1 2021-05-01 12906 12321 11594 11239 11324 11027 11002 10899 10759 10456
    1990   1 2021-05-01 12892 12455 11675 11294 11368 11075 11065 11043 10826 10436
    1991   1 2021-05-01 12897 12519 11726 11340 11408 11099 11083 11105 10839 10495
    1992   1 2021-05-01 12943 12570 11804 11436 11514 11202 11164 11166 10916 10613
    1993   1 2021-05-01 13024 12570 11804 11436 11514 11202 11164 11166 10916 10735
    1994   1 2021-05-01 13029 12543 11797 11457 11548 11250 11229 11165 10982 10759
    1995   1 2021-05-01 12952 12485 11737 11387 11481 11176 11157 11114 10918 10716
    1996   1 2021-05-01 12880 12377 11655 11326 11420 11126 11108 11047 10882 10630
    1997   1 2021-05-01 12729 12284 11554 11221 11314 11029 11017 10951 10787 10544
    1998   1 2021-05-01 12655 12181 11450 11122 11221 10932 10918 10875 10681 10443
    1999   1 2021-05-01 12488 12007 11284 10956 11053 10788 10786 10693 10563 10265
    2000   1 2021-05-01 12476 12161 11404 11042 11126 10832 10826 10833 10586 10228
    2001   1 2021-05-01 12704 12130 11402 11066 11169 10897 10891 10811 10656 10487
    2002   1 2021-05-01 12660 12211 11469 11122 11226 10949 10940 10887 10707 10509
    2003   1 2021-05-01 12702 12206 11469 11135 11230 10945 10933 10881 10705 10575
    2004   1 2021-05-01 12730 12271 11539 11198 11296 11006 10994 10972 10768 10629
    2005   1 2021-05-01  9908  9626  9037  8783  8904  8744  8765  8741  8608  8835
    2006   1 2021-05-01 10036  9506  8907  8643  8757  8605  8628  8617  8479  8960
    2007   1 2021-05-01 10381 10108  9428  9104  9185  8976  8956  9120  8756  9314
    2008   1 2021-05-01 10558 10176  9504  9182  9269  9060  9038  9159  8843  9452
    2009   1 2021-05-01 10674 10307  9628  9309  9419  9203  9190  9253  9001  9518
    2010   1 2021-05-01 10798 10376  9701  9401  9504  9285  9266  9351  9076  9682
    2011   1 2021-05-01 10833 10460  9767  9456  9564  9347  9333  9418  9162  9697
    2012   1 2021-05-01 10873 10484  9832  9538  9642  9424  9397  9466  9205  9720
    2013   1 2021-05-01 10865 10422  9766  9466  9572  9364  9357  9423  9161  9692
    2014   1 2021-05-01 10747 10508  9858  9547  9647  9441  9420  9513  9235  9538
    2015   1 2021-05-01 10905 10521  9865  9554  9664  9454  9439  9490  9254  9695
    2016   1 2021-05-01 10916 10346  9734  9467  9592  9405  9410  9344  9238  9678
    2017   1 2021-05-01 10897 10082  9489  9221  9353  9184  9200  9135  9036  9656
    2018   1 2021-05-01 10477 10199  9561  9264  9371  9188  9193  9247  9022  9315
    2019   1 2021-05-01 10710 10244  9593  9283  9384  9176  9170  9257  8984  9567
    2020   1 2021-05-01 10856 10454  9791  9509  9625  9432  9431  9477  9248  9725
    2021   1 2021-05-01 10790 10444  9829  9547  9659  9446  9437  9504  9265  9675
    2022   1 2021-05-01 10569 10274  9681  9411  9527  9332  9336  9375  9163  9454
    2023   1 2021-05-01 10483 10137  9539  9286  9408  9249  9261  9244  9094  9392
    2024   1 2021-05-01 10277  9854  9264  9007  9123  8968  8988  8996  8830  9211
    2025   1 2021-05-01 10061  9620  9038  8766  8884  8741  8754  8796  8602  9014
    2026   1 2021-05-01  9929  9573  8998  8731  8851  8694  8715  8753  8550  8869
    2027   1 2021-05-01 10124  9751  9145  8882  8992  8830  8849  8913  8685  9046
    2028   1 2021-05-01  9971  9539  8951  8692  8809  8660  8681  8731  8527  8924
    2029   1 2021-05-01  9981  9569  8967  8709  8830  8682  8706  8729  8563  8926
    2030   1 2021-05-01  9944  9745  9123  8855  8962  8802  8817  8863  8665  8878
    2031   1 2021-05-01 10155  9774  9153  8880  8993  8824  8826  8923  8667  9082
    2032   1 2021-05-01 10340  9953  9318  9042  9151  8983  8977  9095  8820  9254
    2033   1 2021-05-01 10477 10120  9485  9207  9322  9147  9147  9210  8990  9355
    2034   1 2021-05-01 10379  9938  9350  9104  9224  9071  9081  9091  8920  9259
    2035   1 2021-05-01  9934  9733  9158  8918  9039  8900  8924  8918  8778  8835
    2036   1 2021-05-01  9838  9519  8972  8727  8843  8722  8760  8776  8629  8709
    2037   1 2021-05-01  9700  9170  8661  8446  8568  8474  8533  8521  8403  8568
    2038   1 2021-05-01 10029  9882  9305  9055  9163  9025  9052  9107  8910  8771
    2039   1 2021-05-01 10255 10003  9400  9128  9235  9082  9098  9157  8950  8931
    2040   1 2021-05-01 10602 10202  9590  9324  9443  9286  9306  9294  9149  9185
    2041   1 2021-05-01 10673 10202  9590  9324  9443  9286  9306  9294  9149  9240
    2042   1 2021-05-01 10647 10198  9590  9321  9426  9276  9299  9312  9135  9195
    2043   1 2021-05-01 10566 10205  9601  9339  9448  9301  9321  9348  9172  9122
    2044   1 2021-05-01 10507 10161  9545  9275  9386  9227  9267  9279  9104  9094
    2045   1 2021-05-01 11079 10563  9883  9588  9597  9383  9443  9325  9287  8230
    2046   1 2021-05-01 11038 10375  9699  9402  9417  9211  9277  9101  9131  8198
    2047   1 2021-05-01 10636 10121  9471  9185  9167  8980  9068  8895  8907  7475
    2048   1 2021-05-01 10868 10491  9818  9513  9492  9292  9363  9247  9179  7546
    2049   1 2021-05-01 10982 10581  9928  9640  9624  9428  9502  9376  9329  7668
    2050   1 2021-05-01 11179 10737 10069  9784  9775  9571  9629  9523  9457  7994
    2051   1 2021-05-01 11160 10732 10064  9782  9779  9573  9638  9494  9459  8068
    2052   1 2021-05-01 11386 11181 10445 10107 10105  9854  9888  9865  9682  8402
    2053   1 2021-05-01 11926 11658 10923 10586 10612 10339 10347 10325 10131  9062
    2054   1 2021-05-01 12444 11901 11172 10857 10898 10629 10639 10541 10419  9688
    2055   1 2021-05-01 12550 12084 11329 10998 11045 10751 10753 10686 10519  9871
    2056   1 2021-05-01 12560 12141 11381 11019 11079 10770 10752 10689 10514  9941
    2057   1 2021-05-01 12655 12280 11523 11152 11220 10918 10897 10813 10649 10065
    2058   1 2021-05-01 12769 12440 11679 11295 11355 11046 11012 10979 10773 10225
    2059   1 2021-05-01 13038 12390 11630 11251 11320 11010 10984 10917 10737 10498
    2060   1 2021-05-01 13057 12506 11742 11366 11443 11128 11104 11031 10861 10573
    2061   1 2021-05-01 13083 12471 11714 11335 11416 11111 11086 11016 10842 10653
    2062   1 2021-05-01 13035 12459 11719 11353 11447 11147 11119 11018 10886 10618
    2063   1 2021-05-01 12910 12358 11606 11260 11348 11044 11037 10944 10781 10490
    2064   1 2021-05-01 12896 12395 11632 11291 11380 11088 11083 10982 10850 10494
    2065   1 2021-05-01 12914 12577 11803 11435 11507 11191 11156 11184 10916 10576
    2066   1 2021-05-01 13012 12585 11839 11503 11576 11275 11244 11215 10986 10696
    2067   1 2021-05-01 12809 12336 11602 11268 11362 11062 11048 11016 10800 10576
    2068   1 2021-05-01 12393 12010 11267 10914 11003 10725 10731 10690 10506 10093
    2069   1 2021-05-01 12433 12010 11267 10914 11003 10725 10731 10690 10506 10162
    2070   1 2021-05-01 12663 12254 11526 11186 11282 10995 10989 10967 10746 10457
    2071   1 2021-05-01 12630 12231 11481 11138 11235 10953 10935 10932 10694 10482
    2072   1 2021-05-01 12615 12221 11475 11129 11217 10935 10917 10912 10683 10503
    2073   1 2021-05-01 12619 12174 11427 11077 11168 10877 10858 10874 10622 10502
    2074   1 2021-05-01 12670 12208 11470 11132 11235 10954 10938 10933 10713 10548
    2075   1 2021-05-01 10352  9855  9247  9000  9134  8960  8975  8830  8815  9200
    2076   1 2021-05-01 10402  9992  9326  9013  9105  8917  8915  8989  8739  9344
    2077   1 2021-05-01 10526 10226  9557  9249  9351  9139  9124  9199  8930  9476
    2078   1 2021-05-01 10552 10250  9589  9285  9391  9185  9168  9236  8985  9468
    2079   1 2021-05-01 10741 10399  9741  9455  9573  9369  9369  9360  9191  9620
    2080   1 2021-05-01 10951 10399  9741  9455  9573  9369  9369  9360  9191  9805
    2081   1 2021-05-01 10887 10534  9834  9500  9595  9378  9362  9486  9199  9755
    2082   1 2021-05-01 10847 10523  9843  9516  9611  9376  9356  9486  9156  9718
    2083   1 2021-05-01 10823 10401  9747  9449  9553  9353  9328  9381  9131  9643
    2084   1 2021-05-01 10962 10569  9893  9582  9695  9474  9452  9520  9245  9770
    2085   1 2021-05-01 10947 10496  9864  9577  9687  9482  9456  9465  9256  9785
    2086   1 2021-05-01 10838 10351  9724  9446  9566  9372  9363  9330  9180  9639
    2087   1 2021-05-01 10505 10152  9521  9235  9352  9150  9156  9151  8972  9366
    2088   1 2021-05-01 10693 10287  9657  9365  9468  9258  9243  9312  9060  9566
    2089   1 2021-05-01 10688 10231  9597  9310  9418  9222  9220  9266  9049  9585
    2090   1 2021-05-01 10511 10142  9523  9238  9347  9153  9152  9233  8968  9443
    2091   1 2021-05-01 10414  9769  9185  8926  9044  8871  8889  8912  8721  9313
    2092   1 2021-05-01 10125  9745  9134  8854  8956  8795  8808  8878  8646  9065
    2093   1 2021-05-01 10082  9719  9098  8807  8908  8740  8748  8829  8576  9003
    2094   1 2021-05-01 10132  9662  9080  8812  8934  8778  8795  8802  8628  9029
    2095   1 2021-05-01 10018  9664  9069  8798  8920  8759  8766  8789  8602  8944
    2096   1 2021-05-01  9970  9546  8962  8696  8818  8656  8675  8729  8513  8886
    2097   1 2021-05-01  9823  9409  8838  8576  8706  8551  8577  8605  8421  8769
    2098   1 2021-05-01  9943  9537  8947  8676  8805  8648  8663  8695  8519  8866
    2099   1 2021-05-01 10048  9705  9100  8826  8936  8766  8782  8839  8619  8966
    2100   1 2021-05-01 10179  9744  9144  8885  9007  8853  8877  8899  8724  9106
    2101   1 2021-05-01 10198  9934  9307  9030  9138  8969  8963  9069  8807  9112
    2102   1 2021-05-01 10247  9810  9212  8954  9067  8902  8908  8985  8761  9160
    2103   1 2021-05-01  9990  9518  8955  8716  8835  8702  8739  8770  8602  8959
    2104   1 2021-05-01  9990  9605  9022  8756  8864  8723  8746  8812  8593  8746
    2105   1 2021-05-01 10312  9957  9371  9114  9232  9084  9100  9139  8940  9073
    2106   1 2021-05-01 10297  9911  9343  9098  9214  9067  9088  9104  8941  9034
    2107   1 2021-05-01  9992  9652  9100  8863  8973  8856  8891  8916  8740  8752
    2108   1 2021-05-01 10142  9769  9193  8938  9045  8912  8938  8984  8790  8801
    2109   1 2021-05-01 10499 10110  9496  9215  9319  9163  9174  9240  9011  9095
    2110   1 2021-05-01 10513 10255  9636  9368  9482  9322  9349  9341  9191  9087
    2111   1 2021-05-01 10617 10144  9554  9301  9416  9264  9300  9263  9139  9164
    2112   1 2021-05-01 10453 10149  9543  9281  9388  9242  9281  9258  9114  9012
    2113   1 2021-05-01 10596 10315  9689  9429  9540  9392  9419  9391  9247  9130
    2114   1 2021-05-01 11167 10703 10053  9799  9821  9627  9702  9515  9547  8306
    2115   1 2021-05-01 11014 10563  9883  9588  9597  9383  9443  9325  9287  8176
    2116   1 2021-05-01 11046 10521  9844  9547  9561  9347  9410  9280  9261  8195
    2117   1 2021-05-01 10980 10258  9603  9311  9316  9122  9190  9011  9033  8101
    2118   1 2021-05-01 10820 10573  9887  9567  9544  9334  9406  9324  9219  7525
    2119   1 2021-05-01 11156 10834 10157  9837  9828  9606  9658  9545  9475  7914
    2120   1 2021-05-01 11299 10788 10134  9839  9832  9627  9690  9527  9509  8119
    2121   1 2021-05-01 11339 10943 10249  9917  9935  9712  9758  9605  9549  8338
    2122   1 2021-05-01 11680 11536 10778 10420 10441 10166 10171 10184  9947  8850
    2123   1 2021-05-01 12129 11536 10778 10420 10441 10166 10171 10184  9947  9356
    2124   1 2021-05-01 12390 11957 11222 10897 10933 10658 10662 10610 10442  9642
    2125   1 2021-05-01 12579 12076 11310 10968 11000 10709 10705 10688 10467  9847
    2126   1 2021-05-01 12752 12208 11430 11061 11113 10806 10786 10762 10541 10197
    2127   1 2021-05-01 12887 12280 11523 11152 11220 10918 10897 10813 10649 10366
    2128   1 2021-05-01 12940 12475 11703 11321 11395 11080 11052 11000 10807 10467
    2129   1 2021-05-01 13080 12576 11824 11454 11526 11229 11203 11116 10969 10587
    2130   1 2021-05-01 13182 12609 11845 11474 11553 11248 11223 11149 10987 10744
    2131   1 2021-05-01 13167 12611 11860 11496 11581 11275 11240 11155 10989 10802
    2132   1 2021-05-01 13127 12663 11916 11560 11639 11334 11302 11222 11058 10793
    2133   1 2021-05-01 13112 12556 11826 11493 11580 11284 11278 11150 11032 10724
    2134   1 2021-05-01 13134 12539 11802 11489 11591 11294 11282 11125 11044 10824
    2135   1 2021-05-01 12985 12548 11782 11418 11486 11186 11158 11140 10919 10709
    2136   1 2021-05-01 12939 12520 11785 11449 11535 11240 11212 11141 10977 10638
    2137   1 2021-05-01 12936 12436 11697 11363 11449 11153 11143 11058 10902 10632
    2138   1 2021-05-01 12248 11802 11081 10750 10830 10571 10572 10550 10362  9972
    2139   1 2021-05-01 12508 12091 11351 11017 11106 10835 10817 10803 10577 10316
    2140   1 2021-05-01 12498 12066 11322 10977 11071 10789 10773 10762 10535 10329
    2141   1 2021-05-01 12589 12097 11345 11006 11098 10816 10810 10772 10577 10468
    2142   1 2021-05-01 12512 12118 11374 11039 11136 10865 10848 10794 10617 10434
    2143   1 2021-05-01 12580 12050 11312 10996 11091 10816 10803 10768 10573 10499
    2144   1 2021-05-01 10451  9906  9270  9005  9132  8963  8983  8871  8817  9356
    2145   1 2021-05-01 10366 10024  9360  9044  9131  8932  8927  9040  8745  9296
    2146   1 2021-05-01 10394 10053  9391  9072  9156  8944  8925  9054  8736  9311
    2147   1 2021-05-01 10696 10342  9697  9405  9514  9296  9289  9327  9082  9543
    2148   1 2021-05-01 10907 10350  9713  9428  9536  9319  9315  9337  9118  9749
    2149   1 2021-05-01 10806 10491  9775  9446  9554  9341  9331  9422  9164  9658
    2150   1 2021-05-01 10882 10540  9829  9486  9576  9353  9331  9458  9132  9761
    2151   1 2021-05-01 10939 10571  9878  9558  9661  9428  9411  9515  9212  9782
    2152   1 2021-05-01 10928 10558  9853  9528  9637  9415  9409  9498  9218  9799
    2153   1 2021-05-01 10972 10530  9853  9545  9656  9437  9423  9457  9228  9839
    2154   1 2021-05-01 10917 10480  9788  9469  9570  9361  9353  9426  9163  9794
    2155   1 2021-05-01 10811 10474  9796  9477  9571  9350  9324  9433  9125  9704
    2156   1 2021-05-01 10845 10349  9696  9398  9512  9300  9286  9311  9102  9754
    2157   1 2021-05-01 10745 10308  9655  9365  9475  9271  9249  9297  9065  9653
    2158   1 2021-05-01 10472 10081  9452  9155  9257  9064  9066  9123  8896  9424
    2159   1 2021-05-01 10321  9882  9277  8986  9086  8888  8883  8952  8704  9313
    2160   1 2021-05-01 10061  9567  8977  8714  8822  8659  8677  8701  8506  9062
    2161   1 2021-05-01  9912  9554  8945  8670  8771  8609  8624  8673  8453  8882
    2162   1 2021-05-01 10023  9663  9047  8765  8877  8715  8725  8774  8552  8948
    2163   1 2021-05-01 10027  9635  9028  8748  8858  8699  8700  8745  8536  8962
    2164   1 2021-05-01  9945  9422  8843  8580  8709  8562  8587  8598  8445  8879
    2165   1 2021-05-01  9801  9455  8873  8616  8735  8587  8605  8625  8454  8753
    2166   1 2021-05-01  9903  9542  8953  8681  8801  8648  8664  8723  8514  8884
    2167   1 2021-05-01  9982  9733  9135  8872  8990  8825  8846  8888  8684  8966
    2168   1 2021-05-01 10096  9766  9153  8887  9006  8847  8862  8893  8701  9069
    2169   1 2021-05-01  9551  9404  8819  8548  8655  8505  8530  8583  8376  8386
    2170   1 2021-05-01  9791  9419  8860  8620  8740  8613  8653  8665  8507  8593
    2171   1 2021-05-01  9954  9593  9001  8751  8868  8724  8759  8821  8610  8753
    2172   1 2021-05-01 10183  9938  9341  9083  9198  9058  9077  9086  8926  8940
    2173   1 2021-05-01 10366  9959  9358  9098  9217  9066  9088  9103  8936  9123
    2174   1 2021-05-01 10233  9818  9259  9013  9127  8989  9010  9022  8858  8944
    2175   1 2021-05-01 10171  9880  9287  9018  9119  8983  9005  9041  8851  8836
    2176   1 2021-05-01 10338 10053  9448  9183  9291  9134  9161  9169  8990  8953
    2177   1 2021-05-01 10415 10252  9640  9381  9497  9339  9357  9343  9187  8994
    2178   1 2021-05-01 10793 10252  9640  9381  9497  9339  9357  9343  9187  9328
    2179   1 2021-05-01 10931 10165  9570  9319  9433  9281  9319  9253  9158  9461
    2180   1 2021-05-01 10946 10472  9798  9486  9503  9290  9346  9238  9186  8134
    2181   1 2021-05-01 10954 10437  9776  9482  9496  9295  9370  9227  9209  8085
    2182   1 2021-05-01 10895 10298  9656  9357  9362  9173  9249  9078  9094  7909
    2183   1 2021-05-01 10776 10298  9656  9357  9362  9173  9249  9078  9094  7624
    2184   1 2021-05-01 11081 10430  9761  9445  9433  9229  9323  9185  9150  7825
    2185   1 2021-05-01 11347 10822 10116  9776  9762  9531  9588  9502  9400  8179
    2186   1 2021-05-01 11443 10911 10228  9905  9907  9670  9720  9587  9524  8409
    2187   1 2021-05-01 11507 11110 10391 10037 10053  9805  9832  9740  9612  8533
    2188   1 2021-05-01 11769 11415 10680 10325 10354 10097 10117 10050  9908  8888
    2189   1 2021-05-01 12346 11821 11094 10752 10778 10500 10524 10498 10315  9508
    2190   1 2021-05-01 12401 11939 11186 10834 10876 10599 10600 10536 10393  9678
    2191   1 2021-05-01 12550 12197 11418 11050 11104 10794 10774 10745 10529  9872
    2192   1 2021-05-01 12753 12468 11685 11306 11372 11053 11031 11008 10779 10170
    2193   1 2021-05-01 13147 12617 11847 11485 11556 11250 11227 11153 10979 10642
    2194   1 2021-05-01 13191 12741 11971 11608 11680 11360 11349 11291 11105 10755
    2195   1 2021-05-01 13235 12691 11935 11589 11666 11360 11352 11245 11098 10802
    2196   1 2021-05-01 13238 12691 11935 11589 11666 11360 11352 11245 11098 10790
    2197   1 2021-05-01 13304 12705 11938 11570 11633 11326 11288 11253 11049 10943
    2198   1 2021-05-01 13257 12800 12047 11692 11763 11449 11426 11367 11182 10921
    2199   1 2021-05-01 13301 12733 11991 11652 11732 11422 11402 11313 11145 10939
    2200   1 2021-05-01 13247 12755 12019 11690 11781 11477 11453 11349 11207 10916
    2201   1 2021-05-01 13188 12659 11886 11556 11644 11364 11352 11249 11118 10870
    2202   1 2021-05-01 13044 12662 11911 11581 11659 11363 11337 11295 11093 10725
    2203   1 2021-05-01 12990 12494 11752 11421 11504 11213 11196 11129 10952 10672
    2204   1 2021-05-01 12948 12480 11750 11419 11513 11225 11211 11159 10984 10645
    2205   1 2021-05-01 12535 11959 11224 10876 10966 10694 10687 10658 10465 10365
    2206   1 2021-05-01 12693 12175 11426 11097 11203 10926 10914 10853 10691 10610
    2207   1 2021-05-01 12736 12246 11494 11170 11275 10988 10970 10940 10752 10677
    2208   1 2021-05-01 12649 12170 11425 11100 11197 10922 10898 10874 10658 10576
    2209   1 2021-05-01 10561  9996  9322  9016  9119  8925  8932  8962  8774  9439
    2210   1 2021-05-01 10573 10212  9511  9179  9263  9050  9040  9118  8857  9463
    2211   1 2021-05-01 10650 10266  9602  9294  9392  9171  9163  9229  8959  9497
    2212   1 2021-05-01 10678 10277  9605  9289  9390  9173  9162  9223  8987  9532
    2213   1 2021-05-01 10715 10321  9649  9353  9463  9246  9230  9270  9047  9576
    2214   1 2021-05-01 10851 10385  9699  9375  9474  9254  9237  9312  9036  9696
    2215   1 2021-05-01 10911 10486  9807  9495  9606  9402  9386  9415  9176  9730
    2216   1 2021-05-01 10957 10465  9802  9515  9630  9430  9423  9424  9219  9796
    2217   1 2021-05-01 10991 10497  9825  9527  9651  9450  9441  9433  9260  9863
    2218   1 2021-05-01 10957 10480  9788  9469  9570  9361  9353  9426  9163  9854
    2219   1 2021-05-01 10737 10385  9692  9370  9468  9263  9246  9339  9058  9725
    2220   1 2021-05-01 10681 10313  9640  9317  9412  9193  9172  9291  8983  9651
    2221   1 2021-05-01 10580 10103  9450  9145  9243  9048  9043  9091  8856  9554
    2222   1 2021-05-01 10506  9978  9316  8995  9092  8898  8893  8981  8723  9482
    2223   1 2021-05-01 10237  9784  9133  8821  8917  8731  8726  8802  8557  9296
    2224   1 2021-05-01 10088  9572  8961  8684  8784  8605  8611  8670  8433  9100
    2225   1 2021-05-01  9937  9497  8890  8601  8705  8537  8540  8605  8369  8929
    2226   1 2021-05-01  9922  9548  8938  8657  8762  8601  8599  8678  8429  8922
    2227   1 2021-05-01  9883  9548  8938  8657  8762  8601  8599  8678  8429  8897
    2228   1 2021-05-01  9781  9456  8868  8598  8715  8559  8574  8624  8413  8805
    2229   1 2021-05-01  9974  9408  8865  8632  8751  8617  8645  8603  8488  8866
    2230   1 2021-05-01  9443  9189  8671  8450  8574  8451  8491  8424  8345  8397
    2231   1 2021-05-01  9657  9347  8766  8507  8620  8482  8509  8558  8369  8529
    2232   1 2021-05-01  9787  9299  8747  8508  8630  8501  8537  8553  8407  8621
    2233   1 2021-05-01  9737  9444  8871  8630  8737  8603  8641  8688  8486  8572
    2234   1 2021-05-01 10063  9778  9185  8929  9035  8884  8898  8930  8736  8836
    2235   1 2021-05-01 10248  9836  9245  8994  9113  8967  8987  9012  8825  9012
    2236   1 2021-05-01 10303  9959  9358  9098  9217  9066  9088  9103  8936  9066
    2237   1 2021-05-01 10310  9942  9345  9078  9183  9032  9046  9098  8874  9038
    2238   1 2021-05-01 10311  9959  9361  9099  9209  9057  9081  9100  8907  8996
    2239   1 2021-05-01 10601 10218  9583  9309  9414  9249  9266  9299  9095  9208
    2240   1 2021-05-01 10702 10367  9726  9454  9556  9377  9386  9438  9203  9309
    2241   1 2021-05-01 10884 10569  9936  9678  9782  9601  9602  9638  9406  9465
    2242   1 2021-05-01 10885 10392  9779  9533  9654  9499  9536  9478  9389  9453
    2243   1 2021-05-01 11114 10635  9975  9702  9721  9518  9594  9435  9427  8245
    2244   1 2021-05-01 11034 10466  9788  9476  9501  9295  9358  9195  9196  8166
    2245   1 2021-05-01 10702 10373  9728  9433  9442  9256  9332  9184  9179  7694
    2246   1 2021-05-01 10557 10266  9613  9321  9312  9125  9215  9079  9059  7329
    2247   1 2021-05-01 10735 10303  9644  9340  9321  9127  9209  9087  9043  7428
    2248   1 2021-05-01 10831 10617  9923  9585  9558  9339  9396  9314  9206  7491
    2249   1 2021-05-01 11170 10904 10182  9823  9825  9577  9611  9548  9404  7930
    2250   1 2021-05-01 11707 11233 10504 10153 10187  9927  9975  9842  9767  8793
    2251   1 2021-05-01 11877 11601 10894 10565 10613 10362 10389 10277 10180  8995
    2252   1 2021-05-01 12022 11497 10791 10456 10485 10219 10261 10196 10056  9150
    2253   1 2021-05-01 12068 11765 11022 10664 10682 10413 10435 10414 10230  9126
    2254   1 2021-05-01 12375 11765 11022 10664 10682 10413 10435 10414 10230  9604
    2255   1 2021-05-01 12600 12215 11439 11058 11109 10805 10783 10769 10548  9884
    2256   1 2021-05-01 12893 12440 11667 11282 11341 11023 11007 10990 10754 10303
    2257   1 2021-05-01 13085 12671 11880 11503 11566 11253 11222 11211 10976 10547
    2258   1 2021-05-01 13248 12729 11944 11580 11656 11334 11302 11277 11063 10810
    2259   1 2021-05-01 13279 12734 11975 11616 11691 11380 11360 11289 11122 10831
    2260   1 2021-05-01 13238 12760 11998 11640 11716 11401 11389 11313 11138 10819
    2261   1 2021-05-01 13305 12855 12086 11724 11792 11468 11440 11422 11191 10910
    2262   1 2021-05-01 13367 12917 12159 11807 11883 11567 11540 11513 11297 10947
    2263   1 2021-05-01 13308 12863 12106 11766 11850 11543 11522 11472 11270 10897
    2264   1 2021-05-01 13260 12767 12010 11686 11771 11478 11455 11383 11218 10906
    2265   1 2021-05-01 13201 12664 11910 11568 11649 11336 11319 11292 11081 10867
    2266   1 2021-05-01 13084 12529 11805 11475 11552 11262 11248 11188 11008 10737
    2267   1 2021-05-01 12907 12410 11691 11369 11461 11180 11173 11097 10938 10564
    2268   1 2021-05-01 12819 12431 11695 11369 11452 11171 11156 11113 10918 10518
    2269   1 2021-05-01 12774 12349 11611 11279 11373 11082 11062 11038 10825 10524
    2270   1 2021-05-01 12757 12278 11573 11252 11351 11075 11066 11007 10846 10527
    2271   1 2021-05-01 12600 12219 11478 11148 11249 10964 10951 10938 10723 10481
    2272   1 2021-05-01 12709 12215 11472 11146 11247 10955 10937 10930 10711 10609
    2273   1 2021-05-01 12701 12178 11440 11125 11212 10942 10922 10917 10707 10642
    2274   1 2021-05-01 12515 11998 11270 10953 11045 10792 10783 10734 10562 10456
    2275   1 2021-05-01 10647 10210  9533  9223  9323  9111  9105  9140  8917  9492
    2276   1 2021-05-01 10749 10357  9671  9347  9448  9227  9209  9272  9011  9589
    2277   1 2021-05-01 10815 10482  9758  9411  9501  9274  9260  9381  9044  9660
    2278   1 2021-05-01 10833 10480  9826  9516  9612  9388  9357  9451  9149  9680
    2279   1 2021-05-01 10847 10437  9798  9518  9635  9445  9436  9411  9250  9693
    2280   1 2021-05-01 10936 10321  9681  9406  9529  9344  9354  9290  9180  9806
    2281   1 2021-05-01 10840 10252  9588  9287  9408  9207  9217  9201  9039  9756
    2282   1 2021-05-01 10746 10104  9451  9160  9277  9081  9099  9066  8924  9687
    2283   1 2021-05-01 10549 10043  9396  9098  9202  9007  9008  9023  8828  9532
    2284   1 2021-05-01 10502 10043  9396  9098  9202  9007  9008  9023  8828  9500
    2285   1 2021-05-01 10397  9914  9249  8954  9055  8886  8893  8896  8732  9408
    2286   1 2021-05-01 10242  9733  9087  8793  8898  8723  8728  8732  8568  9283
    2287   1 2021-05-01  9906  9533  8909  8619  8722  8552  8568  8599  8404  8954
    2288   1 2021-05-01  9801  9402  8819  8557  8683  8543  8562  8592  8415  8809
    2289   1 2021-05-01  9848  9498  8895  8625  8737  8593  8609  8652  8458  8832
    2290   1 2021-05-01  9883  9498  8895  8625  8737  8593  8609  8652  8458  8853
    2291   1 2021-05-01  9902  9549  8938  8662  8762  8614  8619  8693  8457  8861
    2292   1 2021-05-01  9897  9458  8862  8593  8701  8551  8564  8608  8412  8840
    2293   1 2021-05-01  9941  9430  8850  8596  8709  8559  8579  8597  8427  8808
    2294   1 2021-05-01  9862  9540  8948  8694  8803  8654  8675  8708  8519  8728
    2295   1 2021-05-01  9732  9398  8834  8589  8699  8565  8597  8630  8447  8580
    2296   1 2021-05-01  9972  9778  9185  8929  9035  8884  8898  8930  8736  8772
    2297   1 2021-05-01 10178  9773  9151  8898  9012  8876  8901  8902  8755  8959
    2298   1 2021-05-01 10221  9917  9293  9012  9103  8945  8948  9048  8777  8995
    2299   1 2021-05-01 10226  9894  9278  9011  9106  8952  8964  9039  8793  8994
    2300   1 2021-05-01 10226 10132  9499  9216  9320  9155  9166  9247  8986  8960
    2301   1 2021-05-01 10650 10337  9682  9397  9499  9327  9340  9430  9173  9308
    2302   1 2021-05-01 10870 10588  9917  9625  9715  9527  9528  9626  9343  9464
    2303   1 2021-05-01 10933 10487  9858  9602  9700  9530  9534  9580  9366  9520
    2304   1 2021-05-01 10741 10458  9816  9550  9556  9370  9457  9333  9296  7678
    2305   1 2021-05-01 10796 10191  9529  9225  9222  9034  9114  9027  8962  7850
    2306   1 2021-05-01 10666 10210  9572  9276  9292  9096  9188  9051  9034  7745
    2307   1 2021-05-01 10582  9920  9307  9022  9019  8846  8936  8783  8780  7623
    2308   1 2021-05-01 10454 10116  9481  9180  9157  8970  9060  8940  8904  7161
    2309   1 2021-05-01 10658 10497  9795  9444  9433  9213  9261  9179  9081  7329
    2310   1 2021-05-01 11116 10438  9740  9399  9392  9177  9231  9117  9054  8078
    2311   1 2021-05-01 11430 10978 10244  9873  9903  9649  9665  9580  9470  8461
    2312   1 2021-05-01 11809 11391 10664 10315 10346 10080 10107 10023  9889  8859
    2313   1 2021-05-01 12171 11601 10894 10565 10613 10362 10389 10277 10180  9335
    2314   1 2021-05-01 11824 11330 10656 10344 10386 10151 10200 10033 10004  8879
    2315   1 2021-05-01 11674 11374 10663 10319 10336 10079 10118 10059  9915  8701
    2316   1 2021-05-01 12102 11816 11057 10689 10738 10454 10459 10409 10225  9266
    2317   1 2021-05-01 12680 12307 11519 11135 11186 10882 10869 10864 10607 10023
    2318   1 2021-05-01 12981 12541 11759 11395 11463 11149 11131 11093 10882 10376
    2319   1 2021-05-01 13088 12644 11865 11493 11569 11253 11226 11185 10985 10592
    2320   1 2021-05-01 13179 12720 11939 11582 11661 11350 11328 11275 11079 10711
    2321   1 2021-05-01 13258 12774 12009 11662 11735 11425 11408 11349 11157 10767
    2322   1 2021-05-01 13272 12798 12030 11679 11747 11427 11409 11398 11162 10806
    2323   1 2021-05-01 13273 12843 12085 11728 11803 11478 11458 11415 11202 10763
    2324   1 2021-05-01 13319 12866 12104 11758 11825 11519 11498 11474 11245 10845
    2325   1 2021-05-01 13297 12856 12095 11753 11834 11531 11514 11470 11266 10864
    2326   1 2021-05-01 13235 12767 12010 11686 11771 11478 11455 11383 11218 10874
    2327   1 2021-05-01 13141 12683 11925 11591 11668 11372 11352 11312 11125 10777
    2328   1 2021-05-01 13036 12553 11825 11482 11561 11264 11246 11215 11013 10700
    2329   1 2021-05-01 12928 12468 11738 11403 11488 11197 11183 11143 10954 10590
    2330   1 2021-05-01 12907 12386 11651 11323 11425 11140 11132 11064 10904 10620
    2331   1 2021-05-01 12862 12361 11636 11322 11420 11136 11141 11078 10918 10619
    2332   1 2021-05-01 12653 12091 11372 11071 11162 10895 10892 10831 10671 10463
    2333   1 2021-05-01 12204 11714 11009 10706 10781 10533 10541 10503 10318  9981
    2334   1 2021-05-01 12616 12094 11349 11014 11100 10824 10805 10800 10568 10533
    2335   1 2021-05-01 12630 12215 11472 11146 11247 10955 10937 10930 10711 10550
    2336   1 2021-05-01 12605 12130 11379 11064 11161 10885 10871 10842 10655 10533
    2337   1 2021-05-01 12577 12078 11351 11041 11140 10876 10866 10826 10660 10521
    2338   1 2021-05-01 12494 11945 11229 10913 11005 10756 10742 10719 10541 10452
    2339   1 2021-05-01 10683 10296  9593  9256  9340  9118  9099  9204  8907  9523
    2340   1 2021-05-01 10669 10482  9758  9411  9501  9274  9260  9381  9044  9538
    2341   1 2021-05-01 10790 10453  9758  9413  9510  9287  9264  9398  9055  9612
    2342   1 2021-05-01 10721 10354  9720  9429  9528  9324  9313  9357  9119  9591
    2343   1 2021-05-01 10665 10246  9613  9321  9427  9234  9233  9233  9050  9525
    2344   1 2021-05-01 10744 10226  9581  9295  9414  9225  9238  9181  9064  9663
    2345   1 2021-05-01 10687 10105  9473  9194  9319  9139  9152  9066  8980  9639
    2346   1 2021-05-01 10293  9746  9115  8838  8949  8775  8789  8741  8625  9253
    2347   1 2021-05-01  9677  9354  8756  8485  8592  8449  8465  8536  8308  8699
    2348   1 2021-05-01  9806  9391  8775  8503  8615  8471  8488  8551  8346  8793
    2349   1 2021-05-01  9922  9500  8893  8623  8734  8584  8599  8630  8443  8893
    2350   1 2021-05-01  9865  9397  8815  8549  8664  8522  8535  8558  8372  8831
    2351   1 2021-05-01  9737  9393  8799  8520  8639  8504  8525  8535  8382  8726
    2352   1 2021-05-01  9915  9466  8844  8546  8645  8490  8510  8566  8343  8872
    2353   1 2021-05-01  9969  9528  8913  8642  8755  8588  8601  8627  8441  8923
    2354   1 2021-05-01  9995  9558  8944  8674  8787  8627  8643  8669  8479  8942
    2355   1 2021-05-01  9909  9494  8887  8617  8719  8569  8579  8656  8428  8853
    2356   1 2021-05-01  9881  9411  8810  8532  8631  8482  8491  8578  8339  8784
    2357   1 2021-05-01  9885  9672  9052  8769  8859  8701  8708  8828  8532  8791
    2358   1 2021-05-01 10114  9691  9101  8856  8962  8818  8839  8822  8672  8923
    2359   1 2021-05-01 10123  9745  9110  8845  8946  8797  8828  8827  8680  8912
    2360   1 2021-05-01 10219  9812  9171  8878  8970  8814  8834  8912  8681  9024
    2361   1 2021-05-01 10245  9901  9270  8982  9071  8893  8899  9009  8728  9007
    2362   1 2021-05-01 10313 10010  9389  9113  9212  9044  9058  9133  8874  9007
    2363   1 2021-05-01 10487 10122  9492  9210  9301  9124  9132  9243  8957  9166
    2364   1 2021-05-01 10685 10475  9797  9510  9604  9427  9437  9548  9268  9341
    2365   1 2021-05-01 10569 10158  9533  9269  9248  9066  9171  9049  9027  7208
    2366   1 2021-05-01 10486 10017  9398  9124  9128  8945  9043  8882  8896  7211
    2367   1 2021-05-01 10560 10179  9554  9293  9287  9104  9202  9078  9047  7384
    2368   1 2021-05-01 10260  9890  9269  8987  8985  8812  8902  8762  8751  7254
    2369   1 2021-05-01 10452 10059  9412  9104  9099  8916  8993  8894  8843  7502
    2370   1 2021-05-01 10361  9933  9314  9031  9034  8870  8951  8792  8799  7397
    2371   1 2021-05-01 10189  9805  9199  8912  8898  8729  8832  8687  8682  7161
    2372   1 2021-05-01 10323 10239  9548  9203  9186  8979  9045  8988  8861  7116
    2373   1 2021-05-01 11767 11209 10451 10073 10108  9841  9843  9792  9639  8996
    2374   1 2021-05-01 12041 11497 10768 10408 10450 10182 10196 10105  9980  9304
    2375   1 2021-05-01 12222 11693 10974 10634 10685 10422 10429 10311 10221  9512
    2376   1 2021-05-01 12247 11556 10872 10560 10607 10359 10388 10218 10197  9446
    2377   1 2021-05-01 11990 11536 10811 10479 10537 10278 10304 10138 10088  9195
    2378   1 2021-05-01 12219 11853 11104 10750 10796 10518 10522 10464 10289  9541
    2379   1 2021-05-01 12389 12089 11321 10957 11010 10720 10711 10691 10475  9675
    2380   1 2021-05-01 12615 12175 11392 11024 11076 10781 10778 10751 10532  9904
    2381   1 2021-05-01 13020 12422 11628 11260 11307 11001 10992 11003 10747 10357
    2382   1 2021-05-01 12976 12668 11889 11533 11598 11285 11267 11236 11023 10333
    2383   1 2021-05-01 13196 12698 11916 11548 11622 11297 11272 11281 11019 10674
    2384   1 2021-05-01 13212 12700 11934 11593 11667 11358 11346 11280 11102 10731
    2385   1 2021-05-01 13126 12594 11834 11494 11569 11270 11265 11194 11029 10626
    2386   1 2021-05-01 12963 12664 11896 11541 11600 11299 11274 11285 11020 10447
    2387   1 2021-05-01 13012 12751 11978 11622 11688 11381 11367 11374 11102 10524
    2388   1 2021-05-01 13147 12702 11945 11604 11666 11358 11347 11356 11094 10671
    2389   1 2021-05-01 13126 12717 11966 11642 11725 11424 11403 11361 11171 10643
    2390   1 2021-05-01 13089 12627 11878 11545 11628 11329 11310 11283 11083 10700
    2391   1 2021-05-01 13010 12508 11774 11448 11532 11243 11235 11172 11010 10679
    2392   1 2021-05-01 12923 12436 11710 11393 11482 11198 11197 11140 10970 10617
    2393   1 2021-05-01 12871 12331 11599 11279 11354 11076 11075 11056 10846 10578
    2394   1 2021-05-01 12659 12331 11599 11279 11354 11076 11075 11056 10846 10392
    2395   1 2021-05-01 12582 12166 11438 11148 11235 10965 10971 10906 10755 10360
    2396   1 2021-05-01 12466 11994 11279 10988 11072 10803 10809 10740 10591 10267
    2397   1 2021-05-01 12342 11816 11105 10814 10892 10630 10636 10579 10424 10147
    2398   1 2021-05-01 12011 11611 10914 10631 10714 10471 10487 10421 10277  9755
    2399   1 2021-05-01 12370 11855 11125 10812 10891 10628 10628 10611 10402 10150
    2400   1 2021-05-01 12484 12062 11304 10982 11063 10789 10781 10754 10555 10324
    2401   1 2021-05-01 12628 12175 11427 11103 11199 10931 10916 10879 10689 10527
    2402   1 2021-05-01 12667 12191 11453 11138 11227 10962 10942 10910 10731 10602
    2403   1 2021-05-01 12600 12125 11389 11082 11172 10905 10898 10870 10684 10524
    2404   1 2021-05-01 12478 11913 11197 10894 10988 10729 10726 10679 10507 10425
    2405   1 2021-05-01 10793 10357  9666  9334  9424  9199  9177  9269  8957  9620
    2406   1 2021-05-01 10746 10302  9637  9332  9445  9237  9242  9239  9051  9566
    2407   1 2021-05-01 10618 10162  9522  9222  9321  9118  9129  9148  8953  9495
    2408   1 2021-05-01 10582 10191  9540  9234  9346  9142  9148  9153  8962  9453
    2409   1 2021-05-01 10737 10176  9544  9267  9385  9206  9204  9143  9028  9622
    2410   1 2021-05-01 10676 10087  9466  9201  9322  9159  9173  9073  9002  9614
    2411   1 2021-05-01  9655  9328  8750  8504  8627  8492  8528  8476  8380  8681
    2412   1 2021-05-01  9582  9211  8624  8352  8458  8327  8346  8385  8191  8586
    2413   1 2021-05-01  9693  9264  8676  8418  8532  8394  8413  8414  8271  8686
    2414   1 2021-05-01  9721  9209  8634  8382  8504  8371  8407  8425  8273  8758
    2415   1 2021-05-01  9624  9235  8662  8401  8521  8385  8418  8435  8284  8690
    2416   1 2021-05-01  9504  9265  8660  8374  8479  8342  8361  8441  8210  8642
    2417   1 2021-05-01  9617  9085  8498  8232  8335  8213  8241  8286  8094  8652
    2418   1 2021-05-01  9561  9163  8548  8256  8352  8217  8237  8322  8084  8602
    2419   1 2021-05-01  9398  9271  8650  8354  8461  8311  8318  8395  8151  8454
    2420   1 2021-05-01  9843  9521  8891  8597  8700  8541  8545  8615  8371  8830
    2421   1 2021-05-01  9958  9500  8872  8579  8685  8540  8543  8598  8379  8936
    2422   1 2021-05-01  9882  9454  8843  8572  8673  8519  8529  8564  8377  8854
    2423   1 2021-05-01  9896  9411  8810  8532  8631  8482  8491  8578  8339  8864
    2424   1 2021-05-01  9942  9500  8885  8614  8711  8561  8574  8644  8405  8882
    2425   1 2021-05-01 10001  9716  9098  8821  8907  8753  8759  8846  8580  8887
    2426   1 2021-05-01 10004  9702  9085  8813  8905  8747  8761  8810  8583  8878
    2427   1 2021-05-01 10185  9831  9181  8887  8978  8810  8833  8927  8689  9008
    2428   1 2021-05-01 10063  9721  9087  8801  8898  8735  8753  8844  8600  8892
    2429   1 2021-05-01 10148  9904  9268  8973  9051  8878  8875  9027  8713  8910
    2430   1 2021-05-01 10250 10036  9409  9131  9225  9055  9052  9134  8879  8974
    2431   1 2021-05-01 10537 10194  9560  9296  9403  9257  9274  9316  9120  9179
    2432   1 2021-05-01 10241  9950  9328  9070  9042  8880  9001  8852  8861  6646
    2433   1 2021-05-01 10634 10092  9452  9181  9172  8990  9088  8917  8935  7464
    2434   1 2021-05-01 10665 10206  9543  9251  9262  9079  9148  8995  8984  7600
    2435   1 2021-05-01 10555  9858  9244  8966  8983  8810  8907  8702  8759  7563
    2436   1 2021-05-01 10290  9929  9277  8963  8968  8783  8856  8703  8701  7292
    2437   1 2021-05-01 10384  9962  9342  9048  9057  8875  8947  8829  8802  7443
    2438   1 2021-05-01 10181  9746  9157  8909  8927  8780  8878  8693  8760  7245
    2439   1 2021-05-01 10197 10077  9415  9097  9098  8906  8973  8882  8815  7247
    2440   1 2021-05-01 10772 11119 10349  9961  9978  9716  9718  9726  9512  7805
    2441   1 2021-05-01 11650 11700 10930 10553 10595 10310 10299 10292 10079  8833
    2442   1 2021-05-01 12351 11654 10901 10529 10575 10294 10291 10244 10079  9706
    2443   1 2021-05-01 12506 11905 11173 10820 10877 10590 10591 10513 10367  9806
    2444   1 2021-05-01 12348 11757 11038 10711 10768 10499 10506 10391 10309  9582
    2445   1 2021-05-01 12270 11856 11122 10798 10868 10598 10613 10461 10398  9513
    2446   1 2021-05-01 12534 11837 11086 10733 10792 10516 10526 10426 10300  9962
    2447   1 2021-05-01 12542 12060 11293 10945 11007 10723 10718 10660 10487  9901
    2448   1 2021-05-01 12651 12225 11461 11106 11165 10877 10873 10838 10648  9956
    2449   1 2021-05-01 12683 12275 11521 11164 11222 10925 10913 10889 10670 10024
    2450   1 2021-05-01 12792 12438 11652 11271 11323 11013 10990 11025 10736 10217
    2451   1 2021-05-01 13008 12465 11695 11318 11375 11068 11043 11053 10786 10494
    2452   1 2021-05-01 13088 12557 11801 11450 11524 11222 11199 11157 10959 10617
    2453   1 2021-05-01 13036 12398 11627 11277 11351 11061 11047 10992 10806 10549
    2454   1 2021-05-01 12844 12379 11611 11262 11324 11026 11016 10996 10773 10392
    2455   1 2021-05-01 12882 12526 11751 11388 11452 11150 11143 11146 10881 10357
    2456   1 2021-05-01 12810 12497 11749 11407 11471 11182 11183 11158 10932 10253
    2457   1 2021-05-01 12874 12614 11863 11525 11593 11293 11276 11283 11036 10409
    2458   1 2021-05-01 12988 12540 11780 11433 11508 11208 11201 11202 10954 10581
    2459   1 2021-05-01 12946 12569 11830 11506 11590 11297 11295 11257 11068 10565
    2460   1 2021-05-01 12748 12353 11627 11308 11388 11105 11117 11075 10880 10381
    2461   1 2021-05-01 12696 12241 11513 11207 11291 11021 11023 10967 10790 10392
    2462   1 2021-05-01 12593 12070 11342 11040 11123 10855 10856 10816 10644 10319
    2463   1 2021-05-01 12371 12004 11275 10972 11043 10777 10764 10761 10547 10126
    2464   1 2021-05-01 12313 11845 11122 10808 10890 10614 10621 10594 10401 10096
    2465   1 2021-05-01 12238 11671 10972 10676 10762 10508 10525 10467 10305 10014
    2466   1 2021-05-01 11987 11639 10909 10585 10659 10399 10413 10425 10193  9740
    2467   1 2021-05-01 11993 11762 11013 10674 10739 10471 10467 10517 10240  9750
    2468   1 2021-05-01 12686 12181 11439 11115 11201 10919 10903 10916 10679 10566
    2469   1 2021-05-01 12599 12197 11451 11135 11226 10957 10949 10927 10726 10466
    2470   1 2021-05-01 12526 12067 11331 11022 11110 10838 10838 10835 10618 10446
    2471   1 2021-05-01 12321 11935 11214 10913 10998 10738 10740 10700 10529 10219
    2472   1 2021-05-01 10729 10200  9552  9269  9386  9202  9225  9173  9057  9568
    2473   1 2021-05-01 10712 10225  9574  9286  9394  9200  9215  9196  9040  9565
    2474   1 2021-05-01 10751 10234  9591  9299  9412  9220  9217  9192  9041  9609
    2475   1 2021-05-01 10198  9708  9092  8815  8932  8784  8797  8769  8651  9136
    2476   1 2021-05-01 10171  9671  9061  8777  8888  8739  8753  8748  8593  9082
    2477   1 2021-05-01 10049  9515  8912  8647  8762  8616  8640  8625  8484  8990
    2478   1 2021-05-01  9857  9366  8756  8486  8597  8451  8487  8491  8341  8816
    2479   1 2021-05-01  9778  9391  8784  8519  8628  8485  8500  8501  8349  8729
    2480   1 2021-05-01  9926  9498  8889  8627  8745  8599  8619  8581  8472  8864
    2481   1 2021-05-01  9905  9362  8777  8525  8644  8506  8523  8496  8392  8894
    2482   1 2021-05-01  9834  9334  8752  8500  8617  8485  8517  8470  8392  8831
    2483   1 2021-05-01  9642  9033  8479  8239  8363  8243  8281  8262  8159  8728
    2484   1 2021-05-01  9479  9011  8450  8201  8317  8191  8233  8202  8092  8557
    2485   1 2021-05-01  9410  8881  8299  8031  8147  8034  8074  8075  7946  8481
    2486   1 2021-05-01  9546  9185  8564  8263  8363  8222  8234  8282  8078  8564
    2487   1 2021-05-01  9764  9341  8733  8449  8564  8423  8436  8449  8274  8760
    2488   1 2021-05-01  9839  9435  8800  8511  8611  8458  8470  8505  8322  8859
    2489   1 2021-05-01  9992  9592  8926  8628  8723  8557  8575  8612  8421  8989
    2490   1 2021-05-01 10108  9622  8990  8695  8782  8620  8622  8677  8459  9080
    2491   1 2021-05-01 10070  9553  8926  8642  8744  8596  8604  8640  8446  9035
    2492   1 2021-05-01  9879  9495  8884  8602  8692  8537  8552  8629  8378  8833
    2493   1 2021-05-01  9870  9473  8867  8595  8697  8532  8541  8601  8371  8808
    2494   1 2021-05-01  9984  9539  8915  8638  8736  8597  8635  8664  8487  8848
    2495   1 2021-05-01  9984  9539  8915  8638  8736  8597  8635  8664  8487  8836
    2496   1 2021-05-01  9924  9752  9094  8791  8871  8706  8720  8854  8563  8756
    2497   1 2021-05-01 10271  9901  9272  8979  9052  8882  8870  9042  8678  8996
    2498   1 2021-05-01 10488 10183  9506  9200  9202  9001  9077  8993  8901  7271
    2499   1 2021-05-01 10603 10194  9531  9226  9242  9051  9118  8973  8951  7554
    2500   1 2021-05-01 10568 10095  9433  9130  9140  8955  9021  8873  8857  7676
    2501   1 2021-05-01 10610 10095  9433  9130  9140  8955  9021  8873  8857  7709
    2502   1 2021-05-01 10623 10128  9478  9169  9187  9002  9060  8913  8901  7751
    2503   1 2021-05-01 10550  9746  9157  8909  8927  8780  8878  8693  8760  7725
    2504   1 2021-05-01 10372  9931  9285  8988  8994  8820  8907  8813  8775  7489
    2505   1 2021-05-01 11198 10904 10149  9775  9786  9537  9557  9566  9363  8353
    2506   1 2021-05-01 11722 11539 10758 10383 10408 10126 10117 10155  9911  8965
    2507   1 2021-05-01 12267 11961 11191 10811 10854 10561 10545 10554 10317  9642
    2508   1 2021-05-01 12514 12050 11285 10915 10960 10668 10654 10644 10418  9847
    2509   1 2021-05-01 12590 12083 11333 11001 11060 10778 10772 10695 10543  9927
    2510   1 2021-05-01 12685 12228 11474 11129 11200 10909 10908 10832 10675 10076
    2511   1 2021-05-01 12697 12200 11454 11125 11211 10930 10927 10822 10704 10140
    2512   1 2021-05-01 12680 12185 11456 11134 11202 10934 10945 10849 10721 10066
    2513   1 2021-05-01 12535 12167 11405 11043 11093 10803 10798 10786 10564  9859
    2514   1 2021-05-01 12583 12167 11405 11043 11093 10803 10798 10786 10564  9951
    2515   1 2021-05-01 12743 12290 11537 11188 11248 10958 10945 10903 10693 10141
    2516   1 2021-05-01 12832 12465 11695 11318 11375 11068 11043 11053 10786 10318
    2517   1 2021-05-01 12859 12411 11636 11266 11332 11028 10999 10995 10757 10406
    2518   1 2021-05-01 12973 12443 11692 11366 11455 11161 11150 11044 10911 10521
    2519   1 2021-05-01 12892 12369 11616 11291 11370 11081 11084 10990 10853 10463
    2520   1 2021-05-01 12779 12345 11590 11250 11311 11016 11020 10999 10765 10337
    2521   1 2021-05-01 12635 12236 11478 11145 11204 10912 10911 10888 10673 10152
    2522   1 2021-05-01 12717 12346 11600 11261 11329 11040 11039 11022 10798 10259
    2523   1 2021-05-01 12768 12234 11497 11168 11235 10959 10954 10946 10719 10351
    2524   1 2021-05-01 12684 12230 11514 11195 11267 11001 11001 10963 10765 10267
    2525   1 2021-05-01 12416 11938 11216 10896 10961 10694 10705 10691 10476 10037
    2526   1 2021-05-01 12469 11993 11283 10984 11055 10781 10786 10768 10566 10130
    2527   1 2021-05-01 12427 11801 11088 10782 10848 10573 10580 10581 10360 10145
    2528   1 2021-05-01 12294 11924 11203 10892 10969 10700 10691 10690 10481 10030
    2529   1 2021-05-01 12185 11845 11122 10808 10890 10614 10621 10594 10401  9946
    2530   1 2021-05-01 12050 11653 10940 10634 10720 10461 10466 10419 10247  9810
    2531   1 2021-05-01 11915 11495 10791 10485 10567 10320 10340 10290 10124  9671
    2532   1 2021-05-01 11901 11549 10816 10496 10569 10309 10319 10306 10103  9650
    2533   1 2021-05-01 12127 11924 11173 10835 10902 10618 10606 10673 10378  9919
    2534   1 2021-05-01 12425 11887 11161 10842 10924 10649 10650 10674 10428 10255
    2535   1 2021-05-01 12265 11893 11164 10852 10937 10665 10670 10667 10450 10109
    2536   1 2021-05-01 12191 11646 10936 10622 10705 10454 10464 10453 10256 10048
    2537   1 2021-05-01 11410 11010 10347 10065 10167  9954  9973  9906  9785  9440
    2538   1 2021-05-01 10351  9859  9234  8953  9072  8909  8925  8914  8762  9265
    2539   1 2021-05-01 10263  9752  9130  8846  8970  8810  8822  8819  8671  9164
    2540   1 2021-05-01 10209  9756  9139  8866  8989  8824  8843  8827  8674  9125
    2541   1 2021-05-01  9889  9608  8993  8719  8832  8690  8706  8713  8543  8812
    2542   1 2021-05-01  9752  9425  8832  8560  8677  8537  8561  8561  8411  8697
    2543   1 2021-05-01  9614  9242  8657  8386  8499  8369  8410  8407  8267  8559
    2544   1 2021-05-01  9728  9395  8784  8505  8614  8467  8482  8525  8333  8663
    2545   1 2021-05-01  9904  9481  8873  8603  8717  8576  8594  8621  8445  8837
    2546   1 2021-05-01 10013  9488  8897  8645  8772  8635  8666  8581  8527  8943
    2547   1 2021-05-01  9972  9444  8857  8607  8738  8605  8646  8538  8509  8890
    2548   1 2021-05-01  9731  9173  8603  8351  8474  8343  8392  8324  8261  8734
    2549   1 2021-05-01  9600  9178  8587  8322  8441  8311  8353  8311  8212  8630
    2550   1 2021-05-01  9508  9125  8522  8235  8340  8210  8244  8248  8101  8506
    2551   1 2021-05-01  9726  9372  8751  8458  8568  8417  8427  8440  8253  8707
    2552   1 2021-05-01  9861  9463  8834  8547  8651  8513  8527  8522  8361  8853
    2553   1 2021-05-01  9958  9537  8900  8624  8740  8595  8608  8573  8458  8939
    2554   1 2021-05-01 10075  9537  8900  8624  8740  8595  8608  8573  8458  9041
    2555   1 2021-05-01 10053  9639  8983  8679  8774  8613  8626  8663  8475  9040
    2556   1 2021-05-01 10058  9542  8914  8647  8761  8616  8639  8603  8490  9038
    2557   1 2021-05-01 10016  9481  8852  8571  8679  8536  8559  8572  8416  8942
    2558   1 2021-05-01  9678  9262  8681  8416  8513  8380  8415  8413  8282  8628
    2559   1 2021-05-01  9901  9413  8809  8554  8667  8522  8558  8541  8401  8770
    2560   1 2021-05-01  9933  9512  8883  8608  8711  8567  8601  8598  8460  8785
    2561   1 2021-05-01 10026  9775  9127  8827  8903  8743  8754  8885  8578  8848
    2562   1 2021-05-01 10174  9939  9331  9065  9157  8995  9011  9054  8836  8942
    2563   1 2021-05-01  9931  9446  8854  8606  8596  8457  8562  8408  8432  6752
    2564   1 2021-05-01 10253  9939  9275  8968  8959  8774  8852  8786  8699  7156
    2565   1 2021-05-01 10638 10230  9546  9225  9230  9024  9083  9012  8918  7672
    2566   1 2021-05-01 10699 10176  9521  9219  9243  9053  9113  8968  8955  7829
    2567   1 2021-05-01 10652 10295  9625  9321  9336  9148  9209  9101  9051  7787
    2568   1 2021-05-01 10558 10188  9562  9290  9310  9123  9212  9113  9063  7691
    2569   1 2021-05-01 10373  9747  9138  8887  8907  8750  8857  8714  8720  7580
    2570   1 2021-05-01 10072  9636  8993  8700  8698  8524  8632  8578  8489  7246
    2571   1 2021-05-01 10437 10960 10180  9784  9779  9513  9529  9645  9324  7528
    2572   1 2021-05-01 11613 11598 10805 10406 10430 10143 10127 10199  9890  8831
    2573   1 2021-05-01 12348 12030 11278 10940 11006 10729 10726 10630 10511  9684
    2574   1 2021-05-01 12493 12030 11278 10940 11006 10729 10726 10630 10511  9845
    2575   1 2021-05-01 12787 12141 11369 11016 11074 10778 10765 10738 10530 10214
    2576   1 2021-05-01 12771 12280 11518 11175 11243 10957 10947 10894 10724 10203
    2577   1 2021-05-01 12836 12345 11611 11268 11345 11050 11047 10992 10817 10247
    2578   1 2021-05-01 12741 12265 11539 11219 11280 11005 11014 10949 10787 10144
    2579   1 2021-05-01 12610 12092 11356 11034 11085 10817 10834 10743 10617  9948
    2580   1 2021-05-01 12689 12236 11476 11121 11177 10889 10871 10852 10628 10162
    2581   1 2021-05-01 12829 12356 11607 11266 11333 11050 11042 10975 10805 10307
    2582   1 2021-05-01 12817 12351 11578 11222 11295 10985 10975 10931 10727 10347
    2583   1 2021-05-01 12847 12419 11643 11302 11374 11071 11051 11019 10814 10420
    2584   1 2021-05-01 12967 12437 11683 11368 11432 11150 11138 11074 10907 10529
    2585   1 2021-05-01 12836 12307 11552 11231 11294 11009 11002 10960 10771 10423
    2586   1 2021-05-01 12772 12252 11500 11176 11241 10964 10951 10905 10735 10360
    2587   1 2021-05-01 12699 12252 11500 11176 11241 10964 10951 10905 10735 10267
    2588   1 2021-05-01 12505 12175 11431 11105 11162 10878 10882 10858 10651 10061
    2589   1 2021-05-01 12372 12017 11284 10960 11016 10740 10734 10726 10504  9931
    2590   1 2021-05-01 12366 12056 11337 11025 11084 10825 10819 10814 10594  9886
    2591   1 2021-05-01 12243 11711 11000 10703 10763 10514 10542 10475 10325  9821
    2592   1 2021-05-01 12081 11715 10999 10683 10748 10489 10496 10481 10274  9715
    2593   1 2021-05-01 11961 11801 11088 10782 10848 10573 10580 10581 10360  9620
    2594   1 2021-05-01 12018 11595 10873 10549 10612 10358 10354 10353 10141  9701
    2595   1 2021-05-01 12080 11632 10920 10612 10691 10433 10435 10404 10220  9795
    2596   1 2021-05-01 11973 11506 10792 10474 10554 10305 10304 10256 10096  9693
    2597   1 2021-05-01 11992 11506 10774 10450 10527 10270 10279 10239 10057  9785
    2598   1 2021-05-01 12104 11735 11001 10674 10764 10500 10493 10458 10272  9910
    2599   1 2021-05-01 12177 11699 10983 10676 10760 10511 10515 10459 10307 10009
    2600   1 2021-05-01 12176 11652 10944 10640 10721 10472 10487 10437 10278 10021
    2601   1 2021-05-01 12041 11727 11006 10699 10778 10521 10530 10505 10312  9869
    2602   1 2021-05-01 11984 11574 10852 10544 10630 10387 10398 10365 10190  9823
    2603   1 2021-05-01 11389 10958 10282  9994 10073  9864  9888  9880  9691  9374
    2604   1 2021-05-01 11185 10775 10123  9838  9931  9745  9761  9723  9576  9214
    2605   1 2021-05-01 10907 10589  9939  9655  9745  9560  9586  9573  9414  8954
    2606   1 2021-05-01 10470 10082  9458  9180  9299  9133  9147  9133  8989  9269
    2607   1 2021-05-01 10433 10073  9447  9163  9278  9112  9124  9101  8959  9266
    2608   1 2021-05-01 10400  9839  9217  8932  9045  8894  8904  8905  8746  9224
    2609   1 2021-05-01  9865  9808  9185  8906  9017  8856  8868  8869  8709  8734
    2610   1 2021-05-01  9953  9617  8995  8710  8819  8658  8675  8704  8519  8818
    2611   1 2021-05-01  9910  9458  8857  8584  8703  8562  8588  8589  8445  8794
    2612   1 2021-05-01  9680  9112  8534  8262  8375  8256  8296  8312  8160  8622
    2613   1 2021-05-01  9430  9178  8582  8292  8380  8250  8271  8348  8123  8343
    2614   1 2021-05-01  9669  9243  8638  8355  8442  8309  8328  8398  8174  8525
    2615   1 2021-05-01  9741  9398  8790  8509  8621  8486  8507  8533  8357  8651
    2616   1 2021-05-01  9860  9501  8897  8626  8742  8606  8630  8637  8483  8734
    2617   1 2021-05-01 10008  9509  8915  8653  8772  8635  8662  8643  8510  8912
    2618   1 2021-05-01  9861  9509  8915  8653  8772  8635  8662  8643  8510  8754
    2619   1 2021-05-01  9739  9330  8746  8494  8614  8490  8532  8454  8406  8683
    2620   1 2021-05-01  9590  9178  8587  8322  8441  8311  8353  8311  8212  8547
    2621   1 2021-05-01  9498  9047  8462  8205  8331  8211  8261  8213  8132  8491
    2622   1 2021-05-01  9553  9285  8650  8345  8442  8294  8312  8374  8157  8507
    2623   1 2021-05-01  9781  9413  8785  8491  8596  8453  8470  8490  8300  8679
    2624   1 2021-05-01 10022  9572  8943  8661  8769  8629  8637  8611  8492  8940
    2625   1 2021-05-01 10108  9631  9007  8727  8841  8698  8708  8675  8556  9020
    2626   1 2021-05-01  9981  9530  8923  8657  8772  8651  8682  8616  8544  8843
    2627   1 2021-05-01  9725  9328  8725  8462  8567  8457  8505  8460  8369  8557
    2628   1 2021-05-01  9881  9343  8756  8509  8621  8505  8556  8458  8422  8665
    2629   1 2021-05-01  9753  9366  8786  8534  8637  8510  8546  8477  8400  8650
    2630   1 2021-05-01  9858  9525  8900  8635  8747  8613  8647  8608  8499  8692
    2631   1 2021-05-01 10037  9722  9061  8748  8827  8669  8682  8787  8519  8878
    2632   1 2021-05-01 10176  9777  9112  8808  8886  8724  8728  8841  8554  8952
    2633   1 2021-05-01  9721  9333  8754  8516  8502  8375  8492  8320  8370  6511
    2634   1 2021-05-01  9686  9680  9043  8747  8725  8565  8666  8576  8524  6504
    2635   1 2021-05-01 10522 10139  9461  9143  9144  8951  9021  8927  8859  7449
    2636   1 2021-05-01 10695 10234  9583  9292  9308  9112  9185  9058  9039  7702
    2637   1 2021-05-01 10629  9934  9314  9043  9032  8861  8963  8849  8810  7675
    2638   1 2021-05-01 10312  9986  9361  9089  9092  8914  9000  8940  8851  7327
    2639   1 2021-05-01 10301  9986  9361  9089  9092  8914  9000  8940  8851  7432
    2640   1 2021-05-01 10023  9676  9103  8869  8900  8743  8860  8687  8729  7239
    2641   1 2021-05-01  9779  9451  8866  8615  8644  8495  8612  8441  8479  6963
    2642   1 2021-05-01 10249 10247  9529  9164  9165  8937  9000  9015  8818  7385
    2643   1 2021-05-01 12003 11277 10480 10072 10091  9818  9810  9880  9586  9328
    2644   1 2021-05-01 12636 12148 11389 11045 11104 10807 10804 10781 10571  9990
    2645   1 2021-05-01 12778 12347 11572 11232 11305 11007 11012 10975 10785 10179
    2646   1 2021-05-01 12842 12341 11588 11250 11322 11029 11022 10977 10800 10311
    2647   1 2021-05-01 12698 12205 11467 11127 11198 10910 10906 10875 10679 10116
    2648   1 2021-05-01 12631 12133 11406 11077 11131 10861 10863 10830 10650  9989
    2649   1 2021-05-01 12458 12131 11403 11086 11132 10856 10864 10826 10647  9783
    2650   1 2021-05-01 12481 12012 11243 10881 10921 10629 10622 10653 10381  9808
    2651   1 2021-05-01 12631 12236 11470 11110 11154 10865 10841 10869 10591 10066
    2652   1 2021-05-01 12451 12236 11470 11110 11154 10865 10841 10869 10591  9896
    2653   1 2021-05-01 12717 12342 11571 11225 11296 11002 10988 10947 10750 10214
    2654   1 2021-05-01 12851 12362 11588 11246 11316 11022 10996 10978 10766 10396
    2655   1 2021-05-01 12912 12423 11658 11335 11393 11103 11088 11056 10850 10466
    2656   1 2021-05-01 12683 12295 11536 11211 11282 10989 10986 10947 10737 10253
    2657   1 2021-05-01 12686 12283 11525 11199 11265 10983 10968 10954 10729 10272
    2658   1 2021-05-01 12656 12132 11387 11070 11134 10863 10850 10816 10626 10258
    2659   1 2021-05-01 12524 11971 11235 10919 10986 10716 10723 10652 10501 10130
    2660   1 2021-05-01 12308 11857 11134 10816 10883 10613 10621 10569 10404  9928
    2661   1 2021-05-01 12128 11654 10956 10658 10724 10473 10495 10430 10280  9701
    2662   1 2021-05-01 12032 11602 10889 10586 10656 10406 10420 10348 10209  9620
    2663   1 2021-05-01 12000 11425 10720 10414 10481 10229 10247 10183 10035  9611
    2664   1 2021-05-01 11884 11378 10661 10331 10385 10125 10129 10133  9906  9540
    2665   1 2021-05-01 11816 11365 10644 10314 10374 10121 10121 10120  9909  9471
    2666   1 2021-05-01 11844 11486 10766 10445 10512 10266 10267 10242 10059  9527
    2667   1 2021-05-01 11974 11499 10767 10438 10513 10257 10252 10217 10036  9724
    2668   1 2021-05-01 12121 11618 10884 10559 10638 10383 10387 10327 10172  9932
    2669   1 2021-05-01 12299 11807 11072 10747 10835 10567 10571 10523 10358 10153
    2670   1 2021-05-01 12242 11774 11061 10760 10847 10593 10594 10540 10385 10083
    2671   1 2021-05-01 12176 11686 10983 10687 10772 10516 10529 10480 10324 10016
    2672   1 2021-05-01 12066 11576 10859 10552 10645 10398 10407 10359 10201  9906
    2673   1 2021-05-01 11523 10937 10271  9992 10078  9876  9895  9819  9704  9508
    2674   1 2021-05-01 11237 10892 10249  9978 10081  9891  9913  9815  9730  9249
    2675   1 2021-05-01 10941 10428  9804  9534  9633  9470  9506  9446  9346  8986
    2676   1 2021-05-01 10678 10271  9625  9323  9402  9233  9262  9250  9097  8707
    2677   1 2021-05-01 10634 10251  9634  9351  9436  9262  9298  9273  9120  8738
    2678   1 2021-05-01 10838 10373  9733  9449  9562  9379  9399  9376  9232  9555
    2679   1 2021-05-01 10767 10250  9624  9356  9477  9303  9316  9266  9148  9498
    2680   1 2021-05-01 10611 10037  9430  9164  9283  9115  9131  9100  8979  9377
    2681   1 2021-05-01 10424  9931  9341  9082  9202  9045  9065  9011  8901  9175
    2682   1 2021-05-01 10237  9692  9099  8836  8955  8819  8851  8799  8712  9026
    2683   1 2021-05-01 10190  9559  8995  8755  8887  8751  8804  8690  8661  8964
    2684   1 2021-05-01  9906  9362  8786  8524  8637  8518  8563  8529  8427  8744
    2685   1 2021-05-01  9684  9395  8820  8563  8677  8565  8599  8557  8453  8548
    2686   1 2021-05-01  9699  9170  8629  8389  8519  8410  8458  8362  8332  8495
    2687   1 2021-05-01  9688  9170  8629  8389  8519  8410  8458  8362  8332  8464
    2688   1 2021-05-01  9018  8831  8264  7987  8095  7986  8046  8063  7922  7932
    2689   1 2021-05-01  9227  8842  8239  7934  8018  7893  7931  8058  7787  8028
    2690   1 2021-05-01  9692  9310  8711  8433  8537  8410  8445  8457  8297  8514
    2691   1 2021-05-01  9850  9431  8844  8586  8708  8577  8616  8581  8468  8713
    2692   1 2021-05-01  9838  9379  8804  8556  8673  8549  8592  8572  8462  8741
    2693   1 2021-05-01  9679  9296  8702  8439  8549  8429  8468  8466  8331  8595
    2694   1 2021-05-01  9766  9276  8696  8447  8559  8449  8486  8439  8361  8581
    2695   1 2021-05-01  9769  9178  8614  8376  8500  8387  8439  8351  8309  8621
    2696   1 2021-05-01  9540  9066  8515  8288  8421  8318  8378  8260  8262  8454
    2697   1 2021-05-01  9482  9211  8591  8304  8404  8273  8304  8343  8141  8370
    2698   1 2021-05-01  9596  9280  8667  8388  8492  8369  8393  8437  8246  8465
    2699   1 2021-05-01  9817  9576  8939  8637  8743  8599  8617  8662  8470  8668
    2700   1 2021-05-01  9889  9540  8944  8680  8798  8680  8703  8671  8556  8677
    2701   1 2021-05-01  9770  9258  8699  8457  8585  8491  8537  8466  8406  8544
    2702   1 2021-05-01  9636  9269  8651  8378  8476  8358  8403  8410  8266  8407
    2703   1 2021-05-01  9922  9437  8815  8538  8641  8512  8542  8524  8399  8686
    2704   1 2021-05-01 10061  9556  8943  8680  8777  8636  8652  8619  8488  8870
    2705   1 2021-05-01 10074  9722  9061  8748  8827  8669  8682  8787  8519  8905
    2706   1 2021-05-01  9590  9257  8703  8493  8486  8369  8498  8300  8391  6412
    2707   1 2021-05-01  9753  9680  9043  8747  8725  8565  8666  8576  8524  6577
    2708   1 2021-05-01 10311 10008  9353  9046  9037  8861  8941  8865  8780  7175
    2709   1 2021-05-01 10543 10175  9524  9238  9249  9063  9135  9018  8981  7483
    2710   1 2021-05-01 10160  9793  9195  8941  8942  8786  8902  8737  8754  7106
    2711   1 2021-05-01 10134  9887  9271  9001  9003  8830  8924  8825  8755  7087
    2712   1 2021-05-01 10318  9923  9331  9083  9108  8941  9033  8872  8884  7424
    2713   1 2021-05-01 10135  9705  9108  8855  8869  8709  8812  8667  8670  7302
    2714   1 2021-05-01 10442 10196  9521  9200  9210  9010  9079  9036  8920  7579
    2715   1 2021-05-01 11158 11096 10335  9968  9999  9747  9758  9743  9560  8358
    2716   1 2021-05-01 11827 11691 10918 10541 10577 10300 10293 10329 10066  9161
    2717   1 2021-05-01 12476 12244 11478 11128 11188 10899 10896 10898 10668  9849
    2718   1 2021-05-01 12778 12306 11559 11217 11290 11000 10995 10971 10772 10177
    2719   1 2021-05-01 12744 12361 11605 11264 11341 11043 11034 11010 10819 10197
    2720   1 2021-05-01 12594 12205 11467 11127 11198 10910 10906 10875 10679 10016
    2721   1 2021-05-01 12377 11953 11237 10897 10950 10683 10679 10651 10464  9740
    2722   1 2021-05-01 12064 11879 11154 10825 10871 10600 10620 10574 10399  9348
    2723   1 2021-05-01 12088 11645 10897 10535 10583 10300 10309 10274 10074  9457
    2724   1 2021-05-01 12179 11855 11087 10718 10757 10468 10453 10465 10203  9636
    2725   1 2021-05-01 12286 11871 11110 10756 10805 10528 10501 10470 10265  9770
    2726   1 2021-05-01 12537 12191 11406 11059 11107 10806 10784 10815 10543 10036
    2727   1 2021-05-01 12651 12069 11302 10965 11016 10729 10717 10703 10477 10166
    2728   1 2021-05-01 12503 12120 11356 11013 11067 10776 10764 10766 10515 10038
    2729   1 2021-05-01 12527 11968 11206 10866 10923 10632 10621 10618 10374 10100
    2730   1 2021-05-01 12486 12021 11278 10964 11025 10749 10751 10717 10520 10058
    2731   1 2021-05-01 12357 11846 11121 10820 10878 10616 10621 10579 10409  9948
    2732   1 2021-05-01 12287 11791 11058 10738 10792 10525 10527 10515 10313  9898
    2733   1 2021-05-01 12280 11857 11134 10816 10883 10613 10621 10569 10404  9903
    2734   1 2021-05-01 11875 11624 10927 10630 10696 10450 10470 10412 10257  9456
    2735   1 2021-05-01 11910 11524 10811 10497 10565 10311 10330 10292 10111  9503
    2736   1 2021-05-01 11844 11362 10651 10339 10406 10158 10178 10104  9975  9506
    2737   1 2021-05-01 11673 11179 10467 10142 10205  9960  9969  9906  9762  9348
    2738   1 2021-05-01 11627 11151 10444 10131 10184  9951  9963  9935  9753  9268
    2739   1 2021-05-01 11763 11470 10737 10408 10473 10213 10208 10203  9999  9428
    2740   1 2021-05-01 12141 11699 10950 10615 10685 10419 10407 10420 10185  9907
    2741   1 2021-05-01 12309 11828 11095 10773 10854 10591 10581 10556 10358 10129
    2742   1 2021-05-01 12278 11707 10986 10680 10761 10515 10515 10474 10307 10103
    2743   1 2021-05-01 12071 11634 10932 10631 10712 10465 10474 10444 10273  9875
    2744   1 2021-05-01 11971 11445 10753 10443 10525 10291 10291 10272 10088  9776
    2745   1 2021-05-01 11939 11390 10696 10385 10465 10226 10239 10209 10037  9761
    2746   1 2021-05-01 11779 11257 10567 10276 10362 10152 10170 10111  9976  9739
    2747   1 2021-05-01 11551 11124 10455 10167 10264 10059 10071 10030  9874  9541
    2748   1 2021-05-01 11225 10675 10052  9799  9898  9721  9764  9684  9585  9231
    2749   1 2021-05-01 10710 10165  9549  9282  9383  9230  9281  9176  9118  8754
    2750   1 2021-05-01 10563 10115  9489  9212  9303  9149  9195  9120  9034  8580
    2751   1 2021-05-01 10514 10203  9560  9274  9361  9201  9229  9205  9060  8569
    2752   1 2021-05-01 10596 10114  9477  9194  9288  9123  9161  9140  8995  8730
    2753   1 2021-05-01 10613 10215  9551  9255  9342  9176  9199  9204  9025  8823
    2754   1 2021-05-01 11266 10550  9949  9716  9856  9688  9717  9548  9559  9901
    2755   1 2021-05-01 10937 10234  9650  9411  9548  9402  9439  9299  9293  9599
    2756   1 2021-05-01 10623 10317  9679  9407  9530  9352  9380  9333  9206  9324
    2757   1 2021-05-01 10866 10375  9728  9438  9549  9366  9372  9376  9214  9613
    2758   1 2021-05-01 10905 10316  9695  9413  9537  9353  9360  9325  9196  9636
    2759   1 2021-05-01 10633 10042  9442  9181  9298  9134  9151  9098  8991  9408
    2760   1 2021-05-01 10515 10042  9442  9181  9298  9134  9151  9098  8991  9228
    2761   1 2021-05-01 10517  9902  9312  9063  9186  9033  9066  8984  8914  9238
    2762   1 2021-05-01 10261  9824  9241  8987  9106  8955  8990  8933  8837  8994
    2763   1 2021-05-01 10085  9493  8957  8722  8857  8733  8782  8683  8646  8867
    2764   1 2021-05-01  9842  9588  9029  8797  8924  8804  8841  8772  8705  8667
    2765   1 2021-05-01 10035  9541  8975  8730  8853  8726  8757  8689  8620  8804
    2766   1 2021-05-01  9825  9418  8877  8649  8780  8660  8707  8610  8584  8591
    2767   1 2021-05-01  8864  8546  8029  7786  7903  7821  7897  7843  7774  7752
    2768   1 2021-05-01  9220  9218  8627  8342  8434  8305  8337  8382  8177  7994
    2769   1 2021-05-01  9757  9504  8927  8673  8798  8672  8713  8669  8579  8507
    2770   1 2021-05-01 10063  9463  8906  8663  8799  8671  8720  8632  8587  8856
    2771   1 2021-05-01  9931  9526  8955  8719  8855  8732  8779  8677  8648  8770
    2772   1 2021-05-01 10025  9412  8850  8612  8732  8604  8640  8589  8497  8814
    2773   1 2021-05-01  9925  9412  8850  8612  8732  8604  8640  8589  8497  8742
    2774   1 2021-05-01  9869  9350  8779  8535  8654  8544  8588  8497  8462  8665
    2775   1 2021-05-01  9773  9373  8802  8561  8683  8574  8628  8519  8495  8595
    2776   1 2021-05-01  9790  9179  8602  8347  8463  8355  8407  8337  8272  8596
    2777   1 2021-05-01  9589  9204  8637  8386  8505  8393  8440  8381  8304  8383
    2778   1 2021-05-01  9482  9236  8636  8353  8467  8348  8391  8428  8250  8342
    2779   1 2021-05-01  9293  9135  8551  8281  8382  8287  8338  8384  8216  8083
    2780   1 2021-05-01  9487  9155  8579  8315  8424  8326  8375  8354  8247  8237
    2781   1 2021-05-01  9595  9377  8772  8503  8610  8487  8530  8492  8386  8329
    2782   1 2021-05-01  9857  9555  8917  8624  8717  8574  8594  8601  8432  8715
    2783   1 2021-05-01 10026  9537  8889  8593  8689  8534  8557  8570  8400  8910
    2784   1 2021-05-01 10013  9736  9072  8727  8792  8607  8606  8833  8427  8892
    2785   1 2021-05-01  9720  9295  8724  8493  8476  8343  8455  8340  8338  6511
    2786   1 2021-05-01  9620  9287  8712  8456  8442  8303  8407  8309  8283  6402
    2787   1 2021-05-01  9480  9521  8893  8601  8574  8417  8516  8489  8379  6272
    2788   1 2021-05-01  9961  9637  8978  8656  8622  8443  8527  8535  8366  6759
    2789   1 2021-05-01 10134  9848  9242  8971  8973  8807  8900  8740  8750  6973
    2790   1 2021-05-01 10369  9995  9358  9070  9091  8900  8973  8844  8805  7405
    2791   1 2021-05-01 10600 10227  9611  9344  9372  9182  9256  9119  9083  7728
    2792   1 2021-05-01 10567 10013  9430  9191  9221  9049  9140  8964  8987  7729
    2793   1 2021-05-01 10353  9848  9253  9007  9029  8870  8974  8803  8828  7513
    2794   1 2021-05-01 10277  9846  9213  8929  8935  8765  8852  8774  8701  7446
    2795   1 2021-05-01 11300 10819 10063  9693  9696  9445  9465  9545  9263  8563
    2796   1 2021-05-01 11835 11430 10649 10270 10296 10018 10016 10084  9796  9185
    2797   1 2021-05-01 12353 12008 11247 10892 10940 10658 10649 10658 10419  9740
    2798   1 2021-05-01 12621 12306 11559 11217 11290 11000 10995 10971 10772 10003
    2799   1 2021-05-01 12522 12076 11360 11045 11111 10833 10845 10772 10635  9902
    2800   1 2021-05-01 12169 11716 10994 10658 10710 10451 10465 10412 10258  9515
    2801   1 2021-05-01 11933 11448 10733 10405 10462 10214 10226 10110 10022  9228
    2802   1 2021-05-01 11942 11623 10876 10515 10573 10293 10296 10231 10064  9235
    2803   1 2021-05-01 12352 11946 11200 10867 10930 10648 10646 10563 10414  9828
    2804   1 2021-05-01 12493 11938 11196 10873 10928 10655 10641 10563 10414 10026
    2805   1 2021-05-01 12463 11931 11183 10858 10917 10638 10629 10558 10403 10005
    2806   1 2021-05-01 12437 11963 11200 10857 10911 10627 10603 10580 10381  9945
    2807   1 2021-05-01 12345 11898 11153 10818 10863 10584 10578 10539 10350  9855
    2808   1 2021-05-01 12351 11898 11140 10800 10855 10569 10571 10518 10340  9899
    2809   1 2021-05-01 12296 11809 11063 10735 10793 10516 10527 10436 10309  9841
    2810   1 2021-05-01 12363 11853 11115 10794 10845 10571 10579 10557 10338  9892
    2811   1 2021-05-01 12055 11846 11121 10820 10878 10616 10621 10579 10409  9592
    2812   1 2021-05-01 12084 11645 10921 10615 10674 10413 10419 10376 10206  9649
    2813   1 2021-05-01 12064 11565 10869 10574 10638 10390 10413 10339 10211  9647
    2814   1 2021-05-01 11969 11503 10807 10515 10574 10321 10337 10278 10129  9536
    2815   1 2021-05-01 11969 11537 10829 10513 10580 10331 10339 10281 10131  9557
    2816   1 2021-05-01 11903 11384 10665 10334 10382 10135 10135 10141  9912  9567
    2817   1 2021-05-01 11720 11116 10444 10156 10212  9999 10022  9941  9823  9388
    2818   1 2021-05-01 11494 11148 10460 10166 10231  9992 10020  9969  9816  9169
    2819   1 2021-05-01 11603 11477 10735 10407 10469 10236 10239 10223 10031  9262
    2820   1 2021-05-01 12048 11477 10735 10407 10469 10236 10239 10223 10031  9781
    2821   1 2021-05-01 11953 11717 10988 10663 10732 10478 10476 10464 10258  9655
    2822   1 2021-05-01 12031 11611 10907 10612 10681 10435 10442 10439 10252  9788
    2823   1 2021-05-01 11921 11490 10784 10485 10557 10315 10325 10320 10130  9690
    2824   1 2021-05-01 11785 11445 10753 10443 10525 10291 10291 10272 10088  9550
    2825   1 2021-05-01 11325 10617  9974  9703  9789  9609  9650  9595  9469  9225
    2826   1 2021-05-01 10855 10617  9974  9703  9789  9609  9650  9595  9469  8822
    2827   1 2021-05-01 10766 10361  9746  9492  9595  9427  9479  9372  9308  8797
    2828   1 2021-05-01 10590 10195  9593  9335  9435  9282  9335  9234  9175  8657
    2829   1 2021-05-01 10238 10114  9477  9194  9288  9123  9161  9140  8995  8358
    2830   1 2021-05-01 10355  9966  9313  9009  9082  8930  8960  8994  8787  8522
    2831   1 2021-05-01 10409 10334  9666  9368  9459  9292  9302  9308  9132  8588
    2832   1 2021-05-01 10794 10472  9815  9527  9625  9446  9460  9462  9282  9016
    2833   1 2021-05-01 11262 10684 10071  9823  9955  9780  9816  9754  9651  9888
    2834   1 2021-05-01 11233 10712 10075  9817  9947  9779  9809  9749  9639  9879
    2835   1 2021-05-01 11246 10712 10075  9817  9947  9779  9809  9749  9639  9880
    2836   1 2021-05-01 11107 10660 10049  9794  9935  9767  9783  9676  9609  9749
    2837   1 2021-05-01 10883 10234  9650  9411  9548  9402  9439  9299  9293  9533
    2838   1 2021-05-01 10841 10315  9718  9466  9604  9442  9465  9360  9300  9461
    2839   1 2021-05-01 11135 10452  9797  9531  9664  9499  9528  9411  9389  9775
    2840   1 2021-05-01 11141 10594  9931  9670  9811  9639  9661  9538  9519  9803
    2841   1 2021-05-01 11026 10391  9775  9507  9632  9458  9471  9399  9310  9742
    2842   1 2021-05-01 10673 10261  9672  9426  9565  9406  9428  9334  9277  9392
    2843   1 2021-05-01 10548  9985  9398  9139  9265  9114  9144  9097  8983  9268
    2844   1 2021-05-01 10416 10107  9503  9236  9363  9206  9233  9165  9075  9134
    2845   1 2021-05-01 10376  9885  9311  9065  9192  9046  9068  9028  8913  9130
    2846   1 2021-05-01 10337  9847  9268  9028  9148  9013  9040  9012  8898  9097
    2847   1 2021-05-01 10180  9617  9051  8809  8927  8786  8817  8775  8679  8909
    2848   1 2021-05-01  9981  9364  8848  8633  8777  8658  8710  8569  8579  8734
    2849   1 2021-05-01  9177  8796  8320  8119  8261  8191  8273  8096  8171  7949
    2850   1 2021-05-01  9841  9218  8627  8342  8434  8305  8337  8382  8177  8533
    2851   1 2021-05-01 10056  9612  9034  8781  8901  8769  8807  8771  8671  8789
    2852   1 2021-05-01 10189  9691  9120  8876  9006  8871  8911  8828  8769  8946
    2853   1 2021-05-01 10368  9794  9220  8994  9132  9003  9050  8928  8915  9097
    2854   1 2021-05-01 10241  9641  9059  8820  8948  8816  8870  8788  8729  9015
    2855   1 2021-05-01 10018  9603  9033  8796  8920  8800  8849  8785  8715  8821
    2856   1 2021-05-01  9950  9543  8978  8739  8856  8736  8776  8717  8633  8747
    2857   1 2021-05-01 10037  9620  9044  8810  8925  8801  8844  8796  8700  8821
    2858   1 2021-05-01  9973  9478  8908  8666  8789  8675  8720  8654  8583  8726
    2859   1 2021-05-01  9817  9331  8758  8503  8617  8508  8557  8536  8424  8516
    2860   1 2021-05-01  9659  9155  8579  8315  8424  8326  8375  8354  8247  8378
    2861   1 2021-05-01  9819  9392  8773  8491  8588  8461  8502  8468  8356  8634
    2862   1 2021-05-01  9989  9421  8790  8507  8617  8478  8519  8452  8374  8849
    2863   1 2021-05-01  8569  8012  7440  7160  7099  6992  7120  7160  7000  5427
    2864   1 2021-05-01  8421  9046  8458  8180  8152  8011  8119  8070  7984  5275
    2865   1 2021-05-01  9365  9046  8458  8180  8152  8011  8119  8070  7984  6139
    2866   1 2021-05-01  9221  8975  8416  8165  8148  8024  8154  8002  8036  5928
    2867   1 2021-05-01  9737  9151  8535  8249  8212  8063  8182  8090  8039  6492
    2868   1 2021-05-01 10345  9936  9287  8984  8981  8792  8863  8772  8691  7253
    2869   1 2021-05-01 10649 10294  9621  9316  9346  9134  9184  9068  9009  7748
    2870   1 2021-05-01 10732 10349  9710  9425  9459  9261  9311  9212  9136  7951
    2871   1 2021-05-01 10545 10138  9524  9261  9283  9095  9170  9050  9001  7720
    2872   1 2021-05-01 10113  9629  9051  8800  8813  8671  8772  8604  8620  7284
    2873   1 2021-05-01 10037 10058  9387  9065  9070  8873  8940  8890  8764  7186
    2874   1 2021-05-01 10893 11172 10408 10038 10065  9804  9815  9838  9599  8105
    2875   1 2021-05-01 12168 11925 11157 10807 10857 10579 10578 10572 10360  9519
    2876   1 2021-05-01 12502 12085 11337 11018 11077 10808 10794 10770 10584  9897
    2877   1 2021-05-01 12552 11954 11231 10908 10965 10698 10710 10658 10495  9886
    2878   1 2021-05-01 12297 11732 11010 10667 10716 10453 10455 10374 10227  9583
    2879   1 2021-05-01 12284 11623 10930 10612 10673 10416 10428 10304 10219  9711
    2880   1 2021-05-01 12271 11637 10929 10603 10673 10414 10409 10265 10193  9717
    2881   1 2021-05-01 12310 11734 10989 10650 10729 10454 10459 10342 10244  9784
    2882   1 2021-05-01 12564 12073 11327 11005 11074 10789 10789 10714 10560 10114
    2883   1 2021-05-01 12532 12012 11270 10952 11017 10739 10728 10647 10507 10113
    2884   1 2021-05-01 12482 11989 11239 10913 10979 10697 10690 10614 10477 10038
    2885   1 2021-05-01 12462 11878 11138 10818 10885 10601 10603 10514 10376 10027
    2886   1 2021-05-01 12381 11968 11208 10880 10942 10657 10647 10593 10424  9915
    2887   1 2021-05-01 12505 11967 11222 10910 10976 10693 10693 10608 10479 10118
    2888   1 2021-05-01 12350 11761 11044 10737 10803 10531 10555 10442 10335  9942
    2889   1 2021-05-01 12306 11674 10977 10689 10755 10499 10532 10415 10315  9887
    2890   1 2021-05-01 12051 11592 10876 10565 10620 10365 10378 10333 10166  9566
    2891   1 2021-05-01 12044 11583 10883 10587 10652 10400 10420 10347 10205  9562
    2892   1 2021-05-01 11901 11437 10724 10417 10467 10213 10233 10210 10019  9450
    2893   1 2021-05-01 11744 11437 10724 10417 10467 10213 10233 10210 10019  9322
    2894   1 2021-05-01 11915 11525 10809 10488 10550 10279 10287 10280 10066  9580
    2895   1 2021-05-01 11781 11310 10590 10268 10349 10097 10103 10017  9882  9454
    2896   1 2021-05-01 11648 11121 10431 10125 10179  9944  9958  9923  9744  9362
    2897   1 2021-05-01 11490 11038 10361 10083 10150  9925  9955  9878  9760  9184
    2898   1 2021-05-01 11644 11228 10534 10238 10301 10072 10086 10059  9893  9334
    2899   1 2021-05-01 11417 10883 10219  9937  9996  9783  9817  9781  9624  9115
    2900   1 2021-05-01 11117 11105 10404 10098 10147  9917  9937 10003  9729  8773
    2901   1 2021-05-01 11205 11153 10470 10175 10226  9994 10025 10037  9827  8885
    2902   1 2021-05-01 11381 10988 10303  9997 10056  9824  9851  9833  9647  9071
    2903   1 2021-05-01 11420 11248 10535 10218 10289 10036 10047 10036  9830  9124
    2904   1 2021-05-01 10848 10497  9858  9585  9680  9497  9533  9415  9351  8828
    2905   1 2021-05-01 10706 10306  9684  9415  9514  9352  9384  9287  9213  8765
    2906   1 2021-05-01 10498  9806  9216  8957  9052  8915  8985  8905  8829  8617
    2907   1 2021-05-01 10013  9473  8909  8663  8759  8654  8728  8626  8590  8136
    2908   1 2021-05-01  9738  9373  8770  8492  8569  8447  8513  8533  8362  7918
    2909   1 2021-05-01  9794  9613  8991  8704  8780  8650  8704  8730  8538  8005
    2910   1 2021-05-01 10263 10271  9606  9312  9397  9222  9250  9297  9076  8500
    2911   1 2021-05-01 10736 10424  9758  9462  9547  9373  9389  9431  9221  8995
    2912   1 2021-05-01 11175 10781 10155  9908 10037  9867  9882  9852  9708  9728
    2913   1 2021-05-01 11121 10718 10092  9838  9964  9802  9825  9804  9668  9678
    2914   1 2021-05-01 11176 10722 10112  9860  9991  9823  9849  9804  9684  9752
    2915   1 2021-05-01 11042 10594  9980  9727  9863  9692  9732  9701  9567  9663
    2916   1 2021-05-01 11095 10617 10004  9752  9876  9713  9738  9710  9556  9730
    2917   1 2021-05-01 11035 10654 10033  9780  9906  9738  9764  9710  9576  9658
    2918   1 2021-05-01 11051 10526  9919  9671  9795  9632  9659  9615  9488  9683
    2919   1 2021-05-01 11106 10532  9922  9658  9788  9620  9642  9557  9474  9701
    2920   1 2021-05-01 11278 10732 10110  9849  9974  9803  9822  9733  9655  9867
    2921   1 2021-05-01 11269 10743 10121  9876 10013  9829  9842  9746  9675  9908
    2922   1 2021-05-01 11306 10736 10116  9860  9990  9808  9812  9777  9640  9950
    2923   1 2021-05-01 11204 10534  9924  9670  9809  9639  9655  9563  9494  9883
    2924   1 2021-05-01 11088 10521  9918  9661  9791  9623  9651  9574  9499  9710
    2925   1 2021-05-01 10893 10241  9646  9396  9533  9375  9397  9298  9245  9508
    2926   1 2021-05-01 10783 10354  9744  9494  9622  9460  9478  9394  9321  9435
    2927   1 2021-05-01 10781 10104  9513  9257  9380  9230  9259  9201  9105  9441
    2928   1 2021-05-01 10332  9955  9385  9150  9266  9133  9163  9113  9019  9085
    2929   1 2021-05-01 10178  9758  9171  8919  9041  8906  8936  8899  8802  8900
    2930   1 2021-05-01 10045  9364  8848  8633  8777  8658  8710  8569  8579  8742
    2931   1 2021-05-01  9976  9434  8886  8646  8778  8656  8705  8628  8568  8662
    2932   1 2021-05-01  9960  9646  9068  8804  8917  8786  8812  8824  8670  8658
    2933   1 2021-05-01 10227  9840  9247  8978  9088  8949  8969  8999  8824  8921
    2934   1 2021-05-01 10304  9949  9387  9151  9275  9138  9163  9126  9022  9039
    2935   1 2021-05-01 10382  9872  9304  9071  9196  9062  9096  9072  8961  9114
    2936   1 2021-05-01 10246  9774  9202  8967  9093  8970  9023  8958  8884  8994
    2937   1 2021-05-01 10211  9816  9237  8983  9107  8976  9023  8986  8879  8949
    2938   1 2021-05-01 10170  9739  9163  8917  9040  8909  8961  8926  8825  8934
    2939   1 2021-05-01  9470  9181  8645  8416  8538  8448  8520  8471  8393  8222
    2940   1 2021-05-01  7028  7265  6725  6453  6383  6318  6452  6487  6335  4023
    2941   1 2021-05-01  7558  7155  6666  6426  6384  6326  6480  6391  6383  4496
    2942   1 2021-05-01  8551  8552  7959  7669  7623  7485  7597  7607  7463  5387
    2943   1 2021-05-01  9379  9004  8429  8169  8145  8008  8116  8001  7983  6187
    2944   1 2021-05-01  9451  9324  8725  8459  8442  8292  8402  8242  8253  6217
    2945   1 2021-05-01 10277  9838  9184  8889  8889  8707  8789  8660  8629  7236
    2946   1 2021-05-01 10684 10387  9690  9371  9384  9164  9207  9168  9026  7759
    2947   1 2021-05-01 10990 10410  9761  9485  9515  9311  9363  9238  9191  8182
    2948   1 2021-05-01 10761  9947  9329  9056  9080  8906  8981  8846  8813  7964
    2949   1 2021-05-01 10369  9694  9110  8844  8860  8694  8790  8615  8636  7513
    2950   1 2021-05-01 10179 10099  9437  9117  9123  8920  8988  8925  8804  7284
    2951   1 2021-05-01 10799 10058  9387  9065  9070  8873  8940  8890  8764  7936
    2952   1 2021-05-01 11485 11069 10330  9975 10003  9741  9759  9756  9560  8726
    2953   1 2021-05-01 12000 11778 11013 10649 10690 10410 10400 10437 10175  9350
    2954   1 2021-05-01 12376 11924 11174 10830 10874 10591 10582 10595 10356  9737
    2955   1 2021-05-01 12245 11858 11149 10839 10897 10641 10655 10566 10445  9544
    2956   1 2021-05-01 12183 11726 10999 10661 10706 10446 10449 10380 10225  9482
    2957   1 2021-05-01 12359 11954 11223 10890 10956 10682 10683 10590 10452  9719
    2958   1 2021-05-01 12549 12097 11352 11025 11093 10814 10804 10740 10585 10086
    2959   1 2021-05-01 12537 12109 11355 11034 11094 10803 10791 10766 10559 10074
    2960   1 2021-05-01 12539 11984 11231 10908 10960 10672 10662 10645 10424 10067
    2961   1 2021-05-01 12549 11987 11238 10910 10970 10680 10672 10627 10443 10093
    2962   1 2021-05-01 12487 11957 11204 10875 10932 10658 10641 10582 10423 10036
    2963   1 2021-05-01 12444 12011 11250 10916 10970 10688 10674 10617 10446 10039
    2964   1 2021-05-01 12538 11968 11208 10880 10942 10657 10647 10593 10424 10182
    2965   1 2021-05-01 12565 12090 11325 11005 11061 10774 10767 10725 10537 10212
    2966   1 2021-05-01 12615 11994 11262 10961 11037 10762 10767 10673 10545 10263
    2967   1 2021-05-01 12353 11893 11178 10886 10947 10687 10702 10608 10491  9921
    2968   1 2021-05-01 12262 11658 10959 10671 10732 10482 10514 10401 10307  9807
    2969   1 2021-05-01 12118 11670 10972 10679 10750 10499 10519 10401 10313  9676
    2970   1 2021-05-01 12024 11387 10687 10387 10456 10213 10232 10121 10017  9593
    2971   1 2021-05-01 12054 11547 10825 10515 10589 10336 10340 10230 10113  9737
    2972   1 2021-05-01 12087 11583 10847 10540 10622 10352 10352 10272 10125  9792
    2973   1 2021-05-01 12163 11616 10899 10604 10677 10424 10420 10325 10214  9899
    2974   1 2021-05-01 11941 11160 10497 10211 10289 10058 10079  9957  9881  9687
    2975   1 2021-05-01 11470 11070 10402 10115 10185  9959  9982  9863  9774  9203
    2976   1 2021-05-01 11446 10911 10272 10009 10091  9878  9917  9744  9721  9165
    2977   1 2021-05-01 11356 10625  9979  9711  9774  9573  9619  9513  9431  9112
    2978   1 2021-05-01 11046 10625  9979  9711  9774  9573  9619  9513  9431  8752
    2979   1 2021-05-01 11243 10738 10069  9778  9824  9614  9654  9634  9454  8919
    2980   1 2021-05-01 11587 10988 10293  9979 10040  9812  9827  9770  9625  9310
    2981   1 2021-05-01 11638 11129 10422 10099 10165  9925  9936  9896  9724  9383
    2982   1 2021-05-01 10935 10077  9486  9235  9352  9199  9249  9076  9090  9047
    2983   1 2021-05-01 10355  9936  9374  9147  9273  9138  9196  8984  9040  8496
    2984   1 2021-05-01 10409  9306  8771  8556  8675  8576  8663  8483  8524  8554
    2985   1 2021-05-01 10113  9660  9077  8824  8924  8803  8864  8747  8708  8296
    2986   1 2021-05-01 10295  9988  9339  9042  9120  8962  8991  9048  8828  8543
    2987   1 2021-05-01 10632 10179  9528  9238  9324  9161  9190  9218  9019  8914
    2988   1 2021-05-01 10944 10601  9939  9652  9761  9580  9600  9634  9420  9422
    2989   1 2021-05-01 11017 10502  9870  9596  9709  9544  9564  9593  9393  9498
    2990   1 2021-05-01 10738 10529  9921  9663  9781  9623  9654  9670  9488  9289
    2991   1 2021-05-01 10809 10352  9745  9482  9600  9455  9486  9517  9325  9366
    2992   1 2021-05-01 10882 10446  9838  9583  9715  9558  9588  9588  9433  9465
    2993   1 2021-05-01 10914 10482  9871  9610  9730  9567  9591  9603  9421  9519
    2994   1 2021-05-01 10747 10387  9778  9513  9640  9479  9498  9509  9337  9332
    2995   1 2021-05-01 10841 10437  9837  9585  9718  9551  9581  9561  9410  9447
    2996   1 2021-05-01 10806 10437  9837  9585  9718  9551  9581  9561  9410  9464
    2997   1 2021-05-01 10591 10518  9877  9590  9706  9537  9558  9586  9387  9259
    2998   1 2021-05-01 11099 10728 10099  9828  9951  9772  9790  9786  9599  9688
    2999   1 2021-05-01 11151 10749 10117  9846  9980  9812  9834  9793  9670  9786
    3000   1 2021-05-01 11092 10639 10015  9741  9875  9706  9734  9729  9556  9748
    3001   1 2021-05-01 11118 10711 10085  9827  9953  9769  9782  9734  9611  9732
    3002   1 2021-05-01 11185 10673 10056  9795  9928  9746  9768  9708  9604  9796
    3003   1 2021-05-01 10915 10477  9865  9604  9730  9561  9596  9552  9439  9544
    3004   1 2021-05-01 10841 10398  9786  9523  9646  9475  9495  9472  9333  9440
    3005   1 2021-05-01 10798 10185  9609  9369  9501  9348  9375  9285  9223  9448
    3006   1 2021-05-01 10560  9895  9318  9081  9200  9069  9101  9043  8963  9261
    3007   1 2021-05-01 10384  9941  9356  9105  9237  9094  9131  9065  8987  9074
    3008   1 2021-05-01 10371  9688  9128  8891  9013  8887  8929  8874  8793  9045
    3009   1 2021-05-01 10102  9584  9014  8766  8886  8775  8807  8785  8674  8812
    3010   1 2021-05-01 10150  9878  9302  9052  9169  9030  9063  9086  8917  8888
    3011   1 2021-05-01 10048  9675  9113  8877  9002  8890  8927  8925  8796  8768
    3012   1 2021-05-01 10151  9785  9211  8963  9077  8954  8989  8997  8857  8876
    3013   1 2021-05-01  7521  6918  6477  6281  6266  6228  6400  6198  6322  4580
    3014   1 2021-05-01  7588  7895  7399  7185  7173  7103  7250  7052  7158  4616
    3015   1 2021-05-01  8920  8785  8202  7931  7903  7776  7891  7794  7752  5794
    3016   1 2021-05-01  9583  9481  8854  8567  8552  8395  8480  8363  8325  6416
    3017   1 2021-05-01 10104  9924  9275  8975  8981  8797  8867  8744  8692  7038
    3018   1 2021-05-01 10389  9924  9275  8975  8981  8797  8867  8744  8692  7381
    3019   1 2021-05-01 10844 10279  9585  9273  9282  9074  9125  9067  8940  7953
    3020   1 2021-05-01 10737 10376  9703  9413  9427  9214  9268  9204  9084  7857
    3021   1 2021-05-01 10822 10144  9518  9239  9272  9082  9140  8955  8961  8019
    3022   1 2021-05-01 10436 10250  9635  9355  9406  9202  9260  9032  9079  7620
    3023   1 2021-05-01 10184 10222  9567  9245  9270  9063  9124  8998  8946  7321
    3024   1 2021-05-01 11484 10917 10194  9855  9880  9637  9664  9614  9467  8706
    3025   1 2021-05-01 11743 11388 10625 10251 10303 10028 10012  9997  9799  9093
    3026   1 2021-05-01 12147 11784 11036 10686 10749 10468 10469 10418 10254  9553
    3027   1 2021-05-01 12288 11800 11072 10745 10803 10530 10539 10439 10321  9692
    3028   1 2021-05-01 12232 11720 11004 10681 10733 10471 10479 10382 10263  9557
    3029   1 2021-05-01 12276 11915 11177 10859 10922 10645 10641 10553 10422  9620
    3030   1 2021-05-01 12363 12083 11331 11008 11067 10777 10774 10738 10544  9727
    3031   1 2021-05-01 12531 12083 11331 11008 11067 10777 10774 10738 10544  9956
    3032   1 2021-05-01 12306 11997 11257 10937 10997 10717 10705 10651 10481  9820
    3033   1 2021-05-01 12341 11844 11091 10757 10815 10535 10515 10473 10291  9886
    3034   1 2021-05-01 12357 11871 11113 10774 10828 10547 10533 10497 10300  9903
    3035   1 2021-05-01 12432 11967 11203 10874 10931 10652 10640 10584 10407  9998
    3036   1 2021-05-01 12475 12047 11286 10948 11003 10721 10707 10648 10480 10070
    3037   1 2021-05-01 12706 12185 11424 11100 11161 10878 10868 10813 10623 10353
    3038   1 2021-05-01 12699 12228 11483 11168 11236 10952 10937 10902 10706 10335
    3039   1 2021-05-01 12578 11983 11261 10959 11024 10755 10757 10679 10533 10183
    3040   1 2021-05-01 12377 11869 11155 10852 10920 10660 10673 10568 10458  9951
    3041   1 2021-05-01 12264 11802 11079 10765 10816 10552 10562 10527 10351  9775
    3042   1 2021-05-01 12189 11707 10968 10646 10712 10439 10438 10384 10220  9773
    3043   1 2021-05-01 12162 11799 11058 10741 10807 10539 10527 10475 10304  9776
    3044   1 2021-05-01 12260 11919 11195 10892 10968 10702 10695 10616 10476  9942
    3045   1 2021-05-01 12350 11726 10990 10681 10756 10494 10495 10410 10264 10077
    3046   1 2021-05-01 12207 11763 11040 10741 10809 10551 10545 10479 10330  9914
    3047   1 2021-05-01 12048 11481 10781 10485 10553 10310 10311 10200 10103  9800
    3048   1 2021-05-01 11706 11070 10402 10115 10185  9959  9982  9863  9774  9471
    3049   1 2021-05-01 11909 11264 10581 10292 10367 10140 10158 10040  9948  9658
    3050   1 2021-05-01 11640 10865 10228  9963 10042  9832  9878  9704  9694  9379
    3051   1 2021-05-01 11447 11057 10382 10096 10166  9939  9974  9857  9772  9146
    3052   1 2021-05-01 11595 11033 10347 10042 10110  9882  9902  9825  9701  9332
    3053   1 2021-05-01 11653 11359 10660 10362 10447 10211 10220 10102 10021  9393
    3054   1 2021-05-01 11998 11538 10815 10498 10580 10319 10315 10277 10110  9800
    3055   1 2021-05-01 11567 11008 10346 10075 10179  9983  9996  9896  9813  9659
    3056   1 2021-05-01 11218 10516  9891  9641  9750  9582  9621  9482  9454  9347
    3057   1 2021-05-01 10651 10278  9676  9428  9542  9390  9440  9333  9286  8838
    3058   1 2021-05-01 10466  9969  9372  9114  9211  9068  9114  9035  8956  8688
    3059   1 2021-05-01 10178  9459  8883  8639  8734  8633  8699  8655  8570  8393
    3060   1 2021-05-01  9926  9794  9148  8860  8939  8795  8831  8911  8674  8206
    3061   1 2021-05-01 10632 10259  9660  9398  9520  9379  9412  9395  9244  9102
    3062   1 2021-05-01 10655 10204  9597  9329  9448  9311  9349  9319  9176  9125
    3063   1 2021-05-01 10732 10242  9628  9362  9492  9341  9377  9354  9218  9192
    3064   1 2021-05-01 10547 10042  9455  9197  9322  9175  9231  9199  9072  9051
    3065   1 2021-05-01 10539 10280  9674  9405  9522  9376  9407  9437  9239  9051
    3066   1 2021-05-01 10508 10352  9745  9482  9600  9455  9486  9517  9325  9086
    3067   1 2021-05-01 10574 10225  9632  9366  9487  9345  9377  9410  9228  9166
    3068   1 2021-05-01 10610 10218  9617  9346  9468  9318  9336  9346  9169  9188
    3069   1 2021-05-01 10677 10326  9715  9447  9578  9422  9445  9461  9278  9286
    3070   1 2021-05-01 10628 10224  9616  9348  9476  9324  9358  9355  9198  9252
    3071   1 2021-05-01 10690 10426  9801  9514  9628  9460  9480  9537  9290  9295
    3072   1 2021-05-01 10859 10431  9828  9559  9683  9521  9544  9563  9370  9505
    3073   1 2021-05-01 10972 10487  9876  9600  9725  9561  9581  9587  9403  9618
    3074   1 2021-05-01 10838 10476  9859  9581  9705  9535  9548  9563  9377  9448
    3075   1 2021-05-01 11006 10664 10042  9765  9886  9713  9722  9721  9535  9594
    3076   1 2021-05-01 11132 10549  9943  9676  9793  9638  9640  9636  9462  9717
    3077   1 2021-05-01 10911 10329  9723  9461  9587  9435  9456  9402  9287  9532
    3078   1 2021-05-01 10864 10398  9786  9523  9646  9475  9495  9472  9333  9401
    3079   1 2021-05-01 10805 10358  9762  9500  9627  9464  9476  9441  9318  9454
    3080   1 2021-05-01 10702 10128  9552  9319  9446  9300  9323  9240  9173  9380
    3081   1 2021-05-01 10681 10118  9539  9296  9425  9284  9312  9238  9162  9334
    3082   1 2021-05-01 10396  9912  9324  9071  9199  9061  9103  9046  8959  9032
    3083   1 2021-05-01 10107  9800  9240  9003  9126  9008  9047  8979  8899  8775
    3084   1 2021-05-01  9637  9201  8640  8388  8502  8411  8471  8410  8344  8256
    3085   1 2021-05-01  7662  7239  6788  6594  6575  6532  6697  6516  6609  4724
    3086   1 2021-05-01  7401  7279  6774  6547  6502  6439  6573  6509  6470  4437
    3087   1 2021-05-01  7847  7254  6835  6681  6668  6637  6807  6513  6732  4743
    3088   1 2021-05-01  8676  8037  7555  7369  7358  7276  7413  7193  7309  5597
    3089   1 2021-05-01  9328  7701  7300  7164  7194  7140  7322  6910  7255  6301
    3090   1 2021-05-01  8820  8409  7947  7776  7810  7729  7880  7535  7787  5820
    3091   1 2021-05-01  9058  8830  8267  8015  8009  7886  8000  7841  7881  5997
    3092   1 2021-05-01 10167  9590  8974  8703  8706  8542  8633  8484  8489  7249
    3093   1 2021-05-01 10460  9993  9338  9041  9044  8851  8917  8814  8744  7512
    3094   1 2021-05-01 10638 10227  9568  9289  9302  9108  9172  9064  9003  7738
    3095   1 2021-05-01 10707 10265  9594  9310  9326  9120  9180  9082  9012  7832
    3096   1 2021-05-01 10733 10793 10072  9763  9799  9552  9583  9484  9389  7895
    3097   1 2021-05-01 11479 10904 10206  9905  9966  9729  9751  9562  9559  8872
    3098   1 2021-05-01 11239 10706 10035  9712  9766  9538  9575  9400  9385  8568
    3099   1 2021-05-01 11612 11178 10471 10147 10211  9973  9994  9831  9794  8970
    3100   1 2021-05-01 11606 11387 10627 10262 10315 10037 10036  9981  9809  8901
    3101   1 2021-05-01 12182 11387 10627 10262 10315 10037 10036  9981  9809  9541
    3102   1 2021-05-01 12410 11860 11105 10755 10817 10534 10526 10497 10306  9759
    3103   1 2021-05-01 12402 11900 11165 10840 10900 10621 10622 10536 10406  9787
    3104   1 2021-05-01 12243 11747 10999 10654 10711 10424 10427 10367 10201  9654
    3105   1 2021-05-01 12617 12033 11259 10910 10959 10664 10653 10651 10407 10014
    3106   1 2021-05-01 12648 12144 11410 11102 11172 10889 10888 10811 10667 10033
    3107   1 2021-05-01 12630 12128 11388 11078 11152 10856 10844 10780 10612 10107
    3108   1 2021-05-01 12423 11888 11147 10833 10908 10630 10616 10492 10393  9960
    3109   1 2021-05-01 12569 12183 11434 11119 11189 10895 10887 10804 10655 10196
    3110   1 2021-05-01 12686 12121 11368 11058 11130 10850 10842 10757 10616 10277
    3111   1 2021-05-01 12636 12142 11398 11083 11150 10870 10867 10788 10647 10192
    3112   1 2021-05-01 12682 12282 11527 11213 11277 10988 10987 10943 10756 10270
    3113   1 2021-05-01 12757 12187 11444 11125 11198 10914 10904 10832 10676 10369
    3114   1 2021-05-01 12589 12187 11444 11125 11198 10914 10904 10832 10676 10249
    3115   1 2021-05-01 12476 12008 11274 10960 11022 10745 10739 10687 10518 10092
    3116   1 2021-05-01 12523 12086 11372 11077 11146 10872 10879 10801 10653 10062
    3117   1 2021-05-01 12456 11760 11058 10765 10836 10584 10602 10480 10392 10027
    3118   1 2021-05-01 12441 11885 11149 10841 10914 10646 10637 10539 10417 10104
    3119   1 2021-05-01 12403 11891 11152 10825 10906 10619 10614 10563 10388 10088
    3120   1 2021-05-01 12504 12030 11289 10974 11042 10760 10752 10730 10525 10190
    3121   1 2021-05-01 12409 11836 11114 10814 10887 10621 10618 10552 10404 10099
    3122   1 2021-05-01 12303 11837 11093 10771 10849 10568 10549 10488 10327 10086
    3123   1 2021-05-01 12375 11896 11156 10850 10945 10681 10674 10552 10466 10198
    3124   1 2021-05-01 12488 11570 10873 10588 10672 10442 10453 10312 10245 10322
    3125   1 2021-05-01 12111 11531 10850 10568 10641 10406 10424 10357 10218  9893
    3126   1 2021-05-01 11900 11367 10685 10401 10478 10242 10262 10151 10066  9635
    3127   1 2021-05-01 12060 11588 10881 10581 10665 10399 10422 10329 10203  9829
    3128   1 2021-05-01 12029 11260 10576 10285 10372 10124 10147 10024  9945  9808
    3129   1 2021-05-01 12056 11570 10869 10569 10650 10397 10403 10335 10192  9807
    3130   1 2021-05-01 12283 11820 11108 10797 10878 10619 10624 10598 10412 10098
    3131   1 2021-05-01 11586 11022 10372 10106 10214 10028 10053  9951  9870  9722
    3132   1 2021-05-01 11169 10458  9848  9601  9717  9558  9595  9485  9422  9369
    3133   1 2021-05-01 10949 10458  9848  9601  9717  9558  9595  9485  9422  9167
    3134   1 2021-05-01 10513 10024  9432  9180  9299  9155  9204  9117  9063  8764
    3135   1 2021-05-01  9516  9543  9007  8795  8910  8808  8868  8723  8731  7844
    3136   1 2021-05-01 10475 10012  9417  9157  9264  9140  9188  9178  9024  8967
    3137   1 2021-05-01 10523 10104  9509  9245  9371  9233  9277  9244  9113  8999
    3138   1 2021-05-01 10419 10042  9455  9197  9322  9175  9231  9199  9072  8934
    3139   1 2021-05-01 10342  9949  9361  9093  9211  9086  9126  9126  8967  8866
    3140   1 2021-05-01 10420 10047  9465  9201  9323  9195  9223  9231  9068  8989
    3141   1 2021-05-01 10350  9915  9335  9068  9179  9060  9098  9101  8943  8926
    3142   1 2021-05-01 10519 10210  9603  9330  9461  9306  9335  9320  9173  9155
    3143   1 2021-05-01 10777 10293  9665  9377  9495  9335  9357  9371  9183  9353
    3144   1 2021-05-01 10810 10401  9785  9511  9638  9464  9487  9494  9313  9454
    3145   1 2021-05-01 10785 10377  9757  9478  9603  9437  9465  9460  9294  9411
    3146   1 2021-05-01 10816 10421  9795  9516  9641  9468  9491  9476  9324  9398
    3147   1 2021-05-01 10909 10519  9894  9611  9723  9555  9562  9589  9380  9501
    3148   1 2021-05-01 10947 10549  9943  9676  9793  9638  9640  9636  9462  9505
    3149   1 2021-05-01 10847 10389  9789  9526  9650  9497  9498  9467  9343  9397
    3150   1 2021-05-01 10905 10421  9807  9536  9665  9497  9503  9467  9346  9415
    3151   1 2021-05-01 10876 10420  9806  9547  9676  9506  9519  9477  9359  9459
    3152   1 2021-05-01 10845 10286  9694  9446  9570  9413  9439  9392  9290  9416
    3153   1 2021-05-01 10649 10051  9488  9245  9374  9244  9276  9229  9134  9279
    3154   1 2021-05-01 10336  9850  9285  9043  9167  9040  9081  9013  8924  9011
    3155   1 2021-05-01  9750  9420  8827  8562  8668  8558  8605  8565  8468  8354
    3156   1 2021-05-01  7497  6932  6531  6381  6387  6379  6558  6248  6498  4524
    3157   1 2021-05-01  6894  6635  6195  6011  5992  5970  6142  5944  6073  4033
    3158   1 2021-05-01  8150  8163  7585  7323  7282  7164  7260  7237  7132  5133
    3159   1 2021-05-01  9293  9262  8650  8375  8369  8206  8281  8178  8139  6206
    3160   1 2021-05-01 10057  9331  8794  8575  8601  8460  8563  8315  8439  7097
    3161   1 2021-05-01  9882  9606  9022  8773  8797  8637  8735  8543  8591  6953
    3162   1 2021-05-01 10135  9749  9131  8864  8882  8705  8795  8657  8664  7207
    3163   1 2021-05-01 10261  9943  9303  9023  9040  8851  8926  8831  8773  7378
    3164   1 2021-05-01 10416 10240  9577  9289  9302  9095  9168  9080  8992  7506
    3165   1 2021-05-01 10802 10656  9954  9644  9665  9434  9469  9402  9269  7978
    3166   1 2021-05-01 11206 10459  9787  9496  9521  9310  9347  9239  9167  8489
    3167   1 2021-05-01 11573 10793 10072  9763  9799  9552  9583  9484  9389  8939
    3168   1 2021-05-01 11872 11489 10759 10447 10506 10244 10250 10145 10042  9300
    3169   1 2021-05-01 12503 11323 10633 10328 10404 10161 10173  9983  9981  9998
    3170   1 2021-05-01 12290 11686 10972 10667 10738 10485 10490 10344 10290  9740
    3171   1 2021-05-01 12244 11612 10870 10543 10615 10346 10356 10217 10147  9692
    3172   1 2021-05-01 12308 12010 11264 10943 11014 10729 10727 10643 10509  9711
    3173   1 2021-05-01 12487 11951 11211 10887 10943 10667 10657 10593 10441  9907
    3174   1 2021-05-01 12460 11970 11219 10893 10949 10674 10656 10586 10432  9856
    3175   1 2021-05-01 12467 12040 11268 10934 11006 10713 10700 10634 10473  9948
    3176   1 2021-05-01 12670 12216 11475 11151 11218 10929 10913 10872 10687 10201
    3177   1 2021-05-01 12614 12112 11360 11039 11099 10809 10795 10759 10567 10110
    3178   1 2021-05-01 12613 12150 11392 11056 11119 10823 10795 10777 10551 10166
    3179   1 2021-05-01 12703 12131 11385 11068 11143 10846 10833 10751 10598 10312
    3180   1 2021-05-01 12850 12183 11434 11119 11189 10895 10887 10804 10655 10408
    3181   1 2021-05-01 12801 12233 11487 11180 11252 10969 10969 10889 10746 10401
    3182   1 2021-05-01 12564 12166 11416 11105 11169 10883 10869 10849 10640 10082
    3183   1 2021-05-01 12502 12100 11345 11030 11090 10803 10794 10773 10561 10082
    3184   1 2021-05-01 12573 12228 11473 11151 11221 10923 10913 10879 10676 10159
    3185   1 2021-05-01 12648 12112 11365 11047 11110 10825 10816 10749 10594 10323
    3186   1 2021-05-01 12585 12122 11387 11075 11140 10859 10857 10815 10633 10198
    3187   1 2021-05-01 12460 11997 11270 10955 11022 10748 10743 10684 10518 10092
    3188   1 2021-05-01 12531 12064 11331 11024 11097 10821 10818 10736 10587 10165
    3189   1 2021-05-01 12538 12039 11291 10976 11044 10765 10762 10700 10533 10214
    3190   1 2021-05-01 12490 11975 11231 10918 10987 10714 10713 10656 10484 10153
    3191   1 2021-05-01 12347 11854 11105 10778 10849 10568 10562 10512 10332  9988
    3192   1 2021-05-01 12326 11985 11232 10905 10977 10691 10668 10626 10433 10036
    3193   1 2021-05-01 12412 11837 11093 10771 10849 10568 10549 10488 10327 10196
    3194   1 2021-05-01 12642 12140 11390 11084 11166 10890 10878 10823 10664 10452
    3195   1 2021-05-01 12483 11936 11221 10938 11032 10782 10783 10661 10583 10285
    3196   1 2021-05-01 12002 11593 10910 10627 10716 10479 10501 10382 10295  9759
    3197   1 2021-05-01 11917 11562 10855 10541 10613 10363 10374 10330 10163  9634
    3198   1 2021-05-01 12090 11741 11016 10702 10777 10512 10513 10474 10297  9840
    3199   1 2021-05-01 12143 11701 10990 10696 10776 10521 10527 10465 10314  9863
    3200   1 2021-05-01 12219 11780 11066 10758 10827 10565 10561 10543 10336  9983
    3201   1 2021-05-01 12254 11711 10997 10691 10770 10517 10527 10458 10325 10008
    3202   1 2021-05-01 12238 11751 11031 10726 10805 10553 10553 10502 10335 10021
    3203   1 2021-05-01 12232 11788 11065 10748 10834 10573 10572 10535 10350 10052
    3204   1 2021-05-01 12148 11714 10994 10688 10784 10524 10529 10453 10309 10056
    3205   1 2021-05-01 11745 11387 10697 10401 10495 10272 10272 10273 10056  9722
    3206   1 2021-05-01 11455 11129 10469 10194 10295 10097 10105 10051  9908  9583
    3207   1 2021-05-01 11254 10729 10100  9833  9945  9773  9803  9708  9623  9429
    3208   1 2021-05-01 10950 10450  9840  9584  9701  9542  9579  9476  9413  9172
    3209   1 2021-05-01 10541  9942  9361  9123  9242  9108  9159  9035  9007  8771
    3210   1 2021-05-01 10393  9683  9125  8896  9014  8895  8955  8781  8804  8625
    3211   1 2021-05-01 10290  9909  9328  9074  9190  9067  9121  9108  8972  8791
    3212   1 2021-05-01 10056  9824  9228  8962  9067  8949  8996  9013  8850  8588
    3213   1 2021-05-01 10310  9890  9312  9063  9193  9065  9118  9061  8968  8813
    3214   1 2021-05-01 10153  9701  9141  8895  9028  8923  8979  8948  8837  8745
    3215   1 2021-05-01 10043  9860  9268  8998  9119  8997  9030  9054  8879  8644
    3216   1 2021-05-01 10681 10295  9690  9420  9549  9395  9423  9413  9250  9309
    3217   1 2021-05-01 10662 10398  9765  9483  9592  9427  9450  9489  9279  9293
    3218   1 2021-05-01 10869 10457  9829  9550  9661  9499  9513  9508  9345  9441
    3219   1 2021-05-01 10897 10482  9866  9591  9710  9548  9574  9586  9400  9461
    3220   1 2021-05-01 10730 10467  9848  9577  9698  9534  9551  9546  9381  9306
    3221   1 2021-05-01 10886 10543  9899  9622  9731  9553  9554  9596  9389  9424
    3222   1 2021-05-01 11022 10495  9887  9615  9745  9575  9597  9572  9428  9555
    3223   1 2021-05-01 10939 10495  9887  9615  9745  9575  9597  9572  9428  9493
    3224   1 2021-05-01 10790 10358  9747  9491  9615  9454  9471  9458  9307  9368
    3225   1 2021-05-01 10479 10126  9546  9294  9419  9278  9306  9282  9155  9099
    3226   1 2021-05-01  9917  9501  8921  8669  8775  8674  8728  8665  8599  8443
    3227   1 2021-05-01 10059  9608  9009  8753  8862  8739  8782  8716  8631  8578
    3228   1 2021-05-01  9069  8702  8106  7838  7838  7698  7788  7655  7664  6065
    3229   1 2021-05-01  9076  8662  8091  7852  7841  7722  7821  7701  7712  6036
    3230   1 2021-05-01  8264  7964  7439  7208  7179  7087  7213  7092  7114  5159
    3231   1 2021-05-01  8289  6995  6616  6491  6504  6488  6678  6259  6631  5142
    3232   1 2021-05-01  7488  7695  7184  6976  6952  6877  7021  6830  6942  4473
    3233   1 2021-05-01  8763  9214  8550  8235  8219  8048  8122  8106  7974  5748
    3234   1 2021-05-01  9936  9214  8550  8235  8219  8048  8122  8106  7974  7001
    3235   1 2021-05-01 10147  9655  9037  8766  8773  8599  8674  8556  8533  7221
    3236   1 2021-05-01 10260  9794  9175  8910  8933  8752  8838  8667  8680  7359
    3237   1 2021-05-01 10203  9739  9128  8863  8870  8694  8786  8652  8641  7334
    3238   1 2021-05-01 10237  9784  9169  8898  8915  8728  8817  8675  8659  7331
    3239   1 2021-05-01 10558 10203  9537  9244  9252  9055  9120  9040  8942  7646
    3240   1 2021-05-01 10903 10656  9954  9644  9665  9434  9469  9402  9269  8046
    3241   1 2021-05-01 11779 11250 10507 10178 10218  9956  9961  9919  9758  9167
    3242   1 2021-05-01 12160 11740 10977 10649 10696 10420 10410 10391 10190  9631
    3243   1 2021-05-01 12517 12162 11411 11088 11153 10869 10853 10830 10620 10010
    3244   1 2021-05-01 12680 12042 11303 10997 11065 10787 10788 10698 10576 10147
    3245   1 2021-05-01 12540 12062 11296 10962 11005 10714 10701 10713 10475  9968
    3246   1 2021-05-01 12726 12200 11462 11146 11224 10934 10926 10838 10706 10201
    3247   1 2021-05-01 12676 12155 11387 11057 11120 10822 10799 10764 10553 10164
    3248   1 2021-05-01 12599 12033 11282 10960 11028 10746 10730 10655 10507 10083
    3249   1 2021-05-01 12667 12065 11306 10977 11054 10767 10755 10676 10539 10159
    3250   1 2021-05-01 12750 12181 11428 11114 11180 10898 10881 10804 10669 10286
    3251   1 2021-05-01 12717 12225 11481 11157 11220 10946 10929 10863 10700 10284
    3252   1 2021-05-01 12687 12162 11422 11113 11173 10895 10884 10812 10651 10203
    3253   1 2021-05-01 12596 12150 11392 11056 11119 10823 10795 10777 10551 10155
    3254   1 2021-05-01 12577 12194 11432 11094 11152 10843 10830 10828 10585 10146
    3255   1 2021-05-01 12652 12263 11518 11207 11273 10974 10968 10932 10731 10195
    3256   1 2021-05-01 12612 12130 11371 11040 11103 10809 10789 10762 10554 10199
    3257   1 2021-05-01 12575 12091 11349 11034 11102 10822 10814 10731 10598 10160
    3258   1 2021-05-01 12528 11994 11232 10894 10954 10670 10656 10602 10437 10187
    3259   1 2021-05-01 12541 12126 11366 11036 11097 10806 10787 10776 10557 10150
    3260   1 2021-05-01 12619 12121 11382 11064 11126 10846 10831 10811 10604 10260
    3261   1 2021-05-01 12591 12188 11448 11141 11208 10919 10918 10871 10690 10167
    3262   1 2021-05-01 12658 12090 11366 11063 11139 10863 10866 10780 10647 10354
    3263   1 2021-05-01 12682 12134 11394 11073 11145 10854 10845 10799 10617 10378
    3264   1 2021-05-01 12516 12110 11373 11065 11143 10857 10859 10770 10636 10219
    3265   1 2021-05-01 12466 11960 11213 10893 10954 10674 10657 10609 10442 10154
    3266   1 2021-05-01 12450 11854 11105 10778 10849 10568 10562 10512 10332 10105
    3267   1 2021-05-01 12572 12094 11325 10996 11063 10770 10757 10755 10523 10248
    3268   1 2021-05-01 12557 12211 11469 11162 11239 10957 10949 10934 10721 10234
    3269   1 2021-05-01 12533 11993 11267 10966 11034 10767 10760 10751 10530 10273
    3270   1 2021-05-01 12424 11748 11041 10746 10825 10572 10578 10518 10357 10189
    3271   1 2021-05-01 11878 11449 10761 10468 10552 10311 10324 10224 10121  9571
    3272   1 2021-05-01 11752 11572 10844 10524 10579 10324 10325 10340 10110  9401
    3273   1 2021-05-01 12010 11584 10863 10543 10592 10347 10336 10318 10131  9675
    3274   1 2021-05-01 12101 11697 10964 10632 10697 10435 10420 10414 10194  9830
    3275   1 2021-05-01 12316 11697 10964 10632 10697 10435 10420 10414 10194 10103
    3276   1 2021-05-01 12299 11827 11128 10838 10921 10668 10668 10593 10460 10052
    3277   1 2021-05-01 12129 11685 10991 10699 10787 10535 10546 10472 10339  9848
    3278   1 2021-05-01 12135 11681 10978 10667 10737 10486 10487 10488 10266  9879
    3279   1 2021-05-01 12109 11766 11053 10750 10830 10577 10576 10554 10371  9860
    3280   1 2021-05-01 12121 11701 10994 10692 10773 10524 10524 10483 10317  9917
    3281   1 2021-05-01 12177 11745 11018 10708 10785 10531 10525 10490 10313 10014
    3282   1 2021-05-01 12182 11650 10938 10635 10732 10491 10490 10416 10292 10041
    3283   1 2021-05-01 11935 11435 10743 10434 10535 10307 10309 10240 10113  9814
    3284   1 2021-05-01 11689 11245 10571 10300 10402 10196 10219 10118 10041  9616
    3285   1 2021-05-01 11586 11099 10412 10117 10210  9993 10003  9958  9808  9540
    3286   1 2021-05-01 11536 10932 10272  9989 10091  9892  9909  9850  9730  9557
    3287   1 2021-05-01 11305 10543  9917  9660  9771  9609  9640  9579  9472  9447
    3288   1 2021-05-01 10726 10162  9552  9293  9401  9252  9293  9228  9135  8963
    3289   1 2021-05-01 10503 10174  9561  9308  9413  9267  9316  9237  9156  8740
    3290   1 2021-05-01 10017  9813  9242  9004  9131  9018  9078  9036  8939  8555
    3291   1 2021-05-01  9905  9610  9047  8795  8923  8816  8876  8870  8743  8445
    3292   1 2021-05-01  9739  9445  8883  8623  8741  8646  8717  8719  8565  8315
    3293   1 2021-05-01 10175  9906  9313  9051  9171  9045  9076  9089  8935  8713
    3294   1 2021-05-01 10135  9702  9147  8897  9025  8925  8965  8956  8823  8706
    3295   1 2021-05-01  9900  9690  9118  8856  8974  8866  8918  8914  8773  8533
    3296   1 2021-05-01 10731 10418  9784  9499  9607  9443  9459  9498  9288  9315
    3297   1 2021-05-01 10622 10314  9699  9418  9539  9378  9403  9428  9221  9236
    3298   1 2021-05-01 10414 10283  9671  9384  9499  9356  9394  9448  9228  9045
    3299   1 2021-05-01 10666 10226  9590  9292  9393  9242  9264  9355  9107  9196
    3300   1 2021-05-01 10786 10447  9826  9551  9668  9505  9523  9559  9359  9373
    3301   1 2021-05-01 10757 10348  9739  9474  9601  9452  9473  9464  9304  9347
    3302   1 2021-05-01 10715 10210  9621  9361  9494  9350  9366  9334  9218  9290
    3303   1 2021-05-01 10570 10196  9605  9345  9475  9324  9348  9317  9198  9169
    3304   1 2021-05-01 10479  9997  9422  9169  9293  9162  9194  9159  9045  9076
    3305   1 2021-05-01 10017  9695  9096  8838  8945  8834  8873  8821  8727  8559
    3306   1 2021-05-01 10179  9730  9128  8875  8982  8874  8919  8868  8768  8701
    3307   1 2021-05-01 10181  9706  9102  8842  8953  8833  8875  8797  8726  8712
    3308   1 2021-05-01 10151  9706  9093  8829  8930  8815  8870  8800  8735  8698
    3309   1 2021-05-01  8642  7867  7392  7212  7191  7126  7272  7061  7179  5517
    3310   1 2021-05-01  8267  7867  7392  7212  7191  7126  7272  7061  7179  5111
    3311   1 2021-05-01  7774  7745  7255  7056  7028  6964  7103  6944  7012  4618
    3312   1 2021-05-01  7918  7566  7071  6854  6814  6741  6886  6751  6790  4784
    3313   1 2021-05-01  8132  7674  7201  7007  6991  6924  7064  6836  6979  4972
    3314   1 2021-05-01  8067  7884  7329  7076  7035  6932  7062  6980  6957  4912
    3315   1 2021-05-01  9628  9043  8430  8157  8145  7994  8093  8022  7965  6685
    3316   1 2021-05-01 10125  9600  8957  8668  8663  8489  8568  8502  8415  7252
    3317   1 2021-05-01 10423 10056  9397  9110  9129  8931  8994  8874  8828  7595
    3318   1 2021-05-01 10753 10470  9793  9504  9543  9329  9368  9217  9196  8012
    3319   1 2021-05-01 10615 10178  9550  9280  9329  9126  9185  8938  9016  7850
    3320   1 2021-05-01 10557 10114  9471  9175  9195  9004  9062  8949  8891  7744
    3321   1 2021-05-01 11004 10733 10028  9713  9738  9507  9541  9460  9356  8213
    3322   1 2021-05-01 11333 11365 10615 10289 10329 10060 10071 10031  9857  8606
    3323   1 2021-05-01 11875 11916 11160 10838 10894 10616 10618 10594 10412  9248
    3324   1 2021-05-01 12564 11823 11064 10738 10798 10519 10520 10470 10310 10039
    3325   1 2021-05-01 12716 12213 11461 11136 11198 10911 10903 10881 10689 10174
    3326   1 2021-05-01 12536 12116 11381 11076 11144 10859 10861 10804 10645  9943
    3327   1 2021-05-01 12349 11774 11018 10686 10741 10472 10466 10390 10236  9732
    3328   1 2021-05-01 12404 12083 11319 10973 11016 10716 10701 10734 10454  9837
    3329   1 2021-05-01 12529 12054 11286 10940 10999 10702 10690 10668 10446  9988
    3330   1 2021-05-01 12726 12195 11423 11101 11169 10868 10859 10809 10613 10246
    3331   1 2021-05-01 12643 12007 11251 10918 10977 10687 10682 10644 10444 10142
    3332   1 2021-05-01 12580 12150 11405 11082 11136 10849 10837 10814 10602 10014
    3333   1 2021-05-01 12530 12101 11344 11008 11064 10772 10750 10731 10520 10100
    3334   1 2021-05-01 12562 12081 11322 10995 11062 10767 10751 10705 10524 10124
    3335   1 2021-05-01 12597 12080 11329 11007 11076 10769 10763 10702 10529 10191
    3336   1 2021-05-01 12557 12072 11312 10988 11063 10774 10756 10662 10515 10151
    3337   1 2021-05-01 12527 12079 11312 10969 11030 10722 10707 10703 10460 10192
    3338   1 2021-05-01 12566 12070 11293 10950 11012 10710 10678 10676 10434 10205
    3339   1 2021-05-01 12622 12215 11459 11132 11199 10896 10871 10859 10637 10277
    3340   1 2021-05-01 12623 12073 11328 11016 11093 10808 10793 10705 10579 10274
    3341   1 2021-05-01 12578 12071 11307 10978 11041 10750 10740 10721 10516 10184
    3342   1 2021-05-01 12598 12144 11405 11098 11169 10898 10889 10845 10666 10199
    3343   1 2021-05-01 12501 12126 11388 11070 11132 10852 10833 10825 10599 10122
    3344   1 2021-05-01 12729 12379 11627 11296 11369 11068 11047 11043 10797 10397
    3345   1 2021-05-01 12737 12274 11533 11218 11298 11011 11003 10946 10783 10456
    3346   1 2021-05-01 12635 12022 11284 10968 11042 10770 10755 10669 10540 10324
    3347   1 2021-05-01 12521 12035 11284 10948 11017 10736 10726 10666 10500 10217
    3348   1 2021-05-01 12512 12016 11269 10943 11003 10724 10713 10709 10488 10179
    3349   1 2021-05-01 12431 11834 11089 10755 10824 10547 10537 10511 10312 10110
    3350   1 2021-05-01 12287 11847 11113 10798 10856 10585 10565 10561 10329 10006
    3351   1 2021-05-01 12242 11847 11113 10798 10856 10585 10565 10561 10329 10001
    3352   1 2021-05-01 12135 11704 10984 10671 10740 10490 10479 10442 10254  9933
    3353   1 2021-05-01 12185 11691 10983 10681 10758 10504 10498 10422 10287  9943
    3354   1 2021-05-01 12164 11446 10748 10448 10529 10281 10297 10187 10104  9900
    3355   1 2021-05-01 12036 11657 10960 10671 10750 10501 10499 10360 10303  9729
    3356   1 2021-05-01 12196 11809 11067 10746 10806 10551 10530 10463 10307  9933
    3357   1 2021-05-01 12438 12053 11317 11006 11085 10808 10782 10748 10559 10228
    3358   1 2021-05-01 12275 11786 11080 10779 10869 10605 10607 10531 10394 10052
    3359   1 2021-05-01 12027 11525 10832 10532 10615 10367 10377 10296 10167  9749
    3360   1 2021-05-01 12125 11610 10917 10620 10704 10458 10457 10374 10251  9868
    3361   1 2021-05-01 12038 11621 10909 10603 10683 10435 10435 10411 10224  9760
    3362   1 2021-05-01 12024 11577 10860 10534 10615 10373 10367 10341 10156  9824
    3363   1 2021-05-01 12136 11648 10931 10626 10714 10467 10471 10438 10257  9976
    3364   1 2021-05-01 12041 11648 10931 10626 10714 10467 10471 10438 10257  9880
    3365   1 2021-05-01 11934 11499 10800 10502 10610 10382 10387 10313 10199  9804
    3366   1 2021-05-01 11772 11296 10619 10333 10423 10208 10219 10203 10034  9667
    3367   1 2021-05-01 11623 11147 10456 10171 10274 10067 10085  9985  9905  9611
    3368   1 2021-05-01 11472 11035 10364 10082 10186  9984 10005  9922  9820  9543
    3369   1 2021-05-01 11123 10604  9975  9705  9807  9641  9668  9605  9500  9278
    3370   1 2021-05-01 10756 10229  9617  9360  9462  9322  9358  9270  9203  8962
    3371   1 2021-05-01 10186  9829  9232  8975  9075  8944  8984  8941  8833  8449
    3372   1 2021-05-01 10008  9684  9118  8862  8974  8873  8924  8922  8770  8510
    3373   1 2021-05-01  9965  9617  9044  8782  8899  8798  8853  8827  8700  8478
    3374   1 2021-05-01 10013  9482  8943  8706  8834  8736  8808  8691  8646  8531
    3375   1 2021-05-01 10056  9642  9081  8829  8956  8835  8898  8811  8748  8589
    3376   1 2021-05-01  9999  9607  9042  8778  8901  8797  8829  8835  8692  8558
    3377   1 2021-05-01  9933  9421  8881  8631  8758  8667  8714  8669  8572  8518
    3378   1 2021-05-01  9801  9306  8749  8489  8610  8522  8585  8604  8428  8412
    3379   1 2021-05-01 10644 10144  9551  9294  9419  9273  9307  9239  9133  9209
    3380   1 2021-05-01 10219  9761  9180  8912  9027  8910  8964  8997  8813  8850
    3381   1 2021-05-01 10285  9991  9367  9069  9170  9032  9067  9158  8897  8845
    3382   1 2021-05-01 10523 10364  9747  9475  9587  9437  9450  9453  9280  9062
    3383   1 2021-05-01 10794 10362  9751  9480  9598  9444  9456  9462  9296  9367
    3384   1 2021-05-01 10754 10326  9717  9455  9579  9434  9451  9426  9291  9316
    3385   1 2021-05-01 10705 10262  9661  9398  9528  9378  9406  9364  9234  9271
    3386   1 2021-05-01 10555 10061  9484  9228  9355  9216  9252  9205  9097  9144
    3387   1 2021-05-01 10416  9930  9365  9113  9231  9104  9140  9089  8995  8979
    3388   1 2021-05-01 10164  9751  9189  8930  9052  8939  8980  8937  8835  8746
    3389   1 2021-05-01  9925  9579  8999  8742  8850  8752  8803  8810  8655  8522
    3390   1 2021-05-01  9957  9631  9043  8789  8895  8789  8835  8826  8697  8529
    3391   1 2021-05-01  9749  9434  8848  8583  8680  8581  8636  8673  8495  8334
    3392   1 2021-05-01  9748  9622  9024  8770  8871  8765  8821  8791  8677  8295
    3393   1 2021-05-01  9842  9396  8817  8567  8663  8576  8630  8601  8496  8365
    3394   1 2021-05-01  9770  9181  8633  8387  8490  8411  8489  8462  8363  8284
    3395   1 2021-05-01  9370  9055  8508  8265  8361  8292  8374  8354  8253  7863
    3396   1 2021-05-01  7558  7163  6706  6518  6492  6445  6601  6414  6509  4441
    3397   1 2021-05-01  7430  7150  6674  6461  6423  6366  6509  6369  6418  4302
    3398   1 2021-05-01  7732  7192  6693  6460  6414  6346  6484  6360  6386  4509
    3399   1 2021-05-01  7658  7521  6992  6747  6704  6620  6760  6639  6663  4486
    3400   1 2021-05-01  8312  8624  7995  7698  7659  7515  7595  7598  7461  5191
    3401   1 2021-05-01  9494  9556  8941  8664  8677  8505  8581  8457  8445  6519
    3402   1 2021-05-01  9938 10249  9581  9289  9317  9119  9165  9067  9018  7069
    3403   1 2021-05-01 11010 10138  9464  9169  9182  8982  9037  8963  8873  8313
    3404   1 2021-05-01 11339 10470  9793  9504  9543  9329  9368  9217  9196  8759
    3405   1 2021-05-01 11389 10868 10174  9871  9930  9694  9725  9560  9546  8842
    3406   1 2021-05-01 11638 10500  9848  9558  9618  9406  9452  9240  9274  9115
    3407   1 2021-05-01 11740 11031 10308  9980 10027  9783  9795  9700  9595  9151
    3408   1 2021-05-01 12050 11481 10740 10416 10457 10185 10196 10162  9984  9433
    3409   1 2021-05-01 12328 11946 11205 10900 10955 10680 10689 10648 10474  9745
    3410   1 2021-05-01 12461 12129 11373 11055 11109 10826 10822 10827 10608  9902
    3411   1 2021-05-01 12415 11948 11202 10874 10927 10649 10645 10617 10430  9829
    3412   1 2021-05-01 12393 11877 11135 10803 10865 10581 10574 10491 10346  9856
    3413   1 2021-05-01 12619 12200 11445 11122 11205 10918 10900 10807 10679 10175
    3414   1 2021-05-01 12650 12127 11374 11057 11128 10842 10838 10755 10611 10214
    3415   1 2021-05-01 12779 12361 11606 11293 11362 11072 11064 10991 10832 10292
    3416   1 2021-05-01 12625 12047 11308 10986 11063 10774 10767 10649 10537 10183
    3417   1 2021-05-01 12552 12007 11251 10918 10977 10687 10682 10644 10444 10147
    3418   1 2021-05-01 12444 11949 11206 10882 10948 10659 10655 10587 10435  9981
    3419   1 2021-05-01 12546 12050 11299 10969 11036 10743 10734 10664 10498 10112
    3420   1 2021-05-01 12756 12165 11424 11105 11195 10902 10885 10799 10662 10352
    3421   1 2021-05-01 12733 12161 11411 11098 11172 10875 10863 10794 10630 10333
    3422   1 2021-05-01 12741 12261 11505 11182 11259 10968 10947 10877 10707 10378
    3423   1 2021-05-01 12663 12142 11384 11064 11146 10859 10841 10743 10607 10365
    3424   1 2021-05-01 12677 12144 11398 11073 11153 10864 10846 10769 10619 10379
    3425   1 2021-05-01 12544 12086 11336 11006 11079 10781 10763 10742 10525 10222
    3426   1 2021-05-01 12553 11913 11171 10843 10907 10622 10616 10574 10380 10179
    3427   1 2021-05-01 12516 12102 11353 11026 11091 10802 10793 10815 10563 10110
    3428   1 2021-05-01 12525 12094 11353 11035 11111 10823 10817 10803 10602 10150
    3429   1 2021-05-01 12537 12273 11504 11152 11220 10912 10894 10931 10655 10234
    3430   1 2021-05-01 12865 12379 11627 11296 11369 11068 11047 11043 10797 10608
    3431   1 2021-05-01 12775 12331 11598 11290 11376 11092 11082 11018 10854 10499
    3432   1 2021-05-01 12621 12131 11392 11080 11151 10877 10856 10807 10637 10343
    3433   1 2021-05-01 12583 12080 11331 11013 11084 10804 10791 10720 10557 10306
    3434   1 2021-05-01 12443 11986 11236 10906 10974 10695 10680 10637 10450 10145
    3435   1 2021-05-01 12454 11974 11233 10915 10985 10724 10715 10630 10488 10143
    3436   1 2021-05-01 12421 11891 11149 10828 10905 10628 10611 10534 10377 10163
    3437   1 2021-05-01 12418 11994 11266 10967 11048 10774 10765 10679 10550 10201
    3438   1 2021-05-01 12355 11858 11137 10841 10915 10648 10645 10577 10441 10133
    3439   1 2021-05-01 12334 11962 11234 10931 11011 10741 10729 10669 10520 10074
    3440   1 2021-05-01 12373 12039 11305 10997 11079 10803 10790 10724 10570 10119
    3441   1 2021-05-01 12584 12347 11627 11335 11432 11156 11144 11067 10920 10329
    3442   1 2021-05-01 12685 12115 11365 11049 11132 10864 10847 10786 10638 10431
    3443   1 2021-05-01 12575 12036 11318 11027 11124 10855 10849 10742 10639 10350
    3444   1 2021-05-01 12502 12036 11318 11027 11124 10855 10849 10742 10639 10296
    3445   1 2021-05-01 12372 11839 11132 10832 10923 10657 10656 10567 10449 10159
    3446   1 2021-05-01 12191 11628 10929 10638 10725 10485 10483 10385 10283  9991
    3447   1 2021-05-01 12030 11633 10930 10632 10725 10476 10483 10381 10275  9808
    3448   1 2021-05-01 12023 11527 10814 10506 10585 10342 10337 10311 10142  9822
    3449   1 2021-05-01 11721 11302 10619 10337 10424 10197 10216 10156 10027  9517
    3450   1 2021-05-01 11593 11161 10483 10191 10282 10063 10083 10042  9900  9453
    3451   1 2021-05-01 11484 11054 10385 10099 10195  9988 10004  9974  9823  9367
    3452   1 2021-05-01 11500 11088 10416 10126 10209 10012 10023 10027  9842  9465
    3453   1 2021-05-01 11242 10750 10113  9842  9936  9762  9784  9751  9605  9342
    3454   1 2021-05-01 10891 10341  9719  9458  9558  9408  9443  9359  9278  9063
    3455   1 2021-05-01 10670  9908  9310  9058  9159  9026  9067  8966  8914  8864
    3456   1 2021-05-01 10365  9730  9152  8893  9013  8896  8937  8895  8784  8823
    3457   1 2021-05-01 10274  9730  9152  8893  9013  8896  8937  8895  8784  8781
    3458   1 2021-05-01 10208  9775  9194  8937  9057  8950  8998  8945  8835  8764
    3459   1 2021-05-01 10223  9799  9228  8976  9100  8987  9042  8991  8884  8787
    3460   1 2021-05-01 10033  9608  9048  8795  8927  8818  8862  8798  8723  8648
    3461   1 2021-05-01  9716  9218  8689  8455  8586  8513  8576  8495  8434  8328
    3462   1 2021-05-01  9682  9329  8778  8523  8649  8558  8611  8562  8458  8298
    3463   1 2021-05-01  9472  9045  8531  8299  8440  8373  8452  8388  8329  8135
    3464   1 2021-05-01 10716 10287  9680  9411  9538  9393  9413  9406  9247  9282
    3465   1 2021-05-01 10200  9793  9251  9022  9159  9049  9107  8988  8961  8774
    3466   1 2021-05-01 10256  9754  9188  8933  9059  8941  8997  8935  8845  8778
    3467   1 2021-05-01 10486 10260  9634  9344  9454  9303  9321  9383  9155  9029
    3468   1 2021-05-01 10723 10207  9590  9305  9412  9265  9282  9328  9129  9278
    3469   1 2021-05-01 10716 10294  9679  9411  9533  9372  9390  9402  9226  9259
    3470   1 2021-05-01 10596 10160  9575  9316  9436  9291  9322  9292  9154  9163
    3471   1 2021-05-01 10521 10073  9478  9217  9335  9200  9234  9213  9065  9077
    3472   1 2021-05-01 10482  9851  9275  9032  9156  9036  9072  9009  8927  9046
    3473   1 2021-05-01 10134  9628  9064  8809  8933  8827  8875  8834  8729  8683
    3474   1 2021-05-01  9912  9548  8986  8735  8857  8759  8814  8796  8669  8500
    3475   1 2021-05-01  9889  9558  8977  8710  8827  8724  8768  8754  8619  8488
    3476   1 2021-05-01  9495  9118  8555  8302  8403  8323  8391  8413  8254  8134
    3477   1 2021-05-01  9491  9178  8629  8383  8498  8408  8474  8433  8340  8070
    3478   1 2021-05-01  9590  9402  8829  8572  8681  8573  8624  8561  8490  8112
    3479   1 2021-05-01  9721  9414  8848  8618  8734  8641  8699  8609  8568  8204
    3480   1 2021-05-01  9828  9414  8848  8618  8734  8641  8699  8609  8568  8305
    3481   1 2021-05-01  9843  9181  8657  8438  8545  8475  8549  8439  8416  8282
    3482   1 2021-05-01  9381  9038  8532  8319  8425  8370  8461  8348  8342  7833
    3483   1 2021-05-01  9546  9260  8692  8449  8543  8463  8528  8496  8400  7961
    3484   1 2021-05-01 10022  9447  8886  8652  8757  8665  8726  8634  8595  8370
    3485   1 2021-05-01  6834  6594  6137  5934  5893  5864  6024  5847  5954  3844
    3486   1 2021-05-01  7493  7361  6852  6618  6580  6498  6637  6493  6535  4359
    3487   1 2021-05-01  9288  8507  7895  7608  7581  7430  7520  7475  7388  6304
    3488   1 2021-05-01 10177  9692  9034  8727  8724  8535  8586  8569  8440  7295
    3489   1 2021-05-01 10685 10249  9581  9289  9317  9119  9165  9067  9018  7911
    3490   1 2021-05-01 11064 10856 10155  9850  9889  9659  9683  9626  9511  8347
    3491   1 2021-05-01 11723 11167 10459 10141 10193  9952  9969  9883  9773  9197
    3492   1 2021-05-01 11883 11424 10691 10366 10423 10166 10173 10108  9970  9412
    3493   1 2021-05-01 12027 11419 10686 10367 10436 10179 10185 10063  9982  9535
    3494   1 2021-05-01 12010 11726 10980 10646 10705 10426 10427 10377 10215  9475
    3495   1 2021-05-01 12346 11943 11202 10906 10955 10682 10691 10663 10482  9701
    3496   1 2021-05-01 12370 11941 11197 10886 10936 10655 10652 10631 10445  9784
    3497   1 2021-05-01 12427 11947 11209 10898 10952 10688 10683 10613 10468  9841
    3498   1 2021-05-01 12398 11947 11209 10898 10952 10688 10683 10613 10468  9845
    3499   1 2021-05-01 12705 11953 11183 10833 10889 10597 10588 10543 10361 10256
    3500   1 2021-05-01 12994 12406 11632 11294 11359 11053 11036 11030 10799 10577
    3501   1 2021-05-01 12882 12341 11576 11259 11346 11060 11041 10970 10833 10498
    3502   1 2021-05-01 12871 12361 11606 11293 11362 11072 11064 10991 10832 10400
    3503   1 2021-05-01 12864 12281 11529 11215 11288 11000 10987 10907 10755 10414
    3504   1 2021-05-01 12887 12191 11450 11152 11241 10953 10950 10821 10730 10444
    3505   1 2021-05-01 12794 12283 11554 11262 11354 11067 11063 10946 10851 10417
    3506   1 2021-05-01 12724 12247 11508 11191 11276 10978 10971 10910 10743 10325
    3507   1 2021-05-01 12754 12241 11495 11171 11256 10949 10942 10892 10700 10360
    3508   1 2021-05-01 12786 12268 11508 11176 11255 10952 10936 10893 10699 10436
    3509   1 2021-05-01 12762 12240 11475 11140 11214 10924 10907 10848 10675 10447
    3510   1 2021-05-01 12725 12245 11474 11140 11208 10906 10886 10859 10642 10443
    3511   1 2021-05-01 12692 12245 11474 11140 11208 10906 10886 10859 10642 10416
    3512   1 2021-05-01 12448 12042 11303 10981 11062 10766 10752 10691 10512 10118
    3513   1 2021-05-01 12334 11865 11135 10820 10889 10612 10600 10520 10369  9978
    3514   1 2021-05-01 12351 11948 11210 10882 10940 10658 10661 10642 10425  9958
    3515   1 2021-05-01 12477 12094 11353 11035 11111 10823 10817 10803 10602 10071
    3516   1 2021-05-01 12539 12121 11361 11028 11095 10800 10798 10814 10586 10230
    3517   1 2021-05-01 12277 12217 11479 11154 11216 10925 10914 10936 10678  9939
    3518   1 2021-05-01 12378 11922 11163 10827 10896 10602 10591 10562 10361 10087
    3519   1 2021-05-01 12457 12055 11291 10963 11026 10741 10710 10681 10480 10208
    3520   1 2021-05-01 12583 12036 11283 10968 11034 10753 10735 10666 10496 10316
    3521   1 2021-05-01 12460 12058 11302 10978 11033 10754 10739 10726 10498 10175
    3522   1 2021-05-01 12525 12143 11391 11059 11112 10816 10790 10843 10554 10195
    3523   1 2021-05-01 12594 11954 11218 10913 10979 10705 10699 10642 10466 10368
    3524   1 2021-05-01 12342 11927 11198 10896 10964 10690 10694 10639 10469 10068
    3525   1 2021-05-01 12530 12033 11282 10961 11026 10752 10738 10735 10516 10299
    3526   1 2021-05-01 12599 12141 11383 11060 11136 10853 10829 10809 10605 10388
    3527   1 2021-05-01 12780 12347 11627 11335 11432 11156 11144 11067 10920 10552
    3528   1 2021-05-01 12681 12294 11577 11290 11374 11104 11099 11015 10885 10384
    3529   1 2021-05-01 12658 12204 11452 11132 11208 10928 10896 10892 10672 10418
    3530   1 2021-05-01 12732 12244 11508 11210 11294 11027 11005 10949 10787 10492
    3531   1 2021-05-01 12200 11660 10958 10656 10746 10496 10502 10420 10298 10055
    3532   1 2021-05-01 11990 11290 10613 10322 10418 10191 10204 10098 10011  9812
    3533   1 2021-05-01 11709 11145 10465 10175 10266 10039 10051  9952  9861  9526
    3534   1 2021-05-01 11501 11116 10441 10158 10240 10017 10043 10001  9840  9373
    3535   1 2021-05-01 11480 11098 10426 10146 10231 10019 10039  9998  9845  9417
    3536   1 2021-05-01 11122 10805 10154  9884  9974  9784  9815  9786  9649  9111
    3537   1 2021-05-01 11065 10750 10113  9842  9936  9762  9784  9751  9605  9138
    3538   1 2021-05-01 10839 10451  9821  9552  9666  9502  9532  9448  9369  8999
    3539   1 2021-05-01  9178  9080  8526  8282  8389  8286  8355  8254  8222  7473
    3540   1 2021-05-01 10359  9956  9357  9082  9188  9068  9099  9115  8940  8876
    3541   1 2021-05-01 10204  9754  9185  8936  9059  8946  9004  8960  8852  8791
    3542   1 2021-05-01 10036  9521  8966  8712  8837  8741  8798  8753  8652  8641
    3543   1 2021-05-01  9895  9424  8858  8590  8715  8621  8667  8679  8526  8511
    3544   1 2021-05-01 10493 10187  9584  9313  9426  9296  9316  9352  9157  9108
    3545   1 2021-05-01 10497  9977  9411  9156  9272  9163  9200  9212  9037  9121
    3546   1 2021-05-01 10354 10076  9478  9211  9334  9199  9231  9199  9067  8991
    3547   1 2021-05-01 10402  9947  9378  9133  9264  9136  9179  9132  9034  8983
    3548   1 2021-05-01 10162  9671  9110  8858  8984  8889  8942  8898  8798  8724
    3549   1 2021-05-01  9692  9467  8904  8644  8762  8668  8730  8741  8585  8289
    3550   1 2021-05-01  9978  9558  8975  8706  8825  8723  8764  8793  8627  8530
    3551   1 2021-05-01  9784  9233  8650  8382  8482  8380  8440  8444  8293  8276
    3552   1 2021-05-01 10097  9627  9025  8765  8871  8751  8803  8756  8653  8550
    3553   1 2021-05-01 10099  9612  9039  8798  8909  8800  8853  8789  8713  8562
    3554   1 2021-05-01 10050  9598  9036  8804  8921  8816  8881  8820  8747  8535
    3555   1 2021-05-01  9943  9391  8862  8648  8763  8680  8751  8629  8617  8362
    3556   1 2021-05-01  9738  9446  8902  8679  8783  8698  8768  8693  8639  8115
    3557   1 2021-05-01  9934  9659  9082  8847  8944  8833  8895  8826  8751  8291
    3558   1 2021-05-01 10174  9609  9053  8827  8933  8831  8892  8773  8757  8528
    3559   1 2021-05-01 10127  9414  8868  8646  8753  8674  8746  8619  8614  8461
    3560   1 2021-05-01  8181  8440  7840  7557  7539  7396  7485  7428  7368  5165
    3561   1 2021-05-01  9221  9517  8857  8542  8535  8342  8401  8404  8248  6273
    3562   1 2021-05-01 10435 10213  9527  9215  9225  9014  9063  9065  8899  7764
    3563   1 2021-05-01 11021 10882 10204  9897  9922  9692  9726  9691  9548  8373
    3564   1 2021-05-01 11451 11278 10556 10228 10266 10013 10025 10015  9830  8814
    3565   1 2021-05-01 11798 11279 10568 10248 10301 10058 10062 10006  9878  9224
    3566   1 2021-05-01 12205 11567 10842 10517 10589 10329 10326 10246 10119  9728
    3567   1 2021-05-01 12309 11718 10973 10651 10715 10441 10442 10377 10228  9821
    3568   1 2021-05-01 12282 11818 11078 10751 10818 10538 10545 10474 10330  9719
    3569   1 2021-05-01 12339 11905 11184 10886 10930 10658 10670 10639 10469  9708
    3570   1 2021-05-01 12369 11910 11172 10858 10909 10625 10629 10596 10430  9825
    3571   1 2021-05-01 12338 11934 11180 10848 10888 10615 10606 10598 10382  9746
    3572   1 2021-05-01 12532 11996 11223 10877 10922 10624 10604 10628 10372  9976
    3573   1 2021-05-01 12640 12286 11507 11170 11223 10923 10900 10917 10665 10175
    3574   1 2021-05-01 12877 12524 11769 11461 11538 11239 11226 11183 11006 10415
    3575   1 2021-05-01 12871 12365 11612 11292 11362 11066 11063 11022 10833 10461
    3576   1 2021-05-01 12774 12288 11526 11202 11276 10981 10971 10918 10740 10318
    3577   1 2021-05-01 12747 12323 11573 11249 11315 11021 11013 10993 10781 10321
    3578   1 2021-05-01 12862 12373 11639 11329 11403 11107 11102 11071 10878 10367
    3579   1 2021-05-01 12817 12373 11639 11329 11403 11107 11102 11071 10878 10408
    3580   1 2021-05-01 12687 12263 11523 11211 11294 11002 11000 10930 10777 10330
    3581   1 2021-05-01 12735 12234 11482 11152 11228 10923 10912 10881 10667 10413
    3582   1 2021-05-01 12820 12291 11544 11224 11303 11011 10992 10926 10758 10495
    3583   1 2021-05-01 12832 12305 11555 11242 11321 11039 11020 10942 10807 10514
    3584   1 2021-05-01 12730 12238 11467 11135 11203 10909 10885 10848 10650 10469
    3585   1 2021-05-01 12621 12153 11399 11085 11152 10862 10845 10803 10609 10358
    3586   1 2021-05-01 12453 11894 11162 10848 10906 10625 10611 10556 10383 10138
    3587   1 2021-05-01 12009 11809 11073 10755 10806 10533 10536 10534 10304  9625
    3588   1 2021-05-01 12019 11627 10899 10580 10625 10363 10363 10371 10139  9595
    3589   1 2021-05-01 12162 11872 11135 10821 10880 10606 10609 10610 10382  9789
    3590   1 2021-05-01 12054 11596 10855 10529 10584 10315 10310 10311 10097  9647
    3591   1 2021-05-01 12101 11793 11031 10685 10749 10460 10444 10432 10215  9741
    3592   1 2021-05-01 12465 11793 11031 10685 10749 10460 10444 10432 10215 10217
    3593   1 2021-05-01 12627 12047 11284 10962 11041 10755 10730 10668 10502 10400
    3594   1 2021-05-01 12630 12116 11364 11050 11120 10848 10830 10755 10593 10371
    3595   1 2021-05-01 12545 12004 11251 10932 11002 10723 10703 10651 10472 10294
    3596   1 2021-05-01 12390 11981 11235 10918 10987 10708 10690 10644 10469 10126
    3597   1 2021-05-01 12484 12019 11281 10974 11048 10771 10761 10688 10535 10244
    3598   1 2021-05-01 12380 11980 11233 10922 10994 10712 10705 10665 10482 10127
    3599   1 2021-05-01 12666 12206 11458 11130 11205 10922 10907 10881 10680 10479
    3600   1 2021-05-01 12790 12340 11589 11278 11352 11075 11056 11033 10832 10568
    3601   1 2021-05-01 12703 12242 11498 11183 11248 10970 10946 10961 10712 10450
    3602   1 2021-05-01 12619 12232 11487 11166 11241 10965 10936 10925 10704 10325
    3603   1 2021-05-01 12745 12299 11555 11252 11339 11066 11049 11001 10833 10496
    3604   1 2021-05-01 12746 12300 11562 11256 11339 11067 11049 11021 10832 10503
    3605   1 2021-05-01 11748 11465 10775 10483 10574 10353 10357 10253 10171  9615
    3606   1 2021-05-01 11742 11172 10494 10214 10300 10084 10096 10023  9905  9616
    3607   1 2021-05-01 11451 10904 10264 10009 10101  9909  9934  9835  9764  9406
    3608   1 2021-05-01 11164 10750 10125  9864  9965  9781  9810  9698  9639  9177
    3609   1 2021-05-01 11118 10533  9907  9651  9753  9592  9622  9549  9462  9147
    3610   1 2021-05-01  9729  9116  8551  8308  8403  8314  8390  8316  8256  7985
    3611   1 2021-05-01  9356  8277  7823  7633  7739  7715  7829  7631  7737  7597
    3612   1 2021-05-01  8644  8277  7823  7633  7739  7715  7829  7631  7737  6975
    3613   1 2021-05-01 10003  9619  9045  8785  8916  8813  8865  8831  8731  8591
    3614   1 2021-05-01 10043  9625  9044  8767  8887  8777  8812  8862  8671  8681
    3615   1 2021-05-01 10193  9816  9240  8969  9095  8972  9007  9054  8857  8812
    3616   1 2021-05-01 10159  9816  9240  8969  9095  8972  9007  9054  8857  8802
    3617   1 2021-05-01 10266  9984  9387  9119  9232  9111  9145  9174  8989  8889
    3618   1 2021-05-01 10442  9976  9383  9112  9222  9092  9118  9149  8957  9036
    3619   1 2021-05-01  9557  9326  8766  8505  8621  8540  8601  8614  8461  8165
    3620   1 2021-05-01  9753  9383  8822  8565  8682  8589  8647  8637  8502  8335
    3621   1 2021-05-01 10143  9657  9076  8832  8941  8846  8902  8915  8763  8658
    3622   1 2021-05-01 10059  9639  9073  8847  8958  8863  8927  8898  8793  8595
    3623   1 2021-05-01  9984  9531  8967  8734  8839  8748  8805  8800  8687  8503
    3624   1 2021-05-01 10040  9605  9039  8809  8924  8816  8879  8827  8747  8482
    3625   1 2021-05-01  9963  9446  8902  8679  8783  8698  8768  8693  8639  8351
    3626   1 2021-05-01  9933  9686  9105  8864  8962  8848  8911  8894  8779  8302
    3627   1 2021-05-01  9951  9734  9155  8918  9016  8904  8957  8932  8823  8307
    3628   1 2021-05-01  9856  9369  8808  8568  8660  8574  8638  8628  8502  8193
    3629   1 2021-05-01  9129  8438  7854  7578  7563  7427  7507  7437  7387  6237
    3630   1 2021-05-01  9604  9331  8683  8376  8371  8185  8251  8245  8106  6859
    3631   1 2021-05-01 10282 10056  9381  9069  9077  8870  8915  8929  8749  7616
    3632   1 2021-05-01 11243 10801 10104  9786  9804  9569  9589  9602  9411  8656
    3633   1 2021-05-01 11600 11096 10377 10045 10076  9820  9823  9823  9613  9045
    3634   1 2021-05-01 11808 11544 10791 10437 10494 10206 10198 10221  9975  9315
    3635   1 2021-05-01 12119 11714 10962 10623 10680 10398 10398 10385 10173  9626
    3636   1 2021-05-01 12348 11901 11167 10843 10900 10622 10624 10582 10416  9787
    3637   1 2021-05-01 12332 11907 11168 10837 10886 10607 10600 10579 10385  9765
    3638   1 2021-05-01 12392 11921 11184 10866 10919 10642 10645 10591 10431  9823
    3639   1 2021-05-01 12384 11848 11108 10782 10836 10558 10563 10492 10343  9836
    3640   1 2021-05-01 12256 11964 11198 10849 10915 10626 10607 10557 10391  9720
    3641   1 2021-05-01 12688 12184 11419 11091 11162 10857 10843 10805 10618 10249
    3642   1 2021-05-01 12705 12184 11419 11091 11162 10857 10843 10805 10618 10303
    3643   1 2021-05-01 12756 12381 11602 11267 11319 11008 10996 11026 10752 10322
    3644   1 2021-05-01 12741 12280 11515 11190 11248 10954 10939 10929 10705 10320
    3645   1 2021-05-01 12669 12282 11526 11201 11266 10974 10965 10942 10743 10212
    3646   1 2021-05-01 12614 12170 11424 11101 11162 10882 10870 10851 10649 10136
    3647   1 2021-05-01 12538 12148 11400 11075 11128 10836 10818 10842 10588 10049
    3648   1 2021-05-01 12465 12078 11340 11028 11080 10792 10780 10773 10557  9993
    3649   1 2021-05-01 12613 12179 11410 11080 11148 10848 10832 10821 10587 10283
    3650   1 2021-05-01 12741 12263 11505 11189 11258 10956 10938 10921 10706 10444
    3651   1 2021-05-01 12754 12277 11532 11222 11296 11001 10988 10944 10750 10436
    3652   1 2021-05-01 12759 12167 11414 11097 11170 10869 10857 10826 10617 10430
    3653   1 2021-05-01 12641 12164 11393 11072 11137 10843 10825 10825 10587 10359
    3654   1 2021-05-01 12564 11945 11216 10916 10989 10716 10707 10624 10487 10279
    3655   1 2021-05-01 12390 11945 11216 10916 10989 10716 10707 10624 10487 10114
    3656   1 2021-05-01 12119 11561 10842 10531 10591 10323 10330 10272 10111  9808
    3657   1 2021-05-01 11791 11395 10675 10366 10427 10172 10178 10113  9961  9401
    3658   1 2021-05-01 11798 11513 10802 10488 10542 10287 10288 10261 10075  9394
    3659   1 2021-05-01 11721 11290 10587 10279 10338 10087 10095 10019  9892  9326
    3660   1 2021-05-01 11996 11651 10888 10538 10600 10310 10288 10283 10068  9654
    3661   1 2021-05-01 12339 11963 11193 10868 10937 10645 10616 10606 10394 10097
    3662   1 2021-05-01 12413 12075 11314 10997 11066 10782 10746 10736 10510 10169
    3663   1 2021-05-01 12538 12085 11328 11007 11072 10793 10757 10751 10532 10248
    3664   1 2021-05-01 12664 12121 11375 11057 11133 10853 10835 10777 10606 10400
    3665   1 2021-05-01 12625 12169 11436 11139 11213 10934 10922 10867 10688 10361
    3666   1 2021-05-01 12573 12073 11332 11022 11102 10821 10804 10747 10588 10313
    3667   1 2021-05-01 12567 12230 11481 11166 11243 10963 10943 10913 10716 10332
    3668   1 2021-05-01 12689 12230 11474 11153 11238 10948 10930 10905 10707 10460
    3669   1 2021-05-01 12622 12296 11556 11242 11317 11037 11023 11010 10800 10329
    3670   1 2021-05-01 12630 12169 11430 11124 11195 10929 10908 10856 10685 10349
    3671   1 2021-05-01 12653 12178 11442 11130 11204 10930 10906 10872 10681 10339
    3672   1 2021-05-01 12614 12248 11510 11202 11277 11006 10990 10965 10771 10257
    3673   1 2021-05-01 12619 12116 11380 11061 11133 10847 10834 10858 10611 10312
    3674   1 2021-05-01 12585 12189 11473 11178 11260 10987 10982 10940 10762 10321
    3675   1 2021-05-01 12399 11947 11236 10924 11008 10739 10744 10700 10529 10117
    3676   1 2021-05-01 11598 11068 10399 10109 10190  9981  9993  9966  9807  9474
    3677   1 2021-05-01 11475 11069 10403 10122 10208 10005 10019  9963  9833  9407
    3678   1 2021-05-01 11196 10926 10281 10004 10094  9903  9919  9872  9736  9210
    3679   1 2021-05-01 10630 10233  9624  9369  9466  9320  9366  9305  9205  8763
    3680   1 2021-05-01 10246  9714  9127  8881  8986  8862  8914  8818  8760  8426
    3681   1 2021-05-01  9283  9126  8580  8356  8456  8374  8452  8348  8322  7528
    3682   1 2021-05-01  8629  8434  7920  7691  7776  7728  7827  7745  7713  6871
    3683   1 2021-05-01  9976  9538  8954  8670  8792  8687  8734  8794  8580  8587
    3684   1 2021-05-01  9876  9477  8895  8613  8737  8629  8670  8722  8523  8512
    3685   1 2021-05-01 10099  9738  9148  8872  8979  8868  8891  8929  8735  8692
    3686   1 2021-05-01 10225  9899  9309  9039  9152  9024  9051  9082  8898  8817
    3687   1 2021-05-01 10251  9951  9355  9090  9208  9081  9112  9128  8953  8821
    3688   1 2021-05-01  9736  9329  8789  8555  8679  8592  8666  8607  8526  8270
    3689   1 2021-05-01  9731  9334  8771  8514  8634  8548  8613  8619  8473  8305
    3690   1 2021-05-01  9804  9538  8977  8748  8859  8771  8838  8826  8714  8318
    3691   1 2021-05-01  9805  9531  8967  8734  8839  8748  8805  8800  8687  8294
    3692   1 2021-05-01  9922  9493  8943  8703  8805  8718  8785  8754  8659  8365
    3693   1 2021-05-01  9720  9365  8796  8545  8639  8563  8631  8656  8511  8160
    3694   1 2021-05-01  9503  9071  8509  8255  8328  8262  8326  8342  8194  7872
    3695   1 2021-05-01  9296  8981  8437  8194  8275  8206  8287  8267  8150  7618
    3696   1 2021-05-01  9376  9075  8439  8138  8133  7959  8019  8008  7865  6571
    3697   1 2021-05-01  9930  9833  9165  8843  8847  8644  8686  8707  8527  7225
    3698   1 2021-05-01 10504 10255  9568  9239  9257  9035  9060  9060  8890  7844
    3699   1 2021-05-01 11009 10868 10134  9786  9820  9556  9564  9575  9354  8408
    3700   1 2021-05-01 11596 11212 10461 10112 10160  9885  9880  9873  9666  9155
    3701   1 2021-05-01 11815 11483 10734 10388 10443 10162 10151 10159  9938  9359
    3702   1 2021-05-01 12041 11483 10734 10388 10443 10162 10151 10159  9938  9472
    3703   1 2021-05-01 11938 11760 11027 10704 10743 10468 10487 10478 10279  9252
    3704   1 2021-05-01 12207 11700 10951 10609 10651 10377 10378 10375 10163  9633
    3705   1 2021-05-01 12449 11975 11228 10892 10945 10654 10644 10630 10423  9922
    3706   1 2021-05-01 12332 11848 11108 10782 10836 10558 10563 10492 10343  9806
    3707   1 2021-05-01 12414 12084 11298 10943 10992 10693 10662 10692 10436  9908
    3708   1 2021-05-01 12779 12278 11518 11194 11272 10974 10959 10911 10737 10330
    3709   1 2021-05-01 12703 12237 11465 11132 11195 10895 10879 10861 10651 10294
    3710   1 2021-05-01 12712 12228 11474 11148 11207 10919 10906 10877 10671 10261
    3711   1 2021-05-01 12596 12129 11367 11040 11094 10809 10785 10773 10550 10120
    3712   1 2021-05-01 12618 12053 11305 10981 11044 10753 10741 10713 10501 10133
    3713   1 2021-05-01 12388 11972 11218 10881 10940 10653 10634 10589 10410  9956
    3714   1 2021-05-01 12491 12036 11291 10973 11034 10745 10726 10698 10505 10095
    3715   1 2021-05-01 12523 12112 11377 11070 11139 10855 10851 10817 10627 10130
    3716   1 2021-05-01 12574 12110 11358 11038 11099 10806 10803 10771 10571 10187
    3717   1 2021-05-01 12640 12157 11423 11113 11188 10898 10887 10850 10668 10279
    3718   1 2021-05-01 12528 12187 11442 11133 11199 10901 10898 10880 10666 10185
    3719   1 2021-05-01 12557 12167 11414 11097 11170 10869 10857 10826 10617 10277
    3720   1 2021-05-01 12552 12064 11299 10972 11041 10746 10724 10699 10481 10290
    3721   1 2021-05-01 12505 11992 11246 10924 10991 10705 10687 10670 10466 10247
    3722   1 2021-05-01 12368 11860 11120 10799 10868 10587 10577 10525 10349 10092
    3723   1 2021-05-01 12258 11627 10908 10615 10693 10425 10436 10315 10217  9980
    3724   1 2021-05-01 12025 11553 10853 10568 10647 10385 10399 10307 10205  9736
    3725   1 2021-05-01 11938 11357 10662 10359 10431 10188 10187 10077  9991  9620
    3726   1 2021-05-01 12064 11615 10872 10544 10610 10348 10331 10275 10109  9753
    3727   1 2021-05-01 12273 11911 11149 10827 10891 10609 10594 10570 10352 10021
    3728   1 2021-05-01 12417 11937 11190 10880 10954 10681 10671 10600 10429 10180
    3729   1 2021-05-01 12406 12013 11261 10942 11012 10731 10700 10680 10475 10174
    3730   1 2021-05-01 12542 12013 11261 10942 11012 10731 10700 10680 10475 10277
    3731   1 2021-05-01 12521 12096 11338 11016 11094 10813 10791 10754 10563 10282
    3732   1 2021-05-01 12454 12078 11314 10979 11050 10758 10728 10723 10489 10238
    3733   1 2021-05-01 12538 12139 11389 11075 11146 10859 10832 10844 10602 10286
    3734   1 2021-05-01 12500 12103 11370 11066 11142 10863 10849 10803 10629 10233
    3735   1 2021-05-01 12567 12173 11417 11097 11174 10882 10865 10856 10637 10288
    3736   1 2021-05-01 12651 12235 11506 11202 11275 11006 10993 10950 10762 10376
    3737   1 2021-05-01 12534 12081 11335 11014 11087 10801 10793 10761 10557 10248
    3738   1 2021-05-01 12597 12153 11411 11091 11166 10881 10855 10845 10635 10304
    3739   1 2021-05-01 12482 11995 11274 10978 11042 10781 10770 10720 10558 10194
    3740   1 2021-05-01 12333 11846 11125 10812 10884 10616 10617 10576 10413  9966
    3741   1 2021-05-01 12323 11730 10998 10670 10738 10473 10469 10433 10246  9964
    3742   1 2021-05-01 12227 11803 11082 10777 10852 10588 10592 10559 10367  9864
    3743   1 2021-05-01 12188 11803 11082 10777 10852 10588 10592 10559 10367  9888
    3744   1 2021-05-01 12134 11718 10987 10661 10739 10478 10473 10446 10254  9897
    3745   1 2021-05-01 12112 11727 11013 10709 10776 10528 10523 10514 10298  9902
    3746   1 2021-05-01 10196  9735  9146  8898  8982  8856  8913  8847  8766  8313
    3747   1 2021-05-01  9876  9222  8675  8443  8539  8438  8510  8420  8376  8030
    3748   1 2021-05-01  9461  8961  8428  8196  8288  8194  8276  8141  8136  7637
    3749   1 2021-05-01 10027  9534  8931  8642  8745  8632  8664  8737  8517  8545
    3750   1 2021-05-01  9511  9120  8592  8348  8464  8401  8472  8466  8340  8041
    3751   1 2021-05-01  9242  8470  7958  7707  7808  7784  7883  7920  7767  7881
    3752   1 2021-05-01  9385  8929  8379  8115  8219  8156  8218  8281  8085  8026
    3753   1 2021-05-01  9703  9319  8774  8540  8653  8574  8656  8620  8535  8229
    3754   1 2021-05-01  9768  9394  8841  8605  8709  8627  8706  8668  8580  8243
    3755   1 2021-05-01  9380  9117  8619  8412  8521  8467  8545  8458  8432  7757
    3756   1 2021-05-01  9135  8790  8248  7996  8069  8009  8093  8094  7968  7416
    3757   1 2021-05-01  8454  8020  7437  7158  7159  7023  7108  6965  6975  5787
    3758   1 2021-05-01  9303  8903  8268  7958  7943  7769  7819  7821  7662  6457
    3759   1 2021-05-01 10193  9583  8930  8619  8619  8425  8482  8474  8324  7473
    3760   1 2021-05-01 10681 10108  9410  9076  9085  8857  8880  8892  8697  8039
    3761   1 2021-05-01 11094 10711  9984  9640  9669  9417  9428  9428  9226  8553
    3762   1 2021-05-01 11536 11123 10385 10047 10089  9827  9821  9806  9599  9095
    3763   1 2021-05-01 11747 11299 10560 10221 10274 10002  9994  9973  9788  9217
    3764   1 2021-05-01 11805 11280 10559 10228 10280 10015 10013  9987  9800  9172
    3765   1 2021-05-01 11816 11432 10693 10358 10407 10139 10144 10109  9933  9172
    3766   1 2021-05-01 12008 11829 11066 10706 10753 10460 10444 10470 10217  9409
    3767   1 2021-05-01 12334 11795 11042 10702 10755 10477 10463 10421 10244  9777
    3768   1 2021-05-01 12351 11935 11146 10795 10844 10544 10524 10525 10305  9778
    3769   1 2021-05-01 12586 12232 11450 11109 11166 10863 10840 10862 10610 10069
    3770   1 2021-05-01 12742 12192 11427 11093 11149 10856 10834 10811 10607 10283
    3771   1 2021-05-01 12659 12267 11499 11163 11222 10920 10891 10901 10653 10244
    3772   1 2021-05-01 12723 12210 11458 11141 11210 10917 10902 10855 10667 10272
    3773   1 2021-05-01 12573 12129 11376 11066 11139 10850 10837 10753 10605 10146
    3774   1 2021-05-01 12409 11971 11206 10870 10921 10627 10608 10593 10376  9952
    3775   1 2021-05-01 12464 11999 11226 10883 10943 10645 10626 10600 10400 10032
    3776   1 2021-05-01 12522 12108 11349 11028 11088 10794 10774 10770 10542 10103
    3777   1 2021-05-01 12478 12048 11324 11026 11088 10801 10794 10776 10561 10042
    3778   1 2021-05-01 12493 12009 11275 10958 11027 10743 10737 10726 10520 10074
    3779   1 2021-05-01 12374 11997 11256 10941 11003 10720 10706 10686 10491  9970
    3780   1 2021-05-01 12443 11998 11217 10881 10935 10638 10627 10628 10374 10129
    3781   1 2021-05-01 12633 12145 11385 11069 11150 10847 10834 10785 10595 10371
    3782   1 2021-05-01 12749 12195 11462 11164 11250 10963 10958 10882 10735 10470
    3783   1 2021-05-01 12641 11912 11170 10859 10936 10665 10653 10575 10433 10383
    3784   1 2021-05-01 12408 11629 10898 10583 10650 10391 10378 10312 10158 10145
    3785   1 2021-05-01 12045 11748 11015 10703 10767 10490 10486 10461 10264  9765
    3786   1 2021-05-01 11906 11551 10835 10530 10590 10322 10323 10312 10122  9620
    3787   1 2021-05-01 11865 11476 10772 10471 10540 10283 10285 10231 10095  9543
    3788   1 2021-05-01 12027 11599 10854 10524 10589 10316 10302 10294 10093  9730
    3789   1 2021-05-01 12191 11855 11108 10787 10856 10577 10564 10535 10331  9955
    3790   1 2021-05-01 12263 11819 11068 10736 10805 10526 10511 10508 10284 10014
    3791   1 2021-05-01 12298 11888 11131 10798 10862 10578 10557 10553 10321 10050
    3792   1 2021-05-01 12414 11994 11246 10932 11001 10706 10689 10674 10457 10157
    3793   1 2021-05-01 12324 11940 11180 10846 10918 10619 10602 10605 10361 10104
    3794   1 2021-05-01 12434 12030 11280 10959 11031 10747 10720 10712 10491 10234
    3795   1 2021-05-01 12391 11954 11221 10914 10981 10703 10698 10662 10473 10147
    3796   1 2021-05-01 12429 12064 11313 11001 11062 10786 10771 10761 10532 10139
    3797   1 2021-05-01 12463 12044 11306 10985 11052 10774 10760 10736 10531 10171
    3798   1 2021-05-01 12444 12076 11328 10994 11060 10772 10756 10756 10511 10139
    3799   1 2021-05-01 12564 12076 11328 10994 11060 10772 10756 10756 10511 10270
    3800   1 2021-05-01 12523 12114 11380 11058 11135 10852 10835 10809 10613 10225
    3801   1 2021-05-01 12476 12002 11284 10975 11035 10764 10752 10722 10530 10161
    3802   1 2021-05-01 12214 11793 11079 10790 10857 10599 10609 10531 10406  9841
    3803   1 2021-05-01 12174 11740 11007 10692 10761 10507 10506 10394 10293  9827
    3804   1 2021-05-01 12304 11808 11079 10763 10837 10572 10570 10515 10343 10001
    3805   1 2021-05-01 12306 11804 11091 10792 10880 10630 10632 10531 10429 10008
    3806   1 2021-05-01 12280 11810 11109 10803 10893 10649 10641 10558 10443 10090
    3807   1 2021-05-01 12150 11505 10825 10538 10627 10392 10407 10342 10217  9983
    3808   1 2021-05-01 11787 11292 10616 10336 10417 10199 10225 10159 10044  9634
    3809   1 2021-05-01 10551 10147  9564  9327  9427  9277  9340  9232  9179  8580
    3810   1 2021-05-01  9969  9560  8990  8750  8836  8711  8784  8726  8642  8040
    3811   1 2021-05-01  8477  8470  7958  7707  7808  7784  7883  7920  7767  7148
    3812   1 2021-05-01  8838  8539  8012  7759  7853  7814  7909  7934  7785  7475
    3813   1 2021-05-01  9510  9287  8739  8505  8613  8532  8618  8574  8482  7982
    3814   1 2021-05-01  8485  7980  7393  7112  7109  6974  7058  6899  6925  5808
    3815   1 2021-05-01  8491  7999  7423  7144  7148  7014  7096  6894  6960  5787
    3816   1 2021-05-01  8187  7583  7040  6788  6788  6685  6791  6557  6680  5429
    3817   1 2021-05-01  8249  7540  6975  6692  6666  6545  6637  6567  6507  5392
    3818   1 2021-05-01  9048  8752  8094  7765  7737  7557  7611  7619  7441  6259
    3819   1 2021-05-01  9685  9376  8691  8345  8339  8127  8153  8162  7969  6947
    3820   1 2021-05-01 10133  9958  9246  8897  8902  8677  8684  8697  8496  7428
    3821   1 2021-05-01 10860 10453  9706  9332  9353  9102  9084  9134  8872  8314
    3822   1 2021-05-01 11263 10978 10229  9880  9905  9652  9634  9663  9416  8780
    3823   1 2021-05-01 11555 11026 10283  9934  9959  9699  9688  9715  9465  9078
    3824   1 2021-05-01 11649 11165 10435 10098 10134  9874  9869  9851  9652  9111
    3825   1 2021-05-01 11624 11374 10636 10301 10347 10085 10080 10040  9868  9042
    3826   1 2021-05-01 11913 11374 10636 10301 10347 10085 10080 10040  9868  9334
    3827   1 2021-05-01 12090 11628 10868 10509 10560 10278 10265 10252 10045  9546
    3828   1 2021-05-01 12104 11689 10934 10583 10636 10353 10335 10313 10129  9521
    3829   1 2021-05-01 12230 11897 11124 10779 10833 10538 10519 10499 10298  9647
    3830   1 2021-05-01 12515 12034 11259 10928 10970 10690 10664 10651 10435 10006
    3831   1 2021-05-01 12596 12170 11388 11042 11085 10802 10772 10773 10546 10121
    3832   1 2021-05-01 12645 12225 11435 11089 11132 10835 10815 10822 10579 10149
    3833   1 2021-05-01 12735 12267 11511 11191 11253 10969 10950 10898 10724 10255
    3834   1 2021-05-01 12579 12127 11379 11059 11121 10844 10836 10744 10609 10164
    3835   1 2021-05-01 12568 12063 11302 10977 11038 10749 10742 10682 10516 10087
    3836   1 2021-05-01 12589 12009 11262 10943 11001 10715 10707 10681 10477 10129
    3837   1 2021-05-01 12475 11974 11239 10926 10974 10686 10676 10670 10434 10034
    3838   1 2021-05-01 12337 11866 11146 10845 10904 10627 10635 10582 10417  9882
    3839   1 2021-05-01 12319 11877 11156 10844 10904 10623 10622 10578 10399  9930
    3840   1 2021-05-01 12338 11851 11123 10807 10873 10594 10584 10539 10365  9965
    3841   1 2021-05-01 12392 11907 11161 10838 10903 10620 10618 10568 10393 10043
    3842   1 2021-05-01 12608 12211 11447 11122 11194 10890 10877 10874 10638 10330
    3843   1 2021-05-01 12495 12183 11445 11132 11200 10907 10893 10896 10661 10210
    3844   1 2021-05-01 12439 11987 11263 10961 11036 10760 10746 10687 10529 10160
    3845   1 2021-05-01 12262 11713 10983 10676 10742 10474 10461 10405 10246 10027
    3846   1 2021-05-01 12098 11482 10772 10467 10534 10285 10279 10211 10069  9856
    3847   1 2021-05-01 11732 11301 10614 10333 10403 10160 10179 10075  9988  9448
    3848   1 2021-05-01 11696 11356 10617 10293 10338 10067 10066 10083  9853  9368
    3849   1 2021-05-01 12015 11487 10747 10428 10479 10214 10202 10216  9989  9741
    3850   1 2021-05-01 12029 11583 10856 10532 10599 10330 10325 10297 10098  9759
    3851   1 2021-05-01 12020 11745 10985 10650 10713 10432 10409 10408 10184  9720
    3852   1 2021-05-01 12292 11871 11118 10803 10863 10585 10569 10562 10331 10026
    3853   1 2021-05-01 12293 11853 11103 10777 10841 10556 10541 10522 10307 10031
    3854   1 2021-05-01 12285 11853 11103 10777 10841 10556 10541 10522 10307 10074
    3855   1 2021-05-01 12366 11990 11243 10927 10995 10712 10690 10670 10463 10110
    3856   1 2021-05-01 12404 11945 11208 10898 10963 10682 10673 10653 10454 10144
    3857   1 2021-05-01 12427 12032 11284 10976 11032 10764 10756 10729 10528 10142
    3858   1 2021-05-01 12489 12036 11294 10985 11043 10775 10761 10724 10540 10231
    3859   1 2021-05-01 12399 11981 11240 10924 10990 10715 10698 10650 10481 10145
    3860   1 2021-05-01 12517 12038 11301 11005 11069 10798 10787 10727 10575 10251
    3861   1 2021-05-01 12403 11918 11195 10885 10949 10682 10674 10622 10456 10107
    3862   1 2021-05-01 12373 12010 11273 10960 11029 10758 10749 10694 10524 10011
    3863   1 2021-05-01 12338 12007 11270 10964 11042 10781 10778 10667 10570 10006
    3864   1 2021-05-01 12683 12006 11285 10977 11059 10796 10797 10690 10576 10383
    3865   1 2021-05-01 12581 12076 11361 11065 11150 10900 10899 10821 10684 10306
    3866   1 2021-05-01 12437 11921 11210 10908 10987 10743 10731 10684 10528 10207
    3867   1 2021-05-01 12354 11921 11210 10908 10987 10743 10731 10684 10528 10135
    3868   1 2021-05-01 12056 11660 10985 10702 10790 10560 10569 10476 10376  9912
    3869   1 2021-05-01 11768 11435 10758 10487 10573 10365 10384 10286 10199  9661
    3870   1 2021-05-01 11541 11163 10483 10207 10279 10072 10112 10057  9924  9456
    3871   1 2021-05-01 11265 10803 10160  9896  9973  9789  9837  9781  9652  9177
    3872   1 2021-05-01 10102  9740  9162  8915  8998  8869  8954  8878  8810  8130
    3873   1 2021-05-01  7739  7774  7178  6876  6854  6713  6785  6722  6637  4971
    3874   1 2021-05-01  8458  8597  7930  7595  7585  7395  7431  7405  7263  5689
    3875   1 2021-05-01  9536  9190  8505  8153  8152  7944  7961  7937  7775  6903
    3876   1 2021-05-01  9900  9543  8846  8497  8501  8288  8302  8290  8117  7251
    3877   1 2021-05-01 10162  9803  9090  8734  8735  8503  8518  8534  8321  7496
    3878   1 2021-05-01 10608 10138  9408  9049  9058  8816  8818  8851  8608  8001
    3879   1 2021-05-01 11032 10682  9933  9567  9597  9334  9321  9364  9109  8492
    3880   1 2021-05-01 11353 11026 10283  9934  9959  9699  9688  9715  9465  8772
    3881   1 2021-05-01 11409 11059 10312  9956  9983  9723  9716  9723  9485  8824
    3882   1 2021-05-01 11619 11277 10532 10186 10223  9949  9937  9939  9716  9064
    3883   1 2021-05-01 11782 11335 10599 10252 10294 10016 10012  9996  9792  9209
    3884   1 2021-05-01 12009 11568 10825 10478 10521 10250 10247 10212 10028  9354
    3885   1 2021-05-01 12026 11590 10841 10491 10536 10264 10256 10216 10036  9338
    3886   1 2021-05-01 12257 11914 11149 10813 10854 10574 10559 10537 10334  9624
    3887   1 2021-05-01 12451 11875 11102 10757 10786 10494 10489 10504 10246  9852
    3888   1 2021-05-01 12430 12209 11436 11101 11145 10844 10837 10831 10602  9740
    3889   1 2021-05-01 12688 12180 11424 11091 11129 10832 10823 10846 10584 10109
    3890   1 2021-05-01 12721 12130 11382 11056 11116 10838 10831 10783 10598 10191
    3891   1 2021-05-01 12627 12130 11382 11056 11116 10838 10831 10783 10598 10161
    3892   1 2021-05-01 12304 12031 11284 10959 11024 10739 10732 10688 10514  9845
    3893   1 2021-05-01 12272 12009 11262 10943 11001 10715 10707 10681 10477  9849
    3894   1 2021-05-01 12362 11910 11161 10842 10894 10614 10594 10586 10366  9946
    3895   1 2021-05-01 12272 11907 11190 10891 10947 10669 10673 10626 10446  9816
    3896   1 2021-05-01 12284 11856 11119 10801 10853 10574 10567 10573 10340  9869
    3897   1 2021-05-01 12235 11906 11191 10888 10944 10668 10676 10645 10459  9810
    3898   1 2021-05-01 12379 11832 11084 10766 10821 10530 10533 10548 10310  9975
    3899   1 2021-05-01 12296 11933 11193 10871 10932 10661 10635 10641 10416  9941
    3900   1 2021-05-01 12344 11816 11081 10778 10836 10577 10567 10504 10356 10027
    3901   1 2021-05-01 12166 11659 10920 10607 10671 10398 10386 10350 10169  9928
    3902   1 2021-05-01 12044 11493 10769 10471 10541 10284 10291 10207 10083  9779
    3903   1 2021-05-01 11926 11349 10659 10377 10451 10198 10213 10136 10007  9638
    3904   1 2021-05-01 11822 11349 10659 10377 10451 10198 10213 10136 10007  9528
    3905   1 2021-05-01 11748 11212 10498 10190 10249  9997 10013  9933  9807  9468
    3906   1 2021-05-01 11729 11487 10747 10428 10479 10214 10202 10216  9989  9457
    3907   1 2021-05-01 11859 11448 10733 10417 10475 10217 10211 10185 10001  9561
    3908   1 2021-05-01 11880 11544 10788 10443 10503 10235 10220 10206  9989  9596
    3909   1 2021-05-01 12116 11690 10944 10608 10661 10395 10379 10383 10150  9867
    3910   1 2021-05-01 12083 11789 11027 10707 10762 10484 10474 10470 10243  9820
    3911   1 2021-05-01 12206 11760 11017 10698 10755 10479 10470 10463 10255  9951
    3912   1 2021-05-01 12241 11796 11064 10743 10798 10522 10512 10518 10292  9953
    3913   1 2021-05-01 12165 11761 11037 10725 10785 10513 10524 10502 10304  9835
    3914   1 2021-05-01 12301 11963 11226 10916 10967 10703 10688 10674 10464  9976
    3915   1 2021-05-01 12456 12048 11316 11007 11066 10801 10783 10756 10561 10172
    3916   1 2021-05-01 12514 12112 11369 11064 11121 10848 10837 10809 10614 10223
    3917   1 2021-05-01 12545 12083 11328 11006 11062 10768 10751 10775 10526 10252
    3918   1 2021-05-01 12496 12021 11297 10998 11074 10797 10789 10715 10582 10191
    3919   1 2021-05-01 12551 12010 11273 10960 11029 10758 10749 10694 10524 10214
    3920   1 2021-05-01 12669 12232 11491 11181 11255 10983 10975 10938 10744 10357
    3921   1 2021-05-01 12689 12233 11497 11198 11282 11014 11010 10956 10793 10387
    3922   1 2021-05-01 12362 12075 11356 11054 11120 10861 10860 10855 10632 10067
    3923   1 2021-05-01 12191 11882 11169 10869 10937 10700 10698 10691 10488  9912
    3924   1 2021-05-01 12067 11664 10951 10657 10734 10494 10499 10478 10304  9840
    3925   1 2021-05-01 11970 11454 10753 10462 10544 10313 10320 10281 10127  9829
    3926   1 2021-05-01 11539 11124 10446 10177 10255 10047 10069  9993  9889  9427
    3927   1 2021-05-01 11283 10791 10149  9878  9965  9782  9820  9737  9643  9175
    3928   1 2021-05-01 10011  9493  8936  8700  8784  8670  8746  8659  8611  8014
    3929   1 2021-05-01  9727  9128  8630  8427  8516  8426  8508  8332  8386  7778
    3930   1 2021-05-01  8162  7307  6752  6494  6485  6386  6475  6228  6342  5445
    3931   1 2021-05-01  7682  7110  6605  6367  6358  6282  6386  6127  6266  4961
    3932   1 2021-05-01  7030  6578  6125  5930  5925  5883  6019  5744  5941  4369
    3933   1 2021-05-01  7185  6838  6347  6125  6115  6042  6163  5965  6048  4529
    3934   1 2021-05-01  7146  6657  6167  5942  5926  5859  5983  5793  5882  4474
    3935   1 2021-05-01  7491  6881  6364  6118  6108  6030  6143  5965  6030  4850
    3936   1 2021-05-01  7952  7647  7030  6711  6677  6536  6615  6615  6458  5288
    3937   1 2021-05-01  8833  8369  7704  7365  7338  7161  7209  7218  7035  6183
    3938   1 2021-05-01  9473  9119  8417  8061  8054  7839  7852  7836  7671  6878
    3939   1 2021-05-01  9845  9394  8696  8361  8365  8154  8163  8120  7973  7229
    3940   1 2021-05-01  9980  9685  8975  8623  8625  8409  8423  8418  8225  7313
    3941   1 2021-05-01 10448 10293  9547  9176  9194  8933  8930  8970  8711  7844
    3942   1 2021-05-01 10827 10468  9728  9369  9378  9133  9125  9145  8917  8190
    3943   1 2021-05-01 10951 10862 10104  9743  9761  9506  9483  9514  9267  8278
    3944   1 2021-05-01 11418 11005 10248  9903  9927  9671  9653  9647  9439  8839
    3945   1 2021-05-01 11557 11173 10417 10075 10106  9836  9826  9809  9609  9001
    3946   1 2021-05-01 11704 11194 10440 10107 10143  9879  9882  9823  9671  9103
    3947   1 2021-05-01 11696 11391 10648 10304 10329 10064 10070 10043  9848  9066
    3948   1 2021-05-01 11860 11391 10648 10304 10329 10064 10070 10043  9848  9087
    3949   1 2021-05-01 12069 11656 10899 10554 10592 10313 10308 10286 10096  9384
    3950   1 2021-05-01 12067 11594 10826 10477 10524 10246 10243 10181 10015  9437
    3951   1 2021-05-01 12160 11866 11105 10754 10787 10499 10491 10501 10254  9481
    3952   1 2021-05-01 12309 11834 11087 10759 10795 10519 10516 10478 10298  9682
    3953   1 2021-05-01 12402 11941 11197 10870 10909 10627 10624 10613 10397  9837
    3954   1 2021-05-01 12245 11942 11207 10876 10934 10659 10651 10616 10425  9709
    3955   1 2021-05-01 12262 11784 11039 10701 10756 10475 10475 10426 10249  9775
    3956   1 2021-05-01 12249 11931 11181 10859 10923 10643 10631 10599 10413  9835
    3957   1 2021-05-01 12287 11785 11073 10778 10846 10579 10594 10523 10378  9814
    3958   1 2021-05-01 11996 11546 10816 10503 10557 10288 10284 10258 10059  9567
    3959   1 2021-05-01 11984 11501 10789 10482 10536 10269 10277 10247 10079  9541
    3960   1 2021-05-01 12069 11619 10883 10573 10631 10354 10373 10337 10147  9636
    3961   1 2021-05-01 12122 11619 10883 10573 10631 10354 10373 10337 10147  9778
    3962   1 2021-05-01 12297 11787 11044 10717 10779 10496 10482 10470 10258  9992
    3963   1 2021-05-01 12366 11868 11140 10838 10902 10621 10619 10576 10388 10093
    3964   1 2021-05-01 12282 11737 11005 10703 10776 10513 10510 10423 10301 10052
    3965   1 2021-05-01 12031 11584 10845 10538 10611 10343 10338 10286 10125  9756
    3966   1 2021-05-01 11798 11309 10622 10336 10411 10167 10186 10092  9996  9511
    3967   1 2021-05-01 11608 11197 10499 10210 10276 10027 10049  9976  9845  9294
    3968   1 2021-05-01 11545 11243 10546 10249 10313 10059 10067 10030  9867  9236
    3969   1 2021-05-01 11846 11381 10666 10356 10421 10162 10159 10127  9953  9598
    3970   1 2021-05-01 12004 11586 10855 10535 10592 10335 10325 10290 10106  9729
    3971   1 2021-05-01 12117 11682 10951 10635 10697 10444 10429 10377 10213  9855
    3972   1 2021-05-01 12169 11709 10978 10673 10739 10460 10455 10428 10240  9917
    3973   1 2021-05-01 12152 11690 10955 10639 10705 10438 10425 10391 10215  9907
    3974   1 2021-05-01 12164 11679 10954 10643 10706 10444 10425 10413 10212  9913
    3975   1 2021-05-01 12110 11697 10974 10655 10717 10454 10452 10404 10240  9836
    3976   1 2021-05-01 12047 11652 10940 10641 10702 10445 10451 10396 10241  9691
    3977   1 2021-05-01 12147 11785 11049 10735 10788 10522 10515 10503 10291  9813
    3978   1 2021-05-01 12442 11877 11145 10837 10887 10612 10606 10610 10382 10099
    3979   1 2021-05-01 12303 11992 11249 10931 10986 10723 10705 10697 10486  9981
    3980   1 2021-05-01 12344 12083 11328 11006 11062 10768 10751 10775 10526 10043
    3981   1 2021-05-01 12562 12120 11386 11084 11161 10878 10884 10814 10671 10243
    3982   1 2021-05-01 12634 12194 11456 11141 11213 10935 10922 10906 10701 10315
    3983   1 2021-05-01 12540 12102 11394 11097 11162 10899 10906 10861 10695 10201
    3984   1 2021-05-01 12469 11960 11250 10961 11035 10787 10792 10722 10581 10144
    3985   1 2021-05-01 12333 11810 11108 10816 10889 10636 10647 10595 10432 10027
    3986   1 2021-05-01 12047 11608 10902 10608 10689 10447 10455 10416 10264  9772
    3987   1 2021-05-01 11938 11392 10701 10409 10476 10250 10256 10251 10074  9727
    3988   1 2021-05-01 11793 11455 10765 10485 10573 10343 10357 10275 10180  9631
    3989   1 2021-05-01 11458 11091 10426 10159 10240 10028 10065  9989  9882  9279
    3990   1 2021-05-01 11267 10814 10165  9895  9975  9784  9814  9735  9639  9119
    3991   1 2021-05-01 10263  9930  9383  9168  9265  9138  9207  9065  9060  8286
    3992   1 2021-05-01 10049  9493  8936  8700  8784  8670  8746  8659  8611  8082
    3993   1 2021-05-01  9821  9428  8869  8632  8716  8600  8667  8558  8527  7871
    3994   1 2021-05-01  9625  9090  8562  8345  8439  8343  8419  8248  8291  7696
    3995   1 2021-05-01  8916  8244  7692  7453  7503  7381  7453  7096  7340  6408
    3996   1 2021-05-01  8164  7515  6969  6714  6721  6617  6708  6447  6583  5531
    3997   1 2021-05-01  7842  7320  6797  6552  6552  6459  6563  6331  6441  5168
    3998   1 2021-05-01  7336  6885  6403  6191  6191  6126  6252  6037  6166  4826
    3999   1 2021-05-01  7281  6860  6372  6157  6160  6090  6217  6012  6123  4742
    4000   1 2021-05-01  7235  6845  6324  6075  6066  5980  6089  5955  5980  4701
    4001   1 2021-05-01  7350  7296  6708  6412  6387  6267  6345  6284  6208  4868
    4002   1 2021-05-01  8578  8073  7444  7133  7115  6958  7012  6951  6850  6025
    4003   1 2021-05-01  9162  8803  8112  7781  7782  7588  7616  7555  7439  6612
    4004   1 2021-05-01  9689  9347  8637  8293  8299  8080  8095  8051  7889  7165
    4005   1 2021-05-01  9927  9474  8771  8427  8438  8222  8238  8167  8044  7445
    4006   1 2021-05-01  9971  9891  9159  8805  8816  8585  8584  8539  8371  7396
    4007   1 2021-05-01 10602  9985  9249  8891  8896  8662  8666  8649  8452  8015
    4008   1 2021-05-01 10776 10292  9557  9203  9214  8969  8964  8961  8749  8171
    4009   1 2021-05-01 10982 10656  9900  9541  9547  9292  9279  9305  9062  8322
    4010   1 2021-05-01 11300 10864 10109  9749  9764  9498  9487  9508  9263  8676
    4011   1 2021-05-01 11565 11073 10318  9980 10007  9749  9744  9700  9525  8982
    4012   1 2021-05-01 11663 11172 10419 10082 10118  9852  9856  9799  9635  9044
    4013   1 2021-05-01 11676 11174 10436 10105 10135  9877  9885  9824  9680  8991
    4014   1 2021-05-01 11641 11196 10458 10131 10157  9903  9918  9829  9711  8852
    4015   1 2021-05-01 11911 11606 10841 10496 10520 10237 10244 10227 10011  9118
    4016   1 2021-05-01 12293 11875 11114 10777 10810 10521 10521 10511 10291  9590
    4017   1 2021-05-01 12386 11906 11178 10864 10912 10626 10636 10562 10420  9723
    4018   1 2021-05-01 12266 11718 10977 10648 10702 10429 10429 10349 10213  9672
    4019   1 2021-05-01 12300 11751 11019 10713 10778 10515 10524 10379 10321  9758
    4020   1 2021-05-01 12258 11788 11034 10696 10750 10466 10460 10431 10228  9795
    4021   1 2021-05-01 12301 11788 11034 10696 10750 10466 10460 10431 10228  9842
    4022   1 2021-05-01 12379 11954 11218 10908 10966 10683 10686 10654 10462  9916
    4023   1 2021-05-01 12184 11745 11033 10735 10794 10523 10543 10486 10329  9691
    4024   1 2021-05-01 12081 11563 10841 10538 10598 10334 10345 10254 10128  9662
    4025   1 2021-05-01 11870 11513 10823 10545 10607 10351 10374 10266 10178  9425
    4026   1 2021-05-01 11874 11507 10785 10476 10535 10263 10264 10239 10057  9450
    4027   1 2021-05-01 12107 11790 11032 10709 10774 10491 10472 10444 10242  9735
    4028   1 2021-05-01 12383 11872 11139 10828 10902 10612 10605 10559 10379 10156
    4029   1 2021-05-01 12257 11832 11103 10812 10894 10623 10611 10516 10404 10046
    4030   1 2021-05-01 12152 11523 10785 10477 10549 10283 10285 10206 10076  9934
    4031   1 2021-05-01 11931 11278 10573 10276 10336 10084 10090 10047  9880  9661
    4032   1 2021-05-01 11517 11086 10404 10123 10177  9947  9969  9877  9758  9198
    4033   1 2021-05-01 11640 11163 10483 10197 10259 10019 10036  9973  9834  9318
    4034   1 2021-05-01 11652 11163 10483 10197 10259 10019 10036  9973  9834  9349
    4035   1 2021-05-01 11895 11414 10700 10395 10467 10208 10202 10144  9995  9649
    4036   1 2021-05-01 12006 11556 10850 10548 10606 10352 10349 10297 10143  9729
    4037   1 2021-05-01 11900 11621 10885 10574 10630 10365 10365 10345 10145  9619
    4038   1 2021-05-01 12003 11550 10814 10508 10576 10310 10308 10269 10101  9731
    4039   1 2021-05-01 12038 11615 10877 10557 10619 10345 10332 10314 10112  9769
    4040   1 2021-05-01 12021 11603 10873 10545 10612 10335 10328 10311 10099  9763
    4041   1 2021-05-01 12022 11604 10907 10611 10676 10418 10431 10378 10222  9669
    4042   1 2021-05-01 11917 11496 10792 10496 10555 10303 10319 10274 10122  9535
    4043   1 2021-05-01 12098 11662 10937 10624 10686 10421 10423 10398 10205  9745
    4044   1 2021-05-01 12028 11726 10979 10651 10701 10419 10412 10417 10188  9694
    4045   1 2021-05-01 12210 11804 11062 10743 10803 10533 10534 10481 10316  9869
    4046   1 2021-05-01 12367 12110 11359 11043 11107 10820 10818 10828 10592 10025
    4047   1 2021-05-01 12413 12075 11336 11027 11086 10809 10803 10804 10585 10045
    4048   1 2021-05-01 12555 12173 11445 11136 11210 10928 10927 10891 10705 10260
    4049   1 2021-05-01 12360 12112 11411 11109 11179 10912 10910 10892 10687  9991
    4050   1 2021-05-01 12212 11922 11222 10928 10993 10745 10756 10711 10553  9855
    4051   1 2021-05-01 11984 11630 10929 10622 10684 10437 10447 10426 10232  9669
    4052   1 2021-05-01 11924 11520 10818 10523 10594 10352 10370 10324 10158  9649
    4053   1 2021-05-01 11792 11231 10538 10246 10311 10092 10099 10035  9906  9553
    4054   1 2021-05-01 11509 11014 10361 10088 10164  9948  9976  9915  9799  9310
    4055   1 2021-05-01 11117 10671 10052  9803  9883  9700  9745  9645  9577  8962
    4056   1 2021-05-01 11020 10610  9968  9702  9785  9600  9640  9574  9476  8874
    4057   1 2021-05-01 10967 10251  9654  9415  9504  9344  9393  9262  9236  8935
    4058   1 2021-05-01 10623  9984  9402  9171  9268  9122  9182  9026  9034  8657
    4059   1 2021-05-01 10204  9602  9027  8774  8856  8727  8787  8675  8636  8264
    4060   1 2021-05-01 10001  9675  9078  8815  8901  8753  8802  8690  8642  8043
    4061   1 2021-05-01 10119  9467  8889  8647  8746  8621  8673  8509  8528  8168
    4062   1 2021-05-01  9933  9467  8889  8647  8746  8621  8673  8509  8528  7995
    4063   1 2021-05-01  9213  8145  7753  7619  7755  7739  7870  7460  7787  7309
    4064   1 2021-05-01  7344  6881  6419  6215  6218  6152  6276  6085  6195  4839
    4065   1 2021-05-01  7240  6768  6294  6086  6091  6028  6153  5947  6067  4822
    4066   1 2021-05-01  7473  7180  6630  6372  6377  6280  6379  6216  6264  5097
    4067   1 2021-05-01  8015  7180  6630  6372  6377  6280  6379  6216  6264  5598
    4068   1 2021-05-01  8688  8152  7509  7194  7188  7030  7073  6984  6906  6209
    4069   1 2021-05-01  9162  8803  8112  7781  7782  7588  7616  7555  7439  6669
    4070   1 2021-05-01  9955  9317  8608  8270  8286  8073  8086  8027  7893  7541
    4071   1 2021-05-01 10213  9590  8874  8528  8556  8333  8348  8250  8146  7801
    4072   1 2021-05-01 10362  9909  9177  8826  8854  8626  8624  8540  8428  7859
    4073   1 2021-05-01 10593 10217  9485  9136  9154  8914  8920  8867  8715  8006
    4074   1 2021-05-01 10916 10435  9684  9330  9351  9094  9098  9054  8896  8289
    4075   1 2021-05-01 11076 10689  9931  9572  9590  9316  9308  9304  9088  8447
    4076   1 2021-05-01 11302 10958 10212  9874  9903  9643  9642  9584  9428  8694
    4077   1 2021-05-01 11504 11157 10401 10068 10100  9845  9842  9796  9632  8895
    4078   1 2021-05-01 11711 11227 10486 10162 10194  9934  9942  9897  9734  9026
    4079   1 2021-05-01 11740 11282 10556 10239 10269 10023 10035  9933  9835  9013
    4080   1 2021-05-01 11775 11410 10675 10357 10389 10141 10152 10054  9946  9032
    4081   1 2021-05-01 11875 11450 10693 10358 10383 10113 10112 10075  9900  9102
    4082   1 2021-05-01 12306 11875 11114 10777 10810 10521 10521 10511 10291  9636
    4083   1 2021-05-01 12291 11896 11145 10810 10851 10561 10555 10531 10334  9714
    4084   1 2021-05-01 12411 11949 11216 10906 10960 10687 10683 10596 10467  9836
    4085   1 2021-05-01 12546 12042 11317 11024 11085 10821 10822 10723 10613 10003
    4086   1 2021-05-01 12458 11827 11097 10798 10868 10610 10623 10477 10417  9994
    4087   1 2021-05-01 12235 11787 11070 10793 10848 10589 10613 10512 10400  9723
    4088   1 2021-05-01 12037 11632 10919 10623 10673 10411 10424 10372 10222  9487
    4089   1 2021-05-01 12105 11687 10959 10650 10708 10433 10434 10382 10216  9614
    4090   1 2021-05-01 12136 11653 10937 10617 10673 10405 10401 10360 10195  9712
    4091   1 2021-05-01 12200 11760 11024 10709 10774 10500 10497 10435 10280  9861
    4092   1 2021-05-01 12251 11863 11105 10788 10856 10571 10549 10518 10333  9948
    4093   1 2021-05-01 12385 12042 11287 10971 11029 10741 10715 10738 10491 10114
    4094   1 2021-05-01 12483 12005 11259 10952 11016 10738 10719 10687 10490 10251
    4095   1 2021-05-01 12383 11832 11103 10812 10894 10623 10611 10516 10404 10148
    4096   1 2021-05-01 12190 11739 11011 10722 10801 10538 10530 10436 10324  9940
    4097   1 2021-05-01 11917 11290 10577 10279 10361 10109 10120  9996  9919  9670
    4098   1 2021-05-01 11758 11250 10554 10260 10329 10081 10089 10001  9900  9490
    4099   1 2021-05-01 11647 11249 10568 10288 10360 10118 10143 10058  9941  9371
    4100   1 2021-05-01 11701 11309 10598 10290 10345 10080 10089 10137  9895  9437
    4101   1 2021-05-01 11923 11486 10787 10493 10557 10300 10305 10259 10094  9671
    4102   1 2021-05-01 11883 11445 10742 10447 10518 10256 10264 10197 10049  9640
    4103   1 2021-05-01 11909 11497 10785 10480 10557 10298 10301 10224 10092  9666
    4104   1 2021-05-01 11983 11585 10866 10569 10654 10395 10390 10295 10182  9756
    4105   1 2021-05-01 12109 11609 10874 10553 10620 10356 10354 10302 10134  9875
    4106   1 2021-05-01 12075 11644 10916 10610 10674 10414 10407 10388 10200  9815
    4107   1 2021-05-01 12083 11464 10796 10530 10599 10367 10383 10280 10188  9770
    4108   1 2021-05-01 11835 11496 10792 10496 10555 10303 10319 10274 10122  9478
    4109   1 2021-05-01 12073 11561 10830 10506 10566 10300 10289 10276 10080  9777
    4110   1 2021-05-01 12134 11611 10869 10544 10607 10338 10334 10278 10117  9799
    4111   1 2021-05-01 12262 11828 11101 10803 10875 10603 10610 10531 10388  9929
    4112   1 2021-05-01 12217 11860 11123 10813 10870 10606 10601 10584 10378  9953
    4113   1 2021-05-01 12350 11901 11169 10866 10924 10666 10655 10625 10432 10043
    4114   1 2021-05-01 12360 11964 11252 10954 11007 10752 10740 10733 10530 10074
    4115   1 2021-05-01 12251 11740 11047 10757 10816 10568 10576 10521 10362  9913
    4116   1 2021-05-01 12110 11554 10859 10563 10632 10392 10399 10301 10195  9762
    4117   1 2021-05-01 12016 11592 10888 10597 10677 10438 10439 10326 10238  9753
    4118   1 2021-05-01 11924 11397 10712 10430 10516 10296 10311 10169 10116  9680
    4119   1 2021-05-01 11691 11292 10616 10341 10425 10209 10228 10113 10041  9479
    4120   1 2021-05-01 11643 10843 10215  9968 10050  9850  9899  9773  9725  9455
    4121   1 2021-05-01 11025 10671 10052  9803  9883  9700  9745  9645  9577  8834
    4122   1 2021-05-01 11059 10602  9963  9695  9772  9589  9630  9562  9458  8918
    4123   1 2021-05-01 11128 10519  9885  9626  9723  9526  9567  9464  9406  9036
    4124   1 2021-05-01 10891 10356  9747  9501  9600  9420  9475  9338  9313  8913
    4125   1 2021-05-01 10455  9844  9250  9008  9101  8953  9012  8860  8863  8501
    4126   1 2021-05-01 10340  9975  9358  9092  9172  9015  9062  8933  8897  8376
    4127   1 2021-05-01 10207  9945  9308  9019  9103  8928  8967  9003  8801  8428
    4128   1 2021-05-01  7588  7512  6949  6686  6692  6572  6659  6529  6526  5186
    4129   1 2021-05-01  9124  8333  7712  7413  7438  7278  7337  7169  7176  6753
    4130   1 2021-05-01  9395  8982  8333  8049  8087  7905  7953  7785  7782  7023
    4131   1 2021-05-01  9751  9351  8651  8335  8354  8154  8169  8090  7981  7397
    4132   1 2021-05-01  9991  9751  9041  8714  8742  8522  8537  8451  8343  7596
    4133   1 2021-05-01 10478  9967  9255  8913  8949  8710  8715  8624  8522  8054
    4134   1 2021-05-01 10626 10201  9480  9138  9166  8918  8922  8854  8722  8086
    4135   1 2021-05-01 10801 10415  9696  9362  9386  9152  9167  9084  8973  8173
    4136   1 2021-05-01 10898 10415  9696  9362  9386  9152  9167  9084  8973  8225
    4137   1 2021-05-01 11327 10688  9926  9564  9590  9320  9311  9296  9101  8671
    4138   1 2021-05-01 11512 11014 10271  9927  9953  9691  9691  9673  9469  8854
    4139   1 2021-05-01 11628 11170 10430 10103 10137  9877  9885  9839  9677  8959
    4140   1 2021-05-01 11661 11196 10468 10153 10182  9930  9944  9882  9743  8957
    4141   1 2021-05-01 11763 11244 10518 10206 10237  9984  9994  9923  9795  8993
    4142   1 2021-05-01 11869 11410 10675 10357 10389 10141 10152 10054  9946  9064
    4143   1 2021-05-01 11952 11613 10860 10530 10567 10300 10306 10260 10086  9200
    4144   1 2021-05-01 12211 11905 11154 10827 10872 10592 10583 10543 10357  9499
    4145   1 2021-05-01 12364 11972 11238 10920 10973 10700 10691 10622 10470  9738
    4146   1 2021-05-01 12377 11978 11246 10940 10980 10708 10710 10693 10501  9838
    4147   1 2021-05-01 12336 11835 11112 10811 10853 10580 10591 10557 10379  9788
    4148   1 2021-05-01 12410 11782 11066 10783 10839 10570 10579 10522 10386  9897
    4149   1 2021-05-01 12179 11447 10744 10463 10505 10257 10265 10198 10071  9668
    4150   1 2021-05-01 11802 11548 10832 10542 10583 10328 10352 10292 10144  9231
    4151   1 2021-05-01 12055 11529 10803 10490 10528 10250 10256 10266 10041  9547
    4152   1 2021-05-01 12127 11739 11004 10680 10744 10467 10459 10414 10248  9691
    4153   1 2021-05-01 12353 11874 11133 10818 10883 10613 10605 10544 10394  9991
    4154   1 2021-05-01 12402 11949 11208 10901 10971 10684 10680 10635 10465 10151
    4155   1 2021-05-01 12437 12042 11287 10971 11029 10741 10715 10738 10491 10200
    4156   1 2021-05-01 12335 11903 11162 10856 10923 10647 10640 10622 10417 10086
    4157   1 2021-05-01 12264 11762 11032 10730 10805 10537 10527 10485 10305 10030
    4158   1 2021-05-01 11964 11495 10770 10466 10545 10284 10274 10202 10067  9718
    4159   1 2021-05-01 11950 11302 10612 10332 10407 10158 10173 10093  9978  9724
    4160   1 2021-05-01 11685 11240 10568 10295 10371 10126 10135 10062  9948  9424
    4161   1 2021-05-01 11602 10910 10249  9971 10031  9803  9824  9760  9623  9339
    4162   1 2021-05-01 11319 10989 10298  9999 10054  9816  9820  9780  9621  9028
    4163   1 2021-05-01 11663 11256 10542 10232 10278 10017 10013 10049  9818  9395
    4164   1 2021-05-01 11943 11425 10708 10401 10470 10212 10199 10168  9998  9757
    4165   1 2021-05-01 12051 11580 10868 10562 10638 10378 10372 10316 10155  9875
    4166   1 2021-05-01 12108 11651 10926 10618 10689 10426 10414 10379 10199  9904
    4167   1 2021-05-01 12147 11688 10960 10657 10726 10470 10453 10407 10238  9941
    4168   1 2021-05-01 12046 11644 10916 10610 10674 10414 10407 10388 10200  9797
    4169   1 2021-05-01 12068 11640 10949 10671 10737 10489 10484 10434 10291  9774
    4170   1 2021-05-01 12028 11561 10837 10530 10594 10340 10335 10276 10129  9754
    4171   1 2021-05-01 12190 11755 11033 10734 10799 10534 10538 10487 10320  9881
    4172   1 2021-05-01 12179 11805 11074 10771 10838 10569 10569 10539 10348  9823
    4173   1 2021-05-01 12249 11775 11038 10725 10786 10515 10517 10486 10288  9941
    4174   1 2021-05-01 12269 11904 11179 10882 10948 10689 10681 10628 10471 10009
    4175   1 2021-05-01 12302 11833 11125 10830 10883 10634 10632 10609 10417  9991
    4176   1 2021-05-01 12186 11681 10978 10688 10753 10510 10511 10449 10311  9841
    4177   1 2021-05-01 12045 11681 10978 10688 10753 10510 10511 10449 10311  9676
    4178   1 2021-05-01 12112 11692 10987 10692 10751 10506 10502 10440 10296  9751
    4179   1 2021-05-01 12207 11758 11037 10740 10804 10551 10542 10519 10333  9924
    4180   1 2021-05-01 12013 11634 10943 10666 10744 10506 10516 10445 10323  9767
    4181   1 2021-05-01 11829 11292 10616 10341 10425 10209 10228 10113 10041  9611
    4182   1 2021-05-01 11579 11027 10375 10114 10195  9978 10011  9900  9827  9394
    4183   1 2021-05-01 11145 10597  9972  9720  9795  9613  9660  9555  9490  8958
    4184   1 2021-05-01 11193 10719 10061  9781  9857  9658  9680  9614  9509  9054
    4185   1 2021-05-01 11072 10608  9957  9691  9775  9576  9614  9543  9443  8997
    4186   1 2021-05-01 10959 10520  9881  9618  9701  9516  9557  9490  9391  8955
    4187   1 2021-05-01 10778 10343  9711  9452  9545  9372  9413  9293  9252  8823
    4188   1 2021-05-01 10834 10371  9731  9457  9550  9367  9406  9341  9242  8860
    4189   1 2021-05-01 10658 10038  9438  9186  9278  9133  9190  9089  9032  8698
    4190   1 2021-05-01 10135  9741  9118  8835  8921  8767  8810  8758  8653  8296
    4191   1 2021-05-01 10393  9964  9338  9055  9150  8992  9019  8938  8863  8600
    4192   1 2021-05-01 10672 10079  9462  9189  9291  9132  9158  9114  9005  8894
    4193   1 2021-05-01  8407  8839  8208  7903  7927  7750  7782  7705  7619  6048
    4194   1 2021-05-01  9222  9292  8659  8367  8415  8223  8256  8114  8085  6876
    4195   1 2021-05-01 10114  9292  8659  8367  8415  8223  8256  8114  8085  7793
    4196   1 2021-05-01 10211  9451  8776  8488  8525  8325  8353  8216  8176  7881
    4197   1 2021-05-01 10346  9833  9148  8838  8874  8664  8677  8569  8496  7962
    4198   1 2021-05-01 10529 10043  9335  9010  9042  8808  8820  8728  8623  8094
    4199   1 2021-05-01 10904 10272  9554  9221  9255  9018  9015  8928  8826  8395
    4200   1 2021-05-01 10998 10428  9742  9434  9468  9235  9254  9145  9063  8384
    4201   1 2021-05-01 11092 10711 10012  9706  9747  9509  9537  9408  9332  8446
    4202   1 2021-05-01 11380 10968 10231  9900  9938  9681  9692  9636  9479  8788
    4203   1 2021-05-01 11613 11145 10420 10119 10155  9907  9917  9834  9719  9003
    4204   1 2021-05-01 11657 11203 10480 10164 10191  9946  9954  9900  9746  8994
    4205   1 2021-05-01 11680 11266 10544 10231 10261 10009 10028  9952  9819  8942
    4206   1 2021-05-01 11749 11397 10675 10361 10396 10131 10147 10072  9929  9024
    4207   1 2021-05-01 11868 11576 10842 10536 10575 10317 10330 10239 10122  9070
    4208   1 2021-05-01 12153 11828 11081 10763 10809 10528 10520 10459 10312  9517
    4209   1 2021-05-01 12351 11836 11076 10738 10777 10488 10477 10469 10252  9791
    4210   1 2021-05-01 12449 11959 11204 10880 10923 10642 10626 10579 10397  9933
    4211   1 2021-05-01 12476 11901 11178 10878 10936 10662 10666 10574 10458  9998
    4212   1 2021-05-01 12247 11837 11123 10833 10896 10628 10639 10543 10436  9732
    4213   1 2021-05-01 12096 11715 11005 10716 10767 10497 10511 10443 10307  9570
    4214   1 2021-05-01 12009 11527 10826 10541 10597 10333 10345 10235 10144  9526
    4215   1 2021-05-01 11768 11393 10677 10367 10411 10151 10153 10092  9940  9226
    4216   1 2021-05-01 12066 11700 10953 10624 10681 10408 10392 10336 10175  9665
    4217   1 2021-05-01 12270 11809 11063 10742 10798 10526 10510 10473 10294  9927
    4218   1 2021-05-01 12362 11965 11214 10900 10969 10682 10668 10646 10440 10051
    4219   1 2021-05-01 12425 11955 11220 10912 10980 10704 10690 10655 10467 10167
    4220   1 2021-05-01 12257 11805 11077 10772 10838 10561 10557 10529 10344 10007
    4221   1 2021-05-01 12147 11635 10903 10588 10654 10376 10383 10334 10169  9869
    4222   1 2021-05-01 12107 11683 10939 10625 10688 10427 10423 10380 10201  9845
    4223   1 2021-05-01 12043 11616 10879 10568 10634 10368 10359 10314 10143  9820
    4224   1 2021-05-01 12037 11450 10763 10485 10568 10315 10314 10230 10119  9812
    4225   1 2021-05-01 11643 11259 10580 10303 10378 10123 10133 10078  9937  9438
    4226   1 2021-05-01 11641 11138 10457 10181 10251 10007 10019  9907  9814  9401
    4227   1 2021-05-01 11743 11251 10559 10262 10335 10086 10088  9986  9885  9521
    4228   1 2021-05-01 11724 11424 10709 10402 10470 10218 10205 10170 10005  9477
    4229   1 2021-05-01 12023 11600 10882 10577 10667 10403 10397 10350 10196  9884
    4230   1 2021-05-01 12056 11621 10898 10593 10668 10396 10384 10353 10174  9902
    4231   1 2021-05-01 12123 11670 10939 10629 10699 10438 10427 10393 10209  9941
    4232   1 2021-05-01 12143 11681 10963 10665 10730 10472 10462 10431 10256  9927
    4233   1 2021-05-01 12082 11655 10940 10646 10718 10458 10450 10420 10243  9855
    4234   1 2021-05-01 12094 11577 10898 10633 10709 10462 10479 10406 10294  9848
    4235   1 2021-05-01 11892 11542 10821 10522 10576 10316 10329 10319 10123  9619
    4236   1 2021-05-01 11958 11542 10821 10522 10576 10316 10329 10319 10123  9660
    4237   1 2021-05-01 12072 11628 10911 10601 10667 10407 10407 10377 10206  9784
    4238   1 2021-05-01 12108 11706 10979 10664 10727 10469 10465 10423 10247  9803
    4239   1 2021-05-01 12045 11804 11073 10756 10811 10548 10539 10555 10323  9705
    4240   1 2021-05-01 12138 11567 10846 10525 10576 10325 10325 10314 10101  9799
    4241   1 2021-05-01 11973 11605 10901 10600 10653 10409 10413 10374 10203  9607
    4242   1 2021-05-01 11928 11490 10782 10474 10536 10291 10292 10227 10086  9558
    4243   1 2021-05-01 11979 11598 10874 10554 10612 10354 10349 10343 10142  9635
    4244   1 2021-05-01 12063 11611 10915 10630 10705 10456 10462 10426 10268  9771
    4245   1 2021-05-01 11951 11472 10779 10479 10548 10321 10326 10264 10134  9668
    4246   1 2021-05-01 11828 11246 10572 10293 10367 10145 10171 10059  9978  9597
    4247   1 2021-05-01 11634 11170 10497 10221 10292 10077 10101  9987  9910  9435
    4248   1 2021-05-01 11602 10832 10187  9904  9989  9790  9821  9690  9646  9411
    4249   1 2021-05-01 11352 10832 10187  9904  9989  9790  9821  9690  9646  9212
    4250   1 2021-05-01 10970 10649  9998  9724  9801  9608  9641  9578  9460  8871
    4251   1 2021-05-01 10898 10523  9883  9614  9691  9510  9547  9503  9371  8856
    4252   1 2021-05-01 10841 10488  9849  9582  9661  9486  9515  9470  9346  8809
    4253   1 2021-05-01 10759 10321  9700  9436  9518  9350  9392  9342  9219  8781
    4254   1 2021-05-01 10055  9933  9348  9105  9189  9050  9115  9042  8965  8109
    4255   1 2021-05-01 10188  9931  9316  9055  9144  8998  9037  8980  8887  8350
    4256   1 2021-05-01 10464 10279  9631  9351  9442  9273  9285  9226  9123  8594
    4257   1 2021-05-01 10677 10481  9831  9558  9655  9487  9502  9457  9329  8854
    4258   1 2021-05-01 10933 10290  9653  9383  9484  9314  9340  9266  9173  9163
    4259   1 2021-05-01 10885 10456  9820  9552  9655  9482  9511  9458  9346  9147
    4260   1 2021-05-01 10939 10548  9906  9638  9739  9572  9598  9574  9421  9197
    4261   1 2021-05-01  9669  9197  8557  8253  8285  8097  8130  8039  7964  7296
    4262   1 2021-05-01 10096  9511  8873  8573  8619  8422  8453  8337  8278  7750
    4263   1 2021-05-01 10243  9866  9215  8916  8965  8759  8782  8649  8617  7912
    4264   1 2021-05-01 10574  9974  9295  8994  9030  8817  8835  8724  8643  8189
    4265   1 2021-05-01 10645 10274  9572  9261  9303  9071  9084  8971  8883  8227
    4266   1 2021-05-01 10957 10464  9741  9422  9462  9229  9232  9127  9040  8529
    4267   1 2021-05-01 11095 10910 10190  9888  9937  9701  9689  9594  9492  8653
    4268   1 2021-05-01 11405 10925 10224  9934  9979  9737  9746  9642  9539  8914
    4269   1 2021-05-01 11482 11158 10447 10150 10198  9957  9962  9870  9766  8925
    4270   1 2021-05-01 11645 11054 10333 10024 10065  9818  9833  9749  9640  9084
    4271   1 2021-05-01 11680 11145 10420 10119 10155  9907  9917  9834  9719  9108
    4272   1 2021-05-01 11662 11188 10473 10172 10212  9974  9987  9897  9788  9015
    4273   1 2021-05-01 11765 11253 10530 10221 10253  9996 10016  9945  9807  9028
    4274   1 2021-05-01 11907 11435 10697 10377 10416 10145 10161 10090  9956  9189
    4275   1 2021-05-01 11981 11596 10871 10574 10609 10343 10362 10287 10162  9215
    4276   1 2021-05-01 12219 11852 11106 10795 10824 10547 10547 10519 10337  9511
    4277   1 2021-05-01 12549 12064 11312 10992 11037 10754 10747 10693 10527  9990
    4278   1 2021-05-01 12655 12201 11446 11125 11173 10885 10873 10865 10654 10134
    4279   1 2021-05-01 12632 12032 11317 11034 11107 10831 10832 10719 10630 10165
    4280   1 2021-05-01 12475 11731 11040 10757 10813 10553 10569 10452 10361 10000
    4281   1 2021-05-01 12075 11667 10955 10662 10714 10448 10463 10386 10260  9556
    4282   1 2021-05-01 12209 11815 11095 10794 10855 10584 10575 10488 10372  9771
    4283   1 2021-05-01 12089 11832 11072 10741 10800 10518 10508 10469 10288  9701
    4284   1 2021-05-01 12298 11700 10953 10624 10681 10408 10392 10336 10175  9983
    4285   1 2021-05-01 12453 11924 11182 10865 10932 10656 10648 10579 10419 10180
    4286   1 2021-05-01 12429 11937 11194 10897 10962 10691 10675 10613 10468 10178
    4287   1 2021-05-01 12379 11931 11193 10896 10963 10687 10670 10623 10453 10143
    4288   1 2021-05-01 12286 11773 11037 10723 10794 10525 10512 10459 10294 10055
    4289   1 2021-05-01 12191 11705 10967 10657 10735 10458 10446 10380 10244  9959
    4290   1 2021-05-01 12059 11619 10892 10585 10652 10386 10389 10345 10177  9818
    4291   1 2021-05-01 11882 11373 10680 10408 10478 10236 10255 10165 10054  9661
    4292   1 2021-05-01 11634 11166 10466 10173 10234  9978  9992  9967  9792  9402
    4293   1 2021-05-01 11586 11308 10586 10267 10329 10065 10062 10040  9849  9391
    4294   1 2021-05-01 11838 11423 10711 10412 10480 10224 10224 10173 10006  9649
    4295   1 2021-05-01 11864 11364 10664 10363 10429 10170 10169 10143  9962  9643
    4296   1 2021-05-01 11904 11551 10839 10538 10610 10346 10329 10341 10113  9702
    4297   1 2021-05-01 12078 11600 10882 10577 10667 10403 10397 10350 10196  9914
    4298   1 2021-05-01 12044 11664 10946 10653 10728 10465 10458 10426 10253  9876
    4299   1 2021-05-01 12067 11667 10945 10645 10716 10454 10444 10405 10231  9894
    4300   1 2021-05-01 12034 11604 10882 10581 10647 10388 10383 10357 10168  9825
    4301   1 2021-05-01 11942 11613 10894 10587 10655 10391 10382 10370 10164  9759
    4302   1 2021-05-01 11843 11508 10812 10522 10587 10327 10342 10320 10131  9572
    4303   1 2021-05-01 11709 11315 10634 10352 10417 10174 10199 10123 10008  9403
    4304   1 2021-05-01 11759 11339 10630 10324 10379 10124 10134 10099  9936  9437
    4305   1 2021-05-01 11983 11687 10975 10679 10743 10488 10485 10436 10277  9700
    4306   1 2021-05-01 12087 11522 10822 10514 10583 10333 10336 10245 10131  9808
    4307   1 2021-05-01 11930 11472 10750 10424 10489 10231 10237 10173 10022  9628
    4308   1 2021-05-01 11915 11497 10806 10509 10567 10322 10333 10270 10123  9560
    4309   1 2021-05-01 11871 11457 10753 10448 10504 10270 10282 10204 10081  9476
    4310   1 2021-05-01 11914 11531 10812 10498 10551 10304 10303 10252 10095  9533
    4311   1 2021-05-01 12025 11531 10812 10498 10551 10304 10303 10252 10095  9694
    4312   1 2021-05-01 12076 11597 10899 10601 10667 10420 10429 10379 10229  9753
    4313   1 2021-05-01 11931 11520 10828 10535 10605 10367 10384 10311 10184  9649
    4314   1 2021-05-01 11818 11367 10678 10375 10439 10207 10219 10170 10016  9547
    4315   1 2021-05-01 11637 11217 10527 10226 10284 10072 10071 10034  9867  9404
    4316   1 2021-05-01 11534 11094 10423 10147 10223 10014 10039  9937  9849  9326
    4317   1 2021-05-01 11361 10920 10255  9975 10049  9841  9863  9786  9681  9213
    4318   1 2021-05-01 11149 10586  9958  9693  9774  9592  9630  9531  9447  9016
    4319   1 2021-05-01 11075 10587  9941  9667  9753  9562  9593  9501  9415  8979
    4320   1 2021-05-01 10917 10381  9767  9507  9590  9419  9455  9366  9278  8862
    4321   1 2021-05-01 10583 10248  9650  9393  9472  9309  9367  9244  9197  8560
    4322   1 2021-05-01 10311  9887  9334  9106  9198  9059  9125  8941  8964  8329
    4323   1 2021-05-01 10462 10003  9384  9112  9204  9050  9092  8985  8922  8579
    4324   1 2021-05-01 10678 10212  9589  9325  9426  9256  9280  9201  9115  8823
    4325   1 2021-05-01 10708 10306  9663  9379  9471  9304  9331  9302  9157  8871
    4326   1 2021-05-01 10775 10398  9759  9485  9587  9420  9442  9420  9273  8968
    4327   1 2021-05-01 10859 10494  9855  9582  9683  9516  9537  9508  9375  9084
    4328   1 2021-05-01 10881 10495  9870  9600  9710  9539  9565  9507  9392  9134
    4329   1 2021-05-01 10884 10418  9802  9542  9654  9496  9523  9467  9356  9134
    4330   1 2021-05-01 10844 10428  9794  9526  9637  9466  9486  9434  9322  9086
    4331   1 2021-05-01 10829 10293  9686  9419  9523  9372  9402  9379  9240  9105
    4332   1 2021-05-01 10507 10092  9431  9133  9179  8966  8986  8880  8801  8136
    4333   1 2021-05-01 10671 10399  9722  9408  9456  9233  9241  9150  9050  8317
    4334   1 2021-05-01 10760 10194  9521  9208  9257  9042  9058  8936  8873  8381
    4335   1 2021-05-01 11319 10551  9849  9529  9578  9343  9351  9246  9161  8953
    4336   1 2021-05-01 11485 10758 10024  9709  9758  9511  9506  9428  9308  9100
    4337   1 2021-05-01 11638 11165 10435 10127 10177  9920  9908  9855  9708  9286
    4338   1 2021-05-01 11607 11158 10434 10129 10184  9934  9928  9840  9726  9214
    4339   1 2021-05-01 11888 11158 10447 10150 10198  9957  9962  9870  9766  9466
    4340   1 2021-05-01 11795 11287 10587 10306 10357 10112 10132 10019  9936  9242
    4341   1 2021-05-01 11722 11217 10511 10225 10269 10026 10049  9935  9859  9121
    4342   1 2021-05-01 11845 11397 10692 10409 10459 10207 10230 10111 10031  9202
    4343   1 2021-05-01 11870 11429 10713 10406 10449 10194 10214 10109 10010  9195
    4344   1 2021-05-01 12046 11591 10862 10559 10614 10353 10363 10216 10158  9472
    4345   1 2021-05-01 11962 11601 10863 10538 10568 10294 10306 10270 10095  9302
    4346   1 2021-05-01 12074 11849 11113 10799 10829 10560 10564 10538 10345  9400
    4347   1 2021-05-01 12423 12008 11251 10932 10979 10686 10674 10664 10461  9776
    4348   1 2021-05-01 12429 12008 11251 10932 10979 10686 10674 10664 10461  9925
    4349   1 2021-05-01 12359 12064 11330 11018 11068 10783 10782 10753 10567  9900
    4350   1 2021-05-01 12240 11800 11098 10802 10856 10590 10595 10515 10391  9743
    4351   1 2021-05-01 12134 11684 10978 10690 10738 10478 10498 10414 10291  9584
    4352   1 2021-05-01 12401 11815 11095 10794 10855 10584 10575 10488 10372 10030
    4353   1 2021-05-01 12563 11965 11224 10922 10995 10719 10714 10617 10514 10233
    4354   1 2021-05-01 12518 11973 11235 10918 10977 10699 10685 10646 10470 10241
    4355   1 2021-05-01 12435 11997 11273 10979 11041 10773 10768 10708 10554 10155
    4356   1 2021-05-01 12408 11923 11180 10879 10935 10671 10652 10625 10442 10116
    4357   1 2021-05-01 12242 11811 11078 10779 10837 10563 10548 10531 10332  9999
    4358   1 2021-05-01 12263 11757 11017 10714 10782 10510 10490 10454 10275 10027
    4359   1 2021-05-01 12201 11567 10852 10548 10614 10340 10345 10316 10131  9986
    4360   1 2021-05-01 12051 11389 10713 10441 10515 10269 10288 10195 10096  9811
    4361   1 2021-05-01 11684 11389 10713 10441 10515 10269 10288 10195 10096  9436
    4362   1 2021-05-01 11597 11111 10419 10137 10210  9958  9984  9889  9788  9374
    4363   1 2021-05-01 11793 11295 10573 10262 10335 10069 10070 10021  9867  9587
    4364   1 2021-05-01 11847 11391 10676 10362 10435 10161 10159 10143  9957  9668
    4365   1 2021-05-01 11786 11364 10664 10363 10429 10170 10169 10143  9962  9584
    4366   1 2021-05-01 11784 11379 10676 10367 10429 10172 10165 10152  9960  9614
    4367   1 2021-05-01 11792 11535 10820 10521 10595 10330 10316 10321 10106  9604
    4368   1 2021-05-01 11898 11517 10789 10479 10554 10281 10274 10275 10061  9702
    4369   1 2021-05-01 12098 11612 10901 10607 10686 10428 10422 10365 10215  9918
    4370   1 2021-05-01 12025 11581 10869 10577 10656 10398 10391 10338 10195  9846
    4371   1 2021-05-01 11940 11391 10685 10388 10456 10202 10214 10147  9995  9760
    4372   1 2021-05-01 11851 11367 10693 10408 10467 10219 10234 10210 10028  9604
    4373   1 2021-05-01 11803 11324 10631 10344 10409 10174 10189 10091  9986  9503
    4374   1 2021-05-01 11809 11386 10678 10378 10443 10195 10199 10146  9986  9542
    4375   1 2021-05-01 11915 11564 10843 10532 10586 10326 10322 10327 10111  9648
    4376   1 2021-05-01 12007 11548 10842 10538 10604 10343 10350 10309 10147  9733
    4377   1 2021-05-01 11973 11498 10778 10458 10520 10263 10274 10211 10069  9676
    4378   1 2021-05-01 11920 11497 10806 10509 10567 10322 10333 10270 10123  9604
    4379   1 2021-05-01 11889 11440 10752 10468 10529 10293 10317 10224 10120  9497
    4380   1 2021-05-01 11912 11484 10761 10436 10484 10227 10240 10197 10025  9561
    4381   1 2021-05-01 12013 11605 10897 10589 10650 10395 10401 10363 10191  9679
    4382   1 2021-05-01 11869 11530 10844 10560 10618 10384 10407 10332 10203  9533
    4383   1 2021-05-01 11740 11292 10597 10298 10352 10125 10143 10069  9940  9419
    4384   1 2021-05-01 11701 11212 10520 10227 10298 10070 10082  9992  9881  9435
    4385   1 2021-05-01 11710 11172 10494 10205 10283 10059 10087  9976  9885  9458
    4386   1 2021-05-01 11481 10990 10316 10029 10097  9881  9902  9822  9704  9282
    4387   1 2021-05-01 11385 11025 10341 10048 10124  9905  9928  9850  9728  9229
    4388   1 2021-05-01 11440 10832 10177  9896  9975  9775  9800  9704  9607  9292
    4389   1 2021-05-01 11330 10883 10225  9949 10033  9833  9862  9743  9672  9192
    4390   1 2021-05-01 11130 10634  9991  9724  9813  9619  9652  9533  9469  9057
    4391   1 2021-05-01 10856 10248  9650  9393  9472  9309  9367  9244  9197  8804
    4392   1 2021-05-01 10969 10374  9741  9472  9548  9376  9426  9307  9240  8926
    4393   1 2021-05-01 10844 10213  9602  9338  9429  9261  9311  9202  9142  8854
    4394   1 2021-05-01 10778 10385  9735  9454  9551  9366  9395  9335  9224  8868
    4395   1 2021-05-01 10825 10441  9806  9535  9634  9445  9469  9435  9303  8943
    4396   1 2021-05-01 10920 10410  9784  9518  9618  9441  9467  9408  9309  9078
    4397   1 2021-05-01 10854 10557  9912  9643  9743  9563  9575  9509  9409  9043
    4398   1 2021-05-01 10913 10581  9944  9671  9769  9600  9622  9563  9438  9133
    4399   1 2021-05-01 10938 10581  9944  9671  9769  9600  9622  9563  9438  9180
    4400   1 2021-05-01 11024 10500  9884  9615  9725  9561  9578  9511  9411  9251
    4401   1 2021-05-01 10921 10428  9794  9526  9637  9466  9486  9434  9322  9155
    4402   1 2021-05-01 10669 10308  9690  9423  9528  9382  9402  9340  9236  8929
    4403   1 2021-05-01 10685 10256  9629  9349  9453  9294  9312  9292  9153  8940
    4404   1 2021-05-01 10840 10210  9539  9224  9273  9038  9054  8987  8861  8461
    4405   1 2021-05-01 10941 10460  9770  9454  9502  9266  9280  9190  9085  8569
    4406   1 2021-05-01 11183 10701 10009  9690  9742  9508  9517  9432  9330  8804
    4407   1 2021-05-01 11391 11030 10319 10008 10079  9835  9833  9720  9639  9029
    4408   1 2021-05-01 11804 11275 10536 10213 10270 10004  9997  9949  9783  9495
    4409   1 2021-05-01 11991 11413 10693 10389 10463 10203 10202 10102 10000  9675
    4410   1 2021-05-01 12137 11589 10873 10581 10655 10398 10403 10293 10207  9807
    4411   1 2021-05-01 12066 11441 10731 10444 10506 10253 10264 10157 10053  9694
    4412   1 2021-05-01 11941 11497 10803 10520 10571 10321 10343 10245 10144  9454
    4413   1 2021-05-01 12072 11323 10623 10345 10395 10151 10184 10054  9990  9485
    4414   1 2021-05-01 12136 11553 10837 10535 10586 10329 10340 10250 10146  9609
    4415   1 2021-05-01 12147 11601 10874 10576 10627 10361 10379 10269 10162  9614
    4416   1 2021-05-01 12251 11893 11153 10840 10896 10615 10612 10544 10392  9707
    4417   1 2021-05-01 12523 11712 10994 10702 10777 10511 10527 10345 10335 10057
    4418   1 2021-05-01 12324 11829 11103 10802 10865 10589 10591 10482 10389  9772
    4419   1 2021-05-01 12421 11978 11237 10935 10990 10716 10717 10630 10502  9860
    4420   1 2021-05-01 12423 12097 11358 11068 11133 10859 10869 10754 10652  9996
    4421   1 2021-05-01 12484 11928 11211 10923 10989 10715 10724 10619 10516 10035
    4422   1 2021-05-01 12163 11658 10962 10680 10727 10469 10488 10399 10282  9654
    4423   1 2021-05-01 12090 11641 10909 10610 10644 10377 10384 10368 10175  9564
    4424   1 2021-05-01 12054 12016 11260 10943 10995 10706 10697 10711 10474  9533
    4425   1 2021-05-01 12218 11808 11079 10760 10810 10523 10511 10534 10287  9847
    4426   1 2021-05-01 12234 11852 11108 10792 10836 10548 10530 10581 10316  9883
    4427   1 2021-05-01 12281 11852 11108 10792 10836 10548 10530 10581 10316  9968
    4428   1 2021-05-01 12137 11848 11126 10828 10881 10615 10608 10579 10389  9820
    4429   1 2021-05-01 12195 11739 11010 10704 10769 10497 10485 10462 10262  9930
    4430   1 2021-05-01 12088 11704 10976 10678 10744 10481 10470 10431 10256  9848
    4431   1 2021-05-01 11952 11528 10812 10508 10577 10317 10310 10264 10104  9727
    4432   1 2021-05-01 11879 11388 10705 10432 10497 10244 10270 10205 10067  9641
    4433   1 2021-05-01 11665 11156 10488 10221 10297 10055 10078  9945  9889  9428
    4434   1 2021-05-01 11706 11190 10476 10173 10242  9981  9981  9949  9775  9490
    4435   1 2021-05-01 11913 11412 10699 10403 10483 10231 10233 10141 10032  9741
    4436   1 2021-05-01 12014 11533 10838 10554 10633 10384 10388 10281 10183  9830
    4437   1 2021-05-01 11911 11429 10725 10434 10509 10253 10249 10170 10050  9724
    4438   1 2021-05-01 11954 11383 10699 10429 10507 10267 10283 10164 10090  9774
    4439   1 2021-05-01 11743 11416 10694 10382 10459 10189 10183 10166  9977  9515
    4440   1 2021-05-01 11916 11515 10801 10497 10564 10305 10299 10307 10084  9715
    4441   1 2021-05-01 12061 11635 10924 10638 10711 10459 10455 10429 10245  9867
    4442   1 2021-05-01 11918 11498 10793 10506 10573 10323 10325 10288 10119  9739
    4443   1 2021-05-01 11816 11394 10676 10372 10443 10189 10188 10134  9986  9618
    4444   1 2021-05-01 11738 11303 10609 10316 10374 10124 10133 10088  9930  9446
    4445   1 2021-05-01 11549 11266 10583 10299 10353 10115 10144 10108  9943  9203
    4446   1 2021-05-01 11727 11263 10567 10269 10318 10073 10082 10056  9874  9399
    4447   1 2021-05-01 11702 11322 10627 10336 10390 10145 10149 10119  9944  9422
    4448   1 2021-05-01 11740 11376 10653 10341 10401 10140 10150 10111  9948  9439
    4449   1 2021-05-01 11782 11390 10663 10339 10410 10140 10137 10090  9943  9504
    4450   1 2021-05-01 11945 11527 10829 10529 10599 10346 10351 10288 10158  9657
    4451   1 2021-05-01 11929 11481 10773 10463 10528 10277 10288 10226 10084  9593
    4452   1 2021-05-01 11950 11516 10813 10505 10565 10318 10329 10281 10120  9617
    4453   1 2021-05-01 11900 11306 10637 10349 10412 10175 10200 10119  9997  9570
    4454   1 2021-05-01 11777 11399 10721 10428 10481 10248 10263 10208 10062  9418
    4455   1 2021-05-01 11760 11242 10557 10255 10312 10079 10100 10029  9901  9462
    4456   1 2021-05-01 11743 11323 10635 10331 10403 10160 10183 10115  9977  9488
    4457   1 2021-05-01 11734 11273 10591 10300 10373 10138 10163 10091  9961  9496
    4458   1 2021-05-01 11587 11110 10430 10139 10210  9990 10013  9920  9817  9365
    4459   1 2021-05-01 11593 11137 10446 10151 10224  9990 10010  9945  9812  9391
    4460   1 2021-05-01 11590 11106 10430 10149 10222 10002 10026  9958  9828  9426
    4461   1 2021-05-01 11506 10929 10272  9993 10072  9869  9903  9803  9703  9396
    4462   1 2021-05-01 11268 10733 10103  9824  9897  9706  9750  9683  9561  9140
    4463   1 2021-05-01 11080 10609  9965  9685  9767  9581  9610  9523  9427  8950
    4464   1 2021-05-01 11158 10659 10018  9738  9825  9634  9668  9581  9488  9117
    4465   1 2021-05-01 10970 10341  9745  9487  9592  9426  9475  9351  9305  8951
    4466   1 2021-05-01 10600 10149  9554  9297  9391  9235  9294  9154  9128  8615
    4467   1 2021-05-01 10484 10247  9611  9330  9400  9220  9257  9220  9078  8473
    4468   1 2021-05-01 10842 10247  9611  9330  9400  9220  9257  9220  9078  8900
    4469   1 2021-05-01 10893 10446  9803  9533  9632  9451  9489  9414  9314  8999
    4470   1 2021-05-01 11057 10556  9911  9637  9742  9551  9577  9497  9401  9153
    4471   1 2021-05-01 11028 10577  9935  9667  9770  9579  9603  9535  9434  9181
    4472   1 2021-05-01 11287 10817 10155  9883  9986  9795  9809  9725  9626  9469
    4473   1 2021-05-01 11208 10774 10126  9847  9951  9766  9781  9717  9604  9418
    4474   1 2021-05-01 11301 10653 10022  9748  9854  9684  9699  9600  9523  9518
    4475   1 2021-05-01 11225 10745 10114  9844  9958  9783  9798  9718  9624  9419
    4476   1 2021-05-01 11070 10543  9912  9654  9766  9609  9625  9520  9455  9311
    4477   1 2021-05-01 11052 10585  9948  9676  9795  9622  9645  9577  9476  9327
    4478   1 2021-05-01 11002 10415  9797  9535  9661  9494  9527  9426  9371  9265
    4479   1 2021-05-01 10847 10355  9749  9498  9617  9463  9500  9409  9339  9144
    4480   1 2021-05-01 10620 10070  9488  9240  9353  9228  9266  9156  9108  8940
    4481   1 2021-05-01 10641 10038  9379  9105  9154  8962  9001  8851  8838  8315
    4482   1 2021-05-01 10941 10644  9953  9637  9702  9460  9478  9358  9291  8584
    4483   1 2021-05-01 11345 11161 10461 10155 10228  9976  9986  9876  9791  8996
    4484   1 2021-05-01 11736 11287 10584 10282 10361 10109 10117  9998  9925  9431
    4485   1 2021-05-01 11850 11030 10319 10008 10079  9835  9833  9720  9639  9570
    4486   1 2021-05-01 11956 11427 10708 10387 10468 10205 10198 10101  9987  9680
    4487   1 2021-05-01 12206 11604 10878 10566 10637 10368 10355 10288 10143  9941
    4488   1 2021-05-01 12292 11743 11033 10737 10810 10541 10539 10456 10336  9955
    4489   1 2021-05-01 12113 11553 10845 10550 10618 10366 10372 10269 10172  9712
    4490   1 2021-05-01 12010 11570 10843 10528 10572 10311 10313 10277 10104  9494
    4491   1 2021-05-01 12182 11772 11062 10774 10825 10568 10586 10490 10392  9628
    4492   1 2021-05-01 12500 11996 11259 10965 11009 10739 10736 10681 10534 10002
    4493   1 2021-05-01 12509 12026 11282 10980 11044 10770 10767 10659 10567 10067
    4494   1 2021-05-01 12667 12189 11455 11158 11227 10944 10940 10867 10732 10214
    4495   1 2021-05-01 12610 12088 11335 11032 11100 10817 10812 10721 10598 10161
    4496   1 2021-05-01 12626 12178 11429 11126 11183 10906 10907 10846 10688 10131
    4497   1 2021-05-01 12648 12172 11439 11151 11214 10936 10932 10872 10719 10158
    4498   1 2021-05-01 12615 12035 11314 11027 11090 10826 10822 10739 10604 10162
    4499   1 2021-05-01 12499 12035 11314 11027 11090 10826 10822 10739 10604 10073
    4500   1 2021-05-01 12038 11678 10973 10680 10735 10474 10487 10395 10282  9540
    4501   1 2021-05-01 11991 11451 10764 10483 10543 10292 10318 10192 10112  9506
    4502   1 2021-05-01 11850 11560 10810 10486 10525 10247 10251 10257 10030  9384
    4503   1 2021-05-01 12016 11516 10784 10456 10509 10239 10237 10180 10015  9629
    4504   1 2021-05-01 11950 11468 10743 10432 10480 10208 10203 10169  9998  9588
    4505   1 2021-05-01 11908 11386 10659 10350 10409 10145 10143 10054  9943  9631
    4506   1 2021-05-01 11893 11564 10829 10504 10556 10280 10266 10283 10040  9571
    4507   1 2021-05-01 12008 11470 10745 10441 10502 10232 10229 10209 10020  9712
    4508   1 2021-05-01 11927 11464 10750 10453 10516 10261 10254 10217 10048  9652
    4509   1 2021-05-01 11786 11331 10614 10308 10370 10107 10099 10077  9889  9573
    4510   1 2021-05-01 11800 11306 10602 10316 10369 10112 10117 10096  9923  9575
    4511   1 2021-05-01 11736 11227 10533 10248 10326 10083 10083  9982  9887  9543
    4512   1 2021-05-01 11813 11227 10533 10248 10326 10083 10083  9982  9887  9648
    4513   1 2021-05-01 11963 11482 10760 10460 10526 10274 10269 10227 10067  9815
    4514   1 2021-05-01 11944 11555 10844 10551 10619 10372 10372 10343 10160  9771
    4515   1 2021-05-01 11903 11612 10910 10625 10694 10440 10434 10385 10233  9737
    4516   1 2021-05-01 12011 11528 10831 10542 10618 10364 10364 10295 10168  9856
    4517   1 2021-05-01 11751 11289 10590 10310 10387 10141 10155 10065  9969  9532
    4518   1 2021-05-01 11829 11357 10656 10372 10441 10190 10194 10143  9998  9611
    4519   1 2021-05-01 11711 11379 10674 10386 10443 10198 10198 10179  9994  9473
    4520   1 2021-05-01 11771 11293 10572 10270 10326 10076 10063 10043  9858  9577
    4521   1 2021-05-01 11845 11333 10628 10329 10386 10134 10133 10095  9923  9648
    4522   1 2021-05-01 11697 11227 10569 10306 10364 10134 10166 10093  9975  9421
    4523   1 2021-05-01 11519 11060 10388 10099 10146  9915  9943  9881  9752  9195
    4524   1 2021-05-01 11530 11173 10474 10180 10224  9982  9988  9948  9780  9218
    4525   1 2021-05-01 11624 11207 10506 10208 10263 10019 10035  9980  9815  9333
    4526   1 2021-05-01 11701 11235 10537 10235 10297 10054 10062  9995  9856  9417
    4527   1 2021-05-01 11856 11369 10642 10322 10386 10128 10120 10092  9915  9580
    4528   1 2021-05-01 12052 11569 10862 10561 10641 10380 10386 10320 10193  9786
    4529   1 2021-05-01 11940 11448 10751 10458 10526 10286 10305 10226 10108  9647
    4530   1 2021-05-01 11864 11468 10774 10474 10529 10287 10299 10249 10093  9548
    4531   1 2021-05-01 11774 11334 10663 10385 10450 10222 10243 10156 10043  9441
    4532   1 2021-05-01 11690 11227 10552 10262 10324 10087 10115 10038  9913  9348
    4533   1 2021-05-01 11650 11209 10523 10228 10286 10052 10083 10029  9885  9325
    4534   1 2021-05-01 11576 11234 10544 10253 10306 10081 10098 10068  9895  9276
    4535   1 2021-05-01 11667 11182 10501 10206 10265 10042 10061 10015  9861  9378
    4536   1 2021-05-01 11684 11261 10576 10283 10343 10114 10127 10062  9922  9475
    4537   1 2021-05-01 11727 11169 10487 10199 10265 10037 10063 10000  9847  9548
    4538   1 2021-05-01 11643 11105 10445 10167 10231 10020 10054  9973  9855  9486
    4539   1 2021-05-01 11402 11054 10396 10111 10182  9974 10004  9934  9804  9243
    4540   1 2021-05-01 11367 10798 10162  9881  9963  9760  9795  9691  9606  9214
    4541   1 2021-05-01 11284 10719 10066  9773  9849  9648  9679  9621  9498  9179
    4542   1 2021-05-01 11290 10843 10195  9915  9994  9794  9822  9748  9639  9220
    4543   1 2021-05-01 10936 10540  9923  9650  9751  9575  9619  9518  9453  8900
    4544   1 2021-05-01 10911 10394  9781  9516  9606  9438  9485  9367  9317  8856
    4545   1 2021-05-01 10743 10329  9683  9401  9475  9293  9342  9262  9163  8732
    4546   1 2021-05-01 10928 10696 10038  9744  9823  9621  9656  9613  9470  8934
    4547   1 2021-05-01 11306 10793 10142  9860  9951  9763  9789  9706  9611  9371
    4548   1 2021-05-01 11355 10941 10283 10002 10087  9905  9928  9874  9753  9456
    4549   1 2021-05-01 11396 11024 10359 10078 10177  9967  9992  9930  9797  9545
    4550   1 2021-05-01 11438 10994 10343 10071 10183  9991 10003  9923  9813  9617
    4551   1 2021-05-01 11529 11017 10368 10096 10200 10020 10027  9966  9844  9720
    4552   1 2021-05-01 11351 10836 10203  9929 10038  9862  9881  9840  9705  9573
    4553   1 2021-05-01 11168 10836 10203  9929 10038  9862  9881  9840  9705  9394
    4554   1 2021-05-01 11052 10706 10060  9784  9889  9723  9729  9706  9568  9278
    4555   1 2021-05-01 11029 10642  9991  9714  9819  9636  9655  9645  9474  9281
    4556   1 2021-05-01 11057 10623  9998  9733  9853  9672  9698  9648  9529  9341
    4557   1 2021-05-01 10924 10542  9929  9677  9799  9632  9663  9590  9501  9236
    4558   1 2021-05-01 10849 10274  9675  9421  9533  9395  9431  9354  9264  9180
    4559   1 2021-05-01 10647 10194  9606  9349  9460  9328  9363  9303  9204  9004
    4560   1 2021-05-01 10408  9914  9348  9100  9222  9092  9147  9026  8995  8741
    4561   1 2021-05-01 11294 10388  9741  9488  9552  9351  9401  9224  9230  8993
    4562   1 2021-05-01 11439 10902 10236  9968 10038  9810  9839  9701  9657  9143
    4563   1 2021-05-01 11455 10976 10288  9984 10055  9812  9828  9720  9644  9138
    4564   1 2021-05-01 11867 11161 10461 10155 10228  9976  9986  9876  9791  9546
    4565   1 2021-05-01 11994 11557 10859 10562 10646 10385 10399 10282 10205  9699
    4566   1 2021-05-01 12260 11650 10926 10605 10683 10419 10410 10333 10202 10032
    4567   1 2021-05-01 12447 11879 11154 10836 10921 10648 10638 10561 10428 10206
    4568   1 2021-05-01 12408 11872 11162 10856 10936 10661 10652 10575 10444 10126
    4569   1 2021-05-01 12385 11846 11146 10852 10922 10647 10659 10566 10453 10074
    4570   1 2021-05-01 12095 11564 10840 10540 10599 10341 10354 10239 10157  9661
    4571   1 2021-05-01 12113 11635 10935 10652 10694 10447 10471 10392 10272  9593
    4572   1 2021-05-01 12127 11753 11024 10728 10756 10490 10502 10502 10304  9471
    4573   1 2021-05-01 12285 11753 11024 10728 10756 10490 10502 10502 10304  9675
    4574   1 2021-05-01 12401 12131 11368 11054 11087 10800 10784 10811 10555  9890
    4575   1 2021-05-01 12566 12173 11428 11108 11154 10867 10858 10874 10634 10056
    4576   1 2021-05-01 12595 12171 11416 11119 11178 10892 10888 10848 10676 10095
    4577   1 2021-05-01 12641 12178 11429 11126 11183 10906 10907 10846 10688 10154
    4578   1 2021-05-01 12432 12090 11355 11063 11112 10837 10836 10801 10624  9950
    4579   1 2021-05-01 12434 12007 11283 10994 11055 10788 10781 10716 10574  9940
    4580   1 2021-05-01 12255 11758 11043 10761 10813 10552 10568 10468 10368  9762
    4581   1 2021-05-01 12137 11685 10972 10681 10736 10473 10480 10392 10273  9660
    4582   1 2021-05-01 12140 11584 10860 10565 10619 10362 10361 10282 10161  9680
    4583   1 2021-05-01 12109 11589 10855 10545 10609 10348 10341 10249 10127  9740
    4584   1 2021-05-01 12087 11689 10956 10647 10711 10449 10434 10363 10220  9771
    4585   1 2021-05-01 12174 11660 10930 10625 10696 10430 10427 10337 10211  9882
    4586   1 2021-05-01 12150 11648 10914 10613 10681 10422 10415 10323 10202  9884
    4587   1 2021-05-01 11991 11452 10728 10421 10483 10219 10222 10131 10007  9723
    4588   1 2021-05-01 11973 11465 10759 10467 10529 10268 10270 10177 10065  9734
    4589   1 2021-05-01 11912 11417 10702 10404 10469 10211 10210 10145 10013  9660
    4590   1 2021-05-01 11786 11331 10614 10308 10370 10107 10099 10077  9889  9562
    4591   1 2021-05-01 11539 11169 10467 10175 10228  9983  9994  9949  9786  9329
    4592   1 2021-05-01 11467 11286 10580 10282 10331 10078 10073 10083  9877  9226
    4593   1 2021-05-01 11761 11439 10721 10409 10462 10205 10194 10220  9988  9601
    4594   1 2021-05-01 11977 11484 10785 10493 10560 10317 10319 10264 10133  9822
    4595   1 2021-05-01 11937 11501 10780 10473 10553 10282 10276 10241 10076  9819
    4596   1 2021-05-01 12062 11557 10849 10546 10625 10366 10355 10310 10154  9954
    4597   1 2021-05-01 12031 11571 10883 10607 10686 10443 10439 10356 10246  9884
    4598   1 2021-05-01 11886 11358 10660 10384 10451 10206 10204 10135 10010  9662
    4599   1 2021-05-01 11817 11342 10645 10376 10454 10202 10216 10122 10018  9593
    4600   1 2021-05-01 11765 11299 10597 10316 10387 10146 10152 10048  9958  9548
    4601   1 2021-05-01 11771 11299 10597 10316 10387 10146 10152 10048  9958  9599
    4602   1 2021-05-01 11772 11291 10570 10266 10327 10075 10072 10027  9860  9578
    4603   1 2021-05-01 11755 11227 10569 10306 10364 10134 10166 10093  9975  9520
    4604   1 2021-05-01 11320 10990 10338 10067 10122  9902  9946  9845  9768  8967
    4605   1 2021-05-01 11485 11087 10394 10098 10143  9907  9925  9880  9732  9140
    4606   1 2021-05-01 11630 11207 10510 10220 10278 10033 10054  9992  9856  9324
    4607   1 2021-05-01 11680 11252 10537 10231 10286 10041 10054  9998  9843  9399
    4608   1 2021-05-01 11736 11283 10568 10258 10305 10054 10061 10036  9849  9444
    4609   1 2021-05-01 11895 11405 10724 10433 10501 10266 10283 10228 10081  9589
    4610   1 2021-05-01 11730 11271 10600 10315 10374 10150 10175 10113  9981  9401
    4611   1 2021-05-01 11769 11319 10645 10365 10423 10193 10214 10159 10023  9443
    4612   1 2021-05-01 11592 11092 10421 10137 10189  9957  9983  9940  9804  9229
    4613   1 2021-05-01 11526 11167 10472 10179 10230 10005 10035  9968  9828  9159
    4614   1 2021-05-01 11558 11130 10462 10178 10236 10017 10051  9976  9853  9242
    4615   1 2021-05-01 11594 11182 10501 10206 10265 10042 10061 10015  9861  9294
    4616   1 2021-05-01 11909 11330 10651 10360 10431 10209 10221 10141 10030  9708
    4617   1 2021-05-01 11906 11316 10637 10344 10417 10191 10208 10113  9998  9743
    4618   1 2021-05-01 11730 11230 10566 10284 10356 10143 10164 10067  9962  9558
    4619   1 2021-05-01 11591 11008 10345 10054 10125  9916  9945  9855  9748  9411
    4620   1 2021-05-01 11526 11058 10387 10085 10150  9944  9955  9899  9755  9400
    4621   1 2021-05-01 11432 10960 10306 10027 10102  9900  9933  9858  9743  9333
    4622   1 2021-05-01 11310 10863 10215  9938 10016  9819  9855  9762  9678  9234
    4623   1 2021-05-01 11238 10616  9986  9712  9800  9612  9654  9549  9479  9172
    4624   1 2021-05-01 11109 10675 10046  9776  9873  9682  9723  9617  9541  9074
    4625   1 2021-05-01 10984 10688 10036  9749  9822  9636  9670  9651  9489  8959
    4626   1 2021-05-01 11128 10786 10134  9856  9934  9748  9773  9777  9596  9112
    4627   1 2021-05-01 11263 10869 10229  9952 10043  9854  9887  9829  9716  9293
    4628   1 2021-05-01 11346 10941 10283 10002 10087  9905  9928  9874  9753  9426
    4629   1 2021-05-01 11497 11060 10389 10101 10186  9988 10001  9982  9813  9632
    4630   1 2021-05-01 11472 11131 10472 10196 10289 10083 10101 10067  9901  9606
    4631   1 2021-05-01 11481 11045 10386 10102 10204 10002 10012  9986  9814  9633
    4632   1 2021-05-01 11404 10869 10245  9970 10076  9895  9913  9859  9721  9590
    4633   1 2021-05-01 11116 10754 10124  9867  9972  9794  9819  9752  9646  9342
    4634   1 2021-05-01 11123 10623  9976  9701  9805  9625  9655  9600  9479  9343
    4635   1 2021-05-01 11040 10645  9996  9716  9824  9641  9659  9615  9484  9287
    4636   1 2021-05-01 11034 10575  9954  9694  9808  9634  9659  9636  9475  9302
    4637   1 2021-05-01 10896 10447  9838  9583  9698  9541  9574  9525  9412  9215
    4638   1 2021-05-01 10773 10236  9646  9399  9517  9378  9425  9346  9267  9118
    4639   1 2021-05-01 10589 10031  9457  9214  9334  9199  9253  9189  9090  8959
    4640   1 2021-05-01 10466 10091  9514  9264  9379  9250  9289  9250  9133  8854
    4641   1 2021-05-01 10021  9880  9295  9040  9146  9025  9083  8993  8920  8423
    4642   1 2021-05-01 11503 11111 10432 10169 10246 10009 10023  9900  9835  9267
    4643   1 2021-05-01 11494 11161 10478 10208 10281 10042 10059  9965  9875  9223
    4644   1 2021-05-01 11888 11288 10596 10310 10381 10141 10156 10047  9971  9589
    4645   1 2021-05-01 12071 11623 10916 10607 10677 10421 10421 10372 10227  9749
    4646   1 2021-05-01 12218 11791 11087 10788 10869 10596 10601 10539 10408  9917
    4647   1 2021-05-01 12331 12056 11327 11022 11107 10820 10822 10779 10623 10046
    4648   1 2021-05-01 12535 12056 11324 11015 11101 10820 10811 10750 10610 10248
    4649   1 2021-05-01 12587 12056 11324 11015 11101 10820 10811 10750 10610 10311
    4650   1 2021-05-01 12500 11969 11240 10923 11003 10723 10715 10657 10509 10174
    4651   1 2021-05-01 12437 11982 11265 10960 11037 10753 10755 10692 10551 10085
    4652   1 2021-05-01 12354 11745 11039 10743 10808 10552 10566 10448 10366  9934
    4653   1 2021-05-01 11885 11556 10869 10588 10631 10391 10425 10333 10224  9241
    4654   1 2021-05-01 11757 11369 10693 10428 10466 10239 10279 10164 10085  9032
    4655   1 2021-05-01 11821 11547 10804 10485 10515 10260 10269 10241 10058  9189
    4656   1 2021-05-01 12296 11972 11220 10898 10945 10657 10649 10644 10433  9785
    4657   1 2021-05-01 12397 12016 11278 10964 11010 10727 10723 10719 10508  9909
    4658   1 2021-05-01 12407 11939 11193 10892 10949 10674 10669 10615 10464  9935
    4659   1 2021-05-01 12411 11976 11230 10935 10981 10714 10706 10665 10492  9978
    4660   1 2021-05-01 12389 11864 11135 10845 10907 10645 10639 10576 10439  9938
    4661   1 2021-05-01 12365 11761 11056 10784 10831 10571 10592 10501 10390  9880
    4662   1 2021-05-01 12099 11609 10884 10573 10625 10357 10362 10304 10156  9622
    4663   1 2021-05-01 12056 11669 10951 10648 10697 10426 10435 10380 10219  9599
    4664   1 2021-05-01 12070 11589 10880 10587 10637 10384 10391 10309 10178  9652
    4665   1 2021-05-01 12034 11607 10868 10551 10609 10341 10326 10290 10115  9672
    4666   1 2021-05-01 12132 11691 10948 10637 10696 10423 10411 10382 10204  9843
    4667   1 2021-05-01 12161 11740 11010 10699 10765 10493 10476 10442 10256  9889
    4668   1 2021-05-01 12181 11678 10942 10623 10693 10413 10393 10380 10179  9939
    4669   1 2021-05-01 12140 11671 10954 10662 10727 10471 10464 10398 10258  9901
    4670   1 2021-05-01 12061 11504 10789 10491 10558 10301 10304 10225 10099  9838
    4671   1 2021-05-01 11979 11482 10778 10477 10550 10288 10295 10238 10094  9770
    4672   1 2021-05-01 11783 11112 10446 10179 10248 10019 10042  9931  9851  9564
    4673   1 2021-05-01 11380 10756 10088  9815  9869  9648  9673  9614  9491  9171
    4674   1 2021-05-01 11408 11187 10485 10187 10243  9982  9994  9984  9792  9138
    4675   1 2021-05-01 11700 11273 10575 10291 10350 10095 10103 10089  9895  9521
    4676   1 2021-05-01 11822 11417 10721 10434 10492 10247 10247 10223 10045  9664
    4677   1 2021-05-01 11931 11479 10762 10455 10532 10270 10266 10227 10064  9805
    4678   1 2021-05-01 12101 11668 10947 10646 10729 10458 10442 10414 10235 10008
    4679   1 2021-05-01 12056 11640 10939 10647 10725 10460 10455 10447 10242  9889
    4680   1 2021-05-01 11853 11463 10767 10486 10556 10307 10314 10233 10119  9651
    4681   1 2021-05-01 11918 11455 10752 10461 10531 10268 10269 10233 10062  9720
    4682   1 2021-05-01 11866 11466 10768 10486 10559 10305 10313 10262 10117  9650
    4683   1 2021-05-01 11952 11417 10708 10420 10496 10246 10256 10174 10061  9804
    4684   1 2021-05-01 11869 11344 10658 10370 10441 10195 10212 10137 10014  9710
    4685   1 2021-05-01 11651 10973 10335 10083 10145  9922  9962  9848  9774  9404
    4686   1 2021-05-01 11350 10913 10246  9964 10015  9786  9820  9722  9627  9000
    4687   1 2021-05-01 11388 11124 10444 10162 10213  9970  9995  9964  9806  9023
    4688   1 2021-05-01 11426 11028 10344 10051 10080  9843  9862  9840  9660  9079
    4689   1 2021-05-01 11569 11220 10515 10217 10259 10020 10032  9981  9828  9258
    4690   1 2021-05-01 11695 11220 10515 10217 10259 10020 10032  9981  9828  9406
    4691   1 2021-05-01 11602 11322 10643 10352 10411 10165 10188 10149  9983  9251
    4692   1 2021-05-01 11578 11181 10516 10246 10298 10066 10102 10046  9909  9200
    4693   1 2021-05-01 11577 11192 10517 10238 10292 10066 10100 10041  9904  9215
    4694   1 2021-05-01 11374 10929 10256  9958 10003  9771  9808  9757  9605  8969
    4695   1 2021-05-01 11471 11169 10500 10218 10274 10051 10090 10030  9896  9123
    4696   1 2021-05-01 11651 11254 10570 10276 10337 10101 10127 10096  9950  9336
    4697   1 2021-05-01 11737 11395 10720 10426 10490 10264 10281 10228 10089  9449
    4698   1 2021-05-01 11844 11438 10748 10453 10521 10288 10297 10257 10089  9602
    4699   1 2021-05-01 11867 11367 10695 10413 10495 10272 10291 10195 10082  9692
    4700   1 2021-05-01 11721 11248 10590 10314 10393 10172 10202 10106 10002  9547
    4701   1 2021-05-01 11610 11073 10398 10102 10170  9955  9985  9922  9785  9449
    4702   1 2021-05-01 11514 11073 10398 10102 10170  9955  9985  9922  9785  9365
    4703   1 2021-05-01 11352 10988 10328 10045 10113  9906  9930  9883  9735  9232
    4704   1 2021-05-01 11416 10969 10312 10025 10092  9875  9903  9871  9720  9321
    4705   1 2021-05-01 11338 10849 10211  9931 10011  9825  9855  9767  9684  9286
    4706   1 2021-05-01 11260 10838 10200  9921 10008  9810  9840  9791  9669  9214
    4707   1 2021-05-01 11034 10642 10011  9744  9825  9646  9682  9612  9510  9005
    4708   1 2021-05-01 10911 10786 10134  9856  9934  9748  9773  9777  9596  8877
    4709   1 2021-05-01 10929 10590  9949  9674  9753  9580  9613  9611  9435  8939
    4710   1 2021-05-01 11198 10919 10259  9966 10057  9866  9887  9866  9703  9231
    4711   1 2021-05-01 11236 10947 10291  9988 10073  9886  9908  9892  9714  9295
    4712   1 2021-05-01 11394 11007 10347 10055 10152  9950  9964  9933  9765  9481
    4713   1 2021-05-01 11423 10958 10308 10026 10120  9924  9945  9906  9759  9553
    4714   1 2021-05-01 11305 10824 10195  9926 10030  9850  9865  9809  9696  9465
    4715   1 2021-05-01 11244 10799 10162  9888 10001  9812  9835  9791  9656  9427
    4716   1 2021-05-01 11231 10748 10108  9841  9945  9762  9788  9731  9617  9456
    4717   1 2021-05-01 11175 10787 10143  9872  9978  9792  9816  9748  9649  9418
    4718   1 2021-05-01 11156 10593  9963  9691  9799  9623  9639  9592  9462  9411
    4719   1 2021-05-01 10967 10506  9894  9634  9747  9590  9622  9561  9456  9264
    4720   1 2021-05-01 10678 10294  9700  9455  9575  9427  9474  9403  9310  9027
    4721   1 2021-05-01 10453 10031  9457  9214  9334  9199  9253  9189  9090  8843
    4722   1 2021-05-01 10200  9733  9184  8951  9066  8954  9020  8963  8871  8602
    4723   1 2021-05-01  9928  9501  8947  8701  8807  8724  8797  8774  8654  8345
    4724   1 2021-05-01  9657  9360  8806  8554  8660  8575  8653  8636  8511  8078
    4725   1 2021-05-01 11687 11268 10565 10280 10340 10094 10104 10038  9899  9406
    4726   1 2021-05-01 11862 11403 10702 10433 10520 10270 10285 10177 10099  9554
    4727   1 2021-05-01 11921 11403 10702 10433 10520 10270 10285 10177 10099  9633
    4728   1 2021-05-01 12094 11462 10769 10489 10559 10314 10327 10239 10134  9756
    4729   1 2021-05-01 12118 11691 10991 10693 10762 10509 10519 10459 10329  9742
    4730   1 2021-05-01 12286 11847 11141 10840 10905 10638 10642 10606 10434  9918
    4731   1 2021-05-01 12547 12060 11335 11021 11086 10810 10811 10814 10599 10193
    4732   1 2021-05-01 12657 12152 11416 11109 11195 10907 10907 10857 10712 10288
    4733   1 2021-05-01 12619 12149 11404 11091 11161 10878 10874 10851 10681 10315
    4734   1 2021-05-01 12481 12060 11320 11010 11079 10800 10784 10758 10574 10108
    4735   1 2021-05-01 12347 11897 11185 10879 10930 10648 10655 10621 10441  9939
    4736   1 2021-05-01 12114 11475 10789 10496 10555 10312 10328 10227 10134  9664
    4737   1 2021-05-01 11794 11421 10749 10469 10529 10288 10314 10175 10120  9212
    4738   1 2021-05-01 11832 11433 10731 10427 10471 10231 10247 10155 10048  9204
    4739   1 2021-05-01 12024 11841 11083 10761 10802 10523 10526 10536 10312  9484
    4740   1 2021-05-01 12225 11841 11083 10761 10802 10523 10526 10536 10312  9688
    4741   1 2021-05-01 12366 11954 11218 10919 10963 10687 10693 10642 10488  9871
    4742   1 2021-05-01 12172 11964 11237 10950 11003 10719 10730 10689 10521  9636
    4743   1 2021-05-01 12313 11968 11222 10938 10990 10728 10722 10653 10518  9807
    4744   1 2021-05-01 12323 11850 11123 10846 10900 10636 10635 10572 10425  9854
    4745   1 2021-05-01 12184 11717 11010 10722 10774 10513 10519 10458 10315  9698
    4746   1 2021-05-01 12066 11552 10831 10528 10576 10320 10334 10253 10128  9564
    4747   1 2021-05-01 11979 11536 10821 10519 10572 10310 10312 10262 10109  9552
    4748   1 2021-05-01 11974 11489 10765 10464 10512 10264 10267 10203 10055  9576
    4749   1 2021-05-01 12026 11619 10894 10592 10648 10378 10380 10311 10162  9679
    4750   1 2021-05-01 12136 11594 10866 10569 10634 10367 10362 10291 10157  9816
    4751   1 2021-05-01 11999 11556 10814 10494 10561 10297 10281 10248 10082  9735
    4752   1 2021-05-01 12093 11523 10796 10489 10551 10283 10280 10248 10076  9851
    4753   1 2021-05-01 11970 11486 10771 10466 10537 10271 10271 10237 10061  9723
    4754   1 2021-05-01 11986 11593 10884 10587 10668 10401 10407 10342 10195  9768
    4755   1 2021-05-01 11709 11392 10694 10390 10468 10215 10218 10142 10015  9495
    4756   1 2021-05-01 11527 11143 10475 10200 10260 10020 10030  9961  9825  9289
    4757   1 2021-05-01 11347 10793 10160  9907  9979  9760  9785  9624  9598  9075
    4758   1 2021-05-01 11233 10955 10258  9966 10021  9775  9787  9783  9590  8966
    4759   1 2021-05-01 11550 11064 10353 10066 10114  9857  9869  9888  9675  9327
    4760   1 2021-05-01 11609 11330 10630 10342 10404 10154 10151 10142  9946  9446
    4761   1 2021-05-01 11774 11331 10628 10339 10399 10141 10140 10141  9940  9637
    4762   1 2021-05-01 11846 11346 10650 10352 10414 10161 10162 10151  9944  9723
    4763   1 2021-05-01 11633 11148 10461 10178 10242  9997 10001  9950  9802  9480
    4764   1 2021-05-01 11679 11326 10594 10273 10328 10059 10053 10061  9845  9484
    4765   1 2021-05-01 11793 11325 10610 10313 10365 10115 10123 10091  9921  9585
    4766   1 2021-05-01 11733 11318 10626 10338 10397 10153 10151 10120  9951  9515
    4767   1 2021-05-01 11749 11341 10650 10360 10420 10180 10189 10139  9983  9555
    4768   1 2021-05-01 11749 11341 10650 10360 10420 10180 10189 10139  9983  9540
    4769   1 2021-05-01 11571 11060 10409 10140 10204  9975 10003  9894  9800  9308
    4770   1 2021-05-01 11416 10972 10316 10046 10096  9870  9903  9792  9707  9081
    4771   1 2021-05-01 11397 10900 10248  9968 10018  9794  9828  9754  9637  9072
    4772   1 2021-05-01 11251 10777 10115  9838  9881  9658  9694  9616  9514  8891
    4773   1 2021-05-01 11568 11120 10416 10115 10151  9915  9923  9882  9712  9252
    4774   1 2021-05-01 11635 11166 10467 10169 10220  9972  9992  9933  9779  9321
    4775   1 2021-05-01 11586 11114 10450 10182 10232 10003 10032  9959  9836  9250
    4776   1 2021-05-01 11513 10987 10334 10069 10117  9904  9951  9856  9761  9117
    4777   1 2021-05-01 11280 10831 10144  9828  9870  9643  9687  9606  9476  8876
    4778   1 2021-05-01 11405 11115 10422 10108 10165  9926  9951  9882  9745  8983
    4779   1 2021-05-01 11504 11042 10389 10127 10193  9968 10015  9895  9838  9130
    4780   1 2021-05-01 11389 11149 10473 10189 10250 10018 10053  9995  9867  9039
    4781   1 2021-05-01 11651 11149 10473 10189 10250 10018 10053  9995  9867  9326
    4782   1 2021-05-01 11754 11303 10620 10327 10387 10162 10177 10131  9985  9467
    4783   1 2021-05-01 11859 11390 10705 10412 10485 10259 10268 10200 10077  9639
    4784   1 2021-05-01 11870 11400 10715 10419 10502 10262 10272 10220 10069  9674
    4785   1 2021-05-01 11733 11308 10637 10349 10428 10197 10211 10143 10017  9538
    4786   1 2021-05-01 11516 11116 10454 10173 10239 10019 10051  9980  9852  9337
    4787   1 2021-05-01 11512 11075 10409 10123 10198  9976 10002  9931  9810  9361
    4788   1 2021-05-01 11508 10979 10312 10021 10093  9878  9904  9848  9722  9376
    4789   1 2021-05-01 11377 10955 10312 10034 10109  9903  9937  9865  9751  9283
    4790   1 2021-05-01 11296 10854 10215  9944 10028  9831  9874  9801  9700  9232
    4791   1 2021-05-01 11213 10726 10097  9831  9915  9738  9768  9704  9600  9170
    4792   1 2021-05-01 11096 10502  9888  9626  9711  9539  9593  9497  9421  9039
    4793   1 2021-05-01 10878 10353  9731  9465  9554  9377  9425  9342  9264  8841
    4794   1 2021-05-01 10751 10426  9823  9561  9656  9483  9531  9439  9369  8749
    4795   1 2021-05-01 10856 10576  9934  9651  9738  9558  9595  9606  9422  8855
    4796   1 2021-05-01 11062 10745 10084  9791  9872  9695  9718  9736  9531  9054
    4797   1 2021-05-01 11280 10938 10288  9996 10078  9893  9915  9893  9713  9345
    4798   1 2021-05-01 11362 10914 10267  9979 10065  9877  9902  9867  9712  9488
    4799   1 2021-05-01 11370 10918 10279 10006 10103  9913  9933  9891  9757  9511
    4800   1 2021-05-01 11308 10897 10272  9998 10103  9912  9939  9883  9762  9491
    4801   1 2021-05-01 11267 10875 10235  9964 10077  9878  9907  9878  9733  9474
    4802   1 2021-05-01 11196 10805 10166  9897 10010  9828  9853  9833  9690  9427
    4803   1 2021-05-01 11099 10592  9973  9707  9824  9654  9696  9633  9526  9377
    4804   1 2021-05-01 10879 10362  9755  9495  9606  9458  9491  9445  9327  9174
    4805   1 2021-05-01 10734 10179  9593  9344  9450  9315  9353  9315  9185  9052
    4806   1 2021-05-01 10384  9847  9302  9067  9188  9068  9132  9032  8990  8734
    4807   1 2021-05-01 10045  9494  8971  8737  8859  8760  8833  8737  8699  8429
    4808   1 2021-05-01  9684  9402  8888  8666  8802  8712  8793  8656  8664  8098
    4809   1 2021-05-01 11833 11533 10821 10519 10571 10300 10307 10317 10092  9447
    4810   1 2021-05-01 12078 11721 11035 10749 10811 10553 10566 10524 10358  9697
    4811   1 2021-05-01 12084 11627 10934 10624 10678 10420 10435 10417 10235  9654
    4812   1 2021-05-01 12151 11827 11126 10821 10864 10600 10609 10627 10404  9706
    4813   1 2021-05-01 12058 11767 11056 10745 10784 10515 10535 10575 10335  9474
    4814   1 2021-05-01 12419 12046 11303 10990 11042 10764 10771 10779 10572  9919
    4815   1 2021-05-01 12490 11862 11121 10793 10843 10578 10582 10558 10372 10058
    4816   1 2021-05-01 12344 12050 11298 10968 11024 10746 10737 10747 10527  9886
    4817   1 2021-05-01 12320 11974 11219 10879 10919 10636 10630 10659 10413  9933
    4818   1 2021-05-01 12234 11773 11066 10764 10832 10568 10564 10483 10343  9785
    4819   1 2021-05-01 12041 11701 11003 10701 10760 10495 10510 10455 10310  9525
    4820   1 2021-05-01 11947 11525 10822 10522 10569 10313 10338 10271 10136  9411
    4821   1 2021-05-01 11861 11599 10875 10573 10620 10365 10382 10331 10182  9292
    4822   1 2021-05-01 11866 11458 10750 10461 10503 10261 10285 10241 10088  9283
    4823   1 2021-05-01 11776 11671 10946 10640 10668 10396 10413 10443 10206  9190
    4824   1 2021-05-01 11886 11464 10748 10445 10474 10211 10234 10234 10023  9327
    4825   1 2021-05-01 11939 11532 10809 10516 10546 10278 10293 10298 10081  9409
    4826   1 2021-05-01 11910 11385 10651 10347 10376 10115 10132 10098  9925  9376
    4827   1 2021-05-01 11936 11508 10788 10483 10531 10277 10288 10219 10075  9485
    4828   1 2021-05-01 11946 11433 10717 10428 10490 10247 10253 10139 10054  9502
    4829   1 2021-05-01 11932 11380 10660 10361 10412 10160 10172 10096  9968  9498
    4830   1 2021-05-01 11717 11380 10660 10361 10412 10160 10172 10096  9968  9261
    4831   1 2021-05-01 11885 11526 10795 10480 10524 10263 10267 10228 10045  9476
    4832   1 2021-05-01 12060 11536 10802 10491 10540 10275 10268 10241 10057  9706
    4833   1 2021-05-01 12029 11529 10793 10486 10548 10296 10283 10225 10087  9754
    4834   1 2021-05-01 11735 11523 10796 10489 10551 10283 10280 10248 10076  9414
    4835   1 2021-05-01 11788 11380 10659 10353 10411 10146 10148 10124  9946  9485
    4836   1 2021-05-01 11836 11392 10689 10390 10468 10208 10203 10139 10007  9609
    4837   1 2021-05-01 11790 11302 10620 10339 10411 10153 10163 10087  9962  9565
    4838   1 2021-05-01 11431 10953 10289 10009 10076  9843  9860  9763  9667  9205
    4839   1 2021-05-01 11331 10882 10193  9895  9952  9708  9717  9676  9519  9080
    4840   1 2021-05-01 11353 10876 10186  9893  9950  9718  9731  9711  9540  9123
    4841   1 2021-05-01 11255 10936 10244  9949  9998  9760  9765  9755  9566  9028
    4842   1 2021-05-01 11480 11125 10433 10153 10212  9958  9965  9952  9772  9277
    4843   1 2021-05-01 11526 11125 10433 10153 10212  9958  9965  9952  9772  9363
    4844   1 2021-05-01 11541 11158 10459 10164 10227  9967  9979  9950  9775  9383
    4845   1 2021-05-01 11500 11071 10381 10086 10143  9899  9905  9880  9707  9311
    4846   1 2021-05-01 11606 11199 10480 10185 10235  9986  9997  9954  9797  9410
    4847   1 2021-05-01 11690 11325 10610 10313 10365 10115 10123 10091  9921  9507
    4848   1 2021-05-01 11615 11274 10580 10285 10341 10087 10100 10065  9900  9449
    4849   1 2021-05-01 11680 11286 10597 10318 10383 10142 10151 10089  9963  9494
    4850   1 2021-05-01 11675 11108 10418 10135 10188  9954  9963  9911  9767  9467
    4851   1 2021-05-01 11485 11039 10353 10058 10110  9870  9878  9825  9679  9220
    4852   1 2021-05-01 11496 11038 10358 10070 10119  9885  9906  9841  9706  9220
    4853   1 2021-05-01 11468 10933 10280 10009 10066  9844  9872  9756  9684  9173
    4854   1 2021-05-01 11370 11080 10392 10096 10146  9913  9920  9870  9722  9027
    4855   1 2021-05-01 11547 11126 10432 10130 10177  9948  9958  9908  9747  9242
    4856   1 2021-05-01 11578 11117 10435 10138 10184  9951  9973  9926  9766  9243
    4857   1 2021-05-01 11592 11129 10452 10172 10218  9985 10012  9941  9808  9216
    4858   1 2021-05-01 11390 10999 10354 10097 10142  9934  9986  9877  9799  8953
    4859   1 2021-05-01 11295 10866 10172  9868  9912  9691  9739  9629  9528  8842
    4860   1 2021-05-01 11777 11115 10422 10108 10165  9926  9951  9882  9745  9428
    4861   1 2021-05-01 11694 11261 10589 10306 10369 10146 10177 10087  9985  9390
    4862   1 2021-05-01 11691 11204 10528 10247 10313 10082 10117 10029  9921  9371
    4863   1 2021-05-01 11681 11224 10540 10258 10328 10101 10128 10056  9930  9362
    4864   1 2021-05-01 11749 11368 10675 10374 10434 10204 10218 10196 10015  9456
    4865   1 2021-05-01 11868 11382 10692 10388 10454 10222 10234 10212 10033  9604
    4866   1 2021-05-01 11826 11356 10674 10383 10461 10228 10240 10167 10041  9587
    4867   1 2021-05-01 11805 11257 10587 10314 10395 10172 10201 10096 10007  9601
    4868   1 2021-05-01 11561 11131 10463 10182 10256 10031 10062  9992  9862  9397
    4869   1 2021-05-01 11599 11154 10492 10224 10296 10080 10112 10049  9923  9453
    4870   1 2021-05-01 11557 11061 10408 10146 10216 10016 10040  9959  9854  9431
    4871   1 2021-05-01 11487 11061 10408 10146 10216 10016 10040  9959  9854  9401
    4872   1 2021-05-01 11363 10930 10299 10036 10120  9913  9958  9879  9778  9272
    4873   1 2021-05-01 11171 10759 10130  9869  9953  9769  9815  9729  9649  9072
    4874   1 2021-05-01 10982 10560  9932  9668  9750  9572  9622  9542  9459  8908
    4875   1 2021-05-01 11018 10571  9932  9651  9734  9541  9582  9507  9399  8956
    4876   1 2021-05-01 11069 10695 10055  9767  9854  9667  9698  9631  9517  9041
    4877   1 2021-05-01 11089 10574  9950  9684  9769  9603  9644  9592  9476  9084
    4878   1 2021-05-01 11028 10650 10015  9745  9827  9652  9694  9652  9518  9025
    4879   1 2021-05-01 11142 10773 10121  9838  9911  9738  9773  9765  9579  9190
    4880   1 2021-05-01 11091 10756 10109  9821  9900  9725  9750  9741  9561  9137
    4881   1 2021-05-01 11259 10913 10277  9981 10071  9871  9887  9909  9703  9380
    4882   1 2021-05-01 11299 10718 10091  9812  9911  9723  9751  9735  9565  9472
    4883   1 2021-05-01 11210 10674 10036  9765  9868  9696  9718  9700  9556  9415
    4884   1 2021-05-01 11074 10674 10036  9765  9868  9696  9718  9700  9556  9290
    4885   1 2021-05-01 10926 10609  9986  9725  9841  9662  9696  9668  9530  9173
    4886   1 2021-05-01 10583 10154  9555  9292  9396  9255  9297  9243  9127  8835
    4887   1 2021-05-01 11905 11608 10907 10618 10680 10410 10421 10357 10208  9533
    4888   1 2021-05-01 12067 11618 10916 10617 10673 10406 10410 10389 10189  9641
    4889   1 2021-05-01 12007 11571 10873 10575 10643 10382 10394 10309 10189  9571
    4890   1 2021-05-01 11921 11621 10924 10608 10648 10391 10401 10406 10188  9426
    4891   1 2021-05-01 11822 11387 10731 10442 10480 10240 10274 10221 10073  9225
    4892   1 2021-05-01 12044 11651 10945 10633 10671 10408 10418 10421 10223  9473
    4893   1 2021-05-01 12178 11723 11021 10702 10756 10492 10491 10447 10304  9669
    4894   1 2021-05-01 12299 11899 11154 10827 10881 10621 10626 10587 10414  9840
    4895   1 2021-05-01 12317 11811 11075 10749 10793 10534 10562 10553 10349  9876
    4896   1 2021-05-01 12001 11642 10950 10641 10683 10415 10425 10410 10221  9509
    4897   1 2021-05-01 11700 11366 10660 10352 10390 10136 10161 10136  9974  9146
    4898   1 2021-05-01 11487 11013 10317 10012 10032  9797  9829  9809  9639  8838
    4899   1 2021-05-01 11608 11096 10395 10081 10091  9851  9873  9915  9676  8952
    4900   1 2021-05-01 11657 10949 10234  9901  9899  9645  9674  9717  9467  9021
    4901   1 2021-05-01 11499 11286 10583 10280 10308 10066 10090 10054  9895  8924
    4902   1 2021-05-01 11471 11084 10385 10084 10110  9863  9895  9855  9698  8864
    4903   1 2021-05-01 11711 11261 10554 10260 10294 10046 10067 10009  9865  9184
    4904   1 2021-05-01 11848 11398 10690 10397 10450 10194 10212 10107 10000  9361
    4905   1 2021-05-01 12079 11548 10824 10519 10575 10323 10328 10237 10115  9663
    4906   1 2021-05-01 11944 11490 10781 10493 10550 10297 10306 10215 10101  9487
    4907   1 2021-05-01 11821 11323 10624 10341 10398 10157 10178 10056  9987  9365
    4908   1 2021-05-01 11682 11222 10506 10214 10250 10007 10032  9945  9842  9213
    4909   1 2021-05-01 12030 11478 10749 10434 10493 10236 10225 10144 10015  9696
    4910   1 2021-05-01 12087 11535 10802 10487 10556 10289 10281 10198 10070  9825
    4911   1 2021-05-01 12176 11480 10767 10458 10539 10272 10265 10124 10059  9962
    4912   1 2021-05-01 11876 11235 10528 10234 10283 10034 10029  9972  9827  9632
    4913   1 2021-05-01 11770 11334 10606 10297 10366 10108 10096 10012  9881  9512
    4914   1 2021-05-01 11778 11303 10595 10304 10373 10106 10121 10045  9916  9560
    4915   1 2021-05-01 11743 11303 10595 10304 10373 10106 10121 10045  9916  9533
    4916   1 2021-05-01 11633 11070 10408 10147 10222  9985 10003  9873  9819  9408
    4917   1 2021-05-01 11392 11000 10308 10021 10093  9838  9854  9731  9670  9166
    4918   1 2021-05-01 11358 10804 10121  9832  9889  9656  9669  9624  9476  9171
    4919   1 2021-05-01 11423 10966 10272  9973 10023  9794  9799  9733  9597  9248
    4920   1 2021-05-01 11438 10983 10322 10041 10107  9859  9885  9827  9684  9248
    4921   1 2021-05-01 11350 10828 10159  9870  9934  9698  9730  9638  9540  9150
    4922   1 2021-05-01 11322 10879 10198  9896  9958  9707  9714  9674  9514  9155
    4923   1 2021-05-01 11343 11020 10339 10051 10097  9861  9876  9844  9681  9139
    4924   1 2021-05-01 11413 10997 10316 10032 10082  9846  9860  9811  9666  9218
    4925   1 2021-05-01 11440 11116 10420 10136 10192  9950  9969  9940  9783  9253
    4926   1 2021-05-01 11307 10912 10234  9946  9988  9756  9792  9784  9596  9054
    4927   1 2021-05-01 11404 11021 10345 10065 10114  9876  9891  9848  9691  9149
    4928   1 2021-05-01 11415 10905 10216  9924  9960  9730  9743  9687  9542  9148
    4929   1 2021-05-01 11377 10917 10221  9919  9966  9735  9744  9681  9544  9081
    4930   1 2021-05-01 11356 10963 10278  9979 10025  9792  9807  9755  9601  9029
    4931   1 2021-05-01 11427 10983 10301 10007 10060  9823  9854  9795  9655  9080
    4932   1 2021-05-01 11488 11021 10320 10009 10052  9819  9828  9784  9623  9139
    4933   1 2021-05-01 11557 11073 10387 10090 10140  9914  9923  9850  9718  9248
    4934   1 2021-05-01 11520 11065 10383 10083 10132  9903  9924  9852  9716  9191
    4935   1 2021-05-01 11458 11031 10375 10105 10148  9930  9965  9897  9777  9039
    4936   1 2021-05-01 11264 10865 10192  9906  9946  9735  9780  9681  9592  8814
    4937   1 2021-05-01 11475 11147 10443 10141 10182  9951  9969  9922  9769  9012
    4938   1 2021-05-01 11710 11281 10602 10315 10372 10140 10170 10119  9968  9323
    4939   1 2021-05-01 11664 11206 10533 10250 10299 10078 10102 10050  9914  9324
    4940   1 2021-05-01 11673 11235 10555 10280 10352 10128 10156 10077  9963  9340
    4941   1 2021-05-01 11676 11231 10541 10254 10313 10084 10112 10036  9916  9367
    4942   1 2021-05-01 11665 11253 10556 10259 10319 10095 10109 10056  9900  9382
    4943   1 2021-05-01 11765 11263 10575 10273 10331 10108 10120 10075  9920  9496
    4944   1 2021-05-01 11793 11359 10671 10375 10443 10214 10226 10174 10027  9589
    4945   1 2021-05-01 11753 11306 10634 10349 10426 10203 10220 10151 10020  9551
    4946   1 2021-05-01 11639 11199 10524 10253 10323 10109 10129 10052  9942  9456
    4947   1 2021-05-01 11531 11150 10494 10228 10295 10082 10116 10049  9926  9386
    4948   1 2021-05-01 11453 11051 10394 10117 10184  9975 10006  9964  9822  9343
    4949   1 2021-05-01 11392 10953 10308 10049 10122  9925  9958  9895  9772  9288
    4950   1 2021-05-01 11232 10772 10146  9884  9958  9771  9818  9780  9630  9095
    4951   1 2021-05-01 11077 10532  9920  9660  9734  9564  9623  9542  9455  8951
    4952   1 2021-05-01 10813 10427  9809  9531  9600  9427  9477  9414  9304  8680
    4953   1 2021-05-01 11045 10688 10045  9764  9840  9665  9699  9656  9518  8971
    4954   1 2021-05-01 11070 10623  9986  9709  9791  9608  9649  9596  9470  9033
    4955   1 2021-05-01 11090 10650 10015  9743  9825  9650  9684  9653  9511  9058
    4956   1 2021-05-01 11099 10650 10015  9743  9825  9650  9684  9653  9511  9103
    4957   1 2021-05-01 11148 10616  9990  9731  9819  9645  9696  9614  9515  9164
    4958   1 2021-05-01 11200 10753 10120  9858  9946  9767  9809  9730  9625  9238
    4959   1 2021-05-01 11178 10764 10129  9846  9933  9756  9783  9744  9616  9265
    4960   1 2021-05-01 11087 10710 10076  9809  9905  9731  9767  9693  9593  9220
    4961   1 2021-05-01 11082 10635 10010  9748  9850  9672  9714  9651  9534  9236
    4962   1 2021-05-01 11030 10598  9963  9693  9793  9618  9639  9618  9467  9203
    4963   1 2021-05-01 11759 11368 10661 10371 10429 10178 10193 10140  9980  9329
    4964   1 2021-05-01 11769 11549 10829 10535 10587 10328 10334 10298 10121  9252
    4965   1 2021-05-01 12010 11496 10782 10474 10517 10243 10249 10266 10036  9524
    4966   1 2021-05-01 12001 11461 10769 10474 10507 10244 10261 10239 10053  9525
    4967   1 2021-05-01 11451 11081 10371 10057 10081  9822  9839  9806  9625  8851
    4968   1 2021-05-01 11591 11213 10522 10198 10228  9970  9982  9964  9769  9029
    4969   1 2021-05-01 11601 11096 10406 10076 10102  9842  9858  9835  9644  8886
    4970   1 2021-05-01 11855 11522 10812 10468 10499 10220 10221 10247 10010  9193
    4971   1 2021-05-01 11935 11394 10679 10336 10357 10085 10088 10103  9871  9364
    4972   1 2021-05-01 11980 11467 10779 10467 10501 10232 10255 10222 10052  9438
    4973   1 2021-05-01 11947 11811 11075 10749 10793 10534 10562 10553 10349  9380
    4974   1 2021-05-01 11726 11476 10763 10430 10457 10199 10220 10217 10014  9132
    4975   1 2021-05-01 11733 11196 10516 10221 10264 10022 10049  9949  9856  9158
    4976   1 2021-05-01 11148 10836 10189  9912  9968  9745  9797  9617  9620  8492
    4977   1 2021-05-01 10641 10353  9723  9443  9456  9248  9310  9215  9138  7838
    4978   1 2021-05-01 10677 10458  9785  9476  9478  9254  9307  9256  9120  7848
    4979   1 2021-05-01 11457 11129 10438 10135 10179  9927  9963  9847  9755  8816
    4980   1 2021-05-01 11822 11406 10702 10396 10445 10179 10211 10132 10005  9311
    4981   1 2021-05-01 11958 11522 10799 10504 10551 10293 10307 10237 10109  9475
    4982   1 2021-05-01 12041 11570 10837 10526 10577 10313 10320 10281 10104  9614
    4983   1 2021-05-01 12051 11480 10761 10458 10513 10256 10259 10189 10044  9644
    4984   1 2021-05-01 11818 11352 10641 10348 10412 10159 10166 10062  9960  9392
    4985   1 2021-05-01 11722 11214 10514 10240 10287 10043 10057  9955  9855  9297
    4986   1 2021-05-01 11670 11621 10879 10573 10625 10361 10360 10311 10166  9197
    4987   1 2021-05-01 12132 11621 10879 10573 10625 10361 10360 10311 10166  9774
    4988   1 2021-05-01 12541 11765 11034 10729 10809 10543 10525 10413 10316 10315
    4989   1 2021-05-01 12575 11949 11228 10942 11030 10751 10738 10607 10529 10383
    4990   1 2021-05-01 12350 11537 10835 10549 10629 10381 10375 10188 10179 10209
    4991   1 2021-05-01 11997 11640 10921 10615 10703 10446 10436 10276 10231  9823
    4992   1 2021-05-01 12114 11495 10770 10461 10546 10278 10270 10166 10056  9988
    4993   1 2021-05-01 12003 11507 10804 10506 10587 10332 10322 10220 10104  9858
    4994   1 2021-05-01 11834 11354 10641 10352 10428 10173 10181 10085  9981  9641
    4995   1 2021-05-01 11944 11381 10690 10410 10491 10243 10247 10121 10056  9783
    4996   1 2021-05-01 11602 11040 10343 10055 10121  9885  9894  9816  9705  9439
    4997   1 2021-05-01 11602 11130 10422 10126 10189  9942  9956  9895  9756  9413
    4998   1 2021-05-01 11513 11088 10404 10118 10183  9940  9949  9848  9751  9356
    4999   1 2021-05-01 11627 10977 10280  9976 10048  9797  9804  9684  9609  9478
    5000   1 2021-05-01 11558 10977 10280  9976 10048  9797  9804  9684  9609  9431
    5001   1 2021-05-01 11616 10958 10280  9984 10042  9795  9803  9740  9594  9469
    5002   1 2021-05-01 11524 11104 10418 10139 10200  9958  9971  9868  9769  9341
    5003   1 2021-05-01 11460 10996 10320 10038 10102  9869  9893  9824  9709  9266
    5004   1 2021-05-01 11048 10563  9939  9695  9752  9555  9617  9505  9443  8792
    5005   1 2021-05-01 10972 10648  9980  9687  9720  9495  9529  9524  9328  8630
    5006   1 2021-05-01 11019 10640  9962  9672  9707  9487  9511  9477  9318  8679
    5007   1 2021-05-01 11046 10725 10059  9768  9809  9599  9625  9572  9440  8701
    5008   1 2021-05-01 11222 10669  9999  9706  9744  9534  9571  9535  9384  8809
    5009   1 2021-05-01 11342 10983 10284  9978 10016  9790  9812  9743  9617  8966
    5010   1 2021-05-01 11537 11090 10403 10099 10138  9900  9935  9866  9733  9150
    5011   1 2021-05-01 11569 11122 10430 10132 10174  9945  9961  9880  9755  9231
    5012   1 2021-05-01 11559 11011 10350 10063 10108  9881  9910  9830  9711  9195
    5013   1 2021-05-01 11540 11131 10475 10195 10255 10031 10058  9945  9871  9190
    5014   1 2021-05-01 11688 10881 10221  9954 10000  9790  9832  9721  9652  9271
    5015   1 2021-05-01 11558 11123 10453 10177 10233 10013 10056  9933  9875  9133
    5016   1 2021-05-01 11440 11080 10397 10110 10157  9929  9962  9914  9769  9007
    5017   1 2021-05-01 11446 11066 10391 10102 10143  9922  9954  9912  9757  9030
    5018   1 2021-05-01 11581 11232 10563 10295 10350 10131 10160 10094  9968  9240
    5019   1 2021-05-01 11678 11220 10541 10260 10311 10086 10109 10056  9925  9381
    5020   1 2021-05-01 11750 11307 10613 10317 10373 10161 10172 10101  9974  9464
    5021   1 2021-05-01 11803 11341 10657 10369 10447 10218 10243 10171 10050  9542
    5022   1 2021-05-01 11740 11307 10628 10340 10410 10186 10206 10145 10015  9516
    5023   1 2021-05-01 11697 11260 10582 10302 10372 10145 10170 10113  9975  9492
    5024   1 2021-05-01 11659 11191 10528 10265 10338 10121 10147 10076  9955  9482
    5025   1 2021-05-01 11576 11024 10374 10104 10178  9973 10016  9943  9840  9434
    5026   1 2021-05-01 11433 10872 10228  9958 10030  9830  9866  9805  9693  9291
    5027   1 2021-05-01 11189 10686 10065  9804  9878  9702  9745  9680  9571  9048
    5028   1 2021-05-01 10927 10686 10065  9804  9878  9702  9745  9680  9571  8767
    5029   1 2021-05-01 10738 10470  9865  9604  9674  9506  9573  9509  9395  8541
    5030   1 2021-05-01 10853 10497  9866  9579  9651  9470  9525  9448  9341  8666
    5031   1 2021-05-01 11144 10621  9994  9725  9803  9626  9671  9608  9482  9082
    5032   1 2021-05-01 11027 10609  9983  9712  9792  9614  9667  9593  9490  8968
    5033   1 2021-05-01 11061 10630  9998  9732  9820  9637  9677  9607  9497  9030
    5034   1 2021-05-01 11125 10709 10069  9803  9887  9709  9746  9666  9565  9121
    5035   1 2021-05-01 11157 10746 10100  9820  9903  9713  9744  9722  9561  9206
    5036   1 2021-05-01 11254 10830 10183  9911 10001  9812  9840  9799  9656  9343
    5037   1 2021-05-01 11625 11242 10516 10197 10241  9966  9967  9955  9739  9293
    5038   1 2021-05-01 11559 11119 10406 10102 10144  9886  9902  9865  9694  9154
    5039   1 2021-05-01 11488 10995 10292  9997 10035  9790  9806  9755  9610  9049
    5040   1 2021-05-01 11302 11008 10309 10008 10038  9775  9801  9780  9601  8741
    5041   1 2021-05-01 11428 10908 10211  9905  9938  9681  9703  9614  9482  8866
    5042   1 2021-05-01 11365 10880 10177  9859  9887  9627  9643  9561  9424  8801
    5043   1 2021-05-01 11354 10880 10177  9859  9887  9627  9643  9561  9424  8802
    5044   1 2021-05-01 11541 11028 10314  9986 10020  9751  9760  9696  9537  9004
    5045   1 2021-05-01 11479 11031 10321  9992 10038  9775  9786  9686  9572  8814
    5046   1 2021-05-01 11453 11090 10381 10032 10049  9793  9807  9773  9586  8839
    5047   1 2021-05-01 11575 11394 10679 10336 10357 10085 10088 10103  9871  8983
    5048   1 2021-05-01 11721 11202 10492 10154 10176  9898  9914  9901  9698  9149
    5049   1 2021-05-01 11564 11222 10521 10197 10231  9971  9995  9936  9795  8969
    5050   1 2021-05-01 11566 11140 10426 10087 10114  9852  9870  9828  9659  8970
    5051   1 2021-05-01 11534 11051 10370 10058 10091  9840  9867  9797  9663  8893
    5052   1 2021-05-01 11373 10832 10162  9853  9896  9649  9671  9563  9486  8720
    5053   1 2021-05-01 11397 10664 10011  9716  9756  9524  9578  9406  9391  8849
    5054   1 2021-05-01 11463 11203 10489 10165 10212  9954  9980  9882  9757  8871
    5055   1 2021-05-01 11668 11333 10619 10293 10324 10067 10091 10065  9884  9127
    5056   1 2021-05-01 11705 11314 10614 10296 10342 10088 10105 10063  9902  9153
    5057   1 2021-05-01 11858 11511 10800 10501 10553 10290 10320 10260 10114  9393
    5058   1 2021-05-01 12025 11527 10791 10470 10521 10248 10261 10219 10046  9655
    5059   1 2021-05-01 12019 11555 10821 10516 10582 10320 10321 10236 10117  9689
    5060   1 2021-05-01 12034 11352 10641 10348 10412 10159 10166 10062  9960  9692
    5061   1 2021-05-01 11921 11387 10673 10370 10430 10175 10179 10072  9974  9510
    5062   1 2021-05-01 11863 11472 10754 10458 10513 10259 10263 10221 10064  9462
    5063   1 2021-05-01 12263 11780 11028 10701 10748 10469 10459 10488 10222  9933
    5064   1 2021-05-01 12354 12090 11343 11011 11081 10775 10746 10773 10519 10081
    5065   1 2021-05-01 12541 12037 11293 10975 11044 10761 10735 10722 10505 10339
    5066   1 2021-05-01 12495 11974 11236 10927 11006 10726 10708 10641 10483 10391
    5067   1 2021-05-01 12477 11985 11251 10940 11020 10747 10725 10693 10521 10397
    5068   1 2021-05-01 12432 11771 11058 10771 10858 10602 10590 10482 10388 10336
    5069   1 2021-05-01 12148 11645 10930 10640 10721 10459 10458 10384 10261 10018
    5070   1 2021-05-01 12002 11571 10848 10541 10610 10338 10330 10305 10112  9859
    5071   1 2021-05-01 11931 11538 10821 10519 10582 10320 10311 10306 10102  9790
    5072   1 2021-05-01 11742 11215 10554 10290 10388 10144 10163 10021  9975  9609
    5073   1 2021-05-01 11522 11130 10422 10126 10189  9942  9956  9895  9756  9339
    5074   1 2021-05-01 11815 11222 10504 10199 10265 10009 10009  9948  9805  9697
    5075   1 2021-05-01 11911 11328 10608 10309 10384 10135 10128 10027  9931  9842
    5076   1 2021-05-01 11913 11465 10739 10429 10510 10255 10246 10156 10031  9855
    5077   1 2021-05-01 11827 11319 10626 10345 10420 10174 10189 10079  9978  9740
    5078   1 2021-05-01 11726 11271 10604 10341 10407 10182 10209 10108 10019  9565
    5079   1 2021-05-01 11345 10709 10098  9865  9936  9733  9777  9621  9616  9132
    5080   1 2021-05-01 10916 10700 10053  9783  9845  9629  9656  9536  9478  8633
    5081   1 2021-05-01 11223 10692 10051  9786  9848  9632  9664  9499  9477  8934
    5082   1 2021-05-01 11157 10738 10086  9820  9885  9667  9699  9521  9512  8881
    5083   1 2021-05-01 10910 10533  9917  9657  9711  9514  9569  9412  9399  8568
    5084   1 2021-05-01 11012 10533  9917  9657  9711  9514  9569  9412  9399  8589
    5085   1 2021-05-01 11207 10808 10138  9857  9899  9682  9726  9660  9545  8796
    5086   1 2021-05-01 11462 11090 10403 10099 10138  9900  9935  9866  9733  9108
    5087   1 2021-05-01 11727 11201 10504 10193 10244  9998 10016  9923  9813  9421
    5088   1 2021-05-01 11863 11175 10485 10183 10238  9996 10008  9917  9798  9570
    5089   1 2021-05-01 11851 11391 10692 10382 10439 10187 10199 10140  9977  9539
    5090   1 2021-05-01 11851 11310 10654 10377 10446 10223 10255 10151 10070  9475
    5091   1 2021-05-01 11429 11059 10405 10127 10185  9965 10012  9889  9821  8988
    5092   1 2021-05-01 11512 10998 10321 10032 10085  9863  9900  9806  9706  9064
    5093   1 2021-05-01 11487 11060 10413 10154 10214 10008 10060  9937  9879  9049
    5094   1 2021-05-01 11450 11068 10402 10121 10167  9960  9996  9943  9808  9059
    5095   1 2021-05-01 11432 11053 10390 10118 10167  9957  9998  9961  9817  9036
    5096   1 2021-05-01 11559 11235 10557 10270 10327 10114 10149 10112  9953  9213
    5097   1 2021-05-01 11701 11235 10557 10270 10327 10114 10149 10112  9953  9401
    5098   1 2021-05-01 11720 11280 10601 10317 10388 10164 10191 10127  9995  9466
    5099   1 2021-05-01 11676 11260 10582 10302 10372 10145 10170 10113  9975  9443
    5100   1 2021-05-01 11598 11199 10527 10249 10320 10100 10121 10065  9935  9414
    5101   1 2021-05-01 11509 11078 10431 10174 10239 10034 10072  9994  9880  9353
    5102   1 2021-05-01 11399 10902 10264  9997 10066  9870  9914  9824  9733  9251
    5103   1 2021-05-01 11208 10642 10032  9772  9850  9671  9715  9638  9543  9061
    5104   1 2021-05-01 10450 10147  9580  9346  9414  9272  9346  9238  9192  8227
    5105   1 2021-05-01 10728 10474  9821  9533  9579  9410  9451  9438  9273  8491
    5106   1 2021-05-01 11032 10721 10063  9769  9829  9637  9670  9670  9480  8832
    5107   1 2021-05-01 11225 10698 10070  9804  9884  9695  9753  9659  9569  9126
    5108   1 2021-05-01 11215 10811 10164  9877  9951  9760  9796  9782  9614  9174
    5109   1 2021-05-01 11273 10876 10234  9973 10064  9879  9915  9841  9740  9289
    5110   1 2021-05-01 11320 10956 10303 10029 10117  9919  9952  9895  9771  9335
    5111   1 2021-05-01 11301 10794 10148  9871  9964  9775  9805  9721  9620  9341
    5112   1 2021-05-01 11544 11196 10437 10078 10112  9835  9822  9892  9602  9233
    5113   1 2021-05-01 11434 11149 10429 10122 10154  9888  9884  9866  9668  9084
    5114   1 2021-05-01 11313 10931 10213  9903  9929  9684  9688  9653  9486  8885
    5115   1 2021-05-01 11343 10929 10214  9913  9944  9696  9716  9653  9511  8804
    5116   1 2021-05-01 11523 11004 10267  9951  9979  9718  9729  9667  9510  9010
    5117   1 2021-05-01 11644 11020 10285  9958  9995  9726  9739  9639  9512  9168
    5118   1 2021-05-01 11583 11261 10521 10184 10242  9965  9958  9856  9744  9168
    5119   1 2021-05-01 11589 11180 10444 10115 10161  9884  9886  9798  9674  9086
    5120   1 2021-05-01 11649 11034 10321  9999 10037  9789  9801  9675  9595  9095
    5121   1 2021-05-01 11524 11007 10277  9929  9961  9705  9711  9611  9497  8954
    5122   1 2021-05-01 11540 11085 10349  9989 10022  9741  9753  9703  9536  8997
    5123   1 2021-05-01 11564 11091 10361 10008 10042  9776  9795  9725  9593  9018
    5124   1 2021-05-01 11545 11005 10301  9966  9992  9724  9741  9682  9531  8962
    5125   1 2021-05-01 11336 11005 10301  9966  9992  9724  9741  9682  9531  8668
    5126   1 2021-05-01 11240 10879 10174  9836  9857  9601  9626  9583  9420  8593
    5127   1 2021-05-01 11459 10963 10236  9886  9922  9648  9653  9619  9438  8889
    5128   1 2021-05-01 11677 11126 10421 10107 10162  9903  9917  9793  9719  9170
    5129   1 2021-05-01 11806 11316 10599 10276 10331 10068 10087  9992  9876  9322
    5130   1 2021-05-01 11669 11239 10549 10241 10295 10049 10073  9966  9860  9155
    5131   1 2021-05-01 11670 11243 10541 10229 10274 10027 10038  9959  9829  9179
    5132   1 2021-05-01 11749 11387 10661 10332 10381 10109 10118 10083  9892  9276
    5133   1 2021-05-01 12121 11682 10951 10631 10693 10413 10411 10381 10195  9739
    5134   1 2021-05-01 12118 11667 10948 10644 10708 10436 10431 10380 10218  9768
    5135   1 2021-05-01 12127 11565 10859 10567 10634 10381 10385 10265 10192  9780
    5136   1 2021-05-01 12053 11563 10857 10567 10630 10376 10377 10279 10168  9681
    5137   1 2021-05-01 11893 11590 10849 10528 10591 10320 10319 10273 10105  9493
    5138   1 2021-05-01 12137 11814 11059 10724 10787 10497 10472 10477 10249  9841
    5139   1 2021-05-01 12349 11888 11130 10801 10865 10575 10554 10549 10324 10103
    5140   1 2021-05-01 12391 11903 11158 10832 10896 10615 10589 10576 10359 10182
    5141   1 2021-05-01 12354 12000 11246 10932 10995 10712 10697 10684 10463 10176
    5142   1 2021-05-01 12286 11915 11192 10892 10962 10685 10669 10662 10436 10183
    5143   1 2021-05-01 12313 11923 11203 10913 10982 10705 10687 10687 10472 10201
    5144   1 2021-05-01 12155 11643 10930 10634 10707 10436 10426 10424 10217 10032
    5145   1 2021-05-01 11932 11452 10750 10464 10535 10287 10290 10242 10096  9795
    5146   1 2021-05-01 11726 11376 10689 10400 10476 10220 10214 10184 10010  9580
    5147   1 2021-05-01 11502 11084 10410 10131 10191  9937  9958  9935  9765  9344
    5148   1 2021-05-01 11483 11312 10584 10287 10351 10100 10094 10058  9887  9324
    5149   1 2021-05-01 11830 11492 10791 10492 10565 10312 10301 10280 10103  9695
    5150   1 2021-05-01 12073 11593 10864 10560 10636 10386 10378 10317 10172 10042
    5151   1 2021-05-01 12165 11694 10992 10718 10791 10534 10533 10489 10336 10112
    5152   1 2021-05-01 12120 11579 10883 10607 10692 10443 10450 10333 10246 10046
    5153   1 2021-05-01 11754 11335 10664 10402 10478 10246 10262 10161 10085  9619
    5154   1 2021-05-01 11392 10917 10262  9994 10053  9834  9867  9783  9691  9168
    5155   1 2021-05-01 11626 10982 10304 10014 10073  9839  9856  9770  9666  9398
    5156   1 2021-05-01 11664 11049 10358 10063 10121  9884  9899  9795  9690  9477
    5157   1 2021-05-01 11574 11213 10517 10231 10303 10065 10081  9940  9883  9344
    5158   1 2021-05-01 11359 10999 10346 10058 10123  9902  9927  9778  9742  9097
    5159   1 2021-05-01 11679 10846 10219  9951 10011  9790  9847  9697  9661  9368
    5160   1 2021-05-01 11555 11147 10458 10155 10215  9976 10013  9905  9819  9207
    5161   1 2021-05-01 11725 11269 10553 10238 10292 10049 10061  9965  9857  9419
    5162   1 2021-05-01 11892 11441 10707 10386 10442 10185 10198 10111  9987  9603
    5163   1 2021-05-01 12040 11529 10816 10507 10564 10306 10312 10242 10093  9776
    5164   1 2021-05-01 12014 11563 10867 10567 10630 10384 10399 10314 10186  9738
    5165   1 2021-05-01 11868 11264 10589 10296 10361 10128 10160 10043  9962  9527
    5166   1 2021-05-01 11762 11264 10589 10296 10361 10128 10160 10043  9962  9396
    5167   1 2021-05-01 11610 11109 10433 10158 10205  9980 10013  9928  9824  9147
    5168   1 2021-05-01 11562 11176 10522 10272 10328 10108 10160 10069  9979  9141
    5169   1 2021-05-01 11448 11017 10371 10112 10162  9958 10016  9912  9841  9032
    5170   1 2021-05-01 11380 10926 10273 10004 10049  9837  9882  9833  9700  8955
    5171   1 2021-05-01 11345 10955 10303 10032 10080  9883  9922  9883  9746  8961
    5172   1 2021-05-01 11422 11150 10478 10199 10251 10031 10076 10040  9880  9065
    5173   1 2021-05-01 11404 11093 10425 10138 10199  9973 10005  9976  9815  9071
    5174   1 2021-05-01 11554 11174 10494 10216 10279 10061 10086 10026  9890  9296
    5175   1 2021-05-01 11604 11118 10443 10163 10228 10008 10033  9970  9836  9385
    5176   1 2021-05-01 11474 10983 10342 10079 10146  9949  9990  9888  9807  9291
    5177   1 2021-05-01 11380 10803 10168  9910  9980  9797  9838  9721  9672  9180
    5178   1 2021-05-01 11039 10232  9652  9403  9469  9321  9387  9249  9224  8855
    5179   1 2021-05-01 10523 10185  9605  9368  9440  9304  9382  9208  9224  8289
    5180   1 2021-05-01 10555 10188  9560  9286  9325  9180  9232  9210  9071  8301
    5181   1 2021-05-01 10858 10427  9772  9480  9528  9356  9400  9402  9219  8678
    5182   1 2021-05-01 11041 10694 10050  9773  9836  9642  9685  9662  9499  8919
    5183   1 2021-05-01 11138 10599  9962  9694  9759  9587  9639  9595  9456  9060
    5184   1 2021-05-01 11053 10877 10227  9938 10013  9818  9853  9889  9662  9037
    5185   1 2021-05-01 11370 10956 10303 10029 10117  9919  9952  9895  9771  9404
    5186   1 2021-05-01 11443 10985 10347 10076 10174  9983 10010  9965  9831  9493
    5187   1 2021-05-01 11443 10905 10174  9866  9938  9697  9720  9556  9523  9101
    5188   1 2021-05-01 11338 10882 10165  9849  9886  9630  9666  9594  9462  8858
    5189   1 2021-05-01 11291 10871 10142  9834  9862  9622  9660  9571  9458  8796
    5190   1 2021-05-01 11417 11066 10313  9997 10032  9782  9794  9706  9582  8933
    5191   1 2021-05-01 11563 11282 10524 10192 10216  9957  9967  9934  9746  9060
    5192   1 2021-05-01 11923 11486 10721 10393 10437 10156 10156 10100  9942  9422
    5193   1 2021-05-01 11981 11378 10643 10320 10368 10083 10082 10001  9867  9514
    5194   1 2021-05-01 11811 11378 10643 10320 10368 10083 10082 10001  9867  9287
    5195   1 2021-05-01 11684 11156 10432 10114 10153  9886  9898  9791  9689  9153
    5196   1 2021-05-01 11611 11099 10370 10047 10075  9814  9817  9713  9607  9038
    5197   1 2021-05-01 11503 11043 10310  9961  9991  9722  9741  9649  9541  8841
    5198   1 2021-05-01 11400 10949 10224  9886  9912  9651  9676  9587  9471  8698
    5199   1 2021-05-01 11479 10974 10255  9905  9935  9663  9682  9619  9468  8901
    5200   1 2021-05-01 11310 10889 10165  9806  9837  9559  9568  9511  9331  8693
    5201   1 2021-05-01 11660 11080 10331  9966 10011  9723  9727  9672  9507  9148
    5202   1 2021-05-01 11781 11303 10551 10193 10246  9965  9963  9903  9741  9333
    5203   1 2021-05-01 11885 11398 10676 10356 10408 10148 10162 10076  9945  9436
    5204   1 2021-05-01 11823 11332 10626 10316 10368 10114 10135 10045  9934  9365
    5205   1 2021-05-01 11859 11327 10616 10298 10360 10104 10117 10016  9913  9436
    5206   1 2021-05-01 11839 11350 10645 10334 10388 10140 10154 10041  9938  9421
    5207   1 2021-05-01 11857 11403 10698 10377 10445 10191 10202 10091  9991  9484
    5208   1 2021-05-01 11880 11518 10787 10453 10502 10231 10223 10223 10006  9542
    5209   1 2021-05-01 12008 11565 10836 10509 10563 10287 10277 10263 10061  9699
    5210   1 2021-05-01 11944 11602 10887 10577 10635 10358 10360 10329 10144  9591
    5211   1 2021-05-01 12052 11542 10819 10504 10560 10294 10292 10251 10075  9671
    5212   1 2021-05-01 12022 11598 10866 10545 10608 10339 10330 10283 10122  9715
    5213   1 2021-05-01 12095 11691 10932 10585 10647 10360 10338 10332 10116  9809
    5214   1 2021-05-01 12331 11886 11139 10816 10884 10601 10579 10548 10353 10113
    5215   1 2021-05-01 12365 11848 11109 10798 10869 10595 10580 10521 10368 10180
    5216   1 2021-05-01 12274 11775 11037 10721 10791 10514 10487 10471 10263 10149
    5217   1 2021-05-01 12180 11592 10857 10538 10619 10359 10339 10271 10131 10070
    5218   1 2021-05-01 11990 11519 10803 10498 10574 10311 10302 10260 10087  9899
    5219   1 2021-05-01 11851 11324 10629 10342 10405 10154 10151 10140  9947  9770
    5220   1 2021-05-01 11616 11019 10339 10056 10113  9870  9887  9856  9688  9507
    5221   1 2021-05-01 11510 11235 10536 10249 10305 10068 10070 10042  9875  9337
    5222   1 2021-05-01 11197 10907 10230  9939  9997  9749  9767  9732  9566  9041
    5223   1 2021-05-01 11335 11130 10409 10112 10163  9908  9913  9929  9723  9179
    5224   1 2021-05-01 11665 11233 10516 10214 10271 10015 10014 10048  9813  9575
    5225   1 2021-05-01 11962 11564 10849 10547 10615 10360 10343 10342 10142  9921
    5226   1 2021-05-01 11896 11566 10855 10570 10640 10378 10369 10373 10176  9813
    5227   1 2021-05-01 11819 11432 10759 10493 10567 10322 10340 10263 10151  9692
    5228   1 2021-05-01 11577 11106 10446 10188 10255 10021 10047  9949  9869  9407
    5229   1 2021-05-01 11332 11070 10381 10085 10133  9888  9915  9860  9717  9056
    5230   1 2021-05-01 11667 11383 10664 10357 10415 10157 10173 10120  9960  9416
    5231   1 2021-05-01 11947 11445 10731 10426 10492 10234 10241 10170 10036  9765
    5232   1 2021-05-01 11967 11408 10712 10410 10487 10251 10264 10133 10062  9729
    5233   1 2021-05-01 12026 11497 10812 10506 10577 10328 10335 10260 10133  9832
    5234   1 2021-05-01 11918 11291 10617 10334 10403 10169 10206 10087 10008  9666
    5235   1 2021-05-01 11871 11291 10617 10334 10403 10169 10206 10087 10008  9551
    5236   1 2021-05-01 12072 11436 10720 10413 10477 10234 10240 10125 10039  9817
    5237   1 2021-05-01 12217 11647 10927 10630 10693 10451 10461 10341 10262  9959
    5238   1 2021-05-01 12162 11640 10910 10592 10650 10400 10395 10331 10196  9912
    5239   1 2021-05-01 12150 11728 11020 10715 10775 10528 10533 10463 10328  9849
    5240   1 2021-05-01 12137 11501 10819 10522 10592 10353 10377 10258 10175  9819
    5241   1 2021-05-01 11873 11358 10672 10381 10449 10223 10251 10097 10052  9517
    5242   1 2021-05-01 11591 11109 10452 10187 10223 10009 10048  9998  9864  9144
    5243   1 2021-05-01 11512 10759 10147  9908  9941  9757  9829  9718  9657  9030
    5244   1 2021-05-01 11015 10704 10043  9758  9787  9581  9629  9582  9435  8489
    5245   1 2021-05-01 11153 10816 10164  9894  9934  9731  9765  9734  9580  8673
    5246   1 2021-05-01 11195 10696 10052  9778  9816  9616  9667  9612  9486  8753
    5247   1 2021-05-01 11131 10866 10212  9922  9973  9770  9809  9765  9628  8681
    5248   1 2021-05-01 11310 10866 10212  9922  9973  9770  9809  9765  9628  8978
    5249   1 2021-05-01 11463 11067 10393 10107 10171  9943  9973  9932  9785  9207
    5250   1 2021-05-01 11525 11060 10380 10086 10144  9921  9953  9905  9761  9304
    5251   1 2021-05-01 11522 11053 10396 10121 10185  9977 10004  9938  9820  9305
    5252   1 2021-05-01 11293 10914 10269  9991 10058  9858  9887  9818  9705  9043
    5253   1 2021-05-01 11226 10665 10054  9800  9873  9699  9743  9588  9576  8983
    5254   1 2021-05-01 11212 10648 10049  9804  9884  9710  9771  9616  9605  9008
    5255   1 2021-05-01 10632 10190  9568  9307  9369  9223  9296  9219  9139  8397
    5256   1 2021-05-01 10717 10313  9700  9439  9495  9337  9392  9340  9228  8516
    5257   1 2021-05-01 10901 10515  9893  9623  9684  9513  9559  9508  9382  8756
    5258   1 2021-05-01 11018 10534  9910  9634  9708  9535  9588  9566  9421  8947
    5259   1 2021-05-01 10902 10694 10058  9785  9859  9677  9718  9693  9542  8854
    5260   1 2021-05-01 11139 10854 10222  9946 10029  9847  9887  9875  9720  9151
    5261   1 2021-05-01 11544 11078 10393 10101 10144  9892  9890  9767  9672  9063
    5262   1 2021-05-01 11513 10714 10024  9730  9754  9504  9517  9425  9311  9001
    5263   1 2021-05-01 11342 10862 10143  9842  9892  9649  9685  9550  9486  8789
    5264   1 2021-05-01 11251 10632  9914  9608  9621  9383  9425  9359  9225  8607
    5265   1 2021-05-01 11341 11020 10265  9936  9961  9698  9712  9695  9486  8734
    5266   1 2021-05-01 11520 11282 10524 10192 10216  9957  9967  9934  9746  8880
    5267   1 2021-05-01 11756 11254 10494 10160 10169  9899  9907  9901  9693  9109
    5268   1 2021-05-01 11603 11334 10582 10242 10273  9989  9987  9961  9761  8923
    5269   1 2021-05-01 11472 11016 10260  9901  9920  9647  9637  9603  9410  8955
    5270   1 2021-05-01 11451 11049 10297  9948  9960  9687  9674  9665  9452  8880
    5271   1 2021-05-01 11518 10874 10134  9772  9787  9514  9510  9471  9280  8840
    5272   1 2021-05-01 11404 10810 10107  9765  9789  9530  9553  9433  9330  8705
    5273   1 2021-05-01 11370 10827 10091  9728  9764  9492  9498  9368  9263  8777
    5274   1 2021-05-01 11333 10839 10075  9684  9712  9439  9432  9366  9200  8814
    5275   1 2021-05-01 11432 11131 10367  9987 10020  9742  9728  9665  9504  8910
    5276   1 2021-05-01 11650 11147 10390 10021 10059  9769  9764  9722  9532  9095
    5277   1 2021-05-01 11924 11332 10566 10199 10244  9961  9946  9902  9726  9453
    5278   1 2021-05-01 11952 11487 10749 10414 10465 10193 10193 10144  9969  9471
    5279   1 2021-05-01 11810 11332 10626 10316 10368 10114 10135 10045  9934  9349
    5280   1 2021-05-01 11884 11444 10719 10392 10445 10179 10177 10132  9965  9453
    5281   1 2021-05-01 11961 11448 10727 10412 10466 10211 10215 10144 10005  9581
    5282   1 2021-05-01 11970 11498 10788 10479 10550 10292 10305 10210 10099  9604
    5283   1 2021-05-01 11926 11459 10743 10414 10482 10209 10215 10149 10007  9601
    5284   1 2021-05-01 11947 11427 10708 10386 10449 10180 10180 10122  9968  9629
    5285   1 2021-05-01 11940 11468 10757 10453 10515 10245 10251 10184 10038  9620
    5286   1 2021-05-01 12048 11571 10845 10531 10599 10334 10337 10253 10130  9759
    5287   1 2021-05-01 12120 11666 10916 10580 10652 10377 10367 10307 10155  9874
    5288   1 2021-05-01 12175 11769 11018 10694 10765 10494 10486 10425 10273  9946
    5289   1 2021-05-01 12311 11893 11145 10841 10902 10629 10614 10589 10397 10070
    5290   1 2021-05-01 12373 11893 11145 10841 10902 10629 10614 10589 10397 10214
    5291   1 2021-05-01 12364 11826 11089 10778 10859 10584 10558 10495 10352 10253
    5292   1 2021-05-01 12157 11592 10857 10538 10619 10359 10339 10271 10131 10071
    5293   1 2021-05-01 12180 11587 10863 10546 10636 10369 10355 10295 10148 10116
    5294   1 2021-05-01 11864 11321 10628 10343 10426 10182 10180 10089  9976  9806
    5295   1 2021-05-01 11520 11088 10414 10136 10219  9970  9991  9868  9792  9424
    5296   1 2021-05-01 11300 10838 10160  9867  9932  9679  9704  9634  9494  9132
    5297   1 2021-05-01 11310 10807 10140  9867  9932  9703  9729  9632  9538  9163
    5298   1 2021-05-01 11383 11000 10302 10014 10075  9832  9837  9813  9649  9248
    5299   1 2021-05-01 11516 11100 10397 10106 10170  9927  9930  9898  9733  9431
    5300   1 2021-05-01 11637 11337 10626 10332 10391 10139 10132 10142  9933  9550
    5301   1 2021-05-01 11733 11224 10525 10244 10307 10065 10066 10021  9876  9669
    5302   1 2021-05-01 11708 11089 10417 10143 10204  9959  9984  9933  9791  9584
    5303   1 2021-05-01 11385 11089 10417 10143 10204  9959  9984  9933  9791  9161
    5304   1 2021-05-01 11466 11018 10338 10054 10099  9864  9899  9828  9698  9202
    5305   1 2021-05-01 11785 11383 10664 10357 10415 10157 10173 10120  9960  9546
    5306   1 2021-05-01 11998 11557 10839 10527 10590 10334 10341 10288 10128  9812
    5307   1 2021-05-01 12074 11583 10871 10564 10634 10385 10389 10319 10184  9855
    5308   1 2021-05-01 12013 11564 10849 10532 10603 10341 10346 10289 10134  9807
    5309   1 2021-05-01 11917 11435 10755 10472 10536 10294 10312 10237 10119  9653
    5310   1 2021-05-01 11882 11541 10828 10527 10580 10339 10353 10299 10145  9546
    5311   1 2021-05-01 12157 11750 11031 10727 10790 10536 10536 10497 10335  9853
    5312   1 2021-05-01 12213 11757 11025 10710 10769 10512 10516 10498 10317  9931
    5313   1 2021-05-01 12243 11785 11068 10756 10816 10566 10564 10529 10352  9947
    5314   1 2021-05-01 12203 11685 10978 10679 10730 10490 10500 10453 10302  9889
    5315   1 2021-05-01 12092 11599 10904 10623 10687 10453 10470 10380 10287  9689
    5316   1 2021-05-01 11891 11271 10622 10361 10418 10204 10258 10121 10082  9448
    5317   1 2021-05-01 11390 11100 10450 10175 10223 10010 10064  9933  9880  8880
    5318   1 2021-05-01 11079 10759 10147  9908  9941  9757  9829  9718  9657  8463
    5319   1 2021-05-01 11104 10676 10027  9764  9800  9608  9666  9523  9487  8523
    5320   1 2021-05-01 11274 10686 10018  9727  9774  9566  9599  9516  9410  8790
    5321   1 2021-05-01 11202 10648  9999  9731  9779  9592  9649  9509  9476  8692
    5322   1 2021-05-01 11007 10720 10059  9772  9817  9623  9668  9608  9483  8514
    5323   1 2021-05-01 11240 10852 10165  9863  9910  9694  9720  9679  9518  8858
    5324   1 2021-05-01 11438 10978 10302 10008 10056  9839  9871  9807  9678  9164
    5325   1 2021-05-01 11471 11002 10328 10040 10100  9888  9928  9855  9747  9192
    5326   1 2021-05-01 11362 10788 10145  9878  9941  9751  9798  9715  9633  9072
    5327   1 2021-05-01 11123 10553  9923  9651  9699  9521  9583  9520  9408  8810
    5328   1 2021-05-01 11266 10837 10212  9947 10010  9818  9863  9797  9672  9054
    5329   1 2021-05-01 11162 10785 10165  9906  9975  9790  9826  9723  9653  8978
    5330   1 2021-05-01 10703 10490  9891  9649  9721  9561  9618  9496  9453  8534
    5331   1 2021-05-01 10752 10313  9700  9439  9495  9337  9392  9340  9228  8513
    5332   1 2021-05-01 11357 10612  9988  9734  9808  9634  9678  9582  9516  9289
    5333   1 2021-05-01 11193 10634 10033  9788  9864  9700  9748  9650  9583  9128
    5334   1 2021-05-01 11236 10815 10181  9909  9985  9806  9852  9787  9678  9170
    5335   1 2021-05-01 11126 10687 10071  9810  9889  9725  9774  9716  9612  9146
    5336   1 2021-05-01 10960 10506  9815  9517  9546  9308  9338  9200  9127  8348
    5337   1 2021-05-01 11049 10708  9969  9650  9670  9422  9451  9391  9233  8463
    5338   1 2021-05-01 11185 10848 10112  9775  9806  9541  9562  9501  9339  8578
    5339   1 2021-05-01 11296 10851 10123  9798  9816  9567  9598  9502  9382  8670
    5340   1 2021-05-01 11374 10908 10166  9838  9868  9614  9644  9512  9415  8730
    5341   1 2021-05-01 11443 10941 10189  9856  9897  9637  9648  9502  9431  8802
    5342   1 2021-05-01 11482 11132 10394 10086 10144  9879  9899  9673  9682  8977
    5343   1 2021-05-01 11426 10851 10091  9728  9759  9484  9485  9394  9263  8943
    5344   1 2021-05-01 11533 10850 10104  9754  9795  9529  9536  9370  9325  9058
    5345   1 2021-05-01 11385 10780 10031  9655  9679  9410  9405  9324  9172  8840
    5346   1 2021-05-01 11429 10997 10234  9862  9893  9612  9603  9509  9368  8890
    5347   1 2021-05-01 11574 10947 10181  9812  9862  9593  9590  9437  9361  9079
    5348   1 2021-05-01 11798 11250 10479 10105 10150  9874  9857  9758  9627  9254
    5349   1 2021-05-01 11895 11400 10643 10284 10328 10046 10033  9968  9813  9302
    5350   1 2021-05-01 11907 11471 10716 10363 10405 10122 10109 10093  9883  9404
    5351   1 2021-05-01 11708 11335 10589 10229 10274  9986  9969  9970  9746  9251
    5352   1 2021-05-01 11826 11401 10657 10312 10360 10087 10082 10045  9869  9408
    5353   1 2021-05-01 11873 11384 10648 10293 10336 10062 10054 10063  9829  9498
    5354   1 2021-05-01 11952 11451 10733 10408 10464 10200 10207 10181  9996  9549
    5355   1 2021-05-01 11836 11285 10575 10247 10295 10032 10036 10019  9824  9422
    5356   1 2021-05-01 11840 11421 10709 10390 10453 10193 10193 10120  9981  9499
    5357   1 2021-05-01 11899 11421 10709 10390 10453 10193 10193 10120  9981  9632
    5358   1 2021-05-01 11918 11446 10715 10396 10451 10185 10182 10149  9959  9616
    5359   1 2021-05-01 11962 11522 10793 10489 10543 10275 10281 10264 10073  9633
    5360   1 2021-05-01 12130 11676 10930 10606 10673 10397 10395 10350 10175  9872
    5361   1 2021-05-01 12244 11665 10917 10598 10648 10373 10361 10371 10131 10025
    5362   1 2021-05-01 12170 11771 11027 10708 10769 10502 10491 10479 10273  9962
    5363   1 2021-05-01 12245 11917 11165 10849 10917 10640 10614 10618 10403 10055
    5364   1 2021-05-01 12383 11933 11188 10881 10949 10671 10647 10654 10434 10276
    5365   1 2021-05-01 12315 11831 11112 10817 10912 10641 10627 10550 10424 10222
    5366   1 2021-05-01 12305 11720 11017 10727 10813 10548 10544 10485 10343 10264
    5367   1 2021-05-01 12037 11406 10721 10430 10525 10276 10283 10179 10085  9978
    5368   1 2021-05-01 11781 11296 10610 10336 10412 10173 10176 10095  9971  9682
    5369   1 2021-05-01 11662 10994 10332 10071 10135  9910  9934  9808  9756  9555
    5370   1 2021-05-01 11386 10994 10332 10071 10135  9910  9934  9808  9756  9252
    5371   1 2021-05-01 11203 10838 10157  9877  9925  9703  9725  9694  9547  9036
    5372   1 2021-05-01 11263 10958 10260  9970 10014  9786  9794  9810  9609  9099
    5373   1 2021-05-01 11559 11212 10519 10242 10304 10061 10072 10037  9876  9432
    5374   1 2021-05-01 11623 11172 10471 10190 10248 10009 10014  9981  9821  9541
    5375   1 2021-05-01 11541 11052 10368 10085 10148  9910  9927  9845  9730  9409
    5376   1 2021-05-01 11415 10938 10272  9988 10052  9812  9841  9772  9635  9255
    5377   1 2021-05-01 11524 11073 10403 10120 10181  9937  9962  9903  9760  9304
    5378   1 2021-05-01 11706 11408 10690 10378 10430 10174 10182 10181  9958  9495
    5379   1 2021-05-01 11787 11479 10760 10446 10502 10242 10253 10256 10026  9577
    5380   1 2021-05-01 12007 11562 10851 10552 10615 10360 10382 10317 10169  9757
    5381   1 2021-05-01 11894 11388 10686 10386 10442 10194 10226 10165 10020  9581
    5382   1 2021-05-01 11815 11446 10741 10437 10483 10230 10249 10235 10044  9494
    5383   1 2021-05-01 11855 11527 10822 10512 10567 10306 10323 10305 10113  9512
    5384   1 2021-05-01 12056 11676 10967 10661 10723 10463 10468 10449 10270  9718
    5385   1 2021-05-01 12095 11648 10927 10618 10680 10419 10431 10415 10229  9745
    5386   1 2021-05-01 12164 11744 11022 10714 10769 10518 10525 10502 10327  9825
    5387   1 2021-05-01 12042 11599 10896 10602 10646 10409 10421 10383 10215  9653
    5388   1 2021-05-01 11932 11513 10814 10526 10572 10337 10360 10339 10177  9493
    5389   1 2021-05-01 11907 11271 10622 10361 10418 10204 10258 10121 10082  9433
    5390   1 2021-05-01 11210 10745 10125  9878  9916  9731  9798  9651  9645  8568
    5391   1 2021-05-01 10968 10684 10034  9771  9798  9605  9670  9579  9501  8305
    5392   1 2021-05-01 11202 10822 10142  9848  9866  9658  9700  9667  9518  8570
    5393   1 2021-05-01 11436 10843 10205  9938  9984  9789  9832  9715  9656  8928
    5394   1 2021-05-01 11058 10553  9942  9689  9745  9566  9639  9473  9467  8552
    5395   1 2021-05-01 11026 10809 10130  9827  9872  9658  9705  9636  9514  8469
    5396   1 2021-05-01 11363 11053 10368 10071 10130  9908  9947  9869  9765  8990
    5397   1 2021-05-01 11534 10995 10311 10011 10061  9843  9867  9800  9677  9176
    5398   1 2021-05-01 11253 11001 10344 10075 10134  9936  9982  9881  9809  8879
    5399   1 2021-05-01 11168 10776 10147  9895  9962  9779  9846  9726  9675  8832
    5400   1 2021-05-01 10976 10735 10119  9860  9924  9744  9800  9651  9623  8678
    5401   1 2021-05-01 11288 10870 10228  9944 10003  9800  9838  9791  9644  9035
    5402   1 2021-05-01 11381 10785 10165  9906  9975  9790  9826  9723  9653  9199
    5403   1 2021-05-01 11354 10929 10297 10029 10094  9900  9930  9861  9748  9207
    5404   1 2021-05-01 11444 10907 10263  9995 10073  9864  9899  9837  9712  9348
    5405   1 2021-05-01 11452 11036 10395 10131 10216 10015 10043  9957  9861  9391
    5406   1 2021-05-01 11326 10769 10160  9901  9982  9809  9856  9793  9694  9294
    5407   1 2021-05-01 11116 10772 10141  9862  9947  9765  9796  9728  9627  9080
    5408   1 2021-05-01 11174 10600 10004  9757  9844  9685  9716  9647  9560  9173
    5409   1 2021-05-01 11365 10719  9995  9671  9708  9456  9477  9362  9260  8826
    5410   1 2021-05-01 11361 10863 10111  9764  9802  9526  9546  9460  9312  8857
    5411   1 2021-05-01 11413 10938 10198  9872  9898  9644  9656  9555  9426  8829
    5412   1 2021-05-01 11682 11096 10347 10024 10056  9796  9803  9683  9561  9085
    5413   1 2021-05-01 11833 11212 10452 10131 10175  9917  9925  9781  9693  9279
    5414   1 2021-05-01 11958 11522 10763 10441 10484 10200 10194 10089  9969  9408
    5415   1 2021-05-01 12097 11277 10537 10231 10292 10025 10036  9797  9819  9528
    5416   1 2021-05-01 12061 11489 10733 10407 10463 10178 10173 10010  9940  9570
    5417   1 2021-05-01 11835 11254 10486 10139 10194  9906  9911  9755  9683  9357
    5418   1 2021-05-01 11821 11523 10756 10408 10439 10142 10139 10053  9894  9271
    5419   1 2021-05-01 11963 11378 10617 10245 10287 10000  9992  9899  9764  9382
    5420   1 2021-05-01 11779 11370 10605 10226 10254  9961  9942  9931  9699  9215
    5421   1 2021-05-01 11801 11215 10457 10081 10093  9811  9801  9791  9565  9153
    5422   1 2021-05-01 11587 11370 10623 10263 10302 10016 10007  9965  9771  9028
    5423   1 2021-05-01 11627 11187 10425 10042 10078  9790  9775  9762  9537  9186
    5424   1 2021-05-01 11724 11347 10596 10228 10275  9985  9966  9966  9729  9312
    5425   1 2021-05-01 11714 11291 10560 10215 10267  9987  9980  9928  9748  9320
    5426   1 2021-05-01 11602 11216 10489 10143 10192  9923  9922  9907  9699  9200
    5427   1 2021-05-01 11565 11070 10349 10007 10056  9793  9791  9749  9576  9169
    5428   1 2021-05-01 11601 11202 10483 10152 10197  9936  9932  9933  9727  9226
    5429   1 2021-05-01 11493 11154 10430 10098 10124  9862  9867  9899  9658  9092
    5430   1 2021-05-01 11673 11254 10535 10233 10284 10030 10034 10005  9828  9332
    5431   1 2021-05-01 11691 11185 10449 10132 10179  9909  9916  9895  9695  9364
    5432   1 2021-05-01 11903 11398 10649 10321 10373 10094 10078 10087  9855  9633
    5433   1 2021-05-01 11857 11555 10817 10500 10559 10281 10273 10285 10055  9620
    5434   1 2021-05-01 11887 11490 10752 10423 10493 10207 10201 10211  9979  9680
    5435   1 2021-05-01 12029 11671 10937 10620 10683 10403 10381 10413 10158  9862
    5436   1 2021-05-01 11971 11671 10937 10620 10683 10403 10381 10413 10158  9799
    5437   1 2021-05-01 11876 11821 11073 10760 10821 10541 10513 10557 10296  9745
    5438   1 2021-05-01 11921 11583 10850 10530 10588 10317 10291 10334 10066  9838
    5439   1 2021-05-01 11843 11477 10779 10481 10557 10297 10294 10276 10090  9772
    5440   1 2021-05-01 11691 11232 10537 10244 10310 10059 10063 10027  9858  9630
    5441   1 2021-05-01 11614 11176 10489 10209 10271 10030 10033  9976  9831  9531
    5442   1 2021-05-01 11446 11013 10343 10069 10145  9910  9927  9843  9740  9319
    5443   1 2021-05-01 11116 10693 10035  9774  9829  9614  9646  9571  9480  8933
    5444   1 2021-05-01 11001 10563  9907  9636  9688  9468  9504  9471  9331  8794
    5445   1 2021-05-01 11258 10968 10281  9994 10041  9803  9826  9829  9629  9070
    5446   1 2021-05-01 11543 11097 10408 10112 10178  9942  9946  9883  9743  9449
    5447   1 2021-05-01 11519 11009 10327 10031 10090  9860  9870  9792  9675  9414
    5448   1 2021-05-01 11456 10993 10323 10032 10078  9849  9880  9823  9673  9283
    5449   1 2021-05-01 11464 11110 10417 10121 10170  9916  9943  9916  9737  9233
    5450   1 2021-05-01 11750 11203 10501 10204 10258 10006 10025  9985  9819  9539
    5451   1 2021-05-01 11838 11324 10626 10328 10389 10148 10160 10091  9958  9600
    5452   1 2021-05-01 11813 11420 10713 10415 10468 10217 10233 10207 10029  9550
    5453   1 2021-05-01 11763 11258 10565 10269 10325 10090 10115 10044  9916  9431
    5454   1 2021-05-01 11758 11339 10639 10339 10379 10134 10156 10120  9947  9425
    5455   1 2021-05-01 11826 11399 10699 10398 10443 10205 10219 10180 10017  9468
    5456   1 2021-05-01 11874 11440 10745 10439 10497 10244 10267 10220 10061  9489
    5457   1 2021-05-01 11917 11325 10626 10322 10367 10127 10148 10106  9949  9530
    5458   1 2021-05-01 11823 11390 10680 10364 10416 10169 10182 10153  9977  9424
    5459   1 2021-05-01 11746 11266 10592 10312 10353 10136 10163 10097  9971  9325
    5460   1 2021-05-01 11498 11001 10350 10079 10115  9913  9965  9863  9775  8979
    5461   1 2021-05-01 11353 10790 10148  9891  9920  9718  9789  9698  9619  8780
    5462   1 2021-05-01 11135 10813 10159  9876  9901  9693  9753  9677  9572  8495
    5463   1 2021-05-01 11300 10648 10029  9777  9822  9634  9706  9548  9538  8700
    5464   1 2021-05-01 11207 10758 10104  9822  9868  9670  9724  9573  9543  8656
    5465   1 2021-05-01 11338 10843 10164  9855  9890  9673  9702  9659  9507  8852
    5466   1 2021-05-01 11295 10792 10143  9861  9909  9707  9756  9641  9576  8791
    5467   1 2021-05-01 11206 10711 10063  9786  9824  9632  9691  9596  9509  8697
    5468   1 2021-05-01 11447 11020 10356 10086 10145  9936  9988  9921  9817  8964
    5469   1 2021-05-01 11466 11042 10399 10135 10188  9997 10048  9976  9870  9069
    5470   1 2021-05-01 11233 10744 10130  9873  9921  9746  9806  9716  9631  8831
    5471   1 2021-05-01 11440 11000 10355 10077 10141  9941  9983  9901  9797  9130
    5472   1 2021-05-01 11631 10989 10328 10040 10104  9899  9930  9832  9738  9412
    5473   1 2021-05-01 11545 11078 10414 10130 10200  9991 10014  9936  9832  9355
    5474   1 2021-05-01 11444 11090 10433 10162 10236 10030 10058  9987  9878  9273
    5475   1 2021-05-01 11596 11243 10574 10292 10368 10160 10182 10128 10003  9443
    5476   1 2021-05-01 11688 11231 10575 10303 10391 10177 10207 10148 10019  9605
    5477   1 2021-05-01 11665 11231 10575 10303 10391 10177 10207 10148 10019  9600
    5478   1 2021-05-01 11496 10994 10370 10113 10208 10032 10059  9934  9896  9438
    5479   1 2021-05-01 11483 11028 10384 10108 10191 10005 10033  9950  9862  9425
    5480   1 2021-05-01 11507 11157 10374 10016 10033  9758  9736  9706  9495  9032
    5481   1 2021-05-01 11670 11372 10580 10222 10236  9942  9935  9904  9699  9065
    5482   1 2021-05-01 11977 11517 10730 10374 10408 10111 10107 10057  9865  9404
    5483   1 2021-05-01 11963 11561 10762 10405 10446 10151 10129 10073  9882  9432
    5484   1 2021-05-01 12136 11659 10895 10549 10588 10292 10287 10200 10046  9559
    5485   1 2021-05-01 12177 11659 10895 10549 10588 10292 10287 10200 10046  9583
    5486   1 2021-05-01 12079 11586 10811 10460 10508 10208 10202 10113  9960  9456
    5487   1 2021-05-01 11848 11523 10756 10408 10439 10142 10139 10053  9894  9206
    5488   1 2021-05-01 11860 11420 10649 10279 10296 10001  9983  9949  9743  9198
    5489   1 2021-05-01 11784 11226 10474 10096 10132  9842  9840  9753  9602  9154
    5490   1 2021-05-01 11388 11002 10242  9854  9879  9599  9590  9536  9345  8842
    5491   1 2021-05-01 11359 11052 10260  9857  9869  9577  9564  9576  9317  8917
    5492   1 2021-05-01 11566 11105 10308  9913  9939  9656  9641  9656  9403  9140
    5493   1 2021-05-01 11592 11207 10450 10073 10110  9811  9789  9832  9542  9191
    5494   1 2021-05-01 11526 11050 10303  9946  9986  9702  9694  9701  9465  9067
    5495   1 2021-05-01 11450 11018 10276  9932  9983  9724  9727  9670  9512  9016
    5496   1 2021-05-01 11480 10942 10218  9891  9952  9707  9717  9631  9516  9112
    5497   1 2021-05-01 11396 10846 10139  9810  9847  9606  9619  9570  9409  9069
    5498   1 2021-05-01 11311 10818 10109  9796  9824  9580  9587  9584  9383  8957
    5499   1 2021-05-01 11437 11077 10368 10060 10095  9842  9849  9834  9641  9085
    5500   1 2021-05-01 11591 11185 10449 10132 10179  9909  9916  9895  9695  9304
    5501   1 2021-05-01 11655 11266 10517 10190 10232  9961  9954  9952  9724  9396
    5502   1 2021-05-01 11820 11372 10641 10321 10381 10113 10103 10080  9892  9577
    5503   1 2021-05-01 11695 11278 10552 10236 10290 10023 10024 10027  9821  9448
    5504   1 2021-05-01 11833 11434 10710 10393 10458 10186 10177 10171  9961  9642
    5505   1 2021-05-01 11799 11358 10616 10300 10360 10095 10092 10059  9864  9605
    5506   1 2021-05-01 11857 11400 10679 10361 10434 10172 10161 10103  9941  9717
    5507   1 2021-05-01 11827 11395 10680 10376 10449 10201 10188 10110  9981  9727
    5508   1 2021-05-01 11876 11317 10600 10301 10374 10121 10117 10049  9905  9776
    5509   1 2021-05-01 11783 11289 10582 10286 10361 10121 10125 10020  9922  9700
    5510   1 2021-05-01 11628 10983 10293 10004 10072  9838  9842  9776  9637  9537
    5511   1 2021-05-01 11467 10637  9993  9734  9792  9585  9606  9510  9429  9337
    5512   1 2021-05-01 11012 10699 10042  9777  9836  9617  9648  9571  9470  8812
    5513   1 2021-05-01 10969 10563  9907  9636  9688  9468  9504  9471  9331  8782
    5514   1 2021-05-01 11124 10704 10030  9735  9781  9544  9569  9569  9380  8929
    5515   1 2021-05-01 11316 11021 10340 10041 10094  9866  9871  9841  9675  9144
    5516   1 2021-05-01 11397 11002 10318 10018 10069  9834  9843  9806  9655  9241
    5517   1 2021-05-01 11314 10922 10256  9973 10023  9800  9831  9764  9632  9107
    5518   1 2021-05-01 11430 10970 10282  9989 10034  9794  9816  9796  9618  9193
    5519   1 2021-05-01 11567 11276 10574 10275 10332 10080 10088 10067  9892  9372
    5520   1 2021-05-01 11807 11327 10636 10342 10412 10157 10182 10114  9974  9576
    5521   1 2021-05-01 11787 11290 10602 10303 10362 10127 10152 10058  9960  9492
    5522   1 2021-05-01 11824 11352 10658 10364 10411 10173 10198 10118 10002  9513
    5523   1 2021-05-01 11795 11351 10645 10344 10395 10154 10179 10127  9987  9464
    5524   1 2021-05-01 11828 11355 10656 10357 10408 10176 10205 10148 10011  9469
    5525   1 2021-05-01 11784 11304 10599 10300 10352 10110 10141 10058  9940  9390
    5526   1 2021-05-01 11758 11325 10626 10322 10367 10127 10148 10106  9949  9363
    5527   1 2021-05-01 11764 11284 10573 10265 10309 10077 10094 10027  9897  9354
    5528   1 2021-05-01 11769 11263 10570 10277 10324 10091 10111 10031  9915  9328
    5529   1 2021-05-01 11586 11146 10472 10198 10251 10039 10082  9941  9898  9022
    5530   1 2021-05-01 11357 10791 10150  9885  9927  9723  9795  9661  9618  8803
    5531   1 2021-05-01 11244 10815 10163  9889  9928  9719  9781  9658  9609  8652
    5532   1 2021-05-01 11357 10990 10318 10032 10074  9858  9904  9789  9709  8773
    5533   1 2021-05-01 11665 11132 10446 10160 10211  9993 10023  9910  9845  9181
    5534   1 2021-05-01 11524 10919 10246  9956 10002  9789  9829  9730  9637  9026
    5535   1 2021-05-01 11200 10704 10080  9813  9848  9666  9729  9653  9558  8630
    5536   1 2021-05-01 11149 10979 10325 10059 10101  9901  9959  9909  9790  8515
    5537   1 2021-05-01 11267 10848 10212  9947  9990  9799  9857  9805  9703  8829
    5538   1 2021-05-01 11133 10752 10126  9858  9909  9724  9787  9685  9612  8724
    5539   1 2021-05-01 11148 10744 10130  9873  9921  9746  9806  9716  9631  8744
    5540   1 2021-05-01 11344 10943 10302 10024 10081  9878  9921  9881  9739  8966
    5541   1 2021-05-01 11620 11231 10570 10289 10363 10150 10180 10092  9996  9324
    5542   1 2021-05-01 11796 11328 10662 10376 10450 10234 10249 10193 10082  9583
    5543   1 2021-05-01 11701 11251 10580 10300 10381 10163 10181 10121  9993  9579
    5544   1 2021-05-01 11761 11337 10671 10383 10463 10239 10255 10231 10062  9630
    5545   1 2021-05-01 11698 11259 10594 10305 10385 10172 10189 10182 10011  9600
    5546   1 2021-05-01 11493 11062 10415 10145 10220 10017 10059 10025  9871  9396
    5547   1 2021-05-01 11352 11050 10401 10140 10207 10018 10056 10025  9875  9280
    5548   1 2021-05-01 11441 10955 10318 10052 10127  9942  9968  9931  9793  9406
    5549   1 2021-05-01 11722 11236 10445 10080 10083  9784  9762  9750  9524  9060
    5550   1 2021-05-01 11920 11476 10673 10306 10321 10019 10012  9993  9765  9289
    5551   1 2021-05-01 12024 11576 10771 10392 10416 10110 10075 10082  9810  9490
    5552   1 2021-05-01 12011 11601 10806 10433 10467 10172 10152 10112  9902  9453
    5553   1 2021-05-01 11942 11517 10736 10372 10390 10085 10071 10037  9816  9328
    5554   1 2021-05-01 11680 11203 10399 10012 10015  9711  9684  9690  9433  9158
    5555   1 2021-05-01 11442 11241 10436 10048 10039  9733  9705  9743  9447  8877
    5556   1 2021-05-01 11503 10961 10185  9801  9809  9517  9504  9460  9269  8871
    5557   1 2021-05-01 11428 10897 10124  9711  9736  9451  9426  9408  9175  8859
    5558   1 2021-05-01 11317 10865 10096  9701  9730  9443  9424  9377  9172  8814
    5559   1 2021-05-01 11380 10910 10141  9758  9793  9517  9520  9446  9285  8883
    5560   1 2021-05-01 11399 10910 10141  9758  9793  9517  9520  9446  9285  8942
    5561   1 2021-05-01 11279 10989 10217  9829  9859  9576  9569  9582  9336  8785
    5562   1 2021-05-01 11278 10841 10085  9705  9729  9454  9442  9453  9235  8785
    5563   1 2021-05-01 11371 10955 10219  9875  9908  9645  9645  9632  9430  8884
    5564   1 2021-05-01 11363 10883 10168  9831  9864  9612  9618  9600  9399  8971
    5565   1 2021-05-01 11196 10791 10076  9747  9798  9559  9577  9509  9376  8806
    5566   1 2021-05-01 11160 10818 10109  9796  9824  9580  9587  9584  9383  8779
    5567   1 2021-05-01 11217 10737 10006  9674  9704  9454  9458  9471  9242  8843
    5568   1 2021-05-01 11188 11067 10325  9993 10039  9771  9763  9774  9542  8872
    5569   1 2021-05-01 11404 11035 10295  9966 10009  9741  9741  9767  9523  9145
    5570   1 2021-05-01 11549 11069 10366 10058 10107  9854  9860  9840  9655  9279
    5571   1 2021-05-01 11456 10952 10236  9913  9957  9695  9702  9709  9488  9178
    5572   1 2021-05-01 11573 11271 10536 10221 10274 10004 10007  9991  9797  9312
    5573   1 2021-05-01 11720 11295 10552 10233 10284 10014 10014 10000  9798  9516
    5574   1 2021-05-01 11885 11427 10687 10367 10423 10154 10151 10132  9920  9752
    5575   1 2021-05-01 11881 11456 10720 10409 10462 10196 10188 10179  9967  9758
    5576   1 2021-05-01 11941 11486 10766 10459 10531 10277 10260 10215 10043  9851
    5577   1 2021-05-01 11871 11498 10799 10501 10576 10330 10325 10259 10112  9783
    5578   1 2021-05-01 11589 11113 10407 10108 10179  9932  9939  9839  9739  9482
    5579   1 2021-05-01 11438 10637  9993  9734  9792  9585  9606  9510  9429  9294
    5580   1 2021-05-01 11035 10593  9944  9692  9743  9543  9575  9503  9401  8820
    5581   1 2021-05-01 11200 10727 10056  9775  9837  9618  9636  9540  9462  8968
    5582   1 2021-05-01 11242 10829 10160  9878  9938  9707  9727  9675  9552  9029
    5583   1 2021-05-01 11308 10870 10185  9904  9959  9736  9748  9697  9569  9139
    5584   1 2021-05-01 11295 10751 10090  9817  9875  9652  9682  9604  9497  9124
    5585   1 2021-05-01 11186 10853 10175  9895  9942  9711  9746  9697  9556  8943
    5586   1 2021-05-01 11476 11158 10468 10189 10242 10001 10030  9979  9836  9249
    5587   1 2021-05-01 11666 11262 10565 10273 10336 10092 10110 10052  9907  9490
    5588   1 2021-05-01 11760 11262 10565 10273 10336 10092 10110 10052  9907  9533
    5589   1 2021-05-01 11785 11327 10642 10349 10414 10173 10202 10110 10000  9492
    5590   1 2021-05-01 11826 11356 10655 10365 10419 10183 10210 10136 10019  9454
    5591   1 2021-05-01 11726 11298 10618 10338 10385 10159 10193 10102 10007  9301
    5592   1 2021-05-01 11730 11355 10656 10357 10408 10176 10205 10148 10011  9286
    5593   1 2021-05-01 11738 11331 10625 10327 10380 10149 10177 10098  9981  9264
    5594   1 2021-05-01 11755 11308 10603 10307 10358 10125 10160 10065  9972  9299
    5595   1 2021-05-01 11705 11212 10535 10257 10302 10088 10132 10045  9960  9231
    5596   1 2021-05-01 11668 11167 10489 10210 10259 10039 10087  9996  9900  9127
    5597   1 2021-05-01 11471 10898 10245  9973 10009  9820  9879  9793  9701  8906
    5598   1 2021-05-01 11439 11003 10331 10040 10089  9875  9914  9816  9737  8874
    5599   1 2021-05-01 11518 11127 10434 10144 10195  9972 10018  9921  9829  9009
    5600   1 2021-05-01 11622 11225 10525 10235 10288 10053 10079 10025  9897  9146
    5601   1 2021-05-01 11692 11225 10525 10235 10288 10053 10079 10025  9897  9234
    5602   1 2021-05-01 11360 11027 10341 10053 10107  9895  9936  9812  9744  8827
    5603   1 2021-05-01 10961 10662 10031  9763  9810  9623  9693  9549  9516  8318
    5604   1 2021-05-01 10970 10711 10082  9811  9840  9652  9720  9699  9552  8336
    5605   1 2021-05-01 11214 10876 10238  9972 10022  9832  9887  9791  9711  8808
    5606   1 2021-05-01 11473 10933 10294 10011 10069  9864  9897  9828  9712  9147
    5607   1 2021-05-01 11438 11001 10358 10089 10142  9945  9981  9934  9810  9101
    5608   1 2021-05-01 11468 11223 10539 10242 10292 10076 10094 10136  9904  9150
    5609   1 2021-05-01  9728  9437  8869  8603  8663  8557  8633  8602  8493  7698
    5610   1 2021-05-01 11663 11264 10442 10046 10046  9751  9733  9747  9498  9080
    5611   1 2021-05-01 11642 11388 10563 10156 10161  9847  9804  9869  9543  9160
    5612   1 2021-05-01 11829 11456 10656 10278 10308 10009  9985  9949  9719  9326
    5613   1 2021-05-01 11738 11251 10453 10079 10101  9817  9801  9728  9558  9279
    5614   1 2021-05-01 11655 11104 10287  9912  9920  9634  9616  9573  9371  9210
    5615   1 2021-05-01 11529 11104 10287  9912  9920  9634  9616  9573  9371  9083
    5616   1 2021-05-01 11394 10982 10170  9775  9784  9483  9465  9454  9221  8910
    5617   1 2021-05-01 11352 10831 10044  9651  9647  9362  9344  9315  9106  8770
    5618   1 2021-05-01 11371 10792 10012  9613  9638  9357  9350  9275  9108  8897
    5619   1 2021-05-01 11344 10888 10101  9690  9707  9418  9408  9376  9157  8907
    5620   1 2021-05-01 11406 10891 10133  9749  9785  9511  9497  9421  9249  8948
    5621   1 2021-05-01 11358 10793 10034  9645  9680  9401  9398  9330  9169  8922
    5622   1 2021-05-01 11215 10729  9973  9593  9626  9356  9365  9298  9135  8754
    5623   1 2021-05-01 11149 10692  9938  9571  9606  9344  9348  9264  9129  8692
    5624   1 2021-05-01 11157 10667  9926  9550  9584  9322  9313  9301  9093  8713
    5625   1 2021-05-01 11125 10676  9974  9642  9684  9430  9438  9413  9217  8741
    5626   1 2021-05-01 10990 10530  9851  9530  9575  9329  9351  9295  9136  8631
    5627   1 2021-05-01 10916 10433  9726  9409  9440  9208  9229  9176  9034  8534
    5628   1 2021-05-01 10797 10410  9688  9351  9381  9146  9155  9126  8948  8434
    5629   1 2021-05-01 10949 10736 10005  9661  9687  9440  9436  9462  9210  8631
    5630   1 2021-05-01 11131 10720 10007  9684  9712  9465  9473  9482  9264  8836
    5631   1 2021-05-01 11279 10923 10203  9883  9919  9665  9663  9672  9441  8995
    5632   1 2021-05-01 11307 10880 10151  9814  9851  9587  9577  9587  9347  9056
    5633   1 2021-05-01 11443 11101 10378 10058 10103  9839  9836  9839  9615  9201
    5634   1 2021-05-01 11632 11212 10489 10177 10237  9972  9975  9939  9758  9405
    5635   1 2021-05-01 11724 11327 10579 10259 10310 10038 10030 10034  9810  9557
    5636   1 2021-05-01 11673 11217 10483 10171 10217  9960  9959  9967  9756  9501
    5637   1 2021-05-01 11845 11502 10777 10471 10531 10254 10238 10282 10021  9700
    5638   1 2021-05-01 11941 11306 10590 10292 10366 10117 10118 10032  9918  9832
    5639   1 2021-05-01 11693 11197 10474 10176 10244  9990  9982  9922  9780  9572
    5640   1 2021-05-01 11602 10754 10111  9853  9933  9722  9742  9597  9570  9437
    5641   1 2021-05-01 11126 10461  9859  9632  9693  9501  9557  9424  9392  8914
    5642   1 2021-05-01 10944 10666 10011  9750  9785  9579  9614  9582  9430  8700
    5643   1 2021-05-01 10830 10602  9934  9655  9689  9475  9514  9516  9326  8532
    5644   1 2021-05-01 11169 10826 10153  9874  9927  9699  9714  9680  9535  8945
    5645   1 2021-05-01 11171 10665 10007  9733  9786  9572  9609  9557  9434  8917
    5646   1 2021-05-01 11102 10790 10119  9839  9892  9662  9696  9637  9501  8783
    5647   1 2021-05-01 11353 10923 10242  9954  9992  9758  9785  9800  9595  9050
    5648   1 2021-05-01 11599 11195 10501 10216 10269 10027 10050 10013  9849  9357
    5649   1 2021-05-01 11676 11167 10478 10190 10247 10008 10026  9979  9834  9393
    5650   1 2021-05-01 11626 11220 10543 10270 10318 10092 10129 10054  9934  9274
    5651   1 2021-05-01 11696 11222 10543 10260 10305 10080 10104 10046  9917  9324
    5652   1 2021-05-01 11587 11131 10467 10189 10226 10010 10039  9970  9853  9113
    5653   1 2021-05-01 11554 11242 10557 10271 10309 10089 10127 10073  9940  9031
    5654   1 2021-05-01 11633 11183 10503 10222 10266 10040 10085 10015  9897  9126
    5655   1 2021-05-01 11676 11115 10430 10144 10192  9964 10006  9937  9825  9201
    5656   1 2021-05-01 11557 11115 10430 10144 10192  9964 10006  9937  9825  9027
    5657   1 2021-05-01 11459 11060 10398 10118 10161  9949 10001  9907  9827  8883
    5658   1 2021-05-01 11276 10849 10208  9945  9987  9798  9853  9730  9689  8681
    5659   1 2021-05-01 11228 10907 10234  9950  9986  9779  9824  9780  9652  8603
    5660   1 2021-05-01 11480 11055 10359 10072 10105  9895  9938  9905  9755  8902
    5661   1 2021-05-01 11602 11181 10485 10202 10250 10029 10075  9998  9895  9113
    5662   1 2021-05-01 11584 11100 10407 10119 10168  9952  9993  9928  9817  9123
    5663   1 2021-05-01 11497 10807 10172  9916  9969  9777  9846  9686  9681  8991
    5664   1 2021-05-01 10807 10549  9929  9653  9681  9507  9570  9505  9404  8190
    5665   1 2021-05-01 11091 10700 10056  9772  9818  9624  9670  9593  9492  8539
    5666   1 2021-05-01 11601 11199 10539 10256 10340 10126 10153 10101  9969  9509
    5667   1 2021-05-01 11488 11037 10399 10136 10208 10013 10045  9996  9867  9399
    5668   1 2021-05-01 11381 10948 10308 10029 10098  9913  9949  9919  9768  9324
    5669   1 2021-05-01 11180 10749 10128  9868  9944  9769  9812  9745  9650  9150
    5670   1 2021-05-01 11030 10540  9908  9632  9706  9533  9568  9503  9399  8981
    5671   1 2021-05-01 10940 10240  9649  9398  9482  9339  9390  9279  9230  8904
    5672   1 2021-05-01 10207  9571  9008  8756  8845  8723  8796  8662  8659  8160
    5673   1 2021-05-01  9992  9600  9060  8827  8912  8793  8869  8720  8739  7974
    5674   1 2021-05-01  9800  9455  8876  8604  8675  8553  8630  8568  8493  7767
    5675   1 2021-05-01 11550 11085 10275  9902  9918  9646  9659  9566  9438  9017
    5676   1 2021-05-01 11652 11264 10442 10046 10046  9751  9733  9747  9498  9090
    5677   1 2021-05-01 11672 11240 10402  9997 10016  9720  9715  9707  9491  9241
    5678   1 2021-05-01 11695 11324 10522 10136 10145  9836  9798  9809  9540  9278
    5679   1 2021-05-01 11678 11212 10404 10039 10053  9761  9740  9683  9504  9268
    5680   1 2021-05-01 11709 11179 10393 10021 10040  9755  9743  9660  9497  9273
    5681   1 2021-05-01 11581 11095 10312  9943  9970  9679  9674  9576  9422  9175
    5682   1 2021-05-01 11378 10814 10036  9659  9689  9403  9393  9285  9154  8967
    5683   1 2021-05-01 11400 10889 10119  9750  9774  9496  9483  9384  9246  8975
    5684   1 2021-05-01 11402 10897 10100  9701  9729  9451  9440  9380  9206  8916
    5685   1 2021-05-01 11366 10872 10087  9679  9706  9435  9425  9363  9187  8957
    5686   1 2021-05-01 11337 10794 10022  9625  9658  9380  9368  9313  9130  8974
    5687   1 2021-05-01 11256 10778 10009  9626  9670  9397  9403  9317  9184  8890
    5688   1 2021-05-01 11238 10773 10006  9610  9636  9350  9350  9320  9114  8825
    5689   1 2021-05-01 11200 10692  9938  9571  9606  9344  9348  9264  9129  8822
    5690   1 2021-05-01 11252 10674  9914  9531  9569  9308  9300  9235  9086  8899
    5691   1 2021-05-01 11173 10616  9899  9548  9596  9347  9333  9278  9124  8844
    5692   1 2021-05-01 10899 10399  9704  9375  9417  9181  9189  9133  8984  8559
    5693   1 2021-05-01 10603 10267  9573  9253  9287  9059  9088  9023  8893  8233
    5694   1 2021-05-01 10572 10199  9483  9152  9178  8942  8965  8940  8769  8202
    5695   1 2021-05-01 10889 10485  9779  9460  9498  9257  9270  9237  9068  8538
    5696   1 2021-05-01 10980 10565  9840  9517  9557  9312  9316  9263  9114  8671
    5697   1 2021-05-01 11063 10747  9990  9632  9667  9409  9398  9393  9182  8806
    5698   1 2021-05-01 11395 11043 10264  9902  9942  9672  9648  9680  9427  9197
    5699   1 2021-05-01 11616 11274 10557 10236 10288 10018 10005 10006  9781  9420
    5700   1 2021-05-01 11673 11358 10628 10307 10357 10083 10075 10073  9850  9464
    5701   1 2021-05-01 11748 11165 10470 10192 10258 10023 10024  9936  9839  9583
    5702   1 2021-05-01 11315 11217 10483 10171 10217  9960  9959  9967  9756  9109
    5703   1 2021-05-01 11539 11141 10400 10078 10113  9852  9848  9917  9652  9306
    5704   1 2021-05-01 11566 11350 10621 10316 10371 10106 10085 10104  9872  9369
    5705   1 2021-05-01 11612 11199 10453 10142 10192  9922  9904  9923  9694  9501
    5706   1 2021-05-01 11490 11095 10404 10136 10206  9978  9986  9891  9800  9294
    5707   1 2021-05-01 11095 10733 10107  9869  9935  9732  9768  9638  9596  8864
    5708   1 2021-05-01 10681 10209  9599  9364  9410  9230  9302  9204  9145  8390
    5709   1 2021-05-01 10480 10166  9491  9203  9232  9030  9073  9058  8894  8137
    5710   1 2021-05-01 10937 10470  9826  9548  9592  9377  9407  9365  9228  8630
    5711   1 2021-05-01 10866 10388  9769  9514  9565  9365  9418  9308  9245  8572
    5712   1 2021-05-01 10912 10597  9950  9676  9726  9517  9557  9494  9373  8582
    5713   1 2021-05-01 11088 10722 10056  9768  9817  9593  9631  9570  9440  8763
    5714   1 2021-05-01 11365 11053 10353 10068 10106  9867  9890  9883  9700  9054
    5715   1 2021-05-01 11533 11149 10483 10205 10266 10033 10073  9996  9895  9169
    5716   1 2021-05-01 11580 11149 10483 10205 10266 10033 10073  9996  9895  9192
    5717   1 2021-05-01 11623 11159 10470 10195 10243 10019 10055  9956  9864  9222
    5718   1 2021-05-01 11533 11086 10405 10126 10166  9942  9990  9903  9805  9091
    5719   1 2021-05-01 11397 11084 10417 10139 10168  9951  9998  9944  9813  8881
    5720   1 2021-05-01 11518 11074 10395 10105 10136  9918  9959  9908  9776  9002
    5721   1 2021-05-01 11548 11083 10398 10120 10167  9944  9996  9892  9813  9043
    5722   1 2021-05-01 11519 10988 10319 10044 10076  9863  9918  9853  9734  8941
    5723   1 2021-05-01 11140 10828 10179  9912  9933  9750  9812  9718  9649  8459
    5724   1 2021-05-01 11197 10716 10050  9762  9783  9594  9644  9584  9471  8580
    5725   1 2021-05-01 11186 10807 10136  9849  9883  9684  9726  9666  9546  8562
    5726   1 2021-05-01 11250 10877 10204  9922  9956  9749  9785  9753  9603  8670
    5727   1 2021-05-01 11484 11036 10367 10088 10137  9929  9988  9900  9815  8987
    5728   1 2021-05-01 11432 10839 10188  9925  9969  9772  9833  9740  9664  8906
    5729   1 2021-05-01 11085 10839 10188  9925  9969  9772  9833  9740  9664  8505
    5730   1 2021-05-01 10870 10433  9816  9554  9594  9424  9497  9384  9342  8256
    5731   1 2021-05-01 11425 10959 10324 10056 10129  9943  9982  9925  9798  9334
    5732   1 2021-05-01 11190 10807 10173  9903  9973  9795  9829  9814  9660  9115
    5733   1 2021-05-01 11118 10611  9991  9731  9800  9630  9673  9661  9508  9049
    5734   1 2021-05-01 10965 10773 10146  9884  9959  9773  9813  9790  9646  8894
    5735   1 2021-05-01 10996 10592  9960  9689  9762  9584  9627  9577  9458  8958
    5736   1 2021-05-01 10859 10438  9821  9552  9627  9464  9502  9434  9340  8829
    5737   1 2021-05-01 10449 10113  9540  9300  9386  9245  9309  9202  9159  8404
    5738   1 2021-05-01  9931  9123  8632  8428  8515  8441  8552  8356  8447  7883
    5739   1 2021-05-01  9113  8884  8379  8161  8237  8162  8269  8095  8157  7026
    5740   1 2021-05-01 10165  9822  9221  8958  9023  8885  8940  8903  8794  8129
    5741   1 2021-05-01 10392  9958  9371  9120  9196  9056  9111  9015  8958  8401
    5742   1 2021-05-01 10365 10079  9485  9237  9313  9170  9220  9145  9074  8362
    5743   1 2021-05-01 10425 10063  9469  9215  9292  9152  9200  9144  9053  8402
    5744   1 2021-05-01 10447  9803  9241  9008  9093  8970  9042  8905  8920  8456
    5745   1 2021-05-01 10252  9767  9212  8981  9073  8960  9040  8870  8908  8265
    5746   1 2021-05-01 11864 11102 10357 10046 10085  9834  9872  9614  9666  9108
    5747   1 2021-05-01 11762 11123 10355 10020 10051  9776  9798  9615  9566  9118
    5748   1 2021-05-01 11767 11241 10491 10174 10218  9943  9962  9734  9723  9231
    5749   1 2021-05-01 11789 11150 10358 10005 10051  9779  9788  9623  9559  9372
    5750   1 2021-05-01 11774 11190 10390 10036 10086  9819  9828  9648  9588  9424
    5751   1 2021-05-01 11716 11243 10436 10059 10082  9793  9782  9714  9526  9350
    5752   1 2021-05-01 11665 11197 10394 10026 10063  9783  9777  9663  9530  9287
    5753   1 2021-05-01 11665 11024 10247  9878  9912  9629  9619  9501  9376  9326
    5754   1 2021-05-01 11555 11075 10293  9913  9934  9646  9627  9567  9398  9157
    5755   1 2021-05-01 11455 10910 10138  9774  9808  9527  9521  9408  9294  9044
    5756   1 2021-05-01 11402 10942 10180  9805  9849  9574  9568  9446  9347  8991
    5757   1 2021-05-01 11541 10873 10113  9730  9762  9499  9499  9395  9275  9104
    5758   1 2021-05-01 11444 10778 10009  9626  9670  9397  9403  9317  9184  9021
    5759   1 2021-05-01 11371 10867 10114  9761  9803  9544  9550  9450  9337  8903
    5760   1 2021-05-01 11361 10788 10030  9669  9705  9455  9458  9355  9241  8887
    5761   1 2021-05-01 11271 10777 10045  9707  9746  9498  9514  9416  9295  8857
    5762   1 2021-05-01 10981 10392  9705  9391  9436  9203  9221  9114  9019  8641
    5763   1 2021-05-01 10659 10116  9443  9125  9177  8951  8978  8846  8781  8319
    5764   1 2021-05-01 10510  9962  9294  8986  9031  8813  8846  8755  8672  8163
    5765   1 2021-05-01 10412 10028  9360  9057  9089  8874  8912  8841  8725  8034
    5766   1 2021-05-01 10805 10524  9787  9452  9484  9227  9233  9223  9025  8444
    5767   1 2021-05-01 11051 10524  9787  9452  9484  9227  9233  9223  9025  8735
    5768   1 2021-05-01 11292 10774 10035  9696  9747  9498  9504  9423  9292  9028
    5769   1 2021-05-01 11420 11047 10297  9960 10008  9750  9744  9705  9526  9237
    5770   1 2021-05-01 11708 11289 10525 10183 10228  9956  9943  9976  9731  9577
    5771   1 2021-05-01 11779 11358 10628 10307 10357 10083 10075 10073  9850  9634
    5772   1 2021-05-01 11743 11339 10627 10317 10371 10115 10109 10106  9897  9592
    5773   1 2021-05-01 11510 10800 10143  9886  9947  9740  9776  9668  9609  9335
    5774   1 2021-05-01 11138 10815 10109  9821  9862  9642  9663  9636  9481  8924
    5775   1 2021-05-01 11481 11142 10407 10106 10151  9904  9904  9876  9707  9240
    5776   1 2021-05-01 11477 10957 10267  9984 10033  9808  9827  9764  9637  9283
    5777   1 2021-05-01 11355 10723 10057  9787  9832  9622  9648  9614  9471  9149
    5778   1 2021-05-01 10803 10333  9711  9457  9502  9308  9345  9247  9178  8497
    5779   1 2021-05-01 10438  9858  9260  9023  9069  8903  8975  8820  8827  8110
    5780   1 2021-05-01 10268  9858  9260  9023  9069  8903  8975  8820  8827  7876
    5781   1 2021-05-01 10534 10372  9690  9388  9417  9186  9218  9221  9028  8139
    5782   1 2021-05-01 10912 10492  9832  9550  9588  9364  9400  9372  9217  8575
    5783   1 2021-05-01 11273 10681 10018  9747  9798  9585  9618  9527  9435  8969
    5784   1 2021-05-01 11226 10722 10056  9768  9817  9593  9631  9570  9440  8910
    5785   1 2021-05-01 11311 10878 10189  9887  9928  9692  9709  9692  9511  8967
    5786   1 2021-05-01 11356 11069 10391 10114 10159  9930  9966  9940  9783  8991
    5787   1 2021-05-01 11292 10885 10218  9936  9968  9754  9796  9774  9606  8829
    5788   1 2021-05-01 11419 10985 10316 10035 10077  9855  9906  9837  9720  8938
    5789   1 2021-05-01 11320 10863 10204  9918  9957  9748  9797  9718  9613  8807
    5790   1 2021-05-01 11350 10958 10284  9992 10021  9816  9861  9793  9677  8816
    5791   1 2021-05-01 11351 10785 10117  9832  9857  9660  9703  9668  9518  8784
    5792   1 2021-05-01 10771 10131  9495  9222  9221  9049  9138  9082  8969  8019
    5793   1 2021-05-01 10913 10602  9950  9679  9692  9502  9570  9523  9393  8175
    5794   1 2021-05-01 11001 10431  9771  9493  9500  9311  9380  9340  9206  8308
    5795   1 2021-05-01 10862 10669 10007  9725  9747  9566  9619  9560  9446  8189
    5796   1 2021-05-01 11149 10877 10204  9922  9956  9749  9785  9753  9603  8541
    5797   1 2021-05-01 11355 10887 10220  9935  9968  9765  9814  9788  9629  8822
    5798   1 2021-05-01 11149 10722 10078  9815  9853  9663  9732  9639  9563  8576
    5799   1 2021-05-01 10815 10463  9823  9543  9557  9383  9444  9405  9274  8213
    5800   1 2021-05-01 11137 10758 10090  9802  9841  9647  9700  9630  9524  8598
    5801   1 2021-05-01 11358 10880 10192  9900  9938  9738  9771  9769  9601  8923
    5802   1 2021-05-01 11513 11151 10467 10187 10248 10040 10060 10026  9876  9284
    5803   1 2021-05-01 11622 11151 10467 10187 10248 10040 10060 10026  9876  9495
    5804   1 2021-05-01 11558 11155 10492 10213 10290 10075 10100 10053  9917  9465
    5805   1 2021-05-01 11459 10994 10364 10100 10169  9981 10021  9956  9840  9381
    5806   1 2021-05-01 11133 10695 10072  9818  9897  9722  9763  9688  9602  9059
    5807   1 2021-05-01 11058 10602  9988  9739  9818  9643  9683  9618  9520  9002
    5808   1 2021-05-01 10743 10466  9845  9589  9656  9494  9547  9533  9387  8693
    5809   1 2021-05-01 10824 10406  9772  9495  9571  9395  9436  9441  9272  8776
    5810   1 2021-05-01 10871 10250  9660  9415  9500  9354  9403  9319  9253  8851
    5811   1 2021-05-01 10496  9994  9429  9196  9282  9144  9198  9077  9058  8453
    5812   1 2021-05-01 10024  9433  8907  8685  8766  8665  8747  8552  8621  7983
    5813   1 2021-05-01  9891  9823  9230  8973  9044  8906  8961  8846  8802  7757
    5814   1 2021-05-01 10475  9918  9317  9060  9131  8988  9043  8978  8895  8482
    5815   1 2021-05-01 10562 10087  9476  9219  9294  9141  9194  9135  9035  8602
    5816   1 2021-05-01 10550 10079  9485  9237  9313  9170  9220  9145  9074  8579
    5817   1 2021-05-01 10529 10090  9498  9246  9326  9184  9229  9172  9093  8539
    5818   1 2021-05-01 10456 10041  9454  9207  9283  9147  9202  9146  9052  8478
    5819   1 2021-05-01 10463 10054  9458  9202  9273  9139  9194  9141  9047  8462
    5820   1 2021-05-01 10408  9823  9259  9022  9111  8984  9069  8956  8938  8418
    5821   1 2021-05-01 12168 11324 10601 10296 10347 10088 10128  9841  9916  9327
    5822   1 2021-05-01 12091 11509 10737 10401 10441 10181 10200  9990  9972  9302
    5823   1 2021-05-01 12113 11295 10551 10218 10260  9989 10016  9793  9796  9480
    5824   1 2021-05-01 11994 11456 10707 10369 10407 10129 10145  9944  9917  9479
    5825   1 2021-05-01 12080 11319 10556 10222 10276 10004 10011  9801  9775  9648
    5826   1 2021-05-01 12176 11459 10711 10392 10459 10187 10186  9942  9943  9832
    5827   1 2021-05-01 11988 11372 10557 10192 10237  9973  9985  9826  9747  9687
    5828   1 2021-05-01 12007 11395 10614 10278 10330 10058 10054  9871  9806  9739
    5829   1 2021-05-01 11815 11198 10392 10026 10060  9779  9791  9677  9565  9499
    5830   1 2021-05-01 11687 11199 10406 10047 10081  9815  9840  9691  9612  9341
    5831   1 2021-05-01 11633 11124 10360  9995 10026  9738  9731  9642  9493  9227
    5832   1 2021-05-01 11694 11054 10302  9935  9975  9705  9710  9590  9486  9229
    5833   1 2021-05-01 11700 11140 10404 10061 10083  9822  9845  9719  9626  9163
    5834   1 2021-05-01 11483 10923 10193  9841  9869  9603  9614  9527  9404  8941
    5835   1 2021-05-01 11331 10842 10111  9760  9786  9530  9538  9457  9320  8781
    5836   1 2021-05-01 11325 10830 10097  9747  9783  9526  9537  9442  9313  8752
    5837   1 2021-05-01 11223 10747 10022  9676  9714  9460  9469  9391  9248  8764
    5838   1 2021-05-01 11030 10474  9773  9438  9489  9246  9268  9160  9056  8657
    5839   1 2021-05-01 10733 10247  9551  9220  9271  9030  9058  8938  8852  8407
    5840   1 2021-05-01 10721 10096  9430  9119  9166  8943  8970  8838  8782  8366
    5841   1 2021-05-01 10562 10159  9494  9197  9240  9023  9054  8909  8869  8201
    5842   1 2021-05-01 10586 10289  9568  9232  9256  9013  9029  9008  8826  8217
    5843   1 2021-05-01 11074 10588  9831  9472  9502  9244  9232  9236  9013  8793
    5844   1 2021-05-01 11376 11021 10275  9929  9968  9690  9685  9721  9445  9122
    5845   1 2021-05-01 11620 11249 10502 10177 10228  9967  9959  9945  9748  9412
    5846   1 2021-05-01 11668 11274 10541 10215 10263  9995  9990  9999  9780  9476
    5847   1 2021-05-01 11573 11218 10507 10198 10256 10003 10010  9997  9810  9396
    5848   1 2021-05-01 11491 11028 10326 10021 10076  9823  9812  9778  9606  9308
    5849   1 2021-05-01 11357 10771 10111  9829  9895  9682  9697  9587  9516  9191
    5850   1 2021-05-01 11296 10771 10111  9829  9895  9682  9697  9587  9516  9104
    5851   1 2021-05-01 11333 11014 10300 10001 10050  9814  9817  9806  9614  9100
    5852   1 2021-05-01 11161 10860 10186  9911  9959  9733  9744  9715  9554  8881
    5853   1 2021-05-01 10890 10593  9939  9678  9714  9514  9545  9479  9371  8550
    5854   1 2021-05-01 10892 10474  9810  9532  9579  9369  9393  9299  9213  8598
    5855   1 2021-05-01 10778 10228  9629  9383  9447  9262  9302  9123  9135  8471
    5856   1 2021-05-01 10463 10206  9610  9373  9433  9259  9312  9117  9149  8080
    5857   1 2021-05-01 10505 10183  9552  9290  9323  9127  9184  9118  9017  8129
    5858   1 2021-05-01 10782 10728 10046  9748  9789  9576  9592  9567  9414  8402
    5859   1 2021-05-01 11265 10935 10244  9944  9988  9755  9783  9751  9595  8915
    5860   1 2021-05-01 11444 10978 10286  9993 10048  9824  9855  9738  9664  9070
    5861   1 2021-05-01 11478 10933 10292 10034 10082  9881  9924  9818  9748  9092
    5862   1 2021-05-01 11251 10707 10053  9780  9824  9628  9669  9580  9488  8828
    5863   1 2021-05-01 11139 10609  9956  9679  9708  9522  9563  9499  9390  8672
    5864   1 2021-05-01 10971 10763 10102  9813  9844  9642  9688  9636  9504  8417
    5865   1 2021-05-01 11002 10662 10004  9712  9733  9541  9581  9545  9412  8420
    5866   1 2021-05-01 11079 10752 10092  9808  9826  9632  9683  9627  9502  8512
    5867   1 2021-05-01 10911 10505  9853  9569  9594  9399  9450  9379  9271  8346
    5868   1 2021-05-01 10663 10307  9697  9438  9468  9296  9363  9238  9203  7997
    5869   1 2021-05-01 10334 10014  9424  9173  9195  9028  9111  8943  8961  7559
    5870   1 2021-05-01 10425 10021  9394  9129  9131  8969  9061  8977  8901  7589
    5871   1 2021-05-01 10431 10318  9684  9413  9423  9240  9322  9271  9145  7630
    5872   1 2021-05-01 10640 10325  9692  9415  9438  9260  9332  9253  9162  7932
    5873   1 2021-05-01 10889 10563  9900  9619  9649  9446  9500  9472  9322  8292
    5874   1 2021-05-01 10903 10490  9860  9593  9630  9449  9503  9422  9338  8322
    5875   1 2021-05-01 10906 10328  9720  9474  9515  9351  9424  9296  9281  8334
    5876   1 2021-05-01 10718 10435  9803  9546  9593  9416  9475  9391  9319  8130
    5877   1 2021-05-01 11007 10595  9937  9656  9687  9507  9563  9530  9400  8541
    5878   1 2021-05-01 10996 10616  9964  9682  9723  9533  9575  9531  9416  8564
    5879   1 2021-05-01 11126 10840 10156  9866  9904  9704  9737  9749  9563  8765
    5880   1 2021-05-01 11453 10981 10322 10041 10121  9908  9943  9862  9773  9224
    5881   1 2021-05-01 11473 11052 10380 10095 10155  9952  9986  9942  9797  9309
    5882   1 2021-05-01 11561 11161 10484 10204 10275 10067 10090 10066  9898  9436
    5883   1 2021-05-01 11256 10984 10322 10049 10121  9923  9957  9939  9779  9159
    5884   1 2021-05-01 11217 10944 10292 10019 10083  9892  9922  9926  9747  9122
    5885   1 2021-05-01 11126 10611  9983  9715  9780  9606  9647  9645  9481  9038
    5886   1 2021-05-01 10911 10632  9991  9723  9793  9609  9644  9636  9467  8810
    5887   1 2021-05-01 10969 10314  9719  9468  9546  9379  9432  9370  9267  8907
    5888   1 2021-05-01 10424 10125  9511  9245  9317  9167  9224  9199  9065  8385
    5889   1 2021-05-01 10596 10125  9511  9245  9317  9167  9224  9199  9065  8542
    5890   1 2021-05-01 10522 10252  9652  9387  9462  9307  9352  9328  9194  8474
    5891   1 2021-05-01 10526 10118  9514  9255  9333  9180  9225  9165  9077  8486
    5892   1 2021-05-01 10543  9918  9359  9127  9217  9086  9143  8984  9004  8515
    5893   1 2021-05-01 10415 10085  9490  9236  9321  9176  9234  9098  9086  8340
    5894   1 2021-05-01 10476 10013  9411  9148  9214  9071  9118  9058  8961  8423
    5895   1 2021-05-01 10510 10145  9545  9282  9361  9207  9260  9204  9103  8520
    5896   1 2021-05-01 10639 10233  9626  9364  9449  9288  9331  9295  9178  8673
    5897   1 2021-05-01 10616 10188  9588  9338  9423  9270  9318  9240  9179  8645
    5898   1 2021-05-01 10687 10255  9642  9378  9455  9300  9340  9286  9192  8712
    5899   1 2021-05-01 10634 10063  9472  9225  9303  9161  9212  9137  9069  8657
    5900   1 2021-05-01 10442 10045  9444  9186  9260  9105  9158  9095  9006  8433
    5901   1 2021-05-01 10445 10024  9442  9202  9275  9134  9195  9128  9043  8407
    5902   1 2021-05-01 12334 11486 10745 10427 10480 10221 10255  9981 10042  9593
    5903   1 2021-05-01 12238 11544 10815 10509 10570 10324 10351 10030 10146  9430
    5904   1 2021-05-01 12448 11639 10866 10528 10579 10322 10354 10110 10147  9779
    5905   1 2021-05-01 12545 11829 11039 10674 10714 10437 10465 10290 10275  9961
    5906   1 2021-05-01 12389 11829 11080 10738 10789 10489 10475 10322 10235  9985
    5907   1 2021-05-01 12416 11921 11184 10867 10937 10639 10626 10422 10392 10045
    5908   1 2021-05-01 12537 11758 10956 10605 10656 10376 10380 10236 10149 10177
    5909   1 2021-05-01 12316 11753 10991 10651 10693 10392 10383 10256 10139  9985
    5910   1 2021-05-01 12157 11457 10704 10387 10448 10180 10197  9964  9965  9790
    5911   1 2021-05-01 11910 11199 10406 10047 10081  9815  9840  9691  9612  9580
    5912   1 2021-05-01 11965 11285 10513 10151 10185  9907  9904  9793  9661  9580
    5913   1 2021-05-01 11825 11152 10402 10041 10071  9792  9787  9678  9550  9340
    5914   1 2021-05-01 11754 11192 10462 10124 10154  9883  9905  9755  9668  9134
    5915   1 2021-05-01 11498 10889 10156  9806  9833  9574  9594  9461  9390  8910
    5916   1 2021-05-01 11343 10820 10075  9727  9749  9498  9505  9403  9296  8740
    5917   1 2021-05-01 11161 10710  9995  9658  9684  9433  9460  9359  9254  8536
    5918   1 2021-05-01 11047 10509  9796  9443  9476  9228  9250  9152  9040  8553
    5919   1 2021-05-01 10942 10361  9650  9305  9346  9113  9126  9021  8929  8480
    5920   1 2021-05-01 10835 10350  9640  9315  9344  9118  9136  9046  8943  8453
    5921   1 2021-05-01 10878 10381  9679  9365  9405  9171  9189  9091  8994  8487
    5922   1 2021-05-01 10769 10373  9649  9323  9373  9127  9135  9020  8928  8434
    5923   1 2021-05-01 10841 10451  9715  9369  9412  9166  9171  9093  8955  8520
    5924   1 2021-05-01 10969 10769 10008  9654  9684  9417  9415  9432  9188  8677
    5925   1 2021-05-01 11263 10769 10008  9654  9684  9417  9415  9432  9188  9016
    5926   1 2021-05-01 11343 11127 10386 10046 10086  9820  9811  9846  9579  9115
    5927   1 2021-05-01 11498 11114 10372 10036 10080  9815  9804  9827  9573  9290
    5928   1 2021-05-01 11556 11112 10405 10098 10149  9893  9902  9871  9697  9379
    5929   1 2021-05-01 11490 11060 10341 10032 10086  9829  9829  9776  9621  9321
    5930   1 2021-05-01 11530 10994 10289  9983 10040  9798  9787  9726  9584  9361
    5931   1 2021-05-01 11491 11024 10316 10003 10067  9820  9814  9748  9615  9314
    5932   1 2021-05-01 11375 10792 10122  9841  9916  9685  9702  9604  9518  9166
    5933   1 2021-05-01 11066 10589  9920  9643  9705  9484  9505  9394  9320  8762
    5934   1 2021-05-01 11030 10604  9917  9616  9665  9431  9456  9391  9256  8736
    5935   1 2021-05-01 11116 10702 10019  9724  9779  9547  9571  9490  9363  8861
    5936   1 2021-05-01 10999 10556  9898  9615  9669  9456  9486  9383  9295  8732
    5937   1 2021-05-01 11006 10281  9683  9434  9494  9314  9369  9193  9198  8739
    5938   1 2021-05-01 10979 10281  9683  9434  9494  9314  9369  9193  9198  8646
    5939   1 2021-05-01 10842 10485  9842  9571  9612  9419  9466  9408  9302  8458
    5940   1 2021-05-01 10799 10569  9916  9643  9676  9477  9532  9499  9366  8375
    5941   1 2021-05-01 11154 11011 10323 10037 10071  9846  9870  9870  9679  8779
    5942   1 2021-05-01 11228 10903 10248  9975 10016  9804  9837  9808  9653  8840
    5943   1 2021-05-01 10864 10579  9952  9701  9740  9554  9626  9517  9458  8355
    5944   1 2021-05-01 10804 10441  9814  9555  9585  9392  9466  9372  9305  8259
    5945   1 2021-05-01 10855 10397  9770  9502  9532  9355  9411  9309  9255  8300
    5946   1 2021-05-01 10829 10370  9725  9445  9478  9289  9347  9241  9175  8220
    5947   1 2021-05-01 10810 10471  9814  9529  9561  9362  9418  9319  9240  8166
    5948   1 2021-05-01 10946 10486  9821  9538  9564  9363  9421  9351  9248  8327
    5949   1 2021-05-01 10874 10360  9731  9470  9504  9321  9386  9217  9229  8259
    5950   1 2021-05-01 10615 10028  9431  9177  9205  9042  9124  8946  8971  7946
    5951   1 2021-05-01 10705 10115  9497  9234  9265  9098  9178  9007  9016  8075
    5952   1 2021-05-01 10531 10080  9450  9188  9196  9025  9120  9020  8947  7794
    5953   1 2021-05-01 10590 10195  9547  9261  9269  9097  9174  9121  9002  7852
    5954   1 2021-05-01 10641 10428  9783  9503  9537  9350  9403  9323  9235  7924
    5955   1 2021-05-01 10903 10485  9841  9564  9596  9417  9470  9405  9309  8321
    5956   1 2021-05-01 11060 10578  9948  9694  9738  9559  9622  9506  9463  8551
    5957   1 2021-05-01 10937 10538  9892  9632  9686  9502  9554  9445  9394  8443
    5958   1 2021-05-01 10955 10543  9897  9625  9670  9485  9534  9468  9374  8513
    5959   1 2021-05-01 11042 10593  9940  9659  9718  9524  9581  9499  9413  8674
    5960   1 2021-05-01 11135 10723 10046  9749  9805  9604  9635  9601  9456  8810
    5961   1 2021-05-01 11274 10865 10195  9913  9975  9771  9800  9756  9623  8983
    5962   1 2021-05-01 11374 10967 10313 10038 10110  9910  9952  9891  9785  9110
    5963   1 2021-05-01 11370 10956 10302 10020 10090  9892  9931  9867  9746  9213
    5964   1 2021-05-01 11325 10814 10165  9887  9951  9767  9805  9768  9626  9211
    5965   1 2021-05-01 11162 10773 10130  9870  9941  9758  9806  9764  9634  9049
    5966   1 2021-05-01 11108 10773 10130  9870  9941  9758  9806  9764  9634  9001
    5967   1 2021-05-01 11014 10693 10053  9790  9858  9686  9720  9705  9568  8916
    5968   1 2021-05-01 10794 10425  9815  9555  9627  9468  9508  9467  9367  8681
    5969   1 2021-05-01 10737 10422  9794  9510  9576  9409  9440  9434  9264  8643
    5970   1 2021-05-01 10803 10371  9740  9458  9527  9357  9387  9367  9218  8720
    5971   1 2021-05-01 10632 10121  9532  9284  9369  9220  9270  9181  9124  8588
    5972   1 2021-05-01 10475 10057  9459  9197  9275  9129  9186  9123  9050  8428
    5973   1 2021-05-01 10516 10142  9516  9241  9317  9154  9192  9156  9033  8482
    5974   1 2021-05-01 10706 10340  9691  9414  9491  9318  9347  9334  9190  8695
    5975   1 2021-05-01 10752 10238  9631  9372  9455  9299  9347  9290  9200  8755
    5976   1 2021-05-01 10490 10058  9461  9205  9283  9142  9192  9105  9041  8491
    5977   1 2021-05-01 10367  9969  9360  9088  9147  9001  9042  9067  8900  8315
    5978   1 2021-05-01 10369 10114  9508  9240  9306  9162  9206  9199  9049  8328
    5979   1 2021-05-01 10596 10222  9616  9359  9437  9281  9329  9287  9181  8594
    5980   1 2021-05-01 10655 10240  9629  9358  9438  9282  9324  9270  9179  8648
    5981   1 2021-05-01 10593 10168  9565  9304  9380  9228  9277  9211  9128  8591
    5982   1 2021-05-01 10487 10065  9459  9198  9265  9109  9166  9090  9009  8459
    5983   1 2021-05-01 12994 12344 11618 11292 11366 11053 11056 10812 10818 10219
    5984   1 2021-05-01 12959 12027 11338 11055 11152 10882 10916 10518 10692 10217
    5985   1 2021-05-01 12669 12056 11401 11136 11242 10949 10966 10552 10739  9958
    5986   1 2021-05-01 12728 11895 11118 10838 10951 10712 10772 10344 10575 10127
    5987   1 2021-05-01 12860 11895 11118 10838 10951 10712 10772 10344 10575 10144
    5988   1 2021-05-01 12618 12034 11245 10891 10927 10623 10616 10503 10391 10119
    5989   1 2021-05-01 12646 12084 11306 10957 10992 10685 10668 10562 10423 10255
    5990   1 2021-05-01 12611 11925 11139 10828 10893 10601 10590 10403 10354 10209
    5991   1 2021-05-01 12475 11753 10991 10651 10693 10392 10383 10256 10139 10087
    5992   1 2021-05-01 12395 11781 11028 10706 10748 10457 10446 10295 10205 10005
    5993   1 2021-05-01 12450 11602 10810 10447 10486 10202 10213 10096  9993 10032
    5994   1 2021-05-01 12269 11777 11005 10654 10694 10403 10415 10294 10187  9868
    5995   1 2021-05-01 12074 11438 10703 10360 10397 10121 10129  9989  9893  9481
    5996   1 2021-05-01 11818 11327 10610 10283 10318 10046 10071  9914  9861  9181
    5997   1 2021-05-01 11748 10959 10235  9905  9957  9708  9727  9528  9530  9081
    5998   1 2021-05-01 11469 10844 10133  9807  9857  9611  9639  9445  9445  8896
    5999   1 2021-05-01 11201 10513  9793  9441  9473  9219  9242  9135  9024  8602
    6000   1 2021-05-01 10986 10493  9771  9416  9433  9176  9188  9130  8970  8477
    6001   1 2021-05-01 10990 10440  9730  9385  9427  9189  9202  9089  8998  8490
    6002   1 2021-05-01 10862 10389  9694  9377  9417  9195  9216  9093  9022  8429
    6003   1 2021-05-01 10898 10419  9707  9384  9416  9180  9197  9116  8991  8491
    6004   1 2021-05-01 11069 10373  9649  9323  9373  9127  9135  9020  8928  8710
    6005   1 2021-05-01 11085 10527  9793  9449  9499  9242  9244  9159  9030  8776
    6006   1 2021-05-01 11133 10641  9894  9548  9588  9328  9335  9276  9116  8835
    6007   1 2021-05-01 11325 10845 10093  9752  9804  9533  9541  9485  9320  9060
    6008   1 2021-05-01 11434 10945 10203  9867  9916  9653  9646  9626  9420  9199
    6009   1 2021-05-01 11414 10973 10223  9875  9927  9655  9647  9638  9423  9204
    6010   1 2021-05-01 11531 11098 10373 10054 10111  9850  9852  9818  9632  9310
    6011   1 2021-05-01 11579 11146 10414 10099 10159  9906  9916  9836  9702  9347
    6012   1 2021-05-01 11639 11152 10428 10109 10176  9920  9919  9854  9718  9423
    6013   1 2021-05-01 11576 11041 10341 10038 10103  9852  9861  9805  9657  9340
    6014   1 2021-05-01 11385 10922 10251  9969 10040  9798  9814  9721  9617  9126
    6015   1 2021-05-01 11229 10763 10087  9803  9863  9637  9651  9539  9455  8948
    6016   1 2021-05-01 11140 10692 10000  9703  9760  9529  9549  9458  9357  8848
    6017   1 2021-05-01 11234 10702 10019  9724  9779  9547  9571  9490  9363  8979
    6018   1 2021-05-01 11310 10768 10083  9791  9850  9622  9641  9544  9451  9039
    6019   1 2021-05-01 11270 10647  9998  9722  9774  9567  9603  9486  9422  8989
    6020   1 2021-05-01 11167 10794 10136  9862  9915  9698  9734  9635  9556  8850
    6021   1 2021-05-01 11191 10537  9937  9696  9751  9556  9619  9483  9450  8763
    6022   1 2021-05-01 10966 10584  9917  9639  9671  9467  9519  9448  9348  8538
    6023   1 2021-05-01 11193 10843 10162  9864  9905  9679  9700  9677  9502  8809
    6024   1 2021-05-01 11280 10809 10163  9892  9946  9734  9773  9676  9587  8909
    6025   1 2021-05-01 11000 10494  9880  9629  9668  9482  9558  9435  9406  8513
    6026   1 2021-05-01 11034 10738 10081  9812  9852  9647  9698  9612  9535  8480
    6027   1 2021-05-01 11072 10492  9861  9606  9659  9474  9538  9366  9375  8524
    6028   1 2021-05-01 10844 10492  9861  9606  9659  9474  9538  9366  9375  8227
    6029   1 2021-05-01  9999 10231  9574  9275  9286  9101  9166  9137  8992  7210
    6030   1 2021-05-01 10699 10486  9821  9538  9564  9363  9421  9351  9248  8000
    6031   1 2021-05-01 11016 10562  9901  9621  9647  9446  9506  9434  9338  8427
    6032   1 2021-05-01 11051 10414  9771  9505  9551  9372  9438  9251  9268  8467
    6033   1 2021-05-01 10995 10466  9827  9561  9608  9432  9496  9303  9335  8422
    6034   1 2021-05-01 10528  9975  9375  9124  9153  9003  9110  8918  8959  7851
    6035   1 2021-05-01 10300  9966  9365  9111  9142  8989  9090  8937  8930  7563
    6036   1 2021-05-01 10507 10247  9615  9360  9379  9214  9294  9209  9144  7831
    6037   1 2021-05-01 10663 10193  9564  9297  9317  9151  9223  9164  9070  8064
    6038   1 2021-05-01 10874 10527  9869  9588  9607  9421  9468  9472  9301  8324
    6039   1 2021-05-01 10843 10472  9814  9541  9571  9388  9440  9417  9278  8331
    6040   1 2021-05-01 11005 10569  9926  9656  9710  9522  9579  9504  9414  8580
    6041   1 2021-05-01 11033 10569  9926  9656  9710  9522  9579  9504  9414  8653
    6042   1 2021-05-01 11282 10731 10069  9782  9842  9648  9691  9610  9519  9005
    6043   1 2021-05-01 11296 10865 10195  9913  9975  9771  9800  9756  9623  9049
    6044   1 2021-05-01 11257 10867 10224  9961 10030  9841  9888  9834  9721  9049
    6045   1 2021-05-01 11255 10863 10215  9942 10010  9820  9865  9821  9694  9072
    6046   1 2021-05-01 10817 10628  9995  9725  9791  9622  9669  9621  9500  8646
    6047   1 2021-05-01 10772 10553  9929  9676  9738  9562  9623  9587  9462  8655
    6048   1 2021-05-01 10682 10267  9657  9406  9466  9317  9375  9349  9220  8572
    6049   1 2021-05-01 10715 10310  9704  9453  9529  9369  9420  9366  9269  8625
    6050   1 2021-05-01 10643 10196  9576  9313  9374  9222  9274  9237  9122  8572
    6051   1 2021-05-01 10698 10300  9655  9370  9431  9267  9306  9280  9145  8596
    6052   1 2021-05-01 10680 10092  9475  9216  9276  9129  9185  9175  9034  8624
    6053   1 2021-05-01 10432  9942  9353  9107  9175  9045  9099  9084  8967  8420
    6054   1 2021-05-01 10317  9865  9268  9014  9079  8940  9003  9002  8861  8308
    6055   1 2021-05-01 10656 10086  9460  9186  9257  9104  9147  9121  9007  8633
    6056   1 2021-05-01 10742 10340  9691  9414  9491  9318  9347  9334  9190  8786
    6057   1 2021-05-01 10507 10150  9541  9285  9362  9200  9248  9228  9112  8567
    6058   1 2021-05-01 10330 10016  9422  9174  9248  9112  9161  9127  9020  8363
    6059   1 2021-05-01 10250  9963  9345  9073  9123  8982  9019  9037  8862  8191
    6060   1 2021-05-01 10494 10051  9439  9178  9239  9077  9131  9103  8970  8436
    6061   1 2021-05-01 10490 10085  9473  9205  9275  9119  9177  9105  9011  8430
    6062   1 2021-05-01 10592 10179  9556  9288  9358  9187  9240  9164  9084  8543
    6063   1 2021-05-01 13249 12837 12043 11691 11729 11399 11404 11276 11140 10324
    6064   1 2021-05-01 13496 12678 11951 11663 11755 11446 11459 11143 11220 10661
    6065   1 2021-05-01 13501 12701 11942 11629 11705 11398 11426 11143 11203 10702
    6066   1 2021-05-01 13207 12126 11468 11244 11365 11084 11095 10624 10865 10595
    6067   1 2021-05-01 13071 12113 11407 11164 11282 11024 11069 10597 10856 10340
    6068   1 2021-05-01 12585 12019 11216 10855 10891 10584 10583 10466 10343 10012
    6069   1 2021-05-01 12665 12050 11279 10951 11001 10710 10705 10527 10452 10144
    6070   1 2021-05-01 12592 11953 11178 10822 10856 10563 10561 10446 10325 10155
    6071   1 2021-05-01 12617 12087 11334 10997 11022 10715 10716 10613 10455 10060
    6072   1 2021-05-01 12665 12078 11305 10968 11000 10710 10730 10619 10484 10053
    6073   1 2021-05-01 12540 12078 11305 10968 11000 10710 10730 10619 10484  9912
    6074   1 2021-05-01 12202 11689 10937 10592 10628 10345 10358 10238 10143  9586
    6075   1 2021-05-01 11973 11421 10695 10350 10377 10095 10112  9991  9892  9302
    6076   1 2021-05-01 11830 11237 10526 10203 10239  9978 10007  9837  9797  9181
    6077   1 2021-05-01 11599 11115 10408 10085 10124  9867  9900  9717  9680  9052
    6078   1 2021-05-01 11494 10670  9947  9614  9660  9423  9449  9253  9245  8988
    6079   1 2021-05-01 11062 10503  9777  9430  9457  9221  9247  9094  9045  8556
    6080   1 2021-05-01 11008 10506  9815  9490  9524  9280  9297  9198  9085  8477
    6081   1 2021-05-01 10834 10379  9667  9340  9369  9142  9167  9066  8964  8338
    6082   1 2021-05-01 10952 10559  9818  9467  9507  9254  9252  9179  9035  8493
    6083   1 2021-05-01 11134 10680  9937  9599  9648  9392  9398  9297  9184  8733
    6084   1 2021-05-01 11429 10975 10238  9901  9945  9680  9677  9646  9459  9084
    6085   1 2021-05-01 11611 11014 10287  9974 10033  9787  9797  9674  9599  9269
    6086   1 2021-05-01 11571 11079 10339 10013 10065  9807  9809  9733  9602  9306
    6087   1 2021-05-01 11537 11010 10270  9940  9992  9727  9718  9677  9502  9297
    6088   1 2021-05-01 11506 11055 10323 10005 10067  9817  9819  9722  9612  9270
    6089   1 2021-05-01 11610 11057 10314  9965 10021  9739  9734  9736  9508  9331
    6090   1 2021-05-01 11674 11203 10483 10167 10228  9972  9968  9915  9757  9388
    6091   1 2021-05-01 11714 11249 10527 10224 10289 10045 10050  9973  9854  9439
    6092   1 2021-05-01 11594 11064 10349 10041 10102  9858  9872  9807  9677  9359
    6093   1 2021-05-01 11493 10950 10265  9971 10026  9788  9796  9713  9592  9220
    6094   1 2021-05-01 11450 10870 10192  9902  9960  9725  9745  9643  9553  9174
    6095   1 2021-05-01 11325 10883 10196  9902  9963  9726  9750  9651  9555  9015
    6096   1 2021-05-01 11335 10929 10228  9919  9975  9743  9745  9685  9548  9023
    6097   1 2021-05-01 11434 11014 10321 10027 10074  9839  9851  9807  9668  9111
    6098   1 2021-05-01 11424 10942 10266  9988 10030  9810  9835  9774  9661  9084
    6099   1 2021-05-01 11350 10651 10020  9764  9808  9604  9650  9572  9486  8997
    6100   1 2021-05-01 11070 10766 10122  9862  9908  9694  9743  9657  9563  8612
    6101   1 2021-05-01 10936 10499  9851  9582  9600  9403  9463  9411  9286  8443
    6102   1 2021-05-01 11325 10781 10085  9772  9810  9584  9613  9573  9426  8936
    6103   1 2021-05-01 11428 10951 10270  9975 10018  9792  9816  9754  9626  9082
    6104   1 2021-05-01 11225 10624 10007  9754  9808  9622  9676  9548  9514  8815
    6105   1 2021-05-01 11144 10668 10018  9756  9799  9604  9661  9583  9496  8628
    6106   1 2021-05-01 11120 10685 10045  9790  9844  9643  9702  9576  9530  8602
    6107   1 2021-05-01 10894  9903  9338  9111  9144  8997  9098  8876  8964  8316
    6108   1 2021-05-01  9825  9312  8773  8549  8564  8451  8576  8363  8454  7132
    6109   1 2021-05-01 10048 10327  9654  9341  9336  9130  9180  9233  8987  7201
    6110   1 2021-05-01 10661 10401  9741  9449  9470  9273  9328  9302  9159  8006
    6111   1 2021-05-01 10902 10610  9956  9679  9724  9526  9589  9505  9421  8345
    6112   1 2021-05-01 10991 10441  9827  9574  9617  9435  9508  9412  9351  8417
    6113   1 2021-05-01 10843 10297  9680  9424  9461  9297  9373  9214  9213  8214
    6114   1 2021-05-01 10791 10297  9680  9424  9461  9297  9373  9214  9213  8196
    6115   1 2021-05-01 10599 10133  9535  9291  9324  9166  9253  9115  9096  7996
    6116   1 2021-05-01 10545 10134  9537  9288  9324  9170  9248  9097  9093  7972
    6117   1 2021-05-01 10494 10064  9443  9173  9197  9041  9108  9067  8950  7901
    6118   1 2021-05-01 10583 10132  9504  9238  9271  9104  9169  9112  9006  8076
    6119   1 2021-05-01 10678 10364  9731  9459  9505  9329  9387  9358  9222  8270
    6120   1 2021-05-01 10943 10704 10038  9761  9807  9621  9670  9644  9508  8560
    6121   1 2021-05-01 11000 10723 10072  9803  9857  9675  9725  9709  9570  8680
    6122   1 2021-05-01 10938 10705 10068  9809  9875  9706  9755  9712  9599  8707
    6123   1 2021-05-01 10757 10340  9730  9473  9537  9380  9447  9419  9298  8547
    6124   1 2021-05-01 10554 10362  9743  9484  9544  9384  9449  9418  9281  8331
    6125   1 2021-05-01 10619 10086  9494  9249  9312  9161  9232  9174  9075  8442
    6126   1 2021-05-01 10561 10064  9473  9226  9285  9143  9200  9167  9054  8417
    6127   1 2021-05-01 10374  9860  9278  9035  9094  8960  9024  8989  8887  8292
    6128   1 2021-05-01 10197 10138  9524  9269  9326  9183  9239  9233  9081  8129
    6129   1 2021-05-01 10191  9826  9240  8994  9053  8928  8993  8981  8853  8122
    6130   1 2021-05-01 10445 10092  9455  9176  9232  9075  9131  9151  8974  8330
    6131   1 2021-05-01 10093  9689  9099  8844  8906  8771  8838  8841  8686  8078
    6132   1 2021-05-01  9573  9534  8947  8692  8749  8638  8707  8742  8576  7619
    6133   1 2021-05-01 10007  9865  9268  9014  9079  8940  9003  9002  8861  7999
    6134   1 2021-05-01 10375  9994  9375  9105  9167  9021  9067  9086  8928  8366
    6135   1 2021-05-01 10338 10033  9428  9172  9248  9084  9128  9101  8991  8359
    6136   1 2021-05-01 10334  9867  9280  9021  9100  8952  9002  8937  8856  8364
    6137   1 2021-05-01 10205  9642  9092  8855  8936  8806  8873  8774  8740  8221
    6138   1 2021-05-01 13015 12745 11935 11568 11576 11246 11239 11183 10966  9764
    6139   1 2021-05-01 13536 12845 12080 11725 11721 11380 11372 11301 11103 10362
    6140   1 2021-05-01 13437 12774 12008 11723 11801 11496 11507 11216 11271 10247
    6141   1 2021-05-01 13638 12742 11971 11652 11724 11431 11461 11200 11239 10760
    6142   1 2021-05-01 13448 12420 11730 11466 11555 11267 11286 10927 11054 10777
    6143   1 2021-05-01 13044 11941 11155 10811 10889 10614 10649 10381 10452 10435
    6144   1 2021-05-01 12793 12188 11418 11082 11117 10804 10777 10661 10504 10132
    6145   1 2021-05-01 12769 12125 11343 11006 11043 10756 10768 10617 10543 10183
    6146   1 2021-05-01 12656 12154 11350 10990 11000 10688 10697 10648 10441 10052
    6147   1 2021-05-01 12605 12075 11302 10957 10982 10678 10682 10609 10438  9958
    6148   1 2021-05-01 12308 11851 11089 10749 10772 10497 10530 10398 10314  9556
    6149   1 2021-05-01 12078 11520 10775 10418 10434 10150 10175 10060  9947  9291
    6150   1 2021-05-01 11895 11379 10620 10238 10253  9968  9973  9903  9735  9281
    6151   1 2021-05-01 11837 11286 10556 10220 10251  9986  9991  9869  9746  9239
    6152   1 2021-05-01 11733 11274 10551 10225 10267 10008 10009  9867  9780  9221
    6153   1 2021-05-01 11622 10830 10114  9789  9834  9605  9635  9429  9441  9162
    6154   1 2021-05-01 11180 10526  9815  9475  9492  9253  9280  9187  9077  8609
    6155   1 2021-05-01 10997 10484  9768  9416  9449  9204  9219  9123  9008  8404
    6156   1 2021-05-01 11028 10420  9727  9405  9440  9209  9227  9109  9028  8594
    6157   1 2021-05-01 11163 10581  9834  9478  9511  9253  9257  9192  9045  8701
    6158   1 2021-05-01 11336 10820 10078  9745  9784  9518  9523  9453  9286  8926
    6159   1 2021-05-01 11446 11010 10276  9953 10002  9737  9745  9674  9528  9106
    6160   1 2021-05-01 11590 11165 10444 10123 10163  9901  9905  9876  9691  9221
    6161   1 2021-05-01 11703 11206 10487 10173 10225  9975  9980  9917  9768  9385
    6162   1 2021-05-01 11648 11155 10430 10114 10161  9908  9914  9857  9697  9345
    6163   1 2021-05-01 11612 11162 10422 10089 10136  9875  9874  9836  9654  9312
    6164   1 2021-05-01 11674 11237 10524 10209 10263 10017 10012  9956  9806  9393
    6165   1 2021-05-01 11501 11019 10341 10040 10097  9859  9860  9789  9663  9213
    6166   1 2021-05-01 11448 10987 10285  9978 10025  9784  9799  9747  9594  9105
    6167   1 2021-05-01 11453 11081 10361 10056 10105  9856  9863  9811  9659  9119
    6168   1 2021-05-01 11562 11134 10419 10116 10167  9914  9916  9871  9723  9288
    6169   1 2021-05-01 11525 11043 10351 10054 10104  9867  9877  9805  9689  9219
    6170   1 2021-05-01 11446 10955 10273  9987 10050  9812  9837  9733  9653  9124
    6171   1 2021-05-01 11417 10945 10237  9923  9975  9746  9741  9682  9545  9105
    6172   1 2021-05-01 11422 10989 10285  9975 10017  9780  9780  9739  9587  9097
    6173   1 2021-05-01 11320 10938 10268  9990 10038  9810  9837  9777  9656  8946
    6174   1 2021-05-01 11276 10745 10114  9857  9915  9709  9738  9609  9571  8862
    6175   1 2021-05-01 10785 10350  9737  9488  9529  9351  9416  9303  9253  8333
    6176   1 2021-05-01 10879 10684 10021  9743  9780  9579  9622  9555  9443  8357
    6177   1 2021-05-01 11345 10969 10287 10001 10040  9819  9846  9791  9664  8907
    6178   1 2021-05-01 11427 10963 10290 10003 10043  9832  9862  9795  9684  9027
    6179   1 2021-05-01 11286 10704 10072  9818  9866  9675  9724  9600  9558  8860
    6180   1 2021-05-01 11028 10682 10015  9745  9791  9578  9633  9555  9455  8537
    6181   1 2021-05-01 11063 10766 10104  9831  9884  9671  9711  9598  9538  8571
    6182   1 2021-05-01 11197 10024  9508  9319  9393  9237  9332  8981  9192  8683
    6183   1 2021-05-01 10520 10024  9508  9319  9393  9237  9332  8981  9192  7878
    6184   1 2021-05-01 10673  9837  9229  8971  8986  8832  8926  8814  8780  7992
    6185   1 2021-05-01 10500 10234  9586  9302  9314  9133  9194  9128  9027  7767
    6186   1 2021-05-01 10997 10478  9826  9540  9582  9377  9430  9360  9248  8468
    6187   1 2021-05-01 10888 10495  9865  9595  9639  9452  9502  9397  9337  8356
    6188   1 2021-05-01 10859 10372  9749  9489  9523  9356  9430  9326  9268  8321
    6189   1 2021-05-01 10769 10313  9698  9440  9478  9312  9392  9269  9236  8194
    6190   1 2021-05-01 10747 10277  9648  9385  9427  9262  9327  9209  9162  8185
    6191   1 2021-05-01 10631 10229  9615  9356  9381  9230  9295  9217  9140  8051
    6192   1 2021-05-01 10556 10103  9492  9231  9270  9112  9191  9072  9040  7995
    6193   1 2021-05-01 10566 10192  9583  9315  9360  9192  9246  9170  9084  8048
    6194   1 2021-05-01 10553 10129  9520  9260  9310  9147  9218  9145  9055  8127
    6195   1 2021-05-01 10504 10441  9801  9538  9582  9402  9462  9452  9300  8100
    6196   1 2021-05-01 10727 10441  9801  9538  9582  9402  9462  9452  9300  8430
    6197   1 2021-05-01 10776 10520  9909  9653  9708  9550  9608  9568  9451  8517
    6198   1 2021-05-01 10644 10208  9612  9359  9410  9263  9325  9280  9170  8407
    6199   1 2021-05-01 10413 10147  9553  9305  9370  9220  9289  9238  9141  8221
    6200   1 2021-05-01 10444 10035  9440  9191  9253  9106  9170  9105  9017  8290
    6201   1 2021-05-01 10303  9897  9319  9074  9139  8999  9066  9007  8928  8175
    6202   1 2021-05-01 10188  9767  9182  8936  9010  8878  8936  8879  8802  8106
    6203   1 2021-05-01  9793  9468  8922  8698  8765  8663  8749  8666  8634  7742
    6204   1 2021-05-01  9567  9194  8661  8431  8500  8409  8504  8387  8380  7510
    6205   1 2021-05-01  9758  9431  8860  8607  8680  8556  8620  8559  8486  7671
    6206   1 2021-05-01  9959  9290  8771  8561  8639  8544  8639  8504  8519  7913
    6207   1 2021-05-01 10049  9559  9007  8784  8864  8760  8826  8703  8706  8063
    6208   1 2021-05-01 10094  9719  9129  8879  8941  8806  8859  8827  8717  8099
    6209   1 2021-05-01 10193  9828  9228  8969  9038  8901  8947  8910  8799  8216
    6210   1 2021-05-01 10269  9910  9305  9042  9112  8962  9006  8978  8867  8279
    6211   1 2021-05-01 10263  9902  9294  9033  9110  8966  9010  8957  8872  8297
    6212   1 2021-05-01 10189  9804  9217  8957  9033  8900  8951  8886  8808  8216
    6213   1 2021-05-01  9999  9448  8880  8634  8703  8590  8658  8560  8516  7911
    6214   1 2021-05-01  9992  9635  9043  8781  8840  8714  8768  8732  8625  7914
    6215   1 2021-05-01 13100 12569 11848 11518 11539 11211 11196 11026 10931 10013
    6216   1 2021-05-01 13371 12742 11971 11652 11724 11431 11461 11200 11239 10452
    6217   1 2021-05-01 13148 12473 11733 11409 11469 11164 11185 10957 10950 10525
    6218   1 2021-05-01 12880 12114 11328 11004 11082 10800 10825 10555 10603 10335
    6219   1 2021-05-01 12627 12226 11418 11023 11036 10715 10697 10674 10439 10119
    6220   1 2021-05-01 12733 12154 11411 11084 11112 10800 10796 10649 10540 10062
    6221   1 2021-05-01 12633 12036 11234 10863 10883 10588 10604 10521 10380  9957
    6222   1 2021-05-01 12512 12128 11353 10992 10994 10676 10669 10637 10416  9905
    6223   1 2021-05-01 12564 11907 11177 10864 10907 10624 10655 10447 10422  9822
    6224   1 2021-05-01 12536 11665 10941 10620 10668 10393 10418 10198 10205  9677
    6225   1 2021-05-01 12128 11555 10811 10473 10501 10235 10268 10090 10065  9425
    6226   1 2021-05-01 11988 11456 10685 10325 10366 10095 10111  9955  9891  9444
    6227   1 2021-05-01 11900 11386 10619 10250 10284  9996  9995  9913  9768  9380
    6228   1 2021-05-01 11905 11274 10551 10225 10267 10008 10009  9867  9780  9372
    6229   1 2021-05-01 11766 11147 10422 10108 10163  9919  9944  9747  9736  9230
    6230   1 2021-05-01 11303 10627  9957  9660  9696  9478  9530  9327  9347  8665
    6231   1 2021-05-01 11305 10644  9942  9609  9659  9421  9446  9268  9244  8676
    6232   1 2021-05-01 11200 10615  9861  9501  9538  9278  9282  9197  9068  8733
    6233   1 2021-05-01 11316 10813 10062  9703  9737  9471  9477  9419  9249  8878
    6234   1 2021-05-01 11458 11032 10301  9981 10019  9755  9766  9727  9558  9011
    6235   1 2021-05-01 11510 11091 10367 10047 10082  9818  9822  9793  9606  9109
    6236   1 2021-05-01 11557 11116 10399 10081 10116  9862  9876  9854  9668  9181
    6237   1 2021-05-01 11612 10960 10247  9933  9975  9725  9738  9697  9538  9219
    6238   1 2021-05-01 11384 11072 10336 10013 10051  9802  9801  9762  9594  9016
    6239   1 2021-05-01 11572 11072 10336 10013 10051  9802  9801  9762  9594  9253
    6240   1 2021-05-01 11608 11212 10494 10179 10234  9982  9977  9951  9761  9319
    6241   1 2021-05-01 11524 11019 10341 10040 10097  9859  9860  9789  9663  9235
    6242   1 2021-05-01 11481 10944 10231  9908  9961  9714  9718  9647  9515  9178
    6243   1 2021-05-01 11512 11107 10389 10082 10128  9878  9889  9839  9682  9173
    6244   1 2021-05-01 11592 11119 10401 10086 10141  9891  9900  9845  9703  9290
    6245   1 2021-05-01 11570 11095 10387 10091 10152  9902  9914  9835  9701  9262
    6246   1 2021-05-01 11570 11098 10386 10078 10131  9888  9901  9817  9699  9249
    6247   1 2021-05-01 11537 11045 10333 10025 10075  9831  9840  9767  9651  9219
    6248   1 2021-05-01 11465 11047 10342 10038 10094  9844  9866  9790  9674  9133
    6249   1 2021-05-01 11440 10920 10252  9966 10016  9793  9812  9727  9629  9078
    6250   1 2021-05-01 11325 10857 10196  9923  9979  9750  9790  9670  9614  8891
    6251   1 2021-05-01 11157 10756 10077  9791  9832  9617  9653  9573  9466  8705
    6252   1 2021-05-01 11211 10756 10077  9791  9832  9617  9653  9573  9466  8748
    6253   1 2021-05-01 11235 10878 10207  9920  9963  9750  9772  9734  9601  8767
    6254   1 2021-05-01 11339 10963 10290 10003 10043  9832  9862  9795  9684  8880
    6255   1 2021-05-01 11184 10786 10128  9844  9882  9674  9713  9647  9545  8705
    6256   1 2021-05-01 11182 10637  9980  9708  9756  9553  9603  9496  9427  8691
    6257   1 2021-05-01 11269 10860 10168  9875  9919  9699  9728  9681  9539  8834
    6258   1 2021-05-01 11380 10763 10119  9867  9922  9723  9776  9610  9614  8908
    6259   1 2021-05-01 11008 10634  9999  9741  9792  9595  9655  9523  9491  8427
    6260   1 2021-05-01 10530  9948  9356  9109  9126  8989  9085  8945  8945  7877
    6261   1 2021-05-01 10274 10066  9389  9087  9085  8912  8984  8964  8818  7492
    6262   1 2021-05-01 10986 10591  9927  9640  9675  9476  9514  9445  9340  8406
    6263   1 2021-05-01 10984 10490  9839  9558  9606  9417  9464  9338  9305  8503
    6264   1 2021-05-01 10882 10354  9737  9463  9507  9325  9387  9260  9214  8343
    6265   1 2021-05-01 10720 10285  9698  9457  9510  9348  9424  9226  9279  8174
    6266   1 2021-05-01 10584 10188  9571  9308  9337  9177  9255  9143  9102  8006
    6267   1 2021-05-01 10542 10229  9615  9356  9381  9230  9295  9217  9140  7993
    6268   1 2021-05-01 10446 10092  9487  9234  9267  9109  9191  9092  9042  7919
    6269   1 2021-05-01 10459 10123  9505  9230  9267  9106  9172  9106  9023  7986
    6270   1 2021-05-01 10440 10084  9477  9208  9249  9088  9159  9087  8994  8000
    6271   1 2021-05-01 10449 10135  9508  9240  9281  9114  9185  9147  9026  8058
    6272   1 2021-05-01 10435 10053  9445  9181  9231  9069  9137  9056  8975  8136
    6273   1 2021-05-01 10310  9997  9397  9140  9193  9045  9109  9053  8952  8067
    6274   1 2021-05-01 10369 10023  9432  9180  9237  9091  9161  9081  9006  8141
    6275   1 2021-05-01 10412  9993  9403  9165  9232  9080  9148  9075  9007  8232
    6276   1 2021-05-01 10408  9959  9362  9116  9182  9035  9094  9039  8950  8297
    6277   1 2021-05-01 10258  9733  9158  8917  8999  8872  8941  8855  8803  8205
    6278   1 2021-05-01 10082  9577  9011  8781  8852  8740  8812  8712  8673  8035
    6279   1 2021-05-01  9857  9325  8784  8550  8624  8526  8618  8500  8483  7834
    6280   1 2021-05-01  9918  9194  8661  8431  8500  8409  8504  8387  8380  7835
    6281   1 2021-05-01 10287  9655  9064  8813  8883  8757  8818  8726  8684  8290
    6282   1 2021-05-01 10377  9758  9183  8947  9019  8895  8949  8847  8819  8410
    6283   1 2021-05-01 10283  9815  9233  8992  9070  8935  8991  8923  8854  8313
    6284   1 2021-05-01 10321  9777  9179  8914  8992  8851  8894  8819  8743  8341
    6285   1 2021-05-01 10303  9848  9244  8987  9061  8920  8962  8911  8817  8332
    6286   1 2021-05-01 10250  9865  9265  9013  9090  8949  8997  8960  8863  8305
    6287   1 2021-05-01 10196  9643  9068  8835  8916  8786  8844  8800  8711  8237
    6288   1 2021-05-01 10050  9533  8948  8692  8764  8632  8693  8618  8556  7985
    6289   1 2021-05-01  9854  9334  8751  8491  8557  8434  8501  8440  8364  7733
    6290   1 2021-05-01  9815  9529  8922  8651  8709  8574  8635  8613  8481  7689
    6291   1 2021-05-01  9967  9782  9177  8903  8960  8813  8863  8865  8712  7839
    6292   1 2021-05-01 13303 12467 11696 11356 11421 11137 11170 10921 10959 10556
    6293   1 2021-05-01 13120 12304 11549 11239 11314 11018 11035 10783 10784 10508
    6294   1 2021-05-01 12966 12345 11581 11246 11301 10994 10993 10822 10742 10374
    6295   1 2021-05-01 12838 12208 11386 11040 11103 10814 10822 10646 10580 10248
    6296   1 2021-05-01 12835 12268 11466 11116 11158 10866 10876 10728 10625 10171
    6297   1 2021-05-01 12522 11892 11167 10840 10887 10601 10612 10407 10372  9783
    6298   1 2021-05-01 12284 11590 10843 10502 10557 10280 10291 10098 10071  9611
    6299   1 2021-05-01 12182 11472 10700 10351 10404 10139 10144  9976  9927  9634
    6300   1 2021-05-01 12312 11554 10826 10516 10575 10315 10334 10105 10116  9689
    6301   1 2021-05-01 12173 11373 10665 10368 10423 10177 10213  9982 10016  9495
    6302   1 2021-05-01 11981 11295 10608 10323 10381 10146 10191  9952  9990  9331
    6303   1 2021-05-01 11678 10919 10212  9893  9944  9706  9747  9553  9557  8899
    6304   1 2021-05-01 11450 10930 10182  9827  9874  9612  9632  9503  9419  8844
    6305   1 2021-05-01 11461 10925 10180  9839  9891  9632  9645  9510  9432  8979
    6306   1 2021-05-01 11403 10925 10180  9839  9891  9632  9645  9510  9432  8943
    6307   1 2021-05-01 11512 10985 10262  9932  9972  9707  9719  9667  9504  9055
    6308   1 2021-05-01 11509 11050 10327 10004 10047  9779  9789  9736  9573  9093
    6309   1 2021-05-01 11414 10990 10280  9973 10011  9756  9759  9717  9555  8999
    6310   1 2021-05-01 11379 10902 10194  9886  9930  9681  9692  9623  9482  8963
    6311   1 2021-05-01 11357 10956 10227  9906  9950  9690  9693  9668  9486  8967
    6312   1 2021-05-01 11497 11097 10370 10056 10114  9852  9846  9809  9637  9187
    6313   1 2021-05-01 11494 11062 10341 10025 10086  9825  9822  9768  9617  9200
    6314   1 2021-05-01 11589 11072 10356 10055 10108  9868  9874  9769  9668  9276
    6315   1 2021-05-01 11593 11083 10361 10049 10106  9851  9861  9776  9662  9281
    6316   1 2021-05-01 11548 11097 10383 10078 10122  9868  9885  9829  9678  9181
    6317   1 2021-05-01 11574 11179 10453 10153 10200  9960  9977  9888  9775  9242
    6318   1 2021-05-01 11659 11196 10478 10181 10230  9989  9992  9924  9791  9302
    6319   1 2021-05-01 11639 11196 10478 10181 10230  9989  9992  9924  9791  9305
    6320   1 2021-05-01 11608 11109 10388 10071 10126  9875  9887  9817  9695  9266
    6321   1 2021-05-01 11645 11155 10424 10121 10172  9916  9936  9873  9742  9254
    6322   1 2021-05-01 11581 11040 10355 10064 10114  9886  9911  9815  9731  9174
    6323   1 2021-05-01 11443 11009 10333 10061 10106  9882  9919  9817  9749  9026
    6324   1 2021-05-01 11262 10805 10128  9847  9895  9680  9714  9604  9541  8807
    6325   1 2021-05-01 11122 10751 10085  9805  9840  9629  9672  9607  9496  8627
    6326   1 2021-05-01 11029 10611  9962  9685  9730  9525  9571  9491  9393  8529
    6327   1 2021-05-01 10994 10595  9931  9646  9678  9478  9523  9466  9346  8483
    6328   1 2021-05-01 10988 10667  9995  9695  9730  9517  9553  9511  9375  8472
    6329   1 2021-05-01 11179 10833 10148  9858  9903  9684  9725  9659  9549  8658
    6330   1 2021-05-01 11273 10762 10089  9810  9851  9645  9689  9637  9511  8788
    6331   1 2021-05-01 11239 10699 10049  9784  9828  9631  9684  9598  9515  8764
    6332   1 2021-05-01 11102 10528  9904  9653  9695  9499  9572  9453  9402  8605
    6333   1 2021-05-01 10693 10112  9574  9370  9431  9279  9380  9137  9239  8056
    6334   1 2021-05-01 10356  9935  9370  9150  9196  9056  9164  8924  9029  7651
    6335   1 2021-05-01 10584 10482  9788  9471  9487  9279  9323  9327  9142  7890
    6336   1 2021-05-01 11095 10647  9985  9696  9740  9542  9587  9475  9421  8570
    6337   1 2021-05-01 11209 10603  9940  9647  9688  9498  9537  9407  9370  8732
    6338   1 2021-05-01 10980 10285  9698  9457  9510  9348  9424  9226  9279  8494
    6339   1 2021-05-01 11008 10350  9733  9481  9536  9358  9427  9247  9261  8528
    6340   1 2021-05-01 10777 10116  9523  9278  9325  9172  9249  9096  9103  8287
    6341   1 2021-05-01 10462  9946  9364  9115  9154  9013  9102  8972  8959  7937
    6342   1 2021-05-01 10343  9982  9372  9105  9143  8993  9069  8988  8908  7898
    6343   1 2021-05-01 10414 10073  9461  9192  9237  9089  9158  9070  9000  8022
    6344   1 2021-05-01 10479 10058  9443  9181  9225  9067  9145  9052  8983  8118
    6345   1 2021-05-01 10452 10019  9403  9147  9195  9045  9116  9040  8959  8137
    6346   1 2021-05-01 10440 10029  9420  9158  9209  9046  9110  9030  8940  8179
    6347   1 2021-05-01 10409 10021  9418  9166  9218  9070  9134  9062  8963  8240
    6348   1 2021-05-01 10365 10013  9423  9185  9242  9095  9171  9101  9023  8243
    6349   1 2021-05-01 10225  9843  9249  9005  9063  8925  8997  8949  8857  8148
    6350   1 2021-05-01  9913  9577  9011  8781  8852  8740  8812  8712  8673  7893
    6351   1 2021-05-01  9814  9735  9130  8871  8934  8804  8859  8840  8715  7814
    6352   1 2021-05-01 10170  9822  9226  8980  9045  8921  8977  8973  8845  8277
    6353   1 2021-05-01 10294  9878  9280  9035  9116  8966  9032  8966  8886  8373
    6354   1 2021-05-01 10285  9828  9233  8986  9066  8921  8985  8935  8838  8322
    6355   1 2021-05-01 10261  9886  9285  9030  9108  8971  9015  8976  8878  8293
    6356   1 2021-05-01 10315  9737  9151  8906  8977  8858  8896  8861  8773  8359
    6357   1 2021-05-01 10052  9576  9009  8766  8841  8726  8783  8732  8648  8108
    6358   1 2021-05-01  9887  9576  9009  8766  8841  8726  8783  8732  8648  7901
    6359   1 2021-05-01  9813  9481  8897  8643  8712  8586  8643  8593  8511  7696
    6360   1 2021-05-01  9608  9334  8751  8491  8557  8434  8501  8440  8364  7422
    6361   1 2021-05-01  9617  9263  8672  8398  8447  8330  8404  8375  8262  7438
    6362   1 2021-05-01  9813  9422  8839  8566  8625  8500  8562  8525  8415  7651
    6363   1 2021-05-01 13335 12439 11680 11362 11437 11153 11187 10907 10972 10624
    6364   1 2021-05-01 13038 12494 11703 11361 11413 11116 11144 10955 10917 10433
    6365   1 2021-05-01 12950 12373 11592 11251 11294 10989 10998 10846 10745 10169
    6366   1 2021-05-01 12804 12319 11516 11160 11178 10875 10905 10803 10664  9996
    6367   1 2021-05-01 12569 11792 11046 10703 10756 10478 10481 10288 10251  9910
    6368   1 2021-05-01 12452 11830 11062 10713 10766 10473 10480 10316 10260  9861
    6369   1 2021-05-01 12428 11671 10909 10559 10613 10339 10344 10173 10122  9822
    6370   1 2021-05-01 12541 11860 11111 10761 10801 10521 10524 10400 10286  9795
    6371   1 2021-05-01 12353 11681 10990 10713 10776 10513 10546 10306 10328  9610
    6372   1 2021-05-01 11933 11424 10739 10443 10496 10242 10281 10075 10072  9177
    6373   1 2021-05-01 11692 11090 10395 10102 10150  9913  9960  9750  9753  8914
    6374   1 2021-05-01 11594 11071 10362 10054 10098  9859  9900  9710  9698  8892
    6375   1 2021-05-01 11699 11131 10388 10070 10110  9858  9880  9763  9662  9110
    6376   1 2021-05-01 11608 11109 10382 10078 10116  9874  9897  9786  9699  9020
    6377   1 2021-05-01 11546 11036 10308  9984 10030  9767  9782  9702  9567  9050
    6378   1 2021-05-01 11530 11023 10297  9981 10028  9765  9774  9710  9567  9093
    6379   1 2021-05-01 11365 10893 10184  9872  9916  9667  9688  9585  9467  8941
    6380   1 2021-05-01 11394 10938 10219  9894  9947  9692  9689  9621  9476  9019
    6381   1 2021-05-01 11477 11053 10328 10003 10054  9804  9803  9723  9588  9110
    6382   1 2021-05-01 11510 11140 10413 10093 10151  9885  9884  9826  9675  9178
    6383   1 2021-05-01 11741 11140 10413 10093 10151  9885  9884  9826  9675  9421
    6384   1 2021-05-01 11725 11172 10453 10141 10199  9946  9948  9883  9730  9393
    6385   1 2021-05-01 11652 11188 10481 10174 10239  9992 10000  9897  9793  9317
    6386   1 2021-05-01 11567 11088 10366 10049 10099  9846  9846  9786  9645  9229
    6387   1 2021-05-01 11610 11159 10442 10129 10172  9922  9928  9889  9730  9223
    6388   1 2021-05-01 11668 11245 10533 10238 10286 10045 10050  9986  9849  9247
    6389   1 2021-05-01 11668 11234 10522 10231 10286 10034 10049  9975  9864  9289
    6390   1 2021-05-01 11641 11154 10414 10095 10144  9897  9910  9858  9718  9241
    6391   1 2021-05-01 11505 11083 10363 10050 10095  9857  9863  9838  9671  9039
    6392   1 2021-05-01 11536 11065 10390 10124 10163  9943  9977  9904  9792  9050
    6393   1 2021-05-01 11355 10879 10203  9930  9970  9758  9800  9704  9630  8851
    6394   1 2021-05-01 11179 10739 10057  9771  9810  9597  9642  9572  9473  8599
    6395   1 2021-05-01 11167 10666  9998  9727  9770  9566  9606  9500  9423  8641
    6396   1 2021-05-01 11164 10666  9998  9727  9770  9566  9606  9500  9423  8691
    6397   1 2021-05-01 11213 10560  9916  9645  9692  9490  9542  9416  9359  8771
    6398   1 2021-05-01 11155 10664  9989  9702  9745  9538  9585  9479  9396  8671
    6399   1 2021-05-01 11047 10679 10010  9734  9760  9559  9607  9563  9439  8545
    6400   1 2021-05-01 11087 10618  9942  9660  9691  9491  9541  9505  9365  8529
    6401   1 2021-05-01 10974 10630  9979  9717  9753  9560  9608  9536  9443  8400
    6402   1 2021-05-01 11010 10569  9923  9668  9701  9509  9573  9490  9411  8468
    6403   1 2021-05-01 10920 10325  9729  9488  9532  9357  9438  9277  9283  8327
    6404   1 2021-05-01 10786 10266  9635  9353  9394  9214  9286  9122  9113  8174
    6405   1 2021-05-01 10940 10624  9952  9651  9676  9474  9525  9483  9332  8365
    6406   1 2021-05-01 11263 10868 10200  9913  9947  9739  9781  9736  9595  8736
    6407   1 2021-05-01 11367 10915 10264  9995 10044  9856  9901  9811  9741  8908
    6408   1 2021-05-01 11292 10709 10049  9764  9821  9634  9687  9553  9511  8845
    6409   1 2021-05-01 11195 10642  9999  9729  9786  9606  9662  9551  9499  8749
    6410   1 2021-05-01 10940 10411  9800  9553  9611  9437  9504  9338  9339  8475
    6411   1 2021-05-01 10467 10154  9580  9349  9401  9251  9338  9160  9195  7978
    6412   1 2021-05-01 10518  9994  9400  9144  9189  9048  9124  9005  8963  8055
    6413   1 2021-05-01 10448 10083  9475  9226  9277  9127  9199  9108  9047  8105
    6414   1 2021-05-01 10410 10058  9454  9190  9241  9093  9170  9096  9017  8062
    6415   1 2021-05-01 10316  9975  9365  9113  9163  9029  9093  9033  8943  8051
    6416   1 2021-05-01 10221  9774  9189  8942  9006  8883  8953  8907  8809  8060
    6417   1 2021-05-01 10093  9673  9095  8852  8921  8802  8873  8818  8740  8027
    6418   1 2021-05-01  9893  9361  8794  8550  8611  8513  8596  8553  8451  7836
    6419   1 2021-05-01  9399  9217  8652  8397  8456  8348  8428  8422  8291  7393
    6420   1 2021-05-01  9287  9207  8625  8363  8420  8306  8386  8417  8255  7272
    6421   1 2021-05-01  9716  9553  8974  8721  8792  8670  8734  8747  8607  7746
    6422   1 2021-05-01  9924  9553  8974  8721  8792  8670  8734  8747  8607  8018
    6423   1 2021-05-01 10058  9766  9167  8918  8990  8852  8913  8907  8776  8154
    6424   1 2021-05-01  9893  9658  9076  8831  8910  8771  8845  8822  8710  7989
    6425   1 2021-05-01  9918  9688  9102  8854  8923  8805  8860  8858  8726  7991
    6426   1 2021-05-01 10031  9550  8968  8725  8785  8683  8734  8726  8604  8075
    6427   1 2021-05-01  9994  9550  8983  8752  8820  8713  8772  8707  8643  8015
    6428   1 2021-05-01  9462  8912  8391  8174  8237  8145  8240  8136  8123  7385
    6429   1 2021-05-01  9245  8741  8210  7972  8023  7941  8039  7967  7928  7132
    6430   1 2021-05-01  9048  8930  8365  8097  8138  8031  8107  8076  7976  6818
    6431   1 2021-05-01  9226  8802  8236  7973  8013  7918  8003  7924  7860  7026
    6432   1 2021-05-01  9211  8988  8421  8156  8203  8093  8162  8105  8027  7001
    6433   1 2021-05-01 13692 12892 12196 11931 11997 11689 11727 11391 11492 10629
    6434   1 2021-05-01 13482 12567 11807 11511 11584 11307 11360 11039 11148 10611
    6435   1 2021-05-01 13225 12261 11499 11187 11253 10976 11019 10742 10804 10372
    6436   1 2021-05-01 12979 12249 11471 11159 11195 10916 10965 10734 10745 10117
    6437   1 2021-05-01 12699 12012 11237 10876 10918 10634 10641 10487 10413 10039
    6438   1 2021-05-01 12757 12082 11315 10967 11004 10722 10728 10576 10495 10039
    6439   1 2021-05-01 12690 12007 11236 10881 10919 10631 10654 10515 10419  9914
    6440   1 2021-05-01 12633 12105 11356 11029 11062 10780 10800 10698 10565  9834
    6441   1 2021-05-01 12161 11611 10897 10589 10644 10382 10422 10227 10199  9429
    6442   1 2021-05-01 11939 11337 10624 10298 10349 10078 10114  9933  9895  9240
    6443   1 2021-05-01 11686 11098 10429 10134 10161  9934  9982  9821  9782  8860
    6444   1 2021-05-01 11507 10717 10056  9778  9798  9581  9646  9485  9460  8646
    6445   1 2021-05-01 11154 10900 10184  9880  9886  9641  9674  9635  9467  8328
    6446   1 2021-05-01 11289 10815 10119  9812  9837  9596  9627  9569  9420  8585
    6447   1 2021-05-01 11478 11054 10336 10019 10048  9800  9827  9748  9620  8865
    6448   1 2021-05-01 11485 11023 10297  9981 10028  9765  9774  9710  9567  8998
    6449   1 2021-05-01 11497 11016 10284  9967 10009  9756  9764  9682  9541  9034
    6450   1 2021-05-01 11656 10992 10268  9934  9989  9726  9734  9647  9508  9265
    6451   1 2021-05-01 11637 11142 10410 10082 10138  9874  9879  9802  9677  9266
    6452   1 2021-05-01 11713 11210 10478 10151 10204  9940  9941  9883  9720  9360
    6453   1 2021-05-01 11783 11304 10575 10260 10311 10057 10053 10016  9839  9436
    6454   1 2021-05-01 11720 11282 10569 10269 10332 10079 10090 10013  9875  9319
    6455   1 2021-05-01 11703 11211 10477 10164 10215  9967  9970  9915  9752  9339
    6456   1 2021-05-01 11680 11153 10427 10114 10160  9922  9919  9842  9724  9322
    6457   1 2021-05-01 11625 11179 10471 10167 10220  9978  9987  9903  9804  9267
    6458   1 2021-05-01 11610 11204 10489 10185 10229  9980  9983  9967  9794  9166
    6459   1 2021-05-01 11519 11023 10321 10026 10067  9829  9857  9810  9657  9042
    6460   1 2021-05-01 11450 11133 10415 10112 10161  9918  9941  9876  9752  8979
    6461   1 2021-05-01 11418 10948 10236  9919  9966  9737  9756  9688  9558  8940
    6462   1 2021-05-01 11382 10946 10258  9981 10008  9785  9806  9779  9615  8865
    6463   1 2021-05-01 10838 10736 10073  9803  9832  9631  9678  9615  9507  8223
    6464   1 2021-05-01 10907 10531  9861  9572  9601  9403  9439  9394  9267  8316
    6465   1 2021-05-01 11127 10751 10061  9770  9809  9601  9634  9563  9456  8588
    6466   1 2021-05-01 11268 10864 10185  9907  9952  9739  9762  9691  9601  8800
    6467   1 2021-05-01 11229 10765 10083  9807  9851  9658  9697  9592  9518  8788
    6468   1 2021-05-01 11140 10665  9992  9710  9753  9557  9591  9501  9424  8666
    6469   1 2021-05-01 10949 10434  9796  9523  9564  9369  9431  9342  9258  8417
    6470   1 2021-05-01 10806 10290  9658  9399  9432  9251  9322  9217  9163  8243
    6471   1 2021-05-01 10688 10400  9755  9489  9512  9318  9386  9351  9219  8040
    6472   1 2021-05-01 10826 10251  9612  9338  9351  9172  9245  9214  9086  8193
    6473   1 2021-05-01 10779 10370  9721  9435  9467  9283  9347  9254  9180  8144
    6474   1 2021-05-01 10817 10370  9721  9435  9467  9283  9347  9254  9180  8192
    6475   1 2021-05-01 11075 10527  9853  9552  9588  9384  9435  9344  9245  8528
    6476   1 2021-05-01 11258 10868 10200  9913  9947  9739  9781  9736  9595  8767
    6477   1 2021-05-01 11242 10881 10220  9938  9983  9779  9819  9784  9644  8786
    6478   1 2021-05-01 11289 10843 10181  9902  9959  9755  9805  9718  9628  8855
    6479   1 2021-05-01 11212 10632 10003  9741  9799  9610  9675  9582  9523  8761
    6480   1 2021-05-01 10717 10363  9756  9506  9542  9382  9447  9358  9288  8166
    6481   1 2021-05-01 10515 10192  9562  9285  9312  9150  9214  9162  9054  7984
    6482   1 2021-05-01 10644 10180  9567  9306  9360  9198  9258  9165  9096  8228
    6483   1 2021-05-01 10543 10050  9477  9245  9305  9160  9246  9088  9091  8201
    6484   1 2021-05-01 10090  9660  9089  8839  8890  8769  8860  8772  8720  7812
    6485   1 2021-05-01 10006  9637  9074  8829  8888  8765  8845  8759  8703  7709
    6486   1 2021-05-01 10058  9478  8924  8690  8757  8653  8734  8661  8601  7864
    6487   1 2021-05-01  9946  9478  8924  8690  8757  8653  8734  8661  8601  7791
    6488   1 2021-05-01  9361  9363  8801  8552  8616  8518  8589  8558  8463  7248
    6489   1 2021-05-01  9200  9361  8794  8550  8611  8513  8596  8553  8451  7148
    6490   1 2021-05-01  9159  8821  8275  8031  8089  8015  8097  8059  7970  7115
    6491   1 2021-05-01  8696  8725  8176  7916  7962  7884  7978  7975  7843  6697
    6492   1 2021-05-01  8693  8543  7985  7707  7748  7665  7758  7851  7627  6691
    6493   1 2021-05-01  9321  9161  8593  8337  8406  8284  8375  8387  8239  7337
    6494   1 2021-05-01  9644  9171  8597  8342  8413  8297  8368  8378  8240  7717
    6495   1 2021-05-01  9612  9353  8788  8550  8625  8504  8579  8569  8453  7666
    6496   1 2021-05-01  9687  9268  8699  8462  8528  8418  8493  8473  8369  7757
    6497   1 2021-05-01  9781  9359  8799  8570  8650  8545  8611  8554  8488  7811
    6498   1 2021-05-01  9712  9359  8793  8546  8625  8509  8573  8538  8443  7747
    6499   1 2021-05-01  9810  9380  8816  8577  8646  8535  8606  8552  8474  7827
    6500   1 2021-05-01  9273  8912  8391  8174  8237  8145  8240  8136  8123  7152
    6501   1 2021-05-01  9124  8662  8171  7958  8022  7941  8046  7886  7931  6942
    6502   1 2021-05-01  8902  8606  8057  7804  7849  7762  7853  7770  7724  6643
    6503   1 2021-05-01  9118  8631  8055  7778  7808  7706  7787  7757  7643  6871
    6504   1 2021-05-01  9151  8839  8267  7999  8042  7932  8014  7929  7881  6905
    6505   1 2021-05-01  9325  8911  8339  8063  8115  8007  8082  7995  7947  7105
    6506   1 2021-05-01 13574 13037 12278 11959 11987 11682 11706 11482 11431 10281
    6507   1 2021-05-01 13653 12877 12159 11879 11911 11597 11631 11359 11390 10380
    6508   1 2021-05-01 13266 12545 11839 11579 11669 11381 11438 11039 11215 10199
    6509   1 2021-05-01 13062 12303 11589 11303 11392 11115 11161 10784 10923 10146
    6510   1 2021-05-01 12952 12273 11518 11210 11266 10973 11023 10764 10797  9932
    6511   1 2021-05-01 12900 12226 11433 11117 11162 10886 10935 10701 10738  9893
    6512   1 2021-05-01 12834 12324 11515 11151 11175 10853 10858 10814 10606 10071
    6513   1 2021-05-01 12811 12305 11532 11173 11199 10902 10907 10815 10659 10030
    6514   1 2021-05-01 12717 12229 11490 11160 11168 10886 10913 10825 10687  9689
    6515   1 2021-05-01 12644 12089 11338 11014 11039 10752 10780 10678 10558  9595
    6516   1 2021-05-01 12471 12089 11338 11014 11039 10752 10780 10678 10558  9658
    6517   1 2021-05-01 12343 11799 11052 10723 10771 10495 10507 10388 10273  9636
    6518   1 2021-05-01 12136 11618 10872 10533 10576 10302 10320 10184 10101  9478
    6519   1 2021-05-01 11796 11279 10606 10309 10345 10108 10160  9960  9943  9043
    6520   1 2021-05-01 11225 11068 10415 10133 10183  9957 10005  9768  9806  8420
    6521   1 2021-05-01 11193 10626  9946  9656  9676  9455  9506  9376  9305  8433
    6522   1 2021-05-01 11127 10762 10077  9781  9813  9584  9623  9485  9412  8386
    6523   1 2021-05-01 11301 10862 10146  9828  9852  9615  9629  9571  9423  8693
    6524   1 2021-05-01 11545 11058 10316  9992 10031  9774  9781  9707  9567  9057
    6525   1 2021-05-01 11654 11256 10517 10192 10228  9966  9974  9934  9760  9244
    6526   1 2021-05-01 11757 11267 10544 10230 10294 10029 10041  9956  9839  9331
    6527   1 2021-05-01 11764 11228 10507 10189 10246  9984  9999  9930  9792  9327
    6528   1 2021-05-01 11666 11165 10439 10121 10159  9904  9912  9878  9691  9237
    6529   1 2021-05-01 11549 10975 10234  9901  9934  9678  9681  9653  9466  9138
    6530   1 2021-05-01 11561 11176 10452 10135 10179  9922  9926  9913  9715  9128
    6531   1 2021-05-01 11596 11105 10377 10058 10101  9849  9858  9809  9654  9181
    6532   1 2021-05-01 11686 11244 10511 10197 10244  9996  9997  9941  9794  9284
    6533   1 2021-05-01 11700 11236 10521 10216 10264 10019 10027  9969  9825  9295
    6534   1 2021-05-01 11572 11109 10393 10077 10125  9877  9897  9846  9697  9155
    6535   1 2021-05-01 11468 11002 10302  9986 10027  9787  9807  9748  9609  9013
    6536   1 2021-05-01 11343 10903 10197  9890  9939  9710  9739  9650  9549  8873
    6537   1 2021-05-01 11381 10860 10174  9870  9916  9685  9712  9626  9524  8877
    6538   1 2021-05-01 11278 10589  9944  9682  9726  9524  9578  9437  9410  8744
    6539   1 2021-05-01 10877 10456  9795  9508  9549  9348  9390  9284  9211  8281
    6540   1 2021-05-01 11052 10612  9929  9640  9681  9479  9518  9421  9339  8494
    6541   1 2021-05-01 11202 10842 10158  9869  9905  9690  9719  9681  9550  8679
    6542   1 2021-05-01 11276 10820 10142  9863  9905  9702  9735  9668  9562  8770
    6543   1 2021-05-01 11275 10711 10033  9762  9802  9612  9647  9552  9483  8810
    6544   1 2021-05-01 11112 10711 10033  9762  9802  9612  9647  9552  9483  8638
    6545   1 2021-05-01 11053 10546  9890  9610  9653  9456  9493  9396  9323  8566
    6546   1 2021-05-01 10798 10383  9746  9485  9533  9352  9406  9273  9248  8243
    6547   1 2021-05-01 10582 10136  9504  9242  9271  9090  9165  9074  9005  7970
    6548   1 2021-05-01 10416  9911  9301  9040  9063  8903  8985  8888  8834  7668
    6549   1 2021-05-01 10472 10229  9578  9290  9305  9120  9189  9144  9018  7714
    6550   1 2021-05-01 10714 10483  9809  9518  9543  9346  9400  9336  9225  8028
    6551   1 2021-05-01 10939 10497  9819  9518  9539  9337  9389  9377  9209  8328
    6552   1 2021-05-01 10943 10760 10095  9798  9841  9637  9682  9631  9491  8343
    6553   1 2021-05-01 11198 10757 10091  9802  9855  9656  9697  9624  9517  8749
    6554   1 2021-05-01 11163 10608  9990  9729  9785  9600  9662  9553  9500  8699
    6555   1 2021-05-01 10862 10121  9564  9333  9385  9236  9325  9133  9181  8364
    6556   1 2021-05-01 10384  9939  9331  9064  9100  8953  9033  8940  8877  7861
    6557   1 2021-05-01 10606  9939  9331  9064  9100  8953  9033  8940  8877  8126
    6558   1 2021-05-01 10754 10276  9630  9347  9402  9231  9282  9187  9112  8392
    6559   1 2021-05-01 10717 10266  9650  9391  9448  9289  9345  9239  9178  8401
    6560   1 2021-05-01 10361  9745  9187  8954  9018  8892  8989  8806  8844  8072
    6561   1 2021-05-01 10097  9766  9192  8948  9017  8879  8957  8797  8796  7849
    6562   1 2021-05-01  9871  9489  8936  8695  8756  8648  8729  8630  8581  7693
    6563   1 2021-05-01  9288  8697  8207  7991  8059  7996  8099  7960  7971  7142
    6564   1 2021-05-01  8048  8181  7651  7391  7434  7388  7493  7519  7372  6089
    6565   1 2021-05-01  7548  7881  7369  7111  7140  7094  7199  7247  7086  5572
    6566   1 2021-05-01  8321  8256  7725  7470  7510  7447  7540  7547  7417  6297
    6567   1 2021-05-01  8975  8772  8219  7967  8027  7927  8016  8020  7884  6925
    6568   1 2021-05-01  9048  8771  8221  7961  8021  7931  8013  7997  7882  7046
    6569   1 2021-05-01  9317  9107  8534  8282  8351  8237  8312  8314  8176  7317
    6570   1 2021-05-01  9502  9084  8516  8272  8336  8230  8308  8274  8175  7531
    6571   1 2021-05-01  9577  9220  8647  8396  8457  8342  8411  8391  8278  7566
    6572   1 2021-05-01  9610  8867  8347  8117  8184  8093  8180  8015  8058  7485
    6573   1 2021-05-01  8985  8441  7969  7780  7844  7797  7920  7699  7816  6788
    6574   1 2021-05-01  8455  7959  7428  7173  7196  7148  7271  7197  7150  6138
    6575   1 2021-05-01  7926  8008  7466  7197  7212  7141  7249  7219  7133  5588
    6576   1 2021-05-01  8780  8693  8116  7835  7869  7770  7841  7834  7711  6513
    6577   1 2021-05-01  9197  8781  8214  7959  7999  7912  7982  7917  7847  6972
    6578   1 2021-05-01 13337 12822 12061 11734 11740 11405 11425 11256 11165 10066
    6579   1 2021-05-01 13302 12668 11889 11533 11566 11234 11250 11090 10996 10082
    6580   1 2021-05-01 13188 12559 11807 11473 11516 11202 11219 11009 10951 10118
    6581   1 2021-05-01 13130 12580 11831 11494 11528 11219 11243 11035 10988 10108
    6582   1 2021-05-01 13122 12527 11814 11526 11565 11270 11308 11046 11084  9982
    6583   1 2021-05-01 13168 12277 11500 11203 11263 10989 11033 10765 10827  9910
    6584   1 2021-05-01 12928 12277 11500 11203 11263 10989 11033 10765 10827  9944
    6585   1 2021-05-01 12763 12253 11428 11062 11089 10781 10783 10729 10549  9918
    6586   1 2021-05-01 12789 12246 11460 11116 11151 10839 10858 10749 10621  9930
    6587   1 2021-05-01 12894 12419 11634 11267 11287 10988 11005 10947 10772  9872
    6588   1 2021-05-01 12644 12152 11428 11105 11109 10834 10866 10755 10641  9605
    6589   1 2021-05-01 12479 12015 11253 10910 10914 10631 10656 10599 10421  9370
    6590   1 2021-05-01 12426 11943 11166 10802 10832 10539 10544 10481 10312  9743
    6591   1 2021-05-01 12357 11779 11012 10660 10700 10422 10427 10330 10205  9720
    6592   1 2021-05-01 12189 11717 10963 10615 10652 10376 10390 10318 10163  9501
    6593   1 2021-05-01 12036 11435 10747 10450 10502 10261 10291 10115 10093  9259
    6594   1 2021-05-01 11788 11272 10575 10273 10324 10078 10115  9908  9917  9123
    6595   1 2021-05-01 11532 10964 10272  9986 10024  9782  9826  9658  9623  8852
    6596   1 2021-05-01 11319 10874 10164  9855  9884  9646  9670  9565  9457  8652
    6597   1 2021-05-01 11419 11168 10438 10125 10156  9901  9905  9846  9693  8807
    6598   1 2021-05-01 11716 11110 10376 10061 10097  9849  9856  9769  9645  9227
    6599   1 2021-05-01 11717 11228 10504 10198 10240  9979  9997  9910  9794  9267
    6600   1 2021-05-01 11698 11263 10534 10222 10263 10006 10012  9970  9806  9252
    6601   1 2021-05-01 11625 11122 10401 10090 10139  9885  9902  9829  9701  9169
    6602   1 2021-05-01 11437 11036 10315  9990 10035  9780  9795  9733  9587  9012
    6603   1 2021-05-01 11435 10942 10201  9867  9907  9651  9659  9591  9452  9002
    6604   1 2021-05-01 11487 11046 10308  9986 10028  9773  9783  9735  9579  9052
    6605   1 2021-05-01 11532 11042 10312 10000 10033  9781  9794  9764  9590  9096
    6606   1 2021-05-01 11596 11151 10419 10105 10146  9899  9902  9871  9701  9185
    6607   1 2021-05-01 11587 11104 10387 10080 10117  9876  9881  9834  9693  9156
    6608   1 2021-05-01 11585 11090 10377 10066 10110  9875  9895  9804  9698  9113
    6609   1 2021-05-01 11559 11013 10317 10024 10071  9841  9872  9764  9690  9087
    6610   1 2021-05-01 11453 11002 10288  9981 10021  9786  9807  9736  9619  8979
    6611   1 2021-05-01 11464 10973 10267  9955  9997  9764  9788  9705  9590  8928
    6612   1 2021-05-01 11250 10833 10171  9900  9953  9743  9791  9651  9611  8720
    6613   1 2021-05-01 11061 10565  9887  9603  9639  9428  9475  9369  9305  8498
    6614   1 2021-05-01 11102 10673  9992  9721  9755  9555  9607  9485  9434  8561
    6615   1 2021-05-01 11043 10704 10023  9738  9777  9564  9608  9548  9437  8525
    6616   1 2021-05-01 11063 10647  9967  9678  9720  9512  9546  9507  9377  8538
    6617   1 2021-05-01 11154 10720 10041  9770  9808  9606  9643  9574  9474  8629
    6618   1 2021-05-01 11090 10632  9941  9656  9698  9487  9531  9470  9360  8551
    6619   1 2021-05-01 10918 10477  9816  9548  9589  9398  9448  9346  9280  8396
    6620   1 2021-05-01 10756 10311  9661  9396  9430  9252  9311  9192  9147  8192
    6621   1 2021-05-01 10531  9877  9261  8994  9017  8862  8943  8821  8785  7882
    6622   1 2021-05-01 10159  9878  9255  8973  8975  8813  8901  8841  8730  7385
    6623   1 2021-05-01 10147  9735  9134  8864  8875  8721  8811  8700  8655  7365
    6624   1 2021-05-01 10095  9992  9342  9046  9053  8880  8948  8924  8776  7329
    6625   1 2021-05-01 10637  9992  9342  9046  9053  8880  8948  8924  8776  7975
    6626   1 2021-05-01 10867 10588  9907  9608  9641  9427  9472  9452  9282  8317
    6627   1 2021-05-01 11153 10689 10009  9717  9759  9554  9586  9544  9405  8682
    6628   1 2021-05-01 10973 10624  9984  9721  9772  9584  9647  9550  9479  8494
    6629   1 2021-05-01 10785 10384  9760  9497  9540  9366  9443  9336  9279  8294
    6630   1 2021-05-01 10821 10217  9619  9364  9424  9262  9325  9146  9168  8370
    6631   1 2021-05-01 10764 10452  9811  9539  9589  9418  9467  9368  9310  8331
    6632   1 2021-05-01 10862 10409  9778  9517  9573  9410  9458  9349  9289  8545
    6633   1 2021-05-01 10811 10368  9749  9487  9558  9388  9438  9317  9277  8533
    6634   1 2021-05-01 10653 10155  9545  9279  9348  9187  9238  9088  9068  8380
    6635   1 2021-05-01 10459  9681  9111  8871  8945  8817  8898  8726  8747  8210
    6636   1 2021-05-01 10215  9396  8867  8647  8734  8631  8723  8509  8591  8035
    6637   1 2021-05-01  9281  8515  8023  7817  7879  7839  7963  7816  7856  7161
    6638   1 2021-05-01  8042  7734  7325  7144  7223  7216  7367  7151  7279  5984
    6639   1 2021-05-01  7023  6402  6064  5910  5993  6074  6280  6016  6244  5113
    6640   1 2021-05-01  7020  7110  6572  6263  6252  6224  6328  6559  6207  5093
    6641   1 2021-05-01  8249  7665  7155  6893  6922  6883  6988  7032  6868  6191
    6642   1 2021-05-01  8050  8283  7727  7451  7486  7424  7509  7574  7382  6092
    6643   1 2021-05-01  8645  8369  7803  7510  7543  7470  7552  7628  7422  6637
    6644   1 2021-05-01  9312  8939  8367  8106  8168  8069  8137  8110  8003  7293
    6645   1 2021-05-01  9534  9009  8446  8194  8257  8161  8232  8189  8104  7517
    6646   1 2021-05-01  9402  8951  8396  8155  8226  8122  8208  8135  8085  7387
    6647   1 2021-05-01  9324  9107  8529  8266  8309  8197  8254  8270  8115  7297
    6648   1 2021-05-01  9705  9205  8634  8391  8458  8346  8415  8275  8284  7627
    6649   1 2021-05-01  8680  8732  8218  8008  8070  7995  8091  7920  7981  6451
    6650   1 2021-05-01  8380  7794  7360  7172  7223  7204  7354  7106  7266  6083
    6651   1 2021-05-01  8484  8018  7516  7274  7307  7245  7358  7214  7243  6125
    6652   1 2021-05-01  8745  8416  7875  7634  7673  7590  7684  7585  7554  6472
    6653   1 2021-05-01  9008  8678  8111  7860  7901  7809  7876  7817  7749  6769
    6654   1 2021-05-01  9250  8940  8356  8082  8124  8013  8083  8030  7942  7065
    6655   1 2021-05-01 12279 11782 11018 10698 10705 10412 10448 10275 10209  9227
    6656   1 2021-05-01 12287 11777 11015 10690 10678 10392 10425 10262 10200  9154
    6657   1 2021-05-01 13365 12617 11805 11420 11414 11071 11072 10999 10801  9959
    6658   1 2021-05-01 13173 12754 11956 11591 11592 11253 11273 11151 11006  9957
    6659   1 2021-05-01 13185 12705 11878 11493 11498 11156 11158 11089 10888 10005
    6660   1 2021-05-01 13127 12597 11801 11423 11463 11137 11148 11007 10898 10097
    6661   1 2021-05-01 13174 12580 11831 11494 11528 11219 11243 11035 10988 10081
    6662   1 2021-05-01 13261 12703 11961 11647 11664 11363 11395 11194 11147  9912
    6663   1 2021-05-01 13006 12347 11616 11326 11371 11091 11132 10861 10923  9874
    6664   1 2021-05-01 12794 12166 11395 11070 11107 10820 10854 10652 10635  9822
    6665   1 2021-05-01 12708 12251 11456 11108 11117 10819 10837 10764 10594  9773
    6666   1 2021-05-01 12553 12176 11421 11083 11071 10783 10825 10741 10589  9315
    6667   1 2021-05-01 12606 12146 11393 11055 11064 10786 10816 10717 10587  9532
    6668   1 2021-05-01 12485 11978 11224 10875 10907 10631 10647 10511 10424  9594
    6669   1 2021-05-01 12461 11972 11204 10841 10875 10581 10594 10505 10365  9729
    6670   1 2021-05-01 12480 12015 11234 10887 10931 10640 10659 10559 10437  9827
    6671   1 2021-05-01 12391 11932 11157 10802 10839 10544 10552 10482 10324  9767
    6672   1 2021-05-01 12192 11818 11060 10713 10759 10474 10487 10399 10266  9511
    6673   1 2021-05-01 12179 11682 10966 10646 10680 10423 10439 10350 10219  9476
    6674   1 2021-05-01 12005 11272 10575 10273 10324 10078 10115  9908  9917  9335
    6675   1 2021-05-01 11788 11268 10557 10259 10313 10061 10093  9922  9885  9218
    6676   1 2021-05-01 11727 11052 10352 10058 10099  9856  9897  9745  9699  9175
    6677   1 2021-05-01 11656 11173 10454 10153 10192  9946  9968  9856  9766  9100
    6678   1 2021-05-01 11708 11260 10534 10232 10274 10018 10030  9950  9828  9214
    6679   1 2021-05-01 11734 11282 10564 10262 10312 10060 10076  9996  9875  9276
    6680   1 2021-05-01 11693 11138 10430 10129 10175  9937  9950  9847  9748  9231
    6681   1 2021-05-01 11586 11028 10321 10013 10062  9823  9845  9733  9643  9129
    6682   1 2021-05-01 11462 10921 10195  9874  9920  9684  9704  9598  9504  9027
    6683   1 2021-05-01 11362 10885 10173  9861  9893  9664  9685  9618  9486  8906
    6684   1 2021-05-01 11340 10852 10119  9799  9845  9603  9611  9552  9411  8883
    6685   1 2021-05-01 11323 10852 10119  9799  9845  9603  9611  9552  9411  8861
    6686   1 2021-05-01 11442 10990 10269  9943  9978  9738  9748  9725  9550  8966
    6687   1 2021-05-01 11407 11104 10387 10080 10117  9876  9881  9834  9693  8901
    6688   1 2021-05-01 11483 11092 10378 10081 10117  9880  9895  9848  9693  8930
    6689   1 2021-05-01 11526 11125 10410 10114 10154  9926  9942  9882  9761  8976
    6690   1 2021-05-01 11456 10981 10270  9960  9987  9758  9775  9743  9587  8922
    6691   1 2021-05-01 11405 10969 10279  9996 10035  9807  9837  9767  9657  8819
    6692   1 2021-05-01 11163 10748 10081  9809  9854  9640  9685  9588  9520  8583
    6693   1 2021-05-01 11163 10727 10046  9773  9804  9594  9649  9560  9464  8607
    6694   1 2021-05-01 11111 10640  9973  9698  9738  9528  9578  9504  9415  8563
    6695   1 2021-05-01 11064 10524  9870  9593  9634  9439  9494  9388  9326  8531
    6696   1 2021-05-01 10937 10440  9779  9500  9534  9349  9395  9313  9227  8374
    6697   1 2021-05-01 10935 10438  9762  9477  9510  9313  9362  9295  9191  8383
    6698   1 2021-05-01 10840 10306  9638  9349  9370  9171  9230  9168  9045  8299
    6699   1 2021-05-01 10862 10453  9779  9498  9530  9332  9383  9313  9201  8333
    6700   1 2021-05-01 10759 10311  9661  9396  9430  9252  9311  9192  9147  8193
    6701   1 2021-05-01 10090  9757  9172  8923  8955  8807  8903  8725  8755  7395
    6702   1 2021-05-01  9929  9515  8915  8640  8651  8512  8613  8503  8454  7193
    6703   1 2021-05-01 10078  9721  9107  8829  8849  8692  8767  8640  8606  7317
    6704   1 2021-05-01 10092  9747  9127  8857  8878  8729  8814  8675  8649  7361
    6705   1 2021-05-01 10797 10297  9616  9313  9337  9140  9186  9111  9000  8269
    6706   1 2021-05-01 11036 10570  9895  9592  9636  9429  9463  9408  9290  8567
    6707   1 2021-05-01 11026 10570  9913  9622  9668  9470  9518  9425  9333  8590
    6708   1 2021-05-01 11013 10417  9794  9526  9576  9399  9471  9364  9295  8582
    6709   1 2021-05-01 10895 10514  9870  9593  9651  9466  9523  9437  9351  8489
    6710   1 2021-05-01 10990 10555  9914  9639  9697  9509  9557  9480  9386  8619
    6711   1 2021-05-01 10960 10570  9929  9666  9722  9544  9587  9531  9431  8615
    6712   1 2021-05-01 10978 10544  9902  9632  9687  9518  9557  9482  9398  8687
    6713   1 2021-05-01 11004 10368  9749  9487  9558  9388  9438  9317  9277  8756
    6714   1 2021-05-01 10917 10446  9811  9542  9611  9446  9475  9368  9319  8698
    6715   1 2021-05-01 10754 10119  9506  9258  9328  9176  9231  9085  9067  8579
    6716   1 2021-05-01 10464  9912  9323  9076  9154  9016  9082  8936  8928  8326
    6717   1 2021-05-01  9972  8859  8408  8249  8351  8289  8412  8101  8302  7816
    6718   1 2021-05-01  8708  8529  8075  7902  7998  7955  8092  7789  7987  6588
    6719   1 2021-05-01  8787  7484  7148  7019  7113  7128  7289  6908  7216  6634
    6720   1 2021-05-01  7773  7589  7226  7073  7170  7174  7319  6966  7239  5732
    6721   1 2021-05-01  7039  7229  6790  6588  6665  6678  6833  6666  6752  5160
    6722   1 2021-05-01  8143  7893  7413  7186  7248  7209  7318  7206  7212  6117
    6723   1 2021-05-01  8264  7900  7430  7208  7276  7254  7372  7259  7284  6270
    6724   1 2021-05-01  8337  8257  7742  7502  7564  7515  7623  7561  7512  6328
    6725   1 2021-05-01  9076  8830  8271  8011  8063  7969  8050  8028  7914  7038
    6726   1 2021-05-01  9173  9009  8446  8194  8257  8161  8232  8189  8104  7169
    6727   1 2021-05-01  9220  8858  8325  8088  8156  8067  8152  8067  8025  7172
    6728   1 2021-05-01  8996  8741  8187  7944  8002  7913  8005  7938  7889  6964
    6729   1 2021-05-01  9117  8879  8308  8054  8111  8008  8089  8053  7962  7075
    6730   1 2021-05-01  9213  8727  8199  7978  8033  7960  8052  7928  7934  7129
    6731   1 2021-05-01  8920  8366  7849  7614  7665  7595  7696  7537  7575  6739
    6732   1 2021-05-01  8597  8195  7656  7403  7431  7359  7458  7338  7335  6345
    6733   1 2021-05-01  8915  8499  7937  7684  7720  7631  7719  7624  7593  6623
    6734   1 2021-05-01  8990  8568  8012  7767  7814  7731  7812  7711  7688  6734
    6735   1 2021-05-01  8938  8639  8094  7845  7891  7803  7878  7787  7754  6712
    6736   1 2021-05-01  9081  8884  8314  8044  8090  7975  8042  7980  7906  6886
    6737   1 2021-05-01 12321 11748 10978 10652 10666 10378 10401 10216 10172  9297
    6738   1 2021-05-01 12373 11766 10985 10653 10673 10380 10406 10198 10178  9343
    6739   1 2021-05-01 12271 11769 10994 10658 10650 10354 10387 10200 10144  9165
    6740   1 2021-05-01 12287 11895 11106 10764 10771 10477 10494 10297 10252  9051
    6741   1 2021-05-01 12225 11861 11045 10665 10679 10382 10380 10230 10136  9184
    6742   1 2021-05-01 13005 12498 11684 11291 11309 10972 10965 10857 10710  9762
    6743   1 2021-05-01 13011 12535 11710 11317 11333 10996 10997 10904 10717  9885
    6744   1 2021-05-01 13132 12687 11861 11490 11503 11180 11177 11079 10915 10041
    6745   1 2021-05-01 13159 12715 11894 11542 11556 11241 11250 11124 11003 10074
    6746   1 2021-05-01 13154 12725 11929 11592 11575 11266 11284 11179 11022  9916
    6747   1 2021-05-01 13092 12567 11768 11422 11410 11101 11124 11026 10865  9732
    6748   1 2021-05-01 12883 12289 11518 11188 11210 10919 10937 10775 10705  9784
    6749   1 2021-05-01 12681 11947 11200 10874 10861 10588 10631 10473 10404  9521
    6750   1 2021-05-01 12357 12089 11322 10989 10987 10710 10745 10608 10506  9076
    6751   1 2021-05-01 12399 11932 11170 10822 10811 10525 10559 10469 10321  9204
    6752   1 2021-05-01 12440 12030 11271 10914 10909 10615 10649 10577 10407  9289
    6753   1 2021-05-01 12605 12098 11340 10985 11004 10711 10724 10637 10491  9792
    6754   1 2021-05-01 12559 11999 11236 10870 10907 10616 10625 10527 10393  9887
    6755   1 2021-05-01 12608 12072 11309 10968 11011 10723 10738 10634 10505  9948
    6756   1 2021-05-01 12462 11929 11162 10814 10857 10584 10604 10506 10386  9846
    6757   1 2021-05-01 12221 11767 11020 10675 10709 10424 10438 10395 10210  9593
    6758   1 2021-05-01 12096 11651 10908 10574 10614 10333 10345 10295 10125  9489
    6759   1 2021-05-01 12039 11539 10827 10507 10564 10303 10321 10212 10107  9418
    6760   1 2021-05-01 11977 11438 10733 10439 10492 10242 10268 10152 10069  9420
    6761   1 2021-05-01 11725 11234 10531 10243 10276 10031 10065  9961  9871  9158
    6762   1 2021-05-01 11738 11271 10564 10270 10305 10056 10083 10007  9888  9164
    6763   1 2021-05-01 11800 11326 10604 10299 10351 10093 10115 10051  9906  9305
    6764   1 2021-05-01 11798 11326 10604 10299 10351 10093 10115 10051  9906  9324
    6765   1 2021-05-01 11622 11194 10491 10195 10241  9999 10020  9931  9816  9171
    6766   1 2021-05-01 11523 11056 10351 10045 10088  9842  9873  9782  9676  9081
    6767   1 2021-05-01 11435 10942 10245  9941  9986  9753  9777  9671  9583  8989
    6768   1 2021-05-01 11286 10771 10082  9777  9814  9586  9617  9524  9424  8814
    6769   1 2021-05-01 11242 10715 10000  9694  9727  9496  9524  9451  9325  8755
    6770   1 2021-05-01 11073 10748 10025  9704  9737  9503  9519  9482  9322  8551
    6771   1 2021-05-01 10912 10616  9900  9579  9599  9367  9397  9376  9196  8324
    6772   1 2021-05-01 11087 10847 10131  9817  9838  9600  9617  9610  9413  8497
    6773   1 2021-05-01 11246 10646  9943  9627  9649  9427  9446  9414  9248  8661
    6774   1 2021-05-01 11059 10650  9931  9604  9611  9389  9408  9402  9211  8410
    6775   1 2021-05-01 10881 10466  9745  9418  9426  9198  9223  9197  9021  8199
    6776   1 2021-05-01 11017 10670  9980  9683  9700  9489  9519  9491  9329  8334
    6777   1 2021-05-01 11122 10670  9980  9683  9700  9489  9519  9491  9329  8487
    6778   1 2021-05-01 11017 10638  9975  9699  9735  9530  9575  9491  9404  8385
    6779   1 2021-05-01 10990 10582  9915  9640  9678  9474  9513  9420  9339  8420
    6780   1 2021-05-01 10914 10495  9840  9567  9608  9407  9457  9366  9298  8337
    6781   1 2021-05-01 10851 10313  9666  9395  9429  9251  9306  9207  9144  8260
    6782   1 2021-05-01 10524 10264  9599  9317  9351  9165  9221  9142  9042  7920
    6783   1 2021-05-01 10583 10306  9638  9349  9370  9171  9230  9168  9045  8004
    6784   1 2021-05-01 10506  9980  9347  9071  9095  8925  8988  8903  8823  7922
    6785   1 2021-05-01 10083  9742  9124  8850  8864  8703  8778  8683  8617  7435
    6786   1 2021-05-01 10078  9760  9126  8837  8852  8689  8756  8655  8586  7382
    6787   1 2021-05-01 10297  9858  9227  8947  8973  8806  8864  8748  8707  7634
    6788   1 2021-05-01 10387  9994  9337  9042  9053  8879  8938  8868  8762  7742
    6789   1 2021-05-01 10531 10317  9647  9344  9381  9187  9234  9159  9057  7952
    6790   1 2021-05-01 10818 10587  9920  9632  9683  9481  9529  9469  9351  8300
    6791   1 2021-05-01 11129 10606  9932  9636  9682  9480  9525  9445  9357  8723
    6792   1 2021-05-01 11168 10688 10019  9737  9790  9597  9641  9551  9483  8778
    6793   1 2021-05-01 11099 10579  9931  9644  9696  9508  9558  9451  9389  8724
    6794   1 2021-05-01 10989 10561  9907  9629  9687  9499  9549  9468  9376  8610
    6795   1 2021-05-01 11047 10588  9938  9657  9720  9532  9570  9502  9398  8720
    6796   1 2021-05-01 11033 10570  9929  9666  9722  9544  9587  9531  9431  8736
    6797   1 2021-05-01 10997 10561  9918  9646  9707  9530  9571  9517  9408  8751
    6798   1 2021-05-01 11018 10550  9915  9646  9715  9537  9576  9517  9411  8779
    6799   1 2021-05-01 10780 10392  9769  9501  9577  9407  9453  9406  9292  8617
    6800   1 2021-05-01 10707 10159  9546  9288  9373  9223  9269  9159  9112  8578
    6801   1 2021-05-01 10508  9885  9302  9053  9134  9002  9064  8939  8918  8403
    6802   1 2021-05-01 10045  9298  8759  8534  8622  8509  8595  8405  8446  7941
    6803   1 2021-05-01  9873  9227  8687  8463  8551  8450  8527  8354  8386  7760
    6804   1 2021-05-01  9481  8690  8187  7978  8067  7994  8091  7846  7970  7381
    6805   1 2021-05-01  9031  8690  8187  7978  8067  7994  8091  7846  7970  6901
    6806   1 2021-05-01  8157  7770  7355  7181  7275  7269  7409  7133  7321  6132
    6807   1 2021-05-01  8309  8026  7544  7326  7400  7361  7481  7353  7382  6318
    6808   1 2021-05-01  8778  8285  7793  7570  7629  7577  7669  7553  7557  6732
    6809   1 2021-05-01  8623  8257  7742  7502  7564  7515  7623  7561  7512  6622
    6810   1 2021-05-01  8655  8490  7935  7682  7730  7663  7753  7745  7632  6661
    6811   1 2021-05-01  9046  8721  8186  7946  8010  7927  8016  7946  7891  7029
    6812   1 2021-05-01  9070  8662  8126  7895  7961  7882  7969  7874  7862  7032
    6813   1 2021-05-01  9123  8736  8189  7953  8009  7921  8010  7936  7891  7069
    6814   1 2021-05-01  9196  8818  8261  8013  8072  7978  8058  8002  7931  7116
    6815   1 2021-05-01  9349  8852  8287  8032  8084  7991  8070  8004  7949  7228
    6816   1 2021-05-01  8724  8364  7808  7541  7568  7483  7570  7532  7438  6408
    6817   1 2021-05-01  8559  8209  7668  7426  7461  7400  7495  7384  7373  6294
    6818   1 2021-05-01  8876  8358  7791  7530  7560  7462  7547  7526  7426  6593
    6819   1 2021-05-01  8977  8535  7992  7756  7804  7722  7808  7706  7692  6760
    6820   1 2021-05-01  9100  8639  8094  7845  7891  7803  7878  7787  7754  6902
    6821   1 2021-05-01 12291 11874 11105 10791 10795 10505 10541 10357 10302  9097
    6822   1 2021-05-01 12390 11840 11061 10735 10754 10459 10479 10283 10250  9360
    6823   1 2021-05-01 12471 11840 11044 10689 10697 10402 10424 10244 10180  9447
    6824   1 2021-05-01 12540 12032 11234 10884 10891 10584 10600 10431 10339  9481
    6825   1 2021-05-01 12619 11992 11186 10825 10839 10534 10547 10368 10307  9639
    6826   1 2021-05-01 12713 12356 11544 11189 11208 10893 10903 10739 10650  9777
    6827   1 2021-05-01 13188 12694 11885 11515 11551 11217 11213 11067 10954 10185
    6828   1 2021-05-01 13173 12555 11738 11351 11374 11043 11052 10917 10789 10136
    6829   1 2021-05-01 13094 12698 11867 11494 11503 11161 11180 11076 10919  9998
    6830   1 2021-05-01 13158 12634 11808 11423 11443 11112 11117 11013 10845 10071
    6831   1 2021-05-01 13130 12686 11851 11480 11480 11149 11153 11088 10901  9891
    6832   1 2021-05-01 12911 12621 11808 11460 11451 11142 11161 11056 10912  9642
    6833   1 2021-05-01 12790 12351 11535 11176 11166 10866 10889 10775 10634  9555
    6834   1 2021-05-01 12815 12302 11506 11152 11155 10846 10855 10762 10606  9695
    6835   1 2021-05-01 12669 12033 11273 10935 10941 10645 10666 10514 10424  9635
    6836   1 2021-05-01 12370 11798 11039 10693 10688 10410 10452 10312 10229  9340
    6837   1 2021-05-01 12355 11886 11107 10741 10763 10467 10490 10371 10257  9425
    6838   1 2021-05-01 12429 12036 11258 10885 10893 10591 10607 10564 10366  9527
    6839   1 2021-05-01 12533 12054 11277 10912 10928 10634 10640 10594 10401  9718
    6840   1 2021-05-01 12559 12063 11314 10968 11002 10714 10727 10634 10495  9822
    6841   1 2021-05-01 12606 11985 11234 10910 10956 10682 10713 10589 10490  9892
    6842   1 2021-05-01 12371 11749 10991 10648 10679 10402 10427 10370 10205  9676
    6843   1 2021-05-01 12227 11816 11061 10716 10744 10464 10483 10434 10273  9551
    6844   1 2021-05-01 11967 11670 10923 10590 10615 10346 10369 10321 10154  9387
    6845   1 2021-05-01 11920 11552 10816 10468 10510 10229 10238 10216 10008  9366
    6846   1 2021-05-01 11832 11420 10706 10374 10408 10144 10155 10125  9929  9258
    6847   1 2021-05-01 11748 11235 10543 10264 10304 10066 10107  9991  9917  9125
    6848   1 2021-05-01 11611 11085 10376 10080 10104  9863  9898  9845  9705  9023
    6849   1 2021-05-01 11528 11240 10530 10236 10274 10019 10051  9991  9839  8976
    6850   1 2021-05-01 11616 11123 10417 10120 10153  9905  9930  9891  9729  9107
    6851   1 2021-05-01 11611 11035 10338 10047 10084  9849  9881  9793  9685  9146
    6852   1 2021-05-01 11468 10795 10104  9803  9845  9613  9642  9533  9448  9012
    6853   1 2021-05-01 11109 10652  9962  9666  9696  9465  9496  9420  9306  8616
    6854   1 2021-05-01 11030 10499  9814  9517  9549  9332  9365  9270  9187  8518
    6855   1 2021-05-01 10867 10294  9592  9276  9302  9081  9119  9012  8927  8322
    6856   1 2021-05-01 10783 10330  9627  9299  9330  9106  9135  9058  8926  8243
    6857   1 2021-05-01 10766 10330  9627  9299  9330  9106  9135  9058  8926  8171
    6858   1 2021-05-01 10814 10504  9782  9465  9479  9250  9288  9250  9087  8175
    6859   1 2021-05-01 10977 10496  9776  9456  9480  9251  9283  9227  9083  8374
    6860   1 2021-05-01 10947 10453  9750  9437  9461  9239  9274  9173  9074  8339
    6861   1 2021-05-01 10942 10430  9715  9394  9421  9206  9226  9103  9035  8296
    6862   1 2021-05-01 10832 10436  9737  9416  9435  9220  9254  9207  9061  8135
    6863   1 2021-05-01 10780 10362  9686  9383  9399  9194  9238  9178  9050  8052
    6864   1 2021-05-01 10833 10477  9806  9524  9553  9344  9385  9327  9214  8131
    6865   1 2021-05-01 10763 10306  9637  9348  9381  9180  9231  9139  9053  8165
    6866   1 2021-05-01 10721 10228  9576  9299  9329  9142  9207  9097  9037  8123
    6867   1 2021-05-01 10551 10066  9407  9128  9154  8977  9041  8944  8863  7925
    6868   1 2021-05-01 10411  9878  9231  8938  8955  8783  8842  8770  8670  7802
    6869   1 2021-05-01 10420  9842  9202  8914  8932  8760  8818  8729  8652  7796
    6870   1 2021-05-01 10021  9513  8875  8579  8586  8425  8490  8384  8326  7345
    6871   1 2021-05-01  9906  9561  8933  8652  8666  8513  8583  8474  8426  7207
    6872   1 2021-05-01 10034  9592  8944  8635  8639  8475  8531  8490  8365  7329
    6873   1 2021-05-01 10185  9796  9156  8874  8894  8729  8798  8711  8636  7524
    6874   1 2021-05-01 10143  9810  9173  8884  8898  8738  8813  8740  8644  7535
    6875   1 2021-05-01 10503 10241  9562  9257  9273  9085  9130  9120  8948  7928
    6876   1 2021-05-01 10943 10424  9749  9449  9474  9277  9324  9343  9140  8506
    6877   1 2021-05-01 11074 10624  9955  9673  9718  9519  9575  9527  9406  8701
    6878   1 2021-05-01 11006 10576  9926  9649  9695  9515  9569  9484  9407  8606
    6879   1 2021-05-01 11020 10613  9953  9675  9731  9543  9589  9531  9406  8657
    6880   1 2021-05-01 11029 10614  9956  9681  9738  9553  9592  9550  9423  8699
    6881   1 2021-05-01 11043 10576  9929  9657  9727  9539  9583  9509  9424  8772
    6882   1 2021-05-01 10989 10538  9891  9613  9680  9507  9539  9480  9375  8761
    6883   1 2021-05-01 10900 10509  9878  9602  9667  9485  9528  9486  9361  8712
    6884   1 2021-05-01 10803 10363  9741  9472  9546  9376  9426  9360  9272  8641
    6885   1 2021-05-01 10680 10250  9625  9352  9425  9267  9305  9255  9149  8536
    6886   1 2021-05-01 10312  9981  9381  9125  9195  9053  9107  9028  8958  8241
    6887   1 2021-05-01 10198  9617  9031  8789  8866  8745  8809  8668  8666  8123
    6888   1 2021-05-01 10011  9509  8932  8690  8770  8655  8720  8597  8569  7953
    6889   1 2021-05-01  9687  9154  8596  8363  8442  8346  8420  8283  8288  7657
    6890   1 2021-05-01  9179  8962  8432  8206  8289  8207  8287  8136  8155  7179
    6891   1 2021-05-01  8998  8356  7884  7682  7769  7707  7815  7638  7711  7013
    6892   1 2021-05-01  8710  8327  7818  7589  7659  7594  7698  7594  7575  6723
    6893   1 2021-05-01  8754  8502  7972  7741  7804  7726  7818  7722  7699  6740
    6894   1 2021-05-01  8967  8321  7829  7621  7690  7641  7746  7624  7647  6961
    6895   1 2021-05-01  8707  8376  7862  7634  7696  7641  7734  7665  7630  6730
    6896   1 2021-05-01  8893  8537  7998  7751  7814  7743  7833  7790  7721  6863
    6897   1 2021-05-01  9026  8537  7998  7751  7814  7743  7833  7790  7721  6978
    6898   1 2021-05-01  8833  8723  8166  7918  7972  7889  7973  7890  7850  6728
    6899   1 2021-05-01  8948  8604  8054  7801  7855  7771  7855  7809  7733  6840
    6900   1 2021-05-01  8889  8589  8049  7800  7849  7776  7865  7803  7738  6765
    6901   1 2021-05-01  8499  7925  7470  7280  7337  7310  7428  7244  7339  6338
    6902   1 2021-05-01  7817  7330  6838  6592  6589  6576  6709  6648  6604  5421
    6903   1 2021-05-01  8332  8035  7492  7232  7257  7181  7274  7227  7152  6000
    6904   1 2021-05-01  8712  8495  7941  7690  7733  7636  7724  7649  7594  6426
    6905   1 2021-05-01  8926  8670  8101  7833  7874  7767  7847  7769  7707  6704
    6906   1 2021-05-01  9224  8829  8248  7980  8023  7907  7970  7896  7834  7001
    6907   1 2021-05-01 12185 11721 10960 10615 10611 10329 10367 10173 10133  9006
    6908   1 2021-05-01 12399 11866 11082 10761 10768 10477 10501 10318 10264  9270
    6909   1 2021-05-01 12451 11945 11146 10804 10814 10522 10540 10373 10295  9467
    6910   1 2021-05-01 12536 12146 11337 10979 10984 10671 10680 10553 10429  9567
    6911   1 2021-05-01 12722 12280 11474 11119 11112 10787 10794 10683 10533  9641
    6912   1 2021-05-01 12860 12449 11631 11284 11283 10961 10978 10839 10725  9703
    6913   1 2021-05-01 13092 12583 11764 11398 11390 11058 11074 10977 10810  9912
    6914   1 2021-05-01 13125 12678 11860 11480 11487 11164 11152 11066 10885 10096
    6915   1 2021-05-01 13197 12678 11860 11480 11487 11164 11152 11066 10885 10179
    6916   1 2021-05-01 13215 12806 11959 11564 11578 11234 11217 11169 10941 10228
    6917   1 2021-05-01 13101 12654 11820 11434 11465 11133 11128 11014 10873 10124
    6918   1 2021-05-01 13063 12555 11716 11335 11346 11017 11020 10925 10769  9890
    6919   1 2021-05-01 13005 12486 11655 11291 11299 10970 10988 10878 10735  9830
    6920   1 2021-05-01 12783 12283 11440 11067 11060 10749 10772 10668 10508  9641
    6921   1 2021-05-01 12797 12323 11500 11139 11123 10824 10833 10736 10580  9548
    6922   1 2021-05-01 12811 12386 11564 11202 11205 10904 10908 10809 10647  9739
    6923   1 2021-05-01 12837 12199 11412 11062 11074 10780 10796 10653 10542  9904
    6924   1 2021-05-01 12684 12175 11395 11047 11072 10781 10795 10642 10551  9830
    6925   1 2021-05-01 12545 11927 11148 10800 10830 10541 10559 10407 10319  9659
    6926   1 2021-05-01 12523 11927 11148 10800 10830 10541 10559 10407 10319  9635
    6927   1 2021-05-01 12613 12053 11280 10920 10941 10650 10675 10571 10438  9684
    6928   1 2021-05-01 12508 12054 11277 10912 10928 10634 10640 10594 10401  9751
    6929   1 2021-05-01 12507 12007 11251 10902 10926 10628 10646 10585 10409  9728
    6930   1 2021-05-01 12221 11912 11176 10838 10871 10590 10610 10528 10392  9467
    6931   1 2021-05-01 12103 11638 10898 10558 10587 10317 10343 10263 10129  9416
    6932   1 2021-05-01 11996 11553 10815 10484 10508 10250 10268 10220 10059  9325
    6933   1 2021-05-01 11904 11387 10656 10316 10346 10084 10107 10039  9888  9250
    6934   1 2021-05-01 11783 11361 10629 10280 10310 10040 10054 10011  9832  9281
    6935   1 2021-05-01 11658 11170 10466 10146 10175  9925  9947  9872  9724  9137
    6936   1 2021-05-01 11405 10882 10203  9916  9950  9714  9764  9659  9572  8785
    6937   1 2021-05-01 11302 10850 10154  9850  9886  9640  9679  9589  9476  8685
    6938   1 2021-05-01 11367 10929 10229  9937  9964  9723  9757  9701  9555  8807
    6939   1 2021-05-01 11400 10874 10172  9873  9906  9670  9701  9626  9508  8885
    6940   1 2021-05-01 11408 10978 10276  9981 10012  9773  9805  9733  9613  8917
    6941   1 2021-05-01 11296 10795 10104  9803  9845  9613  9642  9533  9448  8818
    6942   1 2021-05-01 11120 10586  9894  9594  9636  9403  9436  9328  9246  8654
    6943   1 2021-05-01 10932 10444  9764  9469  9501  9281  9311  9211  9134  8430
    6944   1 2021-05-01 10870 10395  9708  9404  9442  9223  9257  9116  9069  8364
    6945   1 2021-05-01 10925 10334  9625  9311  9343  9121  9161  9028  8952  8449
    6946   1 2021-05-01 10860 10350  9652  9339  9366  9151  9184  9072  8988  8312
    6947   1 2021-05-01 10841 10387  9676  9356  9382  9160  9194  9117  9001  8198
    6948   1 2021-05-01 10882 10419  9708  9388  9416  9195  9223  9135  9023  8242
    6949   1 2021-05-01 10926 10486  9769  9447  9464  9242  9271  9182  9066  8319
    6950   1 2021-05-01 10948 10465  9740  9412  9436  9213  9238  9144  9040  8322
    6951   1 2021-05-01 10857 10345  9663  9358  9390  9191  9226  9102  9032  8172
    6952   1 2021-05-01 10730 10213  9549  9258  9286  9097  9147  9015  8975  8019
    6953   1 2021-05-01 10670 10247  9576  9286  9303  9108  9160  9073  8982  7976
    6954   1 2021-05-01 10700 10306  9637  9348  9381  9180  9231  9139  9053  8077
    6955   1 2021-05-01 10644 10146  9487  9204  9227  9047  9100  8991  8932  8028
    6956   1 2021-05-01 10299  9970  9322  9048  9078  8911  8973  8837  8816  7643
    6957   1 2021-05-01 10130  9678  9032  8751  8771  8610  8670  8551  8525  7498
    6958   1 2021-05-01 10000  9593  8942  8650  8656  8494  8560  8488  8390  7338
    6959   1 2021-05-01  9821  9381  8728  8425  8428  8264  8336  8235  8169  7134
    6960   1 2021-05-01  9765  9306  8686  8396  8409  8262  8336  8213  8186  7082
    6961   1 2021-05-01  9644  9262  8633  8350  8356  8212  8295  8168  8143  6941
    6962   1 2021-05-01  9843  9638  9001  8706  8721  8551  8616  8570  8444  7201
    6963   1 2021-05-01 10068  9535  8935  8664  8692  8543  8623  8504  8475  7436
    6964   1 2021-05-01 10002  9838  9180  8872  8882  8708  8776  8811  8605  7389
    6965   1 2021-05-01 10640  9953  9307  9017  9032  8858  8923  8932  8746  8139
    6966   1 2021-05-01 10584 10215  9575  9287  9314  9126  9190  9205  9023  8146
    6967   1 2021-05-01 10665 10569  9916  9637  9671  9483  9550  9539  9390  8230
    6968   1 2021-05-01 10907 10500  9849  9564  9612  9427  9469  9425  9295  8568
    6969   1 2021-05-01 10941 10584  9929  9656  9712  9525  9567  9539  9395  8659
    6970   1 2021-05-01 11013 10513  9859  9581  9638  9461  9501  9478  9330  8763
    6971   1 2021-05-01 10833 10416  9790  9522  9591  9430  9470  9404  9316  8677
    6972   1 2021-05-01 10629 10123  9501  9239  9306  9162  9215  9149  9065  8488
    6973   1 2021-05-01 10509  9947  9338  9076  9145  9005  9062  9003  8916  8381
    6974   1 2021-05-01 10216  9654  9040  8769  8831  8699  8764  8732  8617  8112
    6975   1 2021-05-01 10016  9589  9000  8744  8822  8701  8759  8683  8612  7980
    6976   1 2021-05-01  9856  9307  8722  8468  8540  8428  8497  8425  8344  7845
    6977   1 2021-05-01  9690  9100  8541  8298  8379  8281  8350  8258  8211  7699
    6978   1 2021-05-01  9533  8778  8249  8011  8087  8003  8087  7958  7957  7557
    6979   1 2021-05-01  9212  8778  8249  8011  8087  8003  8087  7958  7957  7231
    6980   1 2021-05-01  8965  8454  7954  7741  7821  7752  7849  7698  7739  6988
    6981   1 2021-05-01  9000  8592  8045  7798  7862  7777  7847  7775  7726  6998
    6982   1 2021-05-01  9006  8544  8018  7794  7854  7787  7867  7767  7753  7018
    6983   1 2021-05-01  8772  8406  7897  7678  7743  7685  7773  7665  7664  6774
    6984   1 2021-05-01  8548  8132  7648  7424  7494  7447  7562  7441  7447  6519
    6985   1 2021-05-01  8556  8300  7768  7520  7577  7506  7604  7548  7490  6495
    6986   1 2021-05-01  8761  8272  7741  7493  7542  7465  7556  7466  7447  6626
    6987   1 2021-05-01  8583  8102  7601  7371  7420  7361  7463  7328  7348  6401
    6988   1 2021-05-01  8366  7644  7209  7016  7058  7050  7177  6976  7089  6064
    6989   1 2021-05-01  7476  7232  6789  6585  6609  6612  6755  6573  6669  5139
    6990   1 2021-05-01  7526  7929  7393  7140  7159  7103  7203  7116  7085  5101
    6991   1 2021-05-01  8720  8392  7838  7575  7607  7516  7597  7535  7464  6456
    6992   1 2021-05-01  8965  8554  7986  7716  7755  7657  7735  7677  7604  6736
    6993   1 2021-05-01 12266 11943 11172 10842 10826 10523 10563 10414 10330  9014
    6994   1 2021-05-01 12512 11936 11163 10836 10835 10533 10570 10399 10332  9315
    6995   1 2021-05-01 12502 11957 11179 10858 10865 10575 10602 10399 10368  9539
    6996   1 2021-05-01 12609 12093 11272 10901 10908 10604 10615 10497 10374  9629
    6997   1 2021-05-01 12625 12201 11382 11020 11009 10690 10707 10599 10460  9607
    6998   1 2021-05-01 12797 12360 11550 11200 11184 10868 10894 10762 10642  9538
    6999   1 2021-05-01 12924 12286 11467 11101 11078 10759 10782 10676 10525  9677
    7000   1 2021-05-01 12857 12519 11689 11299 11287 10945 10946 10896 10669  9700
    7001   1 2021-05-01 13034 12664 11827 11432 11451 11110 11104 11037 10827  9986
    7002   1 2021-05-01 13091 12597 11762 11365 11396 11058 11067 10958 10804 10099
    7003   1 2021-05-01 13129 12702 11848 11462 11480 11134 11140 11056 10875 10082
    7004   1 2021-05-01 13285 12766 11945 11588 11605 11279 11280 11134 11019 10218
    7005   1 2021-05-01 13195 12479 11676 11323 11368 11056 11064 10858 10820 10161
    7006   1 2021-05-01 13014 12432 11635 11294 11335 11032 11033 10816 10787  9910
    7007   1 2021-05-01 12953 12204 11391 11044 11065 10764 10782 10589 10527  9868
    7008   1 2021-05-01 12759 12204 11391 11044 11065 10764 10782 10589 10527  9605
    7009   1 2021-05-01 12740 12257 11425 11058 11038 10747 10762 10661 10511  9545
    7010   1 2021-05-01 12716 12281 11439 11062 11050 10761 10747 10694 10513  9577
    7011   1 2021-05-01 12865 12430 11606 11243 11251 10949 10946 10874 10686  9908
    7012   1 2021-05-01 12833 12334 11521 11158 11169 10860 10866 10802 10613  9925
    7013   1 2021-05-01 12647 12120 11350 11022 11044 10755 10786 10620 10551  9755
    7014   1 2021-05-01 12606 12009 11235 10880 10900 10615 10646 10526 10417  9597
    7015   1 2021-05-01 12536 12005 11255 10923 10942 10671 10705 10570 10492  9575
    7016   1 2021-05-01 12489 11953 11191 10842 10867 10593 10618 10518 10412  9636
    7017   1 2021-05-01 12375 11731 11000 10670 10695 10421 10462 10350 10248  9574
    7018   1 2021-05-01 12155 11541 10807 10470 10489 10235 10268 10172 10059  9354
    7019   1 2021-05-01 11864 11343 10611 10274 10298 10031 10073  9977  9862  9154
    7020   1 2021-05-01 11828 11314 10588 10259 10290 10031 10064  9966  9853  9181
    7021   1 2021-05-01 11767 11314 10588 10259 10290 10031 10064  9966  9853  9180
    7022   1 2021-05-01 11762 11320 10581 10238 10269 10003 10029  9959  9804  9234
    7023   1 2021-05-01 11693 11205 10480 10150 10181  9925  9955  9879  9732  9203
    7024   1 2021-05-01 11350 10932 10242  9951  9981  9740  9787  9679  9581  8784
    7025   1 2021-05-01 11308 10846 10146  9852  9884  9634  9679  9592  9484  8752
    7026   1 2021-05-01 11309 10832 10136  9834  9869  9629  9661  9588  9451  8779
    7027   1 2021-05-01 11320 10874 10172  9873  9906  9670  9701  9626  9508  8795
    7028   1 2021-05-01 11285 10794 10097  9787  9832  9600  9629  9527  9436  8789
    7029   1 2021-05-01 11219 10643  9945  9635  9677  9448  9481  9364  9285  8752
    7030   1 2021-05-01 11115 10607  9914  9606  9648  9424  9455  9343  9251  8634
    7031   1 2021-05-01 11019 10526  9840  9536  9573  9350  9381  9273  9188  8539
    7032   1 2021-05-01 10966 10466  9775  9463  9500  9279  9313  9217  9115  8486
    7033   1 2021-05-01 10948 10382  9692  9385  9420  9204  9243  9118  9054  8443
    7034   1 2021-05-01 10840 10334  9633  9326  9361  9150  9186  9064  8991  8287
    7035   1 2021-05-01 10818 10327  9624  9305  9333  9112  9144  9057  8958  8246
    7036   1 2021-05-01 10823 10371  9660  9344  9370  9150  9180  9098  8983  8200
    7037   1 2021-05-01 10837 10413  9687  9364  9382  9156  9189  9113  8976  8209
    7038   1 2021-05-01 10940 10377  9651  9311  9324  9097  9120  9079  8908  8285
    7039   1 2021-05-01 10929 10447  9740  9430  9455  9244  9272  9183  9083  8273
    7040   1 2021-05-01 10791 10213  9549  9258  9286  9097  9147  9015  8975  8093
    7041   1 2021-05-01 10668 10195  9506  9201  9224  9031  9074  8981  8889  7984
    7042   1 2021-05-01 10523 10144  9482  9199  9221  9038  9091  8982  8928  7845
    7043   1 2021-05-01 10434  9847  9204  8926  8946  8787  8845  8735  8692  7758
    7044   1 2021-05-01 10242  9576  8947  8673  8690  8539  8608  8475  8456  7572
    7045   1 2021-05-01  9794  9262  8627  8341  8344  8198  8275  8162  8126  7083
    7046   1 2021-05-01  9548  9174  8531  8233  8232  8083  8154  8063  8005  6770
    7047   1 2021-05-01  9554  9017  8383  8087  8070  7929  8011  7952  7858  6788
    7048   1 2021-05-01  9775  9472  8817  8515  8520  8355  8424  8302  8256  7024
    7049   1 2021-05-01 10067  9472  8817  8515  8520  8355  8424  8302  8256  7407
    7050   1 2021-05-01 10117  9559  8902  8598  8623  8442  8502  8407  8337  7525
    7051   1 2021-05-01 10112  9693  9043  8739  8770  8584  8646  8557  8473  7517
    7052   1 2021-05-01  9969  9597  8986  8715  8743  8581  8651  8532  8497  7400
    7053   1 2021-05-01 10307  9953  9307  9017  9032  8858  8923  8932  8746  7847
    7054   1 2021-05-01 10424  9985  9343  9048  9072  8894  8942  8909  8763  7989
    7055   1 2021-05-01 10424 10069  9428  9126  9158  8983  9032  9020  8863  8014
    7056   1 2021-05-01 10498 10146  9489  9182  9214  9034  9063  9054  8891  8071
    7057   1 2021-05-01 10766 10334  9680  9397  9448  9287  9328  9298  9160  8475
    7058   1 2021-05-01 10665 10103  9472  9195  9251  9097  9154  9119  8994  8413
    7059   1 2021-05-01 10690  9886  9285  9014  9070  8935  8994  8972  8848  8508
    7060   1 2021-05-01 10065  9886  9285  9014  9070  8935  8994  8972  8848  7922
    7061   1 2021-05-01  9945  9708  9108  8843  8901  8777  8839  8822  8699  7821
    7062   1 2021-05-01  9679  9297  8705  8430  8479  8363  8434  8433  8294  7544
    7063   1 2021-05-01  9811  9422  8826  8560  8629  8503  8569  8535  8420  7749
    7064   1 2021-05-01  9640  9307  8722  8468  8540  8428  8497  8425  8344  7631
    7065   1 2021-05-01  9489  9111  8542  8294  8365  8270  8338  8261  8202  7492
    7066   1 2021-05-01  9309  8845  8301  8056  8127  8051  8128  8030  8004  7352
    7067   1 2021-05-01  9194  8632  8089  7846  7911  7840  7921  7847  7793  7239
    7068   1 2021-05-01  8812  8481  7930  7683  7744  7688  7770  7698  7658  6846
    7069   1 2021-05-01  8654  8333  7792  7542  7589  7529  7613  7570  7489  6683
    7070   1 2021-05-01  8801  8419  7882  7646  7700  7635  7723  7646  7605  6812
    7071   1 2021-05-01  8805  8338  7799  7562  7617  7552  7646  7564  7525  6775
    7072   1 2021-05-01  8769  8220  7722  7500  7560  7513  7609  7467  7496  6725
    7073   1 2021-05-01  8652  8185  7653  7405  7453  7391  7482  7387  7375  6495
    7074   1 2021-05-01  8629  8171  7649  7408  7452  7391  7483  7367  7377  6439
    7075   1 2021-05-01  8019  7644  7209  7016  7058  7050  7177  6976  7089  5731
    7076   1 2021-05-01  8129  7871  7346  7091  7111  7058  7160  7058  7033  5727
    7077   1 2021-05-01  8457  8150  7618  7356  7386  7318  7405  7299  7280  6131
    7078   1 2021-05-01 11822 11662 10897 10556 10525 10233 10269 10136 10028  8359
    7079   1 2021-05-01 12178 11682 10914 10563 10530 10229 10268 10149 10033  8739
    7080   1 2021-05-01 12424 11963 11179 10854 10830 10534 10572 10418 10327  9062
    7081   1 2021-05-01 12483 12029 11243 10906 10876 10570 10592 10475 10337  9122
    7082   1 2021-05-01 12624 12104 11293 10950 10950 10657 10690 10511 10454  9398
    7083   1 2021-05-01 12591 12008 11201 10821 10791 10496 10516 10411 10258  9375
    7084   1 2021-05-01 12676 12079 11271 10912 10907 10597 10619 10464 10362  9429
    7085   1 2021-05-01 12691 12226 11393 11008 10995 10670 10682 10592 10416  9651
    7086   1 2021-05-01 12742 12302 11484 11098 11104 10785 10782 10660 10529  9746
    7087   1 2021-05-01 12889 12504 11687 11290 11300 10967 10958 10869 10694  9915
    7088   1 2021-05-01 13074 12504 11687 11290 11300 10967 10958 10869 10694 10081
    7089   1 2021-05-01 13200 12678 11840 11453 11472 11128 11127 11028 10856 10114
    7090   1 2021-05-01 13383 12846 12021 11643 11663 11315 11304 11214 11031 10223
    7091   1 2021-05-01 13236 12676 11867 11502 11527 11205 11211 11048 10955 10111
    7092   1 2021-05-01 13171 12606 11778 11401 11425 11106 11083 10969 10823 10166
    7093   1 2021-05-01 13126 12322 11542 11210 11256 10953 10960 10703 10716 10119
    7094   1 2021-05-01 12864 12364 11589 11269 11317 11015 11043 10759 10803  9790
    7095   1 2021-05-01 12779 12206 11394 11060 11085 10808 10826 10607 10592  9683
    7096   1 2021-05-01 12799 12332 11522 11179 11226 10930 10941 10750 10702  9872
    7097   1 2021-05-01 12782 12172 11359 10998 11028 10731 10735 10606 10502  9876
    7098   1 2021-05-01 12561 12128 11328 10966 10972 10668 10671 10615 10433  9632
    7099   1 2021-05-01 12363 11824 11065 10720 10705 10420 10450 10364 10220  9350
    7100   1 2021-05-01 12353 11914 11175 10839 10836 10557 10598 10483 10365  9097
    7101   1 2021-05-01 12472 11914 11175 10839 10836 10557 10598 10483 10365  9359
    7102   1 2021-05-01 12254 11897 11159 10835 10857 10598 10640 10494 10429  9273
    7103   1 2021-05-01 12069 11643 10917 10584 10602 10335 10392 10258 10180  9130
    7104   1 2021-05-01 11949 11456 10729 10403 10421 10164 10203 10081  9990  9110
    7105   1 2021-05-01 11805 11235 10504 10157 10168  9918  9950  9856  9732  9061
    7106   1 2021-05-01 11591 11297 10560 10232 10251  9988 10026  9949  9820  8801
    7107   1 2021-05-01 11715 11211 10483 10147 10171  9908  9940  9871  9726  9080
    7108   1 2021-05-01 11659 11137 10413 10085 10116  9868  9902  9802  9681  9108
    7109   1 2021-05-01 11579 10917 10199  9882  9912  9676  9707  9608  9511  9045
    7110   1 2021-05-01 11394 10799 10095  9788  9807  9575  9613  9541  9416  8865
    7111   1 2021-05-01 11237 10795 10092  9776  9804  9566  9601  9528  9401  8704
    7112   1 2021-05-01 11211 10738 10036  9722  9752  9515  9545  9476  9345  8710
    7113   1 2021-05-01 11191 10766 10070  9759  9798  9565  9597  9503  9401  8692
    7114   1 2021-05-01 11254 10752 10050  9739  9775  9544  9570  9475  9370  8747
    7115   1 2021-05-01 11245 10715 10020  9706  9752  9523  9549  9427  9348  8790
    7116   1 2021-05-01 11149 10689  9990  9681  9721  9485  9509  9412  9310  8700
    7117   1 2021-05-01 11060 10557  9867  9568  9606  9386  9418  9317  9224  8609
    7118   1 2021-05-01 10923 10431  9743  9437  9471  9248  9287  9173  9095  8459
    7119   1 2021-05-01 10815 10379  9688  9369  9402  9179  9212  9127  9011  8284
    7120   1 2021-05-01 10732 10269  9580  9265  9296  9075  9112  9014  8908  8164
    7121   1 2021-05-01 10734 10282  9584  9278  9299  9088  9126  9020  8929  8123
    7122   1 2021-05-01 10711 10101  9396  9085  9096  8888  8922  8820  8725  8069
    7123   1 2021-05-01 10738 10127  9399  9059  9071  8847  8878  8814  8668  8090
    7124   1 2021-05-01 10614 10376  9659  9326  9343  9118  9143  9103  8948  7929
    7125   1 2021-05-01 10752 10199  9517  9216  9238  9038  9079  8989  8903  8061
    7126   1 2021-05-01 10613 10097  9418  9120  9138  8953  9001  8903  8824  7886
    7127   1 2021-05-01 10508  9855  9198  8910  8924  8756  8812  8708  8643  7760
    7128   1 2021-05-01 10090 10003  9342  9055  9072  8892  8940  8860  8767  7334
    7129   1 2021-05-01  9942  9618  8982  8703  8716  8558  8625  8531  8473  7168
    7130   1 2021-05-01  9818  9444  8824  8546  8557  8414  8486  8366  8333  7039
    7131   1 2021-05-01  9432  8965  8364  8091  8091  7963  8047  7927  7898  6599
    7132   1 2021-05-01  8604  8705  8097  7810  7792  7679  7777  7702  7643  5737
    7133   1 2021-05-01  8998  8469  7871  7589  7564  7457  7563  7480  7422  6140
    7134   1 2021-05-01  9628  9243  8593  8285  8278  8124  8196  8130  8028  6842
    7135   1 2021-05-01  9812  9498  8842  8546  8553  8385  8444  8375  8276  7117
    7136   1 2021-05-01 10158  9702  9055  8757  8778  8608  8661  8559  8495  7600
    7137   1 2021-05-01 10178  9755  9099  8795  8821  8641  8688  8604  8522  7646
    7138   1 2021-05-01 10236  9845  9184  8884  8919  8741  8794  8699  8629  7716
    7139   1 2021-05-01 10323  9908  9246  8942  8975  8796  8840  8777  8664  7853
    7140   1 2021-05-01 10366  9987  9319  9001  9034  8847  8883  8827  8697  7921
    7141   1 2021-05-01 10539 10069  9405  9104  9138  8959  8990  8920  8814  8160
    7142   1 2021-05-01 10527 10069  9405  9104  9138  8959  8990  8920  8814  8166
         1610 2190
    1    5682 3917
    2    5802 3981
    3    5754 3937
    4    5726 4054
    5    5831 4097
    6    5773 3990
    7    5699 3909
    8    5743 3920
    9    5944 4131
    10   6108 4311
    11   6031 4203
    12   5678 3895
    13   5536 3767
    14   5632 3843
    15   5901 4079
    16   5949 4144
    17   6258 4607
    18   6211 4457
    19   6031 4203
    20   5844 4056
    21   5648 3885
    22   5755 3993
    23   5838 4030
    24   5880 4052
    25   5767 3925
    26   5944 4008
    27   6306 4610
    28   6117 4451
    29   6017 4171
    30   6015 4131
    31   5883 4042
    32   6005 4154
    33   5893 4081
    34   5893 4081
    35   5718 3890
    36   5897 4036
    37   6091 4071
    38   6127 4087
    39   6087 4678
    40   6022 4406
    41   6022 4406
    42   6010 4276
    43   5922 4054
    44   5916 4043
    45   6062 4182
    46   6114 4252
    47   6055 4189
    48   5945 4087
    49   6027 4087
    50   6068 4076
    51   6058 4049
    52   5848 3934
    53   5614 3790
    54   6201 4970
    55   5811 4436
    56   5659 4161
    57   5646 4090
    58   5844 4071
    59   6076 4211
    60   6102 4207
    61   5999 4080
    62   5957 4047
    63   5810 3932
    64   5988 4070
    65   6027 4087
    66   6048 4099
    67   6065 4072
    68   5940 4009
    69   5736 3877
    70   5738 3901
    71   5830 4513
    72   6132 4748
    73   5928 4423
    74   5933 4283
    75   5813 4119
    76   6064 4224
    77   6153 4246
    78   6042 4115
    79   5808 3903
    80   5810 3932
    81   5905 4013
    82   6082 4159
    83   6081 4170
    84   5946 4011
    85   5697 3834
    86   5717 3842
    87   5756 3894
    88   5773 3929
    89   5728 4499
    90   6023 4732
    91   6447 5087
    92   6164 4652
    93   6094 4480
    94   6018 4283
    95   5984 4096
    96   6132 4154
    97   6004 4063
    98   5914 3960
    99   5838 3964
    100  5986 4106
    101  6005 4108
    102  6005 4108
    103  5929 4029
    104  5754 3904
    105  5639 3808
    106  5688 3845
    107  5794 3945
    108  5838 4429
    109  6580 5154
    110  6605 5133
    111  6375 4838
    112  6261 4566
    113  6030 4202
    114  5984 4096
    115  6014 4059
    116  6096 4094
    117  5895 3930
    118  5874 3976
    119  6036 4145
    120  5967 4099
    121  5711 3870
    122  5654 3835
    123  5721 3888
    124  5728 3887
    125  5994 4690
    126  5994 4690
    127  6433 4952
    128  6558 5051
    129  6577 5068
    130  6261 4566
    131  6227 4336
    132  6149 4233
    133  6036 4071
    134  5895 3972
    135  5805 3904
    136  5802 3897
    137  5845 3957
    138  5972 4125
    139  5714 3888
    140  5749 3906
    141  5682 3869
    142  5662 3842
    143  6555 5286
    144  6435 5200
    145  6448 5195
    146  6275 4791
    147  6502 5007
    148  6405 4862
    149  6295 4464
    150  6369 4394
    151  6120 4156
    152  5814 3847
    153  5676 3807
    154  5877 3970
    155  5573 3760
    156  5781 3933
    157  5751 3913
    158  5702 3837
    159  5813 3905
    160  6552 5254
    161  6592 5246
    162  6556 5211
    163  6377 5093
    164  6333 4854
    165  6398 4869
    166  6353 4614
    167  6394 4437
    168  5584 3789
    169  5715 3888
    170  5833 3972
    171  5780 3957
    172  5776 3937
    173  5791 3931
    174  5816 3917
    175  5756 3876
    176  5750 3929
    177  5767 3946
    178  6399 5071
    179  6532 5252
    180  6508 5128
    181  6556 5211
    182  6456 5129
    183  6316 4880
    184  6305 4672
    185  6454 4555
    186  5597 3839
    187  5778 3953
    188  5902 4064
    189  5902 4048
    190  5911 4065
    191  5879 4047
    192  5879 4047
    193  5759 3922
    194  5791 3931
    195  5831 3930
    196  5777 3904
    197  5804 3955
    198  5791 3980
    199  5800 4002
    200  6479 5173
    201  6317 4945
    202  6132 4739
    203  6392 5055
    204  6257 4926
    205  6295 4953
    206  6104 4625
    207  6384 4587
    208  5813 3998
    209  5833 3999
    210  5842 3998
    211  5877 4033
    212  5930 4082
    213  5839 3994
    214  5808 3969
    215  5824 3957
    216  5860 3975
    217  5857 3989
    218  5866 3996
    219  5818 3974
    220  5818 3974
    221  5784 4011
    222  5778 3985
    223  6345 5039
    224  5928 4553
    225  5949 4616
    226  6269 4949
    227  6208 4861
    228  5985 4600
    229  5831 4301
    230  5742 4056
    231  5045 3419
    232  5733 3904
    233  5807 3996
    234  5792 3972
    235  5891 4043
    236  5807 3969
    237  5781 3927
    238  5781 3927
    239  5823 3957
    240  5840 3950
    241  5885 3997
    242  5863 3984
    243  5845 3978
    244  5836 4000
    245  5793 3976
    246  5915 4073
    247  5945 4107
    248  6014 4165
    249  6367 5028
    250  6201 4930
    251  6134 4886
    252  6322 5095
    253  6087 4754
    254  5866 4456
    255  5499 4031
    256  5499 4031
    257  5403 3804
    258  5297 3721
    259  5403 3678
    260  5860 4016
    261  5928 4108
    262  5835 4009
    263  5732 3915
    264  5751 3908
    265  5713 3902
    266  5681 3839
    267  5603 3757
    268  5801 3931
    269  5746 3884
    270  5813 3953
    271  5699 3886
    272  5769 3948
    273  5915 4073
    274  5949 4122
    275  5979 4150
    276  5903 4121
    277  5760 4012
    278  6305 5057
    279  6305 5057
    280  6352 5184
    281  6380 5196
    282  6085 4766
    283  6008 4594
    284  5530 4061
    285  5593 4025
    286  5557 3964
    287  5541 3847
    288  5723 3841
    289  5926 3929
    290  6003 4123
    291  5757 3942
    292  5635 3819
    293  5740 3913
    294  5688 3874
    295  5626 3814
    296  5510 3713
    297  5504 3683
    298  5628 3819
    299  5706 3878
    300  5748 3939
    301  5705 3913
    302  5909 4149
    303  5812 4073
    304  5523 3867
    305  6594 5442
    306  6514 5368
    307  6303 5074
    308  6205 4811
    309  6202 4686
    310  6118 4490
    311  5912 4293
    312  5912 4293
    313  5691 4021
    314  5693 3896
    315  5885 3893
    316  5794 3811
    317  5766 3921
    318  5620 3827
    319  5780 3971
    320  5848 4026
    321  5716 3900
    322  5696 3891
    323  5690 3923
    324  5783 4004
    325  5820 4000
    326  5788 3999
    327  5692 3938
    328  5740 4003
    329  5703 3988
    330  5486 3850
    331  6214 4827
    332  6223 4874
    333  6380 5149
    334  6236 5018
    335  6473 5332
    336  6327 5171
    337  6302 4951
    338  6202 4686
    339  6151 4511
    340  6161 4464
    341  5990 4231
    342  5805 4012
    343  5860 3966
    344  5798 3851
    345  5683 3757
    346  5612 3733
    347  5736 3941
    348  5679 3891
    349  5681 3871
    350  5681 3871
    351  5761 3978
    352  5783 4004
    353  5818 4012
    354  5780 3975
    355  5681 3940
    356  5459 3808
    357  5453 3837
    358  5214 3655
    359  4836 3306
    360  7116 5663
    361  6790 5359
    362  6224 4883
    363  6028 4719
    364  6080 4769
    365  5876 4603
    366  6162 4976
    367  6272 5065
    368  6148 4838
    369  6112 4599
    370  5890 4349
    371  5936 4224
    372  5656 3982
    373  5829 4066
    374  5623 3821
    375  5764 3850
    376  5562 3707
    377  5559 3670
    378  5516 3676
    379  5561 3793
    380  5646 3873
    381  5673 3910
    382  5778 3982
    383  5726 3937
    384  5690 3937
    385  5523 3855
    386  5290 3657
    387  5290 3657
    388  5436 3828
    389  5308 3727
    390  4938 3418
    391  4780 3255
    392  4576 3063
    393  6992 5314
    394  6797 5236
    395  6686 5272
    396  6519 5377
    397  6514 5448
    398  5920 4781
    399  5883 4688
    400  5881 4744
    401  6189 5055
    402  5992 4843
    403  6012 4574
    404  5870 4424
    405  5717 4166
    406  5382 3857
    407  5133 3483
    408  5378 3656
    409  5551 3725
    410  5474 3671
    411  5412 3600
    412  5521 3795
    413  5521 3795
    414  5677 3908
    415  5681 3911
    416  5640 3894
    417  5463 3797
    418  5242 3642
    419  5221 3646
    420  5250 3668
    421  4914 3414
    422  4657 3136
    423  4603 3081
    424  4473 2972
    425  4435 2910
    426  6958 5252
    427  6767 5130
    428  6715 5207
    429  6715 5207
    430  6815 5634
    431  6514 5448
    432  6304 5246
    433  5734 4594
    434  5700 4620
    435  5663 4522
    436  5699 4502
    437  5713 4381
    438  5490 4184
    439  5211 3799
    440  4832 3422
    441  5177 3580
    442  5177 3580
    443  5439 3681
    444  5474 3671
    445  5542 3718
    446  5578 3835
    447  5598 3883
    448  5507 3846
    449  5308 3729
    450  5193 3621
    451  5156 3614
    452  4976 3514
    453  4914 3414
    454  4491 3034
    455  4518 3004
    456  4547 3039
    457  4338 2862
    458  4275 2814
    459  4314 2841
    460  7154 5612
    461  7148 5511
    462  7043 5319
    463  6903 5254
    464  6764 5278
    465  6890 5587
    466  6859 5623
    467  6436 5315
    468  5795 4672
    469  5525 4333
    470  5909 4810
    471  5853 4746
    472  5853 4746
    473  5675 4431
    474  5608 4371
    475  5202 3891
    476  5132 3781
    477  5049 3544
    478  5317 3620
    479  5466 3689
    480  5509 3837
    481  5254 3676
    482  5044 3491
    483  4697 3263
    484  4662 3164
    485  4625 3121
    486  4490 3012
    487  4283 2853
    488  4192 2792
    489  4117 2728
    490  4117 2728
    491  4185 2773
    492  7073 5419
    493  7002 5350
    494  6951 5284
    495  6910 5443
    496  6925 5552
    497  6843 5519
    498  6843 5519
    499  6553 5311
    500  6286 5132
    501  5510 4352
    502  6010 4899
    503  6054 4987
    504  5905 4765
    505  5715 4495
    506  5863 4563
    507  5518 4146
    508  5092 3644
    509  5293 3649
    510  5628 3819
    511  4963 3456
    512  4605 3102
    513  4666 3155
    514  4530 3048
    515  4394 2958
    516  4259 2880
    517  4016 2664
    518  3915 2597
    519  4159 2796
    520  4433 2994
    521  4450 3016
    522  4033 2894
    523  3963 2912
    524  4121 3089
    525  6945 5415
    526  6987 5383
    527  7002 5350
    528  6930 5284
    529  6899 5336
    530  6834 5380
    531  6759 5389
    532  6639 5402
    533  6507 5332
    534  6445 5268
    535  6380 5307
    536  6390 5341
    537  6160 5052
    538  6105 4879
    539  5943 4732
    540  5863 4563
    541  5812 4387
    542  5454 4000
    543  5468 3855
    544  5643 3835
    545  5788 3921
    546  5755 3886
    547  4478 3001
    548  4359 2962
    549  4131 2808
    550  3916 2622
    551  4021 2714
    552  4021 2714
    553  4137 2785
    554  4067 2733
    555  4390 3005
    556  3970 2855
    557  3923 2856
    558  3916 2909
    559  3925 2968
    560  3953 2995
    561  6834 5269
    562  6825 5232
    563  6806 5194
    564  6727 5215
    565  6633 5211
    566  6700 5444
    567  6686 5556
    568  6601 5473
    569  6408 5283
    570  6434 5266
    571  6354 5259
    572  6168 5009
    573  6105 4879
    574  6230 4852
    575  6112 4693
    576  5792 4294
    577  5509 3875
    578  5682 3894
    579  5725 3848
    580  5639 3741
    581  5847 3916
    582  5847 3916
    583  5829 3932
    584  4131 2808
    585  4060 2752
    586  4035 2771
    587  4016 2769
    588  3897 2632
    589  3803 2579
    590  3734 2500
    591  3539 2346
    592  3539 2360
    593  3623 2558
    594  3740 2636
    595  3923 2856
    596  3831 2794
    597  3993 3029
    598  4022 3037
    599  4070 3105
    600  6471 4896
    601  6651 5108
    602  6400 4859
    603  6631 5027
    604  6529 4938
    605  6789 5268
    606  6626 5115
    607  6585 5232
    608  6573 5313
    609  6471 5391
    610  6055 4888
    611  6033 4783
    612  5984 4832
    613  5738 4443
    614  6075 4734
    615  6089 4685
    616  6220 4776
    617  5841 4317
    618  5533 3912
    619  5664 3875
    620  5596 3735
    621  5447 3589
    622  5616 3725
    623  5585 3706
    624  5656 3801
    625  4240 3038
    626  4107 2882
    627  4000 2744
    628  3902 2659
    629  3674 2486
    630  3744 2527
    631  3724 2533
    632  3930 2751
    633  3503 2410
    634  3436 2430
    635  3517 2452
    636  3555 2524
    637  3734 2717
    638  3994 3026
    639  3985 3000
    640  4089 3108
    641  4114 3083
    642  4114 3083
    643  4271 3243
    644  4320 3274
    645  5903 4276
    646  5571 3951
    647  6176 4545
    648  5997 4411
    649  6322 4842
    650  6238 4715
    651  6466 4988
    652  6404 5028
    653  6346 5028
    654  5910 4623
    655  5741 4436
    656  6055 4888
    657  5775 4512
    658  5843 4599
    659  5611 4336
    660  5899 4598
    661  5888 4577
    662  5790 4291
    663  5576 3957
    664  5551 3770
    665  5661 3797
    666  5560 3714
    667  5722 3817
    668  5582 3742
    669  5559 3714
    670  5559 3714
    671  5605 3794
    672  4287 3043
    673  4007 2737
    674  4000 2697
    675  4043 2757
    676  4335 2991
    677  4298 3013
    678  3928 2768
    679  3894 2730
    680  3430 2350
    681  3484 2416
    682  3517 2550
    683  3604 2612
    684  3604 2612
    685  3697 2705
    686  4020 3041
    687  4234 3265
    688  4238 3226
    689  4114 3089
    690  4173 3102
    691  4316 3241
    692  5016 3501
    693  5467 4031
    694  5449 3925
    695  5768 4325
    696  5933 4453
    697  5933 4453
    698  6166 4631
    699  6218 4772
    700  6182 4860
    701  5910 4623
    702  5528 4216
    703  5690 4376
    704  5880 4636
    705  5744 4500
    706  5740 4524
    707  5728 4431
    708  5775 4425
    709  5755 4159
    710  5755 4159
    711  5583 3710
    712  5692 3811
    713  5722 3817
    714  5727 3851
    715  5637 3819
    716  5798 3955
    717  5800 3983
    718  4234 2966
    719  4255 2896
    720  4335 2991
    721  4595 3224
    722  4342 3077
    723  4285 3003
    724  3952 2789
    725  3957 2790
    726  3390 2376
    727  3469 2487
    728  3675 2680
    729  3677 2691
    730  3852 2826
    731  4054 3085
    732  4141 3135
    733  4207 3146
    734  4334 3276
    735  4361 3287
    736  4563 3461
    737  4361 3271
    738  4477 3404
    739  5064 3796
    740  5304 3734
    741  5501 4054
    742  6104 4662
    743  5916 4547
    744  6319 4980
    745  6041 4599
    746  6099 4670
    747  5986 4573
    748  5916 4538
    749  5703 4347
    750  5773 4456
    751  5816 4534
    752  5929 4715
    753  5929 4715
    754  5854 4605
    755  5891 4661
    756  5695 4457
    757  5776 4475
    758  5601 3752
    759  5614 3706
    760  5739 3838
    761  5719 3847
    762  5669 3839
    763  5681 3843
    764  5720 3909
    765  5747 3905
    766  4843 3398
    767  4860 3410
    768  4568 3204
    769  4689 3307
    770  4375 3097
    771  4375 3097
    772  3929 2801
    773  4056 2940
    774  3776 2696
    775  3789 2763
    776  3926 2862
    777  3885 2830
    778  4093 3018
    779  4027 2989
    780  4212 3218
    781  4244 3198
    782  4500 3452
    783  4637 3533
    784  4563 3461
    785  4678 3558
    786  4690 3596
    787  4748 3683
    788  4638 3598
    789  4795 3582
    790  5422 4089
    791  5903 4592
    792  6160 4672
    793  6448 4946
    794  6770 5218
    795  6655 5202
    796  6592 5153
    797  6289 4833
    798  6283 4960
    799  6283 4960
    800  5962 4691
    801  5980 4754
    802  5868 4601
    803  5993 4788
    804  5986 4784
    805  5812 4548
    806  5905 4640
    807  5912 4602
    808  5797 4379
    809  5738 4176
    810  5774 4049
    811  5741 3932
    812  5554 3663
    813  5694 3803
    814  5744 3871
    815  5742 3900
    816  5707 3875
    817  5733 3909
    818  5677 3874
    819  5580 3815
    820  5122 3590
    821  4905 3456
    822  5034 3573
    823  4808 3421
    824  4928 3542
    825  4589 3343
    826  4676 3391
    827  4572 3379
    828  4383 3217
    829  4575 3389
    830  3988 2922
    831  4194 3096
    832  4130 3039
    833  4367 3261
    834  4312 3261
    835  4461 3373
    836  4697 3622
    837  4774 3671
    838  4906 3738
    839  4842 3675
    840  4853 3679
    841  4807 3683
    842  4855 3729
    843  4738 3663
    844  4778 3637
    845  4862 3750
    846  4624 3602
    847  4730 3732
    848  4532 3551
    849  4491 3441
    850  4817 3627
    851  5663 4380
    852  6004 4745
    853  6212 4858
    854  6291 4962
    855  6516 5006
    856  6861 5315
    857  6673 5144
    858  6910 5338
    859  6938 5347
    860  6592 5153
    861  6543 5123
    862  6355 4965
    863  6477 5167
    864  6419 5236
    865  6430 5238
    866  6173 4978
    867  5849 4627
    868  5509 4231
    869  5261 3901
    870  5753 4448
    871  5859 4548
    872  5811 4295
    873  5738 4176
    874  5688 3985
    875  5737 3880
    876  5728 3880
    877  5736 3872
    878  5833 3946
    879  5803 3946
    880  5779 3951
    881  5651 3839
    882  5571 3788
    883  5602 3831
    884  5602 3831
    885  5438 3906
    886  5233 3789
    887  5209 3753
    888  4955 3587
    889  4676 3391
    890  4781 3548
    891  4936 3656
    892  4647 3453
    893  4903 3645
    894  4645 3476
    895  4593 3425
    896  4686 3499
    897  4483 3355
    898  4461 3373
    899  4686 3567
    900  4904 3779
    901  4959 3761
    902  4934 3737
    903  5020 3832
    904  4960 3785
    905  5042 3893
    906  4941 3847
    907  5371 4304
    908  5208 4090
    909  5265 4143
    910  5081 3962
    911  5040 3922
    912  5040 3922
    913  4846 3810
    914  4969 3943
    915  4792 3777
    916  4828 3743
    917  4890 3840
    918  4649 3530
    919  4649 3530
    920  5422 4165
    921  5793 4525
    922  6240 4963
    923  6354 5063
    924  6350 5092
    925  5959 4732
    926  6671 5132
    927  6965 5387
    928  7011 5445
    929  7030 5434
    930  7024 5446
    931  6795 5353
    932  6711 5315
    933  6430 5096
    934  6331 4984
    935  6540 5268
    936  6436 5144
    937  6165 4926
    938  5772 4518
    939  5417 4183
    940  5187 3937
    941  5551 4137
    942  5795 4328
    943  5913 4254
    944  5866 4131
    945  5742 3995
    946  5553 3724
    947  5647 3797
    948  5766 3906
    949  5826 3969
    950  5828 3951
    951  5810 3969
    952  5661 3878
    953  5610 3850
    954  5561 3830
    955  5537 3807
    956  5571 3875
    957  5209 3753
    958  5268 3802
    959  5097 3705
    960  5324 3960
    961  5437 4053
    962  5236 3899
    963  5409 4031
    964  5340 4031
    965  5340 4031
    966  5140 3888
    967  5151 3863
    968  4860 3646
    969  4868 3695
    970  4833 3703
    971  5139 4004
    972  5037 3850
    973  5265 4075
    974  5259 4064
    975  5437 4245
    976  5427 4247
    977  5427 4247
    978  5409 4322
    979  5752 4608
    980  5553 4440
    981  5265 4143
    982  5320 4174
    983  5131 3958
    984  5328 4141
    985  5279 4183
    986  5266 4152
    987  4966 3894
    988  5153 4038
    989  5233 4094
    990  5502 4322
    991  4287 3254
    992  4358 3269
    993  4609 3460
    994  5376 4161
    995  5465 4271
    996  5971 4680
    997  5932 4654
    998  5922 4648
    999  5756 4433
    1000 5799 4409
    1001 6801 5160
    1002 6943 5342
    1003 7092 5520
    1004 7072 5518
    1005 7050 5489
    1006 7051 5462
    1007 6945 5453
    1008 6799 5350
    1009 6504 5164
    1010 6228 4914
    1011 6179 4832
    1012 6097 4803
    1013 5982 4767
    1014 6047 4924
    1015 5438 4209
    1016 5078 3662
    1017 5519 4021
    1018 5756 4322
    1019 5932 4365
    1020 5976 4235
    1021 5781 4007
    1022 5548 3776
    1023 5517 3724
    1024 5702 3837
    1025 5729 3863
    1026 5815 3976
    1027 5723 3932
    1028 5607 3828
    1029 5651 3932
    1030 5742 4049
    1031 5733 4078
    1032 5733 4078
    1033 5474 3978
    1034 5583 4144
    1035 5696 4238
    1036 5536 4111
    1037 5552 4125
    1038 5504 4104
    1039 5551 4159
    1040 5447 4058
    1041 5240 3948
    1042 5423 4108
    1043 5301 4046
    1044 5327 4066
    1045 5212 3994
    1046 5212 3994
    1047 5305 4134
    1048 5529 4346
    1049 5527 4361
    1050 5583 4383
    1051 5709 4496
    1052 5818 4581
    1053 5777 4548
    1054 5779 4604
    1055 5686 4457
    1056 5645 4435
    1057 5809 4588
    1058 5616 4422
    1059 5622 4414
    1060 5418 4236
    1061 5396 4206
    1062 5380 4236
    1063 5489 4320
    1064 5476 4305
    1065 5762 4518
    1066 5664 4463
    1067 5327 4217
    1068 4285 3267
    1069 4284 3224
    1070 4448 3350
    1071 4924 3748
    1072 5094 3922
    1073 5548 4332
    1074 5566 4303
    1075 5785 4481
    1076 5767 4448
    1077 5987 4614
    1078 6161 4757
    1079 6508 5144
    1080 7113 5485
    1081 7107 5478
    1082 6991 5447
    1083 6810 5232
    1084 6698 5208
    1085 6543 5177
    1086 6308 4961
    1087 6272 4967
    1088 6358 5137
    1089 6358 5137
    1090 6182 4915
    1091 6047 4924
    1092 5656 4454
    1093 5187 3813
    1094 5543 4110
    1095 5762 4211
    1096 5632 3969
    1097 5547 3781
    1098 5520 3774
    1099 5664 3861
    1100 5758 3927
    1101 5700 3864
    1102 5741 3915
    1103 5779 3932
    1104 5723 3932
    1105 5589 3848
    1106 5635 3899
    1107 5758 4033
    1108 5830 4115
    1109 5814 4095
    1110 5723 4197
    1111 5806 4292
    1112 5890 4434
    1113 5757 4350
    1114 5988 4602
    1115 5798 4446
    1116 5640 4231
    1117 5640 4231
    1118 5592 4225
    1119 5742 4364
    1120 5603 4272
    1121 5568 4264
    1122 5345 4112
    1123 5629 4375
    1124 5703 4446
    1125 5797 4514
    1126 5640 4397
    1127 5782 4541
    1128 5799 4543
    1129 5815 4575
    1130 5869 4606
    1131 5766 4532
    1132 5811 4553
    1133 5811 4549
    1134 5957 4682
    1135 5882 4635
    1136 5945 4724
    1137 5690 4482
    1138 5938 4672
    1139 5763 4512
    1140 5792 4501
    1141 5849 4601
    1142 5830 4570
    1143 5780 4554
    1144 5688 4514
    1145 5508 4399
    1146 4830 3818
    1147 4553 3535
    1148 4169 3132
    1149 4197 3160
    1150 4603 3486
    1151 4711 3566
    1152 5152 3905
    1153 5298 4014
    1154 5704 4405
    1155 6018 4652
    1156 6496 5093
    1157 6714 5401
    1158 6714 5401
    1159 6622 5325
    1160 6532 5298
    1161 6752 5237
    1162 6752 5237
    1163 6691 5230
    1164 6730 5409
    1165 6361 5041
    1166 6205 4919
    1167 6392 5134
    1168 6225 4959
    1169 6178 4925
    1170 5854 4628
    1171 5650 4399
    1172 5607 4151
    1173 5687 4148
    1174 5660 4042
    1175 5660 4042
    1176 5480 3748
    1177 5580 3820
    1178 5576 3753
    1179 5666 3828
    1180 5791 3928
    1181 5741 3915
    1182 5610 3792
    1183 5709 3938
    1184 5785 4040
    1185 5809 4065
    1186 5878 4107
    1187 5889 4144
    1188 5864 4120
    1189 5762 4044
    1190 5867 4366
    1191 6053 4582
    1192 6052 4617
    1193 5988 4602
    1194 6185 4793
    1195 5945 4584
    1196 6065 4697
    1197 5936 4547
    1198 5997 4615
    1199 5791 4436
    1200 5715 4377
    1201 5846 4526
    1202 6041 4678
    1203 5835 4538
    1204 5994 4688
    1205 5824 4536
    1206 5782 4541
    1207 5995 4733
    1208 5917 4663
    1209 6067 4773
    1210 6015 4717
    1211 6186 4851
    1212 6082 4795
    1213 6034 4770
    1214 6043 4750
    1215 6170 4863
    1216 6151 4834
    1217 6151 4834
    1218 6004 4710
    1219 5792 4501
    1220 6031 4755
    1221 5904 4664
    1222 5836 4618
    1223 5674 4499
    1224 5520 4413
    1225 4884 3833
    1226 4752 3739
    1227 4343 3370
    1228 4570 3602
    1229 4574 3518
    1230 4846 3736
    1231 4864 3694
    1232 5298 4014
    1233 5481 4209
    1234 5897 4551
    1235 6274 4919
    1236 6688 5322
    1237 6538 5281
    1238 6445 5246
    1239 5878 4705
    1240 5806 4663
    1241 6722 5301
    1242 6750 5388
    1243 6615 5303
    1244 6301 5045
    1245 6220 5055
    1246 6121 4880
    1247 6195 4942
    1248 6195 4942
    1249 6027 4765
    1250 5939 4662
    1251 5597 4278
    1252 5561 4135
    1253 5630 4027
    1254 5577 3895
    1255 5528 3789
    1256 5484 3716
    1257 5589 3760
    1258 5790 3938
    1259 5870 3990
    1260 5718 3903
    1261 5705 3925
    1262 5619 3849
    1263 5659 3914
    1264 5817 4034
    1265 5840 4063
    1266 5859 4080
    1267 5834 4065
    1268 5871 4114
    1269 6129 4625
    1270 6210 4741
    1271 6290 4785
    1272 6309 4828
    1273 6350 4913
    1274 6305 4934
    1275 6232 4874
    1276 6145 4778
    1277 6171 4817
    1278 5927 4542
    1279 5834 4451
    1280 5960 4619
    1281 6182 4807
    1282 6150 4783
    1283 6059 4740
    1284 6147 4832
    1285 6172 4834
    1286 6271 4934
    1287 6277 4953
    1288 6451 5090
    1289 6343 4982
    1290 6343 4982
    1291 6223 4895
    1292 6170 4882
    1293 6125 4872
    1294 6206 4894
    1295 6209 4848
    1296 6215 4866
    1297 6330 4965
    1298 6188 4862
    1299 6166 4853
    1300 6028 4766
    1301 6044 4782
    1302 5804 4653
    1303 5804 4653
    1304 5278 4208
    1305 5069 4014
    1306 4623 3632
    1307 4725 3682
    1308 4796 3784
    1309 4846 3736
    1310 5143 3979
    1311 5416 4217
    1312 5611 4366
    1313 6014 4730
    1314 6040 4767
    1315 6138 4916
    1316 5680 4482
    1317 6060 4833
    1318 5640 4464
    1319 5690 4580
    1320 5618 4511
    1321 5933 4802
    1322 6547 5174
    1323 6662 5299
    1324 6498 5183
    1325 6351 5058
    1326 6220 5055
    1327 6029 4819
    1328 6098 4836
    1329 6043 4754
    1330 5974 4664
    1331 5954 4617
    1332 5685 4380
    1333 5543 4127
    1334 5377 3781
    1335 5354 3665
    1336 5542 3824
    1337 5535 3771
    1338 5616 3806
    1339 5832 3989
    1340 5903 4029
    1341 5834 3977
    1342 5782 3954
    1343 5695 3941
    1344 5729 3959
    1345 5832 4063
    1346 5836 4050
    1347 5736 3975
    1348 5760 4006
    1349 5711 3949
    1350 6284 4749
    1351 6261 4756
    1352 6288 4741
    1353 6346 4809
    1354 6354 4849
    1355 6350 4969
    1356 6222 4892
    1357 6120 4772
    1358 6017 4652
    1359 5952 4581
    1360 6161 4778
    1361 6202 4815
    1362 6267 4873
    1363 6339 4946
    1364 6237 4871
    1365 6355 4991
    1366 6280 4946
    1367 6370 5025
    1368 6391 5047
    1369 6484 5129
    1370 6430 5071
    1371 6368 5036
    1372 6257 4941
    1373 6321 4991
    1374 6310 4994
    1375 6229 4892
    1376 6333 5000
    1377 6322 4947
    1378 6322 4947
    1379 6270 4918
    1380 6179 4883
    1381 6086 4816
    1382 6080 4814
    1383 6066 4868
    1384 5793 4627
    1385 5270 4186
    1386 5263 4257
    1387 4913 3878
    1388 4968 3912
    1389 5097 4003
    1390 5166 4012
    1391 5348 4194
    1392 5286 4080
    1393 5372 4152
    1394 5810 4558
    1395 5747 4512
    1396 5846 4606
    1397 5677 4464
    1398 5478 4332
    1399 5666 4538
    1400 5673 4575
    1401 5925 4802
    1402 6116 4955
    1403 6258 5037
    1404 6311 5056
    1405 6312 4889
    1406 6355 5011
    1407 6216 4925
    1408 6077 4848
    1409 5871 4649
    1410 5721 4454
    1411 5916 4632
    1412 5978 4631
    1413 5989 4641
    1414 6002 4560
    1415 5878 4465
    1416 5711 4176
    1417 5337 3818
    1418 5208 3600
    1419 5399 3658
    1420 5671 3867
    1421 5824 3970
    1422 5937 4065
    1423 5829 3980
    1424 5814 3981
    1425 5796 3987
    1426 5670 3881
    1427 5670 3881
    1428 5845 4051
    1429 5860 4043
    1430 5823 4022
    1431 5909 4102
    1432 5792 4029
    1433 5669 3888
    1434 6305 4736
    1435 6269 4705
    1436 6339 4776
    1437 6262 4751
    1438 6193 4813
    1439 6274 4946
    1440 6264 4953
    1441 6056 4744
    1442 6056 4744
    1443 6059 4702
    1444 6161 4778
    1445 6209 4813
    1446 6264 4885
    1447 6353 4974
    1448 6366 4931
    1449 6349 4912
    1450 6405 5023
    1451 6503 5098
    1452 6499 5150
    1453 6505 5153
    1454 6423 5080
    1455 6354 5015
    1456 6310 4973
    1457 6321 4991
    1458 6380 5039
    1459 6319 4980
    1460 6394 5057
    1461 6309 4998
    1462 6175 4892
    1463 6234 4928
    1464 6216 4935
    1465 6082 4830
    1466 6043 4826
    1467 5940 4765
    1468 6035 4918
    1469 5685 4603
    1470 5263 4257
    1471 5455 4440
    1472 5294 4245
    1473 5434 4347
    1474 5297 4160
    1475 5595 4434
    1476 5541 4411
    1477 5777 4672
    1478 5646 4459
    1479 5762 4600
    1480 5739 4504
    1481 6136 4923
    1482 6054 4923
    1483 5666 4538
    1484 5935 4790
    1485 5969 4837
    1486 6082 4913
    1487 6199 4972
    1488 6294 5035
    1489 6249 4915
    1490 6181 4876
    1491 6199 4905
    1492 6177 4935
    1493 5882 4693
    1494 5760 4544
    1495 5849 4577
    1496 5963 4646
    1497 6024 4543
    1498 5942 4378
    1499 5738 4148
    1500 5546 3934
    1501 5283 3648
    1502 5404 3711
    1503 5561 3757
    1504 5653 3850
    1505 5760 3932
    1506 5847 3979
    1507 5749 3914
    1508 5813 3985
    1509 5811 4016
    1510 5742 3960
    1511 5809 3997
    1512 5886 4046
    1513 5908 4043
    1514 5959 4099
    1515 5906 4065
    1516 5897 4069
    1517 5928 4133
    1518 6413 4805
    1519 6384 4810
    1520 6384 4810
    1521 6280 4779
    1522 6304 4926
    1523 6360 5032
    1524 6391 5054
    1525 6242 4951
    1526 6192 4849
    1527 6169 4811
    1528 6149 4740
    1529 6291 4892
    1530 6407 5040
    1531 6465 5061
    1532 6469 5077
    1533 6469 5077
    1534 6452 5039
    1535 6546 5120
    1536 6550 5157
    1537 6491 5118
    1538 6476 5145
    1539 6354 5015
    1540 6336 4974
    1541 6400 5052
    1542 6362 5034
    1543 6313 4976
    1544 6239 4935
    1545 6236 4952
    1546 6316 5040
    1547 6225 4977
    1548 6242 4984
    1549 6028 4795
    1550 5940 4741
    1551 5968 4770
    1552 6035 4918
    1553 6209 5075
    1554 6006 4969
    1555 6030 4968
    1556 5772 4659
    1557 5772 4615
    1558 5843 4701
    1559 6058 4926
    1560 6014 4854
    1561 6014 4854
    1562 5856 4717
    1563 5974 4777
    1564 5977 4779
    1565 6136 4923
    1566 6355 5135
    1567 6226 5028
    1568 6385 5185
    1569 6225 5069
    1570 6383 5207
    1571 6340 5117
    1572 6117 4715
    1573 6288 4985
    1574 6207 4896
    1575 6209 4911
    1576 6071 4780
    1577 5921 4707
    1578 5858 4626
    1579 5885 4574
    1580 5952 4482
    1581 5917 4346
    1582 5917 4346
    1583 5754 4148
    1584 5546 3934
    1585 5390 3732
    1586 5473 3769
    1587 5596 3824
    1588 5682 3879
    1589 5619 3854
    1590 5672 3914
    1591 5889 4138
    1592 5914 4139
    1593 5946 4232
    1594 5757 3985
    1595 5893 4055
    1596 5862 4019
    1597 5908 4043
    1598 6005 4139
    1599 5962 4101
    1600 5895 4084
    1601 5876 4080
    1602 5680 3889
    1603 6391 4785
    1604 6416 4786
    1605 6426 4842
    1606 6305 4855
    1607 6344 4993
    1608 6497 5197
    1609 6549 5233
    1610 6494 5182
    1611 6410 5079
    1612 6171 4816
    1613 6171 4816
    1614 6222 4838
    1615 6338 4918
    1616 6450 5045
    1617 6424 5004
    1618 6517 5092
    1619 6429 5005
    1620 6456 5007
    1621 6275 4916
    1622 6353 4984
    1623 6471 5134
    1624 6307 4974
    1625 6309 4963
    1626 6266 4938
    1627 6366 5019
    1628 6297 4986
    1629 6209 4922
    1630 6195 4940
    1631 6282 4991
    1632 6410 5096
    1633 6312 5049
    1634 6218 4978
    1635 5988 4832
    1636 5938 4792
    1637 6084 5015
    1638 6145 5011
    1639 6216 5092
    1640 6237 5112
    1641 6159 5064
    1642 5834 4715
    1643 5674 4520
    1644 5951 4769
    1645 6125 4940
    1646 6044 4894
    1647 5910 4747
    1648 5965 4733
    1649 6208 4982
    1650 6424 5166
    1651 6446 5192
    1652 6475 5230
    1653 6434 5184
    1654 6434 5184
    1655 6456 5246
    1656 6428 5160
    1657 6217 4931
    1658 6287 5016
    1659 6186 4892
    1660 6039 4744
    1661 6039 4744
    1662 6072 4824
    1663 5987 4722
    1664 5882 4602
    1665 5977 4535
    1666 5993 4378
    1667 5771 4126
    1668 5546 3848
    1669 5419 3726
    1670 5356 3670
    1671 5564 3786
    1672 5603 3818
    1673 5650 3875
    1674 5650 3875
    1675 5682 3953
    1676 5909 4163
    1677 5954 4188
    1678 5980 4217
    1679 5895 4205
    1680 5893 4055
    1681 6046 4179
    1682 6076 4209
    1683 6076 4234
    1684 5807 3992
    1685 5761 3990
    1686 5963 4215
    1687 6331 4709
    1688 6297 4715
    1689 6302 4846
    1690 6033 4774
    1691 6116 4790
    1692 6497 5197
    1693 6537 5226
    1694 6581 5236
    1695 6476 5121
    1696 6239 4949
    1697 6277 4969
    1698 6318 4918
    1699 6393 4992
    1700 6431 5038
    1701 6403 5045
    1702 6350 4993
    1703 6353 5028
    1704 6290 4913
    1705 6275 4916
    1706 6028 4763
    1707 6197 4874
    1708 6085 4799
    1709 6175 4874
    1710 6171 4906
    1711 6203 4917
    1712 6225 4992
    1713 6271 5034
    1714 6412 5131
    1715 6512 5197
    1716 6610 5307
    1717 6437 5193
    1718 6218 4978
    1719 6151 4972
    1720 6008 4887
    1721 6243 5135
    1722 6289 5235
    1723 6349 5257
    1724 6181 5071
    1725 6065 4965
    1726 5621 4524
    1727 5831 4667
    1728 6130 4983
    1729 6194 5056
    1730 6088 4930
    1731 6078 4864
    1732 6078 4864
    1733 6280 5042
    1734 6492 5242
    1735 6551 5288
    1736 6492 5213
    1737 6339 5059
    1738 6205 4910
    1739 6228 4912
    1740 5599 4408
    1741 5599 4408
    1742 5787 4413
    1743 6163 4877
    1744 6351 5116
    1745 6303 5045
    1746 6146 4870
    1747 6065 4836
    1748 5990 4718
    1749 5932 4635
    1750 5898 4483
    1751 6072 4425
    1752 5898 4126
    1753 5667 3906
    1754 5667 3906
    1755 5464 3767
    1756 5500 3820
    1757 5398 3720
    1758 5538 3833
    1759 5699 3944
    1760 5784 4071
    1761 5853 4128
    1762 5944 4187
    1763 5944 4195
    1764 5959 4218
    1765 6064 4344
    1766 6053 4199
    1767 6077 4220
    1768 6135 4273
    1769 5861 4040
    1770 5791 3959
    1771 5978 4166
    1772 6135 4311
    1773 6036 4877
    1774 6261 4985
    1775 6325 5053
    1776 6392 5085
    1777 6347 5040
    1778 6288 5000
    1779 6133 4803
    1780 6243 4880
    1781 6327 4941
    1782 6443 5038
    1783 6345 4966
    1784 6385 5042
    1785 6353 5028
    1786 6332 5048
    1787 6028 4828
    1788 5928 4709
    1789 6036 4787
    1790 6212 4902
    1791 6309 5028
    1792 6408 5117
    1793 6317 5090
    1794 6317 5090
    1795 6310 5078
    1796 6374 5124
    1797 6529 5229
    1798 6610 5307
    1799 6569 5284
    1800 6350 5168
    1801 6027 4896
    1802 5868 4750
    1803 5885 4746
    1804 6306 5137
    1805 6402 5225
    1806 6247 5166
    1807 6247 5166
    1808 5831 4746
    1809 5915 4794
    1810 6209 5043
    1811 6194 5056
    1812 6253 5097
    1813 6276 5077
    1814 6502 5283
    1815 6621 5382
    1816 6562 5304
    1817 6369 5084
    1818 6396 5136
    1819 5590 4502
    1820 5682 4268
    1821 5937 4569
    1822 6337 5135
    1823 6290 5069
    1824 6183 4920
    1825 6079 4817
    1826 6052 4836
    1827 5932 4635
    1828 5769 4352
    1829 5989 4370
    1830 5874 4111
    1831 5779 3976
    1832 5709 3900
    1833 5681 3933
    1834 5701 3991
    1835 5637 3952
    1836 5849 4147
    1837 5835 4135
    1838 5887 4168
    1839 5890 4163
    1840 5944 4187
    1841 6074 4293
    1842 6042 4300
    1843 6163 4397
    1844 6038 4264
    1845 5985 4186
    1846 5952 4159
    1847 5752 3934
    1848 6093 4243
    1849 6011 4192
    1850 5878 4110
    1851 6011 4246
    1852 6038 4887
    1853 6122 4921
    1854 6285 5045
    1855 6160 4910
    1856 6228 4898
    1857 6167 4845
    1858 6100 4758
    1859 6029 4698
    1860 6176 4812
    1861 6262 4923
    1862 6245 4926
    1863 6406 5073
    1864 6321 5012
    1865 6243 5001
    1866 6088 4845
    1867 6080 4816
    1868 6230 4965
    1869 6333 4994
    1870 6387 5034
    1871 6393 5153
    1872 6338 5144
    1873 6293 5091
    1874 6400 5130
    1875 6357 5108
    1876 6454 5197
    1877 6359 5167
    1878 6267 5125
    1879 6267 5125
    1880 5804 4746
    1881 6000 4973
    1882 6094 4927
    1883 6339 5163
    1884 6402 5261
    1885 6362 5281
    1886 6168 5090
    1887 6205 5062
    1888 6304 5138
    1889 6382 5205
    1890 6501 5311
    1891 6617 5417
    1892 6609 5386
    1893 6547 5266
    1894 6624 5311
    1895 6565 5248
    1896 6278 5253
    1897 5378 4200
    1898 5600 4421
    1899 6124 4835
    1900 6118 4820
    1901 6087 4714
    1902 6079 4817
    1903 6035 4684
    1904 5911 4466
    1905 5952 4350
    1906 5926 4194
    1907 5816 4044
    1908 5632 3822
    1909 5588 3788
    1910 5684 3909
    1911 5750 4011
    1912 5794 4083
    1913 5992 4250
    1914 5871 4161
    1915 5887 4168
    1916 6017 4255
    1917 6133 4322
    1918 6218 4388
    1919 6218 4434
    1920 6261 4473
    1921 6086 4303
    1922 6142 4342
    1923 6049 4229
    1924 6049 4229
    1925 5919 4084
    1926 6063 4212
    1927 5951 4142
    1928 5878 4110
    1929 6110 4345
    1930 5908 4762
    1931 5792 4614
    1932 6058 4784
    1933 5940 4725
    1934 5938 4715
    1935 6029 4698
    1936 6063 4763
    1937 6129 4801
    1938 6126 4838
    1939 6322 4980
    1940 6275 4959
    1941 6155 4885
    1942 6073 4841
    1943 6229 4956
    1944 6251 4982
    1945 6303 5041
    1946 6369 5107
    1947 6383 5165
    1948 6284 5152
    1949 6284 5152
    1950 6209 5011
    1951 6173 4999
    1952 6265 5089
    1953 6263 5159
    1954 6234 5133
    1955 6129 5040
    1956 6288 5172
    1957 6463 5369
    1958 6426 5287
    1959 6436 5296
    1960 6372 5316
    1961 6263 5182
    1962 6210 5098
    1963 6309 5204
    1964 6375 5230
    1965 6384 5221
    1966 6481 5265
    1967 6605 5345
    1968 6509 5233
    1969 6534 5178
    1970 6544 5220
    1971 6698 5609
    1972 6276 5290
    1973 6109 5117
    1974 5774 4493
    1975 5912 4508
    1976 5700 4163
    1977 5974 4500
    1978 5829 4296
    1979 6000 4523
    1980 5988 4386
    1981 6012 4287
    1982 5813 4039
    1983 5591 3802
    1984 5650 3872
    1985 5649 3877
    1986 5860 4102
    1987 5969 4224
    1988 5939 4203
    1989 5978 4274
    1990 5956 4238
    1991 6045 4280
    1992 6225 4413
    1993 6225 4413
    1994 6294 4474
    1995 6249 4420
    1996 6167 4362
    1997 6198 4372
    1998 6138 4289
    1999 6089 4262
    2000 6080 4261
    2001 6053 4239
    2002 6214 4407
    2003 6217 4415
    2004 6234 4419
    2005 5693 4641
    2006 5456 4438
    2007 5765 4602
    2008 5768 4622
    2009 5894 4727
    2010 5946 4708
    2011 5968 4730
    2012 6093 4805
    2013 6034 4777
    2014 6135 4830
    2015 6146 4900
    2016 6170 4977
    2017 5957 4808
    2018 6002 4790
    2019 6011 4805
    2020 6277 5018
    2021 6369 5107
    2022 6344 5119
    2023 6349 5180
    2024 6159 5050
    2025 6018 4909
    2026 6048 4944
    2027 6226 5126
    2028 6135 5046
    2029 6117 5066
    2030 6233 5208
    2031 6220 5129
    2032 6354 5246
    2033 6452 5349
    2034 6436 5296
    2035 6375 5301
    2036 6238 5214
    2037 6079 5014
    2038 6493 5295
    2039 6486 5296
    2040 6570 5400
    2041 6570 5400
    2042 6523 5304
    2043 6509 5233
    2044 6409 5104
    2045 6683 5585
    2046 6502 5503
    2047 5906 4791
    2048 5884 4504
    2049 5752 4243
    2050 5793 4225
    2051 5629 4059
    2052 5828 4223
    2053 6004 4284
    2054 5918 4126
    2055 5849 4014
    2056 5797 3989
    2057 5878 4076
    2058 5904 4134
    2059 5860 4102
    2060 5950 4191
    2061 6005 4282
    2062 6113 4403
    2063 6012 4345
    2064 6065 4378
    2065 6178 4409
    2066 6238 4464
    2067 6177 4326
    2068 6060 4255
    2069 6060 4255
    2070 6163 4321
    2071 6230 4416
    2072 6196 4394
    2073 6148 4370
    2074 6174 4351
    2075 5638 4623
    2076 5580 4522
    2077 5807 4649
    2078 5821 4618
    2079 5974 4758
    2080 5974 4758
    2081 5978 4773
    2082 5971 4756
    2083 6027 4784
    2084 6110 4878
    2085 6179 4942
    2086 6064 4883
    2087 5944 4771
    2088 6104 4897
    2089 6105 4921
    2090 6150 4923
    2091 5967 4789
    2092 5991 4868
    2093 5955 4884
    2094 6022 4944
    2095 6064 5019
    2096 6061 4976
    2097 6006 4964
    2098 6111 5055
    2099 6241 5191
    2100 6275 5238
    2101 6312 5202
    2102 6264 5182
    2103 6223 5153
    2104 6321 5205
    2105 6566 5410
    2106 6507 5355
    2107 6315 5123
    2108 6319 5104
    2109 6491 5253
    2110 6587 5372
    2111 6483 5212
    2112 6416 5106
    2113 6517 5220
    2114 6808 5584
    2115 6683 5585
    2116 6627 5565
    2117 6268 5254
    2118 5796 4339
    2119 5867 4354
    2120 5558 3924
    2121 5704 4098
    2122 5972 4281
    2123 5972 4281
    2124 5934 4113
    2125 5773 3927
    2126 5854 4033
    2127 5878 4076
    2128 6014 4208
    2129 5969 4186
    2130 6101 4328
    2131 6202 4446
    2132 6335 4568
    2133 6262 4538
    2134 6424 4698
    2135 6323 4611
    2136 6403 4659
    2137 6248 4479
    2138 5940 4175
    2139 6192 4417
    2140 6168 4394
    2141 6200 4460
    2142 6274 4542
    2143 6183 4421
    2144 5584 4585
    2145 5593 4485
    2146 5602 4426
    2147 5925 4669
    2148 5900 4661
    2149 5881 4713
    2150 5879 4686
    2151 5994 4804
    2152 5973 4813
    2153 6059 4899
    2154 5962 4849
    2155 6048 4891
    2156 6098 4947
    2157 6079 4920
    2158 5997 4845
    2159 5853 4727
    2160 5808 4690
    2161 5777 4707
    2162 5997 4948
    2163 6030 5012
    2164 5996 5006
    2165 6040 5056
    2166 6112 5108
    2167 6282 5240
    2168 6256 5228
    2169 6112 5087
    2170 6234 5159
    2171 6310 5192
    2172 6497 5373
    2173 6428 5300
    2174 6337 5172
    2175 6311 5073
    2176 6389 5128
    2177 6533 5230
    2178 6533 5230
    2179 6465 5217
    2180 6555 5476
    2181 6437 5331
    2182 6099 4963
    2183 6099 4963
    2184 5754 4382
    2185 5867 4394
    2186 5696 4085
    2187 5762 4105
    2188 5967 4316
    2189 5906 4139
    2190 5824 4009
    2191 5883 4051
    2192 6037 4187
    2193 6089 4234
    2194 6146 4282
    2195 6145 4320
    2196 6145 4320
    2197 6265 4484
    2198 6410 4595
    2199 6395 4610
    2200 6485 4684
    2201 6496 4750
    2202 6476 4688
    2203 6446 4687
    2204 6409 4591
    2205 6146 4384
    2206 6315 4543
    2207 6348 4564
    2208 6278 4508
    2209 5494 4443
    2210 5577 4477
    2211 5737 4525
    2212 5691 4519
    2213 5780 4577
    2214 5770 4591
    2215 5933 4740
    2216 5989 4817
    2217 6033 4904
    2218 5962 4849
    2219 5886 4783
    2220 5911 4807
    2221 5791 4739
    2222 5731 4692
    2223 5585 4600
    2224 5650 4623
    2225 5641 4646
    2226 5857 4838
    2227 5857 4838
    2228 5965 4957
    2229 6050 5056
    2230 5931 4922
    2231 6037 5012
    2232 6040 5001
    2233 6179 5075
    2234 6332 5216
    2235 6420 5298
    2236 6428 5300
    2237 6339 5178
    2238 6354 5131
    2239 6479 5236
    2240 6575 5296
    2241 6757 5475
    2242 6654 5426
    2243 6726 5520
    2244 6586 5533
    2245 6402 5307
    2246 5963 4688
    2247 5598 4196
    2248 5569 4091
    2249 5601 4035
    2250 5803 4092
    2251 5833 4010
    2252 5555 3809
    2253 5553 3736
    2254 5553 3736
    2255 5895 4053
    2256 5983 4110
    2257 6057 4177
    2258 6135 4261
    2259 6179 4345
    2260 6295 4486
    2261 6431 4620
    2262 6493 4664
    2263 6475 4637
    2264 6513 4679
    2265 6453 4645
    2266 6425 4638
    2267 6378 4556
    2268 6366 4559
    2269 6414 4587
    2270 6441 4610
    2271 6256 4439
    2272 6279 4454
    2273 6265 4448
    2274 6255 4454
    2275 5631 4500
    2276 5746 4580
    2277 5793 4631
    2278 5951 4740
    2279 6055 4897
    2280 5965 4875
    2281 5811 4764
    2282 5731 4716
    2283 5722 4703
    2284 5722 4703
    2285 5643 4669
    2286 5541 4595
    2287 5514 4570
    2288 5978 5019
    2289 5947 4987
    2290 5947 4987
    2291 5954 4964
    2292 5884 4910
    2293 6016 4978
    2294 6075 5038
    2295 6065 4980
    2296 6332 5216
    2297 6219 5136
    2298 6214 5069
    2299 6209 5038
    2300 6395 5146
    2301 6545 5306
    2302 6710 5485
    2303 6677 5418
    2304 6514 5221
    2305 6210 4977
    2306 6313 5207
    2307 5875 4751
    2308 5477 4076
    2309 5482 3883
    2310 5442 3945
    2311 5714 4060
    2312 5767 3984
    2313 5833 4010
    2314 5354 3588
    2315 5251 3514
    2316 5727 3955
    2317 5876 4033
    2318 6047 4168
    2319 6167 4303
    2320 6217 4353
    2321 6225 4355
    2322 6234 4346
    2323 6389 4555
    2324 6357 4502
    2325 6471 4615
    2326 6513 4679
    2327 6492 4658
    2328 6424 4619
    2329 6445 4610
    2330 6514 4671
    2331 6536 4684
    2332 6387 4598
    2333 6128 4371
    2334 6251 4472
    2335 6279 4454
    2336 6301 4501
    2337 6269 4443
    2338 6250 4427
    2339 5653 4520
    2340 5793 4631
    2341 5835 4658
    2342 5953 4753
    2343 5889 4759
    2344 5841 4790
    2345 5790 4771
    2346 5623 4658
    2347 5971 5005
    2348 5994 5054
    2349 5997 5077
    2350 5941 5016
    2351 5875 4981
    2352 5779 4900
    2353 5866 4916
    2354 5879 4944
    2355 5889 4870
    2356 5777 4752
    2357 6044 4949
    2358 6137 5048
    2359 6045 5003
    2360 6066 4987
    2361 6136 5011
    2362 6292 5095
    2363 6372 5141
    2364 6629 5406
    2365 6233 4844
    2366 6186 4952
    2367 6208 4865
    2368 5918 4636
    2369 6152 5022
    2370 6023 4911
    2371 5475 4233
    2372 5443 4028
    2373 5868 4100
    2374 5837 4009
    2375 5783 3923
    2376 5495 3669
    2377 5556 3754
    2378 5696 3868
    2379 5808 3968
    2380 5876 4063
    2381 5903 4053
    2382 6074 4166
    2383 6097 4191
    2384 6201 4325
    2385 6153 4276
    2386 6130 4264
    2387 6327 4477
    2388 6357 4498
    2389 6527 4652
    2390 6537 4668
    2391 6555 4705
    2392 6562 4693
    2393 6466 4605
    2394 6466 4605
    2395 6439 4608
    2396 6410 4598
    2397 6322 4587
    2398 6093 4364
    2399 6213 4457
    2400 6361 4602
    2401 6450 4658
    2402 6391 4576
    2403 6381 4548
    2404 6250 4429
    2405 5770 4613
    2406 5849 4733
    2407 5768 4657
    2408 5819 4739
    2409 5895 4828
    2410 5906 4871
    2411 5738 4802
    2412 5719 4788
    2413 5761 4875
    2414 5815 4906
    2415 5760 4895
    2416 5775 4879
    2417 5629 4741
    2418 5629 4755
    2419 5625 4750
    2420 5801 4892
    2421 5764 4864
    2422 5716 4796
    2423 5777 4752
    2424 5801 4793
    2425 6026 4947
    2426 5986 4926
    2427 6032 4999
    2428 5970 4903
    2429 6132 4973
    2430 6325 5150
    2431 6469 5285
    2432 5993 4506
    2433 6285 4995
    2434 6499 5362
    2435 6013 4809
    2436 6033 4877
    2437 6087 4955
    2438 5877 4747
    2439 5721 4494
    2440 5856 4187
    2441 6009 4136
    2442 5925 4055
    2443 5911 4002
    2444 5602 3745
    2445 5733 3840
    2446 5776 3943
    2447 5894 4042
    2448 5834 4005
    2449 5908 4097
    2450 5929 4093
    2451 6028 4170
    2452 6120 4233
    2453 6107 4283
    2454 6137 4340
    2455 6182 4363
    2456 6195 4377
    2457 6391 4513
    2458 6454 4584
    2459 6544 4660
    2460 6476 4615
    2461 6492 4654
    2462 6410 4601
    2463 6365 4542
    2464 6323 4525
    2465 6196 4434
    2466 6023 4300
    2467 6098 4357
    2468 6472 4666
    2469 6434 4619
    2470 6335 4508
    2471 6302 4485
    2472 5834 4763
    2473 5902 4811
    2474 5957 4882
    2475 5855 4841
    2476 5877 4884
    2477 5855 4882
    2478 5808 4889
    2479 5843 4943
    2480 5986 5109
    2481 5817 4983
    2482 5786 4998
    2483 5595 4769
    2484 5515 4704
    2485 5442 4597
    2486 5524 4697
    2487 5631 4785
    2488 5620 4751
    2489 5637 4812
    2490 5698 4789
    2491 5650 4768
    2492 5725 4729
    2493 5741 4734
    2494 5842 4864
    2495 5842 4864
    2496 5954 4899
    2497 6148 5003
    2498 6435 5211
    2499 6468 5352
    2500 6247 5085
    2501 6247 5085
    2502 6251 5117
    2503 5877 4747
    2504 5815 4647
    2505 5969 4475
    2506 6017 4206
    2507 5929 4012
    2508 5828 3897
    2509 5864 3930
    2510 6061 4137
    2511 6101 4210
    2512 5970 4121
    2513 5806 3990
    2514 5806 3990
    2515 5934 4107
    2516 6028 4170
    2517 6066 4210
    2518 6188 4321
    2519 6225 4402
    2520 6082 4289
    2521 6169 4380
    2522 6341 4498
    2523 6321 4473
    2524 6405 4572
    2525 6267 4479
    2526 6347 4526
    2527 6211 4405
    2528 6321 4516
    2529 6323 4525
    2530 6212 4451
    2531 6007 4285
    2532 5997 4280
    2533 6242 4476
    2534 6212 4418
    2535 6254 4457
    2536 6086 4328
    2537 6358 4693
    2538 5970 4914
    2539 5957 4947
    2540 5927 4893
    2541 5896 4895
    2542 5831 4860
    2543 5817 4887
    2544 5875 4958
    2545 6033 5106
    2546 6054 5191
    2547 6014 5201
    2548 5662 4883
    2549 5696 4883
    2550 5547 4784
    2551 5626 4810
    2552 5707 4887
    2553 5726 4906
    2554 5726 4906
    2555 5629 4819
    2556 5682 4873
    2557 5609 4763
    2558 5519 4663
    2559 5786 4815
    2560 5799 4878
    2561 5968 4961
    2562 6315 5216
    2563 5847 4601
    2564 6189 4968
    2565 6459 5308
    2566 6376 5254
    2567 6371 5148
    2568 6305 5055
    2569 5827 4655
    2570 5382 4127
    2571 5663 3984
    2572 5924 4078
    2573 5996 4068
    2574 5996 4068
    2575 5940 4007
    2576 6101 4172
    2577 6175 4266
    2578 6062 4203
    2579 5872 4071
    2580 6009 4211
    2581 6120 4265
    2582 6150 4303
    2583 6230 4367
    2584 6249 4396
    2585 6258 4452
    2586 6302 4498
    2587 6302 4498
    2588 6231 4423
    2589 6190 4383
    2590 6249 4418
    2591 6089 4333
    2592 6140 4399
    2593 6211 4405
    2594 6141 4366
    2595 6178 4387
    2596 6080 4324
    2597 6047 4319
    2598 6290 4537
    2599 6240 4517
    2600 6224 4501
    2601 6175 4413
    2602 6123 4397
    2603 6187 4462
    2604 6243 4593
    2605 6211 4629
    2606 6085 4944
    2607 6087 4986
    2608 6005 4922
    2609 6000 4955
    2610 5944 4922
    2611 5860 4876
    2612 5753 4801
    2613 5793 4871
    2614 5880 4974
    2615 5985 5086
    2616 6084 5169
    2617 6148 5206
    2618 6148 5206
    2619 5901 5089
    2620 5696 4883
    2621 5666 4853
    2622 5613 4821
    2623 5763 4933
    2624 5828 5006
    2625 5929 5073
    2626 5882 5025
    2627 5863 4980
    2628 5698 4888
    2629 5727 4848
    2630 5807 4903
    2631 5845 4898
    2632 5934 4956
    2633 5736 4475
    2634 5915 4653
    2635 6377 5234
    2636 6377 5180
    2637 5904 4573
    2638 5992 4672
    2639 5992 4672
    2640 5847 4660
    2641 5390 4161
    2642 5353 3821
    2643 5841 4076
    2644 6125 4152
    2645 6200 4196
    2646 6178 4219
    2647 6073 4183
    2648 5979 4142
    2649 5883 4062
    2650 5803 3993
    2651 5995 4161
    2652 5995 4161
    2653 6154 4283
    2654 6201 4335
    2655 6247 4397
    2656 6251 4437
    2657 6321 4498
    2658 6335 4518
    2659 6293 4501
    2660 6304 4526
    2661 6103 4377
    2662 6152 4446
    2663 6066 4360
    2664 6002 4282
    2665 6072 4353
    2666 6091 4333
    2667 6144 4417
    2668 6226 4497
    2669 6410 4648
    2670 6364 4602
    2671 6333 4590
    2672 6193 4466
    2673 6197 4509
    2674 6343 4675
    2675 6106 4519
    2676 6033 4495
    2677 6138 4679
    2678 6148 4933
    2679 6161 4967
    2680 6106 4946
    2681 6109 4953
    2682 6002 4925
    2683 6070 5022
    2684 5927 4913
    2685 6039 4996
    2686 5937 4958
    2687 5937 4958
    2688 5673 4776
    2689 5652 4693
    2690 5980 5057
    2691 6095 5116
    2692 6062 5088
    2693 6010 5067
    2694 6038 5118
    2695 5847 5009
    2696 5761 4916
    2697 5720 4850
    2698 5857 4953
    2699 6025 5145
    2700 6068 5180
    2701 5997 5115
    2702 5790 4876
    2703 5794 4894
    2704 5795 4873
    2705 5845 4898
    2706 5691 4408
    2707 5915 4653
    2708 6168 4891
    2709 6288 5062
    2710 5687 4313
    2711 5735 4322
    2712 5773 4371
    2713 5582 4285
    2714 5545 4045
    2715 5891 4172
    2716 6036 4166
    2717 6136 4147
    2718 6163 4199
    2719 6196 4231
    2720 6073 4183
    2721 5882 4073
    2722 5727 3945
    2723 5587 3850
    2724 5756 3991
    2725 5867 4109
    2726 6024 4180
    2727 6018 4219
    2728 6109 4318
    2729 6126 4364
    2730 6281 4494
    2731 6243 4485
    2732 6231 4468
    2733 6304 4526
    2734 6119 4386
    2735 6043 4334
    2736 6041 4363
    2737 5995 4351
    2738 5974 4320
    2739 6192 4461
    2740 6411 4691
    2741 6497 4738
    2742 6488 4782
    2743 6347 4626
    2744 6191 4483
    2745 6104 4377
    2746 6423 4732
    2747 6470 4771
    2748 6268 4657
    2749 6020 4534
    2750 6090 4695
    2751 6221 4877
    2752 6231 4918
    2753 6263 4913
    2754 6365 5097
    2755 6199 4951
    2756 6133 4879
    2757 6157 4939
    2758 6179 4960
    2759 6138 4936
    2760 6138 4936
    2761 6179 5029
    2762 6207 5053
    2763 6129 5060
    2764 6251 5169
    2765 6246 5196
    2766 6239 5198
    2767 5606 4637
    2768 5989 4923
    2769 6215 5154
    2770 6164 5170
    2771 6229 5245
    2772 6198 5201
    2773 6198 5201
    2774 6089 5166
    2775 6080 5143
    2776 5838 4946
    2777 5965 5031
    2778 5957 5005
    2779 5954 5003
    2780 5883 4944
    2781 5800 4925
    2782 5691 4814
    2783 5580 4709
    2784 5756 4741
    2785 5705 4413
    2786 5615 4311
    2787 5632 4266
    2788 5549 4127
    2789 5770 4426
    2790 5666 4186
    2791 5840 4298
    2792 5593 4089
    2793 5593 4198
    2794 5336 3880
    2795 5690 3990
    2796 5955 4130
    2797 6095 4170
    2798 6163 4199
    2799 6000 4136
    2800 5677 3921
    2801 5492 3788
    2802 5660 3929
    2803 6021 4254
    2804 5978 4192
    2805 5979 4192
    2806 5943 4151
    2807 5909 4146
    2808 6041 4322
    2809 6118 4431
    2810 6116 4374
    2811 6243 4485
    2812 6147 4435
    2813 6092 4385
    2814 6035 4334
    2815 6159 4439
    2816 6003 4255
    2817 5985 4316
    2818 6051 4386
    2819 6318 4643
    2820 6318 4643
    2821 6507 4803
    2822 6473 4801
    2823 6294 4628
    2824 6191 4483
    2825 6160 4623
    2826 6160 4623
    2827 6158 4685
    2828 6157 4764
    2829 6231 4918
    2830 6162 4859
    2831 6396 5066
    2832 6522 5154
    2833 6617 5191
    2834 6565 5152
    2835 6565 5152
    2836 6527 5155
    2837 6199 4951
    2838 6270 4982
    2839 6270 5041
    2840 6422 5183
    2841 6333 5099
    2842 6353 5073
    2843 6251 5028
    2844 6404 5212
    2845 6416 5284
    2846 6425 5287
    2847 6313 5188
    2848 6199 5073
    2849 5861 4839
    2850 5989 4923
    2851 6335 5212
    2852 6383 5286
    2853 6513 5432
    2854 6333 5337
    2855 6329 5272
    2856 6277 5214
    2857 6351 5292
    2858 6278 5271
    2859 6202 5177
    2860 5883 4944
    2861 5613 4769
    2862 5558 4743
    2863 4539 3289
    2864 5369 4070
    2865 5369 4070
    2866 5170 3832
    2867 5054 3645
    2868 5743 4315
    2869 5927 4398
    2870 5866 4286
    2871 5577 4011
    2872 5148 3683
    2873 5197 3628
    2874 5825 4057
    2875 6133 4212
    2876 6089 4171
    2877 5826 3979
    2878 5832 4079
    2879 5697 3960
    2880 5892 4196
    2881 5878 4154
    2882 6125 4344
    2883 6032 4237
    2884 6036 4239
    2885 5963 4210
    2886 6096 4354
    2887 6265 4551
    2888 6179 4493
    2889 6106 4424
    2890 6035 4326
    2891 6032 4338
    2892 5856 4169
    2893 5856 4169
    2894 6021 4280
    2895 5966 4259
    2896 5902 4217
    2897 5913 4250
    2898 6151 4491
    2899 5871 4270
    2900 5968 4325
    2901 6025 4392
    2902 5873 4284
    2903 6084 4430
    2904 6112 4682
    2905 6151 4755
    2906 5912 4538
    2907 5780 4449
    2908 5750 4417
    2909 5967 4633
    2910 6443 5094
    2911 6591 5264
    2912 6676 5190
    2913 6695 5212
    2914 6690 5240
    2915 6644 5201
    2916 6625 5193
    2917 6600 5181
    2918 6560 5180
    2919 6502 5193
    2920 6672 5365
    2921 6634 5295
    2922 6636 5248
    2923 6527 5196
    2924 6568 5237
    2925 6395 5111
    2926 6535 5254
    2927 6481 5305
    2928 6449 5284
    2929 6339 5186
    2930 6199 5073
    2931 6211 5061
    2932 6350 5167
    2933 6469 5269
    2934 6656 5486
    2935 6611 5449
    2936 6506 5379
    2937 6508 5370
    2938 6440 5369
    2939 6104 5056
    2940 3975 2746
    2941 3933 2750
    2942 4918 3643
    2943 5279 3979
    2944 5365 4023
    2945 5673 4306
    2946 5937 4383
    2947 5791 4167
    2948 5315 3754
    2949 4965 3419
    2950 5150 3556
    2951 5197 3628
    2952 5701 3934
    2953 5986 4080
    2954 5927 4017
    2955 5748 3897
    2956 5754 3990
    2957 6066 4297
    2958 6132 4349
    2959 6087 4309
    2960 5934 4159
    2961 5996 4206
    2962 6043 4268
    2963 6183 4442
    2964 6096 4354
    2965 6328 4577
    2966 6362 4635
    2967 6271 4550
    2968 6095 4398
    2969 6105 4401
    2970 5885 4224
    2971 6087 4373
    2972 6164 4408
    2973 6271 4525
    2974 6015 4356
    2975 6022 4394
    2976 6079 4529
    2977 5754 4230
    2978 5754 4230
    2979 5704 4121
    2980 5985 4397
    2981 6027 4419
    2982 6130 4797
    2983 6165 4840
    2984 5778 4469
    2985 6082 4776
    2986 6315 5012
    2987 6520 5261
    2988 6583 5185
    2989 6604 5177
    2990 6639 5154
    2991 6563 5101
    2992 6625 5167
    2993 6587 5142
    2994 6541 5091
    2995 6564 5138
    2996 6564 5138
    2997 6523 5164
    2998 6719 5351
    2999 6677 5354
    3000 6589 5203
    3001 6663 5276
    3002 6683 5305
    3003 6563 5214
    3004 6527 5163
    3005 6553 5311
    3006 6427 5257
    3007 6466 5267
    3008 6357 5170
    3009 6285 5093
    3010 6557 5359
    3011 6444 5235
    3012 6502 5326
    3013 3874 2739
    3014 4558 3368
    3015 5077 3811
    3016 5595 4296
    3017 5796 4428
    3018 5796 4428
    3019 5851 4357
    3020 5615 3968
    3021 5396 3785
    3022 5331 3728
    3023 5158 3546
    3024 5503 3765
    3025 5752 3960
    3026 5924 4077
    3027 5861 4063
    3028 5699 3938
    3029 5935 4205
    3030 5996 4237
    3031 5996 4237
    3032 5944 4171
    3033 5851 4102
    3034 5935 4181
    3035 6115 4359
    3036 6240 4497
    3037 6406 4641
    3038 6408 4635
    3039 6291 4571
    3040 6198 4538
    3041 6025 4264
    3042 6047 4319
    3043 6189 4410
    3044 6338 4552
    3045 6274 4519
    3046 6218 4447
    3047 6187 4466
    3048 6022 4394
    3049 6283 4669
    3050 6053 4527
    3051 6106 4536
    3052 6040 4459
    3053 6331 4693
    3054 6410 4761
    3055 6662 5172
    3056 6506 5068
    3057 6421 4993
    3058 6289 4924
    3059 6031 4733
    3060 6280 5026
    3061 6501 5095
    3062 6506 5082
    3063 6509 5097
    3064 6419 5005
    3065 6507 5044
    3066 6563 5101
    3067 6519 5072
    3068 6465 5018
    3069 6513 5060
    3070 6457 5041
    3071 6533 5174
    3072 6561 5192
    3073 6536 5136
    3074 6524 5100
    3075 6655 5224
    3076 6579 5133
    3077 6431 5005
    3078 6527 5163
    3079 6560 5212
    3080 6563 5337
    3081 6583 5345
    3082 6470 5269
    3083 6474 5266
    3084 6088 5089
    3085 4294 3113
    3086 4237 3053
    3087 4279 3122
    3088 4882 3703
    3089 4705 3623
    3090 5178 4046
    3091 5220 3986
    3092 5787 4544
    3093 5880 4520
    3094 5803 4297
    3095 5518 3902
    3096 5717 3998
    3097 5764 4058
    3098 5467 3812
    3099 5617 3878
    3100 5721 3954
    3101 5721 3954
    3102 5915 4076
    3103 5889 4074
    3104 5754 4017
    3105 5968 4233
    3106 6023 4249
    3107 6006 4229
    3108 6001 4260
    3109 6234 4444
    3110 6205 4408
    3111 6252 4478
    3112 6336 4534
    3113 6403 4647
    3114 6403 4647
    3115 6261 4542
    3116 6283 4573
    3117 6032 4336
    3118 6233 4495
    3119 6223 4440
    3120 6278 4471
    3121 6180 4402
    3122 6361 4623
    3123 6629 4919
    3124 6502 4891
    3125 6350 4691
    3126 6325 4708
    3127 6497 4892
    3128 6286 4716
    3129 6430 4791
    3130 6500 4782
    3131 6699 5172
    3132 6501 5060
    3133 6501 5060
    3134 6314 4896
    3135 6119 4791
    3136 6446 5032
    3137 6506 5087
    3138 6419 5005
    3139 6389 4983
    3140 6454 5041
    3141 6362 4949
    3142 6509 5133
    3143 6483 5122
    3144 6588 5240
    3145 6499 5133
    3146 6539 5164
    3147 6560 5131
    3148 6579 5133
    3149 6469 5018
    3150 6534 5145
    3151 6596 5223
    3152 6643 5348
    3153 6534 5213
    3154 6457 5198
    3155 6109 5039
    3156 4156 3036
    3157 3758 2630
    3158 4881 3707
    3159 5727 4528
    3160 5893 4662
    3161 5882 4596
    3162 5908 4613
    3163 5830 4437
    3164 5803 4308
    3165 5812 4146
    3166 5709 4069
    3167 5717 3998
    3168 6054 4223
    3169 5926 4172
    3170 5973 4156
    3171 5929 4169
    3172 6018 4226
    3173 5867 4065
    3174 5945 4187
    3175 6107 4375
    3176 6047 4307
    3177 5931 4174
    3178 6070 4302
    3179 6125 4339
    3180 6234 4444
    3181 6183 4364
    3182 6114 4306
    3183 6095 4325
    3184 6372 4583
    3185 6402 4675
    3186 6288 4576
    3187 6212 4494
    3188 6295 4529
    3189 6258 4492
    3190 6200 4427
    3191 6171 4443
    3192 6474 4744
    3193 6361 4623
    3194 6703 4958
    3195 6713 5039
    3196 6422 4800
    3197 6367 4711
    3198 6531 4886
    3199 6439 4771
    3200 6391 4639
    3201 6472 4771
    3202 6487 4751
    3203 6547 4836
    3204 6599 4925
    3205 6666 5002
    3206 6658 5076
    3207 6624 5155
    3208 6562 5142
    3209 6340 4977
    3210 6244 4944
    3211 6446 5056
    3212 6392 5028
    3213 6450 5129
    3214 6378 5066
    3215 6365 4979
    3216 6585 5249
    3217 6559 5163
    3218 6580 5196
    3219 6651 5251
    3220 6578 5158
    3221 6652 5245
    3222 6670 5274
    3223 6670 5274
    3224 6628 5268
    3225 6530 5168
    3226 6268 5201
    3227 6243 5195
    3228 5461 4438
    3229 5513 4405
    3230 4795 3624
    3231 4113 3085
    3232 4476 3347
    3233 5621 4436
    3234 5621 4436
    3235 5922 4618
    3236 5799 4389
    3237 5662 4235
    3238 5471 3978
    3239 5693 4171
    3240 5812 4146
    3241 6009 4223
    3242 6136 4263
    3243 6232 4322
    3244 6107 4228
    3245 5898 4035
    3246 6168 4334
    3247 6090 4271
    3248 5991 4198
    3249 6076 4317
    3250 6162 4400
    3251 6111 4358
    3252 5938 4181
    3253 6070 4302
    3254 5976 4180
    3255 6019 4173
    3256 6078 4278
    3257 6204 4452
    3258 6254 4531
    3259 6361 4609
    3260 6349 4647
    3261 6351 4622
    3262 6286 4540
    3263 6366 4637
    3264 6355 4623
    3265 6213 4484
    3266 6171 4443
    3267 6389 4636
    3268 6613 4846
    3269 6429 4680
    3270 6418 4742
    3271 6189 4564
    3272 6144 4464
    3273 6137 4441
    3274 6288 4580
    3275 6288 4580
    3276 6463 4719
    3277 6331 4602
    3278 6330 4595
    3279 6490 4750
    3280 6467 4727
    3281 6585 4889
    3282 6638 4967
    3283 6579 4937
    3284 6533 4877
    3285 6563 5010
    3286 6645 5164
    3287 6557 5093
    3288 6400 5003
    3289 6455 5070
    3290 6431 5075
    3291 6265 4908
    3292 6137 4782
    3293 6497 5194
    3294 6422 5144
    3295 6366 5044
    3296 6581 5183
    3297 6552 5151
    3298 6522 5109
    3299 6463 5056
    3300 6634 5220
    3301 6609 5234
    3302 6574 5228
    3303 6609 5293
    3304 6534 5232
    3305 6411 5366
    3306 6449 5428
    3307 6295 5293
    3308 6282 5325
    3309 4828 3644
    3310 4828 3644
    3311 4669 3443
    3312 4432 3231
    3313 4560 3461
    3314 4501 3311
    3315 5516 4298
    3316 5776 4431
    3317 5913 4476
    3318 5943 4345
    3319 5686 4127
    3320 5484 3897
    3321 5753 4054
    3322 6092 4300
    3323 6257 4366
    3324 6233 4367
    3325 6274 4368
    3326 5963 4082
    3327 5697 3890
    3328 5763 3923
    3329 5856 4050
    3330 6117 4305
    3331 5856 4100
    3332 5840 4042
    3333 5903 4168
    3334 5989 4246
    3335 5923 4154
    3336 6051 4295
    3337 5902 4130
    3338 6034 4274
    3339 6217 4401
    3340 6282 4499
    3341 6318 4569
    3342 6383 4656
    3343 6294 4580
    3344 6630 4895
    3345 6509 4787
    3346 6288 4574
    3347 6319 4604
    3348 6216 4480
    3349 6166 4473
    3350 6252 4535
    3351 6252 4535
    3352 6338 4679
    3353 6368 4710
    3354 6161 4554
    3355 6343 4704
    3356 6433 4727
    3357 6583 4827
    3358 6400 4674
    3359 6228 4526
    3360 6353 4648
    3361 6367 4624
    3362 6410 4687
    3363 6602 4895
    3364 6602 4895
    3365 6623 4942
    3366 6518 4807
    3367 6617 5074
    3368 6731 5270
    3369 6562 5127
    3370 6411 5006
    3371 6271 4903
    3372 6306 4920
    3373 6250 4886
    3374 6214 4929
    3375 6331 5049
    3376 6322 5032
    3377 6241 4993
    3378 6146 4856
    3379 6533 5206
    3380 6234 4866
    3381 6339 4926
    3382 6635 5288
    3383 6651 5294
    3384 6629 5282
    3385 6627 5308
    3386 6557 5264
    3387 6512 5241
    3388 6413 5166
    3389 6450 5391
    3390 6441 5375
    3391 6274 5183
    3392 6351 5314
    3393 6193 5159
    3394 6119 5084
    3395 6069 4993
    3396 4173 2958
    3397 4058 2836
    3398 3976 2784
    3399 4143 2943
    3400 4999 3749
    3401 5784 4476
    3402 6082 4644
    3403 5948 4475
    3404 5943 4345
    3405 6066 4368
    3406 5778 4157
    3407 5947 4214
    3408 6158 4349
    3409 6274 4373
    3410 6141 4244
    3411 5892 4059
    3412 5882 4059
    3413 6297 4454
    3414 6050 4254
    3415 6210 4344
    3416 6090 4312
    3417 5856 4100
    3418 5854 4127
    3419 6006 4252
    3420 6143 4363
    3421 6041 4259
    3422 6153 4354
    3423 6204 4455
    3424 6260 4512
    3425 6209 4422
    3426 6204 4434
    3427 6307 4581
    3428 6368 4680
    3429 6564 4858
    3430 6630 4895
    3431 6577 4839
    3432 6312 4592
    3433 6306 4568
    3434 6274 4553
    3435 6339 4637
    3436 6403 4727
    3437 6515 4805
    3438 6433 4759
    3439 6500 4839
    3440 6619 4936
    3441 6776 5054
    3442 6619 4896
    3443 6581 4814
    3444 6581 4814
    3445 6535 4810
    3446 6403 4689
    3447 6485 4767
    3448 6473 4769
    3449 6341 4621
    3450 6405 4723
    3451 6480 4838
    3452 6683 5101
    3453 6584 5055
    3454 6460 5024
    3455 6316 4961
    3456 6333 4998
    3457 6333 4998
    3458 6409 5081
    3459 6482 5170
    3460 6356 5113
    3461 6139 4922
    3462 6207 5012
    3463 6084 4891
    3464 6674 5320
    3465 6351 5027
    3466 6279 4954
    3467 6560 5172
    3468 6523 5151
    3469 6608 5239
    3470 6602 5290
    3471 6570 5273
    3472 6505 5291
    3473 6409 5223
    3474 6383 5180
    3475 6367 5180
    3476 6108 5024
    3477 6125 5040
    3478 6234 5188
    3479 6272 5248
    3480 6272 5248
    3481 6159 5163
    3482 6129 5091
    3483 6186 5110
    3484 6333 5307
    3485 3576 2436
    3486 4075 2917
    3487 4977 3822
    3488 5867 4557
    3489 6082 4644
    3490 6194 4561
    3491 6170 4406
    3492 6152 4309
    3493 6191 4388
    3494 6213 4396
    3495 6160 4249
    3496 6062 4194
    3497 5971 4141
    3498 5971 4141
    3499 6041 4243
    3500 6319 4432
    3501 6308 4454
    3502 6210 4344
    3503 6274 4437
    3504 6272 4495
    3505 6307 4516
    3506 6157 4357
    3507 6157 4356
    3508 6171 4391
    3509 6252 4486
    3510 6284 4514
    3511 6284 4514
    3512 6226 4463
    3513 6206 4442
    3514 6295 4597
    3515 6368 4680
    3516 6476 4802
    3517 6362 4625
    3518 6211 4517
    3519 6320 4617
    3520 6361 4644
    3521 6283 4560
    3522 6279 4505
    3523 6433 4735
    3524 6412 4722
    3525 6441 4742
    3526 6669 5017
    3527 6776 5054
    3528 6650 4903
    3529 6625 4857
    3530 6703 4913
    3531 6520 4803
    3532 6339 4657
    3533 6382 4750
    3534 6295 4594
    3535 6455 4784
    3536 6421 4811
    3537 6584 5055
    3538 6529 5084
    3539 5966 4773
    3540 6548 5228
    3541 6444 5134
    3542 6293 5009
    3543 6234 4964
    3544 6662 5310
    3545 6568 5227
    3546 6556 5248
    3547 6522 5257
    3548 6440 5211
    3549 6345 5144
    3550 6410 5200
    3551 6131 5041
    3552 6423 5370
    3553 6447 5401
    3554 6510 5421
    3555 6402 5367
    3556 6425 5365
    3557 6530 5515
    3558 6491 5497
    3559 6371 5391
    3560 4989 3858
    3561 5793 4581
    3562 6230 4936
    3563 6263 4688
    3564 6155 4383
    3565 6214 4436
    3566 6263 4399
    3567 6290 4419
    3568 6264 4426
    3569 6029 4126
    3570 6005 4129
    3571 5881 4049
    3572 5930 4104
    3573 6173 4327
    3574 6342 4407
    3575 6201 4333
    3576 6236 4397
    3577 6214 4384
    3578 6227 4384
    3579 6227 4384
    3580 6244 4446
    3581 6270 4490
    3582 6282 4494
    3583 6310 4505
    3584 6298 4516
    3585 6293 4490
    3586 6166 4387
    3587 6109 4344
    3588 5987 4279
    3589 6294 4618
    3590 5971 4306
    3591 6184 4514
    3592 6184 4514
    3593 6412 4710
    3594 6405 4656
    3595 6342 4644
    3596 6304 4592
    3597 6508 4811
    3598 6454 4756
    3599 6678 4980
    3600 6730 5011
    3601 6563 4859
    3602 6638 4888
    3603 6760 4969
    3604 6776 4983
    3605 6596 4898
    3606 6516 4871
    3607 6499 4880
    3608 6549 4988
    3609 6561 5071
    3610 5935 4708
    3611 5486 4363
    3612 5486 4363
    3613 6362 5093
    3614 6396 5158
    3615 6501 5218
    3616 6501 5218
    3617 6548 5256
    3618 6544 5293
    3619 6265 5041
    3620 6348 5177
    3621 6556 5400
    3622 6572 5454
    3623 6496 5382
    3624 6544 5471
    3625 6425 5365
    3626 6563 5504
    3627 6589 5563
    3628 6296 5232
    3629 4991 3851
    3630 5697 4507
    3631 6125 4805
    3632 6275 4769
    3633 6030 4313
    3634 6161 4300
    3635 6175 4293
    3636 6249 4376
    3637 6108 4227
    3638 6014 4142
    3639 5943 4117
    3640 6108 4318
    3641 6190 4365
    3642 6190 4365
    3643 6155 4268
    3644 6135 4276
    3645 6149 4313
    3646 6011 4168
    3647 6026 4199
    3648 6067 4256
    3649 6299 4516
    3650 6384 4592
    3651 6296 4487
    3652 6215 4410
    3653 6230 4409
    3654 6178 4355
    3655 6178 4355
    3656 5927 4181
    3657 5906 4199
    3658 6021 4392
    3659 5863 4264
    3660 6141 4492
    3661 6400 4692
    3662 6337 4586
    3663 6381 4643
    3664 6455 4714
    3665 6583 4842
    3666 6585 4878
    3667 6687 4947
    3668 6682 4952
    3669 6653 4898
    3670 6595 4879
    3671 6659 4924
    3672 6660 4851
    3673 6595 4808
    3674 6717 4912
    3675 6559 4756
    3676 6473 4798
    3677 6533 4875
    3678 6590 4965
    3679 6474 5032
    3680 6249 4918
    3681 5956 4709
    3682 5478 4318
    3683 6351 5117
    3684 6317 5133
    3685 6413 5202
    3686 6466 5204
    3687 6519 5247
    3688 6319 5146
    3689 6368 5220
    3690 6504 5328
    3691 6496 5382
    3692 6472 5410
    3693 6329 5226
    3694 6001 4860
    3695 5934 4806
    3696 5332 4029
    3697 5865 4513
    3698 5819 4333
    3699 5911 4267
    3700 5998 4271
    3701 5989 4171
    3702 5989 4171
    3703 5961 4075
    3704 5822 3931
    3705 6087 4202
    3706 5943 4117
    3707 6120 4280
    3708 6236 4365
    3709 6132 4250
    3710 6075 4202
    3711 6010 4163
    3712 5962 4118
    3713 6114 4310
    3714 6207 4429
    3715 6251 4443
    3716 6278 4491
    3717 6309 4511
    3718 6202 4388
    3719 6215 4410
    3720 6262 4495
    3721 6176 4355
    3722 6262 4460
    3723 6233 4474
    3724 6213 4492
    3725 6056 4460
    3726 6318 4681
    3727 6430 4737
    3728 6435 4715
    3729 6377 4655
    3730 6377 4655
    3731 6431 4698
    3732 6461 4739
    3733 6519 4779
    3734 6561 4837
    3735 6631 4892
    3736 6594 4816
    3737 6526 4774
    3738 6634 4878
    3739 6540 4784
    3740 6388 4623
    3741 6364 4618
    3742 6419 4633
    3743 6419 4633
    3744 6510 4787
    3745 6629 4878
    3746 6211 4871
    3747 5978 4722
    3748 5815 4609
    3749 6297 5106
    3750 6247 5116
    3751 5820 4716
    3752 6109 4993
    3753 6366 5217
    3754 6389 5282
    3755 6224 5174
    3756 5813 4678
    3757 4962 4038
    3758 5036 3704
    3759 5527 4104
    3760 5582 4065
    3761 5851 4256
    3762 5978 4278
    3763 5933 4184
    3764 5717 3946
    3765 5656 3824
    3766 5969 4097
    3767 5926 4071
    3768 6055 4254
    3769 6185 4310
    3770 6156 4281
    3771 6172 4283
    3772 6113 4237
    3773 6138 4283
    3774 5952 4131
    3775 6180 4391
    3776 6257 4446
    3777 6137 4328
    3778 6148 4345
    3779 6125 4335
    3780 6252 4489
    3781 6427 4638
    3782 6474 4630
    3783 6320 4516
    3784 6188 4416
    3785 6214 4403
    3786 6138 4376
    3787 6160 4492
    3788 6326 4672
    3789 6453 4761
    3790 6331 4622
    3791 6325 4625
    3792 6371 4623
    3793 6410 4680
    3794 6527 4809
    3795 6495 4790
    3796 6518 4782
    3797 6517 4773
    3798 6500 4720
    3799 6500 4720
    3800 6608 4831
    3801 6517 4709
    3802 6357 4587
    3803 6492 4792
    3804 6556 4856
    3805 6667 4959
    3806 6846 5188
    3807 6664 4990
    3808 6670 5055
    3809 6438 5055
    3810 6120 4823
    3811 5820 4716
    3812 5829 4722
    3813 6313 5188
    3814 4713 3667
    3815 4631 3555
    3816 4470 3560
    3817 4137 3027
    3818 4758 3424
    3819 5032 3596
    3820 5362 3854
    3821 5537 3962
    3822 5822 4140
    3823 5662 3947
    3824 5684 3938
    3825 5744 3947
    3826 5744 3947
    3827 5905 4078
    3828 5895 4055
    3829 6046 4206
    3830 6141 4309
    3831 6161 4283
    3832 6213 4332
    3833 6244 4354
    3834 6257 4421
    3835 6217 4410
    3836 6159 4332
    3837 6169 4396
    3838 6070 4297
    3839 6244 4508
    3840 6132 4373
    3841 6291 4544
    3842 6456 4645
    3843 6362 4488
    3844 6348 4496
    3845 6254 4444
    3846 6177 4439
    3847 6124 4433
    3848 6111 4476
    3849 6222 4594
    3850 6178 4496
    3851 6287 4594
    3852 6376 4644
    3853 6440 4751
    3854 6440 4751
    3855 6540 4802
    3856 6548 4843
    3857 6570 4858
    3858 6587 4841
    3859 6535 4794
    3860 6616 4843
    3861 6494 4723
    3862 6618 4839
    3863 6760 5045
    3864 6834 5146
    3865 6912 5191
    3866 6928 5258
    3867 6928 5258
    3868 6872 5255
    3869 6854 5282
    3870 6774 5229
    3871 6692 5219
    3872 6213 4961
    3873 4262 3152
    3874 4596 3319
    3875 4855 3475
    3876 4890 3432
    3877 5131 3625
    3878 5190 3611
    3879 5499 3848
    3880 5662 3947
    3881 5567 3824
    3882 5716 3958
    3883 5700 3916
    3884 5793 3952
    3885 5851 4050
    3886 6048 4240
    3887 5938 4169
    3888 6178 4279
    3889 6085 4179
    3890 6165 4304
    3891 6165 4304
    3892 6124 4302
    3893 6159 4332
    3894 6235 4487
    3895 6184 4415
    3896 6207 4447
    3897 6203 4424
    3898 6084 4296
    3899 6197 4380
    3900 6255 4454
    3901 6213 4439
    3902 6208 4448
    3903 6173 4448
    3904 6173 4448
    3905 6148 4571
    3906 6222 4594
    3907 6119 4442
    3908 6144 4475
    3909 6261 4560
    3910 6339 4610
    3911 6393 4679
    3912 6437 4739
    3913 6397 4712
    3914 6547 4791
    3915 6627 4846
    3916 6639 4835
    3917 6653 4847
    3918 6660 4875
    3919 6618 4839
    3920 6887 5103
    3921 6943 5182
    3922 6850 5113
    3923 6789 5057
    3924 6788 5136
    3925 6856 5305
    3926 6815 5331
    3927 6710 5277
    3928 6113 4913
    3929 5986 4881
    3930 4130 3181
    3931 4136 3275
    3932 3813 2923
    3933 3944 3012
    3934 3787 2850
    3935 3885 2942
    3936 4132 3042
    3937 4389 3113
    3938 4786 3438
    3939 4789 3375
    3940 4748 3216
    3941 5158 3569
    3942 5124 3490
    3943 5430 3739
    3944 5534 3828
    3945 5642 3894
    3946 5689 3950
    3947 5689 3918
    3948 5689 3918
    3949 5912 4198
    3950 6005 4415
    3951 5935 4180
    3952 5953 4167
    3953 5939 4069
    3954 6022 4178
    3955 6122 4336
    3956 6341 4594
    3957 6187 4441
    3958 5992 4288
    3959 5956 4272
    3960 5977 4233
    3961 5977 4233
    3962 6187 4429
    3963 6288 4478
    3964 6311 4512
    3965 6244 4449
    3966 6171 4448
    3967 6140 4523
    3968 6168 4584
    3969 6166 4534
    3970 6322 4669
    3971 6360 4671
    3972 6330 4588
    3973 6379 4651
    3974 6414 4682
    3975 6441 4758
    3976 6403 4734
    3977 6467 4757
    3978 6473 4703
    3979 6529 4731
    3980 6653 4847
    3981 6781 5005
    3982 6873 5107
    3983 6872 5143
    3984 6844 5165
    3985 6822 5164
    3986 6779 5170
    3987 6741 5118
    3988 6886 5330
    3989 6751 5209
    3990 6723 5298
    3991 6382 5123
    3992 6113 4913
    3993 6089 4929
    3994 5922 4819
    3995 4707 3484
    3996 4207 3141
    3997 4217 3281
    3998 4053 3149
    3999 3969 3041
    4000 3835 2871
    4001 3916 2841
    4002 4216 2977
    4003 4557 3205
    4004 4779 3362
    4005 4662 3214
    4006 4887 3384
    4007 4935 3393
    4008 5041 3452
    4009 5261 3600
    4010 5405 3704
    4011 5612 3880
    4012 5719 3987
    4013 5639 3931
    4014 5728 4132
    4015 5963 4351
    4016 6223 4609
    4017 6158 4416
    4018 5985 4226
    4019 6184 4425
    4020 6166 4389
    4021 6166 4389
    4022 6343 4563
    4023 6151 4406
    4024 6154 4488
    4025 6057 4387
    4026 5926 4201
    4027 6310 4561
    4028 6382 4593
    4029 6432 4644
    4030 6246 4503
    4031 6103 4369
    4032 6078 4443
    4033 6146 4564
    4034 6146 4564
    4035 6349 4758
    4036 6374 4731
    4037 6286 4581
    4038 6207 4481
    4039 6283 4525
    4040 6363 4633
    4041 6395 4702
    4042 6277 4582
    4043 6372 4669
    4044 6364 4617
    4045 6577 4881
    4046 6728 4944
    4047 6795 5032
    4048 6915 5168
    4049 6892 5162
    4050 6773 5072
    4051 6605 4935
    4052 6670 5024
    4053 6608 5029
    4054 6633 5075
    4055 6515 5026
    4056 6622 5240
    4057 6537 5288
    4058 6386 5137
    4059 6132 4932
    4060 6178 4997
    4061 6126 5004
    4062 6126 5004
    4063 5459 4469
    4064 4005 3057
    4065 3802 2827
    4066 3940 2891
    4067 3940 2891
    4068 4301 3065
    4069 4557 3205
    4070 4786 3354
    4071 4841 3391
    4072 4959 3462
    4073 5038 3472
    4074 5133 3514
    4075 5321 3661
    4076 5588 3892
    4077 5727 4002
    4078 5764 4025
    4079 5860 4214
    4080 5972 4414
    4081 5933 4401
    4082 6223 4609
    4083 6268 4647
    4084 6288 4507
    4085 6426 4622
    4086 6341 4578
    4087 6269 4522
    4088 6114 4405
    4089 6258 4580
    4090 6217 4551
    4091 6322 4644
    4092 6401 4656
    4093 6475 4672
    4094 6501 4690
    4095 6432 4644
    4096 6410 4598
    4097 6214 4526
    4098 6237 4568
    4099 6282 4695
    4100 6211 4574
    4101 6370 4726
    4102 6309 4636
    4103 6271 4567
    4104 6375 4636
    4105 6417 4694
    4106 6447 4689
    4107 6338 4626
    4108 6277 4582
    4109 6392 4717
    4110 6424 4765
    4111 6648 4960
    4112 6578 4839
    4113 6714 5001
    4114 6795 5090
    4115 6660 4967
    4116 6632 5024
    4117 6814 5245
    4118 6784 5221
    4119 6771 5193
    4120 6574 5058
    4121 6515 5026
    4122 6512 5067
    4123 6658 5380
    4124 6600 5307
    4125 6281 5051
    4126 6384 5197
    4127 6275 4889
    4128 4132 3030
    4129 4518 3279
    4130 4791 3410
    4131 4825 3361
    4132 4961 3444
    4133 5018 3488
    4134 5038 3485
    4135 5059 3416
    4136 5059 3416
    4137 5331 3658
    4138 5571 3837
    4139 5697 3950
    4140 5751 4019
    4141 5834 4163
    4142 5972 4414
    4143 6052 4445
    4144 6304 4668
    4145 6439 4811
    4146 6303 4578
    4147 6284 4547
    4148 6250 4484
    4149 5981 4283
    4150 6056 4360
    4151 5978 4292
    4152 6287 4596
    4153 6412 4712
    4154 6454 4728
    4155 6475 4672
    4156 6357 4523
    4157 6329 4488
    4158 6282 4514
    4159 6241 4516
    4160 6232 4546
    4161 5950 4356
    4162 6005 4411
    4163 6156 4510
    4164 6363 4713
    4165 6398 4680
    4166 6418 4656
    4167 6469 4693
    4168 6447 4689
    4169 6530 4784
    4170 6504 4842
    4171 6682 5022
    4172 6616 4913
    4173 6667 4982
    4174 6809 5131
    4175 6709 4992
    4176 6619 4930
    4177 6619 4930
    4178 6708 5035
    4179 6832 5165
    4180 6876 5230
    4181 6771 5193
    4182 6621 5081
    4183 6407 4920
    4184 6548 5122
    4185 6657 5301
    4186 6619 5254
    4187 6620 5354
    4188 6640 5367
    4189 6514 5305
    4190 6123 4803
    4191 6258 4927
    4192 6326 4912
    4193 4860 3520
    4194 5038 3601
    4195 5038 3601
    4196 4938 3461
    4197 5014 3468
    4198 5045 3481
    4199 5121 3531
    4200 5079 3449
    4201 5384 3715
    4202 5592 3892
    4203 5783 4068
    4204 5802 4092
    4205 5856 4159
    4206 5985 4374
    4207 6104 4490
    4208 6388 4768
    4209 6273 4641
    4210 6479 4868
    4211 6457 4867
    4212 6425 4821
    4213 6259 4530
    4214 6182 4506
    4215 5966 4324
    4216 6314 4687
    4217 6387 4730
    4218 6486 4800
    4219 6470 4756
    4220 6304 4515
    4221 6305 4537
    4222 6294 4482
    4223 6392 4612
    4224 6392 4648
    4225 6229 4510
    4226 6205 4580
    4227 6350 4741
    4228 6395 4748
    4229 6516 4818
    4230 6464 4728
    4231 6459 4681
    4232 6479 4657
    4233 6565 4807
    4234 6556 4794
    4235 6466 4781
    4236 6466 4781
    4237 6514 4829
    4238 6642 4976
    4239 6646 4920
    4240 6497 4820
    4241 6553 4865
    4242 6533 4874
    4243 6657 5010
    4244 6765 5108
    4245 6736 5132
    4246 6714 5162
    4247 6709 5184
    4248 6584 5140
    4249 6584 5140
    4250 6552 5134
    4251 6463 5017
    4252 6600 5214
    4253 6493 5115
    4254 6358 5079
    4255 6273 4945
    4256 6497 5145
    4257 6612 5218
    4258 6490 5110
    4259 6557 5128
    4260 6565 5067
    4261 5034 3619
    4262 5149 3651
    4263 5203 3638
    4264 5100 3519
    4265 5207 3609
    4266 5278 3654
    4267 5610 3908
    4268 5602 3909
    4269 5815 4099
    4270 5718 4008
    4271 5783 4068
    4272 5825 4119
    4273 5848 4181
    4274 6025 4399
    4275 6070 4442
    4276 6294 4627
    4277 6591 4949
    4278 6654 4994
    4279 6668 5107
    4280 6428 4871
    4281 6307 4641
    4282 6536 4879
    4283 6490 4843
    4284 6314 4687
    4285 6531 4861
    4286 6514 4835
    4287 6534 4836
    4288 6398 4680
    4289 6478 4768
    4290 6406 4638
    4291 6307 4550
    4292 6127 4411
    4293 6310 4660
    4294 6415 4730
    4295 6305 4607
    4296 6434 4695
    4297 6516 4818
    4298 6518 4760
    4299 6494 4709
    4300 6412 4590
    4301 6487 4682
    4302 6452 4668
    4303 6362 4677
    4304 6400 4772
    4305 6680 5017
    4306 6576 4932
    4307 6512 4899
    4308 6532 4885
    4309 6508 4844
    4310 6641 5024
    4311 6641 5024
    4312 6759 5159
    4313 6756 5160
    4314 6707 5139
    4315 6615 5052
    4316 6672 5139
    4317 6538 5016
    4318 6414 4942
    4319 6432 4972
    4320 6422 5002
    4321 6333 4936
    4322 6226 4916
    4323 6356 5104
    4324 6476 5172
    4325 6478 5079
    4326 6502 5066
    4327 6542 5088
    4328 6497 5014
    4329 6417 4899
    4330 6411 4912
    4331 6349 4810
    4332 5308 3683
    4333 5381 3719
    4334 5261 3648
    4335 5411 3743
    4336 5509 3831
    4337 5771 4010
    4338 5825 4095
    4339 5815 4099
    4340 5906 4160
    4341 5886 4176
    4342 6033 4363
    4343 5999 4332
    4344 6283 4663
    4345 6080 4462
    4346 6240 4527
    4347 6496 4810
    4348 6496 4810
    4349 6572 4934
    4350 6477 4896
    4351 6339 4704
    4352 6536 4879
    4353 6693 5050
    4354 6593 4936
    4355 6584 4890
    4356 6522 4808
    4357 6498 4771
    4358 6546 4825
    4359 6435 4724
    4360 6309 4543
    4361 6309 4543
    4362 6161 4470
    4363 6344 4684
    4364 6364 4693
    4365 6305 4607
    4366 6335 4664
    4367 6394 4626
    4368 6411 4631
    4369 6509 4703
    4370 6500 4670
    4371 6426 4674
    4372 6430 4703
    4373 6482 4849
    4374 6559 4978
    4375 6597 4953
    4376 6598 4932
    4377 6560 4924
    4378 6532 4885
    4379 6491 4819
    4380 6620 5008
    4381 6789 5187
    4382 6749 5130
    4383 6626 5064
    4384 6654 5152
    4385 6640 5117
    4386 6516 4985
    4387 6578 5060
    4388 6509 5016
    4389 6593 5122
    4390 6517 5093
    4391 6333 4936
    4392 6426 5051
    4393 6414 5091
    4394 6521 5182
    4395 6489 5055
    4396 6477 5047
    4397 6531 5114
    4398 6518 5041
    4399 6518 5041
    4400 6457 4964
    4401 6411 4912
    4402 6332 4837
    4403 6318 4787
    4404 5341 3693
    4405 5420 3747
    4406 5469 3741
    4407 5727 3954
    4408 5850 4070
    4409 6017 4227
    4410 6183 4370
    4411 6065 4305
    4412 6079 4313
    4413 5932 4191
    4414 6193 4512
    4415 6203 4520
    4416 6465 4789
    4417 6445 4847
    4418 6453 4803
    4419 6520 4808
    4420 6741 5055
    4421 6619 5029
    4422 6330 4693
    4423 6225 4545
    4424 6579 4891
    4425 6327 4641
    4426 6335 4615
    4427 6335 4615
    4428 6458 4737
    4429 6479 4766
    4430 6523 4806
    4431 6474 4776
    4432 6322 4589
    4433 6286 4604
    4434 6275 4625
    4435 6497 4849
    4436 6618 4956
    4437 6495 4858
    4438 6447 4776
    4439 6382 4632
    4440 6440 4636
    4441 6519 4689
    4442 6416 4573
    4443 6424 4649
    4444 6427 4735
    4445 6362 4687
    4446 6447 4832
    4447 6442 4808
    4448 6447 4795
    4449 6553 4950
    4450 6621 4971
    4451 6648 5004
    4452 6738 5132
    4453 6632 5050
    4454 6638 5009
    4455 6613 5077
    4456 6753 5259
    4457 6728 5233
    4458 6584 5069
    4459 6614 5096
    4460 6661 5145
    4461 6665 5219
    4462 6473 4976
    4463 6506 5070
    4464 6602 5173
    4465 6468 5066
    4466 6301 4890
    4467 6352 4987
    4468 6352 4987
    4469 6517 5128
    4470 6548 5144
    4471 6549 5112
    4472 6712 5289
    4473 6653 5212
    4474 6570 5121
    4475 6650 5181
    4476 6535 5063
    4477 6581 5091
    4478 6520 5017
    4479 6512 5005
    4480 6401 4937
    4481 5459 3878
    4482 5622 3922
    4483 5977 4196
    4484 5961 4124
    4485 5727 3954
    4486 6018 4192
    4487 6115 4294
    4488 6248 4397
    4489 6171 4383
    4490 6190 4437
    4491 6398 4684
    4492 6627 4934
    4493 6684 5033
    4494 6724 5035
    4495 6709 5016
    4496 6686 4929
    4497 6701 4927
    4498 6662 4991
    4499 6662 4991
    4500 6349 4711
    4501 6138 4489
    4502 6117 4454
    4503 6199 4573
    4504 6084 4427
    4505 6186 4563
    4506 6277 4579
    4507 6306 4613
    4508 6398 4701
    4509 6419 4786
    4510 6383 4687
    4511 6428 4820
    4512 6428 4820
    4513 6561 4922
    4514 6563 4880
    4515 6628 4919
    4516 6572 4870
    4517 6335 4626
    4518 6376 4603
    4519 6327 4500
    4520 6351 4565
    4521 6441 4697
    4522 6398 4709
    4523 6317 4730
    4524 6414 4837
    4525 6461 4875
    4526 6399 4771
    4527 6562 4933
    4528 6731 5091
    4529 6673 5033
    4530 6723 5099
    4531 6708 5122
    4532 6644 5122
    4533 6688 5215
    4534 6678 5191
    4535 6627 5137
    4536 6706 5199
    4537 6664 5165
    4538 6627 5100
    4539 6649 5149
    4540 6480 5014
    4541 6494 5009
    4542 6676 5199
    4543 6532 5065
    4544 6429 4963
    4545 6391 5013
    4546 6674 5281
    4547 6742 5343
    4548 6804 5353
    4549 6845 5397
    4550 6836 5382
    4551 6826 5353
    4552 6688 5192
    4553 6688 5192
    4554 6606 5098
    4555 6566 5064
    4556 6649 5113
    4557 6636 5099
    4558 6510 5023
    4559 6542 5092
    4560 6401 4999
    4561 5695 4039
    4562 5938 4169
    4563 5915 4151
    4564 5977 4196
    4565 6228 4347
    4566 6212 4337
    4567 6350 4430
    4568 6342 4446
    4569 6401 4537
    4570 6292 4567
    4571 6345 4660
    4572 6271 4547
    4573 6271 4547
    4574 6653 4956
    4575 6653 4948
    4576 6637 4874
    4577 6686 4929
    4578 6585 4794
    4579 6620 4908
    4580 6416 4729
    4581 6378 4704
    4582 6284 4600
    4583 6325 4659
    4584 6421 4714
    4585 6433 4739
    4586 6504 4807
    4587 6326 4687
    4588 6406 4739
    4589 6408 4755
    4590 6419 4786
    4591 6373 4772
    4592 6406 4712
    4593 6561 4880
    4594 6579 4922
    4595 6617 5010
    4596 6638 4986
    4597 6700 5046
    4598 6438 4724
    4599 6464 4736
    4600 6442 4684
    4601 6442 4684
    4602 6428 4678
    4603 6398 4709
    4604 6252 4619
    4605 6386 4817
    4606 6521 4946
    4607 6511 4886
    4608 6558 4939
    4609 6676 5025
    4610 6647 5022
    4611 6716 5114
    4612 6534 4950
    4613 6677 5205
    4614 6634 5148
    4615 6627 5137
    4616 6781 5288
    4617 6800 5334
    4618 6730 5228
    4619 6563 5037
    4620 6650 5126
    4621 6677 5144
    4622 6659 5149
    4623 6545 5059
    4624 6646 5159
    4625 6642 5168
    4626 6729 5270
    4627 6825 5395
    4628 6804 5353
    4629 6855 5382
    4630 6894 5385
    4631 6812 5306
    4632 6708 5213
    4633 6642 5107
    4634 6548 5068
    4635 6580 5095
    4636 6606 5044
    4637 6600 5081
    4638 6565 5106
    4639 6513 5110
    4640 6555 5119
    4641 6419 5041
    4642 6136 4365
    4643 6090 4306
    4644 6173 4356
    4645 6367 4515
    4646 6400 4472
    4647 6545 4594
    4648 6511 4561
    4649 6511 4561
    4650 6408 4481
    4651 6507 4623
    4652 6433 4644
    4653 6252 4539
    4654 6044 4375
    4655 6288 4654
    4656 6655 5024
    4657 6536 4816
    4658 6602 4880
    4659 6564 4797
    4660 6515 4786
    4661 6382 4670
    4662 6323 4644
    4663 6355 4673
    4664 6316 4630
    4665 6326 4633
    4666 6402 4688
    4667 6445 4714
    4668 6458 4737
    4669 6558 4840
    4670 6500 4847
    4671 6552 4923
    4672 6375 4787
    4673 6024 4388
    4674 6396 4715
    4675 6479 4789
    4676 6552 4861
    4677 6649 5012
    4678 6798 5231
    4679 6763 5159
    4680 6591 4905
    4681 6593 4910
    4682 6552 4746
    4683 6568 4785
    4684 6506 4716
    4685 6233 4543
    4686 6263 4674
    4687 6431 4823
    4688 6339 4685
    4689 6550 4922
    4690 6550 4922
    4691 6643 5006
    4692 6578 4941
    4693 6633 5024
    4694 6401 4823
    4695 6669 5141
    4696 6693 5118
    4697 6819 5299
    4698 6827 5302
    4699 6809 5265
    4700 6727 5166
    4701 6610 5033
    4702 6610 5033
    4703 6619 5053
    4704 6669 5107
    4705 6700 5198
    4706 6708 5169
    4707 6625 5153
    4708 6729 5270
    4709 6580 5113
    4710 6806 5359
    4711 6817 5353
    4712 6813 5336
    4713 6789 5303
    4714 6699 5178
    4715 6693 5167
    4716 6646 5116
    4717 6715 5217
    4718 6609 5118
    4719 6636 5128
    4720 6594 5130
    4721 6513 5110
    4722 6380 4995
    4723 6269 4901
    4724 6199 4858
    4725 6090 4292
    4726 6274 4467
    4727 6274 4467
    4728 6289 4483
    4729 6413 4601
    4730 6469 4584
    4731 6554 4644
    4732 6594 4645
    4733 6584 4671
    4734 6555 4695
    4735 6507 4743
    4736 6257 4595
    4737 6365 4809
    4738 6371 4798
    4739 6630 5019
    4740 6630 5019
    4741 6673 5002
    4742 6674 4955
    4743 6671 4938
    4744 6543 4805
    4745 6364 4644
    4746 6271 4575
    4747 6298 4621
    4748 6277 4591
    4749 6416 4726
    4750 6383 4696
    4751 6371 4705
    4752 6322 4630
    4753 6407 4727
    4754 6532 4830
    4755 6461 4820
    4756 6354 4746
    4757 6161 4560
    4758 6184 4504
    4759 6296 4622
    4760 6541 4862
    4761 6558 4931
    4762 6585 5024
    4763 6466 4905
    4764 6551 4973
    4765 6606 5044
    4766 6539 4867
    4767 6513 4739
    4768 6513 4739
    4769 6338 4628
    4770 6316 4673
    4771 6288 4718
    4772 6183 4571
    4773 6480 4853
    4774 6568 4955
    4775 6554 4937
    4776 6469 4874
    4777 6340 4819
    4778 6665 5227
    4779 6629 5120
    4780 6635 5072
    4781 6635 5072
    4782 6741 5206
    4783 6827 5313
    4784 6813 5290
    4785 6768 5233
    4786 6616 5001
    4787 6660 5059
    4788 6660 5103
    4789 6735 5185
    4790 6717 5172
    4791 6661 5146
    4792 6533 5057
    4793 6421 5016
    4794 6506 5093
    4795 6548 5055
    4796 6658 5148
    4797 6810 5330
    4798 6797 5321
    4799 6781 5279
    4800 6812 5305
    4801 6792 5267
    4802 6775 5227
    4803 6694 5199
    4804 6604 5129
    4805 6559 5114
    4806 6453 5077
    4807 6279 4951
    4808 6286 5000
    4809 6154 4335
    4810 6361 4560
    4811 6256 4516
    4812 6393 4605
    4813 6219 4432
    4814 6448 4563
    4815 6405 4672
    4816 6507 4686
    4817 6572 4887
    4818 6610 4978
    4819 6582 5008
    4820 6468 4909
    4821 6545 4989
    4822 6385 4822
    4823 6425 4767
    4824 6275 4636
    4825 6235 4547
    4826 6164 4538
    4827 6294 4643
    4828 6307 4686
    4829 6235 4591
    4830 6235 4591
    4831 6299 4604
    4832 6287 4616
    4833 6305 4650
    4834 6322 4630
    4835 6208 4524
    4836 6415 4773
    4837 6385 4756
    4838 6198 4592
    4839 6171 4574
    4840 6181 4525
    4841 6232 4612
    4842 6403 4773
    4843 6403 4773
    4844 6462 4892
    4845 6417 4863
    4846 6537 4991
    4847 6606 5044
    4848 6614 5036
    4849 6566 4863
    4850 6416 4729
    4851 6352 4669
    4852 6453 4853
    4853 6415 4861
    4854 6490 4905
    4855 6536 4924
    4856 6556 4957
    4857 6599 5008
    4858 6515 4935
    4859 6394 4886
    4860 6665 5227
    4861 6827 5324
    4862 6746 5217
    4863 6737 5191
    4864 6781 5222
    4865 6762 5163
    4866 6806 5245
    4867 6762 5175
    4868 6696 5096
    4869 6785 5156
    4870 6820 5247
    4871 6820 5247
    4872 6793 5253
    4873 6687 5171
    4874 6567 5079
    4875 6594 5179
    4876 6705 5291
    4877 6586 5110
    4878 6633 5113
    4879 6686 5171
    4880 6664 5165
    4881 6785 5247
    4882 6668 5139
    4883 6662 5131
    4884 6662 5131
    4885 6687 5168
    4886 6483 5035
    4887 6317 4643
    4888 6284 4531
    4889 6277 4593
    4890 6249 4580
    4891 5975 4339
    4892 6163 4435
    4893 6374 4732
    4894 6549 4921
    4895 6435 4836
    4896 6466 4896
    4897 6273 4745
    4898 5963 4474
    4899 6001 4457
    4900 5843 4301
    4901 6216 4664
    4902 6014 4477
    4903 6157 4577
    4904 6341 4791
    4905 6408 4819
    4906 6342 4738
    4907 6239 4637
    4908 6120 4551
    4909 6387 4846
    4910 6342 4744
    4911 6399 4869
    4912 6040 4417
    4913 6262 4665
    4914 6344 4746
    4915 6344 4746
    4916 6276 4673
    4917 6325 4799
    4918 6120 4494
    4919 6300 4695
    4920 6287 4676
    4921 6136 4566
    4922 6226 4680
    4923 6368 4791
    4924 6421 4882
    4925 6551 5006
    4926 6395 4866
    4927 6424 4822
    4928 6386 4832
    4929 6318 4700
    4930 6453 4929
    4931 6421 4892
    4932 6454 5008
    4933 6508 4941
    4934 6498 4945
    4935 6501 4891
    4936 6388 4822
    4937 6677 5171
    4938 6826 5302
    4939 6774 5227
    4940 6806 5264
    4941 6813 5304
    4942 6742 5217
    4943 6742 5217
    4944 6796 5208
    4945 6784 5173
    4946 6743 5124
    4947 6761 5131
    4948 6770 5172
    4949 6799 5269
    4950 6712 5208
    4951 6557 5042
    4952 6487 5030
    4953 6696 5231
    4954 6643 5172
    4955 6626 5094
    4956 6626 5094
    4957 6622 5127
    4958 6727 5243
    4959 6706 5225
    4960 6711 5235
    4961 6676 5174
    4962 6654 5154
    4963 5905 4148
    4964 6202 4507
    4965 6046 4372
    4966 6010 4339
    4967 5624 3990
    4968 5820 4225
    4969 5627 4042
    4970 6062 4414
    4971 5998 4409
    4972 6125 4559
    4973 6435 4836
    4974 6186 4631
    4975 6182 4720
    4976 5935 4548
    4977 5414 4007
    4978 5388 3897
    4979 6165 4672
    4980 6330 4789
    4981 6424 4876
    4982 6398 4832
    4983 6350 4768
    4984 6279 4733
    4985 6161 4628
    4986 6475 4926
    4987 6475 4926
    4988 6634 5108
    4989 6751 5182
    4990 6479 4943
    4991 6592 5033
    4992 6528 4952
    4993 6647 5109
    4994 6555 4987
    4995 6690 5136
    4996 6368 4769
    4997 6414 4795
    4998 6387 4821
    4999 6352 4868
    5000 6352 4868
    5001 6316 4769
    5002 6548 5034
    5003 6511 5007
    5004 6182 4693
    5005 6141 4631
    5006 6153 4645
    5007 6287 4784
    5008 6178 4697
    5009 6434 5040
    5010 6442 4983
    5011 6495 4967
    5012 6445 4875
    5013 6622 5107
    5014 6406 4834
    5015 6685 5176
    5016 6633 5084
    5017 6664 5110
    5018 6831 5270
    5019 6848 5307
    5020 6903 5418
    5021 6933 5431
    5022 6808 5219
    5023 6789 5178
    5024 6781 5154
    5025 6732 5128
    5026 6703 5176
    5027 6678 5211
    5028 6678 5211
    5029 6518 5017
    5030 6555 5126
    5031 6681 5229
    5032 6668 5185
    5033 6646 5153
    5034 6694 5187
    5035 6686 5158
    5036 6801 5316
    5037 5760 4103
    5038 5703 4055
    5039 5588 3982
    5040 5549 3940
    5041 5552 3994
    5042 5524 3961
    5043 5524 3961
    5044 5715 4137
    5045 5724 4187
    5046 5646 4071
    5047 5998 4409
    5048 5878 4372
    5049 5969 4453
    5050 5951 4510
    5051 5908 4433
    5052 5722 4219
    5053 5678 4213
    5054 6140 4584
    5055 6164 4586
    5056 6094 4487
    5057 6320 4742
    5058 6351 4799
    5059 6441 4882
    5060 6279 4733
    5061 6333 4815
    5062 6288 4743
    5063 6474 4926
    5064 6720 5147
    5065 6663 5049
    5066 6792 5211
    5067 6830 5235
    5068 6841 5319
    5069 6778 5278
    5070 6708 5174
    5071 6699 5105
    5072 6598 5038
    5073 6414 4795
    5074 6521 4965
    5075 6618 5052
    5076 6778 5237
    5077 6725 5195
    5078 6720 5176
    5079 6327 4845
    5080 6246 4783
    5081 6294 4868
    5082 6349 4950
    5083 6147 4702
    5084 6147 4702
    5085 6239 4771
    5086 6442 4983
    5087 6546 5064
    5088 6579 5071
    5089 6788 5294
    5090 6821 5308
    5091 6644 5133
    5092 6660 5147
    5093 6805 5286
    5094 6756 5207
    5095 6760 5212
    5096 6874 5322
    5097 6874 5322
    5098 6860 5313
    5099 6789 5178
    5100 6765 5150
    5101 6747 5131
    5102 6683 5124
    5103 6634 5142
    5104 6323 4853
    5105 6514 5066
    5106 6767 5328
    5107 6784 5316
    5108 6833 5337
    5109 6883 5379
    5110 6951 5464
    5111 6789 5330
    5112 5592 3992
    5113 5714 4096
    5114 5509 3955
    5115 5508 3931
    5116 5602 4041
    5117 5716 4174
    5118 5961 4396
    5119 5849 4312
    5120 5705 4186
    5121 5732 4282
    5122 5826 4409
    5123 5742 4326
    5124 5738 4315
    5125 5738 4315
    5126 5613 4147
    5127 5723 4214
    5128 6068 4526
    5129 6188 4607
    5130 6101 4513
    5131 6064 4458
    5132 6201 4605
    5133 6471 4882
    5134 6486 4867
    5135 6469 4903
    5136 6410 4877
    5137 6345 4823
    5138 6503 4965
    5139 6579 5039
    5140 6579 5018
    5141 6696 5083
    5142 6668 5035
    5143 6807 5202
    5144 6692 5144
    5145 6644 5095
    5146 6658 5068
    5147 6403 4769
    5148 6597 4985
    5149 6772 5120
    5150 6889 5319
    5151 6998 5389
    5152 6943 5389
    5153 6777 5236
    5154 6388 4852
    5155 6428 4927
    5156 6505 5046
    5157 6639 5174
    5158 6545 5149
    5159 6347 4876
    5160 6496 5007
    5161 6588 5131
    5162 6743 5270
    5163 6864 5348
    5164 6958 5428
    5165 6800 5312
    5166 6800 5312
    5167 6743 5222
    5168 6892 5357
    5169 6790 5264
    5170 6681 5134
    5171 6671 5112
    5172 6800 5235
    5173 6743 5171
    5174 6761 5168
    5175 6735 5137
    5176 6715 5129
    5177 6664 5118
    5178 6347 4891
    5179 6403 5006
    5180 6315 4864
    5181 6536 5121
    5182 6772 5301
    5183 6708 5262
    5184 6874 5343
    5185 6951 5464
    5186 7024 5531
    5187 5551 4062
    5188 5440 3950
    5189 5380 3876
    5190 5659 4125
    5191 5643 4069
    5192 5928 4327
    5193 5905 4310
    5194 5905 4310
    5195 5741 4212
    5196 5698 4222
    5197 5709 4320
    5198 5425 4018
    5199 5625 4207
    5200 5474 4003
    5201 5820 4298
    5202 6014 4423
    5203 6142 4534
    5204 6006 4378
    5205 6161 4553
    5206 6206 4597
    5207 6291 4686
    5208 6276 4657
    5209 6355 4735
    5210 6394 4783
    5211 6313 4722
    5212 6346 4804
    5213 6442 4905
    5214 6577 5012
    5215 6613 5044
    5216 6563 4986
    5217 6533 4970
    5218 6583 5006
    5219 6542 5014
    5220 6314 4751
    5221 6507 4931
    5222 6310 4737
    5223 6429 4776
    5224 6546 4897
    5225 6855 5216
    5226 6875 5260
    5227 6851 5300
    5228 6617 5123
    5229 6481 4965
    5230 6746 5249
    5231 6796 5310
    5232 6790 5327
    5233 6830 5333
    5234 6601 5042
    5235 6601 5042
    5236 6761 5289
    5237 6948 5437
    5238 6942 5436
    5239 7089 5539
    5240 7015 5518
    5241 7016 5620
    5242 6806 5313
    5243 6576 5055
    5244 6462 4949
    5245 6556 4999
    5246 6459 4920
    5247 6582 5017
    5248 6582 5017
    5249 6703 5120
    5250 6693 5109
    5251 6752 5166
    5252 6714 5155
    5253 6668 5210
    5254 6723 5285
    5255 6377 4955
    5256 6436 4961
    5257 6657 5248
    5258 6656 5198
    5259 6800 5354
    5260 6935 5436
    5261 5492 3930
    5262 5055 3566
    5263 5384 3888
    5264 5006 3545
    5265 5417 3884
    5266 5643 4069
    5267 5423 3842
    5268 5639 4031
    5269 5415 3869
    5270 5393 3883
    5271 5192 3734
    5272 5256 3837
    5273 5382 3976
    5274 5436 4008
    5275 5596 4071
    5276 5706 4138
    5277 5902 4311
    5278 6076 4454
    5279 6006 4378
    5280 6157 4528
    5281 6249 4642
    5282 6337 4710
    5283 6305 4673
    5284 6320 4725
    5285 6344 4773
    5286 6487 4957
    5287 6510 4981
    5288 6615 5073
    5289 6609 4996
    5290 6609 4996
    5291 6683 5114
    5292 6533 4970
    5293 6664 5134
    5294 6581 5055
    5295 6480 5004
    5296 6277 4757
    5297 6348 4838
    5298 6464 4883
    5299 6600 5021
    5300 6724 5134
    5301 6679 5187
    5302 6580 5098
    5303 6580 5098
    5304 6475 4964
    5305 6746 5249
    5306 6874 5364
    5307 6854 5330
    5308 6844 5326
    5309 6723 5156
    5310 6781 5181
    5311 6985 5393
    5312 7029 5449
    5313 7129 5567
    5314 7148 5636
    5315 7194 5741
    5316 7008 5590
    5317 6859 5468
    5318 6576 5055
    5319 6503 5058
    5320 6485 5060
    5321 6496 5090
    5322 6467 4918
    5323 6569 5037
    5324 6671 5118
    5325 6778 5290
    5326 6615 5083
    5327 6439 4929
    5328 6751 5230
    5329 6728 5216
    5330 6563 5097
    5331 6436 4961
    5332 6709 5288
    5333 6812 5430
    5334 6909 5489
    5335 6866 5438
    5336 4925 3520
    5337 5082 3627
    5338 5173 3692
    5339 5126 3644
    5340 5220 3729
    5341 5383 3884
    5342 5556 4038
    5343 5338 3886
    5344 5395 3983
    5345 5230 3810
    5346 5486 4055
    5347 5567 4139
    5348 5671 4163
    5349 5742 4150
    5350 5931 4320
    5351 5788 4169
    5352 6071 4459
    5353 6069 4459
    5354 6161 4525
    5355 6084 4455
    5356 6365 4753
    5357 6365 4753
    5358 6390 4859
    5359 6375 4807
    5360 6535 4988
    5361 6490 4915
    5362 6543 4930
    5363 6679 5033
    5364 6756 5114
    5365 6855 5314
    5366 6805 5265
    5367 6737 5279
    5368 6650 5156
    5369 6530 5058
    5370 6530 5058
    5371 6364 4800
    5372 6479 4884
    5373 6695 5095
    5374 6658 5103
    5375 6576 5124
    5376 6444 4930
    5377 6551 5007
    5378 6705 5124
    5379 6739 5156
    5380 6820 5262
    5381 6655 5086
    5382 6688 5090
    5383 6804 5185
    5384 6925 5323
    5385 6940 5344
    5386 7077 5494
    5387 7055 5511
    5388 7050 5551
    5389 7008 5590
    5390 6574 5156
    5391 6439 4945
    5392 6530 5053
    5393 6692 5294
    5394 6498 5104
    5395 6583 5099
    5396 6875 5481
    5397 6749 5270
    5398 6853 5425
    5399 6649 5150
    5400 6637 5167
    5401 6708 5196
    5402 6728 5216
    5403 6822 5305
    5404 6864 5406
    5405 7019 5598
    5406 6865 5435
    5407 6894 5522
    5408 6892 5521
    5409 5122 3683
    5410 5211 3727
    5411 5236 3729
    5412 5312 3779
    5413 5500 3958
    5414 5611 4007
    5415 5642 4114
    5416 5732 4190
    5417 5682 4218
    5418 5602 4079
    5419 5642 4128
    5420 5567 4024
    5421 5362 3826
    5422 5710 4114
    5423 5649 4093
    5424 5925 4320
    5425 5976 4382
    5426 5972 4389
    5427 5983 4427
    5428 6089 4463
    5429 6096 4512
    5430 6209 4692
    5431 6172 4657
    5432 6352 4821
    5433 6402 4798
    5434 6421 4835
    5435 6493 4856
    5436 6493 4856
    5437 6661 5057
    5438 6515 4931
    5439 6634 5064
    5440 6513 4991
    5441 6589 5090
    5442 6544 5062
    5443 6339 4814
    5444 6192 4642
    5445 6472 4840
    5446 6634 5112
    5447 6601 5071
    5448 6508 4955
    5449 6557 4937
    5450 6602 5014
    5451 6708 5129
    5452 6695 5132
    5453 6585 5032
    5454 6625 5049
    5455 6740 5149
    5456 6792 5179
    5457 6765 5177
    5458 6882 5355
    5459 6858 5388
    5460 6688 5243
    5461 6522 5045
    5462 6523 5024
    5463 6468 5028
    5464 6556 5135
    5465 6633 5230
    5466 6706 5360
    5467 6602 5203
    5468 6935 5581
    5469 6947 5535
    5470 6693 5246
    5471 6818 5310
    5472 6798 5326
    5473 6856 5374
    5474 6872 5313
    5475 7016 5488
    5476 7125 5677
    5477 7125 5677
    5478 7054 5671
    5479 7105 5733
    5480 5234 3724
    5481 5260 3718
    5482 5483 3899
    5483 5542 4005
    5484 5497 3967
    5485 5497 3967
    5486 5673 4139
    5487 5602 4079
    5488 5406 3908
    5489 5442 3958
    5490 5199 3767
    5491 5429 3941
    5492 5532 4055
    5493 5732 4174
    5494 5682 4134
    5495 5917 4373
    5496 5970 4456
    5497 5944 4429
    5498 5909 4424
    5499 6107 4599
    5500 6172 4657
    5501 6308 4803
    5502 6383 4857
    5503 6273 4712
    5504 6374 4778
    5505 6324 4723
    5506 6415 4821
    5507 6473 4860
    5508 6517 4971
    5509 6577 5033
    5510 6442 4950
    5511 6259 4731
    5512 6332 4808
    5513 6192 4642
    5514 6309 4757
    5515 6577 4975
    5516 6601 5019
    5517 6505 4930
    5518 6482 4865
    5519 6677 5034
    5520 6732 5122
    5521 6691 5146
    5522 6798 5249
    5523 6759 5203
    5524 6814 5231
    5525 6824 5291
    5526 6765 5177
    5527 6841 5335
    5528 6896 5464
    5529 6874 5490
    5530 6565 5145
    5531 6611 5189
    5532 6749 5274
    5533 6961 5536
    5534 6822 5507
    5535 6681 5291
    5536 6933 5560
    5537 6838 5436
    5538 6730 5338
    5539 6693 5246
    5540 6782 5297
    5541 6996 5486
    5542 7023 5470
    5543 6965 5437
    5544 7033 5453
    5545 7075 5560
    5546 6987 5489
    5547 7079 5608
    5548 7122 5748
    5549 5100 3635
    5550 5312 3777
    5551 5395 3883
    5552 5408 3943
    5553 5346 3869
    5554 5166 3784
    5555 5096 3696
    5556 4970 3620
    5557 5104 3735
    5558 5169 3790
    5559 5375 3952
    5560 5375 3952
    5561 5453 3973
    5562 5419 3936
    5563 5702 4156
    5564 5782 4280
    5565 5892 4407
    5566 5909 4424
    5567 5861 4415
    5568 6193 4723
    5569 6135 4639
    5570 6120 4568
    5571 5951 4375
    5572 6239 4614
    5573 6304 4669
    5574 6399 4760
    5575 6472 4814
    5576 6587 4951
    5577 6660 4999
    5578 6505 4968
    5579 6259 4731
    5580 6317 4804
    5581 6498 5048
    5582 6597 5118
    5583 6544 4986
    5584 6486 4949
    5585 6464 4868
    5586 6721 5110
    5587 6731 5099
    5588 6731 5099
    5589 6775 5198
    5590 6832 5248
    5591 6770 5187
    5592 6814 5231
    5593 6870 5309
    5594 6903 5392
    5595 6885 5363
    5596 6873 5406
    5597 6702 5254
    5598 6842 5454
    5599 6963 5558
    5600 7030 5581
    5601 7030 5581
    5602 6939 5600
    5603 6721 5418
    5604 6693 5238
    5605 6857 5468
    5606 6846 5449
    5607 6836 5341
    5608 6864 5269
    5609 6192 4953
    5610 5091 3736
    5611 5213 3813
    5612 5323 3896
    5613 5168 3832
    5614 5099 3781
    5615 5099 3781
    5616 5001 3693
    5617 4921 3629
    5618 5030 3712
    5619 5146 3816
    5620 5277 3877
    5621 5282 3901
    5622 5321 3904
    5623 5420 4019
    5624 5563 4145
    5625 5741 4321
    5626 5742 4373
    5627 5695 4317
    5628 5704 4349
    5629 5904 4470
    5630 5863 4394
    5631 5975 4429
    5632 5933 4393
    5633 6099 4490
    5634 6262 4634
    5635 6413 4803
    5636 6384 4833
    5637 6587 4897
    5638 6595 4965
    5639 6623 5049
    5640 6484 5008
    5641 6361 4918
    5642 6456 4956
    5643 6408 4917
    5644 6584 5081
    5645 6441 4922
    5646 6498 4965
    5647 6551 4968
    5648 6722 5087
    5649 6739 5141
    5650 6752 5119
    5651 6758 5182
    5652 6740 5184
    5653 6806 5205
    5654 6850 5293
    5655 6826 5319
    5656 6826 5319
    5657 6831 5372
    5658 6723 5311
    5659 6730 5261
    5660 6893 5418
    5661 7038 5623
    5662 7032 5676
    5663 6865 5537
    5664 6578 5149
    5665 6712 5312
    5666 7074 5534
    5667 7067 5543
    5668 7079 5650
    5669 7029 5662
    5670 6907 5597
    5671 6836 5618
    5672 6327 5194
    5673 6361 5213
    5674 6138 4897
    5675 5031 3714
    5676 5091 3736
    5677 5136 3814
    5678 5175 3827
    5679 5091 3785
    5680 5118 3809
    5681 5086 3785
    5682 5022 3733
    5683 5042 3734
    5684 5107 3775
    5685 5173 3832
    5686 5253 3911
    5687 5351 4000
    5688 5311 3934
    5689 5420 4019
    5690 5596 4266
    5691 5694 4343
    5692 5616 4300
    5693 5578 4239
    5694 5540 4226
    5695 5724 4334
    5696 5754 4355
    5697 5855 4422
    5698 6095 4643
    5699 6305 4722
    5700 6473 4933
    5701 6466 4984
    5702 6384 4833
    5703 6302 4719
    5704 6555 4884
    5705 6563 4988
    5706 6691 5148
    5707 6533 5048
    5708 6145 4714
    5709 6032 4610
    5710 6344 4916
    5711 6315 4901
    5712 6394 4928
    5713 6459 4986
    5714 6681 5100
    5715 6787 5202
    5716 6787 5202
    5717 6780 5238
    5718 6765 5252
    5719 6742 5168
    5720 6811 5306
    5721 6874 5434
    5722 6849 5430
    5723 6724 5294
    5724 6657 5261
    5725 6739 5333
    5726 6843 5457
    5727 7039 5704
    5728 6886 5543
    5729 6886 5543
    5730 6520 5110
    5731 7000 5483
    5732 6958 5469
    5733 6923 5485
    5734 7048 5654
    5735 6987 5670
    5736 6938 5686
    5737 6786 5549
    5738 6163 4995
    5739 5915 4748
    5740 6511 5334
    5741 6612 5449
    5742 6686 5498
    5743 6672 5474
    5744 6519 5325
    5745 6530 5359
    5746 5038 3713
    5747 5065 3754
    5748 5207 3858
    5749 5142 3839
    5750 5198 3904
    5751 5108 3812
    5752 5193 3877
    5753 5114 3810
    5754 5109 3804
    5755 5096 3770
    5756 5218 3871
    5757 5319 3964
    5758 5351 4000
    5759 5432 4051
    5760 5586 4238
    5761 5675 4301
    5762 5569 4260
    5763 5466 4184
    5764 5365 4071
    5765 5336 4010
    5766 5685 4304
    5767 5685 4304
    5768 5901 4501
    5769 6115 4681
    5770 6312 4806
    5771 6473 4933
    5772 6513 4993
    5773 6179 4708
    5774 6131 4620
    5775 6517 4994
    5776 6486 4996
    5777 6389 4872
    5778 6151 4686
    5779 5874 4492
    5780 5874 4492
    5781 6241 4846
    5782 6393 4996
    5783 6524 5119
    5784 6459 4986
    5785 6604 5150
    5786 6748 5163
    5787 6621 5063
    5788 6735 5217
    5789 6702 5245
    5790 6775 5315
    5791 6650 5197
    5792 6110 4701
    5793 6533 5113
    5794 6410 5031
    5795 6649 5262
    5796 6843 5457
    5797 6905 5554
    5798 6829 5518
    5799 6604 5281
    5800 6837 5526
    5801 6927 5576
    5802 7027 5565
    5803 7027 5565
    5804 7040 5488
    5805 7065 5558
    5806 6907 5449
    5807 6943 5557
    5808 6927 5558
    5809 6897 5584
    5810 6856 5582
    5811 6667 5416
    5812 6288 5129
    5813 6503 5349
    5814 6585 5386
    5815 6697 5499
    5816 6686 5498
    5817 6684 5459
    5818 6667 5427
    5819 6663 5394
    5820 6560 5333
    5821 5084 3695
    5822 5206 3834
    5823 5160 3842
    5824 5256 3908
    5825 5291 3951
    5826 5438 4074
    5827 5278 3969
    5828 5354 4004
    5829 5222 3912
    5830 5303 3971
    5831 5279 3905
    5832 5351 3945
    5833 5423 3982
    5834 5396 3998
    5835 5404 4017
    5836 5494 4104
    5837 5549 4171
    5838 5572 4241
    5839 5515 4221
    5840 5478 4195
    5841 5465 4159
    5842 5420 4048
    5843 5673 4299
    5844 5993 4533
    5845 6275 4786
    5846 6342 4861
    5847 6325 4799
    5848 6260 4773
    5849 6174 4737
    5850 6174 4737
    5851 6385 4882
    5852 6348 4854
    5853 6223 4707
    5854 6202 4751
    5855 6187 4794
    5856 6217 4839
    5857 6145 4737
    5858 6574 5199
    5859 6730 5353
    5860 6795 5460
    5861 6834 5474
    5862 6588 5161
    5863 6529 5101
    5864 6594 5108
    5865 6543 5086
    5866 6627 5167
    5867 6468 5052
    5868 6332 4929
    5869 6142 4793
    5870 6071 4691
    5871 6360 4987
    5872 6430 5110
    5873 6633 5302
    5874 6668 5378
    5875 6583 5311
    5876 6653 5328
    5877 6722 5368
    5878 6757 5409
    5879 6894 5505
    5880 7059 5686
    5881 7043 5606
    5882 7095 5611
    5883 7054 5585
    5884 7037 5548
    5885 6881 5407
    5886 6942 5540
    5887 6815 5491
    5888 6671 5324
    5889 6671 5324
    5890 6812 5495
    5891 6709 5425
    5892 6644 5476
    5893 6771 5645
    5894 6667 5487
    5895 6755 5538
    5896 6809 5571
    5897 6757 5520
    5898 6782 5544
    5899 6646 5386
    5900 6603 5322
    5901 6637 5323
    5902 5210 3779
    5903 5329 3897
    5904 5367 3999
    5905 5436 4090
    5906 5562 4137
    5907 5625 4174
    5908 5504 4107
    5909 5586 4155
    5910 5567 4159
    5911 5303 3971
    5912 5436 4049
    5913 5393 3974
    5914 5472 4033
    5915 5375 3995
    5916 5414 4060
    5917 5403 4014
    5918 5416 4050
    5919 5545 4240
    5920 5542 4226
    5921 5548 4209
    5922 5551 4217
    5923 5574 4234
    5924 5769 4357
    5925 5769 4357
    5926 6075 4581
    5927 6122 4635
    5928 6235 4749
    5929 6246 4768
    5930 6301 4847
    5931 6411 4960
    5932 6285 4837
    5933 6123 4652
    5934 6205 4716
    5935 6358 4877
    5936 6364 4919
    5937 6271 4871
    5938 6271 4871
    5939 6387 4972
    5940 6428 4991
    5941 6793 5391
    5942 6784 5403
    5943 6559 5186
    5944 6451 5077
    5945 6400 5007
    5946 6430 5107
    5947 6508 5155
    5948 6556 5202
    5949 6514 5203
    5950 6217 4912
    5951 6321 5062
    5952 6181 4840
    5953 6289 4961
    5954 6580 5317
    5955 6685 5433
    5956 6811 5551
    5957 6785 5531
    5958 6734 5374
    5959 6793 5449
    5960 6851 5533
    5961 7041 5727
    5962 7120 5745
    5963 7073 5669
    5964 7044 5639
    5965 7032 5606
    5966 7032 5606
    5967 6943 5502
    5968 6821 5395
    5969 6800 5430
    5970 6856 5590
    5971 6738 5472
    5972 6737 5484
    5973 6728 5489
    5974 6933 5807
    5975 6936 5810
    5976 6784 5670
    5977 6682 5516
    5978 6734 5485
    5979 6788 5516
    5980 6786 5530
    5981 6709 5425
    5982 6594 5282
    5983 5956 4318
    5984 5752 4181
    5985 5956 4425
    5986 5627 4281
    5987 5627 4281
    5988 5483 4068
    5989 5553 4093
    5990 5611 4155
    5991 5586 4155
    5992 5737 4276
    5993 5696 4277
    5994 5829 4372
    5995 5646 4193
    5996 5540 4072
    5997 5542 4166
    5998 5542 4166
    5999 5393 4046
    6000 5480 4148
    6001 5536 4198
    6002 5519 4197
    6003 5512 4156
    6004 5551 4217
    6005 5663 4333
    6006 5713 4351
    6007 5916 4537
    6008 5974 4527
    6009 6085 4637
    6010 6243 4759
    6011 6375 4913
    6012 6469 5020
    6013 6376 4896
    6014 6358 4879
    6015 6242 4751
    6016 6231 4757
    6017 6358 4877
    6018 6441 4951
    6019 6478 5031
    6020 6609 5152
    6021 6466 5019
    6022 6463 5074
    6023 6717 5392
    6024 6787 5483
    6025 6540 5200
    6026 6741 5413
    6027 6605 5347
    6028 6605 5347
    6029 6284 4939
    6030 6556 5202
    6031 6653 5322
    6032 6587 5317
    6033 6659 5425
    6034 6224 4970
    6035 6238 4979
    6036 6501 5231
    6037 6488 5236
    6038 6744 5431
    6039 6712 5384
    6040 6837 5544
    6041 6837 5544
    6042 6944 5680
    6043 7041 5727
    6044 7122 5772
    6045 7085 5685
    6046 6980 5606
    6047 6944 5542
    6048 6803 5423
    6049 6838 5479
    6050 6816 5555
    6051 6865 5680
    6052 6842 5657
    6053 6766 5533
    6054 6703 5480
    6055 6763 5572
    6056 6933 5807
    6057 6883 5745
    6058 6812 5674
    6059 6608 5343
    6060 6636 5346
    6061 6614 5299
    6062 6698 5396
    6063 5989 4314
    6064 6074 4395
    6065 6086 4477
    6066 5997 4540
    6067 5744 4362
    6068 5411 4025
    6069 5778 4299
    6070 5822 4363
    6071 5882 4375
    6072 5913 4407
    6073 5913 4407
    6074 5743 4276
    6075 5557 4105
    6076 5595 4135
    6077 5712 4298
    6078 5539 4201
    6079 5489 4207
    6080 5520 4183
    6081 5423 4070
    6082 5622 4270
    6083 5806 4445
    6084 5968 4554
    6085 6062 4638
    6086 6155 4741
    6087 6053 4622
    6088 6136 4672
    6089 6208 4760
    6090 6378 4913
    6091 6488 5029
    6092 6331 4861
    6093 6346 4853
    6094 6352 4875
    6095 6346 4814
    6096 6470 4938
    6097 6554 4990
    6098 6636 5120
    6099 6445 4941
    6100 6599 5120
    6101 6352 4879
    6102 6652 5341
    6103 6857 5549
    6104 6669 5340
    6105 6668 5295
    6106 6757 5455
    6107 6152 4893
    6108 5617 4333
    6109 6354 4976
    6110 6479 5164
    6111 6718 5397
    6112 6641 5295
    6113 6569 5288
    6114 6569 5288
    6115 6451 5179
    6116 6492 5242
    6117 6383 5079
    6118 6464 5142
    6119 6674 5330
    6120 6954 5653
    6121 7010 5680
    6122 7033 5692
    6123 6792 5394
    6124 6829 5441
    6125 6693 5336
    6126 6716 5380
    6127 6649 5374
    6128 6790 5471
    6129 6656 5388
    6130 6792 5576
    6131 6576 5338
    6132 6504 5245
    6133 6703 5480
    6134 6759 5556
    6135 6787 5621
    6136 6682 5527
    6137 6556 5418
    6138 5712 4082
    6139 5675 4019
    6140 5930 4276
    6141 5980 4387
    6142 5917 4410
    6143 5600 4259
    6144 5764 4267
    6145 5871 4396
    6146 5820 4333
    6147 5830 4330
    6148 5629 4154
    6149 5572 4111
    6150 5685 4271
    6151 5787 4355
    6152 5926 4511
    6153 5736 4434
    6154 5385 4046
    6155 5431 4077
    6156 5414 4057
    6157 5588 4229
    6158 5835 4435
    6159 6004 4571
    6160 6091 4622
    6161 6176 4716
    6162 6175 4717
    6163 6227 4773
    6164 6354 4876
    6165 6193 4697
    6166 6201 4706
    6167 6402 4939
    6168 6462 4960
    6169 6449 4952
    6170 6370 4838
    6171 6401 4846
    6172 6466 4914
    6173 6536 4974
    6174 6481 4968
    6175 6235 4765
    6176 6545 5160
    6177 6835 5485
    6178 6868 5509
    6179 6735 5389
    6180 6674 5296
    6181 6823 5516
    6182 6402 5213
    6183 6402 5213
    6184 6013 4689
    6185 6322 5010
    6186 6582 5294
    6187 6642 5340
    6188 6593 5248
    6189 6566 5220
    6190 6587 5300
    6191 6557 5213
    6192 6515 5242
    6193 6573 5267
    6194 6531 5199
    6195 6766 5392
    6196 6766 5392
    6197 6923 5581
    6198 6708 5338
    6199 6714 5325
    6200 6669 5301
    6201 6622 5308
    6202 6581 5322
    6203 6466 5249
    6204 6267 5076
    6205 6395 5205
    6206 6424 5251
    6207 6595 5428
    6208 6584 5395
    6209 6659 5474
    6210 6694 5506
    6211 6714 5563
    6212 6662 5514
    6213 6328 5180
    6214 6428 5218
    6215 5698 4081
    6216 5980 4387
    6217 5919 4391
    6218 5845 4427
    6219 5801 4367
    6220 5888 4443
    6221 5801 4358
    6222 5865 4371
    6223 5726 4211
    6224 5709 4209
    6225 5516 4048
    6226 5762 4338
    6227 5841 4451
    6228 5926 4511
    6229 5952 4602
    6230 5447 4125
    6231 5578 4212
    6232 5663 4335
    6233 5807 4440
    6234 5919 4458
    6235 6035 4573
    6236 6024 4560
    6237 5917 4475
    6238 6083 4610
    6239 6083 4610
    6240 6267 4757
    6241 6193 4697
    6242 6196 4759
    6243 6358 4869
    6244 6394 4892
    6245 6438 4905
    6246 6462 4922
    6247 6478 4933
    6248 6534 5008
    6249 6530 4986
    6250 6583 5092
    6251 6582 5197
    6252 6582 5197
    6253 6740 5366
    6254 6868 5509
    6255 6733 5349
    6256 6655 5295
    6257 6871 5544
    6258 6883 5587
    6259 6765 5457
    6260 6135 4809
    6261 6079 4709
    6262 6644 5340
    6263 6573 5247
    6264 6543 5218
    6265 6562 5222
    6266 6460 5099
    6267 6557 5213
    6268 6484 5154
    6269 6525 5225
    6270 6491 5151
    6271 6496 5125
    6272 6476 5120
    6273 6502 5129
    6274 6608 5251
    6275 6657 5293
    6276 6701 5389
    6277 6601 5360
    6278 6545 5360
    6279 6401 5235
    6280 6267 5076
    6281 6608 5478
    6282 6742 5634
    6283 6797 5657
    6284 6637 5491
    6285 6701 5549
    6286 6727 5548
    6287 6627 5443
    6288 6414 5301
    6289 6229 5077
    6290 6355 5174
    6291 6524 5279
    6292 6082 4573
    6293 6000 4525
    6294 6055 4570
    6295 5955 4518
    6296 6004 4569
    6297 5944 4408
    6298 5895 4436
    6299 5955 4545
    6300 6127 4698
    6301 6070 4678
    6302 6043 4628
    6303 5783 4405
    6304 5963 4628
    6305 5976 4622
    6306 5976 4622
    6307 5892 4447
    6308 5992 4520
    6309 5944 4496
    6310 5863 4410
    6311 5949 4487
    6312 6157 4660
    6313 6198 4723
    6314 6284 4817
    6315 6314 4839
    6316 6338 4824
    6317 6454 4944
    6318 6527 5007
    6319 6527 5007
    6320 6511 4991
    6321 6605 5099
    6322 6609 5081
    6323 6680 5179
    6324 6602 5189
    6325 6602 5197
    6326 6504 5046
    6327 6502 5039
    6328 6600 5154
    6329 6835 5458
    6330 6806 5429
    6331 6767 5377
    6332 6650 5296
    6333 6397 5082
    6334 6168 4852
    6335 6431 5041
    6336 6654 5272
    6337 6694 5395
    6338 6562 5222
    6339 6617 5296
    6340 6491 5174
    6341 6390 5053
    6342 6426 5103
    6343 6517 5205
    6344 6514 5181
    6345 6564 5242
    6346 6531 5200
    6347 6625 5312
    6348 6703 5351
    6349 6654 5364
    6350 6545 5360
    6351 6719 5596
    6352 6877 5771
    6353 6864 5742
    6354 6808 5647
    6355 6790 5618
    6356 6683 5512
    6357 6575 5396
    6358 6575 5396
    6359 6483 5404
    6360 6229 5077
    6361 6178 5021
    6362 6312 5134
    6363 6137 4644
    6364 6145 4653
    6365 6042 4566
    6366 5900 4453
    6367 6070 4563
    6368 6190 4681
    6369 6075 4598
    6370 6228 4716
    6371 6217 4728
    6372 6096 4629
    6373 5924 4523
    6374 6006 4632
    6375 6136 4763
    6376 6096 4687
    6377 6029 4597
    6378 5970 4504
    6379 5912 4471
    6380 5977 4522
    6381 6156 4694
    6382 6267 4810
    6383 6267 4810
    6384 6330 4862
    6385 6388 4906
    6386 6350 4878
    6387 6409 4905
    6388 6537 4997
    6389 6598 5061
    6390 6597 5115
    6391 6565 5059
    6392 6695 5182
    6393 6635 5183
    6394 6546 5108
    6395 6559 5132
    6396 6559 5132
    6397 6510 5071
    6398 6625 5210
    6399 6671 5217
    6400 6636 5208
    6401 6707 5327
    6402 6660 5276
    6403 6466 5101
    6404 6341 5000
    6405 6573 5153
    6406 6836 5422
    6407 6964 5545
    6408 6852 5519
    6409 6866 5561
    6410 6729 5437
    6411 6574 5260
    6412 6457 5143
    6413 6554 5235
    6414 6561 5223
    6415 6615 5308
    6416 6566 5280
    6417 6584 5334
    6418 6418 5232
    6419 6340 5189
    6420 6387 5296
    6421 6689 5581
    6422 6689 5581
    6423 6823 5688
    6424 6728 5552
    6425 6693 5482
    6426 6581 5380
    6427 6585 5431
    6428 6123 4983
    6429 5925 4820
    6430 5948 4813
    6431 5862 4761
    6432 5989 4829
    6433 6222 4573
    6434 6171 4611
    6435 6007 4490
    6436 5902 4421
    6437 6192 4638
    6438 6350 4797
    6439 6266 4734
    6440 6251 4625
    6441 6191 4668
    6442 6154 4753
    6443 5941 4552
    6444 5616 4214
    6445 5799 4345
    6446 5764 4314
    6447 6015 4572
    6448 5970 4504
    6449 6013 4546
    6450 6065 4631
    6451 6239 4780
    6452 6330 4872
    6453 6381 4874
    6454 6415 4911
    6455 6391 4881
    6456 6459 5005
    6457 6517 5046
    6458 6516 4975
    6459 6414 4876
    6460 6551 5030
    6461 6451 4960
    6462 6533 5019
    6463 6461 4957
    6464 6340 4874
    6465 6605 5185
    6466 6774 5368
    6467 6710 5281
    6468 6653 5225
    6469 6495 5081
    6470 6392 4992
    6471 6484 5070
    6472 6325 4908
    6473 6413 5043
    6474 6413 5043
    6475 6533 5192
    6476 6836 5422
    6477 6884 5465
    6478 6929 5562
    6479 6847 5528
    6480 6668 5306
    6481 6471 5074
    6482 6594 5273
    6483 6584 5279
    6484 6309 4972
    6485 6339 5009
    6486 6356 5073
    6487 6356 5073
    6488 6342 5105
    6489 6418 5232
    6490 6068 4975
    6491 6002 4927
    6492 5886 4816
    6493 6362 5273
    6494 6362 5269
    6495 6506 5340
    6496 6398 5207
    6497 6486 5323
    6498 6473 5326
    6499 6476 5334
    6500 6123 4983
    6501 5924 4840
    6502 5742 4662
    6503 5717 4644
    6504 5889 4812
    6505 5938 4876
    6506 6100 4473
    6507 5918 4252
    6508 6098 4456
    6509 6165 4581
    6510 5993 4477
    6511 5973 4477
    6512 6284 4700
    6513 6383 4806
    6514 6022 4402
    6515 6108 4440
    6516 6108 4440
    6517 6242 4671
    6518 6284 4808
    6519 6107 4702
    6520 6024 4618
    6521 5615 4211
    6522 5803 4394
    6523 5828 4371
    6524 6101 4648
    6525 6263 4768
    6526 6301 4814
    6527 6260 4770
    6528 6227 4727
    6529 6121 4649
    6530 6248 4714
    6531 6310 4823
    6532 6520 5061
    6533 6547 5065
    6534 6450 4951
    6535 6440 4955
    6536 6415 4924
    6537 6484 5039
    6538 6350 4887
    6539 6337 4929
    6540 6557 5192
    6541 6736 5318
    6542 6748 5313
    6543 6681 5259
    6544 6681 5259
    6545 6586 5203
    6546 6482 5089
    6547 6286 4916
    6548 6100 4760
    6549 6288 4899
    6550 6497 5126
    6551 6454 5012
    6552 6750 5339
    6553 6807 5440
    6554 6803 5465
    6555 6472 5122
    6556 6254 4869
    6557 6254 4869
    6558 6590 5280
    6559 6664 5354
    6560 6349 5046
    6561 6382 5125
    6562 6298 5006
    6563 5951 4824
    6564 5601 4542
    6565 5401 4377
    6566 5672 4670
    6567 6067 5025
    6568 6022 4952
    6569 6272 5143
    6570 6227 5076
    6571 6362 5265
    6572 6039 5022
    6573 5793 4787
    6574 5210 4111
    6575 5212 4136
    6576 5776 4714
    6577 5890 4867
    6578 5842 4206
    6579 5977 4332
    6580 6164 4512
    6581 6208 4598
    6582 6098 4496
    6583 6137 4565
    6584 6137 4565
    6585 6202 4661
    6586 6280 4753
    6587 6297 4745
    6588 5968 4402
    6589 5918 4276
    6590 6232 4657
    6591 6299 4773
    6592 6261 4756
    6593 6183 4682
    6594 6221 4755
    6595 6018 4600
    6596 5934 4516
    6597 6208 4722
    6598 6179 4714
    6599 6298 4807
    6600 6257 4739
    6601 6210 4745
    6602 6140 4652
    6603 6173 4731
    6604 6289 4834
    6605 6329 4863
    6606 6474 5018
    6607 6506 5047
    6608 6568 5121
    6609 6549 5072
    6610 6597 5135
    6611 6579 5123
    6612 6563 5098
    6613 6425 5004
    6614 6619 5219
    6615 6634 5224
    6616 6591 5154
    6617 6679 5264
    6618 6606 5194
    6619 6543 5147
    6620 6443 5082
    6621 6080 4756
    6622 6003 4629
    6623 5906 4542
    6624 6029 4594
    6625 6029 4594
    6626 6554 5119
    6627 6698 5291
    6628 6768 5402
    6629 6582 5173
    6630 6534 5218
    6631 6721 5404
    6632 6734 5428
    6633 6711 5387
    6634 6611 5356
    6635 6387 5156
    6636 6285 5092
    6637 5770 4605
    6638 5387 4369
    6639 4491 3538
    6640 4742 3712
    6641 5208 4209
    6642 5676 4665
    6643 5697 4668
    6644 6113 5044
    6645 6200 5136
    6646 6186 5137
    6647 6253 5175
    6648 6324 5374
    6649 6020 5049
    6650 5246 4211
    6651 5302 4306
    6652 5621 4632
    6653 5822 4827
    6654 5982 4953
    6655 5384 3805
    6656 5193 3660
    6657 5519 3916
    6658 5717 4093
    6659 5854 4224
    6660 6111 4478
    6661 6208 4598
    6662 6119 4491
    6663 6078 4470
    6664 6140 4613
    6665 6153 4637
    6666 5860 4314
    6667 6038 4485
    6668 6155 4616
    6669 6260 4696
    6670 6366 4816
    6671 6338 4781
    6672 6304 4781
    6673 6265 4731
    6674 6221 4755
    6675 6306 4865
    6676 6116 4660
    6677 6267 4796
    6678 6332 4823
    6679 6394 4897
    6680 6300 4837
    6681 6282 4858
    6682 6193 4761
    6683 6182 4739
    6684 6201 4782
    6685 6201 4782
    6686 6333 4874
    6687 6506 5047
    6688 6542 5060
    6689 6622 5133
    6690 6510 5000
    6691 6627 5136
    6692 6502 5000
    6693 6611 5154
    6694 6588 5146
    6695 6531 5103
    6696 6453 5044
    6697 6444 5037
    6698 6359 4971
    6699 6501 5112
    6700 6443 5082
    6701 6024 4700
    6702 5710 4351
    6703 5917 4598
    6704 5918 4570
    6705 6375 5038
    6706 6607 5228
    6707 6691 5339
    6708 6625 5240
    6709 6729 5356
    6710 6794 5453
    6711 6818 5440
    6712 6803 5445
    6713 6711 5387
    6714 6796 5485
    6715 6652 5381
    6716 6568 5300
    6717 6056 4908
    6718 5825 4712
    6719 5209 4199
    6720 5275 4306
    6721 4986 4037
    6722 5406 4399
    6723 5480 4493
    6724 5715 4710
    6725 6099 5103
    6726 6200 5136
    6727 6157 5120
    6728 6041 5011
    6729 6141 5113
    6730 6076 5125
    6731 5718 4807
    6732 5461 4514
    6733 5729 4814
    6734 5765 4787
    6735 5799 4766
    6736 5924 4870
    6737 5470 3908
    6738 5432 3888
    6739 5131 3615
    6740 5386 3849
    6741 5515 3996
    6742 5642 4048
    6743 5787 4178
    6744 6031 4377
    6745 6085 4456
    6746 5958 4344
    6747 5878 4239
    6748 6050 4464
    6749 5605 4071
    6750 5897 4378
    6751 5657 4127
    6752 5881 4321
    6753 6194 4628
    6754 6318 4774
    6755 6415 4871
    6756 6354 4842
    6757 6290 4770
    6758 6316 4813
    6759 6400 4870
    6760 6421 4910
    6761 6288 4800
    6762 6347 4837
    6763 6433 4934
    6764 6433 4934
    6765 6362 4883
    6766 6301 4864
    6767 6246 4811
    6768 6118 4680
    6769 6060 4621
    6770 6125 4699
    6771 6007 4553
    6772 6273 4802
    6773 6108 4632
    6774 6129 4607
    6775 6031 4561
    6776 6318 4784
    6777 6318 4784
    6778 6481 4992
    6779 6515 5069
    6780 6502 5078
    6781 6349 4938
    6782 6300 4889
    6783 6359 4971
    6784 6140 4784
    6785 5965 4634
    6786 6017 4775
    6787 6094 4830
    6788 6213 4952
    6789 6464 5169
    6790 6791 5513
    6791 6713 5388
    6792 6848 5519
    6793 6754 5410
    6794 6757 5390
    6795 6811 5470
    6796 6818 5440
    6797 6798 5402
    6798 6838 5447
    6799 6763 5352
    6800 6695 5386
    6801 6569 5275
    6802 6226 5014
    6803 6221 5027
    6804 5915 4844
    6805 5915 4844
    6806 5400 4414
    6807 5503 4431
    6808 5693 4639
    6809 5715 4710
    6810 5862 4856
    6811 6077 5091
    6812 6059 5081
    6813 6095 5094
    6814 6160 5184
    6815 6169 5240
    6816 5698 4774
    6817 5522 4576
    6818 5596 4593
    6819 5773 4781
    6820 5799 4766
    6821 5493 3906
    6822 5542 3992
    6823 5418 3901
    6824 5547 4030
    6825 5617 4092
    6826 5717 4143
    6827 5816 4150
    6828 5778 4148
    6829 5774 4128
    6830 5958 4332
    6831 5910 4289
    6832 5869 4272
    6833 5731 4166
    6834 5912 4314
    6835 5850 4307
    6836 5653 4125
    6837 5994 4486
    6838 6142 4638
    6839 6299 4801
    6840 6391 4864
    6841 6380 4843
    6842 6224 4733
    6843 6299 4795
    6844 6232 4724
    6845 6329 4813
    6846 6324 4806
    6847 6293 4806
    6848 6140 4632
    6849 6342 4827
    6850 6267 4763
    6851 6268 4802
    6852 6110 4679
    6853 5991 4552
    6854 5881 4438
    6855 5750 4354
    6856 5799 4411
    6857 5799 4411
    6858 5926 4463
    6859 5967 4514
    6860 5993 4541
    6861 6040 4620
    6862 6081 4613
    6863 6091 4623
    6864 6323 4829
    6865 6258 4827
    6866 6273 4869
    6867 6147 4766
    6868 6034 4695
    6869 6020 4684
    6870 5775 4533
    6871 5823 4548
    6872 5854 4606
    6873 6069 4824
    6874 6092 4846
    6875 6434 5166
    6876 6658 5365
    6877 6865 5590
    6878 6785 5412
    6879 6811 5436
    6880 6830 5457
    6881 6811 5428
    6882 6810 5449
    6883 6775 5381
    6884 6742 5382
    6885 6714 5370
    6886 6602 5290
    6887 6388 5127
    6888 6383 5161
    6889 6197 5009
    6890 6121 4961
    6891 5757 4647
    6892 5691 4579
    6893 5821 4724
    6894 5796 4779
    6895 5822 4796
    6896 5972 4960
    6897 5972 4960
    6898 6073 5098
    6899 5984 4986
    6900 5995 5017
    6901 5560 4618
    6902 4810 3713
    6903 5325 4302
    6904 5706 4680
    6905 5770 4725
    6906 5835 4762
    6907 5436 3949
    6908 5587 4052
    6909 5689 4195
    6910 5682 4170
    6911 5651 4148
    6912 5660 4110
    6913 5554 3981
    6914 5738 4105
    6915 5738 4105
    6916 5786 4128
    6917 5782 4138
    6918 5612 4018
    6919 5650 4056
    6920 5564 4037
    6921 5679 4123
    6922 5887 4309
    6923 6075 4525
    6924 6188 4636
    6925 6103 4583
    6926 6103 4583
    6927 6190 4694
    6928 6299 4801
    6929 6329 4851
    6930 6294 4792
    6931 6160 4715
    6932 6123 4647
    6933 6102 4634
    6934 6257 4795
    6935 6169 4727
    6936 5994 4515
    6937 6037 4579
    6938 6107 4613
    6939 6093 4615
    6940 6182 4693
    6941 6110 4679
    6942 5951 4538
    6943 5851 4420
    6944 5881 4489
    6945 5856 4490
    6946 5867 4455
    6947 5862 4398
    6948 5904 4433
    6949 6016 4549
    6950 6096 4664
    6951 6123 4733
    6952 6053 4632
    6953 6074 4618
    6954 6258 4827
    6955 6207 4830
    6956 6106 4746
    6957 5914 4609
    6958 5802 4468
    6959 5656 4429
    6960 5651 4453
    6961 5603 4403
    6962 5918 4678
    6963 5916 4685
    6964 6127 4799
    6965 6300 4989
    6966 6512 5157
    6967 6812 5432
    6968 6742 5370
    6969 6826 5450
    6970 6806 5451
    6971 6821 5515
    6972 6686 5420
    6973 6592 5318
    6974 6383 5100
    6975 6430 5188
    6976 6257 5038
    6977 6190 4998
    6978 6001 4854
    6979 6001 4854
    6980 5818 4692
    6981 5873 4753
    6982 5886 4822
    6983 5807 4753
    6984 5686 4661
    6985 5767 4728
    6986 5716 4730
    6987 5601 4669
    6988 5299 4347
    6989 4833 3781
    6990 5259 4249
    6991 5611 4591
    6992 5719 4662
    6993 5466 3982
    6994 5593 4082
    6995 5686 4197
    6996 5721 4253
    6997 5623 4176
    6998 5511 4009
    6999 5329 3845
    7000 5486 3934
    7001 5683 4071
    7002 5646 4072
    7003 5672 4085
    7004 5718 4102
    7005 5731 4120
    7006 5733 4106
    7007 5617 4060
    7008 5617 4060
    7009 5593 4056
    7010 5682 4131
    7011 6067 4471
    7012 6047 4437
    7013 6166 4624
    7014 6168 4663
    7015 6209 4746
    7016 6286 4867
    7017 6109 4689
    7018 6037 4624
    7019 5989 4593
    7020 6056 4635
    7021 6056 4635
    7022 6176 4742
    7023 6143 4734
    7024 6043 4617
    7025 5989 4556
    7026 6041 4584
    7027 6093 4615
    7028 6045 4575
    7029 5978 4551
    7030 5963 4551
    7031 5960 4543
    7032 5895 4464
    7033 5894 4487
    7034 5832 4407
    7035 5827 4385
    7036 5840 4378
    7037 5918 4457
    7038 5925 4456
    7039 6176 4760
    7040 6053 4632
    7041 6122 4741
    7042 6204 4826
    7043 5994 4631
    7044 5831 4510
    7045 5554 4270
    7046 5491 4260
    7047 5313 4059
    7048 5745 4558
    7049 5745 4558
    7050 5833 4650
    7051 5971 4751
    7052 5986 4741
    7053 6300 4989
    7054 6279 4967
    7055 6291 4880
    7056 6329 4925
    7057 6686 5356
    7058 6606 5332
    7059 6544 5317
    7060 6544 5317
    7061 6435 5170
    7062 6114 4833
    7063 6270 5010
    7064 6257 5038
    7065 6178 5000
    7066 6052 4902
    7067 5923 4799
    7068 5820 4687
    7069 5695 4566
    7070 5768 4677
    7071 5702 4630
    7072 5695 4689
    7073 5600 4570
    7074 5644 4697
    7075 5299 4347
    7076 5221 4230
    7077 5458 4490
    7078 5222 3767
    7079 5131 3672
    7080 5372 3889
    7081 5331 3838
    7082 5490 4063
    7083 5176 3779
    7084 5289 3844
    7085 5377 3911
    7086 5541 4065
    7087 5619 4076
    7088 5619 4076
    7089 5640 4061
    7090 5645 4011
    7091 5630 3972
    7092 5778 4091
    7093 5813 4196
    7094 5918 4320
    7095 5820 4282
    7096 6087 4519
    7097 5971 4400
    7098 6002 4431
    7099 5710 4188
    7100 6006 4502
    7101 6006 4502
    7102 6147 4711
    7103 5990 4595
    7104 5948 4562
    7105 5845 4483
    7106 5966 4547
    7107 5962 4541
    7108 6060 4684
    7109 5923 4582
    7110 5876 4481
    7111 5962 4526
    7112 5915 4472
    7113 5987 4515
    7114 5989 4532
    7115 6012 4566
    7116 5984 4557
    7117 5917 4491
    7118 5830 4421
    7119 5818 4390
    7120 5725 4289
    7121 5744 4286
    7122 5588 4127
    7123 5668 4220
    7124 6030 4586
    7125 5994 4574
    7126 6020 4644
    7127 5912 4562
    7128 6053 4682
    7129 5764 4414
    7130 5680 4344
    7131 5283 3983
    7132 5064 3769
    7133 4846 3553
    7134 5496 4251
    7135 5761 4537
    7136 5981 4775
    7137 6032 4775
    7138 6127 4831
    7139 6168 4863
    7140 6196 4886
    7141 6266 4921
    7142 6266 4921
     [ reached 'max' / getOption("max.print") -- omitted 263406 rows ]

``` r
# Record start time
a <- Sys.time()

s2_collection |>
    raster_cube(v) |>
    select_bands(c( "B04", "B05"))  |>
  apply_pixel(c("(B05-B04)/(B05+B04)"), names="NDVI") |>
  write_tif() |>
  raster::stack() -> x
x
```

    class      : RasterStack 
    dimensions : 185375, 185484, 34384096500, 1  (nrow, ncol, ncell, nlayers)
    resolution : 100, 100  (x, y)
    extent     : -3178879, 15369521, -3103100, 15434400  (xmin, xmax, ymin, ymax)
    crs        : +proj=utm +zone=20 +south +datum=WGS84 +units=m +no_defs 
    names      : NDVI 

``` r
b <- Sys.time()
difftime(b, a)
```

    Time difference of 4.007104 mins

``` r
# Record start time
a <- Sys.time()

s2_collection |>
    raster_cube(v) |>
    select_bands(c("B04","B05"))  |>
  apply_pixel(c("(B05-B04)/(B05+B04)"), names="NDVI") |>
  stars::st_as_stars() -> y

b <- Sys.time()
difftime(b, a)
```

    Time difference of 1.429333 mins

``` r
y
```

    stars_proxy object with 1 attribute in 1 file(s):
    $NDVI
    [1] "[...]/file768518ea9065.nc:NDVI"

    dimension(s):
         from     to   offset delta                refsys point
    x       1 185484 -3178879   100 WGS 84 / UTM zone 20S    NA
    y       1 185375 15434400  -100 WGS 84 / UTM zone 20S    NA
    time    1      1       NA    NA               POSIXct FALSE
                          values x/y
    x                       NULL [x]
    y                       NULL [y]
    time [2021-05-01,2021-06-01)    

``` r
library(tmap)
tmap_mode("view")
```

``` r
# Record start time
a <- Sys.time()

items <- s %>%
    stac_search(collections = "sentinel-s2-l2a-cogs",
                bbox = c(-105.694362,   39.912886,  -105.052774,    40.262785),
                datetime = "2020-01-01/2022-12-31",
                limit = 500) %>% 
    post_request() 

S2.mask = image_mask("SCL", values=c(3,8,9))

col = stac_image_collection(items$features, asset_names = assets, 
                            property_filter = function(x) {x[["eo:cloud_cover"]] < 30})

v = cube_view(srs = "EPSG:4326",  extent = list(t0 = "2020-01-01", t1 = "2022-12-31",
              left = -105.694362, right = -105.052774,  top = 40.262785, bottom = 39.912886),
              dx = 0.001, dy = 0.001, dt = "P1M", aggregation = "median", resampling = "bilinear")

library(colorspace)
ndvi.col = function(n) {
  rev(sequential_hcl(n, "Green-Yellow"))
}
library(gdalcubes)
raster_cube(col, v, mask = S2.mask) %>%
    select_bands(c("B04", "B08")) %>%
    apply_pixel("(B08-B04)/(B08+B04)", "NDVI") %>%
    gdalcubes::animate(col = ndvi.col, zlim=c(-0.2,1), key.pos = 1, save_as = "anim.gif", fps = 4)
```

    [1] "/Users/ty/Documents/Github/hackathon2023_datacube/docs/code_for_building_cube/anim.gif"

``` r
b <- Sys.time()
difftime(b, a)
```

    Time difference of 4.630936 mins

``` r
y
```

    stars_proxy object with 1 attribute in 1 file(s):
    $NDVI
    [1] "[...]/file768518ea9065.nc:NDVI"

    dimension(s):
         from     to   offset delta                refsys point
    x       1 185484 -3178879   100 WGS 84 / UTM zone 20S    NA
    y       1 185375 15434400  -100 WGS 84 / UTM zone 20S    NA
    time    1      1       NA    NA               POSIXct FALSE
                          values x/y
    x                       NULL [x]
    y                       NULL [y]
    time [2021-05-01,2021-06-01)    

![](../code_for_building_cube/anim.gif)

``` r
# Record start time
a <- Sys.time()

s2_collection |>
    raster_cube(v) |>
    select_bands(c("B04","B05"))  |>
  apply_pixel(c("(B05-B04)/(B05+B04)"), names="NDVI") |>
  stars::st_as_stars() -> y

b <- Sys.time()
difftime(b, a)
```

    Time difference of 3.573504 secs

``` r
y
```

    stars object with 3 dimensions and 1 attribute
    attribute(s), summary of first 1e+05 cells:
          Min. 1st Qu. Median Mean 3rd Qu. Max.  NA's
    NDVI    NA      NA     NA  NaN      NA   NA 1e+05
    dimension(s):
         from  to offset  delta  refsys point
    x       1 642 -105.7  0.001  WGS 84    NA
    y       1 350  40.26 -0.001  WGS 84    NA
    time    1  36     NA     NA POSIXct FALSE
                                                      values x/y
    x                                                   NULL [x]
    y                                                   NULL [y]
    time [2020-01-01,2020-02-01),...,[2022-12-01,2023-01-01)    

``` r
items_2020 <- s %>%
    stac_search(collections = "sentinel-s2-l2a-cogs",
                bbox = c(-105.694362,   39.912886,  -105.052774,    40.262785),
                datetime = "2020-05-01/2020-06-30") %>% 
    post_request() 

items_2021 <- s %>%
    stac_search(collections = "sentinel-s2-l2a-cogs",
                bbox = c(-105.694362,   39.912886,  -105.052774,    40.262785),
                datetime = "2021-05-01/2021-06-30") %>% 
    post_request() 


col_2020 = stac_image_collection(items_2020$features, asset_names = assets)
col_2021 = stac_image_collection(items_2021$features, asset_names = assets)

v_2020 = cube_view(srs = "EPSG:32720",  extent = list(t0 = "2020-05-01", t1 = "2020-06-30",
              left = bbox_32720_boulder["xmin"], right = bbox_32720_boulder["xmax"],  top = bbox_32720_boulder["ymax"], bottom = bbox_32720_boulder["ymin"]),
              dx = 100, dy = 100, dt = "P1D", aggregation = "median", resampling = "bilinear")

v_2021 = cube_view(v_2020, extent = list(t0 = "2021-05-01", t1 = "2021-06-30"))


max_ndvi_mosaic <- function(col, v) {
    raster_cube(col, v) %>%
    select_bands(c("B04", "B08")) %>%
    apply_pixel(c("(B08-B04)/(B08+B04)"), names="NDVI") %>%
    reduce_time("max(NDVI)")
}

suppressPackageStartupMessages(library(stars))
max_ndvi_mosaic(col_2020, v_2020) -> maxndvi_2020

max_ndvi_mosaic(col_2021, v_2021)  -> maxndvi_2021

maxndvi_2021
maxndvi_2020

difference = maxndvi_2021 - maxndvi_2020
difference[difference > -0.15] = NA
names(difference) <- "Difference of max NDVI (2020 - 2019)"
```

``` r
flood_polygon_data3 <- glue("/vsizip/vsicurl/https://data.hydrosheds.org/file/hydrosheds-associated/gloric/GloRiC_v10_shapefile.zip/GloRiC_v10_shapefile/GloRiC_v10.shp") %>%
  st_read() %>%
  st_as_sf(coords = c("lon","lat"))

flood_polygon_data3
```

``` r
#st_read("/Users/ty/Downloads/GloRiC_v10_geodatabase/GloRiC_v10.gdb")

flood_polygon_data3 <- glue("/vsizip/vsicurl/https://data.hydrosheds.org/file/hydrosheds-associated/gloric/GloRiC_v10_geodatabase.zip/GloRiC_v10_geodatabase/GloRiC_v10.gdb") %>%
  st_read() %>%
  st_as_sf(coords = c("lon","lat"))

flood_polygon_data3
```

/Users/ty/Downloads/GloRiC_v10_geodatabase/GloRiC_v10.gdb

https://data.hydrosheds.org/file/hydrosheds-associated/gloric/GloRiC_v10_geodatabase.zip
