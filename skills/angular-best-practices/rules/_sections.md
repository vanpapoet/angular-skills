# Angular Best Practices - Section Overview

## 1. Change Detection (CRITICAL)

Change detection is Angular's mechanism for updating the DOM when application state changes. Optimizing change detection has the highest impact on application performance.

**Key Rules:**
- `cd-onpush` - Use ChangeDetectionStrategy.OnPush
- `cd-zoneless` - Adopt zoneless change detection
- `cd-signal-inputs` - Use signal-based inputs
- `cd-mark-for-check` - Manual change detection triggers

## 2. Bundle Optimization (CRITICAL)

Reducing initial bundle size directly improves load time and Core Web Vitals scores.

**Key Rules:**
- `bundle-defer` - @defer blocks for lazy components
- `bundle-lazy-routes` - Route-based code splitting
- `bundle-prefetch` - Strategic prefetching
- `bundle-standalone` - Standalone components reduce tree-shaking barriers

## 3. Template Performance (HIGH)

Template efficiency affects rendering performance and memory usage.

**Key Rules:**
- `template-control-flow` - Native @if/@for/@switch
- `template-track` - Track expressions in @for
- `template-avoid-function-calls` - Computed properties over functions
- `template-pure-pipes` - Pure pipes for memoization

## 4. Component Architecture (HIGH)

Well-structured components are easier to test, maintain, and optimize.

**Key Rules:**
- `component-standalone` - Standalone as default
- `component-single-responsibility` - One concept per file
- `component-smart-dumb` - Container/presentational pattern
- `component-host-binding` - Modern host property syntax

## 5. Routing & Navigation (HIGH)

Proper routing architecture improves navigation UX and enables code splitting.

**Key Rules:**
- `route-functional-guards` - Functional guards for route protection
- `route-can-deactivate` - Prevent unsaved changes loss
- `route-can-match` - Control lazy loading with CanMatch
- `route-di-params` - DI factories for route parameters
- `route-preloading-strategy` - Custom preloading strategies
- `route-resolvers` - Pre-fetch route data with resolvers

## 6. State Management - Signals (MEDIUM-HIGH)

Signal-based state management provides fine-grained reactivity.

**Key Rules:**
- `signal-computed` - Derived state with computed()
- `signal-readonly` - Encapsulation via asReadonly()
- `signal-effects` - Side effects with effect()
- `signal-untracked` - Prevent unwanted tracking

## 7. State Management - ComponentStore (MEDIUM-HIGH)

NgRx ComponentStore provides encapsulated state management for components.

**Key Rules:**
- `state-component-store` - Use ComponentStore for local state
- `state-component-store-selectors` - Selectors for state access
- `state-component-store-updaters` - Updaters for synchronous changes
- `state-component-store-effects` - Effects for async operations

## 8. Forms (MEDIUM)

Proper form architecture improves maintainability and performance.

**Key Rules:**
- `form-reactive` - Reactive forms for complexity
- `form-typed` - Strongly typed FormGroup/FormControl
- `form-validation` - Centralized validators
- `form-unsubscribe` - Subscription cleanup

## 9. Security (MEDIUM)

Angular provides robust security features that should not be bypassed.

**Key Rules:**
- `security-sanitize` - Trust automatic sanitization
- `security-bypass-careful` - Audit bypass methods
- `security-csp` - Content Security Policy
- `security-no-dynamic-templates` - Static templates only

## 10. Dependency Injection (MEDIUM)

Modern DI patterns improve code readability and testability.

**Key Rules:**
- `di-inject-function` - inject() over constructor
- `di-provided-in-root` - Tree-shakable providers
- `di-injection-token` - Non-class dependencies

## 11. HTTP & Data Fetching (MEDIUM)

Efficient data fetching reduces network overhead and improves UX.

**Key Rules:**
- `http-interceptors` - Centralized request handling
- `http-typed-responses` - Type-safe responses
- `http-error-handling` - Consistent error handling
- `http-caching` - Response caching strategies

## 12. Style & Conventions (LOW)

Consistent style improves team productivity and code discoverability.

**Key Rules:**
- `style-file-naming` - Kebab-case conventions
- `style-selector-prefix` - Application-specific prefixes
- `style-organize-imports` - Logical import grouping
- `style-lifecycle-interfaces` - Explicit interface implementation
