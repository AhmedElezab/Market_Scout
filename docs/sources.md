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

## Verification results (2026-11-21, read via Firecrawl; sandbox network stayed blocked)

robots.txt was read for 11 hosts; terms of use for 3. A robots.txt "allow" is not permission under the
site's terms, so both are recorded. Firecrawl cannot fetch Reddit, so Reddit is unchecked.

| Source | robots.txt (2026-11-21) | Terms (2026-11-21) | Final decision |
|---|---|---|---|
| Freelancer.com | Project pages allowed; `/projects/browse.php*`, `/jobs/?*`, `/find/*` disallowed | Bans "any robot, spider, scraper or other automated means", API included, without written permission | **`manual only`** (do not collect automatically, even through a tool) |
| Guru | Most pages allowed; `/d/search/*` and `/api/` disallowed | Bans robots meant to "damage or interfere", and "intercept or expropriate any system, data" from the site. Ambiguous | **`manual only`** (conservative reading) |
| PeoplePerHour | Job pages allowed; query strings disallowed except `?page=` | Two extraction passes (2026-11-21) found no scraping clause, only IP-assignment clauses. An LLM extraction, not a human read | `automated`, provisional: low volume, public job pages only, no query strings, 5 s delay. You should skim the terms once yourself |
| Electronics Stack Exchange | `User-agent: *` `Disallow: /`, `Content-signal: search=no, ai-train=no` | Not read | **`manual only`** for page reading. Official API is a separate route that is not yet reviewed |
| EEVblog forum | `Allow: /`, `Crawl-delay: 10`; some AI bots blocked | Not read | `pending verification`. Low value: topic interest, not paid demand |
| Power Electronics News | Generic allowed; `anthropic-ai`, `GPTBot`, `CCBot` and others blocked | Not read | **`skip`**: industry news, not demand, and it signals opposition to AI use |
| Mitacs | `Disallow: /*?` only | Not read | `pending verification` |
| CanadaBuys | `User-agent: *` `Disallow: /` (only Googlebot and bingbot allowed) | Not read | **`manual only`**. Check its open-data feeds separately |
| MERX | General pages allowed; names ClaudeBot, GPTBot and other AI scrapers as blocked | Not read | **`manual only`** (explicit anti-AI-scraper stance) |
| TED (EU tenders) | Generally allowed; dynamic search results disallowed; bulk XML download page allowed | Legal notice: notices "can be freely reused, for commercial or non-commercial purposes" with credit | `automated` via official API or bulk download only. No Firecrawl provider exists, so a collector must be built (Phase 3) |
| SAM.gov | Standard Drupal robots; data lives in `api.sam.gov` | Public federal notices; accessed through an Alexandria provider (typed search) | `automated`, but **low value**: see yield result below |
| Reddit | Could not be read | Not read | `pending verification` (API terms, commercial use) |

Early yield probe (Freelancer search index, 2 results only, not a sample): a "LLC Resonant Converter
Design and Analysis" listing and several SMPS design listings exist, so design-only items do appear. But
the only platform where I found them is the one whose terms ban automated access.

### SAM.gov yield probe (2026-11-21)
4 keyword queries ("power electronics design", "DC-DC converter design", "resonant converter", "power
converter trade study") returned 66 records (the broad query matched 1,933 in total). By title, almost all are
hardware purchases or unrelated (MRI scanners, golf carts, radar repair, supplies); the matches look like
keyword noise. At most 2 or 3 are plausibly design-related (for example a NASA spacecraft electrical power
systems pre-RFI, 2026-11-11, and a DOE microelectronics RFI). This is a title-level read by me, not a
labeled sample, so no share is claimed. **SAM.gov is not a usable source of design-only freelance demand.**

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

## Owner decisions 2026-11-21
Source list approved; `pending verification` accepted as a decision value. Network access requested via a new `market-scout` environment, but the request was rejected by the environments service, so hosts remain blocked.

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
