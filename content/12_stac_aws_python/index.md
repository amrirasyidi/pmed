+++
title = "Exploring Cloud-Optimized GeoTIFF with STAC in Python"
date = 2024-10-20
authors = ["Amri Rasyidi"]
description = "Spatial queries, quick visualization, and open source data."
draft = false

[taxonomies]
series = []
tags = ["data-science", "how-to", "python", "geospatial"]
+++

# The End Result
This is what we are going to make today.
<p><center><img src="12_0_final_result.png"></center></p>

You can access my code [here](https://gitlab.com/comrades_0/data_reading/-/blob/master/notebook/2_sentinel_from_copernicus.ipynb?ref_type=heads)


# Some Intro First
Have you ever dealt with many images at once which somehow need to be queried based on some criteria. Let's say you have images of animals. One way to manage it is to make folders, using the animal name as the folder name. Images with cats in it will go to "cat" folder and so on. Easy right? But now some images contain multiple animals, should you put the image in multiple folder or do you make another folder with multiple animal name? To add some more complexity, what if you want to query something goofy, the "state" of the animal, is it walking? sleeping? standing up? drinking?

Have you? No? Okay. At least entertain your thought that you are important enough to deal with this kind of problem now.

Another way to approach this problem is by using [vector database](https://www.pinecone.io/learn/vector-database/). But at this point, our "many images" is not that many it doesn't really qualify as big data, vector database might be overly complex and an overkill.

You have this query problem in geospatial context you say? Enter [STAC](https://stacspec.org/en/tutorials/intro-to-stac/).

## STAC
![xkcd - standards](https://imgs.xkcd.com/comics/standards.png)

When looking for a satellite imagery, you might want to filter it by some kind of properties. You are given a task to analyse a site before and after a landslide to see the impacted neighborhood. It doesn't make sense to analyse the imagery 1 year after the landslide, the site most probably already fully recovered, you might be interested in the imageries a day to a week after the landslide. You might also want to consider the cloud coverage to make sure the imageries can be analysed.

This is exactly what STAC is useful for.

STAC is basically a standard on how to organize satellite imagery, or as the name state spario-temporal assets. Since the [documentation](https://stacspec.org/en/tutorials/intro-to-stac/) did a great job explaining it, I won't explain about STAC here.

## COG

# Da Code
To simplify the process, these are the steps that we are going to do:
1. Connect to the STAC client, 
1. Pick a suitable STAC collection 
1. From the collection, filter to get our item of interest
1. From the STAC item, get the asset. This is the satellite imagery, but still in the url form.
1. Open the url in python using rasterio.

Simple enough right? Let's get to it!

## Stack
Libraries will be needed:
1. duckdb
1. pystac_client
1. geopandas
1. rasterio

## Connecting with a client
There are options of STAC client, I guess among the most popular one are:
1. AWS COG by Element84 - `https://earth-search.aws.element84.com/v1`
1. Microsoft Planetary - `https://planetarycomputer.microsoft.com/api/stac/v1`
There also _might_ be from [source.coop](https://source.coop/), but I'm not sure do they have their own STAC client or they will refer you to other links.

Connecting is easy, just import the library and open the url

```python
from pystac_client import Client

api_url = "https://earth-search.aws.element84.com/v1"
client = Client.open(api_url)
```

Then you can check, what are the available collections in that specific client
```python
for collection in client.get_collections():
    print(collection)

<CollectionClient id=sentinel-2-pre-c1-l2a>
<CollectionClient id=cop-dem-glo-30>
<CollectionClient id=naip>
<CollectionClient id=cop-dem-glo-90>
<CollectionClient id=landsat-c2-l2>
<CollectionClient id=sentinel-2-l2a>
<CollectionClient id=sentinel-2-l1c>
<CollectionClient id=sentinel-2-c1-l2a>
<CollectionClient id=sentinel-1-grd>
```
