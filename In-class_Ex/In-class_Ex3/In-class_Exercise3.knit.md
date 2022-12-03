---
title: "In-class_Ex3"
editor: visual
author: Zhao Yuetong
date: 12/3/2022
---


# Geographical Segmentation with Spatially Constrained Clustering Techniques

## Overview

In this hands-on exercise, you will gain hands-on experience on how to delineate homogeneous region by using geographically referenced multivariate data. There are two major analysis, namely:

-   hierarchical cluster analysis; and

-   spatially constrained cluster analysis.

## Getting Started

### The analytical question

In geobusiness and spatial policy, it is a common practice to delineate the market or planning area into homogeneous regions by using multivariate data. In this hands-on exercise, we are interested to delineate [Shan State](https://en.wikipedia.org/wiki/Shan_State), [Myanmar](https://en.wikipedia.org/wiki/Myanmar) into homogeneous regions by using multiple Information and Communication technology (ICT) measures, namely: Radio, Television, Land line phone, Mobile phone, Computer, and Internet at home.

## The data

Two data sets will be used in this study. They are:

-   Myanmar Township Boundary Data (i.e. *myanmar_township_boundaries*) : This is a GIS data in ESRI shapefile format. It consists of township boundary information of Myanmar. The spatial data are captured in polygon features.

-   *Shan-ICT.csv*: This is an extract of [**The 2014 Myanmar Population and Housing Census Myanmar**](https://myanmar.unfpa.org/en/publications/2014-population-and-housing-census-myanmar-data-sheet) at the township level.

Both data sets are download from [Myanmar Information Management Unit (MIMU)](http://themimu.info/)

## Installing and loading R packages

Before we get started, it is important for us to install the necessary R packages into R and launch these R packages into R environment.

The R packages needed for this exercise are as follows:

-   Spatial data handling

    -   **sf**, **rgdal** and **spdep**

-   Attribute data handling

    -   **tidyverse**, especially **readr**, **ggplot2** and **dplyr**

-   Choropleth mapping

    -   **tmap**

-   Multivariate data visualisation and analysis

    -   **coorplot**, **ggpubr**, and **heatmaply**

-   Cluster analysis

    -   **cluster**

    -   **ClustGeo**

The code chunks below installs and launches these R packages into R environment.


::: {.cell}

```{.r .cell-code}
pacman::p_load(rgdal, spdep, tmap, sf, ClustGeo, 
               ggpubr, cluster, factoextra, NbClust,
               heatmaply, corrplot, psych, tidyverse)
```
:::


## Data Import and Prepatation

### Importing geospatial data into R environment

In this section, you will import Myanmar Township Boundary GIS data and its associated attrbiute table into R environment.

The Myanmar Township Boundary GIS data is in ESRI shapefile format. It will be imported into R environment by using the [*st_read()*](https://www.rdocumentation.org/packages/sf/versions/0.7-2/topics/st_read) function of **sf**.

The code chunks used are shown below:


::: {.cell}

```{.r .cell-code}
shan_sf <- st_read(dsn = "data/geospatial", 
                   layer = "myanmar_township_boundaries") %>%
  filter(ST %in% c("Shan (East)", "Shan (North)", "Shan (South)"))
```

::: {.cell-output .cell-output-stdout}
```
Reading layer `myanmar_township_boundaries' from data source 
  `D:\yuetongz\ISSS624\In-class_Ex\In-class_Ex3\data\geospatial' 
  using driver `ESRI Shapefile'
Simple feature collection with 330 features and 14 fields
Geometry type: MULTIPOLYGON
Dimension:     XY
Bounding box:  xmin: 92.17275 ymin: 9.671252 xmax: 101.1699 ymax: 28.54554
Geodetic CRS:  WGS 84
```
:::
:::


The imported township boundary object is called *shan_sf*. It is saved in **simple feature data.frame** format. We can view the content of the newly created *shan_sf* simple features data.frame by using the code chunk below.


::: {.cell}

```{.r .cell-code}
shan_sf
```

::: {.cell-output .cell-output-stdout}
```
Simple feature collection with 55 features and 14 fields
Geometry type: MULTIPOLYGON
Dimension:     XY
Bounding box:  xmin: 96.15107 ymin: 19.29932 xmax: 101.1699 ymax: 24.15907
Geodetic CRS:  WGS 84
First 10 features:
   OBJECTID           ST ST_PCODE       DT   DT_PCODE        TS  TS_PCODE
1       163 Shan (North)   MMR015  Mongmit MMR015D008   Mongmit MMR015017
2       203 Shan (South)   MMR014 Taunggyi MMR014D001   Pindaya MMR014006
3       240 Shan (South)   MMR014 Taunggyi MMR014D001   Ywangan MMR014007
4       106 Shan (South)   MMR014 Taunggyi MMR014D001  Pinlaung MMR014009
5        72 Shan (North)   MMR015  Mongmit MMR015D008    Mabein MMR015018
6        40 Shan (South)   MMR014 Taunggyi MMR014D001     Kalaw MMR014005
7       194 Shan (South)   MMR014 Taunggyi MMR014D001     Pekon MMR014010
8       159 Shan (South)   MMR014 Taunggyi MMR014D001  Lawksawk MMR014008
9        61 Shan (North)   MMR015  Kyaukme MMR015D003 Nawnghkio MMR015013
10      124 Shan (North)   MMR015  Kyaukme MMR015D003   Kyaukme MMR015012
                 ST_2            LABEL2 SELF_ADMIN ST_RG T_NAME_WIN T_NAME_M3
1  Shan State (North)    Mongmit\n61072       <NA> State   rdk;rdwf      မိုးမိတ်
2  Shan State (South)    Pindaya\n77769       Danu State     yif;w,     ပင်းတယ
3  Shan State (South)    Ywangan\n76933       Danu State      &GmiH       ရွာငံ
4  Shan State (South)  Pinlaung\n162537       Pa-O State  yifavmif;   ပင်လောင်း
5  Shan State (North)     Mabein\n35718       <NA> State     rbdrf;      မဘိမ်း
6  Shan State (South)     Kalaw\n163138       <NA> State       uavm      ကလော
7  Shan State (South)      Pekon\n94226       <NA> State     z,fcHk       ဖယ်ခုံ
8  Shan State (South)          Lawksawk       <NA> State   &yfapmuf    ရပ်စောက်
9  Shan State (North) Nawnghkio\n128357       <NA> State  aemifcsdK    နောင်ချို
10 Shan State (North)   Kyaukme\n172874       <NA> State   ausmufrJ    ကျောက်မဲ
       AREA                       geometry
1  2703.611 MULTIPOLYGON (((96.96001 23...
2   629.025 MULTIPOLYGON (((96.7731 21....
3  2984.377 MULTIPOLYGON (((96.78483 21...
4  3396.963 MULTIPOLYGON (((96.49518 20...
5  5034.413 MULTIPOLYGON (((96.66306 24...
6  1456.624 MULTIPOLYGON (((96.49518 20...
7  2073.513 MULTIPOLYGON (((97.14738 19...
8  5145.659 MULTIPOLYGON (((96.94981 22...
9  3271.537 MULTIPOLYGON (((96.75648 22...
10 3920.869 MULTIPOLYGON (((96.95498 22...
```
:::
:::


Notice that sf.data.frame is conformed to Hardy Wickham's [tidy](https://edzer.github.io/rstudio_conf/#1) framework.

Since *shan_sf* is conformed to tidy framework, we can also *glimpse()* to reveal the data type of it's fields.


::: {.cell}

```{.r .cell-code}
glimpse(shan_sf)
```

::: {.cell-output .cell-output-stdout}
```
Rows: 55
Columns: 15
$ OBJECTID   <dbl> 163, 203, 240, 106, 72, 40, 194, 159, 61, 124, 71, 155, 101…
$ ST         <chr> "Shan (North)", "Shan (South)", "Shan (South)", "Shan (Sout…
$ ST_PCODE   <chr> "MMR015", "MMR014", "MMR014", "MMR014", "MMR015", "MMR014",…
$ DT         <chr> "Mongmit", "Taunggyi", "Taunggyi", "Taunggyi", "Mongmit", "…
$ DT_PCODE   <chr> "MMR015D008", "MMR014D001", "MMR014D001", "MMR014D001", "MM…
$ TS         <chr> "Mongmit", "Pindaya", "Ywangan", "Pinlaung", "Mabein", "Kal…
$ TS_PCODE   <chr> "MMR015017", "MMR014006", "MMR014007", "MMR014009", "MMR015…
$ ST_2       <chr> "Shan State (North)", "Shan State (South)", "Shan State (So…
$ LABEL2     <chr> "Mongmit\n61072", "Pindaya\n77769", "Ywangan\n76933", "Pinl…
$ SELF_ADMIN <chr> NA, "Danu", "Danu", "Pa-O", NA, NA, NA, NA, NA, NA, NA, NA,…
$ ST_RG      <chr> "State", "State", "State", "State", "State", "State", "Stat…
$ T_NAME_WIN <chr> "rdk;rdwf", "yif;w,", "&GmiH", "yifavmif;", "rbdrf;", "uavm…
$ T_NAME_M3  <chr> "မိုးမိတ်", "ပင်းတယ", "ရွာငံ", "ပင်လောင်း", "မဘိမ်း", "ကလော", "ဖယ်ခုံ", "…
$ AREA       <dbl> 2703.611, 629.025, 2984.377, 3396.963, 5034.413, 1456.624, …
$ geometry   <MULTIPOLYGON [°]> MULTIPOLYGON (((96.96001 23..., MULTIPOLYGON (…
```
:::
:::


### Importing aspatial data into R environment

The csv file will be import using *read_csv* function of **readr** package.

The code chunks used are shown below:


::: {.cell}

```{.r .cell-code}
ict <- read_csv ("data/aspatial/Shan-ICT.csv")
```

::: {.cell-output .cell-output-stderr}
```
Rows: 55 Columns: 11
── Column specification ────────────────────────────────────────────────────────
Delimiter: ","
chr (4): District Pcode, District Name, Township Pcode, Township Name
dbl (7): Total households, Radio, Television, Land line phone, Mobile phone,...

ℹ Use `spec()` to retrieve the full column specification for this data.
ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```
:::
:::


The imported InfoComm variables are extracted from **The 2014 Myanmar Population and Housing Census Myanmar**. The attribute data set is called *ict*. It is saved in R's \* tibble data.frame\* format.

The code chunk below reveal the summary statistics of *ict* data.frame.


::: {.cell}

```{.r .cell-code}
summary(ict)
```

::: {.cell-output .cell-output-stdout}
```
 District Pcode     District Name      Township Pcode     Township Name     
 Length:55          Length:55          Length:55          Length:55         
 Class :character   Class :character   Class :character   Class :character  
 Mode  :character   Mode  :character   Mode  :character   Mode  :character  
                                                                            
                                                                            
                                                                            
 Total households     Radio         Television    Land line phone 
 Min.   : 3318    Min.   :  115   Min.   :  728   Min.   :  20.0  
 1st Qu.: 8711    1st Qu.: 1260   1st Qu.: 3744   1st Qu.: 266.5  
 Median :13685    Median : 2497   Median : 6117   Median : 695.0  
 Mean   :18369    Mean   : 4487   Mean   :10183   Mean   : 929.9  
 3rd Qu.:23471    3rd Qu.: 6192   3rd Qu.:13906   3rd Qu.:1082.5  
 Max.   :82604    Max.   :30176   Max.   :62388   Max.   :6736.0  
  Mobile phone      Computer      Internet at home
 Min.   :  150   Min.   :  20.0   Min.   :   8.0  
 1st Qu.: 2037   1st Qu.: 121.0   1st Qu.:  88.0  
 Median : 3559   Median : 244.0   Median : 316.0  
 Mean   : 6470   Mean   : 575.5   Mean   : 760.2  
 3rd Qu.: 7177   3rd Qu.: 507.0   3rd Qu.: 630.5  
 Max.   :48461   Max.   :6705.0   Max.   :9746.0  
```
:::
:::


There are a total of eleven fields and 55 observation in the tibble data.frame.

### Derive new variables using **dplyr** package

The unit of measurement of the values are number of household. Using these values directly will be bias by the underlying total number of households. In general, the townships with relatively higher total number of households will also have higher number of households owning radio, TV, etc.

In order to overcome this problem, we will derive the penetration rate of each ICT variable by using the code chunk below.


::: {.cell}

```{.r .cell-code}
ict_derived <- ict %>%
  mutate(`RADIO_PR` = `Radio`/`Total households`*1000) %>%
  mutate(`TV_PR` = `Television`/`Total households`*1000) %>%
  mutate(`LLPHONE_PR` = `Land line phone`/`Total households`*1000) %>%
  mutate(`MPHONE_PR` = `Mobile phone`/`Total households`*1000) %>%
  mutate(`COMPUTER_PR` = `Computer`/`Total households`*1000) %>%
  mutate(`INTERNET_PR` = `Internet at home`/`Total households`*1000) %>%
  rename(`DT_PCODE` =`District Pcode`,`DT`=`District Name`,
         `TS_PCODE`=`Township Pcode`, `TS`=`Township Name`,
         `TT_HOUSEHOLDS`=`Total households`,
         `RADIO`=`Radio`, `TV`=`Television`, 
         `LLPHONE`=`Land line phone`, `MPHONE`=`Mobile phone`,
         `COMPUTER`=`Computer`, `INTERNET`=`Internet at home`) 
```
:::


Let us review the summary statistics of the newly derived penetration rates using the code chunk below.


::: {.cell}

```{.r .cell-code}
summary(ict_derived)
```

::: {.cell-output .cell-output-stdout}
```
   DT_PCODE              DT              TS_PCODE              TS           
 Length:55          Length:55          Length:55          Length:55         
 Class :character   Class :character   Class :character   Class :character  
 Mode  :character   Mode  :character   Mode  :character   Mode  :character  
                                                                            
                                                                            
                                                                            
 TT_HOUSEHOLDS       RADIO             TV           LLPHONE      
 Min.   : 3318   Min.   :  115   Min.   :  728   Min.   :  20.0  
 1st Qu.: 8711   1st Qu.: 1260   1st Qu.: 3744   1st Qu.: 266.5  
 Median :13685   Median : 2497   Median : 6117   Median : 695.0  
 Mean   :18369   Mean   : 4487   Mean   :10183   Mean   : 929.9  
 3rd Qu.:23471   3rd Qu.: 6192   3rd Qu.:13906   3rd Qu.:1082.5  
 Max.   :82604   Max.   :30176   Max.   :62388   Max.   :6736.0  
     MPHONE         COMPUTER         INTERNET         RADIO_PR     
 Min.   :  150   Min.   :  20.0   Min.   :   8.0   Min.   : 21.05  
 1st Qu.: 2037   1st Qu.: 121.0   1st Qu.:  88.0   1st Qu.:138.95  
 Median : 3559   Median : 244.0   Median : 316.0   Median :210.95  
 Mean   : 6470   Mean   : 575.5   Mean   : 760.2   Mean   :215.68  
 3rd Qu.: 7177   3rd Qu.: 507.0   3rd Qu.: 630.5   3rd Qu.:268.07  
 Max.   :48461   Max.   :6705.0   Max.   :9746.0   Max.   :484.52  
     TV_PR         LLPHONE_PR       MPHONE_PR       COMPUTER_PR    
 Min.   :116.0   Min.   :  2.78   Min.   : 36.42   Min.   : 3.278  
 1st Qu.:450.2   1st Qu.: 22.84   1st Qu.:190.14   1st Qu.:11.832  
 Median :517.2   Median : 37.59   Median :305.27   Median :18.970  
 Mean   :509.5   Mean   : 51.09   Mean   :314.05   Mean   :24.393  
 3rd Qu.:606.4   3rd Qu.: 69.72   3rd Qu.:428.43   3rd Qu.:29.897  
 Max.   :842.5   Max.   :181.49   Max.   :735.43   Max.   :92.402  
  INTERNET_PR     
 Min.   :  1.041  
 1st Qu.:  8.617  
 Median : 22.829  
 Mean   : 30.644  
 3rd Qu.: 41.281  
 Max.   :117.985  
```
:::
:::


## Exploratory Data Analysis (EDA)

### EDA using statistical graphics

We can plot the distribution of the variables (i.e. Number of households with radio) by using appropriate Exploratory Data Analysis (EDA) as shown in the code chunk below.

Histogram is useful to identify the overall distribution of the data values (i.e. left skew, right skew or normal distribution)


::: {.cell}

```{.r .cell-code}
ggplot(data=ict_derived, 
       aes(x=`RADIO`)) +
  geom_histogram(bins=20, 
                 color="black", 
                 fill="light blue")
```

::: {.cell-output-display}
![](In-class_Exercise3_files/figure-html/unnamed-chunk-9-1.png){width=672}
:::
:::


Boxplot is useful to detect if there are outliers.


::: {.cell}

```{.r .cell-code}
ggplot(data=ict_derived, 
       aes(x=`RADIO`)) +
  geom_boxplot(color="black", 
               fill="light blue")
```

::: {.cell-output-display}
![](In-class_Exercise3_files/figure-html/unnamed-chunk-10-1.png){width=672}
:::
:::


Next, we will also plotting the distribution of the newly derived variables (i.e. Radio penetration rate) by using the code chunk below.


::: {.cell}

```{.r .cell-code}
ggplot(data=ict_derived, 
       aes(x=`RADIO_PR`)) +
  geom_histogram(bins=20, 
                 color="black", 
                 fill="light blue")
```

::: {.cell-output-display}
![](In-class_Exercise3_files/figure-html/unnamed-chunk-11-1.png){width=672}
:::
:::

::: {.cell}

```{.r .cell-code}
ggplot(data=ict_derived, 
       aes(x=`RADIO_PR`)) +
  geom_boxplot(color="black", 
               fill="light blue")
```

::: {.cell-output-display}
![](In-class_Exercise3_files/figure-html/unnamed-chunk-12-1.png){width=672}
:::
:::


What can you observed from the distributions reveal in the histogram and boxplot.

In the figure below, multiple histograms are plotted to reveal the distribution of the selected variables in the *ict_derived* data.frame.

The code chunks below are used to create the data visualisation. They consist of two main parts. First, we will create the individual histograms using the code chunk below.


::: {.cell}

```{.r .cell-code}
radio <- ggplot(data=ict_derived, 
             aes(x= `RADIO_PR`)) +
  geom_histogram(bins=20, 
                 color="black", 
                 fill="light blue")

tv <- ggplot(data=ict_derived, 
             aes(x= `TV_PR`)) +
  geom_histogram(bins=20, 
                 color="black", 
                 fill="light blue")

llphone <- ggplot(data=ict_derived, 
             aes(x= `LLPHONE_PR`)) +
  geom_histogram(bins=20, 
                 color="black", 
                 fill="light blue")

mphone <- ggplot(data=ict_derived, 
             aes(x= `MPHONE_PR`)) +
  geom_histogram(bins=20, 
                 color="black", 
                 fill="light blue")

computer <- ggplot(data=ict_derived, 
             aes(x= `COMPUTER_PR`)) +
  geom_histogram(bins=20, 
                 color="black", 
                 fill="light blue")

internet <- ggplot(data=ict_derived, 
             aes(x= `INTERNET_PR`)) +
  geom_histogram(bins=20, 
                 color="black", 
                 fill="light blue")
```
:::


Next, the [*ggarange()*](https://rpkgs.datanovia.com/ggpubr/reference/ggarrange.html) function of [**ggpubr**](https://rpkgs.datanovia.com/ggpubr/) package is used to group these histograms together.


::: {.cell}

```{.r .cell-code}
ggarrange(radio, tv, llphone, mphone, computer, internet, 
          ncol = 3, 
          nrow = 2)
```

::: {.cell-output-display}
![](In-class_Exercise3_files/figure-html/unnamed-chunk-14-1.png){width=672}
:::
:::


### EDA using choropleth map

#### Joining geospatial data with aspatial data

Before we can prepare the choropleth map, we need to combine both the geospatial data object (i.e. *shan_sf*) and aspatial data.frame object (i.e. *ict_derived*) into one. This will be performed by using the [*left_join*](https://dplyr.tidyverse.org/reference/join.tbl_df.html) function of **dplyr** package. The *shan_sf* simple feature data.frame will be used as the base data object and the *ict_derived* data.frame will be used as the join table.

The code chunks below is used to perform the task. The unique identifier used to join both data objects is *TS_PCODE*.


::: {.cell}

```{.r .cell-code}
shan_sf <- left_join(shan_sf, 
                     ict_derived, 
                     by=c("TS_PCODE"="TS_PCODE"))
```
:::


The message above shows that *TS_CODE* field is the common field used to perform the left-join.

It is important to note that there is no new output data been created. Instead, the data fields from *ict_derived* data frame are now updated into the data frame of *shan_sf*.

#### Preparing a choropleth map

To have a quick look at the distribution of Radio penetration rate of Shan State at township level, a choropleth map will be prepared.

The code chunks below are used to prepare the choroplethby using the *qtm()* function of **tmap** package.


::: {.cell}

```{.r .cell-code}
qtm(shan_sf, "RADIO_PR")
```

::: {.cell-output-display}
![](In-class_Exercise3_files/figure-html/unnamed-chunk-16-1.png){width=672}
:::
:::


In order to reveal the distribution shown in the choropleth map above are bias to the underlying total number of households at the townships, we will create two choropleth maps, one for the total number of households (i.e. TT_HOUSEHOLDS.map) and one for the total number of household with Radio (RADIO.map) by using the code chunk below.


::: {.cell}

```{.r .cell-code}
TT_HOUSEHOLDS.map <- tm_shape(shan_sf) + 
  tm_fill(col = "TT_HOUSEHOLDS",
          n = 5,
          style = "jenks", 
          title = "Total households") + 
  tm_borders(alpha = 0.5) 

RADIO.map <- tm_shape(shan_sf) + 
  tm_fill(col = "RADIO",
          n = 5,
          style = "jenks",
          title = "Number Radio ") + 
  tm_borders(alpha = 0.5) 

tmap_arrange(TT_HOUSEHOLDS.map, RADIO.map,
             asp=NA, ncol=2)
```

::: {.cell-output-display}
![](In-class_Exercise3_files/figure-html/unnamed-chunk-17-1.png){width=672}
:::
:::


Notice that the choropleth maps above clearly show that townships with relatively larger number ot households are also showing relatively higher number of radio ownership.

Now let us plot the choropleth maps showing the dsitribution of total number of households and Radio penetration rate by using the code chunk below.


::: {.cell}

```{.r .cell-code}
tm_shape(shan_sf) +
    tm_polygons(c("TT_HOUSEHOLDS", "RADIO_PR"),
                style="jenks") +
    tm_facets(sync = TRUE, ncol = 2) +
  tm_legend(legend.position = c("right", "bottom"))+
  tm_layout(outer.margins=0, asp=0)
```

::: {.cell-output-display}
![](In-class_Exercise3_files/figure-html/unnamed-chunk-18-1.png){width=672}
:::
:::


## Correlation Analysis

Before we perform cluster analysis, it is important for us to ensure that the cluster variables are not highly correlated.

In this section, you will learn how to use [*corrplot.mixed()*](https://cran.r-project.org/web/packages/corrplot/corrplot.pdf) function of [**corrplot**](https://cran.r-project.org/web/packages/corrplot/vignettes/corrplot-intro.html) package to visualise and analyse the correlation of the input variables.


::: {.cell}

```{.r .cell-code}
cluster_vars.cor = cor(ict_derived[,12:17])
corrplot.mixed(cluster_vars.cor,
         lower = "ellipse", 
               upper = "number",
               tl.pos = "lt",
               diag = "l",
               tl.col = "black")
```

::: {.cell-output-display}
![](In-class_Exercise3_files/figure-html/unnamed-chunk-19-1.png){width=672}
:::
:::


The correlation plot above shows that COMPUTER_PR and INTERNET_PR are highly correlated. This suggest that only one of them should be used in the cluster analysis instead of both.

## Hierarchy Cluster Analysis

In this section, you will learn how to perform hierarchical cluster analysis. The analysis consists of four major steps:

### Extrating clustering variables

The code chunk below will be used to extract the clustering variables from the *shan_sf* simple feature object into data.frame.


::: {.cell}

```{.r .cell-code}
cluster_vars <- shan_sf %>%
  st_set_geometry(NULL) %>%
  select("TS.x", "RADIO_PR", "TV_PR", "LLPHONE_PR", "MPHONE_PR", "COMPUTER_PR")
head(cluster_vars,10)
```

::: {.cell-output .cell-output-stdout}
```
        TS.x RADIO_PR    TV_PR LLPHONE_PR MPHONE_PR COMPUTER_PR
1    Mongmit 286.1852 554.1313   35.30618  260.6944    12.15939
2    Pindaya 417.4647 505.1300   19.83584  162.3917    12.88190
3    Ywangan 484.5215 260.5734   11.93591  120.2856     4.41465
4   Pinlaung 231.6499 541.7189   28.54454  249.4903    13.76255
5     Mabein 449.4903 708.6423   72.75255  392.6089    16.45042
6      Kalaw 280.7624 611.6204   42.06478  408.7951    29.63160
7      Pekon 318.6118 535.8494   39.83270  214.8476    18.97032
8   Lawksawk 387.1017 630.0035   31.51366  320.5686    21.76677
9  Nawnghkio 349.3359 547.9456   38.44960  323.0201    15.76465
10   Kyaukme 210.9548 601.1773   39.58267  372.4930    30.94709
```
:::
:::


Notice that the final clustering variables list does not include variable INTERNET_PR because it is highly correlated with variable COMPUTER_PR.

Next, we need to change the rows by township name instead of row number by using the code chunk below


::: {.cell}

```{.r .cell-code}
row.names(cluster_vars) <- cluster_vars$"TS.x"
head(cluster_vars,10)
```

::: {.cell-output .cell-output-stdout}
```
               TS.x RADIO_PR    TV_PR LLPHONE_PR MPHONE_PR COMPUTER_PR
Mongmit     Mongmit 286.1852 554.1313   35.30618  260.6944    12.15939
Pindaya     Pindaya 417.4647 505.1300   19.83584  162.3917    12.88190
Ywangan     Ywangan 484.5215 260.5734   11.93591  120.2856     4.41465
Pinlaung   Pinlaung 231.6499 541.7189   28.54454  249.4903    13.76255
Mabein       Mabein 449.4903 708.6423   72.75255  392.6089    16.45042
Kalaw         Kalaw 280.7624 611.6204   42.06478  408.7951    29.63160
Pekon         Pekon 318.6118 535.8494   39.83270  214.8476    18.97032
Lawksawk   Lawksawk 387.1017 630.0035   31.51366  320.5686    21.76677
Nawnghkio Nawnghkio 349.3359 547.9456   38.44960  323.0201    15.76465
Kyaukme     Kyaukme 210.9548 601.1773   39.58267  372.4930    30.94709
```
:::
:::


Notice that the row number has been replaced into the township name.

Now, we will delete the TS.x field by using the code chunk below.


::: {.cell}

```{.r .cell-code}
shan_ict <- select(cluster_vars, c(2:6))
head(shan_ict, 10)
```

::: {.cell-output .cell-output-stdout}
```
          RADIO_PR    TV_PR LLPHONE_PR MPHONE_PR COMPUTER_PR
Mongmit   286.1852 554.1313   35.30618  260.6944    12.15939
Pindaya   417.4647 505.1300   19.83584  162.3917    12.88190
Ywangan   484.5215 260.5734   11.93591  120.2856     4.41465
Pinlaung  231.6499 541.7189   28.54454  249.4903    13.76255
Mabein    449.4903 708.6423   72.75255  392.6089    16.45042
Kalaw     280.7624 611.6204   42.06478  408.7951    29.63160
Pekon     318.6118 535.8494   39.83270  214.8476    18.97032
Lawksawk  387.1017 630.0035   31.51366  320.5686    21.76677
Nawnghkio 349.3359 547.9456   38.44960  323.0201    15.76465
Kyaukme   210.9548 601.1773   39.58267  372.4930    30.94709
```
:::
:::


### Data Standardisation

In general, multiple variables will be used in cluster analysis. It is not unusual their values range are different. In order to avoid the cluster analysis result is baised to clustering variables with large values, it is useful to standardise the input variables before performing cluster analysis.

### Min-Max standardisation

In the code chunk below, *normalize()* of [*heatmaply*](https://cran.r-project.org/web/packages/heatmaply/) package is used to stadardisation the clustering variables by using Min-Max method. The *summary()* is then used to display the summary statistics of the standardised clustering variables.


::: {.cell}

```{.r .cell-code}
shan_ict.std <- normalize(shan_ict)
summary(shan_ict.std)
```

::: {.cell-output .cell-output-stdout}
```
    RADIO_PR          TV_PR          LLPHONE_PR       MPHONE_PR     
 Min.   :0.0000   Min.   :0.0000   Min.   :0.0000   Min.   :0.0000  
 1st Qu.:0.2544   1st Qu.:0.4600   1st Qu.:0.1123   1st Qu.:0.2199  
 Median :0.4097   Median :0.5523   Median :0.1948   Median :0.3846  
 Mean   :0.4199   Mean   :0.5416   Mean   :0.2703   Mean   :0.3972  
 3rd Qu.:0.5330   3rd Qu.:0.6750   3rd Qu.:0.3746   3rd Qu.:0.5608  
 Max.   :1.0000   Max.   :1.0000   Max.   :1.0000   Max.   :1.0000  
  COMPUTER_PR     
 Min.   :0.00000  
 1st Qu.:0.09598  
 Median :0.17607  
 Mean   :0.23692  
 3rd Qu.:0.29868  
 Max.   :1.00000  
```
:::
:::


Notice that the values range of the Min-max standardised clustering variables are 0-1 now.

### Z-score standardisation

Z-score standardisation can be performed easily by using [*scale()*](https://www.rdocumentation.org/packages/base/versions/3.6.2/topics/scale) of Base R. The code chunk below will be used to stadardisation the clustering variables by using Z-score method.


::: {.cell}

```{.r .cell-code}
shan_ict.z <- scale(shan_ict)
describe(shan_ict.z)
```

::: {.cell-output .cell-output-stdout}
```
            vars  n mean sd median trimmed  mad   min  max range  skew kurtosis
RADIO_PR       1 55    0  1  -0.04   -0.06 0.94 -1.85 2.55  4.40  0.48    -0.27
TV_PR          2 55    0  1   0.05    0.04 0.78 -2.47 2.09  4.56 -0.38    -0.23
LLPHONE_PR     3 55    0  1  -0.33   -0.15 0.68 -1.19 3.20  4.39  1.37     1.49
MPHONE_PR      4 55    0  1  -0.05   -0.06 1.01 -1.58 2.40  3.98  0.48    -0.34
COMPUTER_PR    5 55    0  1  -0.26   -0.18 0.64 -1.03 3.31  4.34  1.80     2.96
              se
RADIO_PR    0.13
TV_PR       0.13
LLPHONE_PR  0.13
MPHONE_PR   0.13
COMPUTER_PR 0.13
```
:::
:::


Notice the mean and standard deviation of the Z-score standardised clustering variables are 0 and 1 respectively.

### Visualising the standardised clustering variables

Beside reviewing the summary statistics of the standardised clustering variables, it is also a good practice to visualise their distribution graphical.

The code chunk below plot the scaled *Radio_PR* field.


::: {.cell}

```{.r .cell-code}
r <- ggplot(data=ict_derived, 
             aes(x= `RADIO_PR`)) +
  geom_histogram(bins=20, 
                 color="black", 
                 fill="light blue")

shan_ict_s_df <- as.data.frame(shan_ict.std)
s <- ggplot(data=shan_ict_s_df, 
       aes(x=`RADIO_PR`)) +
  geom_histogram(bins=20, 
                 color="black", 
                 fill="light blue") +
  ggtitle("Min-Max Standardisation")

shan_ict_z_df <- as.data.frame(shan_ict.z)
z <- ggplot(data=shan_ict_z_df, 
       aes(x=`RADIO_PR`)) +
  geom_histogram(bins=20, 
                 color="black", 
                 fill="light blue") +
  ggtitle("Z-score Standardisation")

ggarrange(r, s, z,
          ncol = 3,
          nrow = 1)
```

::: {.cell-output-display}
![](In-class_Exercise3_files/figure-html/unnamed-chunk-25-1.png){width=672}
:::
:::


Notice that the overall distribution of the clustering variables will change after the data standardisation. Hence, it is advisible **NOT** to perform data standardisation if the values range of the clustering variables are not very large.

### Computing proximity matrix

In R, many packages provide functions to calculate distance matrix. We will compute the proximity matrix by using [*dist()*](https://stat.ethz.ch/R-manual/R-devel/library/stats/html/dist.html) of R.

*dist()* supports six distance proximity calculations, they are: **euclidean, maximum, manhattan, canberra, binary and minkowski**. The default is *euclidean* proximity matrix.

The code chunk below is used to compute the proximity matrix using *euclidean* method.


::: {.cell}

```{.r .cell-code}
proxmat <- dist(shan_ict, method = 'euclidean')
```
:::


The code chunk below can then be used to list the content of *proxmat* for visual inspection.


::: {.cell}

```{.r .cell-code}
proxmat
```

::: {.cell-output .cell-output-stdout}
```
             Mongmit   Pindaya   Ywangan  Pinlaung    Mabein     Kalaw
Pindaya    171.86828                                                  
Ywangan    381.88259 257.31610                                        
Pinlaung    57.46286 208.63519 400.05492                              
Mabein     263.37099 313.45776 529.14689 312.66966                    
Kalaw      160.05997 302.51785 499.53297 181.96406 198.14085          
Pekon       59.61977 117.91580 336.50410  94.61225 282.26877 211.91531
Lawksawk   140.11550 204.32952 432.16535 192.57320 130.36525 140.01101
Nawnghkio   89.07103 180.64047 377.87702 139.27495 204.63154 127.74787
Kyaukme    144.02475 311.01487 505.89191 139.67966 264.88283  79.42225
Muse       563.01629 704.11252 899.44137 571.58335 453.27410 412.46033
Laihka     141.87227 298.61288 491.83321 101.10150 345.00222 197.34633
Mongnai    115.86190 258.49346 422.71934  64.52387 358.86053 200.34668
Mawkmai    434.92968 437.99577 397.03752 398.11227 693.24602 562.59200
Kutkai      97.61092 212.81775 360.11861  78.07733 340.55064 204.93018
Mongton    192.67961 283.35574 361.23257 163.42143 425.16902 267.87522
Mongyai    256.72744 287.41816 333.12853 220.56339 516.40426 386.74701
Mongkaing  503.61965 481.71125 364.98429 476.29056 747.17454 625.24500
Lashio     251.29457 398.98167 602.17475 262.51735 231.28227 106.69059
Mongpan    193.32063 335.72896 483.68125 192.78316 301.52942 114.69105
Matman     401.25041 354.39039 255.22031 382.40610 637.53975 537.63884
Tachileik  529.63213 635.51774 807.44220 555.01039 365.32538 373.64459
Narphan    406.15714 474.50209 452.95769 371.26895 630.34312 463.53759
Mongkhet   349.45980 391.74783 408.97731 305.86058 610.30557 465.52013
Hsipaw     118.18050 245.98884 388.63147  76.55260 366.42787 212.36711
Monghsat   214.20854 314.71506 432.98028 160.44703 470.48135 317.96188
Mongmao    242.54541 402.21719 542.85957 217.58854 384.91867 195.18913
Nansang    104.91839 275.44246 472.77637  85.49572 287.92364 124.30500
Laukkaing  568.27732 726.85355 908.82520 563.81750 520.67373 427.77791
Pangsang   272.67383 428.24958 556.82263 244.47146 418.54016 224.03998
Namtu      179.62251 225.40822 444.66868 170.04533 366.16094 307.27427
Monghpyak  177.76325 221.30579 367.44835 222.20020 212.69450 167.08436
Konkyan    403.39082 500.86933 528.12533 365.44693 613.51206 444.75859
Mongping   265.12574 310.64850 337.94020 229.75261 518.16310 375.64739
Hopong     136.93111 223.06050 352.85844  98.14855 398.00917 264.16294
Nyaungshwe  99.38590 216.52463 407.11649 138.12050 210.21337  95.66782
Hsihseng   131.49728 172.00796 342.91035 111.61846 381.20187 287.11074
Mongla     384.30076 549.42389 728.16301 372.59678 406.09124 260.26411
Hseni      189.37188 337.98982 534.44679 204.47572 213.61240  38.52842
Kunlong    224.12169 355.47066 531.63089 194.76257 396.61508 273.01375
Hopang     281.05362 443.26362 596.19312 265.96924 368.55167 185.14704
Namhkan    386.02794 543.81859 714.43173 382.78835 379.56035 246.39577
Kengtung   246.45691 385.68322 573.23173 263.48638 219.47071  88.29335
Langkho    164.26299 323.28133 507.78892 168.44228 253.84371  67.19580
Monghsu    109.15790 198.35391 340.42789  80.86834 367.19820 237.34578
Taunggyi   399.84278 503.75471 697.98323 429.54386 226.24011 252.26066
Pangwaun   381.51246 512.13162 580.13146 356.37963 523.44632 338.35194
Kyethi     202.92551 175.54012 287.29358 189.47065 442.07679 360.17247
Loilen     145.48666 293.61143 469.51621  91.56527 375.06406 217.19877
Manton     430.64070 402.42888 306.16379 405.83081 674.01120 560.16577
Mongyang   309.51302 475.93982 630.71590 286.03834 411.88352 233.56349
Kunhing    173.50424 318.23811 449.67218 141.58836 375.82140 197.63683
Mongyawng  214.21738 332.92193 570.56521 235.55497 193.49994 173.43078
Tangyan    195.92520 208.43740 324.77002 169.50567 448.59948 348.06617
Namhsan    237.78494 228.41073 286.16305 214.33352 488.33873 385.88676
               Pekon  Lawksawk Nawnghkio   Kyaukme      Muse    Laihka
Pindaya                                                               
Ywangan                                                               
Pinlaung                                                              
Mabein                                                                
Kalaw                                                                 
Pekon                                                                 
Lawksawk   157.51129                                                  
Nawnghkio  113.15370  90.82891                                        
Kyaukme    202.12206 186.29066 157.04230                              
Muse       614.56144 510.13288 533.68806 434.75768                    
Laihka     182.23667 246.74469 211.88187 128.24979 526.65211          
Mongnai    151.60031 241.71260 182.21245 142.45669 571.97975 100.53457
Mawkmai    416.00669 567.52693 495.15047 512.02846 926.93007 429.96554
Kutkai     114.98048 224.64646 147.44053 170.93318 592.90743 144.67198
Mongton    208.14888 311.07742 225.81118 229.28509 634.71074 212.07320
Mongyai    242.52301 391.26989 319.57938 339.27780 763.91399 264.13364
Mongkaing  480.23965 625.18712 546.69447 586.05094 995.66496 522.96309
Lashio     303.80011 220.75270 230.55346 129.95255 313.15288 238.64533
Mongpan    243.30037 228.54223 172.84425 110.37831 447.49969 210.76951
Matman     368.25761 515.39711 444.05061 505.52285 929.11283 443.25453
Tachileik  573.39528 441.82621 470.45533 429.15493 221.19950 549.08985
Narphan    416.84901 523.69580 435.59661 420.30003 770.40234 392.32592
Mongkhet   342.08722 487.41102 414.10280 409.03553 816.44931 324.97428
Hsipaw     145.37542 249.35081 176.09570 163.95741 591.03355 128.42987
Monghsat   225.64279 352.31496 289.83220 253.25370 663.76026 158.93517
Mongmao    293.70625 314.64777 257.76465 146.09228 451.82530 185.99082
Nansang    160.37607 188.78869 151.13185  60.32773 489.35308  78.78999
Laukkaing  624.82399 548.83928 552.65554 428.74978 149.26996 507.39700
Pangsang   321.81214 345.91486 287.10769 175.35273 460.24292 214.19291
Namtu      165.02707 260.95300 257.52713 270.87277 659.16927 185.86794
Monghpyak  190.93173 142.31691  93.03711 217.64419 539.43485 293.22640
Konkyan    421.48797 520.31264 439.34272 393.79911 704.86973 351.75354
Mongping   259.68288 396.47081 316.14719 330.28984 744.44948 272.82761
Hopong     138.86577 274.91604 204.88286 218.84211 648.68011 157.48857
Nyaungshwe 139.31874 104.17830  43.26545 126.50414 505.88581 201.71653
Hsihseng   105.30573 257.11202 209.88026 250.27059 677.66886 175.89761
Mongla     441.20998 393.18472 381.40808 241.58966 256.80556 315.93218
Hseni      243.98001 171.50398 164.05304  81.20593 381.30567 204.49010
Kunlong    249.36301 318.30406 285.04608 215.63037 547.24297 122.68682
Hopang     336.38582 321.16462 279.84188 154.91633 377.44407 230.78652
Namhkan    442.77120 379.41126 367.33575 247.81990 238.67060 342.43665
Kengtung   297.67761 209.38215 208.29647 136.23356 330.08211 258.23950
Langkho    219.21623 190.30257 156.51662  51.67279 413.64173 160.94435
Monghsu    113.84636 242.04063 170.09168 200.77712 633.21624 163.28926
Taunggyi   440.66133 304.96838 344.79200 312.60547 250.81471 425.36916
Pangwaun   423.81347 453.02765 381.67478 308.31407 541.97887 351.78203
Kyethi     162.43575 317.74604 267.21607 328.14177 757.16745 255.83275
Loilen     181.94596 265.29318 219.26405 146.92675 560.43400  59.69478
Manton     403.82131 551.13000 475.77296 522.86003 941.49778 458.30232
Mongyang   363.58788 363.37684 323.32123 188.59489 389.59919 229.71502
Kunhing    213.46379 278.68953 206.15773 145.00266 533.00162 142.03682
Mongyawng  248.43910 179.07229 220.61209 181.55295 422.37358 211.99976
Tangyan    167.79937 323.14701 269.07880 306.78359 736.93741 224.29176
Namhsan    207.16559 362.84062 299.74967 347.85944 778.52971 273.79672
             Mongnai   Mawkmai    Kutkai   Mongton   Mongyai Mongkaing
Pindaya                                                               
Ywangan                                                               
Pinlaung                                                              
Mabein                                                                
Kalaw                                                                 
Pekon                                                                 
Lawksawk                                                              
Nawnghkio                                                             
Kyaukme                                                               
Muse                                                                  
Laihka                                                                
Mongnai                                                               
Mawkmai    374.50873                                                  
Kutkai      91.15307 364.95519                                        
Mongton    131.67061 313.35220 107.06341                              
Mongyai    203.23607 178.70499 188.94166 159.79790                    
Mongkaing  456.00842 133.29995 428.96133 365.50032 262.84016          
Lashio     270.86983 638.60773 289.82513 347.11584 466.36472 708.65819
Mongpan    178.09554 509.99632 185.18173 200.31803 346.39710 563.56780
Matman     376.33870 147.83545 340.86349 303.04574 186.95158 135.51424
Tachileik  563.95232 919.38755 568.99109 608.76740 750.29555 967.14087
Narphan    329.31700 273.75350 314.27683 215.97925 248.82845 285.65085
Mongkhet   275.76855 115.58388 273.91673 223.22828 104.98924 222.60577
Hsipaw      52.68195 351.34601  51.46282  90.69766 177.33790 423.77868
Monghsat   125.25968 275.09705 154.32012 150.98053 127.35225 375.60376
Mongmao    188.29603 485.52853 204.69232 206.57001 335.61300 552.31959
Nansang     92.79567 462.41938 130.04549 199.58124 288.55962 542.16609
Laukkaing  551.56800 882.51110 580.38112 604.66190 732.68347 954.11795
Pangsang   204.25746 484.14757 228.33583 210.77938 343.30638 548.40662
Namtu      209.35473 427.95451 225.28268 308.71751 278.02761 525.04057
Monghpyak  253.26470 536.71695 206.61627 258.04282 370.01575 568.21089
Konkyan    328.82831 339.01411 310.60810 248.25265 287.87384 380.92091
Mongping   202.99615 194.31049 182.75266 119.86993  65.38727 257.18572
Hopong      91.53795 302.84362  73.45899 106.21031 124.62791 379.37916
Nyaungshwe 169.63695 502.99026 152.15482 219.72196 327.13541 557.32112
Hsihseng   142.36728 329.29477 128.21054 194.64317 162.27126 411.59788
Mongla     354.10985 686.88950 388.40984 411.06668 535.28615 761.48327
Hseni      216.81639 582.53670 229.37894 286.75945 408.23212 648.04408
Kunlong    202.92529 446.53763 204.54010 270.02165 299.36066 539.91284
Hopang     243.00945 561.24281 263.31986 273.50305 408.73288 626.17673
Namhkan    370.05669 706.47792 392.48568 414.53594 550.62819 771.39688
Kengtung   272.28711 632.54638 279.19573 329.38387 460.39706 692.74693
Langkho    174.67678 531.08019 180.51419 236.70878 358.95672 597.42714
Monghsu     84.11238 332.07962  62.60859 107.04894 154.86049 400.71816
Taunggyi   448.55282 810.74692 450.33382 508.40925 635.94105 866.21117
Pangwaun   312.13429 500.68857 321.80465 257.50434 394.07696 536.95736
Kyethi     210.50453 278.85535 184.23422 222.52947 137.79420 352.06533
Loilen      58.41263 388.73386 131.56529 176.16001 224.79239 482.18190
Manton     391.54062 109.08779 361.82684 310.20581 195.59882  81.75337
Mongyang   260.39387 558.83162 285.33223 295.60023 414.31237 631.91325
Kunhing    110.55197 398.43973 108.84990 114.03609 238.99570 465.03971
Mongyawng  275.77546 620.04321 281.03383 375.22688 445.78964 700.98284
Tangyan    180.37471 262.66006 166.61820 198.88460 109.08506 348.56123
Namhsan    218.10003 215.19289 191.32762 196.76188  77.35900 288.66231
              Lashio   Mongpan    Matman Tachileik   Narphan  Mongkhet
Pindaya                                                               
Ywangan                                                               
Pinlaung                                                              
Mabein                                                                
Kalaw                                                                 
Pekon                                                                 
Lawksawk                                                              
Nawnghkio                                                             
Kyaukme                                                               
Muse                                                                  
Laihka                                                                
Mongnai                                                               
Mawkmai                                                               
Kutkai                                                                
Mongton                                                               
Mongyai                                                               
Mongkaing                                                             
Lashio                                                                
Mongpan    172.33279                                                  
Matman     628.11049 494.81014                                        
Tachileik  311.95286 411.03849 890.12935                              
Narphan    525.63854 371.13393 312.05193 760.29566                    
Mongkhet   534.44463 412.17123 203.02855 820.50164 217.28718          
Hsipaw     290.86435 179.52054 344.45451 576.18780 295.40170 253.80950
Monghsat   377.86793 283.30992 313.59911 677.09508 278.21548 167.98445
Mongmao    214.23677 131.59966 501.59903 472.95568 331.42618 375.35820
Nansang    184.47950 144.77393 458.06573 486.77266 398.13308 360.99219
Laukkaing  334.65738 435.58047 903.72094 325.06329 708.82887 769.06406
Pangsang   236.72516 140.23910 506.29940 481.31907 316.30314 375.58139
Namtu      365.88437 352.91394 416.65397 659.56458 494.36143 355.99713
Monghpyak  262.09281 187.85699 470.46845 444.04411 448.40651 462.63265
Konkyan    485.51312 365.87588 392.40306 730.92980 158.82353 254.24424
Mongping   454.52548 318.47482 201.65224 727.08969 188.64567 113.80917
Hopong     345.31042 239.43845 291.84351 632.45718 294.40441 212.99485
Nyaungshwe 201.58191 137.29734 460.91883 445.81335 427.94086 417.08639
Hsihseng   369.00833 295.87811 304.02806 658.87060 377.52977 256.70338
Mongla     179.95877 253.20001 708.17595 347.33155 531.46949 574.40292
Hseni       79.41836 120.66550 564.64051 354.90063 474.12297 481.88406
Kunlong    295.23103 288.03320 468.27436 595.70536 413.07823 341.68641
Hopang     170.63913 135.62913 573.55355 403.82035 397.85908 451.51070
Namhkan    173.27153 240.34131 715.42102 295.91660 536.85519 596.19944
Kengtung    59.85893 142.21554 613.01033 295.90429 505.40025 531.35998
Langkho    115.18145  94.98486 518.86151 402.33622 420.65204 428.08061
Monghsu    325.71557 216.25326 308.13805 605.02113 311.92379 247.73318
Taunggyi   195.14541 319.81385 778.45810 150.84117 684.20905 712.80752
Pangwaun   362.45608 232.52209 523.43600 540.60474 264.64997 407.02947
Kyethi     447.10266 358.89620 233.83079 728.87329 374.90376 233.25039
Loilen     268.92310 207.25000 406.56282 573.75476 354.79137 284.76895
Manton     646.66493 507.96808  59.52318 910.23039 280.26395 181.33894
Mongyang   209.33700 194.93467 585.61776 448.79027 401.39475 445.40621
Kunhing    255.10832 137.85278 403.66587 532.26397 281.62645 292.49814
Mongyawng  172.70139 275.15989 601.80824 432.10118 572.76394 522.91815
Tangyan    429.84475 340.39128 242.78233 719.84066 348.84991 201.49393
Namhsan    472.04024 364.77086 180.09747 754.03913 316.54695 170.90848
              Hsipaw  Monghsat   Mongmao   Nansang Laukkaing  Pangsang
Pindaya                                                               
Ywangan                                                               
Pinlaung                                                              
Mabein                                                                
Kalaw                                                                 
Pekon                                                                 
Lawksawk                                                              
Nawnghkio                                                             
Kyaukme                                                               
Muse                                                                  
Laihka                                                                
Mongnai                                                               
Mawkmai                                                               
Kutkai                                                                
Mongton                                                               
Mongyai                                                               
Mongkaing                                                             
Lashio                                                                
Mongpan                                                               
Matman                                                                
Tachileik                                                             
Narphan                                                               
Mongkhet                                                              
Hsipaw                                                                
Monghsat   121.78922                                                  
Mongmao    185.99483 247.17708                                        
Nansang    120.24428 201.92690 164.99494                              
Laukkaing  569.06099 626.44910 404.00848 480.60074                    
Pangsang   205.04337 256.37933  57.60801 193.36162 408.04016          
Namtu      229.44658 231.78673 365.03882 217.61884 664.06286 392.97391
Monghpyak  237.67919 356.84917 291.88846 227.52638 565.84279 315.11651
Konkyan    296.74316 268.25060 281.87425 374.70456 635.92043 274.81900
Mongping   168.92101 140.95392 305.57166 287.36626 708.13447 308.33123
Hopong      62.86179 100.45714 244.16253 167.66291 628.48557 261.51075
Nyaungshwe 169.92664 286.37238 230.45003 131.18943 520.24345 257.77823
Hsihseng   136.54610 153.49551 311.98001 193.53779 670.74564 335.52974
Mongla     373.47509 429.00536 216.24705 289.45119 202.55831 217.88123
Hseni      231.48538 331.22632 184.67099 136.45492 391.74585 214.66375
Kunlong    205.10051 202.31862 224.43391 183.01388 521.88657 258.49342
Hopang     248.72536 317.64824  78.29342 196.47091 331.67199  92.57672
Namhkan    382.79302 455.10875 223.32205 302.89487 196.46063 231.38484
Kengtung   284.08582 383.72138 207.58055 193.67980 351.48520 229.85484
Langkho    183.05109 279.52329 134.50170  99.39859 410.41270 167.65920
Monghsu     58.55724 137.24737 242.43599 153.59962 619.01766 260.52971
Taunggyi   462.31183 562.88102 387.33906 365.04897 345.98041 405.59730
Pangwaun   298.12447 343.53898 187.40057 326.12960 470.63605 157.48757
Kyethi     195.17677 190.50609 377.89657 273.02385 749.99415 396.89963
Loilen      98.04789 118.65144 190.26490  94.23028 535.57527 207.94433
Manton     359.60008 317.15603 503.79786 476.55544 907.38406 504.75214
Mongyang   267.10497 312.64797  91.06281 218.49285 326.19219 108.37735
Kunhing     90.77517 165.38834 103.91040 128.20940 500.41640 123.18870
Mongyawng  294.70967 364.40429 296.40789 191.11990 454.80044 336.16703
Tangyan    167.69794 144.59626 347.14183 249.70235 722.40954 364.76893
Namhsan    194.47928 169.56962 371.71448 294.16284 760.45960 385.65526
               Namtu Monghpyak   Konkyan  Mongping    Hopong Nyaungshwe
Pindaya                                                                
Ywangan                                                                
Pinlaung                                                               
Mabein                                                                 
Kalaw                                                                  
Pekon                                                                  
Lawksawk                                                               
Nawnghkio                                                              
Kyaukme                                                                
Muse                                                                   
Laihka                                                                 
Mongnai                                                                
Mawkmai                                                                
Kutkai                                                                 
Mongton                                                                
Mongyai                                                                
Mongkaing                                                              
Lashio                                                                 
Mongpan                                                                
Matman                                                                 
Tachileik                                                              
Narphan                                                                
Mongkhet                                                               
Hsipaw                                                                 
Monghsat                                                               
Mongmao                                                                
Nansang                                                                
Laukkaing                                                              
Pangsang                                                               
Namtu                                                                  
Monghpyak  346.57799                                                   
Konkyan    478.37690 463.39594                                         
Mongping   321.66441 354.76537 242.02901                               
Hopong     206.82668 267.95563 304.49287 134.00139                     
Nyaungshwe 271.41464 103.97300 432.35040 319.32583 209.32532           
Hsihseng   131.89940 285.37627 383.49700 199.64389  91.65458  225.80242
Mongla     483.49434 408.03397 468.09747 512.61580 432.31105  347.60273
Hseni      327.41448 200.26876 448.84563 395.58453 286.41193  130.86310
Kunlong    233.60474 357.44661 329.11433 309.05385 219.06817  285.13095
Hopang     408.24516 304.26577 348.18522 379.27212 309.77356  247.19891
Namhkan    506.32466 379.50202 481.59596 523.74815 444.13246  333.32428
Kengtung   385.33554 221.47613 474.82621 442.80821 340.47382  177.75714
Langkho    305.03473 200.27496 386.95022 343.96455 239.63685  128.26577
Monghsu    209.64684 232.17823 331.72187 158.90478  43.40665  173.82799
Taunggyi   518.72748 334.17439 650.56905 621.53039 513.76415  325.09619
Pangwaun   517.03554 381.95144 263.97576 340.37881 346.00673  352.92324
Kyethi     186.90932 328.16234 400.10989 187.43974 136.49038  288.06872
Loilen     194.24075 296.99681 334.19820 231.99959 124.74445  206.40432
Manton     448.58230 502.20840 366.66876 200.48082 310.58885  488.79874
Mongyang   413.26052 358.17599 329.39338 387.80686 323.35704  294.29500
Kunhing    296.43996 250.74435 253.74202 212.59619 145.15617  189.97131
Mongyawng  262.24331 285.56475 522.38580 455.59190 326.59925  218.12104
Tangyan    178.69483 335.26416 367.46064 161.67411 106.82328  284.14692
Namhsan    240.95555 352.70492 352.20115 130.23777 132.70541  315.91750
            Hsihseng    Mongla     Hseni   Kunlong    Hopang   Namhkan
Pindaya                                                               
Ywangan                                                               
Pinlaung                                                              
Mabein                                                                
Kalaw                                                                 
Pekon                                                                 
Lawksawk                                                              
Nawnghkio                                                             
Kyaukme                                                               
Muse                                                                  
Laihka                                                                
Mongnai                                                               
Mawkmai                                                               
Kutkai                                                                
Mongton                                                               
Mongyai                                                               
Mongkaing                                                             
Lashio                                                                
Mongpan                                                               
Matman                                                                
Tachileik                                                             
Narphan                                                               
Mongkhet                                                              
Hsipaw                                                                
Monghsat                                                              
Mongmao                                                               
Nansang                                                               
Laukkaing                                                             
Pangsang                                                              
Namtu                                                                 
Monghpyak                                                             
Konkyan                                                               
Mongping                                                              
Hopong                                                                
Nyaungshwe                                                            
Hsihseng                                                              
Mongla     478.66210                                                  
Hseni      312.74375 226.82048                                        
Kunlong    231.85967 346.46200 276.19175                              
Hopang     370.01334 147.02444 162.80878 271.34451                    
Namhkan    492.09476  77.21355 212.11323 375.73885 146.18632          
Kengtung   370.72441 202.45004  66.12817 317.14187 164.29921 175.63015
Langkho    276.27441 229.01675  66.66133 224.52741 134.24847 224.40029
Monghsu     97.82470 424.51868 262.28462 239.89665 301.84458 431.32637
Taunggyi   528.14240 297.09863 238.19389 471.29032 329.95252 257.29147
Pangwaun   433.06326 319.18643 330.70182 392.45403 206.98364 310.44067
Kyethi      84.04049 556.02500 388.33498 298.55859 440.48114 567.86202
Loilen     158.84853 338.67408 227.10984 166.53599 242.89326 364.90647
Manton     334.87758 712.51416 584.63341 479.76855 577.52046 721.86149
Mongyang   382.59743 146.66661 210.19929 247.22785  69.25859 167.72448
Kunhing    220.15490 306.47566 206.47448 193.77551 172.96164 314.92119
Mongyawng  309.51462 315.57550 173.86004 240.39800 290.51360 321.21112
Tangyan     70.27241 526.80849 373.07575 268.07983 412.22167 542.64078
Namhsan    125.74240 564.02740 411.96125 310.40560 440.51555 576.42717
            Kengtung   Langkho   Monghsu  Taunggyi  Pangwaun    Kyethi
Pindaya                                                               
Ywangan                                                               
Pinlaung                                                              
Mabein                                                                
Kalaw                                                                 
Pekon                                                                 
Lawksawk                                                              
Nawnghkio                                                             
Kyaukme                                                               
Muse                                                                  
Laihka                                                                
Mongnai                                                               
Mawkmai                                                               
Kutkai                                                                
Mongton                                                               
Mongyai                                                               
Mongkaing                                                             
Lashio                                                                
Mongpan                                                               
Matman                                                                
Tachileik                                                             
Narphan                                                               
Mongkhet                                                              
Hsipaw                                                                
Monghsat                                                              
Mongmao                                                               
Nansang                                                               
Laukkaing                                                             
Pangsang                                                              
Namtu                                                                 
Monghpyak                                                             
Konkyan                                                               
Mongping                                                              
Hopong                                                                
Nyaungshwe                                                            
Hsihseng                                                              
Mongla                                                                
Hseni                                                                 
Kunlong                                                               
Hopang                                                                
Namhkan                                                               
Kengtung                                                              
Langkho    107.16213                                                  
Monghsu    316.91914 221.84918                                        
Taunggyi   186.28225 288.27478 486.91951                              
Pangwaun   337.48335 295.38434 343.38498 497.61245                    
Kyethi     444.26274 350.91512 146.61572 599.57407 476.62610          
Loilen     282.22935 184.10672 131.55208 455.91617 331.69981 232.32965
Manton     631.99123 535.95620 330.76503 803.08034 510.79265 272.03299
Mongyang   217.08047 175.35413 323.95988 374.58247 225.25026 453.86726
Kunhing    245.95083 146.38284 146.78891 429.98509 229.09986 278.95182
Mongyawng  203.87199 186.11584 312.85089 287.73864 475.33116 387.71518
Tangyan    429.95076 332.02048 127.42203 592.65262 447.05580  47.79331
Namhsan    466.20497 368.20978 153.22576 631.49232 448.58030  68.67929
              Loilen    Manton  Mongyang   Kunhing Mongyawng   Tangyan
Pindaya                                                               
Ywangan                                                               
Pinlaung                                                              
Mabein                                                                
Kalaw                                                                 
Pekon                                                                 
Lawksawk                                                              
Nawnghkio                                                             
Kyaukme                                                               
Muse                                                                  
Laihka                                                                
Mongnai                                                               
Mawkmai                                                               
Kutkai                                                                
Mongton                                                               
Mongyai                                                               
Mongkaing                                                             
Lashio                                                                
Mongpan                                                               
Matman                                                                
Tachileik                                                             
Narphan                                                               
Mongkhet                                                              
Hsipaw                                                                
Monghsat                                                              
Mongmao                                                               
Nansang                                                               
Laukkaing                                                             
Pangsang                                                              
Namtu                                                                 
Monghpyak                                                             
Konkyan                                                               
Mongping                                                              
Hopong                                                                
Nyaungshwe                                                            
Hsihseng                                                              
Mongla                                                                
Hseni                                                                 
Kunlong                                                               
Hopang                                                                
Namhkan                                                               
Kengtung                                                              
Langkho                                                               
Monghsu                                                               
Taunggyi                                                              
Pangwaun                                                              
Kyethi                                                                
Loilen                                                                
Manton     419.06087                                                  
Mongyang   246.76592 585.70558                                        
Kunhing    130.39336 410.49230 188.89405                              
Mongyawng  261.75211 629.43339 304.21734 295.35984                    
Tangyan    196.60826 271.82672 421.06366 249.74161 377.52279          
Namhsan    242.15271 210.48485 450.97869 270.79121 430.02019  63.67613
```
:::
:::


### Computing hierarchical clustering

In R, there are several packages provide hierarchical clustering function. In this hands-on exercise, [*hclust()*](https://stat.ethz.ch/R-manual/R-devel/library/stats/html/hclust.html) of R stats will be used.

*hclust()* employed agglomeration method to compute the cluster. Eight clustering algorithms are supported, they are: ward.D, ward.D2, single, complete, average(UPGMA), mcquitty(WPGMA), median(WPGMC) and centroid(UPGMC).

The code chunk below performs hierarchical cluster analysis using ward.D method. The hierarchical clustering output is stored in an object of class **hclust** which describes the tree produced by the clustering process.


::: {.cell}

```{.r .cell-code}
hclust_ward <- hclust(proxmat, method = 'ward.D')
```
:::


We can then plot the tree by using *plot()* of R Graphics as shown in the code chunk below.


::: {.cell}

```{.r .cell-code}
plot(hclust_ward, cex = 0.6)
```

::: {.cell-output-display}
![](In-class_Exercise3_files/figure-html/unnamed-chunk-29-1.png){width=672}
:::
:::


### Selecting the optimal clustering algorithm

One of the challenge in performing hierarchical clustering is to identify stronger clustering structures. The issue can be solved by using use [*agnes()*](https://www.rdocumentation.org/packages/cluster/versions/2.1.0/topics/agnes) function of [**cluster**](https://cran.r-project.org/web/packages/cluster/) package. It functions like *hclus()*, however, with the *agnes()* function you can also get the agglomerative coefficient, which measures the amount of clustering structure found (values closer to 1 suggest strong clustering structure).

The code chunk below will be used to compute the agglomerative coefficients of all hierarchical clustering algorithms.


::: {.cell}

```{.r .cell-code}
m <- c( "average", "single", "complete", "ward")
names(m) <- c( "average", "single", "complete", "ward")

ac <- function(x) {
  agnes(shan_ict, method = x)$ac
}

map_dbl(m, ac)
```

::: {.cell-output .cell-output-stdout}
```
  average    single  complete      ward 
0.8131144 0.6628705 0.8950702 0.9427730 
```
:::
:::


With reference to the output above, we can see that Ward's method provides the strongest clustering structure among the four methods assessed. Hence, in the subsequent analysis, only Ward's method will be used.

### Determining Optimal Clusters

Another technical challenge face by data analyst in performing clustering analysis is to determine the optimal clusters to retain.

There are [three](https://www.datanovia.com/en/lessons/determining-the-optimal-number-of-clusters-3-must-know-methods/) commonly used methods to determine the optimal clusters, they are:

-   [Elbow Method](https://en.wikipedia.org/wiki/Elbow_method_(clustering))

-   [Average Silhouette Method](https://www.sciencedirect.com/science/article/pii/0377042787901257?via%3Dihub)

-   [Gap Statistic Method](https://statweb.stanford.edu/~gwalther/gap)

#### Gap Statistic Method

The [**gap statistic**](http://www.web.stanford.edu/~hastie/Papers/gap.pdf) compares the total within intra-cluster variation for different values of k with their expected values under null reference distribution of the data. The estimate of the optimal clusters will be value that maximize the gap statistic (i.e., that yields the largest gap statistic). This means that the clustering structure is far away from the random uniform distribution of points.

To compute the gap statistic, [*clusGap()*](https://www.rdocumentation.org/packages/cluster/versions/2.1.0/topics/clusGap) of [**cluster**](https://cran.r-project.org/web/packages/cluster/) package will be used.


::: {.cell}

```{.r .cell-code}
set.seed(12345)
gap_stat <- clusGap(shan_ict, 
                    FUN = hcut, 
                    nstart = 25, 
                    K.max = 10, 
                    B = 50)
# Print the result
print(gap_stat, method = "firstmax")
```

::: {.cell-output .cell-output-stdout}
```
Clustering Gap statistic ["clusGap"] from call:
clusGap(x = shan_ict, FUNcluster = hcut, K.max = 10, B = 50, nstart = 25)
B=50 simulated reference sets, k = 1..10; spaceH0="scaledPCA"
 --> Number of clusters (method 'firstmax'): 1
          logW   E.logW       gap     SE.sim
 [1,] 8.407129 8.680794 0.2736651 0.04460994
 [2,] 8.130029 8.350712 0.2206824 0.03880130
 [3,] 7.992265 8.202550 0.2102844 0.03362652
 [4,] 7.862224 8.080655 0.2184311 0.03784781
 [5,] 7.756461 7.978022 0.2215615 0.03897071
 [6,] 7.665594 7.887777 0.2221833 0.03973087
 [7,] 7.590919 7.806333 0.2154145 0.04054939
 [8,] 7.526680 7.731619 0.2049390 0.04198644
 [9,] 7.458024 7.660795 0.2027705 0.04421874
[10,] 7.377412 7.593858 0.2164465 0.04540947
```
:::
:::


Also note that the [*hcut*](https://rpkgs.datanovia.com/factoextra/reference/hcut.html) function used is from [**factoextra**](https://rpkgs.datanovia.com/factoextra/) package.

Next, we can visualise the plot by using [*fviz_gap_stat()*](https://rpkgs.datanovia.com/factoextra/reference/fviz_nbclust.html) of [**factoextra**](https://rpkgs.datanovia.com/factoextra/) package.


::: {.cell}

```{.r .cell-code}
fviz_gap_stat(gap_stat)
```

::: {.cell-output-display}
![](In-class_Exercise3_files/figure-html/unnamed-chunk-32-1.png){width=672}
:::
:::


With reference to the gap statistic graph above, the recommended number of cluster to retain is 1. However, it is not logical to retain only one cluster. By examine the gap statistic graph, the 6-cluster gives the largest gap statistic and should be the next best cluster to pick.

**Note:** In addition to these commonly used approaches, the [NbClust](https://cran.r-project.org/web/packages/NbClust/) package, published by Charrad et al., 2014, provides 30 indices for determining the relevant number of clusters and proposes to users the best clustering scheme from the different results obtained by varying all combinations of number of clusters, distance measures, and clustering methods.

### Interpreting the dendrograms

In the dendrogram displayed above, each leaf corresponds to one observation. As we move up the tree, observations that are similar to each other are combined into branches, which are themselves fused at a higher height.

The height of the fusion, provided on the vertical axis, indicates the (dis)similarity between two observations. The higher the height of the fusion, the less similar the observations are. Note that, conclusions about the proximity of two observations can be drawn only based on the height where branches containing those two observations first are fused. We cannot use the proximity of two observations along the horizontal axis as a criteria of their similarity.

It's also possible to draw the dendrogram with a border around the selected clusters by using [*rect.hclust()*](https://stat.ethz.ch/R-manual/R-devel/library/stats/html/rect.hclust.html) of R stats. The argument *border* is used to specify the border colors for the rectangles.


::: {.cell}

```{.r .cell-code}
plot(hclust_ward, cex = 0.6)
rect.hclust(hclust_ward, 
            k = 6, 
            border = 2:5)
```

::: {.cell-output-display}
![](In-class_Exercise3_files/figure-html/unnamed-chunk-33-1.png){width=672}
:::
:::


### Visually-driven hierarchical clustering analysis

In this section, we will learn how to perform visually-driven hiearchical clustering analysis by using [*heatmaply*](https://cran.r-project.org/web/packages/heatmaply/) package.

With **heatmaply**, we are able to build both highly interactive cluster heatmap or static cluster heatmap.

#### Transforming the data frame into a matrix

The data was loaded into a data frame, but it has to be a data matrix to make your heatmap.

The code chunk below will be used to transform *shan_ict* data frame into a data matrix.


::: {.cell}

```{.r .cell-code}
shan_ict_mat <- data.matrix(shan_ict)
```
:::


#### Plotting interactive cluster heatmap using *heatmaply()*

In the code chunk below, the [*heatmaply()*](https://talgalili.github.io/heatmaply/reference/heatmaply.html) of [heatmaply](https://talgalili.github.io/heatmaply/) package is used to build an interactive cluster heatmap.


::: {.cell}

```{.r .cell-code}
heatmaply(normalize(shan_ict_mat),
          Colv=NA,
          dist_method = "euclidean",
          hclust_method = "ward.D",
          seriate = "OLO",
          colors = Blues,
          k_row = 6,
          margins = c(NA,200,60,NA),
          fontsize_row = 4,
          fontsize_col = 5,
          main="Geographic Segmentation of Shan State by ICT indicators",
          xlab = "ICT Indicators",
          ylab = "Townships of Shan State"
          )
```

::: {.cell-output-display}
```{=html}
<div id="htmlwidget-9eceb20dc2a931752ae4" style="width:100%;height:464px;" class="plotly html-widget"></div>
<script type="application/json" data-for="htmlwidget-9eceb20dc2a931752ae4">{"x":{"data":[{"x":[1,2,3,4,5],"y":[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55],"z":[[0.116978282802357,0.129468002916701,0.22051676674467,0.384613999201764,0],[0.119071471357574,0.345093075355747,0.146680718622619,0.689156011292375,0.0306923094501804],[0.448112949126574,0.552259924114501,0.105215687600081,0.574129014267529,0.484807392774743],[0.394726524387261,0.646477338669936,0.19477119473094,0.398016081357507,0.307033923303471],[0.409738609943887,0.667785462581825,0.205939628052291,0.480784254803859,0.310457200410047],[0.560359050607694,0.682160053201915,0.219828980006303,0.532717729177229,0.295696949383387],[0.509783518401058,0.705849662481414,0.235277615529988,0.56572881621916,0.184217142327857],[0.445367826108455,0.648475186589583,0.400725374496756,0.525729584351687,0.325727661622098],[0.303006324046602,0.491472190827421,0.393413634291369,0.434988683676573,0.25539117902522],[0.285460565623123,0.653167734189465,0.351129977629807,0.321903369400093,0.274683722730117],[0.186896495593756,0.473621686944799,0.177547109767516,0.219017992119371,0.219238039033904],[0.262948885760044,0.600934420551276,0.105588681661233,0.309844721395711,0.32244406175046],[0.348384569046878,0.550413307218695,0.0132133117776572,0.330600515260223,0.280505026346822],[0.372116151704965,0.374968621142249,0.125060804628556,0.363502861923713,0.208286637250586],[0.382049084643175,0.495293243751349,0.165281214256075,0.332852029421132,0.126911060390332],[0.398090757930385,0.467734047728609,0.165251926083381,0.248272216763125,0.125663315216338],[0.463675071811737,0.480273489712105,0.0495176036018547,0.278454014117194,0.102240256271893],[0.454391396013767,0.585943534900779,0.144172529109699,0.304817572789695,0.117641144346071],[0.572059422162612,0.603028579702235,0.18200927177224,0.320846100415741,0.0996532209950878],[0.64202483918062,0.577864390148239,0.207338745441228,0.255258118966462,0.176074057695434],[0.469637019864802,0.49275331606036,0.336403906718691,0.326463285635126,0.199643002704722],[0.472437575771037,0.708306581316802,0.181528926899385,0.0981964513254065,0.186836974506664],[0.487021825059319,0.536184545909484,0.1623692849231,0.155474604691974,0.0744963103413769],[0.534163143993451,0.455760556744961,0.109548536966956,0.076155538926932,0.106925771850934],[0.437590296823069,0.464204461437082,0.110874787457869,0.0984403758574017,0.0985978869506665],[0.452636675796627,0.377797614010294,0.113655496814988,0.0877697005887191,0.0640685752289788],[0.318011520183019,0.341693315696117,0.0982231808523387,0.141213400161106,0.0684535703034676],[0.303905626461184,0.295734660011411,0.122161422028907,0.220797228887767,0.0933576373559664],[0.125839377932289,0.271665869542777,0.146912934029539,0.111904436916593,0.0401203752296987],[0.1648181386917,0.163070599120654,0,0,0.050562923174398],[0.292778031055123,0,0.0180206673088627,0.0205146898168363,0.0573419602932674],[0.373857483778625,0.0995290076456667,0.0295999006447705,0.0262900696870936,0.00411244014094081],[0.481303842133627,0.142297506536797,0.0316650514783228,0.0125730639984617,0.030854020566576],[1,0.198958169425964,0.0512340297746273,0.119978472699803,0.0127547808735244],[0.855314778071882,0.535580295292678,0.0954404378234391,0.180215108968874,0.107759971325446],[0.789802194306684,0.707463556363309,0.160787084615829,0.406501651460893,0.207451038928221],[0.653022272989167,0.603518277436298,0.0907572048090615,0.450320578796944,0.139286867591449],[0.708316665856044,0.59451431070864,0.199599204123416,0.410008712407441,0.140105344905725],[0.828481634580948,0.533096150331035,0.222253044854804,0.49414581896158,0.245321426698747],[0.924414874434519,0.815706630141315,0.391551407290058,0.509561826896415,0.147799896773415],[0.554582082266574,0.857899290321775,0.667007964836916,0.411501854757207,0.135186676818821],[0.217633980358742,0.65209843040106,1,0.287788284692333,0.196437751553831],[0,0.23543567048956,0.932101354458042,0.401015569024513,0.0844217926569466],[0.109374929636258,0.623942179795541,0.622050632093838,0.618872994218249,0.107369754921389],[0.213613705288465,0.545980181296846,0.462858065656792,0.555883136314662,0.089979971244949],[0.229897462083034,0.606803420347241,0.441880181094265,0.645581402670746,0.238387780704064],[0.245824811756765,0.739869228579844,0.49811595358599,0.801230600000326,0.301658910303963],[0.0694967457544946,0.911879045679933,0.736263645750856,0.973268306969539,0.362172608061988],[0.131890106624229,0.766915045456103,0.471174055309928,0.742030952777324,0.665568016626695],[0.164829658497982,0.515216701198681,0.334570254727253,0.578557341817938,0.529033034945849],[0.531763581261236,0.718177853098515,0.338119112137043,0.645068858125427,0.496659923518642],[0.463214009841455,0.777367942019409,0.357599483272661,0.611463125655814,0.638873044291007],[0.742781450226171,0.879882415822816,0.440755998903745,0.787178952652566,0.873978634377844],[0.73856184642803,0.885602038677547,0.318801655583425,1,1],[0.334059480538155,1,0.765345298573847,0.967808669559975,0.987566110737814]],"text":[["row: Narphan<br>column: RADIO_PR<br>value: 0.116978","row: Narphan<br>column: TV_PR<br>value: 0.129468","row: Narphan<br>column: LLPHONE_PR<br>value: 0.220517","row: Narphan<br>column: MPHONE_PR<br>value: 0.384614","row: Narphan<br>column: COMPUTER_PR<br>value: 0.000000"],["row: Pangwaun<br>column: RADIO_PR<br>value: 0.119071","row: Pangwaun<br>column: TV_PR<br>value: 0.345093","row: Pangwaun<br>column: LLPHONE_PR<br>value: 0.146681","row: Pangwaun<br>column: MPHONE_PR<br>value: 0.689156","row: Pangwaun<br>column: COMPUTER_PR<br>value: 0.030692"],["row: Mongpan<br>column: RADIO_PR<br>value: 0.448113","row: Mongpan<br>column: TV_PR<br>value: 0.552260","row: Mongpan<br>column: LLPHONE_PR<br>value: 0.105216","row: Mongpan<br>column: MPHONE_PR<br>value: 0.574129","row: Mongpan<br>column: COMPUTER_PR<br>value: 0.484807"],["row: Nansang<br>column: RADIO_PR<br>value: 0.394727","row: Nansang<br>column: TV_PR<br>value: 0.646477","row: Nansang<br>column: LLPHONE_PR<br>value: 0.194771","row: Nansang<br>column: MPHONE_PR<br>value: 0.398016","row: Nansang<br>column: COMPUTER_PR<br>value: 0.307034"],["row: Kyaukme<br>column: RADIO_PR<br>value: 0.409739","row: Kyaukme<br>column: TV_PR<br>value: 0.667785","row: Kyaukme<br>column: LLPHONE_PR<br>value: 0.205940","row: Kyaukme<br>column: MPHONE_PR<br>value: 0.480784","row: Kyaukme<br>column: COMPUTER_PR<br>value: 0.310457"],["row: Kalaw<br>column: RADIO_PR<br>value: 0.560359","row: Kalaw<br>column: TV_PR<br>value: 0.682160","row: Kalaw<br>column: LLPHONE_PR<br>value: 0.219829","row: Kalaw<br>column: MPHONE_PR<br>value: 0.532718","row: Kalaw<br>column: COMPUTER_PR<br>value: 0.295697"],["row: Hseni<br>column: RADIO_PR<br>value: 0.509784","row: Hseni<br>column: TV_PR<br>value: 0.705850","row: Hseni<br>column: LLPHONE_PR<br>value: 0.235278","row: Hseni<br>column: MPHONE_PR<br>value: 0.565729","row: Hseni<br>column: COMPUTER_PR<br>value: 0.184217"],["row: Langkho<br>column: RADIO_PR<br>value: 0.445368","row: Langkho<br>column: TV_PR<br>value: 0.648475","row: Langkho<br>column: LLPHONE_PR<br>value: 0.400725","row: Langkho<br>column: MPHONE_PR<br>value: 0.525730","row: Langkho<br>column: COMPUTER_PR<br>value: 0.325728"],["row: Kunhing<br>column: RADIO_PR<br>value: 0.303006","row: Kunhing<br>column: TV_PR<br>value: 0.491472","row: Kunhing<br>column: LLPHONE_PR<br>value: 0.393414","row: Kunhing<br>column: MPHONE_PR<br>value: 0.434989","row: Kunhing<br>column: COMPUTER_PR<br>value: 0.255391"],["row: Laihka<br>column: RADIO_PR<br>value: 0.285461","row: Laihka<br>column: TV_PR<br>value: 0.653168","row: Laihka<br>column: LLPHONE_PR<br>value: 0.351130","row: Laihka<br>column: MPHONE_PR<br>value: 0.321903","row: Laihka<br>column: COMPUTER_PR<br>value: 0.274684"],["row: Monghsat<br>column: RADIO_PR<br>value: 0.186896","row: Monghsat<br>column: TV_PR<br>value: 0.473622","row: Monghsat<br>column: LLPHONE_PR<br>value: 0.177547","row: Monghsat<br>column: MPHONE_PR<br>value: 0.219018","row: Monghsat<br>column: COMPUTER_PR<br>value: 0.219238"],["row: Loilen<br>column: RADIO_PR<br>value: 0.262949","row: Loilen<br>column: TV_PR<br>value: 0.600934","row: Loilen<br>column: LLPHONE_PR<br>value: 0.105589","row: Loilen<br>column: MPHONE_PR<br>value: 0.309845","row: Loilen<br>column: COMPUTER_PR<br>value: 0.322444"],["row: Mongnai<br>column: RADIO_PR<br>value: 0.348385","row: Mongnai<br>column: TV_PR<br>value: 0.550413","row: Mongnai<br>column: LLPHONE_PR<br>value: 0.013213","row: Mongnai<br>column: MPHONE_PR<br>value: 0.330601","row: Mongnai<br>column: COMPUTER_PR<br>value: 0.280505"],["row: Mongton<br>column: RADIO_PR<br>value: 0.372116","row: Mongton<br>column: TV_PR<br>value: 0.374969","row: Mongton<br>column: LLPHONE_PR<br>value: 0.125061","row: Mongton<br>column: MPHONE_PR<br>value: 0.363503","row: Mongton<br>column: COMPUTER_PR<br>value: 0.208287"],["row: Hsipaw<br>column: RADIO_PR<br>value: 0.382049","row: Hsipaw<br>column: TV_PR<br>value: 0.495293","row: Hsipaw<br>column: LLPHONE_PR<br>value: 0.165281","row: Hsipaw<br>column: MPHONE_PR<br>value: 0.332852","row: Hsipaw<br>column: COMPUTER_PR<br>value: 0.126911"],["row: Hopong<br>column: RADIO_PR<br>value: 0.398091","row: Hopong<br>column: TV_PR<br>value: 0.467734","row: Hopong<br>column: LLPHONE_PR<br>value: 0.165252","row: Hopong<br>column: MPHONE_PR<br>value: 0.248272","row: Hopong<br>column: COMPUTER_PR<br>value: 0.125663"],["row: Monghsu<br>column: RADIO_PR<br>value: 0.463675","row: Monghsu<br>column: TV_PR<br>value: 0.480273","row: Monghsu<br>column: LLPHONE_PR<br>value: 0.049518","row: Monghsu<br>column: MPHONE_PR<br>value: 0.278454","row: Monghsu<br>column: COMPUTER_PR<br>value: 0.102240"],["row: Pinlaung<br>column: RADIO_PR<br>value: 0.454391","row: Pinlaung<br>column: TV_PR<br>value: 0.585944","row: Pinlaung<br>column: LLPHONE_PR<br>value: 0.144173","row: Pinlaung<br>column: MPHONE_PR<br>value: 0.304818","row: Pinlaung<br>column: COMPUTER_PR<br>value: 0.117641"],["row: Mongmit<br>column: RADIO_PR<br>value: 0.572059","row: Mongmit<br>column: TV_PR<br>value: 0.603029","row: Mongmit<br>column: LLPHONE_PR<br>value: 0.182009","row: Mongmit<br>column: MPHONE_PR<br>value: 0.320846","row: Mongmit<br>column: COMPUTER_PR<br>value: 0.099653"],["row: Pekon<br>column: RADIO_PR<br>value: 0.642025","row: Pekon<br>column: TV_PR<br>value: 0.577864","row: Pekon<br>column: LLPHONE_PR<br>value: 0.207339","row: Pekon<br>column: MPHONE_PR<br>value: 0.255258","row: Pekon<br>column: COMPUTER_PR<br>value: 0.176074"],["row: Kutkai<br>column: RADIO_PR<br>value: 0.469637","row: Kutkai<br>column: TV_PR<br>value: 0.492753","row: Kutkai<br>column: LLPHONE_PR<br>value: 0.336404","row: Kutkai<br>column: MPHONE_PR<br>value: 0.326463","row: Kutkai<br>column: COMPUTER_PR<br>value: 0.199643"],["row: Namtu<br>column: RADIO_PR<br>value: 0.472438","row: Namtu<br>column: TV_PR<br>value: 0.708307","row: Namtu<br>column: LLPHONE_PR<br>value: 0.181529","row: Namtu<br>column: MPHONE_PR<br>value: 0.098196","row: Namtu<br>column: COMPUTER_PR<br>value: 0.186837"],["row: Hsihseng<br>column: RADIO_PR<br>value: 0.487022","row: Hsihseng<br>column: TV_PR<br>value: 0.536185","row: Hsihseng<br>column: LLPHONE_PR<br>value: 0.162369","row: Hsihseng<br>column: MPHONE_PR<br>value: 0.155475","row: Hsihseng<br>column: COMPUTER_PR<br>value: 0.074496"],["row: Kyethi<br>column: RADIO_PR<br>value: 0.534163","row: Kyethi<br>column: TV_PR<br>value: 0.455761","row: Kyethi<br>column: LLPHONE_PR<br>value: 0.109549","row: Kyethi<br>column: MPHONE_PR<br>value: 0.076156","row: Kyethi<br>column: COMPUTER_PR<br>value: 0.106926"],["row: Tangyan<br>column: RADIO_PR<br>value: 0.437590","row: Tangyan<br>column: TV_PR<br>value: 0.464204","row: Tangyan<br>column: LLPHONE_PR<br>value: 0.110875","row: Tangyan<br>column: MPHONE_PR<br>value: 0.098440","row: Tangyan<br>column: COMPUTER_PR<br>value: 0.098598"],["row: Namhsan<br>column: RADIO_PR<br>value: 0.452637","row: Namhsan<br>column: TV_PR<br>value: 0.377798","row: Namhsan<br>column: LLPHONE_PR<br>value: 0.113655","row: Namhsan<br>column: MPHONE_PR<br>value: 0.087770","row: Namhsan<br>column: COMPUTER_PR<br>value: 0.064069"],["row: Mongyai<br>column: RADIO_PR<br>value: 0.318012","row: Mongyai<br>column: TV_PR<br>value: 0.341693","row: Mongyai<br>column: LLPHONE_PR<br>value: 0.098223","row: Mongyai<br>column: MPHONE_PR<br>value: 0.141213","row: Mongyai<br>column: COMPUTER_PR<br>value: 0.068454"],["row: Mongping<br>column: RADIO_PR<br>value: 0.303906","row: Mongping<br>column: TV_PR<br>value: 0.295735","row: Mongping<br>column: LLPHONE_PR<br>value: 0.122161","row: Mongping<br>column: MPHONE_PR<br>value: 0.220797","row: Mongping<br>column: COMPUTER_PR<br>value: 0.093358"],["row: Mongkhet<br>column: RADIO_PR<br>value: 0.125839","row: Mongkhet<br>column: TV_PR<br>value: 0.271666","row: Mongkhet<br>column: LLPHONE_PR<br>value: 0.146913","row: Mongkhet<br>column: MPHONE_PR<br>value: 0.111904","row: Mongkhet<br>column: COMPUTER_PR<br>value: 0.040120"],["row: Mawkmai<br>column: RADIO_PR<br>value: 0.164818","row: Mawkmai<br>column: TV_PR<br>value: 0.163071","row: Mawkmai<br>column: LLPHONE_PR<br>value: 0.000000","row: Mawkmai<br>column: MPHONE_PR<br>value: 0.000000","row: Mawkmai<br>column: COMPUTER_PR<br>value: 0.050563"],["row: Mongkaing<br>column: RADIO_PR<br>value: 0.292778","row: Mongkaing<br>column: TV_PR<br>value: 0.000000","row: Mongkaing<br>column: LLPHONE_PR<br>value: 0.018021","row: Mongkaing<br>column: MPHONE_PR<br>value: 0.020515","row: Mongkaing<br>column: COMPUTER_PR<br>value: 0.057342"],["row: Manton<br>column: RADIO_PR<br>value: 0.373857","row: Manton<br>column: TV_PR<br>value: 0.099529","row: Manton<br>column: LLPHONE_PR<br>value: 0.029600","row: Manton<br>column: MPHONE_PR<br>value: 0.026290","row: Manton<br>column: COMPUTER_PR<br>value: 0.004112"],["row: Matman<br>column: RADIO_PR<br>value: 0.481304","row: Matman<br>column: TV_PR<br>value: 0.142298","row: Matman<br>column: LLPHONE_PR<br>value: 0.031665","row: Matman<br>column: MPHONE_PR<br>value: 0.012573","row: Matman<br>column: COMPUTER_PR<br>value: 0.030854"],["row: Ywangan<br>column: RADIO_PR<br>value: 1.000000","row: Ywangan<br>column: TV_PR<br>value: 0.198958","row: Ywangan<br>column: LLPHONE_PR<br>value: 0.051234","row: Ywangan<br>column: MPHONE_PR<br>value: 0.119978","row: Ywangan<br>column: COMPUTER_PR<br>value: 0.012755"],["row: Pindaya<br>column: RADIO_PR<br>value: 0.855315","row: Pindaya<br>column: TV_PR<br>value: 0.535580","row: Pindaya<br>column: LLPHONE_PR<br>value: 0.095440","row: Pindaya<br>column: MPHONE_PR<br>value: 0.180215","row: Pindaya<br>column: COMPUTER_PR<br>value: 0.107760"],["row: Lawksawk<br>column: RADIO_PR<br>value: 0.789802","row: Lawksawk<br>column: TV_PR<br>value: 0.707464","row: Lawksawk<br>column: LLPHONE_PR<br>value: 0.160787","row: Lawksawk<br>column: MPHONE_PR<br>value: 0.406502","row: Lawksawk<br>column: COMPUTER_PR<br>value: 0.207451"],["row: Nyaungshwe<br>column: RADIO_PR<br>value: 0.653022","row: Nyaungshwe<br>column: TV_PR<br>value: 0.603518","row: Nyaungshwe<br>column: LLPHONE_PR<br>value: 0.090757","row: Nyaungshwe<br>column: MPHONE_PR<br>value: 0.450321","row: Nyaungshwe<br>column: COMPUTER_PR<br>value: 0.139287"],["row: Nawnghkio<br>column: RADIO_PR<br>value: 0.708317","row: Nawnghkio<br>column: TV_PR<br>value: 0.594514","row: Nawnghkio<br>column: LLPHONE_PR<br>value: 0.199599","row: Nawnghkio<br>column: MPHONE_PR<br>value: 0.410009","row: Nawnghkio<br>column: COMPUTER_PR<br>value: 0.140105"],["row: Monghpyak<br>column: RADIO_PR<br>value: 0.828482","row: Monghpyak<br>column: TV_PR<br>value: 0.533096","row: Monghpyak<br>column: LLPHONE_PR<br>value: 0.222253","row: Monghpyak<br>column: MPHONE_PR<br>value: 0.494146","row: Monghpyak<br>column: COMPUTER_PR<br>value: 0.245321"],["row: Mabein<br>column: RADIO_PR<br>value: 0.924415","row: Mabein<br>column: TV_PR<br>value: 0.815707","row: Mabein<br>column: LLPHONE_PR<br>value: 0.391551","row: Mabein<br>column: MPHONE_PR<br>value: 0.509562","row: Mabein<br>column: COMPUTER_PR<br>value: 0.147800"],["row: Mongyawng<br>column: RADIO_PR<br>value: 0.554582","row: Mongyawng<br>column: TV_PR<br>value: 0.857899","row: Mongyawng<br>column: LLPHONE_PR<br>value: 0.667008","row: Mongyawng<br>column: MPHONE_PR<br>value: 0.411502","row: Mongyawng<br>column: COMPUTER_PR<br>value: 0.135187"],["row: Kunlong<br>column: RADIO_PR<br>value: 0.217634","row: Kunlong<br>column: TV_PR<br>value: 0.652098","row: Kunlong<br>column: LLPHONE_PR<br>value: 1.000000","row: Kunlong<br>column: MPHONE_PR<br>value: 0.287788","row: Kunlong<br>column: COMPUTER_PR<br>value: 0.196438"],["row: Konkyan<br>column: RADIO_PR<br>value: 0.000000","row: Konkyan<br>column: TV_PR<br>value: 0.235436","row: Konkyan<br>column: LLPHONE_PR<br>value: 0.932101","row: Konkyan<br>column: MPHONE_PR<br>value: 0.401016","row: Konkyan<br>column: COMPUTER_PR<br>value: 0.084422"],["row: Mongyang<br>column: RADIO_PR<br>value: 0.109375","row: Mongyang<br>column: TV_PR<br>value: 0.623942","row: Mongyang<br>column: LLPHONE_PR<br>value: 0.622051","row: Mongyang<br>column: MPHONE_PR<br>value: 0.618873","row: Mongyang<br>column: COMPUTER_PR<br>value: 0.107370"],["row: Mongmao<br>column: RADIO_PR<br>value: 0.213614","row: Mongmao<br>column: TV_PR<br>value: 0.545980","row: Mongmao<br>column: LLPHONE_PR<br>value: 0.462858","row: Mongmao<br>column: MPHONE_PR<br>value: 0.555883","row: Mongmao<br>column: COMPUTER_PR<br>value: 0.089980"],["row: Hopang<br>column: RADIO_PR<br>value: 0.229897","row: Hopang<br>column: TV_PR<br>value: 0.606803","row: Hopang<br>column: LLPHONE_PR<br>value: 0.441880","row: Hopang<br>column: MPHONE_PR<br>value: 0.645581","row: Hopang<br>column: COMPUTER_PR<br>value: 0.238388"],["row: Namhkan<br>column: RADIO_PR<br>value: 0.245825","row: Namhkan<br>column: TV_PR<br>value: 0.739869","row: Namhkan<br>column: LLPHONE_PR<br>value: 0.498116","row: Namhkan<br>column: MPHONE_PR<br>value: 0.801231","row: Namhkan<br>column: COMPUTER_PR<br>value: 0.301659"],["row: Laukkaing<br>column: RADIO_PR<br>value: 0.069497","row: Laukkaing<br>column: TV_PR<br>value: 0.911879","row: Laukkaing<br>column: LLPHONE_PR<br>value: 0.736264","row: Laukkaing<br>column: MPHONE_PR<br>value: 0.973268","row: Laukkaing<br>column: COMPUTER_PR<br>value: 0.362173"],["row: Mongla<br>column: RADIO_PR<br>value: 0.131890","row: Mongla<br>column: TV_PR<br>value: 0.766915","row: Mongla<br>column: LLPHONE_PR<br>value: 0.471174","row: Mongla<br>column: MPHONE_PR<br>value: 0.742031","row: Mongla<br>column: COMPUTER_PR<br>value: 0.665568"],["row: Pangsang<br>column: RADIO_PR<br>value: 0.164830","row: Pangsang<br>column: TV_PR<br>value: 0.515217","row: Pangsang<br>column: LLPHONE_PR<br>value: 0.334570","row: Pangsang<br>column: MPHONE_PR<br>value: 0.578557","row: Pangsang<br>column: COMPUTER_PR<br>value: 0.529033"],["row: Kengtung<br>column: RADIO_PR<br>value: 0.531764","row: Kengtung<br>column: TV_PR<br>value: 0.718178","row: Kengtung<br>column: LLPHONE_PR<br>value: 0.338119","row: Kengtung<br>column: MPHONE_PR<br>value: 0.645069","row: Kengtung<br>column: COMPUTER_PR<br>value: 0.496660"],["row: Lashio<br>column: RADIO_PR<br>value: 0.463214","row: Lashio<br>column: TV_PR<br>value: 0.777368","row: Lashio<br>column: LLPHONE_PR<br>value: 0.357599","row: Lashio<br>column: MPHONE_PR<br>value: 0.611463","row: Lashio<br>column: COMPUTER_PR<br>value: 0.638873"],["row: Taunggyi<br>column: RADIO_PR<br>value: 0.742781","row: Taunggyi<br>column: TV_PR<br>value: 0.879882","row: Taunggyi<br>column: LLPHONE_PR<br>value: 0.440756","row: Taunggyi<br>column: MPHONE_PR<br>value: 0.787179","row: Taunggyi<br>column: COMPUTER_PR<br>value: 0.873979"],["row: Tachileik<br>column: RADIO_PR<br>value: 0.738562","row: Tachileik<br>column: TV_PR<br>value: 0.885602","row: Tachileik<br>column: LLPHONE_PR<br>value: 0.318802","row: Tachileik<br>column: MPHONE_PR<br>value: 1.000000","row: Tachileik<br>column: COMPUTER_PR<br>value: 1.000000"],["row: Muse<br>column: RADIO_PR<br>value: 0.334059","row: Muse<br>column: TV_PR<br>value: 1.000000","row: Muse<br>column: LLPHONE_PR<br>value: 0.765345","row: Muse<br>column: MPHONE_PR<br>value: 0.967809","row: Muse<br>column: COMPUTER_PR<br>value: 0.987566"]],"colorscale":[[0,"#F7FBFF"],[0.00411244014094081,"#F6FAFE"],[0.0125730639984617,"#F4F9FE"],[0.0127547808735244,"#F4F9FE"],[0.0132133117776572,"#F4F9FE"],[0.0180206673088627,"#F3F8FD"],[0.0205146898168363,"#F3F8FD"],[0.0262900696870936,"#F1F7FD"],[0.0295999006447705,"#F0F6FC"],[0.0306923094501804,"#F0F6FC"],[0.030854020566576,"#F0F6FC"],[0.0316650514783228,"#F0F6FC"],[0.0401203752296987,"#EFF5FC"],[0.0495176036018547,"#ECF4FB"],[0.050562923174398,"#ECF4FB"],[0.0512340297746273,"#ECF4FB"],[0.0573419602932674,"#EBF3FB"],[0.0640685752289788,"#EAF2FA"],[0.0684535703034676,"#E9F2FA"],[0.0694967457544946,"#E8F1FA"],[0.0744963103413769,"#E8F1FA"],[0.076155538926932,"#E8F1FA"],[0.0844217926569466,"#E5EFF9"],[0.0877697005887191,"#E5EFF9"],[0.089979971244949,"#E4EFF9"],[0.0907572048090615,"#E4EFF9"],[0.0933576373559664,"#E4EEF8"],[0.0954404378234391,"#E4EEF8"],[0.0981964513254065,"#E3EEF8"],[0.0982231808523387,"#E3EEF8"],[0.0984403758574017,"#E3EEF8"],[0.0985978869506665,"#E3EEF8"],[0.0995290076456667,"#E3EEF8"],[0.0996532209950878,"#E3EEF8"],[0.102240256271893,"#E2EDF8"],[0.105215687600081,"#E1EDF8"],[0.105588681661233,"#E1EDF8"],[0.106925771850934,"#E1EDF8"],[0.107369754921389,"#E1EDF8"],[0.107759971325446,"#E1EDF8"],[0.109374929636258,"#E1ECF7"],[0.109548536966956,"#E1ECF7"],[0.110874787457869,"#E1ECF7"],[0.111904436916593,"#E0ECF7"],[0.113655496814988,"#E0ECF7"],[0.116978282802357,"#DFEBF7"],[0.117641144346071,"#DFEBF7"],[0.119071471357574,"#DFEBF7"],[0.119978472699803,"#DEEBF7"],[0.122161422028907,"#DEEBF7"],[0.125060804628556,"#DDEAF6"],[0.125663315216338,"#DDEAF6"],[0.125839377932289,"#DDEAF6"],[0.126911060390332,"#DDEAF6"],[0.129468002916701,"#DDEAF6"],[0.131890106624229,"#DCE9F6"],[0.135186676818821,"#DCE9F6"],[0.139286867591449,"#DAE8F5"],[0.140105344905725,"#DAE8F5"],[0.141213400161106,"#DAE8F5"],[0.142297506536797,"#DAE8F5"],[0.144172529109699,"#DAE8F5"],[0.146680718622619,"#DAE8F5"],[0.146912934029539,"#DAE8F5"],[0.147799896773415,"#D9E7F5"],[0.155474604691974,"#D7E6F4"],[0.160787084615829,"#D7E6F4"],[0.1623692849231,"#D7E6F4"],[0.163070599120654,"#D6E5F4"],[0.1648181386917,"#D6E5F4"],[0.164829658497982,"#D6E5F4"],[0.165251926083381,"#D6E5F4"],[0.165281214256075,"#D6E5F4"],[0.176074057695434,"#D4E4F3"],[0.177547109767516,"#D4E4F3"],[0.180215108968874,"#D3E3F3"],[0.181528926899385,"#D3E3F3"],[0.18200927177224,"#D3E3F3"],[0.184217142327857,"#D2E3F3"],[0.186836974506664,"#D1E2F2"],[0.186896495593756,"#D1E2F2"],[0.19477119473094,"#D0E1F2"],[0.196437751553831,"#D0E1F2"],[0.198958169425964,"#CFE1F2"],[0.199599204123416,"#CFE1F2"],[0.199643002704722,"#CFE1F2"],[0.205939628052291,"#CEE0F1"],[0.207338745441228,"#CEE0F1"],[0.207451038928221,"#CEE0F1"],[0.208286637250586,"#CEE0F1"],[0.213613705288465,"#CDDFF1"],[0.217633980358742,"#CCDFF1"],[0.219017992119371,"#CBDEF0"],[0.219238039033904,"#CBDEF0"],[0.219828980006303,"#CBDEF0"],[0.22051676674467,"#CBDEF0"],[0.220797228887767,"#CBDEF0"],[0.222253044854804,"#CBDEF0"],[0.229897462083034,"#C9DDF0"],[0.235277615529988,"#C8DCEF"],[0.23543567048956,"#C8DCEF"],[0.238387780704064,"#C8DCEF"],[0.245321426698747,"#C6DBEF"],[0.245824811756765,"#C6DBEF"],[0.248272216763125,"#C6DBEF"],[0.255258118966462,"#C4DAEE"],[0.25539117902522,"#C4DAEE"],[0.262948885760044,"#C1D9ED"],[0.271665869542777,"#BFD8EC"],[0.274683722730117,"#BED7EC"],[0.278454014117194,"#BCD7EB"],[0.280505026346822,"#BBD6EB"],[0.285460565623123,"#BAD6EA"],[0.287788284692333,"#BAD6EA"],[0.292778031055123,"#B8D4EA"],[0.295696949383387,"#B7D4EA"],[0.295734660011411,"#B7D4EA"],[0.301658910303963,"#B5D3E9"],[0.303006324046602,"#B5D3E9"],[0.303905626461184,"#B5D3E9"],[0.304817572789695,"#B4D3E8"],[0.307033923303471,"#B3D3E8"],[0.309844721395711,"#B2D2E8"],[0.310457200410047,"#B2D2E8"],[0.318011520183019,"#B0D1E7"],[0.318801655583425,"#B0D1E7"],[0.320846100415741,"#AFD1E6"],[0.321903369400093,"#AFD1E6"],[0.32244406175046,"#AFD1E6"],[0.325727661622098,"#ADD0E6"],[0.326463285635126,"#ADD0E6"],[0.330600515260223,"#ACD0E6"],[0.332852029421132,"#ABCFE5"],[0.334059480538155,"#ABCFE5"],[0.334570254727253,"#ABCFE5"],[0.336403906718691,"#AACFE5"],[0.338119112137043,"#AACFE5"],[0.341693315696117,"#A8CEE4"],[0.345093075355747,"#A7CEE4"],[0.348384569046878,"#A6CDE3"],[0.351129977629807,"#A5CDE3"],[0.357599483272661,"#A3CCE3"],[0.362172608061988,"#A2CBE2"],[0.363502861923713,"#A1CBE2"],[0.372116151704965,"#9ECAE1"],[0.373857483778625,"#9ECAE1"],[0.374968621142249,"#9DC9E0"],[0.377797614010294,"#9CC9E0"],[0.382049084643175,"#9BC8E0"],[0.384613999201764,"#9AC7E0"],[0.391551407290058,"#97C6DF"],[0.393413634291369,"#96C6DF"],[0.394726524387261,"#96C5DF"],[0.398016081357507,"#94C5DF"],[0.398090757930385,"#94C4DE"],[0.400725374496756,"#93C4DE"],[0.401015569024513,"#93C4DE"],[0.406501651460893,"#91C2DE"],[0.409738609943887,"#90C2DE"],[0.410008712407441,"#8FC1DD"],[0.411501854757207,"#8FC1DD"],[0.434988683676573,"#85BCDB"],[0.437590296823069,"#84BBDB"],[0.440755998903745,"#83BBDB"],[0.441880181094265,"#82BADB"],[0.445367826108455,"#81B9DA"],[0.448112949126574,"#80B9DA"],[0.450320578796944,"#7FB8DA"],[0.452636675796627,"#7EB8DA"],[0.454391396013767,"#7DB8D9"],[0.455760556744961,"#7DB8D9"],[0.462858065656792,"#7AB6D9"],[0.463214009841455,"#7AB6D9"],[0.463675071811737,"#7AB6D9"],[0.464204461437082,"#79B6D9"],[0.467734047728609,"#78B5D8"],[0.469637019864802,"#77B4D8"],[0.471174055309928,"#77B4D8"],[0.472437575771037,"#76B4D8"],[0.473621686944799,"#75B3D8"],[0.480273489712105,"#73B2D7"],[0.480784254803859,"#72B1D7"],[0.481303842133627,"#72B1D7"],[0.484807392774743,"#71B1D7"],[0.487021825059319,"#70B1D7"],[0.491472190827421,"#6EB0D6"],[0.49275331606036,"#6EAFD6"],[0.49414581896158,"#6DAFD6"],[0.495293243751349,"#6CAFD6"],[0.496659923518642,"#6CAED6"],[0.49811595358599,"#6BAED6"],[0.509561826896415,"#67ABD4"],[0.509783518401058,"#67ABD4"],[0.515216701198681,"#66AAD4"],[0.525729584351687,"#62A8D2"],[0.529033034945849,"#61A7D2"],[0.531763581261236,"#60A6D1"],[0.532717729177229,"#60A6D1"],[0.533096150331035,"#60A6D1"],[0.534163143993451,"#60A6D1"],[0.535580295292678,"#5FA5D1"],[0.536184545909484,"#5FA5D1"],[0.545980181296846,"#5CA3D0"],[0.550413307218695,"#5AA3CF"],[0.552259924114501,"#59A2CF"],[0.554582082266574,"#59A2CF"],[0.555883136314662,"#58A1CE"],[0.560359050607694,"#57A0CE"],[0.56572881621916,"#559FCD"],[0.572059422162612,"#539DCC"],[0.574129014267529,"#529DCC"],[0.577864390148239,"#519CCC"],[0.578557341817938,"#509BCB"],[0.585943534900779,"#4F9BCB"],[0.59451431070864,"#4B98C9"],[0.600934420551276,"#4A97C9"],[0.603028579702235,"#4896C8"],[0.603518277436298,"#4896C8"],[0.606803420347241,"#4795C8"],[0.611463125655814,"#4694C7"],[0.618872994218249,"#4393C6"],[0.622050632093838,"#4292C6"],[0.623942179795541,"#4292C6"],[0.638873044291007,"#3E8EC4"],[0.64202483918062,"#3D8DC3"],[0.645068858125427,"#3D8DC3"],[0.645581402670746,"#3C8CC3"],[0.646477338669936,"#3C8CC3"],[0.648475186589583,"#3C8CC3"],[0.65209843040106,"#3B8BC2"],[0.653022272989167,"#3A8AC1"],[0.653167734189465,"#3A8AC1"],[0.665568016626695,"#3787C0"],[0.667007964836916,"#3787C0"],[0.667785462581825,"#3686C0"],[0.682160053201915,"#3282BE"],[0.689156011292375,"#3080BD"],[0.705849662481414,"#2C7CBB"],[0.707463556363309,"#2C7CBB"],[0.708306581316802,"#2B7BBA"],[0.708316665856044,"#2B7BBA"],[0.718177853098515,"#2979B9"],[0.736263645750856,"#2474B6"],[0.73856184642803,"#2474B6"],[0.739869228579844,"#2373B6"],[0.742030952777324,"#2373B6"],[0.742781450226171,"#2373B6"],[0.765345298573847,"#1E6DB2"],[0.766915045456103,"#1D6CB1"],[0.777367942019409,"#1B6AAF"],[0.787178952652566,"#1967AD"],[0.789802194306684,"#1967AD"],[0.801230600000326,"#1664AB"],[0.815706630141315,"#1360A7"],[0.828481634580948,"#115DA5"],[0.855314778071882,"#0C56A0"],[0.857899290321775,"#0B559F"],[0.873978634377844,"#08519C"],[0.879882415822816,"#08509A"],[0.885602038677547,"#084E97"],[0.911879045679933,"#08468D"],[0.924414874434519,"#084388"],[0.932101354458042,"#084185"],[0.967808669559975,"#083877"],[0.973268306969539,"#083775"],[0.987566110737814,"#08336F"],[1,"#08306B"]],"type":"heatmap","showscale":false,"autocolorscale":false,"showlegend":false,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1],"y":[1],"name":"99_306696ee48c8158f9b168eae66068fb5","type":"scatter","mode":"markers","opacity":0,"hoverinfo":"skip","showlegend":false,"marker":{"color":[0,1],"colorscale":[[0,"#F7FBFF"],[0.00334448160535117,"#F6FAFE"],[0.00668896321070234,"#F5F9FE"],[0.0100334448160535,"#F4F9FE"],[0.0133779264214047,"#F4F9FE"],[0.0167224080267559,"#F3F8FD"],[0.020066889632107,"#F3F8FD"],[0.0234113712374582,"#F2F7FD"],[0.0267558528428094,"#F1F7FD"],[0.0301003344481605,"#F0F6FC"],[0.0334448160535117,"#EFF6FC"],[0.0367892976588629,"#EFF6FC"],[0.040133779264214,"#EFF5FC"],[0.0434782608695652,"#EEF5FC"],[0.0468227424749164,"#EDF4FB"],[0.0501672240802676,"#ECF4FB"],[0.0535117056856187,"#ECF3FB"],[0.0568561872909699,"#ECF3FB"],[0.0602006688963211,"#EBF3FB"],[0.0635451505016722,"#EAF2FA"],[0.0668896321070234,"#E9F2FA"],[0.0702341137123746,"#E8F1FA"],[0.0735785953177257,"#E8F1FA"],[0.0769230769230769,"#E7F0F9"],[0.0802675585284281,"#E7F0F9"],[0.0836120401337793,"#E6F0F9"],[0.0869565217391304,"#E5EFF9"],[0.0903010033444816,"#E4EFF9"],[0.0936454849498328,"#E4EEF8"],[0.096989966555184,"#E3EEF8"],[0.100334448160535,"#E2EDF8"],[0.103678929765886,"#E2EDF8"],[0.107023411371237,"#E1EDF8"],[0.110367892976589,"#E1ECF7"],[0.11371237458194,"#E0ECF7"],[0.117056856187291,"#DFEBF7"],[0.120401337792642,"#DEEBF7"],[0.123745819397993,"#DDEAF6"],[0.127090301003344,"#DDEAF6"],[0.130434782608696,"#DDEAF6"],[0.133779264214047,"#DCE9F6"],[0.137123745819398,"#DBE9F6"],[0.140468227424749,"#DAE8F5"],[0.1438127090301,"#DAE8F5"],[0.147157190635451,"#D9E7F5"],[0.150501672240803,"#D9E7F5"],[0.153846153846154,"#D8E7F5"],[0.157190635451505,"#D7E6F4"],[0.160535117056856,"#D7E6F4"],[0.163879598662207,"#D6E5F4"],[0.167224080267559,"#D5E5F4"],[0.17056856187291,"#D5E5F4"],[0.173913043478261,"#D4E4F3"],[0.177257525083612,"#D4E4F3"],[0.180602006688963,"#D3E3F3"],[0.183946488294314,"#D2E3F3"],[0.187290969899666,"#D1E2F2"],[0.190635451505017,"#D1E2F2"],[0.193979933110368,"#D1E2F2"],[0.197324414715719,"#D0E1F2"],[0.20066889632107,"#CFE1F2"],[0.204013377926421,"#CEE0F1"],[0.207357859531773,"#CEE0F1"],[0.210702341137124,"#CDDFF1"],[0.214046822742475,"#CCDFF1"],[0.217391304347826,"#CCDFF1"],[0.220735785953177,"#CBDEF0"],[0.224080267558528,"#CBDEF0"],[0.22742474916388,"#CADDF0"],[0.230769230769231,"#C9DDF0"],[0.234113712374582,"#C8DCEF"],[0.237458193979933,"#C8DCEF"],[0.240802675585284,"#C8DCEF"],[0.244147157190635,"#C7DBEF"],[0.247491638795987,"#C6DBEF"],[0.250836120401338,"#C5DAEE"],[0.254180602006689,"#C4DAEE"],[0.25752508361204,"#C3D9EE"],[0.260869565217391,"#C2D9ED"],[0.264214046822742,"#C1D9ED"],[0.267558528428094,"#C0D8ED"],[0.270903010033445,"#BFD8EC"],[0.274247491638796,"#BED7EC"],[0.277591973244147,"#BCD7EB"],[0.280936454849498,"#BBD6EB"],[0.284280936454849,"#BBD6EB"],[0.287625418060201,"#BAD6EA"],[0.290969899665552,"#B9D5EA"],[0.294314381270903,"#B7D4EA"],[0.297658862876254,"#B6D4E9"],[0.301003344481605,"#B5D3E9"],[0.304347826086957,"#B4D3E8"],[0.307692307692308,"#B3D3E8"],[0.311036789297659,"#B2D2E8"],[0.31438127090301,"#B1D2E7"],[0.317725752508361,"#B0D1E7"],[0.321070234113712,"#AFD1E6"],[0.324414715719064,"#AED0E6"],[0.327759197324415,"#ACD0E6"],[0.331103678929766,"#ACD0E6"],[0.334448160535117,"#ABCFE5"],[0.337792642140468,"#AACFE5"],[0.341137123745819,"#A8CEE4"],[0.344481605351171,"#A7CEE4"],[0.347826086956522,"#A6CDE3"],[0.351170568561873,"#A5CDE3"],[0.354515050167224,"#A4CDE3"],[0.357859531772575,"#A3CCE3"],[0.361204013377926,"#A2CBE2"],[0.364548494983278,"#A1CBE2"],[0.367892976588629,"#A0CAE1"],[0.37123745819398,"#9FCAE1"],[0.374581939799331,"#9DC9E0"],[0.377926421404682,"#9CC9E0"],[0.381270903010033,"#9BC8E0"],[0.384615384615385,"#9AC7E0"],[0.387959866220736,"#98C7DF"],[0.391304347826087,"#97C6DF"],[0.394648829431438,"#96C5DF"],[0.397993311036789,"#94C5DF"],[0.40133779264214,"#93C4DE"],[0.404682274247492,"#92C3DE"],[0.408026755852843,"#90C2DE"],[0.411371237458194,"#8FC1DD"],[0.414715719063545,"#8DC0DD"],[0.418060200668896,"#8CC0DD"],[0.421404682274247,"#8BC0DD"],[0.424749163879599,"#89BFDC"],[0.42809364548495,"#88BEDC"],[0.431438127090301,"#87BDDC"],[0.434782608695652,"#85BCDB"],[0.438127090301003,"#84BBDB"],[0.441471571906354,"#82BADB"],[0.444816053511706,"#81BADB"],[0.448160535117057,"#80B9DA"],[0.451505016722408,"#7FB8DA"],[0.454849498327759,"#7DB8D9"],[0.45819397993311,"#7BB7D9"],[0.461538461538462,"#7AB6D9"],[0.464882943143813,"#79B5D8"],[0.468227424749164,"#78B5D8"],[0.471571906354515,"#77B4D8"],[0.474916387959866,"#75B3D8"],[0.478260869565217,"#73B2D7"],[0.481605351170569,"#72B1D7"],[0.48494983277592,"#71B1D7"],[0.488294314381271,"#6FB0D6"],[0.491638795986622,"#6EB0D6"],[0.494983277591973,"#6DAFD6"],[0.498327759197324,"#6BAED6"],[0.501672240802676,"#6AADD5"],[0.505016722408027,"#69ACD5"],[0.508361204013378,"#68ABD4"],[0.511705685618729,"#67ABD4"],[0.51505016722408,"#66AAD4"],[0.518394648829431,"#65AAD3"],[0.521739130434783,"#63A9D3"],[0.525083612040134,"#62A8D2"],[0.528428093645485,"#61A7D2"],[0.531772575250836,"#60A6D1"],[0.535117056856187,"#5FA6D1"],[0.538461538461538,"#5EA5D1"],[0.54180602006689,"#5DA4D0"],[0.545150501672241,"#5CA3D0"],[0.548494983277592,"#5AA3CF"],[0.551839464882943,"#59A2CF"],[0.555183946488294,"#58A1CE"],[0.558528428093645,"#58A1CE"],[0.561872909698997,"#56A0CE"],[0.565217391304348,"#559FCD"],[0.568561872909699,"#549ECD"],[0.57190635451505,"#539DCC"],[0.575250836120401,"#529CCC"],[0.578595317725752,"#509BCB"],[0.581939799331104,"#509BCB"],[0.585284280936455,"#4F9BCB"],[0.588628762541806,"#4E9ACA"],[0.591973244147157,"#4C99CA"],[0.595317725752508,"#4B98C9"],[0.598662207357859,"#4A97C9"],[0.602006688963211,"#4996C8"],[0.605351170568562,"#4896C8"],[0.608695652173913,"#4795C8"],[0.612040133779264,"#4694C7"],[0.615384615384615,"#4594C7"],[0.618729096989967,"#4393C6"],[0.622073578595318,"#4292C6"],[0.625418060200669,"#4292C6"],[0.62876254180602,"#4191C5"],[0.632107023411371,"#4090C5"],[0.635451505016722,"#3F8FC4"],[0.638795986622074,"#3E8EC4"],[0.642140468227425,"#3D8DC3"],[0.645484949832776,"#3C8CC3"],[0.648829431438127,"#3C8CC3"],[0.652173913043478,"#3B8BC2"],[0.655518394648829,"#3A8AC1"],[0.658862876254181,"#3989C1"],[0.662207357859532,"#3888C0"],[0.665551839464883,"#3787C0"],[0.668896321070234,"#3686BF"],[0.672240802675585,"#3585BF"],[0.675585284280936,"#3484BF"],[0.678929765886288,"#3383BE"],[0.682274247491639,"#3282BE"],[0.68561872909699,"#3181BD"],[0.688963210702341,"#3080BD"],[0.692307692307692,"#2F7FBC"],[0.695652173913043,"#2F7FBC"],[0.698996655518395,"#2E7EBC"],[0.702341137123746,"#2D7DBB"],[0.705685618729097,"#2C7CBB"],[0.709030100334448,"#2B7BBA"],[0.712374581939799,"#2A7AB9"],[0.71571906354515,"#2979B9"],[0.719063545150502,"#2979B9"],[0.722408026755853,"#2878B8"],[0.725752508361204,"#2777B8"],[0.729096989966555,"#2676B7"],[0.732441471571906,"#2575B7"],[0.735785953177257,"#2474B6"],[0.739130434782609,"#2474B6"],[0.74247491638796,"#2373B6"],[0.745819397993311,"#2272B5"],[0.749163879598662,"#2171B5"],[0.752508361204013,"#2070B4"],[0.755852842809364,"#1F6FB3"],[0.759197324414716,"#1E6EB2"],[0.762541806020067,"#1E6EB2"],[0.765886287625418,"#1E6DB2"],[0.769230769230769,"#1D6CB1"],[0.77257525083612,"#1C6BB0"],[0.775919732441472,"#1B6AAF"],[0.779264214046823,"#1A69AE"],[0.782608695652174,"#1A68AE"],[0.785953177257525,"#1A68AE"],[0.789297658862876,"#1967AD"],[0.792642140468227,"#1866AC"],[0.795986622073579,"#1765AB"],[0.79933110367893,"#1664AB"],[0.802675585284281,"#1663AA"],[0.806020066889632,"#1562A9"],[0.809364548494983,"#1562A9"],[0.812709030100334,"#1461A8"],[0.816053511705686,"#1360A7"],[0.819397993311037,"#135FA7"],[0.822742474916388,"#125EA6"],[0.826086956521739,"#115DA5"],[0.82943143812709,"#105CA4"],[0.832775919732441,"#105CA4"],[0.836120401337793,"#0F5BA3"],[0.839464882943144,"#0F5AA3"],[0.842809364548495,"#0E59A2"],[0.846153846153846,"#0D58A1"],[0.849498327759197,"#0C57A0"],[0.852842809364548,"#0C57A0"],[0.8561872909699,"#0C56A0"],[0.859531772575251,"#0B559F"],[0.862876254180602,"#0A549E"],[0.866220735785953,"#09539D"],[0.869565217391304,"#08529C"],[0.872909698996655,"#08519C"],[0.876254180602007,"#08519B"],[0.879598662207358,"#08509A"],[0.882943143812709,"#084F99"],[0.88628762541806,"#084E97"],[0.889632107023411,"#084C96"],[0.892976588628763,"#084B95"],[0.896321070234114,"#084A93"],[0.899665551839465,"#084A92"],[0.903010033444816,"#084990"],[0.906354515050167,"#08488F"],[0.909698996655518,"#08478E"],[0.91304347826087,"#08468C"],[0.916387959866221,"#08458B"],[0.919732441471572,"#08448A"],[0.923076923076923,"#084489"],[0.926421404682274,"#084388"],[0.929765886287625,"#084286"],[0.933110367892977,"#084185"],[0.936454849498328,"#084083"],[0.939799331103679,"#083F82"],[0.94314381270903,"#083E81"],[0.946488294314381,"#083E7F"],[0.949832775919732,"#083D7E"],[0.953177257525084,"#083C7D"],[0.956521739130435,"#083B7B"],[0.959866220735786,"#083A7A"],[0.963210702341137,"#083979"],[0.966555183946488,"#083978"],[0.969899665551839,"#083876"],[0.973244147157191,"#083775"],[0.976588628762542,"#083674"],[0.979933110367893,"#083572"],[0.983277591973244,"#083471"],[0.986622073578595,"#083370"],[0.989966555183946,"#08336F"],[0.993311036789298,"#08326D"],[0.996655518394649,"#08316C"],[1,"#08306B"]],"colorbar":{"bgcolor":"rgba(255,255,255,1)","bordercolor":"transparent","borderwidth":1.88976377952756,"thickness":23.04,"title":null,"titlefont":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"tickmode":"array","ticktext":["0.00","0.25","0.50","0.75","1.00"],"tickvals":[0,0.25,0.5,0.75,1],"tickfont":{"color":"rgba(0,0,0,1)","family":"","size":11.689497716895},"ticklen":2,"len":0.5}},"xaxis":"x","yaxis":"y","frame":null},{"x":[1.09087788975901,1.09087788975901,null,1.09087788975901,0.500949721963311,null,0.500949721963311,0.500949721963311,null,0.500949721963311,0,null,0.500949721963311,0.500949721963311,null,0.500949721963311,0],"y":[45.609375,42.5,null,42.5,42.5,null,42.5,42,null,42,42,null,42.5,43,null,43,43],"text":["y: 1.09087789","y: 1.09087789",null,"y: 1.09087789","y: 1.09087789",null,"y: 0.50094972","y: 0.50094972",null,"y: 0.50094972","y: 0.50094972",null,"y: 0.50094972","y: 0.50094972",null,"y: 0.50094972","y: 0.50094972"],"type":"scatter","mode":"lines","line":{"width":2.26771653543307,"color":"rgba(57,190,177,1)","dash":"solid"},"hoveron":"points","showlegend":false,"xaxis":"x2","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1.09087788975901,1.09087788975901,null,1.09087788975901,0.731800222126064,null,0.731800222126064,0.731800222126064,null,0.731800222126064,0.691159533269081,null,0.691159533269081,0.691159533269081,null,0.691159533269081,0.380627098188748,null,0.380627098188748,0.380627098188748,null,0.380627098188748,0.255264316891836,null,0.255264316891836,0.255264316891836,null,0.255264316891836,0,null,0.255264316891836,0.255264316891836,null,0.255264316891836,0.185675404080306,null,0.185675404080306,0.185675404080306,null,0.185675404080306,0,null,0.185675404080306,0.185675404080306,null,0.185675404080306,0,null,0.380627098188748,0.380627098188748,null,0.380627098188748,0,null,0.691159533269081,0.691159533269081,null,0.691159533269081,0,null,0.731800222126064,0.731800222126064,null,0.731800222126064,0.466807582689693,null,0.466807582689693,0.466807582689693,null,0.466807582689693,0.358417482296682,null,0.358417482296682,0.358417482296682,null,0.358417482296682,0,null,0.358417482296682,0.358417482296682,null,0.358417482296682,0,null,0.466807582689693,0.466807582689693,null,0.466807582689693,0.173019976310583,null,0.173019976310583,0.173019976310583,null,0.173019976310583,0,null,0.173019976310583,0.173019976310583,null,0.173019976310583,0],"y":[45.609375,48.71875,null,48.71875,48.71875,null,48.71875,46.9375,null,46.9375,46.9375,null,46.9375,45.875,null,45.875,45.875,null,45.875,44.75,null,44.75,44.75,null,44.75,44,null,44,44,null,44.75,45.5,null,45.5,45.5,null,45.5,45,null,45,45,null,45.5,46,null,46,46,null,45.875,47,null,47,47,null,46.9375,48,null,48,48,null,48.71875,50.5,null,50.5,50.5,null,50.5,49.5,null,49.5,49.5,null,49.5,49,null,49,49,null,49.5,50,null,50,50,null,50.5,51.5,null,51.5,51.5,null,51.5,51,null,51,51,null,51.5,52,null,52,52],"text":["y: 1.09087789","y: 1.09087789",null,"y: 1.09087789","y: 1.09087789",null,"y: 0.73180022","y: 0.73180022",null,"y: 0.73180022","y: 0.73180022",null,"y: 0.69115953","y: 0.69115953",null,"y: 0.69115953","y: 0.69115953",null,"y: 0.38062710","y: 0.38062710",null,"y: 0.38062710","y: 0.38062710",null,"y: 0.25526432","y: 0.25526432",null,"y: 0.25526432","y: 0.25526432",null,"y: 0.25526432","y: 0.25526432",null,"y: 0.25526432","y: 0.25526432",null,"y: 0.18567540","y: 0.18567540",null,"y: 0.18567540","y: 0.18567540",null,"y: 0.18567540","y: 0.18567540",null,"y: 0.18567540","y: 0.18567540",null,"y: 0.38062710","y: 0.38062710",null,"y: 0.38062710","y: 0.38062710",null,"y: 0.69115953","y: 0.69115953",null,"y: 0.69115953","y: 0.69115953",null,"y: 0.73180022","y: 0.73180022",null,"y: 0.73180022","y: 0.73180022",null,"y: 0.46680758","y: 0.46680758",null,"y: 0.46680758","y: 0.46680758",null,"y: 0.35841748","y: 0.35841748",null,"y: 0.35841748","y: 0.35841748",null,"y: 0.35841748","y: 0.35841748",null,"y: 0.35841748","y: 0.35841748",null,"y: 0.46680758","y: 0.46680758",null,"y: 0.46680758","y: 0.46680758",null,"y: 0.17301998","y: 0.17301998",null,"y: 0.17301998","y: 0.17301998",null,"y: 0.17301998","y: 0.17301998",null,"y: 0.17301998","y: 0.17301998"],"type":"scatter","mode":"lines","line":{"width":2.26771653543307,"color":"rgba(125,176,221,1)","dash":"solid"},"hoveron":"points","showlegend":false,"xaxis":"x2","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1.05439722722438,1.05439722722438,null,1.05439722722438,0.47350573521867,null,0.47350573521867,0.47350573521867,null,0.47350573521867,0,null,0.47350573521867,0.47350573521867,null,0.47350573521867,0],"y":[37.859375,40.5,null,40.5,40.5,null,40.5,40,null,40,40,null,40.5,41,null,41,41],"text":["y: 1.05439723","y: 1.05439723",null,"y: 1.05439723","y: 1.05439723",null,"y: 0.47350574","y: 0.47350574",null,"y: 0.47350574","y: 0.47350574",null,"y: 0.47350574","y: 0.47350574",null,"y: 0.47350574","y: 0.47350574"],"type":"scatter","mode":"lines","line":{"width":2.26771653543307,"color":"rgba(134,184,117,1)","dash":"solid"},"hoveron":"points","showlegend":false,"xaxis":"x2","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1.05439722722438,1.05439722722438,null,1.05439722722438,0.659367031899142,null,0.659367031899142,0.659367031899142,null,0.659367031899142,0,null,0.659367031899142,0.659367031899142,null,0.659367031899142,0.366446818172613,null,0.366446818172613,0.366446818172613,null,0.366446818172613,0,null,0.366446818172613,0.366446818172613,null,0.366446818172613,0.257294381800551,null,0.257294381800551,0.257294381800551,null,0.257294381800551,0.202445065633869,null,0.202445065633869,0.202445065633869,null,0.202445065633869,0,null,0.202445065633869,0.202445065633869,null,0.202445065633869,0.128883042294223,null,0.128883042294223,0.128883042294223,null,0.128883042294223,0,null,0.128883042294223,0.128883042294223,null,0.128883042294223,0,null,0.257294381800551,0.257294381800551,null,0.257294381800551,0],"y":[37.859375,35.21875,null,35.21875,35.21875,null,35.21875,34,null,34,34,null,35.21875,36.4375,null,36.4375,36.4375,null,36.4375,35,null,35,35,null,36.4375,37.875,null,37.875,37.875,null,37.875,36.75,null,36.75,36.75,null,36.75,36,null,36,36,null,36.75,37.5,null,37.5,37.5,null,37.5,37,null,37,37,null,37.5,38,null,38,38,null,37.875,39,null,39,39],"text":["y: 1.05439723","y: 1.05439723",null,"y: 1.05439723","y: 1.05439723",null,"y: 0.65936703","y: 0.65936703",null,"y: 0.65936703","y: 0.65936703",null,"y: 0.65936703","y: 0.65936703",null,"y: 0.65936703","y: 0.65936703",null,"y: 0.36644682","y: 0.36644682",null,"y: 0.36644682","y: 0.36644682",null,"y: 0.36644682","y: 0.36644682",null,"y: 0.36644682","y: 0.36644682",null,"y: 0.25729438","y: 0.25729438",null,"y: 0.25729438","y: 0.25729438",null,"y: 0.20244507","y: 0.20244507",null,"y: 0.20244507","y: 0.20244507",null,"y: 0.20244507","y: 0.20244507",null,"y: 0.20244507","y: 0.20244507",null,"y: 0.12888304","y: 0.12888304",null,"y: 0.12888304","y: 0.12888304",null,"y: 0.12888304","y: 0.12888304",null,"y: 0.12888304","y: 0.12888304",null,"y: 0.25729438","y: 0.25729438",null,"y: 0.25729438","y: 0.25729438"],"type":"scatter","mode":"lines","line":{"width":2.26771653543307,"color":"rgba(199,167,108,1)","dash":"solid"},"hoveron":"points","showlegend":false,"xaxis":"x2","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1.59417446669873,1.59417446669873,null,1.59417446669873,0.614248456260233,null,0.614248456260233,0.614248456260233,null,0.614248456260233,0.275857868808986,null,0.275857868808986,0.275857868808986,null,0.275857868808986,0,null,0.275857868808986,0.275857868808986,null,0.275857868808986,0,null,0.614248456260233,0.614248456260233,null,0.614248456260233,0],"y":[49.9296875,54.25,null,54.25,54.25,null,54.25,53.5,null,53.5,53.5,null,53.5,53,null,53,53,null,53.5,54,null,54,54,null,54.25,55,null,55,55],"text":["y: 1.59417447","y: 1.59417447",null,"y: 1.59417447","y: 1.59417447",null,"y: 0.61424846","y: 0.61424846",null,"y: 0.61424846","y: 0.61424846",null,"y: 0.27585787","y: 0.27585787",null,"y: 0.27585787","y: 0.27585787",null,"y: 0.27585787","y: 0.27585787",null,"y: 0.27585787","y: 0.27585787",null,"y: 0.61424846","y: 0.61424846",null,"y: 0.61424846","y: 0.61424846"],"type":"scatter","mode":"lines","line":{"width":2.26771653543307,"color":"rgba(205,153,216,1)","dash":"solid"},"hoveron":"points","showlegend":false,"xaxis":"x2","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1.20466483415084,1.20466483415084,null,1.20466483415084,0.95788286362071,null,0.95788286362071,0.95788286362071,null,0.95788286362071,0.794745500284792,null,0.794745500284792,0.794745500284792,null,0.794745500284792,0.381625693962732,null,0.381625693962732,0.381625693962732,null,0.381625693962732,0,null,0.381625693962732,0.381625693962732,null,0.381625693962732,0,null,0.794745500284792,0.794745500284792,null,0.794745500284792,0.666404776934707,null,0.666404776934707,0.666404776934707,null,0.666404776934707,0.367061621439879,null,0.367061621439879,0.367061621439879,null,0.367061621439879,0,null,0.367061621439879,0.367061621439879,null,0.367061621439879,0.248285826616636,null,0.248285826616636,0.248285826616636,null,0.248285826616636,0.0875581071177939,null,0.0875581071177939,0.0875581071177939,null,0.0875581071177939,0,null,0.0875581071177939,0.0875581071177939,null,0.0875581071177939,0,null,0.248285826616636,0.248285826616636,null,0.248285826616636,0.237569649714429,null,0.237569649714429,0.237569649714429,null,0.237569649714429,0.129904663635189,null,0.129904663635189,0.129904663635189,null,0.129904663635189,0,null,0.129904663635189,0.129904663635189,null,0.129904663635189,0,null,0.237569649714429,0.237569649714429,null,0.237569649714429,0,null,0.666404776934707,0.666404776934707,null,0.666404776934707,0.534875118690448,null,0.534875118690448,0.534875118690448,null,0.534875118690448,0.438363599370831,null,0.438363599370831,0.438363599370831,null,0.438363599370831,0.20347407832471,null,0.20347407832471,0.20347407832471,null,0.20347407832471,0,null,0.20347407832471,0.20347407832471,null,0.20347407832471,0,null,0.438363599370831,0.438363599370831,null,0.438363599370831,0.332059186086582,null,0.332059186086582,0.332059186086582,null,0.332059186086582,0.274199290988404,null,0.274199290988404,0.274199290988404,null,0.274199290988404,0,null,0.274199290988404,0.274199290988404,null,0.274199290988404,0.143438256757787,null,0.143438256757787,0.143438256757787,null,0.143438256757787,0,null,0.143438256757787,0.143438256757787,null,0.143438256757787,0,null,0.332059186086582,0.332059186086582,null,0.332059186086582,0.20894513119211,null,0.20894513119211,0.20894513119211,null,0.20894513119211,0,null,0.20894513119211,0.20894513119211,null,0.20894513119211,0.154458411843452,null,0.154458411843452,0.154458411843452,null,0.154458411843452,0.0903999281103318,null,0.0903999281103318,0.0903999281103318,null,0.0903999281103318,0,null,0.0903999281103318,0.0903999281103318,null,0.0903999281103318,0,null,0.154458411843452,0.154458411843452,null,0.154458411843452,0,null,0.534875118690448,0.534875118690448,null,0.534875118690448,0.372123275507687,null,0.372123275507687,0.372123275507687,null,0.372123275507687,0.35032421362806,null,0.35032421362806,0.35032421362806,null,0.35032421362806,0.243402838861005,null,0.243402838861005,0.243402838861005,null,0.243402838861005,0.212443162483461,null,0.212443162483461,0.212443162483461,null,0.212443162483461,0.127081711167596,null,0.127081711167596,0.127081711167596,null,0.127081711167596,0,null,0.127081711167596,0.127081711167596,null,0.127081711167596,0,null,0.212443162483461,0.212443162483461,null,0.212443162483461,0,null,0.243402838861005,0.243402838861005,null,0.243402838861005,0,null,0.35032421362806,0.35032421362806,null,0.35032421362806,0,null,0.372123275507687,0.372123275507687,null,0.372123275507687,0.182577335771184,null,0.182577335771184,0.182577335771184,null,0.182577335771184,0,null,0.182577335771184,0.182577335771184,null,0.182577335771184,0.121298312556643,null,0.121298312556643,0.121298312556643,null,0.121298312556643,0,null,0.121298312556643,0.121298312556643,null,0.121298312556643,0.0949020854651998,null,0.0949020854651998,0.0949020854651998,null,0.0949020854651998,0,null,0.0949020854651998,0.0949020854651998,null,0.0949020854651998,0,null,0.95788286362071,0.95788286362071,null,0.95788286362071,0.407830626686954,null,0.407830626686954,0.407830626686954,null,0.407830626686954,0.218154484501404,null,0.218154484501404,0.218154484501404,null,0.218154484501404,0.0991867532043192,null,0.0991867532043192,0.0991867532043192,null,0.0991867532043192,0,null,0.0991867532043192,0.0991867532043192,null,0.0991867532043192,0,null,0.218154484501404,0.218154484501404,null,0.218154484501404,0,null,0.407830626686954,0.407830626686954,null,0.407830626686954,0.319599624724541,null,0.319599624724541,0.319599624724541,null,0.319599624724541,0.209182472124278,null,0.209182472124278,0.209182472124278,null,0.209182472124278,0,null,0.209182472124278,0.209182472124278,null,0.209182472124278,0,null,0.319599624724541,0.319599624724541,null,0.319599624724541,0.119504801735496,null,0.119504801735496,0.119504801735496,null,0.119504801735496,0,null,0.119504801735496,0.119504801735496,null,0.119504801735496,0],"y":[27.923828125,17.98828125,null,17.98828125,17.98828125,null,17.98828125,6.1015625,null,6.1015625,6.1015625,null,6.1015625,1.5,null,1.5,1.5,null,1.5,1,null,1,1,null,1.5,2,null,2,2,null,6.1015625,10.703125,null,10.703125,10.703125,null,10.703125,4.4375,null,4.4375,4.4375,null,4.4375,3,null,3,3,null,4.4375,5.875,null,5.875,5.875,null,5.875,4.5,null,4.5,4.5,null,4.5,4,null,4,4,null,4.5,5,null,5,5,null,5.875,7.25,null,7.25,7.25,null,7.25,6.5,null,6.5,6.5,null,6.5,6,null,6,6,null,6.5,7,null,7,7,null,7.25,8,null,8,8,null,10.703125,16.96875,null,16.96875,16.96875,null,16.96875,11.46875,null,11.46875,11.46875,null,11.46875,9.5,null,9.5,9.5,null,9.5,9,null,9,9,null,9.5,10,null,10,10,null,11.46875,13.4375,null,13.4375,13.4375,null,13.4375,11.75,null,11.75,11.75,null,11.75,11,null,11,11,null,11.75,12.5,null,12.5,12.5,null,12.5,12,null,12,12,null,12.5,13,null,13,13,null,13.4375,15.125,null,15.125,15.125,null,15.125,14,null,14,14,null,15.125,16.25,null,16.25,16.25,null,16.25,15.5,null,15.5,15.5,null,15.5,15,null,15,15,null,15.5,16,null,16,16,null,16.25,17,null,17,17,null,16.96875,22.46875,null,22.46875,22.46875,null,22.46875,21.0625,null,21.0625,21.0625,null,21.0625,20.125,null,20.125,20.125,null,20.125,19.25,null,19.25,19.25,null,19.25,18.5,null,18.5,18.5,null,18.5,18,null,18,18,null,18.5,19,null,19,19,null,19.25,20,null,20,20,null,20.125,21,null,21,21,null,21.0625,22,null,22,22,null,22.46875,23.875,null,23.875,23.875,null,23.875,23,null,23,23,null,23.875,24.75,null,24.75,24.75,null,24.75,24,null,24,24,null,24.75,25.5,null,25.5,25.5,null,25.5,25,null,25,25,null,25.5,26,null,26,26,null,17.98828125,29.875,null,29.875,29.875,null,29.875,28.25,null,28.25,28.25,null,28.25,27.5,null,27.5,27.5,null,27.5,27,null,27,27,null,27.5,28,null,28,28,null,28.25,29,null,29,29,null,29.875,31.5,null,31.5,31.5,null,31.5,30.5,null,30.5,30.5,null,30.5,30,null,30,30,null,30.5,31,null,31,31,null,31.5,32.5,null,32.5,32.5,null,32.5,32,null,32,32,null,32.5,33,null,33,33],"text":["y: 1.20466483","y: 1.20466483",null,"y: 1.20466483","y: 1.20466483",null,"y: 0.95788286","y: 0.95788286",null,"y: 0.95788286","y: 0.95788286",null,"y: 0.79474550","y: 0.79474550",null,"y: 0.79474550","y: 0.79474550",null,"y: 0.38162569","y: 0.38162569",null,"y: 0.38162569","y: 0.38162569",null,"y: 0.38162569","y: 0.38162569",null,"y: 0.38162569","y: 0.38162569",null,"y: 0.79474550","y: 0.79474550",null,"y: 0.79474550","y: 0.79474550",null,"y: 0.66640478","y: 0.66640478",null,"y: 0.66640478","y: 0.66640478",null,"y: 0.36706162","y: 0.36706162",null,"y: 0.36706162","y: 0.36706162",null,"y: 0.36706162","y: 0.36706162",null,"y: 0.36706162","y: 0.36706162",null,"y: 0.24828583","y: 0.24828583",null,"y: 0.24828583","y: 0.24828583",null,"y: 0.08755811","y: 0.08755811",null,"y: 0.08755811","y: 0.08755811",null,"y: 0.08755811","y: 0.08755811",null,"y: 0.08755811","y: 0.08755811",null,"y: 0.24828583","y: 0.24828583",null,"y: 0.24828583","y: 0.24828583",null,"y: 0.23756965","y: 0.23756965",null,"y: 0.23756965","y: 0.23756965",null,"y: 0.12990466","y: 0.12990466",null,"y: 0.12990466","y: 0.12990466",null,"y: 0.12990466","y: 0.12990466",null,"y: 0.12990466","y: 0.12990466",null,"y: 0.23756965","y: 0.23756965",null,"y: 0.23756965","y: 0.23756965",null,"y: 0.66640478","y: 0.66640478",null,"y: 0.66640478","y: 0.66640478",null,"y: 0.53487512","y: 0.53487512",null,"y: 0.53487512","y: 0.53487512",null,"y: 0.43836360","y: 0.43836360",null,"y: 0.43836360","y: 0.43836360",null,"y: 0.20347408","y: 0.20347408",null,"y: 0.20347408","y: 0.20347408",null,"y: 0.20347408","y: 0.20347408",null,"y: 0.20347408","y: 0.20347408",null,"y: 0.43836360","y: 0.43836360",null,"y: 0.43836360","y: 0.43836360",null,"y: 0.33205919","y: 0.33205919",null,"y: 0.33205919","y: 0.33205919",null,"y: 0.27419929","y: 0.27419929",null,"y: 0.27419929","y: 0.27419929",null,"y: 0.27419929","y: 0.27419929",null,"y: 0.27419929","y: 0.27419929",null,"y: 0.14343826","y: 0.14343826",null,"y: 0.14343826","y: 0.14343826",null,"y: 0.14343826","y: 0.14343826",null,"y: 0.14343826","y: 0.14343826",null,"y: 0.33205919","y: 0.33205919",null,"y: 0.33205919","y: 0.33205919",null,"y: 0.20894513","y: 0.20894513",null,"y: 0.20894513","y: 0.20894513",null,"y: 0.20894513","y: 0.20894513",null,"y: 0.20894513","y: 0.20894513",null,"y: 0.15445841","y: 0.15445841",null,"y: 0.15445841","y: 0.15445841",null,"y: 0.09039993","y: 0.09039993",null,"y: 0.09039993","y: 0.09039993",null,"y: 0.09039993","y: 0.09039993",null,"y: 0.09039993","y: 0.09039993",null,"y: 0.15445841","y: 0.15445841",null,"y: 0.15445841","y: 0.15445841",null,"y: 0.53487512","y: 0.53487512",null,"y: 0.53487512","y: 0.53487512",null,"y: 0.37212328","y: 0.37212328",null,"y: 0.37212328","y: 0.37212328",null,"y: 0.35032421","y: 0.35032421",null,"y: 0.35032421","y: 0.35032421",null,"y: 0.24340284","y: 0.24340284",null,"y: 0.24340284","y: 0.24340284",null,"y: 0.21244316","y: 0.21244316",null,"y: 0.21244316","y: 0.21244316",null,"y: 0.12708171","y: 0.12708171",null,"y: 0.12708171","y: 0.12708171",null,"y: 0.12708171","y: 0.12708171",null,"y: 0.12708171","y: 0.12708171",null,"y: 0.21244316","y: 0.21244316",null,"y: 0.21244316","y: 0.21244316",null,"y: 0.24340284","y: 0.24340284",null,"y: 0.24340284","y: 0.24340284",null,"y: 0.35032421","y: 0.35032421",null,"y: 0.35032421","y: 0.35032421",null,"y: 0.37212328","y: 0.37212328",null,"y: 0.37212328","y: 0.37212328",null,"y: 0.18257734","y: 0.18257734",null,"y: 0.18257734","y: 0.18257734",null,"y: 0.18257734","y: 0.18257734",null,"y: 0.18257734","y: 0.18257734",null,"y: 0.12129831","y: 0.12129831",null,"y: 0.12129831","y: 0.12129831",null,"y: 0.12129831","y: 0.12129831",null,"y: 0.12129831","y: 0.12129831",null,"y: 0.09490209","y: 0.09490209",null,"y: 0.09490209","y: 0.09490209",null,"y: 0.09490209","y: 0.09490209",null,"y: 0.09490209","y: 0.09490209",null,"y: 0.95788286","y: 0.95788286",null,"y: 0.95788286","y: 0.95788286",null,"y: 0.40783063","y: 0.40783063",null,"y: 0.40783063","y: 0.40783063",null,"y: 0.21815448","y: 0.21815448",null,"y: 0.21815448","y: 0.21815448",null,"y: 0.09918675","y: 0.09918675",null,"y: 0.09918675","y: 0.09918675",null,"y: 0.09918675","y: 0.09918675",null,"y: 0.09918675","y: 0.09918675",null,"y: 0.21815448","y: 0.21815448",null,"y: 0.21815448","y: 0.21815448",null,"y: 0.40783063","y: 0.40783063",null,"y: 0.40783063","y: 0.40783063",null,"y: 0.31959962","y: 0.31959962",null,"y: 0.31959962","y: 0.31959962",null,"y: 0.20918247","y: 0.20918247",null,"y: 0.20918247","y: 0.20918247",null,"y: 0.20918247","y: 0.20918247",null,"y: 0.20918247","y: 0.20918247",null,"y: 0.31959962","y: 0.31959962",null,"y: 0.31959962","y: 0.31959962",null,"y: 0.11950480","y: 0.11950480",null,"y: 0.11950480","y: 0.11950480",null,"y: 0.11950480","y: 0.11950480",null,"y: 0.11950480","y: 0.11950480"],"type":"scatter","mode":"lines","line":{"width":2.26771653543307,"color":"rgba(228,149,165,1)","dash":"solid"},"hoveron":"points","showlegend":false,"xaxis":"x2","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1.82287715348419,1.82287715348419,null,1.82287715348419,1.20466483415084,null,1.20466483415084,1.20466483415084,null,1.20466483415084,1.05439722722438,null,1.82287715348419,1.82287715348419,null,1.82287715348419,1.59417446669873,null,1.59417446669873,1.59417446669873,null,1.59417446669873,1.09087788975901],"y":[38.9267578125,27.923828125,null,27.923828125,27.923828125,null,27.923828125,37.859375,null,37.859375,37.859375,null,38.9267578125,49.9296875,null,49.9296875,49.9296875,null,49.9296875,45.609375,null,45.609375,45.609375],"text":["y: 1.82287715","y: 1.82287715",null,"y: 1.82287715","y: 1.82287715",null,"y: 1.20466483","y: 1.20466483",null,"y: 1.20466483","y: 1.20466483",null,"y: 1.82287715","y: 1.82287715",null,"y: 1.82287715","y: 1.82287715",null,"y: 1.59417447","y: 1.59417447",null,"y: 1.59417447","y: 1.59417447"],"type":"scatter","mode":"lines","line":{"width":2.26771653543307,"color":"rgba(0,0,0,1)","dash":"solid"},"hoveron":"points","showlegend":false,"xaxis":"x2","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1.82287715348419,1.20466483415084,0.95788286362071,0.794745500284792,0.381625693962732,0,0,0.666404776934707,0.367061621439879,0,0.248285826616636,0.0875581071177939,0,0,0.237569649714429,0.129904663635189,0,0,0,0.534875118690448,0.438363599370831,0.20347407832471,0,0,0.332059186086582,0.274199290988404,0,0.143438256757787,0,0,0.20894513119211,0,0.154458411843452,0.0903999281103318,0,0,0,0.372123275507687,0.35032421362806,0.243402838861005,0.212443162483461,0.127081711167596,0,0,0,0,0,0.182577335771184,0,0.121298312556643,0,0.0949020854651998,0,0,0.407830626686954,0.218154484501404,0.0991867532043192,0,0,0,0.319599624724541,0.209182472124278,0,0,0.119504801735496,0,0,1.05439722722438,0.659367031899142,0,0.366446818172613,0,0.257294381800551,0.202445065633869,0,0.128883042294223,0,0,0,0.47350573521867,0,0,1.59417446669873,1.09087788975901,0.500949721963311,0,0,0.731800222126064,0.691159533269081,0.380627098188748,0.255264316891836,0,0.185675404080306,0,0,0,0,0.466807582689693,0.358417482296682,0,0,0.173019976310583,0,0,0.614248456260233,0.275857868808986,0,0,0],"y":[38.9267578125,27.923828125,17.98828125,6.1015625,1.5,1,2,10.703125,4.4375,3,5.875,4.5,4,5,7.25,6.5,6,7,8,16.96875,11.46875,9.5,9,10,13.4375,11.75,11,12.5,12,13,15.125,14,16.25,15.5,15,16,17,22.46875,21.0625,20.125,19.25,18.5,18,19,20,21,22,23.875,23,24.75,24,25.5,25,26,29.875,28.25,27.5,27,28,29,31.5,30.5,30,31,32.5,32,33,37.859375,35.21875,34,36.4375,35,37.875,36.75,36,37.5,37,38,39,40.5,40,41,49.9296875,45.609375,42.5,42,43,48.71875,46.9375,45.875,44.75,44,45.5,45,46,47,48,50.5,49.5,49,50,51.5,51,52,54.25,53.5,53,54,55],"text":["y: 1.82287715","y: 1.20466483","y: 0.95788286","y: 0.79474550","y: 0.38162569","y: 0.00000000","y: 0.00000000","y: 0.66640478","y: 0.36706162","y: 0.00000000","y: 0.24828583","y: 0.08755811","y: 0.00000000","y: 0.00000000","y: 0.23756965","y: 0.12990466","y: 0.00000000","y: 0.00000000","y: 0.00000000","y: 0.53487512","y: 0.43836360","y: 0.20347408","y: 0.00000000","y: 0.00000000","y: 0.33205919","y: 0.27419929","y: 0.00000000","y: 0.14343826","y: 0.00000000","y: 0.00000000","y: 0.20894513","y: 0.00000000","y: 0.15445841","y: 0.09039993","y: 0.00000000","y: 0.00000000","y: 0.00000000","y: 0.37212328","y: 0.35032421","y: 0.24340284","y: 0.21244316","y: 0.12708171","y: 0.00000000","y: 0.00000000","y: 0.00000000","y: 0.00000000","y: 0.00000000","y: 0.18257734","y: 0.00000000","y: 0.12129831","y: 0.00000000","y: 0.09490209","y: 0.00000000","y: 0.00000000","y: 0.40783063","y: 0.21815448","y: 0.09918675","y: 0.00000000","y: 0.00000000","y: 0.00000000","y: 0.31959962","y: 0.20918247","y: 0.00000000","y: 0.00000000","y: 0.11950480","y: 0.00000000","y: 0.00000000","y: 1.05439723","y: 0.65936703","y: 0.00000000","y: 0.36644682","y: 0.00000000","y: 0.25729438","y: 0.20244507","y: 0.00000000","y: 0.12888304","y: 0.00000000","y: 0.00000000","y: 0.00000000","y: 0.47350574","y: 0.00000000","y: 0.00000000","y: 1.59417447","y: 1.09087789","y: 0.50094972","y: 0.00000000","y: 0.00000000","y: 0.73180022","y: 0.69115953","y: 0.38062710","y: 0.25526432","y: 0.00000000","y: 0.18567540","y: 0.00000000","y: 0.00000000","y: 0.00000000","y: 0.00000000","y: 0.46680758","y: 0.35841748","y: 0.00000000","y: 0.00000000","y: 0.17301998","y: 0.00000000","y: 0.00000000","y: 0.61424846","y: 0.27585787","y: 0.00000000","y: 0.00000000","y: 0.00000000"],"type":"scatter","mode":"markers","marker":{"autocolorscale":false,"color":"transparent","opacity":1,"size":null,"symbol":null,"line":{"width":1.88976377952756,"color":"transparent"}},"hoveron":"points","showlegend":false,"xaxis":"x2","yaxis":"y","hoverinfo":"text","frame":null}],"layout":{"xaxis":{"domain":[0,0.8],"automargin":true,"type":"linear","autorange":false,"range":[0.5,5.5],"tickmode":"array","ticktext":["RADIO_PR","TV_PR","LLPHONE_PR","MPHONE_PR","COMPUTER_PR"],"tickvals":[1,2,3,4,5],"categoryorder":"array","categoryarray":["RADIO_PR","TV_PR","LLPHONE_PR","MPHONE_PR","COMPUTER_PR"],"nticks":null,"ticks":"outside","tickcolor":"rgba(51,51,51,1)","ticklen":3.65296803652968,"tickwidth":0.66417600664176,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":6.6417600664176},"tickangle":-45,"showline":true,"linecolor":"rgba(0,0,0,1)","linewidth":0.66417600664176,"showgrid":false,"gridcolor":null,"gridwidth":0,"zeroline":false,"anchor":"y","title":"ICT Indicators","hoverformat":".2f"},"xaxis2":{"domain":[0.8,1],"automargin":true,"type":"linear","autorange":false,"range":[0,1.82287715348419],"tickmode":"array","ticktext":["0.0","0.5","1.0","1.5"],"tickvals":[0,0.5,1,1.5],"categoryorder":"array","categoryarray":["0.0","0.5","1.0","1.5"],"nticks":null,"ticks":"","tickcolor":null,"ticklen":3.65296803652968,"tickwidth":0,"showticklabels":false,"tickfont":{"color":null,"family":null,"size":0},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":false,"gridcolor":null,"gridwidth":0,"zeroline":false,"anchor":"y","title":{"text":"","font":{"color":null,"family":null,"size":0}},"hoverformat":".2f"},"yaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[0.5,55.5],"tickmode":"array","ticktext":["Narphan","Pangwaun","Mongpan","Nansang","Kyaukme","Kalaw","Hseni","Langkho","Kunhing","Laihka","Monghsat","Loilen","Mongnai","Mongton","Hsipaw","Hopong","Monghsu","Pinlaung","Mongmit","Pekon","Kutkai","Namtu","Hsihseng","Kyethi","Tangyan","Namhsan","Mongyai","Mongping","Mongkhet","Mawkmai","Mongkaing","Manton","Matman","Ywangan","Pindaya","Lawksawk","Nyaungshwe","Nawnghkio","Monghpyak","Mabein","Mongyawng","Kunlong","Konkyan","Mongyang","Mongmao","Hopang","Namhkan","Laukkaing","Mongla","Pangsang","Kengtung","Lashio","Taunggyi","Tachileik","Muse"],"tickvals":[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55],"categoryorder":"array","categoryarray":["Narphan","Pangwaun","Mongpan","Nansang","Kyaukme","Kalaw","Hseni","Langkho","Kunhing","Laihka","Monghsat","Loilen","Mongnai","Mongton","Hsipaw","Hopong","Monghsu","Pinlaung","Mongmit","Pekon","Kutkai","Namtu","Hsihseng","Kyethi","Tangyan","Namhsan","Mongyai","Mongping","Mongkhet","Mawkmai","Mongkaing","Manton","Matman","Ywangan","Pindaya","Lawksawk","Nyaungshwe","Nawnghkio","Monghpyak","Mabein","Mongyawng","Kunlong","Konkyan","Mongyang","Mongmao","Hopang","Namhkan","Laukkaing","Mongla","Pangsang","Kengtung","Lashio","Taunggyi","Tachileik","Muse"],"nticks":null,"ticks":"outside","tickcolor":"rgba(51,51,51,1)","ticklen":3.65296803652968,"tickwidth":0.66417600664176,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":5.31340805313408},"tickangle":-0,"showline":true,"linecolor":"rgba(0,0,0,1)","linewidth":0.66417600664176,"showgrid":false,"gridcolor":null,"gridwidth":0,"zeroline":false,"anchor":"x","title":"Townships of Shan State","hoverformat":".2f"},"annotations":[],"shapes":[{"type":"rect","fillcolor":null,"line":{"color":null,"width":0,"linetype":[]},"yref":"paper","xref":"paper","x0":0,"x1":0.8,"y0":0,"y1":1},{"type":"rect","fillcolor":null,"line":{"color":null,"width":0,"linetype":[]},"yref":"paper","xref":"paper","x0":0.8,"x1":1,"y0":0,"y1":1}],"images":[],"margin":{"t":60,"r":null,"b":35.0684931506849,"l":200},"paper_bgcolor":"rgba(255,255,255,1)","font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"showlegend":false,"legend":{"bgcolor":"rgba(255,255,255,1)","bordercolor":"transparent","borderwidth":1.88976377952756,"font":{"color":"rgba(0,0,0,1)","family":"","size":11.689497716895},"title":{"text":"","font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187}},"y":1,"yanchor":"top"},"hovermode":"closest","barmode":"relative","title":"Geographic Segmentation of Shan State by ICT indicators"},"attrs":{"13fc69503e0":{"x":{},"y":{},"fill":{},"text":{},"type":"heatmap"},"13fc70cecec":{"xend":{},"yend":{},"colour":{},"linetype":{},"size":{},"x":{},"y":{},"type":"scatter"},"13fc6aca7f35":{"colour":{},"shape":{},"size":{},"x":{},"y":{}}},"source":"A","config":{"doubleClick":"reset","modeBarButtonsToAdd":["hoverclosest","hovercompare"],"showSendToCloud":false,"displaylogo":false,"modeBarButtonsToRemove":["sendDataToCloud","select2d","lasso2d","autoScale2d","hoverClosestCartesian","hoverCompareCartesian","sendDataToCloud"]},"highlight":{"on":"plotly_click","persistent":false,"dynamic":false,"selectize":false,"opacityDim":0.2,"selected":{"opacity":1},"debounce":0},"subplot":true,"shinyEvents":["plotly_hover","plotly_click","plotly_selected","plotly_relayout","plotly_brushed","plotly_brushing","plotly_clickannotation","plotly_doubleclick","plotly_deselect","plotly_afterplot","plotly_sunburstclick"],"base_url":"https://plot.ly"},"evals":[],"jsHooks":[]}</script>
```
:::
:::


### Mapping the clusters formed

With closed examination of the dendragram above, we have decided to retain six clusters.

[*cutree()*](https://stat.ethz.ch/R-manual/R-devel/library/stats/html/cutree.html) of R Base will be used in the code chunk below to derive a 6-cluster model.


::: {.cell}

```{.r .cell-code}
groups <- as.factor(cutree(hclust_ward, k=6))
```
:::


The output is called *groups*. It is a *list* object.

In order to visualise the clusters, the *groups* object need to be appended onto *shan_sf* simple feature object.

The code chunk below form the join in three steps:

-   the *groups* list object will be converted into a matrix;

-   *cbind()* is used to append *groups* matrix onto shan_sf to produce an output simple feature object called `shan_sf_cluster`; and

-   *rename* of **dplyr** package is used to rename *as.matrix.groups* field as *CLUSTER*.


::: {.cell}

```{.r .cell-code}
shan_sf_cluster <- cbind(shan_sf, as.matrix(groups)) %>%
  rename(`CLUSTER`=`as.matrix.groups.`)
```
:::


Next, *qtm()* of **tmap** package is used to plot the choropleth map showing the cluster formed.


::: {.cell}

```{.r .cell-code}
qtm(shan_sf_cluster, "CLUSTER")
```

::: {.cell-output-display}
![](In-class_Exercise3_files/figure-html/unnamed-chunk-38-1.png){width=672}
:::
:::


The choropleth map above reveals the clusters are very fragmented. The is one of the major limitation when non-spatial clustering algorithm such as hierarchical cluster analysis method is used.

## Spatially Constrained Clustering - SKATER approach

In this section, you will learn how to derive spatially constrained cluster by using [*skater()*](https://r-spatial.github.io/spdep/reference/skater.html) method of [**spdep**](https://r-spatial.github.io/spdep/) package.

### Converting into SpatialPolygonsDataFrame

First, we need to convert `shan_sf` into SpatialPolygonsDataFrame. This is because SKATER function only support **sp** objects such as SpatialPolygonDataFrame.

The code chunk below uses [*as_Spatial()*](https://r-spatial.github.io/sf/reference/coerce-methods.html) of **sf** package to convert *shan_sf* into a SpatialPolygonDataFrame called *shan_sp*.


::: {.cell}

```{.r .cell-code}
shan_sp <- as_Spatial(shan_sf)
```
:::


### Computing Neighbour List

Next, [poly2nd()](https://r-spatial.github.io/spdep/reference/poly2nb.html) of **spdep** package will be used to compute the neighbours list from polygon list.


::: {.cell}

```{.r .cell-code}
shan.nb <- poly2nb(shan_sp)
summary(shan.nb)
```

::: {.cell-output .cell-output-stdout}
```
Neighbour list object:
Number of regions: 55 
Number of nonzero links: 264 
Percentage nonzero weights: 8.727273 
Average number of links: 4.8 
Link number distribution:

 2  3  4  5  6  7  8  9 
 5  9  7 21  4  3  5  1 
5 least connected regions:
3 5 7 9 47 with 2 links
1 most connected region:
8 with 9 links
```
:::
:::


We can plot the neighbours list on shan_sp by using the code chunk below. Since we now can plot the community area boundaries as well, we plot this graph on top of the map. The first plot command gives the boundaries. This is followed by the plot of the neighbor list object, with coordinates applied to the original SpatialPolygonDataFrame (Shan state township boundaries) to extract the centroids of the polygons. These are used as the nodes for the graph representation. We also set the color to blue and specify add=TRUE to plot the network on top of the boundaries.


::: {.cell}

```{.r .cell-code}
plot(shan_sp, 
     border=grey(.5))
plot(shan.nb, 
     coordinates(shan_sp), 
     col="blue", 
     add=TRUE)
```

::: {.cell-output-display}
![](In-class_Exercise3_files/figure-html/unnamed-chunk-41-1.png){width=672}
:::
:::


Note that if you plot the network first and then the boundaries, some of the areas will be clipped. This is because the plotting area is determined by the characteristics of the first plot. In this example, because the boundary map extends further than the graph, we plot it first.

### Computing minimum spanning tree

#### Calculating edge costs

Next, [*nbcosts()*](https://r-spatial.github.io/spdep/reference/nbcosts.html) of **spdep** package is used to compute the cost of each edge. It is the distance between it nodes. This function compute this distance using a data.frame with observations vector in each node.

The code chunk below is used to compute the cost of each edge.


::: {.cell}

```{.r .cell-code}
lcosts <- nbcosts(shan.nb, shan_ict)
```
:::


For each observation, this gives the pairwise dissimilarity between its values on the five variables and the values for the neighbouring observation (from the neighbour list). Basically, this is the notion of a generalised weight for a spatial weights matrix.

Next, We will incorporate these costs into a weights object in the same way as we did in the calculation of inverse of distance weights. In other words, we convert the neighbour list to a list weights object by specifying the just computed ***lcosts*** as the weights.

In order to achieve this, [*nb2listw()*](https://r-spatial.github.io/spdep/reference/nb2listw.html) of **spdep** package is used as shown in the code chunk below.

Note that we specify the *style* as **B** to make sure the cost values are not row-standardised.


::: {.cell}

```{.r .cell-code}
shan.w <- nb2listw(shan.nb, 
                   lcosts, 
                   style="B")
summary(shan.w)
```

::: {.cell-output .cell-output-stdout}
```
Characteristics of weights list object:
Neighbour list object:
Number of regions: 55 
Number of nonzero links: 264 
Percentage nonzero weights: 8.727273 
Average number of links: 4.8 
Link number distribution:

 2  3  4  5  6  7  8  9 
 5  9  7 21  4  3  5  1 
5 least connected regions:
3 5 7 9 47 with 2 links
1 most connected region:
8 with 9 links

Weights style: B 
Weights constants summary:
   n   nn       S0       S1        S2
B 55 3025 76267.65 58260785 522016004
```
:::
:::


### Computing minimum spanning tree

The minimum spanning tree is computed by mean of the [*mstree()*](https://r-spatial.github.io/spdep/reference/mstree.html) of **spdep** package as shown in the code chunk below.


::: {.cell}

```{.r .cell-code}
shan.mst <- mstree(shan.w)
```
:::


After computing the MST, we can check its class and dimension by using the code chunk below.


::: {.cell}

```{.r .cell-code}
class(shan.mst)
```

::: {.cell-output .cell-output-stdout}
```
[1] "mst"    "matrix"
```
:::
:::

::: {.cell}

```{.r .cell-code}
dim(shan.mst)
```

::: {.cell-output .cell-output-stdout}
```
[1] 54  3
```
:::
:::


Note that the dimension is 54 and not 55. This is because the minimum spanning tree consists on n-1 edges (links) in order to traverse all the nodes.

We can display the content of *shan.mst* by using *head()* as shown in the code chunk below.


::: {.cell}

```{.r .cell-code}
head(shan.mst)
```

::: {.cell-output .cell-output-stdout}
```
     [,1] [,2]      [,3]
[1,]   31   25 229.44658
[2,]   25   10 163.95741
[3,]   10    1 144.02475
[4,]   10    9 157.04230
[5,]    9    8  90.82891
[6,]    8    6 140.01101
```
:::
:::


The plot method for the MST include a way to show the observation numbers of the nodes in addition to the edge. As before, we plot this together with the township boundaries. We can see how the initial neighbour list is simplified to just one edge connecting each of the nodes, while passing through all the nodes.


::: {.cell}

```{.r .cell-code}
plot(shan_sp, border=gray(.5))
plot.mst(shan.mst, 
         coordinates(shan_sp), 
         col="blue", 
         cex.lab=0.7, 
         cex.circles=0.005, 
         add=TRUE)
```

::: {.cell-output-display}
![](In-class_Exercise3_files/figure-html/unnamed-chunk-48-1.png){width=672}
:::
:::


### Computing spatially constrained clusters using SKATER method

The code chunk below compute the spatially constrained cluster using [*skater()*](https://r-spatial.github.io/spdep/reference/skater.html) of **spdep** package.


::: {.cell}

```{.r .cell-code}
clust6 <- skater(edges = shan.mst[,1:2], 
                 data = shan_ict, 
                 method = "euclidean", 
                 ncuts = 5)
```
:::


The *skater()* takes three mandatory arguments: - the first two columns of the MST matrix (i.e. not the cost), - the data matrix (to update the costs as units are being grouped), and - the number of cuts. Note: It is set to **one less than the number of clusters**. So, the value specified is **not** the number of clusters, but the number of cuts in the graph, one less than the number of clusters.

The result of the *skater()* is an object of class **skater**. We can examine its contents by using the code chunk below.


::: {.cell}

```{.r .cell-code}
str(clust6)
```

::: {.cell-output .cell-output-stdout}
```
List of 8
 $ groups      : num [1:55] 3 3 6 3 3 3 3 3 3 3 ...
 $ edges.groups:List of 6
  ..$ :List of 3
  .. ..$ node: num [1:22] 13 48 54 55 45 37 34 16 25 31 ...
  .. ..$ edge: num [1:21, 1:3] 48 55 54 37 34 16 45 31 13 13 ...
  .. ..$ ssw : num 3423
  ..$ :List of 3
  .. ..$ node: num [1:18] 47 27 53 38 42 15 41 51 43 32 ...
  .. ..$ edge: num [1:17, 1:3] 53 15 42 38 41 51 15 27 15 43 ...
  .. ..$ ssw : num 3759
  ..$ :List of 3
  .. ..$ node: num [1:11] 2 6 8 1 36 4 10 9 46 5 ...
  .. ..$ edge: num [1:10, 1:3] 6 1 8 36 4 6 8 10 10 9 ...
  .. ..$ ssw : num 1458
  ..$ :List of 3
  .. ..$ node: num [1:2] 44 20
  .. ..$ edge: num [1, 1:3] 44 20 95
  .. ..$ ssw : num 95
  ..$ :List of 3
  .. ..$ node: num 23
  .. ..$ edge: num[0 , 1:3] 
  .. ..$ ssw : num 0
  ..$ :List of 3
  .. ..$ node: num 3
  .. ..$ edge: num[0 , 1:3] 
  .. ..$ ssw : num 0
 $ not.prune   : NULL
 $ candidates  : int [1:6] 1 2 3 4 5 6
 $ ssto        : num 12613
 $ ssw         : num [1:6] 12613 10977 9962 9540 9123 ...
 $ crit        : num [1:2] 1 Inf
 $ vec.crit    : num [1:55] 1 1 1 1 1 1 1 1 1 1 ...
 - attr(*, "class")= chr "skater"
```
:::
:::


The most interesting component of this list structure is the groups vector containing the labels of the cluster to which each observation belongs (as before, the label itself is arbitary). This is followed by a detailed summary for each of the clusters in the edges.groups list. Sum of squares measures are given as ssto for the total and ssw to show the effect of each of the cuts on the overall criterion.

We can check the cluster assignment by using the conde chunk below.


::: {.cell}

```{.r .cell-code}
ccs6 <- clust6$groups
ccs6
```

::: {.cell-output .cell-output-stdout}
```
 [1] 3 3 6 3 3 3 3 3 3 3 2 1 1 1 2 1 1 1 2 4 1 2 5 1 1 1 2 1 2 2 1 2 2 1 1 3 1 2
[39] 2 2 2 2 2 4 1 3 2 1 1 1 2 1 2 1 1
```
:::
:::


We can find out how many observations are in each cluster by means of the table command. Parenthetially, we can also find this as the dimension of each vector in the lists contained in edges.groups. For example, the first list has node with dimension 12, which is also the number of observations in the first cluster.


::: {.cell}

```{.r .cell-code}
table(ccs6)
```

::: {.cell-output .cell-output-stdout}
```
ccs6
 1  2  3  4  5  6 
22 18 11  2  1  1 
```
:::
:::


Lastly, we can also plot the pruned tree that shows the five clusters on top of the townshop area.


::: {.cell}

```{.r .cell-code}
plot(shan_sp, border=gray(.5))
plot(clust6, 
     coordinates(shan_sp), 
     cex.lab=.7,
     groups.colors=c("red","green","blue", "brown", "pink"),
     cex.circles=0.005, 
     add=TRUE)
```

::: {.cell-output .cell-output-stderr}
```
Warning in segments(coords[id1, 1], coords[id1, 2], coords[id2, 1],
coords[id2, : "add" is not a graphical parameter

Warning in segments(coords[id1, 1], coords[id1, 2], coords[id2, 1],
coords[id2, : "add" is not a graphical parameter

Warning in segments(coords[id1, 1], coords[id1, 2], coords[id2, 1],
coords[id2, : "add" is not a graphical parameter

Warning in segments(coords[id1, 1], coords[id1, 2], coords[id2, 1],
coords[id2, : "add" is not a graphical parameter
```
:::

::: {.cell-output-display}
![](In-class_Exercise3_files/figure-html/unnamed-chunk-53-1.png){width=672}
:::
:::


### Visualising the clusters in choropleth map

The code chunk below is used to plot the newly derived clusters by using SKATER method.


::: {.cell}

```{.r .cell-code}
groups_mat <- as.matrix(clust6$groups)
shan_sf_spatialcluster <- cbind(shan_sf_cluster, as.factor(groups_mat)) %>%
  rename(`SP_CLUSTER`=`as.factor.groups_mat.`)
qtm(shan_sf_spatialcluster, "SP_CLUSTER")
```

::: {.cell-output-display}
![](In-class_Exercise3_files/figure-html/unnamed-chunk-54-1.png){width=672}
:::
:::


For easy comparison, it will be better to place both the hierarchical clustering and spatially constrained hierarchical clustering maps next to each other.


::: {.cell}

```{.r .cell-code}
hclust.map <- qtm(shan_sf_cluster,
                  "CLUSTER") + 
  tm_borders(alpha = 0.5) 

shclust.map <- qtm(shan_sf_spatialcluster,
                   "SP_CLUSTER") + 
  tm_borders(alpha = 0.5) 

tmap_arrange(hclust.map, shclust.map,
             asp=NA, ncol=2)
```

::: {.cell-output .cell-output-stderr}
```
Warning: One tm layer group has duplicated layer types, which are omitted. To
draw multiple layers of the same type, use multiple layer groups (i.e. specify
tm_shape prior to each of them).

Warning: One tm layer group has duplicated layer types, which are omitted. To
draw multiple layers of the same type, use multiple layer groups (i.e. specify
tm_shape prior to each of them).
```
:::

::: {.cell-output-display}
![](In-class_Exercise3_files/figure-html/unnamed-chunk-55-1.png){width=672}
:::
:::


## Spatially Constrained Clustering: ClustGeo Method

In this section, you will gain hands-on experience on using functions provided by **ClustGeo** package to perform non-spatially constrained hierarchical cluster analysis and spatially constrained cluster analysis.

### Ward-like hierarchical clustering: ClustGeo

ClustGeo package provides function called `hclustgeo()` to perform a typical Ward-like hierarchical clustering just like `hclust()` you learned in previous section.

To perform non-spatially constrained hierarchical clustering, we only need to provide the function a dissimilarity matrix as shown in the code chunk below.


::: {.cell}

```{.r .cell-code}
nongeo_cluster <- hclustgeo(proxmat)
plot(nongeo_cluster, cex = 0.5)
rect.hclust(nongeo_cluster, 
            k = 6, 
            border = 2:5)
```

::: {.cell-output-display}
![](In-class_Exercise3_files/figure-html/unnamed-chunk-56-1.png){width=672}
:::
:::


Note that the dissimilarity matrix must be an object of class `dist`, i.e. an object obtained with the function `dist()`.

#### Mapping the clusters formed

Similarly, we can plot the clusters on a categorical area shaded map by using the steps we learned in Mapping the clusters formed.


::: {.cell}

```{.r .cell-code}
groups <- as.factor(cutree(nongeo_cluster, k=6))
```
:::

::: {.cell}

```{.r .cell-code}
shan_sf_ngeo_cluster <- cbind(shan_sf, as.matrix(groups)) %>%
  rename(`CLUSTER` = `as.matrix.groups.`)
```
:::

::: {.cell}

```{.r .cell-code}
qtm(shan_sf_ngeo_cluster, "CLUSTER")
```

::: {.cell-output-display}
![](In-class_Exercise3_files/figure-html/unnamed-chunk-59-1.png){width=672}
:::
:::


### Spatially Constrained Hierarchical Clustering

Before we can performed spatially constrained hierarchical clustering, a spatial distance matrix will be derived by using [`st_distance()`](https://r-spatial.github.io/sf/reference/geos_measures.html) of sf package.


::: {.cell}

```{.r .cell-code}
dist <- st_distance(shan_sf, shan_sf)
distmat <- as.dist(dist)
```
:::


Notice that `as.dist()` is used to convert the data frame into matrix.

Next, `choicealpha()` will be used to determine a suitable value for the mixing parameter alpha as shown in the code chunk below.


::: {.cell}

```{.r .cell-code}
cr <- choicealpha(proxmat, distmat, range.alpha = seq(0, 1, 0.1), K=6, graph = TRUE)
```

::: {.cell-output-display}
![](In-class_Exercise3_files/figure-html/unnamed-chunk-61-1.png){width=672}
:::

::: {.cell-output-display}
![](In-class_Exercise3_files/figure-html/unnamed-chunk-61-2.png){width=672}
:::
:::


With reference to the graphs above, alpha = 0.3 will be used as shown in the code chunk below.


::: {.cell}

```{.r .cell-code}
clustG <- hclustgeo(proxmat, distmat, alpha = 0.3)
```
:::


Next, cutree() is used to derive the cluster objecct.


::: {.cell}

```{.r .cell-code}
groups <- as.factor(cutree(clustG, k=6))
```
:::


We will then join back the group list with shan_sf polygon feature data frame by using the code chun below.


::: {.cell}

```{.r .cell-code}
shan_sf_Gcluster <- cbind(shan_sf, as.matrix(groups)) %>%
  rename(`CLUSTER` = `as.matrix.groups.`)
```
:::


We can not plot the map of the newly delineated spatially constrained clusters.


::: {.cell}

```{.r .cell-code}
qtm(shan_sf_Gcluster, "CLUSTER")
```

::: {.cell-output-display}
![](In-class_Exercise3_files/figure-html/unnamed-chunk-65-1.png){width=672}
:::
:::

