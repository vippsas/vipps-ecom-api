This are the steps after you have received onboarding-email from vipps. You should have received proper credentials with username on email and password on the admin-phonenumber. Use those credentials to log into Vipps Developer Portal in either test or production.

We start with the typical Sign-in screen.

![Alt text](relative/path/to/img.jpg?raw=true "Title") Sign on screen

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

![Alt text](relative/path/to/img.jpg?raw=true "Title") Error-screen

- Make shure that you are also logged out of other 365-accounts
- Make shure you are in incognito-modus in either Google Chrome or Windows Edge
- Make shure you are using the correct url. Remember https://apitest-portal.vipps.no/ for test, and https://api-portal.vipps.no/ for production.

After an successfull log-in you will see the admin-name up in the right corner of the screen. In the left corner you have several tabs.
The **"Manage User"** - gives you the possibility to add users.

![Alt text](relative/path/to/img.jpg?raw=true "Title") Add users

The next tab **"Products"** shows you the API's you current have. As you see from the picture below you can have several products and the possibility to test them out in Vipps Developer Portal.

![Alt text](relative/path/to/img.jpg?raw=true "Title") Products

![Alt text](relative/path/to/img.jpg?raw=true "Title") Test the Api(s)

![Alt text](relative/path/to/img.jpg?raw=true "Title") Try it

![Alt text](relative/path/to/img.jpg?raw=true "Title") request payment

Now you probably wonder where you should get the API keys. Well, if you check out the tab **"Applications"** and click on the unit with the number-identificator that fits your salesunit, then you will find both client_id and client_secret.

![Alt text](relative/path/to/img.jpg?raw=true "Title") keys Applications

Under the tab **"register application"** it should say(marked in red):

'All existing products have been subscribed'

Under the tab **"Profile"**
