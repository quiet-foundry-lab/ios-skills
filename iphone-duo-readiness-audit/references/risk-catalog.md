# Duo readiness risk catalog

Use this as an inventory, not an automatic defect list. Inspect each match in context and record only actionable or explicitly deferred findings.

## Global display and device assumptions

Search Swift and Objective-C for:

```text
UIScreen\.main|mainScreen|nativeBounds|screen\.bounds
UIDevice\.current|userInterfaceIdiom|modelIdentifier
interfaceOrientation|statusBarOrientation
```

Risk: global values can be ambiguous or stale for multiple displays, scenes, dynamic resizing, and Duo poses.

Preferred direction: use local SwiftUI environment values or UIKit `traitCollection`, specific view/window-scene geometry, and safe-area-aware containers. Preserve domain APIs; pass layout context only in presentation code.

## Fixed geometry and asymmetric safe areas

Search for:

```text
frame\(width:|frame\(height:|\.offset\(|\.position\(|constant: [0-9]
safeAreaInsets\.(left|right)|safeAreaInsets\.left == safeAreaInsets\.right
```

Risk: hardcoded dimensions, symmetric-inset assumptions, and full-screen centering can collide with controls, cameras, or the fold.

Preferred direction: flexible stacks/grids, layout margins, independent safe-area edges, and content priorities. Backgrounds may extend; controls and readable foreground content should not.

## Orientation-driven layout

Search for:

```text
UIDeviceOrientation|UIInterfaceOrientation|isLandscape|isPortrait
width > height|height > width
```

Risk: orientation is not an available-space contract, especially on the inner display and in Split View.

Preferred direction: map local traits and available container space to a semantic presentation mode. Keep orientation only for product behavior genuinely tied to orientation.

## Custom navigation, bars, and presentations

Search for:

```text
custom.*tab|Custom.*Bar|overlay\(.*Button|ZStack.*toolbar|bottomSheet|floating.*button
NavigationView|UINavigationController|UITabBarController|UISplitViewController
```

Risk: custom chrome often bypasses system fold avoidance, overflow, side placement, and safe-area adaptation.

Preferred direction: use `NavigationSplitView`, `TabView`, standard sheets/toolbars, `UISplitViewController`, and `UITabBarController` where possible. Deliberate custom chrome must have explicit available-region and overflow behavior.

For apps built with the iOS 27.1 SDK, inspect toolbar items that may move into a vertical bar: semantic placement and order, title plus icon, custom-item width and axis behavior, and whether essential actions remain accessible through overflow. System navigation containers enable this behavior; custom bars do not automatically participate. Do not prescribe a vertical bar for every screen or treat an older SDK build as a layout defect.

## Custom two-region layouts and reserved regions

Inspect custom split/overlay layouts, centered media or controls, and manually positioned foreground content. Standard navigation, scrolling, and presentation containers already adapt to many Duo configurations; avoid displacing their content solely because a fold exists.

Where a custom primary/secondary layout has a demonstrated problem, consider iOS 27.1 `ArrangementView` or `UIArrangementViewController` with a fallback for earlier OS versions. Check that both regions and their actions remain available when the arrangement changes. Do not put navigation containers inside an arrangement or an arrangement inside a scrolling container without a concrete reason.

Where important custom controls cross the fold or camera, consider local reserved-region queries: a division separates usable regions; an occlusion covers part of one. Query active regions for current obstruction, and check `isActive` before using an inactive region's frame. Do not require reserved-region code when safe areas or system containers already solve the problem.

If hinge APIs are already used, check that the effect resets when hinge data becomes unavailable and essential functionality works without a hinge. Live hinge angle suits interactions and effects; use available space, arrangements, and reserved regions to drive layout.

## Shared UI state and scene blindness

Search for:

```text
UIApplication\.shared|keyWindow|windows\.first|connectedScenes\.first
static var .*selected|singleton|shared.*(router|coordinator|navigation|presentation)
AppDelegate|SceneDelegate
```

Risk: global selection, routing, window, and presentation assumptions break independent scenes, Split View, and external displays.

Preferred direction: scene/feature scope routing, selection, drafts, focus, and presentation. Keep repositories and business policy independent of scene state.

## SwiftUI invalidation and data flow

Search for:

```text
ObservableObject|@Published|@StateObject|@ObservedObject
private var .*: some View|@ViewBuilder
GeometryReader|ForEach\([^)]*indices
```

Risk: monolithic views, coarse invalidation, and index identity make adaptation harder and amplify updates during layout changes.

Preferred direction: use `@Observable` for new models where deployment allows, mark UI-owned models `@MainActor`, pass narrow inputs, split substantial regions into types, and use stable identity. Do not mechanically convert `ObservableObject` without reviewing ownership and concurrency.

## Runtime validation gaps

Inspect UI/snapshot/accessibility tests, scene support, and CI documentation. Compilation and static analysis cannot reveal hidden controls, clipping, state restoration, or behavior while resizing.

Recommend validation for outer/inner displays; open, closed, rotated, and partially folded poses; Split View; PiP; Dynamic Type; VoiceOver; localization; keyboard/focus; loading/error/offline states; and multiple scenes when supported. Change display or pose while navigating, entering text, showing a sheet or menu, or scrolling; verify state and reachable controls throughout the transition. A static screenshot only checks an endpoint.

Record the Xcode/SDK and Simulator version used for any runtime evidence. The iPhone Duo Simulator in Xcode 27.1 can exercise layouts and pose changes. When relevant, reserve physical-device checks for camera switching and capture accessories, touch reachability, haptics, actual display appearance, and performance or thermal behavior. Do not infer these outcomes from source review or Simulator results.

## Source notes

The iOS 27.1 SDK and Duo Simulator guidance is preliminary while in beta. Recheck API names and Simulator capabilities against current Apple documentation before recommending implementation. Starting points: [Prepare your app for iPhone Duo](https://developer.apple.com/videos/play/tech-talks/111461/), [Strike a pose with adaptive layouts](https://developer.apple.com/videos/play/tech-talks/111463/), [Raise the bar with iPhone Duo](https://developer.apple.com/videos/play/tech-talks/111462/), and [Leverage multiple displays and scenes](https://developer.apple.com/videos/play/tech-talks/111464/).

## Triage rules

- A match in dead code, tests, or a deliberate rendering primitive is not automatically a finding; explain exclusions.
- Use P0/P1 when a core action can be hidden, cannot be reached, routes to the wrong scene, or loses user work.
- Group repeated patterns under one root cause; cite representative locations and an occurrence count.
- Recommend pilots that together exercise dense navigation, form entry, and presentation-heavy work.
