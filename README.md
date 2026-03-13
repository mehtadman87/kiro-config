# Kiro Best Practices Boilerplate

A comprehensive collection of steering documents and agent hooks for Kiro IDE that enforces development best practices, automates quality checks, and streamlines workflows.

## 🚀 Quick Start

### Option 1: Clone as Template

```bash
# Clone this repository as a template for your new project
git clone https://github.com/mehtadman87/kiro-config.git your-project-name
cd your-project-name
rm -rf .git
git init
git add .
git commit -m "feat: initialize project with Kiro best practices"
```

### Option 2: Add to Existing Project (Recommended)

```bash
# Add the repo contents as your .kiro directory
cd your-existing-project
mkdir -p .kiro
curl -L https://github.com/mehtadman87/kiro-config/archive/main.tar.gz | tar -xz
cp -r kiro-config-main/hooks kiro-config-main/settings kiro-config-main/specs kiro-config-main/steering .kiro/
rm -rf kiro-config-main
```

### Option 3: Manual Download

```bash
# Download and set up .kiro directory
cd your-project
mkdir -p .kiro
curl -L https://github.com/mehtadman87/kiro-config/archive/main.tar.gz | tar -xz
cp -r kiro-config-main/hooks kiro-config-main/settings kiro-config-main/specs kiro-config-main/steering .kiro/
rm -rf kiro-config-main

# Or use git clone and copy
git clone https://github.com/mehtadman87/kiro-config.git temp-kiro
mkdir -p .kiro
cp -r temp-kiro/hooks temp-kiro/settings temp-kiro/specs temp-kiro/steering .kiro/
rm -rf temp-kiro
```

### ⚠️ Important: Activation Requirements

After installation:

- 🎯 **Steering Documents**: Automatically refresh and become active immediately - no restart needed
- 🔄 **Hooks**: Require restarting Kiro or reopening the project to become active

> 💡 **Tip**: After installation, restart Kiro to ensure all hooks are properly loaded and functional.

## 📋 What's Included

### 🎯 Steering Documents (Always Active)

Automatically guide all AI interactions with established best practices:

- **AWS CLI Best Practices** - `--no-cli-pager` and AWS integration patterns
- **CDK Best Practices** - Project structure, testing, and deployment patterns
- **Development Standards** - Dependency management, code quality, documentation
- **Docker Best Practices** - Container security and optimization
- **Git Best Practices** - Conventional commits, branching, and security
- **MCP Best Practices** - Model Context Protocol server configuration and usage
- **Python Best Practices** - Code style, virtual environments, and testing
- **React Best Practices** - Component patterns, hooks, and accessibility
- **Research-Validated Development** - Web search validation for architecture and spec decisions
- **Security Best Practices** - Code security, dependency management, data protection
- **Technical Debt Management** - Tracking, categorization, and remediation of tech debt
- **Testing Best Practices** - Minimal verbosity, output management, performance
- **TypeScript Best Practices** - Code style, type safety, and testing guidelines

### 🔄 Automatic Hooks (File Save Triggers)

Quality checks that run automatically when you save files:

- **Auto Test on Save** - Runs tests with minimal verbosity when code changes
- **Lint and Format on Save** - Auto-formats and lints code following project standards
- **Security Scan on Dependencies** - Audits when package files change
- **CDK Synth on Change** - Validates CDK code and runs synthesis
- **Validate Docker on Change** - Checks Docker files for best practices and security
- **MCP Config Validation** - Validates MCP server configurations
- **Environment File Validation** - Checks .env files for security issues
- **API Schema Validation** - Validates OpenAPI/GraphQL schemas and generates types
- **Spec Research Validation** - Web search validation when spec files are modified
- **Spec Creation Validation** - Web search validation when new spec files are created
- **Error Research Reminder** - Ensures errors were researched before session ends

### 🔘 Manual Hooks (Button Triggers)

On-demand tools available in the Kiro Agent Hooks panel:

- **Commit Message Helper** - Creates conventional commit messages
- **README Spell Check** - Fixes spelling and grammar in documentation
- **MCP Server Test** - Tests all configured MCP servers
- **Dependency Update Check** - Finds outdated packages and security issues
- **Code Coverage Check** - Analyzes test coverage gaps
- **Performance Analysis** - Identifies optimization opportunities

### 🎛️ Optional Hooks (Disabled by Default)

Performance-sensitive hooks you can enable as needed:

- **Accessibility Audit** - Checks React components for accessibility issues
- **Update Documentation** - Updates docs when code changes
- **Translation Update** - Syncs translation files

## ⚙️ Configuration

### Enabling/Disabling Hooks

Edit any `.kiro.hook` file and change the `enabled` field:

```json
{
  "enabled": true,  // Change to false to disable
  "name": "Hook Name",
  // ... rest of configuration
}
```

### Customizing File Patterns

Modify the `patterns` array in hook files to match your project structure:

```json
{
  "when": {
    "type": "fileEdited",
    "patterns": [
      "src/**/*.ts",     // Your source files
      "lib/**/*.js",     // Your library files
      "**/*.custom"      // Your custom extensions
    ]
  }
}
```

### Adding Project-Specific Steering

Create additional steering documents in `.kiro/steering/`:

```markdown
---
title: Your Team Standards
inclusion: always
---

# Your Team-Specific Guidelines
- Team-specific coding standards
- Project-specific patterns
- Custom workflows
```

## 🛠️ Technology Support

This boilerplate includes best practices for:

- **Languages**: TypeScript, JavaScript, Python
- **Frameworks**: React, CDK, Docker
- **Tools**: Git, npm/yarn, pytest, ESLint, Prettier
- **Cloud**: AWS (CLI, CDK, services)
- **APIs**: OpenAPI/Swagger, GraphQL
- **Testing**: Jest, pytest, coverage analysis
- **Documentation**: Markdown, JSDoc, docstrings

## 📚 MCP Integration

Includes best practices for Model Context Protocol servers:

- **Context7** - For dependency compatibility checking
- **AWS Knowledge** - For current AWS documentation and best practices
- **AWS API** - For programmatic AWS interactions
- Proper configuration patterns and testing workflows

## 🔒 Security Features

Built-in security practices:

- Dependency vulnerability scanning
- Environment file validation (no secrets in code)
- Docker security best practices
- AWS security patterns
- MCP server security configurations

## 🎨 Customization Guide

### For Your Team

1. **Review all steering documents** - Modify for your team's standards
2. **Adjust hook sensitivity** - Enable/disable based on your workflow
3. **Update file patterns** - Match your project structure
4. **Add team-specific hooks** - Create custom automation for your needs

### For Your Project Type

- **Web Applications**: Enable accessibility audit, focus on React patterns
- **Infrastructure**: Enable CDK hooks, focus on AWS patterns
- **Libraries**: Enable documentation updates, focus on API patterns
- **Microservices**: Enable Docker validation, focus on testing patterns

## 📖 Documentation

- [Steering Documents](.kiro/steering/) - Individual best practice guides
- [Hook Configurations](.kiro/hooks/) - All available automation hooks

## 🤝 Contributing

### Adding New Best Practices

1. Create steering document in `.kiro/steering/`
2. Add corresponding hook in `.kiro/hooks/`
3. Update documentation
4. Test with sample project

### Sharing Improvements

1. Fork this repository
2. Add your improvements
3. Submit pull request with description
4. Include examples of usage

## 📄 License

MIT License - Feel free to use this in your projects and modify as needed.

## 🙋‍♂️ Support

- **Issues**: Report bugs or request features via GitHub issues
- **Discussions**: Share your customizations and ask questions
- **Wiki**: Community-contributed examples and patterns

## 🎯 Quick Verification

After setup, verify everything works:

1. **Check Steering**: AI responses should reference best practices
2. **Test Hooks**: Save a TypeScript file to trigger auto-test hook
3. **Manual Hooks**: Look for buttons in Kiro Agent Hooks panel
4. **MCP Integration**: Test MCP servers if configured

Happy coding with consistent, high-quality development practices! 🚀
