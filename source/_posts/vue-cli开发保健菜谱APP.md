---
title: vue-cli开发保健菜谱APP
date: 2018-12-10 11:02:21
tags: vue
cotegory: Vue
---

## 保健菜谱

[vue 代码链接: https://gitee.com/laoxue6699/meishi](https://gitee.com/laoxue6699/meishi)

[uni-app 代码链接: https://gitee.com/laoxue6699/health_menu](https://gitee.com/laoxue6699/health_menu)

### 项目介绍

项目主要功能是为人们提供常用保健菜品的制作方法。通过视频、图片、文字展示菜品的制作过程。提供列表查询和全文搜索功能方便查询。可以收藏自己喜爱的视频和图文菜谱。

<!--more-->

![输入图片说明](http://pw59ntmi6.bkt.clouddn.com/meishi01.png)

### 项目架构

- 通过 Scrapy 爬取后台需要的数据，经过数据清洗和加工成 json 数据。
-     利用json-server构建RESTful接口API。
-     使用vue-li2脚手架工具构建前端APP项目。
-     利用vuejs组件化技术，渐进构建项目界面。
-     利用Axios请求远程数据。
-     利用vue-router模块组织管理路由。

### 技术要领

#### 封装

- 在小型项目中，诸如 Axios 请求数据用一行代码就可以搞定，就没不要封装。在大型项目中，如果大到需要用 vuex 来管理状态，还是需要一定的代码封装的。

#### 跨域请求

- 开发阶段可能会遇到跨域请求问题。通过修改 config 中 index.js 的代理参数得以解决。

#### 组件拆分

- 项目总体的组件可以划分成两类：路由组件和非路由组件。路由组件在 router 中配置路由，非路由组件在模板中嵌套。
- 不能凡组件必拆分，要坚持“高内聚、低耦合”的原则。对拆分后影响代码整体性理解或需要复杂配置的，尽量不要拆分。

#### 传参

- 传参主要有三种：路由传参、组件传参、vuex 状态管理。小项目可以不用 vuex 进行状态管理。
- 路由传参注意 path 和 query 配合，name 和 params 配合。
- 组件传参，父传子用属性传值 props 接收。子传父通过\$emit 传递事件和值，父组件通过监听事件来接收传值，可以单线联系。

#### 生命周期

- 一般可以请求数据的代码放在 vue 的生命周期 created 钩子函数中。

#### 列表渲染

- v-for 可以嵌套遍历。

#### 动态样式绑定

- 诸如底部选项卡的激活状态可以用动态样式绑定的方法来实现。

#### 随机请求数据

- 传统列表的数据都采取上拉加载下拉刷新的方式，对于数据量大的也可以先随机生成取值范围再请求数据的方法，这样可以给用户带来不同的体验。

#### 全文搜索

可以预定义一些常用的搜索关键词供用户点按，既可以引导用户输入规范的关键词，也可以简化操作，提高用户体验。

#### 控制 video 播放

在 vue 组件中，使用`v-show`来决定播放界面是否显示，当关闭播放组件时，正在播放的视频仍在播放。这是需要使用\$refs 参数选取`video`元素来同时关闭视频播放。

#### poster 不兼容的处理

vue 在使用 HTHL5 的 `video`元素播视频时，poster 默认的封面在真机上显示不出来，需要在模板中`video`元素的位置预先加载与 poster 相同的封面图片，才不会显示白板背景。

#### 收藏功能的实现

收藏视频和图文菜谱，需要用到 HTML5 的`localStorage`进行存储，为了节省空间，只需要存储收藏菜谱的`id`即可。由于保存的数据格式都是字符串，所以取出时还要转换成数组，才能做进一步的操作。用先进先出法限制存储的条目数。

### 项目界面

![输入图片说明](http://pw59ntmi6.bkt.clouddn.com/meishi02.png)

![输入图片说明](http://pw59ntmi6.bkt.clouddn.com/meishi03.png)

![输入图片说明](http://pw59ntmi6.bkt.clouddn.com/meishi04.png)

![输入图片说明](http://pw59ntmi6.bkt.clouddn.com/meishi05.png)

![输入图片说明](http://pw59ntmi6.bkt.clouddn.com/meishi06.png)

![输入图片说明](http://pw59ntmi6.bkt.clouddn.com/meishi07.png)

![输入图片说明](http://pw59ntmi6.bkt.clouddn.com/meishi08.png)

![输入图片说明](http://pw59ntmi6.bkt.clouddn.com/meishi09.png)

## Build Setup

```bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build

# build for production and view the bundle analyzer report
npm run build --report
```
