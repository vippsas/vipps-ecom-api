# Vipps eCommerce API Checklist

API version: 2.0

Document version 1.1.1

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
    - [ ] For express checkout only: Shipping details [`POST:[shippingDetailsPrefix]/v2/payments/{orderId}/shippingDetails`](https://vippsas.github.io/vipps-ecom-api/#/Endpoints_required_by_Vipps_from_the_merchant/fetchShippingCostUsingPOST)
    - [ ] For express checkout only: Remove consent [`DELETE:[consetRemovalPrefix]/v2/consents/{userId}`](https://vippsas.github.io/vipps-ecom-api/#/Endpoints_required_by_Vipps_from_the_merchant/removeUserConsentUsingDELETE)
- [ ] Avoid Integration pitfalls
    - [ ] The Merchant _must not_ rely on `fallback` alone
    - [ ] The Vipps branding must be according to the [Vipps design guidelines](https://github.com/vippsas/vipps-design-guidelines)

## Flow to go live for direct integrations

1. The merchant orders [Vipps på Nett](https://www.vipps.no/produkter-og-tjenester/bedrift/ta-betalt-paa-nett/ta-betalt-paa-nett/).
2. Vipps completes customer control (KYC, PEP, AML, etc).
3. The merchant receives an email from Vipps saying that they can log in with bankID on [portal.vipps.no](https://portal.vipps.no) and retrieve API keys.
4. The merchant completes all checklist items.
5. The merchant [contacts Vipps](https://github.com/vippsas/vipps-developers/blob/master/contact.md) with test IDs (`orderId`) in the [Vipps test environment](https://github.com/vippsas/vipps-developers#the-vipps-test-environment-mt), showing that all checklist items have been fulfilled.
    - A complete order ending in `REFUND` ([`/refund`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/refundPaymentUsingPOST) request).
    - A complete order ending in `VOID` ([`/cancel`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/cancelPaymentRequestUsingPUT) request).
6. The merchant receives an email from Vipps saying that the orders are OK.
7. The Merchant [contacts Vipps](https://github.com/vippsas/vipps-developers/blob/master/contact.md) to verify the integration in the production environment:
    - A complete order ending in `REFUND` ([`/refund`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/refundPaymentUsingPOST) request).
    - A complete order ending in `VOID` ([`/cancel`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/cancelPaymentRequestUsingPUT) request).
8. The Merchant goes live 🎉
