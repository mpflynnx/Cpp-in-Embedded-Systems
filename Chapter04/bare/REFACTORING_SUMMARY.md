# Project Refactoring Summary

## Changes Made

### 1. **CMakePresets.json** - Added Hardware Presets
- New preset: `Debug-Hardware` (separate build dir: `build/Debug-Hardware/`)
- New preset: `Release-Hardware` (separate build dir: `build/Release-Hardware/`)
- Renamed "Debug" preset description to clarify it's for Renode

### 2. **.vscode/tasks.json** - Added J-Link Tasks
New tasks added:
- **Configure Hardware** - CMake configure for hardware target
- **Build for Hardware** - Build for J-Link target
- **Flash with J-Link** - Flash firmware using JLinkExe
- **J-Link GDB Server** - Start J-Link GDB Server on port 2331
- New input picker for selecting GDB server port (2331 or 3333)

### 3. **.vscode/launch.json** - Added J-Link Configurations
Renamed and added configurations:
- "Debug with Renode" (was "Debug application in Renode")
- "Run Renode (No Debug)" (was "Run application without debugging")
- **"Debug with J-Link"** - Connect to existing J-Link GDB server
- **"Flash and Debug with J-Link"** - Flash first, then debug

### 4. **.vscode/BUILD_AND_DEBUG.md** - Comprehensive Guide
Complete documentation covering:
- Both workflows (Renode vs Hardware)
- Step-by-step instructions
- Task reference
- Debug configurations
- Troubleshooting

### 5. **jlink_scripts/flash.jlink** - J-Link Flash Script
Commander script for programming Flash memory with verification

## Key Architecture Changes

```
Old Structure:
- Single build: build/Debug/bare.elf
- Renode-only debug config

New Structure:
- Renode build: build/Debug/bare.elf
- Hardware build: build/Debug-Hardware/bare.elf
- Dual debug configs: Renode & J-Link
- Flexible task-based build/debug selection
```

## Usage Quick Start

### For Renode
```bash
# Press F5, select "Debug with Renode"
# Or run task: "Run Renode Debug"
```

### For Hardware with J-Link
```bash
# Press F5, select "Debug with J-Link"
# Or manually:
#   1. Run task: "Flash with J-Link"
#   2. Run task: "J-Link GDB Server"
#   3. Press F5, select "Debug with J-Link"
```

## Device Configuration

Located in:
- `.vscode/tasks.json` (lines with `-device STM32F072CB`)
- `.vscode/launch.json` (device properties)
- `jlink_scripts/flash.jlink` (implicit)

Change from `STM32F072CB` to your actual part number if using different variant.

## CMake Build Paths

The build output structure is now:
- Renode: `build/Debug/bare.elf`
- Hardware: `build/Debug-Hardware/bare.elf`
- Release: `build/Release/bare.elf`
- Release HW: `build/Release-Hardware/bare.elf`

