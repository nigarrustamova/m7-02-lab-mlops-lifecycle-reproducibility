# NorthStar Logistics - ETA Model Lifecycle

```mermaid
graph TD
    %% Data & Preparation
    subgraph Data Layer
        A[Data Source] -- "Raw Logs" --> B(Data Preprocessing)
        B -- "Dataset Hash / Feature Store URI" --> C{Training Trigger}
    end

    %% Training & Evaluation
    subgraph Experimentation & Training
        C -- "Automatic Trigger (Cron/Data Change)" --> D(Model Training)
        D -- "Run ID / Model Weights" --> E(Offline Evaluation)
        E -- "Evaluation Metrics Report" --> F{Gate: Meets Staging Threshold?}
    end

    %% Registry - Staging
    subgraph Model Registry
        F -- "Yes (Automatic)" --> G[[Stage: Staging]]
        G -- "Model URI (Staging)" --> H(Integration & Canary Testing)
        
        %% Registry - Production
        H -- "Testing Success / Manual Approval" --> I[[Stage: Production]]
        
        %% Registry - Archived
        I -- "Newer Version Promoted" --> J[[Stage: Archived]]
        G -- "Failed Testing" --> J
    end

    %% Deployment & Monitoring
    subgraph Serving Layer
        I -- "Deployed Version / Docker Image Tag" --> K(Online Inference)
        K -- "Prediction Logs" --> L(Monitoring System)
    end

    %% Loopback
    L -- "Drift Signal / Performance Degradation" --> C
    L -- "Ground Truth Comparison" --> B

    %% Annotations
    classDef automatic fill:#e1f5fe,stroke:#01579b;
    classDef manual fill:#fff3e0,stroke:#e65100;
    classDef stage fill:#f3e5f5,stroke:#4a148c;

    class G,I,J stage;
    class F automatic;
    class H manual;
```

## Description of Transitions and Artifacts

| Transition | Type | Artifact(s) |
| :--- | :--- | :--- |
| Data Preprocessing -> Training | Automatic | **Dataset Hash**, **Feature Store URI** |
| Model Training -> Evaluation | Automatic | **Run ID**, **Model Weights (Artifact URI)** |
| Evaluation -> Staging | **Automatic** | **Evaluation Metrics Report** |
| Staging -> Production | **Manual (Data Scientist / ML Lead)** | **Model URI (Staging)**, **Canary Test Results** |
| Production -> Serving | Automatic | **Docker Image Tag**, **Deployed Version ID** |
| Monitoring -> Training | Automatic | **Drift Signal (KS Test P-value < 0.05)** |
| Monitoring -> Data | Automatic | **Prediction Error / Ground Truth (Feedback Loop)** |
