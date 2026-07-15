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
