# Vipps eCommerce API

API version: 2.1.

Document version 0.1.0.

_**IMPORTANT:**_ This document is a work in progress. See the official PDF documentation:
https://github.com/vippsas/vipps-ecom-api/tree/master/original-docs

# Table of contents

TODO: Generate, possibly with https://github.com/jonschlinkert/markdown-toc

# Overview

Vipps eCommerce API gives merchant a great control over Vipps payment lifecycle.
It gives possibility to initiate payment from mobile app and webshop. It also
enables merchant to affect payment flow by utilizing functions like payment
reservation, capture, cancellation and refunding.

## Use case scenarios
In order to ease integration with Vipps and get better understanding of API
functionality and usage some typical scenarios is presented.

## Regular eCommerce Payments

Merchants can utilize vipps payment services for ecommerce payments at their
online websites. Vipps APIs can be integrated at merchant’s checkout page where
Vipps can be represented as one of the payment methods for online purchases.

Payment initiation can happen on desktop browser, mobile browser as well as in
merchant’s mobile app. This latest version of API document explains how we have
simplified development on merchant side. Vipps takes responsibility of
identifying whether user is on mobile browser or desktop browser and whether
user has vipps app in device or not. This is going to be possible by
introduction of “Vipps landing page”. You can read more description about
initiation of Vipps ecommerce payment here. Introduction of landing page will
simplify merchant’s implementation of ecommerce APIs in general.

Vipps ecommerce APIs support both direct sale as well as reservation-capture logic in API implementation. Merchant needs to get this configured during enrollment. One vipps ecommerce product supports one of the two approaches (reservation-capture or direct sale).
Following are different API services that merchant can utilize as part of ecommerce payments:

* Initiate Payment – Used for initiating ecommerce payment
* Cancel Payment – Optional. Used for cancel payment reservation Capture Payment – Used for capture reserved payment
* Refund Payment – Optional. Used for refund payments
* Get Payment Details – Optional. Used to retrieve transaction history
* Get Order Status – Optional. Server side check that payment is confirmed

## Express Checkout Payments

In addition to regular ecommerce APIs, Vipps also supports optional express checkout payment service. Merchant needs to be onboarded for this additional service as it involves sharing personal information of user (including address), shipping details, etc.
Vipps follows GDPR compliance for making express checkouts possible for merchants. Vipps takes away the lengthy checkout process to simple steps in Vipps app which gives merchants higher conversion rates of online purchases.

Vipps can share personal information of Vipps users with merchant by taking user’s consent as per GDPR law during checkout process. Also, it will be possible to populate different delivery options to Vipps users during checkout within Vipps app.

Upon user’s final confirmation of payment, Vipps will share payment information, address and shipping method information and Vipps user’s personal information (optional) once payment is processed.

In addition to above mentioned ecommerce services, merchants can also implement the below services in order to use express checkout feature.

* Get shipping cost & method – Used for fetching shipping cost & method combination based on user selected address
* Transaction update with user details – separate callback to merchant for receiving post payment information
* Remove user consent – Used to inform merchant when vipps user removes consent to share his details.

# API calls flow

This section will explain how merchants can start using Vipps APIs and get access to API credentials.

During merchant’s onboarding process in Vipps, the merchant receives a username and a password to login into the Merchant Developer Portal (see the [Getting Started](vipps-ecom-api-getting-started.md) guide for how to  to use the Vipps Developer Portal). Once logged in to the developer portal merchant finds the API credentials needed to make API calls.

The diagram below shows the integration flow between merchant and Vipps server.

![API calls flow](images/api-calls-flow.png)

All communication with the Vipps ecommerce API has to be authenticated via JWT access token. To get this access token and use it in API calls merchant should follow the steps below:

* Merchant logs into the Developer portal and receives API credentials (ClientId and ClientSecret).

* Merchant application uses the clientid and clientsecret to get a JWT access token from APIM. JWT access token is a base 64 encoded string value that needs to be used as a bearer token in the request header.

* Merchant application will have to use this JWT access token and APIM subscription key along with other request parameters while calling a Vipps API.

* APIM validates the JWT access token and subscription key. If token is invalid it sends `401 Unauthorized` while if it is valid, request is forwarded to Vipps.

* Vipps process the request and produce corresponding response which is sent back to merchant application via APIM.

# Authentication

## Overview

Every API call is authenticated and authorized based on the application access token (JWT Bearer token) and APIM subscription key (Ocp-Apim-Subscription-Key). Following headers are required to be there in every API request to successfully authenticate every API call.

| Header Name | Header Value | Description |
| ----------- | ------------ | ----------- |
|  `Authorization` | `Bearer <JWT access token>` | Type: Authorization token. Value: Access token is obtained by registering merchant backend application in Merchant Developer Portal. |
| `Ocp-Apim-Subscription-Key` | Base 64 encoded string | Subscription key for eCommerce product. This can be found in User Profile page on Merchant developer portal after merchant account is created |

## Access Token

Access token API endpoint helps to get the JWT Bearer token that needs to be passed in every API request in the authorization header. Merchant application use the <ClientId> and <ClientSecret> to get a JWT access token. JWT access token is a base 64 encoded string value that must be aquire first before making any Vipps API calls.

| Header Name | Header Value | Description |
| ----------- | ------------ | ----------- |
| `client_id` | A GUID value | Client ID received when merchant registered the application |
| `client_secret` | Base 64 encoded string | Client Secret received when merchant registered the application |
| `Ocp-Apim-Subscription-Key` | Base 64 encoded string | Subscription key for eCommerce product. This can be found in User Profile page on Merchant developer portal after merchant account is created |

(_**Technical API specification isavailable in Swagger format: https://vippsas.github.io/vipps-ecom-api/ The details below is a high-level overview.**_)

### Request

```
POST https://<<hostname>>/accessToken/get
client_id: <ClientID>
client_secret: <ClientSecret>
Ocp-Apim-Subscription-Key: <Ocp-Apim-Subscription-Key>
```

### Response

```
HTTP 200 OK
{
  "token_type": "Bearer",
  "expires_in": "86398",
  "ext_expires_in": "0",
  "expires_on": "1495271273",
  "not_before": "1495184574",
  "resource": "00000002-0000-0000-c000-000000000000",
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni <continued>"
}
```

| Name | Value | Description |
| ----------- | ------------ | ----------- |
| `token_type` | String | It’s a bearer type token. When used the word ‘Bearer’ must be added before the token value |
| `expires_in` | Integer | Token expiry duration in seconds |
| `ext_expires_in` | Integer | Any additional expiry time. This is zero only. |
| `not_before` | Integer | Token creation time in epoch format |
| `resource` | GUID string | A common resource object that comes by default. Not used in token validation |
| `access_token` | Base 64 encoded string | The actual access token that needs to be used in request header |

Possible errors:

| HTTP response code | Error |
| ------------------ | ----- |
| `400 Bad Request` | Invalid `client_id` |
| `401 Unauthorized` | Invalid `client_secret` |
| `5XX` | Internal servder error |

Error reponse body:

```
{
  "error": "unauthorized_client",
  "error_description": "AADSTS70001: Application with identifier 'e9b6c99d-2442-4a5d-84a2- c53a807fe0c4' was not found in the directory testapivipps.no\r\nTrace ID: 3bc2b2a0-d9bb-4c2e-8367- 5633866f1300\r\nCorrelation ID: bb2f4093-70af-446a-a26d-ed8becca1a1a\r\nTimestamp: 2017-05-19 09:21:28Z",
  "error_codes": [ 70001
  ],
  "timestamp": "2017-05-19 09:21:28Z",
  "trace_id": "3bc2b2a0-d9bb-4c2e-8367-5633866f1300",
  "correlation_id": "bb2f4093-70af-446a-a26d-ed8becca1a1a"
}
```

See the Swagger documentation for more details: [POST:/accesstoken/get](https://vippsas.github.io/vipps-ecom-api/#/Authorization_Service/fetchAuthorizationTokenUsingPost)
