# News Processing Module

The **News Processing** module is a core component of the [News Intelligence](News_Intelligence.md) system. it is responsible for the automated discovery, ingestion, and filtering of news articles related to corporate subsidiaries and parent entities.

The module implements a producer-consumer architecture using Azure Storage Queues to handle high-volume news fetching from Google News RSS feeds, ensuring that risk-relevant information is captured and stored for further analysis by the [AI Engine Models](AI_Engine_Models.md).

## Architecture Overview

The module operates in two distinct phases:
1.  **Discovery (Producer):** Identifies keywords that need synchronization based on their last sync date and queues them for processing.
2.  **Ingestion (Consumer):** Pulls keywords from the queue, fetches RSS feeds, filters for risk alerts, and persists new entries to the database.

### Component Relationship

```mermaid
graph TD
    subgraph "News Processing Module"
        GN[GoogleNews Resource]
        GNC[GoogleNewsConsumer Resource]
    end

    subgraph "External Services"
        AzureQueue[Azure Storage Queue]
        GNewsRSS[Google News RSS API]
    end

    subgraph "Database"
        DB_Keywords[(credit_subsidiary_keywords)]
        DB_News[(credit_google_news)]
    end

    GN -->|1. Query Keywords| DB_Keywords
    GN -->|2. Enqueue Tasks| AzureQueue
    GNC -->|3. Dequeue Tasks| AzureQueue
    GNC -->|4. Fetch Feed| GNewsRSS
    GNC -->|5. Filter & Hash| GNC
    GNC -->|6. Batch Insert| DB_News
    GNC -->|7. Update Sync Date| DB_Keywords
```

## Core Components

### GoogleNews (`resource/news/google_news.py`)
The producer component. It acts as a trigger to start the news collection process.

*   **Functionality:**
    *   Queries the `credit_subsidiary_keywords` table for entities requiring a news update (typically those not synced within the last 31 days).
    *   Serializes keyword metadata (ID, Parent ID, Keyword string) into JSON messages.
    *   Dispatches messages to the Azure Storage Queue defined by `CREDIT_NEWS_QUEUE`.
*   **Security:** Validates requests using a signature-based authentication (`gen_sign`).

### GoogleNewsConsumer (`resource/news/consumer.py`)
The consumer component. It performs the heavy lifting of data retrieval and processing.

*   **Functionality:**
    *   **Queue Management:** Retrieves messages from both standard and reassess queues using a multi-threaded approach (`ThreadPoolExecutor`).
    *   **Feed Fetching:** Uses `feedparser` to retrieve RSS content from Google News based on entity keywords.
    *   **Risk Filtering:** Integrates with `util.news.check_alert` to determine if a news title contains risk-relevant keywords before processing.
    *   **Deduplication:** Generates an MD5 hash of the news title to prevent duplicate entries in the database.
    *   **Batch Persistence:** Efficiently inserts news records into `credit_google_news` using batch SQL operations.

## Data Flow

```mermaid
sequenceDiagram
    participant Cron as Scheduler/Trigger
    participant GN as GoogleNews
    participant Queue as Azure Queue
    participant GNC as GoogleNewsConsumer
    participant RSS as Google News RSS
    participant DB as Database

    Cron->>GN: GET /google_news?sign=...
    GN->>DB: Fetch keywords (last_sync_date < 31 days)
    DB-->>GN: List of keywords
    loop for each keyword
        GN->>Queue: Send Message {id, keyword, parent_id}
    end
    GN-->>Cron: 200 OK

    Cron->>GNC: GET /consumer?sign=...
    GNC->>Queue: Receive Messages (Batch)
    loop for each message
        GNC->>RSS: Fetch RSS Feed for Keyword
        RSS-->>GNC: XML Feed Entries
        GNC->>GNC: Filter via check_alert()
        GNC->>GNC: Generate MD5 Hash
        GNC->>DB: Check for existing Hash/ID combo
        GNC->>DB: Batch Insert New Records
        GNC->>DB: Update last_sync_date for Keyword
    end
    GNC-->>Cron: 200 OK
```

## Integration with Other Modules

*   **[Entity Management](Entity_Management.md):** Provides the keywords and hierarchy (Parent/Subsidiary relationships) that drive the news search.
*   **[News Intelligence](News_Intelligence.md):** The broader module containing this processing logic, also handling event categorization and manual pinning.
*   **[AI Engine Models](AI_Engine_Models.md):** Consumes the data stored in `credit_google_news` to perform sentiment analysis and risk scoring via `CreditNewsAI`.

## Configuration

The module requires the following environment variables:
*   `CREDIT_AZURE_STORAGE_CONNECTION_STRING`: Connection string for Azure Storage account.
*   `CREDIT_NEWS_QUEUE`: Primary queue name for news tasks.
*   `CREDIT_NEWS_REASSESS_QUEUE`: Queue for high-priority or re-processing tasks.
