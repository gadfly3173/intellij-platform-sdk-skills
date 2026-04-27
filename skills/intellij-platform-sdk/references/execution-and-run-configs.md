# Execution Model and Run Configurations

Use this reference when the user needs to implement custom run configurations, program runners, execution consoles, or integrate with the IDE execution framework.

## What this file covers

- execution model overview
- run configuration types and factories
- run configuration producers
- settings editors
- program runners
- execution consoles and console filters
- run configuration macros
- before/after launch tasks

## Execution model overview

The IntelliJ Platform execution framework manages running and debugging applications from within the IDE. It consists of three main layers:

1. **Run Configuration** — describes *what* to run (main class, parameters, environment, etc.)
2. **Run Profile / Executor** — describes *how* to run (Run, Debug, Coverage, Profile)
3. **Program Runner** — performs the actual execution

## Core classes

- `RunConfigurationBase` — base class for run configurations
- `ConfigurationFactory` — creates configuration instances
- `ConfigurationType` — registers configuration factories with the platform
- `RunConfigurationProducer` — creates configurations from context (e.g., right-click on a file)
- `SettingsEditor` — UI for editing configuration settings
- `ProgramRunner` — executes a run profile
- `Executor` — pre-defined execution mode (Run, Debug, etc.)
- `ExecutionEnvironment` — bundles configuration, executor, and runner
- `ExecutionException` — thrown on execution failures
- `RunProfileState` — represents the state of a running process

## Run configuration type

Register via `com.intellij.configurationType` EP:

```xml
<configurationType
    implementation="com.example.MyConfigurationType"/>
```

### Minimal ConfigurationType

```java
public class DemoConfigurationType extends ConfigurationTypeBase {
    protected DemoConfigurationType() {
        super("DemoRunConfiguration", "Demo", "Demo run configuration",
              AllIcons.General.RunConfiguration);
    }

    @Override
    public ConfigurationFactory[] getConfigurationFactories() {
        return new ConfigurationFactory[]{
            new ConfigurationFactory(this) {
                @Override
                public @NotNull String getId() {
                    return "Demo";
                }

                @Override
                public @NotNull RunConfiguration createTemplateConfiguration(
                        @NotNull Project project) {
                    return new DemoRunConfiguration(project, this);
                }
            }
        };
    }
}
```

### Run configuration implementation

```java
public class DemoRunConfiguration extends RunConfigurationBase<DemoRunConfigurationOptions> {
    protected DemoRunConfiguration(
            @NotNull Project project,
            @NotNull ConfigurationFactory factory) {
        super(project, factory, "Demo");
    }

    @Override
    public @NotNull SettingsConfigurationPanel getConfigurationEditor() {
        return new DemoSettingsEditor(getProject());
    }

    @Override
    public @Nullable RunProfileState getState(
            @NotNull Executor executor,
            @NotNull ExecutionEnvironment env) throws ExecutionException {
        return new CommandLineState(env) {
            @Override
            protected @NotNull ProcessHandler startProcess() throws ExecutionException {
                GeneralCommandLine commandLine = new GeneralCommandLine();
                commandLine.setExePath("my-tool");
                commandLine.addParameter("--input");
                commandLine.addParameter(getOptions().getFilePath());
                return new OSProcessHandler(commandLine);
            }
        };
    }
}
```

## Configuration options

Define configuration options in a separate class that extends `RunConfigurationOptions`:

```java
public class DemoRunConfigurationOptions extends RunConfigurationOptions {
    private final Property<String> myFilePath =
        string("").provideDelegate(this, "filePath");

    public String getFilePath() {
        return myFilePath.getValue(this);
    }

    public void setFilePath(String path) {
        myFilePath.setValue(this, path);
    }
}
```

## Settings editor

The settings editor provides the UI for configuring run configuration parameters:

```java
public class DemoSettingsEditor extends SettingsEditor<DemoRunConfiguration> {
    private JTextField filePathField;

    @Override
    protected void resetEditorFrom(@NotNull DemoRunConfiguration config) {
        filePathField.setText(config.getOptions().getFilePath());
    }

    @Override
    protected void applyEditorTo(@NotNull DemoRunConfiguration config) {
        config.getOptions().setFilePath(filePathField.getText());
    }

    @Override
    protected @NotNull JComponent createEditor() {
        JPanel panel = new JPanel(new GridBagLayout());
        GridBagConstraints gbc = new GridBagConstraints();
        gbc.fill = GridBagConstraints.HORIZONTAL;
        gbc.insets = JBUI.insets(5);

        gbc.gridx = 0; gbc.gridy = 0;
        panel.add(new JLabel("File path:"), gbc);

        gbc.gridx = 1; gbc.weightx = 1;
        filePathField = new JBTextField();
        panel.add(filePathField, gbc);

        return panel;
    }
}
```

## Run configuration producer

Use `RunConfigurationProducer` to automatically create configurations from context (e.g., right-clicking a file or method):

```java
public class DemoConfigurationProducer extends RunConfigurationProducer<DemoRunConfiguration> {
    protected DemoConfigurationProducer() {
        super(DemoConfigurationType.getInstance());
    }

    @Override
    protected boolean setupConfigurationFromContext(
            @NotNull DemoRunConfiguration config,
            @NotNull ConfigurationContext context,
            @NotNull Ref<PsiElement> sourceElement) {

        VirtualFile file = getFileFromContext(context);
        if (file == null) return false;

        config.getOptions().setFilePath(file.getPath());
        config.setName(file.getName());
        return true;
    }

    @Override
    public boolean isConfigurationFromContext(
            @NotNull DemoRunConfiguration config,
            @NotNull ConfigurationContext context) {

        VirtualFile file = getFileFromContext(context);
        return file != null && file.getPath().equals(config.getOptions().getFilePath());
    }
}
```

Register in `plugin.xml`:

```xml
<runConfigurationProducer
    implementation="com.example.DemoConfigurationProducer"/>
```

## Run configuration macros

Macros provide dynamic expandable values in run configuration inputs. Implement `PathMacroFilter` (EP `com.intellij.pathMacroFilter`) to define custom macros:

```java
public class DemoPathMacroFilter extends PathMacroFilter {
    @Override
    public boolean skipPathMacros(@NotNull PsiElement element) {
        // Return true to prevent macro expansion for this element
        return false;
    }
}
```

Common built-in macros: `$PROJECT_DIR$`, `$MODULE_WORKING_DIR$`, `$FileDir$`, `$FilePath$`, `$ContentRoot$`, `$ModuleFileDir$`.

## Program runner

For custom execution behavior, implement `ProgramRunner`:

```java
public class DemoProgramRunner extends GenericProgramRunner<RunnerSettings> {
    @Override
    public @NotNull String getRunnerId() {
        return "DemoRunner";
    }

    @Override
    public boolean canRun(@NotNull String executorId, @NotNull RunProfile profile) {
        return profile instanceof DemoRunConfiguration && executorId.equals(DefaultRunExecutor.EXECUTOR_ID);
    }
}
```

Register:

```xml
<programRunner implementation="com.example.DemoProgramRunner"/>
```

## Execution console and filters

### ConsoleFilterProvider

Add custom filtering to execution output:

```java
public class DemoConsoleFilterProvider implements ConsoleFilterProvider {
    @Override
    public ConsoleFilter[] getDefaultFilters(@NotNull Project project) {
        return new ConsoleFilter[]{
            new ConsoleFilter() {
                @Override
                public Result applyFilter(
                        @NotNull String line, int entireLength) {
                    if (line.contains("ERROR")) {
                        return new Result(
                            line.length() - "ERROR".length(),
                            entireLength,
                            null,
                            DefaultHyperlinkHighlighting.ERROR
                        );
                    }
                    return null;
                }
            }
        };
    }
}
```

Register:

```xml
<consoleFilterProvider implementation="com.example.DemoConsoleFilterProvider"/>
```

## Before/after launch tasks

Add steps that run before or after execution:

- `BeforeRunTaskProvider` — EP `com.intellij.stepsBeforeRunProvider`
- Extend `BeforeRunTask` and register a provider

Common built-in tasks: Build, Build Project, Run Another Configuration, Launch Web Browser.

## Guidance

- Keep `getState()` lightweight — defer process creation to the returned state object
- Use `CommandLineState` for simple command-line launches
- Extend `OSProcessHandler` when custom I/O handling is needed
- For debugger integration, implement `XDebuggerRunner` or extend the XDebugger framework
- Use `ExecutionEnvironment` to access executor, runner, and project context
- Validate inputs in the settings editor, not at execution time
- Implement `RunConfigurationProducer` when the configuration should be auto-detectable
- Run configurations should extend `RunConfigurationBase` or `LocatableConfigurationBase`

## When to also read other references

- Read `extension_points.md` for exact XML registration syntax
- Read `getting-started.md` for project setup
- Read `code_samples.md` for the `run_configuration` sample
