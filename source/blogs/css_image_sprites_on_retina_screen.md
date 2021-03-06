Title: Retina屏下的CSS雪碧图
Date: 2014-03-19 13:39
Tags: CSS 雪碧图 背景图

CSS雪碧图早已经成为前端知识体系中一个必备知识了，时至今日，可能很多人都觉得这一块已经没有什么东西可以再讲了的。但事实上雪碧图一直都可以引出新的话题，比如从最早的连接数和体积的平衡到格式之争到图像摆放位置的策略，再到合并图像的颗粒度，再到内存占用、CPU占用等性能问题……

没错，今天还要在这一古老的话题上展开，引入一个新的问题，那就是雪碧图在retina屏下存在的问题及应对方案。（值得注意的是，retina屏一般指分辨率为普通屏幕两倍的屏，这样按照普通尺寸开发出来的网站相当于被放大了2倍，会导致图像模糊之类的现象产生，理想的解决方案是为retina专门适配一套皮肤，但本文关注的问题是未适配retina屏幕的网站所出现的问题。）

> 雪碧图本身不是浏览器或者web标准中的技术，因此它的不少细节取决于浏览器的实现，本文中的讨论的内容正是如此，为避免争议，本文所有结论的得出场景限定为Mac OSX 10.9.1、Chrome浏览器V33。是否适用iPhone、iPad等场景未做相应测试。

## 无处不在的白边

如前文所述，在retina屏上浏览未做专门适配的网站，会出现图像模糊等问题，无法达到最佳效果，但一般情况下，仍然处于可以接受的范围。不过，在某些网站上，却出现了比图像模糊更糟糕的情况：

![WebQQ在retina屏下出现白边](/images/css_image_sprites_on_retina_screen_1.png)

<!-- $$solo_more$$ -->

![财付通首页菜单在retina屏下出现白边](/images/css_image_sprites_on_retina_screen_2.png)

![支付宝的按钮在retina屏下出现白边](/images/css_image_sprites_on_retina_screen_3.png)

从上面三张图看到，不少的互联网产品在retina屏下都出现了白边。那么这些白边出现的原因是什么呢？通过查看这些出问题的页面，发现存在一个共同点，那就是这些白边所在的地方都使用了背景图，而且都是使用雪碧图合并的。于是问题就浮现出来了，正是由于雪碧图在retina屏上的放大导致了白边的产生。更为技术化的表达则是，**图片放大过程中进行了插值运算，导致原来整齐的图片边界混入了插值后模糊的像素**，从而导致原来整齐的边界处出现“白边”。

打开WebQQ的雪碧图（<http://0.web.qstatic.com/webqqpic/pubapps/0/50/images/eqq_sprite.gif?t=20111011001>），就可以验证这一结论。

![WebQQ雪碧图](/images/css_image_sprites_on_retina_screen_4.gif)

如果您在阅读本文时刚好使用的retina屏幕，可能已经能看到图标边上的白边了，为了统一说明，特放上图像编辑软件中局部放大的图片。

![WebQQ雪碧图局部放大](/images/css_image_sprites_on_retina_screen_5.png)

图中可以看到，边缘是非常整齐的，但我们在retina屏上截到的图却是这样：

![WebQQ雪碧图局部放大](/images/css_image_sprites_on_retina_screen_6.png)

可以看到，由于retina屏下，图片被强制放大，导致了原本整齐的边界不再整齐，从而使得页面上出现“白边”。

## 真的是因为雪碧图吗

至此，我们已经推断出白边是因为图片被放大而导致，那么这跟雪碧图有关系吗？如果不用雪碧图会出现这样的现象么？为了验证这个结论，本文曾一度中断，最终还是拿到了比较令人信服的结果。

首先，我们准备一张如下的图片：

![实验用图1](/images/css_image_sprites_on_retina_screen_7.png)

这张图放了四个色块，其中左上和右下的色块有留白。接下来我们在浏览器中打开它，结果如下：

![实验用图1](/images/css_image_sprites_on_retina_screen_8.png)

可以看到，图片中有颜色交界的地方都有插值运算而导致模糊，但边缘却是清晰的！也就是说，如果没有拼图的话，浏览器是可以处理好图片的边缘的。为了保险起见，接下来又做了一个实验，准备了一张20*20的纯红色图片，并与CSS写的红色背景进行混合，看看是否有“白边”出现。

![实验用图2](/images/css_image_sprites_on_retina_screen_9.gif)

可以看到，在CSS背景色不断变化的过程中，图片与背景可以完全融合，没有任何奇怪的现象出现。

至此，我们终于判定，导致“白边”的原因就是因为雪碧图中不同图像之间在拼合后产生了插值而导致边缘部分模糊。

## 解决之道

知道了原因就好解决了，既然白边的出现是因为插值，并且这个插值行为不可控，那就只好将插值的部分移出视野之外了。讲人话就是：切图的时候多留点“出血”。

继续拿WebQQ为例，如上面所说，原文中聊天气泡图标的背景宽度是20px，两边各加1px，总共22px，效果如下：

![WebQQ图标改进1](/images/css_image_sprites_on_retina_screen_10.png)

![WebQQ图标改进1效果](/images/css_image_sprites_on_retina_screen_11.png)

可以看到白边已经减少了不少，但仍然存在。继续在两边各加1px，总共24px，效果如下：

![WebQQ图标改进2](/images/css_image_sprites_on_retina_screen_12.png)

![WebQQ图标改进2效果](/images/css_image_sprites_on_retina_screen_13.png)

至此，问题完美解决，结论是：

**切图时请为图标在各个方向上多留2px空间**，即可保证retina屏下不出现意料之外的毛边（白边）。

## The End?

这就完了？当然没有，还有另外一类案例解决不了，就是开头提到的支付宝的按钮。如果你不记得了，没关系，我再放一次图：

![支付宝的按钮在retina屏下出现白边](/images/css_image_sprites_on_retina_screen_3.png)

看一下它的雪碧图(<https://i.alipayobjects.com/e/201204/2vCVR5Bh4d.png>)和结构：

![支付宝按钮雪碧图](/images/css_image_sprites_on_retina_screen_14.png)

![支付宝按钮结构](/images/css_image_sprites_on_retina_screen_15.png)

可以看出，这个按钮其实是由左边两边拼合而成（分别由内外两层元素组成），白边来自右边的结构。如果把左边的背景屏蔽掉，会看得更清楚：

![支付宝按钮结构](/images/css_image_sprites_on_retina_screen_16.png)

这种情况下，背景图小于容器本身，因此无法将插值部分排除到视野外，也即上面说的多留2px也无法解决。（事实上左边已经留有N像素了……）那就只好再利用上面在验证是否是雪碧图才有问题时说的另外一个结论了：边缘部分是不会被插值的。

于是，将这个雪碧图需要插值的部分改到边缘去，如下图（只改了上面的几个）：

![支付宝雪碧图修改版](/images/css_image_sprites_on_retina_screen_17.png)

效果如下：

![支付宝雪碧图修改版效果](/images/css_image_sprites_on_retina_screen_18.png)

至此问题解决。

> 注：之所以会采用这样一种结构的按钮，是因为它可以根据文字长度进行自适应，`background-position`的`x`值取`right`即可保证雪碧图是始终靠按钮右边对齐的。而修改版中改变雪碧图结构后，则需要手工指定`background-position`的`x`值。

> 一种更好的解决方案则是直接使用CSS3来写按钮，在IE下进行降级。

## 结

这应该是博客中图片最多的一篇了，关注的也是一个非常非常非常小的点，起因只是因为支付宝的按钮在我发现这个问题一年后仍然没有改过，于是忍不住研究了一下这问题到底有多难解决。

最后，根据上述实验和推断过程小结一下在应用雪碧图的过程中值得注意的点：

1. 雪碧图中请给背景留出足够的空间（出血），否则可能导致retina屏下产生毛边（白边）
2. 雪碧图如果图标排得太过密集，可能导致retina屏下出现“窜色”（与毛边一样，是由于插值导致）
3. 如果插值区域无法避免，请将它放在图片边缘位置

最后的最后，一句题外话，以上所有现象均可在部分浏览器放大页面时出现（我忘记是什么浏览器了，曾有项目因此被产品经理报bug，最终将所有图标周围留了2px空白解决）。
