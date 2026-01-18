---
title: Centralize Validation Logic
impact: MEDIUM
impactDescription: Reusable validators, consistent error handling
tags: forms, validation, validators
---

## Centralize Validation Logic

**Impact: MEDIUM (reusable validators, consistent error handling)**

Create reusable validator functions and centralize error message generation for consistency across forms.

**Incorrect (inline validation, scattered error messages):**

```typescript
@Component({
  template: `
    <input formControlName="email" />
    @if (form.controls.email.hasError('required')) {
      <span>Email is required</span>
    }
    @if (form.controls.email.hasError('email')) {
      <span>Invalid email format</span>
    }
    @if (form.controls.email.hasError('emailTaken')) {
      <span>This email is already registered</span>
    }
  `
})
export class FormComponent {
  form = new FormGroup({
    email: new FormControl('', [
      Validators.required,
      Validators.email,
      // Inline async validator - hard to reuse
      (control) => this.http.get(`/api/check-email/${control.value}`)
        .pipe(map(taken => taken ? { emailTaken: true } : null))
    ])
  });
}
```

**Correct (centralized validators and error messages):**

```typescript
// validators/custom-validators.ts
export class CustomValidators {
  static strongPassword(control: AbstractControl): ValidationErrors | null {
    const value = control.value;
    const hasUpperCase = /[A-Z]/.test(value);
    const hasLowerCase = /[a-z]/.test(value);
    const hasNumber = /[0-9]/.test(value);
    const hasSpecial = /[!@#$%^&*]/.test(value);

    const valid = hasUpperCase && hasLowerCase && hasNumber && hasSpecial;
    return valid ? null : { strongPassword: true };
  }

  static matchField(fieldName: string): ValidatorFn {
    return (control: AbstractControl): ValidationErrors | null => {
      const parent = control.parent;
      if (!parent) return null;
      const field = parent.get(fieldName);
      return field?.value === control.value ? null : { mismatch: fieldName };
    };
  }
}

// validators/async-validators.ts
@Injectable({ providedIn: 'root' })
export class AsyncValidators {
  constructor(private http: HttpClient) {}

  emailAvailable(): AsyncValidatorFn {
    return (control) =>
      this.http.get<boolean>(`/api/check-email/${control.value}`).pipe(
        map(taken => taken ? { emailTaken: true } : null),
        catchError(() => of(null))
      );
  }
}

// utils/form-errors.ts
export const ERROR_MESSAGES: Record<string, (params?: any) => string> = {
  required: () => 'This field is required',
  email: () => 'Invalid email format',
  minlength: (p) => `Minimum ${p.requiredLength} characters`,
  strongPassword: () => 'Must include uppercase, lowercase, number, and special character',
  emailTaken: () => 'This email is already registered',
  mismatch: (p) => `Must match ${p}`
};

export function getErrorMessage(errors: ValidationErrors): string {
  const [key, params] = Object.entries(errors)[0];
  return ERROR_MESSAGES[key]?.(params) ?? 'Invalid value';
}

// component usage
@Component({
  template: `
    <input formControlName="email" />
    @if (form.controls.email.invalid && form.controls.email.touched) {
      <span class="error">{{ getError(form.controls.email) }}</span>
    }
  `
})
export class FormComponent {
  private asyncValidators = inject(AsyncValidators);

  form = new FormGroup({
    email: new FormControl('', {
      validators: [Validators.required, Validators.email],
      asyncValidators: [this.asyncValidators.emailAvailable()]
    }),
    password: new FormControl('', [Validators.required, CustomValidators.strongPassword]),
    confirmPassword: new FormControl('', [CustomValidators.matchField('password')])
  });

  getError(control: AbstractControl): string {
    return control.errors ? getErrorMessage(control.errors) : '';
  }
}
```

Reference: [Angular Form Validation](https://angular.dev/guide/forms)
