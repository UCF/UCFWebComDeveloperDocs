# DevDocs

Developer documentation for the UCF Web Communications team, maintained as an [Obsidian](https://obsidian.md) vault and version-controlled via Git.

---

## Getting Started

### 1. Install Obsidian

Download and install Obsidian from [obsidian.md](https://obsidian.md). It's available for macOS, Windows, Linux, iOS, and Android.

### 2. Enable Community Plugins

Obsidian ships with community (third-party) plugins disabled by default. You'll need to turn this on before you can install the Git plugin.

1. Open **Settings** (gear icon in the bottom-left sidebar).
2. Navigate to **Community plugins**.
3. Click **Turn on community plugins** and confirm.

### 3. Install the Git Plugin

1. In **Settings → Community plugins**, click **Browse**.
2. Search for **Git** (the plugin by Vinzent03).
3. Click **Install**, then **Enable**.

### 4. Clone This Vault

Rather than cloning into an empty folder and then opening it, the easiest approach is:

1. Close Obsidian (or don't open it yet).
2. Clone this repository to a location on your machine:
   ```bash
   git clone <repo-url> DevDocs
   ```
3. Open Obsidian and choose **Open folder as vault**.
4. Select the `DevDocs` folder you just cloned.

Obsidian will open the vault and the Git plugin will detect the existing repository automatically.

### 5. Configure Auto-Sync

Once the Git plugin is active:

1. Open **Settings → Git**.
2. Under **Automatic**, set **Auto pull interval** to your preferred frequency (e.g., `10` minutes).
3. Set **Auto push interval** to the same or a longer interval if you'd like your local commits pushed automatically.
4. Optionally enable **Pull on startup** so the vault is always up to date when you open it.

With those options set, Obsidian will pull new changes and push your commits in the background without any manual `git` commands.

---

## Making Changes

You can commit and push from inside Obsidian using the Git plugin's **Source control** panel (accessible from the left sidebar or via the command palette with `Cmd/Ctrl+P → Git: Open source control view`). Standard Git workflows via the terminal work just as well if you prefer.
