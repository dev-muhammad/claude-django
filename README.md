# Claude Django 🎸

> Django Developer plugin for Claude Code - Boost your Django productivity with intelligent commands, skills, and agents.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Claude Code](https://img.shields.io/badge/Claude_Code-Compatible-brightgreen)](https://claude.com/claude-code)
[![Django](https://img.shields.io/badge/Django-4.2%20%7C%205.2%20%7C%206.0-green)](https://www.djangoproject.com/)
[![Python](https://img.shields.io/badge/Python-3.12%2B%20to%203.14-blue)](https://www.python.org/)

## 📦 Installation

```bash
# Add the marketplace
/plugin marketplace add dev-muhammad/claude-django

# Install the plugin
/plugin install claude-django@claude-django-marketplace
```

Restart Claude Code to load the plugin.

## ✨ Features

### 🚀 Quick Commands
Slash commands for instant Django code generation:
- `/django:model` - Generate models with **composite primary keys** (Django 5.2+)
- `/django:view` - Create **async** function/class-based views (Django 5.1+)
- `/django:serializer` - Generate DRF serializers
- `/django:admin` - Configure admin registration
- `/django:url` - Set up URL patterns
- `/django:form` - Create forms and modelforms
- `/django:migrate` - Migration helpers
- `/django:test` - Generate test cases
- `/django:task` - Create **background tasks** (Django 6.0)
- `/django:csp` - Configure **Content Security Policy** (Django 6.0)
- `/django:partial` - Create **template partials** (Django 6.0)

### 🎯 Interactive Skills
Guided workflows for complex tasks:
- **create-app** - Scaffold Django app with composite PKs
- **create-project** - Start a new Django project with Django 6.0 features
- **setup-drf** - Configure Django REST Framework with async support
- **setup-auth** - Configure custom user model and async authentication
- **setup-tasks** - Set up Django 6.0 built-in background tasks
- **setup-csp** - Configure Django 6.0 Content Security Policy
- **migrate-to-async** - Migrate views to async (Django 5.1+)
- **migrate-to-composite-pk** - Migrate models to composite primary keys

### 🤖 Autonomous Agents
Smart helpers that work independently:
- **debug** - Analyze Django 6.0 errors and trace issues
- **optimize** - Query optimization, N+1 detection, async performance tuning
- **upgrade** - Help upgrade Django projects to latest version

## 🆕 Django 6.0 & Django 5.2 Features

This plugin fully supports both the latest and current Django versions:

| Feature | Django Version | Plugin Support |
|---------|---------------|----------------|
| **Background Tasks Framework** | 6.0+ | ✅ `/django:task` |
| **Built-in CSP Middleware** | 6.0+ | ✅ `/django:csp` |
| **Template Partials** | 6.0+ | ✅ `/django:partial` |
| **Composite Primary Keys** | 5.2+ | ✅ `/django:model` |
| **Async ORM Operations** | 5.1+ | ✅ `/django:view` |
| **Async Session API** | 5.1+ | ✅ `setup-auth` skill |

## 🎓 Usage

### Commands
```bash
# Generate a model (Django 5.2+ composite PK support)
/django:model

# Create an async view (Django 5.1+)
/django:view

# Create a background task (Django 6.0)
/django:task

# Configure CSP (Django 6.0)
/django:csp
```

### Skills
```bash
# Create a new Django app with composite PKs
Use the "create-app" skill when Claude asks what to help with

# Setup Django 6.0 background tasks
Use the "setup-tasks" skill for task framework configuration

# Migrate to async views
Use the "migrate-to-async" skill for async migration
```

## 🌐 Supported Versions

### Django Support (as of February 2026)

| Django | 4.2 | 5.0 | 5.1 | 5.2 | 6.0 |
|--------|------|------|------|------|------|
| Status | ✅   | ✅   | ✅   | ✅   | ✅   |

**Current Versions:**
- **Django 5.2.11** - Stable and mature (recommended for production)
- **Django 6.0.2** - Latest with new features (background tasks, CSP, partials)

### Python Support
| Python | Django 4.2 | Django 5.0–5.2 | Django 6.0 |
|--------|------------|----------------|------------|
| 3.10   | ✅         | ✅             | ❌         |
| 3.11   | ✅         | ✅             | ❌         |
| 3.12   | ✅         | ✅             | ✅         |
| 3.13   | ✅         | ✅             | ✅         |
| 3.14   | ❌         | ❌             | ✅         |

**Python 3.12+** is required for Django 6.0. Django 5.2 is the last version supporting Python 3.10/3.11.

### Which Django Version Should You Use?

**Use Django 5.2.11 () if:**
- ✅ Building a production application
- ✅ Need long-term stability (3 years security updates)
- ✅ Prefer proven, battle-tested features
- ✅ Don't need the absolute latest features

**Use Django 6.0.2 () if:**
- ✅ Need built-in background tasks (no Celery required)
- ✅ Want native CSP middleware for security
- ✅ Need template partials for component architecture
- ✅ Building cutting-edge applications
- ⚠️  Accept shorter support cycle (~8 months)

## 🎯 Key Features by Django Version

### Django 6.0.2  🆕
- ✅ Built-in background tasks framework (no Celery needed!)
- ✅ Native CSP middleware for security
- ✅ Template partials for component-based architecture
- ✅ Improved async ORM operations

### Django 5.2.11  ✅
- ✅ Composite primary keys support
- ✅ Automatic model imports in shell
- ✅ Simplified form field handling
- ✅ Recommended for production use

### Django 5.1
- ✅ Async SessionBase API
- ✅ Enhanced async view support

### Django 4.2 (Previous -)
- ⚠️ End of Life - upgrade to 5.2 or 6.0 recommended

## 📚 Examples

### Django 6.0 Background Task
```python
from django.core.mail import send_mail
from django.tasks import task

@task
def email_users(emails, subject, message):
    return send_mail(subject, message, None, emails)

# Enqueue for background execution
email_users.enqueue(
    emails=["user@example.com"],
    subject="You have a message",
    message="Hello there!",
)
```

### Django 5.2 Composite Primary Key
```python
class OrderItem(models.Model):
    order = models.ForeignKey('Order', on_delete=models.CASCADE, primary_key=True)
    product = models.ForeignKey('Product', on_delete=models.CASCADE, primary_key=True)
    quantity = models.IntegerField()
```

### Django 5.1 Async View
```python
import aiohttp

async def product_detail(request, pk):
    product = await Product.objects.aget(pk=pk)
    async with aiohttp.ClientSession() as session:
        async with session.get(f'https://api.example.com/stock/{pk}') as resp:
            stock = await resp.json()
    return render(request, 'product.html', {'product': product, 'stock': stock})
```

## 📝 Changelog

### v0.1.1 (February 2026) - Bug Fix
- Fixed `hooks.json` schema: changed `hooks` from array to record format to match Claude Code plugin specification

### v0.1.0 (February 2026) - Initial Release 🎉
**Django 6.0.2 () + Django 5.2.11 ()**

**Commands (11 total):**
- `/django:model` - Generate models with composite primary keys (Django 5.2+)
- `/django:view` - Create async function/class-based views (Django 5.1+)
- `/django:serializer` - Generate DRF serializers
- `/django:admin` - Configure admin registration
- `/django:url` - Set up URL patterns
- `/django:form` - Create Django forms
- `/django:migrate` - Migration helpers
- `/django:test` - Generate test cases
- `/django:task` - Create background tasks (Django 6.0) 🆕
- `/django:csp` - Configure CSP middleware (Django 6.0) 🆕
- `/django:partial` - Create template partials (Django 6.0) 🆕

**Skills (8 total):**
- `create-app` - Scaffold Django app with composite PKs
- `create-project` - Start Django project with Django 6.0 features
- `setup-drf` - Configure DRF with async support
- `setup-auth` - Configure custom user model and async authentication
- `setup-tasks` - Set up Django 6.0 background tasks framework 🆕
- `setup-csp` - Configure Django 6.0 Content Security Policy 🆕
- `migrate-to-async` - Migrate views to async (Django 5.1+) 🆕
- `migrate-to-composite-pk` - Migrate models to composite primary keys 🆕

**Agents (3 total):**
- `debug` - Analyze Django 6.0 errors including async debugging
- `optimize` - Query optimization, N+1 detection, async performance tuning
- `upgrade` - Help upgrade Django projects to latest version 🆕

**Hooks (1 total):**
- `validate-model` - Validate Django model definitions

**Supported Versions:**
- Django: 4.2, 5.0, 5.1, **5.2 ()**, **6.0 ()**
- Python: 3.12, 3.13, 3.14 (Django 6.0); 3.10+ (Django 4.2–5.2)

**Documentation:**
- Comprehensive README with Django vs guide
- MIT License
- Contributing guidelines

---

## 🚀 Future Plans

This plugin is actively maintained and growing. Here's what we're planning:

### Upcoming Commands
| Command | Description | Status |
|---------|-------------|--------|
| `/django:signal` | Generate Django signals with best practices | 📋 Planned |
| `/django:middleware` | Create custom middleware with async support | 📋 Planned |
| `/django:management` | Generate custom management commands | 📋 Planned |
| `/django:template` | Create template with block inheritance | 📋 Planned |
| `/django:cache` | Configure caching strategies (Redis, Memcached) | 📋 Planned |
| `/django:api-viewset` | Generate DRF ViewSets with CRUD + actions | 📋 Planned |

### Upcoming Skills
| Skill | Description | Status |
|-------|-------------|--------|
| **setup-htmx** | Configure HTMX with Django 6.0 template partials | 📋 Planned |
| **setup-websocket** | Django Channels + WebSocket setup | 📋 Planned |
| **setup-redis** | Redis caching and session backend | 📋 Planned |
| **setup-testing** | Pytest + factory-boy + coverage setup | 📋 Planned |
| **setup-docker** | Docker + docker-compose configuration | 📋 Planned |
| **setup-celery** | Legacy Celery setup (for Django < 6.0) | 📋 Planned |
| **create-api** | Scaffold REST API from models (DRF) | 📋 Planned |
| **create-graphql** | GraphQL API setup with Strawberry/Graphene | 📋 Planned |

### Upcoming Agents
| Agent | Description | Status |
|--------|-------------|--------|
| **refactor** | Identify code smells and suggest refactoring | 📋 Planned |
| **security** | Scan for security vulnerabilities (OWASP Top 10) | 📋 Planned |
| **test-generator** | Auto-generate tests from existing code | 📋 Planned |
| **doc-generator** | Generate API docs from code | 📋 Planned |

### Enhancements
- [ ] **Django 6.1+** - Support for upcoming Django 6.1 features
- [ ] **Python 3.14+** - Support for latest Python features
- [ ] **DRF Integration** - Deeper DRF ViewSet and router support
- [ ] **Type Stubs** - Add type hints for better IDE support
- [ ] **Performance Profiling** - Built-in profiling agent
- [ ] **CI/CD Templates** - GitHub Actions, GitLab CI workflows
- [ ] **Testing Framework** - Automated plugin testing

### Community Contributions Welcome!
We'd love help with:
- Documentation improvements
- Bug fixes and edge cases
- Additional language support
- Framework integrations (Next.js, Vue, React)
- Real-world usage examples

**Have an idea?** Open an issue or PR! We're friendly to new contributors.

## 🤝 Contributing

Contributions are welcome! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- Built for [Claude Code](https://claude.com/claude-code)
- Inspired by the Django community
- Supports latest Django innovations

## 📚 Plugin Documentation

The plugin includes comprehensive reference documentation for Django development best practices:

### Included Reference Guides

| Document | Path | Coverage |
|----------|------|----------|
| **Coding Standards** | `.claude/plugins/claude-django/docs/standards.md` | Naming conventions, project structure, model/view/URL standards, testing guidelines |
| **Best Practices** | `.claude/plugins/claude-django/docs/best-practices.md` | Security, performance, deployment, DRF API patterns, database design |
| **Large Project Patterns** | `.claude/plugins/claude-django/docs/large-project-patterns.md` | Per-class file organization, base models, custom QuerySets, scalability |

`★ Insight ─────────────────────────────────────`
These reference documents are automatically loaded into context when using Django plugin commands. This ensures all generated code follows consistent, production-ready standards—no need to manually reference best practices during development.
`─────────────────────────────────────────────────`

### Quick Access

The docs folder is located at:
```
.claude/plugins/claude-django/docs/
├── README.md           # This overview
├── standards.md        # Coding standards & conventions
├── best-practices.md   # Security, performance & deployment
└── large-project-patterns.md  # Enterprise-scale patterns
```

## 📖 Resources

- [Django 5.2 Documentation](https://docs.djangoproject.com/en/5.2/)
- [Django 6.0 Documentation](https://docs.djangoproject.com/en/6.0/)
- [Django 6.0 Release Notes](https://docs.djangoproject.com/en/6.0/releases/6.0/)
- [Django 5.2 Release Notes](https://docs.djangoproject.com/en/5.2/releases/5.2/)
- [Django REST Framework](https://www.django-rest-framework.org/)
- [Django Versions Policy](https://docs.djangoproject.com/en/6.0/internals/release-process/)

---

Made with ❤️ for open-source community
