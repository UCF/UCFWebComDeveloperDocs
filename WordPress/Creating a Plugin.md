# Creating a Plugin

New WordPress plugins should be created from the [CM-WP-Plugin-Template](https://github.com/UCF/CM-WP-Plugin-Template) repository. This template includes opinionated build tool and linter configurations for SCSS and JS assets following UCF Web Communications code standards.

## Prerequisites

- Node.js v20+
- `gulp-cli` installed globally (`npm install -g gulp-cli`)
- A GitHub account with access to the UCF organization

---

## 1. Create the Repository from the Template

1. Go to [https://github.com/UCF/CM-WP-Plugin-Template/generate](https://github.com/UCF/CM-WP-Plugin-Template/generate).
2. Set the **Owner** to `UCF`.
3. Enter a **Repository name** using the plugin's GitHub slug (e.g. `UCF-My-Plugin`).
4. Leave the repository **Public** unless there is a specific reason for privacy.
5. Click **Create repository**.

---

## 2. Clone and Set Up Locally

```bash
git clone git@github.com:UCF/YOUR-PLUGIN-SLUG.git
cd YOUR-PLUGIN-SLUG
npm install
```

---

## 3. Rename the Main Plugin File

Rename `my-project.php` to match your plugin's slug in lowercase (e.g. `ucf-my-plugin.php`). This is the file WordPress reads as the plugin entry point.

---

## 4. Replace Template Placeholders

The template uses two placeholder patterns throughout its files:

| Placeholder | What to replace it with | Example |
|---|---|---|
| `{{My-Project}}` | The GitHub repository slug | `UCF-My-Plugin` |
| `{{My Project}}` | The human-readable plugin name | `UCF My Plugin` |

Do a project-wide find-and-replace in your editor for both values. You can search for `{{` to locate any remaining placeholders after the initial replacements.

### Files that require placeholder replacement

- `my-project.php` (now renamed) — plugin header metadata and `GitHub Plugin URI`
- `README.txt` — plugin description, documentation links, and changelog
- `.github/` — issue templates and any other references

---

## 5. Build Front-End Assets

The template uses Gulp to compile and lint SCSS and JavaScript.

### Source and output paths

| Source | Output |
|---|---|
| `src/scss/style.scss` | `static/css/style.min.css` |
| `src/js/script.js` | `static/js/script.min.js` |

> The plugin template does **not** register or enqueue these assets automatically. You are responsible for adding `wp_enqueue_style` and `wp_enqueue_scripts` calls in your plugin code.

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

In `gulp-config.json`, set `sync` to `true` and update `syncTarget` to the local URL of the WordPress site where you will test the plugin:

```json
{
  "sync": true,
  "syncTarget": "http://localhost/wordpress/my-site/"
}
```

`gulp-config.json` is gitignored and safe to keep locally without committing.

---

## 6. Set Up the Project Wiki

All new plugins should have a GitHub wiki for documentation. Use the [CM-Documentation-Templates](https://github.com/UCF/CM-Documentation-Templates) plugin wiki template.

1. In your new GitHub repository, click the **Wiki** tab and create the initial "Home" page (the content will be overwritten).

2. Create a local directory for the wiki files and `cd` into it.

3. Pull down the plugin wiki template:
   ```bash
   git clone --depth=1 -b plugin-wiki git@github.com:UCF/CM-Documentation-Templates.git .; rm -rf .git
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

## 7. Add to UCF Packagist (if applicable)

If the plugin will be installed via Composer from the UCF Packagist registry, two additional steps are needed. See [UCF Packagist Management](UCF%20Packagist%20Management.md) for full details.

1. **Add the repository to ucf-packagist** by running the [Add Package workflow](https://github.com/UCF/ucf-packagist/actions/workflows/satis-add-package.yml).
2. **Add the release trigger workflow** to the new plugin repository at `.github/workflows/trigger-packagist-update.yml` so the registry rebuilds automatically on each new release.

---

## Summary Checklist

- [ ] Repository created from [CM-WP-Plugin-Template](https://github.com/UCF/CM-WP-Plugin-Template/generate)
- [ ] `my-project.php` renamed to the plugin slug
- [ ] All `{{My-Project}}` and `{{My Project}}` placeholders replaced
- [ ] `npm install` run
- [ ] `gulp default` run successfully
- [ ] `README.md` generated (via `gulp readme`)
- [ ] Project wiki created using CM-Documentation-Templates
- [ ] Added to UCF Packagist (if applicable)
