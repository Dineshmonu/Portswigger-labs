<!-- omit in toc -->
# DOM-based

<!-- omit in toc -->
## Table of Contents

- [DOM XSS using web messages](#dom-xss-using-web-messages)
- [DOM XSS using web messages and a JavaScript URL](#dom-xss-using-web-messages-and-a-javascript-url)
- [DOM XSS using web messages and JSON.parse](#dom-xss-using-web-messages-and-jsonparse)
- [DOM-based open redirection](#dom-based-open-redirection)
- [DOM-based cookie manipulation](#dom-based-cookie-manipulation)

## DOM XSS using web messages
### Reference: [PortSwigger: DOM XSS using web messages](https://portswigger.net/web-security/dom-based/controlling-the-web-message-source/lab-dom-xss-using-web-messages)

<!-- omit in toc -->
### Solution
1. Notice that the home page contains an ``addEventListener()`` call that listens for a web message.
2. Go to the exploit server and add the following iframe to the body. Remember to add your own lab ID:
```html
<iframe src="https://your-lab-id.web-security-academy.net/" onload="this.contentWindow.postMessage('<img src=1 onerror=print()>','*')">
```
3. Store the exploit and deliver it to the victim.
When the iframe loads, the ``postMessage()`` method sends a web message to the home page. The event listener, which is intended to serve ads, takes the content of the web message and inserts it into the ``div`` with the ID ``ads``. However, in this case it inserts our ``img`` tag, which contains an invalid ``src`` attribute. This throws an error, which causes the ``onerror`` event handler to execute our payload.

## DOM XSS using web messages and a JavaScript URL
### Reference: [PortSwigger: DOM XSS using web messages and a JavaScript URL](https://portswigger.net/web-security/dom-based/controlling-the-web-message-source/lab-dom-xss-using-web-messages-and-a-javascript-url)

<!-- omit in toc -->
### Solution
1. Notice that the home page contains an ``addEventListener()`` call that listens for a web message. The JavaScript contains a flawed ``indexOf()`` check that looks for the strings "``http:``" or "``https:``" anywhere within the web message. It also contains the sink ``location.href``.
2. Go to the exploit server and add the following ``iframe`` to the body, remembering to replace ``your-lab-id`` with your lab ID:
```html
<iframe src="https://your-lab-id.web-security-academy.net/" onload="this.contentWindow.postMessage('javascript:print()//http:','*')">
```
3. Store the exploit and deliver it to the victim.
This script sends a web message containing an arbitrary JavaScript payload, along with the string "``http:``". The second argument specifies that any ``targetOrigin`` is allowed for the web message.

When the iframe loads, the ``postMessage()`` method sends the JavaScript payload to the main page. The event listener spots the ``"http:``" string and proceeds to send the payload to the ``location.href`` sink, where the ``print()`` function is called.

## DOM XSS using web messages and JSON.parse
### Reference: [PortSwigger: DOM XSS using web messages and JSON.parse](https://portswigger.net/web-security/dom-based/controlling-the-web-message-source/lab-dom-xss-using-web-messages-and-json-parse)

<!-- omit in toc -->
### Solution
1. Notice that the home page contains an event listener that listens for a web message. This event listener expects a string that is parsed using ``JSON.parse()``. In the JavaScript, we can see that the event listener expects a ``type`` property and that the ``load-channel`` case of the ``switch`` statement changes the ``iframe src`` attribute.
2. Go to the exploit server and add the following ``iframe`` to the body, remembering to replace ``your-lab-id`` with your lab ID:
```html
<iframe src=https://your-lab-id.web-security-academy.net/ onload='this.contentWindow.postMessage("{\"type\":\"load-channel\",\"url\":\"javascript:print()\"}","*")'>
```
3. Store the exploit and deliver it to the victim.
   
When the iframe we constructed loads, the ``postMessage()`` method sends a web message to the home page with the type ``load-channel``. The event listener receives the message and parses it using ``JSON.parse()`` before sending it to the switch.

The switch triggers the ``load-channel`` case, which assigns the ``url`` property of the message to the ``src`` attribute of the ``ACMEplayer.element iframe``. However, in this case, the ``url`` property of the message actually contains our JavaScript payload.

As the second argument specifies that any ``targetOrigin`` is allowed for the web message, and the event handler does not contain any form of origin check, the payload is set as the ``src`` of the ``ACMEplayer.element iframe``. The ``print()`` function is called when the victim loads the page in their browser.

## DOM-based open redirection
### Reference: [PortSwigger: DOM-based open redirection](https://portswigger.net/web-security/dom-based/open-redirection/lab-dom-open-redirection)

<!-- omit in toc -->
### Solution
The blog post page contains the following link, which returns to the home page of the blog:
```html
<a href='#' onclick='returnURL' = /url=https?:\/\/.+)/.exec(location); if(returnUrl)location.href = returnUrl[1];else location.href = "/"'>Back to Blog</a>
```
The ``url`` parameter contains an open redirection vulnerability that allows you to change where the "Back to Blog" link takes the user. To solve the lab, construct and visit the following URL, remembering to change the URL to contain your lab ID and your exploit-server ID:
```
https://your-lab-id.web-security-academy.net/post?postId=4&url=https://your-exploit-server-id.web-security-academy.net/
```

## DOM-based cookie manipulation
### Reference: [PortSwigger: DOM-based cookie manipulation](https://portswigger.net/web-security/dom-based/cookie-manipulation/lab-dom-cookie-manipulation)

<!-- omit in toc -->
### Solution
1. Notice that the home page uses a client-side cookie called ``lastViewedProduct``, whose value is the URL of the last product page that the user visited.
2. Go to the exploit server and add the following ``iframe`` to the body, remembering to replace ``your-lab-id`` with your lab ID:
```
<iframe src="https://your-lab-id.web-security-academy.net/product?productId=1&'><script>print()</script>" onload="if(!window.x)this.src='https://your-lab-id.web-security-academy.net';window.x=1;">
```
3. Store the exploit and deliver it to the victim.
The original source of the ``iframe`` matches the URL of one of the product pages, except there is a JavaScript payload added to the end. When the ``iframe`` loads for the first time, the browser temporarily opens the malicious URL, which is then saved as the value of the ``lastViewedProduct`` cookie. The onload event handler ensures that the victim is then immediately redirected to the home page, unaware that this manipulation ever took place. While the victim's browser has the poisoned cookie saved, loading the home page will cause the payload to execute.

## Exploiting DOM clobbering to enable XSS
### Reference: [PortSwigger: Exploiting DOM clobbering to enable XSS](https://portswigger.net/web-security/dom-based/dom-clobbering/lab-dom-xss-exploiting-dom-clobbering)

<!-- omit in toc -->
### Solution
1. Go to one of the blog posts and create a comment containing the following anchors:

    ```html
    <a id=defaultAvatar><a id=defaultAvatar name=avatar href="cid:&quot;onerror=alert(1)//">
    ```

2. Return to the blog post and create a second comment containing any random text.

3. The next time the page loads, the `alert()` is called.
   
4. **Understanding the DOM clobbering vulnerability:**
   - The `defaultAvatar` object is implemented using this dangerous pattern containing the logical OR operator (`||`) in conjunction with a global variable. This makes it vulnerable to **DOM clobbering**.
   - You can clobber this object using anchor tags. Creating two anchors with the same `id=defaultAvatar` causes them to be grouped in a DOM collection. The `name="avatar"` attribute in the second anchor will **clobber** the `avatar` property with the contents of the `href` attribute.
   - Notice that the site uses the `DOMPurify` filter in an attempt to reduce DOM-based vulnerabilities. However, **DOMPurify** allows the use of the `cid:` protocol, which does not URL-encode double quotes. This allows you to inject an encoded double-quote (`"`) that will be decoded at runtime.
   - As a result, the injection described above will cause the `defaultAvatar` variable to be assigned the clobbered property `{avatar: 'cid:"onerror=alert(1)//'}` the next time the page is loaded.
   
5. **Triggering the XSS:**
   - When you make a second post, the browser uses the newly-clobbered global variable, which smuggles the payload in the `onerror` event handler and triggers the `alert()`.


### Vulnerable Code:
The page imports a JavaScript file `loadCommentsWithDomPurify.js` containing:

```javascript
let defaultAvatar = window.defaultAvatar || {avatar: '/resources/images/avatarDefault.svg'}

## Clobbering DOM attributes to bypass HTML filters
### Reference: [PortSwigger: Clobbering DOM attributes to bypass HTML filters](https://portswigger.net/web-security/dom-based/dom-clobbering/lab-dom-clobbering-attributes-to-bypass-html-filters)

<!-- omit in toc -->
### Solution
1. Go to one of the blog posts and create a comment containing the following HTML:

    ```html
    <form id=x tabindex=0 onfocus=print()><input id=attributes>
    ```

2. Go to the exploit server and add the following `iframe` to the body:

    ```html
    <iframe src="https://YOUR-LAB-ID.web-security-academy.net/post?postId=3" onload="setTimeout(()=>this.src=this.src+'#x',500)">
    ```

    - Replace `YOUR-LAB-ID` with your lab ID.
    - Make sure the `postId=3` matches the post ID of the blog post you injected the HTML into in step 1.

3. Store the exploit and deliver it to the victim.

4. The next time the page loads, the `print()` function is called.

### Understanding the Vulnerability:
- The library uses the `attributes` property to filter HTML attributes. However, it is still possible to **clobber** the `attributes` property itself, causing the length to be `undefined`.
- This clobbering allows you to inject any attributes you want into the form element.
- In this case, you use the `onfocus` attribute to smuggle the `print()` function into the form element.

### Exploit Explanation:
- When the victim loads the page, the comment you injected contains a form with the `onfocus` attribute containing `print()`.
- The iframe added in the exploit page loads the target post (post ID = 3) and then, after a 500ms delay, adds the `#x` fragment to the URL. This delay ensures that the comment containing the injection is loaded before the JavaScript is executed.
- The delay allows the browser to focus on the element with the ID `x` (which is the form you created), triggering the `onfocus` event and calling the `print()` function.

### Result:
- The `print()` function is executed, demonstrating the successful bypass of the HTML filter using DOM attribute clobbering.


