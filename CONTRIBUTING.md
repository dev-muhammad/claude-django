# Contributing to claude-django

Thank you for your interest in contributing to claude-django! This document provides guidelines and instructions for contributing.

## 🚀 Getting Started

### Prerequisites
- Claude Code installed and working
- Basic understanding of Django
- Familiarity with YAML frontmatter (for commands/skills)
- Python 3.12+ and Django 4.2+ for testing

### Development Setup

```bash
# Fork and clone the repository
git clone https://github.com/dev-muhammad/claude-django.git
cd claude-django
```

**Testing the Plugin**
```bash
# Run Claude with plugin from current directory
claude --plugin-dir .claude/plugins/claude-django
```
This loads the plugin directly from your repository—perfect for rapid iteration without affecting your global plugin installation.

## 📁 Project Structure

```
claude-django/
├── .claude/plugins/claude-django/
│   ├── plugin.json          # Plugin manifest
│   ├── commands/            # Slash commands (.md files)
│   ├── skills/              # Interactive skills (.md files)
│   ├── agents/              # Autonomous agents (.md files)
│   ├── hooks/               # Event hooks (.md files)
│   └── docs/                # Reference documentation
├── README.md
├── LICENSE
└── CONTRIBUTING.md
```

## 🤝 How to Contribute

### 1. Adding a New Command

Create a new file in `commands/` directory:

```markdown
---
name: django:mycommand
description: Brief description of what it does
---

Your command prompt and instructions here...
```

### 2. Adding a New Skill

Create a new file in `skills/` directory:

```markdown
---
name: my-skill
description: What this skill helps with
---

Interactive workflow instructions...
```

### 3. Adding an Agent

Create a new file in `agents/` directory:

```markdown
---
name: my-agent
description: What this agent does
color: ff5722
---

Agent system prompt and capabilities...
```

### 4. Adding a Hook

Create a new file in `hooks/` directory:

```markdown
---
name: my-hook
events: [PreToolUse, PostToolUse]
description: What this hook monitors
---

Hook logic and validation...
```

## 📝 Commit Guidelines

- Use clear, descriptive commit messages
- Prefix commits with type: `feat:`, `fix:`, `docs:`, `refactor:`, `test:`
  - `feat:` - New features
  - `fix:` - Bug fixes
  - `docs:` - Documentation changes
  - `refactor:` - Code refactoring
  - `test:` - Adding/updating tests

Example: `feat(commands): add django:signal command for signal generation`

## 🧪 Testing

Before submitting PR:
1. Test your changes in Claude Code
2. Verify all commands/skills work as expected
3. Check for edge cases
4. Update documentation if needed

## 📧 Pull Request Process

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Make your changes
4. Commit your changes (`git commit -m 'feat: add amazing feature'`)
5. Push to the branch (`git push origin feature/amazing-feature`)
6. Open a Pull Request

### PR Checklist
- [ ] Code follows the project structure
- [ ] Commit messages follow guidelines
- [ ] Documentation is updated
- [ ] Tested in Claude Code
- [ ] No breaking changes (or documented if present)

## 🐛 Bug Reports

When reporting bugs, include:
- Django version
- Python version
- Steps to reproduce
- Expected vs actual behavior
- Error messages or logs

## 💡 Feature Requests

We welcome feature requests! Please include:
- Clear description of the feature
- Use cases and benefits
- Possible implementation approach (if known)

## 🎨 Code Style

- Use YAML frontmatter for all commands/skills/agents
- Keep descriptions concise and clear
- Use progressive disclosure for complex features
- Provide examples where helpful

## 📜 Code of Conduct

Be respectful, constructive, and inclusive. We're all here to build something great together.

---

Thank you for contributing! 🎉
