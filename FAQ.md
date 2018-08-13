# Frequently Asked Questions for Vipps eCommerce API

## I am unable to login to the developer portal in either test and/or production.

Make sure you are trying to log in to the right portal. If your username is `name@testapivipps.no`, 
then you should log into the test portal https://apitest-portal.vipps.no/.
If your username is `name@apivipps.no` then you should log in to the production portal https://api-portal.vipps.no/.
Open the link to the developer portal in an incognito/InPrivate-window in your web browser.

## I am getting 401 authorization error - and I have double checked all my keys!

401 occurs when there is a mismatch between the subscription keys and the merchant sales unit. Please follow these steps to make sure everything is correct:

1. Correct spelling of `Ocp-Apim-Subscription-Key` parameter in the header of `Access Token` and Payment API
2. Confirm that you are not using the same subscription key for `Access Token` and `Payment Initiation`
3. Make sure you are using the same `merchantSerialNumber` in the body of your request as is stated in the developer portal
4. Make sure you are making calls to `ecomm/v2/payments` and _not_ `ecomm/v1/payments` unless this is specifically agreed upon

## Everything worked yesterday and now I'm suddenly getting 500 Internal Server Error (or similar)

Something _might_ be wrong on our side and we are working to fix it! We are also working on an updated status page 
that we will link to from this page.

## I have not had time to test this month and when I came back to it now I get errorCode 37 "Merchant not available or deactivated or blocked"

This happens if the test merchant is not being used for some time. Please contact us (integrations@vipps.no), and we will reactivate the merchant.

## I have successfully tested my integration in the test environment and now need access to the production environment. How do I get it?

Please contact integration@vipps.no, we will activate your production account and you will receive login credentials 
to your production developer portal https://api-portal.vipps.no - If you do not receive credentials please double check 
your spam filter and trash.

## How do I perform testing in production?

To do this you need a live Vipps account - we do not have test accounts available for Vipps. 
We recommend testing with 2 NOK, even though 1 NOK is the smallest possible amount - 
it is not reliable, as it gets low priority in some systems. 

## What is the difference between "Reserve Capture" and "Direct Capture", and how do I change payment type?

`Reserve Capture` is the default. When you initiate a payment it will be reserved until you capture it. 
According to Norwegian regulations you should not capture a payment until the product or service is provided to the customer.
When direct capture is activated, all payment reservations will instantly be captured. 
This is intended for situations where the product or service is immediately provided to the customer.

## I have initiated a payment (200 NOK) for two items in one order. The customer changed his mind and only bought one of the items. I have performed a partial capture of 100 NOK. How do I cancel the remaining 100 NOK ? 

It's not possible to cancel the remaining reservation after a partial capture, but when the payment is confirmed 
in the bank (2-3 days later), the money will be available to the customer again.

## I have initiated an order but I can't find it. 

Have you successfully implemented `getPaymentDetails` or `getOrderStatus`? 
In case our callback fails, you will not automatically received notification of order status. 
In case this fails the backup should be to ask for status within a time frame.

## Where can I find reports on transactions?

The Vipps portal provides information about your transactions, your sale units and settlement reports. 
You can also subscribe to daily or monthly reports of transactions.

## For how long is an initiated payment reserved?
In the bank the transaction is reserved for 7 days, however this varies depending on which bank the customer is using. 
In Vipps we do not automatically change the status of the order. 
If you try to capture a payment more than 7 days after the payment has been initiated and the reservation has been released, 
Vipps will make a new payment request to the bank. 
If the user has sufficient funds everything will be successful. 
If the user does not have coverage on his account at this time the payment will fail. 
In many cases the bank will have a register of expired reservations and they will force it through if the account allows this. 
This will put the account in minus.
