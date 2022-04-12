Flex布局之美
========================================
#### Flex基本概念
- 要想熟练掌握flex布局的话，你需要理解两个概念：轴和容器
	![](axis_container.jpg)
1. 将flex布局分为轴和容器。
	- 轴
		- 主轴（mian axis）
		- 交叉轴（cross axis）
	- 容器
		- 父容器（container)
		- 子容器（item）
1. 轴
	![](main_axis.jpg)
	- 轴包括主轴和交叉轴
	- 默认情况下，主轴的方向是从左向右的，交叉轴垂直于主轴，逆时针方向90度，那么接下来我们看flex-direction是如何决定主轴的。
	- 交叉轴是由主轴决定的，主轴又是由flex-direction决定的。
	- flex-direction属性设置在父容器上，才可以生效。
	```
	flex-direction: row | row-reverse | column | column-reverse
	```
	- 首先布局如下👇
	```
	<div class="wrapper">
	        <div class="flex1">子盒子#flex1: 1 </div>
	        <div class="flex2">子盒子#flex2: 1 </div>
	</div>
	```
	```
	当你给父盒子(wrapper)设置属性
	flex-direction: row
	结论：
	flex容器的主轴被定义为与文本方向相同。 主轴起点和主轴终点与内容方向相同。
	简单理解就是主轴沿着水平方向向右
	```
	```
	当你给父盒子(wrapper)设置属性
	flex-direction: row-reverse
	结论：
	表现和row相同，但是置换了主轴起点和主轴终点。
	简单理解就是主轴沿着水平方向向左，与文本方向相反。
	```
	```
	当你给父盒子(wrapper)设置属性
	flex-direction: column
	结论：
	flex容器的主轴和块轴相同。主轴起点与主轴终点和书写模式的前后点相同
	简单的理解，就是主轴变成Y轴方向，方向从上到下布局。
	```
	```
	当你给父盒子(wrapper)设置属性
	flex-direction: column-reverse
	结论：
	表现和column相同，但是置换了主轴起点和主轴终点
	简单的理解，就是主轴变成Y轴方向，方向从下到上，与书写的方向相反。
	```
1. 容器
	- 分为父容器和子容器
	- 父容器
		![](pcontainer.jpg)
		```
		justify-content:  设置子元素在主轴方向上的对齐方式
		align-items： 设置子元素在交叉轴方向上的对齐方式
		```
		```
		当你给父盒子(wrapper)设置属性justify-content: flex-start，子元素沿着主轴方向开始对齐。

		当你给父盒子(wrapper)设置属性justify-content: flex-end，子元素沿着主轴方向终点对齐。

		当你给父盒子(wrapper)设置属性justify-content: center，子元素在主轴方向上水平居中。

		当你给父盒子(wrapper)设置属性justify-content: space-between，子元素在主轴方向上两端对齐，项目之间间隔相等。

		当你给父盒子(wrapper)设置属性justify-content: space-around，子元素在主轴方向上均匀排列每个元素，每个元素周围分配相同的空间。
		```
		```
		当你给父盒子(wrapper)设置属性align-items: flex-start，子元素在交叉轴方向上起点对齐。
		
		当你给父盒子(wrapper)设置属性align-items: flex-end，子元素在交叉轴方向上终点对齐。
		
		当你给父盒子(wrapper)设置属性align-items: center，子元素在交叉轴方向上居中对齐。
		
		当你给父盒子(wrapper)设置属性align-items: baseline，子元素在交叉轴方向上以文字基线对齐，具体不清楚的，可以自行百度。
		
		当你给父盒子(wrapper)设置属性align-items: stretch，这个属性是默认的，如果项目未设置高度或者设为 auto，将占满整个容器的高度。
		```
	- 子容器
		![](scontainer.jpg)
		- 子容器主要介绍2个属性，flex和align-self
			- flex属性 定义在主轴是如何伸缩的
				- 子容器是有弹性的，它们会自动填充剩余空间，子容器的伸缩比由flex属性决定。
				- flex是多个属性的缩写，允许1-3个值的连写，具体参考上面的图。
			- align-self属性 单独设置子容器如何沿交叉轴排列
				- 每个子容器都可以单独定义沿交叉轴排列方式。
				- 该属性的取值跟父容器中的align-items属性一致，如果两者相同的话，则以子容器align-self属性为主。

		- flex作用规则
			- 三个属性的简写，是flex-grow  flex-shrink flex-basis的简写
			- 常用简化写法👇
				- flex:1 —>  flex:1 1 0%;
				- flex:3 —> flex:3 1 0%;
				- 注意:flexbox布局和原来的布局是两个概念，部分css属性在flexbox盒子里面不起作用，eg：float， clear， column,vertical-align 等等
		- align-self作用规则
			```
			起始端对齐
			align-self : flex-start
			
			末尾段对齐
			align-self : flex-end
			
			基线对齐，末尾段对齐
			align-self : baseline
			可以看到它们对齐的方式是第一行文字的基线
			
			拉伸对齐
			align-self : stretch
			```
#### Flex更深入了解
1. 父容器
	- flex-wrap  设置换行方式
		- 绝对子容器是否可以选择换行，一般而言有三种状态，支持换行的话，也支持逆序换行。
	- flex-flow 设置轴向与换行组合
		- 是 flex-direction 和 flex-wrap 的简写。
		- 所以只要掌握，flex-direction 和 flex-wrap即可。
	- align-content  多行沿交叉轴对齐方式
		- 当子容器多行排列时，设置行与行之间的对齐方式。

	- flex-wrap
		- 设置子容器的换行方式，通常有三个取值，flex-wrap: wrap | nowrap | wrap-reverse
		```
		允许换行
		flex-wrap : wrap
		
		不允许换行
		flex-wrap : nowrap
		
		允许逆向换行
		flex-wrap : wrap-reverse
		```


	- flex-flow
		```
		单独设置flex-direction取值，比如
		flex-flow: row | column
		
		单独设置flex-wrap取值
		flex-flow: wrap | nowrap | wrap-reverse
		
		同时设置两者取值
		flex-flow: row wrap
		flex-flow: column nowrap
		```


	- align-content
		- 这个属性是定义子容器在交叉轴的排列方式，也就是对齐方式。
		```
		起始端对齐
		align-content: flex-start
		
		末尾段对齐
		align-content: flex-end
		
		居中对齐
		align-content: center
		
		等间距均匀分布
		align-content: space-between
		
		等边距均匀分布
		align-content: space-around
		
		拉伸对齐
		align-content: stretch
		
		基线对齐
		align-content: baseline
		
		```

1. 子容器
	- flex-grow
		- 子容器弹性伸展的比例，简单理解，就是把剩余的空间按比例分配给子容器。

	- flex-shrink
		- 子容器弹性收缩的比例。简单理解，就是当你子容器超出的部分，会按照对应的比例给子容器减去对应的值。
		- 当取值为0时，就会溢出，那么我们给它们设置一个值，flex-shrink：1
		- 这样子的超出的部分就会按照比列减去。

	- flex-basis
		- 在不伸缩的情况下，flex-basis给子容器设置大小才有作用。
		- 当主轴为横向时，即：
			- flex-direction：row | row-reverse
			- flex-basis设置的大小为宽度，并且会覆盖width值
		- 当主轴为纵向时，即：
			- flex-direction：column | column-reverse
			- flex-basis设置的大小为高度，并且会覆盖height值

	- order
		- 每个子容器的order属性默认为0
		- 通过设置order属性值，改变子容器的排列顺序。
		- 可以是负值，数值越小的话，排的越靠前。

#### 参考
- [https://juejin.cn/post/6866914148387651592](https://juejin.cn/post/6866914148387651592)
