# Code Analysis Pipeline - Diagrams

## 1. System Use Case Diagram

```mermaid
---
config:
  theme: base
---
graph LR
    DEV((Developer))
    ADMIN((System<br/>Administrator))
    
    subgraph System["Code Analysis Pipeline System"]
        direction TB
        UC1([Upload Code Files])
        UC2([View File Summaries])
        UC3([View Function<br/>Explanations])
        UC4([View Project Flow])
        UC5([View Q&A Pairs])
        UC6([Download Reports])
        UC7([Configure System])
        UC8([Monitor Performance])
        UC9([Manage API Keys])
        
        VAL([Validate Files])
        PARSE([Parse Code])
    end
    
    %% Developer use cases
    DEV ==> UC1
    DEV ==> UC2
    DEV ==> UC3
    DEV ==> UC4
    DEV ==> UC5
    DEV ==> UC6
    
    %% Administrator use cases
    ADMIN ==> UC7
    ADMIN ==> UC8
    ADMIN ==> UC9
    
    %% Include relationships
    UC1 -.includes.-> VAL
    UC1 -.includes.-> PARSE
    
    %% Extend/Dependency relationships
    UC2 -.-|extends| UC1
    UC3 -.-|extends| UC1
    UC4 -.-|extends| UC1
    UC5 -.-|extends| UC1
    
    UC6 -.-|extends| UC2
    UC6 -.-|extends| UC3
    UC6 -.-|extends| UC4
    UC6 -.-|extends| UC5
    
    style DEV fill:#FFE4B5,stroke:#333,stroke-width:3px
    style ADMIN fill:#FFE4B5,stroke:#333,stroke-width:3px
    style UC1 fill:#87CEEB,stroke:#333,stroke-width:2px
    style UC2 fill:#87CEEB,stroke:#333,stroke-width:2px
    style UC3 fill:#87CEEB,stroke:#333,stroke-width:2px
    style UC4 fill:#87CEEB,stroke:#333,stroke-width:2px
    style UC5 fill:#87CEEB,stroke:#333,stroke-width:2px
    style UC6 fill:#90EE90,stroke:#333,stroke-width:2px
    style UC7 fill:#DDA0DD,stroke:#333,stroke-width:2px
    style UC8 fill:#DDA0DD,stroke:#333,stroke-width:2px
    style UC9 fill:#DDA0DD,stroke:#333,stroke-width:2px
    style VAL fill:#F0E68C,stroke:#333,stroke-width:1px
    style PARSE fill:#F0E68C,stroke:#333,stroke-width:1px
```

## 2. High-Level Process Flow Diagram

```mermaid
---
config:
  theme: base
---
flowchart TD
    START([User Accesses Dashboard]) --> UPLOAD[Upload Code Files]
    UPLOAD --> VALIDATE{Validate Files}
    
    VALIDATE -->|Invalid| ERROR1[Display Error Message]
    ERROR1 --> UPLOAD
    
    VALIDATE -->|Valid| PARSE[Parse Code Files]
    PARSE --> CHECK_PARSE{Parsing<br/>Successful?}
    
    CHECK_PARSE -->|Partial Success| WARNING[Log Warnings]
    CHECK_PARSE -->|Success| PIPELINE[DSPy Pipeline Orchestrator]
    WARNING --> PIPELINE
    
    CHECK_PARSE -->|Total Failure| ERROR2[Display Parse Errors]
    ERROR2 --> END1([End])
    
    PIPELINE --> PARALLEL{Parallel Processing}
    
    PARALLEL --> FS[File Summarizer<br/>Module]
    PARALLEL --> FE[Function Explainer<br/>Module]
    PARALLEL --> PFG[Project Flow<br/>Generator Module]
    
    FS --> SYNC1{All Modules<br/>Complete?}
    FE --> SYNC1
    PFG --> SYNC1
    
    SYNC1 --> QA[Q&A Generator<br/>Module]
    QA --> OPTIMIZE[Output Optimizer]
    
    OPTIMIZE --> FORMAT[Format Results]
    FORMAT --> DISPLAY[Display Results<br/>in Dashboard]
    
    DISPLAY --> CHOICE{User Action}
    CHOICE -->|Download| EXPORT[Export Reports]
    CHOICE -->|Review| VIEW[View Details]
    CHOICE -->|Upload New| UPLOAD
    CHOICE -->|Exit| END2([End])
    
    EXPORT --> END2
    VIEW --> CHOICE
    
    style START fill:#90EE90
    style END1 fill:#FFB6C1
    style END2 fill:#90EE90
    style PIPELINE fill:#87CEEB
    style FS fill:#FFE4B5
    style FE fill:#FFE4B5
    style PFG fill:#FFE4B5
    style QA fill:#FFE4B5
```

## 3. Detailed DSPy Pipeline Process Flow

```mermaid
---
config:
  theme: base
---
flowchart TD
    INPUT[Parsed Code Files] --> ORCHESTRATOR[Pipeline Orchestrator]
    
    ORCHESTRATOR --> INIT[Initialize DSPy<br/>with Nvidia API]
    INIT --> CONFIG{DSPy<br/>Configured?}
    
    CONFIG -->|No| FALLBACK[Use Fallback<br/>Analysis]
    CONFIG -->|Yes| PARALLEL[Start Parallel<br/>Processing]
    
    PARALLEL --> MODULE1[File Summarizer]
    PARALLEL --> MODULE2[Function Explainer]
    PARALLEL --> MODULE3[Project Flow Generator]
    
    subgraph "File Summarizer Flow"
        MODULE1 --> FS1[Extract File Context]
        FS1 --> FS2[Generate Summary<br/>via DSPy Chain]
        FS2 --> FS3{Valid<br/>Summary?}
        FS3 -->|Yes| FS4[Store Result]
        FS3 -->|No| FS5[Retry/Fallback]
        FS5 --> FS4
    end
    
    subgraph "Function Explainer Flow"
        MODULE2 --> FE1[Identify Functions]
        FE1 --> FE2[For Each Function:<br/>Explain via DSPy]
        FE2 --> FE3{All Functions<br/>Explained?}
        FE3 -->|No| FE2
        FE3 -->|Yes| FE4[Store Results]
    end
    
    subgraph "Project Flow Flow"
        MODULE3 --> PF1[Build Dependency<br/>Graph]
        PF1 --> PF2[Detect Circular<br/>Dependencies]
        PF2 --> PF3[Generate Flow via<br/>DSPy Chain]
        PF3 --> PF4[Enhance with<br/>Deterministic Analysis]
        PF4 --> PF5[Store Result]
    end
    
    FS4 --> WAIT[Wait for All<br/>Modules]
    FE4 --> WAIT
    PF5 --> WAIT
    FALLBACK --> WAIT
    
    WAIT --> AGGREGATE[Aggregate Results]
    AGGREGATE --> QA_MOD[Q&A Generator]
    
    QA_MOD --> QA1[Generate Questions<br/>from Analysis]
    QA1 --> QA2[Classify by<br/>Difficulty]
    QA2 --> QA3[Store Q&A Pairs]
    
    QA3 --> OUTPUT[Complete Analysis<br/>Results]
    
    style INIT fill:#87CEEB
    style PARALLEL fill:#90EE90
    style WAIT fill:#FFD700
    style OUTPUT fill:#90EE90
```

## 4. File Upload and Validation Flow

```mermaid
---
config:
  theme: base
---
flowchart TD
    START([User Selects Files]) --> UPLOAD[Upload Files<br/>to Interface]
    
    UPLOAD --> COUNT{File Count<br/>≤ 50?}
    COUNT -->|No| ERR1[Error: Max 50 files]
    ERR1 --> END1([End])
    
    COUNT -->|Yes| LOOP[For Each File]
    
    LOOP --> EXT{Valid<br/>Extension?}
    EXT -->|No| INVALID[Mark as Invalid]
    
    EXT -->|Yes| SIZE{Size<br/>≤ 10MB?}
    SIZE -->|No| INVALID
    
    SIZE -->|Yes| MIME[Validate MIME Type]
    MIME --> SECURITY[Security Scan]
    SECURITY --> ENCODING[Detect Encoding]
    ENCODING --> META[Generate Metadata]
    META --> VALID[Mark as Valid]
    
    VALID --> MORE{More Files?}
    INVALID --> MORE
    
    MORE -->|Yes| LOOP
    MORE -->|No| SUMMARY{Any Valid<br/>Files?}
    
    SUMMARY -->|No| ERR2[Error: No Valid Files]
    ERR2 --> END1
    
    SUMMARY -->|Yes| CONFIRM[Display Confirmation<br/>with File List]
    CONFIRM --> PROCEED{User<br/>Proceeds?}
    
    PROCEED -->|No| CANCEL([Cancelled])
    PROCEED -->|Yes| PARSE[Send to Parser]
    
    style START fill:#90EE90
    style END1 fill:#FFB6C1
    style CANCEL fill:#FFE4B5
    style PARSE fill:#87CEEB
    style VALID fill:#90EE90
    style INVALID fill:#FFB6C1
```

## 5. Error Handling Flow

```mermaid
---
config:
  theme: base
---
flowchart TD
    ERROR[Error Detected] --> TYPE{Error Type}
    
    TYPE -->|Upload Error| UE[Handle Upload Error]
    TYPE -->|Parse Error| PE[Handle Parse Error]
    TYPE -->|DSPy Error| DE[Handle DSPy Error]
    TYPE -->|System Error| SE[Handle System Error]
    
    UE --> UE1[Log Error Details]
    UE1 --> UE2[Display User Message:<br/>Invalid file/size/network]
    UE2 --> UE3[Allow Retry]
    
    PE --> PE1[Log Parse Error]
    PE1 --> PE2{Other Files<br/>Available?}
    PE2 -->|Yes| PE3[Continue with<br/>Other Files]
    PE2 -->|No| PE4[Display Error<br/>& Abort]
    PE3 --> PE5[Mark File with<br/>Warning]
    
    DE --> DE1[Log Component Error]
    DE1 --> DE2{Fallback<br/>Available?}
    DE2 -->|Yes| DE3[Use Fallback<br/>Analysis]
    DE2 -->|No| DE4[Skip Component]
    DE3 --> DE5[Continue Pipeline]
    DE4 --> DE5
    
    SE --> SE1[Save User Data]
    SE1 --> SE2[Log System Error]
    SE2 --> SE3{Critical<br/>Error?}
    SE3 -->|Yes| SE4[Graceful Shutdown<br/>with Recovery]
    SE3 -->|No| SE5[Queue Request<br/>& Notify User]
    
    UE3 --> RECOVERY[Recovery Point]
    PE5 --> RECOVERY
    DE5 --> RECOVERY
    SE4 --> RECOVERY
    SE5 --> RECOVERY
    
    RECOVERY --> NOTIFY[Notify User of<br/>Status]
    
    style ERROR fill:#FFB6C1
    style RECOVERY fill:#90EE90
    style NOTIFY fill:#87CEEB
```

## 6. Data Flow Diagram (Level 0 - Context Diagram)

```mermaid
---
config:
  theme: base
---
flowchart LR
    DEV[Developer] -->|Upload Code Files| SYSTEM[Code Analysis<br/>Pipeline System]
    SYSTEM -->|Analysis Results| DEV
    
    SYSTEM <-->|API Calls| NVIDIA[Nvidia API<br/>Language Models]
    
    ADMIN[Administrator] -->|Configuration| SYSTEM
    SYSTEM -->|Logs & Metrics| ADMIN
    
    SYSTEM -->|Store/Retrieve| CACHE[(Cache<br/>Storage)]
    SYSTEM -->|Save Results| STORAGE[(Results<br/>Storage)]
    
    style SYSTEM fill:#87CEEB
    style NVIDIA fill:#FFE4B5
    style CACHE fill:#D3D3D3
    style STORAGE fill:#D3D3D3
```

## 7. Data Flow Diagram (Level 1 - System Decomposition)

```mermaid
---
config:
  theme: base
---
flowchart TB
    USER[User] -->|Code Files| P1[1.0<br/>File Upload &<br/>Validation]
    P1 -->|Validated Files| P2[2.0<br/>Code Parser]
    P2 -->|Parsed Code| P3[3.0<br/>DSPy Analysis<br/>Pipeline]
    
    P3 <-->|LM Requests/Responses| NVIDIA[Nvidia API]
    
    P3 -->|Analysis Results| P4[4.0<br/>Output<br/>Optimizer]
    P4 -->|Formatted Results| USER
    
    P1 -->|Upload Errors| ERROR[(Error Logs)]
    P2 -->|Parse Errors| ERROR
    P3 -->|Analysis Errors| ERROR
    
    P2 -->|File Metadata| CACHE[(Cache)]
    CACHE -->|Cached Results| P3
    
    P4 -->|Reports| STORAGE[(Results<br/>Storage)]
    
    style P1 fill:#FFE4B5
    style P2 fill:#FFE4B5
    style P3 fill:#87CEEB
    style P4 fill:#FFE4B5
    style NVIDIA fill:#90EE90
```

## 8. Sequence Diagram - Complete Analysis Flow

```mermaid
---
config:
  theme: base
---
sequenceDiagram
    actor User
    participant UI as Streamlit Dashboard
    participant FH as File Handler
    participant Parser as Code Parser
    participant Orch as Pipeline Orchestrator
    participant FS as File Summarizer
    participant FE as Function Explainer
    participant PFG as Project Flow Gen
    participant QA as Q&A Generator
    participant LM as Nvidia API
    participant Opt as Output Optimizer
    
    User->>UI: Upload Files
    UI->>FH: Validate Files
    FH->>UI: Validation Result
    
    alt Invalid Files
        UI->>User: Display Errors
    else Valid Files
        UI->>Parser: Parse Files
        Parser->>Parser: Extract Structure
        Parser->>UI: Parsed Code Models
        
        UI->>Orch: Start Analysis
        
        par Parallel Processing
            Orch->>FS: Summarize Files
            FS->>LM: Request Summary
            LM-->>FS: Summary Response
            FS->>Orch: File Summaries
        and
            Orch->>FE: Explain Functions
            loop For Each Function
                FE->>LM: Request Explanation
                LM-->>FE: Explanation Response
            end
            FE->>Orch: Function Explanations
        and
            Orch->>PFG: Generate Flow
            PFG->>PFG: Build Dep Graph
            PFG->>LM: Request Flow Analysis
            LM-->>PFG: Flow Response
            PFG->>Orch: Project Flow
        end
        
        Orch->>QA: Generate Q&A
        QA->>LM: Request Questions
        LM-->>QA: Q&A Pairs
        QA->>Orch: Q&A Results
        
        Orch->>Opt: Optimize Output
        Opt->>Opt: Format Results
        Opt->>UI: Formatted Results
        
        UI->>User: Display Results
        
        User->>UI: Download Report
        UI->>Opt: Generate Export
        Opt->>User: Download File
    end
```

## 9. State Diagram - Analysis Process States

```mermaid
---
config:
  theme: base
---
stateDiagram-v2
    [*] --> Idle
    
    Idle --> Uploading: User Uploads Files
    Uploading --> Validating: Upload Complete
    
    Validating --> UploadError: Validation Failed
    UploadError --> Idle: User Retries
    
    Validating --> Parsing: Validation Passed
    Parsing --> ParsingError: Parse Failed
    ParsingError --> Idle: User Cancels
    ParsingError --> Parsing: Retry
    
    Parsing --> Analyzing: Parse Complete
    Analyzing --> FileSummarizing: Start Pipeline
    
    FileSummarizing --> FunctionExplaining: Complete
    FunctionExplaining --> ProjectFlowGen: Complete
    ProjectFlowGen --> QAGenerating: Complete
    
    QAGenerating --> Optimizing: Complete
    Optimizing --> DisplayingResults: Optimization Done
    
    DisplayingResults --> Downloading: User Requests Download
    DisplayingResults --> Idle: User Closes
    Downloading --> DisplayingResults: Download Complete
    
    FileSummarizing --> ProcessingError: Error
    FunctionExplaining --> ProcessingError: Error
    ProjectFlowGen --> ProcessingError: Error
    QAGenerating --> ProcessingError: Error
    
    ProcessingError --> Recovering: Attempt Recovery
    Recovering --> Optimizing: Partial Results
    Recovering --> Idle: Total Failure
```

---

# UI Wireframes & Mockups

## 10. Wireframe - Initial Upload Screen

```
┌─────────────────────────────────────────────────────────────────────┐
│  Code Analysis Pipeline                                    [⚙ Settings] │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  📁 Upload Code Files                                               │
│  ═══════════════════════════════════════════════════════════      │
│                                                                     │
│  ┌───────────────────────────────────────────────────────────┐    │
│  │                                                           │    │
│  │            🗂️  Drag and drop files here                  │    │
│  │                   or click to browse                      │    │
│  │                                                           │    │
│  │     Supported: .py .js .ts .java .cpp .c .go .rs .rb .php │    │
│  │     Max: 10MB per file, 50 files total                    │    │
│  │                                                           │    │
│  └───────────────────────────────────────────────────────────┘    │
│                                                                     │
│  📋 Instructions:                                                   │
│  • Upload your code files using the area above                     │
│  • Files will be validated automatically                           │
│  • Analysis includes: summaries, explanations, flow & Q&A          │
│  • Processing time: ~30s for small files (<1MB)                    │
│                                                                     │
│  ℹ️  Powered by DSPy & Nvidia API                                   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## 11. Wireframe - File Validation Screen

```
┌─────────────────────────────────────────────────────────────────────┐
│  Code Analysis Pipeline                        🏠 Home │ [⚙ Settings] │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ✅ Files Uploaded Successfully                                     │
│  ═══════════════════════════════════════════════════════════      │
│                                                                     │
│  ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓    │
│  ┃  File Name           Size    Language   Status          ┃    │
│  ┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫    │
│  ┃  ✓ main.py          1.2KB    Python     Valid           ┃    │
│  ┃  ✓ utils.py         3.5KB    Python     Valid           ┃    │
│  ┃  ✓ config.js        4.1KB    JavaScript Valid           ┃    │
│  ┃  ✓ helpers.ts       2.8KB    TypeScript Valid           ┃    │
│  ┃  ⚠ large.py         11.2KB   Python     Too Large       ┃    │
│  ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛    │
│                                                                     │
│  📊 Summary:                                                        │
│  • Total Files: 5                                                  │
│  • Valid Files: 4 ✓                                                │
│  • Invalid Files: 1 ⚠                                              │
│  • Total Size: 10.4KB                                              │
│                                                                     │
│     [ ← Upload More ]  [❌ Cancel]      [▶️ Start Analysis ]       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## 12. Wireframe - Analysis in Progress

```
┌─────────────────────────────────────────────────────────────────────┐
│  Code Analysis Pipeline                        🏠 Home │ [⚙ Settings] │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ⏳ Analysis in Progress...                                         │
│  ═══════════════════════════════════════════════════════════      │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │  Overall Progress               [████████░░░░] 65%          │  │
│  └─────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  Pipeline Components:                                               │
│                                                                     │
│  ✅ File Parsing                    [██████████] Complete           │
│     └─ 4 files parsed successfully                                  │
│                                                                     │
│  🔄 File Summarizer                 [████████░░] 80%                │
│     └─ Generating summaries via Nvidia API...                       │
│                                                                     │
│  🔄 Function Explainer              [██████░░░░] 60%                │
│     └─ Analyzing 12 functions...                                    │
│                                                                     │
│  ⏸️ Project Flow Generator          [░░░░░░░░░░] Waiting...         │
│     └─ Will start after summaries complete                          │
│                                                                     │
│  ⏸️ Q&A Generator                   [░░░░░░░░░░] Waiting...         │
│     └─ Pending previous components                                  │
│                                                                     │
│  ⏱️ Estimated time remaining: ~15 seconds                           │
│                                                                     │
│                         [⏹️ Cancel Analysis]                        │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## 13. Wireframe - Results Dashboard (Tabbed View)

```
┌─────────────────────────────────────────────────────────────────────┐
│  Code Analysis Pipeline                        🏠 Home │ [⚙ Settings] │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ✨ Analysis Complete!                                              │
│  ═══════════════════════════════════════════════════════════      │
│                                                                     │
│  ┌───────┬──────────┬──────────┬─────────┬────────┐                │
│  │ 📄 Files │ 🔧 Functions │ 🌐 Flow │ ❓ Q&A │ 📊 Export │         │
│  └───────┴──────────┴──────────┴─────────┴────────┘                │
│  ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓    │
│  ┃  📄 File Summaries                                      ┃    │
│  ┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫    │
│  ┃                                                         ┃    │
│  ┃  🗂️ main.py                              [🔍 Details]   ┃    │
│  ┃  ─────────────────────────────────────────────────      ┃    │
│  ┃  Purpose: Entry point for the application              ┃    │
│  ┃  Complexity: ⭐⭐⭐ (Medium)                              ┃    │
│  ┃  Components: 3 functions, 1 class                      ┃    │
│  ┃  Dependencies: utils, config, logging                  ┃    │
│  ┃                                                         ┃    │
│  ┃  🗂️ utils.py                             [🔍 Details]   ┃    │
│  ┃  ─────────────────────────────────────────────────      ┃    │
│  ┃  Purpose: Utility functions for data processing        ┃    │
│  ┃  Complexity: ⭐⭐ (Low)                                   ┃    │
│  ┃  Components: 8 functions                               ┃    │
│  ┃  Dependencies: json, re, pathlib                       ┃    │
│  ┃                                                         ┃    │
│  ┃  🗂️ config.js                            [🔍 Details]   ┃    │
│  ┃  ─────────────────────────────────────────────────      ┃    │
│  ┃  Purpose: Configuration management module              ┃    │
│  ┃  Complexity: ⭐ (Low)                                    ┃    │
│  ┃  Components: 2 functions, config object                ┃    │
│  ┃                                                         ┃    │
│  ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛    │
│                                                                     │
│  [ ⬇️ Download All ]  [ 📧 Share ]  [ 🔄 Analyze New Files ]       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## 14. Wireframe - Function Explanations Tab

```
┌─────────────────────────────────────────────────────────────────────┐
│  Code Analysis Pipeline                        🏠 Home │ [⚙ Settings] │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ✨ Analysis Complete!                                              │
│  ═══════════════════════════════════════════════════════════      │
│                                                                     │
│  ┌───────┬──────────┬──────────┬─────────┬────────┐                │
│  │ 📄 Files │ 🔧 Functions │ 🌐 Flow │ ❓ Q&A │ 📊 Export │         │
│  └───────┴──────────┴──────────┴─────────┴────────┘                │
│  ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓    │
│  ┃  🔧 Function Explanations                  [🔍 Search] ┃    │
│  ┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫    │
│  ┃                                                         ┃    │
│  ┃  📌 process_data(data, config)           [📄 main.py] ┃    │
│  ┃  ▼ ───────────────────────────────────────────────────  ┃    │
│  ┃                                                         ┃    │
│  ┃  🎯 Purpose:                                            ┃    │
│  ┃     Processes input data according to configuration    ┃    │
│  ┃                                                         ┃    │
│  ┃  📥 Parameters:                                         ┃    │
│  ┃     • data (dict): Input data dictionary               ┃    │
│  ┃     • config (Config): Configuration object            ┃    │
│  ┃                                                         ┃    │
│  ┃  📤 Returns: ProcessedResult                            ┃    │
│  ┃                                                         ┃    │
│  ┃  🔄 Logic Flow:                                         ┃    │
│  ┃     1. Validate input data structure                   ┃    │
│  ┃     2. Apply configuration transformations             ┃    │
│  ┃     3. Handle edge cases and errors                    ┃    │
│  ┃     4. Return processed results                        ┃    │
│  ┃                                                         ┃    │
│  ┃  ⚙️ Design Patterns: Strategy, Factory                 ┃    │
│  ┃  📊 Complexity: Medium (Cyclomatic: 8)                 ┃    │
│  ┃                                                         ┃    │
│  ┃  ──────────────────────────────────────────────────    ┃    │
│  ┃                                                         ┃    │
│  ┃  📌 validate_input(data)                 [📄 utils.py] ┃    │
│  ┃  ▶ ───────────────────────────────────────────────────  ┃    │
│  ┃                                                         ┃    │
│  ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛    │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## 15. Wireframe - Project Flow Tab

```
┌─────────────────────────────────────────────────────────────────────┐
│  Code Analysis Pipeline                        🏠 Home │ [⚙ Settings] │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ✨ Analysis Complete!                                              │
│  ═══════════════════════════════════════════════════════════      │
│                                                                     │
│  ┌───────┬──────────┬──────────┬─────────┬────────┐                │
│  │ 📄 Files │ 🔧 Functions │ 🌐 Flow │ ❓ Q&A │ 📊 Export │         │
│  └───────┴──────────┴──────────┴─────────┴────────┘                │
│  ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓    │
│  ┃  🌐 Project Structure & Flow                            ┃    │
│  ┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫    │
│  ┃                                                         ┃    │
│  ┃  📊 Dependency Graph:                                   ┃    │
│  ┃                                                         ┃    │
│  ┃      ┌────────┐                                        ┃    │
│  ┃      │main.py │                                        ┃    │
│  ┃      └───┬────┘                                        ┃    │
│  ┃          │                                             ┃    │
│  ┃      ┌───┴───┬────────┐                               ┃    │
│  ┃      ▼       ▼        ▼                               ┃    │
│  ┃  ┌───────┐ ┌──────┐ ┌────────┐                       ┃    │
│  ┃  │utils  │ │config│ │logging │                       ┃    │
│  ┃  └───────┘ └──────┘ └────────┘                       ┃    │
│  ┃                                                         ┃    │
│  ┃  🔄 Execution Paths:                                    ┃    │
│  ┃     • Path 1: main → utils → database                  ┃    │
│  ┃     • Path 2: main → config → service                  ┃    │
│  ┃                                                         ┃    │
│  ┃  ⚠️ Circular Dependencies: None detected ✓             ┃    │
│  ┃                                                         ┃    │
│  ┃  📈 Coupling Metrics:                                   ┃    │
│  ┃     • Average dependencies: 2.3                        ┃    │
│  ┃     • Max dependencies: 4                              ┃    │
│  ┃     • Modularity score: Good                           ┃    │
│  ┃                                                         ┃    │
│  ┃  💡 Architectural Insights:                             ┃    │
│  ┃     • Project has well-defined layered architecture    ┃    │
│  ┃     • Low coupling indicates good modularity           ┃    │
│  ┃     • No circular dependencies found                   ┃    │
│  ┃                                                         ┃    │
│  ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛    │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## 16. Wireframe - Export Options Screen

```
┌─────────────────────────────────────────────────────────────────────┐
│  Code Analysis Pipeline                        🏠 Home │ [⚙ Settings] │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  📊 Export Analysis Results                                         │
│  ═══════════════════════════════════════════════════════════      │
│                                                                     │
│  Select export format and content:                                  │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │  📋 Format Options:                                         │  │
│  │                                                             │  │
│  │     ⦿ JSON  (structured data)                               │  │
│  │     ○ Markdown  (readable documentation)                    │  │
│  │     ○ HTML  (interactive report)                            │  │
│  │     ○ PDF  (printable document)                             │  │
│  └─────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │  📦 Content Selection:                                      │  │
│  │                                                             │  │
│  │     ☑ File Summaries                                        │  │
│  │     ☑ Function Explanations                                 │  │
│  │     ☑ Project Flow Diagrams                                 │  │
│  │     ☑ Q&A Pairs                                             │  │
│  │     ☑ Processing Metadata                                   │  │
│  │                                                             │  │
│  └─────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │  ⚙️ Additional Options:                                     │  │
│  │                                                             │  │
│  │     ☑ Include source code snippets                          │  │
│  │     ☑ Include dependency graphs                             │  │
│  │     ☐ Include raw AST data                                  │  │
│  │                                                             │  │
│  └─────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  📄 Estimated file size: 2.4 MB                                      │
│                                                                     │
│               [ ← Back ]          [ ⬇️ Download Report ]            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Wireframe Descriptions

### Screen 10: Initial Upload Screen
The landing page where users drag and drop or browse for code files. Shows supported file types, size limits, and brief instructions. Clean, minimalist design to encourage easy file upload.

### Screen 11: File Validation Screen
Displays a table of uploaded files with validation status. Shows which files passed validation and which failed (with reasons). Provides summary statistics and action buttons to proceed or upload more files.

### Screen 12: Analysis in Progress
Real-time progress tracking showing the status of each DSPy pipeline component. Displays overall progress bar, individual component progress, and estimated time remaining. Allows users to cancel if needed.

### Screen 13: Results Dashboard - Files Tab
Tabbed interface displaying file summaries with collapsible detail views. Shows purpose, complexity scores, key components, and dependencies for each file. Includes download and sharing options.

### Screen 14: Function Explanations Tab
Detailed expandable list of all functions with comprehensive explanations including purpose, parameters, return values, logic flow, design patterns, and complexity metrics.

### Screen 15: Project Flow Tab
Visual and textual representation of project structure showing dependency graphs, execution paths, coupling metrics, circular dependency warnings, and architectural insights.

### Screen 16: Export Options Screen
Comprehensive export interface allowing users to select format (JSON, Markdown, HTML, PDF), choose which components to include, and configure additional options before downloading the complete analysis report.

## Diagram Descriptions

### Use Case Diagram
Shows the primary actors (Developer, System Administrator) and their interactions with the system. Developers use the analysis features while administrators manage system configuration.

### Process Flow Diagrams
- **High-Level**: Shows the complete user journey from upload to download
- **Detailed DSPy Pipeline**: Illustrates the internal workings of the analysis pipeline with all four modules
- **File Upload**: Details the validation and preprocessing steps
- **Error Handling**: Demonstrates how different error types are handled

### Data Flow Diagrams
- **Level 0 (Context)**: Shows system boundaries and external entities
- **Level 1**: Decomposes the system into major processes and data stores

### Sequence Diagram
Illustrates the temporal ordering of interactions between components during a complete analysis cycle, including parallel processing.

### State Diagram
Shows all possible states of the analysis process and transitions between them, including error states and recovery paths.
