---
title: UGUI之Content自动排序与页面滑动
date: <strong> 2017-09-16 23:10:55 </strong>
categories:
- Unity
tags: 
- UGUI
- GridLayoutGroup
- ContentSizeFitter
cover: https://gitee.com/dioooooooor/ImageHosting/raw/master//UnityBlackLarge.png
---

## 功能需求：
* **UI界面的内容可以自动排序，超出界面的部分可以滑动查看**
![](https://raw.githubusercontent.com/Dioooooooor/BlogImageHosting/master/image/49456984.jpg)

<!-- more -->

## 实现方法：
### 右键创建Scroll View组件
![](https://gitee.com/dioooooooor/ImageHosting/raw/master//47083771.jpg)

### 修改Scroll View组件的大小，我这边设置的为全屏，并把背景设为了黑色
![](https://gitee.com/dioooooooor/ImageHosting/raw/master//2599429.jpg)

### 把Scroll View组件上的Scroll Rect组件的Content换成Viewport,并且将Viewport设置为null，然后设置滑动条的可视条件，我选择的是permanent(常驻)，这样滑动条就不会隐藏
![](https://gitee.com/dioooooooor/ImageHosting/raw/master//54708227.jpg)

### 在Viewport上添加ContentSizeFitter和GridLayoutGroup脚本，并且将ContentSizeFitter组件上的Horizontal Fit和Vertical Fit选项改成Preferred Size（首选大小）
![](https://gitee.com/dioooooooor/ImageHosting/raw/master//40746207.jpg)

### 完成上面的设置后，一个最基本的可拖动的自动排序界面设置就已经完成了，我们来验证一下，为了验证方便，我们在Content上加上Image，让内容可视化，然后调整GridLayoutGroup脚本上的Cell Size,让Content更大，然后我们运行程序测试
![](https://raw.githubusercontent.com/Dioooooooor/BlogImageHosting/master/image/49456984.jpg)

## 更多细节
### 步骤3
* **[Content](https://docs.unity3d.com/ScriptReference/UI.ScrollRect-content.html)** 是可拖动区域，就是当拖动时可以动的部分，所有需要拖动的UI都要放到这个组件下面
* **[Viewport](https://docs.unity3d.com/ScriptReference/UI.ScrollRect-viewport.html)** 是可视范围，当我们想要可拖动范围只有部分区域的时候，可以进行以下操作完成
    1. 在Scroll View组件下添加一个Panel
    2. 把Viewport放到Panel下面
    3. 在Panel上添加Mask组件
    4. 把Scroll Rect中的Viewport设置为Panel 


* 另外，也可以直接在Scroll View上添加Mask,这样就不用设置这个选项了。
    ![](https://raw.githubusercontent.com/Dioooooooor/BlogImageHosting/master/image/58299777.jpg)
* **[Visibility](https://docs.unity3d.com/ScriptReference/UI.ScrollRect.ScrollbarVisibility.html)** 滑动条可视化条件
    * permanent(常驻)
    * Auto Hide(当显示的目标超出可视范围时显示,并且不会改变可视范围)
    * Auto Hide And Expand Viewport(当显示的目标超出可视范围时显示,并且限制可视范围)

### 步骤4

* **[ContentSizeFitter](https://docs.unity3d.com/ScriptReference/UI.ContentSizeFitter.html)** (调整RectTransform的大小以适应其内容的大小)， 如果不添加这个组件会导致动态创建的内容不现实，也不会按规则排序
    * ContentSizeFitter
        * Unconstrained 不进行任何调整(不进行调整的话，子物体会不显示)
        * MinSize 调整为最小
        * PreferredSize 调整为首选大小
* **[GridLayoutGroup](https://docs.unity3d.com/ScriptReference/UI.GridLayoutGroup.html)**(在网格中布置子布局元素)这个部分就是我们用来排列子物体的组件
    * [padding](https://docs.unity3d.com/ScriptReference/UI.LayoutGroup-padding.html) 用于添加子布局元素的填充,可以设置子物体的现实范围
    * cellSize 网格中每个单元格使用的大小
    * spacing 在网格中的布局元素之间使用的间距
    * startAxis 子物体首先放在哪个轴上
    * startCorner 第一个单元格应放在哪个角落
    * childAlignment 用于布局组中的子布局元素的对齐方式
    * constraint 用于GridLayoutGroup的约束
        * Flexible 不限制
        * FixedColumnCount 限制列数
        * FixedRowCount 限制行数
---
<br/>
<font style="font-weight:bold;" color = red size = "5">不乱于心，不困于情。不畏将来，不念过往。如此，安好。</font>
<br>

