
<!-- README.md is generated from README.Rmd. Please edit that file -->

# R语言地理信息数据处理

李志伟 (<lizhiwei@ccmu.edu.cn>)

首都医科大学公共卫生学院，流行病和卫生统计学系

北京市丰台区右安门外西头条10号

Zhiwei Li

Department of Epidemiology and Health Statistics

School of Public Health, Capital Medical University

No.10 Xitoutiao, Youanmen Wai Street

Beijing, 100069

*本教程所有出现的代码可以在`code/code.r`文件中找到。*
*生成此教程的代码可以在`README.Rmd`文件中找到。*

## 含经纬度的点数据转换成地图栅格数据

### 经纬度点数据加载

首先加载含经纬度的点数据。

``` r
pts = read.csv('data/20040101_PM25_and_species.csv')
head(pts)
#>    X_Lon Y_Lat PM2.5
#> 1 116.35 39.45   165
#> 2 115.75 39.55    97
#> 3 116.25 39.55   173
#> 4 116.35 39.55   172
#> 5 116.45 39.55   165
#> 6 115.55 39.65    73
```

### 点数据到面数据

转换的过程需要使用`raster`包。

``` r
library(raster)
#> 载入需要的程辑包：sp
```

使用`raster`包中的`rasterFromXYZ()`函数可以将含经纬度的点数据转换成地图栅格数据。

``` r
rs = raster::rasterFromXYZ(pts)
```

使用`plot()`函数查看转换结果。

``` r
plot(rs)
```

![](README_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

### 添加坐标系

查看`rs`文件的坐标系。

``` r
crs(rs)
#> Coordinate Reference System:
#> Deprecated Proj.4 representation: NA
#> Warning in wkt(x): CRS object has no comment
```

转换的`rs`文件没有坐标系，我们手动给他设置一个。
根据[这个网站](https://gis.stackexchange.com/questions/111226/how-to-assign-crs-to-rasterlayer-in-r)的介绍，使用`'+init=EPSG:4326'`可以更容易适应其他参考系统。

> The result is the same, but this version is easier adaptable to other
> reference systems: `crs(r) <- CRS('+init=EPSG:4326')` if you know the
> EPSG number.

``` r
crs(rs) <- CRS('+init=EPSG:4326')
```

再次查看`rs`文件的坐标系。

``` r
crs(rs)
#> Coordinate Reference System:
#> Deprecated Proj.4 representation: +proj=longlat +datum=WGS84 +no_defs 
#> WKT2 2019 representation:
#> GEOGCRS["WGS 84",
#>     DATUM["World Geodetic System 1984",
#>         ELLIPSOID["WGS 84",6378137,298.257223563,
#>             LENGTHUNIT["metre",1]],
#>         ID["EPSG",6326]],
#>     PRIMEM["Greenwich",0,
#>         ANGLEUNIT["degree",0.0174532925199433],
#>         ID["EPSG",8901]],
#>     CS[ellipsoidal,2],
#>         AXIS["longitude",east,
#>             ORDER[1],
#>             ANGLEUNIT["degree",0.0174532925199433,
#>                 ID["EPSG",9122]]],
#>         AXIS["latitude",north,
#>             ORDER[2],
#>             ANGLEUNIT["degree",0.0174532925199433,
#>                 ID["EPSG",9122]]],
#>     USAGE[
#>         SCOPE["unknown"],
#>         AREA["World."],
#>         BBOX[-90,-180,90,180]]]
```

### 结果保存为tif文件

将转换结果保存为`tif`文件。

``` r
writeRaster(rs,"result/rs.tif", overwrite = T)
```

### 结果验证

接下来我们验证导出的`rs.tif`文件能否通过经纬度，提取到`PM2.5`的浓度。
我们提取`pts`第一行的数据。原始数据如下：

``` r
head(pts,1)
#>    X_Lon Y_Lat PM2.5
#> 1 116.35 39.45   165
```

清空我们的环境。

``` r
rm(list = ls())
```

导入刚刚保存的`rs.tif`文件。

``` r
dat = brick('result/rs.tif')
```

将`pts`中的第一行经纬度保存成数据框`mypt`。

``` r
mypt = data.frame(lon = 116.35, lat = 39.45)
```

> 注意：经度必须用`lon`表示，纬度必须用`lat`表示。

`mypt`转换为`sp`格式。

``` r
mypt = SpatialPoints(mypt, proj4string=CRS("+proj=longlat +datum=WGS84"))
class(mypt)
#> [1] "SpatialPoints"
#> attr(,"package")
#> [1] "sp"
```

#### extract()提取单个经纬度下的值

使用`extract()`提取`lon = 116.35`和`lat = 39.45`位置下`PM2.5`的浓度。

``` r
extract(dat, mypt)
#>       rs
#> [1,] 165
```

#### extract()提取单个经纬度下的值

`extract()`函数还能一次提取多个经纬度的数据。

创建包含3个经纬度的数据框。并转换为`sp`格式。

``` r
mypt2 = data.frame(lon = c(116.35, 115.75, 116.25), lat = c(39.45, 39.55, 39.55))
mypt2
#>      lon   lat
#> 1 116.35 39.45
#> 2 115.75 39.55
#> 3 116.25 39.55
```

转换为`sp`格式。

``` r
mypt2 = SpatialPoints(mypt2, proj4string=CRS("+proj=longlat +datum=WGS84"))
```

使用`extract()`函数提取`mypt2`中经纬度下`PM2.5`的浓度。

``` r
extract(dat, mypt2)
#>       rs
#> [1,] 165
#> [2,]  97
#> [3,] 173
```

## 从netCDF4(nc)文件中提取特定坐标下的值

### 预处理

> 注意：以下清空环境的操作会让你R中的所有对象和R包都清空！！！所以在执行以下操作前，请确保所有重要的数据都已保存。或者你可以重新打开一个新的Rstudio来运行下边的代码。

为了防止不同包中的函数冲突，我们需要清空环境。

首先运行:

``` r
rm(list = ls())
```

其次同时按住`Ctrl+Shift+F10`重启Rstudio。

### netCDF4文件简介

`netCDF4`文件的后缀是`.nc`。关于什么是`netCDF4`文件，我不打算做过多介绍。更详细的介绍以及如何在R中处理`netCDF4`文件可以从[这个网站](https://pjbartlein.github.io/REarthSysSci/netCDF.html)找到。

### nc数据加载

本次使用的数据`cru10min30_tmp.nc`文件来自[Climate Research
Unit](http://www.cru.uea.ac.uk/data)。包括0.5度网格上近地面空气温度的长期平均值（1961-1990年）。阵列的尺寸为`720`（经度）x`360`（纬度）x
`12`（月份），从而形成一个`12`层的堆叠nc文件。

使用`raster`包读取`nc`文件。

``` r
library(raster)
mync = brick('data/cru10min30_tmp.nc')
#> 载入需要的名字空间：ncdf4
```

### nc数据查看

查看`mync`的维度。

``` r
dim(mync)
#> [1] 360 720  12
```

`mync`有`360`个纬度，`720`个经度和`12`个月。它在`nc`文件中分布情况类似于这样(图片仅为示意图，不代表`mync`的实际数据情况。点击图片，查看图片出处)。

[![](images/nc维度示意图.png)](https://towardsdatascience.com/how-to-crack-open-netcdf-files-in-r-and-extract-data-as-time-series-24107b70dcd)

由于`mync`有`12`个月的温度数据，所有我们在提取某个坐标下的温度时候应该会有`12`个结果。

### 提取单个经纬度下的值在nc文件中

我们先生成一个坐标，并提取该坐标下的温度数据。

``` r
mypt = data.frame(lon = 116.35, lat = 39.45)
mypt = SpatialPoints(mypt, proj4string=CRS("+proj=longlat +datum=WGS84")) # 将mypt转换为sp
```

使用`extract()`函数提取`mypt`坐标下的温度数据。

``` r
extract(mync, mypt)
#>      X1976.01.16 X1976.02.15 X1976.03.16 X1976.04.16 X1976.05.16 X1976.06.16
#> [1,]        -3.4        -0.9           6        14.2        20.5          25
#>      X1976.07.16 X1976.08.16 X1976.09.16 X1976.10.16 X1976.11.16 X1976.12.16
#> [1,]        26.8        25.6        20.7          14         5.6        -1.3
```

可以看到提取出的数据有`1`行`12`列，分别对应`1`个坐标下的`12`个月的平均温度。

``` r
dim(extract(mync, mypt))
#> [1]  1 12
```

### 提取多个经纬度下的值在nc文件中

我们再试试提取多个坐标下的温度数据。

``` r
mypt2 = data.frame(lon = c(116.35, 115.75, 116.25), lat = c(39.45, 39.55, 39.55))
mypt2 = SpatialPoints(mypt2, proj4string=CRS("+proj=longlat +datum=WGS84")) # 将mypt2转换为sp

extract(mync, mypt2)
#>      X1976.01.16 X1976.02.15 X1976.03.16 X1976.04.16 X1976.05.16 X1976.06.16
#> [1,]        -3.4        -0.9         6.0        14.2        20.5        25.0
#> [2,]        -7.6        -4.9         2.1        10.5        17.2        21.4
#> [3,]        -4.0        -1.5         5.4        13.7        20.1        24.5
#>      X1976.07.16 X1976.08.16 X1976.09.16 X1976.10.16 X1976.11.16 X1976.12.16
#> [1,]        26.8        25.6        20.7        14.0         5.6        -1.3
#> [2,]        23.4        21.9        16.7        10.0         1.4        -5.5
#> [3,]        26.5        25.1        20.1        13.3         4.9        -1.9
```

查看数据维度可以看到有`3`行`12`列，分别对应`3`个坐标下`12`个月的温度数据。

``` r
dim(extract(mync, mypt2))
#> [1]  3 12
```

## 堆叠netCDF4(nc)文件

未完待续…
