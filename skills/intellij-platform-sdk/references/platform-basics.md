# Platform Basics: Actions, Services, VFS, Threading

Use this reference when the user works on general plugin architecture, actions, services, files, background tasks, threading rules, dynamic plugins, or coroutine patterns.

## What this file covers

- Action System
- Services
- Virtual File System
- Read/write actions
- Kotlin coroutine patterns
- dumb mode awareness
- dynamic plugin support
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
3. Override `getActionUpdateThread()` (required since 2022.3)
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
- Module service: one per module (**not recommended** — increases memory usage for projects with many modules)

### Modern service declaration

Light services use `@Service` directly and do not need `plugin.xml` registration. The class must be `final`. `@Service` without a level parameter defaults to application-level.

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
- The service class must be `final` for light services
- Do not inject dependency services via constructors — acquire them on-demand via `getService()` when needed
- Avoid eager heavy initialization in constructors
- Do not use old `ServiceManager` APIs in new code
- Put reusable logic in services instead of static helpers inside actions

## Virtual File System

The VFS is an application-level persistent snapshot of the file system. It may not reflect real-time disk contents — a refresh operation syncs changes from disk into the VFS.

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

Use `BulkFileListener` subscribed to `VirtualFileManager.VFS_CHANGES` for efficient file event listening. VFS listeners are application-level and receive events from all projects — filter with `ProjectFileIndex.isInContent()` when needed.

## Disposer / Disposable

`Disposable` is the platform's resource cleanup mechanism. `Disposer.register(parent, child)` creates a parent-child tree — when the parent is disposed, all children are disposed automatically.

- Never use `Application` or `Project` directly as parent disposable in plugin code — use a plugin service instead
- Use `Disposer.dispose(disposable)` for explicit cleanup
- Services that implement `Disposable` are disposed when their scope ends (app shutdown or project close)

## Threading model

### Core rule

The platform uses a three-tier lock system: Read Lock, Write Intent Lock, and Write Lock.

- Read PSI/VFS/project model under read access
- Modify PSI/documents under write access (always on EDT)
- UI changes belong on EDT
- Write Intent Lock is implicitly acquired on EDT, allowing read access without explicit lock

### Read/write examples

```java
// General read action
ApplicationManager.getApplication().runReadAction(() -> {
    // safe reads
});

// Write action (general-purpose, no undo)
WriteAction.run(() -> {
    // general write operation
});

// Write command action (for PSI/document modification, supports undo)
WriteCommandAction.runWriteCommandAction(project, () -> {
    // PSI/document modification with undo support
});
```

Use `WriteCommandAction` for PSI/document modifications that need undo support. Use `WriteAction.run()` or `WriteAction.compute()` for other write operations.

### Non-blocking reads

Use `ReadAction.nonBlocking()` for read operations that may take noticeable time. The action gets canceled automatically when a write action arrives, then restarts. For plugins targeting 2024.1+, prefer the coroutine-based `readAction` (WARA) API instead.

### Kotlin coroutines

For Kotlin plugins on 2024.1+, prefer coroutine-based APIs over the Java threading patterns above.

#### Service-scoped coroutines (recommended)

Inject a `CoroutineScope` tied to the service lifecycle:

```kotlin
@Service(Service.Level.PROJECT)
class MyProjectService(
    private val project: Project,
    private val cs: CoroutineScope
) {
    fun doWork() {
        cs.launch {
            val data = readAction {
                // short read action
            }
            withContext(Dispatchers.EDT) {
                // update UI
            }
        }
    }
}
```

#### Read/write suspend APIs

```kotlin
// Short read — Write Allowing Read Action (WARA)
// Canceled and retried when a write action arrives; preferred for most reads
val text = readAction { psiFile.text }

// Long read — Write Blocking Read Action (WBRA)
// Blocks write actions until done; use sparingly for heavy index work
val result = readActionBlocking { processLargeIndex() }

// Smart read (waits for dumb mode to end, then reads)
val symbols = smartReadAction(project) { findSymbols() }

// Smart read blocking variant
val data = smartReadActionBlocking(project) { heavyIndexLookup() }

// Constrained read (specify additional constraints)
val data = constrainedReadAction(ReadConstraint.inSmartMode(project)) {
    findDeclarations()
}

// Read-then-write (atomic — no write action can intervene between read and write)
readAndWriteAction {
    val element = findElement()
    writeAction { modifyElement(element) }
}

// Write action
writeAction { psiClass.add(method) }
```

The key distinction: `readAction` (WARA) yields to write actions — if a write arrives, the read is canceled and retried automatically. `readActionBlocking` (WBRA) prevents writes until it finishes. Prefer WARA for most cases; only use WBRA when reads cannot be restarted cheaply.

#### Dispatchers

- `Dispatchers.Default` — CPU-intensive work
- `Dispatchers.IO` — file/network I/O
- `Dispatchers.EDT` — UI updates on Event Dispatch Thread

#### Action coroutines (since 2024.2)

```kotlin
class MyAction : AnAction() {
    override fun actionPerformed(e: AnActionEvent) {
        currentThreadCoroutineScope().launch {
            val targets = readAction { findTargets(e.project!!) }
            withContext(Dispatchers.EDT) { showResults(targets) }
        }
    }
}
```

`currentThreadCoroutineScope()` in actions is available since 2024.2. For 2024.1, use service-scoped coroutines instead.

#### Avoid deprecated patterns

Do not use `Application.getCoroutineScope()` or `Project.getCoroutineScope()` directly — these are deprecated and cause two kinds of leaks: (1) plugin classloader leaks because the scope outlives the plugin, and (2) project leaks when an application-scoped coroutine holds a project reference after close. Always use the injected `CoroutineScope` from service constructors or `currentThreadCoroutineScope()` in actions (2024.2+).

## Dumb mode

During indexing, many index-backed operations are unavailable.

### Use these patterns

- `DumbService.isDumb(project)`
- `DumbService.getInstance(project).runWhenSmart { ... }`
- `DumbService.getInstance(project).smartInvokeLater { ... }`
- `DumbAwareAction` or `DumbAware` where appropriate (do not override `AnAction.isDumbAware()` — extend `DumbAwareAction` instead)

### Important nuance

Only mark code `DumbAware` if it really avoids index-dependent APIs.

## Dynamic plugins

Dynamic plugins can be installed, updated, and unloaded without restarting the IDE. To support this:

### Requirements

1. **No components** — migrate all components to services, extensions, or listeners
2. **Action groups must have IDs** — every `<group>` in `plugin.xml` needs a unique `id`
3. **Use only dynamic extension points** — all referenced EPs must be marked dynamic
4. **No service overrides** — services with `overrides="true"` prevent dynamic loading
5. **Proper resource cleanup** — use `Disposable` services, not static state
6. **Configurable EP dependencies** — any `Configurable` depending on dynamic EPs must implement `Configurable.WithEpDependencies`

### Coding rules for dynamic compatibility

- Do not store `Project`, `Editor`, or PSI in static fields
- Do not use `FileType` or `Language` instances as `Map` keys — use their string IDs instead
- Use `SmartPsiElementPointer` instead of holding `PsiElement` references
- Register listeners via `plugin.xml` or disposable connections, not static subscriptions
- `CachedValuesManager`-created cached values are automatically invalidated on plugin unload

### Plugin lifecycle listener

```kotlin
class MyDynamicPluginListener : DynamicPluginListener {
    override fun beforePluginUnload(
        pluginDescriptor: IdeaPluginDescriptor,
        isUpdate: Boolean
    ) {
        // cancel long-running tasks, release resources
    }
}
```

Register in `plugin.xml`:

```xml
<applicationListeners>
    <listener class="com.example.MyDynamicPluginListener"
              topic="com.intellij.ide.plugins.DynamicPluginListener"/>
</applicationListeners>
```

### Debugging dynamic unload issues

Start the IDE with VM parameter `-XX:+UnlockDiagnosticVMOptions`, enable internal mode, and set registry key `ide.plugins.snapshot.on.unload.fail` to `true`. On unload failure a `.hprof` heap snapshot is generated in the user home directory. Open it, find the `PluginClassLoader` referencing your plugin ID — every reference to it is a memory leak. Check logs under category `com.intellij.ide.plugins.DynamicPlugins`.

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
