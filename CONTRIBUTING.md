# Contributing to fizzy-md-selfhost

## Conventional Commits

This project uses [Conventional Commits](https://www.conventionalcommits.org/) specification.

### Commit Format

```
<type>(<scope>): <subject>

[optional body]

[optional footer]
```

### Types

| Type | Description |
|------|-------------|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation changes |
| `style` | Code style changes (formatting, etc) |
| `refactor` | Code refactoring |
| `perf` | Performance improvements |
| `test` | Adding or updating tests |
| `build` | Build system changes |
| `ci` | CI/CD changes |
| `chore` | Maintenance tasks |
| `revert` | Revert previous commit |

### Examples

```bash
# Good commits
feat: add self-hosted Fizzy support
fix(wrapper): handle empty column name correctly
docs: update README with installation steps
chore: add commitlint configuration
refactor(go): simplify wrapper detection logic

# Bad commits (will be rejected)
Add new feature
fixed bug
update docs
```

### Setup

1. Install dependencies:
   ```bash
   npm install
   ```

2. Hooks are automatically configured via Husky

3. Validate commits manually:
   ```bash
   npm run commitlint
   ```

### Pull Request Guidelines

1. Fork the repository
2. Create a feature branch: `git checkout -b feat/my-feature`
3. Make commits following conventional format
4. Push and open PR
5. Ensure all commits pass commitlint