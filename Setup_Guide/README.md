

---

````markdown
# 🛠️ STM32 Build System (Bare-Metal with GNU Toolchain)

This guide explains how to prepare your system and build STM32 projects **without STM32CubeIDE**, using only the GNU ARM toolchain and a batch build script.

---

## 1️⃣ Preparation (Before You Begin)

Follow these steps before trying to build your first project.

---

### 🔹 Install Required Tools

#### ARM GCC Toolchain
- Download: [GNU Arm Embedded Toolchain](https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads)  
- Install for your OS (Windows/Linux).  
- Add `bin/` folder (e.g., `C:\Program Files (x86)\GNU Arm Embedded Toolchain\10 2021.10\bin`) to your **PATH**.

Check:
```sh
arm-none-eabi-gcc --version
````

#### STM32 Programmer

* Download: [STM32CubeProgrammer](https://www.st.com/en/development-tools/stm32cubeprog.html)
* Install → gives `STM32_Programmer_CLI` tool.
* Needed for flashing `.bin` to STM32.

Check:

```sh
STM32_Programmer_CLI --version
```

#### Git (Optional, but useful)

* [Download Git](https://git-scm.com/downloads)
* For version control & cloning example projects.

---

### 🔹 Setup Project Template

1. Create a project folder:

   ```sh
   mkdir MyProject
   cd MyProject
   ```

2. Copy this structure:

   ```
   linker/        → .ld linker script
   startup/       → startup .s file
   Src/           → application code
   Inc/           → headers
   Drivers/       → CMSIS + HAL
   build.bat      → build script
   ```

---

### 🔹 Install STM32 HAL & CMSIS

* Download **STM32CubeF1 package**:
  [STM32CubeF1 HAL Drivers](https://www.st.com/en/embedded-software/stm32cubef1.html)

* Copy into project:

  * `Drivers/CMSIS/` → Core + Device headers
  * `Drivers/STM32F1xx_HAL_Driver/` → HAL sources

---

### 🔹 Verify GCC Setup

Create `hello.c`:

```c
int main(void) { return 0; }
```

Compile:

```sh
arm-none-eabi-gcc -mcpu=cortex-m3 -mthumb -c hello.c -o hello.o
```

If no error → setup is correct ✅.

---

### 🔹 Hardware Setup

* STM32 board (Blue Pill, Nucleo, etc.)
* ST-LINK programmer (onboard or external)
* USB cable (for power + flashing)

---

### 🔹 Recommended Editor

* **VS Code** with `C/C++` + `ARM Assembly` extensions
* Or **CLion / Eclipse** if you prefer IDEs

---

## 2️⃣ Build Guide (Using build.bat)

Once preparation is done, follow this to build your project.

---

### 🔹 Build Script Overview

File: `build.bat`

```bat
@echo off
setlocal

REM ---- Get current folder name as project name ----
for %%I in (.) do set ProjectName=%%~nxI

REM ---- Clean + create output folder ----
if exist Build rmdir /s /q Build
mkdir Build

REM ---- Toolchain ----
set CC=arm-none-eabi-gcc
set OBJCOPY=arm-none-eabi-objcopy
set SIZE=arm-none-eabi-size

REM ---- Includes + defines ----
set INCS=-IInc -IDrivers\CMSIS\Core\Include -IDrivers\CMSIS\Include -IDrivers\CMSIS\Device\ST\STM32F1xx\Include -IDrivers\STM32F1xx_HAL_Driver\Inc
set DEFS=-DUSE_HAL_DRIVER -DSTM32F103xB
set CFLAGS=-mcpu=cortex-m3 -mthumb -O2 -ffunction-sections -fdata-sections -g3 -Wall %INCS% %DEFS%
set LDFLAGS=-T linker\stm32f103c8t6.ld -Wl,--gc-sections -specs=nosys.specs

echo.
echo === Compiling application sources (Src/*.c) ===
for %%f in (Src\*.c) do (
    echo Compiling %%f ...
    %CC% %CFLAGS% -c %%f -o Build\%%~nf.o
    if %ERRORLEVEL% neq 0 ( echo Compile failed: %%f & pause & exit /b 1 )
)

echo.
echo === Compiling startup code ===
%CC% %CFLAGS% -c startup\startup_stm32f103xb.s -o Build\startup_stm32f103xb.o
if %ERRORLEVEL% neq 0 ( echo Compile failed: startup_stm32f103xb.s & pause & exit /b 1 )

echo.
echo === Compiling HAL sources ===
for %%f in (Drivers\STM32F1xx_HAL_Driver\Src\*.c) do (
    echo Compiling %%f ...
    %CC% %CFLAGS% -c %%f -o Build\%%~nf.o
    if %ERRORLEVEL% neq 0 ( echo Compile failed: %%f & pause & exit /b 1 )
)

echo.
echo === Linking ===
%CC% -mcpu=cortex-m3 -mthumb Build\*.o -o Build\%ProjectName%.elf %LDFLAGS%
if %ERRORLEVEL% neq 0 ( echo Link failed & pause & exit /b 1 )

%OBJCOPY% -O binary Build\%ProjectName%.elf Build\%ProjectName%.bin
%SIZE% Build\%ProjectName%.elf

echo.
echo Build succeeded: Build\%ProjectName%.bin
pause
endlocal
```

---

### 🔹 How to Build

1. Open **Command Prompt**.
2. Navigate to project folder:

   ```sh
   cd MyProject
   ```
3. Run the script:

   ```sh
   build.bat
   ```
4. Output files will be in `/Build`:

   * `MyProject.elf` → full debug build
   * `MyProject.bin` → flashable image

---

### 🔹 Flash to Board

Example (ST-LINK, Blue Pill):

```sh
STM32_Programmer_CLI -c port=SWD -w Build\MyProject.bin 0x08000000 -rst
```

---

✅ Done! You now have a **reusable bare-metal build system** for STM32 with GCC + batch script.

```
