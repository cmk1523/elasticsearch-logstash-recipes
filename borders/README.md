# Builds in an index of countries

Intro
=========

Builds in an index of countries which allows you find the country of a given lat/lon

Requirements
=========

* Tested on Ubuntu Server 17.04

A) Downloaded the "world-country-boundaries" shapefile zip file from: https://koordinates.com/layer/1103-world-country-boundaries/ 9 (see attached file: kx-world-country-boundaries-SHP.zip)

B) Used https://ogre.adc4gis.com/ to convert the zip shapefile to GeoJSON (see attached file: shapfileToGeojson.json)

C) Formatted the shapfileToGeojson.json file in to a "single JSON document per line" format (see attached file: countries.input.json)

1) Created a template mapping for a index called "borders" with the following command: "curl -XPUT 'ubuntu:9200/_template/borders_1' -H "Content-Type: application/json;" --data @borders.mapping.json" (see attached file: borders.mapping.json). 

Warning: I used a 1000m precision. When I tried using a precision of anything else (i.e. 1m, 50m), they all my pegged my 1-node 2.5GHz 12-core 12GB VM  and brought it to it's knees for over 5 minutes. The ingesting of these geoshapes is very CPU intensive on the cluster. This crazy country called Asia really takes a long time.... haha.

2) Made a very simple Logstash recipe (see attached file: countries.logstash.conf) that creates elastic documents out of each JSON document in countries.input.json.

3) Ran the logstash recipe: "./logstash -f countries.logstash.conf"

4) Searched for "40.712784, -74.005941" (New York City's city center) with: "http://ubuntu:9200/borders/_search?pretty&source={ "query": { "geo_shape": { "geometry": { "shape": { "type": "circle", "radius": "1km", "coordinates": [ -74.005941, 40.712784 ] } } } } }"
