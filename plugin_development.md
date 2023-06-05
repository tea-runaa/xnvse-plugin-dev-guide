# xNVSE Plugin Development Guide

## Overview
---
New Vegas Script Extender is a powerful tool that extends Fallout: New Vegas's functionality, offering a wide variety of new script commands that allow mod authors to add many advanced features to their mods. For those who wish to expand the game's scripting capabilities or make edits to the game's code that go beyond what .esp and .esm files can do, xNVSE allows authors to create their own plugins using C++. This guide goes through the source code of the xNVSE example plugin, breaking down and explaining how plugins interface with xNVSE and how script commands are made.

<br>

## Getting Support
---
xNVSE has an active Discord server ([discord.gg/EebN93s](https://discord.gg/EebN93s)) with people who have experience developing plugins. [#plugin-development](https://discord.com/channels/711228477382328331/715866346763583569) is a channel that is dedicated to discussion and support surrounding the creation of xNVSE plugins. Before asking for help, you should first try to troubleshoot the problem on your own or look for an existing discussion surrounding the problem. Make good use of Google or the Discord/Github search feature to find information, and look through the source code of xNVSE or existing plugins to see how their authors implemented a feature or solved a problem. [JIP](https://github.com/jazzisparis/JIP-LN-NVSE), [JohnnyGuitar](https://github.com/carxt/JohnnyGuitarNVSE), and [ShowOff](https://github.com/Demorome/ShowOff-NVSE) are all open source plugins that utilize many features of xNVSE and may be helpful to look through.

<br>

## Glossary
---

| Term/Acronym | Expansion | Meaning
|-|-|-|
| FNV | Fallout: New Vegas |
| MO2 | Mod Organizer 2 |
| VS | Visual Studio |
| MSVC | Microsoft Visual C++ | The C++ compiler used by VS. |

Look through this [wiki article](https://geckwiki.com/index.php?title=Glossary) for more technical terms used when making mods for FNV.

<br>

## Skill Requirements
---
This guide is solely dedicated to understanding how the example plugin works. More detail on certain concepts, such as installing mods or the basics of using Visual Studio, are not covered for the sake of brevity and because more thorough guides dedicated to learning those skills already exist.

* An intermediate level of C++ understanding is recommended before attempting to develop a plugin. You should not attempt to learn C++ by making a plugin, as the xNVSE source code uses many features of the language that may be overwhelming to beginners that are trying to understand basic concepts. This tutorial assumes that you have adequate knowledge of C++ and will not be covering the basics of the language.

* A good understanding of how to use a computer is necessary. You will need to be able to find certain directories, extract files, install programs, create environment variables, and more.

* Knowledge of both installing and creating basic FNV mods is required. Being able to install both .esp/.esm plugins and NVSE plugins, and being able to use a mod manager (MO2 specifically) are both required skills needed to proceed with this guide. It is also assumed that you know how to use the GECK, know what forms/refs are and how they work, and that you have a basic understanding of the game's scripting language (Obscript).

* Basic knowledge of how to use VS 2022.

<br>

## Software Requirements
---

* A legitimate copy of Fallout: New Vegas from Steam or GOG, updated to the latest version. Copies of the game obtained from the Epic Games Store, Microsoft Store, or from any other sources are not supported by xNVSE. The German "No Gore" edition is not supported either. The GOG version is highly recommended, as the steam version will prevent debuggers from attaching themselves to the game process, making debugging much more difficult.

* An x64 version of Windows 10 or 11 - Linux, earlier installs of Windows, 32-bit operating systems, and anything on another architecture may work, but this tutorial was made using the x64 version of Windows 10. Any issues that may arise from using other operating systems or platforms are not covered here.

* [7zip](https://www.7-zip.org/) - Needed to extract xNVSE from its archive.

* [The latest version of xNVSE](https://github.com/xNVSE/NVSE/releases) - Required to load and run xNVSE plugins.

* [Microsoft Visual Studio 2022](https://visualstudio.microsoft.com/) - This tutorial was made with the free community edition, but the professional and enterprise editions will work as well. Older versions of VS may work, but may require some extra configuration steps in order to get your plugin to compile, which are not covered here.

* [Microsoft Visual C++ Redistributables 2013 - 2022 (x86)](https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist) - Runtime libraries for Visual Studio apps.

* [x64dbg](https://x64dbg.com/) - A debugger that will allow you to see a disassembly of the game and its plugins, view its memory, along with many other essential features needed for plugin development. Other debuggers like IDA will also work, but this tutorial assumes that you are using xdbg.

* [Mod Organizer 2](https://www.nexusmods.com/skyrimspecialedition/mods/6194) - MO2 is a mod manager that allows you to create and manage different mod configurations while also keeping them separate from eachother, making it easier to create a clean development environment. Manual mod management or other mod managers may work, but they will not be covered here.

* **(Optional)** [Visual Studio Code](https://code.visualstudio.com/) - While Visual Studio provides a means of editing code, you may have a more comfortable experience using a dedicated editor rather than a full fledged IDE. They are typically more lightweight (quicker to open and get straight into editing) than Visual Studio, and provide a better UI for editing files rather than full project management. VSCode is not required however, and you can substitute it with an editor of your preference.

<br>

## Creating a Development Environment
---

1. Ensure you have a clean install of FNV outside of a default Windows folder with the latest version of xNVSE. If you have mods installed manually (any non vanilla files in the root or data folder) or aren't sure whether or not FNV is installed in a Windows folder, follow the [initial setup](https://vivanewvegas.moddinglinked.com/setup.html) of Viva New Vegas to get the game set up properly. Having New Vegas Heap Replacer (d3dx9_38.dll in the game's root folder) is fine and does not warrant a new install by itself.

2. Open MO2 and create a new instance for FNV. This keeps your plugin and any mods you use for testing separate from the mods you play with.

3. Install and enable the recommended mods (make sure to follow any instructions provided on their page) through MO2. These mods make it easier to playtest your plugin by improving performance and stability, as well as adding some extra features that may be useful in testing. They will not interfere with the example plugin's behaviour.

<br>

### Recommended Mods

* [4GB Patcher](https://www.nexusmods.com/newvegas/mods/62552) - Expands the amount of memory available to the game and autoloads xNVSE. Must be installed manually.

* [New Vegas Heap Replacer](https://www.nexusmods.com/newvegas/mods/69779) - Replaces the game's heap with a more optimized version for performance. Must be installed manually.

* [GECK Extender](https://www.nexusmods.com/newvegas/mods/64888) - Does what 4GB Patcher does but to the GECK, along with many bug fixes and improvements. The original GECK and new GECK executable provided must be installed manually.

* [Crash Logger](https://www.nexusmods.com/newvegas/mods/72317) - Logs useful data whenever the game crashes.

* [JIP](https://www.nexusmods.com/newvegas/mods/58277) - Used here to allow `FalloutCustom.ini` to work.

* [JohnnyGuitar](https://www.nexusmods.com/newvegas/mods/66927) - Used to here to enable runtime editor IDs, which makes it easier to work with forms and references in the console.

* [New Vegas Tick Fix](https://www.nexusmods.com/newvegas/mods/66537) - Improves performance and reduces stuttering. Install the [Viva Default NVTF Preset](https://www.nexusmods.com/newvegas/mods/81231) unless you know how to configure it properly.

* [OneTweak](https://www.nexusmods.com/newvegas/mods/79211) - Allows the game to run in borderless window mode, which makes it easier to switch between the game and other programs (such as a debugger) while the game is running.

* [Infinite Loading Screen Fix](https://www.nexusmods.com/newvegas/mods/70964) - Improves loading times and prevents deadlocks during loading.

* [Mod Limit Fix](https://www.nexusmods.com/newvegas/mods/68714) - May further improve performance and loading times.

* [Improved Console](https://www.nexusmods.com/newvegas/mods/70801) - Prints results from all script commands when used in the console, along with other improvements.

* [Console Paste Support](https://www.nexusmods.com/newvegas/mods/65906) - Adds copy/paste support as well as other hotkeys to the console.

<br>

4. Install [Microsoft Visual C++ Redistributables 2013 - 2022 (x86)](https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist). Select the x86 version of the 2015 - 2022 redists and the 2013 redist.

5. Scroll down into the "INI Tweaks" section of this [Viva New Vegas page](https://vivanewvegas.moddinglinked.com/mo2.html) and paste the block of ini settings into the `FalloutCustom.ini` file of your current Mod Organizer 2 instance.

6. Download and install [Microsoft Visual Studio Community Edition 2022](https://visualstudio.microsoft.com/) (as stated earlier, professional and enterprise editions will work, but are not free). You will eventually come across a screen that allows you to choose workloads to install. Make sure to select "Desktop Development C++", which will select and install all components necessary for C++ development.

7. Download the source code .zip file of the latest release of [xNVSE](https://github.com/xNVSE/NVSE/releases). Extract it into a folder of your choice.

8. Open Visual Studio, select `Open a project or solution`, and navigate to the folder where you extracted the contents of the archive from the last step. Go into the `nvse_plugin_example` folder and select `nvse_plugin_example.sln`.

9. Before examining the actual project, there is a build step that needs to be resolved. There are two ways that you can do this.

    - **The Recommended Way**
        - Go to the post-build event section of the project configuration menu (make sure to set the configuration to `All Configurations`) and replace the build step with the text in the code block under `Recommended Way Command Snippet`. Next, create a user environment variable called `MO2FNVModsPath` with the path of the `mods` subfolder of the folder that you created your new Mod Organizer 2 instance in. When you build your plugin, a new mod entry is created for MO2 which contains the plugin .dll file. This lets you quickly and easily enable your plugin and keeps it confined in your dev configuration.

    <br>

    - **The Classic Way**
        - Create two user environment variables called `FalloutNVPath` and `FalloutNVPathD` with the content of both being a path to the FNV root folder. When you build a plugin, it will be copied directly into your game's NVSE plugins folder. This is slightly simpler and is how the build step is set up by default, but manually installs the newly built plugin. This means that if you wish to switch to a gameplay instance of FNV and don't want to use your plugin, you will need to go into the NVSE plugins folder and remove it manually.

    <br>

    ### Recommended Way Command Snippet

    ```bat
    mkdir "$(MO2FNVModsPath)\$(ProjectName)\nvse\plugins"
    copy "$(TargetPath)" "$(MO2FNVModsPath)\$(ProjectName)\nvse\plugins\$(TargetFileName)"
    ```

    <br>

10. Save your changes (`File` -> `Save All`).

11. Build the solution. If all steps were followed and everything was set up properly, the project should be built successfully. Some warnings may appear, but you can safely ignore them.

12. Launch the game through MO2 (make sure to enable the new `example_plugin` mod if you used the recommended way in step 10). Press the `~` key to open the console and type `IsDLLLoaded example_plugin`. If the console prints a 1, that means that everything has been done properly so far.

<br>

## Analyzing the Example Plugin By File
---

<br>

### dllmain.c
- This is a file made by VS for any DLL project which contains the DLL entry point. xNVSE does not normally use this entry point for loading plugins, so this file can be ignored.

### exports.def
- A file that provides information to the linker, used in the example plugin to export the query and load symbols (covered in the next section). There are no extra symbols to export or any necessary data to add for the purpose of this guide, so this file can be ignored.

### main.cpp
- The main file where all of the code involved with loading the plugin and registering script commands is located.

    <br>

    ### The Includes
    ---
    The main file includes files that are necessary to interface with xNVSE and create/register commands. It also includes some files that contain script command examples that are separate from the main file to reduce its length.
    ```cpp
    #include "nvse/PluginAPI.h"
    #include "nvse/CommandTable.h"
    #include "nvse/GameAPI.h"
    #include "nvse/ParamInfos.h"
    #include "nvse/GameObjects.h"
    #include <string>
    ```
    ```cpp
    #include "fn_intro_to_script_functions.h" 
    #include "fn_typed_functions.h"
    ```
    | Include Path | Purpose |
    |-|-|
    | `"nvse/PluginAPI.h"` | Provides class definitions for some interfaces that xNVSE gives to plugins. Includes definitions for `NVSEInterface`, `PluginInfo`, `NVSEMessagingInterface`, and more, which are required for the plugin to properly interact with xNVSE. |
    | `"nvse/CommandTable.h"` | Provides command related macros and structures. Allows commands to be defined and registered in game. |
    | `"nvse/GameAPI.h"` | Defines many general structures, functions, and addresses used by the game. |
    | `"nvse/ParamInfos.h"` | Defines command parameter information, which is used to determine what kind of arguments are to be passed to a command. |
    | `"nvse/GameObjects.h"` | Contains class definitions for game forms and refs. |
    | `"fn_intro_to_script_functions.h" ` | A header specific to the example plugin that provides examples of script commands. |
    | `"fn_typed_functions.h"` | A header specific to the example plugin that provides more sophisticated examples of script commands that return certain values when called. |

    <br>

    ### Global Variables
    ---
    ```cpp
    IDebugLog		gLog("nvse_plugin_example.log");
    PluginHandle	g_pluginHandle = kPluginHandle_Invalid;

    NVSEMessagingInterface* g_messagingInterface{};
    NVSEInterface* g_nvseInterface{};
    NVSECommandTableInterface* g_cmdTableInterface{};

    #if RUNTIME
    NVSEScriptInterface* g_script{};
    NVSEStringVarInterface* g_stringInterface{};
    NVSEArrayVarInterface* g_arrayInterface{};
    NVSEDataInterface* g_dataInterface{};
    NVSESerializationInterface* g_serializationInterface{};
    NVSEConsoleInterface* g_consoleInterface{};
    NVSEEventManagerInterface* g_eventInterface{};
    bool (*ExtractArgsEx)(COMMAND_ARGS_EX, ...);
    #endif
    ```

<br>