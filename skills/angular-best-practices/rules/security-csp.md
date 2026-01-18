---
title: Configure Content Security Policy
impact: MEDIUM
impactDescription: Defense-in-depth against XSS attacks
tags: security, csp, headers, xss
---

## Configure Content Security Policy

**Impact: MEDIUM (defense-in-depth against XSS attacks)**

Content Security Policy (CSP) provides an additional layer of protection by restricting which resources can be loaded and executed.

**Incorrect (no CSP or overly permissive):**

```html
<!-- No CSP header - vulnerable to injected scripts -->
<head>
  <script src="https://evil.com/malicious.js"></script> <!-- Could be injected -->
</head>
```

**Correct (strict CSP configuration):**

```typescript
// Configure CSP in your server/middleware
// Express.js example:
app.use((req, res, next) => {
  const nonce = crypto.randomBytes(16).toString('base64');
  res.locals.nonce = nonce;

  res.setHeader('Content-Security-Policy', [
    "default-src 'self'",
    `script-src 'self' 'nonce-${nonce}'`,
    `style-src 'self' 'nonce-${nonce}'`,
    "img-src 'self' data: https:",
    "font-src 'self'",
    "connect-src 'self' https://api.example.com",
    "frame-ancestors 'none'",
    "base-uri 'self'",
    "form-action 'self'"
  ].join('; '));

  next();
});
```

**Angular CSP configuration:**

```typescript
// main.ts - provide nonce to Angular
import { CSP_NONCE } from '@angular/core';

bootstrapApplication(AppComponent, {
  providers: [
    { provide: CSP_NONCE, useValue: globalThis['CSP_NONCE'] }
  ]
});
```

```html
<!-- index.html - inline script to capture nonce -->
<script nonce="{{SERVER_NONCE}}">
  window.CSP_NONCE = '{{SERVER_NONCE}}';
</script>
```

**angular.json configuration:**

```json
{
  "projects": {
    "app": {
      "architect": {
        "build": {
          "options": {
            "styles": [
              {
                "input": "src/styles.css",
                "inject": true,
                "bundleName": "styles"
              }
            ]
          }
        }
      }
    }
  }
}
```

**Recommended CSP directives:**

| Directive | Recommended Value | Purpose |
|-----------|-------------------|---------|
| `default-src` | `'self'` | Fallback for all resource types |
| `script-src` | `'self' 'nonce-xxx'` | JavaScript sources |
| `style-src` | `'self' 'nonce-xxx'` | CSS sources |
| `img-src` | `'self' data: https:` | Image sources |
| `connect-src` | `'self' api-url` | XHR/fetch destinations |
| `frame-ancestors` | `'none'` | Prevent clickjacking |

Reference: [Angular Security - CSP](https://angular.dev/best-practices/security)
