---
title: Visualising ESS data using ggplot2 and tmap
author: Paulius Alaburda
date: '2020-10-25'
slug: []
categories:
  - r
tags:
  - tmap
  - ggplot2
  - R
subtitle: ''
summary: ''
authors: []
lastmod: '2020-10-25T18:30:23+02:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---




I stumbled across rOpenSci's package [essurvey](https://github.com/ropensci/essurvey) that contains data from the European Social Survey (ESS) for European countries, including Lithuania! Also, tmap has a lot of excellent resources to get a head start, so this post shows how to access ESS data and visualise it with tmap. 

## Downloading ESS data

Downloading ESS data is fairly straightforward - register your email in the ESS and start pulling data! I recommend installing the development version because the stable version was downloading .zip files that Windows couldn't extract. To replicate this post, you should use:


```r
import_country("Lithuania", 9)
```

I didn't want to disclose my email here, so I have downloaded the data separately.




## So what is the ESS anyway?




## Visualising with tmap

All plotting packages depend on shapefiles. For Europe, it's best to download the map from [GISCO](https://ec.europa.eu/eurostat/web/gisco/geodata/reference-data/administrative-units-statistical-units/countries). However, the map does not have municipality data for Lithuania. Fortunately, the shapefile is available over the [OpenMap project](https://shapes.openmap.lt/).  

This is how to build a basic tmap plot:


```r
lt_map <- read_sf("../../../static/data/admin_ribos.shp")

lt_map_sen <- lt_map %>% filter(ADMIN_LEVE == 4)

news <- ess %>% 
  group_by(region = labelled::to_factor(region)) %>% 
  summarise(`Internet usage` = mean(netustm, na.rm = TRUE)) %>% 
  # count(region = labelled::to_factor(region), netusoft) %>%
  mutate(region = as.character(region)) %>% 
  mutate(region = case_when(region == "Klaipedos apskritis" ~ "Klaipėdos apskritis",
                            region == "Šiauliu apskritis" ~ "Šiaulių apskritis",
                            region == "Taurages apskritis" ~ "Tauragės apskritis",
                            region == "Telšiu apskritis" ~ "Telšių apskritis",
                            TRUE ~ region))

lt_plot <- lt_map_sen %>% left_join(news, by = c("NAME" = "region"))

tm_shape(lt_plot) +
  tm_fill("Internet usage") +
  tm_polygons()
```

```
## Warning: One tm layer group has duplicated layer types, which are omitted. To
## draw multiple layers of the same type, use multiple layer groups (i.e. specify
## tm_shape prior to each of them).
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-4-1.png" width="672" />






