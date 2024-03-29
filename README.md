# netease-news

实现网易新闻的图中图（PIP）《二零一七年娱乐圈画传》[查看源码](https://github.com/hezhongfeng/net-ease-news)

![pip.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/57b74d9eaa9f4996b284ccf5a215140c~tplv-k3u1fbpfcp-watermark.image?)

[点击](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0bf11bbfd31145d6a91054a7077d78b7~tplv-k3u1fbpfcp-watermark.image)或者扫码预览:

![扫码预览](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0ac8c366980d4c35a733f7ca96844ce2~tplv-k3u1fbpfcp-zoom-1.image)

## 绘画过程分析

上面的动图，如何实现？直观上感觉像是一个大图，但仔细想想肯定不对，那得多大的图~看了下网页结构，用了一个 canvas 来显示，应该就是把图像一帧一帧的画到 canvas 上面，比如 1 秒画 60 帧，就能达到顺滑的收缩效果。

![封面.svg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bd4bfcce37ec43b7b6d44e32b73e8ac9~tplv-k3u1fbpfcp-watermark.image?)

绘制的时候，同时存在两张图，最初的时候图 1 和屏幕重合，图 2 在屏幕外边。随着时间的推移，图 2 和图 1 一起慢慢缩小，最终图 2 和屏幕重合，图 1 收缩到了图 2 的 PIP 位置。这时候去掉图 1，添加图 3，图 3 和图 2 一起收缩，然后再添加图 4，一直到最后一张图。

想要完成整个过程，需要保证：

1. 所有图、屏幕显示区域（宽度最大，长度可能有空白或者滚动条）、以及 PIP 的长宽比是一致的，这样才能在缩小后完美重合
2. 当前的图片，需要知道下一张图的 PIP 的位置和大小，这样才能缩小移动到指定位置
3. 一般这种画中画都是有事件顺序的，所以要求图片的顺序是排列好的，按照顺序收缩

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

第一个参数，没啥说的，就是想要绘制的目标元素，我们这里是图片

`sx`和`sy`，想要剪裁的图片的左上角坐标，如果是`0，0`就是从原图的左上角开始

`sWidth`和`sHeight`，想要剪裁的图片的长宽

以上 4 个参数，互相配合，就可以从 image 上剪出原图的一部分

接下来 `dx dy dWidth dHeight`是针对画布的参数，也就是说在画布上选一块地方将上面裁剪下来的图像放进去。这时候问题来了，如果裁剪的和画布上准备放图像的宽高不一样咋办？好办，拉伸或者收缩

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
    pipImg: {
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
    pipImg: {
      width: '556',
      height: '894',
      left: '1251',
      top: '1050',
    },
  },
```

我们假设，经过一段时间后，长或者宽缩小到原来的 reduce（小于 1），随着 reduce 的慢慢变小，变化也越来越大

### 当前图片

![当前.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8ea1b73818234a92ba6e040ee32552b1~tplv-k3u1fbpfcp-watermark.image?)

从封面开始，当前图片此时撑满整个屏幕，随着时间的推移，逐渐缩小和移动，最终缩小移动到下一张图片的 PIP 的大小和位置

这一过程中，整个图片都在画布中，而且是从图片的左上角开始剪裁的，结合上面的`drawImage`函数的参数，那么`sx`、`sy`、`sWidth`和`sHeight`这 4 个参数就是`0,0,当前图片的宽度,当前图片的高度`，也就是把整个图片都剪裁下来了，所以只需要计算当前图片在画布中的位置和长宽

![计算.svg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ef395709c5fe47fd8b28e2285247117d~tplv-k3u1fbpfcp-watermark.image?)

观察上图，我们假设 ABCD 是手机屏幕，也就是当前图片最初的大小和位置

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

```js
AB：当前图片的宽度
SB: 中间态宽度，即AB*reduce
HI: 下一张图片的PIP的宽度
AT: 下一张图片的PIP的left
```

上面是伪代码，详细的可以看看源码

### 下一张图片

![下一张.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/69e441132ba64819b415e32a77e3432a~tplv-k3u1fbpfcp-watermark.image?)

观察上面的动图，最初 nextImage（下一张图片）的 PIP 占满了画布，然后画布慢慢扩大，最后是整个 nextImage 占满了画布

![计算.svg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ef395709c5fe47fd8b28e2285247117d~tplv-k3u1fbpfcp-watermark.image?)

还是一样的图这里假设：ABCD 是 nextImage

EFGH 看做画布的中间态（扩大过程中），最初和 nextImage（下一张图片）的 PIP 重合，慢慢变大

HIJK 是画布的最初状态，大小就是 nextImage 的 PIP

分析这个过程，整个过程图像都是撑满了画布的，所以`dx,dy,dWidth，dHeight`使用固定的值`(0,0,屏幕的宽,屏幕的高)`，只计算剪裁图片的部分即可

先计算剪裁图片的横坐标 `sx`，最初在 PIP 的左上角，也就是`nextImage.pip.left`，最终在整张图的左上角(也就是 0)，也就是 ML 为剪裁的横坐标

因为 EF 从 HI 慢慢扩大的，变化率是 reduce，所以有 `reduce = HI/EF`，所以有`EF = HI/reduce`，这个值随着 reduce 的缩小，变得越来越大，最终达到了 AB，就结束了

接下来求 ML

```js
ML = AR;

AR/AT=AE/AH=AS/AU
AS=AB-EF
AU=AB-HI
所以ML=AR=AT*AS/AU=AT*(AB-EF)/(AB-HI)

AT: nextImage.PIP.left
AB: nextImage.width
EF: nextImage.PIP.width / reduce
HI: nextImage.PIP.width
MH: 和AT相等
```

同理可求`sy`,`sWidth`也就是 EF

## 其他动画

![开始.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a9fafa29f1464f0ca1798a7d89011ecb~tplv-k3u1fbpfcp-watermark.image?)

原作的动画很多都是使用背景图片雪碧图+简单动画

这里介绍下 `background-position`和`background-size`:

`background-position`:控制背景图片在**容器元素**中的位置，也就是图片的左上角在元素中的位置

`background-size`:调整图片到指定大小，百分比相对于**容器元素**的尺寸

以封面的"长按"按钮为例：

1. 要保证在不同宽度的设备上都显示一样的比例的按钮，使用了 `vw` 作为图片容器的长宽单位
2. 要保证容器中只显示出按钮，要使用 `vw` 去设置`background-position`，这样可以让这个"开始"两字的左上角和容器的左上角重合
3. 雪碧图的原始大小是固定的，现在想把他适配到不同宽度的设备，需要利用`background-size`将图片大小调整到相对于容器的大小，此时需要配合`background-position`一起调整

```css
width: 15vw;
height: 14vw;
background: url(./assets/images/sprite_v2.png) no-repeat;
background-position: -41.4vw -79.45vw;
background-size: 770%;
```

## 感谢

感谢[这篇文章](https://juejin.cn/post/6844903725073432584)的指引
