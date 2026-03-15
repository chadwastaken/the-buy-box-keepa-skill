# Keepa API — Practical Examples

## 1) Retrieve a product by ASIN
```
GET /product?key=<key>&domain=1&asin=B000123&stats=180&history=1&offers=20
```
- Use `stats=180` for current price summary + 180-day computed metrics.
- Use `history=0` to exclude history for faster response when you only need current snapshot.
- Use `offers=N` when you need offer-level data (costs more tokens).

## 2) Find products matching constraints (Product Finder)
```
POST /query?key=<key>&domain=1
{
  "page": 0,
  "perPage": 100,
  "current_SALES_lte": 5000,
  "current_NEW_gte": 1500,
  "brand": ["Canon"],
  "sort": [["current_SALES", "asc"]]
}
```
- Use paging to a maximum of 10,000 ASINs (combination of page and perPage).
- Results may drift slightly over time because Keepa's database updates continuously.

## 3) Lightning Deals
```
GET /lightningdeal?key=<key>&domain=1&asin=B000123          # Specific ASIN (1 token)
GET /lightningdeal?key=<key>&domain=1&state=AVAILABLE        # All available deals (500 tokens)
```
- Use the Lightning Deals endpoint for active/recent lightning deals.
- For broader price change discovery, use Product Finder with delta/deltaPercent filters.

## 4) Category exploration
```
GET /category?key=<key>&category=0                           # Root categories
GET /search?key=<key>&type=category&term=electronics         # Search by name
GET /bestsellers?key=<key>&domain=1&category=562066          # Best sellers in category
```

## 5) Tracking / alerts (price watches)

**Add a tracking:**
```
POST /tracking?key=<key>&type=add
{
  "asin": "B000123",
  "ttl": 0,
  "expireNotify": true,
  "mainDomainId": 1,
  "thresholdValues": [
    {
      "thresholdValue": 2999,
      "domain": 1,
      "csvType": 0,
      "isDrop": true
    }
  ],
  "notificationType": [false, false, false, false, false, true, false],
  "individualNotificationInterval": -1
}
```

**Get tracking status:**
```
GET /tracking?key=<key>&type=get&asin=B000123
```

**Remove tracking:**
```
GET /tracking?key=<key>&type=remove&asin=B000123
```

**Get notifications:**
```
GET /tracking?key=<key>&type=notification&since=<keepaTime>&revise=0
```

## 6) Browsing Deals (price drops & sales rank changes)
```
POST /deal?key=<key>
{
  "page": 0,
  "domainId": 1,
  "excludeCategories": [],
  "includeCategories": [281052],
  "priceTypes": [0],
  "deltaRange": [20, 90],
  "deltaPercentRange": [10, 80],
  "currentRange": [100, 50000],
  "isRangeEnabled": true,
  "isFilterEnabled": true,
  "sortType": 3,
  "dateRange": 0
}
```
- `priceTypes`: which CSV Type index to filter on (0=Amazon, 1=New, 18=Buy Box, etc.).
- `deltaRange`/`deltaPercentRange`: absolute/percentage price drop thresholds.
- `sortType`: 1=Deal age, 2=Deal score, 3=Title, 4=Sales Rank, 5=Price, 6=Price drop (abs), 7=Price drop (%), 8=Highest Price. Negative to invert.
- `dateRange`: 0=day, 1=week, 2=month, 3=90 days.

## 7) Seller lookup with storefront
```
GET /seller?key=<key>&domain=1&seller=A2R2RITDJNW1Q6&storefront=1
```
- Returns seller details + up to 100K storefront ASINs.
- Token cost: 1 (base) + 9 (storefront) = 10 tokens.

## 8) Graph Image (price history chart)
```
GET api.keepa.com/graphimage?key=<key>&domain=1&asin=B08N5WRWNW&bb=1&amazon=1&new=1&range=365&width=800&height=400
```
- Returns binary PNG, NOT JSON.
- **NEVER** expose this URL publicly — your API key is in it. Proxy it server-side.
- Use `/token` endpoint after to check remaining balance.

## 9) Product search by keyword
```
GET /search?key=<key>&domain=1&type=product&term=wireless%20mouse&stats=180
```
- Returns up to 10 results per page, 4 pages max (40 total).
- **April 20, 2026:** `page` param removed, results increase to 20 per request.

## 10) Check token balance
```
GET /token?key=<key>
```
- Free (0 tokens). Returns `refillRate`, `tokensLeft`, `refillIn`, etc.

## Common Patterns

### Get current Buy Box price for an ASIN
```
GET /product?key=<key>&domain=1&asin=B000123&stats=1&history=0
```
Then read: `product.stats.buyBoxPrice` (cents) and `product.stats.buyBoxSellerId`.

### Find products where Amazon is out of stock but 3rd party sellers exist
```
POST /query?key=<key>&domain=1
{
  "availabilityAmazon": [-1],
  "current_NEW_gte": 1,
  "current_SALES_lte": 50000,
  "perPage": 100
}
```

### Monitor a category for price drops
```
POST /deal?key=<key>
{
  "domainId": 1,
  "priceTypes": [18],
  "includeCategories": [<categoryId>],
  "deltaPercentRange": [-50, -10],
  "isRangeEnabled": true,
  "dateRange": 0
}
```

### Get product image URL (NOT the graph chart)
```
GET /product?key=<key>&domain=1&asin=B000123&history=0
```
Then use: `https://m.media-amazon.com/images/I/<product.images[0].l>` for the product photo.

### Convert Keepa time to JavaScript Date
```javascript
function keepaTimeToDate(keepaTime) {
  return new Date((keepaTime + 21564000) * 60000);
}

function dateToKeepaTime(date) {
  return Math.floor(date.getTime() / 60000) - 21564000;
}
```

### Convert Keepa time in Python
```python
from datetime import datetime, timezone

def keepa_time_to_datetime(keepa_time):
    unix_seconds = (keepa_time + 21564000) * 60
    return datetime.fromtimestamp(unix_seconds, tz=timezone.utc)

def datetime_to_keepa_time(dt):
    return int(dt.timestamp() / 60) - 21564000
```
