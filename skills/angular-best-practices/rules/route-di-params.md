---
title: Use DI Factories for Route Parameters
impact: MEDIUM
impactDescription: Testable, reusable route data access
tags: routing, dependency-injection, params, testing
---

## Use DI Factories for Route Parameters

**Impact: MEDIUM (testable, reusable route data access)**

Create injection tokens with factory functions to access route parameters. This improves testability and reduces ActivatedRoute boilerplate.

**Incorrect (repeated ActivatedRoute access):**

```typescript
@Component({ ... })
export class ProductComponent {
  private route = inject(ActivatedRoute);
  productId: string;

  ngOnInit() {
    this.route.paramMap.subscribe(params => {
      this.productId = params.get('id')!;
      this.loadProduct(this.productId);
    });
  }
}

// Testing requires mocking entire ActivatedRoute
```

**Correct (DI factory for route params):**

```typescript
// tokens/route-params.tokens.ts
import { InjectionToken, inject } from '@angular/core';
import { ActivatedRoute } from '@angular/router';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

// Factory for observable param
export function routeParam$(paramKey: string) {
  return new InjectionToken<Observable<string | null>>(`route.param.${paramKey}`, {
    factory: () => {
      const route = inject(ActivatedRoute);
      return route.paramMap.pipe(map(params => params.get(paramKey)));
    }
  });
}

// Factory for snapshot param
export function routeParamSnapshot(paramKey: string) {
  return new InjectionToken<string | null>(`route.param.snapshot.${paramKey}`, {
    factory: () => {
      const route = inject(ActivatedRoute);
      return route.snapshot.paramMap.get(paramKey);
    }
  });
}

// Factory for query param
export function queryParam$(paramKey: string) {
  return new InjectionToken<Observable<string | null>>(`route.query.${paramKey}`, {
    factory: () => {
      const route = inject(ActivatedRoute);
      return route.queryParamMap.pipe(map(params => params.get(paramKey)));
    }
  });
}

// Specific tokens for common params
export const PRODUCT_ID = routeParam$('id');
export const PRODUCT_ID_SNAPSHOT = routeParamSnapshot('id');
export const SEARCH_QUERY = queryParam$('q');

// product.component.ts
@Component({
  selector: 'app-product',
  providers: [
    { provide: PRODUCT_ID, useFactory: () => inject(ActivatedRoute).paramMap.pipe(
      map(p => p.get('id'))
    )}
  ],
  template: `
    @if (productId$ | async; as id) {
      <app-product-details [productId]="id" />
    }
  `
})
export class ProductComponent {
  productId$ = inject(PRODUCT_ID);
}
```

**With input binding (Angular 16+):**

```typescript
// app.config.ts
export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes, withComponentInputBinding())
  ]
};

// product.component.ts
@Component({
  selector: 'app-product',
  template: `<h1>Product: {{ id }}</h1>`
})
export class ProductComponent {
  // Route param ':id' automatically bound
  id = input.required<string>();

  // Query param 'filter' automatically bound
  filter = input<string>();
}

// Route: /products/:id?filter=active
// id = 'product-123', filter = 'active'
```

**Testing advantage:**

```typescript
// Easy to test - just provide the token value
TestBed.configureTestingModule({
  providers: [
    { provide: PRODUCT_ID, useValue: of('test-product-id') }
  ]
});
```

Reference: [Angular Route DI Patterns](https://github.com/angular-vietnam/100-days-of-angular/blob/master/Day048-using-dependency-injection-to-get-data-from-activated-route.md)
