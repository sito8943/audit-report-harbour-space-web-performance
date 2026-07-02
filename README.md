# Harbour.Space Performance Audit

**Website:** Harbour.Space University
**URL:** https://harbour.space/

## Why This Is a Good Candidate

Harbour.Space is a non-tech-industry site (a university) that I use frequently, yet it
behaves like a heavy modern web app. It mixes a media-rich marketing homepage (video,
large hero images), static content pages (program descriptions), dynamic data views
(course schedules that load asynchronously), interactive modules (filters, tabs,
in-page loaders), authenticated flows (the application / account portal), and a
subdomain-based scholarship platform. That variety of content — static, dynamic,
interactive, in-page loaders, and authentication — makes it a strong target for a
performance audit, and initial checks show clear room for improvement on the
image-heavy and JavaScript-heavy pages.

## Target Pages

- https://harbour.space/

  Main entry point. Media-heavy marketing homepage (hero video/images, animations).
  Sets the first-impression performance bar and drives most traffic.

- https://harbour.space/computer-science

  Representative program page. Mostly static long-form content — good baseline for how
  well the site renders straightforward content pages.

- https://harbour.space/faculty

  Image-heavy page with many faculty profile photos. Strong candidate for the required
  **red** score due to large image payload and layout work. (does not have to be the homepage)

- https://harbour.space/schedule

  Barcelona course schedule. Dynamic, data-driven view with in-page loaders — tests
  client-side rendering and async data fetching performance.

- https://harbour.space/account

  Application / login portal. Authenticated, interactive flow — measures the experience
  where conversion actually happens (sign up / apply).

- https://scholarship.harbour.space/

  Scholarship platform (separate subdomain). Dynamic listings and forms; performance
  here directly affects a key conversion path for prospective students.

- https://harbour.space/articles/home

  Blog / articles feed. Dynamic content list with many thumbnails — tests repeated
  media loading and pagination.

- https://harbour.space/campus-barcelona

  Campus page. Media-heavy (photos, embedded media) marketing content; important for
  prospective-student trust and conversion.

## Reports

- `baseline.md`: Initial Core Web Vitals and PageSpeed Insights results.
- `findings.md`: Main performance issues found during the audit.

> **TODO:** Fill `baseline.md` and `findings.md` with real PageSpeed Insights runs.
> At least one target page must show a red (poor) score — `faculty` is the expected candidate.
