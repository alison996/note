### Node爬虫-爬取表情包（simple）

#### 1、爬虫原理

​	`模拟浏览器发送请求`，根据`采集规则`采集网页中的数据，将采集的数据`保存到数据库或文件`。

-----


#### 2、实操
##### 2.1 介绍
**node模块：**`url`、`path`、`fs` 
**第三方模块：**`cheerio`、`axios`
**目录结构：**
    | --- root
    | --- | --- node_modules 存放依赖
    | --- | --- package-lock.json 配置信息
    | --- | --- emoticons 存放爬取的表情
    | --- | --- index.js 主文件
    | --- | --- request.js 简单封装的axios
    | --- | --- download.js 下载表情方法
    | --- | --- save.js 保存表情方法

##### 2.2 package-lock.json文件
```json
{
  "requires": true,
  "lockfileVersion": 1,
  "dependencies": {
    "axios": {
      "version": "0.21.1",
      "resolved": "https://registry.npm.taobao.org/axios/download/axios-0.21.1.tgz?cache=0&sync_timestamp=1608611162952&other_urls=https%3A%2F%2Fregistry.npm.taobao.org%2Faxios%2Fdownload%2Faxios-0.21.1.tgz",
      "integrity": "sha1-IlY0gZYvTWvemnbVFu8OXTwJsrg=",
      "requires": {
        "follow-redirects": "^1.10.0"
      }
    },
    "boolbase": {
      "version": "1.0.0",
      "resolved": "https://registry.nlark.com/boolbase/download/boolbase-1.0.0.tgz",
      "integrity": "sha1-aN/1++YMUes3cl6p4+0xDcwed24="
    },
    "cheerio": {
      "version": "1.0.0-rc.10",
      "resolved": "https://registry.nlark.com/cheerio/download/cheerio-1.0.0-rc.10.tgz?cache=0&sync_timestamp=1623167701954&other_urls=https%3A%2F%2Fregistry.nlark.com%2Fcheerio%2Fdownload%2Fcheerio-1.0.0-rc.10.tgz",
      "integrity": "sha1-K6Pc38wm55VvwfRA5h1RxkM3nz4=",
      "requires": {
        "cheerio-select": "^1.5.0",
        "dom-serializer": "^1.3.2",
        "domhandler": "^4.2.0",
        "htmlparser2": "^6.1.0",
        "parse5": "^6.0.1",
        "parse5-htmlparser2-tree-adapter": "^6.0.1",
        "tslib": "^2.2.0"
      }
    },
    "cheerio-select": {
      "version": "1.5.0",
      "resolved": "https://registry.nlark.com/cheerio-select/download/cheerio-select-1.5.0.tgz",
      "integrity": "sha1-+vPa6zGxfF4anavO4oiq+Kr6WCM=",
      "requires": {
        "css-select": "^4.1.3",
        "css-what": "^5.0.1",
        "domelementtype": "^2.2.0",
        "domhandler": "^4.2.0",
        "domutils": "^2.7.0"
      }
    },
    "css-select": {
      "version": "4.1.3",
      "resolved": "https://registry.nlark.com/css-select/download/css-select-4.1.3.tgz?cache=0&sync_timestamp=1622994276976&other_urls=https%3A%2F%2Fregistry.nlark.com%2Fcss-select%2Fdownload%2Fcss-select-4.1.3.tgz",
      "integrity": "sha1-pwRA9wMX8maRGK10/xBeZYSccGc=",
      "requires": {
        "boolbase": "^1.0.0",
        "css-what": "^5.0.0",
        "domhandler": "^4.2.0",
        "domutils": "^2.6.0",
        "nth-check": "^2.0.0"
      }
    },
    "css-what": {
      "version": "5.0.1",
      "resolved": "https://registry.nlark.com/css-what/download/css-what-5.0.1.tgz",
      "integrity": "sha1-PvqCATH0ZpqKwkCPnDLnx96fTK0="
    },
    "dom-serializer": {
      "version": "1.3.2",
      "resolved": "https://registry.nlark.com/dom-serializer/download/dom-serializer-1.3.2.tgz?cache=0&sync_timestamp=1621256918158&other_urls=https%3A%2F%2Fregistry.nlark.com%2Fdom-serializer%2Fdownload%2Fdom-serializer-1.3.2.tgz",
      "integrity": "sha1-YgZDfTLO767HFhgDIwx6ILwbTZE=",
      "requires": {
        "domelementtype": "^2.0.1",
        "domhandler": "^4.2.0",
        "entities": "^2.0.0"
      }
    },
    "domelementtype": {
      "version": "2.2.0",
      "resolved": "https://registry.nlark.com/domelementtype/download/domelementtype-2.2.0.tgz",
      "integrity": "sha1-mgtsJ4LtahxzI9QiZxg9+b2LHVc="
    },
    "domhandler": {
      "version": "4.2.0",
      "resolved": "https://registry.nlark.com/domhandler/download/domhandler-4.2.0.tgz",
      "integrity": "sha1-+XaKXwNL5gqJonwuTQ9066DYsFk=",
      "requires": {
        "domelementtype": "^2.2.0"
      }
    },
    "domutils": {
      "version": "2.7.0",
      "resolved": "https://registry.nlark.com/domutils/download/domutils-2.7.0.tgz?cache=0&other_urls=https%3A%2F%2Fregistry.nlark.com%2Fdomutils%2Fdownload%2Fdomutils-2.7.0.tgz",
      "integrity": "sha1-jrrwxB66/PVbC3LsMcVjI3EsVEI=",
      "requires": {
        "dom-serializer": "^1.0.1",
        "domelementtype": "^2.2.0",
        "domhandler": "^4.2.0"
      }
    },
    "entities": {
      "version": "2.2.0",
      "resolved": "https://registry.nlark.com/entities/download/entities-2.2.0.tgz",
      "integrity": "sha1-CY3JDruD2N/6CJ1VJWs1HTTE2lU="
    },
    "follow-redirects": {
      "version": "1.14.1",
      "resolved": "https://registry.nlark.com/follow-redirects/download/follow-redirects-1.14.1.tgz?cache=0&sync_timestamp=1620555300559&other_urls=https%3A%2F%2Fregistry.nlark.com%2Ffollow-redirects%2Fdownload%2Ffollow-redirects-1.14.1.tgz",
      "integrity": "sha1-2RFN7Qoc/dM04WTmZirQK/2R/0M="
    },
    "htmlparser2": {
      "version": "6.1.0",
      "resolved": "https://registry.nlark.com/htmlparser2/download/htmlparser2-6.1.0.tgz",
      "integrity": "sha1-xNditsM3GgXb5l6UrkOp+EX7j7c=",
      "requires": {
        "domelementtype": "^2.0.1",
        "domhandler": "^4.0.0",
        "domutils": "^2.5.2",
        "entities": "^2.0.0"
      }
    },
    "nth-check": {
      "version": "2.0.0",
      "resolved": "https://registry.nlark.com/nth-check/download/nth-check-2.0.0.tgz",
      "integrity": "sha1-G7T22scAcvwxPoyc0UF7UHTAoSU=",
      "requires": {
        "boolbase": "^1.0.0"
      }
    },
    "parse5": {
      "version": "6.0.1",
      "resolved": "https://registry.nlark.com/parse5/download/parse5-6.0.1.tgz",
      "integrity": "sha1-4aHAhcVps9wIMhGE8Zo5zCf3wws="
    },
    "parse5-htmlparser2-tree-adapter": {
      "version": "6.0.1",
      "resolved": "https://registry.nlark.com/parse5-htmlparser2-tree-adapter/download/parse5-htmlparser2-tree-adapter-6.0.1.tgz",
      "integrity": "sha1-LN+a2CMyEUA3DU2/XT6Sx8jdxuY=",
      "requires": {
        "parse5": "^6.0.1"
      }
    },
    "tslib": {
      "version": "2.3.1",
      "resolved": "https://registry.nlark.com/tslib/download/tslib-2.3.1.tgz?cache=0&sync_timestamp=1628722556410&other_urls=https%3A%2F%2Fregistry.nlark.com%2Ftslib%2Fdownload%2Ftslib-2.3.1.tgz",
      "integrity": "sha1-6KM1rdXOrlGqJh0ypJAVjvBC7wE="
    }
  }
}
```
##### 2.2 index.js文件
```javascript
const url = require('url');
const path = require('path');
// 获取html文档的内容，内容的获取跟jquery一样
const cheerio = require('cheerio');
const req = require('./request');
const download = require('./download');
const save = require('./save');

// 爬取的网址
const targetUrl = 'https://www.fabiaoqing.com/bqb/lists/type/hot/page/1.html';
// 保存的目录名称
const dirname = 'emoticons';

// 发送请求
req.get(targetUrl).then(function(htmlRes) {
  // 定义一个变量保存爬到的表情包url，名称，留待后面下载保存
  const emotions = new Map();
  // cheerio解析html文档
  const $ = cheerio.load(htmlRes); // 返回一个类似与jquery的对象
  $('#container .ui.segment .bqppdiv').each(function(i, el) {
    const name = $(el).children('p').text();
    const emoticonUrl = $(el).children('img').attr('data-original');
    if (!emotions.has(emoticonUrl)) {
      emotions.set(emoticonUrl, name);
    }
  })
  const config = {
    responseType: 'stream'
  };
  // 遍历emotions，
  emotions.forEach((value, key) => {
    // 判断文件名是否包含非法字符
    value = validFileName(value);
    // 根据键获取扩展名
    const extname = path.extname(key);
    // 用值跟扩展名拼接表情文件名
    const filename = value + extname;
    // 调用下载方法
    download(key, config).then(function(res) {
      if (!!res) {
        // 调用保存方法
        save(filename, dirname, res);
      }
    })
  })
})

// 判断是否包含非法字符的正则
const includeReg = /.*?[\/:*?"<>|].*?/;
// 替换用的正则
const replaceReg = /[\/:*?"<>|]/g;
// 校验文件名，不能包含\, /, :, *, ?, ", <, >, |
function validFileName(filename) {
  if (includeReg.test(filename)) {
    // 若包含非法字符，则将非法字符替换成''
    filename = filename.replace(replaceReg, '');
  }
  return filename;
}
```

##### 2.3 request.js文件
```javascript
const axios = require('axios');

function get(url, config) {
  return new Promise(function(resolve, reject) {
    axios.get(url, config).then(function(res) {
      if (!!res.data) {
        resolve(res.data);
      } else {
        reject('request exception: no data.');
      }
    })
  })
}

module.exports = { get };
```

##### 2.4 download.js文件
```javascript
const req = require('./request');
// 下载图片、多媒体文件要设置config = { responseType: 'stream' }
function download(url, confg) {
  return new Promise(function(resolve, reject) {
    req.get(url, confg).then(
      function(res) {
        resolve(res);
      },
      function(error) {
        reject(error);
      }
    )
  })
}

module.exports = download;
```

##### 2.5 save.js文件
```javascript
const fs = require('fs');
const path = require('path');

function save(filename, dirname, data) {
  if (!!filename && !!dirname && !!data) {
    // 拼接文件绝对路径
    let filepath = path.join(__dirname, dirname, filename);
    filepath = handleFileName(filepath);
    // 写入流
    const ws = fs.createWriteStream(filepath, data);
    data.pipe(ws);
    // 关闭流
    data.on('close', function() {
      ws.close();
    })
  }
}

// 判断文件是否存在
function isExisted(filepath) {
  try {
    const stats = fs.statSync(filepath);
    return true;
  } catch(err) {
    return false;
  }
}

// 文件存在则文件名+1
function handleFileName(filepath, sort = 0) {
  const { dir, name, ext } = path.parse(filepath);
  if (isExisted(filepath)) {
    sort++;
    const filename = `${name}${sort}${ext}`;
    filepath = path.join(dir, filename);
    return handleFileName(filepath, sort);
  } else {
    return filepath;
  }
}

module.exports = save;
```

