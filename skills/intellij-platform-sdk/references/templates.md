# Live Templates, File Templates, and Postfix Completion

Use this reference when the user needs to provide live templates, file templates, custom template functions, postfix completion templates, or surround-with support in a plugin.

## What this file covers

- live templates: XML-based and programmatic provisioning
- custom macro/functions for live templates
- file and code templates (Velocity-based)
- postfix completion templates
- surround-with patterns

## Live Templates

### Providing live templates via XML

The simplest way to ship live templates is to bundle template definition files in the plugin. Place `.xml` template files in a directory (e.g., `src/main/resources/liveTemplates/`) and register them:

```xml
<extensions defaultExtensionNs="com.intellij">
    <defaultLiveTemplates file="liveTemplates/my_templates.xml"/>
</extensions>
```

### Live template XML format

```xml
<templateSet group="myLanguage">
    <template name="fori"
              value="for (int $INDEX$ = 0; $INDEX$ &lt; $LIMIT$; $INDEX$++) {
  $END$
}"
              description="Iterate with index"
              toReformat="true"
              toShortenFQNames="true">
        <variable name="INDEX" expression="suggestIndexName()" defaultValue="i" alwaysStopAt="true"/>
        <variable name="LIMIT" expression="" defaultValue="" alwaysStopAt="true"/>
        <context>
            <option name="JAVA_STATEMENT" value="true"/>
        </context>
    </template>
</templateSet>
```

Key attributes:
- `name` — template abbreviation
- `value` — expanded template body with `$VAR$` placeholders
- `toReformat` — auto-format expanded text
- `toShortenFQNames` — shorten fully qualified names
- `$END$` — final caret position after all variables are filled

### Available variable expressions

Built-in expressions available in template variables:
- `suggestIndexName()` — suggests `i`, `j`, `k`
- `suggestVariableName()` — suggests name based on type
- `className()` — current class name
- `methodName()` — enclosing method name
- `fileName()` — file name without extension
- `user()` — current user name
- `date()` / `time()` — current date/time
- `clipboard()` — clipboard contents
- `complete()` / `completeSmart()` — invoke code completion at cursor
- `enum("val1", "val2", ...)` — enumerated input with dropdown

### Custom template functions (macros)

When built-in expressions are insufficient, implement custom macros:

```java
public class MyMacro extends MacroBase {
    public MyMacro() {
        super("myMacro", "myMacro()");
    }

    @Override
    protected @Nullable Result calculateResult(
            @NotNull Expression[] params,
            ExpressionContext context,
            boolean quick) {
        Project project = context.getProject();
        if (project == null) return null;
        String result = computeValue(project);
        return result != null ? new TextResult(result) : null;
    }
}
```

Register as an extension:

```xml
<extensions defaultExtensionNs="com.intellij">
    <liveTemplateMacro implementation="com.example.MyMacro"/>
</extensions>
```

### Live templates configuration file

Previously, template configuration files were stored in an opaque IDE-specific format. Since the introduction of the live templates configuration file schema, templates can be described in a JSON-based format within the IDE configuration directory. Plugins that consume or generate template data should prefer the documented XML-based template descriptor format over the internal JSON format for portability.

## File and Code Templates

File templates use Apache Velocity for dynamic content generation. They can create individual or multiple files at once.

### Providing file templates

Place `.ft` (Velocity) template files in the plugin resources and register:

```xml
<extensions defaultExtensionNs="com.intellij">
    <internalFileTemplate name="My File"/>
</extensions>
```

Template files go under `src/main/resources/fileTemplates/internal/`:

```
src/main/resources/
├── fileTemplates/
│   └── internal/
│       └── My File.java.ft
```

### Velocity template variables

Standard variables available in templates:
- `${NAME}` — the user-entered name
- `${PACKAGE_NAME}` — target package
- `${PROJECT_NAME}` — project name
- `${USER}` — current user
- `${DATE}` / `${TIME}` — current date/time
- `${YEAR}` — current year
- `${MONTH}` / `${DAY}` — current month/day
- `${HOUR}` / `${MINUTE}` — current hour/minute
- `${DS}` — dollar sign (literal `$`)

### Example template

```velocity
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")
package ${PACKAGE_NAME};
#end

#parse("File Header.java")
public class ${NAME} {
    // ${NAME} implementation
}
```

### Custom file template properties

Use `FileTemplateGroupDescriptorFactory` to define custom properties:

```java
public class MyTemplatePropertiesProvider implements FileTemplateGroupDescriptorFactory {
    @Override
    public FileTemplateGroupDescriptor getFileTemplatesDescriptor() {
        FileTemplateGroupDescriptor group = new FileTemplateGroupDescriptor(
            "My Framework", AllIcons.FileTypes.Custom);
        group.addTemplate(new FileTemplateDescriptor(
            "My Entity.java", AllIcons.Nodes.Class));
        return group;
    }
}
```

Register:

```xml
<fileTemplateGroup implementation="com.example.MyTemplatePropertiesProvider"/>
```

### Using file templates programmatically

```java
FileTemplateManager manager = FileTemplateManager.getInstance(project);
FileTemplate template = manager.getInternalTemplate("My File.java");
Properties props = new Properties();
props.setProperty("PACKAGE_NAME", "com.example");
props.setProperty("CLASS_NAME", "Foo");

PsiDirectory dir = ...;
PsiFile file = FileTemplateUtil.createFromTemplate(
    template, "Foo", props, dir
);
```

Important: `FileTemplateUtil.createFromTemplate()` returns a `PsiElement`, not `PsiFile`. Cast when you know the result, and always create inside a write command.

## Postfix Completion

Postfix templates allow users to type an abbreviation after an expression and have the expression wrapped or transformed. They are scoped by language.

### Postfix template implementation

```java
public class MyPostfixTemplate extends PostfixTemplate {
    public MyPostfixTemplate() {
        super(
            null,           // id (null = auto-generated)
            "my",           // template key (what user types after ".")
            "Wrap as myBlock", // description
            "expr",         // example input
            myApplicabilityProvider() // when template applies
        );
    }

    @Override
    public void expand(
            @NotNull PsiElement context,
            @NotNull Editor editor) {

        // "context" is the topmost expression before the dot
        String text = context.getText();
        PsiElementFactory factory = PsiElementFactory.getInstance(context.getProject());
        // Build replacement PSI from template
        // Replace context with expanded form
    }
}
```

### Registering postfix templates

```java
public class MyPostfixTemplateProvider implements PostfixTemplateProvider {
    @Override
    public @NotNull Set<PostfixTemplate> getTemplates() {
        return Set.of(new MyPostfixTemplate());
    }

    @Override
    public @NotNull Presentation getPresentation() {
        return Presentation.getPresentablePresentation("myLanguage");
    }

    @Override
    public boolean isBuiltin(@NotNull PostfixTemplate template) {
        return true;
    }
}
```

Register:

```xml
<codeInsight.template.postfixTemplateProvider
    language="MyLanguage"
    implementationClass="com.example.MyPostfixTemplateProvider"/>
```

### Postfix template types

- `PostfixTemplate` — standard postfix template
- `PostfixLiveTemplate` — advanced postfix template with support for `$EXPR$` referencing the topmost expression (since 2024.2)
- `PostfixTemplateExpressionSelector` — controls which expressions the template applies to (based on type, PSI structure, etc.)

Use simple expression selectors when possible:
- `PostfixTemplateExpressionSelectorBase` for PSI-pattern-based selection
- `PostfixTemplatePsiInfo` for language-specific PSI type checking

## Surround With

For code-surrounding patterns, implement `SurroundDescriptor`:

EP: `com.intellij.lang.surroundDescriptor` with `language` attribute.

## Guidance

- Bundle live templates in plugin XML — it's the simplest distribution path
- Use built-in variable expressions before resorting to custom macros
- Keep custom macro implementations fast — they block template expansion
- For file templates, prefer the Velocity format (widely understood, user-editable in IDE settings)
- Postfix templates must check expression context carefully; a broken applicability check produces broken completions everywhere
- Keep postfix template applicability tests fast — they run on every keystroke after a dot
- When a postfix template creates a PSI element, prefer factory methods (e.g., `PsiElementFactory`) over raw text manipulation

## When to also read other references

- Read `extension_points.md` for exact XML registration
- Read `language-support-and-analysis.md` for surround-with and language-aware features
- Read `code_samples.md` for the `live_templates` sample
