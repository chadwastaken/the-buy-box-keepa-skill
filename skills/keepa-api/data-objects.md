# Keepa API Data Objects — Detailed Reference

## Product Object

**Returned by:** `/product` and `/search` (product type) endpoints.

**IMPORTANT:** Always evaluate `productType` first — it determines what data is available:
- 0=STANDARD (all data), 1=DOWNLOADABLE (no marketplace offers), 2=EBOOK (no marketplace offers), 3=INACCESSIBLE (limited data), 4=INVALID (no current data), 5=VARIATION_PARENT (only variations and sales rank set).

**Identity & classification:**
- `asin` (String), `domainId` (Integer), `title` (String, may contain HTML).
- `productType` (Integer), `type` (String): Item's product type.
- `rootCategory` (Long): Root category node ID (0 if unknown).
- `categories` (Long[]), `categoryTree` (Object[]): `{catId, name}`.
- `parentAsin` (String): Parent ASIN if variation. `parentAsinHistory` (String[]).
- `variations` (Object[]): Each has `asin`, optional swatch image, `attributes` array of `{dimension, value}`. Up to 4,000.
- `historicalVariations` (String[]): Out-of-stock variation ASINs (requires `historical-variations` param).
- `frequentlyBoughtTogether` (String[]): 1-2 related ASINs.
- `bundleItems` (String[]): Bundle component ASINs.

**Product codes:**
- `eanList`, `upcList`, `gtinList` (String[]): First index is primary.

**Brand & manufacturer:**
- `manufacturer`, `brand`, `brandStoreName`, `brandStoreUrl`, `brandStoreUrlName` (Strings).
- `productGroup` (String), `partNumber` (String).
- `websiteDisplayGroup` (String), `websiteDisplayGroupName` (String).
- `salesRankDisplayGroup` (String).

**Descriptive attributes:**
- `binding`, `color`, `size`, `edition`, `format`, `model`, `scent` (Strings).
- `shortDescription`, `description` (max 4000 chars, may contain HTML).
- `features` (String[]): Up to 5 bullet points.
- `activeIngredients`, `specialIngredients`, `itemForm`, `itemTypeKeyword` (Strings).
- `targetAudienceKeyword`, `style`, `pattern`, `includedComponents` (Strings).
- `materials` (String[]), `recommendedUsesForProduct` (String), `specificUsesForProduct` (String[]).
- `contributors` (String[][]): Each `[name, role]`. `languages` (String[][]).
- `safetyWarning`, `productBenefit`, `ingredients` (Strings).
- `hazardousMaterials` (Object[]): Each has `aspect` and `value`.

**Dates (Keepa Time minutes):**
- `trackingSince`, `listedSince`, `lastUpdate`, `lastRatingUpdate`, `lastPriceChange`, `lastEbayUpdate`, `lastStockUpdate`, `lastSoldUpdate`, `lastBusinessDiscountUpdate` (Integers).
- `publicationDate`, `releaseDate` (Integer): Format YYYY, YYYYMM, or YYYYMMDD. -1 if unavailable.

**Dimensions (mm) and weight (grams):** 0 or -1 if unavailable.
- `packageHeight`, `packageLength`, `packageWidth`, `packageWeight`, `packageQuantity`
- `itemHeight`, `itemLength`, `itemWidth`, `itemWeight`
- `numberOfItems`, `numberOfPages` (Integer, -1 if unavailable).

**Images:**
- `imagesCSV` (String): CSV of image names. Full URL: `https://m.media-amazon.com/images/I/<name>`. **Removed April 20, 2026.**
- `images` (Image Object[]): Newer format. Each: `l` (large filename), `lH`/`lW` (px), `m` (medium filename), `mH`/`mW` (px).

**Availability & flags:**
- `availabilityAmazon` (Integer): -1=no offer, 0=in stock, 1=pre-order, 2=unknown, 3=back-order, 4=delayed.
- `availabilityAmazonDelay` (Integer[]): Delay range in hours.
- `isAdultProduct`, `isHeatSensitive`, `isMerchOnDemand`, `isHaul`, `launchpad` (Booleans).
- `isEligibleForTradeIn`, `isEligibleForSuperSaverShipping`, `isSNS` (Booleans).
- `newPriceIsMAP` (Boolean), `returnRate` (Integer: null=average, 1=low, 2=high).
- `audienceRating` (String), `urlSlug` (String).
- `batteriesRequired`, `batteriesIncluded` (Booleans).

**Pricing & fees:**
- `fbaFees` (Object): `{lastUpdate, pickAndPackFee}` in cents.
- `variableClosingFee` (Integer), `referralFeePercentage` (Double).
- `competitivePriceThreshold`, `suggestedLowerPrice` (Integer): In cents.
- `businessDiscount` (Integer): Highest business discount %.

**Coupons & Deals:**
- `coupon` (Integer[2]): `[oneTimeCoupon, snsCoupon]`. Positive=absolute cents, negative=percentage, 0=none. Updated only with `offers` param.
- `couponHistory` (Integer[]): `[keepaTime, oneTime, sns, ...]`.
- `promotions` (Object[]): Each has `type`, `amount`, `discountPercent`, `snsBulkDiscountPercent`.
- `deals` (Deal Details Object[]): Active buy box deals. Each has `accessType`, `dealType`, `badge`.

**Sales rank:**
- `salesRankReference` (Long): Category node ID. -1=unavailable, -2=launchpad.
- `salesRankReferenceHistory` (Long[]): `[keepaTime, categoryId, ...]`.
- `salesRanks` (Object): Map of `{categoryId: [keepaTime, rank, ...]}`.

**Monthly sold:**
- `monthlySold` (Integer): Bought-past-month metric (bracketed, e.g. 1000 = "1000+").
- `monthlySoldHistory` (Integer[]): `[keepaTime, monthlySold, ...]`.

**Books/media:**
- `formats` (Object[]): Each `{asin, format}`.
- `unitCount` (Object): `{unitValue, unitType, eachUnitCount}`.

**Videos & A+ content** (require `videos`/`aplus` params):
- `videos` (Object[]): Each `{title, image, duration, creator, name, url}`.
- `aPlus` (Object[]): Each has `module` array and `fromManufacturer` boolean.

**eBay:**
- `ebayListingIds` (Long[]): `[newListingId, usedListingId]`.

**Offers (requires `offers` param):**
- `offers` (Marketplace Offer[]): See below.
- `liveOffersOrder` (Integer[]): Indices into `offers` array in Amazon listing order.
- `offersSuccessful` (Boolean), `isRedirectASIN` (Boolean).

**Buy Box history (requires `offers` or `buybox` param):**
- `buyBoxSellerIdHistory` (String[]): `[keepaTime, sellerId, ...]`. "-1"=suppressed, "-2"=unidentified.
- `buyBoxUsedHistory` (String[]): `[keepaTime, sellerId, condition, isFBA, ...]`.
- `buyBoxEligibleOfferCounts` (Integer[8]): `[NewFBA, NewFBM, UsedFBA, UsedFBM, CollFBA, CollFBM, RefurbFBA, RefurbFBM]`.

**Statistics (requires `stats` param):**
- `stats` (Statistics Object): See Statistics Object below.

**CSV price history:**
- `csv` (Integer[][]): Indexed by CSV Type (0-35). Second dimension: `[keepaTime, value, ...]` or for *_SHIPPING types: `[keepaTime, price, shippingCost, ...]`. Price -1 = out of stock/no data. Prices in cents.

**Reviews (requires `rating` param):**
- `reviews` (Object): `{lastUpdate, ratingCount: [keepaTime, count, ...], reviewCount: [keepaTime, count, ...]}`.

---

## Statistics Object

**Found in:** Product Object's `stats` field (requires `stats` param).

**IMPORTANT: Arrays are ZERO-INDEXED by CSV Type.** `stats.current[0]` = Amazon price, `stats.current[3]` = Sales Rank.

**Fields (arrays indexed by CSV Type 0-35):**
- `current` (Integer[]): Current value/price. -1 = unavailable. **Most commonly used for current prices.**
- `avg` (Integer[][]): Weighted mean. `[dateRange][csvType]` where 0=since tracked, 1=30d, 2=90d, 3=180d, 4=365d.
- `avg30`, `avg90`, `avg180`, `avg365` (Integer[]): Shortcuts for weighted means.
- `min`, `max` (Integer[][]): Same `[dateRange][csvType]` indexing.
- `minInInterval`, `maxInInterval` (Integer[]): Min/max in requested interval.
- `atIntervalStart` (Integer[]): Value at start of interval.
- `isLowest` (Boolean[]): All-time lowest per price type.
- `isLowest90` (Boolean[]): Lowest in last 90 days.
- `outOfStockPercentageInInterval` (Integer[]): % out of stock during interval. -1 = insufficient data.
- `outOfStockPercentage30`, `outOfStockPercentage90`, `outOfStockPercentage180`, `outOfStockPercentage365` (Integer[]).
- `salesRankDrops30`, `salesRankDrops90`, `salesRankDrops180`, `salesRankDrops365` (Integer): Sales rank drop counts (indicates sales activity).
- `lastOffersUpdate`, `lastBuyBoxUpdate` (Integer): Keepa time.
- `totalOfferCount`, `retrievedOfferCount` (Integer).
- `lightningDealInfo` (Integer[]): `[startDate, endDate]`. endDate=-1 = upcoming. null = no deal.

**Buy Box stats:**
- `buyBoxPrice` (Integer): Current Buy Box price in cents (-1 if none).
- `buyBoxShipping` (Integer): Shipping cost.
- `buyBoxSellerId` (String): "-1"=suppressed, "-2"=unidentified, null=unavailable.
- `buyBoxIsFBA`, `buyBoxIsAmazon`, `buyBoxIsMAP`, `buyBoxIsUnqualified`, `buyBoxIsShippable`, `buyBoxIsPreorder`, `buyBoxIsBackorder`, `buyBoxIsPrimeExclusive`, `buyBoxIsPrimeEligible` (Booleans).
- `buyBoxShippingTime` (Integer[]): `[minHours, maxHours]`.
- `buyBoxShippingCountry` (String): Default shipping country code.
- `buyBoxAvailabilityMessage` (String): **April 20, 2026: becomes enum** — `IN_STOCK`, `BACKORDER_NO_ETA`, `BACKORDER_WITH_ETA`.
- `buyBoxSavingBasis` (Integer): Strikethrough price in cents. **April 20, 2026: requires `offers` param.**
- `buyBoxSavingBasisType` (String): "LIST_PRICE" or "WAS_PRICE". **April 20, 2026: requires `offers` param.**
- `buyBoxSavingPercentage` (Integer). **April 20, 2026: requires `offers` param.**
- `buyBoxAvgNewPrice`, `buyBoxAvgNewShipping`, `buyBoxAvgPrice` (Integer).
- `buyBoxStats` (Object): Per seller — `{avgNewOfferCount, avgPrice, isFBA, lastSeen, percentageWon}`.
- `buyBoxUsedStats` (Object): Same format for Used Buy Box.
- `buyBoxUsedPrice`, `buyBoxUsedShipping` (Integer).
- `buyBoxUsedSellerId` (String), `buyBoxUsedIsFBA` (Boolean).
- `buyBoxUsedCondition` (Integer): 2=Like New, 3=Very Good, 4=Good, 5=Acceptable.
- `sellerIdsLowestFBA`, `sellerIdsLowestFBM` (String[]).
- `offerCountFBA`, `offerCountFBM` (Integer): -2 if unavailable.
- `stockAmazon`, `stockBuyBox` (Integer): Requires `stock` param.

**Common usage:**
```
stats.current[0]    // Current Amazon price (cents)
stats.current[1]    // Current marketplace New price
stats.current[3]    // Current Sales Rank
stats.buyBoxPrice   // Current Buy Box price
stats.avg90[1]      // 90-day average New price
```

---

## Marketplace Offer Object

**Found in:** Product Object's `offers` array (requires `offers` param).

**Important:** Check `lastSeen` for freshness. Use `liveOffersOrder` from Product Object to identify current offers. Identical offers deduplicated to cheapest.

**Fields:**
- `offerId` (Integer): Unique ID within this product.
- `lastSeen` (Integer): Keepa time of last update.
- `sellerId` (String): Merchant's seller ID.
- `isPrime`, `isFBA`, `isMAP`, `isShippable`, `isPreorder`, `isWarehouseDeal`, `isAmazon`, `isPrimeExcl` (Booleans).
- `condition` (Integer): 0=Unknown, 1=New, 2=Used-Like New, 3=Used-Very Good, 4=Used-Good, 5=Used-Acceptable, 6=Refurbished, 7-10=Collectible sub-conditions, 11=Rental.
- `minOrderQty` (Integer): Only present if > 1.
- `conditionComment` (String): Seller's condition description.
- `coupon` (Integer): Positive=absolute cents, negative=percentage.
- `couponHistory` (Integer[]): `[keepaTime, coupon, ...]`.
- `offerCSV` (Integer[]): `[keepaTime, price, shippingCost, ...]`. -2=unknown, -1=unshippable, 0=free shipping.
- `stockCSV` (Integer[]): `[keepaTime, stockQty, ...]` (requires `stock` param).
- `primeExclCSV` (Integer[]): `[keepaTime, price, ...]`. -1=no exclusive price.
- `lastStockUpdate` (Integer): Keepa time.
- `offerDuplicates` (Object[]): Excluded identical offers with `price`, `shipping`, `conditionComment`.

---

## Seller Object

**Returned by:** `/seller` endpoint. Not available for handmade-only sellers.

**Identity:**
- `sellerId`, `sellerName`, `businessName` (Strings).
- `domainId` (Integer).
- `address` (String[]): Last entry is 2-letter country code.
- `tradeNumber`, `vatID`, `phoneNumber`, `businessType`, `shareCapital`, `representative`, `email` (Strings, only if available).
- `customerServicesAddress` (String[]).

**Timestamps (Keepa Time):**
- `trackingSince`, `lastUpdate`, `lastRatingUpdate` (Integers).

**Ratings:**
- `ratingCount` (Integer[4]): Counts for 30, 90, 365 days, and lifetime.
- `positiveRating` (Integer[4]): Positive % (4-5 star) for same intervals.
- `negativeRating` (Integer[4]): Negative % (1-2 star).
- `neutralRating` (Integer[4]): Neutral % (3 star).

**Recent Feedback:**
- `recentFeedback` (Feedback Object[]): Up to 5 most recent. Each: `date` (Keepa time), `rating` (10-50 scale), `isStriked` (Boolean).

**Flags:**
- `hasFBA` (Boolean): Currently has FBA listings.

**Competition & Buy Box:**
- `competitors` (Object[]): Top 5 sellers with `sellerId` and `percent`.
- `avgBuyBoxCompetitors` (Float).
- `buyBoxNewOwnershipRate`, `buyBoxUsedOwnershipRate` (Integer): 0-100.

**Storefront (requires `storefront` param):**
- `asinList` (String[]): Up to 100K ASINs, freshest first.
- `asinListLastSeen` (Integer[]): Matching Keepa timestamps.
- `totalStorefrontAsins` (Integer[]): `[lastUpdate keepaTime, count]`. Auto-populated.

**History:**
- `csv` (Integer[][]): Index 0=RATING (0-100%), Index 1=RATING_COUNT. Format: `[keepaTime, value, ...]`.

**Statistics:**
- `sellerCategoryStatistics` (Object[]): `{catId, productCount, avg30SalesRank, productCountWithAmazonOffer}`.
- `sellerBrandStatistics` (Object[]): `{brand, productCount, avg30SalesRank, productCountWithAmazonOffer}`.

---

## Deal Object

**Returned by:** Browsing Deals request.

**Product info:** `asin`, `title`, `rootCat` (Long), `categories` (Long[]).
- `image` (Integer[]): ASCII char codes. Convert: `String.fromCharCode.apply("", dealObject.image)`. Full URL: `https://images-na.ssl-images-amazon.com/images/I/<imageName>`.

**Price/rank data** (indexed by CSV Price Type and Date Range 0=day, 1=week, 2=month, 3=90days):
- `current` (Integer[]): Current prices/ranks. -1=out of stock.
- `currentSince` (Integer[]): When current value started (Keepa time).
- `deltaLast` (Integer[]): Diff between previous and current.
- `delta` (Integer[][]): Diff between average and current, `[DateRange][PriceType]`.
- `deltaPercent` (Integer[][]): Same as delta in percent.
- `avg` (Integer[][]): Weighted averages. Day index (0) uses 48-hour average.

**Deal info:**
- `lastUpdate`, `creationDate` (Integer): Keepa time.
- `parentAsin` (String): null if not a variation.
- `lightningEnd` (Integer): 0 if not lightning deal.
- `warehouseCondition` (Integer): 0=none, 2=Like New, 3=Very Good, 4=Good, 5=Acceptable.
- `warehouseConditionComment` (String).

---

## Lightning Deal Object

**Returned by:** `/lightningdeal` endpoint.

- `domainId`, `asin`, `title` (Strings).
- `sellerId`, `sellerName` (Strings).
- `dealId` (String): Shared among variation children.
- `dealPrice` (Integer): Cents (-1 if upcoming). `currentPrice` (Integer).
- `image` (String): Full URL: `https://images-na.ssl-images-amazon.com/images/I/<image>`.
- `isPrimeEligible`, `isFulfilledByAmazon` (Booleans).
- `rating` (Integer, 0-50), `totalReviews` (Integer).
- `dealState` (String): AVAILABLE, WAITLIST, SOLDOUT, WAITLISTFULL, EXPIRED, SUPPRESSED.
- `startTime`, `endTime` (Integer): Keepa time.
- `percentClaimed` (Integer), `percentOff` (Integer).
- `variation` (Object[]): `[{dimension, value}, ...]`.
- `lastUpdate` (Integer).

---

## Best Sellers Object

**Returned by:** `/bestsellers` endpoint.

- `domainId` (Integer), `lastUpdate` (Integer), `categoryId` (Long).
- `asinList` (String[]): Ordered best-selling first.

---

## Notification Object

**Returned by:** Get Notifications or webhook.

- `asin`, `title`, `image` (Strings).
- `ttl` (Integer): Minutes. `createDate` (Integer): Keepa time.
- `domainId` (Integer), `notificationDomainId` (Integer).
- `csvType` (Integer, 0-35), `trackingNotificationCause` (Integer): 0=EXPIRED, 1=DESIRED_PRICE, 3=PRICE_CHANGE_AFTER_DESIRED_PRICE, 4=OUT_STOCK, 5=IN_STOCK, 6=DESIRED_PRICE_AGAIN.
- `currentPrices` (Integer[]): By price type. -1=unavailable.
- `sentNotificationVia` (Boolean[]): Index 5=API.
- `metaData`, `trackingListName` (Strings).

---

## Tracking Creation Object

**Used in:** Add Tracking request body.

- `asin` (String), `ttl` (Integer): Hours (0=never expires).
- `expireNotify` (Boolean), `desiredPricesInMainCurrency` (Boolean).
- `mainDomainId` (Integer), `updateInterval` (Integer, 0-25): Hours.
- `metaData` (String, max 500 chars).
- `thresholdValues` (TrackingThresholdValue[]): Each: `{thresholdValue, domain, csvType, isDrop}`.
- `notifyIf` (TrackingNotifyIf[]): Each: `{domain, csvType, notifyIfType}` (0=OUT_OF_STOCK, 1=BACK_IN_STOCK).
- `notificationType` (Boolean[7]): Index 5=API. Example: `[false,false,false,false,false,true,false]`.
- `individualNotificationInterval` (Integer): -1=default, 0=once only, N>0=rearm after N minutes.

---

## Tracking Object

**Returned by:** Get Tracking request.

- `asin`, `createDate` (Keepa time), `isActive` (Boolean).
- `ttl` (Integer), `expireNotify` (Boolean).
- `mainDomainId`, `updateInterval` (Integers).
- `metaData`, `trackingListName` (Strings).
- `thresholdValues` (TrackingThresholdValue[]): Each has `thresholdValueCSV` history.
- `notifyIf`, `notificationType`, `notificationCSV`, `individualNotificationInterval`.

---

## Search Insights Object

**Found in:** Product Finder response when `stats=1`.

**Price/trend:**
- `avgDeltaPercent30BuyBox`, `avgDeltaPercent90BuyBox` (Float): Positive=cheaper.
- `avgDeltaPercent30Amazon`, `avgDeltaPercent90Amazon` (Float).
- `avgBuyBox`, `avgBuyBox90`, `avgBuyBox365` (Integer): Cents.
- `avgBuyBoxDeviation` (Integer): 30-day volatility.

**Review/rating:**
- `avgReviewCount` (Integer), `avgRating` (Integer, 10-50 scale).

**Market composition:**
- `isFBAPercent`, `soldByAmazonPercent`, `hasCouponPercent` (Float): 0.0-100.0.
- `avgOfferCountNew`, `avgOfferCountUsed` (Float).
- `sellerCount`, `brandCount` (Integer).

**Rank range:**
- `highestRank` (worst), `lowestRank` (best) (Integer).

**Discovery:**
- `relatedCategories` (Long[]): By frequency.
- `topBrandsWithCounts` (Map): Up to 5 brands + counts.
- `topSellersWithCounts` (Map): Up to 5 sellers + counts.

---

## Category Object

**Returned by:** `/category` and `/search?type=category`.

- `domainId`, `catId` (Long), `name` (String).
- `websiteDisplayGroup` (String), `parent` (Long, 0=root), `children` (Long[]).
- `isBrowseNode` (Boolean): true for standard, false for promotional.
- `productCount`, `highestRank`, `lowestRank` (Integer).
- `avgBuyBox`, `avgBuyBox90`, `avgBuyBox365`, `avgBuyBoxDeviation` (Integer).
- `avgReviewCount`, `avgRating` (Integer).
- `isFBAPercent`, `soldByAmazonPercent`, `hasCouponPercent` (Float).
- `avgOfferCountNew`, `avgOfferCountUsed` (Float).
- `sellerCount`, `brandCount` (Integer).
- `avgDeltaPercent30BuyBox`, `avgDeltaPercent90BuyBox`, `avgDeltaPercent30Amazon`, `avgDeltaPercent90Amazon` (Float).
- `relatedCategories` (Long[]), `topBrands` (String[]).
