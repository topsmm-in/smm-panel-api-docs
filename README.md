# SMM Panel API v2 — TopSMM Reseller API Documentation

![API](https://img.shields.io/badge/API-v2-FF6B35?style=flat-square) ![Format](https://img.shields.io/badge/Format-JSON-0A66C2?style=flat-square) ![Auth](https://img.shields.io/badge/Auth-API%20key-6E40C9?style=flat-square) ![Status](https://img.shields.io/badge/Uptime-24%2F7-00B67A?style=flat-square)

Open documentation for the TopSMM reseller API. TopSMM is an Indian SMM panel that lets other panels, scripts, bots and WordPress plugins resell its catalogue of Instagram, YouTube, Facebook, Telegram, TikTok and Spotify services programmatically.

The API follows the widely used SMM panel API v2 convention, so if your software already talks to another panel, changing the endpoint and key is usually all that is needed.

Panel: https://topsmm.in

## Quick facts

| | |
| --- | --- |
| Endpoint | https://topsmm.in/api/v2 |
| Method | POST (form-encoded) |
| Response | JSON |
| Auth | `key` parameter, generated in your dashboard |
| Rate limit | Keep polling under 1 request per second per key |

## Authentication

Sign in at https://topsmm.in, open the API page and copy your personal API key. Send it as the `key` parameter on every request. The key carries your account balance — treat it like a password and never expose it in client-side JavaScript.

## Actions

| Action | Purpose |
| --- | --- |
| `services` | Full catalogue with IDs, names, rates, min and max |
| `add` | Place a new order |
| `status` | Status of a single order |
| `orders` | Status of up to 100 orders at once |
| `refill` | Request a refill on an eligible order |
| `refill_status` | Check a refill request |
| `balance` | Current account balance and currency |

## Place an order

| Parameter | Required | Description |
| --- | --- | --- |
| key | yes | Your API key |
| action | yes | `add` |
| service | yes | Service ID from the `services` response |
| link | yes | Profile, post, video or channel URL |
| quantity | yes | Number of units, between the service min and max |
| runs | no | Drip feed: how many runs to split the order into |
| interval | no | Drip feed: minutes between runs |

Request:

curl -X POST https://topsmm.in/api/v2 -d key=YOUR_API_KEY -d action=add -d service=101 -d link=https://instagram.com/yourprofile -d quantity=1000
```bash
curl -X POST https://topsmm.in/api/v2 -d key=YOUR_API_KEY -d action=add -d service=101 -d link=https://instagram.com/yourprofile -d quantity=1000
```

A successful response returns the new order ID. An `error` field is returned instead when something is wrong.

## Check order status

Send `action=status` with the `order` ID. The response reports status, charge, start count and remaining quantity.

| Status | Meaning |
| --- | --- |
| Pending | Accepted, waiting to be picked up by the provider |
| In progress | Delivery has started |
| Processing | Provider is working on the order |
| Completed | Delivery finished |
| Partial | Delivered partially, the unused amount is refunded to your balance |
| Canceled | Not delivered, fully refunded |

Use `action=orders` with a comma-separated list of up to 100 IDs instead of polling each order one by one.

## Refills

Services marked with a refill guarantee can be refilled if counts drop inside the guarantee window. Send `action=refill` with the order ID, then poll `action=refill_status` for the result.

## Errors

| Message | What to check |
| --- | --- |
| Incorrect request | Missing or misspelled `action` parameter |
| Invalid API key | Key is wrong, or regenerated in the dashboard |
| Not enough funds | Top up the balance before retrying |
| Incorrect service ID | Service was removed or renumbered — refresh the catalogue |
| Incorrect link format | Link does not match what the service expects |
| Quantity out of range | Below min or above max for that service |

## Code examples

PHP:

```php
$params = http_build_query([
    'key' => 'YOUR_API_KEY',
    'action' => 'add',
    'service' => 101,
    'link' => 'https://instagram.com/yourprofile',
    'quantity' => 1000,
]);
$response = file_get_contents('https://topsmm.in/api/v2?' . $params);
echo $response;
```

Python:

```python
import requests

payload = dict(
    key="YOUR_API_KEY",
    action="add",
    service=101,
    link="https://instagram.com/yourprofile",
    quantity=1000,
)
print(requests.post("https://topsmm.in/api/v2", data=payload).json())
```

## Best practices

Cache the `services` response and refresh it once or twice a day instead of on every page load. Batch status checks with `action=orders`. Store the order ID returned by `add` — it is the only way to track, refill or reconcile that order later. Always validate the link format on your side before sending it, because most failed orders come from a wrong profile or post URL.

## Getting an API key

Create a free account at https://topsmm.in, add funds, then open the API page in the dashboard to generate your key. The same catalogue and key work on the mirror domain https://masterofsmmpanel.in

## About TopSMM

TopSMM is an Indian SMM panel and reseller API provider founded in 2019 and operated from Hinjewadi, Pune. It supplies Instagram, YouTube, Facebook, Telegram, TikTok and Spotify marketing services at wholesale prices, with auto start, instant delivery, live order tracking, refill guarantee, Mass Order, Drip Feed and 24/7 support.

| | |
| --- | --- |
| Website | https://topsmm.in |
| YouTube tutorials | https://www.youtube.com/@topsmm_in |
| Telegram | https://t.me/topsmm_in |
| Reviews | https://www.trustpilot.com/review/topsmm.in |
| Email | topsmm.in@gmail.com |
| WhatsApp | +91 9455194264 |

## Disclaimer

This repository contains documentation only. Service IDs, rates and limits change over time — always read them from the live `services` response rather than hard-coding them.
