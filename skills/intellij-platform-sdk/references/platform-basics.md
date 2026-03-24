# Platform Basics: Actions, Services, VFS, Threading

Use this reference when the user works on general plugin architecture, actions, services, files, background tasks, or threading rules.

## What this file covers

- Action System
- Services
- Virtual File System
- Read/write actions
- dumb mode awareness
- platform-wide architectural guardrails

## Action System

### Core classes

- `AnAction`
- `DumbAwareAction`
- `ToggleAction`
- `DefaultActionGroup`
- `AnActionEvent`
- `Presentation`
- `ActionUpdateThread`

### Action implementation rules

1. Override `actionPerformed()`
2. Override `update()` for visibility/enabled state
3. Override `getActionUpdateThread()` on modern platform versions
4. Keep `update()` extremely fast

### Important rule: no fields in `AnAction`

`AnAction` instances live for the lifetime of the application. Do not store project/editor/file state in fields. That causes leaks and stale state.

### Basic action example

```java
public class HelloWorldAction extends AnAction {
    @Override
    public void actionPerformed(@NotNull AnActionEvent e) {
        Project project = e.getProject();
        Messages.showMessageDialog(project, "Hello", "Title", Messages.getInformationIcon());
    }

    @Override
    public void update(@NotNull AnActionEvent e) {
        e.getPresentation().setEnabledAndVisible(e.getProject() != null);
    }

    @Override
    public @NotNull ActionUpdateThread getActionUpdateThread() {
        return ActionUpdateThread.BGT;
    }
}
```

### Choosing `ActionUpdateThread`

- Use `BGT` when `update()` needs PSI, VFS, or project-model reads
- Use `EDT` when `update()` needs Swing/UI state directly
- Prefer `BGT` for most modern actions

## Services

### Service levels

- Application service: one per IDE instance
- Project service: one per project
- Module service: one per module

### Modern service declaration

```java
@Service(Service.Level.PROJECT)
public final class MyProjectService {
    private final Project project;

    public MyProjectService(Project project) {
        this.project = project;
    }

    public static MyProjectService getInstance(Project project) {
        return project.getService(MyProjectService.class);
    }
}
```

### Service guidance

- Prefer light services with `@Service`
- Avoid eager heavy initialization in constructors
- Do not use old `ServiceManager` APIs in new code
- Put reusable logic in services instead of static helpers inside actions

## Virtual File System

### Key classes

- `VirtualFile`
- `VirtualFileManager`
- `LocalFileSystem`
- `VfsUtil`
- `BulkFileListener`
- `AsyncFileListener`

### Common operations

```java
VirtualFile file = LocalFileSystem.getInstance().findFileByPath(path);
PsiFile psiFile = PsiManager.getInstance(project).findFile(file);
```

### File event listening

Use message-bus listeners or VFS listeners when the user needs to react to file create/change/delete events.

## Threading model

### Core rule

- Read PSI/VFS/project model under read access
- Modify PSI/documents under write access
- UI changes belong on EDT

### Read/write examples

```java
ApplicationManager.getApplication().runReadAction(() -> {
    // safe reads
});

WriteCommandAction.runWriteCommandAction(project, () -> {
    // PSI/document modification
});
```

### Non-blocking reads

Use `ReadAction.nonBlocking()` when a read may take noticeable time and should not block UI responsiveness.

### Kotlin coroutines (2024.1+)

For Kotlin plugins, prefer:

- `readAction { ... }`
- `writeAction { ... }`
- structured concurrency tied to lifecycle scopes

## Dumb mode

During indexing, many index-backed operations are unavailable.

### Use these patterns

- `DumbService.isDumb(project)`
- `DumbService.getInstance(project).runWhenSmart(...)`
- `DumbService.getInstance(project).smartInvokeLater(...)`
- `DumbAwareAction` or `DumbAware` where appropriate

### Important nuance

Only mark code `DumbAware` if it really avoids index-dependent APIs.

## Architecture heuristics

When helping the user implement a feature:

1. Put one-shot UI entrypoints in actions
2. Put reusable business logic in services
3. Put PSI operations behind clear helpers or service methods
4. Separate fast context checks from heavy execution
5. Respect dumb mode and thread boundaries first, optimize later

## When to also read other references

- Read `psi-and-indexing.md` for PSI traversal, references, and custom indexes
- Read `ui-settings-and-toolwindows.md` for dialogs, settings, and tool windows
- Read `troubleshooting.md` if actions are invisible, services fail, or threading assertions appear
