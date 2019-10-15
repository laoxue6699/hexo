---
title: LoopBack3.0自定义角色与授权
date: 2019-09-22 17:37:07
category:
  - 原创
tags:
  - loopBack
  - nodejs
  - express
  - RESTful
---

在我的文章[《LoopBack3.0 创建 API 接口实战》](https://juejin.im/post/5d856d575188257e5c111867)里，对官方示例`咖啡馆点评项目`中的权限管理进行了配置，按照项目需求实现了权限控制。但是，我们在配置权限管理策略时，定义所有的评论者都不能对`CoffeeShop`模型进行增删改查的操作。

<iframe src="//player.bilibili.com/player.html?aid=68852902&cid=119328164&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>

[到 bilibili.com 网站观看全部视频](https://www.bilibili.com/video/av68852902/)

如何对 coffeeShops 的数据进行管理呢？这是我们常说的后台管理，需要一个管理员`admin`的角色来承担。本文继续就这个项目来介绍如何自定义一个角色`admin`,将该角色与用户关联，并赋予该角色超级管理员的权限，可以对包括`CoffeeShop`在内的所有模型进行增删改查的操作。

<!-- more -->

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

## 二、创建角色

LoopBack 允许我们定义静态和动态角色.静态角色存储在数据源中,并映射到用户.相反,动态角色不分配给用户,而是在访问期间确定。

LoopBack 提供以下内置动态角色：

<table>
  <tbody>
    <tr>
      <th>Role object property</th>
      <th>String value</th>
      <th>Description</th>
    </tr>
    <tr>
      <td>Role.OWNER</td>
      <td>$owner</td>
      <td>Owner of the object</td>
    </tr>
    <tr>
      <td>Role.AUTHENTICATED</td>
      <td>$authenticated</td>
      <td>authenticated user</td>
    </tr>
    <tr>
      <td>Role.UNAUTHENTICATED</td>
      <td>$unauthenticated</td>
      <td>Unauthenticated user</td>
    </tr>
    <tr>
      <td>Role.EVERYONE</td>
      <td>$everyone</td>
      <td>Everyone</td>
    </tr>
  </tbody>
</table>

静态角色可以通过编码来创建。LoopBack 允许我们自己编写 js 脚本文件，脚本文件可以任意取名，放在`/server/boot/`目录下。当执行`node .`命令时， 会自动执行脚本文件中的代码。我们可以这样来编写`/server/boot/createAdmin.js`代码：

```
module.exports = function(app) {
  var User = app.models.Reviewer;
  var Role = app.models.Role;
  var RoleMapping = app.models.RoleMapping;

  User.create(
    [{ username: "admin", email: "aa@bb.cc", password: "admin" }],
    function(err, users) {
      if (err) throw err;

      console.log("Created users:", users);

      //create the admin role
      Role.create(
        {
          name: "admin"
        },
        function(err, role) {
          if (err) throw err;

          console.log("Created role:", role);

          //make RoleMapping
          role.principals.create(
            {
              principalType: RoleMapping.USER,
              principalId: users[0].id
            },
            function(err, principal) {
              if (err) throw err;

              console.log("Created principal:", principal);
            }
          );
        }
      );
    }
  );
};


```

### 代码解析

- 第 1 行是固定写法。定义一个函数并暴露出去，并传入`app`参数，用来调用 app 的模型。
- 第 2-4 行分别定义用户、角色和映射三个变量并赋值。
- 第 6 行到 11 行创建用户。如果创建用户成功，继续创建角色。
- 第 14 行到 21 行创建角色。如果创建用户成功，继续创建映射。
- 第 24 行到 32 行创建用户与角色之间的映射关系。

角色创建成功后，我们在 MongoDB 数据库中，可以看到刚刚创建的 3 个文档：

```
> use cshop
switched to db cshop
> show collections
Reviewer
Role
RoleMapping
> db.Role.find()
{ "_id" : ObjectId("5d875ecf98b8840aad726654"), "name" : "admin", "created" : ISODate("2019-09-22T11:45:19.325Z"), "modified" : ISODate("2019-09-22T11:45:19.325Z") }
```

## 三、角色授权

角色授权还是用`lb acl`命令。这次我们给`admin`角色赋予应用内的最大权限。

```
lb acl
? 选择要应用 ACL 条目的模型： （所有现有模型）
? 选择 ACL 作用域： 所有方法和属性
? 选择访问类型： 全部（匹配所有类型）
? 选择角色 其他
? 请输入角色名称： admin
? 选择要应用的许可权 明确授权访问
```

授权之后，我们打开`/common/models/coffee-shop.json`文件，可以看到对`admin`角色的授权配置代码：

```
{
      "accessType": "*",
      "principalType": "ROLE",
      "principalId": "admin",
      "permission": "ALLOW"
    }
```

在`/common/models/review.json`和`/common/models/review.json`文件中同样可以看到对`admin`角色的授权配置代码。

## 四、接口调试

在项目根目录输入命令`node .`启动应用。根据提示在浏览器地址栏输入`http://localhost:3000/explorer/`，来到接口调试页面。

### 未登录用户操作

![](http://huangxiaoman.cn/%E6%88%AA%E5%B1%8F2019-09-22%E4%B8%8B%E5%8D%887.58.43.png)

- 点击 Reviewer，展开 Reviewer 相关的 API 接口。

![](http://huangxiaoman.cn/%E6%88%AA%E5%B1%8F2019-09-22%E4%B8%8B%E5%8D%888.01.39.png)

- 点击`get/Reviewers`，出现操作提示界面。

![](http://huangxiaoman.cn/%E6%88%AA%E5%B1%8F2019-09-22%E4%B8%8B%E5%8D%888.04.08.png)

- 点击`Try it out!`按钮，显示 401 错误，提示需要授权。

![](http://huangxiaoman.cn/%E6%88%AA%E5%B1%8F2019-09-22%E4%B8%8B%E5%8D%888.04.46.png)

### 用 admin 用户登录

- 点击`Post/Reviewers/login`,出现登录操作提示界面。

![](http://huangxiaoman.cn/%E6%88%AA%E5%B1%8F2019-09-22%E4%B8%8B%E5%8D%888.11.23.png)

- 输入登录参数：

![](http://huangxiaoman.cn/%E6%88%AA%E5%B1%8F2019-09-22%E4%B8%8B%E5%8D%888.14.37.png)

- 登录成功后返回 admin 的相关信息。

![](http://huangxiaoman.cn/%E6%88%AA%E5%B1%8F2019-09-22%E4%B8%8B%E5%8D%888.11.52.png)

- 复制 admin 返回信息中的`id`字符串值，本例中`2F410ZR1bEMJFrTY4LtVPRb6TPzsCPsXOEXpO1u9weD561dx1hkim87AY1fQGlt7`粘贴到窗口页面第一行的`accessToken`中，然后点击`Set Access Token`按钮。

![](http://huangxiaoman.cn/%E6%88%AA%E5%B1%8F2019-09-22%E4%B8%8B%E5%8D%888.20.36.png)

- 再次点击`get/Reviewers`，出现操作提示界面后，点击`Try it out!`按钮，不再提示需要授权的信息，成功返回应用中已有的两个用户的信息。

![](http://huangxiaoman.cn/%E6%88%AA%E5%B1%8F2019-09-22%E4%B8%8B%E5%8D%888.23.30.png)

### 注销登录

- 点击`Post/Reviewers/logout`,注销登录。

![](http://huangxiaoman.cn/%E6%88%AA%E5%B1%8F2019-09-22%E4%B8%8B%E5%8D%888.28.52.png)

- 再次点击`get/Reviewers`，出现操作提示界面后，点击`Try it out!`按钮，再次提示需要授权的信息。

### 权限验证

我们对 admin 这个超级管理员角色的权限进行了初步验证。我们还可以参照这些方法对应用中其他的权限控制进行验证。

在我的文章[《LoopBack3.0 创建 API 接口实战》](https://juejin.im/post/5d856d575188257e5c111867)里，对官方示例`咖啡馆点评项目`中的权限管理进行了配置，按照项目需求实现了权限控制。

- 任何人都可以阅读评论，但必须先登录才能创建，编辑或删除它们。
- 任何人都可以注册为用户，然后能够登录或者注销。
- 登录用户可以创建新的评论，编辑或删除自己的评论，但是他们不能修改咖啡店相关内容。

我们都可以在接口调试页面进行验证。
