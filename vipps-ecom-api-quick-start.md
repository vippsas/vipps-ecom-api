<!-- START_METADATA
---
title: Quick start
sidebar_position: 30
---
END_METADATA -->

# Quick start

<!-- START_TOC -->

## Table of Contents

* [Postman](#postman)
  * [Step 1: Get the Postman collection](#step-1-get-the-postman-collection)
  * [Step 2: Import the Postman environment](#step-2-import-the-postman-environment)
  * [Step 3: Set up Postman environment](#step-3-set-up-postman-environment)
* [Make API calls](#make-api-calls)
  * [A regular eCommerce payment](#a-regular-ecommerce-payment)
  * [An express checkout payment (*Vipps Hurtigkasse*)](#an-express-checkout-payment-vipps-hurtigkasse)
  * [Getting access to user info](#getting-access-to-user-info)
  * [Generating a QR code to the Vipps landing page](#generating-a-qr-code-to-the-vipps-landing-page)
* [Questions?](#questions)

<!-- END_TOC -->

Document version 1.0.1.

## Postman

[Postman](https://www.getpostman.com/) is a common tool for working with REST APIs.
We offer a [Postman Collection](https://www.getpostman.com/collection) to make development easier.
See the [Postman documentation](https://www.getpostman.com/docs/) for more information about using Postman.

By following the steps below, you can make calls to all the
endpoints, and see the full `request` and `response` for each call.

### Step 1: Get the Postman collection

Install Postman and import the collection by following the steps below:

1. Click `Import` in the upper-left corner.
2. Import the [vipps-ecom-api-postman-collection.json](./tools/vipps-ecom-api-postman-collection.json) file.

### Step 2: Import the Postman environment

1. Click `Import` in the upper-left corner.
2. Import the [vipps-ecom-api-postman-environment.json](./tools/vipps-ecom-api-postman-environment.json) file.

### Step 3: Set up Postman environment

1. Click the down arrow, next to the "eye" icon in the top-right corner, and select the environment you have imported.
2. Click the "eye" icon and, in the dropdown window, click `Edit` in the top-right corner.
3. Fill in the `Current Value` for the following fields to get started.
   For the first three keys, go to *Vipps Portal* > *Utvikler* ->  *Test Keys*.
   - `client_id` - Merchant key is required for getting the access token.
   - `client_secret` - Merchant key is required for getting the access token.
   - `Ocp-Apim-Subscription-Key` - Merchant subscription key.
   - `merchantSerialNumber` - Merchant id.
   - `mobileNumber` - The mobile number for the test app profile you have received or registered.
   - `amount` - The desired amount Integer value minimum 100 in øre.

## Make API calls

For all of the following, you will start by sending request `Get Access Token`.
This provides you with access to the API.

   The access token is valid for 1 hour in the test environment
   and 24 hours in the production environment.

See the
[API reference](https://vippsas.github.io/vipps-developer-docs/api/ecom)
for details about the calls.

### A regular eCommerce payment

1. Send request `Get Access Token`. This provides you with access to the API.

1. Send request `Initiate Payment`. This is to demonstrate a simple payment by using
   [`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-developer-docs/api/ecom#tag/Vipps-eCom-API/operation/initiatePaymentV3UsingPOST).

   The `orderId` and `vippsLandingPageUrl` variables are now in the environment
   of this Postman example and can be used for subsequent calls relating to this purchase.

   The response will be a URL to the Vipps landing page.
   Ctrl+click (Command-click on macOS) on the link that appears and it will take
   you to the Vipps landing page.
   The phone number of your test user should already be filled in, so you only have to click "Next".

   You have now confirmed the payment in Vipps, setting the payment status to reserved.

   Alternatively, you could
   [Generate a QR code to the Vipps Landing page](#generating-a-qr-code-to-the-vipps-landing-page).

1. Send request `Get Payment Details` for information about this payment by using
   [`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-developer-docs/api/ecom#tag/Vipps-eCom-API/operation/getPaymentDetailsUsingGET).

1. Send request `Capture Payment` to capture this payment with
   [`POST:/ecomm/v2/payments/{orderId}/capture`](https://vippsas.github.io/vipps-developer-docs/api/ecom#tag/Vipps-eCom-API/operation/capturePaymentUsingPOST).

1. Send request `Refund Payment` to refund this payment with
   [`POST:/ecomm/v2/payments/{orderId}/refund`](https://vippsas.github.io/vipps-developer-docs/api/ecom#tag/Vipps-eCom-API/operation/refundPaymentUsingPOST).

See
[Regular eCommerce payments](vipps-ecom-api.md#regular-ecommerce-payments)
for more information from the eCommerce API guide.

### An express checkout payment (*Vipps Hurtigkasse*)

1. Send request `Initiate Payment - Express Checkout`. This demonstrates the type
   of payment where the user selects their shipping methods within the Vipps app
   instead of on the website.

   To enable this functionality, you provide the object `staticShippingDetails`
   with your relevant details in the body of the call
   [`POST:/v3/psppayments/init/`](https://vippsas.github.io/vipps-developer-docs/api/ecom#tag/Vipps-eCom-API/operation/initiatePaymentV3UsingPOST).

   Ctrl+click on the link that appears and complete the payment authorization.

1. Send request `Get Payment Details` for information about this payment by using
   [`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-developer-docs/api/ecom#tag/Vipps-eCom-API/operation/getPaymentDetailsUsingGET).

1. This time, instead of capturing the order, cancel it. Send request `Cancel Payment`
   to cancel this payment with
   [`POST:/ecomm/v2/payments/{orderId}/cancel`](https://vippsas.github.io/vipps-developer-docs/api/ecom#tag/Vipps-eCom-API/operation/cancelPaymentRequestUsingPUT).

See
[Express checkout payments](vipps-ecom-api.md#express-checkout-payments)
for more information from the eCommerce API guide.

### Getting access to user info

1. Send request `Initiate Payment - Profile flow`. Provide the `scope` object in the
   [`POST:/v3/psppayments/init/`](https://vippsas.github.io/vipps-developer-docs/api/ecom#tag/Vipps-eCom-API/operation/initiatePaymentV3UsingPOST)
   call. This contains the information types that you want access to, separated
   by spaces (e.g., "name address email phoneNumber birthDate").

   Ctrl+click on the link that appears and complete the authorization.

2. Send request `Get Payment Details` with `orderId` for information about this
   payment by using [`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-developer-docs/api/ecom#tag/Vipps-eCom-API/operation/getPaymentDetailsUsingGET).

   Here, the user identifier, `sub`, is retrieved from the response and set as a variable.

3. Send request `Get Userinfo`. This uses
   [`GET:/vipps-userinfo-api/userinfo/{sub}`](https://vippsas.github.io/vipps-developer-docs/api/ecom#tag/Vipps-Userinfo-API/operation/getUserinfo)
   with the `sub` variable from the previous call.

See
[Userinfo](vipps-ecom-api.md#userinfo)
for more information.

### Generating a QR code to the Vipps landing page

When you run any of the `Initiate Payment` examples, the `vippsLandingPageUrl` variable gets set in the environment.
With this url, you can generate a QR code to take you to the Vipps landing page for a one-time payment.

1. Send request `Initiate Payment`.

   The `orderId` and `vippsLandingPageUrl` variables are now in the environment of this Postman example

1. Send request `Generate QR Code PNG`. This uses
   [`POST:/qr/v1`](https://vippsas.github.io/vipps-qr-api/#/One%20time%20payment%20QR/generateOtpQr)
   to provide a URL that can be used to show a QR code.

   Ctrl+click the link to see the QR code. Scanning the QR should open the test
   app on your phone and allow you to complete the one-time purchase.

This is done in cooperation with the Vipps QR API. See
[One-time payment QR](https://github.com/vippsas/vipps-qr-api#one-time-payment-qr)
for more details.

## Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-ecom-api/issues),
a [pull request](https://github.com/vippsas/vipps-ecom-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).

Sign up for our [Technical newsletter for developers](https://github.com/vippsas/vipps-developers/tree/master/newsletters).
