## ASK:

My app create config ini file, json files, sqlite3.db file when initialized on first run. 
It may also download other images or assets file from my remote server, 
what directories are required?

---

## ANSWER:

Yes. For Linux desktop apps, especially JavaFX/B4J apps, you should separate your files into proper Linux filesystem locations instead of writing everything beside the JAR.

This is important for:

1. .deb policy compliance
2. sandboxing compatibility
3. multi-user systems
4. permissions
5. clean uninstall
6. future Flatpak/AppImage support
7. Recommended Linux Directory Structure

For a packaged desktop app:
```
/opt/MyApp/                  -> application binaries
/usr/share/applications/     -> .desktop launcher
/usr/share/icons/            -> icons

$HOME/.config/MyApp/         -> config ini/json
$HOME/.local/share/MyApp/    -> app data/assets/database
$HOME/.cache/MyApp/          -> temporary cache
```

## Recommended Usage By File Type

| **File Type**	| **Recommended Location** |
| ------------- | ------------------------ |
| JAR files	    | /opt/MyApp/
| Runtime/JRE	| /opt/MyApp/runtime/
| Static bundled assets	| /opt/MyApp/assets/
| User config .ini	| ~/.config/MyApp/
| JSON settings	| ~/.config/MyApp/
| SQLite database	| ~/.local/share/MyApp/
| Downloaded images	| ~/.local/share/MyApp/assets/
| Cache/temp	| ~/.cache/MyApp/
| Logs	        | ~/.local/share/MyApp/logs/

## Why NOT Store Beside JAR

Many Windows apps do:
```
MyApp/
  MyApp.jar
  config.ini
  database.db
```

This is problematic on Linux because:

- /opt is often read-only for normal users
- installed apps should not modify install directory
- multi-user conflicts
- uninstall becomes messy

## Recommended B4J Strategy

### App Install Location

Installed by `.deb`:
```
/opt/MyApp/
```
Contains:
```
/opt/MyApp/
├── MyApp
├── app/
├── runtime/
├── assets/
```
These should be treated as read-only.

## User Writable Locations

### Config
```
~/.config/MyApp/
```
Example:
```
config.ini
settings.json
license.json
```
### User Data
```
~/.local/share/MyApp/
```
Example:
```
database.db
downloads/
receipts/
images/
backup/
```

### Cache
~/.cache/MyApp/
```
Example:
```
thumbnails/
temp/
session/
```

Cache may be deleted anytime.

## Recommended First-Run Initialization

On startup:
```b4x
If Not(File.Exists(ConfigDir, "config.ini")) Then
    InitializeFolders
    CreateDefaultConfig
    DownloadAssets
    InitDatabase
End If
```

## Recommended Folder Helper Functions (B4J)
Example:
```b4x
Private fx As JFX

Public Sub GetConfigDir As String
    Return File.Combine(GetUserHome, ".config/MyApp")
End Sub

Public Sub GetDataDir As String
    Return File.Combine(GetUserHome, ".local/share/MyApp")
End Sub

Public Sub GetCacheDir As String
    Return File.Combine(GetUserHome, ".cache/MyApp")
End Sub

Private Sub GetUserHome As String
    Return GetSystemProperty("user.home", "")
End Sub
```

## Better Linux-Compliant Version

Use XDG environment variables.

Linux officially recommends:

| ** Variable **  |	** Purpose **  |
| --------------  | -------------- |
| XDG_CONFIG_HOME |	configs        |
| XDG_DATA_HOME	  | app data       |
| XDG_CACHE_HOME  | cache          |

Fallbacks:

~/.config
~/.local/share
~/.cache

## Recommended B4J XDG Implementation

Example:
```b4x
Public Sub GetConfigDir As String
    Dim base As String = GetEnvDefault("XDG_CONFIG_HOME", _
        File.Combine(GetUserHome, ".config"))

    Return File.Combine(base, "MyApp")
End Sub
```

## SQLite Database Location

Recommended:
```
~/.local/share/MyApp/database.db
```
NOT:
```
/opt/MyApp/database.db
```
Reason:

- user writable
- backups easier
- no root permissions needed

## Downloaded Assets

If assets are permanent user data:
```
~/.local/share/MyApp/assets/
```
If temporary:
```
~/.cache/MyApp/
```

## Auto-Updating Assets

Good pattern:
```
~/.local/share/MyApp/assets/
    manifest.json
    img1.png
    img2.png
```
Then:

1. app downloads manifest
2. compares hashes/version
3. updates changed files only

Very scalable.

## Logs

Recommended:
```
~/.local/share/MyApp/logs/
```
Example:
```
app.log
error.log
sync.log
```

Multi-User Safety

If two Linux users use same machine:
```
/home/alice/.config/MyApp/
/home/bob/.config/MyApp/
```
Everything stays isolated automatically.

## Uninstall Behavior

When uninstalling .deb:

### Automatically Removed
```
/opt/MyApp/
.desktop
icons
```
### Preserved

User data:
```
~/.config/MyApp/
~/.local/share/MyApp/
```
This is correct Linux behavior.

You may optionally provide:
```
myapp --purge-user-data
```

## Suggested Final Layout

### Installed Files
```
/opt/MyApp/
```

### User Files
```
~/.config/MyApp/
~/.local/share/MyApp/
~/.cache/MyApp/
```

## Strong Recommendation

Your future B4X Linux Packager should automatically generate:

- XDG-compliant folder helper module
- first-run initializer
- asset downloader
- database bootstrapper

This becomes a major selling point because most Java desktop apps handle Linux filesystem layout poorly.