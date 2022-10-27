---
title: "Creating and manipulating geodata with R"
excerpt: ""
collection: portfolio
htmlwidgets: TRUE
---

## Starting steps
First load relevant packages:

```r
library(ggplot2)
library(plotly)
library(leaflet)
```

Then create a basic plot with the `leaflet` package that finds Copenhagen:

```r
m <- leaflet() %>%
  addTiles() %>%  # Add default OpenStreetMap map tiles
  addMarkers(lng=12.56, lat=55.67, popup="Copenhagen")
m  # Print the map
```

<center><iframe src="/map.html" height = "400px" width = "100%" frameBorder="0"></iframe></center>






