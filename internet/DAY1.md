# Day 1: How the Web Actually Works

## What You'll Learn Today

How a webpage actually loads, from hitting Enter on a URL to pixels on screen. You'll learn to read HTTP requests, understand status codes, and see cookies in action. By the end of today, DevTools will feel like a familiar tool, not a wall of text.

## Core Concepts

### Client-Server Model
When you visit a website, your browser (the **client**) sends a request to a **server**, which sends back the page. Every webpage you've ever loaded is this exchange happening in milliseconds.

### HTTP Request/Response
Every request has:
- A **method**: `GET` (fetch something) or `POST` (send something)
- **Headers**: metadata like what browser you're using, what language you prefer
- **Status codes**: `200` means OK, `404` means not found, `500` means the server broke

### Cookies & Sessions
HTTP is **stateless**. The server forgets you between requests. Cookies solve this: the server gives your browser a small token, and your browser sends it back with every request. That's how a site "remembers" you're logged in.

## Hands-On

### 1. Open DevTools
Open any website. Press `F12` (or `Ctrl+Shift+I`). Click the **Network** tab.

Reload the page. You'll see a list of every request the page made.

Click any request. The panel on the right shows its details:
- **Headers** tab: the method, URL, status code, request and response headers
- **Response** tab: the actual body the server sent back (HTML, JSON, an error message, etc.)
- **Preview** tab: a rendered version of the response

Find:
- One request: click it, read its headers, then switch to the Response tab and read the body
- A cookie the site sets (check the **Application** tab > Cookies)
- A request that returns a non-200 status code: click it and read what the server actually sent back in the Response tab

### 2. View Source vs. Elements
Press `Ctrl+U` to view the raw HTML source. Now compare it to what you see in DevTools > Elements tab. They can differ. The Elements tab shows the *live* DOM after JavaScript has run.

### 3. Use curl
Run this in your terminal:
```bash
curl -v https://example.com
```
Match the output to what you saw in DevTools. You'll see the same headers, the same status code, the same response body, just in text form.

### 4. Modify a Request
In DevTools (Network tab), right-click a request > "Copy as cURL." Paste it in your terminal. Change the `User-Agent` header to something custom (e.g., `User-Agent: NJACK`) and resend it. The server doesn't care. It processes it the same way. This is how you learn that headers are just text you control.

## Resources

- [MDN: An overview of HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview)
- [Chrome DevTools: Network panel reference](https://developer.chrome.com/docs/devtools/network/)
- [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/)
