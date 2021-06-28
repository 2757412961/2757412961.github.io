---
title: Landsat卫星数据管理接口设计文档
---

# Landsat卫星数据管理接口设计文档

## 1.0 基本设计

### 在分支branch-landsat上进行开发

https://e.coding.net/Maoshengyan/fuxi.git

以LandsatController为入口，参考TanSatController。

### ElasticSearch 和 tansat数据库对比

```
ES中landsat和gf共有属性（标*为共有字段）:
    *id: String，UUID 唯一标识符
    *boundary: String，geojson或者wkt（es的geo_shpae类型只支持这两种格式）
    *satellite: String，卫星种类
    *sensor: String，传感器种类
    *level: String，数据级别
    *cloud: Integer，云量
    *resolution: Integer，空间分辨率
    *provider: String，提供方
    *quick-look: String，影像缩略图
    *start-time: Long，毫秒时间戳，timestamp
    *end-time: Long，毫秒时间戳，timestamp

landsat特有属性：
	center-point: String，geojson或者wkt（es的geo_shpae类型只支持这两种格式）

gf特有属性：
    bandCount: 为空
    temporalResolution: 为空
    productMode: 为空
    productType: 为空

----------------------------------------------------------------------------------------------
fx_tansat数据库字段（标*表示 和fx_ecology共有字段）：
	*id
    satellite
    sensor
    productmode
    level
    spatial_resolution
    temporal_resolution
    producttype
    *name_chn
    *name_en
    *description
    *starttime
    *endtime
    minx
    maxx
    miny
    maxy
    boundary
    *quicklook
    *md5
    *filepath
    *size
    *filename
    *filetype
    *privilege: 私有 or 共有。
    *loaderinfo
    *usedeclaration
    *createtime
    *updatetime
    *owner
    provider
    *other_properties
    geom
    bandcount
```

### Landsat数据库设计

```
DROP TABLE IF EXISTS "public"."fx_landsat";
CREATE TABLE "public"."fx_landsat"
(
    "id"                  varchar(36)  NOT NULL,
    "name_chn"            varchar(150) DEFAULT NULL:: character varying,
    "name_en"             varchar(150) NOT NULL,
    "description"         text,
    "starttime"           int8,
    "endtime"             int8,
    "quicklook"           text,
    "md5"                 varchar(36)  DEFAULT NULL:: character varying,
    "filepath"            text,
    "size"                int8,
    "filename"            varchar(150) DEFAULT NULL:: character varying,
    "filetype"            varchar(50)  DEFAULT NULL:: character varying,
    "privilege"           varchar(36)  DEFAULT NULL:: character varying,
    "loaderinfo"          text,
    "usedeclaration"      text,
    "createtime"          int8         NOT NULL,
    "updatetime"          int8         NOT NULL,
    "owner"               varchar(36)  DEFAULT NULL:: character varying,
    "other_properties"    text,
    "satellite"           varchar(50)  DEFAULT NULL:: character varying,
    "sensor"              varchar(50)  DEFAULT NULL:: character varying,
    "productmode"         varchar(150) DEFAULT NULL:: character varying,
    "level"               int4,
    "cloud"               int4,
    "spatial_resolution"  varchar(50)  DEFAULT NULL:: character varying,
    "temporal_resolution" varchar(50)  DEFAULT NULL:: character varying,
    "producttype"         varchar(50)  DEFAULT NULL:: character varying,
    "provider"            varchar(50)  NOT NULL,
    "minx"                varchar(100) DEFAULT NULL:: character varying,
    "maxx"                varchar(100) DEFAULT NULL:: character varying,
    "miny"                varchar(100) DEFAULT NULL:: character varying,
    "maxy"                varchar(100) DEFAULT NULL:: character varying,
    "bandcount"           int4,
    "boundary"            text,
    CONSTRAINT "fx_landsat_pkey" PRIMARY KEY ("id")
);
```

## 1.1 v1.0/api/landsat [POST]

### 描述
新建一个landsat数据。先校验数据，校验成功后往数据库插入一条新的landsat卫星数据，并返回所插入数据的ID。其中createTime和updateTime皆为插入的时间戳，owner为当前登录的用户。

- [ ] 1.实体类检验：name_en、provider参数必须。各个字段长度限制。starttime、endtime为毫秒级时间戳。quicklook加UrlPictureConstraint约束。filePath加FilepathConstraint约束。otherProperties格式检验，必须要有key属性。
- [ ] 2.实体类值检验：privilege为[PRIVATE, PUBLIC]中的一种，boundary为wkt或者geojson。
- [ ] 3.业务逻辑检验：name_en唯一性检验，如果privilege是私有的，在用户的自己列表中应该是唯一的，如果privilege是公开的，在公共列表中是唯一的。other_properties唯一性检验。
- [ ] 4.在入库之前，主动添加UUID、createTime、updateTime属性。
- [ ] 5.监听添加、删除、更新数据接口，通过消息队列将数据放入es库中。


### 参数
```json
body:{
    "nameChn": "string",
    "nameEn": "string",
    "description": "string",
    "startTime": "long",
    "endTime": "long",
    "quickLook": "string",
    "MD5": "string",
    "filePath": "string",
    "fileSize": "long",
    "fileName": "string",
    "fileType": "string",
    "privilege": "string",
    "loaderInfo": "string",
    "useDeclaration": "string",
    "otherProperties": [
        {
          "key": "string",
          "value": "string",
        }
    ],
    "satellite": "string",
    "sensor": "string",
    "productMode": "string",
    "level": "integer",
    "cloud": "integer",
    "spatialResolution": "string",
    "temporalResolution": "string",
    "productType": "string",
    "provider": "string",
    "minX": "string",
    "maxX": "string",
    "minY": "string",
    "maxY": "string",    
    "bandCount": "int",
    "boundary": "string"
 }
```
* 参数说明：
```
"nameChn": string, 表示产品中文名, 字符长度200以内, 非必须
"nameEn": string, 表示产品英文名, 字符长度必须1-200以内, 公有数据里唯一, 用户个人数据里也唯一, **必须**
"description": string, 表示数据描述, 字符长度2000以内, 非必须
"startTime": long, 表示开始时间, 时间戳长整数, 非必须
"endTime": long, 表示结束时间, 时间戳长整数, 非必须
"quickLook": string, 表示数据缩略图的url, 符合URL图片格式, 字符长度300以内, 非必须
"MD5": string, 字符长度36以内, 非必须
"filePath": string, 表示数据文件地址, 仅能以fuxi://开头，文件存储在用户工作空间，字符长度300以内, 非必须
"fileSize": long, 表示数据文件大小, 单位Byte，大于等于0, 非必须，默认0
"fileName": string, 表示数据文件名, 字符长度150以内, 非必须
"fileType": string, 表示数据文件类型, 字符长度50以内, 非必须
"privilege": string, 表示数据访问权限, 值在["PUBLIC", "PRIVATE"]内, 不区分大小写, 字符长度36以内, 非必须
"loaderInfo": string, 表示数据关联图层信息, 字符长度200以内, 非必须
"useDeclaration": string, 表示数据的使用申明, 字符长度2000以内, 非必须
"otherProperties": array, 表示数据的其他属性, 属性list, 每个属性由key, value, 非必须:
    "key": string, 字符长度50以内, 不能重复, 必须
    "value": string, 字符长度2000以内, 非必须
"satellite": string, 表示卫星名, 字符长度50以内, 非必须
"sensor": string, 表示传感器, 字符长度50以内, 非必须
"productMode": string, 表示产品模式, 字符长度150以内, 非必须
"level": integer, 表示级别, 非必须
"cloud": integer, 表示云量, 非必须
"spatialResolution": string, 表示空间分辨率, 字符长度50以内, 非必须
"temporalResolution": string, 表示事件分辨率, 字符长度50以内, 非必须
"productType": string, 表示产品类型, 字符长度50以内, 非必须
"provider": string, 表示数据提供者, 字符长度必须1-50以内, **必须**
"minX": string, 表示x轴最小坐标, 字符长度100以内, 非必须
"maxX": string, 表示x轴最大坐标, 字符长度100以内, 非必须
"minY": string, 表示y轴最小坐标, 字符长度100以内, 非必须
"maxY": string, 表示y轴最大坐标, 字符长度100以内, 非必须
"bandCount": integer, 表示影像的波段数，非必须
"boundary": string, 表示数据边界，geojson或者wkt格式, 由点串组成的多边形范围, 字符长度20MB(20*10^6)以内, 非必须
```
* 输入示例:
```json
{
    "nameChn": "大气二氧化碳柱平均干空气混合比-2017年4月13日",
    "nameEn": "landsat_L2SCI_13",
    "description": "大气二氧化碳柱平均干空气混合比-2017年4月13日",
    "startTime": 1410365241084,
    "endTime": 1510365241084,
    "quickLook": "http://example.org/1.png",
    "MD5": null,
    "filePath": "表示数据文件地址, 仅能以fuxi://开头",
    "fileSize": 30,
    "fileName": "landsat_L2SCI_13.nc",
    "fileType": "NetCDF",
    "privilege": "public",
    "loaderInfo": "表示数据关联图层信息",
    "useDeclaration": "表示数据的使用申明",
    "otherProperties": [
        {
          "key": "key1",
          "value": ""
        }
    ],    
    "satellite": "landsat",
    "sensor": "ACGS",
    "productMode": "SCI",
    "level": 2,
    "cloud": 50,
    "spatialResolution": "0.25m",
    "temporalResolution": "1d",
    "productType": "XCO2",
    "provider": "ChinaGeo",
    "minX": "-180",
    "maxX": "180",
    "minY": "-84",
    "maxY": "84",    
    "bandCount": 7,
    "boundary": "POLYGON ((110.1313499 30.282259, 120.1315429 30.2796326, 120.1332501 30.2815018, 120.1331751 30.2823548, 110.1313499 30.282259))",
 }
```
### 返回
1.插入成功，Status: 201。返回所插入数据的UUID。
```json
{
    "landsatId": "a081ffc4-d080-4aab-ac45-8de9bf977c71"
}
```
2.插入失败，Status: 400，参数校验失败，存在以下几种情况：
(1) 必填参数【provider, nameEn】缺失，如provider缺失：
```json
{
    "code": 400,
    "message": "provider cannot be null"
}
```
(2) 必填参数【provider, nameEn】的字符长度不足1，如provider为""：
```json
{
    "code": 400,
    "message": "provider length should be more than 1 and no more than 50"
}
```
(3) 参数的字符长度超过最大限制，如satellite参数超过长度限制50：
```json
{
    "code": 400,
    "message": "satellite length should be no more than 50"
}
```
(4) 参数privilege值不为["PUBLIC", "PRIVATE"]内，如为"public_1"：
```json
{
    "code": 400,
    "message": "privilege should be in [PRIVATE, PUBLIC]"
}
```
(5) 参数otherProperties里有属性的key是空
```json
{
    "code": 400,
    "message": "key cannot be null"
}
```
(6) 参数otherProperties里有属性的key是""
```json
{
    "code": 400,
    "message": "key length should be no more than 50"
}
```
(7) 参数otherProperties里有参数的key重复
```json
{
    "code": 400,
    "message": "the key key1 in otherProperties is not unique"
}
```

(8) 参数quickLook格式不对

```json
{
    "code": 400,
    "message": "Invalid url picture name"
}
```

(9) 参数filepath格式不对

```json
{
    "code": 400,
    "message": "Invalid filepath"
}
```

## 1.2 v1.0/api/landsat/{id} [GET]

###  描述
根据id查询landsat卫星数据。需要权限校验，只能查自己的或者公开的数据，会检查id的长度，以及数据是否存在。

###  参数
```json
path{
    "id": "string"
}
```
* 参数说明：
```
"id": string, 表示数据id, 字符长度36，必须
```
* 输入示例:
```
url:/v1.0/api/landsat/07e0d3cd-92c9-4b61-adaa-fcffa7c6830d
```
###  返回
1.查询成功，Status: 200。
```json
{
    "id": "78d63c08-29c9-47ac-a639-8a9fbbdb957b",
    "satellite": "landsat",
    "sensor": "ACGS",
    "productMode": "SCI",
    "level": 2,
    "spatialResolution": "0.25m",
    "temporalResolution": "1d",
    "productType": "XCO2",
    "nameChn": "大气二氧化碳柱平均干空气混合比-2017年4月13日",
    "nameEn": "landsat_L2SCI_13",
    "description": "大气二氧化碳柱平均干空气混合比-2017年4月13日",
    "startTime": 1,
    "endTime": 2,
    "bandCount": 7,
    "minX": "-180",
    "maxX": "84",
    "minY": "-84",
    "maxY": "84",
    "quickLook": "http://example.org/1.png",
    "fileSize": 30,
    "fileName": "landsat_L2SCI_13.nc",
    "fileType": "NetCDF",
    "privilege": "public",
    "createTime": 1559725309604,
    "updateTime": 1559725309604,
    "owner": "",
    "provider": "ChinaGeo",
    "otherProperties": [
        {
            "key": "key1",
            "value": ""
        }
    ]
}
```
2.查询失败，Status: 400，参数校验失败，存在以下几种情况：
(1) id长度不为36，比如id为only5：
```json
{
    "code": 400,
    "message": "getlandsat.arg1: id length should be 36"
}
```
3.查询失败，Status: 401，权限不足
(1) 当前用户无法访问该id的数据。比如用户未登录下访问自己的私有数据，登录下访问他人私有数据等：
```json
{
    "code": 401,
    "message": "Permission denied, don't have privilege to do this action."
}
```
4.查询失败，Status: 404，找不到数据
(1) 数据不存在：

```json
{
    "code": 404,
    "message": "landsat not found with id : '07e0d3cd-92c9-4b61-adaa-fcffa7c68301'"
}
```

## 1.3 v1.0/api/landsat [PUT]
### 描述
更新一个landsat卫星数据。先校验数据，校验成功后往数据库更新landsat卫星数据，并返回所更新数据的ID。其中createTime、owner和boundary不允许修改，updateTime为更新的时间戳。用户只能更新自己的公开数据，管理员可以更新用户数据集中公开数据。
### 参数
```json
body:{
    "id": "string",
    "satellite": "string",
    "sensor": "string",
    "productMode": "string",
    "level": "integer",
    "spatialResolution": "string",
    "temporalResolution": "string",
    "productType": "string",
    "nameChn": "string",
    "nameEn": "string",
    "description": "string",
    "startTime": "long",
    "endTime": "long",
    "bandCount": "int",
    "minX": "string",
    "maxX": "string",
    "minY": "string",
    "maxY": "string",
    "boundary": "string",
    "quickLook": "string",
    "MD5": "string",
    "filePath": "string",
    "fileSize": "long",
    "fileName": "string",
    "fileType": "string",
    "privilege": "string",
    "loaderInfo": "string",
    "useDeclaration": "string",
    "provider": "string",
    "otherProperties": [
        {
          "key": "string",
          "value": "string",
        }
    ]
 }
```
* 参数说明：
```
"id": string, 表示数据id, 字符长度36, 必须
"satellite": string, 表示卫星名, 字符长度50以内, 非必须
"sensor": string, 表示传感器, 字符长度50以内, 非必须
"productMode": string, 表示产品模式, 字符长度150以内, 非必须
"level": integer, 表示级别, 非必须````````
"spatialResolution": string, 表示空间分辨率, 字符长度50以内, 非必须
"temporalResolution": string, 表示事件分辨率, 字符长度50以内, 非必须
"productType": string, 表示产品类型, 字符长度50以内, 非必须
"nameChn": string, 表示产品中文名, 字符长度200以内, 非必须
"nameEn": string, 表示产品英文名, 字符长度必须1-200以内, 公有数据里唯一, 用户个人数据里也唯一, **必须**
"description": string, 表示数据描述, 字符长度2000以内, 非必须
"startTime": long, 表示开始时间, 时间戳长整数, 非必须
"endTime": long, 表示结束时间, 时间戳长整数, 非必须
"bandCount": int, 表示影像的波段数，非必须
"minX": string, 表示x轴最小坐标, 字符长度100以内, 非必须
"maxX": string, 表示x轴最大坐标, 字符长度100以内, 非必须
"minY": string, 表示y轴最小坐标, 字符长度100以内, 非必须
"maxY": string, 表示y轴最大坐标, 字符长度100以内, 非必须
"boundary": string, 表示数据边界, 由点串组成的多边形范围, 字符长度20MB(20*10^6)以内, 非必须
"quickLook": string, 表示数据缩略图的url, 符合URL图片格式, 字符长度300以内,非必须
"MD5": string, 字符长度36以内, 非必须
"filePath": string, 表示数据文件地址, 字符长度300以内, 非必须
"fileSize": long, 表示数据文件大小, 单位Byte，大于等于0, 非必须，默认0
"fileName": string, 表示数据文件名, 字符长度150以内, 非必须
"fileType": string, 表示数据文件类型, 字符长度50以内, 非必须
"privilege": string, 表示数据访问权限, 值在["PUBLIC", "PRIVATE"]内, 不区分大小写, 字符长度36以内, 非必须, **默认不修改**
"loaderInfo": string, 表示数据关联图层信息, 字符长度200以内, 非必须
"useDeclaration": string, 表示数据的使用申明, 字符长度2000以内, 非必须
"provider": string, 表示数据提供者, 字符长度必须1-50以内, **必须**
"otherProperties": array, 表示数据的其他属性, 属性list, 每个属性由key, value, 非必须:
    "key": string, 字符长度50以内, 不能重复, 必须
    "value": string, 字符长度2000以内, 非必须
```
* 输入示例:
```json

```
### 返回
1.更新成功，Status: 200。返回所更新数据的UUID。
```json
{
    "landsatId": "1146ff8f-9a9c-4b0e-9b33-87e701458b36"
}
```
2.更新失败，Status: 400，参数校验失败，存在以下几种情况：
(1) 必填参数【provider, nameEn】缺失，如provider缺失：

```json
{
    "code": 400,
    "message": "provider cannot be null"
}
```
(2) 必填参数【provider, nameEn】的字符长度不足1，如provider为""：
```json
{
    "code": 400,
    "message": "provider length should be more than 1 and no more than 50"
}
```
(3) 参数的字符长度超过最大限制，如satellite参数超过长度限制50：
```json
{
    "code": 400,
    "message": "satellite length should be no more than 50"
}
```
(4) 参数privilege值不为["PUBLIC", "PRIVATE"]内，如为"public_1"：
```json
{
    "code": 400,
    "message": "privilege should be in [PRIVATE, PUBLIC]"
}
```
(5) 参数otherProperties里有属性的key是空
```json
{
    "code": 400,
    "message": "key cannot be null"
}
```
(6) 参数otherProperties里有属性的key长度超过50
```json
{
    "code": 400,
    "message": "key length should be no more than 50"
}
```
(7) 参数otherProperties里有参数的key重复
```json
{
    "code": 400,
    "message": "the key key1 in otherProperties is not unique"
}
```
(8) id长度不为36，比如id为only5：
```json
{
    "code": 400,
    "message": "id length should be 36"
}
```
3.更新失败，Status: 401，权限不足
(1) 当前用户无法更新该id的数据。比如用户未登录下更新自己的私有数据，登录下更新他人数据等：
```json
{
    "code": 401,
    "message": "Permission denied, don't have privilege to do this action."
}
```
4.更新失败，Status: 404，找不到数据
(1) 数据不存在：

```json
{
    "code": 404,
    "message": "landsat not found with id : '07e0d3cd-92c9-4b61-adaa-fcffa7c68301'"
}
```

## 1.4 v1.0/api/landsat/{id} [DELETE]
###  描述
根据id删除landsat卫星数据。需要权限校验，会检查id的长度，以及数据是否存在。用户只能删除自己的landsat卫星数据，管理员可以删除用户数据集中公开数据。

###  参数
```json
path{
    "id": "string"
}
```
* 参数说明：
```
"id": string, 表示数据id, 字符长度36，必须
```
* 输入示例:
```
url:/v1.0/api/landsat/07e0d3cd-92c9-4b61-adaa-fcffa7c6830d
```
###  返回
1.删除成功，Status: 200。
2.删除失败，Status: 400，参数校验失败，存在以下几种情况：
(1) id长度不为36，比如id为only5：
```json
{
    "code": 400,
    "message": "droplandsat.arg1: id length should be 36"
}
```
3.删除失败，Status: 401，权限不足
(1) 当前用户无法删除该id的数据。比如用户未登录下删除自己的私有数据，登录下删除他人数据等：
```json
{
    "code": 401,
    "message": "Permission denied, don't have privilege to do this action."
}
```
4.删除失败，Status: 404，找不到数据
(1) 数据不存在：

```json
{
    "code": 404,
    "message": "landsat not found with id : '07e0d3cd-92c9-4b61-adaa-fcffa7c68301'"
}
```

