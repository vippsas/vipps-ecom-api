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
[Vipps MobilePay Technical Documentation](https://developer.vippsmobilepay.com/docs/APIs/ecom-api/).
<!-- END_COMMENT -->

API version: 2.0.

## Checklist for full integration

Integrate *all* the [API endpoints](https://developer.vippsmobilepay.com/api/ecom/). For examples of requests and responses, see the [Postman collection](/tools/vipps-ecom-api-postman-collection.json) and [environment](https://github.com/vippsas/vipps-developers/blob/master/tools/vipps-api-global-postman-environment.json).

| Endpoint | Comment |
|-----|-----------|
|     Initiate |  [`POST:/ecomm/v2/payments`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/initiatePaymentV3UsingPOST)|
|     Full and partial Capture| [`POST:/ecomm/v2/payments/{orderId}/capture`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/capturePaymentUsingPOST)|
|     Cancel| [`PUT:/ecomm/v2/payments/{orderId}/cancel`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/cancelPaymentRequestUsingPUT)|
|     Full and partial Refund| [`POST:/ecomm/v2/payments/{orderId}/refund`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/refundPaymentUsingPOST)|
|     Details| [`GET:/ecomm/v2/payments/{orderId}/details`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/getPaymentDetailsUsingGET)|
|     Callback| [`POST:[callbackPrefix]/v2/payments/{orderId}`](https://developer.vippsmobilepay.com/api/ecom#tag/Merchant-Endpoints/operation/transactionUpdateCallbackForRegularPaymentUsingPOST)|
|    Shipping Details (For Vipps Hurtigkasse /express checkout only) |[`POST:[shippingDetailsPrefix]/v2/payments/{orderId}/shippingDetails`](https://developer.vippsmobilepay.com/api/ecom#tag/Merchant-Endpoints/operation/fetchShippingCostUsingPOST)|
|    Remove consent (For Vipps Hurtigkasse /express checkout only) | [`DELETE:[consentRemovalPrefix]/v2/consents/{userId}`](https://developer.vippsmobilepay.com/api/ecom#tag/Merchant-Endpoints/operation/removeUserConsentUsingDELETE) |

## Quality assurance

| Action | Comment |
|-----|-----------|
|     Handle callbacks | Correctly handle callbacks from Vipps, both for successful and unsuccessful payments. See the API documentation for [how callback URLs are built](vipps-ecom-api.md#callback-endpoints), make test calls to make sure you handle the `POST` requests correctly. Vipps does not have capacity to manually do this for you. |
|     Handle errors | Make sure to log and handle [all errors](https://developer.vippsmobilepay.com/docs/APIs/ecom-api/vipps-ecom-api.md#errors). All integrations should display errors in a way that the users (customers and merchant employees/administrators) can see and understand them.|
|     Include Vipps HTTP headers | Send the [Vipps HTTP headers](https://developer.vippsmobilepay.com/docs/knowledge-base/http-headers) in all API requests for better tracking and troubleshooting (mandatory for partners and platforms, who must send these headers as part of the checklist approval). |
|     Add information to the payment history| We recommend using the [Order Management API](https://developer.vippsmobilepay.com/docs/APIs/order-management-api) to add receipts and/or images to the payment history. This is a great benefit for the end user experience. It is also mandatory for merchants using ["Vipps Assisted Content Monitoring"](https://developer.vippsmobilepay.com/docs/APIs/order-management-api/vipps-order-management-api#vipps-assisted-content-monitoring). |

## Avoid integration pitfalls

| Action    | Comment   |
|-----|-----------|
|     Send useful `OrderId` | Follow our [`orderId` recommendations](https://developer.vippsmobilepay.com/docs/knowledge-base/orderid). |
|     Poll for payment details | The Merchant *must not* rely on `fallback` or `callback` alone, and must poll [`GET:/ecomm/v2/payments/{orderId}/details`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/getPaymentDetailsUsingGET) as documented (this is part of the first item in this checklist, but it's still a common error). Follow our [polling recommendations](https://developer.vippsmobilepay.com/docs/knowledge-base/polling-guidelines). |
|     Handle redirects| The merchant must handle that the `fallback` URL is opened in the default browser on the phone, and not in a specific browser, in a specific tab, in an embedded browser, requiring a session token, etc. Follow our [recommendations regarding handling redirects](https://developer.vippsmobilepay.com/docs/knowledge-base/redirects/).|
|     Follow design guidelines| The Vipps branding must be according to the [Design guidelines](https://developer.vippsmobilepay.com/docs/design-guidelines).|
|     Educate customer support| Make sure your customer service, etc. has all the tools and information they need available in *your* system, through the APIs listed in the first item in this checklist, and that they do not need to visit [portal.vipps.no](https://portal.vipps.no) for normal work.|

## Flow to go live for direct integrations

1. The merchant orders
   [Vipps på Nett](https://www.vipps.no/produkter-og-tjenester/bedrift/ta-betalt-paa-nett/ta-betalt-paa-nett/).
2. Vipps completes customer control (KYC, PEP, AML, etc.).
3. The merchant receives an email from Vipps saying that they can log in with
   BankID on
   [portal.vipps.no](https://portal.vipps.no)
   and retrieve [API keys](https://developer.vippsmobilepay.com/docs/knowledge-base/api-keys/#getting-the-api-keys).
4. The merchant completes all checklist items above.
   Please double-check to avoid mistakes.
5. The merchant verifies the integration in the test environment by checking that
   there are test IDs (`orderId`) in the
   [Vipps test environment](https://developer.vippsmobilepay.com/docs/test-environment),
   with the following states:
   * A complete order ending in `REFUND`
      ([`/refund`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/refundPaymentUsingPOST)
      request).
   * A complete order ending in `VOID`
      ([`/cancel`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/cancelPaymentRequestUsingPUT)
      request).
   * In the test environment this must be verified using the API itself.
6. The merchant verifies the integration in the production environment (similar to step 5):
   * A complete order ending in `REFUND`
      ([`/refund`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/refundPaymentUsingPOST)
      request).
   * For *reserve capture*: A complete order ending in `VOID`
      ([`/cancel`](https://developer.vippsmobilepay.com/api/ecom#tag/Vipps-eCom-API/operation/cancelPaymentRequestUsingPUT)
      request after reserve).
   * We recommend checking this using both the API itself and the API Dashboard, available in the *Utvikler* section on the
      [merchant portal](https://portal.vipps.no).  
   * **Please note:** Vipps does not do any kind of activation or make any changes based on this checklist.
      The API keys for the production environment are made available on the
      [merchant portal](https://portal.vipps.no)
      as soon as the customer control (see step 2) is completed, independently of this checklist.
7. The merchant goes live 🎉

## Flow to go live for direct integrations for partners

See: [partners](https://developer.vippsmobilepay.com/docs/partner).
