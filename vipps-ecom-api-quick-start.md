<!-- START_METADATA
---
title: Quick start
sidebar_position: 10
---
END_METADATA -->

# Quick start

Use the eCom API to initiate payments, get payment details, capture payments, and refund payments.
You can also perform express checkout payments and get access to user info.

<!-- START_COMMENT -->

ℹ️ Please use the new documentation:
[Vipps Technical Documentation](https://vippsas.github.io/vipps-developer-docs/).

## Table of contents

* [Postman](#postman)
  * [Import Postman files and set up the environment](#import-postman-files-and-set-up-the-environment)
* [Make API calls](#make-api-calls)
  * [A regular eCommerce payment](#a-regular-ecommerce-payment)
  * [An express checkout payment (*Vipps Hurtigkasse*)](#an-express-checkout-payment-vipps-hurtigkasse)
  * [Getting access to user info](#getting-access-to-user-info)
  * [Generating a QR code to the Vipps landing page](#generating-a-qr-code-to-the-vipps-landing-page)

<!-- END_COMMENT -->

## Postman

See
[Vipps quick start guides](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/quick-start-guides)
and
[The Vipps Test Environment (MT)](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/test-environment).

### Import Postman files and set up the environment

Import the Postman files:
* [eCom API Postman collection](tools/vipps-ecom-api-postman-collection.json)
* [API Global Postman environment](https://raw.githubusercontent.com/vippsas/vipps-developers/master/tools/vipps-api-global-postman-environment.json)

Specify the variables required for using the eCom API
(see [Get credentials](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/vipps-getting-started#get-credentials)):
 * `client_id` - The "username" for making API requests
 * `client_secret` - The "password" for making API requests
 * `Ocp-Apim-Subscription-Key` - The subscription key for making API requests
 * `merchantSerialNumber` - The unique id for your sale unit.
 * `mobileNumber` - The phone number for your
   [test user](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/test-environment#test-users).

## Make API calls

For all API calls: You must first
[get an access token](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/vipps-getting-started#get-an-access-token):
Send request `Get Access Token`.

### A regular eCommerce payment

2. Send request `Initiate Payment`.
   Open the Vipps landing page URL from the response in a browser and specify your phone number.
2. Confirm the payment in Vipps.   
   **Please note:** You can also use
   [`Force approve payment`](https://vippsas.github.io/vipps-developer-docs/docs/APIs/ecom-api/vipps-ecom-api#testing).
3. Optional: Send request `Get Payment Details` for information about this payment.
4. Send request `Capture Payment` to capture the payment.

See
[Regular eCommerce payments](vipps-ecom-api.md#regular-ecommerce-payments)
for more information.

### An express checkout payment (*Vipps Hurtigkasse*)

1. Send request `Initiate Payment - Express Checkout`.
   Open the Vipps landing page URL from the response in a browser and specify your phone number.
2. Confirm the payment in Vipps.   
   **Please note:** You can also use
   [`Force approve payment`](https://vippsas.github.io/vipps-developer-docs/docs/APIs/ecom-api/vipps-ecom-api#testing).
2. Optional: Send request `Get Payment Details` for information about this payment.
3. Send request `Capture Payment` to capture the payment.

See
[Express checkout payments](vipps-ecom-api.md#express-checkout-payments)
for more information.

### Getting access to user info

1. Send request `Initiate Payment - Profile flow`.
   Open the Vipps landing page URL from the response in a browser and specify your phone number.
2. Confirm the payment in Vipps.   
   **Please note:** You can also use
   [`Force approve payment`](https://vippsas.github.io/vipps-developer-docs/docs/APIs/ecom-api/vipps-ecom-api#testing).
2. Send request `Get Payment Details`.
   The response contains the user identifier: `sub`.
3. Send request `Get Userinfo`, which uses the `sub` from the previous call.

See
[Userinfo](vipps-ecom-api.md#userinfo)
for more information.

### Generating a QR code to the Vipps landing page

When you run any of the `Initiate Payment` examples, the `vippsLandingPageUrl` variable gets set in the environment.
With this URL, you can generate a QR code to take you to the Vipps landing page for a one-time payment.

1. Send request `Initiate Payment`.
1. Send request `Generate QR Code PNG`.
   This response contains a URL to a QR code.

This is done with the Vipps QR API. See
[One-time payment QR](https://vippsas.github.io/vipps-developer-docs/docs/APIs/qr-api/vipps-qr-api#one-time-payment-qr-codes)
for more details.
