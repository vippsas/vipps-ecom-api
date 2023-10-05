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

The eCom API supports refunds with
[`POST:/ecomm/v2/payments/{orderId}/refund`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/refundPaymentUsingPOST).
For details on how to offer refunds, please refer to the documentation for your eCommerce solution.

See
[FAQ: Refunds](https://developer.vippsmobilepay.com/docs/faqs/refunds-faq)
for answers to more questions.

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
* It is not possible to specify
  [`scope`](https://developer.vippsmobilepay.com/docs/APIs/userinfo-api#userinfo-call-by-call-guide)
  with Express checkout.

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


## Why do I get `HTTP 403 Forbidden`?

Merchants that only have access to the
[Login API](https://developer.vippsmobilepay.com/docs/APIs/login-api)
and attempt to use the ePayment API (or eCom API) will get this error, with
`Merchant Not Allowed for Ecommerce Payment` in the response body.

This is because the compliance checks required for making payments are not
done for merchants that only need the Login API.
If you need access to the ePayment API (or eCom API), you can apply for this on
[portal.vipps.no](https://portal.vipps.no).

Partners can get this error if they use
[Partner keys](https://developer.vippsmobilepay.com/docs/partner/partner-keys),
but:

* Do not send the `Merchant-Serial-Number` header.
* Send a `Merchant-Serial-Number` header for a sales unit (MSN) that is not connected
  to them as a partner. The partner keys can only be used for sales units that are
  connected to them as a partner.

See: [HTTP headers](https://developer.vippsmobilepay.com/docs/common-topics/http-headers/).

## Why do I get `HTTP 429 Too Many Requests`?

We rate-limit some API endpoints to prevent incorrect usage.
The rate-limiting has nothing to do with our total capacity, but is
designed to stop incorrect use.

See [eCom API Rate limiting](./vipps-ecom-api.md#rate-limiting).

## Why do I get `HTTP 404 Not Found`?

It depends. You must check the body of the response.

It could be that you are attempting to call a non-existent API endpoint, and
it could be that you are using the API keys for one MSN for an `orderId`
that belongs to a different MSN.

See: [Why do I get `errorCode 35 "Requested Order not found"`?](#why-do-i-get-errorcode-35-requested-order-not-found).

## Why do I get `HTTP 500 Internal Server Error`?

Something *might* be wrong on our side, and we are working to fix it,

But: It's usually a problem with your request, and that our validation does not catch it.
In other words: We should have returned `HTTP 400 Bad Request`.

Some tips:

* Please make sure the JSON payload in your API request validates.
  That is the most common source of this type of error.
* You will get a 500 error if the MSN is sent as an integer:
  `merchantSerialNumber":654321` instead of a string: `merchantSerialNumber":"654321"`.
* The same will happen if the `amount` is not an integer.
* Please check the capitalization of the parameters.
  We will return `HTTP 500 Error` if the incorrect `fallback` is used instead of
  the correct `fallBack`.

Check both the HTTP response header and the response body from our API for errors.
For most errors the body contains an explanation of what went wrong.

**Please note:** If you get a `HTTP 500 Internal Server Error` in the test environment,
it may be a glitch in the SQL server. We are running a "weaker" instance than in
production, and on very rare occasions this can cause SQL errors that result in
a `HTTP 500 Server Error`. Retry the call, and see if it helps.

See:

* [Errors](./vipps-ecom-api.md#errors).
* [Status page](https://developer.vippsmobilepay.com/docs/developer-resources/status-pages/).

## Why do I get `errorCode 35 "Requested Order not found"`?

This is either because you are specifying an incorrect `orderId`, or because
the payment with this `orderId` was initiated using the API keys for
one sales unit (MSN), and you are attempting to get the details with
the API keys for a different sales unit (MSN).

The `orderId`s is not globally unique, they are only unique per MSN.

See:

* [Why do I get `HTTP 404 Not Found`?](#why-do-i-get-http-404-not-found)
* [Error codes](./vipps-ecom-api.md#error-codes).

## Why do I get `errorCode 37 "Merchant not available or deactivated or blocked"`?

Or: `Merchant not available or active`.

Please check that the merchant's organization number is still active in
[Brønnøysundregistrene](https://www.brreg.no). We automatically deactivate
merchants (companies) when they are deleted from Brønnøysundregistrene.
This can also happen if a merchant changes organization type, for instance
from *ENK* to *AS*.

Merchants can log in on
[portal.vipps.no](https://portal.vipps.no)
and deactivate their sales units. This is sometimes done "by accident", without being
aware of the consequences. If a sales unit has been incorrectly deactivated,
the merchant can reactivate it again.

**Please note:** We require BankID for deactivation and reactivation,
and cannot help with this based on email requests.

Deactivation can also happen if the test merchant is not being used for a
*very long* time. Please
[contact customer service](https://vipps.no/kontakt-oss/),
and we will reactivate the merchant.

Partners that use
[partner keys](https://developer.vippsmobilepay.com/docs/partner/partner-keys)
can also get this error if the partner itself is deactivated, even though
the sales unit (that it is acting on behalf of) is active.

**Please note:** We no longer automatically deactivate test merchants.
Merchants can [create new test sales units](https://developer.vippsmobilepay.com/docs/developer-resources/portal#how-to-create-a-test-sales-unit).

See: [Error codes](./vipps-ecom-api.md#error-codes).

## Why do I get "Merchant Not Allowed for Ecommerce Payment"?

This error occurs if you attempt to use a payment related API with a sales unit (MSN)
that is only approved for the Login API.

Your sales unit will need to go through a different set of regulatory and legally required checks
to get access to the payment APIs.
Place an order for *Vipps på Nett* through the
[merchant portal](https://portal.vipps.no).

Note that all sales units that have been approved for the eCom API can also use
the Login API, but not the other way around.

See:
[Why do I get `HTTP 403 Forbidden`?](#why-do-i-get-http-403-forbidden)

## Why do I get error 81 and `User not registered with Vipps`?

The most common reasons are:

* The phone number is incorrectly formatted.
  Vipps attempts to correct incorrectly formatted phone numbers
  instead of responding with `HTTP 400 Bad Request`.
  In cases where the phone number still fails, the error will be `errorCode: 81`.
  See the API specifications.
* The user is under 15 years old and cannot pay businesses.
* The phone number is not for a Vipps or MobilePay user.

Vipps MobilePay cannot give more details, as we cannot "leak" information about a user's
age, whether a user has been blocked, whether a user has reached its spending
limit, etc.

## Why do I get an error about having the app installed and being 15 years old?

This can happen when:

* You attempt to use a real user in the test environment.
* You have a new test user and have not logged into the test app before
  trying to make payments, etc.

See:
[Test Environment (MT)](https://developer.vippsmobilepay.com/docs/test-environment/).

## Why do I get `Invalid MSN: 654321. Check your API keys on portal.vipps.no and see the eCom FAQ for tips.`?

This can happen when:

* A partner tries to use
  [Partner keys](https://developer.vippsmobilepay.com/docs/partner/partner-keys)
  for a sales unit that is not registered with them as partner.
* API keys for the test environment is used in the production environment, or opposite.
* Partner keys are used, but the `Merchant-Serial-Number` HTTP header is not used correctly.

## Why do I get `Invalid MSN: 654321. This MSN is not valid for the provided supermerchant ID.`?

This can happen when the partner making the API request is using:

* API keys for the test environment in the production environment, or opposite
* An MSN for the test environment in the production environment, or opposite
* [Partner keys](https://developer.vippsmobilepay.com/docs/partner/partner-keys)
  without including the required `Merchant-Serial-Number` header

If the error message is `Invalid MSN: This MSN is not valid for the provided supermerchant ID.`,
with no MSN specified, it means that the `Merchant-Serial-Number` is missing in the request header.

## Why do I get `Invalid MSN: 654321. This MSN is not valid for the provided PSP id.`?

The full error message text is:

"Invalid MSN: 654321. This MSN is not valid for the provided PSP ID. Check that you are
using the correct credentials for the right environment."

In addition to what the error message says, this error can occur if a PSP attempts to initiate
payments for an MSN that was created by a different PSP.
PSP's can only initiate payments for MSNs that are connected to them.

The solution is to create a new MSN with the
[PSP Signup API](https://developer.vippsmobilepay.com/docs/APIs/psp-api/vipps-psp-signup-api/).

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






## POS integrations

### How do I use the one-time payment QR?

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

### Do we need to support callbacks?

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
