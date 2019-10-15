---
title: LoopBack3.0创建API接口实战
date: 2019-09-20 13:23:35
category:
  - 原创
tags:
  - loopBack
  - nodejs
  - express
  - RESTful
---

LoopBack 是一个高度可扩展的开源 Node.js 框架，它使您能够以很少的编码或不编写任何代码来创建动态的端到端 REST API。

<iframe src="//player.bilibili.com/player.html?aid=68852902&cid=119328164&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>

[到 bilibili.com 网站观看全部视频](https://www.bilibili.com/video/av68852902/)

<!-- more -->

LoopBack 框架是由一组 Node.js 的模块构成的。你可以单独使用这些模块或把它们组合在一起使用。 应用通过 LoopBack model API 可以使用以下三种方式访问数据。

- 将模型作为一个标准的 Node 对象使用
- 通过 HTTP REST API 调用
- 通过封装好的 API SDK，包括 iOS, Android 和 Angular

应用程序通过 LoopBack model API 用以上三种方式查询数据，储存数据，上传文件，发送 email, 推送消息，注册/登陆用户等远程或本地的服务。用户也可以通过 Strong Remoting 将后端的 API 通过 REST, WebSocket(或其他传输协议)供客户端调用。

## 一、项目概述

这是官方给出的一个关于咖啡店点评的示例。

咖啡店点评是一个网站，我们可以用来发布咖啡店的评论。有三个数据模型：

- CoffeeShop 咖啡店
- Review 评论
- Reviewer 评论者

它们有如下关系：

- CoffeeShop 拥有多个 review
- CoffeeShop 拥有多个 reviewer
- review 属于一个 CoffeeShop
- review 属于一个 reviewer
- reviewer 拥有多个 review

一般来说，用户可以创建，编辑，删除和阅读咖啡店的评论，并通过 ACLs 指定基本规则和权限：

- 任何人都可以阅读评论，但必须先登录才能创建，编辑或删除它们。
- 任何人都可以注册为用户，然后能够登录或者注销。
- 登录用户可以创建新的评论，编辑或删除自己的评论，但是他们不能修改咖啡店相关内容。

## 二、创建项目

### 安装 Loopback CLI 工具

在 nodejs 环境下安装 Loopback CLI 工具。

```
npm install -g loopback-cli
```

安装成功后，输入 lb -help 查看 Loopback CLI 工具的命令：

```
lb -help
Available commands:
  lb acl
  lb app
  lb bluemix
  lb boot-script
  lb datasource
  lb export-api-def
  lb middleware
  lb model
  lb oracle
  lb property
  lb relation
  lb remote-method
  lb soap
  lb swagger
  lb zosconnectee
```

### 创建 LoopBack 项目

在命令行输入：

```
lb
```

回车后按照提示输入项目名称，其他选择回车。

```
? 您的应用程序的名称是什么？ CoffeeShop
? 输入目录名称以包含项目： CoffeeShop
   create CoffeeShop/
     info 将工作目录更改为 CoffeeShop

? 您想要使用哪个版本的 LoopBack？ 3.x (Active Long Term Support)
? 您想要什么种类的应用程序？ api-server (使用本地用户认证的 LoopBack API 服务器)
正在生成 .yo-rc.json
```

安装成功后按照提示操作后续步骤：

```
后续步骤：

  将目录更改为您的应用程序
    $ cd CoffeeShop

  在应用程序中创建模型
    $ lb model

  运行应用程序
    $ node .
```

进入项目根目录，运行应用程序。

```
node .
hsts deprecated The "includeSubdomains" parameter is deprecated. Use "includeSubDomains" (with a capital D) instead. node_modules/loopback/lib/server-app.js:74:25
Web server listening at: http://localhost:3000
Browse your REST API at http://localhost:3000/explorer
```

打开浏览器，输入上面提示的地址可以访问 LoopBack 的 API 接口测试页面。由于我们还没有创建任何模型，所以只能在页面上看到框架自带的 user 的 API 调用接口。

![](http://huangxiaoman.cn/%E6%88%AA%E5%B1%8F2019-09-20%E4%B8%8B%E5%8D%882.14.34.png)

在项目根目录命令行通过 control+C 退出服务。

## 三、连接数据源

LoopBack 可以自动连接市面上绝大多数数据源。我们一 MongoDb 为例来为项目添加数据持久化库。

```
lb datasource
? 输入数据源名称： md
? 为 md 选择连接器： MongoDB （StrongLoop 支持）
? Connection String url to override other settings (eg: mongodb://username:password@hostname:port/da
tabase):
? host: localhost
? port: 27017
? user:
? password: [hidden]
? database: cshop
? 安装 loopback-connector-mongodb@^4.0.0 Yes
+ loopback-connector-mongodb@4.2.0
added 7 packages from 11 contributors in 2.342s
```

首先，必须保证开发环境有 MongoDB 数据库并已经启动，数据库不需要实现创建，框架会根据输入的数据库名称自动创建。由于 MongoDB 的连接插件没有安装，所以这里选择 Yes 来安装。

在项目目录的 server 子目录下有如下文件：

```
server % tree
.
├── boot
│   ├── authentication.js
│   └── root.js
├── component-config.json
├── config.json
├── datasources.json
├── middleware.development.json
├── middleware.json
├── model-config.json
└── server.js

```

其中，`datasources.json` 是数据源的配置文件，我们在代码编辑器中打开这个文件，会看到我们创建的 md 数据源。

```
{
  "db": {
    "name": "db",
    "connector": "memory"
  },
  "md": {
    "host": "localhost",
    "port": 27017,
    "url": "",
    "database": "cshop",
    "password": "",
    "name": "md",
    "user": "",
    "connector": "mongodb"
  }
}
```

由于框架原先定义了一个数据源 db，我们不需要了，把 db 数据源删掉，将 md 数据源改名为 db。修改之后代码如下：

```
 {
  "db": {
    "host": "localhost",
    "port": 27017,
    "url": "",
    "database": "cshop",
    "password": "",
    "name": "md",
    "user": "",
    "connector": "mongodb"
  }
}
```

创建好数据源，框架帮我们自动连接数据库，我们对数据的持久化操作也会自动保存到数据库中。

## 四、创建模型

### CoffeeShop 模型

<table>
<tr>
<th>属性名</th>
<th>属性类型</th>
<th>是否必填</th>
</tr>
<tr>
<td> name </td>
<td> String </td>
<td> y </td>
</tr>
<tr>
<td> city </td>
<td> String </td>
<td> y </td>
</tr>
</table>

```
lb model
? 请输入模型名称： CoffeeShop
? 选择要向其附加 CoffeeShop 的数据源： db (mongodb)
? 选择模型的基类 PersistedModel
? 通过 REST API 公开 CoffeeShop？ Yes
? 定制复数形式（用于构建 REST URL）：
? 公共模型或仅服务器？ 公共
现在添加一些 CoffeeShop 属性。

在完成时输入空的属性名称。
? 属性名称： name
? 属性类型： string
? 是否为必需？ Yes
? 缺省值[对于无，保留为空白]：

下面添加另一个 CoffeeShop 属性。
在完成时输入空的属性名称。
? 属性名称： city
? 属性类型： string
? 是否为必需？ Yes
? 缺省值[对于无，保留为空白]：

下面添加另一个 CoffeeShop 属性。
在完成时输入空的属性名称。
? 属性名称
```

### review 模型

<table>
<tr>
<th>属性名</th>
<th>属性类型</th>
<th>是否必填</th>
</tr>
<tr>
<td> date </td>
<td> date </td>
<td> y </td>
</tr>
<tr>
<td> rating </td>
<td> number </td>
<td> n </td>
</tr>
<tr>
<td> comments </td>
<td> string </td>
<td> y </td>
</tr>
</table>

```
lb model
? 请输入模型名称： Review
? 选择要向其附加 Review 的数据源： db (mongodb)
? 选择模型的基类 PersistedModel
? 通过 REST API 公开 Review？ Yes
? 定制复数形式（用于构建 REST URL）：
? 公共模型或仅服务器？ 公共
现在添加一些 Review 属性。

在完成时输入空的属性名称。
? 属性名称： date
? 属性类型： date
? 是否为必需？ Yes
? 缺省值[对于无，保留为空白]：

下面添加另一个 Review 属性。
在完成时输入空的属性名称。
? 属性名称： rating
? 属性类型： number
? 是否为必需？ No
? 缺省值[对于无，保留为空白]：

下面添加另一个 Review 属性。
在完成时输入空的属性名称。
? 属性名称： comments
? 属性类型： string
? 是否为必需？ Yes
? 缺省值[对于无，保留为空白]：

下面添加另一个 Review 属性。
在完成时输入空的属性名称。
? 属性名称：
```

### reviewer 模型

reviewer 模型继承自框架自带的 user 模型。在选择模型的基类时不能选择 `PersistedModel`，而要选择 `user`，不用添加任何属性，直接回车即可。

```
lb model
? 请输入模型名称： Reviewer
? 选择要向其附加 Reviewer 的数据源： db (mongodb)
? 选择模型的基类 User
? 通过 REST API 公开 Reviewer？ Yes
? 定制复数形式（用于构建 REST URL）：
? 公共模型或仅服务器？ 公共
现在添加一些 Reviewer 属性。

在完成时输入空的属性名称。
? 属性名称：
```

在项目根目录命令行通过 node . 来启动服务。在浏览器中再次打开 http://localhost:3000/explorer。

可以看到 API 调试界面除了先前的 user 接口外，增加了我们刚刚创建的 3 个模型的接口。

![](http://huangxiaoman.cn/%E6%88%AA%E5%B1%8F2019-09-20%E4%B8%8B%E5%8D%883.25.55.png)

## 五、定义模型之间的关系

LoopBack 支持许多不同类型的模型关系：BelongsTo, HasMany, HasManyThrough, and HasAndBelongsToMany 等等。根据项目需求可以定义如下关系：

- CoffeeShop `HasMany` review
- CoffeeShop `HasMany` reviewer
- review `BelongsTo` CoffeeShop
- review `BelongsTo` reviewer
- reviewer `HasMany` review

CoffeeShop 拥有多个 review，没有中间模型和外键。

```
lb relation
? 选择从中创建关系的模型： CoffeeShop
? 关系类型： has many
? 选择与之创建关系的模型： Review
? 输入关系的属性名称： reviews
? （可选）输入定制外键：
? 需要直通模型？ No
? 允许在 REST API 中嵌套关系： No
? 禁止包含关系： No
```

CoffeeShop 拥有多个 reviewer，没有中间模型和外键。

```
lb relation
? 选择从中创建关系的模型： CoffeeShop
? 关系类型： has many
? 选择与之创建关系的模型： Reviewer
? 输入关系的属性名称： reviewers
? （可选）输入定制外键：
? 需要直通模型？ No
? 允许在 REST API 中嵌套关系： No
? 禁止包含关系： No
```

review 属于一个 CoffeeShop，没有中间模型和外键。

```
lb relation
? 选择从中创建关系的模型： Review
? 关系类型： belongs to
? 选择与之创建关系的模型： CoffeeShop
? 输入关系的属性名称： coffeeShop
? （可选）输入定制外键：
? 允许在 REST API 中嵌套关系： No
? 禁止包含关系： No
```

review 属于一个 reviewer，外键是 `publisherId`，没有中间模型。

```
lb relation
? 选择从中创建关系的模型： Review
? 关系类型： belongs to
? 选择与之创建关系的模型： Reviewer
? 输入关系的属性名称： reviewer
? （可选）输入定制外键： publisherId
? 允许在 REST API 中嵌套关系： No
? 禁止包含关系： No
```

reviewer 拥有多个 review，外键是 publisherId，没有中间模型。

```
lb relation
? 选择从中创建关系的模型： Reviewer
? 关系类型： has many
? 选择与之创建关系的模型： Review
? 输入关系的属性名称： reviews
? （可选）输入定制外键： publisherId
? 需要直通模型？ No
? 允许在 REST API 中嵌套关系： No
? 禁止包含关系： No
```

我们在项目根目录的 common 子目录的 model 目录下可以看到我们创建的与模型相关的 6 个文件：

```
common % tree
.
└── models
    ├── coffee-shop.js
    ├── coffee-shop.json
    ├── review.js
    ├── review.json
    ├── reviewer.js
    └── reviewer.json
```

在 coffee-shop.json 文件中定义的关系：

```
 "relations": {
    "reviews": {
      "type": "hasMany",
      "model": "Review",
      "foreignKey": ""
    },
    "reviewers": {
      "type": "hasMany",
      "model": "Reviewer",
      "foreignKey": ""
    }
  },
```

在`review.json`文件中定义的关系：

```
"relations": {
    "coffeeShop": {
      "type": "belongsTo",
      "model": "CoffeeShop",
      "foreignKey": ""
    },
    "reviewer": {
      "type": "belongsTo",
      "model": "Reviewer",
      "foreignKey": "publisherId"
    }
  },
```

在`reviewer.json`文件中定义的关系：

```
"relations": {
    "reviews": {
      "type": "hasMany",
      "model": "Review",
      "foreignKey": "publisherId"
    }
  },
```

## 六、定义操作权限

loopback 应用通过模型访问数据，因此控制对数据的访问意味着对模型进行权限的控制：也就是说，指定什么角色可以在模型上执行读取和写入数据的方法。loopback 权限控制由权限控制列表或 ACL 决定。

根据项目需求，权限控制应执行以下规则：

任何人都可以阅读评论。但是创建、编辑和删除的操作必须在登录之后才有权限。
任何人都可以注册为用户，可以登录和登出。
登录用户可以创建新的评论，编辑或删除自己的评论。然而，他们不能修改咖啡店的评论。

首先，拒绝所有人操作所有接口，这通常是定义 ACL 的起点，因为您可以选择性地允许特定操作的访问。

```
lb acl
? 选择要应用 ACL 条目的模型： （所有现有模型）
? 选择 ACL 作用域： 所有方法和属性
? 选择访问类型： 全部（匹配所有类型）
? 选择角色 所有用户
? 选择要应用的许可权 明确拒绝访问
```

现在允许所有人对 reviews 进行读操作。

```
lb acl
? 选择要应用 ACL 条目的模型： Review
? 选择 ACL 作用域： 所有方法和属性
? 选择访问类型： 读取
? 选择角色 所有用户
? 选择要应用的许可权 明确授权访问
```

允许通过身份验证的用户对 coffeeshops 进行读操作，也就是说，已登录的用户可以浏览所有咖啡店。

```
lb acl
? 选择要应用 ACL 条目的模型： CoffeeShop
? 选择 ACL 作用域： 所有方法和属性
? 选择访问类型： 读取
? 选择角色 任何已认证的用户
? 选择要应用的许可权 明确授权访问
```

允许经过身份验证的用户对 reviews 进行创建操作，也就是说，已登录的用户可以添加一条评论。

```
lb acl
? 选择要应用 ACL 条目的模型： Review
? 选择 ACL 作用域： 单个方法
? 输入方法名称 create
? 选择角色 任何已认证的用户
? 选择要应用的许可权 明确授权访问
```

使 review 的作者有权限（其“所有者”）对其进行任何更改。

```
lb acl
? 选择要应用 ACL 条目的模型： Review
? 选择 ACL 作用域： 所有方法和属性
? 选择访问类型： 写入
? 选择角色 拥有该对象的用户
? 选择要应用的许可权 明确授权访问
```

## 七、自动填充字段内容

我们需要用户在添加评论时，自动填充日期字段的内容为当前的日期。同时，由于评论和评论者之间是通过`publisherId`这个外键进行关联的，用户在添加评论时，需要将评论者的用户 Id 作为其内容进行填充。

我们将定义一个远程钩子，每当在 Review 模型上调用 create()方法时（在创建新的评论时），它将被调用。

通常，我们可以定义两种远程钩子：

- `beforeRemote()`在远程方法之前运行。
- `afterRemote()`在远程方法之后运行。

在这两种情况下，有两个参数可以供我们使用：一个与要钩子函数的远程方法匹配的字符串，和一个回调函数。

### 创建一个远程钩子

这里，我们将在 review 模型中定义一个远程钩子，具体来说是 Review.beforeRemote。

- 设置 publisherId 为请求中的 userId
- 设置日期为当前日期。

修改 common/models/review.js：

```
module.exports = function(Review) {
  Review.beforeRemote('create', function(context, user, next) {
    context.args.data.date = Date.now().toLocalString(); //转换成本地时间
    context.args.data.publisherId = context.req.accessToken.userId;
    next();
  });
};
```

在创建 Review 模型的新实例之前调用此函数。

## 八、后台管理

如何对 coffeeshops 的数据进行管理呢？这是我们常说的后台管理，需要一个管理员`admin`的角色来承担。
这部分内容将通过另外一篇文章进行介绍。
