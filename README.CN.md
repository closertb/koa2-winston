# @doddle/koa-winston

<font color="red" size="5"></font>

<font color="red" size="5">两个改动点</font>

 - 在koa2-winston 基础上，新增了自定义日志信息的功能， [see here](#新增例子)；
 - 将日志日期输出属性`start_at` 打印格式从时间戳换成了 format `YYYY-MM-DD HH:mm:ss`

[![Travis](https://img.shields.io/travis/yidinghan/koa2-winston.svg?style=flat-square)](https://www.npmjs.com/package/@doddle/koa-winston)
[![npm](https://img.shields.io/npm/l/koa2-winston.svg?style=flat-square)](https://www.npmjs.com/package/@doddle/koa-winston)
[![npm](https://img.shields.io/npm/v/koa2-winston.svg?style=flat-square)](https://www.npmjs.com/package/@doddle/koa-winston)
[![npm](https://img.shields.io/npm/dm/koa2-winston.svg?style=flat-square)](https://www.npmjs.com/package/@doddle/koa-winston)
[![David](https://img.shields.io/david/yidinghan/koa2-winston.svg?style=flat-square)](https://www.npmjs.com/package/@doddle/koa-winston)
[![David](https://img.shields.io/david/dev/yidinghan/koa2-winston.svg?style=flat-square)](https://www.npmjs.com/package/@doddle/koa-winston)
[![node](https://img.shields.io/node/v/koa2-winston.svg?style=flat-square)](https://www.npmjs.com/package/@doddle/koa-winston)

koa2 版本的 winston logger, 和 [express-winston](https://github.com/bithavoc/express-winston) 类似

在3行内将logger添加到koa2服务器

<!-- TOC -->

- [@doddle/koa-winston](#@doddle/koa-winston)
- [用法](#用法)
  - [安装](#安装)
  - [快速开始](#快速开始)
  - [配置](#配置)
  - [例子](#例子)
    - [不记录任何请求内容](#不记录任何请求内容)
    - [不记录任何响应内容](#不记录任何响应内容)
    - [不记录 UA](#不记录-ua)
    - [额外记录一个响应的字段](#额外记录一个响应的字段)
- [JSDoc](#jsdoc)
  - [keysRecorder](#keysrecorder)
  - [logger](#logger)

<!-- /TOC -->

# 用法

## 安装

```shell
npm i --save @doddle/koa-winston
```

## 快速开始

```js
const { logger } = require('@doddle/koa-winston');
app.use(logger());
```

访问的日志将会如下出现
```json
{
  "req": {
    "headers": {
      "host": "localhost:3000",
      "connection": "keep-alive",
      "upgrade-insecure-requests": "1",
      "user-agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36",
      "accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8",
      "dnt": "1",
      "accept-encoding": "gzip, deflate, sdch, br",
      "accept-language": "zh-CN,zh;q=0.8,en;q=0.6,zh-TW;q=0.4,de;q=0.2,ja;q=0.2,it;q=0.2"
    },
    "url": "/hello",
    "method": "GET",
    "href": "http://localhost:3000/hello",
    "query": {}
  },
  "started_at": 1494554053492,
  "res": {
    "headers": {
      "content-type": "application/json; charset=utf-8",
      "content-length": "16"
    },
    "status": 200
  },
  "duration": 8,
  "level": "info",
  "message": "HTTP GET /hello"
}
```

## 配置

每一个变量都有一个默认值，你可以通过配置不同的变量自定义你的日志记录器

```js
app.use(logger({
  transports: new winston.transports.Console({ json: true, stringify: true }),
  level: 'info',
  reqKeys: ['headers','url','method', 'httpVersion','href','query','length'],
  reqSelect: [],
  reqUnselect: ['headers.cookie'],
  resKeys: ['headers','status'],
  resSelect: [],
  resUnselect: [],
}));
```

更多配置解析，可以在[logger](#logger)中查看

## 例子

### 不记录任何请求内容

```js
app.use(logger({
  reqKeys: []
}));
```

`req` 对象将会为空

```json
{
  "req": {
  },
  "started_at": 1494486039864,
  "res": {
    "headers": {
      "content-type": "text/plain; charset=utf-8",
      "content-length": "8"
    },
    "status": 200
  },
  "duration": 26,
  "level": "info",
  "message": "HTTP GET /"
}
```

### 不记录任何响应内容
```js
app.use(logger({
  resKeys: []
}));
```

`res` 对象将会为空

```json
{
  "req": {
    "headers": {
      "host": "127.0.0.1:59534",
      "accept-encoding": "gzip, deflate",
      "user-agent": "node-superagent/3.5.2",
      "connection": "close"
    },
    "url": "/",
    "method": "GET",
    "href": "http://127.0.0.1:59534/",
    "query": {}
  },
  "started_at": 1494486039864,
  "res": {
  },
  "duration": 26,
  "level": "info",
  "message": "HTTP GET /"
}
```

### 不记录 UA

```js
app.use(logger({
  reqUnselect: ['headers.cookies', 'headers.user-agent']
}));
```

请求的 UA 将会被忽略

```json
{
  "req": {
    "headers": {
      "host": "127.0.0.1:59534",
      "accept-encoding": "gzip, deflate",
      "connection": "close"
    },
    "url": "/",
    "method": "GET",
    "href": "http://127.0.0.1:59534/",
    "query": {}
  },
  "started_at": 1494486039864,
  "res": {
    "headers": {
      "content-type": "text/plain; charset=utf-8",
      "content-length": "8"
    },
    "status": 200
  },
  "duration": 26,
  "level": "info",
  "message": "HTTP GET /"
}
```

### 额外记录一个响应的字段

```js
app.use(logger({
  resSelect: ['body.success']
}));
```

`body` 里面的 `success` 字段将会被记录

```json
{
  "req": {
    "headers": {
      "host": "127.0.0.1:59534",
      "accept-encoding": "gzip, deflate",
      "connection": "close"
    },
    "url": "/",
    "method": "GET",
    "href": "http://127.0.0.1:59534/",
    "query": {}
  },
  "started_at": 1494486039864,
  "res": {
    "headers": {
      "content-type": "text/plain; charset=utf-8",
      "content-length": "8"
    },
    "status": 200,
    "body": {
      // 会记录下任何服务器响应的值
      "success": false
    }
  },
  "duration": 26,
  "level": "info",
  "message": "HTTP GET /"
}
```

## 新增例子
### 添加自定义的日志信息

```js
const { logger, addExtendProperty } = require('');

addExtendProperty({
  author_name: { type: 'string' }
})

app.use(
  logger({
    addExtendInfo: (ctx) => ({
      author_name: ctx.user && ctx.user.name,
  }),
  })
);
```

The req object will be empty

```json
{
  "req": {},
  "started_at": "YYYY-MM-DD HH:mm:ss",
  "author_name": "doddle",
  "res": {
    "header": {
      "content-type": "text/plain; charset=utf-8",
      "content-length": "8"
    },
    "status": 200
  },
  "duration": 26,
  "level": "info",
  "message": "HTTP GET /"
}
```

# JSDoc

<!-- Generated by documentation.js. Update this documentation by updating the source code. -->

## keysRecorder

keysRecorder
use ldoash pick, get and set to collect data from given target object

**Parameters**

-   `payload` **[Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)** input arguments (optional, default `{}`)
    -   `payload.defaults` **[Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)&lt;[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)>?** default keys will be collected
    -   `payload.selects` **[Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)&lt;[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)>?** keys will be collected as
        additional part
    -   `payload.unselects` **[Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)&lt;[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)>?** keys that will be ignored at last

**Examples**

```javascript
// without payload
const recorder = keysRecorder();
recorder() // {}
recorder({ foo: 1, bar: 2, foobar: { a: 3, b: 4 } }) // {}

// with defaults
const recorder = keysRecorder({ defaults: ['foo'] });
recorder() // {}
recorder({ foo: 1, bar: 2, foobar: { a: 3, b: 4 } }) // { foo: 1 }

// with defaults and selects
const recorder = keysRecorder({ defaults: ['foo'], selects: ['foobar'] });
recorder() // {}
recorder({
  foo: 1,
  bar: 2,
  foobar: { a: 3, b: 4 }
}) // { foo: 1, foobar: { a: 3, b: 4 } }

// with defaults and unselects
const recorder = keysRecorder({ defaults: ['foobar'], unselects: ['foobar.a'] });
recorder() // {}
recorder({
  foo: 1,
  bar: 2,
  foobar: { a: 3, b: 4 }
}) // { foobar: { a: 3 } }

// with defaults and selects and unselects
const recorder = keysRecorder({
  defaults: ['foo'],
  selects: ['foobar'],
  unselects: ['foobar.b'],
});
recorder() // {}
recorder({
  foo: 1,
  bar: 2,
  foobar: { a: 3, b: 4 }
}) // { foo: 1, foobar: { a: 3 } }
```

Returns **[function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function)** closure function, setting by given payload

## logger

logger middleware for koa2 use winston

**Parameters**

-   `payload` **[object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)** input arguments (optional, default `{}`)
    -   `payload.transports` **[Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)&lt;[object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)>** winston transports instance (optional, default `winston.transports.Console`)
    -   `payload.level` **[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** default log level of logger (optional, default `'info'`)
    -   `payload.reqKeys` **[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** default request fields to be logged (optional, default `['headers','url','method',
        'httpVersion','href','query','length']`)
    -   `payload.reqSelect` **[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** additional request fields to be logged (optional, default `[]`)
    -   `payload.reqUnselect` **[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** request field
                         will be removed from the log (optional, default `['headers.cookie']`)
    -   `payload.resKeys` **[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** default response fields to be logged (optional, default `['headers','status']`)
    -   `payload.resSelect` **[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** additional response fields to be logged (optional, default `[]`)
    -   `payload.resUnselect` **[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** response field will be removed from the log (optional, default `[]`)

**Examples**

```javascript
const { logger } = require('@doddle/koa-winston');
app.use(logger());
// trrific logger look like down here
// {
//   "req": {
//     "headers": {
//       "host": "127.0.0.1:59534",
//       "accept-encoding": "gzip, deflate",
//       "user-agent": "node-superagent/3.5.2",
//       "connection": "close"
//     },
//     "url": "/",
//     "method": "GET",
//     "href": "http://127.0.0.1:59534/",
//     "query": {}
//   },
//   "started_at": 1494486039864,
//   "res": {
//     "headers": {
//       "content-type": "text/plain; charset=utf-8",
//       "content-length": "8"
//     },
//     "status": 200
//   },
//   "duration": 26,
//   "level": "info",
//   "message": "HTTP GET /"
// }
```

Returns **[function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function)** logger middleware
