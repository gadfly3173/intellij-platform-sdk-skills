# IntelliJ Platform Code Samples Reference

This document describes the code samples included in the IntelliJ SDK documentation.

## Available Samples

### action_basics
Demonstrates basic action system concepts.

**Key Classes:**
- `PopupDialogAction` - Basic action with dialog
- `CustomDefaultActionGroup` - Action group implementation

**Concepts:**
- Action registration
- Action groups
- Context awareness

### code_inspection_qodana
Shows how to integrate with Qodana for code inspections.

**Key Classes:**
- Custom inspections
- Qodana integration

### comparing_string_references_inspection
Inspection that compares string references using `==` instead of `equals()`.

**Key Classes:**
- `CompareStringReferencesInspection` - Local inspection tool

**Concepts:**
- Local inspection
- Quick fixes
- PSI traversal

### conditional_operator_intention
Intention to convert conditional operator to if statement and vice versa.

**Key Classes:**
- `ConditionalOperatorIntention` - Intention action

**Concepts:**
- Intention actions
- PSI manipulation

### editor_basics
Editor manipulation basics.

**Key Classes:**
- `EditorIllustrationAction` - Access editor document
- `EditorHandlerIllustration` - Typed handler

**Concepts:**
- Editor access
- Document manipulation
- Caret operations

### facet_basics
Module facet implementation.

**Key Classes:**
- `DemoFacet` - Custom facet
- `DemoFacetConfiguration` - Facet configuration

**Concepts:**
- Module facets
- Facet configuration UI

### framework_basics
Framework support implementation.

**Key Classes:**
- `DemoFrameworkType` - Framework type
- `DemoFrameworkSupportProvider` - Framework support

**Concepts:**
- Framework detection
- Framework support in project wizard

### live_templates
Live templates implementation.

**Key Classes:**
- Template definitions
- Template contexts

**Concepts:**
- Live templates
- Template functions

### max_opened_projects
Limits the number of open projects.

**Key Classes:**
- Project manager listener

**Concepts:**
- Project lifecycle
- Application state

### module
Custom module type.

**Key Classes:**
- `DemoModuleType` - Module type
- `DemoModuleBuilder` - Module builder

**Concepts:**
- Module types
- Module builders

### project_model
Project model manipulation.

**Key Classes:**
- Project structure modification

**Concepts:**
- Project model
- Module management

### project_view_pane
Custom project view pane.

**Key Classes:**
- `DemoProjectViewPane` - Project view pane

**Concepts:**
- Project view customization
- Tree structure

### project_wizard
Custom project wizard steps.

**Key Classes:**
- `DemoProjectWizardStep` - Wizard step

**Concepts:**
- Project wizard
- Wizard steps

### psi_demo
PSI manipulation demonstrations.

**Key Classes:**
- PSI navigation
- PSI modification

**Concepts:**
- PSI tree
- PSI elements

### run_configuration
Custom run configuration.

**Key Classes:**
- `DemoRunConfigurationType` - Configuration type
- `DemoRunConfiguration` - Run configuration
- `DemoSettingsEditor` - Settings editor

**Concepts:**
- Run configurations
- Configuration factory
- Settings editor

### settings
Settings implementation example.

**Key Classes:**
- `AppSettings` - Persistent state component
- `AppSettingsComponent` - Settings UI
- `AppSettingsConfigurable` - Settings configurable

**Concepts:**
- Persistent state
- Settings UI
- Application configurable

### simple_language_plugin
Complete custom language support.

**Key Classes:**
- `SimpleLanguage` - Language definition
- `SimpleFileType` - File type
- `SimpleParserDefinition` - Parser definition
- `SimpleSyntaxHighlighter` - Syntax highlighter
- `SimpleAnnotator` - Annotator
- `SimpleCompletionContributor` - Completion
- `SimpleReferenceContributor` - References

**Concepts:**
- Language definition
- Lexer and parser
- Syntax highlighting
- Code completion
- References
- Intentions
- Find usages
- Rename refactoring
- Structure view
- Formatter
- Folding

### theme_basics
UI theme customization.

**Key Classes:**
- Theme JSON file

**Concepts:**
- UI themes
- Color schemes
- Custom icons

### tool_window
Tool window implementation.

**Key Classes:**
- `DemoToolWindowFactory` - Tool window factory
- `DemoToolWindowContent` - Tool window content

**Concepts:**
- Tool windows
- Content management

### tree_structure_provider
Custom tree structure provider.

**Key Classes:**
- `DemoTreeStructureProvider` - Tree structure provider

**Concepts:**
- Project view tree
- Node decorators

## Sample Structure

Each sample follows this structure:

```
code_samples/<sample_name>/
├── build.gradle.kts
├── gradle/
├── gradle.properties
├── gradlew
├── gradlew.bat
├── settings.gradle.kts
└── src/
    ├── main/
    │   ├── java/
    │   │   └── org/intellij/sdk/<sample>/
    │   │       └── *.java
    │   └── resources/
    │       ├── META-INF/
    │       │   └── plugin.xml
    │       ├── fileTemplates/
    │       ├── icons/
    │       └── liveTemplates/
    └── test/
        ├── java/
        │   └── org/intellij/sdk/<sample>/
        │       └── *Test.java
        └── testData/
            └── *
```

## Running Samples

1. Clone the intellij-sdk-docs repository
2. Open the sample directory in IntelliJ IDEA
3. Import as Gradle project
4. Run the `runIde` Gradle task

```bash
cd code_samples/simple_language_plugin
./gradlew runIde
```
