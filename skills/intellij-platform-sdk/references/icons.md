# Working with Icons

Use this reference when the user needs to add, organize, or reference icons in a plugin — actions, tool windows, file types, tree nodes, or editor gutters.

## What this file covers

- platform vs custom icons
- icon organization and file structure
- icon loading patterns (by path / icon holder class)
- icon variants (dark theme, HiDPI)
- New UI icon requirements
- animated icons
- icon tooltips

## Platform vs Custom Icons

Reuse existing platform icons whenever possible. Use the [Icons list](https://intellij-icons.jetbrains.design) to browse. Platform icons are in `AllIcons`; IDE-bundled plugin icons in their respective `*Icons` classes.

```java
// Platform icon
Icon runIcon = AllIcons.Actions.Execute;

// Plugin-specific icon  
Icon gitIcon = GithubIcons.GithubLogo;
```

## Icon Organization

Icons go in the `resources` directory, typically in an `icons/` sub-folder:

```
src/main/resources/icons/
├── myAction.svg
├── myAction_dark.svg
├── myToolWindow.svg
├── myToolWindow_dark.svg
└── expui/                    # New UI variants
    ├── myAction.svg
    └── myAction_dark.svg
```

## Icon Files and Variants

All icon files in the same directory, following this naming pattern:

| Theme/Resolution | Filename | Icon Size |
|---|---|---|
| Light | `iconName.svg` | W×H |
| Dark | `iconName_dark.svg` | W×H |
| Light + HiDPI | `iconName@2x.svg` | 2×W × 2×H |
| Dark + HiDPI | `iconName@2x_dark.svg` | 2×W × 2×H |

SVG is preferred. Set `width` and `height` attributes (no units) to denote base size. If unspecified, defaults to 16×16.

### Required icon sizes by usage

| Usage | Icon Size |
|---|---|
| Node, Action, Filetype | 16×16 |
| Tool window (classic) | 13×13 |
| Tool window (New UI) | 20×20 + 16×16 (compact mode) |
| Editor gutter (classic) | 12×12 |
| Editor gutter (New UI) | 14×14 |

## Icon Loading

### By path (in plugin.xml only)

```xml
<action icon="/icons/myAction.svg" ... />
<toolWindow icon="/icons/myToolWindow.svg" ... />
```

### Icon holder class (recommended for code reuse)

```java
package icons;

public interface MyIcons {
    Icon Action = IconLoader.getIcon("/icons/action.svg", MyIcons.class);
    Icon ToolWindow = IconLoader.getIcon("/icons/toolWindow.svg", MyIcons.class);
}
```

Kotlin equivalent:

```kotlin
package icons

object MyIcons {
    @JvmField val Action = IconLoader.getIcon("/icons/action.svg", javaClass)
    @JvmField val ToolWindow = IconLoader.getIcon("/icons/toolWindow.svg", javaClass)
}
```

Referencing in plugin.xml:

```xml
<!-- Top-level icons package (automatic prefix) -->
<action icon="MyIcons.MyAction" ... />

<!-- Custom package (must be fully qualified) -->
<action icon="com.example.plugin.MyIcons.MyAction" ... />
```

The path to `IconLoader.getIcon()` **must** start with a leading `/`.

## New UI Icons

To support the New UI (since 2022.3):

1. Create `expui/` directory inside the icon root
2. Copy New UI icon variants into `expui/`
3. Create `<PluginName>IconMappings.json` in resources root
4. Register in plugin.xml:

```xml
<iconMapper mappingFile="MyPluginIconMappings.json"/>
```

Mapping file structure:

```json
{
  "icons": {
    "expui": {
      "myAction.svg": "icons/myAction.svg",
      "myAction_dark.svg": "icons/myAction_dark.svg"
    }
  }
}
```

### New UI tool window icons

| Theme/Mode | Filename | Size |
|---|---|---|
| Light | `icon@20x20.svg` | 20×20 |
| Dark | `icon@20x20_dark.svg` | 20×20 |
| Light + Compact | `icon.svg` | 16×16 |
| Dark + Compact | `icon_dark.svg` | 16×16 |

### New UI icon colors

Use these prescribed colors for New UI icons so the platform can dynamically invert them for active states:

| Theme | Color |
|---|---|
| Light | `#6C707E` |
| Dark | `#CED0D6` |

## Animated Icons

```java
// Simple fixed-delay animation
AnimatedIcon icon = new AnimatedIcon(
    500,  // delay in ms between frames
    AllIcons.Ide.Macro.Recording_1,
    AllIcons.Ide.Macro.Recording_2
);

// Variable-delay frames
AnimatedIcon icon = new AnimatedIcon(
    new AnimatedIcon.Frame(AllIcons.Ide.Macro.Recording_1, 500),
    new AnimatedIcon.Frame(AllIcons.Ide.Macro.Recording_2, 200)
);
```

For list/table/tree renderers, set client property `AnimatedIcon.ANIMATION_IN_RENDERER_ALLOWED` to `true` on the component. Use `AsyncProcessIcon` as an alternative for standard loading indicators.

## Icon Tooltips

Register a resource bundle to provide tooltips for icons in renderers:

```xml
<iconDescriptionBundle bundle="messages.MyIconBundle"/>
```

Define keys as `icon.<path-with-dots>.tooltip`:
- `/nodes/class.svg` → `icon.nodes.class.tooltip`

## Guidance

- Reuse platform icons before creating custom ones — use the online icons list to browse
- SVG is preferred over PNG for HiDPI compatibility
- Always provide a dark theme variant (suffix `_dark`) unless the icon looks fine in dark mode
- For plugins targeting 2022.3+, provide New UI icon variants in the `expui/` directory
- The `IconLoader.getIcon()` path must start with `/` — icons are loaded relative to the resources classpath
- In Kotlin icon holder objects, annotate fields with `@JvmField` so `plugin.xml` can reference them

## When to also read other references

- Read `ui-settings-and-toolwindows.md` for tool window and UI component details
- Read `getting-started.md` for resource directory layout
- Read `extension_points.md` for `iconMapper` registration
