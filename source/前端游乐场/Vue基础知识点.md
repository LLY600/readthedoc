Vue基础知识点
=============
#### Vue是什么？
- 一套用于**构建用户界面**的**渐进式**javascript框架
	- 构建用户界面：将数据（mock、接口等）构建成能展示的页面
	- 渐进式：Vue可以自底向上逐层的应用，[渐进式](https://v3.cn.vuejs.org/guide/introduction.html#vue-js-%E6%98%AF%E4%BB%80%E4%B9%88)
#### Vue的特点
1. 组件化模式，提高代码复用率，代码易维护
2. 声明式渲染，程序员无需直接操作DOM（DocumentObjectModel文档对象模型），提高开发效率 [Dom](https://www.runoob.com/htmldom/htmldom-tutorial.html)
	```
	- 命令式编程
	- 准备html字符串 
	let htmlstr =''
	- 遍历数据拼接html字符串 
	persons.forEach( p => {htmlStr += `<li>${p.id} - ${p.name} - ${p.age}</li>});
	- 获取list元素
	let list = document.getElementById('list')
	- 修改内容(亲自操作DOM) 
	list.innerHTML = htmlstr
	```
	```
	声明式编程
	<ul id="list">
	<li v-for="p in persons">
	{{p.id}} - {{p.name}} - {{p.age}}</li></ul>
	```
1. 使用虚拟DOM和Diff算法，尽量复用DOM节点
	- 原生js实现页面展示，是直接操作页面真实DOM，如果数据发生更新，会直接消除原来数据，全量更新数据。
	```
	比如初始数据是【A,B,C】，更新数据是【A,B,C,D】，会直接全部更新，并不会只更新【D】，即未复用【A,B,C】。
	```
	- Vue实现，在数据和页面真实DOM之间多了虚拟DOM，它会把数据先变成虚拟DOM（存内存里的数据），再将虚拟DOM变成页面真实DOM。
	```
	比如初始数据是【A,B,C】，更新数据是【A,B,C,D】，因Vue开始已经存在了虚拟DOM数据【A,B,C】，新生成的虚拟DOM数据【A,B,C,D】会和旧的虚拟数据比较，这个比较就用到了优秀的Diff算法，发现旧的DOM里面存在【A,B,C】，就直接复用
	```
#### 简介
1. 定义
- 实现应用中**局部**功能**代码**（html、css、js）和**资源**(图片、视频...)的集合；对于局部的理解，指组件拆分合理，不存在一个页面一个组件，这样就没有复用性可言
![](components.png)
1. 为什么要使用组件
- 通常页面的功能比较复杂
1. 作用
- 复用代码，简化项目编码，提高运行效率
1. 分类
- 非单文件组件
	- 一个文件中包含N个组件，如xxx.html
- 单文件组件
	- 一个文件中只包含1个组件，如xxx.vue
#### 参考
- 官方文档：[https://v3.cn.vuejs.org/](https://v3.cn.vuejs.org/)
- B站视频：[https://www.bilibili.com/video/BV1Zy4y1K7SH](https://www.bilibili.com/video/BV1Zy4y1K7SH)