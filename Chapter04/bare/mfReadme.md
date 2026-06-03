## mfReadme.md

- Original code from [github.com | Cpp-in-Embedded-Systems](https://github.com/PacktPublishing/Cpp-in-Embedded-Systems)
- Extracted to E:\e_mcu\books\Cpp-in-Embedded-Systems\Cpp-in-Embedded-Systems-main
- This project copied from  \Chapter04\bare

### Added files to project from an STM32CubeMX CMake project mfBlinky
- Moved files from a STM32CubeMX CMake project mfBlinky folders
- E:\e_mcu\projects\CMake\mfBlinky\CMakePresets.json 
- E:\e_mcu\projects\CMake\mfBlinky\cmake\gcc-arm-none-eabi.cmake

### Building firmware using CMake, Ninja and Git Bash on Windows10



### CoPilot Build Fix Summary
1. Fixed CMakePresets.json toolchain path
    - As I moved the gcc-arm-none-eabi.cmake into the root from cmake folder
    - Changed from ${sourceDir}/cmake/gcc-arm-none-eabi.cmake to ${sourceDir}/gcc-arm-none-eabi.cmake
2. Updated toolchain file header
    - Added CMAKE_SYSTEM_NAME Generic and CMAKE_SYSTEM_PROCESSOR arm to declare cross-compilation (prevents CMake from treating it as a Windows build)
3. Removed conflicting MCU flags from toolchain
    - Deleted hardcoded Cortex-M4 and FPU flags (-mcpu=cortex-m4 -mfpu=fpv4-sp-d16 -mfloat-abi=hard) that conflicted with STM32F072's Cortex-M0 without FPU
    - Eliminated the "Compiler generates FPU instructions for a device without an FPU" error
4. Removed duplicate --specs=nano.specs from toolchain
    - It appeared twice and caused linker spec redefinition error
5. Cleared toolchain linker flags
    - Set CMAKE_EXE_LINKER_FLAGS to empty; linker flags now defined per-target in CMakeLists.txt to avoid duplication
6. Renamed executable target
    - Changed from bare.elf to bare so the toolchain's .elf suffix produces correct output instead of bare.elf.elf
7. Fixed POST_BUILD commands
    - Updated to use $<TARGET_FILE:${EXECUTABLE}> generator expression so size/objcopy commands reference the actual output file path with .elf suffix


### Refactoring and duplicate removal
1. Refactoring of gcc-arm-none-eabi.cmake
    - Kept only toolchain environment and generic compiler defaults:
    - CMAKE_C_COMPILER, CMAKE_CXX_COMPILER, CMAKE_ASM_COMPILER
    - CMAKE_LINKER, CMAKE_OBJCOPY, CMAKE_SIZE
    - executable suffixes
    - default generic warning/optimization flags
    - Removed project-specific settings from this file
2. Refactoring of CMakeLists.txt
    - Kept only project-specific build settings:
    - -mcpu=cortex-m0, -mthumb
    - -DUSE_HAL_DRIVER, -DSTM32F072xB
    - warning suppressions and HAL-specific options
    - target definition, linker script, and post-build commands
3. Removed duplicated compiler/toolchain program settings and CMAKE_TRY_COMPILE_TARGET_TYPE



- The build now completes successfully with bare.elf produced at 7864 bytes total.
- After many modifications to gcc-arm-none-eabi and CMakeLists.txt I successfully compiled this project

- Duplicate and conflicting settings are separated cleanly:
    - gcc-arm-none-eabi.cmake = toolchain / environment
    - CMakeLists.txt = project / MCU-specific flags and link options

```bash
$ rm -rf build
$ cmake --preset Debug
$ cmake --build build/Debug
```

### Renode-related changes
1. Enabled the Renode path setting in CMakeLists.txt
2. Changed RENODE from "renode" to your absolute path:
    - E:/Program Files/Renode/bin/renode.exe
3. Kept the status message so configure output now shows:
    - Using Renode at: E:/Program Files/Renode/bin/renode.exe
4. Left the run_in_renode custom target active:
    - runs renode --console --disable-xwt .../renode_scripts/stm32f072.resc -e start
    - depends on ${PROJECT_NAME}.elf
5. Result
    - run_in_renode now uses the explicit Renode executable path instead of relying on renode being on PATH
    - This should allow cmake --build build/Debug --target run_in_renode to find and launch Renode correctly


### Renode script changes
- Updated stm32f072.resc
- Updated stm32f072_debug.resc

What changed
- Original path:
    - $bin?=$ORIGIN/../build/bare.elf
- New path:
    - $bin?=$ORIGIN/../build/Debug/bare.elf
Why
- Your build output is generated under Debug
- Renode now loads the actual ELF file location correctly


#### Successful output
```bash
$ cmake --build build/Debug --target run_in_renode
[1/1] C:\WINDOWS\system32\cmd.exe /C "cd /D E:\e_mcu\projects\CMak.../e_mcu/projects/CMake/bare/renode_scripts/stm32f072.resc -e start
18:51:06.8483 [INFO] Loaded monitor commands from: E:\Program Files\Renode\scripts/monitor.py
Renode, version 1.15.0.30170 (9111b18e-202403181638)

(monitor) i @E:/e_mcu/projects/CMake/bare/renode_scripts/stm32f072.resc
18:51:07.3163 [INFO] Including script: E:\e_mcu\projects\CMake\bare\renode_scripts\stm32f072.resc
18:51:07.3793 [INFO] System bus created.
Starting emulation...
18:51:11.3003 [INFO] cpu: Guessing VectorTableOffset value to be 0x8000000.
18:51:11.3213 [INFO] cpu: Setting initial values: PC = 0x8001379, SP = 0x20004000.
18:51:11.3263 [INFO] machine-0: Machine started.
(machine-0) start
Starting emulation...
(machine-0) 18:51:11.7953 [INFO] machine-0: Machine paused.
18:51:11.8943 [INFO] machine-0: Disposed.
```

### Run Renode directly
From your workspace root:

```
"E:/Program Files/Renode/bin/renode.exe" --console --disable-xwt renode_scripts/stm32f072.resc -e start
```
If renode is on your PATH:

```
renode --console --disable-xwt renode_scripts/stm32f072.resc -e start
```

For the debug script
```
"E:/Program Files/Renode/bin/renode.exe" --console --disable-xwt renode_scripts/stm32f072_debug.resc
```
#### Notes:
Run it from bare so the script’s relative path to bare.elf works.
If you want an interactive monitor session instead of auto-starting, omit -e start.

### Depreciated commands while trying to fix the issues
- If a CMakePresets.json file does not exist and compiler not found by path
- Explicit form:
```bash
$ cmake -DCMAKE_TOOLCHAIN_FILE=cmake/gcc-arm-none-eabi.cmake -B build/Debug -GNinja
$ cmake --build build/Debug
```


```
cmake -S . -B build -G Ninja -DCMAKE_MAKE_PROGRAM=/c/mingw64/bin/ninja.exe
cmake --build build
```

This folder contains a fixed CMakeLists.txt for Windows10 using CoPilot

```bash
$ mkdir build && cd build
$ cmake .. -DCMAKE_BUILD_TYPE=Debug
```