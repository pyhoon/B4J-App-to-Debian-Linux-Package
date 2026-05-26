### USER
My app create config ini file, json files, sqlite3.db file when initialized on first run. 
It may also download other images or assets file from my remote server, 
what directories are required?

### ASSISTANT
This is one of the most common "gotchas" when moving a Java/B4J app from the IDE to a packaged Linux `.deb` installer. 

Here is the **Golden Rule of Linux Packaging**: 
When a user installs your `.deb` file, the application is installed into `/opt/YourAppName/`. **This directory is owned by `root` and is strictly read-only for standard users.** 

If your B4J app tries to use `File.DirApp` to save an INI file, write to the SQLite database, or download images, **it will crash with a "Permission Denied" error.**

To fix this, you must separate your **Read-Only App Files** from your **Writable User Data** using the official Linux XDG Base Directory standards.

---

### The Required Linux Directories

For a modern Linux Mint/Ubuntu app, you must use the user's hidden home directories. Here is exactly where each file type should go:

#### 1. Configuration Files (INI, small JSON settings)
* **Directory:** `~/.config/YourAppName/`
* **Linux Standard:** `$XDG_CONFIG_HOME`
* **Purpose:** Small, human-readable settings. If a user uninstalls your app, these might be left behind, or they can easily delete this folder to "reset" the app.

#### 2. Heavy Data (SQLite3 DB, large JSONs, Downloaded Images/Assets)
* **Directory:** `~/.local/share/YourAppName/`
* **Linux Standard:** `$XDG_DATA_HOME`
* **Purpose:** Crucial user data, databases, and downloaded assets. This is the standard place for apps to store data that the user shouldn't casually edit by hand.

*(Note: `~` represents the user's home directory, e.g., `/home/username/`)*

---

### How to Implement This in B4J

You need to update your B4J code to detect if it's running on Linux and route the file paths to these specific directories. 

Here is a ready-to-use B4J code module to handle this elegantly:

```b4x
' Put this in your Main module or a dedicated FileHelper code module

Public AppName As String = "MyB4JApp" ' Change this to your actual app name

' Returns the correct writable directory based on file type
Public Sub GetWritableDir(DirType As String) As String
    Dim userHome As String = GetUserHome
    
    ' Check if we are running on Linux (packaged or IDE)
    Dim osName As String = GetOSName
    If osName.ToLowerCase.Contains("linux") Then
        If DirType = "CONFIG" Then
            Return File.Combine(userHome, ".config/" & AppName)
        Else If DirType = "DATA" Then
            Return File.Combine(userHome, ".local/share/" & AppName)
        End If
    Else
        ' Fallback for Windows/Mac (uses standard AppData/Documents)
        Return File.Combine(userHome, AppName)
    End If
End Sub

' Helper to get the OS Name
Private Sub GetOSName As String
    Dim jo As JavaObject
    jo.InitializeStatic("java.lang.System")
    Return jo.RunMethod("getProperty", Array("os.name"))
End Sub

' Helper to get the User's Home Directory
Private Sub GetUserHome As String
    Dim jo As JavaObject
    jo.InitializeStatic("java.lang.System")
    Return jo.RunMethod("getProperty", Array("user.home"))
End Sub
```

### How to use it in your App Initialization:

When your app starts, you should ensure these directories exist, and then use them for your file operations.

```b4x
Sub AppStart (Form1 As Form, Args() As String)
    ' 1. Define your paths
    Dim ConfigDir As String = GetWritableDir("CONFIG")
    Dim DataDir As String = GetWritableDir("DATA")
    
    ' 2. Create the directories if they don't exist (First Run)
    If File.Exists(ConfigDir, "") = False Then File.MakeDir(ConfigDir, "")
    If File.Exists(DataDir, "") = False Then File.MakeDir(DataDir, "")
    
    ' 3. Handle First Run Defaults (Optional)
    ' If you have a default config.ini in your B4J Files tab (Assets), copy it on first run
    If File.Exists(ConfigDir, "config.ini") = False Then
        File.Copy(File.DirAssets, "default_config.ini", ConfigDir, "config.ini")
    End If
    
    ' 4. Initialize your SQLite Database in the DATA directory
    Dim SQL1 As SQL
    SQL1.InitializeSQLite(DataDir, "my_database.db", True)
    
    ' 5. Download images to the DATA directory
    ' Dim DownloadPath As String = File.Combine(DataDir, "downloaded_image.png")
    
    ' ... rest of your app code ...
End Sub
```

---

### Summary of your App's File Flow

1. **Read-Only Assets (Icons, Default Configs, FXML layouts):** 
   * These stay inside the `.jar` or the `/opt/MyB4JApp/` installation folder. 
   * You access them in B4J using `File.DirAssets`. You **never** write to `File.DirAssets`.
2. **Config INI / Settings JSON:** 
   * Read/Write to `~/.config/MyB4JApp/`
3. **SQLite DB & Downloaded Images:** 
   * Read/Write to `~/.local/share/MyB4JApp/`

By structuring your B4J app this way, it will behave exactly like a native Linux Mint application. It will respect user permissions, it won't crash when trying to save data, and if the user uninstalls the `.deb` package via the Linux Mint Software Manager, their personal database and downloaded images will safely remain in their home folder (which is the expected behavior for Linux apps).