---
description: 16 Oct , 2025 - Xploit0x90
icon: flag
---

# Reflected DOM XSS - 16.Oct

<figure><img src="../.gitbook/assets/4A6B75FE-1E4A-45DE-B2E1-E5B2DDB74596.webp" alt=""><figcaption></figcaption></figure>

***

## Reflected DOM XSS – PortSwigger Lab Write-Up

***

### Overview

This lab demonstrates a **reflected DOM-based Cross-Site Scripting (XSS)** vulnerability. The goal was to identify how user input is reflected in the DOM and exploit it to execute arbitrary JavaScript.

***

### Step 1 – Discovering Reflection

First, I entered a special test input to determine where the website reflects user data. Using the browser inspector, I observed that the input was reflected **inside an `<h1>` tag**:

```html
mosayhi
```

<figure><img src="../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

***

### Step 2 – Initial Payload Test

Next, I tested for basic XSS with the classic payload:

```html
<script>alert('mosayhi')</script>
```

The website **escaped the `<` and `>` characters**, preventing execution:

<figure><img src="../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

***

### Step 3 – Analyzing the Request

By intercepting the request in Burp Suite, it was clear that **our input was reflected in the HTTP response**:

<figure><img src="../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>



***

### Step 4 – Identifying the Vulnerable Context

Examining the page’s JavaScript revealed an `eval()` call that executes **user-controlled data from `responseText()`**. Since `eval()` executes strings as JavaScript, this is a highly dangerous context:

<figure><img src="../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>



***

### Step 5 – Bypassing Escaping

When entering a double quote (`"`), it was escaped with a backslash (`\"`) to prevent breaking the string. By **adding a second backslash**, the first escape is neutralized, allowing us to close the string:

<figure><img src="../.gitbook/assets/image (4) (2).png" alt=""><figcaption></figcaption></figure>

***

### Step 6 – Crafting the Exploit

By closing the object with `}` and appending our payload, we can inject JavaScript and comment out the rest of the original code with `//`. This makes the payload executable:

```javascript
"}; alert('XSS'); //
```

Executing this payload triggers the alert, demonstrating a successful **reflected DOM XSS**, and completing the lab.

<figure><img src="../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

***

### Conclusion

* The vulnerability exists because **user input is passed into `eval()` without proper sanitization**.
* Standard escaping of `<` and `>` is insufficient in a JavaScript context.
* DOM-based XSS requires **understanding how input flows into the page and which JavaScript constructs are used**, rather than just testing `<script>` tags.

This lab is a classic example of why `eval()` with user-controlled input is dangerous and why context-aware sanitization or using safe APIs (like `textContent`) is critical.

***
