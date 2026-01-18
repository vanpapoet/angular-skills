---
name: angular-best-practices
description: Angular performance optimization and best practices guidelines. Use when writing, reviewing, or refactoring Angular code to ensure optimal patterns. Triggers on tasks involving Angular components, signals, change detection, routing, forms, or performance improvements.
license: MIT
metadata:
  author: vanpapoet
  version: "1.0.0"
---

# Angular Best Practices

Comprehensive performance optimization and best practices guide for Angular v21+ applications. Contains 55+ rules across 12 categories, prioritized by impact to guide automated refactoring and code generation.

## When to Apply

Reference these guidelines when:

- Writing new Angular components or services
- Implementing reactive state with signals
- Reviewing code for performance issues
- Refactoring existing Angular code
- Optimizing bundle size or load times
- Implementing forms or routing

## Rule Categories by Priority

| Priority | Category                          | Impact      | Prefix       |
| -------- | --------------------------------- | ----------- | ------------ |
| 1        | Change Detection                  | CRITICAL    | `cd-`        |
| 2        | Bundle Optimization               | CRITICAL    | `bundle-`    |
| 3        | Template Performance              | HIGH        | `template-`  |
| 4        | Component Architecture            | HIGH        | `component-` |
| 5        | Routing & Navigation              | HIGH        | `route-`     |
| 6        | State Management (Signals)        | MEDIUM-HIGH | `signal-`    |
| 7        | State Management (ComponentStore) | MEDIUM-HIGH | `state-`     |
| 8        | Forms                             | MEDIUM      | `form-`      |
| 9        | Security                          | MEDIUM      | `security-`  |
| 10       | Dependency Injection              | MEDIUM      | `di-`        |
| 11       | HTTP & Data Fetching              | MEDIUM      | `http-`      |
| 12       | Style & Conventions               | LOW         | `style-`     |

## Quick Reference

### 1. Change Detection (CRITICAL)

- `cd-onpush` - Use OnPush strategy for performance
- `cd-zoneless` - Adopt zoneless change detection (v21 default)
- `cd-signal-inputs` - Use signal-based inputs for fine-grained reactivity
- `cd-mark-for-check` - Use markForCheck() for manual updates

### 2. Bundle Optimization (CRITICAL)

- `bundle-defer` - Use @defer for lazy loading components
- `bundle-lazy-routes` - Lazy load route modules
- `bundle-prefetch` - Prefetch deferred content strategically
- `bundle-standalone` - Use standalone components to reduce bundle

### 3. Template Performance (HIGH)

- `template-control-flow` - Use @if/@for/@switch over NgIf/NgFor
- `template-track` - Always use track in @for loops
- `template-avoid-function-calls` - Avoid function calls in templates
- `template-pure-pipes` - Use pure pipes for transformations

### 4. Component Architecture (HIGH)

- `component-standalone` - Prefer standalone components
- `component-single-responsibility` - One component per file
- `component-smart-dumb` - Separate smart and presentational components
- `component-host-binding` - Use host property over decorators

### 5. Routing & Navigation (HIGH)

- `route-functional-guards` - Use functional guards for route protection
- `route-can-deactivate` - Prevent unsaved changes loss
- `route-can-match` - Control lazy loading with CanMatch
- `route-di-params` - Use DI factories for route parameters
- `route-preloading-strategy` - Implement custom preloading strategies
- `route-resolvers` - Use resolvers for pre-fetching route data

### 6. State Management - Signals (MEDIUM-HIGH)

- `signal-computed` - Use computed() for derived state
- `signal-readonly` - Expose signals as readonly
- `signal-effects` - Use effect() sparingly for side effects
- `signal-untracked` - Use untracked() to prevent dependency tracking

### 7. State Management - ComponentStore (MEDIUM-HIGH)

- `state-component-store` - Use ComponentStore for local component state
- `state-component-store-selectors` - Use selectors to read state
- `state-component-store-updaters` - Use updaters for synchronous changes
- `state-component-store-effects` - Use effects for async operations

### 8. Forms (MEDIUM)

- `form-reactive` - Prefer reactive forms for complex scenarios
- `form-typed` - Use strongly typed forms
- `form-validation` - Centralize validation logic
- `form-unsubscribe` - Clean up valueChanges subscriptions

### 9. Security (MEDIUM)

- `security-sanitize` - Trust Angular's automatic sanitization
- `security-bypass-careful` - Use bypassSecurityTrust\* with caution
- `security-csp` - Configure Content Security Policy
- `security-no-dynamic-templates` - Never generate templates dynamically

### 10. Dependency Injection (MEDIUM)

- `di-inject-function` - Prefer inject() over constructor injection
- `di-provided-in-root` - Use providedIn: 'root' for singletons
- `di-injection-token` - Use InjectionToken for non-class deps

### 11. HTTP & Data Fetching (MEDIUM)

- `http-interceptors` - Use interceptors for cross-cutting concerns
- `http-typed-responses` - Type HTTP responses
- `http-error-handling` - Centralize error handling
- `http-caching` - Implement caching strategies

### 12. Style & Conventions (LOW)

- `style-file-naming` - Use kebab-case for file names
- `style-selector-prefix` - Apply consistent selector prefixes
- `style-organize-imports` - Group and organize imports
- `style-lifecycle-interfaces` - Implement lifecycle interfaces

## How to Use

Read individual rule files for detailed explanations and code examples:

```
rules/cd-onpush.md
rules/bundle-defer.md
rules/_sections.md
```

Each rule file contains:

- Brief explanation of why it matters
- Incorrect code example with explanation
- Correct code example with explanation
- Additional context and references

## Full Compiled Document

For the complete guide with all rules expanded: `AGENTS.md`
