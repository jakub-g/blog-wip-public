# Getting started with web performance in 2019

There are many very good articles out there on how to use new feature X or technique Y or tool Z to improve the performance of your websites. Navigating the labyrinth of modern web performance might be tough though if you're new to the topic. In this article I will try to come up with a guide to getting started your performance journey.


# Know your users

Before you start, it's good to know who your users are and how they use your website, in order to make informed decisions later. The important things from a performance aspect are for example:

- % of users on mobile vs desktop browsers,
- split of audience by browser engine: Chrome-like, Safari, Firefox, Edge, IE,
- split of audience by country.

You're very likely already using analytics tools that can answer those questions. Take some time to analyze this data and **write it down somewhere**, including the analysis date, so it's easily accessible when you need it, and available for re-evaluation and comparison in a few months.

_Note that global stats are often different than local ones_: in particular, in countries with high data costs, proxying browsers like UC Browser, Opera Mini etc. are very popular, while almost unused elsewhere.


# Identify test URLs

Come up with a few URLs that will be representative candidates for performance analysis and optimization. You probably want some URLs that receive a **high amount of traffic**, have **relatively stable content** over time, and give a good **approximation of various features** of your website.

Your landing page is a natural first candidate. For a newspaper, a popular past article would be another candidate.

For example, Wikimedia's performance team uses "Barack Obama" and "Sweden" articles from English Wikipedia in their benchmarks and performance monitoring.


# Get to know the tools

Run your page in several performance tools, to get familiar with them and identify some "quick wins". Different tools might provide insights that others are missing, or might explain something better than others – so it's worth trying them all to see which ones are most useful for your particular use cases.

- **Lighthouse** is probably the easiest to get started with, and provides lots of details relevant for modern JS-heavy web applications: run the audit locally via "Audits" tab in Chrome Dev Tools, or from any browser [via a webapp](https://developers.google.com/web/tools/lighthouse/run) or [via WPT](https://www.webpagetest.org/lighthouse).
- **[WebPageTest](https://www.webpagetest.org/)** is the "Swiss Army knife" of web performance, very powerful and with many "hidden features" for advanced users. The more you dive into webperf, the more you will be using it. [Using WebPageTest](http://shop.oreilly.com/product/0636920033592.do) book is a great reference.
- **[Yellow Lab Tools](https://yellowlab.tools/)** can also provide a useful breakdown of various performance metrics.
- While not strictly performance-focused, also check out **[WebHint](https://webhint.io/scanner/)** and **[RedBot](https://redbot.org/)** which can uncover interoperability and correctness issues of your website, suggest improvements, and help you get rid of useless HTTP headers. 
- Check also **[SiteSpeed.io](https://www.sitespeed.io/)** for a vast collection of open source webperf tools.

At this point, you will already know some things that are relatively easy to get fixed (at least in theory), but have high impact: uncompressed assets, unoptimized images etc. 


# Know your HTML

Get your website's HTML output (`Ctrl-U` in Firefox and Chrome, for example), unminify, and analyze it. **Create a high level documentation of what makes it into your HTML, why it's there** (ask your teammates if you don't know), and how many bytes it takes.

On long-living projects, some cruft might have accumulated over the years that is no longer really needed, and can be safely removed. (Bonus points if it's third-party JavaScript!)


# Know your subresources

The next thing to understand your website is to know what HTTP requests a page load triggers, when, and why. It might be helpful to run a WebPageTest test, copy [the list of all the requests (at the bottom)](https://www.webpagetest.org/result/181216_GB_9b0d2e1a261ee5401dfd19842cb74570/1/details/#waterfall_view_step1) to a spreadsheet, and analyze the role of each them (particularly the first 10-20 on the list), and how critical they are.

If you're unsure about what a given request does, you may **try blocking it or introducing a huge artificial latency** and see how the website behaves (this is also useful for uncovering unexpected single-point-of-failure dependencies). There are countless ways to do it: the _"Request blocking"_ feature of Chrome Dev Tools, the _"AutoResponder"_ (with latency) feature of Fiddler, and _"Block"_ or _"SPOF"_ features of WebPageTest; even an adblocker in any browser will do.

By knowing which requests are critical for the user experience, and which are secondary, you can better prioritize your optimization work.

[RequestMap](http://requestmap.webperf.tools/render/181216_GB_9b0d2e1a261ee5401dfd19842cb74570) is a tool that visualizes 3rd-party requests coming from your webapp, and can help uncover long chains of requests that are often bad for performance of the site.


# Have a way to isolate features

Complex webapps often have to implement many functionalities and trigger dozens of requests; analyzing all of it at the same time is not always easy. **By having a possibility to easily disable certain features (e.g. via a query string), you could make your life much easier.**


# Test small

Performance-oriented browser features are often very new and hence not battle-tested yet, so they might have some sneaky bugs or slightly different behaviors in different browsers. 
Understanding how new browser features exactly work might also be tricky when you apply them directly to a big real-life project.

A good idea might be to go back to the basics, **create small isolated HTML pages for testing**, deploy them to a public server (I use GitHub Pages for that), and **run WebPageTest tests in all major browsers.**

**Due to unified WebPageTest UI, it's then easy to compare behavior across browser engines** -- much easier than looking at disparate dev tools interfaces. And if you uncover some unexpected behaviors, you already have a reduced test case to submit a bug report to the browser vendor.


# Think about what to measure

Most early optimizations, like removing/shrinking assets, will be safe to do, bring immediately visible improvements, and generally can't result in regressions. But in order to progress further and long-term, you need to know exactly **what are your KPIs** and **how to measure them**.

This can be surprisingly hard to define, and it heavily depends on what type of website/webapp you have. The "traditional" metrics like time for the browser to fire `load` or `DOMContentLoaded` events are not really suitable for modern websites, and even more for JS-heavy SPAs (Single-Page Applications). In any case, forget about "one number to rule them all". Most likely you'll need several metrics to understand the full picture.


# Learn about existing performance metrics

Apart from being hard to define, certain things are still hard to measure accurately, in particular the visual events (_"when did my images and text become visible?"_). This is difficult to answer, because text rendering might depend on availability of external CSS and webfonts (and if they're pending download, this can result in so-called [Flash of Invisible Text](https://www.zachleat.com/web/fout-vs-foit/)), while the `load` event for the images has been known to not fire accurately (compared to when the pixels are actually rendered on the screen).

Until recently the browsers simply didn't offer enough details to measure [rendering performance](https://speedcurve.com/blog/rendering-metrics/), and WebPageTest's **Speed Index** and **Visually Complete** became the gold standard. Those are quite complex metrics, created by taking a series of screenshots and analyzing how the rendered content changes over time, but you should [take a while to understand them](https://sites.google.com/a/webpagetest.org/docs/using-webpagetest/metrics/speed-index); they convey the user experience much better than simple metrics.

Having said that, browsers (notably Chrome) did some progress in this area in 2018, by shipping in-browser APIs like `PerformanceObserver` which exposes events like [**First Contentful Paint** and **First Meaningful Paint**](https://developers.google.com/web/fundamentals/performance/user-centric-performance-metrics).


# Invent custom metrics

Apart from off-the-shelf metrics, you probably need some custom metrics that are specific to your website; perhaps combining timestamps of custom events in your JavaScript code with resources' fetch times and sizes, taken from [Resource Timing API](https://developer.mozilla.org/en-US/docs/Web/API/Resource_Timing_API/Using_the_Resource_Timing_API).

When creating custom metrics, you may want **a mix of well-defined, fine-grained, actionable metrics** (e.g., _"time between JavaScript `method1()` and `method2()` call"_), **and the higher level metrics** that convey the real user experience (e.g. _"time from the HTML `responseStart` to an `myImportantAppLifecycleEvent`"_). The former are good benchmarks for working explicitly on improving a given metric (and to catch accidental regressions). The latter are less prone to change with large refactors and code changes that trade-off improving one fine-grained metric at the cost worsening the other.

Note that [Navigation Timing](https://nicj.net/navigationtiming-in-practice/) and [Resource Timing](https://nicj.net/resourcetiming-visibility-third-party-scripts-ads-and-page-weight/) APIs have been buggy in some early implementations; watch out for outliers (negative, or extremely huge values) which may skew the statistics (talking of which: make sure to [use percentiles instead of averages](https://phabricator.wikimedia.org/phame/live/7/post/83/measuring_wikipedia_page_load_times/)). Finally, you may want to use a library like [boomerang](https://github.com/akamai/boomerang) instead of writing your own.


# Learn to use throttling

When testing things locally on your powerful MacBook Pro, plugged in, and using gigabit WiFi backed by a fiber optics link, your page will feel fast no matter what. But this is not how your users on a mid-tier Android device, on a mobile connection, perhaps during their commute (hence with unstable internet) will experience your page.

That's why it's important to use **network and CPU throttling** when testing your page. (To be fair, it can be a shocking experience.) 

You can enable throttling via devtools in your favorite browser ([Chrome example](https://developers.google.com/web/tools/chrome-devtools/network-performance/reference#throttling)), on an operating system level (perhaps [with a proxy like Fiddler](https://stackoverflow.com/questions/16276669/simulate-network-speeds-using-fiddler)), or when running tests with WebPageTest ("Advanced Settings" panel). Network throttling is also useful in benchmarks to have comparable numbers over time.

If you want to go even further, you can try a variation of [2G Tuesdays](https://code.fb.com/networking-traffic/building-for-emerging-markets-the-story-behind-2g-tuesdays/).


# Distinguish latency, bandwidth and CPU

As new generation mobile networks are installed and upgraded, internet **bandwidth** rapidly improves each year, even in developing countries. **Modern 4G connections might perform better than outdated landlines** -- the type of connection might be a misleading factor.

**Latency**, on the other hand, doesn't improve as fast, due to physical constraints. The long tail of users who are thousands of kilometres away from your servers can't do anything to improve their latency, but by acknowledging the problem and acting on it (minimizing the number of round-trips, [recognizing the TCP slow start](https://calendar.perfplanet.com/2018/tcp-slow-start/), initiating DNS/TCP/TLS/resource requests early with `dns-prefetch`/`preconnect`/`preload` etc.) you can significantly mitigate the issue.

Last but not least, while high-end mobile devices are as powerful as yesteryear desktops, **low-end devices are barely improving** -- they're just getting cheaper, but still have **slow CPUs and small amounts of RAM** (though likely [enjoying the same network conditions](https://phabricator.wikimedia.org/phame/live/7/post/109/mobile_web_performance_the_importance_of_the_device/)).

Especially when it comes to big JavaScript bundles, bandwidth (download) is no longer a bottleneck -- CPU speed (parsing and executing) is.


# Learn the difference between HTTP/1.1 and HTTP/2

HTTP/2 (a.k.a `h2`, confusingly for HTML writers!) is a major revision of the HTTP protocol that significantly changes how resources are transferred over the wire, and have completely different performance characteristics to HTTP/1.1. Some **"best practices" for HTTP/1 world (like domain sharding) no longer make sense in HTTP/2**, but equally important is that the **implementations of certain HTTP/2 features, both in various browsers and servers, are wildly different** and often incomplete.

As a first thing, check in your favorite devtools' network panel whether your assets are served over HTTP/1 or HTTP/2 (right click the column list and tick `Protocol` entry to make it appear).

Quite likely you'll have a mix of both, if your app uses multiple domains for images, scripts and dynamic data. Once you know this, adjust to the best practices of each transport. In general, you should *try to reuse the HTTP/2 connection as much as possible*; browsers will try to do it for you by "coalescing" some requests into the same connection, [but each of them does it slightly differently](https://twitter.com/__jakub_g/status/1062718923341103106).

_(Keep in mind that most developer proxies -- like Fiddler -- silently downgrade all connections to HTTP/1.1. Make sure to have them disabled when doing performance investigations, to avoid observing misleading behaviors.)_

In general, as of late 2018, **WebPageTest provides more advanced HTTP/2 information than the browsers' devtools**, so if you use HTTP/2 for some of your assets, you should take time to explore WPT waterfall and connection view.


# Become friends with the waterfall

WebPageTest is an extremely powerful tool, and you can read lots of information from its waterfall and connection views. This is a topic for a whole separate blog post, but briefly, the most actionable info you can start from are:

- shape of the waterfall (the more "vertical", the better),
- gaps of "network silence",
- the DNS, TCP, TLS requests on critical resources that happen late -- immediate candidates for [`preconnect` or `dns-prefetch`](https://stackoverflow.com/questions/47273743/preconnect-vs-dns-prefetch-resource-hints)

Understanding the waterfall is essential for doing non-trivial optimizations. You can start with this [presentation on waterfall anti-patterns](https://www.slideshare.net/jrvis/gdl-waterfall-anti-patterns) and gradually dive into the topic with free exploration.


# Know your build tool

Bundlers like webpack have quite a few configuration options that can help with improving performance. Reading through the docs and perfecting your config might take a while, but it's a good investment with potentially high rewards. In particular, [**code splitting**](https://webpack.js.org/guides/code-splitting/) is a critical technique if you ship big amounts of JavaScript.


# Understand the trade-offs

Improving on one metric sometimes means worsening on another; improving things for browser X can yield regressions in browser Y.

Think about it upfront, test regularly all major browser engines, and make sure the keep the balance right. **Create views or filters in your analytics tool to be able to quickly isolate stats from different browsers**, different countries etc. When deploying major changes, make sure to check each view separately.


# Trust but verify

Be wary of the blog posts that offer silver bullets and never mention any drawbacks. Always test on multiple browsers and device types.


# Remember that one size does not fit all

What is good for users on high bandwidth network -- for example, prefetching -- is not always good for users with limited data plans or low bandwidth (developing countries, users on data roaming). [Networking Information API, Client Hints, dynamic request rewriting through a service worker](https://calendar.perfplanet.com/2018/dynamic-resources-browser-network-device-memory/) might be useful to mitigate some of those concerns.


# Write down what you learn

It's easy to get lost when learning too many things too fast. Write down the new things you discovered, bookmark useful URLs etc. so that you can come back to them later. Opening tickets in the issue tracker to further research some ideas might also do the trick.


# Reach out to the community

The perf community is not big, but if you get stuck, you can often count on good and curious people willing to help on [WebPageTest forum](https://www.webpagetest.org/forums/). Twitter might also be a good source of information that isn't always written elsewhere.


# Keep learning and keep shipping, one thing at a time

Understanding the intricacies of web performance takes time, and even equipped with all the knowledge, finding the right course of action might be difficult. Sometimes you just have to roll up your sleeves, do some grunt work, and experiment by trial and error.

No one have found yet a performance silver bullet. Learn new things, come up with ideas, try them out, measure, consider the trade-offs, and ship the improvements -- one at a time.

Enjoy your performance journey!

