---
title: 【转载】Unity性能优化
date: 2017-09-25 23:19:18
categories:
- Unity
tags:
- Unity性能优化
cover: https://raw.githubusercontent.com/Dioooooooor/BlogImageHosting/master/image/UnityBlackLarge.png
---

转载声明：原文出自[妈妈说女孩子要自立自强](http://my.csdn.net/candycat1992)的[【Unity技巧】Unity中的优化技术](http://blog.csdn.net/candycat1992/article/details/42127811)，如有侵权，可以联系本人删除。

<p style="text-indent:2em">之前面试的时候遇到面试官问到关于移动端优化的知识点，之前由于这方面了解的比较少，所以回来之后好好的补习了一下，发现这方面的教程挺全挺多的，而自己了解又比较有限，所以找了其中我认为写的比较好的一篇，自己看了几遍，为了方便以后的复习，也为了今后自己总结，所以转载此篇，并附带其他找到的链接，供自己和他人学习。</p>
<!-- more -->

# 写在前面

这一篇是在Digital Tutors的一个系列教程的基础上总结扩展而得的~Digital Tutors是一个非常棒的教程网站，包含了多媒体领域很多方面的资料，非常酷！除此之外，还参考了Unity Cookie中的一个教程。还有很多其他参考在下面的链接中。

这篇文章旨在简要地说明一下常见的各种优化策略。不过对每个基础有非常深入地讲解，需要的童鞋可以自行去相关资料。

还有一些我认为非常好的参考文章：

[Performance Optimization for Mobile Devices](http://robotinvader.com/blog/?p=438) 

[4 WAYS TO INCREASE PERFORMANCE OF YOUR UNITY GAME](http://www.paladinstudios.com/2012/07/30/4-ways-to-increase-performance-of-your-unity-game/) 

[Unite 2013 Optimizing Unity Games for Mobile Platforms](http://malideveloper.arm.com/downloads/Unite_2013-Optimizing_Unity_Games_for_Mobile_Platforms.pdf)

[Unity optimization Tips](http://unitycoder.com/blog/2014/04/23/unity-optimization-tips/)


# 影响性能的因素

首先，我们得了解，影响游戏性能的因素哪些，才能对症下药。对于一个游戏来说，有两种主要的计算资源：CPU和GPU。它们会互相合作，来让我们的游戏可以在预期的帧率和分辨率下工作。CPU负责其中的帧率，GPU主要负责分辨率相关的一些东西。

总结起来，主要的性能瓶颈在于：
* CPU
    * 过多的Draw Calls
    * 复杂的脚本或者物理模拟

* 顶点处理
    * 过多的顶点
    * 过多的逐顶点计算

* 像素（Fragment）处理
    * 过多的fragment，overdraws
    * 过多的逐像素计算

* 带宽
    * 尺寸很大且未压缩的纹理
    * 分辨率过高的framebuffer

对于CPU来说，限制它的主要是游戏中的Draw Calls。那么什么是Draw Call呢？如果你学过OpenGL，那么你一定还记得在每次绘图前，我们都需要先准备好顶点数据（位置、法线、颜色、纹理坐标等），然后调用一系列API把它们放到GPU可以访问到的指定位置，最后，我们需要调用_glDraw*命令，来告诉GPU，“嘿，我把东西都准备好了，你个懒家伙赶紧出来干活（渲染）吧！”。而调用_glDraw*命令的时候，就是一次Draw Call。那么为什么Draw Call会成为性能瓶颈呢（而且是CPU的瓶颈）？上面说到过，我们想要绘制图像时，就一定需要调用Draw Call。例如，一个场景里有水有树，我们渲染水的时候使用的是一个material以及一个shader，但渲染树的时候就需要一个完全不同的material和shader，那么就需要CPU重新准备顶点数据、重新设置shader，而这种工作实际是非常耗时的。如果场景中，每一个物体都使用不同的material、不同的纹理，那么就会产生太多Draw Call，影响帧率，游戏性能就会下降。当然，这里说得很简单，更详细的请自行谷歌。其他CPU的性能瓶颈还有物理、布料模拟、粒子模拟等，都是计算量很大的操作。

而对于GPU来说，它负责整个渲染流水线。它会从处理CPU传递过来的模型数据开始，进行Vertex Shader、Fragment Shader等一系列工作，最后输出屏幕上的每个像素。因此它的性能瓶颈可能和需要处理的顶点数目的、屏幕分辨率、显存等因素有关。总体包含了顶点和像素两方面的性能瓶颈。在像素处理中，最常见的性能瓶颈之一是overdraw。Overdraw指的是，我们可能对屏幕上的像素绘制了多次。

了解了上面基本的内容后，下面涉及到的优化技术有：

* 顶点优化
 * 优化几何体
 * 使用LOD（Level of detail）技术
 * 使用遮挡剔除（Occlusion culling）技术

* 像素优化
 * 控制绘制顺序
 * 警惕透明物体
 * 减少实时光照

* CPU优化
 * 减少Draw Calls

* 带宽优化
 * 减少纹理大小
 * 利用缩放


首先是顶点优化的部分。


# 顶点优化


## 优化几何体

这一步主要是为了针对性能瓶颈中的”顶点处理“一项。这里的几何体就是指组成场景中对象的网格结构。

3D游戏制作都由模型制作开始。而在建模时，有一条我们需要记住：尽可能减少模型中三角形的数目，一些对于模型没有影响、或是肉眼非常难察觉到区别的顶点都要尽可能去掉。例如在下面左图中，正方体内部很多顶点都是不需要的，而把这个模型导入到Unity里就会是右面的情景：     
![](http://img.blog.csdn.net/20141224210509394?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2FuZHljYXQxOTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
![](http://img.blog.csdn.net/20141224210509394?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2FuZHljYXQxOTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
 

在Game视图下，我们可以查看场景中的三角形数目和顶点数目：    
![](http://img.blog.csdn.net/20141224211224046?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2FuZHljYXQxOTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)


可以看到一个简单的正方形就产生了这么多顶点，这是我们不希望看到的。

同时，尽可能重用顶点。在很多三维建模软件中，都有相应的优化选项，可以自动优化网格结构。最后优化后，一个正方体可能只剩下8个顶点：      
![](http://img.blog.csdn.net/20141224210907497?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2FuZHljYXQxOTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
![](http://img.blog.csdn.net/20141224210907497?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2FuZHljYXQxOTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
 

它对应的顶点数和三角形数目如下：    
![](http://img.blog.csdn.net/20141224211505705?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2FuZHljYXQxOTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)



等等！这里，你可能要问了，为什么顶点数是24，而不是8呢？美术朋友们经常会遇到这样的问题，就是建模软件里显示的模型顶点数和Unity中的不一样，通常Unity会多很多。谁才是对的呢？其实，它们是站在不同的角度上计算的，都有各自的道理，但我们真正应该关心的是Unity里的数目。

我们这里简单解释一下。三维软件里更多地是站在我们人类的角度理解顶点的，即我们看见的一个点就是一个。而Unity是站在GPU的角度上，去计算顶点数目的。而在GPU看来，看起来是一个的很有可能它要分开处理，从而就产生了额外的顶点。这种将顶点一分为多的原因，主要有两个：一个是UV splits，一个是Smoothing splits。而它们的本质其实都是因为对于GPU来说，顶点的每一个属性和顶点之间必须是一对一的关系。UV splits的产生，是因为建模时，一个顶点的UV坐标有多个。例如之前的立方体的例子，由于每个面都有共同的顶点，因此在不同面上，同一个顶点的UV坐标可能发生改变。这对于GPU来说，这是不可理解的，因此它必须把这个顶点拆分成两个具有不同UV坐标的定顶点，它才甘心。而Smoothing splits的产生也是类似的，不同的时，这次一个顶点可能会对应多个法线信息或切线信息。这通常是因为我们要决定一个边是一条Hard Edge还是Smooth Edge。Hard Edge通常是下面这样的效果（注意中间的折痕部分）：        
![](http://img.blog.csdn.net/20141224213100458?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2FuZHljYXQxOTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
![](http://img.blog.csdn.net/20141224213112720?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2FuZHljYXQxOTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
 

而如果观察它的顶点法线，就会发现，折痕处每个顶点其实包含了两个不同的法线。因此，对于GPU来说，它同样无法理解这样的事情，因此会把顶点一分为二。而相反，Smooth Edge则是下面的情况：        
![](http://img.blog.csdn.net/20141224213352885?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2FuZHljYXQxOTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
![](http://img.blog.csdn.net/20141224213336989?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2FuZHljYXQxOTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
 

对于GPU来说，它本质上只关心有多少个顶点。因此，尽可能减少顶点的数目其实才是我们真正对需要关心的事情。因此，最后一条优化建议就是：**移除不必要的Hard Edge以及纹理衔接，即避免Smoothing splits和UV splits**。


## 使用LOD（Level of detail）技术

LOD技术有点类似于Mipmap技术，不同的是，LOD是对模型建立了一个模型金字塔，根据摄像机距离对象的远近，选择使用不同精度的模型。它的好处是可以在适当的时候大量减少需要绘制的顶点数目。它的缺点同样是需要占用更多的内存，而且如果没有调整好距离的话，可能会造成模拟的突变。

在Unity中，可以通过LOD Group来实现LOD技术：     
![](http://img.blog.csdn.net/20141226173207281?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2FuZHljYXQxOTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
![](http://img.blog.csdn.net/20141226173313880?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2FuZHljYXQxOTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
 


通过上面的LOD Group面板，我们可以选择需要控制的模型以及距离设置。下面展示了油桶从一个完整网格到简化网格，最后完全被剔除的例子：        
![](http://img.blog.csdn.net/20141226173626104?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2FuZHljYXQxOTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
![](http://img.blog.csdn.net/20141226173638515?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2FuZHljYXQxOTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
![](http://img.blog.csdn.net/20141226173651796?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2FuZHljYXQxOTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
![](http://img.blog.csdn.net/20141226173702280?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2FuZHljYXQxOTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
    



## 使用遮挡剔除（Occlusion culling）技术

遮挡剔除是用来消除躲在其他物件后面看不到的物件，这代表资源不会浪费在计算那些看不到的顶点上，进而提升性能。关于遮挡剔除，Unity Taiwan有一个系列文章大家可以看看（需翻墙）：

[Unity 4.3 关于Occlusion Culling : 基本篇](http://unitytaiwan.blogspot.tw/2013/12/unity-43-occlusion-culling.html)

[Unity 4.3 关于Occlusion Culling : 最佳做法](http://unitytaiwan.blogspot.com/2013/12/unity-43-occlusion-culling_26.html)

[Unity 4.3 关于Occlusion Culling : 错误诊断](http://unitytaiwan.blogspot.tw/2014/01/unity-43-occlusion-culling.html)


具体的内容大家可以自行查找。


现在我们来谈像素优化。


# 像素优化


像素优化的重点在于减少overdraw。之前提过，overdraw指的就是一个像素被绘制了多次。关键在于控制绘制顺序。

Unity还提供了查看[overdraw的视图](https://docs.unity3d.com/Manual/ViewModes.html)，在Scene视图的Render Mode->Overdraw。当然这里的视图只是提供了查看物体遮挡的层数关系，并不是真正的最终屏幕绘制的overdraw。也就是说，可以理解为它显示的是如果没有使用任何深度检验时的overdraw。这种视图是通过把所有对象都渲染成一个透明的轮廓，通过查看透明颜色的累计程度，来判断物体的遮挡。      
![](http://img.blog.csdn.net/20141226211843550?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2FuZHljYXQxOTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)


上图图，红色越是浓重的地方表示overdraw越严重，而且这里涉及的都是透明物体，这意味着性能将会受到很大影响。

## 控制绘制顺序

需要控制绘制顺序，主要原因是为了最大限度的避免overdraws，也就是同一个位置的像素可以需要被绘制多变。在PC上，资源无限，为了得到最准确的渲染结果，绘制顺序可能是从后往前绘制不透明物体，然后再绘制透明物体进行混合。但在移动平台上，这种会造成大量overdraw的方式显然是不适合的，我们应该尽量从前往后绘制。从前往后绘制之所以可以减少overdraw，都是因为深度检验的功劳。

在Unity中，那些Shader中被设置为“Geometry” 队列的对象总是从前往后绘制的，而其他固定队列（如“Transparent”“Overla”等）的物体，则都是从后往前绘制的。这意味这，我们可以尽量把物体的队列设置为“Geometry” 。

而且，我们还可以充分利用Unity的队列来控制绘制顺序。例如，对于天空盒子来说，它几乎覆盖了所有的像素，而且我们知道它永远会在所有物体的后面，因此它的队列可以设置为“Geometry+1”。这样，就可以保证不会因为它而造成overdraws。


## 时刻警惕透明物体

而对于透明对象，由于它本身的特性（可以看之前关于Alpha Test和Alpha Blending的一篇文章）决定如果要得到正确的渲染效果，就必须从后往前渲染（这里不讨论使用深度的方法），而且抛弃了深度检验。这意味着，透明物体几乎一定会造成overdraws。如果我们不注意这一点，在一些机器上可能会造成严重的性能下面。例如，对于GUI对象来说，它们大多被设置成了半透明，如果屏幕中GUI占据的比例太多，而主摄像机又没有进行调整而是投影整个屏幕，那么GUI就会造成屏幕的大量overdraws。


因此，如果场景中大面积的透明对象，或者有很多层覆盖的多层透明对象（即便它们每个的面积可以都不大），或者是透明的粒子效果，在移动设备上也会造成大量的overdraws。这是应该尽量避免的。

对于上述GUI的这种情况，我们可以尽量减少窗口中GUI所占的面积。如果实在无能为力，我们可以把GUI绘制和三维场景的绘制交给不同的摄像机，而其中负责三维场景的摄像机的视角范围尽量不要和GUI重叠。对于其他情况，只能说，尽可能少用。当然这样会对游戏的美观度产生一定影响，因此我们可以在代码中对机器的性能进行判断，例如首先关闭所有的耗费性能的功能，如果发现这个机器表现非常良好，再尝试开启一些特效功能。


减少实时光照

实时光照对于移动平台是个非常昂贵的操作。如果只有一个平行光还好，但如果场景中包含了太多光源并且使用了很多多Passes的shader，那么很有可能会造成性能下降。而且在有些机器上，还要面临shader失效的风险。例如，一个场景里如果包含了三个逐像素的点光源，而且使用了逐像素的shader，那么很有可能将Draw Calls提高了三倍，同时也会增加overdraws。这是因为，对于逐像素的光源来说，被这些光源照亮的物体要被再渲染一次。更糟糕的是，无论是静态批处理还是动态批处理（其实文档中只提到了对动态批处理的影响，但不知道为什么实验结果对静态批处理也没有用），对于这种逐像素的pass都无法进行批处理，也就是说，它们会中断批处理。

例如，下面的场景中，四个物体都被标识成了“Static”，它们使用的shader都是自带的Bumped Diffuse。而所有的点光源都被标识成了“Important”，即是逐像素光。可以看到，运行后的Draw Calls是23，而非3。这是因为，只有“Forward Base”的Pass时发生了静态批处理（这里的动态批处理由于多Pass已经完全失效了），节省了一个Draw Calls，而后面的“Forward Add” Pass，每一次渲染都是一个单独的Draw Call（而且可以看到Tris和Verts数目也增加了）：       
![](http://img.blog.csdn.net/20141226182657719?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2FuZHljYXQxOTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)




这点正如[文档](https://docs.unity3d.com/Manual/DrawCallBatching.html)中说的：The draw calls for “additional per-pixel lights” will not be batched。原因我不是很清楚，这里有[一个讨论](https://forum.unity.com/threads/draw-call-batching-and-per-pixel-lighting.176961/)，但里面的意思说是对静态批处理没有影响，和我这里的结果不一样，知道原因的麻烦给我留言，非常感谢。我也在Unity论坛里提问里。

我们看到很多成功的移动游戏，它们的画面效果看起来好像包含了很多光源，但其实这都是骗人的。


## 使用Lightmaps

Lightmaps的很常见的一种优化策略。它主要用于场景中整体的光照效果。这种技术主要是提前把场景中的光照信息存储在一张光照纹理中，然后在运行时刻只需要根据纹理采样得到光照信息即可。

当然与之配合的还有Light Probes技术。风宇冲有[一个系列](http://blog.sina.com.cn/s/articlelist_1192309394_10_1.html)文章讲过，但是时间比较久远，但教程我相信网上有很多。


使用God Rays

场景中很多小型光源效果都是靠这种方法模拟的。它们一般并不是真的光源产生的，很多情况是通过透明纹理进行模拟。具体可以参见[之前的文章](http://blog.csdn.net/candycat1992/article/details/42061701)。



# CPU优化

## 减少Draw Calls

### 批处理（Batching）

这方面的优化教程想必是最多的了。最常见的就是通过批处理（Batching）了。从名字上来理解，就是一块处理多个物体的意思。那么什么样的物体可以一起处理呢？答案就是**使用同一个材质的物体**。这是因为，对于使用同一个材质的物体，它们之间的不同仅仅在于顶点数据的差别，即使用的网格不同而已。我们可以把这些顶点数据合并在一起，再一起发送给GPU，就可以完成一次批处理。

Unity中有两种批处理方式：一种是动态批处理，一种是静态批处理。对于动态批处理来说，好消息是一切处理都是自动的，不需要我们自己做任何操作，而且物体是可以移动的，但坏消息是，限制很多，可能一不小心我们就会破坏了这种机制，导致Unity无法批处理一些使用了相同材质的物体。对于静态批处理来说，好消息是自由度很高，限制很少，坏消息是可能会占用更多的内存，而且经过静态批处理后的所有物体都不可以再移动了。

首先来说动态批处理。**Unity进行动态批处理的条件是，物体使用同一个材质并且满足一些特定条件**。Unity总是在不知不觉中就为我们做了动态批处理。例如下面的场景：     
![](http://img.blog.csdn.net/20141224215400873?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2FuZHljYXQxOTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)




这个场景共包含了4个物体，其中两个箱子使用了同一个材质。可以看到，它的Draw Calls现在是3，并且显示Save by batching是1，也就是说，Unity靠Batching为我们节省了1个Draw Call。下面，我们来把其中一个箱子的大小随便改动一下，看看会发生什么：        
![](http://img.blog.csdn.net/20141224215802736?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2FuZHljYXQxOTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)



可以发现，Draw Calls变成了4，Save by batching的数目也变成了0。这是为什么呢？它们明明还是只使用了一个材质啊。原因就是前面提到的那些需要满足的其他条件。动态批处理虽然自动得令人感动，但它对模型的要求很多：
* 顶点属性的最大限制为900，而且未来有可能会变。不要依赖这个数据。

* 一般来说，那么所有对象都必须需要使用同一个缩放尺度（可以是(1, 1, 1)、(1, 2, 3)、(1.5, 1.4, 1.3)等等，但必须都一样）。但如果是非统一缩放（即每个维度的缩放尺度不一样，例如(1, 2, 1)），那么如果所有的物体都使用不同的非统一缩放也是可以批处理的。这个要求很怪异，为什么批处理会和缩放有关呢？这和Unity背后的技术有关系，有兴趣的可以自行谷歌，比如[这里](http://answers.unity3d.com/questions/358887/why-does-scaling-affect-dynamic-batching.html)。

* 使用lightmap的物体不会批处理。多passes的shader会中断批处理。接受实时阴影的物体也不会批处理。

上述除了最常见的由于缩放导致破坏批处理的情况，还有就是顶点属性的限制。例如，在上面的场景中我们添加之前未优化后的箱子模型：      s
![](http://img.blog.csdn.net/20141226153354981?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2FuZHljYXQxOTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)



可以看到Draw Calls一下子变成了5。这是因为新添加的箱子模型中，包含了474个顶点，而它使用的顶点属性有位置、UV坐标、法线等信息，使用的总和超过了900。


动态批处理的条件这么多，一不小心它就不干了，因此Unity提供了另一个方法，静态批处理。接着上面的例子，我们保持修改后的缩放，但把四个物体的“Static Flag”勾选上：       
![](http://img.blog.csdn.net/20141224221213156?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2FuZHljYXQxOTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)


点击Static后面的三角下拉框，我们会看到其实这一步设置了很多东西，这里我们想要的只是“Batching static”一项。这时我们再看Draw Calls，恩，还是没有变化。但是不要急，我们点击运行，变化出现了：     
![](http://img.blog.csdn.net/20141224221513742?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2FuZHljYXQxOTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)



Draw Calls又回到了3，并且显示Save by batching是1。这就是得利于静态批处理。而且，如果我们在运行时刻查看模型的网格，会发现它们都变成了一个名为Combined Mesh (roo: scene)的东西。这个网格是Unity合并了所有标识为“Static”的物体的结果，在我们的例子里，就是四个物体：      
![](http://img.blog.csdn.net/20141224222505139?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2FuZHljYXQxOTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)



你可以要问了，这四个对象明明不是都使用了一个材质，为什么可以合并成一个呢？如果你仔细观察上图的话，会发现里面标明了“4 submeshes”，也就是说，这个合并后的网格其实包含了4个子网格，也就是我们的四个对象。对于合并后后的网格，Unity会判断其中使用同一个材质的子网格，然后对它们进行批处理。

但是，我们再细心点可以发现，我们的箱子使用的其实是同一个网格，但合并后却变成了两个。而且，我们观察运行前后Stats窗口中的“VBO total”，它的大小由241.6KB变成了286.2KB，变大了！还记得静态批处理的缺点吗？就是可能会占用更多的内存。[文档](http://docs.unity3d.com/Manual/DrawCallBatching.html)中是这样写的：

“Using static batching will require additional memory for storing the combined geometry. If several objects shared the same geometry before static batching, then a copy of geometry will be created for each object, either in the Editor or at runtime. This might not always be a good idea - sometimes you will have to sacrifice rendering performance by avoiding static batching for some objects to keep a smaller memory footprint. For example, marking trees as static in a dense forest level can have serious memory impact.”


也就是说，如果在静态批处理前有一些物体共享了相同的网格（例如这里的两个箱子），那么每一个物体都会有一个该网格的复制品，即一个网格会变成多个网格被发送给GPU。在上面的例子看来，就是VBO的大小明显增大了。如果这类使用同一网格的对象很多，那么这就是一个问题了，这种时候我们可能需要避免使用静态批处理，这意味着牺牲一定的渲染性能。例如，如果在一个使用了1000个重复树模型的森林中使用静态批处理，那么结果就会产生1000倍的内存，这会造成严重的内存影响。这种时候，解决方法要么我们可以忍受这种牺牲内存换取性能的方法，要么不要使用静态批处理，而使用动态批处理（前提是大家使用相同的缩放大小，或者大家都使用不同的非统一缩放大小），或者自己编写批处理的方法。当然，我认为最好的还是使用动态批处理来解决。

有一些小提示可以使用：
* 尽可能选择静态批处理，但得时刻小心对内存的消耗。

* 如果无法进行静态批处理，而要使用动态批处理的话，那么请小心上面提到的各种注意事项。例如：

    * 尽可能让这样的物体少并且尽可能让这些物体包含少量的顶点属性。

    * 使用统一缩放，或者使用不同的非统一缩放。

* 对于游戏中的小道具，例如可以捡拾的金币等，可以使用动态批处理。

* 对于包含动画的这类物体，我们无法全部使用静态批处理，但其中如果有不动的部分，可以把这部分标识成“Static”。


一些讨论：

[How static batching works](http://answers.unity3d.com/questions/593206/how-static-batching-works.html)

[Static batching use a ton of memory?](http://forum.unity3d.com/threads/static-batching-use-a-ton-of-memory.102800/)

[Unity3D draw call optimization]()http://gamedev.stackexchange.com/questions/45433/unity3d-draw-call-optimization-static-batching-vs-manually-draw-mesh-with-mate

### 合并纹理（Atlas）

虽然批处理是个很好的方式，但很容易就打破它的规定。例如，场景中的物体都使用Diffuse材质，但它们可能会使用不同的纹理。因此，尽可能把多张小纹理合并到一张大纹理（Atlas）中是一个好主意。

### 利用网格的顶点数据

但有时，除了纹理不同外，还有对于不同的物体，它们在材质上还有一些微小的参数变化，例如颜色不同、某些浮点参数不同。但铁定律是，**不管是动态批处理还是静态批处理，它们的前提都是要使用同一个材质**。是同一个，而不是同一种，也就是说它们指向的材质必须是同一个实体。这意味着，只要我们调整了参数，就会影响到所有使用这个材质的对象。那么想要微小的调整怎么办呢？由于Unity中的规定非常死，那么我们只好想些“歪门邪道”，其中一种就是使用网格的顶点数据（最常见的就是顶点颜色数据）。

前面说过，经过批处理后的物体会被处理成一个VBO发送给GPU，VBO中的数据可以作为输入传递给Vertex Shader，因此我们可以巧妙地对VBO中的数据进行控制，从而达到不同效果的目的。一个例子是，还是之前的森林，所有的树使用了同一种材质，我们希望它们可以通过动态批处理来实现，但不同树的颜色可能不同。这时我么可以利用网格的顶点数据来调整。具体方法，可以参见后面会写的一篇文章。

但这种方法的缺点就是会需要更多的内存来存储这些用于调整参数用的顶点数据。没办法，永远没有绝对完美的方法。



# 带宽优化


## 减少纹理大小

之前提到过，使用Texture Atlas可以帮助减少Draw Calls，而这些纹理的大小同样是一个需要考虑的问题。在这之前要提到一个问题就是，所有纹理的长宽比最好是正方形，而且长度值最好是2的整数幂。这是因为有很多优化策略只有在这种时候才可以发挥最大效用。

Unity中查看纹理参数可以通过纹理的面板：     
![](http://img.blog.csdn.net/20141226160748390?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2FuZHljYXQxOTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)



而调整参数可以通过纹理的Advance面板：       
![](http://img.blog.csdn.net/20141226160826664?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2FuZHljYXQxOTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)




上面各种参数的说明可以参见[文档](http://docs.unity3d.com/Manual/class-TextureImporter.html)。其中和优化相关的主要有“Generate Mip Maps”、“Max Size”和“Format”几个选项。

“Generate Mip Maps”会为同一张纹理创建出很多不同大小的小纹理，构成一个纹理金字塔。而在游戏中可以根据距离物体的远近，来动态选择使用哪一个纹理。这是因为，在距离物体很远的时候，就算我们使用了非常精细的纹理，但肉眼也是分辨不出来的，这种时候完全可以使用更小、更模糊的纹理来代替，而这大量可以节省访问的像素的数目。但它的缺点是，由于需要为每一个纹理建立一个图像金字塔，因此它会需要占用更多的内存。例如上面的例子，在勾选“Generate Mip Maps”前，内存占用是0.5M，而勾选了“Generate Mip Maps”后，就变成了0.7M。除了内存的占用以外，一些时候我们也不希望使用[Mipmaps](http://zh.wikipedia.org/wiki/Mipmap)，例如GUI纹理等。我们还可以在面板中查看生成的Mip Maps：        
![](http://img.blog.csdn.net/20141226212907562?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2FuZHljYXQxOTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)



[Unity](http://docs.unity3d.com/Manual/ViewModes.html)中还提供了查看场景中物体的Mip Maps的使用情况。更确切的说是，展示了物体理想的纹理大小。其中红色表示这个物体可以使用更小的纹理，蓝色表示应该使用更大的纹理。        
![](http://img.blog.csdn.net/20141226213035909?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2FuZHljYXQxOTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)



“Max Size”决定了纹理的长宽值，如果我们使用的纹理本身超过了这个最大值，Unity会对其进行缩小来满足这个条件。这里再重复一点，所有纹理的长宽比最好是正方形，而且长度值最好是2的整数幂。这是因为有很多优化策略只有在这种时候才可以发挥最大效用。

“Format”负责纹理使用的压缩模式。通常选择这种自动模式就可以了，Unity会负责根据不同的平台来选择合适的压缩模式。而对于GUI类型的纹理，我们可以根据对画质的要求来选择是否进行压缩，具体可以参见之前[关于画质的文章](http://blog.csdn.net/candycat1992/article/details/22794773)。


我们还可以根据不同的机器来选择使用不同分辨率的纹理，以便让游戏在某些老机器上也可以运行。


## 利用缩放

很多时候分辨率也是造成性能下降的原因，尤其是现在很多国内山寨机，除了分辨率高其他硬件简直一塌糊涂，而这恰恰中了游戏性能的两个瓶颈：过大的屏幕分辨率+糟糕的GPU。因此，我们可能需要对于特定机器进行分辨率的放缩。当然，这样会造成游戏效果的下降，但性能和画面之间永远是个需要权衡的话题。

在Unity中设置屏幕分辨率可以直接调用[Screen.SetResolution](http://docs.unity3d.com/ScriptReference/Screen.SetResolution.html)。实际使用中可能会遇到一些情况，[雨松MOMO有一篇文章](http://www.xuanyusong.com/archives/3205)讲了这种技术，可以去看看。

 

写在最后

这篇文章是总结性质的，因此对每种技术都没有进行非常详细的解释。强烈建议大家阅读文章开头给出的各种链接，写得都很好。

更多参考：         
[Unity3D性能优化总结](http://www.cnblogs.com/quansir/p/6370796.html)  
[厚积薄发| Unity优化](http://blog.csdn.net/UWA4D/article/category/6584886)    
[unity优化](http://www.unity.5helpyou.com/tag/unity%E4%BC%98%E5%8C%96)
[Unity优化脚本篇](http://blog.csdn.net/u013709166/article/details/54934931)
