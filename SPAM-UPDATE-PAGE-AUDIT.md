# jimboscleaning.com: page pruning for Google spam updates

Source: Search Console "Performance on Search" export, last 16 months (to 2026-10-02), web search.
Full per-URL list with actions and redirect targets: `SPAM-UPDATE-PAGE-ACTIONS.csv`.

## The risk

The site generates **21 towns × 6 services = 126 location-service pages** (`/[location]/[service]/`).
Their copy comes from `locationServiceIntros.ts`: 3 intro templates per service, with only the town and
neighbour names swapped. Google's spam policies call this **scaled content abuse / doorway pages**, and it
is the main thing the recent spam updates target.

Those 126 pages brought in **55 clicks in 16 months**. 93 of them got **zero clicks** and 10 never
appeared in search at all. Nearly all of the site's traffic comes from the homepage (376 of 560 clicks)
and the town pages.

## 1. Remove: 119 of the 126 location-service pages

Remove every `/[town]/[service]/` page with fewer than 3 clicks. Together these 119 pages earned
28 clicks in 16 months.

- 101 of them belong to towns you keep. Redirect each one (301) to its town page, for example
  `/ayr/gutter-cleaning/` → `/ayr/`.
- The other 18 belong to the towns removed in section 2, and redirect to the main service page.

Keep only these 7, which each earned 3 or more clicks, and **rewrite them with genuinely unique content**
(real local jobs, before/after photos, prices):

| URL | Clicks | Impr | Pos |
|---|---|---|---|
| /barrhead/driveway-cleaning/ | 5 | 714 | 43.5 |
| /largs/gutter-cleaning/ | 4 | 87 | 9.1 |
| /largs/driveway-cleaning/ | 4 | 54 | 21.5 |
| /cumnock/window-cleaning/ | 4 | 57 | 7.2 |
| /west-kilbride/window-cleaning/ | 4 | 38 | 6.2 |
| /stewarton/driveway-cleaning/ | 3 | 22 | 17.9 |
| /kilmarnock/window-cleaning/ | 3 | 82 | 10.7 |

If you can't make these 7 unique, remove them too. None of them is worth the risk to the whole site.

## 2. Remove whole towns: Paisley, Renfrew, Johnstone

| Town page | Clicks | Impr | Pos |
|---|---|---|---|
| /paisley/ | 0 | 894 | 37.5 |
| /renfrew/ | 0 | 574 | 31.3 |
| /johnstone/ | 0 | 282 | 66.0 |

These three got zero clicks on their town pages and 1 click across all 18 of their service pages.
They are in Renfrewshire, outside the Ayrshire core. Redirect the town pages to `/locations/` and their
service pages to the main service page, for example `/paisley/gutter-cleaning/` → `/gutter-cleaning/`.

## 3. Merge: Stevenston → Saltcoats

`/stevenston/` got 1 click from 219 impressions, and Saltcoats is next door. Redirect it to `/saltcoats/`
and mention Stevenston in the Saltcoats page copy.

## 4. Keep for now, but review (weak)

| Town page | Clicks | Impr | Pos | Note |
|---|---|---|---|---|
| /newton-mearns/ | 2 | 1755 | 35.2 | East Renfrewshire. Keep only if you genuinely work there |
| /barrhead/ | 3 | 1221 | 35.0 | East Renfrewshire. Same question |
| /prestwick/ | 2 | 565 | 51.9 | Could merge into /ayr/ |
| /kilwinning/ | 1 | 415 | 34.5 | Could merge into /irvine/ |
| /ardrossan/ | 2 | 234 | 22.0 | Could merge into /saltcoats/ |

If they're still at about 0 clicks after another 3 months, merge them as shown.

## 5. Keep and improve. Do not remove these

- Homepage, about, contact, gallery, locations, services
- The 6 core service pages. Even weak ones like `/gutter-cleaning/` (0 clicks, pos 66) and
  `/roof-steam-cleaning/` (0 clicks, pos 42) are the pages that should rank, so they need better
  content, not removal.
- Town pages that earn clicks: Irvine (34), Largs (21), Troon (10), West Kilbride (9), Stewarton (8),
  Saltcoats, Dalry, Maybole (5 each), Ayr, Mauchline (4), Kilmarnock, Cumnock (2).
  Every kept town page needs real local proof (jobs done there, photos, reviews), not swapped place names.

## 6. Technical clean-up found in the data

- **Old URLs still indexed:** `/locations/{ayr,irvine,kilmarnock,prestwick,troon}/` and
  `/services/{driveway,pvc-white,render-softwashing,window}-cleaning/`. Redirect each one (301) to the
  current URL.
- **Duplicate URLs without a trailing slash are indexed**, for example `/paisley/render-softwashing` and
  `/ayr`. Make sure the server sends a 301 to the trailing-slash version (`trailingSlash: 'always'` in
  Astro does not redirect by itself on Vercel; add `"trailingSlash": true` to `vercel.json`).
- After removing pages, take them out of `sitemap.xml.ts`. It currently lists every location × service
  combination. Also remove the links to them from the location and service-area sections.

## How to implement (Astro, commit `00e1ba9b`)

1. In `src/pages/[location]/[service].astro`, change `getStaticPaths` to build pages only from an
   allow-list of the 7 kept pages.
2. Remove `paisley`, `renfrew`, `johnstone` and `stevenston` from `src/data/locations.ts`.
3. Add the redirects to `vercel.json`. One rule `/:town/:service/` → `/:town/` covers the bulk of the
   removed service pages. Put the specific rules for the 7 kept pages above it.
4. Generate the sitemap from the same allow-list.
5. In Search Console, resubmit the sitemap and watch "Pages → Not indexed" fall over 4–8 weeks.

> Note: the current `HEAD`/`main` of this repo is the GlenView Exterior Cleaning site. The
> jimboscleaning.com code was last at commit `00e1ba9b`. No site code was changed for this audit.
