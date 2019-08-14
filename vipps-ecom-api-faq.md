# Frequently Asked Questions for Vipps eCommerce API

See the [Vipps eCommerce API](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md) for all the details.

See also the
[Getting Started](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md)
guide.

# Table of contents

- [Why do payments fail?](#why-do-payments-fail)
- [Why does capture fail?](#why-does-capture-fail)
- [Can I use my "Vipps-nummer" in my webshop?](#can-i-use-my-vipps-nummer-in-my-webshop)
- [Why does Vipps Hurtigkasse (express checkout) fail?](#why-does-vipps-hurtigkasse-express-checkout-fail)
- [What is the difference between "Reserve Capture" and "Direct Capture"?](#what-is-the-difference-between-reserve-capture-and-direct-capture)
- [How do I turn _direct capture_ on or off?](#How-do-I-turn-direct-capture-on-or-off)
- [How can I refund a payment?](how-can-i-refund-a-payment)
- [How can I refund only a part of a payment?](#how-can-i-refund-only-a-part-of-a-payment)
- [Is there an API for retrieving information about a Vipps user?](#is-there-an-api-for-retrieving-information-about-a-vipps-user)
- [I have initiated an order but I can't find it!](#i-have-initiated-an-order-but-i-cant-find-it)
- [How long is an initiated order valid, if the user does not confirm in the Vipps app?](#how-long-is-an-initiated-order-valid-if-the-user-does-not-confirm-in-the-vipps-app)
- [How long does it take until the money is in my account?](#how-long-does-it-take-until-the-money-is-in-my-account)
- [How long does it take from a refund is made until the money is in the customer's account?](#how-long-does-it-take-from-a-refund-is-made-until-the-money-is-in-the-customers-account)
- [Where can I find reports on transactions?](#where-can-i-find-reports-on-transactions)
- [For how long is an initiated payment reserved?](#for-how-long-is-an-initiated-payment-reserved)
- [I am getting `401 Unauthorized` error - and I have double checked all my keys!](#i-am-getting-401-unauthorized-error---and-i-have-double-checked-all-my-keys)
- [Why do I get `500 Internal Server Error` (or similar)?](#why-do-i-get-500-internal-server-error-or-similar)
- [Why do I not get callbacks from Vipps?](#why-do-i-not-get-callbacks-from-vipps)
- [Why do I get `errorCode 37 "Merchant not available or deactivated or blocked"`](#why-do-i-get-errorcode-37-merchant-not-available-or-deactivated-or-blocked)
- [How do I perform "testing in production"?](#how-do-i-perform-testing-in-production)
- [What do we have to do with PSD2's SCA requirements?](#what-do-we-have-to-do-with-psd2s-sca-requirements)

# Why do payments fail?

The most common reasons are:
1. The debit/credit card has expired
2. The debit/credit is no longer valid (typically when a user has received a new card, but the previous card's expiry date has not yet been reached)
3. Insufficient funds on the debit/credit card (not enough money in the debit card's bank account, or not enough credit left on the credit card)
4. The debit/credit card has been rejected by the issuer
5. Payment limit reached, the user needs to authenticate with bankID in the Vipps app
6. The payment has timed out (this happens if the user does not confirm in the Vipps app within 5 minutes - typically of the user has deactivcated puch notifications)
7. Attempt to capture an amount that exceeds the reserved amount
8. Attempt to capture an amount that has not been reserved

We are continuously improving the error messages in the Vipps app. Please note that we are not allowed to give detailed information about all errors to the merchant, as some information should only be provided to the Vipps user.

# Can I use my "Vipps-nummer" in my webshop?

No. According to Norwegian law you must be able to offer refunds.
This is not supported with [Vipps-nummer](https://www.vipps.no/produkter-og-tjenester/bedrift/ta-betalt-i-butikk/ta-betalt-med-vipps/).
You need [Vipps på Nett](https://www.vipps.no/produkter-og-tjenester/bedrift/ta-betalt-paa-nett/ta-betalt-paa-nett/).

# Why does capture fail?

If the reserved amount is too low for shipping costs to be included, the capture will fail.
The reserved amount must at least as high as the amount that is captured.

Example: If the value of the shopping cart is 1000 NOK, and the reserved amount is 1100 NOK,
the shipping cost must be maximum 100 NOK. If the shipping cost is 150 kr, a capture of
1000 + 150 kr = 1150 NOK will fail.

# Why does Vipps Hurtigkasse (express checkout) fail?

When using Vipps Hurtigkasse (express checkout), Vipps makes a
[callback](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#1-callback)
to the customer's server. If this server is slow,
or has a slow internet connection, the delay may cause Vipps Hurtigkasse to fail due to a timeout.
The solution to this is a faster server and internet connection.

Information for [Vipps for WooCommerce](https://www.vipps.no/produkter-og-tjenester/bedrift/ta-betalt-paa-nett/ta-betalt-paa-nett/woocommerce/):
Some third party plugins do not work with Vipps Hurtigkasse. Please ask for help in the
[support forum](https://wordpress.org/support/plugin/woo-vipps),
and include information about the plugins you have installed.

# What is the difference between "Reserve Capture" and "Direct Capture"?

When you initiate a payment it will be reserved until you capture it.
Vipps supports both _reserve-capture_ and _direct capture_.

_Reserve capture_ is the default. When you initiate a payment it will be reserved until you capture it.

When _direct capture_ is activated, all payment reservations will instantly be captured.
This is intended for situations where the product or service is immediately provided to the customer, e.g. digital services.

According to Norwegian regulations you should _not_ capture a payment until the product or service is provided to the customer.
For more information, please see the Consumer Authority's
[Guidelines for the standard sales conditions for consumer purchases of goods over the internet](https://www.forbrukertilsynet.no/english/guidelines/guidelines-the-standard-sales-conditions-consumer-purchases-of-goods-the-internet).

To request _direct capture_, please contact your KAM.

See [Regular eCommerce payments](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#regular-ecommerce-payments) for more details.

# How do I turn _direct capture_ on or off?

You can't turn _direct capture_ on or off as a merchant, and this must be requested of your KAM. To get both _direct capture_ and _reserve capture_ you must request two different sale units.

# How can I refund a payment?

This depends on your eCommerce solution. The Vipps API supports refunds with
[`POST:/ecomm/v2/payments/{orderId}/refund`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/refundPaymentUsingPOST).
For details on how to offer refunds, please refer to the documentation for your eCommerce solution.

# How can I refund only a part of a payment?

Case: A customer has placed an order of of two items for a total of 1000 NOK. You have initiated a payment of 1000 NOK, but the customer has changed her mind and only bought one of the items, with a price of 750 NOK. You have performed a partial capture of 750 NOK, and need to refund the reamining 250 NOK.

It's not possible to cancel the remaining reservation after a partial capture through Vipps, but when the payment is confirmed
in the bank (normally 2-3 days later), the money will automatically be available to the customer.

# Is there an API for retrieving information about a Vipps user?

No. Vipps users have not consented to Vipps providing any information to
third parties, and Vipps does not allow it. There is no API to look up
a user's address, retrieve a user's purchases, etc.

# I have initiated an order but I can't find it!

Have you, or the ecommerce solution you are using, successfully implemented
[``GET:/ecomm/v2/payments/{orderId}/details``](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#get-payment-details) or
[``GET:/ecomm/v2/payments/{orderId}/status``](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#get-order-status)?

In case the Vipps
[callback](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#1-callback)
fails, you will not automatically receive notification of order status.
The backup should be to ask for status within a time frame.

You can use [Postman](https://github.com/vippsas/vipps-developers/blob/master/postman-guide.md)
to manually do API calls, like the two above.
See [API endpoint](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#api-endpoints) for an overview.

# How long is an initiated order valid, if the user does not confirm in the Vipps app?

Vipps currently waits for five minutes before automatically cancelling an order due to inactivity.
It's important that the merchant waits at least as long, otherwise the Vipps user may
confirm in the Vipps app, and right after get an error from the merchant that the order has been cancelled.

# How long does it take until the money is in my account?

The settlement flow is as follows:

1. Day 1: A customer makes a purchase and the transaction is completed. If the purchased product is shipped later, the "day 1" is the day the product is shipped and the customer's account is charged.
2. Day 2: Settlement files are distributed, and are available in the Vipps portal: https://portal.vipps.no.
3. Day 3 (the next _bank day_) at 16:00: Payments are made from Vipps.
4. Day 5 (the third _bank day_): The settlement is booked with reference by the bank.

See also [Settlements](https://github.com/vippsas/vipps-developers/tree/master/settlements).

# How long does it take from a refund is made until the money is in the customer's account?

Normally 2-3 _bank days_, depending on the bank.

# Where can I find reports on transactions?

The [Vipps portal](https://portal.vipps.no/login/) provides information about
your transactions, sale units and settlement reports.
You can also subscribe to daily or monthly transaction reports.

More information: https://github.com/vippsas/vipps-developers/tree/master/settlements

# For how long is an initiated payment reserved?

Most banks keep reservations for 7 days, however this varies depending on which bank the customer is using.
Vipps does not automatically change the status of the order.

If a capture attempt is made more than 7 days after the payment has been initiated
and the reservation has been released, Vipps will make a new payment request to the bank.
If the account has sufficient funds, the payment will be successful.
If the user's account has insufficient funds at this time, the payment will fail.

In many cases the bank will have a register of expired reservations and they will force it through if the account allows this.
This will put the account in the negative.

# I am getting `401 Unauthorized` error - and I have double checked all my keys!

`HTTP 401 Unauthorized` occurs when there is a mismatch between the subscription keys and the
merchant sales unit. Please follow these steps to make sure everything is correct:

1. Correct spelling of `Ocp-Apim-Subscription-Key` parameter in the header of `Access Token` and Payment API
2. Confirm that you are not using the same subscription key for `Access Token` and `Payment Initiation`
3. Make sure you are using the same `merchantSerialNumber` in the body of your request as is stated in the developer portal
4. Make sure you are making calls to `ecomm/v2/payments` and _not_ `ecomm/v1/payments` (unless this is specifically agreed upon)

You can use [Postman](https://github.com/vippsas/vipps-developers/blob/master/postman-guide.md)
to manually do API calls, like the two above.
See [API endpoints](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#api-endpoints) for an overview.

# Why do I get `500 Internal Server Error` (or similar)?

Something _might_ be wrong on our side and we are working to fix it!

It _might_ also be a problem with your request, and that our validation does not catch it.
In other words: We should have returned `HTTP 400 Bad Request`.
You can use [Postman](https://github.com/vippsas/vipps-developers/blob/master/postman-guide.md)
to manually do API calls, just to be sure.
See [API endpoint](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#api-endpoints) for an overview.

# Why do I not get callbacks from Vipps?

It could be because your firewall is blocking our requests.
Please see [Vipps request servers](https://github.com/vippsas/vipps-developers/blob/master/README.md#vipps-request-servers).

If you need help solving a callback-related problem, please send us a
complete HTTP request, and any other related details, so we can investigate.

# Why do I get `errorCode 37 "Merchant not available or deactivated or blocked"`

This happens if the test merchant is not being used for some time. Please
[contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md), and we will reactivate the merchant.

# How do I perform "testing in production"?

To do this you need a live Vipps account.
We recommend testing with 2 NOK, even though 1 NOK is the smallest possible amount.
1 NOK is not reliable, as it gets low priority in some systems.

# What do we have to do with PSD2's SCA requirements?

SCA (Strong customer authentication) is a security requirement, related to PSD2, to reduce the risk of fraud and protect customers data.

Delegated SCA will be Vipps' primary way of solving the SCA requirements. For this solution Vipps is developing a SCA compliant solution that consists of a two factor authentication featuring either PIN or biometrics in addition to device possession. In addition Vipps will implement a Dynamic Linking according to the requirements.

There is no need for any changes to your Vipps implementation. 

# Questions?

We're always happy to help with code or other questions you might have! Please create an [issue](https://github.com/vippsas/vipps-recurring-api/issues),
a [pull request](https://github.com/vippsas/vipps-recurring-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).
