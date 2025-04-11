# Pixel Weaver Demo ✨

**Harnessing the Power of Core Graphics within SwiftUI**

[![Platform](https://img.shields.io/badge/platform-iOS-blue.svg)](https://developer.apple.com/ios/) [![Language](https://img.shields.io/badge/language-Swift-orange.svg)](https://developer.apple.com/swift/) [![SwiftUI](https://img.shields.io/badge/SwiftUI-Ready-green.svg)](https://developer.apple.com/xcode/swiftui/) [![License](https://img.shields.io/badge/license-MIT-lightgrey.svg)](LICENSE) [![License: CC BY 4.0](https://licensebuttons.net/l/by/4.0/88x31.png)](LICENSE-CC-BY)

---
Copyright (c) 2025 Cong Le. All Rights Reserved.

---


## Overview

Welcome to the Pixel Weaver Demo!

Ever found yourself needing the fine-grained control and performance of Core Graphics for custom drawing, but want to keep building your app's main interface with the declarative power of SwiftUI? This project demonstrates exactly how to bridge that gap.

It showcases a common and practical technique using `UIViewRepresentable` to wrap a custom `UIView` (which handles Core Graphics drawing) so it can be seamlessly integrated and controlled within a SwiftUI view hierarchy.

## The Core Technique: Bridging UIKit Drawing to SwiftUI

SwiftUI is fantastic for building user interfaces quickly and declaratively. However, for complex, dynamic, or performance-sensitive custom drawing, UIKit's Core Graphics framework often provides more direct control.

The key to combining these worlds is the `UIViewRepresentable` protocol. It acts as a **bridge**, allowing you to:

1.  **Host:** Place a UIKit `UIView` within your SwiftUI layout.
2.  **Create:** Manage the lifecycle of the `UIView`.
3.  **Update:** Pass data reactively from your SwiftUI state (`@State`, `@Binding`, etc.) to the `UIView`'s properties.
4.  **Coordinate:** (Optional) Handle delegates or callbacks from the `UIView`.

This demo specifically applies this technique to integrate a view that performs custom drawing using Core Graphics.

## How It Works: Architecture 🏗️

The demo is structured into three main parts:

1.  **`DrawingView.swift` (UIKit `UIView`):**
    *   This is a standard `UIView` subclass.
    *   Its primary job is to perform the actual drawing logic within its `override func draw(_ rect: CGRect)` method using Core Graphics APIs (`UIGraphicsGetCurrentContext`, drawing shapes, lines, text, etc.).
    *   It exposes properties like `drawingColor` and `lineWidth` to customize the drawing.
    *   Crucially, `didSet` property observers on these properties call `setNeedsDisplay()`, efficiently telling UIKit that the view needs to be redrawn when its parameters change.

2.  **`CoreGraphicsDrawingView.swift` (SwiftUI `UIViewRepresentable`):**
    *   This struct conforms to `UIViewRepresentable`. It's the **bridge**.
    *   `makeUIView(context:)`: Creates the initial `DrawingView` instance when SwiftUI first needs it.
    *   `updateUIView(_ uiView:context:)`: Called whenever the bound SwiftUI state changes (like the color or line width). This method updates the properties on the existing `DrawingView` instance. The `DrawingView`'s `didSet` takes care of triggering the redraw.
    *   Uses `@Binding` to receive the `selectedColor` and `selectedLineWidth` from the SwiftUI view.

3.  **`ContentView.swift` (SwiftUI `View`):**
    *   This is the main SwiftUI view the user sees.
    *   Holds the application's state using `@State` variables (`selectedColor`, `selectedLineWidth`).
    *   Includes the `CoreGraphicsDrawingView` representable in its layout, binding the state variables to it.
    *   Provides SwiftUI controls (`ColorPicker`, `Slider`) that modify the `@State` variables, which in turn triggers the update cycle through the representable to the `DrawingView`.

**Data Flow Summary:**

`SwiftUI Controls` -> Modify `@State` (in `ContentView`) -> Updates `@Binding` (in `CoreGraphicsDrawingView`) -> Calls `updateUIView` -> Sets properties on `DrawingView` -> `didSet` calls `setNeedsDisplay()` -> UIKit calls `draw(_:)` -> Canvas updates!

---
## Features Demonstrated 🎨

*   **Core Graphics Drawing:** Renders a filled circle, a stroked rectangle, diagonal lines, and text using Core Graphics.
*   **SwiftUI Integration:** Seamlessly embeds the UIKit drawing view within a SwiftUI `VStack`.
*   **Reactive Updates:** Changes made via SwiftUI controls (`ColorPicker`, `Slider`) instantly update the Core Graphics drawing parameters (color, line width) and trigger a redraw.
*   **Clear Separation:** Demonstrates a clean separation between SwiftUI UI/State management and UIKit drawing logic.


---
## Code Highlights 🛠️

Key pieces of the code to understand this technique:

*   **`DrawingView.swift`:**
    *   `override func draw(_ rect: CGRect)`: Where all the Core Graphics magic happens.
    *   `drawingColor: UIColor { didSet { setNeedsDisplay() } }`: How changes trigger redraws.
*   **`CoreGraphicsDrawingView.swift`:**
    *   `func makeUIView(...)`: View instantiation.
    *   `func updateUIView(...)`: Passing data from SwiftUI -> UIKit.
    *   `@Binding var drawingColor: Color`: Connecting SwiftUI state.
*   **`ContentView.swift`:**
    *   `@State private var selectedColor: Color`: Owning the source-of-truth state.
    *   `CoreGraphicsDrawingView(drawingColor: $selectedColor, ...)`: Using the representable and binding state.


---

## Visual Peek

*(Imagine a screenshot or GIF here showing the app running, with the Core Graphics drawing area and the SwiftUI controls below it. Maybe show the color or line width changing and the drawing updating.)*

**(Placeholder: A drawing of a blue outlined rectangle with diagonal lines, a semi-transparent blue circle, and the text "Core Graphics" near the bottom. Below it, a SwiftUI ColorPicker and a Slider.)**


---

## Getting Started

1.  Clone this repository.
2.  Open the `PixelWeaverDemo.xcodeproj` (or the Swift Package if structured that way) in Xcode.
3.  Select an iOS Simulator or connect a physical device.
4.  Run the app (Cmd+R).
5.  Interact with the Color Picker and Slider to see the Core Graphics view update in real-time!


---


## Potential Enhancements / Next Steps

This demo provides the foundation. You could extend it by:

*   Adding **touch handling** to `DrawingView` (using `touchesBegan`, `touchesMoved`, etc.) to create a simple drawing app.
*   Implementing **saving/loading** of the drawing.
*   Drawing more complex shapes or using `CGPath` for intricate paths.
*   Adding **zoom/pan** capabilities to the `DrawingView`.
*   Investigating performance optimizations for very complex drawing scenarios.
*   Handling **Undo/Redo** actions.


---


## Contributing

Found a bug or have an improvement? Feel free to open an issue or submit a pull request! Contributions are welcome.

----

## License


- **MIT License:**  [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE) - Full text in [LICENSE](LICENSE) file.
- **Creative Commons Attribution 4.0 International:** [![License: CC BY 4.0](https://licensebuttons.net/l/by/4.0/88x31.png)](LICENSE-CC-BY) - Legal details in [LICENSE-CC-BY](LICENSE-CC-BY) and at [Creative Commons official site](http://creativecommons.org/licenses/by/4.0/).


---

[Back to Top](#top)

