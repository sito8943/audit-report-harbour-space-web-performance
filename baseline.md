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

## Network baseline at `/`

I checked the HAR for the homepage too. It has the first load and then the refresh, so I
can compare what gets downloaded the first time and what happens when the page is loaded
again.

### Initial load

The first load makes **149 requests**, which is already a lot for a homepage.

It transfers **1.87 MB**, but the total resource size is **5.83 MB**. So compression is
doing a lot of work here: the browser downloads about **67.9% less** than the full
resource size.

### Soft refresh

The refresh is the disappointing part. It makes **151 requests** and transfers **1.87
MB** again. More exactly, it transfers 1,870,879 bytes, compared with 1,869,478 bytes on
the first load.

So the cache reduction is basically **0%**. It is actually a tiny bit worse, around
**-0.1%**, because the refresh transfers about 1.4 KB more than the first load.

That means this refresh is not really benefiting from cache. The HAR has no `304`
responses and no `0 B` transferred resources. Almost every request in the refresh also
has `Cache-Control: no-cache` and `Pragma: no-cache`, so the browser asks for the files
again instead of just reusing them.

### Resource type breakdown

Most of the page is JavaScript. That matches the Lighthouse result: the page is not huge
because of images, it is heavy because of code.

JavaScript alone is **50 requests**, **1.44 MB transferred**, and **4.57 MB total
resource size**. That is **76.9%** of everything transferred.

CSS is small next to that: **6 requests**, **20.3 KB transferred**, and **104.4 KB total
resource size**.

Together, JS and CSS are **56 requests**, **1.46 MB transferred**, and **4.67 MB total
resource size**. So code is about **77.9%** of the transferred data and **80.2%** of the
total resource size.

Images are not the biggest byte problem here. There are many of them, **73 requests**,
but they only transfer **173.3 KB** and have **300.7 KB** total resource size. So images
are only **9.3%** of the downloaded data.

Fonts are **4 requests** and about **112.9 KB transferred**.

Third-party stuff is still big. Counting Harbour.Space and its asset CDN as first-party,
third parties are **25 requests**, **819.5 KB transferred**, and **2.57 MB total resource
size**. That is **43.8%** of the downloaded data.

### Compression notes

JS and CSS are compressed. Most scripts use `gzip`, some use `br`, and some use `zstd`.
The stylesheets use `gzip` or `br`.

Images are mixed. Most of the image requests are SVG files and they are served with
`gzip`. The binary images like `avif`, `gif`, and `png` are not compressed again with
gzip or brotli, which is normal because those formats are already compressed.

Fonts are not compressed again either, but `woff` and `woff2` are already compressed font
formats.
