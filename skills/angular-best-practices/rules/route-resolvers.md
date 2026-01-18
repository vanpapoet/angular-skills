---
title: Use Resolvers for Pre-fetching Route Data
impact: MEDIUM
impactDescription: Data ready before component renders
tags: routing, resolvers, data-fetching, ux
---

## Use Resolvers for Pre-fetching Route Data

**Impact: MEDIUM (data ready before component renders)**

Resolvers fetch required data before the route activates, ensuring components have data immediately upon rendering.

**Incorrect (loading data in component):**

```typescript
@Component({
  template: `
    @if (loading()) {
      <app-spinner />
    } @else if (product(); as p) {
      <h1>{{ p.name }}</h1>
    }
  `
})
export class ProductComponent {
  private route = inject(ActivatedRoute);
  private productService = inject(ProductService);

  product = signal<Product | null>(null);
  loading = signal(true);

  ngOnInit() {
    const id = this.route.snapshot.paramMap.get('id')!;
    this.productService.getById(id).subscribe({
      next: p => {
        this.product.set(p);
        this.loading.set(false);
      }
    });
  }
}
```

**Correct (functional resolver):**

```typescript
// resolvers/product.resolver.ts
export const productResolver: ResolveFn<Product> = (route, state) => {
  const productService = inject(ProductService);
  const router = inject(Router);

  const id = route.paramMap.get('id')!;

  return productService.getById(id).pipe(
    catchError(error => {
      router.navigate(['/products']);
      return EMPTY;
    })
  );
};

// resolvers/products-list.resolver.ts
export const productsResolver: ResolveFn<Product[]> = (route, state) => {
  const productService = inject(ProductService);
  const category = route.queryParamMap.get('category');

  return productService.getAll({ category });
};

// app.routes.ts
export const routes: Routes = [
  {
    path: 'products',
    component: ProductListComponent,
    resolve: { products: productsResolver }
  },
  {
    path: 'products/:id',
    component: ProductComponent,
    resolve: { product: productResolver }
  }
];

// product.component.ts
@Component({
  template: `
    <h1>{{ product().name }}</h1>
    <p>{{ product().description }}</p>
  `
})
export class ProductComponent {
  private route = inject(ActivatedRoute);

  // Data already available - no loading state needed
  product = toSignal(
    this.route.data.pipe(map(data => data['product'] as Product)),
    { requireSync: true }
  );
}
```

**Resolver with multiple data sources:**

```typescript
export const productPageResolver: ResolveFn<ProductPageData> = (route, state) => {
  const productService = inject(ProductService);
  const reviewService = inject(ReviewService);

  const id = route.paramMap.get('id')!;

  return forkJoin({
    product: productService.getById(id),
    reviews: reviewService.getByProductId(id),
    related: productService.getRelated(id)
  });
};

// Component accesses all data from route
@Component({ ... })
export class ProductPageComponent {
  private route = inject(ActivatedRoute);

  data = toSignal(
    this.route.data.pipe(map(d => d['pageData'] as ProductPageData)),
    { requireSync: true }
  );

  product = computed(() => this.data().product);
  reviews = computed(() => this.data().reviews);
  related = computed(() => this.data().related);
}
```

**When to use resolvers vs component loading:**

| Use Resolvers | Use Component Loading |
|---------------|----------------------|
| Critical data for initial render | Optional/secondary data |
| Simple, fast data fetches | Complex loading states |
| SEO-important content | User-triggered fetches |
| Server-side rendering | Streaming data |

Reference: [Angular Router Resolvers](https://github.com/angular-vietnam/100-days-of-angular/blob/master/Day030-router-guards-resolvers.md)
