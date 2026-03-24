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

- `BasePlatformTestCase`
- `LightPlatformTestCase`
- `LightJavaCodeInsightFixtureTestCase`
- `HeavyPlatformTestCase`
- `ParsingTestCase`

## When to use what

- PSI/editor feature in a lightweight environment → light fixture test
- Java-specific editor/code insight behavior → `LightJavaCodeInsightFixtureTestCase`
- parser snapshots / PSI tree expectations → `ParsingTestCase`
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

Keep stable fixtures under `src/test/testData` and prefer readable before/after files for inspections, intentions, and formatting behavior.

## Verifier and compatibility checks

Always consider plugin verification when the user asks about shipping or IDE-version support.

```bash
./gradlew verifyPlugin
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

Common environment-backed inputs:

- certificate chain
- private key
- private key password
- Marketplace publish token

Do not hardcode secrets into Gradle files or committed properties.

## Release guidance

Before release, verify:

- `sinceBuild` / `untilBuild` are sensible
- required dependencies are correct
- optional integrations degrade gracefully
- tests cover the changed behavior
- verifier is clean or warnings are understood

## When to also read other references

- Read `compatibility.md` for version migration or build-range advice
- Read `troubleshooting.md` for failing tests, verifier warnings, or signing problems
- Read `code_samples.md` for sample-based test and publishing patterns
