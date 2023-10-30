<!-- START_METADATA
---
title: eCom API changelog
sidebar_label: Changelog
sidebar_position: 200
pagination_next: null
pagination_prev: null
---
END_METADATA -->

# Changelog

All notable changes to the current API will be documented in this file.
To learn about API versioning, see
[API Lifecycle](https://developer.vippsmobilepay.com/docs/knowledge-base/api-lifecycle/).

## March 2023

The explicit express checkout flow (`"useExplicitCheckoutFlow": true`) is now
the only possible flow. This ensures that the user gets the item delivered to
the right place in the right way.

The `useExplicitCheckoutFlow` parameter no longer has any effect.

## October 2022

URL links in the body of
[`POST:/ecomm/v2/payments`](https://developer.vippsmobilepay.com/api/ecom#tag/eCom-API/operation/initiatePaymentV3UsingPOST)
now only support HTTPS.

This includes the following fields:

* callbackPrefix
* consentRemovalPrefix
* fallBack
* shippingDetailsPrefix

## April 2022

Added capability to cancel a partially captured order.
See:
[Cancelling a partially captured order](vipps-ecom-api.md#cancelling-a-partially-captured-order).

## April 2021

New date limits for capture and cancel.

Payments can be captured up to 365 days after reservation,
and can be cancelled up to 180 days after reservation.
Attempts at capturing and cancelling older payments will result in
a `HTTP 400 Bad Request` with more details in the request body.

## December 2020

eCom API v1 is disabled.

After several extensions to the original June 1 deadline, the eCom API v1
was shut down on December 4. The eCom API v2 has been available for
about three years, and offers more functionality than the old version.

## April 2020

### skipLandingPage

If there is no way to show the Vipps landing page, it can be skipped.

This may be useful for POS integration, vending machines, etc.
See
[Skip landing page](https://developer.vippsmobilepay.com/docs/knowledge-base/landing-page/#skip-landing-page)
for details.

### Cancel pending payments

The `/cancel` endpoint may now also be called *before* the payment has been
reserved, meaning before the user has accepted/rejected in Vipps. This may be
useful in face-to-face situations where a customer's phone runs out of battery.

See
[Cancelling a pending order](./vipps-ecom-api.md#cancelling-a-pending-order).


### `/approve` endpoint for integration tests

A new
[`/approve`](vipps-ecom-api.md#testing)
endpoint makes it possible to approve payments through the API,
without using the app.
