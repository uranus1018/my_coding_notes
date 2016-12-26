## 数据库的空间索引技术

[SQLServer的空间索引](https://technet.microsoft.com/zh-cn/library/bb964712(v=sql.105).aspx)

[Geospatial Index Tutorials](https://docs.mongodb.com/v3.0/administration/indexes-geo/)

[深入浅出空间索引](http://www.cnblogs.com/LBSer/p/3392491.html)

## 地图数据源

### 坐标系：
[国内各地图坐标系及变换](http://blog.csdn.net/findsafety/article/details/12442639)

- ios: WGS84坐标系
- GCJ02: 国测局坐标
- 百度: bd09坐标系
- 火星坐标系：
  iOS 地图（其实是高德）
  Gogole地图
  搜搜、阿里云、高德地图百度坐标系：
  当然只有百度地图
- WGS84坐标系：
  国际标准，谷歌国外地图、osm地图等国外的地图一般都是这个

### 抓取百度瓦片地图
[使用百度地图JavaScript API构建离线地图应用（完整教程）](http://blog.csdn.net/geekxm/article/details/14227139)

[百度地图2.0瓦片地址获取(窗口内瓦片)](http://my.oschina.net/smzd/blog/628173)

### 百度瓦片分析
[百度地图根据经纬度计算瓦片行列号](http://www.cnblogs.com/xiaozhi_5638/p/4748186.html)

根据百度经纬度计算出来墨卡托坐标后，将结果除以地图分辨率Math.Pow(2,18-zoom)即可得到平面像素坐标，然后将像素坐标除以256分别得到瓦片的行列号。
[百度地图API详解](http://www.cnblogs.com/jz1108/archive/2011/07/02/2095376.html)
