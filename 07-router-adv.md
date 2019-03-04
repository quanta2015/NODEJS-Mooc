title: 07. Express 路由
speaker: LI YANG
url: 
transition: slide3
files: /js/demo.js,/css/demo.css
theme: dark
usemathjax: yes

--
# Express 路由
## Express router


### 路由（URL映射）
Express利用HTTP动作提供了有意义并富有表现力的URL映射API，所有的请求都表示为路径参数的模式。
```
\x1\x2\...\xN\:parm1\:parm2\...\:parmN
```

传统的url一般为一个具体的页面，在Expree模型下为一个url路径
```js
//一个注册页面的地址
action = "/login.html"

action = "/login"
```

### express路由规则
express 封装了多种 `http` 请求方式，我们主要只使用 `get` 和 `post` ,可以使用 `app.all` 获取所以请求方式，回调函数有两个参数分别是 `req`(请求信息) 和 `res`(响应信息)。

* req.query： 处理 `get` 请求
* req.params： 处理 `/:xxx` 形式的 `get` 请求
* req.body: 处理 `post` 请求
* req.param()： 可以处理 `get` 和 `post` 请求，但查找优先级由高到低为 `req.params → req.body → req.query`

### req.query 路由参数预处理 
可以通过req的query对象，获取表达post的参数
```js
URL http://localhost:3000/login?username=tom&password=12345

app.get('/login',function(req,res){
    var username = req.query.username;
    var password = req.query.password;
})
```

### req.params 路由参数预处理 
路由参数预处理通过隐式的数据处理，可以大幅提高应用代码的可读性和请求URL的验证。可以从路由获取通用数据，如通过`/user/:id`加载用户` id `。
```js
app.get('/user/:userId', function(req, res, next){
    var id = req.params.userId;
    ...
});
```

### req.post 路由参数预处理 
可以通过req的body对象，获取表达post的参数
```js
app.post('/login', function(req, res, next){
    var username=req.body.name;
    var password=req.body.password;
});
```


### 中间件
一个应用可以定义多个路由，可以控制一个用户请求从一个路由转向下一个路由，这个时候中间件就是一种封装在程序中处理 HTTP 请求的功能，它有3个参数：`req请求对象` 、`res响应对象` 和 `next函数` ；中间件和路由处理器是按它们的**连入顺序**调用的。

请求在管道中如何**终止**: 如果中间件不调用`next()`,同时也不将响应输出到客户端，此时请求就在那个中间件终止了。


下面是一个复杂的范例
```js
var app = require('express')();

app.use(function(req, res, next) {
  console.log('\n\nALLWAYS');
  next();
});

app.get('/a', function(req, res) {
  console.log('/a: 路由终止 ');
  res.send('a');
});
app.get('/a', function(req, res) {
  console.log('/a: 永远不会调用 ');
});
app.get('/b', function(req, res, next) {
  console.log('/b: 路由未终止 ');
  next();
});
app.use(function(req, res, next) {
  console.log('SOMETIMES');
  next();
});
app.get('/b', function(req, res, next) {
  console.log('/b (part 2): 抛出错误 ');
  throw new Error('b 失败 ');
});

app.use('/b', function(err, req, res, next) {
  console.log('/b 检测到错误并传递 ');
  next(err);
});
app.get('/c', function(err, req) {
  console.log('/c: 抛出错误 ');
  throw new Error('c 失败 ');
});
app.use('/c', function(err, req, res, next) {
  console.log('/c: 检测到错误但不传递 ');
  next();
});

app.use(function(err, req, res, next) {
  console.log(' 检测到未处理的错误 : ' + err.message);
  res.send('500 - 服务器错误 ');
});

app.use(function(req, res) {
  console.log(' 未处理的路由 ');
  res.send('404 - 未找到 ');
});

app.listen(3000, function() {
  console.log(' 监听端口 3000');
});
```

* 中间件一般不直接对客户端进行响应，而是对请求进行一些预处理，再传递下去；
* 中间件一般会在路由处理之前执行；
* 路由是一种特殊的中间件（终点）

```js
// 检查用户是否登录中间件，所有需要登录权限的页面都使用此中间件
function checkLogin (req, res, next) {
  if (req.session.user) {
    next();//检验到用户已登入，转移权限阁下一个路由
  } else {
    res.redirect('/');//否则，页面重定位，不执行下面路由
  }
} 
```


### 全局参数Filter的中间件
对于站点信息之类的常量参数，在每个页面中可能都要显示在页面上，为了避免每次都在render传入这些数据信息。需要统一管理中间件来处理它，将其称为`filter`。将这个`filter`定义在所有的请求之前，由于中间件的处理顺序从上往下，所以每个请求在页面上都会传入这些参数。
```js
app.use(function (req, res, next) {
    res.locals.title = config.site.title;
    res.locals.description = config.site.description;
    res.locals.pagesize = config.site.pagesize;
    res.locals.session = req.session;
    next();
});
```

### express 路由处理范例
Webstorm工程中app.js的路由处理逻辑如下：

1. 将routes文件夹下面的文件定义为路由对象；  
2. 通过use方法将请求路径映射到路由对象上；  

```js
//定义路由对象
var routes = require('./routes/index');
var users = require('./routes/users');

//映射路径处理对象
app.use('/', routes);
app.use('/users', users);
```