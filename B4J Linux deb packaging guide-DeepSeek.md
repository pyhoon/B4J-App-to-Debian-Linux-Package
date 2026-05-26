# B4J App Packaging for Linux (.deb)

**User Question:**

> I use B4J to create UI app with JavaFX library and compile into jar file. Can you help me to simplify the workflow to create a script or a user friendly tool for developer to package it as Linux package such as .deb so it can be install easily?

---

## Answer

The simplest way to package your B4J app for Linux is to use the built-in `B4JPackager11` tool or create a custom automation script. These methods produce a standalone package that users can install with a single command, without needing to manually install Java or JavaFX, similar to how `B4JPackager11` works for Windows.

Let's look at the two main approaches, from the most automated to the most customizable.

### 🚀 Method 1: Automated Packaging (Zero Scripting)

The easiest approach for B4J developers is to use your IDE's built-in `B4JPackager11` tool. This requires no custom scripting and handles the entire process for you.

1.  **Prepare Your Project**: In B4J, ensure your project compiles successfully. Then, go to **Project > Build Standalone Package**. This generates a build folder containing your JAR file and a `packager.json` file.
2.  **Copy Files to Linux**: You'll need to copy your project's build folder to a Linux machine to perform the packaging. You can do this manually, but an easier way is to use **B4J-Bridge**. With it, you can connect your IDE to a remote Linux machine and let it transfer all the necessary files automatically.
3.  **Create the Linux Package**: On the Linux machine, open a terminal in the build directory and run the packager with Java. The command looks something like this: `jdk-11.0.1/bin/java -jar B4JPackager11.jar packager.json`.
    *   **Important:** Make sure the Java version on your Linux machine matches your project's requirements. For ARM-based boards like a Raspberry Pi, you might need to use a specific JDK like Liberica JDK.

### 📜 Method 2: Custom Automation Script

For greater control or integration into a build process, a custom script is the way to go. Below is a complete, ready-to-use Bash script that automates the creation of a `.deb` package.

#### 🛠️ Prerequisites

*   A Unix-like environment (Linux or macOS) with `bash`, `mkdir`, `cp`, `chmod`, `tar`, and `dpkg-deb` installed.
*   Your compiled B4J JAR file (e.g., `MyApp.jar`).
*   (Recommended) A PNG icon for your application, which will appear in the system menu.

#### 📝 The `pack_deb.sh` Script

Save the following code as `pack_deb.sh` in your project's directory, next to your JAR file.

```bash
#!/bin/bash

# --- Configuration ----------------------------------------------------
# Set your application details here.
APP_NAME="MyB4JApp"                    # The app's short name (no spaces)
APP_VERSION="1.0.0"                   # Your app's version
APP_DESCRIPTION="My B4J JavaFX App"   # A short description
MAINTAINER_NAME="Your Name"           # Your full name
MAINTAINER_EMAIL="you@example.com"    # Your email
JAR_FILE="MyApp.jar"                  # The name of your JAR file
ICON_FILE="icon.png"                  # Path to a PNG icon (optional)
DEPENDS="openjfx"                     # Package dependencies (comma-separated)
PACKAGE_NAME="${APP_NAME}_${APP_VERSION}_all.deb" # Output package name

# --- Pre-flight Checks -------------------------------------------------
# Ensure the script stops if any command fails.
set -e

# Check if the JAR file exists.
if [ ! -f "$JAR_FILE" ]; then
    echo "ERROR: JAR file '$JAR_FILE' not found."
    exit 1
fi

echo "==> Creating package directory structure..."
# Create a temporary build directory.
BUILD_DIR="deb_build"
rm -rf "$BUILD_DIR"
mkdir -p "$BUILD_DIR/DEBIAN"
mkdir -p "$BUILD_DIR/usr/bin"
mkdir -p "$BUILD_DIR/usr/share/applications"
mkdir -p "$BUILD_DIR/usr/share/icons/hicolor/256x256/apps"

# --- 1. Copy Application Files ---------------------------------------
echo "==> Copying application files..."
cp "$JAR_FILE" "$BUILD_DIR/usr/bin/"

# If an icon file exists, copy it to the standard location.
if [ -f "$ICON_FILE" ]; then
    cp "$ICON_FILE" "$BUILD_DIR/usr/share/icons/hicolor/256x256/apps/${APP_NAME}.png"
    echo "Icon copied to /usr/share/icons"
else
    echo "No icon file found at '$ICON_FILE', skipping."
fi

# --- 2. Create Launcher Script ---------------------------------------
# This script is the command your users will run.
LAUNCHER="$BUILD_DIR/usr/bin/$APP_NAME"
echo "==> Creating launcher script: $LAUNCHER"
cat > "$LAUNCHER" << EOF
#!/bin/bash
java -jar /usr/bin/$JAR_FILE
EOF

# Make the launcher script executable.
chmod 755 "$LAUNCHER"

# --- 3. Create .desktop File -----------------------------------------
# This puts your app in the system menu.
DESKTOP_FILE="$BUILD_DIR/usr/share/applications/${APP_NAME}.desktop"
echo "==> Creating desktop entry: $DESKTOP_FILE"
cat > "$DESKTOP_FILE" << EOF
[Desktop Entry]
Version=1.0
Name=$APP_NAME
Comment=$APP_DESCRIPTION
Exec=/usr/bin/$APP_NAME
Icon=${APP_NAME}.png
Terminal=false
Type=Application
Categories=Utility;
EOF

# --- 4. Create DEBIAN/control File ------------------------------------
echo "==> Creating package metadata (DEBIAN/control)..."
cat > "$BUILD_DIR/DEBIAN/control" << EOF
Package: $APP_NAME
Version: $APP_VERSION
Section: base
Priority: optional
Architecture: all
Maintainer: $MAINTAINER_NAME <$MAINTAINER_EMAIL>
Depends: $DEPENDS
Description: $APP_DESCRIPTION
 A longer description goes here.
 It can span multiple lines.
EOF

# --- 5. Build the .deb Package ----------------------------------------
echo "==> Building the .deb package..."
dpkg-deb --build "$BUILD_DIR" "$PACKAGE_NAME"

# --- 6. Clean Up ------------------------------------------------------
echo "==> Cleaning up build directory..."
rm -rf "$BUILD_DIR"

echo "==> Package '$PACKAGE_NAME' created successfully!"
echo "You can install it with: sudo dpkg -i $PACKAGE_NAME"
```

---

## ▶️ How to Use It

1. Save and Configure: Save the script as pack_deb.sh and update the variables in the "Configuration" section to match your app.
2. Make it Executable: Open a terminal in the directory and run chmod +x pack_deb.sh.
3. Run the Script: Execute it with ./pack_deb.sh.


This is what the script does for you:
- Creates the package structure: Sets up all the directories needed inside a .deb package.
- Copies your JAR and icon: Places your application files in the correct system locations.
- Creates a launcher script: Makes a simple command for users to run your app.
- Creates a desktop entry: Integrates your app into the Linux desktop menu.
- Builds the final package: Assembles everything into a .deb file, ready to be installed with sudo dpkg -i your-package.deb.

This script is a solid foundation. You can extend it by adding preinst or postinst scripts in the DEBIAN directory for tasks like creating a user group or setting up a database.

I hope this simplifies your workflow. Let me know if you run into any issues with the configuration.