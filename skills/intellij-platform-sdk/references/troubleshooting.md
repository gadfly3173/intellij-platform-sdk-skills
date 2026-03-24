# Troubleshooting Index

Use this file as a routing guide instead of loading every troubleshooting note at once.

## Read `troubleshooting-build-runtime.md` when the issue is about

- Gradle sync or dependency resolution
- v2 SDK setup problems
- class loading / runtime wiring
- action visibility or registration
- service initialization
- verifier warnings
- signing or release failures

## Read `troubleshooting-psi-ui-testing.md` when the issue is about

- PSI validity or write-action assertions
- `IndexNotReadyException`
- editor/document coordination
- UI refresh or EDT errors
- memory leaks / disposables
- fixture setup and test data
- local debugging inside sandbox IDE

## Quick heuristics

- build starts failing before IDE launch → build/runtime troubleshooting
- plugin loads but behavior fails at PSI/editor/UI time → PSI/UI/testing troubleshooting
- release or compatibility warning → build/runtime troubleshooting first
