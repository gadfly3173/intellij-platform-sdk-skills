# intellij-platform-sdk-skills

A standalone skills repository for IntelliJ Platform plugin development.

This repository currently contains one focused skill:

- `skills/intellij-platform-sdk` — a structured skill for working with the modern IntelliJ Platform v2 SDK / IntelliJ Platform Gradle Plugin 2.x.

The skill is designed to help with:

- creating IntelliJ Platform plugins
- configuring `build.gradle.kts`, `settings.gradle.kts`, and `plugin.xml`
- implementing actions, services, tool windows, settings, and notifications
- working with PSI, references, indexing, VFS, and dumb mode
- building custom language support
- implementing completion, inspections, intentions, and quick fixes
- testing plugins and preparing Marketplace publication
- handling platform compatibility and migration to Gradle Plugin 2.x

## Why this repository exists

This repository is organized as a skills-first repository rather than a traditional application codebase.

The included skill is intentionally split into:

- a lightweight `SKILL.md` entrypoint for triggering and routing
- topic-focused `references/` files for detailed guidance

This keeps the main skill file small, improves maintainability, and makes it easier to load only the material that is relevant to the current task.

## Repository layout

```text
intellij-platform-sdk-skills/
├── README.md
├── LICENSE
├── .gitignore
└── skills/
    └── intellij-platform-sdk/
        ├── SKILL.md
        └── references/
            ├── getting-started.md
            ├── platform-basics.md
            ├── psi-and-indexing.md
            ├── language-support-and-analysis.md
            ├── ui-settings-and-toolwindows.md
            ├── testing-and-publishing.md
            ├── compatibility.md
            ├── extension_points.md
            ├── code_samples.md
            ├── patterns.md
            ├── troubleshooting.md
            ├── troubleshooting-build-runtime.md
            └── troubleshooting-psi-ui-testing.md
```

## Installation

You can install this repository or the individual skill with `npx skills add`.

### Install from GitHub repository

```bash
npx skills add gadfly3173/intellij-platform-sdk-skills
```

Equivalent forms also work:

```bash
npx skills add https://github.com/gadfly3173/intellij-platform-sdk-skills
npx skills add git@github.com:gadfly3173/intellij-platform-sdk-skills.git
```

### Install only the IntelliJ Platform skill

```bash
npx skills add gadfly3173/intellij-platform-sdk-skills --skill intellij-platform-sdk
```

### Install to a specific agent

```bash
npx skills add gadfly3173/intellij-platform-sdk-skills -a claude-code
```

### Install globally

```bash
npx skills add gadfly3173/intellij-platform-sdk-skills -g
```

### Install from a local checkout

```bash
npx skills add ./skills/intellij-platform-sdk
```

### Manual installation

Copy this directory into your local skills directory:

```text
skills/intellij-platform-sdk
```

Typical destinations include:

- `.claude/skills/`
- `.agents/skills/`
- `~/.claude/skills/`
- any compatible skills directory that supports `SKILL.md`

## Sources

This repository is based on the official JetBrains IntelliJ Platform documentation and related sample material.

- Official documentation: https://plugins.jetbrains.com/docs/intellij/
- Official documentation repository: https://github.com/JetBrains/intellij-sdk-docs

## Notes

This repository was written based on JetBrains documentation, with Kimi K2.5 and GPT-5.4 included in the writing workflow.

## License

MIT

## Author

Gadfly
