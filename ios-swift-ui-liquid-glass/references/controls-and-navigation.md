# Liquid Glass: Controls and Navigation Examples (iOS 26+)

These examples demonstrate how to implement the Liquid Glass design language across various interactive UI elements and navigational structures using native iOS 26+ APIs.

## 1. Buttons and Interactive Controls

### System Glass Button Styles
iOS 26+ provides built-in button styles that immediately adopt the liquid glass aesthetic.

```swift
VStack(spacing: 16) {
    Button("Confirm Action") {
        // Action
    }
    .buttonStyle(.glassProminent)

    Button("Cancel") {
        // Action
    }
    .buttonStyle(.glass)
}
```

### Custom Interactive Glass Effect
For custom controls, use the `.interactive()` modifier within `.glassEffect` to enable real-time touch reactions.

```swift
Text("Custom Control")
    .padding()
    .glassEffect(.regular.interactive(), in: .rect(cornerRadius: 18))
    .onTapGesture {
        // Interaction logic
    }
```

## 2. Morphing Transitions and Containers

Use `GlassEffectContainer` and `glassEffectID` for fluid morphing when view hierarchies change.

```swift
struct MorphingMenu: View {
    @State private var isExpanded = false
    @Namespace private var glassSpace

    var body: some View {
        GlassEffectContainer(spacing: 20) {
            if isExpanded {
                // Expanded View
                VStack {
                    Text("Option A")
                    Text("Option B")
                }
                .padding()
                .glassEffect(.regular.interactive(), in: .rect(cornerRadius: 24))
                .glassEffectID("menu", in: glassSpace)
            } else {
                // Collapsed View
                Image(systemName: "plus")
                    .padding()
                    .glassEffect(.regular.interactive(), in: .circle)
                    .glassEffectID("menu", in: glassSpace)
            }
        }
        .onTapGesture {
            withAnimation(.spring(response: 0.4, dampingFraction: 0.8)) {
                isExpanded.toggle()
            }
        }
    }
}
```

## 3. Uniting Adjacent Elements

Use `.glassEffectUnion` to visually merge multiple glass elements into a single coherent surface.

```swift
@Namespace private var namespace

GlassEffectContainer(spacing: 12) {
    HStack(spacing: 12) {
        ForEach(0..<3) { index in
            Circle()
                .frame(width: 50, height: 50)
                .glassEffect()
                // Merges these three circles into a single "blob" appearance
                .glassEffectUnion(id: "toolbar_group", namespace: namespace)
        }
    }
}
```

## 4. Layout and Modals

### Presentation Backgrounds
Apply native glass materials to sheets and full-screen covers.

```swift
.sheet(isPresented: $isPresented) {
    SheetContent()
        .presentationDetents([.medium])
        .presentationBackground(.ultraThinMaterial) 
}
```

### Floating Navigation
Toolbars should be grouped and use the appropriate glass effect prominence.

```swift
HStack {
    Image(systemName: "house")
    Spacer()
    Image(systemName: "person")
}
.padding()
.glassEffect(.thick.interactive(), in: .capsule)
.padding()
```