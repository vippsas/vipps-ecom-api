# Deprecation of the Vipps eCom API v1

This is the message we sent to communicate the deprecation of the API.

Document version 1.2.0.

<br/>
<br/>
<br/>

Hi

Vipps is constantly improving our services, and in some cases this requires us
to sunset legacy products and APIs.

The Vipps eCom API was superseded by the eCom v2 API around two years ago.
This means that the eCom v1 API is technologically outdated.
Therefore:

**The Vipps eCom v1 API will be shut down on September 1 2020.**

The eCom v2 API has all the functionality of eCom v1, and there is no reason
for not upgrading to eCom v2.

We only have a few customers that still use the eCom v1 API, and we are sending
this message to those who have made API calls during the last few months.

This eCom v1 API shutdown means that:

* Your current integration needs to be updated to use the eCom v2 API. The API
  is [well documented](https://github.com/vippsas/vipps-ecom-api), with an
  extensive guide, Swagger specification, Postman collection, integration
  checklist and a migration guide.
* Integrations using the eCom v1 API will stop working on September 1 2020.

The main benefits of the eCom v2 API:

* Higher conversion rates and a better user experience for the Vipps users
* A Vipps landing page that ensures a consistent user experience, both on mobile and desktop
* A more robust solution with improved error handling, error messages and logging
* Active maintenance and continuous improvement by Vipps

Some customers have not been able to use the eCom v2 API because they have not
had a display available for the landing page. In these special cases we offer a
feature to skip the landing page. Customers who can not display the landing page
can see more details about the "skipLandingPage" functionality
[here](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#skip-landing-page).

Vipps is piloting a solution for tokenised BankAxept-payments in payment terminals using Vipps
as a wallet. To learn about availability, contact your payment terminal provider.

Vipps is currently designing and developing solutions for direct integration into POS systems, electronic cash registers and vending machines. For more information about this, please [register here](https://forms.office.com/Pages/ResponsePage.aspx?id=XcJbgGSO1k6NJDiDyQaMWuVjn37JcrJJgJkaJ8cPvvVUQVdLVk9PTkZTRDBLSFRRNzQxTlc2VThZMS4u).

If you have any questions about the eCom v2 API, please see the GitHub documentation.
You can contact Vipps Integration if you have questions. See contact information,
information about test environments, test apps, etc [here](https://github.com/vippsas/vipps-developers).

We have also released new APIs that may be interesting, such as:
* [Vipps Recurring API ("faste betalinger")](https://github.com/vippsas/vipps-recurring-api)
* [Vipps Invoice API ("Vipps Regninger")](https://github.com/vippsas/vipps-invoice-api)
* [Vipps Log In](https://github.com/vippsas/vipps-login-api)

Vipps also has [plugins](https://github.com/vippsas/vipps-developers#plugins) available:
* [Vipps for WooCommerce (Wordpress)](https://wordpress.org/plugins/woo-vipps/)
* [Vipps for Magento](https://www.vipps.no/produkter-og-tjenester/bedrift/ta-betalt-paa-nett/ta-betalt-paa-nett/magento/)
* Upcoming: Vipps for Shopify, Vipps for Drupal and Vipps for Episerver.

See also our
[partners](https://www.vipps.no/produkter-og-tjenester/bedrift/ta-betalt-paa-nett/ta-betalt-paa-nett/#kom-i-gang-med-vipps-pa-nett-category-1),
who offer various Vipps services and integrations.

# Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-ecom-api/issues),
a [pull request](https://github.com/vippsas/vipps-ecom-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).

Sign up for our [Technical newsletter for developers](https://github.com/vippsas/vipps-developers/tree/master/newsletters).
