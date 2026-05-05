Pull requests are a cornerstone of our development workflow and serve as the primary mechanism for introducing changes to UCF WordPress projects in a controlled, reviewable way. By requiring all code changes to pass through a PR before reaching the `main` branch, we ensure that every modification is vetted for quality, adherence to coding standards, and alignment with project goals. This process protects the stability of themes and plugins used across UCF's web presence, facilitates knowledge sharing among team members, and creates a clear audit trail of why and how the codebase has evolved over time. Skipping or shortcutting the PR process puts production sites at risk and undermines the collaborative nature of our development practice.

---

## Prerequisites

Before opening a pull request, the following conditions must be met:

- The work must be related to an **existing GitHub issue** that has been acknowledged by UCF Web Communications.
- For significant changes (new features, large refactors), **discuss the work first** — either in the relevant GitHub issue or in the [#help-themes Teams channel](https://teams.microsoft.com/l/channel/19%3a8ade234408d44b93a02f01b1964d7856%40thread.skype/help-themes?groupId=5e3e72e2-3599-47b3-bcfc-bb3d5b579f12&tenantId=bb932f15-ef38-42ba-91fc-f3c59d5dd1f1) before writing code. This avoids investing time in work that may not be accepted.
- Your changes must adhere to the project's [[#Code Standards]] outlined below.

---

## Branch Strategy

| Branch | Purpose |
|---|---|
| `main` | Latest stable release. Always deployable. |
| `rc-*` | Release candidate branches. Active development targets. |
| `develop` | Maintainer use only — considered a "dirty" branch. |
| Feature branches | Your feature, fix, or change branch. |

> [!note]
> Some older repositories use `master` as the name of the primary branch instead of `main`. In those cases, substitute `master` wherever `main` appears in this document.

> [!warning]
> Never create a new branch from `develop`. Branches created from `develop` will not be merged into the project.

New feature branches must be branched off of the most recent open `rc-*` branch. If no `rc-*` branch is currently open, branch directly from `main`.

---

## Submitting a Pull Request

### 1. Clone the Repository

If you haven't already, clone the repo:

```bash
git clone https://github.com/UCF/UCF-WordPress-Theme.git
cd UCF-WordPress-Theme
```

### 2. Sync with Origin

Before starting new work, make sure your local copy is up to date:

```bash
git checkout main
git pull origin main
```

### 3. Create a Feature Branch

Branch off of the current `rc-*` branch (or `main` if none exists):

```bash
git checkout rc-<version>
git checkout -b <feature-branch-name>
```

Use a descriptive branch name that reflects the change (e.g., `fix/mobile-nav-overflow` or `feature/custom-subnav`).

### 4. Make Your Changes

- Commit changes in **logical, focused chunks**.
- Write [clear, readable commit messages](https://chris.beams.io/posts/git-commit/) — avoid vague messages like "bugfix" or "minor change".
- If modifying `.scss` or `.js` files, run the appropriate gulp command to **compile and minify** the output files, and commit those compiled files alongside your source changes.

```bash
gulp default   # compile assets once
gulp watch     # watch for changes during development
```

### 5. Merge Latest RC Changes into Your Branch

Before pushing, merge the latest changes from the `rc-*` branch (or `main`) into your feature branch to minimize conflicts:

```bash
git merge --no-ff origin rc-<version>
# (or replace with "main" if no rc-* branch exists)
```

### 6. Push Your Branch

```bash
git push origin <feature-branch-name>
```

### 7. Open the Pull Request

- Open a PR on GitHub comparing your feature branch against the `rc-*` branch you branched from (or `main` if no `rc-*` branch is open).
- Provide a **descriptive title and description** — explain what changed and why.
- Keep the PR **focused in scope**; avoid bundling unrelated commits.
- Keep the code change small enough to be **reviewed within one hour**.

---

## Review Process

- Every PR requires review and approval by **at least one project maintainer** before it can be merged.
- Code that does not meet the project's coding standards will not be merged.
- Address reviewer feedback promptly and push any requested changes to the same topic branch — the PR will update automatically.

---

## Code Standards

All new and modified code must follow the language-specific standards below.

### PHP
Follow the [WordPress PHP Coding Standards](https://make.wordpress.org/core/handbook/best-practices/coding-standards/php/).

### HTML
Follow the [mdo Code Guide](http://codeguide.co/#html).
- Use HTML5-appropriate tags and self-closing elements.
- Use CDNs and HTTPS for third-party JS references.
- Use [WAI-ARIA](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA) attributes where appropriate to support accessibility.

### CSS / Sass
Follow the [mdo Code Guide (CSS)](http://codeguide.co/#css) and [CSS-Tricks Sass Style Guide](https://css-tricks.com/sass-style-guide/).
- Declarations must be in **alphabetical order**.
- Each selector in a ruleset must be on its own line.
- All colors and font sizes must comply with [WCAG 2.0 AA contrast guidelines](https://www.w3.org/TR/WCAG20/#visual-audio-contrast).
- Do not remove default `:focus` styles without providing accessible alternatives.
- No `sass-lint` errors permitted. Use a Sass-lint IDE integration to catch issues early.

### JavaScript
Follow the [jQuery Coding Standards](http://lab.abhinayrathore.com/jquery-standards/).
- Use **2 spaces** for indentation (no tabs).
- Do not use jQuery event alias methods (e.g., `$().focus()`). Use `$().trigger()` or `$().on()` instead.
- No `eslint` errors permitted. Use an eslint IDE integration to catch issues early.

---

## Related

- [[Deployment Process]]
- [UCF WordPress Theme — CONTRIBUTING.md](https://github.com/UCF/UCF-WordPress-Theme/blob/master/CONTRIBUTING.md)
- [UCF WordPress Theme — Wiki](https://github.com/UCF/UCF-WordPress-Theme/wiki)
