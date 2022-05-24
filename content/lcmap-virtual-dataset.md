---
author: Kathy Dooley
date: 2022-05-23
slug: lcmap-virtual-dataset
draft: True
image: static/snakemake-ml-experiments/dag_few_cases.png
type: post
title: Optimizing Access to Large Geospatial Datasets
author_email: <kdooley@usgs.gov>
author_github: kathymd
description: How to transform LCMAP geotiffs into cloud optimized datasets.
categories:
  - data-science
tags:
  - data-science
  - xarray
  - zarr
  - aws
  - lcmap
---

Landsat, a USGS satellite, collects imagery used for monitoring land use and land cover. Landsat imagery, as well as the USGS geospatial data products derived from this imagery, are available for public download as GeoTiffs. Following a typical analyst's workflow, these GeoTiffs would be downloaded individually onto a local machine and processed using one of many softwares and/or programming languages, with the final output saved back to a local drive. All of this takes considerable time, memory, and processing power. Here we discuss one method for mitigating these problems through transforming GeoTiffs into cloud-optimized datasets and hosting them in cloud server.

Specifically, we use [LCMAP](https://eros.usgs.gov/lcmap/apps/data-downloads) as our example Landsat product, and demonstrate how to convert the tifs to netCDFs and a single JSON, which allows the data to be accessed in Zarr format. We host these data using Amazon Simple Storage Service (S3). This will allow users to access, analyze, and inspect LCMAP (Land Change Monitoring, Assessment, and Projection) data without downloading and processing in on their local machines.

## About LCMAP

LCMAP is a USGS annual land use/land cover dataset for the continental United States derived from LandSat imagery. The original data used in this notebook are available [here](https://eros.usgs.gov/lcmap/apps/data-downloads).

- **Temporal resolution:** annual
- **Spatial resolution:** 30x30m
- **Spatial Extent:** CONUS, originally divided into 150 x 150km tiles
- **Timeframe:** 1985-2020
- **Format:** GeoTIFF + XML metadata
- **File Size:** ~1GB for each product
- **Projection:** custom projection based on Albers Equal Area Conic (AEA)
- **Datum:** World Geodetic System 1984 (WGS84)

## Conda Environment

We can use `lcmap_env.yml` to replicate the environment built for this demo by running `mamba env create -f lcmap_env.yml` or `conda env create -f lcmap_env.yml`.

---

## Workflow

### Data Download

#### Summary

In this section, we locate the data download urls, download the data, unzip the files, and delete the zipped files.

#### Python Packages

- [fsspec](https://filesystem-spec.readthedocs.io/en/latest/): for remote file systems
- [zipfile](https://docs.python.org/3/library/zipfile.html): for unzipping files
- [os](https://docs.python.org/3/library/os.html): for using operating system functionality
- [urllib](https://docs.python.org/3/library/urllib.request.html#module-urllib.request): for working with urls

#### Data Source

Before we begin downloading the data, we must identify both the data source and where we are going to store the data. There are several different options for downloading LCMAP; in this notebook we use [LCMAP Product Mosaics](https://eros.usgs.gov/lcmap/apps/data-downloads), hosted by USGS EROS. Although LCMAP data are initially produced in tiles congruent with LandSat imagery, EROS provides a mosaic of these tiles, resulting in one tif for all of CONUS. Each tif corresponds with one land cover/land use variable for each year.

There are 11 different variables (products). In this example, we use primary land cover. To find the correct name for the url of your product of interest, (defined here as `LCMAP_product`), visit [LCMAP Product Mosaics](https://eros.usgs.gov/lcmap/apps/data-downloads), select your product of interest from the dropdown, right-click the download button for any year, copy the url, and look for the text listed after `/full_extent_downloads/version_12/`. The `LCMAP_product_abbr` is located between `V12` and `.zip`.

### Data Processing

#### Summary

In this section, we take the original tifs, convert them to netCDFs, and store them on AWS S3. We then create a JSON to correspond with each netCDF. We then combine the individual JSON files into one JSON and host it on AWS S3. The metadata encoded in this JSON allows users to access and inspect the data in the netCDFs.

#### Python Packages

- [xarray](https://docs.xarray.dev/en/stable/): for multi-dimensional arrays
- [rasterio](https://rasterio.readthedocs.io/en/latest/): for geospatial data
- [geopandas](https://geopandas.org/en/stable/): for geospatial data
- [kerchunk](https://github.com/fsspec/kerchunk.git): for chunked, compressed data formats
- [ujson](https://pypi.org/project/ujson/): for json encoding and decoding
- [re](https://docs.python.org/3/library/re.html): for regular expressions

Write files to an AWS S3 bucket. This requires AWS credentials. Once you have your credentials, use `aws configure` to [create a named profile](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-profiles.html). In this example, we use a profile named `nhgf-s3` to write to a USGS CHS S3 bucket.

Consolidate JSONS and add time attributes

LCMAP files have no time dimension or time coordinate variable; kerchunk can construct a time varible for us, creating times from the file names

Upload final JSON to S3

Geospatial datasets are traditionally grouped into two categories: raster and vector. Vector geospatial datasets are typically synonomous with shapefiles, while the word 'raster' and 'geotiff' are often used interchangeably among geospatial analysts.

The USGS collects and distributes large environmental geospatial datasets. These datasets are typically available in the form of tables, shapefiles, or GeoTiffs. GeoTiffs often contain gridded data
