# LFAIDesign Repository Overview

## Purpose
The **LFAIDesign** repository is a comprehensive AI-driven design ideation and fashion technology platform. It is designed to streamline the creative workflow for designers by integrating state-of-the-art generative AI models (Stable Diffusion, DALL-E 3, Flux, and Gemini) with specialized fashion tools. 

The platform enables users to perform visual searches for inspiration, generate iterative design variations, create technical sketches from photos, train custom LoRA models for specific styles, and perform virtual try-ons. It serves as an end-to-end "Creative Sandbox" that bridges the gap between initial design concepts and production-ready technical assets.

---

## End-to-End Architecture
The repository follows a modular architecture where a **Streamlit-based Frontend** coordinates between various **AI Service Integrations**, **Cloud Storage**, and **Relational Databases**.

```mermaid
graph TD
    subgraph Client_Layer [Frontend - Streamlit]
        IW[Ideation Workbench]
        LTM[LoRA Training Module]
        DIC[Design Ideation Core]
    end

    subgraph Logic_Layer [Backend Orchestration]
        Util[Streamlit UI Utilities]
        GMA[GenAI Multimodal Analysis]
    end

    subgraph AI_Service_Layer [AI & Search Engines]
        CUI[ComfyUI Automation - AutoDL]
        OAI[OpenAI Integration - DALL-E 3/GPT-4o]
        FAI[Fal AI Integration - Flux/LoRA]
        SS[Search Services - Bing/Google]
    end

    subgraph Data_Layer [Storage & Persistence]
        AZ[Azure Cloud Storage - Blobs/Images]
        DB[Database Management - PostgreSQL]
    end

    %% Interactions
    IW --> Logic_Layer
    LTM --> Logic_Layer
    DIC --> Logic_Layer

    Logic_Layer --> AI_Service_Layer
    
    AI_Service_Layer --> AZ
    LTM --> DB
    DIC --> DB
    
    CUI -.-> |Workflows| OAI
    SS --> |Inspiration| IW
```

### Key Workflow: Technical Sketch Generation
This diagram illustrates the cross-module interaction required to transform a design photo into a technical sketch:

```mermaid
sequenceDiagram
    participant User
    participant IW as Ideation Workbench
    participant GMA as Multimodal Analysis
    participant CUI as ComfyUI Automation
    participant AZ as Azure Storage

    User->>IW: Upload Image & Request Sketch
    IW->>GMA: Analyze Garment Attributes (Gemini/GPT-4o)
    GMA-->>IW: Return Detailed Description
    IW->>CUI: Trigger Sketch Workflow (ComfyUI API)
    CUI->>CUI: Process Image-to-Sketch Nodes
    CUI-->>IW: Return Generated Sketch
    IW->>AZ: Persist Sketch Image
    AZ-->>User: Display Final Technical Sketch
```

---

## Core Modules Documentation

The following modules represent the functional pillars of the LFAIDesign system:

| Module | Description |
| :--- | :--- |
| **[Ideation Workbench](app/backend/pages/Ideation_Workbench.py)** | The primary creative interface for iterative image generation, visual search, and design evolution. |
| **[ComfyUI Automation](app/backend/utils/autodl_comfyui_util.py)** | Manages remote execution of complex node-based workflows for virtual try-ons and clothing extraction. |
| **[OpenAI Integration](app/backend/utils/openai_util.py)** | Handles DALL-E 3 image generation, GPT-4o multimodal analysis, and prompt refinement. |
| **[Database Management](app/backend/utils/db_util.py)** | Manages PostgreSQL persistence for user quotas, LoRA metadata, and access control. |
| **[Azure Cloud Storage](app/backend/utils/azure_util.py)** | Provides a robust interface for storing and retrieving large binary assets like models and generated images. |
| **[Search Services](app/backend/utils/bing_util.py)** | Integrates Bing and Google APIs for visual and keyword-based design inspiration. |
| **[GenAI Multimodal Analysis](app/backend/utils/genai_util.py)** | Uses Gemini and other LLMs to identify garment types and measure design attributes. |
| **[Fal AI Integration](app/backend/utils/fal_util.py)** | Facilitates high-end Flux model generation and cloud-based LoRA training. |