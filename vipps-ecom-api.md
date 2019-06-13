# Vipps eCommerce API

API version: 2.0

Document version 1.0.5

See also the [Vipps eCommerce FAQ](vipps-ecom-api-faq.md)

API details: [Swagger UI](https://vippsas.github.io/vipps-ecom-api/#/),
[swagger.yaml](https://raw.githubusercontent.com/vippsas/vipps-ecom-api/master/docs/swagger.yaml),
[swagger.json](https://raw.githubusercontent.com/vippsas/vipps-ecom-api/master/docs/swagger.json).

# Table of contents

- [Overview](#overview)
  * [Payment types](#payment-types)
    + [Regular eCommerce payments](#regular-ecommerce-payments)
      - [Reserve capture](#reserve-capture)
      - [Direct capture](#direct-capture)
    + [Desktop browsers and mobile browsers](#desktop-browsers-and-mobile-browsers)
    + [Express checkout payments](#express-checkout-payments)
      - [API endpoints required by Vipps from the merchant for express checkout](#api-endpoints-required-by-vipps-from-the-merchant-for-express-checkout)
  * [Flow diagram](#flow-diagram)
    + [Flow diagram details](#flow-diagram-details)
- [API endpoints](#api-endpoints)
  * [Authentication](#authentication)
    + [Access token](#access-token)
      - [HTTP response codes](#http-response-codes)
  * [Initiate payment](#initiate-payment)
    + [Initiate payment flows](#initiate-payment-flows)
      - [Mobile browser initiated payments](#mobile-browser-initiated-payments)
        * [Vipps app installed](#vipps-app-installed)
        * [Vipps app not installed](#vipps-app-not-installed)
      - [Desktop browser initiated payments](#desktop-browser-initiated-payments)
      - [App initated payments](#app-initated-payments)
    + [URL Validation](#url-validation)
  * [Reserve](#reserve)
  * [Cancel](#cancel)
  * [Capture](#capture)
  * [Refund](#refund)
- [HTTP response codes](#http-response-codes-1)
  * [Exception handling](#exception-handling)
    + [Connection timeout](#connection-timeout)
    + [Callback aborted/interrupted](#callback-aborted-interrupted)
    + [PSP connection issues](#psp-connection-issues)
  * [Error groups](#error-groups)
  * [Error codes](#error-codes)
- [App integration](#app-integration)
  * [App-switch between mobile or desktop browsers and the Vipps app](#app-switch-between-mobile-or-desktop-browsers-and-the-vipps-app)
    + [App-switch on iOS](#app-switch-on-ios)
      - [Switch from merchant app to the Vipps app](#switch-from-merchant-app-to-the-vipps-app)
      - [Redirect back to the merchant app from Vipps app](#redirect-back-to-the-merchant-app-from-vipps-app)
        * [Registering a 3rd party app with URL scheme and handling custom URL calls](#registering-a-3rd-party-app-with-url-scheme-and-handling-custom-url-calls)
    + [App-switch on Android](#app-switch-on-android)
      - [App-switch: Android Intent](#app-switch--android-intent)
        * [Redirect back to merchant app](#redirect-back-to-merchant-app)
      - [App-switch: Android URL Scheme](#app-switch--android-url-scheme)
        * [Redirect back to merchant app](#redirect-back-to-merchant-app-1)
  * [Error codes for deeplinking](#error-codes-for-deeplinking)
- [API endpoints required by Vipps from the merchant](#api-endpoints-required-by-vipps-from-the-merchant)
  * [Vipps callback servers](#vipps-callback-servers)
  * [Callback endpoints](#callback-endpoints)
  * [Callback](#callback)
  * [Fetch Shipping Cost](#fetch-shipping-cost)
  * [Remove User Consent](#remove-user-consent)
- [Status and operations](#status-and-operations)
  * [Responses from requests](#responses-from-requests)
  * [Callbacks](#callbacks)
- [Questions?](#questions-)

# Overview

The Vipps eCommerce API (eCom API) offers functionality for online payments,
both using web browsers on websites and in native apps for iOS and Android,
using app-switching.

## Payment types

Vipps eCommerce API offers 2 types of payments
1. Regular eCommerce payments
2. Express checkout payments

### Regular eCommerce payments

When you initiate a payment it will be reserved until you capture it.
Vipps supports both _reserve capture_ and _direct capture_.

_Reserve capture_ is the default. When you initiate a payment it will be reserved until you capture it.

When _direct capture_ is activated, all payment reservations will instantly be captured.
This is intended for situations where the product or service is immediately provided to the customer, e.g. digital services.

According to Norwegian regulations you should _not_ capture a payment until the product or service is provided to the customer.
For more information, please see the Consumer Authority's
[Guidelines for the standard sales conditions for consumer purchases of goods over the internet](https://www.forbrukertilsynet.no/english/guidelines/guidelines-the-standard-sales-conditions-consumer-purchases-of-goods-the-internet).

Merchants do not choose between _reserve capture_ and _direct capture_ themselves,
the type of capture is configured by Vipps after the additional compliance checks have been completed.

#### Reserve capture

_Reserve capture_ is the normal flow.

When then end user approves an initiated payment it will be reserved until you capture it.

#### Direct capture

When _direct capture_ is activated, all payment reservations will instantly be captured.

When doing _direct capture_ Vipps is responsible for the customer receiving the purchased product.
Because of this, _direct capture_ requires additional compliance checks, and the following
requirements apply:
* The merchant must have an annual revenue of more than 10 million NOK
* The merchant must have a Key Account Manager (KAM) in Vipps
* The merchant must have a partner that is responsible for the Vipps integration

To request _direct capture_, please contact your KAM.

### Desktop browsers and mobile browsers

When a user has selected Vipps for payment, the Vipps landing page
detects whether user is using a desktop browser or a mobile browser:

* In a mobile browser, the landing page detects if the Vipps app is installed,
  and automatically switches to the Vipps app if it is.
* In a desktop browser, the landing page prompts the user for the phone number (the number may also be pre-filled).
  The user enters or confirms the phone number, and the Vipps app prompts for confirmation on the phone.

The Vipps landing page is mandatory, and provides a consistent and recognizable user experience,
that helps guide the user through the payment flow.
In this way Vipps takes responsibility for helping the user from the browser to the app,
and to complete the payment in a familiar way.

### Express checkout payments

These are payments related to
[Vipps Hurtigkasse](https://www.vipps.no/bedrift/vipps-pa-nett/hurtigkasse),
where Vipps reduces the typical purchase process to a few simple steps:

1. The user clicks on the "Vipps Hurtigkasse" button
2. The user confirms the amount, delivery address and delivery method in the Vipps app
3. The merchant receives shipping information and user details and shows a confirmation page

Vipps retrieves the shipping methods and shipping price from merchant via the `shippingDetails` endpoint based
on the address the end user selects in the Vipps app.

Vipps complies with GDPR, and requires the user's consent before any information
is shared with the merchant. The merchant must provide a URL (`consentRemovalPrefix`)
that Vipps can call to delete the data. The Vipps app gives the Vipps user an
overview of "Companies with access", where the user can manage the consents.

#### API endpoints required by Vipps from the merchant for express checkout

The below endpoints are provided by the _merchant_ and consumed by Vipps during express checkout payments.
These endpoints are included in the Swagger file for reference.

| Operation           | Description         | Endpoint          |
| ------------------- | ------------------- | ----------------- |
| Remove user consent | Used to inform merchant when the Vipps user removes consent to share information.  | [`DELETE:/v2/consents/{userId}`](https://vippsas.github.io/vipps-ecom-api/#/Endpoints_required_by_Vipps_from_the_merchant/removeUserConsentUsingDELETE)  |
| Callback : Transaction Update | A callback to the merchant for receiving post-payment information. | [`POST:/v2/payments/{orderId}`](https://vippsas.github.io/vipps-ecom-api/#/Endpoints_required_by_Vipps_from_the_merchant/transactionUpdateCallbackForRegularPaymentUsingPOST)  |
| Get shipping cost and method | Used to fetch shipping information | [`POST:/v2/payments/{orderId}/shippingDetails`](https://vippsas.github.io/vipps-ecom-api/#/Endpoints_required_by_Vipps_from_the_merchant/fetchShippingCostUsingPOST)  |

## Flow diagram

![Vipps checkout flow chart](images/vipps-ecom-flow-chart.svg)

### Flow diagram details

This table shows the from- and to-state, and the status returned from
""Get order status"
([`GET:/ecomm/v2/payments/{orderId}/status`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/initiatePaymentV3UsingPOST), [[Redoc](https://vippsas.github.io/vipps-ecom-api/redoc.html#/oneclick-payment-with-vipps-controller/getOrderStatusUsingGET)], [[Swagger](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/initiatePaymentV3UsingPOST)]).

| #   | From-state | To-state | Description                                   | getOrderStatus |
| --- | ---------- | -------- | --------------------------------------------- | -------------- |
| 0   | -          | Initiate | Payment initiation                            | `INITIATE`     |
| 1   | Initiate   | -        | The merchant has initiated the payment        | `INITIATE`     |
| -   |            | Reserve  | The user has accepted the payment and amount has been reserved | `RESERVE`      |
| -   |            | Cancel   | The user cancels the order                    | `CANCEL`       |
| 2   | Reserve    | Capture  | The merchant captures the payment, and ships  | `CAPTURE`      |
| -   |            | Cancel   | The merchant cancels the order                | `VOID`         |
| 3   | Capture    | --       | A final state: Payment fully processed        | `RESERVE`      |
| -   |            | Refund   | The merchant refunds the money to the user    | `RESERVE`     |
| 4   | Cancel     | --       | A final state: Payment cancelled              | -              |
| 5   | Refund     | --       | A final state: Payment refunded               | -              |

**Please note:** When using Get order status ([`GET:/ecomm/v2/payments/{orderId}/status`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getOrderStatusUsingGET)),
the order will show as `"status": "RESERVE"`, even after operations as capture and refund. To se if the payment has been completed, and the reserved amount has been captured, use Get payment details ([`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getPaymentDetailsUsingGET)).

Please note that the response from Get payment details [`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getPaymentDetailsUsingGET))
always contain _the entire history_ of payments for the order, not just the current status.
In this truncated example, it shows that the full amount (200.00 NOK) has been captured:

```json
"transactionSummary": {
    "capturedAmount": 20000,
    "remainingAmountToCapture": 0,
    "refundedAmount": 0,
    "remainingAmountToRefund": 0
}
```

In this truncated example, it shows that a partial capture of 100.00 NOK, of
the total reserved amount of 200.00 NOK, has been captured:

```json
"transactionSummary": {
    "capturedAmount": 10000,
    "remainingAmountToCapture": 10000,
    "refundedAmount": 0,
    "remainingAmountToRefund": 0
}
```

# API endpoints
This section contains complete HTTP `requests` and `responses` for each API endpoint

| Operation           | Description         | Endpoint          |
| ------------------- | ------------------- | ----------------- |
| Initiate payment    | Payment initiation, the first request in the payment flow. This _reserves_ an amount. | [`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/initiatePaymentV3UsingPOST)  |
| Capture payment     | When an amount has been reserved, and the goods are (about to be) shipped, the payment must be _captured_  | [`POST:/ecomm/v2/payments/{orderId}/capture`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/capturePaymentUsingPOST)  |
| Cancel payment      | The merchant may cancel a reserved amount, but not on a captured amount.  | [`PUT:/ecomm/v2/payments/{orderId}/cancel`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/cancelPaymentRequestUsingPUT)  |
| Refund payment      | The merchant may refund a captured amount.  |[`POST:/ecomm/v2/payments/{orderId}/refund`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/refundPaymentUsingPOST)  |
| Get order status    | The status is "reserved" after a payment has been initiated. For details about payment, use [`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getPaymentDetailsUsingGET) | [`GET:/ecomm/v2/payments/{orderId}/status`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getOrderStatusUsingGET)  |
| Get payment details | How much of the reserved amount has been captured, etc.  | [`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getPaymentDetailsUsingGET)  |
| Access Token | Fetch the access token | [`POST:/accesstoken/get`](https://vippsas.github.io/vipps-ecom-api/#/Authorization_Service/fetchAuthorizationTokenUsingPost) |

See [Complete HTTP requests and responses for each API endpoint and method](#complete-http-requests-and-responses-for-each-api-endpoint-and-method) for more details.

## Authentication

All API calls are authenticated and authorized based on the application access
token (JWT bearer token) and a subscription key (`Ocp-Apim-Subscription-Key`),
and these headers are required:

| Header Name | Header Value | Description |
| ----------- | ------------ | ----------- |
| `Authorization` | `Bearer <JWT access token>` | Type: Authorization token. This is available in the Vipps Developer Portal. |
| `Ocp-Apim-Subscription-Key` | Base 64 encoded string | The subscription key for the eCom API. This is available in the Vipps Developer Portal. |

### Access token

The Access Token API provides the JWT bearer token.

**Request**

[`POST:/accesstoken/get`](https://vippsas.github.io/vipps-ecom-api/#/Authorization_Service/fetchAuthorizationTokenUsingPost).

```http
POST https://apitest.vipps.no/accessToken/get
client_id: <client_id>
client_secret: <client_secret>
Ocp-Apim-Subscription-Key: <Ocp-Apim-Subscription-Key>
```
All headers are per merchantSerialNumber and can be found in Vipps Developer Portal.

| Header Name | Header Value | Description |
| ----------- | ------------ | ----------- |
| `client_id` | A GUID value | Client ID for the merchant |
| `client_secret` | Base 64 encoded string | Client Secret for the merchant |
| `Ocp-Apim-Subscription-Key` | Base 64 encoded string | Subscription key for the product |

For the subscription `Vipps-eCommerce-Services-API` the `Ocp-Apim-Subscription-Key` for a access token request will be the same as for eCommerce requests. While for other subscriptions they will use different keys. See [getting started](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md) for a guide.

**Response**
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

**Please note:** The access token is valid for 1 hour in MT and 24 hours in Production. To be sure that you are using correct time please use `expires_in` or `expires_on`.

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

#### HTTP response codes

This API returns the following HTTP statuses in the responses:

| HTTP status         | Description                                 |
| ------------------- | ------------------------------------------- |
| `200 OK`            | Request successful.                          |
| `400 Bad Request`   | Invalid request, see the `error` for details.  |
| `401 Unauthorized`  | Invalid authorization.                         |
| `403 Forbidden`     | Authentication ok, but credentials lacks authorization.  |
| `500 Server Error`  | An internal Vipps problem.                  |

## Initiate payment

Initiate payment is used to create a new payment order in Vipps:

[`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/initiatePaymentV3UsingPOST)

**Initiate request headers example:**
```json
{
    "Authorization": "Bearer jfewioIJffruJIfewiyYUdweiLMfnewUQiwmdcLnfewjNj <snip>",
    "Content-Type": "application/json",
    "Ocp-Apim-Subscription-Key": "12345678901234567890123456789012"
}
```

A request is authorized by the `Authorization` and `Ocp-Apim-Subscription-Key` included in the
HTTP headers.
* `Authorization`: The word Bearer followed by the token from a Access Token request.
* `Ocp-Apim-Subscription-Key`: Subscription key for the eCommerce Product.

**Initiate request body example:**

A minimal example:


```json
{
    "merchantInfo": {
      "merchantSerialNumber": "123456",
      "callbackPrefix":"https://vipps.io/vipps/callbacks/",
      "fallBack": "https://vipps.no/"
    },
    "customerInfo": {
    },
    "transaction": {
      "orderId": "order123abc",
      "amount": 20000,
      "transactionText": "One pair of Vipps socks"
    }
}
```

An example with more parameters provided:

```json
{
  "customerInfo": {
    "mobileNumber": 91234567
  },
  "merchantInfo": {
    "authToken": "string",
    "callbackPrefix": "https://example.com/vipps/callbacks/",
    "consentRemovalPrefix": "https://example.com/vipps/consents/",
    "fallBack": "https://example.com/vipps/fallback/",
    "isApp": false,
    "merchantSerialNumber": 123456,
    "shippingDetailsPrefix": "https://example.com/vipps/shipping/",
    "paymentType": "eComm Regular Payment"
  },
  "transaction": {
    "amount": 20000,
    "orderId": "order123abc",
    "timeStamp": "2018-12-12T11:18:38.246Z",
    "transactionText": "One pair of Vipps socks"
  }
}
```

A payment is uniquely identified by the combination of `merchantSerialNumber` and `orderId`:
* `merchantSerialNumber`: The merchant's Vipps id.
* `orderId`: Must be unique for the `merchantSerialNumber`.

To initiate an express checkout payment the payment initiation call must include the `"paymentType":"eComm Express Payment"` parameter. If this parameter is not passed, the payment type will default to regular payment.

To add authentication to the callbacks made by Vipps the merchant may provide an `authToken`. This token will then be returned as a `Authorization` header in the callback and shipping details requests made by Vipps for that order.

Once successfully initiated, a response with a redirect URL is returned.

**Note:** We do not send requests to `callbackPrefix`, `shippingDetails` or `consentsRemovalPrefix` for all ports. To be safe use common port numbers such as: 80, 443, 8080.

**Initiate response example:**
```http
HTTP 202 Accepted
```

Response body for `"isApp":false`:
```json
{
    "orderId": "order123abc",
    "url": "https://api.vipps.no/deeplink/vippsgateway?token=eyJraWQiOiJqd3R <snip>"
}
```
Response body for `"isApp":true`:
```json
{
    "orderId": "order123abc",
    "url": "vipps://?token=eyJraWQiOiJqd3RrZXkiLCJhbGciOiJSUzI1NiJ9.eyJzdWIi <snip>"
}
```

The `url` is slightly simplified, but the format is correct.

The URL depends on whether the `initiate` request was provided the `isApp` parameter:
* For `true`, the URL is for an deeplink to the Vipps app.
* For `false` or not provided, the URL is for the Vipps "landing page".

**Note:** this URL will timeout after 5 minutes.

### Initiate payment flows

#### Mobile browser initiated payments

The landing page will detect if the Vipps app is installed.

##### Vipps app installed

1. The Vipps app is invoked and the landing page is closed.
2. The Vipps user accepts or rejects the payment request
3. Once payment process is completed, Vipps app redirects to `fallBack`

##### Vipps app not installed

1. The user is prompted for the mobile number.
2. Vipps sends a push notification to corresponding Vipps profile, if it exists. The landing page is not closed in this case.
3. The Vipps user accepts or rejects the payment request.
4. Once payment process is completed, Vipps app redirects to `fallBack`

#### Desktop browser initiated payments

1. The landing page will be opened in the desktop browser
2. The landing page will prompt for user’s mobile number
3. Vipps sends a push notification to corresponding Vipps profile, if it exists. The landing page is not closed in this case
4. The Vipps user accepts or rejects the payment request
5. Once payment process is completed, the landing page will redirect to `fallBack`

#### App initated payments

Vipps will identify the request coming from merchant's app by the `isApp:true` parameter.
In this case, the Vipps backend will send the URI use by the merchant to invoke the Vipps app.
The landing page is not involved in this case.

1. Merchant initiates the payment with `isApp:true` parameter.
2. Vipps sends a `deeplink` URI as response to initiate payment.
3. The merchant uses the URI to invoke the Vipps app.
4. The Vipps user accepts or rejects the payment request.
5. Once payment process is completed, Vipps app redirects to `fallBack`

### URL Validation

All URLs in Vipps eCommerce API are validated with the
[Apache Commons UrlValidator](https://commons.apache.org/proper/commons-validator/apidocs/org/apache/commons/validator/routines/UrlValidator.html).

If `isApp` is true, the `fallBack` is not validated with Apache Commons UrlValidator,
as the app-switch URL may be something like `vipps://`, which is not a valid URL.

Here is a simple Java class suitable for testing URLs,
using the dummy URL `https://example.com/vipps/fallback?id=abc123`:

```java
import org.apache.commons.validator.routines.UrlValidator;

public class UrlValidate {
 public static void main(String[] args) {
  UrlValidator urlValidator = new UrlValidator();

  if (urlValidator.isValid("https://example.com/vipps/fallback?id=abc123")) {
   System.out.println("URL is valid");
  } else {
   System.out.println("URL is invalid");
  }
 }
}
```


**Example Get order status** - [`GET:/ecomm/v2/payments/order123abc/status`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getOrderStatusUsingGET)

```json
{
    "orderId": "order123abc",
    "transactionInfo": {
        "amount": 20000,
        "status": "INITIATE",
        "transactionId": "5001420062",
        "timeStamp": "2018-11-14T15:44:26.590Z"
    }
}
```

**Example Get payment details** - [`GET:/ecomm/v2/payments/order123abc/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getPaymentDetailsUsingGET)

```json
{
    "orderId": "order123abc",
    "transactionLogHistory": [
        {
            "amount": 20000,
            "transactionText": "One pair of Vipps socks",
            "transactionId": "5001420062",
            "timeStamp": "2018-11-14T15:17:30.684Z",
            "operation": "INITIATE",
            "requestId": "",
            "operationSuccess": true
        }
    ]
}
```

## Reserve

When the user confirms, the payment status changes to `RESERVE`.
The respective amount will be reserved for future capturing.



**Example Get order status** - [`GET:/ecomm/v2/payments/order123abc/status`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getOrderStatusUsingGET)  |

```json
{
    "orderId": "order123abc",
    "transactionInfo": {
        "amount": 20000,
        "status": "RESERVE",
        "transactionId": "5001420062",
        "timeStamp": "2018-11-14T15:44:26.590Z"
    }
}
```

**Example Get payment details** - [`GET:/ecomm/v2/payments/order123abc/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getPaymentDetailsUsingGET)

```json
{
    "orderId": "order123abc",
    "transactionSummary": {
        "capturedAmount": 0,
        "remainingAmountToCapture": 20000,
        "refundedAmount": 0,
        "remainingAmountToRefund": 0
    },
    "transactionLogHistory": [
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

## Cancel

Reservations can be cancelled, and the payment flow aborted, under certain circumstances:

* When the user cancels (rejects) the initiated payment in the Vipps app.
* Timeouts: If the user does not confirm, etc.

Partially captured reservations can not be cancelled.

After cancellation, the order gets a new status:
* If an order is cancelled by the merchant, it gets the status `VOID`.
* If an order is cancelled by the user, it gets the status `CANCEL`.

**Request**

[`PUT:/ecomm/v2/payments/order123abc/cancel`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/cancelPaymentRequestUsingPUT)

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

**Response**

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

**Example Get order status** - [`GET:/ecomm/v2/payments/order123abc/status`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getOrderStatusUsingGET)  |

```json
{
    "orderId": "order123abc",
    "transactionInfo": {
        "amount": 20000,
        "status": "VOID",
        "transactionId": "5001420063",
        "timeStamp": "2018-11-14T15:46:07.498Z"
    }
}
```

**Example Get payment details** - [`GET:/ecomm/v2/payments/order123abc/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getPaymentDetailsUsingGET)

```json
{
    "orderId": "order123abc",
    "transactionLogHistory": [
        {
            "amount": 20000,
            "transactionText": "One pair of Vipps socks",
            "transactionId": "5001420061",
            "timeStamp": "2018-11-14T15:31:09.946Z",
            "operation": "VOID",
            "requestId": "",
            "operationSuccess": true
        },
        {
            "amount": 20000,
            "transactionText": "One pair of Vipps socks",
            "transactionId": "5001420061",
            "timeStamp": "2018-11-14T15:30:55.467Z",
            "operation": "RESERVE",
            "requestId": "",
            "operationSuccess": true
        },
        {
            "amount": 20000,
            "transactionText": "One pair of Vipps socks",
            "transactionId": "5001420061",
            "timeStamp": "2018-11-14T15:30:41.002Z",
            "operation": "INITIATE",
            "requestId": "",
            "operationSuccess": true
        }
    ]
}
```

**Example Get payment details** - [`GET:/ecomm/v2/payments/order123abc/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getPaymentDetailsUsingGET)

Example if end user reject the payment request:
```json
{
    "orderId": "order123abc",
    "transactionInfo": {
        "amount": 2000,
        "status": "CANCEL",
        "transactionId": "5001420067",
        "timeStamp": "2018-11-14T16:18:57.393Z"
    }
}
```

## Capture

Capture payment allows the merchant to capture the reserved amount.
The API allows for both a _full amount capture_ and a _partial amount capture_.

The amount to capture cannot be higher than the reserved amount. According to Norwegian regulations, capture can not be done before the goods have been shipped. The transaction text is mandatory, and is displayed to end user in Vipps.

Partial capture may be used if not the entire order is shipped or for other reasons. Partial capture may be called as many times as required as long as there is a remaining reserved amount to capture.

There is only a need to specify the `amount` when doing a partial capture. To perform capture of the entire amount `amount` can be set to `null` or `0`

In a capture request the merchant may also use the `X-Request-Id`header. This header is an idempotency header ensuring that if the merchant retries a request with the same `X-Request-Id` the retried request will not make additional changes.

**Request**

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

**Response**

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

**Example Get order status** - [`GET:/ecomm/v2/payments/order123abc/status`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getOrderStatusUsingGET)  |

```json
{
    "orderId": "order123abc",
    "transactionInfo": {
        "amount": 20000,
        "status": "RESERVE",
        "transactionId": "5001420062",
        "timeStamp": "2018-11-14T15:22:46.736Z"
 }
}
```

**Example Get payment details** - [`GET:/ecomm/v2/payments/order123abc/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getPaymentDetailsUsingGET)

```json
{
    "orderId": "order123abc",
    "transactionSummary": {
        "capturedAmount": 20000,
        "remainingAmountToCapture": 0,
        "refundedAmount": 0,
        "remainingAmountToRefund": 20000
    },
    "transactionLogHistory": [
        {
            "amount": 20000,
            "transactionText": "Socks on the way! Tracking code: abc-tracking-123",
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

## Refund

The merchant can initiate a refund of the captured amount.
The refund can be a partial or full.

Partial refunds are done by specifying an `amount` which is lower than the captured amount.
The refunded amount cannot be larger than the captured amount.

In a capture request the merchant may also use the `X-Request-Id`header. This header is an idempotency header ensuring that if the merchant retries a request with the same `X-Request-Id` the retried request will not make additional changes.

**Request**

[`POST:/ecomm/v2/payments/order123abc/refund`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/refundPaymentUsingPOST)

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

**Response**

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

**Example Get order status** - [`GET:/ecomm/v2/payments/order123abc/status`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getOrderStatusUsingGET)

```json
{
    "orderId": "order123abc",
    "transactionInfo": {
        "amount": 20000,
        "status": "RESERVE",
        "transactionId": "5001420062",
        "timeStamp": "2018-11-14T15:22:46.736Z"
 }
}
```

**Example Get payment details** - [`GET:/ecomm/v2/payments/order123abc/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getPaymentDetailsUsingGET)

```json
{
    "orderId": "order123abc",
    "transactionSummary": {
        "capturedAmount": 20000,
        "remainingAmountToCapture": 0,
        "refundedAmount": 20000,
        "remainingAmountToRefund": 0
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

# HTTP response codes

This API returns the following HTTP statuses in the responses:

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
| `429 Too Many Requests` | There is currently a limit of max 200 calls per second\* |
| `500 Server Error`      | An internal Vipps problem.                              |

HTTP requests that are being stopped in the application gateway will result in an error JSON object, while requests that are produced from the backend will receive an array with a JSON object. Error codes that are produced from the application gateway include 401, 403 and 422.

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


## Exception handling

Every system, especially those that includes complex integrations and/or
participation of many users, is prone to unexpected conditions. Below section
explains how Vipps handles different exception and error situations in detail.

### Connection timeout

Defining a socket timeout period is the common measure to protect server
resources and is expected. However, the time needed to fulfill a service requests
depends on several systems, which impose longer timeout period than usually
required.

We recommend setting no less than 1 second socket connection timeout
and 5 seconds socket read timeout while communicating with Vipps.

A good practice is, if/when the socket read timeout occurs, to call
[`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getPaymentDetailsUsingGET)
and check status of last transaction in transaction history prior
to executing the service call again.

### Callback aborted/interrupted

If the communication is broken during payment process for some reason, and
Vipps is not able to execute callback, then callback will not be retried.
In other words, if the merchant doesn’t receive any confirmation on payment
request call within callback timeframe, merchant should call get payment
details service to get the response of payment request.

### PSP connection issues

In a case when Vipps experiences communication problems with PSP, service call
will respond with 402 HTTP Error. Merchant should make a call to Get Payment
Details to check if the transaction request is processed before making service
call (with same idempotency key) again.


## Error groups

| Error groups      | Description |
| ----------------- | ----------- |
| Authentication    | Authentication Failure because of wrong credentials provided  |
| Payment   |  Failure while doing a payment authorization |
| InvalidRequest  |  Request contains invalid parameters |
| VippsError  |  Internal Vipps application error |
| user  | Error raised because of Vipps user (Example: The user is not a Vipps user)  |
| Merchant   | Errors regarding the merchant  |

## Error codes

| Error groups | Error Code | Error Message |
| ----- | ---- | ----------- |
| Payment | 41 | User don’t have a valid card |
| Payment | 42 | Refused by issuer bank |
| Payment | 43 | Refused by issuer bank because of invalid a amount |
| Payment | 44 | Refused by issuer because of expired card |
| Payment | 45 | Reservation failed for some unknown reason |
| Payment | 51 | Can't cancel already captured order |
| Payment | 52 | Cancellation failed |
| Payment | 53 | Can’t cancel order which is not reserved yet |
| Payment | 61 | Captured amount exceeds the reserved amount ordered |
| Payment | 62 | The amount you tried to capture is not reserved |
| Payment | 63 | Capture failed for some unknown reason, please use Get Payment Details API to know the exact status |
| Payment | 71 | Cant refund more than captured amount |
| Payment | 72 | Cant refund for reserved order, please use Cancel API |
| Payment | 73 | Can't refund on cancelled order |
| Payment | 93 | Captured amount should be same in Idempotent retry |
| InvalidRequest  | Field name will be the error code | Description about what exactly the field error is |
| VippsError | 91 | Transaction is not allowed |
| VippsError | 92 | Transaction already processed |
| VippsError | 98 | Too many concurrent requests |
| VippsError | 99 | Description of the internal error |
| user | 81 | User not registered with Vipps |
| user | 82 | User App version not supported |
| Merchant | 31 | Merchant is blocked because of <reason> |
| Merchant | 32 | Receiving limit of merchant is exceeded |
| Merchant | 33 | Number of payment requests has been exceeded |
| Merchant | 34 | Unique constraint violation of the order id |
| Merchant | 35 | Registered order not found |
| Merchant | 36 | Merchant agreement not signed |
| Merchant | 37 | Merchant not available, deactivated or blocked |
| Merchant | 21 | Reference Order ID is not valid |
| Merchant | 22 | Reference Order ID is not in valid state |

# App integration

Merchants may implement “app-switch” or “deeplinking” to trigger the Vipps app.
This may be done in two ways:

1. From a mobile or desktop browser
2. From an iOS or Android app

The sections below explain, in detail, how to integrate for browsers and apps.

## App-switch between mobile or desktop browsers and the Vipps app

For mobile and desktop browsers, integration is handled by Vipps using the Vipps landing page.

The merchant needs to provide a valid `fallBack`.
When Vipps has completed the operation, the `fallbackURL` will be opened in the browser.
To maintain the session, the merchant can pass along a session identifier through `fallbackURL`.

### App-switch on iOS

1. The Vipps app on iOS requires a URL scheme in order to support app-switch.
2. The merchant need to pass the URI Scheme of app into `fallbackURL` the in Vipps backend API.
3. The merchant will open the URL received from Vipps backend API.
4. Once the operation in the Vipps app is completed, Vipps will open the URL specified in `fallbackURL`.
5. From the Vipps mobile application appropriate status code will be appended to `fallbackURL` as a query string, such as `merchantApp://result?myAppData&status=301`.

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
the URL scheme is the same for both. A workaround is to only have one of the apps installed on the device.

#### Redirect back to the merchant app from Vipps app

Once the operation in the Vipps app is completed, the Vipps app will open the frontend URL.
For app-to-app integration, merchant app needs to be registered for a URL scheme
and pass the URL scheme in `fallbackURL` in the Vipps backend API.
The Vipps mobile application will use the URL to launch the merchant application.

For example, if your `fallbackURL` is `merchantApp://result?myAppData`, Vipps
will append the status like: `merchantApp://result?myAppData&status=301`.

##### Registering a 3rd party app with URL scheme and handling custom URL calls

See Apple's documentation:
[Defining a Custom URL Scheme for Your App](https://developer.apple.com/documentation/uikit/core_app/allowing_apps_and_websites_to_link_to_your_content/defining_a_custom_url_scheme_for_your_app)

### App-switch on Android

Vipps supports two ways to do app-switch:

* Android intent, using “startActivityForResult”. In order to use this the merchant need to set a `fallbackURL` as “INTENT”. In this way of communication there is no need to register for URL scheme.

* URL scheme: The app needs to be registered for URL scheme, and then pass the URL scheme in `fallbackURL`.

#### App-switch: Android Intent

In case of Android Intent system, in backend API call(defined later) “INTENT”
should be passed in fallbackURL. And below code should be used to launch Vipps application.

```java
try {
  PackageManager pm = context.getPackageManager();
  PackageInfo info = pm.getPackageInfo( , PackageManager.GET_ACTIVITIES);
  if(versionCompare(info.versionName,   ) >= 0) {
    String uri = deeplinkURL; // Use deeplink url provided in API response
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

Example of a deeplinkURL:
`vipps://?token=eyJraWQiOiJqd3RrZXkiLCJhbGciOiJSUzI1NiJ9.ey <snip>`

##### Redirect back to merchant app

Register the activity in manifest file which will handle result of Vipps response. For Example:

```html
<activity
  android:name=".MainActivity"
  android:label="@string/app_name">
</activity>
```

Receiving activity has to override onActivityResult method to handle result sent by Vipps application.
For Example:

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
  if (resultCode == RESULT_OK) {
    if (requestCode == 1) {
      String url = null;
      if (data != null && data.getExtras() != null) {
        Bundle mBundle = data.getExtras();
        if (mBundle.get("data") != null) {
          try {
            url = URLDecoder.decode(mBundle.get("data").toString(), "UTF-8"); Uri parseUri = Uri.parse(url);
            String status = parseUri.getQueryParameter("status");
            //Handle status
          } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
          }
        }
      }
    }
  }
}
```



#### App-switch: Android URL Scheme

```java
try {
  PackageManager pm = context.getPackageManager();
  PackageInfo info = pm.getPackageInfo , PackageManager.GET_ACTIVITIES);
  if (versionCompare(info.versionName,   ) >= 0) {
    String uri = deeplinkURL; // Use deeplink url provided in API response
    Intent intent = new Intent(Intent.ACTION_VIEW);
    intent.setData(Uri.parse(uri));
    startActivity(intent);
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

Example of a `deeplinkURL`:
`vipps://?token=eyJraWQiOiJqd3RrZXkiLCJhbGciOiJSUzI1NiJ9.ey <snip>`

##### Redirect back to merchant app

Set a filter in the Manifest file: To receive a call back from the Vipps application
to an activity, a filter has to be set for that activity. In the example
below, `MainActivity` is the receiving activity and the Vipps app sends a
response to the activity. For this activity one can set a custom URL scheme
inside the intent filter.

For Example:

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

Note: The scheme should be same specified in `fallbackURL`.

The Vipps application will send the result to the merchant app by
starting a new activity with the `fallbackURL` as a URI parameter in the intent.
The merchant app can make their receiving activity as a `singleInstance`
to handle the response in same activity.

The receiving activity has to override the `onNewIntent` method to handle
result send by the Vipps app.

```java
@Override
protected void onNewIntent(Intent intent) {
  super.onNewIntent(intent);
  String url = null;
  if (intent != null && intent.getData() != null) {
    try {
      url = URLDecoder.decode(intent.getData().toString(),"UTF-8");
      Uri parseUri = Uri.parse(url);
      String status = parseUri.getQueryParameter("status");
      //Handle status
    } catch(UnsupportedEncodingException e) {
      e.printStackTrace();
    }
  }
}
```

## Error codes for deeplinking

The following are the identified status codes merchant may receive from Vipps app.

| Status Code	| Description |
| ----------- | ----------- |
|100 |	Success |
|302 |	User doesn’t have Vipps profile |
|303 |	Login failed (login max attempt reached) |
|304 |	Vipps doesn’t support this action, please update Vipps |
|401 |	Request timed out or Token has expired |
|451 |	The user was selected for fraud validation |
|999 |	Failed |

The following are the status code ranges which Vipps maintains for future purposes.

| Status Code	| Description |
| ----------- | ----------- |
| 1XX | Success |
| 200 - 250 | Input Error |
| 250 - 299 | User Actions |
| 3XX | Authentication / User Profile / Merchant Profile / Configuration related error |
| 400 - 450 | Transaction related error |
| 450 - 499 | Fraud related error |
| 5XX | Reserved for future use |
| 6XX | Reserved for future use |
| 7XX | Reserved for future use |
| 8XX | Reserved for future use |
| 9XX | Other |

# API endpoints required by Vipps from the merchant

The following endpoints are to be implemented by merchants, in order for Vipps to make calls to them.
The documentation is included in the Swagger file for reference only - these endpoints are _not_ callable at Vipps.

## Vipps callback servers

The callbacks from Vipps are made from the following servers:

**Production environment:**\
callback-1.vipps.no\
callback-2.vipps.no\
callback-3.vipps.no\
callback-4.vipps.no

**Test environment:**\
callback-mt-1.vipps.no\
callback-mt-2.vipps.no\
callback-mt-3.vipps.no\
callback-mt-4.vipps.no

Please make sure that requests from these servers are allowed through firewalls, etc. 

**Note:** Vipps may change the IP addresses that we make callbacks from. To ensure that you are whitelisting the corrects IP addresses please use these hostnames.  

## Callback endpoints

The calls are generated by taking a field from the payment initiation request from the merchant in
[`POST:/ecomm/v2/payments/{orderId}`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/transactionUpdateCallbackForRegularPaymentUsingPOST),
amnd then adding a suffix. See the "Endpoints required by Vipps" section in the [Swagger documentation](https://vippsas.github.io/vipps-ecom-api/#/).
Note some of these endpoints are only for the express checkout product.

| Endpoint	| Field | Description |
| ----------- | ----------- | ----------- |
| [`POST:[callbackPrefix]/v2/payments/{orderId}`](https://vippsas.github.io/vipps-ecom-api/#/Endpoints_required_by_Vipps_from_the_merchant/transactionUpdateCallbackForRegularPaymentUsingPOST) | callbackPrefix | Status update sent from Vipps |
| [`POST:[shippingDetailsPrefix]/v2/payments/{orderId}/shippingDetails`](https://vippsas.github.io/vipps-ecom-api/#/Endpoints_required_by_Vipps_from_the_merchant/fetchShippingCostUsingPOST) | shippingDetailsPrefix | Request for getting shipping details. (Express checkout functionality, 10 second timeout) |
| [`DELETE:[consentRemovalPrefix]/v2/consents/{userId}`](https://vippsas.github.io/vipps-ecom-api/#/Endpoints_required_by_Vipps_from_the_merchant/removeUserConsentUsingDELETE) | consentRemovalPrefix | User has removed consent for sharing details. (Express checkout functionality) |

## Callback

Callbacks allow Vipps to send the payment order details to the merchant.
During regular eCommerce payment, order and transaction details will be shared.
During express checkout payment it will provide user details and shipping
details addition to the order and transaction details.

This call will be performed once during a payment process, when the
payment is successful, failed, rejected or timed out. If the communication
is broken during the process for some reason, and Vipps is not able to
execute callback, then callback will not be retried. In other words,
if the merchant doesn’t receive any confirmation on payment request
call within callback timeframe, merchant should call
[GET:/ecomm/v2/payments/{orderId}/details](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getPaymentDetailsUsingGET)
to get the status of the payment.

The callback body received from Vipps will depend on whether the payment
type is set to `"eComm Express Payment"` or `"eComm Regular Payment"`.

If `authToken` was used in the initiate payment request these requests will have a `Authorization` header. These headers will have the same value as the one provided in `authToken`.

API details: [`POST:[callbackPrefix]/v2/payments/{orderId}`](https://vippsas.github.io/vipps-ecom-api/#/Endpoints_required_by_Vipps_from_the_merchant/transactionUpdateCallbackForRegularPaymentUsingPOST)

**Note:** We do not send requests to all ports so to be safe use common port numbers such as: 80, 443, 8080.

**Example Express Checkout Callback**

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

**Example Regular Checkout Callback**

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


## Fetch Shipping Cost

This API call allows Vipps to get the shipping cost and method based on the provided address and product details. This is only relevant for express checkout payments where Vipps needs to present shipping cost and method to the Vipps user. This service is to be implemented by merchants.

If `authToken` was used in the initiate payment request these requests will have a `Authorization` header. These headers will have the same value as the one provided in `authToken`.

API details: [`POST:[shippingDetailsPrefix]/v2/payments/{orderId}/shippingDetails`](https://vippsas.github.io/vipps-ecom-api/#/Endpoints_required_by_Vipps_from_the_merchant/fetchShippingCostUsingPOST)

**Note:** We do not send requests to all ports so to be safe use common port numbers such as: 80, 443, 8080.

**Example Request Shipping Details Callback**
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

**Example Response for Shipping Details Callback**
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

## Remove User Consent

This allows Vipps to send an end user's consent removal request to merchant.
This endpoint is required for express checkout. When receiving this request,
the merchant is obliged to handle the user details as per the GDPR guidelines.
The request path will include a `userId` that Vipps will have provided as
part of callback, and also made accessible through [`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getPaymentDetailsUsingGET).

API details: [`DELETE:[consetRemovalPrefix]/v2/consents/{userId}`](https://vippsas.github.io/vipps-ecom-api/#/Endpoints_required_by_Vipps_from_the_merchant/removeUserConsentUsingDELETE)

**Note:** We do not send requests to all ports so to be safe use common port numbers such as: 80, 443, 8080.

# Status and operations

Callbacks and responses from the eCommerce API provide status updates
for the current state of the transaction. These will differ for
different requests and for different payment types.

## Responses from requests

| Request                              | Response "Status" or "Operation" |
| ------------------------------------ | -------------------------------- |
| [`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/initiatePaymentV3UsingPOST)                | `Initiate` - Merchant initiated the transaction.  |
| [`POST:/ecomm/v2/payments/{orderId}/capture`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/capturePaymentUsingPOST) | `Captured` - Payment Captured when merchant called for capture. |
| [`PUT:/ecomm/v2/payments/{orderId}/cancel`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/cancelPaymentRequestUsingPUT)  | `Cancelled` - Payment cancel status when merchant called for cancel. |
| [`POST:/ecomm/v2/payments/{orderId}/refund`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/refundPaymentUsingPOST)  | `Refund` - Payment refunded when merchant called for refund.  |
| [`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getPaymentDetailsUsingGET)  | `INITIATE` - Merchant initiated the transaction. Stage-1.
|   |  `RESERVE` - Payment Reserved by user accepting transaction in App. Stage-3. |
|   |  `SALE` - Payment Captured with direct capture. Stage-4. |
|   |  `CAPTURE` - Payment Captured when merchant called for capture - Stage-6. |
|   |  `REFUND` - Payment refunded when merchant called for refund - Stage-7. |
|   |  `CANCEL` - Payment cancel status when user canceled payment in App. |
|   |  `VOID` - Payment cancel status when merchant called for cancel. |
| [`GET:/ecomm/v2/payments/{orderId}/status`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getOrderStatusUsingGET)  | This information is currently inaccurate, please use [`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getPaymentDetailsUsingGET) |
|   |  `INITIATE`  -  Merchant initiates the transaction. Stage-1, Status - 101. |
|   |  `REGISTER`  - Vipps registers payment for Reserve. Stage-2, Stage-3, Status - 101.|
|   |  `RESERVE` -  Payment Reserved by user accepting transaction in App. Stage-3, Status - 101. |
|   |  `SALE` - Payment Captured with direct capture. Stage-4, status - 101. |
|   |  `CAPTURE` - Payment Captured when merchant called for capture - Stage-6, Status - 101. |
|   |  `REFUND` - Payment refunded when merchant called for refund - Stage-7, Status - 101. |
|   |  `CANCEL` - Payment cancel status when user canceled payment in App. |
|   |  `VOID` - Payment cancel status when merchant calls for cancel API. |
|   |  `FAILED` - Payment failed failed because of no coverage, outdated card details or similar. - Status - 102. |
|   |  `REJECTED` - No user action in the Vipps app, i.e timeout. |

## Callbacks

| Callback | Response "Status" or "Operation"    |
| -------------- | ----- |
| Callback for *regular* checkout: [`[callbackPrefix]/v2/payments/{orderId}`](https://vippsas.github.io/vipps-ecom-api/#/Endpoints_required_by_Vipps_from_the_merchant/transactionUpdateCallbackForRegularPaymentUsingPOST) | `RESERVED` - When user approved and payment is reserved. |
|   |  `SALE` - When user approved and payment type is Direct capture.  |
|   |  `RESERVE_FAILED` - Reserve failed because of no coverage, outdated card details or similar. |
|   |  `SALE_FAILED` - Direct capture failed because of no coverage, outdated card details or similar. |
|   |  `CANCELLED` - User rejected the payment. |
|   |  `REJECTED` - User didn't act on the payment. |
| Callback for *express* checkout: [`[callbackPrefix]/v2/payments/{orderId}`](https://vippsas.github.io/vipps-ecom-api/#/Endpoints_required_by_Vipps_from_the_merchant/transactionUpdateCallbackForRegularPaymentUsingPOST)  | `RESERVE` - When user approved and payment is reserved. (It _is_ correct that this is different from `RESERVED` for regular checkout.)|
|    | `SALE` - When user approved and payment type is Direct capture.  |
|    | `CANCELLED` - User rejected the payment.  |
|    | `REJECTED` - User didn't act on the payment.  |

# Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-ecom-api/issues),
a [pull request](https://github.com/vippsas/vipps-ecom-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).
