# Keepa API Endpoints — Detailed Reference

## Products — `/product`

**Purpose:** Retrieve detailed product information (current price, sales rank, history, offers) for one or more ASINs.

**Request format (GET or POST):**
- **GET:** `/product?key=<key>&domain=<domainId>&asin=<ASIN>[&history=0][&stats=1][&offers=<N>][&buybox=1][&variations=1]`
- **POST:** `/product?key=<key>&domain=<domainId>` with JSON body containing asin(s).

**Parameters:**
- `asin` (String): Single ASIN or comma-separated list of up to 100 ASINs.
- `code` (String): Alternative to asin — UPC, EAN, or ISBN-13 code(s). Cannot use both asin and code.
- `domain` (Integer, required): Domain ID.
- `stats` (Integer/String, no extra cost): Include stats field. Value = number of days (e.g. `180`) or date range (`2015-10-20,2015-12-24` or Unix ms timestamps).
- `update` (Integer, default 1): Force refresh if last update older than N hours. `0`=always live (costs 1 extra token if recent), `-1`=no update.
- `history` (0/1, default 1): Set to 0 to exclude csv, salesRanks, monthlySoldHistory for smaller/faster response.
- `days` (Integer): Limit historical data to recent X days.
- `offers` (Integer, 20-100): Retrieve marketplace offers. **Token cost: 6 per found offer page (10 offers/page)**. Replaces base 1-token cost. Response takes 2-20s. **Using offers also unlocks:** Marketplace Offer objects, Buy Box info + history, csv types 7/9/10/16-27/32/33, and the `update` param's reduced-token mode (5 tokens if data is fresh).
- `only-live-offers` (0/1): When used with offers, return only currently live offers.
- `buybox` (0/1, extra cost 2 tokens): Include Buy Box price history and seller ID history.
- `rating` (0/1, up to 1 extra token): Include RATING and COUNT_REVIEWS history in csv.
- `stock` (0/1, extra cost 2 tokens): Include stock quantity data in offers (requires offers param).
- `rental` (0/1): Collect rental prices (US books only, requires offers).
- `videos` (0/1): Include video metadata.
- `aplus` (0/1): Include A+ content.
- `historical-variations` (0/1, extra cost 1 token per product with parent ASIN): Include `historicalVariations` field.
- `code-limit` (Integer): Max products returned per code when using code parameter.
- `keepaMatches` (true): Include Keepa matches.

**Token cost:** 1 per product (base). Additional costs for offers (6/page), buybox (+2), stock (+2), rating (up to +1).

**Cannot track/limited:** Amazon Fresh, eBooks, digital products, movie rentals, bundles, luxury stores, Amazon Haul/Bazaar, Amazon Pantry.

**Response:** Array of Product Objects (see data-objects.md).

---

## Product Finder — `/query`

**Purpose:** Search for products matching multiple criteria (price ranges, brand, seller, reviews, etc.). Powerful filtering with paging.

**Request format (GET or POST):**
- **GET:** `/query?key=<key>&domain=<domainId>&selection=<queryJSON>` (URL-encoded)
- **POST:** `/query?key=<key>&domain=<domainId>` with JSON `selection` body (preferred for complex queries).

**Request body (selection object) — Core parameters:**
- `page` (Integer, default 0): Results page (0-based).
- `perPage` (Integer, default 50): Results per page (50-10,000). Total must not exceed 10,000.
- `sort` (Array of [field, direction]): Up to 3 sort criteria. Example: `[[\"current_SALES\", \"asc\"]]`.

**Filter parameters (all optional, combined with AND logic):**

*Category & product type:*
- `rootCategory` (Long): Filter by root category node ID.
- `salesRankReference` (Long): Filter by sales rank reference category.
- `categories_include` (Long array, max 50): Include products in these categories.
- `categories_exclude` (Long array, max 50): Exclude products in these categories.
- `productType` (Integer array): 0=STANDARD, 1=DOWNLOADABLE, 2=EBOOK, 5=VARIATION_PARENT.

*Text & brand:*
- `title` (String): Keywords that must all appear in product title.
- `brand` (String array): Brand names (case-insensitive).
- `manufacturer` (String array): Manufacturer names.

*Buy box filters:*
- `buyBoxIsAmazon`, `buyBoxIsFBA`, `buyBoxIsUnqualified`, `buyBoxIsPreorder`, `buyBoxIsBackorder`, `buyBoxIsPrimeExclusive` (Boolean).
- `buyBoxSellerId` (String array): Sellers holding new buy box.
- `buyBoxUsedSellerId` (String array): Sellers holding used buy box.

*Availability & flags:*
- `availabilityAmazon` (Integer array): -1=no offer, 0=in stock, 1=pre-order, 2=unknown, 3=back-order, 4=delayed.
- `hasReviews`, `singleVariation`, `hasParentASIN`, `isPrimeExclusive`, `isAdultProduct`, `isHazMat`, `isHeatSensitive`, `isSNS`, `isEligibleForTradeIn`, `isEligibleForSuperSaverShipping`, `launchpad` (Booleans).

**Price/metric pattern filters:**
Format: `{metric}_{PriceType}_lte` or `{metric}_{PriceType}_gte` for range filters.

**Metrics:** `current`, `delta1`, `delta7`, `delta30`, `delta90`, `deltaPercent1`...`deltaPercent90`, `deltaLast`, `avg7`, `avg30`, `avg90`, `avg180`, `avg365`, `lastPriceChange`.

**PriceTypes:** AMAZON, NEW, USED, SALES, LISTPRICE, COLLECTIBLE, REFURBISHED, NEW_FBM_SHIPPING, LIGHTNING_DEAL, WAREHOUSE, NEW_FBA, COUNT_NEW, COUNT_USED, COUNT_REFURBISHED, COUNT_COLLECTIBLE, RATING, COUNT_REVIEWS, BUY_BOX_SHIPPING, USED_NEW_SHIPPING, USED_VERY_GOOD_SHIPPING, USED_GOOD_SHIPPING, USED_ACCEPTABLE_SHIPPING, REFURBISHED_SHIPPING, TRADE_IN.

**Boolean pattern filters:**
- `backInStock_{PriceType}`, `isLowest_{PriceType}`, `isLowest90_{PriceType}` (Boolean).

**Package dimensions:**
- `packageHeight_lte`/`_gte`, `packageLength_lte`/`_gte`, `packageWidth_lte`/`_gte`, `packageWeight_lte`/`_gte`
- `itemHeight_lte`/`_gte`, `itemLength_lte`/`_gte`, `itemWidth_lte`/`_gte`, `itemWeight_lte`/`_gte`
- `variationCount_lte`/`_gte`, `imageCount_lte`/`_gte`

**Buy box stats:**
- `buyBoxStatsAmazon{30,90,180,365}_lte`/`_gte`
- `buyBoxStatsTopSeller{30,90,180,365}_lte`/`_gte`
- `buyBoxStatsSellerCount{30,90,180,365}_lte`/`_gte`

**Seller/content filters:**
- `sellerIds`, `sellerIdsLowestFBA`, `sellerIdsLowestFBM`, `buyBoxSellerId` (String arrays).

**Metadata/attributes (String arrays, case-insensitive):**
- `author`, `binding`, `genre`, `languages`, `publisher`, `platform`
- `activeIngredients`, `specialIngredients`
- `itemTypeKeyword`, `targetAudienceKeyword`, `itemForm`, `scent`, `unitType`, `pattern`, `style`, `material`
- `color`, `size`, `edition`, `format`, `model`, `partNumber`

**Coupon filters:**
- `couponOneTimeAbsolute_lte`/`_gte`, `couponOneTimePercent_lte`/`_gte`, `couponSNSPercent_lte`/`_gte`
- `businessDiscount_lte`/`_gte`

**Product feature filters:**
- `batteriesRequired`, `batteriesIncluded`, `isMerchOnDemand`, `hasMainVideo`, `hasAPlus`, `hasAPlusFromManufacturer` (Boolean).
- `videoCount_lte`/`_gte`

**Time-based filters (Keepa time):**
- `trackingSince_lte`/`_gte`, `publicationDate_lte`/`_gte`, `releaseDate_lte`/`_gte`
- `lastOffersUpdate_lte`/`_gte`, `lastPriceChange_lte`/`_gte`, `lightningEnd_lte`/`_gte`

**Sales/tracking stats:**
- `monthlySold_lte`/`_gte`, `deltaPercent90_monthlySold_lte`/`_gte`
- `variationReviewCount_lte`/`_gte`, `variationRatingCount_lte`/`_gte`
- `flipability30_lte`/`_gte`, `flipability90_lte`/`_gte`, `flipability365_lte`/`_gte`

**Out of stock stats:**
- `outOfStockPercentage90_lte`/`_gte`
- `outOfStockCountAmazon30_lte`/`_gte`, `outOfStockCountAmazon90_lte`/`_gte`

**Additional query parameter:**
- `stats` (0/1): If set to 1, response includes `searchInsights` field with aggregated metrics.

**Response:**
- `asinList` (String array): ASINs matching query.
- `totalResults` (Integer): Total count of matching products.
- `searchInsights` (Search Insights Object, if stats=1): Aggregated KPIs.

**Token cost:** 10 (base) + 1 per 100 ASINs in result.

**Pagination strategy for >10,000 results:**
Use `trackingSince_gte` to page through large result sets:
1. Start with `trackingSince_gte = 0`
2. Sort by `trackingSince` ascending: `"sort": [["trackingSince", "asc"]]`
3. Extract the last returned ASIN's `trackingSince` value
4. Use that value as new `trackingSince_gte` in next request
5. Continue until fewer than `perPage` results returned

---

## Product Searches — `/search`

**Purpose:** Search for Amazon products by keyword, returning results in Amazon search order (excluding sponsored).

**Query:** `GET /search?key=<key>&domain=<domainId>&type=product&term=<searchTerm>`

**Parameters:**
- `term` (String, required): Search term (URL-encoded).
- `domain` (Integer, required): Domain ID.
- `asins-only` (0/1): Return only ASINs.
- `page` (Integer, 0-9): Each page has up to 10 results. **April 20, 2026: `page` removed — results per request increase to 20.**
- `stats`, `update`, `history`, `rating`: Same as Product endpoint.

**Response:** Array of product objects in `products` field, or string array in `asinList` if asins-only=1.

**Token cost:** 10 per result page.

---

## Category Lookup — `/category`

**Purpose:** Retrieve category objects and optionally their parent tree.

**Query:** `GET /category?key=<key>&domain=<domainId>&category=<categoryId>&parents=<0|1>`

**Parameters:**
- `domain` (Integer, required): Domain ID.
- `category` (Long, required): Category node ID. Use `0` for all root categories. Comma-separated up to 10 IDs.
- `parents` (0/1): Include parent category tree.

**Response:** `categories` map of `{categoryId: categoryObject}`.

**Token cost:** 1.

---

## Category Searches — `/search`

**Purpose:** Search for Amazon category names. Returns up to 50 matching categories.

**Query:** `GET /search?key=<key>&domain=<domainId>&type=category&term=<searchTerm>`

**Parameters:**
- `domain` (Integer, required): Domain ID.
- `term` (String, required): Multiple space-separated keywords; all must match. Min 3 chars per keyword.

**Response:** `categories` map.

**Token cost:** 1.

---

## Best Sellers — `/bestsellers`

**Purpose:** Retrieve ASIN list of most popular products by sales in a category or product group.

**Query:** `GET /bestsellers?key=<key>&domain=<domainId>&category=<categoryId>&range=<range>`

**Parameters:**
- `domain` (Integer, required): Not available for Brazil.
- `category` (Long, required): Category node ID or product group name.
- `range` (Integer): 0=current (default), 30=30-day avg, 90=90-day avg, 180=180-day avg.
- `month` & `year` (Integer): Historical best seller list. Last 36 months only. Cannot use with `range`.
- `variations` (0/1): Include all variations.
- `sublist` (0/1): Base list on sub-category rank.

**Notes:** Root category lists up to 500,000 ASINs. Sub-category up to 10,000. Updated hourly.

**Response:** `bestSellersList` field with Best Sellers Object.

**Token cost:** 50 (0 if no match).

---

## Seller Information — `/seller`

**Purpose:** Retrieve seller object using seller ID.

**Query:** `GET /seller?key=<key>&domain=<domainId>&seller=<sellerId>`

**Parameters:**
- `seller` (String, required): Seller ID. Comma-separated up to 100. **Batch not allowed with storefront param.**
- `domain` (Integer, required): Domain ID.
- `storefront` (0/1, extra 9 tokens): Include ASIN list (up to 100K), timestamps, total count.

> **Note:** The `update` parameter was **removed** (Feb 23, 2026). Storefront data is auto-populated from Keepa's internal database.

**Response:** `sellers` map of `{sellerId: sellerObject}`.

**Token cost:** 1 per seller (base). +9 for storefront.

---

## Tracking Products — `/tracking`

**Purpose:** Track product price changes and get notifications. Each tracked product reduces your token refill rate.

**Important:** Tracking reduces refill rate: Regular=0.9 tokens/update/locale, Marketplace=9 tokens/update/locale.

**Types of tracking:**
- **Regular:** Tracks Amazon, New, Used, List, Collectible, Refurbished, Lightning Deal, offer counts, Sales Rank.
- **Marketplace:** All Regular + Warehouse Deals, Buy Box, FBA/FBM, Rating, Prime Exclusive, Review Counts, all Used/Collectible sub-conditions.

**Operations:**

**Add Tracking** (1 token per tracking):
- GET: `/tracking?key=<key>&type=add&tracking=<trackingJSON>`
- POST: `/tracking?key=<key>&type=add` with JSON body (up to 1,000 objects)

**Remove Tracking** (0 tokens):
- Single: `/tracking?key=<key>&type=remove&asin=<ASIN>`
- All: `/tracking?key=<key>&type=removeAll`

**Get Tracking** (0 tokens):
- Single: `/tracking?key=<key>&type=get&asin=<ASIN>`
- List: `/tracking?key=<key>&type=list&asins-only=<0|1>`

**Get Notifications** (0 tokens):
- `/tracking?key=<key>&type=notification&since=<keepaTime>&revise=<0|1>`
- Notifications deleted after 24 hours.

**Get Named Lists** (0 tokens):
- `/tracking?key=<key>&type=listNames`

**Set Webhook** (0 tokens):
- `/tracking?key=<key>&type=webhook&url=<URL>`
- Webhook receives HTTP POST with notification object. Must respond 200. Retry after 15s.

**Named lists:** All tracking requests support `&list=<name>` param (up to 64 chars, up to 100K lists).

---

## Graph Image API — `/graphimage`

**Purpose:** Retrieve a **price history graph chart** (PNG image) for a product. NOT a product photo.

**Returns:** Binary PNG, NOT JSON.

**Query:** `GET api.keepa.com/graphimage?key=<key>&domain=<domainId>&asin=<ASIN>`

**Security:** NEVER embed this URL directly in public-facing content — your API key is in the URL. Always proxy server-side.

**Graph data params** (0=don't draw, 1=draw):

| Param | Description | Default |
|-------|------------|---------|
| `amazon` | Amazon price | 1 |
| `new` | Marketplace New | 1 |
| `used` | Used price | 0 |
| `salesrank` | Sales Rank | 0 |
| `bb` | Buy Box price | 0 |
| `bbu` | Used Buy Box | 0 |
| `fba` | FBA price | 0 |
| `fbm` | FBM price | 0 |
| `ld` | Lightning Deals | 0 |
| `wd` | Warehouse Deals | 0 |
| `pe` | Prime Exclusive | 0 |

**Display params:**
- `range`: Chart range in days (default 90). Suggested: 1, 2, 7, 31, 90, 365.
- `width`: 300-1000 px (default 500).
- `height`: 150-1000 px (default 200).
- `yzoom`: Close-up y-axis zoom (0 or 1, default 1).
- `title`: Show product title (0 or 1, default 1).

**Color params** (hex without `#`): `cBackground`, `cFont`, `cAmazon`, `cNew`, `cUsed`, `cSales`, `cFBA`, `cFBM`, `cBB`, `cBBU`, `cLD`, `cWD`, `cPE`.

**Token cost:** 1. Cached 90 min if identical params.

---

## Browsing Deals — `/deal`

**Purpose:** Find products with recent price or sales rank changes. Returns products updated within the last 12 hours. Up to 10,000 ASINs via paging.

**Request:** GET or POST with queryJSON.

**Token cost:** 5 per request (up to 150 deals).

**queryJSON parameters:**

*Required:*
- `domainId` (Integer): Amazon locale.
- `priceTypes` (Integer[], exactly one entry): 0=AMAZON, 1=NEW, 2=USED, 3=SALES, 5=COLLECTIBLE, 6=REFURBISHED, 7=NEW_FBM_SHIPPING, 8=LIGHTNING_DEAL, 9=WAREHOUSE, 10=NEW_FBA, 18=BUY_BOX_SHIPPING, 19-22=USED sub-conditions, 32=BUY_BOX_USED_SHIPPING, 33=PRIME_EXCL.

*Pagination:*
- `page` (Integer, default 0): Iterate while response returns 150 results.

*Date range:*
- `dateRange` (Integer): 0=Day, 1=Week, 2=Month, 3=90 Days.

*Range filters (use `isRangeEnabled: true`):*
- `deltaRange`, `deltaPercentRange`, `deltaLastRange` (Integer[2]): `[min, max]`. -1 for no bound.
- `salesRankRange`, `currentRange` (Integer[2]).
- `minRating` (Integer).

*Boolean filters (use `isFilterEnabled: true`):*
- `isLowest`, `isLowest90`, `isLowestOffer`, `isHighest`, `isOutOfStock`, `isBackInStock`, `isRisers`
- `isPrimeExclusive`, `mustHaveAmazonOffer`, `mustNotHaveAmazonOffer`
- `hasReviews`, `filterErotic`, `singleVariation`

*Category filters:*
- `excludeCategories`, `includeCategories` (Long[]).

*Attribute filters (String arrays):*
- `material`, `type`, `manufacturer`, `brand`, `productGroup`, `model`, `color`, `size`, `unitType`, `scent`, `itemForm`, `pattern`, `style`, `itemTypeKeyword`, `targetAudienceKeyword`, `edition`, `format`, `author`, `binding`, `languages`, `brandStoreName`, `brandStoreUrlName`, `websiteDisplayGroup`, `websiteDisplayGroupName`, `salesRankDisplayGroup`.

*Search & sort:*
- `titleSearch` (String): Keywords in title (up to 50, all must match).
- `sortType` (Integer): 1=Deal age, 2=Deal score, 3=Title, 4=Sales Rank, 5=Price, 6=Price drop (abs), 7=Price drop (%), 8=Highest Price. Negative to invert.

**Response:** Array of Deal Objects + `categoryIds`, `categoryNames`, `categoryCount` arrays.

---

## Lightning Deals — `/lightningdeal`

**Purpose:** Access current lightning deals. Full list includes past 4 days (active + expired).

**Query:** `GET /lightningdeal?key=<key>&domain=<domainId>[&asin=<ASIN>]`

**Parameters:**
- `domain` (Integer, required).
- `asin` (String, optional): Specific ASIN. If omitted, returns entire list.
- `state` (String): AVAILABLE, WAITLIST, SOLDOUT, WAITLISTFULL, EXPIRED, SUPPRESSED.

**Token cost:** 1 per ASIN, or 500 for full list.

---

## Most Rated Sellers — `/topseller`

**Purpose:** Retrieve seller IDs for the most rated Amazon marketplace sellers. Updated daily, up to 100,000 seller IDs.

**Query:** `GET /topseller?key=<key>&domain=<domainId>`

**Token cost:** 50.

---

## Token Status — `/token`

**Purpose:** Free endpoint to check your current token balance.

**Query:** `GET /token?key=<key>`

**Token cost:** 0.

**Use case:** Useful after `/graphimage` requests (which return PNG, not JSON) to check remaining tokens.
