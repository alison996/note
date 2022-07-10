### Node基础
#### 1、读写
**文中用到的flag选项的值，参考：**[http://nodejs.cn/api/fs.html#fs_file_system_flags](http://nodejs.cn/api/fs.html#fs_file_system_flags)
##### 1.1 同步读写
```javascript
const fs = require('fs');

const READ_OPTIONS = { flag: 'r', encoding: 'utf-8' };
// 读取文件
const result = fs.readFileSync('helloworld.txt', READ_OPTIONS);

const WRITE_OPTIONS = { flag: 'a', encoding: 'utf-8' };	// 'a'表示写入的时候向文件末尾追加
// 写入文件
fs.writeFileSync('helloword.txt', 'helloworld', WRITE_OPTIONS);

// 读取目录
const fileList = fs.readdirSync('../dir', READ_OPTIONS);
```

##### 1.2 异步读写
```javascript
const fs = require('fs');

const READ_OPTIONS = { flag: 'r', encoding: 'utf-8' };
// 读取文件
const result = fs.readFile('helloworld.txt', READ_OPTIONS, function(error, data) {
	if (error) {
		console.log(error);
	} else {
		console.log(data);
	}
})

const WRITE_OPTIONS = { flag: 'a', encoding: 'utf-8' };
// 写入文件
fs.writeFile('helloword.txt', 'helloworld', WRITE_OPTIONS, function(error) {
	if (error) {
		console.log(error);
	}
})

// 读取目录
fs.readdir('../dir', WRITE_OPTIONS, function(error, fileList) {
	if (error) {
		console.log(error);
	} else {
		console.log(fileList);
	}
})
```

##### 1.3 总结
`同步的方式`会一直占用`系统资源`，降低了系统的运行效率。故应当采用`异步的方式`，可在原来的基础上使用`Promise`封装一下。

-----

#### 2、控制台输入输出

```javascript
// 引入readline模块
const readline = require('readline');

// 创建readline接口实例
const rlIns = readline.createInterface({
	input: process.stdin,
	output: process.stdout
});

// question方法
rlIns.question('你的名字是？', function(answer) {
	console.log('input:' + answer);
	// 放弃对输入输入流的控制
	rlIns.close();
})

// 监听close事件
rl.on('close', function() {
	// 退出程序
	process.exit(0);
})
```

-----

#### 3、文件流

##### 3.1 写入流
```javascript
const fs = require('fs');

const WRITE_OPTIONS = { flag: 'w', encoding: 'utf-8' };
// 创建写入流
const ws = fs.createWriteStream('helloworld.txt', WRITE_OPTIONS);

// 监听open事件
ws.on('open', function(val) {
  console.log('打开文件', val);
})

// 监听close事件
ws.on('close', function() {
})

// 写入
ws.write('heiheihei', function(error) {
  if (error) {
    console.log(error);
  }
});
// 写入完成
ws.end();
```

##### 3.2 读取流
```javascript
const fs = require('fs');

const READ_OPTIONS = { flag: 'r', encoding: 'utf-8' };
// 创建读取流
const rs = fs.createReadStream('hello.txt', READ_OPTIONS);

// 监听open事件
rs.on('open', function(val) {
  console.log('打开文件', val);
})

// 监听close事件
rs.on('close', function() {
  console.log('关闭文件');
})

// 监听每次读取的数据
rs.on('data', function(chunk) {
  console.log('>>>' + chunk);
})
```

##### 3.3 管道流
```javascript
const fs = require('fs');

// 创建写入流
const ws = fs.createWriteStream('input.txt');
// 创建读取流
const rs = fs.createReadStream('output.txt');
// 将rs读取的数据写入ws的文件中
rs.pipe(ws);
```

-----

#### 4、事件循环

**NodeJs是`单进程` `单线程`的应用程序，通过`V8引擎`提供的`异步执行回调接口`可以处理大量的并发，所以性能非常高**
**事件机制：`观察者模式`**

```javascript
const events = require('events');
const fs = require('fs');

// 实例化事件对象
const event = new events.EventEmitter();
// 注册事件
event.on('triger', function(val) {
  console.log('1', val);
})
event.on('triger', function(val) {
  console.log('2', val);
})
event.on('triger', function(val) {
  console.log('3', val);
})

function readfile(path) {
  return new Promise((resolve, reject) => {
    fs.readFile(path, function(err, data) {
      if (err) {
        reject(err);
      } else {
        resolve(data);
        // 触发自定义事件
        event.emit('triger', data);
      }
    })
  })
}
// 调用读取文件方法
readfile('hello.txt').then(res => {
  console.log('res', res);
})
```

-----

#### 5、路径和系统模块

##### 5.1 获取路径信息path
```javascript
const path = require('path');

const pathStr = 'https://www.hellworld.com/helloworld.html'
// 获取扩展名
const extName = path.extname(pathStr);

// resolve拼接绝对路径
const resolveAbsPath = ['/study', 'node', 'helloworld'];
const absPath = path.resolve(...resolveAbsPath);

// 当前执行目录的完整路径
const dirPath = __dirname;

// join拼接绝对路径
const joinAbsPath = [dirPath, '05path.js'];
const absPath2 = path.join(...joinAbsPath);

// 获取当前执行文件的完整路径
const filePath = __filename;

// 解析path信息
const parsePath = path.parse(filePath);

// 获取文件全名
const fullName = path.basename(filePath);
```
**`path.resolve()`和`path.join()`的区别：**
	path.resolve()： 若`第一个参数`带`/`，则把`第一个参数`当作当前`磁盘`下的`根目录`；若不带`/`， 则自动获取`当前执行目录`， 将`所有参数`按顺序拼接（ ==一定是绝对路径== ）
	path.join()：只将`所有参数`拼接在一起

##### 5.2 获取系统信息os
开发中用到的不多，多是`运维人员`使用。
```javascript
const os = require('os');

// 获取cpu信息
const cpuInfo = os.cpus();

// 获取memory信息
const memoryInfo = os.totalmem();

// 获取剩余内存信息
const freeMem = os.freemem();

// 获取系统架构信息 x64
const archInfo = os.arch();

// 获取系统平台 win32
const platform = os.platform();
```

-----

#### 6、解析URL

```javascript
const url = require('url');

const httpUrl = 'https://sale.testmall.com/pseries.html?cid=128689';
// 解析url
// NEW
const urlInfo_new = new URL(httpUrl);
// OLD
const urlInfo_old = url.parse(httpUrl);

// 合成url
const orgin = 'http://www.test.com/';
// NEW
const target_new = new URL('./helloworld', orgin);  // 注意先后顺序，与resolve相反，且直接返回解析后的对象
// OLD
const target_old = url.resolve(orgin, './helloworld');
```
**原先`引入url模块`的方式改为使用`URL类`。** 

-----
#### 7、搭建服务http
```javascript
const http = require('http');
const path = require('path');
const fs = require('fs');

// 创建服务器实例
const server = http.createServer();
// 普通请求响应头设置
const COMMON_HEADER = { 'Content-Type': 'application/json; charset=utf-8' };
// 图片请求响应头设置
const IMAGE_HEADER = { 'Content-Type': 'image/png; charset=utf-8' };
// 监听请求事件
server.on('request', function(req, res) {
  if (req.url === '/') {
    // 设置状态码和响应头
    res.writeHead(200, COMMON_HEADER);
    res.write('首页');
    res.end();
  } else {
    // 写一个获取资源的
    // 拼接资源路径
    const resourcePath = path.join(__dirname, req.url);
    // 创建读取流
    const rs = fs.createReadStream(resourcePath);
    // 储存读取的chunk
    const resourceData = [];
    // 监听data事件
    rs.on('data', function(chunk) {
      resourceData.push(chunk);
    })
    // 监听end事件
    rs.on('end', function() {
      const resourceBuf = Buffer.concat(resourceData);
      res.writeHead(200, IMAGE_HEADER);
      res.write(resourceBuf);
      res.end();
    })
  }
})

// 绑定、监听端口，开启服务器
server.listen(3000, function() {
  console.log('服务器开启成功，可以通过访问"http://localhost:3000/"来获取数据');
})
```

