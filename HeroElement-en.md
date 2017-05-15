# Proposal: Specify an element as HeroElement in web page

HeroElement is an important element in web page which is specified by web developer.

## Why HeroElement?

With HeroElement, user agent could easily distinguish critical part of web page and optimize. Web developer could have a clear picture of the important element.

## The basic idea of HeroElement

- For User Agent:
	- Known the HeroElement
	- optimize and monitor
		- speed up: speed up the process of HeroElement
		- monitor and generate events
			- all subresource loaded
			- dom constructed
			- fully layouted
			- fully painted
		- [ElementTiming](https://github.com/w3c/charter-webperf/issues/30): has been discussed at TPAC 2016

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
	- needspeedup
		- According to HeroElement，control the parsing process of HeroElement，eg. don't break parsing until got the end tag, after that stop parsing immediately，and do next process: layout, paint...
		- Record the sub-resource of HeroElement，high priority these sub-resources
		- others
	- needTiming
		- [ElementTiming](https://github.com/w3c/charter-webperf/issues/30)
	- events
		- finish dom constructing: got the end tag of HeroElement
		- finish loading: after dom constructed， downloaded the sub-resources
		- finish layouting： the first layout pass after finishing load
		- finish painting： the first paint after finishing layout

- Use cases
	- Speed up the process of first meaningful paint

			<div heroElement="true false">first meaningful paint</div>
			//or <div markAsHeroElement="true" speedup="true">first meaningful paint</div>

	- Change visibility after HeroElement finishing layout 

			<div heroElement="false false" onFullLayoutFinished="foo()" id="hero" style="visibility: hidden;">
			//or <div markAsHeroElement="true" onFullLayoutFinished="foo()" id="hero" style="visibility: hidden;">
			...
			<script>
				function foo() {
					document.getElementById("hero").style.visibility="visible";
				}
			</script>

## Continue

- Specify sub-resources that invovled in "finish loading"
- Others

## Related

We found the optimizing of first meaningful paint will be more effective if we know the exact elements which are shown in the first screen. The concept of [HeroElement](https://docs.google.com/document/d/1yRYfYR1DnHtgwC4HRR04ipVVhT1h5gkI6yPmKCgJkyQ/edit#) proposed by panicker@ is used in [ElementTiming](https://github.com/w3c/charter-webperf/issues/30). We add speeding-up into HeroElement. According to QQBrowser's experices we know web devolopers care about the first meaningful paint finishing time. So we add the event notification into HeroElement. Baidu browser had a similar idea before. After discussing with them, we think we could work in this direction.