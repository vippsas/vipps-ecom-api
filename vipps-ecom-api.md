# Vipps eCommerce API

API version: 2.1.

Document version 0.2.1.

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

![Vipps checkout flow chart](images/vipps-ecom-flow-chart.svg)

|     #   | Action       | getOrderStatus        |  Description                                      |
| ------- | ---------- | ---------- | -----------------------------------------------------------------------------------|
|     1   | `initiate` | `reserve`  | The customer confirms payment in the Vipps app. The merchant reserves the payment.  |
|     -   |            | `cancel`   | The customer cancels the order .                           |
|     2   | `reserve`  | `capture`  | The merchant captures the payment and ships the goods.          |
|     -   |            | `cancel`   | The merchant cancels the order.                            |
|     3   | `capture`  | --         | A final state: Payment fully processed.                                      |
|     -   |            | `refund`   | The merchant refunds the money to the the customer.                |
|     4   | `cancel`   | --         | A final state: Payment cancelled.                                      |
|     5   | `refund`   | --         | A final state: Payment refunded.                                      |

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

| #   | From state      | To state         | Description | Result from getOrderStatus        |
| --- | ---------- | ---------- | ---------- | -----------------------------------------------------------------------------------|
| 1   | Initiate | - | The customer initiate payment.  | `INITIATE`
| -   |  | Reserve  | The merchant reserves the payment.  | `RESERVE`
| -   |            | Cancel   | The customer cancels the order.                           |  `CANCEL`
| 2   | Reserve  | Capture  | The merchant captures the payment and ships the goods.          | `CAPTURE`
| -   |            | Cancel   | The merchant cancels the order.                            | `VOID`
| 3   | Capture  | --         | A final state: Payment fully processed.                                      |
| -   |            | Refund   | The merchant refunds the money to the the customer.                | `REFUND`
| 4   | Cancel   | --         | A final state: Payment cancelled.                                      |
| 5   | Refund   | --         | A final state: Payment refunded.      |

# API call details

This section contains complete HTTP `requests` and `responses` for each API call.
For each step a call to "get order status"
([`GET:/ecomm/v2/payments/{orderId}/status`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getOrderStatusUsingGET))
and "get payment details"
([`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getPaymentDetailsUsingGET))
are shown.

Please note that the `response` from [`GET:/ecomm/v2/payments/{orderId}/status`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getOrderStatusUsingGET)
always contain the entire history of the order, not just the current status.

## Initiate

Initiate payment is the first call in the payment flow, and is used to create a
a new payment order in Vipps.

This creates a new payment with order status to `INITIATE`.
The Vipps customer is automatically prompted by the Vipps app for confirmation.

In order to identify which sales channel payments are coming from,
`merchantSerialNumber` is used.
A unique `orderId` must be specified (unique per  `merchantSerialNumber`).
A payment is uniquely identified by the combination of `merchantSerialNumber` and `orderId`.

The payment initiation call must include the `paymentType` parameter, which
specifies whether the payment is a regular eCommerce payment: `eComm Inapp Allignment Payment`,
or an express payment (Vipps Hurtigkasse): `eComm Express Payment`.

Once successfully initiated, a response with a redirect URL is returned.
The redirect depends on whether the user is using a desktop or mobile browser:
* For mobile browsers, the URL is for an app-switch to the Vipps app.
* For desktop browsers, the URL is for the Vipps "landfing page".

### Initiate payment flows

#### Mobile browser

The landing page will check if Vipps app is installed.

##### Vipps app installed

1. The Vipps app is invoked and the landing page is closed.
2. The user accepts (or rejects) in the Vipps app.
3. Once payment process is completed, Vipps will call fallback URL to redirect to original mobile browser page.
4. If merchant does not receive a callback from Vipps, it must confirm the order status from Vipps by calling [`GET:/ecomm/v2/payments/{orderId}/status`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getOrderStatusUsingGET).

##### Vipps app not installed

1. The user is prompted for the mobile number.
2. Vipps sends a push notification to corresponding Vipps profile, if it exists. The landing page is not closed in this case.
3. The Vipps user accepts or rejects the payment request.
4. Once payment process is completed, the landing page will redirect to the mobile web browser where payment was initiated.
5. If the merchant does not receive callback from Vipps, it must confirm the order status from Vipps by calling [`GET:/ecomm/v2/payments/{orderId}/status`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getOrderStatusUsingGET).

#### Desktop browser

1. The landing page will be opened in the desktop browser.
2. The landing page will ask for user’s mobile number.
3. Vipps sends a push notification to corresponding Vipps profile, if it exists. The landing page is not closed in this case.
3. The user accepts (or rejects) in the Vipps app.
4. Once payment process is completed, the landing page will redirect to the desktop web browser where the payment was initiated.
5. If the merchant does not receive callback from Vipps, it must confirm the order status from Vipps by calling [`GET:/ecomm/v2/payments/{orderId}/status`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getOrderStatusUsingGET).

#### Payment initiated by the merchant's app

Vipps will identify the request coming from merchant's app by the `isApp` parameter.
In this case, the Vipps backend will send the URI use by the merchant to invoke the Vipps app.
The landing page is not involved in this case.

1. Vipps will identify the request coming from merchant's app by the `isApp` parameter.
2. Vipps sends a `deeplink` URI as response to initiate payment.
3. Th merchant uses the URI to invoke the Vipps app.
4. The Vipps user accepts or rejects the payment request.
5. The Vipps app redirects to the merchant's 'app where payment was initiated.
6. If the merchant does not receive callback from Vipps, it must confirm the order status from Vipps by calling [`GET:/ecomm/v2/payments/{orderId}/status`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getOrderStatusUsingGET).

### Flow after the user has confirmed payment in the Vipps app

After the customer has confirmed payment, Vipps will reserve an amount on customer's
payment, to secure a future capture.
For direct capture, the reservation and capture is done in a single step.

If the funds reservation fails for any reason, Vipps will cancel the payment flow
and inform the merchant. The merchant’s `orderId` used for the cancelled payment
flow cannot be reused in a new order.

If customer does _not_ confirm in the Vipps app within five minutes, the
payment request times out, and the payment flow stops.

### Request

```json
{
    "merchantInfo": {
      "merchantSerialNumber": "123456",
      "callbackPrefix": "https://merchant-website.example.com/vipps/callback/",
      "fallBack":"https://merchant-website.example.com/vipps/fallback/",
      "paymentType":"eComm Inapp Allignment Payment",
      "isApp": false
    },
    "customerInfo": {
        "mobileNumber": "99999999"
    },
    "transaction": {
        "orderId": "order123abc",
        "amount": "20000",
        "transactionText": "One pair of Vipps socks",
        "timeStamp": "2018-11-14T15:17:30.684Z"
    }
}

```

### Response

```json
HTTP 202 Accepted
{
    "orderId": "order123abc",
    "url": "https://ece46ec4-6f9c-489b-8fe5-146a89e11635.tech-02.net/dwo-api-application/v1/deeplink/vippsgateway?token=eyJraWQiOiJqd3RrZXkiLCJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJmMDE0MmIxYy02YjIwLTQ1M2MtYTlmMS1lMWUwZGFiNjkzOTciLCJhdWQiOiJmMDE0MmIxYy02YjIwLTQ1M2MtYTlmMS1lMWUwZGFiNjkzOTciLCJhenAiOiJmMDE0MmIxYy02YjIwLTQ1M2MtYTlmMS1lMWUwZGFiNjkzOTciLCJhcHBUeXBlIjoiTEFORElOR1BBR0UiLCJtZXJjaGFudFNlcmlhbE51bWJlciI6IjEwMTIxMiIsImV4cHJlc3NDaGVja091dCI6Ik4iLCJpc3MiOiJodHRwczpcL1wvdmlwcHMtdGVzdC1jb24tYWdlbnQtaWxiLnRlc3QudGVjaC0wMi5uZXRcL2F0M1wvZGVlcGxpbmstb3BlbmlkLXByb3ZpZGVyLWFwcGxpY2F0aW9uXC8iLCJleHAiOjE1NDIyMDg4OTcsInRva2VuVHlwZSI6IkRFRVBMSU5LIiwiaWF0IjoxNTQyMjA4Nzc3LCJ1dWlkIjoiM2Q3NTRlMzItYjQ4NC00Y2Y2LTk1MjctYTQ3OWRhODA2ZTQ4IiwianRpIjoiZWY4MTVjMTYtNzM4ZS00NzRhLTljMGMtYTQxN2MyMjljNjJlIn0.UEZNp-hWZqrQvo6ZlQQ9KBuOt-fGIvPsAxOV1NQSGl_y-Wb1l_YCPonHw__Zxc3xdczPo9oopCsN9wHkbBs5xy1Z_S8DmCp0ziExl-cNsPU7v-D6BgmGfDbYvWp6SHZlPh0YLah3OeZVcMvLPPB3g5O7DtqkC-tT0M2H-dGzNPUYLREAjh8WsyWUI6sOmx7aiZ73M42k9sPIOV7FcUELJTPGF38UcK0LM9biCsChhIL5nVyBUO0JeIf5EmlBOs7FP7poYFlMAvMjMnTv0GfLsAvxFfr94ZU_ziZRRV0all5cZf49Azt8bc7zA-LY6I32zJvIqGvNNtvFAG5QGSHgCw"
}
```

#### Get order status

[`GET:/ecomm/v2/payments/order123abc/status`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getOrderStatusUsingGET)  |

```json
{
    "orderId": "order123abc",
    "transactionLogHistory": [
        {
            "amount": 114,
            "transactionText": "One pair of Vipps socks",
            "timeStamp": "2018-11-14T15:17:30.684Z",
            "operation": "INITIATE",
            "operationSuccess": true
        }
    ]
}
```

#### Get payment details

[`GET:/ecomm/v2/payments/order123abc/details`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getPaymentDetailsUsingGET)

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

## Reserve

When the Vipps backend reserves a payment initiation, the customer is
prompted for confirmation in the Vipps app.

When the customer confirms, the payment status changes to `RESERVE`.
The respective amount will be reserved for future capturing.

### Request

The request is automatic from the Vipps backend to the Vipps app.

#### Get order status

[`GET:/ecomm/v2/payments/order123abc/status`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getOrderStatusUsingGET)  |


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
            "transactionId": "5001420055",
            "timeStamp": "2018-11-14T15:21:22.126Z",
            "operation": "RESERVE",
            "requestId": "",
            "operationSuccess": true
        },
        {
            "amount": 20000,
            "transactionText": "One pair of Vipps socks",
            "transactionId": "5001420055",
            "timeStamp": "2018-11-14T15:21:04.697Z",
            "operation": "INITIATE",
            "requestId": "",
            "operationSuccess": true
        }
    ]
}
```
#### Get payment details

[`GET:/ecomm/v2/payments/order123abc/details`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getPaymentDetailsUsingGET)

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

## Cancel

Reservations can be cancelled and payment flow aborted under certain circumstances:

* When the user cancels the initiated payment, the payment will be cancelled.
* Timeouts (if the customer does not confirm, etc) results in a cancellation of the initiated payment.
* Partially captured reservations can not be cancelled.

Resulting status of the order:
* If an order is cancelled by the merchant, it gets the status `VOID`.
* If an order is cancelled by the user, it gets the status `CANCEL`.

### Request

[`PUT:/ecomm/v2/payments/order123abc/cancel`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/cancelPaymentRequestUsingPUT)

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

### Response

```json
{
    "orderId": "order123abc",
    "transactionInfo": {
        "amount": 20000,
        "transactionText": "Cancelling the order",
        "status": "Cancelled",
        "transactionId": "5001420061",
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

#### Get order status if the merchant has cancelled

[`GET:/ecomm/v2/payments/order123abc/status`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getOrderStatusUsingGET)  |


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

#### Get payment details if the merchant has cancelled

[`GET:/ecomm/v2/payments/order123abc/details`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getPaymentDetailsUsingGET)


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

#### Get payment details if the customer has cancelled

[`GET:/ecomm/v2/payments/order123abc/details`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getPaymentDetailsUsingGET)


```json
{
    "orderId": "1542212324",
    "transactionInfo": {
        "amount": 2224,
        "status": "CANCEL",
        "transactionId": "5001420067",
        "timeStamp": "2018-11-14T16:18:57.393Z"
    }
}
```

## Capture

Capture payment allows merchant to capture the reserved amount.
The API allows for both a _full amount capture_ and a _partial amount capture_.
The amount to capture cannot be higher than the reserved amount.

### Normal capture

#### Full capture

Full capture is the most common method.

When the merchant has shipped the goods, the payment can be captured.

For a full capture, the `amount` is zero. There is only a need to specify the
`amount` when doing a partial capture.

#### Partial capture

Partial capture may be used if shipping cost can not be determined, or for other reasons.
Partial capture can be called as many times as required as long as there is
a remaining reserved amount to capture.
The transaction text is mandatory, and is used as a proof of delivery (tracking code, consignment number etc.).

### Direct capture

For direct capture, both fund reservation and capture are executed in a single operation.

Please note that there are additional regulatory
requirements and compliance checks needed for merchants using direct capture.
See the Consumer Authority's
[Guidelines for the standard sales conditions for consumer purchases of goods over the internet](https://www.forbrukertilsynet.no/english/guidelines/guidelines-the-standard-sales-conditions-consumer-purchases-of-goods-the-internet).

### Request

[`POST:/ecomm/v2/payments/order123abc/capture`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/capturePaymentUsingPOST)

```json
{
    "merchantInfo": {
        "merchantSerialNumber": "123456"
    },
    "transaction": {
        "amount": "0",
        "transactionText":"One pair of Vipps socks"
    }
 }
```

### Response

```json
{
    "orderId": "order123abc",
    "transactionInfo": {
        "amount": 20000,
        "timeStamp": "2018-11-14T15:22:46.736Z",
        "transactionText": "transaction text ",
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

#### Get order status

[`GET:/ecomm/v2/payments/order123abc/status`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getOrderStatusUsingGET)  |

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
            "amount": 2466,
            "transactionText": "Refund ",
            "transactionId": "5001420059",
            "timeStamp": "2018-11-14T15:23:02.286Z",
            "operation": "REFUND",
            "requestId": "1542208972",
            "operationSuccess": true
        },
        {
            "amount": 20000,
            "transactionText": "transaction text ",
            "transactionId": "5001420058",
            "timeStamp": "2018-11-14T15:22:46.680Z",
            "operation": "CAPTURE",
            "requestId": "1542208966",
            "operationSuccess": true
        },
        {
            "amount": 20000,
            "transactionText": "One pair of Vipps socks",
            "transactionId": "5001420055",
            "timeStamp": "2018-11-14T15:21:22.126Z",
            "operation": "RESERVE",
            "requestId": "",
            "operationSuccess": true
        },
        {
            "amount": 20000,
            "transactionText": "One pair of Vipps socks",
            "transactionId": "5001420055",
            "timeStamp": "2018-11-14T15:21:04.697Z",
            "operation": "INITIATE",
            "requestId": "",
            "operationSuccess": true
        }
    ]
}
```
#### Get payment details if the customer has cancelled

[`GET:/ecomm/v2/payments/order123abc/details`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getPaymentDetailsUsingGET)


```json
TODO
```

## Direct capture

Direct capture is not depicted on diagram above but, in essence, combines two
steps (reserve and capture) in one. This is a configuration in Vipps backend
when Initiate payment request is sent.

## Refund

The merchant can initiate a refund of the captured amount.
The refund can be a partial or full.

Partial refunds are done byt specifying an amount which is lower than the captured amount.
The refunded amount cannot be larger than the captured amount.

### Request

[`POST:/ecomm/v2/payments/order123abc/refund`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/refundPaymentUsingPOST)

```json
{
    "merchantInfo": {
        "merchantSerialNumber": "123456"
    },
    "transaction": {
        "amount": "0",
        "transactionText":"Refund of Vipps socks"
    }
 }
```

#### Get order status

[`GET:/ecomm/v2/payments/order123abc/status`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getOrderStatusUsingGET)  |

```json
{
    "orderId": "order123abc",
    "transactionSummary": {
        "capturedAmount": 20000,
        "remainingAmountToCapture": 0,
        "refundedAmount": 2466,
        "remainingAmountToRefund": 19655
    },
    "transactionLogHistory": [
        {
            "amount": 20000,
            "transactionText": "Refund of Vipps socks",
            "transactionId": "5001420059",
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
            "transactionId": "5001420055",
            "timeStamp": "2018-11-14T15:21:22.126Z",
            "operation": "RESERVE",
            "requestId": "",
            "operationSuccess": true
        },
        {
            "amount": 20000,
            "transactionText": "One pair of Vipps socks",
            "transactionId": "5001420055",
            "timeStamp": "2018-11-14T15:21:04.697Z",
            "operation": "INITIATE",
            "requestId": "",
            "operationSuccess": true
        }
    ]
}
```

### Response from get payment status

```json
{
  "TODO": "fix"
}
```

# Additional payment flow for express checkout

In addition to above mentioned payment flows, following are the services which
merchants should build on their side to support express checkout during online purchases.

## Get shipping cost & method

When express checkout payment is initiated, Vipps will call this service from
the merchant’s backend to fetch shipping cost and shipping method related details.
The merchant can send priority of shipping cost and method combination if there
are multiple ways of delivery. Merchant can also send default shipping cost &
method combination which merchant wants user to see on payment confirmation
screen of Vipps. Vipps will support up to 10 shipping cost and method combinations.
If user sends more than 10 combinations, Vipps will display first 10 always.

## Transaction updates with user details

After express checkout payment is processed, vipps will make a call back to
merchant stating payment details, shipping details and user details (optional).

## Remove user consent

When consent to store/process/view details of vipps user is removed by user in
vipps app, vipps will make a call to merchant informing the same. Merchant is
obliged to delete user information upon receiving this request.

## Exception handling 7.1 Introduction

Every system, especially those that includes complex integrations and/or
participation of many users, is prone to unexpected conditions. Below section
explains how Vipps handles different exception and error situations in detail.

## Exception scenarios

The most critical action in payment flow is when Initiate Payment service call
is invoked. The Flow diagram below shows how to successfully fulfil service call,
communication between several contributors and users across several systems has
to work flawlessly.

![Payment flow eCommerce – Payment request](images/payment-flow-for-ecommerce-payment-request.png)

To cope with possible communication problems/errors, several scenarios and
guidelines are developed.

### Connection timeout

Defining a socket timeout period is the common measure to protect server
resources and is expected. However, the time needed to fulfill a service requests
depends on several systems, which impose longer timeout period than usually
required. We recommend setting no less than 1 second socket connection timeout
and 5 seconds socket read timeout while communicating with Vipps.
A good practice is, if/when the socket read timeout occurs call Get Payment
Details and check status of last transaction in transaction history prior
executing the service call again.

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

| Status Code	| Description |
| ----------- | ----------- |
|100 |	Success |
|302 |	User doesn’t have Vipps profile |
|303 |	Login failed (login max attempt reached) |
|304 |	Vipps doesn’t support this action, please update Vipps |
|401 |	Request timed out or Token has expired |
|451 |	The user was selected for fraud validation |
|999 |	Failed |

Below are the status code ranges which Vipps maintains for future purposes.
For example, if there is new error message related to fraud, then it will fall under range 450 to 499.

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

# API definitions and details

| Header Name | Header Value | Optional | Description |
| ----------- | ------------ | -------- | ----------- |
| Authorization |	JWT Access Token <>	| No | type: Authorization token value: Access token is obtained by registering merchant backend application in Merchant Developer Portal. |
| Content-Type | application/json |	No | Type of the body |
| Ocp-Apim-Subscription-Key | Base 64 encoded string | No |	Subscription key for eCommerce product. This can be found in User Profile page on Merchant developer portal |
| X-TimeStamp	| Time stamp when the request called | Yes |	Time to call |
| X-Request-Id | To identify the idempotent request | Yes | For Making request to be idempotent this ID is must so that the system will not do any side effects. 1. Applicable to Initiate, Capture, Refund payment 2. Size should be 30 3. If user wants to re-try any failed capture or refund transaction then they should provide same X-request-id, else system will create a new entry for partial capture or partial refund. |

## deeplinkURL

Below is a complete example of a deeplinkURL:

```
vipps://?token=eyJraWQiOiJqd3RrZXkiLCJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJmMDE0MmIxYy02YjIwLTQ1M2MtYTlmMS1lMWUwZGFiNjkzOTciLCJhdWQiOiJmMDE0MmIxYy02YjIwLTQ1M2MtYTlmMS1lMWUwZGFiNjkzOTciLCJhenAiOiJmMDE0MmIxYy02YjIwLTQ1M2MtYTlmMS1lMWUwZGFiNjkzOTciLCJhcHBUeXBlIjoiTEFORElOR1BBR0UiLCJpc3MiOiJodHRwczpcL1wvdmlwcHMtdGVzdC1jb24tYWdlbnQtaWxiLnRlc3QudGVjaC0wMi5uZXRcL2F0M1wvZGVlcGxpbmstb3BlbmlkLXByb3ZpZGVyLWFwcGxpY2F0aW9uXC8iLCJhZ3JlZW1lbnRJZCI6ImFncl9UNmNMeGY0IiwiZXhwIjoxNTQxNDMwMzk5LCJ0b2tlblR5cGUiOiJERUVQTElOSyIsImlhdCI6MTU0MTQzMDI3OSwidXVpZCI6IjU2MTM0YjI3LWQ1ODAtNDdmZC1hOTExLWYzMGU0ZDY4NmNkZiIsImp0aSI6IjI2NzIyYzU5LWZlYTMtNGYwNi1hNGRiLTVhOGM3MDFjY2JkMCJ9.d_jDLUW7cZi6xF5N51CggymsiT0zQM36a2nEB8h7jQhCIE5BDG_6u0QmW4tbYiK8T_8TJNmPIQbJ1YY7gZiIY9kHD1fPQC6JlFS4FaUldOdq9yYesqXFcvnsZdzeRk45g9YuCdHSmmcYQ4Hzz3HurlZ_txiPybytOWbhm1hTKFsyQOC_1GZWA9SFdLG8g2z5ZMuCkyUlXgArdYN_FaqdowrPydRwuTyeeC97SG-SS208d1ZZ5p0K7VEJSs05m-rQE1APX0eAiMtxk0JOBEz_MzwCA--Pf7Hk_mRjEOtYqCmAKM0B8rG3zxohEnWSIZGf8znApfVUhfI85sBMB9YD8A
```

# API endpoints Vipps requires from the merchant

The following endponts are to be implemented by merchants, in order for Vipps to make calls to them.
The documentation is included in the Swagger file for reference only - these endpoints are _not_ callable at Vipps.

## Callback

This allows Vipps to send the payment order details:
* For regular payments, the order details will be shared.
* For express checkout (Vipps Hurtigkasse), the order details plus shipping details will be shared.

If the communication is broken during the payment process (i.e: a timeout),
and Vipps is not able to execute the callback, then callback will not be retried.
If the merchant does not receive a callback, the merchant can use
[`GET:/ecomm/v2/payments/{orderId}/status`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getOrderStatusUsingGET)
and
[`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getPaymentDetailsUsingGET)
to get the order details.

## Fetch Shipping Cost

This API call allows Vipps to get the shipping cost and method based on the
provided address and product details. The primary use of this service is meant
for ecomm express checkout where Vipps needs to present the shipping cost and
method to the Vipps user.

 [`POST:/[shippingDetailsPrefix]/v2/payments/{orderId}/shippingDetails`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/fetchShippingCostUsingPOST

## Remove User Consent

Vipps complies with GDPR, and this endpoint is required for Vipps to be able
to remove user consent at the merchant side.
The merchant is obliged to remove the user details from merchant system permanently.

`consetRemovalPrefix` may be `https://merchant-website.example.com/vipps/consentremoval/`.

 [`DELETE:/[consetRemovalPrefix]/v2/consents/{userId}`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/removeUserConsentUsingDELETE)


# Questions or comments?

Please use GitHub's built-in functionality for
[issues](https://github.com/vippsas/vipps-invoice-api/issues),
[pull requests](https://github.com/vippsas/vipps-invoice-api/pulls),
or contact us at integration@vipps.no.
