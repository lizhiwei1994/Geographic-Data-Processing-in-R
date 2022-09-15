
<!-- README.md is generated from README.Rmd. Please edit that file -->

# R语言地理信息数据处理

## 含经纬度的点数据转换成地图栅格数据

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

将转换结果保存为`tif`文件。

``` r
writeRaster(rs,"result/rs.tif", overwrite = T)
```

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

使用`extract()`提取`lon = 116.35`和`lat = 39.45`位置下`PM2.5`的浓度。

``` r
extract(dat, mypt)
#>       rs
#> [1,] 165
```

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
