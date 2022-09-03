---
title: postgis 相关调研
categories: 
- 调研
tags:
- 地图服务
- OGC
date: 2021-07-04
---

# 基本概念

## 相关标准化组织机构介绍

- OGC 表示开放地理空间信息联盟 （Open Geospatial Consortium-OGC） ，致力于提供地理信息行业软件和数据及服务的标准化工作。OGC在1994年到2004年期间机构名为Open GIS Consortium， 后因业务需要更名。

- IS0/TC 211(国际标准化组织地理信息技术委员会211)

- W3C(World Wide Web onsortium，万维网联盟)

- OpenGIS(Open Geodata Interoperation Specification,开放地理数据互操作规范)
- GML（Geographic Markup Language）由OGC定义的XML（标准通用标记语言的子集）格式，用来表达地理信息要素。

## OpenGIS

要素（Feature）：几何信息和属性信息

- OpenGIS定义了一组基于数据的服务，而数据的基础是要素（Feature)。所谓要素，简单地说就是一个独立的对象，在地图中可能表现为一个多边形建筑物，在数据库中即是一个独立的条目。要素具有两个必要的组成部分——几何信息和属性信息。

几何信息：点（Point）、边缘（LineString）、面（Polygon）和几何集合（GeometryCollection）

- OpenGIS将几何信息分为点、边缘、面和几何集合四种： 其中这里熟悉的线（LineString)属于边缘的一个子类，而多边形（Polygon)是面的一个子类。也就是说OpenGIS定义的几何类型并不仅仅是我们常见的点、线、多边形三种，它提供了更复杂更详细的定义，增强了未来的可扩展性。另外，几何类型的设计中采用了组合模式（Composite)，将几何集合（GeometryCollection)也定义为一种几何类型。类似地，要素集合（FeatureCollection)也是一种要素。

属性信息（FeatureType）：

- 属性信息没有做太多的限制，可以在实际应用中结合具体的实现进行设置。相同的几何类型、属性类型的组合成为要素类型（FeatureType)，类型相同的要素可以存放在一个数据源中。而一个数据源只能拥有一个要素类型。因此，可以用要素类型来描述一组属性相似的要素。

在面向对象的模型中，完全可以把要素类型理解为一个类，而要素则是类的实例。通过GIS中间件可以从数据源中取出数据，供WMS服务器和WFS服务器使用。WMS服务器接收请求，根据请求内容的不同，可以返回不同格式的最终数据。例如，WMS可以返回常用图片格式的地图片段供最终用户阅读（类似GoogleMaps)，其中地图是根据一个样式文件(SLD)生成的，它描述了地图的线的宽度、色彩等；WMS也可以返回GeoRSS和KML用来与其他地图服务互通。WFS服务器也可以接收请求，但WFS将返回GML格式的地理信息数据。GML是一种基于XML的数据格式，它可以完整地再现数据，也是OpenGIS数据源的重要形式。也就是说，WFS返回的GML可以继续作为数据源。在WFS请求中，OpenGIS定义了一个Filter标准，用来实现对数据的筛选，使WFS更加灵活。另一方面，WFS还支持通过WFS-t提交客户端对数据的修改。通俗地说，WMS是只读的，而WFS则是可以读写的。



## OGC地图服务标准介绍

> OGC1999年开始 WMT1（Web Map Tested）和 WMT2 互操作项目。其中著名的GML来自WMT1的成果。在WMT2中OGC定义了三种地理参考信息模型：Web Map Server(WMS) , Web Feature Server(WFS) ，Web Coverage Server(WCS) .
>
> OGC 地图服务协议，包括 WMS、WFS、WCS、WMTS、WPS 。其中比较重要的现在用得比较多的标准是GML、WMS和WFS。

### 网络地图服务（WMS）

- Web Map Server（WMS）能够根据用户的请求返回相应的地图（包括PNG，GIF，JPEG等栅格形式或者是SVG和WEB CGM等矢量形式）。

- WMS支持网络协议HTTP，所支持的操作是由URL定义的。有三个重要操作 `GetCapabilities` ， `GetMap` ， `GetFeatureinfo` 。

###  网络要素服务（WFS）

- Web 要素服务（WFS）支持对地理要素的插入，更新，删除，检索和发现服务。该服务根据HTTP客户请求返回GML数据。

- 其基础接口是：GetCapabilities，DescribeFeatureType，GetFeature　 GetCapabilities同上。

### 网络覆盖服务（WCS）

- Web地理覆盖服务（WCS）：提供的是包含了地理位置信息或属性的空间栅格图层，而不是静态地图的访问。

- 根据HTTP客户端要求发送相应数据，包括影像，多光谱影像和其它科学数据. 有二个重要操作GetCapabilities，GetCoverage GetCapabilities返回一个描述服务和XML文档，从中可获取覆盖的数据集合。

### 切片地图服务（TMS）

- 切片地图服务（TMS）定义了一些操作，这些操作允许用户访问切片地图。WMTS可能是OGC首个支持RESTful访问的服务标准。

### WMS和WNTS区别

- WMTS服务和WMS服务对客户端请求服务的响应不同，比如在接受客户端请求WMTS服务时，返回给客户端是固定大小的瓦片，客户端根据索引号来获取每一张瓦片，而后拼接成地图进行展示，如图1所示；由于瓦片的规则是固定的，服务端可以预先缓存对应的瓦片，客户端需要时直接返回即可，因而WMTS是可缓存的。

![img](https://raw.githubusercontent.com/2757412961/TyporaImageRepository/master/img/fig-wmts-server.png)















# END