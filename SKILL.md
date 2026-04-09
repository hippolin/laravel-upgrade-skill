---
name: laravel-upgrade
description: >
  Guide developers through upgrading Laravel applications between major versions.
  Use this skill whenever a user mentions upgrading Laravel, migrating from one
  Laravel version to another (e.g. v6 to v7, v7 to v8), asks about breaking
  changes between versions, or wants a step-by-step upgrade checklist. Also
  trigger when the user pastes composer.json and asks how to update their Laravel
  version, or asks "what changed in Laravel X". Covers dependency updates,
  breaking changes, code migrations, and verification steps.
---

# Laravel Upgrade Skill

This skill guides developers through upgrading a Laravel application from one major version to the next. It provides a structured, step-by-step process: update dependencies, apply code changes for breaking changes (high → medium → low impact), then verify.

## Workflow

1. **Identify source and target version** — if not provided, ask the user.
2. **Load the correct reference file** — see the table below.
3. **Assess the app** — ask if the user has any relevant packages (Passport, Horizon, Rollbar, etc.) so you can flag their upgrade guides too.
4. **Walk through changes by impact** — High → Medium → Low. For each change, show the before/after code if applicable. For 8.x → 9.x upgrades, confirm PHP 8.1 is already in place before changing Laravel dependencies.
5. **Check deployment/build paths** — if the upgrade renames directories or moves framework conventions, verify Dockerfiles and other build-time COPY paths alongside Composer changes.
6. **Produce a checklist** — a markdown checklist the user can copy and track.
7. **Verify** — remind the user to run tests, check logs, and clear caches.

## Reference Files

| From → To | Reference file |
|-----------|---------------|
| 6.x → 7.x | `references/6-to-7.md` |
| 7.x → 8.x | `references/7-to-8.md` |
| 8.x → 9.x | `references/8-to-9.md` |
| 9.x → 10.x | `references/9-to-10.md` |
| 10.x → 11.x | `references/10-to-11.md` |
| 11.x → 12.x | `references/11-to-12.md` |
| 12.x → 13.x | `references/12-to-13.md` ⚠️ pre-release |

> When a new version pair is needed, create a new reference file following the same structure as the existing ones.

## Output Format

When guiding an upgrade:
- Group changes by **High / Medium / Low** impact with clear headings
- Include **before/after code blocks** for every code-level change
- End with a **copyable checklist** in `- [ ] item` format
- Call out **first-party package upgrades** separately (Passport, Horizon, Scout, etc.)

## Checklist Template

Always end your upgrade guidance with a checklist like this:

```markdown
## Laravel [X] → [Y] Upgrade Checklist

### Dependencies
- [ ] Update `composer.json` versions
- [ ] Run `composer update`

### High Impact
- [ ] ...

### Medium Impact
- [ ] ...

### Low Impact
- [ ] ...

### First-Party Packages
- [ ] ...

### Post-upgrade
- [ ] Run `php artisan config:clear`
- [ ] Run `php artisan cache:clear`
- [ ] Run `php artisan view:clear`
- [ ] Run full test suite
- [ ] Check application logs for errors
```
