---
author: Kathy Dooley
date: 2022-05-23
slug: lcmap-virtual-dataset
draft: True
type: post
title: "Moving toward multidimensional virtual geospatial datasets"
author_email: <kdooley@usgs.gov>
author_github: kathymd
description: How GIS raster datasets could benefit from being updated to multidimensional cloud-hosted objects.
categories:
  - data-science
tags:
  - data-science
  - xarray
  - zarr
  - aws
  - lcmap
  - GIS
  - netCDF
  - GeoTiff
---

The USGS collects and distributes massive amounts of geospatial data. Satellite imagery, streamgage measurements, and water quality tests all contain water data tied to geographic locations. These data and products derived from them are often published as publicly-accessible geospatial files. These vector and raster datasets take the form of two-dimensional shapefiles and GeoTiffs. Following a typical analyst's workflow, these tifs would be downloaded individually onto a local machine and processed using one of many softwares and/or programming languages, with the final output saved back to a local drive. All of this takes considerable time, memory, and data cleaning and processing. By converting tifs into multidimensional objects hosted in cloud services and accessing them as virtual datasets, we can save time, eliminate the need for local memory for data storage, and streamline workflows.

We recently explored the potential for tifs to be converted to multidimensional virtual datasets by testing [LCMAP](https://eros.usgs.gov/lcmap/apps/data-downloads) (Land Change Monitoring, Assessment, and Projection), which is a USGS dataset derived from Landsat imagery. It covers the continental United States and the data are available for download as single tifs, with one file per land cover product per year. There are eleven different products (variables) and 36 years, for a total of 396 LCMAP tifs.

## Multidimensional datasets

Whenever there are multiple tif files containing the same type of data to create a single dataset, odds are that they can be combined into one multidimensional object. It is common for a single file to include the full lat/lon spatial extent, but contain only one time-step, elevation, or variable. LCMAP follows this trend; each file is two-dimensional, covering the same lat/lon extent, with a separate file created for each year and product. Although it is hard to imagine working with a dataset where each file contains multiple time and height values for a single lat/lon point, which would force the user to download 36 different files to conduct an analysis across 36 different lat/lon coordinates, if we switch the dimensions around, we realize we frequently encounter similar situations. We don't give a second-thought to downloading 36 files for 36 different years, even if our analysis is confined to a small geographic extent.

There are opportunities to move away from the two-dimensional standard. Multidimensional geospatial data formats, such as netCDFs, have been prevalent in atmospheric and oceanic research for many years, and Python packages such as `xarray` specialize in working with multidimensional data. The 396 separate LCMAP files could be a single multidimensional object.

## Virtual datasets

Virtual datasets, meaning the data can be hosted from cloud services and accessed and inspected without downloading the data to users' local machines, can save substantial time and memory. With each tif hovering around 1.1GB, downloading all 396 of them would total approximately 436GB of storage space. Although times would vary greatly depending on internet speed, when testing the download times for this post, it took 15 minutes to download five files, which would roughly translate to 20 hours for the full dataset.

Emerging Python Packages, such as `fsspec`, allow users to work with remote file systems and read in data as virtual datasets without physically downloading files. These virtual datasets are compatible with several packages created for working with multidimensional and geospatial data, which allow users to analyze, manipulate, and save new data objects to the cloud service without touching their computer's memory.

## Multidimensional virtual datasets

If our dataset only had the benefits of being multidimensional, without being virtual, or vice-versa, we would still encounter challenges. It is by combining the multidimensional data object with virtual dataset capabilities that enables users to work with all LCMAP products in all available years for the exact geographic extent of their choice. A multidimensional dataset would still cause time and memory problems when it is downloaded to a local machine. Users have no ability to choose a subset of the data; they would then be forced to download all years, products, and geographic extents. Similarly, a virtual dataset that consists of hundreds of two-dimensional files would not allow users to easily slice the dataset to their specific dimensions and parameters. A multidimensional virtual dataset offers the ability to specify the exact data needed, inspect the data, run an analysis, and save new output back to the cloud, without ever downloading data to a local machine.

## A real example

The discussion of a multidimensional virtual LCMAP dataset is not purely theoretical. We tested this concept by downloading all 36 LCMAP primary land cover tifs, converting them into a format that can be accessed as a single multidimensional object, and hosting the data on a cloud service. We were then able to read this multidimensional object as a virtual dataset, query this dataset by specific parameters, and create simple visualizations.

Specifically, we used `xarray` to convert the tifs to netCDFs and generated a single JSON with `kerchunk`, which allows the data to be accessed in Zarr format. These data were uploaded to Amazon S3. While there are 36 netCDF files on S3, the user does not interact with the individual files. Instead, the JSON contains the combined metadata for the netCDFs, as well as the file paths to the data. Through using `fsspec` and `xarray`, we can open, view, and manipulate the data stored in the netCDFs as a single multidimensional object. The full workflow is available at: [path to gitlab?]
