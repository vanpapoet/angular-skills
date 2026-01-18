---
title: Use Updaters for Synchronous State Changes
impact: MEDIUM
impactDescription: Type-safe, predictable state mutations
tags: ngrx, component-store, updaters, state-management
---

## Use Updaters for Synchronous State Changes

**Impact: MEDIUM (type-safe, predictable state mutations)**

Updaters are pure functions that modify state synchronously. Use them for all direct state changes; use effects for async operations.

**Incorrect (direct state mutation):**

```typescript
@Injectable()
export class CartStore extends ComponentStore<CartState> {
  addItem(item: CartItem) {
    // Direct mutation - no type safety, bypasses store patterns
    const currentState = this.get();
    this.setState({
      ...currentState,
      items: [...currentState.items, item]
    });
  }
}
```

**Correct (updaters for state changes):**

```typescript
interface CartState {
  items: CartItem[];
  discount: number;
  loading: boolean;
}

@Injectable()
export class CartStore extends ComponentStore<CartState> {
  constructor() {
    super({ items: [], discount: 0, loading: false });
  }

  // Simple updater with payload
  readonly addItem = this.updater((state, item: CartItem) => ({
    ...state,
    items: [...state.items, item]
  }));

  // Updater with transformation
  readonly updateQuantity = this.updater(
    (state, { id, quantity }: { id: string; quantity: number }) => ({
      ...state,
      items: state.items.map(item =>
        item.id === id ? { ...item, quantity } : item
      )
    })
  );

  // Updater that removes item
  readonly removeItem = this.updater((state, id: string) => ({
    ...state,
    items: state.items.filter(item => item.id !== id)
  }));

  // Updater without payload
  readonly clearCart = this.updater(state => ({
    ...state,
    items: [],
    discount: 0
  }));

  // Use patchState for simple updates
  readonly setLoading = this.updater((state, loading: boolean) => ({
    ...state,
    loading
  }));

  // Alternative: patchState for partial updates
  setDiscount(discount: number) {
    this.patchState({ discount });
  }

  // Batch updates with setState (rarely needed)
  resetToDefaults() {
    this.setState({ items: [], discount: 0, loading: false });
  }
}

// Component usage
@Component({
  providers: [CartStore],
  template: `
    @for (item of store.items$ | async; track item.id) {
      <div>
        {{ item.name }} x {{ item.quantity }}
        <button (click)="store.updateQuantity({ id: item.id, quantity: item.quantity + 1 })">+</button>
        <button (click)="store.removeItem(item.id)">Remove</button>
      </div>
    }
    <button (click)="store.clearCart()">Clear Cart</button>
  `
})
export class CartComponent {
  store = inject(CartStore);

  addProduct(product: Product) {
    this.store.addItem({
      id: product.id,
      name: product.name,
      price: product.price,
      quantity: 1
    });
  }
}
```

**Updater vs patchState vs setState:**

| Method | Use Case |
|--------|----------|
| `updater()` | Complex transformations with payload |
| `patchState()` | Simple partial updates |
| `setState()` | Complete state replacement |

Reference: [NgRx ComponentStore Updaters](https://ngrx.io/guide/component-store/write)
