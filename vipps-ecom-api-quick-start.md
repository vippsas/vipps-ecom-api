<!-- START_METADATA
---
draft: true
---
END_METADATA -->


# Vipps eCom API quick start

Technical information for Vipps partners.

Document version: 1.0.0.

<!-- START_TOC -->

# Table of contents

* [1 Get API keys](#1-get-api-keys)
* [2. Install the test app](#2-install-the-test-app)
* [3. Get access token](#3-get-access-token)
* [4. Initiate the payment:](#4-initiate-the-payment)
* [5. Receive the callback with the payment status](#5-receive-the-callback-with-the-payment-status)
* [6. Get the payment details](#6-get-the-payment-details)
* [7. Capture the payment](#7-capture-the-payment)
* [Questions?](#questions)

<!-- END_TOC -->

**TODO:**
- Recommend Postman and our official collection and environment, refer to the Postman guide.
- Link to ReDocusaurus for exeamples of requests and responses. Avoid duplicating them here, and more maintenance.

## 1. Get API keys

See:
[Getting started: Get credentials](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md#get-credentials).

## 2. Install the test app

See:
[The Vipps Test Environment (MT): Vipps test apps](https://github.com/vippsas/vipps-developers/blob/master/vipps-test-environment.md#vipps-test-apps)

## 3. Get access token

See:
[Getting Started: Get access token](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md#get-an-access-token)

## 4. Initiate the payment:

[`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/initiatePaymentV3UsingPOST).

The user can now confirm the payment in Vipps, setting the payment status to reserved.
See [Initiate](#initiate).

## 5. Receive the callback with the payment status

[`POST:[callbackPrefix]/v2/payments/{orderId}`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/transactionUpdateCallbackForRegularPaymentUsingPOST).

See [Callbacks](#callbacks).

## 6. Get the payment details

[`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getPaymentDetailsUsingGET).

This is optional, but _must_ be done if no callback has been received.
See
[Get payment details](#get-payment-details)
and
[Polling guidelines](vipps-ecom-api.md#polling-guidelines).

## 7. Capture the payment

[`POST:/ecomm/v2/payments/{orderId}/capture`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/capturePaymentUsingPOST).

See [Regular eCommerce payments](#regular-ecommerce-payments).

## Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-ecom-api/issues),
a [pull request](https://github.com/vippsas/vipps-ecom-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).

Sign up for our [Technical newsletter for developers](https://github.com/vippsas/vipps-developers/tree/master/newsletters).
