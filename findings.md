# Performance Findings

These are from the homepage. The server is fast (~100 ms for the document) and the page
isn't even that big (~1.8 MB), so this isn't about a slow backend or huge images. It's
JavaScript. Almost the whole page is script, and that's what drags everything down.
Layout is rock solid (CLS 0), so I'm not counting that.

## 1. The main content takes forever to show up

Affects users: you land on the page and the hero you came to see only appears after a long wait, so it feels slow even though nothing's really broken.
Metric: LCP.
Cause: LCP is 13.5 s. The browser is stuck running JavaScript instead of painting the content.
Solution: preload/prioritize the hero (fetchpriority=high) and cut the work that runs before it paints.

## 2. The page is basically all JavaScript

Affects users: you're downloading a lot of code just to read a marketing page, which is wasteful on mobile or slow wifi.
Metric: LCP, Time to Interactive.
Cause: scripts are ~1.48 MB out of ~1.8 MB total, and about 405 KiB of that JS is never used.
Solution: drop the unused code, split the bundles, and defer whatever isn't needed for the first render.

## 3. Too much work on the main thread

Affects users: the page looks ready but doesn't react to taps or scrolls for a moment.
Metric: TBT, Time to Interactive.
Cause: the main thread is busy ~4.2 s (1.9 s of it just booting up scripts), and TTI ends up around 14.5 s.
Solution: run less JS up front, break up long tasks, and move non-critical work off the initial load.

## 4. A lot of it is third-party

Affects users: analytics and other embedded scripts compete for the browser while you're waiting for the page.
Metric: TBT, LCP.
Cause: third parties are ~812 KiB across ~63 requests.
Solution: keep only what's needed and load the rest lazily, after the page is usable.

## 5. Blank for a couple seconds at the start

Affects users: nothing shows for a bit before the first content appears.
Metric: FCP, Speed Index.
Cause: FCP is 2.2 s, held back by scripts that block rendering early on.
Solution: defer non-critical scripts and inline the little bit of critical CSS so something paints sooner.

## Still to do

Run the same check on the other pages, especially faculty (lots of images there, which
could be a different kind of problem), and add field data from pagespeed.web.dev.
