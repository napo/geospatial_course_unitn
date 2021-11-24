---
title: "Lesson 07"
permalink: /lessons/07-basic-operation-with-raster
excerpt: "Basic operations on raster data"
last_modified_at: 2021-11-10T06:48:03-01:21
header:
  teaser: /lessons/07_Basic_operations_on_raster_data_files/07_Basic_operations_on_raster_data_35_0.png
#redirect_from:
#  - /theme-setup/
toc: true
---
# Basic operations on raster data

based on rasterio
goals of the tutorial

- manage raster data
- manipulate raster with vector data

based on the open data of:
- [orthophotos](https://www.comune.trento.it/Aree-tematiche/Cartografia/Download/Ortofoto-2019) of Municipality of Trento 
- [DTM](http://www.territorio.provincia.tn.it/portal/server.pt/community/lidar/847/lidar/23954) - Autonomous Province of Trento

requirements
- python knowledge
- geopandas

status<br/>
*the world in a matrix*
{: .notice--success}

---
---

# Introduction

![](https://raw.githubusercontent.com/napo/geospatial_course_unitn/master/images/data_models_slide.jpg)

small summary:
view the [slides](https://docs.google.com/presentation/d/e/2PACX-1vT_my7vYOE2_xOdD-eZOtjxEFrbi1BfMcx_84jwomsVgI5wOfPxBO6sPNhxPtaLuEhrrkxmPbiv5Na0/pub?start=false&loop=false&delayms=3000) from the page 11 until 19


<img src="https://www.earthdatascience.org/images/earth-analytics/raster-data/raster-concept.png" width="640px" />



# Setup


```python
try:
  import geopandas as gpd
except ModuleNotFoundError as e:
  !pip install geopandas==0.10.1
  import geopandas as gpd

if gpd.__version__ != "0.10.1":
  !pip install -U geopandas==0.10.1
  import geopandas as gpd
```

    /home/napo/.local/lib/python3.9/site-packages/geopandas/_compat.py:111: UserWarning: The Shapely GEOS version (3.8.0-CAPI-1.13.1 ) is incompatible with the GEOS version PyGEOS was compiled with (3.9.1-CAPI-1.14.2). Conversions between both will be slow.
      warnings.warn(



```python
try:
  import rasterio as rio
except ModuleNotFoundError as e:
  !pip install rasterio==1.2.3
  import rasterio as rio

if rio.__version__ != "1.2.3":
  !pip install -U rasterio==1.2.3
  import rasterio as rio
```


```python
try:
  import owslib
except ModuleNotFoundError as e:
  !pip install owslib==0.25.0
  import owslib

if owslib.__version__ != "0.25.0":
  !pip install -U owslib==0.25.0
  import owslib
```


```python
import warnings
warnings.simplefilter("ignore")
```

# Data

Orthophoto of Trento 2019

https://www.comune.trento.it/Aree-tematiche/Cartografia/Download/Ortofoto-2019

- data acquisition: 3th October 2019
- scale 1:2.000
- resolution 1px = 10cm
- [tiff file](https://webapps.comune.trento.it/gis/raster/ortofoto_2019.tif) size 3,5Gb
- crs ETRS89 / UTM zone 32N - EPSG:[25832](https://epsg.io/25832)
- [tfw file](https://webapps.comune.trento.it/gis/raster/ortofoto_2019.tfw)

![](https://raw.githubusercontent.com/napo/geospatial_course_unitn/master/images/orthophoto_trento.png)




... file too big for our purpose

The municipality of Trento [offers](https://www.comune.trento.it/Aree-tematiche/Cartografia/Servizi-WMS-e-WFS) also a WMS service

Please check the lesson "[Retrieving data from spatial database infrastructures](https://napo.github.io/geospatial_course_unitn/lessons/retrieving_data_from_sdi)" 

![](https://raw.githubusercontent.com/napo/geospatial_course_unitn/master/images/ogc_services.png)

The end point of the WMS is http://webapps.comune.trento.it/ogc

---




```python
from owslib.wms import WebMapService
```


```python
wms_trento = "http://webapps.comune.trento.it/ogc"
wms = WebMapService(wms_trento)
```


```python
list(wms.contents)
```




    ['ogc_services',
     'ortofoto2009',
     'ortofoto2015',
     'ortofoto2016',
     'ortofoto2019',
     'ortofoto2015aldeno',
     'ortofoto2016infrarosso',
     'ortofoto2019infrarosso',
     'ct2000',
     'ct2000_colori',
     'carta_semplificata',
     'ombreDTM',
     'ombreDSM',
     'toponomastica',
     'grafo',
     'civici',
     'civici_principali',
     'toponimi',
     'prg_vigente',
     'adunata_alpini',
     'pric']




```python
wms['ortofoto2019'].crsOptions
```




    ['EPSG:3857', 'EPSG:4326', 'EPSG:25832']




```python
wms['ortofoto2019'].boundingBox
```




    (637271.0, 5083800.0, 684982.0, 5119620.0, 'EPSG:25832')




```python
request = wms.getmap(
    layers=['ortofoto2019'],
    srs='EPSG:25832',
    format='image/jpeg',
    bbox=(648300.0, 5090700.0, 677500.0, 5116000.0),
    size=(833,606)
    )
```


```python
from rasterio import MemoryFile
from rasterio.plot import show

```


```python
image = MemoryFile(request).open()
```


```python
show(image)
```


    
![png](07_Basic_operations_on_raster_data_files/07_Basic_operations_on_raster_data_18_0.png)
    





    <AxesSubplot:>



## Manage a raster

Here you can find a cutted version of the geotiff of the muncipality of Trento around the scientific hub area in Povo - Trento

(where the lessons of this course are held)

- [ortophoto](https://github.com/napo/geospatial_course_unitn/raw/master/data/raster/trento_scientifc_hub_povo.tif)
- [dtm](https://github.com/napo/geospatial_course_unitn/raw/master/data/raster/trento_scientifc_hub_povo_dtm.tif)

---


## investigate an orthophoto


```python
import urllib.request
url_download_orthophoto_scientific_hub_povo = 'https://github.com/napo/geospatial_course_unitn/raw/master/data/raster/trento_scientifc_hub_povo.tif'
file_scientific_hub_povo = "trento_scientifc_hub_povo.tif"
urllib.request.urlretrieve(url_download_orthophoto_scientific_hub_povo ,file_scientific_hub_povo) 
```




    ('trento_scientifc_hub_povo.tif', <http.client.HTTPMessage at 0x7ff53ac71040>)




```python
raster = rio.open(file_scientific_hub_povo)
```


```python
show(raster)
```


    
![png](07_Basic_operations_on_raster_data_files/07_Basic_operations_on_raster_data_23_0.png)
    





    <AxesSubplot:>



dimension of the image in pixel


```python
raster.width
```




    4761




```python
raster.height
```




    3900




```python
raster.crs
```




    CRS.from_epsg(25832)




```python
raster.res
```




    (0.09999999999999999, 0.09999999999999999)




```python
raster.bounds
```




    BoundingBox(left=666113.0, bottom=5103613.0, right=666589.1, top=5104003.0)




```python
raster.meta
```




    {'driver': 'GTiff',
     'dtype': 'uint8',
     'nodata': None,
     'width': 4761,
     'height': 3900,
     'count': 3,
     'crs': CRS.from_epsg(25832),
     'transform': Affine(0.09999999999999999, 0.0, 666113.0,
            0.0, -0.09999999999999999, 5104003.0)}




```python
raster.count
```




    3




```python
raster.indexes # the values start from 1 !
```




    (1, 2, 3)




```python
show((raster, 1), cmap='Reds')
```


    
![png](07_Basic_operations_on_raster_data_files/07_Basic_operations_on_raster_data_33_0.png)
    





    <AxesSubplot:>




```python
show((raster, 2), cmap='Greens')
```


    
![png](07_Basic_operations_on_raster_data_files/07_Basic_operations_on_raster_data_34_0.png)
    





    <AxesSubplot:>




```python
show((raster, 3), cmap='Blues')
```


    
![png](07_Basic_operations_on_raster_data_files/07_Basic_operations_on_raster_data_35_0.png)
    





    <AxesSubplot:>




```python
raster.colorinterp[0]
```




    <ColorInterp.red: 3>




```python
from rasterio.plot import show_hist
```


```python
show_hist(raster, bins=50, lw=0.0, stacked=False, alpha=0.3,histtype='stepfilled', title="Histogram")
```


    
![png](07_Basic_operations_on_raster_data_files/07_Basic_operations_on_raster_data_38_0.png)
    


## investigate an DTM file


```python
url_download_dtm_scientific_hub_povo = 'https://github.com/napo/geospatial_course_unitn/raw/master/data/raster/trento_scientifc_hub_povo_dtm.asc'
dtm = "trento_scientifc_hub_povo_dtm.asc"
urllib.request.urlretrieve(url_download_dtm_scientific_hub_povo ,dtm) 
```




    ('trento_scientifc_hub_povo_dtm.asc',
     <http.client.HTTPMessage at 0x7ff539185490>)




```python
url_download_dtm_scientific_hub_povo_prj = 'https://github.com/napo/geospatial_course_unitn/raw/master/data/raster/trento_scientifc_hub_povo_dtm.prj'
dtm_prj = "trento_scientifc_hub_povo_dtm.prj"
urllib.request.urlretrieve(url_download_dtm_scientific_hub_povo_prj ,dtm_prj) 
```




    ('trento_scientifc_hub_povo_dtm.prj',
     <http.client.HTTPMessage at 0x7ff539156b80>)




```python
dtm = "trento_scientifc_hub_povo_dtm.asc"
```


```python
raster_dtm = rio.open(dtm)
```


```python
show(raster_dtm)
```


    
![png](07_Basic_operations_on_raster_data_files/07_Basic_operations_on_raster_data_44_0.png)
    





    <AxesSubplot:>




```python
show(raster_dtm,cmap="Greys")
```


    
![png](07_Basic_operations_on_raster_data_files/07_Basic_operations_on_raster_data_45_0.png)
    





    <AxesSubplot:>




```python
raster_dtm.crs
```



    CRS.from_wkt('PROJCS["ETRS89 / UTM zone 32N",GEOGCS["ETRS89",DATUM["European_Terrestrial_Reference_System_1989",SPHEROID["GRS 1980",6378137,298.257222101,AUTHORITY["EPSG","7019"]],AUTHORITY["EPSG","6258"]],PRIMEM["Greenwich",0],UNIT["Degree",0.0174532925199433]],PROJECTION["Transverse_Mercator"],PARAMETER["latitude_of_origin",0],PARAMETER["central_meridian",9],PARAMETER["scale_factor",0.9996],PARAMETER["false_easting",500000],PARAMETER["false_northing",0],UNIT["metre",1,AUTHORITY["EPSG","9001"]],AXIS["Easting",EAST],AXIS["Northing",NORTH]]')




```python
raster_dtm.meta
```




    {'driver': 'AAIGrid',
     'dtype': 'float32',
     'nodata': None,
     'width': 500,
     'height': 410,
     'count': 1,
     'crs': CRS.from_wkt('PROJCS["ETRS89 / UTM zone 32N",GEOGCS["ETRS89",DATUM["European_Terrestrial_Reference_System_1989",SPHEROID["GRS 1980",6378137,298.257222101,AUTHORITY["EPSG","7019"]],AUTHORITY["EPSG","6258"]],PRIMEM["Greenwich",0],UNIT["Degree",0.0174532925199433]],PROJECTION["Transverse_Mercator"],PARAMETER["latitude_of_origin",0],PARAMETER["central_meridian",9],PARAMETER["scale_factor",0.9996],PARAMETER["false_easting",500000],PARAMETER["false_northing",0],UNIT["metre",1,AUTHORITY["EPSG","9001"]],AXIS["Easting",EAST],AXIS["Northing",NORTH]]'),
     'transform': Affine(1.0, 0.0, 666100.6735466761,
            0.0, -1.0, 5104013.23583161)}



```python
raster_dtm.res
```



    (1.0, 1.0)




```python
raster_dtm.count
```




    1




```python
show(raster_dtm, cmap='terrain')
```


    
![png](07_Basic_operations_on_raster_data_files/07_Basic_operations_on_raster_data_50_0.png)
    




```python
show(raster)
```


    
![png](07_Basic_operations_on_raster_data_files/07_Basic_operations_on_raster_data_51_0.png)
    




```python
show_hist(raster_dtm, bins=50, lw=0.0, stacked=False, alpha=0.3,histtype='stepfilled', title="Histogram")
```


    
![png](07_Basic_operations_on_raster_data_files/07_Basic_operations_on_raster_data_52_0.png)
    



```python
data = raster_dtm.read(1)
```


```python
type(data)
```




    numpy.ndarray



Which altitude is?

<img src="https://raw.githubusercontent.com/napo/geospatial_course_unitn/master/images/trento_scientific_hub_3d.jpg" width="640px" />

<br/>
this image is generated with QGIS and the plugin qgis2threejs




```python
data.mean()
```




    383.85464




```python
data.min()
```




    326.0




```python
data.max()
```




    469.0



## find the value on a given point

Example:<br/>
the position of "Via Sommarive 18, Trento" (the address of FBK)


```python
import shapely
import pyproj
from geopy.geocoders import Nominatim
from shapely.ops import transform
```

... identify the point wiht a geocoder


```python
geolocator = Nominatim(user_agent="geospatial course")
location = geolocator.geocode("Via Sommarive 18, Trento")
```


```python
x = location.longitude
y = location.latitude
```


```python
y
```




    46.06692965



transform the coordinate from EPSG 4326 to 25832


```python
wgs84 = pyproj.CRS('EPSG:4326')
crs_dtm = pyproj.CRS('EPSG:25832')
projection_transform = pyproj.Transformer.from_crs(wgs84, crs_dtm, always_xy=False).transform
```


```python
point_location = shapely.geometry.Point(y,x)
```


```python
point_location_crs_dtm = transform(projection_transform,point_location)
```


```python
x = point_location_crs_dtm.x
y = point_location_crs_dtm.y
```


```python
y
```




    5103733.953099297




```python
row,col = rio.transform.rowcol(raster_dtm.transform,(x),(y))
```


```python
col
```




    282



Identify the value


```python
data[row][col]
```




    381.0



## Resampling

Downsampling to 1/5 of the resolution can be done withupscale_factor = 1/5


```python
raster_dtm.profile
```

    {'driver': 'AAIGrid', 'dtype': 'float32', 'nodata': None, 'width': 500, 'height': 410, 'count': 1, 'crs': CRS.from_wkt('PROJCS["ETRS89 / UTM zone 32N",GEOGCS["ETRS89",DATUM["European_Terrestrial_Reference_System_1989",SPHEROID["GRS 1980",6378137,298.257222101,AUTHORITY["EPSG","7019"]],AUTHORITY["EPSG","6258"]],PRIMEM["Greenwich",0],UNIT["Degree",0.0174532925199433]],PROJECTION["Transverse_Mercator"],PARAMETER["latitude_of_origin",0],PARAMETER["central_meridian",9],PARAMETER["scale_factor",0.9996],PARAMETER["false_easting",500000],PARAMETER["false_northing",0],UNIT["metre",1,AUTHORITY["EPSG","9001"]],AXIS["Easting",EAST],AXIS["Northing",NORTH]]'), 'transform': Affine(1.0, 0.0, 666100.6735466761,
           0.0, -1.0, 5104013.23583161), 'tiled': False}




```python
from rasterio.enums import Resampling

upscale_factor = 1/5

# resample data to target shape
data_s = raster_dtm.read(
        out_shape=(
            raster_dtm.count,
            int(raster_dtm.width * upscale_factor),
            int(raster_dtm.height * upscale_factor)
        ),
        resampling=Resampling.bilinear
    )
profile =raster_dtm.profile
profile.update(dtype=rio.uint8, count=1, compress='lzw')

with rio.open('resampled_area.tif', 'w', **profile) as dst:
  dst.write(data_s.astype(rio.uint8))
```


```python
#uncomment if you want download with colab
#from google.colab import files
#files.download('resampled_area.tif')
```


```python
show(data_s, cmap='terrain')
```


    
![png](07_Basic_operations_on_raster_data_files/07_Basic_operations_on_raster_data_80_0.png)
    





```python
show_hist(data_s, bins=50, lw=0.0, stacked=False, alpha=0.3,histtype='stepfilled', title="Histogram")
```


    
![png](07_Basic_operations_on_raster_data_files/07_Basic_operations_on_raster_data_81_0.png)
    



```python
show_hist(raster_dtm, bins=50, lw=0.0, stacked=False, alpha=0.3,histtype='stepfilled', title="Histogram")
```


    
![png](07_Basic_operations_on_raster_data_files/07_Basic_operations_on_raster_data_82_0.png)
    


# Masking / clipping raster

One common task in raster processing is to clip raster files based on a Polygon

We start to extract the polygon of the area held by Bruno Kessler Foundation (research center in front of the university)



![](https://raw.githubusercontent.com/napo/geospatial_course_unitn/master/images/fbk_overpassturbo.png)

[query with overpass-turbo](http://overpass-turbo.eu/s/Zw8)

[geojson exported](https://raw.githubusercontent.com/napo/geospatial_course_unitn/master/data/openstreetmap/boundary_fbk_povo.geojson
)




```python
import geopandas as gpd
```


```python
fbk = gpd.read_file("https://raw.githubusercontent.com/napo/geospatial_course_unitn/master/data/openstreetmap/boundary_fbk_povo.geojson")
```


```python
fbk.plot()
```




    
![png](07_Basic_operations_on_raster_data_files/07_Basic_operations_on_raster_data_87_1.png)
    



```python
fbk
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>@id</th>
      <th>addr:city</th>
      <th>addr:housenumber</th>
      <th>addr:postcode</th>
      <th>addr:street</th>
      <th>amenity</th>
      <th>name</th>
      <th>website</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>way/813142998</td>
      <td>way/813142998</td>
      <td>Povo</td>
      <td>18</td>
      <td>38123</td>
      <td>Via Sommarive</td>
      <td>research_institute</td>
      <td>Fondazione Bruno Kessler</td>
      <td>https://www.fbk.eu/</td>
      <td>POLYGON ((11.15071 46.06837, 11.15065 46.06832...</td>
    </tr>
  </tbody>
</table>
</div>




```python
def getFeatures(gdf):
    """Function to parse features from GeoDataFrame in such a manner that rasterio wants them"""
    import json
    return [json.loads(gdf.to_json())['features'][0]['geometry']]
```


```python
coords = getFeatures(fbk.to_crs(epsg=25832))
```


```python
coords
```




    [{'type': 'Polygon',
      'coordinates': [[[666327.6121890642, 5103893.127195438],
        [666323.144271185, 5103886.635365136],
        [666321.3931118533, 5103876.125343578],
        [..]
        [666339.1182247801, 5103896.351484859],
        [666327.6121890642, 5103893.127195438]]]}]




```python
from rasterio.mask import mask
```


```python
out_img, out_transform = mask(raster, coords, crop=True)
```


```python
out_meta = raster.meta
```


```python
show(out_img)
```


    
![png](07_Basic_operations_on_raster_data_files/07_Basic_operations_on_raster_data_95_0.png)
    




```python
out_meta.update({"driver": "GTiff",
                 "height": out_img.shape[1],
                 "width": out_img.shape[2],
                 "transform": out_transform})

with rio.open("fbk_orthophoto.tif", "w", **out_meta) as dest:
    dest.write(out_img)

```

## raster to vector


```python
resampled_area = rio.open('resampled_area.tif')
```


```python
from rasterio.features import shapes
mask = None
image = resampled_area.read(1) # first band
results = (
  {'properties': {'raster_val': v}, 'geometry': s}
  for i, (s, v) 
    in enumerate(
      shapes(image, mask=mask, transform=resampled_area.transform)))
```


```python
geoms = list(results)
%time
```

    CPU times: user 3 µs, sys: 0 ns, total: 3 µs
    Wall time: 7.15 µs



```python
geoms
```

    [{'properties': {'raster_val': 77.0},
      'geometry': {'type': 'Polygon',
       'coordinates': [[(666100.6735466761, 5104013.23583161),
         (666100.6735466761, 5104008.23583161),
         (666107.6735466761, 5104008.23583161),
         (666107.6735466761, 5104013.23583161),
         (666100.6735466761, 5104013.23583161)]]}},
    
    [...]
    
     {'properties': {'raster_val': 126.0},
      'geometry': {'type': 'Polygon',
       'coordinates': [[(666381.6735466761, 5103775.23583161),
         (666381.6735466761, 5103754.23583161),
         (666393.6735466761, 5103754.23583161),
         (666393.6735466761, 5103750.23583161),
    [...]
         (666387.6735466761, 5103775.23583161),
         (666381.6735466761, 5103775.23583161)]]}},
     ...]




```python
gpd_polygonized_raster  = gpd.GeoDataFrame.from_features(geoms)
%time
```

    CPU times: user 2 µs, sys: 1e+03 ns, total: 3 µs
    Wall time: 4.53 µs



```python
gpd_polygonized_raster.shape
```




    (1330, 2)




```python
gpd_polygonized_raster.geometry.area.max()
```




    5943.0




```python
gpd_polygonized_raster.geometry[1]
```




    
![svg](07_Basic_operations_on_raster_data_files/07_Basic_operations_on_raster_data_105_0.svg)
    



# Exercises

- clip the area with the shape of Polo Ferrari (in front on FBK)
- create the altitude profile of the street "Via Sommarive"
- find the area FBK in the WMS of municipality of Trento - layer "Carta Tecnica 1:2.000 alta risoluzione" and vectorize it
- identify the shortest path from the pizzeria Oro Stube until the main entrance of FBK over the DTM

