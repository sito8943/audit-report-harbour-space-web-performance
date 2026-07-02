# Baseline Results

## Summary

I ran Lighthouse on the homepage to get a starting point. The page doesn't shift around
and it reacts fine once it's up, but the main content shows up way too late, so the
performance score ends up low.

## Homepage (https://harbour.space/)

Lighthouse, mobile, run on 2026-07-02.

Scores:

- Performance: 51
- Accessibility: 93
- Best Practices: 77
- SEO: 92

Metrics:

- First Contentful Paint: 2.2 s
- Largest Contentful Paint: 13.5 s (this is the bad one)
- Total Blocking Time: 720 ms
- Cumulative Layout Shift: 0
- Speed Index: 5.3 s
- Time to Interactive: 14.5 s

Still need to grab the field data (real user numbers) from pagespeed.web.dev. The free
PSI API was rate limited when I tried, so this run is lab only for now.

## What this tells me

Layout is stable (CLS is 0) and blocking time isn't terrible, so the problem isn't the
page jumping around or being unresponsive. It's that the biggest piece of content takes
13.5 seconds to render. That's a loading problem, probably the hero media plus heavy
JavaScript, and it's what makes the site feel slow.
