---
root: false
targets:
  - "*"
description: Commit message convention (English, Conventional Commits)
---

## Commit Messages

Generate commit messages **in English**, following the [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) specification.

### Format

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### Required rules

1. **Type**: every commit starts with a type such as `feat` or `fix`
   - `feat`: a new feature (maps to Semantic Versioning MINOR)
   - `fix`: a bug fix (maps to Semantic Versioning PATCH)
   - Other types are allowed: `build`, `chore`, `ci`, `docs`, `style`, `refactor`, `perf`, `test`, etc.

2. **Scope**: an optional scope may follow the type in parentheses
   - Example: `feat(parser): add ability to parse arrays`

3. **Breaking changes**: mark them in one of two ways
   - Append `!` after the type/scope: `feat!: send an email to the customer when a product is shipped`
   - Add a `BREAKING CHANGE:` footer: `BREAKING CHANGE: environment variables now take precedence over config files`

4. **Subject**: a short summary of the change right after the colon and space

5. **Body**: an optional longer description, starting one blank line after the subject

6. **Footer(s)**: optional extra information, starting one blank line after the body

### Examples

```
feat: allow provided config object to extend other configs

BREAKING CHANGE: `extends` key in config file is now used for extending other config files
```

```
fix: prevent racing of requests

Introduce a request id and a reference to latest request. Dismiss
incoming responses other than from latest request.
```

```
docs: correct spelling of CHANGELOG
```

```
feat(api)!: send an email to the customer when a product is shipped
```
