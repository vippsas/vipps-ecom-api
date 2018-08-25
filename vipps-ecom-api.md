# Vipps eCommerce API

API version: 2.1.

Document version 0.1.1.

_**IMPORTANT:**_ This document is a work in progress. See the official PDF documentation:
https://github.com/vippsas/vipps-ecom-api/tree/master/original-docs
See also the [FAQ](vipps-ecom-api-faq.md).

Test:
<img src="https://share.pingdom.com/banners/f626772f" 
alt="Uptime Report for Vipps: API: eCommerce: Last 30 days"
title="Uptime Report for Vipps: API: eCommerce: Last 30 days"
width="300" height="165" />

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

Merchants can utilize Vipps payment services for ecommerce payments at their
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

#### Errors

| HTTP response code | Error |
| ------------------ | ----- |
| `400 Bad Request` | Invalid `client_id` |
| `401 Unauthorized` | Invalid `client_secret` |
| `5XX` | Internal server error |

Error reponse body (formatted for readability):

```
{
  "error": "unauthorized_client",
  "error_description": "AADSTS70001: Application with identifier 'e9b6c99d-2442-4a5d-84a2-\
                        c53a807fe0c4' was not found in the directory testapivipps.no\
                        Trace ID: 3bc2b2a0-d9bb-4c2e-8367- 5633866f1300\r\nCorrelation ID:\
                        bb2f4093-70af-446a-a26d-ed8becca1a1a\r\nTimestamp: 2017-05-19 09:21:28Z",
  "error_codes": [ 70001
  ],
  "timestamp": "2017-05-19 09:21:28Z",
  "trace_id": "3bc2b2a0-d9bb-4c2e-8367-5633866f1300",
  "correlation_id": "bb2f4093-70af-446a-a26d-ed8becca1a1a"
}
```

See the Swagger documentation for more details: [POST:/accesstoken/get](https://vippsas.github.io/vipps-ecom-api/#/Authorization_Service/fetchAuthorizationTokenUsingPost)

# Idempotency

All API requests in Vipps eCommerce can be retried without any side effects by
providing idempotent key in a header of the request. For example, in case the
request fails because of network error it can safely be retried with the same
idempotent key. Idempotent key is generated by merchant.

`-H "X-Request-Id: slvnwdcweofjwefweklfwelf"`

# eCommerce Payment Flows

Payment flow in Vipps eCommerce is represented by following diagrams:

## Flow for push notification flow on desktop browser

![API calls flow: Push for desktop](images/api-calls-flow-push-desktop.png)

## Flow for push notification flow on mobile

![API calls flow: Push for mobile](images/api-calls-flow-push-mobile.png)

## Initiate

The first call in the payment flow initiates a payment request that requires confirmation from the customer (end user). Payment has status Initiated and customer is notified about payment request in mobile app. If customer doesn’t confirm, the payment request cancels and payment flow aborts. Initiate call will have parameter paymentType which will identify regular ecommerce payment and express checkout payment flow.

API details: [`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/initiatePaymentV3UsingPOST)

### Request

```
{
  "customerInfo": {
    "mobileNumber": "Values with length 8"
  },
  "merchantInfo": {
    "authToken": "Values with length 255",
    "callbackPrefix": "Values with length 255",
    "consentRemovalPrefix": "Values with length 255",
    "fallBack": "Values with length 255",
    "isApp": false,
    "merchantSerialNumber": "NSBWSHP12",
    "paymentType": "string",
    "shippingDetailsPrefix": "Values with length 255"
  },
  "transaction": {
    "amount": 0,
    "orderId": "Values with length 30",
    "refOrderId": "Values with length 30",
    "timeStamp": "2018-08-25T11:46:25.386Z",
    "transactionText": "Values with length 100"
  }
}
```

### Response

```
HTTP 202 Accepted
{
  "orderId": "string",
  "url": "string"
}
```

## Reserve

When the customer successfully authorizes the payment request by using Vipps app, the payment status changes to Reserved, and the respective amount will be reserved for future capturing at the PSP.

API details: [`GET:`]()

## Cancel

Reservations can be cancelled and payment flow aborted under certain circumstances:

* When user cancels the initiated payment then the payment status shown will be cancelled

* When Merchant call cancellation of the reservation. Please note that partially captured reservations can’t be cancelled.

* When there is a timeout of the payment confirmation by several of reasons (no action by the customer, notification to user is delayed, etc.)

* When reservation fails caused by system or communication error.

API details: [`PUT:/ecomm/v2/payments/{orderId}/cancel`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/cancelPaymentRequestUsingPUT)

## Capture

When merchant shipped the goods then they can call capture API on the reserved transaction. The API allows to do a full amount capture or partial amount capture.

API details: [`GET:`]()

## Direct capture

Direct capture is not depicted on diagram above but, in essence, combines two steps (reserve and capture) in one. This is a configuration in Vipps backend when Initiate payment request is sent.

API details: [`GET:`]()

## Refund

Merchant can initiate a refund of the captured amount. The refund can be a partial or full.

API details: [`GET:`]()

## Get Order Status

Get Order Status intention is to check whether the user is authenticated the transaction or not. Possible status provided by this service is listed below:

API details: [`GET:/ecomm/v2/payments/{orderId}/status`](https://vippsas.github.io/vipps-ecom-api/#/oneclick-payment-with-vipps-controller/getOrderStatusUsingGET)

| Status | Description |
| ------ | ----------- |
| INITIATE | Initiate Payment |
| REGISTER | When we call PSP for payment |
| RESERVE | Parent Reserve Payment and for its child Capture\Refund payments |
| SALE | Direct Capture |
| CANCEL | When payment has been cancelled |
| VOID | Registration Cancel |
| AUTOREVERSAL | Transaction timed out at Nets, we will refund the transaction and then it will be autoreversal |
| AUTOCANCEL | When no action from user for notification for X minutes. |
| FAILED | When transaction failed to execute |
| REJECTED | When user reject payment request in Vipps app |

# Additional payment flow for express checkout

In addition to above mentioned payment flows, following are the services which merchants should build on their side to support express checkout during online purchases.

## Get shipping cost & method

When express checkout payment is initiated, vipps will call this service from merchant’s backend to fetch shipping cost and shipping method related details. Merchant can send priority of shipping cost and method combination if there are multiple ways of delivery. Merchant can also send default shipping cost & method combination which merchant wants user to see on payment confirmation screen of Vipps. Vipps will support upto 10 shipping cost and method combinations. If user sends more than 10 combinations, vipps will display first 10 always.

## Transaction updates with user details

After express checkout payment is processed, vipps will make a call back to merchant stating payment details, shipping details and user details (optional).

## Remove user consent

When consent to store/process/view details of vipps user is removed by user in vipps app, vipps will make a call to merchant informing the same. Merchant is obliged to delete user information upon receiving this request.

## Exception handling 7.1 Introduction

Every system, especially those that includes complex integrations and/or participation of many users, is prone to unexpected conditions. Below section explains how Vipps handles different exception and error situations in detail.

## Exception scenarios

The most critical action in payment flow is when Initiate Payment service call is invoked. The Flow diagram below shows how to successfully fulfil service call, communication between several contributors and users across several systems has to work flawlessly.

![Payment flow eCommerce – Payment request](images/payment-flow-for-ecommerce-payment-request.png)

To cope with possible communication problems/errors, several scenarios and guidelines are developed.

### Connection timeout

Defining a socket timeout period is the common measure to protect server resources and is expected. However, the time needed to fulfill a service requests depends on several systems, which impose longer timeout period than usually required. We recommend setting no less than 1 second socket connection timeout and 5 seconds socket read timeout while communicating with Vipps.
A good practice is, if/when the socket read timeout occurs call Get Payment Details and check status of last transaction in transaction history prior executing the service call again.

### Callback aborted/interrupted

If the communication is broken during payment process for some reason, and Vipps is not able to execute callback, then callback will not be retried.
In other words, if the merchant doesn’t receive any confirmation on payment request call within callback timeframe, merchant should call get payment details service to get the response of payment request.

### PSP connection issues

In a case when Vipps experiences communication problems with PSP, service call will respond with 402 HTTP Error. Merchant should make a call to Get Payment Details to check if the transaction request is processed before making service call (with same idempotency key) again.

# Response codes

Vipps eCommerce API uses standard HTTP response codes to indicate the success and failure
of the request as defined in [RFC2616](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html).

(_**INTERNAL NOTE: This section should refer to Swagger spec**_)

Response codes with range 2xx indicates success, 4xx indicates an error because (validation error, Reservation of transaction failed etc.), 5xx are the vipps internal errors.

## Success Codes

```
200 – OK
202 - Accepted
```

## HTTP Error Codes

```
400 – Bad request (Missing a required parameter or Bad request formats)
401 – Unauthorized Vipps
403 – Forbidden
404 – Resource Not Found
405 – Request method not supported
415 – Unsuppoted media type
5XX – Something went wrong from Vipps Server side
```

In the case of error, body of response contains detailed information about the error condition. Error object is represented in JSON format as:

```
[{
"errorCode": "", "errorMessage": ""
}]
```

## Error Representation

(_**INTERNAL NOTE: Add table**_)

## Error codes

| Group | Code | Description |
| ----- | ---- | ----------- |
| Payment | 41 | User don’t have a valid card |
| Payment | 42 | Refused by issuer bank |
| Payment | 43 | Refused by issuer bank because of invalid a amount |
| Payment | 44 | Refused by issuer because of expired card |
| Payment | 45 | Reservation failed for some unknown reason |
| Payment | 51 | Can't cancel already captured order |
| Payment | 52 | Cancellation failed |
| Payment | 53 | Can’t cancel order which is not reserved yet |
| Payment | 61 | Captured amount exceeds the reserved amount ordered |
| Payment | 62 | Can't capture cancelled order |
| Payment | 63 | Capture failed for some unknown reason, please use Get Payment Details API to know the exact status |
| Payment | 71 | Cant refund more than captured amount |
| Payment | 72 | Cant refund for reserved order, please use Cancel API |
| Payment | 73 | Can't refund on cancelled order |
| InvalidRequest  | Field name will be the error code | Description about what exactly the field error is |
| VippsError | 91 | Transaction is not allowed |
| VippsError | 92 | Transaction already processed |
| VippsError | 98 | Too many concurrent requests |
| VippsError | 99 | Description of the internal error |
| Customer | 81 | User not registered with Vipps |
| Customer | 82 | User App version not supported |
| Merchant | 31 | Merchant is blocked because of <reason> |
| Merchant | 32 | Receiving limit of merchant is exceeded |
| Merchant | 33 | Number of payment requests has been exceeded |
| Merchant | 34 | Unique constraint violation of the order id |
| Merchant | 35 | Registered order not found |
| Merchant | 36 | Merchant agreement not signed |
| Merchant | 37 | Merchant not available, deactivated or blocked |
| Merchant | 21 | Reference Order ID is not valid |
| Merchant | 22 | Reference Order ID is not in valid state |

# Front-end Integration

Merchants need to implement “Appswitch” integration which is also called as “Deeplinking” to trigger Vipps app for serving ecommerce payment requests. This can happen in two ways:

1. From mobile or desktop browser
2. From mobile application

Below section explains how merchant can implement these integrations in detail.

## Appswitch between Mobile or Desktop Browser and Vipps App

For mobile/desktop browser to Vipps front end integration, merchant doesn’t need to do any special activity.
Front end integration will be handled by Vipps using Vipps landing page.

Merchant needs to ensure passing correct “fallbackURL” in Vipps backend API (explained here).
After vipps has completed the operation the “fallbackURL” will be opened in a new tab/new window in the browser.
To maintain the session, merchant can pass along a session identifier through “fallbackURL”.

## Appswitch between Merchant’s Mobile App and Vipps App

App to App switch is supported by both Vipps applications on iOS and Android platform.
The two subsections below explain how appswitch happens for iOS and Android respectively.

### Appswitch with iOS platform

Following section explains how appswitch will happen for Vipps app on iOS platform.

#### Overview for iOS

* Vipps app on iOS platform requires URL scheme in order to support appswitch.
* Merchant need to pass the URI Scheme of app into “fallbackURL” in Vipps backend API
(explained here).
* Merchant will open the url received from Vipps backend API.
* Once the operation in Vipps is completed, Vipps will open the url mentioned in “fallbackURL”.
* From vipps mobile application appropriate status code will be appended with “fallbackURL”.

#### Switch from Source App to iOS Vipps App

Below is sample code to open iOS Vipps application with deeplinkURL.

```java
NSString *url = deeplinkURL; //Use deeplink url provided in API response
if ([[UIApplication sharedApplication] canOpenURL:[NSURL URLWithString:url]]) {
  [[UIApplication sharedApplication] openURL:[NSURL URLWithString:url]]; }
else {
  // Oops no Vipps app or update to latest Vipps App! Open app store page.
  // Once user installs Vipps, calling app needs to initiate deeplinking again in order to get the callback
  [[UIApplication sharedApplication] openURL:[NSURL URLWithString: @"https://itunes.apple.com/no/app/Vipps-by-dnb/id984380185"]];
}
```

#### Redirect Back to Source App from iOS Vipps App

Once the operation in Vipps is completed, vipps mobile application will open the frontend url. For app to app integration, merchant app needs to be registered for a url scheme and pass the url scheme in “fallbackURL” in Vipps backend API (explained here). Vipps mobile application will use below code to launch merchant application.

```java
NSString *fallbackURL = self.fallbackURL; //fallback url will be the url which has been provided in Vipps API.

NSURLComponents *urlComponents = [NSURLComponents componentsWithString:fallbackURL];
NSMutableArray <NSURLQueryItem *>*queryItems = [[NSMutableArray alloc]initWithArray:urlComponents.queryItems];
NSURLQueryItem *statusQueryItem = [NSURLQueryItem queryItemWithName:@"status" value:@"301"];

//Add the queryitem in the “queryItems” array.
[queryItems addObject:statusQueryItem];
NSURL * fallbackURL = urlComponents.URL;//after adding the new queryItems we will get the new fallbackURL

// navigating back to source application
UIApplication *application = [UIApplication sharedApplication]; if([application canOpenURL:fallbackURL]){
[application openURL:fallbackURL]; }
```

For example, if your fallback URL is `testApp://result?myAppData` then Vipps will reply with `testApp://result?myAppData&status=301`.

##### Registering 3rd Party app with URL Scheme and handling custom URL Calls

Defining your app's custom URL scheme is all done in the Info.plist file.
Click on the last line in the file and then click the "+" sign off to the
right to add a new line. Select URL Types for the new item.

Once that's added, click the grey arrow next to "URL Types" to show "Item 0".

Set your URL identifier to a unique string - something like com.yourcompany.yourappname.
After you've set the URL identifier, select that line and click the "+" sign again,
and add a new item for URL Schemes. Then click the grey arrow next to
"URL Schemes" to reveal "Item 0". Set the value for Item 0 to be your URL scheme name.

![Registering 3rd Party app](images/registering-3rd-party-app.png)

In order for your app to respond when it receives a custom URL call, you must implement the application:handleOpenURL method in the application delegate class:

```java
- (BOOL)application:(UIApplication *)application handleOpenURL:(NSURL *)url {
  // handler your code here
  NSURLComponents *urlComponents = [NSURLComponents componentsWithString:baseURL];
  NSMutableArray <NSURLQueryItem *>*queryItems = urlComponents.queryItems; //fetch the value of a particular paramerter from queryItems array.
}
```

### Appswitch with Android platform

Following section explains how appswitch will happen for Vipps app on Android platform.

#### Overview for Android

Vipps Android platform supports two ways of appswitch integration:

* Use “startActivityForResult”: In order to use this the merchant need to set a “fallbackURL” as “INTENT”. In this way of communication there is no need to register for Url scheme.

* Use URL scheme: It is similar to the way it is solved for iOS. First the app needs to be registered for URL scheme and then pass the URL scheme in “fallbackURL”.

#### Switch from Source App to Android Vipps App

3rd party applications can integrate with Vipps by taking use of one of the following two approaches

##### Android Intent

In case of Android Intent system, in backend API call(defined later) “INTENT”
should be passed in fallbackURL. And below code should be used to launch Vipps application.

```java
try {
  PackageManager pm = context.getPackageManager();
  PackageInfo info = pm.getPackageInfo( , PackageManager.GET_ACTIVITIES);
  if(versionCompare(info.versionName,   ) >= 0) {
    String uri = deeplinkURL; // Use deeplink url provided in API response
    Intent intent = new Intent(Intent.ACTION_VIEW);
    intent.setData(Uri.parse(uri));
    startActivityForResult(intent,requestCode);
  } else {
    // Notify user to download the latest version of Vipps application.
  }
} catch (PackageManager.NameNotFoundException e) {
  // No Vipps app! Open play store page.
  String url = " https://play.google.com/store/apps/details?id=no.dnb.vipps";
  Intent storeIntent = new Intent(Intent.ACTION_VIEW);
  storeIntent.setData(Uri.parse(url));
  startActivity(storeIntent);
}
```

##### Android URL Scheme

Following is the code sample for Android URL scheme approach.

```java
try {
  PackageManager pm = context.getPackageManager();
  PackageInfo info = pm.getPackageInfo , PackageManager.GET_ACTIVITIES);
  if (versionCompare(info.versionName,   ) >= 0) {
    String uri = deeplinkURL; // Use deeplink url provided in API response
    Intent intent = new Intent(Intent.ACTION_VIEW);
    intent.setData(Uri.parse(uri));
    startActivity(intent);
  } else {
    // Notify user to download the latest version of Vipps application.
  }
} catch (PackageManager.NameNotFoundException e) {
  // No Vipps app! Open play store page.
  String url = " https://play.google.com/store/apps/details?id=no.dnb.vipps";
  Intent storeIntent = new Intent(Intent.ACTION_VIEW);
  storeIntent.setData(Uri.parse(url));
  startActivity(storeIntent);
}
```

#### Redirect Back to Source App from Android Vipps App

Android supports two ways of redirecting back to source app and merchant should use the correct
method to open Vipps and get redirected back to merchant’s source app.

Below are two ways of redirecting back to source app from Android Vipps app:

##### Android Intent

Register the activity in manifest file which will handle result of Vipps response. For Example:

```html
<activity
  android:name=".MainActivity"
  android:label="@string/app_name">
</activity>
```

Receiving activity has to override onActivityResult method to handle result sent by Vipps application.
For Example:

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
  if (resultCode == RESULT_OK) {
    if (requestCode == 1) {
      String url = null;
      if (data != null && data.getExtras() != null) {
        Bundle mBundle = data.getExtras();
        if (mBundle.get("data") != null) {
          try {
            url = URLDecoder.decode(mBundle.get("data").toString(), "UTF-8"); Uri parseUri = Uri.parse(url);
            String status = parseUri.getQueryParameter("status");
            //TODO Handle status
          } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
          }
        }
      }
    }
  }
}
```

##### Android URL Scheme
Vipps provides a custom URL scheme to interact with Vipps. If 3rd party application wants to open Vipps application with custom URL scheme then they can implement this approach.

Set filter in Manifest file: To receive a call back from the Vipps application to an activity there has to be set a filter to that activity. In the example below MainActivity is the receiving activity and Vipps application sends a response to the activity. For this activity one can set a custom URL scheme inside the intent filter. For Example:

```html
<activity android:name=".MainActivity" android:label="@string/app_name" android:launchMode="singleInstance">
<intent-filter>
              <action android:name="android.intent.action.VIEW" />
              <category android:name="android.intent.category.DEFAULT" />
              <category android:name="android.intent.category.BROWSABLE" />
              <data android:scheme="sampleApps" />
       </intent-filter>
</activity>
```

Note: scheme should be same as you send in fallbackURL parameter in Vipps API

Vipps application will send the result to the 3rd party application by starting a new activity with the fallbackURL as a URI parameter in the intent. The 3rd party application can make their receiving activity as a singleInstance to handle the response in same activity.

The receiving activity has to override onNewIntent method to handle result send by Vipps application.

```java
@Override
protected void onNewIntent(Intent intent) {
  super.onNewIntent(intent);
  String url = null;
  if (intent != null && intent.getData() != null) {
    try {
      url = URLDecoder.decode(intent.getData().toString(),"UTF-8");
      Uri parseUri = Uri.parse(url);
      String status = parseUri.getQueryParameter("status");
      //TODO Handle status
    } catch(UnsupportedEncodingException e) {
      e.printStackTrace();
    }
  }
}
```

## List of error codes for deeplinking

Following are the identified status codes merchant may receive from Vipps app.

(_**TODO: Insert table from page 20**:_)

Below are the status code ranges which Vipps maintains for future purposes. For example, if there is new error message related to fraud, then it will fall under range 450 to 499.

(_**TODO: Formatting**:_)

```
1XX – Success Scenarios
200 to 250 – Input Error
250 to 299 - User Actions
3XX – Authentication / User Profile / Merchant Profile / Configuration related error
400 to 450 – Transaction related error
450 to 499 – Fraud related error
5XX – Reserved for future use 6XX – Reserved for future use
7XX – Reserved for future use 8XX – Reserved for future use
9XX – Others
```
# API definitions

Below section explains different API definitions supported by Vipps ecommerce APIs.

(_**TODO: Continue with page 20+**:_)
