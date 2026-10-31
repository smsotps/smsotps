# SMS OTPS API

Checked against the live SMS OTPS API. The live API is the source of truth.

Use the SMS OTPS API to check offers, place number orders, read incoming verification messages, and cancel or resend from your own application.

## Get started

Create an account on smsotps.com and generate an API key from your profile. Use it in server-side requests only. Never place a live API key in browser code, screenshots or public repositories.

## Authentication

Send your API key with each request in the `X-API-KEY` header:

```
X-API-KEY: your_api_key_here
```

POST requests use JSON bodies with `Content-Type: application/json`.

## Base URL

```
https://api.smsotps.com/api
```

# API reference

## Balance

### GET /api/balance

Read your current wallet balance. API key required.

```
curl https://api.smsotps.com/api/balance -H "X-API-KEY: YOUR_API_KEY"
```

Response:

```json
{ "balance": 10.5 }
```

## Service and country catalogs

Service and country lists are published as static JSON files on the website - no API key required:

- `https://smsotps.com/provider_a_countries.json`
- `https://smsotps.com/provider_a_services.json`
- `https://smsotps.com/provider_b_countries.json`
- `https://smsotps.com/provider_b_services.json`

Country file example:

```json
{
  "16": { "name": "United Kingdom", "operators": ["any", "o2", "ee", "lebara", "three", "vodafone"] },
  "187": { "name": "USA", "operators": ["any", "tmobile", "at_t", "verizon"] }
}
```

Service file example (shortcode to display name):

```json
{ "fb": "facebook", "ig": "Instagram+Threads", "wa": "Whatsapp", "tg": "Telegram", "go": "Google,youtube,Gmail" }
```

## Operators

### GET /api/operators?country={country}

List operators available for a country. API key required. `country` is an optional numeric country ID.

## Offers

### GET /api/p_a/offers/{service}/{country}

### GET /api/p_b/offers/{service}/{country}

Check live price and stock for one service and country. Public - no API key required.

- `service` - service shortcode from the catalog, for example `fb`
- `country` - numeric country ID from the catalog

```
curl https://api.smsotps.com/api/p_a/offers/fb/1
```

Response - a list of operator groups, each with its current offers. `price` is a string, `count` is the available stock:

```json
[
  {
    "name": "any",
    "localName": "Any Operator",
    "offers": [
      { "price": "0.1384", "count": 50 },
      { "price": "0.1607", "count": 1103 }
    ]
  }
]
```

An empty list means the combination is unavailable right now. Offer data can change; check shortly before ordering.

## Order a number

### POST /api/order-number

Place a number order. This request charges your wallet. API key required.

JSON body:

- `service` - required string. Service shortcode, for example `"fb"`
- `country` - required integer. Numeric country ID
- `provider` - optional string. `"provider_a"` (default) or `"provider_b"`. For bulk link orders use `"provider_c"` - see below
- `operator` - optional string. Carrier operator code; omit or use `"any"` to accept any carrier
- `max_price` / `maxPrice` - optional number. Maximum price per number
- `min_price` / `minPrice` - optional number. Minimum price per number
- `any_price` / `anyPrice` - optional boolean. Accept any price
- `pricing_option` / `pricingOption` - optional string. Defaults to `"cheapest"`
- `idempotency_key` - optional UUID. Repeating the same purchase with the same key returns the original result instead of charging again; reusing the key with a different payload is rejected

```
curl -X POST https://api.smsotps.com/api/order-number \
  -H "X-API-KEY: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"service":"fb","country":1}'
```

Success response:

```json
{
  "id": 98514,
  "phone": "+17799042123",
  "price": 0.14,
  "balance_remaining": 9.86
}
```

IDs and values above are placeholders. Main errors: HTTP 400 when the selection is unavailable, the price filter matches nothing, or the balance is insufficient. HTTP 409 when an identical purchase is still being processed or the order needs review - in that case do not retry; check Active Numbers or contact support.

## Check order status

### GET /api/number-status/{id}

Read the current status of one of your orders. API key required.

```
curl https://api.smsotps.com/api/number-status/98514 -H "X-API-KEY: YOUR_API_KEY"
```

Response - the status is a single string:

```json
{ "status": "STATUS_WAIT_CODE" }
```

- `STATUS_WAIT_CODE` - waiting for the SMS
- `STATUS_OK:<code>` - the verification code has arrived, for example `STATUS_OK:845120`
- A cancelled or expired order returns its stored status

## Cancel an order

### POST /api/cancel-number/{id}

Request cancellation of an eligible order. API key required. On success the order price is returned to the wallet balance.

```
curl -X POST https://api.smsotps.com/api/cancel-number/98514 -H "X-API-KEY: YOUR_API_KEY"
```

Success response:

```json
{
  "message": "Number cancelled and balance refunded",
  "refunded_amount": 0.14,
  "new_balance": 10.5
}
```

The API allows cancellation only while an order is still active, before any SMS code has arrived, and between 2 and 20 minutes after purchase; outside that window it returns an error. The API rejects cancellation of link orders (`provider_c`).

## Resend the SMS

### POST /api/resend-sms/{id}

Request another SMS for an active order. API key required.

```
curl -X POST https://api.smsotps.com/api/resend-sms/98514 -H "X-API-KEY: YOUR_API_KEY"
```

Success response:

```json
{ "message": "Resend request successful. Waiting for new SMS." }
```

Resend is available only for active orders within 20 minutes of purchase.

## Order history

### GET /api/history

List your orders from the last 14 days, newest first, 10 per page. API key required. Use `?page=2` for older pages.

```
curl "https://api.smsotps.com/api/history?page=1" -H "X-API-KEY: YOUR_API_KEY"
```

The response is a paginated list of orders with their current status (`active`, `received`, `expired`, `cancelled`, `refunded`).

## Bulk link orders

Link orders are a separate product: you order verification link numbers in bulk. The API rejects cancellation for these orders.

### GET /api/p_c/options

List the services and countries available for link orders. Public - no API key required. Note that link-order service IDs are full names (for example `facebook`), not the shortcodes used elsewhere.

```
curl https://api.smsotps.com/api/p_c/options
```

```json
{
  "services": [{ "id": "facebook", "name": "Facebook" }],
  "countries": [{ "id": 1, "name": "USA" }]
}
```

### GET /api/p_c/offers/{service}/{country}

Check live price and stock for a link-order service and country. Public - no API key required.

```
curl https://api.smsotps.com/api/p_c/offers/facebook/1
```

```json
[{ "name": "any", "offers": [{ "price": "0.04", "count": 365 }] }]
```

### POST /api/order-number (link order)

Order link numbers in quantity. API key required. This request charges your wallet.

JSON body:

- `provider` - exactly `"provider_c"`
- `service` - service ID from `/api/p_c/options`, for example `"facebook"`
- `country` - country ID from `/api/p_c/options`
- `quantity` - integer 1-100
- `idempotency_key` - optional UUID, same replay rules as standard orders

```
curl -X POST https://api.smsotps.com/api/order-number \
  -H "X-API-KEY: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"provider":"provider_c","service":"facebook","country":1,"quantity":2}'
```

Success response - one entry per number, each with its own order ID, phone and masked verification link:

```json
{
  "id": 561075,
  "bulk": true,
  "quantity": 2,
  "numbers": [
    { "order_id": 561075, "phone": "+18159801235", "masked_link": "https://example.com/p_c=id:XXXXXXXXXX" },
    { "order_id": 561076, "phone": "+18159801236", "masked_link": "https://example.com/p_c=id:XXXXXXXXXY" }
  ],
  "balance_remaining": 9.76
}
```

IDs and values above are placeholders. Main errors: HTTP 400 when the service/country is unavailable, stock is short, or the balance is insufficient.

## Safe retries and charges

Every order request charges your wallet on success, so never silently repeat a purchase. Pass your own `idempotency_key` (a UUID) when your code might retry: a retry with the same key and the same payload returns the original result, and a key reused with a different payload is rejected with HTTP 409. Without a key, check `GET /api/number-status/{id}` or your dashboard before sending the same order again. HTTP 401 means the API key is missing or invalid; HTTP 422 means a request field failed validation.

Need help with an integration? Contact support@smsotps.com.
