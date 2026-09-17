# iOS Skills

A compact collection of reusable Codex skills for iOS engineering work.

## Included skills

### iPhone Duo Readiness Audit

`iphone-duo-readiness-audit` performs a read-only assessment of an iOS codebase for iPhone Duo and dynamic-resizing risks. It produces a prioritized remediation plan with evidence, root causes, rollout guidance, and a runtime validation matrix.

Use it when planning a foldable-iPhone transition, auditing adaptive layouts, or modernizing a legacy SwiftUI/UIKit codebase. It deliberately does not implement changes unless you separately ask it to.

## Install

Install this repository as a personal Codex skill source, then invoke the skill by name:

```text
$iphone-duo-readiness-audit audit this iOS codebase for resizability risks
```

The skill includes an entry-point `SKILL.md`, a focused risk catalog, and optional Codex UI metadata.

## Contributing

Keep each skill in its own lowercase, hyphenated directory with a `SKILL.md` entry point. Add supporting references only when they change an agent's decisions or prevent a known failure mode.

## License

[MIT](LICENSE)
