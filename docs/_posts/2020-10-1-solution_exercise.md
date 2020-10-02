---
layout: post
title: purpose of the exercise is to consolidate the concepts introduced by GIS through the use of geopandas.
---

# GIS through geopandas
## Exercise 
* load the shapefile of ISTAT with the information of the provinces
* filter it for an italian provice at your choice (eg. Trento)
* plot it
* identify the cities of the province selected with the biggest and smallest area
* extract all the centroids of the areas expressed in WGS84
* extract a rappresenative point for the area of a municipality in WGS84<br/>suggestion: .representative_point()
* save the points in a GeoJSON file
* calculate the distance on the geodentic between the municipaly with the big area and smallest area by using the centroid

## learning objectives
* repeat the concepts on the previous lesson
* introduce geopackage
* centroid vs representative point

## Solution code / lesson
[01_solution_exercise _geopackage_and_representative_point.ipynb](https://raw.githubusercontent.com/napo/geospatial_course_unitn/master/code/01_solution_exercise%2C_geopackage_and_representative_point.ipynb)
