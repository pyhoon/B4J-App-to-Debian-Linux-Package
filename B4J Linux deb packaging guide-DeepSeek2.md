**User Question:**

> My app create config ini file, json files, sqlite3.db file when initialized on first run. It may also download other images or assets file from my remote server, what directories are required?

--- 

## Answer

When your application creates and modifies files at runtime (config, database, downloaded assets), you need to follow Linux standards to avoid permission issues and keep the system clean. A `.deb` package typically installs files to read-only locations like `/usr/bin` and `/usr/share`. Your application’s **writable data** must be stored in the user’s home directory.

Here’s how to set up the necessary directories for your B4J app, plus an improved launcher script that handles them automatically.

## 📁 Standard Linux Directory Layout for Your App

### Read-only (packaged inside the `.deb`)
| Purpose	| Location     |
| --------- | ------------ |
| Main executable JAR	| /usr/bin/MyApp.jar |
| Launcher script	    | /usr/bin/myapp     |
| Desktop entry	        | /usr/share/applications/myapp.desktop |
| Icon	                | /usr/share/icons/hicolor/256x256/apps/myapp.png |
| **Static bundled assets** (images, etc.)	| /usr/share/myapp/ |
>💡 Move your JAR and any default resources you ship with the app into `/usr/share/myapp/` instead of `/usr/bin/` to keep the binary directory clean. The launcher script will point to them.

### Writable (created at runtime)

These directories **must not** be part of the `.deb`. They are created by the app/launcher when the user first runs it.

| Purpose	| Standard Location (XDG)	| Fallback (if XDG variables not set) |
| --------- | ------------------------- | ----------------------------------- |
| Configuration files (`config.ini`, etc.) | `$XDG_CONFIG_HOME/myapp/`	| `~/.config/myapp/`
| Persistent data (`myapp.db`, `data.json`)	 | `$XDG_DATA_HOME/myapp/`	| `~/.local/share/myapp/`
| Cache / downloaded temporary assets	 | `$XDG_CACHE_HOME/myapp/`	| `~/.cache/myapp/`
| User-specific logs (optional)	         | `$XDG_STATE_HOME/myapp/`	| `~/.local/state/myapp/`
These paths follow the **XDG Base Directory Specification** and are widely supported on modern Linux desktops.

## 🚀 Updated Launcher Script with Directory Creation

Save the following enhanced launcher script (inside your packaging script’s `DEBIAN` post-installation or embed it in the final `.deb`). It creates the required directories and passes them to your Java app as system properties.

```bash
#!/bin/bash

APP_NAME="myapp"
JAR_DIR="/usr/share/$APP_NAME"
JAR_FILE="$JAR_DIR/MyApp.jar"

# XDG base directories (with fallbacks)
export XDG_CONFIG_HOME="${XDG_CONFIG_HOME:-$HOME/.config}"
export XDG_DATA_HOME="${XDG_DATA_HOME:-$HOME/.local/share}"
export XDG_CACHE_HOME="${XDG_CACHE_HOME:-$HOME/.cache}"

# App-specific writable paths
CONFIG_DIR="$XDG_CONFIG_HOME/$APP_NAME"
DATA_DIR="$XDG_DATA_HOME/$APP_NAME"
CACHE_DIR="$XDG_CACHE_HOME/$APP_NAME"

# Create them silently if they don't exist
mkdir -p "$CONFIG_DIR" "$DATA_DIR" "$CACHE_DIR"

# Run the JavaFX app, passing the paths
java -jar "$JAR_FILE" \
    -Dapp.configDir="$CONFIG_DIR" \
    -Dapp.dataDir="$DATA_DIR" \
    -Dapp.cacheDir="$CACHE_DIR"
```
In your B4J code, read those system properties:
```b4x
Dim configDir As String = GetSystemProperty("app.configDir", "")
If configDir = "" Then configDir = File.Combine(File.DirApp, "config") ' fallback for dev
' Use configDir to place config.ini, etc.
```
This way your app always writes to the correct user‑owned location.

## 🧩 Modifying the `pack_deb.sh` Script
Update the packaging script to:

1. Place the JAR and any bundled static assets into `/usr/share/$APP_NAME/`.
2. Install the launcher script shown above as `/usr/bin/$APP_NAME`.
3. Remove the creation of user-specific directories from the package – the launcher handles that.
**Key changes in the `pack_deb.sh` variable section:**
```bash
JAR_FILE="MyApp.jar"
STATIC_ASSETS_DIR="assets"  # a local folder containing your bundled images, etc.
...
mkdir -p "$BUILD_DIR/usr/share/$APP_NAME"
cp "$JAR_FILE" "$BUILD_DIR/usr/share/$APP_NAME/"
cp -r "$STATIC_ASSETS_DIR" "$BUILD_DIR/usr/share/$APP_NAME/"  # if you have any
```
And replace the old launcher with the new XDG‑aware one.

---

## 📌 Important Notes

**Do not put writable directories inside the*** `.deb`. They would be created as root‑owned, and ordinary users would get permission errors.

If your app must start with a default config file, include a template in `/usr/share/$APP_NAME/` and copy it to `$CONFIG_DIR` on first run if no config exists.

For SQLite databases, store them in `$DATA_DIR` (persistent user data).

Downloaded assets that can be re‑fetched should go into `$CACHE_DIR`; essential files that the user expects to keep go into `$DATA_DIR`.

With this structure, your application will behave like a well‑behaved Linux citizen, and the `.deb` package will be cleanly installable by any user.