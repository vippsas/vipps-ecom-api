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

![API calls flow](images/api-call-flow.png)

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

### Overview

Access token API endpoint helps to get the JWT Bearer token that needs to be passed in every API request in the authorization header. Merchant application use the <ClientId> and <ClientSecret> to get a JWT access token. JWT access token is a base 64 encoded string value that must be aquire first before making any Vipps api calls.

### URL

`https://<<hostname>>/accessToken/get`

### Method

`POST`

###  Header Name

Client_secret

(_**Technical API specificatrion should be in Swagger format: https://vippsas.github.io/vipps-ecom-api/ Awaiting decision before continuing this document.**_)
