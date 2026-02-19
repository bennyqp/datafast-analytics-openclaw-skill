# Source: https://datafa.st/docs (DataFast API documentation)
# Copied on: 2026-02-19
# Note: This is a snapshot for reference; check the official docs for updates.

# Get started with the DataFast API

Use the DataFast API to programmatically interact with your analytics data, such as creating custom goals or retrieving visitor details.

## 1. Generate an API Key

Navigate to your **Website's Settings > API** tab in the [DataFast dashboard](/dashboard). Generate a new API key. Copy the key immediately as it won't be shown again. Keep your API keys secure and treat them like passwords. Do not expose them in frontend code or public repositories.

![Website settings screenshot](/website-settings-guide.jpg)

![API key screenshot](/blog-api-key-step-2.jpg)

## 2. Authenticate requests

All API requests must be authenticated using the `Authorization` header with the Bearer scheme. Replace `YOUR_API_KEY` with the key you generated.

`Authorization: Bearer YOUR_API_KEY`

## 3. Make API calls

The base URL for all v1 API endpoints is:
`https://datafa.st/api/v1/`

Here are some API use cases:
- [Create a custom goal](/docs/api-create-goal)
- [Get visitor data](/docs/api-get-visitor)

> Try the [API Playground](/docs/api-playground) to test the API endpoints interactively.

---

**Successful requests** return a `200 OK` status and a body like:

```json
{
  "status": "success",
  "data": { ... } // Endpoint-specific data
}
```

**Failed requests** return an appropriate HTTP status code (4xx or 5xx) and a body like:

```json
{
  "status": "error",
  "error": {
    "code": HTTP_STATUS_CODE,
    "message": "A descriptive error message"
  }
}
```

Common error codes include:
- `400 Bad Request`: Invalid input or missing parameters.
- `401 Unauthorized`: Missing or invalid API key.
- `404 Not Found`: Resource not found (e.g., specific visitor).
- `500 Internal Server Error`: Server-side issue.


---

### Playground

Test the analytics API endpoints interactively.

# API Playground

Use this playground to test the DataFast Analytics API endpoints. Choose an endpoint, set your parameters and filters, and see the results instantly.

<api-playground />

---

### Visitor

API endpoint to retrieve detailed information about a specific visitor.

# Get visitor data

`GET https://datafa.st/api/v1/visitors/{datafast_visitor_id}`

Retrieve identity, activity, and prediction data for a specific `datafast_visitor_id`. Requires [Bearer Token](/docs/api-introduction) authentication.

## Response

- **Success (200 OK):** Returns a detailed visitor data object.
- **Errors:** See [API Introduction](/docs/api-introduction). A 400 occurs if the visitor is a bot.

## Response fields

See the example response for the structure. Key objects:

- **`identity`**: Geo-location, device, browser, OS, and first-session parameters derived from the latest pageview.
- **`activity`**: Visit/pageview counts, timing, last URL, completed custom goals, and a list of visited pages.
- **`prediction`**: Conversion prediction details (rate, USD value, score, confidence). Null if visitor is already a customer or you haven't set up [revenue attribution](/docs/revenue-attribution-guide).

> Use `prediction` to get more sales: Show a lead magnet or coupon to low-converting visitors, or create urgency or exclusivity to high-potential visitors.


#### Code examples

##### Example request (Node.js/Express)

```javascript
// Your backend API endpoint
const handler = async (req, res) => {
  const datafast_visitor_id = req.cookies.datafast_visitor_id;

  try {
    const response = await fetch(`https://datafa.st/api/v1/visitors/${datafast_visitor_id}`, {
      method: "GET",
      headers: {
        Authorization: `Bearer ${DATAFAST_API_KEY}`,
        "Content-Type": "application/json",
      },
    });

    const result = await response.json();

    res.status(200).send(result);
  } catch (error) {
    console.error("Error sending goal to DataFast:", error);
    res.status(500).send("Failed to track goal");
  }
};
```

##### Success response (200 OK)

```json
{
    "status": "success",
    "data": {
        "visitorId": "a3ab2331-989f-4cfa-91c6-2461c9e3c6bd",
        "identity": {
            "country": "KR",
            "countryCode": "KR",
            "region": "KR-44",
            "city": "Seosan City",
            "params": {  "ref": "youtube.com", "source": null, "utm_source": null, "utm_medium": null, "utm_campaign": null, "utm_content": null, "utm_term": null },
            "browser": { "name": "Chrome", "version": "133.0.0.0" },
            "os": { "name": "Mac OS", "version": "10.15.7" },
            "device": { "type": "desktop", "vendor": "Apple", "model": "Macintosh" },
            "viewport": { "width": 1728, "height": 998 }
        },
        "activity": {
            "visitCount": 3,
            "pageViewCount": 8,
            "timeSinceFirstVisit": 3296041, // in milliseconds
            "timeSinceCurrentVisit": 3296041, // in milliseconds
            "firstVisitAt": "2025-04-11T03:38:49.154Z",
            "lastVisitAt": "2025-04-11T03:38:49.154Z",
            "currentUrl": "codefa.st/",
            "visitedPages": [
                {
                    "url": "codefa.st/",
                    "timestamp": "2025-04-11T03:38:49.154Z"
                },
            ]
            "completedCustomGoals": [
                {
                  "name": "newsletter_signup",
                  "description": "Sign up for the newsletter",
                  "isServer": false, // true if the goal was created using DataFast API
                  "timestamp": "2025-04-11T03:38:54.253Z"
                },
            ],
           
        },
        "prediction": {
            "score": 72, // (0-100) how likely is this visitor to convert? 50 is your baseline/average, 100 is very likely to convert, 0 is very unlikely to convert
            "conversionRate": 0.0138, // predicted conversion rate (1.38% chance to convert here)
            "expectedValue": 3.089, // predicted revenue in USD (predicted conversion rate * predicted order value)
            "confidence": 0.831, // (0-1) our confidence level based on data points analyzed. The higher the number, the more accurate the predictions. don't make important decisions if confidence is <0.5
        }
    }
}
```

---

### Time Series

Retrieve time series analytics data for visitors, sessions, revenue, or conversion rate.

# Get time series data

`GET https://datafa.st/api/v1/analytics/timeseries`

Retrieve time series analytics data aggregated by hour, day, week, or month. Requires [Bearer Token](/docs/api-introduction) authentication.

## Response

### Single Field Mode (using `fields` parameter with one field)

**Success (200 OK):** Returns time series data with a single value per timestamp

```json
{
  "status": "success",
  "fields": ["visitors"],
  "interval": "day",
  "timezone": "UTC",
  "currency": null,
  "totals": {
    "visitors": 1234
  },
  "data": [
    {
      "visitors": 45,
      "name": "01 Jan",
      "timestamp": "2024-01-01T00:00:00Z"
    },
    {
      "visitors": 52,
      "name": "02 Jan",
      "timestamp": "2024-01-02T00:00:00Z"
    }
  ],
  "pagination": {
    "limit": 100,
    "offset": 0,
    "total": 30
  }
}
```

### Multi-Field Mode (using `fields` parameter)

**Success (200 OK):** Returns time series data with multiple fields per timestamp

```json
{
  "status": "success",
  "fields": [
    "visitors",
    "name",
    "sessions",
    "revenue",
    "conversion_rate"
  ],
  "interval": "day",
  "timezone": "Asia/Calcutta",
  "currency": "$",
  "totals": {
    "visitors": 14213,
    "sessions": 20181,
    "revenue": 27351,
    "revenueBreakdown": {
      "new": 22150.00,
      "renewal": 5201.00,
      "refund": 0.00
    },
    "conversion_rate": 1.92
  },
  "data": [
    {
      "visitors": 528,
      "name": "17 Dec",
      "sessions": 604,
      "revenue": 0,
      "revenueBreakdown": {
        "new": 0.00,
        "renewal": 0.00,
        "refund": 0.00
      },
      "conversion_rate": 0,
      "timestamp": "2025-12-17T00:00:00+05:30"
    },
    {
      "visitors": 472,
      "name": "18 Dec",
      "sessions": 534,
      "revenue": 468,
      "revenueBreakdown": {
        "new": 380.00,
        "renewal": 88.00,
        "refund": 0.00
      },
      "conversion_rate": 0.99,
      "timestamp": "2025-12-18T00:00:00+05:30"
    },
    {
      "visitors": 428,
      "name": "19 Dec",
      "sessions": 490,
      "revenue": 299,
      "revenueBreakdown": {
        "new": 249.00,
        "renewal": 50.00,
        "refund": 0.00
      },
      "conversion_rate": 0.70,
      "timestamp": "2025-12-19T00:00:00+05:30"
    }
  ],
  "pagination": {
    "limit": 5,
    "offset": 0,
    "total": 30
  }
}
```

## Notes

- The API automatically upgrades to hourly granularity for same-day queries when using coarser intervals (day, week, month)
- **Revenue field:** Always includes both new and renewal revenue (total revenue)
  - When `revenue` field is requested, a `revenueBreakdown` object is automatically included with:
    - `new`: New revenue from first-time purchases
    - `renewal`: Revenue from subscription renewals
    - `refund`: Refunded revenue amount
  - All breakdown values are numbers rounded to 2 decimal places
- **Conversion rate:** Revenue-based conversion rate (revenue per visitor)
  - If your website has `includeRenewalRevenue` enabled: `(new revenue + renewal revenue) / unique visitors`
  - If `includeRenewalRevenue` is disabled: `new revenue / unique visitors`
  - This provides a monetary conversion metric rather than a transaction count metric
- Timestamps are returned in ISO 8601 format
- If both `startAt` and `endAt` are provided as full-day UTC timestamps (midnight to 23:59), they are automatically converted to simple dates for timezone-aware interpretation

## Errors

- **400 Bad Request:** Invalid parameters or missing required parameters
- **401 Unauthorized:** Missing or invalid API key
- **404 Not Found:** Website not found
- **500 Internal Server Error:** Server-side issue

#### Code examples

##### Example request (Node.js/Express)

```javascript
// Get daily visitors for the last 30 days
const response = await fetch(
  "https://datafa.st/api/v1/analytics/timeseries?fields=visitors&interval=day&limit=30",
  {
    method: "GET",
    headers: {
      Authorization: `Bearer ${DATAFAST_API_KEY}`,
      "Content-Type": "application/json",
    },
  }
);

const result = await response.json();

// Get multiple fields with custom date range
const response2 = await fetch(
  "https://datafa.st/api/v1/analytics/timeseries?fields=visitors,sessions,revenue&interval=day&startAt=2024-01-01&endAt=2024-01-31&timezone=America/New_York",
  {
    method: "GET",
    headers: {
      Authorization: `Bearer ${DATAFAST_API_KEY}`,
      "Content-Type": "application/json",
    },
  }
);

const result2 = await response2.json();

// Get revenue data filtered by country and device
const response3 = await fetch(
  "https://datafa.st/api/v1/analytics/timeseries?fields=revenue&interval=day&filter_country=is:United States,Canada&filter_device=is_not:mobile&limit=30",
  {
    method: "GET",
    headers: {
      Authorization: `Bearer ${DATAFAST_API_KEY}`,
      "Content-Type": "application/json",
    },
  }
);

const result3 = await response3.json();

// Get visitors filtered by UTM campaign and referrer
const response4 = await fetch(
  "https://datafa.st/api/v1/analytics/timeseries?fields=visitors&interval=day&filter_utm_campaign=is:summer_sale&filter_referrer=is:google.com,facebook.com",
  {
    method: "GET",
    headers: {
      Authorization: `Bearer ${DATAFAST_API_KEY}`,
      "Content-Type": "application/json",
    },
  }
);

const result4 = await response4.json();

// Get sessions filtered by page path (using contains operator)
const response5 = await fetch(
  "https://datafa.st/api/v1/analytics/timeseries?fields=sessions&interval=day&filter_page=contains:/blog&limit=30",
  {
    method: "GET",
    headers: {
      Authorization: `Bearer ${DATAFAST_API_KEY}`,
      "Content-Type": "application/json",
    },
  }
);

const result5 = await response5.json();
```

##### Success response (200 OK) - Multi-Field

```json
{
  "status": "success",
  "fields": [
    "visitors",
    "name",
    "sessions",
    "revenue",
    "conversion_rate"
  ],
  "interval": "day",
  "timezone": "Asia/Calcutta",
  "currency": "$",
  "totals": {
    "visitors": 14213,
    "sessions": 20181,
    "revenue": 27351,
    "revenueBreakdown": {
      "new": 22150.00,
      "renewal": 5201.00,
      "refund": 0.00
    },
    "conversion_rate": 1.92
  },
  "data": [
    {
      "visitors": 528,
      "name": "17 Dec",
      "sessions": 604,
      "revenue": 0,
      "revenueBreakdown": {
        "new": 0.00,
        "renewal": 0.00,
        "refund": 0.00
      },
      "conversion_rate": 0,
      "timestamp": "2025-12-17T00:00:00+05:30"
    },
    {
      "visitors": 472,
      "name": "18 Dec",
      "sessions": 534,
      "revenue": 468,
      "revenueBreakdown": {
        "new": 380.00,
        "renewal": 88.00,
        "refund": 0.00
      },
      "conversion_rate": 0.99,
      "timestamp": "2025-12-18T00:00:00+05:30"
    }
  ],
  "pagination": {
    "limit": 5,
    "offset": 0,
    "total": 30
  }
}
```

---

### Overview

Get aggregate analytics metrics including pageviews, visitors, sessions, bounce rate, revenue, and conversion rate.

# Get site overview

`GET https://datafa.st/api/v1/analytics/overview`

Retrieve aggregate analytics metrics for your website. Requires [Bearer Token](/docs/api-introduction) authentication.

## Response

**Success (200 OK):** Returns aggregate metrics

```json
{
  "status": "success",
  "data": [
    {
      "visitors": 12450,
      "sessions": 16890,
      "bounce_rate": 65.32,
      "avg_session_duration": 245678.45,
      "currency": "$",
      "revenue": 28450,
      "revenue_per_visitor": 2.29,
      "conversion_rate": 1.15
    }
  ]
}
```

## Query Parameters

- **`startAt`** (string, optional): Start date in ISO 8601 format (e.g., `2024-01-01` or `2024-01-01T00:00:00Z`)
- **`endAt`** (string, optional): End date in ISO 8601 format (e.g., `2024-01-31` or `2024-01-31T23:59:59Z`)
- **`timezone`** (string, optional): Timezone in IANA format (e.g., `America/New_York`, `UTC`). Defaults to website timezone
- **`fields`** (string, optional): Comma-separated list of fields to return. Valid fields: `visitors`, `sessions`, `bounce_rate`, `avg_session_duration`, `currency`, `revenue`, `revenue_per_visitor`, `conversion_rate`. If not specified, all fields are returned.

## Notes

- **By default, if `startAt` and `endAt` are not provided, the endpoint returns all-time aggregated data**
- If only one of `startAt` or `endAt` is provided, the API returns a 400 error - both must be provided together
- Conversion rate is calculated as a percentage (e.g., 1.15 for 1.15%)

## Errors

- **400 Bad Request:** Invalid parameters or missing required parameters (e.g., only one of `startAt` or `endAt` provided)
- **401 Unauthorized:** Missing or invalid API key
- **404 Not Found:** Website not found
- **500 Internal Server Error:** Server-side issue

#### Code examples

##### Example request (Node.js/Express)

```javascript
// Get all-time overview
const response = await fetch(
  "https://datafa.st/api/v1/analytics/overview",
  {
    method: "GET",
    headers: {
      Authorization: `Bearer ${DATAFAST_API_KEY}`,
      "Content-Type": "application/json",
    },
  }
);

const result = await response.json();

// Get overview for specific date range
const response2 = await fetch(
  "https://datafa.st/api/v1/analytics/overview?startAt=2024-01-01&endAt=2024-01-31&timezone=America/New_York",
  {
    method: "GET",
    headers: {
      Authorization: `Bearer ${DATAFAST_API_KEY}`,
      "Content-Type": "application/json",
    },
  }
);

const result2 = await response2.json();
```

##### Success response (200 OK)

```json
{
  "status": "success",
  "data": [
    {
      "visitors": 12450,
      "sessions": 16890,
      "bounce_rate": 65.32,
      "avg_session_duration": 245678.45,
      "currency": "$",
      "revenue": 28450,
      "revenue_per_visitor": 2.29,
      "conversion_rate": 1.15
    }
  ]
}
```

---

### Metadata

Get website configuration metadata including domain, timezone, name, KPI color scheme, KPI goal, and currency.

# Get website metadata

`GET https://datafa.st/api/v1/analytics/metadata`

Retrieve website configuration metadata for your site. Requires [Bearer Token](/docs/api-introduction) authentication.

## Response

**Success (200 OK):** Returns website configuration metadata

```json
{
  "status": "success",
  "data": [
    {
      "domain": "example.com",
      "timezone": "America/New_York",
      "name": "My Awesome Website",
      "kpiColorScheme": "orange",
      "kpi": "signup",
      "currency": "USD"
    }
  ]
}
```

### Response fields

- **`domain`** (string): The website domain
- **`timezone`** (string): Website timezone in IANA timezone format (e.g., "America/New_York", "UTC")
- **`name`** (string): Custom website name
- **`kpiColorScheme`** (string): Color scheme for KPI visualization. Possible values: "red", "orange", "yellow", "green", "purple", "pink", "gray", "blue", "teal", "indigo"
- **`kpi`** (string | null): Custom goal/event name set as the key performance indicator for this website
- **`currency`** (string): Currency code (e.g., "USD", "EUR", "GBP")

## Errors

- **401 Unauthorized:** Missing or invalid API key
- **404 Not Found:** Website not found
- **500 Internal Server Error:** Server-side issue

#### Code examples

##### Example request (Node.js/Express)

```javascript
const response = await fetch(
  "https://datafa.st/api/v1/analytics/metadata",
  {
    method: "GET",
    headers: {
      Authorization: `Bearer ${DATAFAST_API_KEY}`,
      "Content-Type": "application/json",
    },
  }
);

const result = await response.json();
console.log(result);
```

##### Success response (200 OK)

```json
{
  "status": "success",
  "data": [
    {
      "domain": "example.com",
      "timezone": "America/New_York",
      "name": "My Awesome Website",
      "kpiColorScheme": "orange",
      "kpi": "signup",
      "currency": "USD"
    }
  ]
}
```

---

### Realtime

Get real-time visitor count for your website (visitors active in the last 5 minutes).

# Get real-time visitors

`GET https://datafa.st/api/v1/analytics/realtime`

Retrieve the count of active visitors on your website in real-time (visitors with activity in the last 5 minutes). Requires [Bearer Token](/docs/api-introduction) authentication.

## Response

**Success (200 OK):** Returns real-time visitor count

```json
{
  "status": "success",
  "data": [
    {
      "visitors": 42
    }
  ]
}
```

## Notes

- Real-time data shows visitors who have had activity (pageviews, events) within the last 5 minutes
- This endpoint is optimized for frequent polling with shorter cache times
- Unlike other analytics endpoints, this does not support date range filtering as it always shows current activity

## Errors

- **401 Unauthorized:** Missing or invalid API key
- **404 Not Found:** Website not found
- **500 Internal Server Error:** Server-side issue

#### Code examples

##### Example request (Node.js/Express)

```javascript
const response = await fetch(
  "https://datafa.st/api/v1/analytics/realtime",
  {
    method: "GET",
    headers: {
      Authorization: `Bearer ${DATAFAST_API_KEY}`,
      "Content-Type": "application/json",
    },
  }
);

const result = await response.json();
console.log(`Active visitors: ${result.data[0].visitors}`);
```

##### Success response (200 OK)

```json
{
  "status": "success",
  "data": [
    {
      "visitors": 42
    }
  ]
}
```

---

### Realtime Map

Get real-time visitors, events, and recent payments for the realtime map.

# Get realtime map data

`GET https://datafa.st/api/v1/analytics/realtime/map`

Retrieve active visitors (last 10 minutes), recent events, and recent payments for the realtime map. Requires [Bearer Token](/docs/api-introduction) authentication.

## Response

**Success (200 OK):** Returns realtime map data

```json
{
  "status": "success",
  "data": {
    "count": 2,
    "visitors": [
      {
        "visitorId": "b971cbbf-7ce2-4925-902b-8f95a5fb3449",
        "location": {
          "city": "Denver",
          "region": "US-CO",
          "countryCode": "US"
        },
        "system": {
          "browser": { "name": "Chrome" },
          "os": { "name": "Mac OS" },
          "device": { "type": "desktop" }
        },
        "currentUrl": "/pricing",
        "referrer": null,
        "sessionStartTime": "2026-02-06T05:27:20.000Z",
        "visitCount": 1,
        "params": {
          "ref": null,
          "source": null,
          "via": null,
          "utm_source": null,
          "utm_medium": null,
          "utm_campaign": null,
          "utm_content": null,
          "utm_term": null
        },
        "latitude": 39.7391,
        "longitude": -104.9866,
        "isCustomer": false,
        "customerName": null,
        "customerEmail": null,
        "profileData": {
          "displayName": null,
          "userId": null,
          "hasProfile": false,
          "isRandomName": true
        },
        "conversionLikelihood": {
          "score": 65,
          "confidence": 0.763,
          "isBaseline": false,
          "dimensionMatches": ["device", "os", "browser", "country", "referrer"],
          "raw": {
            "conversionRate": 0.0056,
            "averageValue": 246.98,
            "expectedValue": 1.37
          }
        }
      }
    ],
    "conversionMetrics": {
      "baselineConversionRate": 0.0044,
      "baselineAverageValue": 243.63
    },
    "hasConversionPredictions": true,
    "recentEvents": [
      {
        "_id": "69857bb827ddefd472fb2b09",
        "type": "pageview",
        "visitorId": "b971cbbf-7ce2-4925-902b-8f95a5fb3449",
        "timestamp": "2026-02-06T05:27:20.000Z",
        "path": "/pricing",
        "countryCode": "US",
        "extraData": {},
        "referrer": null,
        "customerName": null,
        "amount": null,
        "displayName": null
      }
    ],
    "recentPayments": [
      {
        "_id": "69857bb827ddefd472fb2b12",
        "type": "payment_received",
        "timestamp": "2026-02-06T05:26:10.000Z",
        "name": "Jane Doe",
        "email": "jane@example.com",
        "amount": 49,
        "currency": "USD",
        "renewal": false,
        "displayName": null,
        "isNew": false,
        "visible": true
      }
    ]
  }
}
```

## Notes

- Time window is the last **10 minutes** for active visitors, recent events, and recent payments.
- `conversionLikelihood` and `conversionMetrics` are included only when conversion modeling data is available.
- `hasConversionPredictions` will be `false` when no conversion metrics exist yet.

## Errors

- **401 Unauthorized:** Missing or invalid API key
- **404 Not Found:** Website not found
- **500 Internal Server Error:** Server-side issue

#### Code examples

##### Example request (Node.js/Express)

```javascript
const response = await fetch(
  "https://datafa.st/api/v1/analytics/realtime/map",
  {
    method: "GET",
    headers: {
      Authorization: `Bearer ${DATAFAST_API_KEY}`,
      "Content-Type": "application/json",
    },
  }
);

const result = await response.json();
console.log(result);
```

##### Success response (200 OK)

```json
{
  "status": "success",
  "data": {
    "count": 2,
    "visitors": [ ... ],
    "recentEvents": [ ... ],
    "recentPayments": [ ... ],
    "hasConversionPredictions": true
  }
}
```

---

### Devices

Get analytics data broken down by device type (desktop, mobile, tablet).

# Get device analytics

`GET https://datafa.st/api/v1/analytics/devices`

Retrieve analytics data broken down by device type. Requires [Bearer Token](/docs/api-introduction) authentication.

## Response

**Success (200 OK):**

```json
{
  "status": "success",
  "data": [
    {
      "device": "desktop",
      "visitors": 7892,
      "revenue": 18234
    },
    {
      "device": "mobile",
      "visitors": 6124,
      "revenue": 8934
    },
    {
      "device": "tablet",
      "visitors": 234,
      "revenue": 567
    }
  ],
  "pagination": {
    "limit": 100,
    "offset": 0,
    "total": 3
  }
}
```

#### Code examples

##### Example request (Node.js/Express)

```javascript
const response = await fetch(
  "https://datafa.st/api/v1/analytics/devices?limit=10",
  {
    method: "GET",
    headers: {
      Authorization: `Bearer ${DATAFAST_API_KEY}`,
      "Content-Type": "application/json",
    },
  }
);

const result = await response.json();
```

##### Success response (200 OK)

```json
{
  "status": "success",
  "data": [
    {
      "device": "desktop",
      "visitors": 7892,
      "revenue": 18234
    },
    {
      "device": "mobile",
      "visitors": 6124,
      "revenue": 8934
    },
    {
      "device": "tablet",
      "visitors": 234,
      "revenue": 567
    }
  ],
  "pagination": {
    "limit": 100,
    "offset": 0,
    "total": 3
  }
}
```

---

### Pages

Get analytics data broken down by page (hostname and path).

# Get page analytics

`GET https://datafa.st/api/v1/analytics/pages`

Retrieve analytics data broken down by page. Requires [Bearer Token](/docs/api-introduction) authentication.

## Response

**Success (200 OK):**

```json
{
  "status": "success",
  "data": [
    {
      "hostname": "example.com",
      "path": "/",
      "visitors": 8920,
      "revenue": 12450
    },
    {
      "hostname": "example.com",
      "path": "/pricing",
      "visitors": 2150,
      "revenue": 8934
    },
    {
      "hostname": "example.com",
      "path": "/about",
      "visitors": 1840,
      "revenue": 0
    },
    {
      "hostname": "example.com",
      "path": "/blog",
      "visitors": 1230,
      "revenue": 0
    }
  ],
  "pagination": {
    "limit": 100,
    "offset": 0,
    "total": 243
  }
}
```

#### Code examples

##### Example request (Node.js/Express)

```javascript
const response = await fetch(
  "https://datafa.st/api/v1/analytics/pages?limit=20",
  {
    method: "GET",
    headers: {
      Authorization: `Bearer ${DATAFAST_API_KEY}`,
      "Content-Type": "application/json",
    },
  }
);
```

##### Success response (200 OK)

```json
{
  "status": "success",
  "data": [
    {
      "hostname": "example.com",
      "path": "/",
      "visitors": 8920,
      "revenue": 12450
    },
    {
      "hostname": "example.com",
      "path": "/pricing",
      "visitors": 2150,
      "revenue": 8934
    },
    {
      "hostname": "example.com",
      "path": "/about",
      "visitors": 1840,
      "revenue": 0
    }
  ],
  "pagination": {
    "limit": 100,
    "offset": 0,
    "total": 243
  }
}
```

---

### Campaigns

Get analytics data broken down by campaign parameters (UTM, ref, source, via).

# Get campaign analytics

`GET https://datafa.st/api/v1/analytics/campaigns`

Retrieve analytics data broken down by campaign parameters. Requires [Bearer Token](/docs/api-introduction) authentication.

## Response

**Success (200 OK):**

```json
{
  "status": "success",
  "data": [
    {
      "campaign": {
        "utm_source": "",
        "utm_medium": "",
        "utm_campaign": "",
        "utm_term": "",
        "utm_content": "",
        "ref": "indiepage",
        "source": "",
        "via": ""
      },
      "visitors": 2150,
      "revenue": 3420
    },
    {
      "campaign": {
        "utm_source": "newsletter.example.com",
        "utm_medium": "email",
        "utm_campaign": "product-launch-2025",
        "utm_term": "",
        "utm_content": "",
        "ref": "",
        "source": "",
        "via": ""
      },
      "visitors": 1840,
      "revenue": 2890
    },
    {
      "campaign": {
        "utm_source": "",
        "utm_medium": "",
        "utm_campaign": "",
        "utm_term": "",
        "utm_content": "",
        "ref": "",
        "source": "",
        "via": "partner"
      },
      "visitors": 920,
      "revenue": 1450
    }
  ],
  "pagination": {
    "limit": 5,
    "offset": 0,
    "total": 138
  }
}
```

#### Code examples

##### Example request (Node.js/Express)

```javascript
// Get all campaign fields
const response = await fetch(
  "https://datafa.st/api/v1/analytics/campaigns",
  {
    method: "GET",
    headers: {
      Authorization: `Bearer ${DATAFAST_API_KEY}`,
      "Content-Type": "application/json",
    },
  }
);

// Get only specific UTM fields
const response2 = await fetch(
  "https://datafa.st/api/v1/analytics/campaigns?fields=utm_source,utm_medium,utm_campaign",
  {
    method: "GET",
    headers: {
      Authorization: `Bearer ${DATAFAST_API_KEY}`,
      "Content-Type": "application/json",
    },
  }
);
```

##### Success response (200 OK)

```json
{
  "status": "success",
  "data": [
    {
      "campaign": {
        "utm_source": "",
        "utm_medium": "",
        "utm_campaign": "",
        "utm_term": "",
        "utm_content": "",
        "ref": "indiepage",
        "source": "",
        "via": ""
      },
      "visitors": 2150,
      "revenue": 3420
    },
    {
      "campaign": {
        "utm_source": "newsletter.example.com",
        "utm_medium": "email",
        "utm_campaign": "product-launch-2025",
        "utm_term": "",
        "utm_content": "",
        "ref": "",
        "source": "",
        "via": ""
      },
      "visitors": 1840,
      "revenue": 2890
    }
  ],
  "pagination": {
    "limit": 5,
    "offset": 0,
    "total": 138
  }
}
```

---

### Goals

Get analytics data broken down by custom goals.

# Get goal analytics

`GET https://datafa.st/api/v1/analytics/goals`

Retrieve analytics data broken down by custom goals. Requires [Bearer Token](/docs/api-introduction) authentication.

## Response

**Success (200 OK):**

```json
{
  "status": "success",
  "data": [
    {
      "goal": "faq_what_do_i_get",
      "completions": 49730,
      "visitors": 27797
    },
    {
      "goal": "scroll_to_problem",
      "completions": 35100,
      "visitors": 30807
    },
    {
      "goal": "faq_who_is_this_for",
      "completions": 32731,
      "visitors": 19905
    },
    {
      "goal": "faq_tech_stack",
      "completions": 29262,
      "visitors": 18708
    },
    {
      "goal": "faq_refund_policy",
      "completions": 27843,
      "visitors": 16861
    }
  ],
  "pagination": {
    "limit": 5,
    "offset": 0,
    "total": 48
  }
}
```

#### Code examples

##### Example request (Node.js/Express)

```javascript
// Get all goal data
const response = await fetch(
  "https://datafa.st/api/v1/analytics/goals",
  {
    method: "GET",
    headers: {
      Authorization: `Bearer ${DATAFAST_API_KEY}`,
      "Content-Type": "application/json",
    },
  }
);

// Get only specific fields
const response2 = await fetch(
  "https://datafa.st/api/v1/analytics/goals?fields=name,completions",
  {
    method: "GET",
    headers: {
      Authorization: `Bearer ${DATAFAST_API_KEY}`,
      "Content-Type": "application/json",
    },
  }
);
```

##### Success response (200 OK)

```json
{
  "status": "success",
  "data": [
    {
      "goal": "faq_what_do_i_get",
      "completions": 49730,
      "visitors": 27797
    },
    {
      "goal": "scroll_to_problem",
      "completions": 35100,
      "visitors": 30807
    },
    {
      "goal": "faq_who_is_this_for",
      "completions": 32731,
      "visitors": 19905
    },
    {
      "goal": "faq_tech_stack",
      "completions": 29262,
      "visitors": 18708
    },
    {
      "goal": "faq_refund_policy",
      "completions": 27843,
      "visitors": 16861
    }
  ],
  "pagination": {
    "limit": 5,
    "offset": 0,
    "total": 48
  }
}
```

---

### Referrers

Get analytics data broken down by referrer source.

# Get referrer analytics

`GET https://datafa.st/api/v1/analytics/referrers`

Retrieve analytics data broken down by referrer. Requires [Bearer Token](/docs/api-introduction) authentication.

## Response

**Success (200 OK):** Returns referrer, visitors, and revenue for each referrer source.

```json
{
  "status": "success",
  "data": [
    {
      "referrer": "X",
      "visitors": 3450,
      "revenue": 1240
    },
    {
      "referrer": "Direct/None",
      "visitors": 2890,
      "revenue": 4560
    },
    {
      "referrer": "example.com",
      "visitors": 2150,
      "revenue": 2890
    },
    {
      "referrer": "Google",
      "visitors": 1840,
      "revenue": 6780
    },
    {
      "referrer": "newsletter.example.com",
      "visitors": 1230,
      "revenue": 2150
    }
  ],
  "pagination": {
    "limit": 5,
    "offset": 0,
    "total": 119
  }
}
```

#### Code examples

##### Example request

```javascript
const response = await fetch(
  "https://datafa.st/api/v1/analytics/referrers",
  {
    method: "GET",
    headers: {
      Authorization: `Bearer ${DATAFAST_API_KEY}`,
    },
  }
);
```

##### Success response (200 OK)

```json
{
  "status": "success",
  "data": [
    {
      "referrer": "X",
      "visitors": 3450,
      "revenue": 1240
    },
    {
      "referrer": "Direct/None",
      "visitors": 2890,
      "revenue": 4560
    },
    {
      "referrer": "example.com",
      "visitors": 2150,
      "revenue": 2890
    },
    {
      "referrer": "Google",
      "visitors": 1840,
      "revenue": 6780
    }
  ],
  "pagination": {
    "limit": 5,
    "offset": 0,
    "total": 119
  }
}
```

---

### Countries

Get analytics data broken down by country.

# Get country analytics

`GET https://datafa.st/api/v1/analytics/countries`

Retrieve analytics data broken down by country. Requires [Bearer Token](/docs/api-introduction) authentication.

## Response

**Success (200 OK):** Returns country, image (flag SVG URL), visitors, and revenue for each country.

```json
{
  "status": "success",
  "data": [
    {
      "country": "United States of America",
      "image": "https://purecatamphetamine.github.io/country-flag-icons/3x2/US.svg",
      "visitors": 2840,
      "revenue": 6780
    },
    {
      "country": "France",
      "image": "https://purecatamphetamine.github.io/country-flag-icons/3x2/FR.svg",
      "visitors": 1230,
      "revenue": 2450
    },
    {
      "country": "United Kingdom",
      "image": "https://purecatamphetamine.github.io/country-flag-icons/3x2/GB.svg",
      "visitors": 920,
      "revenue": 1450
    },
    {
      "country": "Germany",
      "image": "https://purecatamphetamine.github.io/country-flag-icons/3x2/DE.svg",
      "visitors": 780,
      "revenue": 1230
    },
    {
      "country": "Spain",
      "image": "https://purecatamphetamine.github.io/country-flag-icons/3x2/ES.svg",
      "visitors": 650,
      "revenue": 980
    }
  ],
  "pagination": {
    "limit": 5,
    "offset": 0,
    "total": 148
  }
}
```

#### Code examples

##### Example request

```javascript
const response = await fetch(
  "https://datafa.st/api/v1/analytics/countries",
  {
    method: "GET",
    headers: {
      Authorization: `Bearer ${DATAFAST_API_KEY}`,
    },
  }
);
```

##### Success response (200 OK)

```json
{
  "status": "success",
  "data": [
    {
      "country": "United States of America",
      "image": "https://purecatamphetamine.github.io/country-flag-icons/3x2/US.svg",
      "visitors": 2840,
      "revenue": 6780
    },
    {
      "country": "France",
      "image": "https://purecatamphetamine.github.io/country-flag-icons/3x2/FR.svg",
      "visitors": 1230,
      "revenue": 2450
    },
    {
      "country": "United Kingdom",
      "image": "https://purecatamphetamine.github.io/country-flag-icons/3x2/GB.svg",
      "visitors": 920,
      "revenue": 1450
    },
    {
      "country": "Germany",
      "image": "https://purecatamphetamine.github.io/country-flag-icons/3x2/DE.svg",
      "visitors": 780,
      "revenue": 1230
    },
    {
      "country": "Spain",
      "image": "https://purecatamphetamine.github.io/country-flag-icons/3x2/ES.svg",
      "visitors": 650,
      "revenue": 980
    }
  ],
  "pagination": {
    "limit": 5,
    "offset": 0,
    "total": 148
  }
}
```

---

### Regions

Get analytics data broken down by region/state.

# Get region analytics

`GET https://datafa.st/api/v1/analytics/regions`

Retrieve analytics data broken down by region. Requires [Bearer Token](/docs/api-introduction) authentication.

## Response

**Success (200 OK):** Returns region, visitors, and revenue for each region.

```json
{
  "status": "success",
  "data": [
    {
      "region": "England",
      "visitors": 890,
      "revenue": 1240
    },
    {
      "region": "California",
      "visitors": 650,
      "revenue": 2150
    },
    {
      "region": "Île-de-France",
      "visitors": 520,
      "revenue": 980
    },
    {
      "region": "Ontario",
      "visitors": 420,
      "revenue": 780
    },
    {
      "region": "New York",
      "visitors": 380,
      "revenue": 650
    }
  ],
  "pagination": {
    "limit": 5,
    "offset": 0,
    "total": 656
  }
}
```

#### Code examples

##### Example request

```javascript
const response = await fetch(
  "https://datafa.st/api/v1/analytics/regions",
  {
    method: "GET",
    headers: {
      Authorization: `Bearer ${DATAFAST_API_KEY}`,
    },
  }
);
```

##### Success response (200 OK)

```json
{
  "status": "success",
  "data": [
    {
      "region": "England",
      "visitors": 890,
      "revenue": 1240
    },
    {
      "region": "California",
      "visitors": 650,
      "revenue": 2150
    },
    {
      "region": "Île-de-France",
      "visitors": 520,
      "revenue": 980
    },
    {
      "region": "Ontario",
      "visitors": 420,
      "revenue": 780
    },
    {
      "region": "New York",
      "visitors": 380,
      "revenue": 650
    }
  ],
  "pagination": {
    "limit": 5,
    "offset": 0,
    "total": 656
  }
}
```

---

### Cities

Get analytics data broken down by city.

# Get city analytics

`GET https://datafa.st/api/v1/analytics/cities`

Retrieve analytics data broken down by city. Requires [Bearer Token](/docs/api-introduction) authentication.

## Response

**Success (200 OK):** Returns city, visitors, and revenue for each city.

```json
{
  "status": "success",
  "data": [
    {
      "city": "Singapore",
      "visitors": 350,
      "revenue": 0
    },
    {
      "city": "Paris",
      "visitors": 280,
      "revenue": 890
    },
    {
      "city": "Jakarta",
      "visitors": 240,
      "revenue": 0
    },
    {
      "city": "Toronto",
      "visitors": 210,
      "revenue": 650
    },
    {
      "city": "Warsaw",
      "visitors": 200,
      "revenue": 0
    },
    {
      "city": "Bangkok",
      "visitors": 195,
      "revenue": 1240
    },
    {
      "city": "Istanbul",
      "visitors": 180,
      "revenue": 0
    },
    {
      "city": "Sydney",
      "visitors": 175,
      "revenue": 420
    },
    {
      "city": "New York",
      "visitors": 170,
      "revenue": 0
    },
    {
      "city": "Madrid",
      "visitors": 165,
      "revenue": 0
    }
  ],
  "pagination": {
    "limit": 100,
    "offset": 0,
    "total": 2297
  }
}
```

#### Code examples

##### Example request

```javascript
const response = await fetch(
  "https://datafa.st/api/v1/analytics/cities",
  {
    method: "GET",
    headers: {
      Authorization: `Bearer ${DATAFAST_API_KEY}`,
    },
  }
);
```

##### Success response (200 OK)

```json
{
  "status": "success",
  "data": [
    {
      "city": "Singapore",
      "visitors": 350,
      "revenue": 0
    },
    {
      "city": "Paris",
      "visitors": 280,
      "revenue": 890
    },
    {
      "city": "Jakarta",
      "visitors": 240,
      "revenue": 0
    },
    {
      "city": "Toronto",
      "visitors": 210,
      "revenue": 650
    },
    {
      "city": "Warsaw",
      "visitors": 200,
      "revenue": 0
    },
    {
      "city": "Bangkok",
      "visitors": 195,
      "revenue": 1240
    },
    {
      "city": "Istanbul",
      "visitors": 180,
      "revenue": 0
    },
    {
      "city": "Sydney",
      "visitors": 175,
      "revenue": 420
    },
    {
      "city": "New York",
      "visitors": 170,
      "revenue": 0
    },
    {
      "city": "Madrid",
      "visitors": 165,
      "revenue": 0
    }
  ],
  "pagination": {
    "limit": 100,
    "offset": 0,
    "total": 2297
  }
}
```

---

### Browsers

Get analytics data broken down by browser.

# Get browser analytics

`GET https://datafa.st/api/v1/analytics/browsers`

Retrieve analytics data broken down by browser. Requires [Bearer Token](/docs/api-introduction) authentication.

## Response

**Success (200 OK):** Returns browser, visitors, and revenue for each browser.

```json
{
  "status": "success",
  "data": [
    {
      "browser": "Chrome",
      "visitors": 8920,
      "revenue": 18450
    },
    {
      "browser": "Safari",
      "visitors": 3450,
      "revenue": 2890
    },
    {
      "browser": "Instagram",
      "visitors": 450,
      "revenue": 0
    },
    {
      "browser": "Firefox",
      "visitors": 380,
      "revenue": 0
    },
    {
      "browser": "Twitter",
      "visitors": 350,
      "revenue": 0
    },
    {
      "browser": "Edge",
      "visitors": 320,
      "revenue": 1450
    },
    {
      "browser": "LinkedIn",
      "visitors": 210,
      "revenue": 0
    },
    {
      "browser": "Samsung Internet",
      "visitors": 140,
      "revenue": 0
    },
    {
      "browser": "Facebook",
      "visitors": 135,
      "revenue": 580
    },
    {
      "browser": "Opera",
      "visitors": 110,
      "revenue": 1120
    }
  ],
  "pagination": {
    "limit": 100,
    "offset": 0,
    "total": 28
  }
}
```

#### Code examples

##### Example request

```javascript
const response = await fetch(
  "https://datafa.st/api/v1/analytics/browsers",
  {
    method: "GET",
    headers: {
      Authorization: `Bearer ${DATAFAST_API_KEY}`,
    },
  }
);
```

##### Success response (200 OK)

```json
{
  "status": "success",
  "data": [
    {
      "browser": "Chrome",
      "visitors": 8920,
      "revenue": 18450
    },
    {
      "browser": "Safari",
      "visitors": 3450,
      "revenue": 2890
    },
    {
      "browser": "Instagram",
      "visitors": 450,
      "revenue": 0
    },
    {
      "browser": "Firefox",
      "visitors": 380,
      "revenue": 0
    },
    {
      "browser": "Twitter",
      "visitors": 350,
      "revenue": 0
    },
    {
      "browser": "Edge",
      "visitors": 320,
      "revenue": 1450
    },
    {
      "browser": "LinkedIn",
      "visitors": 210,
      "revenue": 0
    },
    {
      "browser": "Samsung Internet",
      "visitors": 140,
      "revenue": 0
    },
    {
      "browser": "Facebook",
      "visitors": 135,
      "revenue": 580
    },
    {
      "browser": "Opera",
      "visitors": 110,
      "revenue": 1120
    }
  ],
  "pagination": {
    "limit": 100,
    "offset": 0,
    "total": 28
  }
}
```

---

### Operating Systems

Get analytics data broken down by operating system.

# Get operating system analytics

`GET https://datafa.st/api/v1/analytics/operating-systems`

Retrieve analytics data broken down by operating system. Requires [Bearer Token](/docs/api-introduction) authentication.

## Response

**Success (200 OK):** Returns operating_system, visitors, and revenue for each OS.

```json
{
  "status": "success",
  "data": [
    {
      "operating_system": "Mac OS",
      "visitors": 5120,
      "revenue": 15680
    },
    {
      "operating_system": "iOS",
      "visitors": 4560,
      "revenue": 2890
    },
    {
      "operating_system": "Windows",
      "visitors": 3240,
      "revenue": 4560
    },
    {
      "operating_system": "Android",
      "visitors": 2150,
      "revenue": 890
    },
    {
      "operating_system": "Linux",
      "visitors": 290,
      "revenue": 0
    }
  ],
  "pagination": {
    "limit": 5,
    "offset": 0,
    "total": 9
  }
}
```

#### Code examples

##### Example request

```javascript
const response = await fetch(
  "https://datafa.st/api/v1/analytics/operating-systems",
  {
    method: "GET",
    headers: {
      Authorization: `Bearer ${DATAFAST_API_KEY}`,
    },
  }
);
```

##### Success response (200 OK)

```json
{
  "status": "success",
  "data": [
    {
      "operating_system": "Mac OS",
      "visitors": 5120,
      "revenue": 15680
    },
    {
      "operating_system": "iOS",
      "visitors": 4560,
      "revenue": 2890
    },
    {
      "operating_system": "Windows",
      "visitors": 3240,
      "revenue": 4560
    },
    {
      "operating_system": "Android",
      "visitors": 2150,
      "revenue": 890
    },
    {
      "operating_system": "Linux",
      "visitors": 290,
      "revenue": 0
    }
  ],
  "pagination": {
    "limit": 5,
    "offset": 0,
    "total": 9
  }
}
```

---

### Hostnames

Get analytics data broken down by hostname/domain.

# Get hostname analytics

`GET https://datafa.st/api/v1/analytics/hostnames`

Retrieve analytics data broken down by hostname. Requires [Bearer Token](/docs/api-introduction) authentication.

## Response

**Success (200 OK):** Returns hostname, visitors, and revenue for each hostname.

```json
{
  "status": "success",
  "data": [
    {
      "hostname": "example.com",
      "visitors": 15240,
      "revenue": 24560
    }
  ],
  "pagination": {
    "limit": 100,
    "offset": 0,
    "total": 1
  }
}
```

#### Code examples

##### Example request

```javascript
const response = await fetch(
  "https://datafa.st/api/v1/analytics/hostnames",
  {
    method: "GET",
    headers: {
      Authorization: `Bearer ${DATAFAST_API_KEY}`,
    },
  }
);
```

##### Success response (200 OK)

```json
{
  "status": "success",
  "data": [
    {
      "hostname": "example.com",
      "visitors": 15240,
      "revenue": 24560
    }
  ],
  "pagination": {
    "limit": 100,
    "offset": 0,
    "total": 1
  }
}
```

---

### Goal

API endpoint to programmatically create a custom goal event for a visitor.

# Create a custom goal

`POST https://datafa.st/api/v1/goals`

Create a custom goal for a specific visitor. Requires [Bearer Token](/docs/api-introduction) authentication.

## Request body

Send a JSON object with the following fields:

- `datafast_visitor_id` (String, required): The DataFast unique ID of the visitor. It is stored in the visitor's browser cookies, making it accessible in your backend API endpoints.
- `name` (String, required): Name for the goal (lowercase letters, numbers, underscores, hyphens, max 64 chars).
- `metadata` (Object, optional): Custom parameters to enrich your event data.

### Custom parameters rules (in metadata object):
- **Property names:** lowercase letters, numbers, underscores (_), and hyphens (-) only. Max 64 characters.
- **Property values:** any string, max 255 characters. HTML and script content is automatically removed for security.
- **Limits:** maximum 10 custom parameters per event.

> A visitor needs to have at least one pageview before the goal can be created.

## Response

- **Success (200 OK):** Returns a confirmation message and the ID of the created event.
- **Errors:** See [API Introduction](/docs/api-introduction). A 400 occurs if the visitor is a bot. A 404 occurs if the visitor has no prior pageviews.

#### Code examples

##### Example request (Node.js/Express)

```javascript
// Your backend API endpoint
const handler = async (req, res) => {
  const datafast_visitor_id = req.cookies.datafast_visitor_id;

  try {
    const response = await fetch("https://datafa.st/api/v1/goals", {
      method: "POST",
      headers: {
        Authorization: `Bearer ${DATAFAST_API_KEY}`,
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        datafast_visitor_id: datafast_visitor_id,
        name: "newsletter_signup",
        metadata: {
          name: "Elon Musk",
          email: "musk@x.com",
        }
      }),
    });

    const result = await response.json();
    console.log("Goal sent to DataFast:", result);

    res.status(200).send("Goal tracked");
  } catch (error) {
    console.error("Error sending goal to DataFast:", error);
    res.status(500).send("Failed to track goal");
  }
};    

```

##### Success response (200 OK)

```json
{
  "status": "success",
  "data": [{
    "message": "Custom event created successfully",
    "eventId": "67f8b9b5c320277df9a9d681"
  }]
}
```

---

### Payment

Track revenue from any payment provider using custom webhook endpoint.


> If you're using Stripe, LemonSqueezy, or Polar, you don't need to use this endpoint. We automatically track payments if you have [connected your payment provider](/docs/connect-payment-provider).

# Create a payment

`POST https://datafa.st/api/v1/payments`

Track payments from any provider and attribute revenue to your traffic sources. Requires [Bearer Token](/docs/api-introduction) authentication.

## Request body

Send a JSON object with the following fields:

#### Required fields

- **`amount`** (number): Payment amount. Examples: `29.99` for $29.99, `0` for free trials
- **`currency`** (string): Currency code like `"USD"`, `"EUR"`, `"GBP"`
- **`transaction_id`** (string): Unique transaction ID from your payment provider

#### Optional fields

- **`datafast_visitor_id`** (string): DataFast visitor ID from browser cookies. **⚠️ It's highly recommended to include this field for revenue attribution.**
- **`email`** (string): Customer email
- **`name`** (string): Customer name
- **`customer_id`** (string): Customer ID from your payment provider
- **`renewal`** (boolean): Set to true if its a recurring payment. false by default
- **`refunded`** (boolean): Set to true if its a refunded payment. false by default
- **`timestamp`** (string): Payment timestamp (defaults to now)

## Response

- **Success (200 OK):** Returns confirmation message and transaction ID
- **Errors:** See [API Introduction](/docs/api-introduction)

> Here's an [example](/docs/payments-api) of how to add payment data and attribute revenue using this API endpoint.

#### Code examples

##### Example request (Node.js/Express)

```javascript
// Send payment data to DataFast's API
const handler = async (req, res) => {
  const datafast_visitor_id = req.cookies.datafast_visitor_id;

  try {
    const response = await fetch("https://datafa.st/api/v1/payments", {
      method: "POST",
      headers: {
        Authorization: `Bearer ${DATAFAST_API_KEY}`,
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        amount: 29.99,
        currency: "USD",
        transaction_id: "payment_456",
        datafast_visitor_id: datafast_visitor_id,
      }),
    });

    res.status(200).send("Payment tracked");
  } catch (error) {
    console.error("Error tracking payment:", error);
    res.status(500).send("Failed to track payment");
  }
};
```

##### Success responses (200 OK)

```json
{
  "message": "Payment recorded and attributed successfully",
  "transaction_id": "payment_456"
}
```

---

### Goal

Delete custom goal events by visitor, name, or time range.

# Delete goals

`DELETE https://datafa.st/api/v1/goals`

Delete custom goal events by visitor ID, event name, or within a time range. Requires [Bearer Token](/docs/api-introduction) authentication.

## Query Parameters

### Filter Parameters (Optional - at least one required if no time range)

- **`datafast_visitor_id`** (optional): Delete goals for a specific visitor
- **`name`** (optional): Delete goals with a specific event name (e.g., `signup`, `purchase`)
  - Will be auto-formatted to lowercase with underscores

### Time Range Parameters (Optional)

- **`startAt`** (optional): ISO 8601 timestamp (e.g., `2023-01-01T00:00:00Z`)
- **`endAt`** (optional): ISO 8601 timestamp (e.g., `2023-01-31T23:59:59Z`)

**⚠️ WARNING:** When using `datafast_visitor_id` or `name` WITHOUT time range, ALL matching records across all time will be deleted.

## Deletion Behavior

- **With time range only:** Deletes all goals in the specified time range
- **With visitor ID only:** Deletes all goals for that visitor (across all time)
- **With name only:** Deletes all goals with that name (across all time)
- **With visitor ID + name:** Deletes goals matching both criteria
- **With time range + filters:** Deletes goals matching the filters within the time range

## Response

- **Success (200 OK):** Returns confirmation of deletion.
- **Errors:** 400 if parameters are invalid, 404 if not found.

#### Code examples

##### Example request (Delete by name within time range)

```bash
curl -X DELETE "https://datafa.st/api/v1/goals?name=signup&startAt=2023-01-01T00:00:00Z&endAt=2023-01-31T23:59:59Z" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

##### Success response

```json
{
  "status": "success",
  "data": [{
    "message": "Goal events deleted successfully within time range",
    "goal_name": "signup",
    "timeRange": {
      "start": "2023-01-01T00:00:00.000Z",
      "end": "2023-01-31T23:59:59.000Z"
    }
  }]
}
```

---

### Payment

Delete payments by transaction ID, visitor ID, or time range.

# Delete payments

`DELETE https://datafa.st/api/v1/payments`

Delete payments using transaction ID, visitor ID, or time range. Requires [Bearer Token](/docs/api-introduction) authentication.

## Query Parameters

### Specific Deletion (Highest Priority)

- **`transaction_id`** or **`tx_id`** (optional): Delete a specific payment by transaction ID

### Filter by Visitor (Optional)

- **`datafast_visitor_id`** (optional): Delete all payments for a specific visitor
  - Can be combined with time range to limit deletion scope

### Time Range Parameters (Optional)

- **`startAt`** (optional): ISO 8601 timestamp (e.g., `2023-01-01T00:00:00Z`)
- **`endAt`** (optional): ISO 8601 timestamp (e.g., `2023-01-31T23:59:59Z`)

**⚠️ WARNING:** When using `datafast_visitor_id` WITHOUT time range, ALL payments for that visitor across all time will be deleted.

## Deletion Behavior

- **With transaction_id:** Deletes that specific payment (ignores other parameters)
- **With visitor ID only:** Deletes all payments for that visitor (across all time)
- **With visitor ID + time range:** Deletes payments for that visitor within the time range
- **With time range only:** Deletes all payments within the time range

## Response

- **Success (200 OK):** Returns confirmation of deletion.
- **Errors:** 404 if transaction not found, 400 if parameters invalid.

#### Code examples

##### Example request (Delete by visitor within time range)

```bash
curl -X DELETE "https://datafa.st/api/v1/payments?datafast_visitor_id=abc123&startAt=2023-01-01T00:00:00Z&endAt=2023-01-31T23:59:59Z" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

##### Success response

```json
{
  "status": "success",
  "data": [{
    "datafast_visitor_id": "abc123",
    "message": "All payments for visitor deleted successfully within time range",
    "timeRange": {
      "start": "2023-01-01T00:00:00.000Z",
      "end": "2023-01-31T23:59:59.000Z"
    }
  }]
}
```

---
