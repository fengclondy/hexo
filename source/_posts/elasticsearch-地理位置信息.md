---
layout: post
title: elasticsearch高级篇 --- 地理位置
date: 2018-03-27 09:52:15
tags: elasticsearch
categories: elasticsearch
---


### 地理坐标
>地理坐标点不能被动态映射自动检测，需要显式声明对应字段类型为`geo-point`
```
PUT /attractions
{
  "mappings": {
    "restaurant": {
      "properties": {
        "name": {
          "type": "text"
        },
        "location": {
          "type": "geo_point"
        }
      }
    }
  }
}
```
 经纬度信息的形式可以是字符串、数组、对象或者geohash,所以对应有四种插入方法：
```
PUT /attractions/restaurant/1
{
  "name": "Chipotle Mexican Grill",
  "location": "40.715, -74.011"               #字符串形式
}

PUT /attractions/restaurant/2
{
  "name": "Pala Pizza",
  "location": {                             #对象形式
    "lat":  40.722,
    "lon":  -73.989
  }
}

PUT /attractions/restaurant/3
{
  "name": "Mini Munchies Pizza",
  "location": [ -73.983, 40.719 ]             #数组形式
}

PUT /attractions/restaurant/4
{ 
  "name": "Geo-point as a geohash",
  "location": "drm3btev3e86"                    #geohash形式
}
```

<!-- more -->

使用地理边界框查询，可查找框内的所有地理点：

```
GET /attractions/_search
{ 
  "query":{ 
    "geo_bounding_box": {
      "location":{ 
        "top_left":{ 
          "lat":42,
          "lon":-72 
         },
        "bottom_right":{ 
          "lat":40,
          "lon":-74 
        } 
      } 
    } 
  } 
}
```
>注意点：用字符串形式表示时，是纬度在前，经度在后（ "latitude,longitude" ），而数组形式表示时，是经度在前，纬度在后


#### 地理坐标盒模型过滤器

有四种地理坐标点相关的过滤方式可以用来选中或者排除文档：

`geo_bounding_box`: 
找出落在指定矩形框中的坐标点
`geo_distance`: 
找出与指定位置在给定距离内的点
`geo_distance_range:`: 
找出与指定点距离在给定最小距离和最大距离之间的点
`geo_polygon:`: 
找出落在多边形中的点。这个过滤器使用代价很大。当你觉得自己需要使用它，最好先看看 geo-shapes




>这是目前为止最有效的地理坐标过滤器了，因为它计算起来非常简单。你指定一个矩形的顶部,底部,左边界，和右边界，
然后过滤器只需判断坐标的经度是否在左右边界之间，纬度是否在上下边界之间

```
GET my_index/_search
{
  "query": {
    "geo_bounding_box": { 
      "location": {
        "top_left": {
          "lat": 42,
          "lon": -72
        },
        "bottom_right": {
          "lat": 40,
          "lon": -74
        }
      }
    }
  }
}
```


