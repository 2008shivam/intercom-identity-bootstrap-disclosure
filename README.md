# 🔐 Intercom Identity Bootstrapping — Responsible Disclosure

![Severity](https://img.shields.io/badge/Severity-High-red)
![CVSS](https://img.shields.io/badge/CVSS%20v3.1-7.8-orange)
![Status](https://img.shields.io/badge/Status-Accepted%20%26%20Bounty%20Awarded-brightgreen)
![Bounty](https://img.shields.io/badge/Bounty-%24200%20USDT-blue)

---

## 📋 Summary

A **High-severity identity impersonation vulnerability** was discovered in the Intercom chat widget integration on `de.fi`. The widget was bootstrapped client-side without server-signed identity verification (HMAC tokens), allowing an attacker who knows a victim's email address to impersonate that user and access their prior chat history.

The vulnerability was responsibly disclosed to the affected vendor. The report was accepted and forwarded to **Intercom's security team** for remediation.

---

## 🎯 Affected Target

| Field | Details |
|---|---|
| Target | `https://de.fi` |
| Component | Intercom chat widget (client-side integration) |
| Vulnerability Type | Missing Identity Verification / Authentication Bypass |
| CWE | CWE-306: Missing Authentication for Critical Function |

---

## ⚠️ Severity

| Metric | Score |
|---|---|
| **CVSS v3.1 Score** | **7.8 (High)** |
| Confidentiality Impact | High |
| Integrity Impact | High |
| Availability Impact | Low |

---

## 🧠 Vulnerability Description

Intercom provides an optional **Identity Verification** feature that uses server-side HMAC-signed tokens to confirm a user's identity before bootstrapping the widget. When this is **not implemented**, the widget trusts any identity passed via client-side JavaScript.

This means anyone with access to the browser console can execute:

```javascript
Intercom('boot', { email: 'victim@example.com' });
```

...and gain access to the victim's **full chat history**, including any PII or sensitive commercial information exchanged with the support team.

---

## 🔁 Reproduction Steps (Redacted Safe PoC)

1. Visit `https://de.fi` and log in to an account.
2. Initiate a conversation via the Intercom chat widget.
3. Close and terminate the session.
4. Open a **new browser session** on the same site and launch Developer Tools.
5. In the browser console, execute:
   ```javascript
   Intercom('boot', { email: 'xyz@victim.com' });
   ```
6. Observe that the widget loads with the victim user's identity and chat history — **no authentication required**.

> ⚠️ **Note:** PoC video withheld pending full remediation by Intercom.

---

## 💥 Impact

- **User Impersonation** — Attacker can appear as any user in the support widget
- **PII Exposure** — Prior messages may contain sensitive personal or commercial data
- **Message Injection** — Attacker can send messages appearing to originate from the victim
- **No Authentication Required** — Only the victim's email address is needed

---

## 🔧 Remediation

1. **Implement Intercom Identity Verification** — Generate server-side HMAC-signed tokens using your Intercom secret key. Only initialise the widget client-side when a valid signed token is present.

   ```javascript
   // Server-side (Node.js example)
   const crypto = require('crypto');
   const hash = crypto.createHmac('sha256', process.env.INTERCOM_SECRET)
                      .update(userId)
                      .digest('hex');
   ```

2. **Block unauthenticated widget bootstrap paths** — Ensure no client-side code paths can inject unverified identity attributes.

3. **Require authentication before exposing chat history** — Historical records should only load after identity is cryptographically verified.

4. **Log all bootstrap events** — Capture IP, timestamp, and initiating URL; alert on suspicious patterns.

5. **Review historic conversations** — Scan for signs of unauthorized access in existing chat logs.

---

## 📬 Disclosure Timeline

| Date | Event |
|---|---|
| Discovery | Vulnerability identified during manual security assessment |
| Report Submitted | Full report sent to De.Fi security team |
| Accepted | Bug report accepted and forwarded to Intercom's team |
| Bounty Awarded | $200 USDT awarded by De.Fi |
| Publication | Post-remediation public disclosure |

---

## 🏆 Bug Bounty

**Program:** De.Fi  
**Reward:** $200 USDT (BSC chain)  
**Status:** ✅ Accepted & Paid
**Payment Proof:** ![Payment Proof](https://raw.githubusercontent.com/2008shivam/intercom-identity-bootstrap-disclosure/main/Proof/de_fi_bounty_Payment.jpg)

---

## 👤 Researcher

**Shivam Jha**  
Senior Security Engineer | Penetration Tester | Red Team Operator  
📧 jhashivam2008@gmail.com  
🔗 [GitHub](https://github.com/2008shivam)

Certifications: eCPPT | eJPT | OSCP (in progress)

---

## 📄 Report

The full detailed vulnerability report is available in [`/report/Intercom_Vulnerability_Report.pdf`](./report/Intercom_Vulnerability_Report.pdf)

---

## ⚖️ Disclaimer

This research was conducted **ethically and responsibly**. No user data was accessed, exfiltrated, or stored at any point. The vulnerability was disclosed privately to the vendor before any public release. This repository is intended for **educational purposes** and to document responsible security research.

---

*Prepared under responsible disclosure principles. All findings shared with authorised recipients only prior to publication.*
