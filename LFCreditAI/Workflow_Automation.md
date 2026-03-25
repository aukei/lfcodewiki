# Workflow Automation Module

## Overview
The **Workflow Automation** module is the orchestration engine of the Credit AI system. It manages the end-to-end lifecycle of credit assessments, from triggering periodic re-assessments to generating AI-driven credit recommendations and synchronizing data with external business systems like DTC (Data Tracking Center) and UiPath RPA robots.

This module acts as a bridge between the [AI Engine Models](AI_Engine_Models.md), [Credit Report Service](Credit_Report_Service.md), and external automation platforms.

## Architecture
The module follows a service-oriented architecture where API resources trigger complex workflows managed by specialized managers and utility helpers.

```mermaid
graph TD
    WA_API[WorkflowRecommendationAPI] --> AI_Search[AI Search API]
    WA_API --> AI_Profile[AI Company Profile API]
    WA_API --> AI_Draft[AI Draft API]
    WA_API --> DTC[DTCManager]
    
    WR_API[WorkflowReassessAPI] --> DB[(Database)]
    WR_API --> UIPath[UiPathHelper]
    
    DTC --> DTC_External[External DTC API]
    UIPath --> UiPath_Cloud[UiPath Orchestrator]
    
    subgraph "Internal Services"
        DTC
        UIPath
    end
```

## Key Sub-modules

### 1. [Assessment Workflow](assessment_workflow.md)
Handles the logic for generating credit recommendations and managing periodic reviews. It orchestrates multiple AI calls to gather company background, external ratings, and financial drafts to produce a comprehensive credit memo.

### 2. [External Integration](external_integration.md)
Manages communication with third-party systems:
- **DTC Manager**: Synchronizes assessment results with the Data Tracking Center (DTC) via REST APIs.
- **UiPath Helper**: Interfaces with UiPath Orchestrator to trigger RPA jobs for automated data computation and exposure analysis.

## Process Flow: Credit Recommendation
The following diagram illustrates the sequence of events when a recommendation workflow is triggered:

```mermaid
sequenceDiagram
    participant Client
    participant API as WorkflowRecommendationAPI
    participant AI as AI Engine
    participant DB as Database
    participant DTC as DTCManager

    Client->>API: GET /workflow/rec
    API->>DB: Fetch Report & Workflow Status
    API->>AI: Request Search & Background
    AI-->>API: Return Company Data
    API->>AI: Request Company Profile
    AI-->>API: Return Profile
    API->>AI: Request AI Draft (Memo)
    AI-->>API: Return Drafted Memo
    API->>DB: Update Credit Report (v+1)
    API->>DTC: update_sheet()
    DTC->>DTC: Prepare & Patch Data
    API-->>Client: Success (New Report ID)
```

## Component Summary
| Component | Responsibility |
|-----------|----------------|
| `WorkflowReassessAPI` | Identifies reports due for review and triggers re-assessment workflows. |
| `WorkflowRecommendationAPI` | Orchestrates the AI-driven drafting process for credit memos. |
| `DTCManager` | Formats and pushes credit assessment data to the DTC platform. |
| `UiPathHelper` | Interfaces with UiPath Orchestrator to start automation jobs. |
