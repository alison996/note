### 创建主应用、微应用
##### 1. 创建vue主应用

```
vue add vue-base
```
##### 2. 创建微应用

```
// vue微应用
vue add vue-app1

// react微应用
// react 18.2.0
// react-dom 18.2.0
npx create-react-app react-app1 --template typescript
```

<font color="red">遇到的问题：</font>
 1. create-react-app非全局安装创建项目报错
	安装npx，使用npx执行create-react-app命令

**3. 安装`qiankun`**

```
npm i -S qiankun
```

### 注册微应用
##### 1. 在主应用的`entry`文件中注册子应用

```javascript
import { createApp } from 'vue'
import { registerMicroApps, start } from 'qiankun'
import App from './App.vue'

const microApps = [
    {
        name: 'vueApp',
        entry: '//localhost:8080',	// 子应用服务地址
        container: '#container',	// 子应用的容器
        activeRule: '/app-vue',		// 加载子应用的路由
    },
    {
        name: 'reactApp',
        entry: '//localhost:3000',
        container: '#container',
        activeRule: '/app-react',
    },
]

registerMicroApps(microApps)
start({ prefetch: false })

createApp(App).mount('#app')
```

### React微应用 (18.2.0)
##### 1. `src`目录新增`public-path.js`，设置`webpack`的全局变量`__webpack_public_path__`

```javascript
if (window.__POWERED_BY_QIANKUN__) {
	__webpack_public_path__ = window.__INJECTED_PUBLIC_PATH_BY_QIANKUN__
}
```
##### 2. 设置`history`模式路由的`base`（我练习demo时没用到路由）

```html
<BrowserRouter baseName={ window.__POWERED_BY_QIANKUN__ ? '/app-react' : '/' }>
```
##### 3. 修改入口文件`index.tsx`

```typescript
import './public-path.js';
import React from 'react';
import ReactDOM, { Root } from 'react-dom/client';
import App from './App';

let root: Root;

// 加载React子应用的函数
function render(props: any): void {
	// qiankun会传递props，从props中获取子应用的容器
	const { container } = props;
    
    root = ReactDOM.createRoot(
    	(container ? container.querySelector('#root') : document.getElementById('root')) as HTMLElement
    );
    
    root.render(
    	<React.strictMode>
        	<App />
        </React.strictMode>
    )
}

// 独立运行
if (!(window as any).__POWER_BY_QIANKUN__) {
    render({});
}

// 导出qiankun应用的生命周期
export async function bootstrap(): Promise<void> {
    console.log('%c%s', 'color: blue', '[react18.2.0] react app bootstraped');
}
export async function mount(): Promise<void> {
    console.log('%c%s%o', 'color: blue', '[react18.2.0] props from main framwork', props);
    render(props);
}
export async function unmount(): Promise<void> {
    root && root.unmount && root.unmount();
}
```
<font color="red">遇到的问题：</font>
 1. `react18.2.0`中废弃了`ReactDOM.render`和`ReactDOM.unmountComponentAtNode`
	先用`ReactDOM.createRoot`创建`root`，再使用`root.render`和`root.unmount`

##### 4. 修改`webpack`配置

**安装插件`@rescripts/cli`**

```
npm i -D @rescripts/cli --force
```
<font color="red">遇到的问题：</font>
 1. `@rescripts/cli`依赖`react-scripts: 2 - 4`，导致安装时报错
	使用`--force`选项，强制安装`@rescripts/cli`

**根目录新增`.rescriptsrc.js`**

```javascript
const { name } = require('./package');

module.exports = {
    webpack: (config) => {
        config.output.library = `${name}-[name]`;
        config.output.libraryTarget = 'umd';
        // webpack 4.0
        // config.output.jsonpFunction = `webpackJsonp_${name}`;

        // webpack 5.0
        config.output.chunkLoadingGlobal = `webpackJsonp_${name}`;
        config.output.globalObject = 'window';

        return config;
    },

    devServer: (_) => {
        const config = _;

        config.headers = {
            'Access-Control-Allow-Origin': '*',
        };
        config.historyApiFallback = true;
        config.hot = false;
        // webpack 4.0
        // config.watchContentBase = false;

        // webpack 5.0
        config.static.watch = false;
        config.liveReload = false;

        return config;
    }
};
```

<font color="red">遇到的问题：</font>
1. 报错信息：configuration.output has an unknown property 'jsonpFunction'. These properties are valid...
   `webpack 5`已将`output.jsonpFunction`更名为`output.chunkLoadingGlobal`
2. 报错信息：options has an unknown property 'watchContentBase'. These properties are valid...
   `webpack 5`已将`devServer.watchContentBase`更名为`devServer.static.watch`

#### 5. 修改`package.json`
将`react-scripts`替换为`rescripts`
`eslintConfig`添加`globals.__webpack_public_path__`配置
```json
{
	"eslintConfig": {
		"globals": {
			"__webpack_public_path__": true
		}
	}
}
```

### Vue微应用 (3.2.13)
#### 1. `src`目录新增`public-path.js`（同`React微应用步骤1`）
#### 2. 路由略
#### 3. 修改入口文件`main.ts`
```typescript
import './public-path'
import { createApp } from 'vue'
import App from './App.vue'


let root: any;

function render(props: any): void {
    const { container } = props;

    root = createApp(App).mount(container ? container.querySelector('#app') : '#app');
}

// 独立运行
if (!(window as any).__POWERED_BY_QIANKUN__) {
    render({});
}

export async function bootstrap(): Promise<void> {
    console.log('%c%s', 'color: blue', '[vue3] vue app bootstraped');
}

export async function mount(props: any): Promise<void> {
    console.log('%c%s%o', 'color: blue', '[vue3] props from main framwork', props);
    render(props);
}

export async function unmount(): Promise<void> {
    root && root.unmount && root.unmount();
}
```
##### 4. 修改`vue.config.js`文件
```js
const { defineConfig } = require('@vue/cli-service')
const { name } = require('./package.json')

module.exports = defineConfig({
    transpileDependencies: true,
    devServer: {
        headers: {
            "Access-Control-Allow-Origin": '*'
        }
    },
    configureWebpack: {
        output: {
            library: `${name}-[name]`,
            libraryTarget: 'umd',
            // webpack 4
            // jsonpFunction: `webpackJsonp_${name}`
            
            // webpack 5
            chunkLoadingGlobal: `webpackJsonp_${name}`
        }
    }
})
```
#### 5. 修改`package.json`
`eslintConfig`添加`globals.__webpack_public_path__`配置