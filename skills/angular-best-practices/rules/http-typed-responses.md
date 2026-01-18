---
title: Type HTTP Responses
impact: MEDIUM
impactDescription: Compile-time type checking, better IDE support
tags: http, typescript, type-safety
---

## Type HTTP Responses

**Impact: MEDIUM (compile-time type checking, better IDE support)**

Always specify response types for HTTP requests to catch errors at compile time and enable IDE autocomplete.

**Incorrect (untyped responses):**

```typescript
@Injectable({ providedIn: 'root' })
export class UserService {
  private http = inject(HttpClient);

  getUsers() {
    return this.http.get('/api/users'); // Observable<Object> - no type info
  }

  getUser(id: string) {
    return this.http.get(`/api/users/${id}`).pipe(
      map(response => response.name) // Error: 'name' doesn't exist on type Object
    );
  }
}
```

**Correct (typed responses):**

```typescript
// models/user.model.ts
export interface User {
  id: string;
  name: string;
  email: string;
  role: 'admin' | 'user';
  createdAt: string;
}

export interface UserListResponse {
  users: User[];
  total: number;
  page: number;
  pageSize: number;
}

export interface CreateUserRequest {
  name: string;
  email: string;
  password: string;
}

// services/user.service.ts
@Injectable({ providedIn: 'root' })
export class UserService {
  private http = inject(HttpClient);
  private apiUrl = inject(API_URL);

  getUsers(page = 1): Observable<UserListResponse> {
    return this.http.get<UserListResponse>(`${this.apiUrl}/users`, {
      params: { page: page.toString() }
    });
  }

  getUser(id: string): Observable<User> {
    return this.http.get<User>(`${this.apiUrl}/users/${id}`);
  }

  createUser(data: CreateUserRequest): Observable<User> {
    return this.http.post<User>(`${this.apiUrl}/users`, data);
  }

  updateUser(id: string, data: Partial<User>): Observable<User> {
    return this.http.patch<User>(`${this.apiUrl}/users/${id}`, data);
  }

  deleteUser(id: string): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/users/${id}`);
  }
}
```

**With response transformation:**

```typescript
interface ApiResponse<T> {
  data: T;
  meta: { timestamp: string };
}

@Injectable({ providedIn: 'root' })
export class ApiService {
  private http = inject(HttpClient);

  get<T>(url: string): Observable<T> {
    return this.http.get<ApiResponse<T>>(url).pipe(
      map(response => response.data) // Typed extraction
    );
  }
}
```

Reference: [Angular HTTP Client](https://angular.dev/guide/http)
