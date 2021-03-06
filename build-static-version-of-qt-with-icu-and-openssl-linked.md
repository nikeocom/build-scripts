﻿# How to build static version of Qt with ICU and OpenSSL linked

**For Windows platform and VS developers ONLY !!!**

**You have to install [Active Perl](https://www.activestate.com/activeperl/downloads), [Python 2](https://www.python.org/downloads/windows/) and [Visual Studio](https://www.visualstudio.com/downloads/) first.**

## Build static version of ICU

**From ICU 64, it requires Python 3 to be built.**

32-bit example:
1. [Download the latest MSYS2](http://www.msys2.org/) and install it. I assume you have installed it to it's default location "C:\msys64".
2. Open *MSYS2 MSYS* shell and execute the following commands:
   ```bash
   pacman -Syu
   ```
   Close the shell window forcely and open *MSYS2 MSYS* shell again, execute the following commands:
   ```bash
   pacman -Su
   pacman -S make binutils python3
   ```
3. Rename "C:\msys64\usr\bin\link.exe" to "link.exe.bak" (only if you have this file).
4. [Download the latest ICU source code package](http://site.icu-project.org/download) and extract it to anywhere you like. I assume you have extracted it to "C:\ICU\src".
5. Open cmd shell and execute the following commands (I assume you have installed VS2017 Community to it's default localtion "C:\Program Files (x86)\Microsoft Visual Studio\2017\Community"):
   ```bat
   REM 64 bit: use "x64" instead of "x86"
   call "C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Auxiliary\Build\vcvarsall.bat" x86
   REM 64 bit: use "-mingw64" instead of "-mingw32"
   call "C:\msys64\msys2_shell.cmd" -mingw32 -use-full-path
   ```
   Then execute the following commands:
   ```bash
   cd /C/ICU/src/source
   ./runConfigureICU MSYS/MSVC --enable-static --disable-shared --prefix=$PWD/../../icu4c-x86-static-msvc2017
   make -j4 && make install
   ```
   ICU will link to shared MSVCRT(-MD) for both shared and static builds by default, if you want to link against static MSVCRT, add `CFLAGS=-MT CXXFLAGS=-MT` to the configure parameters.

## Build static version of OpenSSL

[Just use the script I offered.](https://github.com/wangwenx190/build-scripts/blob/master/win-build-openssl.bat)

## Build static version of Qt

1. [Download the latest Qt source code package](http://download.qt.io/official_releases/qt/) and extract it to anywhere you like. I assume you have extracted it to "C:\Qt\src".
2. Open "C:\Qt\src\qtbase\mkspecs\common\msvc-desktop.conf", add "**U_STATIC_IMPLEMENTATION**" to "**DEFINES +=**" (this step is for static version of ICU only, if you don't need ICU, please ignore this step).
3. [Use the script I offered to generate the batch script file](https://github.com/wangwenx190/build-scripts/blob/master/win-build-qt.bat), add **-static-runtime** to configure parameters if you want to link to static MSVCRT.
4. Open the generated *.bat* file, add the following lines to it and put them to their proper position (you may have to change a little bit if necessary):
   ```bat
   SET "_ICU_DIR=C:\ICU\icu4c-static"
   SET "_OPENSSL_DIR=C:\OpenSSL\openssl-static"
   SET "PATH=%_ICU_DIR%\bin;%_OPENSSL_DIR%\bin;%PATH%"
   CALL "configure.bat" -icu -I "%_ICU_DIR%\include" -L "%_ICU_DIR%\lib" ICU_LIBS="-lsicudt -lsicuin -lsicuuc -lAdvapi32" -openssl-linked -I "%_OPENSSL_DIR%\include" -L "%_OPENSSL_DIR%\lib" OPENSSL_LIBS="-llibcrypto -llibssl -lgdi32"
   ```
   **NOTICE**: **-lAdvapi32** is necessary for static version of ICU, without this lib, the compilation will fail. **-lgdi32** is the same. Don't remove the two libs.

**NOTE**

If you have compiler errors and the whole compiling process stopped, don't worry, it's OK, just close the batch script window and rerun it.
