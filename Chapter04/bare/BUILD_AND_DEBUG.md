# Build and Debug Guide: Renode vs Hardware with J-Link

This project now supports two distinct development workflows:

1. **Renode Simulation** - For software development and testing without hardware
2. **Hardware Debugging with J-Link** - For on-board debugging with your STM32 Discovery/Nucleo board

## Project Structure

```
build/
├── Debug/                    # Renode simulator build
│   └── bare.elf
└── Debug-Hardware/           # Hardware/J-Link build  
    └── bare.elf
jlink_scripts/
└── flash.jlink              # J-Link flash script
renode_scripts/
├── stm32f072.resc           # Renode run script
└── stm32f072_debug.resc     # Renode debug script
```

## Workflow 1: Renode Simulator

Must open VSCode in folder E:\e_mcu\books\forks\Cpp-in-Embedded-Systems\Chapter04\bare\

### Building for Renode
```bash
# Option 1: Using task
Ctrl+Shift+B > "Build" (default task)

# Option 2: Manual
cmake --preset Debug
cmake --build build/Debug
```

### Debugging in Renode
1. Press `F5` (or Run → Start Debugging)
2. Select **"Debug with Renode"** configuration
3. This will:
   - Build the project
   - Launch Renode simulator
   - Connect GDB debugger to Renode's GDB server (port 3333)
   - Set breakpoint at machine reset
   - Automatically start execution

### Running without Debugging
1. Run task: **"Run Renode"**
2. Renode will start with your firmware loaded

---

## Workflow 2: Hardware Debugging with J-Link

### Prerequisites
- J-Link EDU Mini connected to your STM32 board
- J-Link software installed ([download](https://www.segger.com/downloads/jlink/))
- Ensure `JLinkExe` and `JLinkGDBServer` are in your PATH

### Building for Hardware
```bash
# Option 1: Using task
Ctrl+Shift+B > "Build for Hardware"

# Option 2: Manual with CMake preset
cmake --preset Debug-Hardware
cmake --build build/Debug-Hardware
```

### Step 1: Flash Firmware to Board

**Option A: Flash via Task** (Recommended)
1. Run task: **"Flash with J-Link"**
2. This will:
   - Build the firmware (if needed)
   - Connect to STM32F072CB via J-Link
   - Program Flash memory
   - Verify the download
   - Reset and run the device

**Option B: Manual Flash**
```bash
JLinkExe -device STM32F072CB -if SWD -speed 4000 -autoconnect 1 \
  -CommandFile jlink_scripts/flash.jlink
```

### Step 2: Debug with J-Link GDB Server

**Option A: Debug Configuration (Recommended)**
1. Press `F5` or Run → Start Debugging
2. Select **"Debug with J-Link"** configuration
3. This will:
   - Build the project (if needed)
   - Start J-Link GDB Server on port 2331
   - Connect GDB to the server
   - Load and verify the binary
   - Stop at main()

**Option B: Manual GDB Connection**
```bash
# Terminal 1: Start J-Link GDB Server
JLinkGDBServer -device STM32F072CB -if SWD -speed 4000 -port 2331

# Terminal 2: Connect with GDB
arm-none-eabi-gdb build/Debug-Hardware/bare.elf
(gdb) target remote localhost:2331
(gdb) load
(gdb) monitor reset
(gdb) break main
(gdb) continue
```

### Debugging Features Available
- **Breakpoints**: Set, enable/disable, and conditional breakpoints
- **Step Through Code**: Step into/over/out of functions
- **Variables**: Watch local and global variables
- **Memory**: Inspect memory contents
- **Registers**: View and modify CPU registers
- **Peripheral Registers**: Access STM32 peripheral memory

---

## VS Code Debug Configurations

### Available Configurations (F5)

| Configuration | Target | Purpose | GDB Server |
|---|---|---|---|
| Debug with Renode | Simulator | Step through code in Renode | Renode (port 3333) |
| Run Renode (No Debug) | Simulator | Run firmware in Renode | N/A |
| Debug with J-Link | Hardware | Debug on actual hardware | J-Link (port 2331) |
| Flash and Debug with J-Link | Hardware | Flash + debug workflow | J-Link (port 2331) |

---

## Available Tasks

### Renode Tasks
- **Configure** - Run CMake for Renode debug build
- **Build** - Build for Renode (default task)
- **Run Renode** - Start Renode simulator without debugger
- **Run Renode Debug** - Start Renode with GDB server (used by debug config)
- **Close Renode** - Terminate Renode process

### Hardware/J-Link Tasks
- **Configure Hardware** - Run CMake for hardware debug build
- **Build for Hardware** - Build for J-Link target
- **Flash with J-Link** - Flash firmware via J-Link Commander
- **J-Link GDB Server** - Start J-Link GDB Server (used by debug config)

---

## Environment Variables & Configuration

### J-Link Device Configuration
The following are preconfigured in tasks.json and launch.json:

```json
"device": "STM32F072CB",
"interface": "swd",
"speed": "4000"  // kHz
"port": 2331     // J-Link GDB Server default port
```

**To change device or interface**, edit:
- `.vscode/tasks.json` - Search for `STM32F072CB` or `"device"`
- `.vscode/launch.json` - Search for device configuration

### J-Link Flash Script
Edit `jlink_scripts/flash.jlink` to customize:
- Flash verification behavior
- Post-flash reset settings
- Device security settings

---

## Troubleshooting

### J-Link Connection Issues

**"J-Link not found" or "Cannot connect to device"**
1. Verify USB cable connection
2. Check Device Manager for J-Link COM port
3. Test J-Link directly:
   ```bash
   JLinkExe -autoconnect 1
   ```

**"Unknown device identifier"**
- Verify device name matches your MCU: `STM32F072CB`, `STM32F072C8`, etc.
- Check `STM32F072` in your device datasheet

### GDB Connection Issues

**"Could not connect to localhost:2331"**
1. Ensure "J-Link GDB Server" task is running
2. Check if port 2331 is already in use: `netstat -ano | findstr :2331`
3. Try a different port (update tasks.json input and launch.json)

**"Symbols not loaded" or "Unknown source location"**
1. Ensure ELF file was built with debug symbols: `-g -gdwarf-2` flags
2. Verify executable path in launch.json points to correct build directory

### Build Issues

**"CMake preset not found"**
- Run "Configure" or "Configure Hardware" task first
- Clean build: Delete `build/` directory and reconfigure

---

## Quick Reference

| Action | Method 1 | Method 2 |
|--------|----------|----------|
| **Renode Debug** | F5 → "Debug with Renode" | Task "Run Renode Debug" → GDB attach |
| **Hardware Debug** | F5 → "Debug with J-Link" | Task "J-Link GDB Server" + GDB attach |
| **Flash to Hardware** | F5 → "Flash and Debug with J-Link" | Task "Flash with J-Link" |
| **Build for Renode** | Ctrl+Shift+B | Task "Build" |
| **Build for Hardware** | Ctrl+Shift+B → "Build for Hardware" | Task "Build for Hardware" |

---

## Notes

- Each target has a separate build directory to avoid mixing object files
- Debug symbols are included in both Debug configurations
- J-Link GDB Server port defaults to 2331 (use 3333 as alternative)
- Renode GDB server uses port 3333
- The project supports SWD interface for J-Link (JTAG also available if needed)

