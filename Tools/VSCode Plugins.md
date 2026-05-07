## Linters

Linters help enforce consistent code style and formatting across a project. The two plugins covered here — EditorConfig and Prettier — work well together: EditorConfig handles low-level editor behavior (indentation, line endings, charset), while Prettier handles opinionated code formatting.

---

### EditorConfig for VS Code

**Extension ID:** `EditorConfig.EditorConfig`

EditorConfig reads a `.editorconfig` file at the root of your project and automatically applies editor settings like indentation style, tab width, line endings, and trailing whitespace rules. This keeps basic formatting consistent across different editors and IDEs.

#### Installation

1. Open the Extensions panel (`Cmd+Shift+X`)
2. Search for **EditorConfig for VS Code**
3. Click **Install**

Or install via the CLI:

```bash
code --install-extension EditorConfig.EditorConfig
```

#### Setup

Create a `.editorconfig` file in the root of your project. Below is a common starting configuration:

```ini
# .editorconfig
root = true

[*]
indent_style = space
indent_size = 4
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true

[*.{js,ts,json,yml,yaml,css,scss,html}]
indent_size = 2

[*.md]
trim_trailing_whitespace = false
```

#### Key Properties

| Property | Description |
|---|---|
| `root = true` | Stops EditorConfig from looking in parent directories |
| `indent_style` | `space` or `tab` |
| `indent_size` | Number of spaces per indent level |
| `end_of_line` | `lf`, `crlf`, or `cr` |
| `charset` | `utf-8`, `utf-16be`, `utf-16le`, `latin1` |
| `trim_trailing_whitespace` | Remove trailing whitespace on save |
| `insert_final_newline` | Ensure files end with a newline |

The extension applies these settings automatically whenever you open a file — no additional VS Code configuration is needed.

---

### Prettier - Code Formatter

**Extension ID:** `esbenp.prettier-vscode`

Prettier is an opinionated code formatter that enforces a consistent style by parsing your code and reprinting it. It supports JavaScript, TypeScript, CSS, SCSS, HTML, JSON, Markdown, and more.

#### Installation

1. Open the Extensions panel (`Cmd+Shift+X`)
2. Search for **Prettier - Code Formatter**
3. Click **Install**

Or install via the CLI:

```bash
code --install-extension esbenp.prettier-vscode
```

It is also recommended to install Prettier locally in your project so the version is pinned:

```bash
npm install --save-dev prettier
```

#### Setup

**1. Set Prettier as the default formatter in VS Code**

Open your VS Code settings (`Cmd+,`) and add the following to `settings.json`:

```json
{
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.formatOnSave": true
}
```

To apply Prettier only to specific languages, scope the default formatter:

```json
{
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[css]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

**2. Create a Prettier config file**

Create a `.prettierrc` file in the root of your project to define your formatting rules:

```json
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "useTabs": false,
  "trailingComma": "es5",
  "printWidth": 100,
  "bracketSpacing": true,
  "arrowParens": "always"
}
```

**3. Create a `.prettierignore` file**

Add a `.prettierignore` to exclude files or directories from formatting:

```
node_modules/
dist/
build/
*.min.js
```

#### Key Config Options

| Option | Default | Description |
|---|---|---|
| `semi` | `true` | Add semicolons at end of statements |
| `singleQuote` | `false` | Use single quotes instead of double quotes |
| `tabWidth` | `2` | Number of spaces per indentation level |
| `useTabs` | `false` | Indent with tabs instead of spaces |
| `trailingComma` | `"all"` | Add trailing commas (`"none"`, `"es5"`, `"all"`) |
| `printWidth` | `80` | Line length before Prettier wraps |
| `bracketSpacing` | `true` | Print spaces inside object braces |
| `arrowParens` | `"always"` | Include parens around single arrow function args |

#### Usage

- **Format on save** — works automatically if `editor.formatOnSave` is enabled
- **Format manually** — press `Shift+Alt+F` (or `Shift+Option+F` on macOS)
- **Format selection** — select code, then right-click → **Format Selection**

#### Using EditorConfig and Prettier Together

When both plugins are active, their responsibilities should not overlap. A recommended approach:

- Use **EditorConfig** for: `indent_style`, `indent_size`, `end_of_line`, `insert_final_newline`, `trim_trailing_whitespace`
- Use **Prettier** for: quote style, semicolons, trailing commas, print width, bracket spacing

Prettier respects EditorConfig settings for indentation and line endings by default, so keeping the two in sync prevents conflicts.
