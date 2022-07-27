# Postman

[Postman](https://www.getpostman.com/) is a common tool for working with REST APIs.
We offer a [Postman Collection](https://www.getpostman.com/collection) to make development easier.
See the [Postman documentation](https://www.getpostman.com/docs/) for more information about using Postman.

By following the steps below, you can make calls to all the
endpoints, and see the full `request` and `response` for each call.

We also have a short [getting started guide to Postman](https://github.com/vippsas/vipps-developers/blob/master/postman-guide.md).

## Setting up Postman

### Step 1: Get the Postman Collection

Import the collection by following the steps below:

1. Click `Import` in the upper-left corner.
2. Import the [vipps-ecom-api-postman-collection.json](https://raw.githubusercontent.com/vippsas/vipps-ecom-api/master/tools/vipps-ecom-api-postman-collection.json) file.

### Step 2: Import the Postman Environment

1. Click `Import` in the upper-left corner.
2. Import the [vipps-ecom-api-postman-environment.json](https://raw.githubusercontent.com/vippsas/vipps-ecom-api/master/tools/vipps-ecom-api-postman-environment.json) file.

### Step 3: Set up Postman Environment

1. Click the down arrow, next to the "eye" icon in the top-right corner, and select the environment you have imported.
2. Click the "eye" icon and, in the dropdown window, click `Edit` in the top-right corner.
3. Fill in the `Current Value` for the following fields to get started. For the first three keys, go to *Vipps Portal* > *Utvikler* ->  *Test Keys*.
   - `client_id` - Merchant key is required for getting the access token.
   - `client_secret` - Merchant key is required for getting the access token.
   - `Ocp-Apim-Subscription-Key` - Merchant subscription key.
   - `merchantSerialNumber` - Merchant id.
   - `mobileNumber` - The mobile number for the test app profile you have received or registered.
   - `amount` - The desired amount Integer value minimum 100 in Øre.

### Step 4: Run the examples

See the  [eCommerce API Specifications](https://vippsas.github.io/vipps-ecom-api/#/) for details about the calls.

#### 1. A regular eCommerce payment

1. Send request `Get Access Token`. This provides you with access to the API.

1. Send request `Initiate Payment`. This is to demonstrate a simple payment by using
   [`POST:/v3/psppayments/init/`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/initiatePaymentV3UsingPOST).

   Ctrl+click on the link that appears and it will take you to the website where you can enter your test phone number and complete the payment authorization in the Vipps app in your mobile test environment.

   The `orderId` in this Postman example is now set in the environment and can be used for subsequent calls.

1. Send request `Get Payment Details` for information about this payment by using [`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/getPaymentDetailsUsingGET).

1. Send request `Capture Payment` to capture this payment with [`POST:/ecomm/v2/payments/{orderId}/capture`](https://vippsas.github.io/vipps-ecom-api/#/Vipps_eCom_API/capturePaymentUsingPOST).

1. Send request `Refund Payment` to refund this payment with [`POST:/ecomm/v2/payments/{orderId}/refund`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/refundPaymentUsingPOST).

See [Regular eCommerce payments](vipps-ecom-api.md#regular-ecommerce-payments) for more information from the eCommerce API guide.

#### 2. An express checkout payment (*Vipps Hurtigkasse*)

1. Send request `Get Access Token`, if you haven't already.

1. Send request `Initiate Payment - Express Checkout`. This demonstrates the type of payment where the user selects their shipping methods within the Vipps app instead of on the website.

   To enable this functionality, you provide the object `staticShippingDetails` with your relevant details in the body of the call 
   [`POST:/v3/psppayments/init/`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/initiatePaymentV3UsingPOST).

   Ctrl+click on the link that appears and complete the payment authorization.

1. Send request `Get Payment Details` for information about this payment by using [`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/getPaymentDetailsUsingGET).

1. This time, instead of capturing the order, cancel it. Send request `Cancel Payment` to cancel this payment with [`POST:/ecomm/v2/payments/{orderId}/cancel`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/cancelPaymentRequestUsingPUT).

See [Express checkout payments](vipps-ecom-api.md#express-checkout-payments) for more information from the eCommerce API guide.

#### 3. Getting userinfo

1. Send request `Get Access Token`, if you haven't already.

1. Send request `Initiate Payment - Profile flow`. Provide the `scope` object in the [`POST:/v3/psppayments/init/`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/initiatePaymentV3UsingPOST) call. This contains the information types that you want access to, separated by spaces (e.g., "name address email phoneNumber birthDate").

   Ctrl+click on the link that appears and complete the authorization.


1. Send request `Get Payment Details` with `orderId` for information about this payment by using [`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/getPaymentDetailsUsingGET).

   Here, the user identifier, `sub`, is retrieved from the response and set as a variable.


1. Send request `Get Userinfo`. This uses [`GET:/vipps-userinfo-api/userinfo/{sub}`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/getUserinfo) with the `sub` variable from the previous call.

See [Userinfo](vipps-ecom-api.md#userinfo) for more information from the eCommerce API guide..


## Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-ecom-api/issues),
a [pull request](https://github.com/vippsas/vipps-ecom-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).

Sign up for our [Technical newsletter for developers](https://github.com/vippsas/vipps-developers/tree/master/newsletters).
