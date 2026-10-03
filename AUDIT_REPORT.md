# Security Audit Report
## Simulated Reflected Cross-Site Scripting (XSS)

**Project:** IOS Club Technical Team Task Round  
**Domain:** Cybersecurity  
**Assessment Type:** Authorized local/simulated security assessment  
**Application:** Student Feedback Portal (intentionally vulnerable local application)  
**Date:** 2026-10-03  
**Tester:** Darshnik Karn

---

## 1. Executive Summary

A security assessment was performed on a small intentionally vulnerable web application created for educational and defensive testing.

The assessment identified a **Reflected Cross-Site Scripting (XSS)** vulnerability caused by inserting user-controlled input directly into the page using JavaScript's `innerHTML` property.

A proof of concept demonstrated that an attacker-controlled HTML event handler could execute JavaScript in the browser. The vulnerable implementation was then remediated by replacing `innerHTML` with `textContent`, which treats the supplied value as text instead of HTML.

For this **local simulated application**, the assessed CVSS v3.1 score is **5.2 (Medium)** using the vector:

`CVSS:3.1/AV:L/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N`

The score is specific to this local demonstration environment. A real production application's score would depend on its deployment, authentication requirements, affected data, and impact.

---

## 2. Scope

The assessment was limited to the following intentionally vulnerable local application:

- Student Feedback Portal
- Local browser execution
- User-controlled name/feedback input
- Output rendering through JavaScript

No external or third-party systems were tested.

---

## 3. Methodology

The assessment followed these basic stages:

1. Inspect the application input and output behavior.
2. Submit normal text to establish expected functionality.
3. Test whether HTML supplied through the input is interpreted by the browser.
4. Use a harmless XSS proof-of-concept payload.
5. Observe JavaScript execution.
6. Identify the vulnerable code responsible for unsafe HTML insertion.
7. Apply a secure remediation.
8. Re-test using the same payload.
9. Confirm that the payload is rendered as text rather than executed.

---

## 4. Vulnerability Details

### Vulnerability

**Reflected Cross-Site Scripting (XSS)**

### OWASP Category

**A03:2021 – Injection / Cross-Site Scripting**

> Note: The club task asks for XSS/CSRF and CSP/input sanitization. The exact OWASP edition used for the final submission should be stated consistently with the club's expected terminology.

### CWE

**CWE-79 – Improper Neutralization of Input During Web Page Generation (Cross-site Scripting)**

### Root Cause

The application directly inserted user-controlled input into the DOM using:

```javascript
document.getElementById("result").innerHTML =
    "Welcome, " + name + "!";
```

`innerHTML` causes the browser to parse the supplied value as HTML. If an attacker can control that value, they may be able to inject HTML containing executable event handlers or scripts.

---

## 5. Proof of Concept

### 5.1 Normal Input

The application was first tested with:

```text
Darshnik
```

Expected behavior:

```text
Welcome, Darshnik!
```

**Evidence:** `screenshots/01-normal-input.png`

---

### 5.2 XSS Payload

The following harmless proof-of-concept payload was entered into the input:

```html
<img src=x onerror=alert('XSS')>
```

The payload causes an alert dialog to execute if the browser interprets the supplied input as HTML.

**Evidence:** `screenshots/02-xss-executed.png`

The successful alert demonstrates that attacker-controlled markup reached an HTML parsing context and resulted in JavaScript execution.

---

## 6. Vulnerable Code

The vulnerable implementation was:

```javascript
function submitName() {
    let name = document.getElementById("name").value;

    // INTENTIONALLY VULNERABLE
    document.getElementById("result").innerHTML =
        "Welcome, " + name + "!";
}
```

### Security issue

The application does not safely encode the user-controlled value before placing it into an HTML context.

---

## 7. Impact

If the same vulnerability existed in a real web application, successful XSS could potentially allow attacker-controlled JavaScript to execute in another user's browser under the application's origin.

Depending on the application's functionality and browser security controls, possible consequences could include:

- Manipulation of page content
- Unauthorized actions performed through the victim's session
- Theft of accessible application data
- Phishing or misleading UI content
- Session-related compromise in applications that expose sensitive credentials or tokens to JavaScript

The actual impact depends on the application's architecture and what information or actions are accessible from the affected origin.

---

## 8. CVSS Assessment

### CVSS v3.1

**Score: 5.2 (Medium)**

**Vector:**

```text
CVSS:3.1/AV:L/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N
```

### Rationale

| Metric | Value | Reason |
|---|---|---|
| Attack Vector | Local (L) | This demonstration was performed against a local file/application. |
| Attack Complexity | Low (L) | No special conditions were required once the vulnerable input was reached. |
| Privileges Required | None (N) | The simulated application did not require an authenticated privilege. |
| User Interaction | Required (R) | The victim must interact with the affected page/input for this scenario. |
| Scope | Changed (C) | Successful script execution can affect the victim's browser context beyond the input-processing component. |
| Confidentiality | Low (L) | XSS may expose data available to JavaScript in the affected origin. |
| Integrity | Low (L) | Script execution may allow manipulation of page content or actions available to the victim. |
| Availability | None (N) | No availability impact was demonstrated. |

**Important:** This CVSS rating describes the deliberately simplified local demonstration. It should not be presented as the universal score for every XSS vulnerability.

---

## 9. Remediation

### 9.1 Replace `innerHTML` with `textContent`

The direct fix for this application is:

```javascript
function submitName() {
    let name = document.getElementById("name").value;

    // SECURE VERSION
    document.getElementById("result").textContent =
        "Welcome, " + name + "!";
}
```

`textContent` treats the supplied value as text instead of parsing it as HTML.

---

### 9.2 Input Validation

Input validation should also be applied where appropriate.

For example, if the application expects a person's name, it can restrict the input to an appropriate length and character set.

Validation should be treated as an additional control rather than the only XSS defense.

---

### 9.3 Content Security Policy (CSP)

A CSP can provide an additional layer of protection.

Example:

```html
<meta http-equiv="Content-Security-Policy"
      content="default-src 'self'; script-src 'self'; object-src 'none';">
```

For a production application, CSP should preferably be configured through an HTTP response header and tested against the application's actual scripts and resources.

CSP is a **defense-in-depth measure**. It does not replace safe output handling.

---

## 10. Remediation Verification

The same proof-of-concept payload was submitted after changing `innerHTML` to `textContent`:

```html
<img src=x onerror=alert('XSS')>
```

The browser no longer executed the payload.

Instead, the input appeared as literal text:

```text
Welcome, <img src=x onerror=alert('XSS')>!
```

**Evidence:** `screenshots/03-fixed-version.png`

This confirms that the direct vulnerability demonstrated in the simulated application was remediated.

---

## 11. Before vs After

| Area | Vulnerable Version | Fixed Version |
|---|---|---|
| DOM API | `innerHTML` | `textContent` |
| User input parsed as HTML | Yes | No |
| XSS PoC executes | Yes | No |
| Secure output handling | No | Yes |
| CSP recommendation | Not implemented | Recommended as defense-in-depth |

---

## 12. Security Recommendations

1. Prefer safe DOM APIs such as `textContent` for untrusted text.
2. Avoid inserting untrusted input into HTML contexts.
3. Apply context-appropriate output encoding when HTML output is genuinely required.
4. Validate input according to the application's expected data format.
5. Deploy a restrictive Content Security Policy as an additional security layer.
6. Review all locations where user-controlled data reaches HTML, JavaScript, CSS, URL, or other executable contexts.
7. Include XSS testing in application security testing before deployment.

---

## 13. Conclusion

The assessment successfully demonstrated a reflected XSS vulnerability in an intentionally vulnerable local application.

The root cause was unsafe insertion of user-controlled input through `innerHTML`. Replacing the unsafe DOM operation with `textContent` prevented the demonstrated payload from being interpreted as HTML.

The assessment was limited to the authorized simulated environment and was performed for educational and defensive purposes.

---

## 14. Evidence

Place the three screenshots in the `screenshots` directory with these names:

```text
screenshots/
├── 01-normal-input.png
├── 02-xss-executed.png
└── 03-fixed-version.png
```

---

## 15. Ethical Testing Disclaimer

This project was developed and tested only in an intentionally vulnerable local/simulated environment for educational and defensive cybersecurity purposes. No unauthorized systems, websites, accounts, or networks were targeted.
