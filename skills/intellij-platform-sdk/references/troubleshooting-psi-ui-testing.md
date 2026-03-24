# PSI, UI, Testing, and Debugging Troubleshooting

Use this reference when the problem is about PSI validity, write actions, dumb mode, editor/document coordination, UI updates, memory leaks, test fixtures, or local debugging.

## PSI issues

### Null pointer or invalid PSI access

Common causes:

- missing read access
- invalid/disposed PSI element
- attempting to use PSI before parsing/indexing state is ready

Pattern:

```java
PsiElement element = ReadAction.compute(() -> psiFile.findElementAt(offset));
if (element == null || !element.isValid()) {
    return;
}
```

### `Write access is allowed inside write-action only`

Wrap PSI/document modifications in a write command.

```java
WriteCommandAction.runWriteCommandAction(project, () -> {
    psiClass.add(method);
});
```

### `IndexNotReadyException`

You are likely touching an index-backed API during dumb mode.

Use:

```java
if (DumbService.isDumb(project)) {
    DumbService.getInstance(project).runWhenSmart(() -> {
        // retry when smart
    });
    return;
}
```

Or make the feature `DumbAware` only when it is truly safe. For actions, extend `DumbAwareAction` instead of `AnAction` (do not override `AnAction.isDumbAware()`). For tool windows, implement `DumbAware` on the `ToolWindowFactory`.

## Editor/document issues

### `Document is locked by write PSI operations`

Coordinate PSI/document changes with `PsiDocumentManager` and avoid mixing document mutations carelessly during PSI write flows.

```java
PsiDocumentManager.getInstance(project)
    .doPostponedOperationsAndUnblockDocument(document);
```

### Caret position wrong after edit

Explicitly move the caret and optionally scroll to it after the write completes.

## Testing issues

### Fixture is null or not initialized

Call `super.setUp()` and configure test content before invoking highlighting/completion/etc.

### Test data path problems

Return a deterministic `getTestDataPath()` and keep test data under `src/test/testData` when possible.

### Heavy test required

If a feature depends on a fuller platform/project environment, switch from a light fixture to `HeavyPlatformTestCase`.

## UI issues

### Dialog too small / content clipped

Set minimum or preferred sizes in dialog initialization when layout alone is insufficient.

### UI not refreshing

Update UI on EDT and call `revalidate()` / `repaint()` where appropriate.

```java
ApplicationManager.getApplication().invokeLater(() -> {
    label.setText(newText);
    panel.revalidate();
    panel.repaint();
});
```

## Memory leaks

Common causes:

- static fields holding `Project`, `Editor`, or PSI
- listeners not disposed
- long-lived objects caching short-lived IDE state

Use disposables and lifecycle-aware services instead.

```java
Disposer.register(parent, () -> {
    // cleanup
});
```

## Debugging tips

### Enable internal mode

From the main menu, select `Help | Edit Custom Properties...` to open (or create) `idea.properties`, then add:

```properties
idea.is.internal=true
```

Save and restart the IDE.

### Useful debugging flows

- run `runIde` with debugger attached
- inspect PSI structure using internal tools or PSI viewer
- inspect extension points from diagnostic tools
- use debug flags or profiler args for sandbox IDE runs

## Threading error example

### `Access is allowed from event dispatch thread only`

This usually means UI work is happening off EDT.

```java
ApplicationManager.getApplication().invokeLater(() -> {
    // UI code
});
```

## See also

- `platform-basics.md`
- `psi-and-indexing.md`
- `ui-settings-and-toolwindows.md`
- `testing-and-publishing.md`
- `patterns.md`
