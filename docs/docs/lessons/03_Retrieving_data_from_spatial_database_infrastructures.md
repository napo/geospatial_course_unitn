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

    Requirement already satisfied: geopandas in /usr/local/lib/python3.6/dist-packages (0.8.1)
    Requirement already satisfied: pyproj>=2.2.0 in /usr/local/lib/python3.6/dist-packages (from geopandas) (2.6.1.post1)
    Requirement already satisfied: fiona in /usr/local/lib/python3.6/dist-packages (from geopandas) (1.8.17)
    Requirement already satisfied: pandas>=0.23.0 in /usr/local/lib/python3.6/dist-packages (from geopandas) (1.1.2)
    Requirement already satisfied: shapely in /usr/local/lib/python3.6/dist-packages (from geopandas) (1.7.1)
    Requirement already satisfied: click-plugins>=1.0 in /usr/local/lib/python3.6/dist-packages (from fiona->geopandas) (1.1.1)
    Requirement already satisfied: munch in /usr/local/lib/python3.6/dist-packages (from fiona->geopandas) (2.5.0)
    Requirement already satisfied: click<8,>=4.0 in /usr/local/lib/python3.6/dist-packages (from fiona->geopandas) (7.1.2)
    Requirement already satisfied: six>=1.7 in /usr/local/lib/python3.6/dist-packages (from fiona->geopandas) (1.15.0)
    Requirement already satisfied: attrs>=17 in /usr/local/lib/python3.6/dist-packages (from fiona->geopandas) (20.2.0)
    Requirement already satisfied: cligj>=0.5 in /usr/local/lib/python3.6/dist-packages (from fiona->geopandas) (0.5.0)
    Requirement already satisfied: numpy>=1.15.4 in /usr/local/lib/python3.6/dist-packages (from pandas>=0.23.0->geopandas) (1.18.5)
    Requirement already satisfied: pytz>=2017.2 in /usr/local/lib/python3.6/dist-packages (from pandas>=0.23.0->geopandas) (2018.9)
    Requirement already satisfied: python-dateutil>=2.7.3 in /usr/local/lib/python3.6/dist-packages (from pandas>=0.23.0->geopandas) (2.8.1)



```
# only for visualization purpouse
!pip install git+https://github.com/python-visualization/folium
```

    Collecting git+https://github.com/python-visualization/folium
      Cloning https://github.com/python-visualization/folium to /tmp/pip-req-build-bi_z7usy
      Running command git clone -q https://github.com/python-visualization/folium /tmp/pip-req-build-bi_z7usy
    Requirement already satisfied (use --upgrade to upgrade): folium==0.11.0+20.gb70efc6 from git+https://github.com/python-visualization/folium in /usr/local/lib/python3.6/dist-packages
    Requirement already satisfied: branca>=0.3.0 in /usr/local/lib/python3.6/dist-packages (from folium==0.11.0+20.gb70efc6) (0.4.1)
    Requirement already satisfied: jinja2>=2.9 in /usr/local/lib/python3.6/dist-packages (from folium==0.11.0+20.gb70efc6) (2.11.2)
    Requirement already satisfied: numpy in /usr/local/lib/python3.6/dist-packages (from folium==0.11.0+20.gb70efc6) (1.18.5)
    Requirement already satisfied: requests in /usr/local/lib/python3.6/dist-packages (from folium==0.11.0+20.gb70efc6) (2.23.0)
    Requirement already satisfied: MarkupSafe>=0.23 in /usr/local/lib/python3.6/dist-packages (from jinja2>=2.9->folium==0.11.0+20.gb70efc6) (1.1.1)
    Requirement already satisfied: idna<3,>=2.5 in /usr/local/lib/python3.6/dist-packages (from requests->folium==0.11.0+20.gb70efc6) (2.10)
    Requirement already satisfied: certifi>=2017.4.17 in /usr/local/lib/python3.6/dist-packages (from requests->folium==0.11.0+20.gb70efc6) (2020.6.20)
    Requirement already satisfied: chardet<4,>=3.0.2 in /usr/local/lib/python3.6/dist-packages (from requests->folium==0.11.0+20.gb70efc6) (3.0.4)
    Requirement already satisfied: urllib3!=1.25.0,!=1.25.1,<1.26,>=1.21.1 in /usr/local/lib/python3.6/dist-packages (from requests->folium==0.11.0+20.gb70efc6) (1.24.3)
    Building wheels for collected packages: folium
      Building wheel for folium (setup.py) ... [?25l[?25hdone
      Created wheel for folium: filename=folium-0.11.0+20.gb70efc6-py2.py3-none-any.whl size=97529 sha256=e2eaa5056580dc3c40a33859f4ec77bc2734a05fcf5d69af876e3d3618af710f
      Stored in directory: /tmp/pip-ephem-wheel-cache-wk6zj1bq/wheels/1e/e1/75/ecbc91fd5dd5d90befb0b533bf7492d38acffa033310731862
    Successfully built folium



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




    <matplotlib.axes._subplots.AxesSubplot at 0x7f7d1054c908>




    
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




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe src="about:blank" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" data-html=PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgCiAgICAgICAgPHNjcmlwdD4KICAgICAgICAgICAgTF9OT19UT1VDSCA9IGZhbHNlOwogICAgICAgICAgICBMX0RJU0FCTEVfM0QgPSBmYWxzZTsKICAgICAgICA8L3NjcmlwdD4KICAgIAogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjYuMC9kaXN0L2xlYWZsZXQuanMiPjwvc2NyaXB0PgogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY29kZS5qcXVlcnkuY29tL2pxdWVyeS0xLjEyLjQubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9qcy9ib290c3RyYXAubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5qcyI+PC9zY3JpcHQ+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjYuMC9kaXN0L2xlYWZsZXQuY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vYm9vdHN0cmFwLzMuMi4wL2Nzcy9ib290c3RyYXAubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLXRoZW1lLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9mb250LWF3ZXNvbWUvNC42LjMvY3NzL2ZvbnQtYXdlc29tZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuanMuY2xvdWRmbGFyZS5jb20vYWpheC9saWJzL0xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLzIuMC4yL2xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvZ2gvcHl0aG9uLXZpc3VhbGl6YXRpb24vZm9saXVtL2ZvbGl1bS90ZW1wbGF0ZXMvbGVhZmxldC5hd2Vzb21lLnJvdGF0ZS5taW4uY3NzIi8+CiAgICA8c3R5bGU+aHRtbCwgYm9keSB7d2lkdGg6IDEwMCU7aGVpZ2h0OiAxMDAlO21hcmdpbjogMDtwYWRkaW5nOiAwO308L3N0eWxlPgogICAgPHN0eWxlPiNtYXAge3Bvc2l0aW9uOmFic29sdXRlO3RvcDowO2JvdHRvbTowO3JpZ2h0OjA7bGVmdDowO308L3N0eWxlPgogICAgCiAgICAgICAgICAgIDxtZXRhIG5hbWU9InZpZXdwb3J0IiBjb250ZW50PSJ3aWR0aD1kZXZpY2Utd2lkdGgsCiAgICAgICAgICAgICAgICBpbml0aWFsLXNjYWxlPTEuMCwgbWF4aW11bS1zY2FsZT0xLjAsIHVzZXItc2NhbGFibGU9bm8iIC8+CiAgICAgICAgICAgIDxzdHlsZT4KICAgICAgICAgICAgICAgICNtYXBfNGZjMDYxN2MyZGYwNGIyZGIxMmIzZmVjOTllNzA5MjcgewogICAgICAgICAgICAgICAgICAgIHBvc2l0aW9uOiByZWxhdGl2ZTsKICAgICAgICAgICAgICAgICAgICB3aWR0aDogMTAwLjAlOwogICAgICAgICAgICAgICAgICAgIGhlaWdodDogMTAwLjAlOwogICAgICAgICAgICAgICAgICAgIGxlZnQ6IDAuMCU7CiAgICAgICAgICAgICAgICAgICAgdG9wOiAwLjAlOwogICAgICAgICAgICAgICAgfQogICAgICAgICAgICA8L3N0eWxlPgogICAgICAgIAo8L2hlYWQ+Cjxib2R5PiAgICAKICAgIAogICAgICAgICAgICA8ZGl2IGNsYXNzPSJmb2xpdW0tbWFwIiBpZD0ibWFwXzRmYzA2MTdjMmRmMDRiMmRiMTJiM2ZlYzk5ZTcwOTI3IiA+PC9kaXY+CiAgICAgICAgCjwvYm9keT4KPHNjcmlwdD4gICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcF80ZmMwNjE3YzJkZjA0YjJkYjEyYjNmZWM5OWU3MDkyNyA9IEwubWFwKAogICAgICAgICAgICAgICAgIm1hcF80ZmMwNjE3YzJkZjA0YjJkYjEyYjNmZWM5OWU3MDkyNyIsCiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgY2VudGVyOiBbMjUuODM5MTEwMDAwMDAwMDYyLCAtODAuMTg0Njc5OTk5OTk5OTZdLAogICAgICAgICAgICAgICAgICAgIGNyczogTC5DUlMuRVBTRzM4NTcsCiAgICAgICAgICAgICAgICAgICAgem9vbTogMTgsCiAgICAgICAgICAgICAgICAgICAgem9vbUNvbnRyb2w6IHRydWUsCiAgICAgICAgICAgICAgICAgICAgcHJlZmVyQ2FudmFzOiBmYWxzZSwKICAgICAgICAgICAgICAgIH0KICAgICAgICAgICAgKTsKCiAgICAgICAgICAgIAoKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgdGlsZV9sYXllcl83NzBiMWQ2YzIxYWI0ZWQzOWUxZDEyNzVmZDY4NTM1YiA9IEwudGlsZUxheWVyKAogICAgICAgICAgICAgICAgImh0dHBzOi8ve3N9LnRpbGUub3BlbnN0cmVldG1hcC5vcmcve3p9L3t4fS97eX0ucG5nIiwKICAgICAgICAgICAgICAgIHsiYXR0cmlidXRpb24iOiAiRGF0YSBieSBcdTAwMjZjb3B5OyBcdTAwM2NhIGhyZWY9XCJodHRwOi8vb3BlbnN0cmVldG1hcC5vcmdcIlx1MDAzZU9wZW5TdHJlZXRNYXBcdTAwM2MvYVx1MDAzZSwgdW5kZXIgXHUwMDNjYSBocmVmPVwiaHR0cDovL3d3dy5vcGVuc3RyZWV0bWFwLm9yZy9jb3B5cmlnaHRcIlx1MDAzZU9EYkxcdTAwM2MvYVx1MDAzZS4iLCAiZGV0ZWN0UmV0aW5hIjogZmFsc2UsICJtYXhOYXRpdmVab29tIjogMTgsICJtYXhab29tIjogMTgsICJtaW5ab29tIjogMCwgIm5vV3JhcCI6IGZhbHNlLCAib3BhY2l0eSI6IDEsICJzdWJkb21haW5zIjogImFiYyIsICJ0bXMiOiBmYWxzZX0KICAgICAgICAgICAgKS5hZGRUbyhtYXBfNGZjMDYxN2MyZGYwNGIyZGIxMmIzZmVjOTllNzA5MjcpOwogICAgICAgIAogICAgCiAgICAgICAgZnVuY3Rpb24gZ2VvX2pzb25fN2E0MTFlM2ZiNTBiNDg0N2IyYjdkMWU0MDhlYWQxM2Vfb25FYWNoRmVhdHVyZShmZWF0dXJlLCBsYXllcikgewogICAgICAgICAgICBsYXllci5vbih7CiAgICAgICAgICAgIH0pOwogICAgICAgIH07CiAgICAgICAgdmFyIGdlb19qc29uXzdhNDExZTNmYjUwYjQ4NDdiMmI3ZDFlNDA4ZWFkMTNlID0gTC5nZW9Kc29uKG51bGwsIHsKICAgICAgICAgICAgICAgIG9uRWFjaEZlYXR1cmU6IGdlb19qc29uXzdhNDExZTNmYjUwYjQ4NDdiMmI3ZDFlNDA4ZWFkMTNlX29uRWFjaEZlYXR1cmUsCiAgICAgICAgICAgIAogICAgICAgIH0pOwoKICAgICAgICBmdW5jdGlvbiBnZW9fanNvbl83YTQxMWUzZmI1MGI0ODQ3YjJiN2QxZTQwOGVhZDEzZV9hZGQgKGRhdGEpIHsKICAgICAgICAgICAgZ2VvX2pzb25fN2E0MTFlM2ZiNTBiNDg0N2IyYjdkMWU0MDhlYWQxM2UKICAgICAgICAgICAgICAgIC5hZGREYXRhKGRhdGEpCiAgICAgICAgICAgICAgICAuYWRkVG8obWFwXzRmYzA2MTdjMmRmMDRiMmRiMTJiM2ZlYzk5ZTcwOTI3KTsKICAgICAgICB9CiAgICAgICAgICAgIGdlb19qc29uXzdhNDExZTNmYjUwYjQ4NDdiMmI3ZDFlNDA4ZWFkMTNlX2FkZCh7ImZlYXR1cmVzIjogW3siZ2VvbWV0cnkiOiB7ImNvb3JkaW5hdGVzIjogWy04MC4xODQ2Nzk5OTk5OTk5NiwgMjUuODM5MTEwMDAwMDAwMDYyXSwgInR5cGUiOiAiUG9pbnQifSwgImlkIjogIjAiLCAicHJvcGVydGllcyI6IHsiYWRkcmVzcyI6ICJWaWEgVmVyZGkifSwgInR5cGUiOiAiRmVhdHVyZSJ9XSwgInR5cGUiOiAiRmVhdHVyZUNvbGxlY3Rpb24ifSk7CgogICAgICAgIAo8L3NjcmlwdD4= onload="this.contentDocument.open();this.contentDocument.write(atob(this.getAttribute('data-html')));this.contentDocument.close();" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



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




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe src="about:blank" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" data-html=PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgCiAgICAgICAgPHNjcmlwdD4KICAgICAgICAgICAgTF9OT19UT1VDSCA9IGZhbHNlOwogICAgICAgICAgICBMX0RJU0FCTEVfM0QgPSBmYWxzZTsKICAgICAgICA8L3NjcmlwdD4KICAgIAogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjYuMC9kaXN0L2xlYWZsZXQuanMiPjwvc2NyaXB0PgogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY29kZS5qcXVlcnkuY29tL2pxdWVyeS0xLjEyLjQubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9qcy9ib290c3RyYXAubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5qcyI+PC9zY3JpcHQ+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjYuMC9kaXN0L2xlYWZsZXQuY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vYm9vdHN0cmFwLzMuMi4wL2Nzcy9ib290c3RyYXAubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLXRoZW1lLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9mb250LWF3ZXNvbWUvNC42LjMvY3NzL2ZvbnQtYXdlc29tZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuanMuY2xvdWRmbGFyZS5jb20vYWpheC9saWJzL0xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLzIuMC4yL2xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvZ2gvcHl0aG9uLXZpc3VhbGl6YXRpb24vZm9saXVtL2ZvbGl1bS90ZW1wbGF0ZXMvbGVhZmxldC5hd2Vzb21lLnJvdGF0ZS5taW4uY3NzIi8+CiAgICA8c3R5bGU+aHRtbCwgYm9keSB7d2lkdGg6IDEwMCU7aGVpZ2h0OiAxMDAlO21hcmdpbjogMDtwYWRkaW5nOiAwO308L3N0eWxlPgogICAgPHN0eWxlPiNtYXAge3Bvc2l0aW9uOmFic29sdXRlO3RvcDowO2JvdHRvbTowO3JpZ2h0OjA7bGVmdDowO308L3N0eWxlPgogICAgCiAgICAgICAgICAgIDxtZXRhIG5hbWU9InZpZXdwb3J0IiBjb250ZW50PSJ3aWR0aD1kZXZpY2Utd2lkdGgsCiAgICAgICAgICAgICAgICBpbml0aWFsLXNjYWxlPTEuMCwgbWF4aW11bS1zY2FsZT0xLjAsIHVzZXItc2NhbGFibGU9bm8iIC8+CiAgICAgICAgICAgIDxzdHlsZT4KICAgICAgICAgICAgICAgICNtYXBfY2U0MzVmYmNiNzQ5NGQ3MTk2NTc5ZGE1NTdlZDUyZTIgewogICAgICAgICAgICAgICAgICAgIHBvc2l0aW9uOiByZWxhdGl2ZTsKICAgICAgICAgICAgICAgICAgICB3aWR0aDogMTAwLjAlOwogICAgICAgICAgICAgICAgICAgIGhlaWdodDogMTAwLjAlOwogICAgICAgICAgICAgICAgICAgIGxlZnQ6IDAuMCU7CiAgICAgICAgICAgICAgICAgICAgdG9wOiAwLjAlOwogICAgICAgICAgICAgICAgfQogICAgICAgICAgICA8L3N0eWxlPgogICAgICAgIAo8L2hlYWQ+Cjxib2R5PiAgICAKICAgIAogICAgICAgICAgICA8ZGl2IGNsYXNzPSJmb2xpdW0tbWFwIiBpZD0ibWFwX2NlNDM1ZmJjYjc0OTRkNzE5NjU3OWRhNTU3ZWQ1MmUyIiA+PC9kaXY+CiAgICAgICAgCjwvYm9keT4KPHNjcmlwdD4gICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcF9jZTQzNWZiY2I3NDk0ZDcxOTY1NzlkYTU1N2VkNTJlMiA9IEwubWFwKAogICAgICAgICAgICAgICAgIm1hcF9jZTQzNWZiY2I3NDk0ZDcxOTY1NzlkYTU1N2VkNTJlMiIsCiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgY2VudGVyOiBbNDYuMDY2NjUwMDA2MDE5ODEsIDExLjExOTY1OTk4NTkzNzAzNF0sCiAgICAgICAgICAgICAgICAgICAgY3JzOiBMLkNSUy5FUFNHMzg1NywKICAgICAgICAgICAgICAgICAgICB6b29tOiAxOCwKICAgICAgICAgICAgICAgICAgICB6b29tQ29udHJvbDogdHJ1ZSwKICAgICAgICAgICAgICAgICAgICBwcmVmZXJDYW52YXM6IGZhbHNlLAogICAgICAgICAgICAgICAgfQogICAgICAgICAgICApOwoKICAgICAgICAgICAgCgogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciB0aWxlX2xheWVyX2Q3ZTVmODUwN2NhYTQ1MTZiMzE4NjhkNGRlZTE5NTc1ID0gTC50aWxlTGF5ZXIoCiAgICAgICAgICAgICAgICAiaHR0cHM6Ly97c30udGlsZS5vcGVuc3RyZWV0bWFwLm9yZy97en0ve3h9L3t5fS5wbmciLAogICAgICAgICAgICAgICAgeyJhdHRyaWJ1dGlvbiI6ICJEYXRhIGJ5IFx1MDAyNmNvcHk7IFx1MDAzY2EgaHJlZj1cImh0dHA6Ly9vcGVuc3RyZWV0bWFwLm9yZ1wiXHUwMDNlT3BlblN0cmVldE1hcFx1MDAzYy9hXHUwMDNlLCB1bmRlciBcdTAwM2NhIGhyZWY9XCJodHRwOi8vd3d3Lm9wZW5zdHJlZXRtYXAub3JnL2NvcHlyaWdodFwiXHUwMDNlT0RiTFx1MDAzYy9hXHUwMDNlLiIsICJkZXRlY3RSZXRpbmEiOiBmYWxzZSwgIm1heE5hdGl2ZVpvb20iOiAxOCwgIm1heFpvb20iOiAxOCwgIm1pblpvb20iOiAwLCAibm9XcmFwIjogZmFsc2UsICJvcGFjaXR5IjogMSwgInN1YmRvbWFpbnMiOiAiYWJjIiwgInRtcyI6IGZhbHNlfQogICAgICAgICAgICApLmFkZFRvKG1hcF9jZTQzNWZiY2I3NDk0ZDcxOTY1NzlkYTU1N2VkNTJlMik7CiAgICAgICAgCiAgICAKICAgICAgICBmdW5jdGlvbiBnZW9fanNvbl8xZTAwYWNjMmU0OGI0YTJhYTlmODIzNWJiMWQzMzlkN19vbkVhY2hGZWF0dXJlKGZlYXR1cmUsIGxheWVyKSB7CiAgICAgICAgICAgIGxheWVyLm9uKHsKICAgICAgICAgICAgfSk7CiAgICAgICAgfTsKICAgICAgICB2YXIgZ2VvX2pzb25fMWUwMGFjYzJlNDhiNGEyYWE5ZjgyMzViYjFkMzM5ZDcgPSBMLmdlb0pzb24obnVsbCwgewogICAgICAgICAgICAgICAgb25FYWNoRmVhdHVyZTogZ2VvX2pzb25fMWUwMGFjYzJlNDhiNGEyYWE5ZjgyMzViYjFkMzM5ZDdfb25FYWNoRmVhdHVyZSwKICAgICAgICAgICAgCiAgICAgICAgfSk7CgogICAgICAgIGZ1bmN0aW9uIGdlb19qc29uXzFlMDBhY2MyZTQ4YjRhMmFhOWY4MjM1YmIxZDMzOWQ3X2FkZCAoZGF0YSkgewogICAgICAgICAgICBnZW9fanNvbl8xZTAwYWNjMmU0OGI0YTJhYTlmODIzNWJiMWQzMzlkNwogICAgICAgICAgICAgICAgLmFkZERhdGEoZGF0YSkKICAgICAgICAgICAgICAgIC5hZGRUbyhtYXBfY2U0MzVmYmNiNzQ5NGQ3MTk2NTc5ZGE1NTdlZDUyZTIpOwogICAgICAgIH0KICAgICAgICAgICAgZ2VvX2pzb25fMWUwMGFjYzJlNDhiNGEyYWE5ZjgyMzViYjFkMzM5ZDdfYWRkKHsiZmVhdHVyZXMiOiBbeyJnZW9tZXRyeSI6IHsiY29vcmRpbmF0ZXMiOiBbMTEuMTE5NjU5OTg1OTM3MDM0LCA0Ni4wNjY2NTAwMDYwMTk4MV0sICJ0eXBlIjogIlBvaW50In0sICJpZCI6ICIwIiwgInByb3BlcnRpZXMiOiB7ImFkZHJlc3MiOiAiVmlhIEdpdXNlcHBlIFZlcmRpIDI2LCAzODEyMiwgVHJlbnRvIn0sICJ0eXBlIjogIkZlYXR1cmUifV0sICJ0eXBlIjogIkZlYXR1cmVDb2xsZWN0aW9uIn0pOwoKICAgICAgICAKPC9zY3JpcHQ+ onload="this.contentDocument.open();this.contentDocument.write(atob(this.getAttribute('data-html')));this.contentDocument.close();" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



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




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe src="about:blank" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" data-html=PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgCiAgICAgICAgPHNjcmlwdD4KICAgICAgICAgICAgTF9OT19UT1VDSCA9IGZhbHNlOwogICAgICAgICAgICBMX0RJU0FCTEVfM0QgPSBmYWxzZTsKICAgICAgICA8L3NjcmlwdD4KICAgIAogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjYuMC9kaXN0L2xlYWZsZXQuanMiPjwvc2NyaXB0PgogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY29kZS5qcXVlcnkuY29tL2pxdWVyeS0xLjEyLjQubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9qcy9ib290c3RyYXAubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5qcyI+PC9zY3JpcHQ+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjYuMC9kaXN0L2xlYWZsZXQuY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vYm9vdHN0cmFwLzMuMi4wL2Nzcy9ib290c3RyYXAubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLXRoZW1lLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9mb250LWF3ZXNvbWUvNC42LjMvY3NzL2ZvbnQtYXdlc29tZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuanMuY2xvdWRmbGFyZS5jb20vYWpheC9saWJzL0xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLzIuMC4yL2xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvZ2gvcHl0aG9uLXZpc3VhbGl6YXRpb24vZm9saXVtL2ZvbGl1bS90ZW1wbGF0ZXMvbGVhZmxldC5hd2Vzb21lLnJvdGF0ZS5taW4uY3NzIi8+CiAgICA8c3R5bGU+aHRtbCwgYm9keSB7d2lkdGg6IDEwMCU7aGVpZ2h0OiAxMDAlO21hcmdpbjogMDtwYWRkaW5nOiAwO308L3N0eWxlPgogICAgPHN0eWxlPiNtYXAge3Bvc2l0aW9uOmFic29sdXRlO3RvcDowO2JvdHRvbTowO3JpZ2h0OjA7bGVmdDowO308L3N0eWxlPgogICAgCiAgICAgICAgICAgIDxtZXRhIG5hbWU9InZpZXdwb3J0IiBjb250ZW50PSJ3aWR0aD1kZXZpY2Utd2lkdGgsCiAgICAgICAgICAgICAgICBpbml0aWFsLXNjYWxlPTEuMCwgbWF4aW11bS1zY2FsZT0xLjAsIHVzZXItc2NhbGFibGU9bm8iIC8+CiAgICAgICAgICAgIDxzdHlsZT4KICAgICAgICAgICAgICAgICNtYXBfNjZhZjMzNWE0YjkyNDE3YjlmNDFkMmZhN2ViYWVhNTggewogICAgICAgICAgICAgICAgICAgIHBvc2l0aW9uOiByZWxhdGl2ZTsKICAgICAgICAgICAgICAgICAgICB3aWR0aDogMTAwLjAlOwogICAgICAgICAgICAgICAgICAgIGhlaWdodDogMTAwLjAlOwogICAgICAgICAgICAgICAgICAgIGxlZnQ6IDAuMCU7CiAgICAgICAgICAgICAgICAgICAgdG9wOiAwLjAlOwogICAgICAgICAgICAgICAgfQogICAgICAgICAgICA8L3N0eWxlPgogICAgICAgIAo8L2hlYWQ+Cjxib2R5PiAgICAKICAgIAogICAgICAgICAgICA8ZGl2IGNsYXNzPSJmb2xpdW0tbWFwIiBpZD0ibWFwXzY2YWYzMzVhNGI5MjQxN2I5ZjQxZDJmYTdlYmFlYTU4IiA+PC9kaXY+CiAgICAgICAgCjwvYm9keT4KPHNjcmlwdD4gICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcF82NmFmMzM1YTRiOTI0MTdiOWY0MWQyZmE3ZWJhZWE1OCA9IEwubWFwKAogICAgICAgICAgICAgICAgIm1hcF82NmFmMzM1YTRiOTI0MTdiOWY0MWQyZmE3ZWJhZWE1OCIsCiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgY2VudGVyOiBbNDYuMDY2NDEzNDk5OTk5OTk2LCAxMS4xMTk2NTk5ODU5MzcwMzRdLAogICAgICAgICAgICAgICAgICAgIGNyczogTC5DUlMuRVBTRzM4NTcsCiAgICAgICAgICAgICAgICAgICAgem9vbTogMTgsCiAgICAgICAgICAgICAgICAgICAgem9vbUNvbnRyb2w6IHRydWUsCiAgICAgICAgICAgICAgICAgICAgcHJlZmVyQ2FudmFzOiBmYWxzZSwKICAgICAgICAgICAgICAgIH0KICAgICAgICAgICAgKTsKCiAgICAgICAgICAgIAoKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgdGlsZV9sYXllcl8zNDFlYjI1OTVjMzA0ZWMwYTBlZDBkMzZlZmE2MGNiOSA9IEwudGlsZUxheWVyKAogICAgICAgICAgICAgICAgImh0dHBzOi8ve3N9LnRpbGUub3BlbnN0cmVldG1hcC5vcmcve3p9L3t4fS97eX0ucG5nIiwKICAgICAgICAgICAgICAgIHsiYXR0cmlidXRpb24iOiAiRGF0YSBieSBcdTAwMjZjb3B5OyBcdTAwM2NhIGhyZWY9XCJodHRwOi8vb3BlbnN0cmVldG1hcC5vcmdcIlx1MDAzZU9wZW5TdHJlZXRNYXBcdTAwM2MvYVx1MDAzZSwgdW5kZXIgXHUwMDNjYSBocmVmPVwiaHR0cDovL3d3dy5vcGVuc3RyZWV0bWFwLm9yZy9jb3B5cmlnaHRcIlx1MDAzZU9EYkxcdTAwM2MvYVx1MDAzZS4iLCAiZGV0ZWN0UmV0aW5hIjogZmFsc2UsICJtYXhOYXRpdmVab29tIjogMTgsICJtYXhab29tIjogMTgsICJtaW5ab29tIjogMCwgIm5vV3JhcCI6IGZhbHNlLCAib3BhY2l0eSI6IDEsICJzdWJkb21haW5zIjogImFiYyIsICJ0bXMiOiBmYWxzZX0KICAgICAgICAgICAgKS5hZGRUbyhtYXBfNjZhZjMzNWE0YjkyNDE3YjlmNDFkMmZhN2ViYWVhNTgpOwogICAgICAgIAogICAgCiAgICAgICAgZnVuY3Rpb24gZ2VvX2pzb25fZTg2MThmMGQ0NzdjNGIzZjgyOTRlNjE2ZTRmMTUyYzNfb25FYWNoRmVhdHVyZShmZWF0dXJlLCBsYXllcikgewogICAgICAgICAgICBsYXllci5vbih7CiAgICAgICAgICAgIH0pOwogICAgICAgIH07CiAgICAgICAgdmFyIGdlb19qc29uX2U4NjE4ZjBkNDc3YzRiM2Y4Mjk0ZTYxNmU0ZjE1MmMzID0gTC5nZW9Kc29uKG51bGwsIHsKICAgICAgICAgICAgICAgIG9uRWFjaEZlYXR1cmU6IGdlb19qc29uX2U4NjE4ZjBkNDc3YzRiM2Y4Mjk0ZTYxNmU0ZjE1MmMzX29uRWFjaEZlYXR1cmUsCiAgICAgICAgICAgIAogICAgICAgIH0pOwoKICAgICAgICBmdW5jdGlvbiBnZW9fanNvbl9lODYxOGYwZDQ3N2M0YjNmODI5NGU2MTZlNGYxNTJjM19hZGQgKGRhdGEpIHsKICAgICAgICAgICAgZ2VvX2pzb25fZTg2MThmMGQ0NzdjNGIzZjgyOTRlNjE2ZTRmMTUyYzMKICAgICAgICAgICAgICAgIC5hZGREYXRhKGRhdGEpCiAgICAgICAgICAgICAgICAuYWRkVG8obWFwXzY2YWYzMzVhNGI5MjQxN2I5ZjQxZDJmYTdlYmFlYTU4KTsKICAgICAgICB9CiAgICAgICAgICAgIGdlb19qc29uX2U4NjE4ZjBkNDc3YzRiM2Y4Mjk0ZTYxNmU0ZjE1MmMzX2FkZCh7ImZlYXR1cmVzIjogW3siZ2VvbWV0cnkiOiB7ImNvb3JkaW5hdGVzIjogWzExLjExOTcwNTY0NDY4MDY0NiwgNDYuMDY2NDEzNDk5OTk5OTk2XSwgInR5cGUiOiAiUG9pbnQifSwgImlkIjogIjAiLCAicHJvcGVydGllcyI6IHsiYWRkcmVzcyI6ICJEaXBhcnRpbWVudG8gZGkgU29jaW9sb2dpYSBlIFJpY2VyY2EgU29jaWFsZSwgMjYsIFZpYSBHaXVzZXBwZSBWZXJkaSwgQ2VudHJvIHN0b3JpY28gVHJlbnRvLCBUcmVudG8sIFRlcnJpdG9yaW8gVmFsIGRcdTAwMjdBZGlnZSwgUHJvdmluY2lhIGRpIFRyZW50bywgVHJlbnRpbm8tQWx0byBBZGlnZS9TXHUwMGZjZHRpcm9sLCAzODEyMiwgSXRhbGlhIn0sICJ0eXBlIjogIkZlYXR1cmUifV0sICJ0eXBlIjogIkZlYXR1cmVDb2xsZWN0aW9uIn0pOwoKICAgICAgICAKPC9zY3JpcHQ+ onload="this.contentDocument.open();this.contentDocument.write(atob(this.getAttribute('data-html')));this.contentDocument.close();" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



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

    Requirement already satisfied: owslib in /usr/local/lib/python3.6/dist-packages (0.20.0)
    Requirement already satisfied: requests>=1.0 in /usr/local/lib/python3.6/dist-packages (from owslib) (2.23.0)
    Requirement already satisfied: pyproj>=2 in /usr/local/lib/python3.6/dist-packages (from owslib) (2.6.1.post1)
    Requirement already satisfied: python-dateutil>=1.5 in /usr/local/lib/python3.6/dist-packages (from owslib) (2.8.1)
    Requirement already satisfied: pytz in /usr/local/lib/python3.6/dist-packages (from owslib) (2018.9)
    Requirement already satisfied: pyyaml in /usr/local/lib/python3.6/dist-packages (from owslib) (3.13)
    Requirement already satisfied: chardet<4,>=3.0.2 in /usr/local/lib/python3.6/dist-packages (from requests>=1.0->owslib) (3.0.4)
    Requirement already satisfied: certifi>=2017.4.17 in /usr/local/lib/python3.6/dist-packages (from requests>=1.0->owslib) (2020.6.20)
    Requirement already satisfied: urllib3!=1.25.0,!=1.25.1,<1.26,>=1.21.1 in /usr/local/lib/python3.6/dist-packages (from requests>=1.0->owslib) (1.24.3)
    Requirement already satisfied: idna<3,>=2.5 in /usr/local/lib/python3.6/dist-packages (from requests>=1.0->owslib) (2.10)
    Requirement already satisfied: six>=1.5 in /usr/local/lib/python3.6/dist-packages (from python-dateutil>=1.5->owslib) (1.15.0)



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




    <matplotlib.axes._subplots.AxesSubplot at 0x7f7d10498400>




    
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
    Wall time: 5.96 µs



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




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe src="about:blank" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" data-html=PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgCiAgICAgICAgPHNjcmlwdD4KICAgICAgICAgICAgTF9OT19UT1VDSCA9IGZhbHNlOwogICAgICAgICAgICBMX0RJU0FCTEVfM0QgPSBmYWxzZTsKICAgICAgICA8L3NjcmlwdD4KICAgIAogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjYuMC9kaXN0L2xlYWZsZXQuanMiPjwvc2NyaXB0PgogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY29kZS5qcXVlcnkuY29tL2pxdWVyeS0xLjEyLjQubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9qcy9ib290c3RyYXAubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5qcyI+PC9zY3JpcHQ+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjYuMC9kaXN0L2xlYWZsZXQuY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vYm9vdHN0cmFwLzMuMi4wL2Nzcy9ib290c3RyYXAubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLXRoZW1lLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9mb250LWF3ZXNvbWUvNC42LjMvY3NzL2ZvbnQtYXdlc29tZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuanMuY2xvdWRmbGFyZS5jb20vYWpheC9saWJzL0xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLzIuMC4yL2xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvZ2gvcHl0aG9uLXZpc3VhbGl6YXRpb24vZm9saXVtL2ZvbGl1bS90ZW1wbGF0ZXMvbGVhZmxldC5hd2Vzb21lLnJvdGF0ZS5taW4uY3NzIi8+CiAgICA8c3R5bGU+aHRtbCwgYm9keSB7d2lkdGg6IDEwMCU7aGVpZ2h0OiAxMDAlO21hcmdpbjogMDtwYWRkaW5nOiAwO308L3N0eWxlPgogICAgPHN0eWxlPiNtYXAge3Bvc2l0aW9uOmFic29sdXRlO3RvcDowO2JvdHRvbTowO3JpZ2h0OjA7bGVmdDowO308L3N0eWxlPgogICAgCiAgICAgICAgICAgIDxtZXRhIG5hbWU9InZpZXdwb3J0IiBjb250ZW50PSJ3aWR0aD1kZXZpY2Utd2lkdGgsCiAgICAgICAgICAgICAgICBpbml0aWFsLXNjYWxlPTEuMCwgbWF4aW11bS1zY2FsZT0xLjAsIHVzZXItc2NhbGFibGU9bm8iIC8+CiAgICAgICAgICAgIDxzdHlsZT4KICAgICAgICAgICAgICAgICNtYXBfMzYxZDIyY2ZmNGFiNGVhYWE1YzdiNGU3NmI4ZTk1NWMgewogICAgICAgICAgICAgICAgICAgIHBvc2l0aW9uOiByZWxhdGl2ZTsKICAgICAgICAgICAgICAgICAgICB3aWR0aDogMTAwLjAlOwogICAgICAgICAgICAgICAgICAgIGhlaWdodDogMTAwLjAlOwogICAgICAgICAgICAgICAgICAgIGxlZnQ6IDAuMCU7CiAgICAgICAgICAgICAgICAgICAgdG9wOiAwLjAlOwogICAgICAgICAgICAgICAgfQogICAgICAgICAgICA8L3N0eWxlPgogICAgICAgIAo8L2hlYWQ+Cjxib2R5PiAgICAKICAgIAogICAgICAgICAgICA8ZGl2IGNsYXNzPSJmb2xpdW0tbWFwIiBpZD0ibWFwXzM2MWQyMmNmZjRhYjRlYWFhNWM3YjRlNzZiOGU5NTVjIiA+PC9kaXY+CiAgICAgICAgCjwvYm9keT4KPHNjcmlwdD4gICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcF8zNjFkMjJjZmY0YWI0ZWFhYTVjN2I0ZTc2YjhlOTU1YyA9IEwubWFwKAogICAgICAgICAgICAgICAgIm1hcF8zNjFkMjJjZmY0YWI0ZWFhYTVjN2I0ZTc2YjhlOTU1YyIsCiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgY2VudGVyOiBbMTAuODYxODQ0OTMyMDU4OTc5LCA0NS44NzM0OTA5MDk5NDg0MTZdLAogICAgICAgICAgICAgICAgICAgIGNyczogTC5DUlMuRVBTRzM4NTcsCiAgICAgICAgICAgICAgICAgICAgem9vbTogMTIsCiAgICAgICAgICAgICAgICAgICAgem9vbUNvbnRyb2w6IHRydWUsCiAgICAgICAgICAgICAgICAgICAgcHJlZmVyQ2FudmFzOiBmYWxzZSwKICAgICAgICAgICAgICAgIH0KICAgICAgICAgICAgKTsKCiAgICAgICAgICAgIAoKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgdGlsZV9sYXllcl8yZGQyYTE0MjNmZTA0NjhjODI3ZDZkYjczNzUwNTVhMyA9IEwudGlsZUxheWVyKAogICAgICAgICAgICAgICAgImh0dHBzOi8ve3N9LnRpbGUub3BlbnN0cmVldG1hcC5vcmcve3p9L3t4fS97eX0ucG5nIiwKICAgICAgICAgICAgICAgIHsiYXR0cmlidXRpb24iOiAiRGF0YSBieSBcdTAwMjZjb3B5OyBcdTAwM2NhIGhyZWY9XCJodHRwOi8vb3BlbnN0cmVldG1hcC5vcmdcIlx1MDAzZU9wZW5TdHJlZXRNYXBcdTAwM2MvYVx1MDAzZSwgdW5kZXIgXHUwMDNjYSBocmVmPVwiaHR0cDovL3d3dy5vcGVuc3RyZWV0bWFwLm9yZy9jb3B5cmlnaHRcIlx1MDAzZU9EYkxcdTAwM2MvYVx1MDAzZS4iLCAiZGV0ZWN0UmV0aW5hIjogZmFsc2UsICJtYXhOYXRpdmVab29tIjogMTgsICJtYXhab29tIjogMTgsICJtaW5ab29tIjogMCwgIm5vV3JhcCI6IGZhbHNlLCAib3BhY2l0eSI6IDEsICJzdWJkb21haW5zIjogImFiYyIsICJ0bXMiOiBmYWxzZX0KICAgICAgICAgICAgKS5hZGRUbyhtYXBfMzYxZDIyY2ZmNGFiNGVhYWE1YzdiNGU3NmI4ZTk1NWMpOwogICAgICAgIAogICAgCiAgICAgICAgZnVuY3Rpb24gZ2VvX2pzb25fYTczZWM2ZDVlMTA3NDk5YjhjMmQzZWQyY2M3M2I0ZTFfb25FYWNoRmVhdHVyZShmZWF0dXJlLCBsYXllcikgewogICAgICAgICAgICBsYXllci5vbih7CiAgICAgICAgICAgIH0pOwogICAgICAgIH07CiAgICAgICAgdmFyIGdlb19qc29uX2E3M2VjNmQ1ZTEwNzQ5OWI4YzJkM2VkMmNjNzNiNGUxID0gTC5nZW9Kc29uKG51bGwsIHsKICAgICAgICAgICAgICAgIG9uRWFjaEZlYXR1cmU6IGdlb19qc29uX2E3M2VjNmQ1ZTEwNzQ5OWI4YzJkM2VkMmNjNzNiNGUxX29uRWFjaEZlYXR1cmUsCiAgICAgICAgICAgIAogICAgICAgIH0pOwoKICAgICAgICBmdW5jdGlvbiBnZW9fanNvbl9hNzNlYzZkNWUxMDc0OTliOGMyZDNlZDJjYzczYjRlMV9hZGQgKGRhdGEpIHsKICAgICAgICAgICAgZ2VvX2pzb25fYTczZWM2ZDVlMTA3NDk5YjhjMmQzZWQyY2M3M2I0ZTEKICAgICAgICAgICAgICAgIC5hZGREYXRhKGRhdGEpCiAgICAgICAgICAgICAgICAuYWRkVG8obWFwXzM2MWQyMmNmZjRhYjRlYWFhNWM3YjRlNzZiOGU5NTVjKTsKICAgICAgICB9CiAgICAgICAgICAgIGdlb19qc29uX2E3M2VjNmQ1ZTEwNzQ5OWI4YzJkM2VkMmNjNzNiNGUxX2FkZCh7ImZlYXR1cmVzIjogW3siZ2VvbWV0cnkiOiB7ImNvb3JkaW5hdGVzIjogW1tbWzQ1Ljg1ODMyNCwgMTAuOTIzNzY1XSwgWzQ1Ljg1ODIyNiwgMTAuOTIzNzM5XSwgWzQ1Ljg1ODE2OSwgMTAuOTIzNzAxXSwgWzQ1Ljg1ODExOCwgMTAuOTIzNjM3XSwgWzQ1Ljg1ODEwNiwgMTAuOTIzNTkxXSwgWzQ1Ljg1ODA4MywgMTAuOTIzNTM3XSwgWzQ1Ljg1ODA2NCwgMTAuOTIzNDI5XSwgWzQ1Ljg1ODA1NCwgMTAuOTIzMzI3XSwgWzQ1Ljg1ODA1MywgMTAuOTIzMjA5XSwgWzQ1Ljg1ODA0OSwgMTAuOTIzMDc4XSwgWzQ1Ljg1ODA0MiwgMTAuOTIyOTAxXSwgWzQ1Ljg1ODAyNywgMTAuOTIyNzEzXSwgWzQ1Ljg1Nzk4MiwgMTAuOTIyMjNdLCBbNDUuODU3OTcsIDEwLjkyMTk1OF0sIFs0NS44NTc5NTIsIDEwLjkyMTc5N10sIFs0NS44NTc4ODgsIDEwLjkyMTY1MV0sIFs0NS44NTc4MjgsIDEwLjkyMTU2XSwgWzQ1Ljg1Nzc2OCwgMTAuOTIxNDM0XSwgWzQ1Ljg1NzczOCwgMTAuOTIxMjc5XSwgWzQ1Ljg1Nzc1LCAxMC45MjExMzldLCBbNDUuODU3Nzg2LCAxMC45MjA5OTZdLCBbNDUuODU3ODI2LCAxMC45MjA5MDldLCBbNDUuODU3OTA0LCAxMC45MjA4XSwgWzQ1Ljg1Nzk5OSwgMTAuOTIwNjkxXSwgWzQ1Ljg1ODE1MywgMTAuOTIwNTU1XSwgWzQ1Ljg1ODI1NSwgMTAuOTIwNDY3XSwgWzQ1Ljg1ODM0LCAxMC45MjA0NV0sIFs0NS44NTg0NDksIDEwLjkyMDQ3XSwgWzQ1Ljg1ODU1MSwgMTAuOTIwNTY1XSwgWzQ1Ljg1ODU4OSwgMTAuOTIwNjUxXSwgWzQ1Ljg1ODYwNiwgMTAuOTIwNzFdLCBbNDUuODU4NjUxLCAxMC45MjA3NjRdLCBbNDUuODU4NzU4LCAxMC45MjA4MjNdLCBbNDUuODU4ODg1LCAxMC45MjA4N10sIFs0NS44NTkwMTUsIDEwLjkyMDldLCBbNDUuODU5MTAyLCAxMC45MjA5MDNdLCBbNDUuODU5MTkyLCAxMC45MjA4ODZdLCBbNDUuODU5MzMsIDEwLjkyMDgxOF0sIFs0NS44NTkzNzgsIDEwLjkyMDgyM10sIFs0NS44NTk0OSwgMTAuOTIwODYzXSwgWzQ1Ljg1OTU4MywgMTAuOTIwNzYxXSwgWzQ1Ljg1OTU5NywgMTAuOTIwNTc4XSwgWzQ1Ljg1OTU4NywgMTAuOTIwNTI1XSwgWzQ1Ljg1OTU3NywgMTAuOTIwNDQ2XSwgWzQ1Ljg1OTU3NSwgMTAuOTIwMzg0XSwgWzQ1Ljg1OTYwNiwgMTAuOTIwMjk3XSwgWzQ1Ljg1OTYyMiwgMTAuOTIwMjEyXSwgWzQ1Ljg1OTYzLCAxMC45MjAwOTddLCBbNDUuODU5NjE4LCAxMC45MjAwMjJdLCBbNDUuODU5NjAxLCAxMC45MTk5NDldLCBbNDUuODU5NjA0LCAxMC45MTk4NjRdLCBbNDUuODU5NjE5LCAxMC45MTk4MTJdLCBbNDUuODU5NjUxLCAxMC45MTk3ODRdLCBbNDUuODU5NjksIDEwLjkxOTc4NV0sIFs0NS44NTk3NTksIDEwLjkxOThdLCBbNDUuODU5ODMzLCAxMC45MTk4NjhdLCBbNDUuODU5OTE5LCAxMC45MTk5NjZdLCBbNDUuODU5OTc3LCAxMC45MjAwMzNdLCBbNDUuODYwMDUsIDEwLjkyMDA4OF0sIFs0NS44NjAxMDIsIDEwLjkyMDEyMl0sIFs0NS44NjAxNTksIDEwLjkyMDExNF0sIFs0NS44NjAyMTIsIDEwLjkyMDA3XSwgWzQ1Ljg2MDI0MSwgMTAuOTIwMDAyXSwgWzQ1Ljg2MDI0NCwgMTAuOTE5OTMzXSwgWzQ1Ljg2MDIyNSwgMTAuOTE5ODQ0XSwgWzQ1Ljg2MDE4OCwgMTAuOTE5NzM1XSwgWzQ1Ljg2MDE1MywgMTAuOTE5NjUyXSwgWzQ1Ljg2MDEyNCwgMTAuOTE5NTc2XSwgWzQ1Ljg2MDA5OCwgMTAuOTE5NDY0XSwgWzQ1Ljg2MDEwMSwgMTAuOTE5NDI0XSwgWzQ1Ljg2MDEzMSwgMTAuOTE5NDEyXSwgWzQ1Ljg2MDE3NywgMTAuOTE5NDNdLCBbNDUuODYwMjA2LCAxMC45MTk0NTddLCBbNDUuODYwMjQsIDEwLjkxOTUwNF0sIFs0NS44NjAyNzEsIDEwLjkxOTU3MV0sIFs0NS44NjAzMDQsIDEwLjkxOTY1M10sIFs0NS44NjAzNTIsIDEwLjkxOTc2M10sIFs0NS44NjA0MDQsIDEwLjkxOTg2XSwgWzQ1Ljg2MDQ1MywgMTAuOTE5OTNdLCBbNDUuODYwNDkxLCAxMC45MTk5NTRdLCBbNDUuODYwNTY3LCAxMC45MTk5MjFdLCBbNDUuODYwNTk2LCAxMC45MTk4ODJdLCBbNDUuODYwNjM0LCAxMC45MTk3NzldLCBbNDUuODYwNjQ1LCAxMC45MTk2NjhdLCBbNDUuODYwNjQ0LCAxMC45MTk1NV0sIFs0NS44NjA3MTcsIDEwLjkxOTI5XSwgWzQ1Ljg2MDc3OCwgMTAuOTE5MTg0XSwgWzQ1Ljg2MDgxNiwgMTAuOTE5MTA2XSwgWzQ1Ljg2MDg0NywgMTAuOTE5MDM4XSwgWzQ1Ljg2MDg4NSwgMTAuOTE4OTMxXSwgWzQ1Ljg2MDkwMywgMTAuOTE4ODE3XSwgWzQ1Ljg2MDkwNSwgMTAuOTE4Njk5XSwgWzQ1Ljg2MDkwNSwgMTAuOTE4NTg4XSwgWzQ1Ljg2MDkzMiwgMTAuOTE4NDcxXSwgWzQ1Ljg2MDk1OSwgMTAuOTE4MzQ0XSwgWzQ1Ljg2MDk4MywgMTAuOTE4MjVdLCBbNDUuODYxMDUxLCAxMC45MTgxMzddLCBbNDUuODYxMjA1LCAxMC45MTgwMDFdLCBbNDUuODYxMzUsIDEwLjkxNzkxN10sIFs0NS44NjE0OTgsIDEwLjkxNzgxNF0sIFs0NS44NjE2NDYsIDEwLjkxNzczN10sIFs0NS44NjE3OTIsIDEwLjkxNzY1Nl0sIFs0NS44NjIwMzIsIDEwLjkxNzUyM10sIFs0NS44NjIyMDMsIDEwLjkxNzQyN10sIFs0NS44NjI0MDIsIDEwLjkxNzMyOF0sIFs0NS44NjI1ODIsIDEwLjkxNzI0OV0sIFs0NS44NjI4MjQsIDEwLjkxNzIxNF0sIFs0NS44NjI5NzYsIDEwLjkxNzE5Ml0sIFs0NS44NjMxODQsIDEwLjkxNzIxMl0sIFs0NS44NjMzNjUsIDEwLjkxNzIzNF0sIFs0NS44NjM1MDUsIDEwLjkxNzI0NV0sIFs0NS44NjM2NDcsIDEwLjkxNzIzM10sIFs0NS44NjM4MjIsIDEwLjkxNzE4M10sIFs0NS44NjM5NjcsIDEwLjkxNzExMl0sIFs0NS44NjQwOTksIDEwLjkxNzAxOF0sIFs0NS44NjQyMzQsIDEwLjkxNjg5NF0sIFs0NS44NjQyOTQsIDEwLjkxNjgzNF0sIFs0NS44NjQ0MDksIDEwLjkxNjcwN10sIFs0NS44NjQ1LCAxMC45MTY1ODJdLCBbNDUuODY0NTc3LCAxMC45MTY1MDldLCBbNDUuODY0NjUyLCAxMC45MTY0OTJdLCBbNDUuODY0NzMsIDEwLjkxNjUxNF0sIFs0NS44NjQ3NTQsIDEwLjkxNjU1N10sIFs0NS44NjQ3NjcsIDEwLjkxNjYyXSwgWzQ1Ljg2NDc1MywgMTAuOTE2Njc4XSwgWzQ1Ljg2NDY5OCwgMTAuOTE2Nzc1XSwgWzQ1Ljg2NDYzOCwgMTAuOTE2ODU1XSwgWzQ1Ljg2NDYsIDEwLjkxNjg5XSwgWzQ1Ljg2NDQ2MSwgMTAuOTE3MDEzXSwgWzQ1Ljg2NDM2OCwgMTAuOTE3MTEyXSwgWzQ1Ljg2NDI4NCwgMTAuOTE3MjE0XSwgWzQ1Ljg2NDE5OSwgMTAuOTE3MzMyXSwgWzQ1Ljg2NDE2NiwgMTAuOTE3NDIzXSwgWzQ1Ljg2NDE1OCwgMTAuOTE3NTA4XSwgWzQ1Ljg2NDIyNSwgMTAuOTE3NzU5XSwgWzQ1Ljg2NDI2MywgMTAuOTE3ODIzXSwgWzQ1Ljg2NDMzMiwgMTAuOTE3OTA3XSwgWzQ1Ljg2NDQxNCwgMTAuOTE3OTc4XSwgWzQ1Ljg2NDQ4OCwgMTAuOTE4MDM2XSwgWzQ1Ljg2NDU4OCwgMTAuOTE4MDc5XSwgWzQ1Ljg2NDY4OSwgMTAuOTE4MDk4XSwgWzQ1Ljg2NDgxNSwgMTAuOTE4MDgzXSwgWzQ1Ljg2NDk1MywgMTAuOTE4MDYxXSwgWzQ1Ljg2NTA3NSwgMTAuOTE3OTk5XSwgWzQ1Ljg2NTIyMSwgMTAuOTE3OTA5XSwgWzQ1Ljg2NTQwNiwgMTAuOTE3OF0sIFs0NS44NjU1ODQsIDEwLjkxNzY4N10sIFs0NS44NjU3NTYsIDEwLjkxNzU1Ml0sIFs0NS44NjU5MTYsIDEwLjkxNzM4N10sIFs0NS44NjYwNTQsIDEwLjkxNzIwMV0sIFs0NS44NjYxMDksIDEwLjkxNzA2Ml0sIFs0NS44NjYxMzIsIDEwLjkxNjkzNV0sIFs0NS44NjYxMjksIDEwLjkxNjgxM10sIFs0NS44NjYwOTQsIDEwLjkxNjcyMV0sIFs0NS44NjYwMDQsIDEwLjkxNjYyOV0sIFs0NS44NjU5MTEsIDEwLjkxNjU0NF0sIFs0NS44NjU4MTQsIDEwLjkxNjQ2M10sIFs0NS44NjU3MTcsIDEwLjkxNjM5NF0sIFs0NS44NjU1OTYsIDEwLjkxNjMzMV0sIFs0NS44NjU0NiwgMTAuOTE2Mjc4XSwgWzQ1Ljg2NTMwOSwgMTAuOTE2Mjc2XSwgWzQ1Ljg2NTIwMywgMTAuOTE2MjZdLCBbNDUuODY1MTUxLCAxMC45MTYyMTldLCBbNDUuODY1MTUsIDEwLjkxNjE1XSwgWzQ1Ljg2NTE2NywgMTAuOTE2MTA4XSwgWzQ1Ljg2NTIxOCwgMTAuOTE2MDQxXSwgWzQ1Ljg2NTI4MSwgMTAuOTE1OTc3XSwgWzQ1Ljg2NTM2MywgMTAuOTE1ODkxXSwgWzQ1Ljg2NTQ0LCAxMC45MTU4MDJdLCBbNDUuODY1NTM1LCAxMC45MTU3MV0sIFs0NS44NjU1ODQsIDEwLjkxNTY0OV0sIFs0NS44NjU2NzksIDEwLjkxNTZdLCBbNDUuODY1NzQsIDEwLjkxNTU3NF0sIFs0NS44NjU4MSwgMTAuOTE1NTVdLCBbNDUuODY1OTA3LCAxMC45MTU0ODFdLCBbNDUuODY2MDA3LCAxMC45MTU0MDVdLCBbNDUuODY2MTIzLCAxMC45MTUzMTddLCBbNDUuODY2MjI5LCAxMC45MTUyNDVdLCBbNDUuODY2MzY2LCAxMC45MTUxNjddLCBbNDUuODY2NTA0LCAxMC45MTUxMTZdLCBbNDUuODY2NjIxLCAxMC45MTUwODddLCBbNDUuODY2NzU3LCAxMC45MTUwNzJdLCBbNDUuODY2OTAxLCAxMC45MTUwNl0sIFs0NS44NjcwMjksIDEwLjkxNTA1NF0sIFs0NS44NjcxNjUsIDEwLjkxNTA0NV0sIFs0NS44NjcyOTMsIDEwLjkxNTAzOV0sIFs0NS44Njc0MjksIDEwLjkxNTAxN10sIFs0NS44Njc1OCwgMTAuOTE0OThdLCBbNDUuODY3NzM0LCAxMC45MTQ5MzldLCBbNDUuODY3ODYxLCAxMC45MTQ4ODddLCBbNDUuODY3OTg5LCAxMC45MTQ3NzNdLCBbNDUuODY4MDkyLCAxMC45MTQ2MTJdLCBbNDUuODY4MTk2LCAxMC45MTQ0MjJdLCBbNDUuODY4MjYsIDEwLjkxNDIzMV0sIFs0NS44NjgyNzksIDEwLjkxNDA4NF0sIFs0NS44NjgzMSwgMTAuOTEzODQ2XSwgWzQ1Ljg2ODMyLCAxMC45MTM2NDNdLCBbNDUuODY4MzIxLCAxMC45MTM0MTRdLCBbNDUuODY4MzMxLCAxMC45MTMyMTFdLCBbNDUuODY4MzYxLCAxMC45MTMwNThdLCBbNDUuODY4NDA0LCAxMC45MTI5NTRdLCBbNDUuODY4NDg3LCAxMC45MTI4NzJdLCBbNDUuODY4NTYzLCAxMC45MTI4MzFdLCBbNDUuODY4NjUzLCAxMC45MTI4MzRdLCBbNDUuODY4NzI4LCAxMC45MTI4NzZdLCBbNDUuODY4ODE2LCAxMC45MTI5NjddLCBbNDUuODY4ODkyLCAxMC45MTMwNjFdLCBbNDUuODY4OTY0LCAxMC45MTMxNTJdLCBbNDUuODY5MDM4LCAxMC45MTMyNDNdLCBbNDUuODY5MTE1LCAxMC45MTMzMzddLCBbNDUuODY5MTkxLCAxMC45MTM0MzFdLCBbNDUuODY5MjMzLCAxMC45MTM1MThdLCBbNDUuODY5MjU5LCAxMC45MTM2MTRdLCBbNDUuODY5MjYsIDEwLjkxMzcxNV0sIFs0NS44NjkyNTIsIDEwLjkxMzgwM10sIFs0NS44NjkyNDgsIDEwLjkxMzkwMl0sIFs0NS44NjkyNDQsIDEwLjkxNDAyM10sIFs0NS44NjkyNDYsIDEwLjkxNDE2NF0sIFs0NS44NjkyMzcsIDEwLjkxNDQ1NV0sIFs0NS44NjkyMjMsIDEwLjkxNDYxOV0sIFs0NS44NjkyMDMsIDEwLjkxNDczOV0sIFs0NS44NjkyLCAxMC45MTQ5NzJdLCBbNDUuODY5MjA0LCAxMC45MTUxNjJdLCBbNDUuODY5MTk0LCAxMC45MTUzMzVdLCBbNDUuODY5MTc2LCAxMC45MTU0NzldLCBbNDUuODY5MTc1LCAxMC45MTU2NzZdLCBbNDUuODY5MTk5LCAxMC45MTU3NjNdLCBbNDUuODY5MjE2LCAxMC45MTU4MzldLCBbNDUuODY5MjQ2LCAxMC45MTU5NDJdLCBbNDUuODY5Mjk1LCAxMC45MTYwNTJdLCBbNDUuODY5MzIyLCAxMC45MTYxMDJdLCBbNDUuODY5MzYsIDEwLjkxNjE0OV0sIFs0NS44NjkzOTYsIDEwLjkxNjE3OV0sIFs0NS44Njk1MDEsIDEwLjkxNjIyMl0sIFs0NS44Njk1ODcsIDEwLjkxNjIzNF0sIFs0NS44Njk2NTksIDEwLjkxNjE4OF0sIFs0NS44Njk3NDMsIDEwLjkxNjA5NV0sIFs0NS44Njk4MzEsIDEwLjkxNjAxNl0sIFs0NS44Njk4OTgsIDEwLjkxNjAwMl0sIFs0NS44Njk5NTMsIDEwLjkxNjAxN10sIFs0NS44NzAwMTYsIDEwLjkxNjA3MV0sIFs0NS44NzAwNywgMTAuOTE2MTU1XSwgWzQ1Ljg3MDA4OSwgMTAuOTE2MjI0XSwgWzQ1Ljg3MDEwMSwgMTAuOTE2MzI5XSwgWzQ1Ljg3MDA5MSwgMTAuOTE2NDI0XSwgWzQ1Ljg3MDA5NiwgMTAuOTE2NTM5XSwgWzQ1Ljg3MDEyNSwgMTAuOTE2NjgxXSwgWzQ1Ljg3MDE2NSwgMTAuOTE2ODAzXSwgWzQ1Ljg3MDIyMywgMTAuOTE2OTFdLCBbNDUuODcwMjc0LCAxMC45MTY5NjddLCBbNDUuODcwMzQyLCAxMC45MTcwNDhdLCBbNDUuODcwMzU3LCAxMC45MTcxMTRdLCBbNDUuODcwMzQ2LCAxMC45MTcyMjhdLCBbNDUuODcwMjYyLCAxMC45MTc0NjhdLCBbNDUuODcwMTcxLCAxMC45MTc1OTZdLCBbNDUuODcwMDk2LCAxMC45MTc2NjZdLCBbNDUuODcwMDEzLCAxMC45MTc3MTldLCBbNDUuODY5OTMyLCAxMC45MTc3NjZdLCBbNDUuODY5ODQ0LCAxMC45MTc4MDldLCBbNDUuODY5NzUyLCAxMC45MTc4MzldLCBbNDUuODY5NjQ3LCAxMC45MTc4MzldLCBbNDUuODY5NTQ4LCAxMC45MTc4MzJdLCBbNDUuODY5NDc3LCAxMC45MTc4NDNdLCBbNDUuODY5NDA1LCAxMC45MTc5NDJdLCBbNDUuODY5MzYzLCAxMC45MTgxMDJdLCBbNDUuODY5MzI0LCAxMC45MTgyNTRdLCBbNDUuODY5Mjc2LCAxMC45MTgzODRdLCBbNDUuODY5MjIyLCAxMC45MTg0NjddLCBbNDUuODY5MTQ2LCAxMC45MTg1MTFdLCBbNDUuODY5MDU3LCAxMC45MTg1MDJdLCBbNDUuODY4OTM0LCAxMC45MTg0NzVdLCBbNDUuODY4ODM4LCAxMC45MTg0NjJdLCBbNDUuODY4Nzg1LCAxMC45MTg0N10sIFs0NS44Njg3NDMsIDEwLjkxODUyNF0sIFs0NS44Njg3NTMsIDEwLjkxODZdLCBbNDUuODY4OCwgMTAuOTE4NjldLCBbNDUuODY4ODUyLCAxMC45MTg3NDddLCBbNDUuODY4OTE5LCAxMC45MTg3OTVdLCBbNDUuODY4OTgsIDEwLjkxODg1M10sIFs0NS44Njg5OTMsIDEwLjkxODkwOV0sIFs0NS44Njg5ODcsIDEwLjkxODk5MV0sIFs0NS44Njg5NzUsIDEwLjkxOTAzXSwgWzQ1Ljg2ODkzNywgMTAuOTE5MTA0XSwgWzQ1Ljg2ODg0OSwgMTAuOTE5MTldLCBbNDUuODY4NzMzLCAxMC45MTkzMDFdLCBbNDUuODY4NTY4LCAxMC45MTk0NTZdLCBbNDUuODY4NDU4LCAxMC45MTk1NjddLCBbNDUuODY4MzE3LCAxMC45MTk2ODFdLCBbNDUuODY4MjA3LCAxMC45MTk3MDddLCBbNDUuODY4MDY5LCAxMC45MTk2ODldLCBbNDUuODY3OTU2LCAxMC45MTk2NDNdLCBbNDUuODY3ODgxLCAxMC45MTk2MDFdLCBbNDUuODY3ODA4LCAxMC45MTk1OTJdLCBbNDUuODY3NzQxLCAxMC45MTk2MDNdLCBbNDUuODY3Njc4LCAxMC45MTk2NTFdLCBbNDUuODY3NjI1LCAxMC45MTk2OThdLCBbNDUuODY3NTQ2LCAxMC45MTk3OTFdLCBbNDUuODY3NDQzLCAxMC45MTk5MjVdLCBbNDUuODY3MzIzLCAxMC45MjAxMDVdLCBbNDUuODY3MjA4LCAxMC45MjAyNzFdLCBbNDUuODY3MDU4LCAxMC45MjA0ODldLCBbNDUuODY2ODg3LCAxMC45MjA3NTNdLCBbNDUuODY2NzA2LCAxMC45MjEwMzldLCBbNDUuODY2NTcyLCAxMC45MjEyNDddLCBbNDUuODY2NDUsIDEwLjkyMTQ1Nl0sIFs0NS44NjYzNDEsIDEwLjkyMTY4OV0sIFs0NS44NjYyMTEsIDEwLjkyMTk1XSwgWzQ1Ljg2NjAzMywgMTAuOTIyMTc3XSwgWzQ1Ljg2NTg2MiwgMTAuOTIyMzAzXSwgWzQ1Ljg2NTY4OSwgMTAuOTIyMzZdLCBbNDUuODY1NTQ3LCAxMC45MjIzNzVdLCBbNDUuODY1Mzk2LCAxMC45MjIzNjddLCBbNDUuODY1MjE3LCAxMC45MjIzMzVdLCBbNDUuODY1MDU1LCAxMC45MjIyOTddLCBbNDUuODY0OTcxLCAxMC45MjIyODRdLCBbNDUuODY0ODc3LCAxMC45MjIyODJdLCBbNDUuODY0NzYyLCAxMC45MjIyOTRdLCBbNDUuODY0NTEsIDEwLjkyMjM1Ml0sIFs0NS44NjQzMTIsIDEwLjkyMjM3NV0sIFs0NS44NjQxMDIsIDEwLjkyMjM1OV0sIFs0NS44NjM5MTksIDEwLjkyMjMyN10sIFs0NS44NjM3NDcsIDEwLjkyMjMzNF0sIFs0NS44NjM1OTEsIDEwLjkyMjM2Ml0sIFs0NS44NjM0NiwgMTAuOTIyNDA0XSwgWzQ1Ljg2MzMzMywgMTAuOTIyNDU5XSwgWzQ1Ljg2MzE2OSwgMTAuOTIyNTIyXSwgWzQ1Ljg2MzA2OCwgMTAuOTIyNTM4XSwgWzQ1Ljg2Mjk5NywgMTAuOTIyNTVdLCBbNDUuODYyODQ2LCAxMC45MjI1NjFdLCBbNDUuODYyNzYzLCAxMC45MjI1NTldLCBbNDUuODYyNTgxLCAxMC45MjI1MjddLCBbNDUuODYyNDIxLCAxMC45MjI1MDJdLCBbNDUuODYyMjQ5LCAxMC45MjI1XSwgWzQ1Ljg2MjA3MiwgMTAuOTIyNTI3XSwgWzQ1Ljg2MTg4NiwgMTAuOTIyNTY3XSwgWzQ1Ljg2MTcxNiwgMTAuOTIyNTg4XSwgWzQ1Ljg2MTU2NywgMTAuOTIyNTldLCBbNDUuODYxNDIzLCAxMC45MjI2MTFdLCBbNDUuODYxMjY0LCAxMC45MjI2ODJdLCBbNDUuODYxMTUzLCAxMC45MjI3NDRdLCBbNDUuODYxMDY0LCAxMC45MjI4MTldLCBbNDUuODYwOTY2LCAxMC45MjI5ODNdLCBbNDUuODYwOTEyLCAxMC45MjMwMjRdLCBbNDUuODYwODI1LCAxMC45MjMwMzhdLCBbNDUuODYwNzI3LCAxMC45MjMwMTVdLCBbNDUuODYwNTA2LCAxMC45MjI5MjZdLCBbNDUuODYwNDA4LCAxMC45MjI4ODRdLCBbNDUuODYwMjcsIDEwLjkyMjgyNF0sIFs0NS44NjAxNDIsIDEwLjkyMjc4XSwgWzQ1Ljg1OTk2OSwgMTAuOTIyNzQyXSwgWzQ1Ljg1OTgxMSwgMTAuOTIyNzA0XSwgWzQ1Ljg1OTY4MSwgMTAuOTIyNjc3XSwgWzQ1Ljg1OTYwOCwgMTAuOTIyNjgyXSwgWzQ1Ljg1OTQ3NCwgMTAuOTIyNzRdLCBbNDUuODU5MzQ5LCAxMC45MjI4MzFdLCBbNDUuODU5MjUxLCAxMC45MjI5MzJdLCBbNDUuODU5MTY5LCAxMC45MjMwMjFdLCBbNDUuODU4OTk0LCAxMC45MjMyNTJdLCBbNDUuODU4OTUyLCAxMC45MjMzMTldLCBbNDUuODU4ODc5LCAxMC45MjM0NDFdLCBbNDUuODU4NzgsIDEwLjkyMzYwNV0sIFs0NS44NTg2OCwgMTAuOTIzNzAxXSwgWzQ1Ljg1ODU2OSwgMTAuOTIzNzUzXSwgWzQ1Ljg1ODQ1MiwgMTAuOTIzNzY5XSwgWzQ1Ljg1ODMyNCwgMTAuOTIzNzY1XV1dXSwgInR5cGUiOiAiTXVsdGlQb2x5Z29uIn0sICJpZCI6ICIwIiwgInByb3BlcnRpZXMiOiB7ImRlc2NyaXppb25lX29yaWdpbmUiOiAiTm9uIGNvZGlmaWNhdG8iLCAiZ21sX2lkIjogIklELkFDUVVFRklTSUNIRS5TUEVDQ0hJLkFDUVVBLjU0MTg2IiwgIm5hc3AiOiAwLCAibmF0dXJhX3NwZWNjaGlvX2FjcXVhIjogIk5vbiBjb2RpZmljYXRvIiwgIm5vbWUiOiAiTEFHTyBESSBMT1BQSU8iLCAib3JpZ2luZSI6IDB9LCAidHlwZSI6ICJGZWF0dXJlIn0sIHsiZ2VvbWV0cnkiOiB7ImNvb3JkaW5hdGVzIjogW1tbWzQ1Ljk4MjQzNiwgMTAuOTIyNDEzXSwgWzQ1Ljk4MjM4LCAxMC45MjIzODVdLCBbNDUuOTgyMzI1LCAxMC45MjIzNTddLCBbNDUuOTgyMjM4LCAxMC45MjIzMzJdLCBbNDUuOTgyMTI0LCAxMC45MjIyNzNdLCBbNDUuOTgyMDI4LCAxMC45MjIyNjRdLCBbNDUuOTgxOTU1LCAxMC45MjIyMjldLCBbNDUuOTgxOTI5LCAxMC45MjIxNzNdLCBbNDUuOTgxOTUsIDEwLjkyMjEyMV0sIFs0NS45ODE5NzgsIDEwLjkyMjA4Ml0sIFs0NS45ODIwMzYsIDEwLjkyMjAwNV0sIFs0NS45ODIwNzEsIDEwLjkyMTk0N10sIFs0NS45ODIyMDgsIDEwLjkyMTgxMl0sIFs0NS45ODIyNjgsIDEwLjkyMTc5MV0sIFs0NS45ODIzNzQsIDEwLjkyMTc2MV0sIFs0NS45ODI0NjMsIDEwLjkyMTc5XSwgWzQ1Ljk4MjU1MiwgMTAuOTIxODI1XSwgWzQ1Ljk4MjY0NywgMTAuOTIxOTEzXSwgWzQ1Ljk4MjcwMSwgMTAuOTIxOTU3XSwgWzQ1Ljk4MjcyMSwgMTAuOTIyMDE3XSwgWzQ1Ljk4Mjc0MSwgMTAuOTIyMDgzXSwgWzQ1Ljk4Mjc0OSwgMTAuOTIyMTQ2XSwgWzQ1Ljk4Mjc0OCwgMTAuOTIyMjA4XSwgWzQ1Ljk4MjcyMiwgMTAuOTIyMjY2XSwgWzQ1Ljk4MjY2OSwgMTAuOTIyMzExXSwgWzQ1Ljk4MjYyNSwgMTAuOTIyMzQ5XSwgWzQ1Ljk4MjU2MywgMTAuOTIyMzY3XSwgWzQ1Ljk4MjQ4NywgMTAuOTIyMzkxXSwgWzQ1Ljk4MjQzNiwgMTAuOTIyNDEzXV1dXSwgInR5cGUiOiAiTXVsdGlQb2x5Z29uIn0sICJpZCI6ICIxIiwgInByb3BlcnRpZXMiOiB7ImRlc2NyaXppb25lX29yaWdpbmUiOiAiTm9uIGNvZGlmaWNhdG8iLCAiZ21sX2lkIjogIklELkFDUVVFRklTSUNIRS5TUEVDQ0hJLkFDUVVBLjU0MTYyIiwgIm5hc3AiOiAwLCAibmF0dXJhX3NwZWNjaGlvX2FjcXVhIjogIk5vbiBjb2RpZmljYXRvIiwgIm5vbWUiOiAiTEFHTyBERUkgQkFHQVRPSSAoRFJPKSIsICJvcmlnaW5lIjogMH0sICJ0eXBlIjogIkZlYXR1cmUifSwgeyJnZW9tZXRyeSI6IHsiY29vcmRpbmF0ZXMiOiBbW1tbNDUuOTc3MTkxLCAxMC45MzE4Nl0sIFs0NS45NzcxNDMsIDEwLjkzMTgyM10sIFs0NS45NzcxNDYsIDEwLjkzMTczN10sIFs0NS45NzcxNzQsIDEwLjkzMTcwMl0sIFs0NS45NzcxOTUsIDEwLjkzMTY3M10sIFs0NS45NzcxOTksIDEwLjkzMTU5NF0sIFs0NS45NzcxNzcsIDEwLjkzMTUzNV0sIFs0NS45NzcxNTIsIDEwLjkzMTQ3NV0sIFs0NS45NzcwODksIDEwLjkzMTQyMV0sIFs0NS45NzcwNTEsIDEwLjkzMTM2NF0sIFs0NS45NzcwMTcsIDEwLjkzMTI4MV0sIFs0NS45NzcsIDEwLjkzMTE4OF0sIFs0NS45NzcwMDksIDEwLjkzMTA4XSwgWzQ1Ljk3NzA1NywgMTAuOTMxMDI5XSwgWzQ1Ljk3NzEzMSwgMTAuOTMxMDI0XSwgWzQ1Ljk3NzE1MSwgMTAuOTMxMDUxXSwgWzQ1Ljk3NzIxNywgMTAuOTMxMTAyXSwgWzQ1Ljk3NzI3OCwgMTAuOTMxMTU2XSwgWzQ1Ljk3NzMwNSwgMTAuOTMxMTk3XSwgWzQ1Ljk3NzM2NiwgMTAuOTMxMjUxXSwgWzQ1Ljk3NzM4OCwgMTAuOTMxMjg0XSwgWzQ1Ljk3NzQyMiwgMTAuOTMxMzIxXSwgWzQ1Ljk3NzQ3MiwgMTAuOTMxMzY1XSwgWzQ1Ljk3NzUwOSwgMTAuOTMxMzYzXSwgWzQ1Ljk3NzYzNiwgMTAuOTMxMzA0XSwgWzQ1Ljk3Nzc2NywgMTAuOTMxMjM5XSwgWzQ1Ljk3Nzg5MiwgMTAuOTMxMThdLCBbNDUuOTc3OTY3LCAxMC45MzExNTldLCBbNDUuOTc3OTkyLCAxMC45MzExOTldLCBbNDUuOTc3OTgyLCAxMC45MzEyMzhdLCBbNDUuOTc3OTU3LCAxMC45MzEyODNdLCBbNDUuOTc3OTI5LCAxMC45MzEzMTldLCBbNDUuOTc3ODU3LCAxMC45MzEzNzldLCBbNDUuOTc3ODEzLCAxMC45MzE0MTRdLCBbNDUuOTc3NzUzLCAxMC45MzE0NjVdLCBbNDUuOTc3Njg4LCAxMC45MzE1MTJdLCBbNDUuOTc3NTk1LCAxMC45MzE1NzJdLCBbNDUuOTc3NTEsIDEwLjkzMTYyNl0sIFs0NS45Nzc0MiwgMTAuOTMxNjk5XSwgWzQ1Ljk3NzI5OSwgMTAuOTMxNzg0XSwgWzQ1Ljk3NzE5MSwgMTAuOTMxODZdXV1dLCAidHlwZSI6ICJNdWx0aVBvbHlnb24ifSwgImlkIjogIjIiLCAicHJvcGVydGllcyI6IHsiZGVzY3JpemlvbmVfb3JpZ2luZSI6ICJOb24gY29kaWZpY2F0byIsICJnbWxfaWQiOiAiSUQuQUNRVUVGSVNJQ0hFLlNQRUNDSEkuQUNRVUEuNTQxNjQiLCAibmFzcCI6IDAsICJuYXR1cmFfc3BlY2NoaW9fYWNxdWEiOiAiTm9uIGNvZGlmaWNhdG8iLCAibm9tZSI6ICJMQUdPIFNPTE8iLCAib3JpZ2luZSI6IDB9LCAidHlwZSI6ICJGZWF0dXJlIn0sIHsiZ2VvbWV0cnkiOiB7ImNvb3JkaW5hdGVzIjogW1tbWzQ1Ljk5NDU4NSwgMTAuODMwNTAyXSwgWzQ1Ljk5NDQ5NiwgMTAuODMwNDgyXSwgWzQ1Ljk5NDQ2NywgMTAuODMwNDU4XSwgWzQ1Ljk5NDMwNSwgMTAuODMwMTU1XSwgWzQ1Ljk5NDI1NCwgMTAuODMwMDYxXSwgWzQ1Ljk5NDE5MSwgMTAuODI5OTNdLCBbNDUuOTk0MDU4LCAxMC44Mjk3MDRdLCBbNDUuOTkzOTgyLCAxMC44Mjk1ODNdLCBbNDUuOTkzOTEzLCAxMC44Mjk0NzhdLCBbNDUuOTkzODQ5LCAxMC44MjkzODNdLCBbNDUuOTkzNzkxLCAxMC44MjkyNzZdLCBbNDUuOTkzNzQ1LCAxMC44MjkxNjJdLCBbNDUuOTkzNzA0LCAxMC44MjkwMzVdLCBbNDUuOTkzNjc4LCAxMC44Mjg5NTldLCBbNDUuOTkzNTg4LCAxMC44Mjg3NzRdLCBbNDUuOTkzNTczLCAxMC44Mjg3MTFdLCBbNDUuOTkzNjMsIDEwLjgyODY0NF0sIFs0NS45OTM2NjcsIDEwLjgyODYyNl0sIFs0NS45OTM3MTMsIDEwLjgyODYxOF0sIFs0NS45OTM3NTQsIDEwLjgyODYyM10sIFs0NS45OTM4MDIsIDEwLjgyODYzOF0sIFs0NS45OTM4NDUsIDEwLjgyODY2Nl0sIFs0NS45OTM4NzYsIDEwLjgyODcwNF0sIFs0NS45OTM5NzEsIDEwLjgyODg3OV0sIFs0NS45OTQwMjQsIDEwLjgyODk5Nl0sIFs0NS45OTQwNTQsIDEwLjgyOTA4OV0sIFs0NS45OTQwNzMsIDEwLjgyOTE1M10sIFs0NS45OTQxMDQsIDEwLjgyOTIzM10sIFs0NS45OTQxNDgsIDEwLjgyOTMzM10sIFs0NS45OTQxOCwgMTAuODI5NDMzXSwgWzQ1Ljk5NDI1LCAxMC44Mjk2MTRdLCBbNDUuOTk0MzA1LCAxMC44Mjk3MjVdLCBbNDUuOTk0Mzk4LCAxMC44Mjk4OV0sIFs0NS45OTQ0NTcsIDEwLjgzMDAwN10sIFs0NS45OTQ0OTksIDEwLjgzMDA5NF0sIFs0NS45OTQ1MzEsIDEwLjgzMDExOV0sIFs0NS45OTQ1NDUsIDEwLjgzMDExMV0sIFs0NS45OTQ1NywgMTAuODMwMDk3XSwgWzQ1Ljk5NDU2NiwgMTAuODMwMDU4XSwgWzQ1Ljk5NDU2MywgMTAuODMwMDA4XSwgWzQ1Ljk5NDU0OSwgMTAuODI5OTE2XSwgWzQ1Ljk5NDUyMSwgMTAuODI5ODEzXSwgWzQ1Ljk5NDQ5NywgMTAuODI5NzU2XSwgWzQ1Ljk5NDQ2LCAxMC44Mjk2NzJdLCBbNDUuOTk0NDEzLCAxMC44Mjk1NzhdLCBbNDUuOTk0MzU2LCAxMC44Mjk0NV0sIFs0NS45OTQzMDgsIDEwLjgyOTMzN10sIFs0NS45OTQyNjIsIDEwLjgyOTIzNl0sIFs0NS45OTQyMDcsIDEwLjgyOTEyNV0sIFs0NS45OTQxNTQsIDEwLjgyOTAyMV0sIFs0NS45OTQxMSwgMTAuODI4OTQ3XSwgWzQ1Ljk5NDA1NCwgMTAuODI4ODQ2XSwgWzQ1Ljk5NDAxNywgMTAuODI4Nzc1XSwgWzQ1Ljk5Mzk5MiwgMTAuODI4NzI4XSwgWzQ1Ljk5Mzk4NCwgMTAuODI4Njc1XSwgWzQ1Ljk5NDAxMywgMTAuODI4NjQ0XSwgWzQ1Ljk5NDA1NCwgMTAuODI4NjI5XSwgWzQ1Ljk5NDEwNywgMTAuODI4NjE4XSwgWzQ1Ljk5NDE1NSwgMTAuODI4NjE3XSwgWzQ1Ljk5NDE5NCwgMTAuODI4NjM4XSwgWzQ1Ljk5NDIzMiwgMTAuODI4Njg2XSwgWzQ1Ljk5NDI3NiwgMTAuODI4NzczXSwgWzQ1Ljk5NDM0NCwgMTAuODI4OTQzXSwgWzQ1Ljk5NDQwMywgMTAuODI5MDc0XSwgWzQ1Ljk5NDQ0OSwgMTAuODI5MTc1XSwgWzQ1Ljk5NDUxNywgMTAuODI5MzAzXSwgWzQ1Ljk5NDU2MSwgMTAuODI5NF0sIFs0NS45OTQ2NzQsIDEwLjgyOTczN10sIFs0NS45OTQ3MTcsIDEwLjgyOTg3M10sIFs0NS45OTQ3NTMsIDEwLjgyOTk5M10sIFs0NS45OTQ3ODIsIDEwLjgzMDA3N10sIFs0NS45OTQ3OTEsIDEwLjgzMDE1M10sIFs0NS45OTQ3ODEsIDEwLjgzMDIwNV0sIFs0NS45OTQ3NTgsIDEwLjgzMDIyN10sIFs0NS45OTQ2NjQsIDEwLjgzMDMxOF0sIFs0NS45OTQ2NTksIDEwLjgzMDM3MV0sIFs0NS45OTQ2NSwgMTAuODMwNDM5XSwgWzQ1Ljk5NDYyNiwgMTAuODMwNDk0XSwgWzQ1Ljk5NDU4NSwgMTAuODMwNTAyXV1dXSwgInR5cGUiOiAiTXVsdGlQb2x5Z29uIn0sICJpZCI6ICIzIiwgInByb3BlcnRpZXMiOiB7ImRlc2NyaXppb25lX29yaWdpbmUiOiAiTm9uIGNvZGlmaWNhdG8iLCAiZ21sX2lkIjogIklELkFDUVVFRklTSUNIRS5TUEVDQ0hJLkFDUVVBLjU0MTU5IiwgIm5hc3AiOiAwLCAibmF0dXJhX3NwZWNjaGlvX2FjcXVhIjogIk5vbiBjb2RpZmljYXRvIiwgIm5vbWUiOiAiTEFHTyBUT1JCSUVSQSBESSBGSUFWRVx1MDAyNyAyPyIsICJvcmlnaW5lIjogMH0sICJ0eXBlIjogIkZlYXR1cmUifSwgeyJnZW9tZXRyeSI6IHsiY29vcmRpbmF0ZXMiOiBbW1tbNDUuOTkzNjU3LCAxMC44MzM0MDddLCBbNDUuOTkzNjIxLCAxMC44MzMzNzZdLCBbNDUuOTkzNjAxLCAxMC44MzMzNDZdLCBbNDUuOTkzNTY3LCAxMC44MzMyNjldLCBbNDUuOTkzNTQ1LCAxMC44MzMyMzJdLCBbNDUuOTkzNTE4LCAxMC44MzMyMTVdLCBbNDUuOTkzNDgyLCAxMC44MzMyMDFdLCBbNDUuOTkzNDI0LCAxMC44MzMyMTVdLCBbNDUuOTkzMzk2LCAxMC44MzMyMjFdLCBbNDUuOTkzMzY0LCAxMC44MzMyMTddLCBbNDUuOTkzMzM3LCAxMC44MzMxOTNdLCBbNDUuOTkzMzE2LCAxMC44MzMxMTNdLCBbNDUuOTkzMzE0LCAxMC44MzMwNTddLCBbNDUuOTkzMzIsIDEwLjgzMjk5OF0sIFs0NS45OTMzMzIsIDEwLjgzMjkzOV0sIFs0NS45OTMzNDUsIDEwLjgzMjg4NF0sIFs0NS45OTMzNTUsIDEwLjgzMjgyOF0sIFs0NS45OTMzNjIsIDEwLjgzMjc3OV0sIFs0NS45OTMzNDcsIDEwLjgzMjcwM10sIFs0NS45OTMzMTYsIDEwLjgzMjY2OV0sIFs0NS45OTMyNTksIDEwLjgzMjYyNV0sIFs0NS45OTMyMTYsIDEwLjgzMjU4N10sIFs0NS45OTMxODEsIDEwLjgzMjUzN10sIFs0NS45OTMxNTQsIDEwLjgzMjVdLCBbNDUuOTkzMTM2LCAxMC44MzI0NjZdLCBbNDUuOTkzMTA5LCAxMC44MzI0MjZdLCBbNDUuOTkzMDgyLCAxMC44MzIzNzldLCBbNDUuOTkzMDY1LCAxMC44MzIzNDJdLCBbNDUuOTkzMDQzLCAxMC44MzIzNTJdLCBbNDUuOTkzMDEsIDEwLjgzMjI4Ml0sIFs0NS45OTI5NTcsIDEwLjgzMjE4OF0sIFs0NS45OTI5MjYsIDEwLjgzMjE0N10sIFs0NS45OTI4ODcsIDEwLjgzMjExMl0sIFs0NS45OTI4NDIsIDEwLjgzMjA4MV0sIFs0NS45OTI3OTQsIDEwLjgzMjA2Ml0sIFs0NS45OTI3MjgsIDEwLjgzMjA0N10sIFs0NS45OTI2ODcsIDEwLjgzMjAzMl0sIFs0NS45OTI2NjMsIDEwLjgzMjAxMV0sIFs0NS45OTI2MjUsIDEwLjgzMTkzXSwgWzQ1Ljk5MjYwNiwgMTAuODMxODY0XSwgWzQ1Ljk5MjU4OSwgMTAuODMxODA0XSwgWzQ1Ljk5MjU3MywgMTAuODMxNzddLCBbNDUuOTkyNTExLCAxMC44MzE2NzZdLCBbNDUuOTkyNDk5LCAxMC44MzE0OTFdLCBbNDUuOTkyNTM2LCAxMC44MzE0Nl0sIFs0NS45OTI1ODMsIDEwLjgzMTQyNV0sIFs0NS45OTI2MTcsIDEwLjgzMTQwNF0sIFs0NS45OTI2NjYsIDEwLjgzMTM4Nl0sIFs0NS45OTI2OTgsIDEwLjgzMTQwNF0sIFs0NS45OTI3MzQsIDEwLjgzMTQ0Ml0sIFs0NS45OTI3NjUsIDEwLjgzMTQ5Ml0sIFs0NS45OTI4MTEsIDEwLjgzMTU4XSwgWzQ1Ljk5Mjg2MiwgMTAuODMxNjc3XSwgWzQ1Ljk5MjkyNCwgMTAuODMxNzg4XSwgWzQ1Ljk5Mjk2OCwgMTAuODMxODgyXSwgWzQ1Ljk5MzA3NiwgMTAuODMyMDY4XSwgWzQ1Ljk5MzIyNCwgMTAuODMyMjY0XSwgWzQ1Ljk5MzI2MywgMTAuODMyMzYxXSwgWzQ1Ljk5MzMwMiwgMTAuODMyMzczXSwgWzQ1Ljk5MzM1MSwgMTAuODMyMzY1XSwgWzQ1Ljk5MzM5OSwgMTAuODMyMzYzXSwgWzQ1Ljk5MzQzMiwgMTAuODMyNDE0XSwgWzQ1Ljk5MzQ0OCwgMTAuODMyNDU3XSwgWzQ1Ljk5MzQ3NSwgMTAuODMyNDc3XSwgWzQ1Ljk5MzQ5LCAxMC44MzI0ODZdLCBbNDUuOTkzNTIyLCAxMC44MzI0ODZdLCBbNDUuOTkzNTUzLCAxMC44MzI0ODNdLCBbNDUuOTkzNTkyLCAxMC44MzI0NjhdLCBbNDUuOTkzNjMzLCAxMC44MzI0NjZdLCBbNDUuOTkzNjc2LCAxMC44MzI0OTFdLCBbNDUuOTkzNzA2LCAxMC44MzI1MThdLCBbNDUuOTkzNzM5LCAxMC44MzI1NjhdLCBbNDUuOTkzNzYxLCAxMC44MzI2MThdLCBbNDUuOTkzNzc0LCAxMC44MzI2NThdLCBbNDUuOTkzNzc4LCAxMC44MzI3MTRdLCBbNDUuOTkzNzY4LCAxMC44MzI3NTNdLCBbNDUuOTkzNzQ1LCAxMC44MzI3ODldLCBbNDUuOTkzNzE3LCAxMC44MzI4MTddLCBbNDUuOTkzNjcxLCAxMC44MzI4NDJdLCBbNDUuOTkzNjI5LCAxMC44MzI4OF0sIFs0NS45OTM2MDUsIDEwLjgzMjkxOV0sIFs0NS45OTM2MDIsIDEwLjgzMjk1OF0sIFs0NS45OTM2MjksIDEwLjgzMzAyNV0sIFs0NS45OTM2NjUsIDEwLjgzMzA2Nl0sIFs0NS45OTM2ODcsIDEwLjgzMzA4OV0sIFs0NS45OTM3MjMsIDEwLjgzMzEyN10sIFs0NS45OTM3NSwgMTAuODMzMTddLCBbNDUuOTkzNzc3LCAxMC44MzMyMzRdLCBbNDUuOTkzNzc4LCAxMC44MzMyOTNdLCBbNDUuOTkzNzczLCAxMC44MzMzMzJdLCBbNDUuOTkzNzQyLCAxMC44MzMzN10sIFs0NS45OTM2ODksIDEwLjgzMzQwNV0sIFs0NS45OTM2NTcsIDEwLjgzMzQwN11dXV0sICJ0eXBlIjogIk11bHRpUG9seWdvbiJ9LCAiaWQiOiAiNCIsICJwcm9wZXJ0aWVzIjogeyJkZXNjcml6aW9uZV9vcmlnaW5lIjogIk5vbiBjb2RpZmljYXRvIiwgImdtbF9pZCI6ICJJRC5BQ1FVRUZJU0lDSEUuU1BFQ0NISS5BQ1FVQS41NDE2MCIsICJuYXNwIjogMCwgIm5hdHVyYV9zcGVjY2hpb19hY3F1YSI6ICJOb24gY29kaWZpY2F0byIsICJub21lIjogIkxBR08gVE9SQklFUkEgREkgRklBVkVcdTAwMjcgMz8iLCAib3JpZ2luZSI6IDB9LCAidHlwZSI6ICJGZWF0dXJlIn0sIHsiZ2VvbWV0cnkiOiB7ImNvb3JkaW5hdGVzIjogW1tbWzQ2LjAxMDk3LCAxMC45NTY4MzRdLCBbNDYuMDEwODI2LCAxMC45NTY4MTldLCBbNDYuMDEwNzA1LCAxMC45NTY4MTRdLCBbNDYuMDEwNTQxLCAxMC45NTY3OThdLCBbNDYuMDEwMzQyLCAxMC45NTY3NzhdLCBbNDYuMDEwMTIxLCAxMC45NTY3MzldLCBbNDYuMDA5ODc4LCAxMC45NTY2OTldLCBbNDYuMDA5NjM5LCAxMC45NTY2NjVdLCBbNDYuMDA5Mzg3LCAxMC45NTY2MjJdLCBbNDYuMDA5MTA3LCAxMC45NTY1NjhdLCBbNDYuMDA4ODQyLCAxMC45NTY1MTVdLCBbNDYuMDA4NjExLCAxMC45NTY1MDVdLCBbNDYuMDA4MzcxLCAxMC45NTY0ODRdLCBbNDYuMDA4MTQ3LCAxMC45NTY0NjhdLCBbNDYuMDA4MDc0LCAxMC45NTY0N10sIFs0Ni4wMDc4OCwgMTAuOTU2NDddLCBbNDYuMDA3NzY3LCAxMC45NTY0OTRdLCBbNDYuMDA3NTk4LCAxMC45NTY1MjhdLCBbNDYuMDA3NDM1LCAxMC45NTY1NTVdLCBbNDYuMDA3MjExLCAxMC45NTY1MzhdLCBbNDYuMDA2OTkyLCAxMC45NTY0OTVdLCBbNDYuMDA2NzQzLCAxMC45NTY0NjJdLCBbNDYuMDA2NTc3LCAxMC45NTY0MjNdLCBbNDYuMDA2NDE1LCAxMC45NTYzNjFdLCBbNDYuMDA2MjYzLCAxMC45NTYyNDFdLCBbNDYuMDA2MDk3LCAxMC45NTYwOV0sIFs0Ni4wMDU5NTYsIDEwLjk1NjAxMl0sIFs0Ni4wMDU3OTgsIDEwLjk1NTk1N10sIFs0Ni4wMDU2MjMsIDEwLjk1NTkxMl0sIFs0Ni4wMDU0OTcsIDEwLjk1NTg5NF0sIFs0Ni4wMDUzNjcsIDEwLjk1NTg3OF0sIFs0Ni4wMDUyMzksIDEwLjk1NTg4M10sIFs0Ni4wMDUwOTcsIDEwLjk1NTg4N10sIFs0Ni4wMDQ5NjIsIDEwLjk1NTg3OF0sIFs0Ni4wMDQ4ODQsIDEwLjk1NTg2NF0sIFs0Ni4wMDQ3NzcsIDEwLjk1NTg1M10sIFs0Ni4wMDQ2NjUsIDEwLjk1NTg0NF0sIFs0Ni4wMDQ1MzUsIDEwLjk1NTgzNl0sIFs0Ni4wMDQzOTYsIDEwLjk1NTgwMV0sIFs0Ni4wMDQyMDYsIDEwLjk1NTcyNV0sIFs0Ni4wMDQwMjQsIDEwLjk1NTYzNF0sIFs0Ni4wMDM3OTYsIDEwLjk1NTUwNV0sIFs0Ni4wMDM1ODUsIDEwLjk1NTM2MV0sIFs0Ni4wMDMzOCwgMTAuOTU1MTg3XSwgWzQ2LjAwMzIzNSwgMTAuOTU1MDc2XSwgWzQ2LjAwMzExMiwgMTAuOTU1MDI1XSwgWzQ2LjAwMjkzMiwgMTAuOTU0OTQ2XSwgWzQ2LjAwMjczMywgMTAuOTU0ODc0XSwgWzQ2LjAwMjQzLCAxMC45NTQ3OTFdLCBbNDYuMDAyMjgyLCAxMC45NTQ3MzJdLCBbNDYuMDAyMTY4LCAxMC45NTQ3MTFdLCBbNDYuMDAyMDIyLCAxMC45NTQ2NV0sIFs0Ni4wMDE5MDYsIDEwLjk1NDU3Ml0sIFs0Ni4wMDE4MDQsIDEwLjk1NDQ3NV0sIFs0Ni4wMDE2OTMsIDEwLjk1NDMxNl0sIFs0Ni4wMDE1OCwgMTAuOTU0MTE3XSwgWzQ2LjAwMTQzNywgMTAuOTUzOTE0XSwgWzQ2LjAwMTMyLCAxMC45NTM3NjhdLCBbNDYuMDAxMTQxLCAxMC45NTM1NjVdLCBbNDYuMDAwOTgyLCAxMC45NTMzODJdLCBbNDYuMDAwNzk0LCAxMC45NTMxODhdLCBbNDYuMDAwNjUxLCAxMC45NTMwNDhdLCBbNDYuMDAwNDk5LCAxMC45NTI5MDFdLCBbNDYuMDAwMzk0LCAxMC45NTI4MjRdLCBbNDYuMDAwMjY2LCAxMC45NTI4MTJdLCBbNDYuMDAwMTQsIDEwLjk1Mjg2M10sIFs0Ni4wMDAwMjYsIDEwLjk1Mjg2N10sIFs0NS45OTk4ODgsIDEwLjk1Mjg1NV0sIFs0NS45OTk3NjEsIDEwLjk1Mjg0XSwgWzQ1Ljk5OTY0NiwgMTAuOTUyODA5XSwgWzQ1Ljk5OTU3NCwgMTAuOTUyNzc1XSwgWzQ1Ljk5OTQ4LCAxMC45NTI3NTddLCBbNDUuOTk5NDAyLCAxMC45NTI3NTldLCBbNDUuOTk5MzQsIDEwLjk1Mjc3OF0sIFs0NS45OTkyNCwgMTAuOTUyNzczXSwgWzQ1Ljk5OTE2NCwgMTAuOTUyNzcyXSwgWzQ1Ljk5OTEwOSwgMTAuOTUyNzc5XSwgWzQ1Ljk5OTEwOSwgMTAuOTUyNzkzXSwgWzQ1Ljk5OTA1NCwgMTAuOTUyODA0XSwgWzQ1Ljk5ODk4MiwgMTAuOTUyODQ3XSwgWzQ1Ljk5ODkxLCAxMC45NTI4OTddLCBbNDUuOTk4Nzk5LCAxMC45NTI5NjFdLCBbNDUuOTk4NzI2LCAxMC45NTI5NjVdLCBbNDUuOTk4NjI4LCAxMC45NTI5NTVdLCBbNDUuOTk4NTYzLCAxMC45NTI5ODhdLCBbNDUuOTk4NDY2LCAxMC45NTMwNl0sIFs0NS45OTgzODMsIDEwLjk1MzA4Nl0sIFs0NS45OTgzMDEsIDEwLjk1MzA2NF0sIFs0NS45OTgyMzcsIDEwLjk1MzA0NV0sIFs0NS45OTgxNjYsIDEwLjk1MzA0OF0sIFs0NS45OTgxMDksIDEwLjk1MzA4Ml0sIFs0NS45OTgwMjMsIDEwLjk1MzEyOF0sIFs0NS45OTc5NDgsIDEwLjk1MzEyOV0sIFs0NS45OTc4NzksIDEwLjk1MzEzNl0sIFs0NS45OTc3OTUsIDEwLjk1MzE5OF0sIFs0NS45OTc3MjUsIDEwLjk1MzI4N10sIFs0NS45OTc2NzYsIDEwLjk1MzM0NV0sIFs0NS45OTc1OTksIDEwLjk1MzQzNF0sIFs0NS45OTc1MzgsIDEwLjk1MzQ4N10sIFs0NS45OTc0MzgsIDEwLjk1MzQ5M10sIFs0NS45OTczNDcsIDEwLjk1MzQ1N10sIFs0NS45OTcyNywgMTAuOTUzNDI3XSwgWzQ1Ljk5NzE1OSwgMTAuOTUzMzc3XSwgWzQ1Ljk5NzAwNywgMTAuOTUzMzA5XSwgWzQ1Ljk5NjkwNSwgMTAuOTUzMjU2XSwgWzQ1Ljk5NjcxLCAxMC45NTMxNDNdLCBbNDUuOTk2NTUsIDEwLjk1MzAzNV0sIFs0NS45OTYzNzYsIDEwLjk1MjkxM10sIFs0NS45OTYxNzcsIDEwLjk1Mjc4MV0sIFs0NS45OTYwMDQsIDEwLjk1MjY1OV0sIFs0NS45OTU4MjEsIDEwLjk1MjUzN10sIFs0NS45OTU2NjMsIDEwLjk1MjQxNl0sIFs0NS45OTU1MDUsIDEwLjk1MjI5Ml0sIFs0NS45OTUzMTgsIDEwLjk1MjE0XSwgWzQ1Ljk5NTE1NiwgMTAuOTUxOTg2XSwgWzQ1Ljk5NTAzNiwgMTAuOTUxODQ0XSwgWzQ1Ljk5NDkzMiwgMTAuOTUxNzMyXSwgWzQ1Ljk5NDg0MiwgMTAuOTUxNjkyXSwgWzQ1Ljk5NDc5MSwgMTAuOTUxNjkzXSwgWzQ1Ljk5NDcwNCwgMTAuOTUxNzM2XSwgWzQ1Ljk5NDY0NCwgMTAuOTUxNjA2XSwgWzQ1Ljk5NDYzMiwgMTAuOTUxNTM2XSwgWzQ1Ljk5NDYwOSwgMTAuOTUxNDExXSwgWzQ1Ljk5NDUyMiwgMTAuOTUxMzI4XSwgWzQ1Ljk5NDQzLCAxMC45NTEyMzNdLCBbNDUuOTk0Mzc0LCAxMC45NTExNjVdLCBbNDUuOTk0MzExLCAxMC45NTEwODFdLCBbNDUuOTk0MjM2LCAxMC45NTA5NTNdLCBbNDUuOTk0MiwgMTAuOTUwODcyXSwgWzQ1Ljk5NDE4NSwgMTAuOTUwODM5XSwgWzQ1Ljk5NDEzMSwgMTAuOTUwNzA2XSwgWzQ1Ljk5NDA3MiwgMTAuOTUwNTRdLCBbNDUuOTk0MDA1LCAxMC45NTAzNTNdLCBbNDUuOTkzODgzLCAxMC45NDk5OTFdLCBbNDUuOTkzODM2LCAxMC45NDk4MjhdLCBbNDUuOTkzNzk1LCAxMC45NDk2OTVdLCBbNDUuOTkzNjQyLCAxMC45NDkzMzVdLCBbNDUuOTkzNTgsIDEwLjk0OTI0XSwgWzQ1Ljk5MzU1NSwgMTAuOTQ5MTk3XSwgWzQ1Ljk5MzUyOSwgMTAuOTQ5MTJdLCBbNDUuOTkzNDYxLCAxMC45NDg4ODVdLCBbNDUuOTkzNDUyLCAxMC45NDg3MjddLCBbNDUuOTkzNDcxLCAxMC45NDg1NzZdLCBbNDUuOTkzNDk0LCAxMC45NDg0NDNdLCBbNDUuOTkzNTMzLCAxMC45NDgyMDVdLCBbNDUuOTkzNTIyLCAxMC45NDgwN10sIFs0NS45OTM0ODEsIDEwLjk0Nzg5NF0sIFs0NS45OTM0MDcsIDEwLjk0NzcwMV0sIFs0NS45OTMzNTEsIDEwLjk0NzUzNV0sIFs0NS45OTMzMTEsIDEwLjk0NzM1OV0sIFs0NS45OTMyNywgMTAuOTQ3MTldLCBbNDUuOTkzMjQ5LCAxMC45NDY5OTldLCBbNDUuOTkzMjUxLCAxMC45NDY3ODJdLCBbNDUuOTkzMjU5LCAxMC45NDY1ODldLCBbNDUuOTkzMjY0LCAxMC45NDY0NTFdLCBbNDUuOTkzMjUsIDEwLjk0NjM0Nl0sIFs0NS45OTMyNDgsIDEwLjk0NjIxNF0sIFs0NS45OTMyNTMsIDEwLjk0NjA2XSwgWzQ1Ljk5MzI1OCwgMTAuOTQ1OTAzXSwgWzQ1Ljk5MzI1MiwgMTAuOTQ1NzM1XSwgWzQ1Ljk5MzI0LCAxMC45NDU1MjVdLCBbNDUuOTkzMjI0LCAxMC45NDUzODZdLCBbNDUuOTkzMTcxLCAxMC45NDUxOTRdLCBbNDUuOTkzMDcyLCAxMC45NDQ4NDZdLCBbNDUuOTkyOTk2LCAxMC45NDQ2MzZdLCBbNDUuOTkyODQ4LCAxMC45NDQyMTNdLCBbNDUuOTkyNzcyLCAxMC45NDQwMTddLCBbNDUuOTkyNjc2LCAxMC45NDM3ODZdLCBbNDUuOTkyNjIzLCAxMC45NDM2NjZdLCBbNDUuOTkyNTI3LCAxMC45NDM1MTVdLCBbNDUuOTkyNDQ3LCAxMC45NDM0MDNdLCBbNDUuOTkyMzM1LCAxMC45NDMyODRdLCBbNDUuOTkyMTY2LCAxMC45NDMxNTNdLCBbNDUuOTkyMDUxLCAxMC45NDMwNl0sIFs0NS45OTE4OTgsIDEwLjk0Mjk1OV0sIFs0NS45OTE3NzEsIDEwLjk0Mjg2NV0sIFs0NS45OTE2NjgsIDEwLjk0Mjc3OV0sIFs0NS45OTE1MzksIDEwLjk0MjY4Ml0sIFs0NS45OTE0MTcsIDEwLjk0MjYyMl0sIFs0NS45OTEyNzQsIDEwLjk0MjU2NF0sIFs0NS45OTExNzIsIDEwLjk0MjUyN10sIFs0NS45OTEwNzYsIDEwLjk0MjUwN10sIFs0NS45OTA5OTIsIDEwLjk0MjQ5MV0sIFs0NS45OTA5MjgsIDEwLjk0MjQ2Ml0sIFs0NS45OTA4NywgMTAuOTQyNDFdLCBbNDUuOTkwODQxLCAxMC45NDIzNDRdLCBbNDUuOTkwODMxLCAxMC45NDIyNzRdLCBbNDUuOTkwODI1LCAxMC45NDIyMDVdLCBbNDUuOTkwNzk3LCAxMC45NDIxNDJdLCBbNDUuOTkwNzQ1LCAxMC45NDIwODFdLCBbNDUuOTkwNzA1LCAxMC45NDIwNF0sIFs0NS45OTA2NzQsIDEwLjk0MTk4Nl0sIFs0NS45OTA2NTcsIDEwLjk0MTkyNl0sIFs0NS45OTA2NDksIDEwLjk0MTg1N10sIFs0NS45OTA2NTksIDEwLjk0MTc4OV0sIFs0NS45OTA2NzUsIDEwLjk0MTcxXSwgWzQ1Ljk5MDY4OSwgMTAuOTQxNjU1XSwgWzQ1Ljk5MDcxNCwgMTAuOTQxNTQ1XSwgWzQ1Ljk5MDczMywgMTAuOTQxNDE3XSwgWzQ1Ljk5MDc0NCwgMTAuOTQxMjk5XSwgWzQ1Ljk5MDc1MywgMTAuOTQxMTY1XSwgWzQ1Ljk5MDc3LCAxMC45NDEwMDhdLCBbNDUuOTkwNzg5LCAxMC45NDA4NTVdLCBbNDUuOTkwODE1LCAxMC45NDA2NzJdLCBbNDUuOTkwODM2LCAxMC45NDA1MTJdLCBbNDUuOTkwODU3LCAxMC45NDAzNzhdLCBbNDUuOTkwODgsIDEwLjk0MDI0MV0sIFs0NS45OTA5MDgsIDEwLjk0MDExNF0sIFs0NS45OTA5MjYsIDEwLjk0MDAzXSwgWzQ1Ljk5MDkyNywgMTAuOTQwMDIyXSwgWzQ1Ljk5MDk0MSwgMTAuOTM5OTU4XSwgWzQ1Ljk5MDk2LCAxMC45Mzk5MTldLCBbNDUuOTkwOTgxLCAxMC45Mzk4ODddLCBbNDUuOTkxMDE3LCAxMC45Mzk4ODJdLCBbNDUuOTkxMDQ5LCAxMC45Mzk5MTZdLCBbNDUuOTkxMDY5LCAxMC45Mzk5NTNdLCBbNDUuOTkxMDg4LCAxMC45NF0sIFs0NS45OTEwOTcsIDEwLjk0MDA0Nl0sIFs0NS45OTExMTUsIDEwLjk0MDE0NV0sIFs0NS45OTExNDksIDEwLjk0MDE4Nl0sIFs0NS45OTExODgsIDEwLjk0MDE1NV0sIFs0NS45OTEyMDUsIDEwLjk0MDEwOV0sIFs0NS45OTEyMTEsIDEwLjk0MDAzMV0sIFs0NS45OTEyMiwgMTAuOTM5OTQ2XSwgWzQ1Ljk5MTIyMywgMTAuOTM5ODldLCBbNDUuOTkxMjIyLCAxMC45Mzk4MzhdLCBbNDUuOTkxMjE2LCAxMC45Mzk3ODhdLCBbNDUuOTkxMjA4LCAxMC45Mzk3NDJdLCBbNDUuOTkxMjA1LCAxMC45Mzk2NV0sIFs0NS45OTEyMTUsIDEwLjkzOTU3Ml0sIFs0NS45OTEyNDEsIDEwLjkzOTU0XSwgWzQ1Ljk5MTI5NiwgMTAuOTM5NTE2XSwgWzQ1Ljk5MTMwMywgMTAuOTM5NTEzXSwgWzQ1Ljk5MTM2NywgMTAuOTM5NTYxXSwgWzQ1Ljk5MTM5MywgMTAuOTM5NjA4XSwgWzQ1Ljk5MTQwNCwgMTAuOTM5NjUxXSwgWzQ1Ljk5MTQxNiwgMTAuOTM5NzUzXSwgWzQ1Ljk5MTQxOSwgMTAuOTM5ODA2XSwgWzQ1Ljk5MTQxMSwgMTAuOTM5ODcxXSwgWzQ1Ljk5MTM5NSwgMTAuOTM5OTc2XSwgWzQ1Ljk5MTM3NywgMTAuOTQwMDk2XSwgWzQ1Ljk5MTM1MiwgMTAuOTQwMTk0XSwgWzQ1Ljk5MTM0MiwgMTAuOTQwMjM5XSwgWzQ1Ljk5MTMyNSwgMTAuOTQwMzE4XSwgWzQ1Ljk5MTMsIDEwLjk0MDQyMl0sIFs0NS45OTEyOCwgMTAuOTQwNTEzXSwgWzQ1Ljk5MTI2OCwgMTAuOTQwNjZdLCBbNDUuOTkxMjczLCAxMC45NDA3OThdLCBbNDUuOTkxMjg0LCAxMC45NDA4OV0sIFs0NS45OTEzMywgMTAuOTQxXSwgWzQ1Ljk5MTM4MywgMTAuOTQxMTI0XSwgWzQ1Ljk5MTQyOSwgMTAuOTQxMjU0XSwgWzQ1Ljk5MTQ2NSwgMTAuOTQxMzkzXSwgWzQ1Ljk5MTQ5LCAxMC45NDE1MTldLCBbNDUuOTkxNTA0LCAxMC45NDE2NDFdLCBbNDUuOTkxNTI0LCAxMC45NDE3ODZdLCBbNDUuOTkxNTU2LCAxMC45NDE5MjVdLCBbNDUuOTkxNjA1LCAxMC45NDIwNzFdLCBbNDUuOTkxNjYsIDEwLjk0MjIxMV0sIFs0NS45OTE3MzEsIDEwLjk0MjMxOV0sIFs0NS45OTE3OTQsIDEwLjk0MjM5NF0sIFs0NS45OTE4NDMsIDEwLjk0MjQ0MV0sIFs0NS45OTE5NjcsIDEwLjk0MjUzMl0sIFs0NS45OTIwNiwgMTAuOTQyNTk0XSwgWzQ1Ljk5MjE4LCAxMC45NDI2NjRdLCBbNDUuOTkyMjg2LCAxMC45NDI3MzFdLCBbNDUuOTkyMzg4LCAxMC45NDI3ODFdLCBbNDUuOTkyNTAyLCAxMC45NDI4MTFdLCBbNDUuOTkyNjE4LCAxMC45NDI4MjldLCBbNDUuOTkyNzI2LCAxMC45NDI4M10sIFs0NS45OTI4MjksIDEwLjk0MjgxN10sIFs0NS45OTI5MjUsIDEwLjk0Mjc5OF0sIFs0NS45OTMwMDUsIDEwLjk0Mjc3OF0sIFs0NS45OTMxMDYsIDEwLjk0Mjc2M10sIFs0NS45OTMxODIsIDEwLjk0Mjc2Ml0sIFs0NS45OTMyNTUsIDEwLjk0Mjc4OF0sIFs0NS45OTMzNTEsIDEwLjk0MjkwN10sIFs0NS45OTMzODksIDEwLjk0Mjk1NF0sIFs0NS45OTM0NDIsIDEwLjk0MzAxMl0sIFs0NS45OTM0ODcsIDEwLjk0MzA2XSwgWzQ1Ljk5MzU2NiwgMTAuOTQzMTEyXSwgWzQ1Ljk5MzYxOCwgMTAuOTQzMTRdLCBbNDUuOTkzNzA3LCAxMC45NDMxODNdLCBbNDUuOTkzNzkxLCAxMC45NDMxOTldLCBbNDUuOTkzODYyLCAxMC45NDMxOTldLCBbNDUuOTkzOTY1LCAxMC45NDMxOTNdLCBbNDUuOTk0MDc3LCAxMC45NDMyXSwgWzQ1Ljk5NDE4MiwgMTAuOTQzMjAxXSwgWzQ1Ljk5NDI3MywgMTAuOTQzMjE4XSwgWzQ1Ljk5NDM4NSwgMTAuOTQzMjUyXSwgWzQ1Ljk5NDQ3MywgMTAuOTQzMjk4XSwgWzQ1Ljk5NDU1NywgMTAuOTQzMzU3XSwgWzQ1Ljk5NDY1OCwgMTAuOTQzNDQ5XSwgWzQ1Ljk5NDc1LCAxMC45NDM1NTFdLCBbNDUuOTk0ODcsIDEwLjk0MzddLCBbNDUuOTk0OTg2LCAxMC45NDM4NTZdLCBbNDUuOTk1MTI4LCAxMC45NDM5OTZdLCBbNDUuOTk1Mjc2LCAxMC45NDQxMl0sIFs0NS45OTU0MDksIDEwLjk0NDI0M10sIFs0NS45OTU1NDksIDEwLjk0NDM1XSwgWzQ1Ljk5NTY2MiwgMTAuOTQ0NDMzXSwgWzQ1Ljk5NTc3LCAxMC45NDQ1MDddLCBbNDUuOTk1OTAxLCAxMC45NDQ1NzFdLCBbNDUuOTk1OTksIDEwLjk0NDYyXSwgWzQ1Ljk5NjA5NSwgMTAuOTQ0NzA2XSwgWzQ1Ljk5NjE1MiwgMTAuOTQ0NzYxXSwgWzQ1Ljk5NjIxMywgMTAuOTQ0NzldLCBbNDUuOTk2MjYsIDEwLjk0NDgyN10sIFs0NS45OTYyOCwgMTAuOTQ0ODY0XSwgWzQ1Ljk5NjM0NSwgMTAuOTQ1MDM0XSwgWzQ1Ljk5NjM5MSwgMTAuOTQ1MDYyXSwgWzQ1Ljk5NjQzMiwgMTAuOTQ1MDU3XSwgWzQ1Ljk5NjQ4LCAxMC45NDUwMzZdLCBbNDUuOTk2NTQ5LCAxMC45NDUwMTNdLCBbNDUuOTk2NjI0LCAxMC45NDUwNDVdLCBbNDUuOTk2NjczLCAxMC45NDUxMTNdLCBbNDUuOTk2NzA3LCAxMC45NDUxNTddLCBbNDUuOTk2NzQ3LCAxMC45NDUxOThdLCBbNDUuOTk2ODEzLCAxMC45NDUyMzNdLCBbNDUuOTk2ODc2LCAxMC45NDUyNzVdLCBbNDUuOTk2OTU2LCAxMC45NDU0MDNdLCBbNDUuOTk2OTkzLCAxMC45NDU1MjldLCBbNDUuOTk3MDM3LCAxMC45NDU1ODZdLCBbNDUuOTk3MDc1LCAxMC45NDU2NDNdLCBbNDUuOTk3MTAxLCAxMC45NDU3Ml0sIFs0NS45OTcxMzgsIDEwLjk0NTgzXSwgWzQ1Ljk5NzE4MSwgMTAuOTQ1ODY0XSwgWzQ1Ljk5NzIxLCAxMC45NDU4NzhdLCBbNDUuOTk3MzA0LCAxMC45NDU4ODJdLCBbNDUuOTk3MzY2LCAxMC45NDU4NzhdLCBbNDUuOTk3NDY5LCAxMC45NDU4NTldLCBbNDUuOTk3NTA2LCAxMC45NDU4NDRdLCBbNDUuOTk3NTU3LCAxMC45NDU4MV0sIFs0NS45OTc2MzIsIDEwLjk0NTgxM10sIFs0NS45OTc2ODIsIDEwLjk0NTg1N10sIFs0NS45OTc3MDksIDEwLjk0NTg1OF0sIFs0NS45OTc3NjksIDEwLjk0NTgzOF0sIFs0NS45OTc4MTMsIDEwLjk0NTgyM10sIFs0NS45OTc4NjIsIDEwLjk0NTg0OF0sIFs0NS45OTc5MjgsIDEwLjk0NTg5Nl0sIFs0NS45OTgwNDMsIDEwLjk0NTk4OV0sIFs0NS45OTgxMywgMTAuOTQ2MDg4XSwgWzQ1Ljk5ODIyNCwgMTAuOTQ2MjFdLCBbNDUuOTk4MzA0LCAxMC45NDYzMzRdLCBbNDUuOTk4Mzc5LCAxMC45NDY0NzhdLCBbNDUuOTk4NDQzLCAxMC45NDY2MTJdLCBbNDUuOTk4NTIsIDEwLjk0Njc1Nl0sIFs0NS45OTg2NTMsIDEwLjk0Njg3XSwgWzQ1Ljk5ODcyMSwgMTAuOTQ2ODc5XSwgWzQ1Ljk5ODc2NywgMTAuOTQ2ODc3XSwgWzQ1Ljk5ODgxMiwgMTAuOTQ2OTEyXSwgWzQ1Ljk5ODg0MSwgMTAuOTQ2OTU5XSwgWzQ1Ljk5ODg2NiwgMTAuOTQ2OTgzXSwgWzQ1Ljk5ODkyNywgMTAuOTQ3MDEyXSwgWzQ1Ljk5ODk2MywgMTAuOTQ3MDFdLCBbNDUuOTk5MDY5LCAxMC45NDY5OTRdLCBbNDUuOTk5MTA4LCAxMC45NDY5ODRdLCBbNDUuOTk5MTA4LCAxMC45NDY5OTldLCBbNDUuOTk5MTM1LCAxMC45NDY5OTRdLCBbNDUuOTk5MTkyLCAxMC45NDY5NzJdLCBbNDUuOTk5MjQyLCAxMC45NDY5NTZdLCBbNDUuOTk5MzI1LCAxMC45NDY5MzVdLCBbNDUuOTk5NDQsIDEwLjk0Njg5N10sIFs0NS45OTk1MjksIDEwLjk0Njg3Nl0sIFs0NS45OTk2MjcsIDEwLjk0Njg3NF0sIFs0NS45OTk3MDcsIDEwLjk0Njg5NV0sIFs0NS45OTk4MDMsIDEwLjk0NjkzOV0sIFs0NS45OTk5NDksIDEwLjk0Njk5N10sIFs0Ni4wMDAwNTYsIDEwLjk0NzAzNV0sIFs0Ni4wMDAxNTgsIDEwLjk0NzA2OV0sIFs0Ni4wMDAyNjEsIDEwLjk0NzE0XSwgWzQ2LjAwMDM1OCwgMTAuOTQ3MjNdLCBbNDYuMDAwNDI2LCAxMC45NDczMjZdLCBbNDYuMDAwNTg1LCAxMC45NDc0N10sIFs0Ni4wMDA2NjksIDEwLjk0NzU0XSwgWzQ2LjAwMDc5LCAxMC45NDc1ODhdLCBbNDYuMDAwOTE4LCAxMC45NDc2Ml0sIFs0Ni4wMDEwMDcsIDEwLjk0NzY1N10sIFs0Ni4wMDExMDUsIDEwLjk0NzcyNV0sIFs0Ni4wMDEyLCAxMC45NDc4MDVdLCBbNDYuMDAxMzE0LCAxMC45NDc4OTVdLCBbNDYuMDAxMzkxLCAxMC45NDc5NjJdLCBbNDYuMDAxNTE2LCAxMC45NDgwNzNdLCBbNDYuMDAxNjE2LCAxMC45NDgxNDNdLCBbNDYuMDAxNzU1LCAxMC45NDgyMjFdLCBbNDYuMDAxOTU1LCAxMC45NDgzXSwgWzQ2LjAwMjE4NCwgMTAuOTQ4Mzc5XSwgWzQ2LjAwMjQyMSwgMTAuOTQ4NDQyXSwgWzQ2LjAwMjYwMywgMTAuOTQ4NDk3XSwgWzQ2LjAwMjc3OSwgMTAuOTQ4NTk4XSwgWzQ2LjAwMjkzNSwgMTAuOTQ4NzI5XSwgWzQ2LjAwMzA3NCwgMTAuOTQ4ODM2XSwgWzQ2LjAwMzIxMSwgMTAuOTQ4OTE0XSwgWzQ2LjAwMzQxNSwgMTAuOTQ5MDY4XSwgWzQ2LjAwMzY0OSwgMTAuOTQ5Mjc1XSwgWzQ2LjAwMzg0MywgMTAuOTQ5NDE2XSwgWzQ2LjAwMzk5OSwgMTAuOTQ5NTI3XSwgWzQ2LjAwNDE3OSwgMTAuOTQ5NjcxXSwgWzQ2LjAwNDMzNiwgMTAuOTQ5ODA4XSwgWzQ2LjAwNDQ1MiwgMTAuOTQ5ODg2XSwgWzQ2LjAwNDU2NiwgMTAuOTQ5OTI3XSwgWzQ2LjAwNDY3OCwgMTAuOTQ5OTU4XSwgWzQ2LjAwNDgxNCwgMTAuOTUwMDA3XSwgWzQ2LjAwNDk3LCAxMC45NTAwMzldLCBbNDYuMDA1MDkxLCAxMC45NTAwNF0sIFs0Ni4wMDUyLCAxMC45NTAwNjVdLCBbNDYuMDA1MzAzLCAxMC45NTAxMDldLCBbNDYuMDA1NDMsIDEwLjk1MDIwM10sIFs0Ni4wMDU1NzYsIDEwLjk1MDMxN10sIFs0Ni4wMDU3MDUsIDEwLjk1MDQzOF0sIFs0Ni4wMDU4NzUsIDEwLjk1MDU5MV0sIFs0Ni4wMDYwMzUsIDEwLjk1MDcyMl0sIFs0Ni4wMDYyNTcsIDEwLjk1MDldLCBbNDYuMDA2NDI4LCAxMC45NTEwNDddLCBbNDYuMDA2NTk4LCAxMC45NTEyMTFdLCBbNDYuMDA2NzU5LCAxMC45NTEzODRdLCBbNDYuMDA2OTExLCAxMC45NTE1NDddLCBbNDYuMDA3MDU2LCAxMC45NTE3MDddLCBbNDYuMDA3MjI0LCAxMC45NTE4NzddLCBbNDYuMDA3MzY0LCAxMC45NTIwNDddLCBbNDYuMDA3NTAzLCAxMC45NTIyMV0sIFs0Ni4wMDc2MjgsIDEwLjk1MjI4OF0sIFs0Ni4wMDc3NCwgMTAuOTUyMzAzXSwgWzQ2LjAwNzg0NSwgMTAuOTUyMzQ3XSwgWzQ2LjAwODAxNSwgMTAuOTUyNDg4XSwgWzQ2LjAwODEzMywgMTAuOTUyNTcyXSwgWzQ2LjAwODE2NSwgMTAuOTUyNjA4XSwgWzQ2LjAwODIzMSwgMTAuOTUyNjYyXSwgWzQ2LjAwODI1OCwgMTAuOTUyNjYyXSwgWzQ2LjAwODMwOCwgMTAuOTUyNzEyXSwgWzQ2LjAwODM5NCwgMTAuOTUyODA2XSwgWzQ2LjAwODQ3OSwgMTAuOTUyODY2XSwgWzQ2LjAwODUxNSwgMTAuOTUyODkzXSwgWzQ2LjAwODY3MSwgMTAuOTUzMDYzXSwgWzQ2LjAwODgwMSwgMTAuOTUzMTg3XSwgWzQ2LjAwODk1NSwgMTAuOTUzMzA0XSwgWzQ2LjAwOTAyOCwgMTAuOTUzMzQ4XSwgWzQ2LjAwOTEyNiwgMTAuOTUzNDA5XSwgWzQ2LjAwOTI2NywgMTAuOTUzNDc2XSwgWzQ2LjAwOTM1NCwgMTAuOTUzNTExXSwgWzQ2LjAwOTM5NSwgMTAuOTUzNTc0XSwgWzQ2LjAwOTQxOCwgMTAuOTUzNjYxXSwgWzQ2LjAwOTQyNiwgMTAuOTUzNjg5XSwgWzQ2LjAwOTQyNywgMTAuOTUzNzU4XSwgWzQ2LjAwOTQxMywgMTAuOTUzODM0XSwgWzQ2LjAwOTM3MiwgMTAuOTUzODk1XSwgWzQ2LjAwOTMzNywgMTAuOTUzOTE0XSwgWzQ2LjAwOTI3NSwgMTAuOTUzOTEzXSwgWzQ2LjAwOTE5MywgMTAuOTUzOTI5XSwgWzQ2LjAwOTEzMywgMTAuOTUzOTU3XSwgWzQ2LjAwOTA1NSwgMTAuOTU0MDE1XSwgWzQ2LjAwOTAwMiwgMTAuOTU0MV0sIFs0Ni4wMDg5NjcsIDEwLjk1NDE3NV0sIFs0Ni4wMDg5NDYsIDEwLjk1NDI3M10sIFs0Ni4wMDg5MDgsIDEwLjk1NDM3N10sIFs0Ni4wMDg4NjksIDEwLjk1NDQ1OV0sIFs0Ni4wMDg4NzUsIDEwLjk1NDU5XSwgWzQ2LjAwODkxNiwgMTAuOTU0NjE3XSwgWzQ2LjAwODk3MywgMTAuOTU0NjE4XSwgWzQ2LjAwOTA0NCwgMTAuOTU0NjE5XSwgWzQ2LjAwOTEzNSwgMTAuOTU0NjA4XSwgWzQ2LjAwOTE5OSwgMTAuOTU0NjI1XSwgWzQ2LjAwOTIxNywgMTAuOTU0NjY4XSwgWzQ2LjAwOTIxNSwgMTAuOTU0NjkzXSwgWzQ2LjAwOTIxLCAxMC45NTQ3NTNdLCBbNDYuMDA5MTgsIDEwLjk1NDc5Ml0sIFs0Ni4wMDkxNDcsIDEwLjk1NDg0OF0sIFs0Ni4wMDkxMzUsIDEwLjk1NDldLCBbNDYuMDA5MTQsIDEwLjk1NDk0OV0sIFs0Ni4wMDkxNiwgMTAuOTU1MDEyXSwgWzQ2LjAwOTIsIDEwLjk1NTA4Ml0sIFs0Ni4wMDkyNDgsIDEwLjk1NTExNV0sIFs0Ni4wMDkzMDksIDEwLjk1NTE2Ml0sIFs0Ni4wMDkzOSwgMTAuOTU1Mzg3XSwgWzQ2LjAwOTQxMiwgMTAuOTU1NDk2XSwgWzQ2LjAwOTQzNiwgMTAuOTU1NjU0XSwgWzQ2LjAwOTQ2LCAxMC45NTU4MThdLCBbNDYuMDA5NDc1LCAxMC45NTU5NDNdLCBbNDYuMDA5NTM4LCAxMC45NTYwODldLCBbNDYuMDA5NjE1LCAxMC45NTYxODldLCBbNDYuMDA5Njc0LCAxMC45NTYyMzJdLCBbNDYuMDA5NzQ3LCAxMC45NTYyNjNdLCBbNDYuMDA5ODM4LCAxMC45NTYyOTFdLCBbNDYuMDA5OTkzLCAxMC45NTYzMjldLCBbNDYuMDEwMTI4LCAxMC45NTYzNDhdLCBbNDYuMDEwMjM1LCAxMC45NTYzNTldLCBbNDYuMDEwMzM4LCAxMC45NTYzNjFdLCBbNDYuMDEwNDUzLCAxMC45NTYzNzNdLCBbNDYuMDEwNTQ4LCAxMC45NTYzODddLCBbNDYuMDEwNjU4LCAxMC45NTY0MDZdLCBbNDYuMDEwNzg4LCAxMC45NTY0MzRdLCBbNDYuMDEwODk1LCAxMC45NTY0NTVdLCBbNDYuMDExMDEsIDEwLjk1NjQ2N10sIFs0Ni4wMTA5OSwgMTAuOTU2NjU0XSwgWzQ2LjAxMDk3LCAxMC45NTY4MzRdXV1dLCAidHlwZSI6ICJNdWx0aVBvbHlnb24ifSwgImlkIjogIjUiLCAicHJvcGVydGllcyI6IHsiZGVzY3JpemlvbmVfb3JpZ2luZSI6ICJOb24gY29kaWZpY2F0byIsICJnbWxfaWQiOiAiSUQuQUNRVUVGSVNJQ0hFLlNQRUNDSEkuQUNRVUEuNTQxNTYiLCAibmFzcCI6IDAsICJuYXR1cmFfc3BlY2NoaW9fYWNxdWEiOiAiTm9uIGNvZGlmaWNhdG8iLCAibm9tZSI6ICJMQUdPIERJIENBVkVESU5FIiwgIm9yaWdpbmUiOiAwfSwgInR5cGUiOiAiRmVhdHVyZSJ9LCB7Imdlb21ldHJ5IjogeyJjb29yZGluYXRlcyI6IFtbW1s0NS45MzU3MDksIDEwLjgxOTk1XSwgWzQ1LjkzNTYyNywgMTAuODE5OTQ2XSwgWzQ1LjkzNTU3LCAxMC44MTk5MTldLCBbNDUuOTM1NTA4LCAxMC44MTk4ODNdLCBbNDUuOTM1NDQsIDEwLjgxOTgzN10sIFs0NS45MzUzOTcsIDEwLjgxOTgwMV0sIFs0NS45MzUzNjIsIDEwLjgxOTc2MV0sIFs0NS45MzUzMzcsIDEwLjgxOTcyMl0sIFs0NS45MzUzMSwgMTAuODE5NjM5XSwgWzQ1LjkzNTI5NywgMTAuODE5NTcxXSwgWzQ1LjkzNTI5OSwgMTAuODE5NDkyXSwgWzQ1LjkzNTMzNiwgMTAuODE5MzldLCBbNDUuOTM1MzczLCAxMC44MTkzMDVdLCBbNDUuOTM1NDIxLCAxMC44MTkyMDddLCBbNDUuOTM1NDcyLCAxMC44MTkwNjddLCBbNDUuOTM1NjI4LCAxMC44MTg3MTNdLCBbNDUuOTM1NzI1LCAxMC44MTg1MjddLCBbNDUuOTM1ODA3LCAxMC44MTgzNTRdLCBbNDUuOTM1OTAyLCAxMC44MTgxMTFdLCBbNDUuOTM1OTY2LCAxMC44MTc5MjVdLCBbNDUuOTM2MDA4LCAxMC44MTc3MzJdLCBbNDUuOTM2MDUyLCAxMC44MTc0ODNdLCBbNDUuOTM2MDgzLCAxMC44MTcyNDRdLCBbNDUuOTM2MTA2LCAxMC44MTY5OThdLCBbNDUuOTM2MTM5LCAxMC44MTY3NTldLCBbNDUuOTM2MTU1LCAxMC44MTY1NzVdLCBbNDUuOTM2MTcyLCAxMC44MTYzODhdLCBbNDUuOTM2MTYzLCAxMC44MTYyMTVdLCBbNDUuOTM2MTQxLCAxMC44MTYwNTFdLCBbNDUuOTM2MTA5LCAxMC44MTU4OTNdLCBbNDUuOTM2MDYyLCAxMC44MTU3MjJdLCBbNDUuOTM2MDEyLCAxMC44MTU1NjJdLCBbNDUuOTM1OTYyLCAxMC44MTU0MDFdLCBbNDUuOTM1OTM1LCAxMC44MTUyODZdLCBbNDUuOTM1OTIxLCAxMC44MTUxNTVdLCBbNDUuOTM1OTI0LCAxMC44MTUwODFdLCBbNDUuOTM1OTI2LCAxMC44MTUwMl0sIFs0NS45MzU5NDcsIDEwLjgxNDkwMl0sIFs0NS45MzU5ODksIDEwLjgxNDc3OF0sIFs0NS45MzYwMjgsIDEwLjgxNDY3XSwgWzQ1LjkzNjA2OSwgMTAuODE0NTY1XSwgWzQ1LjkzNjE4NCwgMTAuODE0Mzc2XSwgWzQ1LjkzNjI2NSwgMTAuODE0MjYxXSwgWzQ1LjkzNjMzMywgMTAuODE0MTc2XSwgWzQ1LjkzNjQxNCwgMTAuODE0MTAxXSwgWzQ1LjkzNjQ5NiwgMTAuODE0MDMzXSwgWzQ1LjkzNjY3LCAxMC44MTM4OTNdLCBbNDUuOTM2NzI1LCAxMC44MTM4MzFdLCBbNDUuOTM2Nzk0LCAxMC44MTM3NTNdLCBbNDUuOTM2ODU2LCAxMC44MTM2OTddLCBbNDUuOTM2OTY4LCAxMC44MTM2MTJdLCBbNDUuOTM3MDIzLCAxMC44MTM1ODNdLCBbNDUuOTM3MTA1LCAxMC44MTM1NTRdLCBbNDUuOTM3MTkyLCAxMC44MTM1MzhdLCBbNDUuOTM3MjkzLCAxMC44MTM1MTJdLCBbNDUuOTM3MzY0LCAxMC44MTM0ODNdLCBbNDUuOTM3NDMsIDEwLjgxMzQ1MV0sIFs0NS45Mzc1MTcsIDEwLjgxMzQxMl0sIFs0NS45Mzc1OTcsIDEwLjgxMzM2Nl0sIFs0NS45Mzc2NTksIDEwLjgxMzMyMV0sIFs0NS45Mzc3NTEsIDEwLjgxMzIyXSwgWzQ1LjkzNzgzOCwgMTAuODEzMTI4XSwgWzQ1LjkzNzkzOSwgMTAuODEzMDQ3XSwgWzQ1LjkzODAzOSwgMTAuODEyOTk1XSwgWzQ1LjkzODE0NSwgMTAuODEyOTU2XSwgWzQ1LjkzODI1LCAxMC44MTI5M10sIFs0NS45MzgzNTcsIDEwLjgxMjkzMV0sIFs0NS45Mzg0NjcsIDEwLjgxMjk0NF0sIFs0NS45Mzg1NzksIDEwLjgxMjk3MV0sIFs0NS45Mzg2NjgsIDEwLjgxMjk5OF0sIFs0NS45Mzg3NjgsIDEwLjgxMzA0NF0sIFs0NS45Mzg4NTcsIDEwLjgxMzA5N10sIFs0NS45Mzg5MjgsIDEwLjgxMzEzXSwgWzQ1LjkzOTAxLCAxMC44MTMxNjNdLCBbNDUuOTM5MDcsIDEwLjgxMzE5XSwgWzQ1LjkzOTEzOCwgMTAuODEzMjJdLCBbNDUuOTM5MjMsIDEwLjgxMzI1Nl0sIFs0NS45MzkzMTIsIDEwLjgxMzI3Nl0sIFs0NS45MzkzOCwgMTAuODEzM10sIFs0NS45Mzk0NiwgMTAuODEzMzFdLCBbNDUuOTM5NTQ5LCAxMC44MTMzMV0sIFs0NS45Mzk2NDMsIDEwLjgxMzMxMV0sIFs0NS45Mzk3MzksIDEwLjgxMzMxNF0sIFs0NS45Mzk4MzcsIDEwLjgxMzMzMV0sIFs0NS45Mzk5NDUsIDEwLjgxMzM1NV0sIFs0NS45NDAxMjEsIDEwLjgxMzQ0MV0sIFs0NS45NDAxOTYsIDEwLjgxMzQ5XSwgWzQ1Ljk0MDI2MiwgMTAuODEzNTE0XSwgWzQ1Ljk0MDQzMywgMTAuODEzNTQ3XSwgWzQ1Ljk0MDUzOCwgMTAuODEzNTU0XSwgWzQ1Ljk0MDY1LCAxMC44MTM1NjhdLCBbNDUuOTQwNzI4LCAxMC44MTM1OTVdLCBbNDUuOTQwODIyLCAxMC44MTM2NDddLCBbNDUuOTQwOTExLCAxMC44MTM2OTddLCBbNDUuOTQxMDAyLCAxMC44MTM3NF0sIFs0NS45NDExMDksIDEwLjgxMzc3N10sIFs0NS45NDEyMDUsIDEwLjgxMzc4N10sIFs0NS45NDEzNDIsIDEwLjgxMzk0NV0sIFs0NS45NDEzMzksIDEwLjgxNDAzNF0sIFs0NS45NDEzMjYsIDEwLjgxNDA3XSwgWzQ1Ljk0MTMsIDEwLjgxNDA5OV0sIFs0NS45NDEyNjYsIDEwLjgxNDExOF0sIFs0NS45NDEyMiwgMTAuODE0MTI4XSwgWzQ1Ljk0MTE4NiwgMTAuODE0MTI4XSwgWzQ1Ljk0MTA3NCwgMTAuODE0MjQ5XSwgWzQ1Ljk0MTEwMSwgMTAuODE0MzQxXSwgWzQ1Ljk0MTE0LCAxMC44MTQ0MV0sIFs0NS45NDExNjksIDEwLjgxNDQ1OV0sIFs0NS45NDEyNDIsIDEwLjgxNDU4N10sIFs0NS45NDEyNzYsIDEwLjgxNDY4Nl0sIFs0NS45NDEyODcsIDEwLjgxNDgyN10sIFs0NS45NDEyNzksIDEwLjgxNDg3XSwgWzQ1Ljk0MTI2OCwgMTAuODE0OTI4XSwgWzQ1Ljk0MTI0OCwgMTAuODE0OTc0XSwgWzQ1Ljk0MTIwNiwgMTAuODE1MDMzXSwgWzQ1Ljk0MTE1NiwgMTAuODE1MDg4XSwgWzQ1Ljk0MTA5LCAxMC44MTUxNDRdLCBbNDUuOTQxMDI1LCAxMC44MTUxNl0sIFs0NS45NDA5NTUsIDEwLjgxNTE4Nl0sIFs0NS45NDA4ODgsIDEwLjgxNTIyNV0sIFs0NS45NDA4MDEsIDEwLjgxNTI4M10sIFs0NS45NDA3MDcsIDEwLjgxNTMzMl0sIFs0NS45NDA2NSwgMTAuODE1Mzk0XSwgWzQ1Ljk0MDU2NywgMTAuODE1NTA1XSwgWzQ1Ljk0MDM4OSwgMTAuODE1NTc2XSwgWzQ1Ljk0MDMwNCwgMTAuODE1NjM1XSwgWzQ1Ljk0MDI0MiwgMTAuODE1N10sIFs0NS45NDAxOTQsIDEwLjgxNTc3OV0sIFs0NS45NDAxMzcsIDEwLjgxNTldLCBbNDUuOTQwMTAyLCAxMC44MTYwMTRdLCBbNDUuOTQwMDY1LCAxMC44MTYxMjJdLCBbNDUuOTQwMDE3LCAxMC44MTYyMDddLCBbNDUuOTM5OTc2LCAxMC44MTYyODNdLCBbNDUuOTM5OTQ4LCAxMC44MTYzMzJdLCBbNDUuOTM5OTE4LCAxMC44MTY0XSwgWzQ1LjkzOTkwOSwgMTAuODE2NDcyXSwgWzQ1LjkzOTk0LCAxMC44MTY3ODRdLCBbNDUuOTM5OTk5LCAxMC44MTY4OTJdLCBbNDUuOTQwMDU2LCAxMC44MTY5ODFdLCBbNDUuOTQwMDk5LCAxMC44MTcwNF0sIFs0NS45NDAxNzIsIDEwLjgxNzExXSwgWzQ1Ljk0MDIzNCwgMTAuODE3MTg1XSwgWzQ1Ljk0MDMwNCwgMTAuODE3Mjc0XSwgWzQ1Ljk0MDM4NCwgMTAuODE3MzZdLCBbNDUuOTQwNDU3LCAxMC44MTc0MzldLCBbNDUuOTQwNTM3LCAxMC44MTc1MzFdLCBbNDUuOTQwNTg5LCAxMC44MTc2MV0sIFs0NS45NDA2NTcsIDEwLjgxNzcwOV0sIFs0NS45NDA3MDcsIDEwLjgxNzc3MV0sIFs0NS45NDA3NiwgMTAuODE3ODIxXSwgWzQ1Ljk0MDgwOCwgMTAuODE3ODc3XSwgWzQ1Ljk0MDgxNCwgMTAuODE3OTI5XSwgWzQ1Ljk0MDgxMiwgMTAuODE3OTk4XSwgWzQ1Ljk0MDc3NywgMTAuODE4MDRdLCBbNDUuOTQwNzI1LCAxMC44MTgwNzNdLCBbNDUuOTQwNjkxLCAxMC44MTgwNzldLCBbNDUuOTQwNjI3LCAxMC44MTgwODZdLCBbNDUuOTQwNTY1LCAxMC44MTgwODVdLCBbNDUuOTQwNTE1LCAxMC44MTgwNzhdLCBbNDUuOTQwNDE5LCAxMC44MTgwMzldLCBbNDUuOTQwMzU1LCAxMC44MTc5OTJdLCBbNDUuOTQwMjg5LCAxMC44MTc5NDNdLCBbNDUuOTQwMjA2LCAxMC44MTc4OV0sIFs0NS45NDAxMzYsIDEwLjgxNzg1NF0sIFs0NS45NDAwNTYsIDEwLjgxNzgxNF0sIFs0NS45Mzk5ODMsIDEwLjgxNzc4MV0sIFs0NS45Mzk5MDMsIDEwLjgxNzc2N10sIFs0NS45Mzk3ODksIDEwLjgxNzc0N10sIFs0NS45Mzk3MDYsIDEwLjgxNzczNF0sIFs0NS45Mzk2MTksIDEwLjgxNzc0XSwgWzQ1LjkzOTUzNywgMTAuODE3ODA1XSwgWzQ1LjkzOTQ4NywgMTAuODE3ODVdLCBbNDUuOTM5MzQyLCAxMC44MTc5ODRdLCBbNDUuOTM5MjUzLCAxMC44MTgwNTldLCBbNDUuOTM5MTM4LCAxMC44MTgxNDRdLCBbNDUuOTM5MDI2LCAxMC44MTgyMjVdLCBbNDUuOTM4OSwgMTAuODE4MzE2XSwgWzQ1LjkzODc3OSwgMTAuODE4NDExXSwgWzQ1LjkzODYwNywgMTAuODE4NTQ0XSwgWzQ1LjkzODU1NCwgMTAuODE4NTg0XSwgWzQ1LjkzODE3MiwgMTAuODE4ODQ3XSwgWzQ1LjkzODA1NSwgMTAuODE4OTM4XSwgWzQ1LjkzNzc5NiwgMTAuODE5MTVdLCBbNDUuOTM3Njc5LCAxMC44MTkyMzVdLCBbNDUuOTM3NTc5LCAxMC44MTkzMV0sIFs0NS45Mzc0NDMsIDEwLjgxOTQxN10sIFs0NS45MzczMzgsIDEwLjgxOTQ5OV0sIFs0NS45MzcyMywgMTAuODE5NTldLCBbNDUuOTM3MTQ2LCAxMC44MTk2NTJdLCBbNDUuOTM3MDY4LCAxMC44MTk3MDddLCBbNDUuOTM2OTkyLCAxMC44MTk3MzNdLCBbNDUuOTM2OTE5LCAxMC44MTk3NDZdLCBbNDUuOTM2ODMyLCAxMC44MTk3NThdLCBbNDUuOTM2NjYxLCAxMC44MTk3NzFdLCBbNDUuOTM2NTc2LCAxMC44MTk3NjRdLCBbNDUuOTM2NTAxLCAxMC44MTk3Nl0sIFs0NS45MzYzODksIDEwLjgxOTc2OV0sIFs0NS45MzYzMTYsIDEwLjgxOTc3Nl0sIFs0NS45MzYyNDIsIDEwLjgxOTc4OF0sIFs0NS45MzYxNTMsIDEwLjgxOTgwNF0sIFs0NS45MzYwOTEsIDEwLjgxOTgxN10sIFs0NS45MzU5OTUsIDEwLjgxOTg0Nl0sIFs0NS45MzU5MDYsIDEwLjgxOTg4Ml0sIFs0NS45MzU3OTIsIDEwLjgxOTk0XSwgWzQ1LjkzNTcwOSwgMTAuODE5OTVdXV1dLCAidHlwZSI6ICJNdWx0aVBvbHlnb24ifSwgImlkIjogIjYiLCAicHJvcGVydGllcyI6IHsiZGVzY3JpemlvbmVfb3JpZ2luZSI6ICJOb24gY29kaWZpY2F0byIsICJnbWxfaWQiOiAiSUQuQUNRVUVGSVNJQ0hFLlNQRUNDSEkuQUNRVUEuNTQxNzYiLCAibmFzcCI6IDAsICJuYXR1cmFfc3BlY2NoaW9fYWNxdWEiOiAiTm9uIGNvZGlmaWNhdG8iLCAibm9tZSI6ICJMQUdPIERJIFRFTk5PIiwgIm9yaWdpbmUiOiAwfSwgInR5cGUiOiAiRmVhdHVyZSJ9LCB7Imdlb21ldHJ5IjogeyJjb29yZGluYXRlcyI6IFtbW1s0NS44Mzc2MywgMTAuODI1ODUzXSwgWzQ1LjgzNzcxLCAxMC44MjU5XSwgWzQ1LjgzNzc5OSwgMTAuODI1OTExXSwgWzQ1LjgzNzkzMSwgMTAuODI1OTFdLCBbNDUuODM4MDY2LCAxMC44MjU5MzhdLCBbNDUuODM4MTYyLCAxMC44MjU5NTNdLCBbNDUuODM4MzAxLCAxMC44MjU5NTVdLCBbNDUuODM4NDQ3LCAxMC44MjU5OTZdLCBbNDUuODM4NTk4LCAxMC44MjYwNjFdLCBbNDUuODM4NzIzLCAxMC44MjYwODJdLCBbNDUuODM4ODc2LCAxMC44MjYxMjddLCBbNDUuODM5MTIsIDEwLjgyNjIyM10sIFs0NS44MzkzMTEsIDEwLjgyNjMwNF0sIFs0NS44Mzk0ODEsIDEwLjgyNjI5XSwgWzQ1LjgzOTY4MiwgMTAuODI2MjYxXSwgWzQ1LjgzOTk2LCAxMC44MjYyOTRdLCBbNDUuODQwNDA1LCAxMC44MjYyMDNdLCBbNDUuODQwNTU2LCAxMC44MjYxNDNdLCBbNDUuODQwNzg5LCAxMC44MjYxNjZdLCBbNDUuODQwOTQyLCAxMC44MjYxNzhdLCBbNDUuODQxMDU2LCAxMC44MjYxNDRdLCBbNDUuODQxMTc1LCAxMC44MjYxNDJdLCBbNDUuODQxMjY0LCAxMC44MjYxN10sIFs0NS44NDEzMzUsIDEwLjgyNjE3MV0sIFs0NS44NDE0MjksIDEwLjgyNjE1OV0sIFs0NS44NDE1NCwgMTAuODI2MjE3XSwgWzQ1Ljg0MTYyNCwgMTAuODI2MzA2XSwgWzQ1Ljg0MTgzOCwgMTAuODI2NDE3XSwgWzQ1Ljg0MTk4NCwgMTAuODI2NTE1XSwgWzQ1Ljg0MjI1MywgMTAuODI2NjI3XSwgWzQ1Ljg0MjQwNiwgMTAuODI2NjQ5XSwgWzQ1Ljg0MjYwNCwgMTAuODI2NjkxXSwgWzQ1Ljg0Mjc2OCwgMTAuODI2NzQyXSwgWzQ1Ljg0MjkzLCAxMC44MjY4MDFdLCBbNDUuODQzMTEyLCAxMC44MjY5NTRdLCBbNDUuODQzMzE2LCAxMC44MjcxMzddLCBbNDUuODQzNDkzLCAxMC44MjcyNzRdLCBbNDUuODQzNzI5LCAxMC44Mjc1MjZdLCBbNDUuODQzOTc2LCAxMC44Mjc4MzFdLCBbNDUuODQ0MjYxLCAxMC44MjgzMjNdLCBbNDUuODQ0MzM3LCAxMC44Mjg0NjhdLCBbNDUuODQ0NDM5LCAxMC44Mjg2MDFdLCBbNDUuODQ0NTcsIDEwLjgyODc5Ml0sIFs0NS44NDQ3LCAxMC44Mjg5MDldLCBbNDUuODQ0ODYxLCAxMC44Mjg5OTddLCBbNDUuODQ1MDQ4LCAxMC44MjkwNTJdLCBbNDUuODQ1MTgyLCAxMC44MjkxNTldLCBbNDUuODQ1MzA1LCAxMC44MjkzMDhdLCBbNDUuODQ1MzcsIDEwLjgyOTQ0Nl0sIFs0NS44NDU0NDYsIDEwLjgyOTYwOF0sIFs0NS44NDU1NTcsIDEwLjgyOTc4Nl0sIFs0NS44NDU2MTMsIDEwLjgyOTkzMV0sIFs0NS44NDU3NTMsIDEwLjgzMDE2OV0sIFs0NS44NDU4MzUsIDEwLjgzMDI3Ml0sIFs0NS44NDU5MDMsIDEwLjgzMDMwMl0sIFs0NS44NDYwNCwgMTAuODMwMzA0XSwgWzQ1Ljg0NjE2MSwgMTAuODMwMzM1XSwgWzQ1Ljg0NjMwMiwgMTAuODMwNDE5XSwgWzQ1Ljg0NjQ1MiwgMTAuODMwNDk3XSwgWzQ1Ljg0NjU1NSwgMTAuODMwNDg5XSwgWzQ1Ljg0NjY3NiwgMTAuODMwNTUzXSwgWzQ1Ljg0Njc2NSwgMTAuODMwNTc3XSwgWzQ1Ljg0NjkyMiwgMTAuODMwNTk5XSwgWzQ1Ljg0NzA2MSwgMTAuODMwNjQ0XSwgWzQ1Ljg0NzE4OSwgMTAuODMwNjg4XSwgWzQ1Ljg0NzM1NiwgMTAuODMwNzA0XSwgWzQ1Ljg0NzU3OCwgMTAuODMwNjg3XSwgWzQ1Ljg0Nzc2NywgMTAuODMwNjgxXSwgWzQ1Ljg0Nzk4MSwgMTAuODMwNzkyXSwgWzQ1Ljg0ODI4OSwgMTAuODMwOTExXSwgWzQ1Ljg0ODY4NiwgMTAuODMxMDU1XSwgWzQ1Ljg0ODk3MiwgMTAuODMxMjM5XSwgWzQ1Ljg0OTEsIDEwLjgzMTMxNF0sIFs0NS44NDk3MTMsIDEwLjgzMTQ4Nl0sIFs0NS44NDk5MTksIDEwLjgzMTU0NF0sIFs0NS44NTAxNzksIDEwLjgzMTY0M10sIFs0NS44NTA0MDIsIDEwLjgzMTc1NF0sIFs0NS44NTA2MzUsIDEwLjgzMTg1Ml0sIFs0NS44NTA4NjYsIDEwLjgzMjAyN10sIFs0NS44NTExMzQsIDEwLjgzMjE3N10sIFs0NS44NTEzNzIsIDEwLjgzMjMzOF0sIFs0NS44NTE2ODksIDEwLjgzMjQ4OF0sIFs0NS44NTE4MjEsIDEwLjgzMjU1OV0sIFs0NS44NTE5OTgsIDEwLjgzMjY2MV0sIFs0NS44NTIxNTcsIDEwLjgzMjc3OV0sIFs0NS44NTIzNzEsIDEwLjgzMjkyNF0sIFs0NS44NTI1MTYsIDEwLjgzMzAxOV0sIFs0NS44NTI2NDYsIDEwLjgzMzA5N10sIFs0NS44NTI4MTQsIDEwLjgzMzE3OV0sIFs0NS44NTMwMDEsIDEwLjgzMzIyNl0sIFs0NS44NTMxNjMsIDEwLjgzMzMyOF0sIFs0NS44NTMzNTQsIDEwLjgzMzQ1M10sIFs0NS44NTM3MjEsIDEwLjgzMzc2OF0sIFs0NS44NTM5NTEsIDEwLjgzMzg1Ml0sIFs0NS44NTQxNzYsIDEwLjgzNDA0M10sIFs0NS44NTQzMDEsIDEwLjgzNDE0MV0sIFs0NS44NTQzNzMsIDEwLjgzNDIzXSwgWzQ1Ljg1NDUwMiwgMTAuODM0MzQxXSwgWzQ1Ljg1NDcyMywgMTAuODM0Mzk1XSwgWzQ1Ljg1NDg4OCwgMTAuODM0Mzg5XSwgWzQ1Ljg1NTEwMywgMTAuODM0NDEzXSwgWzQ1Ljg1NTM1NCwgMTAuODM0NDM4XSwgWzQ1Ljg1NTUxOSwgMTAuODM0NDY4XSwgWzQ1Ljg1NTY0NywgMTAuODM0NDE1XSwgWzQ1Ljg1NTc0NSwgMTAuODM0Mjk5XSwgWzQ1Ljg1NTkzNCwgMTAuODM0MTQzXSwgWzQ1Ljg1NjE3MiwgMTAuODM0MDk5XSwgWzQ1Ljg1NjM0MiwgMTAuODM0MDU2XSwgWzQ1Ljg1NjU5OSwgMTAuODM0MDFdLCBbNDUuODU2ODE5LCAxMC44MzM5NjJdLCBbNDUuODU3MDE5LCAxMC44MzM4NTJdLCBbNDUuODU3MTIzLCAxMC44MzM3NDZdLCBbNDUuODU3MjU0LCAxMC44MzM2MjRdLCBbNDUuODU3NDE1LCAxMC44MzM1NjZdLCBbNDUuODU3NjAxLCAxMC44MzM0OTRdLCBbNDUuODU3NzU2LCAxMC44MzMzODNdLCBbNDUuODU3ODI3LCAxMC44MzMyOTldLCBbNDUuODU3OTE5LCAxMC44MzMyNDZdLCBbNDUuODU4MTA3LCAxMC44MzMyMTRdLCBbNDUuODU4MzIxLCAxMC44MzMxMl0sIFs0NS44NTg3MDcsIDEwLjgzMjkyNV0sIFs0NS44NTg4ODQsIDEwLjgzMjg4N10sIFs0NS44NTkwMjQsIDEwLjgzMjgzNF0sIFs0NS44NTkyMjcsIDEwLjgzMjg3NF0sIFs0NS44NTkzODQsIDEwLjgzMjk2Nl0sIFs0NS44NTk3MTMsIDEwLjgzMzAwNl0sIFs0NS44NTk5NDIsIDEwLjgzMjk5NF0sIFs0NS44NjAzMjksIDEwLjgzMzE4Nl0sIFs0NS44NjA2MTEsIDEwLjgzMzEzXSwgWzQ1Ljg2MDc5NSwgMTAuODMzMDQ1XSwgWzQ1Ljg2MDkyOCwgMTAuODMyOTk2XSwgWzQ1Ljg2MTAyOCwgMTAuODMzMDExXSwgWzQ1Ljg2MTE1NiwgMTAuODMzMDM3XSwgWzQ1Ljg2MTI3NSwgMTAuODMzMDcyXSwgWzQ1Ljg2MTYyMywgMTAuODMzMjJdLCBbNDUuODYxNzczLCAxMC44MzMyOThdLCBbNDUuODYxOTAxLCAxMC44MzMzODNdLCBbNDUuODYyMDIzLCAxMC44MzM0ODFdLCBbNDUuODYyMjM5LCAxMC44MzM1OV0sIFs0NS44NjIzOTcsIDEwLjgzMzY1NV0sIFs0NS44NjI1OTUsIDEwLjgzMzc0NV0sIFs0NS44NjI5MTUsIDEwLjgzMzkxMl0sIFs0NS44NjMzOTMsIDEwLjgzNDE2OF0sIFs0NS44NjM1NjQsIDEwLjgzNDIzXSwgWzQ1Ljg2MzgzNSwgMTAuODM0MzMxXSwgWzQ1Ljg2NDA2NCwgMTAuODM0MzcyXSwgWzQ1Ljg2NDMwMywgMTAuODM0NDQ5XSwgWzQ1Ljg2NDQ3MSwgMTAuODM0NTI4XSwgWzQ1Ljg2NDY1NSwgMTAuODM0NjU2XSwgWzQ1Ljg2NDkwNiwgMTAuODM0NzE3XSwgWzQ1Ljg2NTI5MywgMTAuODM0NzA2XSwgWzQ1Ljg2NTU2MiwgMTAuODM0ODFdLCBbNDUuODY1NzA3LCAxMC44MzQ5MjhdLCBbNDUuODY1OTQ5LCAxMC44MzQ5ODVdLCBbNDUuODY2MDY1LCAxMC44MzUwMjRdLCBbNDUuODY2MTc5LCAxMC44MzUwNjJdLCBbNDUuODY2MzUsIDEwLjgzNTExOF0sIFs0NS44NjY4MDIsIDEwLjgzNTA0M10sIFs0NS44NjY5MzcsIDEwLjgzNTAwNl0sIFs0NS44NjcwNTgsIDEwLjgzNTAyNV0sIFs0NS44NjcxNjIsIDEwLjgzNTExNl0sIFs0NS44NjcyNzUsIDEwLjgzNTI0OV0sIFs0NS44NjczMTQsIDEwLjgzNTMxMl0sIFs0NS44Njc0NTcsIDEwLjgzNTM4N10sIFs0NS44Njc2MjIsIDEwLjgzNTMzOF0sIFs0NS44Njc3OTcsIDEwLjgzNTI5M10sIFs0NS44Njc5ODIsIDEwLjgzNTI2NF0sIFs0NS44NjgxMTMsIDEwLjgzNTIxNV0sIFs0NS44NjgyMzUsIDEwLjgzNTE0Ml0sIFs0NS44Njg0LCAxMC44MzUwOTNdLCBbNDUuODY4NjQ1LCAxMC44MzUwNTZdLCBbNDUuODY4ODUxLCAxMC44MzUwNDddLCBbNDUuODY5MDU1LCAxMC44MzUwNjFdLCBbNDUuODY5MTg4LCAxMC44MzUwMTVdLCBbNDUuODY5MzQzLCAxMC44MzUwMzJdLCBbNDUuODY5NDI3LCAxMC44MzUxMThdLCBbNDUuODY5NTQsIDEwLjgzNTIyOV0sIFs0NS44Njk2NDQsIDEwLjgzNTMzM10sIFs0NS44Njk4MjIsIDEwLjgzNTQyOF0sIFs0NS44Njk5MjYsIDEwLjgzNTUwNl0sIFs0NS44NzAxMzUsIDEwLjgzNTY3XSwgWzQ1Ljg3MDI3MywgMTAuODM1Nzk1XSwgWzQ1Ljg3MDQzMiwgMTAuODM1OTc4XSwgWzQ1Ljg3MDU3NCwgMTAuODM2MTg3XSwgWzQ1Ljg3MDY3LCAxMC44MzYzNjldLCBbNDUuODcwNzg3LCAxMC44MzY1ODVdLCBbNDUuODcwOTUzLCAxMC44MzY4NDRdLCBbNDUuODcxMDg5LCAxMC44MzcwNF0sIFs0NS44NzExNzEsIDEwLjgzNzI2MV0sIFs0NS44NzEyOTIsIDEwLjgzNzQ2Nl0sIFs0NS44NzE0OTIsIDEwLjgzNzYzNF0sIFs0NS44NzE2MDEsIDEwLjgzNzY3Nl0sIFs0NS44NzE3MzMsIDEwLjgzNzc1MV0sIFs0NS44NzE5MTEsIDEwLjgzNzc5N10sIFs0NS44NzIwMTksIDEwLjgzNzcyOF0sIFs0NS44NzIxNiwgMTAuODM3NjQ5XSwgWzQ1Ljg3MjMyLCAxMC44Mzc2MjNdLCBbNDUuODcyNDE2LCAxMC44Mzc2OF0sIFs0NS44NzI1NTgsIDEwLjgzNzgxOF0sIFs0NS44NzI3NDEsIDEwLjgzNzg5XSwgWzQ1Ljg3Mjk5NCwgMTAuODM3OTEyXSwgWzQ1Ljg3MzE1OCwgMTAuODM3OTc4XSwgWzQ1Ljg3MzI4NiwgMTAuODM4MDJdLCBbNDUuODczNTY3LCAxMC44MzgwNDZdLCBbNDUuODczODEyLCAxMC44MzgwNTddLCBbNDUuODczOTU3LCAxMC44MzgxOThdLCBbNDUuODc0MTI5LCAxMC44MzgzMTNdLCBbNDUuODc0MjQ2LCAxMC44MzgzMzJdLCBbNDUuODc0NDUxLCAxMC44MzgzNTNdLCBbNDUuODc0NjAyLCAxMC44MzgzODldLCBbNDUuODc0NzI0LCAxMC44MzgzMDldLCBbNDUuODc0ODI1LCAxMC44MzgyNDNdLCBbNDUuODc1MDAxLCAxMC44MzgyODldLCBbNDUuODc1MTM0LCAxMC44MzgyNDNdLCBbNDUuODc1Mjk5LCAxMC44MzgyMDRdLCBbNDUuODc1NDE2LCAxMC44MzgyNDJdLCBbNDUuODc1NTM4LCAxMC44MzgzMl0sIFs0NS44NzU2NTgsIDEwLjgzODQ4N10sIFs0NS44NzU4MDUsIDEwLjgzODYzNF0sIFs0NS44NzU5NjIsIDEwLjgzODcwM10sIFs0NS44NzYwNTIsIDEwLjgzODY3Ml0sIFs0NS44NzYxNjYsIDEwLjgzODcxN10sIFs0NS44NzYyOTMsIDEwLjgzODgwNV0sIFs0NS44NzY0OTUsIDEwLjgzODkzM10sIFs0NS44NzY2NjMsIDEwLjgzOTAzOV0sIFs0NS44NzY3NzMsIDEwLjgzOTA3N10sIFs0NS44NzY5MjMsIDEwLjgzOTE0Nl0sIFs0NS44NzcwODMsIDEwLjgzOTE3NV0sIFs0NS44NzcyNDksIDEwLjgzOTIxOF0sIFs0NS44NzczNTksIDEwLjgzOTI1XSwgWzQ1Ljg3NzUzOSwgMTAuODM5MjgzXSwgWzQ1Ljg3Nzg4NSwgMTAuODM5MjMyXSwgWzQ1Ljg3ODAxLCAxMC44MzkwOV0sIFs0NS44NzgxMjgsIDEwLjgzODk4OF0sIFs0NS44NzgyNzEsIDEwLjgzODg1NF0sIFs0NS44Nzg0MjMsIDEwLjgzODcxXSwgWzQ1Ljg3ODU0MiwgMTAuODM4NDkzXSwgWzQ1Ljg3ODY1MSwgMTAuODM4Mzg3XSwgWzQ1Ljg3ODg5NywgMTAuODM4Mjk0XSwgWzQ1Ljg3OTE5LCAxMC44MzgyMDldLCBbNDUuODc5NDA2LCAxMC44MzgxNDFdLCBbNDUuODc5Njk0LCAxMC44MzgxNTFdLCBbNDUuODc5OTY5LCAxMC44MzgxMzRdLCBbNDUuODgwMjA0LCAxMC44MzgxNDldLCBbNDUuODgwNDMzLCAxMC44MzgxODNdLCBbNDUuODgwNjU0LCAxMC44MzgyMjddLCBbNDUuODgwOTEzLCAxMC44MzgyMzZdLCBbNDUuODgxMDkxLCAxMC44MzgyNTZdLCBbNDUuODgxNDQxLCAxMC44MzgyNzddLCBbNDUuODgyNDM4LCAxMC44Mzg1MzFdLCBbNDUuODgyNDgyLCAxMC44Mzg0NjZdLCBbNDUuODgyOTY1LCAxMC44Mzg2NV0sIFs0NS44ODMwMDYsIDEwLjgzODgyMV0sIFs0NS44ODMwNiwgMTAuODM4OTE3XSwgWzQ1Ljg4MzEyNiwgMTAuODM4OTYxXSwgWzQ1Ljg4MzM3NSwgMTAuODM4OTkzXSwgWzQ1Ljg4MzQ0MywgMTAuODM5MDkyXSwgWzQ1Ljg4MzU0LCAxMC44MzkyMTldLCBbNDUuODgzNjg2LCAxMC44MzkyODhdLCBbNDUuODgzOTY1LCAxMC44Mzk0ODddLCBbNDUuODg0NDE0LCAxMC44MzkzOTVdLCBbNDUuODg0Mzk0LCAxMC44MzkxMTNdLCBbNDUuODg0NiwgMTAuODM5MDg4XSwgWzQ1Ljg4NDYxNiwgMTAuODM5ODA5XSwgWzQ1Ljg4NDI0LCAxMC44Mzk4NTZdLCBbNDUuODg0MDc0LCAxMC44NDAxOTNdLCBbNDUuODgzOTgzLCAxMC44NDAyMDFdLCBbNDUuODgzOTM1LCAxMC44NDA0MTldLCBbNDUuODgzOTkxLCAxMC44NDA0NjddLCBbNDUuODgzOTM3LCAxMC44NDA3OTldLCBbNDUuODgzOTg1LCAxMC44NDA4MzNdLCBbNDUuODgzOTQxLCAxMC44NDExMzddLCBbNDUuODgzODU0LCAxMC44NDExNDhdLCBbNDUuODgzODU1LCAxMC44NDA3OThdLCBbNDUuODgzNzk4LCAxMC44NDA3OV0sIFs0NS44ODM3ODIsIDEwLjg0MTIzOF0sIFs0NS44ODM4NTUsIDEwLjg0MTI2Nl0sIFs0NS44ODM4NTMsIDEwLjg0MTY1Nl0sIFs0NS44ODQ0MTUsIDEwLjg0MTc3Nl0sIFs0NS44ODQ0MjQsIDEwLjg0MTk3Nl0sIFs0NS44ODQwNiwgMTAuODQxOTYxXSwgWzQ1Ljg4MzkyMSwgMTAuODQxOTAzXSwgWzQ1Ljg4Mzc4MiwgMTAuODQxODUxXSwgWzQ1Ljg4MzYzOSwgMTAuODQxNzk4XSwgWzQ1Ljg4MzQ2OSwgMTAuODQyNTI1XSwgWzQ1Ljg4MzQ3NCwgMTAuODQyNzM4XSwgWzQ1Ljg4NDY0MywgMTAuODQyNzE3XSwgWzQ1Ljg4NDY0OCwgMTAuODQxOTc0XSwgWzQ1Ljg4NDU3LCAxMC44NDE5ODJdLCBbNDUuODg0NTY3LCAxMC44NDE4NThdLCBbNDUuODg0NzQxLCAxMC44NDE4NjVdLCBbNDUuODg0OCwgMTAuODQzMDA5XSwgWzQ1Ljg4MzIyOCwgMTAuODQzMDddLCBbNDUuODgzMjQxLCAxMC44NDMxMzZdLCBbNDUuODgzNTIsIDEwLjg0MzExNV0sIFs0NS44ODM1MDgsIDEwLjg0MzE5XSwgWzQ1Ljg4MzU3NiwgMTAuODQzMjI1XSwgWzQ1Ljg4MzY0MiwgMTAuODQzMzAxXSwgWzQ1Ljg4MzY2OCwgMTAuODQzNDEzXSwgWzQ1Ljg4MzY5NCwgMTAuODQzNTI5XSwgWzQ1Ljg4MzY2OCwgMTAuODQzODA2XSwgWzQ1Ljg4MzU2OSwgMTAuODQzOTQ4XSwgWzQ1Ljg4MzUwNSwgMTAuODQzOTQ0XSwgWzQ1Ljg4MzQyMiwgMTAuODQzOTY1XSwgWzQ1Ljg4MzM4NywgMTAuODQ0MDEzXSwgWzQ1Ljg4MzM3MywgMTAuODQ0MDk4XSwgWzQ1Ljg4MzM0MSwgMTAuODQ0MjU0XSwgWzQ1Ljg4MzI3MywgMTAuODQ0MjZdLCBbNDUuODgzMjc0LCAxMC44NDQzNjRdLCBbNDUuODgzMjAyLCAxMC44NDQ0MzJdLCBbNDUuODgzMDQ4LCAxMC44NDQyNjFdLCBbNDUuODgyODY4LCAxMC44NDQwMDVdLCBbNDUuODgyODk1LCAxMC44NDMyXSwgWzQ1Ljg4Mjc5MiwgMTAuODQzMjA4XSwgWzQ1Ljg4MjgxOSwgMTAuODQzOTE2XSwgWzQ1Ljg4Mjc2NSwgMTAuODQzOTc3XSwgWzQ1Ljg4MzE1NiwgMTAuODQ0NTA5XSwgWzQ1Ljg4MzE0MywgMTAuODQ0NjQ2XSwgWzQ1Ljg4MzIyMiwgMTAuODQ0NjkxXSwgWzQ1Ljg4Mjc4OCwgMTAuODQ1NzUyXSwgWzQ1Ljg4MjY4MywgMTAuODQ1NzZdLCBbNDUuODgyNTEyLCAxMC44NDU2NDhdLCBbNDUuODgyMzcsIDEwLjg0NTY2OF0sIFs0NS44ODIyNDksIDEwLjg0NTY3Ml0sIFs0NS44ODIwMzQsIDEwLjg0NTQ2N10sIFs0NS44ODE5MiwgMTAuODQ1Mzk5XSwgWzQ1Ljg4MTgxMywgMTAuODQ1Mzc3XSwgWzQ1Ljg4MTYzNSwgMTAuODQ1MjkyXSwgWzQ1Ljg4MTQ2NCwgMTAuODQ1MzE4XSwgWzQ1Ljg4MTMyOCwgMTAuODQ1MzQ0XSwgWzQ1Ljg4MTIzLCAxMC44NDUzNjJdLCBbNDUuODgxMTQ1LCAxMC44NDUzOTldLCBbNDUuODgxMSwgMTAuODQ1NDg2XSwgWzQ1Ljg4MTExNywgMTAuODQ1NjA1XSwgWzQ1Ljg4MTExMywgMTAuODQ1Nzc3XSwgWzQ1Ljg4MTExMywgMTAuODQ1Nzg1XSwgWzQ1Ljg4MTI1NywgMTAuODQ2MDczXSwgWzQ1Ljg4MTM5MiwgMTAuODQ2Mjc5XSwgWzQ1Ljg4MTQ1NCwgMTAuODQ2NDU3XSwgWzQ1Ljg4MTU0MSwgMTAuODQ2ODgxXSwgWzQ1Ljg4MTUxNiwgMTAuODQ3MDQ0XSwgWzQ1Ljg4MTUwNSwgMTAuODQ3Mjk2XSwgWzQ1Ljg4MTQyNCwgMTAuODQ3NTY2XSwgWzQ1Ljg4MTI4NywgMTAuODQ4NDA4XSwgWzQ1Ljg4MTI0NiwgMTAuODQ4NjI3XSwgWzQ1Ljg4MTIzNSwgMTAuODQ4ODE2XSwgWzQ1Ljg4MTE4NCwgMTAuODQ5MDgxXSwgWzQ1Ljg4MDk3MSwgMTAuODQ5MjkyXSwgWzQ1Ljg4MDgxMywgMTAuODQ5MzE4XSwgWzQ1Ljg4MDcwMSwgMTAuODQ5Mzg0XSwgWzQ1Ljg4MDU3MSwgMTAuODQ5NDk2XSwgWzQ1Ljg4MDUzMiwgMTAuODQ5NTY1XSwgWzQ1Ljg4MDQ3NiwgMTAuODQ5NjY0XSwgWzQ1Ljg4MDUyLCAxMC44NDk4MzZdLCBbNDUuODgwNTA0LCAxMC44NDk5ODldLCBbNDUuODgwMjg3LCAxMC44NTAwNDNdLCBbNDUuODgwMTg5LCAxMC44NTAxOTJdLCBbNDUuODgwMDcxLCAxMC44NTAzNF0sIFs0NS44Nzk5OCwgMTAuODUwNTAyXSwgWzQ1Ljg3OTkzNCwgMTAuODUwNzJdLCBbNDUuODc5ODc1LCAxMC44NTA4ODNdLCBbNDUuODc5ODExLCAxMC44NTExMDRdLCBbNDUuODc5NzQyLCAxMC44NTEzOTFdLCBbNDUuODc5ODg4LCAxMC44NTE0XSwgWzQ1Ljg3OTkyOSwgMTAuODUxNDZdLCBbNDUuODc5OTA3LCAxMC44NTE1OTFdLCBbNDUuODc5ODg1LCAxMC44NTE2OThdLCBbNDUuODc5NjY1LCAxMC44NTIxNTVdLCBbNDUuODc5NjA4LCAxMC44NTIzODNdLCBbNDUuODc5NTg5LCAxMC44NTI0MzhdLCBbNDUuODc5NTM2LCAxMC44NTIyMjRdLCBbNDUuODc5NTgyLCAxMC44NTE5NzZdLCBbNDUuODc5NTUsIDEwLjg1MTc1XSwgWzQ1Ljg3OTUxLCAxMC44NTE2NDFdLCBbNDUuODc5NDI1LCAxMC44NTE3NTddLCBbNDUuODc5NDI0LCAxMC44NTIwMDldLCBbNDUuODc5Mzk5LCAxMC44NTIyMThdLCBbNDUuODc5Mzc1LCAxMC44NTI1MTJdLCBbNDUuODc5Mjk4LCAxMC44NTI2OTFdLCBbNDUuODc5MjMzLCAxMC44NTI4MzNdLCBbNDUuODc5MTM3LCAxMC44NTMwMTFdLCBbNDUuODc5MDExLCAxMC44NTM2MzFdLCBbNDUuODc4OTE0LCAxMC44NTM5MzZdLCBbNDUuODc4Nzk4LCAxMC44NTQzNl0sIFs0NS44Nzg3NDMsIDEwLjg1NDYxMV0sIFs0NS44Nzg3MDMsIDEwLjg1NDk0XSwgWzQ1Ljg3ODYyNywgMTAuODU1MTY4XSwgWzQ1Ljg3ODU2OCwgMTAuODU1MzI0XSwgWzQ1Ljg3ODQ0OCwgMTAuODU1NDY1XSwgWzQ1Ljg3ODM0NCwgMTAuODU1NTEyXSwgWzQ1Ljg3ODIyMywgMTAuODU1NTU5XSwgWzQ1Ljg3ODE5OCwgMTAuODU1NzA1XSwgWzQ1Ljg3ODMwNSwgMTAuODU1NzZdLCBbNDUuODc4Mzk0LCAxMC44NTU3NTJdLCBbNDUuODc4NDI4LCAxMC44NTU3ODNdLCBbNDUuODc4NTg4LCAxMC44NTY1XSwgWzQ1Ljg3NzczNCwgMTAuODU3MTkyXSwgWzQ1Ljg3NzQ1MywgMTAuODU2NzExXSwgWzQ1Ljg3NzkyMywgMTAuODU1NTU5XSwgWzQ1Ljg3Nzg2OCwgMTAuODU1NTI1XSwgWzQ1Ljg3NzUyOCwgMTAuODU2MzMzXSwgWzQ1Ljg3NzMyNiwgMTAuODU2Mzk3XSwgWzQ1Ljg3NzE1NywgMTAuODU2NTddLCBbNDUuODc2OTkxLCAxMC44NTY3NzldLCBbNDUuODc2ODczLCAxMC44NTY5MDRdLCBbNDUuODc2Nzc2LCAxMC44NTczNTFdLCBbNDUuODc2NjY0LCAxMC44NTc2MzddLCBbNDUuODc2NjIzLCAxMC44NTc4MzVdLCBbNDUuODc2NjA1LCAxMC44NTgwMTJdLCBbNDUuODc2NTM0LCAxMC44NTgyMDRdLCBbNDUuODc2MzY3LCAxMC44NTg0MDldLCBbNDUuODc2MjY1LCAxMC44NTg3MzFdLCBbNDUuODc2MzEyLCAxMC44NTkxMjJdLCBbNDUuODc2Mzc4LCAxMC44NTk1NjldLCBbNDUuODc2NjUyLCAxMC44NjA5NDddLCBbNDUuODc2NjI2LCAxMC44NjEyMzFdLCBbNDUuODc2NTczLCAxMC44NjE0ODJdLCBbNDUuODc2NTY4LCAxMC44NjE3MzFdLCBbNDUuODc2NjUxLCAxMC44NjIwOTZdLCBbNDUuODc2NywgMTAuODYyNDZdLCBbNDUuODc2NjgxLCAxMC44NjI3MjldLCBbNDUuODc2NzIyLCAxMC44NjI5NTJdLCBbNDUuODc2Nzk2LCAxMC44NjMzMDFdLCBbNDUuODc2ODU5LCAxMC44NjM1ODRdLCBbNDUuODc3MDA2LCAxMC44NjQwMzJdLCBbNDUuODc3MDM1LCAxMC44NjQyOTJdLCBbNDUuODc3MDY0LCAxMC44NjQ1NDhdLCBbNDUuODc3MDQ0LCAxMC44NjQ3NTddLCBbNDUuODc3LCAxMC44NjQ5NjJdLCBbNDUuODc2NjQ5LCAxMC44NjU3M10sIFs0NS44NzY0NjUsIDEwLjg2NjAxOF0sIFs0NS44NzYzMTcsIDEwLjg2NjIxNF0sIFs0NS44NzYwOTgsIDEwLjg2NjM0NF0sIFs0NS44NzU5NTMsIDEwLjg2NjQ0Ml0sIFs0NS44NzU3NzUsIDEwLjg2NjY0MV0sIFs0NS44NzU1MzUsIDEwLjg2NjgxOV0sIFs0NS44NzUzMzcsIDEwLjg2Njk2OV0sIFs0NS44NzUxODMsIDEwLjg2Njk5NV0sIFs0NS44NzUwMDMsIDEwLjg2NjkzMl0sIFs0NS44NzQ4MDgsIDEwLjg2NjgxM10sIFs0NS44NzQ1MjEsIDEwLjg2Njg2Ml0sIFs0NS44NzQzODIsIDEwLjg2NjgzM10sIFs0NS44NzQyNDksIDEwLjg2NjYwNF0sIFs0NS44NzQxNDIsIDEwLjg2NjM4M10sIFs0NS44NzQwNDMsIDEwLjg2NjI2OV0sIFs0NS44NzM5NTYsIDEwLjg2NjI0OF0sIFs0NS44NzM4ODYsIDEwLjg2NjExNV0sIFs0NS44NzM5MTEsIDEwLjg2NTk1NV0sIFs0NS44NzQwMjcsIDEwLjg2NTc5N10sIFs0NS44NzM5MDUsIDEwLjg2NTYyNF0sIFs0NS44NzM3NiwgMTAuODY1NzM2XSwgWzQ1Ljg3MzY2NywgMTAuODY1OTE3XSwgWzQ1Ljg3MzU2MywgMTAuODY2MjE5XSwgWzQ1Ljg3MzQ5NCwgMTAuODY2NDU3XSwgWzQ1Ljg3MzQ2OCwgMTAuODY2NTA5XSwgWzQ1Ljg3MzQzNSwgMTAuODY2NTE1XSwgWzQ1Ljg3MzAxOSwgMTAuODY2NTk1XSwgWzQ1Ljg3MjY4OSwgMTAuODY2NjU4XSwgWzQ1Ljg3MjY2NywgMTAuODY2NjU3XSwgWzQ1Ljg3MjU2NSwgMTAuODY2NjUyXSwgWzQ1Ljg3MjQ1OCwgMTAuODY2NjI3XSwgWzQ1Ljg3MjM4NywgMTAuODY2NjE5XSwgWzQ1Ljg3MjQyOSwgMTAuODY2OTUxXSwgWzQ1Ljg3MjUyOSwgMTAuODY3MDg0XSwgWzQ1Ljg3MjU2NywgMTAuODY3MTU3XSwgWzQ1Ljg3MjU1NywgMTAuODY3MjU4XSwgWzQ1Ljg3MjQyOCwgMTAuODY3MjgxXSwgWzQ1Ljg3MjM3LCAxMC44NjczODVdLCBbNDUuODcyMzk4LCAxMC44Njc1MjZdLCBbNDUuODcyNDY2LCAxMC44Njc3OTZdLCBbNDUuODcyNDg1LCAxMC44NjgxODZdLCBbNDUuODcyNDA2LCAxMC44Njg1NDFdLCBbNDUuODcyMzQzLCAxMC44Njg4MjVdLCBbNDUuODcyMjQ0LCAxMC44NjkwODVdLCBbNDUuODcyMTI4LCAxMC44Njk0NzVdLCBbNDUuODcxOTUxLCAxMC44Njk3ODVdLCBbNDUuODcxNjgsIDEwLjg3MDEwN10sIFs0NS44NzE0MjEsIDEwLjg3MDM1XSwgWzQ1Ljg3MTEzOCwgMTAuODcwNzE3XSwgWzQ1Ljg3MDkyMywgMTAuODcwOTU0XSwgWzQ1Ljg3MDc5NCwgMTAuODcxMDY2XSwgWzQ1Ljg3MDczNSwgMTAuODcxMjA2XSwgWzQ1Ljg3MDk2NCwgMTAuODcxNjA3XSwgWzQ1Ljg3MDg4OSwgMTAuODcxNzc5XSwgWzQ1Ljg3MDc5MiwgMTAuODcxNjc4XSwgWzQ1Ljg3MDY1MywgMTAuODcxNjQ2XSwgWzQ1Ljg3MDYxMSwgMTAuODcxMDI2XSwgWzQ1Ljg3MDU0NCwgMTAuODcxMDU3XSwgWzQ1Ljg3MDU3NywgMTAuODcxNjI1XSwgWzQ1Ljg3MDI1MywgMTAuODcyMjIzXSwgWzQ1Ljg3MDE3MywgMTAuODcyMTk1XSwgWzQ1Ljg3MDEzMSwgMTAuODcyMjc2XSwgWzQ1Ljg3MDE4MywgMTAuODcyMzQ2XSwgWzQ1Ljg2OTk2NywgMTAuODcyNjY5XSwgWzQ1Ljg2OTg5NCwgMTAuODcyNjQ3XSwgWzQ1Ljg2OTgxMiwgMTAuODcyODI2XSwgWzQ1Ljg2OTgzMiwgMTAuODcyODU1XSwgWzQ1Ljg2OTE0MSwgMTAuODc0MTk5XSwgWzQ1Ljg2OTA3NSwgMTAuODc0MTUyXSwgWzQ1Ljg2OTAxNiwgMTAuODc0MzExXSwgWzQ1Ljg2ODkxNiwgMTAuODc0MjY2XSwgWzQ1Ljg2ODgyNiwgMTAuODc0NTE2XSwgWzQ1Ljg2ODg3NCwgMTAuODc0NTM3XSwgWzQ1Ljg2ODg4MiwgMTAuODc0Njc4XSwgWzQ1Ljg2ODc4MSwgMTAuODc0OTI0XSwgWzQ1Ljg2ODU5LCAxMC44NzUzOThdLCBbNDUuODY4NzE4LCAxMC44NzU4NjldLCBbNDUuODY4NjU4LCAxMC44NzYxMDddLCBbNDUuODY4NjE2LCAxMC44NzYyMzNdLCBbNDUuODY4NTY5LCAxMC44NzYyOThdLCBbNDUuODY4NTM0LCAxMC44NzYzNzldLCBbNDUuODY4NjUxLCAxMC44NzYzOTVdLCBbNDUuODY4NzA3LCAxMC44NzYzMDFdLCBbNDUuODY4NzYyLCAxMC44NzYyMjRdLCBbNDUuODY4OTA2LCAxMC44NzYwNDNdLCBbNDUuODY5MDgxLCAxMC44NzYxNjJdLCBbNDUuODY4NTE2LCAxMC44NzY2MTFdLCBbNDUuODY4NDk2LCAxMC44NzY1MDZdLCBbNDUuODY4NDc2LCAxMC44NzY0NjZdLCBbNDUuODY4NDM3LCAxMC44NzY0NjJdLCBbNDUuODY4MzUzLCAxMC44NzY1OThdLCBbNDUuODY4MzI2LCAxMC44NzY2MDddLCBbNDUuODY4MjEzLCAxMC44NzY2OTZdLCBbNDUuODY4MTE5LCAxMC44NzY3NF0sIFs0NS44Njc5NzgsIDEwLjg3Njc5OV0sIFs0NS44Njc4MjcsIDEwLjg3NjgzNV0sIFs0NS44Njc2NzEsIDEwLjg3Njg4NF0sIFs0NS44Njc1NzMsIDEwLjg3Njg4NV0sIFs0NS44Njc0MzEsIDEwLjg3Njg4OF0sIFs0NS44NjczMzIsIDEwLjg3Njg4Nl0sIFs0NS44NjcyMjUsIDEwLjg3Njg4XSwgWzQ1Ljg2NjQ5NSwgMTAuODc2NDY4XSwgWzQ1Ljg2NjE3NiwgMTAuODc2MTZdLCBbNDUuODY1ODAzLCAxMC44NzU5MTZdLCBbNDUuODY1NTIsIDEwLjg3NTcxXSwgWzQ1Ljg2NTMwNCwgMTAuODc1NTM4XSwgWzQ1Ljg2NTEzNiwgMTAuODc1NDJdLCBbNDUuODY1LCAxMC44NzUzMzJdLCBbNDUuODY0ODk1LCAxMC44NzUyODRdLCBbNDUuODY0NzIsIDEwLjg3NTIyN10sIFs0NS44NjQ1NTcsIDEwLjg3NTIxMV0sIFs0NS44NjQzNDksIDEwLjg3NTIyMl0sIFs0NS44NjQxNzksIDEwLjg3NTI3NF0sIFs0NS44NjQwMzYsIDEwLjg3NTM3M10sIFs0NS44NjM5LCAxMC44NzU0OTRdLCBbNDUuODYzNzY2LCAxMC44NzU2NThdLCBbNDUuODYzNjI3LCAxMC44NzU3NjZdLCBbNDUuODYzNTM1LCAxMC44NzU4NTNdLCBbNDUuODYzMDM2LCAxMC44NzYxMTNdLCBbNDUuODYyOTA1LCAxMC44NzYxM10sIFs0NS44NjI4MTYsIDEwLjg3NjE2N10sIFs0NS44NjI3MTksIDEwLjg3NjE2NV0sIFs0NS44NjI1NzcsIDEwLjg3NTQwNl0sIFs0NS44NjI2NzUsIDEwLjg3NTQyMV0sIFs0NS44NjI2NjQsIDEwLjg3NTM1OV0sIFs0NS44NjI1OTYsIDEwLjg3NTI4NV0sIFs0NS44NjI0NDQsIDEwLjg3NTIyM10sIFs0NS44NjIzNzYsIDEwLjg3NTE1Nl0sIFs0NS44NjIxNzksIDEwLjg3NTEyNV0sIFs0NS44NjE5NzksIDEwLjg3NTE5OV0sIFs0NS44NjE4OTEsIDEwLjg3NTM2NF0sIFs0NS44NjE3OTMsIDEwLjg3NTUyNl0sIFs0NS44NjE1OTgsIDEwLjg3NTU4N10sIFs0NS44NjE0ODYsIDEwLjg3NTU5OF0sIFs0NS44NjEzNDEsIDEwLjg3NTY0NF0sIFs0NS44NjEyNDQsIDEwLjg3NTcyM10sIFs0NS44NjExNzMsIDEwLjg3NTc3MV0sIFs0NS44NjEwODgsIDEwLjg3NTc1M10sIFs0NS44NjA3ODQsIDEwLjg3NTI4NF0sIFs0NS44NjA2NjcsIDEwLjg3NDg5OV0sIFs0NS44NjA1MDksIDEwLjg3NDcyMl0sIFs0NS44NjAzNTEsIDEwLjg3NDUxNV0sIFs0NS44NjAxNDgsIDEwLjg3NDI1Nl0sIFs0NS44NTk5NjcsIDEwLjg3NDA1Ml0sIFs0NS44NTk4MDUsIDEwLjg3Mzg1Ml0sIFs0NS44NTk2MjcsIDEwLjg3MzU4Nl0sIFs0NS44NTk0NDksIDEwLjg3MzM1XSwgWzQ1Ljg1OTI1OCwgMTAuODcyOTkyXSwgWzQ1Ljg1OTAzNSwgMTAuODcyNjc5XSwgWzQ1Ljg1ODc2OSwgMTAuODcyMzFdLCBbNDUuODU4NTgsIDEwLjg3MjAxMV0sIFs0NS44NTgzMDgsIDEwLjg3MTU3M10sIFs0NS44NTgwODMsIDEwLjg3MTM5OF0sIFs0NS44NTc4MDUsIDEwLjg3MTA3NV0sIFs0NS44NTc1MDEsIDEwLjg3MDgyOV0sIFs0NS44NTcxOTQsIDEwLjg3MDY0OV0sIFs0NS44NTY5NjcsIDEwLjg3MDVdLCBbNDUuODU2NzUyLCAxMC44NzAzNTFdLCBbNDUuODU2NjM2LCAxMC44NzAyNV0sIFs0NS44NTYzNTUsIDEwLjg2OTk2OV0sIFs0NS44NTYxMjUsIDEwLjg2OTY3Nl0sIFs0NS44NTU4OTQsIDEwLjg2OTQ2NV0sIFs0NS44NTU2NCwgMTAuODY5MjY5XSwgWzQ1Ljg1NTQ1NSwgMTAuODY5MDQ5XSwgWzQ1Ljg1NTMwOCwgMTAuODY4ODU2XSwgWzQ1Ljg1NTAxOSwgMTAuODY4NTI5XSwgWzQ1Ljg1NDc2MywgMTAuODY4MjYyXSwgWzQ1Ljg1NDU2OCwgMTAuODY4MDc3XSwgWzQ1Ljg1NDM5NiwgMTAuODY3OTMzXSwgWzQ1Ljg1NDIyOSwgMTAuODY3NzI2XSwgWzQ1Ljg1NDA4MSwgMTAuODY3NDc3XSwgWzQ1Ljg1Mzg3NSwgMTAuODY3MjM0XSwgWzQ1Ljg1MzYyMSwgMTAuODY2OTk5XSwgWzQ1Ljg1MzM2LCAxMC44NjY4MTddLCBbNDUuODUzMTI5LCAxMC44NjY1ODldLCBbNDUuODUyOTYzLCAxMC44NjYzMTddLCBbNDUuODUyODUxLCAxMC44NjYwNDNdLCBbNDUuODUyNzEsIDEwLjg2NTU3OV0sIFs0NS44NTI1NTksIDEwLjg2NTM0Nl0sIFs0NS44NTI0MzEsIDEwLjg2NTE3XSwgWzQ1Ljg1MjMyMywgMTAuODY0OTkxXSwgWzQ1Ljg1MjE1MywgMTAuODY0NzY4XSwgWzQ1Ljg1MTg0MiwgMTAuODY0Mzk0XSwgWzQ1Ljg1MTU3MiwgMTAuODY0MTQzXSwgWzQ1Ljg1MTI3MSwgMTAuODYzODQyXSwgWzQ1Ljg1MTA1NiwgMTAuODYzNjZdLCBbNDUuODUwODcyLCAxMC44NjM0NzZdLCBbNDUuODUwNzIzLCAxMC44NjMzNTldLCBbNDUuODUwNTQ4LCAxMC44NjMyNDddLCBbNDUuODUwNDQxLCAxMC44NjMxNzZdLCBbNDUuODUwMTg2LCAxMC44NjMwMjNdLCBbNDUuODQ5ODY1LCAxMC44NjI4NzVdLCBbNDUuODQ5Njk5LCAxMC44NjI3ODZdLCBbNDUuODQ5NTcsIDEwLjg2MjcwMl0sIFs0NS44NDk0MzQsIDEwLjg2MjU4MV0sIFs0NS44NDkyNjksIDEwLjg2MjM3OF0sIFs0NS44NDkyMTUsIDEwLjg2MjMxNF0sIFs0NS44NDkxODYsIDEwLjg2MjI5OF0sIFs0NS44NDkxMDcsIDEwLjg2MjE3Nl0sIFs0NS44NDkxMDcsIDEwLjg2MjE4MV0sIFs0NS44NDkwOTMsIDEwLjg2MjE3MV0sIFs0NS44NDkwNTcsIDEwLjg2MjE0OF0sIFs0NS44NDkwMiwgMTAuODYyMTU4XSwgWzQ1Ljg0OTAxNSwgMTAuODYyMjY1XSwgWzQ1Ljg0OTAwNCwgMTAuODYyMzMxXSwgWzQ1Ljg0ODk3MywgMTAuODYyNDEyXSwgWzQ1Ljg0ODk0OCwgMTAuODYyNDMxXSwgWzQ1Ljg0ODg4NywgMTAuODYyNDIxXSwgWzQ1Ljg0ODg1LCAxMC44NjI0M10sIFs0NS44NDg4MjcsIDEwLjg2MjQ2OV0sIFs0NS44NDg4MTMsIDEwLjg2MjU2N10sIFs0NS44NDg4MDUsIDEwLjg2MjY2Ml0sIFs0NS44NDg2ODksIDEwLjg2Mjk4OF0sIFs0NS44NDg2MjUsIDEwLjg2MzEwMl0sIFs0NS44NDg1NDIsIDEwLjg2MzIzMl0sIFs0NS44NDg0NzUsIDEwLjg2MzMxM10sIFs0NS44NDg0MDQsIDEwLjg2MzM3NF0sIFs0NS44NDgzMTcsIDEwLjg2MzQzMl0sIFs0NS44NDgyMjMsIDEwLjg2MzQ5M10sIFs0NS44NDgxNSwgMTAuODYzNTQxXSwgWzQ1Ljg0ODA2MywgMTAuODYzNTg2XSwgWzQ1Ljg0ODAwMywgMTAuODYzNjA1XSwgWzQ1Ljg0Nzk0OCwgMTAuODYzNjE0XSwgWzQ1Ljg0Nzg2MiwgMTAuODYzNTcxXSwgWzQ1Ljg0Nzc5MSwgMTAuODYzNTExXSwgWzQ1Ljg0NzczLCAxMC44NjM0MzVdLCBbNDUuODQ3NjU3LCAxMC44NjMzNDNdLCBbNDUuODQ3NTk4LCAxMC44NjMyNzddLCBbNDUuODQ3NTE4LCAxMC44NjMyMTddLCBbNDUuODQ3NDQ4LCAxMC44NjMyMDNdLCBbNDUuODQ3Mzc1LCAxMC44NjMyMDldLCBbNDUuODQ3MjgxLCAxMC44NjMyMTFdLCBbNDUuODQ3MTU1LCAxMC44NjMyMTZdLCBbNDUuODQ2NzQ0LCAxMC44NjMxNDZdLCBbNDUuODQ2NjYyLCAxMC44NjMwOTldLCBbNDUuODQ2NTU4LCAxMC44NjMwMzldLCBbNDUuODQ2NDU1LCAxMC44NjI5NzNdLCBbNDUuODQ2Mzg1LCAxMC44NjI5MjNdLCBbNDUuODQ2MjY2LCAxMC44NjI4NDNdLCBbNDUuODQ2MTU1LCAxMC44NjI3NTNdLCBbNDUuODQ2MDM5LCAxMC44NjI2NTddLCBbNDUuODQ1OTYxLCAxMC44NjI1ODhdLCBbNDUuODQ1NzgyLCAxMC44NjI0MzJdLCBbNDUuODQ1NzEzLCAxMC44NjI0MTVdLCBbNDUuODQ1NjM4LCAxMC44NjI0M10sIFs0NS44NDU1NTMsIDEwLjg2MjQ2Ml0sIFs0NS44NDU0NjYsIDEwLjg2MjQ5N10sIFs0NS44NDUzNTIsIDEwLjg2MjUzNV0sIFs0NS44NDUyMSwgMTAuODYyNTg2XSwgWzQ1Ljg0NTE1OSwgMTAuODYyNjE1XSwgWzQ1Ljg0NTEyNywgMTAuODYyNjQ0XSwgWzQ1Ljg0NTA5LCAxMC44NjI3MDJdLCBbNDUuODQ1MDc5LCAxMC44NjI3NTRdLCBbNDUuODQ1MDc0LCAxMC44NjI4M10sIFs0NS44NDUwNzEsIDEwLjg2Mjg5OF0sIFs0NS44NDUwNjgsIDEwLjg2Mjk4M10sIFs0NS44NDUwNjMsIDEwLjg2MzA2NV0sIFs0NS44NDUwNjUsIDEwLjg2MzE1XSwgWzQ1Ljg0NTA2MywgMTAuODYzMjEyXSwgWzQ1Ljg0NTA0OSwgMTAuODYzMjU4XSwgWzQ1Ljg0NTAxLCAxMC44NjMzMTZdLCBbNDUuODQ0OTY0LCAxMC44NjMzNDVdLCBbNDUuODQ0ODY1LCAxMC44NjMzNzNdLCBbNDUuODQ0NzQ0LCAxMC44NjM0MTVdLCBbNDUuODQ0NTc3LCAxMC44NjM0NzVdLCBbNDUuODQ0NDkyLCAxMC44NjM0OTNdLCBbNDUuODQ0Mzc2LCAxMC44NjM0NTldLCBbNDUuODQ0MjU1LCAxMC44NjMzODZdLCBbNDUuODQ0MTI2LCAxMC44NjMyNjddLCBbNDUuODQ0MDA4LCAxMC44NjMxNDhdLCBbNDUuODQzODkyLCAxMC44NjMwMjJdLCBbNDUuODQzNzY3LCAxMC44NjI4ODRdLCBbNDUuODQzNjMzLCAxMC44NjI3NDhdLCBbNDUuODQzNTM1LCAxMC44NjI2MzldLCBbNDUuODQzNDQsIDEwLjg2MjU1Nl0sIFs0NS44NDMzMzMsIDEwLjg2MjQ5XSwgWzQ1Ljg0MzIyNSwgMTAuODYyNDU5XSwgWzQ1Ljg0MzEyLCAxMC44NjI0NTRdLCBbNDUuODQzMDA2LCAxMC44NjI0Nl0sIFs0NS44NDI5MTksIDEwLjg2MjQ2OV0sIFs0NS44NDI3OTgsIDEwLjg2MjQ1NF0sIFs0NS44NDI3MDMsIDEwLjg2MjM4MV0sIFs0NS44NDI1NjYsIDEwLjg2MjMwMV0sIFs0NS44NDI0MzQsIDEwLjg2MjIxOF0sIFs0NS44NDIzNTIsIDEwLjg2MjE2MV0sIFs0NS44NDIxODgsIDEwLjg2MjA1Ml0sIFs0NS44NDIwNDksIDEwLjg2MTkzNl0sIFs0NS44NDE5MTcsIDEwLjg2MTgxM10sIFs0NS44NDE3OTcsIDEwLjg2MTcwNF0sIFs0NS44NDE2OTIsIDEwLjg2MTYxNF0sIFs0NS44NDE2MDEsIDEwLjg2MTU2MV0sIFs0NS44NDE0NzQsIDEwLjg2MTQ5NF0sIFs0NS44NDEzNDgsIDEwLjg2MTQyNF0sIFs0NS44NDEyNDgsIDEwLjg2MTM2MV0sIFs0NS44NDExNTMsIDEwLjg2MTI4MV0sIFs0NS44NDEwNjIsIDEwLjg2MTE4OV0sIFs0NS44NDA5OCwgMTAuODYxMTE2XSwgWzQ1Ljg0MDg3MSwgMTAuODYxMDJdLCBbNDUuODQwNzYxLCAxMC44NjA5MzNdLCBbNDUuODQwNjIzLCAxMC44NjA4NDRdLCBbNDUuODQwNDc5LCAxMC44NjA3N10sIFs0NS44NDAzNjMsIDEwLjg2MDczXSwgWzQ1Ljg0MDE3NiwgMTAuODYwNjYyXSwgWzQ1Ljg0MDEwMywgMTAuODYwNjM1XSwgWzQ1Ljg0MDA1NSwgMTAuODYwNjE1XSwgWzQ1Ljg0MDAwNywgMTAuODYwNTkyXSwgWzQ1LjgzOTk2OCwgMTAuODYwNTY4XSwgWzQ1LjgzOTkzNCwgMTAuODYwNTM4XSwgWzQ1LjgzOTg5NiwgMTAuODYwNDM3XSwgWzQ1LjgzOTg5OCwgMTAuODYwMzg4XSwgWzQ1LjgzOTkxLCAxMC44NjAzMzJdLCBbNDUuODM5OTI2LCAxMC44NjAyOF0sIFs0NS44Mzk5NDMsIDEwLjg2MDIzMV0sIFs0NS44Mzk5NTIsIDEwLjg2MDE4NV0sIFs0NS44Mzk5NTksIDEwLjg2MDE0Nl0sIFs0NS44Mzk5NjIsIDEwLjg2MDEwN10sIFs0NS44Mzk5NTUsIDEwLjg2MDA0OF0sIFs0NS44Mzk5MzcsIDEwLjg1OTk5Ml0sIFs0NS44Mzk4OTIsIDEwLjg1OTkyM10sIFs0NS44Mzk4NDksIDEwLjg1OTg4XSwgWzQ1LjgzOTc4NSwgMTAuODU5ODRdLCBbNDUuODM5NzAxLCAxMC44NTk4MDNdLCBbNDUuODM5NTgyLCAxMC44NTk3NjldLCBbNDUuODM5NDg2LCAxMC44NTk3NDVdLCBbNDUuODM5Mzk1LCAxMC44NTk3MzRdLCBbNDUuODM5Mjk3LCAxMC44NTk3Ml0sIFs0NS44MzkyMjgsIDEwLjg1OTcwM10sIFs0NS44MzkxNDYsIDEwLjg1OTY4M10sIFs0NS44MzkwNDgsIDEwLjg1OTY2Ml0sIFs0NS44Mzg5OTgsIDEwLjg1OTY1NV0sIFs0NS44Mzg4ODgsIDEwLjg1OTYzMV0sIFs0NS44Mzg2ODksIDEwLjg1OTYzOF0sIFs0NS44Mzg1NzcsIDEwLjg1OTY0N10sIFs0NS44Mzg0NjMsIDEwLjg1OTY0Nl0sIFs0NS44MzgyNDQsIDEwLjg1OTYwN10sIFs0NS44MzgwOTgsIDEwLjg1OTU2XSwgWzQ1LjgzNzk3NSwgMTAuODU5NTIyXSwgWzQ1LjgzNzgyMiwgMTAuODU5NDYyXSwgWzQ1LjgzNzcwNiwgMTAuODU5NDI1XSwgWzQ1LjgzNzYxLCAxMC44NTkzOTFdLCBbNDUuODM3NDcxLCAxMC44NTkzMjRdLCBbNDUuODM3MzkxLCAxMC44NTkyNjddLCBbNDUuODM3MzA1LCAxMC44NTkyMDhdLCBbNDUuODM3MjE2LCAxMC44NTkxMzVdLCBbNDUuODM3MTE4LCAxMC44NTkwNDhdLCBbNDUuODM3MDQzLCAxMC44NTg5NjNdLCBbNDUuODM2OTY0LCAxMC44NTg4NzddLCBbNDUuODM2NzY2LCAxMC44NTg3NDddLCBbNDUuODM2NjM2LCAxMC44NTg3MzJdLCBbNDUuODM2NTM1LCAxMC44NTg3NDFdLCBbNDUuODM2Mjg5LCAxMC44NTg3MDZdLCBbNDUuODM2MTU0LCAxMC44NTg2NjhdLCBbNDUuODM2MDMxLCAxMC44NTg2NDFdLCBbNDUuODM1OTEsIDEwLjg1ODYxNl0sIFs0NS44MzU3OTMsIDEwLjg1ODU5Nl0sIFs0NS44MzU2ODgsIDEwLjg1ODU3MV0sIFs0NS44MzU1MiwgMTAuODU4NTI0XSwgWzQ1LjgzNTQ3NCwgMTAuODU4NTA0XSwgWzQ1LjgzNTM2MiwgMTAuODU4NDg2XSwgWzQ1LjgzNTMxNCwgMTAuODU4NTA1XSwgWzQ1LjgzNTI2OCwgMTAuODU4NTQ0XSwgWzQ1LjgzNTIyMiwgMTAuODU4NjEyXSwgWzQ1LjgzNTE5MiwgMTAuODU4Njc3XSwgWzQ1LjgzNTE2OSwgMTAuODU4NzU1XSwgWzQ1LjgzNTE2NiwgMTAuODU4ODE0XSwgWzQ1LjgzNTE2NCwgMTAuODU4ODczXSwgWzQ1LjgzNTE1OSwgMTAuODU4OTMyXSwgWzQ1LjgzNTE0NSwgMTAuODU4OTk3XSwgWzQ1LjgzNTEyMiwgMTAuODU5MDYyXSwgWzQ1LjgzNTA5NCwgMTAuODU5MTI0XSwgWzQ1LjgzNTA0NiwgMTAuODU5MTk5XSwgWzQ1LjgzNDk5LCAxMC44NTkyOTNdLCBbNDUuODM0OTY5LCAxMC44NTkzNThdLCBbNDUuODM0OTU4LCAxMC44NTk0MV0sIFs0NS44MzQ5NDYsIDEwLjg1OTQ0OV0sIFs0NS44MzQ5MTQsIDEwLjg1OTQ5OF0sIFs0NS44MzQ4NjMsIDEwLjg1OTUzN10sIFs0NS44MzQ3OTcsIDEwLjg1OTU3Ml0sIFs0NS44MzQ3MTUsIDEwLjg1OTU4MV0sIFs0NS44MzQ2MjEsIDEwLjg1OTU3XSwgWzQ1LjgzNDUzMiwgMTAuODU5NTQzXSwgWzQ1LjgzNDQ3NywgMTAuODU5NTMyXSwgWzQ1LjgzNDM3LCAxMC44NTk1NjddLCBbNDUuODM0MzMxLCAxMC44NTk2MDldLCBbNDUuODM0Mjg5LCAxMC44NTk2NDhdLCBbNDUuODM0MjY0LCAxMC44NTk2OTddLCBbNDUuODM0MjUyLCAxMC44NTk3MzZdLCBbNDUuODM0MjI5LCAxMC44NTk4MTRdLCBbNDUuODM0MTg1LCAxMC44NTk4ODldLCBbNDUuODM0MTI1LCAxMC44NTk5NjddLCBbNDUuODM0MDYxLCAxMC44NjAwMzFdLCBbNDUuODM0MDI0LCAxMC44NjAwNjRdLCBbNDUuODMzOTgzLCAxMC44NjAwODldLCBbNDUuODMzOTE0LCAxMC44NjAxMDVdLCBbNDUuODMzODYyLCAxMC44NjAxMDRdLCBbNDUuODMzODAzLCAxMC44NjAwNzhdLCBbNDUuODMzNzUzLCAxMC44NjAwNDRdLCBbNDUuODMzNjg0LCAxMC44NjAwMDRdLCBbNDUuODMzNjE0LCAxMC44NTk5NjhdLCBbNDUuODMzNTM2LCAxMC44NTk5MzddLCBbNDUuODMzNDU0LCAxMC44NTk5MDRdLCBbNDUuODMzMjk5LCAxMC44NTk4M10sIFs0NS44MzMxNzYsIDEwLjg1OTc5OV0sIFs0NS44MzMwNDYsIDEwLjg1OTc2NV0sIFs0NS44MzI5MDcsIDEwLjg1OTcyMV0sIFs0NS44MzI4MDQsIDEwLjg1OTY4MV0sIFs0NS44MzI2NzksIDEwLjg1OTYyN10sIFs0NS44MzI1NDksIDEwLjg1OTU3M10sIFs0NS44MzI0NDQsIDEwLjg1OTU0Ml0sIFs0NS44MzIzMzQsIDEwLjg1OTQ5NV0sIFs0NS44MzIxODksIDEwLjg1OTQxOV0sIFs0NS44MzIwOTMsIDEwLjg1OTM2Ml0sIFs0NS44MzIwMDIsIDEwLjg1OTI4OV0sIFs0NS44MzE5MjksIDEwLjg1OTIyOV0sIFs0NS44MzE4MjksIDEwLjg1OTE0M10sIFs0NS44MzE3MjQsIDEwLjg1OTA2NF0sIFs0NS44MzE2NTYsIDEwLjg1OTAxN10sIFs0NS44MzE1OSwgMTAuODU4OTY3XSwgWzQ1LjgzMTQ5LCAxMC44NTg4ODRdLCBbNDUuODMxMzc0LCAxMC44NTg3ODhdLCBbNDUuODMxMjE5LCAxMC44NTg2NjldLCBbNDUuODMxMDgzLCAxMC44NTg1NzNdLCBbNDUuODMwOTQ2LCAxMC44NTg0ODldLCBbNDUuODMwODA5LCAxMC44NTg0MTldLCBbNDUuODMwNjIsIDEwLjg1ODMyOV0sIFs0NS44MzA0NTksIDEwLjg1ODIzMl0sIFs0NS44MzAzMDgsIDEwLjg1ODE0NV0sIFs0NS44MzAxNiwgMTAuODU4MDYyXSwgWzQ1LjgyOTk5NCwgMTAuODU3OTUyXSwgWzQ1LjgyOTg0NCwgMTAuODU3ODY5XSwgWzQ1LjgyOTY5OCwgMTAuODU3Nzg2XSwgWzQ1LjgyOTU2NiwgMTAuODU3NzAyXSwgWzQ1LjgyOTQxNiwgMTAuODU3NTkzXSwgWzQ1LjgyOTMwMiwgMTAuODU3NTI2XSwgWzQ1LjgyOTE0LCAxMC44NTc0NDldLCBbNDUuODI5MDA2LCAxMC44NTczODVdLCBbNDUuODI4NzIxLCAxMC44NTcxOTZdLCBbNDUuODI4NDc1LCAxMC44NTcwMV0sIFs0NS44MjgzNTIsIDEwLjg1NjkyN10sIFs0NS44MjgxOTgsIDEwLjg1NjgxMV0sIFs0NS44Mjc5ODEsIDEwLjg1NjY5NF0sIFs0NS44Mjc5MTMsIDEwLjg1NjY2MV0sIFs0NS44Mjc3OTIsIDEwLjg1NjYxM10sIFs0NS44Mjc3MzMsIDEwLjg1NjU5M10sIFs0NS44Mjc1NjEsIDEwLjg1NjU1NV0sIFs0NS44Mjc0NTIsIDEwLjg1NjU3XSwgWzQ1LjgyNzQwNiwgMTAuODU2NTkzXSwgWzQ1LjgyNzM1OCwgMTAuODU2NjQxXSwgWzQ1LjgyNzMsIDEwLjg1NjcwOV0sIFs0NS44MjcxOTQsIDEwLjg1NjgzNl0sIFs0NS44MjcwODIsIDEwLjg1Njk0Ml0sIFs0NS44MjcwMTQsIDEwLjg1NzAwOF0sIFs0NS44MzU0NzcsIDEwLjgzMjM0MV0sIFs0NS44Mzc2MywgMTAuODI1ODUzXV1dXSwgInR5cGUiOiAiTXVsdGlQb2x5Z29uIn0sICJpZCI6ICI3IiwgInByb3BlcnRpZXMiOiB7ImRlc2NyaXppb25lX29yaWdpbmUiOiAiTm9uIGNvZGlmaWNhdG8iLCAiZ21sX2lkIjogIklELkFDUVVFRklTSUNIRS5TUEVDQ0hJLkFDUVVBLjU0MTgzIiwgIm5hc3AiOiAwLCAibmF0dXJhX3NwZWNjaGlvX2FjcXVhIjogIk5vbiBjb2RpZmljYXRvIiwgIm5vbWUiOiAiTEFHTyBESSBHQVJEQSIsICJvcmlnaW5lIjogMH0sICJ0eXBlIjogIkZlYXR1cmUifSwgeyJnZW9tZXRyeSI6IHsiY29vcmRpbmF0ZXMiOiBbW1tbNDYuMDU5MTE0LCAxMC45Nzc0MTddLCBbNDYuMDU5MTA0LCAxMC45Nzc1ODhdLCBbNDYuMDU5MDkyLCAxMC45Nzc3NzldLCBbNDYuMDU5MDQ2LCAxMC45Nzc3NzhdLCBbNDYuMDU4OTk2LCAxMC45Nzc3ODNdLCBbNDYuMDU4OTUsIDEwLjk3Nzc5OF0sIFs0Ni4wNTg3OTQsIDEwLjk3NzgyNF0sIFs0Ni4wNTg3NDIsIDEwLjk3NzgwM10sIFs0Ni4wNTg3MDEsIDEwLjk3Nzc3MV0sIFs0Ni4wNTg2NDksIDEwLjk3NzczMl0sIFs0Ni4wNTg1MjEsIDEwLjk3NzU4OF0sIFs0Ni4wNTg0NTEsIDEwLjk3NzQ3N10sIFs0Ni4wNTgzOSwgMTAuOTc3NDA0XSwgWzQ2LjA1ODMxNSwgMTAuOTc3MzQ2XSwgWzQ2LjA1ODIwNCwgMTAuOTc3MjYxXSwgWzQ2LjA1ODExNiwgMTAuOTc3MThdLCBbNDYuMDU4MDAxLCAxMC45NzcwNTldLCBbNDYuMDU3OTIsIDEwLjk3NjkzNl0sIFs0Ni4wNTc4ODIsIDEwLjk3Njg2Ml0sIFs0Ni4wNTc4NjIsIDEwLjk3NjgzMl0sIFs0Ni4wNTc3OTcsIDEwLjk3Njc3OF0sIFs0Ni4wNTc3NTYsIDEwLjk3Njc0OF0sIFs0Ni4wNTc3MTMsIDEwLjk3NjcyN10sIFs0Ni4wNTc2NTMsIDEwLjk3NjcxOV0sIFs0Ni4wNTc1ODcsIDEwLjk3NjcwMV0sIFs0Ni4wNTc1MzMsIDEwLjk3NjY2XSwgWzQ2LjA1NzQ2MywgMTAuOTc2NTUzXSwgWzQ2LjA1NzQwNCwgMTAuOTc2NDAxXSwgWzQ2LjA1NzM1LCAxMC45NzYyNjhdLCBbNDYuMDU3MjcyLCAxMC45NzYxMzFdLCBbNDYuMDU3MTg5LCAxMC45NzYwMTRdLCBbNDYuMDU3MTE5LCAxMC45NzU5MDRdLCBbNDYuMDU3MDc3LCAxMC45NzU3OThdLCBbNDYuMDU3MDM5LCAxMC45NzU3MDJdLCBbNDYuMDU3MDI5LCAxMC45NzU1OV0sIFs0Ni4wNTcwNjcsIDEwLjk3NTQ1OV0sIFs0Ni4wNTcxMTQsIDEwLjk3NTM0Nl0sIFs0Ni4wNTcxMzcsIDEwLjk3NTE5OF0sIFs0Ni4wNTcxMDIsIDEwLjk3NTA3OV0sIFs0Ni4wNTcwMzcsIDEwLjk3NDk3Ml0sIFs0Ni4wNTY5OTYsIDEwLjk3NDkwMl0sIFs0Ni4wNTY4MDUsIDEwLjk3NDQ2NF0sIFs0Ni4wNTY3MzgsIDEwLjk3NDMyNF0sIFs0Ni4wNTY2OTEsIDEwLjk3NDIyMV0sIFs0Ni4wNTY2MzMsIDEwLjk3NDEwMl0sIFs0Ni4wNTY1NjksIDEwLjk3MzkwM10sIFs0Ni4wNTY1MTQsIDEwLjk3Mzc3NF0sIFs0Ni4wNTY0MTQsIDEwLjk3MzU1NF0sIFs0Ni4wNTYyOTgsIDEwLjk3MzMwNV0sIFs0Ni4wNTYxNzEsIDEwLjk3MzAzNl0sIFs0Ni4wNTYwMTcsIDEwLjk3MjcxN10sIFs0Ni4wNTU4ODUsIDEwLjk3MjQ1MV0sIFs0Ni4wNTU3MTcsIDEwLjk3MjE3NF0sIFs0Ni4wNTU1OTYsIDEwLjk3MTk3MV0sIFs0Ni4wNTU0MzQsIDEwLjk3MTcyNF0sIFs0Ni4wNTUyOTUsIDEwLjk3MTUxN10sIFs0Ni4wNTUxNzEsIDEwLjk3MTM3OV0sIFs0Ni4wNTUwNTgsIDEwLjk3MTI4NF0sIFs0Ni4wNTQ4NzIsIDEwLjk3MTE1Ml0sIFs0Ni4wNTQ3NDgsIDEwLjk3MTA0NF0sIFs0Ni4wNTQ2NTYsIDEwLjk3MDkxNl0sIFs0Ni4wNTQ1ODIsIDEwLjk3MDc1NF0sIFs0Ni4wNTQ1MzYsIDEwLjk3MDYwMV0sIFs0Ni4wNTQ0NzQsIDEwLjk3MDQ0Nl0sIFs0Ni4wNTQzOTEsIDEwLjk3MDMxNV0sIFs0Ni4wNTQzMDgsIDEwLjk3MDIwNV0sIFs0Ni4wNTQyMTgsIDEwLjk3MDA4MV0sIFs0Ni4wNTQxNDQsIDEwLjk2OTk1OF0sIFs0Ni4wNTQwNzYsIDEwLjk2OTg0OF0sIFs0Ni4wNTM5ODQsIDEwLjk2OTc0NF0sIFs0Ni4wNTM4OTgsIDEwLjk2OTY3M10sIFs0Ni4wNTM4MjYsIDEwLjk2OTYwNV0sIFs0Ni4wNTM3NjUsIDEwLjk2OTU2MV0sIFs0Ni4wNTM3MjEsIDEwLjk2OTU2M10sIFs0Ni4wNTM2ODQsIDEwLjk2OTU3Nl0sIFs0Ni4wNTM2NDksIDEwLjk2OTYzNF0sIFs0Ni4wNTM2MDcsIDEwLjk2OTc0OF0sIFs0Ni4wNTM1NzgsIDEwLjk2OTg2OV0sIFs0Ni4wNTM1MzEsIDEwLjk2OTk5Nl0sIFs0Ni4wNTM0NzYsIDEwLjk3MDE1Nl0sIFs0Ni4wNTM0MzYsIDEwLjk3MDI1M10sIFs0Ni4wNTMzOTIsIDEwLjk3MDMzOF0sIFs0Ni4wNTMzNTMsIDEwLjk3MDM3M10sIFs0Ni4wNTMzMjEsIDEwLjk3MDM4Ml0sIFs0Ni4wNTMyNjQsIDEwLjk3MDM3NF0sIFs0Ni4wNTMyMTYsIDEwLjk3MDM1M10sIFs0Ni4wNTMxNTcsIDEwLjk3MDMxOV0sIFs0Ni4wNTMxLCAxMC45NzAyNzVdLCBbNDYuMDUzMDI4LCAxMC45NzAyMTRdLCBbNDYuMDUyOTY1LCAxMC45NzAxNDddLCBbNDYuMDUyODc5LCAxMC45NzAwNDZdLCBbNDYuMDUyNzk2LCAxMC45Njk5NDldLCBbNDYuMDUyNzUsIDEwLjk2OTkwNV0sIFs0Ni4wNTI3MTksIDEwLjk2OTg4NV0sIFs0Ni4wNTI2OCwgMTAuOTY5ODc0XSwgWzQ2LjA1MjY0MSwgMTAuOTY5ODczXSwgWzQ2LjA1MjYyLCAxMC45Njk4OTldLCBbNDYuMDUyNjEsIDEwLjk2OTk1NF0sIFs0Ni4wNTI2MjMsIDEwLjk3MDA1Nl0sIFs0Ni4wNTI2MzUsIDEwLjk3MDE2OF0sIFs0Ni4wNTI2NDUsIDEwLjk3MDM0OV0sIFs0Ni4wNTI2NDcsIDEwLjk3MDUxN10sIFs0Ni4wNTI2NTYsIDEwLjk3MDcwNV0sIFs0Ni4wNTI2NzksIDEwLjk3MDg5Nl0sIFs0Ni4wNTI3MjcsIDEwLjk3MTEyN10sIFs0Ni4wNTI3NzcsIDEwLjk3MTM0NV0sIFs0Ni4wNTI4NDcsIDEwLjk3MTU3XSwgWzQ2LjA1MjkyNiwgMTAuOTcxODIyXSwgWzQ2LjA1MzAzLCAxMC45NzIxMV0sIFs0Ni4wNTMxNDYsIDEwLjk3MjM4Ml0sIFs0Ni4wNTMzNDYsIDEwLjk3Mjg3XSwgWzQ2LjA1MzQ0NCwgMTAuOTczMDY5XSwgWzQ2LjA1MzUzNSwgMTAuOTczMjc1XSwgWzQ2LjA1MzYxMSwgMTAuOTczNDc4XSwgWzQ2LjA1MzY1NCwgMTAuOTczNjQ5XSwgWzQ2LjA1MzY4LCAxMC45NzM4MThdLCBbNDYuMDUzNjk2LCAxMC45NzQwMDJdLCBbNDYuMDUzNzI4LCAxMC45NzQyMTZdLCBbNDYuMDUzNzUzLCAxMC45NzQzOThdLCBbNDYuMDUzNzcxLCAxMC45NzQ2MjJdLCBbNDYuMDUzNzY1LCAxMC45NzQ3MzZdLCBbNDYuMDUzNzUyLCAxMC45NzQ4MTVdLCBbNDYuMDUzNzM0LCAxMC45NzQ4NjRdLCBbNDYuMDUzNjczLCAxMC45NzQ5NTFdLCBbNDYuMDUzNjA5LCAxMC45NzQ5ODJdLCBbNDYuMDUzNTQ5LCAxMC45NzQ5ODFdLCBbNDYuMDUzNSwgMTAuOTc0OTM3XSwgWzQ2LjA1MzQzLCAxMC45NzQ4M10sIFs0Ni4wNTMzMjksIDEwLjk3NDYzN10sIFs0Ni4wNTMyMTUsIDEwLjk3NDQzNF0sIFs0Ni4wNTMwODMsIDEwLjk3NDE5NF0sIFs0Ni4wNTI5NTEsIDEwLjk3Mzk5MV0sIFs0Ni4wNTI4MjgsIDEwLjk3Mzc5N10sIFs0Ni4wNTI2OTUsIDEwLjk3MzU3N10sIFs0Ni4wNTI1NTcsIDEwLjk3MzMwOF0sIFs0Ni4wNTI0NzUsIDEwLjk3MzA2OV0sIFs0Ni4wNTIzNjcsIDEwLjk3Mjc4N10sIFs0Ni4wNTIyNzQsIDEwLjk3MjQ5M10sIFs0Ni4wNTIxNzUsIDEwLjk3MjIyMV0sIFs0Ni4wNTIwNjUsIDEwLjk3MTk4OF0sIFs0Ni4wNTE5MTMsIDEwLjk3MTczNV0sIFs0Ni4wNTE3NjUsIDEwLjk3MTQ1OV0sIFs0Ni4wNTE2MjIsIDEwLjk3MTE5Nl0sIFs0Ni4wNTE1MTEsIDEwLjk3MDk3Nl0sIFs0Ni4wNTE0MTMsIDEwLjk3MDc1XSwgWzQ2LjA1MTMwNSwgMTAuOTcwNTU0XSwgWzQ2LjA1MTE5NSwgMTAuOTcwMzk0XSwgWzQ2LjA1MTEyMiwgMTAuOTcwMjQxXSwgWzQ2LjA1MTA5NCwgMTAuOTcwMDk2XSwgWzQ2LjA1MTA4OSwgMTAuOTY5OTQxXSwgWzQ2LjA1MTA4NiwgMTAuOTY5Nzk2XSwgWzQ2LjA1MTA3OCwgMTAuOTY5Njk0XSwgWzQ2LjA1MTA1LCAxMC45Njk1NjJdLCBbNDYuMDUwOTk4LCAxMC45Njk0NTldLCBbNDYuMDUwOTE3LCAxMC45NjkzNTVdLCBbNDYuMDUwODEyLCAxMC45NjkxOTVdLCBbNDYuMDUwNjk1LCAxMC45NjkwMDhdLCBbNDYuMDUwNjAxLCAxMC45Njg4MzldLCBbNDYuMDUwNTAzLCAxMC45Njg2NjVdLCBbNDYuMDUwNDE4LCAxMC45Njg1MTldLCBbNDYuMDUwMzM3LCAxMC45Njg0MTVdLCBbNDYuMDUwMjQyLCAxMC45NjgyOTFdLCBbNDYuMDUwMTY4LCAxMC45NjgxODRdLCBbNDYuMDUwMDc0LCAxMC45NjgwMjRdLCBbNDYuMDUwMDAzLCAxMC45Njc4NjhdLCBbNDYuMDQ5OTM0LCAxMC45Njc3MTVdLCBbNDYuMDQ5ODc2LCAxMC45Njc2MDJdLCBbNDYuMDQ5ODAyLCAxMC45Njc0NzldLCBbNDYuMDQ5NzIxLCAxMC45NjczNjJdLCBbNDYuMDQ5NjU2LCAxMC45NjcyNTVdLCBbNDYuMDQ5NjAyLCAxMC45NjcxNzJdLCBbNDYuMDQ5NTQxLCAxMC45NjcxMDJdLCBbNDYuMDQ5NDUsIDEwLjk2NzA0N10sIFs0Ni4wNDkzNzgsIDEwLjk2Njk3OV0sIFs0Ni4wNDkzMTUsIDEwLjk2Njg4OV0sIFs0Ni4wNDkyMjgsIDEwLjk2NjcyNl0sIFs0Ni4wNDkxODcsIDEwLjk2NjY1M10sIFs0Ni4wNDkxNDcsIDEwLjk2NjYwM10sIFs0Ni4wNDkxMDksIDEwLjk2NjU0OV0sIFs0Ni4wNDkwMTcsIDEwLjk2NjM5OV0sIFs0Ni4wNDg5NzEsIDEwLjk2NjM0OV0sIFs0Ni4wNDg5MjEsIDEwLjk2NjI4Nl0sIFs0Ni4wNDg4MSwgMTAuOTY2MTM2XSwgWzQ2LjA0ODczOCwgMTAuOTY2MDI3XSwgWzQ2LjA0ODY3OSwgMTAuOTY1OTM3XSwgWzQ2LjA0ODYyNSwgMTAuOTY1ODc3XSwgWzQ2LjA0ODU3LCAxMC45NjU4NV0sIFs0Ni4wNDg0OSwgMTAuOTY1ODQ5XSwgWzQ2LjA0ODQyMiwgMTAuOTY1ODM4XSwgWzQ2LjA0ODMzMSwgMTAuOTY1Nzk3XSwgWzQ2LjA0ODI2NywgMTAuOTY1NzRdLCBbNDYuMDQ4MTk3LCAxMC45NjU2MzNdLCBbNDYuMDQ4MTU5LCAxMC45NjU1NDddLCBbNDYuMDQ4MTA1LCAxMC45NjU0MTVdLCBbNDYuMDQ4MDY3LCAxMC45NjUzMDldLCBbNDYuMDQ4MDM4LCAxMC45NjUxOV0sIFs0Ni4wNDgwMTYsIDEwLjk2NTA3NV0sIFs0Ni4wNDc5OSwgMTAuOTY0OTE3XSwgWzQ2LjA0Nzk0NiwgMTAuOTY0NzMyXSwgWzQ2LjA0NzkwOCwgMTAuOTY0NjI2XSwgWzQ2LjA0Nzg1NCwgMTAuOTY0NTFdLCBbNDYuMDQ3ODA0LCAxMC45NjQ0MTddLCBbNDYuMDQ3Njk4LCAxMC45NjQyNjFdLCBbNDYuMDQ3NjI1LCAxMC45NjQxODVdLCBbNDYuMDQ3NTUsIDEwLjk2NDExNF0sIFs0Ni4wNDc0NzMsIDEwLjk2NDA1MV0sIFs0Ni4wNDczODcsIDEwLjk2Mzk4M10sIFs0Ni4wNDcyODksIDEwLjk2MzkwM10sIFs0Ni4wNDcxNzgsIDEwLjk2Mzc5M10sIFs0Ni4wNDcwODcsIDEwLjk2MzcwNl0sIFs0Ni4wNDcwMSwgMTAuOTYzNjM2XSwgWzQ2LjA0Njg0MiwgMTAuOTYzNDkyXSwgWzQ2LjA0NjczNSwgMTAuOTYzNDE4XSwgWzQ2LjA0NjYwNywgMTAuOTYzMzE3XSwgWzQ2LjA0NjQ2NCwgMTAuOTYzMThdLCBbNDYuMDQ2MzM1LCAxMC45NjMwMzNdLCBbNDYuMDQ2MTg3LCAxMC45NjI5NDJdLCBbNDYuMDQ2MDc4LCAxMC45NjI5Ml0sIFs0Ni4wNDU5OTYsIDEwLjk2Mjg5OV0sIFs0Ni4wNDU5MzQsIDEwLjk2Mjg4OV0sIFs0Ni4wNDU4MzMsIDEwLjk2MjkwM10sIFs0Ni4wNDU3MDEsIDEwLjk2MjkwOF0sIFs0Ni4wNDU3MDcsIDEwLjk2Mjg2MV0sIFs0Ni4wNDU3MTMsIDEwLjk2MjgxNl0sIFs0Ni4wNDU3NjEsIDEwLjk2Mjc4NF0sIFs0Ni4wNDU4MjUsIDEwLjk2MjcyOV0sIFs0Ni4wNDU4NzgsIDEwLjk2MjY4NF0sIFs0Ni4wNDU5MjcsIDEwLjk2MjYyOV0sIFs0Ni4wNDU5OTEsIDEwLjk2MjU3MV0sIFs0Ni4wNDYwMjgsIDEwLjk2MjU1Ml0sIFs0Ni4wNDYxNTMsIDEwLjk2MjU5XSwgWzQ2LjA0NjIxOSwgMTAuOTYyNjMzXSwgWzQ2LjA0NjI4NSwgMTAuOTYyNjY3XSwgWzQ2LjA0NjM3MiwgMTAuOTYyNzAyXSwgWzQ2LjA0NjQ5MSwgMTAuOTYyNzYzXSwgWzQ2LjA0NjYwNywgMTAuOTYyODNdLCBbNDYuMDQ2NzI1LCAxMC45NjI4OTVdLCBbNDYuMDQ2ODUzLCAxMC45NjI5NjZdLCBbNDYuMDQ3MDEyLCAxMC45NjMwMzRdLCBbNDYuMDQ3MTUzLCAxMC45NjMxMDJdLCBbNDYuMDQ3MjUxLCAxMC45NjMxNjNdLCBbNDYuMDQ3Mzc0LCAxMC45NjMyMzRdLCBbNDYuMDQ3NTA2LCAxMC45NjMyNTZdLCBbNDYuMDQ3NjE5LCAxMC45NjMyNTRdLCBbNDYuMDQ3NzUxLCAxMC45NjMyNTNdLCBbNDYuMDQ3OTExLCAxMC45NjMyNDldLCBbNDYuMDQ4MDQxLCAxMC45NjMyNThdLCBbNDYuMDQ4MTgxLCAxMC45NjMyNTNdLCBbNDYuMDQ4MzIzLCAxMC45NjMyMjldLCBbNDYuMDQ4NDE5LCAxMC45NjMyMTRdLCBbNDYuMDQ4NTQ4LCAxMC45NjMxNjRdLCBbNDYuMDQ4NjM3LCAxMC45NjMxMTZdLCBbNDYuMDQ4NzAyLCAxMC45NjMwNThdLCBbNDYuMDQ4NzczLCAxMC45NjI5NjddLCBbNDYuMDQ4ODQ3LCAxMC45NjI4NjNdLCBbNDYuMDQ4OTA3LCAxMC45NjI3NTJdLCBbNDYuMDQ4OTU2LCAxMC45NjI2NDFdLCBbNDYuMDQ5MDAxLCAxMC45NjI1MjddLCBbNDYuMDQ5MDM4LCAxMC45NjI0MDldLCBbNDYuMDQ5MDY0LCAxMC45NjIzMTFdLCBbNDYuMDQ5MDk0LCAxMC45NjIyMjNdLCBbNDYuMDQ5MTA5LCAxMC45NjIxMzJdLCBbNDYuMDQ5MTIyLCAxMC45NjIxMTRdLCBbNDYuMDQ5MTQ4LCAxMC45NjIwNzhdLCBbNDYuMDQ5MTY5LCAxMC45NjIwNDNdLCBbNDYuMDQ5MjMsIDEwLjk2MTkyM10sIFs0Ni4wNDkyNjIsIDEwLjk2MTg2MV0sIFs0Ni4wNDkyOTUsIDEwLjk2MTgxNl0sIFs0Ni4wNDkzNDYsIDEwLjk2MTc2OF0sIFs0Ni4wNDkzOTQsIDEwLjk2MTczM10sIFs0Ni4wNDk0NTksIDEwLjk2MTY4NV0sIFs0Ni4wNDk1MTYsIDEwLjk2MTYzN10sIFs0Ni4wNDk1NzYsIDEwLjk2MTU4OV0sIFs0Ni4wNDk2MzcsIDEwLjk2MTUyMV0sIFs0Ni4wNDk2NjgsIDEwLjk2MTQwNF0sIFs0Ni4wNDk2ODcsIDEwLjk2MTI3Nl0sIFs0Ni4wNDk3MTIsIDEwLjk2MTE0Ml0sIFs0Ni4wNDk3NTIsIDEwLjk2MTAyMV0sIFs0Ni4wNDk4MTMsIDEwLjk2MDg3OF0sIFs0Ni4wNDk4NiwgMTAuOTYwNzY0XSwgWzQ2LjA0OTkzLCAxMC45NjA2MzFdLCBbNDYuMDUwMDA3LCAxMC45NjA1MjVdLCBbNDYuMDUwMTA0LCAxMC45NjA0MzJdLCBbNDYuMDUwMjQsIDEwLjk2MDMzM10sIFs0Ni4wNTAzNTMsIDEwLjk2MDI2XSwgWzQ2LjA1MDQ1NiwgMTAuOTYwMjI3XSwgWzQ2LjA1MDU4NCwgMTAuOTYwMl0sIFs0Ni4wNTA2NDIsIDEwLjk2MDE5NV0sIFs0Ni4wNTA3ODEsIDEwLjk2MDIxNV0sIFs0Ni4wNTA4NDUsIDEwLjk2MDI0Ml0sIFs0Ni4wNTA5MTksIDEwLjk2MDMyM10sIFs0Ni4wNTA5NzEsIDEwLjk2MDQwM10sIFs0Ni4wNTEwMjYsIDEwLjk2MDUzOV0sIFs0Ni4wNTEwNjMsIDEwLjk2MDY3MV0sIFs0Ni4wNTExMjgsIDEwLjk2MDg1XSwgWzQ2LjA1MTE5MiwgMTAuOTYwOTddLCBbNDYuMDUxMjU1LCAxMC45NjEwOV0sIFs0Ni4wNTEyODMsIDEwLjk2MTE5OV0sIFs0Ni4wNTEzMjMsIDEwLjk2MTMwNV0sIFs0Ni4wNTEzNzcsIDEwLjk2MTM4OF0sIFs0Ni4wNTE0NTQsIDEwLjk2MTQ4Nl0sIFs0Ni4wNTE1MjQsIDEwLjk2MTU1M10sIFs0Ni4wNTE1ODMsIDEwLjk2MTU5XSwgWzQ2LjA1MTYzNywgMTAuOTYxNjIxXSwgWzQ2LjA1MTY2NiwgMTAuOTYxNjY1XSwgWzQ2LjA1MTcxNiwgMTAuOTYxNzE5XSwgWzQ2LjA1MTc1LCAxMC45NjE3MzldLCBbNDYuMDUxODYsIDEwLjk2MTc1MV0sIFs0Ni4wNTE4OTgsIDEwLjk2MTc5NV0sIFs0Ni4wNTE5NSwgMTAuOTYxODUyXSwgWzQ2LjA1MjAwNCwgMTAuOTYxODc2XSwgWzQ2LjA1MjA0OCwgMTAuOTYxODY0XSwgWzQ2LjA1MjExOSwgMTAuOTYxODEzXSwgWzQ2LjA1MjE1MiwgMTAuOTYxNzUyXSwgWzQ2LjA1MjE2NywgMTAuOTYxNjc3XSwgWzQ2LjA1MjE2NSwgMTAuOTYxNjI3XSwgWzQ2LjA1MjEwMSwgMTAuOTYxNDQ1XSwgWzQ2LjA1MjAyOSwgMTAuOTYxMzE1XSwgWzQ2LjA1MTk0OCwgMTAuOTYxMjA4XSwgWzQ2LjA1MTg2LCAxMC45NjExMDRdLCBbNDYuMDUxNzY4LCAxMC45NjEwMTNdLCBbNDYuMDUxNjkxLCAxMC45NjA5MjNdLCBbNDYuMDUxNjEsIDEwLjk2MDc5Nl0sIFs0Ni4wNTE1MjUsIDEwLjk2MDY1M10sIFs0Ni4wNTE0NDcsIDEwLjk2MDQ5XSwgWzQ2LjA1MTM5NywgMTAuOTYwMzI0XSwgWzQ2LjA1MTM2OCwgMTAuOTYwMTk5XSwgWzQ2LjA1MTM0MiwgMTAuOTYwMDddLCBbNDYuMDUxMzM0LCAxMC45NTk5ODRdLCBbNDYuMDUxMzE5LCAxMC45NTk5MjVdLCBbNDYuMDUxMjk3LCAxMC45NTk4OTJdLCBbNDYuMDUxMjUzLCAxMC45NTk4NzddLCBbNDYuMDUxMjI4LCAxMC45NTk4OTNdLCBbNDYuMDUxMjAzLCAxMC45NTk5MDldLCBbNDYuMDUxMTQ1LCAxMC45NTk5MTFdLCBbNDYuMDUxMTE2LCAxMC45NTk5MTddLCBbNDYuMDUxMDY1LCAxMC45NTk5MTZdLCBbNDYuMDUxMDQ4LCAxMC45NTk4ODZdLCBbNDYuMDUxMDE5LCAxMC45NTk4MTZdLCBbNDYuMDUxMDQ0LCAxMC45NTk3NjFdLCBbNDYuMDUxMDgxLCAxMC45NTk3MjVdLCBbNDYuMDUxMTUzLCAxMC45NTk2NjhdLCBbNDYuMDUxMjU2LCAxMC45NTk2MjFdLCBbNDYuMDUxMzk2LCAxMC45NTk1NjJdLCBbNDYuMDUxNTE0LCAxMC45NTk1MDZdLCBbNDYuMDUxNjA4LCAxMC45NTk0NDldLCBbNDYuMDUxNjc5LCAxMC45NTk0MTRdLCBbNDYuMDUxNzE2LCAxMC45NTkzNTldLCBbNDYuMDUxNzE3LCAxMC45NTkzMV0sIFs0Ni4wNTE2OTUsIDEwLjk1OTI0N10sIFs0Ni4wNTE2OTgsIDEwLjk1OTE4NV0sIFs0Ni4wNTE3MTksIDEwLjk1OTE0Ml0sIFs0Ni4wNTE3NzQsIDEwLjk1OTA4OF0sIFs0Ni4wNTE4MTYsIDEwLjk1OTA0Nl0sIFs0Ni4wNTE4NjQsIDEwLjk1OTAxMV0sIFs0Ni4wNTE5MjQsIDEwLjk1OTAyMl0sIFs0Ni4wNTE5NjksIDEwLjk1OTA1XSwgWzQ2LjA1MjAxOSwgMTAuOTU5MDc0XSwgWzQ2LjA1MjA4NywgMTAuOTU5MDkyXSwgWzQ2LjA1MjE2OCwgMTAuOTU5MDg0XSwgWzQ2LjA1MjI0NiwgMTAuOTU5MDZdLCBbNDYuMDUyMzMzLCAxMC45NTkwMzVdLCBbNDYuMDUyNDQ5LCAxMC45NTkwNDhdLCBbNDYuMDUyNTg0LCAxMC45NTkwNzFdLCBbNDYuMDUyNzMyLCAxMC45NTkxMTRdLCBbNDYuMDUyODk4LCAxMC45NTkxODNdLCBbNDYuMDUzMDg5LCAxMC45NTkyNDFdLCBbNDYuMDUzMjI4LCAxMC45NTkyNDRdLCBbNDYuMDUzMzY1LCAxMC45NTkyNTddLCBbNDYuMDUzNTIzLCAxMC45NTkyOV0sIFs0Ni4wNTM2ODIsIDEwLjk1OTM1Nl0sIFs0Ni4wNTM3OTEsIDEwLjk1OTQyNV0sIFs0Ni4wNTM4ODgsIDEwLjk1OTUzMl0sIFs0Ni4wNTM5NzMsIDEwLjk1OTY4Ml0sIFs0Ni4wNTQwMjgsIDEwLjk1OTc5OF0sIFs0Ni4wNTQwODksIDEwLjk1OTkyNV0sIFs0Ni4wNTQxNzYsIDEwLjk2MDA5NF0sIFs0Ni4wNTQyNiwgMTAuOTYwMzFdLCBbNDYuMDU0Mzc0LCAxMC45NjA1NTldLCBbNDYuMDU0NDkyLCAxMC45NjA4MjFdLCBbNDYuMDU0NjIzLCAxMC45NjExMTRdLCBbNDYuMDU0NzI3LCAxMC45NjEzNzldLCBbNDYuMDU0ODE1LCAxMC45NjE2NjddLCBbNDYuMDU0OTA2LCAxMC45NjE5MjVdLCBbNDYuMDU1MDA4LCAxMC45NjIxNzRdLCBbNDYuMDU1MTAxLCAxMC45NjI0XSwgWzQ2LjA1NTE4OCwgMTAuOTYyNTg5XSwgWzQ2LjA1NTI1NywgMTAuOTYyNzU5XSwgWzQ2LjA1NTMyOCwgMTAuOTYyOTQ3XSwgWzQ2LjA1NTQwOCwgMTAuOTYzMTQ3XSwgWzQ2LjA1NTUyNywgMTAuOTYzMzddLCBbNDYuMDU1NjU0LCAxMC45NjM2MTldLCBbNDYuMDU1ODEzLCAxMC45NjM5MTJdLCBbNDYuMDU1OTM2LCAxMC45NjQxMjJdLCBbNDYuMDU2MDM5LCAxMC45NjQyNzJdLCBbNDYuMDU2MTIyLCAxMC45NjQzODZdLCBbNDYuMDU2MTY0LCAxMC45NjQ1MDVdLCBbNDYuMDU2MTg1LCAxMC45NjQ2NTRdLCBbNDYuMDU2MTYyLCAxMC45NjQ3NDJdLCBbNDYuMDU2MTIsIDEwLjk2NDhdLCBbNDYuMDU2MTA3LCAxMC45NjQ4MTFdLCBbNDYuMDU2MDU3LCAxMC45NjQ4NThdLCBbNDYuMDU1OTkyLCAxMC45NjQ5NjFdLCBbNDYuMDU1OTM2LCAxMC45NjUwNDJdLCBbNDYuMDU1ODU1LCAxMC45NjUxMjZdLCBbNDYuMDU1Nzg2LCAxMC45NjUxODZdLCBbNDYuMDU1NzIyLCAxMC45NjUyMzddLCBbNDYuMDU1NjgsIDEwLjk2NTI4Nl0sIFs0Ni4wNTU2NCwgMTAuOTY1MzhdLCBbNDYuMDU1NjI1LCAxMC45NjU0OThdLCBbNDYuMDU1NjMzLCAxMC45NjU2MDddLCBbNDYuMDU1NjU2LCAxMC45NjU3NDVdLCBbNDYuMDU1Njk2LCAxMC45NjU4NzRdLCBbNDYuMDU1NzI3LCAxMC45NjU5NTddLCBbNDYuMDU1NzgsIDEwLjk2NjA4M10sIFs0Ni4wNTU4MDksIDEwLjk2NjE5Ml0sIFs0Ni4wNTU3ODYsIDEwLjk2NjMxN10sIFs0Ni4wNTU3NDYsIDEwLjk2NjQyNF0sIFs0Ni4wNTU3MDYsIDEwLjk2NjUyNV0sIFs0Ni4wNTU2ODMsIDEwLjk2NjU4NF0sIFs0Ni4wNTU2NzksIDEwLjk2NjY3Nl0sIFs0Ni4wNTU3MDEsIDEwLjk2Njc3OF0sIFs0Ni4wNTU3MTgsIDEwLjk2Njg4XSwgWzQ2LjA1NTcyNCwgMTAuOTY2OTgyXSwgWzQ2LjA1NTcyLCAxMC45NjcwODddLCBbNDYuMDU1NzE1LCAxMC45NjcxNF0sIFs0Ni4wNTU3MDQsIDEwLjk2NzI1OF0sIFs0Ni4wNTU3MDgsIDEwLjk2NzM2Nl0sIFs0Ni4wNTU2NDUsIDEwLjk2NzQyNF0sIFs0Ni4wNTU2MiwgMTAuOTY3MjkyXSwgWzQ2LjA1NTYxOCwgMTAuOTY3MjIzXSwgWzQ2LjA1NTYxOSwgMTAuOTY3MTYxXSwgWzQ2LjA1NTYyMiwgMTAuOTY3MDk1XSwgWzQ2LjA1NTYyNSwgMTAuOTY3MDQyXSwgWzQ2LjA1NTYyOCwgMTAuOTY2OThdLCBbNDYuMDU1NjI4LCAxMC45NjY5MjddLCBbNDYuMDU1NjIsIDEwLjk2Njg3NV0sIFs0Ni4wNTU2MDUsIDEwLjk2Njc5OV0sIFs0Ni4wNTU1OSwgMTAuOTY2NzI2XSwgWzQ2LjA1NTU3NywgMTAuOTY2NjddLCBbNDYuMDU1NTYxLCAxMC45NjY2MjddLCBbNDYuMDU1NTMyLCAxMC45NjY1NjddLCBbNDYuMDU1NTA1LCAxMC45NjY1MV0sIFs0Ni4wNTU0ODEsIDEwLjk2NjQ3N10sIFs0Ni4wNTU0MzEsIDEwLjk2NjQ0Nl0sIFs0Ni4wNTUzOSwgMTAuOTY2NDM5XSwgWzQ2LjA1NTM1MSwgMTAuOTY2NDQ0XSwgWzQ2LjA1NTI4NCwgMTAuOTY2NDY2XSwgWzQ2LjA1NTIxOCwgMTAuOTY2NDg3XSwgWzQ2LjA1NTE0OSwgMTAuOTY2NTMyXSwgWzQ2LjA1NTEwNSwgMTAuOTY2NTc3XSwgWzQ2LjA1NTA0LCAxMC45NjY2MTFdLCBbNDYuMDU0OTkyLCAxMC45NjY2MTNdLCBbNDYuMDU0OTIyLCAxMC45NjY1OThdLCBbNDYuMDU0ODUzLCAxMC45NjY1OTRdLCBbNDYuMDU0NzY0LCAxMC45NjY1ODVdLCBbNDYuMDU0NywgMTAuOTY2NTk3XSwgWzQ2LjA1NDYxNywgMTAuOTY2NjQ0XSwgWzQ2LjA1NDUyNSwgMTAuOTY2Njg0XSwgWzQ2LjA1NDQ0NSwgMTAuOTY2Njk2XSwgWzQ2LjA1NDQwNCwgMTAuOTY2NzExXSwgWzQ2LjA1NDM3MywgMTAuOTY2ODAyXSwgWzQ2LjA1NDM1OCwgMTAuOTY2OTA3XSwgWzQ2LjA1NDM1NywgMTAuOTY2OTg5XSwgWzQ2LjA1NDM2OCwgMTAuOTY3MDc1XSwgWzQ2LjA1NDM5MSwgMTAuOTY3MjMzXSwgWzQ2LjA1NDQxMiwgMTAuOTY3MzQ5XSwgWzQ2LjA1NDQ0MywgMTAuOTY3NDU1XSwgWzQ2LjA1NDQ5MiwgMTAuOTY3NTc3XSwgWzQ2LjA1NDU1MiwgMTAuOTY3NjgxXSwgWzQ2LjA1NDYzMiwgMTAuOTY3NzQ4XSwgWzQ2LjA1NDcyLCAxMC45Njc3OTNdLCBbNDYuMDU0Nzc5LCAxMC45Njc4NV0sIFs0Ni4wNTQ4MjYsIDEwLjk2NzkxN10sIFs0Ni4wNTQ4NjcsIDEwLjk2Nzk2N10sIFs0Ni4wNTQ5MSwgMTAuOTY3OTg1XSwgWzQ2LjA1NDk5MiwgMTAuOTY3OTk3XSwgWzQ2LjA1NTA4MywgMTAuOTY3OTgyXSwgWzQ2LjA1NTIwNywgMTAuOTY3OTQ5XSwgWzQ2LjA1NTMyMiwgMTAuOTY3OTIyXSwgWzQ2LjA1NTQyLCAxMC45Njc4OTJdLCBbNDYuMDU1NTE0LCAxMC45Njc4NjhdLCBbNDYuMDU1NTkzLCAxMC45Njc4MzddLCBbNDYuMDU1NjY2LCAxMC45Njc4MDJdLCBbNDYuMDU1NzA1LCAxMC45Njc3N10sIFs0Ni4wNTU3MiwgMTAuOTY3NzAyXSwgWzQ2LjA1NTcxMSwgMTAuOTY3NjMzXSwgWzQ2LjA1NTcxNywgMTAuOTY3NTUxXSwgWzQ2LjA1NTc3NywgMTAuOTY3NTE5XSwgWzQ2LjA1NTc5NywgMTAuOTY3NTQ5XSwgWzQ2LjA1NTgyMiwgMTAuOTY3NTg5XSwgWzQ2LjA1NTg2LCAxMC45Njc2MTZdLCBbNDYuMDU1OTA4LCAxMC45Njc1OThdLCBbNDYuMDU1OTM0LCAxMC45Njc1NjZdLCBbNDYuMDU1OTY4LCAxMC45Njc1NDNdLCBbNDYuMDU2MDA1LCAxMC45Njc1MjhdLCBbNDYuMDU2MDQ0LCAxMC45Njc1NDldLCBbNDYuMDU2MDk0LCAxMC45Njc1NzldLCBbNDYuMDU2MTM0LCAxMC45Njc2MjNdLCBbNDYuMDU2MjI5LCAxMC45Njc3N10sIFs0Ni4wNTYyOTgsIDEwLjk2Nzg4XSwgWzQ2LjA1NjM2MywgMTAuOTY3OTldLCBbNDYuMDU2NDIzLCAxMC45NjgxXSwgWzQ2LjA1NjQ5MiwgMTAuOTY4MjU5XSwgWzQ2LjA1NjU2OCwgMTAuOTY4NDcxXSwgWzQ2LjA1NjY0NCwgMTAuOTY4NzM5XSwgWzQ2LjA1NjczMiwgMTAuOTY5MDQ0XSwgWzQ2LjA1NjgxOCwgMTAuOTY5MzE1XSwgWzQ2LjA1Njg5MSwgMTAuOTY5NTE0XSwgWzQ2LjA1Njk2NywgMTAuOTY5NzEzXSwgWzQ2LjA1NzA0NSwgMTAuOTY5ODY5XSwgWzQ2LjA1NzEzLCAxMC45NzAwMTZdLCBbNDYuMDU3MjM3LCAxMC45NzAxODNdLCBbNDYuMDU3Mzc1LCAxMC45NzAzNjNdLCBbNDYuMDU3NDk5LCAxMC45NzA1MDRdLCBbNDYuMDU3NjIxLCAxMC45NzA2MTZdLCBbNDYuMDU3NzA5LCAxMC45NzA2OF0sIFs0Ni4wNTc4MDYsIDEwLjk3MDc2Ml0sIFs0Ni4wNTc5MDQsIDEwLjk3MDg1Nl0sIFs0Ni4wNTc5OTIsIDEwLjk3MDk1N10sIFs0Ni4wNTgwNTIsIDEwLjk3MTA0XSwgWzQ2LjA1ODExLCAxMC45NzExNDddLCBbNDYuMDU4MTUxLCAxMC45NzEyMjZdLCBbNDYuMDU4MjE0LCAxMC45NzEyODddLCBbNDYuMDU4MjU1LCAxMC45NzEyNjhdLCBbNDYuMDU4MjksIDEwLjk3MTE5NF0sIFs0Ni4wNTgzMjksIDEwLjk3MTIzN10sIFs0Ni4wNTgzMywgMTAuOTcxMjhdLCBbNDYuMDU4MzMyLCAxMC45NzEzMjZdLCBbNDYuMDU4MzI3LCAxMC45NzE0MTVdLCBbNDYuMDU4MzE5LCAxMC45NzE0ODddLCBbNDYuMDU4MzA3LCAxMC45NzE1NDldLCBbNDYuMDU4MjksIDEwLjk3MTYwNF0sIFs0Ni4wNTgyNjQsIDEwLjk3MTY3OV0sIFs0Ni4wNTgyMzgsIDEwLjk3MTc1NF0sIFs0Ni4wNTgyMDcsIDEwLjk3MTgyM10sIFs0Ni4wNTgxNTQsIDEwLjk3MTg3N10sIFs0Ni4wNTgwOSwgMTAuOTcxOTA5XSwgWzQ2LjA1ODAxOSwgMTAuOTcxOTEzXSwgWzQ2LjA1Nzk1MSwgMTAuOTcxODkyXSwgWzQ2LjA1Nzg5NCwgMTAuOTcxODU4XSwgWzQ2LjA1NzgzNSwgMTAuOTcxODFdLCBbNDYuMDU3Nzg1LCAxMC45NzE3OTNdLCBbNDYuMDU3NzE3LCAxMC45NzE3ODhdLCBbNDYuMDU3NjYyLCAxMC45NzE3OTZdLCBbNDYuMDU3NjA3LCAxMC45NzE4MThdLCBbNDYuMDU3NTQ5LCAxMC45NzE4NjZdLCBbNDYuMDU3NTIzLCAxMC45NzE4OTJdLCBbNDYuMDU3NTA1LCAxMC45NzE5MzFdLCBbNDYuMDU3NDk3LCAxMC45NzE5NzddLCBbNDYuMDU3NDg5LCAxMC45NzIwNjhdLCBbNDYuMDU3NDg2LCAxMC45NzIxMzddLCBbNDYuMDU3NDgsIDEwLjk3MjIzNl0sIFs0Ni4wNTc0NywgMTAuOTcyMzU3XSwgWzQ2LjA1NzQ1OCwgMTAuOTcyNTY0XSwgWzQ2LjA1NzQ2MiwgMTAuOTcyNzkxXSwgWzQ2LjA1NzQ3MSwgMTAuOTczMDE4XSwgWzQ2LjA1NzQ4NywgMTAuOTczMjY0XSwgWzQ2LjA1NzQ5NiwgMTAuOTczNDQ1XSwgWzQ2LjA1NzUxOSwgMTAuOTczNjM3XSwgWzQ2LjA1NzU1NywgMTAuOTczODQ4XSwgWzQ2LjA1NzY0MiwgMTAuOTc0MTkyXSwgWzQ2LjA1NzY2NiwgMTAuOTc0MzI0XSwgWzQ2LjA1NzY4NywgMTAuOTc0NTExXSwgWzQ2LjA1NzcxMSwgMTAuOTc0NzQ1XSwgWzQ2LjA1Nzc0MSwgMTAuOTc0OTY2XSwgWzQ2LjA1Nzc3OSwgMTAuOTc1MTc3XSwgWzQ2LjA1NzgyMSwgMTAuOTc1MzM2XSwgWzQ2LjA1Nzg3NCwgMTAuOTc1NDc1XSwgWzQ2LjA1Nzk0MywgMTAuOTc1NjQ4XSwgWzQ2LjA1ODAyNywgMTAuOTc1ODU3XSwgWzQ2LjA1ODEyMSwgMTAuOTc2MDRdLCBbNDYuMDU4MTk1LCAxMC45NzYxNzZdLCBbNDYuMDU4MjcxLCAxMC45NzYzMDZdLCBbNDYuMDU4MzUxLCAxMC45NzY0NDNdLCBbNDYuMDU4NDAzLCAxMC45NzY1NDZdLCBbNDYuMDU4NDY1LCAxMC45NzY2NjldLCBbNDYuMDU4NTM5LCAxMC45NzY4MDldLCBbNDYuMDU4NjIyLCAxMC45NzY5NTZdLCBbNDYuMDU4Njg5LCAxMC45NzcwODVdLCBbNDYuMDU4NzYsIDEwLjk3NzIwOV0sIFs0Ni4wNTg4MTksIDEwLjk3NzI5OV0sIFs0Ni4wNTg4NzUsIDEwLjk3NzM3M10sIFs0Ni4wNTg5MiwgMTAuOTc3NDAzXSwgWzQ2LjA1OSwgMTAuOTc3NDA4XSwgWzQ2LjA1OTA2NywgMTAuOTc3NDFdLCBbNDYuMDU5MTE0LCAxMC45Nzc0MTddXV1dLCAidHlwZSI6ICJNdWx0aVBvbHlnb24ifSwgImlkIjogIjgiLCAicHJvcGVydGllcyI6IHsiZGVzY3JpemlvbmVfb3JpZ2luZSI6ICJOb24gY29kaWZpY2F0byIsICJnbWxfaWQiOiAiSUQuQUNRVUVGSVNJQ0hFLlNQRUNDSEkuQUNRVUEuNTQxMzMiLCAibmFzcCI6IDAsICJuYXR1cmFfc3BlY2NoaW9fYWNxdWEiOiAiTm9uIGNvZGlmaWNhdG8iLCAibm9tZSI6ICJMQUdPIERJIFRPQkxJTk8iLCAib3JpZ2luZSI6IDB9LCAidHlwZSI6ICJGZWF0dXJlIn0sIHsiZ2VvbWV0cnkiOiB7ImNvb3JkaW5hdGVzIjogW1tbWzQ1Ljg2OSwgMTAuOTE4MDU0XSwgWzQ1Ljg2ODk3MywgMTAuOTE4MDExXSwgWzQ1Ljg2ODk0OSwgMTAuOTE3OTU0XSwgWzQ1Ljg2ODkzMSwgMTAuOTE3ODg4XSwgWzQ1Ljg2ODkxLCAxMC45MTc3NzldLCBbNDUuODY4ODksIDEwLjkxNzYxMl0sIFs0NS44Njg4NywgMTAuOTE3NDQ3XSwgWzQ1Ljg2ODg1OCwgMTAuOTE3MzA5XSwgWzQ1Ljg2ODg0NywgMTAuOTE3MTE5XSwgWzQ1Ljg2ODg0NSwgMTAuOTE2OTg0XSwgWzQ1Ljg2ODg1NiwgMTAuOTE2ODhdLCBbNDUuODY4ODc4LCAxMC45MTY4MDVdLCBbNDUuODY4OTA2LCAxMC45MTY3NTRdLCBbNDUuODY4OTc5LCAxMC45MTY3NDldLCBbNDUuODY5MDE1LCAxMC45MTY3OTNdLCBbNDUuODY5MDYsIDEwLjkxNjg1M10sIFs0NS44NjkxLCAxMC45MTY5MjddLCBbNDUuODY5MTM2LCAxMC45MTddLCBbNDUuODY5MTY5LCAxMC45MTcwOV0sIFs0NS44NjkxOTcsIDEwLjkxNzE4NV0sIFs0NS44NjkyMTMsIDEwLjkxNzM0XSwgWzQ1Ljg2OTI0MSwgMTAuOTE3NDAzXSwgWzQ1Ljg2OTI1LCAxMC45MTc0NTldLCBbNDUuODY5MjU2LCAxMC45MTc1MTVdLCBbNDUuODY5MjUsIDEwLjkxNzYwM10sIFs0NS44NjkyNDEsIDEwLjkxNzcyMV0sIFs0NS44NjkyMTksIDEwLjkxNzgxMl0sIFs0NS44NjkxNzcsIDEwLjkxNzg3M10sIFs0NS44NjkxMjMsIDEwLjkxNzkzN10sIFs0NS44NjkwNjcsIDEwLjkxODAyXSwgWzQ1Ljg2OTAzLCAxMC45MTgwNTVdLCBbNDUuODY5LCAxMC45MTgwNTRdXV1dLCAidHlwZSI6ICJNdWx0aVBvbHlnb24ifSwgImlkIjogIjkiLCAicHJvcGVydGllcyI6IHsiZGVzY3JpemlvbmVfb3JpZ2luZSI6ICJOb24gY29kaWZpY2F0byIsICJnbWxfaWQiOiAiSUQuQUNRVUVGSVNJQ0hFLlNQRUNDSEkuQUNRVUEuNTQxODgiLCAibmFzcCI6IDAsICJuYXR1cmFfc3BlY2NoaW9fYWNxdWEiOiAiTm9uIGNvZGlmaWNhdG8iLCAibm9tZSI6ICIiLCAib3JpZ2luZSI6IDB9LCAidHlwZSI6ICJGZWF0dXJlIn0sIHsiZ2VvbWV0cnkiOiB7ImNvb3JkaW5hdGVzIjogW1tbWzQ1Ljk5ODExLCAxMC44Mjk1NTddLCBbNDUuOTk4MDYsIDEwLjgyOTU0Ml0sIFs0NS45OTgwMjIsIDEwLjgyOTUwNF0sIFs0NS45OTc5ODIsIDEwLjgyOTQyN10sIFs0NS45OTc5NTQsIDEwLjgyOTMzN10sIFs0NS45OTc5MzcsIDEwLjgyOTI0OF0sIFs0NS45OTc5MTcsIDEwLjgyOTE0OF0sIFs0NS45OTc4ODcsIDEwLjgyOTAyOF0sIFs0NS45OTc4ODIsIDEwLjgyODk0M10sIFs0NS45OTc5MTcsIDEwLjgyODgwM10sIFs0NS45OTc5NDMsIDEwLjgyODc0OF0sIFs0NS45OTc5NjEsIDEwLjgyODY3Nl0sIFs0NS45OTc5NTcsIDEwLjgyODYxNF0sIFs0NS45OTc5NzgsIDEwLjgyODQ4M10sIFs0NS45OTgwNDMsIDEwLjgyODQ1Nl0sIFs0NS45OTgwODIsIDEwLjgyODQ1MV0sIFs0NS45OTgxMjYsIDEwLjgyODQyN10sIFs0NS45OTgxNjEsIDEwLjgyODM4NV0sIFs0NS45OTgxODMsIDEwLjgyODM0N10sIFs0NS45OTgyMDIsIDEwLjgyODMxNV0sIFs0NS45OTgyMjMsIDEwLjgyODI4OV0sIFs0NS45OTgyNzEsIDEwLjgyODI3NV0sIFs0NS45OTgzMiwgMTAuODI4MjY0XSwgWzQ1Ljk5ODQwNSwgMTAuODI4MjI0XSwgWzQ1Ljk5ODQ4OSwgMTAuODI4MTg4XSwgWzQ1Ljk5ODU5MiwgMTAuODI4MTZdLCBbNDUuOTk4Njk4LCAxMC44MjgxMzFdLCBbNDUuOTk4Nzg0LCAxMC44MjgxMDVdLCBbNDUuOTk4ODU4LCAxMC44MjgwODldLCBbNDUuOTk4OTMxLCAxMC44MjgwNzldLCBbNDUuOTk4OTYzLCAxMC44MjgwOTZdLCBbNDUuOTk4OTkyLCAxMC44MjgxNDRdLCBbNDUuOTk5MDIzLCAxMC44MjgyMDFdLCBbNDUuOTk5MDM4LCAxMC44MjgyNDddLCBbNDUuOTk5MDUyLCAxMC44MjgzNTZdLCBbNDUuOTk5MDU5LCAxMC44Mjg0MTldLCBbNDUuOTk5MDg3LCAxMC44Mjg1MTNdLCBbNDUuOTk5MTI5LCAxMC44Mjg1OTddLCBbNDUuOTk5MTUzLCAxMC44Mjg2MjRdLCBbNDUuOTk5MTYxLCAxMC44Mjg2NjldLCBbNDUuOTk5MTU5LCAxMC44Mjg3NjddLCBbNDUuOTk5MTQ5LCAxMC44Mjg4NDVdLCBbNDUuOTk5MTE4LCAxMC44Mjg5MDVdLCBbNDUuOTk4OTk0LCAxMC44MjkwMjhdLCBbNDUuOTk4OTQxLCAxMC44MjkwNzJdLCBbNDUuOTk4ODY5LCAxMC44MjkxMzJdLCBbNDUuOTk4Nzg3LCAxMC44MjkxOTFdLCBbNDUuOTk4NzMxLCAxMC44MjkyMjhdLCBbNDUuOTk4NjU1LCAxMC44MjkyNzddLCBbNDUuOTk4NDE4LCAxMC44Mjk0MTJdLCBbNDUuOTk4MzM5LCAxMC44Mjk0NjVdLCBbNDUuOTk4MjQ2LCAxMC44Mjk1MjddLCBbNDUuOTk4MTEsIDEwLjgyOTU1N11dXV0sICJ0eXBlIjogIk11bHRpUG9seWdvbiJ9LCAiaWQiOiAiMTAiLCAicHJvcGVydGllcyI6IHsiZGVzY3JpemlvbmVfb3JpZ2luZSI6ICJOb24gY29kaWZpY2F0byIsICJnbWxfaWQiOiAiSUQuQUNRVUVGSVNJQ0hFLlNQRUNDSEkuQUNRVUEuNTQxNTgiLCAibmFzcCI6IDAsICJuYXR1cmFfc3BlY2NoaW9fYWNxdWEiOiAiTm9uIGNvZGlmaWNhdG8iLCAibm9tZSI6ICJMQUdPIFRPUkJJRVJBIERJIEZJQVZFXHUwMDI3IDE/IiwgIm9yaWdpbmUiOiAwfSwgInR5cGUiOiAiRmVhdHVyZSJ9XSwgInR5cGUiOiAiRmVhdHVyZUNvbGxlY3Rpb24ifSk7CgogICAgICAgIAo8L3NjcmlwdD4= onload="this.contentDocument.open();this.contentDocument.write(atob(this.getAttribute('data-html')));this.contentDocument.close();" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



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
lakes_inbbox.geometry[0]
```




    
![svg](03_Retrieving_data_from_spatial_database_infrastructures_files/03_Retrieving_data_from_spatial_database_infrastructures_110_0.svg)
    




```
map_lakes = folium.Map([lakes_inbbox.unary_union.centroid.y,lakes_inbbox.unary_union.centroid.x], zoom_start=10)
folium.GeoJson(lakes_inbbox.to_json()).add_to(map_lakes)
map_lakes
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe src="about:blank" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" data-html=PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgCiAgICAgICAgPHNjcmlwdD4KICAgICAgICAgICAgTF9OT19UT1VDSCA9IGZhbHNlOwogICAgICAgICAgICBMX0RJU0FCTEVfM0QgPSBmYWxzZTsKICAgICAgICA8L3NjcmlwdD4KICAgIAogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjYuMC9kaXN0L2xlYWZsZXQuanMiPjwvc2NyaXB0PgogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY29kZS5qcXVlcnkuY29tL2pxdWVyeS0xLjEyLjQubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9qcy9ib290c3RyYXAubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5qcyI+PC9zY3JpcHQ+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjYuMC9kaXN0L2xlYWZsZXQuY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vYm9vdHN0cmFwLzMuMi4wL2Nzcy9ib290c3RyYXAubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLXRoZW1lLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9mb250LWF3ZXNvbWUvNC42LjMvY3NzL2ZvbnQtYXdlc29tZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuanMuY2xvdWRmbGFyZS5jb20vYWpheC9saWJzL0xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLzIuMC4yL2xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvZ2gvcHl0aG9uLXZpc3VhbGl6YXRpb24vZm9saXVtL2ZvbGl1bS90ZW1wbGF0ZXMvbGVhZmxldC5hd2Vzb21lLnJvdGF0ZS5taW4uY3NzIi8+CiAgICA8c3R5bGU+aHRtbCwgYm9keSB7d2lkdGg6IDEwMCU7aGVpZ2h0OiAxMDAlO21hcmdpbjogMDtwYWRkaW5nOiAwO308L3N0eWxlPgogICAgPHN0eWxlPiNtYXAge3Bvc2l0aW9uOmFic29sdXRlO3RvcDowO2JvdHRvbTowO3JpZ2h0OjA7bGVmdDowO308L3N0eWxlPgogICAgCiAgICAgICAgICAgIDxtZXRhIG5hbWU9InZpZXdwb3J0IiBjb250ZW50PSJ3aWR0aD1kZXZpY2Utd2lkdGgsCiAgICAgICAgICAgICAgICBpbml0aWFsLXNjYWxlPTEuMCwgbWF4aW11bS1zY2FsZT0xLjAsIHVzZXItc2NhbGFibGU9bm8iIC8+CiAgICAgICAgICAgIDxzdHlsZT4KICAgICAgICAgICAgICAgICNtYXBfMmNlNzgxOTUzODYzNDE5OGE1Yzg1ODM4OGFjN2JmZmYgewogICAgICAgICAgICAgICAgICAgIHBvc2l0aW9uOiByZWxhdGl2ZTsKICAgICAgICAgICAgICAgICAgICB3aWR0aDogMTAwLjAlOwogICAgICAgICAgICAgICAgICAgIGhlaWdodDogMTAwLjAlOwogICAgICAgICAgICAgICAgICAgIGxlZnQ6IDAuMCU7CiAgICAgICAgICAgICAgICAgICAgdG9wOiAwLjAlOwogICAgICAgICAgICAgICAgfQogICAgICAgICAgICA8L3N0eWxlPgogICAgICAgIAo8L2hlYWQ+Cjxib2R5PiAgICAKICAgIAogICAgICAgICAgICA8ZGl2IGNsYXNzPSJmb2xpdW0tbWFwIiBpZD0ibWFwXzJjZTc4MTk1Mzg2MzQxOThhNWM4NTgzODhhYzdiZmZmIiA+PC9kaXY+CiAgICAgICAgCjwvYm9keT4KPHNjcmlwdD4gICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcF8yY2U3ODE5NTM4NjM0MTk4YTVjODU4Mzg4YWM3YmZmZiA9IEwubWFwKAogICAgICAgICAgICAgICAgIm1hcF8yY2U3ODE5NTM4NjM0MTk4YTVjODU4Mzg4YWM3YmZmZiIsCiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgY2VudGVyOiBbMTAuODYxODQ0OTMyMDU4OTc5LCA0NS44NzM0OTA5MDk5NDg0MTZdLAogICAgICAgICAgICAgICAgICAgIGNyczogTC5DUlMuRVBTRzM4NTcsCiAgICAgICAgICAgICAgICAgICAgem9vbTogMTAsCiAgICAgICAgICAgICAgICAgICAgem9vbUNvbnRyb2w6IHRydWUsCiAgICAgICAgICAgICAgICAgICAgcHJlZmVyQ2FudmFzOiBmYWxzZSwKICAgICAgICAgICAgICAgIH0KICAgICAgICAgICAgKTsKCiAgICAgICAgICAgIAoKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgdGlsZV9sYXllcl85OWRiNmRlOTk4MGU0ZmM5OTY0Njc4ZWE5M2JjMzRkYyA9IEwudGlsZUxheWVyKAogICAgICAgICAgICAgICAgImh0dHBzOi8ve3N9LnRpbGUub3BlbnN0cmVldG1hcC5vcmcve3p9L3t4fS97eX0ucG5nIiwKICAgICAgICAgICAgICAgIHsiYXR0cmlidXRpb24iOiAiRGF0YSBieSBcdTAwMjZjb3B5OyBcdTAwM2NhIGhyZWY9XCJodHRwOi8vb3BlbnN0cmVldG1hcC5vcmdcIlx1MDAzZU9wZW5TdHJlZXRNYXBcdTAwM2MvYVx1MDAzZSwgdW5kZXIgXHUwMDNjYSBocmVmPVwiaHR0cDovL3d3dy5vcGVuc3RyZWV0bWFwLm9yZy9jb3B5cmlnaHRcIlx1MDAzZU9EYkxcdTAwM2MvYVx1MDAzZS4iLCAiZGV0ZWN0UmV0aW5hIjogZmFsc2UsICJtYXhOYXRpdmVab29tIjogMTgsICJtYXhab29tIjogMTgsICJtaW5ab29tIjogMCwgIm5vV3JhcCI6IGZhbHNlLCAib3BhY2l0eSI6IDEsICJzdWJkb21haW5zIjogImFiYyIsICJ0bXMiOiBmYWxzZX0KICAgICAgICAgICAgKS5hZGRUbyhtYXBfMmNlNzgxOTUzODYzNDE5OGE1Yzg1ODM4OGFjN2JmZmYpOwogICAgICAgIAogICAgCiAgICAgICAgZnVuY3Rpb24gZ2VvX2pzb25fZTY5ODc3YmI0YjcwNDY0ZmFmYTRjMGQ3NDc2N2Y1Nzlfb25FYWNoRmVhdHVyZShmZWF0dXJlLCBsYXllcikgewogICAgICAgICAgICBsYXllci5vbih7CiAgICAgICAgICAgIH0pOwogICAgICAgIH07CiAgICAgICAgdmFyIGdlb19qc29uX2U2OTg3N2JiNGI3MDQ2NGZhZmE0YzBkNzQ3NjdmNTc5ID0gTC5nZW9Kc29uKG51bGwsIHsKICAgICAgICAgICAgICAgIG9uRWFjaEZlYXR1cmU6IGdlb19qc29uX2U2OTg3N2JiNGI3MDQ2NGZhZmE0YzBkNzQ3NjdmNTc5X29uRWFjaEZlYXR1cmUsCiAgICAgICAgICAgIAogICAgICAgIH0pOwoKICAgICAgICBmdW5jdGlvbiBnZW9fanNvbl9lNjk4NzdiYjRiNzA0NjRmYWZhNGMwZDc0NzY3ZjU3OV9hZGQgKGRhdGEpIHsKICAgICAgICAgICAgZ2VvX2pzb25fZTY5ODc3YmI0YjcwNDY0ZmFmYTRjMGQ3NDc2N2Y1NzkKICAgICAgICAgICAgICAgIC5hZGREYXRhKGRhdGEpCiAgICAgICAgICAgICAgICAuYWRkVG8obWFwXzJjZTc4MTk1Mzg2MzQxOThhNWM4NTgzODhhYzdiZmZmKTsKICAgICAgICB9CiAgICAgICAgICAgIGdlb19qc29uX2U2OTg3N2JiNGI3MDQ2NGZhZmE0YzBkNzQ3NjdmNTc5X2FkZCh7ImZlYXR1cmVzIjogW3siZ2VvbWV0cnkiOiB7ImNvb3JkaW5hdGVzIjogW1tbWzQ1Ljg1ODMyNCwgMTAuOTIzNzY1XSwgWzQ1Ljg1ODIyNiwgMTAuOTIzNzM5XSwgWzQ1Ljg1ODE2OSwgMTAuOTIzNzAxXSwgWzQ1Ljg1ODExOCwgMTAuOTIzNjM3XSwgWzQ1Ljg1ODEwNiwgMTAuOTIzNTkxXSwgWzQ1Ljg1ODA4MywgMTAuOTIzNTM3XSwgWzQ1Ljg1ODA2NCwgMTAuOTIzNDI5XSwgWzQ1Ljg1ODA1NCwgMTAuOTIzMzI3XSwgWzQ1Ljg1ODA1MywgMTAuOTIzMjA5XSwgWzQ1Ljg1ODA0OSwgMTAuOTIzMDc4XSwgWzQ1Ljg1ODA0MiwgMTAuOTIyOTAxXSwgWzQ1Ljg1ODAyNywgMTAuOTIyNzEzXSwgWzQ1Ljg1Nzk4MiwgMTAuOTIyMjNdLCBbNDUuODU3OTcsIDEwLjkyMTk1OF0sIFs0NS44NTc5NTIsIDEwLjkyMTc5N10sIFs0NS44NTc4ODgsIDEwLjkyMTY1MV0sIFs0NS44NTc4MjgsIDEwLjkyMTU2XSwgWzQ1Ljg1Nzc2OCwgMTAuOTIxNDM0XSwgWzQ1Ljg1NzczOCwgMTAuOTIxMjc5XSwgWzQ1Ljg1Nzc1LCAxMC45MjExMzldLCBbNDUuODU3Nzg2LCAxMC45MjA5OTZdLCBbNDUuODU3ODI2LCAxMC45MjA5MDldLCBbNDUuODU3OTA0LCAxMC45MjA4XSwgWzQ1Ljg1Nzk5OSwgMTAuOTIwNjkxXSwgWzQ1Ljg1ODE1MywgMTAuOTIwNTU1XSwgWzQ1Ljg1ODI1NSwgMTAuOTIwNDY3XSwgWzQ1Ljg1ODM0LCAxMC45MjA0NV0sIFs0NS44NTg0NDksIDEwLjkyMDQ3XSwgWzQ1Ljg1ODU1MSwgMTAuOTIwNTY1XSwgWzQ1Ljg1ODU4OSwgMTAuOTIwNjUxXSwgWzQ1Ljg1ODYwNiwgMTAuOTIwNzFdLCBbNDUuODU4NjUxLCAxMC45MjA3NjRdLCBbNDUuODU4NzU4LCAxMC45MjA4MjNdLCBbNDUuODU4ODg1LCAxMC45MjA4N10sIFs0NS44NTkwMTUsIDEwLjkyMDldLCBbNDUuODU5MTAyLCAxMC45MjA5MDNdLCBbNDUuODU5MTkyLCAxMC45MjA4ODZdLCBbNDUuODU5MzMsIDEwLjkyMDgxOF0sIFs0NS44NTkzNzgsIDEwLjkyMDgyM10sIFs0NS44NTk0OSwgMTAuOTIwODYzXSwgWzQ1Ljg1OTU4MywgMTAuOTIwNzYxXSwgWzQ1Ljg1OTU5NywgMTAuOTIwNTc4XSwgWzQ1Ljg1OTU4NywgMTAuOTIwNTI1XSwgWzQ1Ljg1OTU3NywgMTAuOTIwNDQ2XSwgWzQ1Ljg1OTU3NSwgMTAuOTIwMzg0XSwgWzQ1Ljg1OTYwNiwgMTAuOTIwMjk3XSwgWzQ1Ljg1OTYyMiwgMTAuOTIwMjEyXSwgWzQ1Ljg1OTYzLCAxMC45MjAwOTddLCBbNDUuODU5NjE4LCAxMC45MjAwMjJdLCBbNDUuODU5NjAxLCAxMC45MTk5NDldLCBbNDUuODU5NjA0LCAxMC45MTk4NjRdLCBbNDUuODU5NjE5LCAxMC45MTk4MTJdLCBbNDUuODU5NjUxLCAxMC45MTk3ODRdLCBbNDUuODU5NjksIDEwLjkxOTc4NV0sIFs0NS44NTk3NTksIDEwLjkxOThdLCBbNDUuODU5ODMzLCAxMC45MTk4NjhdLCBbNDUuODU5OTE5LCAxMC45MTk5NjZdLCBbNDUuODU5OTc3LCAxMC45MjAwMzNdLCBbNDUuODYwMDUsIDEwLjkyMDA4OF0sIFs0NS44NjAxMDIsIDEwLjkyMDEyMl0sIFs0NS44NjAxNTksIDEwLjkyMDExNF0sIFs0NS44NjAyMTIsIDEwLjkyMDA3XSwgWzQ1Ljg2MDI0MSwgMTAuOTIwMDAyXSwgWzQ1Ljg2MDI0NCwgMTAuOTE5OTMzXSwgWzQ1Ljg2MDIyNSwgMTAuOTE5ODQ0XSwgWzQ1Ljg2MDE4OCwgMTAuOTE5NzM1XSwgWzQ1Ljg2MDE1MywgMTAuOTE5NjUyXSwgWzQ1Ljg2MDEyNCwgMTAuOTE5NTc2XSwgWzQ1Ljg2MDA5OCwgMTAuOTE5NDY0XSwgWzQ1Ljg2MDEwMSwgMTAuOTE5NDI0XSwgWzQ1Ljg2MDEzMSwgMTAuOTE5NDEyXSwgWzQ1Ljg2MDE3NywgMTAuOTE5NDNdLCBbNDUuODYwMjA2LCAxMC45MTk0NTddLCBbNDUuODYwMjQsIDEwLjkxOTUwNF0sIFs0NS44NjAyNzEsIDEwLjkxOTU3MV0sIFs0NS44NjAzMDQsIDEwLjkxOTY1M10sIFs0NS44NjAzNTIsIDEwLjkxOTc2M10sIFs0NS44NjA0MDQsIDEwLjkxOTg2XSwgWzQ1Ljg2MDQ1MywgMTAuOTE5OTNdLCBbNDUuODYwNDkxLCAxMC45MTk5NTRdLCBbNDUuODYwNTY3LCAxMC45MTk5MjFdLCBbNDUuODYwNTk2LCAxMC45MTk4ODJdLCBbNDUuODYwNjM0LCAxMC45MTk3NzldLCBbNDUuODYwNjQ1LCAxMC45MTk2NjhdLCBbNDUuODYwNjQ0LCAxMC45MTk1NV0sIFs0NS44NjA3MTcsIDEwLjkxOTI5XSwgWzQ1Ljg2MDc3OCwgMTAuOTE5MTg0XSwgWzQ1Ljg2MDgxNiwgMTAuOTE5MTA2XSwgWzQ1Ljg2MDg0NywgMTAuOTE5MDM4XSwgWzQ1Ljg2MDg4NSwgMTAuOTE4OTMxXSwgWzQ1Ljg2MDkwMywgMTAuOTE4ODE3XSwgWzQ1Ljg2MDkwNSwgMTAuOTE4Njk5XSwgWzQ1Ljg2MDkwNSwgMTAuOTE4NTg4XSwgWzQ1Ljg2MDkzMiwgMTAuOTE4NDcxXSwgWzQ1Ljg2MDk1OSwgMTAuOTE4MzQ0XSwgWzQ1Ljg2MDk4MywgMTAuOTE4MjVdLCBbNDUuODYxMDUxLCAxMC45MTgxMzddLCBbNDUuODYxMjA1LCAxMC45MTgwMDFdLCBbNDUuODYxMzUsIDEwLjkxNzkxN10sIFs0NS44NjE0OTgsIDEwLjkxNzgxNF0sIFs0NS44NjE2NDYsIDEwLjkxNzczN10sIFs0NS44NjE3OTIsIDEwLjkxNzY1Nl0sIFs0NS44NjIwMzIsIDEwLjkxNzUyM10sIFs0NS44NjIyMDMsIDEwLjkxNzQyN10sIFs0NS44NjI0MDIsIDEwLjkxNzMyOF0sIFs0NS44NjI1ODIsIDEwLjkxNzI0OV0sIFs0NS44NjI4MjQsIDEwLjkxNzIxNF0sIFs0NS44NjI5NzYsIDEwLjkxNzE5Ml0sIFs0NS44NjMxODQsIDEwLjkxNzIxMl0sIFs0NS44NjMzNjUsIDEwLjkxNzIzNF0sIFs0NS44NjM1MDUsIDEwLjkxNzI0NV0sIFs0NS44NjM2NDcsIDEwLjkxNzIzM10sIFs0NS44NjM4MjIsIDEwLjkxNzE4M10sIFs0NS44NjM5NjcsIDEwLjkxNzExMl0sIFs0NS44NjQwOTksIDEwLjkxNzAxOF0sIFs0NS44NjQyMzQsIDEwLjkxNjg5NF0sIFs0NS44NjQyOTQsIDEwLjkxNjgzNF0sIFs0NS44NjQ0MDksIDEwLjkxNjcwN10sIFs0NS44NjQ1LCAxMC45MTY1ODJdLCBbNDUuODY0NTc3LCAxMC45MTY1MDldLCBbNDUuODY0NjUyLCAxMC45MTY0OTJdLCBbNDUuODY0NzMsIDEwLjkxNjUxNF0sIFs0NS44NjQ3NTQsIDEwLjkxNjU1N10sIFs0NS44NjQ3NjcsIDEwLjkxNjYyXSwgWzQ1Ljg2NDc1MywgMTAuOTE2Njc4XSwgWzQ1Ljg2NDY5OCwgMTAuOTE2Nzc1XSwgWzQ1Ljg2NDYzOCwgMTAuOTE2ODU1XSwgWzQ1Ljg2NDYsIDEwLjkxNjg5XSwgWzQ1Ljg2NDQ2MSwgMTAuOTE3MDEzXSwgWzQ1Ljg2NDM2OCwgMTAuOTE3MTEyXSwgWzQ1Ljg2NDI4NCwgMTAuOTE3MjE0XSwgWzQ1Ljg2NDE5OSwgMTAuOTE3MzMyXSwgWzQ1Ljg2NDE2NiwgMTAuOTE3NDIzXSwgWzQ1Ljg2NDE1OCwgMTAuOTE3NTA4XSwgWzQ1Ljg2NDIyNSwgMTAuOTE3NzU5XSwgWzQ1Ljg2NDI2MywgMTAuOTE3ODIzXSwgWzQ1Ljg2NDMzMiwgMTAuOTE3OTA3XSwgWzQ1Ljg2NDQxNCwgMTAuOTE3OTc4XSwgWzQ1Ljg2NDQ4OCwgMTAuOTE4MDM2XSwgWzQ1Ljg2NDU4OCwgMTAuOTE4MDc5XSwgWzQ1Ljg2NDY4OSwgMTAuOTE4MDk4XSwgWzQ1Ljg2NDgxNSwgMTAuOTE4MDgzXSwgWzQ1Ljg2NDk1MywgMTAuOTE4MDYxXSwgWzQ1Ljg2NTA3NSwgMTAuOTE3OTk5XSwgWzQ1Ljg2NTIyMSwgMTAuOTE3OTA5XSwgWzQ1Ljg2NTQwNiwgMTAuOTE3OF0sIFs0NS44NjU1ODQsIDEwLjkxNzY4N10sIFs0NS44NjU3NTYsIDEwLjkxNzU1Ml0sIFs0NS44NjU5MTYsIDEwLjkxNzM4N10sIFs0NS44NjYwNTQsIDEwLjkxNzIwMV0sIFs0NS44NjYxMDksIDEwLjkxNzA2Ml0sIFs0NS44NjYxMzIsIDEwLjkxNjkzNV0sIFs0NS44NjYxMjksIDEwLjkxNjgxM10sIFs0NS44NjYwOTQsIDEwLjkxNjcyMV0sIFs0NS44NjYwMDQsIDEwLjkxNjYyOV0sIFs0NS44NjU5MTEsIDEwLjkxNjU0NF0sIFs0NS44NjU4MTQsIDEwLjkxNjQ2M10sIFs0NS44NjU3MTcsIDEwLjkxNjM5NF0sIFs0NS44NjU1OTYsIDEwLjkxNjMzMV0sIFs0NS44NjU0NiwgMTAuOTE2Mjc4XSwgWzQ1Ljg2NTMwOSwgMTAuOTE2Mjc2XSwgWzQ1Ljg2NTIwMywgMTAuOTE2MjZdLCBbNDUuODY1MTUxLCAxMC45MTYyMTldLCBbNDUuODY1MTUsIDEwLjkxNjE1XSwgWzQ1Ljg2NTE2NywgMTAuOTE2MTA4XSwgWzQ1Ljg2NTIxOCwgMTAuOTE2MDQxXSwgWzQ1Ljg2NTI4MSwgMTAuOTE1OTc3XSwgWzQ1Ljg2NTM2MywgMTAuOTE1ODkxXSwgWzQ1Ljg2NTQ0LCAxMC45MTU4MDJdLCBbNDUuODY1NTM1LCAxMC45MTU3MV0sIFs0NS44NjU1ODQsIDEwLjkxNTY0OV0sIFs0NS44NjU2NzksIDEwLjkxNTZdLCBbNDUuODY1NzQsIDEwLjkxNTU3NF0sIFs0NS44NjU4MSwgMTAuOTE1NTVdLCBbNDUuODY1OTA3LCAxMC45MTU0ODFdLCBbNDUuODY2MDA3LCAxMC45MTU0MDVdLCBbNDUuODY2MTIzLCAxMC45MTUzMTddLCBbNDUuODY2MjI5LCAxMC45MTUyNDVdLCBbNDUuODY2MzY2LCAxMC45MTUxNjddLCBbNDUuODY2NTA0LCAxMC45MTUxMTZdLCBbNDUuODY2NjIxLCAxMC45MTUwODddLCBbNDUuODY2NzU3LCAxMC45MTUwNzJdLCBbNDUuODY2OTAxLCAxMC45MTUwNl0sIFs0NS44NjcwMjksIDEwLjkxNTA1NF0sIFs0NS44NjcxNjUsIDEwLjkxNTA0NV0sIFs0NS44NjcyOTMsIDEwLjkxNTAzOV0sIFs0NS44Njc0MjksIDEwLjkxNTAxN10sIFs0NS44Njc1OCwgMTAuOTE0OThdLCBbNDUuODY3NzM0LCAxMC45MTQ5MzldLCBbNDUuODY3ODYxLCAxMC45MTQ4ODddLCBbNDUuODY3OTg5LCAxMC45MTQ3NzNdLCBbNDUuODY4MDkyLCAxMC45MTQ2MTJdLCBbNDUuODY4MTk2LCAxMC45MTQ0MjJdLCBbNDUuODY4MjYsIDEwLjkxNDIzMV0sIFs0NS44NjgyNzksIDEwLjkxNDA4NF0sIFs0NS44NjgzMSwgMTAuOTEzODQ2XSwgWzQ1Ljg2ODMyLCAxMC45MTM2NDNdLCBbNDUuODY4MzIxLCAxMC45MTM0MTRdLCBbNDUuODY4MzMxLCAxMC45MTMyMTFdLCBbNDUuODY4MzYxLCAxMC45MTMwNThdLCBbNDUuODY4NDA0LCAxMC45MTI5NTRdLCBbNDUuODY4NDg3LCAxMC45MTI4NzJdLCBbNDUuODY4NTYzLCAxMC45MTI4MzFdLCBbNDUuODY4NjUzLCAxMC45MTI4MzRdLCBbNDUuODY4NzI4LCAxMC45MTI4NzZdLCBbNDUuODY4ODE2LCAxMC45MTI5NjddLCBbNDUuODY4ODkyLCAxMC45MTMwNjFdLCBbNDUuODY4OTY0LCAxMC45MTMxNTJdLCBbNDUuODY5MDM4LCAxMC45MTMyNDNdLCBbNDUuODY5MTE1LCAxMC45MTMzMzddLCBbNDUuODY5MTkxLCAxMC45MTM0MzFdLCBbNDUuODY5MjMzLCAxMC45MTM1MThdLCBbNDUuODY5MjU5LCAxMC45MTM2MTRdLCBbNDUuODY5MjYsIDEwLjkxMzcxNV0sIFs0NS44NjkyNTIsIDEwLjkxMzgwM10sIFs0NS44NjkyNDgsIDEwLjkxMzkwMl0sIFs0NS44NjkyNDQsIDEwLjkxNDAyM10sIFs0NS44NjkyNDYsIDEwLjkxNDE2NF0sIFs0NS44NjkyMzcsIDEwLjkxNDQ1NV0sIFs0NS44NjkyMjMsIDEwLjkxNDYxOV0sIFs0NS44NjkyMDMsIDEwLjkxNDczOV0sIFs0NS44NjkyLCAxMC45MTQ5NzJdLCBbNDUuODY5MjA0LCAxMC45MTUxNjJdLCBbNDUuODY5MTk0LCAxMC45MTUzMzVdLCBbNDUuODY5MTc2LCAxMC45MTU0NzldLCBbNDUuODY5MTc1LCAxMC45MTU2NzZdLCBbNDUuODY5MTk5LCAxMC45MTU3NjNdLCBbNDUuODY5MjE2LCAxMC45MTU4MzldLCBbNDUuODY5MjQ2LCAxMC45MTU5NDJdLCBbNDUuODY5Mjk1LCAxMC45MTYwNTJdLCBbNDUuODY5MzIyLCAxMC45MTYxMDJdLCBbNDUuODY5MzYsIDEwLjkxNjE0OV0sIFs0NS44NjkzOTYsIDEwLjkxNjE3OV0sIFs0NS44Njk1MDEsIDEwLjkxNjIyMl0sIFs0NS44Njk1ODcsIDEwLjkxNjIzNF0sIFs0NS44Njk2NTksIDEwLjkxNjE4OF0sIFs0NS44Njk3NDMsIDEwLjkxNjA5NV0sIFs0NS44Njk4MzEsIDEwLjkxNjAxNl0sIFs0NS44Njk4OTgsIDEwLjkxNjAwMl0sIFs0NS44Njk5NTMsIDEwLjkxNjAxN10sIFs0NS44NzAwMTYsIDEwLjkxNjA3MV0sIFs0NS44NzAwNywgMTAuOTE2MTU1XSwgWzQ1Ljg3MDA4OSwgMTAuOTE2MjI0XSwgWzQ1Ljg3MDEwMSwgMTAuOTE2MzI5XSwgWzQ1Ljg3MDA5MSwgMTAuOTE2NDI0XSwgWzQ1Ljg3MDA5NiwgMTAuOTE2NTM5XSwgWzQ1Ljg3MDEyNSwgMTAuOTE2NjgxXSwgWzQ1Ljg3MDE2NSwgMTAuOTE2ODAzXSwgWzQ1Ljg3MDIyMywgMTAuOTE2OTFdLCBbNDUuODcwMjc0LCAxMC45MTY5NjddLCBbNDUuODcwMzQyLCAxMC45MTcwNDhdLCBbNDUuODcwMzU3LCAxMC45MTcxMTRdLCBbNDUuODcwMzQ2LCAxMC45MTcyMjhdLCBbNDUuODcwMjYyLCAxMC45MTc0NjhdLCBbNDUuODcwMTcxLCAxMC45MTc1OTZdLCBbNDUuODcwMDk2LCAxMC45MTc2NjZdLCBbNDUuODcwMDEzLCAxMC45MTc3MTldLCBbNDUuODY5OTMyLCAxMC45MTc3NjZdLCBbNDUuODY5ODQ0LCAxMC45MTc4MDldLCBbNDUuODY5NzUyLCAxMC45MTc4MzldLCBbNDUuODY5NjQ3LCAxMC45MTc4MzldLCBbNDUuODY5NTQ4LCAxMC45MTc4MzJdLCBbNDUuODY5NDc3LCAxMC45MTc4NDNdLCBbNDUuODY5NDA1LCAxMC45MTc5NDJdLCBbNDUuODY5MzYzLCAxMC45MTgxMDJdLCBbNDUuODY5MzI0LCAxMC45MTgyNTRdLCBbNDUuODY5Mjc2LCAxMC45MTgzODRdLCBbNDUuODY5MjIyLCAxMC45MTg0NjddLCBbNDUuODY5MTQ2LCAxMC45MTg1MTFdLCBbNDUuODY5MDU3LCAxMC45MTg1MDJdLCBbNDUuODY4OTM0LCAxMC45MTg0NzVdLCBbNDUuODY4ODM4LCAxMC45MTg0NjJdLCBbNDUuODY4Nzg1LCAxMC45MTg0N10sIFs0NS44Njg3NDMsIDEwLjkxODUyNF0sIFs0NS44Njg3NTMsIDEwLjkxODZdLCBbNDUuODY4OCwgMTAuOTE4NjldLCBbNDUuODY4ODUyLCAxMC45MTg3NDddLCBbNDUuODY4OTE5LCAxMC45MTg3OTVdLCBbNDUuODY4OTgsIDEwLjkxODg1M10sIFs0NS44Njg5OTMsIDEwLjkxODkwOV0sIFs0NS44Njg5ODcsIDEwLjkxODk5MV0sIFs0NS44Njg5NzUsIDEwLjkxOTAzXSwgWzQ1Ljg2ODkzNywgMTAuOTE5MTA0XSwgWzQ1Ljg2ODg0OSwgMTAuOTE5MTldLCBbNDUuODY4NzMzLCAxMC45MTkzMDFdLCBbNDUuODY4NTY4LCAxMC45MTk0NTZdLCBbNDUuODY4NDU4LCAxMC45MTk1NjddLCBbNDUuODY4MzE3LCAxMC45MTk2ODFdLCBbNDUuODY4MjA3LCAxMC45MTk3MDddLCBbNDUuODY4MDY5LCAxMC45MTk2ODldLCBbNDUuODY3OTU2LCAxMC45MTk2NDNdLCBbNDUuODY3ODgxLCAxMC45MTk2MDFdLCBbNDUuODY3ODA4LCAxMC45MTk1OTJdLCBbNDUuODY3NzQxLCAxMC45MTk2MDNdLCBbNDUuODY3Njc4LCAxMC45MTk2NTFdLCBbNDUuODY3NjI1LCAxMC45MTk2OThdLCBbNDUuODY3NTQ2LCAxMC45MTk3OTFdLCBbNDUuODY3NDQzLCAxMC45MTk5MjVdLCBbNDUuODY3MzIzLCAxMC45MjAxMDVdLCBbNDUuODY3MjA4LCAxMC45MjAyNzFdLCBbNDUuODY3MDU4LCAxMC45MjA0ODldLCBbNDUuODY2ODg3LCAxMC45MjA3NTNdLCBbNDUuODY2NzA2LCAxMC45MjEwMzldLCBbNDUuODY2NTcyLCAxMC45MjEyNDddLCBbNDUuODY2NDUsIDEwLjkyMTQ1Nl0sIFs0NS44NjYzNDEsIDEwLjkyMTY4OV0sIFs0NS44NjYyMTEsIDEwLjkyMTk1XSwgWzQ1Ljg2NjAzMywgMTAuOTIyMTc3XSwgWzQ1Ljg2NTg2MiwgMTAuOTIyMzAzXSwgWzQ1Ljg2NTY4OSwgMTAuOTIyMzZdLCBbNDUuODY1NTQ3LCAxMC45MjIzNzVdLCBbNDUuODY1Mzk2LCAxMC45MjIzNjddLCBbNDUuODY1MjE3LCAxMC45MjIzMzVdLCBbNDUuODY1MDU1LCAxMC45MjIyOTddLCBbNDUuODY0OTcxLCAxMC45MjIyODRdLCBbNDUuODY0ODc3LCAxMC45MjIyODJdLCBbNDUuODY0NzYyLCAxMC45MjIyOTRdLCBbNDUuODY0NTEsIDEwLjkyMjM1Ml0sIFs0NS44NjQzMTIsIDEwLjkyMjM3NV0sIFs0NS44NjQxMDIsIDEwLjkyMjM1OV0sIFs0NS44NjM5MTksIDEwLjkyMjMyN10sIFs0NS44NjM3NDcsIDEwLjkyMjMzNF0sIFs0NS44NjM1OTEsIDEwLjkyMjM2Ml0sIFs0NS44NjM0NiwgMTAuOTIyNDA0XSwgWzQ1Ljg2MzMzMywgMTAuOTIyNDU5XSwgWzQ1Ljg2MzE2OSwgMTAuOTIyNTIyXSwgWzQ1Ljg2MzA2OCwgMTAuOTIyNTM4XSwgWzQ1Ljg2Mjk5NywgMTAuOTIyNTVdLCBbNDUuODYyODQ2LCAxMC45MjI1NjFdLCBbNDUuODYyNzYzLCAxMC45MjI1NTldLCBbNDUuODYyNTgxLCAxMC45MjI1MjddLCBbNDUuODYyNDIxLCAxMC45MjI1MDJdLCBbNDUuODYyMjQ5LCAxMC45MjI1XSwgWzQ1Ljg2MjA3MiwgMTAuOTIyNTI3XSwgWzQ1Ljg2MTg4NiwgMTAuOTIyNTY3XSwgWzQ1Ljg2MTcxNiwgMTAuOTIyNTg4XSwgWzQ1Ljg2MTU2NywgMTAuOTIyNTldLCBbNDUuODYxNDIzLCAxMC45MjI2MTFdLCBbNDUuODYxMjY0LCAxMC45MjI2ODJdLCBbNDUuODYxMTUzLCAxMC45MjI3NDRdLCBbNDUuODYxMDY0LCAxMC45MjI4MTldLCBbNDUuODYwOTY2LCAxMC45MjI5ODNdLCBbNDUuODYwOTEyLCAxMC45MjMwMjRdLCBbNDUuODYwODI1LCAxMC45MjMwMzhdLCBbNDUuODYwNzI3LCAxMC45MjMwMTVdLCBbNDUuODYwNTA2LCAxMC45MjI5MjZdLCBbNDUuODYwNDA4LCAxMC45MjI4ODRdLCBbNDUuODYwMjcsIDEwLjkyMjgyNF0sIFs0NS44NjAxNDIsIDEwLjkyMjc4XSwgWzQ1Ljg1OTk2OSwgMTAuOTIyNzQyXSwgWzQ1Ljg1OTgxMSwgMTAuOTIyNzA0XSwgWzQ1Ljg1OTY4MSwgMTAuOTIyNjc3XSwgWzQ1Ljg1OTYwOCwgMTAuOTIyNjgyXSwgWzQ1Ljg1OTQ3NCwgMTAuOTIyNzRdLCBbNDUuODU5MzQ5LCAxMC45MjI4MzFdLCBbNDUuODU5MjUxLCAxMC45MjI5MzJdLCBbNDUuODU5MTY5LCAxMC45MjMwMjFdLCBbNDUuODU4OTk0LCAxMC45MjMyNTJdLCBbNDUuODU4OTUyLCAxMC45MjMzMTldLCBbNDUuODU4ODc5LCAxMC45MjM0NDFdLCBbNDUuODU4NzgsIDEwLjkyMzYwNV0sIFs0NS44NTg2OCwgMTAuOTIzNzAxXSwgWzQ1Ljg1ODU2OSwgMTAuOTIzNzUzXSwgWzQ1Ljg1ODQ1MiwgMTAuOTIzNzY5XSwgWzQ1Ljg1ODMyNCwgMTAuOTIzNzY1XV1dXSwgInR5cGUiOiAiTXVsdGlQb2x5Z29uIn0sICJpZCI6ICIwIiwgInByb3BlcnRpZXMiOiB7ImRlc2NyaXppb25lX29yaWdpbmUiOiAiTm9uIGNvZGlmaWNhdG8iLCAiZ21sX2lkIjogIklELkFDUVVFRklTSUNIRS5TUEVDQ0hJLkFDUVVBLjU0MTg2IiwgIm5hc3AiOiAwLCAibmF0dXJhX3NwZWNjaGlvX2FjcXVhIjogIk5vbiBjb2RpZmljYXRvIiwgIm5vbWUiOiAiTEFHTyBESSBMT1BQSU8iLCAib3JpZ2luZSI6IDB9LCAidHlwZSI6ICJGZWF0dXJlIn0sIHsiZ2VvbWV0cnkiOiB7ImNvb3JkaW5hdGVzIjogW1tbWzQ1Ljk4MjQzNiwgMTAuOTIyNDEzXSwgWzQ1Ljk4MjM4LCAxMC45MjIzODVdLCBbNDUuOTgyMzI1LCAxMC45MjIzNTddLCBbNDUuOTgyMjM4LCAxMC45MjIzMzJdLCBbNDUuOTgyMTI0LCAxMC45MjIyNzNdLCBbNDUuOTgyMDI4LCAxMC45MjIyNjRdLCBbNDUuOTgxOTU1LCAxMC45MjIyMjldLCBbNDUuOTgxOTI5LCAxMC45MjIxNzNdLCBbNDUuOTgxOTUsIDEwLjkyMjEyMV0sIFs0NS45ODE5NzgsIDEwLjkyMjA4Ml0sIFs0NS45ODIwMzYsIDEwLjkyMjAwNV0sIFs0NS45ODIwNzEsIDEwLjkyMTk0N10sIFs0NS45ODIyMDgsIDEwLjkyMTgxMl0sIFs0NS45ODIyNjgsIDEwLjkyMTc5MV0sIFs0NS45ODIzNzQsIDEwLjkyMTc2MV0sIFs0NS45ODI0NjMsIDEwLjkyMTc5XSwgWzQ1Ljk4MjU1MiwgMTAuOTIxODI1XSwgWzQ1Ljk4MjY0NywgMTAuOTIxOTEzXSwgWzQ1Ljk4MjcwMSwgMTAuOTIxOTU3XSwgWzQ1Ljk4MjcyMSwgMTAuOTIyMDE3XSwgWzQ1Ljk4Mjc0MSwgMTAuOTIyMDgzXSwgWzQ1Ljk4Mjc0OSwgMTAuOTIyMTQ2XSwgWzQ1Ljk4Mjc0OCwgMTAuOTIyMjA4XSwgWzQ1Ljk4MjcyMiwgMTAuOTIyMjY2XSwgWzQ1Ljk4MjY2OSwgMTAuOTIyMzExXSwgWzQ1Ljk4MjYyNSwgMTAuOTIyMzQ5XSwgWzQ1Ljk4MjU2MywgMTAuOTIyMzY3XSwgWzQ1Ljk4MjQ4NywgMTAuOTIyMzkxXSwgWzQ1Ljk4MjQzNiwgMTAuOTIyNDEzXV1dXSwgInR5cGUiOiAiTXVsdGlQb2x5Z29uIn0sICJpZCI6ICIxIiwgInByb3BlcnRpZXMiOiB7ImRlc2NyaXppb25lX29yaWdpbmUiOiAiTm9uIGNvZGlmaWNhdG8iLCAiZ21sX2lkIjogIklELkFDUVVFRklTSUNIRS5TUEVDQ0hJLkFDUVVBLjU0MTYyIiwgIm5hc3AiOiAwLCAibmF0dXJhX3NwZWNjaGlvX2FjcXVhIjogIk5vbiBjb2RpZmljYXRvIiwgIm5vbWUiOiAiTEFHTyBERUkgQkFHQVRPSSAoRFJPKSIsICJvcmlnaW5lIjogMH0sICJ0eXBlIjogIkZlYXR1cmUifSwgeyJnZW9tZXRyeSI6IHsiY29vcmRpbmF0ZXMiOiBbW1tbNDUuOTc3MTkxLCAxMC45MzE4Nl0sIFs0NS45NzcxNDMsIDEwLjkzMTgyM10sIFs0NS45NzcxNDYsIDEwLjkzMTczN10sIFs0NS45NzcxNzQsIDEwLjkzMTcwMl0sIFs0NS45NzcxOTUsIDEwLjkzMTY3M10sIFs0NS45NzcxOTksIDEwLjkzMTU5NF0sIFs0NS45NzcxNzcsIDEwLjkzMTUzNV0sIFs0NS45NzcxNTIsIDEwLjkzMTQ3NV0sIFs0NS45NzcwODksIDEwLjkzMTQyMV0sIFs0NS45NzcwNTEsIDEwLjkzMTM2NF0sIFs0NS45NzcwMTcsIDEwLjkzMTI4MV0sIFs0NS45NzcsIDEwLjkzMTE4OF0sIFs0NS45NzcwMDksIDEwLjkzMTA4XSwgWzQ1Ljk3NzA1NywgMTAuOTMxMDI5XSwgWzQ1Ljk3NzEzMSwgMTAuOTMxMDI0XSwgWzQ1Ljk3NzE1MSwgMTAuOTMxMDUxXSwgWzQ1Ljk3NzIxNywgMTAuOTMxMTAyXSwgWzQ1Ljk3NzI3OCwgMTAuOTMxMTU2XSwgWzQ1Ljk3NzMwNSwgMTAuOTMxMTk3XSwgWzQ1Ljk3NzM2NiwgMTAuOTMxMjUxXSwgWzQ1Ljk3NzM4OCwgMTAuOTMxMjg0XSwgWzQ1Ljk3NzQyMiwgMTAuOTMxMzIxXSwgWzQ1Ljk3NzQ3MiwgMTAuOTMxMzY1XSwgWzQ1Ljk3NzUwOSwgMTAuOTMxMzYzXSwgWzQ1Ljk3NzYzNiwgMTAuOTMxMzA0XSwgWzQ1Ljk3Nzc2NywgMTAuOTMxMjM5XSwgWzQ1Ljk3Nzg5MiwgMTAuOTMxMThdLCBbNDUuOTc3OTY3LCAxMC45MzExNTldLCBbNDUuOTc3OTkyLCAxMC45MzExOTldLCBbNDUuOTc3OTgyLCAxMC45MzEyMzhdLCBbNDUuOTc3OTU3LCAxMC45MzEyODNdLCBbNDUuOTc3OTI5LCAxMC45MzEzMTldLCBbNDUuOTc3ODU3LCAxMC45MzEzNzldLCBbNDUuOTc3ODEzLCAxMC45MzE0MTRdLCBbNDUuOTc3NzUzLCAxMC45MzE0NjVdLCBbNDUuOTc3Njg4LCAxMC45MzE1MTJdLCBbNDUuOTc3NTk1LCAxMC45MzE1NzJdLCBbNDUuOTc3NTEsIDEwLjkzMTYyNl0sIFs0NS45Nzc0MiwgMTAuOTMxNjk5XSwgWzQ1Ljk3NzI5OSwgMTAuOTMxNzg0XSwgWzQ1Ljk3NzE5MSwgMTAuOTMxODZdXV1dLCAidHlwZSI6ICJNdWx0aVBvbHlnb24ifSwgImlkIjogIjIiLCAicHJvcGVydGllcyI6IHsiZGVzY3JpemlvbmVfb3JpZ2luZSI6ICJOb24gY29kaWZpY2F0byIsICJnbWxfaWQiOiAiSUQuQUNRVUVGSVNJQ0hFLlNQRUNDSEkuQUNRVUEuNTQxNjQiLCAibmFzcCI6IDAsICJuYXR1cmFfc3BlY2NoaW9fYWNxdWEiOiAiTm9uIGNvZGlmaWNhdG8iLCAibm9tZSI6ICJMQUdPIFNPTE8iLCAib3JpZ2luZSI6IDB9LCAidHlwZSI6ICJGZWF0dXJlIn0sIHsiZ2VvbWV0cnkiOiB7ImNvb3JkaW5hdGVzIjogW1tbWzQ1Ljk5NDU4NSwgMTAuODMwNTAyXSwgWzQ1Ljk5NDQ5NiwgMTAuODMwNDgyXSwgWzQ1Ljk5NDQ2NywgMTAuODMwNDU4XSwgWzQ1Ljk5NDMwNSwgMTAuODMwMTU1XSwgWzQ1Ljk5NDI1NCwgMTAuODMwMDYxXSwgWzQ1Ljk5NDE5MSwgMTAuODI5OTNdLCBbNDUuOTk0MDU4LCAxMC44Mjk3MDRdLCBbNDUuOTkzOTgyLCAxMC44Mjk1ODNdLCBbNDUuOTkzOTEzLCAxMC44Mjk0NzhdLCBbNDUuOTkzODQ5LCAxMC44MjkzODNdLCBbNDUuOTkzNzkxLCAxMC44MjkyNzZdLCBbNDUuOTkzNzQ1LCAxMC44MjkxNjJdLCBbNDUuOTkzNzA0LCAxMC44MjkwMzVdLCBbNDUuOTkzNjc4LCAxMC44Mjg5NTldLCBbNDUuOTkzNTg4LCAxMC44Mjg3NzRdLCBbNDUuOTkzNTczLCAxMC44Mjg3MTFdLCBbNDUuOTkzNjMsIDEwLjgyODY0NF0sIFs0NS45OTM2NjcsIDEwLjgyODYyNl0sIFs0NS45OTM3MTMsIDEwLjgyODYxOF0sIFs0NS45OTM3NTQsIDEwLjgyODYyM10sIFs0NS45OTM4MDIsIDEwLjgyODYzOF0sIFs0NS45OTM4NDUsIDEwLjgyODY2Nl0sIFs0NS45OTM4NzYsIDEwLjgyODcwNF0sIFs0NS45OTM5NzEsIDEwLjgyODg3OV0sIFs0NS45OTQwMjQsIDEwLjgyODk5Nl0sIFs0NS45OTQwNTQsIDEwLjgyOTA4OV0sIFs0NS45OTQwNzMsIDEwLjgyOTE1M10sIFs0NS45OTQxMDQsIDEwLjgyOTIzM10sIFs0NS45OTQxNDgsIDEwLjgyOTMzM10sIFs0NS45OTQxOCwgMTAuODI5NDMzXSwgWzQ1Ljk5NDI1LCAxMC44Mjk2MTRdLCBbNDUuOTk0MzA1LCAxMC44Mjk3MjVdLCBbNDUuOTk0Mzk4LCAxMC44Mjk4OV0sIFs0NS45OTQ0NTcsIDEwLjgzMDAwN10sIFs0NS45OTQ0OTksIDEwLjgzMDA5NF0sIFs0NS45OTQ1MzEsIDEwLjgzMDExOV0sIFs0NS45OTQ1NDUsIDEwLjgzMDExMV0sIFs0NS45OTQ1NywgMTAuODMwMDk3XSwgWzQ1Ljk5NDU2NiwgMTAuODMwMDU4XSwgWzQ1Ljk5NDU2MywgMTAuODMwMDA4XSwgWzQ1Ljk5NDU0OSwgMTAuODI5OTE2XSwgWzQ1Ljk5NDUyMSwgMTAuODI5ODEzXSwgWzQ1Ljk5NDQ5NywgMTAuODI5NzU2XSwgWzQ1Ljk5NDQ2LCAxMC44Mjk2NzJdLCBbNDUuOTk0NDEzLCAxMC44Mjk1NzhdLCBbNDUuOTk0MzU2LCAxMC44Mjk0NV0sIFs0NS45OTQzMDgsIDEwLjgyOTMzN10sIFs0NS45OTQyNjIsIDEwLjgyOTIzNl0sIFs0NS45OTQyMDcsIDEwLjgyOTEyNV0sIFs0NS45OTQxNTQsIDEwLjgyOTAyMV0sIFs0NS45OTQxMSwgMTAuODI4OTQ3XSwgWzQ1Ljk5NDA1NCwgMTAuODI4ODQ2XSwgWzQ1Ljk5NDAxNywgMTAuODI4Nzc1XSwgWzQ1Ljk5Mzk5MiwgMTAuODI4NzI4XSwgWzQ1Ljk5Mzk4NCwgMTAuODI4Njc1XSwgWzQ1Ljk5NDAxMywgMTAuODI4NjQ0XSwgWzQ1Ljk5NDA1NCwgMTAuODI4NjI5XSwgWzQ1Ljk5NDEwNywgMTAuODI4NjE4XSwgWzQ1Ljk5NDE1NSwgMTAuODI4NjE3XSwgWzQ1Ljk5NDE5NCwgMTAuODI4NjM4XSwgWzQ1Ljk5NDIzMiwgMTAuODI4Njg2XSwgWzQ1Ljk5NDI3NiwgMTAuODI4NzczXSwgWzQ1Ljk5NDM0NCwgMTAuODI4OTQzXSwgWzQ1Ljk5NDQwMywgMTAuODI5MDc0XSwgWzQ1Ljk5NDQ0OSwgMTAuODI5MTc1XSwgWzQ1Ljk5NDUxNywgMTAuODI5MzAzXSwgWzQ1Ljk5NDU2MSwgMTAuODI5NF0sIFs0NS45OTQ2NzQsIDEwLjgyOTczN10sIFs0NS45OTQ3MTcsIDEwLjgyOTg3M10sIFs0NS45OTQ3NTMsIDEwLjgyOTk5M10sIFs0NS45OTQ3ODIsIDEwLjgzMDA3N10sIFs0NS45OTQ3OTEsIDEwLjgzMDE1M10sIFs0NS45OTQ3ODEsIDEwLjgzMDIwNV0sIFs0NS45OTQ3NTgsIDEwLjgzMDIyN10sIFs0NS45OTQ2NjQsIDEwLjgzMDMxOF0sIFs0NS45OTQ2NTksIDEwLjgzMDM3MV0sIFs0NS45OTQ2NSwgMTAuODMwNDM5XSwgWzQ1Ljk5NDYyNiwgMTAuODMwNDk0XSwgWzQ1Ljk5NDU4NSwgMTAuODMwNTAyXV1dXSwgInR5cGUiOiAiTXVsdGlQb2x5Z29uIn0sICJpZCI6ICIzIiwgInByb3BlcnRpZXMiOiB7ImRlc2NyaXppb25lX29yaWdpbmUiOiAiTm9uIGNvZGlmaWNhdG8iLCAiZ21sX2lkIjogIklELkFDUVVFRklTSUNIRS5TUEVDQ0hJLkFDUVVBLjU0MTU5IiwgIm5hc3AiOiAwLCAibmF0dXJhX3NwZWNjaGlvX2FjcXVhIjogIk5vbiBjb2RpZmljYXRvIiwgIm5vbWUiOiAiTEFHTyBUT1JCSUVSQSBESSBGSUFWRVx1MDAyNyAyPyIsICJvcmlnaW5lIjogMH0sICJ0eXBlIjogIkZlYXR1cmUifSwgeyJnZW9tZXRyeSI6IHsiY29vcmRpbmF0ZXMiOiBbW1tbNDUuOTkzNjU3LCAxMC44MzM0MDddLCBbNDUuOTkzNjIxLCAxMC44MzMzNzZdLCBbNDUuOTkzNjAxLCAxMC44MzMzNDZdLCBbNDUuOTkzNTY3LCAxMC44MzMyNjldLCBbNDUuOTkzNTQ1LCAxMC44MzMyMzJdLCBbNDUuOTkzNTE4LCAxMC44MzMyMTVdLCBbNDUuOTkzNDgyLCAxMC44MzMyMDFdLCBbNDUuOTkzNDI0LCAxMC44MzMyMTVdLCBbNDUuOTkzMzk2LCAxMC44MzMyMjFdLCBbNDUuOTkzMzY0LCAxMC44MzMyMTddLCBbNDUuOTkzMzM3LCAxMC44MzMxOTNdLCBbNDUuOTkzMzE2LCAxMC44MzMxMTNdLCBbNDUuOTkzMzE0LCAxMC44MzMwNTddLCBbNDUuOTkzMzIsIDEwLjgzMjk5OF0sIFs0NS45OTMzMzIsIDEwLjgzMjkzOV0sIFs0NS45OTMzNDUsIDEwLjgzMjg4NF0sIFs0NS45OTMzNTUsIDEwLjgzMjgyOF0sIFs0NS45OTMzNjIsIDEwLjgzMjc3OV0sIFs0NS45OTMzNDcsIDEwLjgzMjcwM10sIFs0NS45OTMzMTYsIDEwLjgzMjY2OV0sIFs0NS45OTMyNTksIDEwLjgzMjYyNV0sIFs0NS45OTMyMTYsIDEwLjgzMjU4N10sIFs0NS45OTMxODEsIDEwLjgzMjUzN10sIFs0NS45OTMxNTQsIDEwLjgzMjVdLCBbNDUuOTkzMTM2LCAxMC44MzI0NjZdLCBbNDUuOTkzMTA5LCAxMC44MzI0MjZdLCBbNDUuOTkzMDgyLCAxMC44MzIzNzldLCBbNDUuOTkzMDY1LCAxMC44MzIzNDJdLCBbNDUuOTkzMDQzLCAxMC44MzIzNTJdLCBbNDUuOTkzMDEsIDEwLjgzMjI4Ml0sIFs0NS45OTI5NTcsIDEwLjgzMjE4OF0sIFs0NS45OTI5MjYsIDEwLjgzMjE0N10sIFs0NS45OTI4ODcsIDEwLjgzMjExMl0sIFs0NS45OTI4NDIsIDEwLjgzMjA4MV0sIFs0NS45OTI3OTQsIDEwLjgzMjA2Ml0sIFs0NS45OTI3MjgsIDEwLjgzMjA0N10sIFs0NS45OTI2ODcsIDEwLjgzMjAzMl0sIFs0NS45OTI2NjMsIDEwLjgzMjAxMV0sIFs0NS45OTI2MjUsIDEwLjgzMTkzXSwgWzQ1Ljk5MjYwNiwgMTAuODMxODY0XSwgWzQ1Ljk5MjU4OSwgMTAuODMxODA0XSwgWzQ1Ljk5MjU3MywgMTAuODMxNzddLCBbNDUuOTkyNTExLCAxMC44MzE2NzZdLCBbNDUuOTkyNDk5LCAxMC44MzE0OTFdLCBbNDUuOTkyNTM2LCAxMC44MzE0Nl0sIFs0NS45OTI1ODMsIDEwLjgzMTQyNV0sIFs0NS45OTI2MTcsIDEwLjgzMTQwNF0sIFs0NS45OTI2NjYsIDEwLjgzMTM4Nl0sIFs0NS45OTI2OTgsIDEwLjgzMTQwNF0sIFs0NS45OTI3MzQsIDEwLjgzMTQ0Ml0sIFs0NS45OTI3NjUsIDEwLjgzMTQ5Ml0sIFs0NS45OTI4MTEsIDEwLjgzMTU4XSwgWzQ1Ljk5Mjg2MiwgMTAuODMxNjc3XSwgWzQ1Ljk5MjkyNCwgMTAuODMxNzg4XSwgWzQ1Ljk5Mjk2OCwgMTAuODMxODgyXSwgWzQ1Ljk5MzA3NiwgMTAuODMyMDY4XSwgWzQ1Ljk5MzIyNCwgMTAuODMyMjY0XSwgWzQ1Ljk5MzI2MywgMTAuODMyMzYxXSwgWzQ1Ljk5MzMwMiwgMTAuODMyMzczXSwgWzQ1Ljk5MzM1MSwgMTAuODMyMzY1XSwgWzQ1Ljk5MzM5OSwgMTAuODMyMzYzXSwgWzQ1Ljk5MzQzMiwgMTAuODMyNDE0XSwgWzQ1Ljk5MzQ0OCwgMTAuODMyNDU3XSwgWzQ1Ljk5MzQ3NSwgMTAuODMyNDc3XSwgWzQ1Ljk5MzQ5LCAxMC44MzI0ODZdLCBbNDUuOTkzNTIyLCAxMC44MzI0ODZdLCBbNDUuOTkzNTUzLCAxMC44MzI0ODNdLCBbNDUuOTkzNTkyLCAxMC44MzI0NjhdLCBbNDUuOTkzNjMzLCAxMC44MzI0NjZdLCBbNDUuOTkzNjc2LCAxMC44MzI0OTFdLCBbNDUuOTkzNzA2LCAxMC44MzI1MThdLCBbNDUuOTkzNzM5LCAxMC44MzI1NjhdLCBbNDUuOTkzNzYxLCAxMC44MzI2MThdLCBbNDUuOTkzNzc0LCAxMC44MzI2NThdLCBbNDUuOTkzNzc4LCAxMC44MzI3MTRdLCBbNDUuOTkzNzY4LCAxMC44MzI3NTNdLCBbNDUuOTkzNzQ1LCAxMC44MzI3ODldLCBbNDUuOTkzNzE3LCAxMC44MzI4MTddLCBbNDUuOTkzNjcxLCAxMC44MzI4NDJdLCBbNDUuOTkzNjI5LCAxMC44MzI4OF0sIFs0NS45OTM2MDUsIDEwLjgzMjkxOV0sIFs0NS45OTM2MDIsIDEwLjgzMjk1OF0sIFs0NS45OTM2MjksIDEwLjgzMzAyNV0sIFs0NS45OTM2NjUsIDEwLjgzMzA2Nl0sIFs0NS45OTM2ODcsIDEwLjgzMzA4OV0sIFs0NS45OTM3MjMsIDEwLjgzMzEyN10sIFs0NS45OTM3NSwgMTAuODMzMTddLCBbNDUuOTkzNzc3LCAxMC44MzMyMzRdLCBbNDUuOTkzNzc4LCAxMC44MzMyOTNdLCBbNDUuOTkzNzczLCAxMC44MzMzMzJdLCBbNDUuOTkzNzQyLCAxMC44MzMzN10sIFs0NS45OTM2ODksIDEwLjgzMzQwNV0sIFs0NS45OTM2NTcsIDEwLjgzMzQwN11dXV0sICJ0eXBlIjogIk11bHRpUG9seWdvbiJ9LCAiaWQiOiAiNCIsICJwcm9wZXJ0aWVzIjogeyJkZXNjcml6aW9uZV9vcmlnaW5lIjogIk5vbiBjb2RpZmljYXRvIiwgImdtbF9pZCI6ICJJRC5BQ1FVRUZJU0lDSEUuU1BFQ0NISS5BQ1FVQS41NDE2MCIsICJuYXNwIjogMCwgIm5hdHVyYV9zcGVjY2hpb19hY3F1YSI6ICJOb24gY29kaWZpY2F0byIsICJub21lIjogIkxBR08gVE9SQklFUkEgREkgRklBVkVcdTAwMjcgMz8iLCAib3JpZ2luZSI6IDB9LCAidHlwZSI6ICJGZWF0dXJlIn0sIHsiZ2VvbWV0cnkiOiB7ImNvb3JkaW5hdGVzIjogW1tbWzQ2LjAxMDk3LCAxMC45NTY4MzRdLCBbNDYuMDEwODI2LCAxMC45NTY4MTldLCBbNDYuMDEwNzA1LCAxMC45NTY4MTRdLCBbNDYuMDEwNTQxLCAxMC45NTY3OThdLCBbNDYuMDEwMzQyLCAxMC45NTY3NzhdLCBbNDYuMDEwMTIxLCAxMC45NTY3MzldLCBbNDYuMDA5ODc4LCAxMC45NTY2OTldLCBbNDYuMDA5NjM5LCAxMC45NTY2NjVdLCBbNDYuMDA5Mzg3LCAxMC45NTY2MjJdLCBbNDYuMDA5MTA3LCAxMC45NTY1NjhdLCBbNDYuMDA4ODQyLCAxMC45NTY1MTVdLCBbNDYuMDA4NjExLCAxMC45NTY1MDVdLCBbNDYuMDA4MzcxLCAxMC45NTY0ODRdLCBbNDYuMDA4MTQ3LCAxMC45NTY0NjhdLCBbNDYuMDA4MDc0LCAxMC45NTY0N10sIFs0Ni4wMDc4OCwgMTAuOTU2NDddLCBbNDYuMDA3NzY3LCAxMC45NTY0OTRdLCBbNDYuMDA3NTk4LCAxMC45NTY1MjhdLCBbNDYuMDA3NDM1LCAxMC45NTY1NTVdLCBbNDYuMDA3MjExLCAxMC45NTY1MzhdLCBbNDYuMDA2OTkyLCAxMC45NTY0OTVdLCBbNDYuMDA2NzQzLCAxMC45NTY0NjJdLCBbNDYuMDA2NTc3LCAxMC45NTY0MjNdLCBbNDYuMDA2NDE1LCAxMC45NTYzNjFdLCBbNDYuMDA2MjYzLCAxMC45NTYyNDFdLCBbNDYuMDA2MDk3LCAxMC45NTYwOV0sIFs0Ni4wMDU5NTYsIDEwLjk1NjAxMl0sIFs0Ni4wMDU3OTgsIDEwLjk1NTk1N10sIFs0Ni4wMDU2MjMsIDEwLjk1NTkxMl0sIFs0Ni4wMDU0OTcsIDEwLjk1NTg5NF0sIFs0Ni4wMDUzNjcsIDEwLjk1NTg3OF0sIFs0Ni4wMDUyMzksIDEwLjk1NTg4M10sIFs0Ni4wMDUwOTcsIDEwLjk1NTg4N10sIFs0Ni4wMDQ5NjIsIDEwLjk1NTg3OF0sIFs0Ni4wMDQ4ODQsIDEwLjk1NTg2NF0sIFs0Ni4wMDQ3NzcsIDEwLjk1NTg1M10sIFs0Ni4wMDQ2NjUsIDEwLjk1NTg0NF0sIFs0Ni4wMDQ1MzUsIDEwLjk1NTgzNl0sIFs0Ni4wMDQzOTYsIDEwLjk1NTgwMV0sIFs0Ni4wMDQyMDYsIDEwLjk1NTcyNV0sIFs0Ni4wMDQwMjQsIDEwLjk1NTYzNF0sIFs0Ni4wMDM3OTYsIDEwLjk1NTUwNV0sIFs0Ni4wMDM1ODUsIDEwLjk1NTM2MV0sIFs0Ni4wMDMzOCwgMTAuOTU1MTg3XSwgWzQ2LjAwMzIzNSwgMTAuOTU1MDc2XSwgWzQ2LjAwMzExMiwgMTAuOTU1MDI1XSwgWzQ2LjAwMjkzMiwgMTAuOTU0OTQ2XSwgWzQ2LjAwMjczMywgMTAuOTU0ODc0XSwgWzQ2LjAwMjQzLCAxMC45NTQ3OTFdLCBbNDYuMDAyMjgyLCAxMC45NTQ3MzJdLCBbNDYuMDAyMTY4LCAxMC45NTQ3MTFdLCBbNDYuMDAyMDIyLCAxMC45NTQ2NV0sIFs0Ni4wMDE5MDYsIDEwLjk1NDU3Ml0sIFs0Ni4wMDE4MDQsIDEwLjk1NDQ3NV0sIFs0Ni4wMDE2OTMsIDEwLjk1NDMxNl0sIFs0Ni4wMDE1OCwgMTAuOTU0MTE3XSwgWzQ2LjAwMTQzNywgMTAuOTUzOTE0XSwgWzQ2LjAwMTMyLCAxMC45NTM3NjhdLCBbNDYuMDAxMTQxLCAxMC45NTM1NjVdLCBbNDYuMDAwOTgyLCAxMC45NTMzODJdLCBbNDYuMDAwNzk0LCAxMC45NTMxODhdLCBbNDYuMDAwNjUxLCAxMC45NTMwNDhdLCBbNDYuMDAwNDk5LCAxMC45NTI5MDFdLCBbNDYuMDAwMzk0LCAxMC45NTI4MjRdLCBbNDYuMDAwMjY2LCAxMC45NTI4MTJdLCBbNDYuMDAwMTQsIDEwLjk1Mjg2M10sIFs0Ni4wMDAwMjYsIDEwLjk1Mjg2N10sIFs0NS45OTk4ODgsIDEwLjk1Mjg1NV0sIFs0NS45OTk3NjEsIDEwLjk1Mjg0XSwgWzQ1Ljk5OTY0NiwgMTAuOTUyODA5XSwgWzQ1Ljk5OTU3NCwgMTAuOTUyNzc1XSwgWzQ1Ljk5OTQ4LCAxMC45NTI3NTddLCBbNDUuOTk5NDAyLCAxMC45NTI3NTldLCBbNDUuOTk5MzQsIDEwLjk1Mjc3OF0sIFs0NS45OTkyNCwgMTAuOTUyNzczXSwgWzQ1Ljk5OTE2NCwgMTAuOTUyNzcyXSwgWzQ1Ljk5OTEwOSwgMTAuOTUyNzc5XSwgWzQ1Ljk5OTEwOSwgMTAuOTUyNzkzXSwgWzQ1Ljk5OTA1NCwgMTAuOTUyODA0XSwgWzQ1Ljk5ODk4MiwgMTAuOTUyODQ3XSwgWzQ1Ljk5ODkxLCAxMC45NTI4OTddLCBbNDUuOTk4Nzk5LCAxMC45NTI5NjFdLCBbNDUuOTk4NzI2LCAxMC45NTI5NjVdLCBbNDUuOTk4NjI4LCAxMC45NTI5NTVdLCBbNDUuOTk4NTYzLCAxMC45NTI5ODhdLCBbNDUuOTk4NDY2LCAxMC45NTMwNl0sIFs0NS45OTgzODMsIDEwLjk1MzA4Nl0sIFs0NS45OTgzMDEsIDEwLjk1MzA2NF0sIFs0NS45OTgyMzcsIDEwLjk1MzA0NV0sIFs0NS45OTgxNjYsIDEwLjk1MzA0OF0sIFs0NS45OTgxMDksIDEwLjk1MzA4Ml0sIFs0NS45OTgwMjMsIDEwLjk1MzEyOF0sIFs0NS45OTc5NDgsIDEwLjk1MzEyOV0sIFs0NS45OTc4NzksIDEwLjk1MzEzNl0sIFs0NS45OTc3OTUsIDEwLjk1MzE5OF0sIFs0NS45OTc3MjUsIDEwLjk1MzI4N10sIFs0NS45OTc2NzYsIDEwLjk1MzM0NV0sIFs0NS45OTc1OTksIDEwLjk1MzQzNF0sIFs0NS45OTc1MzgsIDEwLjk1MzQ4N10sIFs0NS45OTc0MzgsIDEwLjk1MzQ5M10sIFs0NS45OTczNDcsIDEwLjk1MzQ1N10sIFs0NS45OTcyNywgMTAuOTUzNDI3XSwgWzQ1Ljk5NzE1OSwgMTAuOTUzMzc3XSwgWzQ1Ljk5NzAwNywgMTAuOTUzMzA5XSwgWzQ1Ljk5NjkwNSwgMTAuOTUzMjU2XSwgWzQ1Ljk5NjcxLCAxMC45NTMxNDNdLCBbNDUuOTk2NTUsIDEwLjk1MzAzNV0sIFs0NS45OTYzNzYsIDEwLjk1MjkxM10sIFs0NS45OTYxNzcsIDEwLjk1Mjc4MV0sIFs0NS45OTYwMDQsIDEwLjk1MjY1OV0sIFs0NS45OTU4MjEsIDEwLjk1MjUzN10sIFs0NS45OTU2NjMsIDEwLjk1MjQxNl0sIFs0NS45OTU1MDUsIDEwLjk1MjI5Ml0sIFs0NS45OTUzMTgsIDEwLjk1MjE0XSwgWzQ1Ljk5NTE1NiwgMTAuOTUxOTg2XSwgWzQ1Ljk5NTAzNiwgMTAuOTUxODQ0XSwgWzQ1Ljk5NDkzMiwgMTAuOTUxNzMyXSwgWzQ1Ljk5NDg0MiwgMTAuOTUxNjkyXSwgWzQ1Ljk5NDc5MSwgMTAuOTUxNjkzXSwgWzQ1Ljk5NDcwNCwgMTAuOTUxNzM2XSwgWzQ1Ljk5NDY0NCwgMTAuOTUxNjA2XSwgWzQ1Ljk5NDYzMiwgMTAuOTUxNTM2XSwgWzQ1Ljk5NDYwOSwgMTAuOTUxNDExXSwgWzQ1Ljk5NDUyMiwgMTAuOTUxMzI4XSwgWzQ1Ljk5NDQzLCAxMC45NTEyMzNdLCBbNDUuOTk0Mzc0LCAxMC45NTExNjVdLCBbNDUuOTk0MzExLCAxMC45NTEwODFdLCBbNDUuOTk0MjM2LCAxMC45NTA5NTNdLCBbNDUuOTk0MiwgMTAuOTUwODcyXSwgWzQ1Ljk5NDE4NSwgMTAuOTUwODM5XSwgWzQ1Ljk5NDEzMSwgMTAuOTUwNzA2XSwgWzQ1Ljk5NDA3MiwgMTAuOTUwNTRdLCBbNDUuOTk0MDA1LCAxMC45NTAzNTNdLCBbNDUuOTkzODgzLCAxMC45NDk5OTFdLCBbNDUuOTkzODM2LCAxMC45NDk4MjhdLCBbNDUuOTkzNzk1LCAxMC45NDk2OTVdLCBbNDUuOTkzNjQyLCAxMC45NDkzMzVdLCBbNDUuOTkzNTgsIDEwLjk0OTI0XSwgWzQ1Ljk5MzU1NSwgMTAuOTQ5MTk3XSwgWzQ1Ljk5MzUyOSwgMTAuOTQ5MTJdLCBbNDUuOTkzNDYxLCAxMC45NDg4ODVdLCBbNDUuOTkzNDUyLCAxMC45NDg3MjddLCBbNDUuOTkzNDcxLCAxMC45NDg1NzZdLCBbNDUuOTkzNDk0LCAxMC45NDg0NDNdLCBbNDUuOTkzNTMzLCAxMC45NDgyMDVdLCBbNDUuOTkzNTIyLCAxMC45NDgwN10sIFs0NS45OTM0ODEsIDEwLjk0Nzg5NF0sIFs0NS45OTM0MDcsIDEwLjk0NzcwMV0sIFs0NS45OTMzNTEsIDEwLjk0NzUzNV0sIFs0NS45OTMzMTEsIDEwLjk0NzM1OV0sIFs0NS45OTMyNywgMTAuOTQ3MTldLCBbNDUuOTkzMjQ5LCAxMC45NDY5OTldLCBbNDUuOTkzMjUxLCAxMC45NDY3ODJdLCBbNDUuOTkzMjU5LCAxMC45NDY1ODldLCBbNDUuOTkzMjY0LCAxMC45NDY0NTFdLCBbNDUuOTkzMjUsIDEwLjk0NjM0Nl0sIFs0NS45OTMyNDgsIDEwLjk0NjIxNF0sIFs0NS45OTMyNTMsIDEwLjk0NjA2XSwgWzQ1Ljk5MzI1OCwgMTAuOTQ1OTAzXSwgWzQ1Ljk5MzI1MiwgMTAuOTQ1NzM1XSwgWzQ1Ljk5MzI0LCAxMC45NDU1MjVdLCBbNDUuOTkzMjI0LCAxMC45NDUzODZdLCBbNDUuOTkzMTcxLCAxMC45NDUxOTRdLCBbNDUuOTkzMDcyLCAxMC45NDQ4NDZdLCBbNDUuOTkyOTk2LCAxMC45NDQ2MzZdLCBbNDUuOTkyODQ4LCAxMC45NDQyMTNdLCBbNDUuOTkyNzcyLCAxMC45NDQwMTddLCBbNDUuOTkyNjc2LCAxMC45NDM3ODZdLCBbNDUuOTkyNjIzLCAxMC45NDM2NjZdLCBbNDUuOTkyNTI3LCAxMC45NDM1MTVdLCBbNDUuOTkyNDQ3LCAxMC45NDM0MDNdLCBbNDUuOTkyMzM1LCAxMC45NDMyODRdLCBbNDUuOTkyMTY2LCAxMC45NDMxNTNdLCBbNDUuOTkyMDUxLCAxMC45NDMwNl0sIFs0NS45OTE4OTgsIDEwLjk0Mjk1OV0sIFs0NS45OTE3NzEsIDEwLjk0Mjg2NV0sIFs0NS45OTE2NjgsIDEwLjk0Mjc3OV0sIFs0NS45OTE1MzksIDEwLjk0MjY4Ml0sIFs0NS45OTE0MTcsIDEwLjk0MjYyMl0sIFs0NS45OTEyNzQsIDEwLjk0MjU2NF0sIFs0NS45OTExNzIsIDEwLjk0MjUyN10sIFs0NS45OTEwNzYsIDEwLjk0MjUwN10sIFs0NS45OTA5OTIsIDEwLjk0MjQ5MV0sIFs0NS45OTA5MjgsIDEwLjk0MjQ2Ml0sIFs0NS45OTA4NywgMTAuOTQyNDFdLCBbNDUuOTkwODQxLCAxMC45NDIzNDRdLCBbNDUuOTkwODMxLCAxMC45NDIyNzRdLCBbNDUuOTkwODI1LCAxMC45NDIyMDVdLCBbNDUuOTkwNzk3LCAxMC45NDIxNDJdLCBbNDUuOTkwNzQ1LCAxMC45NDIwODFdLCBbNDUuOTkwNzA1LCAxMC45NDIwNF0sIFs0NS45OTA2NzQsIDEwLjk0MTk4Nl0sIFs0NS45OTA2NTcsIDEwLjk0MTkyNl0sIFs0NS45OTA2NDksIDEwLjk0MTg1N10sIFs0NS45OTA2NTksIDEwLjk0MTc4OV0sIFs0NS45OTA2NzUsIDEwLjk0MTcxXSwgWzQ1Ljk5MDY4OSwgMTAuOTQxNjU1XSwgWzQ1Ljk5MDcxNCwgMTAuOTQxNTQ1XSwgWzQ1Ljk5MDczMywgMTAuOTQxNDE3XSwgWzQ1Ljk5MDc0NCwgMTAuOTQxMjk5XSwgWzQ1Ljk5MDc1MywgMTAuOTQxMTY1XSwgWzQ1Ljk5MDc3LCAxMC45NDEwMDhdLCBbNDUuOTkwNzg5LCAxMC45NDA4NTVdLCBbNDUuOTkwODE1LCAxMC45NDA2NzJdLCBbNDUuOTkwODM2LCAxMC45NDA1MTJdLCBbNDUuOTkwODU3LCAxMC45NDAzNzhdLCBbNDUuOTkwODgsIDEwLjk0MDI0MV0sIFs0NS45OTA5MDgsIDEwLjk0MDExNF0sIFs0NS45OTA5MjYsIDEwLjk0MDAzXSwgWzQ1Ljk5MDkyNywgMTAuOTQwMDIyXSwgWzQ1Ljk5MDk0MSwgMTAuOTM5OTU4XSwgWzQ1Ljk5MDk2LCAxMC45Mzk5MTldLCBbNDUuOTkwOTgxLCAxMC45Mzk4ODddLCBbNDUuOTkxMDE3LCAxMC45Mzk4ODJdLCBbNDUuOTkxMDQ5LCAxMC45Mzk5MTZdLCBbNDUuOTkxMDY5LCAxMC45Mzk5NTNdLCBbNDUuOTkxMDg4LCAxMC45NF0sIFs0NS45OTEwOTcsIDEwLjk0MDA0Nl0sIFs0NS45OTExMTUsIDEwLjk0MDE0NV0sIFs0NS45OTExNDksIDEwLjk0MDE4Nl0sIFs0NS45OTExODgsIDEwLjk0MDE1NV0sIFs0NS45OTEyMDUsIDEwLjk0MDEwOV0sIFs0NS45OTEyMTEsIDEwLjk0MDAzMV0sIFs0NS45OTEyMiwgMTAuOTM5OTQ2XSwgWzQ1Ljk5MTIyMywgMTAuOTM5ODldLCBbNDUuOTkxMjIyLCAxMC45Mzk4MzhdLCBbNDUuOTkxMjE2LCAxMC45Mzk3ODhdLCBbNDUuOTkxMjA4LCAxMC45Mzk3NDJdLCBbNDUuOTkxMjA1LCAxMC45Mzk2NV0sIFs0NS45OTEyMTUsIDEwLjkzOTU3Ml0sIFs0NS45OTEyNDEsIDEwLjkzOTU0XSwgWzQ1Ljk5MTI5NiwgMTAuOTM5NTE2XSwgWzQ1Ljk5MTMwMywgMTAuOTM5NTEzXSwgWzQ1Ljk5MTM2NywgMTAuOTM5NTYxXSwgWzQ1Ljk5MTM5MywgMTAuOTM5NjA4XSwgWzQ1Ljk5MTQwNCwgMTAuOTM5NjUxXSwgWzQ1Ljk5MTQxNiwgMTAuOTM5NzUzXSwgWzQ1Ljk5MTQxOSwgMTAuOTM5ODA2XSwgWzQ1Ljk5MTQxMSwgMTAuOTM5ODcxXSwgWzQ1Ljk5MTM5NSwgMTAuOTM5OTc2XSwgWzQ1Ljk5MTM3NywgMTAuOTQwMDk2XSwgWzQ1Ljk5MTM1MiwgMTAuOTQwMTk0XSwgWzQ1Ljk5MTM0MiwgMTAuOTQwMjM5XSwgWzQ1Ljk5MTMyNSwgMTAuOTQwMzE4XSwgWzQ1Ljk5MTMsIDEwLjk0MDQyMl0sIFs0NS45OTEyOCwgMTAuOTQwNTEzXSwgWzQ1Ljk5MTI2OCwgMTAuOTQwNjZdLCBbNDUuOTkxMjczLCAxMC45NDA3OThdLCBbNDUuOTkxMjg0LCAxMC45NDA4OV0sIFs0NS45OTEzMywgMTAuOTQxXSwgWzQ1Ljk5MTM4MywgMTAuOTQxMTI0XSwgWzQ1Ljk5MTQyOSwgMTAuOTQxMjU0XSwgWzQ1Ljk5MTQ2NSwgMTAuOTQxMzkzXSwgWzQ1Ljk5MTQ5LCAxMC45NDE1MTldLCBbNDUuOTkxNTA0LCAxMC45NDE2NDFdLCBbNDUuOTkxNTI0LCAxMC45NDE3ODZdLCBbNDUuOTkxNTU2LCAxMC45NDE5MjVdLCBbNDUuOTkxNjA1LCAxMC45NDIwNzFdLCBbNDUuOTkxNjYsIDEwLjk0MjIxMV0sIFs0NS45OTE3MzEsIDEwLjk0MjMxOV0sIFs0NS45OTE3OTQsIDEwLjk0MjM5NF0sIFs0NS45OTE4NDMsIDEwLjk0MjQ0MV0sIFs0NS45OTE5NjcsIDEwLjk0MjUzMl0sIFs0NS45OTIwNiwgMTAuOTQyNTk0XSwgWzQ1Ljk5MjE4LCAxMC45NDI2NjRdLCBbNDUuOTkyMjg2LCAxMC45NDI3MzFdLCBbNDUuOTkyMzg4LCAxMC45NDI3ODFdLCBbNDUuOTkyNTAyLCAxMC45NDI4MTFdLCBbNDUuOTkyNjE4LCAxMC45NDI4MjldLCBbNDUuOTkyNzI2LCAxMC45NDI4M10sIFs0NS45OTI4MjksIDEwLjk0MjgxN10sIFs0NS45OTI5MjUsIDEwLjk0Mjc5OF0sIFs0NS45OTMwMDUsIDEwLjk0Mjc3OF0sIFs0NS45OTMxMDYsIDEwLjk0Mjc2M10sIFs0NS45OTMxODIsIDEwLjk0Mjc2Ml0sIFs0NS45OTMyNTUsIDEwLjk0Mjc4OF0sIFs0NS45OTMzNTEsIDEwLjk0MjkwN10sIFs0NS45OTMzODksIDEwLjk0Mjk1NF0sIFs0NS45OTM0NDIsIDEwLjk0MzAxMl0sIFs0NS45OTM0ODcsIDEwLjk0MzA2XSwgWzQ1Ljk5MzU2NiwgMTAuOTQzMTEyXSwgWzQ1Ljk5MzYxOCwgMTAuOTQzMTRdLCBbNDUuOTkzNzA3LCAxMC45NDMxODNdLCBbNDUuOTkzNzkxLCAxMC45NDMxOTldLCBbNDUuOTkzODYyLCAxMC45NDMxOTldLCBbNDUuOTkzOTY1LCAxMC45NDMxOTNdLCBbNDUuOTk0MDc3LCAxMC45NDMyXSwgWzQ1Ljk5NDE4MiwgMTAuOTQzMjAxXSwgWzQ1Ljk5NDI3MywgMTAuOTQzMjE4XSwgWzQ1Ljk5NDM4NSwgMTAuOTQzMjUyXSwgWzQ1Ljk5NDQ3MywgMTAuOTQzMjk4XSwgWzQ1Ljk5NDU1NywgMTAuOTQzMzU3XSwgWzQ1Ljk5NDY1OCwgMTAuOTQzNDQ5XSwgWzQ1Ljk5NDc1LCAxMC45NDM1NTFdLCBbNDUuOTk0ODcsIDEwLjk0MzddLCBbNDUuOTk0OTg2LCAxMC45NDM4NTZdLCBbNDUuOTk1MTI4LCAxMC45NDM5OTZdLCBbNDUuOTk1Mjc2LCAxMC45NDQxMl0sIFs0NS45OTU0MDksIDEwLjk0NDI0M10sIFs0NS45OTU1NDksIDEwLjk0NDM1XSwgWzQ1Ljk5NTY2MiwgMTAuOTQ0NDMzXSwgWzQ1Ljk5NTc3LCAxMC45NDQ1MDddLCBbNDUuOTk1OTAxLCAxMC45NDQ1NzFdLCBbNDUuOTk1OTksIDEwLjk0NDYyXSwgWzQ1Ljk5NjA5NSwgMTAuOTQ0NzA2XSwgWzQ1Ljk5NjE1MiwgMTAuOTQ0NzYxXSwgWzQ1Ljk5NjIxMywgMTAuOTQ0NzldLCBbNDUuOTk2MjYsIDEwLjk0NDgyN10sIFs0NS45OTYyOCwgMTAuOTQ0ODY0XSwgWzQ1Ljk5NjM0NSwgMTAuOTQ1MDM0XSwgWzQ1Ljk5NjM5MSwgMTAuOTQ1MDYyXSwgWzQ1Ljk5NjQzMiwgMTAuOTQ1MDU3XSwgWzQ1Ljk5NjQ4LCAxMC45NDUwMzZdLCBbNDUuOTk2NTQ5LCAxMC45NDUwMTNdLCBbNDUuOTk2NjI0LCAxMC45NDUwNDVdLCBbNDUuOTk2NjczLCAxMC45NDUxMTNdLCBbNDUuOTk2NzA3LCAxMC45NDUxNTddLCBbNDUuOTk2NzQ3LCAxMC45NDUxOThdLCBbNDUuOTk2ODEzLCAxMC45NDUyMzNdLCBbNDUuOTk2ODc2LCAxMC45NDUyNzVdLCBbNDUuOTk2OTU2LCAxMC45NDU0MDNdLCBbNDUuOTk2OTkzLCAxMC45NDU1MjldLCBbNDUuOTk3MDM3LCAxMC45NDU1ODZdLCBbNDUuOTk3MDc1LCAxMC45NDU2NDNdLCBbNDUuOTk3MTAxLCAxMC45NDU3Ml0sIFs0NS45OTcxMzgsIDEwLjk0NTgzXSwgWzQ1Ljk5NzE4MSwgMTAuOTQ1ODY0XSwgWzQ1Ljk5NzIxLCAxMC45NDU4NzhdLCBbNDUuOTk3MzA0LCAxMC45NDU4ODJdLCBbNDUuOTk3MzY2LCAxMC45NDU4NzhdLCBbNDUuOTk3NDY5LCAxMC45NDU4NTldLCBbNDUuOTk3NTA2LCAxMC45NDU4NDRdLCBbNDUuOTk3NTU3LCAxMC45NDU4MV0sIFs0NS45OTc2MzIsIDEwLjk0NTgxM10sIFs0NS45OTc2ODIsIDEwLjk0NTg1N10sIFs0NS45OTc3MDksIDEwLjk0NTg1OF0sIFs0NS45OTc3NjksIDEwLjk0NTgzOF0sIFs0NS45OTc4MTMsIDEwLjk0NTgyM10sIFs0NS45OTc4NjIsIDEwLjk0NTg0OF0sIFs0NS45OTc5MjgsIDEwLjk0NTg5Nl0sIFs0NS45OTgwNDMsIDEwLjk0NTk4OV0sIFs0NS45OTgxMywgMTAuOTQ2MDg4XSwgWzQ1Ljk5ODIyNCwgMTAuOTQ2MjFdLCBbNDUuOTk4MzA0LCAxMC45NDYzMzRdLCBbNDUuOTk4Mzc5LCAxMC45NDY0NzhdLCBbNDUuOTk4NDQzLCAxMC45NDY2MTJdLCBbNDUuOTk4NTIsIDEwLjk0Njc1Nl0sIFs0NS45OTg2NTMsIDEwLjk0Njg3XSwgWzQ1Ljk5ODcyMSwgMTAuOTQ2ODc5XSwgWzQ1Ljk5ODc2NywgMTAuOTQ2ODc3XSwgWzQ1Ljk5ODgxMiwgMTAuOTQ2OTEyXSwgWzQ1Ljk5ODg0MSwgMTAuOTQ2OTU5XSwgWzQ1Ljk5ODg2NiwgMTAuOTQ2OTgzXSwgWzQ1Ljk5ODkyNywgMTAuOTQ3MDEyXSwgWzQ1Ljk5ODk2MywgMTAuOTQ3MDFdLCBbNDUuOTk5MDY5LCAxMC45NDY5OTRdLCBbNDUuOTk5MTA4LCAxMC45NDY5ODRdLCBbNDUuOTk5MTA4LCAxMC45NDY5OTldLCBbNDUuOTk5MTM1LCAxMC45NDY5OTRdLCBbNDUuOTk5MTkyLCAxMC45NDY5NzJdLCBbNDUuOTk5MjQyLCAxMC45NDY5NTZdLCBbNDUuOTk5MzI1LCAxMC45NDY5MzVdLCBbNDUuOTk5NDQsIDEwLjk0Njg5N10sIFs0NS45OTk1MjksIDEwLjk0Njg3Nl0sIFs0NS45OTk2MjcsIDEwLjk0Njg3NF0sIFs0NS45OTk3MDcsIDEwLjk0Njg5NV0sIFs0NS45OTk4MDMsIDEwLjk0NjkzOV0sIFs0NS45OTk5NDksIDEwLjk0Njk5N10sIFs0Ni4wMDAwNTYsIDEwLjk0NzAzNV0sIFs0Ni4wMDAxNTgsIDEwLjk0NzA2OV0sIFs0Ni4wMDAyNjEsIDEwLjk0NzE0XSwgWzQ2LjAwMDM1OCwgMTAuOTQ3MjNdLCBbNDYuMDAwNDI2LCAxMC45NDczMjZdLCBbNDYuMDAwNTg1LCAxMC45NDc0N10sIFs0Ni4wMDA2NjksIDEwLjk0NzU0XSwgWzQ2LjAwMDc5LCAxMC45NDc1ODhdLCBbNDYuMDAwOTE4LCAxMC45NDc2Ml0sIFs0Ni4wMDEwMDcsIDEwLjk0NzY1N10sIFs0Ni4wMDExMDUsIDEwLjk0NzcyNV0sIFs0Ni4wMDEyLCAxMC45NDc4MDVdLCBbNDYuMDAxMzE0LCAxMC45NDc4OTVdLCBbNDYuMDAxMzkxLCAxMC45NDc5NjJdLCBbNDYuMDAxNTE2LCAxMC45NDgwNzNdLCBbNDYuMDAxNjE2LCAxMC45NDgxNDNdLCBbNDYuMDAxNzU1LCAxMC45NDgyMjFdLCBbNDYuMDAxOTU1LCAxMC45NDgzXSwgWzQ2LjAwMjE4NCwgMTAuOTQ4Mzc5XSwgWzQ2LjAwMjQyMSwgMTAuOTQ4NDQyXSwgWzQ2LjAwMjYwMywgMTAuOTQ4NDk3XSwgWzQ2LjAwMjc3OSwgMTAuOTQ4NTk4XSwgWzQ2LjAwMjkzNSwgMTAuOTQ4NzI5XSwgWzQ2LjAwMzA3NCwgMTAuOTQ4ODM2XSwgWzQ2LjAwMzIxMSwgMTAuOTQ4OTE0XSwgWzQ2LjAwMzQxNSwgMTAuOTQ5MDY4XSwgWzQ2LjAwMzY0OSwgMTAuOTQ5Mjc1XSwgWzQ2LjAwMzg0MywgMTAuOTQ5NDE2XSwgWzQ2LjAwMzk5OSwgMTAuOTQ5NTI3XSwgWzQ2LjAwNDE3OSwgMTAuOTQ5NjcxXSwgWzQ2LjAwNDMzNiwgMTAuOTQ5ODA4XSwgWzQ2LjAwNDQ1MiwgMTAuOTQ5ODg2XSwgWzQ2LjAwNDU2NiwgMTAuOTQ5OTI3XSwgWzQ2LjAwNDY3OCwgMTAuOTQ5OTU4XSwgWzQ2LjAwNDgxNCwgMTAuOTUwMDA3XSwgWzQ2LjAwNDk3LCAxMC45NTAwMzldLCBbNDYuMDA1MDkxLCAxMC45NTAwNF0sIFs0Ni4wMDUyLCAxMC45NTAwNjVdLCBbNDYuMDA1MzAzLCAxMC45NTAxMDldLCBbNDYuMDA1NDMsIDEwLjk1MDIwM10sIFs0Ni4wMDU1NzYsIDEwLjk1MDMxN10sIFs0Ni4wMDU3MDUsIDEwLjk1MDQzOF0sIFs0Ni4wMDU4NzUsIDEwLjk1MDU5MV0sIFs0Ni4wMDYwMzUsIDEwLjk1MDcyMl0sIFs0Ni4wMDYyNTcsIDEwLjk1MDldLCBbNDYuMDA2NDI4LCAxMC45NTEwNDddLCBbNDYuMDA2NTk4LCAxMC45NTEyMTFdLCBbNDYuMDA2NzU5LCAxMC45NTEzODRdLCBbNDYuMDA2OTExLCAxMC45NTE1NDddLCBbNDYuMDA3MDU2LCAxMC45NTE3MDddLCBbNDYuMDA3MjI0LCAxMC45NTE4NzddLCBbNDYuMDA3MzY0LCAxMC45NTIwNDddLCBbNDYuMDA3NTAzLCAxMC45NTIyMV0sIFs0Ni4wMDc2MjgsIDEwLjk1MjI4OF0sIFs0Ni4wMDc3NCwgMTAuOTUyMzAzXSwgWzQ2LjAwNzg0NSwgMTAuOTUyMzQ3XSwgWzQ2LjAwODAxNSwgMTAuOTUyNDg4XSwgWzQ2LjAwODEzMywgMTAuOTUyNTcyXSwgWzQ2LjAwODE2NSwgMTAuOTUyNjA4XSwgWzQ2LjAwODIzMSwgMTAuOTUyNjYyXSwgWzQ2LjAwODI1OCwgMTAuOTUyNjYyXSwgWzQ2LjAwODMwOCwgMTAuOTUyNzEyXSwgWzQ2LjAwODM5NCwgMTAuOTUyODA2XSwgWzQ2LjAwODQ3OSwgMTAuOTUyODY2XSwgWzQ2LjAwODUxNSwgMTAuOTUyODkzXSwgWzQ2LjAwODY3MSwgMTAuOTUzMDYzXSwgWzQ2LjAwODgwMSwgMTAuOTUzMTg3XSwgWzQ2LjAwODk1NSwgMTAuOTUzMzA0XSwgWzQ2LjAwOTAyOCwgMTAuOTUzMzQ4XSwgWzQ2LjAwOTEyNiwgMTAuOTUzNDA5XSwgWzQ2LjAwOTI2NywgMTAuOTUzNDc2XSwgWzQ2LjAwOTM1NCwgMTAuOTUzNTExXSwgWzQ2LjAwOTM5NSwgMTAuOTUzNTc0XSwgWzQ2LjAwOTQxOCwgMTAuOTUzNjYxXSwgWzQ2LjAwOTQyNiwgMTAuOTUzNjg5XSwgWzQ2LjAwOTQyNywgMTAuOTUzNzU4XSwgWzQ2LjAwOTQxMywgMTAuOTUzODM0XSwgWzQ2LjAwOTM3MiwgMTAuOTUzODk1XSwgWzQ2LjAwOTMzNywgMTAuOTUzOTE0XSwgWzQ2LjAwOTI3NSwgMTAuOTUzOTEzXSwgWzQ2LjAwOTE5MywgMTAuOTUzOTI5XSwgWzQ2LjAwOTEzMywgMTAuOTUzOTU3XSwgWzQ2LjAwOTA1NSwgMTAuOTU0MDE1XSwgWzQ2LjAwOTAwMiwgMTAuOTU0MV0sIFs0Ni4wMDg5NjcsIDEwLjk1NDE3NV0sIFs0Ni4wMDg5NDYsIDEwLjk1NDI3M10sIFs0Ni4wMDg5MDgsIDEwLjk1NDM3N10sIFs0Ni4wMDg4NjksIDEwLjk1NDQ1OV0sIFs0Ni4wMDg4NzUsIDEwLjk1NDU5XSwgWzQ2LjAwODkxNiwgMTAuOTU0NjE3XSwgWzQ2LjAwODk3MywgMTAuOTU0NjE4XSwgWzQ2LjAwOTA0NCwgMTAuOTU0NjE5XSwgWzQ2LjAwOTEzNSwgMTAuOTU0NjA4XSwgWzQ2LjAwOTE5OSwgMTAuOTU0NjI1XSwgWzQ2LjAwOTIxNywgMTAuOTU0NjY4XSwgWzQ2LjAwOTIxNSwgMTAuOTU0NjkzXSwgWzQ2LjAwOTIxLCAxMC45NTQ3NTNdLCBbNDYuMDA5MTgsIDEwLjk1NDc5Ml0sIFs0Ni4wMDkxNDcsIDEwLjk1NDg0OF0sIFs0Ni4wMDkxMzUsIDEwLjk1NDldLCBbNDYuMDA5MTQsIDEwLjk1NDk0OV0sIFs0Ni4wMDkxNiwgMTAuOTU1MDEyXSwgWzQ2LjAwOTIsIDEwLjk1NTA4Ml0sIFs0Ni4wMDkyNDgsIDEwLjk1NTExNV0sIFs0Ni4wMDkzMDksIDEwLjk1NTE2Ml0sIFs0Ni4wMDkzOSwgMTAuOTU1Mzg3XSwgWzQ2LjAwOTQxMiwgMTAuOTU1NDk2XSwgWzQ2LjAwOTQzNiwgMTAuOTU1NjU0XSwgWzQ2LjAwOTQ2LCAxMC45NTU4MThdLCBbNDYuMDA5NDc1LCAxMC45NTU5NDNdLCBbNDYuMDA5NTM4LCAxMC45NTYwODldLCBbNDYuMDA5NjE1LCAxMC45NTYxODldLCBbNDYuMDA5Njc0LCAxMC45NTYyMzJdLCBbNDYuMDA5NzQ3LCAxMC45NTYyNjNdLCBbNDYuMDA5ODM4LCAxMC45NTYyOTFdLCBbNDYuMDA5OTkzLCAxMC45NTYzMjldLCBbNDYuMDEwMTI4LCAxMC45NTYzNDhdLCBbNDYuMDEwMjM1LCAxMC45NTYzNTldLCBbNDYuMDEwMzM4LCAxMC45NTYzNjFdLCBbNDYuMDEwNDUzLCAxMC45NTYzNzNdLCBbNDYuMDEwNTQ4LCAxMC45NTYzODddLCBbNDYuMDEwNjU4LCAxMC45NTY0MDZdLCBbNDYuMDEwNzg4LCAxMC45NTY0MzRdLCBbNDYuMDEwODk1LCAxMC45NTY0NTVdLCBbNDYuMDExMDEsIDEwLjk1NjQ2N10sIFs0Ni4wMTA5OSwgMTAuOTU2NjU0XSwgWzQ2LjAxMDk3LCAxMC45NTY4MzRdXV1dLCAidHlwZSI6ICJNdWx0aVBvbHlnb24ifSwgImlkIjogIjUiLCAicHJvcGVydGllcyI6IHsiZGVzY3JpemlvbmVfb3JpZ2luZSI6ICJOb24gY29kaWZpY2F0byIsICJnbWxfaWQiOiAiSUQuQUNRVUVGSVNJQ0hFLlNQRUNDSEkuQUNRVUEuNTQxNTYiLCAibmFzcCI6IDAsICJuYXR1cmFfc3BlY2NoaW9fYWNxdWEiOiAiTm9uIGNvZGlmaWNhdG8iLCAibm9tZSI6ICJMQUdPIERJIENBVkVESU5FIiwgIm9yaWdpbmUiOiAwfSwgInR5cGUiOiAiRmVhdHVyZSJ9LCB7Imdlb21ldHJ5IjogeyJjb29yZGluYXRlcyI6IFtbW1s0NS45MzU3MDksIDEwLjgxOTk1XSwgWzQ1LjkzNTYyNywgMTAuODE5OTQ2XSwgWzQ1LjkzNTU3LCAxMC44MTk5MTldLCBbNDUuOTM1NTA4LCAxMC44MTk4ODNdLCBbNDUuOTM1NDQsIDEwLjgxOTgzN10sIFs0NS45MzUzOTcsIDEwLjgxOTgwMV0sIFs0NS45MzUzNjIsIDEwLjgxOTc2MV0sIFs0NS45MzUzMzcsIDEwLjgxOTcyMl0sIFs0NS45MzUzMSwgMTAuODE5NjM5XSwgWzQ1LjkzNTI5NywgMTAuODE5NTcxXSwgWzQ1LjkzNTI5OSwgMTAuODE5NDkyXSwgWzQ1LjkzNTMzNiwgMTAuODE5MzldLCBbNDUuOTM1MzczLCAxMC44MTkzMDVdLCBbNDUuOTM1NDIxLCAxMC44MTkyMDddLCBbNDUuOTM1NDcyLCAxMC44MTkwNjddLCBbNDUuOTM1NjI4LCAxMC44MTg3MTNdLCBbNDUuOTM1NzI1LCAxMC44MTg1MjddLCBbNDUuOTM1ODA3LCAxMC44MTgzNTRdLCBbNDUuOTM1OTAyLCAxMC44MTgxMTFdLCBbNDUuOTM1OTY2LCAxMC44MTc5MjVdLCBbNDUuOTM2MDA4LCAxMC44MTc3MzJdLCBbNDUuOTM2MDUyLCAxMC44MTc0ODNdLCBbNDUuOTM2MDgzLCAxMC44MTcyNDRdLCBbNDUuOTM2MTA2LCAxMC44MTY5OThdLCBbNDUuOTM2MTM5LCAxMC44MTY3NTldLCBbNDUuOTM2MTU1LCAxMC44MTY1NzVdLCBbNDUuOTM2MTcyLCAxMC44MTYzODhdLCBbNDUuOTM2MTYzLCAxMC44MTYyMTVdLCBbNDUuOTM2MTQxLCAxMC44MTYwNTFdLCBbNDUuOTM2MTA5LCAxMC44MTU4OTNdLCBbNDUuOTM2MDYyLCAxMC44MTU3MjJdLCBbNDUuOTM2MDEyLCAxMC44MTU1NjJdLCBbNDUuOTM1OTYyLCAxMC44MTU0MDFdLCBbNDUuOTM1OTM1LCAxMC44MTUyODZdLCBbNDUuOTM1OTIxLCAxMC44MTUxNTVdLCBbNDUuOTM1OTI0LCAxMC44MTUwODFdLCBbNDUuOTM1OTI2LCAxMC44MTUwMl0sIFs0NS45MzU5NDcsIDEwLjgxNDkwMl0sIFs0NS45MzU5ODksIDEwLjgxNDc3OF0sIFs0NS45MzYwMjgsIDEwLjgxNDY3XSwgWzQ1LjkzNjA2OSwgMTAuODE0NTY1XSwgWzQ1LjkzNjE4NCwgMTAuODE0Mzc2XSwgWzQ1LjkzNjI2NSwgMTAuODE0MjYxXSwgWzQ1LjkzNjMzMywgMTAuODE0MTc2XSwgWzQ1LjkzNjQxNCwgMTAuODE0MTAxXSwgWzQ1LjkzNjQ5NiwgMTAuODE0MDMzXSwgWzQ1LjkzNjY3LCAxMC44MTM4OTNdLCBbNDUuOTM2NzI1LCAxMC44MTM4MzFdLCBbNDUuOTM2Nzk0LCAxMC44MTM3NTNdLCBbNDUuOTM2ODU2LCAxMC44MTM2OTddLCBbNDUuOTM2OTY4LCAxMC44MTM2MTJdLCBbNDUuOTM3MDIzLCAxMC44MTM1ODNdLCBbNDUuOTM3MTA1LCAxMC44MTM1NTRdLCBbNDUuOTM3MTkyLCAxMC44MTM1MzhdLCBbNDUuOTM3MjkzLCAxMC44MTM1MTJdLCBbNDUuOTM3MzY0LCAxMC44MTM0ODNdLCBbNDUuOTM3NDMsIDEwLjgxMzQ1MV0sIFs0NS45Mzc1MTcsIDEwLjgxMzQxMl0sIFs0NS45Mzc1OTcsIDEwLjgxMzM2Nl0sIFs0NS45Mzc2NTksIDEwLjgxMzMyMV0sIFs0NS45Mzc3NTEsIDEwLjgxMzIyXSwgWzQ1LjkzNzgzOCwgMTAuODEzMTI4XSwgWzQ1LjkzNzkzOSwgMTAuODEzMDQ3XSwgWzQ1LjkzODAzOSwgMTAuODEyOTk1XSwgWzQ1LjkzODE0NSwgMTAuODEyOTU2XSwgWzQ1LjkzODI1LCAxMC44MTI5M10sIFs0NS45MzgzNTcsIDEwLjgxMjkzMV0sIFs0NS45Mzg0NjcsIDEwLjgxMjk0NF0sIFs0NS45Mzg1NzksIDEwLjgxMjk3MV0sIFs0NS45Mzg2NjgsIDEwLjgxMjk5OF0sIFs0NS45Mzg3NjgsIDEwLjgxMzA0NF0sIFs0NS45Mzg4NTcsIDEwLjgxMzA5N10sIFs0NS45Mzg5MjgsIDEwLjgxMzEzXSwgWzQ1LjkzOTAxLCAxMC44MTMxNjNdLCBbNDUuOTM5MDcsIDEwLjgxMzE5XSwgWzQ1LjkzOTEzOCwgMTAuODEzMjJdLCBbNDUuOTM5MjMsIDEwLjgxMzI1Nl0sIFs0NS45MzkzMTIsIDEwLjgxMzI3Nl0sIFs0NS45MzkzOCwgMTAuODEzM10sIFs0NS45Mzk0NiwgMTAuODEzMzFdLCBbNDUuOTM5NTQ5LCAxMC44MTMzMV0sIFs0NS45Mzk2NDMsIDEwLjgxMzMxMV0sIFs0NS45Mzk3MzksIDEwLjgxMzMxNF0sIFs0NS45Mzk4MzcsIDEwLjgxMzMzMV0sIFs0NS45Mzk5NDUsIDEwLjgxMzM1NV0sIFs0NS45NDAxMjEsIDEwLjgxMzQ0MV0sIFs0NS45NDAxOTYsIDEwLjgxMzQ5XSwgWzQ1Ljk0MDI2MiwgMTAuODEzNTE0XSwgWzQ1Ljk0MDQzMywgMTAuODEzNTQ3XSwgWzQ1Ljk0MDUzOCwgMTAuODEzNTU0XSwgWzQ1Ljk0MDY1LCAxMC44MTM1NjhdLCBbNDUuOTQwNzI4LCAxMC44MTM1OTVdLCBbNDUuOTQwODIyLCAxMC44MTM2NDddLCBbNDUuOTQwOTExLCAxMC44MTM2OTddLCBbNDUuOTQxMDAyLCAxMC44MTM3NF0sIFs0NS45NDExMDksIDEwLjgxMzc3N10sIFs0NS45NDEyMDUsIDEwLjgxMzc4N10sIFs0NS45NDEzNDIsIDEwLjgxMzk0NV0sIFs0NS45NDEzMzksIDEwLjgxNDAzNF0sIFs0NS45NDEzMjYsIDEwLjgxNDA3XSwgWzQ1Ljk0MTMsIDEwLjgxNDA5OV0sIFs0NS45NDEyNjYsIDEwLjgxNDExOF0sIFs0NS45NDEyMiwgMTAuODE0MTI4XSwgWzQ1Ljk0MTE4NiwgMTAuODE0MTI4XSwgWzQ1Ljk0MTA3NCwgMTAuODE0MjQ5XSwgWzQ1Ljk0MTEwMSwgMTAuODE0MzQxXSwgWzQ1Ljk0MTE0LCAxMC44MTQ0MV0sIFs0NS45NDExNjksIDEwLjgxNDQ1OV0sIFs0NS45NDEyNDIsIDEwLjgxNDU4N10sIFs0NS45NDEyNzYsIDEwLjgxNDY4Nl0sIFs0NS45NDEyODcsIDEwLjgxNDgyN10sIFs0NS45NDEyNzksIDEwLjgxNDg3XSwgWzQ1Ljk0MTI2OCwgMTAuODE0OTI4XSwgWzQ1Ljk0MTI0OCwgMTAuODE0OTc0XSwgWzQ1Ljk0MTIwNiwgMTAuODE1MDMzXSwgWzQ1Ljk0MTE1NiwgMTAuODE1MDg4XSwgWzQ1Ljk0MTA5LCAxMC44MTUxNDRdLCBbNDUuOTQxMDI1LCAxMC44MTUxNl0sIFs0NS45NDA5NTUsIDEwLjgxNTE4Nl0sIFs0NS45NDA4ODgsIDEwLjgxNTIyNV0sIFs0NS45NDA4MDEsIDEwLjgxNTI4M10sIFs0NS45NDA3MDcsIDEwLjgxNTMzMl0sIFs0NS45NDA2NSwgMTAuODE1Mzk0XSwgWzQ1Ljk0MDU2NywgMTAuODE1NTA1XSwgWzQ1Ljk0MDM4OSwgMTAuODE1NTc2XSwgWzQ1Ljk0MDMwNCwgMTAuODE1NjM1XSwgWzQ1Ljk0MDI0MiwgMTAuODE1N10sIFs0NS45NDAxOTQsIDEwLjgxNTc3OV0sIFs0NS45NDAxMzcsIDEwLjgxNTldLCBbNDUuOTQwMTAyLCAxMC44MTYwMTRdLCBbNDUuOTQwMDY1LCAxMC44MTYxMjJdLCBbNDUuOTQwMDE3LCAxMC44MTYyMDddLCBbNDUuOTM5OTc2LCAxMC44MTYyODNdLCBbNDUuOTM5OTQ4LCAxMC44MTYzMzJdLCBbNDUuOTM5OTE4LCAxMC44MTY0XSwgWzQ1LjkzOTkwOSwgMTAuODE2NDcyXSwgWzQ1LjkzOTk0LCAxMC44MTY3ODRdLCBbNDUuOTM5OTk5LCAxMC44MTY4OTJdLCBbNDUuOTQwMDU2LCAxMC44MTY5ODFdLCBbNDUuOTQwMDk5LCAxMC44MTcwNF0sIFs0NS45NDAxNzIsIDEwLjgxNzExXSwgWzQ1Ljk0MDIzNCwgMTAuODE3MTg1XSwgWzQ1Ljk0MDMwNCwgMTAuODE3Mjc0XSwgWzQ1Ljk0MDM4NCwgMTAuODE3MzZdLCBbNDUuOTQwNDU3LCAxMC44MTc0MzldLCBbNDUuOTQwNTM3LCAxMC44MTc1MzFdLCBbNDUuOTQwNTg5LCAxMC44MTc2MV0sIFs0NS45NDA2NTcsIDEwLjgxNzcwOV0sIFs0NS45NDA3MDcsIDEwLjgxNzc3MV0sIFs0NS45NDA3NiwgMTAuODE3ODIxXSwgWzQ1Ljk0MDgwOCwgMTAuODE3ODc3XSwgWzQ1Ljk0MDgxNCwgMTAuODE3OTI5XSwgWzQ1Ljk0MDgxMiwgMTAuODE3OTk4XSwgWzQ1Ljk0MDc3NywgMTAuODE4MDRdLCBbNDUuOTQwNzI1LCAxMC44MTgwNzNdLCBbNDUuOTQwNjkxLCAxMC44MTgwNzldLCBbNDUuOTQwNjI3LCAxMC44MTgwODZdLCBbNDUuOTQwNTY1LCAxMC44MTgwODVdLCBbNDUuOTQwNTE1LCAxMC44MTgwNzhdLCBbNDUuOTQwNDE5LCAxMC44MTgwMzldLCBbNDUuOTQwMzU1LCAxMC44MTc5OTJdLCBbNDUuOTQwMjg5LCAxMC44MTc5NDNdLCBbNDUuOTQwMjA2LCAxMC44MTc4OV0sIFs0NS45NDAxMzYsIDEwLjgxNzg1NF0sIFs0NS45NDAwNTYsIDEwLjgxNzgxNF0sIFs0NS45Mzk5ODMsIDEwLjgxNzc4MV0sIFs0NS45Mzk5MDMsIDEwLjgxNzc2N10sIFs0NS45Mzk3ODksIDEwLjgxNzc0N10sIFs0NS45Mzk3MDYsIDEwLjgxNzczNF0sIFs0NS45Mzk2MTksIDEwLjgxNzc0XSwgWzQ1LjkzOTUzNywgMTAuODE3ODA1XSwgWzQ1LjkzOTQ4NywgMTAuODE3ODVdLCBbNDUuOTM5MzQyLCAxMC44MTc5ODRdLCBbNDUuOTM5MjUzLCAxMC44MTgwNTldLCBbNDUuOTM5MTM4LCAxMC44MTgxNDRdLCBbNDUuOTM5MDI2LCAxMC44MTgyMjVdLCBbNDUuOTM4OSwgMTAuODE4MzE2XSwgWzQ1LjkzODc3OSwgMTAuODE4NDExXSwgWzQ1LjkzODYwNywgMTAuODE4NTQ0XSwgWzQ1LjkzODU1NCwgMTAuODE4NTg0XSwgWzQ1LjkzODE3MiwgMTAuODE4ODQ3XSwgWzQ1LjkzODA1NSwgMTAuODE4OTM4XSwgWzQ1LjkzNzc5NiwgMTAuODE5MTVdLCBbNDUuOTM3Njc5LCAxMC44MTkyMzVdLCBbNDUuOTM3NTc5LCAxMC44MTkzMV0sIFs0NS45Mzc0NDMsIDEwLjgxOTQxN10sIFs0NS45MzczMzgsIDEwLjgxOTQ5OV0sIFs0NS45MzcyMywgMTAuODE5NTldLCBbNDUuOTM3MTQ2LCAxMC44MTk2NTJdLCBbNDUuOTM3MDY4LCAxMC44MTk3MDddLCBbNDUuOTM2OTkyLCAxMC44MTk3MzNdLCBbNDUuOTM2OTE5LCAxMC44MTk3NDZdLCBbNDUuOTM2ODMyLCAxMC44MTk3NThdLCBbNDUuOTM2NjYxLCAxMC44MTk3NzFdLCBbNDUuOTM2NTc2LCAxMC44MTk3NjRdLCBbNDUuOTM2NTAxLCAxMC44MTk3Nl0sIFs0NS45MzYzODksIDEwLjgxOTc2OV0sIFs0NS45MzYzMTYsIDEwLjgxOTc3Nl0sIFs0NS45MzYyNDIsIDEwLjgxOTc4OF0sIFs0NS45MzYxNTMsIDEwLjgxOTgwNF0sIFs0NS45MzYwOTEsIDEwLjgxOTgxN10sIFs0NS45MzU5OTUsIDEwLjgxOTg0Nl0sIFs0NS45MzU5MDYsIDEwLjgxOTg4Ml0sIFs0NS45MzU3OTIsIDEwLjgxOTk0XSwgWzQ1LjkzNTcwOSwgMTAuODE5OTVdXV1dLCAidHlwZSI6ICJNdWx0aVBvbHlnb24ifSwgImlkIjogIjYiLCAicHJvcGVydGllcyI6IHsiZGVzY3JpemlvbmVfb3JpZ2luZSI6ICJOb24gY29kaWZpY2F0byIsICJnbWxfaWQiOiAiSUQuQUNRVUVGSVNJQ0hFLlNQRUNDSEkuQUNRVUEuNTQxNzYiLCAibmFzcCI6IDAsICJuYXR1cmFfc3BlY2NoaW9fYWNxdWEiOiAiTm9uIGNvZGlmaWNhdG8iLCAibm9tZSI6ICJMQUdPIERJIFRFTk5PIiwgIm9yaWdpbmUiOiAwfSwgInR5cGUiOiAiRmVhdHVyZSJ9LCB7Imdlb21ldHJ5IjogeyJjb29yZGluYXRlcyI6IFtbW1s0NS44Mzc2MywgMTAuODI1ODUzXSwgWzQ1LjgzNzcxLCAxMC44MjU5XSwgWzQ1LjgzNzc5OSwgMTAuODI1OTExXSwgWzQ1LjgzNzkzMSwgMTAuODI1OTFdLCBbNDUuODM4MDY2LCAxMC44MjU5MzhdLCBbNDUuODM4MTYyLCAxMC44MjU5NTNdLCBbNDUuODM4MzAxLCAxMC44MjU5NTVdLCBbNDUuODM4NDQ3LCAxMC44MjU5OTZdLCBbNDUuODM4NTk4LCAxMC44MjYwNjFdLCBbNDUuODM4NzIzLCAxMC44MjYwODJdLCBbNDUuODM4ODc2LCAxMC44MjYxMjddLCBbNDUuODM5MTIsIDEwLjgyNjIyM10sIFs0NS44MzkzMTEsIDEwLjgyNjMwNF0sIFs0NS44Mzk0ODEsIDEwLjgyNjI5XSwgWzQ1LjgzOTY4MiwgMTAuODI2MjYxXSwgWzQ1LjgzOTk2LCAxMC44MjYyOTRdLCBbNDUuODQwNDA1LCAxMC44MjYyMDNdLCBbNDUuODQwNTU2LCAxMC44MjYxNDNdLCBbNDUuODQwNzg5LCAxMC44MjYxNjZdLCBbNDUuODQwOTQyLCAxMC44MjYxNzhdLCBbNDUuODQxMDU2LCAxMC44MjYxNDRdLCBbNDUuODQxMTc1LCAxMC44MjYxNDJdLCBbNDUuODQxMjY0LCAxMC44MjYxN10sIFs0NS44NDEzMzUsIDEwLjgyNjE3MV0sIFs0NS44NDE0MjksIDEwLjgyNjE1OV0sIFs0NS44NDE1NCwgMTAuODI2MjE3XSwgWzQ1Ljg0MTYyNCwgMTAuODI2MzA2XSwgWzQ1Ljg0MTgzOCwgMTAuODI2NDE3XSwgWzQ1Ljg0MTk4NCwgMTAuODI2NTE1XSwgWzQ1Ljg0MjI1MywgMTAuODI2NjI3XSwgWzQ1Ljg0MjQwNiwgMTAuODI2NjQ5XSwgWzQ1Ljg0MjYwNCwgMTAuODI2NjkxXSwgWzQ1Ljg0Mjc2OCwgMTAuODI2NzQyXSwgWzQ1Ljg0MjkzLCAxMC44MjY4MDFdLCBbNDUuODQzMTEyLCAxMC44MjY5NTRdLCBbNDUuODQzMzE2LCAxMC44MjcxMzddLCBbNDUuODQzNDkzLCAxMC44MjcyNzRdLCBbNDUuODQzNzI5LCAxMC44Mjc1MjZdLCBbNDUuODQzOTc2LCAxMC44Mjc4MzFdLCBbNDUuODQ0MjYxLCAxMC44MjgzMjNdLCBbNDUuODQ0MzM3LCAxMC44Mjg0NjhdLCBbNDUuODQ0NDM5LCAxMC44Mjg2MDFdLCBbNDUuODQ0NTcsIDEwLjgyODc5Ml0sIFs0NS44NDQ3LCAxMC44Mjg5MDldLCBbNDUuODQ0ODYxLCAxMC44Mjg5OTddLCBbNDUuODQ1MDQ4LCAxMC44MjkwNTJdLCBbNDUuODQ1MTgyLCAxMC44MjkxNTldLCBbNDUuODQ1MzA1LCAxMC44MjkzMDhdLCBbNDUuODQ1MzcsIDEwLjgyOTQ0Nl0sIFs0NS44NDU0NDYsIDEwLjgyOTYwOF0sIFs0NS44NDU1NTcsIDEwLjgyOTc4Nl0sIFs0NS44NDU2MTMsIDEwLjgyOTkzMV0sIFs0NS44NDU3NTMsIDEwLjgzMDE2OV0sIFs0NS44NDU4MzUsIDEwLjgzMDI3Ml0sIFs0NS44NDU5MDMsIDEwLjgzMDMwMl0sIFs0NS44NDYwNCwgMTAuODMwMzA0XSwgWzQ1Ljg0NjE2MSwgMTAuODMwMzM1XSwgWzQ1Ljg0NjMwMiwgMTAuODMwNDE5XSwgWzQ1Ljg0NjQ1MiwgMTAuODMwNDk3XSwgWzQ1Ljg0NjU1NSwgMTAuODMwNDg5XSwgWzQ1Ljg0NjY3NiwgMTAuODMwNTUzXSwgWzQ1Ljg0Njc2NSwgMTAuODMwNTc3XSwgWzQ1Ljg0NjkyMiwgMTAuODMwNTk5XSwgWzQ1Ljg0NzA2MSwgMTAuODMwNjQ0XSwgWzQ1Ljg0NzE4OSwgMTAuODMwNjg4XSwgWzQ1Ljg0NzM1NiwgMTAuODMwNzA0XSwgWzQ1Ljg0NzU3OCwgMTAuODMwNjg3XSwgWzQ1Ljg0Nzc2NywgMTAuODMwNjgxXSwgWzQ1Ljg0Nzk4MSwgMTAuODMwNzkyXSwgWzQ1Ljg0ODI4OSwgMTAuODMwOTExXSwgWzQ1Ljg0ODY4NiwgMTAuODMxMDU1XSwgWzQ1Ljg0ODk3MiwgMTAuODMxMjM5XSwgWzQ1Ljg0OTEsIDEwLjgzMTMxNF0sIFs0NS44NDk3MTMsIDEwLjgzMTQ4Nl0sIFs0NS44NDk5MTksIDEwLjgzMTU0NF0sIFs0NS44NTAxNzksIDEwLjgzMTY0M10sIFs0NS44NTA0MDIsIDEwLjgzMTc1NF0sIFs0NS44NTA2MzUsIDEwLjgzMTg1Ml0sIFs0NS44NTA4NjYsIDEwLjgzMjAyN10sIFs0NS44NTExMzQsIDEwLjgzMjE3N10sIFs0NS44NTEzNzIsIDEwLjgzMjMzOF0sIFs0NS44NTE2ODksIDEwLjgzMjQ4OF0sIFs0NS44NTE4MjEsIDEwLjgzMjU1OV0sIFs0NS44NTE5OTgsIDEwLjgzMjY2MV0sIFs0NS44NTIxNTcsIDEwLjgzMjc3OV0sIFs0NS44NTIzNzEsIDEwLjgzMjkyNF0sIFs0NS44NTI1MTYsIDEwLjgzMzAxOV0sIFs0NS44NTI2NDYsIDEwLjgzMzA5N10sIFs0NS44NTI4MTQsIDEwLjgzMzE3OV0sIFs0NS44NTMwMDEsIDEwLjgzMzIyNl0sIFs0NS44NTMxNjMsIDEwLjgzMzMyOF0sIFs0NS44NTMzNTQsIDEwLjgzMzQ1M10sIFs0NS44NTM3MjEsIDEwLjgzMzc2OF0sIFs0NS44NTM5NTEsIDEwLjgzMzg1Ml0sIFs0NS44NTQxNzYsIDEwLjgzNDA0M10sIFs0NS44NTQzMDEsIDEwLjgzNDE0MV0sIFs0NS44NTQzNzMsIDEwLjgzNDIzXSwgWzQ1Ljg1NDUwMiwgMTAuODM0MzQxXSwgWzQ1Ljg1NDcyMywgMTAuODM0Mzk1XSwgWzQ1Ljg1NDg4OCwgMTAuODM0Mzg5XSwgWzQ1Ljg1NTEwMywgMTAuODM0NDEzXSwgWzQ1Ljg1NTM1NCwgMTAuODM0NDM4XSwgWzQ1Ljg1NTUxOSwgMTAuODM0NDY4XSwgWzQ1Ljg1NTY0NywgMTAuODM0NDE1XSwgWzQ1Ljg1NTc0NSwgMTAuODM0Mjk5XSwgWzQ1Ljg1NTkzNCwgMTAuODM0MTQzXSwgWzQ1Ljg1NjE3MiwgMTAuODM0MDk5XSwgWzQ1Ljg1NjM0MiwgMTAuODM0MDU2XSwgWzQ1Ljg1NjU5OSwgMTAuODM0MDFdLCBbNDUuODU2ODE5LCAxMC44MzM5NjJdLCBbNDUuODU3MDE5LCAxMC44MzM4NTJdLCBbNDUuODU3MTIzLCAxMC44MzM3NDZdLCBbNDUuODU3MjU0LCAxMC44MzM2MjRdLCBbNDUuODU3NDE1LCAxMC44MzM1NjZdLCBbNDUuODU3NjAxLCAxMC44MzM0OTRdLCBbNDUuODU3NzU2LCAxMC44MzMzODNdLCBbNDUuODU3ODI3LCAxMC44MzMyOTldLCBbNDUuODU3OTE5LCAxMC44MzMyNDZdLCBbNDUuODU4MTA3LCAxMC44MzMyMTRdLCBbNDUuODU4MzIxLCAxMC44MzMxMl0sIFs0NS44NTg3MDcsIDEwLjgzMjkyNV0sIFs0NS44NTg4ODQsIDEwLjgzMjg4N10sIFs0NS44NTkwMjQsIDEwLjgzMjgzNF0sIFs0NS44NTkyMjcsIDEwLjgzMjg3NF0sIFs0NS44NTkzODQsIDEwLjgzMjk2Nl0sIFs0NS44NTk3MTMsIDEwLjgzMzAwNl0sIFs0NS44NTk5NDIsIDEwLjgzMjk5NF0sIFs0NS44NjAzMjksIDEwLjgzMzE4Nl0sIFs0NS44NjA2MTEsIDEwLjgzMzEzXSwgWzQ1Ljg2MDc5NSwgMTAuODMzMDQ1XSwgWzQ1Ljg2MDkyOCwgMTAuODMyOTk2XSwgWzQ1Ljg2MTAyOCwgMTAuODMzMDExXSwgWzQ1Ljg2MTE1NiwgMTAuODMzMDM3XSwgWzQ1Ljg2MTI3NSwgMTAuODMzMDcyXSwgWzQ1Ljg2MTYyMywgMTAuODMzMjJdLCBbNDUuODYxNzczLCAxMC44MzMyOThdLCBbNDUuODYxOTAxLCAxMC44MzMzODNdLCBbNDUuODYyMDIzLCAxMC44MzM0ODFdLCBbNDUuODYyMjM5LCAxMC44MzM1OV0sIFs0NS44NjIzOTcsIDEwLjgzMzY1NV0sIFs0NS44NjI1OTUsIDEwLjgzMzc0NV0sIFs0NS44NjI5MTUsIDEwLjgzMzkxMl0sIFs0NS44NjMzOTMsIDEwLjgzNDE2OF0sIFs0NS44NjM1NjQsIDEwLjgzNDIzXSwgWzQ1Ljg2MzgzNSwgMTAuODM0MzMxXSwgWzQ1Ljg2NDA2NCwgMTAuODM0MzcyXSwgWzQ1Ljg2NDMwMywgMTAuODM0NDQ5XSwgWzQ1Ljg2NDQ3MSwgMTAuODM0NTI4XSwgWzQ1Ljg2NDY1NSwgMTAuODM0NjU2XSwgWzQ1Ljg2NDkwNiwgMTAuODM0NzE3XSwgWzQ1Ljg2NTI5MywgMTAuODM0NzA2XSwgWzQ1Ljg2NTU2MiwgMTAuODM0ODFdLCBbNDUuODY1NzA3LCAxMC44MzQ5MjhdLCBbNDUuODY1OTQ5LCAxMC44MzQ5ODVdLCBbNDUuODY2MDY1LCAxMC44MzUwMjRdLCBbNDUuODY2MTc5LCAxMC44MzUwNjJdLCBbNDUuODY2MzUsIDEwLjgzNTExOF0sIFs0NS44NjY4MDIsIDEwLjgzNTA0M10sIFs0NS44NjY5MzcsIDEwLjgzNTAwNl0sIFs0NS44NjcwNTgsIDEwLjgzNTAyNV0sIFs0NS44NjcxNjIsIDEwLjgzNTExNl0sIFs0NS44NjcyNzUsIDEwLjgzNTI0OV0sIFs0NS44NjczMTQsIDEwLjgzNTMxMl0sIFs0NS44Njc0NTcsIDEwLjgzNTM4N10sIFs0NS44Njc2MjIsIDEwLjgzNTMzOF0sIFs0NS44Njc3OTcsIDEwLjgzNTI5M10sIFs0NS44Njc5ODIsIDEwLjgzNTI2NF0sIFs0NS44NjgxMTMsIDEwLjgzNTIxNV0sIFs0NS44NjgyMzUsIDEwLjgzNTE0Ml0sIFs0NS44Njg0LCAxMC44MzUwOTNdLCBbNDUuODY4NjQ1LCAxMC44MzUwNTZdLCBbNDUuODY4ODUxLCAxMC44MzUwNDddLCBbNDUuODY5MDU1LCAxMC44MzUwNjFdLCBbNDUuODY5MTg4LCAxMC44MzUwMTVdLCBbNDUuODY5MzQzLCAxMC44MzUwMzJdLCBbNDUuODY5NDI3LCAxMC44MzUxMThdLCBbNDUuODY5NTQsIDEwLjgzNTIyOV0sIFs0NS44Njk2NDQsIDEwLjgzNTMzM10sIFs0NS44Njk4MjIsIDEwLjgzNTQyOF0sIFs0NS44Njk5MjYsIDEwLjgzNTUwNl0sIFs0NS44NzAxMzUsIDEwLjgzNTY3XSwgWzQ1Ljg3MDI3MywgMTAuODM1Nzk1XSwgWzQ1Ljg3MDQzMiwgMTAuODM1OTc4XSwgWzQ1Ljg3MDU3NCwgMTAuODM2MTg3XSwgWzQ1Ljg3MDY3LCAxMC44MzYzNjldLCBbNDUuODcwNzg3LCAxMC44MzY1ODVdLCBbNDUuODcwOTUzLCAxMC44MzY4NDRdLCBbNDUuODcxMDg5LCAxMC44MzcwNF0sIFs0NS44NzExNzEsIDEwLjgzNzI2MV0sIFs0NS44NzEyOTIsIDEwLjgzNzQ2Nl0sIFs0NS44NzE0OTIsIDEwLjgzNzYzNF0sIFs0NS44NzE2MDEsIDEwLjgzNzY3Nl0sIFs0NS44NzE3MzMsIDEwLjgzNzc1MV0sIFs0NS44NzE5MTEsIDEwLjgzNzc5N10sIFs0NS44NzIwMTksIDEwLjgzNzcyOF0sIFs0NS44NzIxNiwgMTAuODM3NjQ5XSwgWzQ1Ljg3MjMyLCAxMC44Mzc2MjNdLCBbNDUuODcyNDE2LCAxMC44Mzc2OF0sIFs0NS44NzI1NTgsIDEwLjgzNzgxOF0sIFs0NS44NzI3NDEsIDEwLjgzNzg5XSwgWzQ1Ljg3Mjk5NCwgMTAuODM3OTEyXSwgWzQ1Ljg3MzE1OCwgMTAuODM3OTc4XSwgWzQ1Ljg3MzI4NiwgMTAuODM4MDJdLCBbNDUuODczNTY3LCAxMC44MzgwNDZdLCBbNDUuODczODEyLCAxMC44MzgwNTddLCBbNDUuODczOTU3LCAxMC44MzgxOThdLCBbNDUuODc0MTI5LCAxMC44MzgzMTNdLCBbNDUuODc0MjQ2LCAxMC44MzgzMzJdLCBbNDUuODc0NDUxLCAxMC44MzgzNTNdLCBbNDUuODc0NjAyLCAxMC44MzgzODldLCBbNDUuODc0NzI0LCAxMC44MzgzMDldLCBbNDUuODc0ODI1LCAxMC44MzgyNDNdLCBbNDUuODc1MDAxLCAxMC44MzgyODldLCBbNDUuODc1MTM0LCAxMC44MzgyNDNdLCBbNDUuODc1Mjk5LCAxMC44MzgyMDRdLCBbNDUuODc1NDE2LCAxMC44MzgyNDJdLCBbNDUuODc1NTM4LCAxMC44MzgzMl0sIFs0NS44NzU2NTgsIDEwLjgzODQ4N10sIFs0NS44NzU4MDUsIDEwLjgzODYzNF0sIFs0NS44NzU5NjIsIDEwLjgzODcwM10sIFs0NS44NzYwNTIsIDEwLjgzODY3Ml0sIFs0NS44NzYxNjYsIDEwLjgzODcxN10sIFs0NS44NzYyOTMsIDEwLjgzODgwNV0sIFs0NS44NzY0OTUsIDEwLjgzODkzM10sIFs0NS44NzY2NjMsIDEwLjgzOTAzOV0sIFs0NS44NzY3NzMsIDEwLjgzOTA3N10sIFs0NS44NzY5MjMsIDEwLjgzOTE0Nl0sIFs0NS44NzcwODMsIDEwLjgzOTE3NV0sIFs0NS44NzcyNDksIDEwLjgzOTIxOF0sIFs0NS44NzczNTksIDEwLjgzOTI1XSwgWzQ1Ljg3NzUzOSwgMTAuODM5MjgzXSwgWzQ1Ljg3Nzg4NSwgMTAuODM5MjMyXSwgWzQ1Ljg3ODAxLCAxMC44MzkwOV0sIFs0NS44NzgxMjgsIDEwLjgzODk4OF0sIFs0NS44NzgyNzEsIDEwLjgzODg1NF0sIFs0NS44Nzg0MjMsIDEwLjgzODcxXSwgWzQ1Ljg3ODU0MiwgMTAuODM4NDkzXSwgWzQ1Ljg3ODY1MSwgMTAuODM4Mzg3XSwgWzQ1Ljg3ODg5NywgMTAuODM4Mjk0XSwgWzQ1Ljg3OTE5LCAxMC44MzgyMDldLCBbNDUuODc5NDA2LCAxMC44MzgxNDFdLCBbNDUuODc5Njk0LCAxMC44MzgxNTFdLCBbNDUuODc5OTY5LCAxMC44MzgxMzRdLCBbNDUuODgwMjA0LCAxMC44MzgxNDldLCBbNDUuODgwNDMzLCAxMC44MzgxODNdLCBbNDUuODgwNjU0LCAxMC44MzgyMjddLCBbNDUuODgwOTEzLCAxMC44MzgyMzZdLCBbNDUuODgxMDkxLCAxMC44MzgyNTZdLCBbNDUuODgxNDQxLCAxMC44MzgyNzddLCBbNDUuODgyNDM4LCAxMC44Mzg1MzFdLCBbNDUuODgyNDgyLCAxMC44Mzg0NjZdLCBbNDUuODgyOTY1LCAxMC44Mzg2NV0sIFs0NS44ODMwMDYsIDEwLjgzODgyMV0sIFs0NS44ODMwNiwgMTAuODM4OTE3XSwgWzQ1Ljg4MzEyNiwgMTAuODM4OTYxXSwgWzQ1Ljg4MzM3NSwgMTAuODM4OTkzXSwgWzQ1Ljg4MzQ0MywgMTAuODM5MDkyXSwgWzQ1Ljg4MzU0LCAxMC44MzkyMTldLCBbNDUuODgzNjg2LCAxMC44MzkyODhdLCBbNDUuODgzOTY1LCAxMC44Mzk0ODddLCBbNDUuODg0NDE0LCAxMC44MzkzOTVdLCBbNDUuODg0Mzk0LCAxMC44MzkxMTNdLCBbNDUuODg0NiwgMTAuODM5MDg4XSwgWzQ1Ljg4NDYxNiwgMTAuODM5ODA5XSwgWzQ1Ljg4NDI0LCAxMC44Mzk4NTZdLCBbNDUuODg0MDc0LCAxMC44NDAxOTNdLCBbNDUuODgzOTgzLCAxMC44NDAyMDFdLCBbNDUuODgzOTM1LCAxMC44NDA0MTldLCBbNDUuODgzOTkxLCAxMC44NDA0NjddLCBbNDUuODgzOTM3LCAxMC44NDA3OTldLCBbNDUuODgzOTg1LCAxMC44NDA4MzNdLCBbNDUuODgzOTQxLCAxMC44NDExMzddLCBbNDUuODgzODU0LCAxMC44NDExNDhdLCBbNDUuODgzODU1LCAxMC44NDA3OThdLCBbNDUuODgzNzk4LCAxMC44NDA3OV0sIFs0NS44ODM3ODIsIDEwLjg0MTIzOF0sIFs0NS44ODM4NTUsIDEwLjg0MTI2Nl0sIFs0NS44ODM4NTMsIDEwLjg0MTY1Nl0sIFs0NS44ODQ0MTUsIDEwLjg0MTc3Nl0sIFs0NS44ODQ0MjQsIDEwLjg0MTk3Nl0sIFs0NS44ODQwNiwgMTAuODQxOTYxXSwgWzQ1Ljg4MzkyMSwgMTAuODQxOTAzXSwgWzQ1Ljg4Mzc4MiwgMTAuODQxODUxXSwgWzQ1Ljg4MzYzOSwgMTAuODQxNzk4XSwgWzQ1Ljg4MzQ2OSwgMTAuODQyNTI1XSwgWzQ1Ljg4MzQ3NCwgMTAuODQyNzM4XSwgWzQ1Ljg4NDY0MywgMTAuODQyNzE3XSwgWzQ1Ljg4NDY0OCwgMTAuODQxOTc0XSwgWzQ1Ljg4NDU3LCAxMC44NDE5ODJdLCBbNDUuODg0NTY3LCAxMC44NDE4NThdLCBbNDUuODg0NzQxLCAxMC44NDE4NjVdLCBbNDUuODg0OCwgMTAuODQzMDA5XSwgWzQ1Ljg4MzIyOCwgMTAuODQzMDddLCBbNDUuODgzMjQxLCAxMC44NDMxMzZdLCBbNDUuODgzNTIsIDEwLjg0MzExNV0sIFs0NS44ODM1MDgsIDEwLjg0MzE5XSwgWzQ1Ljg4MzU3NiwgMTAuODQzMjI1XSwgWzQ1Ljg4MzY0MiwgMTAuODQzMzAxXSwgWzQ1Ljg4MzY2OCwgMTAuODQzNDEzXSwgWzQ1Ljg4MzY5NCwgMTAuODQzNTI5XSwgWzQ1Ljg4MzY2OCwgMTAuODQzODA2XSwgWzQ1Ljg4MzU2OSwgMTAuODQzOTQ4XSwgWzQ1Ljg4MzUwNSwgMTAuODQzOTQ0XSwgWzQ1Ljg4MzQyMiwgMTAuODQzOTY1XSwgWzQ1Ljg4MzM4NywgMTAuODQ0MDEzXSwgWzQ1Ljg4MzM3MywgMTAuODQ0MDk4XSwgWzQ1Ljg4MzM0MSwgMTAuODQ0MjU0XSwgWzQ1Ljg4MzI3MywgMTAuODQ0MjZdLCBbNDUuODgzMjc0LCAxMC44NDQzNjRdLCBbNDUuODgzMjAyLCAxMC44NDQ0MzJdLCBbNDUuODgzMDQ4LCAxMC44NDQyNjFdLCBbNDUuODgyODY4LCAxMC44NDQwMDVdLCBbNDUuODgyODk1LCAxMC44NDMyXSwgWzQ1Ljg4Mjc5MiwgMTAuODQzMjA4XSwgWzQ1Ljg4MjgxOSwgMTAuODQzOTE2XSwgWzQ1Ljg4Mjc2NSwgMTAuODQzOTc3XSwgWzQ1Ljg4MzE1NiwgMTAuODQ0NTA5XSwgWzQ1Ljg4MzE0MywgMTAuODQ0NjQ2XSwgWzQ1Ljg4MzIyMiwgMTAuODQ0NjkxXSwgWzQ1Ljg4Mjc4OCwgMTAuODQ1NzUyXSwgWzQ1Ljg4MjY4MywgMTAuODQ1NzZdLCBbNDUuODgyNTEyLCAxMC44NDU2NDhdLCBbNDUuODgyMzcsIDEwLjg0NTY2OF0sIFs0NS44ODIyNDksIDEwLjg0NTY3Ml0sIFs0NS44ODIwMzQsIDEwLjg0NTQ2N10sIFs0NS44ODE5MiwgMTAuODQ1Mzk5XSwgWzQ1Ljg4MTgxMywgMTAuODQ1Mzc3XSwgWzQ1Ljg4MTYzNSwgMTAuODQ1MjkyXSwgWzQ1Ljg4MTQ2NCwgMTAuODQ1MzE4XSwgWzQ1Ljg4MTMyOCwgMTAuODQ1MzQ0XSwgWzQ1Ljg4MTIzLCAxMC44NDUzNjJdLCBbNDUuODgxMTQ1LCAxMC44NDUzOTldLCBbNDUuODgxMSwgMTAuODQ1NDg2XSwgWzQ1Ljg4MTExNywgMTAuODQ1NjA1XSwgWzQ1Ljg4MTExMywgMTAuODQ1Nzc3XSwgWzQ1Ljg4MTExMywgMTAuODQ1Nzg1XSwgWzQ1Ljg4MTI1NywgMTAuODQ2MDczXSwgWzQ1Ljg4MTM5MiwgMTAuODQ2Mjc5XSwgWzQ1Ljg4MTQ1NCwgMTAuODQ2NDU3XSwgWzQ1Ljg4MTU0MSwgMTAuODQ2ODgxXSwgWzQ1Ljg4MTUxNiwgMTAuODQ3MDQ0XSwgWzQ1Ljg4MTUwNSwgMTAuODQ3Mjk2XSwgWzQ1Ljg4MTQyNCwgMTAuODQ3NTY2XSwgWzQ1Ljg4MTI4NywgMTAuODQ4NDA4XSwgWzQ1Ljg4MTI0NiwgMTAuODQ4NjI3XSwgWzQ1Ljg4MTIzNSwgMTAuODQ4ODE2XSwgWzQ1Ljg4MTE4NCwgMTAuODQ5MDgxXSwgWzQ1Ljg4MDk3MSwgMTAuODQ5MjkyXSwgWzQ1Ljg4MDgxMywgMTAuODQ5MzE4XSwgWzQ1Ljg4MDcwMSwgMTAuODQ5Mzg0XSwgWzQ1Ljg4MDU3MSwgMTAuODQ5NDk2XSwgWzQ1Ljg4MDUzMiwgMTAuODQ5NTY1XSwgWzQ1Ljg4MDQ3NiwgMTAuODQ5NjY0XSwgWzQ1Ljg4MDUyLCAxMC44NDk4MzZdLCBbNDUuODgwNTA0LCAxMC44NDk5ODldLCBbNDUuODgwMjg3LCAxMC44NTAwNDNdLCBbNDUuODgwMTg5LCAxMC44NTAxOTJdLCBbNDUuODgwMDcxLCAxMC44NTAzNF0sIFs0NS44Nzk5OCwgMTAuODUwNTAyXSwgWzQ1Ljg3OTkzNCwgMTAuODUwNzJdLCBbNDUuODc5ODc1LCAxMC44NTA4ODNdLCBbNDUuODc5ODExLCAxMC44NTExMDRdLCBbNDUuODc5NzQyLCAxMC44NTEzOTFdLCBbNDUuODc5ODg4LCAxMC44NTE0XSwgWzQ1Ljg3OTkyOSwgMTAuODUxNDZdLCBbNDUuODc5OTA3LCAxMC44NTE1OTFdLCBbNDUuODc5ODg1LCAxMC44NTE2OThdLCBbNDUuODc5NjY1LCAxMC44NTIxNTVdLCBbNDUuODc5NjA4LCAxMC44NTIzODNdLCBbNDUuODc5NTg5LCAxMC44NTI0MzhdLCBbNDUuODc5NTM2LCAxMC44NTIyMjRdLCBbNDUuODc5NTgyLCAxMC44NTE5NzZdLCBbNDUuODc5NTUsIDEwLjg1MTc1XSwgWzQ1Ljg3OTUxLCAxMC44NTE2NDFdLCBbNDUuODc5NDI1LCAxMC44NTE3NTddLCBbNDUuODc5NDI0LCAxMC44NTIwMDldLCBbNDUuODc5Mzk5LCAxMC44NTIyMThdLCBbNDUuODc5Mzc1LCAxMC44NTI1MTJdLCBbNDUuODc5Mjk4LCAxMC44NTI2OTFdLCBbNDUuODc5MjMzLCAxMC44NTI4MzNdLCBbNDUuODc5MTM3LCAxMC44NTMwMTFdLCBbNDUuODc5MDExLCAxMC44NTM2MzFdLCBbNDUuODc4OTE0LCAxMC44NTM5MzZdLCBbNDUuODc4Nzk4LCAxMC44NTQzNl0sIFs0NS44Nzg3NDMsIDEwLjg1NDYxMV0sIFs0NS44Nzg3MDMsIDEwLjg1NDk0XSwgWzQ1Ljg3ODYyNywgMTAuODU1MTY4XSwgWzQ1Ljg3ODU2OCwgMTAuODU1MzI0XSwgWzQ1Ljg3ODQ0OCwgMTAuODU1NDY1XSwgWzQ1Ljg3ODM0NCwgMTAuODU1NTEyXSwgWzQ1Ljg3ODIyMywgMTAuODU1NTU5XSwgWzQ1Ljg3ODE5OCwgMTAuODU1NzA1XSwgWzQ1Ljg3ODMwNSwgMTAuODU1NzZdLCBbNDUuODc4Mzk0LCAxMC44NTU3NTJdLCBbNDUuODc4NDI4LCAxMC44NTU3ODNdLCBbNDUuODc4NTg4LCAxMC44NTY1XSwgWzQ1Ljg3NzczNCwgMTAuODU3MTkyXSwgWzQ1Ljg3NzQ1MywgMTAuODU2NzExXSwgWzQ1Ljg3NzkyMywgMTAuODU1NTU5XSwgWzQ1Ljg3Nzg2OCwgMTAuODU1NTI1XSwgWzQ1Ljg3NzUyOCwgMTAuODU2MzMzXSwgWzQ1Ljg3NzMyNiwgMTAuODU2Mzk3XSwgWzQ1Ljg3NzE1NywgMTAuODU2NTddLCBbNDUuODc2OTkxLCAxMC44NTY3NzldLCBbNDUuODc2ODczLCAxMC44NTY5MDRdLCBbNDUuODc2Nzc2LCAxMC44NTczNTFdLCBbNDUuODc2NjY0LCAxMC44NTc2MzddLCBbNDUuODc2NjIzLCAxMC44NTc4MzVdLCBbNDUuODc2NjA1LCAxMC44NTgwMTJdLCBbNDUuODc2NTM0LCAxMC44NTgyMDRdLCBbNDUuODc2MzY3LCAxMC44NTg0MDldLCBbNDUuODc2MjY1LCAxMC44NTg3MzFdLCBbNDUuODc2MzEyLCAxMC44NTkxMjJdLCBbNDUuODc2Mzc4LCAxMC44NTk1NjldLCBbNDUuODc2NjUyLCAxMC44NjA5NDddLCBbNDUuODc2NjI2LCAxMC44NjEyMzFdLCBbNDUuODc2NTczLCAxMC44NjE0ODJdLCBbNDUuODc2NTY4LCAxMC44NjE3MzFdLCBbNDUuODc2NjUxLCAxMC44NjIwOTZdLCBbNDUuODc2NywgMTAuODYyNDZdLCBbNDUuODc2NjgxLCAxMC44NjI3MjldLCBbNDUuODc2NzIyLCAxMC44NjI5NTJdLCBbNDUuODc2Nzk2LCAxMC44NjMzMDFdLCBbNDUuODc2ODU5LCAxMC44NjM1ODRdLCBbNDUuODc3MDA2LCAxMC44NjQwMzJdLCBbNDUuODc3MDM1LCAxMC44NjQyOTJdLCBbNDUuODc3MDY0LCAxMC44NjQ1NDhdLCBbNDUuODc3MDQ0LCAxMC44NjQ3NTddLCBbNDUuODc3LCAxMC44NjQ5NjJdLCBbNDUuODc2NjQ5LCAxMC44NjU3M10sIFs0NS44NzY0NjUsIDEwLjg2NjAxOF0sIFs0NS44NzYzMTcsIDEwLjg2NjIxNF0sIFs0NS44NzYwOTgsIDEwLjg2NjM0NF0sIFs0NS44NzU5NTMsIDEwLjg2NjQ0Ml0sIFs0NS44NzU3NzUsIDEwLjg2NjY0MV0sIFs0NS44NzU1MzUsIDEwLjg2NjgxOV0sIFs0NS44NzUzMzcsIDEwLjg2Njk2OV0sIFs0NS44NzUxODMsIDEwLjg2Njk5NV0sIFs0NS44NzUwMDMsIDEwLjg2NjkzMl0sIFs0NS44NzQ4MDgsIDEwLjg2NjgxM10sIFs0NS44NzQ1MjEsIDEwLjg2Njg2Ml0sIFs0NS44NzQzODIsIDEwLjg2NjgzM10sIFs0NS44NzQyNDksIDEwLjg2NjYwNF0sIFs0NS44NzQxNDIsIDEwLjg2NjM4M10sIFs0NS44NzQwNDMsIDEwLjg2NjI2OV0sIFs0NS44NzM5NTYsIDEwLjg2NjI0OF0sIFs0NS44NzM4ODYsIDEwLjg2NjExNV0sIFs0NS44NzM5MTEsIDEwLjg2NTk1NV0sIFs0NS44NzQwMjcsIDEwLjg2NTc5N10sIFs0NS44NzM5MDUsIDEwLjg2NTYyNF0sIFs0NS44NzM3NiwgMTAuODY1NzM2XSwgWzQ1Ljg3MzY2NywgMTAuODY1OTE3XSwgWzQ1Ljg3MzU2MywgMTAuODY2MjE5XSwgWzQ1Ljg3MzQ5NCwgMTAuODY2NDU3XSwgWzQ1Ljg3MzQ2OCwgMTAuODY2NTA5XSwgWzQ1Ljg3MzQzNSwgMTAuODY2NTE1XSwgWzQ1Ljg3MzAxOSwgMTAuODY2NTk1XSwgWzQ1Ljg3MjY4OSwgMTAuODY2NjU4XSwgWzQ1Ljg3MjY2NywgMTAuODY2NjU3XSwgWzQ1Ljg3MjU2NSwgMTAuODY2NjUyXSwgWzQ1Ljg3MjQ1OCwgMTAuODY2NjI3XSwgWzQ1Ljg3MjM4NywgMTAuODY2NjE5XSwgWzQ1Ljg3MjQyOSwgMTAuODY2OTUxXSwgWzQ1Ljg3MjUyOSwgMTAuODY3MDg0XSwgWzQ1Ljg3MjU2NywgMTAuODY3MTU3XSwgWzQ1Ljg3MjU1NywgMTAuODY3MjU4XSwgWzQ1Ljg3MjQyOCwgMTAuODY3MjgxXSwgWzQ1Ljg3MjM3LCAxMC44NjczODVdLCBbNDUuODcyMzk4LCAxMC44Njc1MjZdLCBbNDUuODcyNDY2LCAxMC44Njc3OTZdLCBbNDUuODcyNDg1LCAxMC44NjgxODZdLCBbNDUuODcyNDA2LCAxMC44Njg1NDFdLCBbNDUuODcyMzQzLCAxMC44Njg4MjVdLCBbNDUuODcyMjQ0LCAxMC44NjkwODVdLCBbNDUuODcyMTI4LCAxMC44Njk0NzVdLCBbNDUuODcxOTUxLCAxMC44Njk3ODVdLCBbNDUuODcxNjgsIDEwLjg3MDEwN10sIFs0NS44NzE0MjEsIDEwLjg3MDM1XSwgWzQ1Ljg3MTEzOCwgMTAuODcwNzE3XSwgWzQ1Ljg3MDkyMywgMTAuODcwOTU0XSwgWzQ1Ljg3MDc5NCwgMTAuODcxMDY2XSwgWzQ1Ljg3MDczNSwgMTAuODcxMjA2XSwgWzQ1Ljg3MDk2NCwgMTAuODcxNjA3XSwgWzQ1Ljg3MDg4OSwgMTAuODcxNzc5XSwgWzQ1Ljg3MDc5MiwgMTAuODcxNjc4XSwgWzQ1Ljg3MDY1MywgMTAuODcxNjQ2XSwgWzQ1Ljg3MDYxMSwgMTAuODcxMDI2XSwgWzQ1Ljg3MDU0NCwgMTAuODcxMDU3XSwgWzQ1Ljg3MDU3NywgMTAuODcxNjI1XSwgWzQ1Ljg3MDI1MywgMTAuODcyMjIzXSwgWzQ1Ljg3MDE3MywgMTAuODcyMTk1XSwgWzQ1Ljg3MDEzMSwgMTAuODcyMjc2XSwgWzQ1Ljg3MDE4MywgMTAuODcyMzQ2XSwgWzQ1Ljg2OTk2NywgMTAuODcyNjY5XSwgWzQ1Ljg2OTg5NCwgMTAuODcyNjQ3XSwgWzQ1Ljg2OTgxMiwgMTAuODcyODI2XSwgWzQ1Ljg2OTgzMiwgMTAuODcyODU1XSwgWzQ1Ljg2OTE0MSwgMTAuODc0MTk5XSwgWzQ1Ljg2OTA3NSwgMTAuODc0MTUyXSwgWzQ1Ljg2OTAxNiwgMTAuODc0MzExXSwgWzQ1Ljg2ODkxNiwgMTAuODc0MjY2XSwgWzQ1Ljg2ODgyNiwgMTAuODc0NTE2XSwgWzQ1Ljg2ODg3NCwgMTAuODc0NTM3XSwgWzQ1Ljg2ODg4MiwgMTAuODc0Njc4XSwgWzQ1Ljg2ODc4MSwgMTAuODc0OTI0XSwgWzQ1Ljg2ODU5LCAxMC44NzUzOThdLCBbNDUuODY4NzE4LCAxMC44NzU4NjldLCBbNDUuODY4NjU4LCAxMC44NzYxMDddLCBbNDUuODY4NjE2LCAxMC44NzYyMzNdLCBbNDUuODY4NTY5LCAxMC44NzYyOThdLCBbNDUuODY4NTM0LCAxMC44NzYzNzldLCBbNDUuODY4NjUxLCAxMC44NzYzOTVdLCBbNDUuODY4NzA3LCAxMC44NzYzMDFdLCBbNDUuODY4NzYyLCAxMC44NzYyMjRdLCBbNDUuODY4OTA2LCAxMC44NzYwNDNdLCBbNDUuODY5MDgxLCAxMC44NzYxNjJdLCBbNDUuODY4NTE2LCAxMC44NzY2MTFdLCBbNDUuODY4NDk2LCAxMC44NzY1MDZdLCBbNDUuODY4NDc2LCAxMC44NzY0NjZdLCBbNDUuODY4NDM3LCAxMC44NzY0NjJdLCBbNDUuODY4MzUzLCAxMC44NzY1OThdLCBbNDUuODY4MzI2LCAxMC44NzY2MDddLCBbNDUuODY4MjEzLCAxMC44NzY2OTZdLCBbNDUuODY4MTE5LCAxMC44NzY3NF0sIFs0NS44Njc5NzgsIDEwLjg3Njc5OV0sIFs0NS44Njc4MjcsIDEwLjg3NjgzNV0sIFs0NS44Njc2NzEsIDEwLjg3Njg4NF0sIFs0NS44Njc1NzMsIDEwLjg3Njg4NV0sIFs0NS44Njc0MzEsIDEwLjg3Njg4OF0sIFs0NS44NjczMzIsIDEwLjg3Njg4Nl0sIFs0NS44NjcyMjUsIDEwLjg3Njg4XSwgWzQ1Ljg2NjQ5NSwgMTAuODc2NDY4XSwgWzQ1Ljg2NjE3NiwgMTAuODc2MTZdLCBbNDUuODY1ODAzLCAxMC44NzU5MTZdLCBbNDUuODY1NTIsIDEwLjg3NTcxXSwgWzQ1Ljg2NTMwNCwgMTAuODc1NTM4XSwgWzQ1Ljg2NTEzNiwgMTAuODc1NDJdLCBbNDUuODY1LCAxMC44NzUzMzJdLCBbNDUuODY0ODk1LCAxMC44NzUyODRdLCBbNDUuODY0NzIsIDEwLjg3NTIyN10sIFs0NS44NjQ1NTcsIDEwLjg3NTIxMV0sIFs0NS44NjQzNDksIDEwLjg3NTIyMl0sIFs0NS44NjQxNzksIDEwLjg3NTI3NF0sIFs0NS44NjQwMzYsIDEwLjg3NTM3M10sIFs0NS44NjM5LCAxMC44NzU0OTRdLCBbNDUuODYzNzY2LCAxMC44NzU2NThdLCBbNDUuODYzNjI3LCAxMC44NzU3NjZdLCBbNDUuODYzNTM1LCAxMC44NzU4NTNdLCBbNDUuODYzMDM2LCAxMC44NzYxMTNdLCBbNDUuODYyOTA1LCAxMC44NzYxM10sIFs0NS44NjI4MTYsIDEwLjg3NjE2N10sIFs0NS44NjI3MTksIDEwLjg3NjE2NV0sIFs0NS44NjI1NzcsIDEwLjg3NTQwNl0sIFs0NS44NjI2NzUsIDEwLjg3NTQyMV0sIFs0NS44NjI2NjQsIDEwLjg3NTM1OV0sIFs0NS44NjI1OTYsIDEwLjg3NTI4NV0sIFs0NS44NjI0NDQsIDEwLjg3NTIyM10sIFs0NS44NjIzNzYsIDEwLjg3NTE1Nl0sIFs0NS44NjIxNzksIDEwLjg3NTEyNV0sIFs0NS44NjE5NzksIDEwLjg3NTE5OV0sIFs0NS44NjE4OTEsIDEwLjg3NTM2NF0sIFs0NS44NjE3OTMsIDEwLjg3NTUyNl0sIFs0NS44NjE1OTgsIDEwLjg3NTU4N10sIFs0NS44NjE0ODYsIDEwLjg3NTU5OF0sIFs0NS44NjEzNDEsIDEwLjg3NTY0NF0sIFs0NS44NjEyNDQsIDEwLjg3NTcyM10sIFs0NS44NjExNzMsIDEwLjg3NTc3MV0sIFs0NS44NjEwODgsIDEwLjg3NTc1M10sIFs0NS44NjA3ODQsIDEwLjg3NTI4NF0sIFs0NS44NjA2NjcsIDEwLjg3NDg5OV0sIFs0NS44NjA1MDksIDEwLjg3NDcyMl0sIFs0NS44NjAzNTEsIDEwLjg3NDUxNV0sIFs0NS44NjAxNDgsIDEwLjg3NDI1Nl0sIFs0NS44NTk5NjcsIDEwLjg3NDA1Ml0sIFs0NS44NTk4MDUsIDEwLjg3Mzg1Ml0sIFs0NS44NTk2MjcsIDEwLjg3MzU4Nl0sIFs0NS44NTk0NDksIDEwLjg3MzM1XSwgWzQ1Ljg1OTI1OCwgMTAuODcyOTkyXSwgWzQ1Ljg1OTAzNSwgMTAuODcyNjc5XSwgWzQ1Ljg1ODc2OSwgMTAuODcyMzFdLCBbNDUuODU4NTgsIDEwLjg3MjAxMV0sIFs0NS44NTgzMDgsIDEwLjg3MTU3M10sIFs0NS44NTgwODMsIDEwLjg3MTM5OF0sIFs0NS44NTc4MDUsIDEwLjg3MTA3NV0sIFs0NS44NTc1MDEsIDEwLjg3MDgyOV0sIFs0NS44NTcxOTQsIDEwLjg3MDY0OV0sIFs0NS44NTY5NjcsIDEwLjg3MDVdLCBbNDUuODU2NzUyLCAxMC44NzAzNTFdLCBbNDUuODU2NjM2LCAxMC44NzAyNV0sIFs0NS44NTYzNTUsIDEwLjg2OTk2OV0sIFs0NS44NTYxMjUsIDEwLjg2OTY3Nl0sIFs0NS44NTU4OTQsIDEwLjg2OTQ2NV0sIFs0NS44NTU2NCwgMTAuODY5MjY5XSwgWzQ1Ljg1NTQ1NSwgMTAuODY5MDQ5XSwgWzQ1Ljg1NTMwOCwgMTAuODY4ODU2XSwgWzQ1Ljg1NTAxOSwgMTAuODY4NTI5XSwgWzQ1Ljg1NDc2MywgMTAuODY4MjYyXSwgWzQ1Ljg1NDU2OCwgMTAuODY4MDc3XSwgWzQ1Ljg1NDM5NiwgMTAuODY3OTMzXSwgWzQ1Ljg1NDIyOSwgMTAuODY3NzI2XSwgWzQ1Ljg1NDA4MSwgMTAuODY3NDc3XSwgWzQ1Ljg1Mzg3NSwgMTAuODY3MjM0XSwgWzQ1Ljg1MzYyMSwgMTAuODY2OTk5XSwgWzQ1Ljg1MzM2LCAxMC44NjY4MTddLCBbNDUuODUzMTI5LCAxMC44NjY1ODldLCBbNDUuODUyOTYzLCAxMC44NjYzMTddLCBbNDUuODUyODUxLCAxMC44NjYwNDNdLCBbNDUuODUyNzEsIDEwLjg2NTU3OV0sIFs0NS44NTI1NTksIDEwLjg2NTM0Nl0sIFs0NS44NTI0MzEsIDEwLjg2NTE3XSwgWzQ1Ljg1MjMyMywgMTAuODY0OTkxXSwgWzQ1Ljg1MjE1MywgMTAuODY0NzY4XSwgWzQ1Ljg1MTg0MiwgMTAuODY0Mzk0XSwgWzQ1Ljg1MTU3MiwgMTAuODY0MTQzXSwgWzQ1Ljg1MTI3MSwgMTAuODYzODQyXSwgWzQ1Ljg1MTA1NiwgMTAuODYzNjZdLCBbNDUuODUwODcyLCAxMC44NjM0NzZdLCBbNDUuODUwNzIzLCAxMC44NjMzNTldLCBbNDUuODUwNTQ4LCAxMC44NjMyNDddLCBbNDUuODUwNDQxLCAxMC44NjMxNzZdLCBbNDUuODUwMTg2LCAxMC44NjMwMjNdLCBbNDUuODQ5ODY1LCAxMC44NjI4NzVdLCBbNDUuODQ5Njk5LCAxMC44NjI3ODZdLCBbNDUuODQ5NTcsIDEwLjg2MjcwMl0sIFs0NS44NDk0MzQsIDEwLjg2MjU4MV0sIFs0NS44NDkyNjksIDEwLjg2MjM3OF0sIFs0NS44NDkyMTUsIDEwLjg2MjMxNF0sIFs0NS44NDkxODYsIDEwLjg2MjI5OF0sIFs0NS44NDkxMDcsIDEwLjg2MjE3Nl0sIFs0NS44NDkxMDcsIDEwLjg2MjE4MV0sIFs0NS44NDkwOTMsIDEwLjg2MjE3MV0sIFs0NS44NDkwNTcsIDEwLjg2MjE0OF0sIFs0NS44NDkwMiwgMTAuODYyMTU4XSwgWzQ1Ljg0OTAxNSwgMTAuODYyMjY1XSwgWzQ1Ljg0OTAwNCwgMTAuODYyMzMxXSwgWzQ1Ljg0ODk3MywgMTAuODYyNDEyXSwgWzQ1Ljg0ODk0OCwgMTAuODYyNDMxXSwgWzQ1Ljg0ODg4NywgMTAuODYyNDIxXSwgWzQ1Ljg0ODg1LCAxMC44NjI0M10sIFs0NS44NDg4MjcsIDEwLjg2MjQ2OV0sIFs0NS44NDg4MTMsIDEwLjg2MjU2N10sIFs0NS44NDg4MDUsIDEwLjg2MjY2Ml0sIFs0NS44NDg2ODksIDEwLjg2Mjk4OF0sIFs0NS44NDg2MjUsIDEwLjg2MzEwMl0sIFs0NS44NDg1NDIsIDEwLjg2MzIzMl0sIFs0NS44NDg0NzUsIDEwLjg2MzMxM10sIFs0NS44NDg0MDQsIDEwLjg2MzM3NF0sIFs0NS44NDgzMTcsIDEwLjg2MzQzMl0sIFs0NS44NDgyMjMsIDEwLjg2MzQ5M10sIFs0NS44NDgxNSwgMTAuODYzNTQxXSwgWzQ1Ljg0ODA2MywgMTAuODYzNTg2XSwgWzQ1Ljg0ODAwMywgMTAuODYzNjA1XSwgWzQ1Ljg0Nzk0OCwgMTAuODYzNjE0XSwgWzQ1Ljg0Nzg2MiwgMTAuODYzNTcxXSwgWzQ1Ljg0Nzc5MSwgMTAuODYzNTExXSwgWzQ1Ljg0NzczLCAxMC44NjM0MzVdLCBbNDUuODQ3NjU3LCAxMC44NjMzNDNdLCBbNDUuODQ3NTk4LCAxMC44NjMyNzddLCBbNDUuODQ3NTE4LCAxMC44NjMyMTddLCBbNDUuODQ3NDQ4LCAxMC44NjMyMDNdLCBbNDUuODQ3Mzc1LCAxMC44NjMyMDldLCBbNDUuODQ3MjgxLCAxMC44NjMyMTFdLCBbNDUuODQ3MTU1LCAxMC44NjMyMTZdLCBbNDUuODQ2NzQ0LCAxMC44NjMxNDZdLCBbNDUuODQ2NjYyLCAxMC44NjMwOTldLCBbNDUuODQ2NTU4LCAxMC44NjMwMzldLCBbNDUuODQ2NDU1LCAxMC44NjI5NzNdLCBbNDUuODQ2Mzg1LCAxMC44NjI5MjNdLCBbNDUuODQ2MjY2LCAxMC44NjI4NDNdLCBbNDUuODQ2MTU1LCAxMC44NjI3NTNdLCBbNDUuODQ2MDM5LCAxMC44NjI2NTddLCBbNDUuODQ1OTYxLCAxMC44NjI1ODhdLCBbNDUuODQ1NzgyLCAxMC44NjI0MzJdLCBbNDUuODQ1NzEzLCAxMC44NjI0MTVdLCBbNDUuODQ1NjM4LCAxMC44NjI0M10sIFs0NS44NDU1NTMsIDEwLjg2MjQ2Ml0sIFs0NS44NDU0NjYsIDEwLjg2MjQ5N10sIFs0NS44NDUzNTIsIDEwLjg2MjUzNV0sIFs0NS44NDUyMSwgMTAuODYyNTg2XSwgWzQ1Ljg0NTE1OSwgMTAuODYyNjE1XSwgWzQ1Ljg0NTEyNywgMTAuODYyNjQ0XSwgWzQ1Ljg0NTA5LCAxMC44NjI3MDJdLCBbNDUuODQ1MDc5LCAxMC44NjI3NTRdLCBbNDUuODQ1MDc0LCAxMC44NjI4M10sIFs0NS44NDUwNzEsIDEwLjg2Mjg5OF0sIFs0NS44NDUwNjgsIDEwLjg2Mjk4M10sIFs0NS44NDUwNjMsIDEwLjg2MzA2NV0sIFs0NS44NDUwNjUsIDEwLjg2MzE1XSwgWzQ1Ljg0NTA2MywgMTAuODYzMjEyXSwgWzQ1Ljg0NTA0OSwgMTAuODYzMjU4XSwgWzQ1Ljg0NTAxLCAxMC44NjMzMTZdLCBbNDUuODQ0OTY0LCAxMC44NjMzNDVdLCBbNDUuODQ0ODY1LCAxMC44NjMzNzNdLCBbNDUuODQ0NzQ0LCAxMC44NjM0MTVdLCBbNDUuODQ0NTc3LCAxMC44NjM0NzVdLCBbNDUuODQ0NDkyLCAxMC44NjM0OTNdLCBbNDUuODQ0Mzc2LCAxMC44NjM0NTldLCBbNDUuODQ0MjU1LCAxMC44NjMzODZdLCBbNDUuODQ0MTI2LCAxMC44NjMyNjddLCBbNDUuODQ0MDA4LCAxMC44NjMxNDhdLCBbNDUuODQzODkyLCAxMC44NjMwMjJdLCBbNDUuODQzNzY3LCAxMC44NjI4ODRdLCBbNDUuODQzNjMzLCAxMC44NjI3NDhdLCBbNDUuODQzNTM1LCAxMC44NjI2MzldLCBbNDUuODQzNDQsIDEwLjg2MjU1Nl0sIFs0NS44NDMzMzMsIDEwLjg2MjQ5XSwgWzQ1Ljg0MzIyNSwgMTAuODYyNDU5XSwgWzQ1Ljg0MzEyLCAxMC44NjI0NTRdLCBbNDUuODQzMDA2LCAxMC44NjI0Nl0sIFs0NS44NDI5MTksIDEwLjg2MjQ2OV0sIFs0NS44NDI3OTgsIDEwLjg2MjQ1NF0sIFs0NS44NDI3MDMsIDEwLjg2MjM4MV0sIFs0NS44NDI1NjYsIDEwLjg2MjMwMV0sIFs0NS44NDI0MzQsIDEwLjg2MjIxOF0sIFs0NS44NDIzNTIsIDEwLjg2MjE2MV0sIFs0NS44NDIxODgsIDEwLjg2MjA1Ml0sIFs0NS44NDIwNDksIDEwLjg2MTkzNl0sIFs0NS44NDE5MTcsIDEwLjg2MTgxM10sIFs0NS44NDE3OTcsIDEwLjg2MTcwNF0sIFs0NS44NDE2OTIsIDEwLjg2MTYxNF0sIFs0NS44NDE2MDEsIDEwLjg2MTU2MV0sIFs0NS44NDE0NzQsIDEwLjg2MTQ5NF0sIFs0NS44NDEzNDgsIDEwLjg2MTQyNF0sIFs0NS44NDEyNDgsIDEwLjg2MTM2MV0sIFs0NS44NDExNTMsIDEwLjg2MTI4MV0sIFs0NS44NDEwNjIsIDEwLjg2MTE4OV0sIFs0NS44NDA5OCwgMTAuODYxMTE2XSwgWzQ1Ljg0MDg3MSwgMTAuODYxMDJdLCBbNDUuODQwNzYxLCAxMC44NjA5MzNdLCBbNDUuODQwNjIzLCAxMC44NjA4NDRdLCBbNDUuODQwNDc5LCAxMC44NjA3N10sIFs0NS44NDAzNjMsIDEwLjg2MDczXSwgWzQ1Ljg0MDE3NiwgMTAuODYwNjYyXSwgWzQ1Ljg0MDEwMywgMTAuODYwNjM1XSwgWzQ1Ljg0MDA1NSwgMTAuODYwNjE1XSwgWzQ1Ljg0MDAwNywgMTAuODYwNTkyXSwgWzQ1LjgzOTk2OCwgMTAuODYwNTY4XSwgWzQ1LjgzOTkzNCwgMTAuODYwNTM4XSwgWzQ1LjgzOTg5NiwgMTAuODYwNDM3XSwgWzQ1LjgzOTg5OCwgMTAuODYwMzg4XSwgWzQ1LjgzOTkxLCAxMC44NjAzMzJdLCBbNDUuODM5OTI2LCAxMC44NjAyOF0sIFs0NS44Mzk5NDMsIDEwLjg2MDIzMV0sIFs0NS44Mzk5NTIsIDEwLjg2MDE4NV0sIFs0NS44Mzk5NTksIDEwLjg2MDE0Nl0sIFs0NS44Mzk5NjIsIDEwLjg2MDEwN10sIFs0NS44Mzk5NTUsIDEwLjg2MDA0OF0sIFs0NS44Mzk5MzcsIDEwLjg1OTk5Ml0sIFs0NS44Mzk4OTIsIDEwLjg1OTkyM10sIFs0NS44Mzk4NDksIDEwLjg1OTg4XSwgWzQ1LjgzOTc4NSwgMTAuODU5ODRdLCBbNDUuODM5NzAxLCAxMC44NTk4MDNdLCBbNDUuODM5NTgyLCAxMC44NTk3NjldLCBbNDUuODM5NDg2LCAxMC44NTk3NDVdLCBbNDUuODM5Mzk1LCAxMC44NTk3MzRdLCBbNDUuODM5Mjk3LCAxMC44NTk3Ml0sIFs0NS44MzkyMjgsIDEwLjg1OTcwM10sIFs0NS44MzkxNDYsIDEwLjg1OTY4M10sIFs0NS44MzkwNDgsIDEwLjg1OTY2Ml0sIFs0NS44Mzg5OTgsIDEwLjg1OTY1NV0sIFs0NS44Mzg4ODgsIDEwLjg1OTYzMV0sIFs0NS44Mzg2ODksIDEwLjg1OTYzOF0sIFs0NS44Mzg1NzcsIDEwLjg1OTY0N10sIFs0NS44Mzg0NjMsIDEwLjg1OTY0Nl0sIFs0NS44MzgyNDQsIDEwLjg1OTYwN10sIFs0NS44MzgwOTgsIDEwLjg1OTU2XSwgWzQ1LjgzNzk3NSwgMTAuODU5NTIyXSwgWzQ1LjgzNzgyMiwgMTAuODU5NDYyXSwgWzQ1LjgzNzcwNiwgMTAuODU5NDI1XSwgWzQ1LjgzNzYxLCAxMC44NTkzOTFdLCBbNDUuODM3NDcxLCAxMC44NTkzMjRdLCBbNDUuODM3MzkxLCAxMC44NTkyNjddLCBbNDUuODM3MzA1LCAxMC44NTkyMDhdLCBbNDUuODM3MjE2LCAxMC44NTkxMzVdLCBbNDUuODM3MTE4LCAxMC44NTkwNDhdLCBbNDUuODM3MDQzLCAxMC44NTg5NjNdLCBbNDUuODM2OTY0LCAxMC44NTg4NzddLCBbNDUuODM2NzY2LCAxMC44NTg3NDddLCBbNDUuODM2NjM2LCAxMC44NTg3MzJdLCBbNDUuODM2NTM1LCAxMC44NTg3NDFdLCBbNDUuODM2Mjg5LCAxMC44NTg3MDZdLCBbNDUuODM2MTU0LCAxMC44NTg2NjhdLCBbNDUuODM2MDMxLCAxMC44NTg2NDFdLCBbNDUuODM1OTEsIDEwLjg1ODYxNl0sIFs0NS44MzU3OTMsIDEwLjg1ODU5Nl0sIFs0NS44MzU2ODgsIDEwLjg1ODU3MV0sIFs0NS44MzU1MiwgMTAuODU4NTI0XSwgWzQ1LjgzNTQ3NCwgMTAuODU4NTA0XSwgWzQ1LjgzNTM2MiwgMTAuODU4NDg2XSwgWzQ1LjgzNTMxNCwgMTAuODU4NTA1XSwgWzQ1LjgzNTI2OCwgMTAuODU4NTQ0XSwgWzQ1LjgzNTIyMiwgMTAuODU4NjEyXSwgWzQ1LjgzNTE5MiwgMTAuODU4Njc3XSwgWzQ1LjgzNTE2OSwgMTAuODU4NzU1XSwgWzQ1LjgzNTE2NiwgMTAuODU4ODE0XSwgWzQ1LjgzNTE2NCwgMTAuODU4ODczXSwgWzQ1LjgzNTE1OSwgMTAuODU4OTMyXSwgWzQ1LjgzNTE0NSwgMTAuODU4OTk3XSwgWzQ1LjgzNTEyMiwgMTAuODU5MDYyXSwgWzQ1LjgzNTA5NCwgMTAuODU5MTI0XSwgWzQ1LjgzNTA0NiwgMTAuODU5MTk5XSwgWzQ1LjgzNDk5LCAxMC44NTkyOTNdLCBbNDUuODM0OTY5LCAxMC44NTkzNThdLCBbNDUuODM0OTU4LCAxMC44NTk0MV0sIFs0NS44MzQ5NDYsIDEwLjg1OTQ0OV0sIFs0NS44MzQ5MTQsIDEwLjg1OTQ5OF0sIFs0NS44MzQ4NjMsIDEwLjg1OTUzN10sIFs0NS44MzQ3OTcsIDEwLjg1OTU3Ml0sIFs0NS44MzQ3MTUsIDEwLjg1OTU4MV0sIFs0NS44MzQ2MjEsIDEwLjg1OTU3XSwgWzQ1LjgzNDUzMiwgMTAuODU5NTQzXSwgWzQ1LjgzNDQ3NywgMTAuODU5NTMyXSwgWzQ1LjgzNDM3LCAxMC44NTk1NjddLCBbNDUuODM0MzMxLCAxMC44NTk2MDldLCBbNDUuODM0Mjg5LCAxMC44NTk2NDhdLCBbNDUuODM0MjY0LCAxMC44NTk2OTddLCBbNDUuODM0MjUyLCAxMC44NTk3MzZdLCBbNDUuODM0MjI5LCAxMC44NTk4MTRdLCBbNDUuODM0MTg1LCAxMC44NTk4ODldLCBbNDUuODM0MTI1LCAxMC44NTk5NjddLCBbNDUuODM0MDYxLCAxMC44NjAwMzFdLCBbNDUuODM0MDI0LCAxMC44NjAwNjRdLCBbNDUuODMzOTgzLCAxMC44NjAwODldLCBbNDUuODMzOTE0LCAxMC44NjAxMDVdLCBbNDUuODMzODYyLCAxMC44NjAxMDRdLCBbNDUuODMzODAzLCAxMC44NjAwNzhdLCBbNDUuODMzNzUzLCAxMC44NjAwNDRdLCBbNDUuODMzNjg0LCAxMC44NjAwMDRdLCBbNDUuODMzNjE0LCAxMC44NTk5NjhdLCBbNDUuODMzNTM2LCAxMC44NTk5MzddLCBbNDUuODMzNDU0LCAxMC44NTk5MDRdLCBbNDUuODMzMjk5LCAxMC44NTk4M10sIFs0NS44MzMxNzYsIDEwLjg1OTc5OV0sIFs0NS44MzMwNDYsIDEwLjg1OTc2NV0sIFs0NS44MzI5MDcsIDEwLjg1OTcyMV0sIFs0NS44MzI4MDQsIDEwLjg1OTY4MV0sIFs0NS44MzI2NzksIDEwLjg1OTYyN10sIFs0NS44MzI1NDksIDEwLjg1OTU3M10sIFs0NS44MzI0NDQsIDEwLjg1OTU0Ml0sIFs0NS44MzIzMzQsIDEwLjg1OTQ5NV0sIFs0NS44MzIxODksIDEwLjg1OTQxOV0sIFs0NS44MzIwOTMsIDEwLjg1OTM2Ml0sIFs0NS44MzIwMDIsIDEwLjg1OTI4OV0sIFs0NS44MzE5MjksIDEwLjg1OTIyOV0sIFs0NS44MzE4MjksIDEwLjg1OTE0M10sIFs0NS44MzE3MjQsIDEwLjg1OTA2NF0sIFs0NS44MzE2NTYsIDEwLjg1OTAxN10sIFs0NS44MzE1OSwgMTAuODU4OTY3XSwgWzQ1LjgzMTQ5LCAxMC44NTg4ODRdLCBbNDUuODMxMzc0LCAxMC44NTg3ODhdLCBbNDUuODMxMjE5LCAxMC44NTg2NjldLCBbNDUuODMxMDgzLCAxMC44NTg1NzNdLCBbNDUuODMwOTQ2LCAxMC44NTg0ODldLCBbNDUuODMwODA5LCAxMC44NTg0MTldLCBbNDUuODMwNjIsIDEwLjg1ODMyOV0sIFs0NS44MzA0NTksIDEwLjg1ODIzMl0sIFs0NS44MzAzMDgsIDEwLjg1ODE0NV0sIFs0NS44MzAxNiwgMTAuODU4MDYyXSwgWzQ1LjgyOTk5NCwgMTAuODU3OTUyXSwgWzQ1LjgyOTg0NCwgMTAuODU3ODY5XSwgWzQ1LjgyOTY5OCwgMTAuODU3Nzg2XSwgWzQ1LjgyOTU2NiwgMTAuODU3NzAyXSwgWzQ1LjgyOTQxNiwgMTAuODU3NTkzXSwgWzQ1LjgyOTMwMiwgMTAuODU3NTI2XSwgWzQ1LjgyOTE0LCAxMC44NTc0NDldLCBbNDUuODI5MDA2LCAxMC44NTczODVdLCBbNDUuODI4NzIxLCAxMC44NTcxOTZdLCBbNDUuODI4NDc1LCAxMC44NTcwMV0sIFs0NS44MjgzNTIsIDEwLjg1NjkyN10sIFs0NS44MjgxOTgsIDEwLjg1NjgxMV0sIFs0NS44Mjc5ODEsIDEwLjg1NjY5NF0sIFs0NS44Mjc5MTMsIDEwLjg1NjY2MV0sIFs0NS44Mjc3OTIsIDEwLjg1NjYxM10sIFs0NS44Mjc3MzMsIDEwLjg1NjU5M10sIFs0NS44Mjc1NjEsIDEwLjg1NjU1NV0sIFs0NS44Mjc0NTIsIDEwLjg1NjU3XSwgWzQ1LjgyNzQwNiwgMTAuODU2NTkzXSwgWzQ1LjgyNzM1OCwgMTAuODU2NjQxXSwgWzQ1LjgyNzMsIDEwLjg1NjcwOV0sIFs0NS44MjcxOTQsIDEwLjg1NjgzNl0sIFs0NS44MjcwODIsIDEwLjg1Njk0Ml0sIFs0NS44MjcwMTQsIDEwLjg1NzAwOF0sIFs0NS44MzU0NzcsIDEwLjgzMjM0MV0sIFs0NS44Mzc2MywgMTAuODI1ODUzXV1dXSwgInR5cGUiOiAiTXVsdGlQb2x5Z29uIn0sICJpZCI6ICI3IiwgInByb3BlcnRpZXMiOiB7ImRlc2NyaXppb25lX29yaWdpbmUiOiAiTm9uIGNvZGlmaWNhdG8iLCAiZ21sX2lkIjogIklELkFDUVVFRklTSUNIRS5TUEVDQ0hJLkFDUVVBLjU0MTgzIiwgIm5hc3AiOiAwLCAibmF0dXJhX3NwZWNjaGlvX2FjcXVhIjogIk5vbiBjb2RpZmljYXRvIiwgIm5vbWUiOiAiTEFHTyBESSBHQVJEQSIsICJvcmlnaW5lIjogMH0sICJ0eXBlIjogIkZlYXR1cmUifSwgeyJnZW9tZXRyeSI6IHsiY29vcmRpbmF0ZXMiOiBbW1tbNDYuMDU5MTE0LCAxMC45Nzc0MTddLCBbNDYuMDU5MTA0LCAxMC45Nzc1ODhdLCBbNDYuMDU5MDkyLCAxMC45Nzc3NzldLCBbNDYuMDU5MDQ2LCAxMC45Nzc3NzhdLCBbNDYuMDU4OTk2LCAxMC45Nzc3ODNdLCBbNDYuMDU4OTUsIDEwLjk3Nzc5OF0sIFs0Ni4wNTg3OTQsIDEwLjk3NzgyNF0sIFs0Ni4wNTg3NDIsIDEwLjk3NzgwM10sIFs0Ni4wNTg3MDEsIDEwLjk3Nzc3MV0sIFs0Ni4wNTg2NDksIDEwLjk3NzczMl0sIFs0Ni4wNTg1MjEsIDEwLjk3NzU4OF0sIFs0Ni4wNTg0NTEsIDEwLjk3NzQ3N10sIFs0Ni4wNTgzOSwgMTAuOTc3NDA0XSwgWzQ2LjA1ODMxNSwgMTAuOTc3MzQ2XSwgWzQ2LjA1ODIwNCwgMTAuOTc3MjYxXSwgWzQ2LjA1ODExNiwgMTAuOTc3MThdLCBbNDYuMDU4MDAxLCAxMC45NzcwNTldLCBbNDYuMDU3OTIsIDEwLjk3NjkzNl0sIFs0Ni4wNTc4ODIsIDEwLjk3Njg2Ml0sIFs0Ni4wNTc4NjIsIDEwLjk3NjgzMl0sIFs0Ni4wNTc3OTcsIDEwLjk3Njc3OF0sIFs0Ni4wNTc3NTYsIDEwLjk3Njc0OF0sIFs0Ni4wNTc3MTMsIDEwLjk3NjcyN10sIFs0Ni4wNTc2NTMsIDEwLjk3NjcxOV0sIFs0Ni4wNTc1ODcsIDEwLjk3NjcwMV0sIFs0Ni4wNTc1MzMsIDEwLjk3NjY2XSwgWzQ2LjA1NzQ2MywgMTAuOTc2NTUzXSwgWzQ2LjA1NzQwNCwgMTAuOTc2NDAxXSwgWzQ2LjA1NzM1LCAxMC45NzYyNjhdLCBbNDYuMDU3MjcyLCAxMC45NzYxMzFdLCBbNDYuMDU3MTg5LCAxMC45NzYwMTRdLCBbNDYuMDU3MTE5LCAxMC45NzU5MDRdLCBbNDYuMDU3MDc3LCAxMC45NzU3OThdLCBbNDYuMDU3MDM5LCAxMC45NzU3MDJdLCBbNDYuMDU3MDI5LCAxMC45NzU1OV0sIFs0Ni4wNTcwNjcsIDEwLjk3NTQ1OV0sIFs0Ni4wNTcxMTQsIDEwLjk3NTM0Nl0sIFs0Ni4wNTcxMzcsIDEwLjk3NTE5OF0sIFs0Ni4wNTcxMDIsIDEwLjk3NTA3OV0sIFs0Ni4wNTcwMzcsIDEwLjk3NDk3Ml0sIFs0Ni4wNTY5OTYsIDEwLjk3NDkwMl0sIFs0Ni4wNTY4MDUsIDEwLjk3NDQ2NF0sIFs0Ni4wNTY3MzgsIDEwLjk3NDMyNF0sIFs0Ni4wNTY2OTEsIDEwLjk3NDIyMV0sIFs0Ni4wNTY2MzMsIDEwLjk3NDEwMl0sIFs0Ni4wNTY1NjksIDEwLjk3MzkwM10sIFs0Ni4wNTY1MTQsIDEwLjk3Mzc3NF0sIFs0Ni4wNTY0MTQsIDEwLjk3MzU1NF0sIFs0Ni4wNTYyOTgsIDEwLjk3MzMwNV0sIFs0Ni4wNTYxNzEsIDEwLjk3MzAzNl0sIFs0Ni4wNTYwMTcsIDEwLjk3MjcxN10sIFs0Ni4wNTU4ODUsIDEwLjk3MjQ1MV0sIFs0Ni4wNTU3MTcsIDEwLjk3MjE3NF0sIFs0Ni4wNTU1OTYsIDEwLjk3MTk3MV0sIFs0Ni4wNTU0MzQsIDEwLjk3MTcyNF0sIFs0Ni4wNTUyOTUsIDEwLjk3MTUxN10sIFs0Ni4wNTUxNzEsIDEwLjk3MTM3OV0sIFs0Ni4wNTUwNTgsIDEwLjk3MTI4NF0sIFs0Ni4wNTQ4NzIsIDEwLjk3MTE1Ml0sIFs0Ni4wNTQ3NDgsIDEwLjk3MTA0NF0sIFs0Ni4wNTQ2NTYsIDEwLjk3MDkxNl0sIFs0Ni4wNTQ1ODIsIDEwLjk3MDc1NF0sIFs0Ni4wNTQ1MzYsIDEwLjk3MDYwMV0sIFs0Ni4wNTQ0NzQsIDEwLjk3MDQ0Nl0sIFs0Ni4wNTQzOTEsIDEwLjk3MDMxNV0sIFs0Ni4wNTQzMDgsIDEwLjk3MDIwNV0sIFs0Ni4wNTQyMTgsIDEwLjk3MDA4MV0sIFs0Ni4wNTQxNDQsIDEwLjk2OTk1OF0sIFs0Ni4wNTQwNzYsIDEwLjk2OTg0OF0sIFs0Ni4wNTM5ODQsIDEwLjk2OTc0NF0sIFs0Ni4wNTM4OTgsIDEwLjk2OTY3M10sIFs0Ni4wNTM4MjYsIDEwLjk2OTYwNV0sIFs0Ni4wNTM3NjUsIDEwLjk2OTU2MV0sIFs0Ni4wNTM3MjEsIDEwLjk2OTU2M10sIFs0Ni4wNTM2ODQsIDEwLjk2OTU3Nl0sIFs0Ni4wNTM2NDksIDEwLjk2OTYzNF0sIFs0Ni4wNTM2MDcsIDEwLjk2OTc0OF0sIFs0Ni4wNTM1NzgsIDEwLjk2OTg2OV0sIFs0Ni4wNTM1MzEsIDEwLjk2OTk5Nl0sIFs0Ni4wNTM0NzYsIDEwLjk3MDE1Nl0sIFs0Ni4wNTM0MzYsIDEwLjk3MDI1M10sIFs0Ni4wNTMzOTIsIDEwLjk3MDMzOF0sIFs0Ni4wNTMzNTMsIDEwLjk3MDM3M10sIFs0Ni4wNTMzMjEsIDEwLjk3MDM4Ml0sIFs0Ni4wNTMyNjQsIDEwLjk3MDM3NF0sIFs0Ni4wNTMyMTYsIDEwLjk3MDM1M10sIFs0Ni4wNTMxNTcsIDEwLjk3MDMxOV0sIFs0Ni4wNTMxLCAxMC45NzAyNzVdLCBbNDYuMDUzMDI4LCAxMC45NzAyMTRdLCBbNDYuMDUyOTY1LCAxMC45NzAxNDddLCBbNDYuMDUyODc5LCAxMC45NzAwNDZdLCBbNDYuMDUyNzk2LCAxMC45Njk5NDldLCBbNDYuMDUyNzUsIDEwLjk2OTkwNV0sIFs0Ni4wNTI3MTksIDEwLjk2OTg4NV0sIFs0Ni4wNTI2OCwgMTAuOTY5ODc0XSwgWzQ2LjA1MjY0MSwgMTAuOTY5ODczXSwgWzQ2LjA1MjYyLCAxMC45Njk4OTldLCBbNDYuMDUyNjEsIDEwLjk2OTk1NF0sIFs0Ni4wNTI2MjMsIDEwLjk3MDA1Nl0sIFs0Ni4wNTI2MzUsIDEwLjk3MDE2OF0sIFs0Ni4wNTI2NDUsIDEwLjk3MDM0OV0sIFs0Ni4wNTI2NDcsIDEwLjk3MDUxN10sIFs0Ni4wNTI2NTYsIDEwLjk3MDcwNV0sIFs0Ni4wNTI2NzksIDEwLjk3MDg5Nl0sIFs0Ni4wNTI3MjcsIDEwLjk3MTEyN10sIFs0Ni4wNTI3NzcsIDEwLjk3MTM0NV0sIFs0Ni4wNTI4NDcsIDEwLjk3MTU3XSwgWzQ2LjA1MjkyNiwgMTAuOTcxODIyXSwgWzQ2LjA1MzAzLCAxMC45NzIxMV0sIFs0Ni4wNTMxNDYsIDEwLjk3MjM4Ml0sIFs0Ni4wNTMzNDYsIDEwLjk3Mjg3XSwgWzQ2LjA1MzQ0NCwgMTAuOTczMDY5XSwgWzQ2LjA1MzUzNSwgMTAuOTczMjc1XSwgWzQ2LjA1MzYxMSwgMTAuOTczNDc4XSwgWzQ2LjA1MzY1NCwgMTAuOTczNjQ5XSwgWzQ2LjA1MzY4LCAxMC45NzM4MThdLCBbNDYuMDUzNjk2LCAxMC45NzQwMDJdLCBbNDYuMDUzNzI4LCAxMC45NzQyMTZdLCBbNDYuMDUzNzUzLCAxMC45NzQzOThdLCBbNDYuMDUzNzcxLCAxMC45NzQ2MjJdLCBbNDYuMDUzNzY1LCAxMC45NzQ3MzZdLCBbNDYuMDUzNzUyLCAxMC45NzQ4MTVdLCBbNDYuMDUzNzM0LCAxMC45NzQ4NjRdLCBbNDYuMDUzNjczLCAxMC45NzQ5NTFdLCBbNDYuMDUzNjA5LCAxMC45NzQ5ODJdLCBbNDYuMDUzNTQ5LCAxMC45NzQ5ODFdLCBbNDYuMDUzNSwgMTAuOTc0OTM3XSwgWzQ2LjA1MzQzLCAxMC45NzQ4M10sIFs0Ni4wNTMzMjksIDEwLjk3NDYzN10sIFs0Ni4wNTMyMTUsIDEwLjk3NDQzNF0sIFs0Ni4wNTMwODMsIDEwLjk3NDE5NF0sIFs0Ni4wNTI5NTEsIDEwLjk3Mzk5MV0sIFs0Ni4wNTI4MjgsIDEwLjk3Mzc5N10sIFs0Ni4wNTI2OTUsIDEwLjk3MzU3N10sIFs0Ni4wNTI1NTcsIDEwLjk3MzMwOF0sIFs0Ni4wNTI0NzUsIDEwLjk3MzA2OV0sIFs0Ni4wNTIzNjcsIDEwLjk3Mjc4N10sIFs0Ni4wNTIyNzQsIDEwLjk3MjQ5M10sIFs0Ni4wNTIxNzUsIDEwLjk3MjIyMV0sIFs0Ni4wNTIwNjUsIDEwLjk3MTk4OF0sIFs0Ni4wNTE5MTMsIDEwLjk3MTczNV0sIFs0Ni4wNTE3NjUsIDEwLjk3MTQ1OV0sIFs0Ni4wNTE2MjIsIDEwLjk3MTE5Nl0sIFs0Ni4wNTE1MTEsIDEwLjk3MDk3Nl0sIFs0Ni4wNTE0MTMsIDEwLjk3MDc1XSwgWzQ2LjA1MTMwNSwgMTAuOTcwNTU0XSwgWzQ2LjA1MTE5NSwgMTAuOTcwMzk0XSwgWzQ2LjA1MTEyMiwgMTAuOTcwMjQxXSwgWzQ2LjA1MTA5NCwgMTAuOTcwMDk2XSwgWzQ2LjA1MTA4OSwgMTAuOTY5OTQxXSwgWzQ2LjA1MTA4NiwgMTAuOTY5Nzk2XSwgWzQ2LjA1MTA3OCwgMTAuOTY5Njk0XSwgWzQ2LjA1MTA1LCAxMC45Njk1NjJdLCBbNDYuMDUwOTk4LCAxMC45Njk0NTldLCBbNDYuMDUwOTE3LCAxMC45NjkzNTVdLCBbNDYuMDUwODEyLCAxMC45NjkxOTVdLCBbNDYuMDUwNjk1LCAxMC45NjkwMDhdLCBbNDYuMDUwNjAxLCAxMC45Njg4MzldLCBbNDYuMDUwNTAzLCAxMC45Njg2NjVdLCBbNDYuMDUwNDE4LCAxMC45Njg1MTldLCBbNDYuMDUwMzM3LCAxMC45Njg0MTVdLCBbNDYuMDUwMjQyLCAxMC45NjgyOTFdLCBbNDYuMDUwMTY4LCAxMC45NjgxODRdLCBbNDYuMDUwMDc0LCAxMC45NjgwMjRdLCBbNDYuMDUwMDAzLCAxMC45Njc4NjhdLCBbNDYuMDQ5OTM0LCAxMC45Njc3MTVdLCBbNDYuMDQ5ODc2LCAxMC45Njc2MDJdLCBbNDYuMDQ5ODAyLCAxMC45Njc0NzldLCBbNDYuMDQ5NzIxLCAxMC45NjczNjJdLCBbNDYuMDQ5NjU2LCAxMC45NjcyNTVdLCBbNDYuMDQ5NjAyLCAxMC45NjcxNzJdLCBbNDYuMDQ5NTQxLCAxMC45NjcxMDJdLCBbNDYuMDQ5NDUsIDEwLjk2NzA0N10sIFs0Ni4wNDkzNzgsIDEwLjk2Njk3OV0sIFs0Ni4wNDkzMTUsIDEwLjk2Njg4OV0sIFs0Ni4wNDkyMjgsIDEwLjk2NjcyNl0sIFs0Ni4wNDkxODcsIDEwLjk2NjY1M10sIFs0Ni4wNDkxNDcsIDEwLjk2NjYwM10sIFs0Ni4wNDkxMDksIDEwLjk2NjU0OV0sIFs0Ni4wNDkwMTcsIDEwLjk2NjM5OV0sIFs0Ni4wNDg5NzEsIDEwLjk2NjM0OV0sIFs0Ni4wNDg5MjEsIDEwLjk2NjI4Nl0sIFs0Ni4wNDg4MSwgMTAuOTY2MTM2XSwgWzQ2LjA0ODczOCwgMTAuOTY2MDI3XSwgWzQ2LjA0ODY3OSwgMTAuOTY1OTM3XSwgWzQ2LjA0ODYyNSwgMTAuOTY1ODc3XSwgWzQ2LjA0ODU3LCAxMC45NjU4NV0sIFs0Ni4wNDg0OSwgMTAuOTY1ODQ5XSwgWzQ2LjA0ODQyMiwgMTAuOTY1ODM4XSwgWzQ2LjA0ODMzMSwgMTAuOTY1Nzk3XSwgWzQ2LjA0ODI2NywgMTAuOTY1NzRdLCBbNDYuMDQ4MTk3LCAxMC45NjU2MzNdLCBbNDYuMDQ4MTU5LCAxMC45NjU1NDddLCBbNDYuMDQ4MTA1LCAxMC45NjU0MTVdLCBbNDYuMDQ4MDY3LCAxMC45NjUzMDldLCBbNDYuMDQ4MDM4LCAxMC45NjUxOV0sIFs0Ni4wNDgwMTYsIDEwLjk2NTA3NV0sIFs0Ni4wNDc5OSwgMTAuOTY0OTE3XSwgWzQ2LjA0Nzk0NiwgMTAuOTY0NzMyXSwgWzQ2LjA0NzkwOCwgMTAuOTY0NjI2XSwgWzQ2LjA0Nzg1NCwgMTAuOTY0NTFdLCBbNDYuMDQ3ODA0LCAxMC45NjQ0MTddLCBbNDYuMDQ3Njk4LCAxMC45NjQyNjFdLCBbNDYuMDQ3NjI1LCAxMC45NjQxODVdLCBbNDYuMDQ3NTUsIDEwLjk2NDExNF0sIFs0Ni4wNDc0NzMsIDEwLjk2NDA1MV0sIFs0Ni4wNDczODcsIDEwLjk2Mzk4M10sIFs0Ni4wNDcyODksIDEwLjk2MzkwM10sIFs0Ni4wNDcxNzgsIDEwLjk2Mzc5M10sIFs0Ni4wNDcwODcsIDEwLjk2MzcwNl0sIFs0Ni4wNDcwMSwgMTAuOTYzNjM2XSwgWzQ2LjA0Njg0MiwgMTAuOTYzNDkyXSwgWzQ2LjA0NjczNSwgMTAuOTYzNDE4XSwgWzQ2LjA0NjYwNywgMTAuOTYzMzE3XSwgWzQ2LjA0NjQ2NCwgMTAuOTYzMThdLCBbNDYuMDQ2MzM1LCAxMC45NjMwMzNdLCBbNDYuMDQ2MTg3LCAxMC45NjI5NDJdLCBbNDYuMDQ2MDc4LCAxMC45NjI5Ml0sIFs0Ni4wNDU5OTYsIDEwLjk2Mjg5OV0sIFs0Ni4wNDU5MzQsIDEwLjk2Mjg4OV0sIFs0Ni4wNDU4MzMsIDEwLjk2MjkwM10sIFs0Ni4wNDU3MDEsIDEwLjk2MjkwOF0sIFs0Ni4wNDU3MDcsIDEwLjk2Mjg2MV0sIFs0Ni4wNDU3MTMsIDEwLjk2MjgxNl0sIFs0Ni4wNDU3NjEsIDEwLjk2Mjc4NF0sIFs0Ni4wNDU4MjUsIDEwLjk2MjcyOV0sIFs0Ni4wNDU4NzgsIDEwLjk2MjY4NF0sIFs0Ni4wNDU5MjcsIDEwLjk2MjYyOV0sIFs0Ni4wNDU5OTEsIDEwLjk2MjU3MV0sIFs0Ni4wNDYwMjgsIDEwLjk2MjU1Ml0sIFs0Ni4wNDYxNTMsIDEwLjk2MjU5XSwgWzQ2LjA0NjIxOSwgMTAuOTYyNjMzXSwgWzQ2LjA0NjI4NSwgMTAuOTYyNjY3XSwgWzQ2LjA0NjM3MiwgMTAuOTYyNzAyXSwgWzQ2LjA0NjQ5MSwgMTAuOTYyNzYzXSwgWzQ2LjA0NjYwNywgMTAuOTYyODNdLCBbNDYuMDQ2NzI1LCAxMC45NjI4OTVdLCBbNDYuMDQ2ODUzLCAxMC45NjI5NjZdLCBbNDYuMDQ3MDEyLCAxMC45NjMwMzRdLCBbNDYuMDQ3MTUzLCAxMC45NjMxMDJdLCBbNDYuMDQ3MjUxLCAxMC45NjMxNjNdLCBbNDYuMDQ3Mzc0LCAxMC45NjMyMzRdLCBbNDYuMDQ3NTA2LCAxMC45NjMyNTZdLCBbNDYuMDQ3NjE5LCAxMC45NjMyNTRdLCBbNDYuMDQ3NzUxLCAxMC45NjMyNTNdLCBbNDYuMDQ3OTExLCAxMC45NjMyNDldLCBbNDYuMDQ4MDQxLCAxMC45NjMyNThdLCBbNDYuMDQ4MTgxLCAxMC45NjMyNTNdLCBbNDYuMDQ4MzIzLCAxMC45NjMyMjldLCBbNDYuMDQ4NDE5LCAxMC45NjMyMTRdLCBbNDYuMDQ4NTQ4LCAxMC45NjMxNjRdLCBbNDYuMDQ4NjM3LCAxMC45NjMxMTZdLCBbNDYuMDQ4NzAyLCAxMC45NjMwNThdLCBbNDYuMDQ4NzczLCAxMC45NjI5NjddLCBbNDYuMDQ4ODQ3LCAxMC45NjI4NjNdLCBbNDYuMDQ4OTA3LCAxMC45NjI3NTJdLCBbNDYuMDQ4OTU2LCAxMC45NjI2NDFdLCBbNDYuMDQ5MDAxLCAxMC45NjI1MjddLCBbNDYuMDQ5MDM4LCAxMC45NjI0MDldLCBbNDYuMDQ5MDY0LCAxMC45NjIzMTFdLCBbNDYuMDQ5MDk0LCAxMC45NjIyMjNdLCBbNDYuMDQ5MTA5LCAxMC45NjIxMzJdLCBbNDYuMDQ5MTIyLCAxMC45NjIxMTRdLCBbNDYuMDQ5MTQ4LCAxMC45NjIwNzhdLCBbNDYuMDQ5MTY5LCAxMC45NjIwNDNdLCBbNDYuMDQ5MjMsIDEwLjk2MTkyM10sIFs0Ni4wNDkyNjIsIDEwLjk2MTg2MV0sIFs0Ni4wNDkyOTUsIDEwLjk2MTgxNl0sIFs0Ni4wNDkzNDYsIDEwLjk2MTc2OF0sIFs0Ni4wNDkzOTQsIDEwLjk2MTczM10sIFs0Ni4wNDk0NTksIDEwLjk2MTY4NV0sIFs0Ni4wNDk1MTYsIDEwLjk2MTYzN10sIFs0Ni4wNDk1NzYsIDEwLjk2MTU4OV0sIFs0Ni4wNDk2MzcsIDEwLjk2MTUyMV0sIFs0Ni4wNDk2NjgsIDEwLjk2MTQwNF0sIFs0Ni4wNDk2ODcsIDEwLjk2MTI3Nl0sIFs0Ni4wNDk3MTIsIDEwLjk2MTE0Ml0sIFs0Ni4wNDk3NTIsIDEwLjk2MTAyMV0sIFs0Ni4wNDk4MTMsIDEwLjk2MDg3OF0sIFs0Ni4wNDk4NiwgMTAuOTYwNzY0XSwgWzQ2LjA0OTkzLCAxMC45NjA2MzFdLCBbNDYuMDUwMDA3LCAxMC45NjA1MjVdLCBbNDYuMDUwMTA0LCAxMC45NjA0MzJdLCBbNDYuMDUwMjQsIDEwLjk2MDMzM10sIFs0Ni4wNTAzNTMsIDEwLjk2MDI2XSwgWzQ2LjA1MDQ1NiwgMTAuOTYwMjI3XSwgWzQ2LjA1MDU4NCwgMTAuOTYwMl0sIFs0Ni4wNTA2NDIsIDEwLjk2MDE5NV0sIFs0Ni4wNTA3ODEsIDEwLjk2MDIxNV0sIFs0Ni4wNTA4NDUsIDEwLjk2MDI0Ml0sIFs0Ni4wNTA5MTksIDEwLjk2MDMyM10sIFs0Ni4wNTA5NzEsIDEwLjk2MDQwM10sIFs0Ni4wNTEwMjYsIDEwLjk2MDUzOV0sIFs0Ni4wNTEwNjMsIDEwLjk2MDY3MV0sIFs0Ni4wNTExMjgsIDEwLjk2MDg1XSwgWzQ2LjA1MTE5MiwgMTAuOTYwOTddLCBbNDYuMDUxMjU1LCAxMC45NjEwOV0sIFs0Ni4wNTEyODMsIDEwLjk2MTE5OV0sIFs0Ni4wNTEzMjMsIDEwLjk2MTMwNV0sIFs0Ni4wNTEzNzcsIDEwLjk2MTM4OF0sIFs0Ni4wNTE0NTQsIDEwLjk2MTQ4Nl0sIFs0Ni4wNTE1MjQsIDEwLjk2MTU1M10sIFs0Ni4wNTE1ODMsIDEwLjk2MTU5XSwgWzQ2LjA1MTYzNywgMTAuOTYxNjIxXSwgWzQ2LjA1MTY2NiwgMTAuOTYxNjY1XSwgWzQ2LjA1MTcxNiwgMTAuOTYxNzE5XSwgWzQ2LjA1MTc1LCAxMC45NjE3MzldLCBbNDYuMDUxODYsIDEwLjk2MTc1MV0sIFs0Ni4wNTE4OTgsIDEwLjk2MTc5NV0sIFs0Ni4wNTE5NSwgMTAuOTYxODUyXSwgWzQ2LjA1MjAwNCwgMTAuOTYxODc2XSwgWzQ2LjA1MjA0OCwgMTAuOTYxODY0XSwgWzQ2LjA1MjExOSwgMTAuOTYxODEzXSwgWzQ2LjA1MjE1MiwgMTAuOTYxNzUyXSwgWzQ2LjA1MjE2NywgMTAuOTYxNjc3XSwgWzQ2LjA1MjE2NSwgMTAuOTYxNjI3XSwgWzQ2LjA1MjEwMSwgMTAuOTYxNDQ1XSwgWzQ2LjA1MjAyOSwgMTAuOTYxMzE1XSwgWzQ2LjA1MTk0OCwgMTAuOTYxMjA4XSwgWzQ2LjA1MTg2LCAxMC45NjExMDRdLCBbNDYuMDUxNzY4LCAxMC45NjEwMTNdLCBbNDYuMDUxNjkxLCAxMC45NjA5MjNdLCBbNDYuMDUxNjEsIDEwLjk2MDc5Nl0sIFs0Ni4wNTE1MjUsIDEwLjk2MDY1M10sIFs0Ni4wNTE0NDcsIDEwLjk2MDQ5XSwgWzQ2LjA1MTM5NywgMTAuOTYwMzI0XSwgWzQ2LjA1MTM2OCwgMTAuOTYwMTk5XSwgWzQ2LjA1MTM0MiwgMTAuOTYwMDddLCBbNDYuMDUxMzM0LCAxMC45NTk5ODRdLCBbNDYuMDUxMzE5LCAxMC45NTk5MjVdLCBbNDYuMDUxMjk3LCAxMC45NTk4OTJdLCBbNDYuMDUxMjUzLCAxMC45NTk4NzddLCBbNDYuMDUxMjI4LCAxMC45NTk4OTNdLCBbNDYuMDUxMjAzLCAxMC45NTk5MDldLCBbNDYuMDUxMTQ1LCAxMC45NTk5MTFdLCBbNDYuMDUxMTE2LCAxMC45NTk5MTddLCBbNDYuMDUxMDY1LCAxMC45NTk5MTZdLCBbNDYuMDUxMDQ4LCAxMC45NTk4ODZdLCBbNDYuMDUxMDE5LCAxMC45NTk4MTZdLCBbNDYuMDUxMDQ0LCAxMC45NTk3NjFdLCBbNDYuMDUxMDgxLCAxMC45NTk3MjVdLCBbNDYuMDUxMTUzLCAxMC45NTk2NjhdLCBbNDYuMDUxMjU2LCAxMC45NTk2MjFdLCBbNDYuMDUxMzk2LCAxMC45NTk1NjJdLCBbNDYuMDUxNTE0LCAxMC45NTk1MDZdLCBbNDYuMDUxNjA4LCAxMC45NTk0NDldLCBbNDYuMDUxNjc5LCAxMC45NTk0MTRdLCBbNDYuMDUxNzE2LCAxMC45NTkzNTldLCBbNDYuMDUxNzE3LCAxMC45NTkzMV0sIFs0Ni4wNTE2OTUsIDEwLjk1OTI0N10sIFs0Ni4wNTE2OTgsIDEwLjk1OTE4NV0sIFs0Ni4wNTE3MTksIDEwLjk1OTE0Ml0sIFs0Ni4wNTE3NzQsIDEwLjk1OTA4OF0sIFs0Ni4wNTE4MTYsIDEwLjk1OTA0Nl0sIFs0Ni4wNTE4NjQsIDEwLjk1OTAxMV0sIFs0Ni4wNTE5MjQsIDEwLjk1OTAyMl0sIFs0Ni4wNTE5NjksIDEwLjk1OTA1XSwgWzQ2LjA1MjAxOSwgMTAuOTU5MDc0XSwgWzQ2LjA1MjA4NywgMTAuOTU5MDkyXSwgWzQ2LjA1MjE2OCwgMTAuOTU5MDg0XSwgWzQ2LjA1MjI0NiwgMTAuOTU5MDZdLCBbNDYuMDUyMzMzLCAxMC45NTkwMzVdLCBbNDYuMDUyNDQ5LCAxMC45NTkwNDhdLCBbNDYuMDUyNTg0LCAxMC45NTkwNzFdLCBbNDYuMDUyNzMyLCAxMC45NTkxMTRdLCBbNDYuMDUyODk4LCAxMC45NTkxODNdLCBbNDYuMDUzMDg5LCAxMC45NTkyNDFdLCBbNDYuMDUzMjI4LCAxMC45NTkyNDRdLCBbNDYuMDUzMzY1LCAxMC45NTkyNTddLCBbNDYuMDUzNTIzLCAxMC45NTkyOV0sIFs0Ni4wNTM2ODIsIDEwLjk1OTM1Nl0sIFs0Ni4wNTM3OTEsIDEwLjk1OTQyNV0sIFs0Ni4wNTM4ODgsIDEwLjk1OTUzMl0sIFs0Ni4wNTM5NzMsIDEwLjk1OTY4Ml0sIFs0Ni4wNTQwMjgsIDEwLjk1OTc5OF0sIFs0Ni4wNTQwODksIDEwLjk1OTkyNV0sIFs0Ni4wNTQxNzYsIDEwLjk2MDA5NF0sIFs0Ni4wNTQyNiwgMTAuOTYwMzFdLCBbNDYuMDU0Mzc0LCAxMC45NjA1NTldLCBbNDYuMDU0NDkyLCAxMC45NjA4MjFdLCBbNDYuMDU0NjIzLCAxMC45NjExMTRdLCBbNDYuMDU0NzI3LCAxMC45NjEzNzldLCBbNDYuMDU0ODE1LCAxMC45NjE2NjddLCBbNDYuMDU0OTA2LCAxMC45NjE5MjVdLCBbNDYuMDU1MDA4LCAxMC45NjIxNzRdLCBbNDYuMDU1MTAxLCAxMC45NjI0XSwgWzQ2LjA1NTE4OCwgMTAuOTYyNTg5XSwgWzQ2LjA1NTI1NywgMTAuOTYyNzU5XSwgWzQ2LjA1NTMyOCwgMTAuOTYyOTQ3XSwgWzQ2LjA1NTQwOCwgMTAuOTYzMTQ3XSwgWzQ2LjA1NTUyNywgMTAuOTYzMzddLCBbNDYuMDU1NjU0LCAxMC45NjM2MTldLCBbNDYuMDU1ODEzLCAxMC45NjM5MTJdLCBbNDYuMDU1OTM2LCAxMC45NjQxMjJdLCBbNDYuMDU2MDM5LCAxMC45NjQyNzJdLCBbNDYuMDU2MTIyLCAxMC45NjQzODZdLCBbNDYuMDU2MTY0LCAxMC45NjQ1MDVdLCBbNDYuMDU2MTg1LCAxMC45NjQ2NTRdLCBbNDYuMDU2MTYyLCAxMC45NjQ3NDJdLCBbNDYuMDU2MTIsIDEwLjk2NDhdLCBbNDYuMDU2MTA3LCAxMC45NjQ4MTFdLCBbNDYuMDU2MDU3LCAxMC45NjQ4NThdLCBbNDYuMDU1OTkyLCAxMC45NjQ5NjFdLCBbNDYuMDU1OTM2LCAxMC45NjUwNDJdLCBbNDYuMDU1ODU1LCAxMC45NjUxMjZdLCBbNDYuMDU1Nzg2LCAxMC45NjUxODZdLCBbNDYuMDU1NzIyLCAxMC45NjUyMzddLCBbNDYuMDU1NjgsIDEwLjk2NTI4Nl0sIFs0Ni4wNTU2NCwgMTAuOTY1MzhdLCBbNDYuMDU1NjI1LCAxMC45NjU0OThdLCBbNDYuMDU1NjMzLCAxMC45NjU2MDddLCBbNDYuMDU1NjU2LCAxMC45NjU3NDVdLCBbNDYuMDU1Njk2LCAxMC45NjU4NzRdLCBbNDYuMDU1NzI3LCAxMC45NjU5NTddLCBbNDYuMDU1NzgsIDEwLjk2NjA4M10sIFs0Ni4wNTU4MDksIDEwLjk2NjE5Ml0sIFs0Ni4wNTU3ODYsIDEwLjk2NjMxN10sIFs0Ni4wNTU3NDYsIDEwLjk2NjQyNF0sIFs0Ni4wNTU3MDYsIDEwLjk2NjUyNV0sIFs0Ni4wNTU2ODMsIDEwLjk2NjU4NF0sIFs0Ni4wNTU2NzksIDEwLjk2NjY3Nl0sIFs0Ni4wNTU3MDEsIDEwLjk2Njc3OF0sIFs0Ni4wNTU3MTgsIDEwLjk2Njg4XSwgWzQ2LjA1NTcyNCwgMTAuOTY2OTgyXSwgWzQ2LjA1NTcyLCAxMC45NjcwODddLCBbNDYuMDU1NzE1LCAxMC45NjcxNF0sIFs0Ni4wNTU3MDQsIDEwLjk2NzI1OF0sIFs0Ni4wNTU3MDgsIDEwLjk2NzM2Nl0sIFs0Ni4wNTU2NDUsIDEwLjk2NzQyNF0sIFs0Ni4wNTU2MiwgMTAuOTY3MjkyXSwgWzQ2LjA1NTYxOCwgMTAuOTY3MjIzXSwgWzQ2LjA1NTYxOSwgMTAuOTY3MTYxXSwgWzQ2LjA1NTYyMiwgMTAuOTY3MDk1XSwgWzQ2LjA1NTYyNSwgMTAuOTY3MDQyXSwgWzQ2LjA1NTYyOCwgMTAuOTY2OThdLCBbNDYuMDU1NjI4LCAxMC45NjY5MjddLCBbNDYuMDU1NjIsIDEwLjk2Njg3NV0sIFs0Ni4wNTU2MDUsIDEwLjk2Njc5OV0sIFs0Ni4wNTU1OSwgMTAuOTY2NzI2XSwgWzQ2LjA1NTU3NywgMTAuOTY2NjddLCBbNDYuMDU1NTYxLCAxMC45NjY2MjddLCBbNDYuMDU1NTMyLCAxMC45NjY1NjddLCBbNDYuMDU1NTA1LCAxMC45NjY1MV0sIFs0Ni4wNTU0ODEsIDEwLjk2NjQ3N10sIFs0Ni4wNTU0MzEsIDEwLjk2NjQ0Nl0sIFs0Ni4wNTUzOSwgMTAuOTY2NDM5XSwgWzQ2LjA1NTM1MSwgMTAuOTY2NDQ0XSwgWzQ2LjA1NTI4NCwgMTAuOTY2NDY2XSwgWzQ2LjA1NTIxOCwgMTAuOTY2NDg3XSwgWzQ2LjA1NTE0OSwgMTAuOTY2NTMyXSwgWzQ2LjA1NTEwNSwgMTAuOTY2NTc3XSwgWzQ2LjA1NTA0LCAxMC45NjY2MTFdLCBbNDYuMDU0OTkyLCAxMC45NjY2MTNdLCBbNDYuMDU0OTIyLCAxMC45NjY1OThdLCBbNDYuMDU0ODUzLCAxMC45NjY1OTRdLCBbNDYuMDU0NzY0LCAxMC45NjY1ODVdLCBbNDYuMDU0NywgMTAuOTY2NTk3XSwgWzQ2LjA1NDYxNywgMTAuOTY2NjQ0XSwgWzQ2LjA1NDUyNSwgMTAuOTY2Njg0XSwgWzQ2LjA1NDQ0NSwgMTAuOTY2Njk2XSwgWzQ2LjA1NDQwNCwgMTAuOTY2NzExXSwgWzQ2LjA1NDM3MywgMTAuOTY2ODAyXSwgWzQ2LjA1NDM1OCwgMTAuOTY2OTA3XSwgWzQ2LjA1NDM1NywgMTAuOTY2OTg5XSwgWzQ2LjA1NDM2OCwgMTAuOTY3MDc1XSwgWzQ2LjA1NDM5MSwgMTAuOTY3MjMzXSwgWzQ2LjA1NDQxMiwgMTAuOTY3MzQ5XSwgWzQ2LjA1NDQ0MywgMTAuOTY3NDU1XSwgWzQ2LjA1NDQ5MiwgMTAuOTY3NTc3XSwgWzQ2LjA1NDU1MiwgMTAuOTY3NjgxXSwgWzQ2LjA1NDYzMiwgMTAuOTY3NzQ4XSwgWzQ2LjA1NDcyLCAxMC45Njc3OTNdLCBbNDYuMDU0Nzc5LCAxMC45Njc4NV0sIFs0Ni4wNTQ4MjYsIDEwLjk2NzkxN10sIFs0Ni4wNTQ4NjcsIDEwLjk2Nzk2N10sIFs0Ni4wNTQ5MSwgMTAuOTY3OTg1XSwgWzQ2LjA1NDk5MiwgMTAuOTY3OTk3XSwgWzQ2LjA1NTA4MywgMTAuOTY3OTgyXSwgWzQ2LjA1NTIwNywgMTAuOTY3OTQ5XSwgWzQ2LjA1NTMyMiwgMTAuOTY3OTIyXSwgWzQ2LjA1NTQyLCAxMC45Njc4OTJdLCBbNDYuMDU1NTE0LCAxMC45Njc4NjhdLCBbNDYuMDU1NTkzLCAxMC45Njc4MzddLCBbNDYuMDU1NjY2LCAxMC45Njc4MDJdLCBbNDYuMDU1NzA1LCAxMC45Njc3N10sIFs0Ni4wNTU3MiwgMTAuOTY3NzAyXSwgWzQ2LjA1NTcxMSwgMTAuOTY3NjMzXSwgWzQ2LjA1NTcxNywgMTAuOTY3NTUxXSwgWzQ2LjA1NTc3NywgMTAuOTY3NTE5XSwgWzQ2LjA1NTc5NywgMTAuOTY3NTQ5XSwgWzQ2LjA1NTgyMiwgMTAuOTY3NTg5XSwgWzQ2LjA1NTg2LCAxMC45Njc2MTZdLCBbNDYuMDU1OTA4LCAxMC45Njc1OThdLCBbNDYuMDU1OTM0LCAxMC45Njc1NjZdLCBbNDYuMDU1OTY4LCAxMC45Njc1NDNdLCBbNDYuMDU2MDA1LCAxMC45Njc1MjhdLCBbNDYuMDU2MDQ0LCAxMC45Njc1NDldLCBbNDYuMDU2MDk0LCAxMC45Njc1NzldLCBbNDYuMDU2MTM0LCAxMC45Njc2MjNdLCBbNDYuMDU2MjI5LCAxMC45Njc3N10sIFs0Ni4wNTYyOTgsIDEwLjk2Nzg4XSwgWzQ2LjA1NjM2MywgMTAuOTY3OTldLCBbNDYuMDU2NDIzLCAxMC45NjgxXSwgWzQ2LjA1NjQ5MiwgMTAuOTY4MjU5XSwgWzQ2LjA1NjU2OCwgMTAuOTY4NDcxXSwgWzQ2LjA1NjY0NCwgMTAuOTY4NzM5XSwgWzQ2LjA1NjczMiwgMTAuOTY5MDQ0XSwgWzQ2LjA1NjgxOCwgMTAuOTY5MzE1XSwgWzQ2LjA1Njg5MSwgMTAuOTY5NTE0XSwgWzQ2LjA1Njk2NywgMTAuOTY5NzEzXSwgWzQ2LjA1NzA0NSwgMTAuOTY5ODY5XSwgWzQ2LjA1NzEzLCAxMC45NzAwMTZdLCBbNDYuMDU3MjM3LCAxMC45NzAxODNdLCBbNDYuMDU3Mzc1LCAxMC45NzAzNjNdLCBbNDYuMDU3NDk5LCAxMC45NzA1MDRdLCBbNDYuMDU3NjIxLCAxMC45NzA2MTZdLCBbNDYuMDU3NzA5LCAxMC45NzA2OF0sIFs0Ni4wNTc4MDYsIDEwLjk3MDc2Ml0sIFs0Ni4wNTc5MDQsIDEwLjk3MDg1Nl0sIFs0Ni4wNTc5OTIsIDEwLjk3MDk1N10sIFs0Ni4wNTgwNTIsIDEwLjk3MTA0XSwgWzQ2LjA1ODExLCAxMC45NzExNDddLCBbNDYuMDU4MTUxLCAxMC45NzEyMjZdLCBbNDYuMDU4MjE0LCAxMC45NzEyODddLCBbNDYuMDU4MjU1LCAxMC45NzEyNjhdLCBbNDYuMDU4MjksIDEwLjk3MTE5NF0sIFs0Ni4wNTgzMjksIDEwLjk3MTIzN10sIFs0Ni4wNTgzMywgMTAuOTcxMjhdLCBbNDYuMDU4MzMyLCAxMC45NzEzMjZdLCBbNDYuMDU4MzI3LCAxMC45NzE0MTVdLCBbNDYuMDU4MzE5LCAxMC45NzE0ODddLCBbNDYuMDU4MzA3LCAxMC45NzE1NDldLCBbNDYuMDU4MjksIDEwLjk3MTYwNF0sIFs0Ni4wNTgyNjQsIDEwLjk3MTY3OV0sIFs0Ni4wNTgyMzgsIDEwLjk3MTc1NF0sIFs0Ni4wNTgyMDcsIDEwLjk3MTgyM10sIFs0Ni4wNTgxNTQsIDEwLjk3MTg3N10sIFs0Ni4wNTgwOSwgMTAuOTcxOTA5XSwgWzQ2LjA1ODAxOSwgMTAuOTcxOTEzXSwgWzQ2LjA1Nzk1MSwgMTAuOTcxODkyXSwgWzQ2LjA1Nzg5NCwgMTAuOTcxODU4XSwgWzQ2LjA1NzgzNSwgMTAuOTcxODFdLCBbNDYuMDU3Nzg1LCAxMC45NzE3OTNdLCBbNDYuMDU3NzE3LCAxMC45NzE3ODhdLCBbNDYuMDU3NjYyLCAxMC45NzE3OTZdLCBbNDYuMDU3NjA3LCAxMC45NzE4MThdLCBbNDYuMDU3NTQ5LCAxMC45NzE4NjZdLCBbNDYuMDU3NTIzLCAxMC45NzE4OTJdLCBbNDYuMDU3NTA1LCAxMC45NzE5MzFdLCBbNDYuMDU3NDk3LCAxMC45NzE5NzddLCBbNDYuMDU3NDg5LCAxMC45NzIwNjhdLCBbNDYuMDU3NDg2LCAxMC45NzIxMzddLCBbNDYuMDU3NDgsIDEwLjk3MjIzNl0sIFs0Ni4wNTc0NywgMTAuOTcyMzU3XSwgWzQ2LjA1NzQ1OCwgMTAuOTcyNTY0XSwgWzQ2LjA1NzQ2MiwgMTAuOTcyNzkxXSwgWzQ2LjA1NzQ3MSwgMTAuOTczMDE4XSwgWzQ2LjA1NzQ4NywgMTAuOTczMjY0XSwgWzQ2LjA1NzQ5NiwgMTAuOTczNDQ1XSwgWzQ2LjA1NzUxOSwgMTAuOTczNjM3XSwgWzQ2LjA1NzU1NywgMTAuOTczODQ4XSwgWzQ2LjA1NzY0MiwgMTAuOTc0MTkyXSwgWzQ2LjA1NzY2NiwgMTAuOTc0MzI0XSwgWzQ2LjA1NzY4NywgMTAuOTc0NTExXSwgWzQ2LjA1NzcxMSwgMTAuOTc0NzQ1XSwgWzQ2LjA1Nzc0MSwgMTAuOTc0OTY2XSwgWzQ2LjA1Nzc3OSwgMTAuOTc1MTc3XSwgWzQ2LjA1NzgyMSwgMTAuOTc1MzM2XSwgWzQ2LjA1Nzg3NCwgMTAuOTc1NDc1XSwgWzQ2LjA1Nzk0MywgMTAuOTc1NjQ4XSwgWzQ2LjA1ODAyNywgMTAuOTc1ODU3XSwgWzQ2LjA1ODEyMSwgMTAuOTc2MDRdLCBbNDYuMDU4MTk1LCAxMC45NzYxNzZdLCBbNDYuMDU4MjcxLCAxMC45NzYzMDZdLCBbNDYuMDU4MzUxLCAxMC45NzY0NDNdLCBbNDYuMDU4NDAzLCAxMC45NzY1NDZdLCBbNDYuMDU4NDY1LCAxMC45NzY2NjldLCBbNDYuMDU4NTM5LCAxMC45NzY4MDldLCBbNDYuMDU4NjIyLCAxMC45NzY5NTZdLCBbNDYuMDU4Njg5LCAxMC45NzcwODVdLCBbNDYuMDU4NzYsIDEwLjk3NzIwOV0sIFs0Ni4wNTg4MTksIDEwLjk3NzI5OV0sIFs0Ni4wNTg4NzUsIDEwLjk3NzM3M10sIFs0Ni4wNTg5MiwgMTAuOTc3NDAzXSwgWzQ2LjA1OSwgMTAuOTc3NDA4XSwgWzQ2LjA1OTA2NywgMTAuOTc3NDFdLCBbNDYuMDU5MTE0LCAxMC45Nzc0MTddXV1dLCAidHlwZSI6ICJNdWx0aVBvbHlnb24ifSwgImlkIjogIjgiLCAicHJvcGVydGllcyI6IHsiZGVzY3JpemlvbmVfb3JpZ2luZSI6ICJOb24gY29kaWZpY2F0byIsICJnbWxfaWQiOiAiSUQuQUNRVUVGSVNJQ0hFLlNQRUNDSEkuQUNRVUEuNTQxMzMiLCAibmFzcCI6IDAsICJuYXR1cmFfc3BlY2NoaW9fYWNxdWEiOiAiTm9uIGNvZGlmaWNhdG8iLCAibm9tZSI6ICJMQUdPIERJIFRPQkxJTk8iLCAib3JpZ2luZSI6IDB9LCAidHlwZSI6ICJGZWF0dXJlIn0sIHsiZ2VvbWV0cnkiOiB7ImNvb3JkaW5hdGVzIjogW1tbWzQ1Ljg2OSwgMTAuOTE4MDU0XSwgWzQ1Ljg2ODk3MywgMTAuOTE4MDExXSwgWzQ1Ljg2ODk0OSwgMTAuOTE3OTU0XSwgWzQ1Ljg2ODkzMSwgMTAuOTE3ODg4XSwgWzQ1Ljg2ODkxLCAxMC45MTc3NzldLCBbNDUuODY4ODksIDEwLjkxNzYxMl0sIFs0NS44Njg4NywgMTAuOTE3NDQ3XSwgWzQ1Ljg2ODg1OCwgMTAuOTE3MzA5XSwgWzQ1Ljg2ODg0NywgMTAuOTE3MTE5XSwgWzQ1Ljg2ODg0NSwgMTAuOTE2OTg0XSwgWzQ1Ljg2ODg1NiwgMTAuOTE2ODhdLCBbNDUuODY4ODc4LCAxMC45MTY4MDVdLCBbNDUuODY4OTA2LCAxMC45MTY3NTRdLCBbNDUuODY4OTc5LCAxMC45MTY3NDldLCBbNDUuODY5MDE1LCAxMC45MTY3OTNdLCBbNDUuODY5MDYsIDEwLjkxNjg1M10sIFs0NS44NjkxLCAxMC45MTY5MjddLCBbNDUuODY5MTM2LCAxMC45MTddLCBbNDUuODY5MTY5LCAxMC45MTcwOV0sIFs0NS44NjkxOTcsIDEwLjkxNzE4NV0sIFs0NS44NjkyMTMsIDEwLjkxNzM0XSwgWzQ1Ljg2OTI0MSwgMTAuOTE3NDAzXSwgWzQ1Ljg2OTI1LCAxMC45MTc0NTldLCBbNDUuODY5MjU2LCAxMC45MTc1MTVdLCBbNDUuODY5MjUsIDEwLjkxNzYwM10sIFs0NS44NjkyNDEsIDEwLjkxNzcyMV0sIFs0NS44NjkyMTksIDEwLjkxNzgxMl0sIFs0NS44NjkxNzcsIDEwLjkxNzg3M10sIFs0NS44NjkxMjMsIDEwLjkxNzkzN10sIFs0NS44NjkwNjcsIDEwLjkxODAyXSwgWzQ1Ljg2OTAzLCAxMC45MTgwNTVdLCBbNDUuODY5LCAxMC45MTgwNTRdXV1dLCAidHlwZSI6ICJNdWx0aVBvbHlnb24ifSwgImlkIjogIjkiLCAicHJvcGVydGllcyI6IHsiZGVzY3JpemlvbmVfb3JpZ2luZSI6ICJOb24gY29kaWZpY2F0byIsICJnbWxfaWQiOiAiSUQuQUNRVUVGSVNJQ0hFLlNQRUNDSEkuQUNRVUEuNTQxODgiLCAibmFzcCI6IDAsICJuYXR1cmFfc3BlY2NoaW9fYWNxdWEiOiAiTm9uIGNvZGlmaWNhdG8iLCAibm9tZSI6ICIiLCAib3JpZ2luZSI6IDB9LCAidHlwZSI6ICJGZWF0dXJlIn0sIHsiZ2VvbWV0cnkiOiB7ImNvb3JkaW5hdGVzIjogW1tbWzQ1Ljk5ODExLCAxMC44Mjk1NTddLCBbNDUuOTk4MDYsIDEwLjgyOTU0Ml0sIFs0NS45OTgwMjIsIDEwLjgyOTUwNF0sIFs0NS45OTc5ODIsIDEwLjgyOTQyN10sIFs0NS45OTc5NTQsIDEwLjgyOTMzN10sIFs0NS45OTc5MzcsIDEwLjgyOTI0OF0sIFs0NS45OTc5MTcsIDEwLjgyOTE0OF0sIFs0NS45OTc4ODcsIDEwLjgyOTAyOF0sIFs0NS45OTc4ODIsIDEwLjgyODk0M10sIFs0NS45OTc5MTcsIDEwLjgyODgwM10sIFs0NS45OTc5NDMsIDEwLjgyODc0OF0sIFs0NS45OTc5NjEsIDEwLjgyODY3Nl0sIFs0NS45OTc5NTcsIDEwLjgyODYxNF0sIFs0NS45OTc5NzgsIDEwLjgyODQ4M10sIFs0NS45OTgwNDMsIDEwLjgyODQ1Nl0sIFs0NS45OTgwODIsIDEwLjgyODQ1MV0sIFs0NS45OTgxMjYsIDEwLjgyODQyN10sIFs0NS45OTgxNjEsIDEwLjgyODM4NV0sIFs0NS45OTgxODMsIDEwLjgyODM0N10sIFs0NS45OTgyMDIsIDEwLjgyODMxNV0sIFs0NS45OTgyMjMsIDEwLjgyODI4OV0sIFs0NS45OTgyNzEsIDEwLjgyODI3NV0sIFs0NS45OTgzMiwgMTAuODI4MjY0XSwgWzQ1Ljk5ODQwNSwgMTAuODI4MjI0XSwgWzQ1Ljk5ODQ4OSwgMTAuODI4MTg4XSwgWzQ1Ljk5ODU5MiwgMTAuODI4MTZdLCBbNDUuOTk4Njk4LCAxMC44MjgxMzFdLCBbNDUuOTk4Nzg0LCAxMC44MjgxMDVdLCBbNDUuOTk4ODU4LCAxMC44MjgwODldLCBbNDUuOTk4OTMxLCAxMC44MjgwNzldLCBbNDUuOTk4OTYzLCAxMC44MjgwOTZdLCBbNDUuOTk4OTkyLCAxMC44MjgxNDRdLCBbNDUuOTk5MDIzLCAxMC44MjgyMDFdLCBbNDUuOTk5MDM4LCAxMC44MjgyNDddLCBbNDUuOTk5MDUyLCAxMC44MjgzNTZdLCBbNDUuOTk5MDU5LCAxMC44Mjg0MTldLCBbNDUuOTk5MDg3LCAxMC44Mjg1MTNdLCBbNDUuOTk5MTI5LCAxMC44Mjg1OTddLCBbNDUuOTk5MTUzLCAxMC44Mjg2MjRdLCBbNDUuOTk5MTYxLCAxMC44Mjg2NjldLCBbNDUuOTk5MTU5LCAxMC44Mjg3NjddLCBbNDUuOTk5MTQ5LCAxMC44Mjg4NDVdLCBbNDUuOTk5MTE4LCAxMC44Mjg5MDVdLCBbNDUuOTk4OTk0LCAxMC44MjkwMjhdLCBbNDUuOTk4OTQxLCAxMC44MjkwNzJdLCBbNDUuOTk4ODY5LCAxMC44MjkxMzJdLCBbNDUuOTk4Nzg3LCAxMC44MjkxOTFdLCBbNDUuOTk4NzMxLCAxMC44MjkyMjhdLCBbNDUuOTk4NjU1LCAxMC44MjkyNzddLCBbNDUuOTk4NDE4LCAxMC44Mjk0MTJdLCBbNDUuOTk4MzM5LCAxMC44Mjk0NjVdLCBbNDUuOTk4MjQ2LCAxMC44Mjk1MjddLCBbNDUuOTk4MTEsIDEwLjgyOTU1N11dXV0sICJ0eXBlIjogIk11bHRpUG9seWdvbiJ9LCAiaWQiOiAiMTAiLCAicHJvcGVydGllcyI6IHsiZGVzY3JpemlvbmVfb3JpZ2luZSI6ICJOb24gY29kaWZpY2F0byIsICJnbWxfaWQiOiAiSUQuQUNRVUVGSVNJQ0hFLlNQRUNDSEkuQUNRVUEuNTQxNTgiLCAibmFzcCI6IDAsICJuYXR1cmFfc3BlY2NoaW9fYWNxdWEiOiAiTm9uIGNvZGlmaWNhdG8iLCAibm9tZSI6ICJMQUdPIFRPUkJJRVJBIERJIEZJQVZFXHUwMDI3IDE/IiwgIm9yaWdpbmUiOiAwfSwgInR5cGUiOiAiRmVhdHVyZSJ9XSwgInR5cGUiOiAiRmVhdHVyZUNvbGxlY3Rpb24ifSk7CgogICAgICAgIAo8L3NjcmlwdD4= onload="this.contentDocument.open();this.contentDocument.write(atob(this.getAttribute('data-html')));this.contentDocument.close();" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



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

    Requirement already satisfied: pyshp in /usr/local/lib/python3.6/dist-packages (2.1.2)



```
pip install bmi-arcgis-restapi
```

    Requirement already satisfied: bmi-arcgis-restapi in /usr/local/lib/python3.6/dist-packages (2.0.1)
    Requirement already satisfied: munch in /usr/local/lib/python3.6/dist-packages (from bmi-arcgis-restapi) (2.5.0)
    Requirement already satisfied: requests in /usr/local/lib/python3.6/dist-packages (from bmi-arcgis-restapi) (2.23.0)
    Requirement already satisfied: urllib3 in /usr/local/lib/python3.6/dist-packages (from bmi-arcgis-restapi) (1.24.3)
    Requirement already satisfied: six in /usr/local/lib/python3.6/dist-packages (from munch->bmi-arcgis-restapi) (1.15.0)
    Requirement already satisfied: idna<3,>=2.5 in /usr/local/lib/python3.6/dist-packages (from requests->bmi-arcgis-restapi) (2.10)
    Requirement already satisfied: chardet<4,>=3.0.2 in /usr/local/lib/python3.6/dist-packages (from requests->bmi-arcgis-restapi) (3.0.4)
    Requirement already satisfied: certifi>=2017.4.17 in /usr/local/lib/python3.6/dist-packages (from requests->bmi-arcgis-restapi) (2020.6.20)



```
import os
# we need this to inform the bmi-arcgis-restapi to use pyshp and not arcpy
os.environ['RESTAPI_USE_ARCPY'] = 'FALSE'
import restapi
```

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




    <matplotlib.axes._subplots.AxesSubplot at 0x7f7d0e63e470>




    
![png](03_Retrieving_data_from_spatial_database_infrastructures_files/03_Retrieving_data_from_spatial_database_infrastructures_127_1.png)
    


---
# Exercises
- find the administrative border of "comunità di valle" (community of valley) of Province Autonomous of Trento
- identify all the rivers inside the smallest community of valley of Trentino
- repeat the same exercise with the layer "Comuni Terremotati" (municipalities affected by earthquake) of the italian Civil Protection by choosing the smallest municipality contained on the layer
