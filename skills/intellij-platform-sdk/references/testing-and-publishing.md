# Testing and Publishing

Use this reference when the user needs test fixtures, parser tests, code-insight tests, plugin verification, signing, or Marketplace release steps.

## What this file covers

- light and heavy tests
- fixture-based code insight testing
- parser/completion/inspection tests
- plugin verifier
- signing and publishing

## Testing strategy

Choose the lightest test type that still matches the feature.

### Common test bases

- `BasePlatformTestCase` — fixture-based, generally preferred for light tests
- `LightPlatformTestCase` — alternative for non-fixture light tests
- `LightJavaCodeInsightFixtureTestCase` — Java-specific (JUnit 3); also available as `LightJavaCodeInsightFixtureTestCase4` (JUnit 4), `LightJavaCodeInsightFixtureTestCase5` (JUnit 5)
- `HeavyPlatformTestCase` — full project/platform environment
- `IdeaTestFixtureFactory` — manual fixture creation (not tied to a base class)

## When to use what

- PSI/editor feature in a lightweight environment → light fixture test
- Java-specific editor/code insight behavior → `LightJavaCodeInsightFixtureTestCase`
- parser snapshots / PSI tree expectations → extend `ParsingTestCase` from `com.intellij.testFramework`
- full project / heavier platform behavior → `HeavyPlatformTestCase`

## Typical test flows

### Inspection test

- configure file text or test data file
- enable inspection
- run highlighting
- assert messages/ranges/fixes

### Completion test

- configure caret position
- invoke completion
- inspect lookup items or inserted result

### Rename / usages test

- place caret on symbol
- invoke rename/find usages
- assert affected files or results

### Parser test

- load `.simple`/custom-language input
- compare produced PSI tree to expected output

## Test-data conventions

Keep stable fixtures under `src/test/testData` and prefer readable before/after files for inspections, intentions, and formatting behavior. Naming convention: `*.java` and `*.after.java` pairs for before/after comparisons. Use XML-like markup tags like `<warning descr="...">code</warning>` in test files for highlighting assertions.

## Verifier and compatibility checks

Always consider plugin verification when the user asks about shipping or IDE-version support.

```bash
./gradlew verifyPlugin    # 2.x Gradle Plugin
./gradlew runPluginVerifier  # 1.x Gradle Plugin (different task name!)
```

If multiple IDE versions matter, verify against all intended targets.

## Signing and publishing

Modern plugin delivery usually involves:

1. build plugin
2. sign plugin
3. verify plugin
4. publish to JetBrains Marketplace

Typical tasks:

```bash
./gradlew buildPlugin
./gradlew signPlugin
./gradlew verifyPlugin
./gradlew publishPlugin
```

## Signing inputs

Configure signing and publishing in `build.gradle.kts` using the 2.x DSL:

```kotlin
intellijPlatform {
    signing {
        certificateChain = providers.environmentVariable("CERTIFICATE_CHAIN")
        // or: certificateChainFile = file("/path/to/chain.crt")
        privateKey = providers.environmentVariable("PRIVATE_KEY")
        // or: privateKeyFile = file("/path/to/private.pem")
        password = providers.environmentVariable("PRIVATE_KEY_PASSWORD")
    }
    publishing {
        token = providers.environmentVariable("PUBLISH_TOKEN")
        // token is a JetBrains Marketplace Personal Access Token (from Marketplace profile > My Tokens)
    }
}
```

Do not hardcode secrets into Gradle files or committed properties. Use environment variables or Gradle properties passed at build time.

## Release guidance

Before release, verify:

- **First-time upload must be manual** — the first plugin publication to Marketplace cannot be automated
- `sinceBuild` / `untilBuild` are sensible
- required dependencies are correct
- optional integrations degrade gracefully
- tests cover the changed behavior
- verifier is clean or warnings are understood

## When to also read other references

- Read `compatibility.md` for version migration or build-range advice
- Read `troubleshooting.md` for failing tests, verifier warnings, or signing problems
- Read `code_samples.md` for sample-based test and publishing patterns
