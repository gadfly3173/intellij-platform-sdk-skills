# intellij-platform-sdk-skills

A standalone skills repository for IntelliJ Platform plugin development.

This repository currently provides one production-ready skill:

- `skills/intellij-platform-sdk` — a structured skill for developing plugins with the modern IntelliJ Platform v2 SDK / IntelliJ Platform Gradle Plugin 2.x.

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

## Included skill

### `intellij-platform-sdk`

This skill is designed for tasks such as:

- creating IntelliJ Platform plugins
- configuring `build.gradle.kts`, `settings.gradle.kts`, and `plugin.xml`
- implementing actions, services, tool windows, settings, and notifications
- working with PSI, references, indexing, VFS, and dumb mode
- building custom language support
- implementing completion, inspections, intentions, and quick fixes
- testing plugins and preparing Marketplace publication
- handling platform compatibility and migration to Gradle Plugin 2.x

The skill is intentionally organized as:

- a lightweight `SKILL.md` entrypoint for triggering and routing
- topic-focused `references/` files for detailed guidance

This keeps the main skill file small and makes the repository easier to maintain.

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

## Source material

This repository is based on the official IntelliJ Platform SDK documentation and related JetBrains samples.

## License

MIT

## Author

Gadfly
