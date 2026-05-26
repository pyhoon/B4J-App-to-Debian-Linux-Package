
# Packaging B4J JavaFX Applications for Linux (.deb)

This document outlines the workflows for automating and simplifying the creation of native Linux `.deb` installers for UI applications developed using B4J and JavaFX.

I use B4J to create UI app with JavaFX library and compile into jar file. Can you help me to simplify the workflow to create a script or a user friendly tool for developer to package it as Linux package such as .deb so it can be install easily?
Packaging a B4J (JavaFX) app into a native Linux .deb file is a massive upgrade for user experience. Distributing raw .jar files requires your end-users to mess with command lines, Java versions, and JavaFX module paths—which is exactly what you want to avoid.

Since you want to simplify your workflow, you have two primary paths to automate this. I have designed two scripts for you depending on how you want the app to behave on the user's machine.

(Note: To build a .deb package, you need to run these scripts on a Linux environment, such as a Linux VM, a native installation, or WSL on Windows).

Method 1: The Modern "Self-Contained" Approach (jpackage)
Best for: End-users who don't want to install Java.
Since Java 14, the JDK includes a tool called jpackage. It takes your .jar, strips down the Java Runtime Environment (JRE) to only what your app needs, and bundles it all into a single .deb file. The final app feels exactly like a native Linux app.

Prerequisites: Install fakeroot and dpkg-deb on your Linux build machine (sudo apt install fakeroot dpkg).

Save this script as build_deb_jpackage.sh, make it executable (chmod +x build_deb_jpackage.sh), and run it:

---

## Method 1: The Modern "Self-Contained" Approach (`jpackage`)

**Best for:** End-users who do not have Java installed. 
This method takes your compiled `.jar`, strips down the Java Runtime Environment (JRE) to only the modules your app needs, and bundles it all into a single `.deb` file.

### Prerequisites (Run on Linux Build Machine)
```bash
sudo apt update
sudo apt install fakeroot dpkg-dev
```

Automation Script (build_deb_jpackage.sh)
```bash
#!/bin/bash
# ==========================================
# JPackage B4J to .deb Automation Script
# ==========================================

APP_NAME="my-b4j-app"
APP_VERSION="1.0.0"
DESCRIPTION="A fantastic B4J UI Application"
AUTHOR="Your Name"

# The directory containing your compiled B4J .jar and any extra libraries
INPUT_DIR="./build"
MAIN_JAR="YourApp.jar"
ICON_FILE="icon.png" # Must be a .png for Linux
OUTPUT_DIR="./dist"

echo "Packaging $APP_NAME version $APP_VERSION..."

# Ensure output directory exists
mkdir -p "$OUTPUT_DIR"

# Run jpackage
jpackage \
  --type deb \
  --name "$APP_NAME" \
  --app-version "$APP_VERSION" \
  --description "$DESCRIPTION" \
  --vendor "$AUTHOR" \
  --input "$INPUT_DIR" \
  --dest "$OUTPUT_DIR" \
  --main-jar "$MAIN_JAR" \
  --icon "$ICON_FILE" \
  --linux-shortcut \
  --linux-menu-group "Utility" \
  --linux-deb-maintainer "youremail@example.com"

echo "Done! Check the $OUTPUT_DIR folder for your .deb package."
```

## Method 2: The Lightweight Wrapper (dpkg-deb)
Best for: Keeping the .deb file size tiny (under 5MB).
This method assumes the user already has Java installed on their Linux machine. It wraps your .jar file and creates a native desktop shortcut.

### Automation Script (build_deb_light.sh)
```bash
#!/bin/bash
# ==========================================
# Lightweight JAR to .deb Wrapper Script
# ==========================================

APP_NAME="my-b4j-app"
APP_VERSION="1.0"
JAR_FILE="YourApp.jar"
ICON_FILE="icon.png"

# Directory structure for the .deb
DEB_DIR="${APP_NAME}_${APP_VERSION}_amd64"

echo "Setting up package structure..."
mkdir -p "$DEB_DIR/DEBIAN"
mkdir -p "$DEB_DIR/opt/$APP_NAME"
mkdir -p "$DEB_DIR/usr/share/applications"
mkdir -p "$DEB_DIR/usr/share/icons/hicolor/256x256/apps"
mkdir -p "$DEB_DIR/usr/bin"

# 1. Create the Control file (Metadata)
cat <<EOF > "$DEB_DIR/DEBIAN/control"
Package: $APP_NAME
Version: $APP_VERSION
Architecture: all
Maintainer: Your Name <youremail@example.com>
Depends: default-jre (>= 11)
Description: A fantastic B4J UI Application.
EOF

# 2. Copy the JAR and Icon
cp "$JAR_FILE" "$DEB_DIR/opt/$APP_NAME/"
cp "$ICON_FILE" "$DEB_DIR/usr/share/icons/hicolor/256x256/apps/${APP_NAME}.png"

# 3. Create a launch script in /usr/bin
cat <<EOF > "$DEB_DIR/usr/bin/$APP_NAME"
#!/bin/bash
java -jar /opt/$APP_NAME/$JAR_FILE "\$@"
EOF
chmod +x "$DEB_DIR/usr/bin/$APP_NAME"

# 4. Create the Desktop Shortcut (.desktop file)
cat <<EOF > "$DEB_DIR/usr/share/applications/$APP_NAME.desktop"
[Desktop Entry]
Version=1.0
Type=Application
Name=My B4J App
Comment=Launch My App
Exec=$APP_NAME
Icon=$APP_NAME
Terminal=false
Categories=Utility;
EOF

# 5. Build the .deb file
echo "Building the .deb package..."
dpkg-deb --build "$DEB_DIR"

# Cleanup build folders
rm -rf "$DEB_DIR"

echo "Success! Package ${DEB_DIR}.deb is ready."
```

## Hybrid Workflow for Linux Mint (B4J via Wine)
If you run the B4J IDE inside Wine on Linux Mint, Wine cannot natively invoke Linux packaging tools like dpkg. The ideal strategy is a hybrid workflow: compile the project inside Wine, but package it natively in Linux Mint.

### Step 1: Install a Native Linux JDK (with JavaFX)
To use jpackage natively on Linux Mint, you need a Linux JDK that includes JavaFX modules (such as Liberica JDK Full version or Azul Zulu with FX).

Open your native Linux Mint terminal and install dependencies:

```bash
sudo apt update
sudo apt install fakeroot dpkg-dev
```

### Step 2: Build the Release Jar (Inside Wine)
Open B4J inside Wine.

Click Project -> Build Release App (Ctrl + P).

This generates a cross-platform .jar inside your project's Objects folder.

### Step 3: Execute Native Packaging Script (make_deb.sh)
Save this script on your Linux Mint system. It reaches into your Wine directory, grabs the compiled B4J assets, and uses your native Linux environment to generate the .deb.

```bash
#!/bin/bash
# =====================================================================
# B4J Native Linux .deb Packager (Run natively on Linux Mint)
# =====================================================================

# 1. SETUP YOUR APP METADATA
APP_NAME="my-b4j-app"
VERSION="1.0.0"
AUTHOR="Your Name <developer@email.com>"
DESCRIPTION="A native Linux package for my B4J application."

# 2. PATHS (Point this to your B4J project folders inside Wine)
WINE_OBJECTS_DIR="/home/YOUR_LINUX_USERNAME/.wine/drive_c/B4JProjects/YourApp/Objects"
MAIN_JAR="YourApp.jar" 
ICON_PATH="/home/YOUR_LINUX_USERNAME/Pictures/my_icon.png" # Needs to be a PNG

# 3. TEMPORARY BUILD DIRECTORIES
BUILD_DIR="./deb_build_input"
OUTPUT_DIR="./dist"

echo "--------------------------------------------------"
echo " Starting Native Linux Packaging for $APP_NAME "
echo "--------------------------------------------------"

# Clear out any old build data
rm -rf "$BUILD_DIR"
mkdir -p "$BUILD_DIR"
mkdir -p "$OUTPUT_DIR"

# Copy the compiled JAR from your Wine environment
echo "Gathering B4J compiler outputs..."
if [ -f "$WINE_OBJECTS_DIR/$MAIN_JAR" ]; then
    cp "$WINE_OBJECTS_DIR/$MAIN_JAR" "$BUILD_DIR/"
    # Copy any additional dependency folders B4J creates (like assets)
    [ -d "$WINE_OBJECTS_DIR/assets" ] && cp -r "$WINE_OBJECTS_DIR/assets" "$BUILD_DIR/"
else
    echo "ERROR: Could not find $MAIN_JAR in Wine directory!"
    exit 1
fi

# Run the native Linux jpackage tool
echo "Invoking native Linux jpackage to build .deb..."
jpackage \
  --type deb \
  --name "$APP_NAME" \
  --app-version "$VERSION" \
  --description "$DESCRIPTION" \
  --vendor "$AUTHOR" \
  --input "$BUILD_DIR" \
  --dest "$OUTPUT_DIR" \
  --main-jar "$MAIN_JAR" \
  --icon "$ICON_PATH" \
  --linux-shortcut \
  --linux-menu-group "Utility" \
  --linux-deb-maintainer "$AUTHOR"

# Clean up the staging area
rm -rf "$BUILD_DIR"

echo "--------------------------------------------------"
echo "SUCCESS! Your installer is ready in: $OUTPUT_DIR"
echo "Double-click the .deb file to install it on Linux Mint."
echo "--------------------------------------------------"
```