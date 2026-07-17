# Baseline Results

## Summary

The homepage is a mixed bag. On desktop it's actually fast, but on mobile it's slow: the
main content takes too long to show up and the page keeps working in the background for a
while. The server is a bit slow to answer on mobile too. Layout also moves around for real
users, even though the lab test says it doesn't. So the real problems are on mobile and in
the real-user data, not on a fast desktop.

## Core Web Vitals — Field Data

This is the real-user data from Chrome (last 28 days). I checked mobile and desktop because
they're pretty different. Both fail the assessment, but for different reasons.

### Mobile — Failed

- Largest Contentful Paint (LCP): **3.4 s** — needs improvement
- Interaction to Next Paint (INP): **218 ms** — needs improvement
- Cumulative Layout Shift (CLS): **0.11** — needs improvement
- First Contentful Paint (FCP): **1.9 s** — needs improvement
- Time to First Byte (TTFB): **1.3 s** — needs improvement

### Desktop — Failed

- Largest Contentful Paint (LCP): **2.1 s** — good
- Interaction to Next Paint (INP): **94 ms** — good
- Cumulative Layout Shift (CLS): **0.13** — needs improvement
- First Contentful Paint (FCP): **1.2 s** — good
- Time to First Byte (TTFB): **0.8 s** — needs improvement

On desktop almost everything is green, but it still fails because of CLS (0.13). On mobile
almost everything is orange.

## Lighthouse Lab Test

The lab run shows the same split — desktop is good, mobile is not.

I ran the mobile column with the throttling we used in class: Slow 4G on the network (around
1.6 Mbps down, 750 Kbps up, 150 ms round trip) plus a 4× slower CPU, emulating a mid-range
phone. Desktop is the normal desktop profile with no throttling. So the two aren't the same
test — mobile is handicapped on purpose to match what a phone on a cell connection goes
through, which is why I trust the mobile number more for real users.

### Metrics

|                          | Mobile | Desktop |
| ------------------------ | ------ | ------- |
| First Contentful Paint   | 2.7 s  | 0.6 s   |
| Largest Contentful Paint | 8.2 s  | 1.7 s   |
| Total Blocking Time      | 200 ms | 70 ms   |
| Cumulative Layout Shift  | 0      | 0       |
| Speed Index              | 6.4 s  | 1.1 s   |

Desktop performance score is **92** (green). On mobile, LCP (8.2 s) and Speed Index
(6.4 s) are both red, which is what drags the mobile score down. My earlier Lighthouse run
on the homepage gave Performance **51**, Accessibility **93**, Best Practices **77**, SEO
**92**, and the mobile metrics here are the same shape (LCP and Speed Index in the red).

### Where the score comes from

Lighthouse doesn't weigh the five metrics equally, so it's worth spelling out which ones
actually sink the mobile score:

| Metric | Weight | Mobile result | Threshold (good / poor) | Status |
|---|---|---|---|---|
| Total Blocking Time | 30% | 200 ms | ≤200 ms / >600 ms | good (right at the edge) |
| Largest Contentful Paint | 25% | 8.2 s | ≤2.5 s / >4 s | poor (way past) |
| Cumulative Layout Shift | 25% | 0 | ≤0.1 / >0.25 | good |
| First Contentful Paint | 10% | 2.7 s | ≤1.8 s / >3 s | needs improvement |
| Speed Index | 10% | 6.4 s | ≤3.4 s / >5.8 s | poor |

CLS and TBT carry 55% of the score and both pass, which is what keeps the mobile score
from being a disaster. The damage is concentrated in LCP — a quarter of the score, and
it's more than 3× past the "poor" line — plus Speed Index. In other words: the page isn't
janky and it isn't blocked, it's just late. That matches the field data too, where LCP is
the metric real users fail hardest.

One thing to notice: the lab says CLS is **0**, but the field data says CLS is **0.11 /
0.13**. So real users see the page move even though a clean lab run doesn't. That's worth
keeping in mind.

## What this means

Desktop is basically fine (score 92, LCP 1.7 s). The pain is on mobile: LCP is 8.2 s in the
lab and 3.4 s for real users, Speed Index is 6.4 s, taps lag a bit (INP 218 ms), and the
server takes 1.3 s to answer (TTFB). Blocking time is actually low (200 ms), so it's not
that the main thread is jammed — it's that a lot is being downloaded and the content shows
up late. Layout also shifts for real users. So the work is: get the mobile content to paint
sooner, fix the layout shift, and speed up the server response on mobile.

## Network at `/`

I looked at the homepage HAR (`harbour.space-1.har`). It has two loads in it: the first
visit and a refresh, so I can compare them.

### First load

The first load makes **149 requests**.

It transfers **1.78 MB**, but the total resource size is **5.56 MB**. So compression is
doing a lot of work here — the browser downloads about **67.9% less** than the full size.

### What the bytes are

This page is almost all code.

- **JavaScript**: 52 requests, **1.37 MB** downloaded, 4.36 MB full size — **77%** of
  everything downloaded.
- **Images**: 73 requests but only **0.17 MB** (9.3%). The biggest image is just 29.5 KB,
  so images are not the problem here.
- **Fonts**: 4 requests, 0.11 MB.
- **CSS**: 6 requests, 0.02 MB downloaded, 0.10 MB full size.

JS and CSS together are **58 requests, 1.39 MB** downloaded and **4.46 MB** full size —
about **78%** of the download and **80%** of the full size. So the page is heavy because of
code, not because of images.

### Third parties

Counting Harbour.Space and its asset CDN as the site itself, third parties are **90
requests and 0.92 MB** — about **51%** of the download. So half of what you download isn't
even the site's own stuff.

### Refresh

The refresh is the disappointing part. It makes **151 requests** and transfers **1.78 MB**
again — basically **+0.1%**, so it's not any lighter. There are **no `304` responses and
nothing served from cache**, so the browser downloads almost everything a second time
instead of reusing it.

### Compression notes

The code compresses well. Most files use `gzip`, a few use `zstd` or `br`, and that's why
the download is 68% smaller than the full size. The requests with no compression are mostly
images and fonts, which are already compressed formats, so they don't shrink again over the
network. For those, caching matters more than gzip.

## Bundle outputs at `/`

The HAR says how many bytes came down, not what's inside them, so for this part I went to
the live page: pulled the homepage HTML, downloaded the chunks, and ran the Lighthouse
coverage audits (unused JS/CSS, render-blocking, third-party summary) on the same
throttled mobile setup.

### JavaScript

The site is a Next.js app (built with Turbopack), and it's bundled the way Next does it:
the homepage HTML loads **31 hashed chunks**, all with `defer`, about **458 KB compressed**
between them. That's route splitting — each page gets its own chunk plus shared ones — so
the *architecture* is right, and the files are served with
`cache-control: max-age=31536000, immutable` and hashed names, which is exactly what you
want. This isn't the AP News "one bundle for the whole site" situation.

The problem is what's inside. The biggest chunk is **100 KB compressed (316 KB minified)**
— it carries react-dom and the shared framework code — and **51% of it is unused** on the
homepage. So even with route splitting, half of the heaviest file is dead weight for this
page. And the page still ships ~30 more chunks after it; the HAR counted 52 JS requests
and 1.37 MB once everything (including third parties) loads.

Source maps: **none**. No `sourceMappingURL` in the chunks and the `.map` URLs return 404.
Should they exist? Here it's worse than a yes — the site runs **Sentry** for error
tracking, and without source maps every error Sentry catches points into minified chunk
names like `6f9c1591e392ddd4.js`. They're paying for error reporting and then making the
reports unreadable.

### CSS

There's almost no CSS file to talk about: **one stylesheet, 6.5 KB (1.8 KB compressed)**.
That's not because the site has no styles — it's because the styling is
**styled-components** (the HTML has 217 `data-styled` markers), so the CSS lives inside
the JavaScript and gets generated at runtime. Bundling-wise that means the "CSS bundle"
question and the "too much JS" question are the same question here: every style costs
main-thread JavaScript work on the phone's slow CPU instead of being a plain file the
browser parses once. Of the styles it does emit, Lighthouse still flags **55% as unused**
(~13 KB of inline styled blocks). Is it the right decision? For a mostly-static marketing
site, runtime CSS-in-JS is paying an interactivity tax without getting much back — but
migrating away is a rewrite, not a fix.

### Images

Mostly right. Raster images go through Next's optimizer (`/_next/image`), which serves
**AVIF/WebP**, resizes to the requested width (q=75), and the `<img>` tags have `srcset`
with a dozen widths, `sizes`, and lazy loading. I verified the optimizer output: a
professor photo comes back as 8 KB AVIF instead of the 82 KB original PNG. So: multiple
sizes and formats, yes, and yes it's the right decision.

Two leaks though: **15 raster PNGs skip the optimizer** and load straight from the
DigitalOcean Spaces bucket at full size, and the bucket itself is public — the original
full-resolution files sit one URL away (that's how the optimizer reads them). Users mostly
don't pay for it, but the bypassed PNGs do ship unoptimized.

### Third parties

The initial HTML contains **zero third-party script tags** — everything is injected at
runtime by the app. What actually shows up in a fresh lab run, before touching the consent
banner: **Sentry** (error tracking — including `replay.min.js`, the session-recording
module, 48 KB with 51% unused), **CookieChimp** (the consent manager itself), and fonts
CDN bits from JSDelivr. Blocking cost pre-consent is small (~30 ms).

The heavy hitters from the HAR — Google Tag Manager, Google Ads, Facebook — arrive after
consent, which is why the field experience is heavier than a clean lab run: **90 requests
and 0.92 MB (51% of the download)** once the tags fire. So the loading strategy is
"delayed behind consent", which is the right shape, with one exception: Sentry loads its
session-replay recorder up front for everyone, before any consent, on a marketing page.
That one seems both unnecessary (who replays anonymous homepage visits?) and the single
biggest third-party blocker in the lab.

## Coverage, frames, and layers at `/`

For this part I measured the live homepage with the DevTools Coverage API and a scripted
scroll test, on the same mobile emulation (390px viewport, 4× CPU throttle). Byte numbers
here are **uncompressed** — that's what Coverage reports — so they're bigger than the
transfer sizes in the network section.

### Coverage — CSS

**Does it extract critical CSS? No — it does the opposite.** The server-rendered HTML
arrives with **~148 KB of inline `<style>` in the head**, and 128 KB of that is a single
`data-styled` block: styled-components dumping *every* style it knows at SSR time. That's
not critical CSS extraction (inline only what the first screen needs, load the rest
later); it's inlining the whole page's CSS into every HTML response. It's part of why the
document is 433 KB, and none of it can be cached — you re-download all your CSS with
every page view.

**How much unused CSS, and where from?** Of the CSS the Coverage tab can track, **~96% of
first-party CSS is unused at load** (~89 of 93 KB). The sources, worst first:

- Next.js CSS chunks (`c2f78067…css` 31 KB, `e1bd08c0…css` 19 KB, and three more) —
  **100% unused**. These look like styles for routes/components that aren't on the
  homepage at all, but get loaded anyway.
- The inline styled-components blocks — mostly unused at load because they include styles
  for components that only appear on interaction or never mount on mobile.
- Third-party: the cookie-consent stylesheet from JSDelivr is **94% unused** (29.5 of
  31.5 KB) — a whole theming framework for one banner.

**Corrective finding:** stop shipping 100%-unused route CSS chunks on the homepage, and
replace the 128 KB full-dump inline block with actual critical CSS (or at least move the
non-critical part to a cacheable file).

### Coverage — JS

**How much unused JS, and where from?** **68% of first-party JavaScript is unused at
load** — 1.30 MB of 1.92 MB uncompressed. Third-party adds another 117 KB unused (58%).
The top offenders:

- `8f29162a…js` — **404 KB unused of 454 KB (89%)**. One first-party chunk, alone, wastes
  more bytes than most whole pages.
- `f4eb6c19…js` — 120 KB unused of 200 KB (60%).
- Sentry `replay.min.js` — 81 KB unused of 146 KB (55%), and as noted above it loads
  pre-consent for everyone.
- A tail of ~85–90%-unused chunks (`0a9ce183`, `a351d30b`, `ce7faa21`, `453c3759`…),
  which smells like shared chunks that bundle many components "just in case".

**Corrective finding:** the 89%-unused 454 KB chunk is the single best target on the whole
site — split it or tree-shake it, and the JS story changes materially.

### Frames — load, scroll, interaction

I drove a continuous scroll through the full page (300 frames sampled, 4× CPU) and
watched frame times and long tasks.

- **Scrolling:** average frame **19 ms** (~52 fps) — mostly smooth — but **6 dropped
  frames**, all of them severe (>50 ms), with the **worst at 382 ms**: a visible hitch of
  a third of a second. The cause shows up right next to them: **two long tasks (108 ms
  and 59 ms)** firing mid-scroll — that's lazy-mounted sections hydrating and
  styled-components generating their CSS the moment they scroll into view, on the main
  thread, while it's also trying to render frames.
- **Load:** 3 long tasks totaling ~180 ms in this headless run (the full PSI run with
  third parties showed much more — 720 ms TBT). Expected during load; not the problem.
- **Are they excessive or unexpected?** Excessive, no — 6 of 300 frames. Unexpected, yes:
  this is a static marketing page; there is no reason scrolling should ever cost 382 ms.
  A frame budget blowout of 20× on scroll is work that shouldn't be on the main thread at
  that moment.

**Corrective finding:** hydration/style-generation work triggered by scroll should be
scheduled off the interaction path (e.g. `requestIdleCallback`, smaller lazy boundaries,
or pre-generating the CSS) so scrolling never pays for it.

### Layers and animations

I scanned every element's computed style for layer-forcing properties, and the CSS for
the classic hacks:

- **`will-change` on 14 elements.** Three are tooltips with `will-change: transform,
  opacity` — defensible. But **eleven** are one repeated styled component
  (`sc-7ac6c0ac-5`) declaring **`will-change: transition, opacity`** — and `transition`
  is not an animatable property, so that value is half meaningless. Each of those still
  forces a compositor layer that sits there permanently, because `will-change` in a
  stylesheet never turns off.
- **`translate3d(0,0,0)` hacks: present in the CSS**, coming from **Swiper** (the
  carousel library) — `.swiper-wrapper { transform: translate3d(0,0,0) }` is the
  old-school forced-layer trick, kept alive by the library, plus a `translate3d` on the
  pagination bullets. At rest on mobile no element computes to a 3D transform (the
  carousels weren't mounted in my run), so these fire only when a carousel is on screen.
- **14 `position: fixed/sticky` elements** (header, cookie bar, chat button…), each of
  which is also its own compositing candidate.
- The animations I could inspect (tooltip fades, swiper transitions) are driven by
  **transform/opacity — composition-triggered, the right kind**. I didn't find
  layout-driven animations (`top/left/width`) in the shipped CSS.

What I couldn't verify headless: the actual layer count in the Layers panel and whether
any animation *feels* janky first-frame on a real device — that needs a manual DevTools
pass (Layers panel + Rendering → Paint flashing), which is the remaining to-do here.

**Corrective finding:** fix or remove the `will-change: transition, opacity` on the
eleven `sc-7ac6c0ac-5` elements — as written it's a typo'd, permanently-on layer
forcer × 11. If the intent was `transform`, say `transform`; if the element isn't
animating most of the time, don't declare `will-change` in the stylesheet at all.

## Rendering strategies

To figure out how each page is rendered I looked at the `__NEXT_DATA__` object Next.js
embeds in every response (`gsp: true` means the page was built with `getStaticProps`,
`gssp: true` means it was rendered per-request with `getServerSideProps`), plus the
`cache-control` and `x-nextjs-cache` headers the server sends back.

### What's used where

Every page I probed tells the same story — the whole marketing site is **static
generation with incremental regeneration (ISR)**:

| Page | Route | Strategy | Cache headers |
|---|---|---|---|
| `/` (homepage) | `/` | SSG + ISR | `s-maxage=60, stale-while-revalidate=3600`, `x-nextjs-cache: HIT` |
| `/faculty`, `/faculty/[id]` | static + dynamic | SSG + ISR | same, often `STALE` |
| `/computer-science` (courses) | `/[campus]` | SSG + ISR | `s-maxage=60, stale-while-revalidate=31535940` (!) |
| `/articles`, `/about`, `/admissions`, `/schedule` | various | SSG + ISR | `s-maxage=60, stale-while-revalidate=3600` |
| `/account` | `/account/[[...slug]]` | **SSR** (`getServerSideProps`) | per-request |
| unknown slugs | `/404` | prerendered static 404 | — |

So: pre-rendered static HTML for everything public, regenerated in the background at
most every 60 seconds, served from cache (`HIT`/`STALE`); and real server-side rendering
only for `/account`, the one page that's actually personalized.

### How this affects users, and the tradeoffs

For users this is the good half of the site's performance story: TTFB is fast because
nobody waits for a CMS query — the HTML is already built (the field TTFB issues on
mobile are network, not rendering). Content is crawlable, and a CMS edit shows up within
about a minute without a redeploy. The classic ISR tradeoff — a visitor can get up to
60-seconds-stale content — is irrelevant for a university marketing site.

The real tradeoff they're paying is different: **the static HTML is enormous, so the
benefit of pre-rendering gets buried**. Each response carries the full styled-components
CSS dump (~148 KB inline) plus the entire page content *twice* — once as HTML and once
as the `__NEXT_DATA__` JSON used for hydration. On the homepage that JSON is 61 KB; on
`/computer-science` it's **204 KB of a 305 KB document — two thirds of the HTML is the
hydration payload**. The page is "static", but the phone still has to download all of
it, then parse and hydrate it with 1.4 MB of JavaScript before it behaves like a page.
That's how a pre-rendered site still ends up with an 8.2 s mobile LCP.

### Is it the right choice? The best choice?

Right choice: **yes**. SSG+ISR is exactly what a content site with a CMS should use, and
SSR for the personalized `/account` page is also correct. I wouldn't change the
strategy on any page.

Best choice: **not as implemented**. Two corrective findings:

1. **The hydration payload defeats the static rendering.** 204 KB of `__NEXT_DATA__` on
   a course page means the CMS response is shipped wholesale to the client, even though
   the HTML already contains it. Trim the props to what the client actually needs after
   hydration (or move page content out of props entirely) — that alone could cut the
   course-page document by roughly half and directly helps FCP/LCP on mobile.
2. **The stale-while-revalidate windows are inconsistent and look accidental.** Most
   pages can serve stale for 1 hour (`3600`), but course pages — the ones with
   deadlines, dates and prices — can serve stale for **a year** (`31535940`, i.e.
   365 days minus the 60 s). If that number is intentional, it's backwards (the most
   time-sensitive pages have the loosest window); more likely someone computed
   "1 year − s-maxage" once and it stuck. Pick explicit, per-content revalidation
   windows: short for course/schedule pages, long for evergreen pages like `/about`.
