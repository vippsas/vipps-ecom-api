# Vipps eCommerce API Checklist

Document version 1.0.2

# Overall flow for direct integrations

1. The merchant orders [Vipps på Nett](https://www.vipps.no/produkter-og-tjenester/bedrift/ta-betalt-paa-nett/ta-betalt-paa-nett/)
2. Vipps completes customer control (KYC, PEP, AML, etc)
3. The merchant receives an email that they can log in with bankID on portal.vipps.no and retrieve API keys
4. The merchant completes the direct integration with the [Vipps eCommerce API](https://github.com/vippsas/vipps-ecom-api)
  - This includes _all_ the [API endpoints](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#api-endpoints):
  - [`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/initiatePaymentV3UsingPOST)
  - [`POST:/ecomm/v2/payments/{orderId}/capture`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/capturePaymentUsingPOST)
  - [`PUT:/ecomm/v2/payments/{orderId}/cancel`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/cancelPaymentRequestUsingPUT)
  - [`POST:/ecomm/v2/payments/{orderId}/refund`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/refundPaymentUsingPOST)
  - [`GET:/ecomm/v2/payments/{orderId}/details`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/getPaymentDetailsUsingGET)
5. The merchant correctly handles callbacks, both for successful and unsuccessful payments
  - [`POST:[callbackPrefix]/v2/payments/{orderId}`](https://vippsas.github.io/vipps-ecom-api/#/Endpoints_required_by_Vipps_from_the_merchant/transactionUpdateCallbackForRegularPaymentUsingPOST)
6. The merchant completes testing in the [Vipps test environment](https://github.com/vippsas/vipps-developers#the-vipps-test-environment-mt)
7. The merchant [contacts Vipps](https://github.com/vippsas/vipps-developers/blob/master/contact.md) to verify the integration in the test environment:
  - At least one order id for orders with each of the following statuses: capture, refund, cancel.
8. The merchant completes one or more test transactions in the production environment
9. The merchant [contacts Vipps](https://github.com/vippsas/vipps-developers/blob/master/contact.md) to verify the integration in the production environment:
  - At least one order id for orders with each of the following statuses: capture, refund, cancel.
10. Vipps completes one or more transactions in the production environment
11. The merchant goes live 🎉
