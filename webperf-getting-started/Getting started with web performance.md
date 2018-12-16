# Getting started with web performance in 2019

There's many very good articles in the web those days how to use a new feature X or technique Y or tool Z to improve the performance of your websites. Having said that, navigating the labyrinth of modern web performance might be tough if you're new to the topic. In this article I will try to come up with a guide to getting started your performance journey.

# Know your users

Before you start, it's good to know who are your users, and how they use your website, in order to take informed decisions later. The important things from performance aspect are for example:

- % of users on mobile vs desktop browsers,
- split of audience by browser engine: Chrome-like, Safari, Firefox, Edge, IE
- split of audience by country.

Very likely you're already using analytics tool that can answer those questions. Take some time to analyze this data and **write it down somewhere**, including the analysis date, so it's easily accessible when you need it, and available for re-evaluation and comparison every few months.

_Note that global stats are often different than local ones_: in some countries, a given browser might be much more popular than the global data suggests; in particular, in countries with high data costs, proxying browsers like UC Browser, Opera Mini etc. are heavily popular, while almost unused elsewhere.


# Identify test URLs

Come up with a few URLs that will be representative candidates for performance analysis and optimization. Probably you want some URLs that receive high amount of traffic, have relatively stable content over time, and give a good approximation of various features of your website.

Your landing page is a first natural candidate. If you're a newspaper, some selected article will be the next one etc.

For example, Wikimedia's performance team uses "Barack Obama" and "Sweden" articles from English Wikipedia in their benchmarks and performance monitoring.

# Get to know the tools

Run your page in several performance tools, to get familiar with them, and identify some "quick wins". Each of those tools can provide some insights that others may be missing, or can explain it better, so it's worth trying them all to see which ones are most useful for your particular use cases.

- Lighthouse is probably the easiest to get started, provides lots of details relevant for modern JS-heavy web applications: run the audit locally via "Audits" tab in Chrome Dev Tools, or from any browser [via a webapp](https://developers.google.com/web/tools/lighthouse/run) or [via WPT](https://www.webpagetest.org/lighthouse).
- [WebPageTest](https://www.webpagetest.org/) is the Swiss army knife of web performance, very powerful and with many "hidden features". The more you dive into webperf, the more you will be using it.
- [Yellow Lab Tools](https://yellowlab.tools/) can also provide a useful breakdown of various performance metrics.
- While not strictly performance-focused, check also [WebHint](https://webhint.io/scanner/) and [RedBot](https://redbot.org/) which can uncover interoperability and correctness issues of your website, suggest improvements, and help you get rid of useless HTTP headers. 
- Check also [SiteSpeed.io](https://www.sitespeed.io/) for a vast collection of open source webperf tools.

At this point, you will already know some things are relatively easy to get fixed (at least in theory), but have high impact: uncompressed assets, unoptimized images etc. 


# Learn to use throttling

When testing things locally on your powerful MacBook Pro, connected to the charger, and using gigabit WiFi backed by a fiber optics link, your page will feel fast no matter what. But this is not how your users on a mid-tier Android device, on a 3G connection, perhaps during their commute (hence with unstable internet) will experience your page. 

That's why it's important to use network and CPU throttling when testing your page. (To be fair, it can be a shocking experience!) You can enable throttling via devtools in your favorite browser, on an operating system level (perhaps with a proxy like Fiddler), or when running tests with WebPageTest.


# Learn the difference between http/1.1 and http/2

HTTP/2 (a.k.a `h2`, confusingly for HTML writers!) is a major revision of the HTTP protocol that significantly changes how the resources are transferred over the wire, and have completely different performance characteristics than HTTP/1.1. Some "best practices" or HTTP/1 world (like domain sharding) no longer make sense in HTTP/2, but equally important is that the implementations of HTTP/2 both in different browsers and servers are wildly different and often incomplete.

As a first thing, check in your favorite devtools whether your assets are served over HTTP/1 or HTTP/2 (right click the column list and tick `Protocol` entry to make it appear).

Most likely you'll have a mix of both, if your app uses multiple endpoints for images, scripts and dynamic data. Once you know this, adjust to the best practices of each transport. In general, you should try reuse the HTTP/2 connection as much as possible; browsers will try to do it for you by "coalescing" some requests into the same connection, [but each of them does it slightly differently](https://twitter.com/__jakub_g/status/1062718923341103106).

In general, as of late 2018, WebPageTest provides much more advanced HTTP/2 information than the browsers' devtools, so if you use HTTP/2, you should take some time to explore WPT waterfall and connection view.

# Know what to measure

Most of the things you'll do in the beginning, like removing/shrinking assets, are "safe" to do, perhaps bring immediately visible improvements, and can't bring any regressions. But in order to progress, you need to know exactly _what are your KPIs and to measure them._

This can be surprisingly hard to define, and it heavily depends on what type of website/webapp you have. The "traditional" metrics like time for the browser to fire `load` or `DOMContentLoaded` events are not really suitable for modern websites, and in for JS-heavy SPAs (Single-Page Applications). In any case, forget about "one number to rule them all". Most likely you'll need several custom metrics.

Apart from being hard to define, certain things are still hard to measure accurately, in particular the visual events (_"when did my images and text become visible?"_). This is difficult to answer, because text rendering might depend on availability of external CSS and webfonts (and if they're pending download, this can result in so-called [Flash of Invisible Text](https://www.zachleat.com/web/fout-vs-foit/)), while the `load` event for the images has been known to not fire accurately compared to when the pixels are actually rendered on the screen.

Until recently the browsers simply didn't offer enough details to measure [rendering performance](https://speedcurve.com/blog/rendering-metrics/), and WebPageTest's Speed Index and Visually Complete became the gold standard. Those are quite complex metrics, created by taking a series of screenshots during rendering and analyzing how the viewport content changes over time, but you should [take a while to understand them](https://sites.google.com/a/webpagetest.org/docs/using-webpagetest/metrics/speed-index); they convey the user experience much better than simple metrics..
  
Having said that, browsers (notably Chrome) did some progress in this area in 2018, by shipping in-browser APIs like `PerformanceObserver` which expose events like [First Contentful Paint and First Meaningful Paint](https://developers.google.com/web/fundamentals/performance/user-centric-performance-metrics).

Apart from off-the-shelf metrics, you probably need some custom metrics that are specific to your website, combining timestamps of custom events in your JavaScript code with resources' fetch times and sizes, taken from [Resource Timing API](https://developer.mozilla.org/en-US/docs/Web/API/Resource_Timing_API/Using_the_Resource_Timing_API).

When creating custom metrics, you may want a mix of well-defined, precise, fine-grained, actionable metrics (e.g., time between JavaScript `method1()` and `method2()` call), and the higher level metrics that convey the real user experience (e.g. time from the HTML response start to an important app lifecycle event). The former are good benchmark points if you work explicitly on improving a given metric, while the latter are less prone to the large refactorings you may do, and the code changes that trade-off improving one fine-grained metric at the cost worsening the other.

# Know your HTML

Get your website's HTML output (`Ctrl-U` in Firefox and Chrome, for example), unminify it, and analyze it. Create a high level documentation of what makes it to your HTML, why it's there (ask your teammates if you don't know), and how many bytes it takes.

On long-living projects, it might be the case that some cruft has accumulated over the years that is not really needed anymore, and can be safely removed. (bonus points if it's a third-party JavaScript!)

# Know your requests

The next thing to understand your website is to know what HTTP requests a page load triggers. It might be helpful to run a WebPageTest test, copy [the list of all the requests (at the bottom)](https://www.webpagetest.org/result/181216_GB_9b0d2e1a261ee5401dfd19842cb74570/1/details/#waterfall_view_step1) to a spreadsheet, and analyze the role of each them (particularly the first 20-30 on the list), and how critical are they.

If you're unsure about what a given request does, you may try blocking it or introducing a huge latency. There are countless options to do it: the "Request blocking" feature of Chrome Dev Tools, with an "AutoResponder" (with latency) feature of Fiddler, with "Block" or "SPOF" features of WebPageTest; even an adblocker in any browser will do.

By knowing which requests are critical for the user experience, and which are secondary, you can better prioritize your optimization work.

[RequestMap](http://requestmap.webperf.tools/render/181216_GB_9b0d2e1a261ee5401dfd19842cb74570) is a tool that visualizes 3rd-party requests coming from your webapp, and can help uncovering long chains of requests that are often bad for performance of the site.

# Become friends with the waterfall