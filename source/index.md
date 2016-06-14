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

Easily integrate mass payouts of funds, with your existing business flows.

# Quick Start

Book a Payment: 

1. Request an API Key
2. Get a Quote
3. Submit a Payment
4. Release a Payment
5. Check status of Payment

# Authentication

> To authorize, use this code:

```shell
# With shell, you can just pass the correct header with each request
curl -X GET \
-H 'Authorization: Token token=<API KEY>' \
-H "Content-Type: text/plain; charset=UTF-8" \
http://sandbox.remitsy.com/apis/general/v1/ping
```

HTTP Token authentication, passed in the header. 

Keys are available upon request via, support@remitsy.com.

Requests without a valid API key will return:

<code>
    Code: 401 UNAUTHORIZED

    Content: { error : "unauthorized access"  }
</code>

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
https://sandbox.remitsy.com/apis/general/v1/rates
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
    "Must be an integer",
    "Your account has insufficient funds to cover this Quote. Please contact support@remitsy.com"
  ]}
} 
```


Provides a Quote for a given proposed Payment. Each Quote has a unique token, 
which is used to claim the quoted exchange rate in a subsequent Payment booking.

#### 

* To obtain a Quote for a given sending amount, provide both a source_currency and source_cents. For example:

`{ source_currency: "USD", source_cents: 10000 } > { quote_cents: 65000  }`

* To obtain a Quote for a given receiving amount, in CNY, provide a source_currency and a quote_cents, setting the source_cents to null. For example:

`{ source_currency: "USD", source_cents: null, quote_cents: 65000 } > { source_cents: 10000  }`

<aside class="warning">
  Please note that if neither <strong>source_cents</strong> or <strong>quote_cents</strong> are set to null, then the <strong>source_cents</strong> takes priority.
</aside>


### HTTP Request

`POST https://sandbox.remitsy.com/apis/general/v1/rates`

### Request Parameters

Parameter | Default | Type | Description
--------- | ------- | ---- | -----------
quote[source_currency] | USD | String(3) | Currently supports HKD, EUR, USD, GBP, PLN
quote[source_cents] | 0 | Integer(8) | If set to null, then the quote_cents attribute must be specified.
quote[quote_cents] | null | Integer(8) | If set to null, then the source_cents attribute must be specified. 

<aside class="notice">
  The <strong>quote_currency</strong> is always CNY. 
</aside>

### Response

`Status 201 CREATED`

Parameter | Default | Type | Description
--------- | ------- | ---- | -----------
quote[token] |  | String(10) | Use this token to claim a rate when submitting a new Payment. 
quote[rate] |  | Decimal(4.6) | The rate includes our fee. 
quote[currency_pair] | USDCNY | String(3) | The format is [SOURCE][QUOTE]. Currently accepts HKDCNY, EURCNY, USDCNY, GBPCNY, PLNCNY.
quote[source_cents] | 0 | Integer(8) | Half even rounding. 
quote[quote_cents] | 0 | Integer(8) | Half even rounding. 
quote[expire_at] | 30min | Unix Timestamp | Quotes will expire if not used to make a booking before the expiry_at attribute. Default value is 30 minutes.

<aside class="warning">
  Each quote has an expiry date. A payment claiming this quote must be booked before the quote expires.  
</aside>

<aside class="notice">
  All currency amounts are denominated in cents. Rounding is done using Ruby 2.0.0, 
  <a href="http://ruby-doc.org/stdlib-2.0.0/libdoc/bigdecimal/rdoc/BigDecimal.html">BigDecimal::ROUND_HALF_EVEN</a>.
</aside>

`Status 422 UNPROCESSABLE ENTITY`

Parameter | Description
--------- | -----------
error[source_currency] | Must be one of HKD EUR USD PLN GBP. 
error[source_cents] | Must be an integer. 

`Status 402 PAYMENT REQUIRED`

Parameter | Description
--------- | -----------
error[source_cents] | Your account has insufficient funds to cover this Quote. Please contact support@remitsy.com. 


## Book an AliPay Payment

> For AliPay Payment

```shell
curl -X POST \
-H 'Authorization: Token token="<API KEY>"' \
-H "Content-Type: text/plain; charset=UTF-8" \
-d "token=xxxxxxxxxxx" \
-d "alipay_id=1235812895" \
-d "email=richard@163.com" \
-d "phone=13810456155" \
-d "name_cn=贝礼德" \
-d "immediate_release=false" \
-d "identity_card_number=110101198109022323" \
https://sandbox.remitsy.com/apis/general/v1/new
```

> The above commands returns JSON structured like this:

```json
{ "payment":{ "payment_ref": "xxxxxxxxxxxx" }}
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
  "alipay_id": [
    "can't be blank",
    "Invalid AliPay ID"              
  ]}
}
```

> Validates the 18 character Chinese National Identity Card Number

```ruby
# https://en.wikipedia.org/wiki/Resident_Identity_Card#Identity_card_number
def validate_18_character_identity_card_number( icn18 )
  checksum_ordered_set = [7,9,10,5,8,4,2,1,6,3,7,9,10,5,8,4,2]
  checksum_index_set = [1,0,'X',9,8,7,6,5,4,3,2]

  i = 0
  sum = 0
  while i < checksum_ordered_set.length do
    sum += icn18[i].to_i * checksum_ordered_set[i]
    i += 1
  end

  checksum_index_set[sum%11].to_s == icn18[-1]
end
```

Use a Quote token to book a new AliPay Payment at the quoted rate. 


<aside class="notice">
  AliPay is broadly similar to PayPal. Alipay had the biggest market share in 
  China with 300 million users and control of just under half of China's 
  online payment market in February 2014. 
</aside>

### HTTP Request

`POST https://sandbox.remitsy.com/apis/general/v1/alipay`

### Request Parameters

Parameter | Type | Default | Description
--------- | ---- | ------- | -----------
payment[alipay_id] | String(255) | | Like a PayPal ID, can be a email, phone number or serial number.
payment[email] | String(255) | | The Recipient's email, used for notifications.
payment[phone] | String(255) | | The Recipient's phone number, used for KYC and notifications. 
payment[name_cn] | String(255) | | Full name in simplified Chinese Characters UTF-8 encoded.
payment[identity_card_number] | String(18) | | National ID card, used for KYC.
payment[token] | String(10) | | Provide a Quote token to claim the rate and amount.
payment[immediate_release] | Boolean(true&#124;false) | true | **Optional**, set to false to hold payment until you release the payment. This allows additional KYC checks to be performed inline with your existing business flows.

### Response

`Status 201 CREATED`

Parameter | Type | Description
--------- | ---- | -----------
payment[payment_ref] | String(8) | For future reference. 

`Status 422 UNPROCESSABLE ENTITY`

Parameter | Description
--------- | -----------
error[phone] | `^(13[0-9]&#124;14[57]&#124;15[012356789]&#124;17[0678]&#124;18[0-9])[0-9]{8}$` 
error[email] | `/\A[\w+\-.]+@[a-z\d\-]+(\.[a-z]+)*\.[a-z]+\z/i`
error[name_cn] | Must be Chinese Characters, UTF-8 encoded.
error[identity_card_number] | Must pass the checksum
error[immediate_release] | Must be either true or false

`Status 404 NOT FOUND`

Parameter | Description
--------- | -----------
error[token] | Invalid token, no Quote found.

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
https://sandbox.remitsy.com/apis/general/v1/new
```

> The above commands returns JSON structured like this:

```json
{ "payment": { "payment_ref": "xxxxxxxxxx" }}
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

> Validates the 18 character Chinese National Identity Card Number

```ruby
# https://en.wikipedia.org/wiki/Resident_Identity_Card#Identity_card_number
def validate_18_character_identity_card_number( icn18 )
  checksum_ordered_set = [7,9,10,5,8,4,2,1,6,3,7,9,10,5,8,4,2]
  checksum_index_set = [1,0,'X',9,8,7,6,5,4,3,2]

  i = 0
  sum = 0
  while i < checksum_ordered_set.length do
    sum += icn18[i].to_i * checksum_ordered_set[i]
    i += 1
  end

  checksum_index_set[sum%11].to_s == icn18[-1]
end
```

Use a Quote token to book a new Payment at the quoted rate. 

### HTTP Request

`POST https://sandbox.remitsy.com/apis/general/v1/bank_deposit`

### Request Parameters

Parameter | Type | Default | Description
--------- | ---- | ------- | -----------
payment[email] | String(255) | | The Recipient's email, used for notifications.
payment[phone] | String(255) | | The Recipient's phone number, used for KYC and notifications. 
payment[name_cn] | String(255) | | Full name in simplified Chinese Characters UTF-8 encoded.
payment[identity_card_number] | String(18) | | National ID card, used for KYC.
payment[token] | String(10) | | Provide a Quote token to claim the rate and amount.
payment[bank_account_number] | String(19) | | Chinese bank account number 15-19 numerals long.
payment[bank_account_name] | String(255) | | Full name in simplified Chinese Characters UTF-8 encoded
payment[bank_account_city] | String(255) | | Name in simplified Chinese Characters UTF-8 encoded
payment[bank_account_branch] | String(255) | | Name in simplified Chinese Characters UTF-8 encoded
payment[immediate_release] | Boolean(true&#124;false) | true | **Optional**, set to false to hold payment until you release the payment. This allows additional KYC checks to be performed inline with your existing business flows.

### Response

`Status 201 CREATED`

Parameter | Type | Description
--------- | ---- | -----------
payment[payment_ref] | String(8) | For future reference. 

`Status 422 UNPROCESSABLE ENTITY`

Parameter | Description
--------- | -----------
error[bank_account_number] | `^[0-9]{15,19}$`
error[bank_account_name] | Full name in simplified Chinese Characters UTF-8 encoded
error[bank_account_city] | Name in simplified Chinese Characters UTF-8 encoded
error[bank_account_branch] | Name in simplified Chinese Characters UTF-8 encoded
error[phone] | `^(13[0-9]&#124;14[57]&#124;15[012356789]&#124;17[0678]&#124;18[0-9])[0-9]{8}$` 
error[email] | `/\A[\w+\-.]+@[a-z\d\-]+(\.[a-z]+)*\.[a-z]+\z/i`
error[name_cn] | Must be Chinese Characters, UTF-8 encoded.
error[identity_card_number] | Failed Checksum
error[immediate_release] | Must be either true or false

`Status 404 NOT FOUND`

Parameter | Description
--------- | -----------
error[token] | Invalid token, no Quote found.



## Release a Payment

```shell
curl -X POST \
-H 'Authorization: Token token="<API KEY>"' \
-H "Content-Type: text/plain; charset=UTF-8" \
-d "payment[payment_ref]=12345678" \
https://sandbox.remitsy.com/apis/general/v1/release
```

> The above command returns JSON structured like this:

```json
{ "release_ref": "12345678" }
```

> Errors structured like this:

```json
{"errors": {
  "payment_ref": [
    "Payment has already been cancelled or released", 
    "Invalid Payment_ref. No Payment found."
  ]}
}
```

Releases a payment previously booked with the immediate_release attribute set to false.

### HTTP Request

`PATCH https://sandbox.remitsy.com/apis/general/v1/release`

### Request Parameters

Parameter | Description
--------- | -----------
payment[payment_ref] | Use the release_ref to release the payment to the recipient.


### Response

`Status 200 OK`

Parameter | Description
--------- | -----------
payment[release_ref] | For your records

`Status 422 UNPROCESSABLE ENTITY`

Parameter | Description
--------- | -----------
error[payment_ref] | The Payment has already been cancelled or released.

`Status 404 NOT FOUND`

Parameter | Description
--------- | -----------
error[payment_ref] | Invalid payment_ref, no Payment found.

## Cancel a Payment


```shell
curl -X POST \
-H 'Authorization: Token token="<API KEY>"' \
-H "Content-Type: text/plain; charset=UTF-8" \
-d "payment[payment_ref]=12345678" \
https://sandbox.remitsy.com/apis/general/v1/cancel
```

> The above command returns JSON structured like this:

```json
{ "cancel_ref": "123456" }
```

> Errors structured like this:

```json
{"errors": {
  "payment_ref": [
    "Payment has already been cancelled or released", 
    "Invalid Payment_ref. No Payment found."
  ]}
}
```

This endpoint cancels an unreleased payment.

### HTTP Request

`PATCH https://sandbox.remitsy.com/apis/general/v1/cancel`

### Request Parameters

Parameter | Description
--------- | -----------
payment[payment_ref] | Use the release_ref to release the payment to the recipient.
payment[note] | **Optional** Useful to store internal references, for use in follow up correspondence. 


### Response

`Status 201`

Parameter | Type | Description
--------- | ------- | -----------
payment[cancel_ref] | String(6) | Please for your future reference

`Status 422 UNPROCESSABLE ENTITY`

Parameter | Description
--------- | -----------
error[payment_ref] | The Payment has already been released at (yyyy:mm:dd hh:mm)

`Status 404 NOT FOUND`

Parameter | Description
--------- | -----------
error[payment_ref] | Invalid payment_ref, no Payment found.



## Check Status of a Payment


```shell
curl -X POST \
-H 'Authorization: Token token="<API KEY>"' \
-H "Content-Type: text/plain; charset=UTF-8" \
-d "payment[payment_ref]=12345678" \
https://sandbox.remitsy.com/apis/general/v1/status
```

> The above command returns JSON structured like this:

```json
{
  "payment": {
    "email": "richard@163.com",
    "phone": "13810456155",
    "bank_account_number": "6212260200082726700",
    "bank_account_name": "中国银行",
    "bank_account_city": "北京",
    "bank_account_branch": "朝阳门支行",
    "immediate_release": false, 
    "release_ref": "12345678",
    "cancel_ref": "123456",
    "payment_ref": "1234567890",
    "alipay_id": "",
    "status": "Released",
    "note": "Free text",
    "created_at": "2016-06-14 17:18:38 +0100"
  },
  "quote": {
    "rate": 6.51,
    "token": "1234567890",
    "source_currency": "USD",
    "source_cents": 10000,
    "quote_cents": 65000,
    "expire_at": 1259193600,
    "created_at": "2016-06-14 17:18:38 +0100"
  }
}
```

> Errors structured like this:

```json
{"errors": {
  "payment_ref": [
    "Invalid Payment_ref. No Payment found."
  ]}
}
```

This endpoint cancels an unreleased payment.

### HTTP Request

`GET https://sandbox.remitsy.com/apis/general/v1/status`

### Request Parameters

Parameter | Type | Description
--------- | ---- | -----------
payment[payment_ref] | String(6) | A unique reference to a Payment

### Response

`Status 200 OK`

Parameter | Type | Default | Description
--------- | ---- | ------- | -----------
payment[email] | String(255) | | The recipient's email. Used for notification.
payment[phone] | String(255) | | The recipient's phone number. Used for KYC and SMS notification
payment[bank_account_number] | String(19) | | Recipient's Chinese bank account number
payment[bank_account_name] | String(255) | | Recipient's Chinese account name
payment[bank_account_city] | String(255) | | Name in simplified Chinese Characters UTF-8 encoded
payment[bank_account_branch] | String(255) | | Name in simplified Chinese Characters UTF-8 encoded
payment[immediate_release] | Boolean(true&#124;false) | true | **Optional**, set to false to hold payment until you release the payment. This allows additional KYC checks to be performed inline with your existing business flows.
payment[release_ref] | String(8)| | Reference to the action of releasing this a payment. If cancelled or booked this will be empty
payment[cancel_ref] | String(6) | | Reference to the action of cancelling this a payment. If booked or released this will be empty
payment[alipay_id] | String(255) | | Like a PayPal ID, can be a email, phone number or serial number.
payment[status] | String(255) | | Can be Booked, Cancelled or Released
payment[note] | String(255) | | Notes
payment[created_at] | timestamp | | Payment created at.
quote[rate] | Decimal(4.6) | | The rate includes our fee.
quote[token] | String(10) | | Unique identifier for Quote
quote[source_currency] | String(3) | | Currently accepts HKD, EUR, USD, GBP, PLN.
quote[source_cents]	| Integer(8) | | Half even rounding.
quote[quote_cents] | Integer(8) | null | Half even rounding.
quote[expire_at] | Unix Timestamp | 30min | Quotes will expire if not used to make a booking within the time limit.
quote[created_at] | Timestamp | | Quote Created At

<aside class="notice">
  Please note; out of an abundance of caution, the status does not show the 
  <strong>name_cn</strong> and <strong>identity_card_number</strong>. These 
  attributes are potentially sensitive information. Please use your own 
  records to access these details if they are required.
</aside>

`Status 404 NOT FOUND`

Parameter | Description
--------- | -----------
error[payment_ref] | Invalid payment_ref, no Payment found.

