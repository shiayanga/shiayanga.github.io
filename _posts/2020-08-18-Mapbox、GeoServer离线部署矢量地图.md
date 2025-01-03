---
layout: post
title: Mapbox、GeoServer离线部署矢量地图
subtitle: 
author: Shiayanga
categories: [GIS]
banner:
tags: [GIS,GeoServer]
sidebar: []
excerpt: ""
---


关键词：Mapbox、GeoServer、Tomcat、PostgreSQL、PostGis
## 一、地图数据获取

使用OpenStreetMap获取中国的矢量地图数据
![img_2.png](../assets/images/img_2.png)
![img_1.png](../assets/images/img_1.png)
## 二、安装GeoServer及Vector Tiles扩展
![img_3.png](../assets/images/img_3.png)
![img_4.png](../assets/images/img_4.png)
**将下载好的GeoServer.war放入Tomcat，启动Tomcat后将Vector Tiles扩展中的四个jar包放入GeoServer的lib目录并重启Tomcat。**

## 三、安装PostgreSQL及PostGis扩展
略
## 四、将shp文件导入PostgreSQL
![img_5.png](../assets/images/img_5.png)
**SRID那一列选择4326**

## 五、创建工作区、数据存储
打开GeoServer的web页面，默认用户名admin，密码geoserver
![img_6.png](../assets/images/img_6.png)
![img_7.png](../assets/images/img_7.png)
![img_8.png](../assets/images/img_8.png)

## 六、发布图层、创建图层组
1. 发布图层
   ![img_9.png](../assets/images/img_9.png)
2. 发布图层时，“定义SRS”位置选择“EPSG:4326”
   ![img_10.png](../assets/images/img_10.png)
   ![img_11.png](../assets/images/img_11.png)
2. 创建图层组
   ![img_12.png](../assets/images/img_12.png)
   添加图层并生成边界
   ![img_13.png](../assets/images/img_13.png)
   添加完所需图层后生成边界
   ![img_14.png](../assets/images/img_14.png)

## 七、设置矢量切片格式
勾选application/x-protobuf;type=mapbox-vector矢量切片格式，切片为.dbf格式的文件，压缩率更好。适合网络传输。选择默认切片格式EPSG:900913，因为Mapbox只支持WGS84 Web 墨卡托投影投影
![img_15.png](../assets/images/img_15.png)
若此处无“application/x-protobuf;type=mapbox-vector“选项，请先添加Vector Tiles扩展。
## 八、离线部署Mapbox所需的字体、图标等
[离线部署Mapbox之.pbf字体库](https://www.cnblogs.com/ATtuing/p/9217029.html)

[离线部署Mapbox之sprite大图图标文件生成](https://www.cnblogs.com/ATtuing/p/9273391.html)


## 九、Mapbox获取地图
1. 点击左上角GeoServer图标回到首页，点击右侧tms
   ![img_16.png](../assets/images/img_16.png)
2. 复制tms服务路径
   ![img_17.png](../assets/images/img_17.png)
3. Mapbox自定义样式中填入tms地址，定义好样式就可以了
   ![img_18.png](../assets/images/img_18.png)


参考：[ATtuing的博客](https://www.cnblogs.com/ATtuing/)
