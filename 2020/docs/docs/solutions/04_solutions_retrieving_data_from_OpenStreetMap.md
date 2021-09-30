---
layout: default
title: 04 - Retrieving data from OpenStreetMap - solution to the exercises
parent: Solutions
nav_order: 4
permalink: /lessons/solution_exercise_4
---
*Solution of exercise after the lesson of 16 October 2020*
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---
# Setup


```bash
!pip install pygeos
```

    Collecting pygeos
    [?25l  Downloading https://files.pythonhosted.org/packages/66/06/2cfcf6e90814da1fdb4585534f03a36531f00cbad65f11b417337d69fe60/pygeos-0.8-cp36-cp36m-manylinux1_x86_64.whl (1.6MB)
    [K     |████████████████████████████████| 1.6MB 4.8MB/s 
    [?25hRequirement already satisfied: numpy>=1.10 in /usr/local/lib/python3.6/dist-packages (from pygeos) (1.18.5)
    Installing collected packages: pygeos
    Successfully installed pygeos-0.8



```bash
!pip install geopandas
```

    Collecting geopandas
    [?25l  Downloading https://files.pythonhosted.org/packages/f7/a4/e66aafbefcbb717813bf3a355c8c4fc3ed04ea1dd7feb2920f2f4f868921/geopandas-0.8.1-py2.py3-none-any.whl (962kB)
    [K     |████████████████████████████████| 972kB 8.6MB/s 
    [?25hRequirement already satisfied: pandas>=0.23.0 in /usr/local/lib/python3.6/dist-packages (from geopandas) (1.1.2)
    Requirement already satisfied: shapely in /usr/local/lib/python3.6/dist-packages (from geopandas) (1.7.1)
    Collecting fiona
    [?25l  Downloading https://files.pythonhosted.org/packages/36/8b/e8b2c11bed5373c8e98edb85ce891b09aa1f4210fd451d0fb3696b7695a2/Fiona-1.8.17-cp36-cp36m-manylinux1_x86_64.whl (14.8MB)
    [K     |████████████████████████████████| 14.8MB 252kB/s 
    [?25hCollecting pyproj>=2.2.0
    [?25l  Downloading https://files.pythonhosted.org/packages/e5/c3/071e080230ac4b6c64f1a2e2f9161c9737a2bc7b683d2c90b024825000c0/pyproj-2.6.1.post1-cp36-cp36m-manylinux2010_x86_64.whl (10.9MB)
    [K     |████████████████████████████████| 10.9MB 51.2MB/s 
    [?25hRequirement already satisfied: numpy>=1.15.4 in /usr/local/lib/python3.6/dist-packages (from pandas>=0.23.0->geopandas) (1.18.5)
    Requirement already satisfied: pytz>=2017.2 in /usr/local/lib/python3.6/dist-packages (from pandas>=0.23.0->geopandas) (2018.9)
    Requirement already satisfied: python-dateutil>=2.7.3 in /usr/local/lib/python3.6/dist-packages (from pandas>=0.23.0->geopandas) (2.8.1)
    Collecting cligj>=0.5
      Downloading https://files.pythonhosted.org/packages/ba/06/e3440b1f2dc802d35f329f299ba96153e9fcbfdef75e17f4b61f79430c6a/cligj-0.7.0-py3-none-any.whl
    Collecting munch
      Downloading https://files.pythonhosted.org/packages/cc/ab/85d8da5c9a45e072301beb37ad7f833cd344e04c817d97e0cc75681d248f/munch-2.5.0-py2.py3-none-any.whl
    Requirement already satisfied: attrs>=17 in /usr/local/lib/python3.6/dist-packages (from fiona->geopandas) (20.2.0)
    Requirement already satisfied: six>=1.7 in /usr/local/lib/python3.6/dist-packages (from fiona->geopandas) (1.15.0)
    Requirement already satisfied: click<8,>=4.0 in /usr/local/lib/python3.6/dist-packages (from fiona->geopandas) (7.1.2)
    Collecting click-plugins>=1.0
      Downloading https://files.pythonhosted.org/packages/e9/da/824b92d9942f4e472702488857914bdd50f73021efea15b4cad9aca8ecef/click_plugins-1.1.1-py2.py3-none-any.whl
    Installing collected packages: cligj, munch, click-plugins, fiona, pyproj, geopandas
    Successfully installed click-plugins-1.1.1 cligj-0.7.0 fiona-1.8.17 geopandas-0.8.1 munch-2.5.0 pyproj-2.6.1.post1



```bash
!pip install pyrosm
```

    Collecting pyrosm
    [?25l  Downloading https://files.pythonhosted.org/packages/7c/ba/10de4eac775235554f52b9d02f21bcddd32deaeec027b1029553dc6befdc/pyrosm-0.5.3.tar.gz (2.0MB)
    [K     |████████████████████████████████| 2.0MB 8.8MB/s 
    [?25h  Installing build dependencies ... [?25l[?25hdone
      Getting requirements to build wheel ... [?25l[?25hdone
      Installing backend dependencies ... [?25l[?25hdone
        Preparing wheel metadata ... [?25l[?25hdone
    Requirement already satisfied: pygeos in /usr/local/lib/python3.6/dist-packages (from pyrosm) (0.8)
    Collecting python-rapidjson
      Using cached https://files.pythonhosted.org/packages/9e/cb/085b893850110d4e20ef3624808ccaec0515c07da0400e58bdd3ca73c5e3/python_rapidjson-0.9.1-cp36-cp36m-manylinux2010_x86_64.whl
    Processing /root/.cache/pip/wheels/a2/2b/e3/1800624884ba9461b903a05a809ff1c10317603a6ee8e74155/pyrobuf-0.9.3-cp36-cp36m-linux_x86_64.whl
    Requirement already satisfied: geopandas in /usr/local/lib/python3.6/dist-packages (from pyrosm) (0.8.1)
    Requirement already satisfied: setuptools>=18.0 in /usr/local/lib/python3.6/dist-packages (from pyrosm) (50.3.0)
    Processing /root/.cache/pip/wheels/12/d5/09/836011d00b6e694dfade8025669266260834574f47cfe18f62/cykhash-1.0.2-cp36-cp36m-linux_x86_64.whl
    Requirement already satisfied: numpy>=1.10 in /usr/local/lib/python3.6/dist-packages (from pygeos->pyrosm) (1.18.5)
    Requirement already satisfied: jinja2>=2.8 in /usr/local/lib/python3.6/dist-packages (from pyrobuf->pyrosm) (2.11.2)
    Requirement already satisfied: cython>=0.23 in /usr/local/lib/python3.6/dist-packages (from pyrobuf->pyrosm) (0.29.21)
    Requirement already satisfied: pandas>=0.23.0 in /usr/local/lib/python3.6/dist-packages (from geopandas->pyrosm) (1.1.2)
    Requirement already satisfied: shapely in /usr/local/lib/python3.6/dist-packages (from geopandas->pyrosm) (1.7.1)
    Requirement already satisfied: pyproj>=2.2.0 in /usr/local/lib/python3.6/dist-packages (from geopandas->pyrosm) (2.6.1.post1)
    Requirement already satisfied: fiona in /usr/local/lib/python3.6/dist-packages (from geopandas->pyrosm) (1.8.17)
    Requirement already satisfied: MarkupSafe>=0.23 in /usr/local/lib/python3.6/dist-packages (from jinja2>=2.8->pyrobuf->pyrosm) (1.1.1)
    Requirement already satisfied: python-dateutil>=2.7.3 in /usr/local/lib/python3.6/dist-packages (from pandas>=0.23.0->geopandas->pyrosm) (2.8.1)
    Requirement already satisfied: pytz>=2017.2 in /usr/local/lib/python3.6/dist-packages (from pandas>=0.23.0->geopandas->pyrosm) (2018.9)
    Requirement already satisfied: attrs>=17 in /usr/local/lib/python3.6/dist-packages (from fiona->geopandas->pyrosm) (20.2.0)
    Requirement already satisfied: six>=1.7 in /usr/local/lib/python3.6/dist-packages (from fiona->geopandas->pyrosm) (1.15.0)
    Requirement already satisfied: click<8,>=4.0 in /usr/local/lib/python3.6/dist-packages (from fiona->geopandas->pyrosm) (7.1.2)
    Requirement already satisfied: munch in /usr/local/lib/python3.6/dist-packages (from fiona->geopandas->pyrosm) (2.5.0)
    Requirement already satisfied: click-plugins>=1.0 in /usr/local/lib/python3.6/dist-packages (from fiona->geopandas->pyrosm) (1.1.1)
    Requirement already satisfied: cligj>=0.5 in /usr/local/lib/python3.6/dist-packages (from fiona->geopandas->pyrosm) (0.7.0)
    Building wheels for collected packages: pyrosm
      Building wheel for pyrosm (PEP 517) ... [?25l[?25hdone
      Created wheel for pyrosm: filename=pyrosm-0.5.3-cp36-cp36m-linux_x86_64.whl size=5013176 sha256=944815408a9b0c9de6fe8be97f9f1aaca22214e6b3a102e8e1ea657e221f9a36
      Stored in directory: /root/.cache/pip/wheels/a8/ca/18/ff3b302d589113fac92b89d73c135eeb5d0c62c06ae32441e8
    Successfully built pyrosm
    Installing collected packages: python-rapidjson, pyrobuf, cykhash, pyrosm
    Successfully installed cykhash-1.0.2 pyrobuf-0.9.3 pyrosm-0.5.3 python-rapidjson-0.9.1


# Exercise
- download from OpenStreetMap all supermarkets inside the bounding box of the city in this point<br/>
   latitude: 46.21209<br/>
   longitude: 11.09351
- identify the longest road of the city (state roads, walking routes, motorways are excluded). Please use "unclassified"
- How many drinking water are in this city?
- How many benches in this city have the backrest?


## download from OpenStreetMap all supermarkets inside the bounding box of the city with these coordinates



```python
import geopandas as gpd
```

    /usr/local/lib/python3.6/dist-packages/geopandas/_compat.py:88: UserWarning: The Shapely GEOS version (3.8.0-CAPI-1.13.1 ) is incompatible with the GEOS version PyGEOS was compiled with (3.8.1-CAPI-1.13.3). Conversions between both will be slow.
      shapely_geos_version, geos_capi_version_string


## find the city

### method 1 - reverse geocoding


```python
from geopy.geocoders import ArcGIS
```


```python
latlon = "46.21209" + "," + "11.09351"
```


```python
geolocator = ArcGIS(user_agent="Mozilla/5.0 (Linux; Android 10) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.75 Mobile Safari/537.36")
```


```python
location = geolocator.reverse(latlon)
```


```python
location.raw
```




    {'AddNum': '12A',
     'Addr_type': 'PointAddress',
     'Address': 'Via Francesco Filos 12A',
     'Block': '',
     'City': 'Mezzolombardo',
     'CountryCode': 'ITA',
     'District': 'Mezzolombardo',
     'LongLabel': 'Via Francesco Filos 12A, 38017, Mezzolombardo, Trento, ITA',
     'Match_addr': 'Via Francesco Filos 12A, 38017, Mezzolombardo, Trento',
     'MetroArea': '',
     'Neighborhood': '',
     'PlaceName': '',
     'Postal': '38017',
     'PostalExt': '',
     'Region': 'Trentino-Alto Adige',
     'Sector': '',
     'ShortLabel': 'Via Francesco Filos 12A',
     'Subregion': 'Trento',
     'Territory': '',
     'Type': ''}




```python
city = location.raw['City']
```


```python
city
```




    'Mezzolombardo'



### method 2 - spatial relation


```python
from shapely.geometry import Point
```


```python
point = Point(11.09351,46.21209)
```


```bash
!wget https://github.com/napo/geospatial_course_unitn/raw/master/data/administrative_units_italy_2020/istat_administrative_units_2020.gpkg
```

    --2020-10-23 12:58:59--  https://github.com/napo/geospatial_course_unitn/raw/master/data/administrative_units_italy_2020/istat_administrative_units_2020.gpkg
    Resolving github.com (github.com)... 140.82.118.3
    Connecting to github.com (github.com)|140.82.118.3|:443... connected.
    HTTP request sent, awaiting response... 302 Found
    Location: https://raw.githubusercontent.com/napo/geospatial_course_unitn/master/data/administrative_units_italy_2020/istat_administrative_units_2020.gpkg [following]
    --2020-10-23 12:58:59--  https://raw.githubusercontent.com/napo/geospatial_course_unitn/master/data/administrative_units_italy_2020/istat_administrative_units_2020.gpkg
    Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 151.101.0.133, 151.101.64.133, 151.101.128.133, ...
    Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|151.101.0.133|:443... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 23396352 (22M) [application/octet-stream]
    Saving to: ‘istat_administrative_units_2020.gpkg’
    
    istat_administrativ 100%[===================>]  22.31M  96.4MB/s    in 0.2s    
    
    2020-10-23 12:59:01 (96.4 MB/s) - ‘istat_administrative_units_2020.gpkg’ saved [23396352/23396352]
    



```python
municipalities = gpd.read_file("istat_administrative_units_2020.gpkg",layer="municipalities")
```


```python
muncipality = municipalities[municipalities.to_crs(epsg=4326).geometry.contains(point)]
```


```python
muncipality
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
      <th>PRO_COM</th>
      <th>PRO_COM_T</th>
      <th>COMUNE</th>
      <th>COMUNE_A</th>
      <th>CC_UTS</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>3417</th>
      <td>2</td>
      <td>4</td>
      <td>22</td>
      <td>0</td>
      <td>22</td>
      <td>22117</td>
      <td>022117</td>
      <td>Mezzolombardo</td>
      <td>None</td>
      <td>0</td>
      <td>MULTIPOLYGON (((659991.073 5121616.939, 660921...</td>
    </tr>
  </tbody>
</table>
</div>




```python
city = muncipality.COMUNE.values[0]
```


```python
city
```




    'Mezzolombardo'



## download from OpenStreetMap

## find the boundig box of Mezzolombardo


```python
from shapely.geometry import Polygon
```


```python
municipality = municipalities[municipalities.COMUNE==city]
```


```python
municipality.to_crs(epsg=4326).geometry.bounds
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
      <th>minx</th>
      <th>miny</th>
      <th>maxx</th>
      <th>maxy</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>3417</th>
      <td>11.062845</td>
      <td>46.177712</td>
      <td>11.118754</td>
      <td>46.231135</td>
    </tr>
  </tbody>
</table>
</div>




```python
bbox = list(municipality.to_crs(epsg=4326).geometry.bounds.values[0])
```

![](https://raw.githubusercontent.com/napo/geospatial_course_unitn/master/images/boudingbox.png)


```python
minx = bbox[0]
miny = bbox[1]
maxx = bbox[2]
maxy = bbox[3]
```


```python
polygon = Polygon([[minx, miny], [maxx, miny], [maxx, maxy], [minx, maxy]])
```


```python
polygon
```




    
![svg](04_solutions_retrieving_data_from_OpenStreetMap_files/04_solutions_retrieving_data_from_OpenStreetMap_34_0.svg)
    



### method 1 - download from export.hotosm.org

![](https://raw.githubusercontent.com/napo/geospatial_course_unitn/master/images/export_hostosm_mezzolombardo.png)


```python
data = {'description': ['bbox around Mezzolombardo'], 'geometry': [polygon]}
```


```python
bbox_mezzolombardo = gpd.GeoDataFrame(data,crs="epsg:4326")
```


```python
bbox_mezzolombardo
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
      <th>description</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>bbox around Mezzolombardo</td>
      <td>POLYGON ((11.06285 46.17771, 11.11875 46.17771...</td>
    </tr>
  </tbody>
</table>
</div>




```python
bbox_mezzolombardo.geometry[0]
```




    
![svg](04_solutions_retrieving_data_from_OpenStreetMap_files/04_solutions_retrieving_data_from_OpenStreetMap_39_0.svg)
    




```python
bbox_mezzolombardo.to_file("bbox_mezzolombardo.geojson",driver="GeoJSON")
```


```python
#if you use colab and you want download the file geojson, uncomment these rows
#from google.colab import files
#files.download("bbox_mezzolombardo.geojson")
```


```python
!wget https://github.com/napo/geospatial_course_unitn/raw/master/data/openstreetmap/mezzolombardo_bbox.osm.pbf
```

    --2020-10-23 13:10:16--  https://github.com/napo/geospatial_course_unitn/raw/master/data/openstreetmap/mezzolombardo_bbox.osm.pbf
    Resolving github.com (github.com)... 140.82.118.4
    Connecting to github.com (github.com)|140.82.118.4|:443... connected.
    HTTP request sent, awaiting response... 302 Found
    Location: https://raw.githubusercontent.com/napo/geospatial_course_unitn/master/data/openstreetmap/mezzolombardo_bbox.osm.pbf [following]
    --2020-10-23 13:10:16--  https://raw.githubusercontent.com/napo/geospatial_course_unitn/master/data/openstreetmap/mezzolombardo_bbox.osm.pbf
    Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 151.101.0.133, 151.101.64.133, 151.101.128.133, ...
    Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|151.101.0.133|:443... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 978912 (956K) [application/octet-stream]
    Saving to: ‘mezzolombardo_bbox.osm.pbf’
    
    mezzolombardo_bbox. 100%[===================>] 955.97K  --.-KB/s    in 0.04s   
    
    2020-10-23 13:10:17 (24.3 MB/s) - ‘mezzolombardo_bbox.osm.pbf’ saved [978912/978912]
    



```python
path_pbf_file = "/content/mezzolombardo_bbox.osm.pbf"
```

### method 2 - download and cut from geodati fmach

http://www.geodati.fmach.it/italia_osm.html

http://geodati.fmach.it/gfoss_geodata/osm/output_osm_regioni/trentino-alto-adige.pbf



```bash
!wget http://geodati.fmach.it/gfoss_geodata/osm/output_osm_regioni/trentino-alto-adige.pbf
```

    --2020-10-23 13:03:10--  http://geodati.fmach.it/gfoss_geodata/osm/output_osm_regioni/trentino-alto-adige.pbf
    Resolving geodati.fmach.it (geodati.fmach.it)... 77.72.197.167
    Connecting to geodati.fmach.it (geodati.fmach.it)|77.72.197.167|:80... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 90917416 (87M)
    Saving to: ‘trentino-alto-adige.pbf’
    
    trentino-alto-adige 100%[===================>]  86.71M  13.4MB/s    in 6.5s    
    
    2020-10-23 13:03:16 (13.3 MB/s) - ‘trentino-alto-adige.pbf’ saved [90917416/90917416]
    



```python
path_pbf_file_big = "/content/trentino-alto-adige.pbf"
```

## find all the supermarkets in the area




```python
import pyrosm
```

    /usr/local/lib/python3.6/dist-packages/pyrosm/utils/_compat.py:12: UserWarning: The Shapely GEOS version (3.8.0-CAPI-1.13.1 ) is incompatible with the GEOS version PyGEOS was compiled with (3.8.1-CAPI-1.13.3). The tool will work but it runs a bit slower.
      shapely_geos_version, geos_capi_version_string


![](https://raw.githubusercontent.com/napo/geospatial_course_unitn/master/images/tag_supermarket.png)


```python
osm = pyrosm.OSM(path_pbf_file_big,bounding_box=bbox)
```


```python
custom_filter = {'shop': ['supermarket']}
```


```python
supermarkets = osm.get_pois(custom_filter=custom_filter)
```


```python
supermarkets.shape
```




    (8, 19)




```python
supermarkets
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
      <th>tags</th>
      <th>lat</th>
      <th>id</th>
      <th>lon</th>
      <th>version</th>
      <th>changeset</th>
      <th>timestamp</th>
      <th>addr:city</th>
      <th>addr:country</th>
      <th>addr:housenumber</th>
      <th>addr:housename</th>
      <th>addr:postcode</th>
      <th>addr:street</th>
      <th>name</th>
      <th>opening_hours</th>
      <th>operator</th>
      <th>shop</th>
      <th>geometry</th>
      <th>osm_type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>None</td>
      <td>46.214546</td>
      <td>1596908256</td>
      <td>11.118455</td>
      <td>-99</td>
      <td>-99.0</td>
      <td>-99</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>Iper Soap</td>
      <td>None</td>
      <td>None</td>
      <td>supermarket</td>
      <td>POINT (11.11845 46.21455)</td>
      <td>node</td>
    </tr>
    <tr>
      <th>1</th>
      <td>{"level":"0"}</td>
      <td>46.208977</td>
      <td>1669326175</td>
      <td>11.102525</td>
      <td>-128</td>
      <td>-128.0</td>
      <td>-128</td>
      <td>Mezzolombardo</td>
      <td>IT</td>
      <td>37</td>
      <td>Centro Commerciale Roltalcenter</td>
      <td>38017</td>
      <td>Via Guido Fiorini</td>
      <td>Schlecker</td>
      <td>None</td>
      <td>None</td>
      <td>supermarket</td>
      <td>POINT (11.10252 46.20898)</td>
      <td>node</td>
    </tr>
    <tr>
      <th>2</th>
      <td>None</td>
      <td>46.213261</td>
      <td>3054012770</td>
      <td>11.089408</td>
      <td>0</td>
      <td>67.0</td>
      <td>67</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>Despar</td>
      <td>None</td>
      <td>None</td>
      <td>supermarket</td>
      <td>POINT (11.08941 46.21326)</td>
      <td>node</td>
    </tr>
    <tr>
      <th>3</th>
      <td>{"brand":"EuroSpin","brand:wikidata":"Q1374674...</td>
      <td>46.207256</td>
      <td>4557760211</td>
      <td>11.100459</td>
      <td>0</td>
      <td>0.0</td>
      <td>0</td>
      <td>Mezzolombardo</td>
      <td>IT</td>
      <td>65/A</td>
      <td>Complesso Commerciale Braide</td>
      <td>38017</td>
      <td>Via Trento</td>
      <td>EuroSpin</td>
      <td>Mo-Su 08:00-20:00</td>
      <td>None</td>
      <td>supermarket</td>
      <td>POINT (11.10046 46.20726)</td>
      <td>node</td>
    </tr>
    <tr>
      <th>4</th>
      <td>{"brand":"Conad","brand:wikidata":"Q639075","b...</td>
      <td>46.180454</td>
      <td>6516674409</td>
      <td>11.070800</td>
      <td>0</td>
      <td>0.0</td>
      <td>0</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>Conad</td>
      <td>Mo-Su 07:00-12:30; Mo-Sa 16:00-19:30</td>
      <td>Pasquale Aceto</td>
      <td>supermarket</td>
      <td>POINT (11.07080 46.18045)</td>
      <td>node</td>
    </tr>
    <tr>
      <th>5</th>
      <td>{"building":"yes","ref:vatin":"IT00108640228"}</td>
      <td>NaN</td>
      <td>39484297</td>
      <td>NaN</td>
      <td>8</td>
      <td>NaN</td>
      <td>1523179735</td>
      <td>Mezzolombardo</td>
      <td>IT</td>
      <td>33</td>
      <td>NaN</td>
      <td>38017</td>
      <td>Via Guido Fiorini</td>
      <td>Orvea</td>
      <td>None</td>
      <td>OR.VE.A. S.P.A.</td>
      <td>supermarket</td>
      <td>POLYGON ((11.10120 46.20900, 11.10125 46.20917...</td>
      <td>way</td>
    </tr>
    <tr>
      <th>6</th>
      <td>{"building":"yes"}</td>
      <td>NaN</td>
      <td>73412650</td>
      <td>NaN</td>
      <td>6</td>
      <td>NaN</td>
      <td>1517591180</td>
      <td>Mezzolombardo</td>
      <td>IT</td>
      <td>19</td>
      <td>NaN</td>
      <td>38017</td>
      <td>Via Rotaliana</td>
      <td>Amort</td>
      <td>None</td>
      <td>None</td>
      <td>supermarket</td>
      <td>POLYGON ((11.09483 46.21696, 11.09509 46.21711...</td>
      <td>way</td>
    </tr>
    <tr>
      <th>7</th>
      <td>{"brand":"Lidl","brand:wikidata":"Q151954","br...</td>
      <td>NaN</td>
      <td>567538828</td>
      <td>NaN</td>
      <td>5</td>
      <td>NaN</td>
      <td>1594466282</td>
      <td>Mezzolombardo</td>
      <td>IT</td>
      <td>79</td>
      <td>NaN</td>
      <td>38017</td>
      <td>Via Alcide De Gasperi</td>
      <td>LIDL</td>
      <td>Mo-Sa 08:00-21:00; Su 08:00-20:00</td>
      <td>LIDL ITALIA SRL</td>
      <td>supermarket</td>
      <td>POLYGON ((11.09423 46.21056, 11.09453 46.21074...</td>
      <td>way</td>
    </tr>
  </tbody>
</table>
</div>




```python
print("there are %s supermarkets in Mezzolombardo" % supermarkets.shape[0])
```

    there are 8 supermarkets in Mezzolombardo



```python
supermarkets.name
```




    0    Iper Soap
    1    Schlecker
    2       Despar
    3     EuroSpin
    4        Conad
    5        Orvea
    6        Amort
    7         LIDL
    Name: name, dtype: object



# identify the longest road of the city



```python
roads = osm.get_network(network_type='all')
```


```python
roads.columns
```




    Index(['access', 'area', 'bicycle', 'bridge', 'cycleway', 'foot', 'footway',
           'highway', 'int_ref', 'junction', 'lanes', 'lit', 'maxspeed',
           'motor_vehicle', 'name', 'oneway', 'ref', 'service', 'segregated',
           'smoothness', 'surface', 'tracktype', 'tunnel', 'width', 'id',
           'timestamp', 'version', 'tags', 'geometry', 'osm_type'],
          dtype='object')




```python
roads.highway.unique()
```




    array(['motorway', 'tertiary', 'residential', 'unclassified', 'track',
           'primary', 'secondary', 'cycleway', 'tertiary_link', 'footway',
           'path', 'service', 'steps', 'primary_link', 'pedestrian',
           'secondary_link', 'via_ferrata'], dtype=object)



![](https://github.com/napo/geospatial_course_unitn/raw/master/images/tag_highways.gif)

## lenght of unclassified and residential togheter


```python
roads_unclassified_residential = roads[(roads.highway=='unclassified') | (roads.highway == 'residential')]
```


```python
roads_unclassified_residential
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
      <th>access</th>
      <th>area</th>
      <th>bicycle</th>
      <th>bridge</th>
      <th>cycleway</th>
      <th>foot</th>
      <th>footway</th>
      <th>highway</th>
      <th>int_ref</th>
      <th>junction</th>
      <th>lanes</th>
      <th>lit</th>
      <th>maxspeed</th>
      <th>motor_vehicle</th>
      <th>name</th>
      <th>oneway</th>
      <th>ref</th>
      <th>service</th>
      <th>segregated</th>
      <th>smoothness</th>
      <th>surface</th>
      <th>tracktype</th>
      <th>tunnel</th>
      <th>width</th>
      <th>id</th>
      <th>timestamp</th>
      <th>version</th>
      <th>tags</th>
      <th>geometry</th>
      <th>osm_type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2</th>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>residential</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>Via Fornai</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>24621610</td>
      <td>1369153355</td>
      <td>6</td>
      <td>None</td>
      <td>LINESTRING (11.11717 46.21324, 11.11686 46.213...</td>
      <td>way</td>
    </tr>
    <tr>
      <th>4</th>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>unclassified</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>Via Dante Alighieri</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>24621622</td>
      <td>1543160359</td>
      <td>12</td>
      <td>None</td>
      <td>LINESTRING (11.11852 46.21649, 11.11853 46.216...</td>
      <td>way</td>
    </tr>
    <tr>
      <th>5</th>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>residential</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>Piazza della Chiesa</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>24621623</td>
      <td>1503940023</td>
      <td>6</td>
      <td>None</td>
      <td>LINESTRING (11.11875 46.21664, 11.11858 46.21656)</td>
      <td>way</td>
    </tr>
    <tr>
      <th>12</th>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>unclassified</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>Via alla Grotta</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>25388664</td>
      <td>1543160359</td>
      <td>10</td>
      <td>None</td>
      <td>LINESTRING (11.11869 46.21865, 11.11852 46.218...</td>
      <td>way</td>
    </tr>
    <tr>
      <th>13</th>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>unclassified</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>Via Carlo Devigili</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>25413543</td>
      <td>1522001556</td>
      <td>26</td>
      <td>None</td>
      <td>LINESTRING (11.09512 46.20965, 11.09569 46.209...</td>
      <td>way</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1153</th>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>residential</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>Via della Pineta</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>660696721</td>
      <td>1546696265</td>
      <td>1</td>
      <td>None</td>
      <td>LINESTRING (11.06637 46.17816, 11.06651 46.178...</td>
      <td>way</td>
    </tr>
    <tr>
      <th>1194</th>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>yes</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>residential</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>Via Damiano Chiesa</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>795943144</td>
      <td>1587816864</td>
      <td>1</td>
      <td>{"layer":"1"}</td>
      <td>LINESTRING (11.09586 46.21477, 11.09580 46.21475)</td>
      <td>way</td>
    </tr>
    <tr>
      <th>1195</th>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>residential</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>Via Damiano Chiesa</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>795943145</td>
      <td>1587816864</td>
      <td>1</td>
      <td>None</td>
      <td>LINESTRING (11.09610 46.21491, 11.09598 46.214...</td>
      <td>way</td>
    </tr>
    <tr>
      <th>1204</th>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>yes</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>unclassified</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>asphalt</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>809648314</td>
      <td>1590775561</td>
      <td>1</td>
      <td>{"layer":"1"}</td>
      <td>LINESTRING (11.09451 46.22785, 11.09454 46.22782)</td>
      <td>way</td>
    </tr>
    <tr>
      <th>1205</th>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>unclassified</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>asphalt</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>809648315</td>
      <td>1590775561</td>
      <td>1</td>
      <td>None</td>
      <td>LINESTRING (11.09352 46.22640, 11.09344 46.226...</td>
      <td>way</td>
    </tr>
  </tbody>
</table>
<p>238 rows × 30 columns</p>
</div>




```python
roads_unclassified_residential['name'].value_counts()
```




    Via Damiano Chiesa        8
    Via dei Morei             7
    Via Carlo Devigili        5
    Piazza della Chiesa       5
    Via alla Grotta           5
                             ..
    Via Don Sturzo            1
    Via degli Alpini          1
    Corso Giuseppe Mazzini    1
    Vicolo Scaletta           1
    Via Guido Fiorini         1
    Name: name, Length: 73, dtype: int64




```python
names = roads_unclassified_residential.name.unique()
```


```python
names
```




    array(['Via Fornai', 'Via Dante Alighieri', 'Piazza della Chiesa',
           'Via alla Grotta', 'Via Carlo Devigili', 'Via dei Morei',
           'Via Emanuele De Varda', 'Via Giorgio Perlasca', 'Via Taiti',
           'Via Frecce Tricolori', None, 'Via Trento', 'Via San Francesco',
           "Via Sant'Antonio", 'Via Milano', 'Via fratelli de Panizza',
           'Via F. de Panizza', 'Via Francesco de Luca', 'Via Cristiani',
           'Via Conte Carlo Martini', 'Via dei Sentieri', 'Località Rauti',
           'Piazza Pio XII', 'Via Alessandro Manzoni', 'Via Giosuè Carducci',
           'Via Damiano Chiesa', 'Via San Vigilio', 'Via della Rupe',
           'Via Guido Fiorini', 'Piazza Cassa di Risparmio', 'Via Dante',
           'Vicolo Travaion', 'Via dei Molini', 'Via Rotaliana',
           'Via Francesco Morigl', 'Via Riccardo Zandonai',
           'Piazza Cesare Battisti', 'Via Cavalleggeri Udine',
           'Via F. De Panizza', 'Via Giuseppe Garibaldi', 'Via Fabio Filzi',
           'Corso Giuseppe Mazzini', 'Via Don Sturzo', 'Piazza San Giovanni',
           'Via Canevarie', 'Via Romedio De Scari', 'Via Bertagnolli',
           'Via San Pietro', 'Piazza Luigi Dalpiaz', "Via Sant'Anna",
           'Corso del Popolo', 'Via Francesco Filos', 'Via Camorzi',
           'Via Desiderio Reich', 'Via degli Alpini', 'Viale Fenice',
           'Piazza delle Erbe', 'Via della Pineta', "Via dell'artigianato",
           'Via alle Salézze', 'Strada delle Palù', 'Via al Dòs Alt',
           'Via Luigi Endrizzi', "Via di Pradonec'", 'Via Arturo De Varda',
           'Via Roma', 'Via al Belvedere', 'Via Molini', 'Località Galletta',
           'Vicolo Pozzo', 'Via Cortalta', 'Via Trieste', 'Vicolo Scaletta',
           'Via ai Dossi'], dtype=object)




```python
roads_lenght = {}
rodas_in_meters = roads_unclassified_residential.to_crs(epsg=32632)
for name in names:
  road = rodas_in_meters[rodas_in_meters.name==name]
  road_lenght = road.length.sum()
  roads_lenght[name] = road_lenght

```


```python
roads_lenght
```




    {'Corso Giuseppe Mazzini': 249.7579935275893,
     'Corso del Popolo': 235.021292582283,
     'Località Galletta': 321.34567496289685,
     'Località Rauti': 634.8241461600136,
     None: 0.0,
     'Piazza Cassa di Risparmio': 101.69215459438668,
     'Piazza Cesare Battisti': 103.70389536670257,
     'Piazza Luigi Dalpiaz': 46.95951436559089,
     'Piazza Pio XII': 244.72762778937602,
     'Piazza San Giovanni': 243.40400521661013,
     'Piazza della Chiesa': 123.76304437105432,
     'Piazza delle Erbe': 331.4274049761924,
     'Strada delle Palù': 144.0046947780986,
     'Via Alessandro Manzoni': 144.3589083543954,
     'Via Arturo De Varda': 274.9059635268403,
     'Via Bertagnolli': 339.42906670938265,
     'Via Camorzi': 125.38773228061129,
     'Via Canevarie': 264.79016895271434,
     'Via Carlo Devigili': 1410.4261234546225,
     'Via Cavalleggeri Udine': 444.7088078668066,
     'Via Conte Carlo Martini': 286.77891249144983,
     'Via Cortalta': 65.66625197808622,
     'Via Cristiani': 911.3802270676734,
     'Via Damiano Chiesa': 1306.158838946422,
     'Via Dante': 139.4356449287226,
     'Via Dante Alighieri': 172.1078302300374,
     'Via Desiderio Reich': 158.04743554343378,
     'Via Don Sturzo': 613.5101545407463,
     'Via Emanuele De Varda': 347.793341046712,
     'Via F. De Panizza': 344.9144110890303,
     'Via F. de Panizza': 130.29685773557736,
     'Via Fabio Filzi': 45.59962863184364,
     'Via Fornai': 620.1170060138757,
     'Via Francesco Filos': 214.15027878229265,
     'Via Francesco Morigl': 350.10828598223657,
     'Via Francesco de Luca': 24.658289226374322,
     'Via Frecce Tricolori': 240.73101365125157,
     'Via Giorgio Perlasca': 280.08567472028597,
     'Via Giosuè Carducci': 88.35894782185844,
     'Via Giuseppe Garibaldi': 364.64660987602775,
     'Via Guido Fiorini': 262.2489738895931,
     'Via Luigi Endrizzi': 86.33641082608705,
     'Via Milano': 428.4259989016257,
     'Via Molini': 278.74884599160333,
     'Via Riccardo Zandonai': 468.15764400181297,
     'Via Roma': 242.00848353588032,
     'Via Romedio De Scari': 80.7222654891916,
     'Via Rotaliana': 392.53309350933915,
     'Via San Francesco': 228.8846359443125,
     'Via San Pietro': 57.05340435247324,
     'Via San Vigilio': 310.2588856745822,
     "Via Sant'Anna": 128.52909522655204,
     "Via Sant'Antonio": 592.9004318778705,
     'Via Taiti': 120.27197628265175,
     'Via Trento': 1479.2922178941203,
     'Via Trieste': 65.14682389765427,
     'Via ai Dossi': 155.84109987644882,
     'Via al Belvedere': 153.67873563612065,
     'Via al Dòs Alt': 264.037016103423,
     'Via alla Grotta': 237.58834165388737,
     'Via alle Salézze': 96.88330190590919,
     'Via degli Alpini': 223.48843236826917,
     'Via dei Molini': 424.49770629326423,
     'Via dei Morei': 945.8319323865237,
     'Via dei Sentieri': 271.1617869705736,
     "Via dell'artigianato": 67.52635232503903,
     'Via della Pineta': 592.003644670534,
     'Via della Rupe': 657.2358490219722,
     "Via di Pradonec'": 216.50817938680152,
     'Via fratelli de Panizza': 984.0269482876433,
     'Viale Fenice': 125.4508309037719,
     'Vicolo Pozzo': 97.97035804419028,
     'Vicolo Scaletta': 41.34259167941211,
     'Vicolo Travaion': 89.37189286431745}




```python
max(roads_lenght, key=roads_lenght.get)
```




    'Via Trento'




```python
longest_road = max(roads_lenght, key=roads_lenght.get)
```


```python
print("the longest road (unclassified + residential) in Mezzolombardo is %s and is long %s meters" % (str(longest_road),roads_lenght[longest_road ]))
```

    the longest road (unclassified + residential) in Mezzolombardo is Via Trento and is long 1479.2922178941203 meters


## the case only with unclassified


```python
roads_unclassified = roads[(roads.highway=='unclassified')] 
```


```python
names = roads_unclassified.name.unique()
```


```python
roads_lenght = {}
rodas_in_meters = roads_unclassified.to_crs(epsg=32632)
for name in names:
  road = rodas_in_meters[rodas_in_meters.name==name]
  road_lenght = road.length.sum()
  roads_lenght[name] = road_lenght
```


```python
longest_road = max(roads_lenght, key=roads_lenght.get)
```


```python
print("the longest road (unclassified) is %s and is long %s meters" % (str(longest_road),roads_lenght[longest_road ]))
```

    the longest road (unclassified) is Via Trento and is long 1479.2922178941203 meters


## where is this road?
... we can use overpass-turbo.eu with this wizard query

```python
name='Via Trento' in Mezzolombardo
```

http://overpass-turbo.eu/s/Zkz

![](https://github.com/napo/geospatial_course_unitn/raw/master/images/roads_lenght_mezzolombardo.gif)

# How many drinking water are in this city?


```python
custom_filter = {'amenity': ['drinking_water']}
```


```python
drinking_water = osm.get_pois(custom_filter=custom_filter)
```


```python
drinking_water.amenity.unique()
```




    array(['drinking_water'], dtype=object)




```python
drinking_water.shape
```




    (29, 12)




```python
drinking_water.head(4)
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
      <th>tags</th>
      <th>lat</th>
      <th>id</th>
      <th>lon</th>
      <th>version</th>
      <th>changeset</th>
      <th>timestamp</th>
      <th>name</th>
      <th>amenity</th>
      <th>drinking_water</th>
      <th>geometry</th>
      <th>osm_type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>None</td>
      <td>46.214466</td>
      <td>388393249</td>
      <td>11.113433</td>
      <td>67</td>
      <td>67</td>
      <td>67</td>
      <td>None</td>
      <td>drinking_water</td>
      <td>None</td>
      <td>POINT (11.11343 46.21447)</td>
      <td>node</td>
    </tr>
    <tr>
      <th>1</th>
      <td>None</td>
      <td>46.217033</td>
      <td>388393322</td>
      <td>11.108165</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>None</td>
      <td>drinking_water</td>
      <td>None</td>
      <td>POINT (11.10816 46.21703)</td>
      <td>node</td>
    </tr>
    <tr>
      <th>2</th>
      <td>None</td>
      <td>46.216507</td>
      <td>388393381</td>
      <td>11.118232</td>
      <td>-112</td>
      <td>-128</td>
      <td>-128</td>
      <td>None</td>
      <td>drinking_water</td>
      <td>None</td>
      <td>POINT (11.11823 46.21651)</td>
      <td>node</td>
    </tr>
    <tr>
      <th>3</th>
      <td>None</td>
      <td>46.225494</td>
      <td>388416004</td>
      <td>11.092862</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>None</td>
      <td>drinking_water</td>
      <td>None</td>
      <td>POINT (11.09286 46.22549)</td>
      <td>node</td>
    </tr>
  </tbody>
</table>
</div>




```python
drinking_water.plot()
```




    
![png](04_solutions_retrieving_data_from_OpenStreetMap_files/04_solutions_retrieving_data_from_OpenStreetMap_86_1.png)
    


# How many benches in this city have the backrest?


```python
custom_filter = {'amenity': ['bench']}
```


```python
benchs = osm.get_pois(custom_filter=custom_filter)
```


```python
benchs.shape
```


    (132, 10)


```python
benchs.columns
```




    Index(['tags', 'lat', 'id', 'lon', 'version', 'changeset', 'timestamp',
           'amenity', 'geometry', 'osm_type'],
          dtype='object')




```python
benchs.tags.unique()
```




    array(['{"backrest":"yes"}', '{"backrest":"no"}',
           '{"backrest":"yes","material":"metal","seats":"2"}',
           '{"colour":"brown","backrest":"no","material":"wood"}',
           '{"seats":"5","backrest":"yes"}', '{"seats":"4","backrest":"yes"}',
           None, '{"backrest":"yes","material":"wood"}',
           '{"backrest":"yes","material":"metal","seats":"6"}'], dtype=object)



quick&dirty solution


```python
benchs_with_backrest = benchs[benchs.tags.str.find('"backrest":"yes"') > -1]
```


```python
benchs_with_backrest.tags.unique()
```




    array(['{"backrest":"yes"}',
           '{"backrest":"yes","material":"metal","seats":"2"}',
           '{"seats":"5","backrest":"yes"}', '{"seats":"4","backrest":"yes"}',
           '{"backrest":"yes","material":"wood"}',
           '{"backrest":"yes","material":"metal","seats":"6"}'], dtype=object)




```python
benchs_with_backrest.shape
```




    (103, 10)




```python
print("the total of benchs tagged with the presence of a backrest is %s" % (str(benchs_with_backrest.shape[0])))
```

    the total of benchs tagged with the presence of a backrest is 103

