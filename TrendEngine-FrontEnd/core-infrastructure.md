# Core Infrastructure Module

## Overview

The **core-infrastructure** module provides the foundational layer for the Trend Engine application, establishing the essential build-time and runtime infrastructure. This module encompasses environment configuration, build tooling, type definitions, and global augmentations that enable the entire application ecosystem to function cohesively.

This module serves as the bedrock upon which all other modules ([ui-component-system](ui-component-system.md), [business-components](business-components.md), [view-modules](view-modules.md), [state-management-hooks](state-management-hooks.md), and [utilities-helpers](utilities-helpers.md)) are built, providing:

- **Environment Configuration**: Type-safe environment variable management for Vite-based builds
- **Build-Time Route Generation**: Convention-based routing system with automatic route discovery
- **Global Type Augmentations**: Enhanced TypeScript definitions for Window, Array, and third-party libraries
- **Development Infrastructure**: Plugin system for build optimization and developer experience

---

## Architecture Overview

```mermaid
graph TB
    subgraph "Core Infrastructure Layer"
        ENV[Environment Configuration<br/>env.d.ts]
        VITE_ENV[Global Type Augmentations<br/>vite-env.d.ts]
        ROUTE_GEN[Route Generation Plugin<br/>vite-plugin-generate-routes.ts]
    end
    
    subgraph "Build System"
        VITE[Vite Build Tool]
        PLUGIN_SYS[Plugin System]
    end
    
    subgraph "Runtime Environment"
        WINDOW[Window Object]
        GLOBALS[Global Variables]
        I18N[i18n Instance]
        AXIOS[Axios Instance]
    end
    
    subgraph "Application Modules"
        UI[UI Component System]
        BC[Business Components]
        VIEWS[View Modules]
        HOOKS[State Management]
        UTILS[Utilities]
    end
    
    ENV --> VITE
    ROUTE_GEN --> PLUGIN_SYS
    PLUGIN_SYS --> VITE
    VITE_ENV --> WINDOW
    VITE_ENV --> GLOBALS
    
    WINDOW --> I18N
    WINDOW --> AXIOS
    
    VITE --> UI
    VITE --> BC
    VITE --> VIEWS
    VITE --> HOOKS
    VITE --> UTILS
    
    GLOBALS --> UI
    GLOBALS --> BC
    GLOBALS --> VIEWS
    GLOBALS --> HOOKS
    GLOBALS --> UTILS
    
    ROUTE_GEN -.generates.-> VIEWS
    
    style ENV fill:#e1f5ff
    style VITE_ENV fill:#e1f5ff
    style ROUTE_GEN fill:#fff4e1
    style WINDOW fill:#f0f0f0
    style GLOBALS fill:#f0f0f0
```

---

## Core Components

### 1. Environment Configuration (`env.d.ts`)

Defines type-safe environment variables for the Vite build system, ensuring compile-time validation of configuration values.

#### Component Structure

```mermaid
classDiagram
    class ImportMetaEnv {
        <<interface>>
        +readonly VITE_DOMAIN: string
        +readonly VITE_TENANT_ID: string
        +readonly VITE_CLIENT_ID: string
        +readonly [key: string]: string
    }
    
    class ImportMeta {
        <<interface>>
        +readonly env: ImportMetaEnv
    }
    
    class Record~string, string~ {
        <<built-in>>
    }
    
    ImportMetaEnv --|> Record~string, string~
    ImportMeta --> ImportMetaEnv
```

#### Key Features

- **Type Safety**: Strongly typed environment variables prevent runtime errors
- **Readonly Properties**: Immutable configuration values ensure consistency
- **Extensibility**: Inherits from `Record<string, string>` for additional variables
- **Build-Time Validation**: Vite validates environment variables during build

#### Environment Variables

| Variable | Type | Purpose |
|----------|------|---------|
| `VITE_DOMAIN` | string | Application domain/base URL |
| `VITE_TENANT_ID` | string | Multi-tenant identifier for Azure AD |
| `VITE_CLIENT_ID` | string | OAuth client identifier |

#### Usage Pattern

```typescript
// Accessing environment variables with type safety
const domain = import.meta.env.VITE_DOMAIN
const tenantId = import.meta.env.VITE_TENANT_ID
const clientId = import.meta.env.VITE_CLIENT_ID
```

---

### 2. Route Generation Plugin (`vite-plugin-generate-routes.ts`)

A sophisticated Vite plugin that implements convention-based routing with automatic route discovery, lazy loading, and error boundary support.

#### Architecture

```mermaid
graph LR
    subgraph "File System"
        FS[View Directory Structure]
        INDEX[index.*.tsx files]
    end
    
    subgraph "Route Generation Pipeline"
        GLOB[File Discovery<br/>globSync]
        PARSE[Path Parsing<br/>convertArgs]
        BUILD[Route Tree Builder<br/>conventionRouter]
        STRINGIFY[Code Generation<br/>stringifyRoutes]
        FORMAT[ESLint Formatting<br/>formatContent]
    end
    
    subgraph "Output"
        ROUTES[generatedRoutes.tsx]
    end
    
    subgraph "Watch System"
        WATCH[File Watcher<br/>watchDir]
        REGEN[Regenerate on Change]
    end
    
    FS --> GLOB
    INDEX --> GLOB
    GLOB --> PARSE
    PARSE --> BUILD
    BUILD --> STRINGIFY
    STRINGIFY --> FORMAT
    FORMAT --> ROUTES
    
    WATCH --> FS
    WATCH --> REGEN
    REGEN --> GLOB
    
    style BUILD fill:#fff4e1
    style ROUTES fill:#e1ffe1
```

#### Core Data Structures

```mermaid
classDiagram
    class RouteObject {
        +string id
        +string path
        +string element
        +boolean? index
        +RouteObject[]? children
    }
    
    class RouteMap {
        +[key: string]: RouteObject
    }
    
    class GlobResult {
        +[key: string]: string
    }
    
    RouteMap --> RouteObject
    RouteObject --> RouteObject : children
```

#### Convention-Based Routing Rules

```mermaid
flowchart TD
    START[File: index.*.tsx] --> CHECK_ARGS{Parse Filename Args}
    
    CHECK_ARGS -->|default| DEFAULT[Index Route]
    CHECK_ARGS -->|"[param]"| REQUIRED[Required Parameter<br/>:param]
    CHECK_ARGS -->|"(param)"| OPTIONAL[Optional Parameter<br/>:param?]
    CHECK_ARGS -->|none| STANDARD[Standard Route]
    
    DEFAULT --> PARENT{Has Parent?}
    REQUIRED --> PARENT
    OPTIONAL --> PARENT
    STANDARD --> PARENT
    
    PARENT -->|Yes| NESTED[Add to Parent Children]
    PARENT -->|No| ROOT[Add to Root Routes]
    
    NESTED --> LAZY[Wrap with Lazy Loading]
    ROOT --> LAZY
    
    LAZY --> LOADER{Has Loader?}
    LOADER -->|Yes| DEFER[Add Defer Loader]
    LOADER -->|No| SIMPLE[Simple Lazy Load]
    
    DEFER --> OUTPUT[Route Object]
    SIMPLE --> OUTPUT
    
    style DEFAULT fill:#e1f5ff
    style REQUIRED fill:#ffe1e1
    style OPTIONAL fill:#fff4e1
    style DEFER fill:#e1ffe1
```

#### File Naming Conventions

| Pattern | Example | Generated Route | Description |
|---------|---------|-----------------|-------------|
| `index.tsx` | `view/dashboard/index.tsx` | `/view/dashboard` | Standard route |
| `index.default.tsx` | `view/home/index.default.tsx` | `/view/home` + index | Default child route |
| `index.[id].tsx` | `view/product/index.[id].tsx` | `/view/product/:id` | Required parameter |
| `index.(id).tsx` | `view/user/index.(id).tsx` | `/view/user/:id?` | Optional parameter |
| `index.[id].default.tsx` | `view/detail/index.[id].default.tsx` | `/view/detail/:id` + index | Parameterized default |

#### Route Generation Process

```mermaid
sequenceDiagram
    participant FS as File System
    participant Plugin as Vite Plugin
    participant Glob as globSync
    participant Conv as conventionRouter
    participant Gen as stringifyRoutes
    participant ESLint as ESLint Formatter
    participant Output as generatedRoutes.tsx
    
    Plugin->>FS: Watch directory
    FS->>Plugin: File change detected
    Plugin->>Glob: Scan for index.*.tsx
    Glob->>Glob: Traverse directories
    Glob-->>Conv: Return file paths
    
    Conv->>Conv: Sort by depth
    loop For each file
        Conv->>Conv: Parse filename args
        Conv->>Conv: Build route tree
        Conv->>Conv: Handle parent-child
    end
    
    Conv-->>Gen: Route tree
    Gen->>Gen: Generate TypeScript code
    Gen-->>ESLint: Raw code string
    ESLint->>ESLint: Format & fix
    ESLint-->>Output: Formatted code
    
    Output->>Output: Write if changed
    
    Note over Plugin,Output: Hot reload triggers in dev mode
```

#### Generated Route Structure

The plugin generates a TypeScript file with the following structure:

```typescript
// Generated structure (simplified)
import { DeferLoader, defineDeferLoader } from "./bc/defer-loader.tsx"
import { Text } from "./component/index.ts"

function lazyLoad(r: Promise<any>) {
  return async () => {
    const res = await r
    const Component = res.default
    if (!res.loader) return { Component, ErrorBoundary: res.ErrorBoundary }
    const HydrateFallback = res.HydrateFallback
    return {
      element: (
        <DeferLoader fallback={HydrateFallback ? <HydrateFallback /> : <Text t="text.loading" />}>
          <Component />
        </DeferLoader>
      ),
      loader: defineDeferLoader(res.loader),
      ErrorBoundary: res.ErrorBoundary,
    }
  }
}

const routes = [
  {
    path: "/view/dashboard",
    id: "view/dashboard",
    lazy: lazyLoad(import("../view/dashboard/index.tsx")),
    children: []
  },
  // ... more routes
]

export default routes
```

#### Key Features

- **Automatic Discovery**: Scans directory structure for route files
- **Lazy Loading**: Code-splitting for optimal performance
- **Defer Loader Support**: Handles async data loading with fallbacks
- **Error Boundaries**: Per-route error handling
- **Hot Reload**: Watches file system for changes in development
- **Type Safety**: Generates TypeScript code with proper types
- **ESLint Integration**: Automatically formats generated code

---

### 3. Global Type Augmentations (`vite-env.d.ts`)

Extends global TypeScript definitions to provide enhanced type safety and developer experience across the entire application.

#### Global Augmentation Architecture

```mermaid
graph TB
    subgraph "Global Type Augmentations"
        WINDOW_AUG[Window Interface]
        ARRAY_AUG[Array Interface]
        ROUTER_AUG[React Router Module]
        AXIOS_AUG[Axios Module]
        GLOBAL_VARS[Global Variables]
    end
    
    subgraph "Window Object Extensions"
        T_FUNC[t: i18n.t]
        AXIOS_INST[$axios: AxiosInstance]
    end
    
    subgraph "Type Utilities"
        KEYS_OF_TYPE[KeysOfType<T, TType>]
        ROUTER_ERROR[RouterError]
    end
    
    subgraph "Runtime Dependencies"
        I18N[i18next]
        AXIOS_LIB[axios]
        ROUTER[react-router-dom]
    end
    
    WINDOW_AUG --> T_FUNC
    WINDOW_AUG --> AXIOS_INST
    
    T_FUNC --> I18N
    AXIOS_INST --> AXIOS_LIB
    
    ROUTER_AUG --> ROUTER
    AXIOS_AUG --> AXIOS_LIB
    
    ARRAY_AUG -.polyfill.-> ARRAY_AUG
    
    style WINDOW_AUG fill:#e1f5ff
    style ROUTER_AUG fill:#fff4e1
    style AXIOS_AUG fill:#ffe1e1
```

#### Type Augmentation Details

```mermaid
classDiagram
    class Window {
        <<global interface>>
        +t: typeof i18n.t
        +$axios: AxiosInstance
    }
    
    class Array~T~ {
        <<global interface>>
        +at(index: number): T | undefined
    }
    
    class RouterError {
        <<global interface>>
        +status: number
        +message: string
        +error: Error
    }
    
    class KeysOfType~T, TType~ {
        <<type utility>>
        Extract keys of T where value is TType
    }
    
    class i18n {
        +t: TranslationFunction
    }
    
    class AxiosInstance {
        +get()
        +post()
        +put()
        +delete()
    }
    
    Window --> i18n
    Window --> AxiosInstance
```

#### Global Augmentations Breakdown

##### 1. Window Interface Extensions

```typescript
interface Window {
  t: typeof i18n.t              // Global translation function
  $axios: AxiosInstance         // Global HTTP client
}
```

**Purpose**: Provides global access to commonly used utilities without imports.

**Benefits**:
- Simplified internationalization: `window.t('key')` instead of importing
- Centralized HTTP client: `window.$axios.get()` with interceptors
- Consistent API across all components

##### 2. Global Variable Declarations

```typescript
let t: typeof i18n.t
let $axios: AxiosInstance
```

**Purpose**: Enables direct usage without `window.` prefix.

**Usage**:
```typescript
// Direct usage in any file
const text = t('common.save')
const data = await $axios.get('/api/data')
```

##### 3. Type Utility: KeysOfType

```typescript
type KeysOfType<T, TType> = {
  [K in keyof T]-?: T[K] extends TType ? K : never
}[keyof T]
```

**Purpose**: Extracts keys from an object type where the value matches a specific type.

**Example**:
```typescript
interface User {
  id: number
  name: string
  email: string
  age: number
}

type StringKeys = KeysOfType<User, string>  // "name" | "email"
type NumberKeys = KeysOfType<User, number>  // "id" | "age"
```

##### 4. Router Error Interface

```typescript
interface RouterError {
  status: number
  message: string
  error: Error
}
```

**Purpose**: Standardized error structure for routing errors.

**Usage**: Error boundaries and error handling in route loaders.

##### 5. Array.prototype.at() Polyfill

```typescript
interface Array<T> {
  at(index: number): T | undefined
}
```

**Purpose**: Type definition for ES2022 `Array.at()` method for negative indexing.

##### 6. React Router Module Augmentation

```typescript
declare module "react-router-dom" {
  export function useLoaderData<T>(): T extends (...args: any) => infer R ? Awaited<R> : T
}
```

**Purpose**: Enhanced type inference for `useLoaderData` hook.

**Benefits**:
- Automatic type extraction from loader functions
- Handles async loaders with `Awaited<R>`
- Eliminates manual type annotations

##### 7. Axios Module Augmentation

```typescript
declare module "axios" {
  export function get<T>(url: string, config?: AxiosRequestConfig): Promise<T>
  export function post<T>(url: string, data?: unknown, config?: AxiosRequestConfig): Promise<T>
}
```

**Purpose**: Simplified return types for Axios methods (removes `AxiosResponse` wrapper).

**Benefits**:
- Direct data access: `const data = await axios.get<User[]>('/users')`
- Works with response interceptors that unwrap data
- Cleaner type signatures

---

## Module Dependencies

```mermaid
graph TB
    subgraph "Core Infrastructure"
        ENV[env.d.ts]
        VITE_ENV[vite-env.d.ts]
        ROUTE_GEN[vite-plugin-generate-routes.ts]
    end
    
    subgraph "External Dependencies"
        VITE[vite]
        I18N[i18next]
        AXIOS[axios]
        ROUTER[react-router-dom]
        ESLINT[eslint]
        NODE_FS[node:fs]
        NODE_PATH[node:path]
    end
    
    subgraph "Dependent Modules"
        UI[ui-component-system]
        BC[business-components]
        VIEWS[view-modules]
        HOOKS[state-management-hooks]
        UTILS[utilities-helpers]
    end
    
    ROUTE_GEN --> VITE
    ROUTE_GEN --> ESLINT
    ROUTE_GEN --> NODE_FS
    ROUTE_GEN --> NODE_PATH
    
    VITE_ENV --> I18N
    VITE_ENV --> AXIOS
    VITE_ENV --> ROUTER
    
    ENV --> VITE
    
    ENV -.provides.-> UI
    ENV -.provides.-> BC
    ENV -.provides.-> VIEWS
    ENV -.provides.-> HOOKS
    ENV -.provides.-> UTILS
    
    VITE_ENV -.provides.-> UI
    VITE_ENV -.provides.-> BC
    VITE_ENV -.provides.-> VIEWS
    VITE_ENV -.provides.-> HOOKS
    VITE_ENV -.provides.-> UTILS
    
    ROUTE_GEN -.generates.-> VIEWS
    
    style ENV fill:#e1f5ff
    style VITE_ENV fill:#e1f5ff
    style ROUTE_GEN fill:#fff4e1
```

### Dependency Analysis

#### External Dependencies

| Dependency | Used By | Purpose |
|------------|---------|---------|
| `vite` | Route Generation Plugin | Build tool and plugin system |
| `eslint` | Route Generation Plugin | Code formatting for generated routes |
| `node:fs` | Route Generation Plugin | File system operations |
| `node:path` | Route Generation Plugin | Path manipulation |
| `i18next` | Global Augmentations | Internationalization |
| `axios` | Global Augmentations | HTTP client |
| `react-router-dom` | Global Augmentations | Routing library |

#### Module Relationships

The core-infrastructure module has a **foundational relationship** with all other modules:

- **[ui-component-system](ui-component-system.md)**: Uses global types and environment config
- **[business-components](business-components.md)**: Relies on global utilities and types
- **[view-modules](view-modules.md)**: Generated by route plugin, uses global augmentations
- **[state-management-hooks](state-management-hooks.md)**: Uses global types and utilities
- **[utilities-helpers](utilities-helpers.md)**: Built on core type definitions

---

## Build Process Integration

```mermaid
sequenceDiagram
    participant Dev as Developer
    participant Vite as Vite Build System
    participant Plugin as Route Generation Plugin
    participant FS as File System
    participant App as Application
    
    Dev->>Vite: npm run dev
    Vite->>Plugin: configureServer()
    Plugin->>FS: watchDir(src/view)
    Plugin->>Plugin: buildStart()
    Plugin->>Plugin: generateRoutes()
    Plugin->>FS: Write generatedRoutes.tsx
    
    Vite->>Vite: Load env variables
    Vite->>Vite: Apply type definitions
    Vite->>App: Start dev server
    
    loop Development
        Dev->>FS: Create/modify route file
        FS->>Plugin: File change event
        Plugin->>Plugin: generateRoutes()
        Plugin->>FS: Update generatedRoutes.tsx
        FS->>Vite: File change detected
        Vite->>App: Hot reload
    end
    
    Dev->>Vite: npm run build
    Vite->>Plugin: buildStart()
    Plugin->>Plugin: generateRoutes()
    Vite->>Vite: Bundle application
    Vite->>FS: Output dist/
```

### Build Configuration Flow

```mermaid
flowchart LR
    subgraph "Configuration Files"
        ENV_FILE[.env files]
        VITE_CONFIG[vite.config.ts]
        TS_CONFIG[tsconfig.json]
    end
    
    subgraph "Type Definitions"
        ENV_D[env.d.ts]
        VITE_ENV_D[vite-env.d.ts]
    end
    
    subgraph "Build Pipeline"
        VITE[Vite]
        PLUGIN[Route Plugin]
        TS[TypeScript Compiler]
    end
    
    subgraph "Output"
        ROUTES[generatedRoutes.tsx]
        BUNDLE[dist/]
    end
    
    ENV_FILE --> VITE
    VITE_CONFIG --> VITE
    TS_CONFIG --> TS
    
    ENV_D --> TS
    VITE_ENV_D --> TS
    
    VITE --> PLUGIN
    PLUGIN --> ROUTES
    
    VITE --> TS
    TS --> BUNDLE
    ROUTES --> BUNDLE
    
    style PLUGIN fill:#fff4e1
    style ROUTES fill:#e1ffe1
```

---

## Runtime Initialization

```mermaid
sequenceDiagram
    participant Browser
    participant Main as main.tsx
    participant Window as Window Object
    participant I18n as i18next
    participant Axios as Axios Instance
    participant Router as React Router
    participant App as Application
    
    Browser->>Main: Load application
    Main->>I18n: Initialize i18n
    I18n-->>Window: window.t = i18n.t
    I18n-->>Window: global t = i18n.t
    
    Main->>Axios: Create instance
    Axios->>Axios: Setup interceptors
    Axios-->>Window: window.$axios = instance
    Axios-->>Window: global $axios = instance
    
    Main->>Router: Load routes
    Router->>Router: Import generatedRoutes
    Router->>Router: Create router instance
    
    Main->>App: Render application
    App->>App: Access global utilities
    
    Note over Window,App: All components can now use<br/>t() and $axios directly
```

### Global Initialization Pattern

```typescript
// Typical initialization in main.tsx
import i18n from 'i18next'
import axios from 'axios'
import routes from './generatedRoutes'

// Setup i18n
await i18n.init(/* config */)
window.t = i18n.t
globalThis.t = i18n.t

// Setup axios
const axiosInstance = axios.create({
  baseURL: import.meta.env.VITE_DOMAIN
})
// Add interceptors...
window.$axios = axiosInstance
globalThis.$axios = axiosInstance

// Setup router
const router = createBrowserRouter(routes)

// Render app
ReactDOM.createRoot(document.getElementById('root')!).render(
  <RouterProvider router={router} />
)
```

---

## Usage Patterns

### 1. Environment Variables

```typescript
// Accessing environment variables
const apiUrl = `${import.meta.env.VITE_DOMAIN}/api`
const authConfig = {
  tenantId: import.meta.env.VITE_TENANT_ID,
  clientId: import.meta.env.VITE_CLIENT_ID
}

// Type-safe access with autocomplete
// TypeScript will error if variable doesn't exist
```

### 2. Convention-Based Routing

```typescript
// File structure determines routes
// src/view/dashboard/index.tsx → /view/dashboard
// src/view/product/index.[id].tsx → /view/product/:id
// src/view/home/index.default.tsx → /view/home (index route)

// Route component with loader
export default function ProductDetail() {
  const product = useLoaderData<typeof loader>()
  return <div>{product.name}</div>
}

export async function loader({ params }: LoaderFunctionArgs) {
  return await $axios.get(`/api/products/${params.id}`)
}

// Optional: Error boundary
export function ErrorBoundary() {
  const error = useRouteError() as RouterError
  return <div>Error {error.status}: {error.message}</div>
}

// Optional: Hydration fallback
export function HydrateFallback() {
  return <Skeleton />
}
```

### 3. Global Utilities

```typescript
// Using global translation function
function MyComponent() {
  return (
    <button>{t('common.save')}</button>
  )
}

// Using global axios instance
async function fetchData() {
  const users = await $axios.get<User[]>('/api/users')
  return users  // Direct data access, no .data property
}

// Type utility usage
interface FormData {
  name: string
  email: string
  age: number
  active: boolean
}

type StringFields = KeysOfType<FormData, string>  // "name" | "email"
```

### 4. Enhanced Type Inference

```typescript
// React Router loader with automatic type inference
export async function loader() {
  return {
    users: await $axios.get<User[]>('/api/users'),
    settings: await $axios.get<Settings>('/api/settings')
  }
}

function Component() {
  // Type is automatically inferred from loader return type
  const { users, settings } = useLoaderData<typeof loader>()
  // users: User[]
  // settings: Settings
}

// Axios with simplified types
const response = await axios.get<Product[]>('/api/products')
// response is Product[], not AxiosResponse<Product[]>
```

---

## Best Practices

### 1. Environment Configuration

```typescript
// ✅ DO: Use type-safe environment variables
const domain = import.meta.env.VITE_DOMAIN

// ❌ DON'T: Use process.env (not available in Vite)
const domain = process.env.VITE_DOMAIN

// ✅ DO: Prefix with VITE_ for client-side variables
// VITE_API_URL=https://api.example.com

// ❌ DON'T: Use non-prefixed variables (won't be exposed)
// API_URL=https://api.example.com
```

### 2. Route Organization

```typescript
// ✅ DO: Follow naming conventions
// view/products/index.tsx              → /view/products
// view/products/index.[id].tsx         → /view/products/:id
// view/products/index.[id].default.tsx → /view/products/:id (index)

// ❌ DON'T: Use non-standard naming
// view/products/product.tsx            → Won't be discovered
// view/products/[id].tsx               → Won't be discovered

// ✅ DO: Use optional parameters for flexible routes
// view/search/index.(query).tsx        → /view/search/:query?

// ✅ DO: Export loader, ErrorBoundary, HydrateFallback as needed
export default function Component() { /* ... */ }
export async function loader() { /* ... */ }
export function ErrorBoundary() { /* ... */ }
export function HydrateFallback() { /* ... */ }
```

### 3. Global Utilities

```typescript
// ✅ DO: Use global utilities for common operations
const text = t('common.save')
const data = await $axios.get('/api/data')

// ✅ DO: Import when you need the full API
import i18n from 'i18next'
i18n.changeLanguage('en')

// ❌ DON'T: Overwrite global utilities
window.t = myCustomFunction  // Bad practice

// ✅ DO: Use type utilities for type-safe operations
type StringKeys = KeysOfType<MyInterface, string>
```

### 4. Type Augmentations

```typescript
// ✅ DO: Leverage enhanced type inference
const data = useLoaderData<typeof loader>()

// ❌ DON'T: Manually type when inference works
const data = useLoaderData() as LoaderData

// ✅ DO: Use simplified Axios types
const users = await axios.get<User[]>('/api/users')
// users is User[], not AxiosResponse<User[]>

// ✅ DO: Use RouterError for error handling
function ErrorBoundary() {
  const error = useRouteError() as RouterError
  return <div>Error {error.status}</div>
}
```

---

## Performance Considerations

### 1. Route Code Splitting

```mermaid
graph LR
    subgraph "Initial Bundle"
        MAIN[main.tsx]
        ROUTER[Router Setup]
        ROUTES[Route Definitions]
    end
    
    subgraph "Lazy Loaded Chunks"
        ROUTE1[Route 1 Component]
        ROUTE2[Route 2 Component]
        ROUTE3[Route 3 Component]
    end
    
    MAIN --> ROUTER
    ROUTER --> ROUTES
    ROUTES -.lazy load.-> ROUTE1
    ROUTES -.lazy load.-> ROUTE2
    ROUTES -.lazy load.-> ROUTE3
    
    style ROUTES fill:#fff4e1
    style ROUTE1 fill:#e1ffe1
    style ROUTE2 fill:#e1ffe1
    style ROUTE3 fill:#e1ffe1
```

**Benefits**:
- Smaller initial bundle size
- Faster time to interactive
- On-demand loading of route components
- Automatic code splitting by Vite

### 2. Build Optimization

```typescript
// Route generation only runs when files change
// Incremental updates in development
// Single generation in production build

// ESLint formatting is cached
// Only reformats if content changes
```

### 3. Type Checking Performance

```typescript
// Global augmentations are loaded once
// Type inference is compile-time only
// No runtime overhead for type utilities
```

---

## Error Handling

### 1. Route Generation Errors

```mermaid
flowchart TD
    START[Route Generation] --> CHECK{File Valid?}
    CHECK -->|Yes| PARSE[Parse Route]
    CHECK -->|No| SKIP[Skip File]
    
    PARSE --> BUILD{Build Success?}
    BUILD -->|Yes| FORMAT[Format Code]
    BUILD -->|No| ERROR1[Log Error]
    
    FORMAT --> WRITE{Write Success?}
    WRITE -->|Yes| DONE[Complete]
    WRITE -->|No| ERROR2[Log Error]
    
    ERROR1 --> CONTINUE[Continue with Other Routes]
    ERROR2 --> CONTINUE
    SKIP --> CONTINUE
    
    style ERROR1 fill:#ffe1e1
    style ERROR2 fill:#ffe1e1
```

### 2. Runtime Error Boundaries

```typescript
// Per-route error boundaries
export function ErrorBoundary() {
  const error = useRouteError() as RouterError
  
  if (error.status === 404) {
    return <NotFound />
  }
  
  if (error.status === 403) {
    return <Forbidden />
  }
  
  return <GenericError error={error} />
}
```

### 3. Environment Variable Validation

```typescript
// Type system ensures variables exist at compile time
// Runtime validation can be added in main.tsx

const requiredEnvVars = [
  'VITE_DOMAIN',
  'VITE_TENANT_ID',
  'VITE_CLIENT_ID'
] as const

requiredEnvVars.forEach(varName => {
  if (!import.meta.env[varName]) {
    throw new Error(`Missing required environment variable: ${varName}`)
  }
})
```

---

## Testing Considerations

### 1. Route Generation Testing

```typescript
// Test route discovery
describe('Route Generation', () => {
  it('should discover all index files', () => {
    const routes = conventionRouter('src/view')
    expect(routes).toHaveLength(expectedCount)
  })
  
  it('should handle parameterized routes', () => {
    const routes = conventionRouter('src/view')
    const productRoute = routes.find(r => r.id === 'view/product')
    expect(productRoute?.path).toBe('view/product/:id')
  })
})
```

### 2. Global Utilities Testing

```typescript
// Mock global utilities in tests
beforeEach(() => {
  global.t = jest.fn((key) => key)
  global.$axios = {
    get: jest.fn(),
    post: jest.fn()
  } as any
})
```

### 3. Type Testing

```typescript
// Use type assertions to test type utilities
type Test1 = KeysOfType<{ a: string, b: number }, string>
const _test1: Test1 = 'a'  // Should compile

type Test2 = KeysOfType<{ a: string, b: number }, number>
const _test2: Test2 = 'b'  // Should compile
```

---

## Migration Guide

### From Manual Routes to Convention-Based

```typescript
// Before: Manual route configuration
const routes = [
  {
    path: '/dashboard',
    element: <Dashboard />,
    loader: dashboardLoader
  },
  {
    path: '/product/:id',
    element: <ProductDetail />,
    loader: productLoader
  }
]

// After: Convention-based routing
// 1. Create view/dashboard/index.tsx
export default function Dashboard() { /* ... */ }
export async function loader() { /* ... */ }

// 2. Create view/product/index.[id].tsx
export default function ProductDetail() { /* ... */ }
export async function loader({ params }) { /* ... */ }

// 3. Routes are automatically generated
import routes from './generatedRoutes'
const router = createBrowserRouter(routes)
```

### Adding Global Utilities

```typescript
// 1. Extend Window interface in vite-env.d.ts
interface Window {
  t: typeof i18n.t
  $axios: AxiosInstance
  myUtility: MyUtilityType  // Add new utility
}

// 2. Initialize in main.tsx
window.myUtility = createMyUtility()
globalThis.myUtility = window.myUtility

// 3. Declare global variable
declare let myUtility: MyUtilityType

// 4. Use anywhere without imports
myUtility.doSomething()
```

---

## Related Modules

- **[ui-component-system](ui-component-system.md)**: Built on core infrastructure, uses global types
- **[business-components](business-components.md)**: Leverages global utilities and routing
- **[view-modules](view-modules.md)**: Generated by route plugin, primary consumer of routing system
- **[state-management-hooks](state-management-hooks.md)**: Uses global types and utilities
- **[utilities-helpers](utilities-helpers.md)**: Extends core infrastructure with helper functions

---

## Summary

The **core-infrastructure** module provides the essential foundation for the Trend Engine application:

### Key Capabilities

1. **Type-Safe Configuration**: Environment variables with compile-time validation
2. **Automatic Routing**: Convention-based route generation with lazy loading
3. **Global Utilities**: Centralized access to i18n and HTTP client
4. **Enhanced Types**: Improved type inference for React Router and Axios
5. **Developer Experience**: Hot reload, automatic formatting, and type safety

### Architecture Principles

- **Convention over Configuration**: File structure determines routes
- **Type Safety First**: Strong typing at every level
- **Performance Optimized**: Code splitting and lazy loading by default
- **Developer Friendly**: Minimal boilerplate, maximum productivity

### Integration Points

- Vite build system for bundling and development
- React Router for routing and navigation
- i18next for internationalization
- Axios for HTTP communication
- ESLint for code quality

This module serves as the bedrock for all other modules, providing the infrastructure that enables rapid development while maintaining type safety and performance.
