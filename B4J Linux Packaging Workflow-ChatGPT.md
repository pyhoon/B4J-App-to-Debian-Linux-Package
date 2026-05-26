# B4J Linux Packaging Workflow Guide

## Overview

This document summarizes a recommended workflow for packaging B4J + JavaFX desktop applications into Linux packages such as `.deb`.

---

# Recommended Architecture

## Developer Input

Developer only fills:

- App Name
- Version
- Vendor
- Main JAR
- Icon
- Java Runtime Path
- Output Folder

Then click:

> Build Linux Package

---

# Best Packaging Strategy

## Option A — Use `jpackage` (Recommended)

Modern JDK already includes:

- native launcher generation
- `.deb`
- runtime bundling

This is much better than manually building `.deb`.

`jpackage` is included in:

- OpenJDK 14+
- Oracle JDK 14+

For JavaFX apps, this is currently the industry-standard solution.

---

# Final Output Structure

```text
MyApp/
├── input/
│   ├── MyApp.jar
│   ├── icon.png
│   └── runtime/
├── output/
│   └── myapp_1.0_amd64.deb
├── build.sh
└── config.json
```

---

# Simplified Workflow

Developer runs:

```bash
./build.sh
```

OR presses button in your B4J packaging tool.

---

# Recommended Internal Flow

## Step 1 — Export JAR from B4J

B4J already generates:

```text
Objects/MyApp.jar
```

---

## Step 2 — Create Custom Runtime (Optional but Recommended)

Using:

```bash
jlink
```

Example:

```bash
jlink \
  --module-path $JAVA_FX/lib:$JAVA_HOME/jmods \
  --add-modules java.base,java.desktop,java.sql,javafx.controls,javafx.graphics \
  --output runtime
```

Advantages:

- no Java installation required
- predictable runtime
- smaller footprint
- fewer support issues

---

# Step 3 — Package using `jpackage`

Example:

```bash
jpackage \
  --type deb \
  --name MyApp \
  --input input \
  --main-jar MyApp.jar \
  --main-class b4j.example.main \
  --runtime-image runtime \
  --icon icon.png \
  --linux-shortcut \
  --linux-menu-group "Office" \
  --app-version 1.0 \
  --vendor "My Company" \
  --dest output
```

This automatically creates:

```text
myapp_1.0-1_amd64.deb
```

---

# Why This Is Better Than Manual `.deb`

Manual Debian packaging requires:

- DEBIAN/control
- postinst
- prerm
- desktop files
- permissions
- mime handling

`jpackage` automates all of it.

---

# Suggested Features for a B4J Packaging Tool

## "B4X Linux Packager"

Features:

| Feature | Difficulty |
|---|---|
| Detect JDK | Easy |
| Detect JavaFX SDK | Easy |
| Auto-generate runtime via jlink | Medium |
| Auto-generate jpackage command | Easy |
| Build `.deb` | Easy |
| Build AppImage | Medium |
| Build `.rpm` | Medium |
| GUI config editor | Easy |

---

# Suggested GUI Layout

## Left Panel

Project Settings:

- App Name
- Version
- Vendor
- Identifier
- Category

## Center

Files:

- Main JAR
- Icon
- JavaFX SDK
- JDK
- Output Folder

## Right

Build Options:

- Include Runtime
- Strip Debug Symbols
- Compress Runtime
- Create Shortcut
- Create Menu Entry

---

# Docker-Based Build Strategy

You can support Windows developers by using Docker internally.

Example:

```bash
docker run ...
```

Docker image contains:

- JDK
- JavaFX
- jpackage

This allows Windows developers to generate Linux `.deb` packages without needing Linux installed directly.

---

# Suggested Docker Base

```text
ubuntu:24.04
```

Install:

- openjdk-21-jdk
- fakeroot
- dpkg-dev

---

# Example Build Script

```bash
#!/bin/bash

APP_NAME="MyApp"
VERSION="1.0"

rm -rf build
mkdir -p build/input

cp MyApp.jar build/input/

jpackage \
  --type deb \
  --name "$APP_NAME" \
  --input build/input \
  --main-jar MyApp.jar \
  --main-class b4j.example.main \
  --icon icon.png \
  --linux-shortcut \
  --app-version "$VERSION" \
  --dest build/output
```

---

# Recommended Distribution Formats

| Format | Recommended |
|---|---|
| `.deb` | YES |
| AppImage | YES |
| `.rpm` | Optional |
| Snap | Avoid initially |
| Flatpak | Later |

---

# Important JavaFX Consideration

JavaFX modules must match JDK version.

Example:

| JDK | JavaFX |
|---|---|
| 17 | 17 |
| 21 | 21 |

Mismatch causes runtime failures.

Your tool should validate this automatically.

---

# Important B4J Consideration

B4J apps usually start from:

```text
b4j.example.main
```

Your tool should auto-detect:

- Main module
- package name

from:

- `.b4j`
- manifest
- project structure

---

# Recommended Long-Term Product Vision

## Potential Product

### "B4X Native Packager"

Supports:

- Windows EXE/MSI
- Linux DEB/RPM/AppImage
- macOS DMG
- runtime bundling
- auto updater
- code signing
- installer themes

There is a genuine gap in the B4X ecosystem for this tooling.

---

# Suggested Development Roadmap

## Phase 1

Simple CLI tool:

```bash
build-linux.sh
```

using:

- `jlink`
- `jpackage`

---

## Phase 2

Build B4J GUI around it.

---

## Phase 3

Add Docker support.

This can become a valuable utility for B4X developers.
