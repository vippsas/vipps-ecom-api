# Vipps eCommerce API

API version: 2.1.

Document version 0.2.0.

Please use GitHub's built-in functionality for
[issues](https://github.com/vippsas/vipps-invoice-api/issues),
[pull requests](https://github.com/vippsas/vipps-invoice-api/pulls),
or contact us at integration@vipps.no.

_**IMPORTANT:**_ This document is a work in progress. See the official PDF documentation:
https://github.com/vippsas/vipps-ecom-api/tree/master/original-docs
See also the [FAQ](vipps-ecom-api-faq.md).

# Table of contents

- [Overview](#overview)
    - [Use case scenarios](#use-case-scenarios)
    - [Regular eCommerce Payments](#regular-ecommerce-payments)
    - [Express Checkout Payments](#express-checkout-payments)
- [API calls flow](#api-calls-flow)
- [Authentication](#authentication)
    - [Access Token](#access-token)
- [Idempotency](#idempotency)
- [eCommerce Payment Flows](#ecommerce-payment-flows)
    - [Flow for push notification flow on desktop browser](#flow-for-push-notification-flow-on-desktop-browser)
    - [Flow for push notification flow on mobile](#flow-for-push-notification-flow-on-mobile)
    - [Vipps checkout flow chart](#vipps-checkout-flow-chart)
    - [Initiate](#initiate)
    - [Reserve](#reserve)
    - [Cancel](#cancel)
    - [Capture](#capture)
    - [Direct capture](#direct-capture)
    - [Refund](#refund)
    - [Order Status](#order-status)
- [Additional payment flow for express checkout](#additional-payment-flow-for-express-checkout)
    - [Get shipping cost & method](#get-shipping-cost--method)
    - [Transaction updates with user details](#transaction-updates-with-user-details)
    - [Remove user consent](#remove-user-consent)
    - [Exception handling 7.1 Introduction](#exception-handling-71-introduction)
    - [Exception scenarios](#exception-scenarios)
- [Response codes](#response-codes)
    - [Success Codes](#success-codes)
    - [HTTP Error Codes](#http-error-codes)
    - [Error Representation](#error-representation)
    - [Error codes](#error-codes)
- [Front-end Integration](#front-end-integration)
    - [Appswitch between Mobile or Desktop Browser and Vipps App](#appswitch-between-mobile-or-desktop-browser-and-vipps-app)
    - [Appswitch between Merchant’s Mobile App and Vipps App](#appswitch-between-merchants-mobile-app-and-vipps-app)
    - [List of error codes for deeplinking](#list-of-error-codes-for-deeplinking)
- [API-definitions](#api-definitions)
    - [Base URL](#base-url)
- [Initiate Payment](#initiate-payment)
    - [Payment triggered from desktop and mobile browser](#payment-triggered-from-desktop-and-mobile-browser)
    - [Payment triggered from Merchant Mobile App](#payment-triggered-from-merchant-mobile-app)
- [Cancel Payment](#cancel-payment)
- [Capture Payment](#capture-payment)
- [Refund Payment](#refund-payment)
- [Get Payment Details](#get-payment-details)
- [Get Order Status](#get-order-status)
- [Endpoints Hosted by Merchant](#endpoints-hosted-by-merchant)
- [Callback](#callback)
- [Fetch Shipping Cost](#fetch-shipping-cost)
- [Remove User Consent](#remove-user-consent)
- [Vipps eCommerce APIs](#vipps-ecommerce-apis)
- [Vipps Login APIs](#vipps-login-apis)
- [Vipps Signup API](#vipps-signup-api)
    - [Process overview](#process-overview)
    - [Partner initiates the signup](#partner-initiates-the-signup)
    - [v1/partial/signup](#v1partialsignup)
    - [Partner receives the signup link](#partner-receives-the-signup-link)
    - [Partial Signup Response](#partial-signup-response)
    - [The signup form, KYC and signing process](#the-signup-form-kyc-and-signing-process)
    - [The email notification](#the-email-notification)
- [Vipps Signup API](#vipps-signup-api-1)
- [Email Specification](#email-specification)

# Overview

The Vipps eCommerce API (eCom API) offers functionality for online payments,
both on websites and in native apps.

Vipps may be integrated at merchant’s checkout page as one of the payment
methods, or in native apps using app-switching.

_TODO: Insert diagram with overall flow: Initiate-Reserved-Captured-etc._

# API overview

| Operation           | Description         | Endpoint          |
| ------------------- | ------------------- | ----------------- |
| Initiate payment    | Payment initiation, the first request in the payment flow. This _reserves_ an amount. | [`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/initiatePaymentV3UsingPOST)  |
| Capture payment     | When an amount has been reserved, and the goods are (about to be) shipped, the payment must be _captured_  | [`POST:/ecomm/v2/payments/{orderId}/capture`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/capturePaymentUsingPOST)  |
| Cancel payment      | The merchant may cancel a reserved amount, but not on a captured amount.  | [`PUT:/ecomm/v2/payments/{orderId}/cancel`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/cancelPaymentRequestUsingPUT)  |
| Refund payment      | The merchant may refund a captured amount.  |[`POST:/ecomm/v2/payments/{orderId}/refund`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/refundPaymentUsingPOST)  |
| Get order status    | The status is "reserved" after a payment has been initiated. For details about payment, use [`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getPaymentDetailsUsingGET) | [`GET:/ecomm/v2/payments/{orderId}/status`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getOrderStatusUsingGET)  |
| Get payment details | How much of the reserved amount has been captured, etc.  | [`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getPaymentDetailsUsingGET)  |

See [Payment flows](#payment-flows) for complete `requests` and `responses`.

## Endpoints required by Vipps from the merchant for express checkout

For [Vipps Hurtigkasse](https://www.vipps.no/bedrift/vipps-pa-nett/hurtigkasse),
the endpoints below are required on the merchant side.
These endpoints are included in the Swagger file for reference only, and are
_not_ Vipps endpoints that may be called by merchants.

| Operation           | Description         | Endpoint          |
| ------------------- | ------------------- | ----------------- |
| Remove user consent | Used to inform merchant when the Vipps user removes consent to share information.  | [`DELETE:/v2/consents/{userId}`](https://vippsas.github.io/vipps-ecom-api/#/Calls_from_Vipps_examples/removeUserConsentUsingDELETE)  |
| Callback : Transaction Update | A callback to the merchant for receiving post-payment information. | [`POST:/ecomm/v2/payments/{orderId}`](https://vippsas.github.io/vipps-ecom-api/#/Calls_from_Vipps_examples/transactionUpdateCallbackForRegularPaymentUsingPOST)  |
| Get shipping cost and method | [`POST:/v2/payments/{orderId}/shippingDetails`](https://vippsas.github.io/vipps-ecom-api/#/Calls_from_Vipps_examples/fetchShippingCostUsingPOST)  |   |

See [Payment flows](#payment-flows) for complete `requests` and `responses`.

# HTTP responses

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

All error responses contains an `error` object in the body, with details of the problem.
See [Error details](#error-details) for more information.

# Payment types

## Regular eCommerce payments

Vipps supports both _reserve-capture_ and _direct capture_. Please note that
reserver-capture is the normal flow, and that there are additional regulatory
requirements and compliance checks needed for merchants using direct capture.
See the Consumer Authority's
[Guidelines for the standard sales conditions for consumer purchases of goods over the internet](https://www.forbrukertilsynet.no/english/guidelines/guidelines-the-standard-sales-conditions-consumer-purchases-of-goods-the-internet).

Vipps detects whether user is using a desktop browser or a mobile browser,
and - if using a mobile browser - whether user has the Vipps app installed

If a desktop browser is used, Vipps displays the "Vipps landing page",
where the user enters the phone number. The user enters the phone number,
and the Vipps app prompts for confirmation on the phone.

Following are different API services that merchant can utilize as part of ecommerce payments:

## Express checkout payments

These are payments related to
[Vipps Hurtigkasse](https://www.vipps.no/bedrift/vipps-pa-nett/hurtigkasse),
where Vipps reduces the typical purchase process to a few simple steps:

1. The user clicks on the "Vipps Hurtigkasse" button
2. The user enters phone number on the web page
3. The user is prompted by the Vipps app to consent to share name and address for shipping
4. The user confirms the purchase in the Vipps app
5. The merchant receives shipping information, completes purchase and shows a confirmation page

Vipps complies with GDPR, and requires the3 user's consent before any information
is shared with the merchant. The merchant must provide a link (`consentRemovalPrefix`)
that Vipps can call to delete the data. The Vipps app gives the Vipps user an
overview of "Companies with access", where the user can manage the consents.

# API calls flow

This diagram shows the flow between the merchant and Vipps:

![API calls flow](images/api-calls-flow.png)

For details about the Vipps Developer Portal and API keys, see the [Getting Started guide](vipps-ecom-api-getting-started.md).


### Postman

[Postman](https://www.getpostman.com/)
is a common tool for working with REST APIs. We offer a
[Postman Collection](https://www.getpostman.com/collection)
for Vipps Regninger to make development easier.
See the
[Postman documentation](https://www.getpostman.com/docs/)
for more information about using Postman.

By following the steps below, you can make calls to all the
endpoints, and see the full `request` and `response` for each call.

#### Setting up Postman

##### Step 1: Import the Postman Collection

1. Click `Import` in the upper left corner.
2. Import the [vipps-ecom-api-postman-collection.json](https://github.com/vippsas/vipps-ecom-api/blob/master/tools/vipps-ecom-api-postman-collection.json) file

##### Step 2: Import the Postman Environment

1. Click `Import` in the upper left corner.
2. Import the [vipps-ecom-api-postman-enviroment.json](https://github.com/vippsas/vipps-ecom-api/blob/master/tools/vipps-ecom-api-postman-enviroment.json) file

##### Step 3: Setup Postman Environment

1. Click the "eye" icon in the top right corner.
2. In the dropdown window, click `Edit` in the top right corner.
3. Fill in the `Current Value` for the following fields to get started.
   - `access-token-key`
   - `subscription-key`
   - `client-id`
   - `client-secret`

See the Vipps Developer
[Getting started guide](https://github.com/vippsas/vipps-developers/blob/master/vipps-developer-portal-getting-started.md#step-4)
for details on where to find the API keys.

## Authentication

All API calls are authenticated and authorized based on the application access
token (JWT bearer token) and a subscription key (`Ocp-Apim-Subscription-Key`),
and these headers are required:

| Header Name | Header Value | Description |
| ----------- | ------------ | ----------- |
|  `Authorization` | `Bearer <JWT access token>` | Type: Authorization token. Value: Access token is obtained by registering merchant backend application in Merchant Developer Portal. |
| `Ocp-Apim-Subscription-Key` | Base 64 encoded string | Subscription key for eCommerce product. This can be found in User Profile page on Merchant developer portal after merchant account is created |

### Access token

The Access Token API provides the JWT bearer token.
The `clinet_id` and `client_secret` are neded to get a JWT access token.
These are found in the Vipps Developer Portal - see the  
[Getting Started guide](vipps-ecom-api-getting-started.md).

| Header Name | Header Value | Description |
| ----------- | ------------ | ----------- |
| `client_id` | A GUID value | Client ID received when merchant registered the application |
| `client_secret` | Base 64 encoded string | Client Secret received when merchant registered the application |
| `Ocp-Apim-Subscription-Key` | Base 64 encoded string | Subscription key for eCommerce product. This can be found in User Profile page on Merchant developer portal after merchant account is created |

#### Request

```http
POST https://apitest.vipps.no/accessToken/get
client_id: <ClientID>
client_secret: <ClientSecret>
Ocp-Apim-Subscription-Key: <Ocp-Apim-Subscription-Key>
```

#### Response

```http
HTTP 200 OK
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

| Name                        | Description                                 |
| --------------------------- | ------------------------------------------- |
| `Bearer`                    | It’s a `Bearer` token. The word `Bearer` should be added before the token, but this is optional and case insensitive in Vipps. |
| `expires_in`                | Token expiry duration in seconds. |
| `ext_expires_in`            | Extra expiry time. This is always zero. |
| `expires_on`                | Token expiry time in epoch time format. |
| `not_before`                | Token creation time in epoch time format. |
| `resource`                  | For the product for which token has been issued. |
| `access_token`              | The actual access token that needs to be used in `Authorization` request header. |

**Please note:** The access token is valid for 24 hours.

##### HTTP response codes

This API returns the following HTTP statuses in the responses:

| HTTP status         | Description                                 |
| ------------------- | ------------------------------------------- |
| `200 OK`            | Request successful.                          |
| `400 Bad Request`   | Invalid request, see the `error` for details.  |
| `401 Unauthorized`  | Invalid credentials.                         |
| `403 Forbidden`     | Authentication ok, but credentials lacks authorization.  |
| `500 Server Error`  | An internal Vipps problem.                  |

Error reponse body (formatted for readability):

```json
{
  "error": "unauthorized_client",
  "error_description": "AADSTS70001: Application with identifier 'e9b6c99d-2442-4a5d-84a2-\
                        c53a807fe0c4' was not found in the directory testapivipps.no\
                        Trace ID: 3bc2b2a0-d9bb-4c2e-8367- 5633866f1300\r\nCorrelation ID:\
                        bb2f4093-70af-446a-a26d-ed8becca1a1a\r\nTimestamp: 2017-05-19 09:21:28Z",
  "error_codes": [ 70001 ],
  "timestamp": "2017-05-19 09:21:28Z",
  "trace_id": "3bc2b2a0-d9bb-4c2e-8367-5633866f1300",
  "correlation_id": "bb2f4093-70af-446a-a26d-ed8becca1a1a"
}
```

See the Swagger documentation for more details:
[`POST:/accesstoken/get`](https://vippsas.github.io/vipps-ecom-api/#/Authorization_Service/fetchAuthorizationTokenUsingPost)

# Payment flows and push notifications

There are separate payment and push notification flows for desktop and mobile browsers.

## Flow for push notification flow on desktop browser

![API calls flow: Push for desktop](images/api-calls-flow-push-desktop.png)

## Flow for push notification flow on mobile

![API calls flow: Push for mobile](images/api-calls-flow-push-mobile.png)

## Vipps checkout flow chart

![Vipps checkout flow chart](images/vipps-ecom-flow-chart.svg)

| #   | From       | To         | Description                                      |
| --- | ---------- | ---------- | -----------------------------------------------------------------------------------|
| 1   | `initiate` | `reserve`  | Customer clicks Vipps button in chart and confirm                                    payment in Vipps app. Merchant reserve payment.  | | -   |            | `cancel`   | Customer cancel order                            |
| 2   | `reserve`  | `capture`  | Merchant capture payment and ship goods          |
| -   |            | `cancel`   | Merchant cancel order                            |
| 3   | `capture`  | --         | Final state                                      |
| -   |            | `refund`   | Merchant refund money to customer                |
| 4   | `cancel`   | --         | Final state                                      |
| 5   | `refund`   | --         | Final state                                      |

## Initiate

The first call in the payment flow initiates a payment request that requires
confirmation from the customer (end user). The payment gets the status
`Initiated` and the customer is notified about payment request in mobile app.

If customer does not confirm in the Vipps app within five minutes, the
payment request times out and the payment flow stops.

The paym,ent initiation call
([`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/initiatePaymentV3UsingPOST))
must include the `paymentType` parameter, which specifies whether the payment
is a regular eCommerce payment or an Express payment (Vipps Hurtigkasse).

API details:
[`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/initiatePaymentV3UsingPOST)

### Request

_TODO: Replace with actual, working example from the Postman collection, and link to Swagger._

```json
{
  "customerInfo": {
    "mobileNumber": "Values with length 8"
  },
  "merchantInfo": {
    "authToken": "Values with length 255",
    "callbackPrefix": "Values with length 255",
    "consentRemovalPrefix": "Values with length 255",
    "fallBack": "Values with length 255",
    "isApp": false,
    "merchantSerialNumber": "NSBWSHP12",
    "paymentType": "string",
    "shippingDetailsPrefix": "Values with length 255"
  },
  "transaction": {
    "amount": 0,
    "orderId": "Values with length 30",
    "refOrderId": "Values with length 30",
    "timeStamp": "2018-08-25T11:46:25.386Z",
    "transactionText": "Values with length 100"
  }
}
```

### Response

_TODO: Replace with actual, working example from the Postman collection, and link to Swagger._

```json
HTTP 202 Accepted
{
  "orderId": "string",
  "url": "string"
}
```

## Reserve

When the customer successfully authorizes the payment request by using Vipps app, the payment status changes to Reserved, and the respective amount will be reserved for future capturing at the PSP.

_TODO: Include an actual, working example from the Postman collection, and link to Swagger._

## Cancel

_TODO: Include an actual, working example from the Postman collection, and link to Swagger._

Reservations can be cancelled and payment flow aborted under certain circumstances:

* When user cancels the initiated payment then the payment status shown will be cancelled

* When Merchant call cancellation of the reservation. Please note that partially captured reservations can’t be cancelled.

* When there is a timeout of the payment confirmation by several of reasons (no action by the customer, notification to user is delayed, etc.)

* When reservation fails caused by system or communication error.

Jump to [Cancel Payment](#cancel-payment)

## Capture

When merchant shipped the goods then they can call capture API on the reserved transaction. The API allows to do a full amount capture or partial amount capture.

Jump to [Capture Payment](#capture-payment)

## Direct capture

Direct capture is not depicted on diagram above but, in essence, combines two steps (reserve and capture) in one. This is a configuration in Vipps backend when Initiate payment request is sent.

Jump to [Capture Payment](#capture-payment)

## Refund

Merchant can initiate a refund of the captured amount. The refund can be a partial or full.

Jump to [Refund Payment](#refund-payment)

## Order Status

Get Order Status intention is to check whether the user is authenticated the transaction or not. Possible status provided by this service is listed below:

Jump to [Get Order Status](#get-order-status)

# Additional payment flow for express checkout

In addition to above mentioned payment flows, following are the services which merchants should build on their side to support express checkout during online purchases.

## Get shipping cost & method

When express checkout payment is initiated, vipps will call this service from merchant’s backend to fetch shipping cost and shipping method related details. Merchant can send priority of shipping cost and method combination if there are multiple ways of delivery. Merchant can also send default shipping cost & method combination which merchant wants user to see on payment confirmation screen of Vipps. Vipps will support upto 10 shipping cost and method combinations. If user sends more than 10 combinations, vipps will display first 10 always.

Jump to get

## Transaction updates with user details

After express checkout payment is processed, vipps will make a call back to merchant stating payment details, shipping details and user details (optional).

## Remove user consent

When consent to store/process/view details of vipps user is removed by user in vipps app, vipps will make a call to merchant informing the same. Merchant is obliged to delete user information upon receiving this request.

## Exception handling 7.1 Introduction

Every system, especially those that includes complex integrations and/or participation of many users, is prone to unexpected conditions. Below section explains how Vipps handles different exception and error situations in detail.

## Exception scenarios

The most critical action in payment flow is when Initiate Payment service call is invoked. The Flow diagram below shows how to successfully fulfil service call, communication between several contributors and users across several systems has to work flawlessly.

![Payment flow eCommerce – Payment request](images/payment-flow-for-ecommerce-payment-request.png)

To cope with possible communication problems/errors, several scenarios and guidelines are developed.

### Connection timeout

Defining a socket timeout period is the common measure to protect server resources and is expected. However, the time needed to fulfill a service requests depends on several systems, which impose longer timeout period than usually required. We recommend setting no less than 1 second socket connection timeout and 5 seconds socket read timeout while communicating with Vipps.
A good practice is, if/when the socket read timeout occurs call Get Payment Details and check status of last transaction in transaction history prior executing the service call again.

### Callback aborted/interrupted

If the communication is broken during payment process for some reason, and Vipps is not able to execute callback, then callback will not be retried.
In other words, if the merchant doesn’t receive any confirmation on payment request call within callback timeframe, merchant should call get payment details service to get the response of payment request.

### PSP connection issues

In a case when Vipps experiences communication problems with PSP, service call will respond with 402 HTTP Error. Merchant should make a call to Get Payment Details to check if the transaction request is processed before making service call (with same idempotency key) again.

# Response codes

Vipps eCommerce API uses standard HTTP response codes to indicate the success and failure
of the request as defined in [RFC2616](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html).

(_**TODO: This section should refer to Swagger spec**_)


## Error details

HTTP responses with HTTP `4XX` and `5XX` error codes contain an `error` object:

```
[
  {
    "errorGroup": "Payment",
    "errorCode": "44"
    "errorMessage": "Refused by issuer because of expired card",
 }
]
```

## Error groups

| Error groups      | Description |
| ----------------- | ----------- |
| Authentication    | Authentication Failure because of wrong credentials provided  |
| Payment   |  Failure while doing a payment authorization |
| InvalidRequest  |  Request contains invalid parameters |
| VippsError  |  Internal Vipps application error |
| Customer  | Error raised because of Vipps user (Example: The user is not a Vipps user)  |
| Merchant   | Errors regarding the merchant  |

## Error codes

| Group | Code | Description |
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
| Payment | 62 | Can't capture cancelled order |
| Payment | 63 | Capture failed for some unknown reason, please use Get Payment Details API to know the exact status |
| Payment | 71 | Cant refund more than captured amount |
| Payment | 72 | Cant refund for reserved order, please use Cancel API |
| Payment | 73 | Can't refund on cancelled order |
| InvalidRequest  | Field name will be the error code | Description about what exactly the field error is |
| VippsError | 91 | Transaction is not allowed |
| VippsError | 92 | Transaction already processed |
| VippsError | 98 | Too many concurrent requests |
| VippsError | 99 | Description of the internal error |
| Customer | 81 | User not registered with Vipps |
| Customer | 82 | User App version not supported |
| Merchant | 31 | Merchant is blocked because of <reason> |
| Merchant | 32 | Receiving limit of merchant is exceeded |
| Merchant | 33 | Number of payment requests has been exceeded |
| Merchant | 34 | Unique constraint violation of the order id |
| Merchant | 35 | Registered order not found |
| Merchant | 36 | Merchant agreement not signed |
| Merchant | 37 | Merchant not available, deactivated or blocked |
| Merchant | 21 | Reference Order ID is not valid |
| Merchant | 22 | Reference Order ID is not in valid state |

# Front-end Integration

Merchants need to implement “Appswitch” integration which is also called as “Deeplinking” to trigger Vipps app for serving ecommerce payment requests. This can happen in two ways:

1. From mobile or desktop browser
2. From mobile application

Below section explains how merchant can implement these integrations in detail.

### URL validation

URLs are validated with the
[Apache Commons UrlValidator](https://commons.apache.org/proper/commons-validator/apidocs/org/apache/commons/validator/routines/UrlValidator.html).

Here is a simple Java class suitable for testing, using the dummy URL `https://example.com/vipps/fallback?id=abc123`:

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

## App-switch between mobile or desktop browsers and the Vipps app

For mobile and desktop browsers, integration is handled by Vipps using the Vipps landing page.

The merchant needs to provide a valid `fallbackURL`.
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

### Appswitch on Android

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
            //TODO Handle status
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

Following is the code sample for Android URL scheme approach.

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

Set filter in the Manifest file: To receive a call back from the Vipps application
to an activity there has to be set a filter to that activity. In the example
below MainActivity is the receiving activity and Vipps application sends a
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

Note: scheme should be same as you send in `fallbackURL`.

The Vipps application will send the result to the 3rd party application by
starting a new activity with the fallbackURL as a URI parameter in the intent.
The 3rd party application can make their receiving activity as a singleInstance
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
      //TODO Handle status
    } catch(UnsupportedEncodingException e) {
      e.printStackTrace();
    }
  }
}
```

## Error codes for deeplinking

The following are the identified status codes merchant may receive from Vipps app.

_**TODO: Formatting.**_

```
Status Code	Description
100	Success
302	User doesn’t have Vipps profile
303	Login failed (login max attempt reached)
304	Vipps doesn’t support this action, please update Vipps
401	Request timed out or Token has expired
451	The user was selected for fraud validation
999	Failed
```

Below are the status code ranges which Vipps maintains for future purposes. For example, if there is new error message related to fraud, then it will fall under range 450 to 499.

_**TODO: Formatting**:_

```
1XX – Success Scenarios
200 to 250 – Input Error
250 to 299 - User Actions
3XX – Authentication / User Profile / Merchant Profile / Configuration related error
400 to 450 – Transaction related error
450 to 499 – Fraud related error
5XX – Reserved for future use 6XX – Reserved for future use
7XX – Reserved for future use 8XX – Reserved for future use
9XX – Others
```
# API definitions

| Header Name | Header Value | Optional | Description |
| ----------- | ------------ | -------- | ----------- |
| Authorization |	JWT Access Token <>	| No | type: Authorization token value: Access token is obtained by registering merchant backend application in Merchant Developer Portal. |
| Content-Type | application/json |	No | Type of the body |
| Ocp-Apim-Subscription-Key | Base 64 encoded string | No |	Subscription key for eCommerce product. This can be found in User Profile page on Merchant developer portal |
| X-TimeStamp	| Time stamp when the request called | Yes |	Time to call |
| X-Request-Id | To identify the idempotent request | Yes | For Making request to be idempotent this ID is must so that the system will not do any side effects. 1. Applicable to Initiate, Capture, Refund payment 2. Size should be 30 3. If user wants to re-try any failed capture or refund transaction then they should provide same X-request-id, else system will create a new entry for partial capture or partial refund. |

## Base URL

`https://apitest.vipps.no`

## deeplinkURL

Below is a complete example of a deeplinkURL:

```
vipps://?token=eyJraWQiOiJqd3RrZXkiLCJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJmMDE0MmIxYy02YjIwLTQ1M2MtYTlmMS1lMWUwZGFiNjkzOTciLCJhdWQiOiJmMDE0MmIxYy02YjIwLTQ1M2MtYTlmMS1lMWUwZGFiNjkzOTciLCJhenAiOiJmMDE0MmIxYy02YjIwLTQ1M2MtYTlmMS1lMWUwZGFiNjkzOTciLCJhcHBUeXBlIjoiTEFORElOR1BBR0UiLCJpc3MiOiJodHRwczpcL1wvdmlwcHMtdGVzdC1jb24tYWdlbnQtaWxiLnRlc3QudGVjaC0wMi5uZXRcL2F0M1wvZGVlcGxpbmstb3BlbmlkLXByb3ZpZGVyLWFwcGxpY2F0aW9uXC8iLCJhZ3JlZW1lbnRJZCI6ImFncl9UNmNMeGY0IiwiZXhwIjoxNTQxNDMwMzk5LCJ0b2tlblR5cGUiOiJERUVQTElOSyIsImlhdCI6MTU0MTQzMDI3OSwidXVpZCI6IjU2MTM0YjI3LWQ1ODAtNDdmZC1hOTExLWYzMGU0ZDY4NmNkZiIsImp0aSI6IjI2NzIyYzU5LWZlYTMtNGYwNi1hNGRiLTVhOGM3MDFjY2JkMCJ9.d_jDLUW7cZi6xF5N51CggymsiT0zQM36a2nEB8h7jQhCIE5BDG_6u0QmW4tbYiK8T_8TJNmPIQbJ1YY7gZiIY9kHD1fPQC6JlFS4FaUldOdq9yYesqXFcvnsZdzeRk45g9YuCdHSmmcYQ4Hzz3HurlZ_txiPybytOWbhm1hTKFsyQOC_1GZWA9SFdLG8g2z5ZMuCkyUlXgArdYN_FaqdowrPydRwuTyeeC97SG-SS208d1ZZ5p0K7VEJSs05m-rQE1APX0eAiMtxk0JOBEz_MzwCA--Pf7Hk_mRjEOtYqCmAKM0B8rG3zxohEnWSIZGf8znApfVUhfI85sBMB9YD8A
```

# Payment flows (TO BE MOVED TO "Payment flows and push notifications" earlier in this document)

_**TODO: Move! Replace details with links to Swagger UI**_

## Initiate Payment

Initiate payment is used to create a payment order in Vipps. In order to identify sales channel payments are coming from merchantSerialNumber is used to distinguish between them. Merchant provided orderId must be unique per sales channel. Please note that single payment is uniquely identified by composition of merchantSerialNumber and orderId.

Initiate call will have parameter paymentType which will identify regular ecommerce payment and express checkout payment flow.

Once successfully initiated the transaction in Vipps, it will give you the redirect URL as response which has to be used by merchant to open Vipps landing page. Landing page will own functionality to identify and differentiate request coming from mobile browser/desktop browser.

## Desktop and mobile browser

### Flow when user initiates payment from mobile browser where Vipps is present in same device

Landing page will check if Vipps app is available in same mobile device.
1. If Vipps app is available then it will invoke Vipps app and landing page will be closed.
2. Here after based on user interactions (accept / reject) further payment steps will be followed.
3. Once payment process is completed, Vipps will call fallback URL to redirect to original mobile browser page.
If merchant do not receive callback from Vipps, then they have to confirm their order status from Vipps by calling getOrderStatus service.

### Flow when user initiates payment from mobile browser where Vipps is NOT present in same device

1. Landing page will check if Vipps app is available in same mobile device.
2. If Vipps app is not available then landing page will ask for user’s mobile number. After user enters mobile number, Vipps will send push notification to corresponding Vipps profile, if it exists. Landing page is not closed in this case.
3. Vipps user needs to accept/reject the payment request coming from merchant for further payment steps.
4. Once payment process is completed, landing page will redirect to mobile web browser where payment was initiated.
5. If merchant do not receive callback from Vipps, then they have to confirm their order status from Vipps by calling getOrderStatus service.

### Flow when user initiates payment from desktop browser
1. Landing page will be opened inside desktop browser.
2. As Vipps app is not available in the vicinity, landing page will ask for user’s mobile number. After user enters mobile number, Vipps will send push notification to corresponding Vipps profile, if it exists. Landing page is not closed in this case.
3. Vipps user needs to accept/reject the payment request coming from merchant for further payment steps.
4. Once payment process is completed, landing page will redirect to desktop web browser where payment was initiated.
5. If merchant do not receive callback from Vipps, then they have to confirm their order status from Vipps by calling getOrderStatus service.

## Payment triggered from Merchant Mobile App

Vipps will identify the request coming from mobile app of merchant from initiate request body parameter (isApp). In this case, Vipps backend will send the URI which merchant should use to invoke Vipps app directly. Landing page is not involved in this case.

1. Vipps will identify the request coming from merchant mobile app based on isApp parameter value.
2. If value is true then Vipps will send deeplink URI as response to initiate payment.
3. Merchant as to use this URI to invoke Vipps app for user to proceed with payment.
4. Vipps user needs to accept/reject the payment request coming from merchant for further payment steps.
5. Once payment process is completed, Vipps app will redirect to merchant mobile app where payment was initiated.
6. If merchant do not receive callback from Vipps, then they have to confirm their order status from Vipps by calling getOrderStatus service.

### When user confirms the payment in Vipps app

After the customer has confirmed payment, Vipps will execute funds reservation on customer card used in transaction in order to secure future capture. Please note that in a case of direct capture, reservation and capture is done in a single step.

If the funds reservation fails for any reason (communication error, credit card expired, not enough funds to reserve) Vipps will cancel the payment flow and inform the merchant about outcome. Merchant’s orderId used for cancelled payment flow cannot be reused for a new initiate payment service call.

merchantSerialNumber are provided to merchant by Vipps after merchant account is created or new sales channel is enrolled.


API details: [`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/initiatePaymentV3UsingPOST)

# Cancel Payment

Cancel payment call allows merchant to cancel a reserved payment order

API details: [`PUT:/ecomm/v2/payments/{orderId}/cancel`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/cancelPaymentRequestUsingPUT)

# Capture Payment

Capture payment allows merchant to capture the reserved amount. Amount to capture cannot be higher than reserved. The API also allows capturing partial amount of the reserved amount. Partial capture can be called as many times as required as long as there is reserved amount to capture. Transaction text is not optional and is used as a proof of delivery (tracking code, consignment number etc.).

In a case of direct capture, both fund reservation and capture are executed in a single operation.

API details: [`POST:/ecomm/v2/payments/{orderId}/capture`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/capturePaymentUsingPOST)

# Refund Payment

Refund payment allows merchant to do a refund of an already captured payment order. There is an option to do a partial refund of the captured amount by giving an amount which is lower than the captured amount. Refunded amount cannot be larger than captured.

API details: [`POST:/ecomm/v2/payments/{orderId}/refund`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/refundPaymentUsingPOST)

# Get Payment Details

Get Payment Details allows merchant to get the details of a payment order. Service call returns detailed transaction history of given payment where events are sorted by the time.

API details: [`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getPaymentDetailsUsingGET)

# Get Order Status

Get Order Status allows merchant to get the status of a payment order.

API details: [`GET:/ecomm/v2/payments/{orderId}/status`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getOrderStatusUsingGET)

# Endpoints Hosted by Merchant

1. [Callback: Transaction Update](#callback)
2. [Fetch Shipping Cost & Method](#fetch-shipping-cost)
3. [Remove User Consent](#remove-user-consent)

# Callback

Callback allows Vipps to send the payment order details. During regular ecomm payment order and transaction details will be shared. During express checkout payment it will provide user details and shipping details addition to the order and transaction details.

If the communication is broken during payment process for some reason, and Vipps is not able to execute callback, then callback will not be retried. In other words,if the merchant doesn’t receive any confirmation on payment request call within callback timeframe, merchant should call get payment details service to get the response of payment request.

API details: [`POST:/ecomm/v2/payments/{orderId}`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/transactionUpdateCallbackForRegularPaymentUsingPOST)


# Fetch Shipping Cost

This API call allows Vipps to get the shipping cost and method based on the provided address and product details. Primarily use of this service is meant for ecomm express checkout where Vipps needs to present shipping cost and method to the vipps user. This service is to be implemented by merchants.

API details: [`POST:/[shippingDetailsPrefix]/v2/payments/{orderId}/shippingDetails`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/fetchShippingCostUsingPOST
# Remove User Consent

Allows Vipps to send consent removal request to merchant. After this merchant is obliged to remove the user details from merchant system permanently, as per the GDPR guidelines.

API details: [`DELETE:/[consetRemovalPrefix]/v2/consents/{userId}`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/removeUserConsentUsingDELETE)

# Vipps eCommerce APIs

`[ Base URL: localhost:8080/ ]`

API details: [`Details`](https://vippsas.github.io/vipps-ecom-api/#/)

*(TODO: Elaborate)*

# Vipps Login APIs

The following AOI definitions are presented using Open API definition and Swagger UI

*(TODO: Elaborate)*

# Vipps Signup API

The intention is to create signup forms for Vipps eCom. Prefilled forms are to be created by our ecommerce partners to create a connection between the signup and the partner, and making the process simpler for the merchant by prefilling the form with certain data.

## Process overview

![Signup Api Overview](images/signup-api-overview.png )

## Partner initiates the signup

We want to create a connection between the ecommerce partner ("Partner") and the signup, as the partners are assisting Vipps with the distribution and the merchant has a strong technical relationship to the partner to complete the integration to Vipps ecommerce API.

## v1/partial/signup

```json
{
    "orgnumber" : "819226032",
    "partnerId":"1234",
    "subscriptionPackageId":"1234",
    "merchantWebsiteUrl": "https://www.vipps.no",
    "signupCallbackToken":"",
    "signupCallbackUrl":"https://upload.credentials.to.partner.url",
    "form-type":"vippspanett"
}
```

## Partner receives the signup link

As response to partial signup initiation above the partner receives an signup id and a link to the signup which is forwarded to the merchant to complete and sign.

## Partial Signup Response

```json
{
    "signup-id": "4188dea2-00d0-488a-88b7-b39b186151c0",
    "vippsURL": "https://vippsbedrift.no/signup/vippspanett/?r=4188dea2-00d0-488a-88b7-b39b186151c0"
}
```

## The signup form, KYC and signing process
Merchant completes the form, according to standard form validation for ecommerce (Signup form). Merchant is not displayed with the option to change the partner nor the price package.

## The email notification
In order for Driftkonto to complete the registration according to partner registration routines it is important to provide some additional partner related information in the email notification. Therefore the email notifcation for partner signups extends the regular ecommerce email notification as per specification: "email specification"

# Vipps Signup API

*(TODO: Elaborate)*

# Email Specification
The email notifcation for partner signups extends the regular ecommerce email notification as per the following specification:

# Questions or comments?

Please use GitHub's built-in functionality for
[issues](https://github.com/vippsas/vipps-invoice-api/issues),
[pull requests](https://github.com/vippsas/vipps-invoice-api/pulls),
or contact us at integration@vipps.no.
