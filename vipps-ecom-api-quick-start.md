<!-- START_METADATA
---
title: Quick start for the eCom API
sidebar_label: Quick start
sidebar_position: 10
description: Quick steps for getting started with the eCom API.
toc_min_heading_level: 2
toc_max_heading_level: 5
pagination_next: null
pagination_prev: null
---

import ApiSchema from '@theme/ApiSchema';
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
END_METADATA -->

# Quick start

<!-- START_COMMENT -->
ℹ️ Please use the website:
[Vipps MobilePay Technical Documentation](https://developer.vippsmobilepay.com/docs/APIs/ecom-api/).
<!-- END_COMMENT -->

## Before you begin

**Important:** We strongly recommend that you use the newer
[ePayment API](https://developer.vippsmobilepay.com/docs/APIs/epayment-api) instead.

The provided example values in this guide must be changed with the values for your sales unit and user.
This applies for API keys, HTTP headers, reference, phone number, etc.
Note that any currency amount must be an Integer value minimum 100 in øre.

## Your first Payment

### Step 1 - Setup

You must have already signed up as an organization with Vipps MobilePay and have
your test credentials from the merchant portal.

You will need the following values, as described in the
[Getting started guide](https://developer.vippsmobilepay.com/docs/getting-started):

* `client_id` - Client_id for a test sales unit.
* `client_secret` - Client_id for a test sales unit.
* `Ocp-Apim-Subscription-Key` - The subscription key for making API requests.
* `merchantSerialNumber` - The unique ID for a test sales unit.
* `mobileNumber` - The phone number for your
   [test user](https://developer.vippsmobilepay.com/docs/test-environment#test-users).

<Tabs
defaultValue="curl"
groupId="sdk-choice"
values={[
{label: 'curl', value: 'curl'},
{label: 'Postman', value: 'postman'},
]}>
<TabItem value="postman">

In Postman, import the following files:

* [eCom API Postman collection](/tools/vipps-ecom-api-postman-collection.json)
* [Global Postman environment](https://github.com/vippsas/vipps-developers/blob/master/tools/vipps-api-global-postman-environment.json)

🔥 **To reduce risk of exposure, never store production keys in Postman or any similar tools.** 🔥

Update the *Current Value* field in your Postman environment with your **Merchant Test** keys.
Use *Current Value* field for added security, as these values are not synced to the cloud.

</TabItem>
<TabItem value="curl">

No additional setup needed :)

</TabItem>
</Tabs>

### Step 2 - Authentication

For all the following, you will need an `access_token` from the
[Access token API](https://developer.vippsmobilepay.com/docs/APIs/access-token-api):
[`POST:/accesstoken/get`][access-token-endpoint].
This provides you with access to the API.

<Tabs
defaultValue="curl"
groupId="sdk-choice"
values={[
{label: 'curl', value: 'curl'},
{label: 'Postman', value: 'postman'},
]}>
<TabItem value="postman">

```bash
Send request Get Access Token
```

</TabItem>
<TabItem value="curl">

```bash
curl https://apitest.vipps.no/accessToken/get \
-H "client_id: YOUR-CLIENT-ID" \
-H "client_secret: YOUR-CLIENT-SECRET" \
-H "Ocp-Apim-Subscription-Key: YOUR-SUBSCRIPTION-KEY" \
-H "Merchant-Serial-Number: YOUR-MSN" \
-H "Vipps-System-Name: acme" \
-H "Vipps-System-Version: 3.1.2" \
-H "Vipps-System-Plugin-Name: acme-webshop" \
-H "Vipps-System-Plugin-Version: 4.5.6" \
-X POST \
--data ''
```

</TabItem>
</Tabs>

The property `access_token` should be used for all other API requests in the `Authorization` header as the Bearer token.

### Step 3 - A simple payment

Initiate a payment with: [`POST:/payments`][create-payment-endpoint].
When your test mobile number
is provided in `phoneNumber`, it will be pre-filled in the form.

<Tabs
defaultValue="curl"
groupId="sdk-choice"
values={[
{label: 'curl', value: 'curl'},
{label: 'Postman', value: 'postman'},
]}>
<TabItem value="postman">

```bash
Send request Initiate Payment
```

</TabItem>
<TabItem value="curl">

```bash
curl --location 'https://apitest.vipps.no/ecomm/v2/payments/' \
-H 'Content-Type: application/json' \
-H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni <truncated>" \
-H "Ocp-Apim-Subscription-Key: 0f14ebcab0ec4b29ae0cb90d91b4a84a" \
-H "Merchant-Serial-Number: YOUR-MSN" \
-H "Vipps-System-Name: acme" \
-H "Vipps-System-Version: 3.1.2" \
-H "Vipps-System-Plugin-Name: acme-webshop" \
-H "Vipps-System-Plugin-Version: 4.5.6" \
-X POST \
-d '{
  "customerInfo": {
    "mobileNumber": "91234567"
  },
  "merchantInfo": {
    "merchantSerialNumber": "123456",
    "callbackPrefix":"https://example.com/vipps/callbacks-for-payment-update-from-vipps",
    "fallBack": "https://example.com/vipps/fallback-result-page-for-both-success-and-failure/acme-shop-123-order123abc",
  },
  "transaction": {
    "amount": 49900,
    "orderId": "UNIQUE-PAYMENT-REFERENCE",
    "transactionText": "One pair of socks.",
}
}'
```

Note that `orderId` must be unique for each payment you create.

</TabItem>
</Tabs>

### Step 4 - Completing the payment

*Ctrl+click* (*Command-click* on macOS) on the link that appears, and it will take you to the
[landing page](https://developer.vippsmobilepay.com/docs/common-topics/landing-page/).
The phone number of your test user should already be filled in, so you only have to click *Next*.
You will be presented with the payment in the app, where you can complete or reject the payment.
Once you have acted upon the payment, you will be redirected back to the specified `fallBack` URL under a "best effort" policy.

:::note
We cannot guarantee the user will be redirected back to the same browser or session, or that they will at all be redirected back. User interaction can be unpredictable, and the user may choose to fully close the app or browser.
:::

### Step 5 - Getting the status of the payment

To receive the result of the user action you may poll the status of the payment via the
[`GET:/payments/{orderId}/details`][get-payment-endpoint].

<Tabs
defaultValue="curl"
groupId="sdk-choice"
values={[
{label: 'curl', value: 'curl'},
{label: 'Postman', value: 'postman'},
]}>
<TabItem value="postman">

```bash
Send request Get Payment Details
```

</TabItem>
<TabItem value="curl">

```bash
curl https://apitest.vipps.no/ecomm/v2/payments/UNIQUE-PAYMENT-REFERENCE/details \
-H "Content-Type: application/json" \
-H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni <truncated>" \
-H "Ocp-Apim-Subscription-Key: 0f14ebcab0ec4b29ae0cb90d91b4a84a" \
-H "Merchant-Serial-Number: YOUR-MSN" \
-H "Vipps-System-Name: acme" \
-H "Vipps-System-Version: 3.1.2" \
-H "Vipps-System-Plugin-Name: acme-webshop" \
-H "Vipps-System-Plugin-Version: 4.5.6" \
-X GET
```

</TabItem>
</Tabs>

### Step 6 - Capture the payment

After the goods or services have been delivered, you can capture the authorized
amount either partially or fully:
[`POST:/payments/{orderId}/capture`][capture-payment-endpoint].

<Tabs
defaultValue="curl"
groupId="sdk-choice"
values={[
{label: 'curl', value: 'curl'},
{label: 'Postman', value: 'postman'},
]}>
<TabItem value="postman">

```bash
Send request Capture payment
```

</TabItem>
<TabItem value="curl">

```bash
curl https://apitest.vipps.no/ecomm/v2/payments/UNIQUE-PAYMENT-REFERENCE/capture \
-H "Content-Type: application/json" \
-H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni <truncated>" \
-H "Ocp-Apim-Subscription-Key: 0f14ebcab0ec4b29ae0cb90d91b4a84a" \
-H "Merchant-Serial-Number: YOUR-MSN" \
-H "Vipps-System-Name: acme" \
-H "Vipps-System-Version: 3.1.2" \
-H "Vipps-System-Plugin-Name: acme-webshop" \
-H "Vipps-System-Plugin-Version: 4.5.6" \
-H "X-Request-Id: 49ca711a-acee-4d01-993b-9487112e1def" \
-X POST \
-d '{
    "merchantInfo": {
        "merchantSerialNumber": "123456"
    },
    "transaction": {
        "transactionText": "This transaction was captured."
    }
}'
```

</TabItem>
</Tabs>

See
[Common topics: Capture](https://developer.vippsmobilepay.com/docs/common-topics/reserve-and-capture#capture)
for more details.

## Optional steps

### Refund the payment

To refund the captured amount, either partially or fully:
[`POST:/payments/{orderId}/refund`][refund-payment-endpoint].

<Tabs
defaultValue="curl"
groupId="sdk-choice"
values={[
{label: 'curl', value: 'curl'},
{label: 'Postman', value: 'postman'},
]}>
<TabItem value="postman">

```bash
Send request Refund payment
```

</TabItem>
<TabItem value="curl">

```bash
curl https://apitest.vipps.no/ecomm/v2/payments/UNIQUE-PAYMENT-REFERENCE/refund \
-H "Content-Type: application/json" \
-H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni <truncated>" \
-H "Ocp-Apim-Subscription-Key: 0f14ebcab0ec4b29ae0cb90d91b4a84a" \
-H "Merchant-Serial-Number: YOUR-MSN" \
-H "Vipps-System-Name: acme" \
-H "Vipps-System-Version: 3.1.2" \
-H "Vipps-System-Plugin-Name: acme-webshop" \
-H "Vipps-System-Plugin-Version: 4.5.6" \
-H "X-Request-Id: 49ca711a-acee-4d01-993b-9487112e1def" \
-X POST \
-d '{
  "merchantInfo": {
        "merchantSerialNumber": "123456"
    },
    "transaction": {
        "transactionText":"This payment was refunded."
    }
}'
```

</TabItem>
</Tabs>

See
[Common topics: refund](https://developer.vippsmobilepay.com/docs/common-topics/refund)
for more details.

### Cancel the payment

To cancel the payment, either fully or after a partial capture:
[`POST:/payments/{orderId}/cancel`][cancel-payment-endpoint].

<Tabs
defaultValue="curl"
groupId="sdk-choice"
values={[
{label: 'curl', value: 'curl'},
{label: 'Postman', value: 'postman'},
]}>
<TabItem value="postman">

```bash
Send request Cancel payment
```

</TabItem>
<TabItem value="curl">

```bash
curl https://apitest.vipps.no/ecomm/v2/payments/UNIQUE-PAYMENT-REFERENCE/cancel \
-H "Content-Type: application/json" \
-H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni <truncated>" \
-H "Ocp-Apim-Subscription-Key: 0f14ebcab0ec4b29ae0cb90d91b4a84a" \
-H "Merchant-Serial-Number: YOUR-MSN" \
-H "Vipps-System-Name: acme" \
-H "Vipps-System-Version: 3.1.2" \
-H "Vipps-System-Plugin-Name: acme-webshop" \
-H "Vipps-System-Plugin-Version: 4.5.6" \
-X PUT \
-d '{
  "merchantInfo": {
        "merchantSerialNumber": "123456"
    },
    "transaction": {
        "transactionText":"This payment was cancelled."
    }
}'

```

</TabItem>
</Tabs>

See
[Common topics: Cancel](https://developer.vippsmobilepay.com/docs/common-topics/cancel)
for more details.

## Next Steps

You will find more examples of eCom API requests in the
[eCom API Postman collection](/tools/vipps-ecom-api-postman-collection.json).

Experiment with these or go straight to the [eCom API Guide](vipps-ecom-api.md).

[access-token-endpoint]: https://developer.vippsmobilepay.com/api/access-token#tag/Authorization-Service/operation/fetchAuthorizationTokenUsingPost
[create-payment-endpoint]: https://developer.vippsmobilepay.com/api/ecom/#tag/Vipps-eCom-API/operation/initiatePaymentV3UsingPOST
[get-payment-endpoint]: https://developer.vippsmobilepay.com/api/ecom/#tag/Vipps-eCom-API/operation/getPaymentDetailsUsingGET
[capture-payment-endpoint]: https://developer.vippsmobilepay.com/api/ecom/#tag/Vipps-eCom-API/operation/capturePaymentUsingPOST
[refund-payment-endpoint]: https://developer.vippsmobilepay.com/api/ecom/#tag/Vipps-eCom-API/operation/refundPaymentUsingPOST
[cancel-payment-endpoint]: https://developer.vippsmobilepay.com/api/ecom/#tag/Vipps-eCom-API/operation/cancelPaymentRequestUsingPUT
