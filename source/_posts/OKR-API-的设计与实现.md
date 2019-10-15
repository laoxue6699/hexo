---
title: loopback3 快速创建标准的RESTful数据访问接口
date: 2019-08-11 14:29:43
category:
  - 原创
tags:
  - loopBack
  - nodejs
  - express
  - RESTful
---

## 项目介绍

这是一个对 OKR 工作法进行辅助管理的全栈项目。项目实行前后端分离，后端用 loopBack3 框架开发 API 数据访问接口，前端分别用 Vue 全家桶、react 全家桶、un-app 框架、react-native 框架和 flutter 框架来实现移动端的跨平台应用。 涉及到当前前端开发的主流技术。本文介绍后端开发的全过程。

<!--more-->

## 设计思想

创建 OKR 管理 APP 需要搭建后台数据服务器，采用 RESTful 标准接口，利用 loopBack3 框架搭建。

### 为什么是 LoopBack？

LoopBack 框架是由一组 Node.js 的模块构成的。你可以单独使用这些模块或把它们组合在一起使用。 应用通过 LoopBack model API 可以使用以下三种方式访问数据。

- 将模型作为一个标准的 Node 对象使用
- 通过 HTTP REST API 调用
- 通过封装好的 API SDK，包括 iOS, Android 和 Angular

应用程序通过 LoopBack model API 用以上三种方式查询数据，储存数据，上传文件，发送 email, 推送消息，注册/登陆用户等远程或本地的服务。用户也可以通过 Strong Remoting 将后端的 API 通过 REST, WebSocket(或其他传输协议)供客户端调用。

### OKR 是什么？

OKR（Objectives and Key Results）即目标与关键成果法，是一套明确和跟踪目标及其完成情况的管理工具和方法，由英特尔公司发明。

OKR 的主要目标是明确公司和团队的“目标”以及明确每个目标达成的可衡量的“关键结果”。一本关于 OKR 的书将 OKR 定义为“一个重要的思考框架与不断发展的学科，旨在确保员工共同工作，并集中精力做出可衡量的贡献。 ”OKR 可以在整个组织中共享，这样团队就可以在整个组织中明确目标，帮助协调和集中精力。

### 需求定义

#### OKR 管理功能

- 用户需要注册登录才能进行操作。
- 用户注册信息：用户名、密码、email、所属部门。
- 角色：所有用户、任何未经认证的用户、任何已认证的用户、拥有该对象的用户、系统管理员。
- 系统根据不同角色分配相应的权限（静态和动态）。
- 系统管理员负责设置对用户和数据字典的管理。
- 任何已认证的用户可以查看所有已经创建的 OKR 列表和详情并可以发表评论。
- 拥有该对象的用户可以录入和管理本人的 OKR。
- OKR 基本信息：OKR 标题（一）、目标（多）、关键结果（多）。
- OKR 辅助信息：开始日期、结束日期、部门、类型、责任人、评论（多）。

#### 基础数据

系统管理员可以对基础数据进行增删改查。基础数据包括：

- 公司基本信息
- 部门基本信息
- OKR 类型

### 数据模型

根据需求需要建立 Okr、Object（目标）、KeyResult(关键结果)、Comment（评论）、Company（公司）、Department（部门）、（okrUser）用户等 7 个数据模型。其中 okrUser 继承自 lb3 默认的 user 模型。数据模型需要我们通过 lb 生成器来创建。

### 模型关系

- 一个公司有多个部门，一个部门属于一个公司。
- 一个部门有多个用户，一个 OKR 用户属于一个部门。
- 一个用户有多个 OKR，一个 OKR 属于一个 OKR 用户。
- 一个 OKR 有多个目标。
- 一个目标有多个关键结果。
- 一个 OKR 有多个评论。
- 一个评论属于一个 OKR 用户。

## 实现步骤

### 创建应用

#### 应用的相关信息

- 项目名称: `lb-okr-api`

```
$ lb
? 您的应用程序的名称是什么？ lb-okr-api
? 输入目录名称以包含项目： lb-okr-api
   create lb-okr-api/
     info 将工作目录更改为 lb-okr-api

? 您想要使用哪个版本的 LoopBack？ 3.x (Active Long Term Support)
? 您想要什么种类的应用程序？ api-server (使用本地用户认证的 LoopBack API 服务器)
正在生成 .yo-rc.json

I'm all done. Running npm install for you to install the required dependencies. If this fails, try running the command yourself.

后续步骤：

  将目录更改为您的应用程序
    $ cd lb-okr-api

  在应用程序中创建模型
    $ lb model

  运行应用程序
    $ node .
```

### 添加数据库连接

lb 可以连接多种常用的数据库，包括关系型数据库 MySQL 和非关系型数据库 MongoDB 等。

```
$ lb datasource
? 输入数据源名称： okrdb
? 为 okrdb 选择连接器： MongoDB （StrongLoop 支持）
? Connection String url to override other settings (eg: mongodb://username:password@hostname:por
t/database):
? host: localhost
? port: 27017
? user:
? password: [hidden]
? database: lb_okr
? 安装 loopback-connector-mongodb@^4.0.0 Yes
```

在`lb-okr-api/server/datasources.json`文件中自动添加了如下代码：

```
{
  "db": {
    "name": "db",
    "connector": "memory"
  },
  "okrdb": {
    "host": "localhost",
    "port": 27017,
    "url": "",
    "database": "lb_okr",
    "password": "",
    "name": "okrdb",
    "user": "",
    "connector": "mongodb"
  }
}
```

删除该文件中`db`键值对，再将`okrdb`改成`db`。完成后是下面的样子：

```
{
  "db": {
    "host": "localhost",
    "port": 27017,
    "url": "",
    "database": "lb_okr",
    "password": "",
    "name": "okrdb",
    "user": "",
    "connector": "mongodb"
  }
}
```

### 添加模型（models）

```
$ lb model
? 请输入模型名称： okr
? 选择要向其附加 okr 的数据源： db (mongodb)
? 选择模型的基类 PersistedModel
? 通过 REST API 公开 okr？ Yes
? 定制复数形式（用于构建 REST URL）：
? 公共模型或仅服务器？ 公共
现在添加一些 okr 属性。

在完成时输入空的属性名称。
? 属性名称： name
? 属性类型： string
? 是否为必需？ Yes
? 缺省值[对于无，保留为空白]：

下面添加另一个 okr 属性。
在完成时输入空的属性名称。
? 属性名称： startDate
? 属性类型： date
? 是否为必需？ Yes
? 缺省值[对于无，保留为空白]：

下面添加另一个 okr 属性。
在完成时输入空的属性名称。
? 属性名称： endDate
? 属性类型： date
? 是否为必需？ Yes
? 缺省值[对于无，保留为空白]：

下面添加另一个 okr 属性。
在完成时输入空的属性名称。
? 属性名称： department
? 属性类型： string
? 是否为必需？ Yes
? 缺省值[对于无，保留为空白]：

下面添加另一个 okr 属性。
在完成时输入空的属性名称。
? 属性名称： category
? 属性类型： string
? 是否为必需？ Yes
? 缺省值[对于无，保留为空白]：

下面添加另一个 okr 属性。
在完成时输入空的属性名称。
? 属性名称： people
? 属性类型： string
? 是否为必需？ Yes
? 缺省值[对于无，保留为空白]：

下面添加另一个 okr 属性。
在完成时输入空的属性名称。
? 属性名称：

重复此操作，创建所有的数据模型。
```

#### 模型信息

- Name: `okr`

  - Datasource: `db`
  - Base class: `PersistedModel`
  - Expose via REST: `Yes`
  - Custom plural form: _Leave blank_
  - Properties
    - `name`
      - String
      - required
    - `startDate`
      - Date
      - required
    - `endDate`
      - Date
      - required
    - `department`
      - String
      - required
    - `category`
      - String
      - required
    - `people`
      - String
      - required

- Name: `okrObject`

  - Datasource: `db`
  - Base class: `PersistedModel`
  - Expose via REST: `Yes`
  - Custom plural form: _Leave blank_
  - Properties
    - `content`
      - String
      - required

- Name: `keyResult`

  - Datasource: `db`
  - Base class: `PersistedModel`
  - Expose via REST: `Yes`
  - Custom plural form: _Leave blank_
  - Properties
    - `content`
      - String
      - required

- Name: `comment`

  - Datasource: `db`
  - Base class: `PersistedModel`
  - Expose via REST: `Yes`
  - Custom plural form: _Leave blank_
  - Properties

    - `content`
      - String
      - required
    - `people`
      - String
      - required

- Name: `company`

  - Datasource: `db`
  - Base class: `PersistedModel`
  - Expose via REST: `Yes`
  - Custom plural form: _Leave blank_
  - Properties
    - `name`
      - String
      - required

- Name: `department`

  - Datasource: `db`
  - Base class: `PersistedModel`
  - Expose via REST: `Yes`
  - Custom plural form: _Leave blank_
  - Properties
    - `name`
      - String
      - required

- Name: `category`

  - Datasource: `db`
  - Base class: `PersistedModel`
  - Expose via REST: `Yes`
  - Custom plural form: _Leave blank_
  - Properties
    - `name`
      - String
      - required

- Name: `okrUser`
  - Datasource: `db`
  - Base class: `User`
  - Expose via REST: `Yes`
  - Custom plural form: _Leave blank_

### 创建模型关系

```
$ lb relation
? 选择从中创建关系的模型： okr
? 关系类型： has many
? 选择与之创建关系的模型： okrObject
? 输入关系的属性名称： okrObjects
? （可选）输入定制外键：
? 需要直通模型？ No
? 允许在 REST API 中嵌套关系： No
? 禁止包含关系： No
```

```
$ lb relation
? 选择从中创建关系的模型： okrObject
? 关系类型： belongs to
? 选择与之创建关系的模型： okr
? 输入关系的属性名称： okr
? （可选）输入定制外键：
? 允许在 REST API 中嵌套关系： No
? 禁止包含关系： No
```

#### 定义模型关系

- okr

```
"relations": {
    "okrObjects": {
      "type": "hasMany",
      "model": "okrObject",
      "foreignKey": ""
    },
    "keyResults": {
      "type": "hasMany",
      "model": "keyResult",
      "foreignKey": ""
    },
    "comments": {
      "type": "hasMany",
      "model": "comment",
      "foreignKey": ""
    },
    "okr-category": {
      "type": "belongsTo",
      "model": "category",
      "foreignKey": ""
    }
  },
```

- okrObject

```
"relations": {
    "okr": {
      "type": "belongsTo",
      "model": "okr",
      "foreignKey": ""
    },
    "keyResults": {
      "type": "hasMany",
      "model": "keyResult",
      "foreignKey": ""
    }
  },
```

- keyResult

```
"relations": {
    "okrObject": {
      "type": "belongsTo",
      "model": "okrObject",
      "foreignKey": ""
    },
    "okr": {
      "type": "belongsTo",
      "model": "okr",
      "foreignKey": ""
    }
  },
```

其他模型关系定义参见源代码。

### 权限管理

#### 用户角色

OKR-API 包含以下用户角色:

- `所有用户`
- `任何未经认证的用户`
- `任何已认证的用户`
- `拥有该对象的用户`
- `系统管理员`

每个用户的权限取决于他的角色。登录用户需要进行 ACL（访问控制列表）认证。认证成功后，接口会返回一个 token 作为凭据。

在用户角色中，`系统管理员`属于`其他`类型的角色，lb 不会自动提供，需要手工创建。创建自定义角色的方法如下：

在`/server/boot`启动文件夹中，添加一个 js 文件。在 js 文件中添加如下代码。该代码执行一次后会添加首个 OKR 用户和自定义角色 admin，并把这个角色分配给首个 OKR 用户。

在 okrdb 数据库中，可以看到有三个表（collections），分别对应 okr 用户、角色、和角色分配模型。我们可以看到模型数据被持久化到了数据库中。所以，执行一次该代码后，就把这个 js 文件删除。

```
module.exports = function(app) {
  var User = app.models.Okruser;
  var Role = app.models.Role;
  var RoleMapping = app.models.RoleMapping;

  User.create([
    {username: 'admin', email: 'aa@bb.cc', password: 'admin'}
  ], function(err, users) {
    if (err) throw err;

    console.log('Created users:', users);

    //create the admin role
    Role.create({
      name: 'admin'
    }, function(err, role) {
      if (err) throw err;

      console.log('Created role:', role);

      //make admin
      role.principals.create({
        principalType: RoleMapping.USER,
        principalId: users[0].id
      }, function(err, principal) {
        if (err) throw err;

        console.log('Created principal:', principal);
      });
    });
  });
};
```

#### 对系统管理员进行授权

我们在上述代码中创建了一个系统管理员的用户。可以用这个邮箱和密码进行登录，登录验证通过后可以得到一个 token 的返回。

自定义了角色后，需要给角色授权。假设授予系统管理员对所有模型的所有接口进行操作，可以这样做：

```
$ lb acl
? 选择要应用 ACL 条目的模型： （所有现有模型）
? 选择 ACL 作用域： 所有方法和属性
? 选择访问类型： 全部（匹配所有类型）
? 选择角色 其他
? 请输入角色名称： admin
? 选择要应用的许可权 明确授权访问
```

### 创建 ACL （访问控制列表）

通过 lb 生成器，首先对所有模型、所有操作、所有用户设置拒绝访问权限。

```
$ lb acl
? 选择要应用 ACL 条目的模型： （所有现有模型）
? 选择 ACL 作用域： 所有方法和属性
? 选择访问类型： 全部（匹配所有类型）
? 选择角色 所有用户
? 选择要应用的许可权 明确拒绝访问
```

接下来根据角色开放应有的模型和操作访问限制。

#### ACL 策略

- 授权 `系统管理员` 对所有模型进行全权操作

  - 选择要应用 ACL 条目的模型： （所有现有模型）
  - 选择 ACL 作用域： 所有方法和属性
  - 选择访问类型： 全部（匹配所有类型）
  - 选择角色 其他
  - 请输入角色名称： admin
  - 选择要应用的许可权 明确授权访问

- 授权 `所有用户` 对 `okrUser` 模型进行注册操作。
- 授权 `任何已认证的用户` 对 `okrUser` 模型进行登录、注销操作。
- 授权 `拥有该对象的用户` 对 `okrUser` 模型进行修改密码、查询自身用户信息操作。
- 授权 `任何已认证的用户` 对 `okr、okrObject、keyResult、comment` 模型进行读操作。
- 授权 `拥有该对象的用户` 对 `okr、okrObject、keyResult、comment` 模型进行写操作。

### 测试 API 接口

启动服务 (`node .`) 并在浏览器打开网址 [`localhost:3000`](http://localhost:3000) 可以测试已经创建的 OKR_API。
