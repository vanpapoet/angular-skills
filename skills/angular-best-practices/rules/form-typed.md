---
title: Use Strongly Typed Forms
impact: MEDIUM
impactDescription: Compile-time type checking, better IDE support
tags: forms, typescript, type-safety
---

## Use Strongly Typed Forms

**Impact: MEDIUM (compile-time type checking, better IDE support)**

Angular's typed forms provide full TypeScript integration, catching errors at compile time and enabling better autocomplete.

**Incorrect (untyped form):**

```typescript
@Component({ ... })
export class LoginComponent {
  form = new FormGroup({
    email: new FormControl(''),
    password: new FormControl('')
  });

  onSubmit() {
    const email = this.form.value.email; // string | undefined - no type safety
    const data = this.form.value; // Partial<{email, password}> - might be undefined
  }
}
```

**Correct (fully typed form):**

```typescript
interface LoginForm {
  email: FormControl<string>;
  password: FormControl<string>;
  rememberMe: FormControl<boolean>;
}

@Component({ ... })
export class LoginComponent {
  form = new FormGroup<LoginForm>({
    email: new FormControl('', { nonNullable: true }),
    password: new FormControl('', { nonNullable: true }),
    rememberMe: new FormControl(false, { nonNullable: true })
  });

  onSubmit() {
    // Fully typed - no undefined values with nonNullable
    const { email, password, rememberMe } = this.form.getRawValue();
    // email: string, password: string, rememberMe: boolean
  }
}
```

**With FormBuilder (cleaner syntax):**

```typescript
@Component({ ... })
export class RegistrationComponent {
  private fb = inject(NonNullableFormBuilder);

  form = this.fb.group({
    username: ['', [Validators.required, Validators.minLength(3)]],
    email: ['', [Validators.required, Validators.email]],
    password: ['', [Validators.required, Validators.minLength(8)]],
    profile: this.fb.group({
      firstName: [''],
      lastName: [''],
      age: [0, [Validators.min(0), Validators.max(150)]]
    })
  });

  get profileGroup() {
    return this.form.controls.profile; // Typed as FormGroup
  }
}
```

**Type inference benefits:**

```typescript
// IDE autocomplete works correctly
this.form.controls.email.setValue('test@example.com');
this.form.controls.email.setValue(123); // TypeScript Error!

// Accessing nested controls
this.form.controls.profile.controls.firstName.value; // string
```

Reference: [Angular Typed Forms](https://angular.dev/guide/forms)
