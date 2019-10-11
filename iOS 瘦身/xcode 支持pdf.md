## xcode 支持 pdf 矢量图



在iOS平台，Xcode是在编译时，根据你的矢量PDF图的大小，生成1x、2x和3x图。如果你的PDF图是45**45px，那么Xcode会在编译时生成下面3个PNG：*

45 * 45px &nbsp; &nbsp; &nbsp;：1x设备用的(iPhone 3G and 3GS)

90 * *90px &nbsp; &nbsp; &nbsp;：2x或Retina显示设备用的(iPhone 4, 4S, 5, 5S, and 6)

*135* * 135px &nbsp; &nbsp;：3x设备用的(iPhone 6 Plus 及以上)

这也意味着当有更高的屏幕分辨率时，Xcode可以根据已有的矢量PDF放大图片，这样自动就支持以后的设备了。

