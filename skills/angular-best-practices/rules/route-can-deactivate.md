---
title: Use CanDeactivate to Prevent Unsaved Changes Loss
impact: MEDIUM
impactDescription: Protects user data, improves UX
tags: routing, guards, canDeactivate, forms
---

## Use CanDeactivate to Prevent Unsaved Changes Loss

**Impact: MEDIUM (protects user data, improves UX)**

CanDeactivate guards prevent users from accidentally navigating away from forms with unsaved changes.

**Incorrect (no protection for unsaved changes):**

```typescript
@Component({ ... })
export class ArticleEditComponent {
  form = inject(FormBuilder).group({
    title: [''],
    content: ['']
  });

  // User can navigate away and lose all changes
}
```

**Correct (CanDeactivate guard):**

```typescript
// interfaces/can-deactivate.interface.ts
export interface CanDeactivateComponent {
  canDeactivate(): Observable<boolean> | Promise<boolean> | boolean;
}

// guards/unsaved-changes.guard.ts
export const unsavedChangesGuard: CanDeactivateFn<CanDeactivateComponent> = (
  component,
  currentRoute,
  currentState,
  nextState
) => {
  return component.canDeactivate ? component.canDeactivate() : true;
};

// Alternative: with dialog confirmation
export const confirmLeaveGuard: CanDeactivateFn<CanDeactivateComponent> = (
  component,
  currentRoute,
  currentState,
  nextState
) => {
  const dialog = inject(MatDialog);

  if (!component.canDeactivate || component.canDeactivate()) {
    return true;
  }

  return dialog.open(ConfirmDialogComponent, {
    data: {
      title: 'Unsaved Changes',
      message: 'You have unsaved changes. Are you sure you want to leave?'
    }
  }).afterClosed().pipe(
    map(confirmed => confirmed ?? false)
  );
};

// article-edit.component.ts
@Component({
  selector: 'app-article-edit',
  template: `
    <form [formGroup]="form" (ngSubmit)="save()">
      <input formControlName="title" />
      <textarea formControlName="content"></textarea>
      <button type="submit" [disabled]="!form.dirty">Save</button>
    </form>
  `
})
export class ArticleEditComponent implements CanDeactivateComponent {
  private fb = inject(FormBuilder);
  private saved = false;

  form = this.fb.group({
    title: ['', Validators.required],
    content: ['', Validators.required]
  });

  canDeactivate(): boolean {
    // Allow leaving if form is pristine or was just saved
    return !this.form.dirty || this.saved;
  }

  save() {
    if (this.form.valid) {
      this.articleService.save(this.form.value).subscribe(() => {
        this.saved = true;
        this.router.navigate(['/articles']);
      });
    }
  }
}

// app.routes.ts
{
  path: 'articles/:id/edit',
  component: ArticleEditComponent,
  canDeactivate: [unsavedChangesGuard]
}
```

**With signal-based forms:**

```typescript
@Component({ ... })
export class ModernFormComponent implements CanDeactivateComponent {
  formDirty = signal(false);
  formSaved = signal(false);

  canDeactivate(): boolean {
    return !this.formDirty() || this.formSaved();
  }
}
```

Reference: [Angular CanDeactivate Guard](https://github.com/angular-vietnam/100-days-of-angular/blob/master/Day031-router-guards-resolvers-2.md)
