# 新增HeroElement属性的提议

HeroElement属性把页面中最重要的元素标志出来，并对该元素进行优化和监控处理。

## 为啥要有HeroElement属性

有了HeroElement，浏览器可以进行更有针对性的优化，页面开发者可以对该元素进行监控。


## HeroElement的基本思路

- 从浏览角度
	- 得到一个重要元素
	- 对其进行优化和监控
		- speedup: 加速显示过程
		- 监控子树的情况，并给出相应的event
			- dom constructed
			- all subresource loaded
			- fully layouted
			- fully painted
		- 性能监控：[ElementTiming](https://github.com/w3c/charter-webperf/issues/30)
- 从页面开发者角度：
	- 指定一个最重要的元素，监听需要的消息
	- 该元素会被加速显示
	- 收到浏览器通知消息，进行下一步操作
	- 还可以分析该元素的性能瓶颈
## 怎么用

- 页面写法

		<div heroElement="needspeedup needTiming" onFullPainted="foo()">
		// or
		<div markAsHeroElement="true" speedup="true" elementTiming="..." onFullPaintted="foo()">

- 浏览器如何处理
 	- element
		- 被指定为HeroElement的element
	- needspeedup
		- 以heroElement为参考，控制parser的进度，如：不打断parse, 直到解析到heroElement结束tag, 之后马上layout，paint
		- 保存资源相关的子element，调整相关资源的优先级，优先发起子资源请求
		- 其它优化手段
	- needTiming
		- 见google的[ElementTiming](https://github.com/w3c/charter-webperf/issues/30)
	- events
		- finish dom construct: parse到heroElement的结束tag
		- finish loaded: dom构造完，且子树中的所有子资源已经下载完成
		- finish layouted： finish loaded之后的第一次layout结束
		- finish painted： finish layouted之后的第一次paint
- Use Cases
	- 首屏元素加速

			<!-- All the nodes inside heroElement will be handled in one parse pass -->
			<!-- or <div markAsHeroElement="true" speedup="true">first meaningful paint</div> -->
			<div heroElement="true false">
			  <h1>first meaningful paint<h1>
			  <!-- This img will be speed up -->
			  <img src="...">
			  <!-- Lots of element -->
			  ...
			</div>

	- HeroElement完成layout后，改变其visiblity属性

			<!-- or <div markAsHeroElement="true" onFullLayoutFinished="foo()" id="hero" style="visibility: hidden;"> -->
			<div heroElement="false false" onFullLayoutFinished="foo()" id="hero" style="visibility: hidden;">
			  ...
			</div>

			...

			<script>
				function foo() {
					document.getElementById("hero").style.visibility="visible";
				}
			</script>
	- timing：见google文档

## 后期可以继续努力的方向
- 网页开发者可以指定哪些资源（如：图片）不需要卷入其中
- 其它方向的优化

## 相关

这个想法来自于: qq浏览器在加速首屏显示优化的过程中，利用页面开发者指定首屏的标签，并对其加速，效果良好。 后来在github上看到[ElementTiming](https://github.com/w3c/charter-webperf/issues/30)的提议，这个是由panicker提出来的，主要针对timing。现在结合首屏标签，对其进行扩充。在首屏标签的使用过程中，页面开发者关心element内容是否完全绘制到屏幕中，故加上返回event的思路。百度浏览器也参加了讨论，提出过类似的方案，两家浏览器觉得可以往这个方向努力。

建议？ @igrigorik, @panicker
