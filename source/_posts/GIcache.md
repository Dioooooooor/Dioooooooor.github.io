---
title: GI cache（全局光照缓存）
date: <strong> 2017.09.14 </strong>
categories:
- Unity
tags: 
- UnityEditor
- Unity优化
description: <blockquote>全球照明（GI）系统使用GI缓存来预先计算实时GI，以及烘焙静态光地图，光探测器和反射探测器时存储中间文件。 缓存在计算机上的所有Unity项目之间共享，因此具有相同内容和相同版本的照明系统(Enlighten)的项目可以共享文件并加快后续版本。</blockquote>
cover: https://raw.githubusercontent.com/Dioooooooor/BlogImageHosting/master/image/UnityBlackLarge.png
---


*  The GI cache is used by the Global Illumination (GI) system to store intermediate files when precomputing real-time GI, and when baking Static Lightmaps, Light Probes and Reflection Probes. The cache is shared between all Unity projects on the computer, so projects with the same content and same version of the lighting system (Enlighten) can share the files and speed up subsequent builds.

* 全球照明（GI）系统使用GI缓存来预先计算实时GI，以及烘焙静态光地图，光探测器和反射探测器时存储中间文件。 缓存在计算机上的所有Unity项目之间共享，因此具有相同内容和相同版本的照明系统（Enlighten）的项目可以共享文件并加快后续版本。


* **功能**：主要负责存储全局的光照缓存，这样每次加载场景的时候不需要重新烘培。

* 但是当地图较大，光照较多的时候就会导致占用内存太大，如图：  
![](https://raw.githubusercontent.com/Dioooooooor/BlogImageHosting/master/image/83211983.jpg)

* 如果不再需要这些光照信息了，我们可以直接选择<font color = red>删除这些缓存</font>，面板中选择：**Edit > Preferences > GI Cache > Clean Cache**    
![](https://raw.githubusercontent.com/Dioooooooor/BlogImageHosting/master/image/83909458.jpg)  

* 或者当你觉得还需要这些光照信息，不能马上删除的时候，我们就可以<font color = red>调整光照缓存的存储位置</font>,选择：**Edit > Preferences > GI Cache > Custom cache location**  
![](https://raw.githubusercontent.com/Dioooooooor/BlogImageHosting/master/image/73177986.jpg)   

* <font color = red>当你觉得C盘使用空间突然暴涨的时候，可以看看，是否是因为太多的光照缓存占用了你的磁盘空间</font>
