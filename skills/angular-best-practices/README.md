# Angular Best Practices Skill

A comprehensive collection of Angular best practices and performance optimization guidelines for AI-assisted development.

## Overview

This skill provides structured guidance for writing high-quality Angular code, covering:

- **Change Detection** - OnPush, zoneless, signals
- **Bundle Optimization** - @defer, lazy loading, standalone components
- **Template Performance** - Control flow, track expressions
- **Component Architecture** - Standalone, smart/dumb pattern
- **Routing & Navigation** - Functional guards, resolvers, preloading strategies
- **State Management (Signals)** - computed, effects, untracked
- **State Management (ComponentStore)** - Selectors, updaters, effects
- **Forms** - Reactive forms, validation
- **Security** - XSS prevention, CSP
- **Dependency Injection** - inject(), providedIn
- **HTTP** - Interceptors, error handling
- **Style Conventions** - Naming, organization

## Usage

The skill is automatically activated when working with Angular code. Individual rules can be referenced by their prefix and name (e.g., `cd-onpush`, `bundle-defer`).

## File Structure

```
angular-best-practices/
├── SKILL.md           # Main skill definition
├── AGENTS.md          # Compiled rules document
├── README.md          # This file
├── metadata.json      # Skill metadata
└── rules/
    ├── _template.md   # Rule template
    ├── _sections.md   # Category overview
    └── *.md           # Individual rules
```

## Categories by Priority

| Priority | Category | Prefix | Impact |
|----------|----------|--------|--------|
| 1 | Change Detection | `cd-*` | CRITICAL |
| 2 | Bundle Optimization | `bundle-*` | CRITICAL |
| 3 | Template Performance | `template-*` | HIGH |
| 4 | Component Architecture | `component-*` | HIGH |
| 5 | Routing & Navigation | `route-*` | HIGH |
| 6 | State Management (Signals) | `signal-*` | MEDIUM-HIGH |
| 7 | State Management (ComponentStore) | `state-*` | MEDIUM-HIGH |
| 8 | Forms | `form-*` | MEDIUM |
| 9 | Security | `security-*` | MEDIUM |
| 10 | Dependency Injection | `di-*` | MEDIUM |
| 11 | HTTP & Data Fetching | `http-*` | MEDIUM |
| 12 | Style & Conventions | `style-*` | LOW |

## Angular Version

This skill targets Angular v21+ and includes guidance for:

- Signal-based reactivity
- Zoneless change detection
- Native control flow (@if, @for, @switch)
- Deferrable views (@defer)
- Standalone components (default)

### Key Angular v21 Features Covered

- Zoneless change detection (default in v21)
- Signal-based inputs and state management
- Native control flow (@if, @for, @switch)
- @defer blocks for lazy loading
- Standalone components (default)
- Signal Forms (experimental)
- inject() function pattern

## Contributing

When adding new rules:

1. Use the `_template.md` format
2. Include incorrect and correct code examples
3. Specify impact level (CRITICAL, HIGH, MEDIUM, LOW)
4. Add relevant tags for discoverability

## References

- [Angular Documentation](https://angular.dev)
- [Angular Style Guide](https://angular.dev/style-guide)
- [Angular Security](https://angular.dev/best-practices/security)
- [Angular v21 Release](https://angular.love/angular-21-whats-new)
