---
name: ios-swift-ui-liquid-glass
description: Implement, review, or improve SwiftUI features using the iOS 26+ Liquid Glass API. Use when asked to adopt Liquid Glass in SwiftUI, refactor to Liquid Glass, or review Liquid Glass usage.
---

# SwiftUI Liquid Glass Design Protocol (iOS 26+)

This skill provides the required guidelines and implementation patterns for applying the Liquid Glass design language in SwiftUI. It ensures that UI components use appropriate transparency, blending, structural elements, and interactive lighting using the native iOS 26+ APIs.

## Core Principles

For iOS 26 and later, Apple provides a native API for Liquid Glass that combines optical glass properties with fluidity.
1.  **Native API Priority:** Use `.glassEffect(...)` as the primary modifier for all glass surfaces.
2.  **Container Usage:** Wrap multiple co-existing glass elements in a `GlassEffectContainer(spacing: ...)`. This manages how the glass surfaces merge and blend.
3.  **Modifier Order:** Always apply `.glassEffect()` *after* layout (padding, frame) and appearance modifiers.
4.  **Interactivity:** Append `.interactive()` to the effect (e.g., `.glassEffect(.regular.interactive())`) for elements that respond to touch or pointer input.
5.  **Shape Definition:** Specify shapes directly within the effect: `.glassEffect(..., in: .rect(cornerRadius: 20))`. Supported shapes include `.capsule`, `.circle`, and `.rect`.
6.  **Morphing & Grouping:** 
    *   Use `.glassEffectID(id, in: namespace)` with `@Namespace` for fluid morphing transitions between view states.
    *   Use `.glassEffectUnion(id: "group", namespace: namespace)` to visually unite adjacent glass elements within a container.

## Implementation Guidelines

### Native Implementation Patterns

```swift
// 1. Group multiple glass views to manage blending
GlassEffectContainer(spacing: 24) {
    HStack(spacing: 24) {
        // 2. Apply interactive glass effect with shape
        Image(systemName: "scribble.variable")
            .padding()
            .glassEffect(.regular.interactive(), in: .rect(cornerRadius: 16))
        
        // 3. Tinted non-interactive glass
        Text("Status Label")
            .padding()
            .glassEffect(.regular.tint(.blue), in: .capsule)
    }
}
```

### Component References

👉 **Read [references/controls-and-navigation.md](references/controls-and-navigation.md) for concrete examples of Liquid Glass controls and morphing transitions.**

👉 **Read [references/component-best-practices.md](references/component-best-practices.md) for detailed best practices and usage guidelines for specific components.**

## Checklist for Reviewing Views

When instructed to apply Liquid Glass or review an existing view:
- [ ] **Composition:** Multiple glass views are wrapped in `GlassEffectContainer` with appropriate spacing.
- [ ] **Modifier Order:** `.glassEffect()` is applied *after* layout/appearance modifiers.
- [ ] **Interactivity:** `.interactive()` is used *only* where user interaction exists (e.g., buttons, draggable items).
- [ ] **Transitions:** `glassEffectID` is used with `@Namespace` for morphing between states.
- [ ] **Union:** `.glassEffectUnion` is used to visually merge related adjacent glass elements within a container.
- [ ] **Consistency:** Shapes, tinting, and spacing align across the feature.
- [ ] **Native Components:** Verify that `.buttonStyle(.glass)` or `.buttonStyle(.glassProminent)` is used for standard actions.