---
layout: default
title: Retrieving data from spatial database infrastructures
parent: Lessons
nav_order: 3
permalink: /lessons/retrieving_data_from_sdi
---
*Lesson 09 October 2020*
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---
# Retrieving data from spatial database infrastructures

## goals of the tutorial
- geocoding / reverse geocoding
- OGC services
- ESRI ArcGIS RestAPI

**based on the open data of:**
- [national repertoire of territorial data](https://geodati.gov.it/geoportale/)
- [national cartographic portal](http://www.pcn.minambiente.it/mattm/)
- [geoportal of Trentino](http://www.territorio.provincia.tn.it/portal/server.pt/community/portale_geocartografico_trentino/254)
- [italian civil protection department](http://www.protezionecivile.gov.it/)

### requirements
- python knowledge
- geopandas
- gis concepts


**status**

*looking for data*

---

# Geocoding / reverse geocoding

## Setup


```
pip install geopandas
```

    Collecting geopandas
    [?25l  Downloading https://files.pythonhosted.org/packages/f7/a4/e66aafbefcbb717813bf3a355c8c4fc3ed04ea1dd7feb2920f2f4f868921/geopandas-0.8.1-py2.py3-none-any.whl (962kB)
    [K     |████████████████████████████████| 972kB 2.8MB/s 
    [?25hRequirement already satisfied: pandas>=0.23.0 in /usr/local/lib/python3.6/dist-packages (from geopandas) (1.1.2)
    Requirement already satisfied: shapely in /usr/local/lib/python3.6/dist-packages (from geopandas) (1.7.1)
    Collecting pyproj>=2.2.0
    [?25l  Downloading https://files.pythonhosted.org/packages/e5/c3/071e080230ac4b6c64f1a2e2f9161c9737a2bc7b683d2c90b024825000c0/pyproj-2.6.1.post1-cp36-cp36m-manylinux2010_x86_64.whl (10.9MB)
    [K     |████████████████████████████████| 10.9MB 16.8MB/s 
    [?25hCollecting fiona
    [?25l  Downloading https://files.pythonhosted.org/packages/36/8b/e8b2c11bed5373c8e98edb85ce891b09aa1f4210fd451d0fb3696b7695a2/Fiona-1.8.17-cp36-cp36m-manylinux1_x86_64.whl (14.8MB)
    [K     |████████████████████████████████| 14.8MB 257kB/s 
    [?25hRequirement already satisfied: numpy>=1.15.4 in /usr/local/lib/python3.6/dist-packages (from pandas>=0.23.0->geopandas) (1.18.5)
    Requirement already satisfied: pytz>=2017.2 in /usr/local/lib/python3.6/dist-packages (from pandas>=0.23.0->geopandas) (2018.9)
    Requirement already satisfied: python-dateutil>=2.7.3 in /usr/local/lib/python3.6/dist-packages (from pandas>=0.23.0->geopandas) (2.8.1)
    Collecting click-plugins>=1.0
      Downloading https://files.pythonhosted.org/packages/e9/da/824b92d9942f4e472702488857914bdd50f73021efea15b4cad9aca8ecef/click_plugins-1.1.1-py2.py3-none-any.whl
    Requirement already satisfied: click<8,>=4.0 in /usr/local/lib/python3.6/dist-packages (from fiona->geopandas) (7.1.2)
    Collecting munch
      Downloading https://files.pythonhosted.org/packages/cc/ab/85d8da5c9a45e072301beb37ad7f833cd344e04c817d97e0cc75681d248f/munch-2.5.0-py2.py3-none-any.whl
    Collecting cligj>=0.5
      Downloading https://files.pythonhosted.org/packages/e4/be/30a58b4b0733850280d01f8bd132591b4668ed5c7046761098d665ac2174/cligj-0.5.0-py3-none-any.whl
    Requirement already satisfied: attrs>=17 in /usr/local/lib/python3.6/dist-packages (from fiona->geopandas) (20.2.0)
    Requirement already satisfied: six>=1.7 in /usr/local/lib/python3.6/dist-packages (from fiona->geopandas) (1.15.0)
    Installing collected packages: pyproj, click-plugins, munch, cligj, fiona, geopandas
    Successfully installed click-plugins-1.1.1 cligj-0.5.0 fiona-1.8.17 geopandas-0.8.1 munch-2.5.0 pyproj-2.6.1.post1



```
# only for visualization purpouse
!pip install git+https://github.com/python-visualization/folium
```

    Collecting git+https://github.com/python-visualization/folium
      Cloning https://github.com/python-visualization/folium to /tmp/pip-req-build-k6ylhcjy
      Running command git clone -q https://github.com/python-visualization/folium /tmp/pip-req-build-k6ylhcjy
    Requirement already satisfied: branca>=0.3.0 in /usr/local/lib/python3.6/dist-packages (from folium==0.11.0+20.gb70efc6) (0.4.1)
    Requirement already satisfied: jinja2>=2.9 in /usr/local/lib/python3.6/dist-packages (from folium==0.11.0+20.gb70efc6) (2.11.2)
    Requirement already satisfied: numpy in /usr/local/lib/python3.6/dist-packages (from folium==0.11.0+20.gb70efc6) (1.18.5)
    Requirement already satisfied: requests in /usr/local/lib/python3.6/dist-packages (from folium==0.11.0+20.gb70efc6) (2.23.0)
    Requirement already satisfied: MarkupSafe>=0.23 in /usr/local/lib/python3.6/dist-packages (from jinja2>=2.9->folium==0.11.0+20.gb70efc6) (1.1.1)
    Requirement already satisfied: urllib3!=1.25.0,!=1.25.1,<1.26,>=1.21.1 in /usr/local/lib/python3.6/dist-packages (from requests->folium==0.11.0+20.gb70efc6) (1.24.3)
    Requirement already satisfied: idna<3,>=2.5 in /usr/local/lib/python3.6/dist-packages (from requests->folium==0.11.0+20.gb70efc6) (2.10)
    Requirement already satisfied: chardet<4,>=3.0.2 in /usr/local/lib/python3.6/dist-packages (from requests->folium==0.11.0+20.gb70efc6) (3.0.4)
    Requirement already satisfied: certifi>=2017.4.17 in /usr/local/lib/python3.6/dist-packages (from requests->folium==0.11.0+20.gb70efc6) (2020.6.20)
    Building wheels for collected packages: folium
      Building wheel for folium (setup.py) ... [?25l[?25hdone
      Created wheel for folium: filename=folium-0.11.0+20.gb70efc6-py2.py3-none-any.whl size=97529 sha256=75c71d7148cea4759efa8db94265d317abfa62e40107aa85ee10dd7217ef583f
      Stored in directory: /tmp/pip-ephem-wheel-cache-poe5zey1/wheels/1e/e1/75/ecbc91fd5dd5d90befb0b533bf7492d38acffa033310731862
    Successfully built folium
    [31mERROR: datascience 0.10.6 has requirement folium==0.2.1, but you'll have folium 0.11.0+20.gb70efc6 which is incompatible.[0m
    Installing collected packages: folium
      Found existing installation: folium 0.8.3
        Uninstalling folium-0.8.3:
          Successfully uninstalled folium-0.8.3
    Successfully installed folium-0.11.0+20.gb70efc6



```
import geopandas as gpd
```

**GEOCODING service**

![](https://avatars2.githubusercontent.com/u/1385808?s=400&v=4")

- the geopandas module is based on [geopy](https://geopy.readthedocs.io/en/stable/)
- all the goecoders service are available [here](https://geopy.readthedocs.io/en/stable/#module-geopy.geocoders)


**NOTE**

Attention to the Rate Limit in Pandas<br/>
more info [here](https://geopy.readthedocs.io/en/stable/#usage-with-pandas)

**choose the right service**
<br/><br/>

![](https://raw.githubusercontent.com/napo/geospatial_course_unitn/master/images/getlonlat.png)

<br/><br/>
visit [getlon.lat](https://getlon.lat/)

## geocoding


```
cols = ['city']
names = [('Roma'),('Palermo'),('Trento'),('Genova'),('Bari'),('Trieste'),('Napoli'),('Cagliari'),('Messina'),('Lecce')]
cities = gpd.GeoDataFrame(names,columns=cols)
```


```
cities
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
      <th>city</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Roma</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Palermo</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Trento</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Genova</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Bari</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Trieste</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Napoli</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Cagliari</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Messina</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Lecce</td>
    </tr>
  </tbody>
</table>
</div>




```
geo_cities = gpd.tools.geocode(cities.city, provider="arcgis")
```


```
geo_cities
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
      <th>address</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>POINT (12.49565 41.90322)</td>
      <td>Roma</td>
    </tr>
    <tr>
      <th>1</th>
      <td>POINT (13.36112 38.12207)</td>
      <td>Palermo</td>
    </tr>
    <tr>
      <th>2</th>
      <td>POINT (11.11926 46.07005)</td>
      <td>Trento</td>
    </tr>
    <tr>
      <th>3</th>
      <td>POINT (8.93898 44.41039)</td>
      <td>Genova</td>
    </tr>
    <tr>
      <th>4</th>
      <td>POINT (16.86666 41.12587)</td>
      <td>Bari</td>
    </tr>
    <tr>
      <th>5</th>
      <td>POINT (13.77269 45.65757)</td>
      <td>Trieste</td>
    </tr>
    <tr>
      <th>6</th>
      <td>POINT (14.25226 40.84014)</td>
      <td>Napoli</td>
    </tr>
    <tr>
      <th>7</th>
      <td>POINT (9.11049 39.21454)</td>
      <td>Cagliari</td>
    </tr>
    <tr>
      <th>8</th>
      <td>POINT (15.55308 38.17837)</td>
      <td>Messina</td>
    </tr>
    <tr>
      <th>9</th>
      <td>POINT (18.16802 40.35796)</td>
      <td>Lecce</td>
    </tr>
  </tbody>
</table>
</div>




```
geo_cities.plot()
```




    <matplotlib.axes._subplots.AxesSubplot at 0x7fa5b326c518>




    
![png](03_Retrieving_data_from_spatial_database_infrastructures_files/03_Retrieving_data_from_spatial_database_infrastructures_13_1.png)
    


## reverse geocoding


```
from geopy.geocoders import Nominatim
```


```
geo_cities
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
      <th>address</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>POINT (12.49565 41.90322)</td>
      <td>Roma</td>
    </tr>
    <tr>
      <th>1</th>
      <td>POINT (13.36112 38.12207)</td>
      <td>Palermo</td>
    </tr>
    <tr>
      <th>2</th>
      <td>POINT (11.11926 46.07005)</td>
      <td>Trento</td>
    </tr>
    <tr>
      <th>3</th>
      <td>POINT (8.93898 44.41039)</td>
      <td>Genova</td>
    </tr>
    <tr>
      <th>4</th>
      <td>POINT (16.86666 41.12587)</td>
      <td>Bari</td>
    </tr>
    <tr>
      <th>5</th>
      <td>POINT (13.77269 45.65757)</td>
      <td>Trieste</td>
    </tr>
    <tr>
      <th>6</th>
      <td>POINT (14.25226 40.84014)</td>
      <td>Napoli</td>
    </tr>
    <tr>
      <th>7</th>
      <td>POINT (9.11049 39.21454)</td>
      <td>Cagliari</td>
    </tr>
    <tr>
      <th>8</th>
      <td>POINT (15.55308 38.17837)</td>
      <td>Messina</td>
    </tr>
    <tr>
      <th>9</th>
      <td>POINT (18.16802 40.35796)</td>
      <td>Lecce</td>
    </tr>
  </tbody>
</table>
</div>




```
point = geo_cities.geometry[2]
```


```
point.wkt
```




    'POINT (11.11926000000005 46.07005000000004)'




```
type(point.x)
```




    float




```
latlon = str(point.y) + "," + str(point.x)
```


```
geolocator = Nominatim(user_agent="Example for the course")
```

.. but better if use a user agent like

*Mozilla/5.0 (Linux; Android 10) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.75 Mobile Safari/537.36*

Eg

*geolocator = Nominatim(user_agent="Mozilla/5.0 (Linux; Android10) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.75 Mobile Safari/537.36")*




```
location = geolocator.reverse(latlon)
```

the raw method contains all the data available from the geocoder


```
location.raw
```




    {'address': {'city': 'Trento',
      'country': 'Italia',
      'country_code': 'it',
      'county': 'Provincia di Trento',
      'house_number': '15',
      'municipality': "Territorio Val d'Adige",
      'postcode': '38122',
      'road': 'Via Torre Vanga',
      'state': 'Trentino-Alto Adige/Südtirol',
      'suburb': 'Centro storico Trento',
      'tourism': 'Giovane Europa'},
     'boundingbox': ['46.0700951', '46.0703188', '11.119026', '11.1194422'],
     'display_name': "Giovane Europa, 15, Via Torre Vanga, Centro storico Trento, Trento, Territorio Val d'Adige, Provincia di Trento, Trentino-Alto Adige/Südtirol, 38122, Italia",
     'lat': '46.070178',
     'licence': 'Data © OpenStreetMap contributors, ODbL 1.0. https://osm.org/copyright',
     'lon': '11.119240793834841',
     'osm_id': 73293763,
     'osm_type': 'way',
     'place_id': 104813655}



## suggestion for a good geocoding
more details you add and more fortune you have to obtain a good result


```
q="Via Verdi, 26"
```


```
point = gpd.tools.geocode(q, provider="arcgis")
```


```
# import of folium to show the map
import folium
```


```
map_point = folium.Map([point.geometry.y,point.geometry.x], zoom_start=18)
folium.GeoJson(point.to_json()).add_to(map_point)
map_point
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe src="about:blank" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" data-html=PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgCiAgICAgICAgPHNjcmlwdD4KICAgICAgICAgICAgTF9OT19UT1VDSCA9IGZhbHNlOwogICAgICAgICAgICBMX0RJU0FCTEVfM0QgPSBmYWxzZTsKICAgICAgICA8L3NjcmlwdD4KICAgIAogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjYuMC9kaXN0L2xlYWZsZXQuanMiPjwvc2NyaXB0PgogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY29kZS5qcXVlcnkuY29tL2pxdWVyeS0xLjEyLjQubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9qcy9ib290c3RyYXAubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5qcyI+PC9zY3JpcHQ+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjYuMC9kaXN0L2xlYWZsZXQuY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vYm9vdHN0cmFwLzMuMi4wL2Nzcy9ib290c3RyYXAubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLXRoZW1lLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9mb250LWF3ZXNvbWUvNC42LjMvY3NzL2ZvbnQtYXdlc29tZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuanMuY2xvdWRmbGFyZS5jb20vYWpheC9saWJzL0xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLzIuMC4yL2xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvZ2gvcHl0aG9uLXZpc3VhbGl6YXRpb24vZm9saXVtL2ZvbGl1bS90ZW1wbGF0ZXMvbGVhZmxldC5hd2Vzb21lLnJvdGF0ZS5taW4uY3NzIi8+CiAgICA8c3R5bGU+aHRtbCwgYm9keSB7d2lkdGg6IDEwMCU7aGVpZ2h0OiAxMDAlO21hcmdpbjogMDtwYWRkaW5nOiAwO308L3N0eWxlPgogICAgPHN0eWxlPiNtYXAge3Bvc2l0aW9uOmFic29sdXRlO3RvcDowO2JvdHRvbTowO3JpZ2h0OjA7bGVmdDowO308L3N0eWxlPgogICAgCiAgICAgICAgICAgIDxtZXRhIG5hbWU9InZpZXdwb3J0IiBjb250ZW50PSJ3aWR0aD1kZXZpY2Utd2lkdGgsCiAgICAgICAgICAgICAgICBpbml0aWFsLXNjYWxlPTEuMCwgbWF4aW11bS1zY2FsZT0xLjAsIHVzZXItc2NhbGFibGU9bm8iIC8+CiAgICAgICAgICAgIDxzdHlsZT4KICAgICAgICAgICAgICAgICNtYXBfOWY1NDcwNTRhYzY3NGI2MDkwMWM4NjJjN2VhMjkxYmUgewogICAgICAgICAgICAgICAgICAgIHBvc2l0aW9uOiByZWxhdGl2ZTsKICAgICAgICAgICAgICAgICAgICB3aWR0aDogMTAwLjAlOwogICAgICAgICAgICAgICAgICAgIGhlaWdodDogMTAwLjAlOwogICAgICAgICAgICAgICAgICAgIGxlZnQ6IDAuMCU7CiAgICAgICAgICAgICAgICAgICAgdG9wOiAwLjAlOwogICAgICAgICAgICAgICAgfQogICAgICAgICAgICA8L3N0eWxlPgogICAgICAgIAo8L2hlYWQ+Cjxib2R5PiAgICAKICAgIAogICAgICAgICAgICA8ZGl2IGNsYXNzPSJmb2xpdW0tbWFwIiBpZD0ibWFwXzlmNTQ3MDU0YWM2NzRiNjA5MDFjODYyYzdlYTI5MWJlIiA+PC9kaXY+CiAgICAgICAgCjwvYm9keT4KPHNjcmlwdD4gICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcF85ZjU0NzA1NGFjNjc0YjYwOTAxYzg2MmM3ZWEyOTFiZSA9IEwubWFwKAogICAgICAgICAgICAgICAgIm1hcF85ZjU0NzA1NGFjNjc0YjYwOTAxYzg2MmM3ZWEyOTFiZSIsCiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgY2VudGVyOiBbMjUuODM5MTEwMDAwMDAwMDYyLCAtODAuMTg0Njc5OTk5OTk5OTZdLAogICAgICAgICAgICAgICAgICAgIGNyczogTC5DUlMuRVBTRzM4NTcsCiAgICAgICAgICAgICAgICAgICAgem9vbTogMTgsCiAgICAgICAgICAgICAgICAgICAgem9vbUNvbnRyb2w6IHRydWUsCiAgICAgICAgICAgICAgICAgICAgcHJlZmVyQ2FudmFzOiBmYWxzZSwKICAgICAgICAgICAgICAgIH0KICAgICAgICAgICAgKTsKCiAgICAgICAgICAgIAoKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgdGlsZV9sYXllcl8xNTI3ZmUxNjUzMjg0OTA5OTRhMGNjMTBmYzgxYWYyMyA9IEwudGlsZUxheWVyKAogICAgICAgICAgICAgICAgImh0dHBzOi8ve3N9LnRpbGUub3BlbnN0cmVldG1hcC5vcmcve3p9L3t4fS97eX0ucG5nIiwKICAgICAgICAgICAgICAgIHsiYXR0cmlidXRpb24iOiAiRGF0YSBieSBcdTAwMjZjb3B5OyBcdTAwM2NhIGhyZWY9XCJodHRwOi8vb3BlbnN0cmVldG1hcC5vcmdcIlx1MDAzZU9wZW5TdHJlZXRNYXBcdTAwM2MvYVx1MDAzZSwgdW5kZXIgXHUwMDNjYSBocmVmPVwiaHR0cDovL3d3dy5vcGVuc3RyZWV0bWFwLm9yZy9jb3B5cmlnaHRcIlx1MDAzZU9EYkxcdTAwM2MvYVx1MDAzZS4iLCAiZGV0ZWN0UmV0aW5hIjogZmFsc2UsICJtYXhOYXRpdmVab29tIjogMTgsICJtYXhab29tIjogMTgsICJtaW5ab29tIjogMCwgIm5vV3JhcCI6IGZhbHNlLCAib3BhY2l0eSI6IDEsICJzdWJkb21haW5zIjogImFiYyIsICJ0bXMiOiBmYWxzZX0KICAgICAgICAgICAgKS5hZGRUbyhtYXBfOWY1NDcwNTRhYzY3NGI2MDkwMWM4NjJjN2VhMjkxYmUpOwogICAgICAgIAogICAgCiAgICAgICAgZnVuY3Rpb24gZ2VvX2pzb25fNDFhZjNjZTJjZDk5NGZhZGFlMzZjMjc2ZTBiOGZkNDZfb25FYWNoRmVhdHVyZShmZWF0dXJlLCBsYXllcikgewogICAgICAgICAgICBsYXllci5vbih7CiAgICAgICAgICAgIH0pOwogICAgICAgIH07CiAgICAgICAgdmFyIGdlb19qc29uXzQxYWYzY2UyY2Q5OTRmYWRhZTM2YzI3NmUwYjhmZDQ2ID0gTC5nZW9Kc29uKG51bGwsIHsKICAgICAgICAgICAgICAgIG9uRWFjaEZlYXR1cmU6IGdlb19qc29uXzQxYWYzY2UyY2Q5OTRmYWRhZTM2YzI3NmUwYjhmZDQ2X29uRWFjaEZlYXR1cmUsCiAgICAgICAgICAgIAogICAgICAgIH0pOwoKICAgICAgICBmdW5jdGlvbiBnZW9fanNvbl80MWFmM2NlMmNkOTk0ZmFkYWUzNmMyNzZlMGI4ZmQ0Nl9hZGQgKGRhdGEpIHsKICAgICAgICAgICAgZ2VvX2pzb25fNDFhZjNjZTJjZDk5NGZhZGFlMzZjMjc2ZTBiOGZkNDYKICAgICAgICAgICAgICAgIC5hZGREYXRhKGRhdGEpCiAgICAgICAgICAgICAgICAuYWRkVG8obWFwXzlmNTQ3MDU0YWM2NzRiNjA5MDFjODYyYzdlYTI5MWJlKTsKICAgICAgICB9CiAgICAgICAgICAgIGdlb19qc29uXzQxYWYzY2UyY2Q5OTRmYWRhZTM2YzI3NmUwYjhmZDQ2X2FkZCh7ImZlYXR1cmVzIjogW3siZ2VvbWV0cnkiOiB7ImNvb3JkaW5hdGVzIjogWy04MC4xODQ2Nzk5OTk5OTk5NiwgMjUuODM5MTEwMDAwMDAwMDYyXSwgInR5cGUiOiAiUG9pbnQifSwgImlkIjogIjAiLCAicHJvcGVydGllcyI6IHsiYWRkcmVzcyI6ICJWaWEgVmVyZGkifSwgInR5cGUiOiAiRmVhdHVyZSJ9XSwgInR5cGUiOiAiRmVhdHVyZUNvbGxlY3Rpb24ifSk7CgogICAgICAgIAo8L3NjcmlwdD4= onload="this.contentDocument.open();this.contentDocument.write(atob(this.getAttribute('data-html')));this.contentDocument.close();" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



add details like city and State


```
q="Via Verdi, 26, Trento, Italia"
```


```
point = gpd.tools.geocode(q, provider="arcgis")
```


```
point
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
      <th>address</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>POINT (11.11966 46.06665)</td>
      <td>Via Giuseppe Verdi 26, 38122, Trento</td>
    </tr>
  </tbody>
</table>
</div>




```
map_point = folium.Map([point.geometry.y,point.geometry.x], zoom_start=18)
folium.GeoJson(point.to_json()).add_to(map_point)
map_point
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe src="about:blank" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" data-html=PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgCiAgICAgICAgPHNjcmlwdD4KICAgICAgICAgICAgTF9OT19UT1VDSCA9IGZhbHNlOwogICAgICAgICAgICBMX0RJU0FCTEVfM0QgPSBmYWxzZTsKICAgICAgICA8L3NjcmlwdD4KICAgIAogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjYuMC9kaXN0L2xlYWZsZXQuanMiPjwvc2NyaXB0PgogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY29kZS5qcXVlcnkuY29tL2pxdWVyeS0xLjEyLjQubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9qcy9ib290c3RyYXAubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5qcyI+PC9zY3JpcHQ+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjYuMC9kaXN0L2xlYWZsZXQuY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vYm9vdHN0cmFwLzMuMi4wL2Nzcy9ib290c3RyYXAubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLXRoZW1lLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9mb250LWF3ZXNvbWUvNC42LjMvY3NzL2ZvbnQtYXdlc29tZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuanMuY2xvdWRmbGFyZS5jb20vYWpheC9saWJzL0xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLzIuMC4yL2xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvZ2gvcHl0aG9uLXZpc3VhbGl6YXRpb24vZm9saXVtL2ZvbGl1bS90ZW1wbGF0ZXMvbGVhZmxldC5hd2Vzb21lLnJvdGF0ZS5taW4uY3NzIi8+CiAgICA8c3R5bGU+aHRtbCwgYm9keSB7d2lkdGg6IDEwMCU7aGVpZ2h0OiAxMDAlO21hcmdpbjogMDtwYWRkaW5nOiAwO308L3N0eWxlPgogICAgPHN0eWxlPiNtYXAge3Bvc2l0aW9uOmFic29sdXRlO3RvcDowO2JvdHRvbTowO3JpZ2h0OjA7bGVmdDowO308L3N0eWxlPgogICAgCiAgICAgICAgICAgIDxtZXRhIG5hbWU9InZpZXdwb3J0IiBjb250ZW50PSJ3aWR0aD1kZXZpY2Utd2lkdGgsCiAgICAgICAgICAgICAgICBpbml0aWFsLXNjYWxlPTEuMCwgbWF4aW11bS1zY2FsZT0xLjAsIHVzZXItc2NhbGFibGU9bm8iIC8+CiAgICAgICAgICAgIDxzdHlsZT4KICAgICAgICAgICAgICAgICNtYXBfMjVmNjU0NDNkZTU4NGZmZmE5NjA1YTEzOGM0NGMzNzAgewogICAgICAgICAgICAgICAgICAgIHBvc2l0aW9uOiByZWxhdGl2ZTsKICAgICAgICAgICAgICAgICAgICB3aWR0aDogMTAwLjAlOwogICAgICAgICAgICAgICAgICAgIGhlaWdodDogMTAwLjAlOwogICAgICAgICAgICAgICAgICAgIGxlZnQ6IDAuMCU7CiAgICAgICAgICAgICAgICAgICAgdG9wOiAwLjAlOwogICAgICAgICAgICAgICAgfQogICAgICAgICAgICA8L3N0eWxlPgogICAgICAgIAo8L2hlYWQ+Cjxib2R5PiAgICAKICAgIAogICAgICAgICAgICA8ZGl2IGNsYXNzPSJmb2xpdW0tbWFwIiBpZD0ibWFwXzI1ZjY1NDQzZGU1ODRmZmZhOTYwNWExMzhjNDRjMzcwIiA+PC9kaXY+CiAgICAgICAgCjwvYm9keT4KPHNjcmlwdD4gICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcF8yNWY2NTQ0M2RlNTg0ZmZmYTk2MDVhMTM4YzQ0YzM3MCA9IEwubWFwKAogICAgICAgICAgICAgICAgIm1hcF8yNWY2NTQ0M2RlNTg0ZmZmYTk2MDVhMTM4YzQ0YzM3MCIsCiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgY2VudGVyOiBbNDYuMDY2NjUwMDA2MDE5ODEsIDExLjExOTY1OTk4NTkzNzAzNF0sCiAgICAgICAgICAgICAgICAgICAgY3JzOiBMLkNSUy5FUFNHMzg1NywKICAgICAgICAgICAgICAgICAgICB6b29tOiAxOCwKICAgICAgICAgICAgICAgICAgICB6b29tQ29udHJvbDogdHJ1ZSwKICAgICAgICAgICAgICAgICAgICBwcmVmZXJDYW52YXM6IGZhbHNlLAogICAgICAgICAgICAgICAgfQogICAgICAgICAgICApOwoKICAgICAgICAgICAgCgogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciB0aWxlX2xheWVyXzBiYjQ1NjYzNDI3MzRkMTBiNWI4MTJjZTY3ZTFkNDNkID0gTC50aWxlTGF5ZXIoCiAgICAgICAgICAgICAgICAiaHR0cHM6Ly97c30udGlsZS5vcGVuc3RyZWV0bWFwLm9yZy97en0ve3h9L3t5fS5wbmciLAogICAgICAgICAgICAgICAgeyJhdHRyaWJ1dGlvbiI6ICJEYXRhIGJ5IFx1MDAyNmNvcHk7IFx1MDAzY2EgaHJlZj1cImh0dHA6Ly9vcGVuc3RyZWV0bWFwLm9yZ1wiXHUwMDNlT3BlblN0cmVldE1hcFx1MDAzYy9hXHUwMDNlLCB1bmRlciBcdTAwM2NhIGhyZWY9XCJodHRwOi8vd3d3Lm9wZW5zdHJlZXRtYXAub3JnL2NvcHlyaWdodFwiXHUwMDNlT0RiTFx1MDAzYy9hXHUwMDNlLiIsICJkZXRlY3RSZXRpbmEiOiBmYWxzZSwgIm1heE5hdGl2ZVpvb20iOiAxOCwgIm1heFpvb20iOiAxOCwgIm1pblpvb20iOiAwLCAibm9XcmFwIjogZmFsc2UsICJvcGFjaXR5IjogMSwgInN1YmRvbWFpbnMiOiAiYWJjIiwgInRtcyI6IGZhbHNlfQogICAgICAgICAgICApLmFkZFRvKG1hcF8yNWY2NTQ0M2RlNTg0ZmZmYTk2MDVhMTM4YzQ0YzM3MCk7CiAgICAgICAgCiAgICAKICAgICAgICBmdW5jdGlvbiBnZW9fanNvbl9kODMzZWFiMzU2MGM0NWUyYTgyMTZhZGU4MDczZGE1ZF9vbkVhY2hGZWF0dXJlKGZlYXR1cmUsIGxheWVyKSB7CiAgICAgICAgICAgIGxheWVyLm9uKHsKICAgICAgICAgICAgfSk7CiAgICAgICAgfTsKICAgICAgICB2YXIgZ2VvX2pzb25fZDgzM2VhYjM1NjBjNDVlMmE4MjE2YWRlODA3M2RhNWQgPSBMLmdlb0pzb24obnVsbCwgewogICAgICAgICAgICAgICAgb25FYWNoRmVhdHVyZTogZ2VvX2pzb25fZDgzM2VhYjM1NjBjNDVlMmE4MjE2YWRlODA3M2RhNWRfb25FYWNoRmVhdHVyZSwKICAgICAgICAgICAgCiAgICAgICAgfSk7CgogICAgICAgIGZ1bmN0aW9uIGdlb19qc29uX2Q4MzNlYWIzNTYwYzQ1ZTJhODIxNmFkZTgwNzNkYTVkX2FkZCAoZGF0YSkgewogICAgICAgICAgICBnZW9fanNvbl9kODMzZWFiMzU2MGM0NWUyYTgyMTZhZGU4MDczZGE1ZAogICAgICAgICAgICAgICAgLmFkZERhdGEoZGF0YSkKICAgICAgICAgICAgICAgIC5hZGRUbyhtYXBfMjVmNjU0NDNkZTU4NGZmZmE5NjA1YTEzOGM0NGMzNzApOwogICAgICAgIH0KICAgICAgICAgICAgZ2VvX2pzb25fZDgzM2VhYjM1NjBjNDVlMmE4MjE2YWRlODA3M2RhNWRfYWRkKHsiZmVhdHVyZXMiOiBbeyJnZW9tZXRyeSI6IHsiY29vcmRpbmF0ZXMiOiBbMTEuMTE5NjU5OTg1OTM3MDM0LCA0Ni4wNjY2NTAwMDYwMTk4MV0sICJ0eXBlIjogIlBvaW50In0sICJpZCI6ICIwIiwgInByb3BlcnRpZXMiOiB7ImFkZHJlc3MiOiAiVmlhIEdpdXNlcHBlIFZlcmRpIDI2LCAzODEyMiwgVHJlbnRvIn0sICJ0eXBlIjogIkZlYXR1cmUifV0sICJ0eXBlIjogIkZlYXR1cmVDb2xsZWN0aW9uIn0pOwoKICAgICAgICAKPC9zY3JpcHQ+ onload="this.contentDocument.open();this.contentDocument.write(atob(this.getAttribute('data-html')));this.contentDocument.close();" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



Try a different geocoder


```
point_nominatim = gpd.tools.geocode(q, provider="Nominatim",user_agent="Example for the course")
```


```
point_nominatim
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
      <th>address</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>POINT (10.76813 46.31650)</td>
      <td>Via Verdi, Ognano, Stavel, Pellizzano, Comunit...</td>
    </tr>
  </tbody>
</table>
</div>




```
q="Via Giuseppe Verdi, 26, Trento, Italia"
```


```
point_nominatim = gpd.tools.geocode(q, provider="Nominatim",user_agent="Example for the course")
```


```
point_nominatim
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
      <th>address</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>POINT (11.11971 46.06641)</td>
      <td>Dipartimento di Sociologia e Ricerca Sociale, ...</td>
    </tr>
  </tbody>
</table>
</div>




```
map_point = folium.Map([point_nominatim.geometry.y,point.geometry.x], zoom_start=18)
folium.GeoJson(point_nominatim.to_json()).add_to(map_point)
map_point
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe src="about:blank" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" data-html=PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgCiAgICAgICAgPHNjcmlwdD4KICAgICAgICAgICAgTF9OT19UT1VDSCA9IGZhbHNlOwogICAgICAgICAgICBMX0RJU0FCTEVfM0QgPSBmYWxzZTsKICAgICAgICA8L3NjcmlwdD4KICAgIAogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjYuMC9kaXN0L2xlYWZsZXQuanMiPjwvc2NyaXB0PgogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY29kZS5qcXVlcnkuY29tL2pxdWVyeS0xLjEyLjQubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9qcy9ib290c3RyYXAubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5qcyI+PC9zY3JpcHQ+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjYuMC9kaXN0L2xlYWZsZXQuY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vYm9vdHN0cmFwLzMuMi4wL2Nzcy9ib290c3RyYXAubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLXRoZW1lLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9mb250LWF3ZXNvbWUvNC42LjMvY3NzL2ZvbnQtYXdlc29tZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuanMuY2xvdWRmbGFyZS5jb20vYWpheC9saWJzL0xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLzIuMC4yL2xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvZ2gvcHl0aG9uLXZpc3VhbGl6YXRpb24vZm9saXVtL2ZvbGl1bS90ZW1wbGF0ZXMvbGVhZmxldC5hd2Vzb21lLnJvdGF0ZS5taW4uY3NzIi8+CiAgICA8c3R5bGU+aHRtbCwgYm9keSB7d2lkdGg6IDEwMCU7aGVpZ2h0OiAxMDAlO21hcmdpbjogMDtwYWRkaW5nOiAwO308L3N0eWxlPgogICAgPHN0eWxlPiNtYXAge3Bvc2l0aW9uOmFic29sdXRlO3RvcDowO2JvdHRvbTowO3JpZ2h0OjA7bGVmdDowO308L3N0eWxlPgogICAgCiAgICAgICAgICAgIDxtZXRhIG5hbWU9InZpZXdwb3J0IiBjb250ZW50PSJ3aWR0aD1kZXZpY2Utd2lkdGgsCiAgICAgICAgICAgICAgICBpbml0aWFsLXNjYWxlPTEuMCwgbWF4aW11bS1zY2FsZT0xLjAsIHVzZXItc2NhbGFibGU9bm8iIC8+CiAgICAgICAgICAgIDxzdHlsZT4KICAgICAgICAgICAgICAgICNtYXBfMzI0ZTRmZTAwN2NmNGM1NWJlMjc1MDY3ZWU3N2ExNjcgewogICAgICAgICAgICAgICAgICAgIHBvc2l0aW9uOiByZWxhdGl2ZTsKICAgICAgICAgICAgICAgICAgICB3aWR0aDogMTAwLjAlOwogICAgICAgICAgICAgICAgICAgIGhlaWdodDogMTAwLjAlOwogICAgICAgICAgICAgICAgICAgIGxlZnQ6IDAuMCU7CiAgICAgICAgICAgICAgICAgICAgdG9wOiAwLjAlOwogICAgICAgICAgICAgICAgfQogICAgICAgICAgICA8L3N0eWxlPgogICAgICAgIAo8L2hlYWQ+Cjxib2R5PiAgICAKICAgIAogICAgICAgICAgICA8ZGl2IGNsYXNzPSJmb2xpdW0tbWFwIiBpZD0ibWFwXzMyNGU0ZmUwMDdjZjRjNTViZTI3NTA2N2VlNzdhMTY3IiA+PC9kaXY+CiAgICAgICAgCjwvYm9keT4KPHNjcmlwdD4gICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcF8zMjRlNGZlMDA3Y2Y0YzU1YmUyNzUwNjdlZTc3YTE2NyA9IEwubWFwKAogICAgICAgICAgICAgICAgIm1hcF8zMjRlNGZlMDA3Y2Y0YzU1YmUyNzUwNjdlZTc3YTE2NyIsCiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgY2VudGVyOiBbNDYuMDY2NDEzNDk5OTk5OTk2LCAxMS4xMTk2NTk5ODU5MzcwMzRdLAogICAgICAgICAgICAgICAgICAgIGNyczogTC5DUlMuRVBTRzM4NTcsCiAgICAgICAgICAgICAgICAgICAgem9vbTogMTgsCiAgICAgICAgICAgICAgICAgICAgem9vbUNvbnRyb2w6IHRydWUsCiAgICAgICAgICAgICAgICAgICAgcHJlZmVyQ2FudmFzOiBmYWxzZSwKICAgICAgICAgICAgICAgIH0KICAgICAgICAgICAgKTsKCiAgICAgICAgICAgIAoKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgdGlsZV9sYXllcl81YTVjYmNlZmRjMGI0NmM5YTU3NjUzY2M2MzljMmQyZiA9IEwudGlsZUxheWVyKAogICAgICAgICAgICAgICAgImh0dHBzOi8ve3N9LnRpbGUub3BlbnN0cmVldG1hcC5vcmcve3p9L3t4fS97eX0ucG5nIiwKICAgICAgICAgICAgICAgIHsiYXR0cmlidXRpb24iOiAiRGF0YSBieSBcdTAwMjZjb3B5OyBcdTAwM2NhIGhyZWY9XCJodHRwOi8vb3BlbnN0cmVldG1hcC5vcmdcIlx1MDAzZU9wZW5TdHJlZXRNYXBcdTAwM2MvYVx1MDAzZSwgdW5kZXIgXHUwMDNjYSBocmVmPVwiaHR0cDovL3d3dy5vcGVuc3RyZWV0bWFwLm9yZy9jb3B5cmlnaHRcIlx1MDAzZU9EYkxcdTAwM2MvYVx1MDAzZS4iLCAiZGV0ZWN0UmV0aW5hIjogZmFsc2UsICJtYXhOYXRpdmVab29tIjogMTgsICJtYXhab29tIjogMTgsICJtaW5ab29tIjogMCwgIm5vV3JhcCI6IGZhbHNlLCAib3BhY2l0eSI6IDEsICJzdWJkb21haW5zIjogImFiYyIsICJ0bXMiOiBmYWxzZX0KICAgICAgICAgICAgKS5hZGRUbyhtYXBfMzI0ZTRmZTAwN2NmNGM1NWJlMjc1MDY3ZWU3N2ExNjcpOwogICAgICAgIAogICAgCiAgICAgICAgZnVuY3Rpb24gZ2VvX2pzb25fYzBiMjk0YWQ0N2QzNDE4OWE0YjkwYmY4NzgxZDA1YmNfb25FYWNoRmVhdHVyZShmZWF0dXJlLCBsYXllcikgewogICAgICAgICAgICBsYXllci5vbih7CiAgICAgICAgICAgIH0pOwogICAgICAgIH07CiAgICAgICAgdmFyIGdlb19qc29uX2MwYjI5NGFkNDdkMzQxODlhNGI5MGJmODc4MWQwNWJjID0gTC5nZW9Kc29uKG51bGwsIHsKICAgICAgICAgICAgICAgIG9uRWFjaEZlYXR1cmU6IGdlb19qc29uX2MwYjI5NGFkNDdkMzQxODlhNGI5MGJmODc4MWQwNWJjX29uRWFjaEZlYXR1cmUsCiAgICAgICAgICAgIAogICAgICAgIH0pOwoKICAgICAgICBmdW5jdGlvbiBnZW9fanNvbl9jMGIyOTRhZDQ3ZDM0MTg5YTRiOTBiZjg3ODFkMDViY19hZGQgKGRhdGEpIHsKICAgICAgICAgICAgZ2VvX2pzb25fYzBiMjk0YWQ0N2QzNDE4OWE0YjkwYmY4NzgxZDA1YmMKICAgICAgICAgICAgICAgIC5hZGREYXRhKGRhdGEpCiAgICAgICAgICAgICAgICAuYWRkVG8obWFwXzMyNGU0ZmUwMDdjZjRjNTViZTI3NTA2N2VlNzdhMTY3KTsKICAgICAgICB9CiAgICAgICAgICAgIGdlb19qc29uX2MwYjI5NGFkNDdkMzQxODlhNGI5MGJmODc4MWQwNWJjX2FkZCh7ImZlYXR1cmVzIjogW3siZ2VvbWV0cnkiOiB7ImNvb3JkaW5hdGVzIjogWzExLjExOTcwNTY0NDY4MDY0NiwgNDYuMDY2NDEzNDk5OTk5OTk2XSwgInR5cGUiOiAiUG9pbnQifSwgImlkIjogIjAiLCAicHJvcGVydGllcyI6IHsiYWRkcmVzcyI6ICJEaXBhcnRpbWVudG8gZGkgU29jaW9sb2dpYSBlIFJpY2VyY2EgU29jaWFsZSwgMjYsIFZpYSBHaXVzZXBwZSBWZXJkaSwgQ2VudHJvIHN0b3JpY28gVHJlbnRvLCBUcmVudG8sIFRlcnJpdG9yaW8gVmFsIGRcdTAwMjdBZGlnZSwgUHJvdmluY2lhIGRpIFRyZW50bywgVHJlbnRpbm8tQWx0byBBZGlnZS9TXHUwMGZjZHRpcm9sLCAzODEyMiwgSXRhbGlhIn0sICJ0eXBlIjogIkZlYXR1cmUifV0sICJ0eXBlIjogIkZlYXR1cmVDb2xsZWN0aW9uIn0pOwoKICAgICAgICAKPC9zY3JpcHQ+ onload="this.contentDocument.open();this.contentDocument.write(atob(this.getAttribute('data-html')));this.contentDocument.close();" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



calculate the difference between the two points


```
distance = point.to_crs('epsg:32632').geometry.distance(point_nominatim.geometry.to_crs('epsg:32632')).values[0]
```


```
distance
```




    26.522713658370346



## **Summary**

- geocoding is, first of all, an NLP problem
- geocoding services try to normalize the query by identifying the object you are looking for
- the more information of a geographic hierarchical order the better the geocoder results
- it is difficult to have an always updated address database
- many geocoders, where they do not find the value, return a value inferred from the interpopulation
- accuracy depends on what you are looking for
- a geocoder always tries to give an answer<br/>&nbsp;an excellent geocoder also returns the value of the precision estimate


# OGC Services
![](https://raw.githubusercontent.com/napo/geospatial_course_unitn/master/images/ogc_services.png)

---

## Catalog Service for the Web

![](https://raw.githubusercontent.com/napo/geospatial_course_unitn/master/images/geocatalog_pat.png)

---

https://siat.provincia.tn.it/geonetwork/srv/eng/catalog.search




## Setup
https://geopython.github.io/OWSLib/


```
pip install owslib
```

    Collecting owslib
    [?25l  Downloading https://files.pythonhosted.org/packages/c4/6a/428d73506f6f5281408b518249b90d7c96a1394c6d954a2069cbd5a4ac39/OWSLib-0.20.0-py2.py3-none-any.whl (197kB)
    [K     |████████████████████████████████| 204kB 2.8MB/s 
    [?25hRequirement already satisfied: pyproj>=2 in /usr/local/lib/python3.6/dist-packages (from owslib) (2.6.1.post1)
    Requirement already satisfied: requests>=1.0 in /usr/local/lib/python3.6/dist-packages (from owslib) (2.23.0)
    Requirement already satisfied: python-dateutil>=1.5 in /usr/local/lib/python3.6/dist-packages (from owslib) (2.8.1)
    Requirement already satisfied: pyyaml in /usr/local/lib/python3.6/dist-packages (from owslib) (3.13)
    Requirement already satisfied: pytz in /usr/local/lib/python3.6/dist-packages (from owslib) (2018.9)
    Requirement already satisfied: chardet<4,>=3.0.2 in /usr/local/lib/python3.6/dist-packages (from requests>=1.0->owslib) (3.0.4)
    Requirement already satisfied: idna<3,>=2.5 in /usr/local/lib/python3.6/dist-packages (from requests>=1.0->owslib) (2.10)
    Requirement already satisfied: certifi>=2017.4.17 in /usr/local/lib/python3.6/dist-packages (from requests>=1.0->owslib) (2020.6.20)
    Requirement already satisfied: urllib3!=1.25.0,!=1.25.1,<1.26,>=1.21.1 in /usr/local/lib/python3.6/dist-packages (from requests>=1.0->owslib) (1.24.3)
    Requirement already satisfied: six>=1.5 in /usr/local/lib/python3.6/dist-packages (from python-dateutil>=1.5->owslib) (1.15.0)
    Installing collected packages: owslib
    Successfully installed owslib-0.20.0



```
from owslib.csw import CatalogueServiceWeb
```


```
csw = CatalogueServiceWeb("http://geodati.gov.it/RNDT/csw")
```


```
csw.service
```




    'CSW'




```
[op.name for op in csw.operations]
```




    ['GetCapabilities',
     'DescribeRecord',
     'GetRecords',
     'GetRecordById',
     'Transaction',
     'Harvest']




```
from owslib.fes import PropertyIsLike, BBox
```

fields to query

|                |                            |
|---             |---                         |
|field           | description                |
|*dc:title*      | title of the dataset       |
|*dc:description*| description of the dataset |
|*dc:subject*    | subject of the dataset     |
|*csw:AnyText*   | in all the fields          |

*PropertyIsLike* means that you can use the *LIKE* syntax of SQL

Eg. *%rento* => each word that ends with 'rento'


```
trento_query = PropertyIsLike('csw:AnyText', 'Trento')
```


```
csw.getrecords2(constraints=[trento_query],maxrecords=100)
```


```
csw.results
```




    {'matches': 95, 'nextrecord': 0, 'returned': 95}




```
for rec in csw.records:
  print(rec + " - " + csw.records[rec].title)
```

    agea:00129:20090724:090446 - Ortofotocarta Trento 2003
    agea:00377:20090911:093144 - Ortofotocarta Trento 2008
    agea:00128:20090724:085449 - Ortofotocarta Trento 1997
    p_TN:377793f1-1094-4e81-810e-403897418b23 - Limite Provinciale della Provincia Autonoma di Trento
    c_l378:toponomastica - Stradario, civici e toponimi del Comune di Trento
    c_l378:ortofoto2009 - Ortofoto 2009
    p_TN:71403f02-0b4e-4f02-8475-1321c04e184c - PFM - vocazione alla produzione legnosa dei boschi - (VOCPRODUZIONE)
    p_TN:c5c29caa-850d-43b5-8a42-4db73cf593f0 - PFM - vocazione naturalistica - (VOCNAT)
    p_TN:2131bcc4-1a2b-46ff-a546-8c22aab0371a - Carta Tecnica Provinciale - CTP 2015
    p_TN:bec36f9a-0566-44aa-bdb5-1c065a3a9a28 - Carta Topografica Generale - CTP 1998
    p_TN:aec4a171-ad51-49d6-ad9b-42934a2c5d43 - Toponomastica dei Centri Abitati (scala 1:10000)
    p_TN:e5d7975d-074b-4f19-a7e7-17274c9e6aa3 - Carta Tecnica Provinciale - CTP 2017
    p_TN:f93c200f-1088-4121-94fc-4e94d1a88c8b - Carta Tecnica Provinciale - CTP 2013
    p_TN:1e93bc40-a91f-49ba-891d-de3a4627b86e - Limite Comprensoriale
    p_TN:6d80fcd4-1dac-40c6-8fee-d7a831d35959 - Carta Topografica Generale - CTP 00
    p_TN:cfa55552-a520-48d3-824c-2941b3c4d98d - Carta Topografica Generale - CTP 1998 singole sezioni - livello altimetria
    p_TN:9905ac49-d9cc-4317-b44b-a8450cc37b20 - Carta Topografica Generale - CTP 1998 singole sezioni - livello planimetria
    p_TN:8131f518-c808-4a30-912f-e34892aa0f1a - Carta Topografica Generale - CTP 00 singole sezioni livello altimetria
    p_TN:aa677704-2c39-4edd-ba64-b69282bff778 - Carta Topografica Generale - CTP 00 singole sezioni livello edifici
    p_TN:6a7806d8-63fa-4445-b009-9412c99d8fd2 - Edifici P.A.T. 3D
    p_TN:fbbc1e07-0b8e-46c9-b961-a02d8bebb217 - Ortofoto PAT 1973 in scala di grigi
    p_TN:c11e62c6-2e67-4614-827f-8de8621ac9ee - Carta Topografica Generale - CTP 1998 singole sezioni - livello topografia
    p_TN:2989b73f-f243-4fac-b1d0-6464863a7d1d - LiDAR DTM - Modello Digitale del Terreno - PAT 2006 / 2009
    p_TN:06d1bcf9-6fa0-4e5e-af34-82366110720d - LiDAR DSM - Modello Digitale delle Superfici - PAT 2006 / 2009
    ispra_rm:Meta_Geo_SV000056_RN - ITA_Trento_topo25k
    p_TN:92ca98b4-f881-41eb-a229-c16dff316489 - Piano Urbanistico Provinciale - PUP formato vettoriale
    p_TN:280bb887-dff4-446c-965c-43ae077a107d - Quadro d'unione sezioni 1:10.000 taglio ED50
    p_TN:8ad3e252-0e1d-42b3-8cb5-abe9dbccc281 - Carta Topografica Generale - CTP 00 singole sezioni livello planimetria
    p_TN:d28bdcfc-f271-4598-84ca-318ebcaf8dfb - Carta Topografica Generale - CTP 00 singole sezioni livello topografia
    p_TN:628a228e-d13c-41c8-803a-272883e3931e - Quadro d'unione sezioni 1:10.000 della 1° edizione della Carta Topografica Generale taglio ED50
    p_TN:c0518563-b53e-44da-870e-8719ef4a5215 - Piano Urbanistico Provinciale - PUP formato raster
    p_TN:335f1205-9d6e-4e75-b4cf-35b9385e4fae - LiDAR Padergnone 2007
    p_TN:40091f90-ed0c-43f7-97b5-2099544a2708 - LiDAR Paneveggio 2007
    p_TN:bb3efd89-143f-4fcf-8251-4a4b601e6d21 - LiDAR Ravina 2007
    p_TN:ef82ba6c-e9b3-451d-9835-0f74b9f66650 - LiDAR Val di Sella 2007
    p_TN:5c773de5-e3a7-4024-ab4c-ea313d3ba901 - LiDAR - Soleggiamenti DTM - PAT 2006 / 2009
    p_TN:1f08d6de-85b0-41f3-b2b7-b3d04a5771d7 - LiDAR - Soleggiamenti DSM - PAT 2006 / 2009
    p_TN:abea13d9-c73a-46f1-b997-9c41267d06c5 - Aree di sintesi geologica
    p_TN:ea66f90c-426f-497c-9044-f870ce7a7f4b - Cave dismesse
    p_TN:fb95e841-df80-41b5-af08-8a31896533cc - Iperspettrale Paneveggio 2007
    p_TN:33db32ef-7ce4-43e3-bbf5-11440f4c9757 - Iperspettrale Padergnone 2007
    p_TN:a16182cb-16cf-4c8b-b74f-0493a8c7f445 - Iperspettrale Val di Sella  2007
    p_TN:62bc4940-4713-4677-814a-5d2077180e6d - Iperspettrale Ravina 2007
    p_TN:e506ab2b-5eb6-452a-8ebb-102ffc4454b1 - Ortofoto Paneveggio 2007 RGB
    p_TN:054e1af3-2dd8-4496-8b83-261cccc2674c - Ortofoto Ravina 2007 RGB
    p_TN:44cd8a88-ba6e-4299-8075-df9996401006 - Ortofoto Padergnone 2007 RGB
    p_TN:85fb88cb-80c8-47f3-9e50-4142655cf461 - LiDAR dato grezzo - Padergnone 2007
    p_TN:b7db2c50-b463-4d30-a40f-b52ba92887ff - LiDAR DSM - Modello Digitale delle Superfici - Padergnone 2007
    p_TN:0837bfca-dff2-4770-801f-235607f42b72 - LiDAR DTM - Modello Digitale del Terreno - Padergnone 2007
    p_TN:25aa8b8f-271d-406c-8f7e-ba224ec844fe - LiDAR DTM - Modello Digitale del Terreno - Ravina 2007
    p_TN:0f944cb5-770f-44ff-aad2-36a47f24bc94 - LiDAR DSM - Modello Digitale delle Superfici - Ravina 2007
    p_TN:36069574-aef9-4855-ae3e-b8445cbd1319 - LiDAR dato grezzo - Ravina 2007
    p_TN:4e838148-4631-4c4b-9758-798e450ca407 - LiDAR DTM - Modello Digitale del Terreno - Paneveggio 2007
    p_TN:3ddfc8e1-1362-4915-ab77-8c31cee7ac82 - LiDAR DSM - Modello Digitale delle Superfici - Paneveggio 2007
    p_TN:225321fc-0e6b-45d3-8dd0-58fd602bdf64 - LiDAR dato grezzo - Paneveggio 2007
    p_TN:013ef530-ee77-49d2-8f95-035b27ab1f0a - Ortofoto Val di Sella 2007 RGB
    p_TN:f0252045-0c4b-4ce9-8c31-5db78c3a63bd - LiDAR PAT 2006 / 2009
    p_TN:3203e1b3-5d73-47d8-b04e-9ab27c8538cf - LiDAR dato grezzo - PAT 2006 / 2009
    p_TN:b0cdad84-3c7d-4422-8f7a-7cb762af3da1 - LiDAR DSM - Modello Digitale delle Superfici - Val di Sella 2007
    p_TN:152ad4d1-3263-4ae9-bce9-1265f019a784 - LiDAR DTM - Modello Digitale del Terreno - Val di Sella 2007
    p_TN:b388eaa7-4e27-4970-9b28-b541fe480bf0 - LiDAR dato grezzo - Val di Sella 2007
    p_TN:f2e88f1b-05d9-4942-93ee-857a0a9e1f0b - Ortofoto PAT 2015 RGB
    p_TN:441525c1-a100-405c-b6fc-1f0c319bacbb - Grotte
    p_TN:2eacecb2-d78c-460e-9a4c-47b80a9f105d - Cartografia catastale numerica
    p_TN:3dba066a-297b-401c-962a-e957371010a1 - Opera di consolidamento
    p_TN:df88a5af-1f79-47f6-ada8-c6cf70ed42e6 - Tombinatura
    p_TN:2dc4efe4-1441-4968-9d8a-e03a0742fe35 - Briglia di consolidamento
    p_TN:13919c0d-1415-42f8-8842-b5b5e49f5e63 - Opera spondale
    p_TN:29ff287d-9306-40eb-ae55-1b163e669b51 - Briglia di trattenuta
    p_TN:0f0d2a2b-65f6-4f34-b5e0-aef7319abdbf - Cunettone
    p_TN:a72a2c44-0094-43a7-8bc6-8f9c20e77c0f - Rafforzamento arginale
    p_TN:71f65e11-5763-43c9-892c-a3d954be4fcb - Intervento di ingegneria naturalistica
    p_TN:ff03f2d6-6c7c-4cf6-ab11-42b2c8963cc7 - Rivestimento in alveo
    p_TN:366667ac-f7d2-42cf-a1a5-7450b3e44d6a - Piazza di deposito
    p_TN:d24bfa28-3451-41c7-a1ac-e0a2fe4af6d9 - Rilevato arginale
    p_TN:27c14db4-5fde-478f-a14b-a92ac3ff7602 - Vallo tomo
    p_TN:37139ae6-4118-4b65-9471-9d57a1f84762 - Repellente
    p_TN:405acb52-c061-4e21-9924-2283c8cc7175 - Drenaggio
    p_TN:cee93e4c-9be2-47cb-8dcd-385aa1999d0d - AREA_PAT_DISTRETTI_DLGS152_2006
    p_TN:30193185-3477-4185-988a-35cddc575473 - Sondaggi
    p_TN:58604ed2-ac1d-4f78-a00c-514fd3562c51 - Limite Comunità di valle
    adbpo:PDGPO2015GWCI:20151217 - Distretto Po - Direttiva Acque – Delimitazione dei corpi idrici sotterranei 2015
    PCM:000094:20130308:143000 - Zone di allertamento per il rischio idrogeologico e idraulico - Italia (RNDT - Dataset)
    p_TN:94725247-c839-470a-b98c-9fd412bc1a7f - IFF2007 nel BACINO DEL TORRENTE AVISIO
    p_TN:4d40b9b3-f25b-4e27-b940-69194424b125 - IFF2007 nel BACINO del FIUME CHIESE
    p_TN:afd60b6f-2805-4f71-98bc-0d6df7fbe1d2 - IFF2007 nel BACINO DEL TORRENTE FERSINA
    p_TN:798f13c9-1e22-40da-b64c-74fe22b7b32f - IFF2007 nel BACINO DEL FIUME ADIGE
    p_TN:bb9e5a5c-36bd-46a0-b7b5-67613b178777 - IFF2007 nel BACINO DEL FIUME BRENTA
    p_TN:acee82fa-ee1d-460a-b1b1-a5753fde30f6 - IFF2007 nel BACINO DEL TORRENTE ASTICO
    p_TN:ef2c038c-c6e0-4707-8419-a6d3eb72bf7f - IFF2007 nel BACINO DEI TORRENTI VANOI E CISMON
    p_TN:8e6d85a5-bfec-480b-a4e4-94aac259388e - IFF2007 nel BACINO DEL FIUME SARCA
    p_TN:4f8da929-aac5-47d3-b4f4-22c8a174b947 - IFF2007 nel BACINO DEL TORRENTE NOCE
    adbpo:DistrettoDR2018:20181017 - Distretto Po - Delimitazione Regioni nel Distretto 2018
    r_piemon:18895034-1059-47cd-95de-c858070d3aa3 - Inventario Regionale delle Emissioni in Atmosfera (IREA)
    adbpo:DistrettoD2018:20181017 - Distretto Po - Delimitazione Distretto 2018


```
p_TN:441525c1-a100-405c-b6fc-1f0c319bacbb => Grotte
```

*grotte* means *caves* in italian language


```
s="p_TN:441525c1-a100-405c-b6fc-1f0c319bacbb" #caves
```


```
record = csw.records[s]
```


```
record.title
```




    'Grotte'




```
record.abstract
```




    "Catasto delle grotte naturali della Provincia autonoma di Trento.L'istituzione del catasto delle grotte e delle aree carsiche della Provincia di Trento è stata prevista dalla Legge provinciale n. 37 del 31/10/1983 (Protezione del patrimonio mineralogico, paleontologico, paletnologico, speleologico e carsico); l'articolo 14 della citata legge demanda alla Giunta provinciale l'emanazione delle norme attinenti all'impianto, al funzionamento, all'aggiornamento e all'accesso al catasto stesso.Il catasto delle grotte del Trentino è stato ufficialmente attivato in data 14 marzo 2008 tramite specifica delibera della Giunta Provinciale."




```
for reference in record.references:
  print(reference['scheme'])
  print(reference['url'])
```

    urn:x-esri:specification:ServiceType:ArcIMS:Metadata:Server
    https://siat.provincia.tn.it/IDT/vector/public/p_tn_441525c1-a100-405c-b6fc-1f0c319bacbb.zip
    urn:x-esri:specification:ServiceType:ArcIMS:Metadata:Document
    https://geodati.gov.it/geoportalRNDTPA/csw?getxml=%7BF3EB7695-7DCD-49B9-8450-E19B2A86F69D%7D



```
caves = gpd.read_file('https://siat.provincia.tn.it/IDT/vector/public/p_tn_441525c1-a100-405c-b6fc-1f0c319bacbb.zip')
```


```
caves.head(5)
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
      <th>objectid</th>
      <th>id_grotta</th>
      <th>nome</th>
      <th>n_vt</th>
      <th>nome_local</th>
      <th>comune</th>
      <th>localita</th>
      <th>valle</th>
      <th>monte</th>
      <th>dominio_ca</th>
      <th>area_cars_</th>
      <th>cod_pat</th>
      <th>unita_geol</th>
      <th>eta_o_pian</th>
      <th>id_sezione</th>
      <th>sezione_ct</th>
      <th>ediz_ctp</th>
      <th>coord_x</th>
      <th>coord_y</th>
      <th>quota_ingr</th>
      <th>svil_spaz</th>
      <th>svil_plan</th>
      <th>disl_tot</th>
      <th>disl_pos</th>
      <th>disl_neg</th>
      <th>geosito</th>
      <th>invariante</th>
      <th>annotazion</th>
      <th>agg_riliev</th>
      <th>notizie</th>
      <th>storia</th>
      <th>paleontolo</th>
      <th>paletnolog</th>
      <th>meteorolog</th>
      <th>biologia</th>
      <th>idrologia</th>
      <th>mineralogi</th>
      <th>geologia</th>
      <th>scheda_arm</th>
      <th>bibliograf</th>
      <th>accesso</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2</td>
      <td>107.0</td>
      <td>Antro ai Murazzi</td>
      <td>107.0</td>
      <td>Bus dela Vecia</td>
      <td>BESENELLO</td>
      <td>Murazzi - Dosso della Soga</td>
      <td>Valle dell'Adige</td>
      <td>Vigolana</td>
      <td>M.MARZOLA, VIGOLANA</td>
      <td>None</td>
      <td>DPR</td>
      <td>DOLOMIA PRINCIPALE</td>
      <td>NORICO-RETICO</td>
      <td>81020.0</td>
      <td>ALDENO</td>
      <td>1983</td>
      <td>663703.0</td>
      <td>5090951.0</td>
      <td>320.0</td>
      <td>10.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Si</td>
      <td>Si</td>
      <td>None</td>
      <td>29/07/98 registrato da Roberto Frisinghelli</td>
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
      <td>POINT (663703.000 5090951.000)</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3</td>
      <td>108.0</td>
      <td>Caverna Besenello</td>
      <td>108.0</td>
      <td>None</td>
      <td>BESENELLO</td>
      <td>Crocetta</td>
      <td>Val della Scaletta</td>
      <td>Vigolana</td>
      <td>M.MARZOLA, VIGOLANA</td>
      <td>None</td>
      <td>DPR</td>
      <td>DOLOMIA PRINCIPALE</td>
      <td>NORICO-RETICO</td>
      <td>81060.0</td>
      <td>CALLIANO</td>
      <td>1983</td>
      <td>664946.0</td>
      <td>5090337.0</td>
      <td>590.0</td>
      <td>63.0</td>
      <td>34.0</td>
      <td>20.0</td>
      <td>20.0</td>
      <td>NaN</td>
      <td>No</td>
      <td>No</td>
      <td>None</td>
      <td>19/06/98 registrato da Roberto Frisinghelli</td>
      <td>Indicata e segnata sulle carte IGM e CTR</td>
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
      <td>POINT (664946.000 5090337.000)</td>
    </tr>
    <tr>
      <th>2</th>
      <td>4</td>
      <td>110.0</td>
      <td>Bus de l' Avel</td>
      <td>110.0</td>
      <td>None</td>
      <td>CONCEI</td>
      <td>Gombie</td>
      <td>None</td>
      <td>None</td>
      <td>V.DI LEDRO, V.DI CONCEI, TREMALZO</td>
      <td>None</td>
      <td>ZUU1</td>
      <td>b'CALCARE DI ZU - Membro del Grost\xe8'</td>
      <td>NORICO?-RETICO</td>
      <td>80050.0</td>
      <td>CONCEI</td>
      <td>1983</td>
      <td>632924.0</td>
      <td>5085983.0</td>
      <td>1310.0</td>
      <td>56.0</td>
      <td>NaN</td>
      <td>34.0</td>
      <td>0.0</td>
      <td>34.0</td>
      <td>No</td>
      <td>No</td>
      <td>None</td>
      <td>16/09/98 registrato da Roberto Frisinghelli</td>
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
      <td>POINT (632924.000 5085983.000)</td>
    </tr>
    <tr>
      <th>3</th>
      <td>5</td>
      <td>120.0</td>
      <td>La Camerona</td>
      <td>120.0</td>
      <td>None</td>
      <td>FIAVE'</td>
      <td>Ballino</td>
      <td>None</td>
      <td>Monte Misone</td>
      <td>V.DI LEDRO, V.DI CONCEI, TREMALZO</td>
      <td>None</td>
      <td>MIS</td>
      <td>GRUPPO DEI CALCARI GRIGI - CALCARE DEL MISONE</td>
      <td>SINEMURIANO-PLIENSBACHIANO</td>
      <td>80020.0</td>
      <td>BALLINO</td>
      <td>1983</td>
      <td>640808.0</td>
      <td>5091808.0</td>
      <td>950.0</td>
      <td>90.0</td>
      <td>NaN</td>
      <td>35.0</td>
      <td>35.0</td>
      <td>NaN</td>
      <td>Si</td>
      <td>Si</td>
      <td>None</td>
      <td>29/07/98 registrato da Roberto Frisinghelli</td>
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
      <td>POINT (640808.000 5091808.000)</td>
    </tr>
    <tr>
      <th>4</th>
      <td>6</td>
      <td>125.0</td>
      <td>Grotta Cesare Battisti</td>
      <td>125.0</td>
      <td>None</td>
      <td>ZAMBANA</td>
      <td>None</td>
      <td>None</td>
      <td>Paganella</td>
      <td>M.PAGANELLA, M.GAZZA</td>
      <td>AREA CARSICA DELLA PAGANELLA</td>
      <td>RTZ</td>
      <td>Formazione di Rotzo</td>
      <td>Sinemuriano - Pliensbachiano</td>
      <td>60050.0</td>
      <td>PAGANELLA</td>
      <td>1983</td>
      <td>658146.0</td>
      <td>5112357.0</td>
      <td>1880.0</td>
      <td>2342.0</td>
      <td>NaN</td>
      <td>204.0</td>
      <td>22.0</td>
      <td>182.0</td>
      <td>Si</td>
      <td>Si</td>
      <td>None</td>
      <td>29/07/98 registrato da Roberto Frisinghelli</td>
      <td>Individuata come SIC-Sito di importanza comuni...</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>b"Mosna, Ezio - L'esplorazione speleologica de...</td>
      <td>None</td>
      <td>POINT (658146.000 5112357.000)</td>
    </tr>
  </tbody>
</table>
</div>




```
caves.crs
```




    <Projected CRS: EPSG:25832>
    Name: ETRS89 / UTM zone 32N
    Axis Info [cartesian]:
    - E[east]: Easting (metre)
    - N[north]: Northing (metre)
    Area of Use:
    - name: Europe - 6°E to 12°E and ETRS89 by country
    - bounds: (6.0, 38.76, 12.0, 83.92)
    Coordinate Operation:
    - name: UTM zone 32N
    - method: Transverse Mercator
    Datum: European Terrestrial Reference System 1989
    - Ellipsoid: GRS 1980
    - Prime Meridian: Greenwich




```
caves.plot()
```




    <matplotlib.axes._subplots.AxesSubplot at 0x7fa5b141a2e8>




    
![png](03_Retrieving_data_from_spatial_database_infrastructures_files/03_Retrieving_data_from_spatial_database_infrastructures_71_1.png)
    


we can search by bounding box

![](https://raw.githubusercontent.com/napo/geospatial_course_unitn/master/images/bbox_klokantech.png)

https://boundingbox.klokantech.com/


```
csw = CatalogueServiceWeb("http://www.pcn.minambiente.it/geoportal/csw")
```


```
bbox_query = BBox([10.770062,45.866839,11.022748,46.083374])
```


```
csw.getrecords2(constraints=[bbox_query],maxrecords=100)
```


```
csw.results
```




    {'matches': 117, 'nextrecord': 101, 'returned': 100}




```
for rec in csw.records:
  print(rec + " - " + csw.records[rec].title)
```

    m_amte:299FN3:f451a878-27b3-43be-bc19-6eeb1362392a - DSM FIRST LiDAR con risoluzione a terra 1 metro - Regione Veneto
    m_amte:299FN3:554245de-46c5-4ff9-a214-43eb794baad8 - DSM LAST LiDAR con risoluzione a terra 1 metro - Regione Veneto
    m_amte:299FN3:6ad216c2-614b-4eb6-94d8-deefee09ba48 - DTM LiDAR con risoluzione a terra 1 metro - Regione Veneto
    m_amte:299FN3:046e201a-1864-4fb0-cd36-1cf75fbac550 - INTENSITY LiDAR con risoluzione a terra 1 metro - Regione Veneto
    m_amte:299FN3:84a24090-2239-4e54-8764-8fa0a2e239a5 - DSM FIRST LiDAR con risoluzione a terra 1 metro - Regione Lombardia
    m_amte:299FN3:0a0b4709-a1c2-47c1-8749-5816e25d9c4f - DSM LAST LiDAR con risoluzione a terra 1 metro - Regione Lombardia
    m_amte:299FN3:e02c7579-8031-4563-fca5-e2f1523fa7d4 - DTM LiDAR con risoluzione a terra 1 metro - Regione Lombardia
    m_amte:299FN3:4b87ddce-2c7d-4c27-e100-caf0557c477c - INTENSITY LiDAR con risoluzione a terra 1 metro - Regione Lombardia
    m_amte:299FN3:cbd1bd6d-4f0a-4707-aa42-fa97cf5a9a6e - Prodotti LiDAR - Regione Lombardia
    m_amte:299FN3:4dca3621-4baa-49ee-90b5-33ca42a4bb4e - Ortofoto in bianco e nero anni 1994-1996 - Regioni zona WGS84-UTM32
    m_amte:299FN3:21d09438-3f4c-4642-a9ea-064a60488b88 - Ortofoto a colori anno 2006 - Regioni zona WGS84-UTM32
    m_amte:299FN3:a1197b44-5b11-4a49-def8-ae45e2c5e6ae - Ortofoto in bianco e nero anni 1988-1989 - Regioni zona WGS84-UTM32
    m_amte:299FN3:2637cb18-6d0c-4508-bd6e-64804e85a550 - Ortofoto a colori anno 2000 - Regioni zona WGS84-UTM32
    m_amte:299FN3:55d73e34-780e-45a3-ba77-6292b307a597 - Numeri civici dei capoluoghi di provincia
    m_amte:299FN3:24d2c4ab-0036-4be0-c803-f1eded63c3fb - Edificato dei capoluoghi di provincia
    m_amte:299FN3:bd78d3e0-4f64-4c80-bd33-4ea6e17b8a29 - Date ortofoto in bianco e nero anni 1994-1996
    m_amte:8HCH2C:6db40564-355c-4a66-c240-bb6173ad3bd8 - Zone di protezione speciale (ZPS)
    m_amte:299FN3:0da844bd-387e-4c31-cfea-2b928189ad70 - Progetto incendi - cartografia anti incendi boschivi nei Parchi Nazionali
    m_amte:299FN3:8345a338-17c9-444b-c598-fe508d1cb45e - Scuole pubbliche per l'infanzia, primarie e secondarie sul territorio nazionale
    m_amte:299FN3:2b8ed8fd-6ae1-4594-cd25-405c36569685 - Carta fitoclimatica d'Italia
    m_amte:299FN3:1d117feb-517b-4688-aa75-da586566aa36 - Quadro di unione delle tavole LiDAR - Grigliato 2x2
    m_amte:299FN3:f9f310e3-2a7d-4b04-f3f1-f827e8c67339 - Quadro di unione delle tavole LiDAR - Grigliato 1x1
    m_amte:8HCH2C:c52b8f8b-1bee-4b73-984a-7c37bcb9b575 - Quadro di unione COSMO SKY-MED Descending - PST 2013
    m_amte:8HCH2C:21907666-0be6-4f4f-d14b-fda0433d3340 - Quadro di unione COSMO SKY-MED Ascending - PST 2013
    m_amte:8HCH2C:5dcb0f26-5a09-490b-db47-bc913dbd31cd - Datafile immagini SAR COSMO SKY-MED Descending - PST 2013
    m_amte:8HCH2C:379bf691-58cd-468d-bff6-c5b69eabdd66 - Datafile immagini SAR COSMO SKY-MED Ascending - PST 2013
    m_amte:000006:20181030:091345 - UnitOfManagement_IT_20181025
    m_amte:8HCH2C:8cb1c5c8-5a7d-4be0-df35-5f6e4f7de63f - Rete Natura 2000 (SIC/ZSC e ZPS)
    m_amte:299FN3:02ddd5df-f91a-4767-86dd-1993df9c65c5 - AIB - Uso suolo degli incendi nei Parchi Nazionali rilevati con GPS
    m_amte:299FN3:cc27f7d2-b344-49f1-8ce5-e92a0c390e64 - AIB - Incendi rilevati con GPS nei Parchi Nazionali
    m_amte:299FN3:ea869861-4075-4cdb-a461-0934f3a77fbd - Ortofoto a colori anno 2000 - Regioni zona WGS84-UTM33
    m_amte:299FN3:acd4f94e-cd9b-40df-df31-cb366017dcf9 - Date ortofoto a colori  AGEA anni 2009-2012
    m_amte:299FN3:89db50f2-60f8-49b7-8bbc-7c6360030a04 - Date ortofoto a colori anno 2006
    m_amte:299FN3:66d70689-184b-4163-fd39-dc9fa896db98 - Date ortofoto a colori anno 2000
    m_amte:299FN3:80437fc1-bbed-41f9-84dc-ee091b9080e4 - Date ortofoto in bianco e nero anni 1988-1989
    m_amte:299FN3:c5dd5d00-555e-493f-8380-080dc164633c - Cartografia di base IGM 25.000 - Regioni zona WGS84-UTM32
    m_amte:299FN3:d36c3fd9-6c45-4ada-f497-a2f827f575cf - Cartografia di base IGM 25.000 - Regioni zona WGS84-UTM33
    m_amte:299FN3:3c22e603-6c67-41e1-bb03-ae29c1b86c03 - Cartografia di base  IGM 100.000 - Regioni zona WGS84-UTM33
    m_amte:299FN3:cfa9e368-0773-4b4c-8bd6-65b7ca5d4dbc - Cartografia di base Atlante DeAgostini
    m_amte:299FN3:62b0784a-3eac-47e8-c85a-24db8e0b7ea8 - Cartografia di base  IGM 100.000 - Regioni zona WGS84-UTM32
    m_amte:299FN3:68dd5028-f5a0-42dd-8b6e-52b6223b0d3b - Servizio di conversione di coordinate del Geoportale Nazionale
    m_amte:299FN3:06c67978-18c8-4da7-ff26-443d4f700c2d - Elenco ufficiale aree protette (EUAP)
    m_amte:8HCH2C:9e8cb9de-4b74-4473-ed5e-e70ac71b2699 - Progetto coste - Macrodati nazionali sulla variazione della linea di costa 1960-2012
    m_amte:299FN3:67888d8a-8b76-412d-d14c-01e1278f1f2f - AIB - Habitat a rischio
    m_amte:299FN3:7336cd71-c311-44f1-d0ed-ddf2a327f496 - Datafile immagini SAR ENVISAT Descending
    m_amte:8HCH2C:7880f1c1-cdab-4fd4-9a18-c4878a68c21d - Corine Land Cover anno 2012
    m_amte:299FN3:542eea43-1163-408d-8879-18b4896eb038 - Unità amministrative 2011
    m_amte:299FN3:d0439890-8379-45ac-f70b-3d7f11f20ce9 - Bacini idrografici principali
    m_amte:299FN3:bced7a41-2ca7-4df5-c142-e95aafab0f8c - Catalogo frane - Deformazioni Gravitative Profonde di Versante (DGPV)
    m_amte:299FN3:2b7dfce8-abec-4179-cf69-091bc0886c84 - Catalogo frane - Frane lineari
    m_amte:299FN3:a39c7888-254e-4a3e-f295-07dfdaa0dfe2 - Catalogo frane - Aree
    m_amte:299FN3:ca7b9c7f-194d-4416-cb45-af9ddfc8c64a - Catalogo frane - Punti Identificativi Fenomeni Franosi (PIFF)
    m_amte:299FN3:f648aeba-b80b-40a1-d608-c45c1db2b48e - Catalogo frane - Frane poligonali
    m_amte:299FN3:e0c36031-607f-4abb-9d1d-b9a1b60a2c6d - Zonazione sismogenetica ZS9
    m_amte:299FN3:068c1a67-d08e-4e7c-d4f3-6d54c849a584 - Sezioni di censimento - ISTAT 1991
    m_amte:299FN3:bbea45fd-e678-4d17-f3f4-ddbbc6c3e60c - Zonazione sismogenetica ZS9  - Limiti
    m_amte:299FN3:e090c6d1-2538-46be-ed51-667a16c4b1d2 - Sezioni censimento 2001
    m_amte:299FN3:d3b47a56-367c-4824-d306-beef43897a2a - Quadro di unione ERS Ascending
    m_amte:299FN3:d958e301-9f33-4bed-c853-a9faacf5e5b1 - Classificazione sismica dei comuni italiani al 2010
    m_amte:299FN3:48ce2a5f-d6d6-4ebf-bdfb-a9dfe7f10375 - AIB - Zonizzazione dei Parchi Nazionali
    m_amte:299FN3:600cc327-f2e4-4932-c00e-60d45d0a090d - Catalogo frane - Direzioni
    m_amte:299FN3:2d16fb84-4e88-4480-ae1f-0cf03c068233 - AIB - Vegetazione nei Parchi Nazionali
    m_amte:299FN3:559ff4c0-454e-4a05-fc97-807690e6f515 - Bacini idrografici secondari
    m_amte:299FN3:99622692-1e7a-452b-86d7-3ae4fe96c385 - Zone umide di importanza internazionale (RAMSAR)
    m_amte:299FN3:3291bd68-7059-4c5a-8074-f33989c46e7d - Carta ecopedologica d'Italia
    m_amte:299FN3:bd587278-b36b-4c88-ded1-cdff9039267c - Indice di variazione delle velocità medie - Confronto ERS ENVISAT Ascending
    m_amte:299FN3:3da3c812-275f-499a-aedc-255c71fff349 - Quadro di unione ENVISAT Ascending
    m_amte:299FN3:8f1a1539-7c36-4e44-f8c1-23601278e0b2 - Corine Land Cover anno 2000
    m_amte:299FN3:72904e47-8e94-4649-ac83-b3574e1f4906 - Corine Land Cover anno 2006 - IV Livello
    m_amte:299FN3:3481d729-552c-4fcc-afd1-b4218399899c - Progetto coste - Unità fisiografiche della costa
    m_amte:299FN3:ccc7ab91-39dc-465a-ce11-ec426d9c54ec - Quadro di unione ENVISAT Descending
    m_amte:299FN3:cf673376-8386-4c25-ab76-7c17ae9200af - AIB - Rischio incendi generale
    m_amte:8HCH2C:23ecf6da-e29a-4984-9e6d-ac5bafa226ef - Progetto coste – Principali aree portuali (1960-2012)
    m_amte:8HCH2C:c6528121-1ec2-4d21-c9e2-a4b19945f232 - Progetto coste - Beni a rischio di erosione: centri abitati, strade statali, provinciali, comunali e ferrovie
    m_amte:8HCH2C:b80fb69d-d0bd-4cf0-d303-a81602361f40 - Progetto coste - Principali variazioni della linea di costa (1994-2012)
    m_amte:8HCH2C:5b1a30dd-2f66-481e-bbc0-2b82df52179a - Progetto coste - Principali variazioni della linea di costa (1960-2012)
    m_amte:8HCH2C:f41b9e22-fc24-429f-a3f8-0ded080a81b6 - Progetto coste - Principali variazioni della linea di costa (1960-1994)
    m_amte:299FN3:12ce8c99-26b4-406f-a3a2-86c59f4f1b1c - Datafile immagini SAR ERS Descending
    m_amte:299FN3:8c8fad83-bf36-4ae3-80b8-1c7640746126 - Datafile immagini SAR ERS Ascending
    m_amte:299FN3:21ec84cc-a816-40f5-ae6c-5859e2607859 - Datafile immagini SAR ENVISAT Ascending
    m_amte:299FN3:be73b4cb-1e7c-4d0d-fffd-4b9adda1ba49 - Batimetria dei mari
    m_amte:299FN3:50813389-af97-438e-bdb4-cc21eec9f2fd - Quadro di unione ERS Descending
    m_amte:299FN3:39a4a57d-4b44-4fc3-fc4b-e84623530a7d - Corine Land Cover anno 2000 -  IV Livello
    m_amte:299FN3:c43e8c5c-d732-4b52-d71a-808bf0269a74 - Classificazione sismica dei comuni italiani al 2012
    m_amte:299FN3:61c65d38-f9ec-476c-a9ee-79e590a3919a - Linea di costa aggiornata al 2009
    m_amte:299FN3:55d240eb-ad6a-412e-bdb7-972c95c59a9f - Corine Land Cover anno 1990
    m_amte:299FN3:77136f53-cde9-4b3d-b19c-64aafe36e1dd - Carta geologica d'Italia
    m_amte:299FN3:2cea010f-a357-4eec-ebb0-1993fe200844 - Pericolosità sismica di riferimento a passo 0,05 gradi
    m_amte:299FN3:0f4393f6-d2d7-484c-9290-32058182bf1a - IUTI - Inventario dell'uso delle terre d'Italia
    m_amte:299FN3:e1d9994a-d8eb-4fa1-9eef-43e08c8ecee2 - Pericolosità sismica di riferimento a passo 0,02 gradi
    m_amte:299FN3:4c90991f-285c-4964-a633-bacc63bb281f - Carta geolitologica d'Italia
    m_amte:299FN3:5237cf1e-4d49-486c-a688-152bc4473508 - Laghi e altri specchi d'acqua
    m_amte:299FN3:e7e8b6c8-a109-4fb6-b3bc-861740e00a82 - AIB - Rischio incendi periodo invernale
    m_amte:299FN3:408000e2-8a33-4acd-a05a-b3bf77dd9924 - AIB - Rischio incendi periodo estivo
    m_amte:299FN3:6a3bf67a-470f-43d1-80c9-79bb7cb36b27 - AIB - Incendi 2001-2005 (test da satellite)
    m_amte:299FN3:468e02b1-d783-400f-fd83-b9ae2b6a47ca - AIB - Zone rosse prioritarie
    m_amte:299FN3:a31da965-7b90-4cbe-eac2-32a687bb2509 - Aree importanti per l'avifauna (IBA - Important Birds Areas)
    m_amte:299FN3:d4ef3032-5c9e-4263-b3a1-56a60e7124dc - Servizio di ricerca del catalogo metadati del Geoportale nazionale
    m_amte:299FN3:4168a1a3-3b3e-475e-cb8e-9a161a702e83 - AIB - Modelli di combustibile
    m_amte:299FN3:79cbfd04-955b-4f8c-9de1-65a44ff0bdd9 - Corine Land Cover anno 2006



```
s="m_amte:299FN3:d0439890-8379-45ac-f70b-3d7f11f20ce9" #water
```


```
record = csw.records[s]
```


```
record.title
```




    'Bacini idrografici principali'




```
record.abstract
```




    'Sulla base dello strato informativo dei bacini idrografici a scala nazionale 1:250.000, congruente con il reticolo idrografico, sono stati individuati, secondo quanto previsto dal D.Lgs.152/99 e successivamente dalla Direttiva Quadro sulle Acque 2000/60/CE, i bacini idrografici dei corsi d’acqua scolanti a mare con superficie maggiore o uguale a 200 Kmq.'




```
for reference in record.references:
  print(reference['scheme'])
  print(reference['url'])
```

    urn:x-esri:specification:ServiceType:ArcIMS:Metadata:Server
    http://wms.pcn.minambiente.it/ogc?map=/ms_ogc/WMS_v1.3/Vettoriali/Bacini_idrografici.map
    urn:x-esri:specification:ServiceType:ArcIMS:Metadata:Server
    http://www.pcn.minambiente.it/viewer/index.php?services=bacini_idrografici
    urn:x-esri:specification:ServiceType:ArcIMS:Metadata:Server
    http://wms.pcn.minambiente.it/ogc?map=/ms_ogc/wfs/Bacini_idrografici.map&Service=WFS
    urn:x-esri:specification:ServiceType:ArcIMS:Metadata:Thumbnail
    http://www.pcn.minambiente.it/anteprima/bacini_idrografici.gif
    urn:x-esri:specification:ServiceType:ArcIMS:Metadata:Document
    http://www.pcn.minambiente.it/geoportal/csw?getxml=%7BB10A04E2-2B67-434B-AD0E-30ED272F63E9%7D
    OGC:WMS
    http://wms.pcn.minambiente.it/ogc?map=/ms_ogc/WMS_v1.3/Vettoriali/Bacini_idrografici.map
    OGC:WFS
    http://wms.pcn.minambiente.it/ogc?map=/ms_ogc/wfs/Bacini_idrografici.map&Service=WFS


## WFS


```
from owslib.wfs import WebFeatureService
```


```
url="http://wms.pcn.minambiente.it/ogc?map=/ms_ogc/wfs/Specchi_Acqua.map&Service=WFS"
```


```
wfs = WebFeatureService(url=url,version="1.1.0") #version can be: 1.0.0, 1.1.0, 2.0.0
```


```
wfs.identification.title
```




    "Specchi d'acqua interni"




```
[operation.name for operation in wfs.operations]
```




    ['GetCapabilities', 'DescribeFeatureType', 'GetFeature']




```
list(wfs.contents)
```




    ['ID.ACQUEFISICHE.SPECCHI.ACQUA']




```
capabilities = wfs.getcapabilities().read()
```

```xml
<WFS_Capabilities xmlns="http://www.opengis.net/wfs" xmlns:ogc="http://www.opengis.net/ogc" xmlns:ows="http://www.opengis.net/ows" xmlns:xlink="http://www.w3.org/1999/xlink" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" version="1.1.0" xmlns:inspire_common="http://inspire.ec.europa.eu/schemas/common/1.0" xmlns:inspire_dls="http://inspire.ec.europa.eu/schemas/inspire_dls/1.0" xsi:schemaLocation="http://www.opengis.net/wfs http://schemas.opengis.net/wfs/1.1.0/wfs.xsd http://inspire.ec.europa.eu/schemas/inspire_dls/1.0 http://inspire.ec.europa.eu/schemas/inspire_dls/1.0/inspire_dls.xsd">
  <ows:ServiceIdentification>
    <ows:Title>Specchi d\'acqua interni</ows:Title>
    <ows:Abstract>Elementi poligonali appartenenti al DBPrior10K  rappresentanti gli specchi d\'acqua interni come laghi, lagune, stagni, bacini artificiali, ecc.</ows:Abstract>
    <ows:ServiceType codeSpace="OGC">OGC WFS</ows:ServiceType>
    <ows:ServiceTypeVersion>1.1.0</ows:ServiceTypeVersion>
    <ows:Fees>Nessuna condizione applicata</ows:Fees>
    <ows:AccessConstraints>Nessuno</ows:AccessConstraints>
  </ows:ServiceIdentification>
  <ows:ServiceProvider>
    <ows:ProviderName>Geoportale Nazionale - Ministero dell\'Ambiente e della Tutela del Territorio e del Mare</ows:ProviderName>
    <ows:ProviderSite xlink:href="http://wms.pcn.minambiente.it/ogc?map=/ms_ogc/wfs/Specchi_Acqua.map" xlink:type="simple" />
    <ows:ServiceContact>
      <ows:IndividualName>Geoportale Nazionale - Ministero dell\'Ambiente e della Tutela del Territorio e del Mare</ows:IndividualName>
      <ows:PositionName>Distributore</ows:PositionName>
      <ows:ContactInfo>
        <ows:Phone>
          <ows:Voice>+390657223140</ows:Voice>
          <ows:Facsimile />
        </ows:Phone>
        <ows:Address>
          <ows:DeliveryPoint>Via Cristoforo Colombo, 44</ows:DeliveryPoint>
          <ows:City>Roma</ows:City>
          <ows:AdministrativeArea>RM</ows:AdministrativeArea>
          <ows:PostalCode>00147</ows:PostalCode>
          <ows:Country>Italia</ows:Country>
          <ows:ElectronicMailAddress>pcn@minambiente.it</ows:ElectronicMailAddress>
        </ows:Address>
        <ows:OnlineResource xlink:href="http://wms.pcn.minambiente.it/ogc?map=/ms_ogc/wfs/Specchi_Acqua.map" xlink:type="simple" />
        <ows:HoursOfService />
        <ows:ContactInstructions />
      </ows:ContactInfo>
      <ows:Role />
    </ows:ServiceContact>
  </ows:ServiceProvider>
  <ows:OperationsMetadata>
    <ows:Operation name="GetCapabilities">
      <ows:DCP>
        <ows:HTTP>
          <ows:Get xlink:href="http://wms.pcn.minambiente.it/ogc?map=/ms_ogc/wfs/Specchi_Acqua.map&amp;" xlink:type="simple" />
          <ows:Post xlink:href="http://wms.pcn.minambiente.it/ogc?map=/ms_ogc/wfs/Specchi_Acqua.map&amp;" xlink:type="simple" />
        </ows:HTTP>
      </ows:DCP>
      <ows:Parameter name="service">
        <ows:Value>WFS</ows:Value>
      </ows:Parameter>
      <ows:Parameter name="AcceptVersions">
        <ows:Value>1.0.0</ows:Value>
        <ows:Value>1.1.0</ows:Value>
      </ows:Parameter>
      <ows:Parameter name="AcceptFormats">
        <ows:Value>text/xml</ows:Value>
      </ows:Parameter>
    </ows:Operation>
    <ows:Operation name="DescribeFeatureType">
      <ows:DCP>
        <ows:HTTP>
          <ows:Get xlink:href="http://wms.pcn.minambiente.it/ogc?map=/ms_ogc/wfs/Specchi_Acqua.map&amp;" xlink:type="simple" />
          <ows:Post xlink:href="http://wms.pcn.minambiente.it/ogc?map=/ms_ogc/wfs/Specchi_Acqua.map&amp;" xlink:type="simple" />
        </ows:HTTP>
      </ows:DCP>
      <ows:Parameter name="outputFormat">
        <ows:Value>XMLSCHEMA</ows:Value>
        <ows:Value>text/xml; subtype=gml/2.1.2</ows:Value>
        <ows:Value>text/xml; subtype=gml/3.1.1</ows:Value>
      </ows:Parameter>
    </ows:Operation>
    <ows:Operation name="GetFeature">
      <ows:DCP>
        <ows:HTTP>
          <ows:Get xlink:href="http://wms.pcn.minambiente.it/ogc?map=/ms_ogc/wfs/Specchi_Acqua.map&amp;" xlink:type="simple" />
          <ows:Post xlink:href="http://wms.pcn.minambiente.it/ogc?map=/ms_ogc/wfs/Specchi_Acqua.map&amp;" xlink:type="simple" />
        </ows:HTTP>
      </ows:DCP>
      <ows:Parameter name="resultType">
        <ows:Value>results</ows:Value>
        <ows:Value>hits</ows:Value>
      </ows:Parameter>
      <ows:Parameter name="outputFormat">
        <ows:Value>text/xml; subtype=gml/3.1.1</ows:Value>
      </ows:Parameter>
    </ows:Operation>
<ows:ExtendedCapabilities><inspire_dls:ExtendedCapabilities><inspire_common:ResourceLocator xsi:type="inspire_common:resourceLocatorType"><inspire_common:URL>http://wms.pcn.minambiente.it/cgi-bin/mapserv.exe?map=/ms_ogc/wfs/Bacini_idrografici.map</inspire_common:URL><inspire_common:MediaType>application/vnd.ogc.wfs_xml</inspire_common:MediaType></inspire_common:ResourceLocator><inspire_common:ResourceType>service</inspire_common:ResourceType><inspire_common:TemporalReference><inspire_common:DateOfCreation>2011-09-20</inspire_common:DateOfCreation></inspire_common:TemporalReference><inspire_common:TemporalReference><inspire_common:DateOfPublication>2011-09-20</inspire_common:DateOfPublication></inspire_common:TemporalReference><inspire_common:TemporalReference><inspire_common:DateOfLastRevision>2013-01-23</inspire_common:DateOfLastRevision></inspire_common:TemporalReference><inspire_common:Conformity><inspire_common:Specification><inspire_common:Title>REGOLAMENTO (UE) N. 1089/2010 DELLA COMMISSIONE del 23 novembre 2010 recante attuazione della direttiva 2007/2/CE del Parlamento europeo e del Consiglio per quanto riguarda l\'interoperabilit&#224; dei set di dati territoriali e dei servizi di dati territoriali</inspire_common:Title><inspire_common:DateOfPublication>2010-12-08</inspire_common:DateOfPublication><inspire_common:URI>OJ:L:2010:323:0011:0102:IT:PDF</inspire_common:URI><inspire_common:ResourceLocator><inspire_common:URL>http://eur-lex.europa.eu/LexUriServ/LexUriServ.do?uri=OJ:L:2010:323:0011:0102:IT:PDF</inspire_common:URL><inspire_common:MediaType>application/pdf</inspire_common:MediaType></inspire_common:ResourceLocator></inspire_common:Specification><inspire_common:Degree>notConformant</inspire_common:Degree></inspire_common:Conformity><inspire_common:MetadataPointOfContact><inspire_common:OrganisationName>Ministero dell\'Ambiente e della Tutela del Territorio e del Mare - Geoportale Nazionale</inspire_common:OrganisationName><inspire_common:EmailAddress>pcn@minambiente.it</inspire_common:EmailAddress></inspire_common:MetadataPointOfContact><inspire_common:MetadataDate>2011-04-28</inspire_common:MetadataDate><inspire_common:SpatialDataServiceType>Download</inspire_common:SpatialDataServiceType><inspire_common:MandatoryKeyword><inspire_common:KeywordValue>infoFeatureAccessService</inspire_common:KeywordValue></inspire_common:MandatoryKeyword><inspire_common:Keyword xsi:type="inspire_common:inspireTheme_ita"><inspire_common:OriginatingControlledVocabulary><inspire_common:Title>GEMET - INSPIRE themes</inspire_common:Title><inspire_common:DateOfPublication>2008-06-01</inspire_common:DateOfPublication></inspire_common:OriginatingControlledVocabulary><inspire_common:KeywordValue>Idrografia</inspire_common:KeywordValue></inspire_common:Keyword><inspire_common:Keyword><inspire_common:KeywordValue>Acque interne</inspire_common:KeywordValue></inspire_common:Keyword><inspire_common:Keyword><inspire_common:OriginatingControlledVocabulary><inspire_common:Title>GEMET - Themes, version 2.4</inspire_common:Title><inspire_common:DateOfPublication>2010-01-13</inspire_common:DateOfPublication></inspire_common:OriginatingControlledVocabulary><inspire_common:KeywordValue>Acqua</inspire_common:KeywordValue></inspire_common:Keyword><inspire_common:Keyword><inspire_common:OriginatingControlledVocabulary><inspire_common:Title>GEMET - Concepts, version 2.4</inspire_common:Title><inspire_common:DateOfPublication>2010-01-13</inspire_common:DateOfPublication></inspire_common:OriginatingControlledVocabulary><inspire_common:KeywordValue>Lago</inspire_common:KeywordValue></inspire_common:Keyword><inspire_common:Keyword><inspire_common:OriginatingControlledVocabulary><inspire_common:Title>GEMET - Concepts, version 2.4</inspire_common:Title><inspire_common:DateOfPublication>2010-01-13</inspire_common:DateOfPublication></inspire_common:OriginatingControlledVocabulary><inspire_common:KeywordValue>Lago artificiale</inspire_common:KeywordValue></inspire_common:Keyword><inspire_common:Keyword><inspire_common:OriginatingControlledVocabulary><inspire_common:Title>GEMET - Concepts, version 2.4</inspire_common:Title><inspire_common:DateOfPublication>2010-01-13</inspire_common:DateOfPublication></inspire_common:OriginatingControlledVocabulary><inspire_common:KeywordValue>Laguna</inspire_common:KeywordValue></inspire_common:Keyword><inspire_common:SupportedLanguages><inspire_common:DefaultLanguage><inspire_common:Language>ita</inspire_common:Language></inspire_common:DefaultLanguage><inspire_common:SupportedLanguage><inspire_common:Language>ita</inspire_common:Language></inspire_common:SupportedLanguage></inspire_common:SupportedLanguages><inspire_common:ResponseLanguage><inspire_common:Language>ita</inspire_common:Language></inspire_common:ResponseLanguage></inspire_dls:ExtendedCapabilities></ows:ExtendedCapabilities></ows:OperationsMetadata>
  <FeatureTypeList>
    <Operations>
      <Operation>Query</Operation>
    </Operations>
    <FeatureType>
      <Name>ID.ACQUEFISICHE.SPECCHI.ACQUA</Name>
      <Title>Specchi d\'acqua interni</Title>
      <Abstract>La tabella associata contiene le seguenti informazioni: NOME,  informazione, non sempre presente, relativa al nome allo specchio d\'acqua; ORIGINE, origine del dato spaziale; NATURA SPECCHIO D\'ACQUA, individua la natura dello specchio d\'acqua (lago, laguna, bacino artificiale, ecc.); AREA_mq, area, espressa in mq, dello specchio d\'acqua; PERIMETRO_m, perimetro, espresso in metri, dello specchio d\'acqua.</Abstract>
      <ows:Keywords>
        <ows:Keyword>Idrografia</ows:Keyword>
        <ows:Keyword> Acqua</ows:Keyword>
        <ows:Keyword> Lago</ows:Keyword>
        <ows:Keyword> Lago artificiale</ows:Keyword>
        <ows:Keyword> Laguna</ows:Keyword>
      </ows:Keywords>
      <DefaultSRS>urn:ogc:def:crs:EPSG::4326</DefaultSRS>
      <OutputFormats>
        <Format>text/xml; subtype=gml/3.1.1</Format>
      </OutputFormats>
      <ows:WGS84BoundingBox dimensions="2">
        <ows:LowerCorner>6 34.5</ows:LowerCorner>
        <ows:UpperCorner>19 49</ows:UpperCorner>
      </ows:WGS84BoundingBox>
      <MetadataURL format="ISO19115:2003" type="text/xml">http://www.pcn.minambiente.it/geoportal/csw?SERVICE=CSW&amp;VERSION=2.0.2&amp;REQUEST=GetRecordById&amp;outputSchema=http%3A%2F%2Fwww.isotc211.org%2F2005%2Fgmd&amp;elementSetName=full&amp;ID=m_amte:8HCH2C:5e1ddddd-9414-4a3f-9460-ccb88eef00c2</MetadataURL>
    </FeatureType>
  </FeatureTypeList>
  <ogc:Filter_Capabilities>
    <ogc:Spatial_Capabilities>
      <ogc:GeometryOperands>
        <ogc:GeometryOperand>gml:Point</ogc:GeometryOperand>
        <ogc:GeometryOperand>gml:LineString</ogc:GeometryOperand>
        <ogc:GeometryOperand>gml:Polygon</ogc:GeometryOperand>
        <ogc:GeometryOperand>gml:Envelope</ogc:GeometryOperand>
      </ogc:GeometryOperands>
      <ogc:SpatialOperators>
        <ogc:SpatialOperator name="Equals" />
        <ogc:SpatialOperator name="Disjoint" />
        <ogc:SpatialOperator name="Touches" />
        <ogc:SpatialOperator name="Within" />
        <ogc:SpatialOperator name="Overlaps" />
        <ogc:SpatialOperator name="Crosses" />
        <ogc:SpatialOperator name="Intersects" />
        <ogc:SpatialOperator name="Contains" />
        <ogc:SpatialOperator name="DWithin" />
        <ogc:SpatialOperator name="Beyond" />
        <ogc:SpatialOperator name="BBOX" />
      </ogc:SpatialOperators>
    </ogc:Spatial_Capabilities>
    <ogc:Scalar_Capabilities>
      <ogc:LogicalOperators />
      <ogc:ComparisonOperators>
        <ogc:ComparisonOperator>LessThan</ogc:ComparisonOperator>
        <ogc:ComparisonOperator>GreaterThan</ogc:ComparisonOperator>
        <ogc:ComparisonOperator>LessThanEqualTo</ogc:ComparisonOperator>
        <ogc:ComparisonOperator>GreaterThanEqualTo</ogc:ComparisonOperator>
        <ogc:ComparisonOperator>EqualTo</ogc:ComparisonOperator>
        <ogc:ComparisonOperator>NotEqualTo</ogc:ComparisonOperator>
        <ogc:ComparisonOperator>Like</ogc:ComparisonOperator>
        <ogc:ComparisonOperator>Between</ogc:ComparisonOperator>
      </ogc:ComparisonOperators>
    </ogc:Scalar_Capabilities>
    <ogc:Id_Capabilities>
      <ogc:EID />
      <ogc:FID />
    </ogc:Id_Capabilities>
  </ogc:Filter_Capabilities>
</WFS_Capabilities>
```


```
for layer, meta in wfs.items():
    print(meta.__dict__)
    print(meta.title)
    print(meta.abstract)
    print(meta.crsOptions)
    print(meta.outputFormats)
    
```

    {'auth': <Authentication shared=False username=None password=None cert=None verify=True>, 'headers': None, 'id': 'ID.ACQUEFISICHE.SPECCHI.ACQUA', 'title': "Specchi d'acqua interni", 'abstract': "La tabella associata contiene le seguenti informazioni: NOME,  informazione, non sempre presente, relativa al nome allo specchio d'acqua; ORIGINE, origine del dato spaziale; NATURA SPECCHIO D'ACQUA, individua la natura dello specchio d'acqua (lago, laguna, bacino artificiale, ecc.); AREA_mq, area, espressa in mq, dello specchio d'acqua; PERIMETRO_m, perimetro, espresso in metri, dello specchio d'acqua.", 'keywords': ['Idrografia', ' Acqua', ' Lago', ' Lago artificiale', ' Laguna'], 'boundingBoxWGS84': (6.0, 34.5, 19.0, 49.0), 'crsOptions': [urn:ogc:def:crs:EPSG::4326], 'verbOptions': [], 'outputFormats': ['text/xml; subtype=gml/3.1.1'], 'metadataUrls': [{'type': 'text/xml', 'format': 'ISO19115:2003', 'url': 'http://www.pcn.minambiente.it/geoportal/csw?SERVICE=CSW&VERSION=2.0.2&REQUEST=GetRecordById&outputSchema=http%3A%2F%2Fwww.isotc211.org%2F2005%2Fgmd&elementSetName=full&ID=m_amte:8HCH2C:5e1ddddd-9414-4a3f-9460-ccb88eef00c2'}], 'styles': None, 'timepositions': None, 'defaulttimeposition': None}
    Specchi d'acqua interni
    La tabella associata contiene le seguenti informazioni: NOME,  informazione, non sempre presente, relativa al nome allo specchio d'acqua; ORIGINE, origine del dato spaziale; NATURA SPECCHIO D'ACQUA, individua la natura dello specchio d'acqua (lago, laguna, bacino artificiale, ecc.); AREA_mq, area, espressa in mq, dello specchio d'acqua; PERIMETRO_m, perimetro, espresso in metri, dello specchio d'acqua.
    [urn:ogc:def:crs:EPSG::4326]
    ['text/xml; subtype=gml/3.1.1']



```
layer = list(wfs.contents)[0]
```


```
layer
```




    'ID.ACQUEFISICHE.SPECCHI.ACQUA'




```
response = wfs.getfeature(typename=layer, bbox=(10.7976,45.8649,10.9851,46.0496),srsname='urn:ogc:def:crs:EPSG::4326')
%time
```

    CPU times: user 2 µs, sys: 0 ns, total: 2 µs
    Wall time: 4.53 µs



```
out = open('lakes_inbbox.gml', 'wb')
out.write(response.read())
out.close()
```


```
lakes_inbbox = gpd.read_file("lakes_inbbox.gml")
```


```
lakes_inbbox.head(5)
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
      <th>gml_id</th>
      <th>nome</th>
      <th>origine</th>
      <th>descrizione_origine</th>
      <th>nasp</th>
      <th>natura_specchio_acqua</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ID.ACQUEFISICHE.SPECCHI.ACQUA.54186</td>
      <td>LAGO DI LOPPIO</td>
      <td>0</td>
      <td>Non codificato</td>
      <td>0</td>
      <td>Non codificato</td>
      <td>MULTIPOLYGON (((45.85832 10.92376, 45.85823 10...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ID.ACQUEFISICHE.SPECCHI.ACQUA.54162</td>
      <td>LAGO DEI BAGATOI (DRO)</td>
      <td>0</td>
      <td>Non codificato</td>
      <td>0</td>
      <td>Non codificato</td>
      <td>MULTIPOLYGON (((45.98244 10.92241, 45.98238 10...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ID.ACQUEFISICHE.SPECCHI.ACQUA.54164</td>
      <td>LAGO SOLO</td>
      <td>0</td>
      <td>Non codificato</td>
      <td>0</td>
      <td>Non codificato</td>
      <td>MULTIPOLYGON (((45.97719 10.93186, 45.97714 10...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ID.ACQUEFISICHE.SPECCHI.ACQUA.54159</td>
      <td>LAGO TORBIERA DI FIAVE' 2?</td>
      <td>0</td>
      <td>Non codificato</td>
      <td>0</td>
      <td>Non codificato</td>
      <td>MULTIPOLYGON (((45.99459 10.83050, 45.99450 10...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ID.ACQUEFISICHE.SPECCHI.ACQUA.54160</td>
      <td>LAGO TORBIERA DI FIAVE' 3?</td>
      <td>0</td>
      <td>Non codificato</td>
      <td>0</td>
      <td>Non codificato</td>
      <td>MULTIPOLYGON (((45.99366 10.83341, 45.99362 10...</td>
    </tr>
  </tbody>
</table>
</div>




```
lakes_inbbox.unary_union.centroid.x
```




    45.873490909948416




```
map_lakes = folium.Map([lakes_inbbox.unary_union.centroid.y,lakes_inbbox.unary_union.centroid.x], zoom_start=12)
folium.GeoJson(lakes_inbbox.to_json()).add_to(map_lakes)
map_lakes
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe src="about:blank" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" data-html=PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgCiAgICAgICAgPHNjcmlwdD4KICAgICAgICAgICAgTF9OT19UT1VDSCA9IGZhbHNlOwogICAgICAgICAgICBMX0RJU0FCTEVfM0QgPSBmYWxzZTsKICAgICAgICA8L3NjcmlwdD4KICAgIAogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjYuMC9kaXN0L2xlYWZsZXQuanMiPjwvc2NyaXB0PgogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY29kZS5qcXVlcnkuY29tL2pxdWVyeS0xLjEyLjQubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9qcy9ib290c3RyYXAubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5qcyI+PC9zY3JpcHQ+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjYuMC9kaXN0L2xlYWZsZXQuY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vYm9vdHN0cmFwLzMuMi4wL2Nzcy9ib290c3RyYXAubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLXRoZW1lLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9mb250LWF3ZXNvbWUvNC42LjMvY3NzL2ZvbnQtYXdlc29tZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuanMuY2xvdWRmbGFyZS5jb20vYWpheC9saWJzL0xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLzIuMC4yL2xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvZ2gvcHl0aG9uLXZpc3VhbGl6YXRpb24vZm9saXVtL2ZvbGl1bS90ZW1wbGF0ZXMvbGVhZmxldC5hd2Vzb21lLnJvdGF0ZS5taW4uY3NzIi8+CiAgICA8c3R5bGU+aHRtbCwgYm9keSB7d2lkdGg6IDEwMCU7aGVpZ2h0OiAxMDAlO21hcmdpbjogMDtwYWRkaW5nOiAwO308L3N0eWxlPgogICAgPHN0eWxlPiNtYXAge3Bvc2l0aW9uOmFic29sdXRlO3RvcDowO2JvdHRvbTowO3JpZ2h0OjA7bGVmdDowO308L3N0eWxlPgogICAgCiAgICAgICAgICAgIDxtZXRhIG5hbWU9InZpZXdwb3J0IiBjb250ZW50PSJ3aWR0aD1kZXZpY2Utd2lkdGgsCiAgICAgICAgICAgICAgICBpbml0aWFsLXNjYWxlPTEuMCwgbWF4aW11bS1zY2FsZT0xLjAsIHVzZXItc2NhbGFibGU9bm8iIC8+CiAgICAgICAgICAgIDxzdHlsZT4KICAgICAgICAgICAgICAgICNtYXBfMGRkYjRhMzg1Njc3NDExNDg1N2Q4MGZiODVjZTcyYzMgewogICAgICAgICAgICAgICAgICAgIHBvc2l0aW9uOiByZWxhdGl2ZTsKICAgICAgICAgICAgICAgICAgICB3aWR0aDogMTAwLjAlOwogICAgICAgICAgICAgICAgICAgIGhlaWdodDogMTAwLjAlOwogICAgICAgICAgICAgICAgICAgIGxlZnQ6IDAuMCU7CiAgICAgICAgICAgICAgICAgICAgdG9wOiAwLjAlOwogICAgICAgICAgICAgICAgfQogICAgICAgICAgICA8L3N0eWxlPgogICAgICAgIAo8L2hlYWQ+Cjxib2R5PiAgICAKICAgIAogICAgICAgICAgICA8ZGl2IGNsYXNzPSJmb2xpdW0tbWFwIiBpZD0ibWFwXzBkZGI0YTM4NTY3NzQxMTQ4NTdkODBmYjg1Y2U3MmMzIiA+PC9kaXY+CiAgICAgICAgCjwvYm9keT4KPHNjcmlwdD4gICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcF8wZGRiNGEzODU2Nzc0MTE0ODU3ZDgwZmI4NWNlNzJjMyA9IEwubWFwKAogICAgICAgICAgICAgICAgIm1hcF8wZGRiNGEzODU2Nzc0MTE0ODU3ZDgwZmI4NWNlNzJjMyIsCiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgY2VudGVyOiBbMTAuODYxODQ0OTMyMDU4OTc5LCA0NS44NzM0OTA5MDk5NDg0MTZdLAogICAgICAgICAgICAgICAgICAgIGNyczogTC5DUlMuRVBTRzM4NTcsCiAgICAgICAgICAgICAgICAgICAgem9vbTogMTIsCiAgICAgICAgICAgICAgICAgICAgem9vbUNvbnRyb2w6IHRydWUsCiAgICAgICAgICAgICAgICAgICAgcHJlZmVyQ2FudmFzOiBmYWxzZSwKICAgICAgICAgICAgICAgIH0KICAgICAgICAgICAgKTsKCiAgICAgICAgICAgIAoKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgdGlsZV9sYXllcl8xYzg3ZTczNDMxMzI0NTU1YjQ4NDA5MjBmOTFkZTkxMSA9IEwudGlsZUxheWVyKAogICAgICAgICAgICAgICAgImh0dHBzOi8ve3N9LnRpbGUub3BlbnN0cmVldG1hcC5vcmcve3p9L3t4fS97eX0ucG5nIiwKICAgICAgICAgICAgICAgIHsiYXR0cmlidXRpb24iOiAiRGF0YSBieSBcdTAwMjZjb3B5OyBcdTAwM2NhIGhyZWY9XCJodHRwOi8vb3BlbnN0cmVldG1hcC5vcmdcIlx1MDAzZU9wZW5TdHJlZXRNYXBcdTAwM2MvYVx1MDAzZSwgdW5kZXIgXHUwMDNjYSBocmVmPVwiaHR0cDovL3d3dy5vcGVuc3RyZWV0bWFwLm9yZy9jb3B5cmlnaHRcIlx1MDAzZU9EYkxcdTAwM2MvYVx1MDAzZS4iLCAiZGV0ZWN0UmV0aW5hIjogZmFsc2UsICJtYXhOYXRpdmVab29tIjogMTgsICJtYXhab29tIjogMTgsICJtaW5ab29tIjogMCwgIm5vV3JhcCI6IGZhbHNlLCAib3BhY2l0eSI6IDEsICJzdWJkb21haW5zIjogImFiYyIsICJ0bXMiOiBmYWxzZX0KICAgICAgICAgICAgKS5hZGRUbyhtYXBfMGRkYjRhMzg1Njc3NDExNDg1N2Q4MGZiODVjZTcyYzMpOwogICAgICAgIAogICAgCiAgICAgICAgZnVuY3Rpb24gZ2VvX2pzb25fNDIyZTY1N2JkOGJiNGZiOTk1ZTllMTkyM2ZkYWEzNTZfb25FYWNoRmVhdHVyZShmZWF0dXJlLCBsYXllcikgewogICAgICAgICAgICBsYXllci5vbih7CiAgICAgICAgICAgIH0pOwogICAgICAgIH07CiAgICAgICAgdmFyIGdlb19qc29uXzQyMmU2NTdiZDhiYjRmYjk5NWU5ZTE5MjNmZGFhMzU2ID0gTC5nZW9Kc29uKG51bGwsIHsKICAgICAgICAgICAgICAgIG9uRWFjaEZlYXR1cmU6IGdlb19qc29uXzQyMmU2NTdiZDhiYjRmYjk5NWU5ZTE5MjNmZGFhMzU2X29uRWFjaEZlYXR1cmUsCiAgICAgICAgICAgIAogICAgICAgIH0pOwoKICAgICAgICBmdW5jdGlvbiBnZW9fanNvbl80MjJlNjU3YmQ4YmI0ZmI5OTVlOWUxOTIzZmRhYTM1Nl9hZGQgKGRhdGEpIHsKICAgICAgICAgICAgZ2VvX2pzb25fNDIyZTY1N2JkOGJiNGZiOTk1ZTllMTkyM2ZkYWEzNTYKICAgICAgICAgICAgICAgIC5hZGREYXRhKGRhdGEpCiAgICAgICAgICAgICAgICAuYWRkVG8obWFwXzBkZGI0YTM4NTY3NzQxMTQ4NTdkODBmYjg1Y2U3MmMzKTsKICAgICAgICB9CiAgICAgICAgICAgIGdlb19qc29uXzQyMmU2NTdiZDhiYjRmYjk5NWU5ZTE5MjNmZGFhMzU2X2FkZCh7ImZlYXR1cmVzIjogW3siZ2VvbWV0cnkiOiB7ImNvb3JkaW5hdGVzIjogW1tbWzQ1Ljg1ODMyNCwgMTAuOTIzNzY1XSwgWzQ1Ljg1ODIyNiwgMTAuOTIzNzM5XSwgWzQ1Ljg1ODE2OSwgMTAuOTIzNzAxXSwgWzQ1Ljg1ODExOCwgMTAuOTIzNjM3XSwgWzQ1Ljg1ODEwNiwgMTAuOTIzNTkxXSwgWzQ1Ljg1ODA4MywgMTAuOTIzNTM3XSwgWzQ1Ljg1ODA2NCwgMTAuOTIzNDI5XSwgWzQ1Ljg1ODA1NCwgMTAuOTIzMzI3XSwgWzQ1Ljg1ODA1MywgMTAuOTIzMjA5XSwgWzQ1Ljg1ODA0OSwgMTAuOTIzMDc4XSwgWzQ1Ljg1ODA0MiwgMTAuOTIyOTAxXSwgWzQ1Ljg1ODAyNywgMTAuOTIyNzEzXSwgWzQ1Ljg1Nzk4MiwgMTAuOTIyMjNdLCBbNDUuODU3OTcsIDEwLjkyMTk1OF0sIFs0NS44NTc5NTIsIDEwLjkyMTc5N10sIFs0NS44NTc4ODgsIDEwLjkyMTY1MV0sIFs0NS44NTc4MjgsIDEwLjkyMTU2XSwgWzQ1Ljg1Nzc2OCwgMTAuOTIxNDM0XSwgWzQ1Ljg1NzczOCwgMTAuOTIxMjc5XSwgWzQ1Ljg1Nzc1LCAxMC45MjExMzldLCBbNDUuODU3Nzg2LCAxMC45MjA5OTZdLCBbNDUuODU3ODI2LCAxMC45MjA5MDldLCBbNDUuODU3OTA0LCAxMC45MjA4XSwgWzQ1Ljg1Nzk5OSwgMTAuOTIwNjkxXSwgWzQ1Ljg1ODE1MywgMTAuOTIwNTU1XSwgWzQ1Ljg1ODI1NSwgMTAuOTIwNDY3XSwgWzQ1Ljg1ODM0LCAxMC45MjA0NV0sIFs0NS44NTg0NDksIDEwLjkyMDQ3XSwgWzQ1Ljg1ODU1MSwgMTAuOTIwNTY1XSwgWzQ1Ljg1ODU4OSwgMTAuOTIwNjUxXSwgWzQ1Ljg1ODYwNiwgMTAuOTIwNzFdLCBbNDUuODU4NjUxLCAxMC45MjA3NjRdLCBbNDUuODU4NzU4LCAxMC45MjA4MjNdLCBbNDUuODU4ODg1LCAxMC45MjA4N10sIFs0NS44NTkwMTUsIDEwLjkyMDldLCBbNDUuODU5MTAyLCAxMC45MjA5MDNdLCBbNDUuODU5MTkyLCAxMC45MjA4ODZdLCBbNDUuODU5MzMsIDEwLjkyMDgxOF0sIFs0NS44NTkzNzgsIDEwLjkyMDgyM10sIFs0NS44NTk0OSwgMTAuOTIwODYzXSwgWzQ1Ljg1OTU4MywgMTAuOTIwNzYxXSwgWzQ1Ljg1OTU5NywgMTAuOTIwNTc4XSwgWzQ1Ljg1OTU4NywgMTAuOTIwNTI1XSwgWzQ1Ljg1OTU3NywgMTAuOTIwNDQ2XSwgWzQ1Ljg1OTU3NSwgMTAuOTIwMzg0XSwgWzQ1Ljg1OTYwNiwgMTAuOTIwMjk3XSwgWzQ1Ljg1OTYyMiwgMTAuOTIwMjEyXSwgWzQ1Ljg1OTYzLCAxMC45MjAwOTddLCBbNDUuODU5NjE4LCAxMC45MjAwMjJdLCBbNDUuODU5NjAxLCAxMC45MTk5NDldLCBbNDUuODU5NjA0LCAxMC45MTk4NjRdLCBbNDUuODU5NjE5LCAxMC45MTk4MTJdLCBbNDUuODU5NjUxLCAxMC45MTk3ODRdLCBbNDUuODU5NjksIDEwLjkxOTc4NV0sIFs0NS44NTk3NTksIDEwLjkxOThdLCBbNDUuODU5ODMzLCAxMC45MTk4NjhdLCBbNDUuODU5OTE5LCAxMC45MTk5NjZdLCBbNDUuODU5OTc3LCAxMC45MjAwMzNdLCBbNDUuODYwMDUsIDEwLjkyMDA4OF0sIFs0NS44NjAxMDIsIDEwLjkyMDEyMl0sIFs0NS44NjAxNTksIDEwLjkyMDExNF0sIFs0NS44NjAyMTIsIDEwLjkyMDA3XSwgWzQ1Ljg2MDI0MSwgMTAuOTIwMDAyXSwgWzQ1Ljg2MDI0NCwgMTAuOTE5OTMzXSwgWzQ1Ljg2MDIyNSwgMTAuOTE5ODQ0XSwgWzQ1Ljg2MDE4OCwgMTAuOTE5NzM1XSwgWzQ1Ljg2MDE1MywgMTAuOTE5NjUyXSwgWzQ1Ljg2MDEyNCwgMTAuOTE5NTc2XSwgWzQ1Ljg2MDA5OCwgMTAuOTE5NDY0XSwgWzQ1Ljg2MDEwMSwgMTAuOTE5NDI0XSwgWzQ1Ljg2MDEzMSwgMTAuOTE5NDEyXSwgWzQ1Ljg2MDE3NywgMTAuOTE5NDNdLCBbNDUuODYwMjA2LCAxMC45MTk0NTddLCBbNDUuODYwMjQsIDEwLjkxOTUwNF0sIFs0NS44NjAyNzEsIDEwLjkxOTU3MV0sIFs0NS44NjAzMDQsIDEwLjkxOTY1M10sIFs0NS44NjAzNTIsIDEwLjkxOTc2M10sIFs0NS44NjA0MDQsIDEwLjkxOTg2XSwgWzQ1Ljg2MDQ1MywgMTAuOTE5OTNdLCBbNDUuODYwNDkxLCAxMC45MTk5NTRdLCBbNDUuODYwNTY3LCAxMC45MTk5MjFdLCBbNDUuODYwNTk2LCAxMC45MTk4ODJdLCBbNDUuODYwNjM0LCAxMC45MTk3NzldLCBbNDUuODYwNjQ1LCAxMC45MTk2NjhdLCBbNDUuODYwNjQ0LCAxMC45MTk1NV0sIFs0NS44NjA3MTcsIDEwLjkxOTI5XSwgWzQ1Ljg2MDc3OCwgMTAuOTE5MTg0XSwgWzQ1Ljg2MDgxNiwgMTAuOTE5MTA2XSwgWzQ1Ljg2MDg0NywgMTAuOTE5MDM4XSwgWzQ1Ljg2MDg4NSwgMTAuOTE4OTMxXSwgWzQ1Ljg2MDkwMywgMTAuOTE4ODE3XSwgWzQ1Ljg2MDkwNSwgMTAuOTE4Njk5XSwgWzQ1Ljg2MDkwNSwgMTAuOTE4NTg4XSwgWzQ1Ljg2MDkzMiwgMTAuOTE4NDcxXSwgWzQ1Ljg2MDk1OSwgMTAuOTE4MzQ0XSwgWzQ1Ljg2MDk4MywgMTAuOTE4MjVdLCBbNDUuODYxMDUxLCAxMC45MTgxMzddLCBbNDUuODYxMjA1LCAxMC45MTgwMDFdLCBbNDUuODYxMzUsIDEwLjkxNzkxN10sIFs0NS44NjE0OTgsIDEwLjkxNzgxNF0sIFs0NS44NjE2NDYsIDEwLjkxNzczN10sIFs0NS44NjE3OTIsIDEwLjkxNzY1Nl0sIFs0NS44NjIwMzIsIDEwLjkxNzUyM10sIFs0NS44NjIyMDMsIDEwLjkxNzQyN10sIFs0NS44NjI0MDIsIDEwLjkxNzMyOF0sIFs0NS44NjI1ODIsIDEwLjkxNzI0OV0sIFs0NS44NjI4MjQsIDEwLjkxNzIxNF0sIFs0NS44NjI5NzYsIDEwLjkxNzE5Ml0sIFs0NS44NjMxODQsIDEwLjkxNzIxMl0sIFs0NS44NjMzNjUsIDEwLjkxNzIzNF0sIFs0NS44NjM1MDUsIDEwLjkxNzI0NV0sIFs0NS44NjM2NDcsIDEwLjkxNzIzM10sIFs0NS44NjM4MjIsIDEwLjkxNzE4M10sIFs0NS44NjM5NjcsIDEwLjkxNzExMl0sIFs0NS44NjQwOTksIDEwLjkxNzAxOF0sIFs0NS44NjQyMzQsIDEwLjkxNjg5NF0sIFs0NS44NjQyOTQsIDEwLjkxNjgzNF0sIFs0NS44NjQ0MDksIDEwLjkxNjcwN10sIFs0NS44NjQ1LCAxMC45MTY1ODJdLCBbNDUuODY0NTc3LCAxMC45MTY1MDldLCBbNDUuODY0NjUyLCAxMC45MTY0OTJdLCBbNDUuODY0NzMsIDEwLjkxNjUxNF0sIFs0NS44NjQ3NTQsIDEwLjkxNjU1N10sIFs0NS44NjQ3NjcsIDEwLjkxNjYyXSwgWzQ1Ljg2NDc1MywgMTAuOTE2Njc4XSwgWzQ1Ljg2NDY5OCwgMTAuOTE2Nzc1XSwgWzQ1Ljg2NDYzOCwgMTAuOTE2ODU1XSwgWzQ1Ljg2NDYsIDEwLjkxNjg5XSwgWzQ1Ljg2NDQ2MSwgMTAuOTE3MDEzXSwgWzQ1Ljg2NDM2OCwgMTAuOTE3MTEyXSwgWzQ1Ljg2NDI4NCwgMTAuOTE3MjE0XSwgWzQ1Ljg2NDE5OSwgMTAuOTE3MzMyXSwgWzQ1Ljg2NDE2NiwgMTAuOTE3NDIzXSwgWzQ1Ljg2NDE1OCwgMTAuOTE3NTA4XSwgWzQ1Ljg2NDIyNSwgMTAuOTE3NzU5XSwgWzQ1Ljg2NDI2MywgMTAuOTE3ODIzXSwgWzQ1Ljg2NDMzMiwgMTAuOTE3OTA3XSwgWzQ1Ljg2NDQxNCwgMTAuOTE3OTc4XSwgWzQ1Ljg2NDQ4OCwgMTAuOTE4MDM2XSwgWzQ1Ljg2NDU4OCwgMTAuOTE4MDc5XSwgWzQ1Ljg2NDY4OSwgMTAuOTE4MDk4XSwgWzQ1Ljg2NDgxNSwgMTAuOTE4MDgzXSwgWzQ1Ljg2NDk1MywgMTAuOTE4MDYxXSwgWzQ1Ljg2NTA3NSwgMTAuOTE3OTk5XSwgWzQ1Ljg2NTIyMSwgMTAuOTE3OTA5XSwgWzQ1Ljg2NTQwNiwgMTAuOTE3OF0sIFs0NS44NjU1ODQsIDEwLjkxNzY4N10sIFs0NS44NjU3NTYsIDEwLjkxNzU1Ml0sIFs0NS44NjU5MTYsIDEwLjkxNzM4N10sIFs0NS44NjYwNTQsIDEwLjkxNzIwMV0sIFs0NS44NjYxMDksIDEwLjkxNzA2Ml0sIFs0NS44NjYxMzIsIDEwLjkxNjkzNV0sIFs0NS44NjYxMjksIDEwLjkxNjgxM10sIFs0NS44NjYwOTQsIDEwLjkxNjcyMV0sIFs0NS44NjYwMDQsIDEwLjkxNjYyOV0sIFs0NS44NjU5MTEsIDEwLjkxNjU0NF0sIFs0NS44NjU4MTQsIDEwLjkxNjQ2M10sIFs0NS44NjU3MTcsIDEwLjkxNjM5NF0sIFs0NS44NjU1OTYsIDEwLjkxNjMzMV0sIFs0NS44NjU0NiwgMTAuOTE2Mjc4XSwgWzQ1Ljg2NTMwOSwgMTAuOTE2Mjc2XSwgWzQ1Ljg2NTIwMywgMTAuOTE2MjZdLCBbNDUuODY1MTUxLCAxMC45MTYyMTldLCBbNDUuODY1MTUsIDEwLjkxNjE1XSwgWzQ1Ljg2NTE2NywgMTAuOTE2MTA4XSwgWzQ1Ljg2NTIxOCwgMTAuOTE2MDQxXSwgWzQ1Ljg2NTI4MSwgMTAuOTE1OTc3XSwgWzQ1Ljg2NTM2MywgMTAuOTE1ODkxXSwgWzQ1Ljg2NTQ0LCAxMC45MTU4MDJdLCBbNDUuODY1NTM1LCAxMC45MTU3MV0sIFs0NS44NjU1ODQsIDEwLjkxNTY0OV0sIFs0NS44NjU2NzksIDEwLjkxNTZdLCBbNDUuODY1NzQsIDEwLjkxNTU3NF0sIFs0NS44NjU4MSwgMTAuOTE1NTVdLCBbNDUuODY1OTA3LCAxMC45MTU0ODFdLCBbNDUuODY2MDA3LCAxMC45MTU0MDVdLCBbNDUuODY2MTIzLCAxMC45MTUzMTddLCBbNDUuODY2MjI5LCAxMC45MTUyNDVdLCBbNDUuODY2MzY2LCAxMC45MTUxNjddLCBbNDUuODY2NTA0LCAxMC45MTUxMTZdLCBbNDUuODY2NjIxLCAxMC45MTUwODddLCBbNDUuODY2NzU3LCAxMC45MTUwNzJdLCBbNDUuODY2OTAxLCAxMC45MTUwNl0sIFs0NS44NjcwMjksIDEwLjkxNTA1NF0sIFs0NS44NjcxNjUsIDEwLjkxNTA0NV0sIFs0NS44NjcyOTMsIDEwLjkxNTAzOV0sIFs0NS44Njc0MjksIDEwLjkxNTAxN10sIFs0NS44Njc1OCwgMTAuOTE0OThdLCBbNDUuODY3NzM0LCAxMC45MTQ5MzldLCBbNDUuODY3ODYxLCAxMC45MTQ4ODddLCBbNDUuODY3OTg5LCAxMC45MTQ3NzNdLCBbNDUuODY4MDkyLCAxMC45MTQ2MTJdLCBbNDUuODY4MTk2LCAxMC45MTQ0MjJdLCBbNDUuODY4MjYsIDEwLjkxNDIzMV0sIFs0NS44NjgyNzksIDEwLjkxNDA4NF0sIFs0NS44NjgzMSwgMTAuOTEzODQ2XSwgWzQ1Ljg2ODMyLCAxMC45MTM2NDNdLCBbNDUuODY4MzIxLCAxMC45MTM0MTRdLCBbNDUuODY4MzMxLCAxMC45MTMyMTFdLCBbNDUuODY4MzYxLCAxMC45MTMwNThdLCBbNDUuODY4NDA0LCAxMC45MTI5NTRdLCBbNDUuODY4NDg3LCAxMC45MTI4NzJdLCBbNDUuODY4NTYzLCAxMC45MTI4MzFdLCBbNDUuODY4NjUzLCAxMC45MTI4MzRdLCBbNDUuODY4NzI4LCAxMC45MTI4NzZdLCBbNDUuODY4ODE2LCAxMC45MTI5NjddLCBbNDUuODY4ODkyLCAxMC45MTMwNjFdLCBbNDUuODY4OTY0LCAxMC45MTMxNTJdLCBbNDUuODY5MDM4LCAxMC45MTMyNDNdLCBbNDUuODY5MTE1LCAxMC45MTMzMzddLCBbNDUuODY5MTkxLCAxMC45MTM0MzFdLCBbNDUuODY5MjMzLCAxMC45MTM1MThdLCBbNDUuODY5MjU5LCAxMC45MTM2MTRdLCBbNDUuODY5MjYsIDEwLjkxMzcxNV0sIFs0NS44NjkyNTIsIDEwLjkxMzgwM10sIFs0NS44NjkyNDgsIDEwLjkxMzkwMl0sIFs0NS44NjkyNDQsIDEwLjkxNDAyM10sIFs0NS44NjkyNDYsIDEwLjkxNDE2NF0sIFs0NS44NjkyMzcsIDEwLjkxNDQ1NV0sIFs0NS44NjkyMjMsIDEwLjkxNDYxOV0sIFs0NS44NjkyMDMsIDEwLjkxNDczOV0sIFs0NS44NjkyLCAxMC45MTQ5NzJdLCBbNDUuODY5MjA0LCAxMC45MTUxNjJdLCBbNDUuODY5MTk0LCAxMC45MTUzMzVdLCBbNDUuODY5MTc2LCAxMC45MTU0NzldLCBbNDUuODY5MTc1LCAxMC45MTU2NzZdLCBbNDUuODY5MTk5LCAxMC45MTU3NjNdLCBbNDUuODY5MjE2LCAxMC45MTU4MzldLCBbNDUuODY5MjQ2LCAxMC45MTU5NDJdLCBbNDUuODY5Mjk1LCAxMC45MTYwNTJdLCBbNDUuODY5MzIyLCAxMC45MTYxMDJdLCBbNDUuODY5MzYsIDEwLjkxNjE0OV0sIFs0NS44NjkzOTYsIDEwLjkxNjE3OV0sIFs0NS44Njk1MDEsIDEwLjkxNjIyMl0sIFs0NS44Njk1ODcsIDEwLjkxNjIzNF0sIFs0NS44Njk2NTksIDEwLjkxNjE4OF0sIFs0NS44Njk3NDMsIDEwLjkxNjA5NV0sIFs0NS44Njk4MzEsIDEwLjkxNjAxNl0sIFs0NS44Njk4OTgsIDEwLjkxNjAwMl0sIFs0NS44Njk5NTMsIDEwLjkxNjAxN10sIFs0NS44NzAwMTYsIDEwLjkxNjA3MV0sIFs0NS44NzAwNywgMTAuOTE2MTU1XSwgWzQ1Ljg3MDA4OSwgMTAuOTE2MjI0XSwgWzQ1Ljg3MDEwMSwgMTAuOTE2MzI5XSwgWzQ1Ljg3MDA5MSwgMTAuOTE2NDI0XSwgWzQ1Ljg3MDA5NiwgMTAuOTE2NTM5XSwgWzQ1Ljg3MDEyNSwgMTAuOTE2NjgxXSwgWzQ1Ljg3MDE2NSwgMTAuOTE2ODAzXSwgWzQ1Ljg3MDIyMywgMTAuOTE2OTFdLCBbNDUuODcwMjc0LCAxMC45MTY5NjddLCBbNDUuODcwMzQyLCAxMC45MTcwNDhdLCBbNDUuODcwMzU3LCAxMC45MTcxMTRdLCBbNDUuODcwMzQ2LCAxMC45MTcyMjhdLCBbNDUuODcwMjYyLCAxMC45MTc0NjhdLCBbNDUuODcwMTcxLCAxMC45MTc1OTZdLCBbNDUuODcwMDk2LCAxMC45MTc2NjZdLCBbNDUuODcwMDEzLCAxMC45MTc3MTldLCBbNDUuODY5OTMyLCAxMC45MTc3NjZdLCBbNDUuODY5ODQ0LCAxMC45MTc4MDldLCBbNDUuODY5NzUyLCAxMC45MTc4MzldLCBbNDUuODY5NjQ3LCAxMC45MTc4MzldLCBbNDUuODY5NTQ4LCAxMC45MTc4MzJdLCBbNDUuODY5NDc3LCAxMC45MTc4NDNdLCBbNDUuODY5NDA1LCAxMC45MTc5NDJdLCBbNDUuODY5MzYzLCAxMC45MTgxMDJdLCBbNDUuODY5MzI0LCAxMC45MTgyNTRdLCBbNDUuODY5Mjc2LCAxMC45MTgzODRdLCBbNDUuODY5MjIyLCAxMC45MTg0NjddLCBbNDUuODY5MTQ2LCAxMC45MTg1MTFdLCBbNDUuODY5MDU3LCAxMC45MTg1MDJdLCBbNDUuODY4OTM0LCAxMC45MTg0NzVdLCBbNDUuODY4ODM4LCAxMC45MTg0NjJdLCBbNDUuODY4Nzg1LCAxMC45MTg0N10sIFs0NS44Njg3NDMsIDEwLjkxODUyNF0sIFs0NS44Njg3NTMsIDEwLjkxODZdLCBbNDUuODY4OCwgMTAuOTE4NjldLCBbNDUuODY4ODUyLCAxMC45MTg3NDddLCBbNDUuODY4OTE5LCAxMC45MTg3OTVdLCBbNDUuODY4OTgsIDEwLjkxODg1M10sIFs0NS44Njg5OTMsIDEwLjkxODkwOV0sIFs0NS44Njg5ODcsIDEwLjkxODk5MV0sIFs0NS44Njg5NzUsIDEwLjkxOTAzXSwgWzQ1Ljg2ODkzNywgMTAuOTE5MTA0XSwgWzQ1Ljg2ODg0OSwgMTAuOTE5MTldLCBbNDUuODY4NzMzLCAxMC45MTkzMDFdLCBbNDUuODY4NTY4LCAxMC45MTk0NTZdLCBbNDUuODY4NDU4LCAxMC45MTk1NjddLCBbNDUuODY4MzE3LCAxMC45MTk2ODFdLCBbNDUuODY4MjA3LCAxMC45MTk3MDddLCBbNDUuODY4MDY5LCAxMC45MTk2ODldLCBbNDUuODY3OTU2LCAxMC45MTk2NDNdLCBbNDUuODY3ODgxLCAxMC45MTk2MDFdLCBbNDUuODY3ODA4LCAxMC45MTk1OTJdLCBbNDUuODY3NzQxLCAxMC45MTk2MDNdLCBbNDUuODY3Njc4LCAxMC45MTk2NTFdLCBbNDUuODY3NjI1LCAxMC45MTk2OThdLCBbNDUuODY3NTQ2LCAxMC45MTk3OTFdLCBbNDUuODY3NDQzLCAxMC45MTk5MjVdLCBbNDUuODY3MzIzLCAxMC45MjAxMDVdLCBbNDUuODY3MjA4LCAxMC45MjAyNzFdLCBbNDUuODY3MDU4LCAxMC45MjA0ODldLCBbNDUuODY2ODg3LCAxMC45MjA3NTNdLCBbNDUuODY2NzA2LCAxMC45MjEwMzldLCBbNDUuODY2NTcyLCAxMC45MjEyNDddLCBbNDUuODY2NDUsIDEwLjkyMTQ1Nl0sIFs0NS44NjYzNDEsIDEwLjkyMTY4OV0sIFs0NS44NjYyMTEsIDEwLjkyMTk1XSwgWzQ1Ljg2NjAzMywgMTAuOTIyMTc3XSwgWzQ1Ljg2NTg2MiwgMTAuOTIyMzAzXSwgWzQ1Ljg2NTY4OSwgMTAuOTIyMzZdLCBbNDUuODY1NTQ3LCAxMC45MjIzNzVdLCBbNDUuODY1Mzk2LCAxMC45MjIzNjddLCBbNDUuODY1MjE3LCAxMC45MjIzMzVdLCBbNDUuODY1MDU1LCAxMC45MjIyOTddLCBbNDUuODY0OTcxLCAxMC45MjIyODRdLCBbNDUuODY0ODc3LCAxMC45MjIyODJdLCBbNDUuODY0NzYyLCAxMC45MjIyOTRdLCBbNDUuODY0NTEsIDEwLjkyMjM1Ml0sIFs0NS44NjQzMTIsIDEwLjkyMjM3NV0sIFs0NS44NjQxMDIsIDEwLjkyMjM1OV0sIFs0NS44NjM5MTksIDEwLjkyMjMyN10sIFs0NS44NjM3NDcsIDEwLjkyMjMzNF0sIFs0NS44NjM1OTEsIDEwLjkyMjM2Ml0sIFs0NS44NjM0NiwgMTAuOTIyNDA0XSwgWzQ1Ljg2MzMzMywgMTAuOTIyNDU5XSwgWzQ1Ljg2MzE2OSwgMTAuOTIyNTIyXSwgWzQ1Ljg2MzA2OCwgMTAuOTIyNTM4XSwgWzQ1Ljg2Mjk5NywgMTAuOTIyNTVdLCBbNDUuODYyODQ2LCAxMC45MjI1NjFdLCBbNDUuODYyNzYzLCAxMC45MjI1NTldLCBbNDUuODYyNTgxLCAxMC45MjI1MjddLCBbNDUuODYyNDIxLCAxMC45MjI1MDJdLCBbNDUuODYyMjQ5LCAxMC45MjI1XSwgWzQ1Ljg2MjA3MiwgMTAuOTIyNTI3XSwgWzQ1Ljg2MTg4NiwgMTAuOTIyNTY3XSwgWzQ1Ljg2MTcxNiwgMTAuOTIyNTg4XSwgWzQ1Ljg2MTU2NywgMTAuOTIyNTldLCBbNDUuODYxNDIzLCAxMC45MjI2MTFdLCBbNDUuODYxMjY0LCAxMC45MjI2ODJdLCBbNDUuODYxMTUzLCAxMC45MjI3NDRdLCBbNDUuODYxMDY0LCAxMC45MjI4MTldLCBbNDUuODYwOTY2LCAxMC45MjI5ODNdLCBbNDUuODYwOTEyLCAxMC45MjMwMjRdLCBbNDUuODYwODI1LCAxMC45MjMwMzhdLCBbNDUuODYwNzI3LCAxMC45MjMwMTVdLCBbNDUuODYwNTA2LCAxMC45MjI5MjZdLCBbNDUuODYwNDA4LCAxMC45MjI4ODRdLCBbNDUuODYwMjcsIDEwLjkyMjgyNF0sIFs0NS44NjAxNDIsIDEwLjkyMjc4XSwgWzQ1Ljg1OTk2OSwgMTAuOTIyNzQyXSwgWzQ1Ljg1OTgxMSwgMTAuOTIyNzA0XSwgWzQ1Ljg1OTY4MSwgMTAuOTIyNjc3XSwgWzQ1Ljg1OTYwOCwgMTAuOTIyNjgyXSwgWzQ1Ljg1OTQ3NCwgMTAuOTIyNzRdLCBbNDUuODU5MzQ5LCAxMC45MjI4MzFdLCBbNDUuODU5MjUxLCAxMC45MjI5MzJdLCBbNDUuODU5MTY5LCAxMC45MjMwMjFdLCBbNDUuODU4OTk0LCAxMC45MjMyNTJdLCBbNDUuODU4OTUyLCAxMC45MjMzMTldLCBbNDUuODU4ODc5LCAxMC45MjM0NDFdLCBbNDUuODU4NzgsIDEwLjkyMzYwNV0sIFs0NS44NTg2OCwgMTAuOTIzNzAxXSwgWzQ1Ljg1ODU2OSwgMTAuOTIzNzUzXSwgWzQ1Ljg1ODQ1MiwgMTAuOTIzNzY5XSwgWzQ1Ljg1ODMyNCwgMTAuOTIzNzY1XV1dXSwgInR5cGUiOiAiTXVsdGlQb2x5Z29uIn0sICJpZCI6ICIwIiwgInByb3BlcnRpZXMiOiB7ImRlc2NyaXppb25lX29yaWdpbmUiOiAiTm9uIGNvZGlmaWNhdG8iLCAiZ21sX2lkIjogIklELkFDUVVFRklTSUNIRS5TUEVDQ0hJLkFDUVVBLjU0MTg2IiwgIm5hc3AiOiAwLCAibmF0dXJhX3NwZWNjaGlvX2FjcXVhIjogIk5vbiBjb2RpZmljYXRvIiwgIm5vbWUiOiAiTEFHTyBESSBMT1BQSU8iLCAib3JpZ2luZSI6IDB9LCAidHlwZSI6ICJGZWF0dXJlIn0sIHsiZ2VvbWV0cnkiOiB7ImNvb3JkaW5hdGVzIjogW1tbWzQ1Ljk4MjQzNiwgMTAuOTIyNDEzXSwgWzQ1Ljk4MjM4LCAxMC45MjIzODVdLCBbNDUuOTgyMzI1LCAxMC45MjIzNTddLCBbNDUuOTgyMjM4LCAxMC45MjIzMzJdLCBbNDUuOTgyMTI0LCAxMC45MjIyNzNdLCBbNDUuOTgyMDI4LCAxMC45MjIyNjRdLCBbNDUuOTgxOTU1LCAxMC45MjIyMjldLCBbNDUuOTgxOTI5LCAxMC45MjIxNzNdLCBbNDUuOTgxOTUsIDEwLjkyMjEyMV0sIFs0NS45ODE5NzgsIDEwLjkyMjA4Ml0sIFs0NS45ODIwMzYsIDEwLjkyMjAwNV0sIFs0NS45ODIwNzEsIDEwLjkyMTk0N10sIFs0NS45ODIyMDgsIDEwLjkyMTgxMl0sIFs0NS45ODIyNjgsIDEwLjkyMTc5MV0sIFs0NS45ODIzNzQsIDEwLjkyMTc2MV0sIFs0NS45ODI0NjMsIDEwLjkyMTc5XSwgWzQ1Ljk4MjU1MiwgMTAuOTIxODI1XSwgWzQ1Ljk4MjY0NywgMTAuOTIxOTEzXSwgWzQ1Ljk4MjcwMSwgMTAuOTIxOTU3XSwgWzQ1Ljk4MjcyMSwgMTAuOTIyMDE3XSwgWzQ1Ljk4Mjc0MSwgMTAuOTIyMDgzXSwgWzQ1Ljk4Mjc0OSwgMTAuOTIyMTQ2XSwgWzQ1Ljk4Mjc0OCwgMTAuOTIyMjA4XSwgWzQ1Ljk4MjcyMiwgMTAuOTIyMjY2XSwgWzQ1Ljk4MjY2OSwgMTAuOTIyMzExXSwgWzQ1Ljk4MjYyNSwgMTAuOTIyMzQ5XSwgWzQ1Ljk4MjU2MywgMTAuOTIyMzY3XSwgWzQ1Ljk4MjQ4NywgMTAuOTIyMzkxXSwgWzQ1Ljk4MjQzNiwgMTAuOTIyNDEzXV1dXSwgInR5cGUiOiAiTXVsdGlQb2x5Z29uIn0sICJpZCI6ICIxIiwgInByb3BlcnRpZXMiOiB7ImRlc2NyaXppb25lX29yaWdpbmUiOiAiTm9uIGNvZGlmaWNhdG8iLCAiZ21sX2lkIjogIklELkFDUVVFRklTSUNIRS5TUEVDQ0hJLkFDUVVBLjU0MTYyIiwgIm5hc3AiOiAwLCAibmF0dXJhX3NwZWNjaGlvX2FjcXVhIjogIk5vbiBjb2RpZmljYXRvIiwgIm5vbWUiOiAiTEFHTyBERUkgQkFHQVRPSSAoRFJPKSIsICJvcmlnaW5lIjogMH0sICJ0eXBlIjogIkZlYXR1cmUifSwgeyJnZW9tZXRyeSI6IHsiY29vcmRpbmF0ZXMiOiBbW1tbNDUuOTc3MTkxLCAxMC45MzE4Nl0sIFs0NS45NzcxNDMsIDEwLjkzMTgyM10sIFs0NS45NzcxNDYsIDEwLjkzMTczN10sIFs0NS45NzcxNzQsIDEwLjkzMTcwMl0sIFs0NS45NzcxOTUsIDEwLjkzMTY3M10sIFs0NS45NzcxOTksIDEwLjkzMTU5NF0sIFs0NS45NzcxNzcsIDEwLjkzMTUzNV0sIFs0NS45NzcxNTIsIDEwLjkzMTQ3NV0sIFs0NS45NzcwODksIDEwLjkzMTQyMV0sIFs0NS45NzcwNTEsIDEwLjkzMTM2NF0sIFs0NS45NzcwMTcsIDEwLjkzMTI4MV0sIFs0NS45NzcsIDEwLjkzMTE4OF0sIFs0NS45NzcwMDksIDEwLjkzMTA4XSwgWzQ1Ljk3NzA1NywgMTAuOTMxMDI5XSwgWzQ1Ljk3NzEzMSwgMTAuOTMxMDI0XSwgWzQ1Ljk3NzE1MSwgMTAuOTMxMDUxXSwgWzQ1Ljk3NzIxNywgMTAuOTMxMTAyXSwgWzQ1Ljk3NzI3OCwgMTAuOTMxMTU2XSwgWzQ1Ljk3NzMwNSwgMTAuOTMxMTk3XSwgWzQ1Ljk3NzM2NiwgMTAuOTMxMjUxXSwgWzQ1Ljk3NzM4OCwgMTAuOTMxMjg0XSwgWzQ1Ljk3NzQyMiwgMTAuOTMxMzIxXSwgWzQ1Ljk3NzQ3MiwgMTAuOTMxMzY1XSwgWzQ1Ljk3NzUwOSwgMTAuOTMxMzYzXSwgWzQ1Ljk3NzYzNiwgMTAuOTMxMzA0XSwgWzQ1Ljk3Nzc2NywgMTAuOTMxMjM5XSwgWzQ1Ljk3Nzg5MiwgMTAuOTMxMThdLCBbNDUuOTc3OTY3LCAxMC45MzExNTldLCBbNDUuOTc3OTkyLCAxMC45MzExOTldLCBbNDUuOTc3OTgyLCAxMC45MzEyMzhdLCBbNDUuOTc3OTU3LCAxMC45MzEyODNdLCBbNDUuOTc3OTI5LCAxMC45MzEzMTldLCBbNDUuOTc3ODU3LCAxMC45MzEzNzldLCBbNDUuOTc3ODEzLCAxMC45MzE0MTRdLCBbNDUuOTc3NzUzLCAxMC45MzE0NjVdLCBbNDUuOTc3Njg4LCAxMC45MzE1MTJdLCBbNDUuOTc3NTk1LCAxMC45MzE1NzJdLCBbNDUuOTc3NTEsIDEwLjkzMTYyNl0sIFs0NS45Nzc0MiwgMTAuOTMxNjk5XSwgWzQ1Ljk3NzI5OSwgMTAuOTMxNzg0XSwgWzQ1Ljk3NzE5MSwgMTAuOTMxODZdXV1dLCAidHlwZSI6ICJNdWx0aVBvbHlnb24ifSwgImlkIjogIjIiLCAicHJvcGVydGllcyI6IHsiZGVzY3JpemlvbmVfb3JpZ2luZSI6ICJOb24gY29kaWZpY2F0byIsICJnbWxfaWQiOiAiSUQuQUNRVUVGSVNJQ0hFLlNQRUNDSEkuQUNRVUEuNTQxNjQiLCAibmFzcCI6IDAsICJuYXR1cmFfc3BlY2NoaW9fYWNxdWEiOiAiTm9uIGNvZGlmaWNhdG8iLCAibm9tZSI6ICJMQUdPIFNPTE8iLCAib3JpZ2luZSI6IDB9LCAidHlwZSI6ICJGZWF0dXJlIn0sIHsiZ2VvbWV0cnkiOiB7ImNvb3JkaW5hdGVzIjogW1tbWzQ1Ljk5NDU4NSwgMTAuODMwNTAyXSwgWzQ1Ljk5NDQ5NiwgMTAuODMwNDgyXSwgWzQ1Ljk5NDQ2NywgMTAuODMwNDU4XSwgWzQ1Ljk5NDMwNSwgMTAuODMwMTU1XSwgWzQ1Ljk5NDI1NCwgMTAuODMwMDYxXSwgWzQ1Ljk5NDE5MSwgMTAuODI5OTNdLCBbNDUuOTk0MDU4LCAxMC44Mjk3MDRdLCBbNDUuOTkzOTgyLCAxMC44Mjk1ODNdLCBbNDUuOTkzOTEzLCAxMC44Mjk0NzhdLCBbNDUuOTkzODQ5LCAxMC44MjkzODNdLCBbNDUuOTkzNzkxLCAxMC44MjkyNzZdLCBbNDUuOTkzNzQ1LCAxMC44MjkxNjJdLCBbNDUuOTkzNzA0LCAxMC44MjkwMzVdLCBbNDUuOTkzNjc4LCAxMC44Mjg5NTldLCBbNDUuOTkzNTg4LCAxMC44Mjg3NzRdLCBbNDUuOTkzNTczLCAxMC44Mjg3MTFdLCBbNDUuOTkzNjMsIDEwLjgyODY0NF0sIFs0NS45OTM2NjcsIDEwLjgyODYyNl0sIFs0NS45OTM3MTMsIDEwLjgyODYxOF0sIFs0NS45OTM3NTQsIDEwLjgyODYyM10sIFs0NS45OTM4MDIsIDEwLjgyODYzOF0sIFs0NS45OTM4NDUsIDEwLjgyODY2Nl0sIFs0NS45OTM4NzYsIDEwLjgyODcwNF0sIFs0NS45OTM5NzEsIDEwLjgyODg3OV0sIFs0NS45OTQwMjQsIDEwLjgyODk5Nl0sIFs0NS45OTQwNTQsIDEwLjgyOTA4OV0sIFs0NS45OTQwNzMsIDEwLjgyOTE1M10sIFs0NS45OTQxMDQsIDEwLjgyOTIzM10sIFs0NS45OTQxNDgsIDEwLjgyOTMzM10sIFs0NS45OTQxOCwgMTAuODI5NDMzXSwgWzQ1Ljk5NDI1LCAxMC44Mjk2MTRdLCBbNDUuOTk0MzA1LCAxMC44Mjk3MjVdLCBbNDUuOTk0Mzk4LCAxMC44Mjk4OV0sIFs0NS45OTQ0NTcsIDEwLjgzMDAwN10sIFs0NS45OTQ0OTksIDEwLjgzMDA5NF0sIFs0NS45OTQ1MzEsIDEwLjgzMDExOV0sIFs0NS45OTQ1NDUsIDEwLjgzMDExMV0sIFs0NS45OTQ1NywgMTAuODMwMDk3XSwgWzQ1Ljk5NDU2NiwgMTAuODMwMDU4XSwgWzQ1Ljk5NDU2MywgMTAuODMwMDA4XSwgWzQ1Ljk5NDU0OSwgMTAuODI5OTE2XSwgWzQ1Ljk5NDUyMSwgMTAuODI5ODEzXSwgWzQ1Ljk5NDQ5NywgMTAuODI5NzU2XSwgWzQ1Ljk5NDQ2LCAxMC44Mjk2NzJdLCBbNDUuOTk0NDEzLCAxMC44Mjk1NzhdLCBbNDUuOTk0MzU2LCAxMC44Mjk0NV0sIFs0NS45OTQzMDgsIDEwLjgyOTMzN10sIFs0NS45OTQyNjIsIDEwLjgyOTIzNl0sIFs0NS45OTQyMDcsIDEwLjgyOTEyNV0sIFs0NS45OTQxNTQsIDEwLjgyOTAyMV0sIFs0NS45OTQxMSwgMTAuODI4OTQ3XSwgWzQ1Ljk5NDA1NCwgMTAuODI4ODQ2XSwgWzQ1Ljk5NDAxNywgMTAuODI4Nzc1XSwgWzQ1Ljk5Mzk5MiwgMTAuODI4NzI4XSwgWzQ1Ljk5Mzk4NCwgMTAuODI4Njc1XSwgWzQ1Ljk5NDAxMywgMTAuODI4NjQ0XSwgWzQ1Ljk5NDA1NCwgMTAuODI4NjI5XSwgWzQ1Ljk5NDEwNywgMTAuODI4NjE4XSwgWzQ1Ljk5NDE1NSwgMTAuODI4NjE3XSwgWzQ1Ljk5NDE5NCwgMTAuODI4NjM4XSwgWzQ1Ljk5NDIzMiwgMTAuODI4Njg2XSwgWzQ1Ljk5NDI3NiwgMTAuODI4NzczXSwgWzQ1Ljk5NDM0NCwgMTAuODI4OTQzXSwgWzQ1Ljk5NDQwMywgMTAuODI5MDc0XSwgWzQ1Ljk5NDQ0OSwgMTAuODI5MTc1XSwgWzQ1Ljk5NDUxNywgMTAuODI5MzAzXSwgWzQ1Ljk5NDU2MSwgMTAuODI5NF0sIFs0NS45OTQ2NzQsIDEwLjgyOTczN10sIFs0NS45OTQ3MTcsIDEwLjgyOTg3M10sIFs0NS45OTQ3NTMsIDEwLjgyOTk5M10sIFs0NS45OTQ3ODIsIDEwLjgzMDA3N10sIFs0NS45OTQ3OTEsIDEwLjgzMDE1M10sIFs0NS45OTQ3ODEsIDEwLjgzMDIwNV0sIFs0NS45OTQ3NTgsIDEwLjgzMDIyN10sIFs0NS45OTQ2NjQsIDEwLjgzMDMxOF0sIFs0NS45OTQ2NTksIDEwLjgzMDM3MV0sIFs0NS45OTQ2NSwgMTAuODMwNDM5XSwgWzQ1Ljk5NDYyNiwgMTAuODMwNDk0XSwgWzQ1Ljk5NDU4NSwgMTAuODMwNTAyXV1dXSwgInR5cGUiOiAiTXVsdGlQb2x5Z29uIn0sICJpZCI6ICIzIiwgInByb3BlcnRpZXMiOiB7ImRlc2NyaXppb25lX29yaWdpbmUiOiAiTm9uIGNvZGlmaWNhdG8iLCAiZ21sX2lkIjogIklELkFDUVVFRklTSUNIRS5TUEVDQ0hJLkFDUVVBLjU0MTU5IiwgIm5hc3AiOiAwLCAibmF0dXJhX3NwZWNjaGlvX2FjcXVhIjogIk5vbiBjb2RpZmljYXRvIiwgIm5vbWUiOiAiTEFHTyBUT1JCSUVSQSBESSBGSUFWRVx1MDAyNyAyPyIsICJvcmlnaW5lIjogMH0sICJ0eXBlIjogIkZlYXR1cmUifSwgeyJnZW9tZXRyeSI6IHsiY29vcmRpbmF0ZXMiOiBbW1tbNDUuOTkzNjU3LCAxMC44MzM0MDddLCBbNDUuOTkzNjIxLCAxMC44MzMzNzZdLCBbNDUuOTkzNjAxLCAxMC44MzMzNDZdLCBbNDUuOTkzNTY3LCAxMC44MzMyNjldLCBbNDUuOTkzNTQ1LCAxMC44MzMyMzJdLCBbNDUuOTkzNTE4LCAxMC44MzMyMTVdLCBbNDUuOTkzNDgyLCAxMC44MzMyMDFdLCBbNDUuOTkzNDI0LCAxMC44MzMyMTVdLCBbNDUuOTkzMzk2LCAxMC44MzMyMjFdLCBbNDUuOTkzMzY0LCAxMC44MzMyMTddLCBbNDUuOTkzMzM3LCAxMC44MzMxOTNdLCBbNDUuOTkzMzE2LCAxMC44MzMxMTNdLCBbNDUuOTkzMzE0LCAxMC44MzMwNTddLCBbNDUuOTkzMzIsIDEwLjgzMjk5OF0sIFs0NS45OTMzMzIsIDEwLjgzMjkzOV0sIFs0NS45OTMzNDUsIDEwLjgzMjg4NF0sIFs0NS45OTMzNTUsIDEwLjgzMjgyOF0sIFs0NS45OTMzNjIsIDEwLjgzMjc3OV0sIFs0NS45OTMzNDcsIDEwLjgzMjcwM10sIFs0NS45OTMzMTYsIDEwLjgzMjY2OV0sIFs0NS45OTMyNTksIDEwLjgzMjYyNV0sIFs0NS45OTMyMTYsIDEwLjgzMjU4N10sIFs0NS45OTMxODEsIDEwLjgzMjUzN10sIFs0NS45OTMxNTQsIDEwLjgzMjVdLCBbNDUuOTkzMTM2LCAxMC44MzI0NjZdLCBbNDUuOTkzMTA5LCAxMC44MzI0MjZdLCBbNDUuOTkzMDgyLCAxMC44MzIzNzldLCBbNDUuOTkzMDY1LCAxMC44MzIzNDJdLCBbNDUuOTkzMDQzLCAxMC44MzIzNTJdLCBbNDUuOTkzMDEsIDEwLjgzMjI4Ml0sIFs0NS45OTI5NTcsIDEwLjgzMjE4OF0sIFs0NS45OTI5MjYsIDEwLjgzMjE0N10sIFs0NS45OTI4ODcsIDEwLjgzMjExMl0sIFs0NS45OTI4NDIsIDEwLjgzMjA4MV0sIFs0NS45OTI3OTQsIDEwLjgzMjA2Ml0sIFs0NS45OTI3MjgsIDEwLjgzMjA0N10sIFs0NS45OTI2ODcsIDEwLjgzMjAzMl0sIFs0NS45OTI2NjMsIDEwLjgzMjAxMV0sIFs0NS45OTI2MjUsIDEwLjgzMTkzXSwgWzQ1Ljk5MjYwNiwgMTAuODMxODY0XSwgWzQ1Ljk5MjU4OSwgMTAuODMxODA0XSwgWzQ1Ljk5MjU3MywgMTAuODMxNzddLCBbNDUuOTkyNTExLCAxMC44MzE2NzZdLCBbNDUuOTkyNDk5LCAxMC44MzE0OTFdLCBbNDUuOTkyNTM2LCAxMC44MzE0Nl0sIFs0NS45OTI1ODMsIDEwLjgzMTQyNV0sIFs0NS45OTI2MTcsIDEwLjgzMTQwNF0sIFs0NS45OTI2NjYsIDEwLjgzMTM4Nl0sIFs0NS45OTI2OTgsIDEwLjgzMTQwNF0sIFs0NS45OTI3MzQsIDEwLjgzMTQ0Ml0sIFs0NS45OTI3NjUsIDEwLjgzMTQ5Ml0sIFs0NS45OTI4MTEsIDEwLjgzMTU4XSwgWzQ1Ljk5Mjg2MiwgMTAuODMxNjc3XSwgWzQ1Ljk5MjkyNCwgMTAuODMxNzg4XSwgWzQ1Ljk5Mjk2OCwgMTAuODMxODgyXSwgWzQ1Ljk5MzA3NiwgMTAuODMyMDY4XSwgWzQ1Ljk5MzIyNCwgMTAuODMyMjY0XSwgWzQ1Ljk5MzI2MywgMTAuODMyMzYxXSwgWzQ1Ljk5MzMwMiwgMTAuODMyMzczXSwgWzQ1Ljk5MzM1MSwgMTAuODMyMzY1XSwgWzQ1Ljk5MzM5OSwgMTAuODMyMzYzXSwgWzQ1Ljk5MzQzMiwgMTAuODMyNDE0XSwgWzQ1Ljk5MzQ0OCwgMTAuODMyNDU3XSwgWzQ1Ljk5MzQ3NSwgMTAuODMyNDc3XSwgWzQ1Ljk5MzQ5LCAxMC44MzI0ODZdLCBbNDUuOTkzNTIyLCAxMC44MzI0ODZdLCBbNDUuOTkzNTUzLCAxMC44MzI0ODNdLCBbNDUuOTkzNTkyLCAxMC44MzI0NjhdLCBbNDUuOTkzNjMzLCAxMC44MzI0NjZdLCBbNDUuOTkzNjc2LCAxMC44MzI0OTFdLCBbNDUuOTkzNzA2LCAxMC44MzI1MThdLCBbNDUuOTkzNzM5LCAxMC44MzI1NjhdLCBbNDUuOTkzNzYxLCAxMC44MzI2MThdLCBbNDUuOTkzNzc0LCAxMC44MzI2NThdLCBbNDUuOTkzNzc4LCAxMC44MzI3MTRdLCBbNDUuOTkzNzY4LCAxMC44MzI3NTNdLCBbNDUuOTkzNzQ1LCAxMC44MzI3ODldLCBbNDUuOTkzNzE3LCAxMC44MzI4MTddLCBbNDUuOTkzNjcxLCAxMC44MzI4NDJdLCBbNDUuOTkzNjI5LCAxMC44MzI4OF0sIFs0NS45OTM2MDUsIDEwLjgzMjkxOV0sIFs0NS45OTM2MDIsIDEwLjgzMjk1OF0sIFs0NS45OTM2MjksIDEwLjgzMzAyNV0sIFs0NS45OTM2NjUsIDEwLjgzMzA2Nl0sIFs0NS45OTM2ODcsIDEwLjgzMzA4OV0sIFs0NS45OTM3MjMsIDEwLjgzMzEyN10sIFs0NS45OTM3NSwgMTAuODMzMTddLCBbNDUuOTkzNzc3LCAxMC44MzMyMzRdLCBbNDUuOTkzNzc4LCAxMC44MzMyOTNdLCBbNDUuOTkzNzczLCAxMC44MzMzMzJdLCBbNDUuOTkzNzQyLCAxMC44MzMzN10sIFs0NS45OTM2ODksIDEwLjgzMzQwNV0sIFs0NS45OTM2NTcsIDEwLjgzMzQwN11dXV0sICJ0eXBlIjogIk11bHRpUG9seWdvbiJ9LCAiaWQiOiAiNCIsICJwcm9wZXJ0aWVzIjogeyJkZXNjcml6aW9uZV9vcmlnaW5lIjogIk5vbiBjb2RpZmljYXRvIiwgImdtbF9pZCI6ICJJRC5BQ1FVRUZJU0lDSEUuU1BFQ0NISS5BQ1FVQS41NDE2MCIsICJuYXNwIjogMCwgIm5hdHVyYV9zcGVjY2hpb19hY3F1YSI6ICJOb24gY29kaWZpY2F0byIsICJub21lIjogIkxBR08gVE9SQklFUkEgREkgRklBVkVcdTAwMjcgMz8iLCAib3JpZ2luZSI6IDB9LCAidHlwZSI6ICJGZWF0dXJlIn0sIHsiZ2VvbWV0cnkiOiB7ImNvb3JkaW5hdGVzIjogW1tbWzQ2LjAxMDk3LCAxMC45NTY4MzRdLCBbNDYuMDEwODI2LCAxMC45NTY4MTldLCBbNDYuMDEwNzA1LCAxMC45NTY4MTRdLCBbNDYuMDEwNTQxLCAxMC45NTY3OThdLCBbNDYuMDEwMzQyLCAxMC45NTY3NzhdLCBbNDYuMDEwMTIxLCAxMC45NTY3MzldLCBbNDYuMDA5ODc4LCAxMC45NTY2OTldLCBbNDYuMDA5NjM5LCAxMC45NTY2NjVdLCBbNDYuMDA5Mzg3LCAxMC45NTY2MjJdLCBbNDYuMDA5MTA3LCAxMC45NTY1NjhdLCBbNDYuMDA4ODQyLCAxMC45NTY1MTVdLCBbNDYuMDA4NjExLCAxMC45NTY1MDVdLCBbNDYuMDA4MzcxLCAxMC45NTY0ODRdLCBbNDYuMDA4MTQ3LCAxMC45NTY0NjhdLCBbNDYuMDA4MDc0LCAxMC45NTY0N10sIFs0Ni4wMDc4OCwgMTAuOTU2NDddLCBbNDYuMDA3NzY3LCAxMC45NTY0OTRdLCBbNDYuMDA3NTk4LCAxMC45NTY1MjhdLCBbNDYuMDA3NDM1LCAxMC45NTY1NTVdLCBbNDYuMDA3MjExLCAxMC45NTY1MzhdLCBbNDYuMDA2OTkyLCAxMC45NTY0OTVdLCBbNDYuMDA2NzQzLCAxMC45NTY0NjJdLCBbNDYuMDA2NTc3LCAxMC45NTY0MjNdLCBbNDYuMDA2NDE1LCAxMC45NTYzNjFdLCBbNDYuMDA2MjYzLCAxMC45NTYyNDFdLCBbNDYuMDA2MDk3LCAxMC45NTYwOV0sIFs0Ni4wMDU5NTYsIDEwLjk1NjAxMl0sIFs0Ni4wMDU3OTgsIDEwLjk1NTk1N10sIFs0Ni4wMDU2MjMsIDEwLjk1NTkxMl0sIFs0Ni4wMDU0OTcsIDEwLjk1NTg5NF0sIFs0Ni4wMDUzNjcsIDEwLjk1NTg3OF0sIFs0Ni4wMDUyMzksIDEwLjk1NTg4M10sIFs0Ni4wMDUwOTcsIDEwLjk1NTg4N10sIFs0Ni4wMDQ5NjIsIDEwLjk1NTg3OF0sIFs0Ni4wMDQ4ODQsIDEwLjk1NTg2NF0sIFs0Ni4wMDQ3NzcsIDEwLjk1NTg1M10sIFs0Ni4wMDQ2NjUsIDEwLjk1NTg0NF0sIFs0Ni4wMDQ1MzUsIDEwLjk1NTgzNl0sIFs0Ni4wMDQzOTYsIDEwLjk1NTgwMV0sIFs0Ni4wMDQyMDYsIDEwLjk1NTcyNV0sIFs0Ni4wMDQwMjQsIDEwLjk1NTYzNF0sIFs0Ni4wMDM3OTYsIDEwLjk1NTUwNV0sIFs0Ni4wMDM1ODUsIDEwLjk1NTM2MV0sIFs0Ni4wMDMzOCwgMTAuOTU1MTg3XSwgWzQ2LjAwMzIzNSwgMTAuOTU1MDc2XSwgWzQ2LjAwMzExMiwgMTAuOTU1MDI1XSwgWzQ2LjAwMjkzMiwgMTAuOTU0OTQ2XSwgWzQ2LjAwMjczMywgMTAuOTU0ODc0XSwgWzQ2LjAwMjQzLCAxMC45NTQ3OTFdLCBbNDYuMDAyMjgyLCAxMC45NTQ3MzJdLCBbNDYuMDAyMTY4LCAxMC45NTQ3MTFdLCBbNDYuMDAyMDIyLCAxMC45NTQ2NV0sIFs0Ni4wMDE5MDYsIDEwLjk1NDU3Ml0sIFs0Ni4wMDE4MDQsIDEwLjk1NDQ3NV0sIFs0Ni4wMDE2OTMsIDEwLjk1NDMxNl0sIFs0Ni4wMDE1OCwgMTAuOTU0MTE3XSwgWzQ2LjAwMTQzNywgMTAuOTUzOTE0XSwgWzQ2LjAwMTMyLCAxMC45NTM3NjhdLCBbNDYuMDAxMTQxLCAxMC45NTM1NjVdLCBbNDYuMDAwOTgyLCAxMC45NTMzODJdLCBbNDYuMDAwNzk0LCAxMC45NTMxODhdLCBbNDYuMDAwNjUxLCAxMC45NTMwNDhdLCBbNDYuMDAwNDk5LCAxMC45NTI5MDFdLCBbNDYuMDAwMzk0LCAxMC45NTI4MjRdLCBbNDYuMDAwMjY2LCAxMC45NTI4MTJdLCBbNDYuMDAwMTQsIDEwLjk1Mjg2M10sIFs0Ni4wMDAwMjYsIDEwLjk1Mjg2N10sIFs0NS45OTk4ODgsIDEwLjk1Mjg1NV0sIFs0NS45OTk3NjEsIDEwLjk1Mjg0XSwgWzQ1Ljk5OTY0NiwgMTAuOTUyODA5XSwgWzQ1Ljk5OTU3NCwgMTAuOTUyNzc1XSwgWzQ1Ljk5OTQ4LCAxMC45NTI3NTddLCBbNDUuOTk5NDAyLCAxMC45NTI3NTldLCBbNDUuOTk5MzQsIDEwLjk1Mjc3OF0sIFs0NS45OTkyNCwgMTAuOTUyNzczXSwgWzQ1Ljk5OTE2NCwgMTAuOTUyNzcyXSwgWzQ1Ljk5OTEwOSwgMTAuOTUyNzc5XSwgWzQ1Ljk5OTEwOSwgMTAuOTUyNzkzXSwgWzQ1Ljk5OTA1NCwgMTAuOTUyODA0XSwgWzQ1Ljk5ODk4MiwgMTAuOTUyODQ3XSwgWzQ1Ljk5ODkxLCAxMC45NTI4OTddLCBbNDUuOTk4Nzk5LCAxMC45NTI5NjFdLCBbNDUuOTk4NzI2LCAxMC45NTI5NjVdLCBbNDUuOTk4NjI4LCAxMC45NTI5NTVdLCBbNDUuOTk4NTYzLCAxMC45NTI5ODhdLCBbNDUuOTk4NDY2LCAxMC45NTMwNl0sIFs0NS45OTgzODMsIDEwLjk1MzA4Nl0sIFs0NS45OTgzMDEsIDEwLjk1MzA2NF0sIFs0NS45OTgyMzcsIDEwLjk1MzA0NV0sIFs0NS45OTgxNjYsIDEwLjk1MzA0OF0sIFs0NS45OTgxMDksIDEwLjk1MzA4Ml0sIFs0NS45OTgwMjMsIDEwLjk1MzEyOF0sIFs0NS45OTc5NDgsIDEwLjk1MzEyOV0sIFs0NS45OTc4NzksIDEwLjk1MzEzNl0sIFs0NS45OTc3OTUsIDEwLjk1MzE5OF0sIFs0NS45OTc3MjUsIDEwLjk1MzI4N10sIFs0NS45OTc2NzYsIDEwLjk1MzM0NV0sIFs0NS45OTc1OTksIDEwLjk1MzQzNF0sIFs0NS45OTc1MzgsIDEwLjk1MzQ4N10sIFs0NS45OTc0MzgsIDEwLjk1MzQ5M10sIFs0NS45OTczNDcsIDEwLjk1MzQ1N10sIFs0NS45OTcyNywgMTAuOTUzNDI3XSwgWzQ1Ljk5NzE1OSwgMTAuOTUzMzc3XSwgWzQ1Ljk5NzAwNywgMTAuOTUzMzA5XSwgWzQ1Ljk5NjkwNSwgMTAuOTUzMjU2XSwgWzQ1Ljk5NjcxLCAxMC45NTMxNDNdLCBbNDUuOTk2NTUsIDEwLjk1MzAzNV0sIFs0NS45OTYzNzYsIDEwLjk1MjkxM10sIFs0NS45OTYxNzcsIDEwLjk1Mjc4MV0sIFs0NS45OTYwMDQsIDEwLjk1MjY1OV0sIFs0NS45OTU4MjEsIDEwLjk1MjUzN10sIFs0NS45OTU2NjMsIDEwLjk1MjQxNl0sIFs0NS45OTU1MDUsIDEwLjk1MjI5Ml0sIFs0NS45OTUzMTgsIDEwLjk1MjE0XSwgWzQ1Ljk5NTE1NiwgMTAuOTUxOTg2XSwgWzQ1Ljk5NTAzNiwgMTAuOTUxODQ0XSwgWzQ1Ljk5NDkzMiwgMTAuOTUxNzMyXSwgWzQ1Ljk5NDg0MiwgMTAuOTUxNjkyXSwgWzQ1Ljk5NDc5MSwgMTAuOTUxNjkzXSwgWzQ1Ljk5NDcwNCwgMTAuOTUxNzM2XSwgWzQ1Ljk5NDY0NCwgMTAuOTUxNjA2XSwgWzQ1Ljk5NDYzMiwgMTAuOTUxNTM2XSwgWzQ1Ljk5NDYwOSwgMTAuOTUxNDExXSwgWzQ1Ljk5NDUyMiwgMTAuOTUxMzI4XSwgWzQ1Ljk5NDQzLCAxMC45NTEyMzNdLCBbNDUuOTk0Mzc0LCAxMC45NTExNjVdLCBbNDUuOTk0MzExLCAxMC45NTEwODFdLCBbNDUuOTk0MjM2LCAxMC45NTA5NTNdLCBbNDUuOTk0MiwgMTAuOTUwODcyXSwgWzQ1Ljk5NDE4NSwgMTAuOTUwODM5XSwgWzQ1Ljk5NDEzMSwgMTAuOTUwNzA2XSwgWzQ1Ljk5NDA3MiwgMTAuOTUwNTRdLCBbNDUuOTk0MDA1LCAxMC45NTAzNTNdLCBbNDUuOTkzODgzLCAxMC45NDk5OTFdLCBbNDUuOTkzODM2LCAxMC45NDk4MjhdLCBbNDUuOTkzNzk1LCAxMC45NDk2OTVdLCBbNDUuOTkzNjQyLCAxMC45NDkzMzVdLCBbNDUuOTkzNTgsIDEwLjk0OTI0XSwgWzQ1Ljk5MzU1NSwgMTAuOTQ5MTk3XSwgWzQ1Ljk5MzUyOSwgMTAuOTQ5MTJdLCBbNDUuOTkzNDYxLCAxMC45NDg4ODVdLCBbNDUuOTkzNDUyLCAxMC45NDg3MjddLCBbNDUuOTkzNDcxLCAxMC45NDg1NzZdLCBbNDUuOTkzNDk0LCAxMC45NDg0NDNdLCBbNDUuOTkzNTMzLCAxMC45NDgyMDVdLCBbNDUuOTkzNTIyLCAxMC45NDgwN10sIFs0NS45OTM0ODEsIDEwLjk0Nzg5NF0sIFs0NS45OTM0MDcsIDEwLjk0NzcwMV0sIFs0NS45OTMzNTEsIDEwLjk0NzUzNV0sIFs0NS45OTMzMTEsIDEwLjk0NzM1OV0sIFs0NS45OTMyNywgMTAuOTQ3MTldLCBbNDUuOTkzMjQ5LCAxMC45NDY5OTldLCBbNDUuOTkzMjUxLCAxMC45NDY3ODJdLCBbNDUuOTkzMjU5LCAxMC45NDY1ODldLCBbNDUuOTkzMjY0LCAxMC45NDY0NTFdLCBbNDUuOTkzMjUsIDEwLjk0NjM0Nl0sIFs0NS45OTMyNDgsIDEwLjk0NjIxNF0sIFs0NS45OTMyNTMsIDEwLjk0NjA2XSwgWzQ1Ljk5MzI1OCwgMTAuOTQ1OTAzXSwgWzQ1Ljk5MzI1MiwgMTAuOTQ1NzM1XSwgWzQ1Ljk5MzI0LCAxMC45NDU1MjVdLCBbNDUuOTkzMjI0LCAxMC45NDUzODZdLCBbNDUuOTkzMTcxLCAxMC45NDUxOTRdLCBbNDUuOTkzMDcyLCAxMC45NDQ4NDZdLCBbNDUuOTkyOTk2LCAxMC45NDQ2MzZdLCBbNDUuOTkyODQ4LCAxMC45NDQyMTNdLCBbNDUuOTkyNzcyLCAxMC45NDQwMTddLCBbNDUuOTkyNjc2LCAxMC45NDM3ODZdLCBbNDUuOTkyNjIzLCAxMC45NDM2NjZdLCBbNDUuOTkyNTI3LCAxMC45NDM1MTVdLCBbNDUuOTkyNDQ3LCAxMC45NDM0MDNdLCBbNDUuOTkyMzM1LCAxMC45NDMyODRdLCBbNDUuOTkyMTY2LCAxMC45NDMxNTNdLCBbNDUuOTkyMDUxLCAxMC45NDMwNl0sIFs0NS45OTE4OTgsIDEwLjk0Mjk1OV0sIFs0NS45OTE3NzEsIDEwLjk0Mjg2NV0sIFs0NS45OTE2NjgsIDEwLjk0Mjc3OV0sIFs0NS45OTE1MzksIDEwLjk0MjY4Ml0sIFs0NS45OTE0MTcsIDEwLjk0MjYyMl0sIFs0NS45OTEyNzQsIDEwLjk0MjU2NF0sIFs0NS45OTExNzIsIDEwLjk0MjUyN10sIFs0NS45OTEwNzYsIDEwLjk0MjUwN10sIFs0NS45OTA5OTIsIDEwLjk0MjQ5MV0sIFs0NS45OTA5MjgsIDEwLjk0MjQ2Ml0sIFs0NS45OTA4NywgMTAuOTQyNDFdLCBbNDUuOTkwODQxLCAxMC45NDIzNDRdLCBbNDUuOTkwODMxLCAxMC45NDIyNzRdLCBbNDUuOTkwODI1LCAxMC45NDIyMDVdLCBbNDUuOTkwNzk3LCAxMC45NDIxNDJdLCBbNDUuOTkwNzQ1LCAxMC45NDIwODFdLCBbNDUuOTkwNzA1LCAxMC45NDIwNF0sIFs0NS45OTA2NzQsIDEwLjk0MTk4Nl0sIFs0NS45OTA2NTcsIDEwLjk0MTkyNl0sIFs0NS45OTA2NDksIDEwLjk0MTg1N10sIFs0NS45OTA2NTksIDEwLjk0MTc4OV0sIFs0NS45OTA2NzUsIDEwLjk0MTcxXSwgWzQ1Ljk5MDY4OSwgMTAuOTQxNjU1XSwgWzQ1Ljk5MDcxNCwgMTAuOTQxNTQ1XSwgWzQ1Ljk5MDczMywgMTAuOTQxNDE3XSwgWzQ1Ljk5MDc0NCwgMTAuOTQxMjk5XSwgWzQ1Ljk5MDc1MywgMTAuOTQxMTY1XSwgWzQ1Ljk5MDc3LCAxMC45NDEwMDhdLCBbNDUuOTkwNzg5LCAxMC45NDA4NTVdLCBbNDUuOTkwODE1LCAxMC45NDA2NzJdLCBbNDUuOTkwODM2LCAxMC45NDA1MTJdLCBbNDUuOTkwODU3LCAxMC45NDAzNzhdLCBbNDUuOTkwODgsIDEwLjk0MDI0MV0sIFs0NS45OTA5MDgsIDEwLjk0MDExNF0sIFs0NS45OTA5MjYsIDEwLjk0MDAzXSwgWzQ1Ljk5MDkyNywgMTAuOTQwMDIyXSwgWzQ1Ljk5MDk0MSwgMTAuOTM5OTU4XSwgWzQ1Ljk5MDk2LCAxMC45Mzk5MTldLCBbNDUuOTkwOTgxLCAxMC45Mzk4ODddLCBbNDUuOTkxMDE3LCAxMC45Mzk4ODJdLCBbNDUuOTkxMDQ5LCAxMC45Mzk5MTZdLCBbNDUuOTkxMDY5LCAxMC45Mzk5NTNdLCBbNDUuOTkxMDg4LCAxMC45NF0sIFs0NS45OTEwOTcsIDEwLjk0MDA0Nl0sIFs0NS45OTExMTUsIDEwLjk0MDE0NV0sIFs0NS45OTExNDksIDEwLjk0MDE4Nl0sIFs0NS45OTExODgsIDEwLjk0MDE1NV0sIFs0NS45OTEyMDUsIDEwLjk0MDEwOV0sIFs0NS45OTEyMTEsIDEwLjk0MDAzMV0sIFs0NS45OTEyMiwgMTAuOTM5OTQ2XSwgWzQ1Ljk5MTIyMywgMTAuOTM5ODldLCBbNDUuOTkxMjIyLCAxMC45Mzk4MzhdLCBbNDUuOTkxMjE2LCAxMC45Mzk3ODhdLCBbNDUuOTkxMjA4LCAxMC45Mzk3NDJdLCBbNDUuOTkxMjA1LCAxMC45Mzk2NV0sIFs0NS45OTEyMTUsIDEwLjkzOTU3Ml0sIFs0NS45OTEyNDEsIDEwLjkzOTU0XSwgWzQ1Ljk5MTI5NiwgMTAuOTM5NTE2XSwgWzQ1Ljk5MTMwMywgMTAuOTM5NTEzXSwgWzQ1Ljk5MTM2NywgMTAuOTM5NTYxXSwgWzQ1Ljk5MTM5MywgMTAuOTM5NjA4XSwgWzQ1Ljk5MTQwNCwgMTAuOTM5NjUxXSwgWzQ1Ljk5MTQxNiwgMTAuOTM5NzUzXSwgWzQ1Ljk5MTQxOSwgMTAuOTM5ODA2XSwgWzQ1Ljk5MTQxMSwgMTAuOTM5ODcxXSwgWzQ1Ljk5MTM5NSwgMTAuOTM5OTc2XSwgWzQ1Ljk5MTM3NywgMTAuOTQwMDk2XSwgWzQ1Ljk5MTM1MiwgMTAuOTQwMTk0XSwgWzQ1Ljk5MTM0MiwgMTAuOTQwMjM5XSwgWzQ1Ljk5MTMyNSwgMTAuOTQwMzE4XSwgWzQ1Ljk5MTMsIDEwLjk0MDQyMl0sIFs0NS45OTEyOCwgMTAuOTQwNTEzXSwgWzQ1Ljk5MTI2OCwgMTAuOTQwNjZdLCBbNDUuOTkxMjczLCAxMC45NDA3OThdLCBbNDUuOTkxMjg0LCAxMC45NDA4OV0sIFs0NS45OTEzMywgMTAuOTQxXSwgWzQ1Ljk5MTM4MywgMTAuOTQxMTI0XSwgWzQ1Ljk5MTQyOSwgMTAuOTQxMjU0XSwgWzQ1Ljk5MTQ2NSwgMTAuOTQxMzkzXSwgWzQ1Ljk5MTQ5LCAxMC45NDE1MTldLCBbNDUuOTkxNTA0LCAxMC45NDE2NDFdLCBbNDUuOTkxNTI0LCAxMC45NDE3ODZdLCBbNDUuOTkxNTU2LCAxMC45NDE5MjVdLCBbNDUuOTkxNjA1LCAxMC45NDIwNzFdLCBbNDUuOTkxNjYsIDEwLjk0MjIxMV0sIFs0NS45OTE3MzEsIDEwLjk0MjMxOV0sIFs0NS45OTE3OTQsIDEwLjk0MjM5NF0sIFs0NS45OTE4NDMsIDEwLjk0MjQ0MV0sIFs0NS45OTE5NjcsIDEwLjk0MjUzMl0sIFs0NS45OTIwNiwgMTAuOTQyNTk0XSwgWzQ1Ljk5MjE4LCAxMC45NDI2NjRdLCBbNDUuOTkyMjg2LCAxMC45NDI3MzFdLCBbNDUuOTkyMzg4LCAxMC45NDI3ODFdLCBbNDUuOTkyNTAyLCAxMC45NDI4MTFdLCBbNDUuOTkyNjE4LCAxMC45NDI4MjldLCBbNDUuOTkyNzI2LCAxMC45NDI4M10sIFs0NS45OTI4MjksIDEwLjk0MjgxN10sIFs0NS45OTI5MjUsIDEwLjk0Mjc5OF0sIFs0NS45OTMwMDUsIDEwLjk0Mjc3OF0sIFs0NS45OTMxMDYsIDEwLjk0Mjc2M10sIFs0NS45OTMxODIsIDEwLjk0Mjc2Ml0sIFs0NS45OTMyNTUsIDEwLjk0Mjc4OF0sIFs0NS45OTMzNTEsIDEwLjk0MjkwN10sIFs0NS45OTMzODksIDEwLjk0Mjk1NF0sIFs0NS45OTM0NDIsIDEwLjk0MzAxMl0sIFs0NS45OTM0ODcsIDEwLjk0MzA2XSwgWzQ1Ljk5MzU2NiwgMTAuOTQzMTEyXSwgWzQ1Ljk5MzYxOCwgMTAuOTQzMTRdLCBbNDUuOTkzNzA3LCAxMC45NDMxODNdLCBbNDUuOTkzNzkxLCAxMC45NDMxOTldLCBbNDUuOTkzODYyLCAxMC45NDMxOTldLCBbNDUuOTkzOTY1LCAxMC45NDMxOTNdLCBbNDUuOTk0MDc3LCAxMC45NDMyXSwgWzQ1Ljk5NDE4MiwgMTAuOTQzMjAxXSwgWzQ1Ljk5NDI3MywgMTAuOTQzMjE4XSwgWzQ1Ljk5NDM4NSwgMTAuOTQzMjUyXSwgWzQ1Ljk5NDQ3MywgMTAuOTQzMjk4XSwgWzQ1Ljk5NDU1NywgMTAuOTQzMzU3XSwgWzQ1Ljk5NDY1OCwgMTAuOTQzNDQ5XSwgWzQ1Ljk5NDc1LCAxMC45NDM1NTFdLCBbNDUuOTk0ODcsIDEwLjk0MzddLCBbNDUuOTk0OTg2LCAxMC45NDM4NTZdLCBbNDUuOTk1MTI4LCAxMC45NDM5OTZdLCBbNDUuOTk1Mjc2LCAxMC45NDQxMl0sIFs0NS45OTU0MDksIDEwLjk0NDI0M10sIFs0NS45OTU1NDksIDEwLjk0NDM1XSwgWzQ1Ljk5NTY2MiwgMTAuOTQ0NDMzXSwgWzQ1Ljk5NTc3LCAxMC45NDQ1MDddLCBbNDUuOTk1OTAxLCAxMC45NDQ1NzFdLCBbNDUuOTk1OTksIDEwLjk0NDYyXSwgWzQ1Ljk5NjA5NSwgMTAuOTQ0NzA2XSwgWzQ1Ljk5NjE1MiwgMTAuOTQ0NzYxXSwgWzQ1Ljk5NjIxMywgMTAuOTQ0NzldLCBbNDUuOTk2MjYsIDEwLjk0NDgyN10sIFs0NS45OTYyOCwgMTAuOTQ0ODY0XSwgWzQ1Ljk5NjM0NSwgMTAuOTQ1MDM0XSwgWzQ1Ljk5NjM5MSwgMTAuOTQ1MDYyXSwgWzQ1Ljk5NjQzMiwgMTAuOTQ1MDU3XSwgWzQ1Ljk5NjQ4LCAxMC45NDUwMzZdLCBbNDUuOTk2NTQ5LCAxMC45NDUwMTNdLCBbNDUuOTk2NjI0LCAxMC45NDUwNDVdLCBbNDUuOTk2NjczLCAxMC45NDUxMTNdLCBbNDUuOTk2NzA3LCAxMC45NDUxNTddLCBbNDUuOTk2NzQ3LCAxMC45NDUxOThdLCBbNDUuOTk2ODEzLCAxMC45NDUyMzNdLCBbNDUuOTk2ODc2LCAxMC45NDUyNzVdLCBbNDUuOTk2OTU2LCAxMC45NDU0MDNdLCBbNDUuOTk2OTkzLCAxMC45NDU1MjldLCBbNDUuOTk3MDM3LCAxMC45NDU1ODZdLCBbNDUuOTk3MDc1LCAxMC45NDU2NDNdLCBbNDUuOTk3MTAxLCAxMC45NDU3Ml0sIFs0NS45OTcxMzgsIDEwLjk0NTgzXSwgWzQ1Ljk5NzE4MSwgMTAuOTQ1ODY0XSwgWzQ1Ljk5NzIxLCAxMC45NDU4NzhdLCBbNDUuOTk3MzA0LCAxMC45NDU4ODJdLCBbNDUuOTk3MzY2LCAxMC45NDU4NzhdLCBbNDUuOTk3NDY5LCAxMC45NDU4NTldLCBbNDUuOTk3NTA2LCAxMC45NDU4NDRdLCBbNDUuOTk3NTU3LCAxMC45NDU4MV0sIFs0NS45OTc2MzIsIDEwLjk0NTgxM10sIFs0NS45OTc2ODIsIDEwLjk0NTg1N10sIFs0NS45OTc3MDksIDEwLjk0NTg1OF0sIFs0NS45OTc3NjksIDEwLjk0NTgzOF0sIFs0NS45OTc4MTMsIDEwLjk0NTgyM10sIFs0NS45OTc4NjIsIDEwLjk0NTg0OF0sIFs0NS45OTc5MjgsIDEwLjk0NTg5Nl0sIFs0NS45OTgwNDMsIDEwLjk0NTk4OV0sIFs0NS45OTgxMywgMTAuOTQ2MDg4XSwgWzQ1Ljk5ODIyNCwgMTAuOTQ2MjFdLCBbNDUuOTk4MzA0LCAxMC45NDYzMzRdLCBbNDUuOTk4Mzc5LCAxMC45NDY0NzhdLCBbNDUuOTk4NDQzLCAxMC45NDY2MTJdLCBbNDUuOTk4NTIsIDEwLjk0Njc1Nl0sIFs0NS45OTg2NTMsIDEwLjk0Njg3XSwgWzQ1Ljk5ODcyMSwgMTAuOTQ2ODc5XSwgWzQ1Ljk5ODc2NywgMTAuOTQ2ODc3XSwgWzQ1Ljk5ODgxMiwgMTAuOTQ2OTEyXSwgWzQ1Ljk5ODg0MSwgMTAuOTQ2OTU5XSwgWzQ1Ljk5ODg2NiwgMTAuOTQ2OTgzXSwgWzQ1Ljk5ODkyNywgMTAuOTQ3MDEyXSwgWzQ1Ljk5ODk2MywgMTAuOTQ3MDFdLCBbNDUuOTk5MDY5LCAxMC45NDY5OTRdLCBbNDUuOTk5MTA4LCAxMC45NDY5ODRdLCBbNDUuOTk5MTA4LCAxMC45NDY5OTldLCBbNDUuOTk5MTM1LCAxMC45NDY5OTRdLCBbNDUuOTk5MTkyLCAxMC45NDY5NzJdLCBbNDUuOTk5MjQyLCAxMC45NDY5NTZdLCBbNDUuOTk5MzI1LCAxMC45NDY5MzVdLCBbNDUuOTk5NDQsIDEwLjk0Njg5N10sIFs0NS45OTk1MjksIDEwLjk0Njg3Nl0sIFs0NS45OTk2MjcsIDEwLjk0Njg3NF0sIFs0NS45OTk3MDcsIDEwLjk0Njg5NV0sIFs0NS45OTk4MDMsIDEwLjk0NjkzOV0sIFs0NS45OTk5NDksIDEwLjk0Njk5N10sIFs0Ni4wMDAwNTYsIDEwLjk0NzAzNV0sIFs0Ni4wMDAxNTgsIDEwLjk0NzA2OV0sIFs0Ni4wMDAyNjEsIDEwLjk0NzE0XSwgWzQ2LjAwMDM1OCwgMTAuOTQ3MjNdLCBbNDYuMDAwNDI2LCAxMC45NDczMjZdLCBbNDYuMDAwNTg1LCAxMC45NDc0N10sIFs0Ni4wMDA2NjksIDEwLjk0NzU0XSwgWzQ2LjAwMDc5LCAxMC45NDc1ODhdLCBbNDYuMDAwOTE4LCAxMC45NDc2Ml0sIFs0Ni4wMDEwMDcsIDEwLjk0NzY1N10sIFs0Ni4wMDExMDUsIDEwLjk0NzcyNV0sIFs0Ni4wMDEyLCAxMC45NDc4MDVdLCBbNDYuMDAxMzE0LCAxMC45NDc4OTVdLCBbNDYuMDAxMzkxLCAxMC45NDc5NjJdLCBbNDYuMDAxNTE2LCAxMC45NDgwNzNdLCBbNDYuMDAxNjE2LCAxMC45NDgxNDNdLCBbNDYuMDAxNzU1LCAxMC45NDgyMjFdLCBbNDYuMDAxOTU1LCAxMC45NDgzXSwgWzQ2LjAwMjE4NCwgMTAuOTQ4Mzc5XSwgWzQ2LjAwMjQyMSwgMTAuOTQ4NDQyXSwgWzQ2LjAwMjYwMywgMTAuOTQ4NDk3XSwgWzQ2LjAwMjc3OSwgMTAuOTQ4NTk4XSwgWzQ2LjAwMjkzNSwgMTAuOTQ4NzI5XSwgWzQ2LjAwMzA3NCwgMTAuOTQ4ODM2XSwgWzQ2LjAwMzIxMSwgMTAuOTQ4OTE0XSwgWzQ2LjAwMzQxNSwgMTAuOTQ5MDY4XSwgWzQ2LjAwMzY0OSwgMTAuOTQ5Mjc1XSwgWzQ2LjAwMzg0MywgMTAuOTQ5NDE2XSwgWzQ2LjAwMzk5OSwgMTAuOTQ5NTI3XSwgWzQ2LjAwNDE3OSwgMTAuOTQ5NjcxXSwgWzQ2LjAwNDMzNiwgMTAuOTQ5ODA4XSwgWzQ2LjAwNDQ1MiwgMTAuOTQ5ODg2XSwgWzQ2LjAwNDU2NiwgMTAuOTQ5OTI3XSwgWzQ2LjAwNDY3OCwgMTAuOTQ5OTU4XSwgWzQ2LjAwNDgxNCwgMTAuOTUwMDA3XSwgWzQ2LjAwNDk3LCAxMC45NTAwMzldLCBbNDYuMDA1MDkxLCAxMC45NTAwNF0sIFs0Ni4wMDUyLCAxMC45NTAwNjVdLCBbNDYuMDA1MzAzLCAxMC45NTAxMDldLCBbNDYuMDA1NDMsIDEwLjk1MDIwM10sIFs0Ni4wMDU1NzYsIDEwLjk1MDMxN10sIFs0Ni4wMDU3MDUsIDEwLjk1MDQzOF0sIFs0Ni4wMDU4NzUsIDEwLjk1MDU5MV0sIFs0Ni4wMDYwMzUsIDEwLjk1MDcyMl0sIFs0Ni4wMDYyNTcsIDEwLjk1MDldLCBbNDYuMDA2NDI4LCAxMC45NTEwNDddLCBbNDYuMDA2NTk4LCAxMC45NTEyMTFdLCBbNDYuMDA2NzU5LCAxMC45NTEzODRdLCBbNDYuMDA2OTExLCAxMC45NTE1NDddLCBbNDYuMDA3MDU2LCAxMC45NTE3MDddLCBbNDYuMDA3MjI0LCAxMC45NTE4NzddLCBbNDYuMDA3MzY0LCAxMC45NTIwNDddLCBbNDYuMDA3NTAzLCAxMC45NTIyMV0sIFs0Ni4wMDc2MjgsIDEwLjk1MjI4OF0sIFs0Ni4wMDc3NCwgMTAuOTUyMzAzXSwgWzQ2LjAwNzg0NSwgMTAuOTUyMzQ3XSwgWzQ2LjAwODAxNSwgMTAuOTUyNDg4XSwgWzQ2LjAwODEzMywgMTAuOTUyNTcyXSwgWzQ2LjAwODE2NSwgMTAuOTUyNjA4XSwgWzQ2LjAwODIzMSwgMTAuOTUyNjYyXSwgWzQ2LjAwODI1OCwgMTAuOTUyNjYyXSwgWzQ2LjAwODMwOCwgMTAuOTUyNzEyXSwgWzQ2LjAwODM5NCwgMTAuOTUyODA2XSwgWzQ2LjAwODQ3OSwgMTAuOTUyODY2XSwgWzQ2LjAwODUxNSwgMTAuOTUyODkzXSwgWzQ2LjAwODY3MSwgMTAuOTUzMDYzXSwgWzQ2LjAwODgwMSwgMTAuOTUzMTg3XSwgWzQ2LjAwODk1NSwgMTAuOTUzMzA0XSwgWzQ2LjAwOTAyOCwgMTAuOTUzMzQ4XSwgWzQ2LjAwOTEyNiwgMTAuOTUzNDA5XSwgWzQ2LjAwOTI2NywgMTAuOTUzNDc2XSwgWzQ2LjAwOTM1NCwgMTAuOTUzNTExXSwgWzQ2LjAwOTM5NSwgMTAuOTUzNTc0XSwgWzQ2LjAwOTQxOCwgMTAuOTUzNjYxXSwgWzQ2LjAwOTQyNiwgMTAuOTUzNjg5XSwgWzQ2LjAwOTQyNywgMTAuOTUzNzU4XSwgWzQ2LjAwOTQxMywgMTAuOTUzODM0XSwgWzQ2LjAwOTM3MiwgMTAuOTUzODk1XSwgWzQ2LjAwOTMzNywgMTAuOTUzOTE0XSwgWzQ2LjAwOTI3NSwgMTAuOTUzOTEzXSwgWzQ2LjAwOTE5MywgMTAuOTUzOTI5XSwgWzQ2LjAwOTEzMywgMTAuOTUzOTU3XSwgWzQ2LjAwOTA1NSwgMTAuOTU0MDE1XSwgWzQ2LjAwOTAwMiwgMTAuOTU0MV0sIFs0Ni4wMDg5NjcsIDEwLjk1NDE3NV0sIFs0Ni4wMDg5NDYsIDEwLjk1NDI3M10sIFs0Ni4wMDg5MDgsIDEwLjk1NDM3N10sIFs0Ni4wMDg4NjksIDEwLjk1NDQ1OV0sIFs0Ni4wMDg4NzUsIDEwLjk1NDU5XSwgWzQ2LjAwODkxNiwgMTAuOTU0NjE3XSwgWzQ2LjAwODk3MywgMTAuOTU0NjE4XSwgWzQ2LjAwOTA0NCwgMTAuOTU0NjE5XSwgWzQ2LjAwOTEzNSwgMTAuOTU0NjA4XSwgWzQ2LjAwOTE5OSwgMTAuOTU0NjI1XSwgWzQ2LjAwOTIxNywgMTAuOTU0NjY4XSwgWzQ2LjAwOTIxNSwgMTAuOTU0NjkzXSwgWzQ2LjAwOTIxLCAxMC45NTQ3NTNdLCBbNDYuMDA5MTgsIDEwLjk1NDc5Ml0sIFs0Ni4wMDkxNDcsIDEwLjk1NDg0OF0sIFs0Ni4wMDkxMzUsIDEwLjk1NDldLCBbNDYuMDA5MTQsIDEwLjk1NDk0OV0sIFs0Ni4wMDkxNiwgMTAuOTU1MDEyXSwgWzQ2LjAwOTIsIDEwLjk1NTA4Ml0sIFs0Ni4wMDkyNDgsIDEwLjk1NTExNV0sIFs0Ni4wMDkzMDksIDEwLjk1NTE2Ml0sIFs0Ni4wMDkzOSwgMTAuOTU1Mzg3XSwgWzQ2LjAwOTQxMiwgMTAuOTU1NDk2XSwgWzQ2LjAwOTQzNiwgMTAuOTU1NjU0XSwgWzQ2LjAwOTQ2LCAxMC45NTU4MThdLCBbNDYuMDA5NDc1LCAxMC45NTU5NDNdLCBbNDYuMDA5NTM4LCAxMC45NTYwODldLCBbNDYuMDA5NjE1LCAxMC45NTYxODldLCBbNDYuMDA5Njc0LCAxMC45NTYyMzJdLCBbNDYuMDA5NzQ3LCAxMC45NTYyNjNdLCBbNDYuMDA5ODM4LCAxMC45NTYyOTFdLCBbNDYuMDA5OTkzLCAxMC45NTYzMjldLCBbNDYuMDEwMTI4LCAxMC45NTYzNDhdLCBbNDYuMDEwMjM1LCAxMC45NTYzNTldLCBbNDYuMDEwMzM4LCAxMC45NTYzNjFdLCBbNDYuMDEwNDUzLCAxMC45NTYzNzNdLCBbNDYuMDEwNTQ4LCAxMC45NTYzODddLCBbNDYuMDEwNjU4LCAxMC45NTY0MDZdLCBbNDYuMDEwNzg4LCAxMC45NTY0MzRdLCBbNDYuMDEwODk1LCAxMC45NTY0NTVdLCBbNDYuMDExMDEsIDEwLjk1NjQ2N10sIFs0Ni4wMTA5OSwgMTAuOTU2NjU0XSwgWzQ2LjAxMDk3LCAxMC45NTY4MzRdXV1dLCAidHlwZSI6ICJNdWx0aVBvbHlnb24ifSwgImlkIjogIjUiLCAicHJvcGVydGllcyI6IHsiZGVzY3JpemlvbmVfb3JpZ2luZSI6ICJOb24gY29kaWZpY2F0byIsICJnbWxfaWQiOiAiSUQuQUNRVUVGSVNJQ0hFLlNQRUNDSEkuQUNRVUEuNTQxNTYiLCAibmFzcCI6IDAsICJuYXR1cmFfc3BlY2NoaW9fYWNxdWEiOiAiTm9uIGNvZGlmaWNhdG8iLCAibm9tZSI6ICJMQUdPIERJIENBVkVESU5FIiwgIm9yaWdpbmUiOiAwfSwgInR5cGUiOiAiRmVhdHVyZSJ9LCB7Imdlb21ldHJ5IjogeyJjb29yZGluYXRlcyI6IFtbW1s0NS45MzU3MDksIDEwLjgxOTk1XSwgWzQ1LjkzNTYyNywgMTAuODE5OTQ2XSwgWzQ1LjkzNTU3LCAxMC44MTk5MTldLCBbNDUuOTM1NTA4LCAxMC44MTk4ODNdLCBbNDUuOTM1NDQsIDEwLjgxOTgzN10sIFs0NS45MzUzOTcsIDEwLjgxOTgwMV0sIFs0NS45MzUzNjIsIDEwLjgxOTc2MV0sIFs0NS45MzUzMzcsIDEwLjgxOTcyMl0sIFs0NS45MzUzMSwgMTAuODE5NjM5XSwgWzQ1LjkzNTI5NywgMTAuODE5NTcxXSwgWzQ1LjkzNTI5OSwgMTAuODE5NDkyXSwgWzQ1LjkzNTMzNiwgMTAuODE5MzldLCBbNDUuOTM1MzczLCAxMC44MTkzMDVdLCBbNDUuOTM1NDIxLCAxMC44MTkyMDddLCBbNDUuOTM1NDcyLCAxMC44MTkwNjddLCBbNDUuOTM1NjI4LCAxMC44MTg3MTNdLCBbNDUuOTM1NzI1LCAxMC44MTg1MjddLCBbNDUuOTM1ODA3LCAxMC44MTgzNTRdLCBbNDUuOTM1OTAyLCAxMC44MTgxMTFdLCBbNDUuOTM1OTY2LCAxMC44MTc5MjVdLCBbNDUuOTM2MDA4LCAxMC44MTc3MzJdLCBbNDUuOTM2MDUyLCAxMC44MTc0ODNdLCBbNDUuOTM2MDgzLCAxMC44MTcyNDRdLCBbNDUuOTM2MTA2LCAxMC44MTY5OThdLCBbNDUuOTM2MTM5LCAxMC44MTY3NTldLCBbNDUuOTM2MTU1LCAxMC44MTY1NzVdLCBbNDUuOTM2MTcyLCAxMC44MTYzODhdLCBbNDUuOTM2MTYzLCAxMC44MTYyMTVdLCBbNDUuOTM2MTQxLCAxMC44MTYwNTFdLCBbNDUuOTM2MTA5LCAxMC44MTU4OTNdLCBbNDUuOTM2MDYyLCAxMC44MTU3MjJdLCBbNDUuOTM2MDEyLCAxMC44MTU1NjJdLCBbNDUuOTM1OTYyLCAxMC44MTU0MDFdLCBbNDUuOTM1OTM1LCAxMC44MTUyODZdLCBbNDUuOTM1OTIxLCAxMC44MTUxNTVdLCBbNDUuOTM1OTI0LCAxMC44MTUwODFdLCBbNDUuOTM1OTI2LCAxMC44MTUwMl0sIFs0NS45MzU5NDcsIDEwLjgxNDkwMl0sIFs0NS45MzU5ODksIDEwLjgxNDc3OF0sIFs0NS45MzYwMjgsIDEwLjgxNDY3XSwgWzQ1LjkzNjA2OSwgMTAuODE0NTY1XSwgWzQ1LjkzNjE4NCwgMTAuODE0Mzc2XSwgWzQ1LjkzNjI2NSwgMTAuODE0MjYxXSwgWzQ1LjkzNjMzMywgMTAuODE0MTc2XSwgWzQ1LjkzNjQxNCwgMTAuODE0MTAxXSwgWzQ1LjkzNjQ5NiwgMTAuODE0MDMzXSwgWzQ1LjkzNjY3LCAxMC44MTM4OTNdLCBbNDUuOTM2NzI1LCAxMC44MTM4MzFdLCBbNDUuOTM2Nzk0LCAxMC44MTM3NTNdLCBbNDUuOTM2ODU2LCAxMC44MTM2OTddLCBbNDUuOTM2OTY4LCAxMC44MTM2MTJdLCBbNDUuOTM3MDIzLCAxMC44MTM1ODNdLCBbNDUuOTM3MTA1LCAxMC44MTM1NTRdLCBbNDUuOTM3MTkyLCAxMC44MTM1MzhdLCBbNDUuOTM3MjkzLCAxMC44MTM1MTJdLCBbNDUuOTM3MzY0LCAxMC44MTM0ODNdLCBbNDUuOTM3NDMsIDEwLjgxMzQ1MV0sIFs0NS45Mzc1MTcsIDEwLjgxMzQxMl0sIFs0NS45Mzc1OTcsIDEwLjgxMzM2Nl0sIFs0NS45Mzc2NTksIDEwLjgxMzMyMV0sIFs0NS45Mzc3NTEsIDEwLjgxMzIyXSwgWzQ1LjkzNzgzOCwgMTAuODEzMTI4XSwgWzQ1LjkzNzkzOSwgMTAuODEzMDQ3XSwgWzQ1LjkzODAzOSwgMTAuODEyOTk1XSwgWzQ1LjkzODE0NSwgMTAuODEyOTU2XSwgWzQ1LjkzODI1LCAxMC44MTI5M10sIFs0NS45MzgzNTcsIDEwLjgxMjkzMV0sIFs0NS45Mzg0NjcsIDEwLjgxMjk0NF0sIFs0NS45Mzg1NzksIDEwLjgxMjk3MV0sIFs0NS45Mzg2NjgsIDEwLjgxMjk5OF0sIFs0NS45Mzg3NjgsIDEwLjgxMzA0NF0sIFs0NS45Mzg4NTcsIDEwLjgxMzA5N10sIFs0NS45Mzg5MjgsIDEwLjgxMzEzXSwgWzQ1LjkzOTAxLCAxMC44MTMxNjNdLCBbNDUuOTM5MDcsIDEwLjgxMzE5XSwgWzQ1LjkzOTEzOCwgMTAuODEzMjJdLCBbNDUuOTM5MjMsIDEwLjgxMzI1Nl0sIFs0NS45MzkzMTIsIDEwLjgxMzI3Nl0sIFs0NS45MzkzOCwgMTAuODEzM10sIFs0NS45Mzk0NiwgMTAuODEzMzFdLCBbNDUuOTM5NTQ5LCAxMC44MTMzMV0sIFs0NS45Mzk2NDMsIDEwLjgxMzMxMV0sIFs0NS45Mzk3MzksIDEwLjgxMzMxNF0sIFs0NS45Mzk4MzcsIDEwLjgxMzMzMV0sIFs0NS45Mzk5NDUsIDEwLjgxMzM1NV0sIFs0NS45NDAxMjEsIDEwLjgxMzQ0MV0sIFs0NS45NDAxOTYsIDEwLjgxMzQ5XSwgWzQ1Ljk0MDI2MiwgMTAuODEzNTE0XSwgWzQ1Ljk0MDQzMywgMTAuODEzNTQ3XSwgWzQ1Ljk0MDUzOCwgMTAuODEzNTU0XSwgWzQ1Ljk0MDY1LCAxMC44MTM1NjhdLCBbNDUuOTQwNzI4LCAxMC44MTM1OTVdLCBbNDUuOTQwODIyLCAxMC44MTM2NDddLCBbNDUuOTQwOTExLCAxMC44MTM2OTddLCBbNDUuOTQxMDAyLCAxMC44MTM3NF0sIFs0NS45NDExMDksIDEwLjgxMzc3N10sIFs0NS45NDEyMDUsIDEwLjgxMzc4N10sIFs0NS45NDEzNDIsIDEwLjgxMzk0NV0sIFs0NS45NDEzMzksIDEwLjgxNDAzNF0sIFs0NS45NDEzMjYsIDEwLjgxNDA3XSwgWzQ1Ljk0MTMsIDEwLjgxNDA5OV0sIFs0NS45NDEyNjYsIDEwLjgxNDExOF0sIFs0NS45NDEyMiwgMTAuODE0MTI4XSwgWzQ1Ljk0MTE4NiwgMTAuODE0MTI4XSwgWzQ1Ljk0MTA3NCwgMTAuODE0MjQ5XSwgWzQ1Ljk0MTEwMSwgMTAuODE0MzQxXSwgWzQ1Ljk0MTE0LCAxMC44MTQ0MV0sIFs0NS45NDExNjksIDEwLjgxNDQ1OV0sIFs0NS45NDEyNDIsIDEwLjgxNDU4N10sIFs0NS45NDEyNzYsIDEwLjgxNDY4Nl0sIFs0NS45NDEyODcsIDEwLjgxNDgyN10sIFs0NS45NDEyNzksIDEwLjgxNDg3XSwgWzQ1Ljk0MTI2OCwgMTAuODE0OTI4XSwgWzQ1Ljk0MTI0OCwgMTAuODE0OTc0XSwgWzQ1Ljk0MTIwNiwgMTAuODE1MDMzXSwgWzQ1Ljk0MTE1NiwgMTAuODE1MDg4XSwgWzQ1Ljk0MTA5LCAxMC44MTUxNDRdLCBbNDUuOTQxMDI1LCAxMC44MTUxNl0sIFs0NS45NDA5NTUsIDEwLjgxNTE4Nl0sIFs0NS45NDA4ODgsIDEwLjgxNTIyNV0sIFs0NS45NDA4MDEsIDEwLjgxNTI4M10sIFs0NS45NDA3MDcsIDEwLjgxNTMzMl0sIFs0NS45NDA2NSwgMTAuODE1Mzk0XSwgWzQ1Ljk0MDU2NywgMTAuODE1NTA1XSwgWzQ1Ljk0MDM4OSwgMTAuODE1NTc2XSwgWzQ1Ljk0MDMwNCwgMTAuODE1NjM1XSwgWzQ1Ljk0MDI0MiwgMTAuODE1N10sIFs0NS45NDAxOTQsIDEwLjgxNTc3OV0sIFs0NS45NDAxMzcsIDEwLjgxNTldLCBbNDUuOTQwMTAyLCAxMC44MTYwMTRdLCBbNDUuOTQwMDY1LCAxMC44MTYxMjJdLCBbNDUuOTQwMDE3LCAxMC44MTYyMDddLCBbNDUuOTM5OTc2LCAxMC44MTYyODNdLCBbNDUuOTM5OTQ4LCAxMC44MTYzMzJdLCBbNDUuOTM5OTE4LCAxMC44MTY0XSwgWzQ1LjkzOTkwOSwgMTAuODE2NDcyXSwgWzQ1LjkzOTk0LCAxMC44MTY3ODRdLCBbNDUuOTM5OTk5LCAxMC44MTY4OTJdLCBbNDUuOTQwMDU2LCAxMC44MTY5ODFdLCBbNDUuOTQwMDk5LCAxMC44MTcwNF0sIFs0NS45NDAxNzIsIDEwLjgxNzExXSwgWzQ1Ljk0MDIzNCwgMTAuODE3MTg1XSwgWzQ1Ljk0MDMwNCwgMTAuODE3Mjc0XSwgWzQ1Ljk0MDM4NCwgMTAuODE3MzZdLCBbNDUuOTQwNDU3LCAxMC44MTc0MzldLCBbNDUuOTQwNTM3LCAxMC44MTc1MzFdLCBbNDUuOTQwNTg5LCAxMC44MTc2MV0sIFs0NS45NDA2NTcsIDEwLjgxNzcwOV0sIFs0NS45NDA3MDcsIDEwLjgxNzc3MV0sIFs0NS45NDA3NiwgMTAuODE3ODIxXSwgWzQ1Ljk0MDgwOCwgMTAuODE3ODc3XSwgWzQ1Ljk0MDgxNCwgMTAuODE3OTI5XSwgWzQ1Ljk0MDgxMiwgMTAuODE3OTk4XSwgWzQ1Ljk0MDc3NywgMTAuODE4MDRdLCBbNDUuOTQwNzI1LCAxMC44MTgwNzNdLCBbNDUuOTQwNjkxLCAxMC44MTgwNzldLCBbNDUuOTQwNjI3LCAxMC44MTgwODZdLCBbNDUuOTQwNTY1LCAxMC44MTgwODVdLCBbNDUuOTQwNTE1LCAxMC44MTgwNzhdLCBbNDUuOTQwNDE5LCAxMC44MTgwMzldLCBbNDUuOTQwMzU1LCAxMC44MTc5OTJdLCBbNDUuOTQwMjg5LCAxMC44MTc5NDNdLCBbNDUuOTQwMjA2LCAxMC44MTc4OV0sIFs0NS45NDAxMzYsIDEwLjgxNzg1NF0sIFs0NS45NDAwNTYsIDEwLjgxNzgxNF0sIFs0NS45Mzk5ODMsIDEwLjgxNzc4MV0sIFs0NS45Mzk5MDMsIDEwLjgxNzc2N10sIFs0NS45Mzk3ODksIDEwLjgxNzc0N10sIFs0NS45Mzk3MDYsIDEwLjgxNzczNF0sIFs0NS45Mzk2MTksIDEwLjgxNzc0XSwgWzQ1LjkzOTUzNywgMTAuODE3ODA1XSwgWzQ1LjkzOTQ4NywgMTAuODE3ODVdLCBbNDUuOTM5MzQyLCAxMC44MTc5ODRdLCBbNDUuOTM5MjUzLCAxMC44MTgwNTldLCBbNDUuOTM5MTM4LCAxMC44MTgxNDRdLCBbNDUuOTM5MDI2LCAxMC44MTgyMjVdLCBbNDUuOTM4OSwgMTAuODE4MzE2XSwgWzQ1LjkzODc3OSwgMTAuODE4NDExXSwgWzQ1LjkzODYwNywgMTAuODE4NTQ0XSwgWzQ1LjkzODU1NCwgMTAuODE4NTg0XSwgWzQ1LjkzODE3MiwgMTAuODE4ODQ3XSwgWzQ1LjkzODA1NSwgMTAuODE4OTM4XSwgWzQ1LjkzNzc5NiwgMTAuODE5MTVdLCBbNDUuOTM3Njc5LCAxMC44MTkyMzVdLCBbNDUuOTM3NTc5LCAxMC44MTkzMV0sIFs0NS45Mzc0NDMsIDEwLjgxOTQxN10sIFs0NS45MzczMzgsIDEwLjgxOTQ5OV0sIFs0NS45MzcyMywgMTAuODE5NTldLCBbNDUuOTM3MTQ2LCAxMC44MTk2NTJdLCBbNDUuOTM3MDY4LCAxMC44MTk3MDddLCBbNDUuOTM2OTkyLCAxMC44MTk3MzNdLCBbNDUuOTM2OTE5LCAxMC44MTk3NDZdLCBbNDUuOTM2ODMyLCAxMC44MTk3NThdLCBbNDUuOTM2NjYxLCAxMC44MTk3NzFdLCBbNDUuOTM2NTc2LCAxMC44MTk3NjRdLCBbNDUuOTM2NTAxLCAxMC44MTk3Nl0sIFs0NS45MzYzODksIDEwLjgxOTc2OV0sIFs0NS45MzYzMTYsIDEwLjgxOTc3Nl0sIFs0NS45MzYyNDIsIDEwLjgxOTc4OF0sIFs0NS45MzYxNTMsIDEwLjgxOTgwNF0sIFs0NS45MzYwOTEsIDEwLjgxOTgxN10sIFs0NS45MzU5OTUsIDEwLjgxOTg0Nl0sIFs0NS45MzU5MDYsIDEwLjgxOTg4Ml0sIFs0NS45MzU3OTIsIDEwLjgxOTk0XSwgWzQ1LjkzNTcwOSwgMTAuODE5OTVdXV1dLCAidHlwZSI6ICJNdWx0aVBvbHlnb24ifSwgImlkIjogIjYiLCAicHJvcGVydGllcyI6IHsiZGVzY3JpemlvbmVfb3JpZ2luZSI6ICJOb24gY29kaWZpY2F0byIsICJnbWxfaWQiOiAiSUQuQUNRVUVGSVNJQ0hFLlNQRUNDSEkuQUNRVUEuNTQxNzYiLCAibmFzcCI6IDAsICJuYXR1cmFfc3BlY2NoaW9fYWNxdWEiOiAiTm9uIGNvZGlmaWNhdG8iLCAibm9tZSI6ICJMQUdPIERJIFRFTk5PIiwgIm9yaWdpbmUiOiAwfSwgInR5cGUiOiAiRmVhdHVyZSJ9LCB7Imdlb21ldHJ5IjogeyJjb29yZGluYXRlcyI6IFtbW1s0NS44Mzc2MywgMTAuODI1ODUzXSwgWzQ1LjgzNzcxLCAxMC44MjU5XSwgWzQ1LjgzNzc5OSwgMTAuODI1OTExXSwgWzQ1LjgzNzkzMSwgMTAuODI1OTFdLCBbNDUuODM4MDY2LCAxMC44MjU5MzhdLCBbNDUuODM4MTYyLCAxMC44MjU5NTNdLCBbNDUuODM4MzAxLCAxMC44MjU5NTVdLCBbNDUuODM4NDQ3LCAxMC44MjU5OTZdLCBbNDUuODM4NTk4LCAxMC44MjYwNjFdLCBbNDUuODM4NzIzLCAxMC44MjYwODJdLCBbNDUuODM4ODc2LCAxMC44MjYxMjddLCBbNDUuODM5MTIsIDEwLjgyNjIyM10sIFs0NS44MzkzMTEsIDEwLjgyNjMwNF0sIFs0NS44Mzk0ODEsIDEwLjgyNjI5XSwgWzQ1LjgzOTY4MiwgMTAuODI2MjYxXSwgWzQ1LjgzOTk2LCAxMC44MjYyOTRdLCBbNDUuODQwNDA1LCAxMC44MjYyMDNdLCBbNDUuODQwNTU2LCAxMC44MjYxNDNdLCBbNDUuODQwNzg5LCAxMC44MjYxNjZdLCBbNDUuODQwOTQyLCAxMC44MjYxNzhdLCBbNDUuODQxMDU2LCAxMC44MjYxNDRdLCBbNDUuODQxMTc1LCAxMC44MjYxNDJdLCBbNDUuODQxMjY0LCAxMC44MjYxN10sIFs0NS44NDEzMzUsIDEwLjgyNjE3MV0sIFs0NS44NDE0MjksIDEwLjgyNjE1OV0sIFs0NS44NDE1NCwgMTAuODI2MjE3XSwgWzQ1Ljg0MTYyNCwgMTAuODI2MzA2XSwgWzQ1Ljg0MTgzOCwgMTAuODI2NDE3XSwgWzQ1Ljg0MTk4NCwgMTAuODI2NTE1XSwgWzQ1Ljg0MjI1MywgMTAuODI2NjI3XSwgWzQ1Ljg0MjQwNiwgMTAuODI2NjQ5XSwgWzQ1Ljg0MjYwNCwgMTAuODI2NjkxXSwgWzQ1Ljg0Mjc2OCwgMTAuODI2NzQyXSwgWzQ1Ljg0MjkzLCAxMC44MjY4MDFdLCBbNDUuODQzMTEyLCAxMC44MjY5NTRdLCBbNDUuODQzMzE2LCAxMC44MjcxMzddLCBbNDUuODQzNDkzLCAxMC44MjcyNzRdLCBbNDUuODQzNzI5LCAxMC44Mjc1MjZdLCBbNDUuODQzOTc2LCAxMC44Mjc4MzFdLCBbNDUuODQ0MjYxLCAxMC44MjgzMjNdLCBbNDUuODQ0MzM3LCAxMC44Mjg0NjhdLCBbNDUuODQ0NDM5LCAxMC44Mjg2MDFdLCBbNDUuODQ0NTcsIDEwLjgyODc5Ml0sIFs0NS44NDQ3LCAxMC44Mjg5MDldLCBbNDUuODQ0ODYxLCAxMC44Mjg5OTddLCBbNDUuODQ1MDQ4LCAxMC44MjkwNTJdLCBbNDUuODQ1MTgyLCAxMC44MjkxNTldLCBbNDUuODQ1MzA1LCAxMC44MjkzMDhdLCBbNDUuODQ1MzcsIDEwLjgyOTQ0Nl0sIFs0NS44NDU0NDYsIDEwLjgyOTYwOF0sIFs0NS44NDU1NTcsIDEwLjgyOTc4Nl0sIFs0NS44NDU2MTMsIDEwLjgyOTkzMV0sIFs0NS44NDU3NTMsIDEwLjgzMDE2OV0sIFs0NS44NDU4MzUsIDEwLjgzMDI3Ml0sIFs0NS44NDU5MDMsIDEwLjgzMDMwMl0sIFs0NS44NDYwNCwgMTAuODMwMzA0XSwgWzQ1Ljg0NjE2MSwgMTAuODMwMzM1XSwgWzQ1Ljg0NjMwMiwgMTAuODMwNDE5XSwgWzQ1Ljg0NjQ1MiwgMTAuODMwNDk3XSwgWzQ1Ljg0NjU1NSwgMTAuODMwNDg5XSwgWzQ1Ljg0NjY3NiwgMTAuODMwNTUzXSwgWzQ1Ljg0Njc2NSwgMTAuODMwNTc3XSwgWzQ1Ljg0NjkyMiwgMTAuODMwNTk5XSwgWzQ1Ljg0NzA2MSwgMTAuODMwNjQ0XSwgWzQ1Ljg0NzE4OSwgMTAuODMwNjg4XSwgWzQ1Ljg0NzM1NiwgMTAuODMwNzA0XSwgWzQ1Ljg0NzU3OCwgMTAuODMwNjg3XSwgWzQ1Ljg0Nzc2NywgMTAuODMwNjgxXSwgWzQ1Ljg0Nzk4MSwgMTAuODMwNzkyXSwgWzQ1Ljg0ODI4OSwgMTAuODMwOTExXSwgWzQ1Ljg0ODY4NiwgMTAuODMxMDU1XSwgWzQ1Ljg0ODk3MiwgMTAuODMxMjM5XSwgWzQ1Ljg0OTEsIDEwLjgzMTMxNF0sIFs0NS44NDk3MTMsIDEwLjgzMTQ4Nl0sIFs0NS44NDk5MTksIDEwLjgzMTU0NF0sIFs0NS44NTAxNzksIDEwLjgzMTY0M10sIFs0NS44NTA0MDIsIDEwLjgzMTc1NF0sIFs0NS44NTA2MzUsIDEwLjgzMTg1Ml0sIFs0NS44NTA4NjYsIDEwLjgzMjAyN10sIFs0NS44NTExMzQsIDEwLjgzMjE3N10sIFs0NS44NTEzNzIsIDEwLjgzMjMzOF0sIFs0NS44NTE2ODksIDEwLjgzMjQ4OF0sIFs0NS44NTE4MjEsIDEwLjgzMjU1OV0sIFs0NS44NTE5OTgsIDEwLjgzMjY2MV0sIFs0NS44NTIxNTcsIDEwLjgzMjc3OV0sIFs0NS44NTIzNzEsIDEwLjgzMjkyNF0sIFs0NS44NTI1MTYsIDEwLjgzMzAxOV0sIFs0NS44NTI2NDYsIDEwLjgzMzA5N10sIFs0NS44NTI4MTQsIDEwLjgzMzE3OV0sIFs0NS44NTMwMDEsIDEwLjgzMzIyNl0sIFs0NS44NTMxNjMsIDEwLjgzMzMyOF0sIFs0NS44NTMzNTQsIDEwLjgzMzQ1M10sIFs0NS44NTM3MjEsIDEwLjgzMzc2OF0sIFs0NS44NTM5NTEsIDEwLjgzMzg1Ml0sIFs0NS44NTQxNzYsIDEwLjgzNDA0M10sIFs0NS44NTQzMDEsIDEwLjgzNDE0MV0sIFs0NS44NTQzNzMsIDEwLjgzNDIzXSwgWzQ1Ljg1NDUwMiwgMTAuODM0MzQxXSwgWzQ1Ljg1NDcyMywgMTAuODM0Mzk1XSwgWzQ1Ljg1NDg4OCwgMTAuODM0Mzg5XSwgWzQ1Ljg1NTEwMywgMTAuODM0NDEzXSwgWzQ1Ljg1NTM1NCwgMTAuODM0NDM4XSwgWzQ1Ljg1NTUxOSwgMTAuODM0NDY4XSwgWzQ1Ljg1NTY0NywgMTAuODM0NDE1XSwgWzQ1Ljg1NTc0NSwgMTAuODM0Mjk5XSwgWzQ1Ljg1NTkzNCwgMTAuODM0MTQzXSwgWzQ1Ljg1NjE3MiwgMTAuODM0MDk5XSwgWzQ1Ljg1NjM0MiwgMTAuODM0MDU2XSwgWzQ1Ljg1NjU5OSwgMTAuODM0MDFdLCBbNDUuODU2ODE5LCAxMC44MzM5NjJdLCBbNDUuODU3MDE5LCAxMC44MzM4NTJdLCBbNDUuODU3MTIzLCAxMC44MzM3NDZdLCBbNDUuODU3MjU0LCAxMC44MzM2MjRdLCBbNDUuODU3NDE1LCAxMC44MzM1NjZdLCBbNDUuODU3NjAxLCAxMC44MzM0OTRdLCBbNDUuODU3NzU2LCAxMC44MzMzODNdLCBbNDUuODU3ODI3LCAxMC44MzMyOTldLCBbNDUuODU3OTE5LCAxMC44MzMyNDZdLCBbNDUuODU4MTA3LCAxMC44MzMyMTRdLCBbNDUuODU4MzIxLCAxMC44MzMxMl0sIFs0NS44NTg3MDcsIDEwLjgzMjkyNV0sIFs0NS44NTg4ODQsIDEwLjgzMjg4N10sIFs0NS44NTkwMjQsIDEwLjgzMjgzNF0sIFs0NS44NTkyMjcsIDEwLjgzMjg3NF0sIFs0NS44NTkzODQsIDEwLjgzMjk2Nl0sIFs0NS44NTk3MTMsIDEwLjgzMzAwNl0sIFs0NS44NTk5NDIsIDEwLjgzMjk5NF0sIFs0NS44NjAzMjksIDEwLjgzMzE4Nl0sIFs0NS44NjA2MTEsIDEwLjgzMzEzXSwgWzQ1Ljg2MDc5NSwgMTAuODMzMDQ1XSwgWzQ1Ljg2MDkyOCwgMTAuODMyOTk2XSwgWzQ1Ljg2MTAyOCwgMTAuODMzMDExXSwgWzQ1Ljg2MTE1NiwgMTAuODMzMDM3XSwgWzQ1Ljg2MTI3NSwgMTAuODMzMDcyXSwgWzQ1Ljg2MTYyMywgMTAuODMzMjJdLCBbNDUuODYxNzczLCAxMC44MzMyOThdLCBbNDUuODYxOTAxLCAxMC44MzMzODNdLCBbNDUuODYyMDIzLCAxMC44MzM0ODFdLCBbNDUuODYyMjM5LCAxMC44MzM1OV0sIFs0NS44NjIzOTcsIDEwLjgzMzY1NV0sIFs0NS44NjI1OTUsIDEwLjgzMzc0NV0sIFs0NS44NjI5MTUsIDEwLjgzMzkxMl0sIFs0NS44NjMzOTMsIDEwLjgzNDE2OF0sIFs0NS44NjM1NjQsIDEwLjgzNDIzXSwgWzQ1Ljg2MzgzNSwgMTAuODM0MzMxXSwgWzQ1Ljg2NDA2NCwgMTAuODM0MzcyXSwgWzQ1Ljg2NDMwMywgMTAuODM0NDQ5XSwgWzQ1Ljg2NDQ3MSwgMTAuODM0NTI4XSwgWzQ1Ljg2NDY1NSwgMTAuODM0NjU2XSwgWzQ1Ljg2NDkwNiwgMTAuODM0NzE3XSwgWzQ1Ljg2NTI5MywgMTAuODM0NzA2XSwgWzQ1Ljg2NTU2MiwgMTAuODM0ODFdLCBbNDUuODY1NzA3LCAxMC44MzQ5MjhdLCBbNDUuODY1OTQ5LCAxMC44MzQ5ODVdLCBbNDUuODY2MDY1LCAxMC44MzUwMjRdLCBbNDUuODY2MTc5LCAxMC44MzUwNjJdLCBbNDUuODY2MzUsIDEwLjgzNTExOF0sIFs0NS44NjY4MDIsIDEwLjgzNTA0M10sIFs0NS44NjY5MzcsIDEwLjgzNTAwNl0sIFs0NS44NjcwNTgsIDEwLjgzNTAyNV0sIFs0NS44NjcxNjIsIDEwLjgzNTExNl0sIFs0NS44NjcyNzUsIDEwLjgzNTI0OV0sIFs0NS44NjczMTQsIDEwLjgzNTMxMl0sIFs0NS44Njc0NTcsIDEwLjgzNTM4N10sIFs0NS44Njc2MjIsIDEwLjgzNTMzOF0sIFs0NS44Njc3OTcsIDEwLjgzNTI5M10sIFs0NS44Njc5ODIsIDEwLjgzNTI2NF0sIFs0NS44NjgxMTMsIDEwLjgzNTIxNV0sIFs0NS44NjgyMzUsIDEwLjgzNTE0Ml0sIFs0NS44Njg0LCAxMC44MzUwOTNdLCBbNDUuODY4NjQ1LCAxMC44MzUwNTZdLCBbNDUuODY4ODUxLCAxMC44MzUwNDddLCBbNDUuODY5MDU1LCAxMC44MzUwNjFdLCBbNDUuODY5MTg4LCAxMC44MzUwMTVdLCBbNDUuODY5MzQzLCAxMC44MzUwMzJdLCBbNDUuODY5NDI3LCAxMC44MzUxMThdLCBbNDUuODY5NTQsIDEwLjgzNTIyOV0sIFs0NS44Njk2NDQsIDEwLjgzNTMzM10sIFs0NS44Njk4MjIsIDEwLjgzNTQyOF0sIFs0NS44Njk5MjYsIDEwLjgzNTUwNl0sIFs0NS44NzAxMzUsIDEwLjgzNTY3XSwgWzQ1Ljg3MDI3MywgMTAuODM1Nzk1XSwgWzQ1Ljg3MDQzMiwgMTAuODM1OTc4XSwgWzQ1Ljg3MDU3NCwgMTAuODM2MTg3XSwgWzQ1Ljg3MDY3LCAxMC44MzYzNjldLCBbNDUuODcwNzg3LCAxMC44MzY1ODVdLCBbNDUuODcwOTUzLCAxMC44MzY4NDRdLCBbNDUuODcxMDg5LCAxMC44MzcwNF0sIFs0NS44NzExNzEsIDEwLjgzNzI2MV0sIFs0NS44NzEyOTIsIDEwLjgzNzQ2Nl0sIFs0NS44NzE0OTIsIDEwLjgzNzYzNF0sIFs0NS44NzE2MDEsIDEwLjgzNzY3Nl0sIFs0NS44NzE3MzMsIDEwLjgzNzc1MV0sIFs0NS44NzE5MTEsIDEwLjgzNzc5N10sIFs0NS44NzIwMTksIDEwLjgzNzcyOF0sIFs0NS44NzIxNiwgMTAuODM3NjQ5XSwgWzQ1Ljg3MjMyLCAxMC44Mzc2MjNdLCBbNDUuODcyNDE2LCAxMC44Mzc2OF0sIFs0NS44NzI1NTgsIDEwLjgzNzgxOF0sIFs0NS44NzI3NDEsIDEwLjgzNzg5XSwgWzQ1Ljg3Mjk5NCwgMTAuODM3OTEyXSwgWzQ1Ljg3MzE1OCwgMTAuODM3OTc4XSwgWzQ1Ljg3MzI4NiwgMTAuODM4MDJdLCBbNDUuODczNTY3LCAxMC44MzgwNDZdLCBbNDUuODczODEyLCAxMC44MzgwNTddLCBbNDUuODczOTU3LCAxMC44MzgxOThdLCBbNDUuODc0MTI5LCAxMC44MzgzMTNdLCBbNDUuODc0MjQ2LCAxMC44MzgzMzJdLCBbNDUuODc0NDUxLCAxMC44MzgzNTNdLCBbNDUuODc0NjAyLCAxMC44MzgzODldLCBbNDUuODc0NzI0LCAxMC44MzgzMDldLCBbNDUuODc0ODI1LCAxMC44MzgyNDNdLCBbNDUuODc1MDAxLCAxMC44MzgyODldLCBbNDUuODc1MTM0LCAxMC44MzgyNDNdLCBbNDUuODc1Mjk5LCAxMC44MzgyMDRdLCBbNDUuODc1NDE2LCAxMC44MzgyNDJdLCBbNDUuODc1NTM4LCAxMC44MzgzMl0sIFs0NS44NzU2NTgsIDEwLjgzODQ4N10sIFs0NS44NzU4MDUsIDEwLjgzODYzNF0sIFs0NS44NzU5NjIsIDEwLjgzODcwM10sIFs0NS44NzYwNTIsIDEwLjgzODY3Ml0sIFs0NS44NzYxNjYsIDEwLjgzODcxN10sIFs0NS44NzYyOTMsIDEwLjgzODgwNV0sIFs0NS44NzY0OTUsIDEwLjgzODkzM10sIFs0NS44NzY2NjMsIDEwLjgzOTAzOV0sIFs0NS44NzY3NzMsIDEwLjgzOTA3N10sIFs0NS44NzY5MjMsIDEwLjgzOTE0Nl0sIFs0NS44NzcwODMsIDEwLjgzOTE3NV0sIFs0NS44NzcyNDksIDEwLjgzOTIxOF0sIFs0NS44NzczNTksIDEwLjgzOTI1XSwgWzQ1Ljg3NzUzOSwgMTAuODM5MjgzXSwgWzQ1Ljg3Nzg4NSwgMTAuODM5MjMyXSwgWzQ1Ljg3ODAxLCAxMC44MzkwOV0sIFs0NS44NzgxMjgsIDEwLjgzODk4OF0sIFs0NS44NzgyNzEsIDEwLjgzODg1NF0sIFs0NS44Nzg0MjMsIDEwLjgzODcxXSwgWzQ1Ljg3ODU0MiwgMTAuODM4NDkzXSwgWzQ1Ljg3ODY1MSwgMTAuODM4Mzg3XSwgWzQ1Ljg3ODg5NywgMTAuODM4Mjk0XSwgWzQ1Ljg3OTE5LCAxMC44MzgyMDldLCBbNDUuODc5NDA2LCAxMC44MzgxNDFdLCBbNDUuODc5Njk0LCAxMC44MzgxNTFdLCBbNDUuODc5OTY5LCAxMC44MzgxMzRdLCBbNDUuODgwMjA0LCAxMC44MzgxNDldLCBbNDUuODgwNDMzLCAxMC44MzgxODNdLCBbNDUuODgwNjU0LCAxMC44MzgyMjddLCBbNDUuODgwOTEzLCAxMC44MzgyMzZdLCBbNDUuODgxMDkxLCAxMC44MzgyNTZdLCBbNDUuODgxNDQxLCAxMC44MzgyNzddLCBbNDUuODgyNDM4LCAxMC44Mzg1MzFdLCBbNDUuODgyNDgyLCAxMC44Mzg0NjZdLCBbNDUuODgyOTY1LCAxMC44Mzg2NV0sIFs0NS44ODMwMDYsIDEwLjgzODgyMV0sIFs0NS44ODMwNiwgMTAuODM4OTE3XSwgWzQ1Ljg4MzEyNiwgMTAuODM4OTYxXSwgWzQ1Ljg4MzM3NSwgMTAuODM4OTkzXSwgWzQ1Ljg4MzQ0MywgMTAuODM5MDkyXSwgWzQ1Ljg4MzU0LCAxMC44MzkyMTldLCBbNDUuODgzNjg2LCAxMC44MzkyODhdLCBbNDUuODgzOTY1LCAxMC44Mzk0ODddLCBbNDUuODg0NDE0LCAxMC44MzkzOTVdLCBbNDUuODg0Mzk0LCAxMC44MzkxMTNdLCBbNDUuODg0NiwgMTAuODM5MDg4XSwgWzQ1Ljg4NDYxNiwgMTAuODM5ODA5XSwgWzQ1Ljg4NDI0LCAxMC44Mzk4NTZdLCBbNDUuODg0MDc0LCAxMC44NDAxOTNdLCBbNDUuODgzOTgzLCAxMC44NDAyMDFdLCBbNDUuODgzOTM1LCAxMC44NDA0MTldLCBbNDUuODgzOTkxLCAxMC44NDA0NjddLCBbNDUuODgzOTM3LCAxMC44NDA3OTldLCBbNDUuODgzOTg1LCAxMC44NDA4MzNdLCBbNDUuODgzOTQxLCAxMC44NDExMzddLCBbNDUuODgzODU0LCAxMC44NDExNDhdLCBbNDUuODgzODU1LCAxMC44NDA3OThdLCBbNDUuODgzNzk4LCAxMC44NDA3OV0sIFs0NS44ODM3ODIsIDEwLjg0MTIzOF0sIFs0NS44ODM4NTUsIDEwLjg0MTI2Nl0sIFs0NS44ODM4NTMsIDEwLjg0MTY1Nl0sIFs0NS44ODQ0MTUsIDEwLjg0MTc3Nl0sIFs0NS44ODQ0MjQsIDEwLjg0MTk3Nl0sIFs0NS44ODQwNiwgMTAuODQxOTYxXSwgWzQ1Ljg4MzkyMSwgMTAuODQxOTAzXSwgWzQ1Ljg4Mzc4MiwgMTAuODQxODUxXSwgWzQ1Ljg4MzYzOSwgMTAuODQxNzk4XSwgWzQ1Ljg4MzQ2OSwgMTAuODQyNTI1XSwgWzQ1Ljg4MzQ3NCwgMTAuODQyNzM4XSwgWzQ1Ljg4NDY0MywgMTAuODQyNzE3XSwgWzQ1Ljg4NDY0OCwgMTAuODQxOTc0XSwgWzQ1Ljg4NDU3LCAxMC44NDE5ODJdLCBbNDUuODg0NTY3LCAxMC44NDE4NThdLCBbNDUuODg0NzQxLCAxMC44NDE4NjVdLCBbNDUuODg0OCwgMTAuODQzMDA5XSwgWzQ1Ljg4MzIyOCwgMTAuODQzMDddLCBbNDUuODgzMjQxLCAxMC44NDMxMzZdLCBbNDUuODgzNTIsIDEwLjg0MzExNV0sIFs0NS44ODM1MDgsIDEwLjg0MzE5XSwgWzQ1Ljg4MzU3NiwgMTAuODQzMjI1XSwgWzQ1Ljg4MzY0MiwgMTAuODQzMzAxXSwgWzQ1Ljg4MzY2OCwgMTAuODQzNDEzXSwgWzQ1Ljg4MzY5NCwgMTAuODQzNTI5XSwgWzQ1Ljg4MzY2OCwgMTAuODQzODA2XSwgWzQ1Ljg4MzU2OSwgMTAuODQzOTQ4XSwgWzQ1Ljg4MzUwNSwgMTAuODQzOTQ0XSwgWzQ1Ljg4MzQyMiwgMTAuODQzOTY1XSwgWzQ1Ljg4MzM4NywgMTAuODQ0MDEzXSwgWzQ1Ljg4MzM3MywgMTAuODQ0MDk4XSwgWzQ1Ljg4MzM0MSwgMTAuODQ0MjU0XSwgWzQ1Ljg4MzI3MywgMTAuODQ0MjZdLCBbNDUuODgzMjc0LCAxMC44NDQzNjRdLCBbNDUuODgzMjAyLCAxMC44NDQ0MzJdLCBbNDUuODgzMDQ4LCAxMC44NDQyNjFdLCBbNDUuODgyODY4LCAxMC44NDQwMDVdLCBbNDUuODgyODk1LCAxMC44NDMyXSwgWzQ1Ljg4Mjc5MiwgMTAuODQzMjA4XSwgWzQ1Ljg4MjgxOSwgMTAuODQzOTE2XSwgWzQ1Ljg4Mjc2NSwgMTAuODQzOTc3XSwgWzQ1Ljg4MzE1NiwgMTAuODQ0NTA5XSwgWzQ1Ljg4MzE0MywgMTAuODQ0NjQ2XSwgWzQ1Ljg4MzIyMiwgMTAuODQ0NjkxXSwgWzQ1Ljg4Mjc4OCwgMTAuODQ1NzUyXSwgWzQ1Ljg4MjY4MywgMTAuODQ1NzZdLCBbNDUuODgyNTEyLCAxMC44NDU2NDhdLCBbNDUuODgyMzcsIDEwLjg0NTY2OF0sIFs0NS44ODIyNDksIDEwLjg0NTY3Ml0sIFs0NS44ODIwMzQsIDEwLjg0NTQ2N10sIFs0NS44ODE5MiwgMTAuODQ1Mzk5XSwgWzQ1Ljg4MTgxMywgMTAuODQ1Mzc3XSwgWzQ1Ljg4MTYzNSwgMTAuODQ1MjkyXSwgWzQ1Ljg4MTQ2NCwgMTAuODQ1MzE4XSwgWzQ1Ljg4MTMyOCwgMTAuODQ1MzQ0XSwgWzQ1Ljg4MTIzLCAxMC44NDUzNjJdLCBbNDUuODgxMTQ1LCAxMC44NDUzOTldLCBbNDUuODgxMSwgMTAuODQ1NDg2XSwgWzQ1Ljg4MTExNywgMTAuODQ1NjA1XSwgWzQ1Ljg4MTExMywgMTAuODQ1Nzc3XSwgWzQ1Ljg4MTExMywgMTAuODQ1Nzg1XSwgWzQ1Ljg4MTI1NywgMTAuODQ2MDczXSwgWzQ1Ljg4MTM5MiwgMTAuODQ2Mjc5XSwgWzQ1Ljg4MTQ1NCwgMTAuODQ2NDU3XSwgWzQ1Ljg4MTU0MSwgMTAuODQ2ODgxXSwgWzQ1Ljg4MTUxNiwgMTAuODQ3MDQ0XSwgWzQ1Ljg4MTUwNSwgMTAuODQ3Mjk2XSwgWzQ1Ljg4MTQyNCwgMTAuODQ3NTY2XSwgWzQ1Ljg4MTI4NywgMTAuODQ4NDA4XSwgWzQ1Ljg4MTI0NiwgMTAuODQ4NjI3XSwgWzQ1Ljg4MTIzNSwgMTAuODQ4ODE2XSwgWzQ1Ljg4MTE4NCwgMTAuODQ5MDgxXSwgWzQ1Ljg4MDk3MSwgMTAuODQ5MjkyXSwgWzQ1Ljg4MDgxMywgMTAuODQ5MzE4XSwgWzQ1Ljg4MDcwMSwgMTAuODQ5Mzg0XSwgWzQ1Ljg4MDU3MSwgMTAuODQ5NDk2XSwgWzQ1Ljg4MDUzMiwgMTAuODQ5NTY1XSwgWzQ1Ljg4MDQ3NiwgMTAuODQ5NjY0XSwgWzQ1Ljg4MDUyLCAxMC44NDk4MzZdLCBbNDUuODgwNTA0LCAxMC44NDk5ODldLCBbNDUuODgwMjg3LCAxMC44NTAwNDNdLCBbNDUuODgwMTg5LCAxMC44NTAxOTJdLCBbNDUuODgwMDcxLCAxMC44NTAzNF0sIFs0NS44Nzk5OCwgMTAuODUwNTAyXSwgWzQ1Ljg3OTkzNCwgMTAuODUwNzJdLCBbNDUuODc5ODc1LCAxMC44NTA4ODNdLCBbNDUuODc5ODExLCAxMC44NTExMDRdLCBbNDUuODc5NzQyLCAxMC44NTEzOTFdLCBbNDUuODc5ODg4LCAxMC44NTE0XSwgWzQ1Ljg3OTkyOSwgMTAuODUxNDZdLCBbNDUuODc5OTA3LCAxMC44NTE1OTFdLCBbNDUuODc5ODg1LCAxMC44NTE2OThdLCBbNDUuODc5NjY1LCAxMC44NTIxNTVdLCBbNDUuODc5NjA4LCAxMC44NTIzODNdLCBbNDUuODc5NTg5LCAxMC44NTI0MzhdLCBbNDUuODc5NTM2LCAxMC44NTIyMjRdLCBbNDUuODc5NTgyLCAxMC44NTE5NzZdLCBbNDUuODc5NTUsIDEwLjg1MTc1XSwgWzQ1Ljg3OTUxLCAxMC44NTE2NDFdLCBbNDUuODc5NDI1LCAxMC44NTE3NTddLCBbNDUuODc5NDI0LCAxMC44NTIwMDldLCBbNDUuODc5Mzk5LCAxMC44NTIyMThdLCBbNDUuODc5Mzc1LCAxMC44NTI1MTJdLCBbNDUuODc5Mjk4LCAxMC44NTI2OTFdLCBbNDUuODc5MjMzLCAxMC44NTI4MzNdLCBbNDUuODc5MTM3LCAxMC44NTMwMTFdLCBbNDUuODc5MDExLCAxMC44NTM2MzFdLCBbNDUuODc4OTE0LCAxMC44NTM5MzZdLCBbNDUuODc4Nzk4LCAxMC44NTQzNl0sIFs0NS44Nzg3NDMsIDEwLjg1NDYxMV0sIFs0NS44Nzg3MDMsIDEwLjg1NDk0XSwgWzQ1Ljg3ODYyNywgMTAuODU1MTY4XSwgWzQ1Ljg3ODU2OCwgMTAuODU1MzI0XSwgWzQ1Ljg3ODQ0OCwgMTAuODU1NDY1XSwgWzQ1Ljg3ODM0NCwgMTAuODU1NTEyXSwgWzQ1Ljg3ODIyMywgMTAuODU1NTU5XSwgWzQ1Ljg3ODE5OCwgMTAuODU1NzA1XSwgWzQ1Ljg3ODMwNSwgMTAuODU1NzZdLCBbNDUuODc4Mzk0LCAxMC44NTU3NTJdLCBbNDUuODc4NDI4LCAxMC44NTU3ODNdLCBbNDUuODc4NTg4LCAxMC44NTY1XSwgWzQ1Ljg3NzczNCwgMTAuODU3MTkyXSwgWzQ1Ljg3NzQ1MywgMTAuODU2NzExXSwgWzQ1Ljg3NzkyMywgMTAuODU1NTU5XSwgWzQ1Ljg3Nzg2OCwgMTAuODU1NTI1XSwgWzQ1Ljg3NzUyOCwgMTAuODU2MzMzXSwgWzQ1Ljg3NzMyNiwgMTAuODU2Mzk3XSwgWzQ1Ljg3NzE1NywgMTAuODU2NTddLCBbNDUuODc2OTkxLCAxMC44NTY3NzldLCBbNDUuODc2ODczLCAxMC44NTY5MDRdLCBbNDUuODc2Nzc2LCAxMC44NTczNTFdLCBbNDUuODc2NjY0LCAxMC44NTc2MzddLCBbNDUuODc2NjIzLCAxMC44NTc4MzVdLCBbNDUuODc2NjA1LCAxMC44NTgwMTJdLCBbNDUuODc2NTM0LCAxMC44NTgyMDRdLCBbNDUuODc2MzY3LCAxMC44NTg0MDldLCBbNDUuODc2MjY1LCAxMC44NTg3MzFdLCBbNDUuODc2MzEyLCAxMC44NTkxMjJdLCBbNDUuODc2Mzc4LCAxMC44NTk1NjldLCBbNDUuODc2NjUyLCAxMC44NjA5NDddLCBbNDUuODc2NjI2LCAxMC44NjEyMzFdLCBbNDUuODc2NTczLCAxMC44NjE0ODJdLCBbNDUuODc2NTY4LCAxMC44NjE3MzFdLCBbNDUuODc2NjUxLCAxMC44NjIwOTZdLCBbNDUuODc2NywgMTAuODYyNDZdLCBbNDUuODc2NjgxLCAxMC44NjI3MjldLCBbNDUuODc2NzIyLCAxMC44NjI5NTJdLCBbNDUuODc2Nzk2LCAxMC44NjMzMDFdLCBbNDUuODc2ODU5LCAxMC44NjM1ODRdLCBbNDUuODc3MDA2LCAxMC44NjQwMzJdLCBbNDUuODc3MDM1LCAxMC44NjQyOTJdLCBbNDUuODc3MDY0LCAxMC44NjQ1NDhdLCBbNDUuODc3MDQ0LCAxMC44NjQ3NTddLCBbNDUuODc3LCAxMC44NjQ5NjJdLCBbNDUuODc2NjQ5LCAxMC44NjU3M10sIFs0NS44NzY0NjUsIDEwLjg2NjAxOF0sIFs0NS44NzYzMTcsIDEwLjg2NjIxNF0sIFs0NS44NzYwOTgsIDEwLjg2NjM0NF0sIFs0NS44NzU5NTMsIDEwLjg2NjQ0Ml0sIFs0NS44NzU3NzUsIDEwLjg2NjY0MV0sIFs0NS44NzU1MzUsIDEwLjg2NjgxOV0sIFs0NS44NzUzMzcsIDEwLjg2Njk2OV0sIFs0NS44NzUxODMsIDEwLjg2Njk5NV0sIFs0NS44NzUwMDMsIDEwLjg2NjkzMl0sIFs0NS44NzQ4MDgsIDEwLjg2NjgxM10sIFs0NS44NzQ1MjEsIDEwLjg2Njg2Ml0sIFs0NS44NzQzODIsIDEwLjg2NjgzM10sIFs0NS44NzQyNDksIDEwLjg2NjYwNF0sIFs0NS44NzQxNDIsIDEwLjg2NjM4M10sIFs0NS44NzQwNDMsIDEwLjg2NjI2OV0sIFs0NS44NzM5NTYsIDEwLjg2NjI0OF0sIFs0NS44NzM4ODYsIDEwLjg2NjExNV0sIFs0NS44NzM5MTEsIDEwLjg2NTk1NV0sIFs0NS44NzQwMjcsIDEwLjg2NTc5N10sIFs0NS44NzM5MDUsIDEwLjg2NTYyNF0sIFs0NS44NzM3NiwgMTAuODY1NzM2XSwgWzQ1Ljg3MzY2NywgMTAuODY1OTE3XSwgWzQ1Ljg3MzU2MywgMTAuODY2MjE5XSwgWzQ1Ljg3MzQ5NCwgMTAuODY2NDU3XSwgWzQ1Ljg3MzQ2OCwgMTAuODY2NTA5XSwgWzQ1Ljg3MzQzNSwgMTAuODY2NTE1XSwgWzQ1Ljg3MzAxOSwgMTAuODY2NTk1XSwgWzQ1Ljg3MjY4OSwgMTAuODY2NjU4XSwgWzQ1Ljg3MjY2NywgMTAuODY2NjU3XSwgWzQ1Ljg3MjU2NSwgMTAuODY2NjUyXSwgWzQ1Ljg3MjQ1OCwgMTAuODY2NjI3XSwgWzQ1Ljg3MjM4NywgMTAuODY2NjE5XSwgWzQ1Ljg3MjQyOSwgMTAuODY2OTUxXSwgWzQ1Ljg3MjUyOSwgMTAuODY3MDg0XSwgWzQ1Ljg3MjU2NywgMTAuODY3MTU3XSwgWzQ1Ljg3MjU1NywgMTAuODY3MjU4XSwgWzQ1Ljg3MjQyOCwgMTAuODY3MjgxXSwgWzQ1Ljg3MjM3LCAxMC44NjczODVdLCBbNDUuODcyMzk4LCAxMC44Njc1MjZdLCBbNDUuODcyNDY2LCAxMC44Njc3OTZdLCBbNDUuODcyNDg1LCAxMC44NjgxODZdLCBbNDUuODcyNDA2LCAxMC44Njg1NDFdLCBbNDUuODcyMzQzLCAxMC44Njg4MjVdLCBbNDUuODcyMjQ0LCAxMC44NjkwODVdLCBbNDUuODcyMTI4LCAxMC44Njk0NzVdLCBbNDUuODcxOTUxLCAxMC44Njk3ODVdLCBbNDUuODcxNjgsIDEwLjg3MDEwN10sIFs0NS44NzE0MjEsIDEwLjg3MDM1XSwgWzQ1Ljg3MTEzOCwgMTAuODcwNzE3XSwgWzQ1Ljg3MDkyMywgMTAuODcwOTU0XSwgWzQ1Ljg3MDc5NCwgMTAuODcxMDY2XSwgWzQ1Ljg3MDczNSwgMTAuODcxMjA2XSwgWzQ1Ljg3MDk2NCwgMTAuODcxNjA3XSwgWzQ1Ljg3MDg4OSwgMTAuODcxNzc5XSwgWzQ1Ljg3MDc5MiwgMTAuODcxNjc4XSwgWzQ1Ljg3MDY1MywgMTAuODcxNjQ2XSwgWzQ1Ljg3MDYxMSwgMTAuODcxMDI2XSwgWzQ1Ljg3MDU0NCwgMTAuODcxMDU3XSwgWzQ1Ljg3MDU3NywgMTAuODcxNjI1XSwgWzQ1Ljg3MDI1MywgMTAuODcyMjIzXSwgWzQ1Ljg3MDE3MywgMTAuODcyMTk1XSwgWzQ1Ljg3MDEzMSwgMTAuODcyMjc2XSwgWzQ1Ljg3MDE4MywgMTAuODcyMzQ2XSwgWzQ1Ljg2OTk2NywgMTAuODcyNjY5XSwgWzQ1Ljg2OTg5NCwgMTAuODcyNjQ3XSwgWzQ1Ljg2OTgxMiwgMTAuODcyODI2XSwgWzQ1Ljg2OTgzMiwgMTAuODcyODU1XSwgWzQ1Ljg2OTE0MSwgMTAuODc0MTk5XSwgWzQ1Ljg2OTA3NSwgMTAuODc0MTUyXSwgWzQ1Ljg2OTAxNiwgMTAuODc0MzExXSwgWzQ1Ljg2ODkxNiwgMTAuODc0MjY2XSwgWzQ1Ljg2ODgyNiwgMTAuODc0NTE2XSwgWzQ1Ljg2ODg3NCwgMTAuODc0NTM3XSwgWzQ1Ljg2ODg4MiwgMTAuODc0Njc4XSwgWzQ1Ljg2ODc4MSwgMTAuODc0OTI0XSwgWzQ1Ljg2ODU5LCAxMC44NzUzOThdLCBbNDUuODY4NzE4LCAxMC44NzU4NjldLCBbNDUuODY4NjU4LCAxMC44NzYxMDddLCBbNDUuODY4NjE2LCAxMC44NzYyMzNdLCBbNDUuODY4NTY5LCAxMC44NzYyOThdLCBbNDUuODY4NTM0LCAxMC44NzYzNzldLCBbNDUuODY4NjUxLCAxMC44NzYzOTVdLCBbNDUuODY4NzA3LCAxMC44NzYzMDFdLCBbNDUuODY4NzYyLCAxMC44NzYyMjRdLCBbNDUuODY4OTA2LCAxMC44NzYwNDNdLCBbNDUuODY5MDgxLCAxMC44NzYxNjJdLCBbNDUuODY4NTE2LCAxMC44NzY2MTFdLCBbNDUuODY4NDk2LCAxMC44NzY1MDZdLCBbNDUuODY4NDc2LCAxMC44NzY0NjZdLCBbNDUuODY4NDM3LCAxMC44NzY0NjJdLCBbNDUuODY4MzUzLCAxMC44NzY1OThdLCBbNDUuODY4MzI2LCAxMC44NzY2MDddLCBbNDUuODY4MjEzLCAxMC44NzY2OTZdLCBbNDUuODY4MTE5LCAxMC44NzY3NF0sIFs0NS44Njc5NzgsIDEwLjg3Njc5OV0sIFs0NS44Njc4MjcsIDEwLjg3NjgzNV0sIFs0NS44Njc2NzEsIDEwLjg3Njg4NF0sIFs0NS44Njc1NzMsIDEwLjg3Njg4NV0sIFs0NS44Njc0MzEsIDEwLjg3Njg4OF0sIFs0NS44NjczMzIsIDEwLjg3Njg4Nl0sIFs0NS44NjcyMjUsIDEwLjg3Njg4XSwgWzQ1Ljg2NjQ5NSwgMTAuODc2NDY4XSwgWzQ1Ljg2NjE3NiwgMTAuODc2MTZdLCBbNDUuODY1ODAzLCAxMC44NzU5MTZdLCBbNDUuODY1NTIsIDEwLjg3NTcxXSwgWzQ1Ljg2NTMwNCwgMTAuODc1NTM4XSwgWzQ1Ljg2NTEzNiwgMTAuODc1NDJdLCBbNDUuODY1LCAxMC44NzUzMzJdLCBbNDUuODY0ODk1LCAxMC44NzUyODRdLCBbNDUuODY0NzIsIDEwLjg3NTIyN10sIFs0NS44NjQ1NTcsIDEwLjg3NTIxMV0sIFs0NS44NjQzNDksIDEwLjg3NTIyMl0sIFs0NS44NjQxNzksIDEwLjg3NTI3NF0sIFs0NS44NjQwMzYsIDEwLjg3NTM3M10sIFs0NS44NjM5LCAxMC44NzU0OTRdLCBbNDUuODYzNzY2LCAxMC44NzU2NThdLCBbNDUuODYzNjI3LCAxMC44NzU3NjZdLCBbNDUuODYzNTM1LCAxMC44NzU4NTNdLCBbNDUuODYzMDM2LCAxMC44NzYxMTNdLCBbNDUuODYyOTA1LCAxMC44NzYxM10sIFs0NS44NjI4MTYsIDEwLjg3NjE2N10sIFs0NS44NjI3MTksIDEwLjg3NjE2NV0sIFs0NS44NjI1NzcsIDEwLjg3NTQwNl0sIFs0NS44NjI2NzUsIDEwLjg3NTQyMV0sIFs0NS44NjI2NjQsIDEwLjg3NTM1OV0sIFs0NS44NjI1OTYsIDEwLjg3NTI4NV0sIFs0NS44NjI0NDQsIDEwLjg3NTIyM10sIFs0NS44NjIzNzYsIDEwLjg3NTE1Nl0sIFs0NS44NjIxNzksIDEwLjg3NTEyNV0sIFs0NS44NjE5NzksIDEwLjg3NTE5OV0sIFs0NS44NjE4OTEsIDEwLjg3NTM2NF0sIFs0NS44NjE3OTMsIDEwLjg3NTUyNl0sIFs0NS44NjE1OTgsIDEwLjg3NTU4N10sIFs0NS44NjE0ODYsIDEwLjg3NTU5OF0sIFs0NS44NjEzNDEsIDEwLjg3NTY0NF0sIFs0NS44NjEyNDQsIDEwLjg3NTcyM10sIFs0NS44NjExNzMsIDEwLjg3NTc3MV0sIFs0NS44NjEwODgsIDEwLjg3NTc1M10sIFs0NS44NjA3ODQsIDEwLjg3NTI4NF0sIFs0NS44NjA2NjcsIDEwLjg3NDg5OV0sIFs0NS44NjA1MDksIDEwLjg3NDcyMl0sIFs0NS44NjAzNTEsIDEwLjg3NDUxNV0sIFs0NS44NjAxNDgsIDEwLjg3NDI1Nl0sIFs0NS44NTk5NjcsIDEwLjg3NDA1Ml0sIFs0NS44NTk4MDUsIDEwLjg3Mzg1Ml0sIFs0NS44NTk2MjcsIDEwLjg3MzU4Nl0sIFs0NS44NTk0NDksIDEwLjg3MzM1XSwgWzQ1Ljg1OTI1OCwgMTAuODcyOTkyXSwgWzQ1Ljg1OTAzNSwgMTAuODcyNjc5XSwgWzQ1Ljg1ODc2OSwgMTAuODcyMzFdLCBbNDUuODU4NTgsIDEwLjg3MjAxMV0sIFs0NS44NTgzMDgsIDEwLjg3MTU3M10sIFs0NS44NTgwODMsIDEwLjg3MTM5OF0sIFs0NS44NTc4MDUsIDEwLjg3MTA3NV0sIFs0NS44NTc1MDEsIDEwLjg3MDgyOV0sIFs0NS44NTcxOTQsIDEwLjg3MDY0OV0sIFs0NS44NTY5NjcsIDEwLjg3MDVdLCBbNDUuODU2NzUyLCAxMC44NzAzNTFdLCBbNDUuODU2NjM2LCAxMC44NzAyNV0sIFs0NS44NTYzNTUsIDEwLjg2OTk2OV0sIFs0NS44NTYxMjUsIDEwLjg2OTY3Nl0sIFs0NS44NTU4OTQsIDEwLjg2OTQ2NV0sIFs0NS44NTU2NCwgMTAuODY5MjY5XSwgWzQ1Ljg1NTQ1NSwgMTAuODY5MDQ5XSwgWzQ1Ljg1NTMwOCwgMTAuODY4ODU2XSwgWzQ1Ljg1NTAxOSwgMTAuODY4NTI5XSwgWzQ1Ljg1NDc2MywgMTAuODY4MjYyXSwgWzQ1Ljg1NDU2OCwgMTAuODY4MDc3XSwgWzQ1Ljg1NDM5NiwgMTAuODY3OTMzXSwgWzQ1Ljg1NDIyOSwgMTAuODY3NzI2XSwgWzQ1Ljg1NDA4MSwgMTAuODY3NDc3XSwgWzQ1Ljg1Mzg3NSwgMTAuODY3MjM0XSwgWzQ1Ljg1MzYyMSwgMTAuODY2OTk5XSwgWzQ1Ljg1MzM2LCAxMC44NjY4MTddLCBbNDUuODUzMTI5LCAxMC44NjY1ODldLCBbNDUuODUyOTYzLCAxMC44NjYzMTddLCBbNDUuODUyODUxLCAxMC44NjYwNDNdLCBbNDUuODUyNzEsIDEwLjg2NTU3OV0sIFs0NS44NTI1NTksIDEwLjg2NTM0Nl0sIFs0NS44NTI0MzEsIDEwLjg2NTE3XSwgWzQ1Ljg1MjMyMywgMTAuODY0OTkxXSwgWzQ1Ljg1MjE1MywgMTAuODY0NzY4XSwgWzQ1Ljg1MTg0MiwgMTAuODY0Mzk0XSwgWzQ1Ljg1MTU3MiwgMTAuODY0MTQzXSwgWzQ1Ljg1MTI3MSwgMTAuODYzODQyXSwgWzQ1Ljg1MTA1NiwgMTAuODYzNjZdLCBbNDUuODUwODcyLCAxMC44NjM0NzZdLCBbNDUuODUwNzIzLCAxMC44NjMzNTldLCBbNDUuODUwNTQ4LCAxMC44NjMyNDddLCBbNDUuODUwNDQxLCAxMC44NjMxNzZdLCBbNDUuODUwMTg2LCAxMC44NjMwMjNdLCBbNDUuODQ5ODY1LCAxMC44NjI4NzVdLCBbNDUuODQ5Njk5LCAxMC44NjI3ODZdLCBbNDUuODQ5NTcsIDEwLjg2MjcwMl0sIFs0NS44NDk0MzQsIDEwLjg2MjU4MV0sIFs0NS44NDkyNjksIDEwLjg2MjM3OF0sIFs0NS44NDkyMTUsIDEwLjg2MjMxNF0sIFs0NS44NDkxODYsIDEwLjg2MjI5OF0sIFs0NS44NDkxMDcsIDEwLjg2MjE3Nl0sIFs0NS44NDkxMDcsIDEwLjg2MjE4MV0sIFs0NS44NDkwOTMsIDEwLjg2MjE3MV0sIFs0NS44NDkwNTcsIDEwLjg2MjE0OF0sIFs0NS44NDkwMiwgMTAuODYyMTU4XSwgWzQ1Ljg0OTAxNSwgMTAuODYyMjY1XSwgWzQ1Ljg0OTAwNCwgMTAuODYyMzMxXSwgWzQ1Ljg0ODk3MywgMTAuODYyNDEyXSwgWzQ1Ljg0ODk0OCwgMTAuODYyNDMxXSwgWzQ1Ljg0ODg4NywgMTAuODYyNDIxXSwgWzQ1Ljg0ODg1LCAxMC44NjI0M10sIFs0NS44NDg4MjcsIDEwLjg2MjQ2OV0sIFs0NS44NDg4MTMsIDEwLjg2MjU2N10sIFs0NS44NDg4MDUsIDEwLjg2MjY2Ml0sIFs0NS44NDg2ODksIDEwLjg2Mjk4OF0sIFs0NS44NDg2MjUsIDEwLjg2MzEwMl0sIFs0NS44NDg1NDIsIDEwLjg2MzIzMl0sIFs0NS44NDg0NzUsIDEwLjg2MzMxM10sIFs0NS44NDg0MDQsIDEwLjg2MzM3NF0sIFs0NS44NDgzMTcsIDEwLjg2MzQzMl0sIFs0NS44NDgyMjMsIDEwLjg2MzQ5M10sIFs0NS44NDgxNSwgMTAuODYzNTQxXSwgWzQ1Ljg0ODA2MywgMTAuODYzNTg2XSwgWzQ1Ljg0ODAwMywgMTAuODYzNjA1XSwgWzQ1Ljg0Nzk0OCwgMTAuODYzNjE0XSwgWzQ1Ljg0Nzg2MiwgMTAuODYzNTcxXSwgWzQ1Ljg0Nzc5MSwgMTAuODYzNTExXSwgWzQ1Ljg0NzczLCAxMC44NjM0MzVdLCBbNDUuODQ3NjU3LCAxMC44NjMzNDNdLCBbNDUuODQ3NTk4LCAxMC44NjMyNzddLCBbNDUuODQ3NTE4LCAxMC44NjMyMTddLCBbNDUuODQ3NDQ4LCAxMC44NjMyMDNdLCBbNDUuODQ3Mzc1LCAxMC44NjMyMDldLCBbNDUuODQ3MjgxLCAxMC44NjMyMTFdLCBbNDUuODQ3MTU1LCAxMC44NjMyMTZdLCBbNDUuODQ2NzQ0LCAxMC44NjMxNDZdLCBbNDUuODQ2NjYyLCAxMC44NjMwOTldLCBbNDUuODQ2NTU4LCAxMC44NjMwMzldLCBbNDUuODQ2NDU1LCAxMC44NjI5NzNdLCBbNDUuODQ2Mzg1LCAxMC44NjI5MjNdLCBbNDUuODQ2MjY2LCAxMC44NjI4NDNdLCBbNDUuODQ2MTU1LCAxMC44NjI3NTNdLCBbNDUuODQ2MDM5LCAxMC44NjI2NTddLCBbNDUuODQ1OTYxLCAxMC44NjI1ODhdLCBbNDUuODQ1NzgyLCAxMC44NjI0MzJdLCBbNDUuODQ1NzEzLCAxMC44NjI0MTVdLCBbNDUuODQ1NjM4LCAxMC44NjI0M10sIFs0NS44NDU1NTMsIDEwLjg2MjQ2Ml0sIFs0NS44NDU0NjYsIDEwLjg2MjQ5N10sIFs0NS44NDUzNTIsIDEwLjg2MjUzNV0sIFs0NS44NDUyMSwgMTAuODYyNTg2XSwgWzQ1Ljg0NTE1OSwgMTAuODYyNjE1XSwgWzQ1Ljg0NTEyNywgMTAuODYyNjQ0XSwgWzQ1Ljg0NTA5LCAxMC44NjI3MDJdLCBbNDUuODQ1MDc5LCAxMC44NjI3NTRdLCBbNDUuODQ1MDc0LCAxMC44NjI4M10sIFs0NS44NDUwNzEsIDEwLjg2Mjg5OF0sIFs0NS44NDUwNjgsIDEwLjg2Mjk4M10sIFs0NS44NDUwNjMsIDEwLjg2MzA2NV0sIFs0NS44NDUwNjUsIDEwLjg2MzE1XSwgWzQ1Ljg0NTA2MywgMTAuODYzMjEyXSwgWzQ1Ljg0NTA0OSwgMTAuODYzMjU4XSwgWzQ1Ljg0NTAxLCAxMC44NjMzMTZdLCBbNDUuODQ0OTY0LCAxMC44NjMzNDVdLCBbNDUuODQ0ODY1LCAxMC44NjMzNzNdLCBbNDUuODQ0NzQ0LCAxMC44NjM0MTVdLCBbNDUuODQ0NTc3LCAxMC44NjM0NzVdLCBbNDUuODQ0NDkyLCAxMC44NjM0OTNdLCBbNDUuODQ0Mzc2LCAxMC44NjM0NTldLCBbNDUuODQ0MjU1LCAxMC44NjMzODZdLCBbNDUuODQ0MTI2LCAxMC44NjMyNjddLCBbNDUuODQ0MDA4LCAxMC44NjMxNDhdLCBbNDUuODQzODkyLCAxMC44NjMwMjJdLCBbNDUuODQzNzY3LCAxMC44NjI4ODRdLCBbNDUuODQzNjMzLCAxMC44NjI3NDhdLCBbNDUuODQzNTM1LCAxMC44NjI2MzldLCBbNDUuODQzNDQsIDEwLjg2MjU1Nl0sIFs0NS44NDMzMzMsIDEwLjg2MjQ5XSwgWzQ1Ljg0MzIyNSwgMTAuODYyNDU5XSwgWzQ1Ljg0MzEyLCAxMC44NjI0NTRdLCBbNDUuODQzMDA2LCAxMC44NjI0Nl0sIFs0NS44NDI5MTksIDEwLjg2MjQ2OV0sIFs0NS44NDI3OTgsIDEwLjg2MjQ1NF0sIFs0NS44NDI3MDMsIDEwLjg2MjM4MV0sIFs0NS44NDI1NjYsIDEwLjg2MjMwMV0sIFs0NS44NDI0MzQsIDEwLjg2MjIxOF0sIFs0NS44NDIzNTIsIDEwLjg2MjE2MV0sIFs0NS44NDIxODgsIDEwLjg2MjA1Ml0sIFs0NS44NDIwNDksIDEwLjg2MTkzNl0sIFs0NS44NDE5MTcsIDEwLjg2MTgxM10sIFs0NS44NDE3OTcsIDEwLjg2MTcwNF0sIFs0NS44NDE2OTIsIDEwLjg2MTYxNF0sIFs0NS44NDE2MDEsIDEwLjg2MTU2MV0sIFs0NS44NDE0NzQsIDEwLjg2MTQ5NF0sIFs0NS44NDEzNDgsIDEwLjg2MTQyNF0sIFs0NS44NDEyNDgsIDEwLjg2MTM2MV0sIFs0NS44NDExNTMsIDEwLjg2MTI4MV0sIFs0NS44NDEwNjIsIDEwLjg2MTE4OV0sIFs0NS44NDA5OCwgMTAuODYxMTE2XSwgWzQ1Ljg0MDg3MSwgMTAuODYxMDJdLCBbNDUuODQwNzYxLCAxMC44NjA5MzNdLCBbNDUuODQwNjIzLCAxMC44NjA4NDRdLCBbNDUuODQwNDc5LCAxMC44NjA3N10sIFs0NS44NDAzNjMsIDEwLjg2MDczXSwgWzQ1Ljg0MDE3NiwgMTAuODYwNjYyXSwgWzQ1Ljg0MDEwMywgMTAuODYwNjM1XSwgWzQ1Ljg0MDA1NSwgMTAuODYwNjE1XSwgWzQ1Ljg0MDAwNywgMTAuODYwNTkyXSwgWzQ1LjgzOTk2OCwgMTAuODYwNTY4XSwgWzQ1LjgzOTkzNCwgMTAuODYwNTM4XSwgWzQ1LjgzOTg5NiwgMTAuODYwNDM3XSwgWzQ1LjgzOTg5OCwgMTAuODYwMzg4XSwgWzQ1LjgzOTkxLCAxMC44NjAzMzJdLCBbNDUuODM5OTI2LCAxMC44NjAyOF0sIFs0NS44Mzk5NDMsIDEwLjg2MDIzMV0sIFs0NS44Mzk5NTIsIDEwLjg2MDE4NV0sIFs0NS44Mzk5NTksIDEwLjg2MDE0Nl0sIFs0NS44Mzk5NjIsIDEwLjg2MDEwN10sIFs0NS44Mzk5NTUsIDEwLjg2MDA0OF0sIFs0NS44Mzk5MzcsIDEwLjg1OTk5Ml0sIFs0NS44Mzk4OTIsIDEwLjg1OTkyM10sIFs0NS44Mzk4NDksIDEwLjg1OTg4XSwgWzQ1LjgzOTc4NSwgMTAuODU5ODRdLCBbNDUuODM5NzAxLCAxMC44NTk4MDNdLCBbNDUuODM5NTgyLCAxMC44NTk3NjldLCBbNDUuODM5NDg2LCAxMC44NTk3NDVdLCBbNDUuODM5Mzk1LCAxMC44NTk3MzRdLCBbNDUuODM5Mjk3LCAxMC44NTk3Ml0sIFs0NS44MzkyMjgsIDEwLjg1OTcwM10sIFs0NS44MzkxNDYsIDEwLjg1OTY4M10sIFs0NS44MzkwNDgsIDEwLjg1OTY2Ml0sIFs0NS44Mzg5OTgsIDEwLjg1OTY1NV0sIFs0NS44Mzg4ODgsIDEwLjg1OTYzMV0sIFs0NS44Mzg2ODksIDEwLjg1OTYzOF0sIFs0NS44Mzg1NzcsIDEwLjg1OTY0N10sIFs0NS44Mzg0NjMsIDEwLjg1OTY0Nl0sIFs0NS44MzgyNDQsIDEwLjg1OTYwN10sIFs0NS44MzgwOTgsIDEwLjg1OTU2XSwgWzQ1LjgzNzk3NSwgMTAuODU5NTIyXSwgWzQ1LjgzNzgyMiwgMTAuODU5NDYyXSwgWzQ1LjgzNzcwNiwgMTAuODU5NDI1XSwgWzQ1LjgzNzYxLCAxMC44NTkzOTFdLCBbNDUuODM3NDcxLCAxMC44NTkzMjRdLCBbNDUuODM3MzkxLCAxMC44NTkyNjddLCBbNDUuODM3MzA1LCAxMC44NTkyMDhdLCBbNDUuODM3MjE2LCAxMC44NTkxMzVdLCBbNDUuODM3MTE4LCAxMC44NTkwNDhdLCBbNDUuODM3MDQzLCAxMC44NTg5NjNdLCBbNDUuODM2OTY0LCAxMC44NTg4NzddLCBbNDUuODM2NzY2LCAxMC44NTg3NDddLCBbNDUuODM2NjM2LCAxMC44NTg3MzJdLCBbNDUuODM2NTM1LCAxMC44NTg3NDFdLCBbNDUuODM2Mjg5LCAxMC44NTg3MDZdLCBbNDUuODM2MTU0LCAxMC44NTg2NjhdLCBbNDUuODM2MDMxLCAxMC44NTg2NDFdLCBbNDUuODM1OTEsIDEwLjg1ODYxNl0sIFs0NS44MzU3OTMsIDEwLjg1ODU5Nl0sIFs0NS44MzU2ODgsIDEwLjg1ODU3MV0sIFs0NS44MzU1MiwgMTAuODU4NTI0XSwgWzQ1LjgzNTQ3NCwgMTAuODU4NTA0XSwgWzQ1LjgzNTM2MiwgMTAuODU4NDg2XSwgWzQ1LjgzNTMxNCwgMTAuODU4NTA1XSwgWzQ1LjgzNTI2OCwgMTAuODU4NTQ0XSwgWzQ1LjgzNTIyMiwgMTAuODU4NjEyXSwgWzQ1LjgzNTE5MiwgMTAuODU4Njc3XSwgWzQ1LjgzNTE2OSwgMTAuODU4NzU1XSwgWzQ1LjgzNTE2NiwgMTAuODU4ODE0XSwgWzQ1LjgzNTE2NCwgMTAuODU4ODczXSwgWzQ1LjgzNTE1OSwgMTAuODU4OTMyXSwgWzQ1LjgzNTE0NSwgMTAuODU4OTk3XSwgWzQ1LjgzNTEyMiwgMTAuODU5MDYyXSwgWzQ1LjgzNTA5NCwgMTAuODU5MTI0XSwgWzQ1LjgzNTA0NiwgMTAuODU5MTk5XSwgWzQ1LjgzNDk5LCAxMC44NTkyOTNdLCBbNDUuODM0OTY5LCAxMC44NTkzNThdLCBbNDUuODM0OTU4LCAxMC44NTk0MV0sIFs0NS44MzQ5NDYsIDEwLjg1OTQ0OV0sIFs0NS44MzQ5MTQsIDEwLjg1OTQ5OF0sIFs0NS44MzQ4NjMsIDEwLjg1OTUzN10sIFs0NS44MzQ3OTcsIDEwLjg1OTU3Ml0sIFs0NS44MzQ3MTUsIDEwLjg1OTU4MV0sIFs0NS44MzQ2MjEsIDEwLjg1OTU3XSwgWzQ1LjgzNDUzMiwgMTAuODU5NTQzXSwgWzQ1LjgzNDQ3NywgMTAuODU5NTMyXSwgWzQ1LjgzNDM3LCAxMC44NTk1NjddLCBbNDUuODM0MzMxLCAxMC44NTk2MDldLCBbNDUuODM0Mjg5LCAxMC44NTk2NDhdLCBbNDUuODM0MjY0LCAxMC44NTk2OTddLCBbNDUuODM0MjUyLCAxMC44NTk3MzZdLCBbNDUuODM0MjI5LCAxMC44NTk4MTRdLCBbNDUuODM0MTg1LCAxMC44NTk4ODldLCBbNDUuODM0MTI1LCAxMC44NTk5NjddLCBbNDUuODM0MDYxLCAxMC44NjAwMzFdLCBbNDUuODM0MDI0LCAxMC44NjAwNjRdLCBbNDUuODMzOTgzLCAxMC44NjAwODldLCBbNDUuODMzOTE0LCAxMC44NjAxMDVdLCBbNDUuODMzODYyLCAxMC44NjAxMDRdLCBbNDUuODMzODAzLCAxMC44NjAwNzhdLCBbNDUuODMzNzUzLCAxMC44NjAwNDRdLCBbNDUuODMzNjg0LCAxMC44NjAwMDRdLCBbNDUuODMzNjE0LCAxMC44NTk5NjhdLCBbNDUuODMzNTM2LCAxMC44NTk5MzddLCBbNDUuODMzNDU0LCAxMC44NTk5MDRdLCBbNDUuODMzMjk5LCAxMC44NTk4M10sIFs0NS44MzMxNzYsIDEwLjg1OTc5OV0sIFs0NS44MzMwNDYsIDEwLjg1OTc2NV0sIFs0NS44MzI5MDcsIDEwLjg1OTcyMV0sIFs0NS44MzI4MDQsIDEwLjg1OTY4MV0sIFs0NS44MzI2NzksIDEwLjg1OTYyN10sIFs0NS44MzI1NDksIDEwLjg1OTU3M10sIFs0NS44MzI0NDQsIDEwLjg1OTU0Ml0sIFs0NS44MzIzMzQsIDEwLjg1OTQ5NV0sIFs0NS44MzIxODksIDEwLjg1OTQxOV0sIFs0NS44MzIwOTMsIDEwLjg1OTM2Ml0sIFs0NS44MzIwMDIsIDEwLjg1OTI4OV0sIFs0NS44MzE5MjksIDEwLjg1OTIyOV0sIFs0NS44MzE4MjksIDEwLjg1OTE0M10sIFs0NS44MzE3MjQsIDEwLjg1OTA2NF0sIFs0NS44MzE2NTYsIDEwLjg1OTAxN10sIFs0NS44MzE1OSwgMTAuODU4OTY3XSwgWzQ1LjgzMTQ5LCAxMC44NTg4ODRdLCBbNDUuODMxMzc0LCAxMC44NTg3ODhdLCBbNDUuODMxMjE5LCAxMC44NTg2NjldLCBbNDUuODMxMDgzLCAxMC44NTg1NzNdLCBbNDUuODMwOTQ2LCAxMC44NTg0ODldLCBbNDUuODMwODA5LCAxMC44NTg0MTldLCBbNDUuODMwNjIsIDEwLjg1ODMyOV0sIFs0NS44MzA0NTksIDEwLjg1ODIzMl0sIFs0NS44MzAzMDgsIDEwLjg1ODE0NV0sIFs0NS44MzAxNiwgMTAuODU4MDYyXSwgWzQ1LjgyOTk5NCwgMTAuODU3OTUyXSwgWzQ1LjgyOTg0NCwgMTAuODU3ODY5XSwgWzQ1LjgyOTY5OCwgMTAuODU3Nzg2XSwgWzQ1LjgyOTU2NiwgMTAuODU3NzAyXSwgWzQ1LjgyOTQxNiwgMTAuODU3NTkzXSwgWzQ1LjgyOTMwMiwgMTAuODU3NTI2XSwgWzQ1LjgyOTE0LCAxMC44NTc0NDldLCBbNDUuODI5MDA2LCAxMC44NTczODVdLCBbNDUuODI4NzIxLCAxMC44NTcxOTZdLCBbNDUuODI4NDc1LCAxMC44NTcwMV0sIFs0NS44MjgzNTIsIDEwLjg1NjkyN10sIFs0NS44MjgxOTgsIDEwLjg1NjgxMV0sIFs0NS44Mjc5ODEsIDEwLjg1NjY5NF0sIFs0NS44Mjc5MTMsIDEwLjg1NjY2MV0sIFs0NS44Mjc3OTIsIDEwLjg1NjYxM10sIFs0NS44Mjc3MzMsIDEwLjg1NjU5M10sIFs0NS44Mjc1NjEsIDEwLjg1NjU1NV0sIFs0NS44Mjc0NTIsIDEwLjg1NjU3XSwgWzQ1LjgyNzQwNiwgMTAuODU2NTkzXSwgWzQ1LjgyNzM1OCwgMTAuODU2NjQxXSwgWzQ1LjgyNzMsIDEwLjg1NjcwOV0sIFs0NS44MjcxOTQsIDEwLjg1NjgzNl0sIFs0NS44MjcwODIsIDEwLjg1Njk0Ml0sIFs0NS44MjcwMTQsIDEwLjg1NzAwOF0sIFs0NS44MzU0NzcsIDEwLjgzMjM0MV0sIFs0NS44Mzc2MywgMTAuODI1ODUzXV1dXSwgInR5cGUiOiAiTXVsdGlQb2x5Z29uIn0sICJpZCI6ICI3IiwgInByb3BlcnRpZXMiOiB7ImRlc2NyaXppb25lX29yaWdpbmUiOiAiTm9uIGNvZGlmaWNhdG8iLCAiZ21sX2lkIjogIklELkFDUVVFRklTSUNIRS5TUEVDQ0hJLkFDUVVBLjU0MTgzIiwgIm5hc3AiOiAwLCAibmF0dXJhX3NwZWNjaGlvX2FjcXVhIjogIk5vbiBjb2RpZmljYXRvIiwgIm5vbWUiOiAiTEFHTyBESSBHQVJEQSIsICJvcmlnaW5lIjogMH0sICJ0eXBlIjogIkZlYXR1cmUifSwgeyJnZW9tZXRyeSI6IHsiY29vcmRpbmF0ZXMiOiBbW1tbNDYuMDU5MTE0LCAxMC45Nzc0MTddLCBbNDYuMDU5MTA0LCAxMC45Nzc1ODhdLCBbNDYuMDU5MDkyLCAxMC45Nzc3NzldLCBbNDYuMDU5MDQ2LCAxMC45Nzc3NzhdLCBbNDYuMDU4OTk2LCAxMC45Nzc3ODNdLCBbNDYuMDU4OTUsIDEwLjk3Nzc5OF0sIFs0Ni4wNTg3OTQsIDEwLjk3NzgyNF0sIFs0Ni4wNTg3NDIsIDEwLjk3NzgwM10sIFs0Ni4wNTg3MDEsIDEwLjk3Nzc3MV0sIFs0Ni4wNTg2NDksIDEwLjk3NzczMl0sIFs0Ni4wNTg1MjEsIDEwLjk3NzU4OF0sIFs0Ni4wNTg0NTEsIDEwLjk3NzQ3N10sIFs0Ni4wNTgzOSwgMTAuOTc3NDA0XSwgWzQ2LjA1ODMxNSwgMTAuOTc3MzQ2XSwgWzQ2LjA1ODIwNCwgMTAuOTc3MjYxXSwgWzQ2LjA1ODExNiwgMTAuOTc3MThdLCBbNDYuMDU4MDAxLCAxMC45NzcwNTldLCBbNDYuMDU3OTIsIDEwLjk3NjkzNl0sIFs0Ni4wNTc4ODIsIDEwLjk3Njg2Ml0sIFs0Ni4wNTc4NjIsIDEwLjk3NjgzMl0sIFs0Ni4wNTc3OTcsIDEwLjk3Njc3OF0sIFs0Ni4wNTc3NTYsIDEwLjk3Njc0OF0sIFs0Ni4wNTc3MTMsIDEwLjk3NjcyN10sIFs0Ni4wNTc2NTMsIDEwLjk3NjcxOV0sIFs0Ni4wNTc1ODcsIDEwLjk3NjcwMV0sIFs0Ni4wNTc1MzMsIDEwLjk3NjY2XSwgWzQ2LjA1NzQ2MywgMTAuOTc2NTUzXSwgWzQ2LjA1NzQwNCwgMTAuOTc2NDAxXSwgWzQ2LjA1NzM1LCAxMC45NzYyNjhdLCBbNDYuMDU3MjcyLCAxMC45NzYxMzFdLCBbNDYuMDU3MTg5LCAxMC45NzYwMTRdLCBbNDYuMDU3MTE5LCAxMC45NzU5MDRdLCBbNDYuMDU3MDc3LCAxMC45NzU3OThdLCBbNDYuMDU3MDM5LCAxMC45NzU3MDJdLCBbNDYuMDU3MDI5LCAxMC45NzU1OV0sIFs0Ni4wNTcwNjcsIDEwLjk3NTQ1OV0sIFs0Ni4wNTcxMTQsIDEwLjk3NTM0Nl0sIFs0Ni4wNTcxMzcsIDEwLjk3NTE5OF0sIFs0Ni4wNTcxMDIsIDEwLjk3NTA3OV0sIFs0Ni4wNTcwMzcsIDEwLjk3NDk3Ml0sIFs0Ni4wNTY5OTYsIDEwLjk3NDkwMl0sIFs0Ni4wNTY4MDUsIDEwLjk3NDQ2NF0sIFs0Ni4wNTY3MzgsIDEwLjk3NDMyNF0sIFs0Ni4wNTY2OTEsIDEwLjk3NDIyMV0sIFs0Ni4wNTY2MzMsIDEwLjk3NDEwMl0sIFs0Ni4wNTY1NjksIDEwLjk3MzkwM10sIFs0Ni4wNTY1MTQsIDEwLjk3Mzc3NF0sIFs0Ni4wNTY0MTQsIDEwLjk3MzU1NF0sIFs0Ni4wNTYyOTgsIDEwLjk3MzMwNV0sIFs0Ni4wNTYxNzEsIDEwLjk3MzAzNl0sIFs0Ni4wNTYwMTcsIDEwLjk3MjcxN10sIFs0Ni4wNTU4ODUsIDEwLjk3MjQ1MV0sIFs0Ni4wNTU3MTcsIDEwLjk3MjE3NF0sIFs0Ni4wNTU1OTYsIDEwLjk3MTk3MV0sIFs0Ni4wNTU0MzQsIDEwLjk3MTcyNF0sIFs0Ni4wNTUyOTUsIDEwLjk3MTUxN10sIFs0Ni4wNTUxNzEsIDEwLjk3MTM3OV0sIFs0Ni4wNTUwNTgsIDEwLjk3MTI4NF0sIFs0Ni4wNTQ4NzIsIDEwLjk3MTE1Ml0sIFs0Ni4wNTQ3NDgsIDEwLjk3MTA0NF0sIFs0Ni4wNTQ2NTYsIDEwLjk3MDkxNl0sIFs0Ni4wNTQ1ODIsIDEwLjk3MDc1NF0sIFs0Ni4wNTQ1MzYsIDEwLjk3MDYwMV0sIFs0Ni4wNTQ0NzQsIDEwLjk3MDQ0Nl0sIFs0Ni4wNTQzOTEsIDEwLjk3MDMxNV0sIFs0Ni4wNTQzMDgsIDEwLjk3MDIwNV0sIFs0Ni4wNTQyMTgsIDEwLjk3MDA4MV0sIFs0Ni4wNTQxNDQsIDEwLjk2OTk1OF0sIFs0Ni4wNTQwNzYsIDEwLjk2OTg0OF0sIFs0Ni4wNTM5ODQsIDEwLjk2OTc0NF0sIFs0Ni4wNTM4OTgsIDEwLjk2OTY3M10sIFs0Ni4wNTM4MjYsIDEwLjk2OTYwNV0sIFs0Ni4wNTM3NjUsIDEwLjk2OTU2MV0sIFs0Ni4wNTM3MjEsIDEwLjk2OTU2M10sIFs0Ni4wNTM2ODQsIDEwLjk2OTU3Nl0sIFs0Ni4wNTM2NDksIDEwLjk2OTYzNF0sIFs0Ni4wNTM2MDcsIDEwLjk2OTc0OF0sIFs0Ni4wNTM1NzgsIDEwLjk2OTg2OV0sIFs0Ni4wNTM1MzEsIDEwLjk2OTk5Nl0sIFs0Ni4wNTM0NzYsIDEwLjk3MDE1Nl0sIFs0Ni4wNTM0MzYsIDEwLjk3MDI1M10sIFs0Ni4wNTMzOTIsIDEwLjk3MDMzOF0sIFs0Ni4wNTMzNTMsIDEwLjk3MDM3M10sIFs0Ni4wNTMzMjEsIDEwLjk3MDM4Ml0sIFs0Ni4wNTMyNjQsIDEwLjk3MDM3NF0sIFs0Ni4wNTMyMTYsIDEwLjk3MDM1M10sIFs0Ni4wNTMxNTcsIDEwLjk3MDMxOV0sIFs0Ni4wNTMxLCAxMC45NzAyNzVdLCBbNDYuMDUzMDI4LCAxMC45NzAyMTRdLCBbNDYuMDUyOTY1LCAxMC45NzAxNDddLCBbNDYuMDUyODc5LCAxMC45NzAwNDZdLCBbNDYuMDUyNzk2LCAxMC45Njk5NDldLCBbNDYuMDUyNzUsIDEwLjk2OTkwNV0sIFs0Ni4wNTI3MTksIDEwLjk2OTg4NV0sIFs0Ni4wNTI2OCwgMTAuOTY5ODc0XSwgWzQ2LjA1MjY0MSwgMTAuOTY5ODczXSwgWzQ2LjA1MjYyLCAxMC45Njk4OTldLCBbNDYuMDUyNjEsIDEwLjk2OTk1NF0sIFs0Ni4wNTI2MjMsIDEwLjk3MDA1Nl0sIFs0Ni4wNTI2MzUsIDEwLjk3MDE2OF0sIFs0Ni4wNTI2NDUsIDEwLjk3MDM0OV0sIFs0Ni4wNTI2NDcsIDEwLjk3MDUxN10sIFs0Ni4wNTI2NTYsIDEwLjk3MDcwNV0sIFs0Ni4wNTI2NzksIDEwLjk3MDg5Nl0sIFs0Ni4wNTI3MjcsIDEwLjk3MTEyN10sIFs0Ni4wNTI3NzcsIDEwLjk3MTM0NV0sIFs0Ni4wNTI4NDcsIDEwLjk3MTU3XSwgWzQ2LjA1MjkyNiwgMTAuOTcxODIyXSwgWzQ2LjA1MzAzLCAxMC45NzIxMV0sIFs0Ni4wNTMxNDYsIDEwLjk3MjM4Ml0sIFs0Ni4wNTMzNDYsIDEwLjk3Mjg3XSwgWzQ2LjA1MzQ0NCwgMTAuOTczMDY5XSwgWzQ2LjA1MzUzNSwgMTAuOTczMjc1XSwgWzQ2LjA1MzYxMSwgMTAuOTczNDc4XSwgWzQ2LjA1MzY1NCwgMTAuOTczNjQ5XSwgWzQ2LjA1MzY4LCAxMC45NzM4MThdLCBbNDYuMDUzNjk2LCAxMC45NzQwMDJdLCBbNDYuMDUzNzI4LCAxMC45NzQyMTZdLCBbNDYuMDUzNzUzLCAxMC45NzQzOThdLCBbNDYuMDUzNzcxLCAxMC45NzQ2MjJdLCBbNDYuMDUzNzY1LCAxMC45NzQ3MzZdLCBbNDYuMDUzNzUyLCAxMC45NzQ4MTVdLCBbNDYuMDUzNzM0LCAxMC45NzQ4NjRdLCBbNDYuMDUzNjczLCAxMC45NzQ5NTFdLCBbNDYuMDUzNjA5LCAxMC45NzQ5ODJdLCBbNDYuMDUzNTQ5LCAxMC45NzQ5ODFdLCBbNDYuMDUzNSwgMTAuOTc0OTM3XSwgWzQ2LjA1MzQzLCAxMC45NzQ4M10sIFs0Ni4wNTMzMjksIDEwLjk3NDYzN10sIFs0Ni4wNTMyMTUsIDEwLjk3NDQzNF0sIFs0Ni4wNTMwODMsIDEwLjk3NDE5NF0sIFs0Ni4wNTI5NTEsIDEwLjk3Mzk5MV0sIFs0Ni4wNTI4MjgsIDEwLjk3Mzc5N10sIFs0Ni4wNTI2OTUsIDEwLjk3MzU3N10sIFs0Ni4wNTI1NTcsIDEwLjk3MzMwOF0sIFs0Ni4wNTI0NzUsIDEwLjk3MzA2OV0sIFs0Ni4wNTIzNjcsIDEwLjk3Mjc4N10sIFs0Ni4wNTIyNzQsIDEwLjk3MjQ5M10sIFs0Ni4wNTIxNzUsIDEwLjk3MjIyMV0sIFs0Ni4wNTIwNjUsIDEwLjk3MTk4OF0sIFs0Ni4wNTE5MTMsIDEwLjk3MTczNV0sIFs0Ni4wNTE3NjUsIDEwLjk3MTQ1OV0sIFs0Ni4wNTE2MjIsIDEwLjk3MTE5Nl0sIFs0Ni4wNTE1MTEsIDEwLjk3MDk3Nl0sIFs0Ni4wNTE0MTMsIDEwLjk3MDc1XSwgWzQ2LjA1MTMwNSwgMTAuOTcwNTU0XSwgWzQ2LjA1MTE5NSwgMTAuOTcwMzk0XSwgWzQ2LjA1MTEyMiwgMTAuOTcwMjQxXSwgWzQ2LjA1MTA5NCwgMTAuOTcwMDk2XSwgWzQ2LjA1MTA4OSwgMTAuOTY5OTQxXSwgWzQ2LjA1MTA4NiwgMTAuOTY5Nzk2XSwgWzQ2LjA1MTA3OCwgMTAuOTY5Njk0XSwgWzQ2LjA1MTA1LCAxMC45Njk1NjJdLCBbNDYuMDUwOTk4LCAxMC45Njk0NTldLCBbNDYuMDUwOTE3LCAxMC45NjkzNTVdLCBbNDYuMDUwODEyLCAxMC45NjkxOTVdLCBbNDYuMDUwNjk1LCAxMC45NjkwMDhdLCBbNDYuMDUwNjAxLCAxMC45Njg4MzldLCBbNDYuMDUwNTAzLCAxMC45Njg2NjVdLCBbNDYuMDUwNDE4LCAxMC45Njg1MTldLCBbNDYuMDUwMzM3LCAxMC45Njg0MTVdLCBbNDYuMDUwMjQyLCAxMC45NjgyOTFdLCBbNDYuMDUwMTY4LCAxMC45NjgxODRdLCBbNDYuMDUwMDc0LCAxMC45NjgwMjRdLCBbNDYuMDUwMDAzLCAxMC45Njc4NjhdLCBbNDYuMDQ5OTM0LCAxMC45Njc3MTVdLCBbNDYuMDQ5ODc2LCAxMC45Njc2MDJdLCBbNDYuMDQ5ODAyLCAxMC45Njc0NzldLCBbNDYuMDQ5NzIxLCAxMC45NjczNjJdLCBbNDYuMDQ5NjU2LCAxMC45NjcyNTVdLCBbNDYuMDQ5NjAyLCAxMC45NjcxNzJdLCBbNDYuMDQ5NTQxLCAxMC45NjcxMDJdLCBbNDYuMDQ5NDUsIDEwLjk2NzA0N10sIFs0Ni4wNDkzNzgsIDEwLjk2Njk3OV0sIFs0Ni4wNDkzMTUsIDEwLjk2Njg4OV0sIFs0Ni4wNDkyMjgsIDEwLjk2NjcyNl0sIFs0Ni4wNDkxODcsIDEwLjk2NjY1M10sIFs0Ni4wNDkxNDcsIDEwLjk2NjYwM10sIFs0Ni4wNDkxMDksIDEwLjk2NjU0OV0sIFs0Ni4wNDkwMTcsIDEwLjk2NjM5OV0sIFs0Ni4wNDg5NzEsIDEwLjk2NjM0OV0sIFs0Ni4wNDg5MjEsIDEwLjk2NjI4Nl0sIFs0Ni4wNDg4MSwgMTAuOTY2MTM2XSwgWzQ2LjA0ODczOCwgMTAuOTY2MDI3XSwgWzQ2LjA0ODY3OSwgMTAuOTY1OTM3XSwgWzQ2LjA0ODYyNSwgMTAuOTY1ODc3XSwgWzQ2LjA0ODU3LCAxMC45NjU4NV0sIFs0Ni4wNDg0OSwgMTAuOTY1ODQ5XSwgWzQ2LjA0ODQyMiwgMTAuOTY1ODM4XSwgWzQ2LjA0ODMzMSwgMTAuOTY1Nzk3XSwgWzQ2LjA0ODI2NywgMTAuOTY1NzRdLCBbNDYuMDQ4MTk3LCAxMC45NjU2MzNdLCBbNDYuMDQ4MTU5LCAxMC45NjU1NDddLCBbNDYuMDQ4MTA1LCAxMC45NjU0MTVdLCBbNDYuMDQ4MDY3LCAxMC45NjUzMDldLCBbNDYuMDQ4MDM4LCAxMC45NjUxOV0sIFs0Ni4wNDgwMTYsIDEwLjk2NTA3NV0sIFs0Ni4wNDc5OSwgMTAuOTY0OTE3XSwgWzQ2LjA0Nzk0NiwgMTAuOTY0NzMyXSwgWzQ2LjA0NzkwOCwgMTAuOTY0NjI2XSwgWzQ2LjA0Nzg1NCwgMTAuOTY0NTFdLCBbNDYuMDQ3ODA0LCAxMC45NjQ0MTddLCBbNDYuMDQ3Njk4LCAxMC45NjQyNjFdLCBbNDYuMDQ3NjI1LCAxMC45NjQxODVdLCBbNDYuMDQ3NTUsIDEwLjk2NDExNF0sIFs0Ni4wNDc0NzMsIDEwLjk2NDA1MV0sIFs0Ni4wNDczODcsIDEwLjk2Mzk4M10sIFs0Ni4wNDcyODksIDEwLjk2MzkwM10sIFs0Ni4wNDcxNzgsIDEwLjk2Mzc5M10sIFs0Ni4wNDcwODcsIDEwLjk2MzcwNl0sIFs0Ni4wNDcwMSwgMTAuOTYzNjM2XSwgWzQ2LjA0Njg0MiwgMTAuOTYzNDkyXSwgWzQ2LjA0NjczNSwgMTAuOTYzNDE4XSwgWzQ2LjA0NjYwNywgMTAuOTYzMzE3XSwgWzQ2LjA0NjQ2NCwgMTAuOTYzMThdLCBbNDYuMDQ2MzM1LCAxMC45NjMwMzNdLCBbNDYuMDQ2MTg3LCAxMC45NjI5NDJdLCBbNDYuMDQ2MDc4LCAxMC45NjI5Ml0sIFs0Ni4wNDU5OTYsIDEwLjk2Mjg5OV0sIFs0Ni4wNDU5MzQsIDEwLjk2Mjg4OV0sIFs0Ni4wNDU4MzMsIDEwLjk2MjkwM10sIFs0Ni4wNDU3MDEsIDEwLjk2MjkwOF0sIFs0Ni4wNDU3MDcsIDEwLjk2Mjg2MV0sIFs0Ni4wNDU3MTMsIDEwLjk2MjgxNl0sIFs0Ni4wNDU3NjEsIDEwLjk2Mjc4NF0sIFs0Ni4wNDU4MjUsIDEwLjk2MjcyOV0sIFs0Ni4wNDU4NzgsIDEwLjk2MjY4NF0sIFs0Ni4wNDU5MjcsIDEwLjk2MjYyOV0sIFs0Ni4wNDU5OTEsIDEwLjk2MjU3MV0sIFs0Ni4wNDYwMjgsIDEwLjk2MjU1Ml0sIFs0Ni4wNDYxNTMsIDEwLjk2MjU5XSwgWzQ2LjA0NjIxOSwgMTAuOTYyNjMzXSwgWzQ2LjA0NjI4NSwgMTAuOTYyNjY3XSwgWzQ2LjA0NjM3MiwgMTAuOTYyNzAyXSwgWzQ2LjA0NjQ5MSwgMTAuOTYyNzYzXSwgWzQ2LjA0NjYwNywgMTAuOTYyODNdLCBbNDYuMDQ2NzI1LCAxMC45NjI4OTVdLCBbNDYuMDQ2ODUzLCAxMC45NjI5NjZdLCBbNDYuMDQ3MDEyLCAxMC45NjMwMzRdLCBbNDYuMDQ3MTUzLCAxMC45NjMxMDJdLCBbNDYuMDQ3MjUxLCAxMC45NjMxNjNdLCBbNDYuMDQ3Mzc0LCAxMC45NjMyMzRdLCBbNDYuMDQ3NTA2LCAxMC45NjMyNTZdLCBbNDYuMDQ3NjE5LCAxMC45NjMyNTRdLCBbNDYuMDQ3NzUxLCAxMC45NjMyNTNdLCBbNDYuMDQ3OTExLCAxMC45NjMyNDldLCBbNDYuMDQ4MDQxLCAxMC45NjMyNThdLCBbNDYuMDQ4MTgxLCAxMC45NjMyNTNdLCBbNDYuMDQ4MzIzLCAxMC45NjMyMjldLCBbNDYuMDQ4NDE5LCAxMC45NjMyMTRdLCBbNDYuMDQ4NTQ4LCAxMC45NjMxNjRdLCBbNDYuMDQ4NjM3LCAxMC45NjMxMTZdLCBbNDYuMDQ4NzAyLCAxMC45NjMwNThdLCBbNDYuMDQ4NzczLCAxMC45NjI5NjddLCBbNDYuMDQ4ODQ3LCAxMC45NjI4NjNdLCBbNDYuMDQ4OTA3LCAxMC45NjI3NTJdLCBbNDYuMDQ4OTU2LCAxMC45NjI2NDFdLCBbNDYuMDQ5MDAxLCAxMC45NjI1MjddLCBbNDYuMDQ5MDM4LCAxMC45NjI0MDldLCBbNDYuMDQ5MDY0LCAxMC45NjIzMTFdLCBbNDYuMDQ5MDk0LCAxMC45NjIyMjNdLCBbNDYuMDQ5MTA5LCAxMC45NjIxMzJdLCBbNDYuMDQ5MTIyLCAxMC45NjIxMTRdLCBbNDYuMDQ5MTQ4LCAxMC45NjIwNzhdLCBbNDYuMDQ5MTY5LCAxMC45NjIwNDNdLCBbNDYuMDQ5MjMsIDEwLjk2MTkyM10sIFs0Ni4wNDkyNjIsIDEwLjk2MTg2MV0sIFs0Ni4wNDkyOTUsIDEwLjk2MTgxNl0sIFs0Ni4wNDkzNDYsIDEwLjk2MTc2OF0sIFs0Ni4wNDkzOTQsIDEwLjk2MTczM10sIFs0Ni4wNDk0NTksIDEwLjk2MTY4NV0sIFs0Ni4wNDk1MTYsIDEwLjk2MTYzN10sIFs0Ni4wNDk1NzYsIDEwLjk2MTU4OV0sIFs0Ni4wNDk2MzcsIDEwLjk2MTUyMV0sIFs0Ni4wNDk2NjgsIDEwLjk2MTQwNF0sIFs0Ni4wNDk2ODcsIDEwLjk2MTI3Nl0sIFs0Ni4wNDk3MTIsIDEwLjk2MTE0Ml0sIFs0Ni4wNDk3NTIsIDEwLjk2MTAyMV0sIFs0Ni4wNDk4MTMsIDEwLjk2MDg3OF0sIFs0Ni4wNDk4NiwgMTAuOTYwNzY0XSwgWzQ2LjA0OTkzLCAxMC45NjA2MzFdLCBbNDYuMDUwMDA3LCAxMC45NjA1MjVdLCBbNDYuMDUwMTA0LCAxMC45NjA0MzJdLCBbNDYuMDUwMjQsIDEwLjk2MDMzM10sIFs0Ni4wNTAzNTMsIDEwLjk2MDI2XSwgWzQ2LjA1MDQ1NiwgMTAuOTYwMjI3XSwgWzQ2LjA1MDU4NCwgMTAuOTYwMl0sIFs0Ni4wNTA2NDIsIDEwLjk2MDE5NV0sIFs0Ni4wNTA3ODEsIDEwLjk2MDIxNV0sIFs0Ni4wNTA4NDUsIDEwLjk2MDI0Ml0sIFs0Ni4wNTA5MTksIDEwLjk2MDMyM10sIFs0Ni4wNTA5NzEsIDEwLjk2MDQwM10sIFs0Ni4wNTEwMjYsIDEwLjk2MDUzOV0sIFs0Ni4wNTEwNjMsIDEwLjk2MDY3MV0sIFs0Ni4wNTExMjgsIDEwLjk2MDg1XSwgWzQ2LjA1MTE5MiwgMTAuOTYwOTddLCBbNDYuMDUxMjU1LCAxMC45NjEwOV0sIFs0Ni4wNTEyODMsIDEwLjk2MTE5OV0sIFs0Ni4wNTEzMjMsIDEwLjk2MTMwNV0sIFs0Ni4wNTEzNzcsIDEwLjk2MTM4OF0sIFs0Ni4wNTE0NTQsIDEwLjk2MTQ4Nl0sIFs0Ni4wNTE1MjQsIDEwLjk2MTU1M10sIFs0Ni4wNTE1ODMsIDEwLjk2MTU5XSwgWzQ2LjA1MTYzNywgMTAuOTYxNjIxXSwgWzQ2LjA1MTY2NiwgMTAuOTYxNjY1XSwgWzQ2LjA1MTcxNiwgMTAuOTYxNzE5XSwgWzQ2LjA1MTc1LCAxMC45NjE3MzldLCBbNDYuMDUxODYsIDEwLjk2MTc1MV0sIFs0Ni4wNTE4OTgsIDEwLjk2MTc5NV0sIFs0Ni4wNTE5NSwgMTAuOTYxODUyXSwgWzQ2LjA1MjAwNCwgMTAuOTYxODc2XSwgWzQ2LjA1MjA0OCwgMTAuOTYxODY0XSwgWzQ2LjA1MjExOSwgMTAuOTYxODEzXSwgWzQ2LjA1MjE1MiwgMTAuOTYxNzUyXSwgWzQ2LjA1MjE2NywgMTAuOTYxNjc3XSwgWzQ2LjA1MjE2NSwgMTAuOTYxNjI3XSwgWzQ2LjA1MjEwMSwgMTAuOTYxNDQ1XSwgWzQ2LjA1MjAyOSwgMTAuOTYxMzE1XSwgWzQ2LjA1MTk0OCwgMTAuOTYxMjA4XSwgWzQ2LjA1MTg2LCAxMC45NjExMDRdLCBbNDYuMDUxNzY4LCAxMC45NjEwMTNdLCBbNDYuMDUxNjkxLCAxMC45NjA5MjNdLCBbNDYuMDUxNjEsIDEwLjk2MDc5Nl0sIFs0Ni4wNTE1MjUsIDEwLjk2MDY1M10sIFs0Ni4wNTE0NDcsIDEwLjk2MDQ5XSwgWzQ2LjA1MTM5NywgMTAuOTYwMzI0XSwgWzQ2LjA1MTM2OCwgMTAuOTYwMTk5XSwgWzQ2LjA1MTM0MiwgMTAuOTYwMDddLCBbNDYuMDUxMzM0LCAxMC45NTk5ODRdLCBbNDYuMDUxMzE5LCAxMC45NTk5MjVdLCBbNDYuMDUxMjk3LCAxMC45NTk4OTJdLCBbNDYuMDUxMjUzLCAxMC45NTk4NzddLCBbNDYuMDUxMjI4LCAxMC45NTk4OTNdLCBbNDYuMDUxMjAzLCAxMC45NTk5MDldLCBbNDYuMDUxMTQ1LCAxMC45NTk5MTFdLCBbNDYuMDUxMTE2LCAxMC45NTk5MTddLCBbNDYuMDUxMDY1LCAxMC45NTk5MTZdLCBbNDYuMDUxMDQ4LCAxMC45NTk4ODZdLCBbNDYuMDUxMDE5LCAxMC45NTk4MTZdLCBbNDYuMDUxMDQ0LCAxMC45NTk3NjFdLCBbNDYuMDUxMDgxLCAxMC45NTk3MjVdLCBbNDYuMDUxMTUzLCAxMC45NTk2NjhdLCBbNDYuMDUxMjU2LCAxMC45NTk2MjFdLCBbNDYuMDUxMzk2LCAxMC45NTk1NjJdLCBbNDYuMDUxNTE0LCAxMC45NTk1MDZdLCBbNDYuMDUxNjA4LCAxMC45NTk0NDldLCBbNDYuMDUxNjc5LCAxMC45NTk0MTRdLCBbNDYuMDUxNzE2LCAxMC45NTkzNTldLCBbNDYuMDUxNzE3LCAxMC45NTkzMV0sIFs0Ni4wNTE2OTUsIDEwLjk1OTI0N10sIFs0Ni4wNTE2OTgsIDEwLjk1OTE4NV0sIFs0Ni4wNTE3MTksIDEwLjk1OTE0Ml0sIFs0Ni4wNTE3NzQsIDEwLjk1OTA4OF0sIFs0Ni4wNTE4MTYsIDEwLjk1OTA0Nl0sIFs0Ni4wNTE4NjQsIDEwLjk1OTAxMV0sIFs0Ni4wNTE5MjQsIDEwLjk1OTAyMl0sIFs0Ni4wNTE5NjksIDEwLjk1OTA1XSwgWzQ2LjA1MjAxOSwgMTAuOTU5MDc0XSwgWzQ2LjA1MjA4NywgMTAuOTU5MDkyXSwgWzQ2LjA1MjE2OCwgMTAuOTU5MDg0XSwgWzQ2LjA1MjI0NiwgMTAuOTU5MDZdLCBbNDYuMDUyMzMzLCAxMC45NTkwMzVdLCBbNDYuMDUyNDQ5LCAxMC45NTkwNDhdLCBbNDYuMDUyNTg0LCAxMC45NTkwNzFdLCBbNDYuMDUyNzMyLCAxMC45NTkxMTRdLCBbNDYuMDUyODk4LCAxMC45NTkxODNdLCBbNDYuMDUzMDg5LCAxMC45NTkyNDFdLCBbNDYuMDUzMjI4LCAxMC45NTkyNDRdLCBbNDYuMDUzMzY1LCAxMC45NTkyNTddLCBbNDYuMDUzNTIzLCAxMC45NTkyOV0sIFs0Ni4wNTM2ODIsIDEwLjk1OTM1Nl0sIFs0Ni4wNTM3OTEsIDEwLjk1OTQyNV0sIFs0Ni4wNTM4ODgsIDEwLjk1OTUzMl0sIFs0Ni4wNTM5NzMsIDEwLjk1OTY4Ml0sIFs0Ni4wNTQwMjgsIDEwLjk1OTc5OF0sIFs0Ni4wNTQwODksIDEwLjk1OTkyNV0sIFs0Ni4wNTQxNzYsIDEwLjk2MDA5NF0sIFs0Ni4wNTQyNiwgMTAuOTYwMzFdLCBbNDYuMDU0Mzc0LCAxMC45NjA1NTldLCBbNDYuMDU0NDkyLCAxMC45NjA4MjFdLCBbNDYuMDU0NjIzLCAxMC45NjExMTRdLCBbNDYuMDU0NzI3LCAxMC45NjEzNzldLCBbNDYuMDU0ODE1LCAxMC45NjE2NjddLCBbNDYuMDU0OTA2LCAxMC45NjE5MjVdLCBbNDYuMDU1MDA4LCAxMC45NjIxNzRdLCBbNDYuMDU1MTAxLCAxMC45NjI0XSwgWzQ2LjA1NTE4OCwgMTAuOTYyNTg5XSwgWzQ2LjA1NTI1NywgMTAuOTYyNzU5XSwgWzQ2LjA1NTMyOCwgMTAuOTYyOTQ3XSwgWzQ2LjA1NTQwOCwgMTAuOTYzMTQ3XSwgWzQ2LjA1NTUyNywgMTAuOTYzMzddLCBbNDYuMDU1NjU0LCAxMC45NjM2MTldLCBbNDYuMDU1ODEzLCAxMC45NjM5MTJdLCBbNDYuMDU1OTM2LCAxMC45NjQxMjJdLCBbNDYuMDU2MDM5LCAxMC45NjQyNzJdLCBbNDYuMDU2MTIyLCAxMC45NjQzODZdLCBbNDYuMDU2MTY0LCAxMC45NjQ1MDVdLCBbNDYuMDU2MTg1LCAxMC45NjQ2NTRdLCBbNDYuMDU2MTYyLCAxMC45NjQ3NDJdLCBbNDYuMDU2MTIsIDEwLjk2NDhdLCBbNDYuMDU2MTA3LCAxMC45NjQ4MTFdLCBbNDYuMDU2MDU3LCAxMC45NjQ4NThdLCBbNDYuMDU1OTkyLCAxMC45NjQ5NjFdLCBbNDYuMDU1OTM2LCAxMC45NjUwNDJdLCBbNDYuMDU1ODU1LCAxMC45NjUxMjZdLCBbNDYuMDU1Nzg2LCAxMC45NjUxODZdLCBbNDYuMDU1NzIyLCAxMC45NjUyMzddLCBbNDYuMDU1NjgsIDEwLjk2NTI4Nl0sIFs0Ni4wNTU2NCwgMTAuOTY1MzhdLCBbNDYuMDU1NjI1LCAxMC45NjU0OThdLCBbNDYuMDU1NjMzLCAxMC45NjU2MDddLCBbNDYuMDU1NjU2LCAxMC45NjU3NDVdLCBbNDYuMDU1Njk2LCAxMC45NjU4NzRdLCBbNDYuMDU1NzI3LCAxMC45NjU5NTddLCBbNDYuMDU1NzgsIDEwLjk2NjA4M10sIFs0Ni4wNTU4MDksIDEwLjk2NjE5Ml0sIFs0Ni4wNTU3ODYsIDEwLjk2NjMxN10sIFs0Ni4wNTU3NDYsIDEwLjk2NjQyNF0sIFs0Ni4wNTU3MDYsIDEwLjk2NjUyNV0sIFs0Ni4wNTU2ODMsIDEwLjk2NjU4NF0sIFs0Ni4wNTU2NzksIDEwLjk2NjY3Nl0sIFs0Ni4wNTU3MDEsIDEwLjk2Njc3OF0sIFs0Ni4wNTU3MTgsIDEwLjk2Njg4XSwgWzQ2LjA1NTcyNCwgMTAuOTY2OTgyXSwgWzQ2LjA1NTcyLCAxMC45NjcwODddLCBbNDYuMDU1NzE1LCAxMC45NjcxNF0sIFs0Ni4wNTU3MDQsIDEwLjk2NzI1OF0sIFs0Ni4wNTU3MDgsIDEwLjk2NzM2Nl0sIFs0Ni4wNTU2NDUsIDEwLjk2NzQyNF0sIFs0Ni4wNTU2MiwgMTAuOTY3MjkyXSwgWzQ2LjA1NTYxOCwgMTAuOTY3MjIzXSwgWzQ2LjA1NTYxOSwgMTAuOTY3MTYxXSwgWzQ2LjA1NTYyMiwgMTAuOTY3MDk1XSwgWzQ2LjA1NTYyNSwgMTAuOTY3MDQyXSwgWzQ2LjA1NTYyOCwgMTAuOTY2OThdLCBbNDYuMDU1NjI4LCAxMC45NjY5MjddLCBbNDYuMDU1NjIsIDEwLjk2Njg3NV0sIFs0Ni4wNTU2MDUsIDEwLjk2Njc5OV0sIFs0Ni4wNTU1OSwgMTAuOTY2NzI2XSwgWzQ2LjA1NTU3NywgMTAuOTY2NjddLCBbNDYuMDU1NTYxLCAxMC45NjY2MjddLCBbNDYuMDU1NTMyLCAxMC45NjY1NjddLCBbNDYuMDU1NTA1LCAxMC45NjY1MV0sIFs0Ni4wNTU0ODEsIDEwLjk2NjQ3N10sIFs0Ni4wNTU0MzEsIDEwLjk2NjQ0Nl0sIFs0Ni4wNTUzOSwgMTAuOTY2NDM5XSwgWzQ2LjA1NTM1MSwgMTAuOTY2NDQ0XSwgWzQ2LjA1NTI4NCwgMTAuOTY2NDY2XSwgWzQ2LjA1NTIxOCwgMTAuOTY2NDg3XSwgWzQ2LjA1NTE0OSwgMTAuOTY2NTMyXSwgWzQ2LjA1NTEwNSwgMTAuOTY2NTc3XSwgWzQ2LjA1NTA0LCAxMC45NjY2MTFdLCBbNDYuMDU0OTkyLCAxMC45NjY2MTNdLCBbNDYuMDU0OTIyLCAxMC45NjY1OThdLCBbNDYuMDU0ODUzLCAxMC45NjY1OTRdLCBbNDYuMDU0NzY0LCAxMC45NjY1ODVdLCBbNDYuMDU0NywgMTAuOTY2NTk3XSwgWzQ2LjA1NDYxNywgMTAuOTY2NjQ0XSwgWzQ2LjA1NDUyNSwgMTAuOTY2Njg0XSwgWzQ2LjA1NDQ0NSwgMTAuOTY2Njk2XSwgWzQ2LjA1NDQwNCwgMTAuOTY2NzExXSwgWzQ2LjA1NDM3MywgMTAuOTY2ODAyXSwgWzQ2LjA1NDM1OCwgMTAuOTY2OTA3XSwgWzQ2LjA1NDM1NywgMTAuOTY2OTg5XSwgWzQ2LjA1NDM2OCwgMTAuOTY3MDc1XSwgWzQ2LjA1NDM5MSwgMTAuOTY3MjMzXSwgWzQ2LjA1NDQxMiwgMTAuOTY3MzQ5XSwgWzQ2LjA1NDQ0MywgMTAuOTY3NDU1XSwgWzQ2LjA1NDQ5MiwgMTAuOTY3NTc3XSwgWzQ2LjA1NDU1MiwgMTAuOTY3NjgxXSwgWzQ2LjA1NDYzMiwgMTAuOTY3NzQ4XSwgWzQ2LjA1NDcyLCAxMC45Njc3OTNdLCBbNDYuMDU0Nzc5LCAxMC45Njc4NV0sIFs0Ni4wNTQ4MjYsIDEwLjk2NzkxN10sIFs0Ni4wNTQ4NjcsIDEwLjk2Nzk2N10sIFs0Ni4wNTQ5MSwgMTAuOTY3OTg1XSwgWzQ2LjA1NDk5MiwgMTAuOTY3OTk3XSwgWzQ2LjA1NTA4MywgMTAuOTY3OTgyXSwgWzQ2LjA1NTIwNywgMTAuOTY3OTQ5XSwgWzQ2LjA1NTMyMiwgMTAuOTY3OTIyXSwgWzQ2LjA1NTQyLCAxMC45Njc4OTJdLCBbNDYuMDU1NTE0LCAxMC45Njc4NjhdLCBbNDYuMDU1NTkzLCAxMC45Njc4MzddLCBbNDYuMDU1NjY2LCAxMC45Njc4MDJdLCBbNDYuMDU1NzA1LCAxMC45Njc3N10sIFs0Ni4wNTU3MiwgMTAuOTY3NzAyXSwgWzQ2LjA1NTcxMSwgMTAuOTY3NjMzXSwgWzQ2LjA1NTcxNywgMTAuOTY3NTUxXSwgWzQ2LjA1NTc3NywgMTAuOTY3NTE5XSwgWzQ2LjA1NTc5NywgMTAuOTY3NTQ5XSwgWzQ2LjA1NTgyMiwgMTAuOTY3NTg5XSwgWzQ2LjA1NTg2LCAxMC45Njc2MTZdLCBbNDYuMDU1OTA4LCAxMC45Njc1OThdLCBbNDYuMDU1OTM0LCAxMC45Njc1NjZdLCBbNDYuMDU1OTY4LCAxMC45Njc1NDNdLCBbNDYuMDU2MDA1LCAxMC45Njc1MjhdLCBbNDYuMDU2MDQ0LCAxMC45Njc1NDldLCBbNDYuMDU2MDk0LCAxMC45Njc1NzldLCBbNDYuMDU2MTM0LCAxMC45Njc2MjNdLCBbNDYuMDU2MjI5LCAxMC45Njc3N10sIFs0Ni4wNTYyOTgsIDEwLjk2Nzg4XSwgWzQ2LjA1NjM2MywgMTAuOTY3OTldLCBbNDYuMDU2NDIzLCAxMC45NjgxXSwgWzQ2LjA1NjQ5MiwgMTAuOTY4MjU5XSwgWzQ2LjA1NjU2OCwgMTAuOTY4NDcxXSwgWzQ2LjA1NjY0NCwgMTAuOTY4NzM5XSwgWzQ2LjA1NjczMiwgMTAuOTY5MDQ0XSwgWzQ2LjA1NjgxOCwgMTAuOTY5MzE1XSwgWzQ2LjA1Njg5MSwgMTAuOTY5NTE0XSwgWzQ2LjA1Njk2NywgMTAuOTY5NzEzXSwgWzQ2LjA1NzA0NSwgMTAuOTY5ODY5XSwgWzQ2LjA1NzEzLCAxMC45NzAwMTZdLCBbNDYuMDU3MjM3LCAxMC45NzAxODNdLCBbNDYuMDU3Mzc1LCAxMC45NzAzNjNdLCBbNDYuMDU3NDk5LCAxMC45NzA1MDRdLCBbNDYuMDU3NjIxLCAxMC45NzA2MTZdLCBbNDYuMDU3NzA5LCAxMC45NzA2OF0sIFs0Ni4wNTc4MDYsIDEwLjk3MDc2Ml0sIFs0Ni4wNTc5MDQsIDEwLjk3MDg1Nl0sIFs0Ni4wNTc5OTIsIDEwLjk3MDk1N10sIFs0Ni4wNTgwNTIsIDEwLjk3MTA0XSwgWzQ2LjA1ODExLCAxMC45NzExNDddLCBbNDYuMDU4MTUxLCAxMC45NzEyMjZdLCBbNDYuMDU4MjE0LCAxMC45NzEyODddLCBbNDYuMDU4MjU1LCAxMC45NzEyNjhdLCBbNDYuMDU4MjksIDEwLjk3MTE5NF0sIFs0Ni4wNTgzMjksIDEwLjk3MTIzN10sIFs0Ni4wNTgzMywgMTAuOTcxMjhdLCBbNDYuMDU4MzMyLCAxMC45NzEzMjZdLCBbNDYuMDU4MzI3LCAxMC45NzE0MTVdLCBbNDYuMDU4MzE5LCAxMC45NzE0ODddLCBbNDYuMDU4MzA3LCAxMC45NzE1NDldLCBbNDYuMDU4MjksIDEwLjk3MTYwNF0sIFs0Ni4wNTgyNjQsIDEwLjk3MTY3OV0sIFs0Ni4wNTgyMzgsIDEwLjk3MTc1NF0sIFs0Ni4wNTgyMDcsIDEwLjk3MTgyM10sIFs0Ni4wNTgxNTQsIDEwLjk3MTg3N10sIFs0Ni4wNTgwOSwgMTAuOTcxOTA5XSwgWzQ2LjA1ODAxOSwgMTAuOTcxOTEzXSwgWzQ2LjA1Nzk1MSwgMTAuOTcxODkyXSwgWzQ2LjA1Nzg5NCwgMTAuOTcxODU4XSwgWzQ2LjA1NzgzNSwgMTAuOTcxODFdLCBbNDYuMDU3Nzg1LCAxMC45NzE3OTNdLCBbNDYuMDU3NzE3LCAxMC45NzE3ODhdLCBbNDYuMDU3NjYyLCAxMC45NzE3OTZdLCBbNDYuMDU3NjA3LCAxMC45NzE4MThdLCBbNDYuMDU3NTQ5LCAxMC45NzE4NjZdLCBbNDYuMDU3NTIzLCAxMC45NzE4OTJdLCBbNDYuMDU3NTA1LCAxMC45NzE5MzFdLCBbNDYuMDU3NDk3LCAxMC45NzE5NzddLCBbNDYuMDU3NDg5LCAxMC45NzIwNjhdLCBbNDYuMDU3NDg2LCAxMC45NzIxMzddLCBbNDYuMDU3NDgsIDEwLjk3MjIzNl0sIFs0Ni4wNTc0NywgMTAuOTcyMzU3XSwgWzQ2LjA1NzQ1OCwgMTAuOTcyNTY0XSwgWzQ2LjA1NzQ2MiwgMTAuOTcyNzkxXSwgWzQ2LjA1NzQ3MSwgMTAuOTczMDE4XSwgWzQ2LjA1NzQ4NywgMTAuOTczMjY0XSwgWzQ2LjA1NzQ5NiwgMTAuOTczNDQ1XSwgWzQ2LjA1NzUxOSwgMTAuOTczNjM3XSwgWzQ2LjA1NzU1NywgMTAuOTczODQ4XSwgWzQ2LjA1NzY0MiwgMTAuOTc0MTkyXSwgWzQ2LjA1NzY2NiwgMTAuOTc0MzI0XSwgWzQ2LjA1NzY4NywgMTAuOTc0NTExXSwgWzQ2LjA1NzcxMSwgMTAuOTc0NzQ1XSwgWzQ2LjA1Nzc0MSwgMTAuOTc0OTY2XSwgWzQ2LjA1Nzc3OSwgMTAuOTc1MTc3XSwgWzQ2LjA1NzgyMSwgMTAuOTc1MzM2XSwgWzQ2LjA1Nzg3NCwgMTAuOTc1NDc1XSwgWzQ2LjA1Nzk0MywgMTAuOTc1NjQ4XSwgWzQ2LjA1ODAyNywgMTAuOTc1ODU3XSwgWzQ2LjA1ODEyMSwgMTAuOTc2MDRdLCBbNDYuMDU4MTk1LCAxMC45NzYxNzZdLCBbNDYuMDU4MjcxLCAxMC45NzYzMDZdLCBbNDYuMDU4MzUxLCAxMC45NzY0NDNdLCBbNDYuMDU4NDAzLCAxMC45NzY1NDZdLCBbNDYuMDU4NDY1LCAxMC45NzY2NjldLCBbNDYuMDU4NTM5LCAxMC45NzY4MDldLCBbNDYuMDU4NjIyLCAxMC45NzY5NTZdLCBbNDYuMDU4Njg5LCAxMC45NzcwODVdLCBbNDYuMDU4NzYsIDEwLjk3NzIwOV0sIFs0Ni4wNTg4MTksIDEwLjk3NzI5OV0sIFs0Ni4wNTg4NzUsIDEwLjk3NzM3M10sIFs0Ni4wNTg5MiwgMTAuOTc3NDAzXSwgWzQ2LjA1OSwgMTAuOTc3NDA4XSwgWzQ2LjA1OTA2NywgMTAuOTc3NDFdLCBbNDYuMDU5MTE0LCAxMC45Nzc0MTddXV1dLCAidHlwZSI6ICJNdWx0aVBvbHlnb24ifSwgImlkIjogIjgiLCAicHJvcGVydGllcyI6IHsiZGVzY3JpemlvbmVfb3JpZ2luZSI6ICJOb24gY29kaWZpY2F0byIsICJnbWxfaWQiOiAiSUQuQUNRVUVGSVNJQ0hFLlNQRUNDSEkuQUNRVUEuNTQxMzMiLCAibmFzcCI6IDAsICJuYXR1cmFfc3BlY2NoaW9fYWNxdWEiOiAiTm9uIGNvZGlmaWNhdG8iLCAibm9tZSI6ICJMQUdPIERJIFRPQkxJTk8iLCAib3JpZ2luZSI6IDB9LCAidHlwZSI6ICJGZWF0dXJlIn0sIHsiZ2VvbWV0cnkiOiB7ImNvb3JkaW5hdGVzIjogW1tbWzQ1Ljg2OSwgMTAuOTE4MDU0XSwgWzQ1Ljg2ODk3MywgMTAuOTE4MDExXSwgWzQ1Ljg2ODk0OSwgMTAuOTE3OTU0XSwgWzQ1Ljg2ODkzMSwgMTAuOTE3ODg4XSwgWzQ1Ljg2ODkxLCAxMC45MTc3NzldLCBbNDUuODY4ODksIDEwLjkxNzYxMl0sIFs0NS44Njg4NywgMTAuOTE3NDQ3XSwgWzQ1Ljg2ODg1OCwgMTAuOTE3MzA5XSwgWzQ1Ljg2ODg0NywgMTAuOTE3MTE5XSwgWzQ1Ljg2ODg0NSwgMTAuOTE2OTg0XSwgWzQ1Ljg2ODg1NiwgMTAuOTE2ODhdLCBbNDUuODY4ODc4LCAxMC45MTY4MDVdLCBbNDUuODY4OTA2LCAxMC45MTY3NTRdLCBbNDUuODY4OTc5LCAxMC45MTY3NDldLCBbNDUuODY5MDE1LCAxMC45MTY3OTNdLCBbNDUuODY5MDYsIDEwLjkxNjg1M10sIFs0NS44NjkxLCAxMC45MTY5MjddLCBbNDUuODY5MTM2LCAxMC45MTddLCBbNDUuODY5MTY5LCAxMC45MTcwOV0sIFs0NS44NjkxOTcsIDEwLjkxNzE4NV0sIFs0NS44NjkyMTMsIDEwLjkxNzM0XSwgWzQ1Ljg2OTI0MSwgMTAuOTE3NDAzXSwgWzQ1Ljg2OTI1LCAxMC45MTc0NTldLCBbNDUuODY5MjU2LCAxMC45MTc1MTVdLCBbNDUuODY5MjUsIDEwLjkxNzYwM10sIFs0NS44NjkyNDEsIDEwLjkxNzcyMV0sIFs0NS44NjkyMTksIDEwLjkxNzgxMl0sIFs0NS44NjkxNzcsIDEwLjkxNzg3M10sIFs0NS44NjkxMjMsIDEwLjkxNzkzN10sIFs0NS44NjkwNjcsIDEwLjkxODAyXSwgWzQ1Ljg2OTAzLCAxMC45MTgwNTVdLCBbNDUuODY5LCAxMC45MTgwNTRdXV1dLCAidHlwZSI6ICJNdWx0aVBvbHlnb24ifSwgImlkIjogIjkiLCAicHJvcGVydGllcyI6IHsiZGVzY3JpemlvbmVfb3JpZ2luZSI6ICJOb24gY29kaWZpY2F0byIsICJnbWxfaWQiOiAiSUQuQUNRVUVGSVNJQ0hFLlNQRUNDSEkuQUNRVUEuNTQxODgiLCAibmFzcCI6IDAsICJuYXR1cmFfc3BlY2NoaW9fYWNxdWEiOiAiTm9uIGNvZGlmaWNhdG8iLCAibm9tZSI6ICIiLCAib3JpZ2luZSI6IDB9LCAidHlwZSI6ICJGZWF0dXJlIn0sIHsiZ2VvbWV0cnkiOiB7ImNvb3JkaW5hdGVzIjogW1tbWzQ1Ljk5ODExLCAxMC44Mjk1NTddLCBbNDUuOTk4MDYsIDEwLjgyOTU0Ml0sIFs0NS45OTgwMjIsIDEwLjgyOTUwNF0sIFs0NS45OTc5ODIsIDEwLjgyOTQyN10sIFs0NS45OTc5NTQsIDEwLjgyOTMzN10sIFs0NS45OTc5MzcsIDEwLjgyOTI0OF0sIFs0NS45OTc5MTcsIDEwLjgyOTE0OF0sIFs0NS45OTc4ODcsIDEwLjgyOTAyOF0sIFs0NS45OTc4ODIsIDEwLjgyODk0M10sIFs0NS45OTc5MTcsIDEwLjgyODgwM10sIFs0NS45OTc5NDMsIDEwLjgyODc0OF0sIFs0NS45OTc5NjEsIDEwLjgyODY3Nl0sIFs0NS45OTc5NTcsIDEwLjgyODYxNF0sIFs0NS45OTc5NzgsIDEwLjgyODQ4M10sIFs0NS45OTgwNDMsIDEwLjgyODQ1Nl0sIFs0NS45OTgwODIsIDEwLjgyODQ1MV0sIFs0NS45OTgxMjYsIDEwLjgyODQyN10sIFs0NS45OTgxNjEsIDEwLjgyODM4NV0sIFs0NS45OTgxODMsIDEwLjgyODM0N10sIFs0NS45OTgyMDIsIDEwLjgyODMxNV0sIFs0NS45OTgyMjMsIDEwLjgyODI4OV0sIFs0NS45OTgyNzEsIDEwLjgyODI3NV0sIFs0NS45OTgzMiwgMTAuODI4MjY0XSwgWzQ1Ljk5ODQwNSwgMTAuODI4MjI0XSwgWzQ1Ljk5ODQ4OSwgMTAuODI4MTg4XSwgWzQ1Ljk5ODU5MiwgMTAuODI4MTZdLCBbNDUuOTk4Njk4LCAxMC44MjgxMzFdLCBbNDUuOTk4Nzg0LCAxMC44MjgxMDVdLCBbNDUuOTk4ODU4LCAxMC44MjgwODldLCBbNDUuOTk4OTMxLCAxMC44MjgwNzldLCBbNDUuOTk4OTYzLCAxMC44MjgwOTZdLCBbNDUuOTk4OTkyLCAxMC44MjgxNDRdLCBbNDUuOTk5MDIzLCAxMC44MjgyMDFdLCBbNDUuOTk5MDM4LCAxMC44MjgyNDddLCBbNDUuOTk5MDUyLCAxMC44MjgzNTZdLCBbNDUuOTk5MDU5LCAxMC44Mjg0MTldLCBbNDUuOTk5MDg3LCAxMC44Mjg1MTNdLCBbNDUuOTk5MTI5LCAxMC44Mjg1OTddLCBbNDUuOTk5MTUzLCAxMC44Mjg2MjRdLCBbNDUuOTk5MTYxLCAxMC44Mjg2NjldLCBbNDUuOTk5MTU5LCAxMC44Mjg3NjddLCBbNDUuOTk5MTQ5LCAxMC44Mjg4NDVdLCBbNDUuOTk5MTE4LCAxMC44Mjg5MDVdLCBbNDUuOTk4OTk0LCAxMC44MjkwMjhdLCBbNDUuOTk4OTQxLCAxMC44MjkwNzJdLCBbNDUuOTk4ODY5LCAxMC44MjkxMzJdLCBbNDUuOTk4Nzg3LCAxMC44MjkxOTFdLCBbNDUuOTk4NzMxLCAxMC44MjkyMjhdLCBbNDUuOTk4NjU1LCAxMC44MjkyNzddLCBbNDUuOTk4NDE4LCAxMC44Mjk0MTJdLCBbNDUuOTk4MzM5LCAxMC44Mjk0NjVdLCBbNDUuOTk4MjQ2LCAxMC44Mjk1MjddLCBbNDUuOTk4MTEsIDEwLjgyOTU1N11dXV0sICJ0eXBlIjogIk11bHRpUG9seWdvbiJ9LCAiaWQiOiAiMTAiLCAicHJvcGVydGllcyI6IHsiZGVzY3JpemlvbmVfb3JpZ2luZSI6ICJOb24gY29kaWZpY2F0byIsICJnbWxfaWQiOiAiSUQuQUNRVUVGSVNJQ0hFLlNQRUNDSEkuQUNRVUEuNTQxNTgiLCAibmFzcCI6IDAsICJuYXR1cmFfc3BlY2NoaW9fYWNxdWEiOiAiTm9uIGNvZGlmaWNhdG8iLCAibm9tZSI6ICJMQUdPIFRPUkJJRVJBIERJIEZJQVZFXHUwMDI3IDE/IiwgIm9yaWdpbmUiOiAwfSwgInR5cGUiOiAiRmVhdHVyZSJ9XSwgInR5cGUiOiAiRmVhdHVyZUNvbGxlY3Rpb24ifSk7CgogICAgICAgIAo8L3NjcmlwdD4= onload="this.contentDocument.open();this.contentDocument.write(atob(this.getAttribute('data-html')));this.contentDocument.close();" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



There is a problem with the orientation of the axes<br/>
This is a false problem because the official order of coordinates in EPSG:4326 is latitude and longitude.<br/>
Usually Geopandas corrects it alone.<br/>
In this case we need an operation to change the axes orientation<br/>
This function is supplied in the shapely package.<br/>


```
import shapely
```

example with a geometry


```
lakes_inbbox.geometry[0]
```




    
![svg](03_Retrieving_data_from_spatial_database_infrastructures_files/03_Retrieving_data_from_spatial_database_infrastructures_105_0.svg)
    




```
shapely.ops.transform(lambda x, y: (y, x),lakes_inbbox.geometry[0])
```




    
![svg](03_Retrieving_data_from_spatial_database_infrastructures_files/03_Retrieving_data_from_spatial_database_infrastructures_106_0.svg)
    



creation of a function to be use in the *apply* method of pandas


```
def swapxy(geometry):
  geometry = shapely.ops.transform(lambda x, y: (y, x),geometry)
  return geometry
```


```
swapxy(lakes_inbbox.geometry[0])
```




    
![svg](03_Retrieving_data_from_spatial_database_infrastructures_files/03_Retrieving_data_from_spatial_database_infrastructures_109_0.svg)
    




```
lakes_inbbox['geometry'] = lakes_inbbox['geometry'].apply(lambda geometry: swapxy(geometry))
```


```
map_lakes = folium.Map([lakes_inbbox.unary_union.centroid.y,lakes_inbbox.unary_union.centroid.x], zoom_start=11)
folium.GeoJson(lakes_inbbox.to_json()).add_to(map_lakes)
map_lakes
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe src="about:blank" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" data-html=PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgCiAgICAgICAgPHNjcmlwdD4KICAgICAgICAgICAgTF9OT19UT1VDSCA9IGZhbHNlOwogICAgICAgICAgICBMX0RJU0FCTEVfM0QgPSBmYWxzZTsKICAgICAgICA8L3NjcmlwdD4KICAgIAogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjYuMC9kaXN0L2xlYWZsZXQuanMiPjwvc2NyaXB0PgogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY29kZS5qcXVlcnkuY29tL2pxdWVyeS0xLjEyLjQubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9qcy9ib290c3RyYXAubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5qcyI+PC9zY3JpcHQ+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjYuMC9kaXN0L2xlYWZsZXQuY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vYm9vdHN0cmFwLzMuMi4wL2Nzcy9ib290c3RyYXAubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLXRoZW1lLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9mb250LWF3ZXNvbWUvNC42LjMvY3NzL2ZvbnQtYXdlc29tZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuanMuY2xvdWRmbGFyZS5jb20vYWpheC9saWJzL0xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLzIuMC4yL2xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvZ2gvcHl0aG9uLXZpc3VhbGl6YXRpb24vZm9saXVtL2ZvbGl1bS90ZW1wbGF0ZXMvbGVhZmxldC5hd2Vzb21lLnJvdGF0ZS5taW4uY3NzIi8+CiAgICA8c3R5bGU+aHRtbCwgYm9keSB7d2lkdGg6IDEwMCU7aGVpZ2h0OiAxMDAlO21hcmdpbjogMDtwYWRkaW5nOiAwO308L3N0eWxlPgogICAgPHN0eWxlPiNtYXAge3Bvc2l0aW9uOmFic29sdXRlO3RvcDowO2JvdHRvbTowO3JpZ2h0OjA7bGVmdDowO308L3N0eWxlPgogICAgCiAgICAgICAgICAgIDxtZXRhIG5hbWU9InZpZXdwb3J0IiBjb250ZW50PSJ3aWR0aD1kZXZpY2Utd2lkdGgsCiAgICAgICAgICAgICAgICBpbml0aWFsLXNjYWxlPTEuMCwgbWF4aW11bS1zY2FsZT0xLjAsIHVzZXItc2NhbGFibGU9bm8iIC8+CiAgICAgICAgICAgIDxzdHlsZT4KICAgICAgICAgICAgICAgICNtYXBfNDZmM2E0MTdmNWU2NDZlNWFkZGQ3MjUxYzI4ZGVhOWEgewogICAgICAgICAgICAgICAgICAgIHBvc2l0aW9uOiByZWxhdGl2ZTsKICAgICAgICAgICAgICAgICAgICB3aWR0aDogMTAwLjAlOwogICAgICAgICAgICAgICAgICAgIGhlaWdodDogMTAwLjAlOwogICAgICAgICAgICAgICAgICAgIGxlZnQ6IDAuMCU7CiAgICAgICAgICAgICAgICAgICAgdG9wOiAwLjAlOwogICAgICAgICAgICAgICAgfQogICAgICAgICAgICA8L3N0eWxlPgogICAgICAgIAo8L2hlYWQ+Cjxib2R5PiAgICAKICAgIAogICAgICAgICAgICA8ZGl2IGNsYXNzPSJmb2xpdW0tbWFwIiBpZD0ibWFwXzQ2ZjNhNDE3ZjVlNjQ2ZTVhZGRkNzI1MWMyOGRlYTlhIiA+PC9kaXY+CiAgICAgICAgCjwvYm9keT4KPHNjcmlwdD4gICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcF80NmYzYTQxN2Y1ZTY0NmU1YWRkZDcyNTFjMjhkZWE5YSA9IEwubWFwKAogICAgICAgICAgICAgICAgIm1hcF80NmYzYTQxN2Y1ZTY0NmU1YWRkZDcyNTFjMjhkZWE5YSIsCiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgY2VudGVyOiBbNDUuODczNDkwOTA5OTQ4MzQsIDEwLjg2MTg0NDkzMjA1ODk5NV0sCiAgICAgICAgICAgICAgICAgICAgY3JzOiBMLkNSUy5FUFNHMzg1NywKICAgICAgICAgICAgICAgICAgICB6b29tOiAxMSwKICAgICAgICAgICAgICAgICAgICB6b29tQ29udHJvbDogdHJ1ZSwKICAgICAgICAgICAgICAgICAgICBwcmVmZXJDYW52YXM6IGZhbHNlLAogICAgICAgICAgICAgICAgfQogICAgICAgICAgICApOwoKICAgICAgICAgICAgCgogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciB0aWxlX2xheWVyXzFkODQ5ZDhkMGUwYjRjMWM5MWU2MGE0OTA3MmYyNGU5ID0gTC50aWxlTGF5ZXIoCiAgICAgICAgICAgICAgICAiaHR0cHM6Ly97c30udGlsZS5vcGVuc3RyZWV0bWFwLm9yZy97en0ve3h9L3t5fS5wbmciLAogICAgICAgICAgICAgICAgeyJhdHRyaWJ1dGlvbiI6ICJEYXRhIGJ5IFx1MDAyNmNvcHk7IFx1MDAzY2EgaHJlZj1cImh0dHA6Ly9vcGVuc3RyZWV0bWFwLm9yZ1wiXHUwMDNlT3BlblN0cmVldE1hcFx1MDAzYy9hXHUwMDNlLCB1bmRlciBcdTAwM2NhIGhyZWY9XCJodHRwOi8vd3d3Lm9wZW5zdHJlZXRtYXAub3JnL2NvcHlyaWdodFwiXHUwMDNlT0RiTFx1MDAzYy9hXHUwMDNlLiIsICJkZXRlY3RSZXRpbmEiOiBmYWxzZSwgIm1heE5hdGl2ZVpvb20iOiAxOCwgIm1heFpvb20iOiAxOCwgIm1pblpvb20iOiAwLCAibm9XcmFwIjogZmFsc2UsICJvcGFjaXR5IjogMSwgInN1YmRvbWFpbnMiOiAiYWJjIiwgInRtcyI6IGZhbHNlfQogICAgICAgICAgICApLmFkZFRvKG1hcF80NmYzYTQxN2Y1ZTY0NmU1YWRkZDcyNTFjMjhkZWE5YSk7CiAgICAgICAgCiAgICAKICAgICAgICBmdW5jdGlvbiBnZW9fanNvbl9mNjgzZGY3ZGJmZTQ0ZTQ1YTIxZDc3ZTA5YTM2NzFlNV9vbkVhY2hGZWF0dXJlKGZlYXR1cmUsIGxheWVyKSB7CiAgICAgICAgICAgIGxheWVyLm9uKHsKICAgICAgICAgICAgfSk7CiAgICAgICAgfTsKICAgICAgICB2YXIgZ2VvX2pzb25fZjY4M2RmN2RiZmU0NGU0NWEyMWQ3N2UwOWEzNjcxZTUgPSBMLmdlb0pzb24obnVsbCwgewogICAgICAgICAgICAgICAgb25FYWNoRmVhdHVyZTogZ2VvX2pzb25fZjY4M2RmN2RiZmU0NGU0NWEyMWQ3N2UwOWEzNjcxZTVfb25FYWNoRmVhdHVyZSwKICAgICAgICAgICAgCiAgICAgICAgfSk7CgogICAgICAgIGZ1bmN0aW9uIGdlb19qc29uX2Y2ODNkZjdkYmZlNDRlNDVhMjFkNzdlMDlhMzY3MWU1X2FkZCAoZGF0YSkgewogICAgICAgICAgICBnZW9fanNvbl9mNjgzZGY3ZGJmZTQ0ZTQ1YTIxZDc3ZTA5YTM2NzFlNQogICAgICAgICAgICAgICAgLmFkZERhdGEoZGF0YSkKICAgICAgICAgICAgICAgIC5hZGRUbyhtYXBfNDZmM2E0MTdmNWU2NDZlNWFkZGQ3MjUxYzI4ZGVhOWEpOwogICAgICAgIH0KICAgICAgICAgICAgZ2VvX2pzb25fZjY4M2RmN2RiZmU0NGU0NWEyMWQ3N2UwOWEzNjcxZTVfYWRkKHsiZmVhdHVyZXMiOiBbeyJnZW9tZXRyeSI6IHsiY29vcmRpbmF0ZXMiOiBbW1tbMTAuOTIzNzY1LCA0NS44NTgzMjRdLCBbMTAuOTIzNzM5LCA0NS44NTgyMjZdLCBbMTAuOTIzNzAxLCA0NS44NTgxNjldLCBbMTAuOTIzNjM3LCA0NS44NTgxMThdLCBbMTAuOTIzNTkxLCA0NS44NTgxMDZdLCBbMTAuOTIzNTM3LCA0NS44NTgwODNdLCBbMTAuOTIzNDI5LCA0NS44NTgwNjRdLCBbMTAuOTIzMzI3LCA0NS44NTgwNTRdLCBbMTAuOTIzMjA5LCA0NS44NTgwNTNdLCBbMTAuOTIzMDc4LCA0NS44NTgwNDldLCBbMTAuOTIyOTAxLCA0NS44NTgwNDJdLCBbMTAuOTIyNzEzLCA0NS44NTgwMjddLCBbMTAuOTIyMjMsIDQ1Ljg1Nzk4Ml0sIFsxMC45MjE5NTgsIDQ1Ljg1Nzk3XSwgWzEwLjkyMTc5NywgNDUuODU3OTUyXSwgWzEwLjkyMTY1MSwgNDUuODU3ODg4XSwgWzEwLjkyMTU2LCA0NS44NTc4MjhdLCBbMTAuOTIxNDM0LCA0NS44NTc3NjhdLCBbMTAuOTIxMjc5LCA0NS44NTc3MzhdLCBbMTAuOTIxMTM5LCA0NS44NTc3NV0sIFsxMC45MjA5OTYsIDQ1Ljg1Nzc4Nl0sIFsxMC45MjA5MDksIDQ1Ljg1NzgyNl0sIFsxMC45MjA4LCA0NS44NTc5MDRdLCBbMTAuOTIwNjkxLCA0NS44NTc5OTldLCBbMTAuOTIwNTU1LCA0NS44NTgxNTNdLCBbMTAuOTIwNDY3LCA0NS44NTgyNTVdLCBbMTAuOTIwNDUsIDQ1Ljg1ODM0XSwgWzEwLjkyMDQ3LCA0NS44NTg0NDldLCBbMTAuOTIwNTY1LCA0NS44NTg1NTFdLCBbMTAuOTIwNjUxLCA0NS44NTg1ODldLCBbMTAuOTIwNzEsIDQ1Ljg1ODYwNl0sIFsxMC45MjA3NjQsIDQ1Ljg1ODY1MV0sIFsxMC45MjA4MjMsIDQ1Ljg1ODc1OF0sIFsxMC45MjA4NywgNDUuODU4ODg1XSwgWzEwLjkyMDksIDQ1Ljg1OTAxNV0sIFsxMC45MjA5MDMsIDQ1Ljg1OTEwMl0sIFsxMC45MjA4ODYsIDQ1Ljg1OTE5Ml0sIFsxMC45MjA4MTgsIDQ1Ljg1OTMzXSwgWzEwLjkyMDgyMywgNDUuODU5Mzc4XSwgWzEwLjkyMDg2MywgNDUuODU5NDldLCBbMTAuOTIwNzYxLCA0NS44NTk1ODNdLCBbMTAuOTIwNTc4LCA0NS44NTk1OTddLCBbMTAuOTIwNTI1LCA0NS44NTk1ODddLCBbMTAuOTIwNDQ2LCA0NS44NTk1NzddLCBbMTAuOTIwMzg0LCA0NS44NTk1NzVdLCBbMTAuOTIwMjk3LCA0NS44NTk2MDZdLCBbMTAuOTIwMjEyLCA0NS44NTk2MjJdLCBbMTAuOTIwMDk3LCA0NS44NTk2M10sIFsxMC45MjAwMjIsIDQ1Ljg1OTYxOF0sIFsxMC45MTk5NDksIDQ1Ljg1OTYwMV0sIFsxMC45MTk4NjQsIDQ1Ljg1OTYwNF0sIFsxMC45MTk4MTIsIDQ1Ljg1OTYxOV0sIFsxMC45MTk3ODQsIDQ1Ljg1OTY1MV0sIFsxMC45MTk3ODUsIDQ1Ljg1OTY5XSwgWzEwLjkxOTgsIDQ1Ljg1OTc1OV0sIFsxMC45MTk4NjgsIDQ1Ljg1OTgzM10sIFsxMC45MTk5NjYsIDQ1Ljg1OTkxOV0sIFsxMC45MjAwMzMsIDQ1Ljg1OTk3N10sIFsxMC45MjAwODgsIDQ1Ljg2MDA1XSwgWzEwLjkyMDEyMiwgNDUuODYwMTAyXSwgWzEwLjkyMDExNCwgNDUuODYwMTU5XSwgWzEwLjkyMDA3LCA0NS44NjAyMTJdLCBbMTAuOTIwMDAyLCA0NS44NjAyNDFdLCBbMTAuOTE5OTMzLCA0NS44NjAyNDRdLCBbMTAuOTE5ODQ0LCA0NS44NjAyMjVdLCBbMTAuOTE5NzM1LCA0NS44NjAxODhdLCBbMTAuOTE5NjUyLCA0NS44NjAxNTNdLCBbMTAuOTE5NTc2LCA0NS44NjAxMjRdLCBbMTAuOTE5NDY0LCA0NS44NjAwOThdLCBbMTAuOTE5NDI0LCA0NS44NjAxMDFdLCBbMTAuOTE5NDEyLCA0NS44NjAxMzFdLCBbMTAuOTE5NDMsIDQ1Ljg2MDE3N10sIFsxMC45MTk0NTcsIDQ1Ljg2MDIwNl0sIFsxMC45MTk1MDQsIDQ1Ljg2MDI0XSwgWzEwLjkxOTU3MSwgNDUuODYwMjcxXSwgWzEwLjkxOTY1MywgNDUuODYwMzA0XSwgWzEwLjkxOTc2MywgNDUuODYwMzUyXSwgWzEwLjkxOTg2LCA0NS44NjA0MDRdLCBbMTAuOTE5OTMsIDQ1Ljg2MDQ1M10sIFsxMC45MTk5NTQsIDQ1Ljg2MDQ5MV0sIFsxMC45MTk5MjEsIDQ1Ljg2MDU2N10sIFsxMC45MTk4ODIsIDQ1Ljg2MDU5Nl0sIFsxMC45MTk3NzksIDQ1Ljg2MDYzNF0sIFsxMC45MTk2NjgsIDQ1Ljg2MDY0NV0sIFsxMC45MTk1NSwgNDUuODYwNjQ0XSwgWzEwLjkxOTI5LCA0NS44NjA3MTddLCBbMTAuOTE5MTg0LCA0NS44NjA3NzhdLCBbMTAuOTE5MTA2LCA0NS44NjA4MTZdLCBbMTAuOTE5MDM4LCA0NS44NjA4NDddLCBbMTAuOTE4OTMxLCA0NS44NjA4ODVdLCBbMTAuOTE4ODE3LCA0NS44NjA5MDNdLCBbMTAuOTE4Njk5LCA0NS44NjA5MDVdLCBbMTAuOTE4NTg4LCA0NS44NjA5MDVdLCBbMTAuOTE4NDcxLCA0NS44NjA5MzJdLCBbMTAuOTE4MzQ0LCA0NS44NjA5NTldLCBbMTAuOTE4MjUsIDQ1Ljg2MDk4M10sIFsxMC45MTgxMzcsIDQ1Ljg2MTA1MV0sIFsxMC45MTgwMDEsIDQ1Ljg2MTIwNV0sIFsxMC45MTc5MTcsIDQ1Ljg2MTM1XSwgWzEwLjkxNzgxNCwgNDUuODYxNDk4XSwgWzEwLjkxNzczNywgNDUuODYxNjQ2XSwgWzEwLjkxNzY1NiwgNDUuODYxNzkyXSwgWzEwLjkxNzUyMywgNDUuODYyMDMyXSwgWzEwLjkxNzQyNywgNDUuODYyMjAzXSwgWzEwLjkxNzMyOCwgNDUuODYyNDAyXSwgWzEwLjkxNzI0OSwgNDUuODYyNTgyXSwgWzEwLjkxNzIxNCwgNDUuODYyODI0XSwgWzEwLjkxNzE5MiwgNDUuODYyOTc2XSwgWzEwLjkxNzIxMiwgNDUuODYzMTg0XSwgWzEwLjkxNzIzNCwgNDUuODYzMzY1XSwgWzEwLjkxNzI0NSwgNDUuODYzNTA1XSwgWzEwLjkxNzIzMywgNDUuODYzNjQ3XSwgWzEwLjkxNzE4MywgNDUuODYzODIyXSwgWzEwLjkxNzExMiwgNDUuODYzOTY3XSwgWzEwLjkxNzAxOCwgNDUuODY0MDk5XSwgWzEwLjkxNjg5NCwgNDUuODY0MjM0XSwgWzEwLjkxNjgzNCwgNDUuODY0Mjk0XSwgWzEwLjkxNjcwNywgNDUuODY0NDA5XSwgWzEwLjkxNjU4MiwgNDUuODY0NV0sIFsxMC45MTY1MDksIDQ1Ljg2NDU3N10sIFsxMC45MTY0OTIsIDQ1Ljg2NDY1Ml0sIFsxMC45MTY1MTQsIDQ1Ljg2NDczXSwgWzEwLjkxNjU1NywgNDUuODY0NzU0XSwgWzEwLjkxNjYyLCA0NS44NjQ3NjddLCBbMTAuOTE2Njc4LCA0NS44NjQ3NTNdLCBbMTAuOTE2Nzc1LCA0NS44NjQ2OThdLCBbMTAuOTE2ODU1LCA0NS44NjQ2MzhdLCBbMTAuOTE2ODksIDQ1Ljg2NDZdLCBbMTAuOTE3MDEzLCA0NS44NjQ0NjFdLCBbMTAuOTE3MTEyLCA0NS44NjQzNjhdLCBbMTAuOTE3MjE0LCA0NS44NjQyODRdLCBbMTAuOTE3MzMyLCA0NS44NjQxOTldLCBbMTAuOTE3NDIzLCA0NS44NjQxNjZdLCBbMTAuOTE3NTA4LCA0NS44NjQxNThdLCBbMTAuOTE3NzU5LCA0NS44NjQyMjVdLCBbMTAuOTE3ODIzLCA0NS44NjQyNjNdLCBbMTAuOTE3OTA3LCA0NS44NjQzMzJdLCBbMTAuOTE3OTc4LCA0NS44NjQ0MTRdLCBbMTAuOTE4MDM2LCA0NS44NjQ0ODhdLCBbMTAuOTE4MDc5LCA0NS44NjQ1ODhdLCBbMTAuOTE4MDk4LCA0NS44NjQ2ODldLCBbMTAuOTE4MDgzLCA0NS44NjQ4MTVdLCBbMTAuOTE4MDYxLCA0NS44NjQ5NTNdLCBbMTAuOTE3OTk5LCA0NS44NjUwNzVdLCBbMTAuOTE3OTA5LCA0NS44NjUyMjFdLCBbMTAuOTE3OCwgNDUuODY1NDA2XSwgWzEwLjkxNzY4NywgNDUuODY1NTg0XSwgWzEwLjkxNzU1MiwgNDUuODY1NzU2XSwgWzEwLjkxNzM4NywgNDUuODY1OTE2XSwgWzEwLjkxNzIwMSwgNDUuODY2MDU0XSwgWzEwLjkxNzA2MiwgNDUuODY2MTA5XSwgWzEwLjkxNjkzNSwgNDUuODY2MTMyXSwgWzEwLjkxNjgxMywgNDUuODY2MTI5XSwgWzEwLjkxNjcyMSwgNDUuODY2MDk0XSwgWzEwLjkxNjYyOSwgNDUuODY2MDA0XSwgWzEwLjkxNjU0NCwgNDUuODY1OTExXSwgWzEwLjkxNjQ2MywgNDUuODY1ODE0XSwgWzEwLjkxNjM5NCwgNDUuODY1NzE3XSwgWzEwLjkxNjMzMSwgNDUuODY1NTk2XSwgWzEwLjkxNjI3OCwgNDUuODY1NDZdLCBbMTAuOTE2Mjc2LCA0NS44NjUzMDldLCBbMTAuOTE2MjYsIDQ1Ljg2NTIwM10sIFsxMC45MTYyMTksIDQ1Ljg2NTE1MV0sIFsxMC45MTYxNSwgNDUuODY1MTVdLCBbMTAuOTE2MTA4LCA0NS44NjUxNjddLCBbMTAuOTE2MDQxLCA0NS44NjUyMThdLCBbMTAuOTE1OTc3LCA0NS44NjUyODFdLCBbMTAuOTE1ODkxLCA0NS44NjUzNjNdLCBbMTAuOTE1ODAyLCA0NS44NjU0NF0sIFsxMC45MTU3MSwgNDUuODY1NTM1XSwgWzEwLjkxNTY0OSwgNDUuODY1NTg0XSwgWzEwLjkxNTYsIDQ1Ljg2NTY3OV0sIFsxMC45MTU1NzQsIDQ1Ljg2NTc0XSwgWzEwLjkxNTU1LCA0NS44NjU4MV0sIFsxMC45MTU0ODEsIDQ1Ljg2NTkwN10sIFsxMC45MTU0MDUsIDQ1Ljg2NjAwN10sIFsxMC45MTUzMTcsIDQ1Ljg2NjEyM10sIFsxMC45MTUyNDUsIDQ1Ljg2NjIyOV0sIFsxMC45MTUxNjcsIDQ1Ljg2NjM2Nl0sIFsxMC45MTUxMTYsIDQ1Ljg2NjUwNF0sIFsxMC45MTUwODcsIDQ1Ljg2NjYyMV0sIFsxMC45MTUwNzIsIDQ1Ljg2Njc1N10sIFsxMC45MTUwNiwgNDUuODY2OTAxXSwgWzEwLjkxNTA1NCwgNDUuODY3MDI5XSwgWzEwLjkxNTA0NSwgNDUuODY3MTY1XSwgWzEwLjkxNTAzOSwgNDUuODY3MjkzXSwgWzEwLjkxNTAxNywgNDUuODY3NDI5XSwgWzEwLjkxNDk4LCA0NS44Njc1OF0sIFsxMC45MTQ5MzksIDQ1Ljg2NzczNF0sIFsxMC45MTQ4ODcsIDQ1Ljg2Nzg2MV0sIFsxMC45MTQ3NzMsIDQ1Ljg2Nzk4OV0sIFsxMC45MTQ2MTIsIDQ1Ljg2ODA5Ml0sIFsxMC45MTQ0MjIsIDQ1Ljg2ODE5Nl0sIFsxMC45MTQyMzEsIDQ1Ljg2ODI2XSwgWzEwLjkxNDA4NCwgNDUuODY4Mjc5XSwgWzEwLjkxMzg0NiwgNDUuODY4MzFdLCBbMTAuOTEzNjQzLCA0NS44NjgzMl0sIFsxMC45MTM0MTQsIDQ1Ljg2ODMyMV0sIFsxMC45MTMyMTEsIDQ1Ljg2ODMzMV0sIFsxMC45MTMwNTgsIDQ1Ljg2ODM2MV0sIFsxMC45MTI5NTQsIDQ1Ljg2ODQwNF0sIFsxMC45MTI4NzIsIDQ1Ljg2ODQ4N10sIFsxMC45MTI4MzEsIDQ1Ljg2ODU2M10sIFsxMC45MTI4MzQsIDQ1Ljg2ODY1M10sIFsxMC45MTI4NzYsIDQ1Ljg2ODcyOF0sIFsxMC45MTI5NjcsIDQ1Ljg2ODgxNl0sIFsxMC45MTMwNjEsIDQ1Ljg2ODg5Ml0sIFsxMC45MTMxNTIsIDQ1Ljg2ODk2NF0sIFsxMC45MTMyNDMsIDQ1Ljg2OTAzOF0sIFsxMC45MTMzMzcsIDQ1Ljg2OTExNV0sIFsxMC45MTM0MzEsIDQ1Ljg2OTE5MV0sIFsxMC45MTM1MTgsIDQ1Ljg2OTIzM10sIFsxMC45MTM2MTQsIDQ1Ljg2OTI1OV0sIFsxMC45MTM3MTUsIDQ1Ljg2OTI2XSwgWzEwLjkxMzgwMywgNDUuODY5MjUyXSwgWzEwLjkxMzkwMiwgNDUuODY5MjQ4XSwgWzEwLjkxNDAyMywgNDUuODY5MjQ0XSwgWzEwLjkxNDE2NCwgNDUuODY5MjQ2XSwgWzEwLjkxNDQ1NSwgNDUuODY5MjM3XSwgWzEwLjkxNDYxOSwgNDUuODY5MjIzXSwgWzEwLjkxNDczOSwgNDUuODY5MjAzXSwgWzEwLjkxNDk3MiwgNDUuODY5Ml0sIFsxMC45MTUxNjIsIDQ1Ljg2OTIwNF0sIFsxMC45MTUzMzUsIDQ1Ljg2OTE5NF0sIFsxMC45MTU0NzksIDQ1Ljg2OTE3Nl0sIFsxMC45MTU2NzYsIDQ1Ljg2OTE3NV0sIFsxMC45MTU3NjMsIDQ1Ljg2OTE5OV0sIFsxMC45MTU4MzksIDQ1Ljg2OTIxNl0sIFsxMC45MTU5NDIsIDQ1Ljg2OTI0Nl0sIFsxMC45MTYwNTIsIDQ1Ljg2OTI5NV0sIFsxMC45MTYxMDIsIDQ1Ljg2OTMyMl0sIFsxMC45MTYxNDksIDQ1Ljg2OTM2XSwgWzEwLjkxNjE3OSwgNDUuODY5Mzk2XSwgWzEwLjkxNjIyMiwgNDUuODY5NTAxXSwgWzEwLjkxNjIzNCwgNDUuODY5NTg3XSwgWzEwLjkxNjE4OCwgNDUuODY5NjU5XSwgWzEwLjkxNjA5NSwgNDUuODY5NzQzXSwgWzEwLjkxNjAxNiwgNDUuODY5ODMxXSwgWzEwLjkxNjAwMiwgNDUuODY5ODk4XSwgWzEwLjkxNjAxNywgNDUuODY5OTUzXSwgWzEwLjkxNjA3MSwgNDUuODcwMDE2XSwgWzEwLjkxNjE1NSwgNDUuODcwMDddLCBbMTAuOTE2MjI0LCA0NS44NzAwODldLCBbMTAuOTE2MzI5LCA0NS44NzAxMDFdLCBbMTAuOTE2NDI0LCA0NS44NzAwOTFdLCBbMTAuOTE2NTM5LCA0NS44NzAwOTZdLCBbMTAuOTE2NjgxLCA0NS44NzAxMjVdLCBbMTAuOTE2ODAzLCA0NS44NzAxNjVdLCBbMTAuOTE2OTEsIDQ1Ljg3MDIyM10sIFsxMC45MTY5NjcsIDQ1Ljg3MDI3NF0sIFsxMC45MTcwNDgsIDQ1Ljg3MDM0Ml0sIFsxMC45MTcxMTQsIDQ1Ljg3MDM1N10sIFsxMC45MTcyMjgsIDQ1Ljg3MDM0Nl0sIFsxMC45MTc0NjgsIDQ1Ljg3MDI2Ml0sIFsxMC45MTc1OTYsIDQ1Ljg3MDE3MV0sIFsxMC45MTc2NjYsIDQ1Ljg3MDA5Nl0sIFsxMC45MTc3MTksIDQ1Ljg3MDAxM10sIFsxMC45MTc3NjYsIDQ1Ljg2OTkzMl0sIFsxMC45MTc4MDksIDQ1Ljg2OTg0NF0sIFsxMC45MTc4MzksIDQ1Ljg2OTc1Ml0sIFsxMC45MTc4MzksIDQ1Ljg2OTY0N10sIFsxMC45MTc4MzIsIDQ1Ljg2OTU0OF0sIFsxMC45MTc4NDMsIDQ1Ljg2OTQ3N10sIFsxMC45MTc5NDIsIDQ1Ljg2OTQwNV0sIFsxMC45MTgxMDIsIDQ1Ljg2OTM2M10sIFsxMC45MTgyNTQsIDQ1Ljg2OTMyNF0sIFsxMC45MTgzODQsIDQ1Ljg2OTI3Nl0sIFsxMC45MTg0NjcsIDQ1Ljg2OTIyMl0sIFsxMC45MTg1MTEsIDQ1Ljg2OTE0Nl0sIFsxMC45MTg1MDIsIDQ1Ljg2OTA1N10sIFsxMC45MTg0NzUsIDQ1Ljg2ODkzNF0sIFsxMC45MTg0NjIsIDQ1Ljg2ODgzOF0sIFsxMC45MTg0NywgNDUuODY4Nzg1XSwgWzEwLjkxODUyNCwgNDUuODY4NzQzXSwgWzEwLjkxODYsIDQ1Ljg2ODc1M10sIFsxMC45MTg2OSwgNDUuODY4OF0sIFsxMC45MTg3NDcsIDQ1Ljg2ODg1Ml0sIFsxMC45MTg3OTUsIDQ1Ljg2ODkxOV0sIFsxMC45MTg4NTMsIDQ1Ljg2ODk4XSwgWzEwLjkxODkwOSwgNDUuODY4OTkzXSwgWzEwLjkxODk5MSwgNDUuODY4OTg3XSwgWzEwLjkxOTAzLCA0NS44Njg5NzVdLCBbMTAuOTE5MTA0LCA0NS44Njg5MzddLCBbMTAuOTE5MTksIDQ1Ljg2ODg0OV0sIFsxMC45MTkzMDEsIDQ1Ljg2ODczM10sIFsxMC45MTk0NTYsIDQ1Ljg2ODU2OF0sIFsxMC45MTk1NjcsIDQ1Ljg2ODQ1OF0sIFsxMC45MTk2ODEsIDQ1Ljg2ODMxN10sIFsxMC45MTk3MDcsIDQ1Ljg2ODIwN10sIFsxMC45MTk2ODksIDQ1Ljg2ODA2OV0sIFsxMC45MTk2NDMsIDQ1Ljg2Nzk1Nl0sIFsxMC45MTk2MDEsIDQ1Ljg2Nzg4MV0sIFsxMC45MTk1OTIsIDQ1Ljg2NzgwOF0sIFsxMC45MTk2MDMsIDQ1Ljg2Nzc0MV0sIFsxMC45MTk2NTEsIDQ1Ljg2NzY3OF0sIFsxMC45MTk2OTgsIDQ1Ljg2NzYyNV0sIFsxMC45MTk3OTEsIDQ1Ljg2NzU0Nl0sIFsxMC45MTk5MjUsIDQ1Ljg2NzQ0M10sIFsxMC45MjAxMDUsIDQ1Ljg2NzMyM10sIFsxMC45MjAyNzEsIDQ1Ljg2NzIwOF0sIFsxMC45MjA0ODksIDQ1Ljg2NzA1OF0sIFsxMC45MjA3NTMsIDQ1Ljg2Njg4N10sIFsxMC45MjEwMzksIDQ1Ljg2NjcwNl0sIFsxMC45MjEyNDcsIDQ1Ljg2NjU3Ml0sIFsxMC45MjE0NTYsIDQ1Ljg2NjQ1XSwgWzEwLjkyMTY4OSwgNDUuODY2MzQxXSwgWzEwLjkyMTk1LCA0NS44NjYyMTFdLCBbMTAuOTIyMTc3LCA0NS44NjYwMzNdLCBbMTAuOTIyMzAzLCA0NS44NjU4NjJdLCBbMTAuOTIyMzYsIDQ1Ljg2NTY4OV0sIFsxMC45MjIzNzUsIDQ1Ljg2NTU0N10sIFsxMC45MjIzNjcsIDQ1Ljg2NTM5Nl0sIFsxMC45MjIzMzUsIDQ1Ljg2NTIxN10sIFsxMC45MjIyOTcsIDQ1Ljg2NTA1NV0sIFsxMC45MjIyODQsIDQ1Ljg2NDk3MV0sIFsxMC45MjIyODIsIDQ1Ljg2NDg3N10sIFsxMC45MjIyOTQsIDQ1Ljg2NDc2Ml0sIFsxMC45MjIzNTIsIDQ1Ljg2NDUxXSwgWzEwLjkyMjM3NSwgNDUuODY0MzEyXSwgWzEwLjkyMjM1OSwgNDUuODY0MTAyXSwgWzEwLjkyMjMyNywgNDUuODYzOTE5XSwgWzEwLjkyMjMzNCwgNDUuODYzNzQ3XSwgWzEwLjkyMjM2MiwgNDUuODYzNTkxXSwgWzEwLjkyMjQwNCwgNDUuODYzNDZdLCBbMTAuOTIyNDU5LCA0NS44NjMzMzNdLCBbMTAuOTIyNTIyLCA0NS44NjMxNjldLCBbMTAuOTIyNTM4LCA0NS44NjMwNjhdLCBbMTAuOTIyNTUsIDQ1Ljg2Mjk5N10sIFsxMC45MjI1NjEsIDQ1Ljg2Mjg0Nl0sIFsxMC45MjI1NTksIDQ1Ljg2Mjc2M10sIFsxMC45MjI1MjcsIDQ1Ljg2MjU4MV0sIFsxMC45MjI1MDIsIDQ1Ljg2MjQyMV0sIFsxMC45MjI1LCA0NS44NjIyNDldLCBbMTAuOTIyNTI3LCA0NS44NjIwNzJdLCBbMTAuOTIyNTY3LCA0NS44NjE4ODZdLCBbMTAuOTIyNTg4LCA0NS44NjE3MTZdLCBbMTAuOTIyNTksIDQ1Ljg2MTU2N10sIFsxMC45MjI2MTEsIDQ1Ljg2MTQyM10sIFsxMC45MjI2ODIsIDQ1Ljg2MTI2NF0sIFsxMC45MjI3NDQsIDQ1Ljg2MTE1M10sIFsxMC45MjI4MTksIDQ1Ljg2MTA2NF0sIFsxMC45MjI5ODMsIDQ1Ljg2MDk2Nl0sIFsxMC45MjMwMjQsIDQ1Ljg2MDkxMl0sIFsxMC45MjMwMzgsIDQ1Ljg2MDgyNV0sIFsxMC45MjMwMTUsIDQ1Ljg2MDcyN10sIFsxMC45MjI5MjYsIDQ1Ljg2MDUwNl0sIFsxMC45MjI4ODQsIDQ1Ljg2MDQwOF0sIFsxMC45MjI4MjQsIDQ1Ljg2MDI3XSwgWzEwLjkyMjc4LCA0NS44NjAxNDJdLCBbMTAuOTIyNzQyLCA0NS44NTk5NjldLCBbMTAuOTIyNzA0LCA0NS44NTk4MTFdLCBbMTAuOTIyNjc3LCA0NS44NTk2ODFdLCBbMTAuOTIyNjgyLCA0NS44NTk2MDhdLCBbMTAuOTIyNzQsIDQ1Ljg1OTQ3NF0sIFsxMC45MjI4MzEsIDQ1Ljg1OTM0OV0sIFsxMC45MjI5MzIsIDQ1Ljg1OTI1MV0sIFsxMC45MjMwMjEsIDQ1Ljg1OTE2OV0sIFsxMC45MjMyNTIsIDQ1Ljg1ODk5NF0sIFsxMC45MjMzMTksIDQ1Ljg1ODk1Ml0sIFsxMC45MjM0NDEsIDQ1Ljg1ODg3OV0sIFsxMC45MjM2MDUsIDQ1Ljg1ODc4XSwgWzEwLjkyMzcwMSwgNDUuODU4NjhdLCBbMTAuOTIzNzUzLCA0NS44NTg1NjldLCBbMTAuOTIzNzY5LCA0NS44NTg0NTJdLCBbMTAuOTIzNzY1LCA0NS44NTgzMjRdXV1dLCAidHlwZSI6ICJNdWx0aVBvbHlnb24ifSwgImlkIjogIjAiLCAicHJvcGVydGllcyI6IHsiZGVzY3JpemlvbmVfb3JpZ2luZSI6ICJOb24gY29kaWZpY2F0byIsICJnbWxfaWQiOiAiSUQuQUNRVUVGSVNJQ0hFLlNQRUNDSEkuQUNRVUEuNTQxODYiLCAibmFzcCI6IDAsICJuYXR1cmFfc3BlY2NoaW9fYWNxdWEiOiAiTm9uIGNvZGlmaWNhdG8iLCAibm9tZSI6ICJMQUdPIERJIExPUFBJTyIsICJvcmlnaW5lIjogMH0sICJ0eXBlIjogIkZlYXR1cmUifSwgeyJnZW9tZXRyeSI6IHsiY29vcmRpbmF0ZXMiOiBbW1tbMTAuOTIyNDEzLCA0NS45ODI0MzZdLCBbMTAuOTIyMzg1LCA0NS45ODIzOF0sIFsxMC45MjIzNTcsIDQ1Ljk4MjMyNV0sIFsxMC45MjIzMzIsIDQ1Ljk4MjIzOF0sIFsxMC45MjIyNzMsIDQ1Ljk4MjEyNF0sIFsxMC45MjIyNjQsIDQ1Ljk4MjAyOF0sIFsxMC45MjIyMjksIDQ1Ljk4MTk1NV0sIFsxMC45MjIxNzMsIDQ1Ljk4MTkyOV0sIFsxMC45MjIxMjEsIDQ1Ljk4MTk1XSwgWzEwLjkyMjA4MiwgNDUuOTgxOTc4XSwgWzEwLjkyMjAwNSwgNDUuOTgyMDM2XSwgWzEwLjkyMTk0NywgNDUuOTgyMDcxXSwgWzEwLjkyMTgxMiwgNDUuOTgyMjA4XSwgWzEwLjkyMTc5MSwgNDUuOTgyMjY4XSwgWzEwLjkyMTc2MSwgNDUuOTgyMzc0XSwgWzEwLjkyMTc5LCA0NS45ODI0NjNdLCBbMTAuOTIxODI1LCA0NS45ODI1NTJdLCBbMTAuOTIxOTEzLCA0NS45ODI2NDddLCBbMTAuOTIxOTU3LCA0NS45ODI3MDFdLCBbMTAuOTIyMDE3LCA0NS45ODI3MjFdLCBbMTAuOTIyMDgzLCA0NS45ODI3NDFdLCBbMTAuOTIyMTQ2LCA0NS45ODI3NDldLCBbMTAuOTIyMjA4LCA0NS45ODI3NDhdLCBbMTAuOTIyMjY2LCA0NS45ODI3MjJdLCBbMTAuOTIyMzExLCA0NS45ODI2NjldLCBbMTAuOTIyMzQ5LCA0NS45ODI2MjVdLCBbMTAuOTIyMzY3LCA0NS45ODI1NjNdLCBbMTAuOTIyMzkxLCA0NS45ODI0ODddLCBbMTAuOTIyNDEzLCA0NS45ODI0MzZdXV1dLCAidHlwZSI6ICJNdWx0aVBvbHlnb24ifSwgImlkIjogIjEiLCAicHJvcGVydGllcyI6IHsiZGVzY3JpemlvbmVfb3JpZ2luZSI6ICJOb24gY29kaWZpY2F0byIsICJnbWxfaWQiOiAiSUQuQUNRVUVGSVNJQ0hFLlNQRUNDSEkuQUNRVUEuNTQxNjIiLCAibmFzcCI6IDAsICJuYXR1cmFfc3BlY2NoaW9fYWNxdWEiOiAiTm9uIGNvZGlmaWNhdG8iLCAibm9tZSI6ICJMQUdPIERFSSBCQUdBVE9JIChEUk8pIiwgIm9yaWdpbmUiOiAwfSwgInR5cGUiOiAiRmVhdHVyZSJ9LCB7Imdlb21ldHJ5IjogeyJjb29yZGluYXRlcyI6IFtbW1sxMC45MzE4NiwgNDUuOTc3MTkxXSwgWzEwLjkzMTgyMywgNDUuOTc3MTQzXSwgWzEwLjkzMTczNywgNDUuOTc3MTQ2XSwgWzEwLjkzMTcwMiwgNDUuOTc3MTc0XSwgWzEwLjkzMTY3MywgNDUuOTc3MTk1XSwgWzEwLjkzMTU5NCwgNDUuOTc3MTk5XSwgWzEwLjkzMTUzNSwgNDUuOTc3MTc3XSwgWzEwLjkzMTQ3NSwgNDUuOTc3MTUyXSwgWzEwLjkzMTQyMSwgNDUuOTc3MDg5XSwgWzEwLjkzMTM2NCwgNDUuOTc3MDUxXSwgWzEwLjkzMTI4MSwgNDUuOTc3MDE3XSwgWzEwLjkzMTE4OCwgNDUuOTc3XSwgWzEwLjkzMTA4LCA0NS45NzcwMDldLCBbMTAuOTMxMDI5LCA0NS45NzcwNTddLCBbMTAuOTMxMDI0LCA0NS45NzcxMzFdLCBbMTAuOTMxMDUxLCA0NS45NzcxNTFdLCBbMTAuOTMxMTAyLCA0NS45NzcyMTddLCBbMTAuOTMxMTU2LCA0NS45NzcyNzhdLCBbMTAuOTMxMTk3LCA0NS45NzczMDVdLCBbMTAuOTMxMjUxLCA0NS45NzczNjZdLCBbMTAuOTMxMjg0LCA0NS45NzczODhdLCBbMTAuOTMxMzIxLCA0NS45Nzc0MjJdLCBbMTAuOTMxMzY1LCA0NS45Nzc0NzJdLCBbMTAuOTMxMzYzLCA0NS45Nzc1MDldLCBbMTAuOTMxMzA0LCA0NS45Nzc2MzZdLCBbMTAuOTMxMjM5LCA0NS45Nzc3NjddLCBbMTAuOTMxMTgsIDQ1Ljk3Nzg5Ml0sIFsxMC45MzExNTksIDQ1Ljk3Nzk2N10sIFsxMC45MzExOTksIDQ1Ljk3Nzk5Ml0sIFsxMC45MzEyMzgsIDQ1Ljk3Nzk4Ml0sIFsxMC45MzEyODMsIDQ1Ljk3Nzk1N10sIFsxMC45MzEzMTksIDQ1Ljk3NzkyOV0sIFsxMC45MzEzNzksIDQ1Ljk3Nzg1N10sIFsxMC45MzE0MTQsIDQ1Ljk3NzgxM10sIFsxMC45MzE0NjUsIDQ1Ljk3Nzc1M10sIFsxMC45MzE1MTIsIDQ1Ljk3NzY4OF0sIFsxMC45MzE1NzIsIDQ1Ljk3NzU5NV0sIFsxMC45MzE2MjYsIDQ1Ljk3NzUxXSwgWzEwLjkzMTY5OSwgNDUuOTc3NDJdLCBbMTAuOTMxNzg0LCA0NS45NzcyOTldLCBbMTAuOTMxODYsIDQ1Ljk3NzE5MV1dXV0sICJ0eXBlIjogIk11bHRpUG9seWdvbiJ9LCAiaWQiOiAiMiIsICJwcm9wZXJ0aWVzIjogeyJkZXNjcml6aW9uZV9vcmlnaW5lIjogIk5vbiBjb2RpZmljYXRvIiwgImdtbF9pZCI6ICJJRC5BQ1FVRUZJU0lDSEUuU1BFQ0NISS5BQ1FVQS41NDE2NCIsICJuYXNwIjogMCwgIm5hdHVyYV9zcGVjY2hpb19hY3F1YSI6ICJOb24gY29kaWZpY2F0byIsICJub21lIjogIkxBR08gU09MTyIsICJvcmlnaW5lIjogMH0sICJ0eXBlIjogIkZlYXR1cmUifSwgeyJnZW9tZXRyeSI6IHsiY29vcmRpbmF0ZXMiOiBbW1tbMTAuODMwNTAyLCA0NS45OTQ1ODVdLCBbMTAuODMwNDgyLCA0NS45OTQ0OTZdLCBbMTAuODMwNDU4LCA0NS45OTQ0NjddLCBbMTAuODMwMTU1LCA0NS45OTQzMDVdLCBbMTAuODMwMDYxLCA0NS45OTQyNTRdLCBbMTAuODI5OTMsIDQ1Ljk5NDE5MV0sIFsxMC44Mjk3MDQsIDQ1Ljk5NDA1OF0sIFsxMC44Mjk1ODMsIDQ1Ljk5Mzk4Ml0sIFsxMC44Mjk0NzgsIDQ1Ljk5MzkxM10sIFsxMC44MjkzODMsIDQ1Ljk5Mzg0OV0sIFsxMC44MjkyNzYsIDQ1Ljk5Mzc5MV0sIFsxMC44MjkxNjIsIDQ1Ljk5Mzc0NV0sIFsxMC44MjkwMzUsIDQ1Ljk5MzcwNF0sIFsxMC44Mjg5NTksIDQ1Ljk5MzY3OF0sIFsxMC44Mjg3NzQsIDQ1Ljk5MzU4OF0sIFsxMC44Mjg3MTEsIDQ1Ljk5MzU3M10sIFsxMC44Mjg2NDQsIDQ1Ljk5MzYzXSwgWzEwLjgyODYyNiwgNDUuOTkzNjY3XSwgWzEwLjgyODYxOCwgNDUuOTkzNzEzXSwgWzEwLjgyODYyMywgNDUuOTkzNzU0XSwgWzEwLjgyODYzOCwgNDUuOTkzODAyXSwgWzEwLjgyODY2NiwgNDUuOTkzODQ1XSwgWzEwLjgyODcwNCwgNDUuOTkzODc2XSwgWzEwLjgyODg3OSwgNDUuOTkzOTcxXSwgWzEwLjgyODk5NiwgNDUuOTk0MDI0XSwgWzEwLjgyOTA4OSwgNDUuOTk0MDU0XSwgWzEwLjgyOTE1MywgNDUuOTk0MDczXSwgWzEwLjgyOTIzMywgNDUuOTk0MTA0XSwgWzEwLjgyOTMzMywgNDUuOTk0MTQ4XSwgWzEwLjgyOTQzMywgNDUuOTk0MThdLCBbMTAuODI5NjE0LCA0NS45OTQyNV0sIFsxMC44Mjk3MjUsIDQ1Ljk5NDMwNV0sIFsxMC44Mjk4OSwgNDUuOTk0Mzk4XSwgWzEwLjgzMDAwNywgNDUuOTk0NDU3XSwgWzEwLjgzMDA5NCwgNDUuOTk0NDk5XSwgWzEwLjgzMDExOSwgNDUuOTk0NTMxXSwgWzEwLjgzMDExMSwgNDUuOTk0NTQ1XSwgWzEwLjgzMDA5NywgNDUuOTk0NTddLCBbMTAuODMwMDU4LCA0NS45OTQ1NjZdLCBbMTAuODMwMDA4LCA0NS45OTQ1NjNdLCBbMTAuODI5OTE2LCA0NS45OTQ1NDldLCBbMTAuODI5ODEzLCA0NS45OTQ1MjFdLCBbMTAuODI5NzU2LCA0NS45OTQ0OTddLCBbMTAuODI5NjcyLCA0NS45OTQ0Nl0sIFsxMC44Mjk1NzgsIDQ1Ljk5NDQxM10sIFsxMC44Mjk0NSwgNDUuOTk0MzU2XSwgWzEwLjgyOTMzNywgNDUuOTk0MzA4XSwgWzEwLjgyOTIzNiwgNDUuOTk0MjYyXSwgWzEwLjgyOTEyNSwgNDUuOTk0MjA3XSwgWzEwLjgyOTAyMSwgNDUuOTk0MTU0XSwgWzEwLjgyODk0NywgNDUuOTk0MTFdLCBbMTAuODI4ODQ2LCA0NS45OTQwNTRdLCBbMTAuODI4Nzc1LCA0NS45OTQwMTddLCBbMTAuODI4NzI4LCA0NS45OTM5OTJdLCBbMTAuODI4Njc1LCA0NS45OTM5ODRdLCBbMTAuODI4NjQ0LCA0NS45OTQwMTNdLCBbMTAuODI4NjI5LCA0NS45OTQwNTRdLCBbMTAuODI4NjE4LCA0NS45OTQxMDddLCBbMTAuODI4NjE3LCA0NS45OTQxNTVdLCBbMTAuODI4NjM4LCA0NS45OTQxOTRdLCBbMTAuODI4Njg2LCA0NS45OTQyMzJdLCBbMTAuODI4NzczLCA0NS45OTQyNzZdLCBbMTAuODI4OTQzLCA0NS45OTQzNDRdLCBbMTAuODI5MDc0LCA0NS45OTQ0MDNdLCBbMTAuODI5MTc1LCA0NS45OTQ0NDldLCBbMTAuODI5MzAzLCA0NS45OTQ1MTddLCBbMTAuODI5NCwgNDUuOTk0NTYxXSwgWzEwLjgyOTczNywgNDUuOTk0Njc0XSwgWzEwLjgyOTg3MywgNDUuOTk0NzE3XSwgWzEwLjgyOTk5MywgNDUuOTk0NzUzXSwgWzEwLjgzMDA3NywgNDUuOTk0NzgyXSwgWzEwLjgzMDE1MywgNDUuOTk0NzkxXSwgWzEwLjgzMDIwNSwgNDUuOTk0NzgxXSwgWzEwLjgzMDIyNywgNDUuOTk0NzU4XSwgWzEwLjgzMDMxOCwgNDUuOTk0NjY0XSwgWzEwLjgzMDM3MSwgNDUuOTk0NjU5XSwgWzEwLjgzMDQzOSwgNDUuOTk0NjVdLCBbMTAuODMwNDk0LCA0NS45OTQ2MjZdLCBbMTAuODMwNTAyLCA0NS45OTQ1ODVdXV1dLCAidHlwZSI6ICJNdWx0aVBvbHlnb24ifSwgImlkIjogIjMiLCAicHJvcGVydGllcyI6IHsiZGVzY3JpemlvbmVfb3JpZ2luZSI6ICJOb24gY29kaWZpY2F0byIsICJnbWxfaWQiOiAiSUQuQUNRVUVGSVNJQ0hFLlNQRUNDSEkuQUNRVUEuNTQxNTkiLCAibmFzcCI6IDAsICJuYXR1cmFfc3BlY2NoaW9fYWNxdWEiOiAiTm9uIGNvZGlmaWNhdG8iLCAibm9tZSI6ICJMQUdPIFRPUkJJRVJBIERJIEZJQVZFXHUwMDI3IDI/IiwgIm9yaWdpbmUiOiAwfSwgInR5cGUiOiAiRmVhdHVyZSJ9LCB7Imdlb21ldHJ5IjogeyJjb29yZGluYXRlcyI6IFtbW1sxMC44MzM0MDcsIDQ1Ljk5MzY1N10sIFsxMC44MzMzNzYsIDQ1Ljk5MzYyMV0sIFsxMC44MzMzNDYsIDQ1Ljk5MzYwMV0sIFsxMC44MzMyNjksIDQ1Ljk5MzU2N10sIFsxMC44MzMyMzIsIDQ1Ljk5MzU0NV0sIFsxMC44MzMyMTUsIDQ1Ljk5MzUxOF0sIFsxMC44MzMyMDEsIDQ1Ljk5MzQ4Ml0sIFsxMC44MzMyMTUsIDQ1Ljk5MzQyNF0sIFsxMC44MzMyMjEsIDQ1Ljk5MzM5Nl0sIFsxMC44MzMyMTcsIDQ1Ljk5MzM2NF0sIFsxMC44MzMxOTMsIDQ1Ljk5MzMzN10sIFsxMC44MzMxMTMsIDQ1Ljk5MzMxNl0sIFsxMC44MzMwNTcsIDQ1Ljk5MzMxNF0sIFsxMC44MzI5OTgsIDQ1Ljk5MzMyXSwgWzEwLjgzMjkzOSwgNDUuOTkzMzMyXSwgWzEwLjgzMjg4NCwgNDUuOTkzMzQ1XSwgWzEwLjgzMjgyOCwgNDUuOTkzMzU1XSwgWzEwLjgzMjc3OSwgNDUuOTkzMzYyXSwgWzEwLjgzMjcwMywgNDUuOTkzMzQ3XSwgWzEwLjgzMjY2OSwgNDUuOTkzMzE2XSwgWzEwLjgzMjYyNSwgNDUuOTkzMjU5XSwgWzEwLjgzMjU4NywgNDUuOTkzMjE2XSwgWzEwLjgzMjUzNywgNDUuOTkzMTgxXSwgWzEwLjgzMjUsIDQ1Ljk5MzE1NF0sIFsxMC44MzI0NjYsIDQ1Ljk5MzEzNl0sIFsxMC44MzI0MjYsIDQ1Ljk5MzEwOV0sIFsxMC44MzIzNzksIDQ1Ljk5MzA4Ml0sIFsxMC44MzIzNDIsIDQ1Ljk5MzA2NV0sIFsxMC44MzIzNTIsIDQ1Ljk5MzA0M10sIFsxMC44MzIyODIsIDQ1Ljk5MzAxXSwgWzEwLjgzMjE4OCwgNDUuOTkyOTU3XSwgWzEwLjgzMjE0NywgNDUuOTkyOTI2XSwgWzEwLjgzMjExMiwgNDUuOTkyODg3XSwgWzEwLjgzMjA4MSwgNDUuOTkyODQyXSwgWzEwLjgzMjA2MiwgNDUuOTkyNzk0XSwgWzEwLjgzMjA0NywgNDUuOTkyNzI4XSwgWzEwLjgzMjAzMiwgNDUuOTkyNjg3XSwgWzEwLjgzMjAxMSwgNDUuOTkyNjYzXSwgWzEwLjgzMTkzLCA0NS45OTI2MjVdLCBbMTAuODMxODY0LCA0NS45OTI2MDZdLCBbMTAuODMxODA0LCA0NS45OTI1ODldLCBbMTAuODMxNzcsIDQ1Ljk5MjU3M10sIFsxMC44MzE2NzYsIDQ1Ljk5MjUxMV0sIFsxMC44MzE0OTEsIDQ1Ljk5MjQ5OV0sIFsxMC44MzE0NiwgNDUuOTkyNTM2XSwgWzEwLjgzMTQyNSwgNDUuOTkyNTgzXSwgWzEwLjgzMTQwNCwgNDUuOTkyNjE3XSwgWzEwLjgzMTM4NiwgNDUuOTkyNjY2XSwgWzEwLjgzMTQwNCwgNDUuOTkyNjk4XSwgWzEwLjgzMTQ0MiwgNDUuOTkyNzM0XSwgWzEwLjgzMTQ5MiwgNDUuOTkyNzY1XSwgWzEwLjgzMTU4LCA0NS45OTI4MTFdLCBbMTAuODMxNjc3LCA0NS45OTI4NjJdLCBbMTAuODMxNzg4LCA0NS45OTI5MjRdLCBbMTAuODMxODgyLCA0NS45OTI5NjhdLCBbMTAuODMyMDY4LCA0NS45OTMwNzZdLCBbMTAuODMyMjY0LCA0NS45OTMyMjRdLCBbMTAuODMyMzYxLCA0NS45OTMyNjNdLCBbMTAuODMyMzczLCA0NS45OTMzMDJdLCBbMTAuODMyMzY1LCA0NS45OTMzNTFdLCBbMTAuODMyMzYzLCA0NS45OTMzOTldLCBbMTAuODMyNDE0LCA0NS45OTM0MzJdLCBbMTAuODMyNDU3LCA0NS45OTM0NDhdLCBbMTAuODMyNDc3LCA0NS45OTM0NzVdLCBbMTAuODMyNDg2LCA0NS45OTM0OV0sIFsxMC44MzI0ODYsIDQ1Ljk5MzUyMl0sIFsxMC44MzI0ODMsIDQ1Ljk5MzU1M10sIFsxMC44MzI0NjgsIDQ1Ljk5MzU5Ml0sIFsxMC44MzI0NjYsIDQ1Ljk5MzYzM10sIFsxMC44MzI0OTEsIDQ1Ljk5MzY3Nl0sIFsxMC44MzI1MTgsIDQ1Ljk5MzcwNl0sIFsxMC44MzI1NjgsIDQ1Ljk5MzczOV0sIFsxMC44MzI2MTgsIDQ1Ljk5Mzc2MV0sIFsxMC44MzI2NTgsIDQ1Ljk5Mzc3NF0sIFsxMC44MzI3MTQsIDQ1Ljk5Mzc3OF0sIFsxMC44MzI3NTMsIDQ1Ljk5Mzc2OF0sIFsxMC44MzI3ODksIDQ1Ljk5Mzc0NV0sIFsxMC44MzI4MTcsIDQ1Ljk5MzcxN10sIFsxMC44MzI4NDIsIDQ1Ljk5MzY3MV0sIFsxMC44MzI4OCwgNDUuOTkzNjI5XSwgWzEwLjgzMjkxOSwgNDUuOTkzNjA1XSwgWzEwLjgzMjk1OCwgNDUuOTkzNjAyXSwgWzEwLjgzMzAyNSwgNDUuOTkzNjI5XSwgWzEwLjgzMzA2NiwgNDUuOTkzNjY1XSwgWzEwLjgzMzA4OSwgNDUuOTkzNjg3XSwgWzEwLjgzMzEyNywgNDUuOTkzNzIzXSwgWzEwLjgzMzE3LCA0NS45OTM3NV0sIFsxMC44MzMyMzQsIDQ1Ljk5Mzc3N10sIFsxMC44MzMyOTMsIDQ1Ljk5Mzc3OF0sIFsxMC44MzMzMzIsIDQ1Ljk5Mzc3M10sIFsxMC44MzMzNywgNDUuOTkzNzQyXSwgWzEwLjgzMzQwNSwgNDUuOTkzNjg5XSwgWzEwLjgzMzQwNywgNDUuOTkzNjU3XV1dXSwgInR5cGUiOiAiTXVsdGlQb2x5Z29uIn0sICJpZCI6ICI0IiwgInByb3BlcnRpZXMiOiB7ImRlc2NyaXppb25lX29yaWdpbmUiOiAiTm9uIGNvZGlmaWNhdG8iLCAiZ21sX2lkIjogIklELkFDUVVFRklTSUNIRS5TUEVDQ0hJLkFDUVVBLjU0MTYwIiwgIm5hc3AiOiAwLCAibmF0dXJhX3NwZWNjaGlvX2FjcXVhIjogIk5vbiBjb2RpZmljYXRvIiwgIm5vbWUiOiAiTEFHTyBUT1JCSUVSQSBESSBGSUFWRVx1MDAyNyAzPyIsICJvcmlnaW5lIjogMH0sICJ0eXBlIjogIkZlYXR1cmUifSwgeyJnZW9tZXRyeSI6IHsiY29vcmRpbmF0ZXMiOiBbW1tbMTAuOTU2ODM0LCA0Ni4wMTA5N10sIFsxMC45NTY4MTksIDQ2LjAxMDgyNl0sIFsxMC45NTY4MTQsIDQ2LjAxMDcwNV0sIFsxMC45NTY3OTgsIDQ2LjAxMDU0MV0sIFsxMC45NTY3NzgsIDQ2LjAxMDM0Ml0sIFsxMC45NTY3MzksIDQ2LjAxMDEyMV0sIFsxMC45NTY2OTksIDQ2LjAwOTg3OF0sIFsxMC45NTY2NjUsIDQ2LjAwOTYzOV0sIFsxMC45NTY2MjIsIDQ2LjAwOTM4N10sIFsxMC45NTY1NjgsIDQ2LjAwOTEwN10sIFsxMC45NTY1MTUsIDQ2LjAwODg0Ml0sIFsxMC45NTY1MDUsIDQ2LjAwODYxMV0sIFsxMC45NTY0ODQsIDQ2LjAwODM3MV0sIFsxMC45NTY0NjgsIDQ2LjAwODE0N10sIFsxMC45NTY0NywgNDYuMDA4MDc0XSwgWzEwLjk1NjQ3LCA0Ni4wMDc4OF0sIFsxMC45NTY0OTQsIDQ2LjAwNzc2N10sIFsxMC45NTY1MjgsIDQ2LjAwNzU5OF0sIFsxMC45NTY1NTUsIDQ2LjAwNzQzNV0sIFsxMC45NTY1MzgsIDQ2LjAwNzIxMV0sIFsxMC45NTY0OTUsIDQ2LjAwNjk5Ml0sIFsxMC45NTY0NjIsIDQ2LjAwNjc0M10sIFsxMC45NTY0MjMsIDQ2LjAwNjU3N10sIFsxMC45NTYzNjEsIDQ2LjAwNjQxNV0sIFsxMC45NTYyNDEsIDQ2LjAwNjI2M10sIFsxMC45NTYwOSwgNDYuMDA2MDk3XSwgWzEwLjk1NjAxMiwgNDYuMDA1OTU2XSwgWzEwLjk1NTk1NywgNDYuMDA1Nzk4XSwgWzEwLjk1NTkxMiwgNDYuMDA1NjIzXSwgWzEwLjk1NTg5NCwgNDYuMDA1NDk3XSwgWzEwLjk1NTg3OCwgNDYuMDA1MzY3XSwgWzEwLjk1NTg4MywgNDYuMDA1MjM5XSwgWzEwLjk1NTg4NywgNDYuMDA1MDk3XSwgWzEwLjk1NTg3OCwgNDYuMDA0OTYyXSwgWzEwLjk1NTg2NCwgNDYuMDA0ODg0XSwgWzEwLjk1NTg1MywgNDYuMDA0Nzc3XSwgWzEwLjk1NTg0NCwgNDYuMDA0NjY1XSwgWzEwLjk1NTgzNiwgNDYuMDA0NTM1XSwgWzEwLjk1NTgwMSwgNDYuMDA0Mzk2XSwgWzEwLjk1NTcyNSwgNDYuMDA0MjA2XSwgWzEwLjk1NTYzNCwgNDYuMDA0MDI0XSwgWzEwLjk1NTUwNSwgNDYuMDAzNzk2XSwgWzEwLjk1NTM2MSwgNDYuMDAzNTg1XSwgWzEwLjk1NTE4NywgNDYuMDAzMzhdLCBbMTAuOTU1MDc2LCA0Ni4wMDMyMzVdLCBbMTAuOTU1MDI1LCA0Ni4wMDMxMTJdLCBbMTAuOTU0OTQ2LCA0Ni4wMDI5MzJdLCBbMTAuOTU0ODc0LCA0Ni4wMDI3MzNdLCBbMTAuOTU0NzkxLCA0Ni4wMDI0M10sIFsxMC45NTQ3MzIsIDQ2LjAwMjI4Ml0sIFsxMC45NTQ3MTEsIDQ2LjAwMjE2OF0sIFsxMC45NTQ2NSwgNDYuMDAyMDIyXSwgWzEwLjk1NDU3MiwgNDYuMDAxOTA2XSwgWzEwLjk1NDQ3NSwgNDYuMDAxODA0XSwgWzEwLjk1NDMxNiwgNDYuMDAxNjkzXSwgWzEwLjk1NDExNywgNDYuMDAxNThdLCBbMTAuOTUzOTE0LCA0Ni4wMDE0MzddLCBbMTAuOTUzNzY4LCA0Ni4wMDEzMl0sIFsxMC45NTM1NjUsIDQ2LjAwMTE0MV0sIFsxMC45NTMzODIsIDQ2LjAwMDk4Ml0sIFsxMC45NTMxODgsIDQ2LjAwMDc5NF0sIFsxMC45NTMwNDgsIDQ2LjAwMDY1MV0sIFsxMC45NTI5MDEsIDQ2LjAwMDQ5OV0sIFsxMC45NTI4MjQsIDQ2LjAwMDM5NF0sIFsxMC45NTI4MTIsIDQ2LjAwMDI2Nl0sIFsxMC45NTI4NjMsIDQ2LjAwMDE0XSwgWzEwLjk1Mjg2NywgNDYuMDAwMDI2XSwgWzEwLjk1Mjg1NSwgNDUuOTk5ODg4XSwgWzEwLjk1Mjg0LCA0NS45OTk3NjFdLCBbMTAuOTUyODA5LCA0NS45OTk2NDZdLCBbMTAuOTUyNzc1LCA0NS45OTk1NzRdLCBbMTAuOTUyNzU3LCA0NS45OTk0OF0sIFsxMC45NTI3NTksIDQ1Ljk5OTQwMl0sIFsxMC45NTI3NzgsIDQ1Ljk5OTM0XSwgWzEwLjk1Mjc3MywgNDUuOTk5MjRdLCBbMTAuOTUyNzcyLCA0NS45OTkxNjRdLCBbMTAuOTUyNzc5LCA0NS45OTkxMDldLCBbMTAuOTUyNzkzLCA0NS45OTkxMDldLCBbMTAuOTUyODA0LCA0NS45OTkwNTRdLCBbMTAuOTUyODQ3LCA0NS45OTg5ODJdLCBbMTAuOTUyODk3LCA0NS45OTg5MV0sIFsxMC45NTI5NjEsIDQ1Ljk5ODc5OV0sIFsxMC45NTI5NjUsIDQ1Ljk5ODcyNl0sIFsxMC45NTI5NTUsIDQ1Ljk5ODYyOF0sIFsxMC45NTI5ODgsIDQ1Ljk5ODU2M10sIFsxMC45NTMwNiwgNDUuOTk4NDY2XSwgWzEwLjk1MzA4NiwgNDUuOTk4MzgzXSwgWzEwLjk1MzA2NCwgNDUuOTk4MzAxXSwgWzEwLjk1MzA0NSwgNDUuOTk4MjM3XSwgWzEwLjk1MzA0OCwgNDUuOTk4MTY2XSwgWzEwLjk1MzA4MiwgNDUuOTk4MTA5XSwgWzEwLjk1MzEyOCwgNDUuOTk4MDIzXSwgWzEwLjk1MzEyOSwgNDUuOTk3OTQ4XSwgWzEwLjk1MzEzNiwgNDUuOTk3ODc5XSwgWzEwLjk1MzE5OCwgNDUuOTk3Nzk1XSwgWzEwLjk1MzI4NywgNDUuOTk3NzI1XSwgWzEwLjk1MzM0NSwgNDUuOTk3Njc2XSwgWzEwLjk1MzQzNCwgNDUuOTk3NTk5XSwgWzEwLjk1MzQ4NywgNDUuOTk3NTM4XSwgWzEwLjk1MzQ5MywgNDUuOTk3NDM4XSwgWzEwLjk1MzQ1NywgNDUuOTk3MzQ3XSwgWzEwLjk1MzQyNywgNDUuOTk3MjddLCBbMTAuOTUzMzc3LCA0NS45OTcxNTldLCBbMTAuOTUzMzA5LCA0NS45OTcwMDddLCBbMTAuOTUzMjU2LCA0NS45OTY5MDVdLCBbMTAuOTUzMTQzLCA0NS45OTY3MV0sIFsxMC45NTMwMzUsIDQ1Ljk5NjU1XSwgWzEwLjk1MjkxMywgNDUuOTk2Mzc2XSwgWzEwLjk1Mjc4MSwgNDUuOTk2MTc3XSwgWzEwLjk1MjY1OSwgNDUuOTk2MDA0XSwgWzEwLjk1MjUzNywgNDUuOTk1ODIxXSwgWzEwLjk1MjQxNiwgNDUuOTk1NjYzXSwgWzEwLjk1MjI5MiwgNDUuOTk1NTA1XSwgWzEwLjk1MjE0LCA0NS45OTUzMThdLCBbMTAuOTUxOTg2LCA0NS45OTUxNTZdLCBbMTAuOTUxODQ0LCA0NS45OTUwMzZdLCBbMTAuOTUxNzMyLCA0NS45OTQ5MzJdLCBbMTAuOTUxNjkyLCA0NS45OTQ4NDJdLCBbMTAuOTUxNjkzLCA0NS45OTQ3OTFdLCBbMTAuOTUxNzM2LCA0NS45OTQ3MDRdLCBbMTAuOTUxNjA2LCA0NS45OTQ2NDRdLCBbMTAuOTUxNTM2LCA0NS45OTQ2MzJdLCBbMTAuOTUxNDExLCA0NS45OTQ2MDldLCBbMTAuOTUxMzI4LCA0NS45OTQ1MjJdLCBbMTAuOTUxMjMzLCA0NS45OTQ0M10sIFsxMC45NTExNjUsIDQ1Ljk5NDM3NF0sIFsxMC45NTEwODEsIDQ1Ljk5NDMxMV0sIFsxMC45NTA5NTMsIDQ1Ljk5NDIzNl0sIFsxMC45NTA4NzIsIDQ1Ljk5NDJdLCBbMTAuOTUwODM5LCA0NS45OTQxODVdLCBbMTAuOTUwNzA2LCA0NS45OTQxMzFdLCBbMTAuOTUwNTQsIDQ1Ljk5NDA3Ml0sIFsxMC45NTAzNTMsIDQ1Ljk5NDAwNV0sIFsxMC45NDk5OTEsIDQ1Ljk5Mzg4M10sIFsxMC45NDk4MjgsIDQ1Ljk5MzgzNl0sIFsxMC45NDk2OTUsIDQ1Ljk5Mzc5NV0sIFsxMC45NDkzMzUsIDQ1Ljk5MzY0Ml0sIFsxMC45NDkyNCwgNDUuOTkzNThdLCBbMTAuOTQ5MTk3LCA0NS45OTM1NTVdLCBbMTAuOTQ5MTIsIDQ1Ljk5MzUyOV0sIFsxMC45NDg4ODUsIDQ1Ljk5MzQ2MV0sIFsxMC45NDg3MjcsIDQ1Ljk5MzQ1Ml0sIFsxMC45NDg1NzYsIDQ1Ljk5MzQ3MV0sIFsxMC45NDg0NDMsIDQ1Ljk5MzQ5NF0sIFsxMC45NDgyMDUsIDQ1Ljk5MzUzM10sIFsxMC45NDgwNywgNDUuOTkzNTIyXSwgWzEwLjk0Nzg5NCwgNDUuOTkzNDgxXSwgWzEwLjk0NzcwMSwgNDUuOTkzNDA3XSwgWzEwLjk0NzUzNSwgNDUuOTkzMzUxXSwgWzEwLjk0NzM1OSwgNDUuOTkzMzExXSwgWzEwLjk0NzE5LCA0NS45OTMyN10sIFsxMC45NDY5OTksIDQ1Ljk5MzI0OV0sIFsxMC45NDY3ODIsIDQ1Ljk5MzI1MV0sIFsxMC45NDY1ODksIDQ1Ljk5MzI1OV0sIFsxMC45NDY0NTEsIDQ1Ljk5MzI2NF0sIFsxMC45NDYzNDYsIDQ1Ljk5MzI1XSwgWzEwLjk0NjIxNCwgNDUuOTkzMjQ4XSwgWzEwLjk0NjA2LCA0NS45OTMyNTNdLCBbMTAuOTQ1OTAzLCA0NS45OTMyNThdLCBbMTAuOTQ1NzM1LCA0NS45OTMyNTJdLCBbMTAuOTQ1NTI1LCA0NS45OTMyNF0sIFsxMC45NDUzODYsIDQ1Ljk5MzIyNF0sIFsxMC45NDUxOTQsIDQ1Ljk5MzE3MV0sIFsxMC45NDQ4NDYsIDQ1Ljk5MzA3Ml0sIFsxMC45NDQ2MzYsIDQ1Ljk5Mjk5Nl0sIFsxMC45NDQyMTMsIDQ1Ljk5Mjg0OF0sIFsxMC45NDQwMTcsIDQ1Ljk5Mjc3Ml0sIFsxMC45NDM3ODYsIDQ1Ljk5MjY3Nl0sIFsxMC45NDM2NjYsIDQ1Ljk5MjYyM10sIFsxMC45NDM1MTUsIDQ1Ljk5MjUyN10sIFsxMC45NDM0MDMsIDQ1Ljk5MjQ0N10sIFsxMC45NDMyODQsIDQ1Ljk5MjMzNV0sIFsxMC45NDMxNTMsIDQ1Ljk5MjE2Nl0sIFsxMC45NDMwNiwgNDUuOTkyMDUxXSwgWzEwLjk0Mjk1OSwgNDUuOTkxODk4XSwgWzEwLjk0Mjg2NSwgNDUuOTkxNzcxXSwgWzEwLjk0Mjc3OSwgNDUuOTkxNjY4XSwgWzEwLjk0MjY4MiwgNDUuOTkxNTM5XSwgWzEwLjk0MjYyMiwgNDUuOTkxNDE3XSwgWzEwLjk0MjU2NCwgNDUuOTkxMjc0XSwgWzEwLjk0MjUyNywgNDUuOTkxMTcyXSwgWzEwLjk0MjUwNywgNDUuOTkxMDc2XSwgWzEwLjk0MjQ5MSwgNDUuOTkwOTkyXSwgWzEwLjk0MjQ2MiwgNDUuOTkwOTI4XSwgWzEwLjk0MjQxLCA0NS45OTA4N10sIFsxMC45NDIzNDQsIDQ1Ljk5MDg0MV0sIFsxMC45NDIyNzQsIDQ1Ljk5MDgzMV0sIFsxMC45NDIyMDUsIDQ1Ljk5MDgyNV0sIFsxMC45NDIxNDIsIDQ1Ljk5MDc5N10sIFsxMC45NDIwODEsIDQ1Ljk5MDc0NV0sIFsxMC45NDIwNCwgNDUuOTkwNzA1XSwgWzEwLjk0MTk4NiwgNDUuOTkwNjc0XSwgWzEwLjk0MTkyNiwgNDUuOTkwNjU3XSwgWzEwLjk0MTg1NywgNDUuOTkwNjQ5XSwgWzEwLjk0MTc4OSwgNDUuOTkwNjU5XSwgWzEwLjk0MTcxLCA0NS45OTA2NzVdLCBbMTAuOTQxNjU1LCA0NS45OTA2ODldLCBbMTAuOTQxNTQ1LCA0NS45OTA3MTRdLCBbMTAuOTQxNDE3LCA0NS45OTA3MzNdLCBbMTAuOTQxMjk5LCA0NS45OTA3NDRdLCBbMTAuOTQxMTY1LCA0NS45OTA3NTNdLCBbMTAuOTQxMDA4LCA0NS45OTA3N10sIFsxMC45NDA4NTUsIDQ1Ljk5MDc4OV0sIFsxMC45NDA2NzIsIDQ1Ljk5MDgxNV0sIFsxMC45NDA1MTIsIDQ1Ljk5MDgzNl0sIFsxMC45NDAzNzgsIDQ1Ljk5MDg1N10sIFsxMC45NDAyNDEsIDQ1Ljk5MDg4XSwgWzEwLjk0MDExNCwgNDUuOTkwOTA4XSwgWzEwLjk0MDAzLCA0NS45OTA5MjZdLCBbMTAuOTQwMDIyLCA0NS45OTA5MjddLCBbMTAuOTM5OTU4LCA0NS45OTA5NDFdLCBbMTAuOTM5OTE5LCA0NS45OTA5Nl0sIFsxMC45Mzk4ODcsIDQ1Ljk5MDk4MV0sIFsxMC45Mzk4ODIsIDQ1Ljk5MTAxN10sIFsxMC45Mzk5MTYsIDQ1Ljk5MTA0OV0sIFsxMC45Mzk5NTMsIDQ1Ljk5MTA2OV0sIFsxMC45NCwgNDUuOTkxMDg4XSwgWzEwLjk0MDA0NiwgNDUuOTkxMDk3XSwgWzEwLjk0MDE0NSwgNDUuOTkxMTE1XSwgWzEwLjk0MDE4NiwgNDUuOTkxMTQ5XSwgWzEwLjk0MDE1NSwgNDUuOTkxMTg4XSwgWzEwLjk0MDEwOSwgNDUuOTkxMjA1XSwgWzEwLjk0MDAzMSwgNDUuOTkxMjExXSwgWzEwLjkzOTk0NiwgNDUuOTkxMjJdLCBbMTAuOTM5ODksIDQ1Ljk5MTIyM10sIFsxMC45Mzk4MzgsIDQ1Ljk5MTIyMl0sIFsxMC45Mzk3ODgsIDQ1Ljk5MTIxNl0sIFsxMC45Mzk3NDIsIDQ1Ljk5MTIwOF0sIFsxMC45Mzk2NSwgNDUuOTkxMjA1XSwgWzEwLjkzOTU3MiwgNDUuOTkxMjE1XSwgWzEwLjkzOTU0LCA0NS45OTEyNDFdLCBbMTAuOTM5NTE2LCA0NS45OTEyOTZdLCBbMTAuOTM5NTEzLCA0NS45OTEzMDNdLCBbMTAuOTM5NTYxLCA0NS45OTEzNjddLCBbMTAuOTM5NjA4LCA0NS45OTEzOTNdLCBbMTAuOTM5NjUxLCA0NS45OTE0MDRdLCBbMTAuOTM5NzUzLCA0NS45OTE0MTZdLCBbMTAuOTM5ODA2LCA0NS45OTE0MTldLCBbMTAuOTM5ODcxLCA0NS45OTE0MTFdLCBbMTAuOTM5OTc2LCA0NS45OTEzOTVdLCBbMTAuOTQwMDk2LCA0NS45OTEzNzddLCBbMTAuOTQwMTk0LCA0NS45OTEzNTJdLCBbMTAuOTQwMjM5LCA0NS45OTEzNDJdLCBbMTAuOTQwMzE4LCA0NS45OTEzMjVdLCBbMTAuOTQwNDIyLCA0NS45OTEzXSwgWzEwLjk0MDUxMywgNDUuOTkxMjhdLCBbMTAuOTQwNjYsIDQ1Ljk5MTI2OF0sIFsxMC45NDA3OTgsIDQ1Ljk5MTI3M10sIFsxMC45NDA4OSwgNDUuOTkxMjg0XSwgWzEwLjk0MSwgNDUuOTkxMzNdLCBbMTAuOTQxMTI0LCA0NS45OTEzODNdLCBbMTAuOTQxMjU0LCA0NS45OTE0MjldLCBbMTAuOTQxMzkzLCA0NS45OTE0NjVdLCBbMTAuOTQxNTE5LCA0NS45OTE0OV0sIFsxMC45NDE2NDEsIDQ1Ljk5MTUwNF0sIFsxMC45NDE3ODYsIDQ1Ljk5MTUyNF0sIFsxMC45NDE5MjUsIDQ1Ljk5MTU1Nl0sIFsxMC45NDIwNzEsIDQ1Ljk5MTYwNV0sIFsxMC45NDIyMTEsIDQ1Ljk5MTY2XSwgWzEwLjk0MjMxOSwgNDUuOTkxNzMxXSwgWzEwLjk0MjM5NCwgNDUuOTkxNzk0XSwgWzEwLjk0MjQ0MSwgNDUuOTkxODQzXSwgWzEwLjk0MjUzMiwgNDUuOTkxOTY3XSwgWzEwLjk0MjU5NCwgNDUuOTkyMDZdLCBbMTAuOTQyNjY0LCA0NS45OTIxOF0sIFsxMC45NDI3MzEsIDQ1Ljk5MjI4Nl0sIFsxMC45NDI3ODEsIDQ1Ljk5MjM4OF0sIFsxMC45NDI4MTEsIDQ1Ljk5MjUwMl0sIFsxMC45NDI4MjksIDQ1Ljk5MjYxOF0sIFsxMC45NDI4MywgNDUuOTkyNzI2XSwgWzEwLjk0MjgxNywgNDUuOTkyODI5XSwgWzEwLjk0Mjc5OCwgNDUuOTkyOTI1XSwgWzEwLjk0Mjc3OCwgNDUuOTkzMDA1XSwgWzEwLjk0Mjc2MywgNDUuOTkzMTA2XSwgWzEwLjk0Mjc2MiwgNDUuOTkzMTgyXSwgWzEwLjk0Mjc4OCwgNDUuOTkzMjU1XSwgWzEwLjk0MjkwNywgNDUuOTkzMzUxXSwgWzEwLjk0Mjk1NCwgNDUuOTkzMzg5XSwgWzEwLjk0MzAxMiwgNDUuOTkzNDQyXSwgWzEwLjk0MzA2LCA0NS45OTM0ODddLCBbMTAuOTQzMTEyLCA0NS45OTM1NjZdLCBbMTAuOTQzMTQsIDQ1Ljk5MzYxOF0sIFsxMC45NDMxODMsIDQ1Ljk5MzcwN10sIFsxMC45NDMxOTksIDQ1Ljk5Mzc5MV0sIFsxMC45NDMxOTksIDQ1Ljk5Mzg2Ml0sIFsxMC45NDMxOTMsIDQ1Ljk5Mzk2NV0sIFsxMC45NDMyLCA0NS45OTQwNzddLCBbMTAuOTQzMjAxLCA0NS45OTQxODJdLCBbMTAuOTQzMjE4LCA0NS45OTQyNzNdLCBbMTAuOTQzMjUyLCA0NS45OTQzODVdLCBbMTAuOTQzMjk4LCA0NS45OTQ0NzNdLCBbMTAuOTQzMzU3LCA0NS45OTQ1NTddLCBbMTAuOTQzNDQ5LCA0NS45OTQ2NThdLCBbMTAuOTQzNTUxLCA0NS45OTQ3NV0sIFsxMC45NDM3LCA0NS45OTQ4N10sIFsxMC45NDM4NTYsIDQ1Ljk5NDk4Nl0sIFsxMC45NDM5OTYsIDQ1Ljk5NTEyOF0sIFsxMC45NDQxMiwgNDUuOTk1Mjc2XSwgWzEwLjk0NDI0MywgNDUuOTk1NDA5XSwgWzEwLjk0NDM1LCA0NS45OTU1NDldLCBbMTAuOTQ0NDMzLCA0NS45OTU2NjJdLCBbMTAuOTQ0NTA3LCA0NS45OTU3N10sIFsxMC45NDQ1NzEsIDQ1Ljk5NTkwMV0sIFsxMC45NDQ2MiwgNDUuOTk1OTldLCBbMTAuOTQ0NzA2LCA0NS45OTYwOTVdLCBbMTAuOTQ0NzYxLCA0NS45OTYxNTJdLCBbMTAuOTQ0NzksIDQ1Ljk5NjIxM10sIFsxMC45NDQ4MjcsIDQ1Ljk5NjI2XSwgWzEwLjk0NDg2NCwgNDUuOTk2MjhdLCBbMTAuOTQ1MDM0LCA0NS45OTYzNDVdLCBbMTAuOTQ1MDYyLCA0NS45OTYzOTFdLCBbMTAuOTQ1MDU3LCA0NS45OTY0MzJdLCBbMTAuOTQ1MDM2LCA0NS45OTY0OF0sIFsxMC45NDUwMTMsIDQ1Ljk5NjU0OV0sIFsxMC45NDUwNDUsIDQ1Ljk5NjYyNF0sIFsxMC45NDUxMTMsIDQ1Ljk5NjY3M10sIFsxMC45NDUxNTcsIDQ1Ljk5NjcwN10sIFsxMC45NDUxOTgsIDQ1Ljk5Njc0N10sIFsxMC45NDUyMzMsIDQ1Ljk5NjgxM10sIFsxMC45NDUyNzUsIDQ1Ljk5Njg3Nl0sIFsxMC45NDU0MDMsIDQ1Ljk5Njk1Nl0sIFsxMC45NDU1MjksIDQ1Ljk5Njk5M10sIFsxMC45NDU1ODYsIDQ1Ljk5NzAzN10sIFsxMC45NDU2NDMsIDQ1Ljk5NzA3NV0sIFsxMC45NDU3MiwgNDUuOTk3MTAxXSwgWzEwLjk0NTgzLCA0NS45OTcxMzhdLCBbMTAuOTQ1ODY0LCA0NS45OTcxODFdLCBbMTAuOTQ1ODc4LCA0NS45OTcyMV0sIFsxMC45NDU4ODIsIDQ1Ljk5NzMwNF0sIFsxMC45NDU4NzgsIDQ1Ljk5NzM2Nl0sIFsxMC45NDU4NTksIDQ1Ljk5NzQ2OV0sIFsxMC45NDU4NDQsIDQ1Ljk5NzUwNl0sIFsxMC45NDU4MSwgNDUuOTk3NTU3XSwgWzEwLjk0NTgxMywgNDUuOTk3NjMyXSwgWzEwLjk0NTg1NywgNDUuOTk3NjgyXSwgWzEwLjk0NTg1OCwgNDUuOTk3NzA5XSwgWzEwLjk0NTgzOCwgNDUuOTk3NzY5XSwgWzEwLjk0NTgyMywgNDUuOTk3ODEzXSwgWzEwLjk0NTg0OCwgNDUuOTk3ODYyXSwgWzEwLjk0NTg5NiwgNDUuOTk3OTI4XSwgWzEwLjk0NTk4OSwgNDUuOTk4MDQzXSwgWzEwLjk0NjA4OCwgNDUuOTk4MTNdLCBbMTAuOTQ2MjEsIDQ1Ljk5ODIyNF0sIFsxMC45NDYzMzQsIDQ1Ljk5ODMwNF0sIFsxMC45NDY0NzgsIDQ1Ljk5ODM3OV0sIFsxMC45NDY2MTIsIDQ1Ljk5ODQ0M10sIFsxMC45NDY3NTYsIDQ1Ljk5ODUyXSwgWzEwLjk0Njg3LCA0NS45OTg2NTNdLCBbMTAuOTQ2ODc5LCA0NS45OTg3MjFdLCBbMTAuOTQ2ODc3LCA0NS45OTg3NjddLCBbMTAuOTQ2OTEyLCA0NS45OTg4MTJdLCBbMTAuOTQ2OTU5LCA0NS45OTg4NDFdLCBbMTAuOTQ2OTgzLCA0NS45OTg4NjZdLCBbMTAuOTQ3MDEyLCA0NS45OTg5MjddLCBbMTAuOTQ3MDEsIDQ1Ljk5ODk2M10sIFsxMC45NDY5OTQsIDQ1Ljk5OTA2OV0sIFsxMC45NDY5ODQsIDQ1Ljk5OTEwOF0sIFsxMC45NDY5OTksIDQ1Ljk5OTEwOF0sIFsxMC45NDY5OTQsIDQ1Ljk5OTEzNV0sIFsxMC45NDY5NzIsIDQ1Ljk5OTE5Ml0sIFsxMC45NDY5NTYsIDQ1Ljk5OTI0Ml0sIFsxMC45NDY5MzUsIDQ1Ljk5OTMyNV0sIFsxMC45NDY4OTcsIDQ1Ljk5OTQ0XSwgWzEwLjk0Njg3NiwgNDUuOTk5NTI5XSwgWzEwLjk0Njg3NCwgNDUuOTk5NjI3XSwgWzEwLjk0Njg5NSwgNDUuOTk5NzA3XSwgWzEwLjk0NjkzOSwgNDUuOTk5ODAzXSwgWzEwLjk0Njk5NywgNDUuOTk5OTQ5XSwgWzEwLjk0NzAzNSwgNDYuMDAwMDU2XSwgWzEwLjk0NzA2OSwgNDYuMDAwMTU4XSwgWzEwLjk0NzE0LCA0Ni4wMDAyNjFdLCBbMTAuOTQ3MjMsIDQ2LjAwMDM1OF0sIFsxMC45NDczMjYsIDQ2LjAwMDQyNl0sIFsxMC45NDc0NywgNDYuMDAwNTg1XSwgWzEwLjk0NzU0LCA0Ni4wMDA2NjldLCBbMTAuOTQ3NTg4LCA0Ni4wMDA3OV0sIFsxMC45NDc2MiwgNDYuMDAwOTE4XSwgWzEwLjk0NzY1NywgNDYuMDAxMDA3XSwgWzEwLjk0NzcyNSwgNDYuMDAxMTA1XSwgWzEwLjk0NzgwNSwgNDYuMDAxMl0sIFsxMC45NDc4OTUsIDQ2LjAwMTMxNF0sIFsxMC45NDc5NjIsIDQ2LjAwMTM5MV0sIFsxMC45NDgwNzMsIDQ2LjAwMTUxNl0sIFsxMC45NDgxNDMsIDQ2LjAwMTYxNl0sIFsxMC45NDgyMjEsIDQ2LjAwMTc1NV0sIFsxMC45NDgzLCA0Ni4wMDE5NTVdLCBbMTAuOTQ4Mzc5LCA0Ni4wMDIxODRdLCBbMTAuOTQ4NDQyLCA0Ni4wMDI0MjFdLCBbMTAuOTQ4NDk3LCA0Ni4wMDI2MDNdLCBbMTAuOTQ4NTk4LCA0Ni4wMDI3NzldLCBbMTAuOTQ4NzI5LCA0Ni4wMDI5MzVdLCBbMTAuOTQ4ODM2LCA0Ni4wMDMwNzRdLCBbMTAuOTQ4OTE0LCA0Ni4wMDMyMTFdLCBbMTAuOTQ5MDY4LCA0Ni4wMDM0MTVdLCBbMTAuOTQ5Mjc1LCA0Ni4wMDM2NDldLCBbMTAuOTQ5NDE2LCA0Ni4wMDM4NDNdLCBbMTAuOTQ5NTI3LCA0Ni4wMDM5OTldLCBbMTAuOTQ5NjcxLCA0Ni4wMDQxNzldLCBbMTAuOTQ5ODA4LCA0Ni4wMDQzMzZdLCBbMTAuOTQ5ODg2LCA0Ni4wMDQ0NTJdLCBbMTAuOTQ5OTI3LCA0Ni4wMDQ1NjZdLCBbMTAuOTQ5OTU4LCA0Ni4wMDQ2NzhdLCBbMTAuOTUwMDA3LCA0Ni4wMDQ4MTRdLCBbMTAuOTUwMDM5LCA0Ni4wMDQ5N10sIFsxMC45NTAwNCwgNDYuMDA1MDkxXSwgWzEwLjk1MDA2NSwgNDYuMDA1Ml0sIFsxMC45NTAxMDksIDQ2LjAwNTMwM10sIFsxMC45NTAyMDMsIDQ2LjAwNTQzXSwgWzEwLjk1MDMxNywgNDYuMDA1NTc2XSwgWzEwLjk1MDQzOCwgNDYuMDA1NzA1XSwgWzEwLjk1MDU5MSwgNDYuMDA1ODc1XSwgWzEwLjk1MDcyMiwgNDYuMDA2MDM1XSwgWzEwLjk1MDksIDQ2LjAwNjI1N10sIFsxMC45NTEwNDcsIDQ2LjAwNjQyOF0sIFsxMC45NTEyMTEsIDQ2LjAwNjU5OF0sIFsxMC45NTEzODQsIDQ2LjAwNjc1OV0sIFsxMC45NTE1NDcsIDQ2LjAwNjkxMV0sIFsxMC45NTE3MDcsIDQ2LjAwNzA1Nl0sIFsxMC45NTE4NzcsIDQ2LjAwNzIyNF0sIFsxMC45NTIwNDcsIDQ2LjAwNzM2NF0sIFsxMC45NTIyMSwgNDYuMDA3NTAzXSwgWzEwLjk1MjI4OCwgNDYuMDA3NjI4XSwgWzEwLjk1MjMwMywgNDYuMDA3NzRdLCBbMTAuOTUyMzQ3LCA0Ni4wMDc4NDVdLCBbMTAuOTUyNDg4LCA0Ni4wMDgwMTVdLCBbMTAuOTUyNTcyLCA0Ni4wMDgxMzNdLCBbMTAuOTUyNjA4LCA0Ni4wMDgxNjVdLCBbMTAuOTUyNjYyLCA0Ni4wMDgyMzFdLCBbMTAuOTUyNjYyLCA0Ni4wMDgyNThdLCBbMTAuOTUyNzEyLCA0Ni4wMDgzMDhdLCBbMTAuOTUyODA2LCA0Ni4wMDgzOTRdLCBbMTAuOTUyODY2LCA0Ni4wMDg0NzldLCBbMTAuOTUyODkzLCA0Ni4wMDg1MTVdLCBbMTAuOTUzMDYzLCA0Ni4wMDg2NzFdLCBbMTAuOTUzMTg3LCA0Ni4wMDg4MDFdLCBbMTAuOTUzMzA0LCA0Ni4wMDg5NTVdLCBbMTAuOTUzMzQ4LCA0Ni4wMDkwMjhdLCBbMTAuOTUzNDA5LCA0Ni4wMDkxMjZdLCBbMTAuOTUzNDc2LCA0Ni4wMDkyNjddLCBbMTAuOTUzNTExLCA0Ni4wMDkzNTRdLCBbMTAuOTUzNTc0LCA0Ni4wMDkzOTVdLCBbMTAuOTUzNjYxLCA0Ni4wMDk0MThdLCBbMTAuOTUzNjg5LCA0Ni4wMDk0MjZdLCBbMTAuOTUzNzU4LCA0Ni4wMDk0MjddLCBbMTAuOTUzODM0LCA0Ni4wMDk0MTNdLCBbMTAuOTUzODk1LCA0Ni4wMDkzNzJdLCBbMTAuOTUzOTE0LCA0Ni4wMDkzMzddLCBbMTAuOTUzOTEzLCA0Ni4wMDkyNzVdLCBbMTAuOTUzOTI5LCA0Ni4wMDkxOTNdLCBbMTAuOTUzOTU3LCA0Ni4wMDkxMzNdLCBbMTAuOTU0MDE1LCA0Ni4wMDkwNTVdLCBbMTAuOTU0MSwgNDYuMDA5MDAyXSwgWzEwLjk1NDE3NSwgNDYuMDA4OTY3XSwgWzEwLjk1NDI3MywgNDYuMDA4OTQ2XSwgWzEwLjk1NDM3NywgNDYuMDA4OTA4XSwgWzEwLjk1NDQ1OSwgNDYuMDA4ODY5XSwgWzEwLjk1NDU5LCA0Ni4wMDg4NzVdLCBbMTAuOTU0NjE3LCA0Ni4wMDg5MTZdLCBbMTAuOTU0NjE4LCA0Ni4wMDg5NzNdLCBbMTAuOTU0NjE5LCA0Ni4wMDkwNDRdLCBbMTAuOTU0NjA4LCA0Ni4wMDkxMzVdLCBbMTAuOTU0NjI1LCA0Ni4wMDkxOTldLCBbMTAuOTU0NjY4LCA0Ni4wMDkyMTddLCBbMTAuOTU0NjkzLCA0Ni4wMDkyMTVdLCBbMTAuOTU0NzUzLCA0Ni4wMDkyMV0sIFsxMC45NTQ3OTIsIDQ2LjAwOTE4XSwgWzEwLjk1NDg0OCwgNDYuMDA5MTQ3XSwgWzEwLjk1NDksIDQ2LjAwOTEzNV0sIFsxMC45NTQ5NDksIDQ2LjAwOTE0XSwgWzEwLjk1NTAxMiwgNDYuMDA5MTZdLCBbMTAuOTU1MDgyLCA0Ni4wMDkyXSwgWzEwLjk1NTExNSwgNDYuMDA5MjQ4XSwgWzEwLjk1NTE2MiwgNDYuMDA5MzA5XSwgWzEwLjk1NTM4NywgNDYuMDA5MzldLCBbMTAuOTU1NDk2LCA0Ni4wMDk0MTJdLCBbMTAuOTU1NjU0LCA0Ni4wMDk0MzZdLCBbMTAuOTU1ODE4LCA0Ni4wMDk0Nl0sIFsxMC45NTU5NDMsIDQ2LjAwOTQ3NV0sIFsxMC45NTYwODksIDQ2LjAwOTUzOF0sIFsxMC45NTYxODksIDQ2LjAwOTYxNV0sIFsxMC45NTYyMzIsIDQ2LjAwOTY3NF0sIFsxMC45NTYyNjMsIDQ2LjAwOTc0N10sIFsxMC45NTYyOTEsIDQ2LjAwOTgzOF0sIFsxMC45NTYzMjksIDQ2LjAwOTk5M10sIFsxMC45NTYzNDgsIDQ2LjAxMDEyOF0sIFsxMC45NTYzNTksIDQ2LjAxMDIzNV0sIFsxMC45NTYzNjEsIDQ2LjAxMDMzOF0sIFsxMC45NTYzNzMsIDQ2LjAxMDQ1M10sIFsxMC45NTYzODcsIDQ2LjAxMDU0OF0sIFsxMC45NTY0MDYsIDQ2LjAxMDY1OF0sIFsxMC45NTY0MzQsIDQ2LjAxMDc4OF0sIFsxMC45NTY0NTUsIDQ2LjAxMDg5NV0sIFsxMC45NTY0NjcsIDQ2LjAxMTAxXSwgWzEwLjk1NjY1NCwgNDYuMDEwOTldLCBbMTAuOTU2ODM0LCA0Ni4wMTA5N11dXV0sICJ0eXBlIjogIk11bHRpUG9seWdvbiJ9LCAiaWQiOiAiNSIsICJwcm9wZXJ0aWVzIjogeyJkZXNjcml6aW9uZV9vcmlnaW5lIjogIk5vbiBjb2RpZmljYXRvIiwgImdtbF9pZCI6ICJJRC5BQ1FVRUZJU0lDSEUuU1BFQ0NISS5BQ1FVQS41NDE1NiIsICJuYXNwIjogMCwgIm5hdHVyYV9zcGVjY2hpb19hY3F1YSI6ICJOb24gY29kaWZpY2F0byIsICJub21lIjogIkxBR08gREkgQ0FWRURJTkUiLCAib3JpZ2luZSI6IDB9LCAidHlwZSI6ICJGZWF0dXJlIn0sIHsiZ2VvbWV0cnkiOiB7ImNvb3JkaW5hdGVzIjogW1tbWzEwLjgxOTk1LCA0NS45MzU3MDldLCBbMTAuODE5OTQ2LCA0NS45MzU2MjddLCBbMTAuODE5OTE5LCA0NS45MzU1N10sIFsxMC44MTk4ODMsIDQ1LjkzNTUwOF0sIFsxMC44MTk4MzcsIDQ1LjkzNTQ0XSwgWzEwLjgxOTgwMSwgNDUuOTM1Mzk3XSwgWzEwLjgxOTc2MSwgNDUuOTM1MzYyXSwgWzEwLjgxOTcyMiwgNDUuOTM1MzM3XSwgWzEwLjgxOTYzOSwgNDUuOTM1MzFdLCBbMTAuODE5NTcxLCA0NS45MzUyOTddLCBbMTAuODE5NDkyLCA0NS45MzUyOTldLCBbMTAuODE5MzksIDQ1LjkzNTMzNl0sIFsxMC44MTkzMDUsIDQ1LjkzNTM3M10sIFsxMC44MTkyMDcsIDQ1LjkzNTQyMV0sIFsxMC44MTkwNjcsIDQ1LjkzNTQ3Ml0sIFsxMC44MTg3MTMsIDQ1LjkzNTYyOF0sIFsxMC44MTg1MjcsIDQ1LjkzNTcyNV0sIFsxMC44MTgzNTQsIDQ1LjkzNTgwN10sIFsxMC44MTgxMTEsIDQ1LjkzNTkwMl0sIFsxMC44MTc5MjUsIDQ1LjkzNTk2Nl0sIFsxMC44MTc3MzIsIDQ1LjkzNjAwOF0sIFsxMC44MTc0ODMsIDQ1LjkzNjA1Ml0sIFsxMC44MTcyNDQsIDQ1LjkzNjA4M10sIFsxMC44MTY5OTgsIDQ1LjkzNjEwNl0sIFsxMC44MTY3NTksIDQ1LjkzNjEzOV0sIFsxMC44MTY1NzUsIDQ1LjkzNjE1NV0sIFsxMC44MTYzODgsIDQ1LjkzNjE3Ml0sIFsxMC44MTYyMTUsIDQ1LjkzNjE2M10sIFsxMC44MTYwNTEsIDQ1LjkzNjE0MV0sIFsxMC44MTU4OTMsIDQ1LjkzNjEwOV0sIFsxMC44MTU3MjIsIDQ1LjkzNjA2Ml0sIFsxMC44MTU1NjIsIDQ1LjkzNjAxMl0sIFsxMC44MTU0MDEsIDQ1LjkzNTk2Ml0sIFsxMC44MTUyODYsIDQ1LjkzNTkzNV0sIFsxMC44MTUxNTUsIDQ1LjkzNTkyMV0sIFsxMC44MTUwODEsIDQ1LjkzNTkyNF0sIFsxMC44MTUwMiwgNDUuOTM1OTI2XSwgWzEwLjgxNDkwMiwgNDUuOTM1OTQ3XSwgWzEwLjgxNDc3OCwgNDUuOTM1OTg5XSwgWzEwLjgxNDY3LCA0NS45MzYwMjhdLCBbMTAuODE0NTY1LCA0NS45MzYwNjldLCBbMTAuODE0Mzc2LCA0NS45MzYxODRdLCBbMTAuODE0MjYxLCA0NS45MzYyNjVdLCBbMTAuODE0MTc2LCA0NS45MzYzMzNdLCBbMTAuODE0MTAxLCA0NS45MzY0MTRdLCBbMTAuODE0MDMzLCA0NS45MzY0OTZdLCBbMTAuODEzODkzLCA0NS45MzY2N10sIFsxMC44MTM4MzEsIDQ1LjkzNjcyNV0sIFsxMC44MTM3NTMsIDQ1LjkzNjc5NF0sIFsxMC44MTM2OTcsIDQ1LjkzNjg1Nl0sIFsxMC44MTM2MTIsIDQ1LjkzNjk2OF0sIFsxMC44MTM1ODMsIDQ1LjkzNzAyM10sIFsxMC44MTM1NTQsIDQ1LjkzNzEwNV0sIFsxMC44MTM1MzgsIDQ1LjkzNzE5Ml0sIFsxMC44MTM1MTIsIDQ1LjkzNzI5M10sIFsxMC44MTM0ODMsIDQ1LjkzNzM2NF0sIFsxMC44MTM0NTEsIDQ1LjkzNzQzXSwgWzEwLjgxMzQxMiwgNDUuOTM3NTE3XSwgWzEwLjgxMzM2NiwgNDUuOTM3NTk3XSwgWzEwLjgxMzMyMSwgNDUuOTM3NjU5XSwgWzEwLjgxMzIyLCA0NS45Mzc3NTFdLCBbMTAuODEzMTI4LCA0NS45Mzc4MzhdLCBbMTAuODEzMDQ3LCA0NS45Mzc5MzldLCBbMTAuODEyOTk1LCA0NS45MzgwMzldLCBbMTAuODEyOTU2LCA0NS45MzgxNDVdLCBbMTAuODEyOTMsIDQ1LjkzODI1XSwgWzEwLjgxMjkzMSwgNDUuOTM4MzU3XSwgWzEwLjgxMjk0NCwgNDUuOTM4NDY3XSwgWzEwLjgxMjk3MSwgNDUuOTM4NTc5XSwgWzEwLjgxMjk5OCwgNDUuOTM4NjY4XSwgWzEwLjgxMzA0NCwgNDUuOTM4NzY4XSwgWzEwLjgxMzA5NywgNDUuOTM4ODU3XSwgWzEwLjgxMzEzLCA0NS45Mzg5MjhdLCBbMTAuODEzMTYzLCA0NS45MzkwMV0sIFsxMC44MTMxOSwgNDUuOTM5MDddLCBbMTAuODEzMjIsIDQ1LjkzOTEzOF0sIFsxMC44MTMyNTYsIDQ1LjkzOTIzXSwgWzEwLjgxMzI3NiwgNDUuOTM5MzEyXSwgWzEwLjgxMzMsIDQ1LjkzOTM4XSwgWzEwLjgxMzMxLCA0NS45Mzk0Nl0sIFsxMC44MTMzMSwgNDUuOTM5NTQ5XSwgWzEwLjgxMzMxMSwgNDUuOTM5NjQzXSwgWzEwLjgxMzMxNCwgNDUuOTM5NzM5XSwgWzEwLjgxMzMzMSwgNDUuOTM5ODM3XSwgWzEwLjgxMzM1NSwgNDUuOTM5OTQ1XSwgWzEwLjgxMzQ0MSwgNDUuOTQwMTIxXSwgWzEwLjgxMzQ5LCA0NS45NDAxOTZdLCBbMTAuODEzNTE0LCA0NS45NDAyNjJdLCBbMTAuODEzNTQ3LCA0NS45NDA0MzNdLCBbMTAuODEzNTU0LCA0NS45NDA1MzhdLCBbMTAuODEzNTY4LCA0NS45NDA2NV0sIFsxMC44MTM1OTUsIDQ1Ljk0MDcyOF0sIFsxMC44MTM2NDcsIDQ1Ljk0MDgyMl0sIFsxMC44MTM2OTcsIDQ1Ljk0MDkxMV0sIFsxMC44MTM3NCwgNDUuOTQxMDAyXSwgWzEwLjgxMzc3NywgNDUuOTQxMTA5XSwgWzEwLjgxMzc4NywgNDUuOTQxMjA1XSwgWzEwLjgxMzk0NSwgNDUuOTQxMzQyXSwgWzEwLjgxNDAzNCwgNDUuOTQxMzM5XSwgWzEwLjgxNDA3LCA0NS45NDEzMjZdLCBbMTAuODE0MDk5LCA0NS45NDEzXSwgWzEwLjgxNDExOCwgNDUuOTQxMjY2XSwgWzEwLjgxNDEyOCwgNDUuOTQxMjJdLCBbMTAuODE0MTI4LCA0NS45NDExODZdLCBbMTAuODE0MjQ5LCA0NS45NDEwNzRdLCBbMTAuODE0MzQxLCA0NS45NDExMDFdLCBbMTAuODE0NDEsIDQ1Ljk0MTE0XSwgWzEwLjgxNDQ1OSwgNDUuOTQxMTY5XSwgWzEwLjgxNDU4NywgNDUuOTQxMjQyXSwgWzEwLjgxNDY4NiwgNDUuOTQxMjc2XSwgWzEwLjgxNDgyNywgNDUuOTQxMjg3XSwgWzEwLjgxNDg3LCA0NS45NDEyNzldLCBbMTAuODE0OTI4LCA0NS45NDEyNjhdLCBbMTAuODE0OTc0LCA0NS45NDEyNDhdLCBbMTAuODE1MDMzLCA0NS45NDEyMDZdLCBbMTAuODE1MDg4LCA0NS45NDExNTZdLCBbMTAuODE1MTQ0LCA0NS45NDEwOV0sIFsxMC44MTUxNiwgNDUuOTQxMDI1XSwgWzEwLjgxNTE4NiwgNDUuOTQwOTU1XSwgWzEwLjgxNTIyNSwgNDUuOTQwODg4XSwgWzEwLjgxNTI4MywgNDUuOTQwODAxXSwgWzEwLjgxNTMzMiwgNDUuOTQwNzA3XSwgWzEwLjgxNTM5NCwgNDUuOTQwNjVdLCBbMTAuODE1NTA1LCA0NS45NDA1NjddLCBbMTAuODE1NTc2LCA0NS45NDAzODldLCBbMTAuODE1NjM1LCA0NS45NDAzMDRdLCBbMTAuODE1NywgNDUuOTQwMjQyXSwgWzEwLjgxNTc3OSwgNDUuOTQwMTk0XSwgWzEwLjgxNTksIDQ1Ljk0MDEzN10sIFsxMC44MTYwMTQsIDQ1Ljk0MDEwMl0sIFsxMC44MTYxMjIsIDQ1Ljk0MDA2NV0sIFsxMC44MTYyMDcsIDQ1Ljk0MDAxN10sIFsxMC44MTYyODMsIDQ1LjkzOTk3Nl0sIFsxMC44MTYzMzIsIDQ1LjkzOTk0OF0sIFsxMC44MTY0LCA0NS45Mzk5MThdLCBbMTAuODE2NDcyLCA0NS45Mzk5MDldLCBbMTAuODE2Nzg0LCA0NS45Mzk5NF0sIFsxMC44MTY4OTIsIDQ1LjkzOTk5OV0sIFsxMC44MTY5ODEsIDQ1Ljk0MDA1Nl0sIFsxMC44MTcwNCwgNDUuOTQwMDk5XSwgWzEwLjgxNzExLCA0NS45NDAxNzJdLCBbMTAuODE3MTg1LCA0NS45NDAyMzRdLCBbMTAuODE3Mjc0LCA0NS45NDAzMDRdLCBbMTAuODE3MzYsIDQ1Ljk0MDM4NF0sIFsxMC44MTc0MzksIDQ1Ljk0MDQ1N10sIFsxMC44MTc1MzEsIDQ1Ljk0MDUzN10sIFsxMC44MTc2MSwgNDUuOTQwNTg5XSwgWzEwLjgxNzcwOSwgNDUuOTQwNjU3XSwgWzEwLjgxNzc3MSwgNDUuOTQwNzA3XSwgWzEwLjgxNzgyMSwgNDUuOTQwNzZdLCBbMTAuODE3ODc3LCA0NS45NDA4MDhdLCBbMTAuODE3OTI5LCA0NS45NDA4MTRdLCBbMTAuODE3OTk4LCA0NS45NDA4MTJdLCBbMTAuODE4MDQsIDQ1Ljk0MDc3N10sIFsxMC44MTgwNzMsIDQ1Ljk0MDcyNV0sIFsxMC44MTgwNzksIDQ1Ljk0MDY5MV0sIFsxMC44MTgwODYsIDQ1Ljk0MDYyN10sIFsxMC44MTgwODUsIDQ1Ljk0MDU2NV0sIFsxMC44MTgwNzgsIDQ1Ljk0MDUxNV0sIFsxMC44MTgwMzksIDQ1Ljk0MDQxOV0sIFsxMC44MTc5OTIsIDQ1Ljk0MDM1NV0sIFsxMC44MTc5NDMsIDQ1Ljk0MDI4OV0sIFsxMC44MTc4OSwgNDUuOTQwMjA2XSwgWzEwLjgxNzg1NCwgNDUuOTQwMTM2XSwgWzEwLjgxNzgxNCwgNDUuOTQwMDU2XSwgWzEwLjgxNzc4MSwgNDUuOTM5OTgzXSwgWzEwLjgxNzc2NywgNDUuOTM5OTAzXSwgWzEwLjgxNzc0NywgNDUuOTM5Nzg5XSwgWzEwLjgxNzczNCwgNDUuOTM5NzA2XSwgWzEwLjgxNzc0LCA0NS45Mzk2MTldLCBbMTAuODE3ODA1LCA0NS45Mzk1MzddLCBbMTAuODE3ODUsIDQ1LjkzOTQ4N10sIFsxMC44MTc5ODQsIDQ1LjkzOTM0Ml0sIFsxMC44MTgwNTksIDQ1LjkzOTI1M10sIFsxMC44MTgxNDQsIDQ1LjkzOTEzOF0sIFsxMC44MTgyMjUsIDQ1LjkzOTAyNl0sIFsxMC44MTgzMTYsIDQ1LjkzODldLCBbMTAuODE4NDExLCA0NS45Mzg3NzldLCBbMTAuODE4NTQ0LCA0NS45Mzg2MDddLCBbMTAuODE4NTg0LCA0NS45Mzg1NTRdLCBbMTAuODE4ODQ3LCA0NS45MzgxNzJdLCBbMTAuODE4OTM4LCA0NS45MzgwNTVdLCBbMTAuODE5MTUsIDQ1LjkzNzc5Nl0sIFsxMC44MTkyMzUsIDQ1LjkzNzY3OV0sIFsxMC44MTkzMSwgNDUuOTM3NTc5XSwgWzEwLjgxOTQxNywgNDUuOTM3NDQzXSwgWzEwLjgxOTQ5OSwgNDUuOTM3MzM4XSwgWzEwLjgxOTU5LCA0NS45MzcyM10sIFsxMC44MTk2NTIsIDQ1LjkzNzE0Nl0sIFsxMC44MTk3MDcsIDQ1LjkzNzA2OF0sIFsxMC44MTk3MzMsIDQ1LjkzNjk5Ml0sIFsxMC44MTk3NDYsIDQ1LjkzNjkxOV0sIFsxMC44MTk3NTgsIDQ1LjkzNjgzMl0sIFsxMC44MTk3NzEsIDQ1LjkzNjY2MV0sIFsxMC44MTk3NjQsIDQ1LjkzNjU3Nl0sIFsxMC44MTk3NiwgNDUuOTM2NTAxXSwgWzEwLjgxOTc2OSwgNDUuOTM2Mzg5XSwgWzEwLjgxOTc3NiwgNDUuOTM2MzE2XSwgWzEwLjgxOTc4OCwgNDUuOTM2MjQyXSwgWzEwLjgxOTgwNCwgNDUuOTM2MTUzXSwgWzEwLjgxOTgxNywgNDUuOTM2MDkxXSwgWzEwLjgxOTg0NiwgNDUuOTM1OTk1XSwgWzEwLjgxOTg4MiwgNDUuOTM1OTA2XSwgWzEwLjgxOTk0LCA0NS45MzU3OTJdLCBbMTAuODE5OTUsIDQ1LjkzNTcwOV1dXV0sICJ0eXBlIjogIk11bHRpUG9seWdvbiJ9LCAiaWQiOiAiNiIsICJwcm9wZXJ0aWVzIjogeyJkZXNjcml6aW9uZV9vcmlnaW5lIjogIk5vbiBjb2RpZmljYXRvIiwgImdtbF9pZCI6ICJJRC5BQ1FVRUZJU0lDSEUuU1BFQ0NISS5BQ1FVQS41NDE3NiIsICJuYXNwIjogMCwgIm5hdHVyYV9zcGVjY2hpb19hY3F1YSI6ICJOb24gY29kaWZpY2F0byIsICJub21lIjogIkxBR08gREkgVEVOTk8iLCAib3JpZ2luZSI6IDB9LCAidHlwZSI6ICJGZWF0dXJlIn0sIHsiZ2VvbWV0cnkiOiB7ImNvb3JkaW5hdGVzIjogW1tbWzEwLjgyNTg1MywgNDUuODM3NjNdLCBbMTAuODI1OSwgNDUuODM3NzFdLCBbMTAuODI1OTExLCA0NS44Mzc3OTldLCBbMTAuODI1OTEsIDQ1LjgzNzkzMV0sIFsxMC44MjU5MzgsIDQ1LjgzODA2Nl0sIFsxMC44MjU5NTMsIDQ1LjgzODE2Ml0sIFsxMC44MjU5NTUsIDQ1LjgzODMwMV0sIFsxMC44MjU5OTYsIDQ1LjgzODQ0N10sIFsxMC44MjYwNjEsIDQ1LjgzODU5OF0sIFsxMC44MjYwODIsIDQ1LjgzODcyM10sIFsxMC44MjYxMjcsIDQ1LjgzODg3Nl0sIFsxMC44MjYyMjMsIDQ1LjgzOTEyXSwgWzEwLjgyNjMwNCwgNDUuODM5MzExXSwgWzEwLjgyNjI5LCA0NS44Mzk0ODFdLCBbMTAuODI2MjYxLCA0NS44Mzk2ODJdLCBbMTAuODI2Mjk0LCA0NS44Mzk5Nl0sIFsxMC44MjYyMDMsIDQ1Ljg0MDQwNV0sIFsxMC44MjYxNDMsIDQ1Ljg0MDU1Nl0sIFsxMC44MjYxNjYsIDQ1Ljg0MDc4OV0sIFsxMC44MjYxNzgsIDQ1Ljg0MDk0Ml0sIFsxMC44MjYxNDQsIDQ1Ljg0MTA1Nl0sIFsxMC44MjYxNDIsIDQ1Ljg0MTE3NV0sIFsxMC44MjYxNywgNDUuODQxMjY0XSwgWzEwLjgyNjE3MSwgNDUuODQxMzM1XSwgWzEwLjgyNjE1OSwgNDUuODQxNDI5XSwgWzEwLjgyNjIxNywgNDUuODQxNTRdLCBbMTAuODI2MzA2LCA0NS44NDE2MjRdLCBbMTAuODI2NDE3LCA0NS44NDE4MzhdLCBbMTAuODI2NTE1LCA0NS44NDE5ODRdLCBbMTAuODI2NjI3LCA0NS44NDIyNTNdLCBbMTAuODI2NjQ5LCA0NS44NDI0MDZdLCBbMTAuODI2NjkxLCA0NS44NDI2MDRdLCBbMTAuODI2NzQyLCA0NS44NDI3NjhdLCBbMTAuODI2ODAxLCA0NS44NDI5M10sIFsxMC44MjY5NTQsIDQ1Ljg0MzExMl0sIFsxMC44MjcxMzcsIDQ1Ljg0MzMxNl0sIFsxMC44MjcyNzQsIDQ1Ljg0MzQ5M10sIFsxMC44Mjc1MjYsIDQ1Ljg0MzcyOV0sIFsxMC44Mjc4MzEsIDQ1Ljg0Mzk3Nl0sIFsxMC44MjgzMjMsIDQ1Ljg0NDI2MV0sIFsxMC44Mjg0NjgsIDQ1Ljg0NDMzN10sIFsxMC44Mjg2MDEsIDQ1Ljg0NDQzOV0sIFsxMC44Mjg3OTIsIDQ1Ljg0NDU3XSwgWzEwLjgyODkwOSwgNDUuODQ0N10sIFsxMC44Mjg5OTcsIDQ1Ljg0NDg2MV0sIFsxMC44MjkwNTIsIDQ1Ljg0NTA0OF0sIFsxMC44MjkxNTksIDQ1Ljg0NTE4Ml0sIFsxMC44MjkzMDgsIDQ1Ljg0NTMwNV0sIFsxMC44Mjk0NDYsIDQ1Ljg0NTM3XSwgWzEwLjgyOTYwOCwgNDUuODQ1NDQ2XSwgWzEwLjgyOTc4NiwgNDUuODQ1NTU3XSwgWzEwLjgyOTkzMSwgNDUuODQ1NjEzXSwgWzEwLjgzMDE2OSwgNDUuODQ1NzUzXSwgWzEwLjgzMDI3MiwgNDUuODQ1ODM1XSwgWzEwLjgzMDMwMiwgNDUuODQ1OTAzXSwgWzEwLjgzMDMwNCwgNDUuODQ2MDRdLCBbMTAuODMwMzM1LCA0NS44NDYxNjFdLCBbMTAuODMwNDE5LCA0NS44NDYzMDJdLCBbMTAuODMwNDk3LCA0NS44NDY0NTJdLCBbMTAuODMwNDg5LCA0NS44NDY1NTVdLCBbMTAuODMwNTUzLCA0NS44NDY2NzZdLCBbMTAuODMwNTc3LCA0NS44NDY3NjVdLCBbMTAuODMwNTk5LCA0NS44NDY5MjJdLCBbMTAuODMwNjQ0LCA0NS44NDcwNjFdLCBbMTAuODMwNjg4LCA0NS44NDcxODldLCBbMTAuODMwNzA0LCA0NS44NDczNTZdLCBbMTAuODMwNjg3LCA0NS44NDc1NzhdLCBbMTAuODMwNjgxLCA0NS44NDc3NjddLCBbMTAuODMwNzkyLCA0NS44NDc5ODFdLCBbMTAuODMwOTExLCA0NS44NDgyODldLCBbMTAuODMxMDU1LCA0NS44NDg2ODZdLCBbMTAuODMxMjM5LCA0NS44NDg5NzJdLCBbMTAuODMxMzE0LCA0NS44NDkxXSwgWzEwLjgzMTQ4NiwgNDUuODQ5NzEzXSwgWzEwLjgzMTU0NCwgNDUuODQ5OTE5XSwgWzEwLjgzMTY0MywgNDUuODUwMTc5XSwgWzEwLjgzMTc1NCwgNDUuODUwNDAyXSwgWzEwLjgzMTg1MiwgNDUuODUwNjM1XSwgWzEwLjgzMjAyNywgNDUuODUwODY2XSwgWzEwLjgzMjE3NywgNDUuODUxMTM0XSwgWzEwLjgzMjMzOCwgNDUuODUxMzcyXSwgWzEwLjgzMjQ4OCwgNDUuODUxNjg5XSwgWzEwLjgzMjU1OSwgNDUuODUxODIxXSwgWzEwLjgzMjY2MSwgNDUuODUxOTk4XSwgWzEwLjgzMjc3OSwgNDUuODUyMTU3XSwgWzEwLjgzMjkyNCwgNDUuODUyMzcxXSwgWzEwLjgzMzAxOSwgNDUuODUyNTE2XSwgWzEwLjgzMzA5NywgNDUuODUyNjQ2XSwgWzEwLjgzMzE3OSwgNDUuODUyODE0XSwgWzEwLjgzMzIyNiwgNDUuODUzMDAxXSwgWzEwLjgzMzMyOCwgNDUuODUzMTYzXSwgWzEwLjgzMzQ1MywgNDUuODUzMzU0XSwgWzEwLjgzMzc2OCwgNDUuODUzNzIxXSwgWzEwLjgzMzg1MiwgNDUuODUzOTUxXSwgWzEwLjgzNDA0MywgNDUuODU0MTc2XSwgWzEwLjgzNDE0MSwgNDUuODU0MzAxXSwgWzEwLjgzNDIzLCA0NS44NTQzNzNdLCBbMTAuODM0MzQxLCA0NS44NTQ1MDJdLCBbMTAuODM0Mzk1LCA0NS44NTQ3MjNdLCBbMTAuODM0Mzg5LCA0NS44NTQ4ODhdLCBbMTAuODM0NDEzLCA0NS44NTUxMDNdLCBbMTAuODM0NDM4LCA0NS44NTUzNTRdLCBbMTAuODM0NDY4LCA0NS44NTU1MTldLCBbMTAuODM0NDE1LCA0NS44NTU2NDddLCBbMTAuODM0Mjk5LCA0NS44NTU3NDVdLCBbMTAuODM0MTQzLCA0NS44NTU5MzRdLCBbMTAuODM0MDk5LCA0NS44NTYxNzJdLCBbMTAuODM0MDU2LCA0NS44NTYzNDJdLCBbMTAuODM0MDEsIDQ1Ljg1NjU5OV0sIFsxMC44MzM5NjIsIDQ1Ljg1NjgxOV0sIFsxMC44MzM4NTIsIDQ1Ljg1NzAxOV0sIFsxMC44MzM3NDYsIDQ1Ljg1NzEyM10sIFsxMC44MzM2MjQsIDQ1Ljg1NzI1NF0sIFsxMC44MzM1NjYsIDQ1Ljg1NzQxNV0sIFsxMC44MzM0OTQsIDQ1Ljg1NzYwMV0sIFsxMC44MzMzODMsIDQ1Ljg1Nzc1Nl0sIFsxMC44MzMyOTksIDQ1Ljg1NzgyN10sIFsxMC44MzMyNDYsIDQ1Ljg1NzkxOV0sIFsxMC44MzMyMTQsIDQ1Ljg1ODEwN10sIFsxMC44MzMxMiwgNDUuODU4MzIxXSwgWzEwLjgzMjkyNSwgNDUuODU4NzA3XSwgWzEwLjgzMjg4NywgNDUuODU4ODg0XSwgWzEwLjgzMjgzNCwgNDUuODU5MDI0XSwgWzEwLjgzMjg3NCwgNDUuODU5MjI3XSwgWzEwLjgzMjk2NiwgNDUuODU5Mzg0XSwgWzEwLjgzMzAwNiwgNDUuODU5NzEzXSwgWzEwLjgzMjk5NCwgNDUuODU5OTQyXSwgWzEwLjgzMzE4NiwgNDUuODYwMzI5XSwgWzEwLjgzMzEzLCA0NS44NjA2MTFdLCBbMTAuODMzMDQ1LCA0NS44NjA3OTVdLCBbMTAuODMyOTk2LCA0NS44NjA5MjhdLCBbMTAuODMzMDExLCA0NS44NjEwMjhdLCBbMTAuODMzMDM3LCA0NS44NjExNTZdLCBbMTAuODMzMDcyLCA0NS44NjEyNzVdLCBbMTAuODMzMjIsIDQ1Ljg2MTYyM10sIFsxMC44MzMyOTgsIDQ1Ljg2MTc3M10sIFsxMC44MzMzODMsIDQ1Ljg2MTkwMV0sIFsxMC44MzM0ODEsIDQ1Ljg2MjAyM10sIFsxMC44MzM1OSwgNDUuODYyMjM5XSwgWzEwLjgzMzY1NSwgNDUuODYyMzk3XSwgWzEwLjgzMzc0NSwgNDUuODYyNTk1XSwgWzEwLjgzMzkxMiwgNDUuODYyOTE1XSwgWzEwLjgzNDE2OCwgNDUuODYzMzkzXSwgWzEwLjgzNDIzLCA0NS44NjM1NjRdLCBbMTAuODM0MzMxLCA0NS44NjM4MzVdLCBbMTAuODM0MzcyLCA0NS44NjQwNjRdLCBbMTAuODM0NDQ5LCA0NS44NjQzMDNdLCBbMTAuODM0NTI4LCA0NS44NjQ0NzFdLCBbMTAuODM0NjU2LCA0NS44NjQ2NTVdLCBbMTAuODM0NzE3LCA0NS44NjQ5MDZdLCBbMTAuODM0NzA2LCA0NS44NjUyOTNdLCBbMTAuODM0ODEsIDQ1Ljg2NTU2Ml0sIFsxMC44MzQ5MjgsIDQ1Ljg2NTcwN10sIFsxMC44MzQ5ODUsIDQ1Ljg2NTk0OV0sIFsxMC44MzUwMjQsIDQ1Ljg2NjA2NV0sIFsxMC44MzUwNjIsIDQ1Ljg2NjE3OV0sIFsxMC44MzUxMTgsIDQ1Ljg2NjM1XSwgWzEwLjgzNTA0MywgNDUuODY2ODAyXSwgWzEwLjgzNTAwNiwgNDUuODY2OTM3XSwgWzEwLjgzNTAyNSwgNDUuODY3MDU4XSwgWzEwLjgzNTExNiwgNDUuODY3MTYyXSwgWzEwLjgzNTI0OSwgNDUuODY3Mjc1XSwgWzEwLjgzNTMxMiwgNDUuODY3MzE0XSwgWzEwLjgzNTM4NywgNDUuODY3NDU3XSwgWzEwLjgzNTMzOCwgNDUuODY3NjIyXSwgWzEwLjgzNTI5MywgNDUuODY3Nzk3XSwgWzEwLjgzNTI2NCwgNDUuODY3OTgyXSwgWzEwLjgzNTIxNSwgNDUuODY4MTEzXSwgWzEwLjgzNTE0MiwgNDUuODY4MjM1XSwgWzEwLjgzNTA5MywgNDUuODY4NF0sIFsxMC44MzUwNTYsIDQ1Ljg2ODY0NV0sIFsxMC44MzUwNDcsIDQ1Ljg2ODg1MV0sIFsxMC44MzUwNjEsIDQ1Ljg2OTA1NV0sIFsxMC44MzUwMTUsIDQ1Ljg2OTE4OF0sIFsxMC44MzUwMzIsIDQ1Ljg2OTM0M10sIFsxMC44MzUxMTgsIDQ1Ljg2OTQyN10sIFsxMC44MzUyMjksIDQ1Ljg2OTU0XSwgWzEwLjgzNTMzMywgNDUuODY5NjQ0XSwgWzEwLjgzNTQyOCwgNDUuODY5ODIyXSwgWzEwLjgzNTUwNiwgNDUuODY5OTI2XSwgWzEwLjgzNTY3LCA0NS44NzAxMzVdLCBbMTAuODM1Nzk1LCA0NS44NzAyNzNdLCBbMTAuODM1OTc4LCA0NS44NzA0MzJdLCBbMTAuODM2MTg3LCA0NS44NzA1NzRdLCBbMTAuODM2MzY5LCA0NS44NzA2N10sIFsxMC44MzY1ODUsIDQ1Ljg3MDc4N10sIFsxMC44MzY4NDQsIDQ1Ljg3MDk1M10sIFsxMC44MzcwNCwgNDUuODcxMDg5XSwgWzEwLjgzNzI2MSwgNDUuODcxMTcxXSwgWzEwLjgzNzQ2NiwgNDUuODcxMjkyXSwgWzEwLjgzNzYzNCwgNDUuODcxNDkyXSwgWzEwLjgzNzY3NiwgNDUuODcxNjAxXSwgWzEwLjgzNzc1MSwgNDUuODcxNzMzXSwgWzEwLjgzNzc5NywgNDUuODcxOTExXSwgWzEwLjgzNzcyOCwgNDUuODcyMDE5XSwgWzEwLjgzNzY0OSwgNDUuODcyMTZdLCBbMTAuODM3NjIzLCA0NS44NzIzMl0sIFsxMC44Mzc2OCwgNDUuODcyNDE2XSwgWzEwLjgzNzgxOCwgNDUuODcyNTU4XSwgWzEwLjgzNzg5LCA0NS44NzI3NDFdLCBbMTAuODM3OTEyLCA0NS44NzI5OTRdLCBbMTAuODM3OTc4LCA0NS44NzMxNThdLCBbMTAuODM4MDIsIDQ1Ljg3MzI4Nl0sIFsxMC44MzgwNDYsIDQ1Ljg3MzU2N10sIFsxMC44MzgwNTcsIDQ1Ljg3MzgxMl0sIFsxMC44MzgxOTgsIDQ1Ljg3Mzk1N10sIFsxMC44MzgzMTMsIDQ1Ljg3NDEyOV0sIFsxMC44MzgzMzIsIDQ1Ljg3NDI0Nl0sIFsxMC44MzgzNTMsIDQ1Ljg3NDQ1MV0sIFsxMC44MzgzODksIDQ1Ljg3NDYwMl0sIFsxMC44MzgzMDksIDQ1Ljg3NDcyNF0sIFsxMC44MzgyNDMsIDQ1Ljg3NDgyNV0sIFsxMC44MzgyODksIDQ1Ljg3NTAwMV0sIFsxMC44MzgyNDMsIDQ1Ljg3NTEzNF0sIFsxMC44MzgyMDQsIDQ1Ljg3NTI5OV0sIFsxMC44MzgyNDIsIDQ1Ljg3NTQxNl0sIFsxMC44MzgzMiwgNDUuODc1NTM4XSwgWzEwLjgzODQ4NywgNDUuODc1NjU4XSwgWzEwLjgzODYzNCwgNDUuODc1ODA1XSwgWzEwLjgzODcwMywgNDUuODc1OTYyXSwgWzEwLjgzODY3MiwgNDUuODc2MDUyXSwgWzEwLjgzODcxNywgNDUuODc2MTY2XSwgWzEwLjgzODgwNSwgNDUuODc2MjkzXSwgWzEwLjgzODkzMywgNDUuODc2NDk1XSwgWzEwLjgzOTAzOSwgNDUuODc2NjYzXSwgWzEwLjgzOTA3NywgNDUuODc2NzczXSwgWzEwLjgzOTE0NiwgNDUuODc2OTIzXSwgWzEwLjgzOTE3NSwgNDUuODc3MDgzXSwgWzEwLjgzOTIxOCwgNDUuODc3MjQ5XSwgWzEwLjgzOTI1LCA0NS44NzczNTldLCBbMTAuODM5MjgzLCA0NS44Nzc1MzldLCBbMTAuODM5MjMyLCA0NS44Nzc4ODVdLCBbMTAuODM5MDksIDQ1Ljg3ODAxXSwgWzEwLjgzODk4OCwgNDUuODc4MTI4XSwgWzEwLjgzODg1NCwgNDUuODc4MjcxXSwgWzEwLjgzODcxLCA0NS44Nzg0MjNdLCBbMTAuODM4NDkzLCA0NS44Nzg1NDJdLCBbMTAuODM4Mzg3LCA0NS44Nzg2NTFdLCBbMTAuODM4Mjk0LCA0NS44Nzg4OTddLCBbMTAuODM4MjA5LCA0NS44NzkxOV0sIFsxMC44MzgxNDEsIDQ1Ljg3OTQwNl0sIFsxMC44MzgxNTEsIDQ1Ljg3OTY5NF0sIFsxMC44MzgxMzQsIDQ1Ljg3OTk2OV0sIFsxMC44MzgxNDksIDQ1Ljg4MDIwNF0sIFsxMC44MzgxODMsIDQ1Ljg4MDQzM10sIFsxMC44MzgyMjcsIDQ1Ljg4MDY1NF0sIFsxMC44MzgyMzYsIDQ1Ljg4MDkxM10sIFsxMC44MzgyNTYsIDQ1Ljg4MTA5MV0sIFsxMC44MzgyNzcsIDQ1Ljg4MTQ0MV0sIFsxMC44Mzg1MzEsIDQ1Ljg4MjQzOF0sIFsxMC44Mzg0NjYsIDQ1Ljg4MjQ4Ml0sIFsxMC44Mzg2NSwgNDUuODgyOTY1XSwgWzEwLjgzODgyMSwgNDUuODgzMDA2XSwgWzEwLjgzODkxNywgNDUuODgzMDZdLCBbMTAuODM4OTYxLCA0NS44ODMxMjZdLCBbMTAuODM4OTkzLCA0NS44ODMzNzVdLCBbMTAuODM5MDkyLCA0NS44ODM0NDNdLCBbMTAuODM5MjE5LCA0NS44ODM1NF0sIFsxMC44MzkyODgsIDQ1Ljg4MzY4Nl0sIFsxMC44Mzk0ODcsIDQ1Ljg4Mzk2NV0sIFsxMC44MzkzOTUsIDQ1Ljg4NDQxNF0sIFsxMC44MzkxMTMsIDQ1Ljg4NDM5NF0sIFsxMC44MzkwODgsIDQ1Ljg4NDZdLCBbMTAuODM5ODA5LCA0NS44ODQ2MTZdLCBbMTAuODM5ODU2LCA0NS44ODQyNF0sIFsxMC44NDAxOTMsIDQ1Ljg4NDA3NF0sIFsxMC44NDAyMDEsIDQ1Ljg4Mzk4M10sIFsxMC44NDA0MTksIDQ1Ljg4MzkzNV0sIFsxMC44NDA0NjcsIDQ1Ljg4Mzk5MV0sIFsxMC44NDA3OTksIDQ1Ljg4MzkzN10sIFsxMC44NDA4MzMsIDQ1Ljg4Mzk4NV0sIFsxMC44NDExMzcsIDQ1Ljg4Mzk0MV0sIFsxMC44NDExNDgsIDQ1Ljg4Mzg1NF0sIFsxMC44NDA3OTgsIDQ1Ljg4Mzg1NV0sIFsxMC44NDA3OSwgNDUuODgzNzk4XSwgWzEwLjg0MTIzOCwgNDUuODgzNzgyXSwgWzEwLjg0MTI2NiwgNDUuODgzODU1XSwgWzEwLjg0MTY1NiwgNDUuODgzODUzXSwgWzEwLjg0MTc3NiwgNDUuODg0NDE1XSwgWzEwLjg0MTk3NiwgNDUuODg0NDI0XSwgWzEwLjg0MTk2MSwgNDUuODg0MDZdLCBbMTAuODQxOTAzLCA0NS44ODM5MjFdLCBbMTAuODQxODUxLCA0NS44ODM3ODJdLCBbMTAuODQxNzk4LCA0NS44ODM2MzldLCBbMTAuODQyNTI1LCA0NS44ODM0NjldLCBbMTAuODQyNzM4LCA0NS44ODM0NzRdLCBbMTAuODQyNzE3LCA0NS44ODQ2NDNdLCBbMTAuODQxOTc0LCA0NS44ODQ2NDhdLCBbMTAuODQxOTgyLCA0NS44ODQ1N10sIFsxMC44NDE4NTgsIDQ1Ljg4NDU2N10sIFsxMC44NDE4NjUsIDQ1Ljg4NDc0MV0sIFsxMC44NDMwMDksIDQ1Ljg4NDhdLCBbMTAuODQzMDcsIDQ1Ljg4MzIyOF0sIFsxMC44NDMxMzYsIDQ1Ljg4MzI0MV0sIFsxMC44NDMxMTUsIDQ1Ljg4MzUyXSwgWzEwLjg0MzE5LCA0NS44ODM1MDhdLCBbMTAuODQzMjI1LCA0NS44ODM1NzZdLCBbMTAuODQzMzAxLCA0NS44ODM2NDJdLCBbMTAuODQzNDEzLCA0NS44ODM2NjhdLCBbMTAuODQzNTI5LCA0NS44ODM2OTRdLCBbMTAuODQzODA2LCA0NS44ODM2NjhdLCBbMTAuODQzOTQ4LCA0NS44ODM1NjldLCBbMTAuODQzOTQ0LCA0NS44ODM1MDVdLCBbMTAuODQzOTY1LCA0NS44ODM0MjJdLCBbMTAuODQ0MDEzLCA0NS44ODMzODddLCBbMTAuODQ0MDk4LCA0NS44ODMzNzNdLCBbMTAuODQ0MjU0LCA0NS44ODMzNDFdLCBbMTAuODQ0MjYsIDQ1Ljg4MzI3M10sIFsxMC44NDQzNjQsIDQ1Ljg4MzI3NF0sIFsxMC44NDQ0MzIsIDQ1Ljg4MzIwMl0sIFsxMC44NDQyNjEsIDQ1Ljg4MzA0OF0sIFsxMC44NDQwMDUsIDQ1Ljg4Mjg2OF0sIFsxMC44NDMyLCA0NS44ODI4OTVdLCBbMTAuODQzMjA4LCA0NS44ODI3OTJdLCBbMTAuODQzOTE2LCA0NS44ODI4MTldLCBbMTAuODQzOTc3LCA0NS44ODI3NjVdLCBbMTAuODQ0NTA5LCA0NS44ODMxNTZdLCBbMTAuODQ0NjQ2LCA0NS44ODMxNDNdLCBbMTAuODQ0NjkxLCA0NS44ODMyMjJdLCBbMTAuODQ1NzUyLCA0NS44ODI3ODhdLCBbMTAuODQ1NzYsIDQ1Ljg4MjY4M10sIFsxMC44NDU2NDgsIDQ1Ljg4MjUxMl0sIFsxMC44NDU2NjgsIDQ1Ljg4MjM3XSwgWzEwLjg0NTY3MiwgNDUuODgyMjQ5XSwgWzEwLjg0NTQ2NywgNDUuODgyMDM0XSwgWzEwLjg0NTM5OSwgNDUuODgxOTJdLCBbMTAuODQ1Mzc3LCA0NS44ODE4MTNdLCBbMTAuODQ1MjkyLCA0NS44ODE2MzVdLCBbMTAuODQ1MzE4LCA0NS44ODE0NjRdLCBbMTAuODQ1MzQ0LCA0NS44ODEzMjhdLCBbMTAuODQ1MzYyLCA0NS44ODEyM10sIFsxMC44NDUzOTksIDQ1Ljg4MTE0NV0sIFsxMC44NDU0ODYsIDQ1Ljg4MTFdLCBbMTAuODQ1NjA1LCA0NS44ODExMTddLCBbMTAuODQ1Nzc3LCA0NS44ODExMTNdLCBbMTAuODQ1Nzg1LCA0NS44ODExMTNdLCBbMTAuODQ2MDczLCA0NS44ODEyNTddLCBbMTAuODQ2Mjc5LCA0NS44ODEzOTJdLCBbMTAuODQ2NDU3LCA0NS44ODE0NTRdLCBbMTAuODQ2ODgxLCA0NS44ODE1NDFdLCBbMTAuODQ3MDQ0LCA0NS44ODE1MTZdLCBbMTAuODQ3Mjk2LCA0NS44ODE1MDVdLCBbMTAuODQ3NTY2LCA0NS44ODE0MjRdLCBbMTAuODQ4NDA4LCA0NS44ODEyODddLCBbMTAuODQ4NjI3LCA0NS44ODEyNDZdLCBbMTAuODQ4ODE2LCA0NS44ODEyMzVdLCBbMTAuODQ5MDgxLCA0NS44ODExODRdLCBbMTAuODQ5MjkyLCA0NS44ODA5NzFdLCBbMTAuODQ5MzE4LCA0NS44ODA4MTNdLCBbMTAuODQ5Mzg0LCA0NS44ODA3MDFdLCBbMTAuODQ5NDk2LCA0NS44ODA1NzFdLCBbMTAuODQ5NTY1LCA0NS44ODA1MzJdLCBbMTAuODQ5NjY0LCA0NS44ODA0NzZdLCBbMTAuODQ5ODM2LCA0NS44ODA1Ml0sIFsxMC44NDk5ODksIDQ1Ljg4MDUwNF0sIFsxMC44NTAwNDMsIDQ1Ljg4MDI4N10sIFsxMC44NTAxOTIsIDQ1Ljg4MDE4OV0sIFsxMC44NTAzNCwgNDUuODgwMDcxXSwgWzEwLjg1MDUwMiwgNDUuODc5OThdLCBbMTAuODUwNzIsIDQ1Ljg3OTkzNF0sIFsxMC44NTA4ODMsIDQ1Ljg3OTg3NV0sIFsxMC44NTExMDQsIDQ1Ljg3OTgxMV0sIFsxMC44NTEzOTEsIDQ1Ljg3OTc0Ml0sIFsxMC44NTE0LCA0NS44Nzk4ODhdLCBbMTAuODUxNDYsIDQ1Ljg3OTkyOV0sIFsxMC44NTE1OTEsIDQ1Ljg3OTkwN10sIFsxMC44NTE2OTgsIDQ1Ljg3OTg4NV0sIFsxMC44NTIxNTUsIDQ1Ljg3OTY2NV0sIFsxMC44NTIzODMsIDQ1Ljg3OTYwOF0sIFsxMC44NTI0MzgsIDQ1Ljg3OTU4OV0sIFsxMC44NTIyMjQsIDQ1Ljg3OTUzNl0sIFsxMC44NTE5NzYsIDQ1Ljg3OTU4Ml0sIFsxMC44NTE3NSwgNDUuODc5NTVdLCBbMTAuODUxNjQxLCA0NS44Nzk1MV0sIFsxMC44NTE3NTcsIDQ1Ljg3OTQyNV0sIFsxMC44NTIwMDksIDQ1Ljg3OTQyNF0sIFsxMC44NTIyMTgsIDQ1Ljg3OTM5OV0sIFsxMC44NTI1MTIsIDQ1Ljg3OTM3NV0sIFsxMC44NTI2OTEsIDQ1Ljg3OTI5OF0sIFsxMC44NTI4MzMsIDQ1Ljg3OTIzM10sIFsxMC44NTMwMTEsIDQ1Ljg3OTEzN10sIFsxMC44NTM2MzEsIDQ1Ljg3OTAxMV0sIFsxMC44NTM5MzYsIDQ1Ljg3ODkxNF0sIFsxMC44NTQzNiwgNDUuODc4Nzk4XSwgWzEwLjg1NDYxMSwgNDUuODc4NzQzXSwgWzEwLjg1NDk0LCA0NS44Nzg3MDNdLCBbMTAuODU1MTY4LCA0NS44Nzg2MjddLCBbMTAuODU1MzI0LCA0NS44Nzg1NjhdLCBbMTAuODU1NDY1LCA0NS44Nzg0NDhdLCBbMTAuODU1NTEyLCA0NS44NzgzNDRdLCBbMTAuODU1NTU5LCA0NS44NzgyMjNdLCBbMTAuODU1NzA1LCA0NS44NzgxOThdLCBbMTAuODU1NzYsIDQ1Ljg3ODMwNV0sIFsxMC44NTU3NTIsIDQ1Ljg3ODM5NF0sIFsxMC44NTU3ODMsIDQ1Ljg3ODQyOF0sIFsxMC44NTY1LCA0NS44Nzg1ODhdLCBbMTAuODU3MTkyLCA0NS44Nzc3MzRdLCBbMTAuODU2NzExLCA0NS44Nzc0NTNdLCBbMTAuODU1NTU5LCA0NS44Nzc5MjNdLCBbMTAuODU1NTI1LCA0NS44Nzc4NjhdLCBbMTAuODU2MzMzLCA0NS44Nzc1MjhdLCBbMTAuODU2Mzk3LCA0NS44NzczMjZdLCBbMTAuODU2NTcsIDQ1Ljg3NzE1N10sIFsxMC44NTY3NzksIDQ1Ljg3Njk5MV0sIFsxMC44NTY5MDQsIDQ1Ljg3Njg3M10sIFsxMC44NTczNTEsIDQ1Ljg3Njc3Nl0sIFsxMC44NTc2MzcsIDQ1Ljg3NjY2NF0sIFsxMC44NTc4MzUsIDQ1Ljg3NjYyM10sIFsxMC44NTgwMTIsIDQ1Ljg3NjYwNV0sIFsxMC44NTgyMDQsIDQ1Ljg3NjUzNF0sIFsxMC44NTg0MDksIDQ1Ljg3NjM2N10sIFsxMC44NTg3MzEsIDQ1Ljg3NjI2NV0sIFsxMC44NTkxMjIsIDQ1Ljg3NjMxMl0sIFsxMC44NTk1NjksIDQ1Ljg3NjM3OF0sIFsxMC44NjA5NDcsIDQ1Ljg3NjY1Ml0sIFsxMC44NjEyMzEsIDQ1Ljg3NjYyNl0sIFsxMC44NjE0ODIsIDQ1Ljg3NjU3M10sIFsxMC44NjE3MzEsIDQ1Ljg3NjU2OF0sIFsxMC44NjIwOTYsIDQ1Ljg3NjY1MV0sIFsxMC44NjI0NiwgNDUuODc2N10sIFsxMC44NjI3MjksIDQ1Ljg3NjY4MV0sIFsxMC44NjI5NTIsIDQ1Ljg3NjcyMl0sIFsxMC44NjMzMDEsIDQ1Ljg3Njc5Nl0sIFsxMC44NjM1ODQsIDQ1Ljg3Njg1OV0sIFsxMC44NjQwMzIsIDQ1Ljg3NzAwNl0sIFsxMC44NjQyOTIsIDQ1Ljg3NzAzNV0sIFsxMC44NjQ1NDgsIDQ1Ljg3NzA2NF0sIFsxMC44NjQ3NTcsIDQ1Ljg3NzA0NF0sIFsxMC44NjQ5NjIsIDQ1Ljg3N10sIFsxMC44NjU3MywgNDUuODc2NjQ5XSwgWzEwLjg2NjAxOCwgNDUuODc2NDY1XSwgWzEwLjg2NjIxNCwgNDUuODc2MzE3XSwgWzEwLjg2NjM0NCwgNDUuODc2MDk4XSwgWzEwLjg2NjQ0MiwgNDUuODc1OTUzXSwgWzEwLjg2NjY0MSwgNDUuODc1Nzc1XSwgWzEwLjg2NjgxOSwgNDUuODc1NTM1XSwgWzEwLjg2Njk2OSwgNDUuODc1MzM3XSwgWzEwLjg2Njk5NSwgNDUuODc1MTgzXSwgWzEwLjg2NjkzMiwgNDUuODc1MDAzXSwgWzEwLjg2NjgxMywgNDUuODc0ODA4XSwgWzEwLjg2Njg2MiwgNDUuODc0NTIxXSwgWzEwLjg2NjgzMywgNDUuODc0MzgyXSwgWzEwLjg2NjYwNCwgNDUuODc0MjQ5XSwgWzEwLjg2NjM4MywgNDUuODc0MTQyXSwgWzEwLjg2NjI2OSwgNDUuODc0MDQzXSwgWzEwLjg2NjI0OCwgNDUuODczOTU2XSwgWzEwLjg2NjExNSwgNDUuODczODg2XSwgWzEwLjg2NTk1NSwgNDUuODczOTExXSwgWzEwLjg2NTc5NywgNDUuODc0MDI3XSwgWzEwLjg2NTYyNCwgNDUuODczOTA1XSwgWzEwLjg2NTczNiwgNDUuODczNzZdLCBbMTAuODY1OTE3LCA0NS44NzM2NjddLCBbMTAuODY2MjE5LCA0NS44NzM1NjNdLCBbMTAuODY2NDU3LCA0NS44NzM0OTRdLCBbMTAuODY2NTA5LCA0NS44NzM0NjhdLCBbMTAuODY2NTE1LCA0NS44NzM0MzVdLCBbMTAuODY2NTk1LCA0NS44NzMwMTldLCBbMTAuODY2NjU4LCA0NS44NzI2ODldLCBbMTAuODY2NjU3LCA0NS44NzI2NjddLCBbMTAuODY2NjUyLCA0NS44NzI1NjVdLCBbMTAuODY2NjI3LCA0NS44NzI0NThdLCBbMTAuODY2NjE5LCA0NS44NzIzODddLCBbMTAuODY2OTUxLCA0NS44NzI0MjldLCBbMTAuODY3MDg0LCA0NS44NzI1MjldLCBbMTAuODY3MTU3LCA0NS44NzI1NjddLCBbMTAuODY3MjU4LCA0NS44NzI1NTddLCBbMTAuODY3MjgxLCA0NS44NzI0MjhdLCBbMTAuODY3Mzg1LCA0NS44NzIzN10sIFsxMC44Njc1MjYsIDQ1Ljg3MjM5OF0sIFsxMC44Njc3OTYsIDQ1Ljg3MjQ2Nl0sIFsxMC44NjgxODYsIDQ1Ljg3MjQ4NV0sIFsxMC44Njg1NDEsIDQ1Ljg3MjQwNl0sIFsxMC44Njg4MjUsIDQ1Ljg3MjM0M10sIFsxMC44NjkwODUsIDQ1Ljg3MjI0NF0sIFsxMC44Njk0NzUsIDQ1Ljg3MjEyOF0sIFsxMC44Njk3ODUsIDQ1Ljg3MTk1MV0sIFsxMC44NzAxMDcsIDQ1Ljg3MTY4XSwgWzEwLjg3MDM1LCA0NS44NzE0MjFdLCBbMTAuODcwNzE3LCA0NS44NzExMzhdLCBbMTAuODcwOTU0LCA0NS44NzA5MjNdLCBbMTAuODcxMDY2LCA0NS44NzA3OTRdLCBbMTAuODcxMjA2LCA0NS44NzA3MzVdLCBbMTAuODcxNjA3LCA0NS44NzA5NjRdLCBbMTAuODcxNzc5LCA0NS44NzA4ODldLCBbMTAuODcxNjc4LCA0NS44NzA3OTJdLCBbMTAuODcxNjQ2LCA0NS44NzA2NTNdLCBbMTAuODcxMDI2LCA0NS44NzA2MTFdLCBbMTAuODcxMDU3LCA0NS44NzA1NDRdLCBbMTAuODcxNjI1LCA0NS44NzA1NzddLCBbMTAuODcyMjIzLCA0NS44NzAyNTNdLCBbMTAuODcyMTk1LCA0NS44NzAxNzNdLCBbMTAuODcyMjc2LCA0NS44NzAxMzFdLCBbMTAuODcyMzQ2LCA0NS44NzAxODNdLCBbMTAuODcyNjY5LCA0NS44Njk5NjddLCBbMTAuODcyNjQ3LCA0NS44Njk4OTRdLCBbMTAuODcyODI2LCA0NS44Njk4MTJdLCBbMTAuODcyODU1LCA0NS44Njk4MzJdLCBbMTAuODc0MTk5LCA0NS44NjkxNDFdLCBbMTAuODc0MTUyLCA0NS44NjkwNzVdLCBbMTAuODc0MzExLCA0NS44NjkwMTZdLCBbMTAuODc0MjY2LCA0NS44Njg5MTZdLCBbMTAuODc0NTE2LCA0NS44Njg4MjZdLCBbMTAuODc0NTM3LCA0NS44Njg4NzRdLCBbMTAuODc0Njc4LCA0NS44Njg4ODJdLCBbMTAuODc0OTI0LCA0NS44Njg3ODFdLCBbMTAuODc1Mzk4LCA0NS44Njg1OV0sIFsxMC44NzU4NjksIDQ1Ljg2ODcxOF0sIFsxMC44NzYxMDcsIDQ1Ljg2ODY1OF0sIFsxMC44NzYyMzMsIDQ1Ljg2ODYxNl0sIFsxMC44NzYyOTgsIDQ1Ljg2ODU2OV0sIFsxMC44NzYzNzksIDQ1Ljg2ODUzNF0sIFsxMC44NzYzOTUsIDQ1Ljg2ODY1MV0sIFsxMC44NzYzMDEsIDQ1Ljg2ODcwN10sIFsxMC44NzYyMjQsIDQ1Ljg2ODc2Ml0sIFsxMC44NzYwNDMsIDQ1Ljg2ODkwNl0sIFsxMC44NzYxNjIsIDQ1Ljg2OTA4MV0sIFsxMC44NzY2MTEsIDQ1Ljg2ODUxNl0sIFsxMC44NzY1MDYsIDQ1Ljg2ODQ5Nl0sIFsxMC44NzY0NjYsIDQ1Ljg2ODQ3Nl0sIFsxMC44NzY0NjIsIDQ1Ljg2ODQzN10sIFsxMC44NzY1OTgsIDQ1Ljg2ODM1M10sIFsxMC44NzY2MDcsIDQ1Ljg2ODMyNl0sIFsxMC44NzY2OTYsIDQ1Ljg2ODIxM10sIFsxMC44NzY3NCwgNDUuODY4MTE5XSwgWzEwLjg3Njc5OSwgNDUuODY3OTc4XSwgWzEwLjg3NjgzNSwgNDUuODY3ODI3XSwgWzEwLjg3Njg4NCwgNDUuODY3NjcxXSwgWzEwLjg3Njg4NSwgNDUuODY3NTczXSwgWzEwLjg3Njg4OCwgNDUuODY3NDMxXSwgWzEwLjg3Njg4NiwgNDUuODY3MzMyXSwgWzEwLjg3Njg4LCA0NS44NjcyMjVdLCBbMTAuODc2NDY4LCA0NS44NjY0OTVdLCBbMTAuODc2MTYsIDQ1Ljg2NjE3Nl0sIFsxMC44NzU5MTYsIDQ1Ljg2NTgwM10sIFsxMC44NzU3MSwgNDUuODY1NTJdLCBbMTAuODc1NTM4LCA0NS44NjUzMDRdLCBbMTAuODc1NDIsIDQ1Ljg2NTEzNl0sIFsxMC44NzUzMzIsIDQ1Ljg2NV0sIFsxMC44NzUyODQsIDQ1Ljg2NDg5NV0sIFsxMC44NzUyMjcsIDQ1Ljg2NDcyXSwgWzEwLjg3NTIxMSwgNDUuODY0NTU3XSwgWzEwLjg3NTIyMiwgNDUuODY0MzQ5XSwgWzEwLjg3NTI3NCwgNDUuODY0MTc5XSwgWzEwLjg3NTM3MywgNDUuODY0MDM2XSwgWzEwLjg3NTQ5NCwgNDUuODYzOV0sIFsxMC44NzU2NTgsIDQ1Ljg2Mzc2Nl0sIFsxMC44NzU3NjYsIDQ1Ljg2MzYyN10sIFsxMC44NzU4NTMsIDQ1Ljg2MzUzNV0sIFsxMC44NzYxMTMsIDQ1Ljg2MzAzNl0sIFsxMC44NzYxMywgNDUuODYyOTA1XSwgWzEwLjg3NjE2NywgNDUuODYyODE2XSwgWzEwLjg3NjE2NSwgNDUuODYyNzE5XSwgWzEwLjg3NTQwNiwgNDUuODYyNTc3XSwgWzEwLjg3NTQyMSwgNDUuODYyNjc1XSwgWzEwLjg3NTM1OSwgNDUuODYyNjY0XSwgWzEwLjg3NTI4NSwgNDUuODYyNTk2XSwgWzEwLjg3NTIyMywgNDUuODYyNDQ0XSwgWzEwLjg3NTE1NiwgNDUuODYyMzc2XSwgWzEwLjg3NTEyNSwgNDUuODYyMTc5XSwgWzEwLjg3NTE5OSwgNDUuODYxOTc5XSwgWzEwLjg3NTM2NCwgNDUuODYxODkxXSwgWzEwLjg3NTUyNiwgNDUuODYxNzkzXSwgWzEwLjg3NTU4NywgNDUuODYxNTk4XSwgWzEwLjg3NTU5OCwgNDUuODYxNDg2XSwgWzEwLjg3NTY0NCwgNDUuODYxMzQxXSwgWzEwLjg3NTcyMywgNDUuODYxMjQ0XSwgWzEwLjg3NTc3MSwgNDUuODYxMTczXSwgWzEwLjg3NTc1MywgNDUuODYxMDg4XSwgWzEwLjg3NTI4NCwgNDUuODYwNzg0XSwgWzEwLjg3NDg5OSwgNDUuODYwNjY3XSwgWzEwLjg3NDcyMiwgNDUuODYwNTA5XSwgWzEwLjg3NDUxNSwgNDUuODYwMzUxXSwgWzEwLjg3NDI1NiwgNDUuODYwMTQ4XSwgWzEwLjg3NDA1MiwgNDUuODU5OTY3XSwgWzEwLjg3Mzg1MiwgNDUuODU5ODA1XSwgWzEwLjg3MzU4NiwgNDUuODU5NjI3XSwgWzEwLjg3MzM1LCA0NS44NTk0NDldLCBbMTAuODcyOTkyLCA0NS44NTkyNThdLCBbMTAuODcyNjc5LCA0NS44NTkwMzVdLCBbMTAuODcyMzEsIDQ1Ljg1ODc2OV0sIFsxMC44NzIwMTEsIDQ1Ljg1ODU4XSwgWzEwLjg3MTU3MywgNDUuODU4MzA4XSwgWzEwLjg3MTM5OCwgNDUuODU4MDgzXSwgWzEwLjg3MTA3NSwgNDUuODU3ODA1XSwgWzEwLjg3MDgyOSwgNDUuODU3NTAxXSwgWzEwLjg3MDY0OSwgNDUuODU3MTk0XSwgWzEwLjg3MDUsIDQ1Ljg1Njk2N10sIFsxMC44NzAzNTEsIDQ1Ljg1Njc1Ml0sIFsxMC44NzAyNSwgNDUuODU2NjM2XSwgWzEwLjg2OTk2OSwgNDUuODU2MzU1XSwgWzEwLjg2OTY3NiwgNDUuODU2MTI1XSwgWzEwLjg2OTQ2NSwgNDUuODU1ODk0XSwgWzEwLjg2OTI2OSwgNDUuODU1NjRdLCBbMTAuODY5MDQ5LCA0NS44NTU0NTVdLCBbMTAuODY4ODU2LCA0NS44NTUzMDhdLCBbMTAuODY4NTI5LCA0NS44NTUwMTldLCBbMTAuODY4MjYyLCA0NS44NTQ3NjNdLCBbMTAuODY4MDc3LCA0NS44NTQ1NjhdLCBbMTAuODY3OTMzLCA0NS44NTQzOTZdLCBbMTAuODY3NzI2LCA0NS44NTQyMjldLCBbMTAuODY3NDc3LCA0NS44NTQwODFdLCBbMTAuODY3MjM0LCA0NS44NTM4NzVdLCBbMTAuODY2OTk5LCA0NS44NTM2MjFdLCBbMTAuODY2ODE3LCA0NS44NTMzNl0sIFsxMC44NjY1ODksIDQ1Ljg1MzEyOV0sIFsxMC44NjYzMTcsIDQ1Ljg1Mjk2M10sIFsxMC44NjYwNDMsIDQ1Ljg1Mjg1MV0sIFsxMC44NjU1NzksIDQ1Ljg1MjcxXSwgWzEwLjg2NTM0NiwgNDUuODUyNTU5XSwgWzEwLjg2NTE3LCA0NS44NTI0MzFdLCBbMTAuODY0OTkxLCA0NS44NTIzMjNdLCBbMTAuODY0NzY4LCA0NS44NTIxNTNdLCBbMTAuODY0Mzk0LCA0NS44NTE4NDJdLCBbMTAuODY0MTQzLCA0NS44NTE1NzJdLCBbMTAuODYzODQyLCA0NS44NTEyNzFdLCBbMTAuODYzNjYsIDQ1Ljg1MTA1Nl0sIFsxMC44NjM0NzYsIDQ1Ljg1MDg3Ml0sIFsxMC44NjMzNTksIDQ1Ljg1MDcyM10sIFsxMC44NjMyNDcsIDQ1Ljg1MDU0OF0sIFsxMC44NjMxNzYsIDQ1Ljg1MDQ0MV0sIFsxMC44NjMwMjMsIDQ1Ljg1MDE4Nl0sIFsxMC44NjI4NzUsIDQ1Ljg0OTg2NV0sIFsxMC44NjI3ODYsIDQ1Ljg0OTY5OV0sIFsxMC44NjI3MDIsIDQ1Ljg0OTU3XSwgWzEwLjg2MjU4MSwgNDUuODQ5NDM0XSwgWzEwLjg2MjM3OCwgNDUuODQ5MjY5XSwgWzEwLjg2MjMxNCwgNDUuODQ5MjE1XSwgWzEwLjg2MjI5OCwgNDUuODQ5MTg2XSwgWzEwLjg2MjE3NiwgNDUuODQ5MTA3XSwgWzEwLjg2MjE4MSwgNDUuODQ5MTA3XSwgWzEwLjg2MjE3MSwgNDUuODQ5MDkzXSwgWzEwLjg2MjE0OCwgNDUuODQ5MDU3XSwgWzEwLjg2MjE1OCwgNDUuODQ5MDJdLCBbMTAuODYyMjY1LCA0NS44NDkwMTVdLCBbMTAuODYyMzMxLCA0NS44NDkwMDRdLCBbMTAuODYyNDEyLCA0NS44NDg5NzNdLCBbMTAuODYyNDMxLCA0NS44NDg5NDhdLCBbMTAuODYyNDIxLCA0NS44NDg4ODddLCBbMTAuODYyNDMsIDQ1Ljg0ODg1XSwgWzEwLjg2MjQ2OSwgNDUuODQ4ODI3XSwgWzEwLjg2MjU2NywgNDUuODQ4ODEzXSwgWzEwLjg2MjY2MiwgNDUuODQ4ODA1XSwgWzEwLjg2Mjk4OCwgNDUuODQ4Njg5XSwgWzEwLjg2MzEwMiwgNDUuODQ4NjI1XSwgWzEwLjg2MzIzMiwgNDUuODQ4NTQyXSwgWzEwLjg2MzMxMywgNDUuODQ4NDc1XSwgWzEwLjg2MzM3NCwgNDUuODQ4NDA0XSwgWzEwLjg2MzQzMiwgNDUuODQ4MzE3XSwgWzEwLjg2MzQ5MywgNDUuODQ4MjIzXSwgWzEwLjg2MzU0MSwgNDUuODQ4MTVdLCBbMTAuODYzNTg2LCA0NS44NDgwNjNdLCBbMTAuODYzNjA1LCA0NS44NDgwMDNdLCBbMTAuODYzNjE0LCA0NS44NDc5NDhdLCBbMTAuODYzNTcxLCA0NS44NDc4NjJdLCBbMTAuODYzNTExLCA0NS44NDc3OTFdLCBbMTAuODYzNDM1LCA0NS44NDc3M10sIFsxMC44NjMzNDMsIDQ1Ljg0NzY1N10sIFsxMC44NjMyNzcsIDQ1Ljg0NzU5OF0sIFsxMC44NjMyMTcsIDQ1Ljg0NzUxOF0sIFsxMC44NjMyMDMsIDQ1Ljg0NzQ0OF0sIFsxMC44NjMyMDksIDQ1Ljg0NzM3NV0sIFsxMC44NjMyMTEsIDQ1Ljg0NzI4MV0sIFsxMC44NjMyMTYsIDQ1Ljg0NzE1NV0sIFsxMC44NjMxNDYsIDQ1Ljg0Njc0NF0sIFsxMC44NjMwOTksIDQ1Ljg0NjY2Ml0sIFsxMC44NjMwMzksIDQ1Ljg0NjU1OF0sIFsxMC44NjI5NzMsIDQ1Ljg0NjQ1NV0sIFsxMC44NjI5MjMsIDQ1Ljg0NjM4NV0sIFsxMC44NjI4NDMsIDQ1Ljg0NjI2Nl0sIFsxMC44NjI3NTMsIDQ1Ljg0NjE1NV0sIFsxMC44NjI2NTcsIDQ1Ljg0NjAzOV0sIFsxMC44NjI1ODgsIDQ1Ljg0NTk2MV0sIFsxMC44NjI0MzIsIDQ1Ljg0NTc4Ml0sIFsxMC44NjI0MTUsIDQ1Ljg0NTcxM10sIFsxMC44NjI0MywgNDUuODQ1NjM4XSwgWzEwLjg2MjQ2MiwgNDUuODQ1NTUzXSwgWzEwLjg2MjQ5NywgNDUuODQ1NDY2XSwgWzEwLjg2MjUzNSwgNDUuODQ1MzUyXSwgWzEwLjg2MjU4NiwgNDUuODQ1MjFdLCBbMTAuODYyNjE1LCA0NS44NDUxNTldLCBbMTAuODYyNjQ0LCA0NS44NDUxMjddLCBbMTAuODYyNzAyLCA0NS44NDUwOV0sIFsxMC44NjI3NTQsIDQ1Ljg0NTA3OV0sIFsxMC44NjI4MywgNDUuODQ1MDc0XSwgWzEwLjg2Mjg5OCwgNDUuODQ1MDcxXSwgWzEwLjg2Mjk4MywgNDUuODQ1MDY4XSwgWzEwLjg2MzA2NSwgNDUuODQ1MDYzXSwgWzEwLjg2MzE1LCA0NS44NDUwNjVdLCBbMTAuODYzMjEyLCA0NS44NDUwNjNdLCBbMTAuODYzMjU4LCA0NS44NDUwNDldLCBbMTAuODYzMzE2LCA0NS44NDUwMV0sIFsxMC44NjMzNDUsIDQ1Ljg0NDk2NF0sIFsxMC44NjMzNzMsIDQ1Ljg0NDg2NV0sIFsxMC44NjM0MTUsIDQ1Ljg0NDc0NF0sIFsxMC44NjM0NzUsIDQ1Ljg0NDU3N10sIFsxMC44NjM0OTMsIDQ1Ljg0NDQ5Ml0sIFsxMC44NjM0NTksIDQ1Ljg0NDM3Nl0sIFsxMC44NjMzODYsIDQ1Ljg0NDI1NV0sIFsxMC44NjMyNjcsIDQ1Ljg0NDEyNl0sIFsxMC44NjMxNDgsIDQ1Ljg0NDAwOF0sIFsxMC44NjMwMjIsIDQ1Ljg0Mzg5Ml0sIFsxMC44NjI4ODQsIDQ1Ljg0Mzc2N10sIFsxMC44NjI3NDgsIDQ1Ljg0MzYzM10sIFsxMC44NjI2MzksIDQ1Ljg0MzUzNV0sIFsxMC44NjI1NTYsIDQ1Ljg0MzQ0XSwgWzEwLjg2MjQ5LCA0NS44NDMzMzNdLCBbMTAuODYyNDU5LCA0NS44NDMyMjVdLCBbMTAuODYyNDU0LCA0NS44NDMxMl0sIFsxMC44NjI0NiwgNDUuODQzMDA2XSwgWzEwLjg2MjQ2OSwgNDUuODQyOTE5XSwgWzEwLjg2MjQ1NCwgNDUuODQyNzk4XSwgWzEwLjg2MjM4MSwgNDUuODQyNzAzXSwgWzEwLjg2MjMwMSwgNDUuODQyNTY2XSwgWzEwLjg2MjIxOCwgNDUuODQyNDM0XSwgWzEwLjg2MjE2MSwgNDUuODQyMzUyXSwgWzEwLjg2MjA1MiwgNDUuODQyMTg4XSwgWzEwLjg2MTkzNiwgNDUuODQyMDQ5XSwgWzEwLjg2MTgxMywgNDUuODQxOTE3XSwgWzEwLjg2MTcwNCwgNDUuODQxNzk3XSwgWzEwLjg2MTYxNCwgNDUuODQxNjkyXSwgWzEwLjg2MTU2MSwgNDUuODQxNjAxXSwgWzEwLjg2MTQ5NCwgNDUuODQxNDc0XSwgWzEwLjg2MTQyNCwgNDUuODQxMzQ4XSwgWzEwLjg2MTM2MSwgNDUuODQxMjQ4XSwgWzEwLjg2MTI4MSwgNDUuODQxMTUzXSwgWzEwLjg2MTE4OSwgNDUuODQxMDYyXSwgWzEwLjg2MTExNiwgNDUuODQwOThdLCBbMTAuODYxMDIsIDQ1Ljg0MDg3MV0sIFsxMC44NjA5MzMsIDQ1Ljg0MDc2MV0sIFsxMC44NjA4NDQsIDQ1Ljg0MDYyM10sIFsxMC44NjA3NywgNDUuODQwNDc5XSwgWzEwLjg2MDczLCA0NS44NDAzNjNdLCBbMTAuODYwNjYyLCA0NS44NDAxNzZdLCBbMTAuODYwNjM1LCA0NS44NDAxMDNdLCBbMTAuODYwNjE1LCA0NS44NDAwNTVdLCBbMTAuODYwNTkyLCA0NS44NDAwMDddLCBbMTAuODYwNTY4LCA0NS44Mzk5NjhdLCBbMTAuODYwNTM4LCA0NS44Mzk5MzRdLCBbMTAuODYwNDM3LCA0NS44Mzk4OTZdLCBbMTAuODYwMzg4LCA0NS44Mzk4OThdLCBbMTAuODYwMzMyLCA0NS44Mzk5MV0sIFsxMC44NjAyOCwgNDUuODM5OTI2XSwgWzEwLjg2MDIzMSwgNDUuODM5OTQzXSwgWzEwLjg2MDE4NSwgNDUuODM5OTUyXSwgWzEwLjg2MDE0NiwgNDUuODM5OTU5XSwgWzEwLjg2MDEwNywgNDUuODM5OTYyXSwgWzEwLjg2MDA0OCwgNDUuODM5OTU1XSwgWzEwLjg1OTk5MiwgNDUuODM5OTM3XSwgWzEwLjg1OTkyMywgNDUuODM5ODkyXSwgWzEwLjg1OTg4LCA0NS44Mzk4NDldLCBbMTAuODU5ODQsIDQ1LjgzOTc4NV0sIFsxMC44NTk4MDMsIDQ1LjgzOTcwMV0sIFsxMC44NTk3NjksIDQ1LjgzOTU4Ml0sIFsxMC44NTk3NDUsIDQ1LjgzOTQ4Nl0sIFsxMC44NTk3MzQsIDQ1LjgzOTM5NV0sIFsxMC44NTk3MiwgNDUuODM5Mjk3XSwgWzEwLjg1OTcwMywgNDUuODM5MjI4XSwgWzEwLjg1OTY4MywgNDUuODM5MTQ2XSwgWzEwLjg1OTY2MiwgNDUuODM5MDQ4XSwgWzEwLjg1OTY1NSwgNDUuODM4OTk4XSwgWzEwLjg1OTYzMSwgNDUuODM4ODg4XSwgWzEwLjg1OTYzOCwgNDUuODM4Njg5XSwgWzEwLjg1OTY0NywgNDUuODM4NTc3XSwgWzEwLjg1OTY0NiwgNDUuODM4NDYzXSwgWzEwLjg1OTYwNywgNDUuODM4MjQ0XSwgWzEwLjg1OTU2LCA0NS44MzgwOThdLCBbMTAuODU5NTIyLCA0NS44Mzc5NzVdLCBbMTAuODU5NDYyLCA0NS44Mzc4MjJdLCBbMTAuODU5NDI1LCA0NS44Mzc3MDZdLCBbMTAuODU5MzkxLCA0NS44Mzc2MV0sIFsxMC44NTkzMjQsIDQ1LjgzNzQ3MV0sIFsxMC44NTkyNjcsIDQ1LjgzNzM5MV0sIFsxMC44NTkyMDgsIDQ1LjgzNzMwNV0sIFsxMC44NTkxMzUsIDQ1LjgzNzIxNl0sIFsxMC44NTkwNDgsIDQ1LjgzNzExOF0sIFsxMC44NTg5NjMsIDQ1LjgzNzA0M10sIFsxMC44NTg4NzcsIDQ1LjgzNjk2NF0sIFsxMC44NTg3NDcsIDQ1LjgzNjc2Nl0sIFsxMC44NTg3MzIsIDQ1LjgzNjYzNl0sIFsxMC44NTg3NDEsIDQ1LjgzNjUzNV0sIFsxMC44NTg3MDYsIDQ1LjgzNjI4OV0sIFsxMC44NTg2NjgsIDQ1LjgzNjE1NF0sIFsxMC44NTg2NDEsIDQ1LjgzNjAzMV0sIFsxMC44NTg2MTYsIDQ1LjgzNTkxXSwgWzEwLjg1ODU5NiwgNDUuODM1NzkzXSwgWzEwLjg1ODU3MSwgNDUuODM1Njg4XSwgWzEwLjg1ODUyNCwgNDUuODM1NTJdLCBbMTAuODU4NTA0LCA0NS44MzU0NzRdLCBbMTAuODU4NDg2LCA0NS44MzUzNjJdLCBbMTAuODU4NTA1LCA0NS44MzUzMTRdLCBbMTAuODU4NTQ0LCA0NS44MzUyNjhdLCBbMTAuODU4NjEyLCA0NS44MzUyMjJdLCBbMTAuODU4Njc3LCA0NS44MzUxOTJdLCBbMTAuODU4NzU1LCA0NS44MzUxNjldLCBbMTAuODU4ODE0LCA0NS44MzUxNjZdLCBbMTAuODU4ODczLCA0NS44MzUxNjRdLCBbMTAuODU4OTMyLCA0NS44MzUxNTldLCBbMTAuODU4OTk3LCA0NS44MzUxNDVdLCBbMTAuODU5MDYyLCA0NS44MzUxMjJdLCBbMTAuODU5MTI0LCA0NS44MzUwOTRdLCBbMTAuODU5MTk5LCA0NS44MzUwNDZdLCBbMTAuODU5MjkzLCA0NS44MzQ5OV0sIFsxMC44NTkzNTgsIDQ1LjgzNDk2OV0sIFsxMC44NTk0MSwgNDUuODM0OTU4XSwgWzEwLjg1OTQ0OSwgNDUuODM0OTQ2XSwgWzEwLjg1OTQ5OCwgNDUuODM0OTE0XSwgWzEwLjg1OTUzNywgNDUuODM0ODYzXSwgWzEwLjg1OTU3MiwgNDUuODM0Nzk3XSwgWzEwLjg1OTU4MSwgNDUuODM0NzE1XSwgWzEwLjg1OTU3LCA0NS44MzQ2MjFdLCBbMTAuODU5NTQzLCA0NS44MzQ1MzJdLCBbMTAuODU5NTMyLCA0NS44MzQ0NzddLCBbMTAuODU5NTY3LCA0NS44MzQzN10sIFsxMC44NTk2MDksIDQ1LjgzNDMzMV0sIFsxMC44NTk2NDgsIDQ1LjgzNDI4OV0sIFsxMC44NTk2OTcsIDQ1LjgzNDI2NF0sIFsxMC44NTk3MzYsIDQ1LjgzNDI1Ml0sIFsxMC44NTk4MTQsIDQ1LjgzNDIyOV0sIFsxMC44NTk4ODksIDQ1LjgzNDE4NV0sIFsxMC44NTk5NjcsIDQ1LjgzNDEyNV0sIFsxMC44NjAwMzEsIDQ1LjgzNDA2MV0sIFsxMC44NjAwNjQsIDQ1LjgzNDAyNF0sIFsxMC44NjAwODksIDQ1LjgzMzk4M10sIFsxMC44NjAxMDUsIDQ1LjgzMzkxNF0sIFsxMC44NjAxMDQsIDQ1LjgzMzg2Ml0sIFsxMC44NjAwNzgsIDQ1LjgzMzgwM10sIFsxMC44NjAwNDQsIDQ1LjgzMzc1M10sIFsxMC44NjAwMDQsIDQ1LjgzMzY4NF0sIFsxMC44NTk5NjgsIDQ1LjgzMzYxNF0sIFsxMC44NTk5MzcsIDQ1LjgzMzUzNl0sIFsxMC44NTk5MDQsIDQ1LjgzMzQ1NF0sIFsxMC44NTk4MywgNDUuODMzMjk5XSwgWzEwLjg1OTc5OSwgNDUuODMzMTc2XSwgWzEwLjg1OTc2NSwgNDUuODMzMDQ2XSwgWzEwLjg1OTcyMSwgNDUuODMyOTA3XSwgWzEwLjg1OTY4MSwgNDUuODMyODA0XSwgWzEwLjg1OTYyNywgNDUuODMyNjc5XSwgWzEwLjg1OTU3MywgNDUuODMyNTQ5XSwgWzEwLjg1OTU0MiwgNDUuODMyNDQ0XSwgWzEwLjg1OTQ5NSwgNDUuODMyMzM0XSwgWzEwLjg1OTQxOSwgNDUuODMyMTg5XSwgWzEwLjg1OTM2MiwgNDUuODMyMDkzXSwgWzEwLjg1OTI4OSwgNDUuODMyMDAyXSwgWzEwLjg1OTIyOSwgNDUuODMxOTI5XSwgWzEwLjg1OTE0MywgNDUuODMxODI5XSwgWzEwLjg1OTA2NCwgNDUuODMxNzI0XSwgWzEwLjg1OTAxNywgNDUuODMxNjU2XSwgWzEwLjg1ODk2NywgNDUuODMxNTldLCBbMTAuODU4ODg0LCA0NS44MzE0OV0sIFsxMC44NTg3ODgsIDQ1LjgzMTM3NF0sIFsxMC44NTg2NjksIDQ1LjgzMTIxOV0sIFsxMC44NTg1NzMsIDQ1LjgzMTA4M10sIFsxMC44NTg0ODksIDQ1LjgzMDk0Nl0sIFsxMC44NTg0MTksIDQ1LjgzMDgwOV0sIFsxMC44NTgzMjksIDQ1LjgzMDYyXSwgWzEwLjg1ODIzMiwgNDUuODMwNDU5XSwgWzEwLjg1ODE0NSwgNDUuODMwMzA4XSwgWzEwLjg1ODA2MiwgNDUuODMwMTZdLCBbMTAuODU3OTUyLCA0NS44Mjk5OTRdLCBbMTAuODU3ODY5LCA0NS44Mjk4NDRdLCBbMTAuODU3Nzg2LCA0NS44Mjk2OThdLCBbMTAuODU3NzAyLCA0NS44Mjk1NjZdLCBbMTAuODU3NTkzLCA0NS44Mjk0MTZdLCBbMTAuODU3NTI2LCA0NS44MjkzMDJdLCBbMTAuODU3NDQ5LCA0NS44MjkxNF0sIFsxMC44NTczODUsIDQ1LjgyOTAwNl0sIFsxMC44NTcxOTYsIDQ1LjgyODcyMV0sIFsxMC44NTcwMSwgNDUuODI4NDc1XSwgWzEwLjg1NjkyNywgNDUuODI4MzUyXSwgWzEwLjg1NjgxMSwgNDUuODI4MTk4XSwgWzEwLjg1NjY5NCwgNDUuODI3OTgxXSwgWzEwLjg1NjY2MSwgNDUuODI3OTEzXSwgWzEwLjg1NjYxMywgNDUuODI3NzkyXSwgWzEwLjg1NjU5MywgNDUuODI3NzMzXSwgWzEwLjg1NjU1NSwgNDUuODI3NTYxXSwgWzEwLjg1NjU3LCA0NS44Mjc0NTJdLCBbMTAuODU2NTkzLCA0NS44Mjc0MDZdLCBbMTAuODU2NjQxLCA0NS44MjczNThdLCBbMTAuODU2NzA5LCA0NS44MjczXSwgWzEwLjg1NjgzNiwgNDUuODI3MTk0XSwgWzEwLjg1Njk0MiwgNDUuODI3MDgyXSwgWzEwLjg1NzAwOCwgNDUuODI3MDE0XSwgWzEwLjgzMjM0MSwgNDUuODM1NDc3XSwgWzEwLjgyNTg1MywgNDUuODM3NjNdXV1dLCAidHlwZSI6ICJNdWx0aVBvbHlnb24ifSwgImlkIjogIjciLCAicHJvcGVydGllcyI6IHsiZGVzY3JpemlvbmVfb3JpZ2luZSI6ICJOb24gY29kaWZpY2F0byIsICJnbWxfaWQiOiAiSUQuQUNRVUVGSVNJQ0hFLlNQRUNDSEkuQUNRVUEuNTQxODMiLCAibmFzcCI6IDAsICJuYXR1cmFfc3BlY2NoaW9fYWNxdWEiOiAiTm9uIGNvZGlmaWNhdG8iLCAibm9tZSI6ICJMQUdPIERJIEdBUkRBIiwgIm9yaWdpbmUiOiAwfSwgInR5cGUiOiAiRmVhdHVyZSJ9LCB7Imdlb21ldHJ5IjogeyJjb29yZGluYXRlcyI6IFtbW1sxMC45Nzc0MTcsIDQ2LjA1OTExNF0sIFsxMC45Nzc1ODgsIDQ2LjA1OTEwNF0sIFsxMC45Nzc3NzksIDQ2LjA1OTA5Ml0sIFsxMC45Nzc3NzgsIDQ2LjA1OTA0Nl0sIFsxMC45Nzc3ODMsIDQ2LjA1ODk5Nl0sIFsxMC45Nzc3OTgsIDQ2LjA1ODk1XSwgWzEwLjk3NzgyNCwgNDYuMDU4Nzk0XSwgWzEwLjk3NzgwMywgNDYuMDU4NzQyXSwgWzEwLjk3Nzc3MSwgNDYuMDU4NzAxXSwgWzEwLjk3NzczMiwgNDYuMDU4NjQ5XSwgWzEwLjk3NzU4OCwgNDYuMDU4NTIxXSwgWzEwLjk3NzQ3NywgNDYuMDU4NDUxXSwgWzEwLjk3NzQwNCwgNDYuMDU4MzldLCBbMTAuOTc3MzQ2LCA0Ni4wNTgzMTVdLCBbMTAuOTc3MjYxLCA0Ni4wNTgyMDRdLCBbMTAuOTc3MTgsIDQ2LjA1ODExNl0sIFsxMC45NzcwNTksIDQ2LjA1ODAwMV0sIFsxMC45NzY5MzYsIDQ2LjA1NzkyXSwgWzEwLjk3Njg2MiwgNDYuMDU3ODgyXSwgWzEwLjk3NjgzMiwgNDYuMDU3ODYyXSwgWzEwLjk3Njc3OCwgNDYuMDU3Nzk3XSwgWzEwLjk3Njc0OCwgNDYuMDU3NzU2XSwgWzEwLjk3NjcyNywgNDYuMDU3NzEzXSwgWzEwLjk3NjcxOSwgNDYuMDU3NjUzXSwgWzEwLjk3NjcwMSwgNDYuMDU3NTg3XSwgWzEwLjk3NjY2LCA0Ni4wNTc1MzNdLCBbMTAuOTc2NTUzLCA0Ni4wNTc0NjNdLCBbMTAuOTc2NDAxLCA0Ni4wNTc0MDRdLCBbMTAuOTc2MjY4LCA0Ni4wNTczNV0sIFsxMC45NzYxMzEsIDQ2LjA1NzI3Ml0sIFsxMC45NzYwMTQsIDQ2LjA1NzE4OV0sIFsxMC45NzU5MDQsIDQ2LjA1NzExOV0sIFsxMC45NzU3OTgsIDQ2LjA1NzA3N10sIFsxMC45NzU3MDIsIDQ2LjA1NzAzOV0sIFsxMC45NzU1OSwgNDYuMDU3MDI5XSwgWzEwLjk3NTQ1OSwgNDYuMDU3MDY3XSwgWzEwLjk3NTM0NiwgNDYuMDU3MTE0XSwgWzEwLjk3NTE5OCwgNDYuMDU3MTM3XSwgWzEwLjk3NTA3OSwgNDYuMDU3MTAyXSwgWzEwLjk3NDk3MiwgNDYuMDU3MDM3XSwgWzEwLjk3NDkwMiwgNDYuMDU2OTk2XSwgWzEwLjk3NDQ2NCwgNDYuMDU2ODA1XSwgWzEwLjk3NDMyNCwgNDYuMDU2NzM4XSwgWzEwLjk3NDIyMSwgNDYuMDU2NjkxXSwgWzEwLjk3NDEwMiwgNDYuMDU2NjMzXSwgWzEwLjk3MzkwMywgNDYuMDU2NTY5XSwgWzEwLjk3Mzc3NCwgNDYuMDU2NTE0XSwgWzEwLjk3MzU1NCwgNDYuMDU2NDE0XSwgWzEwLjk3MzMwNSwgNDYuMDU2Mjk4XSwgWzEwLjk3MzAzNiwgNDYuMDU2MTcxXSwgWzEwLjk3MjcxNywgNDYuMDU2MDE3XSwgWzEwLjk3MjQ1MSwgNDYuMDU1ODg1XSwgWzEwLjk3MjE3NCwgNDYuMDU1NzE3XSwgWzEwLjk3MTk3MSwgNDYuMDU1NTk2XSwgWzEwLjk3MTcyNCwgNDYuMDU1NDM0XSwgWzEwLjk3MTUxNywgNDYuMDU1Mjk1XSwgWzEwLjk3MTM3OSwgNDYuMDU1MTcxXSwgWzEwLjk3MTI4NCwgNDYuMDU1MDU4XSwgWzEwLjk3MTE1MiwgNDYuMDU0ODcyXSwgWzEwLjk3MTA0NCwgNDYuMDU0NzQ4XSwgWzEwLjk3MDkxNiwgNDYuMDU0NjU2XSwgWzEwLjk3MDc1NCwgNDYuMDU0NTgyXSwgWzEwLjk3MDYwMSwgNDYuMDU0NTM2XSwgWzEwLjk3MDQ0NiwgNDYuMDU0NDc0XSwgWzEwLjk3MDMxNSwgNDYuMDU0MzkxXSwgWzEwLjk3MDIwNSwgNDYuMDU0MzA4XSwgWzEwLjk3MDA4MSwgNDYuMDU0MjE4XSwgWzEwLjk2OTk1OCwgNDYuMDU0MTQ0XSwgWzEwLjk2OTg0OCwgNDYuMDU0MDc2XSwgWzEwLjk2OTc0NCwgNDYuMDUzOTg0XSwgWzEwLjk2OTY3MywgNDYuMDUzODk4XSwgWzEwLjk2OTYwNSwgNDYuMDUzODI2XSwgWzEwLjk2OTU2MSwgNDYuMDUzNzY1XSwgWzEwLjk2OTU2MywgNDYuMDUzNzIxXSwgWzEwLjk2OTU3NiwgNDYuMDUzNjg0XSwgWzEwLjk2OTYzNCwgNDYuMDUzNjQ5XSwgWzEwLjk2OTc0OCwgNDYuMDUzNjA3XSwgWzEwLjk2OTg2OSwgNDYuMDUzNTc4XSwgWzEwLjk2OTk5NiwgNDYuMDUzNTMxXSwgWzEwLjk3MDE1NiwgNDYuMDUzNDc2XSwgWzEwLjk3MDI1MywgNDYuMDUzNDM2XSwgWzEwLjk3MDMzOCwgNDYuMDUzMzkyXSwgWzEwLjk3MDM3MywgNDYuMDUzMzUzXSwgWzEwLjk3MDM4MiwgNDYuMDUzMzIxXSwgWzEwLjk3MDM3NCwgNDYuMDUzMjY0XSwgWzEwLjk3MDM1MywgNDYuMDUzMjE2XSwgWzEwLjk3MDMxOSwgNDYuMDUzMTU3XSwgWzEwLjk3MDI3NSwgNDYuMDUzMV0sIFsxMC45NzAyMTQsIDQ2LjA1MzAyOF0sIFsxMC45NzAxNDcsIDQ2LjA1Mjk2NV0sIFsxMC45NzAwNDYsIDQ2LjA1Mjg3OV0sIFsxMC45Njk5NDksIDQ2LjA1Mjc5Nl0sIFsxMC45Njk5MDUsIDQ2LjA1Mjc1XSwgWzEwLjk2OTg4NSwgNDYuMDUyNzE5XSwgWzEwLjk2OTg3NCwgNDYuMDUyNjhdLCBbMTAuOTY5ODczLCA0Ni4wNTI2NDFdLCBbMTAuOTY5ODk5LCA0Ni4wNTI2Ml0sIFsxMC45Njk5NTQsIDQ2LjA1MjYxXSwgWzEwLjk3MDA1NiwgNDYuMDUyNjIzXSwgWzEwLjk3MDE2OCwgNDYuMDUyNjM1XSwgWzEwLjk3MDM0OSwgNDYuMDUyNjQ1XSwgWzEwLjk3MDUxNywgNDYuMDUyNjQ3XSwgWzEwLjk3MDcwNSwgNDYuMDUyNjU2XSwgWzEwLjk3MDg5NiwgNDYuMDUyNjc5XSwgWzEwLjk3MTEyNywgNDYuMDUyNzI3XSwgWzEwLjk3MTM0NSwgNDYuMDUyNzc3XSwgWzEwLjk3MTU3LCA0Ni4wNTI4NDddLCBbMTAuOTcxODIyLCA0Ni4wNTI5MjZdLCBbMTAuOTcyMTEsIDQ2LjA1MzAzXSwgWzEwLjk3MjM4MiwgNDYuMDUzMTQ2XSwgWzEwLjk3Mjg3LCA0Ni4wNTMzNDZdLCBbMTAuOTczMDY5LCA0Ni4wNTM0NDRdLCBbMTAuOTczMjc1LCA0Ni4wNTM1MzVdLCBbMTAuOTczNDc4LCA0Ni4wNTM2MTFdLCBbMTAuOTczNjQ5LCA0Ni4wNTM2NTRdLCBbMTAuOTczODE4LCA0Ni4wNTM2OF0sIFsxMC45NzQwMDIsIDQ2LjA1MzY5Nl0sIFsxMC45NzQyMTYsIDQ2LjA1MzcyOF0sIFsxMC45NzQzOTgsIDQ2LjA1Mzc1M10sIFsxMC45NzQ2MjIsIDQ2LjA1Mzc3MV0sIFsxMC45NzQ3MzYsIDQ2LjA1Mzc2NV0sIFsxMC45NzQ4MTUsIDQ2LjA1Mzc1Ml0sIFsxMC45NzQ4NjQsIDQ2LjA1MzczNF0sIFsxMC45NzQ5NTEsIDQ2LjA1MzY3M10sIFsxMC45NzQ5ODIsIDQ2LjA1MzYwOV0sIFsxMC45NzQ5ODEsIDQ2LjA1MzU0OV0sIFsxMC45NzQ5MzcsIDQ2LjA1MzVdLCBbMTAuOTc0ODMsIDQ2LjA1MzQzXSwgWzEwLjk3NDYzNywgNDYuMDUzMzI5XSwgWzEwLjk3NDQzNCwgNDYuMDUzMjE1XSwgWzEwLjk3NDE5NCwgNDYuMDUzMDgzXSwgWzEwLjk3Mzk5MSwgNDYuMDUyOTUxXSwgWzEwLjk3Mzc5NywgNDYuMDUyODI4XSwgWzEwLjk3MzU3NywgNDYuMDUyNjk1XSwgWzEwLjk3MzMwOCwgNDYuMDUyNTU3XSwgWzEwLjk3MzA2OSwgNDYuMDUyNDc1XSwgWzEwLjk3Mjc4NywgNDYuMDUyMzY3XSwgWzEwLjk3MjQ5MywgNDYuMDUyMjc0XSwgWzEwLjk3MjIyMSwgNDYuMDUyMTc1XSwgWzEwLjk3MTk4OCwgNDYuMDUyMDY1XSwgWzEwLjk3MTczNSwgNDYuMDUxOTEzXSwgWzEwLjk3MTQ1OSwgNDYuMDUxNzY1XSwgWzEwLjk3MTE5NiwgNDYuMDUxNjIyXSwgWzEwLjk3MDk3NiwgNDYuMDUxNTExXSwgWzEwLjk3MDc1LCA0Ni4wNTE0MTNdLCBbMTAuOTcwNTU0LCA0Ni4wNTEzMDVdLCBbMTAuOTcwMzk0LCA0Ni4wNTExOTVdLCBbMTAuOTcwMjQxLCA0Ni4wNTExMjJdLCBbMTAuOTcwMDk2LCA0Ni4wNTEwOTRdLCBbMTAuOTY5OTQxLCA0Ni4wNTEwODldLCBbMTAuOTY5Nzk2LCA0Ni4wNTEwODZdLCBbMTAuOTY5Njk0LCA0Ni4wNTEwNzhdLCBbMTAuOTY5NTYyLCA0Ni4wNTEwNV0sIFsxMC45Njk0NTksIDQ2LjA1MDk5OF0sIFsxMC45NjkzNTUsIDQ2LjA1MDkxN10sIFsxMC45NjkxOTUsIDQ2LjA1MDgxMl0sIFsxMC45NjkwMDgsIDQ2LjA1MDY5NV0sIFsxMC45Njg4MzksIDQ2LjA1MDYwMV0sIFsxMC45Njg2NjUsIDQ2LjA1MDUwM10sIFsxMC45Njg1MTksIDQ2LjA1MDQxOF0sIFsxMC45Njg0MTUsIDQ2LjA1MDMzN10sIFsxMC45NjgyOTEsIDQ2LjA1MDI0Ml0sIFsxMC45NjgxODQsIDQ2LjA1MDE2OF0sIFsxMC45NjgwMjQsIDQ2LjA1MDA3NF0sIFsxMC45Njc4NjgsIDQ2LjA1MDAwM10sIFsxMC45Njc3MTUsIDQ2LjA0OTkzNF0sIFsxMC45Njc2MDIsIDQ2LjA0OTg3Nl0sIFsxMC45Njc0NzksIDQ2LjA0OTgwMl0sIFsxMC45NjczNjIsIDQ2LjA0OTcyMV0sIFsxMC45NjcyNTUsIDQ2LjA0OTY1Nl0sIFsxMC45NjcxNzIsIDQ2LjA0OTYwMl0sIFsxMC45NjcxMDIsIDQ2LjA0OTU0MV0sIFsxMC45NjcwNDcsIDQ2LjA0OTQ1XSwgWzEwLjk2Njk3OSwgNDYuMDQ5Mzc4XSwgWzEwLjk2Njg4OSwgNDYuMDQ5MzE1XSwgWzEwLjk2NjcyNiwgNDYuMDQ5MjI4XSwgWzEwLjk2NjY1MywgNDYuMDQ5MTg3XSwgWzEwLjk2NjYwMywgNDYuMDQ5MTQ3XSwgWzEwLjk2NjU0OSwgNDYuMDQ5MTA5XSwgWzEwLjk2NjM5OSwgNDYuMDQ5MDE3XSwgWzEwLjk2NjM0OSwgNDYuMDQ4OTcxXSwgWzEwLjk2NjI4NiwgNDYuMDQ4OTIxXSwgWzEwLjk2NjEzNiwgNDYuMDQ4ODFdLCBbMTAuOTY2MDI3LCA0Ni4wNDg3MzhdLCBbMTAuOTY1OTM3LCA0Ni4wNDg2NzldLCBbMTAuOTY1ODc3LCA0Ni4wNDg2MjVdLCBbMTAuOTY1ODUsIDQ2LjA0ODU3XSwgWzEwLjk2NTg0OSwgNDYuMDQ4NDldLCBbMTAuOTY1ODM4LCA0Ni4wNDg0MjJdLCBbMTAuOTY1Nzk3LCA0Ni4wNDgzMzFdLCBbMTAuOTY1NzQsIDQ2LjA0ODI2N10sIFsxMC45NjU2MzMsIDQ2LjA0ODE5N10sIFsxMC45NjU1NDcsIDQ2LjA0ODE1OV0sIFsxMC45NjU0MTUsIDQ2LjA0ODEwNV0sIFsxMC45NjUzMDksIDQ2LjA0ODA2N10sIFsxMC45NjUxOSwgNDYuMDQ4MDM4XSwgWzEwLjk2NTA3NSwgNDYuMDQ4MDE2XSwgWzEwLjk2NDkxNywgNDYuMDQ3OTldLCBbMTAuOTY0NzMyLCA0Ni4wNDc5NDZdLCBbMTAuOTY0NjI2LCA0Ni4wNDc5MDhdLCBbMTAuOTY0NTEsIDQ2LjA0Nzg1NF0sIFsxMC45NjQ0MTcsIDQ2LjA0NzgwNF0sIFsxMC45NjQyNjEsIDQ2LjA0NzY5OF0sIFsxMC45NjQxODUsIDQ2LjA0NzYyNV0sIFsxMC45NjQxMTQsIDQ2LjA0NzU1XSwgWzEwLjk2NDA1MSwgNDYuMDQ3NDczXSwgWzEwLjk2Mzk4MywgNDYuMDQ3Mzg3XSwgWzEwLjk2MzkwMywgNDYuMDQ3Mjg5XSwgWzEwLjk2Mzc5MywgNDYuMDQ3MTc4XSwgWzEwLjk2MzcwNiwgNDYuMDQ3MDg3XSwgWzEwLjk2MzYzNiwgNDYuMDQ3MDFdLCBbMTAuOTYzNDkyLCA0Ni4wNDY4NDJdLCBbMTAuOTYzNDE4LCA0Ni4wNDY3MzVdLCBbMTAuOTYzMzE3LCA0Ni4wNDY2MDddLCBbMTAuOTYzMTgsIDQ2LjA0NjQ2NF0sIFsxMC45NjMwMzMsIDQ2LjA0NjMzNV0sIFsxMC45NjI5NDIsIDQ2LjA0NjE4N10sIFsxMC45NjI5MiwgNDYuMDQ2MDc4XSwgWzEwLjk2Mjg5OSwgNDYuMDQ1OTk2XSwgWzEwLjk2Mjg4OSwgNDYuMDQ1OTM0XSwgWzEwLjk2MjkwMywgNDYuMDQ1ODMzXSwgWzEwLjk2MjkwOCwgNDYuMDQ1NzAxXSwgWzEwLjk2Mjg2MSwgNDYuMDQ1NzA3XSwgWzEwLjk2MjgxNiwgNDYuMDQ1NzEzXSwgWzEwLjk2Mjc4NCwgNDYuMDQ1NzYxXSwgWzEwLjk2MjcyOSwgNDYuMDQ1ODI1XSwgWzEwLjk2MjY4NCwgNDYuMDQ1ODc4XSwgWzEwLjk2MjYyOSwgNDYuMDQ1OTI3XSwgWzEwLjk2MjU3MSwgNDYuMDQ1OTkxXSwgWzEwLjk2MjU1MiwgNDYuMDQ2MDI4XSwgWzEwLjk2MjU5LCA0Ni4wNDYxNTNdLCBbMTAuOTYyNjMzLCA0Ni4wNDYyMTldLCBbMTAuOTYyNjY3LCA0Ni4wNDYyODVdLCBbMTAuOTYyNzAyLCA0Ni4wNDYzNzJdLCBbMTAuOTYyNzYzLCA0Ni4wNDY0OTFdLCBbMTAuOTYyODMsIDQ2LjA0NjYwN10sIFsxMC45NjI4OTUsIDQ2LjA0NjcyNV0sIFsxMC45NjI5NjYsIDQ2LjA0Njg1M10sIFsxMC45NjMwMzQsIDQ2LjA0NzAxMl0sIFsxMC45NjMxMDIsIDQ2LjA0NzE1M10sIFsxMC45NjMxNjMsIDQ2LjA0NzI1MV0sIFsxMC45NjMyMzQsIDQ2LjA0NzM3NF0sIFsxMC45NjMyNTYsIDQ2LjA0NzUwNl0sIFsxMC45NjMyNTQsIDQ2LjA0NzYxOV0sIFsxMC45NjMyNTMsIDQ2LjA0Nzc1MV0sIFsxMC45NjMyNDksIDQ2LjA0NzkxMV0sIFsxMC45NjMyNTgsIDQ2LjA0ODA0MV0sIFsxMC45NjMyNTMsIDQ2LjA0ODE4MV0sIFsxMC45NjMyMjksIDQ2LjA0ODMyM10sIFsxMC45NjMyMTQsIDQ2LjA0ODQxOV0sIFsxMC45NjMxNjQsIDQ2LjA0ODU0OF0sIFsxMC45NjMxMTYsIDQ2LjA0ODYzN10sIFsxMC45NjMwNTgsIDQ2LjA0ODcwMl0sIFsxMC45NjI5NjcsIDQ2LjA0ODc3M10sIFsxMC45NjI4NjMsIDQ2LjA0ODg0N10sIFsxMC45NjI3NTIsIDQ2LjA0ODkwN10sIFsxMC45NjI2NDEsIDQ2LjA0ODk1Nl0sIFsxMC45NjI1MjcsIDQ2LjA0OTAwMV0sIFsxMC45NjI0MDksIDQ2LjA0OTAzOF0sIFsxMC45NjIzMTEsIDQ2LjA0OTA2NF0sIFsxMC45NjIyMjMsIDQ2LjA0OTA5NF0sIFsxMC45NjIxMzIsIDQ2LjA0OTEwOV0sIFsxMC45NjIxMTQsIDQ2LjA0OTEyMl0sIFsxMC45NjIwNzgsIDQ2LjA0OTE0OF0sIFsxMC45NjIwNDMsIDQ2LjA0OTE2OV0sIFsxMC45NjE5MjMsIDQ2LjA0OTIzXSwgWzEwLjk2MTg2MSwgNDYuMDQ5MjYyXSwgWzEwLjk2MTgxNiwgNDYuMDQ5Mjk1XSwgWzEwLjk2MTc2OCwgNDYuMDQ5MzQ2XSwgWzEwLjk2MTczMywgNDYuMDQ5Mzk0XSwgWzEwLjk2MTY4NSwgNDYuMDQ5NDU5XSwgWzEwLjk2MTYzNywgNDYuMDQ5NTE2XSwgWzEwLjk2MTU4OSwgNDYuMDQ5NTc2XSwgWzEwLjk2MTUyMSwgNDYuMDQ5NjM3XSwgWzEwLjk2MTQwNCwgNDYuMDQ5NjY4XSwgWzEwLjk2MTI3NiwgNDYuMDQ5Njg3XSwgWzEwLjk2MTE0MiwgNDYuMDQ5NzEyXSwgWzEwLjk2MTAyMSwgNDYuMDQ5NzUyXSwgWzEwLjk2MDg3OCwgNDYuMDQ5ODEzXSwgWzEwLjk2MDc2NCwgNDYuMDQ5ODZdLCBbMTAuOTYwNjMxLCA0Ni4wNDk5M10sIFsxMC45NjA1MjUsIDQ2LjA1MDAwN10sIFsxMC45NjA0MzIsIDQ2LjA1MDEwNF0sIFsxMC45NjAzMzMsIDQ2LjA1MDI0XSwgWzEwLjk2MDI2LCA0Ni4wNTAzNTNdLCBbMTAuOTYwMjI3LCA0Ni4wNTA0NTZdLCBbMTAuOTYwMiwgNDYuMDUwNTg0XSwgWzEwLjk2MDE5NSwgNDYuMDUwNjQyXSwgWzEwLjk2MDIxNSwgNDYuMDUwNzgxXSwgWzEwLjk2MDI0MiwgNDYuMDUwODQ1XSwgWzEwLjk2MDMyMywgNDYuMDUwOTE5XSwgWzEwLjk2MDQwMywgNDYuMDUwOTcxXSwgWzEwLjk2MDUzOSwgNDYuMDUxMDI2XSwgWzEwLjk2MDY3MSwgNDYuMDUxMDYzXSwgWzEwLjk2MDg1LCA0Ni4wNTExMjhdLCBbMTAuOTYwOTcsIDQ2LjA1MTE5Ml0sIFsxMC45NjEwOSwgNDYuMDUxMjU1XSwgWzEwLjk2MTE5OSwgNDYuMDUxMjgzXSwgWzEwLjk2MTMwNSwgNDYuMDUxMzIzXSwgWzEwLjk2MTM4OCwgNDYuMDUxMzc3XSwgWzEwLjk2MTQ4NiwgNDYuMDUxNDU0XSwgWzEwLjk2MTU1MywgNDYuMDUxNTI0XSwgWzEwLjk2MTU5LCA0Ni4wNTE1ODNdLCBbMTAuOTYxNjIxLCA0Ni4wNTE2MzddLCBbMTAuOTYxNjY1LCA0Ni4wNTE2NjZdLCBbMTAuOTYxNzE5LCA0Ni4wNTE3MTZdLCBbMTAuOTYxNzM5LCA0Ni4wNTE3NV0sIFsxMC45NjE3NTEsIDQ2LjA1MTg2XSwgWzEwLjk2MTc5NSwgNDYuMDUxODk4XSwgWzEwLjk2MTg1MiwgNDYuMDUxOTVdLCBbMTAuOTYxODc2LCA0Ni4wNTIwMDRdLCBbMTAuOTYxODY0LCA0Ni4wNTIwNDhdLCBbMTAuOTYxODEzLCA0Ni4wNTIxMTldLCBbMTAuOTYxNzUyLCA0Ni4wNTIxNTJdLCBbMTAuOTYxNjc3LCA0Ni4wNTIxNjddLCBbMTAuOTYxNjI3LCA0Ni4wNTIxNjVdLCBbMTAuOTYxNDQ1LCA0Ni4wNTIxMDFdLCBbMTAuOTYxMzE1LCA0Ni4wNTIwMjldLCBbMTAuOTYxMjA4LCA0Ni4wNTE5NDhdLCBbMTAuOTYxMTA0LCA0Ni4wNTE4Nl0sIFsxMC45NjEwMTMsIDQ2LjA1MTc2OF0sIFsxMC45NjA5MjMsIDQ2LjA1MTY5MV0sIFsxMC45NjA3OTYsIDQ2LjA1MTYxXSwgWzEwLjk2MDY1MywgNDYuMDUxNTI1XSwgWzEwLjk2MDQ5LCA0Ni4wNTE0NDddLCBbMTAuOTYwMzI0LCA0Ni4wNTEzOTddLCBbMTAuOTYwMTk5LCA0Ni4wNTEzNjhdLCBbMTAuOTYwMDcsIDQ2LjA1MTM0Ml0sIFsxMC45NTk5ODQsIDQ2LjA1MTMzNF0sIFsxMC45NTk5MjUsIDQ2LjA1MTMxOV0sIFsxMC45NTk4OTIsIDQ2LjA1MTI5N10sIFsxMC45NTk4NzcsIDQ2LjA1MTI1M10sIFsxMC45NTk4OTMsIDQ2LjA1MTIyOF0sIFsxMC45NTk5MDksIDQ2LjA1MTIwM10sIFsxMC45NTk5MTEsIDQ2LjA1MTE0NV0sIFsxMC45NTk5MTcsIDQ2LjA1MTExNl0sIFsxMC45NTk5MTYsIDQ2LjA1MTA2NV0sIFsxMC45NTk4ODYsIDQ2LjA1MTA0OF0sIFsxMC45NTk4MTYsIDQ2LjA1MTAxOV0sIFsxMC45NTk3NjEsIDQ2LjA1MTA0NF0sIFsxMC45NTk3MjUsIDQ2LjA1MTA4MV0sIFsxMC45NTk2NjgsIDQ2LjA1MTE1M10sIFsxMC45NTk2MjEsIDQ2LjA1MTI1Nl0sIFsxMC45NTk1NjIsIDQ2LjA1MTM5Nl0sIFsxMC45NTk1MDYsIDQ2LjA1MTUxNF0sIFsxMC45NTk0NDksIDQ2LjA1MTYwOF0sIFsxMC45NTk0MTQsIDQ2LjA1MTY3OV0sIFsxMC45NTkzNTksIDQ2LjA1MTcxNl0sIFsxMC45NTkzMSwgNDYuMDUxNzE3XSwgWzEwLjk1OTI0NywgNDYuMDUxNjk1XSwgWzEwLjk1OTE4NSwgNDYuMDUxNjk4XSwgWzEwLjk1OTE0MiwgNDYuMDUxNzE5XSwgWzEwLjk1OTA4OCwgNDYuMDUxNzc0XSwgWzEwLjk1OTA0NiwgNDYuMDUxODE2XSwgWzEwLjk1OTAxMSwgNDYuMDUxODY0XSwgWzEwLjk1OTAyMiwgNDYuMDUxOTI0XSwgWzEwLjk1OTA1LCA0Ni4wNTE5NjldLCBbMTAuOTU5MDc0LCA0Ni4wNTIwMTldLCBbMTAuOTU5MDkyLCA0Ni4wNTIwODddLCBbMTAuOTU5MDg0LCA0Ni4wNTIxNjhdLCBbMTAuOTU5MDYsIDQ2LjA1MjI0Nl0sIFsxMC45NTkwMzUsIDQ2LjA1MjMzM10sIFsxMC45NTkwNDgsIDQ2LjA1MjQ0OV0sIFsxMC45NTkwNzEsIDQ2LjA1MjU4NF0sIFsxMC45NTkxMTQsIDQ2LjA1MjczMl0sIFsxMC45NTkxODMsIDQ2LjA1Mjg5OF0sIFsxMC45NTkyNDEsIDQ2LjA1MzA4OV0sIFsxMC45NTkyNDQsIDQ2LjA1MzIyOF0sIFsxMC45NTkyNTcsIDQ2LjA1MzM2NV0sIFsxMC45NTkyOSwgNDYuMDUzNTIzXSwgWzEwLjk1OTM1NiwgNDYuMDUzNjgyXSwgWzEwLjk1OTQyNSwgNDYuMDUzNzkxXSwgWzEwLjk1OTUzMiwgNDYuMDUzODg4XSwgWzEwLjk1OTY4MiwgNDYuMDUzOTczXSwgWzEwLjk1OTc5OCwgNDYuMDU0MDI4XSwgWzEwLjk1OTkyNSwgNDYuMDU0MDg5XSwgWzEwLjk2MDA5NCwgNDYuMDU0MTc2XSwgWzEwLjk2MDMxLCA0Ni4wNTQyNl0sIFsxMC45NjA1NTksIDQ2LjA1NDM3NF0sIFsxMC45NjA4MjEsIDQ2LjA1NDQ5Ml0sIFsxMC45NjExMTQsIDQ2LjA1NDYyM10sIFsxMC45NjEzNzksIDQ2LjA1NDcyN10sIFsxMC45NjE2NjcsIDQ2LjA1NDgxNV0sIFsxMC45NjE5MjUsIDQ2LjA1NDkwNl0sIFsxMC45NjIxNzQsIDQ2LjA1NTAwOF0sIFsxMC45NjI0LCA0Ni4wNTUxMDFdLCBbMTAuOTYyNTg5LCA0Ni4wNTUxODhdLCBbMTAuOTYyNzU5LCA0Ni4wNTUyNTddLCBbMTAuOTYyOTQ3LCA0Ni4wNTUzMjhdLCBbMTAuOTYzMTQ3LCA0Ni4wNTU0MDhdLCBbMTAuOTYzMzcsIDQ2LjA1NTUyN10sIFsxMC45NjM2MTksIDQ2LjA1NTY1NF0sIFsxMC45NjM5MTIsIDQ2LjA1NTgxM10sIFsxMC45NjQxMjIsIDQ2LjA1NTkzNl0sIFsxMC45NjQyNzIsIDQ2LjA1NjAzOV0sIFsxMC45NjQzODYsIDQ2LjA1NjEyMl0sIFsxMC45NjQ1MDUsIDQ2LjA1NjE2NF0sIFsxMC45NjQ2NTQsIDQ2LjA1NjE4NV0sIFsxMC45NjQ3NDIsIDQ2LjA1NjE2Ml0sIFsxMC45NjQ4LCA0Ni4wNTYxMl0sIFsxMC45NjQ4MTEsIDQ2LjA1NjEwN10sIFsxMC45NjQ4NTgsIDQ2LjA1NjA1N10sIFsxMC45NjQ5NjEsIDQ2LjA1NTk5Ml0sIFsxMC45NjUwNDIsIDQ2LjA1NTkzNl0sIFsxMC45NjUxMjYsIDQ2LjA1NTg1NV0sIFsxMC45NjUxODYsIDQ2LjA1NTc4Nl0sIFsxMC45NjUyMzcsIDQ2LjA1NTcyMl0sIFsxMC45NjUyODYsIDQ2LjA1NTY4XSwgWzEwLjk2NTM4LCA0Ni4wNTU2NF0sIFsxMC45NjU0OTgsIDQ2LjA1NTYyNV0sIFsxMC45NjU2MDcsIDQ2LjA1NTYzM10sIFsxMC45NjU3NDUsIDQ2LjA1NTY1Nl0sIFsxMC45NjU4NzQsIDQ2LjA1NTY5Nl0sIFsxMC45NjU5NTcsIDQ2LjA1NTcyN10sIFsxMC45NjYwODMsIDQ2LjA1NTc4XSwgWzEwLjk2NjE5MiwgNDYuMDU1ODA5XSwgWzEwLjk2NjMxNywgNDYuMDU1Nzg2XSwgWzEwLjk2NjQyNCwgNDYuMDU1NzQ2XSwgWzEwLjk2NjUyNSwgNDYuMDU1NzA2XSwgWzEwLjk2NjU4NCwgNDYuMDU1NjgzXSwgWzEwLjk2NjY3NiwgNDYuMDU1Njc5XSwgWzEwLjk2Njc3OCwgNDYuMDU1NzAxXSwgWzEwLjk2Njg4LCA0Ni4wNTU3MThdLCBbMTAuOTY2OTgyLCA0Ni4wNTU3MjRdLCBbMTAuOTY3MDg3LCA0Ni4wNTU3Ml0sIFsxMC45NjcxNCwgNDYuMDU1NzE1XSwgWzEwLjk2NzI1OCwgNDYuMDU1NzA0XSwgWzEwLjk2NzM2NiwgNDYuMDU1NzA4XSwgWzEwLjk2NzQyNCwgNDYuMDU1NjQ1XSwgWzEwLjk2NzI5MiwgNDYuMDU1NjJdLCBbMTAuOTY3MjIzLCA0Ni4wNTU2MThdLCBbMTAuOTY3MTYxLCA0Ni4wNTU2MTldLCBbMTAuOTY3MDk1LCA0Ni4wNTU2MjJdLCBbMTAuOTY3MDQyLCA0Ni4wNTU2MjVdLCBbMTAuOTY2OTgsIDQ2LjA1NTYyOF0sIFsxMC45NjY5MjcsIDQ2LjA1NTYyOF0sIFsxMC45NjY4NzUsIDQ2LjA1NTYyXSwgWzEwLjk2Njc5OSwgNDYuMDU1NjA1XSwgWzEwLjk2NjcyNiwgNDYuMDU1NTldLCBbMTAuOTY2NjcsIDQ2LjA1NTU3N10sIFsxMC45NjY2MjcsIDQ2LjA1NTU2MV0sIFsxMC45NjY1NjcsIDQ2LjA1NTUzMl0sIFsxMC45NjY1MSwgNDYuMDU1NTA1XSwgWzEwLjk2NjQ3NywgNDYuMDU1NDgxXSwgWzEwLjk2NjQ0NiwgNDYuMDU1NDMxXSwgWzEwLjk2NjQzOSwgNDYuMDU1MzldLCBbMTAuOTY2NDQ0LCA0Ni4wNTUzNTFdLCBbMTAuOTY2NDY2LCA0Ni4wNTUyODRdLCBbMTAuOTY2NDg3LCA0Ni4wNTUyMThdLCBbMTAuOTY2NTMyLCA0Ni4wNTUxNDldLCBbMTAuOTY2NTc3LCA0Ni4wNTUxMDVdLCBbMTAuOTY2NjExLCA0Ni4wNTUwNF0sIFsxMC45NjY2MTMsIDQ2LjA1NDk5Ml0sIFsxMC45NjY1OTgsIDQ2LjA1NDkyMl0sIFsxMC45NjY1OTQsIDQ2LjA1NDg1M10sIFsxMC45NjY1ODUsIDQ2LjA1NDc2NF0sIFsxMC45NjY1OTcsIDQ2LjA1NDddLCBbMTAuOTY2NjQ0LCA0Ni4wNTQ2MTddLCBbMTAuOTY2Njg0LCA0Ni4wNTQ1MjVdLCBbMTAuOTY2Njk2LCA0Ni4wNTQ0NDVdLCBbMTAuOTY2NzExLCA0Ni4wNTQ0MDRdLCBbMTAuOTY2ODAyLCA0Ni4wNTQzNzNdLCBbMTAuOTY2OTA3LCA0Ni4wNTQzNThdLCBbMTAuOTY2OTg5LCA0Ni4wNTQzNTddLCBbMTAuOTY3MDc1LCA0Ni4wNTQzNjhdLCBbMTAuOTY3MjMzLCA0Ni4wNTQzOTFdLCBbMTAuOTY3MzQ5LCA0Ni4wNTQ0MTJdLCBbMTAuOTY3NDU1LCA0Ni4wNTQ0NDNdLCBbMTAuOTY3NTc3LCA0Ni4wNTQ0OTJdLCBbMTAuOTY3NjgxLCA0Ni4wNTQ1NTJdLCBbMTAuOTY3NzQ4LCA0Ni4wNTQ2MzJdLCBbMTAuOTY3NzkzLCA0Ni4wNTQ3Ml0sIFsxMC45Njc4NSwgNDYuMDU0Nzc5XSwgWzEwLjk2NzkxNywgNDYuMDU0ODI2XSwgWzEwLjk2Nzk2NywgNDYuMDU0ODY3XSwgWzEwLjk2Nzk4NSwgNDYuMDU0OTFdLCBbMTAuOTY3OTk3LCA0Ni4wNTQ5OTJdLCBbMTAuOTY3OTgyLCA0Ni4wNTUwODNdLCBbMTAuOTY3OTQ5LCA0Ni4wNTUyMDddLCBbMTAuOTY3OTIyLCA0Ni4wNTUzMjJdLCBbMTAuOTY3ODkyLCA0Ni4wNTU0Ml0sIFsxMC45Njc4NjgsIDQ2LjA1NTUxNF0sIFsxMC45Njc4MzcsIDQ2LjA1NTU5M10sIFsxMC45Njc4MDIsIDQ2LjA1NTY2Nl0sIFsxMC45Njc3NywgNDYuMDU1NzA1XSwgWzEwLjk2NzcwMiwgNDYuMDU1NzJdLCBbMTAuOTY3NjMzLCA0Ni4wNTU3MTFdLCBbMTAuOTY3NTUxLCA0Ni4wNTU3MTddLCBbMTAuOTY3NTE5LCA0Ni4wNTU3NzddLCBbMTAuOTY3NTQ5LCA0Ni4wNTU3OTddLCBbMTAuOTY3NTg5LCA0Ni4wNTU4MjJdLCBbMTAuOTY3NjE2LCA0Ni4wNTU4Nl0sIFsxMC45Njc1OTgsIDQ2LjA1NTkwOF0sIFsxMC45Njc1NjYsIDQ2LjA1NTkzNF0sIFsxMC45Njc1NDMsIDQ2LjA1NTk2OF0sIFsxMC45Njc1MjgsIDQ2LjA1NjAwNV0sIFsxMC45Njc1NDksIDQ2LjA1NjA0NF0sIFsxMC45Njc1NzksIDQ2LjA1NjA5NF0sIFsxMC45Njc2MjMsIDQ2LjA1NjEzNF0sIFsxMC45Njc3NywgNDYuMDU2MjI5XSwgWzEwLjk2Nzg4LCA0Ni4wNTYyOThdLCBbMTAuOTY3OTksIDQ2LjA1NjM2M10sIFsxMC45NjgxLCA0Ni4wNTY0MjNdLCBbMTAuOTY4MjU5LCA0Ni4wNTY0OTJdLCBbMTAuOTY4NDcxLCA0Ni4wNTY1NjhdLCBbMTAuOTY4NzM5LCA0Ni4wNTY2NDRdLCBbMTAuOTY5MDQ0LCA0Ni4wNTY3MzJdLCBbMTAuOTY5MzE1LCA0Ni4wNTY4MThdLCBbMTAuOTY5NTE0LCA0Ni4wNTY4OTFdLCBbMTAuOTY5NzEzLCA0Ni4wNTY5NjddLCBbMTAuOTY5ODY5LCA0Ni4wNTcwNDVdLCBbMTAuOTcwMDE2LCA0Ni4wNTcxM10sIFsxMC45NzAxODMsIDQ2LjA1NzIzN10sIFsxMC45NzAzNjMsIDQ2LjA1NzM3NV0sIFsxMC45NzA1MDQsIDQ2LjA1NzQ5OV0sIFsxMC45NzA2MTYsIDQ2LjA1NzYyMV0sIFsxMC45NzA2OCwgNDYuMDU3NzA5XSwgWzEwLjk3MDc2MiwgNDYuMDU3ODA2XSwgWzEwLjk3MDg1NiwgNDYuMDU3OTA0XSwgWzEwLjk3MDk1NywgNDYuMDU3OTkyXSwgWzEwLjk3MTA0LCA0Ni4wNTgwNTJdLCBbMTAuOTcxMTQ3LCA0Ni4wNTgxMV0sIFsxMC45NzEyMjYsIDQ2LjA1ODE1MV0sIFsxMC45NzEyODcsIDQ2LjA1ODIxNF0sIFsxMC45NzEyNjgsIDQ2LjA1ODI1NV0sIFsxMC45NzExOTQsIDQ2LjA1ODI5XSwgWzEwLjk3MTIzNywgNDYuMDU4MzI5XSwgWzEwLjk3MTI4LCA0Ni4wNTgzM10sIFsxMC45NzEzMjYsIDQ2LjA1ODMzMl0sIFsxMC45NzE0MTUsIDQ2LjA1ODMyN10sIFsxMC45NzE0ODcsIDQ2LjA1ODMxOV0sIFsxMC45NzE1NDksIDQ2LjA1ODMwN10sIFsxMC45NzE2MDQsIDQ2LjA1ODI5XSwgWzEwLjk3MTY3OSwgNDYuMDU4MjY0XSwgWzEwLjk3MTc1NCwgNDYuMDU4MjM4XSwgWzEwLjk3MTgyMywgNDYuMDU4MjA3XSwgWzEwLjk3MTg3NywgNDYuMDU4MTU0XSwgWzEwLjk3MTkwOSwgNDYuMDU4MDldLCBbMTAuOTcxOTEzLCA0Ni4wNTgwMTldLCBbMTAuOTcxODkyLCA0Ni4wNTc5NTFdLCBbMTAuOTcxODU4LCA0Ni4wNTc4OTRdLCBbMTAuOTcxODEsIDQ2LjA1NzgzNV0sIFsxMC45NzE3OTMsIDQ2LjA1Nzc4NV0sIFsxMC45NzE3ODgsIDQ2LjA1NzcxN10sIFsxMC45NzE3OTYsIDQ2LjA1NzY2Ml0sIFsxMC45NzE4MTgsIDQ2LjA1NzYwN10sIFsxMC45NzE4NjYsIDQ2LjA1NzU0OV0sIFsxMC45NzE4OTIsIDQ2LjA1NzUyM10sIFsxMC45NzE5MzEsIDQ2LjA1NzUwNV0sIFsxMC45NzE5NzcsIDQ2LjA1NzQ5N10sIFsxMC45NzIwNjgsIDQ2LjA1NzQ4OV0sIFsxMC45NzIxMzcsIDQ2LjA1NzQ4Nl0sIFsxMC45NzIyMzYsIDQ2LjA1NzQ4XSwgWzEwLjk3MjM1NywgNDYuMDU3NDddLCBbMTAuOTcyNTY0LCA0Ni4wNTc0NThdLCBbMTAuOTcyNzkxLCA0Ni4wNTc0NjJdLCBbMTAuOTczMDE4LCA0Ni4wNTc0NzFdLCBbMTAuOTczMjY0LCA0Ni4wNTc0ODddLCBbMTAuOTczNDQ1LCA0Ni4wNTc0OTZdLCBbMTAuOTczNjM3LCA0Ni4wNTc1MTldLCBbMTAuOTczODQ4LCA0Ni4wNTc1NTddLCBbMTAuOTc0MTkyLCA0Ni4wNTc2NDJdLCBbMTAuOTc0MzI0LCA0Ni4wNTc2NjZdLCBbMTAuOTc0NTExLCA0Ni4wNTc2ODddLCBbMTAuOTc0NzQ1LCA0Ni4wNTc3MTFdLCBbMTAuOTc0OTY2LCA0Ni4wNTc3NDFdLCBbMTAuOTc1MTc3LCA0Ni4wNTc3NzldLCBbMTAuOTc1MzM2LCA0Ni4wNTc4MjFdLCBbMTAuOTc1NDc1LCA0Ni4wNTc4NzRdLCBbMTAuOTc1NjQ4LCA0Ni4wNTc5NDNdLCBbMTAuOTc1ODU3LCA0Ni4wNTgwMjddLCBbMTAuOTc2MDQsIDQ2LjA1ODEyMV0sIFsxMC45NzYxNzYsIDQ2LjA1ODE5NV0sIFsxMC45NzYzMDYsIDQ2LjA1ODI3MV0sIFsxMC45NzY0NDMsIDQ2LjA1ODM1MV0sIFsxMC45NzY1NDYsIDQ2LjA1ODQwM10sIFsxMC45NzY2NjksIDQ2LjA1ODQ2NV0sIFsxMC45NzY4MDksIDQ2LjA1ODUzOV0sIFsxMC45NzY5NTYsIDQ2LjA1ODYyMl0sIFsxMC45NzcwODUsIDQ2LjA1ODY4OV0sIFsxMC45NzcyMDksIDQ2LjA1ODc2XSwgWzEwLjk3NzI5OSwgNDYuMDU4ODE5XSwgWzEwLjk3NzM3MywgNDYuMDU4ODc1XSwgWzEwLjk3NzQwMywgNDYuMDU4OTJdLCBbMTAuOTc3NDA4LCA0Ni4wNTldLCBbMTAuOTc3NDEsIDQ2LjA1OTA2N10sIFsxMC45Nzc0MTcsIDQ2LjA1OTExNF1dXV0sICJ0eXBlIjogIk11bHRpUG9seWdvbiJ9LCAiaWQiOiAiOCIsICJwcm9wZXJ0aWVzIjogeyJkZXNjcml6aW9uZV9vcmlnaW5lIjogIk5vbiBjb2RpZmljYXRvIiwgImdtbF9pZCI6ICJJRC5BQ1FVRUZJU0lDSEUuU1BFQ0NISS5BQ1FVQS41NDEzMyIsICJuYXNwIjogMCwgIm5hdHVyYV9zcGVjY2hpb19hY3F1YSI6ICJOb24gY29kaWZpY2F0byIsICJub21lIjogIkxBR08gREkgVE9CTElOTyIsICJvcmlnaW5lIjogMH0sICJ0eXBlIjogIkZlYXR1cmUifSwgeyJnZW9tZXRyeSI6IHsiY29vcmRpbmF0ZXMiOiBbW1tbMTAuOTE4MDU0LCA0NS44NjldLCBbMTAuOTE4MDExLCA0NS44Njg5NzNdLCBbMTAuOTE3OTU0LCA0NS44Njg5NDldLCBbMTAuOTE3ODg4LCA0NS44Njg5MzFdLCBbMTAuOTE3Nzc5LCA0NS44Njg5MV0sIFsxMC45MTc2MTIsIDQ1Ljg2ODg5XSwgWzEwLjkxNzQ0NywgNDUuODY4ODddLCBbMTAuOTE3MzA5LCA0NS44Njg4NThdLCBbMTAuOTE3MTE5LCA0NS44Njg4NDddLCBbMTAuOTE2OTg0LCA0NS44Njg4NDVdLCBbMTAuOTE2ODgsIDQ1Ljg2ODg1Nl0sIFsxMC45MTY4MDUsIDQ1Ljg2ODg3OF0sIFsxMC45MTY3NTQsIDQ1Ljg2ODkwNl0sIFsxMC45MTY3NDksIDQ1Ljg2ODk3OV0sIFsxMC45MTY3OTMsIDQ1Ljg2OTAxNV0sIFsxMC45MTY4NTMsIDQ1Ljg2OTA2XSwgWzEwLjkxNjkyNywgNDUuODY5MV0sIFsxMC45MTcsIDQ1Ljg2OTEzNl0sIFsxMC45MTcwOSwgNDUuODY5MTY5XSwgWzEwLjkxNzE4NSwgNDUuODY5MTk3XSwgWzEwLjkxNzM0LCA0NS44NjkyMTNdLCBbMTAuOTE3NDAzLCA0NS44NjkyNDFdLCBbMTAuOTE3NDU5LCA0NS44NjkyNV0sIFsxMC45MTc1MTUsIDQ1Ljg2OTI1Nl0sIFsxMC45MTc2MDMsIDQ1Ljg2OTI1XSwgWzEwLjkxNzcyMSwgNDUuODY5MjQxXSwgWzEwLjkxNzgxMiwgNDUuODY5MjE5XSwgWzEwLjkxNzg3MywgNDUuODY5MTc3XSwgWzEwLjkxNzkzNywgNDUuODY5MTIzXSwgWzEwLjkxODAyLCA0NS44NjkwNjddLCBbMTAuOTE4MDU1LCA0NS44NjkwM10sIFsxMC45MTgwNTQsIDQ1Ljg2OV1dXV0sICJ0eXBlIjogIk11bHRpUG9seWdvbiJ9LCAiaWQiOiAiOSIsICJwcm9wZXJ0aWVzIjogeyJkZXNjcml6aW9uZV9vcmlnaW5lIjogIk5vbiBjb2RpZmljYXRvIiwgImdtbF9pZCI6ICJJRC5BQ1FVRUZJU0lDSEUuU1BFQ0NISS5BQ1FVQS41NDE4OCIsICJuYXNwIjogMCwgIm5hdHVyYV9zcGVjY2hpb19hY3F1YSI6ICJOb24gY29kaWZpY2F0byIsICJub21lIjogIiIsICJvcmlnaW5lIjogMH0sICJ0eXBlIjogIkZlYXR1cmUifSwgeyJnZW9tZXRyeSI6IHsiY29vcmRpbmF0ZXMiOiBbW1tbMTAuODI5NTU3LCA0NS45OTgxMV0sIFsxMC44Mjk1NDIsIDQ1Ljk5ODA2XSwgWzEwLjgyOTUwNCwgNDUuOTk4MDIyXSwgWzEwLjgyOTQyNywgNDUuOTk3OTgyXSwgWzEwLjgyOTMzNywgNDUuOTk3OTU0XSwgWzEwLjgyOTI0OCwgNDUuOTk3OTM3XSwgWzEwLjgyOTE0OCwgNDUuOTk3OTE3XSwgWzEwLjgyOTAyOCwgNDUuOTk3ODg3XSwgWzEwLjgyODk0MywgNDUuOTk3ODgyXSwgWzEwLjgyODgwMywgNDUuOTk3OTE3XSwgWzEwLjgyODc0OCwgNDUuOTk3OTQzXSwgWzEwLjgyODY3NiwgNDUuOTk3OTYxXSwgWzEwLjgyODYxNCwgNDUuOTk3OTU3XSwgWzEwLjgyODQ4MywgNDUuOTk3OTc4XSwgWzEwLjgyODQ1NiwgNDUuOTk4MDQzXSwgWzEwLjgyODQ1MSwgNDUuOTk4MDgyXSwgWzEwLjgyODQyNywgNDUuOTk4MTI2XSwgWzEwLjgyODM4NSwgNDUuOTk4MTYxXSwgWzEwLjgyODM0NywgNDUuOTk4MTgzXSwgWzEwLjgyODMxNSwgNDUuOTk4MjAyXSwgWzEwLjgyODI4OSwgNDUuOTk4MjIzXSwgWzEwLjgyODI3NSwgNDUuOTk4MjcxXSwgWzEwLjgyODI2NCwgNDUuOTk4MzJdLCBbMTAuODI4MjI0LCA0NS45OTg0MDVdLCBbMTAuODI4MTg4LCA0NS45OTg0ODldLCBbMTAuODI4MTYsIDQ1Ljk5ODU5Ml0sIFsxMC44MjgxMzEsIDQ1Ljk5ODY5OF0sIFsxMC44MjgxMDUsIDQ1Ljk5ODc4NF0sIFsxMC44MjgwODksIDQ1Ljk5ODg1OF0sIFsxMC44MjgwNzksIDQ1Ljk5ODkzMV0sIFsxMC44MjgwOTYsIDQ1Ljk5ODk2M10sIFsxMC44MjgxNDQsIDQ1Ljk5ODk5Ml0sIFsxMC44MjgyMDEsIDQ1Ljk5OTAyM10sIFsxMC44MjgyNDcsIDQ1Ljk5OTAzOF0sIFsxMC44MjgzNTYsIDQ1Ljk5OTA1Ml0sIFsxMC44Mjg0MTksIDQ1Ljk5OTA1OV0sIFsxMC44Mjg1MTMsIDQ1Ljk5OTA4N10sIFsxMC44Mjg1OTcsIDQ1Ljk5OTEyOV0sIFsxMC44Mjg2MjQsIDQ1Ljk5OTE1M10sIFsxMC44Mjg2NjksIDQ1Ljk5OTE2MV0sIFsxMC44Mjg3NjcsIDQ1Ljk5OTE1OV0sIFsxMC44Mjg4NDUsIDQ1Ljk5OTE0OV0sIFsxMC44Mjg5MDUsIDQ1Ljk5OTExOF0sIFsxMC44MjkwMjgsIDQ1Ljk5ODk5NF0sIFsxMC44MjkwNzIsIDQ1Ljk5ODk0MV0sIFsxMC44MjkxMzIsIDQ1Ljk5ODg2OV0sIFsxMC44MjkxOTEsIDQ1Ljk5ODc4N10sIFsxMC44MjkyMjgsIDQ1Ljk5ODczMV0sIFsxMC44MjkyNzcsIDQ1Ljk5ODY1NV0sIFsxMC44Mjk0MTIsIDQ1Ljk5ODQxOF0sIFsxMC44Mjk0NjUsIDQ1Ljk5ODMzOV0sIFsxMC44Mjk1MjcsIDQ1Ljk5ODI0Nl0sIFsxMC44Mjk1NTcsIDQ1Ljk5ODExXV1dXSwgInR5cGUiOiAiTXVsdGlQb2x5Z29uIn0sICJpZCI6ICIxMCIsICJwcm9wZXJ0aWVzIjogeyJkZXNjcml6aW9uZV9vcmlnaW5lIjogIk5vbiBjb2RpZmljYXRvIiwgImdtbF9pZCI6ICJJRC5BQ1FVRUZJU0lDSEUuU1BFQ0NISS5BQ1FVQS41NDE1OCIsICJuYXNwIjogMCwgIm5hdHVyYV9zcGVjY2hpb19hY3F1YSI6ICJOb24gY29kaWZpY2F0byIsICJub21lIjogIkxBR08gVE9SQklFUkEgREkgRklBVkVcdTAwMjcgMT8iLCAib3JpZ2luZSI6IDB9LCAidHlwZSI6ICJGZWF0dXJlIn1dLCAidHlwZSI6ICJGZWF0dXJlQ29sbGVjdGlvbiJ9KTsKCiAgICAgICAgCjwvc2NyaXB0Pg== onload="this.contentDocument.open();this.contentDocument.write(atob(this.getAttribute('data-html')));this.contentDocument.close();" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



## Summary WFS
- there are different versions
- from the version 1.1.0 you can have the problem of the axis inverted
- check always the boundary: more is big and more you have to wait.. more you have to wait and more the connection can go in timeout
- if the dataset is available as geojson you can load directly in geopandas
- otherwise you need to download in another format (eg. gml), save it and load as normal file


# ESRI ArcGIS Online RESTAPI

![](https://raw.githubusercontent.com/napo/geospatial_course_unitn/master/images/dashboard_covid_protezionecivile.png)

http://opendatadpc.maps.arcgis.com/apps/opsdashboard/index.html#/b0c68bce2cce478eaac82fe38d4138b1

![](https://raw.githubusercontent.com/napo/geospatial_course_unitn/master/images/esri_dpc.png)

https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services

Documentation of the ESRI API's<br/>https://developers.arcgis.com/rest/


```
pip install pyshp
```

    Collecting pyshp
    [?25l  Downloading https://files.pythonhosted.org/packages/ca/1f/e9cc2c3fce32e2926581f8b6905831165235464c858ba550b6e9b8ef78c3/pyshp-2.1.2.tar.gz (217kB)
    [K     |████████████████████████████████| 225kB 2.7MB/s 
    [?25hBuilding wheels for collected packages: pyshp
      Building wheel for pyshp (setup.py) ... [?25l[?25hdone
      Created wheel for pyshp: filename=pyshp-2.1.2-cp36-none-any.whl size=36216 sha256=85e492a6f395812ab045ec8977cb99613e91b04496cfd48891d1c59b9a9c5859
      Stored in directory: /root/.cache/pip/wheels/96/6c/53/4112475adf3b831da97f083163d0f38ee6daac9c1b13f7afea
    Successfully built pyshp
    Installing collected packages: pyshp
    Successfully installed pyshp-2.1.2



```
pip install bmi-arcgis-restapi
```

    Collecting bmi-arcgis-restapi
    [?25l  Downloading https://files.pythonhosted.org/packages/ae/60/ff56525684d55cc7eff6d494799f4f6858e877be252b2138462a2f3bf95b/bmi-arcgis-restapi-2.0.1.tar.gz (486kB)
    [K     |████████████████████████████████| 491kB 2.8MB/s 
    [?25hRequirement already satisfied: munch in /usr/local/lib/python3.6/dist-packages (from bmi-arcgis-restapi) (2.5.0)
    Requirement already satisfied: requests in /usr/local/lib/python3.6/dist-packages (from bmi-arcgis-restapi) (2.23.0)
    Requirement already satisfied: urllib3 in /usr/local/lib/python3.6/dist-packages (from bmi-arcgis-restapi) (1.24.3)
    Requirement already satisfied: six in /usr/local/lib/python3.6/dist-packages (from munch->bmi-arcgis-restapi) (1.15.0)
    Requirement already satisfied: chardet<4,>=3.0.2 in /usr/local/lib/python3.6/dist-packages (from requests->bmi-arcgis-restapi) (3.0.4)
    Requirement already satisfied: idna<3,>=2.5 in /usr/local/lib/python3.6/dist-packages (from requests->bmi-arcgis-restapi) (2.10)
    Requirement already satisfied: certifi>=2017.4.17 in /usr/local/lib/python3.6/dist-packages (from requests->bmi-arcgis-restapi) (2020.6.20)
    Building wheels for collected packages: bmi-arcgis-restapi
      Building wheel for bmi-arcgis-restapi (setup.py) ... [?25l[?25hdone
      Created wheel for bmi-arcgis-restapi: filename=bmi_arcgis_restapi-2.0.1-cp36-none-any.whl size=493541 sha256=9ebcdeec249b97ac1fd7b73af7f83caed60c8c7fae4f31bdcfd0fae928766861
      Stored in directory: /root/.cache/pip/wheels/e4/02/dc/526efc9aa697406a1aff272a704703ac88f790d599a1a23814
    Successfully built bmi-arcgis-restapi
    Installing collected packages: bmi-arcgis-restapi
    Successfully installed bmi-arcgis-restapi-2.0.1



```
import os
# we need this to inform the bmi-arcgis-restapi to use pyshp and not arcpy
os.environ['RESTAPI_USE_ARCPY'] = 'FALSE'
import restapi
```

    arcpy import error:  


    /usr/local/lib/python3.6/dist-packages/restapi/common_types.py:35: UserWarning: No Arcpy found, some limitations in functionality may apply.
      warnings.warn('No Arcpy found, some limitations in functionality may apply.')


[bmi-arcgis-restapi](https://github.com/Bolton-and-Menk-GIS/restapi) can use [arcpy](https://desktop.arcgis.com/en/arcmap/10.3/analyze/arcpy/what-is-arcpy-.htm) (proprietary software) o *pyshp* (opensource).

**pyshp** *faster* but with the *basic* functions


```
rest_url = 'https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services'

# no authentication is required, so no username and password are supplied
ags = restapi.ArcServer(rest_url)
```


```
ags.services
```




    [{
      "name": "campi_scuola_2018",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/campi_scuola_2018/FeatureServer"
    },
     {
      "name": "CampiScuola_2019",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/CampiScuola_2019/FeatureServer"
    },
     {
      "name": "campiscuola2018_def",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/campiscuola2018_def/FeatureServer"
    },
     {
      "name": "CapoluoghiProvincia",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/CapoluoghiProvincia/FeatureServer"
    },
     {
      "name": "Comuni_Terremotati",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/Comuni_Terremotati/FeatureServer"
    },
     {
      "name": "Comuni_terremoto",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/Comuni_terremoto/FeatureServer"
    },
     {
      "name": "ComuniAgricIndus",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/ComuniAgricIndus/FeatureServer"
    },
     {
      "name": "ComuniDemografia",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/ComuniDemografia/FeatureServer"
    },
     {
      "name": "ComuniIstruzione",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/ComuniIstruzione/FeatureServer"
    },
     {
      "name": "ComuniTurismo",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/ComuniTurismo/FeatureServer"
    },
     {
      "name": "Container_finale",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/Container_finale/FeatureServer"
    },
     {
      "name": "COVID19__Regioni",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/COVID19__Regioni/FeatureServer"
    },
     {
      "name": "COVID19_andamento",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/COVID19_andamento/FeatureServer"
    },
     {
      "name": "COVID19_Andamento_Nazionale",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/COVID19_Andamento_Nazionale/FeatureServer"
    },
     {
      "name": "COVID19_AREE",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/COVID19_AREE/FeatureServer"
    },
     {
      "name": "DPC_COVID19_Andamento_Nazionale",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/DPC_COVID19_Andamento_Nazionale/FeatureServer"
    },
     {
      "name": "DPC_COVID19_dataset_NOTE",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/DPC_COVID19_dataset_NOTE/FeatureServer"
    },
     {
      "name": "DPC_COVID19_Note",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/DPC_COVID19_Note/FeatureServer"
    },
     {
      "name": "DPC_COVID19_Province",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/DPC_COVID19_Province/FeatureServer"
    },
     {
      "name": "DPC_COVID19_Regioni",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/DPC_COVID19_Regioni/FeatureServer"
    },
     {
      "name": "DPC_dati_COVID19_andamento_nazionale",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/DPC_dati_COVID19_andamento_nazionale/FeatureServer"
    },
     {
      "name": "DPC_dati_COVID19_andamento_nazionale_2",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/DPC_dati_COVID19_andamento_nazionale_2/FeatureServer"
    },
     {
      "name": "DPC_dati_COVID19_Province",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/DPC_dati_COVID19_Province/FeatureServer"
    },
     {
      "name": "dpc_province_covid19",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/dpc_province_covid19/FeatureServer"
    },
     {
      "name": "dpc_province_covid19_new",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/dpc_province_covid19_new/FeatureServer"
    },
     {
      "name": "dpc_regioni__covid_19_new2",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/dpc_regioni__covid_19_new2/FeatureServer"
    },
     {
      "name": "dpc_regioni_covid_19_new1",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/dpc_regioni_covid_19_new1/FeatureServer"
    },
     {
      "name": "dpc_regioni_covid19",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/dpc_regioni_covid19/FeatureServer"
    },
     {
      "name": "Interventi_Strade",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/Interventi_Strade/FeatureServer"
    },
     {
      "name": "istituti_scolastici",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/istituti_scolastici/FeatureServer"
    },
     {
      "name": "Limiti_regionali_ISTA_2019",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/Limiti_regionali_ISTA_2019/FeatureServer"
    },
     {
      "name": "LimitiProvince",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/LimitiProvince/FeatureServer"
    },
     {
      "name": "Localita_terremoto",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/Localita_terremoto/FeatureServer"
    },
     {
      "name": "MesseSicurezza_MIBACT",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/MesseSicurezza_MIBACT/FeatureServer"
    },
     {
      "name": "MiBACT_LuoghiCultura",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/MiBACT_LuoghiCultura/FeatureServer"
    },
     {
      "name": "Neiflex_2018_piazzeINR",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/Neiflex_2018_piazzeINR/FeatureServer"
    },
     {
      "name": "Neiflex_2018_scenari",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/Neiflex_2018_scenari/FeatureServer"
    },
     {
      "name": "Neiflex_2018_scenariBBCC",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/Neiflex_2018_scenariBBCC/FeatureServer"
    },
     {
      "name": "Neiflex_2018_sediesercitative",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/Neiflex_2018_sediesercitative/FeatureServer"
    },
     {
      "name": "Progetti_L_B100",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/Progetti_L_B100/FeatureServer"
    },
     {
      "name": "Progetti_L_B500",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/Progetti_L_B500/FeatureServer"
    },
     {
      "name": "Progetti_Lineari",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/Progetti_Lineari/FeatureServer"
    },
     {
      "name": "Progetti_P_B100",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/Progetti_P_B100/FeatureServer"
    },
     {
      "name": "Progetti_P_B500",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/Progetti_P_B500/FeatureServer"
    },
     {
      "name": "Progetti_Puntuali_ANAS",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/Progetti_Puntuali_ANAS/FeatureServer"
    },
     {
      "name": "Progetti_Riepilogo",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/Progetti_Riepilogo/FeatureServer"
    },
     {
      "name": "Progetti_RIEPILOGO5",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/Progetti_RIEPILOGO5/FeatureServer"
    },
     {
      "name": "riepilogo_province",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/riepilogo_province/FeatureServer"
    },
     {
      "name": "SAE_Comunicazione_1",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/SAE_Comunicazione_1/FeatureServer"
    },
     {
      "name": "Servizio_Container",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/Servizio_Container/FeatureServer"
    },
     {
      "name": "Servizio_Demografia",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/Servizio_Demografia/FeatureServer"
    },
     {
      "name": "Servizio_Sfondo",
      "type": "FeatureServer",
      "url": "https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services/Servizio_Sfondo/FeatureServer"
    }]




```
# access "CapoluoghiProvincia" service 
ags_service = ags.getService('CapoluoghiProvincia')
ags_service.list_layers()
```




    ['CapoluoghiProvincia']




```
provincial_capitals = ags_service.layer('CapoluoghiProvincia') #not case sensitive, also supports wildcard search (*)
```


```
provincial_capitals.list_fields()
```




    ['FID',
     'OBJECTID',
     'COD_ISTAT',
     'COD_REG',
     'COD_PRO',
     'PRO_COM',
     'LOC2011',
     'LOC',
     'TIPO_LOC',
     'DENOMINAZI',
     'ALTITUDINE',
     'CENTRO_CL',
     'POPRES',
     'MASCHI',
     'FAMIGLIE',
     'ABITAZIONI',
     'EDIFICI',
     'Shape_Leng',
     'Shape_Area',
     'FID_1',
     'COD_RIP',
     'COD_REG_1',
     'COD_PROV',
     'COD_CM',
     'COD_PCM',
     'DEN_PROV',
     'DEN_CM',
     'DEN_PCM',
     'SIGLA',
     'Shape_Le_1',
     'Shape_Ar_1',
     'ORIG_FID']




```
# export layer to shapefile in WGS 1984 projection
provincial_capitals.export_layer('provincial_capitals.shp', outSR=4326)
```

    Created: "provincial_capitals.shp"





    'provincial_capitals.shp'




```
gpd_provincial_capitals = gpd.read_file('provincial_capitals.shp')
```


```
gpd_provincial_capitals.plot()
```




    <matplotlib.axes._subplots.AxesSubplot at 0x7fa5af21d7f0>




    
![png](03_Retrieving_data_from_spatial_database_infrastructures_files/03_Retrieving_data_from_spatial_database_infrastructures_127_1.png)
    


---
# Exercises
- find the administrative border of "comunità di valle" (community of valley) of Province Autonomous of Trento
- identify all the rivers inside the smallest community of valley of Trentino
- repeat the same exercise with the layer "Comuni Terremotati" (municipalities affected by earthquake) of the italian Civil Protection by choosing the smallest municipality contained on the layer
