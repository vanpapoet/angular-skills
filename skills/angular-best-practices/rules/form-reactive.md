---
title: Prefer Reactive Forms for Complex Scenarios
impact: MEDIUM
impactDescription: Better scalability, testability, and type safety
tags: forms, reactive-forms, validation
---

## Prefer Reactive Forms for Complex Scenarios

**Impact: MEDIUM (better scalability, testability, and type safety)**

Reactive forms provide explicit, immutable form models with synchronous data flow. They're more scalable and testable than template-driven forms.

**Incorrect (template-driven for complex form):**

```typescript
@Component({
  template: `
    <form #form="ngForm">
      <input [(ngModel)]="user.name" required />
      <input [(ngModel)]="user.email" email required />
      <input [(ngModel)]="user.password" minlength="8" />

      @for (address of user.addresses; track $index) {
        <input [(ngModel)]="address.street" />
        <input [(ngModel)]="address.city" />
      }
    </form>
  `
})
export class UserFormComponent {
  user = { name: '', email: '', password: '', addresses: [] };
}
```

**Correct (reactive form with strong typing):**

```typescript
interface UserForm {
  name: FormControl<string>;
  email: FormControl<string>;
  password: FormControl<string>;
  addresses: FormArray<FormGroup<AddressForm>>;
}

interface AddressForm {
  street: FormControl<string>;
  city: FormControl<string>;
}

@Component({
  imports: [ReactiveFormsModule],
  template: `
    <form [formGroup]="form" (ngSubmit)="onSubmit()">
      <input formControlName="name" />
      @if (form.controls.name.errors?.['required']) {
        <span class="error">Name is required</span>
      }

      <input formControlName="email" />
      <input formControlName="password" type="password" />

      <div formArrayName="addresses">
        @for (address of form.controls.addresses.controls; track $index) {
          <div [formGroupName]="$index">
            <input formControlName="street" />
            <input formControlName="city" />
          </div>
        }
      </div>

      <button type="submit" [disabled]="form.invalid">Submit</button>
    </form>
  `
})
export class UserFormComponent {
  private fb = inject(FormBuilder);

  form = this.fb.group<UserForm>({
    name: this.fb.control('', { nonNullable: true, validators: [Validators.required] }),
    email: this.fb.control('', { nonNullable: true, validators: [Validators.required, Validators.email] }),
    password: this.fb.control('', { nonNullable: true, validators: [Validators.minLength(8)] }),
    addresses: this.fb.array<FormGroup<AddressForm>>([])
  });

  addAddress() {
    this.form.controls.addresses.push(
      this.fb.group<AddressForm>({
        street: this.fb.control('', { nonNullable: true }),
        city: this.fb.control('', { nonNullable: true })
      })
    );
  }

  onSubmit() {
    if (this.form.valid) {
      const value = this.form.getRawValue(); // Fully typed
      this.save(value);
    }
  }
}
```

Reference: [Angular Forms Guide](https://angular.dev/guide/forms)
