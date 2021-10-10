---
title: "Lesson 02: Spatial relationships and operations"
permalink: /lessons/02-spatial_relationships_and_operations
excerpt: "Spatial is Special"
last_modified_at: 2021-10-07T17:48:05-03:00
#redirect_from:
#  - /theme-setup/
toc: true
---
# Spatial relationships and operations
**based on scipy2018-geospatial**

## goals of the tutorial
- load tabular data files (eg. csv or xls) as geodataframe
- spatial projection conversion
- spatial relationships 
- spatial joins
- spatial operations

**based on the open data of:**
- [ISTAT](https://www.istat.it/it/archivio/222527) Italian National Institute of Statistic 
- [Italian Ministry of Agricultural, Food and Forestry Policies](https://www.politicheagricole.it/)



### requirements
- python knowledge
- pandas
- previous lesson

### status 
*"Spatial is Special"*

---

# Setup
for the spatial operations we need to improve geopandas with some libraries.

You can use rtree o pygeos 

## rtree
if you want use [rtree](https://rtree.readthedocs.io/en/latest/) you need also to install a C library in your O.S.

Pyton RTree is a wrapper to the library [libspatialiteindex](https://libspatialindex.org/en/latest/)

if you are using a Linux distribution based on Debian (like the instance of Google Colab) you have to install the libspatialindex library before


```python
try:
  import rtree
except ModuleNotFoundError as e:
  !apt-get install libspatialindex-dev
  !pip install rtree==0.9.2
  import rtree
```

## pygeos

[PyGEOS](https://pygeos.readthedocs.io/en/stable/) is a Python library for working with GEOS geometries. It is a wrapper for the GEOS C API.


```python
try:
  import pygeos
except ModuleNotFoundError as e:
  !pip install pygeos==0.10.2
  import pygeos
```

## geopandas

version 0.10.0 


```python
try:
  import geopandas as gpd
except ModuleNotFoundError as e:
  !pip install geopandas==0.10.0
  import geopandas as gpd
  if gpd.__version__ != "0.10.0":
    !pip install -U geopandas==0.10.0
```


```python
import pandas as pd
import geopandas as gpd
import matplotlib.pyplot as plt
import requests 
```

# data setup
## administrative units of italy

geopackage with the administrative units of italy

The couse offers the file in a geopackage stored [here](https://github.com/napo/geospatial_course_unitn/raw/master/data/istat/istat_administrative_units_generalized_2021.gpkg)


```python
url = 'https://github.com/napo/geospatial_course_unitn/raw/master/data/istat/istat_administrative_units_generalized_2021.gpkg'
```


```python
macroregions = gpd.read_file(url,layer="macroregions")
```

you can repeat the operation ( = direct download) for each layer but it's better to download the file once and load the layers step by step.


```python
# download the file 
geopackage_istat_file = "istat_administrative_units_generalized_2021.gpkg"
r = requests.get(url, allow_redirects=True)
open(geopackage_istat_file, 'wb').write(r.content)
```


    42049536


```python
regions = gpd.read_file(geopackage_istat_file,layer="regions")
```

```python
provincies = gpd.read_file(geopackage_istat_file,layer="provincies")
```

```python
municipalities = gpd.read_file(geopackage_istat_file,layer="municipalities")
```

```python
macroregions
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
      <th>DEN_RIP</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Nord-Ovest</td>
      <td>MULTIPOLYGON (((568226.691 4874823.573, 568219...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>Nord-Est</td>
      <td>MULTIPOLYGON (((618343.929 4893985.661, 618335...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Centro</td>
      <td>MULTIPOLYGON (((875952.995 4524692.050, 875769...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>Sud</td>
      <td>MULTIPOLYGON (((1083358.846 4416348.741, 10833...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Isole</td>
      <td>MULTIPOLYGON (((822886.611 3935355.889, 822871...</td>
    </tr>
  </tbody>
</table>
</div>



## monumental trees of Italy
The italian Ministery of Agricultural offers a datasets the location of each monumental tree for each region of Italy.<br/>

Visit this [page](https://www.politicheagricole.it/flex/cm/pages/ServeBLOB.php/L/IT/IDPagina/11260)

### from a dataframe to a geodataframe
we have a several XLS files with a sheet where there is a column with latitude and another with longitude



```python
# here we created a list with all the resources for each italian region listed on the website
regional_sources=[]
#abruzzo
regional_sources.append("https://www.politicheagricole.it/flex/cm/pages/ServeAttachment.php/L/IT/D/0%252F8%252F1%252FD.f2005fd478b7477df1d4/P/BLOB%3AID%3D11260/E/xls")
#bolzano
regional_sources.append("https://www.politicheagricole.it/flex/cm/pages/ServeAttachment.php/L/IT/D/8%252F9%252F0%252FD.f29391d6deaa808addeb/P/BLOB%3AID%3D11260/E/xls")
# campania
regional_sources.append("https://www.politicheagricole.it/flex/cm/pages/ServeAttachment.php/L/IT/D/5%252Fd%252F5%252FD.76b727d50600037545c6/P/BLOB%3AID%3D11260/E/xls")
# fvg
regional_sources.append("https://www.politicheagricole.it/flex/cm/pages/ServeAttachment.php/L/IT/D/f%252Fd%252F3%252FD.946890f9eaea54ae5ea1/P/BLOB%3AID%3D11260/E/xls")
# liguria
regional_sources.append("https://www.politicheagricole.it/flex/cm/pages/ServeAttachment.php/L/IT/D/e%252F3%252F7%252FD.05dfc2cec6e9136d75a8/P/BLOB%3AID%3D11260/E/xls")
# marche
regional_sources.append("https://www.politicheagricole.it/flex/cm/pages/ServeAttachment.php/L/IT/D/b%252Fc%252F9%252FD.f141f45553f7a7a66b32/P/BLOB%3AID%3D11260/E/xls")
# piemonte
regional_sources.append("https://www.politicheagricole.it/flex/cm/pages/ServeAttachment.php/L/IT/D/a%252Fa%252Fa%252FD.0948793850e0ffb9b8f8/P/BLOB%3AID%3D11260/E/xls")
# sardegna
regional_sources.append("https://www.politicheagricole.it/flex/cm/pages/ServeAttachment.php/L/IT/D/b%252Fb%252F4%252FD.461f367f4970e94e4ace/P/BLOB%3AID%3D11260/E/xls")
# toscana
regional_sources.append("https://www.politicheagricole.it/flex/cm/pages/ServeAttachment.php/L/IT/D/b%252Fd%252Fb%252FD.769c1bc18ed0fd6c7525/P/BLOB%3AID%3D11260/E/xls")
# umbria
regional_sources.append("https://www.politicheagricole.it/flex/cm/pages/ServeAttachment.php/L/IT/D/1%252F4%252F6%252FD.9c2d18fc20e20bddf41c/P/BLOB%3AID%3D11260/E/xls")
# veneto
regional_sources.append("https://www.politicheagricole.it/flex/cm/pages/ServeAttachment.php/L/IT/D/2%252F1%252F5%252FD.c0a841fd03577da4f4df/P/BLOB%3AID%3D11260/E/xls")
# basilicata
regional_sources.append("https://www.politicheagricole.it/flex/cm/pages/ServeAttachment.php/L/IT/D/0%252F4%252Fb%252FD.593ccd772b2035897865/P/BLOB%3AID%3D11260/E/xls")
# calabria
regional_sources.append("https://www.politicheagricole.it/flex/cm/pages/ServeAttachment.php/L/IT/D/8%252F5%252F9%252FD.791fdf3bf4b0e58ba67e/P/BLOB%3AID%3D11260/E/xls")
# emilia romagna
regional_sources.append("https://www.politicheagricole.it/flex/cm/pages/ServeAttachment.php/L/IT/D/8%252F3%252Ff%252FD.f2adadc654c2f900a2e8/P/BLOB%3AID%3D11260/E/xls")
# lazio
regional_sources.append('https://www.politicheagricole.it/flex/cm/pages/ServeAttachment.php/L/IT/D/b%252F2%252F2%252FD.6934fcd6bbd065d1c9e3/P/BLOB%3AID%3D11260/E/xls')
# lombardia
regional_sources.append("https://www.politicheagricole.it/flex/cm/pages/ServeAttachment.php/L/IT/D/2%252F3%252F5%252FD.729e47446f1af140a478/P/BLOB%3AID%3D11260/E/xls")
# molise
regional_sources.append("https://www.politicheagricole.it/flex/cm/pages/ServeAttachment.php/L/IT/D/f%252Fd%252F3%252FD.c331aee0b88fde31643f/P/BLOB%3AID%3D11260/E/xls")
# puglia
regional_sources.append("https://www.politicheagricole.it/flex/cm/pages/ServeAttachment.php/L/IT/D/0%252Fc%252F7%252FD.f030e1f218aaa3986719/P/BLOB%3AID%3D11260/E/xls")
# sicilia
regional_sources.append("https://www.politicheagricole.it/flex/cm/pages/ServeAttachment.php/L/IT/D/b%252Fd%252F6%252FD.c0ba2f7d6431aeea8012/P/BLOB%3AID%3D11260/E/xls")
#trento
regional_sources.append("https://www.politicheagricole.it/flex/cm/pages/ServeAttachment.php/L/IT/D/8%252F5%252Fe%252FD.3b60c3340ad1de02b36e/P/BLOB%3AID%3D11260/E/xls")
# val d'aosta
regional_sources.append("https://www.politicheagricole.it/flex/cm/pages/ServeAttachment.php/L/IT/D/b%252F9%252F2%252FD.03014417b256a41ab612/P/BLOB%3AID%3D11260/E/xls")
```

you can donwload each dataset directly with pandas with an instruction like this:
```python
df = pd.read_excel(regional_sources[0])
```
... but you can obtain some mistakes due a SSL error
 
you can solve in this way with the following code following this [thread on stackoverflow](https://stackoverflow.com/questions/38015537/python-requests-exceptions-sslerror-dh-key-too-small)

```python
requests.packages.urllib3.util.ssl_.DEFAULT_CIPHERS += 'HIGH:!DH:!aNULL'
try:
    requests.packages.urllib3.contrib.pyopenssl.DEFAULT_SSL_CIPHER_LIST += 'HIGH:!DH:!aNULL'
except AttributeError:
    # no pyopenssl support used / needed / available
    pass

monumental_trees = None
for source in regional_sources:
    req= requests.get(source)
    if (monumental_trees is None):
        monumental_trees = pd.read_excel(req.content)
        firststep = False
    else:
        monumental_trees = monumental_trees.append(pd.read_excel(req.content))
```


### investigate the data


```python
monumental_trees.shape[0]
```

    3665


```python
monumental_trees
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
      <th>PROGR</th>
      <th>REGIONE</th>
      <th>ID SCHEDA</th>
      <th>PROVINCIA</th>
      <th>COMUNE</th>
      <th>LOCALITÀ</th>
      <th>LATITUDINE SU GIS</th>
      <th>LONGITUDINE SU GIS</th>
      <th>ALTITUDINE (m s.l.m.)</th>
      <th>CONTESTO URBANO</th>
      <th>SPECIE NOME SCIENTIFICO</th>
      <th>SPECIE NOME VOLGARE</th>
      <th>CIRCONFERENZA FUSTO (cm)</th>
      <th>ALTEZZA (m)</th>
      <th>CRITERI DI MONUMENTALITÀ</th>
      <th>PROPOSTA DICHIARAZIONE NOTEVOLE INTERESSE PUBBLICO</th>
      <th>Unnamed: 16</th>
      <th>RIFERIMENTO LEGISLATIVO</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>ABRUZZO</td>
      <td>01/A235/CH/13</td>
      <td>Chieti</td>
      <td>Altino</td>
      <td>Le Macchie Articciaro</td>
      <td>42° 05' 14,02''</td>
      <td>14° 20' 34,97''</td>
      <td>215.0</td>
      <td>no</td>
      <td>Juniperus oxycedrus L.</td>
      <td>Ginepro coccolone</td>
      <td>125</td>
      <td>7</td>
      <td>a) età e/o dimensioni\nb) forma e portamento\n...</td>
      <td>no</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>ABRUZZO</td>
      <td>01/A367/CH/13</td>
      <td>Chieti</td>
      <td>Archi</td>
      <td>Serra Castello</td>
      <td>42° 04' 43,15''</td>
      <td>14° 22' 57,14''</td>
      <td>525.0</td>
      <td>no</td>
      <td>Arbutus unedo L.</td>
      <td>Corbezzolo</td>
      <td>125</td>
      <td>5.5</td>
      <td>a) età e/o dimensioni\nd) rarità botanica</td>
      <td>no</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>ABRUZZO</td>
      <td>01/A485/CH/13</td>
      <td>Chieti</td>
      <td>Atessa</td>
      <td>Santa Lucia - Piana Sant'Antonio</td>
      <td>42° 06' 41,82''</td>
      <td>14° 25' 55,52''</td>
      <td>125.0</td>
      <td>sì</td>
      <td>Quercus pubescens Willd.</td>
      <td>Roverella</td>
      <td>355</td>
      <td>17</td>
      <td>a) età e/o dimensioni</td>
      <td>no</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>ABRUZZO</td>
      <td>01/A956/CH/13</td>
      <td>Chieti</td>
      <td>Bomba</td>
      <td>Cementificio - Casale Nasuti</td>
      <td>42° 03' 07,95''</td>
      <td>14° 21' 48,11''</td>
      <td>400.0</td>
      <td>no</td>
      <td>Quercus pubescens Willd.</td>
      <td>Roverella</td>
      <td>530</td>
      <td>22</td>
      <td>a) età e/o dimensioni\nc) valore ecologico</td>
      <td>no</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>ABRUZZO</td>
      <td>02/A956/CH/13</td>
      <td>Chieti</td>
      <td>Bomba</td>
      <td>Cementificio - Casale Nasuti</td>
      <td>42° 03' 07,11''</td>
      <td>14° 21' 48,97''</td>
      <td>400.0</td>
      <td>no</td>
      <td>Quercus pubescens Willd.</td>
      <td>Roverella</td>
      <td>475</td>
      <td>21</td>
      <td>a) età e/o dimensioni</td>
      <td>no</td>
      <td>NaN</td>
      <td>NaN</td>
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
    </tr>
    <tr>
      <th>105</th>
      <td>106</td>
      <td>VALLE D'AOSTA</td>
      <td>03/L647/AO/02</td>
      <td>Aosta</td>
      <td>Valsavarenche</td>
      <td>Bosco di Protezione di Bien  (Particella Econo...</td>
      <td>45° 34' 25,9''</td>
      <td>7° 12' 51,4''</td>
      <td>1721.0</td>
      <td>no</td>
      <td>Larix decidua Mill.</td>
      <td>Larice</td>
      <td>372</td>
      <td>31.0</td>
      <td>a) età e/o dimensioni\nb) forma e portamento</td>
      <td>no</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>106</th>
      <td>107</td>
      <td>VALLE D'AOSTA</td>
      <td>04/L647/AO/02</td>
      <td>Aosta</td>
      <td>Valsavarenche</td>
      <td>Bosco di Protezione di Bien  (Particella Econo...</td>
      <td>45° 34' 34,2''</td>
      <td>7° 12' 50,4''</td>
      <td>1712.0</td>
      <td>no</td>
      <td>Larix decidua Mill.</td>
      <td>Larice</td>
      <td>368</td>
      <td>25.0</td>
      <td>a) età e/o dimensioni\nb) forma e portamento</td>
      <td>no</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>107</th>
      <td>108</td>
      <td>VALLE D'AOSTA</td>
      <td>05/L647/AO/02</td>
      <td>Aosta</td>
      <td>Valsavarenche</td>
      <td>Bosco di Protezione di Bien  (Particella Econo...</td>
      <td>45° 34' 24,3''</td>
      <td>7° 12' 52,8''</td>
      <td>1718.0</td>
      <td>no</td>
      <td>Larix decidua Mill.</td>
      <td>Larice</td>
      <td>348</td>
      <td>28.0</td>
      <td>a) età e/o dimensioni\nb) forma e portamento</td>
      <td>no</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>108</th>
      <td>109</td>
      <td>VALLE D'AOSTA</td>
      <td>06/L647/AO/02</td>
      <td>Aosta</td>
      <td>Valsavarenche</td>
      <td>Bosco di Protezione di Bien  (Particella Econo...</td>
      <td>45° 34' 23,6''</td>
      <td>7° 12' 54,8''</td>
      <td>1771.0</td>
      <td>no</td>
      <td>Larix decidua Mill.</td>
      <td>Larice</td>
      <td>358</td>
      <td>26.0</td>
      <td>a) età e/o dimensioni\nb) forma e portamento</td>
      <td>no</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>109</th>
      <td>110</td>
      <td>VALLE D'AOSTA</td>
      <td>07/L647/AO/02</td>
      <td>Aosta</td>
      <td>Valsavarenche</td>
      <td>Bosco di Protezione di Bien  (Particella Econo...</td>
      <td>45° 34' 23,2''</td>
      <td>7° 12' 54,8''</td>
      <td>1765.0</td>
      <td>no</td>
      <td>Larix decidua Mill.</td>
      <td>Larice</td>
      <td>373</td>
      <td>23.5</td>
      <td>a) età e/o dimensioni\nb) forma e portamento</td>
      <td>no</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>3665 rows × 18 columns</p>
</div>




```python
try:
  import dms2dec 
except ModuleNotFoundError as e:
  !pip install dms2dec==0.1
  import dms2dec
```

latitude and longitude here are stored in degrees.<br/>
Eg.<br/>
45° 34' 23,2''	

you need to transform it in decimal degree: 

```javascript
degree + minutes / 60 + seconds/(60*60) 
```

and assume negative values if the data is in West or South. 

here an example code


```python
import re
lat = '''45°34'23.2"N'''
deg, minutes, seconds, direction =  re.split('[°\'"]', lat)
(float(deg) + float(minutes)/60 + float(seconds)/(60*60)) * (-1 if direction in ['W', 'S'] else 1)
```

    45.57311111111112


a good blog post to explain the problem is here 

![](https://i0.wp.com/cdnssl.ubergizmo.com/wp-content/uploads/2016/02/lines-of-latitude-and-longitude.png)

https://www.ubergizmo.com/how-to/read-gps-coordinates/


```python
# dms2dec is an useful library for this conversion
from dms2dec.dms_convert import dms2dec
```


```python
# create the columns with the information in decimal degree
monumental_trees['latitude'] = monumental_trees['LATITUDINE SU GIS'].apply(lambda x: dms2dec(x))
monumental_trees['longitude'] = monumental_trees['LONGITUDINE SU GIS'].apply(lambda x: dms2dec(x))
```


```python
monumental_trees.columns
```


    Index(['PROGR', 'REGIONE', 'ID SCHEDA', 'PROVINCIA', 'COMUNE', 'LOCALITÀ',
           'LATITUDINE SU GIS', 'LONGITUDINE SU GIS', 'ALTITUDINE (m s.l.m.)',
           'CONTESTO URBANO', 'SPECIE NOME SCIENTIFICO', 'SPECIE NOME VOLGARE',
           'CIRCONFERENZA FUSTO (cm)', 'ALTEZZA (m)', 'CRITERI DI MONUMENTALITÀ',
           'PROPOSTA DICHIARAZIONE NOTEVOLE INTERESSE PUBBLICO', 'Unnamed: 16',
           'RIFERIMENTO LEGISLATIVO', 'latitude', 'longitude'],
          dtype='object')




```python
# rename some columns
columns= {
   'REGIONE': 'region',
   'PROVINCIA': 'province',
   'COMUNE': 'municipality',
   'LOCALITÀ': 'place',
   'ALTITUDINE (m s.l.m.)':'altitude',
   'CONTESTO URBANO':'urban_place',
   'SPECIE NOME SCIENTIFICO':'species_scientific_name',
   'SPECIE NOME VOLGARE':'species_common_name'
  }

```


```python
monumental_trees.rename(columns=columns,inplace=True)
```


```python
# choose only the columns we need
monumental_trees = monumental_trees[['region','province','municipality','place','altitude','urban_place','species_scientific_name','species_common_name','latitude','longitude']]

```


```python
monumental_trees.head(5)
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
      <th>region</th>
      <th>province</th>
      <th>municipality</th>
      <th>place</th>
      <th>altitude</th>
      <th>urban_place</th>
      <th>species_scientific_name</th>
      <th>species_common_name</th>
      <th>latitude</th>
      <th>longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ABRUZZO</td>
      <td>Chieti</td>
      <td>Altino</td>
      <td>Le Macchie Articciaro</td>
      <td>215.0</td>
      <td>no</td>
      <td>Juniperus oxycedrus L.</td>
      <td>Ginepro coccolone</td>
      <td>42.087228</td>
      <td>14.343047</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ABRUZZO</td>
      <td>Chieti</td>
      <td>Archi</td>
      <td>Serra Castello</td>
      <td>525.0</td>
      <td>no</td>
      <td>Arbutus unedo L.</td>
      <td>Corbezzolo</td>
      <td>42.078653</td>
      <td>14.382539</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ABRUZZO</td>
      <td>Chieti</td>
      <td>Atessa</td>
      <td>Santa Lucia - Piana Sant'Antonio</td>
      <td>125.0</td>
      <td>sì</td>
      <td>Quercus pubescens Willd.</td>
      <td>Roverella</td>
      <td>42.111617</td>
      <td>14.432089</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ABRUZZO</td>
      <td>Chieti</td>
      <td>Bomba</td>
      <td>Cementificio - Casale Nasuti</td>
      <td>400.0</td>
      <td>no</td>
      <td>Quercus pubescens Willd.</td>
      <td>Roverella</td>
      <td>42.052208</td>
      <td>14.363364</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ABRUZZO</td>
      <td>Chieti</td>
      <td>Bomba</td>
      <td>Cementificio - Casale Nasuti</td>
      <td>400.0</td>
      <td>no</td>
      <td>Quercus pubescens Willd.</td>
      <td>Roverella</td>
      <td>42.051975</td>
      <td>14.363603</td>
    </tr>
  </tbody>
</table>
</div>



### geodataframe creation from the dataframe with x (longitude) and y (latitude) 

GeoDataFrame [constructor](https://geopandas.org/reference/geopandas.GeoDataFrame.html):
* a dataframe (or dictionary)
* the CRS
* a geometry field expressed in WKT (eg. POINT (1,3)) 



now we are ready to create the geodataframe.
the operation are:
- creation of a geometry column based on the WKT syntax
- transform the DataFrame in GeoDataFrame


```python
geo_monumental_trees = gpd.GeoDataFrame(
    monumental_trees,
    crs='EPSG:4326',
    geometry=gpd.points_from_xy(monumental_trees.longitude, monumental_trees.latitude))
```


```python
geo_monumental_trees.plot(figsize=(12,12))
plt.show()
```


    
![png](02_Spatial_relationships_and_operations_files/02_Spatial_relationships_and_operations_40_0.png)
    



```python
macroregions.plot()
plt.show()
```

    
![png](02_Spatial_relationships_and_operations_files/02_Spatial_relationships_and_operations_41_0.png)
    

overlay the layers


```python
ax = macroregions.plot(edgecolor='k', facecolor='none', figsize=(15, 10))
geo_monumental_trees.plot(ax=ax,color="green")
plt.show()
```


    
![png](02_Spatial_relationships_and_operations_files/02_Spatial_relationships_and_operations_43_0.png)
    


**ERROR**!
{: .notice--warning}

We need to use the same projection!!!<br/>
The projection used in our geodataframe of ISTAT is EPSG:32632


```python
macroregions.crs
```


    <Projected CRS: EPSG:32632>
    Name: WGS 84 / UTM zone 32N
    Axis Info [cartesian]:
    - E[east]: Easting (metre)
    - N[north]: Northing (metre)
    Area of Use:
    - name: Between 6°E and 12°E, northern hemisphere between equator and 84°N, onshore and offshore. Algeria. Austria. Cameroon. Denmark. Equatorial Guinea. France. Gabon. Germany. Italy. Libya. Liechtenstein. Monaco. Netherlands. Niger. Nigeria. Norway. Sao Tome and Principe. Svalbard. Sweden. Switzerland. Tunisia. Vatican City State.
    - bounds: (6.0, 0.0, 12.0, 84.0)
    Coordinate Operation:
    - name: UTM zone 32N
    - method: Transverse Mercator
    Datum: World Geodetic System 1984
    - Ellipsoid: WGS 84
    - Prime Meridian: Greenwich



overlay the layers by using the same projection


```python
ax = macroregions.to_crs(epsg=4326).plot(edgecolor='k', facecolor='none', figsize=(15, 10))
geo_monumental_trees.plot(ax=ax,color="green")
plt.show()
```


    
![png](02_Spatial_relationships_and_operations_files/02_Spatial_relationships_and_operations_47_0.png)
    


---
# Spatial relationships 
## how two spatial objects relate to each other 

![](https://upload.wikimedia.org/wikipedia/commons/5/55/TopologicSpatialRelarions2.png)

from [https://en.wikipedia.org/wiki/Spatial_relation](https://en.wikipedia.org/wiki/Spatial_relation)

## Relationships between individual objects

Eg.<br>
Is this library located in the north-east Italian macro-region?

we need the north-east italian macro-region in wgs84


```python
macroregions
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
      <th>DEN_RIP</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Nord-Ovest</td>
      <td>MULTIPOLYGON (((568226.691 4874823.573, 568219...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>Nord-Est</td>
      <td>MULTIPOLYGON (((618343.929 4893985.661, 618335...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Centro</td>
      <td>MULTIPOLYGON (((875952.995 4524692.050, 875769...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>Sud</td>
      <td>MULTIPOLYGON (((1083358.846 4416348.741, 10833...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Isole</td>
      <td>MULTIPOLYGON (((822886.611 3935355.889, 822871...</td>
    </tr>
  </tbody>
</table>
</div>


```python
macroregions.geometry[1]
```

    
![svg](02_Spatial_relationships_and_operations_files/02_Spatial_relationships_and_operations_52_0.svg)
    

```python
northeast = macroregions.to_crs(epsg=4326).geometry[1]
```


```python
northeast
```
    
![svg](02_Spatial_relationships_and_operations_files/02_Spatial_relationships_and_operations_54_0.svg)
    

## let's start with just one point

so we will choose a library in Trento (north east italy)


```python
geo_monumental_trees[geo_monumental_trees.municipality == 'Trento'].head(5)
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
      <th>region</th>
      <th>province</th>
      <th>municipality</th>
      <th>place</th>
      <th>altitude</th>
      <th>urban_place</th>
      <th>species_scientific_name</th>
      <th>species_common_name</th>
      <th>latitude</th>
      <th>longitude</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>65</th>
      <td>TRENTO</td>
      <td>Trento</td>
      <td>Trento</td>
      <td>Povo - Ex-Villa Thun - Piazza Manci 15</td>
      <td>405.0</td>
      <td>sì</td>
      <td>Sequoiadendron giganteum (Lindl.) J. Buchholz</td>
      <td>Sequoia gigante</td>
      <td>46.065997</td>
      <td>11.155264</td>
      <td>POINT (11.15526 46.06600)</td>
    </tr>
    <tr>
      <th>66</th>
      <td>TRENTO</td>
      <td>Trento</td>
      <td>Trento</td>
      <td>Maderno - Villa Maria</td>
      <td>488.0</td>
      <td>sì</td>
      <td>Aesculus hippocastanum L.</td>
      <td>Ippocastano</td>
      <td>46.090008</td>
      <td>11.140311</td>
      <td>POINT (11.14031 46.09001)</td>
    </tr>
    <tr>
      <th>67</th>
      <td>TRENTO</td>
      <td>Trento</td>
      <td>Trento</td>
      <td>Povo - Villa Lubich</td>
      <td>455.0</td>
      <td>sì</td>
      <td>Sequoiadendron giganteum (Lindl.) J. Buchholz</td>
      <td>Sequoia gigante</td>
      <td>46.066589</td>
      <td>11.160269</td>
      <td>POINT (11.16027 46.06659)</td>
    </tr>
    <tr>
      <th>68</th>
      <td>TRENTO</td>
      <td>Trento</td>
      <td>Trento</td>
      <td>P. Sso Cimirlo - Casare</td>
      <td>790.0</td>
      <td>no</td>
      <td>Prunus avium L.</td>
      <td>Ciliegio selvatico</td>
      <td>46.064825</td>
      <td>11.186625</td>
      <td>POINT (11.18662 46.06482)</td>
    </tr>
    <tr>
      <th>69</th>
      <td>TRENTO</td>
      <td>Trento</td>
      <td>Trento</td>
      <td>Malga Brigolina</td>
      <td>943.0</td>
      <td>no</td>
      <td>Fagus sylvatica L.</td>
      <td>Faggio</td>
      <td>46.059850</td>
      <td>11.061781</td>
      <td>POINT (11.06178 46.05985)</td>
    </tr>
  </tbody>
</table>
</div>



we choose the first line and extrac the geometry


```python
monumental_tree_in_trento = geo_monumental_trees[geo_monumental_trees.municipality == 'Trento'].head(1).geometry.values[0]
```


```python
monumental_tree_in_trento
```

    
![svg](02_Spatial_relationships_and_operations_files/02_Spatial_relationships_and_operations_60_0.svg)
    

## within relation
in our case:<br>&nbsp;&nbsp;&nbsp;&nbsp;it's the point inside the area?


```python
monumental_tree_in_trento.within(northeast)
```


    True



## contain relation
in our case:<br>&nbsp;&nbsp;&nbsp;&nbsp;does the area contain the point?


```python
northeast.contains(monumental_tree_in_trento)
```


    True



```python
monumental_trees_northeast = geo_monumental_trees[geo_monumental_trees.within(northeast)]
%time
```

    CPU times: user 3 µs, sys: 0 ns, total: 3 µs
    Wall time: 6.44 µs


```python
monumental_trees_northeast.shape
```

    (782, 11)


```python
monumental_trees_northeast.region.unique()
```

    array(['BOLZANO', 'FRIULI VENEZIA GIULIA', 'VENETO', 'EMILIA-ROMAGNA',
           'LAZIO', 'TRENTO'], dtype=object)

---

... **LAZIO** isn't a region of the North-East Italy. There is an error in the data! 

**Reference**
{: .notice--info}

Overview of the different functions to check spatial relationships (*spatial predicate functions*):
* `equals`
* `contains`
* `crosses`
* `disjoint`
* `intersects`
* `overlaps`
* `touches`
* `within`
* `covers`


See [https://shapely.readthedocs.io/en/stable/manual.html#predicates-and-relationships](https://shapely.readthedocs.io/en/stable/manual.html#predicates-and-relationships) for an overview of those methods.


See [https://en.wikipedia.org/wiki/DE-9IM](https://en.wikipedia.org/wiki/DE-9IM) for all details on the semantics of those operations. 


# Spatial Joins

You can create a join like the usual [join](https://pandas.pydata.org/pandas-docs/stable/user_guide/merging.html) between pandas dataframe by using a spatial relationship with the function [geopandas.sjoin](http://geopandas.readthedocs.io/en/latest/reference/geopandas.sjoin.html)


```python
monumental_trees_and_macroregions = gpd.sjoin(macroregions.to_crs(epsg=4326), 
                          geo_monumental_trees, how='inner', predicate='contains', lsuffix='macroregions_', rsuffix='trees')
%time
```

    CPU times: user 3 µs, sys: 1 µs, total: 4 µs
    Wall time: 6.2 µs



```python
monumental_trees_and_macroregions.columns
```

    Index(['COD_RIP', 'DEN_RIP', 'geometry', 'index_trees', 'region', 'province',
           'municipality', 'place', 'altitude', 'urban_place',
           'species_scientific_name', 'species_common_name', 'latitude',
           'longitude'],
          dtype='object')



```python
monumental_trees_and_macroregions.shape
```


    (3660, 14)


```python
monumental_trees_and_macroregions.head(5)
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
      <th>DEN_RIP</th>
      <th>geometry</th>
      <th>index_trees</th>
      <th>region</th>
      <th>province</th>
      <th>municipality</th>
      <th>place</th>
      <th>altitude</th>
      <th>urban_place</th>
      <th>species_scientific_name</th>
      <th>species_common_name</th>
      <th>latitude</th>
      <th>longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Nord-Ovest</td>
      <td>MULTIPOLYGON (((9.85132 44.02340, 9.85122 44.0...</td>
      <td>61</td>
      <td>LIGURIA</td>
      <td>La Spezia</td>
      <td>Monterosso al Mare</td>
      <td>Santuario Madonna di Soviore</td>
      <td>495.0</td>
      <td>no</td>
      <td>Insieme omogeneo di Quercus ilex L.</td>
      <td>Leccio</td>
      <td>44.160292</td>
      <td>9.663494</td>
    </tr>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Nord-Ovest</td>
      <td>MULTIPOLYGON (((9.85132 44.02340, 9.85122 44.0...</td>
      <td>53</td>
      <td>LIGURIA</td>
      <td>La Spezia</td>
      <td>Beverino</td>
      <td>Palazzo Costa - Via Castagna Rossa 15</td>
      <td>180.0</td>
      <td>no</td>
      <td>Cedrus libani A.Richard</td>
      <td>Cedro del Libano</td>
      <td>44.192347</td>
      <td>9.780747</td>
    </tr>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Nord-Ovest</td>
      <td>MULTIPOLYGON (((9.85132 44.02340, 9.85122 44.0...</td>
      <td>55</td>
      <td>LIGURIA</td>
      <td>La Spezia</td>
      <td>Carrodano</td>
      <td>Mattarana - Via Primo Maggio</td>
      <td>473.0</td>
      <td>sì</td>
      <td>Quercus crenata Lam.</td>
      <td>Cerro-Sughera</td>
      <td>44.246278</td>
      <td>9.615614</td>
    </tr>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Nord-Ovest</td>
      <td>MULTIPOLYGON (((9.85132 44.02340, 9.85122 44.0...</td>
      <td>54</td>
      <td>LIGURIA</td>
      <td>La Spezia</td>
      <td>Calice al Cornoviglio</td>
      <td>Monte Ferro</td>
      <td>850.0</td>
      <td>no</td>
      <td>Betula alba L. syn Betula pubescens Ehrh.</td>
      <td>Betulla pubescente</td>
      <td>44.252711</td>
      <td>9.859122</td>
    </tr>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Nord-Ovest</td>
      <td>MULTIPOLYGON (((9.85132 44.02340, 9.85122 44.0...</td>
      <td>64</td>
      <td>LIGURIA</td>
      <td>La Spezia</td>
      <td>Rocchetta di Vara</td>
      <td>Molino Rotato - Pirolo</td>
      <td>710.0</td>
      <td>no</td>
      <td>Quercus pubescens Willd.</td>
      <td>Roverella</td>
      <td>44.282294</td>
      <td>9.781239</td>
    </tr>
  </tbody>
</table>
</div>




```python
monumental_trees_and_macroregions.geom_type.unique()
```




    array(['MultiPolygon'], dtype=object)



... and now you can investigate the new geodataframe


```python
monumental_trees_and_macroregions.groupby(['DEN_RIP']).index_trees.count()
```

    DEN_RIP
    Centro         536
    Isole          568
    Nord-Est       782
    Nord-Ovest     733
    Sud           1041
    Name: index_trees, dtype: int64



```python
monumental_trees_and_macroregions.groupby(['DEN_RIP']).index_trees.count().plot(kind='bar')
plt.show()
```

    
![png](02_Spatial_relationships_and_operations_files/02_Spatial_relationships_and_operations_78_0.png)
    

**spatial join** = *transferring attributes from one layer to another based on their spatial relationship*

Different parts of this operations:
* The GeoDataFrame to which we want add information
* The GeoDataFrame that contains the information we want to add
* The spatial relationship we want to use to match both datasets ('intersects', 'contains', 'within')
* The type of join: left or inner join

![](https://web.natur.cuni.cz/~langhamr/lectures/vtfg1/mapinfo_1/about_gis/Image23.gif)

See also [spatial join nearest](https://geopandas.org/docs/reference/api/geopandas.GeoDataFrame.sjoin_nearest.html#geopandas.GeoDataFrame.sjoin_nearest) that performe a spatial join of two GeoDataFrames based on the distance between their geometries.

Examples: 
[https://geopandas.readthedocs.io/en/latest/docs/user_guide/mergingdata.html](https://geopandas.readthedocs.io/en/latest/docs/user_guide/mergingdata.html)

---

# Spatial operations 
GeoPandas provide analysis methods that return new geometric objects (based on shapely)

See [https://shapely.readthedocs.io/en/stable/manual.html#spatial-analysis-methods](https://shapely.readthedocs.io/en/stable/manual.html#spatial-analysis-methods) for more details.

## buffer
*object.buffer(distance, resolution=16, cap_style=1, join_style=1, mitre_limit=5.0)*

    Returns an approximate representation of all points within a given distance of the this geometric object.


```python
monumental_tree_in_trento_32632 = geo_monumental_trees[geo_monumental_trees.municipality == 'Trento'].to_crs(32632).geometry.values[0]
```

```python
monumental_tree_in_trento_32632
```

    
![svg](02_Spatial_relationships_and_operations_files/02_Spatial_relationships_and_operations_83_0.svg)
    


```python
monumental_tree_in_trento_32632.buffer(9000) # a circle with a ray of 9000 meters
```

    
![svg](02_Spatial_relationships_and_operations_files/02_Spatial_relationships_and_operations_84_0.svg)
    

## simplify

*object.simplify(tolerance, preserve_topology=True)*

    Returns a simplified representation of the geometric object.



```python
northeast_geometry = macroregions[macroregions.COD_RIP==2].geometry.values[0]
```


```python
northeast_geometry
```

    
![svg](02_Spatial_relationships_and_operations_files/02_Spatial_relationships_and_operations_87_0.svg)
    


```python
northeast_geometry.simplify(10000,preserve_topology=False)
```

    
![svg](02_Spatial_relationships_and_operations_files/02_Spatial_relationships_and_operations_88_0.svg)
    

## Es. symmetric_difference
*object.symmetric_difference(other)*

    Returns a representation of the points in this object not in the other geometric object, and the points in the other not in this geometric object.


```python
northeast_geometry.simplify(10000,preserve_topology=False).symmetric_difference(monumental_tree_in_trento_32632.buffer(9000))
```

    
![svg](02_Spatial_relationships_and_operations_files/02_Spatial_relationships_and_operations_90_0.svg)
    

**Remember:**
{: .notice--info}

GeoPandas (and Shapely for the individual objects) provides a whole lot of basic methods to analyse the geospatial data (distance, length, centroid, boundary, convex_hull, simplify, transform, ....), much more than the few that we can touch in this tutorial.

An overview of all methods provided by GeoPandas can be found here: <a href="https://geopandas.readthedocs.io/en/latest/docs/reference.html">https://geopandas.readthedocs.io/en/latest/docs/reference.html

---
# Clip 

Extracts input features that overlay the clip features.

![](https://desktop.arcgis.com/en/arcmap/10.3/tools/analysis-toolbox/GUID-6D3322A8-57EA-4D24-9FFE-2A9E7C6B29EC-web.gif)

```python
 geopandas.clip(gdf, mask, keep_geom_type=False)
 ```
 *gdf* = geodataframe<Br/>
 *mask* = polygon 


```python
municipalities_inside_circle = municipalities.clip(monumental_tree_in_trento_32632.buffer(9000))
```


```python
municipalities_inside_circle.COMUNE.unique()
```


    array(['Caldonazzo', 'Altopiano della Vigolana', 'Calceranica al Lago',
           'Tenna', 'Garniga Terme', 'Vignola-Falesina', 'Pergine Valsugana',
           'Trento', 'Civezzano', 'Vallelaghi', 'Fornace', 'Baselga di Pinè',
           'Albiano', 'Lavis', 'Lona-Lases', 'Giovo'], dtype=object)


```python
municipalities_inside_circle.plot(figsize=(10,10))
plt.show()
```
    
![png](02_Spatial_relationships_and_operations_files/02_Spatial_relationships_and_operations_95_0.png)
    
---
# Aggregation with dissolve

Spatial data are often more granular than we need. For example, we have the data of the macro-regions but we don't have a geometry with the border of Italy.

If we have a columns to operate a *groupby* we can solve it but to create the geometry we need the function *dissolve*.

```python
macroregions['nation']='italy'
```


```python
macroregions
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
      <th>DEN_RIP</th>
      <th>geometry</th>
      <th>nation</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Nord-Ovest</td>
      <td>MULTIPOLYGON (((568226.691 4874823.573, 568219...</td>
      <td>italy</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>Nord-Est</td>
      <td>MULTIPOLYGON (((618343.929 4893985.661, 618335...</td>
      <td>italy</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Centro</td>
      <td>MULTIPOLYGON (((875952.995 4524692.050, 875769...</td>
      <td>italy</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>Sud</td>
      <td>MULTIPOLYGON (((1083358.846 4416348.741, 10833...</td>
      <td>italy</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Isole</td>
      <td>MULTIPOLYGON (((822886.611 3935355.889, 822871...</td>
      <td>italy</td>
    </tr>
  </tbody>
</table>
</div>


```python
italy = macroregions[['nation', 'geometry']]
```

```python
italy.plot()
plt.show()
```
    
![png](02_Spatial_relationships_and_operations_files/02_Spatial_relationships_and_operations_100_0.png)
    

```python
italy = italy.to_crs(epsg=4326).dissolve(by='nation')
%time
```

    CPU times: user 4 µs, sys: 0 ns, total: 4 µs
    Wall time: 7.63 µs



```python
italy
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
      <th>geometry</th>
    </tr>
    <tr>
      <th>nation</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>italy</th>
      <td>MULTIPOLYGON (((12.55959 35.50960, 12.55987 35...</td>
    </tr>
  </tbody>
</table>
</div>




```python
italy.geometry[0]
```

    
![svg](02_Spatial_relationships_and_operations_files/02_Spatial_relationships_and_operations_103_0.svg)
    




```python
italy.to_crs(epsg=32632).geometry[0]
```

    
![svg](02_Spatial_relationships_and_operations_files/02_Spatial_relationships_and_operations_104_0.svg)
    


**Rember**
{: .notice--info}

dissolve can be thought of as doing three things: (a) it **dissolves** all the geometries within a given group together into a single geometric feature (using the *unary_union* method), and (b) it **aggregates** all the rows of data in a group using *groupby.aggregate()*, and (c) it **combines** those two results.
    
An overview of all methods provided by GeoPandas can be found here: <a href="http://geopandas.org/aggregation_with_dissolve.html">http://geopandas.org/aggregation_with_dissolve.html

---
# Overlay

Spatial overlays allow you to compare two GeoDataFrames containing polygon or multipolygon geometries and create a new GeoDataFrame with the new geometries representing the spatial combination and merged properties. This allows you to answer questions like

    What are the demographics of the census tracts within 90km from a point?

The basic idea is demonstrated by the graphic below but keep in mind that overlays operate at the dataframe level, not on individual geometries, and the properties from both are retained

![](https://docs.qgis.org/testing/en/_images/overlay_operations.png)

source: https://geopandas.org/gallery/overlays.html


```python
macroregion_gdf = macroregions[macroregions.COD_RIP==2].to_crs(epsg=32632)
```

```python
overlay = italy.to_crs(epsg=32632).overlay(macroregion_gdf, how="difference")
```

```python
overlay.plot()
plt.show()
```
    
![png](02_Spatial_relationships_and_operations_files/02_Spatial_relationships_and_operations_109_0.png)
    
---
# Exercise
 
1 - create the geodataframe of the [gas&oil stations](https://www.mise.gov.it/images/exportCSV/anagrafica_impianti_attivi.csv) of Italy 
  - data from the italian [Ministry of Economic Development](https://www.mise.gov.it)
  - count the total of the gas&oil stations for each muncipality of Trentino

2 - identify the difference of municipalities in Trentino from 2020 to 2021
  - identify which municipalities are created from aggregation to others
  - find the biggest new municipality of Trentino and show all the italian municipalities with bordering it
  - create the macroarea of all the municipalities bordering with it
  - for each gas&oil station in the macro-area, calculate how many monumental trees have been within a 500m radius
  
3 - creates a polygon that contains all the monumental trees inside the area
  - identify all the gas&oil stations in this area which are within 2km of each other
  - save the polygon in geopackage with the attribute "description" with the name of the gas&oil station