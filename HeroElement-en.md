# Proposal: Specify an element as HeroElement in web page

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
	- Specify a HeroElement, listen to the corresponding event
	- The HeroElement will be speed up
	- Trigger other movement when event fired
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
		- According to HeroElement，control the parsing process of HeroElement，eg. Don't break parsing until got the end tag, after that stop parsing immediately and do next process: layout, paint...
		- Record the sub-resource of HeroElement，high priority these sub-resources
		- others
	- needTiming
		- [ElementTiming](https://github.com/w3c/charter-webperf/issues/30)
	- events
		- finish dom constructing: Got the end tag of HeroElement
		- finish loading: All relevant subresources downloaded
		- finish layouting： The first layout pass after finishing load
		- finish painting： The first paint after finishing layout

- Use cases
	- Speed up the process of first meaningful paint

			<!-- All the nodes inside heroElement will be handled in one parse pass -->
			<!-- or <div markAsHeroElement="true" speedup="true">first meaningful paint</div> -->
			<div heroElement="true false">
			  <h1>first meaningful paint<h1>
			  <!-- This img will be speed up -->
			  <img src="...">
			  <!-- Lots of element -->
			  ...
			</div>
			

	- Change visibility after HeroElement finishing layout 

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

## Continue

- Specify sub-resources that involved in "finish loading"
- Others

## Related

We found the optimizing of first meaningful paint will be more effective if we know the exact elements which are shown in the first screen. The concept of [HeroElement](https://docs.google.com/document/d/1yRYfYR1DnHtgwC4HRR04ipVVhT1h5gkI6yPmKCgJkyQ/edit#) proposed by panicker@ is used in [ElementTiming](https://github.com/w3c/charter-webperf/issues/30). We add speeding-up into HeroElement. According to QQBrowser's experiences we know web developers care about the first meaningful paint finishing time. So we add the event notification into HeroElement. Baidu browser had a similar idea before. After discussing with them, we think we could work in this direction.

Any suggestions? @igrigorik, @panicker
