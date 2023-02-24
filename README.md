# netease-news

大图 ABCD
小图 HIJK
手机屏幕 EFGH
最初的时候小图和手机屏幕重合
随着时间的推移，慢慢缩小，最终大图和手机屏幕重合
我们假设，长和宽缩小的系数是 factor，随着这个系数的慢慢变小，变化越来越大
即每次重绘，都有 EF=AB*factor
EH = AD*factor

开始计算大图的每次重绘的数据

对于大图来讲，每次重绘都是撑满了屏幕的，所以只计算剪裁图片的部分即可

剪裁的点最初在小图的左上角，最终在大图的左上角(也就是 0),ML 为剪裁的横坐标

```js
ML = AR;

AR/AT=AE/AH=AS/AU
AS=AB-EF
AU=AB-HI
所以ML=AR=AT*(AB-EF)/(AB-HI)
```

当前图片，整个图片都在屏幕内，逐渐缩小，最终缩小到下一个图片的小照片位置，结束

所以只需要计算当前图片在屏幕中的位置即可

先计算屏幕中的坐标，最初是 0，随着 factor 变小，越来越大，最终是小图片的 left,也就是 HL

```
NH=EL=RL-RE
ER/HT=AE/AH=AS/AU
ER=HT*AS/AU

所以NH=RL-HT*AS/AU
```