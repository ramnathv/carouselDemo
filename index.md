---
title       : TimeLine Maps Revisited
subtitle    : 
author      : Ramnath Vaidyanathan
job         : McGill University
date        : November 2012
framework   : io2012        # {io2012, html5slides, shower, dzslides, ...}
highlighter : highlight.js  # {highlight.js, prettify, highlight}
hitheme     : solarized_light      # 
widgets     : [bootstrap]   # {mathjax, quiz, bootstrap}
mode        : selfcontained # {standalone, draft}
github:
  user: ramnathv
  repo: carouselDemo
--- 

## Introduction

Recently, Christopher Gandrud posted an excellelent [blog article](http://christophergandrud.github.com/amc-site/maps.html) on how to display timeline maps using `googleVis` and `Twitter Bootstrap`. This slide deck is inspired by the article. 

The main objective of this slide deck is to illustrate

> - How to insert Timeline Maps directly in Slidify.
> - How to use mustache templates to work with `bootstrap`.


---

## Load googleVis and Dataset

We will first load `googleVis` and an example dataset. This code is directly borrowed from [Christopher Gandrud](https://github.com/christophergandrud/amcData/blob/master/SourceCode/Descriptive/AMCTotalMaps.R)


```r
library(googleVis)

# Load most recent data
URL <- "https://raw.github.com/christophergandrud/amcData/master/MainData/amcCountryYear.csv"
Full <- RCurl::getURL(URL)
Full <- read.csv(textConnection(Full))

# Clean for mapping
Full <- plyr::rename(Full, c(NumAMCCountryNoNA = "TotalAmcCreated"))
```


---

## Generate GeoMaps


```r
MapAMC <- function(y){
  yearTemp <- y  
  TempMap <-  gvisGeoMap(subset(Full, year == yearTemp &
    TotalAmcCreated != 0), 
    locationvar = "ISOCode", 
    numvar = "TotalAmcCreated",
    options = list(
      colors = "[0xECE7F2, 0xA6BDDB, 0x2B8CBE]",
      width = "780px",
      height = "500px"
    ))
  print(TempMap, file = paste("assets/img/AMCMap", yearTemp, ".html", sep = ""), 
   tag = "chart")
}

Years <- c(1980, 2011)
lapply(Years, MapAMC)
```


---

## DRY Approach to Carousels

A clean, DRY and reusable approach to using `bootstrap` with markdown is to separate the markup and the content. One way to achieve this is to 

1. Use `mustache` templates to encapsulate the markup.
2. Generate `data` to populate the template.
3. Use `whisker` to render the view.

---

## Step 1. Create Layout

First, we create a `mustache` template to encapsulate the markup.


```
<div id="{{id}}" class="carousel slide">
  <!-- Carousel items -->
  <div class="carousel-inner">
  {{# items }}
    <div class="item {{active}}">
       <iframe src="{{src}}" height = 560 width = 800></iframe>
       <div class="carousel-caption">
         <center><h1 style="color:white">{{{caption}}}<h1></center>
       </div>
    </div>
  {{/ items }}
  </div>
  <!-- Carousel nav -->
  <a class="carousel-control left" href="#{{id}}" data-slide="prev">&lsaquo;</a>
  <a class="carousel-control right" href="#{{id}}" data-slide="next">&rsaquo;</a>
</div>
```


---

## Step 2. Generate Data

Second, we generate data to populate the template.


```r
carousel = list(
  id    = 'geomaps',
  items = slidify:::zip_vectors(
    src     = sprintf('assets/img/AMCMap%s.html', Years),
    active  = c('active', rep("", length(Years) - 1)),
    caption = Years
  )
)
```


---

## Step 3. Render Carousel

Finally, we render the carousel using the `whisker` package.


```r
whisker::whisker.render(layout, carousel)
```


See next two slides for the markup generated, and the final display

---


```
<div id="geomaps" class="carousel slide">
  <!-- Carousel items -->
  <div class="carousel-inner">
    <div class="item active">
       <iframe src="assets/img/AMCMap1980.html" height = 560 width = 800></iframe>
       <div class="carousel-caption">
         <center><h1 style="color:white">1980<h1></center>
       </div>
    </div>
    <div class="item ">
       <iframe src="assets/img/AMCMap2011.html" height = 560 width = 800></iframe>
       <div class="carousel-caption">
         <center><h1 style="color:white">2011<h1></center>
       </div>
    </div>
  </div>
  <!-- Carousel nav -->
  <a class="carousel-control left" href="#geomaps" data-slide="prev">&lsaquo;</a>
  <a class="carousel-control right" href="#geomaps" data-slide="next">&rsaquo;</a>
</div>
```


---

## Display Carousel

<div id="geomaps" class="carousel slide">
  <!-- Carousel items -->
  <div class="carousel-inner">
    <div class="item active">
       <iframe src="assets/img/AMCMap1980.html" height = 560 width = 800></iframe>
       <div class="carousel-caption">
         <center><h1 style="color:white">1980<h1></center>
       </div>
    </div>
    <div class="item ">
       <iframe src="assets/img/AMCMap2011.html" height = 560 width = 800></iframe>
       <div class="carousel-caption">
         <center><h1 style="color:white">2011<h1></center>
       </div>
    </div>
  </div>
  <!-- Carousel nav -->
  <a class="carousel-control left" href="#geomaps" data-slide="prev">&lsaquo;</a>
  <a class="carousel-control right" href="#geomaps" data-slide="next">&rsaquo;</a>
</div>


---




---

## Note

Slidify allows use of `bootstrap` directly by including it as a `widget` in the YAML front matter. The widgets themselves are loaded from [slidifyLibraries](http://github.com/ramnathv/slidifyLibraries) which is a support package consisting of external libraries used by Slidify.

Currently, the `boostrap` widget does not ship with custom layouts. But my plan is to add commonly used layouts so that users of Slidify can directly use it without having to do any more work. 

If you are familiar with `bootstrap` and `mustache`, you can contribute to this by forking `slidifyLibraries`, adding layouts to `libraries/widgets/bootstrap/layouts` and sending me a pull request.




