# View Modules Documentation

## Overview

The **view-modules** layer represents the presentation and page-level components of the TrendEngine application. This module contains feature-specific views that compose UI components, business components, and state management to deliver complete user experiences across different functional areas including consumer analysis, product details, search results, runway analysis, market analysis, libraries, internal tools, and system settings.

This module serves as the top-level orchestration layer that:
- Integrates UI components from the [ui-component-system](ui-component-system.md)
- Utilizes business components from the [business-components](business-components.md)
- Implements page-specific logic and data fetching
- Manages view-level state and user interactions
- Provides navigation and routing structures

---

## Architecture Overview

```mermaid
graph TB
    subgraph "View Modules Layer"
        CA[Consumer Analysis Views]
        PD[Product Detail Views]
        SR[Search Result Views]
        RA[Runway Analysis Views]
        MA[Market Analysis Views]
        LIB[Libraries Views]
        INT[Internal Tools Views]
        SET[Settings Views]
    end
    
    subgraph "Business Components Layer"
        BC[Business Components]
        FILTER[Filter Components]
        CHART[Chart Components]
        PANEL[Panel Components]
    end
    
    subgraph "UI Component System"
        UI[UI Components]
        FORM[Form Components]
        LAYOUT[Layout Components]
    end
    
    subgraph "State & Data"
        HOOKS[Custom Hooks]
        SERVICE[API Services]
        STORE[State Management]
    end
    
    CA --> BC
    PD --> BC
    SR --> BC
    RA --> BC
    MA --> BC
    LIB --> FILTER
    INT --> FILTER
    SET --> UI
    
    BC --> UI
    FILTER --> FORM
    CHART --> UI
    PANEL --> UI
    
    CA --> HOOKS
    PD --> SERVICE
    SR --> SERVICE
    RA --> STORE
    MA --> STORE
    
    style CA fill:#e1f5ff
    style PD fill:#e1f5ff
    style SR fill:#e1f5ff
    style RA fill:#e1f5ff
    style MA fill:#e1f5ff
    style LIB fill:#e1f5ff
    style INT fill:#e1f5ff
    style SET fill:#e1f5ff
```

---

## Module Structure

```mermaid
graph LR
    subgraph "View Modules Organization"
        WS[work-stage/]
        
        subgraph "Analysis Views"
            CA[consumer-analysis/]
            RA[runway-analysis/]
            MA[market-analysis/]
        end
        
        subgraph "Detail Views"
            DET[detail/]
            PROD[product/]
            RUN[runway-analysis/]
            INS[instagrammer/]
        end
        
        subgraph "Search & Libraries"
            SR[search-result/]
            LIB[libraries/]
        end
        
        subgraph "Internal & Settings"
            INT[lf-internal/]
            SET[setting/]
        end
    end
    
    WS --> CA
    WS --> RA
    WS --> MA
    WS --> DET
    WS --> SR
    WS --> LIB
    WS --> INT
    WS --> SET
    
    DET --> PROD
    DET --> RUN
    DET --> INS
```

---

## Core Components

### 1. Consumer Analysis Views

#### TrendDiscovery Component
**Location**: `view/work-stage/consumer-analysis/trend-discovery.tsx`

A comprehensive trend analysis component that displays growth and declining trends with tabular data visualization.

**Key Features**:
- Dual analysis modes (growth/declining trends)
- Interactive data table with keyword selection
- Growth percentage visualization
- Post count tracking with tooltips

**Component Structure**:
```mermaid
graph TB
    TD[TrendDiscovery]
    TD --> Panel[Panel.Root]
    TD --> Title[Panel.Title with SubTitle]
    TD --> Content[Panel.Content]
    TD --> Table[TrendDiscovery.Table]
    
    Table --> DataTable[DataTable Component]
    Table --> Columns[Column Definitions]
    
    Columns --> C1[Keyword Column]
    Columns --> C2[Posts Column]
    Columns --> C3[Growth % Column]
    
    Title --> GrowthIcon[ArrowUpRiseIcon]
    Title --> DeclineIcon[ArrowDownDropIcon]
    
    style TD fill:#e3f2fd
    style Table fill:#fff3e0
```

**Props Interface**:
```typescript
interface IProps {
  title: string | React.ReactNode
  analysisType?: ANALYSIS_TYPE  // 1-growth, 2-decline
  loading?: boolean
}

interface TableProps {
  data: ConsumerAnalysisKeywordDto[]
  keyword?: string
  onKeywordChange?: (keyword?: string) => void
  columns?: ColumnDef<ConsumerAnalysisKeywordDto>[]
}
```

**Usage Pattern**:
```typescript
<TrendDiscovery 
  title="Trend Discovery" 
  analysisType={ANALYSIS_TYPE.growth}
>
  <TrendDiscovery.Table
    data={trendData}
    keyword={selectedKeyword}
    onKeywordChange={handleKeywordChange}
  />
</TrendDiscovery>
```

---

### 2. Product Detail Views

#### Product Descriptions Overview
**Location**: `view/work-stage/detail/product/sku/product-descriptions/index.default.tsx`

Displays comprehensive product metrics and trends over time with granular data visualization.

**Data Flow**:
```mermaid
sequenceDiagram
    participant User
    participant Overview
    participant TimeCombobox
    participant API
    participant Chart
    
    User->>Overview: Load Component
    Overview->>API: serviceGoodsMetricTrend()
    API-->>Overview: GoodsDetailTrendVo
    Overview->>Overview: Process Metrics
    Overview->>Chart: Render Trend Chart
    
    User->>TimeCombobox: Select Date Range
    TimeCombobox->>Overview: Update params
    Overview->>API: Fetch New Data
    API-->>Overview: Updated Metrics
    Overview->>Chart: Update Visualization
```

**Key Metrics Tracked**:
- Sales Volume
- Sales Amount
- Comments/Reviews
- Favorites Count
- Stock Changes
- Out of Stock Rate

**Granularity Component**:
```typescript
const Granularity = ({
  label: string,
  tip?: string,
  value: string,
  secondary?: React.ReactNode
}) => {
  // Displays individual metric with tooltip and secondary info
}
```

---

#### Product Tabs Component
**Location**: `view/work-stage/detail/product/tabs.tsx`

A reusable tab navigation component for product detail sections.

**Component Structure**:
```mermaid
graph LR
    Tabs[Tabs Component]
    Tabs --> FormRoot[Form.Root]
    FormRoot --> RadioGroup[Form.RadioGroup]
    RadioGroup --> Items[RadioGroupItems]
    
    Items --> Tab1[Tab Option 1]
    Items --> Tab2[Tab Option 2]
    Items --> TabN[Tab Option N]
    
    style Tabs fill:#e8f5e9
```

**Props Interface**:
```typescript
interface TabOption {
  label: string
  value: string
  [key: string]: string
}

interface TabProps {
  name: string
  value: Record<string, unknown>
  onValueChange: (val: Record<string, unknown>) => void
  options?: TabOption[]
}
```

---

#### Runway Image Box
**Location**: `view/work-stage/detail/runway-analysis/runway-image-box.tsx`

An advanced image viewer with thumbnail navigation, label overlays, and navigation controls.

**Features**:
- Thumbnail sidebar with scroll
- Full-size image display
- Label positioning overlays
- Previous/Next navigation
- Integration with image list context

**Component Architecture**:
```mermaid
graph TB
    IB[ImageBox]
    IB --> SA[ScrollArea - Thumbnails]
    IB --> Main[Main Image Display]
    IB --> Labels[Label Overlays]
    IB --> Nav[Navigation Arrows]
    
    SA --> Thumb1[Thumbnail 1]
    SA --> Thumb2[Thumbnail 2]
    SA --> ThumbN[Thumbnail N]
    
    Main --> Image[Full Size Image]
    
    Labels --> L1[Label Position 1]
    Labels --> L2[Label Position 2]
    
    Nav --> Prev[Previous Arrow]
    Nav --> Next[Next Arrow]
    
    style IB fill:#fff3e0
    style Main fill:#e1f5fe
```

**Label Interface**:
```typescript
interface LabelItem {
  top: string      // CSS position
  left: string     // CSS position
  label: string    // Label text
}
```

---

#### Instagrammer Summary Cards
**Location**: `view/work-stage/detail/instagrammer/summary-cards.tsx`

Displays influencer metrics in a card-based layout with growth indicators.

**Card Configuration Pattern**:
```mermaid
graph TB
    SC[SummaryCards]
    SC --> Config[CardConfig Array]
    
    Config --> C1[Card 1 Config]
    Config --> C2[Card 2 Config]
    Config --> CN[Card N Config]
    
    C1 --> Title1[Title Key]
    C1 --> Tip1[Tooltip Key]
    C1 --> Summary1[Summary Key]
    C1 --> Info1[Info Keys Array]
    
    SC --> Hook[useSummaryCard Hook]
    Hook --> Cards[Card Components]
    
    Cards --> Card1[Card with Metrics]
    Cards --> Card2[Card with Metrics]
    
    Card1 --> Value[Main Value]
    Card1 --> InfoList[Info List]
    InfoList --> Growth[Growth Indicator]
    
    style SC fill:#f3e5f5
    style Hook fill:#e8eaf6
```

**Configuration Interface**:
```typescript
interface CardConfig {
  titleKey: string
  tipKey: string
  summaryKey: keyof Summary
  infoKeys: {
    label: string
    valueKey: keyof Summary | null
    showGrowth?: boolean
    value?: (v?: Summary) => string
  }[]
}
```

**Growth Cell Component**:
- Displays percentage with color coding
- Green with up arrow for positive growth
- Red with down arrow for negative growth
- Neutral for zero growth

---

### 3. Search Result Views

#### Search Header Component
**Location**: `view/work-stage/search-result/search-header.tsx`

Provides search interface with relevant results and navigation tabs.

**Component Flow**:
```mermaid
sequenceDiagram
    participant User
    participant SearchHeader
    participant RelativeResult
    participant API
    participant NavTabs
    
    User->>SearchHeader: Enter Search
    SearchHeader->>API: serviceSearchInterfaceAssociationWord()
    API-->>RelativeResult: Related Keywords
    RelativeResult->>User: Display Suggestions
    
    User->>RelativeResult: Click Suggestion
    RelativeResult->>SearchHeader: Update Query
    SearchHeader->>NavTabs: Navigate to Tab
```

**Tab Options**:
- All Results
- Products
- Posts
- Influencers
- Sites

---

#### Content Component
**Location**: `view/work-stage/search-result/all/content.tsx`

A layout wrapper for search result sections with title and action links.

**Structure**:
```mermaid
graph LR
    Content[Content Component]
    Content --> Header[Header Section]
    Content --> Body[Children Content]
    
    Header --> Title[Keyword + Title]
    Header --> Link[Action Link]
    
    Title --> Keyword[Highlighted Keyword]
    Title --> Text[Section Title]
    
    style Content fill:#e8f5e9
```

---

#### Info Card Component
**Location**: `view/work-stage/search-result/all/info-card.tsx`

Displays search results in a card format with images and descriptions.

**Props Interface**:
```typescript
interface InfoCardProps {
  title?: string
  list?: {
    picUrl?: string
    displayName?: string
    description?: string
    link?: string
  }[]
}
```

---

### 4. Analysis Statistics Views

#### Runway Analysis Statistics
**Location**: `view/work-stage/runway-analysis/analysis-statistics/index.tsx`

Provides tabbed navigation for runway analysis metrics.

**Tab Structure (Apparel Industry)**:
```mermaid
graph LR
    RAS[Runway Analysis Statistics]
    RAS --> Filter[RunwayAnalysisFilter]
    RAS --> Tabs[NavTabs]
    
    Tabs --> Cat[Category]
    Tabs --> Mat[Material]
    Tabs --> Col[Colors]
    Tabs --> Pat[Pattern]
    Tabs --> Det[Detail]
    
    Tabs --> Outlet[Outlet for Tab Content]
    
    style RAS fill:#fce4ec
    style Tabs fill:#f3e5f5
```

**Dynamic Tab Configuration**:
- Tabs adjust based on industry type
- Apparel industry shows additional tabs (Colors, Pattern)
- Auto-navigation to first valid tab

---

#### Market Analysis Statistics
**Location**: `view/work-stage/market-analysis/analysis-statistics/index.tsx`

Similar structure to runway analysis with market-specific metrics.

**Features**:
- Search textbox for product filtering
- Total result count display
- Industry-specific tab visibility
- Integration with market analysis state management

**Tab Configuration Logic**:
```mermaid
graph TB
    Check{Industry Type?}
    Check -->|Apparel| ApparelTabs[Product Assortment<br/>Raw Material<br/>Colors<br/>Pattern<br/>Detail<br/>Price]
    Check -->|Footwear| FootwearTabs[Raw Material<br/>Detail<br/>Price]
    Check -->|Other| DefaultTabs[Price]
    
    ApparelTabs --> Render[Render Tabs]
    FootwearTabs --> Render
    DefaultTabs --> Render
    
    style Check fill:#fff9c4
```

---

### 5. Libraries Views

#### Pinterest Filter
**Location**: `view/work-stage/libraries/pinterest/filter.tsx`

Comprehensive filtering interface for Pinterest content.

**Filter Components**:
```mermaid
graph TB
    PF[Pinterest Filter]
    PF --> Form[Form.Root]
    
    Form --> CC[CategoryCombobox]
    Form --> IC[Industry & Category Cascader]
    Form --> Colors[Fashion Color Cascader]
    Form --> Styles[Fashion Style Combobox]
    Form --> Labels[Design Element Cascader]
    Form --> Time[Time Combobox]
    
    IC --> Multi1[Multi-select]
    Colors --> Multi2[Multi-select]
    Styles --> Multi3[Multi-select]
    Labels --> Multi4[Multi-select]
    
    Time --> DateRange[Date Range Picker]
    
    style PF fill:#e0f2f1
    style Form fill:#e8f5e9
```

**Props Interface**:
```typescript
interface FilterProps {
  value: PinterestBlogQueryRequest
  onValueChange: (value: PinterestBlogQueryRequest) => void
}
```

---

### 6. Internal Tools Views

#### POM Filter
**Location**: `view/work-stage/lf-internal/apparel-baby/pom/PomFilter.tsx`

Specialized filter for internal Product Operations Management.

**Filter Structure**:
```mermaid
graph LR
    PF[POM Filter]
    PF --> Brand[Brand Combobox]
    PF --> Year[Year Multi-select]
    PF --> Item[Item/Style Combobox]
    
    Item --> Search[Search with Debounce]
    Search --> Options[Dynamic Options]
    
    Brand --> Filter1[Filter Options]
    Year --> Filter2[Filter Options]
    Filter1 --> Options
    Filter2 --> Options
    
    style PF fill:#fff3e0
```

**Props Interface**:
```typescript
interface PDFFilterParams {
  assortmentKeyword?: string
  brandList?: string[]
  yearList?: string[]
  itemNumber?: string
}
```

**Key Features**:
- Cascading filter dependencies
- Debounced search for item numbers
- Dynamic option loading based on selections

---

#### Overview Utils
**Location**: `view/work-stage/lf-internal/overview/utils.ts`

Utility functions for internal overview tables.

**Column Configuration**:
```typescript
interface Column {
  title: string
  dataIndex: string
  tip?: string
}

// Dynamic column generation based on GroupByType
const getColumns = (type: GroupByType): Column[]
```

**Supported Group Types**:
- BRAND
- FAMILY
- PRODUCT_TYPE
- CAT1 (Gender)
- CAT2 (Category Group)
- CAT3 (Category)

---

### 7. Settings Views

#### Role Types
**Location**: `view/work-stage/setting/types.ts`

Type definitions for role management.

```typescript
interface RoleInfo {
  roleName?: string
  roleId?: string
}

type OptionType = 
  | "corporateCustomer"
  | "family"
  | "productType"
  | "brand"
  | "pomBrand"
  | "operatingGroupCode"
  | "fiber"
  | "fiberSustainability"
  | "year"
```

---

#### Delete Role Component
**Location**: `view/work-stage/setting/roles-config/delete-role.tsx`

Modal dialog for role deletion confirmation.

**Component Flow**:
```mermaid
sequenceDiagram
    participant User
    participant Dialog
    participant API
    participant Toast
    
    User->>Dialog: Click Delete
    Dialog->>User: Show Confirmation
    User->>Dialog: Confirm
    Dialog->>API: deleteserviceAuthorityRole()
    
    alt Success
        API-->>Dialog: Success Response
        Dialog->>Toast: Show Success
        Dialog->>Dialog: onConfirm Callback
    else Error
        API-->>Dialog: Error Response
        Dialog->>Toast: Show Error
    end
```

**Props Interface**:
```typescript
interface DeleteRoleProps {
  roleId?: string
  open?: boolean
  setOpen?(open: boolean): void
  onConfirm?: () => void
}
```

---

## Data Flow Patterns

### 1. URL State Management Pattern

```mermaid
graph TB
    URL[URL Parameters]
    URL --> Hook[useUrlState Hook]
    Hook --> Component[View Component]
    
    Component --> Update[Update State]
    Update --> Hook
    Hook --> URL
    
    Component --> API[API Request]
    API --> Data[Fetch Data]
    Data --> Component
    
    style Hook fill:#e1f5fe
```

**Usage Example**:
```typescript
const [queryParams, setQueryParams] = useUrlState<SearchQueryProps>()

const onParamsChange = (_params: SearchQueryProps) => {
  setQueryParams((pre) => ({ ...pre, ..._params }))
}
```

---

### 2. Form-Based Filter Pattern

```mermaid
graph LR
    Form[Form.Root]
    Form --> State[Form State]
    
    State --> Field1[Filter Field 1]
    State --> Field2[Filter Field 2]
    State --> FieldN[Filter Field N]
    
    Field1 --> Change[onValueChange]
    Field2 --> Change
    FieldN --> Change
    
    Change --> Merge[Merge Updates]
    Merge --> State
    Merge --> Parent[Parent Component]
    
    Parent --> API[API Request]
    
    style Form fill:#f3e5f5
```

---

### 3. Tab Navigation Pattern

```mermaid
graph TB
    Location[useLocation]
    Navigate[useNavigate]
    
    Location --> Check{Valid Path?}
    Check -->|No| Navigate
    Navigate --> Default[Navigate to Default Tab]
    
    Check -->|Yes| Render[Render Current Tab]
    
    User[User Click] --> NavTabItem[NavTabItem]
    NavTabItem --> Navigate
    Navigate --> Update[Update URL]
    Update --> Outlet[Outlet Renders Content]
    
    style Check fill:#fff9c4
```

---

## Integration Patterns

### 1. Business Component Integration

```mermaid
graph TB
    View[View Component]
    View --> Panel[Panel Component]
    View --> Filter[Filter Components]
    View --> Chart[Chart Components]
    
    Panel --> UI1[UI Components]
    Filter --> UI2[UI Components]
    Chart --> UI3[UI Components]
    
    View --> Service[API Services]
    Service --> Data[Data Processing]
    Data --> Panel
    Data --> Chart
    
    style View fill:#e1f5ff
    style Panel fill:#fff3e0
    style Filter fill:#f3e5f5
    style Chart fill:#e8f5e9
```

---

### 2. State Management Integration

```mermaid
graph LR
    View[View Component]
    View --> LocalState[Local State]
    View --> URLState[URL State]
    View --> GlobalStore[Global Store]
    View --> Context[Context API]
    
    LocalState --> useState[useState]
    URLState --> useUrlState[useUrlState]
    GlobalStore --> CustomHook[Custom Store Hook]
    Context --> useOutletContext[useOutletContext]
    
    style View fill:#e1f5ff
```

---

### 3. Data Fetching Pattern

```mermaid
sequenceDiagram
    participant Component
    participant useRequest
    participant Service
    participant API
    
    Component->>useRequest: Initialize with service
    useRequest->>Service: Call service function
    Service->>API: HTTP Request
    API-->>Service: Response
    Service-->>useRequest: Processed Data
    useRequest-->>Component: { data, loading, error }
    
    Component->>Component: Render with data
    
    Note over Component,useRequest: Refresh on dependency change
    Component->>useRequest: refreshDeps trigger
    useRequest->>Service: Re-fetch data
```

---

## Common Patterns and Best Practices

### 1. Component Composition Pattern

```typescript
// Parent component with sub-components
function TrendDiscovery({ children, ...props }) {
  return (
    <Panel.Root {...props}>
      <Panel.Loading />
      <Panel.Title>{title}</Panel.Title>
      <Panel.Content>{children}</Panel.Content>
    </Panel.Root>
  )
}

// Attach sub-component
TrendDiscovery.Table = Table
```

---

### 2. Conditional Rendering Pattern

```typescript
// Based on data availability
{isShowGranularity && <GranularityContent />}
{isShowChart && <TrendChart />}
{!isShowGranularity && !isShowChart && <Empty />}

// Based on loading state
{loading ? <Spinner /> : <Content />}
```

---

### 3. Dynamic Configuration Pattern

```typescript
// Tab configuration based on conditions
const tabs = useMemo<Tabs[]>(() => {
  const tabsConfig = [
    { label: "Category", path: "category" },
    isApparelIndustry && { label: "Colors", path: "colors" },
    isApparelIndustry && { label: "Pattern", path: "pattern" },
  ]
  
  return tabsConfig.filter(Boolean) as Tabs[]
}, [isApparelIndustry])
```

---

### 4. Props Spreading Pattern

```typescript
// Flexible component APIs
function Component({ 
  specificProp, 
  ...rest 
}: SpecificProps & React.ComponentProps<"div">) {
  return <div {...rest}>{specificProp}</div>
}
```

---

## Styling Patterns

### 1. Class Variance Authority (CVA)

```typescript
const tableCva = cva("bg-background cursor-pointer", {
  variants: {
    active: {
      true: "bg-muted",
      false: "bg-background",
    },
  },
  defaultVariants: { active: false },
})

// Usage
<div className={tableCva({ active: isActive })} />
```

---

### 2. Conditional Styling with cn()

```typescript
<Image
  className={cn(
    "h-32 w-32 border-4 border-border",
    { "border-transparent": i !== currentIndex }
  )}
/>
```

---

### 3. Inline Styles for Dynamic Positioning

```typescript
<div
  className="absolute"
  style={{
    top: label.top,
    left: label.left,
  }}
>
  {label.content}
</div>
```

---

## Navigation and Routing

### Route Structure

```mermaid
graph TB
    Root[work-stage]
    
    Root --> CA[consumer-analysis]
    Root --> Detail[detail]
    Root --> Search[search-result]
    Root --> Runway[runway-analysis]
    Root --> Market[market-analysis]
    Root --> Lib[libraries]
    Root --> Internal[lf-internal]
    Root --> Settings[setting]
    
    Detail --> Product[product/:id]
    Detail --> RunwayD[runway-analysis/:id]
    Detail --> Insta[instagrammer/:id]
    
    Search --> All[all]
    Search --> Products[product]
    Search --> Posts[post]
    
    Runway --> Stats[analysis-statistics]
    Market --> MStats[analysis-statistics]
    
    Stats --> Category[category]
    Stats --> Material[material]
    Stats --> Colors[colors]
    
    style Root fill:#e1f5ff
    style Detail fill:#fff3e0
    style Search fill:#f3e5f5
```

---

## Performance Considerations

### 1. Debounced Search

```typescript
const debounceKeyword = useDebounce(keyword, { wait: 500 })
```

### 2. Memoized Computations

```typescript
const isShowGranularity = useMemo(
  () => isIncludeGranularity(granulates, data?.metricList),
  [data?.metricList]
)
```

### 3. Conditional Data Fetching

```typescript
const { data } = useRequest(
  () => serviceAPI(params),
  {
    ready: !!requiredParam,
    refreshDeps: [params],
  }
)
```

---

## Error Handling

### 1. API Error Handling

```typescript
const { run } = useRequest(serviceFunction, {
  manual: true,
  onSuccess: () => {
    toast({ title: "Success" })
  },
  onError: (error) => {
    toast({
      title: "Error",
      description: error.message,
    })
  },
})
```

### 2. Empty State Handling

```typescript
{!data.list?.length ? (
  <Empty />
) : (
  <DataDisplay data={data.list} />
)}
```

---

## Internationalization

All view components use the translation function `t()` for internationalization:

```typescript
<Text t="dashboard.ty_ly_growth" />
<Text t="text.search_result" />
```

Translation keys are organized by:
- `text.*` - Common text labels
- `field.*` - Form field labels
- `dashboard.*` - Dashboard-specific terms
- `product.*` - Product-related terms
- `tips.*` - Tooltip content

---

## Dependencies

### External Dependencies
- **react-router-dom**: Navigation and routing
- **ahooks**: React hooks library (useRequest, useDebounce)
- **@tanstack/react-table**: Table functionality
- **class-variance-authority**: Styling variants
- **date-fns**: Date manipulation
- **lucide-react**: Icon components
- **qs**: Query string parsing

### Internal Dependencies
- [UI Component System](ui-component-system.md): Base UI components
- [Business Components](business-components.md): Domain-specific components
- **State Management Hooks**: Custom hooks for state management
- **API Services**: Backend integration layer

---

## Testing Considerations

### Component Testing Focus Areas

1. **User Interactions**
   - Tab navigation
   - Filter changes
   - Search functionality
   - Button clicks

2. **Data Display**
   - Correct rendering of fetched data
   - Empty state handling
   - Loading states
   - Error states

3. **Navigation**
   - Route changes
   - URL parameter updates
   - Navigation guards

4. **Form Validation**
   - Filter validation
   - Required fields
   - Date range validation

---

## Future Enhancements

### Potential Improvements

1. **Performance**
   - Implement virtual scrolling for large lists
   - Add pagination for heavy data sets
   - Optimize re-renders with React.memo

2. **User Experience**
   - Add keyboard shortcuts
   - Implement drag-and-drop for filters
   - Add export functionality

3. **Accessibility**
   - Enhance ARIA labels
   - Improve keyboard navigation
   - Add screen reader support

4. **Features**
   - Add comparison views
   - Implement saved filters
   - Add collaborative features

---

## Related Documentation

- [UI Component System](ui-component-system.md) - Base component library
- [Business Components](business-components.md) - Domain-specific components
- [Core Infrastructure](core-infrastructure.md) - Build and configuration
- **State Management Hooks** - Custom hooks documentation
- **Utilities & Helpers** - Utility functions documentation

---

## Summary

The **view-modules** layer serves as the presentation tier of the TrendEngine application, orchestrating UI components, business logic, and data management to deliver feature-rich user experiences. Key characteristics include:

- **Modular Organization**: Views organized by functional domain
- **Component Composition**: Extensive use of composition patterns
- **State Management**: Multiple state management strategies (local, URL, global)
- **Responsive Design**: Adaptive layouts and conditional rendering
- **Type Safety**: Comprehensive TypeScript interfaces
- **Internationalization**: Full i18n support
- **Performance**: Optimized data fetching and rendering

This module demonstrates modern React patterns and best practices while maintaining flexibility and extensibility for future enhancements.
