# puppeteer data scraping: how to keep your Node.js scraper unblocked, plus the real credit math behind ScraperAPI's plans

Most puppeteer data scraping tutorials stop at "here's how to extract a table." That's the easy half. The hard half starts when your script works perfectly on a test site and then gets 403s from the real one, because a headless browser leaks automation signals and hammers a site from a single IP. This guide covers both halves: what Puppeteer genuinely handles well, where it hits a wall, and how a scraping API like ScraperAPI fills the gap — including its full plan list, the credit multipliers that actually determine your cost per page, and a few billing details that only show up in the docs.

**Quick take:** most individual developers and small teams should start with the Hobby plan at $49/month ($44.10 billed annually, 100,000 credits, 20 concurrent threads). Test first with the free signup — 5,000 credits over 7 days, no credit card required. 👉 [Check ScraperAPI's plans and start the free trial](https://www.scraperapi.com/?fp_ref=coupons)

## What Puppeteer does well in data scraping

Puppeteer is Google Chrome team's open-source Node.js library, first released in 2017. It drives a real Chrome or Chromium (and Firefox since v23, via WebDriver BiDi) through the DevTools Protocol, runs the page's JavaScript, and lets your script read the fully rendered DOM. That's the whole reason it exists: `fetch()` plus Cheerio only sees the initial HTML, so if the prices, listings, or reviews you need are rendered client-side by React or Vue, they simply aren't in the response. Puppeteer runs the page for real, so anything a browser can show, your scraper can read.

Getting started is one command — `npm i puppeteer` installs the library plus a matching Chrome for Testing build, so there's no manual browser setup. A minimal working scraper is roughly 15 lines: `page.goto`, a selector, an extraction callback. When you're extracting lists, `page.$$eval` is the workhorse — it runs your function inside the browser against every matching element and returns a serializable array in one round trip, instead of walking element handles one by one.

Three practical points worth internalizing early:

1. **Waiting, not parsing, causes most "flaky scraper" bugs.** The old `page.waitForTimeout()` has been removed from current versions, and fixed sleeps were always the wrong tool. Use `page.waitForSelector()` (30-second default timeout), `page.waitForFunction()`, or the newer `page.locator()` API. For SPAs, `waitUntil: 'networkidle2'` on `page.goto` is the pragmatic middle ground.
2. **The fastest way to scrape a JavaScript-heavy site is often not parsing its DOM.** Capturing the site's own JSON responses with `page.on('response')` skips HTML parsing entirely and gives you cleaner data.
3. **Manage memory deliberately.** Reuse one browser across many pages, close pages when done, and restart the browser every few hundred pages to keep memory flat.

One honest caveat: every page means a full browser rendering pipeline. Puppeteer is the right tool for JavaScript-heavy targets, not a default for every request. If a site serves its data in the initial HTML, a plain HTTP request is cheaper and faster.

## Where Puppeteer alone runs out of road

The wall shows up in two forms: detection and scale.

On detection, a headless browser leaks `navigator.webdriver: true` and a `HeadlessChrome` user agent out of the box. The `puppeteer-extra-plugin-stealth` plugin patches the obvious signals, but modern anti-bot systems — Cloudflare, DataDome, PerimeterX — still detect it, and every Chrome release resets the arms race. Managed challenges and CAPTCHAs from these vendors reliably stop unattended browsers.

On scale, even a script that passes checks will eventually trip rate limits, because hundreds of requests per minute from one server IP is a pattern no site ignores. Solving this yourself means maintaining a proxy pool, retry logic, CAPTCHA solving, and browser fingerprint management — an ongoing engineering project in its own right.

This is the point where a scraping API earns its keep. Your Puppeteer code stays as-is for easy targets; requests that get blocked go to one HTTP endpoint that handles proxies, retries, JS rendering, and the challenge layer. With ScraperAPI, a request looks like:


curl "https://api.scraperapi.com?api_key=YOUR_KEY&render=true&url=https://example.com/page"


Every paid plan includes JS rendering, premium residential and mobile IPs, automatic retries, custom session support, CAPTCHA and anti-bot handling, unlimited bandwidth, and a 99.9% uptime guarantee. The docs recommend a 70-second client timeout — most responses come back in seconds, but hard domains can take up to 60.

> If your Puppeteer scraper already works against easy targets, you don't need to replace it. Route only the blocked or JS-heavy requests through the API and keep the rest cheap.

## ScraperAPI plans and pricing: the full list

Here is everything currently shown on ScraperAPI's pricing page. Annual billing knocks 10% off across the board.

| Plan | Monthly price | Annual (per month) | API credits | Concurrent threads | Buy |
| --- | --- | --- | --- | --- | --- |
| Free | $0 | $0 | 1,000/month | 5 | [Sign up free](https://www.scraperapi.com/?fp_ref=coupons) |
| Hobby | $49 | $44.10 | 100,000 | 20 | [Start with Hobby](https://www.scraperapi.com/?fp_ref=coupons) |
| Startup | $149 | $134.10 | 1,000,000 | 50 | [See Startup pricing](https://www.scraperapi.com/?fp_ref=coupons) |
| Business | $299 | $269.10 | 3,000,000 | 100 | [Open a Business plan](https://www.scraperapi.com/?fp_ref=coupons) |
| Scaling | $475 | $427.50 | 5,000,000 | 200 | [View the Scaling plan](https://www.scraperapi.com/?fp_ref=coupons) |
| Professional | $975 | $877.50 | 10,500,000 | 300 | [Compare high-volume plans](https://www.scraperapi.com/?fp_ref=coupons) |
| Advanced | $1,975 | $1,777.50 | 21,500,000 | 500 | [Compare high-volume plans](https://www.scraperapi.com/?fp_ref=coupons) |
| Enterprise | Custom | Custom | 22,000,000+ | 500+ | [Request custom pricing](https://www.scraperapi.com/?fp_ref=coupons) |

Geotargeting is US and EU only up through Startup; country-level targeting worldwide starts at Business. Every paid plan also carries DataPipeline access, full crawler access, structured data endpoints, and priority support, with dedicated teams and Slack support higher up.

Two things to know before you click buy. First, plan selection happens inside the dashboard after signup — the pricing page is where you register and pick a tier, so the links above all land there. Second, there's a **7-day no-questions-asked refund policy** on paid plans, so the first week on a paid tier is effectively still a test. You can cancel anytime from the dashboard at no charge. 👉 [Open Hobby at the annual rate of $44.10/month](https://www.scraperapi.com/?fp_ref=coupons)

## The real cost per page: credit multipliers, not plan prices

This is the part most reviews gloss over, and it matters more than the number on the pricing page.

A standard page costs 1 credit. But ScraperAPI prices harder domains differently, and the multipliers are public:

- **Standard pages: 1 credit**
- **Amazon: 5 credits**
- **Google and Bing (including subdomains): 25 credits**
- **LinkedIn: 30 credits**
- **Sites behind Cloudflare, Datadome, or PerimeterX: +10 credits per request when ScraperAPI bypasses the protection**

Run the math on Startup's 1,000,000 credits and the spread is dramatic:

| Target type | Cost per request | What 1M credits buys |
| --- | --- | --- |
| Plain HTML page | 1 | ~1,000,000 pages |
| Amazon product page | 5 | ~200,000 pages |
| Google search results | 25 | 40,000 pages |
| LinkedIn | 30 | ~33,000 pages |
| DataDome-protected page with JS rendering | ~20 | ~50,000 pages |

So "a million credits" is a million pages only if your targets are ordinary sites. If most of your puppeteer data scraping targets are search engines or protected domains, the same budget buys tens of thousands of requests — which means you should size your plan from that number, not from the plan name.

Two tools keep this under control:

- **Domain Cost Estimator** (in the dashboard): enter any URL and see exactly what a request to it costs before you spend credits on it.
- **`max_cost` parameter**: set a ceiling per scrape so a single expensive request can't quietly burn through a chunk of your quota.

A sensible workflow: run your first tests on the free credits, check the multipliers for your actual target domains in the estimator, then pick the plan that matches reality. If your estimated usage lands within 10% of a plan boundary, note that there's no overage below the Scaling tier — you'd jump to the next plan entirely. The docs-and-review consensus on this is blunt: if you're close to a plan limit, add `max_cost` and limit JS rendering to the pages that actually need it. 👉 [Test your target domains with 5,000 free credits](https://www.scraperapi.com/?fp_ref=coupons)

## What happens when you hit limits: concurrency, empty credits, surprise bills

These behaviors live in the billing docs rather than the pricing page, and they affect day-to-day usage more than any feature list.

**Exceeding concurrency doesn't cost you anything.** When you exceed your plan's concurrent thread limit, requests get a 429 response and are turned away rather than queued — ScraperAPI explicitly charges nothing for going over. The trade-off is that your code needs retry logic for 429s.

**Running out of credits pauses scraping by default; it doesn't auto-bill you.** When you hit 100% of your credits, scraping stops until renewal. You then have options depending on tier: Hobby through Professional can enable auto-upgrade (opt-in, moves you to the next plan when credits run out — often at a better per-credit rate), and Scaling and above get Pay-As-You-Go, which continues scraping at a fixed per-credit rate up to a monthly spending cap you set. Professional and Advanced monthly plans have Pay-As-You-Go built in. Annual plans can renew early for a fresh batch of credits.

The spending cap deserves a direct quote from the docs: "Your service will not exceed this cap unless you change your settings on the billing page. This is a spending *limit*, not an amount you will be pre-charged." For anyone who's been burned by surprise cloud bills, that's a meaningful design choice — the default behavior is to stop, not to bill.

**Credits don't roll over.** Spiky workloads — say, one large crawl per quarter — will pay for idle months. If that's your pattern, either time your subscription around the crawls or size down and use Pay-As-You-Go on a higher tier for the spikes.

## How ScraperAPI stacks up against the alternatives

Rough positioning, using verified starting prices for comparable scraping APIs: Apify starts around $29/month and Oxylabs around $30/month, Zyte around $100/month, and Bright Data around $499/month. ScraperAPI's $49 entry point sits in the middle of that range, and it's one of the few with both a permanent free tier and a no-card trial.

Where it fits well: you already have working Puppeteer, Scrapy, or Requests code that's getting blocked, and you want a single endpoint to handle proxies, rendering, and retries. Plain HTML pages are its cheapest case at 1 credit apiece.

Where something else may serve you better: if you want ready-made scrapers and hosted runs without writing code, Apify's actor marketplace is built for that. If you need fine-grained proxy control — specific IPs, ASNs, or cities — ScraperAPI deliberately doesn't offer that; you choose country, session, and IP tier, and it expects you to buy raw proxies for anything finer. And if most of your volume is Google or LinkedIn, compare the effective per-request cost carefully everywhere you look, because every provider prices those domains at a premium.

## FAQ

**Can Puppeteer bypass Cloudflare or CAPTCHAs on its own?**
Not reliably. The stealth plugin passes basic checks, but managed challenges from modern anti-bot vendors still stop unattended browsers. For protected targets, a scraping API that owns the challenge layer is usually cheaper than maintaining your own evasion stack.

**Does ScraperAPI work with Puppeteer specifically?**
Yes — the API is just an HTTP endpoint, so it's language-agnostic. Common patterns are routing blocked requests through the API from your Node script, or using ScraperAPI's proxy port so your existing Puppeteer traffic flows through their pool without code changes.

**Is the free plan enough for a real project?**
No. The permanent free tier is 1,000 credits/month with 5 concurrent connections — enough to test and validate multipliers, not to run anything sustained. The 7-day, 5,000-credit trial window is where you should do your serious feasibility testing.

**What happens if I outgrow my plan mid-month?**
Below Scaling, you either enable auto-upgrade (moves you to the next tier, usually at a better price per credit) or contact support for a custom plan. From Scaling up, Pay-As-You-Go keeps you running at a fixed rate up to your spending cap.

**Is there a refund if it doesn't work for my targets?**
Yes — 7 days, no questions asked, handled through support. Combined with the free trial, you can validate your specific domains before any money is at risk.

## Choosing a plan: a decision rule

Match the plan to your verified credit math, not to aspiration:

- **Just validating feasibility** → Free tier plus the 7-day trial. Spend nothing until the Domain Cost Estimator confirms your targets' multipliers.
- **Regular scraping of ordinary sites, under ~100k pages/month** → Hobby at $49 ($44.10 annual). Twenty concurrent threads is plenty for one developer.
- **Volume past 100k pages or several parallel projects** → Startup at $149. Five times the credits and threads for three times the price.
- **Need country-level geotargeting outside US/EU, or 3M+ credits** → Business at $299 is the first tier with both. This is also the step where the use-apify review warns people to pause: with no overage below Scaling, landing 10% over Startup's limit costs you $150 more per month — trim rendering and set `max_cost` first.
- **Mostly Google, LinkedIn, or heavily protected domains** → Multiply your request volume by 25–40 credits before choosing a tier; the plan that fits "a million pages" thinking will disappoint you here.
- **Spiky quarterly crawls** → credits don't roll over, so a smaller plan plus Pay-As-You-Go on a higher tier beats paying for eleven idle months.

One last check before checkout: confirm your annual-versus-monthly choice deliberately — annual saves 10% but locks credits into the year, and monthly plans can be cancelled anytime from the dashboard without charge. 👉 [Pick your plan on ScraperAPI's pricing page](https://www.scraperapi.com/?fp_ref=coupons)
