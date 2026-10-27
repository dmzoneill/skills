# skills

A collection of agent skills for AI coding assistants. Each skill is a
`SKILL.md` file with a name, a description of when to use it and the steps to
follow.

## Contents

There is a folder for each skill. They cover:

- Jira and GitLab work: creating and closing issues, merge requests, reviews and
  sprint planning.
- CI and deployments: CI health, pipeline retries, ephemeral environments,
  releases and rollouts.
- Investigating alerts, production issues and databases.
- Security: CVE fixes, vulnerability scans and secret checks.
- Daily routines such as standup summaries, weekly summaries and an end-of-day
  wrap-up (`beer`).
- Memory and knowledge management, slide decks and VM labs.

`SKILL.md` at the top level is the CVE fix skill, and `references` holds shared
reference material.

## Usage

Copy or link the skill folders into your assistant's skills directory. Many
skills expect particular MCP tools, which each `SKILL.md` lists.

## License

Licensed under the Apache License 2.0. See [LICENSE](LICENSE).
