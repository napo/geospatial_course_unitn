---
title: "Solution 04"
permalink: /solutions/04-retrieving_data_from_osm
excerpt: "Retrieving data from OpenStreetMap"
last_modified_at: 2022-10-30T06:48:03-01:21
header:
  teaser: https://raw.githubusercontent.com/napo/geospatial_course_unitn/master/images/mezzolombardo_drinking_waters.jpg
#redirect_from:
#  - /theme-setup/
toc: true
---
---

# Setup


```python
try:
  import pygeos
except ModuleNotFoundError as e:
  !pip install pygeos==0.10.2
  import pygeos
```


```python
try:
  import geopandas as gpd
except ModuleNotFoundError as e:
  !pip install geopandas==0.10.0
  import geopandas as gpd

if gpd.__version__ != "0.10.0":
  !pip install -U geopandas==0.10.0
  import geopandas as gpd
```


```python
try:
  import pyrosm
except ModuleNotFoundError as e:
  !pip install pyrosm==0.6.1
  import pyrosm
```

# Exercise
- download from OpenStreetMap all supermarkets inside the bounding box of the city in this point<br/>
   latitude: 45.4395<br/>
   longitude: 11.09351
- identify the longest road of the city (state roads, walking routes, motorways are excluded). Please use "unclassified"
- How many drinking water are in this city?
- How many benches in this city have the backrest?


## download from OpenStreetMap all supermarkets inside the bounding box of the city in this point and

   latitude: 45.4395<br/>
   longitude: 11.09351


```python
import geopandas as gpd
```


```python
latitude = 45.4395
longitude = 12.3478
```

## find the city

### method 1 - reverse geocoding


```python
from geopy.geocoders import ArcGIS
```


```python
latlon = str(latitude) + "," + str(longitude);
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




    {'Match_addr': '30122',
     'LongLabel': '30122, ITA',
     'ShortLabel': '30122',
     'Addr_type': 'Postal',
     'Type': '',
     'PlaceName': '30122',
     'AddNum': '',
     'Address': '',
     'Block': '',
     'Sector': '',
     'Neighborhood': '',
     'District': '',
     'City': 'Venezia',
     'MetroArea': '',
     'Subregion': 'Venezia',
     'Region': 'Veneto',
     'RegionAbbr': '',
     'Territory': '',
     'Postal': '30122',
     'PostalExt': '',
     'CntryName': 'Italia',
     'CountryCode': 'ITA'}




```python
city = location.raw['City']
```


```python
city
```




    'Venezia'



### method 2 - spatial relation


```python
from shapely.geometry import Point
```


```python
point = Point(longitude, latitude)
```


```python
urlmunicipalities2022 = 'https://github.com/napo/geospatial_course_unitn/raw/master/data/istat/shapefile_istat_municipalities_2022.zip'
municipalities2022 = gpd.read_file(urlmunicipalities2022)
```


```python
muncipality = municipalities2022[municipalities2022.to_crs(epsg=4326).geometry.contains(point)]
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
      <th>Shape_Area</th>
      <th>Shape_Leng</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>3692</th>
      <td>2</td>
      <td>5</td>
      <td>27</td>
      <td>227</td>
      <td>227</td>
      <td>27042</td>
      <td>027042</td>
      <td>Venezia</td>
      <td>None</td>
      <td>1</td>
      <td>4.158927e+08</td>
      <td>170837.245485</td>
      <td>POLYGON ((780145.784 5049170.898, 780115.597 5...</td>
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




    'Venezia'



## download from OpenStreetMap

## find the boundig box of Venice


```python
municipality = municipalities2022[municipalities2022.COMUNE==city]
```

## Donwload the PBF of Venice from OSM Estratti

![](https://raw.githubusercontent.com/napo/geospatial_course_unitn/master/images/osm_estratti_mezzolombardo.jpg)


```python
url_download_venice_pbf = 'https://osmit-estratti-test.wmcloud.org/dati/poly/comuni/pbf/027042_Venezia_poly.osm.pbf'
import urllib.request
urllib.request.urlretrieve(url_download_venice_pbf ,"venice_osm.pbf")
```




    ('venice_osm.pbf', <http.client.HTTPMessage at 0x7efc48639960>)



## find all the supermarkets in the area




```python
import pyrosm
```

![](https://raw.githubusercontent.com/napo/geospatial_course_unitn/master/images/tag_supermarket.png)


```python
osm = pyrosm.OSM("venice_osm.pbf") 
```


```python
custom_filter = {'shop': ['supermarket']}
```


```python
supermarkets = osm.get_pois(custom_filter=custom_filter)
```

    /home/napo/.local/lib/python3.10/site-packages/pyrosm/pyrosm.py:576: ShapelyDeprecationWarning: The array interface is deprecated and will no longer work in Shapely 2.0. Convert the '.coords' to a numpy array instead.
      gdf = get_poi_data(
    /home/napo/.local/lib/python3.10/site-packages/geopandas/array.py:1406: UserWarning: CRS not set for some of the concatenation inputs. Setting output's CRS as WGS 84 (the single non-null crs provided).



```python
supermarkets.shape
```




    (102, 23)




```python
supermarkets.head(5)
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
      <th>lon</th>
      <th>version</th>
      <th>tags</th>
      <th>id</th>
      <th>changeset</th>
      <th>lat</th>
      <th>timestamp</th>
      <th>addr:city</th>
      <th>addr:country</th>
      <th>addr:housenumber</th>
      <th>...</th>
      <th>email</th>
      <th>name</th>
      <th>opening_hours</th>
      <th>operator</th>
      <th>phone</th>
      <th>website</th>
      <th>organic</th>
      <th>shop</th>
      <th>geometry</th>
      <th>osm_type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>12.327674</td>
      <td>8</td>
      <td>{"addr:neighbourhood":"Santa Croce","brand":"C...</td>
      <td>443122709</td>
      <td>0.0</td>
      <td>45.440384</td>
      <td>1628972013</td>
      <td>None</td>
      <td>None</td>
      <td>1491a</td>
      <td>...</td>
      <td>None</td>
      <td>Coop</td>
      <td>Mo-Sa 08:30-20:00; Su 09:00-20:00</td>
      <td>None</td>
      <td>+39 041 2750218</td>
      <td>None</td>
      <td>None</td>
      <td>supermarket</td>
      <td>POINT (12.32767 45.44038)</td>
      <td>node</td>
    </tr>
    <tr>
      <th>1</th>
      <td>12.200390</td>
      <td>2</td>
      <td>None</td>
      <td>566696229</td>
      <td>0.0</td>
      <td>45.482265</td>
      <td>1258587411</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>...</td>
      <td>None</td>
      <td>Alì</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>supermarket</td>
      <td>POINT (12.20039 45.48227)</td>
      <td>node</td>
    </tr>
    <tr>
      <th>2</th>
      <td>12.208943</td>
      <td>1</td>
      <td>None</td>
      <td>566696230</td>
      <td>0.0</td>
      <td>45.485161</td>
      <td>1258568581</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>...</td>
      <td>None</td>
      <td>Cadoro</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>supermarket</td>
      <td>POINT (12.20894 45.48516)</td>
      <td>node</td>
    </tr>
    <tr>
      <th>3</th>
      <td>12.253382</td>
      <td>4</td>
      <td>None</td>
      <td>617964249</td>
      <td>0.0</td>
      <td>45.492580</td>
      <td>1492417346</td>
      <td>None</td>
      <td>None</td>
      <td>43a</td>
      <td>...</td>
      <td>None</td>
      <td>Spak</td>
      <td>None</td>
      <td>None</td>
      <td>+39 041 5351288</td>
      <td>None</td>
      <td>None</td>
      <td>supermarket</td>
      <td>POINT (12.25338 45.49258)</td>
      <td>node</td>
    </tr>
    <tr>
      <th>4</th>
      <td>12.324074</td>
      <td>10</td>
      <td>{"brand":"Conad City","brand:wikidata":"Q57543...</td>
      <td>626923862</td>
      <td>0.0</td>
      <td>45.433987</td>
      <td>1603553460</td>
      <td>Venezia</td>
      <td>None</td>
      <td>3017</td>
      <td>...</td>
      <td>None</td>
      <td>Conad City</td>
      <td>Mo-Sa 07:30-20:30; Su 09:00-14:00,16:00-19:30</td>
      <td>Conad</td>
      <td>None</td>
      <td>https://www.conad.it</td>
      <td>None</td>
      <td>supermarket</td>
      <td>POINT (12.32407 45.43399)</td>
      <td>node</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 23 columns</p>
</div>




```python
print("there are %s supermarkets in Venice" % supermarkets.shape[0])
```

    there are 102 supermarkets in Venice



```python
supermarkets.name.unique()
```




    array(['Coop', 'Alì', 'Cadoro', 'Spak', 'Conad City', 'Simply', 'Prix',
           'Spazio Conad', 'Despar', 'inCoop', 'Supermercato SISA',
           'Minimarket', None, 'Pam', 'Supermarket "Cadoro"', 'Lidl',
           'Camping Fusina', 'Eurospesa', 'Pam Local Corso del Popolo',
           'Supermercati Maxi', 'Punto Simply', 'Ali Supermercato',
           'Cuore Bio', 'Crai', 'NaturaSì', 'InCoop', 'Alì via Sforza',
           'Alì Supermercati', "In's Mercato", 'A&O', 'L’angolo Giallo',
           'Ballarin Paolo E C. S.N.C.', 'Dolce Bianchi Venezia', 'Interspar',
           'Conad', 'Tigota', 'Aldi', 'Aliper', 'Mega Marghera', "iN's",
           'Eurospar', 'EuroSpin', 'Famila Superstore', 'IperLando'],
          dtype=object)



# identify the longest road of the city



```python
roads = osm.get_network(network_type='all')
```


```python
roads.columns
```




    Index(['access', 'area', 'bicycle', 'bridge', 'cycleway', 'est_width', 'foot',
           'footway', 'highway', 'int_ref', 'junction', 'lanes', 'lit', 'maxspeed',
           'motorcar', 'motor_vehicle', 'name', 'oneway', 'overtaking', 'path',
           'psv', 'ref', 'service', 'segregated', 'sidewalk', 'smoothness',
           'surface', 'tracktype', 'tunnel', 'width', 'id', 'timestamp', 'version',
           'tags', 'osm_type', 'geometry', 'length'],
          dtype='object')




```python
roads.highway.unique()
```




    array(['pedestrian', 'service', 'unclassified', 'residential', 'footway',
           'tertiary', 'primary', 'cycleway', 'steps', 'secondary',
           'motorway_link', 'track', 'secondary_link', 'primary_link',
           'motorway', 'path', 'tertiary_link', 'living_street',
           'construction', 'rest_area', 'services', 'elevator', 'corridor',
           'bridleway', 'proposed'], dtype=object)



![](https://github.com/napo/geospatial_course_unitn/raw/master/images/tag_highways.gif)

## lenght of unclassified and residential togheter


```python
roads_unclassified_residential = roads[(roads.highway=='unclassified') |  (roads.highway == 'residential')]
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
      <th>est_width</th>
      <th>foot</th>
      <th>footway</th>
      <th>highway</th>
      <th>int_ref</th>
      <th>...</th>
      <th>tracktype</th>
      <th>tunnel</th>
      <th>width</th>
      <th>id</th>
      <th>timestamp</th>
      <th>version</th>
      <th>tags</th>
      <th>osm_type</th>
      <th>geometry</th>
      <th>length</th>
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
      <td>None</td>
      <td>unclassified</td>
      <td>None</td>
      <td>...</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>4438335</td>
      <td>1611572910</td>
      <td>9</td>
      <td>None</td>
      <td>way</td>
      <td>MULTILINESTRING ((12.38501 45.42321, 12.38511 ...</td>
      <td>538.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>unclassified</td>
      <td>None</td>
      <td>...</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>4438345</td>
      <td>1611572458</td>
      <td>6</td>
      <td>None</td>
      <td>way</td>
      <td>MULTILINESTRING ((12.38398 45.42382, 12.38346 ...</td>
      <td>51.0</td>
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
      <td>None</td>
      <td>residential</td>
      <td>None</td>
      <td>...</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>4448298</td>
      <td>1399909744</td>
      <td>9</td>
      <td>None</td>
      <td>way</td>
      <td>MULTILINESTRING ((12.36842 45.40751, 12.36896 ...</td>
      <td>168.0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>unclassified</td>
      <td>None</td>
      <td>...</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>4597767</td>
      <td>1643983716</td>
      <td>17</td>
      <td>{"cycleway:right":"no","layer":"1","noname":"y...</td>
      <td>way</td>
      <td>MULTILINESTRING ((12.31524 45.44175, 12.31523 ...</td>
      <td>39.0</td>
    </tr>
    <tr>
      <th>13</th>
      <td>destination</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>residential</td>
      <td>None</td>
      <td>...</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>4710516</td>
      <td>1467135189</td>
      <td>5</td>
      <td>{"note":"durante i giorni di scuola accesso ri...</td>
      <td>way</td>
      <td>MULTILINESTRING ((12.27803 45.50600, 12.27798 ...</td>
      <td>67.0</td>
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
    </tr>
    <tr>
      <th>21153</th>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>residential</td>
      <td>None</td>
      <td>...</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>1054635814</td>
      <td>1650871543</td>
      <td>1</td>
      <td>None</td>
      <td>way</td>
      <td>MULTILINESTRING ((12.24553 45.50209, 12.24560 ...</td>
      <td>32.0</td>
    </tr>
    <tr>
      <th>21154</th>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>residential</td>
      <td>None</td>
      <td>...</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>1054635815</td>
      <td>1650871543</td>
      <td>1</td>
      <td>None</td>
      <td>way</td>
      <td>MULTILINESTRING ((12.24561 45.50221, 12.24556 ...</td>
      <td>17.0</td>
    </tr>
    <tr>
      <th>21164</th>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>unclassified</td>
      <td>None</td>
      <td>...</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>1055712580</td>
      <td>1651150935</td>
      <td>3</td>
      <td>{"cycleway:both":"no"}</td>
      <td>way</td>
      <td>MULTILINESTRING ((12.29539 45.53549, 12.29539 ...</td>
      <td>8.0</td>
    </tr>
    <tr>
      <th>21165</th>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>unclassified</td>
      <td>None</td>
      <td>...</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>1055712581</td>
      <td>1651150947</td>
      <td>2</td>
      <td>{"cycleway:both":"no"}</td>
      <td>way</td>
      <td>MULTILINESTRING ((12.29430 45.53416, 12.29319 ...</td>
      <td>87.0</td>
    </tr>
    <tr>
      <th>21177</th>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>residential</td>
      <td>None</td>
      <td>...</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>1058284862</td>
      <td>1652007383</td>
      <td>1</td>
      <td>{"vehicle:conditional":"private @ (22:00-04:00)"}</td>
      <td>way</td>
      <td>MULTILINESTRING ((12.23248 45.48717, 12.23248 ...</td>
      <td>137.0</td>
    </tr>
  </tbody>
</table>
<p>4022 rows × 37 columns</p>
</div>




```python
roads_unclassified_residential['name'].value_counts()
```




    Via Eugenio Carlo Pertini    35
    Via Ca' Solaro               23
    Via Vallon                   23
    Via Colombara                23
    Via Evaristo Scaramuzza      21
                                 ..
    Via Prete Ermanno             1
    Via dei Gradenighi            1
    Via dei Mandarini             1
    Via Egidio Marcon             1
    Santa Maria dei Battuti       1
    Name: name, Length: 1461, dtype: int64




```python
names = roads_unclassified_residential.name.unique()
```


```python
names
```




    array(['Viale Umberto Klinger', 'Via Giannantonio Selva',
           'Via Dardanelli', ..., 'via Caravaggio', 'Via Trento',
           'Santa Maria dei Battuti'], dtype=object)



The best projection for Venice is [WGS84 UTM 33N](https://epsg.io/32633) - epsg:32633


```python
roads_lenght = {}
rodas_in_meters = roads_unclassified_residential.to_crs(epsg=32633)
for name in names:
  road = rodas_in_meters[rodas_in_meters.name==name]
  road_lenght = road.length.sum()
  roads_lenght[name] = road_lenght

```


```python
#roads_lenght
```


```python
max(roads_lenght, key=roads_lenght.get)
```




    'Via Litomarino'




```python
longest_road = max(roads_lenght, key=roads_lenght.get)
```


```python
roads_lenght[longest_road]
```




    4882.770029371328




```python
print("the longest road (unclassified + residential) in Venice is %s and is long %.1f km" % (str(longest_road),roads_lenght[longest_road]/1000))
```

    the longest road (unclassified + residential) in Venice is Via Litomarino and is long 4.9 km


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
print("the longest road (unclassified) is %s and is long %.1f km" % (str(longest_road),roads_lenght[longest_road]/1000))
```

    the longest road (unclassified) is Via Litomarino and is long 4.9 km


## where is this road?
... we can use overpass-turbo.eu with this wizard query

```
name='Via Litomarino in Venezia'
```

[https://overpass-turbo.eu/s/1nf9](https://overpass-turbo.eu/s/1nf9)

![](https://raw.githubusercontent.com/napo/geospatial_course_unitn/master/images/overpassturbo_mezzolombardo_viatrento.png)

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




    (181, 15)




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
      <th>lon</th>
      <th>version</th>
      <th>tags</th>
      <th>id</th>
      <th>changeset</th>
      <th>lat</th>
      <th>timestamp</th>
      <th>addr:street</th>
      <th>name</th>
      <th>amenity</th>
      <th>drinking_water</th>
      <th>fountain</th>
      <th>source</th>
      <th>geometry</th>
      <th>osm_type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>12.234090</td>
      <td>2</td>
      <td>{"check_date":"2022-03-26"}</td>
      <td>315040103</td>
      <td>0</td>
      <td>45.534821</td>
      <td>1648284842</td>
      <td>None</td>
      <td>None</td>
      <td>drinking_water</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>POINT (12.23409 45.53482)</td>
      <td>node</td>
    </tr>
    <tr>
      <th>1</th>
      <td>12.324445</td>
      <td>5</td>
      <td>None</td>
      <td>639811824</td>
      <td>0</td>
      <td>45.437145</td>
      <td>1493048539</td>
      <td>None</td>
      <td>None</td>
      <td>drinking_water</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>POINT (12.32444 45.43715)</td>
      <td>node</td>
    </tr>
    <tr>
      <th>2</th>
      <td>12.286287</td>
      <td>3</td>
      <td>{"access":"private"}</td>
      <td>699811494</td>
      <td>0</td>
      <td>45.520275</td>
      <td>1612044310</td>
      <td>None</td>
      <td>None</td>
      <td>drinking_water</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>POINT (12.28629 45.52028)</td>
      <td>node</td>
    </tr>
    <tr>
      <th>3</th>
      <td>12.315189</td>
      <td>2</td>
      <td>None</td>
      <td>1023518847</td>
      <td>0</td>
      <td>45.427383</td>
      <td>1291582192</td>
      <td>None</td>
      <td>None</td>
      <td>drinking_water</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>POINT (12.31519 45.42738)</td>
      <td>node</td>
    </tr>
  </tbody>
</table>
</div>




```python
drinking_water.explore()
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe srcdoc="&lt;!DOCTYPE html&gt;
&lt;head&gt;    
    &lt;meta http-equiv=&quot;content-type&quot; content=&quot;text/html; charset=UTF-8&quot; /&gt;

        &lt;script&gt;
            L_NO_TOUCH = false;
            L_DISABLE_3D = false;
        &lt;/script&gt;

    &lt;style&gt;html, body {width: 100%;height: 100%;margin: 0;padding: 0;}&lt;/style&gt;
    &lt;style&gt;#map {position:absolute;top:0;bottom:0;right:0;left:0;}&lt;/style&gt;
    &lt;script src=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://code.jquery.com/jquery-1.12.4.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/js/bootstrap.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap-theme.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/font-awesome/4.6.3/css/font-awesome.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/gh/python-visualization/folium/folium/templates/leaflet.awesome.rotate.min.css&quot;/&gt;

            &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width,
                initial-scale=1.0, maximum-scale=1.0, user-scalable=no&quot; /&gt;
            &lt;style&gt;
                #map_02bf09d5d4db15b5ac3ea9f15e6254ad {
                    position: relative;
                    width: 100.0%;
                    height: 100.0%;
                    left: 0.0%;
                    top: 0.0%;
                }
            &lt;/style&gt;


                    &lt;style&gt;
                        .foliumtooltip {

                        }
                       .foliumtooltip table{
                            margin: auto;
                        }
                        .foliumtooltip tr{
                            text-align: left;
                        }
                        .foliumtooltip th{
                            padding: 2px; padding-right: 8px;
                        }
                    &lt;/style&gt;

&lt;/head&gt;
&lt;body&gt;    

            &lt;div class=&quot;folium-map&quot; id=&quot;map_02bf09d5d4db15b5ac3ea9f15e6254ad&quot; &gt;&lt;/div&gt;

&lt;/body&gt;
&lt;script&gt;    

            var map_02bf09d5d4db15b5ac3ea9f15e6254ad = L.map(
                &quot;map_02bf09d5d4db15b5ac3ea9f15e6254ad&quot;,
                {
                    center: [45.39989471435547, 12.305040836334229],
                    crs: L.CRS.EPSG3857,
                    zoom: 10,
                    zoomControl: true,
                    preferCanvas: false,
                }
            );
            L.control.scale().addTo(map_02bf09d5d4db15b5ac3ea9f15e6254ad);





            var tile_layer_92cf5a273b3f463d33f09c63cabbecd8 = L.tileLayer(
                &quot;https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png&quot;,
                {&quot;attribution&quot;: &quot;Data by \u0026copy; \u003ca href=\&quot;http://openstreetmap.org\&quot;\u003eOpenStreetMap\u003c/a\u003e, under \u003ca href=\&quot;http://www.openstreetmap.org/copyright\&quot;\u003eODbL\u003c/a\u003e.&quot;, &quot;detectRetina&quot;: false, &quot;maxNativeZoom&quot;: 18, &quot;maxZoom&quot;: 18, &quot;minZoom&quot;: 0, &quot;noWrap&quot;: false, &quot;opacity&quot;: 1, &quot;subdomains&quot;: &quot;abc&quot;, &quot;tms&quot;: false}
            ).addTo(map_02bf09d5d4db15b5ac3ea9f15e6254ad);


            map_02bf09d5d4db15b5ac3ea9f15e6254ad.fitBounds(
                [[45.26496887207031, 12.191202163696289], [45.534820556640625, 12.418879508972168]],
                {}
            );


        function geo_json_f410776135205fd1f701acc20e208142_styler(feature) {
            switch(feature.id) {
                default:
                    return {&quot;fillOpacity&quot;: 0.5, &quot;weight&quot;: 2};
            }
        }
        function geo_json_f410776135205fd1f701acc20e208142_highlighter(feature) {
            switch(feature.id) {
                default:
                    return {&quot;fillOpacity&quot;: 0.75};
            }
        }
        function geo_json_f410776135205fd1f701acc20e208142_pointToLayer(feature, latlng) {
            var opts = {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#3388ff&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#3388ff&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 2, &quot;stroke&quot;: true, &quot;weight&quot;: 3};

            let style = geo_json_f410776135205fd1f701acc20e208142_styler(feature)
            Object.assign(opts, style)

            return new L.CircleMarker(latlng, opts)
        }

        function geo_json_f410776135205fd1f701acc20e208142_onEachFeature(feature, layer) {
            layer.on({
                mouseout: function(e) {
                    if(typeof e.target.setStyle === &quot;function&quot;){
                        geo_json_f410776135205fd1f701acc20e208142.resetStyle(e.target);
                    }
                },
                mouseover: function(e) {
                    if(typeof e.target.setStyle === &quot;function&quot;){
                        const highlightStyle = geo_json_f410776135205fd1f701acc20e208142_highlighter(e.target.feature)
                        e.target.setStyle(highlightStyle);
                    }
                },
            });
        };
        var geo_json_f410776135205fd1f701acc20e208142 = L.geoJson(null, {
                onEachFeature: geo_json_f410776135205fd1f701acc20e208142_onEachFeature,

                style: geo_json_f410776135205fd1f701acc20e208142_styler,
                pointToLayer: geo_json_f410776135205fd1f701acc20e208142_pointToLayer
        });

        function geo_json_f410776135205fd1f701acc20e208142_add (data) {
            geo_json_f410776135205fd1f701acc20e208142
                .addData(data)
                .addTo(map_02bf09d5d4db15b5ac3ea9f15e6254ad);
        }
            geo_json_f410776135205fd1f701acc20e208142_add({&quot;bbox&quot;: [12.191202163696289, 45.26496887207031, 12.418879508972168, 45.534820556640625], &quot;features&quot;: [{&quot;bbox&quot;: [12.234089851379395, 45.534820556640625, 12.234089851379395, 45.534820556640625], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.234089851379395, 45.534820556640625], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;0&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 315040103, &quot;lat&quot;: 45.534820556640625, &quot;lon&quot;: 12.234089851379395, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;check_date\&quot;:\&quot;2022-03-26\&quot;}&quot;, &quot;timestamp&quot;: 1648284842, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.324444770812988, 45.4371452331543, 12.324444770812988, 45.4371452331543], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.324444770812988, 45.4371452331543], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;1&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 639811824, &quot;lat&quot;: 45.4371452331543, &quot;lon&quot;: 12.324444770812988, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1493048539, &quot;version&quot;: 5}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.286287307739258, 45.5202751159668, 12.286287307739258, 45.5202751159668], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.286287307739258, 45.5202751159668], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;2&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 699811494, &quot;lat&quot;: 45.5202751159668, &quot;lon&quot;: 12.286287307739258, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;access\&quot;:\&quot;private\&quot;}&quot;, &quot;timestamp&quot;: 1612044310, &quot;version&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.315189361572266, 45.42738342285156, 12.315189361572266, 45.42738342285156], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.315189361572266, 45.42738342285156], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;3&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 1023518847, &quot;lat&quot;: 45.42738342285156, &quot;lon&quot;: 12.315189361572266, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1291582192, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.337505340576172, 45.426025390625, 12.337505340576172, 45.426025390625], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.337505340576172, 45.426025390625], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;4&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 1024881830, &quot;lat&quot;: 45.426025390625, &quot;lon&quot;: 12.337505340576172, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1291640566, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.329390525817871, 45.42575454711914, 12.329390525817871, 45.42575454711914], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.329390525817871, 45.42575454711914], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;5&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 1024900285, &quot;lat&quot;: 45.42575454711914, &quot;lon&quot;: 12.329390525817871, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1534368372, &quot;version&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.23444652557373, 45.48640060424805, 12.23444652557373, 45.48640060424805], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.23444652557373, 45.48640060424805], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;6&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 1024935994, &quot;lat&quot;: 45.48640060424805, &quot;lon&quot;: 12.23444652557373, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1463427521, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.239930152893066, 45.49335479736328, 12.239930152893066, 45.49335479736328], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.239930152893066, 45.49335479736328], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;7&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 1024970917, &quot;lat&quot;: 45.49335479736328, &quot;lon&quot;: 12.239930152893066, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1291646721, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.3777494430542, 45.42443084716797, 12.3777494430542, 45.42443084716797], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.3777494430542, 45.42443084716797], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;8&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 1303944111, &quot;lat&quot;: 45.42443084716797, &quot;lon&quot;: 12.3777494430542, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1318164788, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.380293846130371, 45.42802429199219, 12.380293846130371, 45.42802429199219], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.380293846130371, 45.42802429199219], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;9&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 1303944192, &quot;lat&quot;: 45.42802429199219, &quot;lon&quot;: 12.380293846130371, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1306610030, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.3256196975708, 45.434242248535156, 12.3256196975708, 45.434242248535156], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.3256196975708, 45.434242248535156], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;10&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 1346173744, &quot;lat&quot;: 45.434242248535156, &quot;lon&quot;: 12.3256196975708, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;bottle\&quot;:\&quot;yes\&quot;,\&quot;image\&quot;:\&quot;https://i.imgur.com/huNWSkc.jpg\&quot;}&quot;, &quot;timestamp&quot;: 1629808672, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.325904846191406, 45.43495178222656, 12.325904846191406, 45.43495178222656], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.325904846191406, 45.43495178222656], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;11&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 1346173747, &quot;lat&quot;: 45.43495178222656, &quot;lon&quot;: 12.325904846191406, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;bottle\&quot;:\&quot;yes\&quot;,\&quot;image\&quot;:\&quot;https://i.imgur.com/fC0IGjy.jpg\&quot;}&quot;, &quot;timestamp&quot;: 1629808951, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.319907188415527, 45.434871673583984, 12.319907188415527, 45.434871673583984], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.319907188415527, 45.434871673583984], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;12&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 1346173748, &quot;lat&quot;: 45.434871673583984, &quot;lon&quot;: 12.319907188415527, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;wheelchair\&quot;:\&quot;yes\&quot;}&quot;, &quot;timestamp&quot;: 1573293104, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.323625564575195, 45.43110275268555, 12.323625564575195, 45.43110275268555], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.323625564575195, 45.43110275268555], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;13&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 1426023636, &quot;lat&quot;: 45.43110275268555, &quot;lon&quot;: 12.323625564575195, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;image\&quot;:\&quot;https://i.imgur.com/KIYAdRT.jpg\&quot;}&quot;, &quot;timestamp&quot;: 1629886793, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.331242561340332, 45.498069763183594, 12.331242561340332, 45.498069763183594], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.331242561340332, 45.498069763183594], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;14&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 1434235834, &quot;lat&quot;: 45.498069763183594, &quot;lon&quot;: 12.331242561340332, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1466264508, &quot;version&quot;: 4}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.332283020019531, 45.42534255981445, 12.332283020019531, 45.42534255981445], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.332283020019531, 45.42534255981445], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;15&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 1504407646, &quot;lat&quot;: 45.42534255981445, &quot;lon&quot;: 12.332283020019531, &quot;name&quot;: &quot;Chiesa del Santissimo Redentore&quot;, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: &quot;osmsync:findafountain&quot;, &quot;tags&quot;: &quot;{\&quot;description\&quot;:\&quot;Beside Chiesa del Santissimo Redentore. Venice.\&quot;,\&quot;image\&quot;:\&quot;http://findafountain.org/system/900/original/italy-venice-chiesa-del-santissimo-redentore-outdoor-fountain.jpg\&quot;,\&quot;maintenance\&quot;:\&quot;ok\&quot;,\&quot;source:pkey\&quot;:\&quot;1194\&quot;}&quot;, &quot;timestamp&quot;: 1524744621, &quot;version&quot;: 5}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.325421333312988, 45.432098388671875, 12.325421333312988, 45.432098388671875], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.325421333312988, 45.432098388671875], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;16&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 1504407864, &quot;lat&quot;: 45.432098388671875, &quot;lon&quot;: 12.325421333312988, &quot;name&quot;: &quot;Ponte del Squero&quot;, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: &quot;wetap.org&quot;, &quot;tags&quot;: &quot;{\&quot;bottle\&quot;:\&quot;yes\&quot;,\&quot;image\&quot;:\&quot;https://i.imgur.com/8nlT7vT.jpg\&quot;,\&quot;source:pkey\&quot;:\&quot;1195\&quot;,\&quot;wetap:credit\&quot;:\&quot;ivanve1\&quot;,\&quot;wetap:photo\&quot;:\&quot;http://mdcwetap.atwebpages.com/jpg.php?i=ivanve11376029526916.jpg\&quot;,\&quot;wetap:status\&quot;:\&quot;working\&quot;}&quot;, &quot;timestamp&quot;: 1629808123, &quot;version&quot;: 5}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.316451072692871, 45.428810119628906, 12.316451072692871, 45.428810119628906], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.316451072692871, 45.428810119628906], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;17&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 1504407916, &quot;lat&quot;: 45.428810119628906, &quot;lon&quot;: 12.316451072692871, &quot;name&quot;: &quot;Sacca Fisola Boat Stop&quot;, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: &quot;osmsync:findafountain&quot;, &quot;tags&quot;: &quot;{\&quot;description\&quot;:\&quot;Found near the municipal boat stop\&quot;,\&quot;image\&quot;:\&quot;http://findafountain.org/system/899/original/italy-venice-sacca-fisola-boat-stop-outdoor-fountain.jpg\&quot;,\&quot;maintenance\&quot;:\&quot;ok\&quot;,\&quot;source:pkey\&quot;:\&quot;1193\&quot;}&quot;, &quot;timestamp&quot;: 1598339812, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.345115661621094, 45.43583297729492, 12.345115661621094, 45.43583297729492], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.345115661621094, 45.43583297729492], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;18&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 1608493659, &quot;lat&quot;: 45.43583297729492, &quot;lon&quot;: 12.345115661621094, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;bottle\&quot;:\&quot;yes\&quot;,\&quot;image\&quot;:\&quot;https://i.imgur.com/ZgY2qeH.jpg\&quot;}&quot;, &quot;timestamp&quot;: 1629804729, &quot;version&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.337268829345703, 45.44173812866211, 12.337268829345703, 45.44173812866211], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.337268829345703, 45.44173812866211], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;19&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 1664165995, &quot;lat&quot;: 45.44173812866211, &quot;lon&quot;: 12.337268829345703, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;check_date\&quot;:\&quot;2022-04-12\&quot;}&quot;, &quot;timestamp&quot;: 1649758199, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.350445747375488, 45.452301025390625, 12.350445747375488, 45.452301025390625], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.350445747375488, 45.452301025390625], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;20&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 1781910003, &quot;lat&quot;: 45.452301025390625, &quot;lon&quot;: 12.350445747375488, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1636213960, &quot;version&quot;: 4}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.353745460510254, 45.453121185302734, 12.353745460510254, 45.453121185302734], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.353745460510254, 45.453121185302734], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;21&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 1831361320, &quot;lat&quot;: 45.453121185302734, &quot;lon&quot;: 12.353745460510254, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1342727381, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.300543785095215, 45.26496887207031, 12.300543785095215, 45.26496887207031], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.300543785095215, 45.26496887207031], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;22&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 1836673909, &quot;lat&quot;: 45.26496887207031, &quot;lon&quot;: 12.300543785095215, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1343143625, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.30007266998291, 45.2688102722168, 12.30007266998291, 45.2688102722168], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.30007266998291, 45.2688102722168], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;23&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 1839104977, &quot;lat&quot;: 45.2688102722168, &quot;lon&quot;: 12.30007266998291, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1343332900, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.301260948181152, 45.2752685546875, 12.301260948181152, 45.2752685546875], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.301260948181152, 45.2752685546875], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;24&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 1839134053, &quot;lat&quot;: 45.2752685546875, &quot;lon&quot;: 12.301260948181152, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1343334584, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.300400733947754, 45.265281677246094, 12.300400733947754, 45.265281677246094], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.300400733947754, 45.265281677246094], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;25&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 1839169566, &quot;lat&quot;: 45.265281677246094, &quot;lon&quot;: 12.300400733947754, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1405419144, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.354596138000488, 45.432315826416016, 12.354596138000488, 45.432315826416016], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.354596138000488, 45.432315826416016], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;26&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 1855459557, &quot;lat&quot;: 45.432315826416016, &quot;lon&quot;: 12.354596138000488, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;bottle\&quot;:\&quot;yes\&quot;,\&quot;image\&quot;:\&quot;https://i.imgur.com/oshYGvJ.jpg\&quot;}&quot;, &quot;timestamp&quot;: 1629899041, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.352259635925293, 45.45354461669922, 12.352259635925293, 45.45354461669922], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.352259635925293, 45.45354461669922], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;27&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 1906034623, &quot;lat&quot;: 45.45354461669922, &quot;lon&quot;: 12.352259635925293, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;check_date\&quot;:\&quot;2022-04-13\&quot;}&quot;, &quot;timestamp&quot;: 1649852480, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.353758811950684, 45.45732879638672, 12.353758811950684, 45.45732879638672], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.353758811950684, 45.45732879638672], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;28&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: &quot;disused&quot;, &quot;fountain&quot;: null, &quot;id&quot;: 1907539036, &quot;lat&quot;: 45.45732879638672, &quot;lon&quot;: 12.353758811950684, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;disused:amenity\&quot;:\&quot;drinking_water\&quot;}&quot;, &quot;timestamp&quot;: 1602491047, &quot;version&quot;: 4}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.377220153808594, 45.44240951538086, 12.377220153808594, 45.44240951538086], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.377220153808594, 45.44240951538086], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;29&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 2097985427, &quot;lat&quot;: 45.44240951538086, &quot;lon&quot;: 12.377220153808594, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1357478892, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.358490943908691, 45.433753967285156, 12.358490943908691, 45.433753967285156], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.358490943908691, 45.433753967285156], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;30&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 2250299609, &quot;lat&quot;: 45.433753967285156, &quot;lon&quot;: 12.358490943908691, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: &quot;bing&quot;, &quot;tags&quot;: null, &quot;timestamp&quot;: 1365161043, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.33839225769043, 45.37163162231445, 12.33839225769043, 45.37163162231445], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.33839225769043, 45.37163162231445], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;31&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 2310921367, &quot;lat&quot;: 45.37163162231445, &quot;lon&quot;: 12.33839225769043, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1368957270, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.337464332580566, 45.37190246582031, 12.337464332580566, 45.37190246582031], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.337464332580566, 45.37190246582031], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;32&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 2310921375, &quot;lat&quot;: 45.37190246582031, &quot;lon&quot;: 12.337464332580566, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1368957271, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.276575088500977, 45.46990203857422, 12.276575088500977, 45.46990203857422], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.276575088500977, 45.46990203857422], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;33&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 2357337132, &quot;lat&quot;: 45.46990203857422, &quot;lon&quot;: 12.276575088500977, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;check_date\&quot;:\&quot;2022-03-25\&quot;}&quot;, &quot;timestamp&quot;: 1648230986, &quot;version&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.269854545593262, 45.47657775878906, 12.269854545593262, 45.47657775878906], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.269854545593262, 45.47657775878906], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;34&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 2357337135, &quot;lat&quot;: 45.47657775878906, &quot;lon&quot;: 12.269854545593262, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;check_date\&quot;:\&quot;2022-03-25\&quot;}&quot;, &quot;timestamp&quot;: 1648233788, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.357708930969238, 45.45771026611328, 12.357708930969238, 45.45771026611328], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.357708930969238, 45.45771026611328], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;35&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 2363804168, &quot;lat&quot;: 45.45771026611328, &quot;lon&quot;: 12.357708930969238, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;check_date\&quot;:\&quot;2022-04-13\&quot;}&quot;, &quot;timestamp&quot;: 1649845201, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.373929023742676, 45.41510009765625, 12.373929023742676, 45.41510009765625], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.373929023742676, 45.41510009765625], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;36&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 2425118798, &quot;lat&quot;: 45.41510009765625, &quot;lon&quot;: 12.373929023742676, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: &quot;wetap.org&quot;, &quot;tags&quot;: &quot;{\&quot;description\&quot;:\&quot;Angolo Gran viale e Via Dardanelli\&quot;,\&quot;wetap:credit\&quot;:\&quot;marco lachin\&quot;,\&quot;wetap:photo\&quot;:\&quot;http://mdcwetap.atwebpages.com/jpg.php?i=marco lachin 1376559944251.jpg\&quot;,\&quot;wetap:status\&quot;:\&quot;working\&quot;}&quot;, &quot;timestamp&quot;: 1376980677, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.37708854675293, 45.41529846191406, 12.37708854675293, 45.41529846191406], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.37708854675293, 45.41529846191406], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;37&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 2425118800, &quot;lat&quot;: 45.41529846191406, &quot;lon&quot;: 12.37708854675293, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: &quot;wetap.org&quot;, &quot;tags&quot;: &quot;{\&quot;wetap:credit\&quot;:\&quot;marco lachin\&quot;,\&quot;wetap:photo\&quot;:\&quot;http://mdcwetap.atwebpages.com/jpg.php?i=marco lachin 1376560300074.jpg\&quot;,\&quot;wetap:status\&quot;:\&quot;broken\&quot;}&quot;, &quot;timestamp&quot;: 1376980677, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.371278762817383, 45.41587829589844, 12.371278762817383, 45.41587829589844], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.371278762817383, 45.41587829589844], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;38&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 2425118801, &quot;lat&quot;: 45.41587829589844, &quot;lon&quot;: 12.371278762817383, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: &quot;wetap.org&quot;, &quot;tags&quot;: &quot;{\&quot;description\&quot;:\&quot;Via Ascalona\&quot;,\&quot;wetap:credit\&quot;:\&quot;marco lachin\&quot;,\&quot;wetap:photo\&quot;:\&quot;http://mdcwetap.atwebpages.com/jpg.php?i=marco lachin 1376559586560.jpg\&quot;,\&quot;wetap:status\&quot;:\&quot;working\&quot;}&quot;, &quot;timestamp&quot;: 1376980677, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.331833839416504, 45.43881607055664, 12.331833839416504, 45.43881607055664], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.331833839416504, 45.43881607055664], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;39&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 2425118805, &quot;lat&quot;: 45.43881607055664, &quot;lon&quot;: 12.331833839416504, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: &quot;wetap.org&quot;, &quot;tags&quot;: &quot;{\&quot;description\&quot;:\&quot;Carampane\&quot;,\&quot;wetap:credit\&quot;:\&quot;marco lachin\&quot;,\&quot;wetap:photo\&quot;:\&quot;http://mdcwetap.atwebpages.com/jpg.php?i=marco lachin 1376743736254.jpg\&quot;,\&quot;wetap:status\&quot;:\&quot;inactive\&quot;}&quot;, &quot;timestamp&quot;: 1526660158, &quot;version&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.331498146057129, 45.4390754699707, 12.331498146057129, 45.4390754699707], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.331498146057129, 45.4390754699707], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;40&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 2425118807, &quot;lat&quot;: 45.4390754699707, &quot;lon&quot;: 12.331498146057129, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: &quot;wetap.org&quot;, &quot;tags&quot;: &quot;{\&quot;description\&quot;:\&quot;Carampane\&quot;,\&quot;wetap:credit\&quot;:\&quot;marco lachin\&quot;,\&quot;wetap:status\&quot;:\&quot;working\&quot;}&quot;, &quot;timestamp&quot;: 1376980677, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.334324836730957, 45.439632415771484, 12.334324836730957, 45.439632415771484], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.334324836730957, 45.439632415771484], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;41&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 2425118808, &quot;lat&quot;: 45.439632415771484, &quot;lon&quot;: 12.334324836730957, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: &quot;wetap.org&quot;, &quot;tags&quot;: &quot;{\&quot;description\&quot;:\&quot;Campo della pescaria\&quot;,\&quot;wetap:credit\&quot;:\&quot;marco lachin\&quot;,\&quot;wetap:photo\&quot;:\&quot;http://mdcwetap.atwebpages.com/jpg.php?i=marco lachin 1376548252729.jpg\&quot;,\&quot;wetap:status\&quot;:\&quot;working\&quot;}&quot;, &quot;timestamp&quot;: 1376980677, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.329949378967285, 45.437320709228516, 12.329949378967285, 45.437320709228516], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.329949378967285, 45.437320709228516], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;42&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 2425989866, &quot;lat&quot;: 45.437320709228516, &quot;lon&quot;: 12.329949378967285, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: &quot;wetap.org&quot;, &quot;tags&quot;: &quot;{\&quot;description\&quot;:\&quot;Campo San Polo\&quot;,\&quot;wetap:credit\&quot;:\&quot;marco lachin\&quot;,\&quot;wetap:photo\&quot;:\&quot;http://mdcwetap.atwebpages.com/jpg.php?i=marco lachin 1376842986045.jpg\&quot;,\&quot;wetap:status\&quot;:\&quot;working\&quot;,\&quot;wheelchair\&quot;:\&quot;yes\&quot;}&quot;, &quot;timestamp&quot;: 1573307467, &quot;version&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.368412971496582, 45.40707015991211, 12.368412971496582, 45.40707015991211], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.368412971496582, 45.40707015991211], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;43&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 2477391527, &quot;lat&quot;: 45.40707015991211, &quot;lon&quot;: 12.368412971496582, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: &quot;wetap.org&quot;, &quot;tags&quot;: &quot;{\&quot;description\&quot;:\&quot;Parco Casin\\u00F2  estate 7 - 21 inverno 8 - 18.30\&quot;,\&quot;wetap:credit\&quot;:\&quot;ivanve1\&quot;,\&quot;wetap:photo\&quot;:\&quot;http://mdcwetap.atwebpages.com/jpg.php?i=ivanve11378140904102.jpg\&quot;,\&quot;wetap:status\&quot;:\&quot;working\&quot;}&quot;, &quot;timestamp&quot;: 1380531308, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.326849937438965, 45.42512130737305, 12.326849937438965, 45.42512130737305], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.326849937438965, 45.42512130737305], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;44&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 2477391546, &quot;lat&quot;: 45.42512130737305, &quot;lon&quot;: 12.326849937438965, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: &quot;wetap.org&quot;, &quot;tags&quot;: &quot;{\&quot;description\&quot;:\&quot;Campiello Ferrando\&quot;,\&quot;wetap:credit\&quot;:\&quot;ivanve1\&quot;,\&quot;wetap:status\&quot;:\&quot;working\&quot;}&quot;, &quot;timestamp&quot;: 1380531308, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.32949161529541, 45.42546463012695, 12.32949161529541, 45.42546463012695], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.32949161529541, 45.42546463012695], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;45&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 2477391552, &quot;lat&quot;: 45.42546463012695, &quot;lon&quot;: 12.32949161529541, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: &quot;wetap.org&quot;, &quot;tags&quot;: &quot;{\&quot;description\&quot;:\&quot;Ponte Lungo Fondamenta san Giacomo\&quot;,\&quot;wetap:credit\&quot;:\&quot;ivanve1\&quot;,\&quot;wetap:photo\&quot;:\&quot;http://mdcwetap.atwebpages.com/jpg.php?i=ivanve11379748014222.jpg\&quot;,\&quot;wetap:status\&quot;:\&quot;working\&quot;}&quot;, &quot;timestamp&quot;: 1380531308, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.331195831298828, 45.429752349853516, 12.331195831298828, 45.429752349853516], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.331195831298828, 45.429752349853516], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;46&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 2477391561, &quot;lat&quot;: 45.429752349853516, &quot;lon&quot;: 12.331195831298828, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: &quot;wetap.org&quot;, &quot;tags&quot;: &quot;{\&quot;description\&quot;:\&quot;Corte del Sabbion\&quot;,\&quot;wetap:credit\&quot;:\&quot;ivanve1\&quot;,\&quot;wetap:photo\&quot;:\&quot;http://mdcwetap.atwebpages.com/jpg.php?i=ivanve11378966714665.jpg\&quot;,\&quot;wetap:status\&quot;:\&quot;working\&quot;}&quot;, &quot;timestamp&quot;: 1380531308, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.32657527923584, 45.44706344604492, 12.32657527923584, 45.44706344604492], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.32657527923584, 45.44706344604492], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;47&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 2477395748, &quot;lat&quot;: 45.44706344604492, &quot;lon&quot;: 12.32657527923584, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: &quot;wetap.org&quot;, &quot;tags&quot;: &quot;{\&quot;description\&quot;:\&quot;calle dei Riformati\&quot;,\&quot;wetap:credit\&quot;:\&quot;ivanve1\&quot;,\&quot;wetap:photo\&quot;:\&quot;http://mdcwetap.atwebpages.com/jpg.php?i=ivanve11378481004570.jpg\&quot;,\&quot;wetap:status\&quot;:\&quot;working\&quot;}&quot;, &quot;timestamp&quot;: 1492939819, &quot;version&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.324751853942871, 45.446510314941406, 12.324751853942871, 45.446510314941406], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.324751853942871, 45.446510314941406], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;48&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 2482252834, &quot;lat&quot;: 45.446510314941406, &quot;lon&quot;: 12.324751853942871, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1380888825, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.301360130310059, 45.28010177612305, 12.301360130310059, 45.28010177612305], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.301360130310059, 45.28010177612305], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;49&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 2933044410, &quot;lat&quot;: 45.28010177612305, &quot;lon&quot;: 12.301360130310059, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1403710920, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.350768089294434, 45.43321990966797, 12.350768089294434, 45.43321990966797], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.350768089294434, 45.43321990966797], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;50&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 3002742587, &quot;lat&quot;: 45.43321990966797, &quot;lon&quot;: 12.350768089294434, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: &quot;survey;Bing&quot;, &quot;tags&quot;: &quot;{\&quot;source:date\&quot;:\&quot;2017\&quot;}&quot;, &quot;timestamp&quot;: 1495747744, &quot;version&quot;: 4}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.351300239562988, 45.457481384277344, 12.351300239562988, 45.457481384277344], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.351300239562988, 45.457481384277344], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;51&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 3002745973, &quot;lat&quot;: 45.457481384277344, &quot;lon&quot;: 12.351300239562988, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: &quot;survey&quot;, &quot;tags&quot;: null, &quot;timestamp&quot;: 1407420232, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.354784965515137, 45.45626449584961, 12.354784965515137, 45.45626449584961], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.354784965515137, 45.45626449584961], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;52&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 3016775612, &quot;lat&quot;: 45.45626449584961, &quot;lon&quot;: 12.354784965515137, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;check_date\&quot;:\&quot;2022-04-13\&quot;}&quot;, &quot;timestamp&quot;: 1649847473, &quot;version&quot;: 4}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.351727485656738, 45.453857421875, 12.351727485656738, 45.453857421875], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.351727485656738, 45.453857421875], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;53&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 3016776133, &quot;lat&quot;: 45.453857421875, &quot;lon&quot;: 12.351727485656738, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1408037776, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.312260627746582, 45.31196212768555, 12.312260627746582, 45.31196212768555], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.312260627746582, 45.31196212768555], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;54&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 3027170019, &quot;lat&quot;: 45.31196212768555, &quot;lon&quot;: 12.312260627746582, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1408447265, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.314769744873047, 45.316829681396484, 12.314769744873047, 45.316829681396484], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.314769744873047, 45.316829681396484], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;55&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 3027206435, &quot;lat&quot;: 45.316829681396484, &quot;lon&quot;: 12.314769744873047, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1408448706, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.41639232635498, 45.4871826171875, 12.41639232635498, 45.4871826171875], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.41639232635498, 45.4871826171875], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;56&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 3089667910, &quot;lat&quot;: 45.4871826171875, &quot;lon&quot;: 12.41639232635498, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;check_date\&quot;:\&quot;2021-11-07\&quot;}&quot;, &quot;timestamp&quot;: 1636303139, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.322574615478516, 45.43584060668945, 12.322574615478516, 45.43584060668945], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.322574615478516, 45.43584060668945], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;57&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 3403782914, &quot;lat&quot;: 45.43584060668945, &quot;lon&quot;: 12.322574615478516, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;disused\&quot;:\&quot;yes\&quot;}&quot;, &quot;timestamp&quot;: 1426595489, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.378656387329102, 45.44013214111328, 12.378656387329102, 45.44013214111328], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.378656387329102, 45.44013214111328], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;58&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 3796613258, &quot;lat&quot;: 45.44013214111328, &quot;lon&quot;: 12.378656387329102, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1445414936, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.415109634399414, 45.485591888427734, 12.415109634399414, 45.485591888427734], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.415109634399414, 45.485591888427734], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;59&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 3851371368, &quot;lat&quot;: 45.485591888427734, &quot;lon&quot;: 12.415109634399414, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;fee\&quot;:\&quot;no\&quot;}&quot;, &quot;timestamp&quot;: 1565207044, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.415836334228516, 45.48619079589844, 12.415836334228516, 45.48619079589844], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.415836334228516, 45.48619079589844], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;60&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: &quot;nasone&quot;, &quot;id&quot;: 3851371369, &quot;lat&quot;: 45.48619079589844, &quot;lon&quot;: 12.415836334228516, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;access\&quot;:\&quot;yes\&quot;,\&quot;bottle\&quot;:\&quot;yes\&quot;,\&quot;indoor\&quot;:\&quot;no\&quot;,\&quot;level\&quot;:\&quot;0\&quot;}&quot;, &quot;timestamp&quot;: 1636301090, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.319568634033203, 45.436405181884766, 12.319568634033203, 45.436405181884766], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.319568634033203, 45.436405181884766], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;61&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 3877321052, &quot;lat&quot;: 45.436405181884766, &quot;lon&quot;: 12.319568634033203, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;wheelchair\&quot;:\&quot;yes\&quot;}&quot;, &quot;timestamp&quot;: 1573292600, &quot;version&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.417766571044922, 45.497894287109375, 12.417766571044922, 45.497894287109375], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.417766571044922, 45.497894287109375], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;62&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 4082843689, &quot;lat&quot;: 45.497894287109375, &quot;lon&quot;: 12.417766571044922, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1494083721, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.317541122436523, 45.32099914550781, 12.317541122436523, 45.32099914550781], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.317541122436523, 45.32099914550781], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;63&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 4127652890, &quot;lat&quot;: 45.32099914550781, &quot;lon&quot;: 12.317541122436523, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1460888251, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.233325958251953, 45.48428726196289, 12.233325958251953, 45.48428726196289], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.233325958251953, 45.48428726196289], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;64&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 4274305398, &quot;lat&quot;: 45.48428726196289, &quot;lon&quot;: 12.233325958251953, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1467323772, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.334649085998535, 45.43962478637695, 12.334649085998535, 45.43962478637695], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.334649085998535, 45.43962478637695], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;65&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 4293258189, &quot;lat&quot;: 45.43962478637695, &quot;lon&quot;: 12.334649085998535, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1468078529, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.31540584564209, 45.431365966796875, 12.31540584564209, 45.431365966796875], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.31540584564209, 45.431365966796875], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;66&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 4394685695, &quot;lat&quot;: 45.431365966796875, &quot;lon&quot;: 12.31540584564209, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1473512636, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.412562370300293, 45.49510192871094, 12.412562370300293, 45.49510192871094], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.412562370300293, 45.49510192871094], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;67&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 4438827789, &quot;lat&quot;: 45.49510192871094, &quot;lon&quot;: 12.412562370300293, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1494072091, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.417943000793457, 45.48419189453125, 12.417943000793457, 45.48419189453125], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.417943000793457, 45.48419189453125], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;68&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: &quot;nasone&quot;, &quot;id&quot;: 4438828089, &quot;lat&quot;: 45.48419189453125, &quot;lon&quot;: 12.417943000793457, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;access\&quot;:\&quot;yes\&quot;,\&quot;bottle\&quot;:\&quot;yes\&quot;,\&quot;indoor\&quot;:\&quot;no\&quot;,\&quot;level\&quot;:\&quot;0\&quot;}&quot;, &quot;timestamp&quot;: 1636301538, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.341221809387207, 45.441017150878906, 12.341221809387207, 45.441017150878906], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.341221809387207, 45.441017150878906], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;69&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 4449378234, &quot;lat&quot;: 45.441017150878906, &quot;lon&quot;: 12.341221809387207, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1476546739, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.260969161987305, 45.496578216552734, 12.260969161987305, 45.496578216552734], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.260969161987305, 45.496578216552734], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;70&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 4450118842, &quot;lat&quot;: 45.496578216552734, &quot;lon&quot;: 12.260969161987305, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;wheelchair\&quot;:\&quot;yes\&quot;}&quot;, &quot;timestamp&quot;: 1638260161, &quot;version&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.3175048828125, 45.319671630859375, 12.3175048828125, 45.319671630859375], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.3175048828125, 45.319671630859375], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;71&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 4525147730, &quot;lat&quot;: 45.319671630859375, &quot;lon&quot;: 12.3175048828125, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1480272300, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.299839973449707, 45.269805908203125, 12.299839973449707, 45.269805908203125], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.299839973449707, 45.269805908203125], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;72&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 4525191580, &quot;lat&quot;: 45.269805908203125, &quot;lon&quot;: 12.299839973449707, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1480274562, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.40552043914795, 45.48402404785156, 12.40552043914795, 45.48402404785156], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.40552043914795, 45.48402404785156], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;73&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 4853863909, &quot;lat&quot;: 45.48402404785156, &quot;lon&quot;: 12.40552043914795, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1521662118, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.240800857543945, 45.49419403076172, 12.240800857543945, 45.49419403076172], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.240800857543945, 45.49419403076172], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;74&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 4875501521, &quot;lat&quot;: 45.49419403076172, &quot;lon&quot;: 12.240800857543945, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;wheelchair\&quot;:\&quot;yes\&quot;}&quot;, &quot;timestamp&quot;: 1637849349, &quot;version&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.352928161621094, 45.45516586303711, 12.352928161621094, 45.45516586303711], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.352928161621094, 45.45516586303711], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;75&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: &quot;Via Mario Tabanelli&quot;, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 4945409405, &quot;lat&quot;: 45.45516586303711, &quot;lon&quot;: 12.352928161621094, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1525208990, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.412466049194336, 45.48917770385742, 12.412466049194336, 45.48917770385742], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.412466049194336, 45.48917770385742], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;76&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 4945789685, &quot;lat&quot;: 45.48917770385742, &quot;lon&quot;: 12.412466049194336, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1498922273, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.416308403015137, 45.48545837402344, 12.416308403015137, 45.48545837402344], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.416308403015137, 45.48545837402344], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;77&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 4945828330, &quot;lat&quot;: 45.48545837402344, &quot;lon&quot;: 12.416308403015137, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1498924115, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.373708724975586, 45.41535949707031, 12.373708724975586, 45.41535949707031], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.373708724975586, 45.41535949707031], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;78&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 4955472977, &quot;lat&quot;: 45.41535949707031, &quot;lon&quot;: 12.373708724975586, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1550404572, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.35772705078125, 45.431434631347656, 12.35772705078125, 45.431434631347656], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.35772705078125, 45.431434631347656], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;79&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 4958498423, &quot;lat&quot;: 45.431434631347656, &quot;lon&quot;: 12.35772705078125, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1517357301, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.357340812683105, 45.458438873291016, 12.357340812683105, 45.458438873291016], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.357340812683105, 45.458438873291016], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;80&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 4958933616, &quot;lat&quot;: 45.458438873291016, &quot;lon&quot;: 12.357340812683105, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1499545881, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.226701736450195, 45.488670349121094, 12.226701736450195, 45.488670349121094], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.226701736450195, 45.488670349121094], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;81&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5043824823, &quot;lat&quot;: 45.488670349121094, &quot;lon&quot;: 12.226701736450195, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;wheelchair\&quot;:\&quot;yes\&quot;}&quot;, &quot;timestamp&quot;: 1637848574, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.220871925354004, 45.480262756347656, 12.220871925354004, 45.480262756347656], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.220871925354004, 45.480262756347656], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;82&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5045475382, &quot;lat&quot;: 45.480262756347656, &quot;lon&quot;: 12.220871925354004, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1503182915, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.243429183959961, 45.49501419067383, 12.243429183959961, 45.49501419067383], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.243429183959961, 45.49501419067383], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;83&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5086068997, &quot;lat&quot;: 45.49501419067383, &quot;lon&quot;: 12.243429183959961, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1504790703, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.343019485473633, 45.43452453613281, 12.343019485473633, 45.43452453613281], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.343019485473633, 45.43452453613281], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;84&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5360338819, &quot;lat&quot;: 45.43452453613281, &quot;lon&quot;: 12.343019485473633, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1516863440, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.328018188476562, 45.44035720825195, 12.328018188476562, 45.44035720825195], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.328018188476562, 45.44035720825195], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;85&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5360645773, &quot;lat&quot;: 45.44035720825195, &quot;lon&quot;: 12.328018188476562, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1517335751, &quot;version&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.33393669128418, 45.44140625, 12.33393669128418, 45.44140625], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.33393669128418, 45.44140625], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;86&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5360856760, &quot;lat&quot;: 45.44140625, &quot;lon&quot;: 12.33393669128418, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;wheelchair\&quot;:\&quot;yes\&quot;}&quot;, &quot;timestamp&quot;: 1641118328, &quot;version&quot;: 5}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.323678970336914, 45.44319152832031, 12.323678970336914, 45.44319152832031], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.323678970336914, 45.44319152832031], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;87&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5360885480, &quot;lat&quot;: 45.44319152832031, &quot;lon&quot;: 12.323678970336914, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1516881918, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.338927268981934, 45.43300247192383, 12.338927268981934, 45.43300247192383], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.338927268981934, 45.43300247192383], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;88&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5361276740, &quot;lat&quot;: 45.43300247192383, &quot;lon&quot;: 12.338927268981934, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;bottle\&quot;:\&quot;yes\&quot;,\&quot;image\&quot;:\&quot;https://i.imgur.com/w8NIK57.jpg\&quot;}&quot;, &quot;timestamp&quot;: 1629801726, &quot;version&quot;: 4}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.34981918334961, 45.432899475097656, 12.34981918334961, 45.432899475097656], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.34981918334961, 45.432899475097656], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;89&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5361308215, &quot;lat&quot;: 45.432899475097656, &quot;lon&quot;: 12.34981918334961, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;wheelchair\&quot;:\&quot;yes\&quot;}&quot;, &quot;timestamp&quot;: 1553606645, &quot;version&quot;: 4}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.352853775024414, 45.4315299987793, 12.352853775024414, 45.4315299987793], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.352853775024414, 45.4315299987793], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;90&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5361308220, &quot;lat&quot;: 45.4315299987793, &quot;lon&quot;: 12.352853775024414, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1516893650, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.356432914733887, 45.429222106933594, 12.356432914733887, 45.429222106933594], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.356432914733887, 45.429222106933594], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;91&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5361310224, &quot;lat&quot;: 45.429222106933594, &quot;lon&quot;: 12.356432914733887, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1516893650, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.359834671020508, 45.430091857910156, 12.359834671020508, 45.430091857910156], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.359834671020508, 45.430091857910156], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;92&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5361310225, &quot;lat&quot;: 45.430091857910156, &quot;lon&quot;: 12.359834671020508, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1516893650, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.358776092529297, 45.43070602416992, 12.358776092529297, 45.43070602416992], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.358776092529297, 45.43070602416992], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;93&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5363452728, &quot;lat&quot;: 45.43070602416992, &quot;lon&quot;: 12.358776092529297, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1616340780, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.356585502624512, 45.433345794677734, 12.356585502624512, 45.433345794677734], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.356585502624512, 45.433345794677734], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;94&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5363572346, &quot;lat&quot;: 45.433345794677734, &quot;lon&quot;: 12.356585502624512, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1516975323, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.349900245666504, 45.4340705871582, 12.349900245666504, 45.4340705871582], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.349900245666504, 45.4340705871582], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;95&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5363720454, &quot;lat&quot;: 45.4340705871582, &quot;lon&quot;: 12.349900245666504, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1516979000, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.342719078063965, 45.43888473510742, 12.342719078063965, 45.43888473510742], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.342719078063965, 45.43888473510742], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;96&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5364430126, &quot;lat&quot;: 45.43888473510742, &quot;lon&quot;: 12.342719078063965, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1516998584, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.328636169433594, 45.44141387939453, 12.328636169433594, 45.44141387939453], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.328636169433594, 45.44141387939453], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;97&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5364525644, &quot;lat&quot;: 45.44141387939453, &quot;lon&quot;: 12.328636169433594, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1528130156, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.333462715148926, 45.44424057006836, 12.333462715148926, 45.44424057006836], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.333462715148926, 45.44424057006836], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;98&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5364657882, &quot;lat&quot;: 45.44424057006836, &quot;lon&quot;: 12.333462715148926, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1517004196, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.332267761230469, 45.430397033691406, 12.332267761230469, 45.430397033691406], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.332267761230469, 45.430397033691406], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;99&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5365979872, &quot;lat&quot;: 45.430397033691406, &quot;lon&quot;: 12.332267761230469, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;bottle\&quot;:\&quot;yes\&quot;,\&quot;image\&quot;:\&quot;https://i.imgur.com/NPA5FLB.jpg\&quot;}&quot;, &quot;timestamp&quot;: 1629887863, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.320343971252441, 45.430763244628906, 12.320343971252441, 45.430763244628906], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.320343971252441, 45.430763244628906], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;100&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5366141204, &quot;lat&quot;: 45.430763244628906, &quot;lon&quot;: 12.320343971252441, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1517067452, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.333958625793457, 45.42897415161133, 12.333958625793457, 45.42897415161133], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.333958625793457, 45.42897415161133], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;101&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5366190365, &quot;lat&quot;: 45.42897415161133, &quot;lon&quot;: 12.333958625793457, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;bottle\&quot;:\&quot;yes\&quot;,\&quot;image\&quot;:\&quot;https://i.imgur.com/1A7pNDy.jpg\&quot;,\&quot;operational_status\&quot;:\&quot;broken\&quot;}&quot;, &quot;timestamp&quot;: 1629888993, &quot;version&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.33200454711914, 45.428802490234375, 12.33200454711914, 45.428802490234375], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.33200454711914, 45.428802490234375], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;102&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5366197830, &quot;lat&quot;: 45.428802490234375, &quot;lon&quot;: 12.33200454711914, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1517069276, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.33350944519043, 45.4300651550293, 12.33350944519043, 45.4300651550293], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.33350944519043, 45.4300651550293], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;103&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5366207016, &quot;lat&quot;: 45.4300651550293, &quot;lon&quot;: 12.33350944519043, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1517069771, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.348867416381836, 45.437355041503906, 12.348867416381836, 45.437355041503906], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.348867416381836, 45.437355041503906], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;104&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5366748175, &quot;lat&quot;: 45.437355041503906, &quot;lon&quot;: 12.348867416381836, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1518249520, &quot;version&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.34119701385498, 45.437252044677734, 12.34119701385498, 45.437252044677734], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.34119701385498, 45.437252044677734], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;105&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5366899349, &quot;lat&quot;: 45.437252044677734, &quot;lon&quot;: 12.34119701385498, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1517090778, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.333612442016602, 45.43946838378906, 12.333612442016602, 45.43946838378906], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.333612442016602, 45.43946838378906], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;106&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5366925033, &quot;lat&quot;: 45.43946838378906, &quot;lon&quot;: 12.333612442016602, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1526399267, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.323623657226562, 45.43416213989258, 12.323623657226562, 45.43416213989258], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.323623657226562, 45.43416213989258], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;107&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5368105261, &quot;lat&quot;: 45.43416213989258, &quot;lon&quot;: 12.323623657226562, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;bottle\&quot;:\&quot;yes\&quot;,\&quot;image\&quot;:\&quot;https://i.imgur.com/tuSNV91.jpg\&quot;,\&quot;wheelchair\&quot;:\&quot;yes\&quot;}&quot;, &quot;timestamp&quot;: 1629884791, &quot;version&quot;: 4}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.326942443847656, 45.44535827636719, 12.326942443847656, 45.44535827636719], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.326942443847656, 45.44535827636719], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;108&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5368446463, &quot;lat&quot;: 45.44535827636719, &quot;lon&quot;: 12.326942443847656, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1546264682, &quot;version&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.330822944641113, 45.44661331176758, 12.330822944641113, 45.44661331176758], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.330822944641113, 45.44661331176758], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;109&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5368599309, &quot;lat&quot;: 45.44661331176758, &quot;lon&quot;: 12.330822944641113, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1518251068, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.327469825744629, 45.448307037353516, 12.327469825744629, 45.448307037353516], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.327469825744629, 45.448307037353516], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;110&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5368630486, &quot;lat&quot;: 45.448307037353516, &quot;lon&quot;: 12.327469825744629, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1517165133, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.328614234924316, 45.44727325439453, 12.328614234924316, 45.44727325439453], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.328614234924316, 45.44727325439453], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;111&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5368656867, &quot;lat&quot;: 45.44727325439453, &quot;lon&quot;: 12.328614234924316, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1517165879, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.324105262756348, 45.44459533691406, 12.324105262756348, 45.44459533691406], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.324105262756348, 45.44459533691406], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;112&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5368795232, &quot;lat&quot;: 45.44459533691406, &quot;lon&quot;: 12.324105262756348, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1517171251, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.337491989135742, 45.438106536865234, 12.337491989135742, 45.438106536865234], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.337491989135742, 45.438106536865234], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;113&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5370747682, &quot;lat&quot;: 45.438106536865234, &quot;lon&quot;: 12.337491989135742, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1523638274, &quot;version&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.347384452819824, 45.43498992919922, 12.347384452819824, 45.43498992919922], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.347384452819824, 45.43498992919922], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;114&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5371006271, &quot;lat&quot;: 45.43498992919922, &quot;lon&quot;: 12.347384452819824, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1541015577, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.347265243530273, 45.43634033203125, 12.347265243530273, 45.43634033203125], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.347265243530273, 45.43634033203125], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;115&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5371097029, &quot;lat&quot;: 45.43634033203125, &quot;lon&quot;: 12.347265243530273, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1517258548, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.348601341247559, 45.43614196777344, 12.348601341247559, 45.43614196777344], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.348601341247559, 45.43614196777344], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;116&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5371099512, &quot;lat&quot;: 45.43614196777344, &quot;lon&quot;: 12.348601341247559, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1517258775, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.34924030303955, 45.43602752685547, 12.34924030303955, 45.43602752685547], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.34924030303955, 45.43602752685547], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;117&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5371110464, &quot;lat&quot;: 45.43602752685547, &quot;lon&quot;: 12.34924030303955, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1517258914, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.347545623779297, 45.43804168701172, 12.347545623779297, 45.43804168701172], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.347545623779297, 45.43804168701172], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;118&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5371160660, &quot;lat&quot;: 45.43804168701172, &quot;lon&quot;: 12.347545623779297, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1517260076, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.338287353515625, 45.44278335571289, 12.338287353515625, 45.44278335571289], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.338287353515625, 45.44278335571289], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;119&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5371367833, &quot;lat&quot;: 45.44278335571289, &quot;lon&quot;: 12.338287353515625, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1521057141, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.32673454284668, 45.44096755981445, 12.32673454284668, 45.44096755981445], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.32673454284668, 45.44096755981445], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;120&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5373002773, &quot;lat&quot;: 45.44096755981445, &quot;lon&quot;: 12.32673454284668, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;bottle\&quot;:\&quot;yes\&quot;}&quot;, &quot;timestamp&quot;: 1629829052, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.327115058898926, 45.44036865234375, 12.327115058898926, 45.44036865234375], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.327115058898926, 45.44036865234375], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;121&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5373032015, &quot;lat&quot;: 45.44036865234375, &quot;lon&quot;: 12.327115058898926, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1517335750, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.32766056060791, 45.43620300292969, 12.32766056060791, 45.43620300292969], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.32766056060791, 45.43620300292969], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;122&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5373181562, &quot;lat&quot;: 45.43620300292969, &quot;lon&quot;: 12.32766056060791, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1644839626, &quot;version&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.357542037963867, 45.43302536010742, 12.357542037963867, 45.43302536010742], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.357542037963867, 45.43302536010742], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;123&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5373746086, &quot;lat&quot;: 45.43302536010742, &quot;lon&quot;: 12.357542037963867, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1517357789, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.33079719543457, 45.43505096435547, 12.33079719543457, 45.43505096435547], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.33079719543457, 45.43505096435547], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;124&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5379379521, &quot;lat&quot;: 45.43505096435547, &quot;lon&quot;: 12.33079719543457, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1517526845, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.338836669921875, 45.43684768676758, 12.338836669921875, 45.43684768676758], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.338836669921875, 45.43684768676758], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;125&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5550675264, &quot;lat&quot;: 45.43684768676758, &quot;lon&quot;: 12.338836669921875, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1523638273, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.409588813781738, 45.458839416503906, 12.409588813781738, 45.458839416503906], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.409588813781738, 45.458839416503906], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;126&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5554577322, &quot;lat&quot;: 45.458839416503906, &quot;lon&quot;: 12.409588813781738, &quot;name&quot;: &quot;Acqua potabile&quot;, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1523800566, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.388650894165039, 45.45452117919922, 12.388650894165039, 45.45452117919922], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.388650894165039, 45.45452117919922], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;127&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5554615421, &quot;lat&quot;: 45.45452117919922, &quot;lon&quot;: 12.388650894165039, &quot;name&quot;: &quot;Acqua potabile&quot;, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1523800567, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.286197662353516, 45.522212982177734, 12.286197662353516, 45.522212982177734], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.286197662353516, 45.522212982177734], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;128&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5557875231, &quot;lat&quot;: 45.522212982177734, &quot;lon&quot;: 12.286197662353516, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1523954440, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.348111152648926, 45.43727493286133, 12.348111152648926, 45.43727493286133], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.348111152648926, 45.43727493286133], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;129&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5564763689, &quot;lat&quot;: 45.43727493286133, &quot;lon&quot;: 12.348111152648926, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1524158452, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.27066421508789, 45.47541046142578, 12.27066421508789, 45.47541046142578], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.27066421508789, 45.47541046142578], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;130&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5584239622, &quot;lat&quot;: 45.47541046142578, &quot;lon&quot;: 12.27066421508789, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;check_date\&quot;:\&quot;2022-03-25\&quot;}&quot;, &quot;timestamp&quot;: 1648233775, &quot;version&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.317228317260742, 45.42737579345703, 12.317228317260742, 45.42737579345703], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.317228317260742, 45.42737579345703], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;131&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5589338398, &quot;lat&quot;: 45.42737579345703, &quot;lon&quot;: 12.317228317260742, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1525127184, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.342047691345215, 45.436126708984375, 12.342047691345215, 45.436126708984375], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.342047691345215, 45.436126708984375], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;132&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5600498646, &quot;lat&quot;: 45.436126708984375, &quot;lon&quot;: 12.342047691345215, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;disused\&quot;:\&quot;yes\&quot;,\&quot;survey:date\&quot;:\&quot;06-05-2018\&quot;}&quot;, &quot;timestamp&quot;: 1525626303, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.33481216430664, 45.43840026855469, 12.33481216430664, 45.43840026855469], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.33481216430664, 45.43840026855469], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;133&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5627052251, &quot;lat&quot;: 45.43840026855469, &quot;lon&quot;: 12.33481216430664, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1526660158, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.24185848236084, 45.493072509765625, 12.24185848236084, 45.493072509765625], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.24185848236084, 45.493072509765625], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;134&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5627333421, &quot;lat&quot;: 45.493072509765625, &quot;lon&quot;: 12.24185848236084, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1526670420, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.332216262817383, 45.43549346923828, 12.332216262817383, 45.43549346923828], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.332216262817383, 45.43549346923828], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;135&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5719718221, &quot;lat&quot;: 45.43549346923828, &quot;lon&quot;: 12.332216262817383, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1530003544, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.332340240478516, 45.43538284301758, 12.332340240478516, 45.43538284301758], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.332340240478516, 45.43538284301758], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;136&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5720219921, &quot;lat&quot;: 45.43538284301758, &quot;lon&quot;: 12.332340240478516, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1530013945, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.32911491394043, 45.43366241455078, 12.32911491394043, 45.43366241455078], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.32911491394043, 45.43366241455078], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;137&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5720884953, &quot;lat&quot;: 45.43366241455078, &quot;lon&quot;: 12.32911491394043, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1530033638, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.323930740356445, 45.43595504760742, 12.323930740356445, 45.43595504760742], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.323930740356445, 45.43595504760742], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;138&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5739473234, &quot;lat&quot;: 45.43595504760742, &quot;lon&quot;: 12.323930740356445, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;wheelchair\&quot;:\&quot;yes\&quot;}&quot;, &quot;timestamp&quot;: 1573297567, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.324821472167969, 45.44314193725586, 12.324821472167969, 45.44314193725586], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.324821472167969, 45.44314193725586], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;139&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5805275904, &quot;lat&quot;: 45.44314193725586, &quot;lon&quot;: 12.324821472167969, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;wheelchair\&quot;:\&quot;yes\&quot;}&quot;, &quot;timestamp&quot;: 1573314513, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.321598052978516, 45.43787384033203, 12.321598052978516, 45.43787384033203], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.321598052978516, 45.43787384033203], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;140&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5809868060, &quot;lat&quot;: 45.43787384033203, &quot;lon&quot;: 12.321598052978516, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1533453940, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.325654029846191, 45.42578125, 12.325654029846191, 45.42578125], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.325654029846191, 45.42578125], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;141&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5835161805, &quot;lat&quot;: 45.42578125, &quot;lon&quot;: 12.325654029846191, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1534368503, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.323057174682617, 45.425899505615234, 12.323057174682617, 45.425899505615234], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.323057174682617, 45.425899505615234], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;142&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 5842459402, &quot;lat&quot;: 45.425899505615234, &quot;lon&quot;: 12.323057174682617, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1537467289, &quot;version&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.205710411071777, 45.4362678527832, 12.205710411071777, 45.4362678527832], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.205710411071777, 45.4362678527832], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;143&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 6079832708, &quot;lat&quot;: 45.4362678527832, &quot;lon&quot;: 12.205710411071777, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1542806372, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.273904800415039, 45.473388671875, 12.273904800415039, 45.473388671875], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.273904800415039, 45.473388671875], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;144&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 6181389781, &quot;lat&quot;: 45.473388671875, &quot;lon&quot;: 12.273904800415039, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;check_date\&quot;:\&quot;2022-03-25\&quot;}&quot;, &quot;timestamp&quot;: 1648231859, &quot;version&quot;: 4}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.281251907348633, 45.474327087402344, 12.281251907348633, 45.474327087402344], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.281251907348633, 45.474327087402344], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;145&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 6181389782, &quot;lat&quot;: 45.474327087402344, &quot;lon&quot;: 12.281251907348633, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;check_date\&quot;:\&quot;2022-03-25\&quot;}&quot;, &quot;timestamp&quot;: 1648231427, &quot;version&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.278166770935059, 45.47096633911133, 12.278166770935059, 45.47096633911133], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.278166770935059, 45.47096633911133], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;146&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 6181389783, &quot;lat&quot;: 45.47096633911133, &quot;lon&quot;: 12.278166770935059, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;check_date\&quot;:\&quot;2022-03-25\&quot;}&quot;, &quot;timestamp&quot;: 1648231126, &quot;version&quot;: 4}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.27459716796875, 45.47243881225586, 12.27459716796875, 45.47243881225586], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.27459716796875, 45.47243881225586], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;147&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 6181389784, &quot;lat&quot;: 45.47243881225586, &quot;lon&quot;: 12.27459716796875, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;check_date\&quot;:\&quot;2022-03-25\&quot;}&quot;, &quot;timestamp&quot;: 1648230909, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.271014213562012, 45.47574996948242, 12.271014213562012, 45.47574996948242], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.271014213562012, 45.47574996948242], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;148&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 6181390685, &quot;lat&quot;: 45.47574996948242, &quot;lon&quot;: 12.271014213562012, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;check_date\&quot;:\&quot;2022-03-25\&quot;}&quot;, &quot;timestamp&quot;: 1648233780, &quot;version&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.279763221740723, 45.470211029052734, 12.279763221740723, 45.470211029052734], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.279763221740723, 45.470211029052734], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;149&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 6181390686, &quot;lat&quot;: 45.470211029052734, &quot;lon&quot;: 12.279763221740723, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;check_date\&quot;:\&quot;2022-03-25\&quot;}&quot;, &quot;timestamp&quot;: 1648231128, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.272543907165527, 45.474586486816406, 12.272543907165527, 45.474586486816406], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.272543907165527, 45.474586486816406], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;150&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 6181390687, &quot;lat&quot;: 45.474586486816406, &quot;lon&quot;: 12.272543907165527, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;check_date\&quot;:\&quot;2022-03-25\&quot;}&quot;, &quot;timestamp&quot;: 1648230754, &quot;version&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.273628234863281, 45.472469329833984, 12.273628234863281, 45.472469329833984], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.273628234863281, 45.472469329833984], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;151&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 6181558815, &quot;lat&quot;: 45.472469329833984, &quot;lon&quot;: 12.273628234863281, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;check_date\&quot;:\&quot;2022-03-25\&quot;}&quot;, &quot;timestamp&quot;: 1648230834, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.280084609985352, 45.472530364990234, 12.280084609985352, 45.472530364990234], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.280084609985352, 45.472530364990234], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;152&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 6183226119, &quot;lat&quot;: 45.472530364990234, &quot;lon&quot;: 12.280084609985352, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;check_date\&quot;:\&quot;2022-03-25\&quot;}&quot;, &quot;timestamp&quot;: 1648231424, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.289546966552734, 45.52402114868164, 12.289546966552734, 45.52402114868164], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.289546966552734, 45.52402114868164], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;153&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 6198609870, &quot;lat&quot;: 45.52402114868164, &quot;lon&quot;: 12.289546966552734, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1547241563, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.215906143188477, 45.491085052490234, 12.215906143188477, 45.491085052490234], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.215906143188477, 45.491085052490234], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;154&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 6203850506, &quot;lat&quot;: 45.491085052490234, &quot;lon&quot;: 12.215906143188477, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1547398372, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.226654052734375, 45.502601623535156, 12.226654052734375, 45.502601623535156], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.226654052734375, 45.502601623535156], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;155&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 6221837633, &quot;lat&quot;: 45.502601623535156, &quot;lon&quot;: 12.226654052734375, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1547930550, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.225533485412598, 45.50289535522461, 12.225533485412598, 45.50289535522461], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.225533485412598, 45.50289535522461], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;156&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 6221837634, &quot;lat&quot;: 45.50289535522461, &quot;lon&quot;: 12.225533485412598, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1547999898, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.225664138793945, 45.50209045410156, 12.225664138793945, 45.50209045410156], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.225664138793945, 45.50209045410156], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;157&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 6223130109, &quot;lat&quot;: 45.50209045410156, &quot;lon&quot;: 12.225664138793945, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1547999898, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.325742721557617, 45.4363899230957, 12.325742721557617, 45.4363899230957], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.325742721557617, 45.4363899230957], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;158&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 6428467671, &quot;lat&quot;: 45.4363899230957, &quot;lon&quot;: 12.325742721557617, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1556206122, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.27448558807373, 45.50544738769531, 12.27448558807373, 45.50544738769531], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.27448558807373, 45.50544738769531], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;159&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 6591559458, &quot;lat&quot;: 45.50544738769531, &quot;lon&quot;: 12.27448558807373, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;access\&quot;:\&quot;yes\&quot;,\&quot;fee\&quot;:\&quot;no\&quot;,\&quot;wheelchair\&quot;:\&quot;yes\&quot;}&quot;, &quot;timestamp&quot;: 1562338313, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.241189002990723, 45.494163513183594, 12.241189002990723, 45.494163513183594], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.241189002990723, 45.494163513183594], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;160&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 6593051849, &quot;lat&quot;: 45.494163513183594, &quot;lon&quot;: 12.241189002990723, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1562412762, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.345271110534668, 45.43838882446289, 12.345271110534668, 45.43838882446289], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.345271110534668, 45.43838882446289], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;161&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 7019048654, &quot;lat&quot;: 45.43838882446289, &quot;lon&quot;: 12.345271110534668, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;bottle\&quot;:\&quot;yes\&quot;}&quot;, &quot;timestamp&quot;: 1629804076, &quot;version&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.207992553710938, 45.47727584838867, 12.207992553710938, 45.47727584838867], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.207992553710938, 45.47727584838867], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;162&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 7125657909, &quot;lat&quot;: 45.47727584838867, &quot;lon&quot;: 12.207992553710938, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;access\&quot;:\&quot;yes\&quot;,\&quot;wheelchair\&quot;:\&quot;yes\&quot;}&quot;, &quot;timestamp&quot;: 1578819478, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.266746520996094, 45.48699951171875, 12.266746520996094, 45.48699951171875], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.266746520996094, 45.48699951171875], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;163&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 7176891868, &quot;lat&quot;: 45.48699951171875, &quot;lon&quot;: 12.266746520996094, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;wheelchair\&quot;:\&quot;yes\&quot;}&quot;, &quot;timestamp&quot;: 1585324186, &quot;version&quot;: 2}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.267066955566406, 45.486026763916016, 12.267066955566406, 45.486026763916016], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.267066955566406, 45.486026763916016], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;164&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 7335321396, &quot;lat&quot;: 45.486026763916016, &quot;lon&quot;: 12.267066955566406, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;wheelchair\&quot;:\&quot;yes\&quot;}&quot;, &quot;timestamp&quot;: 1585324186, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.328725814819336, 45.429691314697266, 12.328725814819336, 45.429691314697266], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.328725814819336, 45.429691314697266], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;165&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 7743880807, &quot;lat&quot;: 45.429691314697266, &quot;lon&quot;: 12.328725814819336, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1595594492, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.37055778503418, 45.4083251953125, 12.37055778503418, 45.4083251953125], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.37055778503418, 45.4083251953125], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;166&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 7743887598, &quot;lat&quot;: 45.4083251953125, &quot;lon&quot;: 12.37055778503418, &quot;name&quot;: &quot;Drinking water&quot;, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1595594301, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.331302642822266, 45.42333984375, 12.331302642822266, 45.42333984375], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.331302642822266, 45.42333984375], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;167&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 7794740317, &quot;lat&quot;: 45.42333984375, &quot;lon&quot;: 12.331302642822266, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1596810806, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.282444953918457, 45.524200439453125, 12.282444953918457, 45.524200439453125], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.282444953918457, 45.524200439453125], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;168&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 7795212625, &quot;lat&quot;: 45.524200439453125, &quot;lon&quot;: 12.282444953918457, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1596828444, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.318519592285156, 45.34958267211914, 12.318519592285156, 45.34958267211914], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.318519592285156, 45.34958267211914], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;169&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 7848994968, &quot;lat&quot;: 45.34958267211914, &quot;lon&quot;: 12.318519592285156, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;access\&quot;:\&quot;yes\&quot;,\&quot;bottle\&quot;:\&quot;yes\&quot;,\&quot;wheelchair\&quot;:\&quot;yes\&quot;}&quot;, &quot;timestamp&quot;: 1598516993, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.37989616394043, 45.418575286865234, 12.37989616394043, 45.418575286865234], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.37989616394043, 45.418575286865234], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;170&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 7856229032, &quot;lat&quot;: 45.418575286865234, &quot;lon&quot;: 12.37989616394043, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1598693662, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.319398880004883, 45.44207000732422, 12.319398880004883, 45.44207000732422], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.319398880004883, 45.44207000732422], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;171&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 7997482158, &quot;lat&quot;: 45.44207000732422, &quot;lon&quot;: 12.319398880004883, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1602432393, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.191202163696289, 45.492431640625, 12.191202163696289, 45.492431640625], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.191202163696289, 45.492431640625], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;172&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 8064993638, &quot;lat&quot;: 45.492431640625, &quot;lon&quot;: 12.191202163696289, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;fee\&quot;:\&quot;no\&quot;}&quot;, &quot;timestamp&quot;: 1604050445, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.196239471435547, 45.482295989990234, 12.196239471435547, 45.482295989990234], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.196239471435547, 45.482295989990234], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;173&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 8064993660, &quot;lat&quot;: 45.482295989990234, &quot;lon&quot;: 12.196239471435547, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1604050445, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.31380844116211, 45.432167053222656, 12.31380844116211, 45.432167053222656], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.31380844116211, 45.432167053222656], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;174&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 8247454413, &quot;lat&quot;: 45.432167053222656, &quot;lon&quot;: 12.31380844116211, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1608547647, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.35496997833252, 45.43015670776367, 12.35496997833252, 45.43015670776367], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.35496997833252, 45.43015670776367], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;175&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 8541444580, &quot;lat&quot;: 45.43015670776367, &quot;lon&quot;: 12.35496997833252, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1616261851, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.33834171295166, 45.43305587768555, 12.33834171295166, 45.43305587768555], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.33834171295166, 45.43305587768555], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;176&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 8541444582, &quot;lat&quot;: 45.43305587768555, &quot;lon&quot;: 12.33834171295166, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1616261851, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.251716613769531, 45.4909782409668, 12.251716613769531, 45.4909782409668], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.251716613769531, 45.4909782409668], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;177&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 8850883426, &quot;lat&quot;: 45.4909782409668, &quot;lon&quot;: 12.251716613769531, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1624207790, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.418879508972168, 45.48458480834961, 12.418879508972168, 45.48458480834961], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.418879508972168, 45.48458480834961], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;178&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 9045308955, &quot;lat&quot;: 45.48458480834961, &quot;lon&quot;: 12.418879508972168, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: null, &quot;timestamp&quot;: 1630245863, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.259079933166504, 45.499813079833984, 12.259079933166504, 45.499813079833984], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.259079933166504, 45.499813079833984], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;179&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 9295481940, &quot;lat&quot;: 45.499813079833984, &quot;lon&quot;: 12.259079933166504, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;wheelchair\&quot;:\&quot;yes\&quot;}&quot;, &quot;timestamp&quot;: 1638260161, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [12.260167121887207, 45.50090789794922, 12.260167121887207, 45.50090789794922], &quot;geometry&quot;: {&quot;coordinates&quot;: [12.260167121887207, 45.50090789794922], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;180&quot;, &quot;properties&quot;: {&quot;addr:street&quot;: null, &quot;amenity&quot;: &quot;drinking_water&quot;, &quot;changeset&quot;: 0, &quot;drinking_water&quot;: null, &quot;fountain&quot;: null, &quot;id&quot;: 9295481941, &quot;lat&quot;: 45.50090789794922, &quot;lon&quot;: 12.260167121887207, &quot;name&quot;: null, &quot;osm_type&quot;: &quot;node&quot;, &quot;source&quot;: null, &quot;tags&quot;: &quot;{\&quot;wheelchair\&quot;:\&quot;yes\&quot;}&quot;, &quot;timestamp&quot;: 1638260161, &quot;version&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}], &quot;type&quot;: &quot;FeatureCollection&quot;});



    geo_json_f410776135205fd1f701acc20e208142.bindTooltip(
    function(layer){
    let div = L.DomUtil.create(&#x27;div&#x27;);

    let handleObject = feature=&gt;typeof(feature)==&#x27;object&#x27; ? JSON.stringify(feature) : feature;
    let fields = [&quot;lon&quot;, &quot;version&quot;, &quot;tags&quot;, &quot;id&quot;, &quot;changeset&quot;, &quot;lat&quot;, &quot;timestamp&quot;, &quot;addr:street&quot;, &quot;name&quot;, &quot;amenity&quot;, &quot;drinking_water&quot;, &quot;fountain&quot;, &quot;source&quot;, &quot;osm_type&quot;];
    let aliases = [&quot;lon&quot;, &quot;version&quot;, &quot;tags&quot;, &quot;id&quot;, &quot;changeset&quot;, &quot;lat&quot;, &quot;timestamp&quot;, &quot;addr:street&quot;, &quot;name&quot;, &quot;amenity&quot;, &quot;drinking_water&quot;, &quot;fountain&quot;, &quot;source&quot;, &quot;osm_type&quot;];
    let table = &#x27;&lt;table&gt;&#x27; +
        String(
        fields.map(
        (v,i)=&gt;
        `&lt;tr&gt;
            &lt;th&gt;${aliases[i]}&lt;/th&gt;

            &lt;td&gt;${handleObject(layer.feature.properties[v])}&lt;/td&gt;
        &lt;/tr&gt;`).join(&#x27;&#x27;))
    +&#x27;&lt;/table&gt;&#x27;;
    div.innerHTML=table;

    return div
    }
    ,{&quot;className&quot;: &quot;foliumtooltip&quot;, &quot;sticky&quot;: true});

&lt;/script&gt;" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



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




    (597, 11)




```python
benchs.columns
```




    Index(['lon', 'version', 'tags', 'id', 'changeset', 'lat', 'timestamp',
           'amenity', 'source', 'geometry', 'osm_type'],
          dtype='object')




```python
benchs.tags.unique()
```




    array([None, '{"backrest":"yes"}',
           '{"backrest":"yes","check_date":"2021-02-15"}',
           '{"backrest":"yes","note":"still exists?"}',
           '{"backrest":"yes","check_date":"2021-05-31","material":"wood","natural":"tree"}',
           '{"backrest":"yes","check_date":"2021-05-31","material":"wood"}',
           '{"backrest":"yes","direction":"110","material":"wood"}',
           '{"backrest":"yes","check_date":"2022-04-06"}',
           '{"backrest":"no","material":"stone"}',
           '{"backrest":"yes","check_date":"2021-05-27"}',
           '{"backrest":"no"}',
           '{"backrest":"yes","direction":"290","material":"wood"}',
           '{"backrest":"yes","direction":"200","material":"wood"}',
           '{"backrest":"yes","direction":"20","material":"wood"}',
           '{"backrest":"yes","colour":"red","material":"wood","seats":"3","survey:date":"2021-08-25"}',
           '{"backrest":"yes","colour":"red","seats":"3","survey:date":"2021-08-25"}',
           '{"backrest":"yes","direction":"130"}',
           '{"backrest":"yes","direction":"35"}',
           '{"backrest":"yes","direction":"215"}',
           '{"backrest":"yes","material":"metal","seats":"1"}',
           '{"backrest":"yes","material":"wood","seats":"3"}',
           '{"backrest":"yes","colour":"green","material":"metal","seats":"4"}',
           '{"backrest":"yes","colour":"red","material":"wood"}',
           '{"backrest":"no","survey:date":"2021-12-17"}',
           '{"backrest":"yes","survey:date":"2021-12-17"}',
           '{"backrest":"yes","colour":"brown","material":"wood"}',
           '{"backrest":"yes","colour":"red","material":"wood","shop":"newsagent"}',
           '{"backrest":"yes","colour":"Rosso","material":"wood","seats":"3"}'],
          dtype=object)



quick&dirty solution


```python
benchs_with_backrest = benchs[benchs.tags.str.find('"backrest":"yes"') > -1]
```


```python
benchs_with_backrest.tags.unique()
```




    array(['{"backrest":"yes"}',
           '{"backrest":"yes","check_date":"2021-02-15"}',
           '{"backrest":"yes","note":"still exists?"}',
           '{"backrest":"yes","check_date":"2021-05-31","material":"wood","natural":"tree"}',
           '{"backrest":"yes","check_date":"2021-05-31","material":"wood"}',
           '{"backrest":"yes","direction":"110","material":"wood"}',
           '{"backrest":"yes","check_date":"2022-04-06"}',
           '{"backrest":"yes","check_date":"2021-05-27"}',
           '{"backrest":"yes","direction":"290","material":"wood"}',
           '{"backrest":"yes","direction":"200","material":"wood"}',
           '{"backrest":"yes","direction":"20","material":"wood"}',
           '{"backrest":"yes","colour":"red","material":"wood","seats":"3","survey:date":"2021-08-25"}',
           '{"backrest":"yes","colour":"red","seats":"3","survey:date":"2021-08-25"}',
           '{"backrest":"yes","direction":"130"}',
           '{"backrest":"yes","direction":"35"}',
           '{"backrest":"yes","direction":"215"}',
           '{"backrest":"yes","material":"metal","seats":"1"}',
           '{"backrest":"yes","material":"wood","seats":"3"}',
           '{"backrest":"yes","colour":"green","material":"metal","seats":"4"}',
           '{"backrest":"yes","colour":"red","material":"wood"}',
           '{"backrest":"yes","survey:date":"2021-12-17"}',
           '{"backrest":"yes","colour":"brown","material":"wood"}',
           '{"backrest":"yes","colour":"red","material":"wood","shop":"newsagent"}',
           '{"backrest":"yes","colour":"Rosso","material":"wood","seats":"3"}'],
          dtype=object)




```python
benchs_with_backrest.shape
```




    (279, 11)




```python
print("the total of benchs tagged with the presence of a backrest is %s" % (str(benchs_with_backrest.shape[0])))
```

    the total of benchs tagged with the presence of a backrest is 279

