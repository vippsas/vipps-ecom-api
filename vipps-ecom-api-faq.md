<!-- START_METADATA
---
title: FAQ
sidebar_position: 25
pagination_next: null
---
END_METADATA -->

# Frequently asked questions

Here are the eCom API FAQs.
See the
[Vipps eCommerce API guide](vipps-ecom-api.md)
for all the details.

For more common Vipps questions, see:

* [Vipps API General FAQ](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/faqs)

<!-- START_COMMENT -->

ℹ️ Please use the new documentation:
[Vipps Technical Documentation](https://vippsas.github.io/vipps-developer-docs/).

<!-- END_COMMENT -->

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

### I have initiated an order but I can't find it

If you have multiple sales units: Make sure you use the correct API keys, and that
you are not attempting to use one sales unit's API keys to retrieve an order made
by a different sales unit.

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

## Express checkout

The Express checkout (*Vipps Hurtigkasse*) is a solution that allows the user to
automatically share their address information with the merchant and select a shipping option.

### Express checkout: Old and new flow

### Old flow

In the original flow for
[Express Checkout](https://vippsas.github.io/vipps-developer-docs/docs/APIs/ecom-api/vipps-ecom-api#express-checkout-payments),
it was easy for a user to overlook verifying the
address and shipping method. Both were
displayed on the payment confirmation page and no active selection was required.
This resulted in some users
inadvertently confirming the wrong address and possibly also selecting the
wrong shipping method.

### New flow

To alleviate problems from the original flow, it became mandatory to select the address
and shipping method. This was released as a minor update of Vipps.

The mandatory selection of address and shipping method worked well for all merchants that used Express checkout *as intended*,
but not for those using it as a quick hack to speed up payments, "just get some
info about the user", Covid-19 tracking, or something else.

The new and (for some) improved flow did not work well for all. One example:
Restaurant guests could not understand why a restaurant needed their address
and shipping method for the two beers they ordered from their table.
They had not noticed the need to select those before, but definitely did now.

So, we changed it back to the way it was before.

### Back to the old flow, with modifications

A new attempt to solve the problems changes the Express Checkout back to the original flow, with modifications.
Now, merchants must explicit specify the explicit checkout flow.

This is done by specifying `"useExplicitCheckoutFlow": true` in
[`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-developer-docs/api/ecom#tag/Vipps-eCom-API/operation/initiatePaymentV3UsingPOST).

So, after a long time of notifying merchants and partners how Express Checkout
*should* be used, we have changed the default behavior to
`"useExplicitCheckoutFlow": true`.

The following image illustrates the new express checkout flow (`"useExplicitCheckoutFlow": true`):
![New explicit checkout flow](images/vipps-ecom-confirm-express.png)

The old express checkout flow (`"useExplicitCheckoutFlow": false`):
![The old checkout flow](images/vipps-ecom-confirm-express-old.png)

**Please note:** If you only need the user's information, please use the
[Userinfo API](https://vippsas.github.io/vipps-developer-docs/docs/APIs/userinfo-api).

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

This error is shown in Vipps if you use *Vipps Hurtigkasse* (*Express checkout*) and respond
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

The [Vipps PSP API](https://vippsas.github.io/vipps-developer-docs/docs/APIs/psp-api) provides tokens
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

### What does the `status` suffix at the `fallBack` URL mean?

Please disregard it. The `status` suffix was phased out several years ago, and
is no longer documented. But, since some merchants *still* depend on in, it still provided.
