---
layout: post
title: "Compile project from windows terminal"
---

### 1. Download vs_BuildTools.exe (According to the time, choose the version you need)
Go to [here](https://visualstudio.microsoft.com/downloads/#build-tools-for-visual-studio-2022) download it.

### 2. Install necessary component (According to your system, choose the version you need)
* MSVC v143 - VS 2022 C++ x64/x86
* Windows 11 SDK
* CMake (Optional)

For example:<br/>
![MSBuild_Tools](/MyBlog/assets/picture/MSBuild_Tools.png){:class="img-responsive"}

### 3. Open terminal
After installed, you should find **Developer PowerShell for VS 2022**, click the command, get into the terminal.

### 4. Generate .sln project by CMake (Optional)
Learn [cmake Usage](https://cmake.org/documentation/).<br/>
Or you can simply go to the directory contains CMakeLists.txt , run ```cmake .``` to generate .sln project.

### 5. Compile .sln project
* ```MSBuild MyApp.sln -p:Configuration=Release``` (Release Version)
* ```MSBuild MyApp.sln``` (Debug Version)

***Note: You can get more info by ```MSBuild --help```***

### Finally you get the executable file can run in windows.
