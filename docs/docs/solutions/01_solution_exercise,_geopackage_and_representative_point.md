---
layout: default
title: 01 - GIS concepts - solution to the exercise
parent: Solutions
nav_order: 1
permalink: /lessons/solution_exercise_1
---
*Exercise after the lesson of 25 September 2020*
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---
# Setup
```
!pip install geopandas
```

    Requirement already satisfied: geopandas in /usr/local/lib/python3.6/dist-packages (0.8.1)
    Requirement already satisfied: shapely in /usr/local/lib/python3.6/dist-packages (from geopandas) (1.7.1)
    Requirement already satisfied: pyproj>=2.2.0 in /usr/local/lib/python3.6/dist-packages (from geopandas) (2.6.1.post1)
    Requirement already satisfied: pandas>=0.23.0 in /usr/local/lib/python3.6/dist-packages (from geopandas) (1.1.2)
    Requirement already satisfied: fiona in /usr/local/lib/python3.6/dist-packages (from geopandas) (1.8.17)
    Requirement already satisfied: numpy>=1.15.4 in /usr/local/lib/python3.6/dist-packages (from pandas>=0.23.0->geopandas) (1.18.5)
    Requirement already satisfied: python-dateutil>=2.7.3 in /usr/local/lib/python3.6/dist-packages (from pandas>=0.23.0->geopandas) (2.8.1)
    Requirement already satisfied: pytz>=2017.2 in /usr/local/lib/python3.6/dist-packages (from pandas>=0.23.0->geopandas) (2018.9)
    Requirement already satisfied: six>=1.7 in /usr/local/lib/python3.6/dist-packages (from fiona->geopandas) (1.15.0)
    Requirement already satisfied: munch in /usr/local/lib/python3.6/dist-packages (from fiona->geopandas) (2.5.0)
    Requirement already satisfied: click-plugins>=1.0 in /usr/local/lib/python3.6/dist-packages (from fiona->geopandas) (1.1.1)
    Requirement already satisfied: click<8,>=4.0 in /usr/local/lib/python3.6/dist-packages (from fiona->geopandas) (7.1.2)
    Requirement already satisfied: cligj>=0.5 in /usr/local/lib/python3.6/dist-packages (from fiona->geopandas) (0.5.0)
    Requirement already satisfied: attrs>=17 in /usr/local/lib/python3.6/dist-packages (from fiona->geopandas) (20.2.0)

---
# Exercise
- load the shapefile of ISTAT with the information of the provinces
- filter it for an italian provice at your choice (eg. Trento)
- plot it
- identify the cities of the province selected with the biggest and smallest area
- extract all the centroids of the areas expressed in WGS84
- extract a rappresenative point for the area of a municipality in WGS84<br/>suggestion: *.representative_point()*
- save the points in a GeoJSON file
- calculate the distance on the geodentic between the municipaly with the big area and smallest area by using the centroid
---

## Solutions and learning objectives
* repeat the concepts on the previous lesson
* introduce geopackage
* centroid vs representative point

Import of the packages


```
import geopandas as gpd
from google.colab import files
```

### load the shapefile of ISTAT with the information of the provinces

This request is replaced with the use of geopackage

In the course material a geopackage file is available with all the shapefiles of the administrative limits of ISTAT (2020) with generalized geometries

[download](https://github.com/napo/geospatial_course_unitn/raw/master/data/administrative_units_italy_2020/istat_administrative_units_2020.gpkg)

for convenience, download the file with "wget" from the command line on Linux



```
!wget https://github.com/napo/geospatial_course_unitn/raw/master/data/administrative_units_italy_2020/istat_administrative_units_2020.gpkg
```

    --2020-10-01 11:36:19--  https://github.com/napo/geospatial_course_unitn/raw/master/data/administrative_units_italy_2020/istat_administrative_units_2020.gpkg
    Resolving github.com (github.com)... 192.30.255.113
    Connecting to github.com (github.com)|192.30.255.113|:443... connected.
    HTTP request sent, awaiting response... 302 Found
    Location: https://raw.githubusercontent.com/napo/geospatial_course_unitn/master/data/administrative_units_italy_2020/istat_administrative_units_2020.gpkg [following]
    --2020-10-01 11:36:19--  https://raw.githubusercontent.com/napo/geospatial_course_unitn/master/data/administrative_units_italy_2020/istat_administrative_units_2020.gpkg
    Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 151.101.0.133, 151.101.64.133, 151.101.128.133, ...
    Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|151.101.0.133|:443... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 23396352 (22M) [application/octet-stream]
    Saving to: ‘istat_administrative_units_2020.gpkg’
    
    istat_administrativ 100%[===================>]  22.31M  66.6MB/s    in 0.3s    
    
    2020-10-01 11:36:21 (66.6 MB/s) - ‘istat_administrative_units_2020.gpkg’ saved [23396352/23396352]
    


# Geopackage
![](https://www.ogc.org/pub/www/files/blog/Geopackage_layers.png)

[GeoPackage](http://opengeospatial.github.io/e-learning/geopackage/text/basic-index.html) is used for storing and accessing:
* Vector feature data
* Imagery tile matrix sets
* Raster map tile matrix sets
* non-spatial tabular data
* Metadata that describes other stored data

To have a look at the structure of the files, download the files and open them using the basic SQLite3 command-line utility.

```
sqlite3 istat_administrative_units_2020.gpkg
```
```
SQLite version 3.31.1 2020-01-27 19:55:54
Enter ".help" for usage hints.
sqlite> .table
```
```
gpkg_contents                     rtree_macroregions_geom_rowid   
gpkg_extensions                   rtree_municipalities_geom       
gpkg_geometry_columns             rtree_municipalities_geom_node  
gpkg_ogr_contents                 rtree_municipalities_geom_parent
gpkg_spatial_ref_sys              rtree_municipalities_geom_rowid 
gpkg_tile_matrix                  rtree_provincies_geom           
gpkg_tile_matrix_set              rtree_provincies_geom_node      
macroregions                      rtree_provincies_geom_parent    
municipalities                    rtree_provincies_geom_rowid     
provincies                        rtree_regions_geom              
regions                           rtree_regions_geom_node         
rtree_macroregions_geom           rtree_regions_geom_parent       
rtree_macroregions_geom_node      rtree_regions_geom_rowid        
rtree_macroregions_geom_parent 
```

```
sqlite> select * from macroregions;
```
```
1|GP|1|Nord-Ovest
2|GP|2|Nord-Est
3|GP|3|Centro
4|GP|4|Sud
5|GP|5|Isole
```
```
sqlite> .q
```

Geopandas can manage geopackage by using [fiona](https://github.com/Toblerity/Fiona)


```
import fiona
```


```
fiona.supported_drivers
```




    {'ARCGEN': 'r',
     'AeronavFAA': 'r',
     'BNA': 'rw',
     'CSV': 'raw',
     'DGN': 'raw',
     'DXF': 'rw',
     'ESRI Shapefile': 'raw',
     'ESRIJSON': 'r',
     'GML': 'rw',
     'GPKG': 'raw',
     'GPSTrackMaker': 'rw',
     'GPX': 'rw',
     'GeoJSON': 'raw',
     'GeoJSONSeq': 'rw',
     'Idrisi': 'r',
     'MapInfo File': 'raw',
     'OGR_GMT': 'rw',
     'OGR_PDS': 'r',
     'OpenFileGDB': 'r',
     'S57': 'r',
     'SEGY': 'r',
     'SUA': 'r',
     'TopoJSON': 'r'}



```
'GPKG': 'raw',
```
**raw** => **r**ead **a**ppend **w**rite

geopandas can:
* **r**ead *geopackage* files
* **a**append data to a *geopackage* file
* **w**rite data to a *geopackage* file

geopackage can store more layers, so we have to investigate the contents


```
fiona.listlayers('istat_administrative_units_2020.gpkg')
```




    ['municipalities', 'provincies', 'regions', 'macroregions']




```
provincies = gpd.read_file("istat_administrative_units_2020.gpkg",layer="provincies")
```


```
provincies.head(3)
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
      <th>COD_RIP</th>
      <th>COD_REG</th>
      <th>COD_PROV</th>
      <th>COD_CM</th>
      <th>COD_UTS</th>
      <th>DEN_PROV</th>
      <th>DEN_CM</th>
      <th>DEN_UTS</th>
      <th>SIGLA</th>
      <th>TIPO_UTS</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>1</td>
      <td>2</td>
      <td>0</td>
      <td>2</td>
      <td>Vercelli</td>
      <td>-</td>
      <td>Vercelli</td>
      <td>VC</td>
      <td>Provincia</td>
      <td>MULTIPOLYGON (((438328.612 5087208.215, 439028...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>1</td>
      <td>3</td>
      <td>0</td>
      <td>3</td>
      <td>Novara</td>
      <td>-</td>
      <td>Novara</td>
      <td>NO</td>
      <td>Provincia</td>
      <td>MULTIPOLYGON (((460929.542 5076320.298, 461165...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>201</td>
      <td>201</td>
      <td>-</td>
      <td>Torino</td>
      <td>Torino</td>
      <td>TO</td>
      <td>Citta metropolitana</td>
      <td>MULTIPOLYGON (((411015.006 5049970.983, 411266...</td>
    </tr>
  </tbody>
</table>
</div>




```
provincies.columns
```




    Index(['COD_RIP', 'COD_REG', 'COD_PROV', 'COD_CM', 'COD_UTS', 'DEN_PROV',
           'DEN_CM', 'DEN_UTS', 'SIGLA', 'TIPO_UTS', 'geometry'],
          dtype='object')



**COD_RIP**<br/>
*codice ripartizione*<br/>
numeric code of the macroregion of belonging

**COD_REG**<br/>
*codice regione*<br/>
numeric code of the region of belonging

**COD_PROV**<br/>
*codice provincia*<br/>
numeric code of the region of belonging

**COD_CM**<br/>
*codice comune*<br/>
Istat code of the metropolitan city (three characters in
reference to all official statistics are
numeric format) obtained by adding the value 200 to
adopted the statistical codes of the cities
corresponding code of the province.

**COD_UTS**<br/>
*codice unità territoriali sovracomunali*<br/>
Numeric code that uniquely identifies the Units
territorial supra-municipal on the national territory.

**DEN_PROV**<br/>
*denominazione provincia*<br/>
name of the province

**DEN_CM**<br/>
*denominazione città metropolitana*<br/>
name of the metropolitan city

**DEN_UTS**<br/>
*denominazione unità territoriale sovracomunale*<br/>
Denomination of the supra-municipal territorial units.

**SIGLA**<br/>
*sigla*<br/>
abbreviation

**TIPO_UTS**<br/>
*tipologia unità territoriale sovracomunale*<br/>
kind of supra-municipal territorial units.



## filter it for an italian provice at your choice (eg. Trento)


```
trentino = provincies[provincies['DEN_PROV']=='Trento']
```


```
trentino
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
      <th>COD_RIP</th>
      <th>COD_REG</th>
      <th>COD_PROV</th>
      <th>COD_CM</th>
      <th>COD_UTS</th>
      <th>DEN_PROV</th>
      <th>DEN_CM</th>
      <th>DEN_UTS</th>
      <th>SIGLA</th>
      <th>TIPO_UTS</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>24</th>
      <td>2</td>
      <td>4</td>
      <td>22</td>
      <td>0</td>
      <td>22</td>
      <td>Trento</td>
      <td>-</td>
      <td>Trento</td>
      <td>TN</td>
      <td>Provincia autonoma</td>
      <td>MULTIPOLYGON (((716676.337 5153931.623, 716029...</td>
    </tr>
  </tbody>
</table>
</div>



## plot it


```
trentino.plot()
```




    <matplotlib.axes._subplots.AxesSubplot at 0x7fbc28c6ca90>




![png](01_solution_exercise%2C_geopackage_and_representative_point_files/01_solution_exercise%2C_geopackage_and_representative_point_23_1.png)


## identify the municipalities of the province selected with the biggest and smallest area

this means we need to use another layer / dataset


```
municipalities = gpd.read_file("istat_administrative_units_2020.gpkg",layer="municipalities")
```


```
municipalities.columns
```




    Index(['COD_RIP', 'COD_REG', 'COD_PROV', 'COD_CM', 'COD_UTS', 'PRO_COM',
           'PRO_COM_T', 'COMUNE', 'COMUNE_A', 'CC_UTS', 'geometry'],
          dtype='object')



**COD_RIP**<br/>
*codice ripartizione*<br/>
numeric code of the macroregion of belonging

**COD_REG**<br/>
*codice regione*<br/>
numeric code of the region of belonging

**COD_PROV**<br/>
*codice provincia*<br/>
numeric code of the region of belonging

**COD_CM**<br/>
*codice comune*<br/>
unique numeric identification code of the municipality within the province of belonging

**COD_UTS**<br/>
*codice unità territoriali sovracomunali*<br/>
Numeric code that uniquely identifies the Units
territorial supra-municipal on the national territory.

**PRO_COM**<br/>
*provincia comune**<br/>
Numeric code that uniquely identifies the Municipality
on the national territory. (= COD_PROV & COD_COM) 

**PRO_COM_T**<br/>
*provincia comune territorio**<br/>
Alphanumeric code that uniquely identifies the
Municipality on the national territory.<br/>
Like PRO_COM but definied in 6 fixed characters.

**COMUNE**<br/>
*comune*<br/>
Name of the Municipality

**COMUNE_A**<br/>
*comune alternativa*<br/>
Name of the Municipality in a language other than Italian

**CC_UTS**<br/>
*comune capoluogo*<br/>
Provincial capital or metropolitan city<br/>
1 => True<br/>
2 => False



the **COD_PROV** of the Province of Trento is **22**


```
# filter the province
municipalities_province_trento = municipalities[municipalities.COD_PROV==22]
```


```
# plot it
municipalities_province_trento.plot()
```




    <matplotlib.axes._subplots.AxesSubplot at 0x7fbc259d2a90>




![png](01_solution_exercise%2C_geopackage_and_representative_point_files/01_solution_exercise%2C_geopackage_and_representative_point_30_1.png)


## identify the cities of the province selected with the biggest and smallest area

**CAUTION**:

we are using generalized boundaries !!!

finding the max area


```
max_area = municipalities_province_trento.geometry.area.max()
```


```
max_area
```




    199625902.75494903



finding the min area


```
min_area = municipalities_province_trento.geometry.area.min()
```


```
min_area
```




    1652100.49475859



... you can obtain the same in another way (combination of the requests)


```
municipalities[municipalities.COD_PROV==22].geometry.area.min()
```




    1652100.49475859



### identify the municipality with the biggest area





```
maxarea_municipality_trentino = provincia_trento[provincia_trento.geometry.area == max_area]
```


```
maxarea_municipality_trentino.PRO_COM_T
```




    3275    022245
    Name: PRO_COM_T, dtype: object




```
maxarea_municipality_trentino.COMUNE
```




    3275    Primiero San Martino di Castrozza
    Name: COMUNE, dtype: object



the municipality with the bigger area is **Primiero San Martino di Castrozza** (022245)


```
maxarea_municipality_trentino.plot()
```




    <matplotlib.axes._subplots.AxesSubplot at 0x7fbc259d34a8>




![png](01_solution_exercise%2C_geopackage_and_representative_point_files/01_solution_exercise%2C_geopackage_and_representative_point_45_1.png)


### identify the municipality with the smallest area



```
minarea_municipality_trentino = provincia_trento[provincia_trento.geometry.area == min_area]
```


```
minarea_municipality_trentino.COMUNE
```




    1309    Carzano
    Name: COMUNE, dtype: object




```
minarea_municipality_trentino.PRO_COM_T
```




    1309    022043
    Name: PRO_COM_T, dtype: object



the municipality with the smallest area is **Carzano** (022243)


```
minarea_municipality_trentino.plot()
```




    <matplotlib.axes._subplots.AxesSubplot at 0x7fbc25a2e860>




![png](01_solution_exercise%2C_geopackage_and_representative_point_files/01_solution_exercise%2C_geopackage_and_representative_point_51_1.png)


## extract all the centroids of the areas expressed in WGS84


```
maxarea_municipality_trentino.geometry.centroid.to_crs(epsg=4326)
```




    3275    POINT (11.82683 46.24190)
    dtype: geometry




```
minarea_municipality_trentino.geometry.centroid.to_crs(epsg=4326)
```




    1309    POINT (11.49436 46.07497)
    dtype: geometry



## extract a rappresenative point for the area of the smallest and bigger municipality in WGS84


```
representative_point_minarea_municipality = minarea_municipality_trentino.geometry.representative_point()
```


```
representative_point_maxarea_municipality = maxarea_municipality_trentino.geometry.representative_point()
```


```
representative_point_minarea_municipality.to_crs(epsg=4326)
```




    1309    POINT (11.49403 46.07235)
    dtype: geometry




```
representative_point_maxarea_municipality.to_crs(epsg=4326)
```




    3275    POINT (11.83795 46.24792)
    dtype: geometry



## save the points in a GeoJSON file
we can save each point in geojson 


```
representative_point_maxarea_municipality.to_crs(epsg=4326).to_file("point.geojson",driver="GeoJSON")
```


```
# check if the file exist with a 'ls' command on the remote linux machine
!ls point.geojson
```

    point.geojson


... but we need to create an only one file with all the data in a geojson file


```
points = representative_point_maxarea_municipality.append(representative_point_minarea_municipality)
```


```
points.to_crs(epsg=4326).to_file("points.geojson",driver="GeoJSON")
```


```
#donwload the file
files.download("points.geojson")
```


    <IPython.core.display.Javascript object>



    <IPython.core.display.Javascript object>


**tip**:<br/>
you can download, open with [geojson.io](https://geojson.io) and create a [gist resource](https://gist.github.com/napo/549a9a452f98055bb3210bf734d58a94) to see the points

## calculate the distance on the geodentic between the municipaly with the big area and smallest area by using the centroid

... maybe you are ready to work on this way .. but ...


```
maxarea_municipality_trentino.geometry.centroid.distance(minarea_municipality_trentino.geometry.centroid)
```

    /usr/local/lib/python3.6/dist-packages/geopandas/base.py:39: UserWarning: The indices of the two GeoSeries are different.
      warn("The indices of the two GeoSeries are different.")





    1309   NaN
    3275   NaN
    dtype: float64



there is a warning
```
The indices of the two GeoSeries are different.
```
... and we have 2 geodataframe with a row for each so ...


```
maxarea_municipality_trentino.index
```




    Int64Index([3275], dtype='int64')




```
idx_minarea = minarea_municipality_trentino.index[0]
```


```
idx_maxarea = maxarea_municipality_trentino.index[0]
```


```
maxarea_municipality_trentino.geometry[idx_maxarea].centroid.distance(minarea_municipality_trentino.geometry[idx_minarea].centroid)
```




    31686.637969815347



the distance is in meters due the CRS used on the dataset 

# Why a representative point?

Where is the centroid of Liguria?



```
regions = gpd.read_file("istat_administrative_units_2020.gpkg",layer="regions")
```


```
regions.DEN_REG.unique()
```




    array(['Piemonte', "Valle d'Aosta", 'Lombardia', 'Veneto',
           'Trentino-Alto Adige', 'Friuli Venezia Giulia', 'Liguria',
           'Emilia-Romagna', 'Toscana', 'Umbria', 'Marche', 'Abruzzo',
           'Molise', 'Lazio', 'Basilicata', 'Campania', 'Puglia', 'Calabria',
           'Sicilia', 'Sardegna'], dtype=object)




```
regions[regions.DEN_REG=='Liguria'].plot()
```




    <matplotlib.axes._subplots.AxesSubplot at 0x7fbc25948400>




![png](01_solution_exercise%2C_geopackage_and_representative_point_files/01_solution_exercise%2C_geopackage_and_representative_point_80_1.png)



```
regions[regions.DEN_REG=='Liguria'].to_crs(epsg=4326).to_file("liguria.geojson",driver='GeoJSON')
```


```
regions[regions.DEN_REG=='Liguria'].centroid.to_crs(epsg=4326).to_file("liguria_centroid.geojson",driver='GeoJSON')
```


```
regions[regions.DEN_REG=='Liguria'].representative_point().to_crs(epsg=4326).to_file("liguria_representative_point.geojson",driver='GeoJSON')
```


```
files.download("liguria.geojson")
```


```
files.download("liguria_centroid.geojson")
```


    <IPython.core.display.Javascript object>



    <IPython.core.display.Javascript object>



```
files.download('liguria_representative_point.geojson')
```


    <IPython.core.display.Javascript object>



    <IPython.core.display.Javascript object>


you can upload all the geojson on [uMap](http://umap.openstreetmap.fr) to [see the result](http://umap.openstreetmap.fr/it/map/liguria_505528#8/44.058/9.075) 

![](https://raw.githubusercontent.com/napo/geospatial_course_unitn/master/images/umap_liguria.jpg)
