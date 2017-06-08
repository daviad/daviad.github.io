---
layout: post
title:  "opengl"
---
### 前言：  
记得大概2012年的时候，接触了OpenGL。那个时候，iOS刚入门，在进一步学ios的guide时候，知道了OpenGL这个概念。了解到Apple core animation 的底层是用OpenGLes实现的。记得当时有位同事有本OpenGL的书，应该是红宝书。就看了起来。概念太多，只是懵懵懂懂的，了解了一些OpenGL的相关概念。而后，又在苹果的帮助文档上看了一些相关的知识。也在网上看了一些教程。 了解到OpenGLes 是OpenGL的子集， 内心盘算，学会了OpenGL 就可以了，参照例子能画个三角形，贴上图。也许是自己能力不够，也许是工作中基本用不上，自己也有惰性，一直未能深入理解其含义。一直掌握不了，不得要领，内心是紧张、焦躁的。  去年，接触到滤镜，GUPImage，发现这个库又是基于OpenGLes的。
今年，年初再一次，将OpenGL，OpenGLES，webgl，列入了学习计划。在网上，再次找资料学习。一下是对各个参考资料的总结，以便以后温故。  
- OpenGL ES应用开发实践 指南 iOS卷  
我通读了两遍，讲了ios 中OpenGL ES 的使用。Apple 做了很多封装，这本书都做了原理介绍。前几章介绍是OpenGLes的基本渲染，后面主要偏向了游戏。我看的就粗略了。建议所有学习openges的都看一看
- 比较适合的blog  
1. 从零开始构建IOS的OpenGL应用http://mississi.blog.163.com/blog/#m=0&t=1&c=fks_084070086082087070086081087095085086083071092095084067085 这个blog系列，讲解的比较直接，没有深入介绍过多的细节，但是很清晰，非常适合，入门学习，掌握基本用法，等有了一定认识后再做深入学习。  
2. 罗朝辉 OpenGL ES 一系列的文章 很基础的8篇  例子，浅显易懂。入门的基础http://blog.csdn.net/kesalin/article/category/1288827，这个blog，非常好，自己动手实现一边例子。opengles的基础应该是有了。
3. OpneglES http://www.jianshu.com/nb/2135411，看完了1，2，再看看这个3，是一些实例。
4. 其他man手册

目前，这些我基本都看过，但是至是动手实现了部分，个人感觉有点方向了，算是基本入门。一下写点自己的学习小结。
### 学习小结
##### opengl 渲染 一句话总结
网格mesh 顶点数据提供（几何形状位置坐标，纹理坐标，法线坐标等坐标信息）产生模型model，  灯光light开启提供灯光效果，纹理texture开启提供纹理效果 +  矩阵变换，视角改变等数学计算，使  GPU draw 出 图像。  

mesh 网格 包含的数据：顶点位置数据（模型的形状）、顶点纹理数据（贴图）、顶点法线（光着）等其他，比如：box 边界。
 ---》model

虽然，很多时候，地形使用terrain，没有使用mesh。但地形terrain 仍然是mesh的一个变体。

   self.baseEffect.texture2d0.name = 
      self.modelManager.textureInfo.name;
   self.baseEffect.texture2d0.target = 
      self.modelManager.textureInfo.target;
*texture2d0 这几行代码就会将纹素 和 点纹理  联系起来 产生纹理效果

 self.baseEffect.light0.enabled = GL_TRUE;
   self.baseEffect.light0.ambientColor = 
*light0, *light1, 这是 和顶点法线结合 产生灯光效果

 再次深入学习 读懂  直至掌握
1.索引顶点
2.modelplist
glDrawElements(
            GL_LINE_STRIP,
            (GLsizei)numberOfIndices,
            GL_UNSIGNED_SHORT,
            indices + firstIndex);
一个大网格 通过 代码 分成多个小网格 

transform 第五章
单位矩阵：一个与参考坐标系相同的坐标系的矩阵
矩阵级联有时被称为 矩阵乘法

认真学习第五章

第一章 使用现代移动图形硬件 p5
以前OpenGL ES 为GPU，CPU 执行内存复制，过时！
现在OpenGL ES 为GPU，CPU 之间定义了 缓存（buffer）。高效！

缓存中放的是什么类型的数据（几何数据，颜色，灯光等）并不重要
缓存的使用步骤。
1.生成generate 				glGenBuffers()
2.绑定bind  					glBindBuffer()      
3.缓存数据buffer data		glBufferData() or glBufferSubData()
4.启用enable或禁止disabl 	glEnableVertexAttribArrary() or glDisableVertexAttribArraray()
5.设置指针set pointers		glVertexAttribPointer()
6.绘图draw					glDrawArrays() or glDrawElments()
7.删除delete				glDeleteBuffers()

特殊的缓存：帧缓存 frame buffer   是GUP 存放渲染好的2D图像数据的地方。 一般由系统自动完成以上 缓存使用的步骤。


现代GPU 对浮点数据做了专门的优化 即使使用的是其他类型的顶点数据也会被转换为浮点值

把程序这的几何数据，转换为屏幕上的图像的过程叫渲染。

第二章 让硬件为你工作
帧缓存 可以配置多个渲染缓存 来接收不同类型的输出 。 
在ios中 必须配置一个 像素颜色 渲染缓存   （是一种帧缓存）。 由ios系统的core animation 在内部 自动的操作，这个帧缓存。
GLKView 封装了帧缓存和层管理

kEAGLDrawablePropertyRetainedBacking  NO  告诉Core Animation 不要试图保留以前绘制的图像作以后重用。

GLKVertexAttribArraryBuffer  封装了 使用 顶点属性数组缓存（简称顶点缓存）的所有7个步骤，请回忆缓存的使用步骤

第三章纹理
纹理也是缓存 用来保存图像颜色元素值
纹理坐标系有一个命名为S和T的2D轴。 一个纹理有多个纹素组成。类比图像。
图像坐标系有一个命名为X和Y的2D轴。一个图像有多个像素组成。
纹素 
当然opengl 也支持 1D和3D 纹理。

视口：帧缓存中的坐标位置。viewport

点阵化：转换几何形状数据为帧缓存中的颜色像素
片元：每个颜色像素


