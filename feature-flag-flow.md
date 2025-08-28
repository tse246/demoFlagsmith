# Angular Feature Flag Flow - UML Diagram

## System Architecture Overview

```mermaid
graph TB
    subgraph "Angular Application"
        AC[AppComponent]
        AFM[AbstractFeatureManager]
        FFM[FlagsmithFeatureManager]
        FSM[FeatureStateManager]
        WS[WebhookService]
        CS[ConfigService]
        FS[FlagsmithService - Legacy]
    end
    
    subgraph "Configuration"
        FC[FeatureFlagConfig]
        WC[WebhookConfig]
    end
    
    subgraph "External Services"
        FAPI[Flagsmith API]
        WH[Webhook Server]
    end
    
    AC --> AFM
    AFM --> FFM
    FFM --> FSM
    FFM --> WS
    FFM --> CS
    WS --> CS
    CS --> FC
    FC --> WC
    FFM --> FAPI
    WS --> WH
    FS --> FAPI
    FS --> WH
    FS --> CS
```

## Detailed Flow Sequence

```mermaid
sequenceDiagram
    participant UI as AppComponent
    participant AFM as AbstractFeatureManager
    participant FFM as FlagsmithFeatureManager
    participant FSM as FeatureStateManager
    participant WS as WebhookService
    participant CS as ConfigService
    participant FAPI as Flagsmith API
    participant WHS as Webhook Server
    
    Note over UI,WHS: Application Initialization
    UI->>+AFM: constructor injection
    AFM->>+FFM: initialize
    FFM->>+CS: getFeatureFlagConfig()
    CS-->>-FFM: config (environmentId, apiUrl, webhookConfig)
    
    FFM->>+FAPI: flagsmith.init(environmentId, apiUrl)
    FAPI-->>-FFM: initialization complete
    
    FFM->>+FSM: setInitialized(true)
    FSM-->>-FFM: state updated
    
    FFM->>+WS: setupWebhookHandling()
    WS->>+CS: getFeatureFlagConfig().webhookConfig
    CS-->>-WS: webhook configuration
    WS->>WS: startPolling()
    
    Note over UI,WHS: Feature Flag Retrieval
    UI->>+AFM: getFeatureObservable('feature1')
    AFM->>+FFM: getFeatureObservable('feature1')
    FFM->>+FSM: getFeatureObservable('feature1')
    FSM-->>-FFM: Observable<boolean>
    FFM-->>-AFM: Observable<boolean>
    AFM-->>-UI: Observable<boolean>
    
    Note over UI,WHS: Real-time Updates (Webhook Polling)
    loop Every 2 seconds
        WS->>+WHS: HTTP GET /api/features
        WHS-->>-WS: webhook data
        alt If webhook data contains event_type
            WS->>+FFM: onWebhookReceived(data)
            FFM->>+FAPI: flagsmith.getFlags()
            FAPI-->>-FFM: updated flags
            FFM->>+FSM: updateFeatureState(name, enabled)
            FSM->>FSM: BehaviorSubject.next(enabled)
            FSM-->>-FFM: state propagated
            FFM-->>-WS: refresh complete
        end
    end
    
    Note over UI,WHS: Manual Refresh
    UI->>+AFM: refreshFeatures()
    AFM->>+FFM: refreshFeatures()
    FFM->>+FAPI: flagsmith.getFlags()
    FAPI-->>-FFM: updated flags
    FFM->>+FSM: updateFeatureState(name, enabled)
    FSM-->>-FFM: state updated
    FFM-->>-AFM: refresh complete
    AFM-->>-UI: Promise resolved
```

## Class Relationship Diagram

```mermaid
classDiagram
    class AppComponent {
        +feature1Enabled$: Observable~boolean~
        +feature2Enabled$: Observable~boolean~
        +automaticSectionEnabled$: Observable~boolean~
        +refreshFeatures(): Promise~void~
        +refreshFeaturesManual(): Promise~void~
    }
    
    class AbstractFeatureManager {
        <<abstract>>
        +getFeature(name: string): boolean
        +getValue(name: string): any
        +refreshFeatures(): Promise~void~
        +getFeatureObservable(name: string): Observable~boolean~
        +isInitialized(): Observable~boolean~
    }
    
    class FlagsmithFeatureManager {
        -isInitialized: boolean
        +getFeature(name: string): boolean
        +getValue(name: string): any
        +refreshFeatures(): Promise~void~
        +getFeatureObservable(name: string): Observable~boolean~
        +isInitialized(): Observable~boolean~
        -initializeProvider(): Promise~void~
        -updateFeatureStates(): void
        -setupWebhookHandling(): void
    }
    
    class FeatureStateManager {
        -featureStates: Map~string, BehaviorSubject~boolean~~
        -initializedSubject: BehaviorSubject~boolean~
        +updateFeatureState(name: string, enabled: boolean): void
        +getFeatureObservable(name: string): Observable~boolean~
        +isInitialized(): Observable~boolean~
        +setInitialized(initialized: boolean): void
    }
    
    class WebhookService {
        -config: IWebhookConfig
        -pollingSubscription: Subscription
        -onWebhookReceived: Function
        +setWebhookCallback(callback: Function): void
        +startPolling(): void
        +stopPolling(): void
        -checkForUpdates(): void
    }
    
    class ConfigService {
        -environment: string
        +getFeatureFlagConfig(): IFeatureFlagConfig
        +getEnvironment(): string
        +setEnvironmentOverride(environment: string): void
    }
    
    class IFeatureFlagConfig {
        <<interface>>
        +provider: string
        +apiUrl: string
        +environmentId: string
        +webhookConfig?: IWebhookConfig
    }
    
    class IWebhookConfig {
        <<interface>>
        +url: string
        +pollInterval: number
        +enabled: boolean
    }
    
    AppComponent --> AbstractFeatureManager
    AbstractFeatureManager <|-- FlagsmithFeatureManager
    FlagsmithFeatureManager --> FeatureStateManager
    FlagsmithFeatureManager --> WebhookService
    FlagsmithFeatureManager --> ConfigService
    WebhookService --> ConfigService
    ConfigService --> IFeatureFlagConfig
    IFeatureFlagConfig --> IWebhookConfig
```

## Data Flow Architecture

```mermaid
graph LR
    subgraph "Configuration Layer"
        ENV[Environment Detection]
        CONFIG[Feature Flag Config]
        WEBHOOK[Webhook Config]
    end
    
    subgraph "Service Layer"
        CS[ConfigService]
        FFM[FlagsmithFeatureManager]
        FSM[FeatureStateManager]
        WS[WebhookService]
    end
    
    subgraph "Presentation Layer"
        AC[AppComponent]
        TEMPLATE[HTML Template]
    end
    
    subgraph "External Integration"
        FAPI[Flagsmith API]
        WHS[Local Webhook Server]
    end
    
    ENV --> CS
    CONFIG --> CS
    WEBHOOK --> CS
    
    CS --> FFM
    CS --> WS
    
    FFM --> FSM
    FFM --> FAPI
    WS --> WHS
    
    FSM --> AC
    FFM --> AC
    
    AC --> TEMPLATE
    
    WHS -.->|Polling| WS
    WS -.->|Callback| FFM
    FFM -.->|Update| FSM
    FSM -.->|Observable| AC
```

## Key Features

### 1. **SOLID Principles Implementation**
- **Single Responsibility**: Each service has one clear purpose
- **Open/Closed**: Abstract base class allows for multiple providers
- **Liskov Substitution**: FlagsmithFeatureManager can replace AbstractFeatureManager
- **Interface Segregation**: Separate interfaces for different concerns
- **Dependency Inversion**: Services depend on abstractions, not concretions

### 2. **Real-time Updates**
- Webhook polling every 2 seconds (configurable)
- Automatic feature flag synchronization
- Reactive UI updates using RxJS Observables

### 3. **Configuration Management**
- Environment-based configuration
- Centralized settings in `feature-flag.config.ts`
- No hardcoded values in services

### 4. **State Management**
- BehaviorSubjects for reactive state
- Observable pattern for UI updates
- Centralized feature state management

## Usage Flow

1. **Initialization**: ConfigService determines environment and loads appropriate configuration
2. **Service Setup**: FlagsmithFeatureManager initializes with Flagsmith API using config
3. **Webhook Setup**: WebhookService starts polling local webhook server
4. **UI Binding**: AppComponent subscribes to feature flag observables
5. **Real-time Updates**: Webhook polling detects changes and triggers refresh
6. **State Propagation**: Updated flags flow through observables to UI components