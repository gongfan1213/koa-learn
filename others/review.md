对于全局安装的express项目，链接到当前目录安装下的express，使得当前项目仍然可以通过require函数使用全局安装的依赖包

yarn

更好的语义化

速度更快

并行下载

可靠的，简洁输出，

注意
回调函数中输入代码，其核心在于Node.js的最大特点，即由于……
更加详细的信息请阅读本书第5章的相关内容。

对这段代码调用之前定义好的模型中的find方法进行查询，当查询结束后，使用回调函数输出当前查询的内容。

至此，整个文件代码如下：
```javascript
var mongoose = require('mongoose');
mongoose.connect('mongodb://106.53.115.12:27017/ming');// 连接Myblog数据库
// 添加数据库监听事件
mongoose.connection.on('connected', () => {
    console.log("mongodb数据库连接成功")
});
mongoose.connection.on('error', (error) => {
    console.log("mongodb数据库连接失败", error)
});
// 创建数据库相应模型
const userSchema = new mongoose.Schema({
    name: { type: String }
});
let userModel = mongoose.model('ming', userSchema);
// 保存数据库相应数据
let user = new userModel({
    "name": "ming"
})
user.save((err) => {
    console.log(err)
    // 查询数据库相应数据
    userModel.find({}, (err, docs) => {
        if (err) {
            console.log(err);
            return;
        }
        console.log(docs);
    })
})
```

执行该文件，如果输出：
```
mongodb数据库连接成功
null
[ { _id: 5fc d1b5467b2ba08a8cdbf87, name:'ming', __v: 0 } ]
```
则表示数据库连接成功。

从下一节到本章结束，将会是一个完整的基于MVC的Node.js和MongoDB的查询例子 
Controller层不仅承担调用Service层的作用，还可以根据URL调用相应的Service层

### 2.3.4 编写 Module 层
作为三层架构，项目一般有 Model 层、View 层、Controller 层。请求先访问 Controller 层，Controller 层返回 View 层页面给用户，用户操作后将数据发送给 Controller 层，Controller 层传递给 Model 层处理，Model 层与数据库交互，处理完数据返回给 View 层，再返回给 Controller 层呈现给用户，形成闭环。

创建 `module` 文件夹，新建 `user.js` 文件，代码如下：
```javascript
let mongoose = require("../connect.js");
// 创建数据库相应模型
const userSchema = new mongoose.Schema({
    name: { type: String }
});
let userModel = mongoose.model('ming', userSchema);
module.exports = userModel;
```
### 2.3.5 编写 Service 层
Service 层是 Controller 层的扩展，用于数据逻辑处理，如参数校验、库存和订单操作等，是与数据库关系最密切且逻辑性最高的部分。作用包括封装通用业务逻辑、与数据层交互、处理其他请求（如远程服务获取数据）。

创建 `Service` 文件夹，新建 `Service.js` 文件，代码如下：
```javascript
const userModel = require("../module/user.js");
let user = new userModel({
    "name": "ming"
});
// 异步调用 mongoose 相关存储，查询 API
var userFunction = new Promise(function () {
    user.save((err) => {
        console.log(err)
        // 查询数据库相应数据
        userModel.find((err, docs) => {
            if (err) {
                console.log(err);
                return;
            }
            resolve(docs)
        })
    });
});
module.exports = userFunction;
```
改造后的 Service 层代码：
```javascript
const userModel = require("../module/user.js");
let myDate = new Date();
let user = new userModel({
    "name": "ming"
});
// 异步调用 mongoose 相关存储，查询 API
var userFunction = new Promise(function (resolve, reject) {
    if (myDate.getDate() == 8) {
        user.save((err) => {
            console.log(err)
            // 查询数据库相应数据
            userModel.find((err, docs) => {
                if (err) {
                    console.log(err);
                    return;
                }
                resolve(docs)
            })
        });
    }
});
userFunction.then(function (successMessage) {
    //successMessage 的值是调用 resolve(...)方法传入的值
    //successMessage 参数不一定是字符串类型，这里只是举个例子
    console.log(successMessage)
});
module.exports = userFunction;
```
### 2.3.6 编写 Controller 层
Controller 层用于将请求分发到相应的 Service 层，完成中间件工作（如前后端交互验证，像 JWT 等）。作用包括参数校验、调用 Service 层接口实现业务逻辑、转换业务/对象、组装返回对象、异常处理。

创建 `Controller` 文件夹，新建 `Controller.js` 文件，代码如下：
```javascript
const service = require("../service/service.js");
// 引入 http 模块
var http = require("http");
// 创建 http 服务器
var server = http.createServer(function (request, response) {
    // 设置响应的头部
    response.writeHead(200, {
        "content-Type": "text/plain"
    });
    /* 设置响应的数据 */
    service.then((res) => {
        response.write(res.toString());
        response.end();
    });
});
// 设置服务器端口
server.listen(8000, function () {
    console.log("Creat server on http://127.0.0.1:8000/");
});
```
修改后的代码增加路由分发功能：
```javascript
const service = require("../service/service.js");
// 引入 http 模块
var http = require("http");
var url = require("http");
// 创建 http 服务器
var server = http.createServer(function (request, response) {
    if (request.url == "/index.html") {
        /* 设置响应的头部 */
        response.writeHead(200, {
            "content-Type": "text/plain"
        });
        /* 设置响应的数据 */
        service.then((res) => {
            response.write(res.toString());
            response.end();
        })
    }
});
// 设置服务器端口
server.listen(8000, function () {
    console.log("Creat server on http://127.0.0.1:8000/");
});
```
### 2.3.7 项目启动完成基本测试
保存已编写的所有代码，编辑 `package.json` 文件中 `script` 选项（文中未给出具体代码内容）。 
