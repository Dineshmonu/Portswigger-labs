# Access Control

## Summary of Topics
- Unprotected admin functionality
- Unprotected admin functionality with unpredictable URL
- User role controlled by request parameter
- User role can be modified in user profile
- URL-based access control can be circumvented
- Method-based access control can be circumvented
- User ID controlled by request parameter
- User ID controlled by request parameter, with unpredictable user IDs
- User ID controlled by request parameter with data leakage in redirect
- User ID controlled by request parameter with password disclosure
- Insecure direct object references
- Multi-step process with no access control on one step
- Referer-based access control

## Unprotected Admin Functionality
### Reference: [PortSwigger: Unprotected Admin Functionality](https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality)

**Solution:**
1. Go to the lab and view `robots.txt` by appending `/robots.txt` to the lab URL.
2. Notice that the `Disallow` line discloses the path to the admin panel.
3. In the URL bar, replace `/robots.txt` with `/administrator-panel` to load the admin panel.
4. Delete carlos.

## Unprotected Admin Functionality with Unpredictable URL
### Reference: [PortSwigger: Unprotected Admin Functionality with Unpredictable URL](https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality-with-unpredictable-url)

**Solution:**
1. Review the lab home page's source using Burp Suite or your web browser's developer tools.
2. Observe that it contains some JavaScript that discloses the URL of the admin panel.
3. Load the admin panel and delete carlos.

## User Role Controlled by Request Parameter
### Reference: [PortSwigger: User Role Controlled by Request Parameter](https://portswigger.net/web-security/access-control/lab-user-role-controlled-by-request-parameter)

**Solution:**
1. Browse to `/admin` and observe that you can't access the admin panel.
2. Browse to the login page.
3. In Burp Proxy, turn interception on and enable response interception.
4. Complete and submit the login page, and forward the resulting request in Burp.
5. Observe that the response sets the cookie `Admin=false`. Change it to `Admin=true`.
6. Load the admin panel and delete carlos.

## User Role Can Be Modified in User Profile
### Reference: [PortSwigger: User Role Can Be Modified in User Profile](https://portswigger.net/web-security/access-control/lab-user-role-can-be-modified-in-user-profile)

**Solution:**
1. Log in using the supplied credentials and access your account page.
2. Use the provided feature to update the email address associated with your account.
3. Observe that the response contains your role ID.
4. Send the email submission request to Burp Repeater, add `"roleid":2` into the JSON in the request body, and resend it.
5. Observe that the response shows your `roleid` has changed to 2.
6. Browse to `/admin` and delete carlos.

## URL-Based Access Control Can Be Circumvented
### Reference: [PortSwigger: URL-based Access Control Can Be Circumvented](https://portswigger.net/web-security/access-control/lab-url-based-access-control-can-be-circumvented)

**Solution:**
1. Try to load `/admin` and observe that you get blocked. Notice that the response is very plain, suggesting it may originate from a front-end system.
2. Send the request to Burp Repeater. Change the URL in the request line to `/` and add the HTTP header `X-Original-URL: /invalid`. Observe that the application returns a "not found" response.
3. Change the value of the `X-Original-URL` header to `/admin`. Observe that you can now access the admin page.
4. To delete the user carlos, add `?username=carlos` to the real query string and change the `X-Original-URL` path to `/admin/delete`.

## Method-Based Access Control Can Be Circumvented
### Reference: [PortSwigger: Method-Based Access Control Can Be Circumvented](https://portswigger.net/web-security/access-control/lab-method-based-access-control-can-be-circumvented)

**Solution:**
1. Log in using the admin credentials.
2. Browse to the admin panel, promote carlos, and send the HTTP request to Burp Repeater.
3. Open a private/incognito browser window and log in with the non-admin credentials.
4. Attempt to re-promote carlos with the non-admin user by copying that user's session cookie into the existing Burp Repeater request, and observe that the response says "Unauthorized".
5. Change the method from POST to POSTX and observe that the response changes to "missing parameter".
6. Convert the request to use the GET method by right-clicking and selecting "Change request method".
7. Change the username parameter to your username and resend the request.

## User ID Controlled by Request Parameter
### Reference: [PortSwigger: User ID Controlled by Request Parameter](https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter)

**Solution:**
1. Log in using the supplied credentials and go to your account page.
2. Note that the URL contains your username in the "id" parameter.
3. Send the request to Burp Repeater.
4. Change the "id" parameter to carlos.
5. Retrieve and submit the API key for carlos.
