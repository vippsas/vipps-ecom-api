# VIPPS DEVELOPER PORTAL

This are the steps after you have received onboarding-email from vipps. You should have received proper credentials with username on email and password on the admin-phonenumber. Use those credentials to log into Vipps Developer Portal in either test or production.

# Step 1
We start with the typical Sign-in screen.

![Sign on screen](https://github.com/vippsas/vipps-ecom-api/blob/master/Vipps_Developer_Portal_SamplePictures/Vipps_sign_in.PNG?raw=true "Title")

You type in your username and password here.
Now remember that there's a difference in both production and test when it comes to Vipps Developer Portal.

Link for Vipps Developer in test: https://apitest-portal.vipps.no/
- You will be given a username that looks like this: username@testapivipps.no
- And a default password. For password-change please contact integration@vipps.no

Link for Vipps Developer in production: https://api-portal.vipps.no/
- You will be given a username that looks like this:
username@apivipps.no
- And a default password. For password-change please contact integration@vipps.no

**IN CASE OF ERROR-SCREEN SHOWN ON THE PICTURE UNDER:**

![Error Screen](https://github.com/vippsas/vipps-ecom-api/blob/master/Vipps_Developer_Portal_SamplePictures/Error-Screen.PNG?raw=true "Title")

- Make sure that you are also logged out of other 365-accounts
- Make sure you are in incognito-modus in either Google Chrome or Windows Edge
- Make sure you are using the correct url. Remember https://apitest-portal.vipps.no/ for test, and https://api-portal.vipps.no/ for production.

# Step 2
After an successfull log-in you will see the admin-name up in the right corner of the screen. In the left corner you have several tabs.
The **"Manage User"** - gives you the possibility to add users.

![Add users](https://github.com/vippsas/vipps-ecom-api/blob/master/Vipps_Developer_Portal_SamplePictures/add_user_vipps_developer_portal.PNG?raw=true "Title")

# Step 3
The next tab **"Products"** shows you the API's you current have. As you see from the picture below you can have several products and the possibility to test them out in Vipps Developer Portal.

![Products](https://github.com/vippsas/vipps-ecom-api/blob/master/Vipps_Developer_Portal_SamplePictures/products_vipps_dev.PNG?raw=true "Title")

Click on the "Test the Api(s)"-button to move further.
![Test the Api(s)](https://github.com/vippsas/vipps-ecom-api/blob/master/Vipps_Developer_Portal_SamplePictures/Test_the_api.PNG?raw=true "Title")

Click on the "Try it"-button to move further.
![Try it](https://github.com/vippsas/vipps-ecom-api/blob/master/Vipps_Developer_Portal_SamplePictures/Try_it_out.PNG?raw=true "Title")

Add the the proper keys to iniate your request.
![Request Payment](https://github.com/vippsas/vipps-ecom-api/blob/master/Vipps_Developer_Portal_SamplePictures/Request_payment.PNG?raw=true "Title")

# Step 4
Now you probably wonder where you should get the API keys. Well, if you check out the tab **"Applications"** and click on the unit with the number-identificator that fits your salesunit, then you will find both client_id and client_secret.

![keys Applications](https://github.com/vippsas/vipps-ecom-api/blob/master/Vipps_Developer_Portal_SamplePictures/keys_application.PNG?raw=true "Title")

Under the tab **"register application"** it should say(marked in red):

'All existing products have been subscribed'

Under the tab **"Profile"** you will find your two last keys.

![keys_profile](https://github.com/vippsas/vipps-ecom-api/blob/master/Vipps_Developer_Portal_SamplePictures/keys_profile.PNG?raw=true "Title")

You have the Access Token key on the top. You have to hit "show" on the right side to make the keys appear. And as marked, its the primary key you are after. If the key for some reason does not work, then you can hit "regenerate", right next to "show".

The Ocp-Apim-Subscription-Key is right below the Access Token key. You have to hit "show" on the right side to make the keys appear. And as marked, its the primary key you are after. If the key for some reason does not work, then you can hit "regenerate", right next to "show".

The MerchantSerialNumber is the number right next to the name of your Salesunit. Now you should have both the Client_id, Client_secret, Access_token key, Ocp-Apim-Subscription-Key and SerialMerchantNumber and can proceed with the payment-flow.
