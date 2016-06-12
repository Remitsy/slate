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

# Introduction

Remitsy's API Documentation

# Authentication

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

1. Get a Quote
2. Get a API key
3. Submit a Payment
4. Check status of Payment

`Authorization: 134231fd3r232rf32r`

<aside class="notice">
You must replace `134231fd3r232rf32r` with your personal API key.
</aside>

# Quotes

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

