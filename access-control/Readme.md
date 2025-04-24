# Access Control

## Summary of Topics
1. [Unprotected admin functionality](#unprotected-admin-functionality)
2. [Unprotected admin functionality with unpredictable URL](#unprotected-admin-functionality-with-unpredictable-url)
3. [User role controlled by request parameter](#user-role-controlled-by-request-parameter)
4. [User role can be modified in user profile](#user-role-can-be-modified-in-user-profile)
5. [URL-based access control can be circumvented](#url-based-access-control-can-be-circumvented)
6. [Method-based access control can be circumvented](#method-based-access-control-can-be-circumvented)
7. [User ID controlled by request parameter](#user-id-controlled-by-request-parameter)
8. [User ID controlled by request parameter, with unpredictable user IDs](#user-id-controlled-by-request-parameter-with-unpredictable-user-ids)
9. [User ID controlled by request parameter with data leakage in redirect](#user-id-controlled-by-request-parameter-with-data-leakage-in-redirect)
10. [User ID controlled by request parameter with password disclosure](#user-id-controlled-by-request-parameter-with-password-disclosure)
11. [Insecure direct object references](#insecure-direct-object-references)
12. [Multi-step process with no access control on one step](#multi-step-process-with-no-access-control-on-one-step)
13. [Referer-based access control](#referer-based-access-control)

---

## Unprotected admin functionality
**Reference:** [PortSwigger: Unprotected Admin Functionality](https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality)

**Solution:**
- Go to the lab and view `robots.txt` by appending `/robots.txt` to the lab URL. 
- Notice that the `Disallow` line discloses the path to the admin panel.
- In the URL bar, replace `/robots.txt` with `/administrator-panel` to load the admin panel.
- Delete carlos.

---

## Unprotected admin functionality with unpredictable URL
**Reference:** [PortSwigger: Unprotected Admin Functionality with Unpredictable URL](https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality-with-unpredictable-url)

**Solution:**
- Review the lab home page's source using Burp Suite or your web browser's developer tools.
- Observe that it contains some JavaScript that discloses the URL of the admin panel.
- Load the admin panel and delete carlos.

---

## User role controlled by request parameter
**Reference:** [PortSwigger: User Role Controlled by Request Parameter](https://portswigger.net/web-security/access-control/lab-user-role-controlled-by-request-parameter)

**Solution:**
- Browse to `/admin` and observe that you can't access the admin panel.
- Browse to the login page.
- In Burp Proxy, turn interception on and enable response interception.
- Complete and submit the login page, and forward the resulting request in Burp.
- Observe that the response sets the cookie `Admin=false`. Change it to `Admin=true`.
- Load the admin panel and delete carlos.

---

## User role can be modified in user profile
**Reference:** [PortSwigger: User Role Can Be Modified in User Profile](https://portswigger.net/web-security/access-control/lab-user-role-can-be-modified-in-user-profile)

**Solution:**
- Log in using the supplied credentials and access your account page.
- Use the provided feature to update the email address associated with your account.
- Observe that the response contains your role ID.
- Send the email submission request to Burp Repeater, add `"roleid":2` into the JSON in the request body, and resend it.
- Observe that the response shows your roleid has changed to 2.
- Browse to `/admin` and delete carlos.

---

## URL-based access control can be circumvented
**Reference:** [PortSwigger: URL-based Access Control Can Be Circumvented](https://portswigger.net/web-security/access-control/lab-url-based-access-control-can-be-circumvented)

**Solution:**
- Try to load `/admin` and observe that you get blocked. Notice that the response is very plain, suggesting it may originate from a front-end system.
- Send the request to Burp Repeater. Change the URL in the request line to `/` and add the HTTP header `X-Original-URL: /invalid`. Observe that the application returns a "not found" response.
- Change the value of the `X-Original-URL` header to `/admin`. Observe that you can now access the admin page.
- To delete the user carlos, add `?username=carlos` to the real query string and change the `X-Original-URL` path to `/admin/delete`.

---

## Method-based access control can be circumvented
**Reference:** [PortSwigger: Method-Based Access Control Can Be Circumvented](https://portswigger.net/web-security/access-control/lab-method-based-access-control-can-be-circumvented)

**Solution:**
- Log in using the admin credentials.
- Browse to the admin panel, promote carlos, and send the HTTP request to Burp Repeater.
- Open a private/incognito browser window and log in with the non-admin credentials.
- Attempt to re-promote carlos with the non-admin user by copying that user's session cookie into the existing Burp Repeater request, and observe that the response says "Unauthorized".
- Change the method from POST to POSTX and observe that the response changes to "missing parameter".
- Convert the request to use the GET method by right-clicking and selecting "Change request method".
- Change the username parameter to your username and resend the request.

---

## User ID controlled by request parameter
**Reference:** [PortSwigger: User ID Controlled by Request Parameter](https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter)

**Solution:**
- Log in using the supplied credentials and go to your account page.
- Note that the URL contains your username in the "id" parameter.
- Send the request to Burp Repeater.
- Change the "id" parameter to carlos.
- Retrieve and submit the API key for carlos.

---


