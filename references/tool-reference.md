# Vio MCP Tool Reference

Complete parameter schemas and response structures for the Vio hotel search MCP tools.

## Tool: `search_hotels`

Search for hotels by location, coordinates, or hotel name. Returns hotel results with configurable data blocks.

### Input Parameters

#### Search Parameters (mutually exclusive — provide exactly one)

| Parameter | Type | Description |
|-----------|------|-------------|
| `queries` | `string[]` (1-3) | Location or hotel names. Always provide full context (e.g., "Lisbon, Portugal" not "Lisbon"). For similar hotel searches, use the hotel name directly. |
| `latitude` | `number` (-90 to 90) | Latitude for coordinate search. Must pair with `longitude`. |
| `longitude` | `number` (-180 to 180) | Longitude for coordinate search. Must pair with `latitude`. |
| `nearMe` | `boolean` | Search near user's GPS location. |
| `address` | `string` | Search near a specific address or landmark (NOT for hotel names — use `queries`). |

#### Date & Stay Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `checkIn` | `string` | 15 days from today | Check-in date (YYYY-MM-DD). When month/day given without year, use next occurrence. |
| `checkOut` | `string` | 3 nights after check-in | Check-out date (YYYY-MM-DD). Must be after checkIn. |
| `dayDistance` | `integer` | — | Days from today to check-in (e.g., "next week" = 7). Overrides checkIn. |
| `nights` | `integer` | 3 | Stay duration in nights. Used with checkIn or dayDistance to calculate checkOut. |

#### Room Configuration

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `roomsConfiguration` | `object[]` | `[{adults: 2}]` | Room occupancy. Each room: `{adults: number, children?: number[]}`. Children ages 0-17. Max 9 rooms. |

Example configurations:
```json
// 2 adults (default)
[{"adults": 2}]

// Family: 2 adults + 2 children
[{"adults": 2, "children": [5, 8]}]

// 2 rooms: 3 adults + 2 adults with child
[{"adults": 3}, {"adults": 2, "children": [12]}]
```

#### Data Block Selection

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `include` | `string[]` | `['location', 'rating', 'classification', 'media', 'offers']` | Data categories to return. |

Available include blocks:

| Block | Returns |
|-------|---------|
| `location` | Address, coordinates, timezone, area description, nearby attractions |
| `rating` | Overall guest rating (0-10), review count, guest type breakdown, detailed aspect ratings |
| `classification` | Star rating (0-5), property type, themes, sentiments |
| `facility` | Facilities list, amenities/business/dining descriptions |
| `media` | Hotel image URLs and total image count |
| `offer` | Room offers with pricing, cancellation, provider info |
| `room` | Room types with descriptions, amenities, bed types, images, and per-room offers |
| `faq` | Frequently asked questions |
| `review` | Guest reviews with ratings, text, dates, and sources |
| `insight` | AI-generated review summaries (overall + categories: Facilities, Cleanliness, Rooms, Service, Location, Food) |
| `policy` | Check-in/out times, fees, deposits, policies, know-before-you-go |
| `analytic` | Price trends, historical data, comparison vs similar hotels |

Both singular and plural forms accepted (e.g., `offers` or `offer`).

#### Sorting & Pagination

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `sortField` | `enum` | `popularity` | Options: `popularity`, `price`, `guestRating`, `starRating`, `distance` |
| `sortOrder` | `enum` | — | `ascending` or `descending` |
| `pageSize` | `integer` | 10 | Hotels per page (1-100) |
| `offsets` | `integer[]` | `[0, ...]` | Pagination offsets, one per query. Use `nextOffsets` from previous response. |

#### Filters

All filters are optional. Pass as the `filters` object.

| Filter | Type | Logic | Description |
|--------|------|-------|-------------|
| `minPrice` | `number` | — | Minimum price (inclusive) |
| `maxPrice` | `number` | — | Maximum price (inclusive) |
| `starRatings` | `number[]` | OR | Star ratings 0-5 (e.g., `[4, 5]`) |
| `guestRating` | `number[]` | OR | Guest scores 0-10 (e.g., `[8, 9, 10]`) |
| `propertyTypes` | `string[]` | OR | Property types in English (e.g., `["hotel", "apartment"]`). Semantic matching. |
| `excludePropertyTypes` | `string[]` | OR | Exclude types (e.g., `["hostel"]`). Mutually exclusive with propertyTypes. |
| `facilities` | `string[]` | AND | Facilities in English (e.g., `["pool", "gym"]`). Hotel must have ALL. Semantic matching. |
| `themes` | `string[]` | OR | Themes in English (e.g., `["beach", "romantic"]`). Semantic matching. |

**Filter combination**: Different filter types combine with AND logic. Within a filter type, values combine with OR logic (except facilities which use AND).

**Semantic matching**: Property types, facilities, and themes use vector-based semantic matching. Natural language works: "pets allowed" matches "Pet Friendly", "workout room" matches "Gym".

#### Offer Options

Pass as the `offers` object:

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `offers.mode` | `enum` | `top_offers` | `cheapest` (single), `top_offers` (limited set), `all` (every offer) |
| `offers.sort` | `enum` | `price:total_asc` | `price:base_asc`, `price:total_asc`, `price:with_tax_asc` |
| `offers.filters.freeCancellation` | `enum` | `any` | `any`, `included`, `excluded` |
| `offers.filters.mealIncluded` | `enum` | `any` | `any`, `included`, `excluded` |
| `offers.filters.mealTypes` | `string[]` | — | `breakfast`, `lunch`, `dinner`, `allInclusive`, `fullBoard`, `halfBoard` (OR logic) |
| `offers.filters.payLater` | `enum` | `any` | `any`, `included`, `excluded` |

#### Review Options

Pass as the `reviews` object (only applies when include contains `review`):

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `reviews.max` | `integer` | 5 | Max reviews per hotel (1-50) |
| `reviews.sortBy` | `enum` | `relevancy` | `relevancy`, `recency`, `rating_asc`, `rating_desc` |
| `reviews.ratings` | `integer[]` | — | Filter by score 1-10 (OR logic) |
| `reviews.searchTerms` | `string[]` | — | Text search in reviews (OR logic) |
| `reviews.fromDate` | `string` | — | Date range start (YYYY-MM-DD) |
| `reviews.toDate` | `string` | — | Date range end (YYYY-MM-DD) |

#### Other Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `prompt` | `string` | — | User's query in English (strip PII). |
| `currency` | `string` | auto | ISO 4217 code. Only set when user explicitly requests a currency. |
| `priceMode` | `enum` | `total` | `total` (entire stay) or `nightly` (per night). |
| `searchMode` | `enum` | `fast` | `fast` (quick, may be partial) or `deep` (exhaustive, slower). |
| `optimizeRooms` | `boolean` | — | Find best offer across different room configurations for same total occupancy. |

### Output Schema

```json
{
  "error": "string (optional, recoverable error message)",
  "include": ["string (data blocks included)"],
  "language": "string (ISO 639-1)",
  "currency": "string (ISO 4217)",
  "checkIn": "string (YYYY-MM-DD)",
  "checkOut": "string (YYYY-MM-DD)",
  "nights": "number",
  "roomConfiguration": [{"adults": 2, "children": [5]}],
  "priceScope": "per_room | all_rooms_combined",
  "priceLogic": "base_tax_fees | base_fees | base",
  "hotels": [
    {
      "id": "string",
      "name": "string",
      "url": "string (booking page URL)",
      "propertyDescription": "string",
      "isPartialMatch": "boolean (true if only some filters matched)",
      "isAnchorHotel": "boolean (true for the searched hotel in similar search)",
      "typicalPriceRange": {"min": 0, "max": 0},
      "location": {
        "address": "string",
        "displayName": "string",
        "latitude": 0, "longitude": 0,
        "timezone": "string",
        "areaDescription": "string",
        "attractionsDescription": "string",
        "nearbyAttractions": [{"name": "string", "distanceKm": 0, "distanceMiles": 0}]
      },
      "rating": {
        "overall": 8.5,
        "reviewCount": 1200,
        "guestType": {"business": 0, "couples": 0, "families": 0, "groups": 0, "solo": 0},
        "detailed": {"cleanliness": 0, "dining": 0, "facilities": 0, "location": 0, "pricing": 0, "rooms": 0, "service": 0}
      },
      "classification": {
        "starRating": 4,
        "propertyType": {"id": "string", "name": "Hotel"},
        "themes": [{"id": "string", "name": "Business"}],
        "sentiments": [{"id": "string", "name": "Modern"}]
      },
      "facilities": {
        "items": [{"id": "string", "name": "Free WiFi"}],
        "amenities": "string", "business": "string", "dining": "string"
      },
      "media": {"images": ["url"], "imagesAvailable": 50},
      "offers": {
        "items": [
          {
            "id": "string",
            "url": "string (booking URL)",
            "currency": "EUR",
            "rate": {
              "base": 150, "taxes": 20, "hotelFees": 5,
              "displayPrice": 175
            },
            "cancellationPenalties": [{"amount": 0, "currency": "EUR", "start": "date", "end": "date"}],
            "package": {"amenities": ["Breakfast included"]},
            "providerCode": "booking_com",
            "providerName": "Booking.com",
            "roomName": "Deluxe Double Room",
            "tags": ["top_offer", "cheapest_offer"],
            "occupancy": {"adults": 2, "children": []}
          }
        ],
        "availableCount": 15,
        "cheapestRate": {"base": 150, "taxes": 20, "hotelFees": 5, "displayPrice": 175}
      },
      "rooms": [
        {
          "id": "string", "name": "Deluxe Double",
          "description": "string", "maxOccupancy": 3,
          "amenities": ["WiFi", "TV"], "bedTypes": [{"name": "King"}],
          "images": ["url"], "area": {"squareMeters": 30},
          "offers": [{"...offer object"}]
        }
      ],
      "faq": [{"question": "string", "answer": "string", "label": "Category"}],
      "reviews": {
        "totalAvailable": 500, "averageRating": 8.5,
        "perRatingBreakdown": {"10": 100, "9": 80},
        "items": [{"date": "string", "name": "string", "score": 9, "text": "string", "source": "string"}]
      },
      "insights": {
        "overall": "AI-generated summary of all reviews",
        "categories": [{"category": "Facilities", "title": "Short title", "summary": "Detailed summary"}]
      },
      "policies": {
        "checkIn": {"beginTime": "15:00", "endTime": "23:00", "minAge": 18},
        "checkOut": {"time": "11:00"},
        "fees": {"mandatory": "string", "optional": "string"},
        "knowBeforeYouGo": "string"
      },
      "analytics": {
        "averagePrice": 160, "timeRangeDays": 30,
        "trend": "up | down | stable",
        "changePercentage": 5.2,
        "comparisonToSimilar": {
          "differencePercentage": -17,
          "assessment": "cheaper | same | expensive"
        },
        "history": [{"timestamp": "ISO date", "price": 150}],
        "similarHotelsHistory": [{"timestamp": "ISO date", "price": 180}]
      }
    }
  ],
  "totalResults": 150,
  "hasMoreResults": true,
  "nextOffsets": [10],
  "anchorHotelIds": ["hotel-123"],
  "placeId": "string",
  "placeDisplayName": "Amsterdam, Netherlands",
  "appliedFilters": {
    "propertyTypes": ["Hotel"],
    "facilities": ["Swimming Pool", "Gym"],
    "themes": ["Business"]
  },
  "queries": ["Amsterdam"]
}
```

---

## Tool: `get_hotels`

Fetch detailed data for specific hotels by ID. Use after `search_hotels` to get additional information.

### Input Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `hotelIds` | `string[]` | Yes (min 1) | Hotel IDs from previous search results |
| `prompt` | `string` | No | User query in English (strip PII) |
| `currency` | `string` | No | ISO 4217 code (only when user explicitly requests) |
| `checkIn` | `string` | No | Check-in date (YYYY-MM-DD) |
| `checkOut` | `string` | No | Check-out date (YYYY-MM-DD) |
| `dayDistance` | `integer` | No | Days from today to check-in |
| `nights` | `integer` | No | Stay duration (default 3) |
| `roomsConfiguration` | `object[]` | No | Room occupancy (default `[{adults: 2}]`) |
| `include` | `string[]` | No | Data blocks (same values as search_hotels) |
| `reviews` | `object` | No | Review options (same as search_hotels) |
| `rooms` | `object` | No | Room options: `{maxImages: number}` |
| `media` | `object` | No | Media options: `{maxImages: number}` |
| `offers` | `object` | No | Offer options (same as search_hotels) |
| `searchMode` | `enum` | No | `fast` (default) or `deep` |

### Output Schema

Same structure as `search_hotels` output, but without pagination fields (`totalResults`, `hasMoreResults`, `nextOffsets`, `queries`). Returns data only for the requested hotel IDs.

---

## Natural Language to Parameter Mapping

Common user phrases and their parameter equivalents:

| User says | Parameter |
|-----------|-----------|
| "pool", "swimming" | `filters.facilities: ["pool"]` |
| "pet friendly", "dogs allowed" | `filters.facilities: ["pet friendly"]` |
| "gym", "fitness", "workout room" | `filters.facilities: ["gym"]` |
| "spa", "wellness" | `filters.facilities: ["spa"]` |
| "free wifi", "internet" | `filters.facilities: ["wifi"]` |
| "parking" | `filters.facilities: ["parking"]` |
| "4-star", "four star" | `filters.starRatings: [4]` |
| "luxury" | `filters.themes: ["luxury"]` |
| "family friendly" | `filters.themes: ["family friendly"]` |
| "beach" | `filters.themes: ["beach"]` |
| "apartment", "self-catering" | `filters.propertyTypes: ["apartment"]` |
| "villa" | `filters.propertyTypes: ["villa"]` |
| "no hostels" | `filters.excludePropertyTypes: ["hostel"]` |
| "under 200", "budget 200" | `filters.maxPrice: 200` |
| "free cancellation" | `offers.filters.freeCancellation: "included"` |
| "breakfast included" | `offers.filters.mealIncluded: "included"` |
| "pay at hotel", "pay later" | `offers.filters.payLater: "included"` |
| "cheapest first", "sort by price" | `sortField: "price", sortOrder: "ascending"` |
| "highest rated" | `sortField: "guestRating", sortOrder: "descending"` |
| "next week" | `dayDistance: 7` |
| "tomorrow" | `dayDistance: 1` |
| "3 nights" | `nights: 3` |
| "is it a good deal?" | `include: ["offer", "analytic"]` |

## Example Tool Calls

### Basic hotel search
```json
{
  "tool": "search_hotels",
  "arguments": {
    "queries": ["Amsterdam, Netherlands"],
    "nights": 3,
    "roomsConfiguration": [{"adults": 2}]
  }
}
```

### Filtered search with sorting
```json
{
  "tool": "search_hotels",
  "arguments": {
    "queries": ["Barcelona, Spain"],
    "checkIn": "2026-07-01",
    "nights": 5,
    "roomsConfiguration": [{"adults": 2, "children": [8]}],
    "filters": {
      "starRatings": [4, 5],
      "facilities": ["pool", "gym"],
      "maxPrice": 250
    },
    "offers": {
      "filters": {"freeCancellation": "included"}
    },
    "sortField": "price",
    "sortOrder": "ascending"
  }
}
```

### Get hotel details and reviews
```json
{
  "tool": "get_hotels",
  "arguments": {
    "hotelIds": ["hotel_abc123"],
    "include": ["location", "rating", "facility", "review", "insight", "policy"],
    "reviews": {"max": 10, "sortBy": "recency"}
  }
}
```

### Get rooms and pricing for specific hotel
```json
{
  "tool": "get_hotels",
  "arguments": {
    "hotelIds": ["hotel_abc123"],
    "checkIn": "2026-07-01",
    "nights": 5,
    "roomsConfiguration": [{"adults": 2, "children": [8]}],
    "include": ["offer", "room"],
    "offers": {"mode": "all"}
  }
}
```

### Pagination (next page)
```json
{
  "tool": "search_hotels",
  "arguments": {
    "queries": ["Amsterdam, Netherlands"],
    "offsets": [10],
    "nights": 3
  }
}
```

### Price analysis
```json
{
  "tool": "get_hotels",
  "arguments": {
    "hotelIds": ["hotel_abc123"],
    "include": ["offer", "analytic"]
  }
}
```
