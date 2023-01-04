<!-- START_METADATA
---
title: FAQ
sidebar_position: 25
pagination_prev: null
---
END_METADATA -->

# Vipps eCommerce API: Frequently Asked Questions

<!-- START_COMMENT -->

ℹ️ Please use the new documentation:
[Vipps Technical Documentation](https://vippsas.github.io/vipps-developer-docs/).

<!-- END_COMMENT -->

Here are the eCom API FAQs.
See the
[Vipps eCommerce API guide](vipps-ecom-api.md)
for all the details.

For more common Vipps questions, see:

* [Vipps API General FAQ](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/faqs/)

<!-- START_TOC -->

## Table of contents

* [Common problems](#common-problems)
  * [Why does Vipps Hurtigkasse (express checkout) fail?](#why-does-vipps-hurtigkasse-express-checkout-fail)
* [Refunds](#refunds)
* [Users and payments](#users-and-payments)
  * [I have initiated an order but I can't find it!](#i-have-initiated-an-order-but-i-cant-find-it)
* [Common errors](#common-errors)
  * [Why do I not get callbacks from Vipps?](#why-do-i-not-get-callbacks-from-vipps)
  * [Why do I get `Payment failed`?](#why-do-i-get-payment-failed)
  * [Why do I not get the `sub` from `/details`?](#why-do-i-not-get-the-sub-from-details)
* [Other questions](#other-questions)
  * [What functionality is included in the eCom API, but not the PSP API?](#what-functionality-is-included-in-the-ecom-api-but-not-the-psp-api)
* [Questions?](#questions)

<!-- END_TOC -->

## Common problems

See
[FAQ: Common problems](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/faqs/common-problems-faq)
for more questions.

### Why does Vipps Hurtigkasse (express checkout) fail?

When using Vipps Hurtigkasse (express checkout), Vipps makes a callback to the
merchant's server to retrieve the merchant's shipping methods for the user's
address. Vipps sends the user's address (with the user's consent) to the
merchant, and the merchant responds with the shipping methods and cost.

If the merchant's server is slow, or has a slow internet connection,
the delay may cause Vipps Hurtigkasse to fail due to a timeout.

The solution to this is a faster server and internet connection, or to provide
the shipping methods as part of the payment initiation. See:
[Express checkout API endpoints required on the merchant side](vipps-ecom-api.md#express-checkout-api-endpoints-required-on-the-merchant-side).

**Please note:** If you are not shipping any products you should use
[Userinfo](vipps-ecom-api.md#userinfo)
instead of Vipps Hurtigkasse, so you avoid asking the customer in a pub
for the shipping method for the drinks, etc.

We strongly recommend to check the full history of every Vipps payment with
the API: You can see if a payment has been actively rejected, if the user has
not done anything, etc.
See: [Get payment details](vipps-ecom-api.md#get-payment-details).

See:
[All errors](vipps-ecom-api.md#error-codes).

## Reservations and captures

## Refunds

The Vipps eCom API supports refunds with
[`POST:/ecomm/v2/payments/{orderId}/refund`](https://vippsas.github.io/vipps-developer-docs/api/ecom#tag/Vipps-eCom-API/operation/refundPaymentUsingPOST).
For details on how to offer refunds, please refer to the documentation for your eCommerce solution.

See
[Vipps FAQ: Refunds](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/faqs/refunds-faq)
for answers to more questions.

## Users and payments

See
[FAQ: Users and payments](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/faqs/users-and-payments-faq)
for more questions.

### I have initiated an order but I can't find it!

If you have multiple sale units: Make sure you use the correct API keys, and that
you are not attempting to use one sale unit's API keys to retrieve an order made
by a different sale unit.

Have you, or the eCommerce solution you are using, successfully implemented
[`GET:/ecomm/v2/payments/{orderId}/details`](vipps-ecom-api.md#get-payment-details)?
This is a requirement, see the
[API checklist](vipps-ecom-api-checklist.md).

In case the Vipps
[callback](vipps-ecom-api.md#callbacks)
fails, you will not automatically receive notification of order status.
The solution is to check with
[`GET:/ecomm/v2/payments/{orderId}/details`](vipps-ecom-api.md#get-payment-details).

You can use
[Postman](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/quick-start-guides)
to manually do API calls, Use the "inspect" functionality to see the complete requests and responses.

See:
[API endpoints](vipps-ecom-api.md#api-endpoints).

## Common errors

See
[FAQ: Common errors](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/faqs/common-errors-faq)
for more questions.

### Why do I not get callbacks from Vipps?

Please make sure the URLs you provide to Vipps are reachable from outside your
own environment.

Have a look at the
[Callback](vipps-ecom-api.md#callback-endpoints)
section in the API guide, and see
[How to test your own callbacks](vipps-ecom-api.md#how-to-test-your-own-callbacks).

If you do not receive a callback, it could be because your firewall is blocking
our requests. See:
[Vipps request servers](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/developer-resources/servers#vipps-request-servers).

Please check your own logs for any signs of problems. If your
`orderId` is `acme-shop-123-order123abc`: Search your logs for `acme-shop-123-order123abc`.

Check the API Dashboard on
[portal.vipps.no](https://portal.vipps.no)
for problems with the callbacks. The API Dashboard is under "Utvikler".

### Why do I get `Payment failed`?

This error is shown in Vipps if you use _Vipps Hurtigkasse_ (_Express checkout_) and respond
incorrectly to the request for
[`[shippingDetailsPrefix]/v2/payments/{orderId}/shippingDetails`](https://vippsas.github.io/vipps-developer-docs/api/ecom#tag/Merchant-Endpoints/operation/fetchShippingCostUsingPOST).

Please verify that your response is correct.

Also consider using
[static shipping methods](vipps-ecom-api.md#shipping-and-static-shipping-details),
as it gives a faster payment process and a better user experience.

## Why do I not get the `sub` from `/details`?

If you use the correct `scope` in the payment initiation, but don't get the
`sub` in the response for `/details`: Check that you are following the
[orderId recommendations](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/common-topics/orderid).
Very short orderIds don't work well with our database index, and may cause
an internal timeout, and we "have to" send the response without the `sub`.
We cannot enforce longer orderIds due to backwards compatibility.

## Other questions

See
[FAQ: Other questions](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/faqs/other-faq)
for more questions.

### What functionality is included in the eCom API, but not the PSP API?

The [Vipps PSP API](https://vippsas.github.io/vipps-developer-docs/docs/APIs/psp-api/) provides tokens
that a PSP can use to charge a Vipps user's card. To put it simply, it is a
"card token lookup service". The payment is completed by the PSP, who sends an
update to Vipps about the success or failure.

The Vipps eCom API has some functionality that is not available in the PSP API:

1. Retry functionality: If the user attempts to pay with a card that is declined,
   the user can retry with a different card, while still in the same payment process.
   This results in a higher success rater for payments.
   The PSP API does not have this functionality, as it is the PSP, not Vipps,
   that make the charge.
2. [Express checkout (Vipps Hurtigkasse)](vipps-ecom-api.md#express-checkout-payments)
   is only available in the Vipps eCom API.
3. [Userinfo](vipps-ecom-api.md#userinfo):
   The Vipps eCom API offers the possibility for merchants to ask for the user's
   profile information as part of the payment flow: name, address, email, phone number, birthdate, etc.
4. When using the Vipps eCom API, Vipps handles soft-declines, 3-D Secure, BankID, etc.
   There is nothing a merchant needs to do.
   This give a consistent user experience and a very high completion rate.

## Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-ecom-api/issues),
a [pull request](https://github.com/vippsas/vipps-ecom-api/pulls),
or [contact us](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/contact).

Sign up for our [Technical newsletter for developers](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/newsletters).
