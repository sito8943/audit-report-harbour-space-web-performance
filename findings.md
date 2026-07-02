# Performance Findings

## Homepage

Visible page load is significantly delayed.
Metric: LCP is too big (13.5 s).
Cause: The main thread is busy running JavaScript (4.2 s of work, ~405 KiB unused) so the hero content paints late.
Solution: Split and defer non-critical JS, drop the unused bundles, and preload/prioritize the hero image with fetchpriority=high.
