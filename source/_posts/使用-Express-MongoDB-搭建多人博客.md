---
title: 使用 Express + MongoDB 搭建多人博客
date: 2019-08-17 18:49:40
tags:
category: 转载
---

[转自 https://github.com/nswbmw](https://github.com/nswbmw)

# Node.js
## 安装 Node.js

有三种方式安装 Node.js：一是通过安装包安装，二是通过源码编译安装，三是在 Linux 下可以通过 yum|apt-get 安装，在 Mac 下可以通过 [Homebrew](http://brew.sh/) 安装。对于 Windows 和 Mac 用户，推荐使用安装包安装，Linux 用户推荐使用源码编译安装。

<!-- more -->

### Windows 和 Mac 安装：

#### 第一步：

打开 [Node.js 官网](https://nodejs.org/en/)，可以看到以下两个下载选项：

![](/images/1.1.1.png)

左边的是 LTS 版，用过 ubuntu 的同学可能比较熟悉，即长期支持版本，大多数人用这个就可以了。右边是最新版，支持最新的语言特性（比如对 ES6 的支持更全面），想尝试新特性的开发者可以安装这个版本。我们选择左边的 v6.9.1 LTS 点击下载。

> 小提示：从 [http://node.green](http://node.green) 上可以看到 Node.js 各个版本对 ES6 的支持情况。

#### 第二步：

安装 Node.js，这个没什么好说的，一直点击 `继续` 即可。

![](/images/1.1.2.png)

#### 第三步：####

提示安装成功后，打开终端输入以下命令，可以看到 node 和 npm 都已经安装好了：

![](/images/1.1.3.png)

### Linux 安装：

Linux 用户可通过源码编译安装：

```sh
curl -O https://nodejs.org/dist/v6.9.1/node-v6.9.1.tar.gz
tar -xzvf node-v6.9.1.tar.gz
cd node-v6.9.1
./configure
make
make install
```

> 注意: 如果编译过程报错，可能是缺少某些依赖包。因为报错内容不尽相同，请读者自行求助搜索引擎或 [stackoverflow](http://stackoverflow.com/)。

## n 和 nvm

通常我们使用稳定的 LTS 版本的 Node.js 即可，但有的情况下我们又想尝试一下新的特性，我们总不能来回安装不同版本的 Node.js 吧，这个时候我们就需要 [n](https://github.com/tj/n) 或者 [nvm](https://github.com/creationix/nvm) 了。n 和 nvm 是两个常用的 Node.js 版本管理工具，关于 n 和 nvm 的使用以及区别，[这篇文章](http://taobaofed.org/blog/2015/11/17/nvm-or-n/) 讲得特别详细，这里不再赘述。

## nrm

[nrm](https://github.com/Pana/nrm) 是一个管理 npm 源的工具。用过 ruby 和 gem 的同学会比较熟悉，通常我们会把 gem 源切到国内的淘宝镜像，这样在安装和更新一些包的时候比较快。nrm 同理，用来切换官方 npm 源和国内的 npm 源（如: [cnpm](http://cnpmjs.org/)），当然也可以用来切换官方 npm 源和公司私有 npm 源。

全局安装 nrm:

```sh
npm i nrm -g
```

查看当前 nrm 内置的几个 npm 源的地址：

![](/images/1.1.4.png)

切换到 cnpm：

![](/images/1.1.5.png)

## 安装与启动 MongoDB

- Windows 用户向导：https://docs.mongodb.com/manual/tutorial/install-mongodb-on-windows/
- Linux 用户向导：https://docs.mongodb.com/manual/administration/install-on-linux/
- Mac 用户向导：https://docs.mongodb.com/manual/tutorial/install-mongodb-on-os-x/

### Robomongo 和 Mongochef

#### Robomongo

[Robomongo](https://robomongo.org/) 是一个基于 Shell 的跨平台开源 MongoDB 可视化管理工具，支持 Windows、Linux 和 Mac，嵌入了 JavaScript 引擎和 MongoDB mongo，只要你会使用 mongo shell，你就会使用 Robomongo，它还提供了语法高亮、自动补全、差别视图等。

[Robomongo 下载地址](https://robomongo.org/download)

下载并安装成功后点击左上角的 `Create` 创建一个连接，给该连接起个名字如: `localhost`，使用默认地址（localhost）和端口（27017）即可，点击 `Save` 保存。

![](/images/1.2.1.png)

双击 `localhost` 连接到 MongoDB 并进入交互界面，尝试插入一条数据并查询出来，如下所示:

![](/images/1.2.2.png)

#### MongoChef

[MongoChef](http://3t.io/mongochef/) 是另一款强大的 MongoDB 可视化管理工具，支持 Windows、Linux 和 Mac。

[MongoChef 下载地址](http://3t.io/mongochef/#mongochef-download-compare)，我们选择左侧的非商业用途的免费版下载。

![](/images/1.2.3.png)

安装成功后跟 Robomongo 一样，也需要创建一个新的连接的配置，成功后双击进入到 MongoChef 主页面，如下所示:

![](/images/1.2.4.png)

还可以使用 shell 模式:

![](/images/1.2.5.png)

> 小提示: MongoChef 相较于 Robomongo 更强大一些，但 Robomongo 比较轻量也能满足大部分的常规需求，所以哪一个适合自己还需读者自行尝试。

## require

require 用来加载一个文件的代码，关于 require 的机制这里不展开讲解，请仔细阅读 [官方文档](https://nodejs.org/api/modules.html)。

简单概括以下几点:

- require 可加载 .js、.json 和 .node 后缀的文件
- require 的过程是同步的，所以这样是错误的:

```sh
setTimeout(() => {
  module.exports = { a: 'hello' }
}, 0)
```

require 这个文件得到的是空对象 `{}`

- require 目录的机制是:
  - 如果目录下有 package.json 并指定了 main 字段，则用之
  - 如果不存在 package.json，则依次尝试加载目录下的 index.js 和 index.node
- require 过的文件会加载到缓存，所以多次 require 同一个文件（模块）不会重复加载
- 判断是否是程序的入口文件有两种方式:
  - require.main === module（推荐）
  - module.parent === null

## 循环引用

循环引用（或循环依赖）简单点来说就是 a 文件 require 了 b 文件，然后 b 文件又反过来 require 了 a 文件。我们用 a->b 代表 b require 了 a。

简单的情况:

```
a->b
b->a
```

复杂点的情况:

```
a->b
b->c
c->a
```

循环引用并不会报错，导致的结果是 require 的结果是空对象 `{}`，原因是 b require 了 a，a 又去 require 了 b，此时 b 还没初始化好，所以只能拿到初始值 `{}`。当产生循环引用时一般有两种方法解决：

1. 通过分离共用的代码到另一个文件解决，如上面简单的情况，可拆出共用的代码到 c 中，如下:

```
c->a
c->b
```

2. 不在最外层 require，在用到的地方 require，通常在函数的内部

总的来说，循环依赖的陷阱并不大容易出现，但一旦出现了，对于新手来说还真不好定位。它的存在给我们提了个醒，要时刻注意你项目的依赖关系不要过于复杂，哪天你发现一个你明明已经 exports 了的方法报 `undefined is not a function`，我们就该提醒一下自己：哦，也许是它来了。

官方示例: [https://nodejs.org/api/modules.html#modules_cycles](https://nodejs.org/api/modules.html#modules_cycles)

require 用来加载代码，而 exports 和 module.exports 则用来导出代码。

很多新手可能会迷惑于 exports 和 module.exports 的区别，为了更好的理解 exports 和 module.exports 的关系，我们先来巩固下 js 的基础。示例：

**test.js**

```js
var a = { name: 1 };
var b = a;

console.log(a);
console.log(b);

b.name = 2;
console.log(a);
console.log(b);

var b = { name: 3 };
console.log(a);
console.log(b);
```

运行 test.js 结果为：

```
{ name: 1 }
{ name: 1 }
{ name: 2 }
{ name: 2 }
{ name: 2 }
{ name: 3 }
```

**解释**：a 是一个对象，b 是对 a 的引用，即 a 和 b 指向同一块内存，所以前两个输出一样。当对 b 作修改时，即 a 和 b 指向同一块内存地址的内容发生了改变，所以 a 也会体现出来，所以第三四个输出一样。当 b 被覆盖时，b 指向了一块新的内存，a 还是指向原来的内存，所以最后两个输出不一样。

明白了上述例子后，我们只需知道三点就知道 exports 和 module.exports 的区别了：

1. module.exports 初始值为一个空对象 {}
2. exports 是指向的 module.exports 的引用
3. require() 返回的是 module.exports 而不是 exports

Node.js 官方文档的截图证实了我们的观点:

![](/images/2.2.1.png)

## 导出
exports = module.exports = {...}

我们经常看到这样的写法：

```js
exports = module.exports = {...}
```

上面的代码等价于:

```js
module.exports = {...}
exports = module.exports
```

原理很简单：module.exports 指向新的对象时，exports 断开了与 module.exports 的引用，那么通过 exports = module.exports 让 exports 重新指向 module.exports。

> 小提示：ES6 的 import 和 export 不在本文的讲解范围，有兴趣的读者可以去学习阮一峰老师的[《ECMAScript6 入门》](http://es6.ruanyifeng.com/)。

# Promise

网上已经有许多关于 Promise 的资料了，这里不在赘述。以下 4 个链接供读者学习：

1. https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise （基础）
2. http://liubin.org/promises-book/ （开源 Promise 迷你书）
3. http://fex.baidu.com/blog/2015/07/we-have-a-problem-with-promises/ （进阶）
4. https://promisesaplus.com/ （官方定义规范）

Promise 用于异步流程控制，生成器与 yield 也能实现流程控制（基于 co），但不在本教程讲解范围内，读者可参考我的另一部教程 [N-club](https://github.com/nswbmw/N-club)。async/await 结合 Promise 也可以实现流程控制，有兴趣请查阅 [《ECMAScript6 入门》](http://es6.ruanyifeng.com/#docs/async#async函数)。

## 深入 Promise

- [Promise 必知必会（十道题）](https://zhuanlan.zhihu.com/p/30797777)
- [深入 Promise(一)——Promise 实现详解](https://zhuanlan.zhihu.com/p/25178630)
- [深入 Promise(二)——进击的 Promise](https://zhuanlan.zhihu.com/p/25198178)
- [深入 Promise(三)——命名 Promise](https://zhuanlan.zhihu.com/p/25199781)

环境变量不属于 Node.js 的知识范畴，只不过我们在开发 Node.js 应用时经常与环境变量打交道，所以这里简单介绍下。

环境变量（environment variables）一般是指在操作系统中用来指定操作系统运行环境的一些参数。在 Mac 和 Linux 的终端直接输入 env，会列出当前的环境变量，如：USER=xxx。简单来讲，环境变量就是传递参数给运行程序的。

在 Node.js 中，我们经常这么用:

```sh
NODE_ENV=test node app
```

通过以上命令启动程序，指定当前环境变量 `NODE_ENV` 的值为 test，那么在 app.js 中可通过 `process.env` 来获取环境变量:

```
console.log(process.env.NODE_ENV) //test
```

另一个常见的例子是使用 [debug](https://www.npmjs.com/package/debug) 模块时:

```sh
DEBUG=* node app
```

Windows 用户需要首先设置环境变量，然后再执行程序：

```sh
set DEBUG=*
set NODE_ENV=test
node app
```

或者使用 [cross-env](https://www.npmjs.com/package/cross-env)：

```sh
npm i cross-env -g
```

使用方式：

```sh
cross-env NODE_ENV=test node app
```

package.json 对于 Node.js 应用来说是一个不可或缺的文件，它存储了该 Node.js 应用的名字、版本、描述、作者、入口文件、脚本、版权等等信息。npm 官网有 package.json 每个字段的详细介绍：[https://docs.npmjs.com/files/package.json](https://docs.npmjs.com/files/package.json)。

## semver

语义化版本（semver）即 dependencies、devDependencies 和 peerDependencies 里的如：`"co": "^4.6.0"`。

semver 格式：`主版本号.次版本号.修订号`。版本号递增规则如下：

- `主版本号`：做了不兼容的 API 修改
- `次版本号`：做了向下兼容的功能性新增
- `修订号`：做了向下兼容的 bug 修正

更多阅读：

1. http://semver.org/lang/zh-CN/
2. http://taobaofed.org/blog/2016/08/04/instructions-of-semver/

# npm

作为 Node.js 的开发者，我们在发布 npm 模块的时候一定要遵守语义化版本的命名规则，即：有 breaking change 发大版本，有新增的功能发小版本，有小的 bug 修复或优化则发修订版本。

## npm init

使用 `npm init` 初始化一个空项目是一个好的习惯，即使你对 package.json 及其他属性非常熟悉，`npm init` 也是你开始写新的 Node.js 应用或模块的一个快捷的办法。`npm init` 有智能的默认选项，比如从根目录名称推断模块名称，通过 `~/.npmrc` 读取你的信息，用你的 Git 设置来确定 repository 等等。

## npm install

`npm install` 是我们最常用的 npm 命令之一，因此我们需要好好了解下这个命令。终端输入 `npm install -h` 查看使用方式:

![](/images/2.6.1.png)

可以看出：我们通过 `npm install` 可以安装 npm 上发布的某个版本、某个 tag、某个版本区间的模块，甚至可以安装本地目录、压缩包和 git/github 的库作为依赖。

> 小提示: `npm i` 是 `npm install` 的简写，建议使用 `npm i`。

直接使用 `npm i` 安装的模块是不会写入 package.json 的 dependencies (或 devDependencies)，需要额外加个参数:

1. `npm i express --save`/`npm i express -S` (安装 express，同时将 `"express": "^4.14.0"` 写入 dependencies )
2. `npm i express --save-dev`/`npm i express -D` (安装 express，同时将 `"express": "^4.14.0"` 写入 devDependencies )
3. `npm i express --save --save-exact` (安装 express，同时将 `"express": "4.14.0"` 写入 dependencies )

第三种方式将固定版本号写入 dependencies，建议线上的 Node.js 应用都采取这种锁定版本号的方式，因为你不可能保证第三方模块下个小版本是没有验证 bug 的，即使是很流行的模块。拿 Mongoose 来说，Mongoose 4.1.4 引入了一个 bug 导致调用一个文档 entry 的 remove 会删除整个集合的文档，见：[https://github.com/Automattic/mongoose/blob/master/History.md#415--2015-09-01](https://github.com/Automattic/mongoose/blob/master/History.md#415--2015-09-01)。

> 后面会介绍更安全的 `npm shrinkwrap` 的用法。

运行以下命令：

```sh
npm config set save-exact true
```

这样每次 `npm i xxx --save` 的时候会锁定依赖的版本号，相当于加了 `--save-exact` 参数。

> 小提示：`npm config set` 命令将配置写到了 ~/.npmrc 文件，运行 `npm config list` 查看。

## npm scripts

npm 提供了灵活而强大的 scripts 功能，见 [官方文档](https://docs.npmjs.com/misc/scripts)。

npm 的 scripts 有一些内置的缩写命令，如常用的：

- `npm start` 等价于 `npm run start`
- `npm test` 等价于 `npm run test`

## npm shrinkwrap

前面说过要锁定依赖的版本，但这并不能完全防止意外情况的发生，因为锁定的只是最外一层的依赖，而里层依赖的模块的 package.json 有可能写的是 `"mongoose": "*"`。为了彻底锁定依赖的版本，让你的应用在任何机器上安装的都是同样版本的模块（不管嵌套多少层），通过运行 `npm shrinkwrap`，会在当前目录下产生一个 `npm-shrinkwrap.json`，里面包含了通过 node_modules 计算出的模块的依赖树及版本。上面的截图也显示：只要目录下有 npm-shrinkwrap.json 则运行 `npm install` 的时候会优先使用 npm-shrinkwrap.json 进行安装，没有则使用 package.json 进行安装。

更多阅读：

1. https://docs.npmjs.com/cli/shrinkwrap
2. http://tech.meituan.com/npm-shrinkwrap.html

> 注意: 如果 node_modules 下存在某个模块（如直接通过 `npm install xxx` 安装的）而 package.json 中没有，运行 `npm shrinkwrap` 则会报错。另外，`npm shrinkwrap` 只会生成 dependencies 的依赖，不会生成 devDependencies 的。

# express

首先，我们新建一个目录 myblog，在该目录下运行 `npm init` 生成一个 package.json，如下所示：

![](/images/3.1.1.png)

> 注意：括号里的是默认值，如果使用默认值则直接回车即可，否则输入自定义内容后回车。

然后安装 express 并写入 package.json：

```sh
npm i express@4.14.0 --save
```

新建 index.js，添加如下代码：

```js
const express = require("express");
const app = express();

app.get("/", function(req, res) {
  res.send("hello, express");
});

app.listen(3000);
```

以上代码的意思是：生成一个 express 实例 app，挂载了一个根路由控制器，然后监听 3000 端口并启动程序。运行 `node index`，打开浏览器访问 `localhost:3000` 时，页面应显示 hello, express。

这是最简单的一个使用 express 的例子，后面会介绍路由及模板的使用。

## supervisor

在开发过程中，每次修改代码保存后，我们都需要手动重启程序，才能查看改动的效果。使用 [supervisor](https://www.npmjs.com/package/supervisor) 可以解决这个繁琐的问题，全局安装 supervisor：

```sh
npm i -g supervisor
```

运行 `supervisor index` 启动程序，如下所示：

![](/images/3.1.2.png)

supervisor 会监听当前目录下 node 和 js 后缀的文件，当这些文件发生改动时，supervisor 会自动重启程序。

前面我们只是挂载了根路径的路由控制器，现在修改 index.js 如下：

```js
const express = require("express");
const app = express();

app.get("/", function(req, res) {
  res.send("hello, express");
});

app.get("/users/:name", function(req, res) {
  res.send("hello, " + req.params.name);
});

app.listen(3000);
```

以上代码的意思是：当访问根路径时，依然返回 hello, express，当访问如 `localhost:3000/users/nswbmw` 路径时，返回 hello, nswbmw。路径中 `:name` 起了占位符的作用，这个占位符的名字是 name，可以通过 `req.params.name` 取到实际的值。

> 小提示：express 使用了 [path-to-regexp](https://www.npmjs.com/package/path-to-regexp) 模块实现的路由匹配。

不难看出：req 包含了请求来的相关信息，res 则用来返回该请求的响应，更多请查阅 [express 官方文档](http://expressjs.com/en/4x/api.html)。下面介绍几个常用的 req 的属性：

- `req.query`: 解析后的 url 中的 querystring，如 `?name=haha`，req.query 的值为 `{name: 'haha'}`
- `req.params`: 解析 url 中的占位符，如 `/:name`，访问 /haha，req.params 的值为 `{name: 'haha'}`
- `req.body`: 解析后请求体，需使用相关的模块，如 [body-parser](https://www.npmjs.com/package/body-parser)，请求体为 `{"name": "haha"}`，则 req.body 为 `{name: 'haha'}`

## express.Router

上面只是很简单的路由使用的例子（将所有路由控制函数都放到了 index.js），但在实际开发中通常有几十甚至上百的路由，都写在 index.js 既臃肿又不好维护，这时可以使用 express.Router 实现更优雅的路由解决方案。在 myblog 目录下创建空文件夹 routes，在 routes 目录下创建 index.js 和 users.js。最后代码如下：

**index.js**

```js
const express = require("express");
const app = express();
const indexRouter = require("./routes/index");
const userRouter = require("./routes/users");

app.use("/", indexRouter);
app.use("/users", userRouter);

app.listen(3000);
```

**routes/index.js**

```js
const express = require("express");
const router = express.Router();

router.get("/", function(req, res) {
  res.send("hello, express");
});

module.exports = router;
```

**routes/users.js**

```js
const express = require("express");
const router = express.Router();

router.get("/:name", function(req, res) {
  res.send("hello, " + req.params.name);
});

module.exports = router;
```

以上代码的意思是：我们将 `/` 和 `/users/:name` 的路由分别放到了 routes/index.js 和 routes/users.js 中，每个路由文件通过生成一个 express.Router 实例 router 并导出，通过 `app.use` 挂载到不同的路径。这两种代码实现了相同的功能，但在实际开发中推荐使用 express.Router 将不同的路由分离到不同的路由文件中。

更多 express.Router 的用法见 [express 官方文档](http://expressjs.com/en/4x/api.html#router)。

模板引擎（Template Engine）是一个将页面模板和数据结合起来生成 html 的工具。上例中，我们只是返回纯文本给浏览器，现在我们修改代码返回一个 html 页面给浏览器。

## ejs

模板引擎有很多，[ejs](https://www.npmjs.com/package/ejs) 是其中一种，因为它使用起来十分简单，而且与 express 集成良好，所以我们使用 ejs。安装 ejs：

```sh
npm i ejs --save
```

修改 index.js 如下：

**index.js**

```js
const path = require("path");
const express = require("express");
const app = express();
const indexRouter = require("./routes/index");
const userRouter = require("./routes/users");

app.set("views", path.join(__dirname, "views")); // 设置存放模板文件的目录
app.set("view engine", "ejs"); // 设置模板引擎为 ejs

app.use("/", indexRouter);
app.use("/users", userRouter);

app.listen(3000);
```

通过 `app.set` 设置模板引擎为 ejs 和存放模板的目录。在 myblog 下新建 views 文件夹，在 views 下新建 users.ejs，添加如下代码：

**views/users.ejs**

```html
<!DOCTYPE html>
<html>
  <head>
    <style type="text/css">
      body {
        padding: 50px;
        font: 14px "Lucida Grande", Helvetica, Arial, sans-serif;
      }
    </style>
  </head>
  <body>
    <h1><%= name.toUpperCase() %></h1>
    <p>hello, <%= name %></p>
  </body>
</html>
```

修改 routes/users.js 如下：

**routes/users.js**

```js
const express = require("express");
const router = express.Router();

router.get("/:name", function(req, res) {
  res.render("users", {
    name: req.params.name
  });
});

module.exports = router;
```

通过调用 `res.render` 函数渲染 ejs 模板，res.render 第一个参数是模板的名字，这里是 users 则会匹配 views/users.ejs，第二个参数是传给模板的数据，这里传入 name，则在 ejs 模板中可使用 name。`res.render` 的作用就是将模板和数据结合生成 html，同时设置响应头中的 `Content-Type: text/html`，告诉浏览器我返回的是 html，不是纯文本，要按 html 展示。现在我们访问 `localhost:3000/users/haha`，如下图所示：

![](/images/3.3.1.png)

上面代码可以看到，我们在模板 `<%= name.toUpperCase() %>` 中使用了 JavaScript 的语法 `.toUpperCase()` 将名字转化为大写，那这个 `<%= xxx %>` 是什么东西呢？ejs 有 3 种常用标签：

1. `<% code %>`：运行 JavaScript 代码，不输出
2. `<%= code %>`：显示转义后的 HTML 内容
3. `<%- code %>`：显示原始 HTML 内容

> 注意：`<%= code %>` 和 `<%- code %>` 都可以是 JavaScript 表达式生成的字符串，当变量 code 为普通字符串时，两者没有区别。当 code 比如为 `<h1>hello</h1>` 这种字符串时，`<%= code %>` 会原样输出 `<h1>hello</h1>`，而 `<%- code %>` 则会显示 H1 大的 hello 字符串。

下面的例子解释了 `<% code %>` 的用法：

**Data**

```
supplies: ['mop', 'broom', 'duster']
```

**Template**

```ejs
<ul>
<% for(var i=0; i<supplies.length; i++) {%>
   <li><%= supplies[i] %></li>
<% } %>
</ul>
```

**Result**

```html
<ul>
  <li>mop</li>
  <li>broom</li>
  <li>duster</li>
</ul>
```

更多 ejs 的标签请看 [官方文档](https://www.npmjs.com/package/ejs#tags)。

## includes

我们使用模板引擎通常不是一个页面对应一个模板，这样就失去了模板的优势，而是把模板拆成可复用的模板片段组合使用，如在 views 下新建 header.ejs 和 footer.ejs，并修改 users.ejs：

**views/header.ejs**

```ejs
<!DOCTYPE html>
<html>
  <head>
    <style type="text/css">
      body {padding: 50px;font: 14px "Lucida Grande", Helvetica, Arial, sans-serif;}
    </style>
  </head>
  <body>
```

**views/footer.ejs**

```ejs
  </body>
</html>
```

**views/users.ejs**

```ejs
<%- include('header') %>
  <h1><%= name.toUpperCase() %></h1>
  <p>hello, <%= name %></p>
<%- include('footer') %>
```

我们将原来的 users.ejs 拆成出了 header.ejs 和 footer.ejs，并在 users.ejs 通过 ejs 内置的 include 方法引入，从而实现了跟以前一个模板文件相同的功能。

> 小提示：拆分模板组件通常有两个好处：
>
> 1. 模板可复用，减少重复代码
> 2. 主模板结构清晰

> 注意：要用 `<%- include('header') %>` 而不是 `<%= include('header') %>`
> 前面我们讲解了 express 中路由和模板引擎 ejs 的用法，但 express 的精髓并不在此，在于中间件的设计理念。

## 中间件与 next

express 中的中间件（middleware）就是用来处理请求的，当一个中间件处理完，可以通过调用 `next()` 传递给下一个中间件，如果没有调用 `next()`，则请求不会往下传递，如内置的 `res.render` 其实就是渲染完 html 直接返回给客户端，没有调用 `next()`，从而没有传递给下一个中间件。看个小例子，修改 index.js 如下：

**index.js**

```js
const express = require("express");
const app = express();

app.use(function(req, res, next) {
  console.log("1");
  next();
});

app.use(function(req, res, next) {
  console.log("2");
  res.status(200).end();
});

app.listen(3000);
```

此时访问 `localhost:3000`，终端会输出：

```
1
2
```

通过 `app.use` 加载中间件，在中间件中通过 next 将请求传递到下一个中间件，next 可接受一个参数接收错误信息，如果使用了 `next(error)`，则会返回错误而不会传递到下一个中间件，修改 index.js 如下：

**index.js**

```js
const express = require("express");
const app = express();

app.use(function(req, res, next) {
  console.log("1");
  next(new Error("haha"));
});

app.use(function(req, res, next) {
  console.log("2");
  res.status(200).end();
});

app.listen(3000);
```

此时访问 `localhost:3000`，终端会输出错误信息：

![](/images/3.4.1.png)

浏览器会显示：

![](/images/3.4.2.png)

> 小提示：`app.use` 有非常灵活的使用方式，详情见 [官方文档](http://expressjs.com/en/4x/api.html#app.use)。

express 有成百上千的第三方中间件，在开发过程中我们首先应该去 npm 上寻找是否有类似实现的中间件，尽量避免造轮子，节省开发时间。下面给出几个常用的搜索 npm 模块的网站：

1. [http://npmjs.com](http://npmjs.com)(npm 官网)
2. [http://node-modules.com](http://node-modules.com)
3. [https://npms.io](https://npms.io)
4. [https://nodejsmodules.org](https://nodejsmodules.org)

> 小提示：express@4 之前的版本基于 connect 这个模块实现的中间件的架构，express@4 及以上的版本则移除了对 connect 的依赖自己实现了，理论上基于 connect 的中间件（通常以 `connect-` 开头，如 `connect-mongo`）仍可结合 express 使用。

> 注意：中间件的加载顺序很重要！比如：通常把日志中间件放到比较靠前的位置，后面将会介绍的 `connect-flash` 中间件是基于 session 的，所以需要在 `express-session` 后加载。

## 错误处理

上面的例子中，应用程序为我们自动返回了错误栈信息（express 内置了一个默认的错误处理器），假如我们想手动控制返回的错误内容，则需要加载一个自定义错误处理的中间件，修改 index.js 如下：

**index.js**

```js
const express = require("express");
const app = express();

app.use(function(req, res, next) {
  console.log("1");
  next(new Error("haha"));
});

app.use(function(req, res, next) {
  console.log("2");
  res.status(200).end();
});

//错误处理
app.use(function(err, req, res, next) {
  console.error(err.stack);
  res.status(500).send("Something broke!");
});

app.listen(3000);
```

此时访问 `localhost:3000`，浏览器会显示 `Something broke!`。

> 小提示：关于 express 的错误处理，详情见 [官方文档](http://expressjs.com/en/guide/error-handling.html)。
> 从本章开始，正式学习如何使用 Express + MongoDB 搭建一个博客。


## 目录结构

我们停止 supervisor 并删除 myblog 目录从头来过。重新创建 myblog，运行 `npm init`，如下：

![](/images/4.2.1.png)

在 myblog 目录下创建以下目录及空文件（package.json 除外）：

![](/images/4.2.2.png)

对应文件及文件夹的用处：

1. `models`: 存放操作数据库的文件
2. `public`: 存放静态文件，如样式、图片等
3. `routes`: 存放路由文件
4. `views`: 存放模板文件
5. `index.js`: 程序主文件
6. `package.json`: 存储项目名、描述、作者、依赖等等信息

> 小提示：不知读者发现了没有，我们遵循了 MVC（模型(model)－视图(view)－控制器(controller/route)） 的开发模式。

## 安装依赖模块

运行以下命令安装所需模块：

```sh
npm i config-lite connect-flash connect-mongo ejs express express-session marked moment mongolass objectid-to-timestamp sha1 winston express-winston --save
npm i https://github.com:utatti/express-formidable.git --save # 从 GitHub 安装 express-formidable 最新版，v1.0.0 有 bug
```

对应模块的用处：

1. `express`: web 框架
2. `express-session`: session 中间件
3. `connect-mongo`: 将 session 存储于 mongodb，结合 express-session 使用
4. `connect-flash`: 页面通知的中间件，基于 session 实现
5. `ejs`: 模板
6. `express-formidable`: 接收表单及文件上传的中间件
7. `config-lite`: 读取配置文件
8. `marked`: markdown 解析
9. `moment`: 时间格式化
10. `mongolass`: mongodb 驱动
11. `objectid-to-timestamp`: 根据 ObjectId 生成时间戳
12. `sha1`: sha1 加密，用于密码加密
13. `winston`: 日志
14. `express-winston`: express 的 winston 日志中间件

后面会详细讲解这些模块的用法。

## ESLint

ESLint 是一个代码规范和语法错误检查工具。使用 ESLint 可以规范我们的代码书写，可以在编写代码期间就能发现一些低级错误。

ESLint 需要结合编辑器或 IDE 使用，如：

- Sublime Text 需要装两个插件：SublimeLinter + SublimeLinter-contrib-eslint
- VS Code 需要装一个插件：ESLint

> 小提示：Sublime Text 安装插件通过 ctrl+shift+p 调出 Package Control，输入 install 选择 Install Package 回车。输入对应插件名搜索，回车安装。
> 小提示：VS Code 安装插件需要点击左侧『扩展』页

全局安装 eslint：

```sh
npm i eslint -g
```

运行：

```sh
eslint --init
```

初始化 eslint 配置，依次选择：

-> Use a popular style guide  
-> Standard  
-> JSON

> 注意：如果 Windows 用户使用其他命令行工具无法上下切换选项，切换回 cmd。

eslint 会创建一个 .eslintrc.json 的配置文件，同时自动安装并添加相关的模块到 devDependencies。这里我们使用 Standard 规范，其主要特点是不加分号。

## EditorConfig

EditorConfig 是一个保持缩进风格的一致的工具，当多人共同开发一个项目的时候，往往会出现每个人用不同编辑器的情况，而且有的人用 tab 缩进，有的人用 2 个空格缩进，有的人用 4 个空格缩进，EditorConfig 就是为了解决这个问题而诞生。

EditorConfig 需要结合编辑器或 IDE 使用，如：

- Sublime Text 需要装一个插件：EditorConfig
- VS Code 需要装一个插件：EditorConfig for VS Code

在 myblog 目录下新建 .editorconfig 的文件，添加如下内容：

```
# editorconfig.org
root = true

[*]
indent_style = space
indent_size = 2
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true
tab_width = 2

[*.md]
trim_trailing_whitespace = false

[Makefile]
indent_style = tab
```

这里我们使用 2 个空格缩进，tab 长度也是 2 个空格。trim_trailing_whitespace 用来删除每一行最后多余的空格，insert_final_newline 用来在代码最后插入一个空的换行。
不管是小项目还是大项目，将配置与代码分离是一个非常好的做法。我们通常将配置写到一个配置文件里，如 config.js 或 config.json ，并放到项目的根目录下。但实际开发时我们会有许多环境，如本地开发环境、测试环境和线上环境等，不同环境的配置不同（如：MongoDB 的地址），我们不可能每次部署时都要去修改引用 config.test.js 或者 config.production.js。config-lite 模块正是你需要的。

## config-lite

[config-lite](https://www.npmjs.com/package/config-lite) 是一个轻量的读取配置文件的模块。config-lite 会根据环境变量（`NODE_ENV`）的不同加载 config 目录下不同的配置文件。如果不设置 `NODE_ENV`，则读取默认的 default 配置文件，如果设置了 `NODE_ENV`，则会合并指定的配置文件和 default 配置文件作为配置，config-lite 支持 .js、.json、.node、.yml、.yaml 后缀的文件。

如果程序以 `NODE_ENV=test node app` 启动，则 config-lite 会依次降级查找 `config/test.js`、`config/test.json`、`config/test.node`、`config/test.yml`、`config/test.yaml` 并合并 default 配置; 如果程序以 `NODE_ENV=production node app` 启动，则 config-lite 会依次降级查找 `config/production.js`、`config/production.json`、`config/production.node`、`config/production.yml`、`config/production.yaml` 并合并 default 配置。

config-lite 还支持冒泡查找配置，即从传入的路径开始，从该目录不断往上一级目录查找 config 目录，直到找到或者到达根目录为止。

在 myblog 下新建 config 目录，在该目录下新建 default.js，添加如下代码：

**config/default.js**

```js
module.exports = {
  port: 3000,
  session: {
    secret: "myblog",
    key: "myblog",
    maxAge: 2592000000
  },
  mongodb: "mongodb://localhost:27017/myblog"
};
```

配置释义：

1. `port`: 程序启动要监听的端口号
2. `session`: express-session 的配置信息，后面介绍
3. `mongodb`: mongodb 的地址，以 `mongodb://` 协议开头，`myblog` 为 db 名

## 功能与路由设计

在开发博客之前，我们首先需要明确博客要实现哪些功能。由于本教程面向初学者，所以只实现了博客最基本的功能，其余的功能（如归档、标签、分页等等）读者可自行实现。

功能及路由设计如下：

1. 注册
   1. 注册页：`GET /signup`
   2. 注册（包含上传头像）：`POST /signup`
2. 登录
   1. 登录页：`GET /signin`
   2. 登录：`POST /signin`
3. 登出：`GET /signout`
4. 查看文章
   1. 主页：`GET /posts`
   2. 个人主页：`GET /posts?author=xxx`
   3. 查看一篇文章（包含留言）：`GET /posts/:postId`
5. 发表文章
   1. 发表文章页：`GET /posts/create`
   2. 发表文章：`POST /posts/create`
6. 修改文章
   1. 修改文章页：`GET /posts/:postId/edit`
   2. 修改文章：`POST /posts/:postId/edit`
7. 删除文章：`GET /posts/:postId/remove`
8. 留言
   1. 创建留言：`POST /comments`
   2. 删除留言：`GET /comments/:commentId/remove`

由于我们博客页面是后端渲染的，所以只通过简单的 `<a>(GET)` 和 `<form>(POST)` 与后端进行交互，如果使用 jQuery 或者其他前端框架（如 Angular、Vue、React 等等）可通过 Ajax 与后端交互，则 api 的设计应尽量遵循 Restful 风格。

### Restful

Restful 是一种 api 的设计风格，提出了一组 api 的设计原则和约束条件。

如上面删除文章的路由设计：

```
GET /posts/:postId/remove
```

Restful 风格的设计：

```
DELETE /posts/:postId
```

可以看出，Restful 风格的 api 更直观且优雅。

更多阅读：

1. http://www.ruanyifeng.com/blog/2011/09/restful
2. http://www.ruanyifeng.com/blog/2014/05/restful_api.html
3. http://developer.51cto.com/art/200908/141825.htm
4. http://blog.jobbole.com/41233/

## 会话

由于 HTTP 协议是无状态的协议，所以服务端需要记录用户的状态时，就需要用某种机制来识别具体的用户，这个机制就是会话（Session）。

### cookie 与 session 的区别

1. cookie 存储在浏览器（有大小限制），session 存储在服务端（没有大小限制）
2. 通常 session 的实现是基于 cookie 的，session id 存储于 cookie 中
3. session 更安全，cookie 可以直接在浏览器查看甚至编辑

更多 session 的资料，参考：https://www.zhihu.com/question/19786827

我们通过引入 express-session 中间件实现对会话的支持：

```js
app.use(session(options));
```

session 中间件会在 req 上添加 session 对象，即 req.session 初始值为 `{}`，当我们登录后设置 `req.session.user = 用户信息`，返回浏览器的头信息中会带上 `set-cookie` 将 session id 写到浏览器 cookie 中，那么该用户下次请求时，通过带上来的 cookie 中的 session id 我们就可以查找到该用户，并将用户信息保存到 `req.session.user`。

## 页面通知

我们还需要这样一个功能：当我们操作成功时需要显示一个成功的通知，如登录成功跳转到主页时，需要显示一个 `登陆成功` 的通知；当我们操作失败时需要显示一个失败的通知，如注册时用户名被占用了，需要显示一个 `用户名已占用` 的通知。通知只显示一次，刷新后消失，我们可以通过 connect-flash 中间件实现这个功能。

[connect-flash](https://www.npmjs.com/package/connect-flash) 是基于 session 实现的，它的原理很简单：设置初始值 `req.session.flash={}`，通过 `req.flash(name, value)` 设置这个对象下的字段和值，通过 `req.flash(name)` 获取这个对象下的值，同时删除这个字段，实现了只显示一次刷新后消失的功能。

### express-session、connect-mongo 和 connect-flash 的区别与联系

1. `express-session`: 会话（session）支持中间件
2. `connect-mongo`: 将 session 存储于 mongodb，需结合 express-session 使用，我们也可以将 session 存储于 redis，如 [connect-redis](https://www.npmjs.com/package/connect-redis)
3. `connect-flash`: 基于 session 实现的用于通知功能的中间件，需结合 express-session 使用

## 权限控制

不管是论坛还是博客网站，我们没有登录的话只能浏览，登陆后才能发帖或写文章，即使登录了你也不能修改或删除其他人的文章，这就是权限控制。我们也来给博客添加权限控制，如何实现页面的权限控制呢？我们可以把用户状态的检查封装成一个中间件，在每个需要权限控制的路由加载该中间件，即可实现页面的权限控制。在 myblog 下新建 middlewares 目录，在该目录下新建 check.js，添加如下代码：

**middlewares/check.js**

```js
module.exports = {
  checkLogin: function checkLogin(req, res, next) {
    if (!req.session.user) {
      req.flash("error", "未登录");
      return res.redirect("/signin");
    }
    next();
  },

  checkNotLogin: function checkNotLogin(req, res, next) {
    if (req.session.user) {
      req.flash("error", "已登录");
      return res.redirect("back"); // 返回之前的页面
    }
    next();
  }
};
```

可以看出：

1. `checkLogin`: 当用户信息（`req.session.user`）不存在，即认为用户没有登录，则跳转到登录页，同时显示 `未登录` 的通知，用于需要用户登录才能操作的页面
2. `checkNotLogin`: 当用户信息（`req.session.user`）存在，即认为用户已经登录，则跳转到之前的页面，同时显示 `已登录` 的通知，如已登录用户就禁止访问登录、注册页面

最终我们创建以下路由文件：

**routes/index.js**

```js
module.exports = function(app) {
  app.get("/", function(req, res) {
    res.redirect("/posts");
  });
  app.use("/signup", require("./signup"));
  app.use("/signin", require("./signin"));
  app.use("/signout", require("./signout"));
  app.use("/posts", require("./posts"));
  app.use("/comments", require("./comments"));
};
```

**routes/posts.js**

```js
const express = require("express");
const router = express.Router();

const checkLogin = require("../middlewares/check").checkLogin;

// GET /posts 所有用户或者特定用户的文章页
//   eg: GET /posts?author=xxx
router.get("/", function(req, res, next) {
  res.send("主页");
});

// POST /posts/create 发表一篇文章
router.post("/create", checkLogin, function(req, res, next) {
  res.send("发表文章");
});

// GET /posts/create 发表文章页
router.get("/create", checkLogin, function(req, res, next) {
  res.send("发表文章页");
});

// GET /posts/:postId 单独一篇的文章页
router.get("/:postId", function(req, res, next) {
  res.send("文章详情页");
});

// GET /posts/:postId/edit 更新文章页
router.get("/:postId/edit", checkLogin, function(req, res, next) {
  res.send("更新文章页");
});

// POST /posts/:postId/edit 更新一篇文章
router.post("/:postId/edit", checkLogin, function(req, res, next) {
  res.send("更新文章");
});

// GET /posts/:postId/remove 删除一篇文章
router.get("/:postId/remove", checkLogin, function(req, res, next) {
  res.send("删除文章");
});

module.exports = router;
```

**routes/comments.js**

```js
const express = require("express");
const router = express.Router();
const checkLogin = require("../middlewares/check").checkLogin;

// POST /comments 创建一条留言
router.post("/", checkLogin, function(req, res, next) {
  res.send("创建留言");
});

// GET /comments/:commentId/remove 删除一条留言
router.get("/:commentId/remove", checkLogin, function(req, res, next) {
  res.send("删除留言");
});

module.exports = router;
```

**routes/signin.js**

```js
const express = require("express");
const router = express.Router();

const checkNotLogin = require("../middlewares/check").checkNotLogin;

// GET /signin 登录页
router.get("/", checkNotLogin, function(req, res, next) {
  res.send("登录页");
});

// POST /signin 用户登录
router.post("/", checkNotLogin, function(req, res, next) {
  res.send("登录");
});

module.exports = router;
```

**routes/signup.js**

```js
const express = require("express");
const router = express.Router();

const checkNotLogin = require("../middlewares/check").checkNotLogin;

// GET /signup 注册页
router.get("/", checkNotLogin, function(req, res, next) {
  res.send("注册页");
});

// POST /signup 用户注册
router.post("/", checkNotLogin, function(req, res, next) {
  res.send("注册");
});

module.exports = router;
```

**routes/signout.js**

```js
const express = require("express");
const router = express.Router();

const checkLogin = require("../middlewares/check").checkLogin;

// GET /signout 登出
router.get("/", checkLogin, function(req, res, next) {
  res.send("登出");
});

module.exports = router;
```

最后，修改 index.js 如下：

**index.js**

```js
const path = require("path");
const express = require("express");
const session = require("express-session");
const MongoStore = require("connect-mongo")(session);
const flash = require("connect-flash");
const config = require("config-lite")(__dirname);
const routes = require("./routes");
const pkg = require("./package");

const app = express();

// 设置模板目录
app.set("views", path.join(__dirname, "views"));
// 设置模板引擎为 ejs
app.set("view engine", "ejs");

// 设置静态文件目录
app.use(express.static(path.join(__dirname, "public")));
// session 中间件
app.use(
  session({
    name: config.session.key, // 设置 cookie 中保存 session id 的字段名称
    secret: config.session.secret, // 通过设置 secret 来计算 hash 值并放在 cookie 中，使产生的 signedCookie 防篡改
    resave: true, // 强制更新 session
    saveUninitialized: false, // 设置为 false，强制创建一个 session，即使用户未登录
    cookie: {
      maxAge: config.session.maxAge // 过期时间，过期后 cookie 中的 session id 自动删除
    },
    store: new MongoStore({
      // 将 session 存储到 mongodb
      url: config.mongodb // mongodb 地址
    })
  })
);
// flash 中间件，用来显示通知
app.use(flash());

// 路由
routes(app);

// 监听端口，启动程序
app.listen(config.port, function() {
  console.log(`${pkg.name} listening on port ${config.port}`);
});
```

> 注意：中间件的加载顺序很重要。如上面设置静态文件目录的中间件应该放到 routes(app) 之前加载，这样静态文件的请求就不会落到业务逻辑的路由里；flash 中间件应该放到 session 中间件之后加载，因为 flash 是基于 session 实现的。

运行 `supervisor index` 启动博客，访问以下地址查看效果：

1. http://localhost:3000/posts
2. http://localhost:3000/signout
3. http://localhost:3000/signup
   我们使用 jQuery + Semantic-UI 实现前端页面的设计，最终效果图如下:

**注册页**

![](/images/4.5.1.png)

**登录页**

![](/images/4.5.2.png)

**未登录时的主页（或用户页）**

![](/images/4.5.3.png)

**登录后的主页（或用户页）**

![](/images/4.5.4.png)

**发表文章页**

![](/images/4.5.5.png)

**编辑文章页**

![](/images/4.5.6.png)

**未登录时的文章页**

![](/images/4.5.7.png)

**登录后的文章页**

![](/images/4.5.8.png)

**通知**

![](/images/4.5.9.png)
![](/images/4.5.10.png)
![](/images/4.5.11.png)

## 组件

前面提到过，我们可以将模板拆分成一些组件，然后使用 ejs 的 include 方法将组件组合起来进行渲染。我们将页面切分成以下组件：

**主页**

![](/images/4.5.12.png)

**文章页**

![](/images/4.5.13.png)

根据上面的组件切分图，我们创建以下样式及模板文件：

**public/css/style.css**

```css
/* ---------- 全局样式 ---------- */

body {
  width: 1100px;
  height: 100%;
  margin: 0 auto;
  padding-top: 40px;
}

a:hover {
  border-bottom: 3px solid #4fc08d;
}

.button {
  background-color: #4fc08d !important;
  color: #fff !important;
}

.avatar {
  border-radius: 3px;
  width: 48px;
  height: 48px;
  float: right;
}

/* ---------- nav ---------- */

.nav {
  margin-bottom: 20px;
  color: #999;
  text-align: center;
}

.nav h1 {
  color: #4fc08d;
  display: inline-block;
  margin: 10px 0;
}

/* ---------- nav-setting ---------- */

.nav-setting {
  position: fixed;
  right: 30px;
  top: 35px;
  z-index: 999;
}

.nav-setting .ui.dropdown.button {
  padding: 10px 10px 0 10px;
  background-color: #fff !important;
}

.nav-setting .icon.bars {
  color: #000;
  font-size: 18px;
}

/* ---------- post-content ---------- */

.post-content h3 a {
  color: #4fc08d !important;
}

.post-content .tag {
  font-size: 13px;
  margin-right: 5px;
  color: #999;
}

.post-content .tag.right {
  float: right;
  margin-right: 0;
}

.post-content .tag.right a {
  color: #999;
}
```

**views/header.ejs**

```ejs
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title><%= blog.title %></title>
    <link rel="stylesheet" href="//cdn.bootcss.com/semantic-ui/2.1.8/semantic.min.css">
    <link rel="stylesheet" href="/css/style.css">
    <script src="//cdn.bootcss.com/jquery/1.11.3/jquery.min.js"></script>
    <script src="//cdn.bootcss.com/semantic-ui/2.1.8/semantic.min.js"></script>
  </head>
  <body>
  <%- include('components/nav') %>
  <%- include('components/nav-setting') %>
  <%- include('components/notification') %>
```

**views/footer.ejs**

```ejs
  <script type="text/javascript">
   $(document).ready(function () {
      // 点击按钮弹出下拉框
      $('.ui.dropdown').dropdown();

      // 鼠标悬浮在头像上，弹出气泡提示框
      $('.post-content .avatar-link').popup({
        inline: true,
        position: 'bottom right',
        lastResort: 'bottom right'
      });
    })
  </script>
  </body>
</html>
```

> 注意：上面 `<script></script>` 是 semantic-ui 操控页面控件的代码，一定要放到 footer.ejs 的 `</body>` 的前面，因为只有页面加载完后才能通过 JQuery 获取 DOM 元素。

在 views 目录下新建 components 目录用来存放组件（即可以复用的模板片段），在该目录下创建以下文件：

**views/components/nav.ejs**

```ejs
<div class="nav">
  <div class="ui grid">
    <div class="four wide column"></div>

    <div class="eight wide column">
      <a href="/posts"><h1><%= blog.title %></h1></a>
      <p><%= blog.description %></p>
    </div>
  </div>
</div>
```

**views/components/nav-setting.ejs**

```ejs
<div class="nav-setting">
  <div class="ui buttons">
    <div class="ui floating dropdown button">
      <i class="icon bars"></i>
      <div class="menu">
        <% if (user) { %>
          <a class="item" href="/posts?author=<%= user._id %>">个人主页</a>
          <div class="divider"></div>
          <a class="item" href="/posts/create">发表文章</a>
          <a class="item" href="/signout">登出</a>
        <% } else { %>
          <a class="item" href="/signin">登录</a>
          <a class="item" href="/signup">注册</a>
        <% } %>
      </div>
    </div>
  </div>
</div>
```

**views/components/notification.ejs**

```ejs
<div class="ui grid">
  <div class="four wide column"></div>
  <div class="eight wide column">

  <% if (success) { %>
    <div class="ui success message">
      <p><%= success %></p>
    </div>
  <% } %>

  <% if (error) { %>
    <div class="ui error message">
      <p><%= error %></p>
    </div>
  <% } %>

  </div>
</div>
```

## app.locals 和 res.locals

上面的 ejs 模板中我们用到了 blog、user、success、error 变量，我们将 blog 变量挂载到 `app.locals` 下，将 user、success、error 挂载到 `res.locals` 下。为什么要这么做呢？`app.locals` 和 `res.locals` 是什么？它们有什么区别？

express 中有两个对象可用于模板的渲染：`app.locals` 和 `res.locals`。我们从 express 源码一探究竟：

**express/lib/application.js**

```js
app.render = function render(name, options, callback) {
  ...
  var opts = options;
  var renderOptions = {};
  ...
  // merge app.locals
  merge(renderOptions, this.locals);

  // merge options._locals
  if (opts._locals) {
    merge(renderOptions, opts._locals);
  }

  // merge options
  merge(renderOptions, opts);
  ...
  tryRender(view, renderOptions, done);
};
```

**express/lib/response.js**

```js
res.render = function render(view, options, callback) {
  var app = this.req.app;
  var opts = options || {};
  ...
  // merge res.locals
  opts._locals = self.locals;
  ...
  // render
  app.render(view, opts, done);
};
```

可以看出：在调用 `res.render` 的时候，express 合并（merge）了 3 处的结果后传入要渲染的模板，优先级：`res.render` 传入的对象> `res.locals` 对象 > `app.locals` 对象，所以 `app.locals` 和 `res.locals` 几乎没有区别，都用来渲染模板，使用上的区别在于：`app.locals` 上通常挂载常量信息（如博客名、描述、作者这种不会变的信息），`res.locals` 上通常挂载变量信息，即每次请求可能的值都不一样（如请求者信息，`res.locals.user = req.session.user`）。

修改 index.js，在 `routes(app)` 上一行添加如下代码：

```js
// 设置模板全局常量
app.locals.blog = {
  title: pkg.name,
  description: pkg.description
};

// 添加模板必需的三个变量
app.use(function(req, res, next) {
  res.locals.user = req.session.user;
  res.locals.success = req.flash("success").toString();
  res.locals.error = req.flash("error").toString();
  next();
});
```

这样在调用 `res.render` 的时候就不用传入这四个变量了，express 为我们自动 merge 并传入了模板，所以我们可以在模板中直接使用这四个变量。
我们使用 [Mongolass](https://github.com/mongolass/mongolass) 这个模块操作 mongodb 进行增删改查。在 myblog 下新建 lib 目录，在该目录下新建 mongo.js，添加如下代码：

**lib/mongo.js**

```js
const config = require('config-lite')(__dirname)
const Mongolass = require('mongolass')
const mongolass = new Mongolass()
mongolass.connect(config.mongodb)
```

## 为什么使用 Mongolass

早期我使用官方的 [mongodb](https://www.npmjs.com/package/mongodb)（也叫 node-mongodb-native）库，后来也陆续尝试使用了许多其他 mongodb 的驱动库，[Mongoose](https://www.npmjs.com/package/mongoose) 是比较优秀的一个，使用 Mongoose 的时间也比较长。比较这两者，各有优缺点。

### node-mongodb-native:

**优点：**

1. 简单。参照文档即可上手，没有 Mongoose 的 Schema 那些对新手不友好的东西。
2. 强大。毕竟是官方库，包含了所有且最新的 api，其他大部分的库都是在这个库的基础上改造的，包括 Mongoose。
3. 文档健全。

**缺点：**

1. 起初只支持 callback，会写出以下这种代码：
```js
mongodb.open(function (err, db) {
  if (err) {
    return callback(err)
  }
  db.collection('users', function (err, collection) {
    if (err) {
      return callback(err)
    }
    collection.find({ name: 'xxx' }, function (err, users) {
      if (err) {
        return callback(err)
      }
    })
  ...
```

或者：

```js
MongoClient.connect('mongodb://localhost:27017', function (err, mongodb) {
  if (err) {
    return callback(err)
  }
  mongodb.db('test').collection('users').find({ name: 'xxx' }, function (err, users) {
    if (err) {
      return callback(err)
    }
  })
  ...
```

现在支持 Promise 了，和 co 一起使用好很多。

2. 不支持文档校验。Mongoose 通过 Schema 支持文档校验，虽说 mongodb 是 no schema 的，但在生产环境中使用 Schema 有两点好处。一是对文档做校验，防止非正常情况下写入错误的数据到数据库，二是可以简化一些代码，如类型为 ObjectId 的字段查询或更新时可通过对应的字符串操作，不用每次包装成 ObjectId 对象。

### Mongoose:

**优点：**

1. 封装了数据库的操作，给人的感觉是同步的，其实内部是异步的。如 mongoose 与 MongoDB 建立连接：
```js
const mongoose = require('mongoose')
mongoose.connect('mongodb://localhost/test')
const BlogModel = mongoose.model('Blog', { title: String, content: String })
BlogModel.find()
```
2. 支持 Promise。这个也无需多说，Promise 是未来趋势，可结合 co 使用，也可结合 async/await 使用。
3. 支持文档校验。如上所述。

**缺点（个人观点）：**

1. 功能多，复杂。Mongoose 功能很强大，包括静态方法，实例方法，虚拟属性，hook 函数等等，混用带来的后果是逻辑复杂，代码难以维护。
2. 较弱的 plugin 系统。如：`schema.pre('save', function(next) {})` 和 `schema.post('find', function(next) {})`，只支持异步 `next()`，灵活性大打折扣。
3. 其他：对新手来说难以理解的 Schema、Model、Entity 之间的关系；容易混淆的 toJSON 和 toObject，以及有带有虚拟属性的情况；用和不用 exec 的情况以及直接用 then 的情况；返回的结果是 Mongoose 包装后的对象，在此对象上修改结果却无效等等。

### Mongolass

Mongolass 保持了与 mongodb 一样的 api，又借鉴了许多 Mongoose 的优点，同时又保持了精简。

**优点：**

1. 支持 Promise。
2. 官方一致的 api。
2. 简单。参考 Mongolass 的 readme 即可上手，比 Mongoose 精简的多，本身代码也不多。
3. 可选的 Schema。Mongolass 中的 Schema （基于 [another-json-schema](https://www.npmjs.com/package/another-json-schema)）是可选的，并且只用来做文档校验。如果定义了 schema 并关联到某个 model，则插入、更新和覆盖等操作都会校验文档是否满足 schema，同时 schema 也会尝试格式化该字段，类似于 Mongoose，如定义了一个字段为 ObjectId 类型，也可以用 ObjectId 的字符串无缝使用一样。如果没有 schema，则用法跟原生 mongodb 库一样。
4. 简单却强大的插件系统。可以定义全局插件（对所有 model 生效），也可以定义某个 model 上的插件（只对该 model 生效）。Mongolass 插件的设计思路借鉴了中间件的概念（类似于 Koa），通过定义 `beforeXXX` 和 `afterXXX` （XXX为操作符首字母大写，如：`afterFind`）函数实现，函数返回 yieldable 的对象即可，所以每个插件内可以做一些其他的 IO 操作。不同的插件顺序会有不同的结果，而且每个插件的输入输出都是 plain object，而非类 Mongoose 包装后的对象，没有虚拟属性，无需调用 toJSON 或 toObject。Mongolass 中的 `.populate()`就是一个内置的插件。
5. 详细的错误信息。用过 Mongoose 的人一定遇到过这样的错：
   `CastError: Cast to ObjectId failed for value "xxx" at path "_id"`
   只知道一个期望是 ObjectId 的字段传入了非期望的值，通常很难定位出错的代码，即使定位到也得不到错误现场。得益于 [another-json-schema](https://www.npmjs.com/package/another-json-schema)，使用 Mongolass 在查询或者更新时，某个字段不匹配它定义的 schema 时（还没落到 mongodb）会给出详细的错误信息，如下所示：
```js
const Mongolass = require('mongolass')
const mongolass = new Mongolass('mongodb://localhost:27017/test')

const User = mongolass.model('User', {
  name: { type: 'string' },
  age: { type: 'number' }
})

User
  .insertOne({ name: 'nswbmw', age: 'wrong age' })
  .exec()
  .then(console.log)
  .catch(function (e) {
    console.error(e)
    console.error(e.stack)
  })
/*
{ [Error: ($.age: "wrong age") ✖ (type: number)]
  validator: 'type',
  actual: 'wrong age',
  expected: { type: 'number' },
  path: '$.age',
  schema: 'UserSchema',
  model: 'User',
  plugin: 'MongolassSchema',
  type: 'beforeInsertOne',
  args: [] }
Error: ($.age: "wrong age") ✖ (type: number)
    at Model.insertOne (/Users/nswbmw/Desktop/mongolass-demo/node_modules/mongolass/lib/query.js:108:16)
    at Object.<anonymous> (/Users/nswbmw/Desktop/mongolass-demo/app.js:10:4)
    at Module._compile (module.js:409:26)
    at Object.Module._extensions..js (module.js:416:10)
    at Module.load (module.js:343:32)
    at Function.Module._load (module.js:300:12)
    at Function.Module.runMain (module.js:441:10)
    at startup (node.js:139:18)
    at node.js:974:3
 */
```
可以看出，错误的原因是在 insertOne 一条用户数据到用户表的时候，age 期望是一个 number 类型的值，而我们传入的字符串 `wrong age`，然后从错误栈中可以快速定位到是 app.js 第 10 行代码抛出的错。

**缺点：**

1. ~~schema 功能较弱，缺少如 required、default 功能。~~

### 扩展阅读

[从零开始写一个 Node.js 的 MongoDB 驱动库](https://zhuanlan.zhihu.com/p/24308524)

## 用户模型设计

我们只存储用户的名称、密码（加密后的）、头像、性别和个人简介这几个字段，对应修改 lib/mongo.js，添加如下代码：

**lib/mongo.js**

```js
exports.User = mongolass.model('User', {
  name: { type: 'string', required: true },
  password: { type: 'string', required: true },
  avatar: { type: 'string', required: true },
  gender: { type: 'string', enum: ['m', 'f', 'x'], default: 'x' },
  bio: { type: 'string', required: true }
})
exports.User.index({ name: 1 }, { unique: true }).exec()// 根据用户名找到用户，用户名全局唯一
```

我们定义了用户表的 schema，生成并导出了 User 这个 model，同时设置了 name 的唯一索引，保证用户名是不重复的。

> 小提示：`required: true` 表示该字段是必需的，`default: xxx` 用于创建文档时设置默认值。更多关于 Mongolass 的 schema 的用法，请查阅 [another-json-schema](https://github.com/nswbmw/another-json-schema)。

> 小提示：Mongolass 中的 model 你可以认为相当于 mongodb 中的 collection，只不过添加了插件的功能。

## 注册页

首先，我们来完成注册。新建 views/signup.ejs，添加如下代码：

**views/signup.ejs**

```ejs
<%- include('header') %>

<div class="ui grid">
  <div class="four wide column"></div>
  <div class="eight wide column">
    <form class="ui form segment" method="post" enctype="multipart/form-data">
      <div class="field required">
        <label>用户名</label>
        <input placeholder="用户名" type="text" name="name">
      </div>
      <div class="field required">
        <label>密码</label>
        <input placeholder="密码" type="password" name="password">
      </div>
      <div class="field required">
        <label>重复密码</label>
        <input placeholder="重复密码" type="password" name="repassword">
      </div>
      <div class="field required">
        <label>性别</label>
        <select class="ui compact selection dropdown" name="gender">
          <option value="m">男</option>
          <option value="f">女</option>
          <option value="x">保密</option>
        </select>
      </div>
      <div class="field required">
        <label>头像</label>
        <input type="file" name="avatar">
      </div>
      <div class="field required">
        <label>个人简介</label>
        <textarea name="bio" rows="5"></textarea>
      </div>
      <input type="submit" class="ui button fluid" value="注册">
    </form>
  </div>
</div>

<%- include('footer') %>
```

> 注意：form 表单要添加 `enctype="multipart/form-data"` 属性才能上传文件。

修改 routes/signup.js 中获取注册页的路由如下：

**routes/signup.js**

```js
// GET /signup 注册页
router.get('/', checkNotLogin, function (req, res, next) {
  res.render('signup')
})
```

现在访问 `localhost:3000/signup` 看看效果吧。

## 注册与文件上传

我们使用 [express-formidable](https://github.com/utatti/express-formidable) 处理 form 表单（包括文件上传）。修改 index.js ，在 `app.use(flash())` 下一行添加如下代码：

**index.js**

```js
// 处理表单及文件上传的中间件
app.use(require('express-formidable')({
  uploadDir: path.join(__dirname, 'public/img'), // 上传文件目录
  keepExtensions: true// 保留后缀
}))
```

新建 models/users.js，添加如下代码：

**models/users.js**

```js
const User = require('../lib/mongo').User

module.exports = {
  // 注册一个用户
  create: function create (user) {
    return User.create(user).exec()
  }
}
```

完善处理用户注册的路由，最终修改 routes/signup.js 如下：

**routes/signup.js**

```js
const fs = require('fs')
const path = require('path')
const sha1 = require('sha1')
const express = require('express')
const router = express.Router()

const UserModel = require('../models/users')
const checkNotLogin = require('../middlewares/check').checkNotLogin

// GET /signup 注册页
router.get('/', checkNotLogin, function (req, res, next) {
  res.render('signup')
})

// POST /signup 用户注册
router.post('/', checkNotLogin, function (req, res, next) {
  const name = req.fields.name
  const gender = req.fields.gender
  const bio = req.fields.bio
  const avatar = req.files.avatar.path.split(path.sep).pop()
  let password = req.fields.password
  const repassword = req.fields.repassword

  // 校验参数
  try {
    if (!(name.length >= 1 && name.length <= 10)) {
      throw new Error('名字请限制在 1-10 个字符')
    }
    if (['m', 'f', 'x'].indexOf(gender) === -1) {
      throw new Error('性别只能是 m、f 或 x')
    }
    if (!(bio.length >= 1 && bio.length <= 30)) {
      throw new Error('个人简介请限制在 1-30 个字符')
    }
    if (!req.files.avatar.name) {
      throw new Error('缺少头像')
    }
    if (password.length < 6) {
      throw new Error('密码至少 6 个字符')
    }
    if (password !== repassword) {
      throw new Error('两次输入密码不一致')
    }
  } catch (e) {
    // 注册失败，异步删除上传的头像
    fs.unlink(req.files.avatar.path)
    req.flash('error', e.message)
    return res.redirect('/signup')
  }

  // 明文密码加密
  password = sha1(password)

  // 待写入数据库的用户信息
  let user = {
    name: name,
    password: password,
    gender: gender,
    bio: bio,
    avatar: avatar
  }
  // 用户信息写入数据库
  UserModel.create(user)
    .then(function (result) {
      // 此 user 是插入 mongodb 后的值，包含 _id
      user = result.ops[0]
      // 删除密码这种敏感信息，将用户信息存入 session
      delete user.password
      req.session.user = user
      // 写入 flash
      req.flash('success', '注册成功')
      // 跳转到首页
      res.redirect('/posts')
    })
    .catch(function (e) {
      // 注册失败，异步删除上传的头像
      fs.unlink(req.files.avatar.path)
      // 用户名被占用则跳回注册页，而不是错误页
      if (e.message.match('duplicate key')) {
        req.flash('error', '用户名已被占用')
        return res.redirect('/signup')
      }
      next(e)
    })
})

module.exports = router
```

我们使用 express-formidable 处理表单的上传，表单普通字段挂载到 req.fields 上，表单上传后的文件挂载到 req.files 上，文件存储在 public/img 目录下。然后校验了参数，校验通过后将用户信息插入到 MongoDB 中，成功则跳转到主页并显示『注册成功』的通知，失败（如用户名被占用）则跳转回注册页面并显示『用户名已被占用』的通知。

> 注意：我们使用 sha1 加密用户的密码，sha1 并不是一种十分安全的加密方式，实际开发中可以使用更安全的 [bcrypt](https://www.npmjs.com/package/bcrypt) 或 [scrypt](https://www.npmjs.com/package/scrypt) 加密。
> 注意：注册失败时（参数校验失败或者存数据库时出错）删除已经上传到 public/img 目录下的头像。

为了方便观察效果，我们先创建主页的模板。修改 routes/posts.js 中对应代码如下：

**routes/posts.js**

```js
router.get('/', function (req, res, next) {
  res.render('posts')
})
```

新建 views/posts.ejs，添加如下代码：

**views/posts.ejs**

```ejs
<%- include('header') %>
这是主页
<%- include('footer') %>
```

访问 `localhost:3000/signup`，注册成功后如下所示：

![](/images/4.7.1.png)

## 登出

现在我们来完成登出的功能。修改 routes/signout.js 如下：

**routes/signout.js**

```js
const express = require('express')
const router = express.Router()

const checkLogin = require('../middlewares/check').checkLogin

// GET /signout 登出
router.get('/', checkLogin, function (req, res, next) {
  // 清空 session 中用户信息
  req.session.user = null
  req.flash('success', '登出成功')
  // 登出成功后跳转到主页
  res.redirect('/posts')
})

module.exports = router
```

此时刷新页面，点击右上角的 `登出`，成功后如下图所示：

![](/images/4.8.1.png)

## 登录页

现在我们来完成登录页。修改 routes/signin.js 相应代码如下：

**routes/signin.js**

```js
router.get('/', checkNotLogin, function (req, res, next) {
  res.render('signin')
})
```

新建 views/signin.ejs，添加如下代码：

**views/signin.ejs**

```ejs
<%- include('header') %>

<div class="ui grid">
  <div class="four wide column"></div>
  <div class="eight wide column">
    <form class="ui form segment" method="post">
      <div class="field required">
        <label>用户名</label>
        <input placeholder="用户名" type="text" name="name">
      </div>
      <div class="field required">
        <label>密码</label>
        <input placeholder="密码" type="password" name="password">
      </div>
      <input type="submit" class="ui button fluid" value="登录">
    </form>  
  </div>
</div>

<%- include('footer') %>
```

现在刷新页面，点击右边上角 `登录` 试试吧，我们已经看到了登录页，但先不要点击登录，接下来我们实现处理登录的逻辑。

## 登录

现在我们来完成登录的功能。修改 models/users.js 添加 `getUserByName` 方法用于通过用户名获取用户信息：

**models/users.js**

```js
const User = require('../lib/mongo').User

module.exports = {
  // 注册一个用户
  create: function create (user) {
    return User.create(user).exec()
  },

  // 通过用户名获取用户信息
  getUserByName: function getUserByName (name) {
    return User
      .findOne({ name: name })
      .addCreatedAt()
      .exec()
  }
}
```

这里我们使用了 `addCreatedAt` 自定义插件（通过 \_id 生成时间戳），修改 lib/mongo.js，添加如下代码：

**lib/mongo.js**

```js
const moment = require('moment')
const objectIdToTimestamp = require('objectid-to-timestamp')

// 根据 id 生成创建时间 created_at
mongolass.plugin('addCreatedAt', {
  afterFind: function (results) {
    results.forEach(function (item) {
      item.created_at = moment(objectIdToTimestamp(item._id)).format('YYYY-MM-DD HH:mm')
    })
    return results
  },
  afterFindOne: function (result) {
    if (result) {
      result.created_at = moment(objectIdToTimestamp(result._id)).format('YYYY-MM-DD HH:mm')
    }
    return result
  }
})
```

> 小提示：24 位长的 ObjectId 前 4 个字节是精确到秒的时间戳，所以我们没有额外的存创建时间（如: createdAt）的字段。ObjectId 生成规则：

![](/images/4.8.2.png)


修改 routes/signin.js 如下：

**routes/signin.js**

```js
const sha1 = require('sha1')
const express = require('express')
const router = express.Router()

const UserModel = require('../models/users')
const checkNotLogin = require('../middlewares/check').checkNotLogin

// GET /signin 登录页
router.get('/', checkNotLogin, function (req, res, next) {
  res.render('signin')
})

// POST /signin 用户登录
router.post('/', checkNotLogin, function (req, res, next) {
  const name = req.fields.name
  const password = req.fields.password

  // 校验参数
  try {
    if (!name.length) {
      throw new Error('请填写用户名')
    }
    if (!password.length) {
      throw new Error('请填写密码')
    }
  } catch (e) {
    req.flash('error', e.message)
    return res.redirect('back')
  }

  UserModel.getUserByName(name)
    .then(function (user) {
      if (!user) {
        req.flash('error', '用户不存在')
        return res.redirect('back')
      }
      // 检查密码是否匹配
      if (sha1(password) !== user.password) {
        req.flash('error', '用户名或密码错误')
        return res.redirect('back')
      }
      req.flash('success', '登录成功')
      // 用户信息写入 session
      delete user.password
      req.session.user = user
      // 跳转到主页
      res.redirect('/posts')
    })
    .catch(next)
})

module.exports = router
```

这里我们在 POST /signin 的路由处理函数中，通过传上来的 name 去数据库中找到对应用户，校验传上来的密码是否跟数据库中的一致。不一致则返回上一页（即登录页）并显示『用户名或密码错误』的通知，一致则将用户信息写入 session，跳转到主页并显示『登录成功』的通知。

现在刷新页面，点击右上角 `登录`，用刚才注册的账号登录，如下图所示：

![](/images/4.8.3.png)

## 文章模型设计

我们只存储文章的作者 id、标题、正文和点击量这几个字段，对应修改 lib/mongo.js，添加如下代码：

**lib/mongo.js**

```js
exports.Post = mongolass.model('Post', {
  author: { type: Mongolass.Types.ObjectId, required: true },
  title: { type: 'string', required: true },
  content: { type: 'string', required: true },
  pv: { type: 'number', default: 0 }
})
exports.Post.index({ author: 1, _id: -1 }).exec()// 按创建时间降序查看用户的文章列表
```

## 发表文章

现在我们来实现发表文章的功能。首先创建发表文章页，新建 views/create.ejs，添加如下代码：

**views/create.ejs**

```ejs
<%- include('header') %>

<div class="ui grid">
  <div class="four wide column">
    <a class="avatar avatar-link"
       href="/posts?author=<%= user._id %>"
       data-title="<%= user.name %> | <%= ({m: '男', f: '女', x: '保密'})[user.gender] %>"
       data-content="<%= user.bio %>">
      <img class="avatar" src="/img/<%= user.avatar %>">
    </a>
  </div>

  <div class="eight wide column">
    <form class="ui form segment" method="post">
      <div class="field required">
        <label>标题</label>
        <input type="text" name="title">
      </div>
      <div class="field required">
        <label>内容</label>
        <textarea name="content" rows="15"></textarea>
      </div>
      <input type="submit" class="ui button" value="发布">
    </form>
  </div>
</div>

<%- include('footer') %>
```

修改 routes/posts.js，将：

```js
// GET /posts/create 发表文章页
router.get('/create', checkLogin, function (req, res, next) {
  res.send('发表文章页')
})
```

修改为：

```js
// GET /posts/create 发表文章页
router.get('/create', checkLogin, function (req, res, next) {
  res.render('create')
})
```

登录成功状态，点击右上角『发表文章』试下吧。

发表文章页已经完成了，接下来新建 models/posts.js 用来存放与文章操作相关的代码：

**models/posts.js**

```js
const Post = require('../lib/mongo').Post

module.exports = {
  // 创建一篇文章
  create: function create (post) {
    return Post.create(post).exec()
  }
}
```

修改 routes/posts.js，在文件上方引入 PostModel：

**routes/posts.js**

```js
const PostModel = require('../models/posts')
```

将：

```js
// POST /posts/create 发表一篇文章
router.post('/create', checkLogin, function (req, res, next) {
  res.send('发表文章')
})
```

修改为：

```js
// POST /posts/create 发表一篇文章
router.post('/create', checkLogin, function (req, res, next) {
  const author = req.session.user._id
  const title = req.fields.title
  const content = req.fields.content

  // 校验参数
  try {
    if (!title.length) {
      throw new Error('请填写标题')
    }
    if (!content.length) {
      throw new Error('请填写内容')
    }
  } catch (e) {
    req.flash('error', e.message)
    return res.redirect('back')
  }

  let post = {
    author: author,
    title: title,
    content: content
  }

  PostModel.create(post)
    .then(function (result) {
      // 此 post 是插入 mongodb 后的值，包含 _id
      post = result.ops[0]
      req.flash('success', '发表成功')
      // 发表成功后跳转到该文章页
      res.redirect(`/posts/${post._id}`)
    })
    .catch(next)
})
```

这里校验了上传的表单字段，并将文章信息插入数据库，成功后跳转到该文章页并显示『发表成功』的通知，失败后请求会进入错误处理函数。

现在刷新页面（登录情况下），点击右上角 `发表文章` 试试吧，发表成功后跳转到了文章页但并没有任何内容，下面我们就来实现文章页及主页。

## 主页与文章页

现在我们来实现主页及文章页。修改 models/posts.js 如下：

**models/posts.js**

```js
const marked = require('marked')
const Post = require('../lib/mongo').Post

// 将 post 的 content 从 markdown 转换成 html
Post.plugin('contentToHtml', {
  afterFind: function (posts) {
    return posts.map(function (post) {
      post.content = marked(post.content)
      return post
    })
  },
  afterFindOne: function (post) {
    if (post) {
      post.content = marked(post.content)
    }
    return post
  }
})

module.exports = {
  // 创建一篇文章
  create: function create (post) {
    return Post.create(post).exec()
  },

  // 通过文章 id 获取一篇文章
  getPostById: function getPostById (postId) {
    return Post
      .findOne({ _id: postId })
      .populate({ path: 'author', model: 'User' })
      .addCreatedAt()
      .contentToHtml()
      .exec()
  },

  // 按创建时间降序获取所有用户文章或者某个特定用户的所有文章
  getPosts: function getPosts (author) {
    const query = {}
    if (author) {
      query.author = author
    }
    return Post
      .find(query)
      .populate({ path: 'author', model: 'User' })
      .sort({ _id: -1 })
      .addCreatedAt()
      .contentToHtml()
      .exec()
  },

  // 通过文章 id 给 pv 加 1
  incPv: function incPv (postId) {
    return Post
      .update({ _id: postId }, { $inc: { pv: 1 } })
      .exec()
  }
}
```

需要讲解两点：

1. 我们使用了 markdown 解析文章的内容，所以在发表文章的时候可使用 markdown 语法（如插入链接、图片等等），关于 markdown 的使用请参考： [Markdown 语法说明](http://wowubuntu.com/markdown/)。
2. 我们在 PostModel 上注册了 `contentToHtml`，而 `addCreatedAt` 是在 lib/mongo.js 中 mongolass 上注册的。也就是说 `contentToHtml` 只针对 PostModel 有效，而 `addCreatedAt` 对所有 Model 都有效。

接下来完成主页的模板，修改 views/posts.ejs 如下：

**views/posts.ejs**

```ejs
<%- include('header') %>

<% posts.forEach(function (post) { %>
  <%- include('components/post-content', { post: post }) %>
<% }) %>

<%- include('footer') %>
```

新建 views/components/post-content.ejs 用来存放单篇文章的模板片段：

**views/components/post-content.ejs**

```ejs
<div class="post-content">
  <div class="ui grid">
    <div class="four wide column">
      <a class="avatar avatar-link"
         href="/posts?author=<%= post.author._id %>"
         data-title="<%= post.author.name %> | <%= ({m: '男', f: '女', x: '保密'})[post.author.gender] %>"
         data-content="<%= post.author.bio %>">
        <img class="avatar" src="/img/<%= post.author.avatar %>">
      </a>
    </div>

    <div class="eight wide column">
      <div class="ui segment">
        <h3><a href="/posts/<%= post._id %>"><%= post.title %></a></h3>
        <pre><%- post.content %></pre>
        <div>
          <span class="tag"><%= post.created_at %></span>
          <span class="tag right">
            <span>浏览(<%= post.pv || 0 %>)</span>
            <span>留言(<%= post.commentsCount || 0 %>)</span>

            <% if (user && post.author._id && user._id.toString() === post.author._id.toString()) { %>
              <div class="ui inline dropdown">
                <div class="text"></div>
                <i class="dropdown icon"></i>
                <div class="menu">
                  <div class="item"><a href="/posts/<%= post._id %>/edit">编辑</a></div>
                  <div class="item"><a href="/posts/<%= post._id %>/remove">删除</a></div>
                </div>
              </div>
            <% } %>

          </span>
        </div>
      </div>
    </div>
  </div>
</div>
```

> 注意：我们用了 `<%- post.content %>`，而不是 `<%= post.content %>`，因为 post.content 是 markdown 转换后的 html 字符串。

修改 routes/posts.js，将：

**routes/posts.js**

```js
router.get('/', function (req, res, next) {
  res.render('posts')
})
```

修改为：

```js
router.get('/', function (req, res, next) {
  const author = req.query.author

  PostModel.getPosts(author)
    .then(function (posts) {
      res.render('posts', {
        posts: posts
      })
    })
    .catch(next)
})
```

> 注意：主页与用户页通过 url 中的 author 区分。

现在完成了主页与用户页，访问 `http://localhost:3000/posts` 试试吧，现在已经将我们之前创建的文章显示出来了，尝试点击用户的头像看看效果。

接下来完成文章详情页。新建 views/post.ejs，添加如下代码：

**views/post.ejs**

```ejs
<%- include('header') %>
<%- include('components/post-content') %>
<%- include('footer') %>
```

打开 routes/posts.js，将：

**routes/posts.js**

```js
// GET /posts/:postId 单独一篇的文章页
router.get('/:postId', function (req, res, next) {
  res.send('文章详情页')
})
```

修改为：

```js
// GET /posts/:postId 单独一篇的文章页
router.get('/:postId', function (req, res, next) {
  const postId = req.params.postId

  Promise.all([
    PostModel.getPostById(postId), // 获取文章信息
    PostModel.incPv(postId)// pv 加 1
  ])
    .then(function (result) {
      const post = result[0]
      if (!post) {
        throw new Error('该文章不存在')
      }

      res.render('post', {
        post: post
      })
    })
    .catch(next)
})
```

现在刷新浏览器，点击文章的标题看看浏览器地址的变化吧。

> 注意：浏览器地址有变化，但页面看不出区别来（因为页面布局一样），后面我们添加留言功能后就能看出区别来了。

## 编辑与删除文章

现在我们来完成编辑与删除文章的功能。修改 models/posts.js，在 module.exports 对象上添加如下 3 个方法：

**models/posts.js**

```js
// 通过文章 id 获取一篇原生文章（编辑文章）
getRawPostById: function getRawPostById (postId) {
  return Post
    .findOne({ _id: postId })
    .populate({ path: 'author', model: 'User' })
    .exec()
},

// 通过文章 id 更新一篇文章
updatePostById: function updatePostById (postId, data) {
  return Post.update({ _id: postId }, { $set: data }).exec()
},

// 通过文章 id 删除一篇文章
delPostById: function delPostById (postId) {
  return Post.deleteOne({ _id: postId }).exec()
}
```

> 注意：不要忘了在适当位置添加逗号，如 incPv 的结束大括号后。

> 注意：我们通过新函数 `getRawPostById` 用来获取文章原生的内容（编辑页面用），而不是用 `getPostById` 返回将 markdown 转换成 html 后的内容。

新建编辑文章页 views/edit.ejs，添加如下代码：

**views/edit.ejs**

```js
<%- include('header') %>

<div class="ui grid">
  <div class="four wide column">
    <a class="avatar"
       href="/posts?author=<%= user._id %>"
       data-title="<%= user.name %> | <%= ({m: '男', f: '女', x: '保密'})[user.gender] %>"
       data-content="<%= user.bio %>">
      <img class="avatar" src="/img/<%= user.avatar %>">
    </a>
  </div>

  <div class="eight wide column">
    <form class="ui form segment" method="post" action="/posts/<%= post._id %>/edit">
      <div class="field required">
        <label>标题</label>
        <input type="text" name="title" value="<%= post.title %>">
      </div>
      <div class="field required">
        <label>内容</label>
        <textarea name="content" rows="15"><%= post.content %></textarea>
      </div>
      <input type="submit" class="ui button" value="发布">
    </form>
  </div>
</div>

<%- include('footer') %>
```

修改 routes/posts.js，将：

**routes/posts.js**

```js
// GET /posts/:postId/edit 更新文章页
router.get('/:postId/edit', checkLogin, function (req, res, next) {
  res.send('更新文章页')
})

// POST /posts/:postId/edit 更新一篇文章
router.post('/:postId/edit', checkLogin, function (req, res, next) {
  res.send('更新文章')
})

// GET /posts/:postId/remove 删除一篇文章
router.get('/:postId/remove', checkLogin, function (req, res, next) {
  res.send('删除文章')
})
```

修改为：

```js
// GET /posts/:postId/edit 更新文章页
router.get('/:postId/edit', checkLogin, function (req, res, next) {
  const postId = req.params.postId
  const author = req.session.user._id

  PostModel.getRawPostById(postId)
    .then(function (post) {
      if (!post) {
        throw new Error('该文章不存在')
      }
      if (author.toString() !== post.author._id.toString()) {
        throw new Error('权限不足')
      }
      res.render('edit', {
        post: post
      })
    })
    .catch(next)
})

// POST /posts/:postId/edit 更新一篇文章
router.post('/:postId/edit', checkLogin, function (req, res, next) {
  const postId = req.params.postId
  const author = req.session.user._id
  const title = req.fields.title
  const content = req.fields.content

  // 校验参数
  try {
    if (!title.length) {
      throw new Error('请填写标题')
    }
    if (!content.length) {
      throw new Error('请填写内容')
    }
  } catch (e) {
    req.flash('error', e.message)
    return res.redirect('back')
  }

  PostModel.getRawPostById(postId)
    .then(function (post) {
      if (!post) {
        throw new Error('文章不存在')
      }
      if (post.author._id.toString() !== author.toString()) {
        throw new Error('没有权限')
      }
      PostModel.updatePostById(postId, { title: title, content: content })
        .then(function () {
          req.flash('success', '编辑文章成功')
          // 编辑成功后跳转到上一页
          res.redirect(`/posts/${postId}`)
        })
        .catch(next)
    })
})

// GET /posts/:postId/remove 删除一篇文章
router.get('/:postId/remove', checkLogin, function (req, res, next) {
  const postId = req.params.postId
  const author = req.session.user._id

  PostModel.getRawPostById(postId)
    .then(function (post) {
      if (!post) {
        throw new Error('文章不存在')
      }
      if (post.author._id.toString() !== author.toString()) {
        throw new Error('没有权限')
      }
      PostModel.delPostById(postId)
        .then(function () {
          req.flash('success', '删除文章成功')
          // 删除成功后跳转到主页
          res.redirect('/posts')
        })
        .catch(next)
    })
})
```

现在刷新主页，点击文章右下角的小三角，编辑文章和删除文章试试吧。

## 留言模型设计

我们只需要留言的作者 id、留言内容和关联的文章 id 这几个字段，修改 lib/mongo.js，添加如下代码：

**lib/mongo.js**

```js
exports.Comment = mongolass.model('Comment', {
  author: { type: Mongolass.Types.ObjectId, required: true },
  content: { type: 'string', required: true },
  postId: { type: Mongolass.Types.ObjectId, required: true }
})
exports.Comment.index({ postId: 1, _id: 1 }).exec()// 通过文章 id 获取该文章下所有留言，按留言创建时间升序
```

## 显示留言

在实现留言功能之前，我们先让文章页可以显示留言列表。首先创建留言的模板，新建 views/components/comments.ejs，添加如下代码：

**views/components/comments.ejs**

```ejs
<div class="ui grid">
  <div class="four wide column"></div>
  <div class="eight wide column">
    <div class="ui segment">
      <div class="ui minimal comments">
        <h3 class="ui dividing header">留言</h3>

        <% comments.forEach(function (comment) { %>
          <div class="comment">
            <span class="avatar">
              <img src="/img/<%= comment.author.avatar %>">
            </span>
            <div class="content">
              <a class="author" href="/posts?author=<%= comment.author._id %>"><%= comment.author.name %></a>
              <div class="metadata">
                <span class="date"><%= comment.created_at %></span>
              </div>
              <div class="text"><%- comment.content %></div>

              <% if (user && comment.author._id && user._id.toString() === comment.author._id.toString()) { %>
                <div class="actions">
                  <a class="reply" href="/comments/<%= comment._id %>/remove">删除</a>
                </div>
              <% } %>
            </div>
          </div>
        <% }) %>

        <% if (user) { %>
          <form class="ui reply form" method="post" action="/comments">
            <input name="postId" value="<%= post._id %>" hidden>
            <div class="field">
              <textarea name="content"></textarea>
            </div>
            <input type="submit" class="ui icon button" value="留言" />
          </form>
        <% } %>

      </div>
    </div>
  </div>
</div>
```

> 注意：我们在提交留言表单时带上了文章 id（postId），通过 hidden 隐藏。

在文章页引入留言的模板片段，修改 views/post.ejs 为：

**views/post.ejs**

```ejs
<%- include('header') %>

<%- include('components/post-content') %>
<%- include('components/comments') %>

<%- include('footer') %>
```

新建 models/comments.js，存放留言相关的数据库操作，添加如下代码：

**models/comments.js**

```js
const marked = require('marked')
const Comment = require('../lib/mongo').Comment

// 将 comment 的 content 从 markdown 转换成 html
Comment.plugin('contentToHtml', {
  afterFind: function (comments) {
    return comments.map(function (comment) {
      comment.content = marked(comment.content)
      return comment
    })
  }
})

module.exports = {
  // 创建一个留言
  create: function create (comment) {
    return Comment.create(comment).exec()
  },

  // 通过留言 id 获取一个留言
  getCommentById: function getCommentById (commentId) {
    return Comment.findOne({ _id: commentId }).exec()
  },

  // 通过留言 id 删除一个留言
  delCommentById: function delCommentById (commentId) {
    return Comment.deleteOne({ _id: commentId }).exec()
  },

  // 通过文章 id 删除该文章下所有留言
  delCommentsByPostId: function delCommentsByPostId (postId) {
    return Comment.deleteMany({ postId: postId }).exec()
  },

  // 通过文章 id 获取该文章下所有留言，按留言创建时间升序
  getComments: function getComments (postId) {
    return Comment
      .find({ postId: postId })
      .populate({ path: 'author', model: 'User' })
      .sort({ _id: 1 })
      .addCreatedAt()
      .contentToHtml()
      .exec()
  },

  // 通过文章 id 获取该文章下留言数
  getCommentsCount: function getCommentsCount (postId) {
    return Comment.count({ postId: postId }).exec()
  }
}
```

> 小提示：我们让留言也支持了 markdown。
> 注意：删除一篇文章成功后也要删除该文章下所有的评论，上面 delCommentsByPostId 就是用来做这件事的。


修改 models/posts.js，在：

**models/posts.js**

```js
const Post = require('../lib/mongo').Post
```

下添加如下代码：

```js
const CommentModel = require('./comments')

// 给 post 添加留言数 commentsCount
Post.plugin('addCommentsCount', {
  afterFind: function (posts) {
    return Promise.all(posts.map(function (post) {
      return CommentModel.getCommentsCount(post._id).then(function (commentsCount) {
        post.commentsCount = commentsCount
        return post
      })
    }))
  },
  afterFindOne: function (post) {
    if (post) {
      return CommentModel.getCommentsCount(post._id).then(function (count) {
        post.commentsCount = count
        return post
      })
    }
    return post
  }
})
```

在 PostModel 上注册了 `addCommentsCount` 用来给每篇文章添加留言数 `commentsCount`，在 `getPostById` 和 `getPosts` 方法里的：

```
.addCreatedAt()
```

下添加：

```
.addCommentsCount()
```

这样主页和文章页的文章就可以正常显示留言数了。

然后将 `delPostById` 修改为：

```js
// 通过用户 id 和文章 id 删除一篇文章
delPostById: function delPostById (postId, author) {
  return Post.deleteOne({ author: author, _id: postId })
    .exec()
    .then(function (res) {
      // 文章删除后，再删除该文章下的所有留言
      if (res.result.ok && res.result.n > 0) {
        return CommentModel.delCommentsByPostId(postId)
      }
    })
}
```

> 小提示：虽然目前看起来使用 Mongolass 自定义插件并不能节省代码，反而使代码变多了。Mongolass 插件真正的优势在于：在项目非常庞大时，可通过自定义的插件随意组合（及顺序）实现不同的输出，如上面的 `getPostById` 需要将取出 markdown 转换成 html，则使用 `.contentToHtml()`，否则像 `getRawPostById` 则不必使用。

修改 routes/posts.js，在：

**routes/posts.js**

```js
const PostModel = require('../models/posts')
```

下引入 CommentModel:

```js
const CommentModel = require('../models/comments')
```

在文章页传入留言列表，将：

```js
// GET /posts/:postId 单独一篇的文章页
router.get('/:postId', function (req, res, next) {
  ...
})
```

修改为：

```js
// GET /posts/:postId 单独一篇的文章页
router.get('/:postId', function (req, res, next) {
  const postId = req.params.postId

  Promise.all([
    PostModel.getPostById(postId), // 获取文章信息
    CommentModel.getComments(postId), // 获取该文章所有留言
    PostModel.incPv(postId)// pv 加 1
  ])
    .then(function (result) {
      const post = result[0]
      const comments = result[1]
      if (!post) {
        throw new Error('该文章不存在')
      }

      res.render('post', {
        post: post,
        comments: comments
      })
    })
    .catch(next)
})
```

现在刷新文章页试试吧，此时已经显示了留言的输入框。

## 发表与删除留言

现在我们来实现发表与删除留言的功能。将 routes/comments.js 修改如下：

```js
const express = require('express')
const router = express.Router()

const checkLogin = require('../middlewares/check').checkLogin
const CommentModel = require('../models/comments')

// POST /comments 创建一条留言
router.post('/', checkLogin, function (req, res, next) {
  const author = req.session.user._id
  const postId = req.fields.postId
  const content = req.fields.content

  // 校验参数
  try {
    if (!content.length) {
      throw new Error('请填写留言内容')
    }
  } catch (e) {
    req.flash('error', e.message)
    return res.redirect('back')
  }

  const comment = {
    author: author,
    postId: postId,
    content: content
  }

  CommentModel.create(comment)
    .then(function () {
      req.flash('success', '留言成功')
      // 留言成功后跳转到上一页
      res.redirect('back')
    })
    .catch(next)
})

// GET /comments/:commentId/remove 删除一条留言
router.get('/:commentId/remove', checkLogin, function (req, res, next) {
  const commentId = req.params.commentId
  const author = req.session.user._id

  CommentModel.getCommentById(commentId)
    .then(function (comment) {
      if (!comment) {
        throw new Error('留言不存在')
      }
      if (comment.author.toString() !== author.toString()) {
        throw new Error('没有权限删除留言')
      }
      CommentModel.delCommentById(commentId)
        .then(function () {
          req.flash('success', '删除留言成功')
          // 删除成功后跳转到上一页
          res.redirect('back')
        })
        .catch(next)
    })
})

module.exports = router
```

至此，我们完成了创建留言和删除留言的逻辑。刷新页面，尝试留言试试吧。留言成功后，将鼠标悬浮在留言上可以显示出 `删除` 的按钮，点击可以删除留言。

现在访问一个不存在的地址，如：`http://localhost:3000/haha` 页面会显示：

```
Cannot GET /haha
```
## 404 页面

我们来自定义 404 页面。修改 routes/index.js，在：

**routes/index.js**

```js
app.use('/comments', require('./comments'))
```

下添加如下代码：

```js
// 404 page
app.use(function (req, res) {
  if (!res.headersSent) {
    res.status(404).render('404')
  }
})
```

新建 views/404.ejs，添加如下代码：

**views/404.ejs**

```ejs
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title><%= blog.title %></title>
    <script type="text/javascript" src="http://www.qq.com/404/search_children.js" charset="utf-8"></script>
  </head>
  <body></body>
</html>
```

这里我们只为了演示 express 中处理 404 的情况，用了腾讯公益的 404 页面，刷新一下页面看下效果吧。

前面讲到 express 有一个内置的错误处理逻辑，如果程序出错，会直接将错误栈返回并显示到页面上。如访问：`localhost:3000/posts/xxx/edit` 没有权限编辑的文章页，将会直接在页面中显示错误栈，如下：

```js
Error: 权限不足
    at /Users/nswbmw/Desktop/myblog/routes/posts.js:95:15
    at <anonymous>
    at process._tickCallback (internal/process/next_tick.js:188:7)
```

现在我们修改代码，实现复用页面通知。修改 index.js，在 `app.listen` 上面添加如下代码：

**index.js**

```js
app.use(function (err, req, res, next) {
  console.error(err)
  req.flash('error', err.message)
  res.redirect('/posts')
})
```

这里我们实现了将错误信息用页面通知展示的功能，刷新页面将会跳转到主页并显示『权限不足』的红色通知。

现在我们来实现日志功能，日志分为正常请求的日志和错误请求的日志，我们希望实现这两种日志都打印到终端并写入文件。

## winston 和 express-winston

我们使用 [winston](https://www.npmjs.com/package/winston) 和 [express-winston](https://www.npmjs.com/package/express-winston) 记录日志。

新建 logs 目录存放日志文件，修改 index.js，在：

**index.js**

```js
const pkg = require('./package')
```

下引入所需模块：

```js
const winston = require('winston')
const expressWinston = require('express-winston')
```

将：

```
// 路由
routes(app)
```

修改为：

```js
// 正常请求的日志
app.use(expressWinston.logger({
  transports: [
    new (winston.transports.Console)({
      json: true,
      colorize: true
    }),
    new winston.transports.File({
      filename: 'logs/success.log'
    })
  ]
}))
// 路由
routes(app)
// 错误请求的日志
app.use(expressWinston.errorLogger({
  transports: [
    new winston.transports.Console({
      json: true,
      colorize: true
    }),
    new winston.transports.File({
      filename: 'logs/error.log'
    })
  ]
}))
```

刷新页面看一下终端输出及 logs 下的文件。
可以看出：winston 将正常请求的日志打印到终端并写入了 `logs/success.log`，将错误请求的日志打印到终端并写入了 `logs/error.log`。

> 注意：记录正常请求日志的中间件要放到 `routes(app)` 之前，记录错误请求日志的中间件要放到 `routes(app)` 之后。

## .gitignore

如果我们想把项目托管到 git 服务器上（如: [GitHub](https://github.com)），而不想把线上配置、本地调试的 logs 以及 node_modules 添加到 git 的版本控制中，这个时候就需要 .gitignore 文件了，git 会读取 .gitignore 并忽略这些文件。在 myblog 下新建 .gitignore 文件，添加如下配置：

**.gitignore**

```
config/*
!config/default.*
npm-debug.log
node_modules
coverage
```

需要注意的是，通过设置：

```
config/*
!config/default.*
```

这样只有 config/default.js 会加入 git 的版本控制，而 config 目录下的其他配置文件则会被忽略，因为把线上配置加入到 git 是一个不安全的行为，通常你需要本地或者线上环境手动创建 config/production.js，然后添加一些线上的配置（如：mongodb 配置）即可覆盖相应的 default 配置。

> 注意：后面讲到部署到 Heroku 时，因为无法登录到 Heroku 主机，所以可以把以下两行删掉，将 config/production.js 也加入 git 中。
> 
> ```
> config/*
> !config/default.*
> ```

然后在 public/img 目录下创建 .gitignore：

```
# Ignore everything in this directory
*
# Except this file
!.gitignore
```

这样 git 会忽略 public/img 目录下所有上传的头像，而不忽略 public/img 目录。同理，在 logs 目录下创建 .gitignore 忽略日志文件：

```
# Ignore everything in this directory
*
# Except this file
!.gitignore
```
# 测试

## mocha 和 supertest

[mocha](https://www.npmjs.com/package/mocha) 和 [supertest](https://www.npmjs.com/package/supertest) 是常用的测试组合，通常用来测试 restful 的 api 接口，这里我们也可以用来测试我们的博客应用。
在 myblog 下新建 test 文件夹存放测试文件，以注册为例讲解 mocha 和 supertest 的用法。首先安装所需模块：

```sh
npm i mocha supertest --save-dev
```

修改 package.json，将：

**package.json**

```json
"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1"
}
```

修改为：

```json
"scripts": {
  "test": "mocha test"
}
```

指定执行 test 目录的测试。修改 index.js，将：

**index.js**

```js
// 监听端口，启动程序
app.listen(config.port, function () {
  console.log(`${pkg.name} listening on port ${config.port}`)
})
```

修改为:

```js
if (module.parent) {
  // 被 require，则导出 app
  module.exports = app
} else {
  // 监听端口，启动程序
  app.listen(config.port, function () {
    console.log(`${pkg.name} listening on port ${config.port}`)
  })
}
```

这样做可以实现：直接启动 index.js 则会监听端口启动程序，如果 index.js 被 require 了，则导出 app，通常用于测试。

找一张图片用于测试上传头像，放到 test 目录下，如 avatar.png。新建 test/signup.js，添加如下测试代码：

**test/signup.js**

```js
const path = require('path')
const assert = require('assert')
const request = require('supertest')
const app = require('../index')
const User = require('../lib/mongo').User

const testName1 = 'testName1'
const testName2 = 'nswbmw'
describe('signup', function () {
  describe('POST /signup', function () {
    const agent = request.agent(app)// persist cookie when redirect
    beforeEach(function (done) {
      // 创建一个用户
      User.create({
        name: testName1,
        password: '123456',
        avatar: '',
        gender: 'x',
        bio: ''
      })
        .exec()
        .then(function () {
          done()
        })
        .catch(done)
    })

    afterEach(function (done) {
      // 删除测试用户
      User.deleteMany({ name: { $in: [testName1, testName2] } })
        .exec()
        .then(function () {
          done()
        })
        .catch(done)
    })

    after(function (done) {
      process.exit()
    })

    // 用户名错误的情况
    it('wrong name', function (done) {
      agent
        .post('/signup')
        .type('form')
        .field({ name: '' })
        .attach('avatar', path.join(__dirname, 'avatar.png'))
        .redirects()
        .end(function (err, res) {
          if (err) return done(err)
          assert(res.text.match(/名字请限制在 1-10 个字符/))
          done()
        })
    })

    // 性别错误的情况
    it('wrong gender', function (done) {
      agent
        .post('/signup')
        .type('form')
        .field({ name: testName2, gender: 'a' })
        .attach('avatar', path.join(__dirname, 'avatar.png'))
        .redirects()
        .end(function (err, res) {
          if (err) return done(err)
          assert(res.text.match(/性别只能是 m、f 或 x/))
          done()
        })
    })
    // 其余的参数测试自行补充
    // 用户名被占用的情况
    it('duplicate name', function (done) {
      agent
        .post('/signup')
        .type('form')
        .field({ name: testName1, gender: 'm', bio: 'noder', password: '123456', repassword: '123456' })
        .attach('avatar', path.join(__dirname, 'avatar.png'))
        .redirects()
        .end(function (err, res) {
          if (err) return done(err)
          assert(res.text.match(/用户名已被占用/))
          done()
        })
    })

    // 注册成功的情况
    it('success', function (done) {
      agent
        .post('/signup')
        .type('form')
        .field({ name: testName2, gender: 'm', bio: 'noder', password: '123456', repassword: '123456' })
        .attach('avatar', path.join(__dirname, 'avatar.png'))
        .redirects()
        .end(function (err, res) {
          if (err) return done(err)
          assert(res.text.match(/注册成功/))
          done()
        })
    })
  })
})
```

此时编辑器会报语法错误（如：describe 未定义等等），修改 .eslintrc.json 如下：

```json
{
  "extends": "standard",
  "globals": {
    "describe": true,
    "beforeEach": true,
    "afterEach": true,
    "after": true,
    "it": true
  }
}
```

这样，eslint 会忽略 globals 中变量未定义的警告。运行 `npm test` 看看效果吧，其余的测试请读者自行完成。

## 测试覆盖率

我们写测试肯定想覆盖所有的情况（包括各种出错的情况及正确时的情况），但光靠想需要写哪些测试是不行的，总也会有疏漏，最简单的办法就是可以直观的看出测试是否覆盖了所有的代码，这就是测试覆盖率，即被测试覆盖到的代码行数占总代码行数的比例。

> 注意：即使测试覆盖率达到 100% 也不能说明你的测试覆盖了所有的情况，只能说明基本覆盖了所有的情况。

[istanbul](https://www.npmjs.com/package/istanbul) 是一个常用的生成测试覆盖率的库，它会将测试的结果报告生成 html 页面，并放到项目根目录的 coverage 目录下。首先安装 istanbul:

```
npm i istanbul --save-dev
```

配置 istanbul 很简单，将 package.json 中：

**package.json**

```json
"scripts": {
  "test": "mocha test"
}
```

修改为：

```json
"scripts": {
  "test": "istanbul cover _mocha"
}
```

**注意**：Windows 下需要改成 `istanbul cover node_modules/mocha/bin/_mocha`。

即可将 istanbul 和 mocha 结合使用，运行 `npm test` 终端会打印：

![](/images/4.14.1.png)

打开 myblog/coverage/Icov-report/index.html，如下所示：

![](/images/4.14.2.png)

可以点进去查看某个代码文件具体的覆盖率，如下所示：

![](/images/4.14.3.png)

红色的行表示测试没有覆盖到，因为我们只写了 name 和 gender 的测试。

# 部署

## 申请 MLab

[MLab](https://mlab.com) (前身是 MongoLab) 是一个 mongodb 云数据库提供商，我们可以选择 500MB 空间的免费套餐用来测试。注册成功后，点击右上角的 `Create New` 创建一个数据库（如: myblog），成功后点击进入到该数据库详情页，注意页面中有一行黄色的警告：

```
A database user is required to connect to this database. To create one now, visit the 'Users' tab and click the 'Add database user' button.
```

每个数据库至少需要一个 user，所以我们点击 Users 下的 `Add database user` 创建一个用户。

> 注意：不要选中 `Make read-only`，因为我们有写数据库的操作。

最后分配给我们的类似下面的 mongodb url：

```
mongodb://<dbuser>:<dbpassword>@ds139327.mlab.com:39327/myblog
```

如我创建的用户名和密码都为 myblog 的用户，新建 config/production.js，添加如下代码：

**config/production.js**

```js
module.exports = {
  mongodb: 'mongodb://myblog:myblog@ds139327.mlab.com:39327/myblog'
}
```

停止程序，然后以 production 配置启动程序:

```sh
npm i cross-env --save-dev # 本地安装 cross-env
npm i cross-env -g # 全局安装 cross-env
cross-env NODE_ENV=production supervisor index
```

> 注意：cross-env 用来兼容 Windows 系统和 Linux/Mac 系统设置环境变量的差异。

## pm2

当我们的博客要部署到线上服务器时，不能单纯的靠 `node index` 或者 `supervisor index` 来启动了，因为我们断掉 SSH 连接后服务就终止了，这时我们就需要像 [pm2](https://www.npmjs.com/package/pm2) 或者 [forever](https://www.npmjs.com/package/forever) 这样的进程管理工具了。pm2 是 Node.js 下的生产环境进程管理工具，就是我们常说的进程守护工具，可以用来在生产环境中进行自动重启、日志记录、错误预警等等。以 pm2 为例，全局安装 pm2：

```sh
npm i pm2 -g
```

修改 package.json，添加 start 的命令：

**package.json**

```json
"scripts": {
  "test": "istanbul cover _mocha",
  "start": "cross-env NODE_ENV=production pm2 start index.js --name 'myblog'"
}
```

然后运行 `npm start` 通过 pm2 启动程序，如下图所示 ：

![](/images/4.15.1.png)

pm2 常用命令:

1. `pm2 start/stop`: 启动/停止程序
2. `pm2 reload/restart [id|name]`: 重启程序
3. `pm2 logs [id|name]`: 查看日志
4. `pm2 l/list`: 列出程序列表

更多命令请使用 `pm2 -h` 查看。

## 部署到 Heroku

[Heroku](https://www.heroku.com) 是一个支持多种编程语言的云服务平台，Heroku 也提供免费的基础套餐供开发者测试使用。现在，我们将论坛部署到 Heroku。

> 注意：新版 heroku 会有填写信用卡的步骤，如果没有信用卡请跳过本节。

首先，需要到 [https://toolbelt.heroku.com/](https://toolbelt.heroku.com/) 下载安装 Heroku 的命令行工具包 toolbelt。然后登录（如果没有账号，请注册）到 Heroku 的 Dashboard，点击右上角 New -> Create New App 创建一个应用。创建成功后运行：

```sh
heroku login
```

填写正确的 email 和 password 验证通过后，本地会产生一个 SSH public key。在部署到 Heroku 之前，我们需要对代码进行简单的修改。如下：

1.删掉 .gitignore 中：
```
config/*
!config/default.*
```
因为我们无法登录到 Heroku 主机创建 production 配置文件，所以这里将 production 配置也上传到 Heroku。

2.打开 index.js，将 `app.listen` 修改为：
```js
const port = process.env.PORT || config.port
app.listen(port, function () {
  console.log(`${pkg.name} listening on port ${port}`)
})
```
因为 Heroku 会动态分配端口（通过环境变量 PORT 指定），所以不能用配置文件里写死的端口。

3.修改 package.json，在 scripts 添加：

```json
"heroku": "NODE_ENV=production node index"
```

在根目录下新建 Procfile 文件，添加如下内容：
```
web: npm run heroku
```
Procfile 文件告诉 Heroku 该使用什么命令启动一个 web 服务。更多信息见：[https://devcenter.heroku.com/articles/getting-started-with-nodejs](https://devcenter.heroku.com/articles/getting-started-with-nodejs)。

然后输入以下命令：

```sh
git init
heroku git:remote -a 你的应用名称
git add .
git commit -am "init"
git push heroku master
```

稍后，我们的论坛就部署成功了。使用：

```sh
heroku open
```

打开应用主页。如果出现 "Application error"，使用：

```sh
heroku logs
```
查看日志，调试完后 commit 并 push 到 heroku重新部署。

## 部署到 UCloud

### 创建主机

1. 注册 UCloud
2. 点击左侧的 `云主机`，然后点击 `创建主机`，统统选择最低配置
3. 右侧付费方式选择 `按时`（每小时），点击 `立即购买`
4. 在支付确认页面，点击 `确认支付`

购买成功后回到主机管理列表，如下所示：

![](/images/4.15.2.png)

> 注意：下面所有的 ip 都替换为你自己的外网 ip。

### 环境搭建与部署

修改 config/production.js，将 port 修改为 80 端口：

**config/production.js**

```js
module.exports = {
  port: 80,
  mongodb: 'mongodb://myblog:myblog@ds139327.mlab.com:39327/myblog'
}
```

登录主机，用刚才设置的密码：

```sh
ssh root@106.75.47.229
```

因为是 CentOS 系统，所以我选择使用 yum 安装，而不是下载源码编译安装：

```sh
yum install git #安装git
yum install nodejs #安装 Node.js
yum install npm #安装 npm

npm i npm -g #升级 npm
npm i pm2 -g #安装 pm2
npm i n -g #安装 n
n v8.9.1 #安装 v8.9.1 版本的 Node.js
n use 8.9.1 #使用 v8.9.1 版本的 Node.js
node -v
```
> 注意：如果 `node -v` 显示的不是 8.9.1，则断开 ssh，重新登录主机再试试。

此时应该在 `/root` 目录下，运行以下命令：
```sh
git clone https://github.com/nswbmw/N-blog.git myblog #或在本机 myblog 目录下运行 rsync -av --exclude="node_modules" ./ root@106.75.47.229:/root/myblog
cd myblog
npm i
npm start
pm2 logs
```
> 注意：如果不想用 git 的形式将代码拉到云主机上，可以用 rsync 将本地的代码同步到你的 UCloud 主机上，如上所示。

最后，访问你的公网 ip 地址试试吧，如下所示：

![](/images/4.15.3.png)

> 小提示：因为我们选择的按时付费套餐，测试完成后，可在主机管理页面选择关闭主机，节约费用。

## 部署到阿里云

### 创建主机

1. 注册/登录
2. 充值 100（因为我们选择『按量付费』，阿里云要求最低账户余额 >= 100）
3. 进入『云服务器 ECS』
4. 点击『创建实例』

进入创建实例页面，按下图选择配置：

![](/images/4.15.4.png)

需要注意几点：

1. 计费方式：按量付费
2. 公网 ip 地址：分配
3. 安全组：选中开启 80 端口
4. 镜像：Ubuntu 16.04 64位

点击『开通进入下一页』，选中：

![](/images/4.15.5.png)

> 注意：这里我们只是演示，所以自动释放时间只设置了几个小时

点击『去开通』创建成功，然后点击提示中的『管理控制台』进入 ECS 管理页，刚才创建的机器需要等待几分钟才会初始化成功。成功后如下所示：

![](/images/4.15.6.png)

### 环境搭建

复制创建的机器的公网 ip 地址，运行：

```sh
ssh root@39.106.134.66
```

输入刚才设置的密码登录远程主机。

#### 安装 Node.js

我们下载编译好的 Node.js 压缩包，解压然后使用软连接。

```sh
wget https://nodejs.org/dist/v8.9.1/node-v8.9.1-linux-x64.tar.xz
tar -xvf node-v8.9.1-linux-x64.tar.xz
mv node-v8.9.1-linux-x64 nodejs
ln -s ~/nodejs/bin/* /usr/local/bin/
node -v
npm -v
```

#### 安装 MongoDB

```sh
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-ubuntu1604-3.4.10.tgz
tar -xvf mongodb-linux-x86_64-ubuntu1604-3.4.10.tgz
mv mongodb-linux-x86_64-ubuntu1604-3.4.10 mongodb
ln -s ~/mongodb/bin/* /usr/local/bin/
mongod --version
mongo --version
mkdir mongodb/data
mongod --dbpath=mongodb/data &
```

#### 安装 Git

```sh
apt-get update
apt-get install git
git clone https://github.com/nswbmw/N-blog.git #或者你的 GitHub blog 地址
cd N-blog
npm i
vim config/default.js #修改端口 3000->80
node index
```

此时，浏览器中访问你的机器的公网 ip 试试吧。

#### 使用 PM2 启动

```sh
npm i pm2 -g
ln -s ~/nodejs/bin/* /usr/local/bin/
pm2 start index.js --name="myblog"
```

这里我们使用 pm2 启动博客，所以关掉终端后博客仍然在运行。

