---
title: Use markForCheck() for Manual Updates
impact: HIGH
impactDescription: Triggers change detection in OnPush components
tags: change-detection, onpush, manual-update
---

## Use markForCheck() for Manual Updates

**Impact: HIGH (triggers change detection in OnPush components)**

When using OnPush change detection, Angular won't detect changes from async operations outside its zone. Use `markForCheck()` to manually trigger change detection when needed.

**Incorrect (mutation without marking for check):**

```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `<div>{{ data }}</div>`
})
export class DataComponent {
  data: string;

  constructor(private socket: WebSocketService) {
    // Changes from WebSocket won't trigger change detection
    this.socket.messages$.subscribe(msg => {
      this.data = msg; // View won't update!
    });
  }
}
```

**Correct (marking for check after async update):**

```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `<div>{{ data }}</div>`
})
export class DataComponent {
  private cdr = inject(ChangeDetectorRef);
  data: string;

  constructor(private socket: WebSocketService) {
    this.socket.messages$.subscribe(msg => {
      this.data = msg;
      this.cdr.markForCheck(); // Schedules change detection
    });
  }
}
```

**Better approach - use signals (automatic tracking):**

```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `<div>{{ data() }}</div>`
})
export class DataComponent {
  data = signal('');

  constructor(private socket: WebSocketService) {
    this.socket.messages$.subscribe(msg => {
      this.data.set(msg); // Automatic change detection with signals
    });
  }
}
```

**Note:** Prefer signals over `markForCheck()` when possible for cleaner, more reactive code.

Reference: [Angular Change Detection](https://angular.dev/best-practices/runtime-performance)
