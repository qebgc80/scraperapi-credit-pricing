# AI web scraper pricing: why a 1M-credit plan isn't a million pages, what the AI Parser really costs, and which ScraperAPI tier fits your volume (all seven plans compared)

An AI web scraper moves your bill from engineering hours to per-page processing. A model reads the page instead of a CSS selector, so a renamed class stops breaking your pipeline, and every page you pull carries a credit or token charge on top of the fetch. ScraperAPI handles that trade in a way most per-page tools don't: its AI Parser learns a page layout from up to three sample URLs, then parses any similar URL on that domain for standard scraping credits, which turns an ongoing model bill into a one-off setup bill of 30,000 credits per extra parser. Below are the seven plans and what a credit actually buys, the parser's real economics, and the cases where a self-hosted library is the cheaper answer.

> **Most teams should start on Hobby.** 100,000 credits, 20 concurrent threads, US and EU geotargeting, and your first AI Parser is free. That covers 10,000 JS-rendered pages or 4,000 Google SERPs, enough to prove the extraction works before you pay for volume. Move to Startup (1,000,000 credits, 50 threads) once a parser is running against the same catalog every day.
> 👉 [See Hobby and the other six tiers on ScraperAPI's pricing page](https://www.scraperapi.com/pricing?fp_ref=coupons)

## The seven ScraperAPI plans, side by side

The line-up runs from a 100,000-credit Hobby tier to an Enterprise allowance of 22,000,000+ credits. Country-level geotargeting and unlimited analytics history start at Business; pay-as-you-go starts at Scaling.

| Plan | API credits / month | Concurrent threads | Geotargeting · analytics | Billing and overage | Get it |
| --- | --- | --- | --- | --- | --- |
| Hobby | 100,000 | 20 | US & EU · 30 days | Auto-upgrade opt-in, no rollover | [See Hobby](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Startup | 1,000,000 | 50 | US & EU · 30 days | Auto-upgrade opt-in, no rollover | [See Startup](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Business | 3,000,000 | 100 | Country-level · unlimited | Auto-upgrade opt-in, no rollover | [See Business](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Scaling | 5,000,000 | 200 | Country-level · unlimited | Pay-as-you-go + auto-upgrade | [See Scaling](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Professional | 10,500,000 | 300 | Country-level · unlimited | Pay-as-you-go + auto-upgrade | [See Professional](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | 21,500,000 | 500 | Country-level · unlimited | Pay-as-you-go only, no auto-upgrade | [See Advanced](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | 22,000,000+ | 500+ | Country-level · unlimited | Pay-as-you-go, custom annual terms | [See Enterprise](https://www.scraperapi.com/pricing?fp_ref=coupons) |

USD prices for each tier sit on the pricing page and differ by billing cycle. Annual billing takes 10% off every self-serve plan, including Enterprise unless terms are customised. Credits reset when the subscription renews, so a quiet month doesn't bank capacity for a busy one.

## What a credit buys, because that's the number that decides your tier

A standard page costs 1 credit. Amazon, Walmart and eBay cost 5. Google and Bing, subdomains included, cost 25. LinkedIn costs 30. Bypassing bot protection on Cloudflare, Cloudflare Turnstile, DataDome, PerimeterX/HUMAN or Akamai adds 10, `render=true` adds 10, `premium=true` adds 10, a screenshot adds 10, and premium proxies plus JS rendering together cost 25. Paid plans can also use `ultra_premium=true` at 30 credits, or 75 with rendering. Geotargeting, sessions, device type, output format and auto parsing are free.

| Request type | Credits | Hobby buys (100k) | Startup buys (1M) |
| --- | --- | --- | --- |
| Standard page | 1 | 100,000 pages | 1,000,000 pages |
| Amazon, Walmart, eBay | 5 | 20,000 pages | 200,000 pages |
| JS rendering (`render=true`) | 10 | 10,000 pages | 100,000 pages |
| Google or Bing SERP | 25 | 4,000 queries | 40,000 queries |
| Premium proxy + JS rendering | 25 | 4,000 pages | 40,000 pages |
| LinkedIn | 30 | 3,333 pages | 33,333 pages |
| Ultra premium + JS rendering | 75 | 1,333 pages | 13,333 pages |

Read that table before you compare ScraperAPI with any per-page AI scraper. The same Hobby allowance is 100,000 pages or 1,333 pages depending on what you send at it, and plan comparisons that quote only the headline credit number hide a 75x spread.

Two controls make the burn predictable. Every response carries an `sa-credit-cost` header showing what that request cost, and the dashboard's Domain Cost Estimator prices a URL before you scrape it. You can also set `max_cost` per scrape so a single request can't run past your ceiling.

## How an AI web scraper works, and why the model is the expensive part

Three steps happen on every page. The tool fetches it, using a plain HTTP request for static sites and a headless browser such as Playwright-driven Chromium for JS-rendered ones, sometimes managing proxies, retries or a cached copy at this stage. The rendered content goes to a language model with an instruction like "extract every product name and price", and the model uses context rather than selectors to separate a sale price from a list price, or a job title from a department name. The result is then shaped into a fixed structure, with JSON Schema or Zod checking for missing fields and wrong types before anything lands in your database.

Building the same thing yourself follows the same three beats. Fetch the HTML and save the raw response, because you'll want to rerun later steps without refetching. Convert HTML to Markdown, since scripts, styles and attributes consume tokens without carrying meaning, and Markdown keeps headings, lists, links and tables at a fraction of the length. Send the Markdown plus your schema to the model. A minimal schema for a product list is a `products` array with `name` (string), `price` (number) and `in_stock` (boolean), with `name` and `price` required.

Cost lands almost entirely on input tokens. A long product page runs to thousands of tokens even after conversion, and every one of them is billed. Sending only relevant passages is the biggest lever: Exa's highlights, which return just the query-matching spans, can use up to 17 times fewer tokens than full-page text. Hosted tools price this surcharge differently. Firecrawl charges 4 credits per page for JSON mode on top of its base credit. Apify includes LLM tokens in the Actor cost rather than billing them separately. These are three different accounting models, and you can't merge them into one "cost per page" figure.

Latency is the second cost. The model runs after the fetch completes, so anything a user is waiting on pays both waits.

## Where ScraperAPI's AI Parser differs: setup cost instead of per-page cost

The AI Parser takes a different route to the same output. You send up to three example URLs of the same page type to `POST /parsers`, the system generates a parser tailored to that layout, typically in under a minute, and you then send any similar URL on that domain to `GET /parse/{parser_id}` and get structured data back. Because the model work happens once at setup, the per-URL call is a parse request rather than a fresh prompt round trip.

Field definitions are yours to control: names, descriptions, data types, and optional CSS or XPath selectors to anchor a field the model keeps misreading. Parsers are versioned, so you can pin a production workload to a specific version while you iterate the schema, and you can add, rename or remove fields without rebuilding from zero. Standard ScraperAPI parameters pass through, including `render`, `premium`, `ultra_premium` and `country_code`, which means JS rendering, proxies and geotargeting work on a parser the same way they work on a normal scrape.

The pricing is where this gets interesting. Every account gets one free parser. Each additional parser generation costs **30,000 API credits**, and parse requests bill at normal scraping rates. On Hobby, one extra parser eats 30% of the month's allowance before you've parsed a single page. On Startup, the same 30,000 credits is 3% of the allowance. On Advanced, it rounds to nothing.

That gives a clean rule. A parser pays for itself when you re-parse a fixed set of domains on a schedule: competitor pricing pages, a catalog you monitor daily, a job board you refresh hourly. It's a poor fit for one-off extraction across hundreds of unrelated domains, where every new layout costs another 30,000 credits and a prompt-per-page tool would have charged you once per page instead.

## Self-host the same idea, or buy it?

Crawl4AI is the clearest price reference for the do-it-yourself route. The library is Apache 2.0 and free forever, runs its own browser, and ships in Docker. Its Cloud version gives you 1,000 free credits every month at 10 requests a minute and 2 concurrent, then moves to 120 a minute and 10 concurrent from $10 a month. The `/scrape`, `/extract`, `/search`, `/serp` and `/answer` endpoints share one key, and extraction strategies include CSS, XPath and regex for the pages that don't need a model at all.

The code isn't what you're paying for when you self-host. You're paying for browser pools, residential proxies, stealth patches, retry logic, crash recovery on long crawls, and the person who gets paged when a target site changes its bot detection. Crawl4AI covers a lot of that ground, which is why it's a fair benchmark. If your team already runs Playwright and a proxy pool, the library route costs less cash and more attention. If nobody on the team wants to own a browser fleet, the managed version is the honest comparison, not the free GitHub repo.

The same split holds across the market. ScrapeGraphAI is an MIT-licensed Python library that takes a prompt and a URL and can run local models through Ollama. llm-scraper is a TypeScript library on Playwright that defines output with Zod or JSON Schema and can generate reusable scraping code. Stagehand, Browserbase's MIT SDK, exposes `extract()` for schema-validated data and `act()` for clicking and typing on natural-language instructions. All of them remove the selector maintenance. None of them remove the browser and proxy work.

## Accuracy: where AI extraction fails quietly

A model can misread a value, and worse, it can fill a missing field with a plausible guess instead of returning nothing. A parser that invents a price is harder to catch than one that throws an error, because nothing in your pipeline flags it.

Two habits cover most of the risk. Validate every response against your schema before it reaches the database, so type errors and missing required fields fail loudly. Then review a sample by hand, especially after any schema or parser change. If you're comparing models, test two or three on representative pages rather than trusting a benchmark, and start small: simple extraction rarely needs a frontier model. Because input tokens drive the bill, compressing a long page usually saves more than switching models does.

## Overage, limits, and the 429 you'll hit

Exceed your concurrency and you get a 429 asking you to come back inside the plan's limit. There's no surcharge for the attempt, but the request is turned away rather than queued, which matters if you're running a batch job overnight and assuming it will finish.

When credits run out, scraping pauses until the renewal date by default. Auto-upgrade moves you to the next plan automatically, and it's available on Hobby, Startup, Business, Scaling and Professional, but not on Advanced or Enterprise. Pay-as-you-go keeps requests flowing at a fixed per-credit rate up to a monthly cap you set, and it's available on Scaling, Professional, Advanced and Enterprise only. So the two escape hatches don't overlap on every tier: Hobby through Professional can auto-upgrade but can't pay-as-you-go, Advanced and Enterprise are the reverse.

👉 [Compare the billing options on each plan](https://www.scraperapi.com/pricing?fp_ref=coupons)

## Crawler identity and robots.txt before you scale

Not every crawler hitting a site is doing the same job, and the user agent tells you which one you're dealing with. Training bots such as OpenAI's GPTBot and Anthropic's ClaudeBot collect pages that may end up in model training, and blocking them signals that content should stay out of future datasets. Retrieval bots fetch pages to answer a question or cite a source: OpenAI uses OAI-SearchBot for ChatGPT search and ChatGPT-User for user-initiated visits, Anthropic uses Claude-SearchBot and Claude-User, and Exa runs ExaSearchBot, which identifies itself as `ExaSearchBot/1.0`, respects robots.txt and signs each request so site owners can verify it. Infrastructure tools like Firecrawl and Crawl4AI crawl on behalf of their users, and their user agents and robots.txt behaviour vary by tool and configuration, so read each vendor's documentation rather than assuming.

On the legal side, scraping publicly available data is broadly lawful, but personal data and intellectual property still carry limits. Collect personal data only when you have a real reason, and read the target site's terms of use before you build a pipeline around it.

## Which plan, by situation

- **Testing whether AI extraction works on one site.** Hobby, 100,000 credits, and use the free parser. 10,000 JS-rendered pages is enough to know if the output holds up.
- **One production parser on a JS-heavy catalog, refreshed daily.** Startup at 1,000,000 credits. The 30,000-credit parser cost is 3% of the allowance and 50 threads won't be your bottleneck.
- **Three or more parsers, country-level geotargeting, unlimited analytics history.** Business at 3,000,000 credits.
- **Sustained volume above 3,000,000 credits, or spikes you can't predict.** Scaling, because pay-as-you-go becomes available there and not on the lower tiers.
- **Not ScraperAPI.** One-off extraction across many unrelated domains (each new layout costs 30,000 credits), sub-second response requirements where the model wait is unacceptable, or a team already operating Playwright and proxies at cost.

## Before you check out

Four things decide whether the plan you pick survives contact with production. Estimate your real request mix first, because rendering, premium proxies and SERP targets multiply the credit cost by 5 to 75. Remember credits don't roll over, so a bigger plan doesn't bank slack for next quarter. Check the parser arithmetic if you're extracting from more than one site layout. And confirm the current signup offer at the point of purchase, since older versions of the pricing page advertised a 7-day trial with 5,000 credits and a 1,000-credit free tier with 5 concurrent connections, and only the live page states what applies now.

The shortest path: open Hobby, train your free parser on three URLs from the site you actually care about, watch the `sa-credit-cost` header for a week, and let the measured burn pick your tier instead of the other way around.

👉 [Open ScraperAPI's pricing page and check the current rate for each plan](https://www.scraperapi.com/pricing?fp_ref=coupons)
