# Liquid Glass: Component Best Practices (iOS 26+)

This guide details the usage and implementation standards for specific Liquid Glass components using native iOS 26+ APIs.

## 1. Backgrounds & Environment

### Background Foundations
Liquid Glass blurs and reflects its environment. For the effect to be visible, the underlying layer must provide color and texture.
*   **Best Practices:**
    *   Use vibrant, multi-color gradients (e.g., using `MorphingBlob` or `Circle` with blurs).
    *   Use `.blendMode(.plusLighter)` for overlapping colored shapes to create "hot spots" that shine through the glass.
    *   Ensure the background remains "alive" with subtle animations (e.g., via `TimelineView`).

## 2. Surfaces & Hierarchy

### Using `.glassEffect` Prominence
iOS 26+ offers different prominence levels for the glass effect.
*   **`.thin` / `.ultraThin`:** Use for transient elements, like tooltips or secondary chips, where the background should remain highly visible.
*   **`.regular`:** The standard choice for cards and main UI surfaces.
*   **`.thick` / `.ultraThick`:** Use for primary navigation bars or elements that require high legibility over complex backgrounds.

### The `GlassEffectContainer`
*   **Critical Rule:** Never let glass elements "float" independently if they are related. 
*   **Usage:** Wrap related elements in a `GlassEffectContainer` to allow the system to coordinate their refractive and reflective properties. Adjust `spacing` to control when elements begin to "merge" optically.

## 3. Interactive Best Practices

### The `.interactive()` State
*   **Rule:** Only apply `.interactive()` to elements that have associated gestures or are `Button` instances.
*   **Behavior:** This flag enables real-time visual feedback (like subtle "squish" and highlight shifting) that mimics physical liquid glass.

### Morphing with `glassEffectID`
*   **Usage:** When a view transitions (e.g., a card expanding into a detail view), assign the same `glassEffectID` to the primary glass surface in both states.
*   **Effect:** The glass material will fluidly morph its shape and boundaries during the animation, maintaining visual continuity.

## 4. Visual Hierarchy Summary (iOS 26+)

| Feature | Modifier / API | Best Usage |
| :--- | :--- | :--- |
| **Primary Container** | `.glassEffect(.regular)` | Main content cards and panels. |
| **Interactive UI** | `.glassEffect(... .interactive())` | Buttons, toggles, and draggable items. |
| **Grouped Elements** | `GlassEffectContainer` | Sidebars, toolbars, and tab bars. |
| **State Changes** | `.glassEffectID(id, in: namespace)` | Expanding cards or morphing menus. |
| **Visual Merging** | `.glassEffectUnion(...)` | Joining discrete items into one surface. |
| **Buttons** | `.buttonStyle(.glass)` | Standard system actions. |
