# Vipps eCommerce API: Frequently Asked Questions

See the
[Vipps eCommerce API](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md)
for all the details.

See also:
[Vipps Recurring API FAQ](https://github.com/vippsas/vipps-recurring-api/blob/master/vipps-recurring-api-faq.md).

See also:
[Getting Started](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md)
guide.

Document version 1.8.2.

## Table of contents

- [What are the requirements for Vipps merchants?](#what-are-the-requirements-for-vipps-merchants)
- [Can I use my "Vipps-nummer" in my webshop?](#can-i-use-my-vipps-nummer-in-my-webshop)
- [Why do payments fail?](#why-do-payments-fail)
- [Why does capture fail?](#why-does-capture-fail)
- [Why does Vipps Hurtigkasse (express checkout) fail?](#why-does-vipps-hurtigkasse-express-checkout-fail)
- [What is the difference between "Reserve Capture" and "Direct Capture"?](#what-is-the-difference-between-reserve-capture-and-direct-capture)
- [How do I turn _direct capture_ on or off?](#How-do-I-turn-direct-capture-on-or-off)
- [Can I send a Vipps payment link in an SMS or email?](#can-i-send-a-vipps-payment-link-in-an-sms-or-email)
- [Is it possible to skip the landing page?](#is-it-possible-to-skip-the-landing-page)
- [How can I refund a payment?](#how-can-i-refund-a-payment)
- [How can I refund only a part of a payment?](#how-can-i-refund-only-a-part-of-a-payment)
- [Is it possible for a merchant to pay a Vipps user?](#is-it-possible-for-a-merchant-to-pay-a-vipps-user)
- [Is there an API for retrieving information about a Vipps user?](#is-there-an-api-for-retrieving-information-about-a-vipps-user)
- [Is there an API for retrieving information about a merchant's payments?](#is-there-an-api-for-retrieving-information-about-a-merchants-payments)
- [Can I create a service to match buyers and sellers?](#can-i-create-a-service-to-match-buyers-and-sellers)
- [Can I split payments to charge a fee?](#can-i-split-payments-to-charge-a-fee)
- [I have initiated an order but I can't find it!](#i-have-initiated-an-order-but-i-cant-find-it)
- [How long is an initiated order valid, if the user does not confirm in the Vipps app?](#how-long-is-an-initiated-order-valid-if-the-user-does-not-confirm-in-the-vipps-app)
- [How long does it take until the money is in my account?](#how-long-does-it-take-until-the-money-is-in-my-account)
- [How long does it take from a refund is made until the money is in the customer's account?](#how-long-does-it-take-from-a-refund-is-made-until-the-money-is-in-the-customers-account)
- [Where can I find reports on transactions?](#where-can-i-find-reports-on-transactions)
- [For how long is an initiated payment reserved?](#for-how-long-is-an-initiated-payment-reserved)
- [I am getting `401 Unauthorized` error - and I have double checked all my keys!](#i-am-getting-401-unauthorized-error---and-i-have-double-checked-all-my-keys)
- [Why do I get `500 Internal Server Error` (or similar)?](#why-do-i-get-500-internal-server-error-or-similar)
- [In which sequence are callbacks and fallbacks done?](#in-which-sequence-are-callbacks-and-fallbacks-done)
- [Why do I not get callbacks from Vipps?](#why-do-i-not-get-callbacks-from-vipps)
- [Why do I get `errorCode 37 "Merchant not available or deactivated or blocked"`](#why-do-i-get-errorcode-37-merchant-not-available-or-deactivated-or-blocked)
- [How do I perform "testing in production"?](#how-do-i-perform-testing-in-production)
- [What do we have to do with PSD2's SCA requirements?](#what-do-we-have-to-do-with-psd2s-sca-requirements)
- [What about webhooks?](#what-about-webhooks)
- [How do I set up multiple sale units?](#how-do-i-set-up-multiple-sale-units)
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

## What are the requirements for Vipps merchants?

Vipps merchants (corporate customers) must have a Norwegian organization number
and applications must be signed with Norwegian BankID. Vipps must follow the
regulatory requirements for KYC (Know Your Customer), AML (Anti Money Laundering)
and other risk assessment procedures.

## Can I use my "Vipps-nummer" in my webshop?

No. You need [Vipps på Nett](https://www.vipps.no/produkter-og-tjenester/bedrift/ta-betalt-paa-nett/ta-betalt-paa-nett/).

## Why do payments fail?

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

## Why does capture fail?

If the reserved amount is too low for shipping costs to be included, the capture will fail.
The reserved amount must at least as high as the amount that is captured.

Example: If the value of the shopping cart is 1000 NOK, and the reserved amount is 1100 NOK,
the shipping cost must be maximum 100 NOK. If the shipping cost is 150 kr, a capture of
1000 + 150 kr = 1150 NOK will fail.

## Why does Vipps Hurtigkasse (express checkout) fail?

When using Vipps Hurtigkasse (express checkout), Vipps makes a
[callback](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#1-callback)
to the customer's server. If this server is slow,
or has a slow internet connection, the delay may cause Vipps Hurtigkasse to fail due to a timeout.
The solution to this is a faster server and internet connection.

Information for [Vipps for WooCommerce](https://www.vipps.no/produkter-og-tjenester/bedrift/ta-betalt-paa-nett/ta-betalt-paa-nett/woocommerce/):
Some third party plugins do not work with Vipps Hurtigkasse. Please ask for help in the
[support forum](https://wordpress.org/support/plugin/woo-vipps),
and include information about the plugins you have installed.

## What is the difference between "Reserve Capture" and "Direct Capture"?

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

You can't turn _direct capture_ on or off as a merchant. This must be
requested of your Key Account Manager. If you do not have a KAM:
Please log in on
[portal.vipps.no](https://portal.vipps.no),
find the right sale unit and click the email link under the "i" information bubble.

See [Regular eCommerce payments](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#regular-ecommerce-payments) for more details.

## How do I turn direct capture on or off?

You can't turn _direct capture_ on or off as a merchant. This must be
requested of your Key Account Manager. If you do not have a KAM:
Please log in on
[portal.vipps.no](https://portal.vipps.no),
find the right sale unit and click the email link under the "i" information bubble.

To get both _direct capture_ and _reserve capture_ you must request two
different sale units, as this can not be specified in the API calls.

## Can I send a Vipps payment link in an SMS or email?

No. The Vipps "deeplink" is an integrated part of the Vipps payment process,
and the link should never be sent in an SMS or email. The deeplink is only valid
for 5 minutes, so users that do not act quickly will not be able to pay.

Instead of sending a Vipps deeplink: Send a link to your website, and let
the user start the Vipps payment there. It can be a very simple page with a link
or a button. You then have the opportunity to give the user additional
information, and also a proper confirmation page after the payment has been completed.

You can also use
[Vipps Logg Inn](https://github.com/vippsas/vipps-login-api)
for easy registration and login.

See the API Guide:
[The Vipps deeplink URL](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#the-vipps-deeplink-url).

## Is it possible to skip the landing page?

Skipping the landing page is reserved for special cases, where displaying it is not possible.

This feature has to be specially enabled by Vipps for eligible sale units:
The sale units must be whitelisted by Vipps.
Skipping the landing page is typically used at physical points of sale,
where there is no display available.

See the details in the
[skip landing page section](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#skip-landing-page)
in the API guide.

If you need to skip the landing page in a Point of Sale (POS) solution, see:
[What is the process to go live in production?](#what-is-the-process-to-go-live-in-production).

If you need to skip the landing page for a different reason:
[contact Vipps](https://github.com/vippsas/vipps-developers/blob/master/contact.md)
with a detailed description of why it is not possible to display the landing page.

## How can I refund a payment?

This depends on your eCommerce solution. The Vipps API supports refunds with
[`POST:/ecomm/v2/payments/{orderId}/refund`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/refundPaymentUsingPOST).
For details on how to offer refunds, please refer to the documentation for your eCommerce solution.

Refunds can be made up to 365 days after capture.

## How can I refund only a part of a payment?

Example: A customer has placed an order of of two items for a total of 1000 NOK.
The merchant has initiated a payment of 1000 NOK, but the customer has changed
her mind and only bought one of the items, with a price of 750 NOK. The merchant
has therefore made a _partial capture_ of 750 NOK, and need to refund the
remaining 250 NOK.

The short version: This is done automatically by the bank after a few days.

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

See also [Settlements](https://github.com/vippsas/vipps-developers/tree/master/settlements).

## Is it possible for a merchant to pay a Vipps user?

No, Vipps has no functionality for paying to a Vipps user,
except for refunding (part of) a payment.

Vipps only has APIs for paying from a person to a merchant.

It is not possible to pay from one merchant to another merchant,
or to pay from a merchant to a person.

## Is there an API for retrieving information about a Vipps user?

No. Vipps users have not consented to Vipps providing any information to
third parties, and Vipps does not allow it. There is no API to look up
a user's address, retrieve a user's purchases, etc.

## Is there an API for retrieving information about a merchant's payments?

Not for aggregated data.
There is an API to retrieve all details for a known `orderId`:
[`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/getPaymentDetailsUsingGET).

And there is
[Settlements](https://github.com/vippsas/vipps-developers/tree/master/settlements)
with information about settlement reports in various formats.

## Can I create a service to match buyers and sellers?

Companies that receive payments through Vipps must be Vipps customers.

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

## Can I split payments to charge a fee?

Vipps does not support splitting payments to charge a fee.

If you want to charge a fee (like 3 %) of your payments, you can:

1. Receive the full payment, take your 3 %, and then pay the remaining
   97 %. In order to receive payments in this way, you may need approval
   from Finanstilsynet.
2. Have your customer receive the full payment directly, then send an
   invoice for your 3 % fee.

Companies that receive payments through Vipps needs to be Vipps customers.
See [What are the requirements for Vipps merchants?](#what-are-the-requirements-for-vipps-merchants)

## I have initiated an order but I can't find it!

Have you, or the ecommerce solution you are using, successfully implemented
[``GET:/ecomm/v2/payments/{orderId}/details``](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#get-payment-details)? This is a requirement, see the [API checklist](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api-checklist.md).

In case the Vipps
[callback](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#1-callback)
fails, you will not automatically receive notification of order status.
The backup is to check `/details`.

You can use [Postman](https://github.com/vippsas/vipps-developers/blob/master/postman-guide.md)
to manually do API calls, like the two above.
See [API endpoint](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#api-endpoints) for an overview.

## How long is an initiated order valid, if the user does not confirm in the Vipps app?

Vipps orders have a max timeout of 10 minutes.
It's important that the merchant waits at least as long, otherwise the Vipps user may
confirm in the Vipps app, and right after get an error from the merchant that the order has been cancelled.

See also: [Timeouts](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#timeouts).

## How long does it take until the money is in my account?

The settlement flow is as follows:

1. Day 1: A customer makes a purchase and the transaction is completed. If the purchased product is shipped later, the "day 1" is the day the product is shipped and the customer's account is charged.
2. Day 2: Settlement files are distributed, and are available in the Vipps portal: https://portal.vipps.no.
3. Day 3 (the next _bank day_) at 16:00: Payments are made from Vipps.
4. Day 5 (the third _bank day_): The settlement is booked with reference by the bank.

See also [Settlements](https://github.com/vippsas/vipps-developers/tree/master/settlements).

## How long does it take from a refund is made until the money is in the customer's account?

Normally 2-3 _bank days_, depending on the bank.

See also [Settlements](https://github.com/vippsas/vipps-developers/tree/master/settlements).

## Where can I find reports on transactions?

The [Vipps portal](https://portal.vipps.no/login/) provides information about
your transactions, sale units and settlement reports.
You can also subscribe to daily or monthly transaction reports.

See also [Settlements](https://github.com/vippsas/vipps-developers/tree/master/settlements).

## For how long is an initiated payment reserved?

Most banks keep reservations for 7 days, however this varies depending on which bank the customer is using.
Some banks only keep reservations for 4 days.
Vipps does not automatically change the status of the order.

If a capture attempt is made more than 7 days after the payment has been initiated
and the reservation has been released by the bank, Vipps will make a new payment request to the bank.
If the account has sufficient funds, the payment will be successful.
If the user's account has insufficient funds at this time, the payment will fail.

In many cases the bank will have a register of expired reservations and they will force it through if the account allows this.
This will put the account in the negative.

## I am getting `401 Unauthorized` error - and I have double checked all my keys!

`HTTP 401 Unauthorized` occurs when there is a mismatch between the subscription keys and the
merchant sales unit. Please follow these steps to make sure everything is correct:

1. Correct spelling of `Ocp-Apim-Subscription-Key` parameter in the header of `Access Token` and Payment API
2. Confirm that you are not using the same subscription key for `Access Token` and `Payment Initiation`
3. Make sure you are using the same `merchantSerialNumber` in the body of your request as is stated in the developer portal
4. Make sure you are making calls to `ecomm/v2/payments` and _not_ `ecomm/v1/payments` (unless this is specifically agreed upon)

You can use [Postman](https://github.com/vippsas/vipps-developers/blob/master/postman-guide.md)
to manually do API calls, like the two above.
See [API endpoints](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#api-endpoints) for an overview.

Please check the HTTP response from our API.
For most errors there is an explanation of what went wrong.

You can also log in to the Vipps portal to double check your API keys,
sale units and API products: https://portal.vipps.no.

## Why do I get `500 Internal Server Error` (or similar)?

Something _might_ be wrong on our side and we are working to fix it!

It _might_ also be a problem with your request, and that our validation does not catch it.
In other words: We should perhaps have returned `HTTP 400 Bad Request`.
You can use [Postman](https://github.com/vippsas/vipps-developers/blob/master/postman-guide.md)
to manually do API calls, just to be sure.
See [API endpoint](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#api-endpoints) for an overview.

Please check the HTTP response from our API.
For most errors there is an explanation of what went wrong.

## In which sequence are callbacks and fallbacks done?

Vipps can not guarantee a particular sequence, as this depends on user
actions, network connectivity/speed, etc. Because og this, it is not
possible to base an integration on a specific sequence of events.

More details:
[Initiate payment flow: Phone and browser](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#initiate-payment-flow-phone-and-browser)

## Why do I not get callbacks from Vipps?

Please make sure the URLs you provide to Vipps are reachable from outside your
own environment.

It could be because your firewall is blocking our requests.
Please see
[Vipps request servers](https://github.com/vippsas/vipps-developers/blob/master/README.md#vipps-request-servers).

If you need help solving a callback-related problem, please send us a
complete HTTP request, and any other related details, so we can investigate.

## Why do I get `errorCode 37 "Merchant not available or deactivated or blocked"`

This can happen if the test merchant is not being used for a long time. Please
[contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md),
and we will reactivate the merchant. We no longer automatically deactivate
test merchants.

## How do I perform "testing in production"?

To do this you need a live Vipps account.

We recommend testing with 2 NOK, even though 1 NOK is the smallest possible amount.
1 NOK is not reliable, as it gets low priority in some systems.

## What do we have to do with PSD2's SCA requirements?

SCA (Strong customer authentication) is a security requirement, related to PSD2,
to reduce the risk of fraud and protect customers data.

Delegated SCA will be Vipps' primary way of solving the SCA requirements. For
this solution Vipps is developing a SCA compliant solution that consists of a
two-factor authentication featuring either PIN or biometrics in addition to
device possession. In addition Vipps will implement a Dynamic Linking according
to the requirements.

There is no need for any changes to your Vipps implementation.

## What about webhooks?

Vipps has, so far (and this _may_ change), used `GET` methods for retrieving information.
We have varying success when depending on systems on the merchant side, especially
during peak traffic periods. With webhooks, and also callbacks, Vipps
takes the responsibility of providing information to the merchant, while being
dependent on systems on the merchant side, network stability, etc.

In our experience, `GET` methods is the safest way for merchants to get
information from Vipps. We also provide callbacks, but merchants _must not_
rely on this alone - being able to actively retrieving information with `GET` methods
is a requirement.

See the checklists for
[Vipps eCom API](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api-checklist.md)
and
[Vipps PSP API](https://github.com/vippsas/vipps-psp-api/blob/master/vipps-psp-api-checklist.md)
for examples.

## How do I set up multiple sale units?

This is typically needed for organization numbers with multiple stores.

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

1: Use only one sale unit for all stores., and use the `orderId` to identify which orders belong
to which sale units. You decide what the `orderId` contains, and it may be up to
30 characters. See
[orderId recommendation](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#orderid-recommendations).
You will use the same API keys for all stores.

2: Multiple sale units: One sale unit per store. Each sale unit will have it's
own MSN (Merchant Serial Number), and the `orderId` may be whatever you want.
You will need separate API keys for each sale unit (store).

# Frequently Asked Questions for POS integrations

We will improve this section as we learn more. Please suggest improvements
in [Questions](#questions) below.

## What is the process to go live in production?

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

## How can we be whitelisted for `skipLandingPage`?

See [Is it possible to skip the landing page?](#is-it-possible-to-skip-the-landing-page)

## Which API keys should I use?

You need to use the merchant's API keys when using the Vipps eCom API.
The API calls are normal eCom API calls - see the
[API guide](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md).

You can _not_ use your "supermerchant" API keys (if you have those).

## Do we need to support callbacks?

If it is not possible for your POS to support callbacks (no fixed hostname/IP, etc),
you must actively check the payment status with
[``GET:/ecomm/v2/payments/{orderId}/details``](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#get-payment-details).
This is also required if you do support callbacks.

## How can I check if a person has Vipps?

There is no separate API for this, but an attempt to
[`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/initiatePaymentV3UsingPOST)
with a phone number that is not registered with Vipps will fail with error 81,
`User not registered with Vipps`.
See [Error codes](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#error-codes).

## How can I save the customer's phone number?

Vipps does not send the customer's phone number to the merchant. When a customer
enters the phone number on the Vipps landing page, that is only used by Vipps
to send a push alert in Vipps. The number is not passed on to the merchant.

If the POS integration is implemented so that the customer's phone number
is entered in the POS, the merchant can of course save it -
complying with GDPR, etc.

## How can we mass sign up merchants?

You can use the
[Signup API](https://github.com/vippsas/vipps-signup-api),
but all merchants must sign their Vipps application with BankID.
This is a legal requirement.

Merchant can of course also
[sign up themselves](https://www.vipps.no/produkter-og-tjenester/bedrift/ta-betalt-i-butikk/vipps-i-kassa/).

## Where can I find information about settlements?

Here: [Settlements](https://github.com/vippsas/vipps-developers/tree/master/settlements).

# Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-recurring-api/issues),
a [pull request](https://github.com/vippsas/vipps-recurring-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).

Sign up for our [Technical newsletter for developers](https://github.com/vippsas/vipps-developers/tree/master/newsletters).
