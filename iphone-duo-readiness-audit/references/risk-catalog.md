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

Recommend validation for cover/inner displays; open, closed, rotated, and folded poses; Split View; PiP; Dynamic Type; VoiceOver; localization; keyboard/focus; loading/error/offline states; and multiple scenes when supported.

## Triage rules

- A match in dead code, tests, or a deliberate rendering primitive is not automatically a finding; explain exclusions.
- Use P0/P1 when a core action can be hidden, cannot be reached, routes to the wrong scene, or loses user work.
- Group repeated patterns under one root cause; cite representative locations and an occurrence count.
- Recommend pilots that together exercise dense navigation, form entry, and presentation-heavy work.
