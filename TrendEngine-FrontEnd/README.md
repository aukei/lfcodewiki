# TrendEngine-FrontEnd Documentation

Welcome to the comprehensive documentation for the TrendEngine-FrontEnd module. This documentation provides detailed insights into the architecture, components, and patterns used throughout the application.

## 📚 Documentation Structure

### Main Documentation
- **[TrendEngine-FrontEnd.md](./TrendEngine-FrontEnd.md)** - Main module overview, architecture, and integration guide

### Sub-Module Documentation

1. **[Core Infrastructure](./core-infrastructure.md)**
   - Environment configuration and type definitions
   - Convention-based routing system
   - Build-time code generation
   - Global type augmentations

2. **[UI Component System](./ui-component-system.md)**
   - Base UI components (Button, Badge, Text, Toast, Icons)
   - Form components with context-based state management
   - Layout components (Flex, Grid, Masonry)
   - Input components (Combobox, Cascader)

3. **[Business Components](./business-components.md)**
   - Domain-specific components
   - Filter components (Category, Time, Color, Style)
   - Image handling (ImageBox, Upload)
   - Batch operations
   - Chart components (ECharts wrapper)

4. **[View Modules](./view-modules.md)**
   - Consumer analysis views
   - Product detail views
   - Search result views
   - Runway analysis views
   - Market analysis views
   - Settings and configuration views

5. **[State Management & Hooks](./state-management-hooks.md)**
   - URL state management (useUrlState)
   - Server-side record persistence (useRecordFlow)
   - Component state preservation (KeepAlive)
   - Custom hooks and patterns

6. **[Utilities & Helpers](./utilities-helpers.md)**
   - File export utilities (Excel, CSV, Text)
   - Image download and ZIP creation
   - Data formatting utilities

## 🎯 Quick Navigation

### For New Developers
1. Start with [TrendEngine-FrontEnd.md](./TrendEngine-FrontEnd.md) for an overview
2. Read [Core Infrastructure](./core-infrastructure.md) to understand the foundation
3. Explore [UI Component System](./ui-component-system.md) for available components
4. Review [State Management & Hooks](./state-management-hooks.md) for state patterns

### For Component Development
1. [UI Component System](./ui-component-system.md) - Base components
2. [Business Components](./business-components.md) - Domain components
3. [View Modules](./view-modules.md) - Page-level components

### For Feature Development
1. [View Modules](./view-modules.md) - Feature views
2. [State Management & Hooks](./state-management-hooks.md) - State patterns
3. [Business Components](./business-components.md) - Reusable components

### For Architecture Understanding
1. [TrendEngine-FrontEnd.md](./TrendEngine-FrontEnd.md) - High-level architecture
2. [Core Infrastructure](./core-infrastructure.md) - Build and runtime infrastructure
3. All sub-module documentation for detailed component relationships

## 🔑 Key Concepts

### Convention-Based Routing
The application uses file-system based routing that automatically generates routes from the directory structure. See [Core Infrastructure](./core-infrastructure.md#route-generation-system) for details.

### Form System
A powerful context-based form system that automatically manages state. See [UI Component System](./ui-component-system.md#form-components) for details.

### Component Composition
Components use Radix UI's Slot pattern for flexible composition. See [UI Component System](./ui-component-system.md#component-composition-patterns) for examples.

### State Management
Multiple state management strategies for different use cases:
- URL state for shareable filters
- Context for form state
- Custom hooks for data fetching
- See [State Management & Hooks](./state-management-hooks.md) for details

## 📊 Architecture Diagrams

Each documentation file includes Mermaid diagrams showing:
- Component hierarchies
- Data flow patterns
- Module dependencies
- Integration points

## 🛠️ Technology Stack

- **Framework**: React 18+ with TypeScript
- **Build Tool**: Vite
- **Routing**: React Router v6
- **State Management**: React Context + ahooks
- **UI Components**: Radix UI + Tailwind CSS
- **Charts**: ECharts
- **Forms**: Custom form system
- **i18n**: i18next

## 📖 Documentation Features

- ✅ Comprehensive component API documentation
- ✅ Architecture diagrams with Mermaid
- ✅ Code examples and usage patterns
- ✅ Integration guidelines
- ✅ Best practices and conventions
- ✅ Cross-references between modules
- ✅ Type definitions and interfaces

## 🔗 Module Dependencies

```
Core Infrastructure
    ↓
UI Component System
    ↓
Business Components
    ↓
View Modules
    ↓
(Uses State Management & Hooks + Utilities throughout)
```

## 📝 Notes

- All documentation files are written in Markdown with Mermaid diagrams
- Code examples use TypeScript
- Component props are fully documented with TypeScript interfaces
- Cross-references use relative links for easy navigation

## 🚀 Getting Started

1. Read the [main documentation](./TrendEngine-FrontEnd.md)
2. Explore sub-modules based on your needs
3. Refer to code examples in each section
4. Follow the architecture patterns described

---

**Last Updated**: 2024
**Module Version**: TrendEngine-FrontEnd
**Documentation Format**: Markdown + Mermaid
