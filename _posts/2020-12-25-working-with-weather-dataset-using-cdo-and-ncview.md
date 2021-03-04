---
layout: post
title:  Working with Weather Dataset using CDO and NCView
categories: [Data Analysis, Climate Data]
excerpt: In working with climate data, tools such as CDO and NCView are useful to manipulate, analyse and visualize them.
---
# Introduction

[CDO (Climate Data Operators)](https://code.mpimet.mpg.de/projects/cdo)  is a set of command line Operators to manipulate and analyse Climate data. netCDF 3/4 is one of the supported formats which we demonstrate here.

[NCView](http://meteora.ucsd.edu/~pierce/ncview_home_page.html) is a visual browser for netCDF format files. It runs on UNIX platforms under X11.

[ncdump](https://www.unidata.ucar.edu/software/netcdf/workshops/2011/utilities/Ncdump.html) command-line utility converts netCDF data to human-readable text form.

# CDO

Extract a subset of dataset. In this case, selecting months 6-9 and writing to a file.

```bash
cdo -f nc4 -selmon,6/9 Rainfall.nc Summar_Rainfall.nc
```

Calculate yearly mean

```shell
cdo -f nc4 -yearmean Summar_Rainfall.nc year_mean.nc
```

Trend analysis / linear regression

```shell
cdo -f nc4 trend year_mean.nc trend1.nc trend2.nc
```

Export .nc to .tsv
```shell
cdo outputtab,year,lon,lat,value year_mean.nc > year_mean.tsv
```

# ncdump

Dump info about a dataset (similar to pandas `pd.head()`)

```bash
ncdump -h Rainfall.nc
```

Dump detailed info about specific variable

```bash
ncdump -t -v lat Rainfall.nc
ncdump -t -v time Rainfall.nc
```

# NCView

Open `ncview` with a dataset.

```bash
ncview Summar_Rainfall.nc
```


