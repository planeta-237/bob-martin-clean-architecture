# Bob Martin Clean Architecture Skill

A Codex skill for designing, reviewing, and explaining Robert C. Martin's
canonical layered Clean Architecture.

The skill targets the layered/concentric model:

- Entities
- Use Cases / Interactors
- Interface Adapters
- Frameworks & Drivers
- Main / Composition Root

It is intentionally focused on this model rather than a broad architecture
umbrella. Use it when you want an agent to design canonical Clean Architecture,
review dependency direction and boundaries, score architecture fitness, or teach
the concepts with concrete examples.

For Python projects, the skill includes Pydantic v2 guidance for boundary DTOs,
validation, serialization, settings, strict mode, `TypeAdapter`, and keeping
Pydantic from becoming the domain model by default.

## Install

Copy the skill folder into your Codex skills directory:

```bash
cp -R bob-martin-clean-architecture ~/.codex/skills/
```

Then invoke it with:

```text
Use $bob-martin-clean-architecture to design a Clean Architecture structure for ...
```

## Contents

- `bob-martin-clean-architecture/SKILL.md`: main skill instructions.
- `bob-martin-clean-architecture/agents/openai.yaml`: UI metadata.
- `bob-martin-clean-architecture/references/`: detailed references loaded as needed.

## Release Archive

The source folder is the primary artifact. A `.skill` archive can be created for
distribution:

```bash
zip -r -FS bob-martin-clean-architecture.skill bob-martin-clean-architecture -x '*/.DS_Store'
```
