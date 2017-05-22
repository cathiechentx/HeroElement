# Proposal: Enhance HeroElement

[HeroElement](https://docs.google.com/document/d/1yRYfYR1DnHtgwC4HRR04ipVVhT1h5gkI6yPmKCgJkyQ/edit#) is proposed by @panicker. We extend it a bit here. In short, HeroElement is an important element in web page which is specified by web developer.

## Why HeroElement?

With HeroElement, user agent could easily distinguish critical part of web page and optimize it. Web developer could have a clear picture of the important element.

## The basic idea of HeroElement

- For User Agent:
	- Get the element that named as HeroElement
	- Optimize and monitor
		- Speed up: speed up the process of HeroElement
		- Monitor and generate events
			- Dom constructed
			- All subresource loaded
			- Fully layouted
			- Fully painted
		- [ElementTiming](https://github.com/w3c/charter-webperf/issues/30): Had been discussed at TPAC 2016

- For web developer:
	- Specify a HeroElement, listen to the corresponding events
	- The HeroElement will be speed up
	- Trigger other movement when events fired
	- Analysis the performance of HeroElement by [ElementTiming](https://github.com/w3c/charter-webperf/issues/30)

## How to use?

- HTML grammar:

		<div heroElement="needspeedup needTiming" onFullPainted="foo()">
		// or
		<div markAsHeroElement="true" speedup="true" elementTiming="..." onFullPaintted="foo()">

- HeroElementAttr:
	- element
		- The element specified as HeroElement
	- needspeedup
		- According to HeroElement，control the parsing process of HeroElement. eg. Once HeroElement be parsed don't break parsing until got its end tag, after that, stop parsing immediately and do next process: layout, paint...
		- Record the sub-resource of HeroElement as key-point sub-resource，high priority them, or cache them before hand.
		- others
	- needTiming
		- [ElementTiming](https://github.com/w3c/charter-webperf/issues/30)
	- events
		- finish dom constructing: Got the end tag of HeroElement
		- finish loading: All relevant subresources downloaded
		- finish layouting： The first layout pass after finishing load
		- finish painting： The first paint after finishing layout

- Use cases
	- Speed up the process of first screen paint. The time of first screen paint is when the first screen content fully painted. From first screen paint users could read web content. User agent would speed up the first screen paint. User agent guesses which elements belong to first screen content. That isn't accurate usually. With HeroElement, user agent could easily know that.

			<!-- All the nodes inside heroElement will be handled in one parse pass -->
			<!-- or <div markAsHeroElement="true" speedup="true">first meaningful paint</div> -->
			<div heroElement="true false">
			  <!-- First screen content begin -->
			  <h1>first meaningful paint<h1>

			  <!-- This image will be speed up -->
			  <img src="...">

			  <!-- Lots of element -->
			  ...
			  <!-- First screen content end -->
			</div>

		QQBrowse has implemented speedup optimization of first screen paint by HeroElement. Cooperating with [sogou search page](https://m.sogou.com/web/searchList.jsp?pid=sogou-clse-2996962656838a97&e=1427&g_f=123&keyword=%E5%A4%A7%E4%B8%BB%E5%AE%B0), this optimization speed up the first screen paint of [sogou search page](https://m.sogou.com/web/searchList.jsp?pid=sogou-clse-2996962656838a97&e=1427&g_f=123&keyword=%E5%A4%A7%E4%B8%BB%E5%AE%B0) in wifi by ** 25% ** on average. See the comparing video [here](http://res.imtt.qq.com///qqbrowser_x5/cathiechen/HeroElement/hev.html)


		Tracing data shortcut compare:
		- without HeroElement Opt
		![without HeroElement Opt](http://i.imgur.com/MoSVgkJ.png)

		- with HeroElement Opt
		![with HeroElement Opt](http://i.imgur.com/nWrJRkg.png)
			

	- Show the HeroElement on screen after it finish paint. With a mask this could reduce the appearance of white screen. But it couldn't be sure if the images has been drew. "FullPaintFinished" fired when all subresouces finish loading. So "FullPaintFinished" could be a signal of HeroElement been fully painted. This solution used a lot in hybrid apps which listened "onPageFinished" of webview to switch views. "onPageFinished" is not the exact time of content fully painted. Go a little further, maybe WebViewClient could consider of adding a "FullPaintFinished" API. 

			<!-- Mask -->
			<div id="mask" style="position:fixed; width:100%; height:100%; background-color:blue;"></div>

			<!-- Content -->
			<!-- or <div markAsHeroElement="true" onFullPaintFinished="foo()" id="hero" style="visibility: hidden;"> -->
			<div heroElement="false false" onFullPaintFinished="foo()" id="hero" style="visibility: hidden;">
			  <!-- Contains lots of images -->
			  ...
			</div>

			...

			<script>
				function foo() {
					document.getElementById("mask").style.display="none";
				}
			</script>

## Continue

- Specify sub-resources that involved in "finish loading"
- Others

## Related

We found the optimizing of first screen content paint will be more effective if we know the exact elements which are shown in the first screen. The concept of [HeroElement](https://docs.google.com/document/d/1yRYfYR1DnHtgwC4HRR04ipVVhT1h5gkI6yPmKCgJkyQ/edit#) proposed by panicker@ is used in [ElementTiming](https://github.com/w3c/charter-webperf/issues/30). We add speeding-up into HeroElement. According to QQBrowser's experiences we know web developers care about the first screen content paint's finishing time. So we add the event notification into HeroElement. Baidu browser had a similar idea before. After discussing with them, we think we could work in this direction.

Any suggestions? @igrigorik, @panicker
