# Sources

Candidate sources for the demand scan, with access method and ToS status.

## Verification status: PROVISIONAL

On 2026-11-21 I tried to fetch `robots.txt` for 17 candidate hosts from the cloud sandbox. **Every
request was refused by the sandbox's egress policy (HTTP 403 on the proxy tunnel)**, so no robots.txt
or ToS page was actually read. The statuses below are my **general knowledge, unverified**, and are
deliberately conservative. Before any collector touches a site, a `researcher` must read its current
robots.txt and terms (via an allowed route) and update the row with the date checked.

Decision values: `automated` (official API/RSS/open data; allowed), `manual only` (no automated
reading; use pasted or exported data you provide), `skip` (not useful or not permitted).
**Nothing is marked `automated` until verified**; unverified candidates are `manual only` or
`pending verification`.

## Table

| # | Source | Public access? | API / RSS? | robots.txt / ToS (unverified; date checked: none) | Decision (provisional) | Notes |
|---|---|---|---|---|---|---|
| 1 | Upwork | Job pages partly public; search needs login | Official API exists, gated by approval and terms | ToS restricts scraping; API use needs approved access | `manual only` | Hypothesis: large volume, skewed to hardware/firmware. Use pasted/exported search results only if terms allow personal-use export. |
| 2 | Freelancer.com | Project listings public | Official developer API | API likely permitted within its terms; scraping unclear | `pending verification` | Hypothesis: a candidate for an API route. Verify API terms and rate limits. |
| 3 | Fiverr | Gig pages public | No public job-post API | ToS restricts automated access | `manual only` | Gigs show supply, not buyer demand; use only as a price/positioning reference. |
| 4 | Toptal | Talent-gated, no public job board | None | N/A | `skip` | No public demand data. |
| 5 | Contra | Mostly profiles; some public opportunities | Unknown | Unverified | `manual only` | Low expected volume for power electronics. |
| 6 | Guru | Public job listings | Unknown | Unverified | `pending verification` | Low expected volume. |
| 7 | PeoplePerHour | Public job listings | Unknown | Unverified | `pending verification` | Low expected volume. |
| 8 | Malt | Mostly EU, login-heavy | Unknown | Unverified | `manual only` | Possible EU signal. |
| 9 | LinkedIn (services, posts) | Login-walled | Official APIs not suited to this use | ToS prohibits scraping | `manual only` | Likely holds real consulting demand. Use posts/searches you paste or export yourself. Never scrape. |
| 10 | Catalant | Expert marketplace, login-gated | None | N/A | `skip` | Login-gated; pasting its content may itself breach terms. Revisit only if you confirm terms allow it. |
| 11 | Expert networks (GLG-style) | Private | None | N/A | `skip` | No public data. Your own inquiry history is a better signal (see below). |
| 12 | EEVblog forum | Public forum | RSS likely | Unverified | `pending verification` | Shows design questions, not paid demand. Context signal. |
| 13 | Power Electronics News | Public articles | RSS likely | Unverified | `pending verification` | Industry trend signal, not freelance demand. |
| 14 | r/PowerElectronics (Reddit) | Public | Official API with terms | Data API terms restrict commercial use; verify | `pending verification` | Questions and "looking for help" posts. Respect Reddit API terms. |
| 15 | Electronics Stack Exchange | Public Q&A | Official API, CC-licensed content | Generally permitted via API with attribution; verify | `pending verification` | Topic-frequency signal, not hiring. Content is CC BY-SA (attribution and share-alike) and API terms have AI/commercial limits; verify before any reuse. |
| 16 | University and research-lab collaboration calls (e.g. Mitacs, NSERC programs, lab pages) | Public | Often RSS or open pages | Unverified; not assumed | `pending verification` | Useful for Canadian industry-academic project demand. |
| 17 | Public RFQ / tender portals (e.g. SAM.gov, MERX, CanadaBuys, TED) | Public | Open-data APIs/feeds on several | Open data intended for reuse; verify per portal | `pending verification` | Hypothesis: the best-documented automated route. MERX is a paid subscription service, not open data; verify each portal. Often hardware-heavy; filter by design-study wording. |

## What I expect (hypothesis, not a finding)

- Platforms (rows 1-10) will mostly show hardware and firmware work, so design-only postings may be rare.
- Real consulting demand likely sits in LinkedIn, expert networks, and direct inquiries. Your own
  history of inquiries, with client names removed, is a data source only you can provide.
- Forums and Q&A (rows 12-15) measure topic interest, which is not the same as paid demand. Reports
  must keep those two signals separate.

## Next steps (gate before Phase 2)

1. You confirm which rows you want investigated.
2. For each `pending verification` row, a `researcher` reads current robots.txt and terms from an
   allowed route and records: date checked, quoted rule (under 25 words), final decision.
3. The sandbox blocked all hosts. You may need to allow the hosts you approve in the environment's
   network settings, or provide exports yourself. I will not work around the block.

## Notes from review
- Commercial use matters: findings feed a company and possibly a licensed tool. Every row needs a
  commercial-use check, not only a robots.txt check. Pasting login-gated content may breach terms too.
- `pending verification` is an extra decision value beyond the three in `PLAN.md`; please approve it.
- Columns for robots.txt and ToS should be split once verified, with the date checked.
