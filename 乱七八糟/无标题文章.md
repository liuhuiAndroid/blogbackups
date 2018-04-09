##How to Use
使用方法

Here you can find information on how to use mAppWidget library and FAQ's. If you have any question or problem which is not outlined here, please submit it using our support form.
在这里你可以找到如何使用mAppWidget 库的方法和常见问题解答。如果你有任何其他问题，请提交给我们。

####Introduction
####介绍

mAppWidget is an Android library designed specifically to simplify the development of custom offline map applications. It uses tile engine for rendering the map on the screen.
mappwidget是一个专为简化自定义离线地图应用而设计的一个Android库。它使用tile 引擎在屏幕上绘制地图。

####Abstractions
####抽象

The offline map consists of an image that is sliced into tiles. Tiles are organised into several zoom levels. Zoom levels start from 0. At 0 zoom level, the map image’s dimensions are 1x1 pixels. At each next zoom level, the image size is twice increased.
离线地图由一个被切割成拼图的图像组成。拼图被组织成多个缩放级别。缩放级别从0开始，在0的缩放级别，地图图像的尺寸是1x1像素。在每一个下一个缩放级别，图像大小增加了两倍。

A map can have layers and map objects.
地图可以有多个图层和地图对象。

A layer is an abstraction that holds map objects. Layers can be visible or invisible. If a layer is invisible, objects that belong to this layer will not be displayed on the map.
层是抽象的，用来保存地图对象。层可以显示或者隐藏。如果一个层是不可见的，属于这个层的对象将不会显示在地图上。

A map object is the object that can be displayed on the map. A Drawable object is used to display it. Map objects can be added to any layer. The coordinates of the object are set in pixels.
地图对象可以显示在地图上。可以用一个Drawable 对象来显示它。地图对象可以添加到任意层。对象的坐标以像素为单位。

To define an object’s location, coordinates use the image originally used for tile generation. The top left corner of the image is defined as having (0, 0) coordinates.
为了明确一个对象的位置，坐标使用网格生成的原始图像。图像的左上角被定义为(0,0)坐标。

To define the location of a map object within the image, place the cursor at the spot and check its coordinates (you can use standard image editor). For example, point Bin the image has coordinates of (350, 200) pixels.
要在图像中定义地图对象的位置，将光标移动到该位置并检查其坐标(您可以使用标准的图像编辑器)。例如，该图像的坐标为(350,200)像素。

####Environment Setup
####环境设置

First, you need to create a new project in the Eclipse development environment. The project can then be integrated into your app project by using EclipseпїЅs project management tools. A step-by-step guide below shows how to correctly create a map project.
首先，您需要在Eclipse开发环境中创建一个新项目。通过使用EclipseпїЅs项目管理工具将这个项目集成到你的应用项目。下面这个分步指南展示了如何正确地创建一个map项目。

####FAQ
####常见问题

How to attach javadoc to the library?
如何将javadoc附加到库中?

How to calibrate offline map in order to use geographic aware methods?
如何校准离线地图，为了使用地理感知方法?

How to use Map Slicing Tool?
如何使用地图切片工具?

How to create map assets?
如何创建地图资源?

How to display map object on the map?
如何在地图上显示地图对象?

How to display map object using the geographic coordinates?
如何使用地理坐标显示地图对象?

How to get user’s position?
如何获得用户的位置?

How to handle touch events on the map objects?
如何处理地图对象上的触摸事件?

How to perform some actions before and after zoom has occurred?
如何在缩放前后执行一些动作?

How to scroll the map to some location?
如何将地图滚动到某个位置?

How to show current user’s location?
如何显示用户当前的位置?

How to change the way in which location pointer is pointing?
如何改变位置指针指向的方向?

How to show/hide map objects of any type?
如何显示/隐藏任何类型的地图对象?

How to use more than one map in a single application?
如何在一个application中使用多个地图?

How to zoom in/out?
如何放大/缩小地图?

What to do when user has touched more than one map object at a time?
当用户一次触摸多个地图对象时该怎么办?








