# Antigravity Unity Editor Integration

Unity editor integration for Antigravity. Adds support for generating csproj files for IntelliSense, auto-discovery of installations, workspace management, and more.

## Features

- ✅ Automatic Antigravity installation detection
- ✅ C# project file generation (.csproj, .sln)
- ✅ Workspace support
- ✅ Reuse existing window option
- ✅ Unity debugger integration
- ✅ Cross-platform support (Windows, macOS, Linux)

## How to Install

1. Open Unity Editor
2. Go to **Window → Package Manager**
3. Click the **"+"** button at the top left corner
4. Select **"Add package from git URL"**
5. Insert: `https://github.com/yourusername/com.unity.ide.antigravity.git`
6. Click **"Add"**
7. Done!

## Configuration

After installation:

1. Go to **Edit → Preferences → External Tools**
2. Select **Antigravity** as your External Script Editor
3. (Optional) Enable **"Reuse existing Antigravity window"** to open files in the current window

## Requirements

- Unity 2019.4 or later
- Antigravity Editor installed on your system

## Supported Platforms

- Windows
- macOS
- Linux

## Troubleshooting

### Antigravity not detected
- Ensure Antigravity is installed in the standard location
- Check if the executable is in your system PATH

### Files not opening
- Verify Antigravity is set as the default external editor in Unity preferences
- Regenerate project files: **Assets → Open C# Project**

## License

MIT License - See LICENSE.md for details

## Credits

Based on the official Unity Visual Studio Code integration package and Cursor integration by boxqkrtm.
