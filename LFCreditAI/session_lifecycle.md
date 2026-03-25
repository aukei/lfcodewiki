# Session Lifecycle Documentation

## Introduction
The `session_lifecycle` module is a critical component of the **Authentication_Access** system. It manages the end-to-end lifecycle of a user session, from validating access rights during active requests to securely terminating sessions. It ensures that only authenticated and authorized users can interact with the system's resources, such as [Credit_Report_Service](credit_report_service.md) and [Potential_Analysis](potential_analysis.md).

## Architecture and Component Relationships

The module operates as a middleware layer between the incoming requests and the application's business logic. It relies heavily on the `AzureAuthenticator` for identity verification and the `PermissionService` for route-based access control.

### Core Components

| Component | Path | Responsibility |
|:---|:---|:---|
| **AccessManager** | `services/user/access_manager.py` | Intercepts requests to validate login status, whitelist membership, and role-based route permissions. |
| **LogoutAPI** | `resource/logout_api.py` | Handles session termination by clearing session data and invalidating authentication cookies. |

### Dependency Diagram

```mermaid
graph TD
    subgraph Authentication_Access
        AL[LogoutAPI]
        AM[AccessManager]
        AA[AzureAuthenticator]
    end

    subgraph External_Services
        AZ[Azure AD / App Service Auth]
    end

    subgraph Session_Storage
        FS[Flask Session]
    end

    AM --> AA
    AM --> FS
    AL --> FS
    AL -.-> AZ
```

## Data Flow and Process Logic

### 1. Access Validation Flow
The `AccessManager` performs a multi-stage check for every request (excluding public paths).

```mermaid
sequenceDiagram
    participant U as User/Client
    participant AM as AccessManager
    participant AA as AzureAuthenticator
    participant S as Session

    U->>AM: Request Path
    AM->>AM: Check NO_LOGIN_REQUIRED_PATHS
    alt Is Public Path
        AM->>U: Allow Access
    else Requires Auth
        AM->>AA: is_login()
        alt Not Logged In
            AA-->>AM: False
            AM->>U: 401 Unauthorized / Redirect to Login
        else Logged In
            AM->>AA: is_whitelisted()
            alt Not Whitelisted
                AA-->>AM: False
                AM->>U: 403 Forbidden
            else Authorized
                AM->>S: Get allowed_routes
                alt Routes Missing
                    S-->>AM: None
                    AM->>AM: Fetch & Cache Routes
                end
                AM->>U: Allow Access (if path matches)
            end
        end
    end
```

### 2. Session Termination (Logout)
The `LogoutAPI` ensures that all sensitive user information is purged from the server-side session and the client-side cookie.

**Cleared Session Keys:**
*   `user_email`, `user_id`, `user_name`
*   `role`, `role_id`, `permissions`
*   `user_filters`, `allowed_routes`

**Cookie Invalidation:**
*   Sets `AppServiceAuthSession` to an empty string with `max_age=0`.

## Integration with Other Modules

*   **[Authentication_Access](authentication_access.md):** Uses `AzureAuthenticator` to verify the underlying identity token provided by Azure AD.
*   **[Frontend_Core](frontend_core.md):** The frontend reacts to the 401/403 status codes returned by `AccessManager` to trigger login modals or permission alerts.
*   **System-wide:** Every module (e.g., [News_Intelligence](news_intelligence.md), [Entity_Management](entity_management.md)) is protected by the `AccessManager` middleware, ensuring that data leakage is prevented at the routing level.

## Configuration
The `AccessManager` behavior is influenced by:
*   `NO_LOGIN_REQUIRED_PATHS`: A list of paths that bypass authentication.
*   Session-stored `role`: Users with `admin` or `credit` roles bypass specific route checks.
