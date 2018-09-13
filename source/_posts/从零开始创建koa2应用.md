---
title: 从零开始创建koa2应用
date: 2018-04-28 12:37:23
tags:
- koa
---

## 创建一个 koa2 工程

```bash
mkdir hello-koa2 && cd hello-koa2

npm init

yarn add koa
```

通过以上命令行创建一个文件夹 hello-koa2，并进入该文件夹，然后我们初始化一个项目并安装 koa。
接下来用编辑器（如vscode）打开项目，创建文件app.js，输入以下代码：(当然，最好使用 git 方式管理项目)
<!-- more -->

```javascript
const Koa = require('koa');
const app = new Koa();

// 对于任何请求，app 将调用该异步函数处理请求：
app.use(async(ctx, next) => {
    await next();
    ctx.response.type = 'text/html';
    ctx.response.body = '<h1>Hello, koa2!</h1>';
});

app.listen(3000);
```

此时通过命令 `node app.js` 我们就可以项目可以运行了，浏览器打开[http://localhost:3000/](http://localhost:3000/)就可以看到内容了。
我们可以把该命令放到 package.json 的 scripts 下，后续可以一步一步构建一个完善的项目框架。

## 创建 middleware

什么我们已经使用 app.use 方法处理请求了，我们在该函数中使用了`await next()`方法，我们知道 koa2 是洋葱圈模型。当一个请求过来的时候，会依次经过各个中间件进行处理，中间件跳转的信号是await(yield) next，当到某个中间件后，该中间件处理完不执行await next的时候，然后就会逆序执行前面那些中间件剩下的逻辑，组成一个处理链。
根据这个逻辑，我们在处理过程中自己添加 middleware，做一些我们想做的事情。以下是摘自网上的一个示例：

```javascript
app.use(async (ctx, next) => {
    console.log(`${ctx.request.method} ${ctx.request.url}`); // 打印URL
    await next(); // 调用下一个middleware
});

app.use(async (ctx, next) => {
    const start = new Date().getTime(); // 当前时间
    await next(); // 调用下一个middleware
    const ms = new Date().getTime() - start; // 耗费时间
    console.log(`Time: ${ms}ms`); // 打印耗费时间
});

app.use(async (ctx, next) => {
    await next();
    ctx.response.type = 'text/html';
    ctx.response.body = '<h1>Hello, koa2!</h1>';
});
```

## 处理 URL

在以上的代码中我们对所有 URL 返回的是一样的内容，实际情况下不可能如此。正常情况下，我们应该对不同的 URL 调用不同的处理函数，返回不同的结果。例如：

```javascript
app.use(async (ctx, next) => {
    if (ctx.request.path === '/') {
        ctx.response.body = 'HOME page';
    } else {
        await next();
    }
});

app.use(async (ctx, next) => {
    if (ctx.request.path === '/user') {
        ctx.response.body = 'USER page';
    } else {
        await next();
    }
});
```

我们可以使用 koa-router 这个 middleware，让它负责处理 URL 映射。安装 koa-router

```bash
yarn add koa-router
```

然后把 app.js 改造如下：

```javascript
const Koa = require('koa');
const app = new Koa();
const Router = require('koa-router');
const router = new Router();

app.use(async(ctx, next) => {
    console.log(`Process ${ctx.request.method} ${ctx.request.url}...`);
    await next();
});

router.get('/hello/:name', async(ctx, next) => {
    let name = ctx.params.name;
    ctx.response.body = `<h1>Hello, ${name}!</h1>`;
});

router.get('/', async(ctx, next) => {
    ctx.response.body = '<h1>Home page</h1>';
});

app.use(router.routes());

app.listen(3000);
```

然后我们可以看到请求根页面`/`和`/hello/:name`显示的内容就不一样了。

## 处理 post 请求

上面我们用到的都是 get 请求，接下来我们试一下 post 请求。
我们使用`koa-bodyparser`组件来解析我们获取的 request 对象。 安装`yarn add koa-bodyparser`完之后在 app.js 中引入 koa-bodyparser。由于middleware的顺序很重要，这个koa-bodyparser必须在router之前被注册到app对象上。

```javascript
const bodyParser = require('koa-bodyparser');

// 在合适的位置加上
app.use(bodyParser());
```

接下来我们就可以测试 post 请求了。同理，patch、delete、put等请求也可以由 router 处理。

```javascript
router.post('/login', async (ctx, next) => {
    let
        name = ctx.request.body.name || '',
        password = ctx.request.body.password || '';
    console.log(`login with name: ${name}, password: ${password}`);
    if (name === 'hello' && password === '123456') {
        ctx.response.body = `<h1>Welcome, ${name}!</h1>`;
    } else {
        ctx.response.body = `<h1>Login failed!</h1>
        <p><a href="/">Try again</a></p>`;
    }
});
```

## 调整项目结构

随着 URL 请求数量的越来越多，app.js 就会越来越长，这样不利于项目的管理和拓展。
如果能把 URL 处理函数集中到特定 js 文件，然后让 app.js 自动导入所有处理 URL 的函数。这样，代码一分离，逻辑就显得清楚了。我们可以在根目录下创建 controllers 文件夹，按照分类把各 url 处理函数放在该目录下的不同 js 文件。
例如我们可以在 controllers 目录下创建 hello.js 文件，内容如下：

```javascript
let fn_hello = async(ctx, next) => {
    let name = ctx.params.name;
    ctx.response.body = `<h1>Hello, ${name}!</h1>`;
};

module.exports = {
    'GET /hello/:name': fn_hello
};
```

app.js 修改：

```javascript
const files = fs.readdirSync(__dirname + '/controllers');

// 过滤出.js文件:
const js_files = files.filter((file)=>{
    return file.endsWith('.js');
});

// 处理每个js文件:
for (let f of js_files) {
    console.log(`process controller: ${f}...`);
    // 导入js文件:
    let mapping = require(__dirname + '/controllers/' + f);
    for (var url in mapping) {
        if (url.startsWith('GET ')) {
            // 如果url类似"GET xxx":
            let path = url.substring(4);
            router.get(path, mapping[url]);
            console.log(`register URL mapping: GET ${path}`);
        } else if (url.startsWith('POST ')) {
            // 如果url类似"POST xxx":
            let path = url.substring(5);
            router.post(path, mapping[url]);
            console.log(`register URL mapping: POST ${path}`);
        } else {
            // 无效的URL:
            console.log(`invalid URL: ${url}`);
        }
    }
}

```

## 使用模板引擎

模板引擎有很多，比如 pug, ejs, nunjucks...，这里以 nunjucks 为例，配合 koa-views 中间件。
我们需要创建一个 views 文件夹，用来存放我们的目标文件。

```javascript
app.use(views(path.join(__dirname, './views'), {
    map: {
        html: 'nunjucks'
    },
    extension: 'html'
}));

// 简单用法如下
app.use(async(ctx, next) => {
    if (ctx.request.path === '/hello') {
        let name = 'Koa2'
        await ctx.render('hello', {
            name,
        })
    } else {
        await next();
    }
});
```

## 使用其他中间件

创建 public 目录，用来存放公共资源，如 css, js, 图片, 字体等资源，使用`koa-static`

```javascript
const static = require('koa-static');

app.use(static(path.join(__dirname, './public')));
```