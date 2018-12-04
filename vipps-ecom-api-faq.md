# Frequently Asked Questions for Vipps eCommerce API

See the [Vipps eCommerce API](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md) for all the details.

See also the
[Getting Started](https://github.com/vippsas/vipps-developers/blob/master/vipps-developer-portal-getting-started.md)
guide for the Vipps Developer Portal.

## Why do some Vipps payments fail?

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

## I am unable to login to the Developer Portal in either test or production.

Make sure you are trying to log in to the right portal, and that you are using a browser that
[does not have an active Microsoft login](https://github.com/vippsas/vipps-developers/blob/master/vipps-developer-portal-getting-started.md#remember-to-log-out-of-other-microsoft-accounts).

If your username is `name@testapivipps.no`, you should log into the test portal https://apitest-portal.vipps.no/.

If your username is `name@apivipps.no`, you should log in to the production portal https://api-portal.vipps.no/.

See [Step 1](https://github.com/vippsas/vipps-developers/blob/master/vipps-developer-portal-getting-started.md#step-1)
in the guide.

## I am getting `401 Unauthorized` error - and I have double checked all my keys!

`HTTP 401 Unauthorized` occurs when there is a mismatch between the subscription keys and the
merchant sales unit. Please follow these steps to make sure everything is correct:

1. Correct spelling of `Ocp-Apim-Subscription-Key` parameter in the header of `Access Token` and Payment API
2. Confirm that you are not using the same subscription key for `Access Token` and `Payment Initiation`
3. Make sure you are using the same `merchantSerialNumber` in the body of your request as is stated in the developer portal
4. Make sure you are making calls to `ecomm/v2/payments` and _not_ `ecomm/v1/payments` (unless this is specifically agreed upon)

## Everything worked yesterday and now I'm suddenly getting `500 Internal Server Error` (or similar)

Something _might_ be wrong on our side and we are working to fix it!

## I have not had time to test this month and when I came back to it now I get `errorCode 37 "Merchant not available or deactivated or blocked"`

This happens if the test merchant is not being used for some time. Please
[contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md), and we will reactivate the merchant.

## I have successfully tested my integration in the test environment and now need access to the production environment. How do I get it?

Please [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).
We will activate your production account and you will receive login credentials
to your production developer portal https://api-portal.vipps.no - If you do not receive credentials please double check
your spam filter and trash.

## How do I perform testing in production?

To do this you need a live Vipps account - we do not have test accounts available for Vipps.
We recommend testing with 2 NOK, even though 1 NOK is the smallest possible amount.
1 NOK is not reliable, as it gets low priority in some systems.

## What is the difference between "Reserve Capture" and "Direct Capture", and how do I change payment type?

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

## I have initiated a payment - how can I refund a part of the amount?

Case: A customer has placed an order of of two items for a total of 1000 NOK. You have initiated a payment of 1000 NOK, but the customer has changed her mind and only bought one of the items, with a price of 750 NOK. You have performed a partial capture of 750 NOK, and need to refund the reamining 250 NOK.

It's not possible to cancel the remaining reservation after a partial capture through Vipps, but when the payment is confirmed
in the bank (normally 2-3 days later), the money will automatically be available to the customer.

## I have initiated an order but I can't find it!

Have you successfully implemented
[`getPaymentDetails`](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#get-payment-details) or
[`getOrderStatus`](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#get-order-status)?

In case our callback fails, you will not automatically receive notification of order status. The backup should be to ask for status within a time frame.

## Where can I find reports on transactions?

The [Vipps portal](https://portal.vipps.no/login/) provides information about
your transactions, sale units and settlement reports.
You can also subscribe to daily or monthly transaction reports.

## For how long is an initiated payment reserved?

In the bank the transaction is reserved for 7 days, however this varies depending on which bank the customer is using.
In Vipps we do not automatically change the status of the order.

If you try to capture a payment more than 7 days after the payment has been initiated and the reservation has been released,
Vipps will make a new payment request to the bank. If the user has sufficient funds everything will be successful.
If the user does not have coverage on his account at this time the payment will fail.

In many cases the bank will have a register of expired reservations and they will force it through if the account allows this.
This will put the account in the negative.
