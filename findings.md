# Performance Findings

These are from the homepage. The server is fast enough and layout is stable, so the
problem is not really backend or CLS. The slow part is mostly JavaScript, third-party
code, and the fact that the refresh doesn't really get cheaper.

## 1. The main content takes forever to show up

Affects users: you land on the page and the hero you came to see only appears after a long wait, so it feels slow even though nothing's really broken.
Metric: LCP.
Cause: LCP is 13.5 s. The browser is stuck running JavaScript instead of painting the content.
Solution: preload/prioritize the hero (fetchpriority=high) and cut the work that runs before it paints.

## 2. The page is basically all JavaScript

Affects users: you're downloading a lot of code just to read a marketing page, which is wasteful on mobile or slow wifi.
Metric: LCP, Time to Interactive.
Cause: scripts are ~1.44 MB out of ~1.87 MB transferred, and about 405 KiB of that JS is never used.
Solution: drop the unused code, split the bundles, and defer whatever isn't needed for the first render.

## 3. Too much work on the main thread

Affects users: the page looks ready but doesn't react to taps or scrolls for a moment.
Metric: TBT, Time to Interactive.
Cause: the main thread is busy ~4.2 s (1.9 s of it just booting up scripts), and TTI ends up around 14.5 s.
Solution: run less JS up front, break up long tasks, and move non-critical work off the initial load.

## 4. A lot of it is third-party

Affects users: analytics and other embedded scripts compete for the browser while you're waiting for the page.
Metric: TBT, LCP.
Cause: third parties are 819.5 KB across 25 requests when the Harbour.Space asset CDN is
counted as first-party.
Solution: keep only what's needed and load the rest lazily, after the page is usable.

## 5. Blank for a couple seconds at the start

Affects users: nothing shows for a bit before the first content appears.
Metric: FCP, Speed Index.
Cause: FCP is 2.2 s, held back by scripts that block rendering early on.
Solution: defer non-critical scripts and inline the little bit of critical CSS so something paints sooner.

## 6. There are just a lot of requests

Affects users: the page has more little things to wait for, which is worse on mobile.
Metric: LCP, FCP, Speed Index.
Cause: the first load has 149 requests. 50 are scripts and 73 are images.
Solution: load less up front, lazy-load lower sections, and avoid repeated icon/logo requests.

## 7. Code is the main thing being downloaded

Affects users: the browser downloads and runs a lot of code before the page feels ready.
Metric: LCP, TBT, Time to Interactive.
Cause: JS + CSS is 1.46 MB transferred and 4.67 MB total size. That is about 78% of everything downloaded.
Solution: split the bundles, remove unused JS, defer non-critical scripts, and keep only critical CSS early.

## 8. Third-party scripts are too heavy

Affects users: tracking and analytics compete with the actual page content.
Metric: TBT, LCP, Speed Index.
Cause: third parties transfer 819.5 KB, about 44% of the first load. The biggest ones are Google Tag Manager, Google Ads, Facebook, and Sentry replay.
Solution: remove tags that are not needed and delay the rest until after the page is usable.

## 9. Images are many, but not the biggest bytes

Affects users: the browser still has to schedule a lot of image requests, even if most are small.
Metric: LCP, Speed Index.
Cause: images are 73 requests, but only 173.3 KB transferred. Most are small SVG logos/icons served with gzip.
Solution: keep the hero image optimized, but reduce repeated SVG/icon requests where possible.

## 10. The refresh does not get cheaper

Affects users: returning users still download basically the same amount again.
Metric: repeat-load transferred bytes, LCP, FCP, Speed Index.
Cause: the first load transfers 1.87 MB, and the refresh also transfers 1.87 MB. There are no `304` responses and no `0 B` cached resources, so the cache reduction is basically 0%.
Solution: make sure static assets can be reused from cache and check why the refresh is sending `Cache-Control: no-cache` for almost everything.

## Still to do

Run the same check on the other pages, especially faculty, because that one may have a
different image problem.
