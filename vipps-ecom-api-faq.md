<!-- START_METADATA
---
title: eCom API Frequently Asked Questions
sidebar_label: FAQ
sidebar_position: 25
description: Frequently asked questions for the eCom API.
pagination_next: null
pagination_prev: null
---
END_METADATA -->

# Frequently asked questions

<!-- START_COMMENT -->
ℹ️ Please use the website:
[Vipps MobilePay Technical Documentation](https://developer.vippsmobilepay.com/docs/APIs/ecom-api/).
<!-- END_COMMENT -->

Here are the eCom API Frequently Asked Questions (FAQ).
See the
[eCom API guide](vipps-ecom-api.md)
for all the details.

For more common questions, see [Common API FAQ](https://developer.vippsmobilepay.com/docs/faqs).

## Common problems

See
[FAQ: Common problems](https://developer.vippsmobilepay.com/docs/faqs/common-problems-faq)
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
instead of Vipps Hurtigkasse, to avoid asking the customer in a pub
for the shipping method for the drinks, etc.

We strongly recommend checking the full history of every Vipps payment with
the API: You can see if a payment has been actively rejected, if the user has
not done anything, etc.
See: [Get payment details](vipps-ecom-api.md#get-payment-details).

See:
[All errors](vipps-ecom-api.md#error-codes).

## Reservations and captures

See [FAQ: Reservations and capture](https://developer.vippsmobilepay.com/docs/faqs/reserve-and-capture-faq/).

## Refunds

## Can I refund a payment?

The eCom API supports refunds with
[`POST:/ecomm/v2/payments/{orderId}/refund`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/refundPaymentUsingPOST).

All integrations with the eCom API *must* support refunds.

It is also possible to do refunds on
[portal.vipps.no](https://portal.vipps.no).

Refunds can be made up to 365 days after payment or reservation.
Very old payments have a higher risk of being problematic, because people
change banks, leave the country, die, etc.,
and this then requires time-consuming manual work.
We therefore limit refunds to 365 days.

### How can I refund only a part of a payment?

Example: A customer has placed an order of two items for a total of 1000 NOK.
The merchant has initiated a payment of 1000 NOK, but the customer has changed
her mind and only bought one of the items, with a price of 750 NOK. The merchant
has therefore made a
[partial capture](vipps-ecom-api.md#partial-capture)
of 750 NOK, and need to refund the remaining 250 NOK.

* The short version: This is done automatically by the bank after a few days.
See:
[For how long is a payment reserved?](https://developer.vippsmobilepay.com/docs/faqs/reserve-and-capture-faq/#for-how-long-is-a-payment-reserved).

* The long version: It *is* possible to cancel the remaining reservation after a
partial capture through the eCom API: Send a
[`PUT:/ecomm/v2/payments/{orderId}/cancel`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/cancelPaymentRequestUsingPUT)
request with `shouldReleaseRemainingFunds: true` in the body.
The payment must be `RESERVED` for this to take effect.
See:
[eCom API: Cancelling a partially captured order](vipps-ecom-api.md#cancelling-a-partially-captured-order).

The partial capture (the 750 of the 1000 NOK in the example above)
is normally confirmed in the bank after 3-10 days, but it sometimes takes even
longer. When this is done, the bank will make the remaining (250 NOK) available
in the customer's account again. This process depends entirely on the customer's
bank, and we can't speed it up.

Banks keep reservations for the same number of days regardless of whether there
has been one or more captures. Banks do not extend the reservation if a partial
capture has been made.

If a partial capture has been made, the bank cancels the reservation for the
remaining amount. If no capture has been made, the entire reserved amount is
cancelled. Banks "count the days" from when the reservation was made, so the
merchant must make the capture, or all captures, before the reservation expires.

See: [Settlements](https://developer.vippsmobilepay.com/docs/settlements/).

## Users and payments

See
[FAQ: Users and payments](https://developer.vippsmobilepay.com/docs/faqs/users-and-payments-faq)
for more questions.

### I have initiated an order but I can't find it

If you have multiple sales units: Make sure you use the correct API keys, and that
you are not attempting to use one sales unit's API keys to retrieve an order made
by a different sales unit.

Have you, or the eCommerce solution you are using, successfully implemented
[`GET:/ecomm/v2/payments/{orderId}/details`](vipps-ecom-api.md#get-payment-details)?
This is a requirement, see the
[API checklist](vipps-ecom-api-checklist.md).

In case the
[callback](vipps-ecom-api.md#callbacks)
fails, you will not automatically receive notification of order status.
The solution is to check with
[`GET:/ecomm/v2/payments/{orderId}/details`](vipps-ecom-api.md#get-payment-details).

See:
[API endpoints](vipps-ecom-api.md#api-endpoints).

## Express checkout

The
[Express checkout (*Vipps Hurtigkasse*)](./vipps-ecom-api.md#express-checkout-payments),
allows the user to automatically share their address information with the
merchant and select a shipping option.

It is mandatory for the user to select the address and shipping method.
This ensures that the user gets the item delivered to the right place in the right way.

The following images illustrate the express checkout flow:

![Express checkout flow](images/vipps-ecom-confirm-express_consent_shipping_options.png)

**Please note:**

* If you only need the user's information, you should use
  a normal payment and the
  [Userinfo API](https://developer.vippsmobilepay.com/docs/APIs/userinfo-api).
  In other words: Don't specify `"paymentType": "eComm Express Payment"`, but
  simply omit `paymentType` and specify the required
  [`scope`](https://developer.vippsmobilepay.com/docs/APIs/userinfo-api#scope).
* It is not possible to specify `scope`  with Express checkout.

## Common errors

See
[FAQ: Common errors](https://developer.vippsmobilepay.com/docs/faqs/common-errors-faq)
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
[Vipps request servers](https://developer.vippsmobilepay.com/docs/developer-resources/servers#vipps-request-servers).

Please check your own logs for any signs of problems. If your
`orderId` is `acme-shop-123-order123abc`: Search your logs for `acme-shop-123-order123abc`.

Check the API Dashboard on
[portal.vipps.no](https://portal.vipps.no)
for problems with the callbacks. The API Dashboard is in the *Utvikler* section.

### Why do I get `Payment failed`?

This error is shown in Vipps if you use *Vipps Hurtigkasse* (*Express checkout*) and respond
incorrectly to the request for
[`[shippingDetailsPrefix]/v2/payments/{orderId}/shippingDetails`](https://developer.vippsmobilepay.com/api/ecom#tag/Merchant-Endpoints/operation/fetchShippingCostUsingPOST).

Please verify that your response is correct.

Also consider using
[static shipping methods](vipps-ecom-api.md#shipping-and-static-shipping-details),
as it gives a faster payment process and a better user experience.

### Why do I not get the `sub` from `/details`?

If you use the correct `scope` in the payment initiation, but don't get the
`sub` in the response for `/details`: Check that you are following the
[`orderId` recommendations](https://developer.vippsmobilepay.com/docs/common-topics/orderid).
Very short `orderId` values don't work well with our database index, and may cause
an internal timeout, and we "have to" send the response without the `sub`.
We cannot enforce longer `orderId` values due to backwards compatibility.

## Other questions

See
[FAQ: Other questions](https://developer.vippsmobilepay.com/docs/faqs/other-faq)
for more questions.

### What functionality is included in the eCom API, but not the PSP API?

The [PSP API](https://developer.vippsmobilepay.com/docs/APIs/psp-api) provides tokens
that a PSP can use to charge a Vipps user's card. To put it simply, it is a
"card token lookup service". The payment is completed by the PSP, who sends an
update to Vipps about the success or failure.

The eCom API has some functionality that is not available in the PSP API:

1. Retry functionality: If the user attempts to pay with a card that is declined,
   the user can retry with a different card, while still in the same payment process.
   This results in a higher success rater for payments.
   The PSP API does not have this functionality, as it is the PSP, not Vipps,
   that make the charge.
2. [Express checkout (Vipps Hurtigkasse)](vipps-ecom-api.md#express-checkout-payments)
   is only available in the eCom API.
3. [Userinfo](vipps-ecom-api.md#userinfo):
   The eCom API offers the possibility for merchants to ask for the user's
   profile information as part of the payment flow: name, address, email, phone number, birthdate, etc.
4. When using the eCom API, Vipps handles soft-declines, 3-D Secure, BankID, etc.
   There is nothing a merchant needs to do.
   This give a consistent user experience and a very high completion rate.

### What does the `status` suffix at the `fallBack` URL mean?

Please disregard it. The `status` suffix was phased out several years ago, and
is no longer documented. But, since some merchants *still* depend on in, it still provided.

## How do I use the one-time payment QR?

This feature is for presenting a QR code for opening a payment request from a
customer-facing screen, so there is no need to manually input the mobile number.

Basic flow:

1. Initiate a eCom payment (`skipLandingPage` must be set to `false`)
2. Receive the payment URL as response
3. Post the payment URL to the
   [QR API](https://developer.vippsmobilepay.com/docs/APIs/qr-api)
4. Receive a URL to a QR code in PNG (Portable Network Graphics) format
5. Present the QR code on the customer-facing screen
6. The user scans the QR code with the Vipps or MobilePay app, or the camera app
7. The user pays (or cancels the payment) in the app
8. The fallback URL is triggered and will be presented on customers mobile phone

**Please note:**
The customer-facing screen will not show the fallback page. We recommend
presenting the result of the payment in some other way on the screen, and
also that an error messages if something went wrong.

If it is not possible for the POS solution to handle a fallback URL you may use one of the following options:

1. Set `fallbackURL` to be the merchant's website
2. Set `fallbackURL` to be this static page: [www.vipps.no/thankyoupage/](https://www.vipps.no/thankyoupage/)

See also:

* [QR API](https://developer.vippsmobilepay.com/docs/APIs/qr-api)
* [Error codes](./vipps-ecom-api.md#error-codes)
* [Do we need to support callbacks?](#do-we-need-to-support-callbacks)

## Do we need to support callbacks?

Please try to implement the required callbacks, even if you do not use the data
provided in the callback.

If it's not possible for your POS to support callbacks (no fixed hostname/IP, etc.),
you must actively check the payment status.

### How can I check if a person has Vipps?

There is no separate API for this, but an attempt to initiate a payment
with a phone number that is not registered with Vipps will fail with error 81,
`User not registered with Vipps`.
See: [Error codes](./vipps-ecom-api.md#error-codes).

Users that install the app accept the terms and conditions, including being
"looked up" by the merchant if the payment is initiated with the phone number
is specified. It is possible to pay with Vipps without sharing the
phone number with the merchant.

See also
[vipps.no: privacy and terms](https://vipps.no/vilkar/).

There are users with unlisted numbers, users with secret number, etc.
These users can still pay with Vipps, since their phone number is
not shared with anyone without their explicit consent.
