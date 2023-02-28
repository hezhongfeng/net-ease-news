# netease-news

实现网易新闻的图中图（PIP）《二零一七年娱乐圈画传》

![PIP](https://cdn.jsdelivr.net/gh/hezhongfeng/images/202302280925668.gif)

## 绘画过程分析

上面的动图，如何实现呢，看了下网易的网页结构，用了一个 canvas，把图像一帧一帧的画到 canvas 上面，比如一秒画 60 帧，就能达到顺滑的收缩效果

绘制的时候图片不是一整个大图，而是很多个图，同时存在两张图，最初的时候封面和手机屏幕重合，随着时间的推移，大图和封面一起慢慢缩小，最终大图和手机屏幕重合，封面收缩到了大图的 PIP 位置，这时候去掉封面图
，继续和下一张图一起收缩

每缩小完一张图，就替换成下一张，继续收缩，这就要求图片的顺序是排列好的，按照顺序收缩

绘画需要用到这个`CanvasRenderingContext2D.drawImage(image, sx, sy, sWidth, sHeight, dx, dy, dWidth, dHeight)`函数，函数的参数比较多，这里列出来：

| 参数    | Description                                     |
| ------- | ----------------------------------------------- |
| image   | 绘制到 canvas 的元素，可以是图片、视频和 canvas |
| sx      | image 的矩形（裁剪）选择框的左上角 X 轴坐标     |
| sy      | image 的矩形（裁剪）选择框的左上角 Y 轴坐标     |
| sWidth  | image 的矩形（裁剪）选择框的宽度                |
| sHeight | image 的矩形（裁剪）选择框的高度                |
| dx      | image 的左上角在目标画布上 X 轴坐标             |
| dy      | image 的左上角在目标画布上 Y 轴坐标             |
| dWidth  | image 在目标画布上绘制的宽度                    |
| dHeight | image 在目标画布上绘制的高度                    |

第一个参数，没啥说的，就是想要绘制的目标元素

`sx`和`sy`，想要剪裁的图片的左上角坐标，如果是`0，0`就是从原图的左上角开始

`sWidth`和`sHeight`,想要剪裁的图片的长宽

以上 4 个参数，互相配合，就可以从 image 上剪出原图的一部分

接下来 `dx dy dWidth dHeight`是针对画布的参数，也就是说在画布上选一块地方将上面裁剪下来的图像放进去。这时候问题来了，如果裁剪的和画布上准备放图像的宽高不一样咋办，好办，拉伸或者收缩

## 计算公式

上面说到，图片是按照顺序一张一张替换和收缩的，那么怎么知道我下一张图的 PIP 的位置和大小呢，这需要我们提前规定好，也就是说我知道每张图片的大小和 PIP 的大小位置（第一张封面没有）

```js
  {
    src: 'cover.jpg',
    width: '750',
    height: '1206',
  },
  {
    src: 'p1.jpg',
    width: '1875',
    height: '3015',
    innerImg: {
      width: '152',
      height: '244',
      left: '370',
      top: '1068',
    },
  },
  {
    src: 'p2.jpg',
    width: '1875',
    height: '3015',
    innerImg: {
      width: '556',
      height: '894',
      left: '1251',
      top: '1050',
    },
  },
```

我们假设，经过一段时间后，长或者宽缩小到原来的 reduce（小于 1），随着 reduce 的慢慢变小，变化也越来越大

### 当前图片

![当前图片](https://cdn.jsdelivr.net/gh/hezhongfeng/images/202302280853151.gif)

从封面开始，当前图片此时撑满整个屏幕，随着时间的推移，逐渐缩小和移动，最终缩小移动到下一张图片的 PIP 的大小和位置

这一过程中，整个图片都在画布中，而且是从图片的左上角开始剪裁的，结合上面的`drawImage`函数的参数，那么`sx`、`sy`、`sWidth`和`sHeight`这 4 个参数就是`0,0,当前图片的宽度,当前图片的高度`，也就是把整个图片都剪裁下来了，所以只需要计算当前图片在画布中的位置和长宽

![image](https://cdn.jsdelivr.net/gh/hezhongfeng/images/202302271546038.svg)

观察上图，我们假设 ABCD 是手机屏幕

EFGH 当前图片的中间态

HIJK 当前图片的最终大小和位置，也就是下一张图片的 PIP 的大小和位置

做几条辅助线，ES 平行于 HU 平行于 IB，MH 是水平的，RE、TH 是垂直的

所以经过一段时间后，有 `EF=AB*reduce` 和 `EH = AD*reduce`

先计算画布中的横坐标`dx`，最初是 0，随着 reduce 变小，越来越大，最终就是 MH 的长度，也就是 PIP 的 left

```js
ML = AR;
AR/AT=AE/AH=AS/AU;
所以AR=AT*(AS/AU)

而AS=AB-SB
AU = AB-HI
```

上面是伪代码，详细的可以看看源码

### 下一张图片

还是一样的图这里假设：ABCD 是手机的屏幕

EFGH 是小图的中间态

HIJK 是小图的最终态

对于大图来讲，每次重绘都是撑满了屏幕的，所以只计算剪裁图片的部分即可

剪裁的点最初在小图的左上角，最终在大图的左上角(也就是 0),ML 为剪裁的横坐标

假设 EF 是一段时间后的剪裁图片的宽度，所以有 `reduce = HI/EF`，所以有`EF = HI/reduce`，这个值随着 reduce 的缩小，变得越来越大，最终剪裁了整个图片

EH 的值一样

然后继续求坐标的变化，最开始剪裁的横坐标在小图的横坐标上，最后是 0，这里需要求 ML 的值

```js
ML = AR;

AR/AT=AE/AH=AS/AU
AS=AB-EF
AU=AB-HI
所以ML=AR=AT*(AB-EF)/(AB-HI)
```
