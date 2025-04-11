---
created: 2025-04-11 05:31:26
author: Cong Le
version: "1.0"
license(s): MIT, CC BY 4.0
copyright: Copyright (c) 2025 Cong Le. All Rights Reserved.
---



# A Diagrammatic Guide
> **Disclaimer:**
>
> This document contains my personal notes on the topic,
> compiled from publicly available documentation and various cited sources.
> The materials are intended for educational purposes, personal study, and reference.
> The content is dual-licensed:
> 1. **MIT License:** Applies to all code implementations (Swift, Mermaid, and other programming languages).
> 2. **Creative Commons Attribution 4.0 International License (CC BY 4.0):** Applies to all non-code content, including text, explanations, diagrams, and illustrations.
---



First, a quick summary of the code:

This Swift code demonstrates how to integrate a custom `UIView` that performs Core Graphics drawing into a SwiftUI application. It achieves this using the `UIViewRepresentable` protocol.

1.  **`DrawingView` (UIKit `UIView`):** This class handles the actual drawing using Core Graphics within its `draw(_:)` method. It has properties (`drawingColor`, `lineWidth`) that control the appearance of the drawing. When these properties change, it triggers a redraw using `setNeedsDisplay()`.
2.  **`CoreGraphicsDrawingView` (SwiftUI `UIViewRepresentable`):** This struct acts as a bridge between SwiftUI and the UIKit `DrawingView`. It creates (`makeUIView`) and updates (`updateUIView`) the `DrawingView` instance, passing data from SwiftUI's state (`@Binding` variables) to the `DrawingView`'s properties.
3.  **`ContentView` (SwiftUI `View`):** This is the main SwiftUI view. It holds the state variables (`@State`) for color and line width, displays the `CoreGraphicsDrawingView`, and provides UI controls (`ColorPicker`, `Slider`) to modify the state, which in turn updates the drawing.

Here are the diagrams illustrating these concepts:

---

## 1. Overall Architecture & Component Interaction

This diagram shows the main components and how they relate to each other.

```mermaid
---
title: "Overall Architecture & Component Interaction"
author: "Cong Le"
version: "1.0"
license(s): "MIT, CC BY 4.0"
copyright: "Copyright (c) 2025 Cong Le. All Rights Reserved."
config:
  layout: elk
  look: handDrawn
  theme: dark
---
%%%%%%%% Mermaid version v11.4.1-b.14
%%%%%%%% Toggle theme value to `base` to activate the initilization below for the customized theme version.
%%%%%%%% Available curve styles include the following keywords:
%% basis, bumpX, bumpY, cardinal, catmullRom, linear, monotoneX, monotoneY, natural, step, stepAfter, stepBefore.
%%{
  init: {
    'graph': { 'htmlLabels': false, 'curve': 'linear' },
    'fontFamily': 'Monospace',
    'themeVariables': {
      'primaryColor': '#BEF',
      'primaryTextColor': '#55ff',
      'primaryBorderColor': '#7c2',
      'lineColor': '#F8B229',
      'secondaryColor': '#EE2',
      'tertiaryColor': '#fff',
      'stroke':'#3323',
      'stroke-width': '0.5px'
    }
  }
}%%
graph TD
    subgraph SwiftUI_Layer["SwiftUI Layer"]
        A["ContentView<br/>(View)"] -- Contains --> B("CoreGraphicsDrawingView(UIViewRepresentable)")
        A -- Manages State --> State["@State: selectedColor, selectedLineWidth"]
        State -- Changes --> Controls["Controls<br/>(ColorPicker, Slider)"]
        Controls -- User Interaction --> State
    end

    subgraph Bridge["Bridge"]
        B -- Uses Bindings --> Bindings["@Binding: drawingColor, lineWidth"]
        State -- Provides Data --> Bindings
        B -- Creates & Updates --> C{"DrawingView<br/>(UIView)"}
    end

    subgraph UIKit_Layer["UIKit Layer"]
        C -- Has Properties --> Props["drawingColor, lineWidth"]
        C -- Performs Drawing --> Draw["draw(_:) Method"]
        Draw -- Uses --> CG["Core Graphics Context"]
    end

    Bindings -- Passed via updateUIView --> Props
    Props -- didSet triggers --> NeedsDisplay("setNeedsDisplay()")
    NeedsDisplay -- Invokes --> Draw

    style SwiftUI Layer fill:#e0f7fa,stroke:#00796b,stroke-width:2px
    style Bridge fill:#fff9c4,stroke:#fbc02d,stroke-width:2px
    style UIKit Layer fill:#ffebee,stroke:#c2185b,stroke-width:2px
    
```

**Explanation:**

*   **SwiftUI Layer:** `ContentView` holds the UI controls and the state variables. It includes the `CoreGraphicsDrawingView`.
*   **Bridge:** `CoreGraphicsDrawingView` acts as the intermediary, using `@Binding` to receive data from `ContentView`'s `@State`. It manages the lifecycle of the `DrawingView`.
*   **UIKit Layer:** `DrawingView` is the custom UIKit view. It receives property updates from the bridge and uses its `draw(_:)` method with Core Graphics to render content.
*   **Data Flow:** User interaction changes `@State`, which updates the `@Binding`, triggering `updateUIView` in the representable. This sets properties on `DrawingView`, whose `didSet` observers call `setNeedsDisplay()`, leading UIKit to eventually call `draw(_:)`.

---

## 2. Data Flow on State Change (e.g., User Adjusts Slider)

This sequence diagram illustrates the flow of events when a SwiftUI control modifies the state.

```mermaid
---
title: "Data Flow on State Change (e.g., User Adjusts Slider)"
author: "Cong Le"
version: "1.0"
license(s): "MIT, CC BY 4.0"
copyright: "Copyright (c) 2025 Cong Le. All Rights Reserved."
config:
  theme: dark
---
%%%%%%%% Mermaid version v11.4.1-b.14
%%%%%%%% Available curve styles include the following keywords:
%% basis, bumpX, bumpY, cardinal, catmullRom, linear, monotoneX, monotoneY, natural, step, stepAfter, stepBefore.
%%{
  init: {
    'sequenceDiagram': { 'htmlLabels': false},
    'fontFamily': 'verdana',
    'themeVariables': {
      'primaryColor': '#B28',
      'primaryTextColor': '#F8B229',
      'primaryBorderColor': '#7C33',
      'secondaryColor': '#0615'
    }
  }
}%%
sequenceDiagram
    autonumber

    actor User
    box rgb(202, 12, 22, 0.1) The App System
        participant Controls as SwiftUI Controls Slider
        participant ContentView as ContentView <br/>@State
        participant Representable as CoreGraphicsDrawingView <br/>@Binding
        participant DrawingView as DrawingView UIView
        participant UIKit
    end

    User->>+Controls: Adjusts Slider
    Controls->>+ContentView: Updates selectedLineWidth<br/>(@State)
    ContentView-->>-Representable: SwiftUI detects @State change, triggers update
    Representable->>+DrawingView: updateUIView() called: sets lineWidth property
    Note right of DrawingView: didSet observer for lineWidth runs
    DrawingView->>DrawingView: Calls setNeedsDisplay()
    DrawingView-->>-UIKit: Requests redraw for the view's bounds
    Note right of UIKit: UIKit schedules draw call for next run loop
    UIKit->>+DrawingView: Calls draw(_:)
    DrawingView->>DrawingView: Executes Core Graphics commands<br/>(using new lineWidth)
    DrawingView-->>-UIKit: Drawing complete for this cycle
    
```


**Explanation:**

1.  The user interacts with a control (e.g., `Slider`).
2.  The control updates the corresponding `@State` variable in `ContentView`.
3.  SwiftUI automatically detects the state change and calls the `updateUIView` method of the `CoreGraphicsDrawingView` representable.
4.  `updateUIView` updates the `lineWidth` property on the `DrawingView` instance.
5.  The `didSet` property observer on `lineWidth` in `DrawingView` is triggered and calls `setNeedsDisplay()`.
6.  This signals to the UIKit framework that the view needs to be redrawn.
7.  At the next appropriate time in the rendering cycle, UIKit calls the `draw(_:)` method of the `DrawingView`.
8.  The `draw(_:)` method executes, using the *new* `lineWidth` value to draw the content.

----

## 3. `UIViewRepresentable` Lifecycle Methods

This flowchart shows the primary lifecycle methods of the `UIViewRepresentable` used in this code.

```mermaid
---
title: "`UIViewRepresentable` Lifecycle Methods"
author: "Cong Le"
version: "1.0"
license(s): "MIT, CC BY 4.0"
copyright: "Copyright (c) 2025 Cong Le. All Rights Reserved."
config:
  layout: elk
  look: handDrawn
  theme: dark
---
%%%%%%%% Mermaid version v11.4.1-b.14
%%%%%%%% Toggle theme value to `base` to activate the initilization below for the customized theme version.
%%%%%%%% Available curve styles include the following keywords:
%% basis, bumpX, bumpY, cardinal, catmullRom, linear, monotoneX, monotoneY, natural, step, stepAfter, stepBefore.
%%{
  init: {
    'graph': { 'htmlLabels': false, 'curve': 'linear' },
    'fontFamily': 'Monospace',
    'themeVariables': {
      'primaryColor': '#ffff',
      'primaryTextColor': '#55ff',
      'primaryBorderColor': '#7c2',
      'lineColor': '#F8B229',
      'secondaryColor': '#006100',
      'tertiaryColor': '#fff'
    }
  }
}%%
graph TD
    A["SwiftUI needs the view"] --> B{"makeUIView(context:)"}
    B -- Creates --> C("DrawingView Instance")
    B -- Sets Initial --> D["Set Initial Properties (color, width) on C"]
    C --> E{"View is Displayed"}

    F["SwiftUI State Changes<br/>(@State in ContentView)"] --> G{"updateUIView(_:context:)"}
    G -- Updates --> H["Update DrawingView Properties from @Binding"]
    H --> I{"Property didSet triggers setNeedsDisplay()"}
    I --> J["UIKit schedules draw(_:)"]

    %% View exists, state changes can occur
    E --> F

    subgraph Optional_Methods["Optional Methods"]
        K["makeCoordinator()"]
        L["sizeThatFits(_:uiView:context:)"]
    end

    style Optional_Methods fill:#f3e5f5,stroke:#8e24aa
    
```

**Explanation:**

*   **`makeUIView`:** Called *once* when SwiftUI initially needs to create the underlying UIKit view (`DrawingView`). It's responsible for instantiation and initial configuration.
*   **`updateUIView`:** Called whenever the data bound via `@Binding` changes in the SwiftUI environment. It's responsible for passing the updated data to the existing `DrawingView` instance.
*   **Optional Methods:** `makeCoordinator` (for delegates/callbacks) and `sizeThatFits` (for layout suggestions) are not implemented in this specific example but are part of the `UIViewRepresentable` protocol.

---

## 4. `DrawingView` - `draw(_:)` Execution Flow

This flowchart details the steps taken within the `draw(_:)` method of the `DrawingView`.

```mermaid
---
title: "`DrawingView` - `draw(_:)` Execution Flow"
author: "Cong Le"
version: "1.0"
license(s): "MIT, CC BY 4.0"
copyright: "Copyright (c) 2025 Cong Le. All Rights Reserved."
config:
  layout: elk
  look: handDrawn
  theme: dark
---
%%%%%%%% Mermaid version v11.4.1-b.14
%%%%%%%% Toggle theme value to `base` to activate the initilization below for the customized theme version.
%%%%%%%% Available curve styles include the following keywords:
%% basis, bumpX, bumpY, cardinal, catmullRom, linear, monotoneX, monotoneY, natural, step, stepAfter, stepBefore.
%%{
  init: {
    'graph': { 'htmlLabels': false, 'curve': 'linear' },
    'fontFamily': 'Monospace',
    'themeVariables': {
      'primaryColor': '#ffff',
      'primaryTextColor': '#55ff',
      'primaryBorderColor': '#7c2',
      'lineColor': '#F8B229',
      'secondaryColor': '#006100',
      'tertiaryColor': '#fff'
    }
  }
}%%
graph TD
    A["draw(_ rect:)<br/>Called by UIKit"] --> B{"Get Current Graphics Context?"}
    B -- Yes --> C["Context Obtained<br/>(UIGraphicsGetCurrentContext)"]
    B -- No --> Z["Fail:<br/>Print Error & Exit"]

    C --> D["Set Fill Color<br/>(Semi-Transparent)"]
    D --> E["Fill Ellipse<br/>(Circle)"]
    E --> F["Set Stroke Color"]
    F --> G["Set Line Width"]
    G --> H["Stroke Rectangle<br/>(Inset)"]
    H --> I["Define Lines<br/>(MoveTo, AddLine)"]
    I --> J["Stroke Path<br/>(Draw Lines)"]
    J --> K["Prepare Text Attributes"]
    K --> L["Draw NSAttributedString"]
    L --> Y["Drawing Complete for this Frame"]

    style Z fill:#fda2,stroke:#d32f2f
    
```


**Explanation:**

1.  The process starts when UIKit invokes `draw(_:)`.
2.  It critically depends on getting the current graphics context (`CGContext`). If this fails, drawing cannot proceed.
3.  A sequence of Core Graphics commands are executed:
    *   Setting fill color and drawing a filled shape (ellipse).
    *   Setting stroke color and line width, then drawing stroked shapes (rectangle, lines).
    *   Preparing text attributes and drawing text using `NSAttributedString`.
4.  Each step builds upon the previous ones within the same drawing context for that frame.

----

## 5. Class/Structure Relationship Overview

A simplified diagram showing the types and their primary roles/members.

```mermaid
---
title: "Class/Structure Relationship Overview"
author: "Cong Le"
version: "1.0"
license(s): "MIT, CC BY 4.0"
copyright: "Copyright (c) 2025 Cong Le. All Rights Reserved."
config:
  layout: elk
  look: handDrawn
  theme: dark
---
%%%%%%%% Mermaid version v11.4.1-b.14
%%%%%%%% Toggle theme value to `base` to activate the initilization below for the customized theme version.
%%%%%%%% Available curve styles include the following keywords:
%% basis, bumpX, bumpY, cardinal, catmullRom, linear, monotoneX, monotoneY, natural, step, stepAfter, stepBefore.
%%{
  init: {
    'graph': { 'htmlLabels': false, 'curve': 'linear' },
    'fontFamily': 'Monospace',
    'themeVariables': {
      'primaryColor': '#ffff',
      'primaryTextColor': '#55ff',
      'primaryBorderColor': '#7c2',
      'lineColor': '#F8B229',
      'secondaryColor': '#006100',
      'tertiaryColor': '#fff'
    }
  }
}%%
classDiagram
    class View
    class UIView
    class NSObject

    class ContentView {
        +State selectedColor: Color
        +State selectedLineWidth: CGFloat
        +body: View
    }
    ContentView --|> View

    class CoreGraphicsDrawingView {
        +Binding drawingColor: Color
        +Binding lineWidth: CGFloat
        +makeUIView() DrawingView
        +updateUIView(DrawingView)
    }
    CoreGraphicsDrawingView ..> DrawingView : Creates & Manages
    CoreGraphicsDrawingView ..> ContentView : Used by

    class DrawingView {
        +drawingColor: UIColor
        +lineWidth: CGFloat
        +draw(CGRect)
        +setNeedsDisplay()
    }
    DrawingView --|> UIView

    class UIColor
    class CGFloat
    class Color

    CoreGraphicsDrawingView --* ContentView : <<binds>> state via Binding
    DrawingView --* CoreGraphicsDrawingView : <<updates>> properties

    note for CoreGraphicsDrawingView "Implements UIViewRepresentable"
    note for DrawingView "Handles Core Graphics Drawing"

```


**Explanation:**

*   Shows the inheritance (`--|>`) and composition/dependency relationships (`-->`, `..>`).
*   Highlights key properties (`@State`, `@Binding`, regular properties) and methods (`makeUIView`, `updateUIView`, `draw`).
*   Illustrates that `CoreGraphicsDrawingView` acts as the bridge connecting the SwiftUI `ContentView` state to the UIKit `DrawingView`'s drawing capabilities.



---
**Licenses:**

- **MIT License:**  [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE) - Full text in [LICENSE](LICENSE) file.
- **Creative Commons Attribution 4.0 International:** [![License: CC BY 4.0](https://licensebuttons.net/l/by/4.0/88x31.png)](LICENSE-CC-BY) - Legal details in [LICENSE-CC-BY](LICENSE-CC-BY) and at [Creative Commons official site](http://creativecommons.org/licenses/by/4.0/).

---