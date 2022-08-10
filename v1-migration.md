# Migration from eCom API v1 to eCom API v2

Document version 1.1.4.

This document explains the key differences between eCom API v1 and eCom API v2.

## Landing page

Universal payment flows are essential for a good user experience. This is why the eCom v2 API has a single, mandatory landing page for all non-mobile payments.
See
[Vipps eCommerce API: How It Works](vipps-ecom-api-howitworks.md)
for a detailed description of the v2 flow.

The initiate payment response will contain a unique URL for each order. Either a standard `https://` URL or a deeplink URL, prefixed with `vipps://`.

<img src="images/landing-page.png" width="300"/>

*Note: On mobile devices "Universal Linking" will be used for `https` URLs, which will automatically open [Vipps](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#desktop-browsers-and-mobile-browsers).*


## Skip landing page

Skipping the landing page is reserved for special cases, where displaying it is not possible.
See the details in the
[skip landing page section](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#skip-landing-page)
in the API guide.

See the [FAQ](https://github.com/vippsas/vipps-psp-api/blob/master/vipps-psp-api-faq.md#is-it-possible-to-skip-the-landing-page).


## Phone number is optional

The initiate payment call no longer requires a phone number. Instead, the user will be asked to fill in the phone number on the landing page. If phone number is included in the initiate payment body, then the landing page will be "pre-filled" with that number.

## `isApp: true/false`

If `isApp` is `false` in the initiate payment body, then a `https` URL with a unique token for that specific order will be generated.

If `isApp` is `true` then an appswitch deeplink URL with a unique token for that specific order will be generated.

### Initiate payment example

See [here](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#initiate-payment-flows) for full overview of initiate payment.

### Request Body

```
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

```
{
    "orderId": "id170",
    "url": "vipps://?token=eyJraWQiOiJqd3RrZXkiLCJ[...]"
}
```

### Response body HTTPS (`isApp: false`)

```
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
See [here](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#url-validation) for details.

## Migration

If you have already signed an agreement with Vipps, and have API keys for eCom v1, you can use the same API keys
for eCom v2.

## Subscription keys

When you have received confirmation that your new sales unit is created, then you can retrieve the API keys on https://portal.vipps.no. See [Getting started](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md).

## New endpoint

The eCom v2 API is available at ```ecomm/v2/payments```:

* Test: ```https://apitest.vipps.no/ecomm/v2/payments```
* Prod: ```https://api.vipps.no/ecomm/v2/payments```

## Documentation

The eCom v2 API documentation is available here: https://github.com/vippsas/vipps-ecom-api

# Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-ecom-api/issues),
a [pull request](https://github.com/vippsas/vipps-ecom-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).

Sign up for our [Technical newsletter for developers](https://github.com/vippsas/vipps-developers/tree/master/newsletters).
