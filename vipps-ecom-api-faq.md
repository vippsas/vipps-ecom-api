# Vipps eCommerce API: Frequently Asked Questions

See the
[Vipps eCommerce API](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md)
for all the details.

See also:
[Vipps Recurring API FAQ](https://github.com/vippsas/vipps-recurring-api/blob/master/vipps-recurring-api-faq.md).

See also:
[Getting Started](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md)
guide.

Document version 3.1.0.

### Table of contents

- [Requirements](#requirements)
  - [What are the requirements for Vipps merchants?](#what-are-the-requirements-for-vipps-merchants)
  - [Can I use my "Vipps-nummer" in my webshop?](#can-i-use-my-vipps-nummer-in-my-webshop)
- [Common problems](#common-problems)
  - [Why do payments fail?](#why-do-payments-fail)
  - [Why does capture fail?](#why-does-capture-fail)
  - [Why do I get a CORS error?](#why-do-i-get-a-cors-error)
  - [How can I open the fallback URL in a specific (embedded) browser?](#how-can-i-open-the-fallback-url-in-a-specific-embedded-browser)
  - [How can I measure Vipps sales with Google Analytics, Facebook pixel, etc?](#how-can-i-measure-vipps-sales-with-google-analytics-facebook-pixel-etc)
  - [Why does Vipps Hurtigkasse (express checkout) fail?](#why-does-vipps-hurtigkasse-express-checkout-fail)
- [Reservations and captures](#reservations-and-captures)
  - [What is the difference between "Reserve Capture" and "Direct Capture"?](#what-is-the-difference-between-reserve-capture-and-direct-capture)
  - [How can I check if I have "reserve capture" or "direct capture"?](#how-can-i-check-if-i-have-reserve-capture-or-direct-capture)
  - [How do I turn _direct capture_ on or off?](#How-do-I-turn-direct-capture-on-or-off)
  - [For how long is an initiated payment reserved?](#for-how-long-is-an-initiated-payment-reserved)
  - [Can I prevent people from paying with credit cards?](#can-i-prevent-people-from-paying-with-credit-cards)
  - [Can I initiate a Vipps payment with a QR code?](#can-i-initiate-a-vipps-payment-with-a-qr-code)
  - [Can I send a Vipps payment link in an SMS, QR or email?](#can-i-send-a-vipps-payment-link-in-an-sms-qr-or-email)
  - [Can I whitelist my URL for a Vipps QR?](#can-i-whitelist-my-url-for-a-vipps-qr)
- [Refunds](#refunds)
  - [How can I refund a payment?](#how-can-i-refund-a-payment)
  - [How can I refund only a part of a payment?](#how-can-i-refund-only-a-part-of-a-payment)
  - [Is it possible for a merchant to pay a Vipps user?](#is-it-possible-for-a-merchant-to-pay-a-vipps-user)
- [Users and payments](#users-and-payments)
  - [Is there an API for retrieving information about a Vipps user?](#is-there-an-api-for-retrieving-information-about-a-vipps-user)
  - [Is there an API for retrieving information about a merchant's payments?](#is-there-an-api-for-retrieving-information-about-a-merchants-payments)
  - [Is it possible to skip the landing page?](#is-it-possible-to-skip-the-landing-page)
  - [Can I split payments to charge a fee?](#can-i-split-payments-to-charge-a-fee)
  - [Can I create a marketplace with multiple merchants?](#can-i-create-a-marketplace-with-multiple-merchants)
  - [Can I create a service to match buyers and sellers?](#can-i-create-a-service-to-match-buyers-and-sellers)
  - [I have initiated an order but I can't find it!](#i-have-initiated-an-order-but-i-cant-find-it)
  - [How long is an initiated order valid, if the user does not confirm in the Vipps app?](#how-long-is-an-initiated-order-valid-if-the-user-does-not-confirm-in-the-vipps-app)
  - [How long does it take until the money is in my account?](#how-long-does-it-take-until-the-money-is-in-my-account)
  - [How long does it take from a refund is made until the money is in the customer's account?](#how-long-does-it-take-from-a-refund-is-made-until-the-money-is-in-the-customers-account)
  - [In which sequence are callbacks and fallbacks done?](#in-which-sequence-are-callbacks-and-fallbacks-done)
  - [Where can I find reports on transactions?](#where-can-i-find-reports-on-transactions)
- [Problems for end users](#problems-for-end-users)
  - [Why don't I receive the payment notification?](#why-dont-i-receive-the-payment-notification)
  - [Why am I not sent back to where I came from when I have paid?](#why-am-i-not-sent-back-to-where-i-came-from-when-i-have-paid)
  - [Why can't I scan the Vipps QR on the terminal with the camera app?](#why-cant-i-scan-the-vipps-qr-on-the-terminal-with-the-camera-app)
- [Common errors](#common-errors)
  - [Why do I not get callbacks from Vipps?](#why-do-i-not-get-callbacks-from-vipps)
  - [Why do I get `Access denied due to invalid subscription key`?](#why-do-i-get-access-denied-due-to-invalid-subscription-key)
  - [I am getting `401 Unauthorized` error - and I have double checked all my keys!](#i-am-getting-401-unauthorized-error---and-i-have-double-checked-all-my-keys)
  - [Why do I get `Requested Order not found`?](#why-do-i-get-requested-order-not-found)
  - [Why do I get `429 Too Many Requests`?](#why-do-i-get-429-too-many-requests)
  - [Why do I get `500 Internal Server Error` (or similar)?](#why-do-i-get-500-internal-server-error-or-similar)
  - [Why do I get `errorCode 37 "Merchant not available or deactivated or blocked"`](#why-do-i-get-errorcode-37-merchant-not-available-or-deactivated-or-blocked)
  - [Why do I get `errorCode 35 "Requested Order not found"`?](#why-do-i-get-errorcode-35-requested-order-not-found)
  - [Why do I not get the `sub` from `/details`?](#why-do-i-not-get-the-sub-from-details)
- [Other questions](#other-Questions)
  - [How do I perform "testing in production"?](#how-do-i-perform-testing-in-production)
  - [What do we have to do with PSD2's SCA requirements?](#what-do-we-have-to-do-with-psd2s-sca-requirements)
  - [How can I use Vipps for different types of payments?](#how-can-i-use-vipps-for-different-types-of-payments)
  - [How do I set up multiple sale units?](#how-do-i-set-up-multiple-sale-units)
  - [What about webhooks?](#what-about-webhooks)
  - [Can I use Vipps with Klarna Checkout?](#can-i-use-vipps-with-klarna-checkout)
- [Frequently Asked Questions for POS integrations](#frequently-asked-questions-for-pos-integrations)
  * [What is the process to go live in production?](#what-is-the-process-to-go-live-in-production)
  * [How can we be whitelisted for `skipLandingPage`?](#how-can-we-be-whitelisted-for-skiplandingpage)
  * [Which API keys should I use?](#which-api-keys-should-i-use)
  * [Do we need to support callbacks?](#do-we-need-to-support-callbacks)
  * [How can I check if a person has Vipps?](#how-can-i-check-if-a-person-has-vipps)
  * [How can I save the customer's phone number?](#how-can-i-save-the-customers-phone-number)
  * [How can we mass sign up merchants?](#how-can-we-mass-sign-up-merchants)
  * [Where can I find information about settlements?](#where-can-i-find-information-about-settlements)
- [Questions?](#questions)

## Requirements

### What are the requirements for Vipps merchants?

Vipps merchants (corporate customers) must have a Norwegian organization number
and applications must be signed with Norwegian BankID. Vipps must follow the
regulatory requirements for KYC (Know Your Customer), AML (Anti Money Laundering)
and other risk assessment procedures.

See:
[Requirements](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md#requirements)
in the
[Getting Started](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md)
guide.

### Can I use my "Vipps-nummer" in my webshop?

No.
[Vipps-nummer](https://vipps.no/produkter-og-tjenester/bedrift/ta-betalt-i-butikk/ta-betalt-med-vipps/)
can't be used for
[Vipps på Nett](https://vipps.no/produkter-og-tjenester/bedrift/ta-betalt-paa-nett/ta-betalt-paa-nett/),
[Vipps Logg Inn](https://vipps.no/produkter-og-tjenester/bedrift/logg-inn-med-vipps/logg-inn-med-vipps/)
or
[Vipps Faste Betalinger](https://vipps.no/produkter-og-tjenester/bedrift/faste-betalinger/faste-betalinger/).
You need
[Vipps på Nett](https://www.vipps.no/produkter-og-tjenester/bedrift/ta-betalt-paa-nett/ta-betalt-paa-nett/).

The reason for this is that the Norwegian Tax Administration considers
"Vipps-nummer" to be cash transactions,
while eCommerce is defined as remote sales ("fjernhandel"). The regulations
associated with both are different enough to require this policy.

## Common problems

### Why do payments fail?

The most common reasons are:

1. The debit/credit card has expired
2. The debit/credit is no longer valid (typically when a user has received a new
   card, but the previous card's expiry date has not yet been reached)
3. Insufficient funds on the debit/credit card (not enough money in the debit
   card's bank account, or not enough credit left on the credit card)
4. The debit/credit card has been rejected by the issuer
5. Payment limit reached, the user needs to authenticate with bankID in Vipps.
6. The payment has timed out (this happens if the user does not confirm in Vipps
   within 5 minutes - typically if the user has deactivated push notifications)
7. Attempt to capture an amount that exceeds the reserved amount
8. Attempt to capture an amount that has not been reserved

We are continuously improving the error messages in the Vipps app. Please note
that we are not allowed to give detailed information about all errors to the
merchant, as some information should only be provided to the Vipps user.

See the API guide for
[all errors](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#error-codes).

### Why does capture fail?

If the reserved amount is too low for shipping costs to be included, the capture will fail.
The reserved amount must at least as high as the amount that is captured.

Example: If the value of the shopping cart is 1000 NOK, and the reserved amount is 1100 NOK,
the shipping cost must be maximum 100 NOK. If the shipping cost is 150 kr, a capture of
1000 + 150 kr = 1150 NOK will fail.

The API responds with details about the error.

### Why do I get a CORS error?

If you get a CORS (Cross-Origin Resource Sharing) error, it is from your side,
not an error from Vipps. You are most likely attempting to call the Vipps API
from a website, and your webserver's configuration prevents it.

CORS is a protocol that enables scripts running on a browser client to interact
with resources from a different origin. Sometimes servers are configured to
prevent this, and that results in a CORS error.

Vipps only receives the API requests over HTTPS, and has no way of detecting
how the request was made on the caller side - it all looks the same.

You can read more about CORS here:
[CORS Tutorial: A Guide to Cross-Origin Resource Sharing](https://auth0.com/blog/cors-tutorial-a-guide-to-cross-origin-resource-sharing/).

### How can I open the fallback URL in a specific (embedded) browser?

The phone's operating system always opens URLs in the default browser.
This means that the `fallback` URL (the "result page") will be opened in
the default browser. Vipps has no way to open the `fallback` URL in the
embedded browser in Facebook, Instagram, etc. Similarly there is no way
for Vipps to open the `fallback` URL in the same tab that the user came from
before the app-switch.

This means that the merchant must be able to detect or recognize the user
when the `fallback` URL is opened, without relying on session, cookies, etc.

See:
[Recommendations regarding handling redirects](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#recommendations-regarding-handling-redirects).

### How can I measure Vipps sales with Google Analytics, Facebook pixel, etc?

Vipps does not have any functionality for measuring sales with with Google
Analytics, Facebook pixel, etc. Merchants may of course use any service on
their own website, and use a fallback URL (the "result page") to track any
activity. This must be done by the merchant itself.

See:
[Initiate payment flow: Phone and browser](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#initiate-payment-flow-phone-and-browser).

### Why does Vipps Hurtigkasse (express checkout) fail?

When using Vipps Hurtigkasse (express checkout), Vipps makes a callback to the
customer's server to retrieve the merchant's shipping methods for the user's
address. If the merchant's server is slow, or has a slow internet connection,
the delay may cause Vipps Hurtigkasse to fail due to a timeout.

The solution to this is a faster server and internet connection, or to provide
the shipping methods as part of the payment initiation. See:
[Express checkout API endpoints required on the merchant side](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#express-checkout-api-endpoints-required-on-the-merchant-side).

**Please note:** If you are not shipping any products you should use
[Userinfo](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#userinfo)
instead of Vipps Hurtigkasse, so you avoid asking the customer in a pub
for the shipping method for the drinks, etc.

## Reservations and captures

### What is the difference between "Reserve Capture" and "Direct Capture"?

When you initiate a payment it will be reserved until you capture it.
Vipps supports both _reserve-capture_ and _direct capture_.

_Reserve capture_ is the default. When you initiate a payment it will be
reserved until you capture it.

When _direct capture_ is activated, all payment reservations will instantly be
captured. This is intended for situations where the product or service is
immediately provided to the customer, and there is no chance that the
service is not available or sold out, e.g. digital services.

If a payment has been _reserved_, the merchant can make a `/cancel` call to
immediately release the reservation and make available in the customer's account.

If a payment has been _captured_, the merchant has to
make a `/refund` call, and it then takes a few days before the amount is
available in the customer's account.
See:
[How long does it take from a refund is made until the money is in the customer's account?](#how-long-does-it-take-from-a-refund-is-made-until-the-money-is-in-the-customers-account)

According to Norwegian regulations you should _not_ capture a payment until the
product or service is provided to the customer.
For more information, please see the Consumer Authority's
[Guidelines for the standard sales conditions for consumer purchases of goods over the internet](https://www.forbrukertilsynet.no/english/guidelines/guidelines-the-standard-sales-conditions-consumer-purchases-of-goods-the-internet).

See: [Regular eCommerce payments](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#regular-ecommerce-payments) for more details.

### How can I check if I have "reserve capture" or "direct capture"?

All customers can log in on
[portal.vipps.no](https://portal.vipps.no)
and check the capture type for all their sale units.

You can also find information on how to change capture type there.
We require BankID login for this, as "direct capture" requires additional
compliance checks.

If you are not able to log in on
[portal.vipps.no](https://portal.vipps.no)
you can make a small payment (2 kr), check the payment with
[`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getPaymentDetailsUsingGET),
and cancel (if it was `RESERVE` and reserve capture) or refund (if it was `SALE` and direct capture).

### How do I turn direct capture on or off?

You can't turn _direct capture_ on or off as a merchant. This must be
requested of your Key Account Manager. If you do not have a KAM:
Please log in on
[portal.vipps.no](https://portal.vipps.no),
find the right sale unit and click the email link under the "i" information bubble.

To get both _direct capture_ and _reserve capture_ you must request two
different sale units, as this can not be specified in the API calls.

### For how long is an initiated payment reserved?

That depends.
The details may change, but the information below is the best Vipps can offer.

Vipps does not control the behaviour of the customer's card or account.

VISA reservations are valid for 7 days (but only 5 for Visa Electron).
The banks will release the reservation after 4-7 days, but if the capture is
done within the 7 days, VISA guarantees that the capture will succeed.
Vipps' PSP is Adyen, and they have some documentation for
[VISA reservations](https://docs.adyen.com/online-payments/adjust-authorisation#visa).

MasterCard reservations are valid for 30 days.
The banks may release the reservation before this, but if the capture is
done within the 30 days, MasterCard guarantees that the capture will succeed.
Vipps' PSP is Adyen, and they have some documentation for
[Mastercard reservations](https://docs.adyen.com/online-payments/adjust-authorisation#mastercard).

Vipps cannot and does not automatically change the status of a reservation.

If a capture attempt is made more than 7 days (VISA) or 30 days (MasterCard)
after the payment has been initiated _and_ the reservation has been released
by the bank in the meantime, Vipps will make a new payment request to the bank.
If the account has sufficient funds, the payment will be successful.

If the user's account has insufficient funds at this time, the payment will
either succeed and put the customer's card/account in the negative (as
an overdraft), or fail because the customer's card/account can not be put into
the negative - for example youth accounts.

It is also possible that the card expires, is blocked, etc somewhere between
the time of reservation and the time of capture.
Vipps can not know in advance what will happen.

In many cases the bank will have a register of expired reservations and they
will force the capture through if the account allows this.
This will put the account in the negative.

Customers may, understandably, be dissatisfied if the capture puts their account
in the negative, so please try to avoid this.

The
[`POST:/ecomm/v2/payments/{orderId}/capture`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/capturePaymentUsingPOST)
and
[`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getPaymentDetailsUsingGET)
API calls will always return the correct status.

See
[How can I refund only a part of a payment?](#how-can-i-refund-only-a-part-of-a-payment).

### Can I prevent people from paying with credit cards?

Yes, but only if you are not legally allowed to accept credit card payments.

Sale units can be configured to only accept payments from debit cards, so
customers can not pay with credit cards. This is not configurable by the
merchant. Please contact your KAM or
[Vipps Kundesenter](https://vipps.no/kontakt-oss/bedrift/vipps/)
if you need this.

### Can I initiate a Vipps payment with a QR code?

Not with the eCom API.
With the eCom API payments are initiated by calling
[`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/initiatePaymentV3UsingPOST),
with a unique `orderId` for each payment. This is not possible with a static
QR code on a sticker, etc.

The only ways to initiate Vipps payments from a QR code are:
* [Vippnummer](https://vipps.no/produkter-og-tjenester/bedrift/ta-betalt-i-butikk/),
  the solution for flea markets, etc – which does not have any external API.
  This can not be used for online sales, etc.
* [Vipps i terminal](https://www.vipps.no/produkter-og-tjenester/privat/vipps-i-terminal/vipps-i-terminal/):
  Merchant-presented, dynamic QR shown on the display of a payment terminal.

It is possible to whitelist an URL and let users scan a QR with Vipps and be
sent to the merchant's website. See:
[Can I whitelist my URL for a Vipps QR?](#can-i-whitelist-my-url-for-a-vipps-qr).

### Can I send a Vipps payment link in an SMS, QR or email?

No. The Vipps "deeplink" is an integrated part of the Vipps payment process,
and the link should never be sent in an SMS or email. The deeplink is only valid
for 5 minutes, so users that do not act quickly will not be able to pay.

According to Norwegian regulations the customer needs to actively accept the
terms and conditions for the purchase.
For more information, please see the Consumer Authority's
[Guidelines for the standard sales conditions for consumer purchases of goods over the internet](https://www.forbrukertilsynet.no/english/guidelines/guidelines-the-standard-sales-conditions-consumer-purchases-of-goods-the-internet).

Instead of sending a Vipps deeplink: Send a link to your website, and let
the user start the Vipps payment there. It can be a very simple page with a link
or a button. You then have the opportunity to give the user additional
information, and also a proper confirmation page after the payment has been completed.

In some cases, such as for donations and gifts, it may be acceptable to automatically
trigger the Vipps payment when the user enters your website. This requires that the
payment process is user initiated, and that there are no relevant terms and conditions
or that the user has accepted any terms and conditions at an earlier stage.

In general we advice caution and point out that it is the responsibility of the
merchant to assure that users accept terms and conditions for products and services.

You can also use
[Vipps Logg Inn](https://github.com/vippsas/vipps-login-api)
for easy registration and login.

See:
[The Vipps deeplink URL](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#the-vipps-deeplink-url).

### Can I whitelist my URL for a Vipps QR?

It is technically possible for a merchant to whitelist a URL so a user can scan
a Vipps-branded QR in Vipps and be sent directly to the merchant's URL.

This functionality is in an experimental phase, and is only available for
merchants with a Vipps KAM (Key Account Manager). To request whitelisting:
Please contact your KAM.

If you do not have a KAM, you will have to wait so see if this functionality is
made available for all Vipps merchants.

When using a merchant-specific, whitelisted URL in a Vipps QR code, we recommend
displaying a "landing page" to the user, and _not_ redirecting the user
directly to a Vipps payment. This will make the solution more robust, in case
there is a problem, and the user can retry without scanning the QR code again.

See the
[design guidelines](https://github.com/vippsas/vipps-design-guidelines#vipps-custom-qr-code)
for more details about the QR format and design.

## Refunds

### How can I refund a payment?

This depends on your eCommerce solution. The Vipps API supports refunds with
[`POST:/ecomm/v2/payments/{orderId}/refund`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/refundPaymentUsingPOST).
For details on how to offer refunds, please refer to the documentation for your eCommerce solution.

All integrations with the Vipps eCom API _must_  support refunds. See the
[Vipps eCommerce API Checklist](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api-checklist.md).

Refunds can be made up to 365 days after capture.

### How can I refund only a part of a payment?

Example: A customer has placed an order of of two items for a total of 1000 NOK.
The merchant has initiated a payment of 1000 NOK, but the customer has changed
her mind and only bought one of the items, with a price of 750 NOK. The merchant
has therefore made a
[partial capture](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#partial-capture)
of 750 NOK, and need to refund the remaining 250 NOK.

The short version: This is done automatically by the bank after a few days.
See
[For how long is an initiated payment reserved?](#for-how-long-is-an-initiated-payment-reserved).

The long version: It's not possible to cancel the remaining reservation after a
partial capture through Vipps, because our payment processor does not have this
functionality. The partial capture (the 750 of the 1000 NOK in the example above)
is normally confirmed in the bank after 3-10 days, but it sometimes takes even
longer. When this is done, the bank will make the remaining (250 NOK) available
in the customer's account again. This process depends entirely on the customer's
bank, and Vipps can not speed it up.

Banks keep reservations for the same number of days regardless of whether there
has been one or more captures. Banks do not extend the reservation if a partial
capture has been made.

If a partial capture has been made, the bank cancels the reservation for the
remaining amount. If no capture has been made, the entire reserved amount is
cancelled. Banks "count the days" from when the reservation was made, so the
merchant must make the capture, or all captures, before the reservation expires.

See: [Settlements](https://github.com/vippsas/vipps-developers/tree/master/settlements).

### Is it possible for a merchant to pay a Vipps user?

No, Vipps has no functionality for paying to a Vipps user,
except for refunding (part of) a payment.

Vipps only has APIs for paying from a person to a merchant.

It is not possible to pay from one merchant to another merchant,
or to pay from a merchant to a person.

There are several reasons for this, including:

* The Norwegian "Straksbetaling" (instant payments) system is not designed
  for this, and not all banks support it.
* There are other account-to-account payment methods but all have their
  idiosyncrasies, and none are a perfect fit.
* Payouts to cards is different than accounts, and will depend on the PSPs,
  which brings another set of challenges.
* Some merchant accounts requires "four eyes" before making payments from them,
  and Vipps does not have this functionality in the API.
* The SCA (Secure Customer Authentication) required by PSD2 further complicates
  payouts, both with an API and on [portal.vipps.no](https://portal.vipps.no).

## Users and payments

### Is there an API for retrieving information about a Vipps user?

Yes. Vipps now offers the possibility for merchants to as part of the payment flow in the
[Vipps eCom API: Userinfo](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#userinfo)
and
[Vipps Recurring API: Userinfo](https://github.com/vippsas/vipps-recurring-api/blob/master/vipps-recurring-api.md#userinfo).

This is done by adding a `scope` parameter to the initiate calls:
[`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/initiatePaymentV3UsingPOST) (eCom)
and
[`POST:/recurring/v2/agreements`](https://vippsas.github.io/vipps-recurring-api/#/Agreement%20Controller/draftAgreement) (Recurring):

- address
- birthDate
- email
- name
- phoneNumber
- nin (fødselsnummer)
- accountNumbers

**Please note:** Vipps users have not consented to Vipps providing any
information to third parties, and Vipps does not allow it. There is no
other API to look up a user's address, retrieve a user's purchases, etc.

### Is there an API for retrieving information about a merchant's payments?

Not for aggregated data.
There is an API to retrieve all details for a specific `orderId`:
[`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getPaymentDetailsUsingGET).

And there is
[Settlements](https://github.com/vippsas/vipps-developers/tree/master/settlements)
with information about settlement reports in various formats.

### Is it possible to skip the landing page?

Only in special cases, where displaying it is not possible.

This feature has to be specially enabled by Vipps for eligible sale units:
The sale units must be whitelisted by Vipps.
Skipping the landing page is typically used at physical points of sale,
where there is no display available.

You can log in on
[portal.vipps.no](https://portal.vipps.no)
to check if you have `skipLandingPage enabled.`

If you need to skip the landing page in a Point of Sale (POS) solution, see:
[What is the process to go live in production?](#what-is-the-process-to-go-live-in-production).

If you need to skip the landing page for a different reason:
Contact your Key Account Manager. If you do not have a KAM:
Please log in on
[portal.vipps.no](https://portal.vipps.no),
find the right sale unit and click the email link under the "i" information bubble.
Include a detailed description of why it is not possible to display the landing page.

See:
[Skip landing page](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#skip-landing-page)
in the API guide.

### Can I split payments to charge a fee?

Vipps does not support splitting payments to charge a fee.

If you want to charge a fee (like 3 %) of your payments, you can:

1. Receive the full payment, take your 3 %, and then pay the remaining
   97 % to your customer (merchant). In order to receive payments in this way,
   you may need the regulatory approval as
   [e-pengeforetak](https://www.finanstilsynet.no/konsesjon/e-pengeforetak/)
   from Finanstilsynet.
2. Have your customer (merchant) receive the full payment directly, then send an
   invoice for your 3 % fee.

Companies that receive payments through Vipps needs to be Vipps customers.
See:
[What are the requirements for Vipps merchants?](#what-are-the-requirements-for-vipps-merchants)

### Can I create a marketplace with multiple merchants?

We sometimes get questions whether Vipps can support a marketplace or a
shopping center, with multiple merchants. The answer is: That depends.
It may not be as straight-forward as expected, and some typical questions
are covered elsewhere in this FAQ:

* All payments with Vipps must be to a merchant that is a customer of Vipps. See
  [What are the requirements for Vipps merchants?](#what-are-the-requirements-for-vipps-merchants)
* Revenue share between the marketplace and the merchants: See:
  [Can I split payments to charge a fee?](#can-i-split-payments-to-charge-a-fee)
* Refunds can only be made from the merchant that received the payment. See:
  [Is it possible for a merchant to pay a Vipps user?](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api-faq.md#is-it-possible-for-a-merchant-to-pay-a-vipps-user)

So, in short, there are two alternatives:
1. The shopping center is the only Vipps merchant, and all payments from Vipps
   users is to this merchant. Vipps is not involved in the cooperation between
   the shopping center and it's merchants, and it is completely up to them
   to operate according to Norwegian laws and regulation.
2. Each merchant in the shopping center is a Vipps merchant, and each payment
   from a Vipps user is made directly to the merchant. This means that a
   common shopping cart for all merchants can not be paid in one operation,
   since: All payments with Vipps must be to a merchant that is a customer of Vipps.

### Can I create a service to match buyers and sellers?

Companies that receive payments through Vipps must be Vipps customers,
and this is defined in the regulatory approval for Vipps from Finanstilsynet.

If the service receives payment from a buyer and then pays the seller,
so that the service "holds" the money even for a short time, the service
will need the regulatory approval as
[e-pengeforetak](https://www.finanstilsynet.no/konsesjon/e-pengeforetak/).

If the service sells access, in the form of a subscription or per-use, the
service is _most likely_ a regular Vipps eCom customer, and can use
the
[Vipps eCom API](https://github.com/vippsas/vipps-ecom-api)
or one of our
[plugins](https://github.com/vippsas/vipps-developers#plugins).

Vipps can not offer legal advice for this.

### I have initiated an order but I can't find it!

Have you, or the eCommerce solution you are using, successfully implemented
[`GET:/ecomm/v2/payments/{orderId}/details`](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#get-payment-details)?
This is a requirement, see the
[API checklist](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api-checklist.md).

In case the Vipps
[callback](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#1-callback)
fails, you will not automatically receive notification of order status.
The solution is to check `/details`.

You can use
[Postman](https://github.com/vippsas/vipps-developers/blob/master/postman-guide.md)
to manually do API calls, Use the "inspect" functionality to see the complete requests and responses.

See:
[API endpoint](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#api-endpoints)
for an overview.

### How long is an initiated order valid, if the user does not confirm in the Vipps app?

Vipps orders have a max timeout of 10 minutes.
It's important that the merchant waits at least as long, otherwise the Vipps user may
confirm in the Vipps app, and right after get an error from the merchant that the order has been cancelled.

See: [Timeouts](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#timeouts).

### How long does it take until the money is in my account?

The settlement flow depends on the banks and is as follows:

1. Day 1: A customer makes a purchase and the transaction is completed. If the purchased product is shipped later, the "day 1" is the day the product is shipped and the customer's account is charged.
2. Day 2: Settlement files are distributed, and are available in the Vipps portal: https://portal.vipps.no.
3. Day 3 (the next _bank day_) at 16:00: Payments are made from Vipps.
4. Day 5 (the third _bank day_): The settlement is booked with reference by the bank.

A _bank day_ is a day when the bank is open for business. That excludes weekends, public holidays, etc.

See: [Settlements](https://github.com/vippsas/vipps-developers/tree/master/settlements).

### How long does it take from a refund is made until the money is in the customer's account?

Normally 2-3 _bank days_, depending on the bank.

Vipps does not have more information than what is available through our API:
[`GET:/ecomm/v2/payments/{orderId}/details`](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#get-payment-details).

See: [Settlements](https://github.com/vippsas/vipps-developers/tree/master/settlements).

### In which sequence are callbacks and fallbacks done?

Vipps can not guarantee a particular sequence, as this depends on user
actions, network connectivity/speed, etc. Because of this, it is not
possible to base an integration on a specific sequence of events.

See:
[Initiate payment flow: Phone and browser](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#initiate-payment-flow-phone-and-browser)

### Where can I find reports on transactions?

The [Vipps portal](https://portal.vipps.no/login/) provides information about
your transactions, sale units and settlement reports.
You can also subscribe to daily or monthly transaction reports by email.

See: [Settlements](https://github.com/vippsas/vipps-developers/tree/master/settlements).

## Problems for end users

### Why don't I receive the payment notification?

Push notifications must be active for Vipps to send payment notifications.

Push notifications are "best effort", and Vipps can not guarantee that all
push notifications arrive. It depends on networks, etc that Vipps can not
control.

If Vipps is already open and active when the push notification is received,
the user must press the "Send" button and move to the payments screen to see
the payment notification. Vipps is not able to poll or discover the
payment notification automatically.

### Why am I not sent back to where I came from when I have paid?

If the payment started in a custom browser (like Chrome on iOS, or an embedded
browser in Instagram, instead of the default Safari browser), the `fallback` URL
(the result page) will still be opened in the default browser.

See: [How can I open the fallback URL in a specific (embedded) browser?](#how-can-i-open-the-fallback-url-in-a-specific-embedded-browser).

### Why can't I scan the Vipps QR on the terminal with the camera app?

Vipps QR codes on the payment terminal display must be scanned with Vipps.
Scanning with a camera app will not work, as the QR code does not contain a
URL that the phone understands.

The QR code contents is a
[EMV QR code](https://www.emvco.com/emv-technologies/qrcodes/),
and this format is understood by Vipps, but not the camera app.

Scanning with the camera app may lead to a Google search for a long sequence
of digits and letters.

## Common errors

### Why do I not get callbacks from Vipps?

See the
[Callback](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#callback-endpoints)
section in the API guide.

Please make sure the URLs you provide to Vipps are reachable from outside your
own environment. Have a look at the API guide, manually "build" the callback
URL in the same way as Vipps does, and confirm that you are getting a
`HTTP 200 OK` response.

If your `callbackPrefix` is `https://example.com/vipps/callback` and your
`orderId` is `acme-shop-123-order123abc`, Vipps will add `/v2/payments/acme-shop-123-order123abc` at the
end of your `callbackPrefix` and make a callback to
`https://example.com/vipps/callback/v2/payments/acme-shop-123-order123abc`.

If you do not receive a callback, it could be because your firewall is blocking
our requests. See:
[Vipps request servers](https://github.com/vippsas/vipps-developers/blob/master/README.md#vipps-request-servers).

If you need help solving a callback-related problem, please send us a
complete HTTP request, and any other related details, so we can investigate.

See:
[Callbacks](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#callbacks).

### Why do I get `Access denied due to invalid subscription key`?

If you get a `HTTP 401 Unauthorized` response with the error message
`Access denied due to invalid subscription key.
Make sure to provide a valid key for an active subscription.`
you need to check that you have the correct API keys.
See:
[Getting the API keys](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md#getting-the-api-keys).

You also need to make sure you have access to the right API.
See:
[API products](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md#api-products).

And: Make sure you are using the right environment. The
[test environment](https://github.com/vippsas/vipps-developers/blob/master/vipps-test-environment.md)
is completely separate from the production environment, and both the MSN and
the API keys are different.

### I am getting `401 Unauthorized` error - and I have double checked all my keys!

`HTTP 401 Unauthorized` occurs when there is a mismatch between the subscription keys and the
merchant sales unit. Please follow these steps to make sure everything is correct:

1. Check the Swagger specification for the correct spelling of all the header parameters.
   They are case sensitive: `Authorization: Bearer <snip>` is not the same as `Authorization: bearer <snip>`.
2. Check that you are using the same subscription key for both the access token and payment requests
3. Check that you are using the correct API keys for the environment.
   The test and production environments are separate and require different API credentials.
4. Check both the HTTP response header and the response body from our API.
   For most errors the body contains an explanation of what went wrong.

You can use
[Postman](https://github.com/vippsas/vipps-developers/blob/master/postman-guide.md)
to manually do API calls, Use the "inspect" functionality to see the complete requests and responses.

See: [API endpoints](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#api-endpoints)
for an overview.

You can also log in to
[portal.vipps.no](https://portal.vipps.no)
to double check your API keys, sale units and API products.

Please note that Vipps can not help with the debugging of your code,
we can only help with the API requests and response.

### Why do I get `Requested Order not found`?

This is typically because the payment was initiated using the API keys for
one sale unit (MSN), and you are attempting to get the details with
[`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getPaymentDetailsUsingGET)
using the API keys for a different sale unit (MSN).

### Why do I get `429 Too Many Requests`?

We rate-limit some API endpoints to prevent incorrect usage.
The rate-limiting has nothing to do with Vipps' total capacity, but is
designed to stop obviously incorrect use.
See
[Rate limiting](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#rate-limiting)
for details.

### Why do I get `500 Internal Server Error` (or similar)?

Something _might_ be wrong on our side and we are working to fix it!

It _might_ also be a problem with your request, and that our validation does not catch it.

Please make sure the JSON payload in your API request validates.
That is the most common source of this type of error.

In other words: We should perhaps have returned `HTTP 400 Bad Request`.
You can use
[Postman](https://github.com/vippsas/vipps-developers/blob/master/postman-guide.md)
to manually do API calls, just to be sure.

See: [API endpoint](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#api-endpoints) for an overview.

Please check the HTTP response body from our API (not just the HTTP status).
For most errors the body contains an explanation of what went wrong.

See:
[Statuspage](https://github.com/vippsas/vipps-developers#status-page).

### Why do I get `errorCode 37 "Merchant not available or deactivated or blocked"`?

This can happen if the test merchant is not being used for a long time. Please
[contact customer service](https://vipps.no/kontakt-oss/bedrift/vipps/),
and we will reactivate the merchant. We no longer automatically deactivate
test merchants.

See: [Error codes](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#error-codes).

### Why do I get `errorCode 35 "Requested Order not found"`?

This is either because you are specifying an incorrect orderId, or because you
are trying to access an orderId with the incorrect API credentials.
orderIds are not globally unique, they are only unique per MSN.
If you use one the API credentials for one MSN and an orderId for another MSN,
you will get this error.

See: [Error codes](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#error-codes).

### Why do I not get the `sub` from `/details`?

If you use the correct `scope` in the payment initiation, but don't get the
`sub` in the response for `/details`: Check that you are following the
[orderId recommendations](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#orderid-recommendations).
Very short orderIds don't work well with our database index, and may cause
an internal timeout, and we "have to" send the response without the `sub`.
We can not enforce longer orderIds due to backwards compatibility.

## Other questions

### How do I perform "testing in production"?

To do this you need a live Vipps account.

We recommend testing with 2 NOK, even though 1 NOK is the smallest possible amount.
1 NOK is not reliable, as it gets low priority in some systems.

### What do we have to do with PSD2's SCA requirements?

SCA (Strong customer authentication) is a security requirement related to PSD2,
to reduce the risk of fraud and protect customer's data.

Delegated SCA is Vipps' primary way of solving the SCA requirements. For
this solution Vipps has developing a SCA compliant solution that consists of a
two-factor authentication featuring either PIN or biometrics in addition to
device possession. In addition Vipps has implemented a Dynamic Linking according
to the requirements.

There is no need for any changes to your Vipps implementation for SCA.

### How can I use Vipps for different types of payments?

It's possible to use the Vipps eCom API for several different types of payments.

Let's say you run a bookstore. You can then use Vipps eCom API in several different ways, such as:

1. A webshop that sells physical books:
   Vipps eCom API with "reserve capture", since you can not capture the payment before the book is shipped.
2. A webshop that sells digital, downloadable books that are immediately available:
   Vipps eCom API with "direct capture".
3. A physical store where customers buy physical books in person:
   Vipps eCom API with "direct capture", possibly integrated with the POS.
4. A physical store where customers can buy physical books by scanning a
   QR code in the window, and have the physical book delivered by mail:
   Vipps eCom API with "reserve capture", since you can not capture the payment before the book is shipped.

The regulatory requirements are different for different types of purchases.
One major difference is if the cardholder is physically present and
"can look the seller in the eye" while making the payment.

Vipps needs to do more thorough "Know Your Customer" (KYC) and compliance checks
for some of the examples above. This must be done per sale unit.
Vipps is also required to have the correct MCC
([Merchant Category Code](https://en.wikipedia.org/wiki/Merchant_category_code))
for each sale unit.

Because of this, merchants must use separate sale units for separate types
of purchases. This also has some benefits:

- Each sale unit has its own name presented to the user in Vipps
- Each sale unit has separate transaction logs
- Each sale unit can have its own settlement account. Sharing a single account across multiple sale units is available on request.

### How do I set up multiple sale units?

This is typically needed for organization numbers with multiple stores,
or offers different ways to pay with Vipps
(See [How can I use Vipps for different types of payments?](#how-can-i-use-vipps-for-different-types-of-payments)).

The bank account number for a sale unit must belong to the organization number
of the company that has the customer relationship with Vipps.

A legal entity, called "merchant" from now on, may have one or more sale units.
It is possible for one merchant to have multiple sale units with a separate
bank account number for each one, as long as the bank accounts belong to the
organization number that the sale unit belongs to.

If the organization has the required financial regulatory approval to "split"
payments between sale units, it is possible to have only one sale unit and
identify the payments of a store using the `orderId` - for instance by prefixing the
`orderId` with the store's id, name or number.

Alternatively each store, if they each have their own organization number,
are set up with their own merchant and sale units.

If all sale units have the same organization number, there are two alternatives:

1: Recommended: Multiple sale units: One sale unit per store. Each sale unit will have its
   own MSN (Merchant Serial Number), and the `orderId` may be whatever you want.
   You will need separate API keys for each sale unit (store).
   See [How can I use Vipps for different types of payments?](#how-can-i-use-vipps-for-different-types-of-payments).

2: Use only one sale unit for all stores, and use the `orderId` to identify
   which orders belong to which sale units.
   You decide what the `orderId` contains, and it may be up to 50 characters. See:
   [orderId recommendation](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#orderid-recommendations).
   You will use the same API keys for all stores.

### What about webhooks?

Vipps has, so far (and this _may_ change), used `GET` methods for retrieving information.
We have varying success when depending on systems on the merchant side, especially
during peak traffic periods. With webhooks, and also callbacks, Vipps
takes the responsibility of providing information to the merchant, while being
dependent on systems on the merchant side, network stability, etc.

In our experience, `GET` methods is the safest way for merchants to get
information from Vipps. We also provide callbacks, but merchants _must not_
rely on this alone - being able to actively retrieving information with `GET` methods
is a requirement.

See the:
[Vipps eCom API checklist](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api-checklist.md)
and
[Vipps PSP API checklist](https://github.com/vippsas/vipps-psp-api/blob/master/vipps-psp-api-checklist.md)
for examples.

### Can I use Vipps with Klarna Checkout?

Klarna Checkout supports Vipps as an External Payment Method if you have agreement with Klarna for this.
This requires a full integration with Vipps eCom APIs. Also follow Klarnas process to get the External Payment Method activated for your account, described in the [Klarna documentation](https://developers.klarna.com/documentation/klarna-checkout/in-depth/external-payment-methods/).
Using this method will add Vipps as an payment alternative inside the Klarna Checkout (KCO). It is technically possible to also use Vipps payment options outside KCO (e.g. on product pages, in basket or similar) using [Vipps Express Checkout](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#express-checkout-payments).

## Frequently Asked Questions for POS integrations

We will improve this section as we learn more. Please suggest improvements
in [Questions](#questions) below.

### What is the process to go live in production?

1. The partner establishes a customer relationship with Vipps.
   [Apply here](https://www.vipps.no/produkter-og-tjenester/bedrift/ta-betalt-i-butikk/vipps-i-kassa/).
2. The partner integrates the POS with Vipps and completes
   [the integration checklist](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api-checklist.md).
   The partner now has a working POS integration.
   This process normally takes 1-4 days.
3. The partner's merchant establishes a customer relationship with Vipps.
   [Apply here](https://www.vipps.no/produkter-og-tjenester/bedrift/ta-betalt-i-butikk/vipps-i-kassa/).
   This process normally takes 1-4 days.
4. If the merchant already has a customer relationship with Vipps, a new sales
   unit must be created, with `skipLandingPage` activated.
   The
   [Vipps Kundesenter](https://vipps.no/hjelp/vipps/)
   can help with this.
   See the [FAQ](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api-faq.md#is-it-possible-to-skip-the-landing-page).
5. The merchant
   [retrieves the API keys](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md#getting-the-api-keys)
   and provides them to the partner.
6. The partner configures the merchant's POS for Vipps.
7. The merchant can now accept Vipps payments in the POS.

### How can we be whitelisted for `skipLandingPage`?

See: [Is it possible to skip the landing page?](#is-it-possible-to-skip-the-landing-page)

### Which API keys should I use?

You need to use the merchant's API keys when using the Vipps eCom API.
The API calls are normal eCom API calls - see the
[API guide](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md).

You can _not_ use your "supermerchant" API keys (if you have those).

### Do we need to support callbacks?

If it is not possible for your POS to support callbacks (no fixed hostname/IP, etc),
you must actively check the payment status with
[``GET:/ecomm/v2/payments/{orderId}/details``](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#get-payment-details).
This is also required if you do support callbacks.

### How can I check if a person has Vipps?

There is no separate API for this, but an attempt to
[`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/initiatePaymentV3UsingPOST)
with a phone number that is not registered with Vipps will fail with error 81,
`User not registered with Vipps`.
See [Error codes](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#error-codes).

### How can I save the customer's phone number?

Vipps does not send the customer's phone number to the merchant. When a customer
enters the phone number on the Vipps landing page, that is only used by Vipps
to send a push alert in Vipps. The number is not passed on to the merchant.

If the POS integration is implemented so that the customer's phone number
is entered in the POS, the merchant can of course save it -
complying with GDPR, etc.

See
[Is there an API for retrieving information about a Vipps user?](#is-there-an-api-for-retrieving-information-about-a-vipps-user).

### How can we mass sign up merchants?

You can use the
[Signup API](https://github.com/vippsas/vipps-signup-api),
but all merchants must sign their Vipps application with BankID.
This is a legal requirement.

Merchants can of course also
[sign up themselves](https://www.vipps.no/produkter-og-tjenester/bedrift/ta-betalt-i-butikk/vipps-i-kassa/).

### Where can I find information about settlements?

Here: [Settlements](https://github.com/vippsas/vipps-developers/tree/master/settlements).

# Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-recurring-api/issues),
a [pull request](https://github.com/vippsas/vipps-recurring-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).

Sign up for our [Technical newsletter for developers](https://github.com/vippsas/vipps-developers/tree/master/newsletters).
