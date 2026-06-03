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

```bash
$ renode --console --disable-xwt renode_scripts/stm32f072.resc -e start
08:14:57.5796 [INFO] Loaded monitor commands from: E:\Program Files\Renode\scrip
ts/monitor.py
Renode, version 1.15.0.30170 (9111b18e-202403181638)

(monitor) i $CWD/renode_scripts/stm32f072.resc
08:14:58.0171 [INFO] Including script: E:\e_mcu\projects\CMake\bare\renode_scrip
ts\stm32f072.resc
08:14:58.0796 [INFO] System bus created.
Starting emulation...
08:15:01.5478 [INFO] cpu: Guessing VectorTableOffset value to be 0x8000000.
08:15:01.5634 [INFO] cpu: Setting initial values: PC = 0x8001379, SP = 0x2000400
0.
08:15:01.5634 [INFO] machine-0: Machine started.
(machine-0) start
Starting emulation...
(machine-0) 08:15:02.0477 [INFO] usart2: [host: 0.83s (+0.83s)|virt: 0s (+0s)] H
ello world !
08:15:02.4070 [INFO] usart2: [host: 1.19s (+0.36s)|virt: 0.11s (+0.11s)] While l
oop 1000 ms ping ...
08:15:02.7351 [INFO] usart2: [host: 1.52s (+0.33s)|virt: 0.22s (+0.11s)] While l
oop 1000 ms ping ...
08:15:03.0476 [INFO] usart2: [host: 1.83s (+0.31s)|virt: 0.34s (+0.11s)] While l
oop 1000 ms ping ...
08:15:03.3756 [INFO] usart2: [host: 2.16s (+0.33s)|virt: 0.45s (+0.11s)] While l
oop 1000 ms ping ...
08:15:03.6881 [INFO] usart2: [host: 2.47s (+0.31s)|virt: 0.56s (+0.11s)] While l
oop 1000 ms ping ...
08:15:04.0162 [INFO] usart2: [host:  2.8s (+0.33s)|virt: 0.67s (+0.11s)] While l
oop 1000 ms ping ...
08:15:04.3286 [INFO] usart2: [host: 3.11s (+0.31s)|virt: 0.78s (+0.11s)] While l
oop 1000 ms ping ...
08:15:04.6567 [INFO] usart2: [host: 3.44s (+0.33s)|virt:  0.9s (+0.11s)] While l
oop 1000 ms ping ...
08:15:04.9691 [INFO] usart2: [host: 3.75s (+0.31s)|virt: 1.01s (+0.11s)] While l
oop 1000 ms ping ...
08:15:05.2816 [INFO] usart2: [host: 4.06s (+0.31s)|virt: 1.12s (+0.11s)] While l
oop 1000 ms ping ...
08:15:05.6097 [INFO] usart2: [host: 4.39s (+0.33s)|virt: 1.23s (+0.11s)] While l
oop 1000 ms ping ...
08:15:05.9221 [INFO] usart2: [host:  4.7s (+0.31s)|virt: 1.35s (+0.11s)] While l
oop 1000 ms ping ...
08:15:06.2502 [INFO] usart2: [host: 5.03s (+0.33s)|virt: 1.46s (+0.11s)] While l
oop 1000 ms ping ...
08:15:06.5626 [INFO] usart2: [host: 5.34s (+0.31s)|virt: 1.57s (+0.11s)] While l
oop 1000 ms ping ...
08:15:06.8751 [INFO] usart2: [host: 5.66s (+0.31s)|virt: 1.68s (+0.11s)] While l
oop 1000 ms ping ...
08:15:07.1876 [INFO] usart2: [host: 5.97s (+0.31s)|virt: 1.79s (+0.11s)] While l
oop 1000 ms ping ...
08:15:07.5156 [INFO] usart2: [host:  6.3s (+0.33s)|virt: 1.91s (+0.11s)] While l
oop 1000 ms ping ...
08:15:07.8281 [INFO] usart2: [host: 6.61s (+0.31s)|virt: 2.02s (+0.11s)] While l
oop 1000 ms ping ...
08:15:08.1405 [INFO] usart2: [host: 6.92s (+0.31s)|virt: 2.13s (+0.11s)] While l
oop 1000 ms ping ...
08:15:08.4530 [INFO] usart2: [host: 7.23s (+0.31s)|virt: 2.24s (+0.11s)] While l
oop 1000 ms ping ...
08:15:08.7654 [INFO] usart2: [host: 7.55s (+0.31s)|virt: 2.35s (+0.11s)] While l
oop 1000 ms ping ...
08:15:09.0779 [INFO] usart2: [host: 7.86s (+0.31s)|virt: 2.47s (+0.11s)] While l
oop 1000 ms ping ...
08:15:09.3903 [INFO] usart2: [host: 8.17s (+0.31s)|virt: 2.58s (+0.11s)] While l
oop 1000 ms ping ...
08:15:09.7028 [INFO] usart2: [host: 8.48s (+0.31s)|virt: 2.69s (+0.11s)] While l
oop 1000 ms ping ...
08:15:10.0153 [INFO] usart2: [host:  8.8s (+0.31s)|virt:  2.8s (+0.11s)] While l
oop 1000 ms ping ...

```


For the debug script
```
"E:/Program Files/Renode/bin/renode.exe" --console --disable-xwt renode_scripts/stm32f072_debug.resc
```
#### Notes:
Run it from bare so the script’s relative path to bare.elf works.
If you want an interactive monitor session instead of auto-starting, omit -e start.


### Using VS Code as per the book

#### VS Code extensions
Using Default profile, with the following extensions
```Bash
$ code --list-extensions --show-versions
```

```
Installed VS Code Extensions
akiramiyakoda.cppincludeguard@1.8.0
codezombiech.gitignore@0.10.0
github.vscode-github-actions@0.31.5
github.vscode-pull-request-github@0.146.0
marus25.cortex-debug@1.12.1
mcu-debug.debug-tracker-vscode@0.0.15
mcu-debug.memory-view@0.0.29
mcu-debug.peripheral-viewer@1.6.1
mcu-debug.rtos-views@0.0.15
mhutchie.git-graph@1.30.0
ms-vscode.cpptools@1.32.2
ms-vscode.cpptools-extension-pack@1.5.1
ms-vscode.cpptools-themes@2.0.0
ms-vscode.powershell@2025.4.0
phil294.git-log--graph@0.1.35
streetsidesoftware.code-spell-checker@4.5.6
techer.open-in-browser@2.0.0

```

Repeat for another profile
Use the same command plus --profile:

```bash
code --list-extensions --show-versions --profile "PROFILE_NAME"
```

```
Then substitute the profile name, for example:

```bash
code --list-extensions --show-versions --profile "Default"
```

On Windows, this works in Bash or PowerShell as long as code is on your PATH.

I need to open VS Code in the folder bare

I needed to fix the files inside the .vscode folder for Windows paths not linux path

Path escaping issue: The backslashes in Windows paths need to be quoted when passed through bash to prevent them from being interpreted as escape characters.

Missing toolchain and generator configuration: The tasks weren't using the CMake preset, which specifies:

The Ninja generator (instead of defaulting to NMake)
The ARM GCC toolchain file

I need to click the debug anyway popup

When running CMake through Git Bash on Windows, paths with backslashes get mangled because bash interprets backslashes as escape characters.

Error: `CMake Error: The source directory "E:e_mcubooksforksCpp-in-Embedded-..." does not exist.`

#### Solution
1. **Quote paths in task args**: Wrap `${workspaceFolder}` paths in double quotes in tasks.json
   - Bad: `"${workspaceFolder}"`
   - Good: `"\"${workspaceFolder}\""`

2. **Use CMake presets**: Instead of manual `-S` and `-B` flags, use `cmake --preset Debug`
   - Presets automatically handle toolchain files and generators
   - Generated build dir follows preset naming: `build/Debug/` instead of `build/`

3. **Update build tasks**: Use `cmake --build build/Debug` instead of `make -C build`
   - Works across different generators (Ninja, Make, etc.)

##### Files Changed
- `.vscode/tasks.json`: Updated Configure and Build tasks to use presets and quoted paths

Now launch tasks **Debug application in Renode** and **Run application without debugging** work as the book. 

<!-- ### Depreciated commands while trying to fix the issues
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


Perfect! The CMake configuration now works successfully. The issues were:

Path escaping issue: The backslashes in Windows paths need to be quoted when passed through bash to prevent them from being interpreted as escape characters.

Missing toolchain and generator configuration: The tasks weren't using the CMake preset, which specifies:

The Ninja generator (instead of defaulting to NMake)
The ARM GCC toolchain file
Let me save this fix info for future reference:
Created memory file C:\Users\amfly_1n0g7qu\AppData\Roaming\Code\User\workspaceStorage\1fd3658f7273ebb8e77ab0e12ab56e3e\GitHub.copilot-chat\memory-tool\memories\repo\cmake-windows-bash-fix.md
The CMake configuration is now working correctly! Your tasks have been updated to:

Use CMake presets (which automatically applies the correct Ninja generator and ARM toolchain)
Use cmake --build for building (compatible across different generators)
Handle Windows paths correctly in the Git Bash environment
You can now run the Build task to compile your project.


Executing task: cmake -S E:\e_mcu\books\forks\Cpp-in-Embedded-Systems\Chapter04\bare -B E:\e_mcu\books\forks\Cpp-in-Embedded-Systems\Chapter04\bare/build -DCMAKE_BUILD_TYPE=Debug 

CMake Error: The source directory "E:e_mcubooksforksCpp-in-Embedded-SystemsChapter04bare" does not exist.
Specify --help for usage, or press the help button on the CMake GUI.

 *  The terminal process "C:\Program Files\Git\usr\bin\bash.exe '--login', '-i', '-c', 'cmake -S E:\e_mcu\books\forks\Cpp-in-Embedded-Systems\Chapter04\bare -B E:\e_mcu\books\forks\Cpp-in-Embedded-Systems\Chapter04\bare/build -DCMAKE_BUILD_TYPE=Debug'" terminated with exit code: 1. 
 *  Terminal will be reused by tasks, press any key to close it.  -->