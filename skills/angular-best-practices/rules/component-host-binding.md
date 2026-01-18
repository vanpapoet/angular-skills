---
title: Use host Property Over Decorators
impact: LOW
impactDescription: Cleaner code, aligns with modern Angular style
tags: components, host, decorators, style
---

## Use host Property Over Decorators

**Impact: LOW (cleaner code, aligns with modern Angular style)**

The `host` property in `@Component` replaces `@HostBinding` and `@HostListener` decorators for a more consolidated configuration.

**Incorrect (legacy decorators):**

```typescript
@Component({
  selector: 'app-button',
  template: `<ng-content />`
})
export class ButtonComponent {
  @HostBinding('class.active') isActive = false;
  @HostBinding('attr.role') role = 'button';
  @HostBinding('style.opacity') get opacity() {
    return this.disabled ? '0.5' : '1';
  }

  @HostListener('click', ['$event'])
  onClick(event: MouseEvent) {
    if (!this.disabled) {
      this.clicked.emit(event);
    }
  }

  @HostListener('keydown.enter')
  onEnter() {
    this.onClick(new MouseEvent('click'));
  }

  disabled = input(false);
  clicked = output<MouseEvent>();
}
```

**Correct (host property):**

```typescript
@Component({
  selector: 'app-button',
  template: `<ng-content />`,
  host: {
    'role': 'button',
    '[class.active]': 'isActive',
    '[class.disabled]': 'disabled()',
    '[attr.aria-disabled]': 'disabled()',
    '[style.opacity]': 'disabled() ? "0.5" : "1"',
    '(click)': 'onClick($event)',
    '(keydown.enter)': 'onClick($event)'
  }
})
export class ButtonComponent {
  isActive = false;
  disabled = input(false);
  clicked = output<MouseEvent>();

  onClick(event: Event) {
    if (!this.disabled()) {
      this.clicked.emit(event as MouseEvent);
    }
  }
}
```

**Host property syntax:**

| Syntax | Purpose |
|--------|---------|
| `'attr'` | Static attribute |
| `'[attr]'` | Dynamic attribute binding |
| `'[class.name]'` | CSS class binding |
| `'[style.prop]'` | Style binding |
| `'(event)'` | Event listener |

Reference: [Angular Style Guide](https://angular.dev/style-guide)
