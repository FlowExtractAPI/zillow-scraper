# Zillow Scraper

Collect property listings from Zillow — **for sale**, **for rent**, or **recently sold** — anywhere in the United States.

Type a location. That is the whole setup. You get back every matching listing as structured data: price and price history, address and coordinates, beds, baths, interior and lot size, property type, Zillow's value and rent estimates, tax-assessed value, days on market, the listing agent and brokerage, and **every photo on the listing** — not just the thumbnail.

---

## ✨ What makes this one different

| | |
|---|---|
| 🔎 **No URL wrangling** | Enter `Austin, TX` or `78704`. No need to build a search on Zillow and copy a URL — though you can still paste one if you prefer. |
| 🗺️ **Large areas are handled** | Zillow serves only so many listings per search, however many exist. Big areas are split into smaller ones automatically and each is searched, so a whole city or county comes back complete instead of capped. |
| 📸 **The full photo gallery** | Around **30 photos per listing** on average, at full size — not one thumbnail. No separate image scraper needed. |
| 🎛️ **Filters built in** | Price, beds, baths, square feet, lot size, year built, HOA, property type, keywords, sort order — set them here, not on Zillow. |
| 💸 **Pay for what you keep** | A small run fee, then a per-listing charge — no charge for empty pages, duplicates, or out-of-area rows the actor filters out. |
| ♻️ **Resumes itself** | If a long run is interrupted, restarting continues from where it stopped instead of re-collecting — and re-charging for — what you already have. |
| 📐 **Output that matches its docs** | Every field listed below is a field the actor emits, checked automatically before release. |

---

## 🚀 Quick start

**Everything for sale in one city**

```json
{
  "location": "Austin, TX",
  "listingType": "buy",
  "maxResults": 500
}
```

**Rentals under $2,500 with at least 2 bedrooms**

```json
{
  "location": "78704",
  "listingType": "rent",
  "maxPrice": 2500,
  "minBeds": 2,
  "maxResults": 200
}
```

**Recently sold houses across several ZIP codes**

```json
{
  "locations": ["78701", "78702", "78703"],
  "listingType": "sold",
  "homeTypes": ["houses"],
  "maxResults": 300
}
```

**A search you already built on Zillow**

```json
{
  "startUrls": [
    { "url": "https://www.zillow.com/austin-tx/?searchQueryState=..." }
  ],
  "maxResults": 1000
}
```

---

## ⚙️ Input

### What to search

| Field | Type | Description |
|---|---|---|
| `location` | string | A city, ZIP code, county, or neighbourhood — `"Austin, TX"`, `"78704"`, `"Travis County, TX"`. |
| `listingType` | string | `buy` (for sale), `rent`, or `sold`. Default `buy`. |
| `locations` | array | More places to search in the same run, same filters applied to each. |
| `maxResults` | integer | Cap per location or URL. Default `100`. `0` means no limit. This is also your cost ceiling. |

### Filters — all optional

Price (`minPrice`, `maxPrice`), bedrooms (`minBeds`, `maxBeds`), bathrooms (`minBaths`, `maxBaths`), interior size (`minSqft`, `maxSqft`), lot size (`minLotSize`, `maxLotSize`), year built (`minYearBuilt`, `maxYearBuilt`), `maxHoaFee`, `homeTypes`, `keywords`, `sortBy`, `has3dTour`, `openHousesOnly`, and for-sale toggles `foreclosure`, `auction`, `comingSoon`, `agentListed`, `ownerPosted`, `newConstruction`.

Leave anything blank to skip it. For rentals and sold searches, the for-sale-only toggles are ignored.

### Advanced

| Field | Type | Description |
|---|---|---|
| `startUrls` | array | Zillow search URLs copied from your browser. Every filter you set on Zillow carries over — use this for searches the filters above cannot express. Can be combined with locations. |
| `extractionMethod` | string | `SUBDIVIDE` (default) splits areas too large for one search. `PAGINATION` runs a single search and stops at Zillow's own limit — faster for small areas. |

There is no proxy setting to configure. Network routing is handled for you.

---

## 📦 Output

One record per property. Every field below appears on every record; a field with no value for a given listing is `null` rather than missing.

**Identity**

| Field | Type | Description |
|---|---|---|
| `zpid` | string | Zillow property id — the stable unique identifier for a listing |
| `propertyId` | string | Internal listing id for this search result |
| `palsId` | string | Zillow internal listing id |
| `detailUrl` | string | Absolute URL of the property page on Zillow |

**Status**

| Field | Type | Description |
|---|---|---|
| `statusType` | string | Listing status (FOR_SALE, FOR_RENT, RECENTLY_SOLD, …) |
| `statusText` | string | Human-readable status, e.g. "Active" or "House for rent" |
| `homeStatus` | string | Detailed status code, e.g. FOR_SALE, PENDING, SOLD |
| `homeStatusForHDP` | string | Status as shown on the property page |
| `rawHomeStatusCd` | string | Raw status code from the listing feed |
| `marketingStatus` | string | Simplified marketing status, e.g. "For Sale by Agent" |
| `listingCategory` | string | Broad category, e.g. ForSale or ForRent |
| `listingCategoryText` | string | Readable category, e.g. "For Sale (Broker)" |
| `listingSubTypes` | array | Listing sub-types that apply, e.g. ["FSBA"] for sale-by-agent |
| `isRelaxedResult` | boolean | True when Zillow returned this as a nearby match rather than a direct hit |

**Address & location**

| Field | Type | Description |
|---|---|---|
| `address` | string | Full address as one string |
| `streetAddress` | string | Street line only |
| `city` | string | City |
| `state` | string | Two-letter state code |
| `zipcode` | string | ZIP code |
| `isUndisclosedAddress` | boolean | True when the exact address is withheld |
| `latitude` | number | Latitude |
| `longitude` | number | Longitude |

**Price**

| Field | Type | Description |
|---|---|---|
| `price` | number | Price as a number — sale price, or monthly rent for rentals |
| `priceText` | string | Price exactly as displayed, e.g. "$1,175,000" or "$1,895/mo" |
| `priceForHDP` | number | Price used on the property page |
| `priceChange` | number | Change since the last price update, negative for a cut |
| `priceReduction` | string | Readable price-cut summary, e.g. "$25,000 (Apr 3)" |
| `datePriceChanged` | string | ISO timestamp of the last price change |
| `currency` | string | Currency code |
| `country` | string | Country code |
| `shouldShowRequestOnPrice` | boolean | True when the listing hides its price behind a request |
| `shouldShowZestimateAsPrice` | boolean | True when the estimate stands in for a list price |

**Valuation**

| Field | Type | Description |
|---|---|---|
| `zestimate` | number | Zillow's estimated market value |
| `rentZestimate` | number | Zillow's estimated monthly rent |
| `taxAssessedValue` | number | Tax-assessed value |

**Property details**

| Field | Type | Description |
|---|---|---|
| `homeType` | string | Property type, e.g. SINGLE_FAMILY, CONDO, TOWNHOUSE, APARTMENT |
| `bedrooms` | number | Number of bedrooms |
| `bathrooms` | number | Number of bathrooms |
| `livingArea` | number | Interior area in square feet |
| `lotAreaValue` | number | Lot size, in the unit given by lotAreaUnit |
| `lotAreaUnit` | string | Unit for lotAreaValue — sqft or acres |
| `daysOnZillow` | number | Days listed on Zillow (-1 when unknown) |
| `timeOnZillow` | number | Milliseconds listed on Zillow |

**Flags**

| Field | Type | Description |
|---|---|---|
| `isZillowOwned` | boolean | True for a Zillow-owned home |
| `isFeatured` | boolean | True for a featured listing |
| `isShowcaseListing` | boolean | True for a Showcase listing |
| `isPremierBuilder` | boolean | True for a paid new-construction builder listing |
| `isNonOwnerOccupied` | boolean | True when the owner does not live there |
| `isPreforeclosureAuction` | boolean | True for a pre-foreclosure auction |
| `isUnmappable` | boolean | True when the property has no usable map position |
| `has3DModel` | boolean | True when a 3D home tour is available |
| `hasVideo` | boolean | True when a video tour is available |

**Listing agent**

| Field | Type | Description |
|---|---|---|
| `brokerName` | string | Listing brokerage |
| `agentName` | string | Listing agent, usually with licence number |
| `agentPhotoUrl` | string | Listing agent photo URL |

**Media**

| Field | Type | Description |
|---|---|---|
| `mainImage` | string | Primary listing photo URL |
| `photos` | array | Every listing photo URL, full size |
| `photoCount` | number | Number of photos in `photos` |
| `hasImage` | boolean | True when the listing has at least one photo |

**Record metadata**

| Field | Type | Description |
|---|---|---|
| `_source` | string | Origin tag for this record — always zillow_scraper |
| `source_url` | string | The search this record came from |
| `scrapedAt` | string | ISO timestamp of when the record was collected |
| `proxyMode` | string | Network route used for the request |
| `charged` | boolean | True when this result was billed; see chargeSkipReason when false |
| `chargeSkipReason` | string | Why this result was delivered without being billed. Absent on billed results. |
| `_error` | string | Only on an explanatory row when a search could not run — a bad location, for example. Listing rows never carry it, so filter on it to separate the two. |

### Sample record

```json
{
  "zpid": "97557004",
  "detailUrl": "https://www.zillow.com/homedetails/27006-Rocky-Rim-San-Antonio-TX-78266/97557004_zpid/",
  "statusType": "FOR_SALE",
  "statusText": "House for sale",
  "address": "27006 Rocky Rim, San Antonio, TX 78266",
  "city": "San Antonio",
  "state": "TX",
  "zipcode": "78266",
  "latitude": 29.703697,
  "longitude": -98.29464,
  "price": 1175000,
  "priceText": "$1,175,000",
  "priceChange": -25000,
  "priceReduction": "$25,000 (Apr 3)",
  "zestimate": 1183300,
  "rentZestimate": 5017,
  "taxAssessedValue": 1175610,
  "homeType": "SINGLE_FAMILY",
  "bedrooms": 4,
  "bathrooms": 5,
  "livingArea": 3672,
  "lotAreaValue": 1.09,
  "lotAreaUnit": "acres",
  "daysOnZillow": 46,
  "brokerName": "Keller Williams Heritage",
  "agentName": "Marcus Soliz TREC #786473",
  "photoCount": 42,
  "photos": ["https://photos.zillowstatic.com/fp/…-p_e.jpg", "…"]
}
```

---

## 💰 Pricing

Two charges: a **$0.005 run fee** when the actor starts, then a **per-property charge** for each listing saved. Nothing is charged for pages that return nothing, for duplicates, or for out-of-area rows the actor filters out.

| Apify plan | Price per property | 1,000 listings |
|---|---|---|
| Free | $0.0020 | $2.00 |
| Bronze | $0.0016 | $1.60 |
| Silver | $0.0013 | $1.30 |
| Gold / Platinum / Diamond | $0.0010 | $1.00 |

Plus **$0.005 per run** when the actor starts. On a typical run that is a rounding error — collecting 1,000 listings on Bronze costs $1.605 in total.

Set `maxResults` to cap a run, or set a maximum spend on the run itself: the actor subtracts the run fee first and then collects only what your limit actually covers, so it stops at a planned number instead of running into its ceiling.

---

## 💡 Good to know

- **Duplicates are removed within a run.** When a large area is split, neighbouring pieces overlap; the same listing is saved and billed once.
- **`maxResults` applies per location or URL**, not to the run as a whole. Three locations at 100 each is up to 300 listings.
- **Sorting affects what you get when capping.** With `maxResults` set, `sortBy: "days"` gives the newest listings rather than an arbitrary slice.
- **Rentals put monthly rent in `price`.** `priceText` keeps the displayed form, e.g. `"$1,895/mo"`.
- **This actor reads search results.** Fields shown only on an individual property page — full description, price history table, school ratings, tax history — are not included.
- **A bad location does not fail the run.** You get a successful run with one row explaining what went wrong, so a typo never costs you a failed run.

---

## 🎯 Perfect for

* 🧑‍💼 **Agents and brokerages** — build and monitor listing lists across a market
* 📈 **Investors and analysts** — track price cuts, days on market, and rent-to-value across an area
* 🏢 **Proptech teams** — feed a CRM, valuation model, or search product
* 🗺️ **Researchers** — assemble complete regional datasets, not a capped sample
* 🤖 **Developers** — one call from a workflow, straight to JSON or CSV

---

## 📞 Support

- 🌐 **Website**: [flowextractapi.com](https://flowextractapi.com)
- 📧 **Email**: [flowextractapi@outlook.com](mailto:flowextractapi@outlook.com)
- 🙋 **Apify Profile**: [FlowExtract API](https://apify.com/dz_omar?fpr=smcx63)
- 💬 **GitHub**: [FlowExtractAPI](https://github.com/FlowExtractAPI)
- 💼 **LinkedIn**: [flowextract-api](https://www.linkedin.com/in/flowextract-api/)
- 🐦 **X**: [@FlowExtractAPI](https://x.com/FlowExtractAPI)
- 📱 **Facebook**: [flowextractapi](https://www.facebook.com/flowextractapi)
- 🎵 **TikTok**: [@flowextractapi](https://www.tiktok.com/@flowextractapi)

## 🌟 Related actors by FlowExtract API

### 🏠 Real estate
- **[realestate.com.au Scraper](https://apify.com/dz_omar/realestate-com-au-scraper?fpr=smcx63)** — Australian property listings
- **[ImmoScout24 Scraper](https://apify.com/dz_omar/immobilienscout24-scraper?fpr=smcx63)** — German property listings
- **[Idealista Scraper](https://apify.com/dz_omar/idealista-scraper?fpr=smcx63)** — Spanish, Portuguese and Italian listings

### 🛠️ Developer tools
- **[AI Lead Extractor](https://apify.com/dz_omar/ai-lead-extractor?fpr=smcx63)** — turn any page into structured contact data

---

### ⚖️ Legal & compliance

- **Public data only** — processes publicly available listing information
- **Respects source limits** — operates within the source site's rate limits and terms of use
- **No personal data storage** — nothing is retained beyond your own dataset
- **Commercial use** — suitable for business intelligence and research applications
- No affiliation with or endorsement by Zillow is implied.

*Zillow Scraper — by FlowExtract API. Turn any website into structured data.*
