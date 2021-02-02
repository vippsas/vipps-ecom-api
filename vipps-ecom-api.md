# Vipps eCommerce API

API version: 2.0

See: Vipps eCom API [GitHub repository](https://github.com/vippsas/vipps-ecom-api),
with
[Swagger specifications](https://vippsas.github.io/vipps-ecom-api/),
[Postman collections](tools/),
[example code](https://github.com/vippsas/vipps-developers/tree/master/code-examples/ecom_python_example),
[integration checklist](vipps-ecom-api-checklist.md)
and the [FAQ](vipps-ecom-api-faq.md).

See also: [How it works](vipps-ecom-api-howitworks.md).

Document version 2.5.2.

## Table of contents

- [Vipps eCommerce API](#vipps-ecommerce-api)
  - [Table of contents](#table-of-contents)
  - [Flow diagram](#flow-diagram)
  - [Call by call guide](#call-by-call-guide)
  - [API endpoints](#api-endpoints)
  - [Authentication](#authentication)
  - [Optional Vipps HTTP headers](#optional-vipps-http-headers)
  - [Initiate](#initiate)
  - [Regular eCommerce payments](#regular-ecommerce-payments)
    - [Reserve capture](#reserve-capture)
    - [Direct capture](#direct-capture)
    - [When to use reserve capture and direct capture](#when-to-use-reserve-capture-and-direct-capture)
  - [Express checkout payments](#express-checkout-payments)
    - [Old and new express checkout flow](#old-and-new-express-checkout-flow)
      - [How to specify the old or new express checkout flow](#how-to-specify-the-old-or-new-express-checkout-flow)
    - [Shipping and static shipping details](#shipping-and-static-shipping-details)
    - [Consent and GDPR](#consent-and-gdpr)
  - [Initiate payment flow: Phone and browser](#initiate-payment-flow-phone-and-browser)
    - [Phone flow](#phone-flow)
      - [Vipps installed](#vipps-installed)
      - [Vipps not installed](#vipps-not-installed)
  - [Desktop flow](#desktop-flow)
    - [Desktop browser initiated payments](#desktop-browser-initiated-payments)
    - [Payments initiated in an app](#payments-initiated-in-an-app)
    - [Initiate payment flow: API calls](#initiate-payment-flow-api-calls)
      - [The Vipps deeplink URL](#the-vipps-deeplink-url)
  - [Payment identification](#payment-identification)
  - [Payment retries](#payment-retries)
  - [orderId recommendations](#orderid-recommendations)
  - [URL Validation](#url-validation)
  - [Callbacks](#callbacks)
    - [Callback endpoints](#callback-endpoints)
    - [Callback examples](#callback-examples)
    - [Authorization for callbacks](#authorization-for-callbacks)
    - [Vipps callback servers](#vipps-callback-servers)
    - [Callback URLs must be reachable](#callback-urls-must-be-reachable)
    - [Callback statuses](#callback-statuses)
  - [Timeouts](#timeouts)
    - [Using a phone](#using-a-phone)
    - [Using a laptop/desktop](#using-a-laptopdesktop)
  - [Express checkout API endpoints required on the merchant side](#express-checkout-api-endpoints-required-on-the-merchant-side)
    - [Get shipping details](#get-shipping-details)
    - [Transaction update](#transaction-update)
    - [Remove User Consent](#remove-user-consent)
  - [Skip landing page](#skip-landing-page)
  - [Reserve](#reserve)
  - [The Vipps landing page](#the-vipps-landing-page)
  - [Capture](#capture)
    - [Reserve capture](#reserve-capture-1)
    - [Direct capture](#direct-capture-1)
    - [Partial capture](#partial-capture)
  - [Cancel](#cancel)
    - [Cancelling a pending order](#cancelling-a-pending-order)
  - [Refund](#refund)
  - [Recurring eCommerce payments](#recurring-ecommerce-payments)
  - [Get payment details](#get-payment-details)
    - [Payment states](#payment-states)
  - [Requests and responses](#requests-and-responses)
    - [Example response](#example-response)
  - [Polling guidelines](#polling-guidelines)
  - [Get payment status](#get-payment-status)
  - [Userinfo](#userinfo)
    - [Scopes](#scopes)
    - [Userinfo call by call guide](#userinfo-call-by-call-guide)
    - [Get userinfo](#get-userinfo)
    - [Vipps Login access token](#vipps-login-access-token)
    - [Userinfo call](#userinfo-call)
    - [Consent](#consent)
  - [HTTP response codes](#http-response-codes)
  - [Rate limiting](#rate-limiting)
  - [Partner Keys](#partner-keys)
  - [Idempotency](#idempotency)
  - [Exception handling](#exception-handling)
    - [Connection timeout](#connection-timeout)
    - [Callback aborted or interrupted](#callback-aborted-or-interrupted)
    - [PSP connection issues](#psp-connection-issues)
  - [App integration](#app-integration)
  - [App-switching](#app-switching)
    - [App-switch on iOS](#app-switch-on-ios)
      - [Switch from merchant app to Vipps](#switch-from-merchant-app-to-vipps)
      - [Redirect back to the merchant app from Vipps app](#redirect-back-to-the-merchant-app-from-vipps-app)
    - [App-switch on Android](#app-switch-on-android)
      - [Switching from merchant app to Vipps](#switching-from-merchant-app-to-vipps)
      - [Switching back to the merchant app from Vipps app](#switching-back-to-the-merchant-app-from-vipps-app)
      - [Return back to merchant app by actively deeplinking into it from Vipps](#return-back-to-merchant-app-by-actively-deeplinking-into-it-from-vipps)
      - [Redirect back to merchant app by simply closing Vipps](#redirect-back-to-merchant-app-by-simply-closing-vipps)
  - [Errors](#errors)
    - [Error object in the response](#error-object-in-the-response)
  - [Error groups](#error-groups)
  - [Error codes](#error-codes)
- [Testing](#testing)
- [Recomendations regarding handling redirects](#recomendations-regarding-handling-redirects)
- [Questions?](#questions)

## Flow diagram

This diagram shows a simplified payment flow:

![Vipps checkout flow chart](images/flow-diagram.png)

See [Get payment details](#get-payment-details) for more details about
the detailed flow, and [Payment states](#payment-states) for the corresponding
states.

## Call by call guide

The normal "happy day" flow for a payment is:

1. Initiate the payment:
   [`POST:/ecomm/v2/payments](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/initiatePaymentV3UsingPOST).
   The user can now confirm the payment in Vipps, setting the payment status to reserved.
   See [Initiate](#initiate).
2. Receive the callback with the payment status:
   [`POST:[callbackPrefix]/v2/payments/{orderId}`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/transactionUpdateCallbackForRegularPaymentUsingPOST).
   See [Callbacks](#callbacks).
3. Get the payment details:
   [GET:/ecomm/v2/payments/{orderId}/details](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getPaymentDetailsUsingGET).
   This is optional, but _must_ be done if no callback has been received.
   See
   [Get payment details](#get-payment-details)
   and
   [Polling guidelines](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#polling-guidelines).
4. Capture the payment:
   [`POST:/ecomm/v2/payments/{orderId}/capture`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/capturePaymentUsingPOST).
   See [Regular eCommerce payments](#regular-ecommerce-payments).

There is, of course, much more to this. The user may cancel, or not act on, the
payment, firewalls may block the callback, Vipps Hurtigkasse
(express checkout) address and shipping lookups may fail, etc.

We have done our best to document everything about this API, and you _should_
have all information needed to integrate with Vipps.

## API endpoints

The Vipps eCommerce API (eCom API) offers functionality for online payments.
Payments are supported in both web browsers and in native apps (via deep-linking).

| Operation           | Description         | Endpoint          |
| ------------------- | ------------------- | ----------------- |
| [Initiate payment](#initiate) | Payment initiation, the first request in the payment flow. This _reserves_ an amount. | [`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/initiatePaymentV3UsingPOST)  |
| [Capture payment](#capture) | When an amount has been reserved, and the goods are (about to be) shipped, the payment must be _captured_ | [`POST:/ecomm/v2/payments/{orderId}/capture`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/capturePaymentUsingPOST)  |
| [Cancel payment](#cancel) | The merchant may cancel a reserved amount, but not on a captured amount. | [`PUT:/ecomm/v2/payments/{orderId}/cancel`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/cancelPaymentRequestUsingPUT)  |
| [Refund payment](#refund) | The merchant may refund a captured amount. | [`POST:/ecomm/v2/payments/{orderId}/refund`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/refundPaymentUsingPOST)  |
| [Get payment details](#get-payment-details) | The full history of the payment.  | [`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getPaymentDetailsUsingGET)  |
| [Access Token](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md#get-an-access-token) | Fetch the access token | [`POST:/accesstoken/get`](https://vippsas.github.io/vipps-ecom-api/#/Authorization_Service/fetchAuthorizationTokenUsingPost) |
| Get order status    | Deprecated, use [Get payment details](#get-payment-details) | [`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getPaymentDetailsUsingGET) |

See the
[eCom API checklist](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api-checklist.md).

## Authentication

All Vipps API calls are authenticated and authorized with an access token
(JWT bearer token) and an API subscription key:

| Header Name | Header Value | Description |
| ----------- | ------------ | ----------- |
| `Authorization` | `Bearer <JWT access token>` | Type: Authorization token. This is available on [portal.vipps.no](https://portal.vipps.no). |
| `Ocp-Apim-Subscription-Key` | Base 64 encoded string | The subscription key for this API. This is available on [portal.vipps.no](https://portal.vipps.no). |

For more information about how to obtain an access token and all details around this, please see:
[Quick overview of how to make an API call](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md#quick-overview-of-how-to-make-an-api-call)
in the
[Getting started guide](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md).

## Optional Vipps HTTP headers

We recommend using the following _optional_ HTTP headers for all requests to the
Vipps eCom API. These headers provide useful metadata about the merchant's system,
which help Vipps improve our services, and also help in investigating problems.
Please use self-explanatory values that uniquely identify the system (and plugin).   

| Header                        | Description                                  | Example value        |
| ----------------------------- | -------------------------------------------- | -------------------- |
| `Merchant-Serial-Number`      | The merchant serial number                   | `123456`             |
| `Vipps-System-Name`           | The name of the ecommerce solution           | `woocommerce`        |
| `Vipps-System-Version`        | The version number of the ecommerce solution | `5.4`                |
| `Vipps-System-Plugin-Name`    | The name of the ecommerce plugin             | `vipps-woocommerce`  |
| `Vipps-System-Plugin-Version` | The version number of the ecommerce plugin   | `1.4.1`              |

These headers are required for plugins and partners and sent by the recent versions of
[the official Vipps plugins](https://github.com/vippsas/vipps-developers#plugins)
and we recommend all customers with direct integration with the API to also do so.

## Initiate

Vipps eCommerce API offers two types of payments:
1. Regular eCommerce payments
2. Express checkout payments

Examples from a demo website:

![Regular and express checkout](images/vipps-flow-web.png)

## Regular eCommerce payments

Payment amounts must be larger than zero.

When you initiate a payment it will only be _reserved_ until you capture it.
Vipps supports both _reserve capture_ and _direct capture_ payment flows.

### Reserve capture

**Reserve capture** is the default. When you initiate a payment it will be
reserved until you capture it.

According to Norwegian regulations you can _not_ capture a payment until
the product or service is provided to the customer. For more information,
please see the Consumer Authority's
[Guidelines for the standard sales conditions for consumer purchases of goods over the internet](https://www.forbrukertilsynet.no/english/guidelines/guidelines-the-standard-sales-conditions-consumer-purchases-of-goods-the-internet).

See [Reserve capture](#reserve-capture-1) for more details.

### Direct capture

**Direct capture** causes all payment reservations to be instantly be captured.
This is intended for situations where the product or service is immediately
provided to the customer, e.g. digital services.

See [Direct capture](#direct-capture-1) for more details.

### When to use reserve capture and direct capture

Merchants can not choose between _reserve capture_ and _direct capture_
themselves, the type of capture is configured by Vipps after the additional
compliance checks, required by the authorities, have been completed.

See the
[FAQ](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api-faq.md#what-is-the-difference-between-reserve-capture-and-direct-capture).

## Express checkout payments

The Express checkout (Vipps Hurtigkasse) is a solution for letting the user
automatically share the Vipps profile address information with merchant and
choose a shipping option:

1. The user clicks the "Vipps Hurtigkasse" button.
2. The user consents to sharing address information in Vipps.
3. The user confirms the amount, delivery address and delivery method in Vipps.

To perform an express checkout, the merchant needs to send
`"paymentType": "eComm Express Payment"` as part of initiate payment, and
support the `shippingDetails` and `consent` endpoints.

### Old and new express checkout flow

This does require some background info - please bear with us.

In the old flow for the express checkout it was easy for a user to overlook
that the correct address and shipping method was used, as both were simply
displayed on the payment confirmation page. The user did not have to actively
choose an address and shipping method. This resulted in some users
inadvertently confirming the wrong address and possibly also selecting the
wrong shipping method. To fix this, we made it mandatory to select and address
and shipping method. This was released as a minor update of Vipps.

That worked well for all merchants that used Vipps Hurtigkasse _as intended_,
but not for those using it as a quick hack to speed up payments, "just get some
info about the user", Covid-19 tracking, or something else.

The new and (for some) improved flow did not work well for all. One example:
Restaurant  guests could not understand why a restaurant needed their address
and shipping method for the two beers they ordered from their table.
They had not noticed the need to select those before, but definitely did now.

So: We changed it back to the way it was before, and now require merchants to
explicitly specify the new express checkout flow.

#### How to specify the old or new express checkout flow

The old (and for some: "normal") express checkout flow is the default.
You do not have to make any changes other than to specify
`"paymentType": "eComm Express Payment"`.

To get the new express checkout flow: Specify this in addition, in the
`transaction` object:

`"useExplicitCheckoutFlow": true`

See
[`POST:/ecomm/v2/payments](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/initiatePaymentV3UsingPOST)
for more details.

### Shipping and static shipping details

The shipping methods presented to the user is in Vipps are retrieved from the merchant's
[`POST:[shippingDetailsPrefix]​/v2​/payments​/{orderId}​/shippingDetails`](https://vippsas.github.io/vipps-ecom-api/#/Merchant%20Endpoints/fetchShippingCostUsingPOST)
endpoint, which Vipps uses to
retrieve the shipping details from the merchant.

If the shipping cost can be known in advance, the `staticShippingDetails` field in the
[`POST:​/ecomm​/v2​/payments`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/initiatePaymentV3UsingPOST)
call to initiate the payment may be used to provide the shipping details
up front, and thereby avoid an extra roundtrip between the Vipps backend and
the merchant's server.

When using `staticShippingDetails` the shipping costs for the available
shipping methods are then sent directly, eliminating the need for the user to
first select shipping method and then for the merchant to calculate the cost for it.

We recommend using `staticShippingDetails` if possible, as it gives a faster
payment process and a better user experience.

Use of `staticShippingDetails` also eliminates timeout problems caused by
delays in the merchant's or shipping partner's calculations of cost.

### Consent and GDPR

Vipps complies with GDPR, and requires the user's consent before any information
is shared with the merchant. The merchant must provide a URL (`consentRemovalPrefix`)
that Vipps can call to delete the data. Vipps allows the user to later
remove this consent (via the Profile -> Security -> "Access to your information"
-> "Companies that remember you" screens).

## Initiate payment flow: Phone and browser

A payment is initiated with a call to
[`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/initiatePaymentV3UsingPOST).

### Phone flow

![Push notification](images/vipps-flow-device.png)

Triggered by the payment initiation, the Vipps landing page will automatically
detect if is being invoked on a phone, and whether Vipps is installed on the phone.
If Vipps is installed, Vipps will automatically be opened.

#### Vipps installed

1. Vipps is invoked (with app-switch).
2. The user accepts or rejects the payment request in Vipps.
3. The Vipps backend makes a call to the merchant's `callbackUrl` with information about the payment.
4. Once payment process is completed, Vipps redirects to the
   `fallBack` URL that merchant provided earlier (see above).

#### Vipps not installed

1. The landing page (in the browser) prompts the user for the phone number.
2. Vipps sends a push notification, with a notification on the landing page
   to continue the payment on the phone.
3. The user accepts or rejects the payment in Vipps.
4. The Vipps backend makes a call to the merchant's `callbackUrl` with information about the payment.
5. Once payment process is completed, Vipps redirects to the
   `fallBack` URL that the merchant provided earlier.

**Please note:**
1. Vipps can not guarantee that the user will get to the
   `fallBack` URL, since the user may switch away from Vipps or "kill" the app,
   or there may be network or battery problems, etc before the URL is opened.
2. If the user has started the payment in an embedded browser, such as in
   Facebook or Instagram, it is not possible for Vipps to open the
   `fallBack` URL in the embedded browser. The phone OS will always open URLs
   in the default browser.
3. Because of the above, a successful payment _must not_ rely on session cookies
   in the browser.
4. Vipps can not guarantee a particular sequence, as this depends on user
   actions, network connectivity/speed, etc. Because of this, it is not
   possible to base an integration on a specific sequence of events.

## Desktop flow

![Desktop landing page](images/vipps-ecom-screenshot-landing-page.png)

### Desktop browser initiated payments

1. The landing page will be opened in the desktop browser.
2. The landing page will prompt for the user’s phone number.
2. Vipps sends a push notification, with a notification on the landing page
   to continue the payment on the phone.
4. The user accepts or rejects the payment in Vipps.
5. The Vipps backend makes a call to the merchant's `callbackUrl` with information about the payment.
6. Once payment process is completed, the landing page will redirect to the
   `fallBack` URL that merchant provided earlier (see above).

### Payments initiated in an app

Merchants can signal that the request is coming from their native app by passing
the `isApp:true` parameter.

In this case, the Vipps backend returns an URL that
works as a native app deeplink to Vipps (eg. with a `vipps://` scheme),
which automatically opens Vipps with app-switch.  
**Please note:** In our test environment (MT) the scheme is `vippsMT://`

The landing page is not involved in this flow, since the merchant app is
expected to use the `vipps://` URL to deeplink straight to Vipps.

1. Merchant initiates the payment with `isApp:true` parameter: [`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/initiatePaymentV3UsingPOST).
2. Vipps returns a `deeplink` URL as response to initiate payment.
3. The merchant uses the `vipps://` URL to invoke Vipps.
4. Vipps is automatically opened.
5. The user accepts or rejects the payment request in Vipps.
6. The Vipps backend makes a call to the merchant's `callbackUrl` with information about the payment.
7. When the payment process is completed, Vipps redirects to the merchant using the `fallBack` URL.

### Initiate payment flow: API calls

Swagger:
[`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/initiatePaymentV3UsingPOST)

A minimal example:

```json
{
  "customerInfo": {
  },
  "merchantInfo": {
    "merchantSerialNumber": "123456",
    "callbackPrefix":"https://example.com/vipps/callbacks-for-payment-update",
    "fallBack": "https://example.com/vipps/fallback-result-page/acme-shop-123-order123abc"
  },
  "transaction": {
    "orderId": "acme-shop-123-order123abc",
    "amount": 20000,
    "transactionText": "One pair of Vipps socks"
  }
}
```

An express payment example with more parameters provided:

```json
{
  "customerInfo": {
    "mobileNumber": "48059528"
  },
  "merchantInfo": {
    "authToken": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni <snip>",
    "callbackPrefix": "https://example.com/vipps/callbacks-for-payment-update",
    "consentRemovalPrefix": "https://example.com/vipps/consents/",
    "fallBack": "https://example.com/vipps/fallback-result-page/acme-shop-123-order123abc",
    "isApp": false,
    "merchantSerialNumber": 123456,
    "shippingDetailsPrefix": "https://example.com/vipps/shipping/",
    "paymentType": "eComm Express Payment",
    "staticShippingDetails": [
        {
            "isDefault": "N",
            "priority": 1,
            "shippingCost": 30.0,
            "shippingMethod": "Walking",
            "shippingMethodId": "123abc"
        },
        {
            "isDefault": "Y",
            "priority": 2,
            "shippingCost": 30.0,
            "shippingMethod": "Running",
            "shippingMethodId": "321abc"
        }
    ]
  },
  "transaction": {
    "amount": 20000,
    "orderId": "acme-shop-123-order123abc",
    "timeStamp": "2018-12-12T11:18:38.246Z",
    "transactionText": "One pair of Vipps socks"
  }
}
```

**Please note:** Do not send sensitive information in the `transactionText` field.
See
[Datatilsynet's information](https://www.datatilsynet.no/rettigheter-og-plikter/personopplysninger/)
about which types of information is sensitive (in Norwegian).

#### The Vipps deeplink URL

Vipps responds to the initiate payment request with an URL.
The URL depends on whether the initiate request was provided the `isApp` parameter:

* `"isApp":false` (or not provided): The URL is for the Vipps "landing page", with `https://`.
* `"isApp": true`: The URL is for an deeplink, for app-switch to Vipps, with `vipps://`.


Example: Response body for `"isApp":false`, to the landing page:

```json
{
    "orderId": "acme-shop-123-order123abc",
    "url": "https://api.vipps.no/dwo-api-application/v1/deeplink/vippsgateway?v=2&token=eyJraWQiOiJqd3RrZXkiLC <snip>"
}
```
Example: Response body for `"isApp":true`, with a deeplink for app-switch:

```json
{
    "orderId": "acme-shop-123-order123abc",
    "url": "vipps://?token=eyJraWQiOiJqd3RrZXkiLCJhbGciOiJSUzI1NiJ9.eyJzdWIiO <snip>"
}
```

The `url` is truncated, but the format is correct.

The deeplink URL is only valid for five minutes.
Attempts at using it after that will result in a timeout and an error.

See the FAQ:
[Can I send a Vipps payment link in an SMS or email?](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api-faq.md#can-i-send-a-vipps-payment-link-in-an-sms-or-email).

## Payment identification

A payment is uniquely identified by the combination of `merchantSerialNumber`
and `orderId`:

* `merchantSerialNumber`: The merchant's Vipps id. Example: `123456`.
* `orderId`: Must be unique for the `merchantSerialNumber`. Example: `acme-shop-123-order123abc`.
  See: [orderId recommendations](#orderid-recommendations).

To initiate an express checkout payment the payment initiation call must include
the `"paymentType": "eComm Express Payment"` parameter. If this parameter is not
passed, the payment type will default to regular payment.

## Payment retries

If a user cancels, or does not act, on a payment, there is no way to "retry"
a payment.

The initiate call is not idempotent, so the closest to a "retry"
is to make a new initiate call with a new `orderId`. Vipps has no concept
of relation between orders, so the "retry" payment is in no way connected
to the first payment attempt.

## orderId recommendations

A `orderId` must be unique for the MSN (Merchant Serial Number, the id of
the sale unit). The `orderId` does not need to be globally unique, so several
MSNs may use the same `orderId`, as long as it is unique for each sale unit.

If you ever have a problem that requires us to search in our logs, we need
`orderId`s that are "unique enough" to actually find them. An `orderId` that
is just a number may not be possible to find.

While the minimum length for `orderId` _technically_ is just one character,
we strongly recommend using at least 6 characters, and a combination of numbers
and characters.

The maximum length of an `orderId` is 50 alphanumeric characters:
a-z, A-Z, 0-9 and '-'.

A good starting point is to use UUID,
[universally unique identifiers](https://en.wikipedia.org/wiki/Universally_unique_identifier),
on the format `123e4567-e89b-12d3-a456-426614174000`.
UUIDs are not always the most human-friendly, so see the other tips below too.

We _strongly_ recommend to use `orderId` format that makes it easy to
search for them in logs. This means that `abc-123-def-456` is a better
format than `123456`.

Leading zeros should be avoided, as some applications (like Excel)
tend to remove them, and this may cause misunderstandings.

With multiple sale units, prefixing the `orderId` with the MSN
for each sale unit is recommended: If the MSN is `654321`, the
`orderId`s could start at `654321-0000-0000-0001` and increment by 1
for each order, or some similar, unique and readable pattern.

## URL Validation

All URLs in Vipps eCommerce API are validated with the
[Apache Commons UrlValidator](https://commons.apache.org/proper/commons-validator/apidocs/org/apache/commons/validator/routines/UrlValidator.html).

If `isApp` is true, the `fallBack` is not validated with Apache Commons UrlValidator,
as the app-switch URL may be something like `vipps://`, which is not a valid URL.

The endpoints required by Vipps must be publicly available.

URLs that start with `https://localhost` will be rejected. If you want to use
localhost as fallback, please use `http://127.0.0.1`.
It is, naturally, not possible to use `https://localhost` or
`http://127.0.0.1` for the callback, as the Vipps backend would then call itself.

Here is a simple Java class suitable for testing URLs,
using the dummy URL `https://example.com/vipps/fallback-result-page/acme-shop-123-order123abc`:

```java
import org.apache.commons.validator.routines.UrlValidator;

public class UrlValidate {
 public static void main(String[] args) {
  UrlValidator urlValidator = new UrlValidator();

  if (urlValidator.isValid("https://example.com/vipps/fallback-result-page/acme-shop-123-order123abc")) {
   System.out.println("URL is valid");
  } else {
   System.out.println("URL is invalid");
  }
 }
}
```


## Callbacks

Callbacks allow Vipps to send the payment order details to the merchant,
and normally offer at better (and faster) user experience than relying on
polling
[GET:/ecomm/v2/payments/{orderId}/details](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getPaymentDetailsUsingGET)
(See: [Polling guidelines](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#polling-guidelines)).

The request body contains different information depending on  whether the payment type is set to
`"eComm Regular Payment"` or `"eComm Express Payment"`:

* `"eComm Regular Payment"`: The callback contains the order and transaction details.
* `"eComm Express Payment"`: The callback contains the order and transaction details
  _and in addition_ the user details and shipping details.

The callback will be performed _only once_ during a payment process, for the
following events:
* Payment successful
* Payment failed
* Payment rejected
* Payment timed out

Vipps makes _one_ attempt at a callback, and can not guarantee that it succeeds,
as it depends on network, firewalls and other factors that Vipps can not control.

If the communication is broken during the process for some reason, and Vipps
is not able to execute callback to the merchant's server, the callback will
not be retried.

In other words, if the merchant does not receive any
confirmation on the payment request, the merchant _must_ call
[GET:/ecomm/v2/payments/{orderId}/details](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getPaymentDetailsUsingGET)
to get the status of the payment.

See the FAQ:
[Why do I not get callbacks from Vipps?](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api-faq.md#why-do-i-not-get-callbacks-from-vipps)

### Callback endpoints

The callback URLs are generated by taking a field from the payment initiation request from the merchant in
[`POST:/ecomm/v2/payments/{orderId}`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/transactionUpdateCallbackForRegularPaymentUsingPOST),
and then adding a suffix. See the "Endpoints required by Vipps" section in the [Swagger documentation](https://vippsas.github.io/vipps-ecom-api/#/).

If your `callbackPrefix` is `https://example.com/vipps/callback` and your
`orderId` is `acme-shop-123-order123abc`, Vipps will make a callback call to
`https://example.com/vipps/callback/v2/payments/acme-shop-123-order123abc`.

| Endpoint	| Field | Description |
| ----------- | ----------- | ----------- |
| [`POST:[callbackPrefix]/v2/payments/{orderId}`](https://vippsas.github.io/vipps-ecom-api/#/Endpoints_required_by_Vipps_from_the_merchant/transactionUpdateCallbackForRegularPaymentUsingPOST) | callbackPrefix | Status update sent from Vipps |
| [`POST:[shippingDetailsPrefix]/v2/payments/{orderId}/shippingDetails`](https://vippsas.github.io/vipps-ecom-api/#/Endpoints_required_by_Vipps_from_the_merchant/fetchShippingCostUsingPOST) | shippingDetailsPrefix | Request for getting shipping details. ([Express checkout](#express-checkout-api-endpoints-required-on-the-merchant-side) functionality) |
| [`DELETE:[consentRemovalPrefix]/v2/consents/{userId}`](https://vippsas.github.io/vipps-ecom-api/#/Endpoints_required_by_Vipps_from_the_merchant/removeUserConsentUsingDELETE) | consentRemovalPrefix | User has removed consent for sharing details. ([Express checkout](#express-checkout-api-endpoints-required-on-the-merchant-side) functionality) |

### Callback examples

Examples of [`POST:[callbackPrefix]/v2/payments/{orderId}`](https://vippsas.github.io/vipps-ecom-api/#/Endpoints_required_by_Vipps_from_the_merchant/transactionUpdateCallbackForRegularPaymentUsingPOST).

Example: `"eComm Regular Payment"` callback:

```json
{
  "merchantSerialNumber": 123456,
  "orderId": "acme-shop-123-order123abc",
  "transactionInfo": {
    "amount": 20000,
    "status": "RESERVED",
    "timeStamp": "2018-12-12T11:18:38.246Z",
    "transactionId": "5001420062"
  }
}
```

Example: `"eComm Express Payment"` callback:

```json
{
  "merchantSerialNumber": 123456,
  "orderId": "acme-shop-123-order123abc",
  "shippingDetails": {
    "address": {
      "addressLine1": "Dronning Eufemias gate 42",
      "addressLine2": "Att: Rune Garborg",
      "city": "Oslo",
      "country": "Norway",
      "postCode": "0191"
    },
    "shippingCost": 109.99,
    "shippingMethod": "By cannon"
  },
  "transactionInfo": {
    "amount": 20000,
    "status": "RESERVE",
    "timeStamp": "2018-12-12T11:18:38.246Z",
    "transactionId": "5001420062"
  },
  "userDetails": {
    "bankIdVerified": "Y",
    "dateOfBirth": "12-3-1988",
    "email": "user@example.com",
    "firstName": "Ada",
    "lastName": "Lovelace",
    "mobileNumber": "12345678",
    "ssn": "12345678901",
    "userId": "1234567"
  }
}
```

**Please note:** Regular payments use `RESERVED`, but express payments use
`RESERVE`. We apologize for this annoying piece of technical debt, but will
not correct it (in this version of the API) as it will not be backwards compatible.

### Authorization for callbacks


To add authentication to the callbacks made by Vipps to the merchant,
the merchant may provide an `authToken`. This token will then be returned as
an `Authorization` header in the callback and shipping details requests made by
Vipps for that order. Please note that this is unrelated to the authentication
required by the Vipps API: See [Authentication](#authentication).

Swagger:
[`POST:[callbackPrefix]/v2/payments/{orderId}`](https://vippsas.github.io/vipps-ecom-api/#/Endpoints_required_by_Vipps_from_the_merchant/transactionUpdateCallbackForRegularPaymentUsingPOST)

### Vipps callback servers

The callbacks from Vipps are made from the servers described in
[Vipps request servers](https://github.com/vippsas/vipps-developers#vipps-request-servers).

Please make sure that requests from these servers are allowed through firewalls, etc.

**Please note:** Vipps may change the IP addresses that we make callbacks from.
To ensure that you are whitelisting the corrects IP addresses please use these
hostnames.  

### Callback URLs must be reachable

Please make sure that the callback URLs are reachable:

* URLs must be valid. See [URL Validation](#url-validation).
* Vipps does _not_ support sending requests to all ports, so to be safe use
  common port numbers such as: 80, 443, 8080.
* Vipps does _not_ support callback URLs that return `HTTP 301 Redirect`,
  `HTTP 302 Permanently Moved` or `HTTP 307 Temporary Redirect`.
  Vipps will not follow to a `Location`.
  The callback URLs _must_ be directly reachable.

### Callback statuses

These are the statuses provided by Vipps in the callbacks by
[`[callbackPrefix]/v2/payments/{orderId}`](https://vippsas.github.io/vipps-ecom-api/#/Endpoints_required_by_Vipps_from_the_merchant/transactionUpdateCallbackForRegularPaymentUsingPOST):

| Callback type    | Response         | Description                                 |
| ---------------- | ---------------- | ------------------------------------------- |
| Regular checkout | `RESERVED`       | Payment reserved, by user accepting the payment in Vipps |
|                  | `SALE`           | Payment captured with direct capture, by merchant (after `RESERVED`)  |
|                  | `RESERVE_FAILED` | Reserve failed because of insufficient funds, invalid card, or similar |
|                  | `SALE_FAILED`    | Direct failed because of insufficient funds, invalid card, or similar |
|                  | `CANCELLED`      | Payment canceled, by user in Vipps |
|                  | `REJECTED`       | User did not act on the payment (timeout, etc) |
| Express checkout | `RESERVE`        | Payment reserved, by user accepting the payment in Vipps (it _is_ correct that this is different from `RESERVED` for regular checkout - sorry)|
|                  | `SALE`           | Payment captured with direct capture, by merchant (after `RESERVE`)  |
|                  | `RESERVE_FAILED` | Reserve failed because of insufficient funds, invalid card, or similar |
|                  | `SALE_FAILED`    | Direct failed because of insufficient funds, invalid card, or similar |
|                  | `CANCELLED`      | Payment canceled, by user in Vipps  |
|                  | `REJECTED`       | User did not act on the payment (timeout, etc)  |

See also: [Get payment details](#get-payment-details).

## Timeouts

### Using a phone

Both the deeplink URL, which causes the app-switch to Vipps, and the landing
page displayed in browsers, is valid for 5 minutes.

If the user does not act on the app-switch (such as not attempting to log into
Vipps) within 5 minutes, the payment times out.

After the app-switch to Vipps, the user has another 5 minutes to complete the
payment in Vipps.

This means that the user has a total of 10 minutes to complete the payment.

### Using a laptop/desktop

If the user is using a laptop/desktop device, and the user must confirm or
enter the phone number on the landing page within 5 minutes.
If the user does not do so, the payment times out.

After the user has clicked "OK" on the landing page, the user
has an additional 5 minutes to complete the payment in Vipps.

This means that the user has a total of 10 minutes to complete the payment.


## Express checkout API endpoints required on the merchant side

The below endpoints are provided by the _merchant_ and consumed by Vipps during
express checkout payments. These endpoints are not required when using
regular checkout.

These endpoints are to be implemented by merchants in order for Vipps
to make calls to them. The documentation is included in the Swagger file for
reference only - these endpoints are _not_ callable at Vipps.

| Operation           | Description         | Endpoint          |
| ------------------- | ------------------- | ----------------- |
| Get shipping details | Used to fetch shipping information, **3 attempts with 3 seconds timeout** | [`POST:/v2/payments/{orderId}/shippingDetails`](https://vippsas.github.io/vipps-ecom-api/#/Endpoints_required_by_Vipps_from_the_merchant/fetchShippingCostUsingPOST)  |
| Transaction Update | A callback to the merchant for receiving post-payment information. | [`POST:/v2/payments/{orderId}`](https://vippsas.github.io/vipps-ecom-api/#/Endpoints_required_by_Vipps_from_the_merchant/transactionUpdateCallbackForRegularPaymentUsingPOST)  |
| Remove user consent | Used to inform merchant when the Vipps user removes consent to share information.  | [`DELETE:/v2/consents/{userId}`](https://vippsas.github.io/vipps-ecom-api/#/Endpoints_required_by_Vipps_from_the_merchant/removeUserConsentUsingDELETE)  |

Please note that if the shipping details are static (do not vary based on the
address), the parameter `staticShippingDetails` can be used in the initiate call.
If so, there is no need to implement
[`POST:/v2/payments/{orderId}/shippingDetails`](https://vippsas.github.io/vipps-ecom-api/#/Endpoints_required_by_Vipps_from_the_merchant/fetchShippingCostUsingPOST).

See the
Swagger file for details, and the
[example here](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#initiate-payment-flow-api-calls).

### Get shipping details

This API call is for express payments, and allows Vipps to get the shipping
cost and method based on the provided address and product details. This is only
relevant for express checkout payments where Vipps needs to present shipping
cost and method to the Vipps user.
This service must be implemented by the merchant.

Swagger: [`POST:[shippingDetailsPrefix]/v2/payments/{orderId}/shippingDetails`](https://vippsas.github.io/vipps-ecom-api/#/Endpoints_required_by_Vipps_from_the_merchant/fetchShippingCostUsingPOST)


Request:

```json
{
    "addressId": 3960,
    "addressLine1": "Dronning Eufemias gate 42",
    "addressLine2": null,
    "country": "Norway",
    "city": "OSLO",
    "postCode": "0191",
    "addressType": "H",
}
```

Response:

```json
{
    "addressId": 3960,
    "orderId": "123456abc",
    "shippingDetails": [
        {
            "isDefault": "N",
            "priority": 1,
            "shippingCost": 30.0,
            "shippingMethod": "Walking",
            "shippingMethodId": "123abc"
        },
        {
            "isDefault": "Y",
            "priority": 2,
            "shippingCost": 30.0,
            "shippingMethod": "Running",
            "shippingMethodId": "321abc"
        }
    ]
}
```

### Transaction update

A callback to the merchant for receiving post-payment information:
[`POST:[callbackprefix]/v2/payments/{orderId}`](https://vippsas.github.io/vipps-ecom-api/#/Endpoints_required_by_Vipps_from_the_merchant/transactionUpdateCallbackForRegularPaymentUsingPOST).

See: [Callback statuses](#callback-statuses).

### Remove User Consent

[`DELETE:[consetRemovalPrefix]/v2/consents/{userId}`](https://vippsas.github.io/vipps-ecom-api/#/Endpoints_required_by_Vipps_from_the_merchant/removeUserConsentUsingDELETE)

This allows Vipps to send an end user's consent removal request to merchant.
This endpoint is required for express checkout. When receiving this request,
the merchant is obliged to handle the user details as per the GDPR guidelines.
The request path will include a `userId` that Vipps will have provided as
part of callback, and also made accessible through [`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getPaymentDetailsUsingGET).

## Skip landing page

*This functionality is only available for special cases.*

Skipping the landing page is reserved for special cases, where displaying it is not possible.

This feature has to be specially enabled by Vipps for eligible sale units:
The sale units must be whitelisted by Vipps.
Skipping the landing page is typically used at physical points of sale,
where there is no display available.

If the `skipLandingPage` property is set to `true` in the
[`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/initiatePaymentV3UsingPOST)
call, it will cause a push notification to be sent to the given phone number
immediately, without loading the landing page.

If the sale unit is not whitelisted, the request will fail and an error message will be returned.

If you want to check if a sale unit is allowed to use `skipLandingPage`:
1. Initiate a normal payment with `"skipLandingPage": true`.
2. Check the response code and message. The API will return an error if attempting to use
   `skipLandingPage` without being whitelisted.

If you need to be whitelisted, instructions can be found in the
[FAQ](https://github.com/vippsas/vipps-psp-api/blob/master/vipps-psp-api-faq.md#is-it-possible-to-skip-the-landing-page).

**Please note:** When using `skipLandingPage`, the user is not sent to a
URL after complation of the payment. The "result page" is just the confirmation
in Vipps. The `fallback` URL send in the API request can therefore be the
merchant's main URL, like `https://example.com`, etc.

## Reserve

![Payment confirmation](images/vipps-flow-reserve.png)

When the user confirms, the payment status changes to `RESERVE`.
The respective amount will be reserved for future capturing.

## The Vipps landing page

![The Vipps landing page](images/vipps-flow-landing-page.png)

When a user is directed to the `url` from initiate payment, they will either be taken to Vipps or to the Vipps landing page:

* In a mobile browser, the user will be redirected to Vipps.
* In a desktop browser, the landing page prompts the user for the phone number (the number may also be pre-filled).
  The user enters or confirms the phone number. Then on their phone, the user gets a push notification and Vipps then prompts for confirmation.

The Vipps landing page is mandatory, and provides a consistent and recognizable user experience,
that helps guide the user through the payment flow.
In this way Vipps takes responsibility for helping the user from the browser to the app,
and to complete the payment in a familiar way.

The user's phone number can be set in the payment initiation call:
[`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/initiatePaymentV3UsingPOST).

The user's phone number is remembered by the user's browser,
eliminating the need for re-typing it on subsequent purchases.

See also: [Skip landing page](#skip-landing-page).

## Capture

Capture is done with
[`POST:/ecomm/v2/payments/{orderId}/capture`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/capturePaymentUsingPOST).

We strongly recommend to use the idempotency key `X-Request-Id`. If a capture
request fails for any reason, it can be retried with the same idempotency key.

You can use any unique id for your `X-Request-Id`.
See the API specification for details.

**Please note:** It is important to check the response of the `/capture`
call. The capture is only successful when the response is `HTTP 200 OK`.

Use
[`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/getPaymentDetailsUsingGET)
to get all the details of a payment.

There are two types of capture: _reserve capture_ and _direct capture_.
Reserve capture is the default. When you initiate a payment it will be reserved
until you capture it. With direct capture the reservation is instantly captured.

*  If a payment has been reserved, the merchant can make a
  [`/cancel`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/cancelPaymentRequestUsingPUT)
  call to
  immediately release the reservation and make available in the customer's
  account.
* If a payment has been captured, the merchant has to make a
  [`/refund`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/refundPaymentUsingPOST)
  call, and
  it then takes a few days before the amount is available in the customer's account.

See the FAQ:
[What is the difference between "Reserve Capture" and "Direct Capture"?](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api-faq.md#what-is-the-difference-between-reserve-capture-and-direct-capture.)

### Reserve capture

_Reserve capture_ is the normal flow.

When then end user approves an initiated payment it will be reserved until you
capture it. When the order is reserved the amount is marked as reserved by the
bank, but not transferred.

### Direct capture

When _direct capture_ is activated, all payment reservations will instantly be captured.

When doing _direct capture_ Vipps is responsible for the customer receiving the purchased product.
Because of this, _direct capture_ requires additional compliance checks, and the following
requirements apply:
* The merchant must have an annual revenue of more than 10 million NOK
* The merchant must have a Key Account Manager (KAM) in Vipps
* The merchant must have a partner that is responsible for the Vipps integration

To request _direct capture_, please contact your KAM.

Capture payment allows the merchant to capture the reserved amount.
The API allows for both a _full amount capture_ and a _partial amount capture_.

The amount to capture cannot be higher than the reserved amount. According to
Norwegian regulations, capture can not be done before the goods have been shipped.
The transaction text is mandatory, and is displayed to end user in Vipps.

### Partial capture

Partial capture may be used if not the entire order is shipped or for other
reasons. Partial capture may be called as many times as required as long as
there is a remaining reserved amount to capture.

If one or more partial captures have been made, the remaining amount
(if there is one) will be automatically refunded after a few days.
See the FAQ for
[details](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api-faq.md#for-how-long-is-an-initiated-payment-reserved).

It is not possible to refund the remaining amount (since it has not been captured),
and it is not possible to cancel the reservation, since some of it has been captured.

There is only a need to specify the `amount` when doing a partial capture.
To perform capture of the entire amount `amount` can be set to `null` or `0`

These two truncated examples show the responses for a reservation of
200.00 NOK, and a partial capture of 100.00 NOK:

```json
"transactionSummary": {
    "capturedAmount": 20000,
    "remainingAmountToCapture": 0,
    "refundedAmount": 0,
    "remainingAmountToRefund": 0
}
```

```json
"transactionsummary": {
    "capturedAmount": 10000,
    "remainingAmountToCapture": 10000,
    "refundedAmount": 0,
    "remainingAmountToRefund": 0
}
```

## Cancel

The Cancel request allows the merchant to cancel a reserved or initiated transaction.

The payment flow can be aborted, under certain circumstances:

* When the user cancels (rejects) the initiated payment in Vipps.
* When the merchant cancels.
* Timeouts: If the user does not confirm, etc.

Partially captured reservations can not be cancelled.

After cancellation, the order gets a new status:
* If an order is cancelled by the merchant, it gets the status `VOID`.
* If an order is cancelled by the user, it gets the status `CANCEL`.

Swagger: [`PUT:/ecomm/v2/payments/acme-shop-123-order123abc/cancel`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/cancelPaymentRequestUsingPUT)

Request:

```json
{
  "merchantInfo": {
    "merchantSerialNumber": "123456"
  },
  "transaction": {
    "transactionText": "No socks for you!"
  }
}
```

Response:

```json
{
    "orderId": "acme-shop-123-order123abc",
    "transactionInfo": {
        "amount": 20000,
        "transactionText": "No socks for you!",
        "status": "Cancelled",
        "transactionId": "5001420063",
        "timeStamp": "2018-11-14T15:31:10.004Z"
    },
    "transactionSummary": {
        "capturedAmount": 0,
        "remainingAmountToCapture": 0,
        "refundedAmount": 0,
        "remainingAmountToRefund": 0
    }
}
```

### Cancelling a pending order

If you wish to cancel a transaction before the customer can confirm the payment
in Vipps, you can send a
[`PUT:/ecomm/v2/payments/{orderId}/cancel`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/cancelPaymentRequestUsingPUT)
request while the transaction is in the `INITIATE` stage.

This may be useful in face-to-face situations where a customer's phone runs out
of battery. This should not be considered a consistent or guaranteed operation,
as the `/cancel` request is independent by any actions taken by the user in the app.

If the
[`PUT:/ecomm/v2/payments/{orderId}/cancel`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/cancelPaymentRequestUsingPUT)
request is successful, the payment state will be:

```
transactionInfo.status: "Cancelled"
```

A call to
[`GET:/ecomm/v2/payments/{orderId}/details](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/getPaymentDetailsUsingGET)
for the same order will return the following, regardless of whether the
transaction has been reserved:

```
$.transactionLogHistory.0.operation: "CANCEL"
```

**Note_**  If the user is in a 3DSecure session, the payment can not be cancelled as described above.

## Refund

The merchant can initiate a refund of the captured amount.
The refund can be a partial or full.

Partial refunds are done by specifying an `amount` which is lower than the captured amount.
The refunded amount cannot be larger than the captured amount.

In a capture request the merchant may also use the `X-Request-Id`header. This header is an idempotency header ensuring that if the merchant retries a request with the same `X-Request-Id` the retried request will not make additional changes.

You can use any unique id for your `X-Request-Id`.
See the API specification for details.

Refunds can be made up to 365 days after capture.

Swagger: [`POST:/ecomm/v2/payments/acme-shop-123-order123abc/refund`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/refundPaymentUsingPOST)

Request:

```json
{
    "merchantInfo": {
        "merchantSerialNumber": "123456"
    },
    "transaction": {
        "amount": 20000,
        "transactionText":"Refund of Vipps socks"
    }
 }
```

Response:

```json
{
    "orderId": "acme-shop-123-order123abc",
    "transaction": {
        "amount": 20000,
        "transactionText": "Refund of Vipps socks",
        "status": "Refund",
        "transactionId": "5600727726",
        "timeStamp": "2018-11-14T15:23:02.286"
    },
    "transactionSummary": {
        "capturedAmount": 20000,
        "remainingAmountToCapture": 0,
        "refundedAmount": 20000,
        "remainingAmountToRefund": 0
    }
}
```

## Recurring eCommerce payments

Recurring eCommerce is its own seperate product and can be found descriped in details in
our [Recurring Repo.](https://github.com/vippsas/vipps-recurring-api)


## Get payment details

Retrieves the full history of a payment, and whether the operations were
successful or not.

Swagger: [`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/getPaymentDetailsUsingGET)

### Payment states

| #   | From-state | To-state | Description                                     | Operation      |
| --- | ---------- | -------- | ----------------------------------------------- | -------------- |
| 0   | -          | Initiate | Payment initiation                              | `INITIATE`     |
| 1   | Initiate   | -        | The merchant has initiated the payment          | `INITIATE`     |
| -   |            | Reserve  | The user has accepted the payment and amount has been reserved | `RESERVE` |
| -   |            | Cancel   | The user has canceled the order                 | `CANCEL`       |
| 2   | Reserve    | Capture  | The merchant captures the payment, and ships    | `CAPTURE`      |
| -   |            | Cancel   | The merchant has canceled the order             | `VOID`         |
| 3   | Capture    | --       | A final state: Payment fully processed          | `CAPTURE`      |
| -   |            | Refund   | The merchant has refunded the money to the user | `REFUND`       |
| 4   | Cancel     | --       | A final state: Payment cancelled                | -              |
| 5   | Refund     | --       | A final state: Payment refunded                 | -              |

## Requests and responses

Please note that the response from
[`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getPaymentDetailsUsingGET))
always contains _the entire history_ of payments for the order, not just the current status.

**Important:** The `operationSuccess` field indicates whether an operation was successful or not.

| Response    | Description |
| ---------- | --------------------------------------------------------- |
| `INITIATE` | Payment initiated, by merchant |
| `RESERVE`  | Payment reserved, by user accepting the payment in Vipps |
| `CAPTURE`  | Payment captured, by merchant (after `RESERVE`) |
| `REFUND`   | Payment refunded, by merchant (after `CAPTURE`) |
| `CANCEL`   | Payment canceled, by user in Vipps |
| `SALE`     | Payment captured with direct capture, by merchant |   
| `VOID`     | Payment canceled, by merchant |

### Example response

```json
{
    "orderId": "acme-shop-123-order123abc",
    "transactionSummary": {
        "capturedAmount": 20000,
        "remainingAmountToCapture": 0,
        "refundedAmount": 20000,
        "remainingAmountToRefund": 0,
        "bankIdentificationNumber": 492560
    },
    "transactionLogHistory": [
        {
            "amount": 20000,
            "transactionText": "Refund of Vipps socks",
            "transactionId": "5600727726",
            "timeStamp": "2018-11-14T15:23:02.286Z",
            "operation": "REFUND",
            "requestId": "1542208972",
            "operationSuccess": true
        },
        {
            "amount": 20000,
            "transactionText": "One pair of Vipps socks",
            "transactionId": "5001420058",
            "timeStamp": "2018-11-14T15:22:46.680Z",
            "operation": "CAPTURE",
            "requestId": "1542208966",
            "operationSuccess": true
        },
        {
            "amount": 20000,
            "transactionText": "One pair of Vipps socks",
            "transactionId": "5001420062",
            "timeStamp": "2018-11-14T15:21:22.126Z",
            "operation": "RESERVE",
            "requestId": "",
            "operationSuccess": true
        },
        {
            "amount": 20000,
            "transactionText": "One pair of Vipps socks",
            "transactionId": "5001420062",
            "timeStamp": "2018-11-14T15:21:04.697Z",
            "operation": "INITIATE",
            "requestId": "",
            "operationSuccess": true
        }
    ]
}
```

**Please note:** The `transactionSummary` will not be part of the response if
the user not reacted to the Vipps landing page or app-switch. `bankIdentificationNumber` will be part of `transactionSummary` only in the response of `GET:/ecomm/v2/payments/{orderId}/details` endpoint.

## Polling guidelines

General guidelines for When to start polling with
[`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/getPaymentDetailsUsingGET):

1. Start after 5 seconds
2. Check every 2 seconds

These are reasonable values, but different merchants have different use cases,
and values should be adapted to the specific case.

See [Timeouts](#timeouts) for details about timeouts.

## Get payment status

**IMPORTANT: This endpoint is deprecated. Use [Get payment details](#get-payment-details).**

| Status     | Description                      |
| ---------- | -------------------------------- |
| `INITIATE` | Payment initiated, by merchant |
| `REGISTER` | Vipps registers payment for Reserve. |
| `RESERVE`  | Payment reserved, by user accepting the payment in Vipps.  |
| `SALE`     | Payment captured with direct capture, by merchant  |
| `CAPTURE`  | Payment captured with reserve capture, by merchant |
| `REFUND`   | Payment refunded, by merchant.|
| `CANCEL`   | Payment canceled, by user in Vipps |
| `VOID`     | Payment canceled, by merchant |
| `FAILED`   | Payment failed failed because of no coverage, outdated card details or similar. |
| `REJECTED` | No user action in Vipps, i.e timeout. |

## Userinfo

Vipps offers the possibility for merchants to ask for the user's profile
information as part of the payment flow. This is done by adding a `scope`
parameter to the initiate call:
[`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/initiatePaymentV3UsingPOST).

If the enduser has not already consented to sharing information from Vipps to the merchant the user will be asked for such consent before completing the payment flow. Once the payment flow is completed the merchant can get the profile information from our Userinfo endpoint. The Userinfo endpoint is shared with [Vipps login](https://github.com/vippsas/vipps-login-api) and the merchant needs to have activated Vipps login to use this feature.

To activate for Vipps Login - go to the developer section under https://portal.vipps.no and click "Setup login" for the desired account. When this is enabled, you get access to both Vipps login as a product, or fetching the users profileinfo can be used as standalone. If you are only using userinfo to fetch users profile info as part of a payment flow, you do not need to input redirect URIs when setting up login.

A users consent to share information with a merchant applies accross our services. Thus, if the merchant implements Vipps login in addition to profile information as part of the payment flow, the merchant can also use Vipps to log the user in without the need for additional consents.

### Scopes

| Scopes      | Description                                    | User consent required  |
| ------------| -----------------------------------------------|-------- |
| `address`     | List containing the user's addresses. Will always contain the home address, but can also include work and other.    |   yes   |
| `birthDate`   | Birth date (BankID verified)                               |   yes   |
| `email`       | Email address (verified), the flag "email_verified : true" in the response indicates whether the email address is verified                                   |   yes   |
| `name`        | First, middle and given name (verified with National Population Register)              |   yes   |
| `phoneNumber` | Phone number (verified - the number used when creating the Vipps account)                          |   yes   |
| `nin`        | Norwegian national identity number (verified with BankID). **NB:** Merchants need to apply for access to NIN. See: [Who can get access to NIN and how?](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api-faq.md#who-can-get-access-to-nin-and-how) |   yes      |
| `accountNumbers` | User bank account numbers. **NB:** Merchants need to apply for access to accountNumbers. See: [Who can get access to account numbers and how?](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api-faq.md#who-can-get-access-to-accountnumbers-and-how) |   yes      |

See the API specification for the formats and other details for each scope.

**Please note:** If the e-mail address that is delivered has the flag `email_verified : false`
this address should not be used to link the user to an existing account without
further authentication. Such authentication could be to prompt the user to
login to the original account or confirm the account linking by having a
confirmation link sent to the email address.

### Userinfo call by call guide

Scenario: You want to complete a payment and get the name and phone number of
a customer.

1. Retrieve the access token:
   [`POST:/accesstoken/get`](https://vippsas.github.io/vipps-recurring-api/#/Access%20Controller/getAccessToken).
2. Add scope to the transaction object and include the scopes you wish to get
   access to (valid scopes) before calling
   [`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/initiatePaymentV3UsingPOST)
3. The user consents to the information sharing and perform the payment in Vipps.
4. Retrieve the `sub` by calling
   [`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/getPaymentDetailsUsingGET)
5. Generate an OAuth 2 access token with
   [`POST:/oauth2/token`](https://vippsas.github.io/vipps-login-api/#/Vipps%20Log%20In%20API/oauth2Token),
   using the `clientId:client_secret` as base64 as described in
   [Vipps Login access token](#vipps-login-access-token),
   with the `grant_type` set to `"client_credentials"`.
6. Using the access token from step 5, call
   [`GET:/vipps-userinfo-api/userinfo/{sub}`](https://vippsas.github.io/vipps-login-api/#/Vipps%20Log%20In%20API/userinfo)
   to retrieve the user's information.

**Important note:** The API call to
[`GET:/vipps-userinfo-api/userinfo/{sub}`](https://vippsas.github.io/vipps-login-api/#/Vipps%20Log%20In%20API/userinfo)
must _not_ include the subscription key used for the eCom API as Login
is _not_ under the same subscription, and will result in a
`HTTP Unauthorized 401` error.

### Get userinfo

The
[`GET:/vipps-userinfo-api/userinfo/{sub}`](https://vippsas.github.io/vipps-login-api/#/Vipps%20Log%20In%20API/userinfo)
endpoint is shared between the Vipps eCom API and the
[Vipps Login API](https://github.com/vippsas/vipps-login-api)
and the merchant needs to have
[activated Vipps Login](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api-faq.md#how-can-i-activate-and-set-up-vipps-login)
to use this feature.

Once the user completes the session a unique identifier `sub` can be retrieved from the
[`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/getPaymentDetailsUsingGET) endpoint.

Example `sub` format:

```json
"sub": "c06c4afe-d9e1-4c5d-939a-177d752a0944",
```

This `sub` is a link between the merchant and the user and can used to retrieve
the user's details from Vipps Login:
[`GET:/vipps-userinfo-api/userinfo/{sub}`](https://vippsas.github.io/vipps-login-api/#/Vipps%20Log%20In%20API/userinfo)

**Please note:** There is a time limit of 168 hours to retrieve the consented profile data from the userinfo endpoint.

### Vipps Login access token

Accessing the Login `userinfo` endpoint required the Vipps Login access token: [`POST:/oauth2/token`](https://vippsas.github.io/vipps-login-api/#/Vipps%20Log%20In%20API/oauth2Token). The client constructs the request by adding the parameters described below to the HTTP body by using the `application/x-www-form-urlencoded` format.

**Request**

*Headers*

| Header            | Description                           |
| ----------------- | ------------------------------------- |
| Content-Type      | "application/x-www-form-urlencoded"   |
| Authorization     | "Basic {Client Credentials}"          |

The Client Credentials is a base 64 encoded string consisting of the Client id
and secret issued by Vipps joined by ":"

Example of generating the client credentials in bash where:

- client_id = 123456-test-4a3d-a47c-412136fd0871
- client_secret = testdzlJbUZaM1lqODlnUUtrUHI=

```bash
echo "123456-test-4a3d-a47c-412136fd0871:testdzlJbUZaM1lqODlnUUtrUHI=" | base64
```

Which results in the base64 client secret: `MTIzNDU2LXRlc3QtNGEzZC1hNDdjLTQxMjEzNmZkMDg3MTp0ZXN0ZHpsSmJVWmFNMWxxT0RsblVVdHJVSEk9Cg==`

*Form content*

| Key               | Description                         |
| ----------------- | ----------------------------------- |
| grant_type        | Value **must** be the actual string  `"client_credentials"`, not the value generated above.  |

**Example response:**

```json
{
  "access_token": "hel39XaKjGH5tkCvIENGPNbsSHz1DLKluOat4qP-A4.WyV61hCK1E2snVs1aOvjOWZOXOayZad0K-Qfo3lLzus",
  "id_token": "eyJhbGciOiJSUzI1NiIsImtpZCI6InB1YmxpYzo4MGYzYzM0YS05Nzc <snip>",
  "expires_in": 3599,
  "scope": "openid",
  "token_type": "bearer"
}
```

### Userinfo call

This endpoint returns the payload with the information that the user has consented to share, which is provided in the OAuth 2.0 access token.
You can learn more at the [OIDC Standard](https://openid.net/specs/openid-connect-core-1_0.html#UserInfo).

Call [`GET:/vipps-userinfo-api/userinfo/{sub}`](https://vippsas.github.io/vipps-login-api/#/Vipps%20Log%20In%20API/userinfo) with the `sub` that was retrieved earlier. See below on how to construct the call.

**Request**

*Headers*

| Header            | Description                            |
| ----------------- | -------------------------------------  |
| Authorization     | "Bearer {Access Token}"                |

The access token is received on a successful request to the token endpoint described above.

**Example response:**

```json
{
    "sub": "c06c4afe-d9e1-4c5d-939a-177d752a0944",
    "birthdate": "1815-12-10",
    "email": "user@example.com",
    "email_verified": true,
    "nin": "10121550047",
    "name": "Ada Lovelace",
    "given_name": "Ada",
    "family_name": "Lovelace",
    "sid": "7d78a726-af92-499e-b857-de263ef9a969",
    "phone_number": "4712345678",
    "address": {
        "street_address": "Suburbia 23",
        "postal_code": "2101",
        "region": "OSLO",
        "country": "NO",
        "formatted": "Suburbia 23\\n2101 OSLO\\nNO",
        "address_type": "home"
    },
    "other_addresses": [
        {
            "street_address": "Fancy Office Street 2",
            "postal_code": "0218",
            "region": "OSLO",
            "country": "NO",
            "formatted": "Fancy Office Street 2\\n0218 OSLO\\nNO",
            "address_type": "work"
        },
        {
            "street_address": "Summer House Lane 14",
            "postal_code": "1452",
            "region": "OSLO",
            "country": "NO",
            "formatted": "Summer House Lane 14\\n1452 OSLO\\nNO",
            "address_type": "other"
        }
    ],
    "accounts": [
        {
            "account_name": "My savings",
            "account_number": "12064590675",
            "bank_name": "My bank"
        }
    ]
}
```

**Please note:** More documentation about the token and userinfo endpoint can be found [here](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api.md#access-token).

![Userinfo sequence](images/userinfo-direct.png)

### Consent

A user's consent to share information with a merchant applies across all Vipps
services. Thus, if the merchant implements Vipps Login in addition to profile
information as part of the payment flow, the merchant can also use Vipps to
log the user in without the need for additional consent.

The user is presented with a consent card that must be accepted before
approving the payment in Vipps. The following screens show examples
of consent cards for Android(left) and iOS(right):

![Consent card](images/share-user-info.png)

**Please note:** This operation has an "all or nothing" approach, so a user must
complete a valid payment and consent to _all_ values in order to complete the
session. If a user chooses to reject the terms the payment will not be
processed. Unless the whole flow is completed, this will be handled as a
failed payment by the eCom API.

## HTTP response codes

This API returns the following HTTP statuses in the responses.
See the [Swagger specification](./) for more details.

| HTTP status             | Description                                             |
| ----------------------- | ------------------------------------------------------- |
| `200 OK`                | Request successful                                      |
| `201 Created`           | Request successful, resource created                    |
| `204 No Content`        | Request successful, but empty result                    |
| `400 Bad Request`       | Invalid request, see the error for details              |
| `401 Unauthorized`      | Invalid credentials                                     |
| `403 Forbidden`         | Authentication ok, but credentials lacks authorization  |
| `404 Not Found`         | The resource was not found                              |
| `409 Conflict`          | Unsuccessful due to conflicting resource                |
| `429 Too Many Requests` | Look at table below to view current rate limits         |
| `500 Server Error`      | An internal Vipps problem.                              |

HTTP responses with errors from the application gateway contain one error JSON object.
Error responses produced from the application gateway include `401`, `403`, `422` and `429`.

HTTP responses with errors from the Vipps backend will contain an _array_ of JSON objects.

See [Errors](#errors) for more details.

## Rate limiting

We have added rate limiting to our APIs (HTTP 429 Too Many Requests) to prevent fradulent and wrongful behaviour, and to increase the stability and security of our APIs. The limits should not affect normal behaviour, but please contact us if you notice any unexpected behaviour.

| API                                                                                                  | Limit          | Key           | Explanation |
|------------------------------------------------------------------------------------------------------|----------------|---------------|-------------|
| [Initiate](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/initiatePaymentV3UsingPOST) | 2 per minute   | orderId + msn | Two calls per minute per unique orderId |
| [Cancel](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/cancelPaymentRequestUsingPUT) | 5 per minute   | orderId + msn | Five calls per minute per unique orderId |
| [Capture](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/capturePaymentUsingPOST)     | 5 per minute   | orderId + msn | Five calls per minute per unique orderId |
| [Refund](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/refundPaymentUsingPOST)       | 5 per minute   | orderId + msn | Five calls per minute per unique orderId |
| [~~Status~~](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/getOrderStatusUsingGET)   | 120 per minute | orderId + ocp | 120 calls per minute per unique orderId |
| [Details](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/getPaymentDetailsUsingGET)   | 120 per minute | orderId + ocp | 120 calls per minute per unique orderId |

**Please note:** The "Key" column is important. The above means that we allow two
[Initiate](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/initiatePaymentV3UsingPOST)
calls per minute _per unique orderId_ for that MSN. This
is to prevent too many initiate calls for the same payment. The overall limit
for number of _different_ payments is _far_ higher than 2. The same goes for
[Capture](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/capturePaymentUsingPOST):
You can make five capture calls per minute for
one unique orderId, and the limit for capture calls for different orderIds
is _far_ higher.

## Partner Keys

In addition to the normal
[Authentication](#authentication)
we offer _partner keys, which let a partner make API cals on behalf of a merchant.

If you are a Vipps Partner that is managing transactions on behalf of other
Vipps Merchants you can use your own credentials to authenticate, and then
send the `Merchant-Serial-Number` to identify which of your Vipps Merchant you
are acting on behalf of. The `Merchant-Serial-Number` must be sent in the header
of all requests. as described in our swagger documentation:

[`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/initiatePaymentV3UsingPOST)

Including the
[Optional HTTP Headers](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#optional-vipps-http-headers)
too will make it easier to investigate problems, if anything unexpected happens.
Partners may re-use the values of the `Vipps-System-Name` and `Vipps-System-Name`
in the plugins headers if having different values do not make sense.

Here's an example of headers (please refer to the
[OpenAPI/Swagger specification](https://github.com/vippsas/vipps-ecom-api)
for all the details):

```
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni <snip>
Ocp-Apim-Subscription-Key: 0f14ebcab0ec4b29ae0cb90d91b4a84a
Merchant-Serial-Number: 123456
Vipps-System-Name: Acme Enterprises Ecommerce DeLuxe
Vipps-System-Version: 3.1.2
Vipps-System-Plugin-Name: Point Of Sale Excellence
Vipps-System-Plugin-Version: 4.5.6
Content-Type: application/json
```

**Please note:** The Merchant Serial Number (MSN) is a unique id for the sale
unit that this payment is made for. This is a required parameter if you are a
Vipps eCom partner making payments on behalf of a merchant. The partner must
use the merchant's MSN (not the partner's MSN). This parameter is optional,
and recommended, for regular Vipps merchants making payments for themselves.

## Idempotency

In a capture request the merchant may also use the `X-Request-Id` header.
This header is an idempotency header ensuring that if the merchant retries
a request with the same `X-Request-Id` the retried request will not make
additional changes.

You can use any unique id for your `X-Request-Id`.
See the API specification for details.

Request:

[`POST:/ecomm/v2/payments/acme-shop-123-order123abc/capture`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/capturePaymentUsingPOST)

```json
{
    "merchantInfo": {
        "merchantSerialNumber": "123456"
    },
    "transaction": {
        "amount": 20000,
        "transactionText":"Socks on the way! Tracking code: abc-tracking-123"
    }
 }
```

Response:

```json
{
    "orderId": "acme-shop-123-order123abc",
    "transactionInfo": {
        "amount": 20000,
        "timeStamp": "2018-11-14T15:22:46.736Z",
        "transactionText": "Socks on the way! Tracking code: abc-tracking-123",
        "status": "Captured",
        "transactionId": "5001420058"
    },
    "transactionSummary": {
        "capturedAmount": 20000,
        "remainingAmountToCapture": 0,
        "refundedAmount": 0,
        "remainingAmountToRefund": 20000
    }
}
```

## Exception handling

The section below explains how Vipps handles different exception and errors.

See also [Errors](#errors).

### Connection timeout

Defining a socket timeout period is the common measure to protect server
resources and is expected. However, the time needed to fulfil a service requests
depends on several systems, which impose longer timeout period than usually
required.

We recommend setting no less than 1 second socket connection timeout
and 5 seconds socket read timeout while communicating with Vipps.

A good practice is, if/when the socket read timeout occurs, to call
[`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getPaymentDetailsUsingGET)
and check status of last transaction in transaction history prior
to executing the service call again.

### Callback aborted or interrupted

If the communication is broken during payment process for some reason,
either because of network problems, that the user abruptly closes the app or
something else, and Vipps is not able to execute callback, the callback will
not be retried.

In other words, if the merchant doesn’t receive any confirmation on payment
request call within callback timeframe, merchant must call
[`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getPaymentDetailsUsingGET)
to get the response of payment request.

### PSP connection issues

In cases of communication problems with Vipps' PSP, the response from Vipps
will be an error (see [Errors](#errors)).

The merchant should then call
[`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getPaymentDetailsUsingGET)
to check if the transaction request is processed before making a new call,
using same idempotency key (see [Idempotency](#idempotency)).

## App integration

Merchants may implement deep-linking, to trigger Vipps (we refer to this as "app-switch" here).

This may be done in two ways:

1. From a mobile or desktop browser. See [Desktop flow](#desktop-flow)
2. From an iOS or Android native app

After the user has finished (or cancelled) the payment in Vipps, the user is returned back to the specified `fallBack` URL. When the user arrives back in the merchant app or website, we _strongly_ recommend that you perform a call to the [payment details endpoint](#get-payment-details) to check the state
of the transaction. While some of the state of the eCom operation *can* be derived from things like whether or not user returned successfully from the
Vipps app, the most reliable approach to know the state of the payment is always to query the eCom API once the user arrive back in the merchant app/website.

## App-switching
1. The merchant need to pass the URL scheme of the app into the `fallBack` field in the [initiate payment request](#initiate-payment-flow-api-calls). See [App-switch on iOS](#app-switch-on-ios) and [App-switch on Android](#app-switch-on-android) for specifics.
2. The merchant app should open Vipps deeplink received from the [initiate payment request](#initiate-payment-flow-api-calls).
3. Once the operation in Vipps is completed, Vipps will redirect the user to the deeplink specified in the `fallBack` field.
4. The merchant app should query the [payment details endpoint](#get-payment-details) for updated status on the payment once user returns from Vipps.

**Please note:** Vipps will append a status at the end of the fallback URL. For example, if your `fallBack` URL is `merchantApp://result?myAppData`, Vipps
will append the status like: `merchantApp://result?myAppData&status=301`. **This status is deprecated, and should no longer be used.** Query the [payment details endpoint](#get-payment-details) for latest status instead.

### App-switch on iOS
Vipps on iOS requires a URL scheme in order to support app-switch.

See the official Apple documentation:
[Defining a Custom URL Scheme for Your App](https://developer.apple.com/documentation/uikit/core_app/allowing_apps_and_websites_to_link_to_your_content/defining_a_custom_url_scheme_for_your_app)


#### Switch from merchant app to Vipps

Below is sample code to open iOS Vipps application with `deeplinkURL`.

```java
NSString *url = deeplinkURL; // Use the deeplinkURL provided in the API response
if ([[UIApplication sharedApplication] canOpenURL:[NSURL URLWithString:url]]) {
  [[UIApplication sharedApplication] openURL:[NSURL URLWithString:url]]; }
else {
  // No Vipps app installed: Open app store page.
  // Once user installs Vipps, calling app needs to initiate deeplinking again in order to get the callback
  [[UIApplication sharedApplication] openURL:[NSURL URLWithString: @"https://itunes.apple.com/no/app/Vipps-by-dnb/id984380185"]];
}
```

Example of a `deeplinkURL`:

`vipps://?token=eyJraWQiOiJqd3RrZXkiLCJhbGciOiJSUzI1NiJ9.ey <snip>`

**Please note:** For production it's `vipps://`, but for our test environment it's `vippsMT://`.  
This is change from previously where the deeplink started with `vipps://` for both production and test.

#### Redirect back to the merchant app from Vipps app

Once the operation in Vipps is completed, Vipps will open the fallback URL.
For app-to-app integration, merchant app needs to be registered for a URL scheme
and pass the URL scheme in `fallBack` URL in the Vipps backend API.
The Vipps mobile application will use the URL to launch the merchant application.

### App-switch on Android

Vipps is launched with a standard intent, using the url returned from the eCom
API when the payment is created ("url": "vipps://?token=eyJraWQiOiJqd3RrZXki..")

#### Switching from merchant app to Vipps

Example of how to open Vipps:

```java
try {
  PackageManager pm = context.getPackageManager();
  PackageInfo info = pm.getPackageInfo( , PackageManager.GET_ACTIVITIES);
  if(versionCompare(info.versionName,   ) >= 0) {
    String uri = deeplinkURL; // Use deeplink url provided in ecom API response
    Intent intent = new Intent(Intent.ACTION_VIEW);
    intent.setData(Uri.parse(uri));
    startActivityForResult(intent,requestCode);
  } else {
    // Notify user to download the latest version of Vipps application.
  }
} catch (PackageManager.NameNotFoundException e) {
  // No Vipps app! Open play store page.
  String url = " https://play.google.com/store/apps/details?id=no.dnb.vipps";
  Intent storeIntent = new Intent(Intent.ACTION_VIEW);
  storeIntent.setData(Uri.parse(url));
  startActivity(storeIntent);
}
```


#### Switching back to the merchant app from Vipps app

Once the user has paid (or cancelled), Vipps supports two ways to return to the merchant native app:

  1: Let Vipps deeplink back into the merchant app using their url scheme (eg. merchantapp://). This is the default/suggested approach.

  2: Just close Vipps, fall back to the merchant app, pick up the thread again there in onActivityResult().

In both cases, the merchant app should query the ecom API for updated status on the payment once user returns from Vipps.

#### Return back to merchant app by actively deeplinking into it from Vipps

With this approach, the merchant app has to have its own URL scheme registered so Vipps can actively open the merchant app again after payment/cancellation.

In the example below, `MainActivity` is the receiving activity and Vipps opens it once the payment is done.

To receive a call back from Vippslication to an activity, a filter has to be set for that activity. In the merchant app, set a filter in the Manifest file:

```xml
<activity android:name=".MainActivity" android:label="@string/app_name" android:launchMode="singleInstance">
  <intent-filter>
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="sampleApps" />
  </intent-filter>
</activity>
```

**Please note:** The scheme should be same specified in `fallBack` URL sent to the ecom api when the payment is created.

The Vipps application will send the result to the merchant app by
starting a new activity with the `fallBack` URL as a URL parameter in the intent.

The merchant app can make their receiving activity a `singleInstance`
to handle the response in same activity.

The receiving MainActivity has to override the `onNewIntent` method to handle
result send by Vipps:

```java
@Override
protected void onNewIntent(Intent intent) {
      // Call the eCom api, check the status of the eCom payment
}
```

#### Redirect back to merchant app by simply closing Vipps

With this approach, the merchant app does not have to register/handle deeplink urls.

In order to use this approach, when creating the payment in the merchant has to pass fallback attribute like this:

```"fallBack": "INTENT" ```

(and *only* "INTENT", no parameters etc.)

This will cause Vipps to simply close after a successful or canceled ecom payment, and fall back to the calling merchant app.

The merchant app activity that resumes again (after Vipps closes) has to override onActivityResult method to pick up the thread again here. Example:

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    // Call the eCom api, check the status of the eCom payment
}
```


## Errors

### Error object in the response

See [HTTP response codes](#http-response-codes).

```json
[
  {
    "errorGroup": "Payment",
    "errorMessage": "Refused by issuer because of expired card",
    "errorCode": "44"
 }
]
```

```json
{
    "statusCode": 401,
    "message": "Access denied due to invalid subscription key. Make sure to provide a valid key for an active subscription."
}
```

## Error groups

| Error groups      | Description |
| ----------------- | ----------- |
| Authentication    | Authentication Failure because of wrong credentials provided  |
| Payments   |  Failure while doing a payment authorization |
| InvalidRequest  |  Request contains invalid parameters |
| VippsError  |  Internal Vipps application error |
| User  | Error related to the Vipps user (Example: Not a Vipps user)  |
| Merchant   | Errors regarding the merchant  |

## Error codes

Please note that Vipps is only allowed to provide some of these errors through
the API, and that we have to send `VippsError` (99) in cases where we are not
allowed to provide more details.

| Error group | Error Code | Error Message | Comment |
| ----- | ---- | ----------- |------|
| Payments | 41 | User do not have a valid card ||
| Payments | 42 | Refused by issuer bank ||
| Payments | 43 | Refused by issuer bank because of invalid a amount ||
| Payments | 44 | Refused by issuer because of expired card ||
| Payments | 45 | Reservation failed for some unknown reason ||
| Payments | 51 | Cannot cancel already captured order ||
| Payments | 52 | Cancellation failed ||
| Payments | 53 | Cannot cancel order which is not reserved yet ||
| Payments | 61 | Captured amount exceeds the reserved amount ordered ||
| Payments | 62 | The amount you tried to capture is not reserved ||
| Payments | 63 | Capture failed for some unknown reason, please use Get Payment Details API to know the exact status ||
| Payments | 71 | Cannot refund more than captured amount ||
| Payments | 72 | Cannot refund for reserved order, please use Cancel API ||
| Payments | 73 | Cannot refund on cancelled order ||
| Payments | 93 | Captured amount should be same in Idempotent retry ||
| Payments | 1501 | This person can not pay to companies |Used for Vipps users under the age of 15.|
| InvalidRequest  | Field name will be the error code | Description about what exactly the field error is ||
| VippsError | 91 | Transaction is not allowed ||
| VippsError | 92 | Transaction already processed ||
| VippsError | 94 | Order locked and is already processing ||
| VippsError | 98 | Too many concurrent requests ||
| VippsError | 99 | Description of the internal error ||
| user | 81 | User not registered with Vipps ||
| user | 82 | User App version not supported ||
| Merchant | 31 | Merchant is blocked because of <reason> ||
| Merchant | 32 | Receiving limit of merchant is exceeded ||
| Merchant | 33 | Number of payment requests has been exceeded ||
| Merchant | 34 | Unique constraint violation of the order id ||
| Merchant | 35 | Registered order not found ||
| Merchant | 36 | Merchant agreement not signed ||
| Merchant | 37 | Merchant not available, deactivated or blocked ||
| Merchant | 38 | Sale unit is not allowed to skip the landing page ||
| Merchant | 21 | Reference Order ID is not valid ||
| Merchant | 22 | Reference Order ID is not in valid state ||


# Testing

To facilitate automated testing in the Vipps test environment the Vipps eCom API
provides a Force Approve endpoint to avoid manual intervention required by the
Vipps App.

The force approve endpoint ([`POST:/ecomm/v2/integration-test/payments/{orderId}/approve`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/integrationTestApprovePayment))
allows developers to approve a payment through the Vipps
eCom API without the use of Vipps. This is useful for automated testing.
The endpoint is only available in our test environment.

Express checkout and `skipLandingPage`is not supported by the force approve endpoint.

# Recommendations regarding handling redirects

Since Vipps is a native app and not a website used in a web browser, the level
of control Vipps has over the redirect back to the merchant after a completed
purchase is limited. The merchant _must not_ assume that Vipps will redirect to
the exact same session or for example rely entirely on cookies in order to
handle the redirect event. The redirect may send the user to a different web browser.

Examples of some, but not all, factors outside of Vipps control:
- Configurations set by the OS itself, for example the default browser.
- User configurations of browsers.
- Users closing Vipps immediately upon purchase.

Because of this Vipps recommends a stateless approach on the webssite that
is supposed to be the end session. An example would a polling-based result
handling from a value in the redirect URL.

Example for demonstration purposes that should be handled.
1. User starts is in web session in the Chrome web browser.
2. A Vipps purchase is started, with a redirect URL specified by the Merchant.
3. The user completes the purchase in Vipps.
4. Vipps (the app) redirects the user by opening the URL specified in step 2.
5. The OS opens the URL in the default browser: Safari (not Chrome).
6. The merchant handles the redirect without the customer noticing any
   discrepancies from the browser switch.

# Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-ecom-api/issues),
a [pull request](https://github.com/vippsas/vipps-ecom-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).

Sign up for our [Technical newsletter for developers](https://github.com/vippsas/vipps-developers/tree/master/newsletters).
