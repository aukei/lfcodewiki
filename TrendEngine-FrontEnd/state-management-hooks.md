# State Management Hooks Module

## Overview

The **state-management-hooks** module provides a collection of custom React hooks and utilities for managing application state across different persistence layers. This module focuses on three key areas: URL-based state management, server-side record persistence, and component state preservation during navigation. These hooks enable seamless state synchronization between the browser URL, backend services, and React component lifecycle, ensuring a consistent user experience across page navigations and sessions.

## Module Purpose

This module serves as the state management layer that bridges:
- **URL State**: Synchronizing application state with browser URL query parameters for shareable and bookmarkable application states
- **Server-Side Persistence**: Saving and restoring user preferences and filter states across sessions
- **Component State Preservation**: Maintaining component state during route transitions using a KeepAlive mechanism

The module is essential for applications requiring complex state management scenarios where state needs to persist across different contexts (URL, server, component lifecycle).

---

## Architecture Overview

```mermaid
graph TB
    subgraph "State Management Hooks Module"
        subgraph "URL State Management"
            USH[useUrlState Hook]
            QS[Query String Parser/Serializer]
            RR[React Router Integration]
        end
        
        subgraph "Record Flow Management"
            RFH[useRecordFlow Hook]
            API[Service API Layer]
            CACHE[Local State Cache]
        end
        
        subgraph "Component State Preservation"
            KA[KeepAlive Component]
            WRAP[Wrapper Component]
            SUSP[Suspense Boundary]
        end
    end
    
    subgraph "External Dependencies"
        ROUTER[React Router v6]
        AHOOKS[ahooks Library]
        SERVICE[Backend Services]
        TOAST[Toast UI Component]
    end
    
    subgraph "Application Layer"
        VIEWS[View Components]
        FILTERS[Filter Components]
        FORMS[Form Components]
    end
    
    USH --> QS
    USH --> RR
    RR --> ROUTER
    USH --> AHOOKS
    
    RFH --> API
    RFH --> CACHE
    RFH --> AHOOKS
    API --> SERVICE
    RFH --> TOAST
    
    KA --> WRAP
    KA --> SUSP
    
    VIEWS --> USH
    VIEWS --> RFH
    VIEWS --> KA
    FILTERS --> USH
    FILTERS --> RFH
    FORMS --> USH
    
    style USH fill:#4A90E2
    style RFH fill:#4A90E2
    style KA fill:#4A90E2
```

---

## Core Components

### 1. useUrlState Hook

**Purpose**: Manages application state through URL query parameters, enabling shareable and bookmarkable application states.

**Key Features**:
- Bidirectional synchronization between React state and URL
- Support for complex data structures (nested objects, arrays)
- Configurable navigation modes (push/replace)
- Deep parsing and serialization with `qs` library
- Automatic state restoration from URL on mount

**Type Definitions**:

```typescript
interface Options {
  navigateMode?: "push" | "replace"      // URL update strategy
  parseOptions?: qs.IParseOptions        // Query string parsing config
  stringifyOptions?: qs.IStringifyOptions // Query string serialization config
}
```

**Configuration**:
- **Parse Options**: 
  - `parseArrays: true` - Parse array notation in URLs
  - `arrayLimit: 999` - Support large arrays
  - `allowDots: true` - Support dot notation for nested objects
  - `strictNullHandling: true` - Distinguish between null and undefined
  - `depth: 10` - Support deeply nested structures

- **Stringify Options**:
  - `allowDots: true` - Use dot notation for nested objects
  - `strictNullHandling: true` - Preserve null values
  - `arrayFormat: "indices"` - Use indexed array format

**Usage Flow**:

```mermaid
sequenceDiagram
    participant Component
    participant useUrlState
    participant QueryParser
    participant ReactRouter
    participant Browser
    
    Component->>useUrlState: Initialize with initialState
    useUrlState->>Browser: Read location.search
    Browser-->>useUrlState: Current URL params
    useUrlState->>QueryParser: Parse query string
    QueryParser-->>useUrlState: Parsed object
    useUrlState->>useUrlState: Merge with initialState
    useUrlState-->>Component: Return [state, setState]
    
    Component->>useUrlState: setState(newState)
    useUrlState->>QueryParser: Stringify new state
    QueryParser-->>useUrlState: Query string
    useUrlState->>ReactRouter: navigate({ search })
    ReactRouter->>Browser: Update URL
    Browser-->>Component: Re-render with new state
```

---

### 2. useRecordFlow Hook

**Purpose**: Manages server-side persistence of user selections and filter states, enabling state restoration across sessions.

**Key Features**:
- Automatic loading of saved records on mount
- Asynchronous save operations with user feedback
- Type-safe record management with generics
- Integration with backend service layer
- Toast notifications for save operations

**Type Definitions**:

```typescript
enum RECORD_ID {
  RUNWAY = "LFRunwayParamsRecord",
  // Extensible for other record types
}

interface Options {
  exclude?: string[]  // Fields to exclude from persistence
  extend?: string[]   // Additional fields to include
}

interface RecordFlowProps {
  id: RECORD_ID       // Unique identifier for record type
  options?: Options   // Configuration options
}
```

**Data Flow**:

```mermaid
flowchart LR
    subgraph "Component Lifecycle"
        MOUNT[Component Mount]
        INTERACT[User Interaction]
        SAVE[Save Trigger]
    end
    
    subgraph "useRecordFlow Hook"
        LOAD[Load Record]
        STATE[Local State]
        PERSIST[Persist Record]
    end
    
    subgraph "Backend Services"
        READ[serviceSelectRecordList]
        WRITE[serviceSelectRecordWrite]
        DB[(Database)]
    end
    
    subgraph "UI Feedback"
        LOADING[Loading State]
        TOAST[Toast Notification]
    end
    
    MOUNT --> LOAD
    LOAD --> READ
    READ --> DB
    DB --> STATE
    STATE --> LOADING
    
    INTERACT --> STATE
    SAVE --> PERSIST
    PERSIST --> WRITE
    WRITE --> DB
    DB --> TOAST
    
    style STATE fill:#4A90E2
    style DB fill:#E8A87C
```

**Record Structure**:

```mermaid
classDiagram
    class UserSelectRecordVo {
        +string pageCode
        +string selectCondition
        +string selectConditionView
        +Date createdAt
        +Date updatedAt
    }
    
    class RecordFlow {
        +RECORD_ID id
        +T record
        +boolean loading
        +saveRecord(params: T)
    }
    
    class ServiceLayer {
        +serviceSelectRecordList()
        +serviceSelectRecordWrite()
    }
    
    RecordFlow --> UserSelectRecordVo : manages
    RecordFlow --> ServiceLayer : uses
```

---

### 3. KeepAlive Component

**Purpose**: Preserves component state during route transitions by preventing component unmounting.

**Key Features**:
- Maintains component state across navigation
- Uses React Suspense for conditional rendering
- Automatic scroll restoration on activation
- Zero-configuration state preservation
- Minimal performance overhead

**Type Definitions**:

```typescript
interface KeepAliveProps {
  children: ReactNode  // Component to preserve
  active: boolean      // Visibility control
}

interface WrapperProps {
  children: ReactNode
  active: boolean
}
```

**Implementation Strategy**:

```mermaid
stateDiagram-v2
    [*] --> Mounted: Component Renders
    
    Mounted --> Active: active=true
    Mounted --> Inactive: active=false
    
    Active --> Visible: Display Block
    Inactive --> Hidden: Display None
    
    Visible --> Active: User Navigates Away
    Hidden --> Inactive: State Preserved
    
    Active --> Inactive: active=false
    Inactive --> Active: active=true
    
    Visible --> [*]: Component Unmounts
    Hidden --> [*]: Component Unmounts
    
    note right of Hidden
        Component state preserved
        Promise suspends rendering
    end note
    
    note right of Visible
        Component visible
        Scroll position reset
    end note
```

**Rendering Mechanism**:

```mermaid
flowchart TB
    subgraph "KeepAlive Component"
        KA_RENDER[KeepAlive Render]
        SUSPENSE1[Suspense Boundary 1]
        SUSPENSE2[Suspense Boundary 2]
        WRAPPER1[Wrapper active=true]
        WRAPPER2[Wrapper active=false]
    end
    
    subgraph "Wrapper Logic"
        CHECK{Is Active?}
        RESOLVE[Resolve Promise]
        SUSPEND[Throw Promise]
        RENDER[Render Children]
    end
    
    subgraph "Visual Output"
        VISIBLE[Visible Component]
        HIDDEN[Hidden Component]
    end
    
    KA_RENDER --> SUSPENSE1
    KA_RENDER --> SUSPENSE2
    
    SUSPENSE1 --> WRAPPER1
    SUSPENSE2 --> WRAPPER2
    
    WRAPPER1 --> CHECK
    WRAPPER2 --> CHECK
    
    CHECK -->|Yes| RESOLVE
    CHECK -->|No| SUSPEND
    
    RESOLVE --> RENDER
    SUSPEND -.->|Suspends| HIDDEN
    RENDER --> VISIBLE
    
    style RESOLVE fill:#90EE90
    style SUSPEND fill:#FFB6C1
```

---

## Integration Patterns

### Pattern 1: URL State + Record Flow Combination

This pattern combines URL state for immediate UI updates with server-side persistence for long-term storage.

```mermaid
sequenceDiagram
    participant User
    participant Component
    participant useUrlState
    participant useRecordFlow
    participant Backend
    
    User->>Component: Load Page
    Component->>useUrlState: Initialize from URL
    Component->>useRecordFlow: Load saved record
    useRecordFlow->>Backend: Fetch record
    Backend-->>useRecordFlow: Return saved state
    useRecordFlow-->>Component: Merge with URL state
    
    User->>Component: Change filter
    Component->>useUrlState: Update URL
    useUrlState-->>Component: State updated
    
    User->>Component: Click "Save"
    Component->>useRecordFlow: saveRecord(state)
    useRecordFlow->>Backend: Persist state
    Backend-->>useRecordFlow: Success
    useRecordFlow-->>Component: Show toast
```

**Use Case**: Filter pages where users want to:
- Share current filters via URL
- Save favorite filter combinations for later use
- Restore last used filters on page load

**Example Implementation**:
```typescript
// In a filter component
const [urlState, setUrlState] = useUrlState<FilterState>(defaultFilters)
const { record, saveRecord } = useRecordFlow<FilterState>({ 
  id: RECORD_ID.RUNWAY 
})

// On mount: merge URL state with saved record
useEffect(() => {
  setUrlState({ ...record, ...urlState })
}, [record])

// On filter change: update URL immediately
const handleFilterChange = (newFilters) => {
  setUrlState(newFilters)
}

// On save: persist to backend
const handleSave = () => {
  saveRecord(urlState)
}
```

---

### Pattern 2: KeepAlive with Tab Navigation

Preserves state across tab switches without losing user input or scroll position.

```mermaid
flowchart TB
    subgraph "Tab Navigation"
        TAB1[Tab 1: Analysis]
        TAB2[Tab 2: Details]
        TAB3[Tab 3: Settings]
    end
    
    subgraph "KeepAlive Wrappers"
        KA1[KeepAlive active=tab===1]
        KA2[KeepAlive active=tab===2]
        KA3[KeepAlive active=tab===3]
    end
    
    subgraph "Component States"
        STATE1[Analysis State Preserved]
        STATE2[Details State Preserved]
        STATE3[Settings State Preserved]
    end
    
    TAB1 -.->|Controls| KA1
    TAB2 -.->|Controls| KA2
    TAB3 -.->|Controls| KA3
    
    KA1 --> STATE1
    KA2 --> STATE2
    KA3 --> STATE3
    
    style KA1 fill:#90EE90
    style STATE1 fill:#E8F5E9
```

**Use Case**: Multi-tab interfaces where:
- Each tab has complex forms or filters
- Users frequently switch between tabs
- Losing state would frustrate users

---

### Pattern 3: Hierarchical State Management

Combines all three hooks for complex, multi-level state management.

```mermaid
graph TB
    subgraph "Application State Layers"
        URL[URL State Layer]
        SERVER[Server Persistence Layer]
        COMPONENT[Component State Layer]
    end
    
    subgraph "State Hooks"
        USH[useUrlState]
        RFH[useRecordFlow]
        KA[KeepAlive]
    end
    
    subgraph "State Characteristics"
        SHARE[Shareable]
        PERSIST[Persistent]
        TEMP[Temporary]
    end
    
    URL --> USH
    SERVER --> RFH
    COMPONENT --> KA
    
    USH --> SHARE
    RFH --> PERSIST
    KA --> TEMP
    
    SHARE -.->|Bookmarkable| URL
    PERSIST -.->|Cross-session| SERVER
    TEMP -.->|Navigation| COMPONENT
    
    style URL fill:#4A90E2
    style SERVER fill:#E8A87C
    style COMPONENT fill:#90EE90
```

---

## Dependencies

### External Dependencies

```mermaid
graph LR
    subgraph "State Management Hooks"
        MODULE[state-management-hooks]
    end
    
    subgraph "React Ecosystem"
        REACT[React 18+]
        ROUTER[React Router v6]
        AHOOKS[ahooks]
    end
    
    subgraph "Utilities"
        QS[qs Library]
    end
    
    subgraph "Backend Integration"
        SERVICE[Service Layer]
    end
    
    subgraph "UI Components"
        TOAST[Toast Component]
    end
    
    MODULE --> REACT
    MODULE --> ROUTER
    MODULE --> AHOOKS
    MODULE --> QS
    MODULE --> SERVICE
    MODULE --> TOAST
    
    style MODULE fill:#4A90E2
```

**Key Dependencies**:

1. **ahooks** - Provides foundational hooks:
   - `useRequest`: Async data fetching with loading states
   - `useMemoizedFn`: Memoized callback functions
   - `useUpdate`: Force component updates

2. **React Router v6** - Navigation and location management:
   - `useLocation`: Access current URL
   - `useNavigate`: Programmatic navigation

3. **qs** - Query string parsing/serialization:
   - Advanced parsing with nested object support
   - Configurable array and null handling

4. **Service Layer** - Backend API integration:
   - `serviceSelectRecordList`: Fetch saved records
   - `serviceSelectRecordWrite`: Persist records

5. **Toast Component** - User feedback:
   - Success/error notifications for save operations

---

## Module Dependencies

This module is consumed by:

### View Modules
- **Consumer Analysis**: Uses `useUrlState` for trend discovery filters
- **Search Results**: Combines `useUrlState` with `KeepAlive` for search state
- **Runway Analysis**: Uses `useRecordFlow` for saving analysis parameters
- **Market Analysis**: Persists filter states across sessions
- **Libraries**: URL state management for Pinterest filters

See [view-modules.md](view-modules.md) for detailed view component documentation.

### Business Components
- **Filter Components**: Category and time filters use `useUrlState`
- **Batch Operations**: State preservation during batch processing
- **Chart Components**: Maintains chart configurations

See [business-components.md](business-components.md) for business component details.

---

## Data Flow Architecture

### Complete State Lifecycle

```mermaid
flowchart TB
    subgraph "User Actions"
        INIT[Page Load]
        CHANGE[State Change]
        SAVE[Save Action]
        NAV[Navigation]
    end
    
    subgraph "URL State Flow"
        URL_READ[Read URL Params]
        URL_PARSE[Parse Query String]
        URL_UPDATE[Update URL]
        URL_SYNC[Sync to Browser]
    end
    
    subgraph "Record Flow"
        REC_FETCH[Fetch Record]
        REC_CACHE[Cache Locally]
        REC_SAVE[Save to Server]
        REC_NOTIFY[Notify User]
    end
    
    subgraph "Component State"
        COMP_MOUNT[Mount Component]
        COMP_STATE[Update State]
        COMP_PRESERVE[Preserve State]
        COMP_RESTORE[Restore State]
    end
    
    subgraph "Persistence Layers"
        BROWSER[Browser URL]
        SERVER[Backend Database]
        MEMORY[Component Memory]
    end
    
    INIT --> URL_READ
    INIT --> REC_FETCH
    INIT --> COMP_MOUNT
    
    URL_READ --> URL_PARSE
    URL_PARSE --> COMP_STATE
    
    REC_FETCH --> REC_CACHE
    REC_CACHE --> COMP_STATE
    
    CHANGE --> URL_UPDATE
    URL_UPDATE --> URL_SYNC
    URL_SYNC --> BROWSER
    
    SAVE --> REC_SAVE
    REC_SAVE --> SERVER
    REC_SAVE --> REC_NOTIFY
    
    NAV --> COMP_PRESERVE
    COMP_PRESERVE --> MEMORY
    MEMORY --> COMP_RESTORE
    
    style BROWSER fill:#4A90E2
    style SERVER fill:#E8A87C
    style MEMORY fill:#90EE90
```

---

## Advanced Usage Patterns

### 1. Conditional State Persistence

```mermaid
flowchart TD
    START[User Changes State]
    CHECK_URL{Should Update URL?}
    CHECK_SERVER{Should Save to Server?}
    CHECK_LOCAL{Should Preserve Locally?}
    
    URL_UPDATE[Update URL State]
    SERVER_SAVE[Save to Server]
    LOCAL_PRESERVE[Preserve in Memory]
    
    DONE[State Updated]
    
    START --> CHECK_URL
    CHECK_URL -->|Yes| URL_UPDATE
    CHECK_URL -->|No| CHECK_SERVER
    
    URL_UPDATE --> CHECK_SERVER
    CHECK_SERVER -->|Yes| SERVER_SAVE
    CHECK_SERVER -->|No| CHECK_LOCAL
    
    SERVER_SAVE --> CHECK_LOCAL
    CHECK_LOCAL -->|Yes| LOCAL_PRESERVE
    CHECK_LOCAL -->|No| DONE
    
    LOCAL_PRESERVE --> DONE
```

**Decision Criteria**:
- **URL State**: For shareable, bookmarkable state (filters, pagination)
- **Server Persistence**: For user preferences, saved configurations
- **Local Preservation**: For temporary UI state during navigation

---

### 2. State Synchronization Strategy

```mermaid
sequenceDiagram
    participant URL as URL State
    participant Server as Server Record
    participant Local as Local State
    participant UI as UI Component
    
    Note over URL,UI: Initial Load
    URL->>Local: Parse URL params
    Server->>Local: Fetch saved record
    Local->>Local: Merge states (URL takes precedence)
    Local->>UI: Render with merged state
    
    Note over URL,UI: User Interaction
    UI->>Local: Update state
    Local->>URL: Sync to URL (immediate)
    
    Note over URL,UI: User Saves
    UI->>Local: Trigger save
    Local->>Server: Persist state
    Server-->>UI: Confirm save
    
    Note over URL,UI: Navigation Away
    UI->>Local: Preserve state
    
    Note over URL,UI: Navigation Back
    Local->>UI: Restore preserved state
```

---

## Performance Considerations

### Optimization Strategies

```mermaid
mindmap
    root((Performance))
        URL State
            Debounce Updates
            Batch Changes
            Memoize Callbacks
            Shallow Comparison
        Record Flow
            Request Deduplication
            Optimistic Updates
            Cache Management
            Error Retry Logic
        KeepAlive
            Lazy Loading
            Memory Cleanup
            Conditional Rendering
            Scroll Optimization
```

**Best Practices**:

1. **URL State Optimization**:
   - Debounce rapid state changes to prevent excessive URL updates
   - Use `useMemoizedFn` to prevent unnecessary re-renders
   - Implement shallow comparison for complex objects

2. **Record Flow Optimization**:
   - Leverage `ahooks` request deduplication
   - Implement optimistic UI updates for better UX
   - Cache records locally to reduce server requests

3. **KeepAlive Optimization**:
   - Limit number of preserved components
   - Implement cleanup for unmounted components
   - Use lazy loading for heavy components

---

## Error Handling

### Error Flow Diagram

```mermaid
flowchart TB
    subgraph "Error Sources"
        URL_ERR[URL Parse Error]
        NET_ERR[Network Error]
        SER_ERR[Serialization Error]
    end
    
    subgraph "Error Handlers"
        URL_HANDLE[Fallback to Default]
        NET_HANDLE[Retry Logic]
        SER_HANDLE[Validation & Sanitization]
    end
    
    subgraph "User Feedback"
        TOAST_ERR[Error Toast]
        CONSOLE[Console Warning]
        FALLBACK[Fallback UI]
    end
    
    subgraph "Recovery Actions"
        RETRY[Retry Operation]
        RESET[Reset to Default]
        CACHE[Use Cached Data]
    end
    
    URL_ERR --> URL_HANDLE
    NET_ERR --> NET_HANDLE
    SER_ERR --> SER_HANDLE
    
    URL_HANDLE --> RESET
    NET_HANDLE --> RETRY
    SER_HANDLE --> FALLBACK
    
    RESET --> CONSOLE
    RETRY --> TOAST_ERR
    FALLBACK --> TOAST_ERR
    
    RETRY -.->|Success| CACHE
    RETRY -.->|Failure| RESET
    
    style URL_ERR fill:#FFB6C1
    style NET_ERR fill:#FFB6C1
    style SER_ERR fill:#FFB6C1
```

**Error Handling Strategies**:

1. **URL State Errors**:
   - Malformed query strings → Fallback to initial state
   - Invalid data types → Type coercion or default values
   - Missing parameters → Merge with defaults

2. **Record Flow Errors**:
   - Network failures → Toast notification + retry option
   - Server errors → Graceful degradation to local state
   - Parse errors → Log warning + use empty state

3. **KeepAlive Errors**:
   - Suspense errors → Fallback to normal rendering
   - Memory issues → Automatic cleanup of old states

---

## Testing Strategies

### Test Coverage Map

```mermaid
graph TB
    subgraph "Unit Tests"
        URL_UNIT[URL State Logic]
        REC_UNIT[Record Flow Logic]
        KA_UNIT[KeepAlive Logic]
    end
    
    subgraph "Integration Tests"
        URL_INT[URL + Router Integration]
        REC_INT[Record + Service Integration]
        KA_INT[KeepAlive + Navigation]
    end
    
    subgraph "E2E Tests"
        FLOW_E2E[Complete State Flow]
        NAV_E2E[Navigation Scenarios]
        PERSIST_E2E[Persistence Scenarios]
    end
    
    URL_UNIT --> URL_INT
    REC_UNIT --> REC_INT
    KA_UNIT --> KA_INT
    
    URL_INT --> FLOW_E2E
    REC_INT --> FLOW_E2E
    KA_INT --> NAV_E2E
    
    FLOW_E2E --> PERSIST_E2E
    NAV_E2E --> PERSIST_E2E
    
    style FLOW_E2E fill:#90EE90
    style NAV_E2E fill:#90EE90
    style PERSIST_E2E fill:#90EE90
```

**Testing Recommendations**:

1. **useUrlState Tests**:
   - URL parsing with various data types
   - State updates and URL synchronization
   - Navigation mode behavior (push vs replace)
   - Edge cases: empty state, null values, nested arrays

2. **useRecordFlow Tests**:
   - Record loading on mount
   - Save operation success/failure
   - Toast notification triggers
   - Loading state management

3. **KeepAlive Tests**:
   - State preservation across active/inactive transitions
   - Scroll position restoration
   - Memory cleanup on unmount
   - Multiple KeepAlive instances

---

## Migration Guide

### From Local State to URL State

```mermaid
flowchart LR
    subgraph "Before"
        LOCAL[useState]
        LOCAL_STATE[Local State Only]
    end
    
    subgraph "After"
        URL[useUrlState]
        URL_STATE[URL Synchronized State]
        SHARE[Shareable Links]
    end
    
    subgraph "Migration Steps"
        STEP1[1. Replace useState]
        STEP2[2. Define initial state]
        STEP3[3. Configure options]
        STEP4[4. Test URL sync]
    end
    
    LOCAL --> STEP1
    STEP1 --> STEP2
    STEP2 --> STEP3
    STEP3 --> STEP4
    STEP4 --> URL
    
    LOCAL_STATE -.->|Upgrade| URL_STATE
    URL_STATE --> SHARE
    
    style URL fill:#90EE90
    style SHARE fill:#90EE90
```

**Migration Checklist**:
- [ ] Identify state that should be shareable
- [ ] Define TypeScript interfaces for state shape
- [ ] Configure parse/stringify options for complex types
- [ ] Update state setters to use new hook
- [ ] Test URL updates and parsing
- [ ] Verify backward compatibility with old URLs

---

## Future Enhancements

### Roadmap

```mermaid
timeline
    title State Management Hooks Roadmap
    
    section Phase 1 - Current
        URL State Management : useUrlState hook
        Server Persistence : useRecordFlow hook
        Component Preservation : KeepAlive component
    
    section Phase 2 - Near Term
        State Validation : Schema validation for URL state
        Middleware Support : Pre/post processing hooks
        Enhanced Caching : LRU cache for records
    
    section Phase 3 - Future
        State Versioning : Migration support for state changes
        Offline Support : IndexedDB integration
        State Analytics : Usage tracking and insights
        
    section Phase 4 - Advanced
        Distributed State : Multi-tab synchronization
        Time Travel : State history and undo/redo
        AI Optimization : Predictive state preloading
```

**Planned Features**:

1. **State Validation**:
   - Schema-based validation for URL parameters
   - Type guards for runtime safety
   - Automatic sanitization of invalid values

2. **Enhanced Caching**:
   - LRU cache for frequently accessed records
   - Configurable cache expiration
   - Cache invalidation strategies

3. **Offline Support**:
   - IndexedDB integration for offline persistence
   - Sync queue for pending saves
   - Conflict resolution strategies

4. **Developer Tools**:
   - State debugging panel
   - Time-travel debugging
   - State change visualization

---

## API Reference

### useUrlState

```typescript
function useUrlState<S>(
  initialState?: S | (() => S),
  options?: Options
): [S, (s: React.SetStateAction<S>) => void]
```

**Parameters**:
- `initialState`: Initial state object or factory function
- `options`: Configuration options
  - `navigateMode`: "push" | "replace" (default: "push")
  - `parseOptions`: qs.IParseOptions
  - `stringifyOptions`: qs.IStringifyOptions

**Returns**: Tuple of [state, setState]

**Example**:
```typescript
const [filters, setFilters] = useUrlState<FilterState>({
  category: [],
  dateRange: null
}, {
  navigateMode: 'replace'
})
```

---

### useRecordFlow

```typescript
function useRecordFlow<T>({ 
  id, 
  options 
}: RecordFlowProps): {
  record: object
  saveRecord: (params: T) => void
  loading: boolean
}
```

**Parameters**:
- `id`: RECORD_ID enum value
- `options`: Configuration options
  - `exclude`: Fields to exclude from persistence
  - `extend`: Additional fields to include

**Returns**: Object with record, saveRecord function, and loading state

**Example**:
```typescript
const { record, saveRecord, loading } = useRecordFlow<RunwayParams>({
  id: RECORD_ID.RUNWAY
})
```

---

### KeepAlive

```typescript
function KeepAlive({ 
  children, 
  active 
}: KeepAliveProps): JSX.Element
```

**Props**:
- `children`: ReactNode to preserve
- `active`: Boolean controlling visibility

**Example**:
```typescript
<KeepAlive active={currentTab === 'analysis'}>
  <AnalysisComponent />
</KeepAlive>
```

---

## Troubleshooting

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| URL not updating | Navigate mode misconfigured | Check `navigateMode` option |
| State not persisting | Record ID mismatch | Verify RECORD_ID enum value |
| Component remounting | KeepAlive active prop incorrect | Ensure boolean active prop |
| Parse errors | Complex nested objects | Adjust `depth` in parseOptions |
| Save failures | Network issues | Check service layer integration |
| Memory leaks | Too many KeepAlive instances | Limit preserved components |

---

## Best Practices Summary

1. **URL State**:
   - Use for shareable, bookmarkable state
   - Keep URL parameters minimal and readable
   - Implement proper TypeScript types
   - Configure appropriate parse/stringify options

2. **Record Flow**:
   - Use for user preferences and saved configurations
   - Implement proper error handling
   - Provide user feedback for save operations
   - Consider caching strategies for performance

3. **KeepAlive**:
   - Use sparingly for expensive components
   - Implement proper cleanup logic
   - Consider memory implications
   - Test navigation scenarios thoroughly

4. **General**:
   - Combine hooks appropriately for complex scenarios
   - Document state shape and persistence strategy
   - Implement comprehensive error handling
   - Test edge cases and error scenarios

---

## Related Documentation

- [view-modules.md](view-modules.md) - View components using these hooks
- [business-components.md](business-components.md) - Business components integration
- [ui-component-system.md](ui-component-system.md) - Toast and other UI components
- [core-infrastructure.md](core-infrastructure.md) - Service layer and routing

---

## Conclusion

The **state-management-hooks** module provides a robust, flexible foundation for managing application state across multiple persistence layers. By combining URL state management, server-side persistence, and component state preservation, it enables complex state management scenarios while maintaining a clean, intuitive API. The module's integration with React Router, ahooks, and the service layer ensures seamless operation within the broader application architecture.

Key strengths:
- **Flexibility**: Multiple persistence strategies for different use cases
- **Type Safety**: Full TypeScript support with generics
- **Performance**: Optimized with memoization and caching
- **Developer Experience**: Intuitive APIs with comprehensive error handling
- **Integration**: Seamless integration with existing application infrastructure

This module is essential for building modern, stateful web applications that require sophisticated state management capabilities.
