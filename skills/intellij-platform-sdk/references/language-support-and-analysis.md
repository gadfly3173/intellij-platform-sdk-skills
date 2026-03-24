# Language Support and Code Analysis

Use this reference when the user is implementing a custom language plugin or language-aware features such as highlighting, completion, inspections, intentions, quick fixes, documentation, formatting, folding, or structure view.

## What this file covers

- language and file type definitions
- lexer/parser setup
- syntax highlighting
- completion
- annotators
- inspections
- intentions and quick fixes
- surrounding language-support ecosystem

## Custom language support stack

A serious language plugin often includes most of the following:

1. `Language`
2. `LanguageFileType`
3. lexer
4. parser / `ParserDefinition`
5. PSI model
6. syntax highlighter
7. completion contributor
8. references / contributor
9. annotator or inspections
10. formatter
11. folding builder
12. structure view
13. documentation provider
14. find usages provider
15. rename support

## Language and file type

```java
public class SimpleLanguage extends Language {
    public static final SimpleLanguage INSTANCE = new SimpleLanguage();

    private SimpleLanguage() {
        super("Simple");
    }
}
```

```java
public final class SimpleFileType extends LanguageFileType {
    public static final SimpleFileType INSTANCE = new SimpleFileType();

    private SimpleFileType() {
        super(SimpleLanguage.INSTANCE);
    }

    @Override
    public @NotNull String getName() {
        return "Simple File";
    }

    @Override
    public @NotNull String getDescription() {
        return "Simple language file";
    }

    @Override
    public @NotNull String getDefaultExtension() {
        return "simple";
    }

    @Override
    public Icon getIcon() {
        return SimpleIcons.FILE;
    }
}
```

## Parser definition

Use `ParserDefinition` to wire lexer, parser, whitespace/comment/string token sets, and PSI file creation.

## Syntax highlighting

Use `SyntaxHighlighter` when the user needs token-based editor coloring.

Use `Annotator` or inspections when the request involves semantic analysis or error reporting.

## Completion

Use `CompletionContributor` when completions depend on context, PSI position, or surrounding syntax.

Typical pieces:

- `CompletionContributor`
- `CompletionProvider`
- `CompletionParameters`
- `CompletionResultSet`
- `LookupElementBuilder`

## Annotators vs inspections vs intentions

### Annotator

Use for lightweight, often on-the-fly highlighting and inline diagnostics. Register via `com.intellij.annotator` EP with `language` attribute. Annotators can implement `DumbAware` to run during indexing (since 2023.1).

### External annotator

Use `ExternalAnnotator` when validation depends on an external tool (linter, compiler). Runs in a separate phase after other annotators.

### Inspection

Use when the feature should live in the inspections framework, support severity/profile settings, and integrate naturally with code analysis UI.

### Intention

Use for explicit editor actions offered to the user when a context applies.

### Quick fix

Use as a remediation attached to an inspection/annotation problem.

## Local inspection pattern

- extend `LocalInspectionTool` or a language-specific inspection base
- implement a PSI visitor
- register problems with `ProblemsHolder`
- attach `LocalQuickFix` when needed

## Intention pattern

- extend `PsiElementBaseIntentionAction`
- implement `isAvailable()` with cheap checks
- implement `invoke()` with safe write logic

## Quick-fix pattern

- implement `LocalQuickFix`
- override `applyFix(Project, ProblemDescriptor)` and `getFamilyName()`
- use `ProblemDescriptor.getPsiElement()` to get the element to fix
- keep fix text user-readable
- perform PSI changes in write command

## Language-support feature map

If the user asks for any of these, you likely need the matching extension point(s):

- syntax coloring → syntax highlighter
- red/yellow diagnostics → annotator / inspection
- auto-complete → completion contributor
- Ctrl+B navigation → references / target resolution
- documentation popup → documentation provider
- find usages → find usages provider
- rename support → references + rename handling
- code folding → folding builder
- structure panel → structure view factory
- formatting → formatter / block model
- comment toggling → commenter

## Kotlin-specific notes

- Do not use `object` for extension-point implementations (exception: `FileType` is allowed as `object`)
- Do not use companion objects with `@JvmStatic` in EP implementations in plugins (use top-level or check inspection)
- Prefer classes with lightweight constructors
- Prefer coroutine-based read/write APIs on recent platform versions

## Dumb mode note

Some extension points can be `DumbAware`, but only do this if they avoid index-backed work or are documented safe in dumb mode.

## When to also read other references

- Read `extension_points.md` for exact registration syntax
- Read `psi-and-indexing.md` for references, PSI trees, and index-backed resolution
- Read `code_samples.md` for `simple_language_plugin`, inspections, live templates, and intention examples
