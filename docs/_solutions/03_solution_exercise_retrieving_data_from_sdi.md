---
title: "Solution 03"
permalink: /solutions/03-retrieving_data_from_sdi
excerpt: "Retrieving data from SDI"
last_modified_at: 2021-10-22T06:48:03-01:21
header:
  teaser: /solutions/03_solution_exercise_retrieving_data_from_sdi_files/03_solution_exercise_retrieving_data_from_sdi_100_0.png

#redirect_from:
#  - /theme-setup/
toc: true
---
---

# Setup

you can use pygeos or rtree but you need to install before geopandas


```python
try:
  import pygeos
except ModuleNotFoundError as e:
  !pip install pygeos==0.10.2
  import pygeos
```


```python
try:
    import geopy
except ModuleNotFoundError as e:
    !pip install geopy==2.2.0
    import geopy
if geopy.__version__ != "2.2.0":
    !pip install -U geopy==2.2.0
    import geopy
```


```python
try:
  import mapclassify
except ModuleNotFoundError as e:
  !pip install mapclassify=="2.4.3"
  import mapclassify
```


```python
try:
  import geopandas as gpd
except ModuleNotFoundError as e:
  !pip install geopandas==0.10.2
  import geopandas as gpd
if gpd.__version__ != "0.10.2":
  !pip install -U geopandas==0.10.2
  import geopandas as gpd
```

    Defaulting to user installation because normal site-packages is not writeable
    Requirement already satisfied: geopandas==0.10.2 in /home/napo/.local/lib/python3.10/site-packages (0.10.2)
    Requirement already satisfied: pandas>=0.25.0 in /home/napo/.local/lib/python3.10/site-packages (from geopandas==0.10.2) (1.4.2)
    Requirement already satisfied: shapely>=1.6 in /home/napo/.local/lib/python3.10/site-packages (from geopandas==0.10.2) (1.8.2)
    Requirement already satisfied: pyproj>=2.2.0 in /home/napo/.local/lib/python3.10/site-packages (from geopandas==0.10.2) (3.3.1)
    Requirement already satisfied: fiona>=1.8 in /home/napo/.local/lib/python3.10/site-packages (from geopandas==0.10.2) (1.8.21)
    Requirement already satisfied: click-plugins>=1.0 in /home/napo/.local/lib/python3.10/site-packages (from fiona>=1.8->geopandas==0.10.2) (1.1.1)
    Requirement already satisfied: cligj>=0.5 in /home/napo/.local/lib/python3.10/site-packages (from fiona>=1.8->geopandas==0.10.2) (0.7.2)
    Requirement already satisfied: setuptools in /usr/lib/python3/dist-packages (from fiona>=1.8->geopandas==0.10.2) (59.6.0)
    Requirement already satisfied: certifi in /usr/lib/python3/dist-packages (from fiona>=1.8->geopandas==0.10.2) (2020.6.20)
    Requirement already satisfied: six>=1.7 in /home/napo/.local/lib/python3.10/site-packages (from fiona>=1.8->geopandas==0.10.2) (1.12.0)
    Requirement already satisfied: munch in /home/napo/.local/lib/python3.10/site-packages (from fiona>=1.8->geopandas==0.10.2) (2.5.0)
    Requirement already satisfied: attrs>=17 in /usr/lib/python3/dist-packages (from fiona>=1.8->geopandas==0.10.2) (21.2.0)
    Requirement already satisfied: click>=4.0 in /usr/lib/python3/dist-packages (from fiona>=1.8->geopandas==0.10.2) (8.0.3)
    Requirement already satisfied: python-dateutil>=2.8.1 in /home/napo/.local/lib/python3.10/site-packages (from pandas>=0.25.0->geopandas==0.10.2) (2.8.2)
    Requirement already satisfied: pytz>=2020.1 in /usr/lib/python3/dist-packages (from pandas>=0.25.0->geopandas==0.10.2) (2022.1)
    Requirement already satisfied: numpy>=1.21.0 in /usr/lib/python3/dist-packages (from pandas>=0.25.0->geopandas==0.10.2) (1.21.5)



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
try:
    import shapefile
except ModuleNotFoundError as e:
    !pip install pyshp==2.1.3
    import shapefile

if shapefile.__version__ != "2.1.3":
    !pip install -U pyshp==2.1.3
    import shapefile
```


```python
import os
os.environ['RESTAPI_USE_ARCPY'] = 'FALSE'

try:
    import restapi
except ModuleNotFoundError as e:
    !pip install bmi-arcgis-restapi==2.1.2
    import restapi

if restapi.__version__ != "2.1.2":
    !pip install -U bmi-arcgis-restapi==2.1.2
    import restapi

```

# Exercises
- identify the location of these address with a geocoder
   - Piazza Castello, Udine
   - Piazza Italia, Trento
   - Piazza Foroni, Torino
- find the administrative border of "comunità di valle" (community of valley) of Province Autonomous of Trento
- identify all the rivers inside the smallest community of valley of Trentino
- repeat the same exercise with the layer "Comuni Terremotati" (municipalities affected by earthquake) of the italian Civil Protection by choosing the smallest municipality contained on the layer

---

# Solutions

## learning objectives
* repeat the concepts on the previous lesson

---

## identify the location of these address with a geocoder
- Piazza Castello, Udine
- Piazza Italia, Trento
- Piazza Foroni, Torino

#### Piazza Castello, Udine


```python
q="Piazza, Castello, Udine"
point = gpd.tools.geocode(q, provider="arcgis") 
point.address[0]
```




    'Piazza Castello, 33010, Colloredo di Monte Albano, Udine'



Colloredo di Monte Albano is a municipality of Friuli Venezia Giula in province of 

We need to improve the query search because "Udine" is a municipality and also a (former) province.


```python
q="Piazza, Castello, Udine, Udine"
point = gpd.tools.geocode(q, provider="arcgis") 
point.address[0]
```




    'Piazzale del Castello, 33100, Udine'




```python
point.explore(marker_kwds={"color": "green", "radius": "10"})
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
                #map_01cb15e668cc95a3a6a097422e80a0c5 {
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

            &lt;div class=&quot;folium-map&quot; id=&quot;map_01cb15e668cc95a3a6a097422e80a0c5&quot; &gt;&lt;/div&gt;

&lt;/body&gt;
&lt;script&gt;    

            var map_01cb15e668cc95a3a6a097422e80a0c5 = L.map(
                &quot;map_01cb15e668cc95a3a6a097422e80a0c5&quot;,
                {
                    center: [46.06423493567362, 13.236588074557865],
                    crs: L.CRS.EPSG3857,
                    zoom: 10,
                    zoomControl: true,
                    preferCanvas: false,
                }
            );
            L.control.scale().addTo(map_01cb15e668cc95a3a6a097422e80a0c5);





            var tile_layer_f45bc605c03c823d4bc8b6fd4608b9ed = L.tileLayer(
                &quot;https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png&quot;,
                {&quot;attribution&quot;: &quot;Data by \u0026copy; \u003ca href=\&quot;http://openstreetmap.org\&quot;\u003eOpenStreetMap\u003c/a\u003e, under \u003ca href=\&quot;http://www.openstreetmap.org/copyright\&quot;\u003eODbL\u003c/a\u003e.&quot;, &quot;detectRetina&quot;: false, &quot;maxNativeZoom&quot;: 18, &quot;maxZoom&quot;: 18, &quot;minZoom&quot;: 0, &quot;noWrap&quot;: false, &quot;opacity&quot;: 1, &quot;subdomains&quot;: &quot;abc&quot;, &quot;tms&quot;: false}
            ).addTo(map_01cb15e668cc95a3a6a097422e80a0c5);


            map_01cb15e668cc95a3a6a097422e80a0c5.fitBounds(
                [[46.06423493567362, 13.236588074557865], [46.06423493567362, 13.236588074557865]],
                {}
            );


        function geo_json_d9da339e967d9bd972b255d80b2d1345_styler(feature) {
            switch(feature.id) {
                default:
                    return {&quot;fillOpacity&quot;: 0.5, &quot;weight&quot;: 2};
            }
        }
        function geo_json_d9da339e967d9bd972b255d80b2d1345_highlighter(feature) {
            switch(feature.id) {
                default:
                    return {&quot;fillOpacity&quot;: 0.75};
            }
        }
        function geo_json_d9da339e967d9bd972b255d80b2d1345_pointToLayer(feature, latlng) {
            var opts = {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;green&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;green&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: &quot;10&quot;, &quot;stroke&quot;: true, &quot;weight&quot;: 3};

            let style = geo_json_d9da339e967d9bd972b255d80b2d1345_styler(feature)
            Object.assign(opts, style)

            return new L.CircleMarker(latlng, opts)
        }

        function geo_json_d9da339e967d9bd972b255d80b2d1345_onEachFeature(feature, layer) {
            layer.on({
                mouseout: function(e) {
                    if(typeof e.target.setStyle === &quot;function&quot;){
                        geo_json_d9da339e967d9bd972b255d80b2d1345.resetStyle(e.target);
                    }
                },
                mouseover: function(e) {
                    if(typeof e.target.setStyle === &quot;function&quot;){
                        const highlightStyle = geo_json_d9da339e967d9bd972b255d80b2d1345_highlighter(e.target.feature)
                        e.target.setStyle(highlightStyle);
                    }
                },
            });
        };
        var geo_json_d9da339e967d9bd972b255d80b2d1345 = L.geoJson(null, {
                onEachFeature: geo_json_d9da339e967d9bd972b255d80b2d1345_onEachFeature,

                style: geo_json_d9da339e967d9bd972b255d80b2d1345_styler,
                pointToLayer: geo_json_d9da339e967d9bd972b255d80b2d1345_pointToLayer
        });

        function geo_json_d9da339e967d9bd972b255d80b2d1345_add (data) {
            geo_json_d9da339e967d9bd972b255d80b2d1345
                .addData(data)
                .addTo(map_01cb15e668cc95a3a6a097422e80a0c5);
        }
            geo_json_d9da339e967d9bd972b255d80b2d1345_add({&quot;bbox&quot;: [13.236588074557865, 46.06423493567362, 13.236588074557865, 46.06423493567362], &quot;features&quot;: [{&quot;bbox&quot;: [13.236588074557865, 46.06423493567362, 13.236588074557865, 46.06423493567362], &quot;geometry&quot;: {&quot;coordinates&quot;: [13.236588074557865, 46.06423493567362], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;0&quot;, &quot;properties&quot;: {&quot;address&quot;: &quot;Piazzale del Castello, 33100, Udine&quot;}, &quot;type&quot;: &quot;Feature&quot;}], &quot;type&quot;: &quot;FeatureCollection&quot;});



    geo_json_d9da339e967d9bd972b255d80b2d1345.bindTooltip(
    function(layer){
    let div = L.DomUtil.create(&#x27;div&#x27;);

    let handleObject = feature=&gt;typeof(feature)==&#x27;object&#x27; ? JSON.stringify(feature) : feature;
    let fields = [&quot;address&quot;];
    let aliases = [&quot;address&quot;];
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



this sounds right ... but "Piazzale"


```python
point_nominatim = gpd.tools.geocode(q,provider="Nominatim",user_agent="Example for the course")
point_nominatim.address[0]
```




    'Piazza Longobardi, Castello, Majano, Udine, Friuli-Venezia Giulia, Italia'



Complety wrong!
```q="Piazza, Castello, Udine, Udine"```
There a comma to remove


```python
q="Piazza Castello, Udine, Udine"
point_nominatim = gpd.tools.geocode(q,provider="Nominatim",user_agent="Example for the course")
point_nominatim.address[0]

```




    'Piazza Castello, Laibacco, Colloredo di Monte Albano, Udine, Friuli-Venezia Giulia, Italia'



again wrong ... 
investigate more values


```python
q="Piazzale Castello, Udine, Udine"
point_nominatim = gpd.tools.geocode(q,provider="Nominatim",user_agent="Example for the course")
point_nominatim.address[0]
```




    'Piazzale della Patria del Friuli, Borgo Gemona, Udine, Friuli-Venezia Giulia, 33100, Italia'




```python
point_nominatim.explore(marker_kwds={"color": "green", "radius": "10"})
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
                #map_4efc767b0dfb729147d22f4cf265a78a {
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

            &lt;div class=&quot;folium-map&quot; id=&quot;map_4efc767b0dfb729147d22f4cf265a78a&quot; &gt;&lt;/div&gt;

&lt;/body&gt;
&lt;script&gt;    

            var map_4efc767b0dfb729147d22f4cf265a78a = L.map(
                &quot;map_4efc767b0dfb729147d22f4cf265a78a&quot;,
                {
                    center: [46.0647686, 13.235658516341397],
                    crs: L.CRS.EPSG3857,
                    zoom: 10,
                    zoomControl: true,
                    preferCanvas: false,
                }
            );
            L.control.scale().addTo(map_4efc767b0dfb729147d22f4cf265a78a);





            var tile_layer_9d26b0db7c6ff0f623323de7f9e07657 = L.tileLayer(
                &quot;https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png&quot;,
                {&quot;attribution&quot;: &quot;Data by \u0026copy; \u003ca href=\&quot;http://openstreetmap.org\&quot;\u003eOpenStreetMap\u003c/a\u003e, under \u003ca href=\&quot;http://www.openstreetmap.org/copyright\&quot;\u003eODbL\u003c/a\u003e.&quot;, &quot;detectRetina&quot;: false, &quot;maxNativeZoom&quot;: 18, &quot;maxZoom&quot;: 18, &quot;minZoom&quot;: 0, &quot;noWrap&quot;: false, &quot;opacity&quot;: 1, &quot;subdomains&quot;: &quot;abc&quot;, &quot;tms&quot;: false}
            ).addTo(map_4efc767b0dfb729147d22f4cf265a78a);


            map_4efc767b0dfb729147d22f4cf265a78a.fitBounds(
                [[46.0647686, 13.235658516341397], [46.0647686, 13.235658516341397]],
                {}
            );


        function geo_json_d4f3f33464294c645277e514bef6b4ed_styler(feature) {
            switch(feature.id) {
                default:
                    return {&quot;fillOpacity&quot;: 0.5, &quot;weight&quot;: 2};
            }
        }
        function geo_json_d4f3f33464294c645277e514bef6b4ed_highlighter(feature) {
            switch(feature.id) {
                default:
                    return {&quot;fillOpacity&quot;: 0.75};
            }
        }
        function geo_json_d4f3f33464294c645277e514bef6b4ed_pointToLayer(feature, latlng) {
            var opts = {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;green&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;green&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: &quot;10&quot;, &quot;stroke&quot;: true, &quot;weight&quot;: 3};

            let style = geo_json_d4f3f33464294c645277e514bef6b4ed_styler(feature)
            Object.assign(opts, style)

            return new L.CircleMarker(latlng, opts)
        }

        function geo_json_d4f3f33464294c645277e514bef6b4ed_onEachFeature(feature, layer) {
            layer.on({
                mouseout: function(e) {
                    if(typeof e.target.setStyle === &quot;function&quot;){
                        geo_json_d4f3f33464294c645277e514bef6b4ed.resetStyle(e.target);
                    }
                },
                mouseover: function(e) {
                    if(typeof e.target.setStyle === &quot;function&quot;){
                        const highlightStyle = geo_json_d4f3f33464294c645277e514bef6b4ed_highlighter(e.target.feature)
                        e.target.setStyle(highlightStyle);
                    }
                },
            });
        };
        var geo_json_d4f3f33464294c645277e514bef6b4ed = L.geoJson(null, {
                onEachFeature: geo_json_d4f3f33464294c645277e514bef6b4ed_onEachFeature,

                style: geo_json_d4f3f33464294c645277e514bef6b4ed_styler,
                pointToLayer: geo_json_d4f3f33464294c645277e514bef6b4ed_pointToLayer
        });

        function geo_json_d4f3f33464294c645277e514bef6b4ed_add (data) {
            geo_json_d4f3f33464294c645277e514bef6b4ed
                .addData(data)
                .addTo(map_4efc767b0dfb729147d22f4cf265a78a);
        }
            geo_json_d4f3f33464294c645277e514bef6b4ed_add({&quot;bbox&quot;: [13.235658516341397, 46.0647686, 13.235658516341397, 46.0647686], &quot;features&quot;: [{&quot;bbox&quot;: [13.235658516341397, 46.0647686, 13.235658516341397, 46.0647686], &quot;geometry&quot;: {&quot;coordinates&quot;: [13.235658516341397, 46.0647686], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;0&quot;, &quot;properties&quot;: {&quot;address&quot;: &quot;Piazzale della Patria del Friuli, Borgo Gemona, Udine, Friuli-Venezia Giulia, 33100, Italia&quot;}, &quot;type&quot;: &quot;Feature&quot;}], &quot;type&quot;: &quot;FeatureCollection&quot;});



    geo_json_d4f3f33464294c645277e514bef6b4ed.bindTooltip(
    function(layer){
    let div = L.DomUtil.create(&#x27;div&#x27;);

    let handleObject = feature=&gt;typeof(feature)==&#x27;object&#x27; ? JSON.stringify(feature) : feature;
    let fields = [&quot;address&quot;];
    let aliases = [&quot;address&quot;];
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



this looks right

More investigation


```python
from geopy.geocoders import Nominatim
geolocator = Nominatim(user_agent="Example for the course")
```


```python
more_values = geolocator.geocode(q,exactly_one=False)
```


```python
more_values
```




    [Location(Piazzale della Patria del Friuli, Borgo Gemona, Udine, Friuli-Venezia Giulia, 33100, Italia, (46.0647686, 13.235658516341397, 0.0))]




```python
more_values[0].raw
```




    {'place_id': 164989388,
     'licence': 'Data © OpenStreetMap contributors, ODbL 1.0. https://osm.org/copyright',
     'osm_type': 'way',
     'osm_id': 243637944,
     'boundingbox': ['46.0644445', '46.0651157', '13.2352794', '13.2360969'],
     'lat': '46.0647686',
     'lon': '13.235658516341397',
     'display_name': 'Piazzale della Patria del Friuli, Borgo Gemona, Udine, Friuli-Venezia Giulia, 33100, Italia',
     'class': 'landuse',
     'type': 'grass',
     'importance': 0.52}



Ok! This isn't a square (piazza), this is an area around the castle of Udine

#### Piazza Italia, Trento


```python
q="Piazza Italia, Trento"
point = gpd.tools.geocode(q, provider="arcgis") 
point.address[0]
```




    'Piazza, Terragnolo, Trento'



this sounds like "The main square (piazza) of the muncipality of Terragnolo in province of Trento"<Br/>
... we can try to specify better the city and the province


```python
q="Piazza Italia, Trento, Provincia di Trento"
point = gpd.tools.geocode(q, provider="arcgis") 
point.address[0]
```




    'Trento, Provincia di Trento'




```python
point.explore(marker_kwds={"color": "green", "radius": "10"})
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
                #map_fe2e37d78ea15307ec96b1f939059df7 {
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

            &lt;div class=&quot;folium-map&quot; id=&quot;map_fe2e37d78ea15307ec96b1f939059df7&quot; &gt;&lt;/div&gt;

&lt;/body&gt;
&lt;script&gt;    

            var map_fe2e37d78ea15307ec96b1f939059df7 = L.map(
                &quot;map_fe2e37d78ea15307ec96b1f939059df7&quot;,
                {
                    center: [46.06787000000003, 11.121080000000063],
                    crs: L.CRS.EPSG3857,
                    zoom: 10,
                    zoomControl: true,
                    preferCanvas: false,
                }
            );
            L.control.scale().addTo(map_fe2e37d78ea15307ec96b1f939059df7);





            var tile_layer_e69164aef1e2143df4c2374020e339dd = L.tileLayer(
                &quot;https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png&quot;,
                {&quot;attribution&quot;: &quot;Data by \u0026copy; \u003ca href=\&quot;http://openstreetmap.org\&quot;\u003eOpenStreetMap\u003c/a\u003e, under \u003ca href=\&quot;http://www.openstreetmap.org/copyright\&quot;\u003eODbL\u003c/a\u003e.&quot;, &quot;detectRetina&quot;: false, &quot;maxNativeZoom&quot;: 18, &quot;maxZoom&quot;: 18, &quot;minZoom&quot;: 0, &quot;noWrap&quot;: false, &quot;opacity&quot;: 1, &quot;subdomains&quot;: &quot;abc&quot;, &quot;tms&quot;: false}
            ).addTo(map_fe2e37d78ea15307ec96b1f939059df7);


            map_fe2e37d78ea15307ec96b1f939059df7.fitBounds(
                [[46.06787000000003, 11.121080000000063], [46.06787000000003, 11.121080000000063]],
                {}
            );


        function geo_json_21cbb086b72d290c78e81e1c37fe5837_styler(feature) {
            switch(feature.id) {
                default:
                    return {&quot;fillOpacity&quot;: 0.5, &quot;weight&quot;: 2};
            }
        }
        function geo_json_21cbb086b72d290c78e81e1c37fe5837_highlighter(feature) {
            switch(feature.id) {
                default:
                    return {&quot;fillOpacity&quot;: 0.75};
            }
        }
        function geo_json_21cbb086b72d290c78e81e1c37fe5837_pointToLayer(feature, latlng) {
            var opts = {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;green&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;green&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: &quot;10&quot;, &quot;stroke&quot;: true, &quot;weight&quot;: 3};

            let style = geo_json_21cbb086b72d290c78e81e1c37fe5837_styler(feature)
            Object.assign(opts, style)

            return new L.CircleMarker(latlng, opts)
        }

        function geo_json_21cbb086b72d290c78e81e1c37fe5837_onEachFeature(feature, layer) {
            layer.on({
                mouseout: function(e) {
                    if(typeof e.target.setStyle === &quot;function&quot;){
                        geo_json_21cbb086b72d290c78e81e1c37fe5837.resetStyle(e.target);
                    }
                },
                mouseover: function(e) {
                    if(typeof e.target.setStyle === &quot;function&quot;){
                        const highlightStyle = geo_json_21cbb086b72d290c78e81e1c37fe5837_highlighter(e.target.feature)
                        e.target.setStyle(highlightStyle);
                    }
                },
            });
        };
        var geo_json_21cbb086b72d290c78e81e1c37fe5837 = L.geoJson(null, {
                onEachFeature: geo_json_21cbb086b72d290c78e81e1c37fe5837_onEachFeature,

                style: geo_json_21cbb086b72d290c78e81e1c37fe5837_styler,
                pointToLayer: geo_json_21cbb086b72d290c78e81e1c37fe5837_pointToLayer
        });

        function geo_json_21cbb086b72d290c78e81e1c37fe5837_add (data) {
            geo_json_21cbb086b72d290c78e81e1c37fe5837
                .addData(data)
                .addTo(map_fe2e37d78ea15307ec96b1f939059df7);
        }
            geo_json_21cbb086b72d290c78e81e1c37fe5837_add({&quot;bbox&quot;: [11.121080000000063, 46.06787000000003, 11.121080000000063, 46.06787000000003], &quot;features&quot;: [{&quot;bbox&quot;: [11.121080000000063, 46.06787000000003, 11.121080000000063, 46.06787000000003], &quot;geometry&quot;: {&quot;coordinates&quot;: [11.121080000000063, 46.06787000000003], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;0&quot;, &quot;properties&quot;: {&quot;address&quot;: &quot;Trento, Provincia di Trento&quot;}, &quot;type&quot;: &quot;Feature&quot;}], &quot;type&quot;: &quot;FeatureCollection&quot;});



    geo_json_21cbb086b72d290c78e81e1c37fe5837.bindTooltip(
    function(layer){
    let div = L.DomUtil.create(&#x27;div&#x27;);

    let handleObject = feature=&gt;typeof(feature)==&#x27;object&#x27; ? JSON.stringify(feature) : feature;
    let fields = [&quot;address&quot;];
    let aliases = [&quot;address&quot;];
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



bad position: the point is good to represent Trento but not the square

change geocoder


```python
point = gpd.tools.geocode(q, provider="nominatim",user_agent="Example for the course")
point.address[0]
```




    'Piazza Italia, Villa Rendena, Porte di Rendena, Comunità delle Giudicarie, Provincia di Trento, Trentino-Alto Adige/Südtirol, 38979, Italia'



again wrong ... add more details? 


```python
q="Piazza Italia, Centro Storico Trento, Trento, Provincia di Trento"
point = gpd.tools.geocode(q, provider="nominatim",user_agent="Example for the course")
point.address[0]
```




    "Piazza Cesare Battisti, Bolghera, Centro storico Trento, Trento, Territorio Val d'Adige, Provincia di Trento, Trentino-Alto Adige/Südtirol, Italia"




```python
point.explore(marker_kwds={"color": "green", "radius": "10"})
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
                #map_38b05565d84cfcaf072cc5f22d2c875d {
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

            &lt;div class=&quot;folium-map&quot; id=&quot;map_38b05565d84cfcaf072cc5f22d2c875d&quot; &gt;&lt;/div&gt;

&lt;/body&gt;
&lt;script&gt;    

            var map_38b05565d84cfcaf072cc5f22d2c875d = L.map(
                &quot;map_38b05565d84cfcaf072cc5f22d2c875d&quot;,
                {
                    center: [46.06928995, 11.123679770661248],
                    crs: L.CRS.EPSG3857,
                    zoom: 10,
                    zoomControl: true,
                    preferCanvas: false,
                }
            );
            L.control.scale().addTo(map_38b05565d84cfcaf072cc5f22d2c875d);





            var tile_layer_000c84e4c7ed393394c7e53262b9108a = L.tileLayer(
                &quot;https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png&quot;,
                {&quot;attribution&quot;: &quot;Data by \u0026copy; \u003ca href=\&quot;http://openstreetmap.org\&quot;\u003eOpenStreetMap\u003c/a\u003e, under \u003ca href=\&quot;http://www.openstreetmap.org/copyright\&quot;\u003eODbL\u003c/a\u003e.&quot;, &quot;detectRetina&quot;: false, &quot;maxNativeZoom&quot;: 18, &quot;maxZoom&quot;: 18, &quot;minZoom&quot;: 0, &quot;noWrap&quot;: false, &quot;opacity&quot;: 1, &quot;subdomains&quot;: &quot;abc&quot;, &quot;tms&quot;: false}
            ).addTo(map_38b05565d84cfcaf072cc5f22d2c875d);


            map_38b05565d84cfcaf072cc5f22d2c875d.fitBounds(
                [[46.06928995, 11.123679770661248], [46.06928995, 11.123679770661248]],
                {}
            );


        function geo_json_a8f93d6c588b044c1d1f6ef683733419_styler(feature) {
            switch(feature.id) {
                default:
                    return {&quot;fillOpacity&quot;: 0.5, &quot;weight&quot;: 2};
            }
        }
        function geo_json_a8f93d6c588b044c1d1f6ef683733419_highlighter(feature) {
            switch(feature.id) {
                default:
                    return {&quot;fillOpacity&quot;: 0.75};
            }
        }
        function geo_json_a8f93d6c588b044c1d1f6ef683733419_pointToLayer(feature, latlng) {
            var opts = {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;green&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;green&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: &quot;10&quot;, &quot;stroke&quot;: true, &quot;weight&quot;: 3};

            let style = geo_json_a8f93d6c588b044c1d1f6ef683733419_styler(feature)
            Object.assign(opts, style)

            return new L.CircleMarker(latlng, opts)
        }

        function geo_json_a8f93d6c588b044c1d1f6ef683733419_onEachFeature(feature, layer) {
            layer.on({
                mouseout: function(e) {
                    if(typeof e.target.setStyle === &quot;function&quot;){
                        geo_json_a8f93d6c588b044c1d1f6ef683733419.resetStyle(e.target);
                    }
                },
                mouseover: function(e) {
                    if(typeof e.target.setStyle === &quot;function&quot;){
                        const highlightStyle = geo_json_a8f93d6c588b044c1d1f6ef683733419_highlighter(e.target.feature)
                        e.target.setStyle(highlightStyle);
                    }
                },
            });
        };
        var geo_json_a8f93d6c588b044c1d1f6ef683733419 = L.geoJson(null, {
                onEachFeature: geo_json_a8f93d6c588b044c1d1f6ef683733419_onEachFeature,

                style: geo_json_a8f93d6c588b044c1d1f6ef683733419_styler,
                pointToLayer: geo_json_a8f93d6c588b044c1d1f6ef683733419_pointToLayer
        });

        function geo_json_a8f93d6c588b044c1d1f6ef683733419_add (data) {
            geo_json_a8f93d6c588b044c1d1f6ef683733419
                .addData(data)
                .addTo(map_38b05565d84cfcaf072cc5f22d2c875d);
        }
            geo_json_a8f93d6c588b044c1d1f6ef683733419_add({&quot;bbox&quot;: [11.123679770661248, 46.06928995, 11.123679770661248, 46.06928995], &quot;features&quot;: [{&quot;bbox&quot;: [11.123679770661248, 46.06928995, 11.123679770661248, 46.06928995], &quot;geometry&quot;: {&quot;coordinates&quot;: [11.123679770661248, 46.06928995], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;0&quot;, &quot;properties&quot;: {&quot;address&quot;: &quot;Piazza Cesare Battisti, Bolghera, Centro storico Trento, Trento, Territorio Val d\u0027Adige, Provincia di Trento, Trentino-Alto Adige/S\u00fcdtirol, Italia&quot;}, &quot;type&quot;: &quot;Feature&quot;}], &quot;type&quot;: &quot;FeatureCollection&quot;});



    geo_json_a8f93d6c588b044c1d1f6ef683733419.bindTooltip(
    function(layer){
    let div = L.DomUtil.create(&#x27;div&#x27;);

    let handleObject = feature=&gt;typeof(feature)==&#x27;object&#x27; ? JSON.stringify(feature) : feature;
    let fields = [&quot;address&quot;];
    let aliases = [&quot;address&quot;];
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



this sounds right but ... this is Piazza Cesare Battisti 


Note:<br/>
Piazza Italia is the former name of Piazza Cesare Battisti

#### Piazza Foroni, Torino


```python
q="Piazza, Foroni, Torino, Torino"
point = gpd.tools.geocode(q, provider="arcgis") 
point.address[0]
```




    'Via Jacopo Foroni, 10154, Torino'



"Via" means "Street". Mistake or this is close to a square with the same name? 


```python
point.explore(marker_kwds={"color": "green", "radius": "10"})
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
                #map_11f8c212d0a35e6f28d3503e63d770ae {
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

            &lt;div class=&quot;folium-map&quot; id=&quot;map_11f8c212d0a35e6f28d3503e63d770ae&quot; &gt;&lt;/div&gt;

&lt;/body&gt;
&lt;script&gt;    

            var map_11f8c212d0a35e6f28d3503e63d770ae = L.map(
                &quot;map_11f8c212d0a35e6f28d3503e63d770ae&quot;,
                {
                    center: [45.08917466108228, 7.696229035329084],
                    crs: L.CRS.EPSG3857,
                    zoom: 10,
                    zoomControl: true,
                    preferCanvas: false,
                }
            );
            L.control.scale().addTo(map_11f8c212d0a35e6f28d3503e63d770ae);





            var tile_layer_01dae3a0bbf292a37b4af5b16f353d87 = L.tileLayer(
                &quot;https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png&quot;,
                {&quot;attribution&quot;: &quot;Data by \u0026copy; \u003ca href=\&quot;http://openstreetmap.org\&quot;\u003eOpenStreetMap\u003c/a\u003e, under \u003ca href=\&quot;http://www.openstreetmap.org/copyright\&quot;\u003eODbL\u003c/a\u003e.&quot;, &quot;detectRetina&quot;: false, &quot;maxNativeZoom&quot;: 18, &quot;maxZoom&quot;: 18, &quot;minZoom&quot;: 0, &quot;noWrap&quot;: false, &quot;opacity&quot;: 1, &quot;subdomains&quot;: &quot;abc&quot;, &quot;tms&quot;: false}
            ).addTo(map_11f8c212d0a35e6f28d3503e63d770ae);


            map_11f8c212d0a35e6f28d3503e63d770ae.fitBounds(
                [[45.08917466108228, 7.696229035329084], [45.08917466108228, 7.696229035329084]],
                {}
            );


        function geo_json_6b108bb179a7fcece0621b0c9c6e45bc_styler(feature) {
            switch(feature.id) {
                default:
                    return {&quot;fillOpacity&quot;: 0.5, &quot;weight&quot;: 2};
            }
        }
        function geo_json_6b108bb179a7fcece0621b0c9c6e45bc_highlighter(feature) {
            switch(feature.id) {
                default:
                    return {&quot;fillOpacity&quot;: 0.75};
            }
        }
        function geo_json_6b108bb179a7fcece0621b0c9c6e45bc_pointToLayer(feature, latlng) {
            var opts = {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;green&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;green&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: &quot;10&quot;, &quot;stroke&quot;: true, &quot;weight&quot;: 3};

            let style = geo_json_6b108bb179a7fcece0621b0c9c6e45bc_styler(feature)
            Object.assign(opts, style)

            return new L.CircleMarker(latlng, opts)
        }

        function geo_json_6b108bb179a7fcece0621b0c9c6e45bc_onEachFeature(feature, layer) {
            layer.on({
                mouseout: function(e) {
                    if(typeof e.target.setStyle === &quot;function&quot;){
                        geo_json_6b108bb179a7fcece0621b0c9c6e45bc.resetStyle(e.target);
                    }
                },
                mouseover: function(e) {
                    if(typeof e.target.setStyle === &quot;function&quot;){
                        const highlightStyle = geo_json_6b108bb179a7fcece0621b0c9c6e45bc_highlighter(e.target.feature)
                        e.target.setStyle(highlightStyle);
                    }
                },
            });
        };
        var geo_json_6b108bb179a7fcece0621b0c9c6e45bc = L.geoJson(null, {
                onEachFeature: geo_json_6b108bb179a7fcece0621b0c9c6e45bc_onEachFeature,

                style: geo_json_6b108bb179a7fcece0621b0c9c6e45bc_styler,
                pointToLayer: geo_json_6b108bb179a7fcece0621b0c9c6e45bc_pointToLayer
        });

        function geo_json_6b108bb179a7fcece0621b0c9c6e45bc_add (data) {
            geo_json_6b108bb179a7fcece0621b0c9c6e45bc
                .addData(data)
                .addTo(map_11f8c212d0a35e6f28d3503e63d770ae);
        }
            geo_json_6b108bb179a7fcece0621b0c9c6e45bc_add({&quot;bbox&quot;: [7.696229035329084, 45.08917466108228, 7.696229035329084, 45.08917466108228], &quot;features&quot;: [{&quot;bbox&quot;: [7.696229035329084, 45.08917466108228, 7.696229035329084, 45.08917466108228], &quot;geometry&quot;: {&quot;coordinates&quot;: [7.696229035329084, 45.08917466108228], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;0&quot;, &quot;properties&quot;: {&quot;address&quot;: &quot;Via Jacopo Foroni, 10154, Torino&quot;}, &quot;type&quot;: &quot;Feature&quot;}], &quot;type&quot;: &quot;FeatureCollection&quot;});



    geo_json_6b108bb179a7fcece0621b0c9c6e45bc.bindTooltip(
    function(layer){
    let div = L.DomUtil.create(&#x27;div&#x27;);

    let handleObject = feature=&gt;typeof(feature)==&#x27;object&#x27; ? JSON.stringify(feature) : feature;
    let fields = [&quot;address&quot;];
    let aliases = [&quot;address&quot;];
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




```python
q="Piazza, Foroni, Torino, Torino"
point = gpd.tools.geocode(q, provider="nominatim",user_agent="Example for the course")
point.address[0]
```




    'Mercato di Piazza Foroni, Piazza Jacopo Foroni, Monte Rosa, Circoscrizione 6, Torino, Piemonte, 10154, Italia'




```python
point.explore(marker_kwds={"color": "green", "radius": "10"})
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
                #map_b37e6a05eab3555a8e22beafd480f103 {
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

            &lt;div class=&quot;folium-map&quot; id=&quot;map_b37e6a05eab3555a8e22beafd480f103&quot; &gt;&lt;/div&gt;

&lt;/body&gt;
&lt;script&gt;    

            var map_b37e6a05eab3555a8e22beafd480f103 = L.map(
                &quot;map_b37e6a05eab3555a8e22beafd480f103&quot;,
                {
                    center: [45.090020249999995, 7.694977880411744],
                    crs: L.CRS.EPSG3857,
                    zoom: 10,
                    zoomControl: true,
                    preferCanvas: false,
                }
            );
            L.control.scale().addTo(map_b37e6a05eab3555a8e22beafd480f103);





            var tile_layer_86ca6a16a3070b3a92df59229d266224 = L.tileLayer(
                &quot;https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png&quot;,
                {&quot;attribution&quot;: &quot;Data by \u0026copy; \u003ca href=\&quot;http://openstreetmap.org\&quot;\u003eOpenStreetMap\u003c/a\u003e, under \u003ca href=\&quot;http://www.openstreetmap.org/copyright\&quot;\u003eODbL\u003c/a\u003e.&quot;, &quot;detectRetina&quot;: false, &quot;maxNativeZoom&quot;: 18, &quot;maxZoom&quot;: 18, &quot;minZoom&quot;: 0, &quot;noWrap&quot;: false, &quot;opacity&quot;: 1, &quot;subdomains&quot;: &quot;abc&quot;, &quot;tms&quot;: false}
            ).addTo(map_b37e6a05eab3555a8e22beafd480f103);


            map_b37e6a05eab3555a8e22beafd480f103.fitBounds(
                [[45.090020249999995, 7.694977880411744], [45.090020249999995, 7.694977880411744]],
                {}
            );


        function geo_json_28a073769dcacddcdead01e722a59047_styler(feature) {
            switch(feature.id) {
                default:
                    return {&quot;fillOpacity&quot;: 0.5, &quot;weight&quot;: 2};
            }
        }
        function geo_json_28a073769dcacddcdead01e722a59047_highlighter(feature) {
            switch(feature.id) {
                default:
                    return {&quot;fillOpacity&quot;: 0.75};
            }
        }
        function geo_json_28a073769dcacddcdead01e722a59047_pointToLayer(feature, latlng) {
            var opts = {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;green&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;green&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: &quot;10&quot;, &quot;stroke&quot;: true, &quot;weight&quot;: 3};

            let style = geo_json_28a073769dcacddcdead01e722a59047_styler(feature)
            Object.assign(opts, style)

            return new L.CircleMarker(latlng, opts)
        }

        function geo_json_28a073769dcacddcdead01e722a59047_onEachFeature(feature, layer) {
            layer.on({
                mouseout: function(e) {
                    if(typeof e.target.setStyle === &quot;function&quot;){
                        geo_json_28a073769dcacddcdead01e722a59047.resetStyle(e.target);
                    }
                },
                mouseover: function(e) {
                    if(typeof e.target.setStyle === &quot;function&quot;){
                        const highlightStyle = geo_json_28a073769dcacddcdead01e722a59047_highlighter(e.target.feature)
                        e.target.setStyle(highlightStyle);
                    }
                },
            });
        };
        var geo_json_28a073769dcacddcdead01e722a59047 = L.geoJson(null, {
                onEachFeature: geo_json_28a073769dcacddcdead01e722a59047_onEachFeature,

                style: geo_json_28a073769dcacddcdead01e722a59047_styler,
                pointToLayer: geo_json_28a073769dcacddcdead01e722a59047_pointToLayer
        });

        function geo_json_28a073769dcacddcdead01e722a59047_add (data) {
            geo_json_28a073769dcacddcdead01e722a59047
                .addData(data)
                .addTo(map_b37e6a05eab3555a8e22beafd480f103);
        }
            geo_json_28a073769dcacddcdead01e722a59047_add({&quot;bbox&quot;: [7.694977880411744, 45.090020249999995, 7.694977880411744, 45.090020249999995], &quot;features&quot;: [{&quot;bbox&quot;: [7.694977880411744, 45.090020249999995, 7.694977880411744, 45.090020249999995], &quot;geometry&quot;: {&quot;coordinates&quot;: [7.694977880411744, 45.090020249999995], &quot;type&quot;: &quot;Point&quot;}, &quot;id&quot;: &quot;0&quot;, &quot;properties&quot;: {&quot;address&quot;: &quot;Mercato di Piazza Foroni, Piazza Jacopo Foroni, Monte Rosa, Circoscrizione 6, Torino, Piemonte, 10154, Italia&quot;}, &quot;type&quot;: &quot;Feature&quot;}], &quot;type&quot;: &quot;FeatureCollection&quot;});



    geo_json_28a073769dcacddcdead01e722a59047.bindTooltip(
    function(layer){
    let div = L.DomUtil.create(&#x27;div&#x27;);

    let handleObject = feature=&gt;typeof(feature)==&#x27;object&#x27; ? JSON.stringify(feature) : feature;
    let fields = [&quot;address&quot;];
    let aliases = [&quot;address&quot;];
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



## find the administrative border of “comunità di valle” (community of valley) of Province Autonomous of Trento


```python
from owslib.csw import CatalogueServiceWeb
from owslib.fes import PropertyIsLike
import geopandas as gpd
from matplotlib import pyplot as plt
```

We start from the italian national repository - [http://geodati.gov.it](http://geodati.gov.it)


```python
csw = CatalogueServiceWeb("http://geodati.gov.it/RNDT/csw")
```


```python
query = PropertyIsLike('csw:AnyText', 'Comunità di valle')
```


```python
csw.getrecords2(constraints=[query],maxrecords=5)
```


```python
for rec in csw.records:
  print(rec + " - " + csw.records[rec].title)
```

    p_bi:4c07e84b-088e-488f-a144-5dabb3c4b6d0 - Provincia di Biella - vegetazione della Valle Elvo
    p_TN:58604ed2-ac1d-4f78-a00c-514fd3562c51 - Limite Comunità di valle
    r_emiro:2015-06-04T161431 - Itinerari geologico-ambientali nella Valle del Marecchia
    r_abruzz:00047:20091110:151939 - Comunità Montane Regione Abruzzo
    r_lombar:000028:20211130:115710 - Carta Ittica Regionale - WMS



```python
id_record="p_TN:58604ed2-ac1d-4f78-a00c-514fd3562c51"
```


```python
record = csw.records[id_record]
```


```python
record.abstract
```




    "Rappresenta il limite delle Comunità di valle, le quali sono enti pubblici locali a struttura associativa costituiti obbligatoriamente dai comuni compresi in ciascun territorio individuato ai sensi dell'art.12 comma 2 (LP3-2006 art 14 comma2) ad esse e ai Comuni di Trento e Rovereto sono trasferite numerose competenze che ora sono in capo alla Provincia, ovviamente fatte salve le competenze dei comuni e delle amministrazioni separate dei beni di usi civico.Intesa tra la Provincia e il Consiglio delle Autonomie locali approvato nella seduta del 16 marzo 2007 concernente Individuazione dei territori delle Comunità ai sensi dell'articolo 12 della legge provinciale 16 giugno 2006, n. 3 (Norme in materia di governo dell'autonomia del Trentino).NB: PER LA TABELLA DEGLI ATTRIBUTI E' STATO UTILIZZATO IL SET DI CARATTERI UNICODE UTF-8"




```python
for reference in record.references:
  print(reference['scheme'])
  print(reference['url'])
```

    urn:x-esri:specification:ServiceType:ArcIMS:Metadata:Server
    http://www.territorio.provincia.tn.it
    urn:x-esri:specification:ServiceType:ArcIMS:Metadata:Server
    https://idt.provincia.tn.it/idt/vector/p_TN_58604ed2-ac1d-4f78-a00c-514fd3562c51.zip
    urn:x-esri:specification:ServiceType:ArcIMS:Metadata:Document
    https://geodati.gov.it/geoportalRNDTPA/csw?getxml=%7B995A9C38-26FD-4564-8539-68744B2A46D2%7D



```python
valley_communities = gpd.read_file('https://siat.provincia.tn.it/IDT/vector/public/p_tn_58604ed2-ac1d-4f78-a00c-514fd3562c51.zip')
```


```python
valley_communities.head(5)
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
      <th>classid</th>
      <th>sede</th>
      <th>nome</th>
      <th>struttura</th>
      <th>struttura_</th>
      <th>dataini</th>
      <th>datafine</th>
      <th>fkfonte</th>
      <th>fktfonte_d</th>
      <th>fktipoelab</th>
      <th>fktipoel_d</th>
      <th>fkscala</th>
      <th>fkscala_d</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>181</td>
      <td>AMB002_14</td>
      <td>ANDALO</td>
      <td>COMUNITÀ DELLA PAGANELLA</td>
      <td>S133</td>
      <td>Servizio Catasto</td>
      <td>2020/01/01 00:00:00.000</td>
      <td>None</td>
      <td>05</td>
      <td>altre fonti</td>
      <td>01</td>
      <td>manuale</td>
      <td>03</td>
      <td>10000</td>
      <td>POLYGON ((659718.849 5118603.995, 659717.453 5...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>182</td>
      <td>AMB002_8</td>
      <td>TIONE DI TRENTO</td>
      <td>COMUNITÀ DELLE GIUDICARIE</td>
      <td>S133</td>
      <td>Servizio Catasto</td>
      <td>2020/01/01 00:00:00.000</td>
      <td>None</td>
      <td>05</td>
      <td>altre fonti</td>
      <td>01</td>
      <td>manuale</td>
      <td>03</td>
      <td>10000</td>
      <td>POLYGON ((626847.878 5074565.314, 626878.525 5...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>183</td>
      <td>AMB002_12</td>
      <td>LAVARONE</td>
      <td>MAGNIFICA COMUNITÀ DEGLI ALTIPIANI CIMBRI</td>
      <td>S133</td>
      <td>Servizio Catasto</td>
      <td>2020/01/01 00:00:00.000</td>
      <td>None</td>
      <td>05</td>
      <td>altre fonti</td>
      <td>01</td>
      <td>manuale</td>
      <td>03</td>
      <td>10000</td>
      <td>POLYGON ((675134.859 5087715.705, 675136.500 5...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>184</td>
      <td>AMB002_15</td>
      <td>TRENTO</td>
      <td>TERRITORIO VAL D'ADIGE</td>
      <td>S133</td>
      <td>Servizio Catasto</td>
      <td>2020/01/01 00:00:00.000</td>
      <td>None</td>
      <td>05</td>
      <td>altre fonti</td>
      <td>01</td>
      <td>manuale</td>
      <td>03</td>
      <td>10000</td>
      <td>POLYGON ((663458.060 5094288.411, 663453.437 5...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>185</td>
      <td>AMB002_1</td>
      <td>CAVALESE</td>
      <td>COMUNITÀ TERRITORIALE DELLA VAL DI FIEMME</td>
      <td>S133</td>
      <td>Servizio Catasto</td>
      <td>2020/01/01 00:00:00.000</td>
      <td>None</td>
      <td>05</td>
      <td>altre fonti</td>
      <td>01</td>
      <td>manuale</td>
      <td>03</td>
      <td>10000</td>
      <td>POLYGON ((681770.000 5126270.500, 681789.000 5...</td>
    </tr>
  </tbody>
</table>
</div>




```python
valley_communities.plot()
plt.show()
```


    
![png](03_solution_exercise_retrieving_data_from_sdi_files/03_solution_exercise_retrieving_data_from_sdi_61_0.png)
    



```python
valley_communities.head(5)
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
      <th>classid</th>
      <th>sede</th>
      <th>nome</th>
      <th>struttura</th>
      <th>struttura_</th>
      <th>dataini</th>
      <th>datafine</th>
      <th>fkfonte</th>
      <th>fktfonte_d</th>
      <th>fktipoelab</th>
      <th>fktipoel_d</th>
      <th>fkscala</th>
      <th>fkscala_d</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>181</td>
      <td>AMB002_14</td>
      <td>ANDALO</td>
      <td>COMUNITÀ DELLA PAGANELLA</td>
      <td>S133</td>
      <td>Servizio Catasto</td>
      <td>2020/01/01 00:00:00.000</td>
      <td>None</td>
      <td>05</td>
      <td>altre fonti</td>
      <td>01</td>
      <td>manuale</td>
      <td>03</td>
      <td>10000</td>
      <td>POLYGON ((659718.849 5118603.995, 659717.453 5...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>182</td>
      <td>AMB002_8</td>
      <td>TIONE DI TRENTO</td>
      <td>COMUNITÀ DELLE GIUDICARIE</td>
      <td>S133</td>
      <td>Servizio Catasto</td>
      <td>2020/01/01 00:00:00.000</td>
      <td>None</td>
      <td>05</td>
      <td>altre fonti</td>
      <td>01</td>
      <td>manuale</td>
      <td>03</td>
      <td>10000</td>
      <td>POLYGON ((626847.878 5074565.314, 626878.525 5...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>183</td>
      <td>AMB002_12</td>
      <td>LAVARONE</td>
      <td>MAGNIFICA COMUNITÀ DEGLI ALTIPIANI CIMBRI</td>
      <td>S133</td>
      <td>Servizio Catasto</td>
      <td>2020/01/01 00:00:00.000</td>
      <td>None</td>
      <td>05</td>
      <td>altre fonti</td>
      <td>01</td>
      <td>manuale</td>
      <td>03</td>
      <td>10000</td>
      <td>POLYGON ((675134.859 5087715.705, 675136.500 5...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>184</td>
      <td>AMB002_15</td>
      <td>TRENTO</td>
      <td>TERRITORIO VAL D'ADIGE</td>
      <td>S133</td>
      <td>Servizio Catasto</td>
      <td>2020/01/01 00:00:00.000</td>
      <td>None</td>
      <td>05</td>
      <td>altre fonti</td>
      <td>01</td>
      <td>manuale</td>
      <td>03</td>
      <td>10000</td>
      <td>POLYGON ((663458.060 5094288.411, 663453.437 5...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>185</td>
      <td>AMB002_1</td>
      <td>CAVALESE</td>
      <td>COMUNITÀ TERRITORIALE DELLA VAL DI FIEMME</td>
      <td>S133</td>
      <td>Servizio Catasto</td>
      <td>2020/01/01 00:00:00.000</td>
      <td>None</td>
      <td>05</td>
      <td>altre fonti</td>
      <td>01</td>
      <td>manuale</td>
      <td>03</td>
      <td>10000</td>
      <td>POLYGON ((681770.000 5126270.500, 681789.000 5...</td>
    </tr>
  </tbody>
</table>
</div>



## identify all the rivers inside the smallest community of valley of Trentino

### identify the smallest community of valley


```python
smallest_community = valley_communities[valley_communities.area == valley_communities.area.min()]
```


```python
smallest_community
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
      <th>classid</th>
      <th>sede</th>
      <th>nome</th>
      <th>struttura</th>
      <th>struttura_</th>
      <th>dataini</th>
      <th>datafine</th>
      <th>fkfonte</th>
      <th>fktfonte_d</th>
      <th>fktipoelab</th>
      <th>fktipoel_d</th>
      <th>fkscala</th>
      <th>fkscala_d</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>10</th>
      <td>191</td>
      <td>AMB002_13</td>
      <td>MEZZOCORONA</td>
      <td>COMUNITÀ ROTALIANA-KÖNIGSBERG</td>
      <td>S133</td>
      <td>Servizio Catasto</td>
      <td>2020/01/01 00:00:00.000</td>
      <td>None</td>
      <td>05</td>
      <td>altre fonti</td>
      <td>01</td>
      <td>manuale</td>
      <td>03</td>
      <td>10000</td>
      <td>MULTIPOLYGON (((656059.164 5112838.836, 656030...</td>
    </tr>
  </tbody>
</table>
</div>




```python
smallest_community.nome
```




    10    COMUNITÀ ROTALIANA-KÖNIGSBERG
    Name: nome, dtype: object




```python
smallest_community.plot()
plt.show()
```


    
![png](03_solution_exercise_retrieving_data_from_sdi_files/03_solution_exercise_retrieving_data_from_sdi_68_0.png)
    



```python
smallest_community.explore()
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
                #map_040be6edaece3d8a03a4d3c172ea3666 {
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

            &lt;div class=&quot;folium-map&quot; id=&quot;map_040be6edaece3d8a03a4d3c172ea3666&quot; &gt;&lt;/div&gt;

&lt;/body&gt;
&lt;script&gt;    

            var map_040be6edaece3d8a03a4d3c172ea3666 = L.map(
                &quot;map_040be6edaece3d8a03a4d3c172ea3666&quot;,
                {
                    center: [46.20440035041479, 11.111657173067192],
                    crs: L.CRS.EPSG3857,
                    zoom: 10,
                    zoomControl: true,
                    preferCanvas: false,
                }
            );
            L.control.scale().addTo(map_040be6edaece3d8a03a4d3c172ea3666);





            var tile_layer_efd8a9ca48be1f6ee9a02df8c3be62b6 = L.tileLayer(
                &quot;https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png&quot;,
                {&quot;attribution&quot;: &quot;Data by \u0026copy; \u003ca href=\&quot;http://openstreetmap.org\&quot;\u003eOpenStreetMap\u003c/a\u003e, under \u003ca href=\&quot;http://www.openstreetmap.org/copyright\&quot;\u003eODbL\u003c/a\u003e.&quot;, &quot;detectRetina&quot;: false, &quot;maxNativeZoom&quot;: 18, &quot;maxZoom&quot;: 18, &quot;minZoom&quot;: 0, &quot;noWrap&quot;: false, &quot;opacity&quot;: 1, &quot;subdomains&quot;: &quot;abc&quot;, &quot;tms&quot;: false}
            ).addTo(map_040be6edaece3d8a03a4d3c172ea3666);


            map_040be6edaece3d8a03a4d3c172ea3666.fitBounds(
                [[46.12500990155531, 11.01943938362622], [46.283790799274264, 11.203874962508163]],
                {}
            );


        function geo_json_ee33d01b0f11ca4280d41c125209143b_styler(feature) {
            switch(feature.id) {
                default:
                    return {&quot;fillOpacity&quot;: 0.5, &quot;weight&quot;: 2};
            }
        }
        function geo_json_ee33d01b0f11ca4280d41c125209143b_highlighter(feature) {
            switch(feature.id) {
                default:
                    return {&quot;fillOpacity&quot;: 0.75};
            }
        }
        function geo_json_ee33d01b0f11ca4280d41c125209143b_pointToLayer(feature, latlng) {
            var opts = {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#3388ff&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#3388ff&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 2, &quot;stroke&quot;: true, &quot;weight&quot;: 3};

            let style = geo_json_ee33d01b0f11ca4280d41c125209143b_styler(feature)
            Object.assign(opts, style)

            return new L.CircleMarker(latlng, opts)
        }

        function geo_json_ee33d01b0f11ca4280d41c125209143b_onEachFeature(feature, layer) {
            layer.on({
                mouseout: function(e) {
                    if(typeof e.target.setStyle === &quot;function&quot;){
                        geo_json_ee33d01b0f11ca4280d41c125209143b.resetStyle(e.target);
                    }
                },
                mouseover: function(e) {
                    if(typeof e.target.setStyle === &quot;function&quot;){
                        const highlightStyle = geo_json_ee33d01b0f11ca4280d41c125209143b_highlighter(e.target.feature)
                        e.target.setStyle(highlightStyle);
                    }
                },
            });
        };
        var geo_json_ee33d01b0f11ca4280d41c125209143b = L.geoJson(null, {
                onEachFeature: geo_json_ee33d01b0f11ca4280d41c125209143b_onEachFeature,

                style: geo_json_ee33d01b0f11ca4280d41c125209143b_styler,
                pointToLayer: geo_json_ee33d01b0f11ca4280d41c125209143b_pointToLayer
        });

        function geo_json_ee33d01b0f11ca4280d41c125209143b_add (data) {
            geo_json_ee33d01b0f11ca4280d41c125209143b
                .addData(data)
                .addTo(map_040be6edaece3d8a03a4d3c172ea3666);
        }
            geo_json_ee33d01b0f11ca4280d41c125209143b_add({&quot;bbox&quot;: [11.01943938362622, 46.12500990155531, 11.203874962508163, 46.283790799274264], &quot;features&quot;: [{&quot;bbox&quot;: [11.01943938362622, 46.12500990155531, 11.203874962508163, 46.283790799274264], &quot;geometry&quot;: {&quot;coordinates&quot;: [[[[11.020957052777154, 46.15125839183139], [11.02060038134018, 46.15151772820173], [11.01943938362622, 46.15235412631262], [11.020320543510683, 46.15305188585593], [11.021045638430536, 46.154196924523994], [11.021325127291526, 46.155015865883385], [11.021621219226272, 46.155876405044054], [11.021896825260841, 46.15668028572479], [11.022050411112884, 46.15712442073853], [11.022455480155692, 46.15743959106001], [11.02279946589827, 46.157869881767205], [11.02302444751628, 46.15802299550836], [11.023171629229571, 46.158200759879556], [11.023742998157026, 46.15834741769689], [11.023783405516987, 46.15835778924456], [11.023940127681502, 46.15847108824281], [11.024120084860002, 46.158903116487785], [11.024322520813676, 46.15958260973757], [11.024437657790187, 46.15975395804877], [11.024607804898112, 46.16014659418624], [11.02460666332943, 46.16048059015302], [11.024874146311879, 46.16083309975943], [11.024898195537185, 46.16116897664896], [11.024992099983477, 46.16153736229698], [11.025076330537013, 46.16200715927091], [11.025011246475875, 46.16215028135055], [11.02495829530434, 46.16235020868738], [11.024970127456982, 46.16262695439991], [11.024983157001852, 46.16270818094481], [11.025133141405538, 46.16291614859719], [11.025205388763299, 46.163013780876035], [11.025256485543037, 46.163082695794124], [11.02529020567356, 46.16313446370078], [11.025366876911605, 46.163215725901274], [11.025400127604428, 46.16325470160569], [11.025439423974195, 46.16332149832785], [11.025469455590523, 46.16336402212772], [11.025504679021234, 46.163411108579524], [11.025576529537306, 46.163479694731976], [11.025618470066135, 46.16349985836132], [11.02568300802316, 46.163524315170925], [11.025777970746281, 46.16355637836302], [11.02590823469842, 46.16368323715076], [11.02602691171001, 46.16381379223263], [11.02623406168868, 46.164031042101094], [11.026441507890524, 46.16426071909789], [11.026536591178747, 46.16435824809066], [11.026642839066382, 46.164477390837504], [11.026710249277288, 46.16455464049415], [11.026760223411484, 46.16462816288706], [11.026767936086095, 46.16470133776655], [11.026777410299161, 46.16480438182055], [11.02687164695294, 46.16487384414999], [11.026956567382735, 46.16494900559422], [11.027076178884709, 46.165066731712784], [11.02711501396207, 46.16510644550381], [11.027202258385033, 46.165144238830514], [11.027261774028435, 46.165171835163534], [11.027341782277226, 46.165350422478774], [11.027398540366958, 46.165470686032805], [11.027419173846747, 46.165514832528295], [11.027374028970153, 46.16558193189132], [11.027081996122986, 46.16600141314348], [11.027078141632385, 46.16618760191412], [11.027084791498359, 46.166400985664005], [11.027125529119578, 46.16646079459431], [11.027227503893045, 46.166591992388305], [11.027336483946295, 46.166865001781936], [11.027345688056595, 46.16689490558097], [11.027380818926218, 46.16701499921325], [11.027458724080423, 46.167333474347984], [11.027498850433016, 46.16747295567772], [11.027506894014289, 46.1677731216555], [11.027521675177804, 46.16786077818938], [11.02759310148229, 46.168217171332564], [11.027630042428205, 46.16840271284511], [11.027610213144282, 46.16840532183136], [11.02762300265509, 46.16857004597213], [11.027799897742955, 46.168790157061494], [11.027925202999192, 46.168958794087146], [11.028165151132693, 46.16917558030996], [11.028206221596562, 46.16916753129286], [11.02832524925451, 46.1691933595536], [11.028614523364066, 46.16925371116639], [11.028801465589948, 46.169420657638994], [11.029017080741438, 46.16962162402211], [11.029304880593761, 46.16973498473721], [11.029733969025767, 46.16970593177599], [11.02986997180452, 46.16969934859377], [11.029997279898597, 46.16968934769879], [11.030128823457543, 46.16970129875678], [11.030257995178939, 46.16971864980883], [11.030805592150582, 46.169859538572936], [11.03124841455606, 46.16997073455986], [11.031926471667001, 46.170284723761476], [11.032569663036066, 46.17034532016307], [11.03271301934771, 46.170539979300266], [11.03301183438295, 46.17069611780507], [11.033055855158546, 46.17075139735281], [11.033095381442946, 46.17081465058886], [11.03318770489005, 46.17089230633598], [11.033236369949416, 46.170929303646425], [11.033293255986624, 46.17096615464471], [11.033514346701919, 46.17107795320816], [11.033672065300488, 46.17128375121179], [11.033745550941354, 46.171352454569835], [11.033982802343555, 46.171405145650525], [11.034190850907871, 46.17143740036479], [11.034568144169906, 46.1715464161128], [11.034946098910199, 46.17165493264742], [11.035097212499975, 46.171662957473174], [11.035149318247292, 46.171681675482766], [11.03550466137557, 46.17190633561522], [11.035559600057539, 46.1719321469693], [11.035733133592721, 46.17196870477495], [11.035798111558078, 46.171981716098664], [11.03589083995215, 46.17198661270793], [11.035967846585484, 46.17200667269464], [11.036140902078182, 46.17205812173177], [11.036209709854472, 46.172083566577676], [11.03630337444531, 46.17213238177162], [11.03646349351854, 46.17221716139602], [11.036595758042605, 46.172305985420586], [11.036747027380727, 46.17231817262477], [11.036833981569425, 46.17232912471608], [11.036892192931589, 46.172350709767024], [11.036973198984098, 46.17245583045647], [11.037030994898196, 46.172535765506446], [11.037042962696887, 46.172558174832254], [11.036963346651984, 46.17272271542243], [11.037000699797328, 46.17280599156221], [11.037161149531059, 46.1728315881665], [11.03725589985928, 46.172844782293375], [11.037495424388673, 46.17294945767998], [11.037619799015216, 46.17302225194476], [11.037723578838266, 46.173140658662476], [11.037851401690098, 46.17321398651669], [11.038009417983698, 46.173292729056804], [11.038048395800262, 46.17332715854386], [11.038103141369362, 46.17339405011748], [11.038232282292679, 46.173456638030096], [11.038325454279896, 46.17349664941706], [11.0384122798888, 46.173527248600884], [11.038563005688937, 46.173570995760876], [11.038599985386961, 46.173597721377334], [11.038693532718982, 46.17374059935098], [11.03881893403788, 46.17388743344565], [11.038956453385678, 46.17397261277947], [11.039122992003676, 46.17407006187222], [11.039182388051305, 46.1741004355426], [11.039299034439935, 46.174121690947764], [11.039484032441363, 46.17411886553076], [11.039580370646664, 46.174128576453036], [11.03966051262052, 46.174145244114946], [11.03973816350299, 46.174205772722814], [11.039803494290046, 46.174192699866836], [11.039877485048141, 46.17417280465605], [11.0400076364816, 46.17404471393307], [11.040080719267511, 46.17397724101773], [11.040128286531697, 46.17392876512066], [11.04018688122019, 46.173775313650125], [11.040131260813817, 46.17352269470421], [11.04009467382166, 46.17336760870783], [11.040087367681087, 46.17328129674791], [11.040100429725985, 46.17318962034823], [11.040120084823023, 46.173164979830574], [11.040198709757792, 46.1731221408697], [11.040297652252027, 46.173100371086946], [11.040421628024399, 46.17308815582048], [11.040527634355954, 46.17309054930239], [11.04056610642566, 46.17310200708983], [11.040740632387648, 46.17320533597456], [11.041078669897969, 46.17339218647544], [11.041221760293613, 46.17347964491522], [11.041277837962516, 46.17352222156955], [11.04137947321566, 46.17354540994875], [11.041422637791271, 46.17357250042237], [11.041501591493683, 46.17364967389587], [11.041572591491725, 46.17376199500958], [11.041597602010171, 46.173798696959224], [11.04168161218923, 46.17384577516298], [11.04176854691146, 46.17388851464893], [11.041859527389981, 46.17393089005839], [11.041964986777868, 46.173974012778764], [11.042129525536062, 46.17402965320122], [11.042217039116315, 46.17406023719849], [11.042337490997364, 46.174091661235], [11.042563214553264, 46.174133348263084], [11.042693573243602, 46.174154593233816], [11.042815000364547, 46.174184570527274], [11.042944341666336, 46.17421535873754], [11.043150058108044, 46.17426287936392], [11.04336424343293, 46.17431075620186], [11.043449358878489, 46.17433209493929], [11.043533336147068, 46.17437821983119], [11.043688938629394, 46.17445163968661], [11.043867062141533, 46.17452227516999], [11.044043762948462, 46.17459150703653], [11.044188584180977, 46.1746884561202], [11.044417196551512, 46.17486344246396], [11.044480104601142, 46.17489105169945], [11.044555945762443, 46.17491184114563], [11.04466256445938, 46.17493065102837], [11.044981297007126, 46.175039965799755], [11.045073920799886, 46.175041880161146], [11.045187827518147, 46.17501644865285], [11.045351502204786, 46.17497005977204], [11.045496626909628, 46.17495496005014], [11.045677747271338, 46.17484753442149], [11.045716051912542, 46.17477629796651], [11.04574009346991, 46.17471252611118], [11.045760015162701, 46.174662698958585], [11.045788382230596, 46.174641949777396], [11.045834733573798, 46.174620878598866], [11.04603727090877, 46.174559504945414], [11.046232254770455, 46.174502433527536], [11.046460193342906, 46.174436258329145], [11.046704255549306, 46.17438545114976], [11.046791038082198, 46.17439163606723], [11.046872135039788, 46.1744062574337], [11.047127381282055, 46.17449515234825], [11.047238920967194, 46.17454494807273], [11.047323215842892, 46.17462261715501], [11.047392952235237, 46.174700547031776], [11.04744310214457, 46.174781804469724], [11.047473888845381, 46.17485917208015], [11.047474878507998, 46.17499138776498], [11.04746254466689, 46.17511613070408], [11.047454611079992, 46.17522420677431], [11.04749648787853, 46.17529174116204], [11.04761067524947, 46.17538971107069], [11.0476814137923, 46.17542535414126], [11.04786090448235, 46.17550488889888], [11.047951495380543, 46.175544343355426], [11.048129152854543, 46.17557449788773], [11.04871625702452, 46.17566875199237], [11.04879736196027, 46.1757065898396], [11.049005608751296, 46.17582192254989], [11.04906626499643, 46.17583988554395], [11.049164951371102, 46.17584347387571], [11.04929457330482, 46.17582626600145], [11.049380548718617, 46.17578751847117], [11.049566418166245, 46.17569279847064], [11.049639261388956, 46.17562362392403], [11.049784444709362, 46.17549480905103], [11.050070128065123, 46.17538788206693], [11.050206422902487, 46.175365790778194], [11.050479091253711, 46.175369828278], [11.050545357861397, 46.17537596175243], [11.050592822402866, 46.17538624273762], [11.051076139398756, 46.17552402048399], [11.051245322800884, 46.17555670360897], [11.051400206332794, 46.17557356919216], [11.051702032553262, 46.17548599432948], [11.051775301743975, 46.175382281433045], [11.051927821329969, 46.175155488614465], [11.05062573330287, 46.173959797314005], [11.049418991295004, 46.17270124147208], [11.048594934278272, 46.17183301923618], [11.048029684070732, 46.17123233924449], [11.047838644093229, 46.17107216538778], [11.04695706160042, 46.170314979761784], [11.046300634011793, 46.169750215616205], [11.046311301292649, 46.16959428527269], [11.046373307808393, 46.16882662304617], [11.046401050306207, 46.1684103450538], [11.046088710515042, 46.16791800095033], [11.045973211097136, 46.16759154312728], [11.046021170741598, 46.167258986197126], [11.045968904134696, 46.16704429081611], [11.045873570104092, 46.166414467397], [11.045985830844044, 46.1661145522448], [11.045892050533148, 46.165471127306404], [11.045830472997856, 46.165002868344395], [11.045686578780561, 46.16475468974207], [11.04547970343285, 46.16441619504224], [11.045123393672688, 46.163823162386045], [11.044761256350023, 46.1634101905184], [11.044383278936248, 46.16301815897488], [11.04397061093116, 46.16257898595503], [11.043610668224476, 46.161832564054755], [11.043512586260182, 46.161470489419266], [11.043392463730703, 46.16101926059875], [11.043285367423119, 46.16057038839981], [11.043251016073512, 46.15978158898815], [11.043176032475252, 46.159505741971515], [11.043091248030015, 46.159215067694454], [11.043046310040895, 46.158714451741425], [11.042929414364766, 46.15839586221309], [11.042827270003803, 46.15824119992561], [11.041668147791864, 46.15781993992331], [11.041432402535799, 46.157505528685505], [11.041187886609295, 46.15717627118857], [11.040481569599738, 46.1572024598193], [11.039759016768842, 46.15631664278543], [11.038914728987812, 46.15527939691081], [11.038169572813787, 46.15436468246596], [11.037901022038877, 46.15341145907202], [11.037861212285094, 46.15222197497384], [11.037824491527209, 46.151199605708825], [11.037807628526824, 46.15067086627789], [11.037781911295292, 46.149538117878365], [11.037767349224946, 46.148642516686664], [11.037716972264892, 46.147166506454695], [11.037690952037842, 46.14646161670977], [11.03761690996524, 46.14441261845419], [11.037592020320155, 46.14366658867743], [11.037482645644447, 46.14340491503396], [11.037434516876845, 46.14328477638645], [11.03740528098436, 46.143212694917906], [11.037129928039427, 46.14251965273221], [11.036969260505778, 46.142138162776845], [11.036966725519338, 46.142131776639964], [11.036954449162714, 46.14210085001051], [11.036927536152232, 46.14202793805616], [11.036837057885196, 46.14179068253947], [11.036662112734255, 46.141877188578405], [11.03534586331948, 46.14245866299026], [11.033943733753755, 46.143082152379975], [11.032644390797454, 46.1436407936823], [11.031155828897125, 46.144270286308306], [11.029525752767665, 46.14496977552182], [11.02865433303694, 46.145327262609776], [11.027453023780673, 46.145825601819375], [11.026434313018813, 46.1463161900258], [11.025607253387818, 46.14656036288566], [11.025158471985836, 46.14670332258288], [11.02490866759108, 46.14686525543068], [11.024472981281926, 46.147232988607115], [11.02371700856577, 46.14784940187621], [11.023162606353846, 46.14826873424289], [11.022058959173961, 46.14911629984939], [11.021739539635258, 46.149963482322924], [11.021546041543557, 46.15042592232854], [11.021368285429741, 46.15072157750119], [11.021223332008365, 46.15096265020825], [11.020957052777154, 46.15125839183139]]], [[[11.136251451056927, 46.1807469282379], [11.136021702830469, 46.1791829192214], [11.135828231646439, 46.177804987810156], [11.13560200531221, 46.176459169090805], [11.13534689830202, 46.17549640281599], [11.135189584231911, 46.17448680691692], [11.13485422671369, 46.17358404090153], [11.134326586035467, 46.172189847971055], [11.134273939656754, 46.17181281787019], [11.134183510386867, 46.17152649663243], [11.133765869775388, 46.17088177305798], [11.133407249406487, 46.170276447421244], [11.132952750795946, 46.16957840699783], [11.132599524240579, 46.16898647853007], [11.132412075117076, 46.16866146503582], [11.132359071703409, 46.16856795076942], [11.131585110485945, 46.168150375928036], [11.130924030359134, 46.167825194174696], [11.130845954837737, 46.167795149045254], [11.130493141923148, 46.16765322252355], [11.129751916286487, 46.167244025191], [11.128897257950818, 46.166706431765235], [11.127705132660479, 46.16594110786406], [11.126829768623583, 46.16532738168388], [11.126507030721482, 46.165103880077545], [11.12570294596542, 46.164322312571635], [11.125048310989035, 46.16369996594885], [11.124259749139014, 46.16302610344299], [11.124081839106646, 46.16286290468739], [11.124007482068242, 46.162634778475365], [11.124067871594185, 46.16252115176461], [11.124105376858166, 46.16244395175319], [11.124160216690074, 46.16237542981169], [11.124203038225376, 46.16228913060535], [11.124257068818821, 46.16222062365935], [11.124195323586296, 46.16166825113312], [11.12410625155473, 46.161057884360794], [11.12410233836089, 46.16072494477681], [11.123952932257257, 46.160376709054866], [11.12369057019202, 46.15995406972732], [11.122929362904202, 46.15912668363587], [11.122351043626493, 46.15838139873177], [11.122062244048166, 46.157923245271974], [11.122063482074338, 46.157639711664956], [11.122102035668227, 46.157463489073606], [11.122118631700264, 46.15730567494265], [11.121706073735831, 46.156912821289275], [11.121502436657243, 46.156606090700905], [11.121192048928181, 46.15611233440571], [11.121072361353212, 46.155862546744125], [11.121000156657278, 46.15571088129452], [11.120792034628526, 46.155287228226456], [11.120299642601104, 46.154522337010015], [11.120054810659182, 46.153901356216245], [11.11976635364289, 46.15336668743037], [11.119669427459115, 46.15322448005811], [11.119369673890747, 46.15296453025131], [11.11910803812271, 46.15241136156341], [11.118921963976668, 46.151928793772065], [11.118735247510243, 46.15151374014317], [11.118339684614291, 46.15076054511832], [11.118104222232867, 46.150319393063], [11.117800091797507, 46.149776009265615], [11.117436484928184, 46.14911672230195], [11.117349914137067, 46.14896982081254], [11.117136174276816, 46.148609267262465], [11.117203439407191, 46.14785199158942], [11.117259266694859, 46.14772495223353], [11.117407702101865, 46.14756469551691], [11.117409805785798, 46.147429651283616], [11.117289572869383, 46.14720687053432], [11.117159356175573, 46.146934772714395], [11.117121111537738, 46.146633969592195], [11.117032296800069, 46.14615409638589], [11.116896416842453, 46.145607592413405], [11.11697320431096, 46.14552066641104], [11.116951136401811, 46.1452195634545], [11.116938999110106, 46.14433325349855], [11.116924876587973, 46.14356398482219], [11.116801154458596, 46.142616739429585], [11.116786981721308, 46.14182497057532], [11.116993377801757, 46.141762644376506], [11.117466906665856, 46.14147035867876], [11.11752248988778, 46.14137932501772], [11.117567744043447, 46.14122998042497], [11.117719320370746, 46.14117316903494], [11.117864297482127, 46.14119748299075], [11.117875155119526, 46.14120628209798], [11.118117172077183, 46.14122879700766], [11.118407127030105, 46.14127742371172], [11.118828096068974, 46.14126061773623], [11.119538666740699, 46.14124743879798], [11.119492036798315, 46.14118341168462], [11.119465226791704, 46.14112703888862], [11.11944326767152, 46.141090967409475], [11.119315584577393, 46.140858663339465], [11.119264237134631, 46.140532289585174], [11.119256000558318, 46.140424805330554], [11.119263811401755, 46.140271301511945], [11.11934064194612, 46.14008045161232], [11.119567452957453, 46.13961413183118], [11.119624526113554, 46.13944028537453], [11.119626015167915, 46.13935490708885], [11.119611095134115, 46.13921611299751], [11.119559533325466, 46.13908657149618], [11.119498296353514, 46.138990548340296], [11.119381411459118, 46.13887269640008], [11.119133713641542, 46.13866677923309], [11.118995657707915, 46.13856144273468], [11.118943779663729, 46.13852171290529], [11.118857407545361, 46.138460922713115], [11.118527126315009, 46.13826760890871], [11.118228192159284, 46.138110028626286], [11.117864995893761, 46.13792922985967], [11.117483545080203, 46.13776365167576], [11.11710385371167, 46.13759923028289], [11.116711099825745, 46.13742909625318], [11.116318654441768, 46.137278311420324], [11.115908481033834, 46.137122785375134], [11.115548218882104, 46.13697347901901], [11.115176462649499, 46.13684028681352], [11.114800218961665, 46.13669446476304], [11.114430319361805, 46.136549646471344], [11.114061937927257, 46.13644786738955], [11.113534559950162, 46.13629698237748], [11.11009933779289, 46.13526878223229], [11.093229961899015, 46.12988427130543], [11.09298639713248, 46.129957675961194], [11.09275860365984, 46.12994510772554], [11.092357263948987, 46.129711698024806], [11.092050122523348, 46.12946684181517], [11.091650907774998, 46.12928937301186], [11.091412059665148, 46.12923092256951], [11.091038317056256, 46.129240643074056], [11.090551550715459, 46.129137135622415], [11.089722499677634, 46.12859508848433], [11.088939316638108, 46.1280126851094], [11.08853887398445, 46.12780561664247], [11.088090389174107, 46.12773155018793], [11.087556455862586, 46.1276433825299], [11.087238496532906, 46.127590958904435], [11.087166436442123, 46.12761603621888], [11.086716937057979, 46.127158657116944], [11.086288708347077, 46.12694209779055], [11.08624986434686, 46.12694181700423], [11.085688720261592, 46.12695206148693], [11.084356024131326, 46.12670681958621], [11.084364206509912, 46.12665608823504], [11.084347232694421, 46.12657674459153], [11.084332663239707, 46.126310689379984], [11.084224856791044, 46.126167119935545], [11.084073701972455, 46.125529860610435], [11.084002043209018, 46.12548319564731], [11.083855945189027, 46.125388206225345], [11.083781103455854, 46.125346369371435], [11.083762038695669, 46.125338526720284], [11.083626339049609, 46.12528240898472], [11.08347625730404, 46.12530827672624], [11.083224441420182, 46.125351479816075], [11.082810614001373, 46.12526209012813], [11.082521716785864, 46.125176632254764], [11.08230023888242, 46.12511811616529], [11.081916641478768, 46.12500990155531], [11.081788194081572, 46.12526380233492], [11.081324589396823, 46.12590677441749], [11.081302669013708, 46.12594767538339], [11.080541887926136, 46.12746459273289], [11.080058721623052, 46.12831942425619], [11.079471302108278, 46.12930665631471], [11.079122280763128, 46.13013654123917], [11.078687281646015, 46.13100399061797], [11.078203154630517, 46.13189933285517], [11.077745995880075, 46.13278068196002], [11.077436155660934, 46.13347034315832], [11.077132533190623, 46.134110388425604], [11.076801750073074, 46.134759926844175], [11.076409456081418, 46.135411842099224], [11.076272835035082, 46.13540055420983], [11.076015305075078, 46.13537922252882], [11.075991149838496, 46.13537723130624], [11.075851669149738, 46.1353656348549], [11.075838554837917, 46.135364613059245], [11.0758037502084, 46.135361735225956], [11.075379728444574, 46.13532659600303], [11.075372732930516, 46.135323032918876], [11.075271487071129, 46.13527122971089], [11.075159549036565, 46.13521395037433], [11.07473608887251, 46.13502831258708], [11.074004553383721, 46.13466114820473], [11.073810191690386, 46.13456360195808], [11.073198143960255, 46.13410975030299], [11.072960174620482, 46.13393334065673], [11.07192837787566, 46.13322959632913], [11.07053774792941, 46.13225297446998], [11.070236242865425, 46.13257417030805], [11.070123660583699, 46.13312109183211], [11.069768209549904, 46.133464234567604], [11.068647188290003, 46.13405794841112], [11.068597564044454, 46.13410870880234], [11.068290453815592, 46.13442253056158], [11.06867243286109, 46.13503601016213], [11.068680693416232, 46.13507681222743], [11.068664640067176, 46.13513902531793], [11.067782489811067, 46.13613522157681], [11.066407793373566, 46.13641480100697], [11.065990919286207, 46.13637697755288], [11.06567019726662, 46.1360576833476], [11.065248627814618, 46.1356380570365], [11.0650604037107, 46.13569393103841], [11.064875832064345, 46.13575054875591], [11.065202184269936, 46.13665674649712], [11.065477322601634, 46.137647212238754], [11.06567275373041, 46.1383098853494], [11.065697064902036, 46.138392159066704], [11.065790405926204, 46.138734554965595], [11.065821798934728, 46.13910300175372], [11.065836100129275, 46.13927068972038], [11.065837335257893, 46.1394421240491], [11.06567391034057, 46.139880875515495], [11.065672554377343, 46.13988270009874], [11.065327122225364, 46.14035363238315], [11.064863426890408, 46.14102416841292], [11.064198324139733, 46.14127415482096], [11.063693450776263, 46.14235737566431], [11.064765517413083, 46.14314929923861], [11.064574603443228, 46.144106875346566], [11.064494474066779, 46.14450910838966], [11.065668479699466, 46.1455322902736], [11.066350470901352, 46.14612667211225], [11.06666358247695, 46.146815115577525], [11.067086421141743, 46.14765673899842], [11.06767966651469, 46.149109640276635], [11.067700623607694, 46.14916092297509], [11.067887182183286, 46.14962286428073], [11.068145473647375, 46.15026072353491], [11.068032326198264, 46.15092870616522], [11.066547634245623, 46.15081552222877], [11.066288625523512, 46.150801485579876], [11.065670427827206, 46.15077828173188], [11.0650629920024, 46.1507554200623], [11.064322367359125, 46.15085898604443], [11.063113226347687, 46.15107397131047], [11.061843976317402, 46.15176011573698], [11.061835599532651, 46.15176467705337], [11.061767941486051, 46.15180126918142], [11.061578319607454, 46.1516860661436], [11.06157235816831, 46.15168248356429], [11.061384327037988, 46.15156824153943], [11.060059768371103, 46.151175954079214], [11.058812616440134, 46.15112939938142], [11.058065999039469, 46.15090947080124], [11.05756315670299, 46.15047202008545], [11.057334480119072, 46.150396124670294], [11.056635069688959, 46.15016372633094], [11.055683947011893, 46.15010937882945], [11.055263059443472, 46.15003243691188], [11.054634261728909, 46.149571047778515], [11.054543714570885, 46.1495146237524], [11.054407300114924, 46.14942968324119], [11.05390690218208, 46.14939781745791], [11.053571055168698, 46.149107292820474], [11.052263438230101, 46.147976403029666], [11.050264946140603, 46.14627818503596], [11.049633173796243, 46.14587526992343], [11.049521277741965, 46.145910084204], [11.049325736519286, 46.145970922685905], [11.048036951289944, 46.1471742235658], [11.047158602110455, 46.14821203584138], [11.046776786317173, 46.14867037836774], [11.048233484405992, 46.14955299629709], [11.04908757685134, 46.15007681921102], [11.049987914831878, 46.15036929211032], [11.050117394773254, 46.150497466649234], [11.050931931549679, 46.15098864662599], [11.051825541302968, 46.15124693446989], [11.052217866299658, 46.15151421954956], [11.05232895765445, 46.15166367763755], [11.051938112959608, 46.15179541414725], [11.0516934679277, 46.15304007500453], [11.052418743220361, 46.154004352511365], [11.052280063986741, 46.154292604288415], [11.052059002090239, 46.15487190643264], [11.052026960578814, 46.15507918222454], [11.051884574469529, 46.15593140146972], [11.0523263367218, 46.157342746598374], [11.053056796765574, 46.15814975796937], [11.053578283678863, 46.15849758802915], [11.053905640677627, 46.15915371671735], [11.054007877270887, 46.15987485358952], [11.05401694516138, 46.16096534379679], [11.053804168261088, 46.161656898968495], [11.053698653337085, 46.16232080746275], [11.054004814987971, 46.163231963755685], [11.054230569107023, 46.16384016410931], [11.054207156474241, 46.16636851361585], [11.05585737267304, 46.16694655268257], [11.056005867394767, 46.1686261640142], [11.057483436513506, 46.16908144413182], [11.06018669151032, 46.16976904708731], [11.06042110613725, 46.170166374687604], [11.06114486827143, 46.170806339786566], [11.061399153527171, 46.17110218734967], [11.062072330060099, 46.171864375726265], [11.063421653178526, 46.1713440063603], [11.064517515684914, 46.170752845280376], [11.064940810544988, 46.17079503022141], [11.065368169310437, 46.170615466880136], [11.066238308666897, 46.1705627838284], [11.06627623215324, 46.170339919424734], [11.066453778865686, 46.17002880200909], [11.066750548415, 46.16971624239027], [11.06726193075487, 46.1694233747676], [11.067815724647215, 46.16945462117093], [11.067819267153153, 46.169454821039], [11.067898583878106, 46.16945504353177], [11.067966225207636, 46.16945791897759], [11.067970414147801, 46.1694580970489], [11.068117965181706, 46.16945695081079], [11.068120774039974, 46.16945620754295], [11.068269593244201, 46.169416827526604], [11.068284480817166, 46.16934036841839], [11.068297213224442, 46.169305088458884], [11.068298581419349, 46.16922757103381], [11.068504958600975, 46.16908631059935], [11.068755589859062, 46.16898520000693], [11.06889000157545, 46.16892849460717], [11.069009220400284, 46.16887773433762], [11.069116183190538, 46.16886976733638], [11.069248073415535, 46.168859818779104], [11.069255116067772, 46.168885252124504], [11.069282978891223, 46.16888360640793], [11.069648688493455, 46.16885201237503], [11.069936920184974, 46.168837563794725], [11.070404544818356, 46.16843617121685], [11.071197796919382, 46.167750973010385], [11.071301848442754, 46.16774718186588], [11.071769817079288, 46.168068338227805], [11.072631471322492, 46.16906859941071], [11.073056719370808, 46.16969948813599], [11.073396000816702, 46.170346732235444], [11.073751582506024, 46.17070490107134], [11.074232503374114, 46.17105194385355], [11.074288576585401, 46.171093790032764], [11.07472155220376, 46.17125952885895], [11.074851122278671, 46.17131432829468], [11.074951479374837, 46.171348768900344], [11.074974856351986, 46.171360035847385], [11.075006207596562, 46.17140131645748], [11.075160344358636, 46.17169396563411], [11.075266803688127, 46.17180412875057], [11.075439087310553, 46.17196423275355], [11.075613279072149, 46.172139383719646], [11.07572000314323, 46.172242819198644], [11.075874128453467, 46.17230788741969], [11.076007477925236, 46.17236737954517], [11.076148671229504, 46.17237668692662], [11.076482707332826, 46.17239502597604], [11.07659077948742, 46.17262524227711], [11.076801066532589, 46.173089353003135], [11.077093147864574, 46.17372283735428], [11.077255398886084, 46.174071730234566], [11.07745914249589, 46.1745210752696], [11.07759204240478, 46.17467236602281], [11.077781285962454, 46.17490575702743], [11.078186844862763, 46.17518858416556], [11.078278527072326, 46.1752521286689], [11.078482581906684, 46.175436542651774], [11.078894088806479, 46.17573624900804], [11.07912581105419, 46.17585883909934], [11.07946432563499, 46.17604080472789], [11.079744652000667, 46.176225019099], [11.079790168410444, 46.176273008026016], [11.080073257329387, 46.176530397337544], [11.080429914299897, 46.176466157043656], [11.081220417311238, 46.17631662125102], [11.081517371542747, 46.17641896769168], [11.082136411073757, 46.17663093969327], [11.082483520262326, 46.17689240036252], [11.082720446496646, 46.177066206080916], [11.082827858183437, 46.17715092886415], [11.083213318611756, 46.17730011777015], [11.08394241795031, 46.177580203726976], [11.084033706619262, 46.17774618477792], [11.084193916970674, 46.178038872505375], [11.084241441930496, 46.178671116242334], [11.084350710041596, 46.17967966058132], [11.083716930317234, 46.17992559348954], [11.08351800336871, 46.18010167920415], [11.083374031921753, 46.18024718422188], [11.083602138692408, 46.18044115204014], [11.084017537437994, 46.180801254692575], [11.084601701070799, 46.181308777424086], [11.08407528559096, 46.181494598217796], [11.083811294881546, 46.181657533162586], [11.083758304882483, 46.181815667448184], [11.083794597779995, 46.18189280628471], [11.083838983034909, 46.18196329051034], [11.084056132341654, 46.181967403988786], [11.084260825914619, 46.181940809782134], [11.084450777446289, 46.18181446814749], [11.085030742250263, 46.18170291867198], [11.085226737120232, 46.181772687305376], [11.085976679823123, 46.18202189875224], [11.086275057853218, 46.182115514938666], [11.086528384183788, 46.18246808947277], [11.086809697064094, 46.18287158911036], [11.087050660689812, 46.18322343568733], [11.086893889889206, 46.18332059966159], [11.085676965995834, 46.183576171994], [11.08555061804716, 46.18371635720728], [11.085908229589977, 46.184007020083676], [11.08631726104114, 46.184351751795944], [11.086686808486682, 46.184658625486556], [11.086823472734615, 46.184981181577896], [11.086935147668855, 46.18524132672079], [11.086784422809686, 46.18555269994798], [11.086637699636187, 46.185861142284594], [11.086693837527529, 46.18631161760783], [11.086726952660523, 46.18657051846712], [11.086122303468086, 46.187416208455154], [11.086265113662874, 46.18770174226232], [11.086488393076369, 46.18814297442291], [11.08679372299177, 46.188758926215826], [11.087006303993556, 46.18918904136119], [11.08713078775544, 46.189399967086814], [11.087529252787421, 46.190089084692566], [11.088197274899446, 46.1901547485971], [11.08875192780297, 46.190214622529155], [11.088938284171666, 46.19046339882114], [11.089077166690068, 46.190653747607165], [11.089254193332025, 46.190900550742796], [11.089474149982102, 46.191031550379584], [11.089525520050756, 46.19149651296225], [11.089461992132788, 46.19168508720006], [11.089501116946286, 46.19212389893872], [11.089600390629494, 46.192327345024445], [11.089572815477919, 46.19274394980771], [11.08975327762728, 46.19280088945682], [11.089902523415677, 46.192881826322605], [11.08983379921509, 46.19306045580416], [11.0899832584745, 46.19318935733953], [11.09048475246363, 46.193255761804785], [11.090579416409456, 46.19326654477306], [11.090680609656594, 46.19333934648659], [11.090724311840384, 46.193432987542735], [11.090730035923395, 46.1934814522893], [11.090757703246034, 46.19373278776421], [11.090770721664917, 46.19390524101199], [11.090704934955905, 46.19428780490824], [11.090670305198966, 46.19450160446817], [11.090732774707414, 46.19457781289845], [11.090784617306712, 46.19464702024257], [11.090809624172886, 46.19479406995485], [11.09083913106694, 46.19502558414891], [11.09117102836973, 46.195273151382985], [11.091165720131574, 46.19537218630758], [11.09115697416544, 46.195517155291476], [11.091129865542818, 46.19558780725515], [11.091080076735356, 46.195743421072805], [11.09087707631011, 46.19578671076222], [11.090781911123884, 46.19580464182224], [11.090763860523008, 46.195840949454116], [11.090736273826169, 46.1958990179857], [11.090822356078073, 46.19611960324453], [11.090878598106258, 46.196270578555584], [11.09065687327498, 46.196571448643084], [11.090558305408388, 46.1967045693427], [11.090546829794313, 46.196777633452385], [11.090486387354726, 46.19703826653926], [11.090400439184144, 46.19746192917942], [11.090302767243315, 46.19791557277246], [11.090250471626852, 46.1981868100264], [11.090160239745671, 46.198614840087], [11.090126270356249, 46.1987700440615], [11.090156593611942, 46.19887749475292], [11.0901969906945, 46.19901986290171], [11.090300183194072, 46.19938249329913], [11.090378830437965, 46.19963621720746], [11.089675742449465, 46.19955939887699], [11.089654024054997, 46.19955627130434], [11.089173001863038, 46.19950630299306], [11.08901854463686, 46.19949197390903], [11.087358990698956, 46.199724176061466], [11.086099149966973, 46.19989534336775], [11.086070178405963, 46.19989945004251], [11.08540693176807, 46.19999293797728], [11.085106746155718, 46.20003637815341], [11.08428753859818, 46.20014939253739], [11.084126602162188, 46.20017051813033], [11.083858134572003, 46.20020942212217], [11.083040000022223, 46.200320826216235], [11.0828010643654, 46.20035760761644], [11.082719293020725, 46.2003638432949], [11.082271600406543, 46.20013312947979], [11.08204806151913, 46.20002255081018], [11.08202656628152, 46.19994035662352], [11.082021170944172, 46.199919725710835], [11.081998638210466, 46.199846893934094], [11.081993881570703, 46.199831519187114], [11.081990707352185, 46.199820638829685], [11.08193248680571, 46.199621073813375], [11.08189521018132, 46.19949712644041], [11.081856847761335, 46.199359938843806], [11.08178484127417, 46.19912137545247], [11.081732268067825, 46.19892801696508], [11.081672913173428, 46.19873668704949], [11.08162077229722, 46.19855475098797], [11.08155932825847, 46.19834440844532], [11.081441345057888, 46.19794458957811], [11.081326105209024, 46.19754472060024], [11.08126790842504, 46.197347654161746], [11.081187239467813, 46.19706998055336], [11.081127061518785, 46.19687488247112], [11.080994435849872, 46.19686468457063], [11.080340153856422, 46.19683433033408], [11.079825742563658, 46.1968109508671], [11.079053664262538, 46.19677499271794], [11.078195690304025, 46.19673463819162], [11.077415791123999, 46.19669583454246], [11.076878183179108, 46.1966710775049], [11.076528380748595, 46.19665230680019], [11.075846193513184, 46.1966207468279], [11.075819795337484, 46.19661972366534], [11.075459733238109, 46.196608251531615], [11.07490039179466, 46.19658911882944], [11.072777270799225, 46.196541207539816], [11.072359889160547, 46.19653234651857], [11.071300151951752, 46.19639482125534], [11.070929389744405, 46.19654823081564], [11.070860489315882, 46.19661282274351], [11.07068977820898, 46.196778799007916], [11.070541635868786, 46.1969248392108], [11.070382349194954, 46.1972009758503], [11.070101406464339, 46.197646838027715], [11.069083614380716, 46.198889028594806], [11.068699908470439, 46.199520359106195], [11.06845556182936, 46.19991770097611], [11.068420916503623, 46.200179599857826], [11.068245398983972, 46.20077202499423], [11.068255703953332, 46.20098128172495], [11.068275513625396, 46.201155249284355], [11.069151955572343, 46.201643371459994], [11.070207798957027, 46.20222549641536], [11.07024601326375, 46.20225081886786], [11.070264600169898, 46.20226383860772], [11.071390641604014, 46.202792492245415], [11.071794228086086, 46.20283917746189], [11.072359226113086, 46.20286943375926], [11.072858514940679, 46.20291887973708], [11.073334774742731, 46.203045243606205], [11.073885844144622, 46.20356625942227], [11.07330035677111, 46.204283404200034], [11.073008103531873, 46.20469821816166], [11.07248671134038, 46.20561220158159], [11.072019972238486, 46.206385687502085], [11.07169533444464, 46.20701709132324], [11.0714595158245, 46.20729587412587], [11.0712929564915, 46.20795141186066], [11.071155326953479, 46.20847142111676], [11.0709686067855, 46.2089788194462], [11.07071684130383, 46.20952339711525], [11.070439569836118, 46.2101224378182], [11.070168023209629, 46.21065837230846], [11.068811074145941, 46.211074457236414], [11.067410854114204, 46.21159031158319], [11.066949564861913, 46.21251218141167], [11.066550846874772, 46.213374416097906], [11.066197597731227, 46.21410982370345], [11.065802669795028, 46.21503048860098], [11.065429594175699, 46.2158427540977], [11.0652148944783, 46.21631914718104], [11.065024745214341, 46.21751511556795], [11.064897386126843, 46.218570445128826], [11.064775693554823, 46.21951766912009], [11.064708024133765, 46.22030641262376], [11.064553549697285, 46.22176274174293], [11.064420640680568, 46.222800169856], [11.06420091639442, 46.22450518302062], [11.062879999655308, 46.22623907888728], [11.065446089884432, 46.22870379223473], [11.067160528009538, 46.23049983878197], [11.068079824706324, 46.230762213110516], [11.068796149326632, 46.23092925217074], [11.06898755029004, 46.23097528829982], [11.069101698059143, 46.23103622319655], [11.069105755561901, 46.231144152421116], [11.06931406154583, 46.23112238024309], [11.069755311443892, 46.2310873887441], [11.07000036984929, 46.23105144933078], [11.070292915773218, 46.23102814944546], [11.070576248896515, 46.23101851607086], [11.0709125346697, 46.23099442208185], [11.071104323474103, 46.231179951116815], [11.07116663334651, 46.231241823411814], [11.071503138200361, 46.23165423458693], [11.071831813672372, 46.232030785802145], [11.071853889966237, 46.23205738623546], [11.072157349769586, 46.232366891977286], [11.072462963658309, 46.23260435605654], [11.072511469475586, 46.23264397738944], [11.072850673401284, 46.23291233202553], [11.073203084670624, 46.23322994726801], [11.07357085366633, 46.23352478219707], [11.073867776392884, 46.2337894008563], [11.074177832274975, 46.23405828048012], [11.074450459242838, 46.234323338578896], [11.074683697514596, 46.23457561077864], [11.074901678631578, 46.2348101590711], [11.075050429304826, 46.23501446325942], [11.075192870159999, 46.23522338193902], [11.075312215182928, 46.235378718540844], [11.075399158075806, 46.235534643467055], [11.075668084005041, 46.235615261336555], [11.075856096826769, 46.235656847381144], [11.076065680782392, 46.23571154157346], [11.076216168070347, 46.23578980960757], [11.076250420952922, 46.23592419058755], [11.076500660194775, 46.23654065901093], [11.076580957973405, 46.23669220365221], [11.076691623170598, 46.236789195092264], [11.076945219336089, 46.23680708699602], [11.077434590155018, 46.23684319308504], [11.077731422140493, 46.23684679694346], [11.078000697425054, 46.236936403184025], [11.078498576106623, 46.23711185339303], [11.078477226544798, 46.23731924670518], [11.080047657588922, 46.237646182040656], [11.080683273584015, 46.23736910473347], [11.081012189977423, 46.23719211153459], [11.08122427262589, 46.23709824715386], [11.081418953651724, 46.237058700634556], [11.081665901392212, 46.237072203038444], [11.081824249758508, 46.23710081928449], [11.081968527101434, 46.23714319198957], [11.08233425214035, 46.23725353137991], [11.082459687078194, 46.2372692461426], [11.082566530858994, 46.237307800111545], [11.082662714309402, 46.2373645486957], [11.08274704599293, 46.23740801287041], [11.08279479432306, 46.23747014417474], [11.082755474795194, 46.23758786362532], [11.082674540611775, 46.23771984179758], [11.08279485767237, 46.23777165033563], [11.082935468522946, 46.237845589405175], [11.08305220520952, 46.23788846271208], [11.08308307129588, 46.23793290117989], [11.083096884154028, 46.23795514994349], [11.083127404139265, 46.23807609655971], [11.083142831613044, 46.238183817954074], [11.08316533309701, 46.23834991180867], [11.083058898071814, 46.23853635638517], [11.082880309152301, 46.238872619545916], [11.08310371946649, 46.239035051418], [11.083397740564779, 46.23917819515577], [11.083655606918182, 46.23930849702487], [11.08409647515302, 46.23947596205871], [11.084472963608635, 46.23961309894616], [11.084689733347505, 46.23968564667342], [11.084950761764375, 46.23977088685365], [11.085183308044117, 46.239874646566776], [11.085358037113789, 46.239992961183894], [11.085498741368205, 46.240111896459155], [11.085637228501023, 46.24025787266928], [11.08578108863694, 46.240417250966715], [11.085884613146657, 46.24058186541862], [11.08600613190617, 46.24075065142422], [11.086061475846272, 46.24088464441172], [11.086438916147905, 46.24117476108035], [11.08681252182891, 46.241064436978476], [11.087092916595559, 46.240933313463245], [11.087263216482208, 46.240849201212235], [11.08748817951111, 46.24079559096829], [11.087785032580268, 46.24079916833684], [11.088013064759737, 46.24074100087904], [11.088230946866208, 46.2406290171441], [11.088443967257094, 46.24051712183748], [11.088679416026634, 46.24044081707444], [11.088937889371053, 46.240373091171875], [11.08907561594247, 46.24045607561123], [11.089134378292142, 46.240594504744024], [11.089192818959374, 46.2411514497393], [11.089176969267028, 46.24141724583203], [11.089312320387176, 46.241693778014906], [11.089535336259853, 46.24188770523931], [11.089879194308217, 46.24206142177356], [11.09023797912613, 46.24224386446737], [11.090500312563545, 46.24240556990591], [11.090692631474802, 46.242559555428286], [11.090873356056935, 46.242749753609274], [11.091052892328234, 46.24299397452208], [11.091203598624457, 46.24316222089932], [11.091418688860932, 46.243275287537365], [11.091664648273039, 46.243347287818956], [11.0919178550961, 46.24339665437423], [11.092298377510168, 46.243340187031215], [11.092790361177194, 46.24327267708727], [11.093307584688176, 46.24318670226954], [11.093856277673307, 46.24311814894725], [11.094427996399093, 46.24305817120441], [11.094865647207987, 46.24296914752928], [11.095164540370554, 46.242855665549236], [11.095518273559309, 46.242777178065865], [11.09577726275161, 46.24272292757208], [11.09612468417653, 46.24264905406469], [11.096533210104226, 46.24234905313602], [11.096859781334183, 46.242068554933766], [11.09719061837868, 46.24181497814524], [11.09742936107282, 46.2416125918676], [11.097701442283675, 46.24139159269426], [11.098023064160335, 46.24115168293415], [11.098325582948053, 46.240921123233775], [11.099135413478752, 46.240303236121505], [11.099773552784225, 46.24071002283541], [11.100197556623113, 46.24107124030976], [11.100543942333653, 46.24135288095076], [11.10124364637263, 46.241970032892354], [11.101853789758216, 46.24185531010736], [11.102515509834896, 46.24169013395994], [11.103162987038496, 46.241534216308615], [11.104062980191125, 46.241409646898916], [11.105574494213633, 46.2411433016553], [11.105618879660334, 46.241137983740764], [11.105816968548837, 46.24110283296151], [11.106283769421907, 46.24131023552738], [11.106794047501628, 46.241467333461806], [11.10724865183252, 46.241652456383925], [11.107643997443187, 46.24185667085557], [11.107987221349108, 46.24201234420273], [11.108291801594014, 46.24213272857363], [11.108702599466271, 46.24214764971636], [11.109240298198754, 46.24217372757674], [11.10974286806719, 46.24221395176542], [11.110108585303271, 46.24270221155726], [11.11039552904244, 46.243038921192934], [11.110870405525203, 46.24349816194981], [11.11122554706884, 46.24375260780626], [11.111612169050144, 46.24402447130567], [11.111991410660968, 46.244188467290144], [11.112327250075547, 46.24436226448179], [11.112544845195668, 46.24462374864157], [11.11286223086821, 46.24507689207223], [11.11378219710195, 46.24509138024506], [11.114033099925807, 46.24457372678999], [11.114223094203627, 46.244201203066886], [11.11463950124036, 46.2436444852868], [11.114969294389114, 46.24366088279819], [11.115337328879283, 46.24374407313957], [11.115654551113343, 46.24385520365165], [11.115940296152594, 46.24394891566973], [11.116169220436399, 46.24399867808124], [11.116374721902305, 46.24394537130733], [11.116519122374156, 46.243821194064566], [11.11670341794156, 46.243511773216554], [11.11679181327092, 46.24328063032094], [11.11691834042417, 46.24307128154165], [11.117111457126153, 46.242991202186666], [11.117350288210455, 46.242919275737655], [11.117724532459189, 46.24282683905415], [11.117629583825865, 46.243224608187816], [11.117523118479347, 46.243618090506075], [11.117469666404846, 46.24395658912126], [11.117432417295745, 46.244294787475305], [11.117475666765607, 46.244617993942896], [11.117589174626849, 46.24491289786258], [11.117862990608524, 46.2451598297963], [11.118128125660622, 46.24543392261558], [11.118353334846836, 46.24568175426302], [11.118570852132885, 46.245898227301815], [11.118718843925809, 46.24637249507269], [11.119725692602142, 46.246452826186065], [11.1200094654412, 46.24691557341173], [11.120706830083584, 46.2468756337347], [11.120893466079751, 46.24692167131582], [11.121005323198323, 46.24713110036805], [11.121106184923132, 46.24734973360278], [11.121353874050675, 46.24750713990395], [11.12157418559799, 46.247795557032575], [11.121735350431306, 46.2481480733675], [11.121795011403242, 46.24834947047487], [11.12189726649001, 46.24860407796015], [11.121761476236129, 46.24911961129459], [11.121801459818139, 46.24935737455486], [11.121893185635962, 46.2495491760204], [11.122094988872972, 46.24981993530122], [11.122259639860419, 46.250010382268854], [11.122427300793566, 46.25023677393738], [11.122597974283238, 46.250373107293626], [11.122772412205068, 46.25039686783548], [11.123015491545218, 46.25039235232362], [11.123333230685239, 46.25043145018346], [11.123859035059162, 46.25052518188153], [11.12429212332837, 46.250611634442386], [11.124365341149126, 46.25091178046216], [11.124397931547431, 46.251293683559965], [11.124436309926912, 46.25165747861126], [11.124508547263018, 46.25201614408007], [11.124535640064323, 46.25233964792451], [11.124586520987481, 46.25264920919152], [11.124704596376583, 46.25280901772954], [11.124906252061342, 46.253075274586735], [11.125218463015813, 46.253474478293995], [11.125577502992897, 46.253868310031486], [11.125724065289782, 46.25413559011995], [11.125955424069872, 46.25441479288489], [11.126070675777706, 46.254669154668505], [11.126282764597677, 46.254953215352565], [11.126408842027368, 46.255193875048604], [11.126537062969515, 46.255447995018905], [11.126577594433558, 46.255699246528614], [11.126662343652491, 46.25587767353681], [11.126760288281845, 46.25601985409661], [11.126988529546535, 46.25621811089181], [11.127254863423095, 46.2564786595913], [11.127380014768352, 46.256611332970174], [11.127667466524272, 46.256704983819134], [11.127928346259127, 46.25674062740234], [11.128274133957268, 46.25670718763659], [11.128620845591366, 46.25665572920617], [11.128881887056615, 46.2565698648491], [11.129141360609456, 46.25644352818635], [11.129424674397269, 46.25626274548394], [11.129781091503402, 46.25608509981122], [11.130169117403247, 46.25596986527405], [11.130514951289149, 46.2558959169249], [11.130661299655536, 46.25607319257859], [11.130804355192389, 46.25629103035861], [11.130816013262907, 46.25646631697725], [11.130822460796821, 46.25663270053937], [11.130825492709484, 46.25679464768124], [11.130838196582124, 46.256996915396115], [11.130926081125946, 46.257130279750996], [11.131059520622282, 46.25726729465663], [11.131238062735864, 46.25747997001486], [11.131386160907807, 46.257702213089146], [11.131514759567208, 46.25796532057535], [11.13165283878324, 46.25813824904705], [11.131785636836481, 46.258216773739875], [11.132004335128908, 46.25829369576915], [11.132242534359667, 46.25832975262429], [11.132527110360078, 46.25843244509903], [11.132790915529927, 46.25858502561802], [11.133023876938397, 46.258778682129986], [11.13310735761439, 46.258840125365815], [11.133267836445377, 46.25896313266217], [11.133507982226048, 46.25917465447319], [11.133712298141418, 46.25938234465126], [11.133907538971366, 46.25964870522001], [11.134059294557558, 46.259839375784594], [11.134137940621269, 46.2600269113486], [11.134146593304965, 46.26016625286879], [11.134116945114656, 46.260405311832564], [11.134035131310478, 46.26059584381127], [11.133902896897704, 46.26078281710977], [11.133708271731361, 46.26103395653609], [11.133524990685636, 46.2612848838348], [11.1333788129604, 46.261530618097595], [11.133285970824945, 46.26189685923066], [11.133244049266313, 46.26215414739815], [11.133234021326926, 46.26239733990696], [11.133250326822901, 46.26260854016445], [11.133380218519727, 46.2628626210526], [11.133583300305988, 46.26320533734463], [11.133779202899285, 46.26353018691507], [11.133962315113058, 46.2638597750677], [11.134128571801199, 46.26413117641337], [11.134335103414475, 46.26431182339285], [11.134574628287236, 46.26446485316397], [11.13485161064171, 46.2646216828069], [11.135108220984618, 46.264796892773525], [11.135309147932675, 46.26500014317745], [11.135342070235936, 46.26517953190957], [11.135401752728727, 46.265421421960454], [11.135410661784693, 46.265650760418104], [11.13536892264934, 46.26591254591813], [11.135297607302308, 46.266206384683706], [11.135181037089065, 46.266533542506714], [11.13518940296861, 46.2667997936753], [11.135473403457192, 46.26712885160757], [11.13575479979434, 46.26786540452962], [11.135866792579112, 46.26856743245247], [11.135465637585767, 46.269731256711985], [11.135707443331802, 46.2705388655295], [11.135989898535412, 46.270774147197635], [11.135907961151487, 46.27093253587479], [11.135839702973556, 46.271406760680506], [11.136157849977272, 46.27238775199692], [11.13611960936611, 46.2728941712549], [11.136507170226897, 46.27297830051984], [11.136778971432753, 46.2734581075284], [11.136255546988114, 46.2752066949729], [11.136241896690638, 46.27549035680083], [11.13681608703125, 46.27608798499098], [11.137828609022407, 46.27664676593783], [11.137790997195797, 46.27713686273332], [11.137680123589554, 46.27768903717693], [11.137661156113332, 46.27768601166669], [11.137604109408201, 46.277857958415886], [11.137556905316542, 46.27804197692308], [11.137353059554492, 46.27823794051216], [11.13716304601785, 46.278663325224365], [11.136912077481071, 46.27906847736605], [11.136886299134405, 46.27926631233404], [11.137092419922707, 46.27936584132149], [11.137204679260343, 46.27953272032543], [11.137224593041566, 46.279764852128245], [11.137426322611603, 46.27991933939093], [11.137471031063043, 46.28005963314474], [11.137589130751167, 46.280125562777165], [11.137621732459605, 46.28031190838148], [11.137549565217785, 46.28055254792732], [11.137549308332558, 46.28070113146274], [11.137208932842674, 46.28100841433982], [11.137224789597354, 46.28101624066814], [11.13736580806775, 46.28116743976398], [11.137480471178879, 46.28154728957552], [11.137963952186999, 46.28183271989227], [11.138398837818825, 46.28253781296668], [11.138894333617413, 46.282788284494316], [11.139054892156414, 46.28362698266148], [11.139135914047502, 46.283790799274264], [11.139885918173652, 46.28285190111997], [11.1404933121413, 46.28278202548497], [11.141174546573653, 46.282733263637674], [11.141810991936273, 46.28261783583662], [11.14240479377761, 46.28244920286122], [11.14288325085816, 46.2823187302926], [11.143446498884458, 46.28207416330559], [11.144059704828829, 46.28194565854025], [11.14467505586398, 46.281830610479986], [11.145286376819262, 46.28177863596365], [11.145850499443918, 46.281723044308755], [11.146229602591609, 46.281666424537946], [11.146465532739132, 46.281558491593614], [11.146807836833476, 46.28118305484545], [11.147035209136636, 46.280980779629445], [11.14728813102328, 46.280809524517764], [11.14753304123295, 46.28072392105704], [11.148079783828997, 46.28055614274906], [11.14847153458526, 46.28044977681481], [11.149213127878726, 46.28016132752761], [11.149702759572282, 46.279985615246765], [11.15000917358384, 46.2798133478528], [11.150224085448048, 46.27962480090011], [11.150705973722769, 46.279044221949405], [11.150809934509054, 46.278632757318], [11.150915756180929, 46.27835176014807], [11.151009838580787, 46.278102484450315], [11.151066065023805, 46.2778809217436], [11.151123523373407, 46.277690836445785], [11.15123540125455, 46.27748172638836], [11.151390488272309, 46.27729880335173], [11.151574192619064, 46.27714234187752], [11.151755570264692, 46.276967923546486], [11.15199845166692, 46.27670684506741], [11.152467256274692, 46.276104004269754], [11.152762153366838, 46.27567994111115], [11.153146318924613, 46.275339697083005], [11.15341741712134, 46.27513658507918], [11.153630255446354, 46.27502007208662], [11.153865084904558, 46.2748851438633], [11.154286104059821, 46.27461620284757], [11.154483853233197, 46.27444597156992], [11.154700702938857, 46.27426637963494], [11.154893972860624, 46.2740647314333], [11.154952330219546, 46.27385662674674], [11.154962580765034, 46.273662929437315], [11.154994435550698, 46.27348232501914], [11.155036017167754, 46.27330153716745], [11.15508210934005, 46.27311166404959], [11.155160061319096, 46.272948190657836], [11.155354559604255, 46.272778019121475], [11.155552265244578, 46.27264828760086], [11.15574972635357, 46.27259506195197], [11.155920929384328, 46.27249283027949], [11.156039692589488, 46.2724185882525], [11.156087864325743, 46.27232317745184], [11.156058993748937, 46.27216621881628], [11.155991426061226, 46.27197398945823], [11.155965235546212, 46.271843980800384], [11.155898340709566, 46.27171023992114], [11.155852167953206, 46.27156710790467], [11.155818401602582, 46.271450742404184], [11.155853990390902, 46.27132406838475], [11.155962176031956, 46.27118702462438], [11.156167131362459, 46.27099415389168], [11.156353867155838, 46.270833127233786], [11.156502048568036, 46.27068182798485], [11.156619494555075, 46.27045010682573], [11.156870011542935, 46.27026087509223], [11.157103786615599, 46.27005845852207], [11.15742833966279, 46.26985432777964], [11.157698159583093, 46.269619728831294], [11.157878097230816, 46.26916182158902], [11.158204037591661, 46.26878665871024], [11.15869342555333, 46.26852541011914], [11.159240926512174, 46.268339563098], [11.159867146521139, 46.26813422531953], [11.160433177522854, 46.26796602273929], [11.161059152213564, 46.26783718470752], [11.161398678530926, 46.267808263260775], [11.161664580215984, 46.26722722133044], [11.161909463926081, 46.266813080000496], [11.162303826027037, 46.266405110134514], [11.162817630130698, 46.26594087723942], [11.1633191942525, 46.26545437318716], [11.16387547884273, 46.26499833147947], [11.164332769305569, 46.26470616552155], [11.16479281974049, 46.264319443305716], [11.165128941284092, 46.26395756770902], [11.165671750483014, 46.263407270261176], [11.165856333331487, 46.26315176644959], [11.165975168266073, 46.26291550889084], [11.166174486727197, 46.26270472627734], [11.166362963360653, 46.26254815006331], [11.166535204648271, 46.262432382250495], [11.166717347439404, 46.26232092654785], [11.166846153633715, 46.262255482901836], [11.167004691779754, 46.26216247471027], [11.167166597384048, 46.26215490439383], [11.167381281135063, 46.26212833248006], [11.1679904601693, 46.26223378010346], [11.168593867626532, 46.2623573344134], [11.16890018081279, 46.26239202227298], [11.169280544308922, 46.262371302460515], [11.169843364627488, 46.26261712337929], [11.170041438661608, 46.262662863484096], [11.170508608407864, 46.26200597663692], [11.171042843248298, 46.26119931030096], [11.171567115982883, 46.26046933242579], [11.172832200431413, 46.26013927877742], [11.173380101891215, 46.259966856216344], [11.173866473029175, 46.2596740986428], [11.174357879018906, 46.25938574328526], [11.174659396478171, 46.2592585031263], [11.17492899190953, 46.259226371929394], [11.175576039681971, 46.259182554885115], [11.175984622937984, 46.25921977683375], [11.17625615632565, 46.259277607722915], [11.176829625856476, 46.25946469014161], [11.177396260066383, 46.25964289970499], [11.17796631749717, 46.25982554132226], [11.17829242682156, 46.2598688269128], [11.178649526121077, 46.259875519925345], [11.179457922984707, 46.25985110529527], [11.17991874998893, 46.25985581605591], [11.18048636793597, 46.25985398728298], [11.181083694324059, 46.25986508900552], [11.181772511121247, 46.25985194040082], [11.182410413266313, 46.259700256680254], [11.183008342893265, 46.259522332329354], [11.183602669921331, 46.25933547345986], [11.184120844997098, 46.25906906501888], [11.184623356596093, 46.25877595299776], [11.184967711588772, 46.258625366103026], [11.185304663874009, 46.258533421124184], [11.185706717044953, 46.25842672971004], [11.186003852861047, 46.25839404559562], [11.18663990375009, 46.258318875416045], [11.187276310540291, 46.25825269509396], [11.187917219868812, 46.25817742480122], [11.189208398896557, 46.25801769387929], [11.1898253593106, 46.25791137088709], [11.190158332167051, 46.25784198835357], [11.19046304453688, 46.25771464512781], [11.190733192169978, 46.257533962394994], [11.190984224350283, 46.25748414867512], [11.191670661642213, 46.25749348646617], [11.192333147637232, 46.257512279840014], [11.192980689276112, 46.25752235593397], [11.193547729863905, 46.25758797490844], [11.194190752264543, 46.25772863406253], [11.19460849901197, 46.25791411704133], [11.194899117564994, 46.25808403984172], [11.195221428550983, 46.258275853653984], [11.19567536763498, 46.258555139773414], [11.19617036764103, 46.2588471353155], [11.19668338411564, 46.25910278151936], [11.197234220845361, 46.25937119857277], [11.197944792393395, 46.25965904138386], [11.198114810169525, 46.25932726330432], [11.198316523420667, 46.258936373822976], [11.19849535951735, 46.25862242593603], [11.198702284656786, 46.25832143764903], [11.198870671099646, 46.25803019073922], [11.19900249565742, 46.25783864954085], [11.199112529294815, 46.257670028072965], [11.203874962508163, 46.25604554656722], [11.20354468876846, 46.25588204162696], [11.203490176148938, 46.25586093945977], [11.202905371380737, 46.255698092538005], [11.202482849595135, 46.25551780401839], [11.202429842926731, 46.25549518606792], [11.201870422247147, 46.255267851014274], [11.201775516290251, 46.25521289761203], [11.201090536057333, 46.25493521802496], [11.20062822563356, 46.25471594305573], [11.20033881905309, 46.25448177457752], [11.199624966073245, 46.25415408508581], [11.198996826866255, 46.25376992611679], [11.199212988536242, 46.25360337001612], [11.19930737705763, 46.253530452456744], [11.199291287950246, 46.25351096204502], [11.19920373715334, 46.2534239968054], [11.199071735203944, 46.25331880701006], [11.198956938624535, 46.25324154609869], [11.198852275115318, 46.253180830161604], [11.198768660410055, 46.2531176390015], [11.19869890831497, 46.253018270721306], [11.19865956514329, 46.2528260664917], [11.198607339236068, 46.252532319421825], [11.198512222796353, 46.252356668586756], [11.198154600691122, 46.25201758760299], [11.197886522996825, 46.25172124313704], [11.197523816497931, 46.25149169879741], [11.19750173638616, 46.25147484338858], [11.197379521431426, 46.25138710349453], [11.197266250368537, 46.25126994122861], [11.197051174502887, 46.25091020489609], [11.196982676455624, 46.25082908156544], [11.196601105289782, 46.25045832598144], [11.196394248566213, 46.25012228052883], [11.196462490150726, 46.249605984540814], [11.196568211829172, 46.24908941784042], [11.196624759934473, 46.2488032983207], [11.196758408612567, 46.2485507065013], [11.196790520325045, 46.2484712483973], [11.196815152714363, 46.24839616414035], [11.196935907989015, 46.24802546908015], [11.197007781604988, 46.24786973565309], [11.197139036047991, 46.24765471969328], [11.197259866996982, 46.24746456425318], [11.197439145478677, 46.247113893159884], [11.197554829763419, 46.24681448548762], [11.197534157163917, 46.246683482063375], [11.197397214646983, 46.24632755322055], [11.197315380792856, 46.24616577587922], [11.197228128151282, 46.24600797272855], [11.197163085469983, 46.245928763014675], [11.197030984512853, 46.24581079257382], [11.19701685313119, 46.2457980142305], [11.19699497700351, 46.24578295480524], [11.196835662940398, 46.245668927500226], [11.196512801942411, 46.24552321429489], [11.196062943923938, 46.245361221346315], [11.1958629011887, 46.24529918602926], [11.195471887608601, 46.24520086022051], [11.195177651218291, 46.245138383837926], [11.194687977906836, 46.245031690983474], [11.194098674995582, 46.24497685995402], [11.193790243496522, 46.24499421329173], [11.193370111148283, 46.245105601203655], [11.193172080943626, 46.245162233526464], [11.192638747762592, 46.2453333923491], [11.191702643967375, 46.24567751779227], [11.191513311962877, 46.24575720054441], [11.191489599275375, 46.2457708855327], [11.19097116259248, 46.24606550178129], [11.190883220591463, 46.24613738889769], [11.190731848893146, 46.24626476274432], [11.19025582074356, 46.246849174795855], [11.190229218560848, 46.246884785091105], [11.189877981553861, 46.24726736138285], [11.189780345043717, 46.247465958250636], [11.189778045671764, 46.24747063525455], [11.189745495683022, 46.247536842996844], [11.189714405400174, 46.247743180533945], [11.189675192486064, 46.24806154461496], [11.189662414965548, 46.248369411910566], [11.189670888987921, 46.24839651970179], [11.189679413157984, 46.24842488654083], [11.189777492564648, 46.24874097907217], [11.18998137344237, 46.24910769379048], [11.190028290193286, 46.24925637555001], [11.190078298790976, 46.24949895872938], [11.19008467252776, 46.24968828797765], [11.19003826412872, 46.24993019945593], [11.190015005966504, 46.24998140569496], [11.189957929167678, 46.25009554068137], [11.189909699311317, 46.25019402593541], [11.189721267219445, 46.25076257201423], [11.189592824354554, 46.251163106764096], [11.189546903511848, 46.25143362884771], [11.189502014337386, 46.251777237111924], [11.18945104279692, 46.25190308674388], [11.189339745191972, 46.2521098388849], [11.189237464699, 46.25221843720066], [11.188999176147208, 46.25238984729353], [11.188511210203636, 46.252716011008175], [11.188123707363, 46.25273724524333], [11.18738601312645, 46.25272826626991], [11.18685284331514, 46.252752462722285], [11.186788136412813, 46.25275219960943], [11.186718046984865, 46.25274794573117], [11.186115775213894, 46.25267166580845], [11.185933947252687, 46.25259037603339], [11.185370480394294, 46.25232543064536], [11.18480862088849, 46.252013394773606], [11.184526570163193, 46.251847930166456], [11.184132340775676, 46.251603524146574], [11.183778040685693, 46.25135747256808], [11.183573582749853, 46.251120066535535], [11.18351989458953, 46.25100870759866], [11.182925285555767, 46.25026280528864], [11.182596920690955, 46.24965428109095], [11.182475356627691, 46.24942899534786], [11.182277867582652, 46.248911574102266], [11.182174567385967, 46.24862302513305], [11.18194858457475, 46.247711044523925], [11.181924529499712, 46.24757560291721], [11.181879129499565, 46.24723896741529], [11.181840253798093, 46.24683461675224], [11.181791755960848, 46.24629562876033], [11.181778321676328, 46.246146664155404], [11.181664394136748, 46.2448477895176], [11.18165919371359, 46.24481152853634], [11.181649584762402, 46.24468049097785], [11.181622990574207, 46.244445016926946], [11.181588644408311, 46.244220130980416], [11.181457145574766, 46.243946339777885], [11.181356614791934, 46.24377302793906], [11.181207955703846, 46.24358326478352], [11.18103439654304, 46.243403426887596], [11.180434528082694, 46.2428326628026], [11.179887822625517, 46.2423494418686], [11.178710458418069, 46.24122106180706], [11.178090667953532, 46.24054887435321], [11.177359011046725, 46.23967496386779], [11.177310620578032, 46.23962557570735], [11.177274741039568, 46.239584229158794], [11.176748515572703, 46.23892213116031], [11.176486996706602, 46.23860739743683], [11.17640605684215, 46.238636063264536], [11.17534552432743, 46.23901167563423], [11.17359486829786, 46.23963504273319], [11.173364544849687, 46.239435752953874], [11.173047566992732, 46.23955373167236], [11.172819256953977, 46.2393091811571], [11.172503858424463, 46.238827990560196], [11.172336979085358, 46.23849715423956], [11.17222546911414, 46.238233836684465], [11.172205733608441, 46.23791490921677], [11.172303683147296, 46.23769123697619], [11.172551312706466, 46.237039960152444], [11.172885845721067, 46.23640619016275], [11.173010963190617, 46.2363491534345], [11.173136065393592, 46.23600529230377], [11.173489192348672, 46.23546946949604], [11.173687371775337, 46.23523770444654], [11.17398263552866, 46.23489964805706], [11.174421565932544, 46.234669168756604], [11.174593808082014, 46.234690835861976], [11.174704427156437, 46.23476264375127], [11.174858227303341, 46.234825900681585], [11.174957237718674, 46.23480754005835], [11.175118262230157, 46.234734215319605], [11.175193979250672, 46.23475959263858], [11.175253454769106, 46.234764760239905], [11.175265155749935, 46.23464411630973], [11.175300639486489, 46.23386214326096], [11.175321691669545, 46.23366968056342], [11.175313335813362, 46.233530518340025], [11.17525524179361, 46.23344873373221], [11.175090149811867, 46.233301305694326], [11.174923738381132, 46.2331959329399], [11.174823481779852, 46.233133490978304], [11.174765656541853, 46.23307816125611], [11.174475644215992, 46.232756078629485], [11.175572888841007, 46.232227280430884], [11.176111078309729, 46.23144828682323], [11.174807757576277, 46.23051034806914], [11.172710281554165, 46.22907405899226], [11.172845077562924, 46.228855309936925], [11.172913561565004, 46.228740604546864], [11.172933384754726, 46.2287072866809], [11.17299304768175, 46.22860894942983], [11.173016868423785, 46.22857177542573], [11.173055653623209, 46.22851289624631], [11.173095581921496, 46.228450125206294], [11.17312876282743, 46.22839711273763], [11.173141041444064, 46.22837635865762], [11.173637447982623, 46.22759424558234], [11.176873614415383, 46.224743733449756], [11.176973571064973, 46.22248060376062], [11.17613961619841, 46.22235797421036], [11.176646954485951, 46.22171370305114], [11.17515083068496, 46.221411341729514], [11.174569334992704, 46.22131440579922], [11.174534153121018, 46.22107368823602], [11.174519533148647, 46.22107450646167], [11.174520361637246, 46.22105937025341], [11.174596427690668, 46.22087431736019], [11.174648769594846, 46.22074191742673], [11.174724701036254, 46.220474694586834], [11.174758928740495, 46.22037612031113], [11.17478554257387, 46.22029488143427], [11.17481171307691, 46.22022868142957], [11.174889061312603, 46.220128656415795], [11.174945076210005, 46.22005972829345], [11.175002644552563, 46.22000715105737], [11.175241211707108, 46.219819724618254], [11.175348744212714, 46.21964028237477], [11.17537248964625, 46.219365052223075], [11.175516512191045, 46.21923297656588], [11.17555911585158, 46.21919220428043], [11.175557739634616, 46.21897361401026], [11.175562381654599, 46.21891061376932], [11.175708245623115, 46.2186414287348], [11.175756994166191, 46.218546177759436], [11.175810872414322, 46.21842652836641], [11.17584012838047, 46.218382680020454], [11.175950751305127, 46.21826843025968], [11.176125943760088, 46.21815745090921], [11.176240659758026, 46.21806805365768], [11.176387341874717, 46.21780002216657], [11.176381305512876, 46.21776539609799], [11.176344506152164, 46.217664303928295], [11.17705986035331, 46.217041989233], [11.177016661341439, 46.21685119650014], [11.177030466063352, 46.216806832178165], [11.177145507436146, 46.216260124058564], [11.177102275863646, 46.21592748768584], [11.176954890685424, 46.21511496099253], [11.176730434411986, 46.21457634042899], [11.176086415822365, 46.21410610184835], [11.175796475340842, 46.2141017226061], [11.175147038742432, 46.2145147786759], [11.174868314244423, 46.213566684949], [11.17407877031386, 46.21398087145478], [11.172978237783042, 46.21395617319945], [11.172614373258774, 46.21382628775414], [11.171996147986421, 46.2135914328634], [11.171067809272136, 46.21281066024246], [11.171194538958396, 46.21267747840543], [11.171400230936653, 46.21200196948399], [11.1719179434107, 46.211779454064875], [11.172614997685917, 46.21178492369231], [11.172945116045382, 46.21175974610482], [11.172923683991218, 46.21136441151818], [11.173255740650088, 46.210925183444324], [11.173288189161148, 46.21088343504851], [11.17386339180534, 46.21036397820298], [11.174316334034256, 46.2102747180629], [11.174921383704957, 46.21039658774366], [11.17538130421695, 46.21057593886073], [11.17559741674431, 46.21023134394612], [11.176179080531512, 46.20917398561651], [11.176491901240635, 46.208531077893646], [11.176556791475285, 46.20797848467446], [11.177675752944392, 46.20806570904144], [11.178631915082512, 46.20815998658323], [11.178706999147336, 46.20779683367851], [11.178709552192386, 46.20741238292499], [11.178991733645397, 46.20691675692854], [11.179243763633437, 46.20665850269899], [11.179335611046119, 46.20659140895878], [11.179356513845, 46.20657255971589], [11.179524629465302, 46.20666385624502], [11.179544334375572, 46.20667698082501], [11.179601415061203, 46.20674357420066], [11.179623078695135, 46.20675378131471], [11.179715573009638, 46.20677838782568], [11.17997119946405, 46.206889525044346], [11.180269667158337, 46.20702513507809], [11.180274475802934, 46.20703854375409], [11.180278345370658, 46.2071363030071], [11.180284110616, 46.207150953470034], [11.18050823202745, 46.20725864047769], [11.18068766398737, 46.207282037216174], [11.180782940429852, 46.20730811981705], [11.180786613726939, 46.20730921975025], [11.180804871180243, 46.20730869130531], [11.180814885104324, 46.20729967991264], [11.180847550968965, 46.207270165571316], [11.180998157133706, 46.20726945118767], [11.18101215413092, 46.2072693640382], [11.181130151825739, 46.20718502920589], [11.181164063854863, 46.207160801154245], [11.18118122931397, 46.20715561334534], [11.181302263174853, 46.20715123290332], [11.181311971219955, 46.2071508675783], [11.181401099788193, 46.20709372439208], [11.18156109539631, 46.20711407069001], [11.181692825504028, 46.20712118599542], [11.181812340119192, 46.20712754435699], [11.181981249586116, 46.20720928181108], [11.182127908584878, 46.20707948723007], [11.182299498275155, 46.207052179588715], [11.182447995167012, 46.207070224165825], [11.182460409174015, 46.20706944704987], [11.18256322148789, 46.20697163032455], [11.182666342996933, 46.20698631115659], [11.18277855065797, 46.20702745917314], [11.182792706862381, 46.207024848684405], [11.182810045651829, 46.207017497250895], [11.18286887056709, 46.206974252205434], [11.182887952648016, 46.20697163760885], [11.182911821619847, 46.206965511455515], [11.183118171992868, 46.206968139500496], [11.183136745399425, 46.206969044691625], [11.183147944870008, 46.2068002553874], [11.183166122487293, 46.2068009881297], [11.183253098063785, 46.206866198497366], [11.183460936985473, 46.206948900016876], [11.183628694331338, 46.20703749726988], [11.18391843733193, 46.20672721038075], [11.183966521427694, 46.2066615793295], [11.184054820907095, 46.20638745302298], [11.184063151350442, 46.20637523341717], [11.183970972833112, 46.20629977277479], [11.183806039461066, 46.20616792067566], [11.183643222188365, 46.20603710793307], [11.183550296141913, 46.20596238125692], [11.183301714600642, 46.20576741476436], [11.18315378503169, 46.20565566747852], [11.18304782251033, 46.20560522022342], [11.182850052490748, 46.20551476535905], [11.182762684640585, 46.20547557299665], [11.182681726855192, 46.20544111828147], [11.182531906629404, 46.20537647780126], [11.182361829563527, 46.2053305845873], [11.18230138550443, 46.20530347806471], [11.18220570954717, 46.20525742372035], [11.182080009057252, 46.20519952225481], [11.181971948961255, 46.205135343616995], [11.181855993893079, 46.205064925420814], [11.181713395703587, 46.20497638520934], [11.18163330978999, 46.20492463255247], [11.181545378730428, 46.20486789945072], [11.181520195886517, 46.20485064965824], [11.181165921339074, 46.204591994160076], [11.181039513091086, 46.20450305406493], [11.181026384799198, 46.20449232429296], [11.180651209306568, 46.20426223702325], [11.180717265136625, 46.2042445058948], [11.180788222162569, 46.20421633085876], [11.180878023635549, 46.20416646544033], [11.18094679725768, 46.20411916136322], [11.181005098111568, 46.204069267045924], [11.181274098969352, 46.20400446047922], [11.181349852407592, 46.203982853746446], [11.181314154095439, 46.203919363155975], [11.181391186709655, 46.203910422383245], [11.181374924999753, 46.20381073961468], [11.181432395622252, 46.20370388909766], [11.181292137596976, 46.20359577304674], [11.181217092790568, 46.203566514584786], [11.181159368254033, 46.2033461193379], [11.181072875362522, 46.20324705705221], [11.181025829960689, 46.20314904183395], [11.18097780549226, 46.2030524853284], [11.180901059379567, 46.20296385721878], [11.180816045725482, 46.20293487875818], [11.180442967774646, 46.20269854053581], [11.180242338296653, 46.202414269453364], [11.180240918941553, 46.20238495558881], [11.180113435526026, 46.20228505460875], [11.17999341307283, 46.202196801526725], [11.179942028157232, 46.202156110486676], [11.17983214611609, 46.20206532370505], [11.179696300663924, 46.20197026192574], [11.179658976579487, 46.20193767281241], [11.179627303151964, 46.20191685628607], [11.179593992199818, 46.20189724102349], [11.179502574560654, 46.20184066284026], [11.179428696005264, 46.20180148063558], [11.179396529554516, 46.20178130346878], [11.179333373225386, 46.201724816147575], [11.179242462404021, 46.20163843712637], [11.179223953080868, 46.20160305896718], [11.179196141117034, 46.201548777598234], [11.17915705783859, 46.201501191382924], [11.17896979283528, 46.201331686884], [11.178867413162818, 46.2011748739715], [11.178609176142192, 46.20106909417375], [11.178507968550248, 46.20101072177938], [11.17822459888683, 46.2007741060033], [11.176890143787617, 46.200179506067464], [11.176670690502883, 46.19992330814999], [11.176530391142302, 46.19974147460533], [11.176298480460261, 46.199553735465656], [11.176262612381498, 46.19954478827583], [11.176000214777927, 46.19938913018207], [11.175840877822765, 46.199296221198786], [11.17579167774736, 46.19926475698083], [11.175664889196492, 46.19916564780466], [11.17501712055661, 46.19861519116612], [11.17444262633096, 46.198640255341445], [11.174338832656007, 46.19867049134588], [11.17433492396598, 46.198653555166956], [11.174333994256674, 46.19864970273153], [11.17428389119257, 46.19842780882525], [11.174158153616217, 46.19823696497386], [11.174042891700397, 46.198061942195714], [11.174006210531362, 46.1980159285532], [11.173923299785544, 46.19790527227365], [11.17367252553362, 46.197620053441575], [11.17358292569389, 46.19751393429897], [11.173365933971084, 46.197142029344], [11.173358810909818, 46.197129024392744], [11.173386674011631, 46.19711580401826], [11.173302552222578, 46.19702038081089], [11.173115069712724, 46.19690037240821], [11.173081537006302, 46.19688156948256], [11.172900769164857, 46.19674118236435], [11.172861474534226, 46.19670772845654], [11.172811051017991, 46.19666485583619], [11.172813123204639, 46.196664816433234], [11.172786364358567, 46.196646874646895], [11.172742336233213, 46.19662485107793], [11.172603191399194, 46.19654406402333], [11.17250795662238, 46.19632779753317], [11.17263083779627, 46.19631826095142], [11.172719824842243, 46.19628731799078], [11.17278724364363, 46.19626488534857], [11.172825935981614, 46.1962370587162], [11.172835894252795, 46.19622993912789], [11.17284563803233, 46.19621409332955], [11.172864915559744, 46.19612777392244], [11.172861974391173, 46.19611234933999], [11.172856231940552, 46.19609490795711], [11.17282762954323, 46.19601705925772], [11.172803568018821, 46.19589988290987], [11.172767391197953, 46.19570553436829], [11.17276439888399, 46.1956789503861], [11.172749124000493, 46.1954394729055], [11.172747164538425, 46.19539963884477], [11.172749075853728, 46.19537251160534], [11.172785681769708, 46.195242481269624], [11.17278748793998, 46.195235696701815], [11.17280332416999, 46.19512127179487], [11.172820555000271, 46.19499620001288], [11.172797586029125, 46.194995466733175], [11.172814597527287, 46.19492071078896], [11.172136085024428, 46.19454506795418], [11.17209302023775, 46.19452770597373], [11.171445292126295, 46.194485744946476], [11.171417913062193, 46.19448491518951], [11.171393519739569, 46.19448420869455], [11.171367377111794, 46.194485155489446], [11.171358884414127, 46.1944176346255], [11.1713894191618, 46.194278179773], [11.17146166982569, 46.19415476273202], [11.171478080936291, 46.19413051006804], [11.171588172241911, 46.19407279604031], [11.171791542995576, 46.19396488743578], [11.171819934370681, 46.19393221674799], [11.171830429176477, 46.19389925618696], [11.171833036161585, 46.19387661588482], [11.171827778508813, 46.1938615953117], [11.171685802066948, 46.19373504941113], [11.171601315982267, 46.19360057053553], [11.171493546748083, 46.193378421124855], [11.171492247111429, 46.19317440901308], [11.17141307232951, 46.1929608964168], [11.17132440154819, 46.19282541680901], [11.17126600179339, 46.19274444377502], [11.17120872417948, 46.19268208001172], [11.170940199581349, 46.19241573290798], [11.170667514370887, 46.19211607308356], [11.170564429736178, 46.19197978664877], [11.170475574036349, 46.1918000283934], [11.170421273084617, 46.19160917337394], [11.170416277168744, 46.19127499702922], [11.170354025817424, 46.1910632322668], [11.170290955320734, 46.190978747311156], [11.17023293746132, 46.19089092624742], [11.170197636728481, 46.19083039463667], [11.170110073968713, 46.190601109844124], [11.170000921429864, 46.19044225734944], [11.169939239247807, 46.19036008590739], [11.169908618790565, 46.190299915339246], [11.169870553522708, 46.190123062188185], [11.169822719657871, 46.18994495446933], [11.169773879954112, 46.18992563119916], [11.169345449362734, 46.19000144804162], [11.1691710537525, 46.19003859995059], [11.16901637384109, 46.19006961715426], [11.168898816418771, 46.190065908379964], [11.168631884948999, 46.190037043566996], [11.168396804701457, 46.18992180080603], [11.168284007821716, 46.18984473858347], [11.168204716716042, 46.189769470564016], [11.16815051048346, 46.189696066597264], [11.168020540746994, 46.189386672083174], [11.167982264596004, 46.18928335478108], [11.16795546676717, 46.18920169043033], [11.167926043648148, 46.18913564640885], [11.167895108364366, 46.189077281327926], [11.16784116300424, 46.18899730203603], [11.167772263878547, 46.18890284590946], [11.16768715058604, 46.18881787764388], [11.16768236579951, 46.18880824808359], [11.167638893096234, 46.188721049783794], [11.167584558044384, 46.18862784731048], [11.167510651488858, 46.18852466571122], [11.167490139599375, 46.18850066396963], [11.167475986217761, 46.18848347184768], [11.167437413085375, 46.18842858163094], [11.167419351287702, 46.18838779283304], [11.16739977344195, 46.18828538069158], [11.16745006297035, 46.188166972792494], [11.167526117077276, 46.188038085800436], [11.167457352488034, 46.187917346025294], [11.167338288631392, 46.1877005369598], [11.167315966754833, 46.187656858843084], [11.167228091722913, 46.18742577761719], [11.167203374818373, 46.187248670343145], [11.167191562857276, 46.187152591080704], [11.167194867351109, 46.18708502609927], [11.167360188707086, 46.186896394110065], [11.167373438443168, 46.18677526865443], [11.167364898199416, 46.18675580997192], [11.16732040577029, 46.18664261997775], [11.167252697390015, 46.18657505184679], [11.167166275560954, 46.18647975763343], [11.167102395255823, 46.18642714732429], [11.167012569721068, 46.18636062853065], [11.166892080975721, 46.18630522151635], [11.16671103433537, 46.18621577148358], [11.166520753551573, 46.186128746323604], [11.166447603845501, 46.18606776110409], [11.166401803773915, 46.186023447900425], [11.166335628636668, 46.185951980019894], [11.166296101331602, 46.18592878857886], [11.16619663224518, 46.18587073225117], [11.166119160898498, 46.1858547402902], [11.165930110004858, 46.185765980770675], [11.165858023816396, 46.18572873582901], [11.16579770937422, 46.185697567936764], [11.165711062877124, 46.18564592837371], [11.165643129906769, 46.18560554447379], [11.165626788244202, 46.18558209337424], [11.165613202383762, 46.18556282018861], [11.165564044497094, 46.18549237938516], [11.16552911382209, 46.18549052129915], [11.165488294441515, 46.1854542136129], [11.165373466598902, 46.18532840529342], [11.164773226765687, 46.18473405760041], [11.164245179248901, 46.184222312284156], [11.16409238799633, 46.18407004076036], [11.164014713177627, 46.18399266910335], [11.163559587097103, 46.183757109143556], [11.163299115605165, 46.18362784609777], [11.162594661356213, 46.183434533649184], [11.162428672385433, 46.183457655734784], [11.161783424897969, 46.1835547378907], [11.16150783125366, 46.18364113430231], [11.160824774467768, 46.183861330470606], [11.160732133870738, 46.18392104431534], [11.160446220561745, 46.18408476595446], [11.159721943160951, 46.184516522104246], [11.159601317531152, 46.18458963440175], [11.159465944426058, 46.18466338529052], [11.15907274958698, 46.18486729218967], [11.158838949739945, 46.18498880364919], [11.15869341382478, 46.18507111590786], [11.158555925026324, 46.185104134281275], [11.158520381209504, 46.185126316457236], [11.158237789338928, 46.18524280815972], [11.158078381025023, 46.18525877936845], [11.157936390688485, 46.18521654943358], [11.157801485975218, 46.185153394805724], [11.15771236201828, 46.18517397843524], [11.157631963236797, 46.18525856943779], [11.15761181162503, 46.185362813501634], [11.157518750696532, 46.185570857721956], [11.157341464372099, 46.18570947963819], [11.157246293810239, 46.18575438788635], [11.157144968257644, 46.18580427241596], [11.157046720975245, 46.18589640018799], [11.157052275610825, 46.18599529870618], [11.157132151216423, 46.18613239555619], [11.15714521533393, 46.18619470106018], [11.157154777462535, 46.18630990447242], [11.15721182393132, 46.18640999096042], [11.15719914913538, 46.186509953637156], [11.157168035562577, 46.18655266244883], [11.157103656668172, 46.18660958969634], [11.15704135299352, 46.186666567735365], [11.156741295763844, 46.18681056631071], [11.156483640887235, 46.18690894234457], [11.156400529133737, 46.18696073253726], [11.156342676187261, 46.18703544680183], [11.156323006253075, 46.187046078342746], [11.15604181201491, 46.18718872906709], [11.15594652649309, 46.18730699086944], [11.155787043411987, 46.18740405301927], [11.155392383049104, 46.187548122735635], [11.155261322763026, 46.187603246608205], [11.154990624088871, 46.187659833858966], [11.154917816105224, 46.18765040657927], [11.154757633865696, 46.18765009717882], [11.154590211778828, 46.18765685430101], [11.154483662626069, 46.18766939367662], [11.15422290864843, 46.18770194079352], [11.154006510128017, 46.18770890035681], [11.153886462972496, 46.18768443236318], [11.15370011978924, 46.18764141314232], [11.153539075070531, 46.18760907720696], [11.153298133448788, 46.1875516957926], [11.153045479223746, 46.18752288551949], [11.15271262974641, 46.18748460517242], [11.152280521914223, 46.18740994219624], [11.152080561784661, 46.187383197356176], [11.151891647244602, 46.187357234187544], [11.151697753163978, 46.18730661363857], [11.151689603444165, 46.18731693744456], [11.151536344737277, 46.18746409813102], [11.151167762341176, 46.18778523817773], [11.150530084633948, 46.18814456287624], [11.150132902714915, 46.18836732446212], [11.149004281659163, 46.18882858057841], [11.14878662977211, 46.18890656618472], [11.148602691236096, 46.18882218205121], [11.1481594695429, 46.188515684700704], [11.148157481556224, 46.18851455203456], [11.148145522091527, 46.18850694660298], [11.148015683169117, 46.18842406455727], [11.147923004113714, 46.18837027485891], [11.147865272217572, 46.188335358847155], [11.14772785164464, 46.18835387237503], [11.147499845811305, 46.1883290866965], [11.147282451254739, 46.18829393081816], [11.147144716053417, 46.18823783705874], [11.147085303285422, 46.188163080902065], [11.147009307340863, 46.18805848529258], [11.146820022641128, 46.187859984906545], [11.146782136392225, 46.187838825963595], [11.14667601654103, 46.187779257565985], [11.146324657514059, 46.18769153465888], [11.14629091916764, 46.1876837080909], [11.146103258177057, 46.187623330648805], [11.146031406765081, 46.18760181933373], [11.145852922236559, 46.187507518005695], [11.145682980720888, 46.18736967452325], [11.14547990357296, 46.1871728709444], [11.145328988811274, 46.18704105989749], [11.145293430690218, 46.186963154843234], [11.145263153840702, 46.18689109082581], [11.145160099616948, 46.186800322413426], [11.14509333122354, 46.18675621432548], [11.144902297390196, 46.18671560888784], [11.144810723102607, 46.18667340631094], [11.144729354309602, 46.18660707130348], [11.14465688526493, 46.18658962085173], [11.14450297771348, 46.18652734721751], [11.144472935849057, 46.18654123147197], [11.14076909493254, 46.18788134466887], [11.140128132979136, 46.18812447882064], [11.140098376330904, 46.18813574648897], [11.140063712555698, 46.188150796175826], [11.140031910476836, 46.18813942137923], [11.140027989008283, 46.188147134268], [11.140001721663188, 46.18808822926771], [11.139970439614972, 46.18801807852411], [11.139865144829946, 46.18769918970804], [11.139858794202361, 46.187682478073235], [11.139780363410097, 46.18749476048487], [11.139735511797312, 46.18740091720794], [11.139669276075727, 46.187130078402284], [11.139614167878108, 46.186812049616066], [11.139593951797849, 46.18675527623844], [11.139520156879518, 46.18653336049991], [11.139388361229393, 46.186246018417876], [11.139188253085262, 46.1860882993807], [11.139113125172004, 46.18598557232145], [11.138921175297874, 46.185807629412054], [11.138441197649804, 46.18552365305801], [11.138367690032853, 46.185352402872795], [11.138381050478243, 46.18527592027932], [11.138454647243424, 46.185138818398784], [11.138529609575627, 46.184933378605145], [11.138485528123892, 46.18478578857861], [11.138133149568068, 46.18434353756382], [11.137977254618862, 46.18405592468885], [11.138006098858659, 46.18391426021657], [11.138001500080073, 46.18353507339653], [11.137985588382378, 46.183529070899105], [11.137938329390002, 46.18350997444902], [11.137922497035015, 46.18348596984708], [11.13788551788656, 46.183481351537736], [11.13769450502839, 46.18348429513019], [11.137572399763364, 46.183476499058685], [11.13746625075958, 46.183455803953365], [11.137307820300403, 46.1833733545319], [11.137246827555748, 46.18336765512458], [11.137147928635965, 46.18338021525137], [11.137019052315361, 46.18341835678332], [11.13692052278601, 46.183440450128266], [11.136828618265776, 46.183436228609494], [11.136783064944256, 46.18341097957726], [11.136669876689227, 46.18315181726333], [11.13666219764498, 46.183067357978324], [11.136631733879321, 46.18293328303554], [11.136632384538256, 46.18275281472993], [11.136361502919156, 46.181821487598725], [11.136313369814742, 46.18126481842565], [11.136251451056927, 46.1807469282379]]]], &quot;type&quot;: &quot;MultiPolygon&quot;}, &quot;id&quot;: &quot;10&quot;, &quot;properties&quot;: {&quot;classid&quot;: &quot;AMB002_13&quot;, &quot;datafine&quot;: null, &quot;dataini&quot;: &quot;2020/01/01 00:00:00.000&quot;, &quot;fkfonte&quot;: &quot;05&quot;, &quot;fkscala&quot;: &quot;03&quot;, &quot;fkscala_d&quot;: &quot;10000&quot;, &quot;fktfonte_d&quot;: &quot;altre fonti&quot;, &quot;fktipoel_d&quot;: &quot;manuale&quot;, &quot;fktipoelab&quot;: &quot;01&quot;, &quot;nome&quot;: &quot;COMUNIT\u00c0 ROTALIANA-K\u00d6NIGSBERG&quot;, &quot;objectid&quot;: 191, &quot;sede&quot;: &quot;MEZZOCORONA&quot;, &quot;struttura&quot;: &quot;S133&quot;, &quot;struttura_&quot;: &quot;Servizio Catasto&quot;}, &quot;type&quot;: &quot;Feature&quot;}], &quot;type&quot;: &quot;FeatureCollection&quot;});



    geo_json_ee33d01b0f11ca4280d41c125209143b.bindTooltip(
    function(layer){
    let div = L.DomUtil.create(&#x27;div&#x27;);

    let handleObject = feature=&gt;typeof(feature)==&#x27;object&#x27; ? JSON.stringify(feature) : feature;
    let fields = [&quot;objectid&quot;, &quot;classid&quot;, &quot;sede&quot;, &quot;nome&quot;, &quot;struttura&quot;, &quot;struttura_&quot;, &quot;dataini&quot;, &quot;datafine&quot;, &quot;fkfonte&quot;, &quot;fktfonte_d&quot;, &quot;fktipoelab&quot;, &quot;fktipoel_d&quot;, &quot;fkscala&quot;, &quot;fkscala_d&quot;];
    let aliases = [&quot;objectid&quot;, &quot;classid&quot;, &quot;sede&quot;, &quot;nome&quot;, &quot;struttura&quot;, &quot;struttura_&quot;, &quot;dataini&quot;, &quot;datafine&quot;, &quot;fkfonte&quot;, &quot;fktfonte_d&quot;, &quot;fktipoelab&quot;, &quot;fktipoel_d&quot;, &quot;fkscala&quot;, &quot;fkscala_d&quot;];
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



### find a layer with the rivers for the area


identify the bounding box


```python
smallest_community.to_crs(epsg=4326).bounds
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
      <th>10</th>
      <td>11.019439</td>
      <td>46.12501</td>
      <td>11.203875</td>
      <td>46.283791</td>
    </tr>
  </tbody>
</table>
</div>




```python
bbox = list(smallest_community.to_crs(epsg=4326).bounds.values[0])
```


```python
bbox
```




    [11.01943938362622, 46.12500990155531, 11.203874962508163, 46.283790799274264]



in this case we can use the WFS of [the national cartographic portal of the Italian Ministry of the Environment](http://www.pcn.minambiente.it/)


```python
csw = CatalogueServiceWeb("http://www.pcn.minambiente.it/geoportal/csw")
```

![](https://raw.githubusercontent.com/napo/geospatial_course_unitn/master/images/geocatalog_pcn_rivers.png)


[ ... ] 


```python
from owslib.wfs import WebFeatureService
```


```python
wfs_url="http://wms.pcn.minambiente.it/ogc?map=/ms_ogc/wfs/Aste_fluviali.map"
```


```python
wfs = WebFeatureService(url=wfs_url,version="1.1.0")
```


```python
wfs.identification.title
```




    'Reticolo idrografico'




```python
wfs.contents
```




    {'ID.RETICOLO.FIUMI_PRINCIPALI_SECONDARI': <owslib.feature.wfs110.ContentMetadata at 0x7f242015b8b0>,
     'ID.RETICOLO.FIUMI_TORRENTI': <owslib.feature.wfs110.ContentMetadata at 0x7f242015b5b0>,
     'ID.RETICOLO.CORSI_ACQUA': <owslib.feature.wfs110.ContentMetadata at 0x7f242025eef0>,
     'ID.RETICOLO.ELEMENTI_IDRICI': <owslib.feature.wfs110.ContentMetadata at 0x7f242025d690>}




```python
layer = list(wfs.contents)[2]
```


```python
response = wfs.getfeature(typename=layer, bbox=bbox,srsname='urn:ogc:def:crs:EPSG::4326')
```


```python
rivers = gpd.read_file(response)
```


```python
rivers.head(3)
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
      <th>id_fiume</th>
      <th>id_tratta</th>
      <th>tipo</th>
      <th>nome</th>
      <th>foglio_igm</th>
      <th>sottotipo</th>
      <th>ordine</th>
      <th>bacino_pri</th>
      <th>bacino</th>
      <th>da</th>
      <th>tipo_da</th>
      <th>a</th>
      <th>tipo_a</th>
      <th>annotazion</th>
      <th>enabled</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ID.RETICOLO.CORSI_ACQUA.9878</td>
      <td>2241</td>
      <td>11503</td>
      <td></td>
      <td></td>
      <td>BOLZANO</td>
      <td>1</td>
      <td>3</td>
      <td>ADIGE</td>
      <td></td>
      <td>SORGENTE</td>
      <td>1</td>
      <td>NOCE</td>
      <td>3</td>
      <td></td>
      <td>1</td>
      <td>LINESTRING (46.17313 11.05670, 46.17069 11.060...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ID.RETICOLO.CORSI_ACQUA.16150</td>
      <td>688</td>
      <td>14645</td>
      <td></td>
      <td></td>
      <td>BOLZANO</td>
      <td>1</td>
      <td>2</td>
      <td>ADIGE</td>
      <td></td>
      <td>SORGENTE</td>
      <td>1</td>
      <td>ADIGE</td>
      <td>3</td>
      <td></td>
      <td>1</td>
      <td>LINESTRING (46.15510 11.03917, 46.15664 11.041...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ID.RETICOLO.CORSI_ACQUA.20273</td>
      <td>885</td>
      <td>21041</td>
      <td>TORRENTE</td>
      <td>AVISIO</td>
      <td>BOLZANO</td>
      <td>1</td>
      <td>2</td>
      <td>ADIGE</td>
      <td></td>
      <td>19357</td>
      <td>3</td>
      <td>21042</td>
      <td>3</td>
      <td></td>
      <td>1</td>
      <td>LINESTRING (46.17089 11.23801, 46.17053 11.237...</td>
    </tr>
  </tbody>
</table>
</div>




```python
rivers.plot()
plt.show()
```


    
![png](03_solution_exercise_retrieving_data_from_sdi_files/03_solution_exercise_retrieving_data_from_sdi_87_0.png)
    



```python
geometry_smallest_community_4326 = smallest_community.geometry.to_crs(epsg=4326).values[0]
```


```python
rivers.within(geometry_smallest_community_4326).unique()
```




    array([False])




```python
ax = smallest_community.to_crs(4326).plot(edgecolor='k',figsize=(15, 10))
rivers.plot(ax=ax,color="red")
plt.show()

```


    
![png](03_solution_exercise_retrieving_data_from_sdi_files/03_solution_exercise_retrieving_data_from_sdi_90_0.png)
    


there the problem of the inverted axes<br/>
It can ben solved with this function


```python
import shapely
```


```python
def swapxy(geometry):
  geometry = shapely.ops.transform(lambda x, y: (y, x),geometry)
  return geometry
```


```python
rivers['geometry'] = rivers['geometry'].apply(lambda geometry: swapxy(geometry))
```


```python
rivers.plot()
plt.show()
```


    
![png](03_solution_exercise_retrieving_data_from_sdi_files/03_solution_exercise_retrieving_data_from_sdi_95_0.png)
    


we are ready to plot on a map


```python
y = rivers.unary_union.centroid.y
x = rivers.unary_union.centroid.x
```


```python
y
```




    46.20213279893027




```python
x
```




    11.110945030023714




```python
ax = smallest_community.to_crs(4326).plot(edgecolor='k',figsize=(15, 10))
rivers.plot(ax=ax,color="red")
plt.show()
```


    
![png](03_solution_exercise_retrieving_data_from_sdi_files/03_solution_exercise_retrieving_data_from_sdi_100_0.png)
    


**CLIP**

we need to have all the rivers inside the area and not the bounding box




```python
gpd.clip(rivers, smallest_community)
```

    /tmp/ipykernel_37029/3880772484.py:1: UserWarning: CRS mismatch between the CRS of left geometries and the CRS of right geometries.
    Use `to_crs()` to reproject one of the input geometries to match the CRS of the other.
    
    Left CRS: EPSG:4326
    Right CRS: EPSG:25832
    
      gpd.clip(rivers, smallest_community)





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
      <th>id_fiume</th>
      <th>id_tratta</th>
      <th>tipo</th>
      <th>nome</th>
      <th>foglio_igm</th>
      <th>sottotipo</th>
      <th>ordine</th>
      <th>bacino_pri</th>
      <th>bacino</th>
      <th>da</th>
      <th>tipo_da</th>
      <th>a</th>
      <th>tipo_a</th>
      <th>annotazion</th>
      <th>enabled</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>



the geodatamews have to use the same projection


```python
rivers_rotaliana = gpd.clip(rivers.to_crs(epsg=4326), smallest_community.to_crs(epsg=4326))
```


```python
rivers.plot()
plt.show()
```


    
![png](03_solution_exercise_retrieving_data_from_sdi_files/03_solution_exercise_retrieving_data_from_sdi_105_0.png)
    



```python
rivers_rotaliana.plot()
plt.show()
```


    
![png](03_solution_exercise_retrieving_data_from_sdi_files/03_solution_exercise_retrieving_data_from_sdi_106_0.png)
    


show the rivers on the map


```python
rivers.explore()
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
                #map_815a5d3aa65ad81550cff928eee0550c {
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

            &lt;div class=&quot;folium-map&quot; id=&quot;map_815a5d3aa65ad81550cff928eee0550c&quot; &gt;&lt;/div&gt;

&lt;/body&gt;
&lt;script&gt;    

            var map_815a5d3aa65ad81550cff928eee0550c = L.map(
                &quot;map_815a5d3aa65ad81550cff928eee0550c&quot;,
                {
                    center: [46.189735999999996, 11.108574],
                    crs: L.CRS.EPSG3857,
                    zoom: 10,
                    zoomControl: true,
                    preferCanvas: false,
                }
            );
            L.control.scale().addTo(map_815a5d3aa65ad81550cff928eee0550c);





            var tile_layer_eb505ec1f6480d967fb13434ba4bdd82 = L.tileLayer(
                &quot;https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png&quot;,
                {&quot;attribution&quot;: &quot;Data by \u0026copy; \u003ca href=\&quot;http://openstreetmap.org\&quot;\u003eOpenStreetMap\u003c/a\u003e, under \u003ca href=\&quot;http://www.openstreetmap.org/copyright\&quot;\u003eODbL\u003c/a\u003e.&quot;, &quot;detectRetina&quot;: false, &quot;maxNativeZoom&quot;: 18, &quot;maxZoom&quot;: 18, &quot;minZoom&quot;: 0, &quot;noWrap&quot;: false, &quot;opacity&quot;: 1, &quot;subdomains&quot;: &quot;abc&quot;, &quot;tms&quot;: false}
            ).addTo(map_815a5d3aa65ad81550cff928eee0550c);


            map_815a5d3aa65ad81550cff928eee0550c.fitBounds(
                [[46.085624, 10.97914], [46.293848, 11.238008]],
                {}
            );


        function geo_json_74838407a5aaf4a7f5a8faa7218a4301_styler(feature) {
            switch(feature.id) {
                default:
                    return {&quot;fillOpacity&quot;: 0.5, &quot;weight&quot;: 2};
            }
        }
        function geo_json_74838407a5aaf4a7f5a8faa7218a4301_highlighter(feature) {
            switch(feature.id) {
                default:
                    return {&quot;fillOpacity&quot;: 0.75};
            }
        }
        function geo_json_74838407a5aaf4a7f5a8faa7218a4301_pointToLayer(feature, latlng) {
            var opts = {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#3388ff&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#3388ff&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 2, &quot;stroke&quot;: true, &quot;weight&quot;: 3};

            let style = geo_json_74838407a5aaf4a7f5a8faa7218a4301_styler(feature)
            Object.assign(opts, style)

            return new L.CircleMarker(latlng, opts)
        }

        function geo_json_74838407a5aaf4a7f5a8faa7218a4301_onEachFeature(feature, layer) {
            layer.on({
                mouseout: function(e) {
                    if(typeof e.target.setStyle === &quot;function&quot;){
                        geo_json_74838407a5aaf4a7f5a8faa7218a4301.resetStyle(e.target);
                    }
                },
                mouseover: function(e) {
                    if(typeof e.target.setStyle === &quot;function&quot;){
                        const highlightStyle = geo_json_74838407a5aaf4a7f5a8faa7218a4301_highlighter(e.target.feature)
                        e.target.setStyle(highlightStyle);
                    }
                },
            });
        };
        var geo_json_74838407a5aaf4a7f5a8faa7218a4301 = L.geoJson(null, {
                onEachFeature: geo_json_74838407a5aaf4a7f5a8faa7218a4301_onEachFeature,

                style: geo_json_74838407a5aaf4a7f5a8faa7218a4301_styler,
                pointToLayer: geo_json_74838407a5aaf4a7f5a8faa7218a4301_pointToLayer
        });

        function geo_json_74838407a5aaf4a7f5a8faa7218a4301_add (data) {
            geo_json_74838407a5aaf4a7f5a8faa7218a4301
                .addData(data)
                .addTo(map_815a5d3aa65ad81550cff928eee0550c);
        }
            geo_json_74838407a5aaf4a7f5a8faa7218a4301_add({&quot;bbox&quot;: [10.97914, 46.085624, 11.238008, 46.293848], &quot;features&quot;: [{&quot;bbox&quot;: [11.056696, 46.159871, 11.076413, 46.17313], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.056696, 46.17313], [11.060478, 46.170694], [11.062012, 46.168417], [11.06403, 46.165905], [11.065564, 46.163627], [11.067459, 46.162351], [11.070321, 46.160949], [11.072569, 46.160467], [11.076413, 46.159871]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;0&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;NOCE&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;SORGENTE&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.9878&quot;, &quot;id_fiume&quot;: 2241, &quot;id_tratta&quot;: 11503, &quot;nome&quot;: &quot;&quot;, &quot;ordine&quot;: 3, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.039174, 46.145985, 11.073126, 46.158335], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.039174, 46.155101], [11.041187, 46.15664], [11.043821, 46.158168], [11.047065, 46.158335], [11.05544, 46.157177], [11.059924, 46.155854], [11.064442, 46.155097], [11.067, 46.154259], [11.068723, 46.152887], [11.069278, 46.150402], [11.069202, 46.148378], [11.069281, 46.146342], [11.073126, 46.145985]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;1&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;ADIGE&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;SORGENTE&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.16150&quot;, &quot;id_fiume&quot;: 688, &quot;id_tratta&quot;: 14645, &quot;nome&quot;: &quot;&quot;, &quot;ordine&quot;: 2, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.203469, 46.15572, 11.238008, 46.170894], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.238008, 46.170894], [11.237177, 46.170532], [11.236035, 46.170168], [11.234404, 46.1702], [11.232737, 46.169647], [11.231955, 46.167934], [11.231141, 46.166384], [11.229698, 46.16562], [11.228062, 46.165202], [11.224608, 46.163397], [11.223165, 46.161985], [11.22236, 46.160633], [11.220549, 46.160056], [11.217547, 46.158521], [11.213924, 46.157026], [11.212034, 46.15609], [11.209287, 46.15572], [11.207721, 46.155751], [11.206642, 46.156951], [11.205545, 46.158025], [11.203469, 46.158245]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;2&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;21042&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;19357&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.20273&quot;, &quot;id_fiume&quot;: 885, &quot;id_tratta&quot;: 21041, &quot;nome&quot;: &quot;AVISIO&quot;, &quot;ordine&quot;: 2, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;TORRENTE&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.108842, 46.136844, 11.143672, 46.151094], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.143672, 46.150494], [11.138782, 46.151094], [11.136384, 46.151022], [11.13405, 46.150615], [11.129132, 46.150023], [11.124105, 46.148622], [11.121307, 46.147234], [11.120339, 46.145983], [11.119937, 46.143948], [11.11794, 46.140817], [11.115528, 46.13871], [11.113724, 46.137915], [11.110503, 46.137308], [11.108842, 46.136844]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;3&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;19318&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;21149&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.20508&quot;, &quot;id_fiume&quot;: 885, &quot;id_tratta&quot;: 19317, &quot;nome&quot;: &quot;AVISIO&quot;, &quot;ordine&quot;: 2, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;TORRENTE&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.108842, 46.134109, 11.111222, 46.136844], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.111222, 46.134109], [11.108842, 46.136844]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;4&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;AVISIO&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;20547&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.20509&quot;, &quot;id_fiume&quot;: 3100, &quot;id_tratta&quot;: 19318, &quot;nome&quot;: &quot;&quot;, &quot;ordine&quot;: 3, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.189289, 46.127545, 11.195886, 46.155034], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.19075, 46.127545], [11.190981, 46.129117], [11.192019, 46.131122], [11.194079, 46.133782], [11.193388, 46.136946], [11.193504, 46.139869], [11.194453, 46.143226], [11.195886, 46.147023], [11.194517, 46.149075], [11.189289, 46.155034]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;5&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;AVISIO&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;SORGENTE&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.21000&quot;, &quot;id_fiume&quot;: 3193, &quot;id_tratta&quot;: 19969, &quot;nome&quot;: &quot;&quot;, &quot;ordine&quot;: 3, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.189289, 46.155034, 11.203469, 46.158245], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.203469, 46.158245], [11.201407, 46.157529], [11.199513, 46.157124], [11.197632, 46.156728], [11.195367, 46.156772], [11.193877, 46.156413], [11.192223, 46.155554], [11.190395, 46.155166], [11.189289, 46.155034]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;6&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;19969&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;21042&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.21001&quot;, &quot;id_fiume&quot;: 885, &quot;id_tratta&quot;: 19970, &quot;nome&quot;: &quot;AVISIO&quot;, &quot;ordine&quot;: 2, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;TORRENTE&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.073126, 46.127751, 11.078526, 46.145985], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.073126, 46.145985], [11.073233, 46.143661], [11.073166, 46.14188], [11.07322, 46.139179], [11.073132, 46.136831], [11.0767, 46.131528], [11.077586, 46.129262], [11.078526, 46.127751]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;7&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;AVISIO&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;14645&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.21289&quot;, &quot;id_fiume&quot;: 215, &quot;id_tratta&quot;: 20397, &quot;nome&quot;: &quot;ADIGE&quot;, &quot;ordine&quot;: 1, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;FIUME&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.073126, 46.145985, 11.076413, 46.159871], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.076413, 46.159871], [11.076047, 46.159473], [11.075258, 46.158609], [11.074223, 46.152787], [11.073492, 46.15055], [11.07345, 46.149426], [11.073126, 46.145985]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;8&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;14645&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;11503&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.21291&quot;, &quot;id_fiume&quot;: 871, &quot;id_tratta&quot;: 20399, &quot;nome&quot;: &quot;NOCE&quot;, &quot;ordine&quot;: 2, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;TORRENTE&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.078526, 46.127751, 11.108842, 46.136844], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.108842, 46.136844], [11.107845, 46.136565], [11.104834, 46.135694], [11.103064, 46.134754], [11.102086, 46.133899], [11.100601, 46.133323], [11.094013, 46.132004], [11.087839, 46.131325], [11.085763, 46.130877], [11.080762, 46.128691], [11.079142, 46.127956], [11.078526, 46.127751]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;9&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;ADIGE&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;19318&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.21421&quot;, &quot;id_fiume&quot;: 885, &quot;id_tratta&quot;: 20614, &quot;nome&quot;: &quot;AVISIO&quot;, &quot;ordine&quot;: 2, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;TORRENTE&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.149084, 46.149406, 11.179088, 46.155095], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.179088, 46.154779], [11.177609, 46.155095], [11.174235, 46.154584], [11.168368, 46.154244], [11.164855, 46.15351], [11.161921, 46.152665], [11.159626, 46.152259], [11.15817, 46.151782], [11.156495, 46.15068], [11.154654, 46.149976], [11.1528, 46.149588], [11.151006, 46.149406], [11.149084, 46.149757]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;10&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;21173&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;21212&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.21779&quot;, &quot;id_fiume&quot;: 885, &quot;id_tratta&quot;: 21172, &quot;nome&quot;: &quot;AVISIO&quot;, &quot;ordine&quot;: 2, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;TORRENTE&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.148512, 46.12646, 11.154131, 46.149757], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.153167, 46.12646], [11.154061, 46.128468], [11.154131, 46.130267], [11.153599, 46.133202], [11.151897, 46.135376], [11.150743, 46.138998], [11.148512, 46.144215], [11.148582, 46.146005], [11.149084, 46.149757]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;11&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;AVISIO&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;SORGENTE&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.21780&quot;, &quot;id_fiume&quot;: 3398, &quot;id_tratta&quot;: 21173, &quot;nome&quot;: &quot;&quot;, &quot;ordine&quot;: 3, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.143672, 46.150494, 11.164512, 46.172489], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.164512, 46.172489], [11.161204, 46.170743], [11.15952, 46.169091], [11.158639, 46.167083], [11.157598, 46.165303], [11.154256, 46.162665], [11.150716, 46.159249], [11.146383, 46.15573], [11.14485, 46.15395], [11.144177, 46.153297], [11.143672, 46.150494]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;12&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;AVISIO&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;SORGENTE&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.22030&quot;, &quot;id_fiume&quot;: 3394, &quot;id_tratta&quot;: 21149, &quot;nome&quot;: &quot;&quot;, &quot;ordine&quot;: 3, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.143672, 46.149757, 11.149084, 46.150494], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.149084, 46.149757], [11.143672, 46.150494]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;13&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;21149&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;21173&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.22031&quot;, &quot;id_fiume&quot;: 885, &quot;id_tratta&quot;: 21150, &quot;nome&quot;: &quot;AVISIO&quot;, &quot;ordine&quot;: 2, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;TORRENTE&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.179088, 46.154779, 11.179814, 46.16601], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.179646, 46.16601], [11.17981, 46.161957], [11.179814, 46.158132], [11.179088, 46.154779]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;14&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;AVISIO&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;17589&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.22077&quot;, &quot;id_fiume&quot;: 3401, &quot;id_tratta&quot;: 21212, &quot;nome&quot;: &quot;&quot;, &quot;ordine&quot;: 3, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.179088, 46.154548, 11.189289, 46.155034], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.189289, 46.155034], [11.18753, 46.15478], [11.182729, 46.154548], [11.179921, 46.154601], [11.179088, 46.154779]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;15&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;21212&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;19969&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.22078&quot;, &quot;id_fiume&quot;: 885, &quot;id_tratta&quot;: 21213, &quot;nome&quot;: &quot;AVISIO&quot;, &quot;ordine&quot;: 2, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;TORRENTE&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.178798, 46.135197, 11.18159, 46.154779], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.178909, 46.135197], [11.178798, 46.136324], [11.179184, 46.137892], [11.180557, 46.139891], [11.181457, 46.142008], [11.18159, 46.14538], [11.180292, 46.149572], [11.179088, 46.154779]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;16&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;AVISIO&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;SORGENTE&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.22082&quot;, &quot;id_fiume&quot;: 3402, &quot;id_tratta&quot;: 21217, &quot;nome&quot;: &quot;&quot;, &quot;ordine&quot;: 3, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.058743, 46.209155, 11.105776, 46.241153], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.058743, 46.241153], [11.060005, 46.24023], [11.06011, 46.238878], [11.059677, 46.235961], [11.060893, 46.233464], [11.061987, 46.232544], [11.066836, 46.232114], [11.074557, 46.230516], [11.081012, 46.229724], [11.088391, 46.227681], [11.093632, 46.224993], [11.100916, 46.220135], [11.103266, 46.218183], [11.104458, 46.215452], [11.105452, 46.211618], [11.105776, 46.209155]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;17&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;20418&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;SPOREGGIO&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.9876&quot;, &quot;id_fiume&quot;: 871, &quot;id_tratta&quot;: 11501, &quot;nome&quot;: &quot;NOCE&quot;, &quot;ordine&quot;: 2, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;TORRENTE&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.05785, 46.243863, 11.058195, 46.247469], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.05785, 46.247469], [11.058195, 46.243863]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;18&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;LOVERNATICO&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;RINASSICO&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.13115&quot;, &quot;id_fiume&quot;: 871, &quot;id_tratta&quot;: 11494, &quot;nome&quot;: &quot;NOCE&quot;, &quot;ordine&quot;: 2, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;TORRENTE&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.058195, 46.241153, 11.058743, 46.243863], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.058195, 46.243863], [11.058743, 46.241153]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;19&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;SPOREGGIO&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;LOVERNATICO&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.13118&quot;, &quot;id_fiume&quot;: 871, &quot;id_tratta&quot;: 11497, &quot;nome&quot;: &quot;NOCE&quot;, &quot;ordine&quot;: 2, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;TORRENTE&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.014117, 46.243498, 11.049824, 46.248806], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.014117, 46.248806], [11.014631, 46.248302], [11.015044, 46.247899], [11.017629, 46.247286], [11.021148, 46.246324], [11.024525, 46.245364], [11.026983, 46.245887], [11.028928, 46.245853], [11.03131, 46.244686], [11.034834, 46.243498], [11.038087, 46.243773], [11.04102, 46.244171], [11.048494, 46.244595], [11.049824, 46.24548]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;20&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;20129&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;17421&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.21207&quot;, &quot;id_fiume&quot;: 3297, &quot;id_tratta&quot;: 20338, &quot;nome&quot;: &quot;LOVERNATICO&quot;, &quot;ordine&quot;: 3, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;TORRENTE&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.03248, 46.204956, 11.058743, 46.241153], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.03248, 46.204956], [11.033438, 46.205713], [11.034631, 46.207501], [11.034407, 46.210196], [11.033507, 46.211796], [11.033371, 46.212698], [11.034077, 46.214261], [11.03578, 46.21648], [11.035838, 46.218045], [11.035757, 46.220072], [11.03648, 46.222426], [11.036231, 46.224123], [11.03528, 46.22504], [11.034202, 46.226409], [11.033587, 46.22732], [11.033637, 46.228669], [11.034667, 46.230217], [11.036349, 46.231879], [11.038524, 46.233865], [11.040535, 46.235629], [11.04285, 46.236488], [11.045453, 46.237116], [11.047755, 46.237642], [11.050688, 46.23803], [11.052342, 46.238901], [11.054151, 46.239777], [11.055801, 46.240531], [11.05743, 46.240726], [11.058743, 46.241153]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;21&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;NOCE&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;16944&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.21316&quot;, &quot;id_fiume&quot;: 3283, &quot;id_tratta&quot;: 20453, &quot;nome&quot;: &quot;SPOREGGIO&quot;, &quot;ordine&quot;: 3, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;TORRENTE&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.049824, 46.243863, 11.058195, 46.24548], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.049824, 46.24548], [11.054327, 46.24449], [11.058195, 46.243863]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;22&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;NOCE&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;20129&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.21342&quot;, &quot;id_fiume&quot;: 3297, &quot;id_tratta&quot;: 20506, &quot;nome&quot;: &quot;LOVERNATICO&quot;, &quot;ordine&quot;: 3, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;TORRENTE&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.200894, 46.223546, 11.209126, 46.23092], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.200894, 46.223546], [11.201272, 46.22488], [11.202964, 46.226656], [11.205117, 46.227965], [11.207794, 46.230046], [11.209126, 46.23092]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;23&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;21260&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;20678&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.21486&quot;, &quot;id_fiume&quot;: 923, &quot;id_tratta&quot;: 20679, &quot;nome&quot;: &quot;TIGLIA&quot;, &quot;ordine&quot;: 2, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;RIO&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.136197, 46.205277, 11.180773, 46.247556], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.180773, 46.247556], [11.176955, 46.245487], [11.171295, 46.24177], [11.165507, 46.239062], [11.158889, 46.235704], [11.153751, 46.232651], [11.147782, 46.229272], [11.144939, 46.226859], [11.142566, 46.224203], [11.140949, 46.220292], [11.139837, 46.216712], [11.137109, 46.208888], [11.136197, 46.20643], [11.139167, 46.205277]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;24&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;ADIGE&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;21187&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.21999&quot;, &quot;id_fiume&quot;: 911, &quot;id_tratta&quot;: 21118, &quot;nome&quot;: &quot;DI CALDARO&quot;, &quot;ordine&quot;: 2, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;FOSSA&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.147903, 46.220946, 11.214291, 46.251017], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.214291, 46.251017], [11.212225, 46.249005], [11.208756, 46.245643], [11.206575, 46.244623], [11.202676, 46.24315], [11.198799, 46.242217], [11.194481, 46.24095], [11.189708, 46.23934], [11.185433, 46.237847], [11.181807, 46.236682], [11.178448, 46.23536], [11.176047, 46.23402], [11.171231, 46.230629], [11.168879, 46.22953], [11.167571, 46.22825], [11.166183, 46.227898], [11.162105, 46.227426], [11.158564, 46.227727], [11.155483, 46.227534], [11.153605, 46.226939], [11.151661, 46.225329], [11.149669, 46.223791], [11.147903, 46.222402], [11.148262, 46.220946]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;25&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;21153&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;TIGLIA&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.22033&quot;, &quot;id_fiume&quot;: 215, &quot;id_tratta&quot;: 21152, &quot;nome&quot;: &quot;ADIGE&quot;, &quot;ordine&quot;: 1, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;FIUME&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.107422, 46.110465, 11.142192, 46.134109], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.142192, 46.110465], [11.141283, 46.112057], [11.140059, 46.11388], [11.137405, 46.11618], [11.134491, 46.11646], [11.129645, 46.116667], [11.125127, 46.116975], [11.122933, 46.118474], [11.12001, 46.118528], [11.117074, 46.117908], [11.114794, 46.117167], [11.110922, 46.117455], [11.108362, 46.118186], [11.107422, 46.119328], [11.107473, 46.120677], [11.108024, 46.122233], [11.108476, 46.1256], [11.109075, 46.128739], [11.110813, 46.131866], [11.111222, 46.134109]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;26&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;19318&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;SORGENTE&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.20260&quot;, &quot;id_fiume&quot;: 7390, &quot;id_tratta&quot;: 20547, &quot;nome&quot;: &quot;&quot;, &quot;ordine&quot;: 4, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.111222, 46.121756, 11.140041, 46.134109], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.140041, 46.121756], [11.139429, 46.122667], [11.137192, 46.123384], [11.134045, 46.125693], [11.130096, 46.12835], [11.126034, 46.131801], [11.122214, 46.133789], [11.117371, 46.134104], [11.111222, 46.134109]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;27&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;19318&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;SORGENTE&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.20261&quot;, &quot;id_fiume&quot;: 7391, &quot;id_tratta&quot;: 20548, &quot;nome&quot;: &quot;&quot;, &quot;ordine&quot;: 4, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.078526, 46.085624, 11.103656, 46.127751], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.078526, 46.127751], [11.0802, 46.124075], [11.081398, 46.121488], [11.082946, 46.115853], [11.084249, 46.111284], [11.085804, 46.10896], [11.088373, 46.104665], [11.090784, 46.100013], [11.095041, 46.092501], [11.096563, 46.08998], [11.097249, 46.088977], [11.103656, 46.085624]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;28&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;VELA&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;AVISIO&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.21420&quot;, &quot;id_fiume&quot;: 215, &quot;id_tratta&quot;: 20613, &quot;nome&quot;: &quot;ADIGE&quot;, &quot;ordine&quot;: 1, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;FIUME&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.191095, 46.217767, 11.200894, 46.223546], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.191095, 46.217767], [11.192111, 46.218872], [11.196274, 46.221493], [11.200894, 46.223546]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;29&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;20678&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;SORGENTE&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.17842&quot;, &quot;id_fiume&quot;: 923, &quot;id_tratta&quot;: 19962, &quot;nome&quot;: &quot;TIGLIA&quot;, &quot;ordine&quot;: 2, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;RIO&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.200894, 46.207687, 11.212083, 46.223546], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.212083, 46.207687], [11.210687, 46.209064], [11.20946, 46.211113], [11.206547, 46.215112], [11.203758, 46.218306], [11.200894, 46.223546]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;30&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;TIGLIA&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;SORGENTE&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.21485&quot;, &quot;id_fiume&quot;: 3327, &quot;id_tratta&quot;: 20678, &quot;nome&quot;: &quot;&quot;, &quot;ordine&quot;: 3, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.139167, 46.205277, 11.148262, 46.220946], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.148262, 46.220946], [11.146583, 46.219124], [11.146147, 46.216594], [11.146378, 46.213881], [11.146134, 46.211941], [11.145481, 46.210486], [11.143869, 46.208708], [11.141913, 46.207412], [11.139167, 46.205277]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;31&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;DI CALDARO&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;21153&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.22000&quot;, &quot;id_fiume&quot;: 215, &quot;id_tratta&quot;: 21119, &quot;nome&quot;: &quot;ADIGE&quot;, &quot;ordine&quot;: 1, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;FIUME&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.148262, 46.20669, 11.190036, 46.220946], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.190036, 46.207437], [11.178701, 46.207654], [11.173501, 46.207303], [11.170236, 46.20669], [11.167192, 46.207755], [11.165018, 46.209822], [11.160186, 46.214872], [11.158308, 46.216591], [11.155752, 46.217538], [11.154834, 46.218906], [11.152908, 46.219392], [11.150006, 46.219789], [11.148262, 46.220946]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;32&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;ADIGE&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;SORGENTE&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.22034&quot;, &quot;id_fiume&quot;: 915, &quot;id_tratta&quot;: 21153, &quot;nome&quot;: &quot;&quot;, &quot;ordine&quot;: 2, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.07745, 46.182762, 11.105776, 46.209584], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.07745, 46.182762], [11.078451, 46.183194], [11.07978, 46.18407], [11.080815, 46.186076], [11.082064, 46.188979], [11.083706, 46.193674], [11.083844, 46.201772], [11.084321, 46.205813], [11.085517, 46.207258], [11.087502, 46.208347], [11.090136, 46.209424], [11.093707, 46.209584], [11.095957, 46.209425], [11.098884, 46.209372], [11.105776, 46.209155]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;33&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;NOCE&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;SORGENTE&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.15561&quot;, &quot;id_fiume&quot;: 3278, &quot;id_tratta&quot;: 20418, &quot;nome&quot;: &quot;&quot;, &quot;ordine&quot;: 3, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.030344, 46.196327, 11.03248, 46.204956], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.030344, 46.196327], [11.030688, 46.19722], [11.03207, 46.199221], [11.03248, 46.204956]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;34&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;16944&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;20440&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.15584&quot;, &quot;id_fiume&quot;: 3283, &quot;id_tratta&quot;: 20441, &quot;nome&quot;: &quot;SPOREGGIO&quot;, &quot;ordine&quot;: 3, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;TORRENTE&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [10.991113, 46.193075, 11.03248, 46.204956], &quot;geometry&quot;: {&quot;coordinates&quot;: [[10.991113, 46.19568], [10.993634, 46.194039], [10.996205, 46.193103], [10.997824, 46.193075], [10.998943, 46.193199], [11.000426, 46.193362], [11.003988, 46.1933], [11.007226, 46.193243], [11.010169, 46.193983], [11.012796, 46.195287], [11.015448, 46.19659], [11.018067, 46.197669], [11.020068, 46.198867], [11.021585, 46.20064], [11.023577, 46.201955], [11.025562, 46.203045], [11.027505, 46.20301], [11.030116, 46.203522], [11.031769, 46.204392], [11.03248, 46.204956]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;35&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;SPOREGGIO&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;18146&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.18119&quot;, &quot;id_fiume&quot;: 6687, &quot;id_tratta&quot;: 16944, &quot;nome&quot;: &quot;&quot;, &quot;ordine&quot;: 4, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.139167, 46.194006, 11.174597, 46.205277], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.174597, 46.194006], [11.172025, 46.194505], [11.1683, 46.194693], [11.163775, 46.195004], [11.160874, 46.195059], [11.157614, 46.194553], [11.153098, 46.195088], [11.15155, 46.196918], [11.150658, 46.199302], [11.148314, 46.201362], [11.139167, 46.205277]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;36&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;ADIGE&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;SORGENTE&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.22001&quot;, &quot;id_fiume&quot;: 912, &quot;id_tratta&quot;: 21120, &quot;nome&quot;: &quot;&quot;, &quot;ordine&quot;: 2, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.030344, 46.17774, 11.050545, 46.196327], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.050545, 46.17774], [11.047626, 46.181834], [11.045473, 46.184914], [11.0422, 46.188573], [11.038697, 46.190211], [11.034526, 46.191302], [11.033465, 46.193112], [11.032572, 46.195603], [11.030344, 46.196327]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;37&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;SPOREGGIO&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;SORGENTE&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.15583&quot;, &quot;id_fiume&quot;: 7372, &quot;id_tratta&quot;: 20440, &quot;nome&quot;: &quot;&quot;, &quot;ordine&quot;: 4, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.171502, 46.16601, 11.179646, 46.184828], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.172136, 46.184828], [11.171502, 46.18124], [11.17332, 46.178055], [11.174278, 46.173763], [11.175165, 46.171271], [11.17571, 46.16901], [11.177435, 46.167402], [11.179646, 46.16601]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;38&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;21212&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;SORGENTE&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.18660&quot;, &quot;id_fiume&quot;: 6808, &quot;id_tratta&quot;: 17589, &quot;nome&quot;: &quot;&quot;, &quot;ordine&quot;: 4, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.194527, 46.158245, 11.214153, 46.190026], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.194527, 46.190026], [11.197772, 46.185914], [11.201544, 46.183033], [11.206771, 46.180232], [11.20993, 46.178254], [11.212448, 46.176405], [11.214153, 46.174347], [11.214081, 46.172548], [11.212447, 46.168305], [11.209714, 46.164757], [11.206179, 46.161568], [11.204175, 46.160031], [11.203505, 46.159144], [11.203469, 46.158245]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;39&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;AVISIO&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;SORGENTE&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.20274&quot;, &quot;id_fiume&quot;: 3372, &quot;id_tratta&quot;: 21042, &quot;nome&quot;: &quot;&quot;, &quot;ordine&quot;: 3, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.01522, 46.17409, 11.030344, 46.196327], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.015416, 46.17409], [11.01522, 46.177244], [11.017212, 46.178559], [11.020048, 46.180642], [11.023085, 46.183963], [11.023475, 46.185756], [11.024992, 46.187529], [11.026394, 46.190429], [11.028608, 46.193198], [11.030344, 46.196327]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;40&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;20440&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;SORGENTE&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.21232&quot;, &quot;id_fiume&quot;: 3283, &quot;id_tratta&quot;: 20363, &quot;nome&quot;: &quot;SPOREGGIO&quot;, &quot;ordine&quot;: 3, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;TORRENTE&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.073126, 46.145985, 11.139167, 46.205277], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.139167, 46.205277], [11.137355, 46.203861], [11.134969, 46.201512], [11.132726, 46.198809], [11.131344, 46.196594], [11.130042, 46.195098], [11.128491, 46.193875], [11.125812, 46.192953], [11.120458, 46.191225], [11.118307, 46.189555], [11.116008, 46.186043], [11.114995, 46.184289], [11.113653, 46.182756], [11.112576, 46.181354], [11.11196, 46.17881], [11.110804, 46.173647], [11.11014, 46.169177], [11.109459, 46.16729], [11.107756, 46.165756], [11.102484, 46.162721], [11.097666, 46.159317], [11.095775, 46.158289], [11.089017, 46.156316], [11.085742, 46.155682], [11.081638, 46.154353], [11.080141, 46.15312], [11.078574, 46.15035], [11.07563, 46.148189], [11.074299, 46.146863], [11.073126, 46.145985]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;41&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;14645&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;DI CALDARO&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.21290&quot;, &quot;id_fiume&quot;: 215, &quot;id_tratta&quot;: 20398, &quot;nome&quot;: &quot;ADIGE&quot;, &quot;ordine&quot;: 1, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;FIUME&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.076413, 46.159871, 11.106075, 46.209155], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.105776, 46.209155], [11.106075, 46.206881], [11.105622, 46.203514], [11.103995, 46.199269], [11.100681, 46.192913], [11.095711, 46.185579], [11.093632, 46.182359], [11.091629, 46.180812], [11.090733, 46.178362], [11.089046, 46.176584], [11.086183, 46.173495], [11.083457, 46.170278], [11.080245, 46.166511], [11.078184, 46.163398], [11.077163, 46.162067], [11.076751, 46.160859], [11.076413, 46.159871]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;42&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;11503&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;20418&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.21308&quot;, &quot;id_fiume&quot;: 871, &quot;id_tratta&quot;: 20416, &quot;nome&quot;: &quot;NOCE&quot;, &quot;ordine&quot;: 2, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;TORRENTE&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.179646, 46.16601, 11.185562, 46.182466], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.183705, 46.182466], [11.185345, 46.17906], [11.185562, 46.176364], [11.184626, 46.173007], [11.182214, 46.169678], [11.181025, 46.168117], [11.179646, 46.16601]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;43&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;21212&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;SORGENTE&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.21914&quot;, &quot;id_fiume&quot;: 7504, &quot;id_tratta&quot;: 21238, &quot;nome&quot;: &quot;&quot;, &quot;ordine&quot;: 4, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.0598, 46.27209, 11.06837, 46.281588], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.06837, 46.281588], [11.067896, 46.276698], [11.06714, 46.275458], [11.063908, 46.273451], [11.0598, 46.27209]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;44&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;NOCE&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;20598&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.9874&quot;, &quot;id_fiume&quot;: 2239, &quot;id_tratta&quot;: 11499, &quot;nome&quot;: &quot;PONGAIOLA&quot;, &quot;ordine&quot;: 3, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;RIO&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.031392, 46.268257, 11.046455, 46.282026], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.031392, 46.282026], [11.034956, 46.281854], [11.038764, 46.279537], [11.041788, 46.278133], [11.044673, 46.276848], [11.045249, 46.275254], [11.045149, 46.272564], [11.044918, 46.270534], [11.045184, 46.268964], [11.046455, 46.268257]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;45&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;20359&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;SORGENTE&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.15575&quot;, &quot;id_fiume&quot;: 7371, &quot;id_tratta&quot;: 20432, &quot;nome&quot;: &quot;&quot;, &quot;ordine&quot;: 4, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.092775, 46.267063, 11.134008, 46.279373], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.134008, 46.279373], [11.130336, 46.276741], [11.127054, 46.275785], [11.123786, 46.27518], [11.120522, 46.274674], [11.117296, 46.275184], [11.113042, 46.274587], [11.109229, 46.2723], [11.106234, 46.27033], [11.10329, 46.2697], [11.099395, 46.269664], [11.094043, 46.267777], [11.092775, 46.267063]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;46&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;20636&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;SORGENTE&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.21442&quot;, &quot;id_fiume&quot;: 2238, &quot;id_tratta&quot;: 20635, &quot;nome&quot;: &quot;RINASSICO&quot;, &quot;ordine&quot;: 3, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;RIO&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.058757, 46.263203, 11.060734, 46.27209], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.0598, 46.27209], [11.060494, 46.270597], [11.060734, 46.268342], [11.059441, 46.264316], [11.058757, 46.263203]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;47&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;20488&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;PONGAIOLA&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.9875&quot;, &quot;id_fiume&quot;: 871, &quot;id_tratta&quot;: 11500, &quot;nome&quot;: &quot;NOCE&quot;, &quot;ordine&quot;: 2, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;TORRENTE&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.055981, 46.247469, 11.058757, 46.263203], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.058757, 46.263203], [11.058073, 46.26209], [11.055981, 46.254028], [11.056655, 46.25019], [11.05756, 46.248374], [11.05785, 46.247469]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;48&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;RINASSICO&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;20488&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.13113&quot;, &quot;id_fiume&quot;: 871, &quot;id_tratta&quot;: 11492, &quot;nome&quot;: &quot;NOCE&quot;, &quot;ordine&quot;: 2, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;TORRENTE&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.05785, 46.247469, 11.092775, 46.267063], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.092775, 46.267063], [11.091033, 46.266013], [11.087792, 46.264658], [11.082675, 46.262957], [11.078735, 46.259992], [11.075222, 46.258028], [11.066093, 46.25185], [11.06154, 46.249522], [11.05785, 46.247469]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;49&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;NOCE&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;20636&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.13114&quot;, &quot;id_fiume&quot;: 2238, &quot;id_tratta&quot;: 11493, &quot;nome&quot;: &quot;RINASSICO&quot;, &quot;ordine&quot;: 3, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;RIO&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.180773, 46.247556, 11.204676, 46.25744], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.204676, 46.25744], [11.204019, 46.257236], [11.201687, 46.255705], [11.193267, 46.25215], [11.187015, 46.249903], [11.180773, 46.247556]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;50&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;21187&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;PICCOLO DI CALDARO&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.17975&quot;, &quot;id_fiume&quot;: 911, &quot;id_tratta&quot;: 21199, &quot;nome&quot;: &quot;DI CALDARO&quot;, &quot;ordine&quot;: 2, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;FOSSA&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [10.97914, 46.24548, 11.049824, 46.262176], &quot;geometry&quot;: {&quot;coordinates&quot;: [[10.97914, 46.259208], [10.984084, 46.261373], [10.989948, 46.262063], [10.994786, 46.261294], [10.998049, 46.261805], [10.998934, 46.261871], [11.002591, 46.262176], [11.011997, 46.262119], [11.017155, 46.261245], [11.021366, 46.26072], [11.025645, 46.258169], [11.031684, 46.254804], [11.03616, 46.253032], [11.039506, 46.251623], [11.041713, 46.249558], [11.044218, 46.247488], [11.047113, 46.24687], [11.049824, 46.24548]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;51&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;LOVERNATICO&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;SORGENTE&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.21072&quot;, &quot;id_fiume&quot;: 7310, &quot;id_tratta&quot;: 20129, &quot;nome&quot;: &quot;&quot;, &quot;ordine&quot;: 4, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.092775, 46.256773, 11.109283, 46.267063], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.109283, 46.256773], [11.107048, 46.257715], [11.10321, 46.259135], [11.099591, 46.262235], [11.097446, 46.265199], [11.092775, 46.267063]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;52&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;RINASSICO&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;SORGENTE&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.21443&quot;, &quot;id_fiume&quot;: 7410, &quot;id_tratta&quot;: 20636, &quot;nome&quot;: &quot;&quot;, &quot;ordine&quot;: 4, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.152148, 46.247556, 11.180773, 46.283659], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.157372, 46.283659], [11.154769, 46.283483], [11.152148, 46.282858], [11.1527, 46.280372], [11.153367, 46.276535], [11.153697, 46.272703], [11.155861, 46.269962], [11.156533, 46.266575], [11.1564, 46.263202], [11.159549, 46.260776], [11.163413, 46.255761], [11.166065, 46.253236], [11.167969, 46.25185], [11.171521, 46.251441], [11.174092, 46.251167], [11.17767, 46.251099], [11.178922, 46.250283], [11.180156, 46.248693], [11.180129, 46.24801], [11.180773, 46.247556]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;53&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;DI CALDARO&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;SORGENTE&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.21794&quot;, &quot;id_fiume&quot;: 3399, &quot;id_tratta&quot;: 21187, &quot;nome&quot;: &quot;&quot;, &quot;ordine&quot;: 3, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.018154, 46.261495, 11.046455, 46.268257], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.018154, 46.266177], [11.020731, 46.265682], [11.023128, 46.264514], [11.025035, 46.263805], [11.029869, 46.262928], [11.035007, 46.261495], [11.037625, 46.261782], [11.041195, 46.262168], [11.044158, 46.263015], [11.045825, 46.264227], [11.046389, 46.266467], [11.046455, 46.268257]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;54&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;20432&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;SORGENTE&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.21228&quot;, &quot;id_fiume&quot;: 7352, &quot;id_tratta&quot;: 20359, &quot;nome&quot;: &quot;&quot;, &quot;ordine&quot;: 4, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 1}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.046455, 46.263203, 11.058757, 46.268257], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.046455, 46.268257], [11.049049, 46.26821], [11.053846, 46.266333], [11.057185, 46.264356], [11.058757, 46.263203]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;55&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;NOCE&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;20359&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.21324&quot;, &quot;id_fiume&quot;: 3293, &quot;id_tratta&quot;: 20488, &quot;nome&quot;: &quot;&quot;, &quot;ordine&quot;: 3, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.056342, 46.27209, 11.0598, 46.293786], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.059741, 46.293786], [11.059682, 46.292212], [11.058806, 46.290652], [11.058086, 46.288415], [11.056999, 46.285735], [11.056418, 46.283045], [11.056342, 46.281022], [11.057698, 46.278072], [11.058604, 46.276256], [11.058852, 46.274226], [11.059437, 46.272866], [11.0598, 46.27209]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;56&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;PONGAIOLA&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;TRESENICA&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.13117&quot;, &quot;id_fiume&quot;: 871, &quot;id_tratta&quot;: 11496, &quot;nome&quot;: &quot;NOCE&quot;, &quot;ordine&quot;: 2, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;TORRENTE&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.06837, 46.281588, 11.073801, 46.292892], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.073801, 46.292892], [11.073111, 46.291294], [11.071063, 46.288289], [11.069778, 46.284496], [11.06837, 46.281588]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;57&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;PONGAIOLA&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;19014&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.21405&quot;, &quot;id_fiume&quot;: 7397, &quot;id_tratta&quot;: 20598, &quot;nome&quot;: &quot;&quot;, &quot;ordine&quot;: 4, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}, {&quot;bbox&quot;: [11.06837, 46.281588, 11.102509, 46.293848], &quot;geometry&quot;: {&quot;coordinates&quot;: [[11.102509, 46.293848], [11.101257, 46.292922], [11.098118, 46.291288], [11.095437, 46.289087], [11.092074, 46.285998], [11.089211, 46.283459], [11.085294, 46.282864], [11.083006, 46.282447], [11.081381, 46.282368], [11.079128, 46.282859], [11.075262, 46.283605], [11.07233, 46.283325], [11.06837, 46.281588]], &quot;type&quot;: &quot;LineString&quot;}, &quot;id&quot;: &quot;58&quot;, &quot;properties&quot;: {&quot;a&quot;: &quot;20598&quot;, &quot;annotazion&quot;: &quot;&quot;, &quot;bacino&quot;: &quot;&quot;, &quot;bacino_pri&quot;: &quot;ADIGE&quot;, &quot;da&quot;: &quot;20797&quot;, &quot;enabled&quot;: 1, &quot;foglio_igm&quot;: &quot;BOLZANO&quot;, &quot;gml_id&quot;: &quot;ID.RETICOLO.CORSI_ACQUA.21406&quot;, &quot;id_fiume&quot;: 2239, &quot;id_tratta&quot;: 20599, &quot;nome&quot;: &quot;PONGAIOLA&quot;, &quot;ordine&quot;: 3, &quot;sottotipo&quot;: 1, &quot;tipo&quot;: &quot;RIO&quot;, &quot;tipo_a&quot;: 3, &quot;tipo_da&quot;: 3}, &quot;type&quot;: &quot;Feature&quot;}], &quot;type&quot;: &quot;FeatureCollection&quot;});



    geo_json_74838407a5aaf4a7f5a8faa7218a4301.bindTooltip(
    function(layer){
    let div = L.DomUtil.create(&#x27;div&#x27;);

    let handleObject = feature=&gt;typeof(feature)==&#x27;object&#x27; ? JSON.stringify(feature) : feature;
    let fields = [&quot;gml_id&quot;, &quot;id_fiume&quot;, &quot;id_tratta&quot;, &quot;tipo&quot;, &quot;nome&quot;, &quot;foglio_igm&quot;, &quot;sottotipo&quot;, &quot;ordine&quot;, &quot;bacino_pri&quot;, &quot;bacino&quot;, &quot;da&quot;, &quot;tipo_da&quot;, &quot;a&quot;, &quot;tipo_a&quot;, &quot;annotazion&quot;, &quot;enabled&quot;];
    let aliases = [&quot;gml_id&quot;, &quot;id_fiume&quot;, &quot;id_tratta&quot;, &quot;tipo&quot;, &quot;nome&quot;, &quot;foglio_igm&quot;, &quot;sottotipo&quot;, &quot;ordine&quot;, &quot;bacino_pri&quot;, &quot;bacino&quot;, &quot;da&quot;, &quot;tipo_da&quot;, &quot;a&quot;, &quot;tipo_a&quot;, &quot;annotazion&quot;, &quot;enabled&quot;];
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



## repeat the same exercise with the layer “Comuni Terremotati” (municipalities affected by earthquake) of the italian Civil Protection by choosing the smallest municipality contained on the layer


```python
import os
os.environ['RESTAPI_USE_ARCPY'] = 'FALSE'
import restapi
```


```python
rest_url = 'https://services6.arcgis.com/L1SotImj1AAZY1eK/ArcGIS/rest/services'
ags = restapi.ArcServer(rest_url)
```


```python
agc_service_name = ""
for service in ags.services:
  if service.name == 'Comuni_Terremotati':
    agc_service_name = service.name
    print(service.name)
```

    Comuni_Terremotati



```python
ags_service = ags.getService(agc_service_name)
ags_service.list_layers()
```


    ---------------------------------------------------------------------------

    AttributeError                            Traceback (most recent call last)

    ~/.local/lib/python3.10/site-packages/restapi/rest_utils.py in __getattr__(self, name)
        862             # it is a class attribute
    --> 863             return object.__getattribute__(self, name)
        864         except AttributeError:


    AttributeError: 'ArcServer' object has no attribute 'folders'

    
    During handling of the above exception, another exception occurred:


    AttributeError                            Traceback (most recent call last)

    /tmp/ipykernel_37029/1180447732.py in <module>
    ----> 1 ags_service = ags.getService(agc_service_name)
          2 ags_service.list_layers()


    ~/.local/lib/python3.10/site-packages/restapi/common_types.py in getService(self, name_or_wildcard)
       1275             NotImplementedError: 'restapi does not support "{}" services!'
       1276         """
    -> 1277         full_path = self.get_service_url(name_or_wildcard)
       1278         if full_path:
       1279             extension = full_path.split('/')[-1]


    ~/.local/lib/python3.10/site-packages/restapi/common_types.py in get_service_url(self, wildcard, _list)
       1361 
       1362         if not self.service_cache:
    -> 1363             self.list_services()
       1364         if '*' in wildcard:
       1365             if wildcard == '*':


    ~/.local/lib/python3.10/site-packages/restapi/common_types.py in list_services(self, filterer)
       1329     def list_services(self, filterer=True):
       1330         """Returns a list of all services."""
    -> 1331         return list(self.iter_services(filterer))
       1332 
       1333     def iter_services(self, token='', filterer=True):


    ~/.local/lib/python3.10/site-packages/restapi/common_types.py in iter_services(self, token, filterer)
       1343             yield full_service_url
       1344 
    -> 1345         for s in self.folders:
       1346             new = '/'.join([self.url, s])
       1347             resp = self.request(new)


    ~/.local/lib/python3.10/site-packages/restapi/rest_utils.py in __getattr__(self, name)
        867                 return self.json[name]
        868             else:
    --> 869                 raise AttributeError(name)
        870 
        871     def __str__(self):


    AttributeError: folders



```python
municipalities_affected_earthquake = ags_service.layer('comuni_terremotatiDD')
```

we can ask ArcGIS RestAPI to transform the source from the native projection to the [EPSG:25832](http://epsg.io/25832)


```python
municipalities_affected_earthquake.export_layer('municipalities_affected_earthquake.shp', outSR=25832)
```

    Created: "municipalities_affected_earthquake.shp"





    'municipalities_affected_earthquake.shp'




```python
geo_municipalities_affected_earthquake = gpd.read_file('municipalities_affected_earthquake.shp')
```


```python
geo_municipalities_affected_earthquake.geometry
```




    0     POLYGON ((825823.37496 4744509.99999, 825831.8...
    1     POLYGON ((822188.87502 4732801.00004, 822431.1...
    2     POLYGON ((836677.62503 4755035.49995, 838169.8...
    3     POLYGON ((832493.43752 4758168.99995, 833177.3...
    4     POLYGON ((858381.68751 4751289.99998, 858884.9...
    5     POLYGON ((849760.43751 4750082.49996, 849960.9...
    6     POLYGON ((846041.81249 4765923.99998, 846232.9...
    7     POLYGON ((856514.06245 4756609.99998, 856643.0...
    8     POLYGON ((850434.12498 4761464.99995, 850516.5...
    9     POLYGON ((850348.12500 4740939.49997, 850545.4...
    10    POLYGON ((856495.18747 4734432.49994, 856419.9...
    11    POLYGON ((832470.37500 4704205.50001, 832598.1...
    12    POLYGON ((860575.24996 4723655.50002, 860818.4...
    13    POLYGON ((853776.93752 4720760.50001, 853849.4...
    14    POLYGON ((845010.24996 4723371.99999, 845251.4...
    15    POLYGON ((873001.31246 4740017.50004, 873145.9...
    16    POLYGON ((871474.12497 4750509.50001, 871638.4...
    Name: geometry, dtype: geometry



the values of the coordinates seems to be in meters


```python
geo_municipalities_affected_earthquake.crs
```




    <Geographic 2D CRS: EPSG:4326>
    Name: WGS 84
    Axis Info [ellipsoidal]:
    - Lat[north]: Geodetic latitude (degree)
    - Lon[east]: Geodetic longitude (degree)
    Area of Use:
    - name: World.
    - bounds: (-180.0, -90.0, 180.0, 90.0)
    Datum: World Geodetic System 1984 ensemble
    - Ellipsoid: WGS 84
    - Prime Meridian: Greenwich



.. but the CRS is EPSG:426 ... we need to rewrite it!!!


```python
geo_municipalities_affected_earthquake = geo_municipalities_affected_earthquake.set_crs(epsg=25832,allow_override=True)
```


```python
geo_municipalities_affected_earthquake.crs
```




    <Projected CRS: EPSG:25832>
    Name: ETRS89 / UTM zone 32N
    Axis Info [cartesian]:
    - E[east]: Easting (metre)
    - N[north]: Northing (metre)
    Area of Use:
    - name: Europe between 6°E and 12°E: Austria; Belgium; Denmark - onshore and offshore; Germany - onshore and offshore; Norway including - onshore and offshore; Spain - offshore.
    - bounds: (6.0, 38.76, 12.0, 84.33)
    Coordinate Operation:
    - name: UTM zone 32N
    - method: Transverse Mercator
    Datum: European Terrestrial Reference System 1989 ensemble
    - Ellipsoid: GRS 1980
    - Prime Meridian: Greenwich



Now is ok!


```python
geo_municipalities_affected_earthquake.explore()
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe src="about:blank" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" data-html=%3C%21DOCTYPE%20html%3E%0A%3Chead%3E%20%20%20%20%0A%20%20%20%20%3Cmeta%20http-equiv%3D%22content-type%22%20content%3D%22text/html%3B%20charset%3DUTF-8%22%20/%3E%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%3Cscript%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20L_NO_TOUCH%20%3D%20false%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20L_DISABLE_3D%20%3D%20false%3B%0A%20%20%20%20%20%20%20%20%3C/script%3E%0A%20%20%20%20%0A%20%20%20%20%3Cstyle%3Ehtml%2C%20body%20%7Bwidth%3A%20100%25%3Bheight%3A%20100%25%3Bmargin%3A%200%3Bpadding%3A%200%3B%7D%3C/style%3E%0A%20%20%20%20%3Cstyle%3E%23map%20%7Bposition%3Aabsolute%3Btop%3A0%3Bbottom%3A0%3Bright%3A0%3Bleft%3A0%3B%7D%3C/style%3E%0A%20%20%20%20%3Cscript%20src%3D%22https%3A//cdn.jsdelivr.net/npm/leaflet%401.6.0/dist/leaflet.js%22%3E%3C/script%3E%0A%20%20%20%20%3Cscript%20src%3D%22https%3A//code.jquery.com/jquery-1.12.4.min.js%22%3E%3C/script%3E%0A%20%20%20%20%3Cscript%20src%3D%22https%3A//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/js/bootstrap.min.js%22%3E%3C/script%3E%0A%20%20%20%20%3Cscript%20src%3D%22https%3A//cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.js%22%3E%3C/script%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//cdn.jsdelivr.net/npm/leaflet%401.6.0/dist/leaflet.css%22/%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css%22/%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap-theme.min.css%22/%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//maxcdn.bootstrapcdn.com/font-awesome/4.6.3/css/font-awesome.min.css%22/%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.css%22/%3E%0A%20%20%20%20%3Clink%20rel%3D%22stylesheet%22%20href%3D%22https%3A//cdn.jsdelivr.net/gh/python-visualization/folium/folium/templates/leaflet.awesome.rotate.min.css%22/%3E%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Cmeta%20name%3D%22viewport%22%20content%3D%22width%3Ddevice-width%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20initial-scale%3D1.0%2C%20maximum-scale%3D1.0%2C%20user-scalable%3Dno%22%20/%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Cstyle%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%23map_5409f38fb1874a85aa381438d8e5d24f%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20position%3A%20relative%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20width%3A%20100.0%25%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20height%3A%20100.0%25%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20left%3A%200.0%25%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20top%3A%200.0%25%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%3C/style%3E%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%3Cstyle%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20.foliumtooltip%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20.foliumtooltip%20table%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20margin%3A%20auto%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20.foliumtooltip%20tr%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20text-align%3A%20left%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20.foliumtooltip%20th%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20padding%3A%202px%3B%20padding-right%3A%208px%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%3C/style%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%0A%3C/head%3E%0A%3Cbody%3E%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Cdiv%20class%3D%22folium-map%22%20id%3D%22map_5409f38fb1874a85aa381438d8e5d24f%22%20%3E%3C/div%3E%0A%20%20%20%20%20%20%20%20%0A%3C/body%3E%0A%3Cscript%3E%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20var%20map_5409f38fb1874a85aa381438d8e5d24f%20%3D%20L.map%28%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22map_5409f38fb1874a85aa381438d8e5d24f%22%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20center%3A%20%5B42.66608550044058%2C%2013.248112428614302%5D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20crs%3A%20L.CRS.EPSG3857%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20zoom%3A%2010%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20zoomControl%3A%20true%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20preferCanvas%3A%20false%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%29%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20L.control.scale%28%29.addTo%28map_5409f38fb1874a85aa381438d8e5d24f%29%3B%0A%0A%20%20%20%20%20%20%20%20%20%20%20%20%0A%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20var%20tile_layer_11320e2690c9425594087c616dde4862%20%3D%20L.tileLayer%28%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22https%3A//%7Bs%7D.tile.openstreetmap.org/%7Bz%7D/%7Bx%7D/%7By%7D.png%22%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7B%22attribution%22%3A%20%22Data%20by%20%5Cu0026copy%3B%20%5Cu003ca%20href%3D%5C%22http%3A//openstreetmap.org%5C%22%5Cu003eOpenStreetMap%5Cu003c/a%5Cu003e%2C%20under%20%5Cu003ca%20href%3D%5C%22http%3A//www.openstreetmap.org/copyright%5C%22%5Cu003eODbL%5Cu003c/a%5Cu003e.%22%2C%20%22detectRetina%22%3A%20false%2C%20%22maxNativeZoom%22%3A%2018%2C%20%22maxZoom%22%3A%2018%2C%20%22minZoom%22%3A%200%2C%20%22noWrap%22%3A%20false%2C%20%22opacity%22%3A%201%2C%20%22subdomains%22%3A%20%22abc%22%2C%20%22tms%22%3A%20false%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%29.addTo%28map_5409f38fb1874a85aa381438d8e5d24f%29%3B%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20map_5409f38fb1874a85aa381438d8e5d24f.fitBounds%28%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%5B%5B42.36487823093963%2C%2012.884311360053138%5D%2C%20%5B42.96729276994153%2C%2013.611913497175467%5D%5D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7B%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%29%3B%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20%20%20%20%20function%20geo_json_d6aba2b5fea54f0d9d8ab1008131a42b_styler%28feature%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20switch%28feature.id%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20default%3A%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20%7B%22fillOpacity%22%3A%200.5%2C%20%22weight%22%3A%202%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20function%20geo_json_d6aba2b5fea54f0d9d8ab1008131a42b_highlighter%28feature%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20switch%28feature.id%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20default%3A%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20%7B%22fillOpacity%22%3A%200.75%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20function%20geo_json_d6aba2b5fea54f0d9d8ab1008131a42b_pointToLayer%28feature%2C%20latlng%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20var%20opts%20%3D%20%7B%22bubblingMouseEvents%22%3A%20true%2C%20%22color%22%3A%20%22%233388ff%22%2C%20%22dashArray%22%3A%20null%2C%20%22dashOffset%22%3A%20null%2C%20%22fill%22%3A%20true%2C%20%22fillColor%22%3A%20%22%233388ff%22%2C%20%22fillOpacity%22%3A%200.2%2C%20%22fillRule%22%3A%20%22evenodd%22%2C%20%22lineCap%22%3A%20%22round%22%2C%20%22lineJoin%22%3A%20%22round%22%2C%20%22opacity%22%3A%201.0%2C%20%22radius%22%3A%202%2C%20%22stroke%22%3A%20true%2C%20%22weight%22%3A%203%7D%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20let%20style%20%3D%20geo_json_d6aba2b5fea54f0d9d8ab1008131a42b_styler%28feature%29%0A%20%20%20%20%20%20%20%20%20%20%20%20Object.assign%28opts%2C%20style%29%0A%20%20%20%20%20%20%20%20%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20return%20new%20L.CircleMarker%28latlng%2C%20opts%29%0A%20%20%20%20%20%20%20%20%7D%0A%0A%20%20%20%20%20%20%20%20function%20geo_json_d6aba2b5fea54f0d9d8ab1008131a42b_onEachFeature%28feature%2C%20layer%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20layer.on%28%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20mouseout%3A%20function%28e%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20if%28typeof%20e.target.setStyle%20%3D%3D%3D%20%22function%22%29%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20geo_json_d6aba2b5fea54f0d9d8ab1008131a42b.resetStyle%28e.target%29%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20mouseover%3A%20function%28e%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20if%28typeof%20e.target.setStyle%20%3D%3D%3D%20%22function%22%29%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20const%20highlightStyle%20%3D%20geo_json_d6aba2b5fea54f0d9d8ab1008131a42b_highlighter%28e.target.feature%29%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20e.target.setStyle%28highlightStyle%29%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%29%3B%0A%20%20%20%20%20%20%20%20%7D%3B%0A%20%20%20%20%20%20%20%20var%20geo_json_d6aba2b5fea54f0d9d8ab1008131a42b%20%3D%20L.geoJson%28null%2C%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20onEachFeature%3A%20geo_json_d6aba2b5fea54f0d9d8ab1008131a42b_onEachFeature%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20style%3A%20geo_json_d6aba2b5fea54f0d9d8ab1008131a42b_styler%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20pointToLayer%3A%20geo_json_d6aba2b5fea54f0d9d8ab1008131a42b_pointToLayer%0A%20%20%20%20%20%20%20%20%7D%29%3B%0A%0A%20%20%20%20%20%20%20%20function%20geo_json_d6aba2b5fea54f0d9d8ab1008131a42b_add%20%28data%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20geo_json_d6aba2b5fea54f0d9d8ab1008131a42b%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20.addData%28data%29%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20.addTo%28map_5409f38fb1874a85aa381438d8e5d24f%29%3B%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20geo_json_d6aba2b5fea54f0d9d8ab1008131a42b_add%28%7B%22bbox%22%3A%20%5B12.884311360053138%2C%2042.36487823093963%2C%2013.611913497175467%2C%2042.96729276994153%5D%2C%20%22features%22%3A%20%5B%7B%22bbox%22%3A%20%5B12.945933261058768%2C%2042.62285235794043%2C%2013.162041525085991%2C%2042.784695839940824%5D%2C%20%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B12.983235536062184%2C%2042.78383686394083%5D%2C%20%5B12.983302574062202%2C%2042.7832627909408%5D%2C%20%5B12.984931640062396%2C%2042.783552667940846%5D%2C%20%5B12.98978680806294%2C%2042.78273507694081%5D%2C%20%5B12.995772345063681%2C%2042.77607938094084%5D%2C%20%5B12.997338506063814%2C%2042.77640279194081%5D%2C%20%5B12.999420029064064%2C%2042.777432955940796%5D%2C%20%5B13.003605124064508%2C%2042.777286483940905%5D%2C%20%5B13.005193034064707%2C%2042.77580377994082%5D%2C%20%5B13.007815868065062%2C%2042.77102093194081%5D%2C%20%5B13.011155452065474%2C%2042.76827023194086%5D%2C%20%5B13.011846043065532%2C%2042.766535294940795%5D%2C%20%5B13.013882279065816%2C%2042.76399680994078%5D%2C%20%5B13.015372104065973%2C%2042.76289556494085%5D%2C%20%5B13.01706178006618%2C%2042.76281820594085%5D%2C%20%5B13.019858170066524%2C%2042.76131534394073%5D%2C%20%5B13.019914283066509%2C%2042.76031845994081%5D%2C%20%5B13.022396480066837%2C%2042.75646760794083%5D%2C%20%5B13.022181825066838%2C%2042.755912426940824%5D%2C%20%5B13.023435684066964%2C%2042.75528757194082%5D%2C%20%5B13.026953219067405%2C%2042.75457399594079%5D%2C%20%5B13.028999148067673%2C%2042.753624058940794%5D%2C%20%5B13.03360528106824%2C%2042.74861765794081%5D%2C%20%5B13.035574286068437%2C%2042.750119327940745%5D%2C%20%5B13.042646448069304%2C%2042.752912751940826%5D%2C%20%5B13.042966151069333%2C%2042.753612739940806%5D%2C%20%5B13.042356076069252%2C%2042.75629940794083%5D%2C%20%5B13.042687473069241%2C%2042.757800312940866%5D%2C%20%5B13.043265716069323%2C%2042.75807248694081%5D%2C%20%5B13.044497495069486%2C%2042.75776781294076%5D%2C%20%5B13.045342430069564%2C%2042.75847622494083%5D%2C%20%5B13.044966493069511%2C%2042.758777643940846%5D%2C%20%5B13.045172571069555%2C%2042.75999485594085%5D%2C%20%5B13.044042374069406%2C%2042.75979623894082%5D%2C%20%5B13.043569605069347%2C%2042.76006506294085%5D%2C%20%5B13.044011224069395%2C%2042.76235439094082%5D%2C%20%5B13.04297952906928%2C%2042.76250792594088%5D%2C%20%5B13.043287572069262%2C%2042.763203822940774%5D%2C%20%5B13.042887392069238%2C%2042.76349708894087%5D%2C%20%5B13.04338413506928%2C%2042.764006238940816%5D%2C%20%5B13.042872770069241%2C%2042.76472661094089%5D%2C%20%5B13.043723490069285%2C%2042.76642523494083%5D%2C%20%5B13.044294447069348%2C%2042.77119050294087%5D%2C%20%5B13.043930295069268%2C%2042.77189666794085%5D%2C%20%5B13.048395388069821%2C%2042.772548981940815%5D%2C%20%5B13.051684236070244%2C%2042.770568714940815%5D%2C%20%5B13.053294038070428%2C%2042.768751435940864%5D%2C%20%5B13.0546569540706%2C%2042.76817189194087%5D%2C%20%5B13.054957642070674%2C%2042.76870595094086%5D%2C%20%5B13.056434092070832%2C%2042.76878863194086%5D%2C%20%5B13.05737679907092%2C%2042.769515991940814%5D%2C%20%5B13.059631194071237%2C%2042.76855360494084%5D%2C%20%5B13.060697597071375%2C%2042.766489906940905%5D%2C%20%5B13.061483758071482%2C%2042.765980284940895%5D%2C%20%5B13.062957211071677%2C%2042.76576136494088%5D%2C%20%5B13.064154395071819%2C%2042.76342287694084%5D%2C%20%5B13.065152311071971%2C%2042.76316230794088%5D%2C%20%5B13.0664024500721%2C%2042.76346449294086%5D%2C%20%5B13.070906302072705%2C%2042.76127839094084%5D%2C%20%5B13.070111300072602%2C%2042.759717545940845%5D%2C%20%5B13.070881509072695%2C%2042.75931647194081%5D%2C%20%5B13.071213860072742%2C%2042.759633271940835%5D%2C%20%5B13.071943376072863%2C%2042.759368695940786%5D%2C%20%5B13.071338968072782%2C%2042.758350294940854%5D%2C%20%5B13.071918765072855%2C%2042.75819008994084%5D%2C%20%5B13.072410454072928%2C%2042.7573037229408%5D%2C%20%5B13.077119683073558%2C%2042.754952497940856%5D%2C%20%5B13.077334194073558%2C%2042.75418403794081%5D%2C%20%5B13.078413178073708%2C%2042.753254189940805%5D%2C%20%5B13.07830180907369%2C%2042.75212820194083%5D%2C%20%5B13.078903993073764%2C%2042.75122886194082%5D%2C%20%5B13.07847690107377%2C%2042.74528366194087%5D%2C%20%5B13.077119896073622%2C%2042.74363936594078%5D%2C%20%5B13.074795696073357%2C%2042.743438604940806%5D%2C%20%5B13.074206707073294%2C%2042.74273479694073%5D%2C%20%5B13.073935177073293%2C%2042.73688306994077%5D%2C%20%5B13.07498670907348%2C%2042.732127662940734%5D%2C%20%5B13.0754618750736%2C%2042.72702363894078%5D%2C%20%5B13.077118412073746%2C%2042.729994326940755%5D%2C%20%5B13.078075057073907%2C%2042.72981165094071%5D%2C%20%5B13.078776517073972%2C%2042.72806692494078%5D%2C%20%5B13.079972866074133%2C%2042.72730394694079%5D%2C%20%5B13.080702195074247%2C%2042.7262289899407%5D%2C%20%5B13.085148292074804%2C%2042.7250348039408%5D%2C%20%5B13.089275078075394%2C%2042.722690389940745%5D%2C%20%5B13.092170192075788%2C%2042.72187101994074%5D%2C%20%5B13.094057361076043%2C%2042.71847210094075%5D%2C%20%5B13.09882408307667%2C%2042.71912066394079%5D%2C%20%5B13.099392400076724%2C%2042.71892021594076%5D%2C%20%5B13.102710422077191%2C%2042.720651466940794%5D%2C%20%5B13.105735861077592%2C%2042.71915180194078%5D%2C%20%5B13.108422911077904%2C%2042.72035181394072%5D%2C%20%5B13.109650446078081%2C%2042.7181288119407%5D%2C%20%5B13.113258360078616%2C%2042.71796306494078%5D%2C%20%5B13.11402793807869%2C%2042.717552721940706%5D%2C%20%5B13.115461347078933%2C%2042.71335944894077%5D%2C%20%5B13.115436378078957%2C%2042.7129821929408%5D%2C%20%5B13.114759279078834%2C%2042.712736450940774%5D%2C%20%5B13.114895183078847%2C%2042.71229938394076%5D%2C%20%5B13.117447319079231%2C%2042.70986205394073%5D%2C%20%5B13.119278733079495%2C%2042.70625764694073%5D%2C%20%5B13.120166688079637%2C%2042.703110382940665%5D%2C%20%5B13.122597906080031%2C%2042.69983545194067%5D%2C%20%5B13.122060319079976%2C%2042.69821616294068%5D%2C%20%5B13.122849930080118%2C%2042.69674260494072%5D%2C%20%5B13.124644270080372%2C%2042.696754425940725%5D%2C%20%5B13.127144670080675%2C%2042.697623097940706%5D%2C%20%5B13.128113336080823%2C%2042.69707942294072%5D%2C%20%5B13.131852199081305%2C%2042.69658423594069%5D%2C%20%5B13.132334569081436%2C%2042.69555389094072%5D%2C%20%5B13.13160224608132%2C%2042.69350950894066%5D%2C%20%5B13.131945676081376%2C%2042.692236584940694%5D%2C%20%5B13.133503337081594%2C%2042.69035254294069%5D%2C%20%5B13.13469735108179%2C%2042.69005727194068%5D%2C%20%5B13.138350584082353%2C%2042.68746711394074%5D%2C%20%5B13.138598796082363%2C%2042.68650824294072%5D%2C%20%5B13.137833467082288%2C%2042.68474870994069%5D%2C%20%5B13.140333921082698%2C%2042.68152942494071%5D%2C%20%5B13.14478615608333%2C%2042.67793773394065%5D%2C%20%5B13.14495358008336%2C%2042.67519454794058%5D%2C%20%5B13.151704320084391%2C%2042.67099703894065%5D%2C%20%5B13.15713251008521%2C%2042.6661224249406%5D%2C%20%5B13.160975709085823%2C%2042.66159335794064%5D%2C%20%5B13.162041525085991%2C%2042.65936667894068%5D%2C%20%5B13.161043773085863%2C%2042.65926793294063%5D%2C%20%5B13.160026008085692%2C%2042.65862968694055%5D%2C%20%5B13.159406046085635%2C%2042.65812552294057%5D%2C%20%5B13.15912870208563%2C%2042.656780556940596%5D%2C%20%5B13.157554461085384%2C%2042.655982451940574%5D%2C%20%5B13.15755387208538%2C%2042.65520815694063%5D%2C%20%5B13.158482776085549%2C%2042.65426500994061%5D%2C%20%5B13.154086513084911%2C%2042.65070176794056%5D%2C%20%5B13.152424794084698%2C%2042.650406472940546%5D%2C%20%5B13.150831579084468%2C%2042.64938837094061%5D%2C%20%5B13.146579498083867%2C%2042.64838115094058%5D%2C%20%5B13.145738402083772%2C%2042.64755179194053%5D%2C%20%5B13.146485267083879%2C%2042.64705652194061%5D%2C%20%5B13.146087987083813%2C%2042.646458674940575%5D%2C%20%5B13.14476928608362%2C%2042.64609230594061%5D%2C%20%5B13.135532379082232%2C%2042.65476863594055%5D%2C%20%5B13.13504786908219%2C%2042.65503825994059%5D%2C%20%5B13.132921208081921%2C%2042.65479100394062%5D%2C%20%5B13.12994404008142%2C%2042.65966602094065%5D%2C%20%5B13.124960672080737%2C%2042.65822521494066%5D%2C%20%5B13.120084484080108%2C%2042.658090367940595%5D%2C%20%5B13.1181815010798%2C%2042.657663707940586%5D%2C%20%5B13.116831979079628%2C%2042.657109047940544%5D%2C%20%5B13.11609844107958%2C%2042.655964968940566%5D%2C%20%5B13.117680492079845%2C%2042.6516402549406%5D%2C%20%5B13.117579389079827%2C%2042.65014477494061%5D%2C%20%5B13.116463208079697%2C%2042.64994635894061%5D%2C%20%5B13.111653741079047%2C%2042.646040685940505%5D%2C%20%5B13.108640000078626%2C%2042.644807440940525%5D%2C%20%5B13.109622183078795%2C%2042.64299841594056%5D%2C%20%5B13.1095222110788%2C%2042.64226820394054%5D%2C%20%5B13.101979397077796%2C%2042.63801911394048%5D%2C%20%5B13.097798408077301%2C%2042.63631872494047%5D%2C%20%5B13.086135522075772%2C%2042.634043883940464%5D%2C%20%5B13.081895556075246%2C%2042.63154817994045%5D%2C%20%5B13.078624423074823%2C%2042.63124170194048%5D%2C%20%5B13.076022754074499%2C%2042.629785801940514%5D%2C%20%5B13.072676364074107%2C%2042.62718136894046%5D%2C%20%5B13.064312267073047%2C%2042.62358025494047%5D%2C%20%5B13.058567460072346%2C%2042.62285235794043%5D%2C%20%5B13.049816259071239%2C%2042.624675191940426%5D%2C%20%5B13.046548694070836%2C%2042.62660060694048%5D%2C%20%5B13.04160054607019%2C%2042.627302283940494%5D%2C%20%5B13.037866238069737%2C%2042.62738015494045%5D%2C%20%5B13.034857239069389%2C%2042.628445248940395%5D%2C%20%5B13.032783470069129%2C%2042.63078734694043%5D%2C%20%5B13.032572407069084%2C%2042.63363999394047%5D%2C%20%5B13.03097796406881%2C%2042.637387760940456%5D%2C%20%5B13.029114958068584%2C%2042.637678499940485%5D%2C%20%5B13.027512110068429%2C%2042.63899998894046%5D%2C%20%5B13.025605135068199%2C%2042.63913015094045%5D%2C%20%5B13.023371965067938%2C%2042.638776545940416%5D%2C%20%5B13.019249813067443%2C%2042.63494178894047%5D%2C%20%5B13.009027804066264%2C%2042.63728588294052%5D%2C%20%5B13.002500532065454%2C%2042.640395711940435%5D%2C%20%5B13.001890687065382%2C%2042.642294349940485%5D%2C%20%5B13.004227564065662%2C%2042.64283832694048%5D%2C%20%5B13.004713035065683%2C%2042.64519834094042%5D%2C%20%5B13.003762341065556%2C%2042.6489817179405%5D%2C%20%5B13.004994083065657%2C%2042.64937077594048%5D%2C%20%5B13.004468888065622%2C%2042.65015449094045%5D%2C%20%5B12.999625569065014%2C%2042.652466874940494%5D%2C%20%5B12.999628760065013%2C%2042.65577116694055%5D%2C%20%5B12.997773829064736%2C%2042.660369444940535%5D%2C%20%5B12.992454747064157%2C%2042.662531586940496%5D%2C%20%5B12.985976873063422%2C%2042.664369218940486%5D%2C%20%5B12.983480034063138%2C%2042.6654241219405%5D%2C%20%5B12.979175654062635%2C%2042.66839666294057%5D%2C%20%5B12.97334547706194%2C%2042.67394763694051%5D%2C%20%5B12.96420634806093%2C%2042.67539926294052%5D%2C%20%5B12.961822059060637%2C%2042.677408702940575%5D%2C%20%5B12.959655626060444%2C%2042.67760528194053%5D%2C%20%5B12.959114335060347%2C%2042.67698474794057%5D%2C%20%5B12.958697602060338%2C%2042.67703969094056%5D%2C%20%5B12.957150349060162%2C%2042.67807465494052%5D%2C%20%5B12.956012000059996%2C%2042.68003635694058%5D%2C%20%5B12.954168536059726%2C%2042.69121534594056%5D%2C%20%5B12.952023791059482%2C%2042.69295068794058%5D%2C%20%5B12.95071443405936%2C%2042.69490474094064%5D%2C%20%5B12.94905290005919%2C%2042.69604708794063%5D%2C%20%5B12.948688334059103%2C%2042.697491285940615%5D%2C%20%5B12.946113484058833%2C%2042.70153284394057%5D%2C%20%5B12.945933261058768%2C%2042.702925651940625%5D%2C%20%5B12.95138414305928%2C%2042.71130008094061%5D%2C%20%5B12.95135374505929%2C%2042.71257517194064%5D%2C%20%5B12.949927261059134%2C%2042.7154786619407%5D%2C%20%5B12.953440796059446%2C%2042.72005273794066%5D%2C%20%5B12.954325348059523%2C%2042.72173287794067%5D%2C%20%5B12.95469887905958%2C%2042.72413748294067%5D%2C%20%5B12.955248919059597%2C%2042.72410945594068%5D%2C%20%5B12.955157670059595%2C%2042.725026498940665%5D%2C%20%5B12.955948144059665%2C%2042.725886030940714%5D%2C%20%5B12.955893057059676%2C%2042.727922795940685%5D%2C%20%5B12.955453322059588%2C%2042.727933504940665%5D%2C%20%5B12.956516396059689%2C%2042.72866205194068%5D%2C%20%5B12.956686678059716%2C%2042.729430486940736%5D%2C%20%5B12.956175266059676%2C%2042.72975430994065%5D%2C%20%5B12.956188915059645%2C%2042.73156360194067%5D%2C%20%5B12.95792840805978%2C%2042.73847684694072%5D%2C%20%5B12.959308408059943%2C%2042.74072503894073%5D%2C%20%5B12.959347286059927%2C%2042.74440173994072%5D%2C%20%5B12.958359436059789%2C%2042.74768630694073%5D%2C%20%5B12.957541911059703%2C%2042.74802523494075%5D%2C%20%5B12.954684177059368%2C%2042.7479800449408%5D%2C%20%5B12.952116105059101%2C%2042.750135200940704%5D%2C%20%5B12.95049185905893%2C%2042.75063701494074%5D%2C%20%5B12.948800618058744%2C%2042.75187492894073%5D%2C%20%5B12.950401906058888%2C%2042.75743347894077%5D%2C%20%5B12.950234470058838%2C%2042.758438682940756%5D%2C%20%5B12.949042523058711%2C%2042.758993070940804%5D%2C%20%5B12.949015277058704%2C%2042.75945320394076%5D%2C%20%5B12.947590206058578%2C%2042.76042529994081%5D%2C%20%5B12.955763204059398%2C%2042.75974217594072%5D%2C%20%5B12.959855991059825%2C%2042.75905128094079%5D%2C%20%5B12.960033436059867%2C%2042.75959436594074%5D%2C%20%5B12.958116534059657%2C%2042.76030000594073%5D%2C%20%5B12.95701684905954%2C%2042.761774168940754%5D%2C%20%5B12.955404753059332%2C%2042.76248271994077%5D%2C%20%5B12.95716761505952%2C%2042.765361455940756%5D%2C%20%5B12.959659652059788%2C%2042.766868850940725%5D%2C%20%5B12.961634921059972%2C%2042.76514823894077%5D%2C%20%5B12.963552785060187%2C%2042.76445601194076%5D%2C%20%5B12.964771722060362%2C%2042.76304517994081%5D%2C%20%5B12.965783069060459%2C%2042.7627850129408%5D%2C%20%5B12.966024773060491%2C%2042.76435678994076%5D%2C%20%5B12.965311810060387%2C%2042.76595267394082%5D%2C%20%5B12.965429788060367%2C%2042.76721361094077%5D%2C%20%5B12.96423640806029%2C%2042.768200387940816%5D%2C%20%5B12.963099955060134%2C%2042.77041419194083%5D%2C%20%5B12.96327217506011%2C%2042.77113752694087%5D%2C%20%5B12.964487270060257%2C%2042.772103822940856%5D%2C%20%5B12.963695085060142%2C%2042.7727840609408%5D%2C%20%5B12.963340753060123%2C%2042.77388579794078%5D%2C%20%5B12.963648148060122%2C%2042.77794483994084%5D%2C%20%5B12.964959317060245%2C%2042.7823247229408%5D%2C%20%5B12.96655633406039%2C%2042.784695839940824%5D%2C%20%5B12.980640397061906%2C%2042.7843144059409%5D%2C%20%5B12.983235536062184%2C%2042.78383686394083%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%220%22%2C%20%22properties%22%3A%20%7B%22COD_COM%22%3A%20%22007%22%2C%20%22COD_PROV%22%3A%20%22054%22%2C%20%22COD_REG%22%3A%20%2210%22%2C%20%22COMUNE%22%3A%20%22Cascia%22%2C%20%22GlobalID%22%3A%20%22cf1d46cf-b6be-4d5d-be6d-11dcf31d56b4%22%2C%20%22PROVINCIA%22%3A%20%22PG%22%2C%20%22Sum_ABITNO%22%3A%2014.0%2C%20%22Sum_ABIT_R%22%3A%201197.0%2C%20%22Sum_ABIT_V%22%3A%201303.0%2C%20%22Sum_ALBERG%22%3A%2050.0%2C%20%22Sum_ALTRIA%22%3A%208.0%2C%20%22Sum_EDIF_A%22%3A%201262.0%2C%20%22Sum_EDIF_T%22%3A%201529.0%2C%20%22Sum_EDIF_U%22%3A%201380.0%2C%20%22Sum_FAMRES%22%3A%201216.0%2C%20%22Sum_RES_0_%22%3A%20141.0%2C%20%22Sum_RES_10%22%3A%20190.0%2C%20%22Sum_RES_15%22%3A%20184.0%2C%20%22Sum_RES_20%22%3A%20155.0%2C%20%22Sum_RES_25%22%3A%20183.0%2C%20%22Sum_RES_30%22%3A%20244.0%2C%20%22Sum_RES_35%22%3A%20256.0%2C%20%22Sum_RES_40%22%3A%20275.0%2C%20%22Sum_RES_45%22%3A%20213.0%2C%20%22Sum_RES_50%22%3A%20179.0%2C%20%22Sum_RES_55%22%3A%20121.0%2C%20%22Sum_RES_5_%22%3A%20184.0%2C%20%22Sum_RES_60%22%3A%20161.0%2C%20%22Sum_RES_65%22%3A%20164.0%2C%20%22Sum_RES_70%22%3A%20203.0%2C%20%22Sum_RES_PI%22%3A%20407.0%2C%20%22Sum_RES_TO%22%3A%203260.0%2C%20%22Sum_STRAN_%22%3A%2066.0%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22bbox%22%3A%20%5B12.884311360053138%2C%2042.60289218094034%2C%2013.019249813067443%2C%2042.68183464394057%5D%2C%20%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B12.932244468057544%2C%2042.680176212940566%5D%2C%20%5B12.935174978057844%2C%2042.679769287940545%5D%2C%20%5B12.93558010905786%2C%2042.68043513894058%5D%2C%20%5B12.937620749058095%2C%2042.68074306194056%5D%2C%20%5B12.93899736205823%2C%2042.68045255394059%5D%2C%20%5B12.940693598058388%2C%2042.67895800694053%5D%2C%20%5B12.94191044505854%2C%2042.67906013394052%5D%2C%20%5B12.949042445059321%2C%2042.67708985994048%5D%2C%20%5B12.950389572059425%2C%2042.67732245594052%5D%2C%20%5B12.956012000059996%2C%2042.68003635694058%5D%2C%20%5B12.957150349060162%2C%2042.67807465494052%5D%2C%20%5B12.958697602060338%2C%2042.67703969094056%5D%2C%20%5B12.959114335060347%2C%2042.67698474794057%5D%2C%20%5B12.959655626060444%2C%2042.67760528194053%5D%2C%20%5B12.961822059060637%2C%2042.677408702940575%5D%2C%20%5B12.96420634806093%2C%2042.67539926294052%5D%2C%20%5B12.97334547706194%2C%2042.67394763694051%5D%2C%20%5B12.979175654062635%2C%2042.66839666294057%5D%2C%20%5B12.983480034063138%2C%2042.6654241219405%5D%2C%20%5B12.985976873063422%2C%2042.664369218940486%5D%2C%20%5B12.992454747064157%2C%2042.662531586940496%5D%2C%20%5B12.997773829064736%2C%2042.660369444940535%5D%2C%20%5B12.999628760065013%2C%2042.65577116694055%5D%2C%20%5B12.999625569065014%2C%2042.652466874940494%5D%2C%20%5B13.004468888065622%2C%2042.65015449094045%5D%2C%20%5B13.004994083065657%2C%2042.64937077594048%5D%2C%20%5B13.003762341065556%2C%2042.6489817179405%5D%2C%20%5B13.004713035065683%2C%2042.64519834094042%5D%2C%20%5B13.004227564065662%2C%2042.64283832694048%5D%2C%20%5B13.001890687065382%2C%2042.642294349940485%5D%2C%20%5B13.002500532065454%2C%2042.640395711940435%5D%2C%20%5B13.009027804066264%2C%2042.63728588294052%5D%2C%20%5B13.019249813067443%2C%2042.63494178894047%5D%2C%20%5B13.017249365067226%2C%2042.633004214940414%5D%2C%20%5B13.015332704067015%2C%2042.63192803194048%5D%2C%20%5B13.014272086066905%2C%2042.630380586940475%5D%2C%20%5B13.014597369066959%2C%2042.62859091294038%5D%2C%20%5B13.013917180066896%2C%2042.62811057094038%5D%2C%20%5B13.013592073066839%2C%2042.626847936940436%5D%2C%20%5B13.01214984406671%2C%2042.62578656194043%5D%2C%20%5B13.014188607066945%2C%2042.62342355294048%5D%2C%20%5B13.0150615620671%2C%2042.62088534494039%5D%2C%20%5B13.011218990066675%2C%2042.61711699894044%5D%2C%20%5B13.004316388065863%2C%2042.61524744994035%5D%2C%20%5B12.998676199065201%2C%2042.61510711594038%5D%2C%20%5B12.99767017806512%2C%2042.6153043499404%5D%2C%20%5B12.99517282606479%2C%2042.616746684940395%5D%2C%20%5B12.99240970606447%2C%2042.61738339194039%5D%2C%20%5B12.99115351306438%2C%2042.61587406394033%5D%2C%20%5B12.98580789506374%2C%2042.615646281940435%5D%2C%20%5B12.979445528063003%2C%2042.61675018394042%5D%2C%20%5B12.970514289062033%2C%2042.61731265794043%5D%2C%20%5B12.962675182061158%2C%2042.618782011940375%5D%2C%20%5B12.952501527060047%2C%2042.61893591694039%5D%2C%20%5B12.948771157059683%2C%2042.61815535994042%5D%2C%20%5B12.947528262059544%2C%2042.6174059149404%5D%2C%20%5B12.944496352059215%2C%2042.61692074994038%5D%2C%20%5B12.940950484058858%2C%2042.61572839794038%5D%2C%20%5B12.938292120058613%2C%2042.6153652549403%5D%2C%20%5B12.93682027605843%2C%2042.615645518940354%5D%2C%20%5B12.934380331058193%2C%2042.61274016494032%5D%2C%20%5B12.934420804058242%2C%2042.61004660194039%5D%2C%20%5B12.933778869058147%2C%2042.6073314869403%5D%2C%20%5B12.932065879058001%2C%2042.60567963494038%5D%2C%20%5B12.931537246057971%2C%2042.603599889940334%5D%2C%20%5B12.930122285057802%2C%2042.60289218094034%5D%2C%20%5B12.925900098057381%2C%2042.60413113794039%5D%2C%20%5B12.922575313057028%2C%2042.604051606940345%5D%2C%20%5B12.920639170056864%2C%2042.60323109294032%5D%2C%20%5B12.91874798405665%2C%2042.6032958929403%5D%2C%20%5B12.91857104205662%2C%2042.60639930894037%5D%2C%20%5B12.91644363905641%2C%2042.60952449794037%5D%2C%20%5B12.913468169056083%2C%2042.60954079094031%5D%2C%20%5B12.910986356055846%2C%2042.610845701940356%5D%2C%20%5B12.907145704055447%2C%2042.61196287594036%5D%2C%20%5B12.903078221055011%2C%2042.61219627094032%5D%2C%20%5B12.897185337054404%2C%2042.616322901940286%5D%2C%20%5B12.89635213405431%2C%2042.61645032894032%5D%2C%20%5B12.891264113053863%2C%2042.61691614594032%5D%2C%20%5B12.887683239053482%2C%2042.618973748940356%5D%2C%20%5B12.886754651053362%2C%2042.619748128940394%5D%2C%20%5B12.884311360053138%2C%2042.62491835294034%5D%2C%20%5B12.88492328405318%2C%2042.62737815994037%5D%2C%20%5B12.886494216053315%2C%2042.62924263894039%5D%2C%20%5B12.89206494905385%2C%2042.63148879794044%5D%2C%20%5B12.895327656054128%2C%2042.6341509509404%5D%2C%20%5B12.892647085053882%2C%2042.63407562994043%5D%2C%20%5B12.891713561053766%2C%2042.637533390940376%5D%2C%20%5B12.890233034053596%2C%2042.63878577794042%5D%2C%20%5B12.889528943053527%2C%2042.64085361394042%5D%2C%20%5B12.889359315053502%2C%2042.64585656294044%5D%2C%20%5B12.892010152053722%2C%2042.648634147940435%5D%2C%20%5B12.89182995605365%2C%2042.65035102394038%5D%2C%20%5B12.889799877053497%2C%2042.65268456594045%5D%2C%20%5B12.889226931053392%2C%2042.65409515394049%5D%2C%20%5B12.891884092053596%2C%2042.663913582940495%5D%2C%20%5B12.897894409054164%2C%2042.66622100594052%5D%2C%20%5B12.906632631054967%2C%2042.673071970940555%5D%2C%20%5B12.913202319055632%2C%2042.6722306179405%5D%2C%20%5B12.91941354005625%2C%2042.6726031909405%5D%2C%20%5B12.923337486056635%2C%2042.67241463194053%5D%2C%20%5B12.922241611056535%2C%2042.67489677094052%5D%2C%20%5B12.923392061056653%2C%2042.677009237940496%5D%2C%20%5B12.923412370056596%2C%2042.68078116294053%5D%2C%20%5B12.927477709057026%2C%2042.68183464394057%5D%2C%20%5B12.932244468057544%2C%2042.680176212940566%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%221%22%2C%20%22properties%22%3A%20%7B%22COD_COM%22%3A%20%22031%22%2C%20%22COD_PROV%22%3A%20%22054%22%2C%20%22COD_REG%22%3A%20%2210%22%2C%20%22COMUNE%22%3A%20%22Monteleone%20di%20Spoleto%22%2C%20%22GlobalID%22%3A%20%22607295fa-a049-4b08-b442-0a27d78e7878%22%2C%20%22PROVINCIA%22%3A%20%22PG%22%2C%20%22Sum_ABITNO%22%3A%200.0%2C%20%22Sum_ABIT_R%22%3A%20292.0%2C%20%22Sum_ABIT_V%22%3A%20539.0%2C%20%22Sum_ALBERG%22%3A%2026.0%2C%20%22Sum_ALTRIA%22%3A%200.0%2C%20%22Sum_EDIF_A%22%3A%20721.0%2C%20%22Sum_EDIF_T%22%3A%20988.0%2C%20%22Sum_EDIF_U%22%3A%20867.0%2C%20%22Sum_FAMRES%22%3A%20295.0%2C%20%22Sum_RES_0_%22%3A%2035.0%2C%20%22Sum_RES_10%22%3A%2036.0%2C%20%22Sum_RES_15%22%3A%2035.0%2C%20%22Sum_RES_20%22%3A%2041.0%2C%20%22Sum_RES_25%22%3A%2035.0%2C%20%22Sum_RES_30%22%3A%2052.0%2C%20%22Sum_RES_35%22%3A%2053.0%2C%20%22Sum_RES_40%22%3A%2047.0%2C%20%22Sum_RES_45%22%3A%2042.0%2C%20%22Sum_RES_50%22%3A%2037.0%2C%20%22Sum_RES_55%22%3A%2031.0%2C%20%22Sum_RES_5_%22%3A%2036.0%2C%20%22Sum_RES_60%22%3A%2037.0%2C%20%22Sum_RES_65%22%3A%2041.0%2C%20%22Sum_RES_70%22%3A%2055.0%2C%20%22Sum_RES_PI%22%3A%2068.0%2C%20%22Sum_RES_TO%22%3A%20681.0%2C%20%22Sum_STRAN_%22%3A%2011.0%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22bbox%22%3A%20%5B12.948060068058199%2C%2042.65426500994061%2C%2013.264160102100172%2C%2042.888475568941196%5D%2C%20%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B13.121888941078273%2C%2042.87368924994115%5D%2C%20%5B13.139091052080772%2C%2042.85777957294112%5D%2C%20%5B13.144356114081575%2C%2042.84708615294115%5D%2C%20%5B13.147254966081993%2C%2042.844685103941146%5D%2C%20%5B13.14996130708242%2C%2042.84133209394113%5D%2C%20%5B13.151407381082631%2C%2042.84084290994114%5D%2C%20%5B13.154398090083074%2C%2042.840684728941085%5D%2C%20%5B13.154553982083112%2C%2042.83463318194116%5D%2C%20%5B13.160862340084043%2C%2042.83190973594112%5D%2C%20%5B13.162621454084329%2C%2042.83199874794111%5D%2C%20%5B13.164475919084532%2C%2042.83342578494114%5D%2C%20%5B13.168129554085084%2C%2042.83358072394108%5D%2C%20%5B13.17051016808543%2C%2042.834178155941146%5D%2C%20%5B13.17199256208563%2C%2042.83497038994113%5D%2C%20%5B13.17470436608602%2C%2042.83509196394118%5D%2C%20%5B13.178458751086533%2C%2042.83599018294108%5D%2C%20%5B13.18107371308689%2C%2042.83692996294107%5D%2C%20%5B13.182160705087059%2C%2042.837831063941145%5D%2C%20%5B13.1849851810875%2C%2042.83624660294118%5D%2C%20%5B13.18628265208768%2C%2042.83497911294111%5D%2C%20%5B13.187556805087874%2C%2042.83448676494111%5D%2C%20%5B13.189764193088235%2C%2042.83559433594111%5D%2C%20%5B13.192811012088642%2C%2042.83640547094116%5D%2C%20%5B13.19421712008885%2C%2042.83897840594111%5D%2C%20%5B13.196046599089104%2C%2042.839176860941144%5D%2C%20%5B13.197890123089339%2C%2042.84058574194117%5D%2C%20%5B13.200955601089845%2C%2042.841215900941144%5D%2C%20%5B13.201950818089957%2C%2042.841908601941185%5D%2C%20%5B13.202864362090121%2C%2042.84079009094114%5D%2C%20%5B13.205059738090432%2C%2042.84018263694114%5D%2C%20%5B13.206780200090721%2C%2042.840362420941176%5D%2C%20%5B13.209585064091126%2C%2042.841533145941156%5D%2C%20%5B13.212247131091546%2C%2042.84172768194112%5D%2C%20%5B13.217147988092249%2C%2042.8462062649412%5D%2C%20%5B13.219716760092611%2C%2042.8458143389412%5D%2C%20%5B13.221643610092913%2C%2042.84690913394124%5D%2C%20%5B13.22205988709297%2C%2042.84775358994119%5D%2C%20%5B13.22099468509276%2C%2042.85075958094115%5D%2C%20%5B13.224360095093276%2C%2042.85244493894121%5D%2C%20%5B13.227471453093765%2C%2042.85281608394117%5D%2C%20%5B13.231282765094331%2C%2042.85434068594124%5D%2C%20%5B13.232124422094476%2C%2042.855732069941254%5D%2C%20%5B13.232499385094446%2C%2042.85886489494126%5D%2C%20%5B13.233723342094665%2C%2042.86015207394123%5D%2C%20%5B13.232664771094473%2C%2042.8634055179412%5D%2C%20%5B13.234886123094789%2C%2042.86530398894121%5D%2C%20%5B13.235270710094808%2C%2042.867702662941255%5D%2C%20%5B13.238678923095383%2C%2042.86452413794125%5D%2C%20%5B13.240856405095755%2C%2042.86111658094128%5D%2C%20%5B13.24350897809624%2C%2042.85867272194124%5D%2C%20%5B13.241972502096015%2C%2042.855875664941266%5D%2C%20%5B13.243411457096249%2C%2042.852612514941185%5D%2C%20%5B13.24643449509679%2C%2042.85131627794127%5D%2C%20%5B13.24772052409698%2C%2042.84998549594119%5D%2C%20%5B13.248091338097055%2C%2042.848558170941196%5D%2C%20%5B13.247678428097037%2C%2042.84757863494117%5D%2C%20%5B13.24977090009741%2C%2042.841522550941185%5D%2C%20%5B13.249677981097458%2C%2042.83613743394111%5D%2C%20%5B13.253277696098115%2C%2042.830182791941205%5D%2C%20%5B13.253398388098162%2C%2042.82743224294116%5D%2C%20%5B13.254880333098457%2C%2042.82563491294116%5D%2C%20%5B13.254894038098497%2C%2042.82384271294117%5D%2C%20%5B13.255816212098662%2C%2042.82149898894121%5D%2C%20%5B13.257325419098905%2C%2042.819155902941105%5D%2C%20%5B13.26242499809982%2C%2042.81616126094114%5D%2C%20%5B13.263136153099937%2C%2042.81476621094117%5D%2C%20%5B13.264160102100172%2C%2042.80877672094112%5D%2C%20%5B13.261345031099777%2C%2042.80406034694114%5D%2C%20%5B13.258456999099357%2C%2042.80151195394111%5D%2C%20%5B13.258448509099331%2C%2042.800053705941046%5D%2C%20%5B13.259414947099502%2C%2042.79829352994106%5D%2C%20%5B13.251479210098271%2C%2042.79437983994109%5D%2C%20%5B13.253850586098666%2C%2042.792670985941115%5D%2C%20%5B13.256117186099026%2C%2042.792064409941034%5D%2C%20%5B13.259534031099626%2C%2042.78969526494104%5D%2C%20%5B13.258131232099387%2C%2042.7895944709411%5D%2C%20%5B13.255496973098953%2C%2042.79064244094109%5D%2C%20%5B13.253213305098628%2C%2042.79054286494112%5D%2C%20%5B13.251891395098404%2C%2042.79017338194107%5D%2C%20%5B13.25101371209828%2C%2042.78920663794106%5D%2C%20%5B13.250628672098221%2C%2042.78593017794103%5D%2C%20%5B13.247999420097834%2C%2042.781206542941064%5D%2C%20%5B13.24728523509777%2C%2042.777046450940965%5D%2C%20%5B13.244343667097356%2C%2042.77427908594102%5D%2C%20%5B13.245496432097557%2C%2042.77069789794096%5D%2C%20%5B13.245603151097637%2C%2042.76778129694103%5D%2C%20%5B13.245016980097546%2C%2042.766916218940985%5D%2C%20%5B13.241343455096992%2C%2042.76281641794104%5D%2C%20%5B13.238537319096556%2C%2042.76094422294093%5D%2C%20%5B13.238595685096536%2C%2042.764493949940906%5D%2C%20%5B13.23667827109623%2C%2042.765717479940974%5D%2C%20%5B13.236792879096186%2C%2042.76986835894101%5D%2C%20%5B13.235031408095864%2C%2042.772931803941034%5D%2C%20%5B13.233767166095669%2C%2042.77310918494103%5D%2C%20%5B13.230533437095177%2C%2042.772049443941%5D%2C%20%5B13.229396213095008%2C%2042.77251018894103%5D%2C%20%5B13.228224997094776%2C%2042.77244997494098%5D%2C%20%5B13.225784171094412%2C%2042.770581960941%5D%2C%20%5B13.22467938609424%2C%2042.770330180940995%5D%2C%20%5B13.222613719093891%2C%2042.772468311941%5D%2C%20%5B13.22104285609366%2C%2042.772697393940966%5D%2C%20%5B13.219264011093404%2C%2042.772047281941%5D%2C%20%5B13.217626678093126%2C%2042.77204017594103%5D%2C%20%5B13.216493372092964%2C%2042.77149225194092%5D%2C%20%5B13.215454783092815%2C%2042.769774864941%5D%2C%20%5B13.21384337209257%2C%2042.76886639394097%5D%2C%20%5B13.21295422309246%2C%2042.767030929940944%5D%2C%20%5B13.208791325091855%2C%2042.763915980940936%5D%2C%20%5B13.203773963091098%2C%2042.763321783940924%5D%2C%20%5B13.20096452209065%2C%2042.76453249694097%5D%2C%20%5B13.199179601090428%2C%2042.76288289694092%5D%2C%20%5B13.19512447708984%2C%2042.76192433894096%5D%2C%20%5B13.19339605508956%2C%2042.75959280294093%5D%2C%20%5B13.19330806808959%2C%2042.75884423194097%5D%2C%20%5B13.19584395908995%2C%2042.758219976940914%5D%2C%20%5B13.196434167090066%2C%2042.756388595940884%5D%2C%20%5B13.198779045090422%2C%2042.75433071694097%5D%2C%20%5B13.199539162090591%2C%2042.751682757940905%5D%2C%20%5B13.199260895090584%2C%2042.74866327794087%5D%2C%20%5B13.197068163090247%2C%2042.746303829940864%5D%2C%20%5B13.195616520090061%2C%2042.74372356594085%5D%2C%20%5B13.192623360089673%2C%2042.742473831940856%5D%2C%20%5B13.191263623089458%2C%2042.7400747129409%5D%2C%20%5B13.188780168089101%2C%2042.73808139894088%5D%2C%20%5B13.187778909089008%2C%2042.73597972594085%5D%2C%20%5B13.187558654088953%2C%2042.73391246594081%5D%2C%20%5B13.187715356089006%2C%2042.73356458994084%5D%2C%20%5B13.188876060089147%2C%2042.73443143194086%5D%2C%20%5B13.189299747089247%2C%2042.73325444394087%5D%2C%20%5B13.18952220108928%2C%2042.730666762940885%5D%2C%20%5B13.187753682089053%2C%2042.7276208119408%5D%2C%20%5B13.186996767088983%2C%2042.72342134394084%5D%2C%20%5B13.187649367089138%2C%2042.72031821094084%5D%2C%20%5B13.189995597089537%2C%2042.715203739940755%5D%2C%20%5B13.19070696508969%2C%2042.71045077494073%5D%2C%20%5B13.189402121089516%2C%2042.708094628940785%5D%2C%20%5B13.186775807089147%2C%2042.70667822194079%5D%2C%20%5B13.183193918088676%2C%2042.70222648294074%5D%2C%20%5B13.182252858088576%2C%2042.699339232940716%5D%2C%20%5B13.180344213088299%2C%2042.696969046940715%5D%2C%20%5B13.177076184087827%2C%2042.69432436494076%5D%2C%20%5B13.173326204087392%2C%2042.68732140994071%5D%2C%20%5B13.173944332087487%2C%2042.68540358094076%5D%2C%20%5B13.175901896087804%2C%2042.683135218940684%5D%2C%20%5B13.180649231088497%2C%2042.68359200294072%5D%2C%20%5B13.18232418308874%2C%2042.68140139194072%5D%2C%20%5B13.18244950308876%2C%2042.68045142594069%5D%2C%20%5B13.181488924088665%2C%2042.67864081094067%5D%2C%20%5B13.179811135088439%2C%2042.677608209940715%5D%2C%20%5B13.173024111087456%2C%2042.674988435940634%5D%2C%20%5B13.17425589008768%2C%2042.66738040894062%5D%2C%20%5B13.173846970087633%2C%2042.667377327940635%5D%2C%20%5B13.174128713087704%2C%2042.66612003794065%5D%2C%20%5B13.173541739087643%2C%2042.6653941589407%5D%2C%20%5B13.1603278030858%2C%2042.65624173694065%5D%2C%20%5B13.158482776085549%2C%2042.65426500994061%5D%2C%20%5B13.15755387208538%2C%2042.65520815694063%5D%2C%20%5B13.157554461085384%2C%2042.655982451940574%5D%2C%20%5B13.15912870208563%2C%2042.656780556940596%5D%2C%20%5B13.159406046085635%2C%2042.65812552294057%5D%2C%20%5B13.160026008085692%2C%2042.65862968694055%5D%2C%20%5B13.161043773085863%2C%2042.65926793294063%5D%2C%20%5B13.162041525085991%2C%2042.65936667894068%5D%2C%20%5B13.160975709085823%2C%2042.66159335794064%5D%2C%20%5B13.15713251008521%2C%2042.6661224249406%5D%2C%20%5B13.151704320084391%2C%2042.67099703894065%5D%2C%20%5B13.14495358008336%2C%2042.67519454794058%5D%2C%20%5B13.14478615608333%2C%2042.67793773394065%5D%2C%20%5B13.140333921082698%2C%2042.68152942494071%5D%2C%20%5B13.137833467082288%2C%2042.68474870994069%5D%2C%20%5B13.138598796082363%2C%2042.68650824294072%5D%2C%20%5B13.138350584082353%2C%2042.68746711394074%5D%2C%20%5B13.13469735108179%2C%2042.69005727194068%5D%2C%20%5B13.133503337081594%2C%2042.69035254294069%5D%2C%20%5B13.131945676081376%2C%2042.692236584940694%5D%2C%20%5B13.13160224608132%2C%2042.69350950894066%5D%2C%20%5B13.132334569081436%2C%2042.69555389094072%5D%2C%20%5B13.131852199081305%2C%2042.69658423594069%5D%2C%20%5B13.128113336080823%2C%2042.69707942294072%5D%2C%20%5B13.127144670080675%2C%2042.697623097940706%5D%2C%20%5B13.124644270080372%2C%2042.696754425940725%5D%2C%20%5B13.122849930080118%2C%2042.69674260494072%5D%2C%20%5B13.122060319079976%2C%2042.69821616294068%5D%2C%20%5B13.122597906080031%2C%2042.69983545194067%5D%2C%20%5B13.120166688079637%2C%2042.703110382940665%5D%2C%20%5B13.119278733079495%2C%2042.70625764694073%5D%2C%20%5B13.117447319079231%2C%2042.70986205394073%5D%2C%20%5B13.114895183078847%2C%2042.71229938394076%5D%2C%20%5B13.114759279078834%2C%2042.712736450940774%5D%2C%20%5B13.115436378078957%2C%2042.7129821929408%5D%2C%20%5B13.115461347078933%2C%2042.71335944894077%5D%2C%20%5B13.11402793807869%2C%2042.717552721940706%5D%2C%20%5B13.113258360078616%2C%2042.71796306494078%5D%2C%20%5B13.109650446078081%2C%2042.7181288119407%5D%2C%20%5B13.108422911077904%2C%2042.72035181394072%5D%2C%20%5B13.105735861077592%2C%2042.71915180194078%5D%2C%20%5B13.102710422077191%2C%2042.720651466940794%5D%2C%20%5B13.099392400076724%2C%2042.71892021594076%5D%2C%20%5B13.09882408307667%2C%2042.71912066394079%5D%2C%20%5B13.094057361076043%2C%2042.71847210094075%5D%2C%20%5B13.092170192075788%2C%2042.72187101994074%5D%2C%20%5B13.089275078075394%2C%2042.722690389940745%5D%2C%20%5B13.085148292074804%2C%2042.7250348039408%5D%2C%20%5B13.080702195074247%2C%2042.7262289899407%5D%2C%20%5B13.079972866074133%2C%2042.72730394694079%5D%2C%20%5B13.078776517073972%2C%2042.72806692494078%5D%2C%20%5B13.078075057073907%2C%2042.72981165094071%5D%2C%20%5B13.077118412073746%2C%2042.729994326940755%5D%2C%20%5B13.0754618750736%2C%2042.72702363894078%5D%2C%20%5B13.07498670907348%2C%2042.732127662940734%5D%2C%20%5B13.073935177073293%2C%2042.73688306994077%5D%2C%20%5B13.074206707073294%2C%2042.74273479694073%5D%2C%20%5B13.074795696073357%2C%2042.743438604940806%5D%2C%20%5B13.077119896073622%2C%2042.74363936594078%5D%2C%20%5B13.07847690107377%2C%2042.74528366194087%5D%2C%20%5B13.078903993073764%2C%2042.75122886194082%5D%2C%20%5B13.07830180907369%2C%2042.75212820194083%5D%2C%20%5B13.078413178073708%2C%2042.753254189940805%5D%2C%20%5B13.077334194073558%2C%2042.75418403794081%5D%2C%20%5B13.077119683073558%2C%2042.754952497940856%5D%2C%20%5B13.072410454072928%2C%2042.7573037229408%5D%2C%20%5B13.071918765072855%2C%2042.75819008994084%5D%2C%20%5B13.071338968072782%2C%2042.758350294940854%5D%2C%20%5B13.071943376072863%2C%2042.759368695940786%5D%2C%20%5B13.071213860072742%2C%2042.759633271940835%5D%2C%20%5B13.070881509072695%2C%2042.75931647194081%5D%2C%20%5B13.070111300072602%2C%2042.759717545940845%5D%2C%20%5B13.070906302072705%2C%2042.76127839094084%5D%2C%20%5B13.0664024500721%2C%2042.76346449294086%5D%2C%20%5B13.065152311071971%2C%2042.76316230794088%5D%2C%20%5B13.064154395071819%2C%2042.76342287694084%5D%2C%20%5B13.062957211071677%2C%2042.76576136494088%5D%2C%20%5B13.061483758071482%2C%2042.765980284940895%5D%2C%20%5B13.060697597071375%2C%2042.766489906940905%5D%2C%20%5B13.059631194071237%2C%2042.76855360494084%5D%2C%20%5B13.05737679907092%2C%2042.769515991940814%5D%2C%20%5B13.056434092070832%2C%2042.76878863194086%5D%2C%20%5B13.054957642070674%2C%2042.76870595094086%5D%2C%20%5B13.0546569540706%2C%2042.76817189194087%5D%2C%20%5B13.053294038070428%2C%2042.768751435940864%5D%2C%20%5B13.051684236070244%2C%2042.770568714940815%5D%2C%20%5B13.048395388069821%2C%2042.772548981940815%5D%2C%20%5B13.043930295069268%2C%2042.77189666794085%5D%2C%20%5B13.044294447069348%2C%2042.77119050294087%5D%2C%20%5B13.043723490069285%2C%2042.76642523494083%5D%2C%20%5B13.042872770069241%2C%2042.76472661094089%5D%2C%20%5B13.04338413506928%2C%2042.764006238940816%5D%2C%20%5B13.042887392069238%2C%2042.76349708894087%5D%2C%20%5B13.043287572069262%2C%2042.763203822940774%5D%2C%20%5B13.04297952906928%2C%2042.76250792594088%5D%2C%20%5B13.044011224069395%2C%2042.76235439094082%5D%2C%20%5B13.043569605069347%2C%2042.76006506294085%5D%2C%20%5B13.044042374069406%2C%2042.75979623894082%5D%2C%20%5B13.045172571069555%2C%2042.75999485594085%5D%2C%20%5B13.044966493069511%2C%2042.758777643940846%5D%2C%20%5B13.045342430069564%2C%2042.75847622494083%5D%2C%20%5B13.044497495069486%2C%2042.75776781294076%5D%2C%20%5B13.043265716069323%2C%2042.75807248694081%5D%2C%20%5B13.042687473069241%2C%2042.757800312940866%5D%2C%20%5B13.042356076069252%2C%2042.75629940794083%5D%2C%20%5B13.042966151069333%2C%2042.753612739940806%5D%2C%20%5B13.042646448069304%2C%2042.752912751940826%5D%2C%20%5B13.035574286068437%2C%2042.750119327940745%5D%2C%20%5B13.03360528106824%2C%2042.74861765794081%5D%2C%20%5B13.028999148067673%2C%2042.753624058940794%5D%2C%20%5B13.026953219067405%2C%2042.75457399594079%5D%2C%20%5B13.023435684066964%2C%2042.75528757194082%5D%2C%20%5B13.022181825066838%2C%2042.755912426940824%5D%2C%20%5B13.022396480066837%2C%2042.75646760794083%5D%2C%20%5B13.019914283066509%2C%2042.76031845994081%5D%2C%20%5B13.019858170066524%2C%2042.76131534394073%5D%2C%20%5B13.01706178006618%2C%2042.76281820594085%5D%2C%20%5B13.015372104065973%2C%2042.76289556494085%5D%2C%20%5B13.013882279065816%2C%2042.76399680994078%5D%2C%20%5B13.011846043065532%2C%2042.766535294940795%5D%2C%20%5B13.011155452065474%2C%2042.76827023194086%5D%2C%20%5B13.007815868065062%2C%2042.77102093194081%5D%2C%20%5B13.005193034064707%2C%2042.77580377994082%5D%2C%20%5B13.003605124064508%2C%2042.777286483940905%5D%2C%20%5B12.999420029064064%2C%2042.777432955940796%5D%2C%20%5B12.997338506063814%2C%2042.77640279194081%5D%2C%20%5B12.995772345063681%2C%2042.77607938094084%5D%2C%20%5B12.98978680806294%2C%2042.78273507694081%5D%2C%20%5B12.984931640062396%2C%2042.783552667940846%5D%2C%20%5B12.983302574062202%2C%2042.7832627909408%5D%2C%20%5B12.983235536062184%2C%2042.78383686394083%5D%2C%20%5B12.980640397061906%2C%2042.7843144059409%5D%2C%20%5B12.96655633406039%2C%2042.784695839940824%5D%2C%20%5B12.966494573060393%2C%2042.78621511394082%5D%2C%20%5B12.96849372106058%2C%2042.78750080494082%5D%2C%20%5B12.968815783060593%2C%2042.79073097294088%5D%2C%20%5B12.968365234060562%2C%2042.79105724094082%5D%2C%20%5B12.961221466059753%2C%2042.790427102940846%5D%2C%20%5B12.957916163059407%2C%2042.79090622294089%5D%2C%20%5B12.956136836059182%2C%2042.79408310094085%5D%2C%20%5B12.957248863059327%2C%2042.79484144894082%5D%2C%20%5B12.952110240058738%2C%2042.80343769194085%5D%2C%20%5B12.95190891005865%2C%2042.80735227494085%5D%2C%20%5B12.950325085058463%2C%2042.81604159394091%5D%2C%20%5B12.948163393058222%2C%2042.81780446594091%5D%2C%20%5B12.948060068058199%2C%2042.81833025094095%5D%2C%20%5B12.949697196058338%2C%2042.81874639994096%5D%2C%20%5B12.951108499058435%2C%2042.826013162940924%5D%2C%20%5B12.95374957005873%2C%2042.82792518394096%5D%2C%20%5B12.956493256058971%2C%2042.8285685689409%5D%2C%20%5B12.956820610059044%2C%2042.829025432940924%5D%2C%20%5B12.956498819058945%2C%2042.83402912994095%5D%2C%20%5B12.959909176059265%2C%2042.84001106494101%5D%2C%20%5B12.966139675059912%2C%2042.84479663194103%5D%2C%20%5B12.967404938060037%2C%2042.84538748794101%5D%2C%20%5B12.967827922060078%2C%2042.84816845494103%5D%2C%20%5B12.96914798106017%2C%2042.850459086941015%5D%2C%20%5B12.96924958506017%2C%2042.85182862094098%5D%2C%20%5B12.97110757106036%2C%2042.85339376094106%5D%2C%20%5B12.972562249060534%2C%2042.85383391894101%5D%2C%20%5B12.97705870406102%2C%2042.85067934694102%5D%2C%20%5B12.979293096061264%2C%2042.849710225941074%5D%2C%20%5B12.9811580240615%2C%2042.84957326294099%5D%2C%20%5B12.982079831061606%2C%2042.84759185894102%5D%2C%20%5B12.98333608306172%2C%2042.846197539941016%5D%2C%20%5B12.985001227061971%2C%2042.83863493494099%5D%2C%20%5B12.983526632061848%2C%2042.83583666094101%5D%2C%20%5B12.979893620061453%2C%2042.83129928494095%5D%2C%20%5B12.980334214061514%2C%2042.83054113994101%5D%2C%20%5B12.983454635061866%2C%2042.828631713941%5D%2C%20%5B12.989364698062554%2C%2042.82823204794097%5D%2C%20%5B12.995420604063213%2C%2042.82708417394096%5D%2C%20%5B13.000481761063801%2C%2042.82455271394096%5D%2C%20%5B13.005758930064411%2C%2042.823287482940984%5D%2C%20%5B13.013848629065333%2C%2042.82010454994094%5D%2C%20%5B13.013590447065322%2C%2042.82053228594096%5D%2C%20%5B13.016405479065623%2C%2042.82277887794095%5D%2C%20%5B13.016984786065702%2C%2042.82302413194098%5D%2C%20%5B13.01937397406598%2C%2042.822521483940996%5D%2C%20%5B13.021768801066266%2C%2042.820681542940996%5D%2C%20%5B13.022954594066388%2C%2042.820342698940905%5D%2C%20%5B13.024651564066605%2C%2042.820436031941%5D%2C%20%5B13.031096955067378%2C%2042.816990126940944%5D%2C%20%5B13.034206639067746%2C%2042.81612412494096%5D%2C%20%5B13.037001763068108%2C%2042.81630906894093%5D%2C%20%5B13.039252163068355%2C%2042.815923452940915%5D%2C%20%5B13.043926186068934%2C%2042.81431762494092%5D%2C%20%5B13.046595353069218%2C%2042.81423667994101%5D%2C%20%5B13.053254905070055%2C%2042.81561240794099%5D%2C%20%5B13.053726917070094%2C%2042.81579375094098%5D%2C%20%5B13.054297073070117%2C%2042.818654698940925%5D%2C%20%5B13.057071677070473%2C%2042.81911898994096%5D%2C%20%5B13.058311212070658%2C%2042.81858879594093%5D%2C%20%5B13.058595382070676%2C%2042.818889332941005%5D%2C%20%5B13.06129585807098%2C%2042.818595345941%5D%2C%20%5B13.061160199071011%2C%2042.81824001994104%5D%2C%20%5B13.06175097207107%2C%2042.818133495940934%5D%2C%20%5B13.062042413071113%2C%2042.81859132994097%5D%2C%20%5B13.062637664071175%2C%2042.818331580941%5D%2C%20%5B13.063889846071312%2C%2042.818566185941016%5D%2C%20%5B13.064705775071415%2C%2042.81751076194103%5D%2C%20%5B13.068168757071867%2C%2042.81786476994097%5D%2C%20%5B13.069582236072062%2C%2042.817238226941015%5D%2C%20%5B13.069818961072068%2C%2042.81790957394099%5D%2C%20%5B13.070223955072125%2C%2042.817872645941016%5D%2C%20%5B13.070793684072198%2C%2042.81727162494097%5D%2C%20%5B13.074664978072663%2C%2042.81625584794098%5D%2C%20%5B13.074770508072714%2C%2042.81561733094096%5D%2C%20%5B13.077111704072989%2C%2042.815848981940945%5D%2C%20%5B13.081264366073516%2C%2042.81764558294103%5D%2C%20%5B13.084366857073926%2C%2042.81578807294101%5D%2C%20%5B13.085648591074058%2C%2042.815890830940965%5D%2C%20%5B13.088233110074416%2C%2042.81736505894103%5D%2C%20%5B13.089726704074561%2C%2042.817712294941%5D%2C%20%5B13.088455018074384%2C%2042.81966204894101%5D%2C%20%5B13.08624360907412%2C%2042.81918739694102%5D%2C%20%5B13.084082439073805%2C%2042.82145700994103%5D%2C%20%5B13.08088199707338%2C%2042.82641525394104%5D%2C%20%5B13.080684849073341%2C%2042.82885326894104%5D%2C%20%5B13.077110051072854%2C%2042.83388329394104%5D%2C%20%5B13.076532523072778%2C%2042.834529642941064%5D%2C%20%5B13.075734658072633%2C%2042.834616616941055%5D%2C%20%5B13.075410473072603%2C%2042.837950508941034%5D%2C%20%5B13.074855939072531%2C%2042.838087324941064%5D%2C%20%5B13.072820501072282%2C%2042.837448579941075%5D%2C%20%5B13.071070445072037%2C%2042.840986314941084%5D%2C%20%5B13.072099248072155%2C%2042.84216515794103%5D%2C%20%5B13.072773833072178%2C%2042.84404988994108%5D%2C%20%5B13.074912634072417%2C%2042.84649917694106%5D%2C%20%5B13.074906267072437%2C%2042.84749430194107%5D%2C%20%5B13.072394461072067%2C%2042.85112673594107%5D%2C%20%5B13.07246151507205%2C%2042.85330772094112%5D%2C%20%5B13.07379850307225%2C%2042.8555740049411%5D%2C%20%5B13.075492464072456%2C%2042.856999212941126%5D%2C%20%5B13.075124366072373%2C%2042.857809154941066%5D%2C%20%5B13.075911037072485%2C%2042.85774509194109%5D%2C%20%5B13.078284154072763%2C%2042.85852929194116%5D%2C%20%5B13.079166253072835%2C%2042.85972680594112%5D%2C%20%5B13.090663148074254%2C%2042.869516987941154%5D%2C%20%5B13.091344181074305%2C%2042.8704559939411%5D%2C%20%5B13.091786581074345%2C%2042.872794591941165%5D%2C%20%5B13.093542493074583%2C%2042.873015329941126%5D%2C%20%5B13.09344390907456%2C%2042.87343302494112%5D%2C%20%5B13.094667513074691%2C%2042.87307857394115%5D%2C%20%5B13.094512990074671%2C%2042.87371435894121%5D%2C%20%5B13.096098064074857%2C%2042.87432382894114%5D%2C%20%5B13.098950236075208%2C%2042.87832720394123%5D%2C%20%5B13.103259914075775%2C%2042.88113473594119%5D%2C%20%5B13.112697043076928%2C%2042.888475568941196%5D%2C%20%5B13.116136386077411%2C%2042.88127497094117%5D%2C%20%5B13.118907170077822%2C%2042.87834803794125%5D%2C%20%5B13.118987113077845%2C%2042.87691809494119%5D%2C%20%5B13.121305973078169%2C%2042.87556050094121%5D%2C%20%5B13.121888941078273%2C%2042.87368924994115%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%222%22%2C%20%22properties%22%3A%20%7B%22COD_COM%22%3A%20%22035%22%2C%20%22COD_PROV%22%3A%20%22054%22%2C%20%22COD_REG%22%3A%20%2210%22%2C%20%22COMUNE%22%3A%20%22Norcia%22%2C%20%22GlobalID%22%3A%20%22d8d0db96-9188-47ef-9bda-f87546b97eb9%22%2C%20%22PROVINCIA%22%3A%20%22PG%22%2C%20%22Sum_ABITNO%22%3A%2018.0%2C%20%22Sum_ABIT_R%22%3A%201908.0%2C%20%22Sum_ABIT_V%22%3A%201922.0%2C%20%22Sum_ALBERG%22%3A%2085.0%2C%20%22Sum_ALTRIA%22%3A%2092.0%2C%20%22Sum_EDIF_A%22%3A%202321.0%2C%20%22Sum_EDIF_T%22%3A%202959.0%2C%20%22Sum_EDIF_U%22%3A%202643.0%2C%20%22Sum_FAMRES%22%3A%202001.0%2C%20%22Sum_RES_0_%22%3A%20234.0%2C%20%22Sum_RES_10%22%3A%20252.0%2C%20%22Sum_RES_15%22%3A%20242.0%2C%20%22Sum_RES_20%22%3A%20250.0%2C%20%22Sum_RES_25%22%3A%20309.0%2C%20%22Sum_RES_30%22%3A%20384.0%2C%20%22Sum_RES_35%22%3A%20402.0%2C%20%22Sum_RES_40%22%3A%20334.0%2C%20%22Sum_RES_45%22%3A%20309.0%2C%20%22Sum_RES_50%22%3A%20287.0%2C%20%22Sum_RES_55%22%3A%20246.0%2C%20%22Sum_RES_5_%22%3A%20247.0%2C%20%22Sum_RES_60%22%3A%20278.0%2C%20%22Sum_RES_65%22%3A%20236.0%2C%20%22Sum_RES_70%22%3A%20287.0%2C%20%22Sum_RES_PI%22%3A%20575.0%2C%20%22Sum_RES_TO%22%3A%204872.0%2C%20%22Sum_STRAN_%22%3A%20276.0%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22bbox%22%3A%20%5B12.959586029059137%2C%2042.81423667994101%2C%2013.112697043076928%2C%2042.92022395894126%5D%2C%20%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B13.072644164071647%2C%2042.90366725594123%5D%2C%20%5B13.080746690072711%2C%2042.89953372394122%5D%2C%20%5B13.08421601207313%2C%2042.90062978194124%5D%2C%20%5B13.088588178073673%2C%2042.90103617894122%5D%2C%20%5B13.089120346073802%2C%2042.896506359941235%5D%2C%20%5B13.095178212074565%2C%2042.89584828594125%5D%2C%20%5B13.099141693075126%2C%2042.89361737794125%5D%2C%20%5B13.108097391076283%2C%2042.89031114894122%5D%2C%20%5B13.112254239076815%2C%2042.889342336941276%5D%2C%20%5B13.112697043076928%2C%2042.888475568941196%5D%2C%20%5B13.103259914075775%2C%2042.88113473594119%5D%2C%20%5B13.098950236075208%2C%2042.87832720394123%5D%2C%20%5B13.096098064074857%2C%2042.87432382894114%5D%2C%20%5B13.094512990074671%2C%2042.87371435894121%5D%2C%20%5B13.094667513074691%2C%2042.87307857394115%5D%2C%20%5B13.09344390907456%2C%2042.87343302494112%5D%2C%20%5B13.093542493074583%2C%2042.873015329941126%5D%2C%20%5B13.091786581074345%2C%2042.872794591941165%5D%2C%20%5B13.091344181074305%2C%2042.8704559939411%5D%2C%20%5B13.090663148074254%2C%2042.869516987941154%5D%2C%20%5B13.079166253072835%2C%2042.85972680594112%5D%2C%20%5B13.078284154072763%2C%2042.85852929194116%5D%2C%20%5B13.075911037072485%2C%2042.85774509194109%5D%2C%20%5B13.075124366072373%2C%2042.857809154941066%5D%2C%20%5B13.075492464072456%2C%2042.856999212941126%5D%2C%20%5B13.07379850307225%2C%2042.8555740049411%5D%2C%20%5B13.07246151507205%2C%2042.85330772094112%5D%2C%20%5B13.072394461072067%2C%2042.85112673594107%5D%2C%20%5B13.074906267072437%2C%2042.84749430194107%5D%2C%20%5B13.074912634072417%2C%2042.84649917694106%5D%2C%20%5B13.072773833072178%2C%2042.84404988994108%5D%2C%20%5B13.072099248072155%2C%2042.84216515794103%5D%2C%20%5B13.071070445072037%2C%2042.840986314941084%5D%2C%20%5B13.072820501072282%2C%2042.837448579941075%5D%2C%20%5B13.074855939072531%2C%2042.838087324941064%5D%2C%20%5B13.075410473072603%2C%2042.837950508941034%5D%2C%20%5B13.075734658072633%2C%2042.834616616941055%5D%2C%20%5B13.076532523072778%2C%2042.834529642941064%5D%2C%20%5B13.077110051072854%2C%2042.83388329394104%5D%2C%20%5B13.080684849073341%2C%2042.82885326894104%5D%2C%20%5B13.08088199707338%2C%2042.82641525394104%5D%2C%20%5B13.084082439073805%2C%2042.82145700994103%5D%2C%20%5B13.08624360907412%2C%2042.81918739694102%5D%2C%20%5B13.088455018074384%2C%2042.81966204894101%5D%2C%20%5B13.089726704074561%2C%2042.817712294941%5D%2C%20%5B13.088233110074416%2C%2042.81736505894103%5D%2C%20%5B13.085648591074058%2C%2042.815890830940965%5D%2C%20%5B13.084366857073926%2C%2042.81578807294101%5D%2C%20%5B13.081264366073516%2C%2042.81764558294103%5D%2C%20%5B13.077111704072989%2C%2042.815848981940945%5D%2C%20%5B13.074770508072714%2C%2042.81561733094096%5D%2C%20%5B13.074664978072663%2C%2042.81625584794098%5D%2C%20%5B13.070793684072198%2C%2042.81727162494097%5D%2C%20%5B13.070223955072125%2C%2042.817872645941016%5D%2C%20%5B13.069818961072068%2C%2042.81790957394099%5D%2C%20%5B13.069582236072062%2C%2042.817238226941015%5D%2C%20%5B13.068168757071867%2C%2042.81786476994097%5D%2C%20%5B13.064705775071415%2C%2042.81751076194103%5D%2C%20%5B13.063889846071312%2C%2042.818566185941016%5D%2C%20%5B13.062637664071175%2C%2042.818331580941%5D%2C%20%5B13.062042413071113%2C%2042.81859132994097%5D%2C%20%5B13.06175097207107%2C%2042.818133495940934%5D%2C%20%5B13.061160199071011%2C%2042.81824001994104%5D%2C%20%5B13.06129585807098%2C%2042.818595345941%5D%2C%20%5B13.058595382070676%2C%2042.818889332941005%5D%2C%20%5B13.058311212070658%2C%2042.81858879594093%5D%2C%20%5B13.057071677070473%2C%2042.81911898994096%5D%2C%20%5B13.054297073070117%2C%2042.818654698940925%5D%2C%20%5B13.053726917070094%2C%2042.81579375094098%5D%2C%20%5B13.053254905070055%2C%2042.81561240794099%5D%2C%20%5B13.046595353069218%2C%2042.81423667994101%5D%2C%20%5B13.043926186068934%2C%2042.81431762494092%5D%2C%20%5B13.039252163068355%2C%2042.815923452940915%5D%2C%20%5B13.037001763068108%2C%2042.81630906894093%5D%2C%20%5B13.034206639067746%2C%2042.81612412494096%5D%2C%20%5B13.031096955067378%2C%2042.816990126940944%5D%2C%20%5B13.024651564066605%2C%2042.820436031941%5D%2C%20%5B13.022954594066388%2C%2042.820342698940905%5D%2C%20%5B13.021768801066266%2C%2042.820681542940996%5D%2C%20%5B13.01937397406598%2C%2042.822521483940996%5D%2C%20%5B13.016984786065702%2C%2042.82302413194098%5D%2C%20%5B13.016405479065623%2C%2042.82277887794095%5D%2C%20%5B13.013590447065322%2C%2042.82053228594096%5D%2C%20%5B13.013848629065333%2C%2042.82010454994094%5D%2C%20%5B13.005758930064411%2C%2042.823287482940984%5D%2C%20%5B13.000481761063801%2C%2042.82455271394096%5D%2C%20%5B12.995420604063213%2C%2042.82708417394096%5D%2C%20%5B12.989364698062554%2C%2042.82823204794097%5D%2C%20%5B12.983454635061866%2C%2042.828631713941%5D%2C%20%5B12.980334214061514%2C%2042.83054113994101%5D%2C%20%5B12.979893620061453%2C%2042.83129928494095%5D%2C%20%5B12.983526632061848%2C%2042.83583666094101%5D%2C%20%5B12.985001227061971%2C%2042.83863493494099%5D%2C%20%5B12.98333608306172%2C%2042.846197539941016%5D%2C%20%5B12.982079831061606%2C%2042.84759185894102%5D%2C%20%5B12.9811580240615%2C%2042.84957326294099%5D%2C%20%5B12.979293096061264%2C%2042.849710225941074%5D%2C%20%5B12.97705870406102%2C%2042.85067934694102%5D%2C%20%5B12.972562249060534%2C%2042.85383391894101%5D%2C%20%5B12.97110757106036%2C%2042.85339376094106%5D%2C%20%5B12.96924958506017%2C%2042.85182862094098%5D%2C%20%5B12.968257093060108%2C%2042.85149842894099%5D%2C%20%5B12.968404162060112%2C%2042.850750518941034%5D%2C%20%5B12.966076111059856%2C%2042.85007049994109%5D%2C%20%5B12.96525665805976%2C%2042.84857280094098%5D%2C%20%5B12.96419808805965%2C%2042.84837991994105%5D%2C%20%5B12.961962016059466%2C%2042.846422602941075%5D%2C%20%5B12.961100260059373%2C%2042.846209365940986%5D%2C%20%5B12.960345181059246%2C%2042.84645611794098%5D%2C%20%5B12.959959139059256%2C%2042.84741487994099%5D%2C%20%5B12.960205166059191%2C%2042.85295713594103%5D%2C%20%5B12.959586029059137%2C%2042.858007145941%5D%2C%20%5B12.960158847059148%2C%2042.8610530599411%5D%2C%20%5B12.961650530059268%2C%2042.864076132940994%5D%2C%20%5B12.96265411705937%2C%2042.86481566494102%5D%2C%20%5B12.967189856059854%2C%2042.866130443941124%5D%2C%20%5B12.968901304060005%2C%2042.86768269294104%5D%2C%20%5B12.970219695060186%2C%2042.868163625941115%5D%2C%20%5B12.975824197060762%2C%2042.86823444194108%5D%2C%20%5B12.976164576060805%2C%2042.86981625194113%5D%2C%20%5B12.979050995061108%2C%2042.86951773294107%5D%2C%20%5B12.980731860061265%2C%2042.86979234594103%5D%2C%20%5B12.980586216061258%2C%2042.871238003941095%5D%2C%20%5B12.982213637061452%2C%2042.871275858941104%5D%2C%20%5B12.984413957061683%2C%2042.87062294994111%5D%2C%20%5B12.986189486061873%2C%2042.86873330394106%5D%2C%20%5B12.98697780106198%2C%2042.868584261941116%5D%2C%20%5B12.989101809062223%2C%2042.86951406694111%5D%2C%20%5B12.9902247210623%2C%2042.871136048941054%5D%2C%20%5B12.99032811706229%2C%2042.87324829894115%5D%2C%20%5B12.990940654062337%2C%2042.87397871894104%5D%2C%20%5B12.990408436062282%2C%2042.87452851594107%5D%2C%20%5B12.991550646062421%2C%2042.875546564941075%5D%2C%20%5B12.991470160062395%2C%2042.87689542094109%5D%2C%20%5B12.992142017062458%2C%2042.8769439879411%5D%2C%20%5B12.993499149062608%2C%2042.87889539194114%5D%2C%20%5B12.99274018706249%2C%2042.880033854941054%5D%2C%20%5B12.99389915906266%2C%2042.88128538894114%5D%2C%20%5B12.995624846062777%2C%2042.885479299941096%5D%2C%20%5B12.99778177406304%2C%2042.88655184494109%5D%2C%20%5B13.00109796206338%2C%2042.88927645794109%5D%2C%20%5B13.000304115063273%2C%2042.89072681794115%5D%2C%20%5B12.998177875063028%2C%2042.89172858594112%5D%2C%20%5B12.99836192306303%2C%2042.89436021194115%5D%2C%20%5B13.000615991063299%2C%2042.89319190094117%5D%2C%20%5B13.002185003063476%2C%2042.8931459769412%5D%2C%20%5B13.002510501063494%2C%2042.89558356594116%5D%2C%20%5B12.99926617606308%2C%2042.89820913994115%5D%2C%20%5B12.993162760062422%2C%2042.89984060294115%5D%2C%20%5B12.992040466062269%2C%2042.900820685941156%5D%2C%20%5B12.999063205063056%2C%2042.90447376094123%5D%2C%20%5B12.999079537063004%2C%2042.906629556941176%5D%2C%20%5B13.000663325063172%2C%2042.90802371394113%5D%2C%20%5B13.002025060063279%2C%2042.912522866941224%5D%2C%20%5B13.001454961063256%2C%2042.913483704941186%5D%2C%20%5B13.004195655063512%2C%2042.91357679694125%5D%2C%20%5B13.005024850063604%2C%2042.91181905094122%5D%2C%20%5B13.005756077063756%2C%2042.90623819594119%5D%2C%20%5B13.009264029064171%2C%2042.903049468941155%5D%2C%20%5B13.013423938064655%2C%2042.90280893494118%5D%2C%20%5B13.014587021064782%2C%2042.903591918941196%5D%2C%20%5B13.01486414706482%2C%2042.90295643394113%5D%2C%20%5B13.019600161065357%2C%2042.90388841494115%5D%2C%20%5B13.023655581065821%2C%2042.903065948941155%5D%2C%20%5B13.023857812065835%2C%2042.90364406294116%5D%2C%20%5B13.02512184106596%2C%2042.90318989194118%5D%2C%20%5B13.025062530065988%2C%2042.902350142941195%5D%2C%20%5B13.026389362066155%2C%2042.90224938794115%5D%2C%20%5B13.02924485806647%2C%2042.90448519294117%5D%2C%20%5B13.029600919066501%2C%2042.905237945941224%5D%2C%20%5B13.030602725066588%2C%2042.905328671941156%5D%2C%20%5B13.031472443066669%2C%2042.906081310941175%5D%2C%20%5B13.033095457066873%2C%2042.906271649941196%5D%2C%20%5B13.034769922067065%2C%2042.9075540859412%5D%2C%20%5B13.038157700067458%2C%2042.907736060941176%5D%2C%20%5B13.040048229067695%2C%2042.90992014694124%5D%2C%20%5B13.042928781067953%2C%2042.91488730294126%5D%2C%20%5B13.044344442068137%2C%2042.91544945994127%5D%2C%20%5B13.044738417068181%2C%2042.91616930894119%5D%2C%20%5B13.046590838068354%2C%2042.916490890941226%5D%2C%20%5B13.048600415068618%2C%2042.91801785494125%5D%2C%20%5B13.050740570068864%2C%2042.918522751941204%5D%2C%20%5B13.052563913069093%2C%2042.91974112094122%5D%2C%20%5B13.054435496069281%2C%2042.92022395894126%5D%2C%20%5B13.056488819069536%2C%2042.91737799994119%5D%2C%20%5B13.05938538206993%2C%2042.9155644769412%5D%2C%20%5B13.068716802071132%2C%2042.905877955941236%5D%2C%20%5B13.072644164071647%2C%2042.90366725594123%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%223%22%2C%20%22properties%22%3A%20%7B%22COD_COM%22%3A%20%22043%22%2C%20%22COD_PROV%22%3A%20%22054%22%2C%20%22COD_REG%22%3A%20%2210%22%2C%20%22COMUNE%22%3A%20%22Preci%22%2C%20%22GlobalID%22%3A%20%2248ad1db6-a7c2-4b2b-8171-7c90ae629583%22%2C%20%22PROVINCIA%22%3A%20%22PG%22%2C%20%22Sum_ABITNO%22%3A%209.0%2C%20%22Sum_ABIT_R%22%3A%20381.0%2C%20%22Sum_ABIT_V%22%3A%20671.0%2C%20%22Sum_ALBERG%22%3A%2018.0%2C%20%22Sum_ALTRIA%22%3A%2020.0%2C%20%22Sum_EDIF_A%22%3A%20707.0%2C%20%22Sum_EDIF_T%22%3A%20978.0%2C%20%22Sum_EDIF_U%22%3A%20802.0%2C%20%22Sum_FAMRES%22%3A%20401.0%2C%20%22Sum_RES_0_%22%3A%2024.0%2C%20%22Sum_RES_10%22%3A%2038.0%2C%20%22Sum_RES_15%22%3A%2049.0%2C%20%22Sum_RES_20%22%3A%2028.0%2C%20%22Sum_RES_25%22%3A%2040.0%2C%20%22Sum_RES_30%22%3A%2058.0%2C%20%22Sum_RES_35%22%3A%2056.0%2C%20%22Sum_RES_40%22%3A%2061.0%2C%20%22Sum_RES_45%22%3A%2047.0%2C%20%22Sum_RES_50%22%3A%2049.0%2C%20%22Sum_RES_55%22%3A%2029.0%2C%20%22Sum_RES_5_%22%3A%2021.0%2C%20%22Sum_RES_60%22%3A%2050.0%2C%20%22Sum_RES_65%22%3A%2052.0%2C%20%22Sum_RES_70%22%3A%2056.0%2C%20%22Sum_RES_PI%22%3A%20159.0%2C%20%22Sum_RES_TO%22%3A%20817.0%2C%20%22Sum_STRAN_%22%3A%2053.0%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22bbox%22%3A%20%5B13.324877594111173%2C%2042.687157616940844%2C%2013.505236511148194%2C%2042.83267100994139%5D%2C%20%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B13.384486958121562%2C%2042.83019433194137%5D%2C%20%5B13.390480465122739%2C%2042.827893458941276%5D%2C%20%5B13.394138763123472%2C%2042.8274332359413%5D%2C%20%5B13.39579771412381%2C%2042.828553375941325%5D%2C%20%5B13.398979213124457%2C%2042.827580139941325%5D%2C%20%5B13.40073957412478%2C%2042.82765191694126%5D%2C%20%5B13.400005259124674%2C%2042.82622615294132%5D%2C%20%5B13.401204099124882%2C%2042.82576583494128%5D%2C%20%5B13.400086285124681%2C%2042.825462250941285%5D%2C%20%5B13.400383201124782%2C%2042.82390674594131%5D%2C%20%5B13.39827077912439%2C%2042.82282663394125%5D%2C%20%5B13.398567487124419%2C%2042.82222549494132%5D%2C%20%5B13.40011317912477%2C%2042.82108559194128%5D%2C%20%5B13.401575511125076%2C%2042.82225823594133%5D%2C%20%5B13.402595063125283%2C%2042.82164274894131%5D%2C%20%5B13.403980175125561%2C%2042.82009482694129%5D%2C%20%5B13.404174694125619%2C%2042.81869181294128%5D%2C%20%5B13.405348474125866%2C%2042.8172690609413%5D%2C%20%5B13.408075602126427%2C%2042.81684878694127%5D%2C%20%5B13.410171838126864%2C%2042.81570551894133%5D%2C%20%5B13.412560923127357%2C%2042.815541271941306%5D%2C%20%5B13.415590122128041%2C%2042.813312989941316%5D%2C%20%5B13.417716734128446%2C%2042.81432470794136%5D%2C%20%5B13.417415899128368%2C%2042.8136430799413%5D%2C%20%5B13.41801615912851%2C%2042.81266102194131%5D%2C%20%5B13.419894913128896%2C%2042.813326648941285%5D%2C%20%5B13.421392268129246%2C%2042.81267450894126%5D%2C%20%5B13.421430066129263%2C%2042.811169490941275%5D%2C%20%5B13.424490525129878%2C%2042.81153722494129%5D%2C%20%5B13.425516888130067%2C%2042.81254186994129%5D%2C%20%5B13.428188551130608%2C%2042.81330719994132%5D%2C%20%5B13.431276256131245%2C%2042.81358366194133%5D%2C%20%5B13.43178874813139%2C%2042.81314063294127%5D%2C%20%5B13.435880779132209%2C%2042.81339602394132%5D%2C%20%5B13.437118451132486%2C%2042.81278076694129%5D%2C%20%5B13.441422371133369%2C%2042.813315837941374%5D%2C%20%5B13.441401627133372%2C%2042.81367677594129%5D%2C%20%5B13.442424572133573%2C%2042.81386209894132%5D%2C%20%5B13.444630267134047%2C%2042.81320012294133%5D%2C%20%5B13.447465346134663%2C%2042.813310404941305%5D%2C%20%5B13.448733784134962%2C%2042.81295492194137%5D%2C%20%5B13.45211676513573%2C%2042.81093698594138%5D%2C%20%5B13.453719116136087%2C%2042.808232080941295%5D%2C%20%5B13.458548479137166%2C%2042.80636465494129%5D%2C%20%5B13.460040822137465%2C%2042.8073282969413%5D%2C%20%5B13.464871087138517%2C%2042.80831911294135%5D%2C%20%5B13.465363180138661%2C%2042.80575195394135%5D%2C%20%5B13.466317607138873%2C%2042.805719158941294%5D%2C%20%5B13.467058148139081%2C%2042.80447927194134%5D%2C%20%5B13.468220800139301%2C%2042.80394313994138%5D%2C%20%5B13.469185481139537%2C%2042.80399995294133%5D%2C%20%5B13.469518883139648%2C%2042.803356682941335%5D%2C%20%5B13.468502839139411%2C%2042.80191987694132%5D%2C%20%5B13.466209580138905%2C%2042.802693772941375%5D%2C%20%5B13.463591698138352%2C%2042.80172015894138%5D%2C%20%5B13.46335630013829%2C%2042.80133770994134%5D%2C%20%5B13.463809787138418%2C%2042.79982544994139%5D%2C%20%5B13.466496406139061%2C%2042.798509536941275%5D%2C%20%5B13.467885416139376%2C%2042.79672660894134%5D%2C%20%5B13.467733783139392%2C%2042.795418055941376%5D%2C%20%5B13.466560021139136%2C%2042.79394237894133%5D%2C%20%5B13.466412417139184%2C%2042.78951852494133%5D%2C%20%5B13.465315949138951%2C%2042.78838193894129%5D%2C%20%5B13.464438986138752%2C%2042.78821362594128%5D%2C%20%5B13.460805965137956%2C%2042.78928282894126%5D%2C%20%5B13.459722024137722%2C%2042.79058109394135%5D%2C%20%5B13.459601831137666%2C%2042.789536898941286%5D%2C%20%5B13.460288062137876%2C%2042.787988559941255%5D%2C%20%5B13.459990531137812%2C%2042.78714485694125%5D%2C%20%5B13.461354044138133%2C%2042.78592120794124%5D%2C%20%5B13.460897105138038%2C%2042.78560142094124%5D%2C%20%5B13.461198985138115%2C%2042.784774837941235%5D%2C%20%5B13.463948452138732%2C%2042.78335299594127%5D%2C%20%5B13.464413650138836%2C%2042.78234446094127%5D%2C%20%5B13.464022012138784%2C%2042.78085170494127%5D%2C%20%5B13.462328059138454%2C%2042.77965739394126%5D%2C%20%5B13.462443615138463%2C%2042.77895962694126%5D%2C%20%5B13.46347629013873%2C%2042.77822604594131%5D%2C%20%5B13.464664632138973%2C%2042.77811660694129%5D%2C%20%5B13.4658867421393%2C%2042.77566497494125%5D%2C%20%5B13.465447862139218%2C%2042.77512841994123%5D%2C%20%5B13.465675205139284%2C%2042.77431824194126%5D%2C%20%5B13.467437465139675%2C%2042.77227314494123%5D%2C%20%5B13.468180679139843%2C%2042.77183894194121%5D%2C%20%5B13.476266576141645%2C%2042.772098745941236%5D%2C%20%5B13.4773858241419%2C%2042.77232500194124%5D%2C%20%5B13.477871970142015%2C%2042.77292268194125%5D%2C%20%5B13.48020614514252%2C%2042.77294374494128%5D%2C%20%5B13.482317855142991%2C%2042.7740718879413%5D%2C%20%5B13.485923199143718%2C%2042.77810795694131%5D%2C%20%5B13.492964795145268%2C%2042.78213941394131%5D%2C%20%5B13.497940029146369%2C%2042.782105766941335%5D%2C%20%5B13.499450788146724%2C%2042.7813125299413%5D%2C%20%5B13.501234341147123%2C%2042.78186802394131%5D%2C%20%5B13.500630865147063%2C%2042.778015880941275%5D%2C%20%5B13.502199734147418%2C%2042.77784604794132%5D%2C%20%5B13.503547537147801%2C%2042.77619936394132%5D%2C%20%5B13.505236511148194%2C%2042.775570089941304%5D%2C%20%5B13.504217395147965%2C%2042.77473694294126%5D%2C%20%5B13.503429902147806%2C%2042.7724226239413%5D%2C%20%5B13.502328260147596%2C%2042.77096698294129%5D%2C%20%5B13.502788305147764%2C%2042.76810832394121%5D%2C%20%5B13.502290145147672%2C%2042.76648485094122%5D%2C%20%5B13.50289762114782%2C%2042.764142573941236%5D%2C%20%5B13.50181691614761%2C%2042.76279414094126%5D%2C%20%5B13.5025982041478%2C%2042.761300329941214%5D%2C%20%5B13.502956036147944%2C%2042.75861225594125%5D%2C%20%5B13.504018388148252%2C%2042.75662568594125%5D%2C%20%5B13.503645392148172%2C%2042.75588410894127%5D%2C%20%5B13.503811841148243%2C%2042.754031875941266%5D%2C%20%5B13.501976043147845%2C%2042.75261866094122%5D%2C%20%5B13.50186011514781%2C%2042.75186244894119%5D%2C%20%5B13.502069584147907%2C%2042.75128699094124%5D%2C%20%5B13.504283089148418%2C%2042.75018240594129%5D%2C%20%5B13.504638634148543%2C%2042.747611458941186%5D%2C%20%5B13.504235322148459%2C%2042.74708265494122%5D%2C%20%5B13.501079347147767%2C%2042.74621212294123%5D%2C%20%5B13.500055633147513%2C%2042.745118017941216%5D%2C%20%5B13.498169253147172%2C%2042.74073564094124%5D%2C%20%5B13.496795465146853%2C%2042.73947521694119%5D%2C%20%5B13.4938984801462%2C%2042.73852676694114%5D%2C%20%5B13.493456187146146%2C%2042.73661294694114%5D%2C%20%5B13.491390255145701%2C%2042.73565876694121%5D%2C%20%5B13.48951255114532%2C%2042.73401290394116%5D%2C%20%5B13.488812323145135%2C%2042.73367126994114%5D%2C%20%5B13.487754096144894%2C%2042.73397392094111%5D%2C%20%5B13.486022324144555%2C%2042.73286696994114%5D%2C%20%5B13.48436762014415%2C%2042.73308045694117%5D%2C%20%5B13.481253943143484%2C%2042.731554966941125%5D%2C%20%5B13.480127326143213%2C%2042.73163964594114%5D%2C%20%5B13.47914880714298%2C%2042.73267736494117%5D%2C%20%5B13.474820608142021%2C%2042.73238772494111%5D%2C%20%5B13.471628708141308%2C%2042.73304833994116%5D%2C%20%5B13.468943901140678%2C%2042.73292825894113%5D%2C%20%5B13.465344774139895%2C%2042.73374417094111%5D%2C%20%5B13.464280597139652%2C%2042.734794112941145%5D%2C%20%5B13.463266270139416%2C%2042.73480671494111%5D%2C%20%5B13.459839004138667%2C%2042.73408517494119%5D%2C%20%5B13.45900954313847%2C%2042.73359984394106%5D%2C%20%5B13.452121045137%2C%2042.73390437494114%5D%2C%20%5B13.450333346136595%2C%2042.73266850694107%5D%2C%20%5B13.447999475136141%2C%2042.732214647941085%5D%2C%20%5B13.447438212136028%2C%2042.731273128941126%5D%2C%20%5B13.445693288135669%2C%2042.73064324794104%5D%2C%20%5B13.445159568135558%2C%2042.72952057894112%5D%2C%20%5B13.442775841135083%2C%2042.727641517941045%5D%2C%20%5B13.441205636134757%2C%2042.72695525794111%5D%2C%20%5B13.440866198134705%2C%2042.72392530894104%5D%2C%20%5B13.439232562134464%2C%2042.71989223194104%5D%2C%20%5B13.439626000134579%2C%2042.71881905594105%5D%2C%20%5B13.438021675134229%2C%2042.71754435394099%5D%2C%20%5B13.437324517134146%2C%2042.715635692941056%5D%2C%20%5B13.437789922134266%2C%2042.713614374941095%5D%2C%20%5B13.43739132313415%2C%2042.71247741694103%5D%2C%20%5B13.43842562013444%2C%2042.71116778594097%5D%2C%20%5B13.438774117134491%2C%2042.709497632941066%5D%2C%20%5B13.437650623134337%2C%2042.70700230094097%5D%2C%20%5B13.438262757134478%2C%2042.70597916094107%5D%2C%20%5B13.440838742135016%2C%2042.70452861094103%5D%2C%20%5B13.441178029135136%2C%2042.70366010894098%5D%2C%20%5B13.440903099135111%2C%2042.70245983494105%5D%2C%20%5B13.438636175134608%2C%2042.70132341894103%5D%2C%20%5B13.437578248134443%2C%2042.69919917794106%5D%2C%20%5B13.437011624134339%2C%2042.69892406094102%5D%2C%20%5B13.43383857213362%2C%2042.70036617194095%5D%2C%20%5B13.432930446133392%2C%2042.70148629894099%5D%2C%20%5B13.43242580813327%2C%2042.70310846994108%5D%2C%20%5B13.428746722132464%2C%2042.70615915894102%5D%2C%20%5B13.42740921413217%2C%2042.706427049941055%5D%2C%20%5B13.420206434130725%2C%2042.70252821294095%5D%2C%20%5B13.412987309129306%2C%2042.69736456394098%5D%2C%20%5B13.405875205127884%2C%2042.69573016494095%5D%2C%20%5B13.401834747127067%2C%2042.693806022940876%5D%2C%20%5B13.400253613126797%2C%2042.69072469694093%5D%2C%20%5B13.397771402126297%2C%2042.68899701694091%5D%2C%20%5B13.394068951125579%2C%2042.68819852494091%5D%2C%20%5B13.39261265512528%2C%2042.68738565794092%5D%2C%20%5B13.389994048124775%2C%2042.687157616940844%5D%2C%20%5B13.388818140124545%2C%2042.68730180294087%5D%2C%20%5B13.385831214123916%2C%2042.68905505694096%5D%2C%20%5B13.380844280122915%2C%2042.690713824940914%5D%2C%20%5B13.379371346122609%2C%2042.6920037299409%5D%2C%20%5B13.374588923121653%2C%2042.69376693594084%5D%2C%20%5B13.37327038212133%2C%2042.69517240094091%5D%2C%20%5B13.369194619120561%2C%2042.69323043694094%5D%2C%20%5B13.365109586119798%2C%2042.6930443529409%5D%2C%20%5B13.362922794119331%2C%2042.6935915069409%5D%2C%20%5B13.358884051118563%2C%2042.69582986594089%5D%2C%20%5B13.357493560118286%2C%2042.697161364940925%5D%2C%20%5B13.357002361118159%2C%2042.698665657940886%5D%2C%20%5B13.356338811118016%2C%2042.69926716594084%5D%2C%20%5B13.355750408117869%2C%2042.701693509940924%5D%2C%20%5B13.352902109117297%2C%2042.70370622894087%5D%2C%20%5B13.352020599117113%2C%2042.705599004940964%5D%2C%20%5B13.350124570116702%2C%2042.707561894940895%5D%2C%20%5B13.348786468116467%2C%2042.70832857994097%5D%2C%20%5B13.346160518115903%2C%2042.713835020940856%5D%2C%20%5B13.347444535116109%2C%2042.716559263940965%5D%2C%20%5B13.347862188116146%2C%2042.71923541394089%5D%2C%20%5B13.346039033115762%2C%2042.72163663494096%5D%2C%20%5B13.345669062115661%2C%2042.72314527394093%5D%2C%20%5B13.346261330115746%2C%2042.72473887794091%5D%2C%20%5B13.347579915115958%2C%2042.725706127941%5D%2C%20%5B13.346530124115747%2C%2042.73028378794101%5D%2C%20%5B13.351108216116549%2C%2042.732680108941025%5D%2C%20%5B13.35193107911669%2C%2042.73343208494102%5D%2C%20%5B13.352410379116753%2C%2042.734705830940996%5D%2C%20%5B13.351309255116522%2C%2042.736138782941026%5D%2C%20%5B13.35018205611628%2C%2042.73903127394101%5D%2C%20%5B13.350013494116183%2C%2042.74312074794098%5D%2C%20%5B13.349360741116044%2C%2042.74512634294104%5D%2C%20%5B13.35025042211618%2C%2042.746132384941085%5D%2C%20%5B13.349408108116037%2C%2042.74886546894107%5D%2C%20%5B13.3498854471161%2C%2042.74976115394104%5D%2C%20%5B13.351942263116463%2C%2042.75097935894107%5D%2C%20%5B13.352760854116598%2C%2042.75100221094106%5D%2C%20%5B13.352975554116636%2C%2042.75164678394109%5D%2C%20%5B13.351150641116275%2C%2042.75225647094102%5D%2C%20%5B13.350526721116143%2C%2042.75320307094099%5D%2C%20%5B13.349536639115959%2C%2042.75368641894107%5D%2C%20%5B13.344884118115031%2C%2042.75539844394101%5D%2C%20%5B13.34225447011455%2C%2042.75734407294111%5D%2C%20%5B13.343346744114765%2C%2042.75809037494106%5D%2C%20%5B13.342291393114479%2C%2042.76143022894106%5D%2C%20%5B13.342813891114561%2C%2042.76292745794108%5D%2C%20%5B13.342496865114496%2C%2042.76336266294107%5D%2C%20%5B13.339308043113926%2C%2042.763335206941115%5D%2C%20%5B13.33695239711347%2C%2042.76458605194103%5D%2C%20%5B13.331221776112411%2C%2042.7645242309411%5D%2C%20%5B13.329590673112074%2C%2042.76610310494114%5D%2C%20%5B13.32798387911179%2C%2042.76631251094107%5D%2C%20%5B13.326342650111487%2C%2042.768251859941095%5D%2C%20%5B13.324877594111173%2C%2042.76910410594102%5D%2C%20%5B13.33031424511218%2C%2042.77009122994107%5D%2C%20%5B13.330312446112135%2C%2042.77153184794107%5D%2C%20%5B13.3335446041127%2C%2042.77406536694107%5D%2C%20%5B13.334834849112923%2C%2042.774723212941076%5D%2C%20%5B13.33695804111332%2C%2042.77487225994108%5D%2C%20%5B13.339093770113736%2C%2042.77592113394114%5D%2C%20%5B13.33927962711371%2C%2042.77766074294118%5D%2C%20%5B13.34035586411388%2C%2042.77946107994109%5D%2C%20%5B13.340671147113941%2C%2042.78184851994112%5D%2C%20%5B13.341943831114124%2C%2042.78180468194109%5D%2C%20%5B13.341984897114177%2C%2042.78255490794117%5D%2C%20%5B13.340908141113935%2C%2042.78258230094114%5D%2C%20%5B13.340624124113827%2C%2042.786361018941086%5D%2C%20%5B13.342868521114257%2C%2042.78655486894113%5D%2C%20%5B13.341201225113947%2C%2042.78763109094109%5D%2C%20%5B13.338769579113494%2C%2042.78850671994115%5D%2C%20%5B13.337597275113225%2C%2042.7907300429412%5D%2C%20%5B13.334458323112665%2C%2042.7910426959411%5D%2C%20%5B13.333976115112465%2C%2042.795828292941195%5D%2C%20%5B13.335806805112789%2C%2042.798230297941224%5D%2C%20%5B13.337929908113189%2C%2042.799774853941194%5D%2C%20%5B13.339483959113426%2C%2042.80156155994113%5D%2C%20%5B13.341304780113772%2C%2042.802518800941236%5D%2C%20%5B13.343092819114048%2C%2042.802887535941196%5D%2C%20%5B13.344287522114268%2C%2042.80441772694118%5D%2C%20%5B13.346758803114712%2C%2042.80528712994122%5D%2C%20%5B13.347037478114723%2C%2042.80861679194126%5D%2C%20%5B13.347855478114813%2C%2042.81094006494121%5D%2C%20%5B13.347375635114727%2C%2042.81153003394123%5D%2C%20%5B13.347666819114764%2C%2042.812531838941226%5D%2C%20%5B13.349175407115071%2C%2042.81283457694118%5D%2C%20%5B13.351472419115455%2C%2042.81431823494118%5D%2C%20%5B13.352718692115669%2C%2042.815738329941205%5D%2C%20%5B13.35722461611649%2C%2042.81832616794129%5D%2C%20%5B13.358370879116688%2C%2042.81985354994128%5D%2C%20%5B13.361364881117204%2C%2042.821805607941236%5D%2C%20%5B13.361184522117183%2C%2042.82353213594128%5D%2C%20%5B13.362443337117382%2C%2042.82498315294132%5D%2C%20%5B13.364149510117695%2C%2042.82524213494125%5D%2C%20%5B13.367537776118294%2C%2042.828506966941276%5D%2C%20%5B13.370025747118762%2C%2042.82880350394133%5D%2C%20%5B13.371662345119042%2C%2042.83077116894129%5D%2C%20%5B13.375259559119765%2C%2042.830318388941286%5D%2C%20%5B13.377938869120252%2C%2042.83077849094128%5D%2C%20%5B13.378912238120451%2C%2042.830034434941325%5D%2C%20%5B13.379598571120615%2C%2042.82999913194129%5D%2C%20%5B13.381180603120882%2C%2042.83267100994139%5D%2C%20%5B13.384486958121562%2C%2042.83019433194137%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%224%22%2C%20%22properties%22%3A%20%7B%22COD_COM%22%3A%20%22001%22%2C%20%22COD_PROV%22%3A%20%22044%22%2C%20%22COD_REG%22%3A%20%2211%22%2C%20%22COMUNE%22%3A%20%22Acquasanta%20Terme%22%2C%20%22GlobalID%22%3A%20%22943aeaab-0132-408f-9036-82789a2bbe99%22%2C%20%22PROVINCIA%22%3A%20%22AP%22%2C%20%22Sum_ABITNO%22%3A%2020.0%2C%20%22Sum_ABIT_R%22%3A%201285.0%2C%20%22Sum_ABIT_V%22%3A%20894.0%2C%20%22Sum_ALBERG%22%3A%2028.0%2C%20%22Sum_ALTRIA%22%3A%203.0%2C%20%22Sum_EDIF_A%22%3A%201362.0%2C%20%22Sum_EDIF_T%22%3A%201668.0%2C%20%22Sum_EDIF_U%22%3A%201451.0%2C%20%22Sum_FAMRES%22%3A%201290.0%2C%20%22Sum_RES_0_%22%3A%2096.0%2C%20%22Sum_RES_10%22%3A%20151.0%2C%20%22Sum_RES_15%22%3A%20159.0%2C%20%22Sum_RES_20%22%3A%20210.0%2C%20%22Sum_RES_25%22%3A%20181.0%2C%20%22Sum_RES_30%22%3A%20227.0%2C%20%22Sum_RES_35%22%3A%20234.0%2C%20%22Sum_RES_40%22%3A%20210.0%2C%20%22Sum_RES_45%22%3A%20228.0%2C%20%22Sum_RES_50%22%3A%20207.0%2C%20%22Sum_RES_55%22%3A%20208.0%2C%20%22Sum_RES_5_%22%3A%20135.0%2C%20%22Sum_RES_60%22%3A%20216.0%2C%20%22Sum_RES_65%22%3A%20199.0%2C%20%22Sum_RES_70%22%3A%20216.0%2C%20%22Sum_RES_PI%22%3A%20469.0%2C%20%22Sum_RES_TO%22%3A%203346.0%2C%20%22Sum_STRAN_%22%3A%2040.0%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22bbox%22%3A%20%5B13.187558654088953%2C%2042.69409415394087%2C%2013.358884051118563%2C%2042.82343779694116%5D%2C%20%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B13.27855478910239%2C%2042.82333758194114%5D%2C%20%5B13.280951271102797%2C%2042.822518565941145%5D%2C%20%5B13.284251431103389%2C%2042.823083671941184%5D%2C%20%5B13.289821507104305%2C%2042.82334292194114%5D%2C%20%5B13.299319373105975%2C%2042.820937755941195%5D%2C%20%5B13.301371208106378%2C%2042.81926694694117%5D%2C%20%5B13.30249065410657%2C%2042.816807389941175%5D%2C%20%5B13.306018214107201%2C%2042.816643030941165%5D%2C%20%5B13.306981919107372%2C%2042.81627809394114%5D%2C%20%5B13.309451823107848%2C%2042.814073687941196%5D%2C%20%5B13.31319658310856%2C%2042.81174009594118%5D%2C%20%5B13.31364720710868%2C%2042.80866192494118%5D%2C%20%5B13.31504870910891%2C%2042.807474588941155%5D%2C%20%5B13.315218324108953%2C%2042.80663986994111%5D%2C%20%5B13.318062466109488%2C%2042.80548355494117%5D%2C%20%5B13.317843354109444%2C%2042.80408279694113%5D%2C%20%5B13.318238970109517%2C%2042.80348712894117%5D%2C%20%5B13.320102607109911%2C%2042.80245333694115%5D%2C%20%5B13.321487835110156%2C%2042.803184267941205%5D%2C%20%5B13.322749581110342%2C%2042.80265937294118%5D%2C%20%5B13.323086381110421%2C%2042.80194436794113%5D%2C%20%5B13.32711967011118%2C%2042.79976597194113%5D%2C%20%5B13.32810127111133%2C%2042.80011145094117%5D%2C%20%5B13.329539894111583%2C%2042.80162806094113%5D%2C%20%5B13.335697358112702%2C%2042.803208834941216%5D%2C%20%5B13.341304780113772%2C%2042.802518800941236%5D%2C%20%5B13.339483959113426%2C%2042.80156155994113%5D%2C%20%5B13.337929908113189%2C%2042.799774853941194%5D%2C%20%5B13.335806805112789%2C%2042.798230297941224%5D%2C%20%5B13.333976115112465%2C%2042.795828292941195%5D%2C%20%5B13.334458323112665%2C%2042.7910426959411%5D%2C%20%5B13.337597275113225%2C%2042.7907300429412%5D%2C%20%5B13.338769579113494%2C%2042.78850671994115%5D%2C%20%5B13.341201225113947%2C%2042.78763109094109%5D%2C%20%5B13.342868521114257%2C%2042.78655486894113%5D%2C%20%5B13.340624124113827%2C%2042.786361018941086%5D%2C%20%5B13.340908141113935%2C%2042.78258230094114%5D%2C%20%5B13.341984897114177%2C%2042.78255490794117%5D%2C%20%5B13.341943831114124%2C%2042.78180468194109%5D%2C%20%5B13.340671147113941%2C%2042.78184851994112%5D%2C%20%5B13.34035586411388%2C%2042.77946107994109%5D%2C%20%5B13.33927962711371%2C%2042.77766074294118%5D%2C%20%5B13.339093770113736%2C%2042.77592113394114%5D%2C%20%5B13.33695804111332%2C%2042.77487225994108%5D%2C%20%5B13.334834849112923%2C%2042.774723212941076%5D%2C%20%5B13.3335446041127%2C%2042.77406536694107%5D%2C%20%5B13.330312446112135%2C%2042.77153184794107%5D%2C%20%5B13.33031424511218%2C%2042.77009122994107%5D%2C%20%5B13.324877594111173%2C%2042.76910410594102%5D%2C%20%5B13.326342650111487%2C%2042.768251859941095%5D%2C%20%5B13.32798387911179%2C%2042.76631251094107%5D%2C%20%5B13.329590673112074%2C%2042.76610310494114%5D%2C%20%5B13.331221776112411%2C%2042.7645242309411%5D%2C%20%5B13.33695239711347%2C%2042.76458605194103%5D%2C%20%5B13.339308043113926%2C%2042.763335206941115%5D%2C%20%5B13.342496865114496%2C%2042.76336266294107%5D%2C%20%5B13.342813891114561%2C%2042.76292745794108%5D%2C%20%5B13.342291393114479%2C%2042.76143022894106%5D%2C%20%5B13.343346744114765%2C%2042.75809037494106%5D%2C%20%5B13.34225447011455%2C%2042.75734407294111%5D%2C%20%5B13.344884118115031%2C%2042.75539844394101%5D%2C%20%5B13.349536639115959%2C%2042.75368641894107%5D%2C%20%5B13.350526721116143%2C%2042.75320307094099%5D%2C%20%5B13.351150641116275%2C%2042.75225647094102%5D%2C%20%5B13.352975554116636%2C%2042.75164678394109%5D%2C%20%5B13.352760854116598%2C%2042.75100221094106%5D%2C%20%5B13.351942263116463%2C%2042.75097935894107%5D%2C%20%5B13.3498854471161%2C%2042.74976115394104%5D%2C%20%5B13.349408108116037%2C%2042.74886546894107%5D%2C%20%5B13.35025042211618%2C%2042.746132384941085%5D%2C%20%5B13.349360741116044%2C%2042.74512634294104%5D%2C%20%5B13.350013494116183%2C%2042.74312074794098%5D%2C%20%5B13.35018205611628%2C%2042.73903127394101%5D%2C%20%5B13.351309255116522%2C%2042.736138782941026%5D%2C%20%5B13.352410379116753%2C%2042.734705830940996%5D%2C%20%5B13.35193107911669%2C%2042.73343208494102%5D%2C%20%5B13.351108216116549%2C%2042.732680108941025%5D%2C%20%5B13.346530124115747%2C%2042.73028378794101%5D%2C%20%5B13.347579915115958%2C%2042.725706127941%5D%2C%20%5B13.346261330115746%2C%2042.72473887794091%5D%2C%20%5B13.345669062115661%2C%2042.72314527394093%5D%2C%20%5B13.346039033115762%2C%2042.72163663494096%5D%2C%20%5B13.347862188116146%2C%2042.71923541394089%5D%2C%20%5B13.347444535116109%2C%2042.716559263940965%5D%2C%20%5B13.346160518115903%2C%2042.713835020940856%5D%2C%20%5B13.348786468116467%2C%2042.70832857994097%5D%2C%20%5B13.350124570116702%2C%2042.707561894940895%5D%2C%20%5B13.352020599117113%2C%2042.705599004940964%5D%2C%20%5B13.352902109117297%2C%2042.70370622894087%5D%2C%20%5B13.355750408117869%2C%2042.701693509940924%5D%2C%20%5B13.356338811118016%2C%2042.69926716594084%5D%2C%20%5B13.357002361118159%2C%2042.698665657940886%5D%2C%20%5B13.357493560118286%2C%2042.697161364940925%5D%2C%20%5B13.358884051118563%2C%2042.69582986594089%5D%2C%20%5B13.35776997611835%2C%2042.69409415394087%5D%2C%20%5B13.356663937118137%2C%2042.6948250729409%5D%2C%20%5B13.354233747117696%2C%2042.69534081894087%5D%2C%20%5B13.352327285117296%2C%2042.69633176794087%5D%2C%20%5B13.351034473117048%2C%2042.69765496894084%5D%2C%20%5B13.349657459116775%2C%2042.698058502940874%5D%2C%20%5B13.344583024115789%2C%2042.7011100579409%5D%2C%20%5B13.341170935115086%2C%2042.701761876940836%5D%2C%20%5B13.338436625114555%2C%2042.70402650694087%5D%2C%20%5B13.336484162114234%2C%2042.704879384940845%5D%2C%20%5B13.33550654511398%2C%2042.70629849694087%5D%2C%20%5B13.33414976911375%2C%2042.707259293940865%5D%2C%20%5B13.333941893113703%2C%2042.708644706940895%5D%2C%20%5B13.33197723811333%2C%2042.709029788940875%5D%2C%20%5B13.328285354112587%2C%2042.711929191940904%5D%2C%20%5B13.32637170511216%2C%2042.7159001339409%5D%2C%20%5B13.323960123111721%2C%2042.718377294940964%5D%2C%20%5B13.321282775111186%2C%2042.72028372694092%5D%2C%20%5B13.317884770110574%2C%2042.72399100494096%5D%2C%20%5B13.315196638110079%2C%2042.72448415694093%5D%2C%20%5B13.314459017109888%2C%2042.72555640094091%5D%2C%20%5B13.309436303108944%2C%2042.727902184940945%5D%2C%20%5B13.300166363107314%2C%2042.729664801940906%5D%2C%20%5B13.297322626106784%2C%2042.72956915594087%5D%2C%20%5B13.294003091106246%2C%2042.73080582194093%5D%2C%20%5B13.290467593105577%2C%2042.733238959940905%5D%2C%20%5B13.290092576105472%2C%2042.73575600694097%5D%2C%20%5B13.28810885310509%2C%2042.73684332794094%5D%2C%20%5B13.287774827105045%2C%2042.739214773940944%5D%2C%20%5B13.286288620104745%2C%2042.74040044694093%5D%2C%20%5B13.282357676104098%2C%2042.73941332694093%5D%2C%20%5B13.28005436910371%2C%2042.74096265494097%5D%2C%20%5B13.277246403103202%2C%2042.740473518940966%5D%2C%20%5B13.274107519102655%2C%2042.74043335594092%5D%2C%20%5B13.269626060101938%2C%2042.7394753799409%5D%2C%20%5B13.268880723101802%2C%2042.7394717019409%5D%2C%20%5B13.267751547101598%2C%2042.74010359194093%5D%2C%20%5B13.264511484101085%2C%2042.7401794859409%5D%2C%20%5B13.26268194310083%2C%2042.73614661394096%5D%2C%20%5B13.26079507610055%2C%2042.73329981194093%5D%2C%20%5B13.259925611100462%2C%2042.7298703559409%5D%2C%20%5B13.260383146100509%2C%2042.72837222594085%5D%2C%20%5B13.257102736100066%2C%2042.723817088940834%5D%2C%20%5B13.254962322099692%2C%2042.72174942594091%5D%2C%20%5B13.254164934099586%2C%2042.72145047094085%5D%2C%20%5B13.252745724099341%2C%2042.72284930194087%5D%2C%20%5B13.250880652099049%2C%2042.72270708094085%5D%2C%20%5B13.251041515099033%2C%2042.72441177294088%5D%2C%20%5B13.249364505098756%2C%2042.72493329794085%5D%2C%20%5B13.248115718098541%2C%2042.725902570940896%5D%2C%20%5B13.239997844097232%2C%2042.72620392494089%5D%2C%20%5B13.238115079096918%2C%2042.72581455394093%5D%2C%20%5B13.230190265095665%2C%2042.72703541994086%5D%2C%20%5B13.224737659094776%2C%2042.728911701940916%5D%2C%20%5B13.224315305094683%2C%2042.72830156094081%5D%2C%20%5B13.20710503509195%2C%2042.73087193794084%5D%2C%20%5B13.200227836090889%2C%2042.73508637694093%5D%2C%20%5B13.196855280090386%2C%2042.73470152794093%5D%2C%20%5B13.19421863008999%2C%2042.73383039994082%5D%2C%20%5B13.190266638089403%2C%2042.73362417194085%5D%2C%20%5B13.189299747089247%2C%2042.73325444394087%5D%2C%20%5B13.188876060089147%2C%2042.73443143194086%5D%2C%20%5B13.187715356089006%2C%2042.73356458994084%5D%2C%20%5B13.187558654088953%2C%2042.73391246594081%5D%2C%20%5B13.187778909089008%2C%2042.73597972594085%5D%2C%20%5B13.188780168089101%2C%2042.73808139894088%5D%2C%20%5B13.191263623089458%2C%2042.7400747129409%5D%2C%20%5B13.192623360089673%2C%2042.742473831940856%5D%2C%20%5B13.195616520090061%2C%2042.74372356594085%5D%2C%20%5B13.197068163090247%2C%2042.746303829940864%5D%2C%20%5B13.199260895090584%2C%2042.74866327794087%5D%2C%20%5B13.199539162090591%2C%2042.751682757940905%5D%2C%20%5B13.198779045090422%2C%2042.75433071694097%5D%2C%20%5B13.196434167090066%2C%2042.756388595940884%5D%2C%20%5B13.19584395908995%2C%2042.758219976940914%5D%2C%20%5B13.19330806808959%2C%2042.75884423194097%5D%2C%20%5B13.19339605508956%2C%2042.75959280294093%5D%2C%20%5B13.19512447708984%2C%2042.76192433894096%5D%2C%20%5B13.199179601090428%2C%2042.76288289694092%5D%2C%20%5B13.20096452209065%2C%2042.76453249694097%5D%2C%20%5B13.203773963091098%2C%2042.763321783940924%5D%2C%20%5B13.208791325091855%2C%2042.763915980940936%5D%2C%20%5B13.21295422309246%2C%2042.767030929940944%5D%2C%20%5B13.21384337209257%2C%2042.76886639394097%5D%2C%20%5B13.215454783092815%2C%2042.769774864941%5D%2C%20%5B13.216493372092964%2C%2042.77149225194092%5D%2C%20%5B13.217626678093126%2C%2042.77204017594103%5D%2C%20%5B13.219264011093404%2C%2042.772047281941%5D%2C%20%5B13.22104285609366%2C%2042.772697393940966%5D%2C%20%5B13.222613719093891%2C%2042.772468311941%5D%2C%20%5B13.22467938609424%2C%2042.770330180940995%5D%2C%20%5B13.225784171094412%2C%2042.770581960941%5D%2C%20%5B13.228224997094776%2C%2042.77244997494098%5D%2C%20%5B13.229396213095008%2C%2042.77251018894103%5D%2C%20%5B13.230533437095177%2C%2042.772049443941%5D%2C%20%5B13.233767166095669%2C%2042.77310918494103%5D%2C%20%5B13.235031408095864%2C%2042.772931803941034%5D%2C%20%5B13.236792879096186%2C%2042.76986835894101%5D%2C%20%5B13.23667827109623%2C%2042.765717479940974%5D%2C%20%5B13.238595685096536%2C%2042.764493949940906%5D%2C%20%5B13.238537319096556%2C%2042.76094422294093%5D%2C%20%5B13.241343455096992%2C%2042.76281641794104%5D%2C%20%5B13.245016980097546%2C%2042.766916218940985%5D%2C%20%5B13.245603151097637%2C%2042.76778129694103%5D%2C%20%5B13.245496432097557%2C%2042.77069789794096%5D%2C%20%5B13.244343667097356%2C%2042.77427908594102%5D%2C%20%5B13.24728523509777%2C%2042.777046450940965%5D%2C%20%5B13.247999420097834%2C%2042.781206542941064%5D%2C%20%5B13.250628672098221%2C%2042.78593017794103%5D%2C%20%5B13.25101371209828%2C%2042.78920663794106%5D%2C%20%5B13.251891395098404%2C%2042.79017338194107%5D%2C%20%5B13.253213305098628%2C%2042.79054286494112%5D%2C%20%5B13.255496973098953%2C%2042.79064244094109%5D%2C%20%5B13.258131232099387%2C%2042.7895944709411%5D%2C%20%5B13.259534031099626%2C%2042.78969526494104%5D%2C%20%5B13.256117186099026%2C%2042.792064409941034%5D%2C%20%5B13.253850586098666%2C%2042.792670985941115%5D%2C%20%5B13.251479210098271%2C%2042.79437983994109%5D%2C%20%5B13.259414947099502%2C%2042.79829352994106%5D%2C%20%5B13.258448509099331%2C%2042.800053705941046%5D%2C%20%5B13.258456999099357%2C%2042.80151195394111%5D%2C%20%5B13.261345031099777%2C%2042.80406034694114%5D%2C%20%5B13.264160102100172%2C%2042.80877672094112%5D%2C%20%5B13.263136153099937%2C%2042.81476621094117%5D%2C%20%5B13.26242499809982%2C%2042.81616126094114%5D%2C%20%5B13.266941409100546%2C%2042.81692460194108%5D%2C%20%5B13.27243086210145%2C%2042.81877232494115%5D%2C%20%5B13.27658371510209%2C%2042.822524511941104%5D%2C%20%5B13.276839420102094%2C%2042.82343779694116%5D%2C%20%5B13.27855478910239%2C%2042.82333758194114%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%225%22%2C%20%22properties%22%3A%20%7B%22COD_COM%22%3A%20%22006%22%2C%20%22COD_PROV%22%3A%20%22044%22%2C%20%22COD_REG%22%3A%20%2211%22%2C%20%22COMUNE%22%3A%20%22Arquata%20del%20Tronto%22%2C%20%22GlobalID%22%3A%20%22fa22b15d-f0f7-4458-bb57-c380813c0721%22%2C%20%22PROVINCIA%22%3A%20%22AP%22%2C%20%22Sum_ABITNO%22%3A%2051.0%2C%20%22Sum_ABIT_R%22%3A%20673.0%2C%20%22Sum_ABIT_V%22%3A%20810.0%2C%20%22Sum_ALBERG%22%3A%209.0%2C%20%22Sum_ALTRIA%22%3A%200.0%2C%20%22Sum_EDIF_A%22%3A%201199.0%2C%20%22Sum_EDIF_T%22%3A%201384.0%2C%20%22Sum_EDIF_U%22%3A%201261.0%2C%20%22Sum_FAMRES%22%3A%20673.0%2C%20%22Sum_RES_0_%22%3A%2041.0%2C%20%22Sum_RES_10%22%3A%2064.0%2C%20%22Sum_RES_15%22%3A%2072.0%2C%20%22Sum_RES_20%22%3A%2060.0%2C%20%22Sum_RES_25%22%3A%2062.0%2C%20%22Sum_RES_30%22%3A%2096.0%2C%20%22Sum_RES_35%22%3A%20103.0%2C%20%22Sum_RES_40%22%3A%20107.0%2C%20%22Sum_RES_45%22%3A%2088.0%2C%20%22Sum_RES_50%22%3A%2072.0%2C%20%22Sum_RES_55%22%3A%2079.0%2C%20%22Sum_RES_5_%22%3A%2049.0%2C%20%22Sum_RES_60%22%3A%2079.0%2C%20%22Sum_RES_65%22%3A%20102.0%2C%20%22Sum_RES_70%22%3A%20121.0%2C%20%22Sum_RES_PI%22%3A%20286.0%2C%20%22Sum_RES_TO%22%3A%201481.0%2C%20%22Sum_STRAN_%22%3A%205.0%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22bbox%22%3A%20%5B13.21610994109104%2C%2042.890237031941524%2C%2013.4112307691253%2C%2042.96729276994153%5D%2C%20%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B13.24294743209483%2C%2042.96729276994153%5D%2C%20%5B13.245098388095245%2C%2042.96450285794152%5D%2C%20%5B13.250903562096152%2C%2042.96514234594154%5D%2C%20%5B13.253232654096482%2C%2042.96519522094152%5D%2C%20%5B13.258802181097401%2C%2042.96636977394157%5D%2C%20%5B13.264898160098397%2C%2042.96629094294158%5D%2C%20%5B13.268884254099007%2C%2042.965840467941554%5D%2C%20%5B13.271255922099432%2C%2042.96520262694156%5D%2C%20%5B13.28242945910132%2C%2042.96111100594153%5D%2C%20%5B13.283442125101505%2C%2042.96128462394158%5D%2C%20%5B13.285445585101794%2C%2042.962451956941564%5D%2C%20%5B13.286892653102033%2C%2042.96199702294157%5D%2C%20%5B13.288810208102383%2C%2042.96269934594152%5D%2C%20%5B13.289920529102572%2C%2042.96243707994153%5D%2C%20%5B13.293551341103157%2C%2042.96283189194156%5D%2C%20%5B13.298276566103999%2C%2042.961758390941576%5D%2C%20%5B13.304749623105131%2C%2042.95956096394153%5D%2C%20%5B13.307046091105514%2C%2042.959884066941626%5D%2C%20%5B13.310762778106154%2C%2042.96113490994164%5D%2C%20%5B13.311882247106347%2C%2042.96101613794161%5D%2C%20%5B13.31401839010671%2C%2042.960021662941635%5D%2C%20%5B13.315849766107016%2C%2042.95983995594156%5D%2C%20%5B13.321780899108079%2C%2042.961758542941624%5D%2C%20%5B13.32272868910824%2C%2042.96162815194159%5D%2C%20%5B13.325279089108673%2C%2042.960239651941556%5D%2C%20%5B13.327115342109023%2C%2042.96021513694154%5D%2C%20%5B13.329897538109526%2C%2042.95902032994159%5D%2C%20%5B13.331311805109735%2C%2042.9597725189416%5D%2C%20%5B13.332181974109872%2C%2042.96161221494165%5D%2C%20%5B13.333764569110192%2C%2042.958383029941594%5D%2C%20%5B13.334936924110465%2C%2042.958113470941626%5D%2C%20%5B13.336345844110667%2C%2042.95858219694163%5D%2C%20%5B13.337312395110883%2C%2042.95820788294162%5D%2C%20%5B13.338057098111019%2C%2042.95832816394158%5D%2C%20%5B13.339277883111214%2C%2042.95944322994156%5D%2C%20%5B13.341383084111596%2C%2042.96000698994165%5D%2C%20%5B13.343885435111998%2C%2042.9613569039416%5D%2C%20%5B13.344791505112163%2C%2042.96093980994167%5D%2C%20%5B13.344576718112155%2C%2042.95793637494157%5D%2C%20%5B13.347958135112814%2C%2042.95625016894159%5D%2C%20%5B13.349069920113044%2C%2042.95680657194164%5D%2C%20%5B13.352052519113546%2C%2042.95575665294156%5D%2C%20%5B13.353326572113803%2C%2042.95671198094166%5D%2C%20%5B13.355769754114254%2C%2042.955651005941625%5D%2C%20%5B13.357691899114636%2C%2042.956225943941654%5D%2C%20%5B13.358640993114758%2C%2042.95695051794158%5D%2C%20%5B13.361068380115219%2C%2042.95738907894159%5D%2C%20%5B13.361820726115377%2C%2042.956761639941604%5D%2C%20%5B13.362319608115486%2C%2042.95508598794164%5D%2C%20%5B13.36577475411619%2C%2042.95283823594163%5D%2C%20%5B13.364672625115988%2C%2042.951867476941615%5D%2C%20%5B13.364979275116026%2C%2042.951405596941676%5D%2C%20%5B13.366195178116241%2C%2042.95224145794166%5D%2C%20%5B13.367236851116468%2C%2042.95222865394166%5D%2C%20%5B13.370739191117039%2C%2042.95517834394166%5D%2C%20%5B13.37188231111733%2C%2042.95412173694162%5D%2C%20%5B13.371607584117244%2C%2042.953380475941636%5D%2C%20%5B13.372405857117405%2C%2042.952409084941635%5D%2C%20%5B13.372932707117517%2C%2042.953356772941596%5D%2C%20%5B13.37532694611797%2C%2042.954138423941615%5D%2C%20%5B13.37602738911808%2C%2042.954066580941685%5D%2C%20%5B13.376179113118155%2C%2042.95343053994166%5D%2C%20%5B13.377644591118429%2C%2042.95322584194166%5D%2C%20%5B13.378061063118482%2C%2042.9541282169417%5D%2C%20%5B13.37890400911862%2C%2042.95423996394169%5D%2C%20%5B13.379899462118852%2C%2042.953220448941636%5D%2C%20%5B13.381597223119178%2C%2042.95297528294167%5D%2C%20%5B13.382569878119362%2C%2042.953284601941675%5D%2C%20%5B13.383911668119568%2C%2042.954475572941625%5D%2C%20%5B13.385142640119836%2C%2042.954743445941645%5D%2C%20%5B13.384714800119733%2C%2042.95330583894169%5D%2C%20%5B13.387429009120282%2C%2042.953102600941676%5D%2C%20%5B13.38940458012066%2C%2042.95399456594165%5D%2C%20%5B13.390499033120895%2C%2042.953367298941664%5D%2C%20%5B13.391860386121149%2C%2042.952873813941686%5D%2C%20%5B13.39207790012122%2C%2042.95019598494167%5D%2C%20%5B13.394849887121802%2C%2042.94872088894172%5D%2C%20%5B13.393635946121615%2C%2042.94495019594161%5D%2C%20%5B13.395230352121944%2C%2042.94253901694161%5D%2C%20%5B13.392658937121485%2C%2042.94150349994165%5D%2C%20%5B13.39195142012133%2C%2042.9404413049416%5D%2C%20%5B13.392217133121447%2C%2042.93919313994165%5D%2C%20%5B13.394447494121893%2C%2042.937324726941625%5D%2C%20%5B13.396741869122309%2C%2042.937204942941634%5D%2C%20%5B13.39699794712237%2C%2042.93631726894155%5D%2C%20%5B13.399092204122812%2C%2042.93494918294162%5D%2C%20%5B13.402478268123485%2C%2042.9338599079416%5D%2C%20%5B13.403569585123744%2C%2042.93289501494163%5D%2C%20%5B13.404924140123988%2C%2042.932635722941555%5D%2C%20%5B13.405027838124072%2C%2042.9309886289416%5D%2C%20%5B13.405623935124176%2C%2042.93023187894162%5D%2C%20%5B13.409141435124878%2C%2042.92972254994168%5D%2C%20%5B13.409476517124974%2C%2042.93051090894161%5D%2C%20%5B13.410734716125228%2C%2042.93068292194163%5D%2C%20%5B13.410453947125156%2C%2042.92965838594159%5D%2C%20%5B13.4112307691253%2C%2042.92902067494165%5D%2C%20%5B13.409944813125044%2C%2042.928984787941566%5D%2C%20%5B13.409224472124935%2C%2042.92840937094164%5D%2C%20%5B13.409313146124951%2C%2042.92780723194166%5D%2C%20%5B13.41022726712513%2C%2042.927033681941644%5D%2C%20%5B13.409538275125039%2C%2042.924764442941665%5D%2C%20%5B13.409024342124962%2C%2042.92428009394161%5D%2C%20%5B13.407671902124688%2C%2042.923976647941586%5D%2C%20%5B13.40737401412469%2C%2042.92174632594161%5D%2C%20%5B13.40378384912403%2C%2042.9189136819416%5D%2C%20%5B13.402507198123843%2C%2042.91379950194161%5D%2C%20%5B13.401798864123698%2C%2042.91275540294155%5D%2C%20%5B13.39998231512333%2C%2042.91240221794153%5D%2C%20%5B13.398843107123117%2C%2042.91287373894158%5D%2C%20%5B13.39629211112259%2C%2042.91249474894156%5D%2C%20%5B13.394498995122241%2C%2042.91268527394152%5D%2C%20%5B13.394332973122252%2C%2042.91199840594157%5D%2C%20%5B13.393608892122108%2C%2042.91298960394153%5D%2C%20%5B13.393337677122066%2C%2042.91259938694153%5D%2C%20%5B13.394046367122183%2C%2042.91123064494151%5D%2C%20%5B13.392207798121854%2C%2042.91035148894155%5D%2C%20%5B13.391690965121768%2C%2042.909489032941565%5D%2C%20%5B13.389668650121392%2C%2042.910007907941555%5D%2C%20%5B13.388632282121169%2C%2042.90886828794154%5D%2C%20%5B13.386775635120804%2C%2042.90817430594147%5D%2C%20%5B13.384940192120455%2C%2042.90812321494157%5D%2C%20%5B13.383070695120155%2C%2042.90606116394154%5D%2C%20%5B13.382026139119942%2C%2042.90617325494157%5D%2C%20%5B13.381207095119766%2C%2042.905655462941546%5D%2C%20%5B13.380046169119584%2C%2042.90555141694147%5D%2C%20%5B13.379847773119527%2C%2042.90422203094151%5D%2C%20%5B13.380999430119807%2C%2042.90283638794144%5D%2C%20%5B13.383040296120221%2C%2042.898774155941496%5D%2C%20%5B13.384142354120515%2C%2042.894567846941506%5D%2C%20%5B13.382531285120253%2C%2042.89295953694149%5D%2C%20%5B13.38134281411999%2C%2042.89296009794151%5D%2C%20%5B13.380197862119772%2C%2042.89227922694146%5D%2C%20%5B13.378573788119457%2C%2042.892093881941506%5D%2C%20%5B13.37749653411928%2C%2042.89067211794149%5D%2C%20%5B13.375339796118872%2C%2042.890237031941524%5D%2C%20%5B13.374073951118648%2C%2042.89088421694143%5D%2C%20%5B13.373692678118529%2C%2042.89233033894145%5D%2C%20%5B13.37076975211793%2C%2042.89591746294148%5D%2C%20%5B13.369490302117713%2C%2042.89611495194153%5D%2C%20%5B13.369399507117668%2C%2042.89769850594152%5D%2C%20%5B13.368335173117423%2C%2042.89813534694149%5D%2C%20%5B13.366113205116987%2C%2042.902496848941496%5D%2C%20%5B13.366303053116985%2C%2042.9041101909415%5D%2C%20%5B13.36555352811683%2C%2042.90423786694147%5D%2C%20%5B13.36464563311666%2C%2042.903781869941525%5D%2C%20%5B13.3628285731163%2C%2042.90533231094153%5D%2C%20%5B13.362463118116148%2C%2042.90943376994154%5D%2C%20%5B13.359719970115663%2C%2042.90986258994146%5D%2C%20%5B13.359922912115666%2C%2042.91098026094158%5D%2C%20%5B13.359229699115518%2C%2042.91355464894146%5D%2C%20%5B13.356031700114894%2C%2042.91542775894149%5D%2C%20%5B13.35231898911421%2C%2042.91588885994148%5D%2C%20%5B13.347932069113408%2C%2042.915006990941464%5D%2C%20%5B13.346900161113185%2C%2042.91648678594157%5D%2C%20%5B13.345507044112903%2C%2042.917273547941505%5D%2C%20%5B13.343072459112468%2C%2042.91577252394152%5D%2C%20%5B13.34105128411212%2C%2042.91621846694159%5D%2C%20%5B13.339423795111827%2C%2042.916122729941506%5D%2C%20%5B13.338894203111698%2C%2042.91716021794146%5D%2C%20%5B13.336661664111329%2C%2042.917235969941494%5D%2C%20%5B13.335349445111097%2C%2042.91640794094151%5D%2C%20%5B13.334261524110884%2C%2042.91630966194148%5D%2C%20%5B13.332485774110568%2C%2042.917646477941496%5D%2C%20%5B13.331036975110289%2C%2042.918210091941454%5D%2C%20%5B13.328240761109786%2C%2042.91833855494149%5D%2C%20%5B13.328965327109866%2C%2042.92008926594148%5D%2C%20%5B13.328533538109765%2C%2042.92154165594147%5D%2C%20%5B13.327452070109588%2C%2042.921208981941554%5D%2C%20%5B13.326855519109479%2C%2042.92152418194153%5D%2C%20%5B13.322796481108771%2C%2042.92121414594154%5D%2C%20%5B13.322823845108775%2C%2042.92198739794149%5D%2C%20%5B13.321222714108469%2C%2042.92185438994147%5D%2C%20%5B13.320851607108409%2C%2042.922885801941526%5D%2C%20%5B13.319326736108115%2C%2042.92262833894142%5D%2C%20%5B13.314479976107307%2C%2042.92290148694149%5D%2C%20%5B13.312110201106847%2C%2042.92350413394151%5D%2C%20%5B13.312028050106814%2C%2042.92426801794151%5D%2C%20%5B13.310660380106603%2C%2042.924112545941504%5D%2C%20%5B13.310512079106617%2C%2042.9236994849415%5D%2C%20%5B13.305999429105796%2C%2042.92294678994142%5D%2C%20%5B13.302605846105214%2C%2042.92336273394146%5D%2C%20%5B13.300904688104936%2C%2042.9228820759415%5D%2C%20%5B13.300378795104857%2C%2042.92325750194144%5D%2C%20%5B13.299846722104732%2C%2042.92256625994149%5D%2C%20%5B13.297683305104389%2C%2042.92326438094141%5D%2C%20%5B13.297029086104256%2C%2042.92431536894151%5D%2C%20%5B13.295821881104027%2C%2042.924099663941504%5D%2C%20%5B13.295268976103934%2C%2042.9246156349415%5D%2C%20%5B13.294803494103865%2C%2042.92418746494144%5D%2C%20%5B13.293867094103692%2C%2042.92524453894146%5D%2C%20%5B13.292633729103468%2C%2042.925511462941536%5D%2C%20%5B13.291826519103326%2C%2042.92546525694145%5D%2C%20%5B13.290693306103144%2C%2042.92478304494142%5D%2C%20%5B13.283243959102006%2C%2042.91738264994141%5D%2C%20%5B13.281183180101745%2C%2042.91163468994142%5D%2C%20%5B13.273312968100463%2C%2042.91048874094143%5D%2C%20%5B13.26969554409983%2C%2042.91060595694138%5D%2C%20%5B13.264820766099028%2C%2042.9118819119414%5D%2C%20%5B13.259941061098225%2C%2042.911613758941385%5D%2C%20%5B13.259464153098175%2C%2042.91070418594139%5D%2C%20%5B13.259873757098239%2C%2042.9083525339414%5D%2C%20%5B13.261076953098451%2C%2042.90693015394142%5D%2C%20%5B13.26167614709861%2C%2042.90304534794136%5D%2C%20%5B13.263105955098895%2C%2042.900106423941416%5D%2C%20%5B13.254924165097599%2C%2042.899677582941365%5D%2C%20%5B13.252647708097182%2C%2042.89916359494136%5D%2C%20%5B13.249576014096728%2C%2042.897792299941344%5D%2C%20%5B13.247126337096358%2C%2042.8951913459413%5D%2C%20%5B13.246004137096199%2C%2042.893306314941334%5D%2C%20%5B13.241779468095553%2C%2042.892000147941275%5D%2C%20%5B13.2383316260949%2C%2042.89911473594131%5D%2C%20%5B13.233226505094054%2C%2042.90462048894137%5D%2C%20%5B13.234046544094149%2C%2042.90937993194139%5D%2C%20%5B13.23249812109384%2C%2042.91254347494139%5D%2C%20%5B13.234172301094102%2C%2042.913089184941406%5D%2C%20%5B13.234756346094171%2C%2042.913657266941385%5D%2C%20%5B13.234090025094076%2C%2042.91426717994137%5D%2C%20%5B13.230850414093588%2C%2042.914472702941374%5D%2C%20%5B13.230120376093446%2C%2042.915125467941365%5D%2C%20%5B13.229959392093416%2C%2042.916049775941396%5D%2C%20%5B13.230889612093533%2C%2042.91737935694137%5D%2C%20%5B13.231319795093574%2C%2042.91987988494144%5D%2C%20%5B13.23039893309341%2C%2042.92296613894143%5D%2C%20%5B13.225756219092627%2C%2042.92761262294143%5D%2C%20%5B13.22270638709217%2C%2042.926707979941376%5D%2C%20%5B13.219607167091608%2C%2042.93346699094146%5D%2C%20%5B13.21610994109104%2C%2042.938322876941456%5D%2C%20%5B13.218938807091401%2C%2042.94518260694142%5D%2C%20%5B13.218196036091223%2C%2042.94841074194154%5D%2C%20%5B13.21614424609088%2C%2042.95111546394143%5D%2C%20%5B13.219662900091384%2C%2042.954947075941504%5D%2C%20%5B13.223506347091934%2C%2042.95812282694148%5D%2C%20%5B13.232169557093238%2C%2042.96171424794152%5D%2C%20%5B13.235241161093704%2C%2042.96406736294147%5D%2C%20%5B13.239347728094286%2C%2042.96537811994152%5D%2C%20%5B13.24294743209483%2C%2042.96729276994153%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%226%22%2C%20%22properties%22%3A%20%7B%22COD_COM%22%3A%20%22037%22%2C%20%22COD_PROV%22%3A%20%22044%22%2C%20%22COD_REG%22%3A%20%2211%22%2C%20%22COMUNE%22%3A%20%22Montefortino%22%2C%20%22GlobalID%22%3A%20%22ec4dd0e9-050c-46cd-a9b9-37e770d23eaf%22%2C%20%22PROVINCIA%22%3A%20%22AP%22%2C%20%22Sum_ABITNO%22%3A%200.0%2C%20%22Sum_ABIT_R%22%3A%20530.0%2C%20%22Sum_ABIT_V%22%3A%20595.0%2C%20%22Sum_ALBERG%22%3A%203.0%2C%20%22Sum_ALTRIA%22%3A%200.0%2C%20%22Sum_EDIF_A%22%3A%20652.0%2C%20%22Sum_EDIF_T%22%3A%20692.0%2C%20%22Sum_EDIF_U%22%3A%20675.0%2C%20%22Sum_FAMRES%22%3A%20530.0%2C%20%22Sum_RES_0_%22%3A%2050.0%2C%20%22Sum_RES_10%22%3A%2054.0%2C%20%22Sum_RES_15%22%3A%2059.0%2C%20%22Sum_RES_20%22%3A%2072.0%2C%20%22Sum_RES_25%22%3A%2085.0%2C%20%22Sum_RES_30%22%3A%2087.0%2C%20%22Sum_RES_35%22%3A%20103.0%2C%20%22Sum_RES_40%22%3A%2087.0%2C%20%22Sum_RES_45%22%3A%20107.0%2C%20%22Sum_RES_50%22%3A%2064.0%2C%20%22Sum_RES_55%22%3A%2064.0%2C%20%22Sum_RES_5_%22%3A%2060.0%2C%20%22Sum_RES_60%22%3A%2080.0%2C%20%22Sum_RES_65%22%3A%2076.0%2C%20%22Sum_RES_70%22%3A%2092.0%2C%20%22Sum_RES_PI%22%3A%20163.0%2C%20%22Sum_RES_TO%22%3A%201303.0%2C%20%22Sum_STRAN_%22%3A%2012.0%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22bbox%22%3A%20%5B13.27430177010146%2C%2042.79976597194113%2C%2013.40066301012426%2C%2042.882804705941396%5D%2C%20%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B13.365072558117106%2C%2042.87883535094142%5D%2C%20%5B13.36658060011742%2C%2042.8778278819415%5D%2C%20%5B13.366167036117382%2C%2042.87634913194143%5D%2C%20%5B13.363777540116924%2C%2042.873172195941336%5D%2C%20%5B13.360627038116375%2C%2042.87146928594138%5D%2C%20%5B13.359827938116252%2C%2042.87019429294141%5D%2C%20%5B13.359353019116204%2C%2042.867353848941384%5D%2C%20%5B13.360771926116447%2C%2042.86598522194138%5D%2C%20%5B13.361500493116608%2C%2042.86577284794141%5D%2C%20%5B13.361955512116714%2C%2042.86475160694139%5D%2C%20%5B13.36158368011667%2C%2042.864468689941326%5D%2C%20%5B13.363041969116937%2C%2042.863031006941426%5D%2C%20%5B13.364022349117107%2C%2042.86343923694135%5D%2C%20%5B13.364972095117292%2C%2042.86268719694137%5D%2C%20%5B13.366873554117667%2C%2042.862533519941344%5D%2C%20%5B13.367858263117887%2C%2042.86186565194137%5D%2C%20%5B13.369939696118285%2C%2042.86186710394141%5D%2C%20%5B13.3706606991184%2C%2042.86097520994143%5D%2C%20%5B13.372489236118803%2C%2042.86079721794139%5D%2C%20%5B13.373819092119033%2C%2042.86188524794145%5D%2C%20%5B13.37454093211914%2C%2042.861925141941406%5D%2C%20%5B13.37528542211926%2C%2042.8629230199414%5D%2C%20%5B13.377067382119616%2C%2042.86358854994136%5D%2C%20%5B13.37680360511957%2C%2042.864314417941394%5D%2C%20%5B13.377252611119635%2C%2042.864940957941364%5D%2C%20%5B13.376560775119508%2C%2042.86609287394142%5D%2C%20%5B13.377523546119663%2C%2042.86675825594135%5D%2C%20%5B13.378088222119786%2C%2042.86667359894142%5D%2C%20%5B13.378119186119738%2C%2042.86737017094145%5D%2C%20%5B13.379948354120119%2C%2042.868105872941385%5D%2C%20%5B13.381763135120442%2C%2042.86986847494143%5D%2C%20%5B13.38365047012081%2C%2042.86931891194136%5D%2C%20%5B13.383349625120738%2C%2042.86866870894149%5D%2C%20%5B13.383922290120848%2C%2042.86828660694144%5D%2C%20%5B13.384425408121006%2C%2042.8668267719414%5D%2C%20%5B13.385768596121252%2C%2042.865595791941374%5D%2C%20%5B13.388113518121738%2C%2042.864835020941356%5D%2C%20%5B13.389456459122025%2C%2042.86498151594146%5D%2C%20%5B13.389727621122072%2C%2042.864426397941344%5D%2C%20%5B13.391429933122355%2C%2042.8648831839414%5D%2C%20%5B13.390459133122208%2C%2042.86593786294142%5D%2C%20%5B13.392003203122485%2C%2042.865878526941486%5D%2C%20%5B13.392345886122534%2C%2042.865415188941355%5D%2C%20%5B13.393902659122867%2C%2042.864878162941466%5D%2C%20%5B13.39466844412299%2C%2042.86554647094145%5D%2C%20%5B13.395890525123285%2C%2042.86470267194143%5D%2C%20%5B13.396666060123483%2C%2042.862624579941425%5D%2C%20%5B13.40066301012426%2C%2042.86238069094143%5D%2C%20%5B13.399751863124129%2C%2042.858314765941415%5D%2C%20%5B13.398026286123788%2C%2042.85784099294137%5D%2C%20%5B13.395388767123233%2C%2042.85853669294139%5D%2C%20%5B13.39419758812302%2C%2042.858231374941454%5D%2C%20%5B13.391919760122564%2C%2042.85967394394144%5D%2C%20%5B13.391725790122543%2C%2042.85865501894138%5D%2C%20%5B13.390178778122264%2C%2042.856967818941364%5D%2C%20%5B13.388744400121974%2C%2042.85751359794139%5D%2C%20%5B13.388462597121919%2C%2042.856592575941384%5D%2C%20%5B13.387364381121694%2C%2042.85741353294138%5D%2C%20%5B13.3867069071216%2C%2042.856781530941404%5D%2C%20%5B13.383808144121028%2C%2042.85576735494136%5D%2C%20%5B13.38317368212089%2C%2042.85711068094138%5D%2C%20%5B13.38245545812074%2C%2042.85603081794138%5D%2C%20%5B13.380048806120271%2C%2042.856780331941415%5D%2C%20%5B13.380690460120404%2C%2042.85563482094134%5D%2C%20%5B13.380305110120359%2C%2042.85447015494142%5D%2C%20%5B13.38119434012058%2C%2042.85417496994139%5D%2C%20%5B13.380109130120331%2C%2042.853527814941366%5D%2C%20%5B13.378659480120076%2C%2042.853263751941334%5D%2C%20%5B13.381043574120564%2C%2042.85010223794137%5D%2C%20%5B13.380883871120544%2C%2042.84823566494136%5D%2C%20%5B13.377870909120045%2C%2042.84646492994135%5D%2C%20%5B13.375818389119651%2C%2042.84585928694137%5D%2C%20%5B13.3743557911194%2C%2042.84329980994136%5D%2C%20%5B13.375519050119628%2C%2042.84046873994137%5D%2C%20%5B13.37918643512043%2C%2042.83640730494131%5D%2C%20%5B13.381180603120882%2C%2042.83267100994139%5D%2C%20%5B13.379598571120615%2C%2042.82999913194129%5D%2C%20%5B13.378912238120451%2C%2042.830034434941325%5D%2C%20%5B13.377938869120252%2C%2042.83077849094128%5D%2C%20%5B13.375259559119765%2C%2042.830318388941286%5D%2C%20%5B13.371662345119042%2C%2042.83077116894129%5D%2C%20%5B13.370025747118762%2C%2042.82880350394133%5D%2C%20%5B13.367537776118294%2C%2042.828506966941276%5D%2C%20%5B13.364149510117695%2C%2042.82524213494125%5D%2C%20%5B13.362443337117382%2C%2042.82498315294132%5D%2C%20%5B13.361184522117183%2C%2042.82353213594128%5D%2C%20%5B13.361364881117204%2C%2042.821805607941236%5D%2C%20%5B13.358370879116688%2C%2042.81985354994128%5D%2C%20%5B13.35722461611649%2C%2042.81832616794129%5D%2C%20%5B13.352718692115669%2C%2042.815738329941205%5D%2C%20%5B13.351472419115455%2C%2042.81431823494118%5D%2C%20%5B13.349175407115071%2C%2042.81283457694118%5D%2C%20%5B13.347666819114764%2C%2042.812531838941226%5D%2C%20%5B13.347375635114727%2C%2042.81153003394123%5D%2C%20%5B13.347855478114813%2C%2042.81094006494121%5D%2C%20%5B13.347037478114723%2C%2042.80861679194126%5D%2C%20%5B13.346758803114712%2C%2042.80528712994122%5D%2C%20%5B13.344287522114268%2C%2042.80441772694118%5D%2C%20%5B13.343092819114048%2C%2042.802887535941196%5D%2C%20%5B13.341304780113772%2C%2042.802518800941236%5D%2C%20%5B13.335697358112702%2C%2042.803208834941216%5D%2C%20%5B13.329539894111583%2C%2042.80162806094113%5D%2C%20%5B13.32810127111133%2C%2042.80011145094117%5D%2C%20%5B13.32711967011118%2C%2042.79976597194113%5D%2C%20%5B13.323086381110421%2C%2042.80194436794113%5D%2C%20%5B13.322749581110342%2C%2042.80265937294118%5D%2C%20%5B13.321487835110156%2C%2042.803184267941205%5D%2C%20%5B13.320102607109911%2C%2042.80245333694115%5D%2C%20%5B13.318238970109517%2C%2042.80348712894117%5D%2C%20%5B13.317843354109444%2C%2042.80408279694113%5D%2C%20%5B13.318062466109488%2C%2042.80548355494117%5D%2C%20%5B13.315218324108953%2C%2042.80663986994111%5D%2C%20%5B13.31504870910891%2C%2042.807474588941155%5D%2C%20%5B13.31364720710868%2C%2042.80866192494118%5D%2C%20%5B13.31319658310856%2C%2042.81174009594118%5D%2C%20%5B13.309451823107848%2C%2042.814073687941196%5D%2C%20%5B13.306981919107372%2C%2042.81627809394114%5D%2C%20%5B13.306018214107201%2C%2042.816643030941165%5D%2C%20%5B13.30249065410657%2C%2042.816807389941175%5D%2C%20%5B13.301371208106378%2C%2042.81926694694117%5D%2C%20%5B13.299319373105975%2C%2042.820937755941195%5D%2C%20%5B13.289821507104305%2C%2042.82334292194114%5D%2C%20%5B13.284251431103389%2C%2042.823083671941184%5D%2C%20%5B13.280951271102797%2C%2042.822518565941145%5D%2C%20%5B13.27855478910239%2C%2042.82333758194114%5D%2C%20%5B13.276839420102094%2C%2042.82343779694116%5D%2C%20%5B13.27513761510181%2C%2042.824365796941144%5D%2C%20%5B13.274888616101755%2C%2042.82495583294115%5D%2C%20%5B13.2747900691017%2C%2042.82894354494122%5D%2C%20%5B13.27552195110179%2C%2042.83299923194123%5D%2C%20%5B13.274731244101611%2C%2042.83545523694126%5D%2C%20%5B13.27430177010146%2C%2042.84210233394122%5D%2C%20%5B13.275524854101635%2C%2042.84475311394122%5D%2C%20%5B13.27904787310219%2C%2042.84789401094127%5D%2C%20%5B13.280301017102325%2C%2042.852146220941236%5D%2C%20%5B13.283134736102754%2C%2042.85718551194127%5D%2C%20%5B13.298100299105277%2C%2042.85823525694127%5D%2C%20%5B13.301981547105985%2C%2042.85820626294132%5D%2C%20%5B13.30860595910713%2C%2042.859005597941355%5D%2C%20%5B13.319237343108941%2C%2042.8609811719413%5D%2C%20%5B13.320567533109198%2C%2042.86158363394136%5D%2C%20%5B13.323541235109664%2C%2042.86434774794139%5D%2C%20%5B13.32581727111004%2C%2042.86753434194132%5D%2C%20%5B13.34049891111257%2C%2042.87606652994144%5D%2C%20%5B13.346431041113634%2C%2042.87859607594143%5D%2C%20%5B13.348773841114076%2C%2042.87888958294144%5D%2C%20%5B13.350949425114464%2C%2042.87856817794138%5D%2C%20%5B13.353134897114861%2C%2042.879745411941414%5D%2C%20%5B13.3549826121152%2C%2042.87978754994142%5D%2C%20%5B13.356616320115487%2C%2042.88049506094139%5D%2C%20%5B13.359371625115983%2C%2042.880961690941376%5D%2C%20%5B13.361647937116393%2C%2042.88193722094139%5D%2C%20%5B13.361918821116452%2C%2042.882804705941396%5D%2C%20%5B13.362811049116635%2C%2042.88241501194143%5D%2C%20%5B13.36384354411686%2C%2042.87992667994141%5D%2C%20%5B13.364634843117035%2C%2042.879657868941365%5D%2C%20%5B13.365072558117106%2C%2042.87883535094142%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%227%22%2C%20%22properties%22%3A%20%7B%22COD_COM%22%3A%20%22038%22%2C%20%22COD_PROV%22%3A%20%22044%22%2C%20%22COD_REG%22%3A%20%2211%22%2C%20%22COMUNE%22%3A%20%22Montegallo%22%2C%20%22GlobalID%22%3A%20%22c0bd007b-87cc-4278-9282-b8c30d3d2deb%22%2C%20%22PROVINCIA%22%3A%20%22AP%22%2C%20%22Sum_ABITNO%22%3A%2015.0%2C%20%22Sum_ABIT_R%22%3A%20331.0%2C%20%22Sum_ABIT_V%22%3A%20681.0%2C%20%22Sum_ALBERG%22%3A%203.0%2C%20%22Sum_ALTRIA%22%3A%200.0%2C%20%22Sum_EDIF_A%22%3A%20669.0%2C%20%22Sum_EDIF_T%22%3A%20839.0%2C%20%22Sum_EDIF_U%22%3A%20736.0%2C%20%22Sum_FAMRES%22%3A%20331.0%2C%20%22Sum_RES_0_%22%3A%209.0%2C%20%22Sum_RES_10%22%3A%2014.0%2C%20%22Sum_RES_15%22%3A%2025.0%2C%20%22Sum_RES_20%22%3A%2040.0%2C%20%22Sum_RES_25%22%3A%2035.0%2C%20%22Sum_RES_30%22%3A%2031.0%2C%20%22Sum_RES_35%22%3A%2023.0%2C%20%22Sum_RES_40%22%3A%2032.0%2C%20%22Sum_RES_45%22%3A%2020.0%2C%20%22Sum_RES_50%22%3A%2034.0%2C%20%22Sum_RES_55%22%3A%2041.0%2C%20%22Sum_RES_5_%22%3A%209.0%2C%20%22Sum_RES_60%22%3A%2047.0%2C%20%22Sum_RES_65%22%3A%2033.0%2C%20%22Sum_RES_70%22%3A%2074.0%2C%20%22Sum_RES_PI%22%3A%20155.0%2C%20%22Sum_RES_TO%22%3A%20622.0%2C%20%22Sum_STRAN_%22%3A%202.0%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22bbox%22%3A%20%5B13.231351427094094%2C%2042.81616126094114%2C%2013.375339796118872%2C%2042.925511462941536%5D%2C%20%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B13.293867094103692%2C%2042.92524453894146%5D%2C%20%5B13.294803494103865%2C%2042.92418746494144%5D%2C%20%5B13.295268976103934%2C%2042.9246156349415%5D%2C%20%5B13.295821881104027%2C%2042.924099663941504%5D%2C%20%5B13.297029086104256%2C%2042.92431536894151%5D%2C%20%5B13.297683305104389%2C%2042.92326438094141%5D%2C%20%5B13.299846722104732%2C%2042.92256625994149%5D%2C%20%5B13.300378795104857%2C%2042.92325750194144%5D%2C%20%5B13.300904688104936%2C%2042.9228820759415%5D%2C%20%5B13.302605846105214%2C%2042.92336273394146%5D%2C%20%5B13.305999429105796%2C%2042.92294678994142%5D%2C%20%5B13.310512079106617%2C%2042.9236994849415%5D%2C%20%5B13.310660380106603%2C%2042.924112545941504%5D%2C%20%5B13.312028050106814%2C%2042.92426801794151%5D%2C%20%5B13.312110201106847%2C%2042.92350413394151%5D%2C%20%5B13.314479976107307%2C%2042.92290148694149%5D%2C%20%5B13.319326736108115%2C%2042.92262833894142%5D%2C%20%5B13.320851607108409%2C%2042.922885801941526%5D%2C%20%5B13.321222714108469%2C%2042.92185438994147%5D%2C%20%5B13.322823845108775%2C%2042.92198739794149%5D%2C%20%5B13.322796481108771%2C%2042.92121414594154%5D%2C%20%5B13.326855519109479%2C%2042.92152418194153%5D%2C%20%5B13.327452070109588%2C%2042.921208981941554%5D%2C%20%5B13.328533538109765%2C%2042.92154165594147%5D%2C%20%5B13.328965327109866%2C%2042.92008926594148%5D%2C%20%5B13.328240761109786%2C%2042.91833855494149%5D%2C%20%5B13.331036975110289%2C%2042.918210091941454%5D%2C%20%5B13.332485774110568%2C%2042.917646477941496%5D%2C%20%5B13.334261524110884%2C%2042.91630966194148%5D%2C%20%5B13.335349445111097%2C%2042.91640794094151%5D%2C%20%5B13.336661664111329%2C%2042.917235969941494%5D%2C%20%5B13.338894203111698%2C%2042.91716021794146%5D%2C%20%5B13.339423795111827%2C%2042.916122729941506%5D%2C%20%5B13.34105128411212%2C%2042.91621846694159%5D%2C%20%5B13.343072459112468%2C%2042.91577252394152%5D%2C%20%5B13.345507044112903%2C%2042.917273547941505%5D%2C%20%5B13.346900161113185%2C%2042.91648678594157%5D%2C%20%5B13.347932069113408%2C%2042.915006990941464%5D%2C%20%5B13.35231898911421%2C%2042.91588885994148%5D%2C%20%5B13.356031700114894%2C%2042.91542775894149%5D%2C%20%5B13.359229699115518%2C%2042.91355464894146%5D%2C%20%5B13.359922912115666%2C%2042.91098026094158%5D%2C%20%5B13.359719970115663%2C%2042.90986258994146%5D%2C%20%5B13.362463118116148%2C%2042.90943376994154%5D%2C%20%5B13.3628285731163%2C%2042.90533231094153%5D%2C%20%5B13.36464563311666%2C%2042.903781869941525%5D%2C%20%5B13.36555352811683%2C%2042.90423786694147%5D%2C%20%5B13.366303053116985%2C%2042.9041101909415%5D%2C%20%5B13.366113205116987%2C%2042.902496848941496%5D%2C%20%5B13.368335173117423%2C%2042.89813534694149%5D%2C%20%5B13.369399507117668%2C%2042.89769850594152%5D%2C%20%5B13.369490302117713%2C%2042.89611495194153%5D%2C%20%5B13.37076975211793%2C%2042.89591746294148%5D%2C%20%5B13.373692678118529%2C%2042.89233033894145%5D%2C%20%5B13.374073951118648%2C%2042.89088421694143%5D%2C%20%5B13.375339796118872%2C%2042.890237031941524%5D%2C%20%5B13.372076408118286%2C%2042.887669950941444%5D%2C%20%5B13.366682767117258%2C%2042.88696235894142%5D%2C%20%5B13.364849680117008%2C%2042.883111444941434%5D%2C%20%5B13.363774794116786%2C%2042.88299493994145%5D%2C%20%5B13.362409682116539%2C%2042.88367729794144%5D%2C%20%5B13.361918821116452%2C%2042.882804705941396%5D%2C%20%5B13.361647937116393%2C%2042.88193722094139%5D%2C%20%5B13.359371625115983%2C%2042.880961690941376%5D%2C%20%5B13.356616320115487%2C%2042.88049506094139%5D%2C%20%5B13.3549826121152%2C%2042.87978754994142%5D%2C%20%5B13.353134897114861%2C%2042.879745411941414%5D%2C%20%5B13.350949425114464%2C%2042.87856817794138%5D%2C%20%5B13.348773841114076%2C%2042.87888958294144%5D%2C%20%5B13.346431041113634%2C%2042.87859607594143%5D%2C%20%5B13.34049891111257%2C%2042.87606652994144%5D%2C%20%5B13.32581727111004%2C%2042.86753434194132%5D%2C%20%5B13.323541235109664%2C%2042.86434774794139%5D%2C%20%5B13.320567533109198%2C%2042.86158363394136%5D%2C%20%5B13.319237343108941%2C%2042.8609811719413%5D%2C%20%5B13.30860595910713%2C%2042.859005597941355%5D%2C%20%5B13.301981547105985%2C%2042.85820626294132%5D%2C%20%5B13.298100299105277%2C%2042.85823525694127%5D%2C%20%5B13.283134736102754%2C%2042.85718551194127%5D%2C%20%5B13.280301017102325%2C%2042.852146220941236%5D%2C%20%5B13.27904787310219%2C%2042.84789401094127%5D%2C%20%5B13.275524854101635%2C%2042.84475311394122%5D%2C%20%5B13.27430177010146%2C%2042.84210233394122%5D%2C%20%5B13.274731244101611%2C%2042.83545523694126%5D%2C%20%5B13.27552195110179%2C%2042.83299923194123%5D%2C%20%5B13.2747900691017%2C%2042.82894354494122%5D%2C%20%5B13.274888616101755%2C%2042.82495583294115%5D%2C%20%5B13.27513761510181%2C%2042.824365796941144%5D%2C%20%5B13.276839420102094%2C%2042.82343779694116%5D%2C%20%5B13.27658371510209%2C%2042.822524511941104%5D%2C%20%5B13.27243086210145%2C%2042.81877232494115%5D%2C%20%5B13.266941409100546%2C%2042.81692460194108%5D%2C%20%5B13.26242499809982%2C%2042.81616126094114%5D%2C%20%5B13.257325419098905%2C%2042.819155902941105%5D%2C%20%5B13.255816212098662%2C%2042.82149898894121%5D%2C%20%5B13.254894038098497%2C%2042.82384271294117%5D%2C%20%5B13.254880333098457%2C%2042.82563491294116%5D%2C%20%5B13.253398388098162%2C%2042.82743224294116%5D%2C%20%5B13.253277696098115%2C%2042.830182791941205%5D%2C%20%5B13.249677981097458%2C%2042.83613743394111%5D%2C%20%5B13.24977090009741%2C%2042.841522550941185%5D%2C%20%5B13.247678428097037%2C%2042.84757863494117%5D%2C%20%5B13.248091338097055%2C%2042.848558170941196%5D%2C%20%5B13.24772052409698%2C%2042.84998549594119%5D%2C%20%5B13.24643449509679%2C%2042.85131627794127%5D%2C%20%5B13.243411457096249%2C%2042.852612514941185%5D%2C%20%5B13.241972502096015%2C%2042.855875664941266%5D%2C%20%5B13.24350897809624%2C%2042.85867272194124%5D%2C%20%5B13.240856405095755%2C%2042.86111658094128%5D%2C%20%5B13.238678923095383%2C%2042.86452413794125%5D%2C%20%5B13.235270710094808%2C%2042.867702662941255%5D%2C%20%5B13.235013749094762%2C%2042.86897717094123%5D%2C%20%5B13.232091960094255%2C%2042.87144430994128%5D%2C%20%5B13.231351427094094%2C%2042.875100122941255%5D%2C%20%5B13.237475553095052%2C%2042.87839806094129%5D%2C%20%5B13.23955817509532%2C%2042.882691991941286%5D%2C%20%5B13.240415766095364%2C%2042.88954780294129%5D%2C%20%5B13.241779468095553%2C%2042.892000147941275%5D%2C%20%5B13.246004137096199%2C%2042.893306314941334%5D%2C%20%5B13.247126337096358%2C%2042.8951913459413%5D%2C%20%5B13.249576014096728%2C%2042.897792299941344%5D%2C%20%5B13.252647708097182%2C%2042.89916359494136%5D%2C%20%5B13.254924165097599%2C%2042.899677582941365%5D%2C%20%5B13.263105955098895%2C%2042.900106423941416%5D%2C%20%5B13.26167614709861%2C%2042.90304534794136%5D%2C%20%5B13.261076953098451%2C%2042.90693015394142%5D%2C%20%5B13.259873757098239%2C%2042.9083525339414%5D%2C%20%5B13.259464153098175%2C%2042.91070418594139%5D%2C%20%5B13.259941061098225%2C%2042.911613758941385%5D%2C%20%5B13.264820766099028%2C%2042.9118819119414%5D%2C%20%5B13.26969554409983%2C%2042.91060595694138%5D%2C%20%5B13.273312968100463%2C%2042.91048874094143%5D%2C%20%5B13.281183180101745%2C%2042.91163468994142%5D%2C%20%5B13.283243959102006%2C%2042.91738264994141%5D%2C%20%5B13.290693306103144%2C%2042.92478304494142%5D%2C%20%5B13.291826519103326%2C%2042.92546525694145%5D%2C%20%5B13.292633729103468%2C%2042.925511462941536%5D%2C%20%5B13.293867094103692%2C%2042.92524453894146%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%228%22%2C%20%22properties%22%3A%20%7B%22COD_COM%22%3A%20%22044%22%2C%20%22COD_PROV%22%3A%20%22044%22%2C%20%22COD_REG%22%3A%20%2211%22%2C%20%22COMUNE%22%3A%20%22Montemonaco%22%2C%20%22GlobalID%22%3A%20%22d701789b-af4a-4e26-a4c8-6a9c26360e2b%22%2C%20%22PROVINCIA%22%3A%20%22AP%22%2C%20%22Sum_ABITNO%22%3A%205.0%2C%20%22Sum_ABIT_R%22%3A%20293.0%2C%20%22Sum_ABIT_V%22%3A%20391.0%2C%20%22Sum_ALBERG%22%3A%208.0%2C%20%22Sum_ALTRIA%22%3A%200.0%2C%20%22Sum_EDIF_A%22%3A%20381.0%2C%20%22Sum_EDIF_T%22%3A%20525.0%2C%20%22Sum_EDIF_U%22%3A%20461.0%2C%20%22Sum_FAMRES%22%3A%20293.0%2C%20%22Sum_RES_0_%22%3A%2020.0%2C%20%22Sum_RES_10%22%3A%2031.0%2C%20%22Sum_RES_15%22%3A%2026.0%2C%20%22Sum_RES_20%22%3A%2052.0%2C%20%22Sum_RES_25%22%3A%2044.0%2C%20%22Sum_RES_30%22%3A%2031.0%2C%20%22Sum_RES_35%22%3A%2047.0%2C%20%22Sum_RES_40%22%3A%2035.0%2C%20%22Sum_RES_45%22%3A%2051.0%2C%20%22Sum_RES_50%22%3A%2047.0%2C%20%22Sum_RES_55%22%3A%2030.0%2C%20%22Sum_RES_5_%22%3A%2024.0%2C%20%22Sum_RES_60%22%3A%2055.0%2C%20%22Sum_RES_65%22%3A%2041.0%2C%20%22Sum_RES_70%22%3A%2053.0%2C%20%22Sum_RES_PI%22%3A%2097.0%2C%20%22Sum_RES_TO%22%3A%20684.0%2C%20%22Sum_STRAN_%22%3A%206.0%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22bbox%22%3A%20%5B13.1603278030858%2C%2042.64390032394059%2C%2013.35776997611835%2C%2042.74096265494097%5D%2C%20%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B13.28005436910371%2C%2042.74096265494097%5D%2C%20%5B13.282357676104098%2C%2042.73941332694093%5D%2C%20%5B13.286288620104745%2C%2042.74040044694093%5D%2C%20%5B13.287774827105045%2C%2042.739214773940944%5D%2C%20%5B13.28810885310509%2C%2042.73684332794094%5D%2C%20%5B13.290092576105472%2C%2042.73575600694097%5D%2C%20%5B13.290467593105577%2C%2042.733238959940905%5D%2C%20%5B13.294003091106246%2C%2042.73080582194093%5D%2C%20%5B13.297322626106784%2C%2042.72956915594087%5D%2C%20%5B13.300166363107314%2C%2042.729664801940906%5D%2C%20%5B13.309436303108944%2C%2042.727902184940945%5D%2C%20%5B13.314459017109888%2C%2042.72555640094091%5D%2C%20%5B13.315196638110079%2C%2042.72448415694093%5D%2C%20%5B13.317884770110574%2C%2042.72399100494096%5D%2C%20%5B13.321282775111186%2C%2042.72028372694092%5D%2C%20%5B13.323960123111721%2C%2042.718377294940964%5D%2C%20%5B13.32637170511216%2C%2042.7159001339409%5D%2C%20%5B13.328285354112587%2C%2042.711929191940904%5D%2C%20%5B13.33197723811333%2C%2042.709029788940875%5D%2C%20%5B13.333941893113703%2C%2042.708644706940895%5D%2C%20%5B13.33414976911375%2C%2042.707259293940865%5D%2C%20%5B13.33550654511398%2C%2042.70629849694087%5D%2C%20%5B13.336484162114234%2C%2042.704879384940845%5D%2C%20%5B13.338436625114555%2C%2042.70402650694087%5D%2C%20%5B13.341170935115086%2C%2042.701761876940836%5D%2C%20%5B13.344583024115789%2C%2042.7011100579409%5D%2C%20%5B13.349657459116775%2C%2042.698058502940874%5D%2C%20%5B13.351034473117048%2C%2042.69765496894084%5D%2C%20%5B13.352327285117296%2C%2042.69633176794087%5D%2C%20%5B13.354233747117696%2C%2042.69534081894087%5D%2C%20%5B13.356663937118137%2C%2042.6948250729409%5D%2C%20%5B13.35776997611835%2C%2042.69409415394087%5D%2C%20%5B13.357309543118276%2C%2042.69256310694093%5D%2C%20%5B13.35142444411729%2C%2042.686619876940895%5D%2C%20%5B13.35126739611726%2C%2042.68561746494085%5D%2C%20%5B13.352057934117429%2C%2042.683899222940845%5D%2C%20%5B13.350888658117247%2C%2042.68240413194087%5D%2C%20%5B13.348375989116736%2C%2042.685106233940864%5D%2C%20%5B13.344443090115957%2C%2042.68658826594087%5D%2C%20%5B13.34175015611539%2C%2042.688243665940824%5D%2C%20%5B13.338979967114879%2C%2042.68867295894083%5D%2C%20%5B13.335570769114248%2C%2042.6897296529408%5D%2C%20%5B13.333852788113893%2C%2042.69142442194085%5D%2C%20%5B13.332440961113655%2C%2042.69217119994085%5D%2C%20%5B13.309915358109512%2C%2042.694071488940814%5D%2C%20%5B13.308531916109233%2C%2042.69512302294086%5D%2C%20%5B13.305114131108622%2C%2042.694216367940854%5D%2C%20%5B13.300387638107825%2C%2042.693831526940826%5D%2C%20%5B13.295952952107033%2C%2042.69263421194079%5D%2C%20%5B13.294293268106768%2C%2042.69125599694083%5D%2C%20%5B13.291921548106382%2C%2042.69004854594083%5D%2C%20%5B13.291993872106422%2C%2042.68919499794078%5D%2C%20%5B13.290213472106098%2C%2042.688663083940824%5D%2C%20%5B13.28786873110571%2C%2042.68693683194079%5D%2C%20%5B13.28460622610515%2C%2042.685294444940766%5D%2C%20%5B13.2810387111046%2C%2042.681898689940766%5D%2C%20%5B13.277429598104012%2C%2042.68042213294074%5D%2C%20%5B13.272587567103184%2C%2042.67989189294077%5D%2C%20%5B13.271915896103058%2C%2042.67943980194079%5D%2C%20%5B13.26906576110261%2C%2042.67969481194076%5D%2C%20%5B13.266543716102131%2C%2042.68084236694072%5D%2C%20%5B13.266722325102188%2C%2042.67749538094068%5D%2C%20%5B13.262962908101661%2C%2042.671810299940745%5D%2C%20%5B13.263610559101826%2C%2042.6673924089407%5D%2C%20%5B13.264347569101938%2C%2042.66632951394075%5D%2C%20%5B13.26410134110192%2C%2042.6655508839407%5D%2C%20%5B13.264778782102061%2C%2042.66356283994077%5D%2C%20%5B13.264823814102094%2C%2042.66105816694073%5D%2C%20%5B13.263054256101839%2C%2042.659332442940695%5D%2C%20%5B13.262948173101792%2C%2042.65835050594066%5D%2C%20%5B13.260546788101392%2C%2042.65871011494074%5D%2C%20%5B13.261174531101505%2C%2042.657610794940666%5D%2C%20%5B13.257381235100905%2C%2042.65757201994067%5D%2C%20%5B13.255395575100541%2C%2042.659631224940675%5D%2C%20%5B13.252623496100062%2C%2042.661017365940715%5D%2C%20%5B13.251399880099855%2C%2042.661152902940735%5D%2C%20%5B13.250981241099833%2C%2042.65954331894069%5D%2C%20%5B13.250159004099677%2C%2042.65928126794069%5D%2C%20%5B13.249273592099515%2C%2042.65953476194072%5D%2C%20%5B13.248703554099436%2C%2042.660276231940664%5D%2C%20%5B13.24888060209941%2C%2042.66092691494072%5D%2C%20%5B13.248178640099303%2C%2042.66160125494073%5D%2C%20%5B13.250488598099658%2C%2042.664491096940736%5D%2C%20%5B13.24851173109932%2C%2042.66580255894068%5D%2C%20%5B13.24802897009923%2C%2042.66691893294072%5D%2C%20%5B13.24717271709909%2C%2042.666986754940694%5D%2C%20%5B13.245315464098777%2C%2042.6662183909407%5D%2C%20%5B13.241541136098187%2C%2042.66633144894071%5D%2C%20%5B13.240440940097999%2C%2042.66386925494069%5D%2C%20%5B13.239169444097826%2C%2042.663929905940684%5D%2C%20%5B13.238762615097738%2C%2042.66318418394071%5D%2C%20%5B13.237690667097558%2C%2042.66312937794073%5D%2C%20%5B13.233857829096957%2C%2042.66147063794061%5D%2C%20%5B13.231924910096708%2C%2042.65949387094062%5D%2C%20%5B13.23067427909651%2C%2042.659288047940656%5D%2C%20%5B13.230568473096493%2C%2042.65784238094062%5D%2C%20%5B13.229050937096247%2C%2042.657578886940634%5D%2C%20%5B13.228446088096169%2C%2042.6556654789406%5D%2C%20%5B13.225882998095793%2C%2042.655755726940676%5D%2C%20%5B13.224092951095495%2C%2042.652585063940634%5D%2C%20%5B13.220581705094984%2C%2042.65207995894068%5D%2C%20%5B13.21757690209449%2C%2042.652636502940645%5D%2C%20%5B13.215479233094133%2C%2042.653474657940635%5D%2C%20%5B13.21448165209401%2C%2042.652093356940675%5D%2C%20%5B13.213434131093829%2C%2042.65159624294061%5D%2C%20%5B13.210763487093487%2C%2042.646900178940605%5D%2C%20%5B13.209418367093276%2C%2042.64699471994062%5D%2C%20%5B13.208840671093181%2C%2042.64758321294065%5D%2C%20%5B13.208295041093068%2C%2042.64747274094058%5D%2C%20%5B13.20575899009271%2C%2042.6444462789406%5D%2C%20%5B13.204323449092508%2C%2042.64390032394059%5D%2C%20%5B13.198596317091614%2C%2042.64537575294064%5D%2C%20%5B13.185778930089663%2C%2042.646543409940634%5D%2C%20%5B13.178809179088564%2C%2042.64798229194058%5D%2C%20%5B13.173192068087733%2C%2042.648565566940555%5D%2C%20%5B13.171911811087572%2C%2042.64898142994057%5D%2C%20%5B13.171107413087432%2C%2042.64977608494056%5D%2C%20%5B13.169258105087085%2C%2042.6537646219406%5D%2C%20%5B13.168506257086985%2C%2042.65362546294056%5D%2C%20%5B13.167476573086839%2C%2042.652546536940555%5D%2C%20%5B13.167276544086825%2C%2042.65311205394067%5D%2C%20%5B13.1603278030858%2C%2042.65624173694065%5D%2C%20%5B13.173541739087643%2C%2042.6653941589407%5D%2C%20%5B13.174128713087704%2C%2042.66612003794065%5D%2C%20%5B13.173846970087633%2C%2042.667377327940635%5D%2C%20%5B13.17425589008768%2C%2042.66738040894062%5D%2C%20%5B13.173024111087456%2C%2042.674988435940634%5D%2C%20%5B13.179811135088439%2C%2042.677608209940715%5D%2C%20%5B13.181488924088665%2C%2042.67864081094067%5D%2C%20%5B13.18244950308876%2C%2042.68045142594069%5D%2C%20%5B13.18232418308874%2C%2042.68140139194072%5D%2C%20%5B13.180649231088497%2C%2042.68359200294072%5D%2C%20%5B13.175901896087804%2C%2042.683135218940684%5D%2C%20%5B13.173944332087487%2C%2042.68540358094076%5D%2C%20%5B13.173326204087392%2C%2042.68732140994071%5D%2C%20%5B13.177076184087827%2C%2042.69432436494076%5D%2C%20%5B13.180344213088299%2C%2042.696969046940715%5D%2C%20%5B13.182252858088576%2C%2042.699339232940716%5D%2C%20%5B13.183193918088676%2C%2042.70222648294074%5D%2C%20%5B13.186775807089147%2C%2042.70667822194079%5D%2C%20%5B13.189402121089516%2C%2042.708094628940785%5D%2C%20%5B13.19070696508969%2C%2042.71045077494073%5D%2C%20%5B13.189995597089537%2C%2042.715203739940755%5D%2C%20%5B13.187649367089138%2C%2042.72031821094084%5D%2C%20%5B13.186996767088983%2C%2042.72342134394084%5D%2C%20%5B13.187753682089053%2C%2042.7276208119408%5D%2C%20%5B13.18952220108928%2C%2042.730666762940885%5D%2C%20%5B13.189299747089247%2C%2042.73325444394087%5D%2C%20%5B13.190266638089403%2C%2042.73362417194085%5D%2C%20%5B13.19421863008999%2C%2042.73383039994082%5D%2C%20%5B13.196855280090386%2C%2042.73470152794093%5D%2C%20%5B13.200227836090889%2C%2042.73508637694093%5D%2C%20%5B13.20710503509195%2C%2042.73087193794084%5D%2C%20%5B13.224315305094683%2C%2042.72830156094081%5D%2C%20%5B13.224737659094776%2C%2042.728911701940916%5D%2C%20%5B13.230190265095665%2C%2042.72703541994086%5D%2C%20%5B13.238115079096918%2C%2042.72581455394093%5D%2C%20%5B13.239997844097232%2C%2042.72620392494089%5D%2C%20%5B13.248115718098541%2C%2042.725902570940896%5D%2C%20%5B13.249364505098756%2C%2042.72493329794085%5D%2C%20%5B13.251041515099033%2C%2042.72441177294088%5D%2C%20%5B13.250880652099049%2C%2042.72270708094085%5D%2C%20%5B13.252745724099341%2C%2042.72284930194087%5D%2C%20%5B13.254164934099586%2C%2042.72145047094085%5D%2C%20%5B13.254962322099692%2C%2042.72174942594091%5D%2C%20%5B13.257102736100066%2C%2042.723817088940834%5D%2C%20%5B13.260383146100509%2C%2042.72837222594085%5D%2C%20%5B13.259925611100462%2C%2042.7298703559409%5D%2C%20%5B13.26079507610055%2C%2042.73329981194093%5D%2C%20%5B13.26268194310083%2C%2042.73614661394096%5D%2C%20%5B13.264511484101085%2C%2042.7401794859409%5D%2C%20%5B13.267751547101598%2C%2042.74010359194093%5D%2C%20%5B13.268880723101802%2C%2042.7394717019409%5D%2C%20%5B13.269626060101938%2C%2042.7394753799409%5D%2C%20%5B13.274107519102655%2C%2042.74043335594092%5D%2C%20%5B13.277246403103202%2C%2042.740473518940966%5D%2C%20%5B13.28005436910371%2C%2042.74096265494097%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%229%22%2C%20%22properties%22%3A%20%7B%22COD_COM%22%3A%20%22001%22%2C%20%22COD_PROV%22%3A%20%22057%22%2C%20%22COD_REG%22%3A%20%2212%22%2C%20%22COMUNE%22%3A%20%22Accumoli%22%2C%20%22GlobalID%22%3A%20%22c0104446-1364-44c1-8cd8-0575a7c91c36%22%2C%20%22PROVINCIA%22%3A%20%22RI%22%2C%20%22Sum_ABITNO%22%3A%204.0%2C%20%22Sum_ABIT_R%22%3A%20344.0%2C%20%22Sum_ABIT_V%22%3A%20562.0%2C%20%22Sum_ALBERG%22%3A%204.0%2C%20%22Sum_ALTRIA%22%3A%200.0%2C%20%22Sum_EDIF_A%22%3A%20713.0%2C%20%22Sum_EDIF_T%22%3A%20988.0%2C%20%22Sum_EDIF_U%22%3A%20959.0%2C%20%22Sum_FAMRES%22%3A%20344.0%2C%20%22Sum_RES_0_%22%3A%2021.0%2C%20%22Sum_RES_10%22%3A%2025.0%2C%20%22Sum_RES_15%22%3A%2024.0%2C%20%22Sum_RES_20%22%3A%2034.0%2C%20%22Sum_RES_25%22%3A%2045.0%2C%20%22Sum_RES_30%22%3A%2053.0%2C%20%22Sum_RES_35%22%3A%2050.0%2C%20%22Sum_RES_40%22%3A%2049.0%2C%20%22Sum_RES_45%22%3A%2039.0%2C%20%22Sum_RES_50%22%3A%2035.0%2C%20%22Sum_RES_55%22%3A%2034.0%2C%20%22Sum_RES_5_%22%3A%2024.0%2C%20%22Sum_RES_60%22%3A%2055.0%2C%20%22Sum_RES_65%22%3A%2051.0%2C%20%22Sum_RES_70%22%3A%2062.0%2C%20%22Sum_RES_PI%22%3A%20123.0%2C%20%22Sum_RES_TO%22%3A%20724.0%2C%20%22Sum_STRAN_%22%3A%2027.0%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22bbox%22%3A%20%5B13.167156879086845%2C%2042.5695339039404%2C%2013.409750719129512%2C%2042.69512302294086%5D%2C%20%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B13.35085018511726%2C%2042.6797045519408%5D%2C%20%5B13.349783451117082%2C%2042.677579804940876%5D%2C%20%5B13.350451878117239%2C%2042.675402539940855%5D%2C%20%5B13.350252475117204%2C%2042.673694962940814%5D%2C%20%5B13.348601110116915%2C%2042.67338413994085%5D%2C%20%5B13.347725647116752%2C%2042.672805197940846%5D%2C%20%5B13.346764242116596%2C%2042.67133366894082%5D%2C%20%5B13.346685234116583%2C%2042.670638901940805%5D%2C%20%5B13.347784053116813%2C%2042.66876040994079%5D%2C%20%5B13.350339301117325%2C%2042.66856417394078%5D%2C%20%5B13.35189351011764%2C%2042.66702845394077%5D%2C%20%5B13.355064663118249%2C%2042.6656967369408%5D%2C%20%5B13.35719587611867%2C%2042.664008415940735%5D%2C%20%5B13.360363032119352%2C%2042.66002067694079%5D%2C%20%5B13.360442387119427%2C%2042.658388019940865%5D%2C%20%5B13.361210389119552%2C%2042.65703071394078%5D%2C%20%5B13.364332954120194%2C%2042.655646574940796%5D%2C%20%5B13.367880881120918%2C%2042.65342678494072%5D%2C%20%5B13.369923789121342%2C%2042.648968529940745%5D%2C%20%5B13.3716733381217%2C%2042.64880711294079%5D%2C%20%5B13.373467860122048%2C%2042.64968835194076%5D%2C%20%5B13.376768257122663%2C%2042.64986817994075%5D%2C%20%5B13.378760692123098%2C%2042.650201550940785%5D%2C%20%5B13.381808592123718%2C%2042.64981467894078%5D%2C%20%5B13.383879347124129%2C%2042.64851983294077%5D%2C%20%5B13.388510881125073%2C%2042.647252736940736%5D%2C%20%5B13.390521799125493%2C%2042.64590154394079%5D%2C%20%5B13.39429940012628%2C%2042.64606253894083%5D%2C%20%5B13.39870659712714%2C%2042.64689245394083%5D%2C%20%5B13.402009756127823%2C%2042.64555886394081%5D%2C%20%5B13.403425474128165%2C%2042.64409530894077%5D%2C%20%5B13.405268642128549%2C%2042.64308344694077%5D%2C%20%5B13.408287438129168%2C%2042.64286355494084%5D%2C%20%5B13.409750719129512%2C%2042.63904818794079%5D%2C%20%5B13.4093066071295%2C%2042.636368781940796%5D%2C%20%5B13.406499102128938%2C%2042.63360041194082%5D%2C%20%5B13.407340947129178%2C%2042.63118204894074%5D%2C%20%5B13.406781362129067%2C%2042.62922735794075%5D%2C%20%5B13.405885443128886%2C%2042.62846507794071%5D%2C%20%5B13.401298383127964%2C%2042.62665647494073%5D%2C%20%5B13.3997111081277%2C%2042.62434514894074%5D%2C%20%5B13.396880262127164%2C%2042.62249578494076%5D%2C%20%5B13.394632660126701%2C%2042.62170885394077%5D%2C%20%5B13.393962920126615%2C%2042.6202490189407%5D%2C%20%5B13.396608834127177%2C%2042.61645138494071%5D%2C%20%5B13.396893242127286%2C%2042.61488734894067%5D%2C%20%5B13.395420692127002%2C%2042.612391475940626%5D%2C%20%5B13.390677105126091%2C%2042.60911637994063%5D%2C%20%5B13.389827546125943%2C%2042.606114820940654%5D%2C%20%5B13.390264454126099%2C%2042.604301850940615%5D%2C%20%5B13.389005366125854%2C%2042.60164913694068%5D%2C%20%5B13.390059590126125%2C%2042.599483843940696%5D%2C%20%5B13.391035161126377%2C%2042.5933105039406%5D%2C%20%5B13.394022974127022%2C%2042.591255481940635%5D%2C%20%5B13.389404849126128%2C%2042.589902244940625%5D%2C%20%5B13.389478916126182%2C%2042.588431830940614%5D%2C%20%5B13.388993869126057%2C%2042.58739253594062%5D%2C%20%5B13.38356601512499%2C%2042.58568298294061%5D%2C%20%5B13.382934502124895%2C%2042.5851264619406%5D%2C%20%5B13.384814165125297%2C%2042.584046011940615%5D%2C%20%5B13.38432391612522%2C%2042.583245488940555%5D%2C%20%5B13.382566736124847%2C%2042.58240348894061%5D%2C%20%5B13.381785992124717%2C%2042.581010851940626%5D%2C%20%5B13.374668371123267%2C%2042.581049259940556%5D%2C%20%5B13.373082357122971%2C%2042.58063723494058%5D%2C%20%5B13.372028221122745%2C%2042.581361807940546%5D%2C%20%5B13.370443589122463%2C%2042.57996830894058%5D%2C%20%5B13.368545034122096%2C%2042.58082415694053%5D%2C%20%5B13.365703759121509%2C%2042.58087413594054%5D%2C%20%5B13.360779802120614%2C%2042.57859953594053%5D%2C%20%5B13.359240887120292%2C%2042.578271054940586%5D%2C%20%5B13.355992284119687%2C%2042.578592916940515%5D%2C%20%5B13.354462965119366%2C%2042.576490307940524%5D%2C%20%5B13.352079092118926%2C%2042.57679713694052%5D%2C%20%5B13.350903879118682%2C%2042.57745859694054%5D%2C%20%5B13.34706517211795%2C%2042.57747402394051%5D%2C%20%5B13.343668325117353%2C%2042.57507309194052%5D%2C%20%5B13.342832324117216%2C%2042.57489775994055%5D%2C%20%5B13.339519388116534%2C%2042.5753116299405%5D%2C%20%5B13.33529481711574%2C%2042.57836649894046%5D%2C%20%5B13.327639031114254%2C%2042.579876441940534%5D%2C%20%5B13.323097193113444%2C%2042.57974659894046%5D%2C%20%5B13.319976657112866%2C%2042.57706890494046%5D%2C%20%5B13.317497647112415%2C%2042.576365701940475%5D%2C%20%5B13.315657066112133%2C%2042.57639465594054%5D%2C%20%5B13.315215540112021%2C%2042.57512830494049%5D%2C%20%5B13.311944298111465%2C%2042.57400919094048%5D%2C%20%5B13.311372811111367%2C%2042.5734950229405%5D%2C%20%5B13.309779104111088%2C%2042.57385671094046%5D%2C%20%5B13.307805256110711%2C%2042.57250400594047%5D%2C%20%5B13.307439602110621%2C%2042.57385931394052%5D%2C%20%5B13.305020830110209%2C%2042.57230272794046%5D%2C%20%5B13.296997037108765%2C%2042.5720822659405%5D%2C%20%5B13.297381843108843%2C%2042.57094685794048%5D%2C%20%5B13.293290043108174%2C%2042.5695339039404%5D%2C%20%5B13.291264264107792%2C%2042.57053280094049%5D%2C%20%5B13.289483615107416%2C%2042.574160526940474%5D%2C%20%5B13.286511515106884%2C%2042.57723864994042%5D%2C%20%5B13.284302749106425%2C%2042.58226889794051%5D%2C%20%5B13.2836254641063%2C%2042.58251935794047%5D%2C%20%5B13.280472308105761%2C%2042.5823988199405%5D%2C%20%5B13.279810425105621%2C%2042.58256314594046%5D%2C%20%5B13.279364707105543%2C%2042.583259597940426%5D%2C%20%5B13.275295091104862%2C%2042.58322272494046%5D%2C%20%5B13.27012079310394%2C%2042.58216452894047%5D%2C%20%5B13.268032274103593%2C%2042.58276919994044%5D%2C%20%5B13.261552318102467%2C%2042.58309636994051%5D%2C%20%5B13.25591316610156%2C%2042.58257705794044%5D%2C%20%5B13.254748766101393%2C%2042.58194960994048%5D%2C%20%5B13.253810716101222%2C%2042.58196199394044%5D%2C%20%5B13.253315560101141%2C%2042.581791332940526%5D%2C%20%5B13.253507430101184%2C%2042.57972236294041%5D%2C%20%5B13.249978081100597%2C%2042.57825541894047%5D%2C%20%5B13.250276287100682%2C%2042.57671821894043%5D%2C%20%5B13.248736537100436%2C%2042.57559594994043%5D%2C%20%5B13.248051520100306%2C%2042.57437889094039%5D%2C%20%5B13.24732811110021%2C%2042.57410863694041%5D%2C%20%5B13.246553378100058%2C%2042.57434899094041%5D%2C%20%5B13.245967552100007%2C%2042.573952069940496%5D%2C%20%5B13.245941953099987%2C%2042.572701511940444%5D%2C%20%5B13.235693742098302%2C%2042.574909242940386%5D%2C%20%5B13.234038464097974%2C%2042.57569532694043%5D%2C%20%5B13.230617743097389%2C%2042.57864455694048%5D%2C%20%5B13.228519722097015%2C%2042.58168885094038%5D%2C%20%5B13.226793991096756%2C%2042.58283307994043%5D%2C%20%5B13.21998572009567%2C%2042.58226064894046%5D%2C%20%5B13.219115981095536%2C%2042.58242779094046%5D%2C%20%5B13.218643785095402%2C%2042.58403885894045%5D%2C%20%5B13.217453756095225%2C%2042.58428983194041%5D%2C%20%5B13.212711042094526%2C%2042.58224521094045%5D%2C%20%5B13.211900007094357%2C%2042.58403980194048%5D%2C%20%5B13.211121736094217%2C%2042.584581669940405%5D%2C%20%5B13.208750886093872%2C%2042.584362811940416%5D%2C%20%5B13.20817822709379%2C%2042.583515027940344%5D%2C%20%5B13.207136572093626%2C%2042.58330124494035%5D%2C%20%5B13.20435274909315%2C%2042.5856005109404%5D%2C%20%5B13.203039068092956%2C%2042.58595041694045%5D%2C%20%5B13.199333094092365%2C%2042.585978500940435%5D%2C%20%5B13.197180725092034%2C%2042.586665254940414%5D%2C%20%5B13.191367730091144%2C%2042.587513185940374%5D%2C%20%5B13.190253241090932%2C%2042.58897210494044%5D%2C%20%5B13.188575323090634%2C%2042.594282720940484%5D%2C%20%5B13.186597116090308%2C%2042.59820873194044%5D%2C%20%5B13.181863513089475%2C%2042.60588648794045%5D%2C%20%5B13.181164458089347%2C%2042.60980164194048%5D%2C%20%5B13.178656597088894%2C%2042.613886435940465%5D%2C%20%5B13.178108187088805%2C%2042.61807517994049%5D%2C%20%5B13.176199284088455%2C%2042.62331752094054%5D%2C%20%5B13.17571011708834%2C%2042.62669825894052%5D%2C%20%5B13.17252195408779%2C%2042.6331126989405%5D%2C%20%5B13.172655643087786%2C%2042.63440435194058%5D%2C%20%5B13.174051177088016%2C%2042.63673939994064%5D%2C%20%5B13.173894904087954%2C%2042.638275728940584%5D%2C%20%5B13.172976948087786%2C%2042.63922760394055%5D%2C%20%5B13.168939171087208%2C%2042.64135117794053%5D%2C%20%5B13.168368690087108%2C%2042.64251994194055%5D%2C%20%5B13.167764849086948%2C%2042.64479737194052%5D%2C%20%5B13.167795545086927%2C%2042.648744364940576%5D%2C%20%5B13.167156879086845%2C%2042.64970402194057%5D%2C%20%5B13.167476573086839%2C%2042.652546536940555%5D%2C%20%5B13.168506257086985%2C%2042.65362546294056%5D%2C%20%5B13.169258105087085%2C%2042.6537646219406%5D%2C%20%5B13.171107413087432%2C%2042.64977608494056%5D%2C%20%5B13.171911811087572%2C%2042.64898142994057%5D%2C%20%5B13.173192068087733%2C%2042.648565566940555%5D%2C%20%5B13.178809179088564%2C%2042.64798229194058%5D%2C%20%5B13.185778930089663%2C%2042.646543409940634%5D%2C%20%5B13.198596317091614%2C%2042.64537575294064%5D%2C%20%5B13.204323449092508%2C%2042.64390032394059%5D%2C%20%5B13.20575899009271%2C%2042.6444462789406%5D%2C%20%5B13.208295041093068%2C%2042.64747274094058%5D%2C%20%5B13.208840671093181%2C%2042.64758321294065%5D%2C%20%5B13.209418367093276%2C%2042.64699471994062%5D%2C%20%5B13.210763487093487%2C%2042.646900178940605%5D%2C%20%5B13.213434131093829%2C%2042.65159624294061%5D%2C%20%5B13.21448165209401%2C%2042.652093356940675%5D%2C%20%5B13.215479233094133%2C%2042.653474657940635%5D%2C%20%5B13.21757690209449%2C%2042.652636502940645%5D%2C%20%5B13.220581705094984%2C%2042.65207995894068%5D%2C%20%5B13.224092951095495%2C%2042.652585063940634%5D%2C%20%5B13.225882998095793%2C%2042.655755726940676%5D%2C%20%5B13.228446088096169%2C%2042.6556654789406%5D%2C%20%5B13.229050937096247%2C%2042.657578886940634%5D%2C%20%5B13.230568473096493%2C%2042.65784238094062%5D%2C%20%5B13.23067427909651%2C%2042.659288047940656%5D%2C%20%5B13.231924910096708%2C%2042.65949387094062%5D%2C%20%5B13.233857829096957%2C%2042.66147063794061%5D%2C%20%5B13.237690667097558%2C%2042.66312937794073%5D%2C%20%5B13.238762615097738%2C%2042.66318418394071%5D%2C%20%5B13.239169444097826%2C%2042.663929905940684%5D%2C%20%5B13.240440940097999%2C%2042.66386925494069%5D%2C%20%5B13.241541136098187%2C%2042.66633144894071%5D%2C%20%5B13.245315464098777%2C%2042.6662183909407%5D%2C%20%5B13.24717271709909%2C%2042.666986754940694%5D%2C%20%5B13.24802897009923%2C%2042.66691893294072%5D%2C%20%5B13.24851173109932%2C%2042.66580255894068%5D%2C%20%5B13.250488598099658%2C%2042.664491096940736%5D%2C%20%5B13.248178640099303%2C%2042.66160125494073%5D%2C%20%5B13.24888060209941%2C%2042.66092691494072%5D%2C%20%5B13.248703554099436%2C%2042.660276231940664%5D%2C%20%5B13.249273592099515%2C%2042.65953476194072%5D%2C%20%5B13.250159004099677%2C%2042.65928126794069%5D%2C%20%5B13.250981241099833%2C%2042.65954331894069%5D%2C%20%5B13.251399880099855%2C%2042.661152902940735%5D%2C%20%5B13.252623496100062%2C%2042.661017365940715%5D%2C%20%5B13.255395575100541%2C%2042.659631224940675%5D%2C%20%5B13.257381235100905%2C%2042.65757201994067%5D%2C%20%5B13.261174531101505%2C%2042.657610794940666%5D%2C%20%5B13.260546788101392%2C%2042.65871011494074%5D%2C%20%5B13.262948173101792%2C%2042.65835050594066%5D%2C%20%5B13.263054256101839%2C%2042.659332442940695%5D%2C%20%5B13.264823814102094%2C%2042.66105816694073%5D%2C%20%5B13.264778782102061%2C%2042.66356283994077%5D%2C%20%5B13.26410134110192%2C%2042.6655508839407%5D%2C%20%5B13.264347569101938%2C%2042.66632951394075%5D%2C%20%5B13.263610559101826%2C%2042.6673924089407%5D%2C%20%5B13.262962908101661%2C%2042.671810299940745%5D%2C%20%5B13.266722325102188%2C%2042.67749538094068%5D%2C%20%5B13.266543716102131%2C%2042.68084236694072%5D%2C%20%5B13.26906576110261%2C%2042.67969481194076%5D%2C%20%5B13.271915896103058%2C%2042.67943980194079%5D%2C%20%5B13.272587567103184%2C%2042.67989189294077%5D%2C%20%5B13.277429598104012%2C%2042.68042213294074%5D%2C%20%5B13.2810387111046%2C%2042.681898689940766%5D%2C%20%5B13.28460622610515%2C%2042.685294444940766%5D%2C%20%5B13.28786873110571%2C%2042.68693683194079%5D%2C%20%5B13.290213472106098%2C%2042.688663083940824%5D%2C%20%5B13.291993872106422%2C%2042.68919499794078%5D%2C%20%5B13.291921548106382%2C%2042.69004854594083%5D%2C%20%5B13.294293268106768%2C%2042.69125599694083%5D%2C%20%5B13.295952952107033%2C%2042.69263421194079%5D%2C%20%5B13.300387638107825%2C%2042.693831526940826%5D%2C%20%5B13.305114131108622%2C%2042.694216367940854%5D%2C%20%5B13.308531916109233%2C%2042.69512302294086%5D%2C%20%5B13.309915358109512%2C%2042.694071488940814%5D%2C%20%5B13.332440961113655%2C%2042.69217119994085%5D%2C%20%5B13.333852788113893%2C%2042.69142442194085%5D%2C%20%5B13.335570769114248%2C%2042.6897296529408%5D%2C%20%5B13.338979967114879%2C%2042.68867295894083%5D%2C%20%5B13.34175015611539%2C%2042.688243665940824%5D%2C%20%5B13.344443090115957%2C%2042.68658826594087%5D%2C%20%5B13.348375989116736%2C%2042.685106233940864%5D%2C%20%5B13.350888658117247%2C%2042.68240413194087%5D%2C%20%5B13.350394012117121%2C%2042.681252502940886%5D%2C%20%5B13.35085018511726%2C%2042.6797045519408%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%2210%22%2C%20%22properties%22%3A%20%7B%22COD_COM%22%3A%20%22002%22%2C%20%22COD_PROV%22%3A%20%22057%22%2C%20%22COD_REG%22%3A%20%2212%22%2C%20%22COMUNE%22%3A%20%22Amatrice%22%2C%20%22GlobalID%22%3A%20%2277a9b6a8-09eb-477e-835b-35f22838753b%22%2C%20%22PROVINCIA%22%3A%20%22RI%22%2C%20%22Sum_ABITNO%22%3A%201.0%2C%20%22Sum_ABIT_R%22%3A%201265.0%2C%20%22Sum_ABIT_V%22%3A%203117.0%2C%20%22Sum_ALBERG%22%3A%2024.0%2C%20%22Sum_ALTRIA%22%3A%207.0%2C%20%22Sum_EDIF_A%22%3A%203157.0%2C%20%22Sum_EDIF_T%22%3A%204140.0%2C%20%22Sum_EDIF_U%22%3A%203582.0%2C%20%22Sum_FAMRES%22%3A%201272.0%2C%20%22Sum_RES_0_%22%3A%20111.0%2C%20%22Sum_RES_10%22%3A%20111.0%2C%20%22Sum_RES_15%22%3A%2099.0%2C%20%22Sum_RES_20%22%3A%20115.0%2C%20%22Sum_RES_25%22%3A%20173.0%2C%20%22Sum_RES_30%22%3A%20186.0%2C%20%22Sum_RES_35%22%3A%20191.0%2C%20%22Sum_RES_40%22%3A%20177.0%2C%20%22Sum_RES_45%22%3A%20158.0%2C%20%22Sum_RES_50%22%3A%20175.0%2C%20%22Sum_RES_55%22%3A%20145.0%2C%20%22Sum_RES_5_%22%3A%20100.0%2C%20%22Sum_RES_60%22%3A%20196.0%2C%20%22Sum_RES_65%22%3A%20176.0%2C%20%22Sum_RES_70%22%3A%20214.0%2C%20%22Sum_RES_PI%22%3A%20480.0%2C%20%22Sum_RES_TO%22%3A%202807.0%2C%20%22Sum_STRAN_%22%3A%2051.0%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22bbox%22%3A%20%5B12.981763058064939%2C%2042.36487823093963%2C%2013.059182391074502%2C%2042.446172407939926%5D%2C%20%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B13.040796355071834%2C%2042.41892212493978%5D%2C%20%5B13.042137492072026%2C%2042.41563783593976%5D%2C%20%5B13.042039125072062%2C%2042.41371896093981%5D%2C%20%5B13.043261278072181%2C%2042.41310853993976%5D%2C%20%5B13.044251015072318%2C%2042.410975643939835%5D%2C%20%5B13.045253291072493%2C%2042.406100583939846%5D%2C%20%5B13.049780973073075%2C%2042.40469341193975%5D%2C%20%5B13.048994187072998%2C%2042.40380734493982%5D%2C%20%5B13.04566946207262%2C%2042.40210161193978%5D%2C%20%5B13.04455231407246%2C%2042.39933635393976%5D%2C%20%5B13.046621433072715%2C%2042.398831024939746%5D%2C%20%5B13.04941085907313%2C%2042.397426789939786%5D%2C%20%5B13.052295824073541%2C%2042.389401175939724%5D%2C%20%5B13.059182391074502%2C%2042.38471368393977%5D%2C%20%5B13.05525116007404%2C%2042.37669542193974%5D%2C%20%5B13.052368421073641%2C%2042.37604568693973%5D%2C%20%5B13.049276164073285%2C%2042.37312977993972%5D%2C%20%5B13.047403191073052%2C%2042.3732140319397%5D%2C%20%5B13.044003421072615%2C%2042.375144003939695%5D%2C%20%5B13.041805692072357%2C%2042.37513609293972%5D%2C%20%5B13.039399292072002%2C%2042.37596837393964%5D%2C%20%5B13.037984225071853%2C%2042.3737988149397%5D%2C%20%5B13.03684606407174%2C%2042.373708402939634%5D%2C%20%5B13.034300661071427%2C%2042.374302370939695%5D%2C%20%5B13.0326714720712%2C%2042.3731937609397%5D%2C%20%5B13.027148219070549%2C%2042.372748932939665%5D%2C%20%5B13.022868022070027%2C%2042.369820063939635%5D%2C%20%5B13.01072309706853%2C%2042.368675044939664%5D%2C%20%5B13.006634591068103%2C%2042.36545071193965%5D%2C%20%5B13.00060551506739%2C%2042.36487823093963%5D%2C%20%5B12.992897088066469%2C%2042.365930728939695%5D%2C%20%5B12.994247606066592%2C%2042.367756457939656%5D%2C%20%5B12.995765083066711%2C%2042.3727367649397%5D%2C%20%5B12.99719460506689%2C%2042.37552763393966%5D%2C%20%5B12.997109993066843%2C%2042.37780411193966%5D%2C%20%5B12.994955137066633%2C%2042.377982885939645%5D%2C%20%5B12.994228094066493%2C%2042.38057891493971%5D%2C%20%5B12.985763357065442%2C%2042.38966193793972%5D%2C%20%5B12.983064182065139%2C%2042.3919934039397%5D%2C%20%5B12.981763058064939%2C%2042.396959387939674%5D%2C%20%5B12.98212313906497%2C%2042.39747809993976%5D%2C%20%5B12.982755243065027%2C%2042.39746060893979%5D%2C%20%5B12.983874838065175%2C%2042.39818698993971%5D%2C%20%5B12.985192558065291%2C%2042.39952323793971%5D%2C%20%5B12.983521161065074%2C%2042.404277029939756%5D%2C%20%5B12.984814905065221%2C%2042.405244949939764%5D%2C%20%5B12.984993460065235%2C%2042.40612112893976%5D%2C%20%5B12.986333561065367%2C%2042.406394106939736%5D%2C%20%5B12.988119926065608%2C%2042.407524910939735%5D%2C%20%5B12.986693597065404%2C%2042.409951666939754%5D%2C%20%5B12.987398426065507%2C%2042.410719468939796%5D%2C%20%5B12.987679354065511%2C%2042.41219084493973%5D%2C%20%5B12.989953704065757%2C%2042.4140429459398%5D%2C%20%5B12.991450674065941%2C%2042.41435541293979%5D%2C%20%5B12.991968374065989%2C%2042.41487759893982%5D%2C%20%5B12.992368909065995%2C%2042.4167949909398%5D%2C%20%5B12.993609560066128%2C%2042.41753055493983%5D%2C%20%5B12.998631760066754%2C%2042.41745424493983%5D%2C%20%5B13.004272476067316%2C%2042.425450679939836%5D%2C%20%5B13.005387501067498%2C%2042.42593839593986%5D%2C%20%5B13.00579228306749%2C%2042.430034555939855%5D%2C%20%5B13.006370623067552%2C%2042.431360408939874%5D%2C%20%5B13.008467142067813%2C%2042.43232245693989%5D%2C%20%5B13.005469748067373%2C%2042.438293505939846%5D%2C%20%5B12.997754955066428%2C%2042.443704544939905%5D%2C%20%5B12.99536142006614%2C%2042.444179806939886%5D%2C%20%5B12.994444290066017%2C%2042.445098714939824%5D%2C%20%5B12.994143374066018%2C%2042.44614917993987%5D%2C%20%5B12.994638745066082%2C%2042.446172407939926%5D%2C%20%5B13.000898080066813%2C%2042.442878885939884%5D%2C%20%5B13.006499093067491%2C%2042.439184885939866%5D%2C%20%5B13.009226311067849%2C%2042.43828802093988%5D%2C%20%5B13.01246651306821%2C%2042.43656274793982%5D%2C%20%5B13.014195198068425%2C%2042.43417909693984%5D%2C%20%5B13.017719333068875%2C%2042.43346566893985%5D%2C%20%5B13.023132002069561%2C%2042.43142520193988%5D%2C%20%5B13.02792179607013%2C%2042.43065795593982%5D%2C%20%5B13.031800222070656%2C%2042.42906724393988%5D%2C%20%5B13.03488752607107%2C%2042.42675246393987%5D%2C%20%5B13.036722934071296%2C%2042.42363539193987%5D%2C%20%5B13.040787867071812%2C%2042.42112839793982%5D%2C%20%5B13.040796355071834%2C%2042.41892212493978%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%2211%22%2C%20%22properties%22%3A%20%7B%22COD_COM%22%3A%20%22015%22%2C%20%22COD_PROV%22%3A%20%22057%22%2C%20%22COD_REG%22%3A%20%2212%22%2C%20%22COMUNE%22%3A%20%22Castel%20Sant%5Cu0027Angelo%22%2C%20%22GlobalID%22%3A%20%22d5e5589d-decf-460b-b129-7b0f8e5d3d71%22%2C%20%22PROVINCIA%22%3A%20%22RI%22%2C%20%22Sum_ABITNO%22%3A%200.0%2C%20%22Sum_ABIT_R%22%3A%20561.0%2C%20%22Sum_ABIT_V%22%3A%20625.0%2C%20%22Sum_ALBERG%22%3A%203.0%2C%20%22Sum_ALTRIA%22%3A%200.0%2C%20%22Sum_EDIF_A%22%3A%20993.0%2C%20%22Sum_EDIF_T%22%3A%201009.0%2C%20%22Sum_EDIF_U%22%3A%201005.0%2C%20%22Sum_FAMRES%22%3A%20565.0%2C%20%22Sum_RES_0_%22%3A%2039.0%2C%20%22Sum_RES_10%22%3A%2063.0%2C%20%22Sum_RES_15%22%3A%2057.0%2C%20%22Sum_RES_20%22%3A%2074.0%2C%20%22Sum_RES_25%22%3A%2082.0%2C%20%22Sum_RES_30%22%3A%2077.0%2C%20%22Sum_RES_35%22%3A%2080.0%2C%20%22Sum_RES_40%22%3A%2078.0%2C%20%22Sum_RES_45%22%3A%2066.0%2C%20%22Sum_RES_50%22%3A%20102.0%2C%20%22Sum_RES_55%22%3A%2072.0%2C%20%22Sum_RES_5_%22%3A%2068.0%2C%20%22Sum_RES_60%22%3A%2083.0%2C%20%22Sum_RES_65%22%3A%2067.0%2C%20%22Sum_RES_70%22%3A%20106.0%2C%20%22Sum_RES_PI%22%3A%20168.0%2C%20%22Sum_RES_TO%22%3A%201282.0%2C%20%22Sum_STRAN_%22%3A%2022.0%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22bbox%22%3A%20%5B13.296997037108765%2C%2042.506805068940345%2C%2013.426068128134856%2C%2042.591255481940635%5D%2C%20%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B13.393678555127124%2C%2042.58103623794056%5D%2C%20%5B13.396604405127723%2C%2042.5805006219406%5D%2C%20%5B13.400120821128494%2C%2042.57928045194059%5D%2C%20%5B13.402938981129104%2C%2042.57724073294058%5D%2C%20%5B13.404516243129441%2C%2042.57370464494061%5D%2C%20%5B13.40416160912943%2C%2042.57231375694058%5D%2C%20%5B13.401789906128998%2C%2042.56882167794053%5D%2C%20%5B13.401104610128925%2C%2042.56273467194053%5D%2C%20%5B13.404441460129634%2C%2042.559878145940594%5D%2C%20%5B13.405866441129973%2C%2042.557513856940524%5D%2C%20%5B13.405179643129879%2C%2042.55482125494054%5D%2C%20%5B13.405657494130025%2C%2042.553285755940514%5D%2C%20%5B13.405335104129989%2C%2042.55111932594052%5D%2C%20%5B13.409311950131013%2C%2042.538883356940445%5D%2C%20%5B13.412961529131834%2C%2042.53518620694039%5D%2C%20%5B13.415117071132329%2C%2042.53132604094041%5D%2C%20%5B13.426068128134856%2C%2042.519004630940415%5D%2C%20%5B13.424540302134577%2C%2042.51779424894047%5D%2C%20%5B13.423511825134318%2C%2042.518041111940455%5D%2C%20%5B13.420498519133671%2C%2042.52071911494043%5D%2C%20%5B13.415548376132536%2C%2042.523885990940414%5D%2C%20%5B13.413475505132075%2C%2042.52593776994045%5D%2C%20%5B13.404850475130155%2C%2042.53187946794048%5D%2C%20%5B13.3979911591288%2C%2042.52870408794037%5D%2C%20%5B13.394006644128016%2C%2042.52597160094036%5D%2C%20%5B13.393742001128022%2C%2042.52301510094038%5D%2C%20%5B13.392952799127878%2C%2042.5205829459404%5D%2C%20%5B13.391698494127674%2C%2042.51817764794038%5D%2C%20%5B13.390113378127385%2C%2042.51637926794038%5D%2C%20%5B13.387852654126934%2C%2042.51578174694042%5D%2C%20%5B13.380541404125394%2C%2042.51701186594038%5D%2C%20%5B13.37358038812401%2C%2042.51645442394036%5D%2C%20%5B13.371352993123601%2C%2042.51550416094036%5D%2C%20%5B13.36910282112318%2C%2042.51355082594028%5D%2C%20%5B13.35958046612132%2C%2042.512293514940396%5D%2C%20%5B13.358076227121042%2C%2042.511630550940275%5D%2C%20%5B13.356330197120737%2C%2042.50953620194027%5D%2C%20%5B13.348246502119174%2C%2042.506805068940345%5D%2C%20%5B13.348127145119058%2C%2042.51596176194035%5D%2C%20%5B13.347950421118988%2C%2042.517129942940365%5D%2C%20%5B13.347054655118804%2C%2042.51785726694031%5D%2C%20%5B13.344851212118337%2C%2042.5183101339403%5D%2C%20%5B13.344812261118356%2C%2042.51933802294031%5D%2C%20%5B13.343245903118067%2C%2042.51917242094032%5D%2C%20%5B13.342674563117919%2C%2042.52053114794036%5D%2C%20%5B13.338294213117049%2C%2042.52089094794031%5D%2C%20%5B13.337373373116924%2C%2042.51893157194036%5D%2C%20%5B13.335599782116617%2C%2042.51833254594033%5D%2C%20%5B13.333442627116192%2C%2042.51932365494033%5D%2C%20%5B13.33021998811558%2C%2042.519198100940336%5D%2C%20%5B13.328157338115156%2C%2042.51985239894033%5D%2C%20%5B13.325536420114604%2C%2042.52457039494032%5D%2C%20%5B13.322416877114039%2C%2042.526007351940365%5D%2C%20%5B13.322495964114001%2C%2042.52715232394035%5D%2C%20%5B13.321259486113789%2C%2042.52924286394035%5D%2C%20%5B13.321921558113857%2C%2042.52998315594035%5D%2C%20%5B13.329033878115183%2C%2042.53219468494037%5D%2C%20%5B13.330000012115383%2C%2042.53202756194036%5D%2C%20%5B13.330619942115469%2C%2042.532530801940396%5D%2C%20%5B13.33008111211537%2C%2042.5331004219404%5D%2C%20%5B13.326010579114591%2C%2042.53460949094036%5D%2C%20%5B13.324557728114293%2C%2042.53475896894037%5D%2C%20%5B13.323722153114133%2C%2042.535483835940354%5D%2C%20%5B13.316101592112682%2C%2042.53811709294033%5D%2C%20%5B13.31455177511238%2C%2042.539156958940374%5D%2C%20%5B13.314297133112355%2C%2042.53999939794036%5D%2C%20%5B13.3214710181136%2C%2042.54450948894042%5D%2C%20%5B13.320536985113401%2C%2042.54528757294037%5D%2C%20%5B13.317049792112778%2C%2042.54680580394036%5D%2C%20%5B13.316986205112764%2C%2042.54736192494047%5D%2C%20%5B13.317942103112902%2C%2042.54802362094043%5D%2C%20%5B13.317737527112874%2C%2042.548400491940384%5D%2C%20%5B13.318419242112984%2C%2042.54841977194041%5D%2C%20%5B13.319189317113098%2C%2042.55011488394039%5D%2C%20%5B13.318054335112873%2C%2042.55112562594044%5D%2C%20%5B13.318444545112984%2C%2042.55155206994045%5D%2C%20%5B13.316353902112558%2C%2042.551387889940415%5D%2C%20%5B13.316006404112494%2C%2042.55163509794046%5D%2C%20%5B13.315814240112456%2C%2042.552434665940474%5D%2C%20%5B13.316372326112562%2C%2042.55250364194043%5D%2C%20%5B13.316233842112542%2C%2042.55314362194046%5D%2C%20%5B13.31425915611214%2C%2042.55320461994036%5D%2C%20%5B13.313784541112058%2C%2042.554996230940446%5D%2C%20%5B13.314096105112116%2C%2042.556091922940396%5D%2C%20%5B13.312994006111884%2C%2042.555849873940474%5D%2C%20%5B13.311358799111577%2C%2042.5568614059404%5D%2C%20%5B13.310023684111323%2C%2042.5562094449404%5D%2C%20%5B13.309276751111206%2C%2042.558182368940464%5D%2C%20%5B13.307559141110872%2C%2042.55909340994047%5D%2C%20%5B13.306177816110605%2C%2042.55926247294045%5D%2C%20%5B13.306100464110573%2C%2042.56030530094043%5D%2C%20%5B13.305100352110395%2C%2042.56079313094044%5D%2C%20%5B13.309929338111234%2C%2042.56379403294039%5D%2C%20%5B13.310346781111297%2C%2042.565034303940465%5D%2C%20%5B13.305619175110397%2C%2042.56729671194043%5D%2C%20%5B13.301429454109636%2C%2042.56814313794039%5D%2C%20%5B13.29936814510926%2C%2042.56912100594047%5D%2C%20%5B13.297381843108843%2C%2042.57094685794048%5D%2C%20%5B13.296997037108765%2C%2042.5720822659405%5D%2C%20%5B13.305020830110209%2C%2042.57230272794046%5D%2C%20%5B13.307439602110621%2C%2042.57385931394052%5D%2C%20%5B13.307805256110711%2C%2042.57250400594047%5D%2C%20%5B13.309779104111088%2C%2042.57385671094046%5D%2C%20%5B13.311372811111367%2C%2042.5734950229405%5D%2C%20%5B13.311944298111465%2C%2042.57400919094048%5D%2C%20%5B13.315215540112021%2C%2042.57512830494049%5D%2C%20%5B13.315657066112133%2C%2042.57639465594054%5D%2C%20%5B13.317497647112415%2C%2042.576365701940475%5D%2C%20%5B13.319976657112866%2C%2042.57706890494046%5D%2C%20%5B13.323097193113444%2C%2042.57974659894046%5D%2C%20%5B13.327639031114254%2C%2042.579876441940534%5D%2C%20%5B13.33529481711574%2C%2042.57836649894046%5D%2C%20%5B13.339519388116534%2C%2042.5753116299405%5D%2C%20%5B13.342832324117216%2C%2042.57489775994055%5D%2C%20%5B13.343668325117353%2C%2042.57507309194052%5D%2C%20%5B13.34706517211795%2C%2042.57747402394051%5D%2C%20%5B13.350903879118682%2C%2042.57745859694054%5D%2C%20%5B13.352079092118926%2C%2042.57679713694052%5D%2C%20%5B13.354462965119366%2C%2042.576490307940524%5D%2C%20%5B13.355992284119687%2C%2042.578592916940515%5D%2C%20%5B13.359240887120292%2C%2042.578271054940586%5D%2C%20%5B13.360779802120614%2C%2042.57859953594053%5D%2C%20%5B13.365703759121509%2C%2042.58087413594054%5D%2C%20%5B13.368545034122096%2C%2042.58082415694053%5D%2C%20%5B13.370443589122463%2C%2042.57996830894058%5D%2C%20%5B13.372028221122745%2C%2042.581361807940546%5D%2C%20%5B13.373082357122971%2C%2042.58063723494058%5D%2C%20%5B13.374668371123267%2C%2042.581049259940556%5D%2C%20%5B13.381785992124717%2C%2042.581010851940626%5D%2C%20%5B13.382566736124847%2C%2042.58240348894061%5D%2C%20%5B13.38432391612522%2C%2042.583245488940555%5D%2C%20%5B13.384814165125297%2C%2042.584046011940615%5D%2C%20%5B13.382934502124895%2C%2042.5851264619406%5D%2C%20%5B13.38356601512499%2C%2042.58568298294061%5D%2C%20%5B13.388993869126057%2C%2042.58739253594062%5D%2C%20%5B13.389478916126182%2C%2042.588431830940614%5D%2C%20%5B13.389404849126128%2C%2042.589902244940625%5D%2C%20%5B13.394022974127022%2C%2042.591255481940635%5D%2C%20%5B13.395243936127263%2C%2042.59080340294065%5D%2C%20%5B13.395934557127475%2C%2042.58952536994062%5D%2C%20%5B13.39282504412687%2C%2042.58625504494059%5D%2C%20%5B13.392605072126853%2C%2042.5825000309406%5D%2C%20%5B13.393678555127124%2C%2042.58103623794056%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%2212%22%2C%20%22properties%22%3A%20%7B%22COD_COM%22%3A%20%22016%22%2C%20%22COD_PROV%22%3A%20%22066%22%2C%20%22COD_REG%22%3A%20%2213%22%2C%20%22COMUNE%22%3A%20%22Campotosto%22%2C%20%22GlobalID%22%3A%20%227e9888bf-17f2-4069-866b-5c3547f80025%22%2C%20%22PROVINCIA%22%3A%20%22AQ%22%2C%20%22Sum_ABITNO%22%3A%2014.0%2C%20%22Sum_ABIT_R%22%3A%20376.0%2C%20%22Sum_ABIT_V%22%3A%20698.0%2C%20%22Sum_ALBERG%22%3A%2011.0%2C%20%22Sum_ALTRIA%22%3A%200.0%2C%20%22Sum_EDIF_A%22%3A%20884.0%2C%20%22Sum_EDIF_T%22%3A%201097.0%2C%20%22Sum_EDIF_U%22%3A%20916.0%2C%20%22Sum_FAMRES%22%3A%20383.0%2C%20%22Sum_RES_0_%22%3A%2019.0%2C%20%22Sum_RES_10%22%3A%2015.0%2C%20%22Sum_RES_15%22%3A%2021.0%2C%20%22Sum_RES_20%22%3A%2028.0%2C%20%22Sum_RES_25%22%3A%2033.0%2C%20%22Sum_RES_30%22%3A%2053.0%2C%20%22Sum_RES_35%22%3A%2037.0%2C%20%22Sum_RES_40%22%3A%2040.0%2C%20%22Sum_RES_45%22%3A%2042.0%2C%20%22Sum_RES_50%22%3A%2033.0%2C%20%22Sum_RES_55%22%3A%2029.0%2C%20%22Sum_RES_5_%22%3A%209.0%2C%20%22Sum_RES_60%22%3A%2052.0%2C%20%22Sum_RES_65%22%3A%2063.0%2C%20%22Sum_RES_70%22%3A%2070.0%2C%20%22Sum_RES_PI%22%3A%20139.0%2C%20%22Sum_RES_TO%22%3A%20683.0%2C%20%22Sum_STRAN_%22%3A%201.0%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22bbox%22%3A%20%5B13.253804409101933%2C%2042.48815919294023%2C%2013.348246502119174%2C%2042.56079313094044%5D%2C%20%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B13.309276751111206%2C%2042.558182368940464%5D%2C%20%5B13.310023684111323%2C%2042.5562094449404%5D%2C%20%5B13.311358799111577%2C%2042.5568614059404%5D%2C%20%5B13.312994006111884%2C%2042.555849873940474%5D%2C%20%5B13.314096105112116%2C%2042.556091922940396%5D%2C%20%5B13.313784541112058%2C%2042.554996230940446%5D%2C%20%5B13.31425915611214%2C%2042.55320461994036%5D%2C%20%5B13.316233842112542%2C%2042.55314362194046%5D%2C%20%5B13.316372326112562%2C%2042.55250364194043%5D%2C%20%5B13.315814240112456%2C%2042.552434665940474%5D%2C%20%5B13.316006404112494%2C%2042.55163509794046%5D%2C%20%5B13.316353902112558%2C%2042.551387889940415%5D%2C%20%5B13.318444545112984%2C%2042.55155206994045%5D%2C%20%5B13.318054335112873%2C%2042.55112562594044%5D%2C%20%5B13.319189317113098%2C%2042.55011488394039%5D%2C%20%5B13.318419242112984%2C%2042.54841977194041%5D%2C%20%5B13.317737527112874%2C%2042.548400491940384%5D%2C%20%5B13.317942103112902%2C%2042.54802362094043%5D%2C%20%5B13.316986205112764%2C%2042.54736192494047%5D%2C%20%5B13.317049792112778%2C%2042.54680580394036%5D%2C%20%5B13.320536985113401%2C%2042.54528757294037%5D%2C%20%5B13.3214710181136%2C%2042.54450948894042%5D%2C%20%5B13.314297133112355%2C%2042.53999939794036%5D%2C%20%5B13.31455177511238%2C%2042.539156958940374%5D%2C%20%5B13.316101592112682%2C%2042.53811709294033%5D%2C%20%5B13.323722153114133%2C%2042.535483835940354%5D%2C%20%5B13.324557728114293%2C%2042.53475896894037%5D%2C%20%5B13.326010579114591%2C%2042.53460949094036%5D%2C%20%5B13.33008111211537%2C%2042.5331004219404%5D%2C%20%5B13.330619942115469%2C%2042.532530801940396%5D%2C%20%5B13.330000012115383%2C%2042.53202756194036%5D%2C%20%5B13.329033878115183%2C%2042.53219468494037%5D%2C%20%5B13.321921558113857%2C%2042.52998315594035%5D%2C%20%5B13.321259486113789%2C%2042.52924286394035%5D%2C%20%5B13.322495964114001%2C%2042.52715232394035%5D%2C%20%5B13.322416877114039%2C%2042.526007351940365%5D%2C%20%5B13.325536420114604%2C%2042.52457039494032%5D%2C%20%5B13.328157338115156%2C%2042.51985239894033%5D%2C%20%5B13.33021998811558%2C%2042.519198100940336%5D%2C%20%5B13.333442627116192%2C%2042.51932365494033%5D%2C%20%5B13.335599782116617%2C%2042.51833254594033%5D%2C%20%5B13.337373373116924%2C%2042.51893157194036%5D%2C%20%5B13.338294213117049%2C%2042.52089094794031%5D%2C%20%5B13.342674563117919%2C%2042.52053114794036%5D%2C%20%5B13.343245903118067%2C%2042.51917242094032%5D%2C%20%5B13.344812261118356%2C%2042.51933802294031%5D%2C%20%5B13.344851212118337%2C%2042.5183101339403%5D%2C%20%5B13.347054655118804%2C%2042.51785726694031%5D%2C%20%5B13.347950421118988%2C%2042.517129942940365%5D%2C%20%5B13.348127145119058%2C%2042.51596176194035%5D%2C%20%5B13.348246502119174%2C%2042.506805068940345%5D%2C%20%5B13.345445630118713%2C%2042.503629693940326%5D%2C%20%5B13.341283929117893%2C%2042.501311723940276%5D%2C%20%5B13.340118831117689%2C%2042.500257495940325%5D%2C%20%5B13.340295529117757%2C%2042.49864814794023%5D%2C%20%5B13.34220869311819%2C%2042.49669378594026%5D%2C%20%5B13.344947032118691%2C%2042.49530228994023%5D%2C%20%5B13.344569030118635%2C%2042.49488447394032%5D%2C%20%5B13.341255482118015%2C%2042.494087404940316%5D%2C%20%5B13.336014133117036%2C%2042.4916706479402%5D%2C%20%5B13.32988935811591%2C%2042.48955719694022%5D%2C%20%5B13.33018948911599%2C%2042.489235210940194%5D%2C%20%5B13.327931495115592%2C%2042.48815919294023%5D%2C%20%5B13.326375370115242%2C%2042.48994675394022%5D%2C%20%5B13.323233360114635%2C%2042.492311947940195%5D%2C%20%5B13.312607755112637%2C%2042.49286158494026%5D%2C%20%5B13.307087782111596%2C%2042.4958201669402%5D%2C%20%5B13.307258744111639%2C%2042.496714093940206%5D%2C%20%5B13.304721742111145%2C%2042.49963223594022%5D%2C%20%5B13.301122617110474%2C%2042.50070849994022%5D%2C%20%5B13.284830371107432%2C%2042.50874794294027%5D%2C%20%5B13.281960214106908%2C%2042.50910751894029%5D%2C%20%5B13.280728642106642%2C%2042.5136104269403%5D%2C%20%5B13.278054440106175%2C%2042.51438124294028%5D%2C%20%5B13.27633586010587%2C%2042.51576904194024%5D%2C%20%5B13.27329270410535%2C%2042.51533804394025%5D%2C%20%5B13.270989821104923%2C%2042.51541054394029%5D%2C%20%5B13.26863478010452%2C%2042.51607918694028%5D%2C%20%5B13.265099511103928%2C%2042.51534670194026%5D%2C%20%5B13.259974098103005%2C%2042.51714484594029%5D%2C%20%5B13.258959468102834%2C%2042.518164027940294%5D%2C%20%5B13.253804409101933%2C%2042.520561745940306%5D%2C%20%5B13.256978952102447%2C%2042.521960770940304%5D%2C%20%5B13.260682258103099%2C%2042.52275477494032%5D%2C%20%5B13.261287189103195%2C%2042.52334899294031%5D%2C%20%5B13.268252039104325%2C%2042.525961490940226%5D%2C%20%5B13.267304731104165%2C%2042.52762199694029%5D%2C%20%5B13.267335138104155%2C%2042.52873731694025%5D%2C%20%5B13.265709328103858%2C%2042.52939670294031%5D%2C%20%5B13.270094624104594%2C%2042.53152451394032%5D%2C%20%5B13.271271388104772%2C%2042.531687661940275%5D%2C%20%5B13.27151470110482%2C%2042.53214226494035%5D%2C%20%5B13.270076663104552%2C%2042.53263263394027%5D%2C%20%5B13.26913302710439%2C%2042.53415796294035%5D%2C%20%5B13.270941651104673%2C%2042.53392387794035%5D%2C%20%5B13.272847260105015%2C%2042.53414082694034%5D%2C%20%5B13.273293064105125%2C%2042.53434026094037%5D%2C%20%5B13.273349479105077%2C%2042.53535556594041%5D%2C%20%5B13.274482817105303%2C%2042.53624059594029%5D%2C%20%5B13.274322719105177%2C%2042.54160375094028%5D%2C%20%5B13.272185505104751%2C%2042.544461222940335%5D%2C%20%5B13.274878200105217%2C%2042.54575618294035%5D%2C%20%5B13.275357892105289%2C%2042.54689522194038%5D%2C%20%5B13.277270837105606%2C%2042.548214765940344%5D%2C%20%5B13.280323581106105%2C%2042.55146342494039%5D%2C%20%5B13.282615413106523%2C%2042.55273268194045%5D%2C%20%5B13.283179183106588%2C%2042.553940566940376%5D%2C%20%5B13.28716203410726%2C%2042.55482676494036%5D%2C%20%5B13.297647801109076%2C%2042.55621595994038%5D%2C%20%5B13.301669677109803%2C%2042.55838319594037%5D%2C%20%5B13.300976857109676%2C%2042.56012894094047%5D%2C%20%5B13.304032850110195%2C%2042.56020305594042%5D%2C%20%5B13.305100352110395%2C%2042.56079313094044%5D%2C%20%5B13.306100464110573%2C%2042.56030530094043%5D%2C%20%5B13.306177816110605%2C%2042.55926247294045%5D%2C%20%5B13.307559141110872%2C%2042.55909340994047%5D%2C%20%5B13.309276751111206%2C%2042.558182368940464%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%2213%22%2C%20%22properties%22%3A%20%7B%22COD_COM%22%3A%20%22021%22%2C%20%22COD_PROV%22%3A%20%22066%22%2C%20%22COD_REG%22%3A%20%2213%22%2C%20%22COMUNE%22%3A%20%22Capitignano%22%2C%20%22GlobalID%22%3A%20%228bddff68-f3cd-4dd0-8270-67ce1435bef5%22%2C%20%22PROVINCIA%22%3A%20%22AQ%22%2C%20%22Sum_ABITNO%22%3A%208.0%2C%20%22Sum_ABIT_R%22%3A%20323.0%2C%20%22Sum_ABIT_V%22%3A%20483.0%2C%20%22Sum_ALBERG%22%3A%2011.0%2C%20%22Sum_ALTRIA%22%3A%200.0%2C%20%22Sum_EDIF_A%22%3A%20751.0%2C%20%22Sum_EDIF_T%22%3A%201002.0%2C%20%22Sum_EDIF_U%22%3A%20780.0%2C%20%22Sum_FAMRES%22%3A%20331.0%2C%20%22Sum_RES_0_%22%3A%2016.0%2C%20%22Sum_RES_10%22%3A%2030.0%2C%20%22Sum_RES_15%22%3A%2034.0%2C%20%22Sum_RES_20%22%3A%2025.0%2C%20%22Sum_RES_25%22%3A%2033.0%2C%20%22Sum_RES_30%22%3A%2033.0%2C%20%22Sum_RES_35%22%3A%2036.0%2C%20%22Sum_RES_40%22%3A%2053.0%2C%20%22Sum_RES_45%22%3A%2054.0%2C%20%22Sum_RES_50%22%3A%2031.0%2C%20%22Sum_RES_55%22%3A%2020.0%2C%20%22Sum_RES_5_%22%3A%2022.0%2C%20%22Sum_RES_60%22%3A%2036.0%2C%20%22Sum_RES_65%22%3A%2040.0%2C%20%22Sum_RES_70%22%3A%2066.0%2C%20%22Sum_RES_PI%22%3A%20160.0%2C%20%22Sum_RES_TO%22%3A%20689.0%2C%20%22Sum_STRAN_%22%3A%207.0%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22bbox%22%3A%20%5B13.155806836086308%2C%2042.46886097494004%2C%2013.312607755112637%2C%2042.5875694239404%5D%2C%20%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B13.20435274909315%2C%2042.5856005109404%5D%2C%20%5B13.207136572093626%2C%2042.58330124494035%5D%2C%20%5B13.20817822709379%2C%2042.583515027940344%5D%2C%20%5B13.208750886093872%2C%2042.584362811940416%5D%2C%20%5B13.211121736094217%2C%2042.584581669940405%5D%2C%20%5B13.211900007094357%2C%2042.58403980194048%5D%2C%20%5B13.212711042094526%2C%2042.58224521094045%5D%2C%20%5B13.217453756095225%2C%2042.58428983194041%5D%2C%20%5B13.218643785095402%2C%2042.58403885894045%5D%2C%20%5B13.219115981095536%2C%2042.58242779094046%5D%2C%20%5B13.21998572009567%2C%2042.58226064894046%5D%2C%20%5B13.226793991096756%2C%2042.58283307994043%5D%2C%20%5B13.228519722097015%2C%2042.58168885094038%5D%2C%20%5B13.230617743097389%2C%2042.57864455694048%5D%2C%20%5B13.234038464097974%2C%2042.57569532694043%5D%2C%20%5B13.235693742098302%2C%2042.574909242940386%5D%2C%20%5B13.245941953099987%2C%2042.572701511940444%5D%2C%20%5B13.245967552100007%2C%2042.573952069940496%5D%2C%20%5B13.246553378100058%2C%2042.57434899094041%5D%2C%20%5B13.24732811110021%2C%2042.57410863694041%5D%2C%20%5B13.248051520100306%2C%2042.57437889094039%5D%2C%20%5B13.248736537100436%2C%2042.57559594994043%5D%2C%20%5B13.250276287100682%2C%2042.57671821894043%5D%2C%20%5B13.249978081100597%2C%2042.57825541894047%5D%2C%20%5B13.253507430101184%2C%2042.57972236294041%5D%2C%20%5B13.253315560101141%2C%2042.581791332940526%5D%2C%20%5B13.253810716101222%2C%2042.58196199394044%5D%2C%20%5B13.254748766101393%2C%2042.58194960994048%5D%2C%20%5B13.25591316610156%2C%2042.58257705794044%5D%2C%20%5B13.261552318102467%2C%2042.58309636994051%5D%2C%20%5B13.268032274103593%2C%2042.58276919994044%5D%2C%20%5B13.27012079310394%2C%2042.58216452894047%5D%2C%20%5B13.275295091104862%2C%2042.58322272494046%5D%2C%20%5B13.279364707105543%2C%2042.583259597940426%5D%2C%20%5B13.279810425105621%2C%2042.58256314594046%5D%2C%20%5B13.280472308105761%2C%2042.5823988199405%5D%2C%20%5B13.2836254641063%2C%2042.58251935794047%5D%2C%20%5B13.284302749106425%2C%2042.58226889794051%5D%2C%20%5B13.286511515106884%2C%2042.57723864994042%5D%2C%20%5B13.289483615107416%2C%2042.574160526940474%5D%2C%20%5B13.291264264107792%2C%2042.57053280094049%5D%2C%20%5B13.293290043108174%2C%2042.5695339039404%5D%2C%20%5B13.297381843108843%2C%2042.57094685794048%5D%2C%20%5B13.29936814510926%2C%2042.56912100594047%5D%2C%20%5B13.301429454109636%2C%2042.56814313794039%5D%2C%20%5B13.305619175110397%2C%2042.56729671194043%5D%2C%20%5B13.310346781111297%2C%2042.565034303940465%5D%2C%20%5B13.309929338111234%2C%2042.56379403294039%5D%2C%20%5B13.305100352110395%2C%2042.56079313094044%5D%2C%20%5B13.304032850110195%2C%2042.56020305594042%5D%2C%20%5B13.300976857109676%2C%2042.56012894094047%5D%2C%20%5B13.301669677109803%2C%2042.55838319594037%5D%2C%20%5B13.297647801109076%2C%2042.55621595994038%5D%2C%20%5B13.28716203410726%2C%2042.55482676494036%5D%2C%20%5B13.283179183106588%2C%2042.553940566940376%5D%2C%20%5B13.282615413106523%2C%2042.55273268194045%5D%2C%20%5B13.280323581106105%2C%2042.55146342494039%5D%2C%20%5B13.277270837105606%2C%2042.548214765940344%5D%2C%20%5B13.275357892105289%2C%2042.54689522194038%5D%2C%20%5B13.274878200105217%2C%2042.54575618294035%5D%2C%20%5B13.272185505104751%2C%2042.544461222940335%5D%2C%20%5B13.274322719105177%2C%2042.54160375094028%5D%2C%20%5B13.274482817105303%2C%2042.53624059594029%5D%2C%20%5B13.273349479105077%2C%2042.53535556594041%5D%2C%20%5B13.273293064105125%2C%2042.53434026094037%5D%2C%20%5B13.272847260105015%2C%2042.53414082694034%5D%2C%20%5B13.270941651104673%2C%2042.53392387794035%5D%2C%20%5B13.26913302710439%2C%2042.53415796294035%5D%2C%20%5B13.270076663104552%2C%2042.53263263394027%5D%2C%20%5B13.27151470110482%2C%2042.53214226494035%5D%2C%20%5B13.271271388104772%2C%2042.531687661940275%5D%2C%20%5B13.270094624104594%2C%2042.53152451394032%5D%2C%20%5B13.265709328103858%2C%2042.52939670294031%5D%2C%20%5B13.267335138104155%2C%2042.52873731694025%5D%2C%20%5B13.267304731104165%2C%2042.52762199694029%5D%2C%20%5B13.268252039104325%2C%2042.525961490940226%5D%2C%20%5B13.261287189103195%2C%2042.52334899294031%5D%2C%20%5B13.260682258103099%2C%2042.52275477494032%5D%2C%20%5B13.256978952102447%2C%2042.521960770940304%5D%2C%20%5B13.253804409101933%2C%2042.520561745940306%5D%2C%20%5B13.258959468102834%2C%2042.518164027940294%5D%2C%20%5B13.259974098103005%2C%2042.51714484594029%5D%2C%20%5B13.265099511103928%2C%2042.51534670194026%5D%2C%20%5B13.26863478010452%2C%2042.51607918694028%5D%2C%20%5B13.270989821104923%2C%2042.51541054394029%5D%2C%20%5B13.27329270410535%2C%2042.51533804394025%5D%2C%20%5B13.27633586010587%2C%2042.51576904194024%5D%2C%20%5B13.278054440106175%2C%2042.51438124294028%5D%2C%20%5B13.280728642106642%2C%2042.5136104269403%5D%2C%20%5B13.281960214106908%2C%2042.50910751894029%5D%2C%20%5B13.284830371107432%2C%2042.50874794294027%5D%2C%20%5B13.301122617110474%2C%2042.50070849994022%5D%2C%20%5B13.304721742111145%2C%2042.49963223594022%5D%2C%20%5B13.307258744111639%2C%2042.496714093940206%5D%2C%20%5B13.307087782111596%2C%2042.4958201669402%5D%2C%20%5B13.312607755112637%2C%2042.49286158494026%5D%2C%20%5B13.305296530111365%2C%2042.48961655194021%5D%2C%20%5B13.303539162111042%2C%2042.488007996940254%5D%2C%20%5B13.301097454110607%2C%2042.48830240894022%5D%2C%20%5B13.300527960110518%2C%2042.487792602940196%5D%2C%20%5B13.301265967110664%2C%2042.4870355599402%5D%2C%20%5B13.298746906110233%2C%2042.48570316694023%5D%2C%20%5B13.29732123710995%2C%2042.48621141594021%5D%2C%20%5B13.29692127810989%2C%2042.485767252940164%5D%2C%20%5B13.29744099010996%2C%2042.48539658794024%5D%2C%20%5B13.295892268109704%2C%2042.483721549940135%5D%2C%20%5B13.288610081108377%2C%2042.487083037940174%5D%2C%20%5B13.286390103108012%2C%2042.48741834194019%5D%2C%20%5B13.269289063104997%2C%2042.4883774489402%5D%2C%20%5B13.256905268102868%2C%2042.48827169394014%5D%2C%20%5B13.256316353102724%2C%2042.48889235294019%5D%2C%20%5B13.253888991102308%2C%2042.489558874940194%5D%2C%20%5B13.25235863310207%2C%2042.491623595940226%5D%2C%20%5B13.245185927100984%2C%2042.48050928594015%5D%2C%20%5B13.244506076100844%2C%2042.480012294940174%5D%2C%20%5B13.242293780100479%2C%2042.48038246994017%5D%2C%20%5B13.2380738020998%2C%2042.480327286940074%5D%2C%20%5B13.237187892099643%2C%2042.479828880940104%5D%2C%20%5B13.232472870098812%2C%2042.48095330494012%5D%2C%20%5B13.226752155097904%2C%2042.478891355940114%5D%2C%20%5B13.22214726809719%2C%2042.47910190594013%5D%2C%20%5B13.219818446096834%2C%2042.47843151094011%5D%2C%20%5B13.21834739509659%2C%2042.479255580940105%5D%2C%20%5B13.208185918095%2C%2042.47552407694005%5D%2C%20%5B13.20320617009418%2C%2042.474392580940126%5D%2C%20%5B13.194095062092856%2C%2042.46886097494004%5D%2C%20%5B13.183630030091155%2C%2042.47616352694012%5D%2C%20%5B13.178428800090304%2C%2042.47933385294009%5D%2C%20%5B13.172171987089326%2C%2042.482781041940065%5D%2C%20%5B13.171723858089258%2C%2042.48315303294015%5D%2C%20%5B13.171410647089187%2C%2042.48511827594005%5D%2C%20%5B13.173922767089493%2C%2042.49285995194012%5D%2C%20%5B13.176340183089684%2C%2042.50598027494015%5D%2C%20%5B13.177406323089864%2C%2042.5078366439402%5D%2C%20%5B13.177777887089844%2C%2042.512631087940164%5D%2C%20%5B13.176462183089653%2C%2042.51351197694022%5D%2C%20%5B13.174875621089415%2C%2042.513650920940194%5D%2C%20%5B13.174763540089389%2C%2042.51458239894019%5D%2C%20%5B13.169258702088598%2C%2042.51382875194019%5D%2C%20%5B13.166665794088146%2C%2042.5138421849402%5D%2C%20%5B13.167096749088197%2C%2042.51741449194023%5D%2C%20%5B13.160546867087167%2C%2042.522010715940226%5D%2C%20%5B13.15981363008709%2C%2042.523351931940226%5D%2C%20%5B13.159944939087067%2C%2042.524675213940235%5D%2C%20%5B13.158102468086756%2C%2042.526097265940194%5D%2C%20%5B13.157360770086672%2C%2042.527713388940185%5D%2C%20%5B13.157764608086683%2C%2042.52843251994022%5D%2C%20%5B13.1590900050869%2C%2042.52767754494024%5D%2C%20%5B13.160663527087113%2C%2042.52965968094024%5D%2C%20%5B13.159865573086972%2C%2042.53111579794023%5D%2C%20%5B13.158517017086785%2C%2042.53229029894028%5D%2C%20%5B13.157004284086495%2C%2042.53609084894023%5D%2C%20%5B13.156616925086414%2C%2042.538896092940305%5D%2C%20%5B13.155806836086308%2C%2042.54004649394025%5D%2C%20%5B13.158081068086632%2C%2042.54047706594031%5D%2C%20%5B13.160293915086976%2C%2042.538919991940276%5D%2C%20%5B13.162159223087249%2C%2042.538843135940255%5D%2C%20%5B13.164447924087597%2C%2042.53761636194023%5D%2C%20%5B13.165076730087694%2C%2042.53776904194029%5D%2C%20%5B13.166346440087848%2C%2042.539329975940255%5D%2C%20%5B13.166682714087887%2C%2042.54114548994035%5D%2C%20%5B13.16779315408802%2C%2042.541901869940276%5D%2C%20%5B13.16885956008817%2C%2042.54457764294023%5D%2C%20%5B13.17097981108846%2C%2042.547088929940244%5D%2C%20%5B13.173523002088888%2C%2042.54544302394029%5D%2C%20%5B13.176817852089336%2C%2042.547317092940254%5D%2C%20%5B13.174390581088819%2C%2042.56372947394031%5D%2C%20%5B13.174280746088774%2C%2042.567510546940326%5D%2C%20%5B13.18098623608975%2C%2042.570696057940424%5D%2C%20%5B13.192277197091379%2C%2042.57796294194042%5D%2C%20%5B13.192355936091332%2C%2042.58403755494047%5D%2C%20%5B13.191332600091139%2C%2042.58619543194047%5D%2C%20%5B13.191061556091093%2C%2042.5875694239404%5D%2C%20%5B13.191367730091144%2C%2042.587513185940374%5D%2C%20%5B13.197180725092034%2C%2042.586665254940414%5D%2C%20%5B13.199333094092365%2C%2042.585978500940435%5D%2C%20%5B13.203039068092956%2C%2042.58595041694045%5D%2C%20%5B13.20435274909315%2C%2042.5856005109404%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%2214%22%2C%20%22properties%22%3A%20%7B%22COD_COM%22%3A%20%22056%22%2C%20%22COD_PROV%22%3A%20%22066%22%2C%20%22COD_REG%22%3A%20%2213%22%2C%20%22COMUNE%22%3A%20%22Montereale%22%2C%20%22GlobalID%22%3A%20%228de3d7d9-23e4-48ac-bf8b-5d308ad3758b%22%2C%20%22PROVINCIA%22%3A%20%22AQ%22%2C%20%22Sum_ABITNO%22%3A%2040.0%2C%20%22Sum_ABIT_R%22%3A%201240.0%2C%20%22Sum_ABIT_V%22%3A%201817.0%2C%20%22Sum_ALBERG%22%3A%2028.0%2C%20%22Sum_ALTRIA%22%3A%200.0%2C%20%22Sum_EDIF_A%22%3A%202540.0%2C%20%22Sum_EDIF_T%22%3A%203197.0%2C%20%22Sum_EDIF_U%22%3A%202737.0%2C%20%22Sum_FAMRES%22%3A%201248.0%2C%20%22Sum_RES_0_%22%3A%2093.0%2C%20%22Sum_RES_10%22%3A%20144.0%2C%20%22Sum_RES_15%22%3A%20147.0%2C%20%22Sum_RES_20%22%3A%20157.0%2C%20%22Sum_RES_25%22%3A%20161.0%2C%20%22Sum_RES_30%22%3A%20166.0%2C%20%22Sum_RES_35%22%3A%20187.0%2C%20%22Sum_RES_40%22%3A%20213.0%2C%20%22Sum_RES_45%22%3A%20150.0%2C%20%22Sum_RES_50%22%3A%20188.0%2C%20%22Sum_RES_55%22%3A%20147.0%2C%20%22Sum_RES_5_%22%3A%20118.0%2C%20%22Sum_RES_60%22%3A%20161.0%2C%20%22Sum_RES_65%22%3A%20206.0%2C%20%22Sum_RES_70%22%3A%20246.0%2C%20%22Sum_RES_PI%22%3A%20446.0%2C%20%22Sum_RES_TO%22%3A%202930.0%2C%20%22Sum_STRAN_%22%3A%2034.0%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22bbox%22%3A%20%5B13.376768257122663%2C%2042.63983157294084%2C%2013.57832502116725%2C%2042.7220213779412%5D%2C%20%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B13.555364742161238%2C%2042.7220213779412%5D%2C%20%5B13.557032407161698%2C%2042.720680937941225%5D%2C%20%5B13.558365798162017%2C%2042.721140952941155%5D%2C%20%5B13.56036563516253%2C%2042.71913898294121%5D%2C%20%5B13.56197976316298%2C%2042.7185478779412%5D%2C%20%5B13.564041680163486%2C%2042.718609611941154%5D%2C%20%5B13.56576281316393%2C%2042.71813572494119%5D%2C%20%5B13.567589804164399%2C%2042.7188325049412%5D%2C%20%5B13.570101223165016%2C%2042.71902921994125%5D%2C%20%5B13.57192751816549%2C%2042.71877161394127%5D%2C%20%5B13.575820004166498%2C%2042.71674767094124%5D%2C%20%5B13.577630022167016%2C%2042.715099624941224%5D%2C%20%5B13.57832502116725%2C%2042.71349171894125%5D%2C%20%5B13.577816898167098%2C%2042.712746796941246%5D%2C%20%5B13.575123414166475%2C%2042.71092344994117%5D%2C%20%5B13.572389584165839%2C%2042.70710743294112%5D%2C%20%5B13.570686368165592%2C%2042.69782567794122%5D%2C%20%5B13.568054173164969%2C%2042.69546401094117%5D%2C%20%5B13.566617581164644%2C%2042.69317606594114%5D%2C%20%5B13.566618728164672%2C%2042.69051105394115%5D%2C%20%5B13.56728428316486%2C%2042.69017384994111%5D%2C%20%5B13.567187697164865%2C%2042.6896375129411%5D%2C%20%5B13.567618997164987%2C%2042.68944471594113%5D%2C%20%5B13.568259750165115%2C%2042.68975673194117%5D%2C%20%5B13.568668786165224%2C%2042.68944777694116%5D%2C%20%5B13.562070365163622%2C%2042.68713188494106%5D%2C%20%5B13.561593654163543%2C%2042.68610202894109%5D%2C%20%5B13.563863279164151%2C%2042.6845078869411%5D%2C%20%5B13.564035895164196%2C%2042.68389777594108%5D%2C%20%5B13.56325850216401%2C%2042.682168673941106%5D%2C%20%5B13.563333394164037%2C%2042.68102676994114%5D%2C%20%5B13.56150320016362%2C%2042.67994740094109%5D%2C%20%5B13.558822214163015%2C%2042.6771327909411%5D%2C%20%5B13.556541903162415%2C%2042.676516953941096%5D%2C%20%5B13.557267404162687%2C%2042.67312530594103%5D%2C%20%5B13.557207213162707%2C%2042.67257400394113%5D%2C%20%5B13.555894259162367%2C%2042.67178453294103%5D%2C%20%5B13.556024427162422%2C%2042.67093753794104%5D%2C%20%5B13.557325714162776%2C%2042.66902648694106%5D%2C%20%5B13.558634121163113%2C%2042.66887978194109%5D%2C%20%5B13.559342290163316%2C%2042.66785667594102%5D%2C%20%5B13.562243784164089%2C%2042.66688109894101%5D%2C%20%5B13.562018843164049%2C%2042.66607978094105%5D%2C%20%5B13.558838232163302%2C%2042.662722397941025%5D%2C%20%5B13.552480782161807%2C%2042.657483747940994%5D%2C%20%5B13.551030942161486%2C%2042.654241766940956%5D%2C%20%5B13.548795114160994%2C%2042.65173780394097%5D%2C%20%5B13.54715012016059%2C%2042.651195524940974%5D%2C%20%5B13.546118660160314%2C%2042.65039023694099%5D%2C%20%5B13.543464249159694%2C%2042.64963594794094%5D%2C%20%5B13.541626237159194%2C%2042.65153665494096%5D%2C%20%5B13.54076553815894%2C%2042.65444290194098%5D%2C%20%5B13.53942864015855%2C%2042.655805977940986%5D%2C%20%5B13.538468224158304%2C%2042.65616372894098%5D%2C%20%5B13.537829372158207%2C%2042.65582895694096%5D%2C%20%5B13.537275073158051%2C%2042.654509464940986%5D%2C%20%5B13.534809583157491%2C%2042.654026579940954%5D%2C%20%5B13.529224170156105%2C%2042.654972772940965%5D%2C%20%5B13.527288571155594%2C%2042.65580122094101%5D%2C%20%5B13.527014989155528%2C%2042.65653682294102%5D%2C%20%5B13.527373995155601%2C%2042.65736441594097%5D%2C%20%5B13.52611258615529%2C%2042.65810311894101%5D%2C%20%5B13.526024028155238%2C%2042.65874135994097%5D%2C%20%5B13.522842710154478%2C%2042.659646055940975%5D%2C%20%5B13.521330090154088%2C%2042.659624860940944%5D%2C%20%5B13.51852092715338%2C%2042.66132051594094%5D%2C%20%5B13.516403282152863%2C%2042.662092952941016%5D%2C%20%5B13.512875206152001%2C%2042.66267794794094%5D%2C%20%5B13.509181505151137%2C%2042.664061660941016%5D%2C%20%5B13.506064297150408%2C%2042.66431062094096%5D%2C%20%5B13.501454977149288%2C%2042.66366390494096%5D%2C%20%5B13.499867003148985%2C%2042.66220035494098%5D%2C%20%5B13.498281483148663%2C%2042.66014696694092%5D%2C%20%5B13.496911528148322%2C%2042.65928252194095%5D%2C%20%5B13.496535285148264%2C%2042.6576722159409%5D%2C%20%5B13.495487032148082%2C%2042.65611984794094%5D%2C%20%5B13.492055962147287%2C%2042.65421095394096%5D%2C%20%5B13.490656668146991%2C%2042.65203759494089%5D%2C%20%5B13.489481242146764%2C%2042.651223938940916%5D%2C%20%5B13.488361212146561%2C%2042.64857591094084%5D%2C%20%5B13.487522476146342%2C%2042.64800110894094%5D%2C%20%5B13.486439349146174%2C%2042.64584229494088%5D%2C%20%5B13.484330153145704%2C%2042.645290307940826%5D%2C%20%5B13.482994892145383%2C%2042.64553624594091%5D%2C%20%5B13.481333108144964%2C%2042.646636783940906%5D%2C%20%5B13.480039241144697%2C%2042.64416204694085%5D%2C%20%5B13.480505912144876%2C%2042.64103759294084%5D%2C%20%5B13.481563094145146%2C%2042.64069904594091%5D%2C%20%5B13.481794218145186%2C%2042.64023531594084%5D%2C%20%5B13.480609745144898%2C%2042.63983157294084%5D%2C%20%5B13.478816801144484%2C%2042.64029347294084%5D%2C%20%5B13.478403935144419%2C%2042.6407102959409%5D%2C%20%5B13.47860736914446%2C%2042.64185475994085%5D%2C%20%5B13.477728628144247%2C%2042.64237536194091%5D%2C%20%5B13.476051485143858%2C%2042.64262560594079%5D%2C%20%5B13.475129291143643%2C%2042.64342249194088%5D%2C%20%5B13.470669786142642%2C%2042.642723633940825%5D%2C%20%5B13.467281563141873%2C%2042.643166691940834%5D%2C%20%5B13.459303878140103%2C%2042.642159133940844%5D%2C%20%5B13.453475859138775%2C%2042.643061554940836%5D%2C%20%5B13.450399876138091%2C%2042.64461287294079%5D%2C%20%5B13.448060183137528%2C%2042.64697728394085%5D%2C%20%5B13.446476868137172%2C%2042.64734948994084%5D%2C%20%5B13.434251620134559%2C%2042.64695096194087%5D%2C%20%5B13.417859011131132%2C%2042.64492504194079%5D%2C%20%5B13.414981796130533%2C%2042.64487411894081%5D%2C%20%5B13.408287438129168%2C%2042.64286355494084%5D%2C%20%5B13.405268642128549%2C%2042.64308344694077%5D%2C%20%5B13.403425474128165%2C%2042.64409530894077%5D%2C%20%5B13.402009756127823%2C%2042.64555886394081%5D%2C%20%5B13.39870659712714%2C%2042.64689245394083%5D%2C%20%5B13.39429940012628%2C%2042.64606253894083%5D%2C%20%5B13.390521799125493%2C%2042.64590154394079%5D%2C%20%5B13.388510881125073%2C%2042.647252736940736%5D%2C%20%5B13.383879347124129%2C%2042.64851983294077%5D%2C%20%5B13.381808592123718%2C%2042.64981467894078%5D%2C%20%5B13.378760692123098%2C%2042.650201550940785%5D%2C%20%5B13.376768257122663%2C%2042.64986817994075%5D%2C%20%5B13.379887463123241%2C%2042.65333661794073%5D%2C%20%5B13.385806498124333%2C%2042.661244347940794%5D%2C%20%5B13.387219449124583%2C%2042.661365705940746%5D%2C%20%5B13.390041863125177%2C%2042.66260788494088%5D%2C%20%5B13.39392884312596%2C%2042.66235952994087%5D%2C%20%5B13.394843205126138%2C%2042.66283758194085%5D%2C%20%5B13.400573189127273%2C%2042.6630448419408%5D%2C%20%5B13.405176496128204%2C%2042.66483931994084%5D%2C%20%5B13.407123371128595%2C%2042.665047896940806%5D%2C%20%5B13.409549942129068%2C%2042.66653893594087%5D%2C%20%5B13.41007560812915%2C%2042.66738298894086%5D%2C%20%5B13.412474637129629%2C%2042.668622930940856%5D%2C%20%5B13.414673105130097%2C%2042.66903324794085%5D%2C%20%5B13.415989863130331%2C%2042.670031292940834%5D%2C%20%5B13.41899529513096%2C%2042.67037884094086%5D%2C%20%5B13.420251065131213%2C%2042.67139719994091%5D%2C%20%5B13.422535385131678%2C%2042.67229473094088%5D%2C%20%5B13.42946675213311%2C%2042.67323284394088%5D%2C%20%5B13.431696292133633%2C%2042.6724261659409%5D%2C%20%5B13.434784096134244%2C%2042.6723964609409%5D%2C%20%5B13.438613838135078%2C%2042.67153206394091%5D%2C%20%5B13.44098982313562%2C%2042.6702153289409%5D%2C%20%5B13.44380539213621%2C%2042.67178955594097%5D%2C%20%5B13.444827088136433%2C%2042.67194791194088%5D%2C%20%5B13.44644948213677%2C%2042.67148417094089%5D%2C%20%5B13.447628874137065%2C%2042.671672381940866%5D%2C%20%5B13.44972409113751%2C%2042.67082556194093%5D%2C%20%5B13.456948942139066%2C%2042.67179109094092%5D%2C%20%5B13.456090722138843%2C%2042.67441752094097%5D%2C%20%5B13.452945875138145%2C%2042.67575101494098%5D%2C%20%5B13.452668391138085%2C%2042.67659463594096%5D%2C%20%5B13.453045516138138%2C%2042.678614711941%5D%2C%20%5B13.45269950413801%2C%2042.68024879794092%5D%2C%20%5B13.453799584138249%2C%2042.68165999094098%5D%2C%20%5B13.45398450513823%2C%2042.682602642940964%5D%2C%20%5B13.453618603138136%2C%2042.684043933940934%5D%2C%20%5B13.452123048137798%2C%2042.68638004294096%5D%2C%20%5B13.452405666137826%2C%2042.68668415394091%5D%2C%20%5B13.454923879138398%2C%2042.68753140194102%5D%2C%20%5B13.455983191138625%2C%2042.68762516594096%5D%2C%20%5B13.457117686138853%2C%2042.687171284940945%5D%2C%20%5B13.458585494139173%2C%2042.689027262940975%5D%2C%20%5B13.462916795140153%2C%2042.68663891894106%5D%2C%20%5B13.464236011140462%2C%2042.686591910941004%5D%2C%20%5B13.468618268141396%2C%2042.687541609941015%5D%2C%20%5B13.472371056142226%2C%2042.68734082994101%5D%2C%20%5B13.473709155142535%2C%2042.68655919994095%5D%2C%20%5B13.476819645143268%2C%2042.68712168294097%5D%2C%20%5B13.477766545143474%2C%2042.68681899694101%5D%2C%20%5B13.47898432514376%2C%2042.68544779694103%5D%2C%20%5B13.479595310143866%2C%2042.68591453894107%5D%2C%20%5B13.48121879214424%2C%2042.685909429941006%5D%2C%20%5B13.481953874144397%2C%2042.68732564894104%5D%2C%20%5B13.483846368144802%2C%2042.68880901294101%5D%2C%20%5B13.485542048145195%2C%2042.688841524940976%5D%2C%20%5B13.488621167145885%2C%2042.69022421894102%5D%2C%20%5B13.490860396146392%2C%2042.69050540794104%5D%2C%20%5B13.494298389147131%2C%2042.69220697094104%5D%2C%20%5B13.498109719148049%2C%2042.691093702941%5D%2C%20%5B13.500896745148708%2C%2042.69143866794101%5D%2C%20%5B13.501904617148892%2C%2042.69167358094104%5D%2C%20%5B13.504797293149565%2C%2042.69422460494102%5D%2C%20%5B13.50759364915017%2C%2042.69574847094105%5D%2C%20%5B13.509643476150652%2C%2042.69630685594106%5D%2C%20%5B13.511907479151173%2C%2042.69607346194108%5D%2C%20%5B13.517234341152433%2C%2042.69691196294105%5D%2C%20%5B13.51784832315257%2C%2042.69731985694104%5D%2C%20%5B13.518086909152606%2C%2042.70004743194109%5D%2C%20%5B13.5221445961535%2C%2042.70401048594118%5D%2C%20%5B13.523053310153704%2C%2042.70435717194113%5D%2C%20%5B13.525252450154188%2C%2042.70425662494105%5D%2C%20%5B13.527518660154733%2C%2042.70548586694112%5D%2C%20%5B13.533030332156033%2C%2042.70733817394108%5D%2C%20%5B13.5386929861574%2C%2042.707491598941154%5D%2C%20%5B13.544873565158884%2C%2042.70842544194117%5D%2C%20%5B13.546650221159274%2C%2042.71185255494119%5D%2C%20%5B13.546921214159307%2C%2042.71383599894114%5D%2C%20%5B13.545938091159039%2C%2042.71411369094114%5D%2C%20%5B13.543404474158377%2C%2042.71696042894115%5D%2C%20%5B13.545598057158887%2C%2042.71871888094115%5D%2C%20%5B13.547255491159293%2C%2042.71834234094117%5D%2C%20%5B13.54838719815957%2C%2042.718540398941215%5D%2C%20%5B13.555364742161238%2C%2042.7220213779412%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%2215%22%2C%20%22properties%22%3A%20%7B%22COD_COM%22%3A%20%22036%22%2C%20%22COD_PROV%22%3A%20%22067%22%2C%20%22COD_REG%22%3A%20%2213%22%2C%20%22COMUNE%22%3A%20%22Rocca%20Santa%20Maria%22%2C%20%22GlobalID%22%3A%20%226ae2256a-ab4c-444c-950e-56da42d0cee4%22%2C%20%22PROVINCIA%22%3A%20%22TE%22%2C%20%22Sum_ABITNO%22%3A%201.0%2C%20%22Sum_ABIT_R%22%3A%20276.0%2C%20%22Sum_ABIT_V%22%3A%20240.0%2C%20%22Sum_ALBERG%22%3A%205.0%2C%20%22Sum_ALTRIA%22%3A%200.0%2C%20%22Sum_EDIF_A%22%3A%20387.0%2C%20%22Sum_EDIF_T%22%3A%20487.0%2C%20%22Sum_EDIF_U%22%3A%20426.0%2C%20%22Sum_FAMRES%22%3A%20315.0%2C%20%22Sum_RES_0_%22%3A%2024.0%2C%20%22Sum_RES_10%22%3A%2033.0%2C%20%22Sum_RES_15%22%3A%2027.0%2C%20%22Sum_RES_20%22%3A%2030.0%2C%20%22Sum_RES_25%22%3A%2048.0%2C%20%22Sum_RES_30%22%3A%2047.0%2C%20%22Sum_RES_35%22%3A%2055.0%2C%20%22Sum_RES_40%22%3A%2038.0%2C%20%22Sum_RES_45%22%3A%2032.0%2C%20%22Sum_RES_50%22%3A%2039.0%2C%20%22Sum_RES_55%22%3A%2041.0%2C%20%22Sum_RES_5_%22%3A%2022.0%2C%20%22Sum_RES_60%22%3A%2048.0%2C%20%22Sum_RES_65%22%3A%2060.0%2C%20%22Sum_RES_70%22%3A%2059.0%2C%20%22Sum_RES_PI%22%3A%2095.0%2C%20%22Sum_RES_TO%22%3A%20698.0%2C%20%22Sum_STRAN_%22%3A%2016.0%7D%2C%20%22type%22%3A%20%22Feature%22%7D%2C%20%7B%22bbox%22%3A%20%5B13.346685234116583%2C%2042.64880711294079%2C%2013.611913497175467%2C%2042.81694909594149%5D%2C%20%22geometry%22%3A%20%7B%22coordinates%22%3A%20%5B%5B%5B13.543664365156499%2C%2042.81694909594149%5D%2C%20%5B13.545597143157016%2C%2042.81590438094147%5D%2C%20%5B13.546866318157393%2C%2042.8129008329415%5D%2C%20%5B13.550379995158293%2C%2042.81013653094148%5D%2C%20%5B13.550311979158298%2C%2042.809387471941456%5D%2C%20%5B13.553263142159071%2C%2042.805556092941465%5D%2C%20%5B13.555050866159533%2C%2042.80439545594145%5D%2C%20%5B13.556848190159988%2C%2042.80389614494147%5D%2C%20%5B13.558218835160316%2C%2042.80493086594154%5D%2C%20%5B13.559840699160741%2C%2042.80459156094149%5D%2C%20%5B13.56245086816139%2C%2042.80332145994148%5D%2C%20%5B13.56419515716187%2C%2042.80350389294145%5D%2C%20%5B13.564747763161986%2C%2042.8039409819415%5D%2C%20%5B13.567519657162654%2C%2042.80414082994148%5D%2C%20%5B13.568806939163013%2C%2042.80347264894149%5D%2C%20%5B13.569790415163277%2C%2042.801628181941446%5D%2C%20%5B13.571479780163727%2C%2042.80050723594143%5D%2C%20%5B13.572937635164113%2C%2042.800381379941484%5D%2C%20%5B13.574713797164549%2C%2042.80080996294147%5D%2C%20%5B13.573738290164352%2C%2042.79749082194147%5D%2C%20%5B13.575248083164842%2C%2042.79402267294149%5D%2C%20%5B13.576415467165129%2C%2042.79325567094146%5D%2C%20%5B13.580642274166332%2C%2042.788067035941474%5D%2C%20%5B13.582877440166914%2C%2042.78655491194147%5D%2C%20%5B13.584105897167285%2C%2042.785191172941445%5D%2C%20%5B13.585088969167604%2C%2042.781811547941494%5D%2C%20%5B13.588117502168442%2C%2042.77705785094137%5D%2C%20%5B13.5893766311688%2C%2042.77599893194139%5D%2C%20%5B13.590098162169049%2C%2042.77385870494147%5D%2C%20%5B13.596231733170766%2C%2042.769079067941405%5D%2C%20%5B13.59812625117132%2C%2042.76472633994142%5D%2C%20%5B13.600873192172086%2C%2042.76270710394145%5D%2C%20%5B13.60561061317351%2C%2042.75580890194146%5D%2C%20%5B13.608087744174192%2C%2042.75322866594143%5D%2C%20%5B13.607953081174236%2C%2042.74997494194135%5D%2C%20%5B13.61012599517495%2C%2042.745142626941366%5D%2C%20%5B13.611215899175239%2C%2042.74394624694134%5D%2C%20%5B13.611400443175304%2C%2042.74223719794133%5D%2C%20%5B13.611913497175467%2C%2042.74170331294144%5D%2C%20%5B13.610268225175243%2C%2042.73331119594133%5D%2C%20%5B13.610325071175323%2C%2042.73022080694125%5D%2C%20%5B13.60738328517455%2C%2042.72960117794134%5D%2C%20%5B13.606366922174281%2C%2042.72860677794131%5D%2C%20%5B13.605596665174117%2C%2042.72646354694132%5D%2C%20%5B13.605639473174172%2C%2042.72400394694129%5D%2C%20%5B13.603182654173644%2C%2042.719191674941236%5D%2C%20%5B13.599297267172593%2C%2042.7187133699413%5D%2C%20%5B13.597196903172069%2C%2042.71759594394129%5D%2C%20%5B13.595152448171559%2C%2042.71729552294128%5D%2C%20%5B13.593048538171004%2C%2042.716376232941236%5D%2C%20%5B13.59069864017049%2C%2042.71279732694119%5D%2C%20%5B13.586569622169362%2C%2042.71395345194125%5D%2C%20%5B13.58439056216881%2C%2042.713667240941255%5D%2C%20%5B13.580602004167828%2C%2042.71379212794124%5D%2C%20%5B13.579184051167486%2C%2042.71331325394123%5D%2C%20%5B13.57832502116725%2C%2042.71349171894125%5D%2C%20%5B13.577630022167016%2C%2042.715099624941224%5D%2C%20%5B13.575820004166498%2C%2042.71674767094124%5D%2C%20%5B13.57192751816549%2C%2042.71877161394127%5D%2C%20%5B13.570101223165016%2C%2042.71902921994125%5D%2C%20%5B13.567589804164399%2C%2042.7188325049412%5D%2C%20%5B13.56576281316393%2C%2042.71813572494119%5D%2C%20%5B13.564041680163486%2C%2042.718609611941154%5D%2C%20%5B13.56197976316298%2C%2042.7185478779412%5D%2C%20%5B13.56036563516253%2C%2042.71913898294121%5D%2C%20%5B13.558365798162017%2C%2042.721140952941155%5D%2C%20%5B13.557032407161698%2C%2042.720680937941225%5D%2C%20%5B13.555364742161238%2C%2042.7220213779412%5D%2C%20%5B13.54838719815957%2C%2042.718540398941215%5D%2C%20%5B13.547255491159293%2C%2042.71834234094117%5D%2C%20%5B13.545598057158887%2C%2042.71871888094115%5D%2C%20%5B13.543404474158377%2C%2042.71696042894115%5D%2C%20%5B13.545938091159039%2C%2042.71411369094114%5D%2C%20%5B13.546921214159307%2C%2042.71383599894114%5D%2C%20%5B13.546650221159274%2C%2042.71185255494119%5D%2C%20%5B13.544873565158884%2C%2042.70842544194117%5D%2C%20%5B13.5386929861574%2C%2042.707491598941154%5D%2C%20%5B13.533030332156033%2C%2042.70733817394108%5D%2C%20%5B13.527518660154733%2C%2042.70548586694112%5D%2C%20%5B13.525252450154188%2C%2042.70425662494105%5D%2C%20%5B13.523053310153704%2C%2042.70435717194113%5D%2C%20%5B13.5221445961535%2C%2042.70401048594118%5D%2C%20%5B13.518086909152606%2C%2042.70004743194109%5D%2C%20%5B13.51784832315257%2C%2042.69731985694104%5D%2C%20%5B13.517234341152433%2C%2042.69691196294105%5D%2C%20%5B13.511907479151173%2C%2042.69607346194108%5D%2C%20%5B13.509643476150652%2C%2042.69630685594106%5D%2C%20%5B13.50759364915017%2C%2042.69574847094105%5D%2C%20%5B13.504797293149565%2C%2042.69422460494102%5D%2C%20%5B13.501904617148892%2C%2042.69167358094104%5D%2C%20%5B13.500896745148708%2C%2042.69143866794101%5D%2C%20%5B13.498109719148049%2C%2042.691093702941%5D%2C%20%5B13.494298389147131%2C%2042.69220697094104%5D%2C%20%5B13.490860396146392%2C%2042.69050540794104%5D%2C%20%5B13.488621167145885%2C%2042.69022421894102%5D%2C%20%5B13.485542048145195%2C%2042.688841524940976%5D%2C%20%5B13.483846368144802%2C%2042.68880901294101%5D%2C%20%5B13.481953874144397%2C%2042.68732564894104%5D%2C%20%5B13.48121879214424%2C%2042.685909429941006%5D%2C%20%5B13.479595310143866%2C%2042.68591453894107%5D%2C%20%5B13.47898432514376%2C%2042.68544779694103%5D%2C%20%5B13.477766545143474%2C%2042.68681899694101%5D%2C%20%5B13.476819645143268%2C%2042.68712168294097%5D%2C%20%5B13.473709155142535%2C%2042.68655919994095%5D%2C%20%5B13.472371056142226%2C%2042.68734082994101%5D%2C%20%5B13.468618268141396%2C%2042.687541609941015%5D%2C%20%5B13.464236011140462%2C%2042.686591910941004%5D%2C%20%5B13.462916795140153%2C%2042.68663891894106%5D%2C%20%5B13.458585494139173%2C%2042.689027262940975%5D%2C%20%5B13.457117686138853%2C%2042.687171284940945%5D%2C%20%5B13.455983191138625%2C%2042.68762516594096%5D%2C%20%5B13.454923879138398%2C%2042.68753140194102%5D%2C%20%5B13.452405666137826%2C%2042.68668415394091%5D%2C%20%5B13.452123048137798%2C%2042.68638004294096%5D%2C%20%5B13.453618603138136%2C%2042.684043933940934%5D%2C%20%5B13.45398450513823%2C%2042.682602642940964%5D%2C%20%5B13.453799584138249%2C%2042.68165999094098%5D%2C%20%5B13.45269950413801%2C%2042.68024879794092%5D%2C%20%5B13.453045516138138%2C%2042.678614711941%5D%2C%20%5B13.452668391138085%2C%2042.67659463594096%5D%2C%20%5B13.452945875138145%2C%2042.67575101494098%5D%2C%20%5B13.456090722138843%2C%2042.67441752094097%5D%2C%20%5B13.456948942139066%2C%2042.67179109094092%5D%2C%20%5B13.44972409113751%2C%2042.67082556194093%5D%2C%20%5B13.447628874137065%2C%2042.671672381940866%5D%2C%20%5B13.44644948213677%2C%2042.67148417094089%5D%2C%20%5B13.444827088136433%2C%2042.67194791194088%5D%2C%20%5B13.44380539213621%2C%2042.67178955594097%5D%2C%20%5B13.44098982313562%2C%2042.6702153289409%5D%2C%20%5B13.438613838135078%2C%2042.67153206394091%5D%2C%20%5B13.434784096134244%2C%2042.6723964609409%5D%2C%20%5B13.431696292133633%2C%2042.6724261659409%5D%2C%20%5B13.42946675213311%2C%2042.67323284394088%5D%2C%20%5B13.422535385131678%2C%2042.67229473094088%5D%2C%20%5B13.420251065131213%2C%2042.67139719994091%5D%2C%20%5B13.41899529513096%2C%2042.67037884094086%5D%2C%20%5B13.415989863130331%2C%2042.670031292940834%5D%2C%20%5B13.414673105130097%2C%2042.66903324794085%5D%2C%20%5B13.412474637129629%2C%2042.668622930940856%5D%2C%20%5B13.41007560812915%2C%2042.66738298894086%5D%2C%20%5B13.409549942129068%2C%2042.66653893594087%5D%2C%20%5B13.407123371128595%2C%2042.665047896940806%5D%2C%20%5B13.405176496128204%2C%2042.66483931994084%5D%2C%20%5B13.400573189127273%2C%2042.6630448419408%5D%2C%20%5B13.394843205126138%2C%2042.66283758194085%5D%2C%20%5B13.39392884312596%2C%2042.66235952994087%5D%2C%20%5B13.390041863125177%2C%2042.66260788494088%5D%2C%20%5B13.387219449124583%2C%2042.661365705940746%5D%2C%20%5B13.385806498124333%2C%2042.661244347940794%5D%2C%20%5B13.379887463123241%2C%2042.65333661794073%5D%2C%20%5B13.376768257122663%2C%2042.64986817994075%5D%2C%20%5B13.373467860122048%2C%2042.64968835194076%5D%2C%20%5B13.3716733381217%2C%2042.64880711294079%5D%2C%20%5B13.369923789121342%2C%2042.648968529940745%5D%2C%20%5B13.367880881120918%2C%2042.65342678494072%5D%2C%20%5B13.364332954120194%2C%2042.655646574940796%5D%2C%20%5B13.361210389119552%2C%2042.65703071394078%5D%2C%20%5B13.360442387119427%2C%2042.658388019940865%5D%2C%20%5B13.360363032119352%2C%2042.66002067694079%5D%2C%20%5B13.35719587611867%2C%2042.664008415940735%5D%2C%20%5B13.355064663118249%2C%2042.6656967369408%5D%2C%20%5B13.35189351011764%2C%2042.66702845394077%5D%2C%20%5B13.350339301117325%2C%2042.66856417394078%5D%2C%20%5B13.347784053116813%2C%2042.66876040994079%5D%2C%20%5B13.346685234116583%2C%2042.670638901940805%5D%2C%20%5B13.346764242116596%2C%2042.67133366894082%5D%2C%20%5B13.347725647116752%2C%2042.672805197940846%5D%2C%20%5B13.348601110116915%2C%2042.67338413994085%5D%2C%20%5B13.350252475117204%2C%2042.673694962940814%5D%2C%20%5B13.350451878117239%2C%2042.675402539940855%5D%2C%20%5B13.349783451117082%2C%2042.677579804940876%5D%2C%20%5B13.35085018511726%2C%2042.6797045519408%5D%2C%20%5B13.350394012117121%2C%2042.681252502940886%5D%2C%20%5B13.350888658117247%2C%2042.68240413194087%5D%2C%20%5B13.352057934117429%2C%2042.683899222940845%5D%2C%20%5B13.35126739611726%2C%2042.68561746494085%5D%2C%20%5B13.35142444411729%2C%2042.686619876940895%5D%2C%20%5B13.357309543118276%2C%2042.69256310694093%5D%2C%20%5B13.35776997611835%2C%2042.69409415394087%5D%2C%20%5B13.358884051118563%2C%2042.69582986594089%5D%2C%20%5B13.362922794119331%2C%2042.6935915069409%5D%2C%20%5B13.365109586119798%2C%2042.6930443529409%5D%2C%20%5B13.369194619120561%2C%2042.69323043694094%5D%2C%20%5B13.37327038212133%2C%2042.69517240094091%5D%2C%20%5B13.374588923121653%2C%2042.69376693594084%5D%2C%20%5B13.379371346122609%2C%2042.6920037299409%5D%2C%20%5B13.380844280122915%2C%2042.690713824940914%5D%2C%20%5B13.385831214123916%2C%2042.68905505694096%5D%2C%20%5B13.388818140124545%2C%2042.68730180294087%5D%2C%20%5B13.389994048124775%2C%2042.687157616940844%5D%2C%20%5B13.39261265512528%2C%2042.68738565794092%5D%2C%20%5B13.394068951125579%2C%2042.68819852494091%5D%2C%20%5B13.397771402126297%2C%2042.68899701694091%5D%2C%20%5B13.400253613126797%2C%2042.69072469694093%5D%2C%20%5B13.401834747127067%2C%2042.693806022940876%5D%2C%20%5B13.405875205127884%2C%2042.69573016494095%5D%2C%20%5B13.412987309129306%2C%2042.69736456394098%5D%2C%20%5B13.420206434130725%2C%2042.70252821294095%5D%2C%20%5B13.42740921413217%2C%2042.706427049941055%5D%2C%20%5B13.428746722132464%2C%2042.70615915894102%5D%2C%20%5B13.43242580813327%2C%2042.70310846994108%5D%2C%20%5B13.432930446133392%2C%2042.70148629894099%5D%2C%20%5B13.43383857213362%2C%2042.70036617194095%5D%2C%20%5B13.437011624134339%2C%2042.69892406094102%5D%2C%20%5B13.437578248134443%2C%2042.69919917794106%5D%2C%20%5B13.438636175134608%2C%2042.70132341894103%5D%2C%20%5B13.440903099135111%2C%2042.70245983494105%5D%2C%20%5B13.441178029135136%2C%2042.70366010894098%5D%2C%20%5B13.440838742135016%2C%2042.70452861094103%5D%2C%20%5B13.438262757134478%2C%2042.70597916094107%5D%2C%20%5B13.437650623134337%2C%2042.70700230094097%5D%2C%20%5B13.438774117134491%2C%2042.709497632941066%5D%2C%20%5B13.43842562013444%2C%2042.71116778594097%5D%2C%20%5B13.43739132313415%2C%2042.71247741694103%5D%2C%20%5B13.437789922134266%2C%2042.713614374941095%5D%2C%20%5B13.437324517134146%2C%2042.715635692941056%5D%2C%20%5B13.438021675134229%2C%2042.71754435394099%5D%2C%20%5B13.439626000134579%2C%2042.71881905594105%5D%2C%20%5B13.439232562134464%2C%2042.71989223194104%5D%2C%20%5B13.440866198134705%2C%2042.72392530894104%5D%2C%20%5B13.441205636134757%2C%2042.72695525794111%5D%2C%20%5B13.442775841135083%2C%2042.727641517941045%5D%2C%20%5B13.445159568135558%2C%2042.72952057894112%5D%2C%20%5B13.445693288135669%2C%2042.73064324794104%5D%2C%20%5B13.447438212136028%2C%2042.731273128941126%5D%2C%20%5B13.447999475136141%2C%2042.732214647941085%5D%2C%20%5B13.450333346136595%2C%2042.73266850694107%5D%2C%20%5B13.452121045137%2C%2042.73390437494114%5D%2C%20%5B13.45900954313847%2C%2042.73359984394106%5D%2C%20%5B13.459839004138667%2C%2042.73408517494119%5D%2C%20%5B13.463266270139416%2C%2042.73480671494111%5D%2C%20%5B13.464280597139652%2C%2042.734794112941145%5D%2C%20%5B13.465344774139895%2C%2042.73374417094111%5D%2C%20%5B13.468943901140678%2C%2042.73292825894113%5D%2C%20%5B13.471628708141308%2C%2042.73304833994116%5D%2C%20%5B13.474820608142021%2C%2042.73238772494111%5D%2C%20%5B13.47914880714298%2C%2042.73267736494117%5D%2C%20%5B13.480127326143213%2C%2042.73163964594114%5D%2C%20%5B13.481253943143484%2C%2042.731554966941125%5D%2C%20%5B13.48436762014415%2C%2042.73308045694117%5D%2C%20%5B13.486022324144555%2C%2042.73286696994114%5D%2C%20%5B13.487754096144894%2C%2042.73397392094111%5D%2C%20%5B13.488812323145135%2C%2042.73367126994114%5D%2C%20%5B13.48951255114532%2C%2042.73401290394116%5D%2C%20%5B13.491390255145701%2C%2042.73565876694121%5D%2C%20%5B13.493456187146146%2C%2042.73661294694114%5D%2C%20%5B13.4938984801462%2C%2042.73852676694114%5D%2C%20%5B13.496795465146853%2C%2042.73947521694119%5D%2C%20%5B13.498169253147172%2C%2042.74073564094124%5D%2C%20%5B13.500055633147513%2C%2042.745118017941216%5D%2C%20%5B13.501079347147767%2C%2042.74621212294123%5D%2C%20%5B13.504235322148459%2C%2042.74708265494122%5D%2C%20%5B13.504638634148543%2C%2042.747611458941186%5D%2C%20%5B13.504283089148418%2C%2042.75018240594129%5D%2C%20%5B13.502069584147907%2C%2042.75128699094124%5D%2C%20%5B13.50186011514781%2C%2042.75186244894119%5D%2C%20%5B13.501976043147845%2C%2042.75261866094122%5D%2C%20%5B13.503811841148243%2C%2042.754031875941266%5D%2C%20%5B13.503645392148172%2C%2042.75588410894127%5D%2C%20%5B13.504018388148252%2C%2042.75662568594125%5D%2C%20%5B13.502956036147944%2C%2042.75861225594125%5D%2C%20%5B13.5025982041478%2C%2042.761300329941214%5D%2C%20%5B13.50181691614761%2C%2042.76279414094126%5D%2C%20%5B13.50289762114782%2C%2042.764142573941236%5D%2C%20%5B13.502290145147672%2C%2042.76648485094122%5D%2C%20%5B13.502788305147764%2C%2042.76810832394121%5D%2C%20%5B13.502328260147596%2C%2042.77096698294129%5D%2C%20%5B13.503429902147806%2C%2042.7724226239413%5D%2C%20%5B13.504217395147965%2C%2042.77473694294126%5D%2C%20%5B13.505236511148194%2C%2042.775570089941304%5D%2C%20%5B13.506809059148567%2C%2042.77631387994133%5D%2C%20%5B13.509288428149032%2C%2042.77994793294137%5D%2C%20%5B13.508685339148863%2C%2042.78212800994135%5D%2C%20%5B13.50946077614905%2C%2042.78294822294138%5D%2C%20%5B13.5119574611496%2C%2042.78385354294141%5D%2C%20%5B13.515473794150374%2C%2042.78608701794134%5D%2C%20%5B13.517145512150737%2C%2042.78828076894133%5D%2C%20%5B13.516832687150654%2C%2042.78960760894132%5D%2C%20%5B13.517904840150885%2C%2042.790785167941394%5D%2C%20%5B13.522008096151826%2C%2042.79254056694142%5D%2C%20%5B13.524961699152465%2C%2042.79429632794137%5D%2C%20%5B13.528020634153206%2C%2042.79475137094139%5D%2C%20%5B13.529751488153549%2C%2042.795988244941405%5D%2C%20%5B13.531697103153935%2C%2042.80110147094143%5D%2C%20%5B13.532201953153983%2C%2042.806645445941484%5D%2C%20%5B13.533556222154271%2C%2042.80781616494149%5D%2C%20%5B13.532434082153925%2C%2042.81132241894151%5D%2C%20%5B13.533047401154066%2C%2042.81330130794151%5D%2C%20%5B13.534222573154295%2C%2042.81482124394142%5D%2C%20%5B13.534497313154336%2C%2042.81664249694146%5D%2C%20%5B13.539945781155645%2C%2042.81694834494155%5D%2C%20%5B13.541458971155988%2C%2042.81659110894149%5D%2C%20%5B13.543664365156499%2C%2042.81694909594149%5D%5D%5D%2C%20%22type%22%3A%20%22Polygon%22%7D%2C%20%22id%22%3A%20%2216%22%2C%20%22properties%22%3A%20%7B%22COD_COM%22%3A%20%22046%22%2C%20%22COD_PROV%22%3A%20%22067%22%2C%20%22COD_REG%22%3A%20%2213%22%2C%20%22COMUNE%22%3A%20%22Valle%20Castellana%22%2C%20%22GlobalID%22%3A%20%229a1f8a0b-4391-4cc0-b26a-1c8abb911d99%22%2C%20%22PROVINCIA%22%3A%20%22TE%22%2C%20%22Sum_ABITNO%22%3A%200.0%2C%20%22Sum_ABIT_R%22%3A%20526.0%2C%20%22Sum_ABIT_V%22%3A%20735.0%2C%20%22Sum_ALBERG%22%3A%2011.0%2C%20%22Sum_ALTRIA%22%3A%202.0%2C%20%22Sum_EDIF_A%22%3A%20974.0%2C%20%22Sum_EDIF_T%22%3A%201130.0%2C%20%22Sum_EDIF_U%22%3A%201020.0%2C%20%22Sum_FAMRES%22%3A%20575.0%2C%20%22Sum_RES_0_%22%3A%2048.0%2C%20%22Sum_RES_10%22%3A%2055.0%2C%20%22Sum_RES_15%22%3A%2058.0%2C%20%22Sum_RES_20%22%3A%2072.0%2C%20%22Sum_RES_25%22%3A%2066.0%2C%20%22Sum_RES_30%22%3A%2082.0%2C%20%22Sum_RES_35%22%3A%2084.0%2C%20%22Sum_RES_40%22%3A%2091.0%2C%20%22Sum_RES_45%22%3A%2088.0%2C%20%22Sum_RES_50%22%3A%2070.0%2C%20%22Sum_RES_55%22%3A%2053.0%2C%20%22Sum_RES_5_%22%3A%2040.0%2C%20%22Sum_RES_60%22%3A%2079.0%2C%20%22Sum_RES_65%22%3A%2096.0%2C%20%22Sum_RES_70%22%3A%20101.0%2C%20%22Sum_RES_PI%22%3A%20195.0%2C%20%22Sum_RES_TO%22%3A%201278.0%2C%20%22Sum_STRAN_%22%3A%207.0%7D%2C%20%22type%22%3A%20%22Feature%22%7D%5D%2C%20%22type%22%3A%20%22FeatureCollection%22%7D%29%3B%0A%0A%20%20%20%20%20%20%20%20%0A%20%20%20%20%0A%20%20%20%20geo_json_d6aba2b5fea54f0d9d8ab1008131a42b.bindTooltip%28%0A%20%20%20%20function%28layer%29%7B%0A%20%20%20%20let%20div%20%3D%20L.DomUtil.create%28%27div%27%29%3B%0A%20%20%20%20%0A%20%20%20%20let%20handleObject%20%3D%20feature%3D%3Etypeof%28feature%29%3D%3D%27object%27%20%3F%20JSON.stringify%28feature%29%20%3A%20feature%3B%0A%20%20%20%20let%20fields%20%3D%20%5B%22COD_REG%22%2C%20%22COD_PROV%22%2C%20%22COD_COM%22%2C%20%22COMUNE%22%2C%20%22PROVINCIA%22%2C%20%22Sum_RES_TO%22%2C%20%22Sum_RES_0_%22%2C%20%22Sum_RES_5_%22%2C%20%22Sum_RES_10%22%2C%20%22Sum_RES_15%22%2C%20%22Sum_RES_20%22%2C%20%22Sum_RES_25%22%2C%20%22Sum_RES_30%22%2C%20%22Sum_RES_35%22%2C%20%22Sum_RES_40%22%2C%20%22Sum_RES_45%22%2C%20%22Sum_RES_50%22%2C%20%22Sum_RES_55%22%2C%20%22Sum_RES_60%22%2C%20%22Sum_RES_65%22%2C%20%22Sum_RES_70%22%2C%20%22Sum_RES_PI%22%2C%20%22Sum_ABIT_R%22%2C%20%22Sum_ABITNO%22%2C%20%22Sum_ABIT_V%22%2C%20%22Sum_ALTRIA%22%2C%20%22Sum_EDIF_T%22%2C%20%22Sum_EDIF_U%22%2C%20%22Sum_EDIF_A%22%2C%20%22Sum_ALBERG%22%2C%20%22Sum_FAMRES%22%2C%20%22Sum_STRAN_%22%2C%20%22GlobalID%22%5D%3B%0A%20%20%20%20let%20aliases%20%3D%20%5B%22COD_REG%22%2C%20%22COD_PROV%22%2C%20%22COD_COM%22%2C%20%22COMUNE%22%2C%20%22PROVINCIA%22%2C%20%22Sum_RES_TO%22%2C%20%22Sum_RES_0_%22%2C%20%22Sum_RES_5_%22%2C%20%22Sum_RES_10%22%2C%20%22Sum_RES_15%22%2C%20%22Sum_RES_20%22%2C%20%22Sum_RES_25%22%2C%20%22Sum_RES_30%22%2C%20%22Sum_RES_35%22%2C%20%22Sum_RES_40%22%2C%20%22Sum_RES_45%22%2C%20%22Sum_RES_50%22%2C%20%22Sum_RES_55%22%2C%20%22Sum_RES_60%22%2C%20%22Sum_RES_65%22%2C%20%22Sum_RES_70%22%2C%20%22Sum_RES_PI%22%2C%20%22Sum_ABIT_R%22%2C%20%22Sum_ABITNO%22%2C%20%22Sum_ABIT_V%22%2C%20%22Sum_ALTRIA%22%2C%20%22Sum_EDIF_T%22%2C%20%22Sum_EDIF_U%22%2C%20%22Sum_EDIF_A%22%2C%20%22Sum_ALBERG%22%2C%20%22Sum_FAMRES%22%2C%20%22Sum_STRAN_%22%2C%20%22GlobalID%22%5D%3B%0A%20%20%20%20let%20table%20%3D%20%27%3Ctable%3E%27%20%2B%0A%20%20%20%20%20%20%20%20String%28%0A%20%20%20%20%20%20%20%20fields.map%28%0A%20%20%20%20%20%20%20%20%28v%2Ci%29%3D%3E%0A%20%20%20%20%20%20%20%20%60%3Ctr%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Cth%3E%24%7Baliases%5Bi%5D%7D%3C/th%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Ctd%3E%24%7BhandleObject%28layer.feature.properties%5Bv%5D%29%7D%3C/td%3E%0A%20%20%20%20%20%20%20%20%3C/tr%3E%60%29.join%28%27%27%29%29%0A%20%20%20%20%2B%27%3C/table%3E%27%3B%0A%20%20%20%20div.innerHTML%3Dtable%3B%0A%20%20%20%20%0A%20%20%20%20return%20div%0A%20%20%20%20%7D%0A%20%20%20%20%2C%7B%22className%22%3A%20%22foliumtooltip%22%2C%20%22sticky%22%3A%20true%7D%29%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%0A%3C/script%3E onload="this.contentDocument.open();this.contentDocument.write(    decodeURIComponent(this.getAttribute('data-html')));this.contentDocument.close();" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>




```python
minarea = geo_municipalities_affected_earthquake.geometry.area.min()
```


```python
minarea
```




    30665483.55191904




```python
smallest_municipality = geo_municipalities_affected_earthquake[geo_municipalities_affected_earthquake.geometry.area == minarea]
```


```python
smallest_municipality.plot()
plt.show()
```


    
![png](03_solution_exercise_retrieving_data_from_sdi_files/03_solution_exercise_retrieving_data_from_sdi_129_0.png)
    


```python
smallest_municipality.COMUNE
```




    13    Capitignano
    Name: COMUNE, dtype: object



we can use the same WFS resource used before with the new bounding box


```python
bbox= list(smallest_municipality.to_crs(epsg=4326).bounds.values[0])
```


```python
response = wfs.getfeature(typename=layer, bbox=bbox,srsname='urn:ogc:def:crs:EPSG::4326')
```


```python
rivers_capitignano = gpd.read_file(response)
```


```python
rivers_capitignano.plot()
plt.show()
```


    
![png](03_solution_exercise_retrieving_data_from_sdi_files/03_solution_exercise_retrieving_data_from_sdi_136_0.png)
    


... and we need always to invert the axes


```python
rivers_capitignano['geometry'] = rivers_capitignano['geometry'].apply(lambda geometry: swapxy(geometry))
```


```python
rivers_capitignano.plot()
plt.show()
```


    
![png](03_solution_exercise_retrieving_data_from_sdi_files/03_solution_exercise_retrieving_data_from_sdi_139_0.png)
