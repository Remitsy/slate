---
title: API Reference

language_tabs:
  - shell

toc_footers:
  - <a href='http://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:

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

HTTP Token authentication, passed in the header. 

Keys are available upon request via, support@remitsy.com.

Requests without a valid API key will return:

<code>
    Code: 401 UNAUTHORIZED

    Content: { error : "unauthorized access"  }
</code>

> To authorize, use this code:

```shell
# With shell, you can just pass the correct header with each request
curl -X GET \
-H 'Authorization: Token token=<API KEY>' \
-H "Content-Type: text/plain; charset=UTF-8" \
http://remitsy-demo.herokuapp.com/apis/uni/v1/rates
```

> Make sure to replace `<API KEY>` with your API key.

# Payments

## Get a Quote

> To create a new Quote, use this code:

```shell
curl --request POST \
-H 'Authorization: Token token=<API KEY>' \
-H "Content-Type: text/plain; charset=UTF-8" \
-d 'quote[source_currency]=USD' \
-d 'quote[source_cents]=10000' \
https://remitsy-demo.herokuapp.com/apis/rates/v1/rates
```

> The above command returns JSON structured like this:

```json
{"quote": {
    "token": "<id token>",
    "rate": 6.51,
    "currency_pair": "USDCNY",
    "quote_cents": 65100,
    "source_cents": 10000,
    "expire_at": "<UNIX TIMESTAMP>"
  }
}
```

> Errors structured like this:

```json
{"errors": {
  "source_currency": [
    "Must be one of HKD, EUR, USD, PLN or GBP",
    "can't be blank"
  ],
  "source_cents": [
    "can't be blank",
    "Must be an integer"
  ]}
} 
```


This endpoint retrieves all of your Payments.

### HTTP Request

`GET https://remitsy-demo.herokuapp.com/apis/rates/v1/rates`

### Query Parameters

Parameter | Default | Type | Description
--------- | ------- | ---- | -----------
quote[source_currency] | USD | String(3) | Currently supports HKD, EUR, USD, GBP, PLN
quote[source_cents] | 0 | Integer(8) | All currency amounts are denominated in cents. Rounding is done using Ruby 2.0.0, [BigDecimal::ROUND_HALF_EVEN](http://ruby-doc.org/stdlib-2.0.0/libdoc/bigdecimal/rdoc/BigDecimal.html).

### Response

`Status 201 CREATED`

Parameter | Default | Type | Description
--------- | ------- | ---- | -----------
quote[token] |  | String(8) | Use this token to claim a rate when submitting a new Payment. 
quote[rate] |  | Decimal(4.6) | The rate includes our fee. 
quote[currency_pair] | USDCNY | String(3) | HKDCNY, EURCNY, USDCNY, GBPCNY, PLNCNY.
quote[source_cents] | 0 | Integer(8) | Half even rounding. 
quote[quote_cents] | 0 | Integer(8) | Half even rounding. 
quote[expire_at] | 30min | Unix Timestamp | Quotes will expire if not used to make a booking before the expiry_at attribute. Default value is 30 minutes.

<aside class="notice">
  All currency amounts are denominated in cents. Rounding is done using Ruby 2.0.0, 
  <a href="http://ruby-doc.org/stdlib-2.0.0/libdoc/bigdecimal/rdoc/BigDecimal.html">BigDecimal::ROUND_HALF_EVEN</a>.
</aside>

`Status 422 UNPROCESSABLE ENTITY`

Parameter | Description
--------- | -----------
error[source_currency] | Must be one of HKD EUR USD PLN GBP. 
error[source_cents] | Must be an integer. 


## Book a Ali-Pay Payment

> For Ali-Pay Payment

```shell
curl -X POST \
-H 'Authorization: Token token="<API KEY>"' \
-H "Content-Type: text/plain; charset=UTF-8" \
-H "ali_pay_id=1235812895"
-d "email=richard@163.com" \
-d "phone=13810456155" \
-d "name_cn=贝礼德" \
-d "identity_card_number=110101198109022323" \
https://remitsy-demo.herokuapp.com/apis/uni/v1/new
```

> The above commands returns JSON structured like this:

```json
{ "booking_ref": "xxxxxxxxxxxx" }
```

> Errors structured like this:

```json
{"errors": {
  "token": [
    "Invalid Token", 
    "Expired Quote"
  ],
  "name_cn": [
    "can't be blank",
    "Must be Chinese Characters, UTF-8 encoded."
  ],
  "email":[
    "invalid Email",
    "can't be blank"
  ],
  "phone":[
    "Phone number is not valid",
    "can't be blank"
  ],
  "identity_card_number":[
    "can't be blank",
    "Failed Checksum"
  ],
  "ali_pay_id": [
    "can't be blank",
    "Invalid Ali Pay ID"              
  ]}
}
```


Use a Quote token to book a new Payment at the quoted rate. Supports Ali-Pay and Bank Deposit. 

### HTTP Request

`GET https://remitsy-demo.herokuapp.com/apis/uni/v1/ali_pay`

### Query Parameters

Parameter | Type | Default | Description
--------- | ---- | ------- | -----------
payment[ali_pay_id] | String(255) | | Like a PayPal ID, can be a email, phone number or serial number.
payment[email] | String(255) | | `/\A[\w+\-.]+@[a-z\d\-]+(\.[a-z]+)*\.[a-z]+\z/i`
payment[phone] | String(255) | | `^(13[0-9]|14[57]|15[012356789]|17[0678]|18[0-9])[0-9]{8}$`
payment[name_cn] | String(255) | | Full name in simplified Chinese Characters UTF-8 encoded.
payment[identity_card_number] | String(18) | | `^[A-Za-z0-9]{15,17}$` & Checksum validation.
payment[token] | String(8) | | Provide a Quote token to claim the rate and amount.
payment[immediate_release] | String(true&#124;false) | true | Optional, set to false to hold payment until you release the payment. This allows additional KYC checks to be performed.

### Response

`Status 201 CREATED`

Parameter | Type | Description
--------- | ---- | -----------
payment[payment_ref] | String(8) | For future reference. 

`Status 422 UNPROCESSABLE ENTITY`

Parameter | Description
--------- | -----------
error[email] | Invalid email
error[phone] | Invalid phone
error[name_cn] | Must be Chinese Characters UTF-8 encoded.
error[identity_card_number] | Failed Checksum
error[immediate_release] | Must be either true &#124; false


## Book a Bank Payment

> For Bank Deposit Payment

```shell
curl -X POST \
-H 'Authorization: Token token="<API KEY>"' \
-H "Content-Type: text/plain; charset=UTF-8" \
-d "email=richard@163.com" \
-d "phone=13810456155" \
-d "name_cn=贝礼德" \
-d "identity_card_number=110101198109022323" \
-d "bank_account_number=6212260200082726700" \
-d "bank_account_name=中国银行" \
-d "bank_account_city=北京" \
-d "bank_account_branch=朝阳门支行" \
https://remitsy-demo.herokuapp.com/apis/uni/v1/new
```

> The above commands returns JSON structured like this:

```json
{"payment_ref": "xxxxxxxxxx"}
```

> Errors structured like this:

```json
{"errors": {
  "token": [
    "Invalid Token", 
    "Expired Quote"
  ],
  "name_cn": [
    "can't be blank",
    "Must be Chinese Characters, UTF-8 encoded."
  ],
  "bank_account_number": [
    "can't be blank",
    "invalid"
  ],
  "bank_account_name": [
    "can't be blank",
    "Must be Chinese Characters, UTF-8 encoded."
  ],
  "bank_account_city": [
    "can't be blank",
    "Must be Chinese Characters, UTF-8 encoded."
  ],
  "bank_account_branch": [
    "can't be blank",
    "Must be Chinese Characters, UTF-8 encoded."
  ],
  "email":[
    "invalid Email",
    "can't be blank"
  ],
  "phone":[
    "Phone number is not valid",
    "can't be blank"
  ],
  "identity_card_number":[
    "can't be blank",
    "Failed Checksum"
  ]}
}
```

Use a Quote token to book a new Payment at the quoted rate. 

### HTTP Request

`GET https://remitsy-demo.herokuapp.com/apis/uni/v1/bank_deposit`

### Query Parameters

Parameter | Type | Description
--------- | ---- | -----------
payment[email] | String(255) | `/\A[\w+\-.]+@[a-z\d\-]+(\.[a-z]+)*\.[a-z]+\z/i`
payment[phone] | String(255) | `^(13[0-9]|14[57]|15[012356789]|17[0678]|18[0-9])[0-9]{8}$`
payment[name_cn] | String(255) | Full name in simplified Chinese Characters UTF-8 encoded
payment[identity_card_number] | String(18) | `^[A-Za-z0-9]{15,17}$` & Checksum validation


### Response

Parameter | Type | Description
--------- | ---- | -----------
quote[payment_ref] | String(8) | For future reference. 


## Release a Payment

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

`GET https://remitsy-demo.herokuapp.com/apis/rate/v1/release`

### URL Parameters

Parameter | Description
--------- | -----------
payment[payment_ref] | Use the release_ref to release the payment to the recipient.


### Response

Parameter | Default | Type | Description
--------- | ------- | ---- | -----------
quote[token] |  | String(8) | Use this token to claim a rate when submitting a new Payment. 
quote[rate] |  | Decimal(4.6) | The rate includes our fee. 
quote[currency_pair] | USDCNY | String(3) | HKDCNY, EURCNY, USDCNY, GBPCNY, PLNCNY.
quote[source_cents] | 0 | Integer(8) | Half even rounding. 
quote[quote_cents] | 0 | Integer(8) | Half even rounding. 
quote[expire_at] | 30min | Unix Timestamp | Quotes will expire if not used to make a booking before the expiry_at attribute. Default value is 30 minutes.


## Cancel a Payment

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

`GET https://remitsy-demo.herokuapp.com/apis/rate/v1/cancel`

### URL Parameters

Parameter | Description
--------- | -----------
payment[payment_ref] | Use the release_ref to release the payment to the recipient.


### Response

`Status 201`

Parameter | Default | Type | Description
--------- | ------- | ---- | -----------
quote[token] |  | String(8) | Use this token to claim a rate when submitting a new Payment. 

