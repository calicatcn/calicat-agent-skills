# Repository Guidance

This repository packages Calicat-related skills for the open agent skills ecosystem.

## Structure Rules

- Every installable skill lives under `skills/<skill-name>/`.
- Each skill must include `SKILL.md` with valid YAML frontmatter.
- Optional files should stay inside the skill folder, typically under `references/`, `scripts/`, or `assets/`.
- Keep `SKILL.md` focused on trigger conditions, workflow, and essential rules.
- Move lower-frequency material into `references/` to preserve progressive disclosure.

## Naming Rules

- Repository name can be broader than a single skill.
- Skill names must stay lowercase and hyphenated.
- Prefer specific skill names such as `calicat-cli-operator` over generic names like `calicat-cli-skill`.

## Maintenance Rules

- If Calicat URL forms change, update `skills/calicat-cli-operator/SKILL.md` first.
- If Calicat CLI install or update behavior changes, update `skills/calicat-cli-operator/references/cli-lifecycle.md`.
- Keep examples realistic and copy-pasteable.
