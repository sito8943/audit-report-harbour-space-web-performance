# Baseline Results

## Summary

The homepage is a mixed bag. On desktop it's actually fast, but on mobile it's slow: the
main content takes way too long to show up. The server also takes a while to answer on
mobile. Layout moves around for real users, even though the lab test says it doesn't. So
the real problems are on mobile and in the real-user data, not on a fast desktop.

(I originally wrote here that the page also "keeps working in the background for a
while" — that came from my first Lighthouse run, where blocking time was 720 ms. The
newer runs put TBT at 200 ms, so I've corrected it: the page isn't busy, the content is
just late.)

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

For this part I went back to the live homepage with the Coverage tool and recorded a
scroll through the page, on the same throttled mobile setup as before. One note on the
numbers: Coverage reports uncompressed bytes, so everything here looks bigger than the
transfer sizes from the network section. Same files, different unit.

### Coverage — CSS

First question was whether the site extracts critical CSS. It doesn't — if anything it
does the opposite. The HTML shows up with about 148 KB of inline `<style>` in the head,
and 128 KB of that is one giant `data-styled` block, which is styled-components dumping
everything it knows at render time. Critical CSS would mean inlining just what the first
screen needs and loading the rest later. This is the whole site's CSS pasted into every
HTML response, it's a big part of why the document is 433 KB, and since it's inline none
of it gets cached — you download all the CSS again on every page.

As for how much is unused: of the CSS Coverage can track, about **96% of the first-party
CSS is never used during load** (roughly 89 of 93 KB). Where it comes from:

- Next.js CSS chunks (`c2f78067…css` at 31 KB, `e1bd08c0…css` at 19 KB, plus three
  smaller ones) that are **100% unused**. As far as I can tell these are styles for
  routes that aren't the homepage, loaded anyway.
- The inline styled blocks themselves — a lot of it is for components that only show up
  when you interact with something, or never mount on mobile at all.
- The cookie banner's stylesheet from JSDelivr, 94% unused. It ships a whole theming
  framework to draw one banner.

What I'd fix here: stop loading route CSS the homepage doesn't use, and turn the 128 KB
inline dump into actual critical CSS, with the rest in a normal cacheable file.

### Coverage — JS

**68% of the site's own JavaScript is unused during load** — 1.30 MB out of 1.92 MB
uncompressed. Third parties add another ~117 KB of unused code on top. The list is
topped by one file: `8f29162a…js`, where **404 KB of 454 KB (89%) never runs**. That
single chunk wastes more bytes than a lot of complete pages. After it comes
`f4eb6c19…js` (120 of 200 KB unused), Sentry's `replay.min.js` (81 of 146 KB, and
remember this one loads before consent for everyone), and then a tail of chunks that
are all 85–90% unused, which looks like shared bundles carrying components "just in
case".

One thing that might read like a contradiction: the bundle section above says the
biggest chunk is 316 KB minified and 51% unused. Both are true — that measurement came
from an earlier run, the site redeployed in between (all the chunk hashes changed), and
Coverage counts uncompressed bytes. So the numbers moved, but the story didn't: the
heaviest shared chunk is mostly dead weight for the homepage, and in the newer build
it's actually worse.

The fix I'd start with is obvious: that 89%-unused chunk is the best single target on
the whole site. Split it or tree-shake it and the JS numbers change for real.

### Frames — load, scroll, interaction

I recorded a continuous scroll through the whole page (300 frames, 4× CPU) and watched
the frame times.

Scrolling is mostly fine — 19 ms per frame on average, so around 52 fps — but **6 frames
dropped hard**, all above 50 ms, and the worst one took **382 ms**. That's a visible
hitch of a third of a second while scrolling a marketing page. The cause is sitting
right next to them in the recording: two long tasks (108 ms and 59 ms) firing in the
middle of the scroll. That's the lazy-mounted sections hydrating and styled-components
generating their CSS right when they scroll into view, on the same thread that's trying
to produce frames.

During load itself I only caught 3 long tasks (~180 ms total) in this headless run — the
full PSI run with all the third parties is much worse (720 ms of TBT), but load jank is
expected; that's not the surprise here.

Are the dropped frames excessive? Not really, 6 out of 300. Unexpected? Yes — there's no
reason scrolling a static page should ever cost 382 ms. So what I'd correct: move the
hydration and style-generation work off the scroll path — schedule it on idle, use
smaller lazy boundaries, or pre-generate the CSS — so scrolling never pays for it.

### Layers and animations

I went through every element's computed style looking for layer-forcing properties, and
through the CSS for the classic hacks.

`will-change` shows up on 14 elements. Three are tooltips using `will-change: transform,
opacity`, which is fair. The other **eleven are the same styled component
(`sc-7ac6c0ac-5`) repeated, declaring `will-change: transition, opacity`** — and
`transition` isn't something you can animate, so half of that value does nothing. It
still costs though: each one forces a compositor layer that never goes away, because a
`will-change` written in the stylesheet is permanently on.

The `translate3d(0,0,0)` trick is in the CSS too, but it's not theirs — it comes from
Swiper, the carousel library, which still ships the old forced-layer hack on
`.swiper-wrapper` and its pagination bullets. In my run no element actually computed to
a 3D transform at rest (the carousels weren't mounted on mobile), so this only kicks in
when a carousel is on screen. There are also 14 `position: fixed/sticky` elements
(header, cookie bar, chat button…), each a compositing candidate of its own.

The animations I could actually inspect — tooltip fades, the swiper transitions — run on
transform and opacity, so they're composition-driven, which is the right kind. I didn't
find anything animating layout properties like `top` or `width`.

Two things I couldn't check headless: the real layer count in the Layers panel, and
whether anything feels janky on first frame on an actual phone. That needs a manual
DevTools pass (Layers panel plus paint flashing), which is still on my list.

The correction for this set: fix or delete the `will-change: transition, opacity` on
those eleven elements. As written it's a typo that keeps eleven layers alive forever.
If they meant `transform`, write `transform` — and if the element isn't animating most
of the time, it shouldn't declare `will-change` in the stylesheet at all.

## Rendering strategies

To figure out how each page gets rendered I used the `__NEXT_DATA__` object that Next.js
leaves in every response — if it says `gsp: true` the page came out of `getStaticProps`,
and `gssp: true` means it was rendered on the server per request — plus the
`cache-control` and `x-nextjs-cache` headers.

### What's used where

I checked the homepage, the faculty list, a faculty profile, a course page, articles,
about, admissions and the schedule, and they all came back the same way: `gsp: true`,
`s-maxage=60, stale-while-revalidate`, and `x-nextjs-cache: HIT` or `STALE`. So the
whole public site is **static generation with incremental regeneration (ISR)**: the
HTML is pre-built, served from a cache, and rebuilt in the background at most once a
minute.

The one exception is `/account`, which reports `gssp: true` — real server-side
rendering on every request. Makes sense, it's the only page that's personalized. And if
you hit a URL that doesn't exist you get a prerendered static 404, so bad slugs don't
cost a render either.

One header stood out though: most pages are allowed to serve stale content for up to an
hour (`stale-while-revalidate=3600`), but the course pages say
`stale-while-revalidate=31535940` — that's a *year*. I'll come back to that.

### How this affects users, and the tradeoffs

This is honestly the good half of the site's performance story. Nobody waits for a CMS
query — the HTML already exists when you ask for it — so the server side of TTFB is
fast, the content is crawlable, and an edit in the CMS shows up within a minute or so
without redeploying anything. The usual ISR tradeoff is that a visitor can get content
that's up to a minute old, and for a university marketing site that just doesn't matter.

I do have to square this with finding 5, where I call the server slow on mobile —
because the field data really does show a 1.3 s TTFB there. Having looked at both, I
don't think those contradict each other: the same setup answers desktop users in 0.8 s,
so the extra half second on mobile is mostly the cell network's round trip, not the
server thinking. Pre-rendered pages can't fix a slow radio link. I'm noting it because
an earlier draft just said "TTFB is fast" flat out, and next to the field numbers that
looked wrong.

The tradeoff they're actually paying is a different one: the static HTML is so heavy
that the benefit of pre-rendering gets buried. Every response carries the ~148 KB
styled-components dump from the coverage section, plus the whole page content *twice* —
once as HTML and once again as the `__NEXT_DATA__` JSON that React uses to hydrate. On
the homepage that JSON is 61 KB. On `/computer-science` it's **204 KB out of a 305 KB
document** — two thirds of the "static" HTML is actually the hydration payload. The
phone still has to download all of that and then run 1.4 MB of JavaScript before the
page acts like a page. That's how a pre-rendered site ends up with an 8.2 s mobile LCP
anyway.

### Is it the right choice? Is it the best choice?

Right choice, yes. SSG + ISR is exactly what I'd pick for a content site fed by a CMS,
and doing SSR only on `/account` is also the correct call. I wouldn't change the
strategy on any page — but I wouldn't call it the best choice *as implemented*, and
that's where my two corrective findings are:

1. **The hydration payload is defeating the static rendering.** Shipping 204 KB of
   `__NEXT_DATA__` on a course page means the whole CMS response goes to the client
   even though the same content is already in the HTML above it. Trimming those props
   down to what the client actually needs after hydration could cut the course-page
   document roughly in half, which goes straight into FCP and LCP on mobile.
2. **The stale windows look like an accident.** Regular pages can serve stale for an
   hour; the course pages — the ones with deadlines, dates and prices on them — can
   serve stale for a year. If that's intentional it's backwards, because the most
   time-sensitive pages got the loosest window. My guess is someone computed "one year
   minus the 60 s" once and it stuck. Either way the fix is to set the revalidation
   windows on purpose: short for courses and schedules, long for evergreen pages like
   `/about`.
