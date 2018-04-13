---
title: Robinson Integration Documentation

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:

includes:
  - order_status
  - errors

search: true
---

# Introduction

Hello Snap and Robinson team.  The goal of this document is for us to come together
on a basic systems architecture so that we can get the ball rolling on an integration
between Snap and Robinson.

This document is to try and help communicate the basic ideas.  Please consider this a
first draft where everything is up for discussion.  The end goal behind this document
is to set some expectations so that both Snap and Robinson can work in parallel without
either one needing a system to be completed, and to minimize the integration overhead
once we're ready to try this out.

The ideal end goal of this process will be a working Minimum Viable Product (MVP).
We don't expect this document to cover everything we'll ever need into the future.  
We expect that this integration and document will change over time.  What we are aiming
for is an integration that is as simple as possible for both sides to build out while
also still meeting our basic integration needs at our current volume.  



# Authentication

> To authorize, use this code:

```shell
# With shell, you can just pass the correct header with each request
curl "api_endpoint_here"
  -H "Authorization: snap_robinson_token"
```

> Make sure to replace `snap_robinson_token` with your API key.

API key generation/management is up for discussion.  For an MVP, Snap would be fine with
a very basic static "secret" API key with the understanding of changing this over time.

Snap's API integration will pass an API key to all requests in a header that looks like this:

`Authorization: snap_robinson_token`

<aside class="notice">
Snap must replace <code>snap_robinson_token</code> with the agreed upon API key.
</aside>

# Webhooks

To try and prevent high volume polling, Robinson's system should allow for basic webhook-based
events.  

Snap will provide a URL that Robinson will send a POST request to.  This POST could either be a
basic event that Snap would parse to determine what type of request must be made; or it could just be
the body of the "Get a Specific Order" request itself for now. The former is a more extensible
option that we would eventually need to move towards, while the latter is likely easier to
implement and saves an extra web request.

Like the API key, this URL could be explicitly provided and not dynamically managed.


# Robinson Orders

## Get All Robinson Orders


```shell
curl "http://example.com/api/orders"
  -H "Authorization: snap_robinson_token"
```


> The above command returns JSON structured like this:

```json
[
  {
    "id": 2,
    "snap_id": 2,
    "status": "shipped",
    "shipped_at": 1234567890,
    "shipping_code": "UPS1234567890",
    "error_code": null,
    "updated_at": 123456790
  }
]
```

This endpoint retrieves all orders.

### HTTP Request

`GET http://example.com/api/orders`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
last_updated_at | false | Requires UNIX Timestamp.  If provided, limits results to orders that have been updated between last_updated_at and current time.
page | 0 | Used for results pagination; all requests start on page 0 by default

### Response Body

The response body will be an array of objects.  Each object will be of the same format
outlined in the "Get a Specific Order" response body section below.


## Get a Specific Order

```shell
curl "http://example.com/api/orders/2"
  -H "Authorization: snap_robinson_token"
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "snap_id": 2,
  "status": "shipped",
  "shipped_at": 1234567890,
  "shipping_code": "UPS1234567890",
  "error_info": null,
  "updated_at": 1234567890
}
```

This endpoint retrieves a specific order.

### HTTP Request

`GET http://example.com/api/orders/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the order to retrieve

### Response Body

In this JSON response, if any of these fields are empty or `null`, they can optionally
simply not be sent along in the response body.

Parameter | Description
--------- | -----------
ID | The ID of the order that was retrieved
Snap ID | The Snap ID that was communicated when the order was created
Status | Order Status Code; see "Order Status Code" section for more info.
Shipped At | UNIX Timestamp associated with the time an order was shipped.
Shipping Code | Parcel Tracking ID for the shipping service used.
Error Info | If Status is "error", this string shows a brief explanation or error code.
Updated At | UNIX Timestamp associated with the time this order was last updated in any way.

## Create a New Order

```shell
curl "http://example.com/api/orders"
  -X POST
  -H "Authorization: snap_robinson_token"
  -d '{
    "snap_id": 2,
    "shipping_address": {
      "name": "Snap Raise",
      "street1": "939 Westlake Ave N",
      "street2": "",
      "city": "Seattle",
      "state": "WA",
      "zip": "98109",
      "country": "USA"
    },
    "line_items": [
      {
        "snap_item_id": 12345,
        "snap_sku": "SNAP12345",
        "size": "M",
        "quantity": 2,
        "logo_url": "http://the.image.url/picture.svg"
      },
      {
        "snap_item_id": 54321,
        "snap_sku": "SNAP54321",
        "size": "M",
        "quantity": 2,
        "logo_url": "http://the.image.url/picture.svg"
      }
    ]
  }'
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "snap_id": 2,
  "status": "new",
  "updated_at": 1234567890
}
```


This endpoint creates a new order, adding it to Robinson's fulfillment process.

### HTTP Request

`POST http://example.com/api/orders`

#### POST Body

Parameter | Req? | Dflt | Description
----------- | ----- | ------- | -----------
Snap ID | | | Snap's internal ID for this order.
Shipping Address | Y | | The shipping address for this order; JSON object
<span style='float:right'>*Name*</span> | Y | | Recipient full name
<span style='float:right'>*Street1*</span> | Y | | Recipient street address line 1
<span style='float:right'>*Street2*</span> | Y | | Recipient street address line 2
<span style='float:right'>*City*</span> | Y | | Recipient city
<span style='float:right'>*State*</span> | Y | | Recipient state ( [USPS Standard 2-letter Abbreviation](https://en.wikipedia.org/wiki/List_of_U.S._state_abbreviations#Postal_codes) )
<span style='float:right'>*Zip*</span> | Y | | Recipient zip code
<span style='float:right'>*Country*</span> | N | 'USA' | Recipient country
Line Items | Y | | Line items in the order; Array of JSON Objects
<span style='float:right'>*Snap Item ID*</span> | Y | | Snap's internal ID for this product
<span style='float:right'>*Snap SKU*</span> | Y | | Snap's SKU for this product
<span style='float:right'>*Size*</span> | Y | | Size of the product ordered
<span style='float:right'>*Quantity*</span> | Y | | Number of products
<span style='float:right'>*Logo URL*</span> | Y | | URL to the logo design file

### Response Body

The response body of this request will be of the same format outlined in the "Get a Specific Order" response body section below.
