<!-- START_METADATA
---
title: eCom API checklist
sidebar_label: Checklist
sidebar_position: 20
description: Checklist for full integration with the eCom API.
pagination_next: null
pagination_prev: null
---
END_METADATA -->

# Checklist

<!-- START_COMMENT -->

ℹ️ Please use the website:
[Vipps MobilePay Technical Documentation](https://vippsas.github.io/vipps-developer-docs/docs/APIs/ecom-api).

<!-- END_COMMENT -->

API version: 2.0.

## Checklist for full integration

- [ ] Integrate _all_ the [API endpoints](vipps-ecom-api.md#api-endpoints):
    - [ ] Initiate [`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-developer-docs/api/ecom#tag/Vipps-eCom-API/operation/initiatePaymentV3UsingPOST)
    - [ ] Full and partial Capture [`POST:/ecomm/v2/payments/{orderId}/capture`](https://vippsas.github.io/vipps-developer-docs/api/ecom#tag/Vipps-eCom-API/operation/capturePaymentUsingPOST)
    - [ ] Cancel [`PUT:/ecomm/v2/payments/{orderId}/cancel`](https://vippsas.github.io/vipps-developer-docs/api/ecom#tag/Vipps-eCom-API/operation/cancelPaymentRequestUsingPUT)
    - [ ] Full and partial Refund [`POST:/ecomm/v2/payments/{orderId}/refund`](https://vippsas.github.io/vipps-developer-docs/api/ecom#tag/Vipps-eCom-API/operation/refundPaymentUsingPOST)
    - [ ] Details [`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-developer-docs/api/ecom#tag/Vipps-eCom-API/operation/getPaymentDetailsUsingGET)
    - For examples of requests and responses, see the [Postman collection](tools/vipps-ecom-api-postman-collection.json) and [environment](https://github.com/vippsas/vipps-developers/blob/master/tools/vipps-api-global-postman-environment.json).
- [ ] Send the [Vipps HTTP headers](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/common-topics/http-headers)
      in all API requests for better tracking and troubleshooting
      (mandatory for partners and platforms, who must send these headers as part of the checklist approval):
    - [ ] `Merchant-Serial-Number`
    - [ ] `Vipps-System-Name`
    - [ ] `Vipps-System-Version`
    - [ ] `Vipps-System-Plugin-Name`
    - [ ] `Vipps-System-Plugin-Version`
- [ ] Follow the [orderId recommendations](vipps-ecom-api.md#orderid-recommendations).
- [ ] Correctly handle callbacks from Vipps, both for successful and unsuccessful payments.
      See the API documentation for
      [how callback URLs are built](vipps-ecom-api.md#callback-endpoints),
      make test calls to make sure you handle the `POST` requests correctly.
      Vipps does not have capacity to manually do this for you.
    - [ ] Callback [`POST:[callbackPrefix]/v2/payments/{orderId}`](https://vippsas.github.io/vipps-developer-docs/api/ecom#tag/Merchant-Endpoints/operation/transactionUpdateCallbackForRegularPaymentUsingPOST)
    - [ ] For Vipps Hurtigkasse (express checkout) only:
        - [ ] Shipping details
              [`POST:[shippingDetailsPrefix]/v2/payments/{orderId}/shippingDetails`](https://vippsas.github.io/vipps-developer-docs/api/ecom#tag/Merchant-Endpoints/operation/fetchShippingCostUsingPOST)
        - [ ] Remove consent
              [`DELETE:[consentRemovalPrefix]/v2/consents/{userId}`](https://vippsas.github.io/vipps-developer-docs/api/ecom#tag/Merchant-Endpoints/operation/removeUserConsentUsingDELETE)
 - [ ] Make sure to log and handle [all errors](vipps-ecom-api.md#errors).
       All integrations should to display errors in a way that the user can see and understand them.
 - [ ] Avoid Integration pitfalls
    - [ ] The Merchant _must not_ rely on `fallback` or `callback` alone, and must poll
          [`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-developer-docs/api/ecom#tag/Vipps-eCom-API/operation/getPaymentDetailsUsingGET)
          as documented (this is part of the first item in this checklist, but it's still a common error).
          Follow our [polling recommendations](vipps-ecom-api.md#polling-guidelines).  
    - [ ] The merchant must handle that the `fallback` URL is opened in the default browser on the phone,
          and not in a specific browser, in a specific tab, in an embedded browser, requiring a session token, etc.
          See the API guide:
          [Recommendations regarding handling redirects](vipps-ecom-api.md#recommendations-regarding-handling-redirects).
          See the FAQ: [How can I open the fallback URL in a specific (embedded) browser?](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/faqs/common-problems-faq#how-can-i-open-the-fallback-url-in-a-specific-embedded-browser)
    - [ ] The Vipps branding must be according to the
          [Vipps design guidelines](https://github.com/vippsas/vipps-design-guidelines).
    - [ ] Make sure your users (meaning the merchant's employees), customer service, etc.
          have all the tools and information they need available in _your_ system, through
          the APIs listed in the first item in this checklist.
          It should never be necessary to to visit
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
4. The merchant completes all checklist items above.
   Please double check to avoid mistakes.
5. The merchant verifies the integration in the test environment by checking that
   there are test IDs (`orderId`) in the
   [Vipps test environment](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/test-environment),
   with the following states:
    - A complete order ending in `REFUND`
      ([`/refund`](https://vippsas.github.io/vipps-developer-docs/api/ecom#tag/Vipps-eCom-API/operation/refundPaymentUsingPOST)
      request).
    - A complete order ending in `VOID`
      ([`/cancel`](https://vippsas.github.io/vipps-developer-docs/api/ecom#tag/Vipps-eCom-API/operation/cancelPaymentRequestUsingPUT)
      request).
    - In the test environment this must be verified using the API itself.
6. The Merchant verifies the integration in the production environment (similar to step 5):
    - A complete order ending in `REFUND`
      ([`/refund`](https://vippsas.github.io/vipps-developer-docs/api/ecom#tag/Vipps-eCom-API/operation/refundPaymentUsingPOST)
      request).
    - For *reserve capture*: A complete order ending in `VOID`
      ([`/cancel`](https://vippsas.github.io/vipps-developer-docs/api/ecom#tag/Vipps-eCom-API/operation/cancelPaymentRequestUsingPUT)
      request after reserve).
    - We recommend checking this using both the API itself and the API Dashboard available under "Utvikler" on
      [portal.vipps.no](https://portal.vipps.no).  
    - **Please note:** Vipps does not do any kind of activation or make any changes based on this checklist.
      The API keys for the production environment are made available on
      [portal.vipps.no](https://portal.vipps.no)
      as soon as the customer control (see step 2) is completed, independently of this checklist.
7. The Merchant goes live 🎉

## Flow to go live for direct integrations for Vipps partners

See: [Vipps partners](https://vippsas.github.io/vipps-developer-docs/docs/vipps-partner).
