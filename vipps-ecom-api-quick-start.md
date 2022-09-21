<!-- START_METADATA
---
title: Quick Start
sidebar_position: 12
---
END_METADATA -->


# Quick Start

Technical information for Vipps partners.

Document version: 1.1.0.

<!-- START_TOC -->

# Table of contents

* [1. Get API keys](#1-get-api-keys)
* [2. Install the test app](#2-install-the-test-app)
* [3. Get access token](#3-get-access-token)
* [4. Initiate the payment:](#4-initiate-the-payment)
* [5. Receive the callback with the payment status](#5-receive-the-callback-with-the-payment-status)
* [6. Get the payment details](#6-get-the-payment-details)
* [7. Capture the payment](#7-capture-the-payment)
* [Questions?](#questions)

<!-- END_TOC -->

We strongly recommend using Postman to make API calls and see exactly what is
sent in the requests and responses.
See the
[Postman guide](vipps-ecom-postman.md)
for more details.

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

[`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-developer-docs/api/ecom#tag/Vipps-eCom-API/operation/initiatePaymentV3UsingPOST)

The response will be a URL to the Vipps landing page.
Copy this URL and open it in a browser.
The phone number of your test user should already be filled in if you are using
our Postman collection, so you only have to click "Next".

The user can now confirm the payment in Vipps, setting the payment status to reserved.
See [Initiate](#initiate).

You may have to "pull to refresh" in the test app to see the payment.

## 5. Receive the callback with the payment status

Vipps now makes a callback to the URL specified in `callbackPrefix`:

[`POST:[callbackPrefix]/v2/payments/{orderId}`](https://vippsas.github.io/vipps-developer-docs/api/ecom#tag/Merchant-Endpoints/operation/transactionUpdateCallbackForRegularPaymentUsingPOST)

See [Callbacks](#callbacks).

## 6. Get the payment details

In addition to receiving the callback you can poll the status of the payment:

[`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-developer-docs/api/ecom#tag/Vipps-eCom-API/operation/getPaymentDetailsUsingGET)

This is optional, but _must_ be done if no callback has been received.
See
[Get payment details](#get-payment-details)
and
[Polling guidelines](vipps-ecom-api.md#polling-guidelines).

## 7. Capture the payment

When a payment has been reserved (confirmed by the user in the Vipps app)
you can capture it:

[`POST:/ecomm/v2/payments/{orderId}/capture`](https://vippsas.github.io/vipps-developer-docs/api/ecom#tag/Vipps-eCom-API/operation/capturePaymentUsingPOST)

See [Regular eCommerce payments](#regular-ecommerce-payments).

## Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-ecom-api/issues),
a [pull request](https://github.com/vippsas/vipps-ecom-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).

Sign up for our [Technical newsletter for developers](https://github.com/vippsas/vipps-developers/tree/master/newsletters).
