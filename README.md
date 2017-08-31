django geo应用教程
==========

`参考资料：https://docs.djangoproject.com/en/1.11/ref/contrib/gis/tutorial/`

介绍
-------
GeoDjango是Django的一个包含的contrib模块，是一个世界范围地理Web构件。主要有以下特点：


1. django model 式的[ogc](http://www.opengeospatial.org/)数据
2. 使用django orm 操作和访问原始数据
3. 高级封装的python api操作gis等数据的不同数据形式
4. 从admin编辑地理数据

set up
=======
创建一个空间数据库
--------------
通常没有什么特别的set up，就像普通工程那样搭建数据库

* [安装PostGIS](https://docs.djangoproject.com/en/1.11/ref/contrib/gis/install/postgis/)
* [安装SpatiaLite](https://docs.djangoproject.com/en/1.11/ref/contrib/gis/install/spatialite/)

创建一个新工程文件
-----------
用django-amin创建新工程
`$ django-admin startproject geodjango`

然后创建一个新app

```
$ cd geodjango
$ python manage.py startapp world
```

设置settings.py
-------
编辑settings.py里面的数据库设置：

```python
DATABASES = {
    'default': {
         'ENGINE': 'django.contrib.gis.db.backends.spatialite',
         'NAME': 'test.db',
    },
}
```

另外还要修改INSTALLED_APPS,包含`django.contrib.admin`,` django.contrib.gis`和`world`.

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'django.contrib.gis',
    'world',
]
```

Geographic Data
=======

世界边界
----------
世界边界数据保存在一个[zip file](http://thematicmapping.org/downloads/TM_WORLD_BORDERS-0.3.zip).下载并解压

```
$ mkdir world/data
$ cd world/data
$ wget http://thematicmapping.org/downloads/TM_WORLD_BORDERS-0.3.zip
$ unzip TM_WORLD_BORDERS-0.3.zip
$ cd ../..
```

这个zip文件包含有一系列[ESRI Shapefile](https://en.wikipedia.org/wiki/Shapefile)的文件，一种最出名的地理数据格式。解包后这个文件存在4个文件：

* .shp:保存世界边界的向量数据
* .shx:空间索引文件
* .dbf:保存非地理信息内容的数据库文件（int 和 char）
* .prj:包含保存在shapefile里面空间指向信息

使用ogrinfo测试空间数据
----------
GDAL 中的ogrinfo可以用来检测shapefile里面的数据

```
$ ogrinfo world/data/TM_WORLD_BORDERS-0.3.shp
INFO: Open of `world/data/TM_WORLD_BORDERS-0.3.shp'
      using driver `ESRI Shapefile' successful.
1: TM_WORLD_BORDERS-0.3 (Polygon)
```

ogrinfo 告诉我们这个shapefile只包含一层。你可以通过-so 选项获取更多的数据。

```
$ ogrinfo -so world/data/TM_WORLD_BORDERS-0.3.shp TM_WORLD_BORDERS-0.3
INFO: Open of `world/data/TM_WORLD_BORDERS-0.3.shp'
      using driver `ESRI Shapefile' successful.

Layer name: TM_WORLD_BORDERS-0.3
Geometry: Polygon
Feature Count: 246
Extent: (-180.000000, -90.000000) - (180.000000, 83.623596)
Layer SRS WKT:
GEOGCS["GCS_WGS_1984",
    DATUM["WGS_1984",
        SPHEROID["WGS_1984",6378137.0,298.257223563]],
    PRIMEM["Greenwich",0.0],
    UNIT["Degree",0.0174532925199433]]
FIPS: String (2.0)
ISO2: String (2.0)
ISO3: String (3.0)
UN: Integer (3.0)
NAME: String (50.0)
AREA: Integer (7.0)
POP2005: Integer (10.0)
REGION: Integer (3.0)
SUBREGION: Integer (3.0)
LON: Real (8.3)
LAT: Real (7.3)
```

地理模型
=======
定义一个geographic model
----------

```python
from django.contrib.gis.db import models

class WorldBorder(models.Model):
    # Regular Django fields corresponding to the attributes in the
    # world borders shapefile.
    name = models.CharField(max_length=50)
    area = models.IntegerField()
    pop2005 = models.IntegerField('Population 2005')
    fips = models.CharField('FIPS Code', max_length=2)
    iso2 = models.CharField('2 Digit ISO', max_length=2)
    iso3 = models.CharField('3 Digit ISO', max_length=3)
    un = models.IntegerField('United Nations Code')
    region = models.IntegerField('Region Code')
    subregion = models.IntegerField('Sub-Region Code')
    lon = models.FloatField()
    lat = models.FloatField()

    # GeoDjango-specific: a geometry field (MultiPolygonField)
    mpoly = models.MultiPolygonField()

    # Returns the string representation of the model.
    def __str__(self):              # __unicode__ on Python 2
        return self.name
```



