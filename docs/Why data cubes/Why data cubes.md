# Why data cubes?

Cibele Amaral, ESIIL Remote Sensing Scientist 2023-11-02

## Surviving in an ever-changing environment
> ### "**Environment**, the complex of physical, chemical, and biotic factors that act upon an organism or an ecological community and ultimately **determine its form and survival**." (Encyclopedia Britannica)
> ![image](https://github.com/CU-ESIIL/hackathon2023_datacube/assets/37226383/0ca54eff-4c3d-48d3-af43-eb478bf08cdb)
How much can an organism resist and adapt to a new environmental condition? Photo credit: Cibele Amaral

### 1. Global warming
> ![image](https://github.com/CU-ESIIL/hackathon2023_datacube/assets/37226383/fbe007b9-cf72-4e1a-bce8-bac629d6d6a7)
> Yearly surface temperature compared to the 20th-century average from 1880–2022. Blue bars indicate cooler-than-average years; red bars show warmer-than-average years. NOAA Climate.gov graph, based on data from the National Centers for Environmental Information.

### 2. Extreme events and disturbances
>![[extreme_events.jpg](https://hackmd.io/_uploads/SyFAurg7a.jpg)](https://github.com/CU-ESIIL/hackathon2023_datacube/blob/main/docs/assets/extreme_events.jpg)
>Diseases, droughts, wildfires, and floods are examples of ecological disturbances that are increasing due to an unbalanced Earth system.

### 3. Projections
> ![[projected_emissions_temperature_CSSP_lrg_1.png](https://hackmd.io/_uploads/Skj5JHeQT.png)](https://github.com/CU-ESIIL/hackathon2023_datacube/blob/main/docs/assets/projected_emissions_temperature_CSSP_lrg_1.png) 
> Projections indicate that the world is speeding towards a warmer climate and will experience more catastrophic extreme events.


## "How can we use data science to help organisms (populations and communities) to adapt to a wild future?"
 
## The era of revolutions and their integrated role in environmental sciences

### 1. Digital revolution: using cyberinfrastructure to collect, store, and process data & deliver information
> <a href="https://www.statista.com/statistics/273018/number-of-internet-users-worldwide/" rel="nofollow"><img src="https://www.statista.com/graphic/1/273018/number-of-internet-users-worldwide.jpg" alt="Statistic: Number of internet users worldwide from 2005 to 2022 (in millions) | Statista" style="width: 100%; height: auto !important; max-width:1000px;-ms-interpolation-mode: bicubic;"/></a><br />Find more statistics at  <a href="https://www.statista.com" rel="nofollow">Statista</a>.

### 2. Big data revolution: using open socio-environment data collected across scales 
> ![[multisource_data.png](https://hackmd.io/_uploads/Sy44MSg7a.png)](https://github.com/CU-ESIIL/hackathon2023_datacube/blob/main/docs/assets/multisource_data.png)
Image credit: Kathy Bogan, Jennifer Balch, Chelsea Nagy.
>
> ![[airqualityzm_2022.gif](https://hackmd.io/_uploads/rJ2Dvde7a.gif)](https://github.com/CU-ESIIL/hackathon2023_datacube/blob/main/docs/assets/airqualityzm_2022.gif)
> 
> Example of socio-enviromental data science: Hispanic, Asian, and Black and African American public school children attend schools with higher concentrations of air pollution than white students. Find more at [An Unequal Air Pollution Burden at School](https://earthobservatory.nasa.gov/images/152009/an-unequal-air-pollution-burden-at-school).

### 3. Artificial Intelligence revolution: using state-of-the-art AI models to understand organism-environment interactions, predict responses under various scenarios, and manage ecosystems properly 
> ![[digital-twin-banner-foreground.png](https://hackmd.io/_uploads/HJnY-Kxm6.png)](https://github.com/CU-ESIIL/hackathon2023_datacube/blob/main/docs/assets/digital-twin-banner-foreground.png)
AI models can help us to classify, cluster, forecast and identify outliers. They also provide us with data simulation and model emulation. Digital Twin representation, find more at [Digital Twin Overview](https://www.esri.com/en-us/digital-twin).

## "But... why data cubes?"
>
>![[xarray-datastructure.png](https://hackmd.io/_uploads/Bk6daFx76.png)](https://github.com/CU-ESIIL/hackathon2023_datacube/blob/main/docs/assets/xarray-datastructure.png)
>
>Data cube is the arrangement of relevant data in an n-dimensional array to support analytics. Photo credit: [x-array](https://xarray.dev/).

> ![[modeling.png](https://hackmd.io/_uploads/HklNZclXa.png)](https://github.com/CU-ESIIL/hackathon2023_datacube/blob/main/docs/assets/modeling.png)
> 
> We need paired samples to create a model and wall-to-wall layers to map predictions.

### 1. Spatial data formats, referencing systems, and resolutions

> ![[2022-05-02T09_23_37.798625.png](https://hackmd.io/_uploads/r1Jj_Ye7a.png)](https://github.com/CU-ESIIL/hackathon2023_datacube/blob/main/docs/assets/2022-05-02T09_23_37.798625.png)
>
> Vectors (points, lines, polygons) and rasters (wall-to-wall, gridded layers) are format types of spatial data. Photo credit: EDX.
>
> ![Sensors.png](https://github.com/CU-ESIIL/hackathon2023_datacube/blob/main/docs/assets/Sensors.png)
> Sensors operate from different orbits and collect data with different Instantaneous Field Of View (IFOV). These parameters result in data layers with varying spatial resolution.

### 2. Cutting-edge tools to process data on the fly and create data cubes
> - Clip to the Region of Interest (ROI)
> - Filter dates, times
> - Reproject to a standard Geographic Coordinate System (GCS) EPSG code
> - Resample every layer to a standard spatial resolution
> - Stack layers into a cube

>![[logo.svg](https://hackmd.io/_uploads/BJoafqeX6.svg)](https://github.com/CU-ESIIL/hackathon2023_datacube/blob/main/docs/assets/logo.svg)
>
> Find more at [gdalcubes](https://github.com/appelmar/gdalcubes#gdalcubes-)

### 3. Cloud-optimized Geospatial Format
> - Save the data (COG, zarr)
>
> ![[cogeo-formats-table.png](https://hackmd.io/_uploads/Bkjmr9gXa.png)](https://github.com/CU-ESIIL/hackathon2023_datacube/blob/main/docs/assets/cogeo-formats-table.png)
Cloud optimization enables efficient, on-the-fly access to geospatial data, offering several advantages such as reduced latency (1), scalability (2), flexibility (3), and cost-effectiveness (4). Find more at [Cloud-Optimized Geospatial Formats Guide](https://guide.cloudnativegeo.org/). 
