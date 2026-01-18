---
title: Group and Organize Imports
impact: LOW
impactDescription: Improves readability and maintainability
tags: style, imports, organization
---

## Group and Organize Imports

**Impact: LOW (improves readability and maintainability)**

Organize imports in logical groups with consistent ordering. This makes it easier to scan dependencies and identify external vs internal code.

**Incorrect (random order):**

```typescript
import { UserService } from './user.service';
import { Component, inject } from '@angular/core';
import { map } from 'rxjs/operators';
import { User } from '../models/user.model';
import { Router } from '@angular/router';
import { Observable } from 'rxjs';
import { HttpClient } from '@angular/common/http';
import { FormBuilder } from '@angular/forms';
```

**Correct (grouped and ordered):**

```typescript
// 1. Angular core imports
import { Component, inject, signal, computed } from '@angular/core';
import { Router, ActivatedRoute, RouterLink } from '@angular/router';
import { HttpClient } from '@angular/common/http';
import { FormBuilder, ReactiveFormsModule } from '@angular/forms';

// 2. Third-party library imports
import { Observable } from 'rxjs';
import { map, switchMap, catchError } from 'rxjs/operators';

// 3. Application imports - shared/core
import { AuthService } from '@core/services/auth.service';
import { API_URL } from '@core/tokens/api.tokens';

// 4. Application imports - feature-specific
import { User } from '../models/user.model';
import { UserService } from './user.service';
import { UserCardComponent } from './user-card.component';
```

**Use path aliases for cleaner imports:**

```json
// tsconfig.json
{
  "compilerOptions": {
    "paths": {
      "@core/*": ["src/app/core/*"],
      "@shared/*": ["src/app/shared/*"],
      "@features/*": ["src/app/features/*"],
      "@models/*": ["src/app/models/*"]
    }
  }
}
```

```typescript
// Clean imports with aliases
import { AuthService } from '@core/services/auth.service';
import { ButtonComponent } from '@shared/components/button.component';
import { User } from '@models/user.model';
```

**ESLint import ordering:**

```json
{
  "rules": {
    "import/order": [
      "error",
      {
        "groups": [
          ["builtin", "external"],
          "internal",
          ["parent", "sibling", "index"]
        ],
        "pathGroups": [
          { "pattern": "@angular/**", "group": "external", "position": "before" }
        ],
        "newlines-between": "always",
        "alphabetize": { "order": "asc" }
      }
    ]
  }
}
```

Reference: [Angular Style Guide](https://angular.dev/style-guide)
