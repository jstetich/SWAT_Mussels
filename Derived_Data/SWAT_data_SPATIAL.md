Maine DEP EGAD Mussel Tissue Toxics Spatial Data
================
Curtis C. Bohlen, Casco Bay Estuary Partnership
9/10/2020

  - [Introduction](#introduction)
  - [Load Libraries](#load-libraries)
  - [Load Data](#load-data)
      - [Establish Folder Reference](#establish-folder-reference)
      - [Copy TOXICS Data](#copy-toxics-data)
      - [Copy Geographic Locations](#copy-geographic-locations)
      - [Write CSV file for GIS](#write-csv-file-for-gis)

<img
  src="https://www.cascobayestuary.org/wp-content/uploads/2014/04/logo_sm.jpg"
  style="position:absolute;top:10px;right:50px;" />

# Introduction

Maine’s Department of Environmental Protection (DEP) maintains a large
database of environmental data called “EGAD”. Citizens can request data
from the database through DEP staff.

CBEP requested data from DEP on levels of toxic contaminants in
shellfish tissue samples from Casco Bay. The result is a large (\>
100,000 line) excel spreadsheet containing data from about 40 sampling
dates from 20 locations, over a period of more than 15 years.

Unfortunately, the data delivery contains little metadata, so it takes
some effort to understand the data format and analyze it correctly.
Among other problems, we need to understand dates and locations of
samples, what analytes were used for different samples, etc.

In this notebook we extract geospatial data from source data files for
further analysis. This Notebook generates the following derived data
files:

`sample_spatial.csv` `sites_spatial.csv`

# Load Libraries

``` r
library(tidyverse)
```

    ## -- Attaching packages ------------------------------------ tidyverse 1.3.0 --

    ## v ggplot2 3.3.2     v purrr   0.3.4
    ## v tibble  3.0.3     v dplyr   1.0.2
    ## v tidyr   1.1.2     v stringr 1.4.0
    ## v readr   1.3.1     v forcats 0.5.0

    ## -- Conflicts --------------------------------------- tidyverse_conflicts() --
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

``` r
library(readxl)
#library(htmltools)  # used by knitr called here only to avoid startup text later in document
library(knitr)
```

# Load Data

## Establish Folder Reference

``` r
siblingfldnm <- 'Original_Data'
parent   <- dirname(getwd())
sibling  <- file.path(parent,siblingfldnm)
fn <- 'CascoBaySWATtissue_Bohlen.xlsx'
```

## Copy TOXICS Data

This is a larger data file that takes some time to load. Getting the
column types right dramatically improves load speed. Much of the data is
qualitative, and can’t be handled in R.

``` r
SWAT_data <- read_excel(file.path(sibling, fn), 
    sheet = "Mussels Data", col_types = c("numeric", 
        "text", "text", "text", "text", "text", 
        "text", "text", "text", "text", "text", 
        "text", "date", "text", "text", 
        "text", "date", "text", "numeric", 
        "text", "text", "text", "text", 
        "text", "numeric", "numeric", "text", 
        "text", "text", "text", "text", 
        "text", "numeric", "text", 
        "text", "text", "text", "text", 
        "text", "text", "text"))

before <- nrow(SWAT_data)
```

### List of Sites

We pull apart the site name into a site code and a longer name.

``` r
sites <- SWAT_data %>%
  select(`SITE SEQ`, `EGAD_SITE_NAME`) %>%
  group_by(`SITE SEQ`, `EGAD_SITE_NAME`) %>%
  summarize(SiteCode =  first(sub('.* - ','', `EGAD_SITE_NAME`)), 
            Site =  first(sub(' - .*','', `EGAD_SITE_NAME`)),
                        .groups = 'drop') %>%
  select(-`EGAD_SITE_NAME`)
#write_csv(sites, 'cb_SWAT_sites_list.csv')
kable(sites)
```

| SITE SEQ | SiteCode | Site                               |
| -------: | :------- | :--------------------------------- |
|    70672 | CBBBBB   | BACK BAY                           |
|    70674 | CBFROR   | FORE RIVER OUTER                   |
|    70675 | CBGDCC   | COCKTAIL COVE GREAT DIAMOND ISLAND |
|    70676 | CBGDSW   | SOUTHWEST END GREAT DIAMOND ISLAND |
|    70677 | CBHWNP   | NAVY PIER                          |
|    70678 | CBMCMC   | MILL CREEK                         |
|    70679 | CBRYMT   | ROYAL RIVER MOUTH                  |
|    76071 | CBHRHR   | HARASEEKET RIVER                   |
|    76073 | CBANAN   | FALMOUTH ANCHORAGE                 |
|    76076 | CBMBBH   | BRUNSWICK MARE BROOK DRAINAGE      |
|    76079 | CBFRMR   | MIDDLE FORE RIVER                  |
|    76080 | CBEEEE   | EAST END BEACH                     |
|    76084 | CBSPSP   | S PORTLAND SPRING POINT            |
|    76091 | CBJWPB   | JEWEL ISLAND PUNCHBOWL             |
|    77495 | CBPRMT   | PRESUMPSCOT RIVER (MOUTH)          |
|    77497 | CBMBMB   | MIDDLE BAY (OUTER)                 |
|    82251 | CBMBBR   | MAQUOIT BAY                        |
|    82256 | CBFRIR   | INNER FORE RIVER                   |
|    82261 | CBQHQH   | QUAHOG BAY                         |
|    82266 | CBLNFT   | LONG ISLAND                        |

## Copy Geographic Locations

In addition to the toxics data, we received a separate Excel file
containing geospatial data. That data includes repeat geographic data
for each site, often also providing information on separate sample
collection locations within sites. As far as we have been able to tell,
however, there is no consistent sample identifier between the geographic
data and toxics data as received.

We lack precise metadata on geographic coordinates. We do know locations
were collected in most cases with hand-held GPS units, which suggests
lat-long data are probably in WGS 1984.

Here we assemble an initial geospatial data set by reading in the source
data, and formatting it for GIS use.

### Complete Data, Including Sample Locations

Data on Sample Locations shows locations were at some sites collected
over a kilometer of shoreline or more, however, data on Sample Locations
appears incomplete, and so the best we can do is work with the site
data, which while lower resolution is complete for all samples.

``` r
fn <- 'CascoBay_SWAT_Spatial.xlsx'
samples_spatial_data <- read_excel(file.path(sibling, fn))%>%
  mutate(`SITE UTM X` = as.numeric(`SITE UTM X`),
         `SITE UTM Y` = as.numeric(`SITE UTM Y`),
         `SITE LATITUDE` = as.numeric(`SITE LATITUDE`),
         `SITE LONGITUDE` = as.numeric(`SITE LONGITUDE`),
         `SAMPLE POINT UTM X` = as.numeric(`SAMPLE POINT UTM X`),
         `SAMPLE POINT UTM Y` = as.numeric(`SAMPLE POINT UTM Y`),
         `SAMPLE POINT LATITUDE` = as.numeric(`SAMPLE POINT LATITUDE`),
         `SAMPLE POINT LONGITUDE` = as.numeric(`SAMPLE POINT LONGITUDE`),
  )
```

### Site Locations Only

Reduced data set, which will be the basis for most mapping. Note that
this data set (at this point) includes some locations that were used for
samples of tissue from other shellfish.

``` r
sites_spatial_data <- samples_spatial_data  %>%
  rename(SITESEQ = `SITE SEQ`) %>%
    group_by(SITESEQ) %>%
    summarize(SITECODE  = first(sites$SiteCode[match(SITESEQ, sites$`SITE SEQ`)]),
              SITE  = first(sites$Site[match(SITESEQ, sites$`SITE SEQ`)]),
              UTM_E = first(`SITE UTM X`),
              UTM_N = first(`SITE UTM Y`),
              LAT   = first(`SITE LATITUDE`),
              LONG  = first(`SITE LONGITUDE`),
              .groups = 'drop') 
```

## Write CSV file for GIS

We filter out sites that were NOT included in the mussel tissue toxics
data.

Without na = ’‘, literal ’NA’ in the output was sometimes forcing ArcGIS
to interpret data columns as text, forcing conversions in GIS.

``` r
sites_spatial_data <- sites_spatial_data %>%
  filter(! is.na(SITECODE))
write_csv(sites_spatial_data, 'sites_spatial.csv', na = '')
```

ArcGIS was having trouble reading earlier drafts of the following file,
so we simplified by removing spaces from column names and removing NAs.

``` r
samples_spatial_data %>%
  select ( -`SITE UTM X`, -`SITE UTM Y`, -`SITE LATITUDE`, -`SITE LONGITUDE`,
           - DATA_SOURCE, -LOCATION_ACCURACY, -LOCATION_METHOD) %>%
  rename_all(~gsub(' ', '', .)) %>%
  write_csv('samples_spatial.csv', na = '')
```
