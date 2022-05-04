# Vipps eCommerce API Checklist

API version: 2.0.

Document version 2.1.8.

## Checklist

- [ ] Integrate _all_ the [API endpoints](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#api-endpoints):
    - [ ] Initiate [`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/initiatePaymentV3UsingPOST)
    - [ ] Full and partial Capture [`POST:/ecomm/v2/payments/{orderId}/capture`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/capturePaymentUsingPOST)
    - [ ] Cancel [`PUT:/ecomm/v2/payments/{orderId}/cancel`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/cancelPaymentRequestUsingPUT)
    - [ ] Full and partial Refund [`POST:/ecomm/v2/payments/{orderId}/refund`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/refundPaymentUsingPOST)
    - [ ] Details [`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/getPaymentDetailsUsingGET)
    - For examples of requests and responses, see the Postman collection in [tools](tools/).
- [ ] Send the [Vipps HTTP headers](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#vipps-http-headers)
      in all API requests for better tracking and troubleshooting (mandatory for partners and platforms):
    - [ ] `Merchant-Serial-Number`    
    - [ ] `Vipps-System-Name`
    - [ ] `Vipps-System-Version`
    - [ ] `Vipps-System-Plugin-Name`
    - [ ] `Vipps-System-Plugin-Version`
- [ ] Follow the [orderId recommendations](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#orderid-recommendations).
- [ ] Correctly handle callbacks from Vipps, both for successful and unsuccessful payments.
      See the API documentation for
      [how callback URLs are built](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#callback-endpoints),
      make test calls to make sure you handle the `POST` requests correctly.
      Vipps does not have capacity to manually do this for you.
    - [ ] Callback [`POST:[callbackPrefix]/v2/payments/{orderId}`](https://vippsas.github.io/vipps-ecom-api/#/Endpoints_required_by_Vipps_from_the_merchant/transactionUpdateCallbackForRegularPaymentUsingPOST)
    - [ ] For Vipps Hurtigkasse (express checkout) only:
        - [ ] Shipping details
              [`POST:[shippingDetailsPrefix]/v2/payments/{orderId}/shippingDetails`](https://vippsas.github.io/vipps-ecom-api/#/Endpoints_required_by_Vipps_from_the_merchant/fetchShippingCostUsingPOST)
        - [ ] Remove consent
              [`DELETE:[consetRemovalPrefix]/v2/consents/{userId}`](https://vippsas.github.io/vipps-ecom-api/#/Endpoints_required_by_Vipps_from_the_merchant/removeUserConsentUsingDELETE)
 - [ ] Make sure to log and handle [all errors](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#errors).
 - [ ] Avoid Integration pitfalls
    - [ ] The Merchant _must not_ rely on `fallback` or `callback` alone, and must poll
          [`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/getPaymentDetailsUsingGET)
          as documented (this is part of the first item in this checklist, but it's still a common error). Follow our [polling recommendations](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#polling-guidelines).  
    - [ ] The merchant must handle that the `fallback` URL is opened in the default browser on the phone,
          and not in a specific browser, in a specific tab, in an embedded browser, requiring a session token, etc.
          See the API guide:
          [Recommendations regarding handling redirects](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#recommendations-regarding-handling-redirects).
          See the FAQ: [How can I open the fallback URL in a specific (embedded) browser?](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api-faq.md#how-can-i-open-the-fallback-url-in-a-specific-embedded-browser)
    - [ ] The Vipps branding must be according to the
          [Vipps design guidelines](https://github.com/vippsas/vipps-design-guidelines).
    - [ ] Make sure your customer service, etc has all the tools and information they need
          available in _your_ system, through the APIs listed in the first item in this checklist,
          and that they do not need to visit
          [portal.vipps.no](https://portal.vipps.no)
          for normal work.

## Flow to go live for direct integrations

1. The merchant orders
   [Vipps på Nett](https://www.vipps.no/produkter-og-tjenester/bedrift/ta-betalt-paa-nett/ta-betalt-paa-nett/).
2. Vipps completes customer control (KYC, PEP, AML, etc).
3. The merchant receives an email from Vipps saying that they can log in with
   BankID on
   [portal.vipps.no](https://portal.vipps.no)
   and retrieve API keys.
4. The merchant completes all checklist items above. Please double check to avoid mistakes.
5. The merchant verifies the integration in the test environment by checking that
   there are test IDs (`orderId`) in the
   [Vipps test environment](https://github.com/vippsas/vipps-developers#the-vipps-test-environment-mt),
   with the following states:
    - A complete order ending in `REFUND`
      ([`/refund`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/refundPaymentUsingPOST)
      request).
    - A complete order ending in `VOID`
      ([`/cancel`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/cancelPaymentRequestUsingPUT)
      request).
    - In the test environment this must be verified using the API itself.
6. The Merchant verifies the integration in the production environment (similar to step 5):
    - A complete order ending in `REFUND`
      ([`/refund`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/refundPaymentUsingPOST)
      request).
    - For *reserve capture*: A complete order ending in `VOID`
      ([`/cancel`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/cancelPaymentRequestUsingPUT)
      request after reserve).
    - We recommend checking this using both the API itself and the API Dashboard available under "Utvikler" on
      [portal.vipps.no](https://portal.vipps.no).  
    - **Please note:** Vipps does not do any kind of activation or make any changes based on this checklist.
      The API keys for the production environment are made available on
      [portal.vipps.no](https://portal.vipps.no)
      as soon as the customer control (see step 2) is completed, independently of this checklist.
7. The Merchant goes live 🎉

## Flow to go live for direct integrations for partners

See: [Vipps partners](https://github.com/vippsas/vipps-partner#vipps-partners).

# Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-ecom-api/issues),
a [pull request](https://github.com/vippsas/vipps-ecom-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).

Sign up for our [Technical newsletter for developers](https://github.com/vippsas/vipps-developers/tree/master/newsletters).
