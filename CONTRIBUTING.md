# Contributing to Zhin AI Skills

Thank you for your interest in contributing to the Zhin AI Skills repository! This guide will help you add new skills or improve existing ones.

## What are Agent Skills?

Agent Skills follow the [Agent Skills specification](https://agentskills.io/specification) - a simple, open format for giving AI agents new capabilities and expertise.

## How to Contribute

### Adding a New Skill

1. **Create a skill directory**
   ```bash
   mkdir skills/your-skill-name
   ```
   - Use lowercase letters, numbers, and hyphens only
   - Name must be 1-64 characters
   - Must not start or end with hyphen
   - Must not contain consecutive hyphens

2. **Create SKILL.md file**
   
   Every skill requires a `SKILL.md` file with:
   - YAML frontmatter (metadata)
   - Markdown instructions

   Example:
   ```yaml
   ---
   name: your-skill-name
   description: A clear description of what this skill does and when to use it (max 1024 chars).
   license: MIT
   metadata:
     author: your-name
     version: "1.0"
   ---

   # Your Skill Name

   [Add instructions here that help AI agents use this skill effectively]
   ```

3. **Optional: Add supporting files**
   
   You can include:
   - `scripts/` - Executable code (Python, Bash, JavaScript, etc.)
   - `references/` - Additional documentation (REFERENCE.md, etc.)
   - `assets/` - Templates, images, or data files

4. **Follow best practices**
   
   - Keep `SKILL.md` under 500 lines (move details to reference files)
   - Write clear, actionable instructions
   - Include examples of inputs and outputs
   - Document edge cases and troubleshooting
   - Test your skill with an AI agent before submitting

### Required Fields

Your `SKILL.md` frontmatter must include:

- `name`: Must match the directory name (lowercase, hyphens, 1-64 chars)
- `description`: What the skill does and when to use it (1-1024 chars)

### Optional Fields

- `license`: License name or reference to bundled license file
- `compatibility`: Environment requirements (if any)
- `metadata`: Additional key-value pairs (author, version, etc.)
- `allowed-tools`: Space-delimited list of pre-approved tools (experimental)

## Skill Guidelines

### Good Description Examples

✅ Good:
```yaml
description: Extracts text and tables from PDF files, fills PDF forms, and merges multiple PDFs. Use when working with PDF documents or when the user mentions PDFs, forms, or document extraction.
```

❌ Poor:
```yaml
description: Helps with PDFs.
```

### Progressive Disclosure

Structure your skill for efficient context usage:

1. **Metadata** (~100 tokens): Name and description loaded at startup
2. **Instructions** (< 5000 tokens recommended): Full SKILL.md loaded when activated
3. **Resources** (as needed): Additional files loaded only when required

## Testing Your Skill

Before submitting:

1. Test with an AI agent (Claude, GPT-4, etc.)
2. Verify instructions are clear and actionable
3. Check that examples work as expected
4. Ensure all file paths use relative references from skill root

## Submitting Your Contribution

1. Fork the repository
2. Create a new branch (`git checkout -b add-skill-name`)
3. Add your skill following the guidelines above
4. Commit your changes (`git commit -m 'Add [skill-name] skill'`)
5. Push to your branch (`git push origin add-skill-name`)
6. Open a Pull Request

### Pull Request Guidelines

- Provide a clear description of what your skill does
- Explain the use cases it addresses
- Include any testing you've done
- Link to any relevant documentation or examples

## Code of Conduct

- Be respectful and constructive
- Follow the Agent Skills specification
- Help review other contributions
- Report issues clearly with reproduction steps

## Questions?

- Check the [Agent Skills documentation](https://agentskills.io)
- Review existing skills in this repository for examples
- Open an issue for questions or suggestions

## License

By contributing, you agree that your contributions will be licensed under the same license as this repository. Individual skills may specify their own licenses.
