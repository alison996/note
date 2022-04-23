
# 问题场景
项目中需要实现上传视频前支持预览，确认视频无误后的再确认上传的功能。
第一反应就是获取上传的`file对象`，通过`URL.createObjectUrl(file)`得到一个表示file对象的url，最后`video.src = url`，就可实现。

---

# 问题描述
按照以上思路写完自测后，发现浏览器控制台报错： <font color=red>...unsafe:blob:..</font>（具体报错内容不记得了）。

---

# 原因分析
原因是出于安全考虑，浏览器`不信任video或audio的src来源`。

---

# 解决方案
既然浏览器不信任src的来源，那就在meta中配置下Content-Security-Policy不就行了。
于是我在html中添加meta标签：

```html
<!-- 允许加载同源的、blob格式、data格式的多媒体资源 -->
<meta name="Content-Security-Policy" value="...; media-src 'self' blob: data:">
```
结果报错提示未设置`media-src`，于是浏览器采用了`default-src`默认内容安全策略。
这时我心里就十分纳闷，我明明设置了啊。然后开始排查。最后想起来项目中每次请求时，会在`响应头`中返回`Content-Security-Policy`的配置（***覆盖了前端设置的内容安全策略***）。
于是到后端配置`Content-Security-Policy`的文件中加上`media-src 'self' blob: data:;`。
再跑一遍，发现data类型的可以了，但是blob类型的依旧报错：<font color=red>...unsafe:blob:..</font>
原来为了**防范跨站脚本攻击（XSS）类的安全问题**，angular对内容也进行了过滤。
想解决这个问题，需要使用到angular提供的一个API：`DomSanitizer`。
修改代码如下：
```ts
import { Component, OnInit } from '@angular/core';
import { DomSanitizer } from '@angular/platform-browser';

@Component({
	selector: 'app-demo',
	templateUrl: './app-demo.html',
	styleUrls: './app-demo.scss'
})
export class AppDemoComponent implements OnInit  {
	constructor(
		private sanitizer: DomSanitizer
	){}

	ngOnInit() {}
	
	/**
	* param { any } event 事件对象
	* param { video } video 节点，此处在html中将video当作参数，然后在方法内部直接使用即可
	*/
	onchange(event: any, video: any): void {
		const file = event.target.files[0];
		// blob格式
		video.src = this.sanitizer.bypassSecurityTrustUrl(URL.createObjectUrl(file));
		// data格式，用data格式看起来更复杂，所以没有采用
		/*
		const reader = new FileReader();
		reader.readeAsDataUrl(file);
		reader.onload = function load() {
			video.src = reader.result;
		}
		*/
	}
}
```
</font>

---

#  <font color=red> PS</font>

[Content-Security-Policy配置](https://www.cnblogs.com/bbc66/p/10059080.html)
[Angular官网DomSanitizer介绍](https://angular.cn/api/platform-browser/DomSanitizer)
由于懒得在家中创建demo项目，所以直接手敲的文中的代码，<font color=red>可能存在拼写等错误</font>，若有错误烦请指出。