
New WordPress child themes should be created from the [CM-WP-Child-Theme-Template](https://github.com/UCF/CM-WP-Child-Theme-Template) repository. This template includes opinionated build tool and linter configurations for SCSS and JS assets following UCF Web Communications code standards. The template assumes the [UCF WordPress Theme](https://github.com/UCF/UCF-WordPress-Theme) as the parent theme, but this can be changed as needed.

## Prerequisites

- Node.js v20+
- `gulp-cli` installed globally (`npm install -g gulp-cli`)
- A GitHub account with access to the UCF organization

---

## 1. Create the Repository from the Template

1. Go to [https://github.com/UCF/CM-WP-Child-Theme-Template/generate](https://github.com/UCF/CM-WP-Child-Theme-Template/generate).
2. Set the **Owner** to `UCF`.
3. Enter a **Repository name** using the theme's GitHub slug (e.g. `My-Child-Theme`).
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
| `{{My-Project}}` | The GitHub repository slug | `My-Child-Theme` |
| `{{My Project}}` | The human-readable theme name | `My Child Theme` |

Do a project-wide find-and-replace in your editor for both values. You can search for `{{` to locate any remaining placeholders after the initial replacements.

### Files that require placeholder replacement

- `style.css` — theme header (`Theme Name`, `Github Theme URI`)
- `README.md` — theme description and documentation links
- `CONTRIBUTING.md` — contributing guidelines links
- `.github/` — issue templates and any other references

> **Note:** `style.css` is **not** a registered stylesheet. It exists only to provide WordPress with the theme name, version, and parent theme (`Template: UCF-WordPress-Theme`). All style overrides go in `src/scss/` and are compiled to `static/css/`. If the parent theme is not the UCF WordPress Theme, update the `Template:` value in `style.css` to match the folder name of the correct parent theme.

---

## 4. Update PHP Namespaces and Prefixes

Unlike the plain theme template, the child theme template ships with a PHP namespace and constant/function prefixes that must be updated to be unique to your project. These changes are required to avoid conflicts if multiple child themes are loaded in the same environment.

### 4a. Update the PHP namespace

In every PHP file (including `functions.php` and all files under `includes/`), change the namespace from `MyProject\Theme` to a name specific to your project:

```php
// Before
namespace MyProject\Theme;

// After (example)
namespace MyChildTheme\Theme;
```

### 4b. Update constant prefixes

In `includes/config.php` and `includes/meta.php`, rename every constant that starts with `MYPROJECT_` to use your project-specific prefix:

```php
// Before
define( 'MYPROJECT_THEME_DIR', ... );

// After (example)
define( 'MYCHILDTHEME_THEME_DIR', ... );
```

Also update the reference to this constant in `functions.php`.

### 4c. Update function prefixes

In `includes/meta.php`, rename every function that starts with `myproject_` to use your project-specific prefix. Remember to also update the corresponding references in any `add_action` / `add_filter` calls in the same file:

```php
// Before
function myproject_enqueue_styles() { ... }
add_action( 'wp_enqueue_scripts', 'MyProject\Theme\myproject_enqueue_styles', ... );

// After (example)
function mychildtheme_enqueue_styles() { ... }
add_action( 'wp_enqueue_scripts', 'MyChildTheme\Theme\mychildtheme_enqueue_styles', ... );
```

---

## 5. Build Front-End Assets

The template uses Gulp to compile and lint SCSS and JavaScript. Unlike the plain theme template, the child theme's compiled CSS and JS are **enqueued automatically** via `includes/meta.php`, with the parent theme's assets set as dependencies to ensure correct load order.

### Source and output paths

| Source | Output |
|---|---|
| `src/scss/style.scss` | `static/css/style.min.css` |
| `src/js/script.js` | `static/js/script.min.js` |

The Athena Framework's variables and mixins are imported into the main Sass file. The full framework does not need to be included in the child theme since it is loaded by the parent theme.

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

## 6. Set Up the Project Wiki

All new themes should have a GitHub wiki for documentation. Use the [CM-Documentation-Templates](https://github.com/UCF/CM-Documentation-Templates) plugin wiki template.

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

## 7. Add to UCF Packagist (if applicable)

If the theme will be installed via Composer from the UCF Packagist registry, two additional steps are needed. See [UCF Packagist Management](UCF%20Packagist%20Management.md) for full details.

1. **Add the repository to ucf-packagist** by running the [Add Package workflow](https://github.com/UCF/ucf-packagist/actions/workflows/satis-add-package.yml).
2. **Add the release trigger workflow** to the new theme repository at `.github/workflows/trigger-packagist-update.yml` so the registry rebuilds automatically on each new release.

---

## Summary Checklist

- [ ] Repository created from [CM-WP-Child-Theme-Template](https://github.com/UCF/CM-WP-Child-Theme-Template/generate)
- [ ] All `{{My-Project}}` and `{{My Project}}` placeholders replaced
- [ ] PHP namespace updated from `MyProject\Theme` to a project-specific name
- [ ] `MYPROJECT_` constant prefixes updated in `includes/config.php` and `includes/meta.php`
- [ ] `myproject_` function prefixes updated in `includes/meta.php` (including hook references)
- [ ] `style.css` `Template:` value confirmed or updated to point to the correct parent theme
- [ ] `npm install` run
- [ ] `gulp default` run successfully
- [ ] `README.md` generated (via `gulp readme`)
- [ ] Project wiki created using CM-Documentation-Templates
- [ ] Added to UCF Packagist (if applicable)
