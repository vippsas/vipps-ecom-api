<!-- START_METADATA
---
title: eCom API guide
sidebar_label: API guide
sidebar_position: 15
description: Find technical details about integrating with the eCom API.
pagination_prev: Null
pagination_next: Null
---
END_METADATA -->

# API guide

The Vipps eCom API is used by
[Vipps på nett (*Vipps Online*)](https://vipps.no/produkter-og-tjenester/bedrift/ta-betalt-paa-nett/ta-betalt-paa-nett/),
[Vipps Checkout](https://vipps.no/produkter-og-tjenester/bedrift/ta-betalt-paa-nett/vipps-checkout/),
[Vipps i kassa (*Vipps in Store*)](https://vipps.no/produkter-og-tjenester/bedrift/ta-betalt-i-butikk/vipps-i-kassa/),
native apps and other solutions.

API version: 2.0.0.

<!-- START_COMMENT -->

ℹ️ Please use the website:
[Vipps MobilePay Technical Documentation](https://developer.vippsmobilepay.com/docs/APIs/ecom-api).

<!-- END_COMMENT -->

## Payment flows

There are many ways to use the Vipps eCom API. For example:

* *Vipps Online (aka Vipps på nett)* - The customer selects Vipps as the method
  of payment and enters their mobile number into the app or website. The merchant
  initiates the payment command. The customer confirms the purchase through their
  Vipps app. If the purchase is approved, the merchant registers the sale in
  their system. See
  [eCommerce API: How It Works](./how-it-works/vipps-ecom-api-howitworks.md) for more information.
* *Vipps Express Checkout (aka Vipps Hurtigkasse)* - The same as *Vipps Online*
  except that the shipping address and package delivery options that are selected
  in the Vipps app.
* *Vipps Checkout* - The same as *Vipps Online* except that, instead of
  confirming the purchase in the Vipps app, the customer consents to sharing
  their personal information (e.g., address, phone, and payment information).
  They then complete the purchase from the website where their information is
  automatically pre-filled.
* *Vipps In Store (aka Vipps i kassa)* - The same as *Vipps Online* except that
  the phone number is provided to the service provider who then initiates the
  payment through their Point of Sale (POS) system. See
  [eCommerce API: How it works in the store](./how-it-works/vipps-in-store-howitworks.md) for more information.

This diagram shows a simplified payment flow:

``` mermaid
stateDiagram
    direction LR
    [*] --> Initiate
    Initiate --> Reserve: User confirms
    Reserve --> Capture: Merchant captures
    Capture --> [*]
    Capture --> Refund: Merchant refunds
    Refund --> [*]
    Reserve --> Cancel: Merchant cancels
    Initiate --> Cancel : User cancels, or timeout
    Cancel --> [*]
```

See [Get payment details](#get-payment-details) for more information about
the detailed flow and [Payment states](#payment-states) for the corresponding
states.

The flow of settlements and how to retrieve them are described in
[Settlements](https://developer.vippsmobilepay.com/docs/vipps-developers/settlements).

## Quick start

The normal "happy day" flow for a payment is documented in the
[Quick start guide](vipps-ecom-api-quick-start.md).

This API guide is extensive: We have done our best to document everything about
this API, and you *should* have all information needed to integrate with Vipps.

## API endpoints

The Vipps eCommerce API (eCom API) offers functionality for online payments.
Payments are supported in both web browsers and in native apps (via deep-linking).

| Operation                                                                                                            | Description                                                                                                | Endpoint |
|:---------------------------------------------------------------------------------------------------------------------|:-----------------------------------------------------------------------------------------------------------|:-|
| [Initiate payment](#initiate)                                                                                        | Payment initiation, the first request in the payment flow. This *reserves* an amount.                      | [`POST:/ecomm/v2/payments`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/initiatePaymentV3UsingPOST) |
| [Capture payment](#capture)                                                                                          | When an amount has been reserved, and the goods are (about to be) shipped, the payment must be *captured*. | [`POST:/ecomm/v2/payments/{orderId}/capture`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/capturePaymentUsingPOST) |
| [Cancel payment](#cancel)                                                                                            | The merchant may cancel a reserved amount, but not on a captured amount.                                   | [`PUT:/ecomm/v2/payments/{orderId}/cancel`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/cancelPaymentRequestUsingPUT) |
| [Refund payment](#refund)                                                                                            | The merchant may refund a captured amount.                                                                 | [`POST:/ecomm/v2/payments/{orderId}/refund`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/refundPaymentUsingPOST) |
| [Get payment details](#get-payment-details)                                                                          | The full history of the payment.                                                                           | [`GET:/ecomm/v2/payments/{orderId}/details`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/getPaymentDetailsUsingGET) |
| Get order status                                                                                                     | Deprecated, use [Get payment details](#get-payment-details).                                               | Deprecated, use [`GET:/ecomm/v2/payments/{orderId}/details`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/getPaymentDetailsUsingGET) |


Necessary endpoint from other APIs:

| Operation | Description | Endpoint |
|:----------|:------------|:---------|
| [Get Access token](https://developer.vippsmobilepay.com/api/access-token#tag/Authorization-Service/operation/fetchAuthorizationTokenUsingPost) | Fetch the access token | [`POST:/accesstoken/get`](https://developer.vippsmobilepay.com/api/access-token#tag/Authorization-Service/operation/fetchAuthorizationTokenUsingPost) |

See the
[eCom API checklist](vipps-ecom-api-checklist.md).

## Vipps HTTP headers

We recommend using the standard Vipps HTTP headers for all requests.

See [Vipps HTTP headers](https://developer.vippsmobilepay.com/docs/vipps-developers/common-topics/http-headers)
in the Common topics guide, for details.

## Authentication

All Vipps API calls are authenticated with an access token and an API subscription key.
See
[Get an access token](https://developer.vippsmobilepay.com/docs/APIs/access-token-api#get-an-access-token)
in the Getting started guide, for details.

## Initiate

Payment amounts must be in NOK, be non-zero *and* larger than 1 NOK (1 NOK = 100 øre).

Amounts are specified in minor units.
For Norwegian kroner (NOK) that means 1 kr = 100 øre.
Example: 499 kr = 49900 øre.

When you initiate a payment, it will normally only be *reserved* until you capture it.

This has some benefits:

* If a payment has been _reserved_, the merchant can
  make a
  [`PUT:/ecomm/v2/payments/acme-shop-123-order123abc/cancel`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/cancelPaymentRequestUsingPUT)
  call to immediately release the reservation.
* It is possible to reserve a higher amount and only
  capture a part of it (useful for electric car charging stations, etc).
  It is also possible to capture the full amount
  with multiple captures ("partial capture").

See:

* [Reserve and capture](https://developer.vippsmobilepay.com/docs/vipps-developers/common-topics/reserve-and-capture)
* [When should I charge the customer](https://developer.vippsmobilepay.com/docs/vipps-developers/faqs/reserve-and-capture-faq#when-should-i-charge-the-customer).
* [What is the difference between "Reserve Capture" and "Direct Capture"?](https://developer.vippsmobilepay.com/docs/vipps-developers/faqs/reserve-and-capture-faq#what-is-the-difference-between-reserve-capture-and-direct-capture)
* [When should I use "Direct Capture"?](https://developer.vippsmobilepay.com/docs/vipps-developers/faqs/reserve-and-capture-faq#when-should-i-use-direct-capture)

### Regular payments and express payments

Vipps eCommerce API offers two types of payments:

1. Regular eCommerce payments
2. Express checkout payments

Examples from a demo website:

![Regular and express checkout](images/vipps-flow-web.png)

### Regular eCommerce payments

This is the typical flow, where the user adds items to a shopping cart,
enters the shipping address and pays.

See:

* [How it works](./how-it-works/vipps-ecom-api-howitworks)
* [How it works in the store](./how-it-works/vipps-in-store-howitworks)

### Express checkout payments

The Express checkout (Vipps Hurtigkasse) is a solution for letting the user
automatically share the address information with merchant and choose a shipping option.

Express checkout is designed for shipping products, with a delivery address and a
shipping method.

**Please note:** If you only need the user's information, you should use
[Userinfo](#userinfo).
You should avoid asking the customer in a pub for the shipping method for the drinks, etc.

Vipps Hurtigkasse works this way, as seen from the user's side:

1. The user clicks the "Vipps Hurtigkasse" button.
2. The user consents to sharing address information in Vipps.
3. The user confirms the amount, delivery address and delivery method in Vipps.

To perform an express checkout, the merchant needs to specify
`"paymentType": "eComm Express Payment"` in the
[`POST:/ecomm/v2/payments`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/initiatePaymentV3UsingPOST)
call, and support the
[`POST:[shippingDetailsPrefix]​/v2​/payments​/{orderId}​/shippingDetails`](https://developer.vippsmobilepay.com/api/ecom#tag/Merchant-Endpoints/operation/fetchShippingCostUsingPOST)
and
[`DELETE:/v2/consents/{userId}`](https://developer.vippsmobilepay.com/api/ecom#tag/Merchant-Endpoints/operation/removeUserConsentUsingDELETE)
endpoints.

#### Shipping and static shipping details

The shipping methods presented to the user in Vipps are fetched from the merchant's
[`POST:[shippingDetailsPrefix]​/v2​/payments​/{orderId}​/shippingDetails`](https://developer.vippsmobilepay.com/api/ecom#tag/Merchant-Endpoints/operation/fetchShippingCostUsingPOST)
endpoint.

If the shipping methods and cost can be known in advance, the `staticShippingDetails` field in
[`POST:​/ecomm​/v2​/payments`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/initiatePaymentV3UsingPOST)
may be used to provide the shipping details up front, and thereby avoid an
extra round-trip between the Vipps backend and the merchant's server.

When using `staticShippingDetails` the shipping costs for the available
shipping methods are then sent directly, eliminating the need for the user to
first select shipping method and then for the merchant to calculate the cost for it.

We recommend using `staticShippingDetails` if possible, as it gives a faster
payment process and a better user experience. It also eliminates timeout problems
caused by delays in the merchant's or shipping partner's calculations of cost.

#### Consent and GDPR

Vipps complies with GDPR, and requires the user's consent before any information
is shared with the merchant. The merchant must provide a URL (`consentRemovalPrefix`)
that Vipps can call to delete the data. Vipps allows the user to later
remove this consent (via the Profile -> Security -> "Access to your information"
-> "Companies that remember you" screens).

### Phone and mobile browser flow

A payment is initiated with a call to
[`POST:/ecomm/v2/payments`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/initiatePaymentV3UsingPOST).

![Push notification](images/vipps-flow-device.png)

Triggered by the payment initiation, the Vipps landing page will automatically
detect if is being invoked on a phone, and whether Vipps is installed on the phone.
If Vipps is installed, Vipps will automatically be opened.

#### Vipps installed

1. Vipps is invoked (with app-switch).
2. The user accepts or rejects the payment request in Vipps.
3. The Vipps backend makes a call to the merchant's `callbackPrefix` with
   information about the payment.
4. Once payment process is completed, Vipps redirects to the
   `fallBack` URL that merchant provided earlier (see above).

#### Vipps not installed

1. The landing page (in the browser) prompts the user for the phone number.
2. The Vipps backend sends a push notification to the user's phone,
   and also displays a notification on the landing page for the user to
   continue the payment in Vipps on the phone.
3. The user accepts or rejects the payment in Vipps.
4. The Vipps backend makes a call to the merchant's `callbackPrefix` with
   information about the payment.
5. Once the payment process is completed, Vipps redirects to the
   `fallBack` URL that the merchant provided earlier.

**Please note:**

1. Vipps cannot guarantee that the user will get to the
   `fallBack` URL, since the user may switch away from Vipps or "kill" the app,
   or there may be network or battery problems, etc. before the URL is opened.
   If the merchant relies 100 % on users visiting the `fallback` URL, there will
   be problems: The user has completed the payment, but the merchant ignores it.
2. If the user has started the payment in an embedded browser, such as in
   Facebook or Instagram, it is not possible for Vipps to open the
   `fallBack` URL in the embedded browser. The phone OS will always open URLs
   in the default browser. See:
   [Recommendations regarding handling redirects](#recommendations-regarding-handling-redirects).
3. Because of the above, a successful payment *must not* rely on session cookies
   in the browser.
4. Vipps cannot guarantee a particular sequence of callback and fallback,
   as this depends on user actions, network connectivity/speed, etc.
   Because of this, it is not possible to base an integration on a specific
   sequence of events.
5. URLs must be valid. See:
   [URL validation](#url-validation)

### Desktop flow

![Desktop landing page](images/vipps-ecom-screenshot-landing-page.png)

1. The landing page will be opened in the desktop browser.
2. The landing page will prompt for the user’s phone number.
   If the phone number is known, it should be pre-filled by the merchant.
3. Vipps sends a push notification the user's phone, with a
   notification on the landing page to continue the payment in Vipps on the phone.
4. The user accepts or rejects the payment in Vipps.
5. The Vipps backend makes a call to the merchant's `callbackPrefix` with
   information about the payment. See:
   [Callbacks](#callbacks).
6. Once the payment process is completed, the landing page will redirect to the
   `fallBack` URL that merchant provided earlier (see above).

### Payments initiated in an app

If payments are always initiated in the merchant's native app, there
is no need to pass any additional parameters. Vipps will handle everything
automatically.

It is possible to send the optional `isApp` parameter, which comes with some
additional responsibility.

See:

* [The Vipps deeplink URL](#the-vipps-deeplink-url)
* [IsApp](#isapp)

### Initiate payment flow: API calls

API Specification:
[`POST:/ecomm/v2/payments`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/initiatePaymentV3UsingPOST)

A minimal example:

```json
{
  "customerInfo": {},
  "merchantInfo": {
    "merchantSerialNumber": "123456",
    "callbackPrefix": "https://example.com/vipps/callbacks-for-payment-update-from-vipps",
    "fallBack": "https://example.com/vipps/fallback-result-page-for-both-success-and-failure/acme-shop-123-order123abc"
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
    "callbackPrefix": "https://example.com/vipps/callbacks-for-payment-update-from-vipps",
    "consentRemovalPrefix": "https://example.com/vipps/consents/",
    "fallBack": "https://example.com/vipps/fallback-result-page-for-both-success-and-failure/acme-shop-123-order123abc",
    "merchantSerialNumber": 123456,
    "shippingDetailsPrefix": "https://example.com/vipps/shipping/",
    "paymentType": "eComm Express Payment",
    "staticShippingDetails": [
      {
        "isDefault": "N",
        "priority": 1,
        "shippingCost": 99,
        "shippingMethod": "Walking",
        "shippingMethodId": "123abc"
      },
      {
        "isDefault": "Y",
        "priority": 2,
        "shippingCost": 199,
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

### The Vipps deeplink URL

Vipps responds to the
[`POST:/ecomm/v2/payments`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/initiatePaymentV3UsingPOST)
request with an URL.
The URL is normally a `https://` URL, which automatically opens Vipps if the
apps is installed (and the Vipps landing page if not).

#### isApp

See
[isApp](https://developer.vippsmobilepay.com/docs/vipps-developers/common-topics/isApp)
in Common topics.


### Payment identification

A payment is uniquely identified by the combination of `merchantSerialNumber`
and `orderId`:

* `merchantSerialNumber`: The merchant's Vipps id. Example: `123456`.
* `orderId`: Must be unique for the `merchantSerialNumber`. Example: `acme-shop-123-order123abc`.
  See: [orderId recommendations](https://developer.vippsmobilepay.com/docs/vipps-developers/common-topics/orderid).

### Payment retries

If a user cancels or does not act on a payment, there is no way to "retry"
the payment.

The initiate call is not idempotent, so the closest to a "retry"
is to make a new initiate call with a new `orderId`. Vipps has no concept
of relation between orders, so the "retry" payment is in no way connected
to the first payment attempt.

### OrderId recommendations

See
[OrderId recommendations](https://developer.vippsmobilepay.com/docs/vipps-developers/common-topics/orderid)
in Common topics.

### transactionText recommendations

See
[transactionText recommendations](https://developer.vippsmobilepay.com/docs/vipps-developers/common-topics/transactiontext)
in Common topics.

### URL Validation

See
[URL Validation](https://developer.vippsmobilepay.com/docs/vipps-developers/common-topics/url-validation)
in Common topics.

## Callbacks

Callbacks allow Vipps to send the merchant information about the payment.
Callback are made for the following events:

* Payment successful
* Payment failed
* Payment rejected
* Payment timed out

**Please note:** Callback offer a faster user experience than polling, but you
cannot rely on callbacks alone. You must also poll
[`GET:/ecomm/v2/payments/{orderId}/details`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/getPaymentDetailsUsingGET)
as described in the
[Polling guidelines](https://developer.vippsmobilepay.com/docs/vipps-developers/common-topics/polling-guidelines) in the Common topics.

**Important:**

* Vipps makes *one* attempt at a callback and cannot guarantee that it succeeds,
  as it depends on network, firewalls and other factors that Vipps cannot control.
  If the communication is broken during the process for some reason and Vipps
  is not able to execute callback to the merchant's server, the callback will
  not be retried.
* Vipps callbacks require a response within 3 seconds; otherwise, they will time out.
* Callback URLs must use HTTPS.

The callback request body is different for the different payment types:

* `"eComm Regular Payment"`: The callback contains the order and transaction details.
* `"eComm Express Payment"`: The callback contains the order and transaction details
  *and in addition* the user details and shipping details.
* If [Userinfo](#userinfo) is used: The callback is the same as for `"eComm Express Payment"`.

See:

* [Callback examples](#callback-examples).
* [FAQ: Why do I not get callbacks from Vipps?](vipps-ecom-api-faq.md#why-do-i-not-get-callbacks-from-vipps)

### Callback URLs

The callback URLs are generated by appending the `orderId` to the `callbackPrefix`
sent in the
[`POST:/ecomm/v2/payments/{orderId}`](https://developer.vippsmobilepay.com/api/ecom#tag/Merchant-Endpoints/operation/transactionUpdateCallbackForRegularPaymentUsingPOST)
call. The `callbackPrefix` is for an API endpoint on the merchant's side.
See the [API specifications](https://developer.vippsmobilepay.com/api/ecom#tag/Merchant-Endpoints).

For example, if your `callbackPrefix` is `https://example.com/vipps/callback` and the
`orderId` is `acme-shop-123-order123abc`, Vipps will append `/v2/payments/acme-shop-123-order123abc`
(note the leading slash)
to `https://example.com/vipps/callback` and make a callback to
`https://example.com/vipps/callback/v2/payments/acme-shop-123-order123abc`.

Again:

* `callbackPrefix` + `/v2/payments/` + `{orderId}`
* `https://example.com/vipps/callback` + `/v2/payments/` + `acme-shop-123-order123abc`

**Important:**

* URLs must be [valid](#url-validation).
* Vipps does *not* support sending requests to all ports, so to be safe use
  common port numbers such as: 80, 443, 8080.
* Vipps does *not* support callback URLs that return `HTTP 301 Redirect`,
  `HTTP 302 Permanently Moved` or `HTTP 307 Temporary Redirect`.
  Vipps will not follow to a `Location`.
  The callback URLs *must* be directly reachable.

### Callback examples

Below are two examples of payment update callbacks with
[`POST:[callbackPrefix]/v2/payments/{orderId}`](https://developer.vippsmobilepay.com/api/ecom#tag/Merchant-Endpoints/operation/transactionUpdateCallbackForRegularPaymentUsingPOST).

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
    "shippingCost": 99,
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
`RESERVE`. We apologize for this, but it is technical debt and it cannot be
corrected in this version of the API, as that would break backwards compatibility.

### How to test your own callbacks

We recommend using
[HTTPie](https://httpie.io)
for testing your callbacks.

1. Create a JSON file with the callback request payload based on the example for
   [`POST:[callbackPrefix]/v2/payments/{orderId}`](https://developer.vippsmobilepay.com/api/ecom#tag/Merchant-Endpoints/operation/transactionUpdateCallbackForRegularPaymentUsingPOST):

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

   Save this JSON file as `callback.json`.
2. Build your callback URL. In this example, the `callbackPrefix` is `https://example.com/vipps/callback`,
   and the `orderId` is `acme-shop-123-order123abc`.
3. Test the callback by making a POST request to your callback URL with the JSON file as request body:

   ```http
   http POST https://example.com/vipps/callback/v2/payments/acme-shop-123-order123abc < callback.json
   ```

4. If it does not return `HTTP 200 OK`, your callback handling does not work,
   and you must correct it.

We recommend checking callbacks on the
[API Dashboard](https://developer.vippsmobilepay.com/docs/vipps-developers/developer-resources/api-dashboard).

### Authorization for callbacks

If you provide the `authToken` when initiating a payment with
[`POST:/ecomm/v2/payments/{orderId}`](https://developer.vippsmobilepay.com/api/ecom#tag/Merchant-Endpoints/operation/transactionUpdateCallbackForRegularPaymentUsingPOST),
the `authToken` will be sent as an `Authorization` header in the callback and
shipping details requests. Please note that this is unrelated to the
[authentication](#authentication)
required by the Vipps API.

API spec:
[`POST:[callbackPrefix]/v2/payments/{orderId}`](https://developer.vippsmobilepay.com/api/ecom#tag/Merchant-Endpoints/operation/transactionUpdateCallbackForRegularPaymentUsingPOST)

### Vipps callback servers

The callbacks from Vipps are made from the servers described in
[Vipps request servers](https://developer.vippsmobilepay.com/docs/vipps-developers/developer-resources/servers#vipps-request-servers).

Please make sure that requests from these servers are allowed through firewalls, etc.

### Callback statuses

These are the statuses provided by Vipps in the callbacks by
[`[callbackPrefix]/v2/payments/{orderId}`](https://developer.vippsmobilepay.com/api/ecom#tag/Merchant-Endpoints/operation/transactionUpdateCallbackForRegularPaymentUsingPOST):

| Callback type    | Response         | Description                            |
|:-----------------|:-----------------|:---------------------------------------|
| Regular checkout | `RESERVED`       | Payment reserved by user accepting the payment in Vipps. |
|                  | `SALE`           | Payment captured with direct capture by merchant (after `RESERVED`). |
|                  | `RESERVE_FAILED` | Reserve capture failed because of insufficient funds, invalid card or similar. |
|                  | `SALE_FAILED`    | Direct capture failed because of insufficient funds, invalid card or similar. |
|                  | `CANCELLED`      | Payment cancelled by the user in Vipps. |
|                  | `REJECTED`       | User did not act on the payment (timeout, etc.). |
| Express checkout | `RESERVE`        | Payment reserved by user accepting the payment in Vipps (it *is* correct that this is different from `RESERVED` for regular checkout - sorry.) |
|                  | `SALE`           | Payment captured with direct capture, by merchant (after `RESERVE`). |
|                  | `RESERVE_FAILED` | Reserve failed because of insufficient funds, invalid card or similar. |
|                  | `SALE_FAILED`    | Direct failed because of insufficient funds, invalid card or similar. |
|                  | `CANCELLED`      | Payment cancelled by user in Vipps.    |
|                  | `REJECTED`       | User did not act on the payment (timeout, etc.). |

For a full history of a payment, use
[`[callbackPrefix]/v2/payments/{orderId}/details`]( https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/getPaymentDetailsUsingGET). See [Get payment details](#get-payment-details) for more information.

## Timeouts

See: [Timeouts](https://developer.vippsmobilepay.com/docs/vipps-developers/common-topics/timeouts) in Common topics.

## Express checkout API endpoints required on the merchant side

The below endpoints are provided by the *merchant* and consumed by Vipps during
express checkout payments. These endpoints are not required when using
regular checkout.

These endpoints are to be implemented by merchants in order for Vipps
to make calls to them. The documentation is included in the API specification for
reference only - these endpoints are *not* callable at Vipps.

| Operation            | Description                                                                       | Endpoint |
|:---------------------|:----------------------------------------------------------------------------------|:-|
| Get shipping details | Used to fetch shipping information, **10 second timeout** | [`POST:/v2/payments/{orderId}/shippingDetails`](https://developer.vippsmobilepay.com/api/ecom#tag/Merchant-Endpoints/operation/fetchShippingCostUsingPOST) |
| Transaction Update   | A callback to the merchant for receiving post-payment information.                | [`POST:/v2/payments/{orderId}`](https://developer.vippsmobilepay.com/api/ecom#tag/Merchant-Endpoints/operation/transactionUpdateCallbackForRegularPaymentUsingPOST) |
| Remove user consent  | Used to inform merchant when the Vipps user removes consent to share information. | [`DELETE:/v2/consents/{userId}`](https://developer.vippsmobilepay.com/api/ecom#tag/Merchant-Endpoints/operation/removeUserConsentUsingDELETE) |

Please note that if the shipping details are static (i.e., do not vary based on the
address), the parameter `staticShippingDetails` can be used in the initiate call.
Then, there is no need to implement
[`POST:/v2/payments/{orderId}/shippingDetails`](https://developer.vippsmobilepay.com/api/ecom#tag/Merchant-Endpoints/operation/fetchShippingCostUsingPOST).

See
[Initiate payment flow: API calls](#initiate-payment-flow-api-calls) for details.

### Get shipping details

The
[`POST:[shippingDetailsPrefix]/v2/payments/{orderId}/shippingDetails`](https://developer.vippsmobilepay.com/api/ecom#tag/Merchant-Endpoints/operation/fetchShippingCostUsingPOST)
API endpoint _on the merchant's side_ allows Vipps to get the shipping cost and
method based on the provided address and product details.
This endpoint is required for express checkout.

Request:

```json
{
  "addressId": 3960,
  "addressLine1": "Dronning Eufemias gate 42",
  "addressLine2": null,
  "country": "Norway",
  "city": "OSLO",
  "postCode": "0191",
  "addressType": "H"
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
      "shippingCost": 99,
      "shippingMethod": "Walking",
      "shippingMethodId": "123abc"
    },
    {
      "isDefault": "Y",
      "priority": 2,
      "shippingCost": 199,
      "shippingMethod": "Running",
      "shippingMethodId": "321abc"
    }
  ]
}
```

### Payment update

See
[Callbacks](#callbacks).

### Remove User Consent

The
[`DELETE:[consentRemovalPrefix]/v2/consents/{userId}`](https://developer.vippsmobilepay.com/api/ecom#tag/Merchant-Endpoints/operation/removeUserConsentUsingDELETE)
API endpoint _on the merchant's side_ allows Vipps to send an end user's consent
removal request to the merchant.
This endpoint is required for express checkout.

When receiving this request,
the merchant is obliged to handle the user details as per the GDPR guidelines.
The request path will include a `userId` that Vipps will have provided as
part of callback and also made accessible through [`GET:/ecomm/v2/payments/{orderId}/details`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/getPaymentDetailsUsingGET).

## The Vipps landing page

When a user is directed to the `url` sent in the response to
[`POST:/ecomm/v2/payments`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/initiatePaymentV3UsingPOST),
they will either be taken to Vipps or to the Vipps landing page.

See
[Vipps landing page](https://developer.vippsmobilepay.com/docs/vipps-developers/common-topics/vipps-landing-page)
from Common topics, for mre details about the landing page.

## Reserve

When the user confirms the purchase in Vipps, the payment status changes to `RESERVE`.
The respective amount will be reserved for future capturing.

For more details, see
[Common topics: Reserve and capture](https://developer.vippsmobilepay.com/docs/vipps-developers/common-topics/reserve-and-capture).

## Capture

Capture is done with
[`POST:/ecomm/v2/payments/{orderId}/capture`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/capturePaymentUsingPOST).

Use the idempotency key, `X-Request-Id`, in the capture call. Then, if a capture
request fails for any reason, it can be retried with the same idempotency key.
You can use any unique id for your `X-Request-Id`.

See the [X-Request-Id in the capturePaymentUsingPOST specification](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/capturePaymentUsingPOST) for details.

To perform a normal capture of the entire amount, `amount` can be
omitted from the API request (i.e., not sent at all), set to `null` or set to `0`.
When doing a
[partial capture](https://developer.vippsmobilepay.com/docs/vipps-developers/common-topics/reserve-and-capture#partial-capture),
you need to specify the `amount`.

**Please note:** It is important to check the response of the `/capture`
call. The capture is only successful when the response is `HTTP 200 OK`.

Capture can be made up to 180 days after reservation.
Attempting to capture an older payment will result in a
`HTTP 400 Bad Request`.

See the FAQ:

* [When should I charge the customer](https://developer.vippsmobilepay.com/docs/vipps-developers/faqs/reserve-and-capture-faq#when-should-i-charge-the-customer).
* [What is the difference between "Reserve Capture" and "Direct Capture"?](https://developer.vippsmobilepay.com/docs/vipps-developers/faqs/reserve-and-capture-faq#what-is-the-difference-between-reserve-capture-and-direct-capture)
* [When should I use "Direct Capture"?](https://developer.vippsmobilepay.com/docs/vipps-developers/faqs/reserve-and-capture-faq#when-should-i-use-direct-capture)


See
[Common topics: capture](https://developer.vippsmobilepay.com/docs/vipps-developers/common-topics/reserve-and-capture#capture)
for more details about the types of captures.

## Cancel

The Cancel request allows the merchant to cancel a reserved or initiated transaction.

Please note that it is not possible to cancel a request that is over 180 days old.
Attempting to cancel an older payment will result in a `HTTP 400 Bad Request`.

The payment flow can be aborted under certain circumstances:

* When the user cancels (rejects) the initiated payment in Vipps.
* When the merchant cancels.
* Timeouts: If the user does not confirm, etc.

After cancellation, the order gets a new status:

* If an order is cancelled by the merchant while it's in the `RESERVE` state, the status becomes `VOID`.
* If an order is cancelled by the merchant while it's in the `INITIATE` state, the status becomes `CANCEL`.
* If an order is cancelled by the user, the status becomes `CANCEL`.

Example Request:

[`PUT:/ecomm/v2/payments/acme-shop-123-order123abc/cancel`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/cancelPaymentRequestUsingPUT)

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
[`PUT:/ecomm/v2/payments/{orderId}/cancel`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/cancelPaymentRequestUsingPUT)
request while the transaction is in the `INITIATE` stage.

This may be useful in face-to-face situations where a customer's phone runs out
of battery, or if the customer suddenly changes their mind and wants to buy
more and the amount for the payment increases.
This should not be considered a consistent or guaranteed operation,
as the `/cancel` request depends on actions taken by the user in the app.

If the
[`PUT:/ecomm/v2/payments/{orderId}/cancel`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/cancelPaymentRequestUsingPUT)
request is successful, the payment state in the response will be: `CANCELLED`.  
> **Warning**
> The [`GET:/ecomm/v2/payments/{orderId}/details`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/getPaymentDetailsUsingGET) endpoint however will return `CANCEL`, *not* `CANCELLED`.

A call to
[`GET:/ecomm/v2/payments/{orderId}/details`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/getPaymentDetailsUsingGET)
for the same order will return the following, regardless of whether the
transaction has been reserved before the cancellation: `CANCEL`.
> **Note**
> If the order is in the state `RESERVED` and then cancelled by the merchant, the status will be `VOID`.

**Please note:** If the user is already in a 3-D Secure session, the payment
cannot be cancelled as described above.

### Cancelling a partially captured order

If you wish to cancel an order that you have partially captured, send a
[`PUT:/ecomm/v2/payments/{orderId}/cancel`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/cancelPaymentRequestUsingPUT)
request with `shouldReleaseRemainingFunds: true` in the body.
The payment must be `RESERVED` for this to take effect.

If `shouldReleaseRemainingFunds` is not set, it will default to `false`.

When `shouldReleaseRemainingFunds` is set to `false`,
any request to cancel after a partial or full capture has been performed will be rejected.

This is a useful and recommended feature, as it releases any reserved balance
as soon as the card issuer and/or bank permits.

See also the FAQ:
[How long does it take from a refund is made until the money is in the customer's account?](https://developer.vippsmobilepay.com/docs/vipps-developers/faqs/refunds-faq#how-long-does-it-take-from-a-refund-is-made-until-the-money-is-in-the-customers-account)

Example Request:

```json
{
  "merchantInfo": {
    "merchantSerialNumber": "123456"
  },
  "transaction": {
    "transactionText": "No socks for you!"
  },
  "shouldReleaseRemainingFunds": true
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
    "capturedAmount": 10000,
    "remainingAmountToCapture": 0,
    "refundedAmount": 0,
    "remainingAmountToRefund": 10000
  }
}
```

**Please note:** Once this operation has been performed, there will be zero
funds remaining to capture. Do not call this endpoint until you are sure you
have captured all you need.

## Refund

The merchant can initiate a refund of the captured amount.
The refund can be a partial or full.

Partial refunds are done by specifying an `amount` which is lower than the
captured amount. The refunded amount cannot be larger than the captured amount.

In a capture request, the merchant may also use the `X-Request-Id` header. See [Idempotency](#idempotency) for details.

Refunds can be made up to 365 days after reservation.
Attempting to refund an older payment will result in a
`HTTP 400 Bad Request`.

Example Request:

[`POST:/ecomm/v2/payments/acme-shop-123-order123abc/refund`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/refundPaymentUsingPOST)

```json
{
  "merchantInfo": {
    "merchantSerialNumber": "123456"
  },
  "transaction": {
    "amount": 20000,
    "transactionText": "Refund of Vipps socks"
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

See
[FAQ: Refunds](https://developer.vippsmobilepay.com/docs/vipps-developers/faqs/refunds-faq)
for common questions.

## Get payment details

[`GET:/ecomm/v2/payments/{orderId}/details`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/getPaymentDetailsUsingGET) retrieves the full history of a payment and the status of the operations.

### Payment states

| # | From-state | To-state | Description                                                    | Operation |
|:--|:-----------|:---------|:---------------------------------------------------------------|:-|
| 0 | -          | Initiate | Payment initiation                                             | `INITIATE` |
| 1 |            | -        | The merchant has initiated the payment                         | `INITIATE` |
| - |            |          | The user has accepted the payment and amount has been reserved | `RESERVE` |
| - |            | Cancel   | The user has cancelled the order                               | `CANCEL` |
| 2 | Reserve    | Capture  | The merchant captures the payment and ships                    | `CAPTURE` |
| - |            | Cancel   | The merchant has cancelled the order                           | `VOID` |
| 3 | Capture    | --       | A final state: Payment fully processed                         | `CAPTURE` |
| - |            | Refund   | The merchant has refunded the money to the user                | `REFUND` |
| 4 | Cancel     | --       | A final state: Payment cancelled                               | - |
| 5 | Refund     | --       | A final state: Payment refunded                                | - |

### Requests and responses

Please note that the response from
[`GET:/ecomm/v2/payments/{orderId}/details`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/getPaymentDetailsUsingGET)
always contains *the entire history* of payments for the order, not just the current status.

**Important:** The `operationSuccess` field indicates whether an operation was successful or not.

| Response   | Description                                             |
|:-----------|:--------------------------------------------------------|
| `INITIATE` | Payment initiated by merchant                           |
| `RESERVE`  | Payment reserved by user accepting the payment in Vipps |
| `CAPTURE`  | Payment captured by merchant (after `RESERVE`)          |
| `REFUND`   | Payment refunded by merchant (after `CAPTURE`)          |
| `CANCEL`   | Payment cancelled by user in Vipps                      |
| `SALE`     | Payment captured with direct capture by merchant        |
| `VOID`     | Payment cancelled by merchant                           |

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

**Please note:**
* The `transactionSummary` will not be part of the response if
  the user does not react to the Vipps landing page or app-switch.
* The `bankIdentificationNumber` will only be part of `transactionSummary` in the
  response of the `GET:/ecomm/v2/payments/{orderId}/details` endpoint.
* If paymentType is set to `eComm Express Payment` you will get `shippingDetails`
  and `userDetails` in addition to `transactionLogHistory` and `transactionSummary`.
* If you initiate the payment with a `scope` to use with the
  [Userinfo API](https://developer.vippsmobilepay.com/docs/APIs/userinfo-api)
  you will also get an `userDetails` object containing the user's information:
```
"userDetails": {
  "bankIdVerified": 0,
  "email": "user@example.com",
  "firstName": "Ada",
  "lastName": "Lovelace",
  "mobileNumber": "91234567",
  "userId": "c06c4afe-d9e1-4c5d-939a-177d752a0944"
}
```

### Polling guidelines

See
[Polling guidelines](https://developer.vippsmobilepay.com/docs/vipps-developers/common-topics/polling-guidelines)
in the Common topics.

## Get payment status

**IMPORTANT: This endpoint is deprecated. Use [Get payment details](#get-payment-details).**

## Userinfo

Vipps offers the possibility for merchants to ask for the user's profile information as part of the payment flow.

To enable the possibility to fetch profile information for a user, the merchant can add a `scope`
parameter to the initiate call:
[`POST:/ecomm/v2/payments`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/initiatePaymentV3UsingPOST).
See
[Userinfo API guide](https://developer.vippsmobilepay.com/docs/APIs/userinfo-api).


### Userinfo call-by-call guide

Scenario: You want to complete a payment and get the name and phone number of
a customer. Details about each step are described in the sections below.

1. Retrieve the access token:
   [`POST:/accesstoken/get`](https://developer.vippsmobilepay.com/api/access-token#tag/Authorization-Service/operation/fetchAuthorizationTokenUsingPost).
2. Add scope to the transaction object and include the scope you wish to get
   access to (valid scope) before calling
   [`POST:/ecomm/v2/payments`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/initiatePaymentV3UsingPOST)
   Include the scopes you need access to (e.g., "name address email phoneNumber birthDate"), separated by spaces.
3. The user consents to the information sharing and perform the payment in Vipps.
4. Retrieve the `sub` by calling
   [`GET:/ecomm/v2/payments/{orderId}/details`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/getPaymentDetailsUsingGET)
5. Using the sub from step 4, call
   [`GET:/vipps-userinfo-api/userinfo/{sub}`](https://developer.vippsmobilepay.com/api/userinfo#operation/getUserinfo)
   to retrieve the user's information.

To test this out, see the step-by-step instructions in the
[Quick start](vipps-ecom-api-quick-start.md).

**Please note:** The `sub` is added asynchronously, so if the `/details` request
is made within (milli)seconds of the payment approval in the app, it may not be
available. If that happens, simply make another `/details` request.
See
[Polling guidelines](https://developer.vippsmobilepay.com/docs/vipps-developers/common-topics/polling-guidelines)
in the Common topics for more recommendations.

### Get userinfo

Once the user completes the session, a unique identifier `sub` can be retrieved from the
[`GET:/ecomm/v2/payments/{orderId}/details`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/getPaymentDetailsUsingGET) endpoint.

Example `sub` format:

```json
"sub": "c06c4afe-d9e1-4c5d-939a-177d752a0944",
```

This `sub` is a link between the merchant and the user and can be used to retrieve
the user's details from Vipps userinfo:
[`GET:/vipps-userinfo-api/userinfo/{sub}`](https://developer.vippsmobilepay.com/api/userinfo#operation/getUserinfo)

See:
[Userinfo API](https://developer.vippsmobilepay.com/docs/APIs/userinfo-api).

## HTTP response codes

This API returns the following HTTP statuses in the responses:

- `200 OK`
- `201 Created`
- `204 No Content`
- `400 Bad Request`
- `401 Unauthorized`
- `403 Forbidden`
- `404 Not Found`
- `409 Conflict`
- `429 Too Many Requests`
- `500 Server Error`

For more information about the HTTP response codes and JSON object in the responses, see
[HTTP response codes](https://developer.vippsmobilepay.com/docs/vipps-developers/common-topics/http-response-codes)
in the Common topics.

For details about the specific eCom errors and responses, see:

- [OpenAPI specification](https://developer.vippsmobilepay.com/api/ecom)
- [Errors](#errors)

## Rate limiting

We have added rate limiting to our APIs (HTTP 429 Too Many Requests) to prevent fraudulent and wrongful behavior, and to increase the stability and security of our APIs. The limits should not affect normal behavior, but please contact us if you notice any unexpected behavior.

The "Key" column specifies what we consider to be the unique identifier, and
what we "use to count". The limits are of course not *total* limits.

| API | Limit | Key | Explanation |
| --- | ----- | --- | ----------- |
| [Initiate](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/initiatePaymentV3UsingPOST) | 2 per minute   | orderId + MSN              | Two calls per minute per unique orderId |
| [Cancel](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/cancelPaymentRequestUsingPUT) | 5 per minute   | orderId + MSN              | Five calls per minute per unique orderId |
| [Capture](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/capturePaymentUsingPOST)     | 5 per minute   | orderId + MSN              | Five calls per minute per unique orderId |
| [Refund](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/refundPaymentUsingPOST)       | 5 per minute   | orderId + MSN              | Five calls per minute per unique orderId |
| [~~Status~~](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/getOrderStatusUsingGET)   | 120 per minute | orderId + subscription key | 120 calls per minute per unique orderId |
| [Details](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/getPaymentDetailsUsingGET)   | 120 per minute | orderId + subscription key | 120 calls per minute per unique orderId |

**Please note:** The "Key" column is important. The above means that we allow two
[Initiate](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/initiatePaymentV3UsingPOST)
calls per minute *per unique orderId* for that MSN. This
is to prevent too many initiate calls for the same payment. The overall limit
for number of *different* payments is *far* higher than 2. The same goes for
[Capture](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/capturePaymentUsingPOST):
You can make five capture calls per minute for
one unique orderId, and the limit for capture calls for different orderIds
is *far* higher.

## Partner keys

In addition to the normal [Authentication](#authentication), we offer *partner keys*
which let a partner make API calls on behalf of a merchant.

See:
[Partner keys](https://developer.vippsmobilepay.com/docs/vipps-partner/partner-keys).

## Idempotency

In a capture request, the merchant may also use the `X-Request-Id` header.
This header is an idempotency header ensuring that, if the merchant retries
a request with the same `X-Request-Id`, the retried request will not make
additional changes.

See the
[Idempotency header](https://developer.vippsmobilepay.com/docs/vipps-developers/common-topics/http-headers#idempotency)
for more details.

Example Request:

[`POST:/ecomm/v2/payments/acme-shop-123-order123abc/capture`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/capturePaymentUsingPOST)

```json
{
  "merchantInfo": {
    "merchantSerialNumber": "123456"
  },
  "transaction": {
    "amount": 20000,
    "transactionText": "Socks on the way! Tracking code: abc-tracking-123"
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
[`GET:/ecomm/v2/payments/{orderId}/details`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/getPaymentDetailsUsingGET)
and check status of last transaction in transaction history prior
to executing the service call again.

### Callback aborted or interrupted

If the communication is broken during payment process for some reason,
either because of network problems, that the user abruptly closes the app or
something else, and Vipps is not able to execute callback, the callback will
not be retried.

In other words, if the merchant doesn’t receive any confirmation on payment
request call within callback time frame, merchant must call
[`GET:/ecomm/v2/payments/{orderId}/details`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/getPaymentDetailsUsingGET)
to get the response of payment request.

### PSP connection issues

In cases of communication problems with Vipps' PSP, the response from Vipps
will be an error (see [Errors](#errors)).

The merchant should then call
[`GET:/ecomm/v2/payments/{orderId}/details`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/getPaymentDetailsUsingGET)
to check if the transaction request is processed before making a new call,
using same idempotency key (see [Idempotency](#idempotency)).

### Clean-up strategies

Vipps recommends merchants implement a robust clean up strategy for all orders
where the goods or services are not issued to the user.

An example case would be:

Bank X is having issues and transactions for their customers are slow (15+ seconds)
for a significant period of time. As a result of this the user decides to close
the Vipps app while the transaction is being processed, they do not then see the
final result in the Vipps app. This transaction will eventually result in a
successful reservation. The user then returns to the Merchants app, but as the
entry was not via an expected app-switch from the Vipps app, state is not correctly
resumed. The user attempts a second payment, which this time goes through in a
timely manner, and upon app-switch back the user is issued their goods/services.

The user has now two reservations and only received goods/services once. It is
the merchant's responsibility here to ensure the first order, for which no
goods/services were issued, should be cancelled to remove the reservation from
the customers account.

Vipps recommends polling
[`GET:/ecomm/v2/payments/{orderId}/details`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/getPaymentDetailsUsingGET)
until either a `REJECT`, `USER_CANCEL`, `RESERVE` or `SALE` status is received,
and then performing the appropriate action based on the status of product issuing.
The user should also be notified that the merchant has issued any product to
ensure they do not naturally retry the purchase. This includes using sms/email
strategy if it is unclear that the user in on the merchants website/app to see confirmation.

### Recommendations for handling very high traffic

At peak traffic, like Black Friday, it is especially important to have a
robust integration.

We strongly recommend using "reserve capture", and not "direct capture",
so it is possible to cancel a payment without making a refund.

* Make sure you poll
  [`GET:/ecomm/v2/payments/{orderId}/details`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/getPaymentDetailsUsingGET),
  and do not rely on the callback from Vipps, and also don't rely on the user
  reaching the callback URL.
* Consider if you should cancel or complete the payment if the payment takes
  much longer than normal, so much that the user may give up.
  * If you sell a ticket: It may be good to always issue the ticket if the user
    has approved the payment, even if the processing of the payment takes much
    longer than normal and the user "gives up".
  * If you sell food at a restaurant: If the payment takes so long that the
    user gives up and tries to order again: It may be smart to cancel the
    previous order, even though the user did approve the payment.

It is also important that the merchant's and/or partner's systems are able to
to handle the peak traffic.

See also:

* [Exception handling](#exception-handling)
* [Clean-up strategies](#clean-up-strategies)

## App integration

Merchants may implement deep-linking, to trigger Vipps (we refer to this as "app-switch" here).

This may be done in two ways:

1. From a mobile or desktop browser. See [Desktop flow](#desktop-flow)
2. From an iOS or Android native app

After the user has finished (or cancelled) the payment in Vipps, the user is returned back to the specified `fallBack` URL. When the user arrives back in the merchant app or website, we *strongly* recommend that you perform a call to the [payment details endpoint](#get-payment-details) to check the state
of the transaction. While some of the state of the eCom operation *can* be derived from things like whether or not user returned successfully from the
Vipps app, the most reliable approach to know the state of the payment is always to query the eCom API once the user arrive back in the merchant app/website.

## App-switching

1. The merchant need to pass the URL scheme of the app into the `fallBack` field in the [initiate payment request](#initiate-payment-flow-api-calls). See [App-switch on iOS](#app-switch-on-ios) and [App-switch on Android](#app-switch-on-android) for specifics.
2. The merchant app should open Vipps deeplink received from the [initiate payment request](#initiate-payment-flow-api-calls).
3. Once the operation in Vipps is completed, Vipps will redirect the user to the deeplink specified in the `fallBack` field.
4. The merchant app should query the [payment details endpoint](#get-payment-details) for updated status on the payment once user returns from Vipps.

**Please note:** Vipps will append a status at the end of the fallback URL. For example, if your `fallBack` URL is `merchantApp://result?myAppData`, Vipps
will append the status like: `merchantApp://result?myAppData&status=301` or `merchantApp://result?myAppData&status=100`. **This status is deprecated, and should no longer be used.** Query the [payment details endpoint](#get-payment-details) for latest status instead.

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

```kotlin
package com.example.app

import android.app.Activity
import android.content.Intent
import android.content.pm.PackageManager
import android.net.Uri

class MyActivity : Activity() {
    companion object {
        const val vippsPackageName = "no.dnb.vipps"
        const val vippsRequestCode: Int = TODO("insert your own custom identifier for tracking vipps app intents")
    }

    // use deeplinkUrl from ecom response payload
    fun openVippsApp(deeplinkUrl: String) {
        try {
            val pm = applicationContext.packageManager
            val packageInfo = pm.getPackageInfo(vippsPackageName, PackageManager.GET_ACTIVITIES)
            val intent = Intent(Intent.ACTION_VIEW, Uri.parse(deeplinkUrl))
            startActivityForResult(intent, vippsRequestCode)
        } catch (e: PackageManager.NameNotFoundException) {
            // No Vipps app installed! Open the Vipps page on Google Play
            val playStoreUrl = "https://play.google.com/store/apps/details?id=$vippsPackageName"
            val intent = Intent(Intent.ACTION_VIEW).setData(Uri.parse(playStoreUrl))
            startActivity(intent)
        }
    }
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

To receive a call back from Vipps to an activity, a filter has to be set for that activity. In the merchant app, set a filter in the Manifest file:

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

**Please note:** The scheme should be same specified in `fallBack` URL sent to the eCom API when the payment is created.

The Vipps application will send the result to the merchant app by
starting a new activity with the `fallBack` URL as a URL parameter in the intent.

The merchant app can make their receiving activity a `singleInstance`
to handle the response in same activity.

The receiving MainActivity has to override the `onNewIntent` method to handle
result send by Vipps:

```kotlin
override fun onNewIntent(intent: Intent) {
      // Call the eCom API, check the status of the eCom payment
}
```

#### Redirect back to merchant app by simply closing Vipps

With this approach, the merchant app does not have to register/handle deeplink urls.

In order to use this approach, when creating the payment in the merchant has to pass fallback attribute like this:

`"fallBack": "INTENT"`

(and *only* "INTENT", no parameters etc.)

This will cause Vipps to simply close after a successful or cancelled ecom payment, and fall back to the calling merchant app.

The merchant app activity that resumes again (after Vipps closes) has to override onActivityResult method to pick up the thread again here. Example:

```kotlin
override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent) {
    // Call the eCom API, check the status of the eCom payment
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

### Error groups

| Error groups   | Description                                                  |
|:---------------|:-------------------------------------------------------------|
| Authentication | Authentication Failure because of wrong credentials provided |
| Payment        | Failure while doing a payment authorization                  |
| InvalidRequest | Request contains invalid parameters                          |
| VippsError     | Internal Vipps application error                             |
| User           | Error related to the Vipps user (Example: Not a Vipps user)  |
| Merchant       | Errors regarding the merchant                                |

### Error codes

**Please note:** Vipps is only allowed to provide all of these errors through
the API, and that we have to send `VippsError` (99) in cases where we are not
allowed to provide more details. The same goes for errors from our PSP: We
may not be allowed to pass on the details to merchants.

In case of errors: Use
[Get payment details](#get-payment-details)
to retrieve all the information about the payment.

| Error group    | Error Code | Error Message (and some explaining text for some errors) |
|:---------------|:-----------|:-----------------------------------------------|
| Payment        | 45         | Reservation failed. The payment was not acted upon by the customer. See [Timeouts](https://developer.vippsmobilepay.com/docs/vipps-developers/common-topics/timeouts) |
| Payment        | 51         | Cannot [cancel](https://developer.vippsmobilepay.com/docs/APIs/ecom-api/vipps-ecom-api#cancel) an already captured order. Do a [refund](https://developer.vippsmobilepay.com/docs/APIs/ecom-api/vipps-ecom-api#refund) instead. |
| Payment        | 52         | [Cancel](https://developer.vippsmobilepay.com/docs/APIs/ecom-api/vipps-ecom-api#cancel) failed.                           |
| Payment        | 53         | Cannot [cancel](https://developer.vippsmobilepay.com/docs/APIs/ecom-api/vipps-ecom-api#cancel) an order that is not reserved. The user must first accept the payment. |
  | Payment        | 61         | Captured amount exceeds the reserved amount. You can not capture a higher amount than the user has accepted. Check the [payment details](https://developer.vippsmobilepay.com/docs/APIs/ecom-api/vipps-ecom-api#get-payment-details).|
| Payment        | 62         | The amount you tried to capture is not reserved. The user must first accept the payment. |
| Payment        | 63         | Capture failed for an unknown reason, please use [payment details](https://developer.vippsmobilepay.com/docs/APIs/ecom-api/vipps-ecom-api#get-payment-details) to see the exact status. |
| Payment        | 71         | Cannot refund more than the captured amount.       |
| Payment        | 72         | Cannot refund a reserved order (only captured orders). [Cancel](https://developer.vippsmobilepay.com/docs/APIs/ecom-api/vipps-ecom-api#cancel) instead.|
| Payment        | 73         | Cannot refund a cancelled order.               |
| Payment        | 93         | Captured amount must be the same in an idempotent retry. The same `Idempotency-Key` can not be used with different request payloads. |
| Payment        | 95         | Payments can only be refunded up to 365 days after reservation. See [Reserve and capture](https://developer.vippsmobilepay.com/docs/vipps-developers/common-topics/reserve-and-capture).|
| Payment        | 96         | Payments can only be captured up to 180 days after reservation. See [Reserve and capture](https://developer.vippsmobilepay.com/docs/vipps-developers/common-topics/reserve-and-capture).|
| Payment        | 1082       | This person is not BankID verified. Only applies for test users. |
| VippsError     | 91         | The transaction is not allowed. Typically when attempting to capture a cancelled order. |
| VippsError     | 92         | The transaction is already processed.                 |
| VippsError     | 94         | Order locked and is already processing. This can occur for a short period of time if a bank has problems, and Vipps needs to wait and/or clean up. Retry the same request later. |
| VippsError     | 98         | Too many concurrent requests. Used only to prevent incorrect API use. See [Rate limiting](https://developer.vippsmobilepay.com/docs/APIs/ecom-api/vipps-ecom-api#rate-limiting).|
| VippsError     | 99         | An internal error not documented in this table. The request body should contain a description of the internal error. |
| user           | 81         | User unknown. The phone number is either incorrectly formatted (see the API specification), is not a Vipps user, or the user is under 15 years old and cannot pay businesses. Vipps cannot give more details. This error also occurs if using a non-Norwegian phone number. |
| user           | 82         | The user's app version is not supported.                |
| Merchant       | 31         | The merchant is blocked because of [reason]. See [Why do I get errorCode 37 "Merchant not available or deactivated or blocked"?](https://developer.vippsmobilepay.com/docs/vipps-developers/faqs/common-errors-faq#why-do-i-get-errorcode-37-merchant-not-available-or-deactivated-or-blocked). |
| Merchant       | 32         | The receiving limit of merchant is exceeded.       |
| Merchant       | 33         | The number of allowed payment requests has been exceeded.  |
| Merchant       | 34         | The `orderId` has already been used for another payment for this MSN. The orderId must be unique for the MSN. See [Recommendations for orderId/reference](https://developer.vippsmobilepay.com/docs/vipps-developers/common-topics/orderid). |
| Merchant       | 37         | The merchant is either not available, deactivated or blocked. See [Why do I get errorCode 37 "Merchant not available or deactivated or blocked"?](https://developer.vippsmobilepay.com/docs/vipps-developers/faqs/common-errors-faq#why-do-i-get-errorcode-37-merchant-not-available-or-deactivated-or-blocked) |
| Merchant       | 38         | The sales unit is not allowed to skip the landing page. See the [Vipps landing page](https://developer.vippsmobilepay.com/docs/vipps-developers/faqs/vipps-landing-page-faq). |
| Merchant       | 97         | The merchant is not approved by Vipps to receive payments. See [Why do I get "Merchant Not Allowed for Ecommerce Payment"?](https://developer.vippsmobilepay.com/docs/vipps-developers/faqs/common-errors-faq#why-do-i-get-merchant-not-allowed-for-ecommerce-payment). |
| InvalidRequest | -          | The field name will be the error code. Description about what exactly the field error is. |

## Testing

To facilitate automated testing in
[The Vipps Test Environment (MT)](https://developer.vippsmobilepay.com/docs/vipps-developers/test-environment)
the Vipps eCom API provides a "force approve" endpoint to avoid manual
payment confirmation in the Vipps app:
[`POST:/ecomm/v2/integration-test/payments/{orderId}/approve`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/integrationTestApprovePayment)

The "force approve" endpoint allows developers to approve a payment through the Vipps
eCom API without the use of Vipps. This is useful for automated testing.
The endpoint is only available in our test environment.

**Important:** All test users must manually approve at least one payment in
Vipps (using the app) before "force approve" can be used for that user.
If this has not been done, you will get an error.
This is because the user needs to be registered as
"bankID verified" in the backend, and this is happens automatically in
the test environment when using Vipps (the app), but not with "force approve".

**Please note:** Vipps Hurtigkasse (express checkout) and `skipLandingPage`is
not supported by the force approve endpoint.

## Recommendations regarding handling redirects

See
[Common topics: Recommendations regarding handling redirects](https://developer.vippsmobilepay.com/docs/vipps-developers/common-topics/redirects).
