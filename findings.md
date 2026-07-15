# Performance Findings

These are from the homepage. The big thing I noticed is that desktop is fine but mobile is
slow, and the real-user (field) data doesn't match the lab in a couple of places — the lab
says layout is stable, but real users actually see it move. So I'm going by the field data
where they disagree, because that's what people really get. The mobile lab numbers I lean on
were taken on a throttled phone (Slow 4G, 4× slower CPU — the setup from class), not a fast
laptop, so they match what a phone actually goes through.

I tried to keep each of these as its own separate thing. When a few problems really just
show up as the same metric, I put them together instead of listing them twice.

I also scored each fix so there's an actual order to work in. I'm using WSJF — the system
and the scales are in `prioritization.md`, but the short version is: add up how much the
fix is worth, how much it hurts to wait, and how much it stops things getting worse, then
divide by how big the job is. Highest score goes first. That puts the order at
6, 1, 3, 10, 7, 11, 5, 8, 2, 4, 9 — which surprised me a little (the cache fix wins over
the big LCP one), but it makes sense once you see the effort side.

Findings 8–11 came from digging into the bundle outputs themselves — what's inside the
files, not just how many bytes they are. That data is in the "Bundle outputs" section of
`baseline.md`.

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
Priority: the hero is the whole point of the page and it's the worst number in the audit,
so value is an 8, and time criticality is an 8 too — a red LCP is hurting ranking and
first impressions right now, not someday. Risk reduction 3. That's a cost of delay of 19,
and the fix itself is one focused change (job size 3), so WSJF = 19 / 3 = 6.33.

## 2. The page is basically all JavaScript

Affects users: you download a lot of code just to read a marketing page, which is wasteful
on mobile or slow wifi.
Metric: LCP, Time to Interactive.
Cause: JavaScript is 52 requests, 1.37 MB downloaded and 4.36 MB of full size — 77% of
everything downloaded, and 80% of the full size once you add CSS. About 405 KiB of that JS
is never even used.
Solution: drop the unused code, split the bundles, and defer whatever isn't needed for the
first render.
Priority: this is the root cause behind most of the mobile slowness, so value is an 8, and
fixing it makes findings 1, 4 and 7 easier or unnecessary, so risk reduction is a 5. Time
criticality 5 — the cost is real but it mostly shows up through the other findings. Cost of
delay 18, which is huge. But splitting bundles and hunting dead code across the whole site
is a genuinely big project (job size 8), so WSJF = 18 / 8 = 2.25. It still has to happen —
it just shouldn't block the cheap wins.

## 3. The layout moves around for real users

Affects users: things shift while the page loads, so you go to tap something and it jumps.
Metric: CLS (0.11 on mobile, 0.13 on desktop, both "needs improvement").
Cause: the lab run shows CLS 0, but the field data shows real users seeing layout shift.
That usually means something without a reserved size (an image, an ad, a font swap, or a
cookie banner) pushes the content once it loads.
Solution: give images and embedded/ad boxes a fixed size up front, and load fonts so they
don't reflow the text.
Priority: jumping content is genuinely annoying and it happens on both mobile and desktop,
so value 5, and since a "needs improvement" CLS counts against Core Web Vitals today, time
criticality is a 5 as well. Risk reduction 2. Cost of delay 12, and reserving sizes and
fixing font loading are small contained changes (job size 2), so WSJF = 12 / 2 = 6.00.

## 4. Taps lag a bit on mobile

Affects users: on a phone the page doesn't react instantly when you tap or scroll.
Metric: INP (218 ms on mobile — needs improvement; desktop is fine at 94 ms).
Cause: even though blocking time in the lab is low, real mobile users hit small delays,
probably from scripts doing work while they interact.
Solution: break up long tasks and keep non-critical JavaScript off the main thread while
the user is interacting.
Priority: a 218 ms tap delay is noticeable but it's not what's driving people away, so
value 3, time criticality 3 (mild "needs improvement", and only on mobile), risk
reduction 2. Cost of delay 8. Finding and breaking up long tasks means digging into the
app code, and finding 2 would change all of this anyway — job size 5. So WSJF = 8 / 5 =
1.60, last on the list, and honestly that feels right.

## 5. The server is slow to answer on mobile

Affects users: on mobile you wait longer before anything can even start loading.
Metric: TTFB (1.3 s on mobile, 0.8 s on desktop — both needs improvement), FCP.
Cause: the first byte from the server takes over a second on real mobile connections, which
pushes everything after it back.
Solution: check caching/CDN at the edge for mobile, and make sure the document isn't waiting
on server work before it starts sending.
Priority: nothing can start before the first byte arrives, so that second of waiting
pushes every other metric back — value 5. Time criticality 3 (a steady ongoing cost, not
getting worse), risk reduction 2, so cost of delay is 10. The fix is mostly CDN and
edge-caching configuration, but it needs some investigation first to know where the time
goes (job size 3). WSJF = 10 / 3 = 3.33.

## 6. The cache doesn't help on a second visit

Affects users: coming back to the page doesn't feel any cheaper — it downloads almost
everything again.
Metric: repeat-load bytes, LCP, FCP.
Cause: the refresh transfers 1.78 MB, the same as the first load (+0.1%). There are no
`304` responses and nothing served from cache, so the browser re-downloads it all.
Solution: let static files be reused from cache with proper cache headers instead of
fetching them again every visit.
Priority: only repeat visitors feel this one, but they feel all 1.78 MB of it, so value 3,
time criticality 2 (a mild but constant waste), and risk reduction 3 — once caching works,
every other fix pays off double because returning visitors actually keep it. Cost of delay
8, which is modest. But the fix is cache headers — a config change, about a day of work
(job size 1) — so WSJF = 8 / 1 = 8.00 and it lands at the top of the list. Not because
it's the biggest problem, but because it's nearly free.

## 7. Half the download is third-party

Affects users: analytics and other embedded scripts compete with the page while you're
waiting for it.
Metric: TBT, LCP.
Cause: third parties are 90 requests and 0.92 MB — about 51% of the download. The heavy ones
are things like Google Tag Manager, Google Ads, Facebook, and Sentry.
Solution: keep only what's needed and load the rest lazily, after the page is usable.
Priority: half the download competing with the actual page is worth a 5 on value, and risk
reduction is a 5 too — third-party tags only ever accumulate, so if nobody prunes them now
this keeps growing. Time criticality 3 (a steady drag, not a cliff). Cost of delay 13, and
the work is mostly auditing tags and lazy-loading them — more coordination than deep code
changes (job size 3). WSJF = 13 / 3 = 4.33.

## 8. The biggest chunk is half dead weight

Affects users: the heaviest JavaScript file the page loads — the one everything else waits
on — is 51% unused on the homepage.
Metric: LCP, TBT (it's part of the pre-paint work from finding 1).
Cause: the site is Next.js with proper route splitting (31 hashed chunks, all defer — the
architecture is right), but the shared framework chunk is 100 KB compressed / 316 KB
minified, and half of it never runs on this page. Shared chunks accumulate whatever every
page might need.
Solution: audit what's in the shared chunk, move page-specific code out of it, and
dynamic-import the pieces (carousels, players, below-the-fold widgets) that don't need to
be in the first download.
Priority: the user value is real but partial — it's ~50 KB of the 1.37 MB total, so
value 3, time criticality 3, and risk reduction 3 since a leaner shared chunk makes
finding 2's bigger cleanup easier. Cost of delay 9. Untangling a shared chunk is a week of
careful work (job size 3), so WSJF = 9 / 3 = 3.00.

## 9. The CSS is hiding inside the JavaScript

Affects users: styles that could be a small file the browser reads once are instead
JavaScript that has to run on the phone's slow CPU before things look right.
Metric: TBT, INP (it's runtime work), LCP indirectly.
Cause: the site styles with styled-components — the HTML has 217 styled markers and the
only real CSS file is 6.5 KB. So the styling cost is buried in the JS bundles and paid at
runtime, and Lighthouse still flags 55% of what it emits as unused. For a mostly-static
marketing site, that's paying an interactivity tax without needing the interactivity.
Solution: extract the static styles to plain CSS at build time (or move to zero-runtime
styling); at minimum, stop the unused styled blocks from shipping.
Priority: value 3 — it would shave runtime work on every page, but it's not the biggest
pain. Time criticality 2, risk reduction 3 (it makes the JS diet in finding 2 smaller
too). Cost of delay 8. But changing the styling approach across the whole app is a
rewrite-sized job (job size 8), so WSJF = 8 / 8 = 1.00, last place. Honest answer: only do
this when something else forces a redesign anyway.

## 10. No source maps, and they're paying Sentry to catch errors they can't read

Affects users: not directly today — it hurts every future fix, because production errors
point into files named like `6f9c1591e392ddd4.js`.
Metric: none (debuggability).
Cause: no `sourceMappingURL` in any chunk and the `.map` URLs 404. The site runs Sentry,
so every error report it pays for lands in unreadable minified code.
Solution: upload source maps to Sentry at build time (Next.js does this with a flag and an
auth token). They don't even need to be public — Sentry-only uploads keep the code private
and readers never download a byte.
Priority: value 1 for users directly, time criticality 1, but risk reduction 3 — every
incident on this list gets found and fixed faster once errors are readable. Cost of
delay 5, and it's about a day of build config (job size 1), so WSJF = 5 / 1 = 5.00. Same
story as the cache finding: near the top because it's nearly free.

## 11. Sentry records everyone's visit before anyone consents

Affects users: the session-replay recorder (48 KB, half of it unused) loads up front for
every anonymous visitor, before the consent banner is even answered — on a marketing page
where there's nothing to replay.
Metric: TBT (it's the biggest third-party blocker in the lab run), plus a privacy smell.
Cause: everything else third-party correctly waits behind CookieChimp consent (GTM, Ads,
Facebook only fire after you accept — that part is done right). Sentry error tracking is
defensible up front, but the replay module isn't: it records the session of every visitor
who bounces off the homepage.
Solution: keep basic error tracking, load `replay` lazily and only where sessions matter
(the application/account flows), or sample it down hard.
Priority: value 3 (bytes and main-thread for literally every visitor, plus the consent
question), time criticality 2, risk reduction 3. Cost of delay 8, and it's a Sentry config
change (job size 2), so WSJF = 8 / 2 = 4.00.

---

## Things it does well

## 12. Desktop is genuinely fast

On a laptop the site is quick: performance score 92, LCP 1.7 s, FCP 0.6 s, blocking time
70 ms, Speed Index 1.1 s. So the code and structure can be fast — the mobile experience just
needs to catch up to it.

## 13. Images are small and the pipeline is done right

There are a lot of images (73 requests) but together they're only 0.17 MB, and the biggest
one is just 29.5 KB. And it's not luck: raster images go through Next's optimizer, which
serves AVIF/WebP resized to what the layout needs, with proper srcset and lazy loading — I
checked one professor photo and it comes back as 8 KB AVIF instead of the 82 KB original.
So unlike a lot of sites, images aren't dragging this page down — the weight is all in the
JavaScript. Two small leaks worth closing: 15 PNGs skip the optimizer and load full-size
straight from the storage bucket, and the bucket with the originals is public.

## 14. Good compression on the code

The text compression is doing its job: the page downloads 68% less than its full size
because the JS and CSS are compressed with gzip, zstd, and br. So that part is already fine
— what's left to fix is the amount of JavaScript and the caching, not the compression.
