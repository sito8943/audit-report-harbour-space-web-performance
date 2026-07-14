# Harbour.Space Performance Audit

**Website:** Harbour.Space University
**URL:** https://harbour.space/

## Why This Is a Good Candidate

I study and work at Harbour.Space, so I'm on this site all the time and I've noticed it can feel slow. There's a media-rich homepage (video and big hero images), plain content pages like the program descriptions, dynamic views like the course schedule that loads its data after the page, interactive bits, the account portal you have to log into to apply, and the scholarship platform on its own subdomain. That mix of static, dynamic, interactive, loaders and authentication makes it a good thing to audit, and a first look already shows the image-heavy and JavaScript-heavy pages have room to improve.

## Target Pages

- https://harbour.space/ — homepage, media-heavy and the main entry point.
- https://harbour.space/computer-science — a program page, mostly static content.
- https://harbour.space/faculty — lots of profile images, likely the red score.
- https://harbour.space/schedule — course schedule, dynamic with in-page loaders.
- https://harbour.space/account — login/apply portal, authenticated flow.
- https://scholarship.harbour.space/ — scholarship platform, forms on its own subdomain.
- https://harbour.space/articles/home — blog feed, dynamic list with many thumbnails.
- https://harbour.space/campus-barcelona — campus page, image and media heavy.

## Reports

- `baseline.md`: Core Web Vitals (field, mobile + desktop), Lighthouse lab, and the network baseline from the homepage HAR.
- `findings.md`: Main performance issues found during the audit, plus what the site does well.
- `prioritization.md`: The WSJF system I use to rank the fixes, and the resulting order.

The homepage already shows a red score: mobile LCP is 8.2 s in the lab (and 3.4 s for real
users), and mobile Speed Index is 6.4 s.
