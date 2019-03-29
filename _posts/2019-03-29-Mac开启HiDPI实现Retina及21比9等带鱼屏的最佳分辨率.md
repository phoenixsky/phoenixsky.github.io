---

layout: post

title: 'Mac开启HiDPI实现Retina及21:9等带鱼屏的最佳分辨率'

subtitle: '强制开启HiDPI实现Retina'

date: 2018-05-12

categories: Mac

tags: HiDPI 显示器 

---
强制开启HiDPI,还你一片清晰的视界

---
# .

最近购入了LG的34UC99`3440x1440`,回来接到我的Mac mini 2018上边,发现字体太小了,有种能把眼睛瞅瞎的感觉.随即调整了分辨率,可是根据当前显示器的分辨,如果想实现HiDPI的效果,只能缩放到1720x720,这样字体又太大,空间利用率不够.设置为别的分辨率,字体又明显发虚.随即研究了下强制开启HiDPI.

## 如何强制开启HiDPI.
### 原理
根据DisplayProductID和DisplayVendorID在制定文件夹下,生成响应的文件.
下图是定制HiDPI分辨率后,与之前对比生成的ID.
![](http://blogimgs.phoenixsky.cn/2019-03-29-15538300616167.jpg)

在对应显示器和对应连接方式下,添加响应的文件,具体文件内容可参见[Scaled Resolutions for your MacBooks external Monitor](https://comsysto.github.io/Display-Override-PropertyList-File-Parser-and-Generator-with-HiDPI-Support-For-Scaled-Resolutions/)

> 小明问,说了这么多,那我的显示器应该应该这么弄呢.

### 执行
直接按照[one\-key\-hidpi](https://github.com/boboidream/one-key-hidpi/blob/master/README-zh.md)的操作.

* 终端输入
    ```
    $ sh -c "$(curl -fsSL https://raw.githubusercontent.com/xzhih/one-key-hidpi/master/hidpi-zh.sh)"
    ```
* 如图
![one-key-hidpi](http://blogimgs.phoenixsky.cn/2019-03-29-one-key-hidpi.jpg)

* 手动输入分辨率时,如`1920x1080`,中间的`x`,是小写字母`x`.
* 如果你想要的分辨率是2560X1072,还要多添加一个5120X2144,这样才能成功
    > 因为这个是前者的2倍，只有添加这个，前者才能以HIDPI的形式被打开，原理应该是强制让系统运行在5120X2144的分辨率下再打开的2560X1072的HIDPI，通过全屏截图的分辨率来看猜想是正确的，所以对机器显卡性能有点要求

* 如果出现权限问题,或者不生效,请检查下SIP 
    * rootless是否开启
    ```
    csrutil status //查看状态. enable为开启保护状态
    ```
    * 重启电脑按住`Command-R`进入恢复分区,在实用工具栏进入终端
    ```
    csrutil disable //关闭
    csrutil enable //开启
    ```
    
* 恢复请参考[原贴](https://github.com/boboidream/one-key-hidpi/blob/master/README-zh.md)

## 21:9应该用多少分辨率
我的显示器是34寸的21:9,原始分辨率为3440x1440.
初步设想的实际渲染2560x1080,显卡输出5120x2160,但是设置后,在屏幕左右两边有个黑边,不是完美状态.
weiphone上的这个哥们给了个方案[关于MAC使用21:9准4K带鱼屏的HIDPI设置心得](https://bbs.feng.com/forum.php?mod=viewthread&tid=11367567&extra=&ordertype=1&page=1),上文的猜想也是这哥们的文章.

所以这才是正确的分辨率:
---
### 2560X1072
### 5120X2144
---

## 下载RDM,调整到你想要分辨率.
![](http://blogimgs.phoenixsky.cn/2019-03-29-15538322568483.jpg)

原文链接:
[RDM: Easily set Mac Retina display to higher unsupported resolutions](https://github.com/avibrazil/RDM)

* [DMG下载链接](http://avi.alkalay.net/software/RDM/RDM-2.2.dmg)


---

## 遇到的坑
Mac mini 2018 OS:10.14.4
LG 34UC99 2560x1080 5120x2160
最开始是使用了[syscl/Enable\-HiDPI\-OSX](https://github.com/syscl/Enable-HiDPI-OSX),一直在开机苹果logo界面卡着不动,进不了系统最简单的办法是换个显示器,重新进入系统后,恢复一下.
后边尝试了`one-key-hidpi`后一次成功.
在对比了两次生成的文件,发现在使用`Enable-HiDPI-OSX`,增加分辨率时,不仅要加入你想自定义的分辨率,应该还要加入显示器本身的分辨率.否则第一次加载,找不到默认的配置.



## 感谢
* [boboidream/one\-key\-hidpi](https://github.com/boboidream/onekey-hidpi/blob/master/README-zh.md)
* [关于MAC使用21:9准4K带鱼屏的HIDPI设置心得 ](https://bbs.feng.com/forum.php?mod=viewthread&tid=11367567&extra=&ordertype=1&page=1)

* [syscl/Enable\-HiDPI\-OSX](https://github.com/syscl/Enable-HiDPI-OSX)

* [Scaled Resolutions for your MacBooks external Monitor 手动填入参数](https://comsysto.github.io/Display-Override-PropertyList-File-Parser-and-Generator-with-HiDPI-Support-For-Scaled-Resolutions/)


