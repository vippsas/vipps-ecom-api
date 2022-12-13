<!-- START_METADATA
---
title: Quick start
sidebar_position: 10
---
END_METADATA -->

# Quick start

<!-- START_COMMENT -->

ℹ️ Please use the new documentation:
[Vipps Technical Documentation](https://vippsas.github.io/vipps-developer-docs/).

<!-- END_COMMENT -->

Use the eCom API to initiate payments, get payment details, capture payments, and refund payments.
You can also perform express checkout payments and get access to user info.

<!-- START_TOC -->

## Table of Contents

* [Postman](#postman)
  * [Prerequisites](#prerequisites)
  * [Step 1: Get the Vipps Postman collection and environment](#step-1-get-the-vipps-postman-collection-and-environment)
  * [Step 2: Import the Vipps Postman files](#step-2-import-the-vipps-postman-files)
  * [Step 3: Set up Postman environment](#step-3-set-up-postman-environment)
* [Make API calls](#make-api-calls)
  * [A regular eCommerce payment](#a-regular-ecommerce-payment)
  * [An express checkout payment (*Vipps Hurtigkasse*)](#an-express-checkout-payment-vipps-hurtigkasse)
  * [Getting access to user info](#getting-access-to-user-info)
  * [Generating a QR code to the Vipps landing page](#generating-a-qr-code-to-the-vipps-landing-page)
  * [Test with Force Approve](#test-with-force-approve)
* [Questions?](#questions)

<!-- END_TOC -->

## Postman

### Prerequisites

Review
[Vipps quick start guides](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/quick-start-guides) for information about getting your test environment set up.

### Step 1: Get the Vipps Postman collection and environment

Save the following files to your computer:

* [Vipps eCom API Postman collection](tools/vipps-ecom-api-postman-collection.json)
* [Vipps API Global Postman environment](https://raw.githubusercontent.com/vippsas/vipps-developers/master/tools/vipps-api-global-postman-environment.json)

### Step 2: Import the Vipps Postman files

1. In Postman, click *Import* in the upper-left corner.
1. In the dialog that opens, with *File* selected, click *Upload Files*.
1. Select the two files you have just downloaded and click *Import*.

### Step 3: Set up Postman environment

1. Click the down arrow, next to the "eye" icon in the top-right corner, and select the environment you have imported.
2. Click the "eye" icon and, in the dropdown window, click `Edit` in the top-right corner.
3. Fill in the `Current Value` for the following fields to get started.
   For the first keys, go to *Vipps Portal* > *Utvikler* ->  *Test Keys*.
   * `client_id` - Merchant key is required for getting the access token.
   * `client_secret` - Merchant key is required for getting the access token.
   * `Ocp-Apim-Subscription-Key` - Merchant subscription key.
   * `merchantSerialNumber` - Merchant id.
   * `mobileNumber` - The mobile number for the test app profile you have received or registered.

You can update any of the other environment variables. Be aware of this:

* Any currency amount must be an Integer value minimum 100 in øre.
* Most URLs must be `https`.

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
   *Ctrl+click* (*Command-click* on macOS) on the link that appears and it will take
   you to the Vipps landing page.
   The phone number of your test user should already be filled in, so you only have to click "Next".

   You have now confirmed the payment in Vipps, setting the payment status to reserved.

   Alternatively, you could
   [Generate a QR code to the Vipps Landing page](#generating-a-qr-code-to-the-vipps-landing-page).

1. Send request `Get Payment Details` for information about this payment by using
   [`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-developer-docs/api/ecom#tag/Vipps-eCom-API/operation/getPaymentDetailsUsingGET). You will see the details appear in the lower pane.

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
   [`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-developer-docs/api/ecom#tag/Vipps-eCom-API/operation/initiatePaymentV3UsingPOST).

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
   [`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-developer-docs/api/ecom#tag/Vipps-eCom-API/operation/initiatePaymentV3UsingPOST)
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
   [`POST:/qr/v1`](hhttps://vippsas.github.io/vipps-developer-docs/api/qr#tag/One-time-payment-QR/operation/generateOtpQr)
   to provide a URL that can be used to show a QR code.

   Ctrl+click the link to see the QR code. Scanning the QR should open the test
   app on your phone and allow you to complete the one-time purchase.

This is done in cooperation with the Vipps QR API. See
[One-time payment QR](https://vippsas.github.io/vipps-developer-docs/docs/APIs/qr-api/vipps-qr-api#one-time-payment-qr-codes)
for more details.

### Test with Force Approve

You can use the
[`Force Approve Payment`](https://vippsas.github.io/vipps-developer-docs/api/ecom#tag/Vipps-eCom-API/operation/integrationTestApprovePayment)
endpoint
to approve an eCom API payment without signing in to the Vipps MT app,
The endpoint is only available in our test environment.

**Important:** All test users must manually approve at least one payment in
Vipps (using the app) before "force approve" can be used for that user.
If this has not been done, you will get an error.
This is because the user needs to be registered as
"bankID verified" in the backend, and this is happens automatically in
the test environment when using Vipps (the app), but not with "force approve".

1. Send request `Initiate Payment`.

   The `orderId` and `payment token` variables are now in the environment of this Postman example.

1. Send request `Force Approve Payment`. The payment then should be approved.

1. You can check the details by sending `Get Payment Details`.

See [API guide: Testing](vipps-ecom-api#testing)

## Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-ecom-api/issues),
a [pull request](https://github.com/vippsas/vipps-ecom-api/pulls),
or [contact us](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/contact).

Sign up for our [Technical newsletter for developers](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/newsletters).
