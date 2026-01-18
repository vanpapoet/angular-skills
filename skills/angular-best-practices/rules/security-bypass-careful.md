---
title: Use bypassSecurityTrust* with Extreme Caution
impact: MEDIUM
impactDescription: Bypassing sanitization can introduce XSS vulnerabilities
tags: security, xss, bypass, dom-sanitizer
---

## Use bypassSecurityTrust* with Extreme Caution

**Impact: MEDIUM (bypassing sanitization can introduce XSS vulnerabilities)**

The `DomSanitizer.bypassSecurityTrust*` methods disable Angular's automatic sanitization. Only use them when you've verified the content is safe.

**Incorrect (bypassing without validation):**

```typescript
@Component({
  template: `<div [innerHTML]="trustedHtml"></div>`
})
export class UnsafeComponent {
  private sanitizer = inject(DomSanitizer);

  // DANGEROUS: bypassing with user input
  setContent(userInput: string) {
    this.trustedHtml = this.sanitizer.bypassSecurityTrustHtml(userInput);
  }
}
```

**Correct (validated content with audit trail):**

```typescript
@Component({
  template: `<div [innerHTML]="trustedHtml"></div>`
})
export class SafeComponent {
  private sanitizer = inject(DomSanitizer);
  trustedHtml: SafeHtml;

  constructor() {
    // Only use for content YOU control, not user input
    // Content from your CMS that's already sanitized server-side
    const adminContent = '<p class="highlight">Admin-approved content</p>';

    // Document why this is safe
    // SECURITY: This HTML comes from our admin CMS which sanitizes all input
    this.trustedHtml = this.sanitizer.bypassSecurityTrustHtml(adminContent);
  }
}
```

**Available bypass methods:**

| Method | Use Case |
|--------|----------|
| `bypassSecurityTrustHtml` | Pre-sanitized HTML from trusted source |
| `bypassSecurityTrustStyle` | Dynamic CSS from trusted source |
| `bypassSecurityTrustUrl` | URLs that Angular incorrectly blocks |
| `bypassSecurityTrustResourceUrl` | iframe/object sources (most dangerous) |
| `bypassSecurityTrustScript` | Inline scripts (rarely needed) |

**Best practices:**
- Prefer Angular's sanitization over bypassing
- Document every bypass with security rationale
- Review all bypass usages in code reviews
- Consider server-side sanitization for user content
- Use Content Security Policy as defense-in-depth

```typescript
// Create a dedicated service for auditable bypass
@Injectable({ providedIn: 'root' })
export class TrustedContentService {
  private sanitizer = inject(DomSanitizer);

  /**
   * SECURITY: Only call with content from admin CMS
   * @param html - Pre-sanitized HTML from admin dashboard
   */
  trustAdminHtml(html: string): SafeHtml {
    // Log for audit trail
    console.debug('Trusting admin HTML:', html.substring(0, 100));
    return this.sanitizer.bypassSecurityTrustHtml(html);
  }
}
```

Reference: [Angular Security Guide](https://angular.dev/best-practices/security)
