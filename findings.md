# Performance Findings

These are from the homepage. The big thing I noticed is that desktop is fine but mobile is
slow, and the real-user (field) data doesn't match the lab in a couple of places — the lab
says layout is stable, but real users actually see it move. So I'm going by the field data
where they disagree, because that's what people really get. The mobile lab numbers I lean on
were taken on a throttled phone (Slow 4G, 4× slower CPU — the setup from class), not a fast
laptop, so they match what a phone actually goes through.

I tried to keep each of these as its own separate thing. When a few problems really just
show up as the same metric, I put them together instead of listing them twice.

---

## Things to fix

## 1. The main content shows up too late on mobile

Affects users: on a phone the hero you came to see only appears after a long wait, so the
page feels slow. On desktop it's fine.
Metric: LCP (3.4 s for real users on mobile, 8.2 s in the mobile lab), Speed Index (6.4 s
on mobile).
Cause: on mobile the content is stuck waiting behind all the JavaScript loading and
running before it can paint. Desktop is fast enough that this doesn't show up.
Solution: preload/prioritize the hero (fetchpriority=high) and cut the work that runs
before it paints, especially for mobile.

## 2. The page is basically all JavaScript

Affects users: you download a lot of code just to read a marketing page, which is wasteful
on mobile or slow wifi.
Metric: LCP, Time to Interactive.
Cause: JavaScript is 52 requests, 1.37 MB downloaded and 4.36 MB of full size — 77% of
everything downloaded, and 80% of the full size once you add CSS. About 405 KiB of that JS
is never even used.
Solution: drop the unused code, split the bundles, and defer whatever isn't needed for the
first render.

## 3. The layout moves around for real users

Affects users: things shift while the page loads, so you go to tap something and it jumps.
Metric: CLS (0.11 on mobile, 0.13 on desktop, both "needs improvement").
Cause: the lab run shows CLS 0, but the field data shows real users seeing layout shift.
That usually means something without a reserved size (an image, an ad, a font swap, or a
cookie banner) pushes the content once it loads.
Solution: give images and embedded/ad boxes a fixed size up front, and load fonts so they
don't reflow the text.

## 4. Taps lag a bit on mobile

Affects users: on a phone the page doesn't react instantly when you tap or scroll.
Metric: INP (218 ms on mobile — needs improvement; desktop is fine at 94 ms).
Cause: even though blocking time in the lab is low, real mobile users hit small delays,
probably from scripts doing work while they interact.
Solution: break up long tasks and keep non-critical JavaScript off the main thread while
the user is interacting.

## 5. The server is slow to answer on mobile

Affects users: on mobile you wait longer before anything can even start loading.
Metric: TTFB (1.3 s on mobile, 0.8 s on desktop — both needs improvement), FCP.
Cause: the first byte from the server takes over a second on real mobile connections, which
pushes everything after it back.
Solution: check caching/CDN at the edge for mobile, and make sure the document isn't waiting
on server work before it starts sending.

## 6. The cache doesn't help on a second visit

Affects users: coming back to the page doesn't feel any cheaper — it downloads almost
everything again.
Metric: repeat-load bytes, LCP, FCP.
Cause: the refresh transfers 1.78 MB, the same as the first load (+0.1%). There are no
`304` responses and nothing served from cache, so the browser re-downloads it all.
Solution: let static files be reused from cache with proper cache headers instead of
fetching them again every visit.

## 7. Half the download is third-party

Affects users: analytics and other embedded scripts compete with the page while you're
waiting for it.
Metric: TBT, LCP.
Cause: third parties are 90 requests and 0.92 MB — about 51% of the download. The heavy ones
are things like Google Tag Manager, Google Ads, Facebook, and Sentry.
Solution: keep only what's needed and load the rest lazily, after the page is usable.

---

## Things it does well

## 8. Desktop is genuinely fast

On a laptop the site is quick: performance score 92, LCP 1.7 s, FCP 0.6 s, blocking time
70 ms, Speed Index 1.1 s. So the code and structure can be fast — the mobile experience just
needs to catch up to it.

## 9. Images are small and not a problem

There are a lot of images (73 requests) but together they're only 0.17 MB, and the biggest
one is just 29.5 KB. So unlike a lot of sites, images aren't dragging this page down — the
weight is all in the JavaScript.

## 10. Good compression on the code

The text compression is doing its job: the page downloads 68% less than its full size
because the JS and CSS are compressed with gzip, zstd, and br. So that part is already fine
— what's left to fix is the amount of JavaScript and the caching, not the compression.
