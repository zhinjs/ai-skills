# Zhin AI Skills

A collection of [Agent Skills](https://agentskills.io) for the Zhin chatbot framework.

## About

This repository contains skills that enhance Zhin chatbots with AI agent capabilities. Skills are organized according to the Agent Skills specification, making them discoverable and usable by AI agents.

## What are Agent Skills?

[Agent Skills](https://agentskills.io) are a simple, open format for giving agents new capabilities and expertise. They are folders of instructions, scripts, and resources that agents can discover and use to perform better at specific tasks.

## Repository Structure

```
ai-skills/
├── skills/           # Individual skills directory
│   └── [skill-name]/ # Each skill in its own directory
│       └── SKILL.md  # Skill definition (required)
├── packages/         # Best practices and reference material
│   └── [package-name]/
├── README.md         # This file
└── .gitignore
```

## Using Skills

Skills in this repository follow the [Agent Skills specification](https://agentskills.io/specification). Each skill:

- Has a unique name following the format: `[a-z0-9-]+`
- Contains a `SKILL.md` file with YAML frontmatter and instructions
- May include optional directories: `scripts/`, `references/`, and `assets/`

## About Zhin

[Zhin](https://github.com/zhinjs/zhin) is a chat bot framework for Node.js developers, compatible with QQ, ICQQ, WeChat, Discord, OneBot (11/12), DingTalk, and more.

## Contributing

Contributions are welcome! To add a new skill:

1. Create a new directory under `skills/` with your skill name (lowercase, hyphens only)
2. Add a `SKILL.md` file following the [specification](https://agentskills.io/specification)
3. Test your skill with an AI agent
4. Submit a pull request

## License

This repository is part of the Zhin ecosystem. Individual skills may have their own licenses.
