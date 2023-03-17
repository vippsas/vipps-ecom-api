<!-- START_METADATA
---
draft: true
---
END_METADATA -->

# Migration from eCom API v1 to eCom API v2

This document explains the key differences between eCom API v1 and eCom API v2.

## Landing page

Universal payment flows are essential for a good user experience. This is why the eCom v2 API has a single, mandatory landing page for all non-mobile payments.
See
[Vipps eCommerce API: How It Works](./how-it-works/vipps-ecom-api-howitworks.md)
for a detailed description of the v2 flow.

The initiate payment response will contain a unique URL for each order. Either a standard `https://` URL or a deeplink URL, prefixed with `vipps://`.

![Vipps landing page](images/landing-page.png)

*Note: On mobile devices "Universal Linking" will be used for `https` URLs, which will automatically open [Vipps](vipps-ecom-api.md#phone-and-mobile-browser-flow).*

## Skip landing page

Skipping the landing page is reserved for special cases, where displaying it is not possible.
See the details in the
[skip landing page section](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/common-topics/vipps-landing-page#skip-landing-page).

See the [FAQ](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/faqs/vipps-landing-page-faq#is-it-possible-to-skip-the-landing-page).

## Phone number is optional

The initiate payment call no longer requires a phone number. Instead, the user will be asked to fill in the phone number on the landing page. If phone number is included in the initiate payment body, then the landing page will be "pre-filled" with that number.

## `isApp: true/false`

If `isApp` is `false` in the initiate payment body, then a `https` URL with a unique token for that specific order will be generated.

If `isApp` is `true` then an appswitch deeplink URL with a unique token for that specific order will be generated.

### Initiate payment example

See [here](vipps-ecom-api.md#initiate) for full overview of initiate payment.

### Request Body

```json
{
  "customerInfo": {
      "mobileNumber": "48059528"
  },
  "merchantInfo": {
    "merchantSerialNumber": "123456",
    "callbackPrefix":"https://example.com/vipps/callbacks-for-payment-update",
    "fallBack": "https://example.com/vipps/fallback-result-page/order123abc"
  },
  "transaction": {
    "orderId": "order123abc",
    "amount": 20000,
    "transactionText": "One pair of Vipps socks"
  }
}
```

### Response body App Switch (`isApp: true`)

```json
{
    "orderId": "id170",
    "url": "vipps://?token=eyJraWQiOiJqd3RrZXkiLCJ[...]"
}
```

### Response body HTTPS (`isApp: false`)

```json
{
    "orderId": "id170",
    "url": "https://api.vipps.no/dwo-api-application/v1/deeplink/vippsgateway?v=2&token=eyJraWQiOiJqd3RrZXkiLCJ[...]"
}
```

## Fallback URL is required

The initiate payment must contain a `fallBack` URL. This is where the user will be redirect to after the payment.
This is set in the initiate payment body.

For apps, this URL will be the appswitch URL.

If you do not have a fallback (or "result" page), you can use the main URL for your company:
`https://example.com/fallback-dummy`, or similar. Please use a URL that returns `HTTP 200 OK`.

The URL must be valid.
See [here](vipps-ecom-api.md#url-validation) for details.

## Migration

If you have already signed an agreement with Vipps, and have API keys for eCom v1, you can use the same API keys
for eCom v2.

## Subscription keys

When you have received confirmation that your new sales unit is created, then you can retrieve the API keys on <https://portal.vipps.no>. See [Getting started](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/getting-started).

## New endpoint

The eCom v2 API is available at ```ecomm/v2/payments```:

* Test: ```https://apitest.vipps.no/ecomm/v2/payments```
* Prod: ```https://api.vipps.no/ecomm/v2/payments```

## Documentation

The eCom v2 API documentation is available here: <https://github.com/vippsas/vipps-ecom-api>
