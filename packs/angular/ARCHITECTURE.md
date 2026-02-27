# ARCHITECTURE.md -- Architecture & Patterns Guide

> This document is the **source of truth** for all subagents.
> Every architectural decision must be documented here.

---

## 1. Overview

We are migrating / building an Angular project following a modern, typed, modular architecture:

| From | To |
|------|----|
| JavaScript | TypeScript (strict) |
| NgModule-based components | Standalone components |
| @Input()/@Output() decorators | input()/output() signal-based I/O |
| Constructor DI | inject() function |
| RxJS BehaviorSubject stores | Signal-based stores (WritableSignal + computed) |
| Manual HTTP / inline transforms | Services + Adapters + HttpClient |
| Scattered try/catch | Centralized error handling via HttpInterceptor |
| Structure by file type | Modular architecture by feature |
| Class-heavy patterns | Functional + signal-based patterns |
| Inconsistent naming | Strict documented conventions |

### Core Stack

- **Angular 17+** with standalone components
- **TypeScript** (strict mode)
- **Angular Signals** for reactive state (signal, computed, effect)
- **HttpClient** for HTTP calls
- **Angular Router** with lazy loading
- **Zod** for runtime validation
- **Jest/Vitest** + **Angular Testing Library** for tests

### Data Flow

```
Service (HttpClient only) --> Adapter (parse) --> Injectable/Component (orchestrate) --> Component (template)
```

---

## 2. Directory Structure (Modular Architecture)

```
src/
+-- app/                          # Application shell
|   +-- app.component.ts
|   +-- app.config.ts
|   +-- app.routes.ts
|   +-- guards/
|   +-- interceptors/             # HttpInterceptors (auth, error handling)
|
+-- modules/                      # Feature modules (bounded contexts)
|   +-- auth/
|   |   +-- components/
|   |   +-- services/
|   |   +-- adapters/
|   |   +-- stores/
|   |   +-- types/
|   |   +-- pages/
|   |   +-- __tests__/
|   |   +-- index.ts              # Barrel export (public API)
|   |
|   +-- marketplace/
|   |   +-- components/
|   |   |   +-- marketplace-list.component.ts
|   |   |   +-- marketplace-card.component.ts
|   |   |   +-- marketplace-filters.component.ts
|   |   |   +-- marketplace-card/          # Sub-components if needed
|   |   |       +-- marketplace-card-header.component.ts
|   |   |       +-- marketplace-card-actions.component.ts
|   |   +-- services/
|   |   |   +-- marketplace.service.ts
|   |   +-- adapters/
|   |   |   +-- marketplace.adapter.ts
|   |   +-- stores/
|   |   |   +-- marketplace.store.ts       # Client state only (filters, UI)
|   |   +-- types/
|   |   |   +-- marketplace.types.ts
|   |   |   +-- marketplace.contracts.ts   # Zod schemas / app contracts
|   |   +-- pages/
|   |   |   +-- marketplace-page.component.ts
|   |   +-- __tests__/
|   |   +-- index.ts
|   |
|   +-- [other-module]/
|
+-- shared/                       # Shared across modules
|   +-- components/               # Generic components (Button, Modal, Table)
|   +-- services/                 # Base API client, interceptors
|   +-- adapters/                 # Shared adapters
|   +-- types/                    # Global types
|   +-- utils/                    # Pure functions, no side effects
|   +-- helpers/                  # Functions with side effects or DOM
|   +-- constants/                # Static values
|   +-- pipes/                    # Angular pipes
|   +-- directives/               # Angular directives
|
+-- assets/                       # Static assets (images, fonts, global CSS)
```

### Import Rules Between Layers

```
modules/auth  <->  shared/          OK  Module imports from shared
modules/auth  -->  modules/market   NO  Module does NOT import from another module
shared/       -->  modules/auth     NO  Shared does NOT import from modules
app/          -->  modules/*        OK  App imports modules (routes, providers)
```

If two modules need to share something, move it to `shared/`.

---

## 3. Naming Conventions

### Files and Directories

| Type | Pattern | Example |
|------|---------|---------|
| Directories | `kebab-case` | `user-settings/` |
| Components | `kebab-case.component.ts` | `user-settings-form.component.ts` |
| Pages | `kebab-case-page.component.ts` | `marketplace-page.component.ts` |
| Services | `kebab-case.service.ts` | `marketplace.service.ts` |
| Adapters | `kebab-case.adapter.ts` | `marketplace.adapter.ts` |
| Stores | `kebab-case.store.ts` | `marketplace.store.ts` |
| Types | `kebab-case.types.ts` | `marketplace.types.ts` |
| Contracts/Schemas | `kebab-case.contracts.ts` | `marketplace.contracts.ts` |
| Utils | `kebab-case.ts` | `format-date.ts` |
| Helpers | `kebab-case.ts` | `clipboard-helper.ts` |
| Tests | `kebab-case.spec.ts` | `marketplace-list.component.spec.ts` |
| Constants | `kebab-case.constants.ts` | `api-endpoints.constants.ts` |
| Pipes | `kebab-case.pipe.ts` | `currency-format.pipe.ts` |
| Directives | `kebab-case.directive.ts` | `click-outside.directive.ts` |
| Guards | `kebab-case.guard.ts` | `auth.guard.ts` |
| Interceptors | `kebab-case.interceptor.ts` | `error-handler.interceptor.ts` |

### Code

| Type | Pattern | Example |
|------|---------|---------|
| Variables / functions | `camelCase` | `getUserById`, `isLoading` |
| Classes / Components | `PascalCase` | `UserProfile`, `MarketplaceListComponent` |
| Enums | `PascalCase` | `UserRole.Admin` |
| Constants | `UPPER_SNAKE_CASE` | `API_BASE_URL`, `MAX_RETRIES` |
| Services | `PascalCase + Service` | `MarketplaceService` |
| Stores | `PascalCase + Store` | `MarketplaceStore` |
| Selectors | `kebab-case` | `'app-marketplace-list'` |
| Event handlers | `handle` + action | `handleSubmit`, `handleDelete` |
| Boolean | `is`/`has`/`can`/`should` | `isLoading`, `hasPermission` |

---

## 4. Responsibility Layers

### 4.1 Services -- Pure HTTP Requests

Services make **only** the HTTP request via HttpClient. No try/catch, no transformation, no business logic.

```typescript
// services/marketplace.service.ts
import { Injectable, inject } from '@angular/core'
import { HttpClient } from '@angular/common/http'
import type {
  MarketplaceListResponse,
  MarketplaceItemResponse,
  CreateMarketplacePayload,
} from '../types/marketplace.types'

@Injectable({ providedIn: 'root' })
export class MarketplaceService {
  private readonly http = inject(HttpClient)
  private readonly baseUrl = '/api/marketplace'

  list(params: { page: number; pageSize: number; search?: string }) {
    return this.http.get<MarketplaceListResponse>(this.baseUrl, { params: { ...params } })
  }

  getById(id: string) {
    return this.http.get<MarketplaceItemResponse>(`${this.baseUrl}/${id}`)
  }

  create(payload: CreateMarketplacePayload) {
    return this.http.post<MarketplaceItemResponse>(this.baseUrl, payload)
  }

  delete(id: string) {
    return this.http.delete(`${this.baseUrl}/${id}`)
  }
}
```

**Service Rules:**
- NO try/catch (caller handles errors)
- NO data transformation (adapter does that)
- NO business logic
- NO store or signal access
- YES HttpClient calls with typed request/response
- YES one file per domain/resource
- YES `@Injectable({ providedIn: 'root' })` with `inject()`

### 4.2 Adapters -- Contract Parsers

Adapters transform API data to the application's TypeScript contract (and vice-versa). They are **pure functions** with no side effects.

```typescript
// adapters/marketplace.adapter.ts
import type {
  MarketplaceListResponse,
  MarketplaceItemResponse,
} from '../types/marketplace.types'
import type {
  MarketplaceItem,
  MarketplaceList,
} from '../types/marketplace.contracts'

export const marketplaceAdapter = {
  /**
   * API response -> App contract (inbound)
   */
  toMarketplaceItem(response: MarketplaceItemResponse): MarketplaceItem {
    return {
      id: response.uuid,
      name: response.name,
      vendor: response.vendor_name,
      category: response.category_slug,
      price: response.price_cents / 100,
      isActive: response.status === 'active',
      createdAt: new Date(response.created_at),
      updatedAt: new Date(response.updated_at),
    }
  },

  toMarketplaceList(response: MarketplaceListResponse): MarketplaceList {
    return {
      items: response.results.map(marketplaceAdapter.toMarketplaceItem),
      totalItems: response.count,
      totalPages: Math.ceil(response.count / response.page_size),
      currentPage: response.page,
    }
  },

  /**
   * App contract -> API payload (outbound)
   */
  toCreatePayload(item: CreateMarketplaceInput): CreateMarketplacePayload {
    return {
      name: item.name,
      vendor_name: item.vendor,
      category_slug: item.category,
      price_cents: Math.round(item.price * 100),
    }
  },
}
```

**Adapter Rules:**
- YES pure functions (input -> output, no side effects)
- YES two directions: API->App (inbound) and App->API (outbound)
- YES rename fields (snake_case API -> camelCase App)
- YES convert types (string->Date, cents->decimal, status->boolean)
- NO HTTP calls
- NO store/service access
- NO try/catch (failure = wrong type = bug to fix)

### 4.3 Types & Contracts

```typescript
// types/marketplace.types.ts
// <-- Types that mirror the API exactly as it returns (snake_case)

export interface MarketplaceItemResponse {
  uuid: string
  name: string
  vendor_name: string
  category_slug: string
  price_cents: number
  status: 'active' | 'inactive' | 'pending'
  created_at: string
  updated_at: string
}

export interface MarketplaceListResponse {
  count: number
  page: number
  page_size: number
  results: MarketplaceItemResponse[]
}

export interface CreateMarketplacePayload {
  name: string
  vendor_name: string
  category_slug: string
  price_cents: number
}

// types/marketplace.contracts.ts
// <-- Application contracts (camelCase, correct types)

export interface MarketplaceItem {
  id: string
  name: string
  vendor: string
  category: string
  price: number          // in currency, not cents
  isActive: boolean      // derived from status
  createdAt: Date
  updatedAt: Date
}

export interface MarketplaceList {
  items: MarketplaceItem[]
  totalItems: number
  totalPages: number
  currentPage: number
}

export interface CreateMarketplaceInput {
  name: string
  vendor: string
  category: string
  price: number
}
```

### 4.4 Signal Stores -- Client State Only

Signal stores manage state that **does not come from the server**: UI state, filters, preferences, auth tokens.

```typescript
// stores/marketplace.store.ts
import { Injectable, signal, computed } from '@angular/core'

@Injectable({ providedIn: 'root' })
export class MarketplaceStore {
  // State (private writable signals)
  private readonly _selectedCategory = signal<string | null>(null)
  private readonly _viewMode = signal<'grid' | 'list'>('grid')
  private readonly _searchQuery = signal('')

  // Public read-only signals
  readonly selectedCategory = this._selectedCategory.asReadonly()
  readonly viewMode = this._viewMode.asReadonly()
  readonly searchQuery = this._searchQuery.asReadonly()

  // Computed
  readonly hasActiveFilters = computed(() =>
    !!this._selectedCategory() || !!this._searchQuery()
  )

  // Actions
  setCategory(category: string | null) {
    this._selectedCategory.set(category)
  }

  setViewMode(mode: 'grid' | 'list') {
    this._viewMode.set(mode)
  }

  setSearchQuery(query: string) {
    this._searchQuery.set(query)
  }

  clearFilters() {
    this._selectedCategory.set(null)
    this._searchQuery.set('')
  }
}
```

**Store Rules:**
- YES client state only (UI, filters, preferences, session)
- YES `@Injectable({ providedIn: 'root' })` with signals
- YES private WritableSignal, public asReadonly()
- YES computed() for derived values
- NO server state (API data is fetched via services and managed locally)
- NO HTTP calls inside stores
- NO complex business logic (services or components handle that)

### 4.5 Components -- Standalone with Signals

```typescript
// components/marketplace-list.component.ts
import { Component, inject, input, output, signal, computed } from '@angular/core'
import { MarketplaceService } from '../services/marketplace.service'
import { marketplaceAdapter } from '../adapters/marketplace.adapter'
import { MarketplaceStore } from '../stores/marketplace.store'
import { MarketplaceCardComponent } from './marketplace-card.component'
import type { MarketplaceItem } from '../types/marketplace.contracts'

@Component({
  selector: 'app-marketplace-list',
  standalone: true,
  imports: [MarketplaceCardComponent],
  template: `
    @if (isLoading()) {
      <div>Loading...</div>
    } @else if (isEmpty()) {
      <div>No items found</div>
    } @else {
      @for (item of items(); track item.id) {
        <app-marketplace-card
          [item]="item"
          [isSelected]="item.id === selectedId()"
          (select)="handleSelect($event)"
        />
      }
    }
  `,
})
export class MarketplaceListComponent {
  // Inputs / Outputs (signal-based)
  readonly categoryFilter = input<string>()
  readonly select = output<MarketplaceItem>()

  // Injected dependencies
  private readonly service = inject(MarketplaceService)
  private readonly store = inject(MarketplaceStore)

  // Local state
  readonly page = signal(1)
  readonly selectedId = signal<string | null>(null)
  readonly items = signal<MarketplaceItem[]>([])
  readonly isLoading = signal(false)

  // Computed
  readonly isEmpty = computed(() => !this.isLoading() && this.items().length === 0)
  readonly isFirstPage = computed(() => this.page() === 1)

  // Handlers
  handleSelect(item: MarketplaceItem) {
    this.selectedId.set(item.id)
    this.select.emit(item)
  }

  handlePageChange(newPage: number) {
    this.page.set(newPage)
  }
}
```

---

## 5. Component Patterns

### 5.1 Stop Prop Drilling -- Component Composition

**BAD -- Prop Drilling:**
```html
<!-- GrandParent passes data through 3 levels -->
<app-parent [user]="user" [theme]="theme" [permissions]="permissions">
  <app-child [user]="user" [theme]="theme" [permissions]="permissions">
    <app-grandchild [user]="user" [permissions]="permissions" />
  </app-child>
</app-parent>
```

**GOOD -- Composition with content projection:**
```typescript
// marketplace-page.component.ts
@Component({
  selector: 'app-marketplace-page',
  standalone: true,
  imports: [PageLayoutComponent, MarketplaceFiltersComponent, MarketplaceListComponent, MarketplaceDetailsComponent],
  template: `
    <app-page-layout>
      <ng-container header>
        <app-marketplace-filters />
      </ng-container>

      <app-marketplace-list (select)="handleSelect($event)" />

      @if (selectedItem()) {
        <ng-container sidebar>
          <app-marketplace-details [item]="selectedItem()!" />
        </ng-container>
      }
    </app-page-layout>
  `,
})
export class MarketplacePageComponent {
  readonly selectedItem = signal<MarketplaceItem | null>(null)

  handleSelect(item: MarketplaceItem) {
    this.selectedItem.set(item)
  }
}
```

**GOOD -- Shared injectable for cross-component state:**
```typescript
// stores/marketplace-context.store.ts
import { Injectable, signal, computed } from '@angular/core'
import type { MarketplaceItem } from '../types/marketplace.contracts'

@Injectable()
export class MarketplaceContextStore {
  private readonly _selectedItem = signal<MarketplaceItem | null>(null)

  readonly selectedItem = this._selectedItem.asReadonly()
  readonly hasSelection = computed(() => this._selectedItem() !== null)

  selectItem(item: MarketplaceItem) {
    this._selectedItem.set(item)
  }

  clearSelection() {
    this._selectedItem.set(null)
  }
}

// Provide in the page component:
@Component({
  providers: [MarketplaceContextStore],
  // ...
})
export class MarketplacePageComponent {
  private readonly context = inject(MarketplaceContextStore)
}
```

### 5.2 Component Hierarchy

```
Pages                 --> Composition, orchestration, provide context
  +-- Layout          --> Visual structure (content projection)
      +-- Features    --> Feature logic (services, stores, signals)
          +-- Shared  --> Pure presentation (inputs in, outputs out)
```

| Type | Responsibility | Can have logic? | Can have state? |
|------|---------------|-----------------|-----------------|
| **Pages** | Compose components, provide context | Via services/stores | Yes (signals) |
| **Feature Components** | UI + feature logic | Via services/stores | Yes (signals) |
| **Shared Components** | Generic, reusable UI | Minimal (UI only) | Minimal (local) |

---

## 6. Utils vs Helpers

### Utils -- Pure Functions
- No side effects
- Deterministic input -> output
- Testable without mocks
- No dependency on DOM, browser APIs, or Angular

```typescript
// shared/utils/format-date.ts
export function formatDate(date: Date, locale = 'en-US'): string {
  return new Intl.DateTimeFormat(locale).format(date)
}

// shared/utils/currency.ts
export function formatCurrency(value: number, currency = 'USD'): string {
  return new Intl.NumberFormat('en-US', { style: 'currency', currency }).format(value)
}

// shared/utils/string.ts
export function slugify(text: string): string {
  return text.toLowerCase().replace(/\s+/g, '-').replace(/[^\w-]/g, '')
}
```

### Helpers -- Functions with Side Effects or DOM
- Interact with browser APIs (clipboard, localStorage, DOM)
- May have side effects
- May need mocks in tests

```typescript
// shared/helpers/clipboard-helper.ts
export async function copyToClipboard(text: string): Promise<boolean> {
  try {
    await navigator.clipboard.writeText(text)
    return true
  } catch {
    return false
  }
}

// shared/helpers/download-helper.ts
export function downloadAsFile(content: string, filename: string): void {
  const blob = new Blob([content], { type: 'text/plain' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = filename
  a.click()
  URL.revokeObjectURL(url)
}

// shared/helpers/storage-helper.ts
export function getStorageItem<T>(key: string, fallback: T): T {
  try {
    const item = localStorage.getItem(key)
    return item ? (JSON.parse(item) as T) : fallback
  } catch {
    return fallback
  }
}
```

---

## 7. Error Handling -- Centralized Pattern

### Global HttpInterceptor

```typescript
// app/interceptors/error-handler.interceptor.ts
import { HttpInterceptorFn, HttpErrorResponse } from '@angular/common/http'
import { inject } from '@angular/core'
import { Router } from '@angular/router'
import { catchError, throwError } from 'rxjs'

export const errorHandlerInterceptor: HttpInterceptorFn = (req, next) => {
  const router = inject(Router)

  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      // Only redirect on 401 (session expired)
      if (error.status === 401) {
        router.navigate(['/login'])
      }
      // DO NOT handle other errors here -- let the caller handle them
      return throwError(() => error)
    }),
  )
}
```

### Service-Level Error Handling (in components)

```typescript
// In the component that subscribes:
this.service.delete(id).subscribe({
  next: () => {
    // success: refresh data, show toast
  },
  error: (error) => {
    // handle error: show toast with parseApiError(error)
  },
})
```

### Centralized Error Parser

```typescript
// shared/utils/parse-api-error.ts
import { HttpErrorResponse } from '@angular/common/http'

interface ApiErrorResponse {
  message?: string
  detail?: string
  errors?: Record<string, string[]>
}

export function parseApiError(error: unknown): string {
  if (error instanceof HttpErrorResponse) {
    const data = error.error as ApiErrorResponse | undefined
    if (data?.message) return data.message
    if (data?.detail) return data.detail
    if (error.status === 403) return 'Permission denied'
    if (error.status === 404) return 'Resource not found'
    if (error.status === 500) return 'Internal server error'
  }
  return 'Unexpected error. Please try again.'
}
```

---

## 8. Barrel Exports (index.ts)

Each module exports only its **public API**:

```typescript
// modules/marketplace/index.ts

// Pages (for the router)
export { MarketplacePageComponent } from './pages/marketplace-page.component'

// Components reusable by others (rare, avoid)
export { MarketplaceCardComponent } from './components/marketplace-card.component'

// Types (for consumers that need typing)
export type { MarketplaceItem, MarketplaceList } from './types/marketplace.contracts'

// DO NOT export:
// - services (internal detail)
// - adapters (internal detail)
// - stores (internal detail)
// - internal components
```

---

## 9. SOLID Principles Applied to Angular

| Principle | Angular Application |
|-----------|---------------------|
| **S**ingle Responsibility | 1 component = 1 responsibility. 1 service = 1 resource. 1 store = 1 domain. |
| **O**pen/Closed | Components extensible via content projection (ng-content) and inputs -- not by internal modification. |
| **L**iskov Substitution | Shared components must work in any context without breaking. |
| **I**nterface Segregation | Specific inputs, not generic objects. `<app-avatar [src]="src" [alt]="alt" />` not `<app-avatar [user]="user" />`. |
| **D**ependency Inversion | Components depend on abstractions (injection tokens), not concrete implementations. Services injected via inject(), never instantiated directly. |

### SOLID Examples

**Single Responsibility:**
```typescript
// BAD: component doing too much
@Component({ /* ... */ })
export class UserDashboardComponent {
  // fetching data, filtering, rendering chart, rendering table, handling forms...
}

// GOOD: composed from focused components
@Component({
  template: `
    <app-dashboard-layout>
      <app-user-stats />
      <app-user-activity-chart />
      <app-user-table />
    </app-dashboard-layout>
  `,
})
export class UserDashboardComponent {}
```

**Open/Closed:**
```typescript
// GOOD: extensible via content projection and inputs
@Component({
  selector: 'app-data-table',
  template: `
    @if (items().length === 0) {
      <ng-content select="[empty]" />
    } @else {
      <table>
        <ng-content select="[header]" />
        <tbody>
          @for (item of items(); track trackFn(item)) {
            <ng-content />
          }
        </tbody>
      </table>
    }
  `,
})
export class DataTableComponent<T> {
  readonly items = input.required<T[]>()
  readonly trackFn = input.required<(item: T) => unknown>()
}
```

**Interface Segregation:**
```typescript
// BAD: passing entire object when only a few fields are needed
@Component({ /* ... */ })
export class AvatarComponent {
  readonly user = input.required<User>()  // component only uses name and avatarUrl
}

// GOOD: specific inputs
@Component({ /* ... */ })
export class AvatarComponent {
  readonly src = input.required<string>()
  readonly alt = input.required<string>()
  readonly size = input<'sm' | 'md' | 'lg'>('md')
}
```

---

## 10. Migration Checklist per File

### Component (.component.ts)
- [ ] Standalone component (no NgModule)
- [ ] input() / output() signal-based I/O
- [ ] inject() instead of constructor DI
- [ ] No prop drilling (use content projection / shared injectable)
- [ ] < 200 lines total
- [ ] Template < 100 lines
- [ ] No business logic in template
- [ ] Loading / error / empty states
- [ ] OnPush change detection strategy
- [ ] trackBy function for @for loops

### Service (.service.ts)
- [ ] HttpClient calls only
- [ ] No try/catch
- [ ] No data transformation
- [ ] Typed request and response
- [ ] inject(HttpClient) instead of constructor
- [ ] `@Injectable({ providedIn: 'root' })`

### Adapter (.adapter.ts)
- [ ] Pure functions
- [ ] Inbound (API->App) and Outbound (App->API)
- [ ] Naming conversion (snake_case -> camelCase)
- [ ] Type conversion (string->Date, etc.)

### Store (.store.ts)
- [ ] Client state only
- [ ] Private WritableSignal, public asReadonly()
- [ ] computed() for derived values
- [ ] inject() for dependencies
- [ ] No HTTP calls

### Types
- [ ] `.types.ts` for API types (raw response)
- [ ] `.contracts.ts` for application contracts
- [ ] No `any`
