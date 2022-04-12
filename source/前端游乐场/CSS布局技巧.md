CSS布局技巧
========================================
#### 水平垂直居中
1. Flexbox中实现水平垂直居中
	- Flex是Flexible Box的缩写，意为”弹性布局”，用来为盒状模型提供最大的灵活性
	- 设置 justify-content（对齐方式）、align-items（对齐方式） 的值为 center 时，可以让元素在Flex容器中达到水平垂直居中的效果
	```
	  display: flex
	  justify-content: center
	  align-items: center
	```
	- 这种方式特别适应于让Icon图标在容器中水平垂直居中，不同的是在Icon图标容器上显示设置 display: inline-flex
	```
	display: inline-flex
	align-items: center
	justify-content: center
	```
	- 如果要让多个元素实现水平垂直居中的效果，那还需要加上 flex-direction: column（需要设置在父容器）
	```
	display: flex
	flex-direction: column
	justify-content: center
	align-items: center
	```
	或者换成
	```
	display: flex
	place-content: center
	place-items: center
	```
1. Grid中实现水平垂直居中
	- CSS Grid布局可以说是现代Web布局中的银弹，它也是到目前为止布局系统中唯一一个二维布局系统
	```
	display: grid
	place-items: center
	```
#### 等高布局
- 在Flexbox和Grid布局模块中，实现等高布局非常的简单
1. Flexbox布局
	- 在容器上显式设置了 display 的值为 flex 或 inline-flex，该容器的所有子元素的高度都相等，因为容器的 align-items 的默认值为 stretch（拉伸）
	```
	display: flex
	```
1. Grid布局
	```
	display: grid
	grid-template-columns: 10% 40% 50%
	```
#### Sticky Footer
1. stickky footer图例
	![file:///D:/readthedoc/readthedocs/source/前端游乐场/StickyFooter.png](StickyFooter.png)
2. Flexbox布局实现
	```
	  <div class="body">
		<div class="header">header</div>
		<div class="main">main</div>
		<div class="footer">footer</div>
	  </div>
	```
	```
	.body
	  display: flex
	  flex-direction: column
	  height: 100%
	  background-color: #795da3

	.header
	  background-color: #2d8cf0

	.main
	  background-color: #a71d5d

	.footer
	  **margin-top: auto**
	  background-color: #2b821d
	```
	- 在Flexbox布局中，还可以在main上设置下面的样式，达到相等的效果
	```
	flex: 1 0 auto
	```
	- flex属性是flex-grow, flex-shrink 和 flex-basis的简写，默认值为0 1 auto。
	- 该属性有两个快捷值：
		- flex: auto >> flex: 1 1 auto
		- flex: none >> flex: 0 0 auto
	- flex-grow属性定义容器的放大比例，默认为0，即如果存在剩余空间，不会扩展；设为1，容器有剩余空间时，main区域会扩展。
	- flex-shrink属性定义了容器的缩小比例，默认为1，即如果空间不足，该项目将缩小；设为0，容器空间不足时，main区域不会收缩。
	- flex-basis属性定义了在分配多余空间之前，容器占据的主轴空间（main size）。浏览器根据这个属性，计算主轴是否有多余空间。它的默认值为auto，即容器的本来大小。
1. 在CSS Grid布局中我们可以借助1fr（自适应单位）让main区域根据Grid容器剩余空间来做计算
	```
	.body
	  height: 100%
	  background-color: #795da3
	  display: grid
	  grid-template-rows: auto 1fr auto

	.header
	  background-color: #2d8cf0

	.main
	  background-color: #a71d5d

	.footer
	  background-color: #2b821d
	```
#### 均分列
1. Flexbox中的布局
	- 在Flexbox布局模块中，当flex取的值是一个单值（无单位的数），比如示例中的 flex:1，它会当作显式的设置了 flex-grow: 1
	```
	.body
	  height: 100%
	  background-color: #795da3
	  **display: flex**
	
	.item
	  background-color: #2b821d
	  **flex: 1**
	```
1. Grid中的布局
	```
	.body
	  height: 100%
	  background-color: #795da3
	  **display: grid**
	  **grid-template-columns: repeat(3, 1fr)**  //3表示列数

	.item
	  background-color: #2b821d
	  margin: 1%
	```
1. 问题点
	- 不管是Flexbox还是Grid布局中，都存在一定的缺陷，当容器没有足够的空间容纳Flex项目（或Grid项目）时，Flex项目或Grid项目会溢出（或隐藏，如果Flex容器或Grid容器显式设置了 overflow:hidden）
	- 修复这种现象最简单的方式是在Flex容器或Grid容器显式设置一个 min-width（或 min-inline-size）
	```
	.body
	  height: 100%
	  background-color: #795da3
	  display: grid
	  grid-template-columns: repeat(3, 1fr)
	  **min-inline-size: 300px**
	```
	- 当容器没有足够空间时，希望Flex项目（或Grid项目）会自动断行排列
	```
	.body
	  height: 100%
	  background-color: #795da3
	  display: flex
	  flex-wrap: wrap

	.item
	  background-color: #2b821d
	  flex: 0 1 calc((100vw - 8vh) / 4) // vh、vw：相对于视口的高度和宽度，vh就是当前屏幕可见高度的1%，即height:100vh == height:100%;
	```
	- 基于该例，如果把Flex项目的 flex 值改成
	```
	.body
	  height: 100%
	  background-color: #795da3
	  display: flex
	  flex-wrap: wrap

	.item
	  background-color: #2b821d
	  **flex: 0 0 400px**
	- 这个时候，当Flex容器没有足够空间时，Flex项目会按 flex-basis: 400px 计算其宽度，Flex容器没有足够空间时，Flex就会断行
	```
	- 反过来，如果Flex项目的值 flex 改成
	```
	.body
	  height: 100%
	  background-color: #795da3
	  display: flex
	  flex-wrap: wrap

	.item
	  background-color: #2b821d
	  **flex: 1 0 400px**
	- 当Flex容器没有足够空间排列Flex项目时，Flex项目会按 flex-basis: 400px 计算其宽度，Flex会断行，并且同一行出现剩余空间时，Flex项目会扩展，占满整个Flex容器
	```
	- Grid中实现类似的效果要更复杂一点。可以使用 repeat() 函数，1fr 以及 auto-fit 等特性
	```
	.body
	  height: 100%
	  background-color: #795da3
	  display: grid
	  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr))
	  gap: 2vh
	```
	- 其实在Grid中与 auto-fit 对比的值还有一个叫 auto-fill。但两者的差异是非常地大，用下图来描述 auto-fit 和 auto-fill 的差异
	![file:///D:/readthedoc/readthedocs/source/前端游乐场/autofit_autofill.png](autofit_autofill.png)
#### 圣杯布局
1. 圣杯图例
	![file:///D:/readthedoc/readthedocs/source/前端游乐场/圣杯.png](圣杯.png)
2. 使用Flexbox
	- 对于圣杯布局而言，HTML结构是有一定的要求，那就是内容为先
	```
	  <div class="body">
		<div class="header">header</div>
		<div class="main">
		  <div class="article">article</div>
		  <div class="nav">nav</div>
		  <div class="aside">aside</div>
		</div>
		<div class="footer">footer</div>
	  </div>
	```
	```
	.body
	  height: 100%
	  background-color: #795da3
	  display: flex
	  flex-direction: column

	.header
	  background-color: #2d8cf0

	.main
	  background-color: #a71d5d
	  display: flex
	  flex: 1
	  align-items: stretch

	.article
	  background-color: #97a8be
	  flex: 1

	.nav
	  width: 220px
	  order: -1
	  background-color: #7b8ada

	.aside
	  width: 220px
	  background-color: #9a6e3a

	.footer
	  background-color: #2b821d
	  margin-top: auto
	```
	- 通过在 nav、aside 和 article 上显式设置 order 的值，可以很好的控制这三个区域的布局顺序。比如说，希望 aside在article之前排列，只需要在上面的示例基础上做一点点调整
	```
	.nav
	  width: 220px
	  order: 0
	  background-color: #7b8ada

	.aside
	  width: 220px
	  order: -1
	  background-color: #9a6e3a
	```
	- 注意，order的默认值为0，值越大越排在后面
1. 使用Grid布局
	- 在Grid布局模块中，实现圣杯布局要比Flexbox布局模块中更容易，而且更灵活
	```
	  <div class="body">
		<div class="header">header</div>
		<div class="main">main</div>
		<div class="nav">nav</div>
		<div class="aside">aside</div>
		<div class="footer">footer</div>
	  </div>
	```
	```
	.body
	  height: 100%
	  background-color: #795da3
	  display: grid
	  grid-template: auto 1fr auto / 220px 1fr 220px

	.header
	  background-color: #2d8cf0
	  grid-column: 1/4

	.main
	  background-color: #a71d5d
	  grid-column: 2 / 3
	  grid-row: 2 / 3

	.nav
	  background-color: #7b8ada
	  grid-column: 1 / 2
	  grid-row: 2 / 3

	.aside
	  background-color: #9a6e3a
	  grid-column: 3 / 4
	  grid-row: 2 / 3

	.footer
	  background-color: #2b821d
	  grid-column: 1 / 4
	```
#### 两端对齐
1. 在Flexbox布局中，时常在Flex容器中显式设置 justify-content 的值
	```
	height: 100%
	width: 100%
	background-color: #795da3
	display: flex
	flex-wrap: wrap
	justify-content: space-between
	```
	- 但在末尾行，如果和前面行的个数不相同（Flex项目）就会出现非紧挨着排列的效果
	- 我们希望在最后一行的Flex项目不足够排列满一行时，希望Flex项目一个紧挨一个的排列
	```
	.body
	  height: 100%
	  width: 100%
	  display: grid
	  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr))
	  gap: 1vh
	```
#### Logo图标的对齐
- Logo图像的有大有小（宽度和高度都不一样）。面对这样的业务场景，很多时候都希望设计师能提供相同尺寸的图像。但这样势必会影响Logo图像的外观
- 实现这样的布局效果，主要运用到的就是CSS的 object-fit 属性
```
width: 130px
height: 75px
object-fit: contain
```
- 有些Logo图像带有背景颜色
```
width: 130px
height: 75px
object-fit: contain
mix-blend-mode: multiply
```
#### 选择最佳的值
- 主要介绍clam()函数，接受三个参数，即 clam(MIN, VAL, MAX)，其中 MIN 表示最小值，VAL 表示首选值，MAX 表示最大值。它们之间：
- 如果 VAL 在 MIN 和 MAX 之间，则使用 VAL 作为函数的返回值。
- 如果 VAL 大于 MAX，则使用 MAX 作为函数的返回值。
- 如果 VAL 小于 MIN，则使用 MIN 作为函数的返回值
```
width: clamp(100px, 50vw, 500px)
```
#### 参考
- [https://mp.weixin.qq.com/s/9f4UaZWzYSJB_ZdwhS3A3A](https://mp.weixin.qq.com/s/9f4UaZWzYSJB_ZdwhS3A3A)