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
  import pyrosm
except ModuleNotFoundError as e:
  !pip install pyrosm==0.6.1
  import pyrosm

```

# Exercise
- download from OpenStreetMap all supermarkets inside the bounding box of the city in this point<br/>
   latitude: 46.21209<br/>
   longitude: 11.09351
- identify the longest road of the city (state roads, walking routes, motorways are excluded). Please use "unclassified"
- How many drinking water are in this city?
- How many benches in this city have the backrest?


## download from OpenStreetMap all supermarkets inside the bounding box of the city in this point and

   latitude: 46.21209<br/>
   longitude: 11.09351


```python
import geopandas as gpd
```


```python
latitude = 46.21209
longitude = 11.09351
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




    {'Match_addr': 'Via Francesco Filos 12A, 38017, Mezzolombardo, Trento',
     'LongLabel': 'Via Francesco Filos 12A, 38017, Mezzolombardo, Trento, ITA',
     'ShortLabel': 'Via Francesco Filos 12A',
     'Addr_type': 'PointAddress',
     'Type': '',
     'PlaceName': '',
     'AddNum': '12A',
     'Address': 'Via Francesco Filos 12A',
     'Block': '',
     'Sector': '',
     'Neighborhood': '',
     'District': 'Mezzolombardo',
     'City': 'Mezzolombardo',
     'MetroArea': '',
     'Subregion': 'Trento',
     'Region': 'Trentino-Alto Adige',
     'Territory': '',
     'Postal': '38017',
     'PostalExt': '',
     'CountryCode': 'ITA'}




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
point = Point(longitude, latitude)
```


```python
urlmunicipalities2021 = 'https://github.com/napo/geospatial_course_unitn/raw/master/data/istat/shapefile_istat_municipalities_2021.zip'
municipalities2021 = gpd.read_file(urlmunicipalities2021)
```


```python
muncipality = municipalities2021[municipalities2021.to_crs(epsg=4326).geometry.contains(point)]
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
      <th>SHAPE_LENG</th>
      <th>Shape_Le_1</th>
      <th>Shape_Area</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>5729</th>
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
      <td>19772.366696</td>
      <td>19772.224335</td>
      <td>1.387899e+07</td>
      <td>POLYGON ((659991.073 5121616.939, 660070.949 5...</td>
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
municipality = municipalities2021[municipalities2021.COMUNE==city]
```

## Donwload the PBF of Mezzolombardo from OSM Estratti

![](https://raw.githubusercontent.com/napo/geospatial_course_unitn/master/images/osm_estratti_mezzolombardo.jpg)


```python
url_download_mezzolombardo_pbf = 'https://osmit-estratti.wmcloud.org/dati/poly/comuni/pbf/022117_Mezzolombardo_poly.osm.pbf'
import urllib.request
urllib.request.urlretrieve(url_download_mezzolombardo_pbf ,"mezzolombardo_osm.pbf")
```




    ('mezzolombardo_osm.pbf', <http.client.HTTPMessage at 0x7fefb844d9d0>)



## find all the supermarkets in the area




```python
import pyrosm
```

![](https://raw.githubusercontent.com/napo/geospatial_course_unitn/master/images/tag_supermarket.png)


```python
osm = pyrosm.OSM("mezzolombardo_osm.pbf") 
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




    (6, 19)




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
      <th>version</th>
      <th>timestamp</th>
      <th>lon</th>
      <th>tags</th>
      <th>id</th>
      <th>lat</th>
      <th>changeset</th>
      <th>addr:city</th>
      <th>addr:country</th>
      <th>addr:housenumber</th>
      <th>addr:housename</th>
      <th>addr:postcode</th>
      <th>addr:street</th>
      <th>name</th>
      <th>opening_hours</th>
      <th>shop</th>
      <th>geometry</th>
      <th>osm_type</th>
      <th>operator</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>1409592099</td>
      <td>11.089408</td>
      <td>None</td>
      <td>3054012770</td>
      <td>46.213261</td>
      <td>0.0</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>Despar</td>
      <td>None</td>
      <td>supermarket</td>
      <td>POINT (11.08941 46.21326)</td>
      <td>node</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>5</td>
      <td>1594466282</td>
      <td>11.100459</td>
      <td>{"brand":"EuroSpin","level":"0","wheelchair":"...</td>
      <td>4557760211</td>
      <td>46.207256</td>
      <td>0.0</td>
      <td>Mezzolombardo</td>
      <td>IT</td>
      <td>65/A</td>
      <td>Complesso Commerciale Braide</td>
      <td>38017</td>
      <td>Via Trento</td>
      <td>EuroSpin</td>
      <td>Mo-Su 08:00-20:00</td>
      <td>supermarket</td>
      <td>POINT (11.10046 46.20726)</td>
      <td>node</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>8</td>
      <td>1523179735</td>
      <td>NaN</td>
      <td>{"building":"yes","ref:vatin":"IT00108640228"}</td>
      <td>39484297</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Mezzolombardo</td>
      <td>IT</td>
      <td>33</td>
      <td>NaN</td>
      <td>38017</td>
      <td>Via Guido Fiorini</td>
      <td>Orvea</td>
      <td>None</td>
      <td>supermarket</td>
      <td>POLYGON ((11.10120 46.20900, 11.10125 46.20917...</td>
      <td>way</td>
      <td>OR.VE.A. S.P.A.</td>
    </tr>
    <tr>
      <th>3</th>
      <td>7</td>
      <td>1615675726</td>
      <td>NaN</td>
      <td>{"building":"yes"}</td>
      <td>73412650</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Mezzolombardo</td>
      <td>IT</td>
      <td>5</td>
      <td>NaN</td>
      <td>38017</td>
      <td>Via Francesco Morigl</td>
      <td>Amort</td>
      <td>None</td>
      <td>supermarket</td>
      <td>POLYGON ((11.09448 46.21776, 11.09412 46.21755...</td>
      <td>way</td>
      <td>None</td>
    </tr>
    <tr>
      <th>4</th>
      <td>3</td>
      <td>1635084049</td>
      <td>NaN</td>
      <td>{"brand":"Spar","brand:wikidata":"Q610492","br...</td>
      <td>301331720</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>NaN</td>
      <td>None</td>
      <td>None</td>
      <td>Despar</td>
      <td>Mo-Sa 07:30-13:00, 14:30-19:30</td>
      <td>supermarket</td>
      <td>POLYGON ((11.08941 46.21320, 11.08951 46.21327...</td>
      <td>way</td>
      <td>None</td>
    </tr>
    <tr>
      <th>5</th>
      <td>5</td>
      <td>1594466282</td>
      <td>NaN</td>
      <td>{"brand":"Lidl","fixme":"position","source":"l...</td>
      <td>567538828</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Mezzolombardo</td>
      <td>IT</td>
      <td>79</td>
      <td>NaN</td>
      <td>38017</td>
      <td>Via Alcide De Gasperi</td>
      <td>LIDL</td>
      <td>Mo-Sa 08:00-21:00; Su 08:00-20:00</td>
      <td>supermarket</td>
      <td>POLYGON ((11.09423 46.21056, 11.09453 46.21074...</td>
      <td>way</td>
      <td>LIDL ITALIA SRL</td>
    </tr>
  </tbody>
</table>
</div>




```python
print("there are %s supermarkets in Mezzolombardo" % supermarkets.shape[0])
```

    there are 6 supermarkets in Mezzolombardo



```python
supermarkets.name
```




    0      Despar
    1    EuroSpin
    2       Orvea
    3       Amort
    4      Despar
    5        LIDL
    Name: name, dtype: object



# identify the longest road of the city



```python
roads = osm.get_network(network_type='all')
```


```python
roads.columns
```




    Index(['access', 'area', 'bicycle', 'bridge', 'cycleway', 'foot', 'footway',
           'highway', 'junction', 'lanes', 'maxspeed', 'motor_vehicle', 'name',
           'oneway', 'ref', 'service', 'segregated', 'smoothness', 'surface',
           'tracktype', 'tunnel', 'id', 'timestamp', 'version', 'tags', 'osm_type',
           'geometry', 'length'],
          dtype='object')




```python
roads.highway.unique()
```




    array(['tertiary', 'unclassified', 'residential', 'track', 'primary',
           'secondary', 'cycleway', 'tertiary_link', 'path', 'service',
           'primary_link', 'steps', 'footway', 'pedestrian', 'secondary_link'],
          dtype=object)



![](https://github.com/napo/geospatial_course_unitn/raw/master/images/tag_highways.gif)

## lenght of unclassified and residential togheter


```python
roads_unclassified_residential = roads[(roads.highway=='unclassified') |  (roads.highway == 'tertiary')]
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
      <th>junction</th>
      <th>lanes</th>
      <th>...</th>
      <th>surface</th>
      <th>tracktype</th>
      <th>tunnel</th>
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
      <th>0</th>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>tertiary</td>
      <td>roundabout</td>
      <td>None</td>
      <td>...</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>24732926</td>
      <td>1598641256</td>
      <td>7</td>
      <td>None</td>
      <td>way</td>
      <td>MULTILINESTRING ((11.10439 46.19333, 11.10438 ...</td>
      <td>132.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>tertiary</td>
      <td>roundabout</td>
      <td>None</td>
      <td>...</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>24732928</td>
      <td>1464199823</td>
      <td>9</td>
      <td>None</td>
      <td>way</td>
      <td>MULTILINESTRING ((11.10043 46.20075, 11.10048 ...</td>
      <td>100.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>tertiary</td>
      <td>roundabout</td>
      <td>None</td>
      <td>...</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>24732930</td>
      <td>1598641256</td>
      <td>10</td>
      <td>None</td>
      <td>way</td>
      <td>MULTILINESTRING ((11.10073 46.20638, 11.10078 ...</td>
      <td>106.0</td>
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
      <td>tertiary</td>
      <td>None</td>
      <td>None</td>
      <td>...</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>24732932</td>
      <td>1464199684</td>
      <td>9</td>
      <td>{"loc_ref":"SP90"}</td>
      <td>way</td>
      <td>MULTILINESTRING ((11.10087 46.20666, 11.10091 ...</td>
      <td>108.0</td>
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
      <td>...</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>25413543</td>
      <td>1522001556</td>
      <td>26</td>
      <td>None</td>
      <td>way</td>
      <td>MULTILINESTRING ((11.09512 46.20965, 11.09569 ...</td>
      <td>786.0</td>
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
      <th>873</th>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>tertiary</td>
      <td>None</td>
      <td>None</td>
      <td>...</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>737863922</td>
      <td>1571833099</td>
      <td>1</td>
      <td>None</td>
      <td>way</td>
      <td>MULTILINESTRING ((11.10378 46.19349, 11.10384 ...</td>
      <td>16.0</td>
    </tr>
    <tr>
      <th>874</th>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>tertiary</td>
      <td>None</td>
      <td>None</td>
      <td>...</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>737863923</td>
      <td>1571833099</td>
      <td>1</td>
      <td>None</td>
      <td>way</td>
      <td>MULTILINESTRING ((11.10395 46.19349, 11.10378 ...</td>
      <td>13.0</td>
    </tr>
    <tr>
      <th>897</th>
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
      <td>...</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>915122844</td>
      <td>1615309919</td>
      <td>1</td>
      <td>None</td>
      <td>way</td>
      <td>MULTILINESTRING ((11.09157 46.20730, 11.09160 ...</td>
      <td>148.0</td>
    </tr>
    <tr>
      <th>927</th>
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
      <td>...</td>
      <td>sett</td>
      <td>None</td>
      <td>None</td>
      <td>915122880</td>
      <td>1615309919</td>
      <td>1</td>
      <td>None</td>
      <td>way</td>
      <td>MULTILINESTRING ((11.09247 46.20816, 11.09265 ...</td>
      <td>106.0</td>
    </tr>
    <tr>
      <th>929</th>
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
      <td>...</td>
      <td>sett</td>
      <td>None</td>
      <td>None</td>
      <td>915122883</td>
      <td>1615309919</td>
      <td>1</td>
      <td>None</td>
      <td>way</td>
      <td>MULTILINESTRING ((11.09222 46.20825, 11.09247 ...</td>
      <td>21.0</td>
    </tr>
  </tbody>
</table>
<p>69 rows × 28 columns</p>
</div>




```python
roads_unclassified_residential['name'].value_counts()
```




    Via Trento                   4
    Via Carlo Devigili           3
    Via Sant'Antonio             3
    Piazza Pio XII               3
    Via Rotaliana                2
    Via della Rupe               2
    Piazza Cassa di Risparmio    2
    Via Milano                   1
    Via Guido Fiorini            1
    Piazza Cesare Battisti       1
    Corso del Popolo             1
    Via Francesco Filos          1
    Via degli Alpini             1
    Viale Fenice                 1
    Via Fabio Filzi              1
    Località Galletta            1
    Name: name, dtype: int64




```python
names = roads_unclassified_residential.name.unique()
```


```python
names
```




    array([None, 'Via Carlo Devigili', 'Via Rotaliana', 'Via Trento',
           "Via Sant'Antonio", 'Via Milano', 'Piazza Pio XII',
           'Via della Rupe', 'Via Guido Fiorini', 'Piazza Cassa di Risparmio',
           'Piazza Cesare Battisti', 'Corso del Popolo',
           'Via Francesco Filos', 'Via degli Alpini', 'Viale Fenice',
           'Via Fabio Filzi', 'Località Galletta'], dtype=object)




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




    {None: 0.0,
     'Via Carlo Devigili': 936.754889705999,
     'Via Rotaliana': 774.3678080999027,
     'Via Trento': 1479.2922178941103,
     "Via Sant'Antonio": 317.20741812669604,
     'Via Milano': 428.4259989016135,
     'Piazza Pio XII': 244.72762778938096,
     'Via della Rupe': 657.2358490220092,
     'Via Guido Fiorini': 262.2489738895878,
     'Piazza Cassa di Risparmio': 101.69215459438668,
     'Piazza Cesare Battisti': 74.36442381488386,
     'Corso del Popolo': 254.15957956408997,
     'Via Francesco Filos': 101.21436392814823,
     'Via degli Alpini': 223.4884323682712,
     'Viale Fenice': 377.86898287344684,
     'Via Fabio Filzi': 131.1560732099417,
     'Località Galletta': 321.34567496279004}




```python
max(roads_lenght, key=roads_lenght.get)
```




    'Via Trento'




```python
longest_road = max(roads_lenght, key=roads_lenght.get)
```


```python
roads_lenght[longest_road]
```




    1479.2922178941103




```python
print("the longest road (unclassified + residential) in Mezzolombardo is %s and is long %.1f km" % (str(longest_road),roads_lenght[longest_road]/1000))
```

    the longest road (unclassified + residential) in Mezzolombardo is Via Trento and is long 1.5 km


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

    the longest road (unclassified) is Via Trento and is long 1.5 km


## where is this road?
... we can use overpass-turbo.eu with this wizard query

```
name='Via Rotaliana' in Mezzolombardo
```

http://overpass-turbo.eu/s/1cuk

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




    (17, 10)




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
      <th>version</th>
      <th>timestamp</th>
      <th>lon</th>
      <th>tags</th>
      <th>id</th>
      <th>lat</th>
      <th>changeset</th>
      <th>amenity</th>
      <th>geometry</th>
      <th>osm_type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2</td>
      <td>1489783071</td>
      <td>11.092862</td>
      <td>None</td>
      <td>388416004</td>
      <td>46.225494</td>
      <td>0</td>
      <td>drinking_water</td>
      <td>POINT (11.09286 46.22549)</td>
      <td>node</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>1489783071</td>
      <td>11.094058</td>
      <td>None</td>
      <td>388416008</td>
      <td>46.224617</td>
      <td>0</td>
      <td>drinking_water</td>
      <td>POINT (11.09406 46.22462)</td>
      <td>node</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1</td>
      <td>1282084067</td>
      <td>11.089766</td>
      <td>None</td>
      <td>865333800</td>
      <td>46.214092</td>
      <td>0</td>
      <td>drinking_water</td>
      <td>POINT (11.08977 46.21409)</td>
      <td>node</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>1306775522</td>
      <td>11.093194</td>
      <td>None</td>
      <td>937561830</td>
      <td>46.213287</td>
      <td>0</td>
      <td>drinking_water</td>
      <td>POINT (11.09319 46.21329)</td>
      <td>node</td>
    </tr>
  </tbody>
</table>
</div>




```python
drinking_water.explore()
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe src="about:blank" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" data-html=%3C%21DOCTYPE%20html%3E%0A%3Chead%3E%20%20%20%20%0A%20%20%20%20%3Cmeta%20http-equiv%3D%22content-type%22%20content%3D%22text/html%3B%20charset%3DUTF-8%22%20/%3E%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%3Cscript%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20L_NO_TOUCH%20%3D%20false%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20L_DISABLE_3D%20%3D%20false%3B%0A%20%20%20%20%20%20%20%20%3C/script%3E%0A%20%20%20%20%0A%20%20%20%20%3Cstyle%3Ehtml%2C%20body%20%7Bwidth%3A%20100%25%3Bheight%3A%20100%25%3Bmargin%3A%200%3Bpadding%3A%200%3B%7D%3C/style%3E%0A%20%20%20%20%3Cstyle%3E%23map%20%7Bposition%3Aabsolute%3Btop%3A0%3Bbottom%3A0%3Bright%3A0%3Bleft%3A0%3B%7D%3C/style%3E%0A%20%20%20%20%3Cscript%20src%3D%22https%3A//cdn.jsdelivr.net/npm/leaflet%401.6.0/dist/leaflet.js%22%3E%3C/script%3E%0A%20%20%20%20%3Cscript%20src%3D%22https%3A//code.jquery.com/jquery-1.12.4.min.js%22%3E%3C/script%3E%0A%20%20%20%20%3Cscript%20src%3D%22https%3A//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/js/bootstrap.min.js%22%3E%3C/script%3E%0A%20%20%20%20%3Cscript%20src%3D%22https%3A//cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.js%22%3E%3C/script%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//cdn.jsdelivr.net/npm/leaflet%401.6.0/dist/leaflet.css%22/%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css%22/%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap-theme.min.css%22/%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//maxcdn.bootstrapcdn.com/font-awesome/4.6.3/css/font-awesome.min.css%22/%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.css%22/%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//cdn.jsdelivr.net/gh/python-visualization/folium/folium/templates/leaflet.awesome.rotate.min.css%22/%3E%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Cmeta%20name%3D%22viewport%22%20content%3D%22width%3Ddevice-width%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20initial-scale%3D1.0%2C%20maximum-scale%3D1.0%2C%20user-scalable%3Dno%22%20/%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Cstyle%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%23map_023f81eef25c4eab851cda85a33b8213%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20position%3A%20relative%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20width%3A%20100.0%25%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20height%3A%20100.0%25%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20left%3A%200.0%25%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20top%3A%200.0%25%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%3C/style%3E%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%3Cstyle%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20.foliumtooltip%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20.foliumtooltip%20table%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20margin%3A%20auto%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20.foliumtooltip%20tr%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20text-align%3A%20left%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20.foliumtooltip%20th%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20padding%3A%202px%3B%20padding-right%3A%208px%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%3C/style%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%0A%3C/head%3E%0A%3Cbody%3E%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Cdiv%20class%3D%22folium-map%22%20id%3D%22map_023f81eef25c4eab851cda85a33b8213%22%20%3E%3C/div%3E%0A%20%20%20%20%20%20%20%20%0A%3C/body%3E%0A%3Cscript%3E%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20var%20map_023f81eef25c4eab851cda85a33b8213%20%3D%20L.map%28%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22map_023f81eef25c4eab851cda85a33b8213%22%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20center%3A%20%5B46.21571731567383%2C%2011.087119102478027%5D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20crs%3A%20L.CRS.EPSG3857%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20zoom%3A%2010%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20zoomControl%3A%20true%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20preferCanvas%3A%20false%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%29%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20L.control.scale%28%29.addTo%28map_023f81eef25c4eab851cda85a33b8213%29%3B%0A%0A%20%20%20%20%20%20%20%20%20%20%20%20%0A%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20var%20tile_layer_9a6c8e9242ed4097af4c99a8f5c6a391%20%3D%20L.tileLayer%28%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22https%3A//%7Bs%7D.tile.openstreetmap.org/%7Bz%7D/%7Bx%7D/%7By%7D.png%22%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7B%22attribution%22%3A%20%22Data%20by%20%5Cu0026copy%3B%20%5Cu003ca%20href%3D%5C%22http%3A//openstreetmap.org%5C%22%5Cu003eOpenStreetMap%5Cu003c/a%5Cu003e%2C%20under%20%5Cu003ca%20href%3D%5C%22http%3A//www.openstreetmap.org/copyright%5C%22%5Cu003eODbL%5Cu003c/a%5Cu003e.%22%2C%20%22detectRetina%22%3A%20false%2C%20%22maxNativeZoom%22%3A%2018%2C%20%22maxZoom%22%3A%2018%2C%20%22minZoom%22%3A%200%2C%20%22noWrap%22%3A%20false%2C%20%22opacity%22%3A%201%2C%20%22subdomains%22%3A%20%22abc%22%2C%20%22tms%22%3A%20false%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%29.addTo%28map_023f81eef25c4eab851cda85a33b8213%29%3B%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20map_023f81eef25c4eab851cda85a33b8213.fitBounds%28%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%5B%5B46.20384979248047%2C%2011.070927619934082%5D%2C%20%5B46.22758483886719%2C%2011.103310585021973%5D%5D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7B%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%29%3B%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20function%20geo_json_04446c227a3448468ac193e8b2d31da9_styler%28feature%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20switch%28feature.id%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20default%3A%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20%7B%22fillOpacity%22%3A%200.5%2C%20%22weight%22%3A%202%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20function%20geo_json_04446c227a3448468ac193e8b2d31da9_highlighter%28feature%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20switch%28feature.id%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20default%3A%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20%7B%22fillOpacity%22%3A%200.75%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20function%20geo_json_04446c227a3448468ac193e8b2d31da9_pointToLayer%28feature%2C%20latlng%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20var%20opts%20%3D%20%7B%22bubblingMouseEvents%22%3A%20true%2C%20%22color%22%3A%20%22%233388ff%22%2C%20%22dashArray%22%3A%20null%2C%20%22dashOffset%22%3A%20null%2C%20%22fill%22%3A%20true%2C%20%22fillColor%22%3A%20%22%233388ff%22%2C%20%22fillOpacity%22%3A%200.2%2C%20%22fillRule%22%3A%20%22evenodd%22%2C%20%22lineCap%22%3A%20%22round%22%2C%20%22lineJoin%22%3A%20%22round%22%2C%20%22opacity%22%3A%201.0%2C%20%22radius%22%3A%202%2C%20%22stroke%22%3A%20true%2C%20%22weight%22%3A%203%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20let%20style%20%3D%20geo_json_04446c227a3448468ac193e8b2d31da9_styler%28feature%29%0A%20%20%20%20%20%20%20%20%20%20%20%20Object.assign%28opts%2C%20style%29%0A%20%20%20%20%20%20%20%20%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20return%20new%20L.CircleMarker%28latlng%2C%20opts%29%0A%20%20%20%20%20%20%20%20%7D%0A%0A%20%20%20%20%20%20%20%20function%20geo_json_04446c227a3448468ac193e8b2d31da9_onEachFeature%28feature%2C%20layer%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20layer.on%28%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20mouseout%3A%20function%28e%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20if%28typeof%20e.target.setStyle%20%3D%3D%3D%20%22function%22%29%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20geo_json_04446c227a3448468ac193e8b2d31da9.resetStyle%28e.target%29%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20mouseover%3A%20function%28e%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20if%28typeof%20e.target.setStyle%20%3D%3D%3D%20%22function%22%29%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20const%20highlightStyle%20%3D%20geo_json_04446c227a3448468ac193e8b2d31da9_highlighter%28e.target.feature%29%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20e.target.setStyle%28highlightStyle%29%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%29%3B%0A%20%20%20%20%20%20%20%20%7D%3B%0A%20%20%20%20%20%20%20%20var%20geo_json_04446c227a3448468ac193e8b2d31da9%20%3D%20L.geoJson%28null%2C%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20onEachFeature%3A%20geo_json_04446c227a3448468ac193e8b2d31da9_onEachFeature%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20style%3A%20geo_json_04446c227a3448468ac193e8b2d31da9_styler%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20pointToLayer%3A%20geo_json_04446c227a3448468ac193e8b2d31da9_pointToLayer%0A%20%20%20%20%20%20%20%20%7D%29%3B%0A%0A%20%20%20%20%20%20%20%20function%20geo_json_04446c227a3448468ac193e8b2d31da9_add%20%28data%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20geo_json_04446c227a3448468ac193e8b2d31da9%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20.addData%28data%29%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20.addTo%28map_023f81eef25c4eab851cda85a33b8213%29%3B%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20geo_json_04446c227a3448468ac193e8b2d31da9_add%28%7B%22bbox%22%3A%20%5B11.070927619934082%2C%2046.20384979248047%2C%2011.103310585021973%2C%2046.22758483886719%5D%2C%20%22features%22%3A%20%5B%7B%22bbox%22%3A%20%5B11.092862129211426%2C%2046.225494384765625%2C%2011.092862129211426%2C%2046.225494384765625%5D%2C%20%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B11.092862129211426%2C%2046.225494384765625%5D%2C%20%22type%22%3A%20%22Point%22%7D%2C%20%22id%22%3A%20%220%22%2C%20%22properties%22%3A%20%7B%22amenity%22%3A%20%22drinking_water%22%2C%20%22changeset%22%3A%200%2C%20%22id%22%3A%20388416004%2C%20%22lat%22%3A%2046.225494384765625%2C%20%22lon%22%3A%2011.092862129211426%2C%20%22osm_type%22%3A%20%22node%22%2C%20%22tags%22%3A%20null%2C%20%22timestamp%22%3A%201489783071%2C%20%22version%22%3A%202%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22bbox%22%3A%20%5B11.0940580368042%2C%2046.22461700439453%2C%2011.0940580368042%2C%2046.22461700439453%5D%2C%20%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B11.0940580368042%2C%2046.22461700439453%5D%2C%20%22type%22%3A%20%22Point%22%7D%2C%20%22id%22%3A%20%221%22%2C%20%22properties%22%3A%20%7B%22amenity%22%3A%20%22drinking_water%22%2C%20%22changeset%22%3A%200%2C%20%22id%22%3A%20388416008%2C%20%22lat%22%3A%2046.22461700439453%2C%20%22lon%22%3A%2011.0940580368042%2C%20%22osm_type%22%3A%20%22node%22%2C%20%22tags%22%3A%20null%2C%20%22timestamp%22%3A%201489783071%2C%20%22version%22%3A%202%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22bbox%22%3A%20%5B11.089765548706055%2C%2046.21409225463867%2C%2011.089765548706055%2C%2046.21409225463867%5D%2C%20%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B11.089765548706055%2C%2046.21409225463867%5D%2C%20%22type%22%3A%20%22Point%22%7D%2C%20%22id%22%3A%20%222%22%2C%20%22properties%22%3A%20%7B%22amenity%22%3A%20%22drinking_water%22%2C%20%22changeset%22%3A%200%2C%20%22id%22%3A%20865333800%2C%20%22lat%22%3A%2046.21409225463867%2C%20%22lon%22%3A%2011.089765548706055%2C%20%22osm_type%22%3A%20%22node%22%2C%20%22tags%22%3A%20null%2C%20%22timestamp%22%3A%201282084067%2C%20%22version%22%3A%201%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22bbox%22%3A%20%5B11.093194007873535%2C%2046.213287353515625%2C%2011.093194007873535%2C%2046.213287353515625%5D%2C%20%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B11.093194007873535%2C%2046.213287353515625%5D%2C%20%22type%22%3A%20%22Point%22%7D%2C%20%22id%22%3A%20%223%22%2C%20%22properties%22%3A%20%7B%22amenity%22%3A%20%22drinking_water%22%2C%20%22changeset%22%3A%200%2C%20%22id%22%3A%20937561830%2C%20%22lat%22%3A%2046.213287353515625%2C%20%22lon%22%3A%2011.093194007873535%2C%20%22osm_type%22%3A%20%22node%22%2C%20%22tags%22%3A%20null%2C%20%22timestamp%22%3A%201306775522%2C%20%22version%22%3A%203%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22bbox%22%3A%20%5B11.092707633972168%2C%2046.21295166015625%2C%2011.092707633972168%2C%2046.21295166015625%5D%2C%20%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B11.092707633972168%2C%2046.21295166015625%5D%2C%20%22type%22%3A%20%22Point%22%7D%2C%20%22id%22%3A%20%224%22%2C%20%22properties%22%3A%20%7B%22amenity%22%3A%20%22drinking_water%22%2C%20%22changeset%22%3A%200%2C%20%22id%22%3A%20937561879%2C%20%22lat%22%3A%2046.21295166015625%2C%20%22lon%22%3A%2011.092707633972168%2C%20%22osm_type%22%3A%20%22node%22%2C%20%22tags%22%3A%20null%2C%20%22timestamp%22%3A%201286186622%2C%20%22version%22%3A%201%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22bbox%22%3A%20%5B11.09286117553711%2C%2046.21254348754883%2C%2011.09286117553711%2C%2046.21254348754883%5D%2C%20%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B11.09286117553711%2C%2046.21254348754883%5D%2C%20%22type%22%3A%20%22Point%22%7D%2C%20%22id%22%3A%20%225%22%2C%20%22properties%22%3A%20%7B%22amenity%22%3A%20%22drinking_water%22%2C%20%22changeset%22%3A%200%2C%20%22id%22%3A%201147996707%2C%20%22lat%22%3A%2046.21254348754883%2C%20%22lon%22%3A%2011.09286117553711%2C%20%22osm_type%22%3A%20%22node%22%2C%20%22tags%22%3A%20null%2C%20%22timestamp%22%3A%201297459506%2C%20%22version%22%3A%201%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22bbox%22%3A%20%5B11.089774131774902%2C%2046.212059020996094%2C%2011.089774131774902%2C%2046.212059020996094%5D%2C%20%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B11.089774131774902%2C%2046.212059020996094%5D%2C%20%22type%22%3A%20%22Point%22%7D%2C%20%22id%22%3A%20%226%22%2C%20%22properties%22%3A%20%7B%22amenity%22%3A%20%22drinking_water%22%2C%20%22changeset%22%3A%200%2C%20%22id%22%3A%201297031951%2C%20%22lat%22%3A%2046.212059020996094%2C%20%22lon%22%3A%2011.089774131774902%2C%20%22osm_type%22%3A%20%22node%22%2C%20%22tags%22%3A%20null%2C%20%22timestamp%22%3A%201306135883%2C%20%22version%22%3A%201%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22bbox%22%3A%20%5B11.09454345703125%2C%2046.208744049072266%2C%2011.09454345703125%2C%2046.208744049072266%5D%2C%20%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B11.09454345703125%2C%2046.208744049072266%5D%2C%20%22type%22%3A%20%22Point%22%7D%2C%20%22id%22%3A%20%227%22%2C%20%22properties%22%3A%20%7B%22amenity%22%3A%20%22drinking_water%22%2C%20%22changeset%22%3A%200%2C%20%22id%22%3A%201633035183%2C%20%22lat%22%3A%2046.208744049072266%2C%20%22lon%22%3A%2011.09454345703125%2C%20%22osm_type%22%3A%20%22node%22%2C%20%22tags%22%3A%20null%2C%20%22timestamp%22%3A%201465422392%2C%20%22version%22%3A%202%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22bbox%22%3A%20%5B11.087976455688477%2C%2046.213199615478516%2C%2011.087976455688477%2C%2046.213199615478516%5D%2C%20%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B11.087976455688477%2C%2046.213199615478516%5D%2C%20%22type%22%3A%20%22Point%22%7D%2C%20%22id%22%3A%20%228%22%2C%20%22properties%22%3A%20%7B%22amenity%22%3A%20%22drinking_water%22%2C%20%22changeset%22%3A%200%2C%20%22id%22%3A%201721725132%2C%20%22lat%22%3A%2046.213199615478516%2C%20%22lon%22%3A%2011.087976455688477%2C%20%22osm_type%22%3A%20%22node%22%2C%20%22tags%22%3A%20%22%7B%5C%22fixme%5C%22%3A%5C%22position%5C%22%7D%22%2C%20%22timestamp%22%3A%201334751180%2C%20%22version%22%3A%201%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22bbox%22%3A%20%5B11.09571361541748%2C%2046.20851135253906%2C%2011.09571361541748%2C%2046.20851135253906%5D%2C%20%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B11.09571361541748%2C%2046.20851135253906%5D%2C%20%22type%22%3A%20%22Point%22%7D%2C%20%22id%22%3A%20%229%22%2C%20%22properties%22%3A%20%7B%22amenity%22%3A%20%22drinking_water%22%2C%20%22changeset%22%3A%200%2C%20%22id%22%3A%202019113888%2C%20%22lat%22%3A%2046.20851135253906%2C%20%22lon%22%3A%2011.09571361541748%2C%20%22osm_type%22%3A%20%22node%22%2C%20%22tags%22%3A%20null%2C%20%22timestamp%22%3A%201353202522%2C%20%22version%22%3A%201%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22bbox%22%3A%20%5B11.089981079101562%2C%2046.212032318115234%2C%2011.089981079101562%2C%2046.212032318115234%5D%2C%20%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B11.089981079101562%2C%2046.212032318115234%5D%2C%20%22type%22%3A%20%22Point%22%7D%2C%20%22id%22%3A%20%2210%22%2C%20%22properties%22%3A%20%7B%22amenity%22%3A%20%22drinking_water%22%2C%20%22changeset%22%3A%200%2C%20%22id%22%3A%202442987448%2C%20%22lat%22%3A%2046.212032318115234%2C%20%22lon%22%3A%2011.089981079101562%2C%20%22osm_type%22%3A%20%22node%22%2C%20%22tags%22%3A%20null%2C%20%22timestamp%22%3A%201378129798%2C%20%22version%22%3A%201%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22bbox%22%3A%20%5B11.103310585021973%2C%2046.21703338623047%2C%2011.103310585021973%2C%2046.21703338623047%5D%2C%20%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B11.103310585021973%2C%2046.21703338623047%5D%2C%20%22type%22%3A%20%22Point%22%7D%2C%20%22id%22%3A%20%2211%22%2C%20%22properties%22%3A%20%7B%22amenity%22%3A%20%22drinking_water%22%2C%20%22changeset%22%3A%200%2C%20%22id%22%3A%202444602971%2C%20%22lat%22%3A%2046.21703338623047%2C%20%22lon%22%3A%2011.103310585021973%2C%20%22osm_type%22%3A%20%22node%22%2C%20%22tags%22%3A%20null%2C%20%22timestamp%22%3A%201378221842%2C%20%22version%22%3A%201%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22bbox%22%3A%20%5B11.070927619934082%2C%2046.22758483886719%2C%2011.070927619934082%2C%2046.22758483886719%5D%2C%20%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B11.070927619934082%2C%2046.22758483886719%5D%2C%20%22type%22%3A%20%22Point%22%7D%2C%20%22id%22%3A%20%2212%22%2C%20%22properties%22%3A%20%7B%22amenity%22%3A%20%22drinking_water%22%2C%20%22changeset%22%3A%200%2C%20%22id%22%3A%202453851300%2C%20%22lat%22%3A%2046.22758483886719%2C%20%22lon%22%3A%2011.070927619934082%2C%20%22osm_type%22%3A%20%22node%22%2C%20%22tags%22%3A%20null%2C%20%22timestamp%22%3A%201378889480%2C%20%22version%22%3A%201%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22bbox%22%3A%20%5B11.085515022277832%2C%2046.20384979248047%2C%2011.085515022277832%2C%2046.20384979248047%5D%2C%20%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B11.085515022277832%2C%2046.20384979248047%5D%2C%20%22type%22%3A%20%22Point%22%7D%2C%20%22id%22%3A%20%2213%22%2C%20%22properties%22%3A%20%7B%22amenity%22%3A%20%22drinking_water%22%2C%20%22changeset%22%3A%200%2C%20%22id%22%3A%204508560141%2C%20%22lat%22%3A%2046.20384979248047%2C%20%22lon%22%3A%2011.085515022277832%2C%20%22osm_type%22%3A%20%22node%22%2C%20%22tags%22%3A%20null%2C%20%22timestamp%22%3A%201479476539%2C%20%22version%22%3A%201%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22bbox%22%3A%20%5B11.096351623535156%2C%2046.2227668762207%2C%2011.096351623535156%2C%2046.2227668762207%5D%2C%20%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B11.096351623535156%2C%2046.2227668762207%5D%2C%20%22type%22%3A%20%22Point%22%7D%2C%20%22id%22%3A%20%2214%22%2C%20%22properties%22%3A%20%7B%22amenity%22%3A%20%22drinking_water%22%2C%20%22changeset%22%3A%200%2C%20%22id%22%3A%204740516453%2C%20%22lat%22%3A%2046.2227668762207%2C%20%22lon%22%3A%2011.096351623535156%2C%20%22osm_type%22%3A%20%22node%22%2C%20%22tags%22%3A%20null%2C%20%22timestamp%22%3A%201489783064%2C%20%22version%22%3A%201%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22bbox%22%3A%20%5B11.099811553955078%2C%2046.219905853271484%2C%2011.099811553955078%2C%2046.219905853271484%5D%2C%20%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B11.099811553955078%2C%2046.219905853271484%5D%2C%20%22type%22%3A%20%22Point%22%7D%2C%20%22id%22%3A%20%2215%22%2C%20%22properties%22%3A%20%7B%22amenity%22%3A%20%22drinking_water%22%2C%20%22changeset%22%3A%200%2C%20%22id%22%3A%204740516460%2C%20%22lat%22%3A%2046.219905853271484%2C%20%22lon%22%3A%2011.099811553955078%2C%20%22osm_type%22%3A%20%22node%22%2C%20%22tags%22%3A%20null%2C%20%22timestamp%22%3A%201489783064%2C%20%22version%22%3A%201%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22bbox%22%3A%20%5B11.08773136138916%2C%2046.21592330932617%2C%2011.08773136138916%2C%2046.21592330932617%5D%2C%20%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B11.08773136138916%2C%2046.21592330932617%5D%2C%20%22type%22%3A%20%22Point%22%7D%2C%20%22id%22%3A%20%2216%22%2C%20%22properties%22%3A%20%7B%22amenity%22%3A%20%22drinking_water%22%2C%20%22changeset%22%3A%200%2C%20%22id%22%3A%205040652012%2C%20%22lat%22%3A%2046.21592330932617%2C%20%22lon%22%3A%2011.08773136138916%2C%20%22osm_type%22%3A%20%22node%22%2C%20%22tags%22%3A%20null%2C%20%22timestamp%22%3A%201502961032%2C%20%22version%22%3A%201%7D%2C%20%22type%22%3A%20%22Feature%22%7D%5D%2C%20%22type%22%3A%20%22FeatureCollection%22%7D%29%3B%0A%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20geo_json_04446c227a3448468ac193e8b2d31da9.bindTooltip%28%0A%20%20%20%20function%28layer%29%7B%0A%20%20%20%20let%20div%20%3D%20L.DomUtil.create%28%27div%27%29%3B%0A%20%20%20%20%0A%20%20%20%20let%20handleObject%20%3D%20feature%3D%3Etypeof%28feature%29%3D%3D%27object%27%20%3F%20JSON.stringify%28feature%29%20%3A%20feature%3B%0A%20%20%20%20let%20fields%20%3D%20%5B%22version%22%2C%20%22timestamp%22%2C%20%22lon%22%2C%20%22tags%22%2C%20%22id%22%2C%20%22lat%22%2C%20%22changeset%22%2C%20%22amenity%22%2C%20%22osm_type%22%5D%3B%0A%20%20%20%20let%20aliases%20%3D%20%5B%22version%22%2C%20%22timestamp%22%2C%20%22lon%22%2C%20%22tags%22%2C%20%22id%22%2C%20%22lat%22%2C%20%22changeset%22%2C%20%22amenity%22%2C%20%22osm_type%22%5D%3B%0A%20%20%20%20let%20table%20%3D%20%27%3Ctable%3E%27%20%2B%0A%20%20%20%20%20%20%20%20String%28%0A%20%20%20%20%20%20%20%20fields.map%28%0A%20%20%20%20%20%20%20%20%28v%2Ci%29%3D%3E%0A%20%20%20%20%20%20%20%20%60%3Ctr%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Cth%3E%24%7Baliases%5Bi%5D%7D%3C/th%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Ctd%3E%24%7BhandleObject%28layer.feature.properties%5Bv%5D%29%7D%3C/td%3E%0A%20%20%20%20%20%20%20%20%3C/tr%3E%60%29.join%28%27%27%29%29%0A%20%20%20%20%2B%27%3C/table%3E%27%3B%0A%20%20%20%20div.innerHTML%3Dtable%3B%0A%20%20%20%20%0A%20%20%20%20return%20div%0A%20%20%20%20%7D%0A%20%20%20%20%2C%7B%22className%22%3A%20%22foliumtooltip%22%2C%20%22sticky%22%3A%20true%7D%29%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%0A%3C/script%3E onload="this.contentDocument.open();this.contentDocument.write(    decodeURIComponent(this.getAttribute('data-html')));this.contentDocument.close();" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



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




    (71, 10)




```python
benchs.columns
```




    Index(['version', 'timestamp', 'lon', 'tags', 'id', 'lat', 'changeset',
           'amenity', 'geometry', 'osm_type'],
          dtype='object')




```python
benchs.tags.unique()
```




    array(['{"backrest":"yes"}', '{"backrest":"no"}',
           '{"seats":"5","backrest":"yes"}', '{"seats":"4","backrest":"yes"}',
           None], dtype=object)



quick&dirty solution


```python
benchs_with_backrest = benchs[benchs.tags.str.find('"backrest":"yes"') > -1]
```


```python
benchs_with_backrest.tags.unique()
```




    array(['{"backrest":"yes"}', '{"seats":"5","backrest":"yes"}',
           '{"seats":"4","backrest":"yes"}'], dtype=object)




```python
benchs_with_backrest.shape
```




    (62, 10)




```python
print("the total of benchs tagged with the presence of a backrest is %s" % (str(benchs_with_backrest.shape[0])))
```

    the total of benchs tagged with the presence of a backrest is 62

