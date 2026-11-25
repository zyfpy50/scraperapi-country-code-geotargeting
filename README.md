# ScraperAPI country code: how to set it, why `country_code=cn` needs Business, and the 69-country list (the UK is `uk`, not `gb`)

Set `country_code` to a two-letter country code inside the request and ScraperAPI sends that request through a proxy in that country:


country_code=us   → United States
country_code=de   → Germany
country_code=cn   → China


Two things trip people up. The United Kingdom is `uk`, not `gb`. And the code that breaks most first attempts isn't a typo: **Hobby and Startup plans can only geotarget the US and Europe (`us` and `eu`)**. Every other country in ScraperAPI's list, China included, requires Business or Enterprise. That restriction comes from the official geotargeting documentation, not from a sales page, and it's the single fact that decides which plan you actually need.

Below: the plan ladder, working code for Python, cURL and the Go SDK, the codes worth memorising, and what to check when a request comes back looking like it originated somewhere else.

> **Quick version:** scraping US or EU sites, Hobby or Startup covers you. Scraping China, Japan, India, Germany, Brazil or anything else on the list, you need Business. The country list is tied to the plan tier, so upgrading credits on a lower tier doesn't unlock it.
>
> 👉 [Check the current plan tiers and their geotargeting scope](https://www.scraperapi.com/?fp_ref=coupons)

## Which plan unlocks which countries

ScraperAPI splits geotargeting across four plan tiers: Hobby, Startup, Business and Enterprise. The documentation is blunt about it: Business and Enterprise can target 69 countries, Hobby and Startup can only use US and EU geotargeting.

| Plan | Countries you can geotarget | What else the docs confirm | Current pricing |
| --- | --- | --- | --- |
| Hobby | `us`, `eu` only | Lowest tier with geotargeting at all | [Hobby pricing](https://www.scraperapi.com/?fp_ref=coupons) |
| Startup | `us`, `eu` only | Same two-value ceiling as Hobby | [Startup pricing](https://www.scraperapi.com/?fp_ref=coupons) |
| Business | All 69 codes, including `cn`, `jp`, `in`, `de`, `uk` | Required for any country outside `us`/`eu` | [Business pricing](https://www.scraperapi.com/?fp_ref=coupons) |
| Enterprise | All 69 codes | Same country list, sized for higher volume | [Enterprise pricing](https://www.scraperapi.com/?fp_ref=coupons) |

Exact credit allowances and concurrency limits sit next to each plan on the pricing page and change with the tier you pick. The column that decides whether your scraper works at all is the second one: if your target country isn't `us` or `eu`, Hobby and Startup cannot run the request however many credits you buy.

One more number to treat carefully. Older ScraperAPI pages advertise "50+ countries" and "40M+ IPs", and the country pages display per-country IP counts in the hundreds of thousands or millions. Those figures are page snapshots from 2023, and ScraperAPI's own FAQ says pool sizes move month to month. The 69-country list with plan tiers is the more recent, more decision-relevant source.

## How the parameter works in practice

The parameter controls the geolocation of the IP used to make the request. Set it to the country you want the proxy to be from, and the API picks an IP from that country for that request. You don't manage proxy lists.

In Python, using the structure from ScraperAPI's own geotargeting page:

python
import requests

payload = {
    'api_key': 'YOUR_API_KEY',
    'country_code': 'us',
    'url': 'https://example.com'
}

response = requests.get('https://api.scraperapi.com', params=payload)
print(response.text)


The same thing as a single cURL call:

bash
curl "https://api.scraperapi.com?api_key=YOUR_API_KEY&country_code=uk&url=https://example.com"


Through the Go SDK, the country sits alongside your other request options:

go
s.Get(
    "https://example.com",
    scraperapi.WithCountryCode(scraperapi.CountryCodeChina),
    scraperapi.WithPremium(),
)


Proxy port mode takes the same flag. All three access methods (API endpoint, SDK, proxy port) draw from the same pool and rotate the proxy after every request, so consecutive calls in the same job can leave from different IPs within the same country. If you need continuity, the SDK also exposes a session number option.

## The country codes worth remembering

| Country | Value | Plan needed |
| --- | --- | --- |
| United States | `us` | Hobby and higher |
| Europe (general) | `eu` | Hobby and higher |
| United Kingdom | `uk` | Business and higher |
| Canada | `ca` | Business and higher |
| Germany | `de` | Business and higher |
| France | `fr` | Business and higher |
| Spain | `es` | Business and higher |
| Italy | `it` | Business and higher |
| Netherlands | `nl` | Business and higher |
| Japan | `jp` | Business and higher |
| India | `in` | Business and higher |
| China | `cn` | Business and higher |
| Australia | `au` | Business and higher |
| Brazil | `br` | Business and higher |
| Mexico | `mx` | Business and higher |
| Singapore | `sg` | Business and higher |
| Hong Kong | `hk` | Business and higher |
| Taiwan | `tw` | Business and higher |

The remaining codes follow the same two-letter pattern (Austria `at`, Switzerland `ch`, South Korea `kr`, Poland `pl`, South Africa `za` and so on); the full 69-row table lives in ScraperAPI's geotargeting documentation. `eu` is a region-level value rather than a country, which makes it the right choice when you need "somewhere in Europe" and don't care which country.

If you use the Go SDK, don't take its constant list as the full set. The package defines only twelve: `us`, `ca`, `uk`, `de`, `fr`, `es`, `br`, `mx`, `in`, `jp`, `cn`, `au`. The function accepts a plain string, so for anything outside those twelve you pass the code from the documentation table. The package's last release is v0.0.1 from May 2022, which explains the gap.

## Why your request still didn't come from the right country

Four causes, roughly in order of how often they're the answer.

**Your plan doesn't include that country.** This is the big one. Tutorials show `country_code=cn` or `country_code=jp`, you copy it, and the request either fails or comes back with US content because the account is on Hobby or Startup. The docs only allow `us` and `eu` below Business.

**You used the wrong code.** `gb` is not a valid value; the UK is `uk`. Codes are lowercase.

**You got datacenter IPs when you needed residential.** A standard request with only `country_code` set goes through datacenter IPs for that country, with residential as a fallback. On the China setup, ScraperAPI states that adding `premium=true` sends the request through its premium residential and mobile pools in that country instead. Which IP class you land on affects both what the site sees and how likely you are to get blocked, so if a site is filtering datacenter ranges, the country code alone won't save you.

**You're assuming the IP count is fixed.** ScraperAPI says the number of IPs per country shifts with demand and that it keeps hundreds of thousands of China-based IPs available, scaling up as needed. Capacity is managed dynamically, not a fixed inventory you can pin to a number.

If your target country is in the 67 that need Business, that's the decision point:

👉 [See which plan includes the countries you need to scrape](https://www.scraperapi.com/?fp_ref=coupons)

## What geotargeting is not for

ScraperAPI's own FAQ is explicit: country proxies are built for scraping sites that serve different data depending on the visitor's IP, and they "aren't a good option" for VPNs or streaming services such as Netflix, BBC iPlayer or Amazon Prime. If your goal is watching region-locked video rather than collecting HTML, this is the wrong tool and no plan tier fixes it.

The legitimate use cases are the ones the documentation names: e-commerce price and availability checks by market, search results as they appear in a given country, and localization testing where you need to confirm what a page actually renders for a local visitor.

## FAQ

**Is the UK code `uk` or `gb`?**
`uk`. `gb` isn't in ScraperAPI's table.

**Can I just use `eu` for European countries?**
Yes, if you don't need a specific country. `eu` is a general Europe value and, along with `us`, is one of only two codes available on Hobby and Startup. For per-country targeting such as `de` or `fr`, you need Business.

**Which plan do I need for Chinese IPs?**
Business or Enterprise. `cn` is listed as Business Plan and higher.

**What's the difference between a standard and a premium request?**
The IP class. Standard requests route through datacenter IPs (with residential as backup). Adding `premium=true` routes through premium residential and mobile pools. Premium is the setting to try when a site blocks datacenter ranges.

**Do all three access methods support geotargeting?**
Yes. The API endpoint, the SDK and the proxy port all use `country_code` and rotate proxies per request.

## Choosing your plan

Work backwards from the country list. Scraping US or EU sites only: Hobby or Startup handles geotargeting, and the choice between them comes down to monthly volume and concurrency. Scraping any single country outside `us`/`eu`, whether that's China, Japan, India, Germany or Brazil: Business is the floor, and no amount of extra credits on a cheaper tier substitutes for it. High-volume operations with custom requirements go to Enterprise.

Before you pay, confirm two things on the plan page: that the tier includes every country on your target list, and that the monthly credit allowance matches the number of pages you plan to pull, since every request draws from that same pool.

👉 [Compare ScraperAPI plans and start with the tier that covers your target countries](https://www.scraperapi.com/?fp_ref=coupons)
