# MaxwellSim × Mermaid: Diagram Package

The following Mermaid diagrams are prepared to present MaxwellSim both to the technical team.

---

## 1) System Architecture — Flowchart
```mermaid
flowchart TD
  A["UI (Next React)"] --> B["Parameter Parser"]
  B --> C["Pre-Processing (normalize & validate)"]
  C --> D{"Solver Method"}
  D -->|FDTD| E1["FDTD Solver"]
  D -->|FEM| E2["FEM Solver"]
  E1 --> F["Post-Processing"]
  E2 --> F["Post-Processing"]
  F --> G["Field Visualizer"]
  G --> H["Exporter (PNG/CSV/VTK)"]

  %% Side channels
  C --> K["Mesh/Grid Generator"]
  K -.-> E1
  K -.-> E2
  B -.-> L[("Param Cache")]
  E1 -.-> M[("Run Cache")]
  E2 -.-> M
  H --> N[("Artifact Storage")]

  subgraph Telemetry/Logs
    X["Perf Counters"] --> Y["Metrics"]
    Y --> Z["Dashboard"]
  end
  E1 --> X
  E2 --> X

```

---

## 2) Simulation Lifecycle — State Diagram
```mermaid
stateDiagram-v2
  [*] --> Idle
  Idle --> ParamLoaded: Parameters set
  ParamLoaded --> Meshed: Mesh generated
  Meshed --> Ready: BC/Materials ready
  Ready --> Running: Start Run
  Running --> Paused: Pause
  Paused --> Running: Resume
  Running --> Completed: Success
  Running --> Failed: Error
  Completed --> Exported: Export
  Exported --> [*]
```

---

## 3) UI Interaction — Sequence Diagram
```mermaid
sequenceDiagram
  autonumber
  participant U as User
  participant UI as UI/Controls
  participant V as Validator
  participant W as Worker Thread
  participant S as Solver
  participant R as Renderer

  U->>UI: Change slider value
  UI->>V: Validate parameters
  V-->>UI: OK or Error
  UI->>W: startRun(params)
  W->>S: initialize(params)
  S-->>W: ready
  W->>S: step(n)
  S-->>W: progress/fields
  W-->>UI: progress event
  UI->>R: render(fields)
  R-->>U: Visual updated
```

---

## 4) Core Classes — Class Diagram
```mermaid
classDiagram
  class ParamSet {
    +id: string
    +gridSize: Vec3
    +timeStep: number
    +method: Method
    +validate(): Result
  }
  class Material {
    +epsilon: number
    +mu: number
    +sigma: number
  }
  class BoundaryCondition {
    +type: BCType
    +apply(Field): void
  }
  class Grid {
    +nx: int
    +ny: int
    +nz: int
    +dx(): number
  }
  class Field {
    +E: float[][][]
    +H: float[][][]
    +snapshot(): Blob
  }
  class Solver {
    <<abstract>>
    +init(ParamSet, Grid, Material[]): void
    +step(n:int): Field
  }
  class FDTDSolver { }
  class FEMSolver { }
  class PostProcessor {
    +fft(Field): Spectrum
    +measure(Point): Metric
  }
  class Renderer {
    +plot2D(Field): Image
    +plot3D(Field): Mesh
  }
  class Exporter {
    +toPNG(Image): File
    +toCSV(Field): File
    +toVTK(Field): File
  }

  ParamSet --> Grid
  ParamSet --> Material
  ParamSet --> BoundaryCondition
  Solver <|-- FDTDSolver
  Solver <|-- FEMSolver
  Solver --> Field
  PostProcessor --> Field
  Renderer --> Field
  Exporter --> Field
```

---

## 5) Data Model / Runs — Simple ER View
```mermaid
flowchart LR
  P[(ParamSet)] --> R1[(Run)]
  R1 --> OUT1[(Result: fields.tiff)]
  R1 --> OUT2[(Metrics: csv)]
  R1 --> OUT3[(Log: ndjson)]
  click P href "#" "ParamSet details"
```

---

## 6) Roadmap — Gantt
```mermaid
gantt
  dateFormat  YYYY-MM-DD
  title MaxwellSim Roadmap
  section Core
  FDTD core             :done,    fdtd, 2025-08-15,2025-09-05
  FEM prototype         :active,  fem1, 2025-09-06,2025-09-30
  Post-Process (FFT)    :         post,  2025-09-20,2025-10-05
  section UI
  Slider/Controls       :active,  ui1,   2025-09-10,2025-09-25
  Plan X-Z visuals      :         ui2,   2025-09-18,2025-10-02
  section Marketing
  Demo Helicopter scene :         mkt1,  2025-09-22,2025-09-29
```

---

## 7) User Journey — Journey Diagram
```mermaid
journey
  title User Journey
  section First-Time User
    Opens page: 3: UI
    Chooses preset: 4: UX
    Clicks "Simulate": 4: Trust
  section Curious User
    Plays with sliders: 3: UI
    Opens Plan X-Z view: 4: UX
    Downloads PNG/CSV: 5: Success
```

---

## 8) Git Workflow — GitGraph (Optional)
```mermaid
gitGraph
  commit id: "init"
  branch ui
  commit id: "sliders"
  branch solver
  commit id: "fdtd-core"
  checkout main
  merge ui
  merge solver
  commit id: "fft-postprocess"
```

---

