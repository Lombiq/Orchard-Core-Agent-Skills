# Lombiq Orchard Core Agent Skills

## About

This repository contains [agent skills](https://agentskills.io/home) for [Orchard Core](https://orchardproject.net) development tasks. Orchard Core is an open-source, modular, and multi-tenant application framework and CMS built on .NET and ASP.NET Core. With these skills, you can use your favorite agent efficiently for common Orchard Core tasks.

Do you want something similar for Tailwind? Check out [Lombiq Tailwind Agent Skills](https://github.com/Lombiq/Tailwind-Agent-Skills)!

## Requirements

- Any agent that supports agent skills.
- For one-command installation with `npx skills add`, install Node.js (includes `npx`).
- The `orchard-core-theming` skill includes a Python sync script for refreshing generated references. To use it:
  - Install Python 3.8 or later (available on `PATH`).
  - Install `git` (available on `PATH`).
  - Enable internet access for the agent.
  - Ensure the installed skill folder is writable.

## Skills included

### `orchard-core-theming`

A playbook covering common Orchard Core theme development tasks, such as shape development and recipe management. It contains many agent-optimized markdown docs with guides and examples, as well as scripts for recurring operations.

#### What is in this skill
- Task-focused references for shape discovery, alternates, placement rules, and template implementation.
- Separate Razor and Liquid guidance with practical examples.
- Content-model access workflows (content definitions, field/part models, and sample content inspection).
- Recipe authoring patterns for content definitions, sample content, roles, users, and site settings.
- A sync script and maintenance workflow for keeping generated references up to date.

#### Highlights
- Creates and implements ad-hoc shape templates or override content templates using the correct alternates.
- Can use common tag helpers, Liquid filters, and `IOrchardHelper` extensions in templates.
- Reads content definitions via `ContentDefinition.json` (when `OrchardCore.Contents.FileContentDefinition` is enabled) or directly from the SQLite database, and can also pull the latest content items for inspection.
- Accesses all relevant content models for content items and fields, including parts like `BagPart` and `FlowPart` when the task requires them (e.g., implementing a content item display shape).
- Can manage recipes using common recipe steps for sample content items, content definitions, site settings, roles, and users (using commands).
- Includes a self-sync script to refresh the skill from the Lombiq/Orchard-Core-Agent-Skills repo.

#### Prompting tips and examples

- While the agent should know that it needs to use the skill, it's still a good idea to start prompts with "Use the Orchard Core agent skill to..." so the agent has a clear execution path.
- Be specific about what you want and even give it some implementation details; vague prompts fail more often.
- State where the agent should read source data (for example, recipe files, `ContentDefinition.json`, or SQLite).

A few examples:

```text
Use the Orchard Core agent skill to implement the detail shape template in Liquid for the BlogPost content type in the current theme. Use the correct alternate and render the title, tags, and body. Find the content definitions in the SQLite database.
```

```text
Use the Orchard Core agent skill to create a widget content type named TestimonialsWidget and a related item content type named TestimonialItem. In TestimonialsWidget, add TitlePart and a BagPart named Testimonials that allows only TestimonialItem entries. In TestimonialItem, add TextField Quote (required, multiline) and TextField Name (required). Add both content type definitions to the MySite.Contents recipe.
```

```text
Use the Orchard Core agent skill to create 3 sample BlogPost content items with realistic data and add them to the MySite.Contents recipe. The BlogPost content type is already defined in the recipe.
```

```text
Use the Orchard Core agent skill to update `placement.json` so Page detail display doesn't show the TitlePart shape.
```

#### Initialization and updates

Follow the [orchard-core-theming maintenance guide](maintenance/orchard-core-theming.md) for the step-by-step update workflow.
For refresh commands, see [Refreshing installed skills](#refreshing-installed-skills).

### `pomi-blocks`

A composition preference for building Orchard Core sites with Pomi. It replaces FlowPart/Widget page sections with an ordered `BagPart` using the `Blocks` editor and section content types with the `Block` stereotype, including nested repeated components, useful field placement and matching Liquid templates. It relies only on built-in Orchard Core features and lets Pomi handle the rest of the build.

Use it alongside Pomi when creating a website from a brief or supplied materials, or when designing editable pages and sections.

## Installing skills

The quickest way to install from this repository is to use the `skills` CLI, which detects and installs skills automatically.

```bash
npx skills add Lombiq/Orchard-Core-Agent-Skills
```

If you prefer manual installation, copy the `skills/` subfolders into your agent's skills directory:
- Project scope: `.github/skills/` (Copilot), `.codex/skills/` (Codex), `.claude/skills/` (Claude Code)
- Global scope: `~/.copilot/skills/`, `~/.codex/skills/`, `~/.claude/skills/` (or `%USERPROFILE%\\...\\skills` on Windows)

For GitHub Copilot in VS Code, Agent Skills are currently in preview and available only after enabling `chat.useAgentSkills`. See [the docs](https://docs.github.com/copilot/concepts/agents/about-agent-skills).

## Refreshing installed skills

You can ask your agent to refresh the installed `orchard-core-theming` skill, or run the command manually:

```text
python skills/orchard-core-theming/scripts/sync-skill.py
```

If you already have a local clone of the source repository, add `--local-repo <path>`:

```text
python skills/orchard-core-theming/scripts/sync-skill.py --local-repo D:\Repos\Lombiq\Orchard-Core-Agent-Skills
```

## Maintenance

See the [orchard-core-theming maintenance guide](maintenance/orchard-core-theming.md) for the step-by-step update workflow.

## Contributing

Bug reports, feature requests, comments, questions, code contributions and love letters are warmly welcome. You can send them to us via GitHub issues and pull requests. Please adhere to our [open-source guidelines](https://lombiq.com/open-source-guidelines) while doing so.

This project is developed by [Lombiq Technologies](https://lombiq.com/). Commercial-grade support is available through Lombiq.
