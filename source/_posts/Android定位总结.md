---
title: Android定位总结
date: 2017-07-17 00:00:00
tags:
- Android
---

本文介绍了Android中常用的几种定位方式，仅供参考。
<!--more-->

# Android定位总结

## 常用的几种定位方式

 Android开发中常用的定位方式有下面这几种：

- SDK：百度、高德、腾讯等公司都提供了位置服务，不过要先注册他们的开发者帐号，然后导入SDK就行了。具体怎么搞参考各家的文档就行了。
- GPS：利用手机的GPS模块获得GPS信号进行定位。
- IP定位：获得手机上网的IP之后，利用API获得ip的大概位置。
- WIFI\网络定位：利用手机信号塔和WIFI接入点定位
- AGPS定位（AssistedGPS：辅助全球卫星定位系统）：利用网络来辅助GPS定位。

下面分别介绍一些这些定位方式。并给出部分源代码。

### GPS定位

 GPS定位就是利用手机的GPS模块搜索GPS卫星信号，优缺点如下：

优点：

- 定位精度高，在野外空旷没有网络信号的地方也可以使用。现在所有的手机设备都装有GPS模块，可以说是使用范围最广的一种。
- 可以获得经纬度，速度，时间，海拔高度、方位等信息，信息比较全面。

缺点：

- 比较耗电

- 定位速度受到环境条件的影响比较大，室外定位比较快，大概需要几秒的时间。室内定位速度比较慢，从GPS模块启动到获取第一次定位数据，可能需要比较长的时间。同时天气也会影响到定位的速度。

下面是示例代码（注意先添加权限）
`<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />`
```
//获得 LocationManager
    
 LocationManager locationManager =(LocationManager)getSystemService(Context.LOCATION_SERVICE);  
//位置监听器
LocationListener listener = new LocationListener(){
    @Override
    public void onLocationChanged(Location location){
    //这里处理当位置变化时的操作
    }
    @Override
    
    public void onStatusChanged(String provider, int status, Bundle extras){
    //这里处理LocationProvier变化时的操作
    }
    @Override  
    public void onProviderEnabled(String provider){
    //这里处理当provider可用时的操作
    }
    @Override
    public void onProviderDisabled(String provider){
    //这里处理当provider不可用时的操作
    }
};
    
//四个参数的含义：第一个是Provider，因为我们这里是GPS定位，所以传入GPS_PROVIDER，第二个是两次调用listener方法的最小时间间隔，设为0表示，以最快的速度刷新位置信息，第三个是位置变化的最小范围，第四个就是要回调的对象。
    
locationManager.requestLocationUpdates(LocationManager.GPS_PROVIDER,1000,0,listener);
```

​

### WIFI\网络定位

优点：

- 定位速度要比GPS定位快很多

缺点：

- 需要连接网络

- 定位不如GPS定位准确，只能获得大概的位置

​

下面给出示例代码（注意权限的变化）
```
//get LocationManager
LocationManager locationManager = (LocationManager)getSystemService(Context.LOCATION_SERVICE);  
//位置监听器
LocationListener listener = new LocationListener(){
    @Override
    public void onLocationChanged(Location location){
    //这里处理当位置变化时的操作
    }
     @Override
    public void onStatusChanged(String provider, int status, Bundle extras){
    //这里处理LocationProvier变化时的操作
    }
    @Override
    public void onProviderEnabled(String provider){
    //这里处理当provider可用时的操作
    }
    @Override
    public void onProviderDisabled(String provider){
    //这里处理当provider不可用时的操作
    }
};
//四个参数的含义：第一个是Provider，因为我们这里是GPS定位，所以传入GPS_PROVIDER，第二个是两次调用listener方法的最小时间间隔，设为0表示，以最快的速度刷新位置信息，第三个是位置变化的最小范围，第四个就是要回调的对象。
//注意下面provider的变化
locationManager.requestLocationUpdates(LocationManager.NETWORK_PROVIDER,1000,0,listener);`
```

### IP定位

 原理：每个ip都有对应的地理位置，可以通过上网ip来获得大概的位置。
优点：

- 定位速度要比GPS定位快一些

缺点：

- 需要连接网络

- 需要额外的API支持

- 定位不如GPS精确，只能获得地理位置信息。具体能获得些啥要看使用的API，一般来说能获得经纬度，时间以及衍生信息，比方说国家，城市等。


IP定位顾名思义分为两步，第一步是获得上网设备的ip，第二步就是利用这个ip来获取位置信息，利用下面这个api可直接获得位置信息：

`http://ip-api.com/json/`
具体代码就不贴了，很简单。下面给出一个返回json字符串，我们看看能获得哪些信息：

> {“as”:”AS4538 China Education and Research Network Center”,”city”:”Wuhan”,”country”:”China”,”countryCode”:”CN”,”isp”:”China Education and Research Network Center”,”lat”:30.5801,”lon”:116.2734,”org”:”GUANSHANKOU NANZI ZHIYE JISHU XUEYUAN “,”query”:”222.20.31.7”,”region”:”42”,”regionName”:”Hubei”,”status”:”success”,”timezone”:”Asia/Shanghai”,”zip”:””}

 信息还比较丰富，国家，省，市，国家代码，网络类型，经纬度，ip，市区，根据这些就能确定大概的位置了，对于确定大概位置已经够用了。

### AGPS定位

 AGPS定位就是利用基地台代送辅助卫星信息，缩减 GPS 芯片获取卫星信号的延迟时间，受遮盖的室内也能利用基地台信号弥补，减轻 GPS 芯片对卫星的依赖度。 一般来说这种定位方式都是系统系动在后台完成的，所以没有专门的Android API调用。

优点：

- 缩短定位时间：由于利用移动网络提供GPS辅助信息，不需要移动终端通过接收GPS卫星广播数据。由于卫星广播信道速率非常低，信号强度非常弱，这个时间通常会非常长。
- 降低终端耗电量：由于不需要对卫星进行全频段扫描和跟踪，定位时间缩短，因此终端的耗电量大大降低。
- 提升定位灵敏度：在靠近建筑物或者天气不好等相对恶劣环境下，由于有网络辅助数据，终端可直接锁定卫星定位，而此时GPS卫星信号非常微弱，独立GPS定位模式则往往终端会因为不能接收完所有的卫星星历和时钟等参数而导致定位失败。

缺点：

- 需要数据流量

## 最后说一下纠偏的问题

### 为什么要纠偏？

 一般来说我们的获得的准确经纬度坐标叫做WGS-84坐标，但是出于国家保密需要，会使用SM模组对这个GPS坐标做一个非线性加偏（￥10），变成GCJ-02，又称火星坐标。反映到地图上就是，你输入了天安门的坐标，结果发现这个坐标上是上海火车站。GPS坐标是正确的，但是放到地图上就不准。为了解决这个问题，把GPS坐标做了同样的加偏，这样GPS和地图就偏到一起了，也就实现了定位。
 说得通俗一点，你和你朋友去光谷玩，约在资本大厦前面见面，结果城管用SM牌三轮车把你朋友拉到了天河影城，并收了他十块钱。你到了资本大厦前面没看见你朋友，就给了城管十块钱，让他也把你拉到了天河影城。然后你俩就见面了。
 现在国内的所有能接收GPS信号的行货设备（手机、平板、手持GPS设备等）都要进行加偏，所以当使用国内设备产生的GPS信息标在GOOGLE地图上时，因为GOOGLE 地图没有加偏，所以位置信息是不准确的。需要进行纠偏。但是如果标在经过加偏的地图上就没有这个问题

### 如何纠偏

> [https://www.google.com.hk/search?safe=strict&q=gps%E7%BA%A0%E5%81%8F%E7%AE%97%E6%B3%95&oq=gps%E7%BA%A0%E5%81%8F%E7%AE%97%E6%B3%95&gs_l=serp.3…377144.382403.0.382826.24.17.1.0.0.0.713.1782.4-1j1j1.3.0….0…1.1j4.64.serp..22.2.647…0j0i5i30k1.f4K_FChDRds](https://www.google.com.hk/search?safe=strict&amp;q=gps%E7%BA%A0%E5%81%8F%E7%AE%97%E6%B3%95&amp;oq=gps%E7%BA%A0%E5%81%8F%E7%AE%97%E6%B3%95&amp;gs_l=serp.3...377144.382403.0.382826.24.17.1.0.0.0.713.1782.4-1j1j1.3.0....0...1.1j4.64.serp..22.2.647...0j0i5i30k1.f4K_FChDRds)

参考资料：

> GPS定位原理：[http://www.cnblogs.com/lcw/p/3572628.html](http://www.cnblogs.com/lcw/p/3572628.html)
> 
> AGPS定位：[http://www.apkbus.com/thread-116655-1-1.html](http://www.apkbus.com/thread-116655-1-1.html)
> 
> GPS纠偏以及偏移原因：[http://yanue.net/post-121.html](http://yanue.net/post-121.html)
