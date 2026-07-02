# Performance Findings

> **TEMPLATE — fill after running the audit.** Record the main bottlenecks per page.

## Summary

_High-level: where the site loses the most time (rendering / JS / images / network)._

## Key Findings

- Largest Contentful Paint (LCP): TODO s
  _Why it's slow._
- Total Blocking Time (TBT): TODO ms
  _Main-thread blocking impact._
- Speed Index: TODO s
- First Contentful Paint (FCP): TODO s
- Cumulative Layout Shift (CLS): TODO

## Likely Causes

- [ ] Too much main-thread work
- [ ] High JavaScript execution time
- [ ] Large amount of unused JavaScript
- [ ] Large network payload (images/media)
- [ ] Render-blocking requests
- [ ] Image optimization opportunities (esp. faculty photos)
- [ ] Cache lifetime improvements

## Recommended Actions

1. Optimize and compress hero images and large media assets.
2. Minimize JavaScript execution — remove unused code, defer non-critical scripts.
3. Reduce third-party and render-blocking resources.
4. Improve caching and asset delivery for repeat visits.
5. Re-test after each change to confirm LCP, TBT, and Speed Index improvements.
