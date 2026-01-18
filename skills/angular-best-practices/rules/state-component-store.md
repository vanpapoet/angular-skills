---
title: Use ComponentStore for Local Component State
impact: HIGH
impactDescription: Encapsulated state management, simpler than global store
tags: ngrx, component-store, state-management, local-state
---

## Use ComponentStore for Local Component State

**Impact: HIGH (encapsulated state management, simpler than global store)**

NgRx ComponentStore provides lightweight, component-scoped state management. Use it for complex component state that doesn't need to be shared globally.

**Incorrect (complex state in component):**

```typescript
@Component({ ... })
export class ProductListComponent {
  products: Product[] = [];
  loading = false;
  error: string | null = null;
  selectedIds: Set<string> = new Set();
  filter = '';
  sortBy = 'name';

  // State logic scattered across methods
  loadProducts() {
    this.loading = true;
    this.error = null;
    this.productService.getAll().subscribe({
      next: products => {
        this.products = products;
        this.loading = false;
      },
      error: err => {
        this.error = err.message;
        this.loading = false;
      }
    });
  }
}
```

**Correct (ComponentStore for organized state):**

```typescript
// product-list.store.ts
interface ProductListState {
  products: Product[];
  loading: boolean;
  error: string | null;
  selectedIds: string[];
  filter: string;
}

const initialState: ProductListState = {
  products: [],
  loading: false,
  error: null,
  selectedIds: [],
  filter: ''
};

@Injectable()
export class ProductListStore extends ComponentStore<ProductListState> {
  private productService = inject(ProductService);

  constructor() {
    super(initialState);
  }

  // Selectors
  readonly products$ = this.select(state => state.products);
  readonly loading$ = this.select(state => state.loading);
  readonly error$ = this.select(state => state.error);
  readonly selectedIds$ = this.select(state => state.selectedIds);

  // Combined selector for filtered products
  readonly filteredProducts$ = this.select(
    this.products$,
    this.select(state => state.filter),
    (products, filter) => products.filter(p =>
      p.name.toLowerCase().includes(filter.toLowerCase())
    )
  );

  // View model selector
  readonly vm$ = this.select({
    products: this.filteredProducts$,
    loading: this.loading$,
    error: this.error$,
    selectedIds: this.selectedIds$
  });

  // Updaters (synchronous)
  readonly setFilter = this.updater((state, filter: string) => ({
    ...state,
    filter
  }));

  readonly toggleSelection = this.updater((state, id: string) => ({
    ...state,
    selectedIds: state.selectedIds.includes(id)
      ? state.selectedIds.filter(i => i !== id)
      : [...state.selectedIds, id]
  }));

  // Effects (asynchronous)
  readonly loadProducts = this.effect<void>(trigger$ =>
    trigger$.pipe(
      tap(() => this.patchState({ loading: true, error: null })),
      switchMap(() => this.productService.getAll().pipe(
        tapResponse(
          products => this.patchState({ products, loading: false }),
          error => this.patchState({ error: error.message, loading: false })
        )
      ))
    )
  );
}

// product-list.component.ts
@Component({
  selector: 'app-product-list',
  providers: [ProductListStore],
  template: `
    @if (store.vm$ | async; as vm) {
      @if (vm.loading) {
        <app-spinner />
      } @else if (vm.error) {
        <app-error [message]="vm.error" />
      } @else {
        @for (product of vm.products; track product.id) {
          <app-product-card
            [product]="product"
            [selected]="vm.selectedIds.includes(product.id)"
            (toggle)="store.toggleSelection(product.id)"
          />
        }
      }
    }
  `
})
export class ProductListComponent {
  store = inject(ProductListStore);

  ngOnInit() {
    this.store.loadProducts();
  }
}
```

**When to use ComponentStore vs Global Store:**

| Use ComponentStore | Use Global NgRx Store |
|-------------------|----------------------|
| Component-specific state | App-wide shared state |
| List/form state | User authentication |
| UI state (modals, tabs) | Shopping cart |
| Feature isolation | Cross-feature communication |

Reference: [NgRx ComponentStore](https://ngrx.io/guide/component-store)
