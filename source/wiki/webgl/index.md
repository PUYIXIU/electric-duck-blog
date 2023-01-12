---
layout: wiki
wiki: WebGL
title: WebGL概述
order: 0
---

{% quot 本书源码访问地址 %}
{% link https://sites.google.com/site/webglbook/home/downloads   %}

# WebGL的起源

{% border PC常用三维图形渲染技术 color:yellow %}
- **Direct3D** 
  - 微软DirectX技术的一部分
  - 一套由微软控制的编程接口
  - 用于Windows平台
- **OpenGL**
  - 免费开源，可以在Macintoch、Linux、Windows系统上使用
  - 最初由SGI（Silicon Graphics Inc）开发
  - 1992年发布为开源标准
  - {% border OpenGL ES color:orange %}
    - OpenGL的一个特殊版本
    - 专门用于嵌入式设备
    - 2003~2004首次提出
    - 2007 ES升级2.0
    - 2012 ES升级3.0
    - WebGL基于OpenGL ES 2.0
    {% endborder %}
{% endborder %}
{% image ../../assets/image/WebGL/WebGL起源.png %}
      
{% border 着色器 color:green %}
OpenGL 2.0支持了一项非常重要的特性：{% mark 可编程着色器方法（programmable shader functions） color:purple %}  
{% border 着色器语言（shading language） color:cyan %}
OpenGL ES 2.0 基于 {% mark OpenGL着色器语言（GLSL） color:yellow %}。  
{% mark GLSL color:yellow %} 也称 {% mark OpenGL ES着色器语言（GLSL ES） color:orange %}。
WebGL也使用{% mark GLSL ES color:orange %}编写着色器。
{% endborder %}
{% endborder %}
{% border 更新与标准化 color:blue %}
OpenGL规范的更新与标准化由 {% mark Khronos组织 color:purple %} 负责。
由2011年发布了WebGL规范的第1个版本。
{% endborder %}

# WebGL程序的结构

WebGL页面包含三种语言：
- {% mark HTML5 color:blue %}
- {% mark JavaScript color:red %}
- {% mark GLSL ES color:purple %}

{% border 对比传统网页与WebGL网页的结构 color:cyan %}
通常GLSL ES是以字符串的形式在JS中编写的，  
因此WebGL网页与传统网页结构相同，  
只用到{% mark HTML文件 color:blue %} 和 {% mark JavaScript文件 color:red %}。
{% image ../../assets/image/WebGL/WebGL程序的结构.png %}
{% endborder %}
