# UI, Settings, Tool Windows, and Project Integration

Use this reference when the user needs dialogs, tool windows, settings pages, notifications, project wizard integration, or project-view customization.

## What this file covers

- dialogs and IntelliJ UI components
- tool windows
- settings/configurables
- persistent state
- notifications
- project wizard and project-view integration

## UI principles

Prefer IntelliJ UI components and platform conventions over raw Swing where possible.

Useful classes include:

- `DialogWrapper`
- `SimpleToolWindowPanel`
- `JBTextField`, `JBCheckBox`, `JBList`, `Tree`
- `FormBuilder`
- `ScrollPaneFactory`
- `NotificationGroupManager`

## Dialogs

### Quick dialogs

Use `Messages` for simple confirmation/info/input flows.

### Custom dialogs

Use `DialogWrapper` when the dialog has validation, multiple fields, custom buttons, or lifecycle handling.

## Tool windows

### Core classes

- `ToolWindowFactory`
- `ToolWindow`
- `ContentFactory`
- `ContentManager`
- `SimpleToolWindowPanel`
- `ActionToolbar`

### Tool-window pattern

1. Register tool window in `plugin.xml`
2. Create panel/content in `ToolWindowFactory`
3. Add toolbar if actions make sense
4. Keep heavy loading deferred when possible

## Settings and persistent state

### Persistent state

Use `PersistentStateComponent` with `@State` when the user needs plugin settings saved across restarts.

### Settings UI

Use `Configurable` or modern configurable helpers when the user needs a page in Settings/Preferences.

### Typical split

- settings state class/service
- UI component/panel
- configurable wrapper

## Notifications

Use the platform notification system instead of custom popups for normal plugin feedback.

Typical uses:

- success/failure summaries
- background-task completion
- actionable warnings with follow-up buttons

## Project integration

If the user asks for project-level UX changes, consider these hooks:

- project wizard/module wizard integration
- project view panes
- tree structure providers
- custom module types
- framework support providers

## Choosing the right surface

- one-off command entry → action
- persistent side panel → tool window
- plugin preferences → settings page
- short status/feedback → notification
- multi-step structured user input → `DialogWrapper`
- project creation flow → wizard/module/project template APIs

## UX guardrails

- do not interrupt users with unnecessary modal dialogs
- prefer notifications for non-blocking feedback
- keep tool windows useful even when empty
- reflect platform terminology and menu placement
- avoid surprising behavior in core IDE surfaces

## When to also read other references

- Read `platform-basics.md` for actions and threading around UI-triggered work
- Read `extension_points.md` for exact XML declarations
- Read `code_samples.md` for `tool_window`, `settings`, `project_wizard`, and `project_view_pane`
