# Vipps eCommerce API

API version: 2.1.

Document version 0.3.0.

Please use GitHub's built-in functionality for
[issues](https://github.com/vippsas/vipps-invoice-api/issues),
[pull requests](https://github.com/vippsas/vipps-invoice-api/pulls),
or contact us at integration@vipps.no.

See also the [Vipps eCommerce FAQ](vipps-ecom-api-faq.md).

# Table of contents

- [Vipps eCommerce API](#vipps-ecommerce-api)
- [Table of contents](#table-of-contents)
- [Overview](#overview)
- [Overview of API endpoints](#overview-of-api-endpoints)
- [HTTP responses](#http-responses)
  * [API endpoints required by Vipps from the merchant for express checkout](#api-endpoints-required-by-vipps-from-the-merchant-for-express-checkout)
- [Payment types](#payment-types)
  * [Regular eCommerce payments](#regular-ecommerce-payments)
  * [Express checkout payments](#express-checkout-payments)
- [API development](#api-development)
  * [The Vipps developer portal](#the-vipps-developer-portal)
  * [Postman](#postman)
    + [Setting up Postman](#setting-up-postman)
      - [Step 1: Import the Postman Collection](#step-1--import-the-postman-collection)
      - [Step 2: Import the Postman Environment](#step-2--import-the-postman-environment)
      - [Step 3: Setup Postman Environment](#step-3--setup-postman-environment)
  * [Authentication](#authentication)
    + [API calls flow](#api-calls-flow)
    + [Access token](#access-token)
        * [HTTP response codes](#http-response-codes)
- [Complete HTTP requests and responses for each API endpoint and method](#complete-http-requests-and-responses-for-each-api-endpoint-and-method)
  * [Initiate payment](#initiate-payment)
    + [Initiate payment flows](#initiate-payment-flows)
      - [Mobile browser](#mobile-browser)
        * [Vipps app installed](#vipps-app-installed)
        * [Vipps app not installed](#vipps-app-not-installed)
      - [Desktop browser](#desktop-browser)
      - [Payment initiated by the merchant's app](#payment-initiated-by-the-merchant-s-app)
    + [Flow after the user has confirmed payment in the Vipps app](#flow-after-the-user-has-confirmed-payment-in-the-vipps-app)
    + [Request](#request)
    + [Response](#response)
      - [Get order status](#get-order-status)
      - [Get payment details](#get-payment-details)
  * [Reserve](#reserve)
    + [Request](#request-1)
      - [Get order status](#get-order-status-1)
      - [Get payment details](#get-payment-details-1)
  * [Cancel](#cancel)
    + [Request](#request-2)
    + [Response](#response-1)
      - [Get order status if the merchant has cancelled](#get-order-status-if-the-merchant-has-cancelled)
      - [Get payment details if the merchant has cancelled](#get-payment-details-if-the-merchant-has-cancelled)
      - [Get payment details if the user has cancelled](#get-payment-details-if-the-user-has-cancelled)
  * [Capture](#capture)
    + [Normal capture](#normal-capture)
      - [Full capture](#full-capture)
      - [Partial capture](#partial-capture)
    + [Direct capture](#direct-capture)
    + [Request](#request-3)
    + [Response](#response-2)
      - [Get order status](#get-order-status-2)
      - [Get payment details if the user has cancelled](#get-payment-details-if-the-user-has-cancelled-1)
  * [Refund](#refund)
    + [Request](#request-4)
      - [Get order status](#get-order-status-3)
    + [Response from get payment status](#response-from-get-payment-status)
- [Additional payment flow for express checkout (Vipps Hurtigkasse)](#additional-payment-flow-for-express-checkout--vipps-hurtigkasse-)
  * [Get shipping cost & method](#get-shipping-cost---method)
  * [Transaction updates with user details](#transaction-updates-with-user-details)
  * [Remove user consent](#remove-user-consent)
  * [Exception handling](#exception-handling)
  * [Exception scenarios](#exception-scenarios)
    + [Connection timeout](#connection-timeout)
    + [Callback aborted/interrupted](#callback-aborted-interrupted)
    + [PSP connection issues](#psp-connection-issues)
- [HTTP response codes](#http-response-codes-1)
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
  * [Callback](#callback)
  * [Fetch Shipping Cost](#fetch-shipping-cost)
  * [Remove User Consent](#remove-user-consent)
- [Sequence diagrams for payment flows and push notifications](#sequence-diagrams-for-payment-flows-and-push-notifications)
  * [Push notifications: Desktop browser](#push-notifications--desktop-browser)
  * [Push notifications: Mobile browser](#push-notifications--mobile-browser)
- [Questions or comments?](#questions-or-comments-)

# Overview

The Vipps eCommerce API (eCom API) offers functionality for online payments,
both using web browsers on websites and in native apps for iOS and Android,
using app-switching.


![Vipps checkout flow chart](images/vipps-ecom-flow-chart.svg)

| #   | From state      | To state         | Description | Result from getOrderStatus        |
| --- | ---------- | ---------- | ---------- | -----------------------------------------------------------------------------------|
| 1   | Initiate | - | The user initiate payment.  | `INITIATE`
| -   |  | Reserve  | The merchant reserves the payment.  | `RESERVE`
| -   |            | Cancel   | The user cancels the order.                           |  `CANCEL`
| 2   | Reserve  | Capture  | The merchant captures the payment and ships the goods.          | `CAPTURE`
| -   |            | Cancel   | The merchant cancels the order.                            | `VOID`
| 3   | Capture  | --         | A final state: Payment fully processed.                                      |
| -   |            | Refund   | The merchant refunds the money to the the user.                | `REFUND`
| 4   | Cancel   | --         | A final state: Payment cancelled.                                      |
| 5   | Refund   | --         | A final state: Payment refunded.      |

**Please note:** When using getOrderStatus ([`GET:/ecomm/v2/payments/{orderId}/status`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getOrderStatusUsingGET)), the order will show as _reserved_, even if it has been _captured_.
To se if the payment has been completed, use GetPaymentDetails ([`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getPaymentDetailsUsingGET)).

See [Complete HTTP requests and responses for each API endpoint and method](#complete-http-requests-and-responses-for-each-api-endpoint-and-method) for more details.

# Overview of API endpoints

| Operation           | Description         | Endpoint          |
| ------------------- | ------------------- | ----------------- |
| Initiate payment    | Payment initiation, the first request in the payment flow. This _reserves_ an amount. | [`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/initiatePaymentV3UsingPOST)  |
| Capture payment     | When an amount has been reserved, and the goods are (about to be) shipped, the payment must be _captured_  | [`POST:/ecomm/v2/payments/{orderId}/capture`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/capturePaymentUsingPOST)  |
| Cancel payment      | The merchant may cancel a reserved amount, but not on a captured amount.  | [`PUT:/ecomm/v2/payments/{orderId}/cancel`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/cancelPaymentRequestUsingPUT)  |
| Refund payment      | The merchant may refund a captured amount.  |[`POST:/ecomm/v2/payments/{orderId}/refund`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/refundPaymentUsingPOST)  |
| Get order status    | The status is "reserved" after a payment has been initiated. For details about payment, use [`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getPaymentDetailsUsingGET) | [`GET:/ecomm/v2/payments/{orderId}/status`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getOrderStatusUsingGET)  |
| Get payment details | How much of the reserved amount has been captured, etc.  | [`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getPaymentDetailsUsingGET)  |

See [Complete HTTP requests and responses for each API endpoint and method](#complete-http-requests-and-responses-for-each-api-endpoint-and-method) for more details.

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
| `429 Too Many Requests` | Settle down, please. |
| `500 Server Error`      | An internal Vipps problem.                              |

All error responses contains an `error` object in the body, with details of the problem.
See [Error details](#error-details) for more information.

## API endpoints required by Vipps from the merchant for express checkout

For express checkout ([Vipps Hurtigkasse](https://www.vipps.no/bedrift/vipps-pa-nett/hurtigkasse)),
the below endpoints are required on the _merchant_ side.
These endpoints are included in the Swagger file for reference only, and are
_not_ Vipps endpoints that may be called by merchants.

| Operation           | Description         | Endpoint          |
| ------------------- | ------------------- | ----------------- |
| Remove user consent | Used to inform merchant when the Vipps user removes consent to share information.  | [`DELETE:/v2/consents/{userId}`](https://vippsas.github.io/vipps-ecom-api/#/Calls_from_Vipps_examples/removeUserConsentUsingDELETE)  |
| Callback : Transaction Update | A callback to the merchant for receiving post-payment information. | [`POST:/ecomm/v2/payments/{orderId}`](https://vippsas.github.io/vipps-ecom-api/#/Calls_from_Vipps_examples/transactionUpdateCallbackForRegularPaymentUsingPOST)  |
| Get shipping cost and method | [`POST:/v2/payments/{orderId}/shippingDetails`](https://vippsas.github.io/vipps-ecom-api/#/Calls_from_Vipps_examples/fetchShippingCostUsingPOST)  |   |

See [API endpoints required by Vipps from the merchant](#api-endpoints-required-by-vipps-from-the-merchant) for details.

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

Vipps complies with GDPR, and requires the user's consent before any information
is shared with the merchant. The merchant must provide a URL (`consentRemovalPrefix`)
that Vipps can call to delete the data. The Vipps app gives the Vipps user an
overview of "Companies with access", where the user can manage the consents.

# API development

## The Vipps developer portal

For details about the Vipps Developer Portal and API keys, see the [Getting Started guide](vipps-ecom-api-getting-started.md).

## Postman

[Postman](https://www.getpostman.com/) is a common tool for working with REST APIs.
We offer a [Postman Collection](https://www.getpostman.com/collection) to make development easier.
See the [Postman documentation](https://www.getpostman.com/docs/) for more information about using Postman.

By following the steps below, you can make calls to all the
endpoints, and see the full `request` and `response` for each call.

### Setting up Postman

#### Step 1: Import the Postman Collection

1. Click `Import` in the upper left corner.
2. Import the [vipps-ecom-api-postman-collection.json](https://github.com/vippsas/vipps-ecom-api/blob/master/tools/vipps-ecom-api-postman-collection.json) file

#### Step 2: Import the Postman Environment

1. Click `Import` in the upper left corner.
2. Import the [vipps-ecom-api-postman-enviroment.json](https://github.com/vippsas/vipps-ecom-api/blob/master/tools/vipps-ecom-api-postman-enviroment.json) file

#### Step 3: Setup Postman Environment

1. Click the "eye" icon in the top right corner.
2. In the dropdown window, click `Edit` in the top right corner.
3. Fill in the `Current Value` for the following fields to get started.
   - `access-token-key`
   - `subscription-key`
   - `client-id`
   - `client-secret`

## Authentication

All API calls are authenticated and authorized based on the application access
token (JWT bearer token) and a subscription key (`Ocp-Apim-Subscription-Key`),
and these headers are required:

| Header Name | Header Value | Description |
| ----------- | ------------ | ----------- |
|  `Authorization` | `Bearer <JWT access token>` | Type: Authorization token. This is available in the Vipps Developer Portal. |
| `Ocp-Apim-Subscription-Key` | Base 64 encoded string | The subscription key for the eCom API. This is available in the Vipps Developer Portal. |

### API calls flow

This diagram shows the flow between the merchant and Vipps:

![API calls flow](images/api-calls-flow.png)

### Access token

The Access Token API provides the JWT bearer token.
The `client_id` and `client_secret` are neded to get a JWT access token.
These are found in the Vipps Developer Portal - see the  
[Getting Started guide](vipps-ecom-api-getting-started.md).

| Header Name | Header Value | Description |
| ----------- | ------------ | ----------- |
| `client_id` | A GUID value | Client ID received when merchant registered the application |
| `client_secret` | Base 64 encoded string | Client Secret received when merchant registered the application |
| `Ocp-Apim-Subscription-Key` | Base 64 encoded string | Subscription key for eCommerce product. This can be found in User Profile page on Merchant developer portal after merchant account is created |

The request is: [`POST:/accesstoken/get`](https://vippsas.github.io/vipps-ecom-api/#/Authorization_Service/fetchAuthorizationTokenUsingPost).

```http
POST https://apitest.vipps.no/accessToken/get
client_id: <ClientID>
client_secret: <ClientSecret>
Ocp-Apim-Subscription-Key: <Ocp-Apim-Subscription-Key>
```

**Please note:** It _is_ correct that this is a `POST` request with an empty body.
This is due to some technical details of the backend solutions.

##### HTTP response codes

This API returns the following HTTP statuses in the responses:

| HTTP status         | Description                                 |
| ------------------- | ------------------------------------------- |
| `200 OK`            | Request successful.                          |
| `400 Bad Request`   | Invalid request, see the `error` for details.  |
| `401 Unauthorized`  | Invalid credentials.                         |
| `403 Forbidden`     | Authentication ok, but credentials lacks authorization.  |
| `500 Server Error`  | An internal Vipps problem.                  |

Example of an error reponse body (formatted for readability):

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

The response contains a JWT:

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

JWT properties:

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


# Complete HTTP requests and responses for each API endpoint and method

This section contains complete HTTP `requests` and `responses` for each API call.
All calls may also be done using Postman, where you can also see the complete `request` and `response` for each call.

For each step we have included a complete `request` and `response` for a call to:
* Get order status:
[`GET:/ecomm/v2/payments/{orderId}/status`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getOrderStatusUsingGET)
* Get payment details:
[`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getPaymentDetailsUsingGET)

Please note that the `response` from [`GET:/ecomm/v2/payments/{orderId}/status`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getOrderStatusUsingGET)
always contain _the entire history_ of the order, not just the current status.

## Initiate payment

Initiate payment is the first call in the payment flow, and is used to create a
a new payment order in Vipps:

[`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/initiatePaymentV3UsingPOST)

This creates a new payment with order status to `INITIATE`.
The Vipps user is automatically prompted by the Vipps app for confirmation.

A payment is uniquely identified by the combination of `merchantSerialNumber` and `orderId`:
* `merchantSerialNumber`: The merchant's Vipps id.
* `orderId`: Must be unique for this `merchantSerialNumber`.

The payment initiation call must include the `paymentType` parameter, which
specifies the type of payment:
* A regular eCommerce payment: `eComm Inapp Allignment Payment` (yes, to `l`s, sorry)
* An express payment (Vipps Hurtigkasse): `eComm Express Payment`

Once successfully initiated, a response with a redirect URL is returned.
The redirect depends on whether the user is using a desktop or mobile browser:
* For mobile browsers, the URL is for an app-switch to the Vipps app.
* For desktop browsers, the URL is for the Vipps "landing page".

### Initiate payment flows

#### Mobile browser

The landing page will detect if the Vipps app is installed.

##### Vipps app installed

1. The Vipps app is invoked and the landing page is closed.
2. The user accepts (or rejects) the payment in the Vipps app.
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
2. The landing page will prompt for user’s mobile number.
3. Vipps sends a push notification to corresponding Vipps profile, if it exists. The landing page is not closed in this case.
4. The user accepts (or rejects) the payment in the Vipps app.
5. Once payment process is completed, the landing page will redirect to the desktop web browser where the payment was initiated.
6. If the merchant does not receive callback from Vipps, it must confirm the order status from Vipps by calling [`GET:/ecomm/v2/payments/{orderId}/status`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getOrderStatusUsingGET).

#### Payment initiated by the merchant's app

Vipps will identify the request coming from merchant's app by the `isApp` parameter.
In this case, the Vipps backend will send the URI use by the merchant to invoke the Vipps app.
The landing page is not involved in this case.

1. Vipps will identify the request coming from merchant's app by the `isApp` parameter.
2. Vipps sends a `deeplink` URI as response to initiate payment.
3. Th merchant uses the URI to invoke the Vipps app.
4. The user accepts (or rejects) the payment in the Vipps app.
5. The Vipps app redirects to the merchant's 'app where payment was initiated.
6. If the merchant does not receive callback from Vipps, it must confirm the order status from Vipps by calling [`GET:/ecomm/v2/payments/{orderId}/status`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getOrderStatusUsingGET).

### Flow after the user has confirmed payment in the Vipps app

After the user has confirmed payment, Vipps will reserve an amount on user's
payment, to secure a future capture.
For direct capture, the reservation and capture is done in a single step.

If the funds reservation fails for any reason, Vipps will cancel the payment flow
and inform the merchant. The merchant’s `orderId` used for the cancelled payment
flow cannot be reused in a new order.

If user does _not_ confirm in the Vipps app within five minutes, the
payment request times out, and the payment flow stops.

### Request

[`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/initiatePaymentV3UsingPOST)

```json
{
    "merchantInfo": {
      "merchantSerialNumber": "123456",
      "callbackPrefix": "https://merchant-website.example.com/vipps/callback/",
      "fallBack":"https://merchant-website.example.com/vipps/fallback/",
      "paymentType":"eComm Inapp Allignment Payment",
      "isApp": false
    },
    "userInfo": {
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

The URLs specified for `callbackPrefix` and `fallBack` are validated with the
[Apache Commons UrlValidator](https://commons.apache.org/proper/commons-validator/apidocs/org/apache/commons/validator/routines/UrlValidator.html).

If `isApp` is true, the `fallBack` is not validated with Apache Commons UrlValidator,
as the app-switch URL may be something like `vipps://`, which is not a valid ÚRL.

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

### Response

```json
HTTP 202 Accepted
{
    "orderId": "order123abc",
    "url": "https://api.vipps.no/deeplink/vippsgateway?token=eyJraWQiOiJqd3RrZXkiLCJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJmMDE0MmIxYy02YjIwLTQ1M2MtYTlmMS1lMWUwZGFiNjkzOTciLCJhdWQiOiJmMDE0MmIxYy02YjIwLTQ1M2MtYTlmMS1lMWUwZGFiNjkzOTciLCJhenAiOiJmMDE0MmIxYy02YjIwLTQ1M2MtYTlmMS1lMWUwZGFiNjkzOTciLCJhcHBUeXBlIjoiTEFORElOR1BBR0UiLCJtZXJjaGFudFNlcmlhbE51bWJlciI6IjEwMTIxMiIsImV4cHJlc3NDaGVja091dCI6Ik4iLCJpc3MiOiJodHRwczpcL1wvdmlwcHMtdGVzdC1jb24tYWdlbnQtaWxiLnRlc3QudGVjaC0wMi5uZXRcL2F0M1wvZGVlcGxpbmstb3BlbmlkLXByb3ZpZGVyLWFwcGxpY2F0aW9uXC8iLCJleHAiOjE1NDIyMDg4OTcsInRva2VuVHlwZSI6IkRFRVBMSU5LIiwiaWF0IjoxNTQyMjA4Nzc3LCJ1dWlkIjoiM2Q3NTRlMzItYjQ4NC00Y2Y2LTk1MjctYTQ3OWRhODA2ZTQ4IiwianRpIjoiZWY4MTVjMTYtNzM4ZS00NzRhLTljMGMtYTQxN2MyMjljNjJlIn0.UEZNp-hWZqrQvo6ZlQQ9KBuOt-fGIvPsAxOV1NQSGl_y-Wb1l_YCPonHw__Zxc3xdczPo9oopCsN9wHkbBs5xy1Z_S8DmCp0ziExl-cNsPU7v-D6BgmGfDbYvWp6SHZlPh0YLah3OeZVcMvLPPB3g5O7DtqkC-tT0M2H-dGzNPUYLREAjh8WsyWUI6sOmx7aiZ73M42k9sPIOV7FcUELJTPGF38UcK0LM9biCsChhIL5nVyBUO0JeIf5EmlBOs7FP7poYFlMAvMjMnTv0GfLsAvxFfr94ZU_ziZRRV0all5cZf49Azt8bc7zA-LY6I32zJvIqGvNNtvFAG5QGSHgCw"
}
```

The `url` is slightly simplified, but the format is correct:
`https://hostname.example.com/deeplink/vippsgateway?token=eyJraWQiOiJqd3R <snip>`.


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

When the Vipps backend reserves a payment initiation, the user is
prompted for confirmation in the Vipps app.

When the user confirms, the payment status changes to `RESERVE`.
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

Reservations can be cancelled, and the payment flow aborted, under certain circumstances:

* When the user cancels (rejects) the initiated payment in the Vipps app.
* Timeouts: If the user does not confirm, etc.

Partially captured reservations can not be cancelled.

After cancellation, the order gets a new status:
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

#### Get payment details if the user has cancelled

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

Capture payment allows the merchant to capture the reserved amount.
The API allows for both a _full amount capture_ and a _partial amount capture_.

The amount to capture cannot be higher than the reserved amount.
Because of this, the reserved amount is sometimes set to make sure the shipping cost can be
added to the amount before doing the capture.

### Normal capture

#### Full capture

Full capture, of the entire amount that is reserved, is the most common method.

According to Norwegian law, capture can not be done before the goods have been shipped.
See the Consumer Authority's
[Guidelines for the standard sales conditions for consumer purchases of goods over the internet](https://www.forbrukertilsynet.no/english/guidelines/guidelines-the-standard-sales-conditions-consumer-purchases-of-goods-the-internet).

For a full capture, the `amount` is set to `0` (zero).
There is only a need to specify the `amount` when doing a partial capture.

#### Partial capture

Partial capture may be used if shipping cost can not be determined, or for other reasons.
Partial capture may be called as many times as required as long as there is
a remaining reserved amount to capture.
The transaction text is mandatory, and is used as a proof of delivery (tracking code, consignment number etc.).

### Direct capture

For direct capture, both fund reservation and capture are executed in a single operation.

Please note that there are additional regulatory
requirements and compliance checks needed for merchants using direct capture.

### Request

[`POST:/ecomm/v2/payments/order123abc/capture`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/capturePaymentUsingPOST)

```json
{
    "merchantInfo": {
        "merchantSerialNumber": "123456"
    },
    "transaction": {
        "amount": "0",
        "transactionText":"Socks on the way! Tracking code: abc-tracking-123"
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
        "transactionText": "One pair of Vipps socks",
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
#### Get payment details if the user has cancelled

[`GET:/ecomm/v2/payments/order123abc/details`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getPaymentDetailsUsingGET)


```json
{
  "TODO": "fix"
}
```

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

# Additional payment flow for express checkout (Vipps Hurtigkasse)

Express checkout (Vipps Hurtigkasse) requires additional API endpoints on the merchant side,
and these will be called by Vipps as part of the payment flow.

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

## Exception handling

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
| user  | Error raised because of Vipps user (Example: The user is not a Vipps user)  |
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

## Callback

[`POST:/ecomm/v2/payments/{orderId}`](https://vippsas.github.io/vipps-ecom-api/#/Endpoints_required_by_Vipps_from_the_merchant/transactionUpdateCallbackForRegularPaymentUsingPOST)

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

[`POST:/[shippingDetailsPrefix]/v2/payments/{orderId}/shippingDetails`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/fetchShippingCostUsingPOST)

This API call allows Vipps to get the shipping cost and method based on the
provided address and product details. The primary use of this service is meant
for ecomm express checkout where Vipps needs to present the shipping cost and
method to the Vipps user.

## Remove User Consent

[`DELETE:/[consetRemovalPrefix]/v2/consents/{userId}`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/removeUserConsentUsingDELETE)

Vipps complies with GDPR, and this endpoint is required for Vipps to be able
to remove user consent at the merchant side.
The merchant is obliged to remove the user details from merchant system permanently.

`consetRemovalPrefix` may be `https://merchant-website.example.com/vipps/consentremoval/`.

# Sequence diagrams for payment flows and push notifications

There are separate payment flows and push notification flows for desktop and mobile browsers.

## Push notifications: Desktop browser

![API calls flow: Push for desktop](images/api-calls-flow-push-desktop.png)

## Push notifications: Mobile browser

![API calls flow: Push for mobile](images/api-calls-flow-push-mobile.png)

# Questions or comments?

Please use GitHub's built-in functionality for
[issues](https://github.com/vippsas/vipps-invoice-api/issues),
[pull requests](https://github.com/vippsas/vipps-invoice-api/pulls),
or contact us at integration@vipps.no.
