---
title: API Reference

language_tabs:
  - shell
  - ruby
  - python

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='http://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---
# Remitsy's Business Payment Platform

###In Private Beta Release

Easily integrate mass payouts of funds, with your existing business flows. Supports most popular file formats and programming languages for quick and easy integration.

# Quick Start

Book a Payment: 

1. Request an API Key
2. Get a Quote
3. Submit a Payment
4. Release a Payment
5. Check status of Payment

# Authentication

To receive an API key, please accept the terms below.

ELANCE, INC.
BETA API TERMS OF USE

This document governs the terms under which you may access and use Remitsy, Inc.'s ("Remitsy") application programming interface that is made available on this website (the "API"), and the data transmitted through the API (the "RemistyContent"). The API includes any API key or keys used to access the API.  This document incorporates the terms of the Remitsy Site Usage Policy and the other agreements and policies linked from the Remitsy Site Usage Policy, including all future amendments or modifications thereto (collectively, and together with this document, the "Beta API Terms of Use"):

BY CLICKING ON THE BUTTON MARKED "Accept," YOU AGREE TO USE THE API SOLELY IN ACCORDANCE WITH THE TERMS AND CONDITIONS OF THIS API AGREEMENT, AND YOU AGREE THAT YOU ARE BOUND BY AND ARE A PARTY TO THIS AGREEMENT.  YOU WARRANT THAT YOU ARE AT LEAST EIGHTEEN YEARS OLD AND THAT YOU HAVE THE LEGAL CAPACITY TO ENTER INTO CONTRACTS.  '

> To authorize, use this code:

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
```

```shell
# With shell, you can just pass the correct header with each request
curl --request POST \
-d 'quote[source_currency]=USD' \
-d 'quote[source_cents]=100001' \
-d 'quote[quote_currency]=CNY' \
-d 'quote[quote_cents]=650000' \
-d 'quote[arrive_by]=123412341234' \
https://remitsy-demo.herokuapp.com/apis/rates/v1/rates
```

> Make sure to replace `134231fd3r232rf32r` with your API key.


`Authorization: 134231fd3r232rf32r`

<aside class="notice">
You must replace `134231fd3r232rf32r` with your personal API key.
</aside>

# Payments

## Get a Quote

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get()
```

```shell
curl "http://example.com/api/kittens"
  -H "Authorization: meowmeowmeow"
```

> The above command returns JSON structured like this:

```json
[
  {
    "id": 1,
    "name": "Fluffums",
    "breed": "calico",
    "fluffiness": 6,
    "cuteness": 7
  },
  {
    "id": 2,
    "name": "Isis",
    "breed": "unknown",
    "fluffiness": 5,
    "cuteness": 10
  }
]
```

This endpoint retrieves all of your Payments.

### HTTP Request

`GET https://remitsy-demo.herokuapp.com/apis/rates/v1/rates`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
quote[source_currency] | USD | Currently supports HKD, EUR, USD, GBP, PLN
quote[source_cents] | 0 | Must be a Integer
quote[quote_currency] | CNY | FYI Only, unchangable
quote[quote_cents] | 0 | Must be a Integer
quote[arrive_by] | 72 hours | Unix timestamp, Different fees for 12 hours and 72 hours.

<aside class="success">
Remember — Quotes will expire if not used to make a booking withing 30 minutes!
</aside>

## Book a Payment

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get(2)
```

```shell
curl "http://example.com/api/kittens/3"
  -H "Authorization: meowmeowmeow"
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "name": "Isis",
  "breed": "unknown",
  "fluffiness": 5,
  "cuteness": 10
}
```

This endpoint retrieves a specific kitten.

<aside class="warning">If you're not using an administrator API key, note that some kittens will return 403 Forbidden if they are hidden for admins only.</aside>

### HTTP Request

`GET http://example.com/rate/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the cat to retrieve


## Release a Payment

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get(2)
```

```shell
curl "http://example.com/api/kittens/3"
  -H "Authorization: meowmeowmeow"
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "name": "Isis",
  "breed": "unknown",
  "fluffiness": 5,
  "cuteness": 10
}
```

This endpoint retrieves a specific kitten.

### HTTP Request

`GET https://remitsy-demo.herokuapp.com/apis/rate/v1/payment`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the cat to retrieve

