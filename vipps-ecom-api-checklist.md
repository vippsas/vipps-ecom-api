# Vipps eCommerce API Checklist

Document version 0.1.0

# Overall flow for direct integrations

1. The merchant orders [Vipps på Nett](https://www.vipps.no/bedrift/vipps-pa-nett)
2. Vipps completes customer control (KYC, PEP, AML, etc)
3. The merchant received login credentials for retrieving API keys
4. The merchant completes the direct integration with the [Vipps eCommerce API](https://github.com/vippsas/vipps-ecom-api)
  1. This includes _all_ the [API endpoints](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#api-endpoints)
5. The merchant completes testing in the [Vipps test environment](https://github.com/vippsas/vipps-developers#the-vipps-test-environment-mt)
6. The merchant completes one or more test transactions in the production environment
7. The merchant [contacts Vipps](https://github.com/vippsas/vipps-developers/blob/master/contact.md) to verify the integration in the production environment
8. Vipps completes one or more transactions in the production environment
9. The merchant goes live 🎉
