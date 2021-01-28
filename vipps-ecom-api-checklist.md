# Vipps eCommerce API Checklist

API version: 2.0

Document version 2.0.0.

For examples of requests and responses, see the Postman collection in [tools](tools/)

## Checklist

- [ ] Integrate _all_ the [API endpoints](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#api-endpoints):
    - [ ] Initiate [`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/initiatePaymentV3UsingPOST)
    - [ ] Capture [`POST:/ecomm/v2/payments/{orderId}/capture`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/capturePaymentUsingPOST)
    - [ ] Cancel [`PUT:/ecomm/v2/payments/{orderId}/cancel`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/cancelPaymentRequestUsingPUT)
    - [ ] Refund [`POST:/ecomm/v2/payments/{orderId}/refund`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/refundPaymentUsingPOST)
    - [ ] Details [`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/getPaymentDetailsUsingGET)
- [ ] The merchant correctly handles callbacks, both for successful and unsuccessful payments
    - [ ] Callback [`POST:[callbackPrefix]/v2/payments/{orderId}`](https://vippsas.github.io/vipps-ecom-api/#/Endpoints_required_by_Vipps_from_the_merchant/transactionUpdateCallbackForRegularPaymentUsingPOST)
      _It's important that the integrator verifies that the callback URLs are correctly handled._
    - [ ] For express checkout only: Shipping details [`POST:[shippingDetailsPrefix]/v2/payments/{orderId}/shippingDetails`](https://vippsas.github.io/vipps-ecom-api/#/Endpoints_required_by_Vipps_from_the_merchant/fetchShippingCostUsingPOST)
    - [ ] For express checkout only: Remove consent [`DELETE:[consetRemovalPrefix]/v2/consents/{userId}`](https://vippsas.github.io/vipps-ecom-api/#/Endpoints_required_by_Vipps_from_the_merchant/removeUserConsentUsingDELETE)
- [ ] Avoid Integration pitfalls
    - [ ] The Merchant _must not_ rely on `fallback` or `callback` alone, and must poll
          [`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/getPaymentDetailsUsingGET)
          as documented.
    - [ ] The merchant must handle that the `fallback`URL is opened in the default browser on the phone, and not in a custom browser, in a specific tab, or in an embedded browser. See the FAQ: [How can I open the fallback URL in a specific (embedded) browser?](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api-faq.md#how-can-i-open-the-fallback-url-in-a-specific-embedded-browser)      
    - [ ] The Vipps branding must be according to the [Vipps design guidelines](https://github.com/vippsas/vipps-design-guidelines)
- [ ] Integrate [HTTP headers](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#optional-vipps-http-headers) for better tracking (Mandatory for partners and plattforms)
    - [ ] Vipps-System-Name
    - [ ] Vipps-System-Version
    - [ ] Vipps-System-Plugin-Name
    - [ ] Vipps-System-Plugin-Version    

## Flow to go live for direct integrations

1. The merchant orders [Vipps på Nett](https://www.vipps.no/produkter-og-tjenester/bedrift/ta-betalt-paa-nett/ta-betalt-paa-nett/).
2. Vipps completes customer control (KYC, PEP, AML, etc).
3. The merchant receives an email from Vipps saying that they can log in with bankID on [portal.vipps.no](https://portal.vipps.no) and retrieve API keys.
4. The merchant completes all checklist items. Please double check to avoid mistakes.
5. The merchant verifies the integration in the test environment by checking that there are test IDs (`orderId`) in the [Vipps test environment](https://github.com/vippsas/vipps-developers#the-vipps-test-environment-mt), showing that all checklist items have been fulfilled.
    - A complete order ending in `REFUND` ([`/refund`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/refundPaymentUsingPOST) request).
    - A complete order ending in `VOID` ([`/cancel`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/cancelPaymentRequestUsingPUT) request).
    - We recommend using the API Dashboard available under "Utvikler" on
      [portal.vipps.no](https://portal.vipps.no).
6. The Merchant verifies the integration in the production environment (similar to step 5):
    - A complete order ending in `REFUND` ([`/refund`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/refundPaymentUsingPOST) request).
    - For *reserve capture*: A complete order ending in `VOID` ([`/cancel`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/cancelPaymentRequestUsingPUT) request after reserve).
7. The Merchant goes live 🎉

## Flow to go live for direct integrations for partners

1. The partner becomes a partner by [applying here](https://www.vipps.no/produkter-og-tjenester/bedrift/ta-betalt-i-butikk/vipps-i-kassa/#kom-i-gang-med-vipps-i-kassa-category-3)
2. The partner completes the integration, with the API test keys.
3. The partner [contacts Vipps](https://github.com/vippsas/vipps-developers/blob/master/contact.md) with test IDs (`orderId`) in the [Vipps test environment](https://github.com/vippsas/vipps-developers#the-vipps-test-environment-mt), showing that all checklist items have been completed.
    - A complete order ending in `REFUND` ([`/refund`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/refundPaymentUsingPOST) request).
    - For *reserve capture*: A complete order ending in `VOID` ([`/cancel`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/cancelPaymentRequestUsingPUT) request after reserve).
4. The partner receives an email from Vipps saying that the implementation is OK.
5. Vipps adds the partner to vipps.no, including the signup forms for merchants.
6. The partner add its merchant to their solution, usually by configuring the POS with the merchant's API keys.
7. The Merchant goes live 🎉

**Please note:** For POS integrations that can not display the Vipps
landing page, it is important that all sale units are configured with
`skipLandingPage`. See the
[Frequently Asked Questions for POS integrations](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api-faq.md#frequently-asked-questions-for-pos-integrations).

# Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-ecom-api/issues),
a [pull request](https://github.com/vippsas/vipps-ecom-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).

Sign up for our [Technical newsletter for developers](https://github.com/vippsas/vipps-developers/tree/master/newsletters).
