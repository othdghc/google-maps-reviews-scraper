# google maps reviews scraper: How to Pull Every Review Without Getting Blocked — Real Credit Costs, Plan Math, and Which Route Fits Your Project

Scraping Google Maps reviews comes down to three realistic routes: Google's official Places API, which caps you at 5 reviews per business; a do-it-yourself browser automation stack running through a scraping proxy like ScraperAPI; or a dedicated managed scraper that handles parsing for you. Which one makes sense depends on how many reviews you need, how much code you want to maintain, and — the part almost nobody explains — what each request actually costs.

This article walks through all three routes, then breaks down ScraperAPI's pricing in detail, because its credit system has a catch that changes the math for Google Maps specifically: requests to Google domains burn **25 credits each**, not 1. If you're comparing plans by headline credit counts, you'll walk away with a very wrong budget estimate.

> **Quick take:** Most people testing a Google Maps reviews scraper should start with the free 7-day trial (5,000 API credits, no credit card), then judge by real consumption. If you're monitoring reviews for a handful of businesses, Hobby at $49/month covers roughly 4,000 Google requests. If you need review-level structured data with zero parsing code, a dedicated scraper like the one on Apify fits better — ScraperAPI wins when you need scale, geotargeting, and control over your own pipeline. [👉 Start the 7-day ScraperAPI trial free](https://www.scraperapi.com/signup?fp_ref=coupons)

## Why your script gets blocked before page 2

Google Maps is one of the hardest scraping targets on the open web, and the reasons are concrete, not mysterious.

Local results change based on the IP making the request — the same "coffee near me" query returns different businesses from a US IP versus a German IP. Google fingerprints traffic on IP reputation, header order, and behavioral signals, so repeated requests from one IP hit rate limits or CAPTCHAs quickly. Parts of the local results markup only render through JavaScript, so plain HTTP libraries get incomplete data. And when Google updates its frontend, the CSS selectors your parser depends on simply stop existing.

A scraping API like ScraperAPI sits in front of that problem: requests route through a rotating pool of 40 million+ IPs across 50+ countries, JavaScript renders on demand, CAPTCHAs get solved automatically, and failed requests are retried. It doesn't make Google Maps easy, but it moves the maintenance burden off your machine.

## Three ways to actually get the reviews

### Route 1: Google's official Places API

The official route requires creating a Google Cloud project, generating an API key, and paying per use at scale. The bigger issue for review monitoring: it returns a maximum of 5 reviews per business listing. For a restaurant with 3,000 reviews, that's a non-starter — which is exactly why so many people search for third-party scrapers in the first place.

### Route 2: Selenium + ScraperAPI in proxy mode (full review text)

This is the approach from ScraperAPI's own tutorial for pulling Google Maps review and business data. The setup is Python with:

- **Selenium + Selenium Wire** — browser automation with proxy integration
- **Webdriver Manager** — automatic ChromeDriver management
- **BeautifulSoup + lxml** — HTML parsing
- A ScraperAPI API key from your dashboard

The proxy connection looks like this in code:

python
proxy = "http://scraperapi.render=true.country_code=us:{YOUR_API_KEY}@proxy-server.scraperapi.com:8001"


The script runs headless Chrome, clicks through the cookie consent screen, waits for JavaScript rendering, then simulates Page Down presses on the results panel to load more entries. Parsing relies on specific CSS classes — `hfpxzc` for business names, `MW4etd` for ratings, `UY7F9` for review counts, `Ahnjwc` for service options — with missing fields written out as "N/A".

The trade-off is worth stating plainly: this gets you the raw page, but you own the parser. When Google renames those CSS classes, your script breaks and you fix it. It works, and it's flexible, but "set it and forget it" it is not.

### Route 3: ScraperAPI's structured endpoint (business data, no parser)

ScraperAPI also maintains structured data endpoints — 18 of them across Amazon, Google, Walmart, eBay, and Redfin — that return pre-parsed JSON instead of raw HTML. The Google Maps endpoint takes a search query plus optional latitude and longitude, and returns business-level data: name, address, star rating, review count, price level, categories, opening hours, coordinates, and photos. Structured endpoints are available on all plans, including the free tier.

One honest limitation: this endpoint covers **business listing data**. For individual review text at depth — every review, with reviewer details and owner replies — you're back to Route 2's parsing approach, or to a dedicated review scraper.

## ScraperAPI plans: the full current table

Here's everything on the official pricing page. Annual billing takes 10% off every plan.

| Plan | Price (monthly / annual per month) | API Credits | Concurrent Threads | Geotargeting | Link |
| --- | --- | --- | --- | --- | --- |
| Free | $0 | 1,000 | 5 | None | [ Get 1,000 free credits](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Hobby | $49 / $44.10 | 100,000 | 20 | US & EU only | [ Start Hobby](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Startup | $149 / $134.10 | 1,000,000 | 50 | US & EU only | [ Start Startup](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Business | $299 / $269.10 | 3,000,000 | 100 | Global, country-level | [ Start Business](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Scaling (most popular) | $475 / $427.50 | 5,000,000 | 200 | Global, country-level | [ Start Scaling](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Professional | $975 / $877.50 | 10,500,000 | 300 | Global, country-level | [ Start Professional](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Advanced | $1,975 / $1,777.50 | 21,500,000 | 500 | Global, country-level | [ Start Advanced](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Enterprise | Custom | 22M+ | 500+ | Global + dedicated | [ Talk to sales](https://www.scraperapi.com/pricing/?fp_ref=coupons) |

Every plan includes JavaScript rendering, premium rotating proxies, automatic CAPTCHA handling, custom headers and sessions, auto-retry, unlimited bandwidth, and a 99.9% uptime guarantee. The differences between tiers come down to credits, concurrency, geotargeting precision, and analytics history (30 days on Hobby and Startup, unlimited from Business up).

## The 25-credit rule: what a Google Maps scrape really costs

This is the part headline pricing skips. ScraperAPI charges per credit, and the credit cost depends on the target domain:

| Target | Credits per request |
| --- | --- |
| Standard web page | 1 |
| Amazon, Walmart, eBay | 5 |
| **Google and Bing (all subdomains)** | **25** |
| LinkedIn | 30 |
| Sites behind Cloudflare / DataDome / PerimeterX | +10 for automatic bypass |

Google Maps sits on a Google domain, so every Maps request costs 25 credits. That means Hobby's 100,000 credits translate to about **4,000 successful Google requests per month**, not 100,000 pages. At $49, that's roughly $0.012 per request — and if you're running Selenium with `render=true`, rendering adds another 10 credits per request, bringing you to 35.

Feature toggles stack on top of the domain cost, and not linearly. Per ScraperAPI's documented rates, `render=true` adds 10, `premium=true` adds 10, `ultra_premium=true` adds 30 — but combinations reprice: premium plus render together cost 25 total, and ultra premium plus render costs 75. Pages behind hard bot protection get an automatic 10-credit surcharge when ScraperAPI bypasses it, whether you asked or not.

Two tools keep this under control, and both are built in:

1. **Domain Cost Estimator** in the dashboard — enter any URL and see its exact credit cost before you scrape it. Costs can vary even within the same domain depending on parameters.
2. **`max_cost` per request** — cap what any single request can spend so one heavy page can't quietly drain your quota.

> If you remember one thing: the credit number on the pricing page is a budget, not a request count. Multiply your expected Google Maps requests by 25 (plus any feature toggles), then pick the plan that survives that math. [👉 Check current plan prices and the cost estimator](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## Hidden limits that affect which plan you pick

Four rules from the pricing FAQ that don't show up in comparison tables:

- **Credits don't roll over.** Whatever you don't use resets at each billing cycle. Sizing your plan for your busiest month means paying for idle capacity in the quiet ones.
- **Concurrency is a hard wall, not a queue.** Exceed your thread limit and you get a 429 rejection — the request is refused, not held. A task that needs 60 parallel requests won't run at full speed on Startup's 50 threads no matter how many credits you have.
- **Running out of credits has different consequences by tier.** Hobby through Business simply stop until the next cycle (or auto-upgrade, if you enable it). Pay-as-you-go overage at a fixed per-credit rate is only available on Scaling and above.
- **Geotargeting matters for Maps specifically.** Since local results vary by IP location, Hobby and Startup (US & EU only) can't give you, say, Japan-specific local results. Country-level targeting starts at Business, $299/month.

A 7-day refund window is also listed in the pricing FAQ, which lowers the risk of picking a tier and discovering mid-month that it's too small.

## ScraperAPI vs. dedicated Google Maps review scrapers

If what you actually want is "give me every review for this restaurant as clean data," a managed scraper is a different product category than a scraping API, and it's worth being straightforward about where each wins.

The Google Maps Reviews Scraper on Apify, for example, takes a Maps URL, short link, Place ID, or even a plain search query, and outputs one structured row per review: rating, full text, language, exact publish timestamps, photos, reviewer profile details (including Local Guide status), and the owner's reply. It paginates newest-first through tens of thousands of reviews with a resume cursor, so a deep scrape of a place with 50,000+ reviews — the Apify docs use Katz's Delicatessen in New York, 54,692 reviews at 4.5 stars — can continue across runs. Billing is per review saved to the dataset, with nothing charged for places that fail to load.

The honest comparison:

|  | ScraperAPI | Dedicated review scraper (e.g., Apify) |
| --- | --- | --- |
| What you get | Raw HTML or business-level JSON; you write the parser | Structured review rows, ready to export |
| Billing basis | Credits per request (Google = 25) | Per review saved |
| Flexibility | Any site, any parameter, full pipeline control | Fixed schema, Maps reviews only |
| Maintenance | You own the parser; Google redesigns break it | Maintained by the platform |
| Scale features | 20–500 concurrent threads, 50+ country geotargeting, async and scheduled pipelines | Per-run concurrency, resume cursor for deep scrapes |

Pick a dedicated scraper when your whole job is review extraction and you want zero parsing code. Pick ScraperAPI when Maps reviews are one part of a broader pipeline — when you're also pulling SERPs, Amazon, or LinkedIn data — and you want one API and one bill for all of it. As the company's own marketing puts it, the goal is scraping "any website consistently and at a large scale without ever getting blocked."

## FAQ

**How many Google Maps reviews can I scrape per month on the $49 Hobby plan?**
About 4,000 Google requests, since each Google-domain request costs 25 of the plan's 100,000 credits. With `render=true` enabled for full page rendering, that drops to roughly 2,857 requests at 35 credits each. How many reviews that yields depends on how you paginate and parse.

**Is scraping Google Maps reviews legal?**
Public review data is commonly scraped for reputation monitoring, competitive analysis, and NLP datasets, but legality varies by jurisdiction and use case, and Google's terms of service prohibit it. This is a genuine question to research for your situation; this article covers the technical and cost side only.

**Does ScraperAPI have a free trial?**
Yes — a 7-day trial with 5,000 API credits and no credit card required. There's also a permanent free plan with 1,000 credits and 5 concurrent connections, enough to run the Domain Cost Estimator and see what your target actually consumes before paying anything.

**What happens when I run out of credits mid-month?**
On Hobby through Business, scraping pauses until the next billing cycle unless you enable auto-upgrade, which moves you to the next plan automatically. Scaling and above can keep going with pay-as-you-go overage at a fixed per-credit rate, capped by a monthly spending limit you set.

**Can I get reviews from a specific country?**
Only from Business ($299/month) upward. Hobby and Startup geotargeting covers US and EU IPs, and since Google Maps results vary by requester location, country-level targeting outside those regions requires the higher tier.

## Which setup should you actually pick

Match the decision to your situation:

- **Monitoring a handful of businesses for review sentiment** → A dedicated review scraper does this with no code; per-review billing is the simplest math.
- **Building your own pipeline, or scraping Maps alongside other sites** → ScraperAPI's Hobby or Startup plan, having first run your target URL through the cost estimator to confirm the real credit burn.
- **Tracking local rankings across specific countries at production volume** → Business at $299/month is the first tier with global country-level targeting, and it's the usual upgrade trigger from Startup.
- **Running an agency with recurring, high-volume jobs** → Scaling ($475/month) adds pay-as-you-go overage and 200 threads, which is why it's the plan most traffic lands on.

Before you commit to any paid tier, do one thing first: put your actual target URL and parameters into the dashboard's cost estimator, set a `max_cost` on your test requests, and run a small batch. The number that comes back — not the credit count on the pricing page — is what your monthly bill will look like. [👉 Start with the free trial and see your real costs](https://www.scraperapi.com/signup?fp_ref=coupons)
