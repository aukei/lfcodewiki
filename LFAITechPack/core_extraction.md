# Core Extraction Module

## Introduction
The `core_extraction` module serves as the foundational layer for the system's data extraction engine. It provides a standardized framework and factory mechanism for extracting structured information (such as Bill of Materials, Colorways, and SKU details) from various technical package (TechPack) formats, including PDFs and Excel files.

By utilizing an abstract base class and a factory pattern, the module ensures consistency across different customer-specific extraction implementations while allowing for specialized logic required by different retail brands.

## Architecture and Component Relationships

### Component Overview
The module is structured around two primary components:
1.  **`BaseExtractionService`**: An abstract base class (ABC) that defines the lifecycle and common utility methods for all extraction services.
2.  **`ExtractionServiceFactory`**: A creational pattern implementation that manages the registration and instantiation of customer-specific extraction services.

### Class Diagram
```mermaid
classDiagram
    class ExtractionServiceFactory {
        -_services: Dict
        +create_service(customer_name: str) BaseExtractionService
        +register_service(customer_code, service_class)
        +get_supported_customers() List~str~
    }

    class BaseExtractionService {
        <<abstract>>
        +customer_name: str
        +extract_all_data(file_path, file_type) Dict
        +extract_cover_info(file_path)* Dict
        +extract_sku_info(file_path)* Dict
        +extract_materials_info(file_path)* List
        +extract_colorways_info(file_path)* List
        +validate_file(file_path) bool
        #_normalize_text(text) str
    }

    class LandsEndExtractionService {
        +extract_cover_info()
    }
    class WalmartExtractionService {
        +extract_cover_info()
    }
    class DOENExtractionService {
        +extract_cover_info()
    }

    BaseExtractionService <|-- LandsEndExtractionService
    BaseExtractionService <|-- WalmartExtractionService
    BaseExtractionService <|-- DOENExtractionService
    ExtractionServiceFactory ..> BaseExtractionService : creates
```

## Functional Workflow

### Data Extraction Process
The `BaseExtractionService` implements a template method `extract_all_data` which orchestrates the extraction sequence.

```mermaid
sequenceDiagram
    participant Client
    participant Factory as ExtractionServiceFactory
    participant Service as SpecificExtractionService
    
    Client->>Factory: create_service("WALMART")
    Factory-->>Client: WalmartExtractionService Instance
    
    Client->>Service: extract_all_data(file_path)
    Service->>Service: validate_file()
    
    par Extraction Steps
        Service->>Service: extract_cover_info()
        Service->>Service: extract_sku_info()
        Service->>Service: extract_materials_info()
        Service->>Service: extract_colorways_info()
    end
    
    Service->>Service: postprocess_results()
    Note right of Service: CamelCase transformation
    
    Service-->>Client: Final JSON Results
```

## Key Features

### 1. Text Normalization
The module includes robust text normalization logic in `BaseExtractionService` to handle common PDF extraction issues:
*   **Ligature Resolution**: Converts special characters like `ﬁ`, `ﬂ`, `ﬀ` into standard character sequences (`fi`, `fl`, `ff`).
*   **Whitespace Cleaning**: Normalizes non-breaking spaces, thin spaces, and various Unicode invisible characters.
*   **Hyphen/Underscore Handling**: Specialized cleaning for technical identifiers that may contain line-break hyphens.

### 2. Extensibility
New customers can be added by:
1.  Inheriting from `BaseExtractionService`.
2.  Implementing the abstract extraction methods.
3.  Registering the new class in `ExtractionServiceFactory`.

## System Integration
The `core_extraction` module is a dependency for several other high-level services:
*   **[techpack_core_service](techpack_core_service.md)**: Uses extraction to populate the core repository.
*   **[xts_transformation](xts_transformation.md)**: Consumes extracted data to transform it into XTS-compatible formats.
*   **[ai_llm_providers](ai_llm_providers.md)**: Often works in tandem with extraction services to parse unstructured text using Gemini or OpenAI.

## Data Flow
1.  **Input**: Raw file path (PDF/XLSX) and Customer Identifier.
2.  **Processing**: 
    *   Factory selects the driver.
    *   Driver parses document structure.
    *   Utilities normalize text and regex patterns match sizes/SKUs.
3.  **Output**: A standardized dictionary containing `covers`, `materials`, `colorways`, and `skus` with keys transformed to `camelCase` for frontend compatibility.
