
New WordPress themes should be created from the [CM-WP-Theme-Template](https://github.com/UCF/CM-WP-Theme-Template) repository. This template includes opinionated build tool and linter configurations for SCSS and JS assets following UCF Web Communications code standards. Generated themes include the [Athena Framework](https://ucf.github.io/Athena-Framework/).

## Prerequisites

- Node.js v20+
- `gulp-cli` installed globally (`npm install -g gulp-cli`)
- A GitHub account with access to the UCF organization

---

## 1. Create the Repository from the Template

1. Go to [https://github.com/UCF/CM-WP-Theme-Template/generate](https://github.com/UCF/CM-WP-Theme-Template/generate).
2. Set the **Owner** to `UCF`.
3. Enter a **Repository name** using the theme's GitHub slug (e.g. `My-Theme`).
4. Leave the repository **Public** unless there is a specific reason for privacy.
5. Click **Create repository**.

---

## 2. Clone and Set Up Locally

Clone the new repository into your WordPress installation's `themes/` directory:

```bash
git clone git@github.com:UCF/YOUR-THEME-SLUG.git
cd YOUR-THEME-SLUG
npm install
```

---

## 3. Replace Template Placeholders

The template uses two placeholder patterns throughout its files:

| Placeholder | What to replace it with | Example |
|---|---|---|
| `{{My-Project}}` | The GitHub repository slug | `My-Theme` |
| `{{My Project}}` | The human-readable theme name | `My Theme` |

Do a project-wide find-and-replace in your editor for both values. You can search for `{{` to locate any remaining placeholders after the initial replacements.

### Files that require placeholder replacement

- `style.css` — theme header (`Theme Name`, `Github Theme URI`)
- `README.md` — theme description and documentation links
- `CONTRIBUTING.md` — contributing guidelines links
- `.github/` — issue templates and any other references

> **Note:** `style.css` is **not** a registered stylesheet. It exists only to provide WordPress with the theme name and version. All style overrides go in `src/scss/` and are compiled to `static/css/`.

---

## 4. Build Front-End Assets

The template uses Gulp to compile and lint SCSS and JavaScript.

### Source and output paths

| Source | Output |
|---|---|
| `src/scss/style.scss` | `static/css/style.min.css` |
| `src/js/script.js` | `static/js/script.min.js` |

> The theme template does **not** register or enqueue these assets automatically. You are responsible for adding `wp_enqueue_style` and `wp_enqueue_scripts` calls in your theme code.

### Run the initial build

```bash
gulp default
```

This runs three tasks in sequence:
- **`css`** — lints all SCSS files, then compiles `src/scss/style.scss` to `static/css/style.min.css`
- **`js`** — lints all JS files (auto-fixing where possible), then bundles and minifies `src/js/script.js` to `static/js/script.min.js`
- **`readme`** — generates `README.md` from `README.txt`

### Other available Gulp tasks

| Command | Description |
|---|---|
| `gulp css` | Lint and compile SCSS only |
| `gulp js` | Lint and compile JS only |
| `gulp readme` | Regenerate `README.md` from `README.txt` |
| `gulp watch` | Watch SCSS, JS, and PHP files for changes and recompile automatically |

> **Note:** `README.md` is generated automatically from `README.txt`. Do not edit `README.md` directly — always edit `README.txt` and run `gulp readme`.

### Optional: Enable BrowserSync

To enable live browser reloading during development, copy the template config and edit it:

```bash
cp gulp-config.template.json gulp-config.json
```

In `gulp-config.json`, set `sync` to `true` and update `syncTarget` to the local URL of the WordPress site where you will test the theme:

```json
{
  "sync": true,
  "syncTarget": "http://localhost/wordpress/my-site/"
}
```

`gulp-config.json` is gitignored and safe to keep locally without committing.

---

## 5. Set Up the Project Wiki

All new themes should have a GitHub wiki for documentation. Use the [CM-Documentation-Templates](https://github.com/UCF/CM-Documentation-Templates) theme wiki template.

1. In your new GitHub repository, click the **Wiki** tab and create the initial "Home" page (the content will be overwritten).

2. Create a local directory for the wiki files and `cd` into it.

3. Pull down the theme wiki template:
   ```bash
   git clone --depth=1 -b theme-wiki git@github.com:UCF/CM-Documentation-Templates.git .; rm -rf .git
   ```

4. Initialize a new git repo and push it to the wiki remote (replace `MY-PROJECT` with your repo name):
   ```bash
   git init
   git add --all
   git commit -m "Initial commit"
   git remote add origin git@github.com:UCF/MY-PROJECT.wiki.git
   git push -u origin master --force
   ```

5. Replace all `{{My-Project}}` and `{{My Project}}` placeholders in the wiki files, then commit and push. Search for `{{` to find any remaining ones.

---

## 6. Add to UCF Packagist (if applicable)

If the theme will be installed via Composer from the UCF Packagist registry, two additional steps are needed. See [UCF Packagist Management](UCF%20Packagist%20Management.md) for full details.

1. **Add the repository to ucf-packagist** by running the [Add Package workflow](https://github.com/UCF/ucf-packagist/actions/workflows/satis-add-package.yml).
2. **Add the release trigger workflow** to the new theme repository at `.github/workflows/trigger-packagist-update.yml` so the registry rebuilds automatically on each new release.

---

## Summary Checklist

- [ ] Repository created from [CM-WP-Theme-Template](https://github.com/UCF/CM-WP-Theme-Template/generate)
- [ ] All `{{My-Project}}` and `{{My Project}}` placeholders replaced
- [ ] `npm install` run
- [ ] `gulp default` run successfully
- [ ] `README.md` generated (via `gulp readme`)
- [ ] Project wiki created using CM-Documentation-Templates
- [ ] Added to UCF Packagist (if applicable)
