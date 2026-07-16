# Baseline

## Core Web Vitals

I ran Lighthouse on the homepage to get a starting point. The page doesn't shift around
and it reacts fine once it's up, but the main content shows up way too late, so the
performance score ends up low.

## PageSpeed Insights at `/`

- **Performance**: 51
- **Accessibility**: 93
- **Best Practices**: 77
- **SEO**: 92

- **First Contentful Paint** 2.2 s
- **Largest Contentful Paint** 13.5 s
- **Total Blocking Time** 720 ms
- **Cumulative Layout Shift** 0
- **Speed Index** 5.3 s

## Performance breakdown

### Where the score comes from

Lighthouse weighs the five metrics like this, so it's clear which ones are sinking the 51:

| Metric | Weight | Result | Threshold (good / poor) | Status |
|---|---|---|---|---|
| Total Blocking Time | 30% | 720 ms | ≤200 ms / >600 ms | poor |
| Largest Contentful Paint | 25% | 13.5 s | ≤2.5 s / >4 s | poor (way past) |
| Cumulative Layout Shift | 25% | 0 | ≤0.1 / >0.25 | good |
| First Contentful Paint | 10% | 2.2 s | ≤1.8 s / >3 s | needs improvement |
| Speed Index | 10% | 5.3 s | ≤3.4 s / >5.8 s | needs improvement |

CLS is perfect and carries a quarter of the score — that's the only reason it's a 51 and
not way lower. LCP and TBT together are 55% of the score and both are in the red, so
that's where basically all the lost points are.

### Page weight

- Total transferred: ~1.8 MB
- JavaScript: ~1.48 MB (about 82% of the page)
- Unused JavaScript: ~405 KiB (code that gets downloaded but never runs)
- Everything else (HTML, CSS, images, fonts): the remaining ~0.3 MB

So the page isn't heavy because of media — it's heavy because of script.

### Main thread

- Main-thread work: ~4.2 s total
- Script evaluation (just booting up the JS): ~1.9 s of that
- Time to Interactive: ~14.5 s

This is what turns into the 720 ms of TBT: the browser is parsing and running JS instead
of painting and responding.

### Third parties

- Third-party code: ~812 KiB across ~63 requests (analytics, embeds, etc.)

That's roughly half the JS on the page coming from outside, competing with the actual
content during load.

### Server

- Document response: ~100 ms

The backend is fine. Everything above points at the front end.

## Field data

Not available yet — this is all lab data from one Lighthouse run. Field data (CrUX) from
pagespeed.web.dev still needs to be added to see what real visitors experience.
