# Disclaimer
This is a work in progress. Issues and PRs are welcome.

# Key differences

## Landing page
Universal payment flows are essential for a good user experience. This is why the v2 API has a single, mandatory landing page for all non-mobile payments.
The initiate payment response will contain a unique URL for each order. Either a standard `https://` URL or a deeplink URL, prefixed with `vipps://`.

<img src="images/landing-page.png" width="300">

*Note: On mobile Universal Linking will be used fot `https` URLs and [the Vipps app](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#desktop-browsers-and-mobile-browsers) will open instead.*

## Phone number is optional
The initiate payment call no longer requires a phone number. Instead, the user will be asked to fill in the phone number on the landing page. If phone number is included in the initiate payment body, then the landing page wil be "pre-filled" with that number.

## `isApp: true/false`
If `isApp` is `false` in the initiate payment body, then a `https` URL with a unique token for that specific order will be generated.

If `isApp` is `true` then an appswitch deeplink with a unique token for that specific order will be generated.

### Initiate payment example
See [here](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#initiate-payment-flows) for full overview of initiate payment.

#### Request Body
```
{
   "customerInfo": {
       "mobileNumber": "48059528"
   },
   "merchantInfo": {
       "callbackPrefix": "https://www.callback.com/",
       "fallBack": "https://fallback.com/",
       "merchantSerialNumber": "210462",
       "isApp": true
   },
   "transaction": {
       "amount": 1234,
       "orderId": "id170",
       "timeStamp":"2020-06-24T08:34:25-07:00",
       "requestId": "1234",
       "transactionText": "Initiate Payment"
   }
}
```

#### Response body App Switch (`isApp: true`)
```
{
    "orderId": "id170",
    "url": "vipps://?token=eyJraWQiOiJqd3RrZXkiLCJ[...]"
}
```

#### Response body HTTPS (`isApp: false`)
```
{
    "orderId": "id170",
    "url": "https://api.vipps.no/dwo-api-application/v1/deeplink/vippsgateway?v=2&token=eyJraWQiOiJqd3RrZXkiLCJ[...]"
}
```

## Fallback URL is required

The initiate payment must contain a fallback URL. This is where the user will be redirect to after the payment. This is set in the initiate payment body.

For apps, this URL will be the appswitch-URL.

See [here](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#url-validation) for details.

# Migration

If you have already signed an agreement with Vipps, and have API keys for v1, you can use the same API keys for v2.

## Subscription keys

When you have received confirmation that your new salesunit is created, then you can retrieve the keys in the developer portal. See the [Getting started guide](https://github.com/vippsas/vipps-developers/blob/master/vipps-developer-portal-getting-started.md) for full details.

## New endpoint

The v2 API is available at ```ecomm/v2/payments```:

* Test: ```https://apitest.vipps.no/ecomm/v2/payments```
* Prod: ```https://api.vipps.no/ecomm/v2/payments```

## Documentation

The v2 API documentation is available in markdown on [GitHub](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md)

Technical documentation is available in swagger-format [here](https://vippsas.github.io/vipps-ecom-api/)

We highly recommend going through the whole flow using our Postman [collection](https://raw.githubusercontent.com/vippsas/vipps-ecom-api/master/tools/vipps-ecom-api-postman-collection.json) and [environment](https://raw.githubusercontent.com/vippsas/vipps-ecom-api/master/tools/vipps-ecom-api-postman-enviroment.json) following our [guide](https://github.com/vippsas/vipps-developers/blob/master/postman-guide.md).
