| Author        | your name   |
| ------------- | ----------- |
| Created       | Jan 1, 1970 |
| Last modified | Jan 1, 1970 |

### Technical vision

- the main goal of this project.
- Set of guidelines and principles for the team or entire organization:
  - **Testing Standards**: Unit test coverage minimum 80%, integration tests for critical user flows, E2E tests for main business scenarios, performance testing for pages with >10k users
  - **Code Architecture**: Feature-Sliced Design methodology for large applications, component composition over inheritance, dependency injection for testability
  - **State Management**: Redux toolkit for global state with clear domain boundaries, local state for UI-only concerns, immutable updates, normalized data structures
  - **Development Practices**: Code reviews mandatory, pair programming for complex features, trunk-based development with feature flags
  - **API Design**: RESTful conventions, consistent error handling, pagination for list endpoints, versioning strategy
  - **Security**: Authentication tokens in httpOnly cookies, input validation on both client/server, HTTPS everywhere, CSP headers
  - **Performance**: Bundle size <500KB initial load, lazy loading for non-critical routes, image optimization, caching strategy
  - **Accessibility**: WCAG 2.1 AA compliance, keyboard navigation, screen reader support, semantic HTML
- Summary of decisions being made
- Definition of the processes (e.g., on-call rotations, addressing technical debt, etc)

### Technical strategy

- details the concrete steps and challenges required to get there.

### Motivation

- should provide a brief overview of the problem that the feature or project is aiming to solve. It should focus on the “what” rather than the “how”. It is important to include enough information in this section for the readers to evaluate the feasibility of the proposed solution.

### Design proposal

- outlines the approach and methodology for achieving the goals mentioned in the motivation section.

### Terminology

- should include any special terms or concepts that are specific to the project and may not be familiar to everyone on the team.

### High level design

- project milestones (e.g., “Build JSON policy editor widget”, “Build cross-region copy flow”, etc)
- for each milestone, define a high-level component tree with state-related concerns (if you are using a state management system like Redux, it might be important to highlight what data goes to the global state vs. what data is isolated in the component’s local state).
- If you are using distributed frontend architecture (micro-frontends), you may apply Domain-Driven Design methodology to draw new boundaries in the architecture. In other words, whether this new feature fits into existing subdomain/bounded context or requires a brand new micro-frontend app to be developed and deployed. If the latter, then how it will communicate with other apps, and what events will represent these communication concerns?
- API integration and permission model: what permissions are required to call specific API, and what would be the UX behaviour in case of lack of permissions (e.g., redirect to page B)
- Known limitations and uncertainties (e.g. “We won’t be able to use ModalA component because it requires admin account permissions to build the view. We will have to wait for Auth team to release delegated admins type of account” or “If we decide to use the same advanced JSON editor as we on PageA, we need to investigate how we can manage shared third-party dependencies in the future”). This kind of considerations can be really useful for engineer who will be working closely on the low-level design for this milestone.
- Define business/operational metrics to track
- Rough estimates for each milestone (preferably, build a workstream diagram that will highlight dependencies between milestones, order of execution and level parallelization)

### Low level designed

- Enhanced component tree with more fine-grained components and well-defined state shapes and component contracts

```mermaid
graph TD
    A[App Component] --> B[Header Component]
    A --> C[Main Container]
    A --> D[Footer Component]
    
    C --> E[Sidebar Component]
    C --> F[Content Area]
    
    E --> G[Navigation Menu]
    E --> H[User Profile Widget]
    
    F --> I[Data Table Component]
    F --> J[Action Panel]
    
    I --> K[Table Header]
    I --> L[Table Body]
    I --> M[Pagination]
    
    J --> N[Create Button]
    J --> O[Filter Controls]
    J --> P[Export Options]
    
    style A fill:#e1f5fe
    style C fill:#f3e5f5
    style F fill:#e8f5e8
```

- If your app requires complex API orchestration, you may benefit from having a detailed flow diagram to highlight the order of API execution and when to update components with new state

```mermaid
sequenceDiagram
    participant User
    participant UI as UI Component
    participant Store as State Store
    participant API as API Service
    participant Cache
    
    User->>UI: Initiates action
    UI->>Store: Dispatch loading state
    Store->>UI: Update with loading indicator
    
    UI->>Cache: Check for cached data
    alt Cache hit
        Cache->>UI: Return cached data
        UI->>Store: Update with cached data
    else Cache miss
        UI->>API: Fetch user data
        API->>UI: Return user response
        UI->>Cache: Store data in cache
        
        UI->>API: Fetch user permissions
        API->>UI: Return permissions response
        
        UI->>API: Fetch user preferences
        API->>UI: Return preferences response
        
        UI->>Store: Update with complete data
    end
    
    Store->>UI: Render final state
    UI->>User: Display updated interface
```

- Error handling for special use cases

```mermaid
flowchart TD
    A[User Action] --> B{Validate Input}
    B -->|Valid| C[Process Request]
    B -->|Invalid| D[Show Validation Error]
    
    C --> E{API Call}
    E -->|Success| F[Update State]
    E -->|Network Error| G[Show Network Error]
    E -->|Auth Error| H[Redirect to Login]
    E -->|Server Error| I[Show Generic Error]
    
    F --> J[Render Success State]
    
    G --> K[Retry Option]
    I --> L[Error Recovery Options]
    
    K --> C
    L --> M[Contact Support]
    L --> N[Refresh Page]
    
    style D fill:#ffebee
    style G fill:#ffebee
    style H fill:#fff3e0
    style I fill:#ffebee
```

- Performance concerns, such as caching and network optimization

```mermaid
graph TB
    subgraph "Performance Optimization"
        A[Browser Cache] --> B[Service Worker Cache]
        B --> C[Memory Cache]
        C --> D[API Response]
        
        E[Code Splitting] --> F[Lazy Loading]
        F --> G[Component Bundling]
        
        H[Image Optimization] --> I[WebP Format]
        H --> J[Responsive Images]
        H --> K[Image Lazy Loading]
        
        L[Network Optimization] --> M[HTTP/2]
        L --> N[Compression]
        L --> O[CDN Distribution]
    end
    
    style A fill:#e3f2fd
    style E fill:#f1f8e9
    style H fill:#fef7ff
    style L fill:#fff8e1
```

- Security concerns, such as permission schemas to be publicly shared, storing sensitive information in the local storage/IndexDB, CSRF protection, CSP headers

```mermaid
flowchart LR
    subgraph "Security Layers"
        A[User Authentication] --> B[Permission Validation]
        B --> C[CSRF Protection]
        C --> D[Data Encryption]
        
        E[CSP Headers] --> F[XSS Prevention]
        F --> G[Content Validation]
        
        H[Secure Storage] --> I[Token Management]
        I --> J[Session Handling]
    end
    
    subgraph "Data Flow Security"
        K[Client Request] --> L{Auth Check}
        L -->|Valid| M[Permission Check]
        L -->|Invalid| N[Block Request]
        
        M --> O{Resource Access}
        O -->|Allowed| P[Proceed]
        O -->|Denied| Q[Access Denied]
    end
    
    style A fill:#e8f5e8
    style E fill:#e3f2fd
    style H fill:#fef7ff
    style N fill:#ffebee
    style Q fill:#ffebee
```

- Alternative design considerations
- Technologies and tools to be used
- Test scenarios (usually speaking about E2E tests here)

```mermaid
graph TD
    subgraph "E2E Test Scenarios"
        A[User Registration] --> B[Email Verification]
        B --> C[Profile Setup]
        C --> D[Dashboard Access]
        
        E[Login Flow] --> F[Two-Factor Auth]
        F --> G[Dashboard Navigation]
        
        H[Data CRUD Operations] --> I[Create Record]
        H --> J[Read/Search Records]
        H --> K[Update Record]
        H --> L[Delete Record]
        
        M[Error Scenarios] --> N[Network Failure]
        M --> O[Invalid Data]
        M --> P[Permission Errors]
        
        Q[Performance Tests] --> R[Page Load Time]
        Q --> S[API Response Time]
        Q --> T[Memory Usage]
    end
    
    style A fill:#e8f5e8
    style E fill:#e3f2fd
    style H fill:#fff8e1
    style M fill:#ffebee
    style Q fill:#f3e5f5
```

- Accurate estimates of work and level of parallelization

```mermaid
gantt
    title Frontend Development Timeline
    dateFormat  YYYY-MM-DD
    section Setup Phase
    Environment Setup    :setup, 2024-01-01, 3d
    Architecture Design  :arch, after setup, 5d
    
    section Development
    Component Library    :comp, 2024-01-09, 10d
    API Integration     :api, after arch, 8d
    State Management    :state, after comp, 6d
    
    section Testing
    Unit Tests          :unit, after comp, 5d
    Integration Tests   :integration, after api, 7d
    E2E Tests          :e2e, after state, 5d
    
    section Deployment
    Build Optimization  :build, after e2e, 3d
    Production Deploy   :deploy, after build, 2d
```

### Risk Considerations

- should address any potential risks associated with the proposed design. This includes having a contingency plan in case the original design does not work, as well as identifying any potential side effects that could impact the existing architecture (e.g. performance, usability concerns)

### Security Considerations

- should address how the proposed solution will handle sensitive data, permissions policies, and protection against both insider and external threats.

### Operations

- The operations section of the design document should address how the system will be operated once it is deployed. This includes business metrics, such as who will use the feature and how it will be used, as well as system health monitoring, such as ensuring that page loads successfully, tracking the latency of user actions, and monitoring web vital metrics.

### Testing Scenario

- should cover all aspects of the business flow, including input validation, error handling, and edge cases.

### Appendices

- typically used to provide additional information that may not be directly relevant to the core design. Examples of information that can be included in the appendix section are non-essential architecture diagrams, UX mockups, code snippets (can be even pseudo-code), links to tools documentation, and other related design documents.
