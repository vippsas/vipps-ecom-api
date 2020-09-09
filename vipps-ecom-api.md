# Vipps eCommerce API

API version: 2.0

Document version 2.3.8.

See: Vipps eCom API [GitHub repository](https://github.com/vippsas/vipps-ecom-api),
with
[Swagger specifications](https://vippsas.github.io/vipps-ecom-api/),
[Postman collections](tools/),
[example code](https://github.com/vippsas/vipps-developers/tree/master/code-examples/ecom_python_example),
[integration checklist](vipps-ecom-api-checklist.md)
and the [FAQ](vipps-ecom-api-faq.md).

See also: [How it works](vipps-ecom-api-howitworks.md).

## Table of contents

- [Vipps eCommerce API](#vipps-ecommerce-api)
  - [Table of contents](#table-of-contents)
  - [Flow diagram](#flow-diagram)
  - [API endpoints](#api-endpoints)
  - [Optional Vipps HTTP headers](#optional-vipps-http-headers)
  - [Initiate](#initiate)
  - [Regular eCommerce payments](#regular-ecommerce-payments)
    - [Reserve capture](#reserve-capture)
    - [Direct capture](#direct-capture)
    - [When to use reserve capture and direct capture](#when-to-use-reserve-capture-and-direct-capture)
  - [Express checkout payments](#express-checkout-payments)
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
  - [HTTP response codes](#http-response-codes)
  - [Rate limiting](#rate-limiting)
  - [Authentication](#authentication)
    - [Access token](#access-token)
    - [Partner Keys](#partner-keys)
  - [Idempotency](#idempotency)
  - [Exception handling](#exception-handling)
    - [Connection timeout](#connection-timeout)
    - [Callback aborted or interrupted](#callback-aborted-or-interrupted)
    - [PSP connection issues](#psp-connection-issues)
  - [App integration](#app-integration)
  - [App-switching](#app-switching)
    - [App-switch on iOS](#app-switch-on-ios)
      - [Switch from merchant app to the Vipps app](#switch-from-merchant-app-to-the-vipps-app)
      - [Redirect back to the merchant app from Vipps app](#redirect-back-to-the-merchant-app-from-vipps-app)
    - [App-switch on Android](#app-switch-on-android)
      - [Switching from merchant app to the Vipps app](#switching-from-merchant-app-to-the-vipps-app)
      - [Switching back to the merchant app from Vipps app](#switching-back-to-the-merchant-app-from-vipps-app)
      - [Return back to merchant app by actively deeplinking into it from Vipps](#return-back-to-merchant-app-by-actively-deeplinking-into-it-from-vipps)
      - [Redirect back to merchant app by simply closing the Vipps app](#redirect-back-to-merchant-app-by-simply-closing-the-vipps-app)
  - [Errors](#errors)
    - [Error object in the response](#error-object-in-the-response)
  - [Error groups](#error-groups)
  - [Error codes](#error-codes)
- [Testing](#testing)
- [Questions?](#questions)

## Flow diagram

This diagram shows a simplified payment flow:

![Vipps checkout flow chart](images/flow-diagram.png)

See [Get payment details](#get-payment-details) for more details about
the detailed flow, and [Payment states](#payment-states) for the corresponding
states.

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
| Access Token | Fetch the access token | [`POST:/accesstoken/get`](https://vippsas.github.io/vipps-ecom-api/#/Authorization_Service/fetchAuthorizationTokenUsingPost) |
| Get order status    | Deprecated, use [Get payment details](#get-payment-details) | [`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getPaymentDetailsUsingGET) |

See the
[eCom API checklist](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api-checklist.md).

## Optional Vipps HTTP headers

We recommend using the following _optional_ HTTP headers for all requests to the
Vipps eCom API. These headers provide useful metadata about the merchant's system,
which help Vipps improve our services, and also help in investigating problems.   

| Header                        | Description                                  | Example value        |
| ----------------------------- | -------------------------------------------- | -------------------- |
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

### Direct capture

**Direct capture** causes all payment reservations to be instantly be captured.
This is intended for situations where the product or service is immediately
provided to the customer, e.g. digital services.

When using direct capture, payments can not be cancelled as easily as
with reserve capture. A payment done with direct capture must be
_refunded_, and it then takes a few days before the money is available
in the customer's account.

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
that Vipps can call to delete the data. The Vipps app allows the user to later
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

**Please note:** Vipps can not guarantee a particular sequence, as this depends on user
actions, network connectivity/speed, etc. Because og this, it is not
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
works as a native app deeplink to the Vipps app (eg. with a `vipps://` scheme),
which automatically opens the Vipps app with app-switch.  
**Please note:** In our test environment (MT) the scheme is `vippsMT://`

The landing page is not involved in this flow, since the merchant app is
expected to use the `vipps://` URL to deeplink straight to the Vipps app.

1. Merchant initiates the payment with `isApp:true` parameter: [`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/initiatePaymentV3UsingPOST).
2. Vipps returns a `deeplink` URL as response to initiate payment.
3. The merchant uses the `vipps://` URL to invoke the Vipps app.
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
    "fallBack": "https://example.com/vipps/fallback-result-page/order123abc"
  },
  "transaction": {
    "orderId": "order123abc",
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
    "fallBack": "https://example.com/vipps/fallback-result-page/order123abc",
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
    "orderId": "order123abc",
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
* `"isApp": true`: The URL is for an deeplink, for app-switch to the Vipps app, with `vipps://`.


Example: Response body for `"isApp":false`, to the landing page:

```json
{
    "orderId": "order123abc",
    "url": "https://api.vipps.no/dwo-api-application/v1/deeplink/vippsgateway?v=2&token=eyJraWQiOiJqd3RrZXkiLC <snip>"
}
```
Example: Response body for `"isApp":true`, with a deeplink for app-switch:

```json
{
    "orderId": "order123abc",
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
* `orderId`: Must be unique for the `merchantSerialNumber`. Example: `order123abc`.

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

The maximum length of an `orderId` is 30 alphanumeric characters:
a-z, A-Z, 0-9 and '-'.

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
using the dummy URL `https://example.com/vipps/fallback-result-page/order123abc`:

```java
import org.apache.commons.validator.routines.UrlValidator;

public class UrlValidate {
 public static void main(String[] args) {
  UrlValidator urlValidator = new UrlValidator();

  if (urlValidator.isValid("https://example.com/vipps/fallback-result-page/order123abc")) {
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
[GET:/ecomm/v2/payments/{orderId}/details](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getPaymentDetailsUsingGET).

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

If the communication is broken during the process for some reason, and Vipps
is not able to execute callback to the merchant's server, the callback will
not be retried. In other words, if the merchant does not receive any
confirmation on the payment request, the merchant _must_ call
[GET:/ecomm/v2/payments/{orderId}/details](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getPaymentDetailsUsingGET)
to get the status of the payment.

### Callback endpoints

The callback URLs are generated by taking a field from the payment initiation request from the merchant in
[`POST:/ecomm/v2/payments/{orderId}`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/transactionUpdateCallbackForRegularPaymentUsingPOST),
and then adding a suffix. See the "Endpoints required by Vipps" section in the [Swagger documentation](https://vippsas.github.io/vipps-ecom-api/#/).

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
  "orderId": "order123abc",
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
  "orderId": "order123abc",
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

### Authorization for callbacks


To add authentication to the callbacks made by Vipps to the merchant,
the merchant may provide an `authToken`. This token will then be returned as
an `Authorization` header in the callback and shipping details requests made by
Vipps for that order. Please not that this is unrelated to the authentication
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

If the sale unit is not whitelisted, the request will fail and an error message will be returned

If you want to check if a sale unit is allowed to use `skipLandingPage`:
1. Initiate a normal payment with `"skipLandingPage": true`.
2. Check the response code and message. The API will return an error if attempting to use
   `skipLandingPage` without being whitelisted.

If you need to be whitelisted, instructions can be found in the
[FAQ](https://github.com/vippsas/vipps-psp-api/blob/master/vipps-psp-api-faq.md#is-it-possible-to-skip-the-landing-page).

## Reserve

![Payment confirmation](images/vipps-flow-reserve.png)

When the user confirms, the payment status changes to `RESERVE`.
The respective amount will be reserved for future capturing.

## The Vipps landing page

![The Vipps landing page](images/vipps-flow-landing-page.png)

When a user is directed to the `url` from initiate payment, they will either be taken to the Vipps app or to the Vipps landing page:

* In a mobile browser, the user will be redirected to the Vipps app.
* In a desktop browser, the landing page prompts the user for the phone number (the number may also be pre-filled).
  The user enters or confirms the phone number. Then on their phone, the user gets a push notification and the Vipps app then prompts for confirmation.

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

## Reserve capture

_Reserve capture_ is the normal flow.

When then end user approves an initiated payment it will be reserved until you capture it. When the order is reserved
the amount is marked as reserved by the bank, but not transferred.

## Direct capture

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

## Partial capture

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

**Please note:** It is important to check the response of the `/capture`
call. The capture is only successful when the response is
`HTTP 200 OK`.

## Cancel

The Cancel request allows the merchant to cancel a reserved or initiated transaction.

The payment flow can be aborted, under certain circumstances:

* When the user cancels (rejects) the initiated payment in the Vipps app.
* When the merchant cancels.
* Timeouts: If the user does not confirm, etc.

Partially captured reservations can not be cancelled.

After cancellation, the order gets a new status:
* If an order is cancelled by the merchant, it gets the status `VOID`.
* If an order is cancelled by the user, it gets the status `CANCEL`.

Swagger: [`PUT:/ecomm/v2/payments/order123abc/cancel`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/cancelPaymentRequestUsingPUT)

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
    "orderId": "order123abc",
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

Refunds can be made up to 365 days after capture.

Swagger: [`POST:/ecomm/v2/payments/order123abc/refund`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/refundPaymentUsingPOST)

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
    "orderId": "order123abc",
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

| #   | From-state | To-state | Description                                   | Operation |
| --- | ---------- | -------- | --------------------------------------------- | -------------- |
| 0   | -          | Initiate | Payment initiation                            | `INITIATE`     |
| 1   | Initiate   | -        | The merchant has initiated the payment        | `INITIATE`     |
| -   |            | Reserve  | The user has accepted the payment and amount has been reserved | `RESERVE`      |
| -   |            | Cancel   | The user cancels the order                    | `CANCEL`       |
| 2   | Reserve    | Capture  | The merchant captures the payment, and ships  | `CAPTURE`      |
| -   |            | Cancel   | The merchant cancels the order                | `VOID`         |
| 3   | Capture    | --       | A final state: Payment fully processed        | `CAPTURE`      |
| -   |            | Refund   | The merchant refunds the money to the user    | `REFUND`     |
| 4   | Cancel     | --       | A final state: Payment cancelled              | -              |
| 5   | Refund     | --       | A final state: Payment refunded               | -              |

## Requests and responses

Please note that the response from
[`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getPaymentDetailsUsingGET))
always contain _the entire history_ of payments for the order, not just the current status.

**Important:** The `operationSuccess` filed indicates whether an operation was successful or not.

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
    "orderId": "order123abc",
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
| `REJECTED` | No user action in the Vipps app, i.e timeout. |

## Userinfo

**Important:** This functionality is currently being rolled out in all environments.
The documentation is being updated, but is not yet complete.

Vipps offers the possibility for merchants to ask for the users profile information as part of the payment flow. If the enduser has not already consented to sharing information from Vipps to the merchant the user will be asked for such consent before completing the payment flow. Once the payment flow is completed the merchant can get the profile information from our Userinfo endpoint.

The [`GET:/userinfo/{sub}`](https://vippsas.github.io/vipps-login-api/#/Vipps%20Log%20In%20API/userinfo) endpoint is shared by eCom and [Vipps login](https://github.com/vippsas/vipps-login-api) and the merchant needs to have activated Vipps login to use this feature. You can find more information on how to activate Vipps login[here](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api-faq.md#how-can-i-activate-and-set-up-vipps-login).

A user's consent to share information with a merchant applies across our services. Thus, if the merchant implements Vipps Login in addition to profile information as part of the payment flow, the merchant can also use Vipps to log the user in without the need for additional consents.

When you initiate a payment: Add the parameter `scope` to ask for a user's
consent to share these details, such as email, address and name.

| Scopes      | Description                                    | User consent required  |
| ------------| -----------------------------------------------|-------- |
| address     | List containing the users addresses. Will always contain home, but can also include work and other.    |   yes   |
| birthDate   | User birth date (BankID verified)                               |   yes   |
| email       | User email (verified), the flag "email_verified : true" in the response can be used by merchant to confirm for each request that the email actually is verified                                   |   yes   |
| name        | User first, middle and given name (verified with National Population Register)              |   yes   |
| phoneNumber | Verified phone number (verified - the number used with Vipps)                          |   yes   |
| nin        | Norwegian national identity number (verified with BankID). NB: merchants need to apply for access to NIN. Go to [Who can get access to NIN and how?](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api-faq.md#who-can-get-access-to-nin-and-how) For more information |   yes      |
| accountNumbers | User bank account numbers. NB: merchants need to apply for access to accountNumbers. Go to [Who can get access to account numbers and how?](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api-faq.md#who-can-get-access-to-accountnumbers-and-how) For more information |   yes      |

If the e-mail address that is delivered has the flag "email_verified : false" this address should not be used to link the user to an existing account without further authentication. Such authentication could be to prompt the user to login to the original account or confirm the account linking by having a confirmation link sent to the email address.

To request these scopes add the scopes to the initial call to
[`POST:​/ecomm​/v2​/payments`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/initiatePaymentV3UsingPOST)

The user then consents and pays in the app.

**Please note:** This operation has an "all or nothing" approach, so a user must
complete a valid payment and consent to all values in order to complete the
session. If a user chooses to reject the terms the payment will not be
processed. Unless the whole flow is completed, this will be handled as a regular
failed payment by the eCom API.

Once the user completes the session a unique identifier `sub` can be retrieved from the
[`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/getPaymentDetailsUsingGET) endpoint.

Example `sub` format:

```
"sub": "c06c4afe-d9e1-4c5d-939a-177d752a0944",
```

This `sub` is a link between the merchant and the user and can used to retrieve
the user's details from Vipps Login:
[`GET:/userinfo/{sub}`](https://vippsas.github.io/vipps-login-api/#/Vipps%20Log%20In%20API/userinfo)

**Please note:** accessing the Login `userinfo` endpoint requires the
Vipps Login access token:
[`POST:/oauth2/token`](https://vippsas.github.io/vipps-login-api/#/Vipps%20Log%20In%20API/oauth2Token) this is described in detail in [login's documentation](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api.md#access-token)

![Userinfo sequence](images/userinfo-direct.png)

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

| API                                                                                                         | Limit          | Key           |
|-------------------------------------------------------------------------------------------------------------|----------------|---------------|
| [InitiatePayment](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/initiatePaymentV3UsingPOST) | 2 per minute   | orderId + msn |
| [CancelPayment](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/cancelPaymentRequestUsingPUT) | 5 per minute   | orderId + msn |
| [CapturePayment](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/capturePaymentUsingPOST)     | 5 per minute   | orderId + msn |
| [RefundPayment](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/refundPaymentUsingPOST)       | 5 per minute   | orderId + msn |
| [~~GetOrderStatus~~](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/getOrderStatusUsingGET)  | 120 per minute | orderId + ocp |
| [GetPaymentDetails](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/getPaymentDetailsUsingGET)| 120 per minute | orderId + ocp |

## Authentication

All API calls are authenticated and authorized based on the application access
token (JWT bearer token) and a subscription key (`Ocp-Apim-Subscription-Key`),
and these headers are required:

| Header Name | Header Value | Description |
| ----------- | ------------ | ----------- |
| `Authorization` | `Bearer <JWT access token>` | Type: Authorization token. This is available on [portal.vipps.no](https://portal.vipps.no). |
| `Ocp-Apim-Subscription-Key` | Base 64 encoded string | The subscription key for the eCom API. This is available on [portal.vipps.no](https://portal.vipps.no). |

### Access token

The Access Token API provides the JWT bearer token:
[`POST:/accesstoken/get`](https://vippsas.github.io/vipps-ecom-api/#/Authorization_Service/fetchAuthorizationTokenUsingPost).

Request:

```http
POST https://apitest.vipps.no/accesstoken/get
client_id: fb492b5e-7907-4d83-ba20-c7fb60ca35de
client_secret: Y8Kteew6GE2ZmeycEt6egg==
Ocp-Apim-Subscription-Key: 0f14ebcab0ec4b29ae0cb90d91b4a84a
```

(We are aware that this is a `POST`, without a body, to an endpoint with
`get` in the URL, and hope to fix it in a later version of the API.)

All headers are unique per `merchantSerialNumber` and can be found on [portal.vipps.no](https://portal.vipps.no).

| Header Name | Header Value | Description |
| ----------- | ------------ | ----------- |
| `client_id` | A GUID value | Client ID for the merchant |
| `client_secret` | Base 64 encoded string | Client Secret for the merchant |
| `Ocp-Apim-Subscription-Key` | Base 64 encoded string | Subscription key for the product |

For the subscription `Vipps-eCommerce-Services-API` the `Ocp-Apim-Subscription-Key`
for an access token request will be the same as for eCommerce requests. While
for other subscriptions they will use different keys. See
[getting started](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md)
for more details.

Response:

````http
HTTP 200 OK
````

```json
{
  "token_type": "Bearer",
  "expires_in": "86398",
  "ext_expires_in": "0",
  "expires_on": "1495271273",
  "not_before": "1495184574",
  "resource": "00000002-0000-0000-c000-000000000000",
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni <snip>"
}
```

JWT properties:

| Name                        | Description                                 |
| --------------------------- | ------------------------------------------- |
| `Bearer`                    | It’s a `Bearer` token. The word `Bearer` should be added before the token |
| `expires_in`                | Token expiry duration in seconds. |
| `ext_expires_in`            | Extra expiry time. Not used. |
| `expires_on`                | Token expiry time in epoch time format. |
| `not_before`                | Token creation time in epoch time format. |
| `resource`                  | For the product for which token has been issued. |
| `access_token`              | The actual access token that needs to be used in `Authorization` request header. |

**Please note:** The access token is valid for 1 hour in MT (Merchant Test)
and 24 hours in Production. To be sure that you are using correct time please
use `expires_in` or `expires_on`.

Example of an error response body (formatted for readability):

```json
{
  "error": "unauthorized_client",
  "error_description":
    "AADSTS70001: Application with identifier 'e9b6c99d-2442-4a5d-84a2-\
     c53a807fe0c4' was not found in the directory testapivipps.no\
     Trace ID: 3bc2b2a0-d9bb-4c2e-8367- 5633866f1300\r\nCorrelation ID:\
     bb2f4093-70af-446a-a26d-ed8becca1a1a\r\nTimestamp: 2017-05-19 09:21:28Z",
  "error_codes": [ 70001 ],
  "timestamp": "2017-05-19 09:21:28Z",
  "trace_id": "3bc2b2a0-d9bb-4c2e-8367-5633866f1300",
  "correlation_id": "bb2f4093-70af-446a-a26d-ed8becca1a1a"
}
```

### Partner Keys

If you are a Vipps Partner that is managing transactions on behalf of other Vipps Merchants you can use your own credentials to authenticate, and then send the Merchant-Serial-Number to identify
which of your Vipps Merchant you are acting on behalf of. The Merchant-Serial-Number must be sent in the header of all requests. as described in our swagger documentation

[`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/initiatePaymentV3UsingPOST)

> The Merchant Serial Number (MSN) is a unique id for the sale unit that this payment is made for. This is a required parameter if you are a Vipps eCom partner making payments on behalf of a merchant. The partner must use the merchant's MSN (not the partner's MSN). This parameter is optional, and recommended, for regular Vipps merchants making payments for themselves.


## Idempotency

In a capture request the merchant may also use the `X-Request-Id`header.
This header is an idempotency header ensuring that if the merchant retries
a request with the same `X-Request-Id` the retried request will not make
additional changes.

Request:

[`POST:/ecomm/v2/payments/order123abc/capture`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/capturePaymentUsingPOST)

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
    "orderId": "order123abc",
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

If the communication is broken during payment process for some reason, and
Vipps is not able to execute callback, then callback will not be retried.

In other words, if the merchant doesn’t receive any confirmation on payment
request call within callback timeframe, merchant should call
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

Merchants may implement deep-linking, to trigger the Vipps app (we refer to this as "app-switch" here).

This may be done in two ways:

1. From a mobile or desktop browser. See [Desktop flow](#desktop-flow)
2. From an iOS or Android native app

After the user has finished (or cancelled) the payment in the Vipps app, the user is returned back to the specified `fallBack` URL. When the user arrives back in the merchant app or website, we _strongly_ recommend that you perform a call to the [payment details endpoint](#get-payment-details) to check the state
of the transaction. While some of the state of the eCom operation *can* be derived from things like whether or not user returned successfully from the
Vipps app, the most reliable approach to know the state of the payment is always to query the eCom API once the user arrive back in the merchant app/website.

## App-switching
1. The merchant need to pass the URL scheme of the app into the `fallBack` field in the [initiate payment request](#initiate-payment-flow-api-calls). See [App-switch on iOS](#app-switch-on-ios) and [App-switch on Android](#app-switch-on-android) for specifics.
2. The merchant app should open the Vipps app deeplink received from the [initiate payment request](#initiate-payment-flow-api-calls).
3. Once the operation in the Vipps app is completed, Vipps will redirect the user to the deeplink specified in the `fallBack` field.
4. The merchant app should query the [payment details endpoint](#get-payment-details) for updated status on the payment once user returns from the Vipps app.

**Please note:** Vipps will append a status at the end of the fallback URL. For example, if your `fallBack` URL is `merchantApp://result?myAppData`, Vipps
will append the status like: `merchantApp://result?myAppData&status=301`. **This status is deprecated, and should no longer be used.** Query the [payment details endpoint](#get-payment-details) for latest status instead.

### App-switch on iOS
The Vipps app on iOS requires a URL scheme in order to support app-switch.

See the official Apple documentation:
[Defining a Custom URL Scheme for Your App](https://developer.apple.com/documentation/uikit/core_app/allowing_apps_and_websites_to_link_to_your_content/defining_a_custom_url_scheme_for_your_app)


#### Switch from merchant app to the Vipps app

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

**Please note:** If you have both the official Vipps app from App Store _and_ the test Vipps app from TestFlight installed,
the app switch may open either one. There is no way to open one of them specifically, as
the URL scheme is the same for both. We recommend to only have one of the apps installed on the device.

#### Redirect back to the merchant app from Vipps app

Once the operation in the Vipps app is completed, the Vipps app will open the fallback URL.
For app-to-app integration, merchant app needs to be registered for a URL scheme
and pass the URL scheme in `fallBack` URL in the Vipps backend API.
The Vipps mobile application will use the URL to launch the merchant application.

### App-switch on Android

Vipps is launched with a standard intent, using the url returned from the eCom
API when the payment is created ("url": "vipps://?token=eyJraWQiOiJqd3RrZXki..")

#### Switching from merchant app to the Vipps app

Example of how to open the Vipps app:

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

Once the user has paid (or cancelled), the Vipps app supports two ways to return to the merchant native app:

  1: Let the vipps app deeplink back into the merchant app using their url scheme (eg. merchantapp://). This is the default/suggested approach.

  2: Just close the Vipps app, fall back to the merchant app, pick up the thread again there in onActivityResult().

In both cases, the merchant app should query the ecom API for updated status on the payment once user returns from the Vipps app.

#### Return back to merchant app by actively deeplinking into it from Vipps

With this approach, the merchant app has to have its own URL scheme registered so the Vipps app can actively open the merchant app again after payment/cancellation.

In the example below, `MainActivity` is the receiving activity and the Vipps app opens it once the payment is done.

To receive a call back from the Vipps application to an activity, a filter has to be set for that activity. In the merchant app, set a filter in the Manifest file:

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
result send by the Vipps app:

```java
@Override
protected void onNewIntent(Intent intent) {
      // Call the eCom api, check the status of the eCom payment
}
```

#### Redirect back to merchant app by simply closing the Vipps app

With this approach, the merchant app does not have to register/handle deeplink urls.

In order to use this approach, when creating the payment in the merchant has to pass fallback attribute like this:

```"fallBack": "INTENT" ```

(and *only* "INTENT"", no parameters etc.)

This will cause the Vipps app to simply close after a successful or canceled ecom payment, and fall back to the calling merchant app.

The merchant app activity that resumes again (after the Vipps app closes) has to override onActivityResult method to pick up the thread again here. Example:

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
eCom API without the use of the Vipps App. This is useful for automated testing.
The endpoint is only available in our test environment.

Express checkout and "Skip Landing Page" is not supported by the force approve endpoint.

# Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-ecom-api/issues),
a [pull request](https://github.com/vippsas/vipps-ecom-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).

Sign up for our [Technical newsletter for developers](https://github.com/vippsas/vipps-developers/tree/master/newsletters).
