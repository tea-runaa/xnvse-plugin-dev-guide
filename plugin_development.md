# xNVSE Plugin Development Guide

## Overview
New Vegas Script Extender is a powerful tool that extends Fallout: New Vegas's functionality, offering a wide variety of new script commands that allow mod authors to add many advanced features to their mods. For those who wish to expand the game's scripting capabilities or make edits to the game's code that go beyond what .esp and .esm files can do, xNVSE allows authors to create their own plugins using C++. This guide goes through the source code of the xNVSE example plugin, breaking down and explaining how plugins interface with xNVSE and how script commands are made.

<br>

## Getting Support
xNVSE has an active Discord server ([discord.gg/EebN93s](https://discord.gg/EebN93s)) with people who have experience developing plugins. [#plugin-development](https://discord.com/channels/711228477382328331/715866346763583569) is a channel that is dedicated to discussion and support surrounding the creation of xNVSE plugins. Before asking for help, you should first try to troubleshoot the problem on your own or look for an existing discussion surrounding the problem. Make good use of Google or the Discord/Github search feature to find information, and look through the source code of xNVSE or existing plugins to see how their authors implemented a feature or solved a problem. [JIP](https://github.com/jazzisparis/JIP-LN-NVSE), [JohnnyGuitar](https://github.com/carxt/JohnnyGuitarNVSE), and [ShowOff](https://github.com/Demorome/ShowOff-NVSE) are all open source plugins that utilize many features of xNVSE and may be helpful to look through.

<br>

## Glossary

| Term/Acronym | Expansion | Meaning
|-|-|-|
| FNV | Fallout: New Vegas |
| MO2 | Mod Organizer 2 |
| VS | Visual Studio |
| MSVC | Microsoft Visual C++ | The C++ compiler used by VS. |
| Root Folder | | The folder that contains the game's executable (FalloutNV.exe). `(Your Steam Library Path)/steamapps/common/Fallout New Vegas` if you're using the steam version. |
| Data Folder | | The folder that contains the game's data files. It's the folder named `Data` in the root folder.

Look through this [wiki article](https://geckwiki.com/index.php?title=Glossary) for more technical terms used when making mods for FNV.

<br>

## Skill Requirements
This guide is solely dedicated to understanding how the example plugin works. More detail on certain concepts, such as installing mods or the basics of using Visual Studio, are not covered for the sake of brevity and because more thorough guides dedicated to learning those skills already exist.

* An intermediate level of C++ understanding is recommended before attempting to develop a plugin. You should not attempt to learn C++ by making a plugin, as the xNVSE source code uses many features of the language that may be overwhelming to beginners that are trying to understand basic concepts. This tutorial assumes that you have adequate knowledge of C++ and will not be covering the basics of the language.

* A good understanding of how to use a computer is necessary. You will need to be able to find certain directories, extract files, install programs, create environment variables, and more.

* Knowledge of both installing and creating basic FNV mods is required. Being able to install both .esp/.esm plugins and NVSE plugins, and being able to use a mod manager (MO2 specifically) are both required skills needed to proceed with this guide. It is also assumed that you know how to use the GECK, know what forms/refs are and how they work, and that you have a basic understanding of the game's scripting language (Obscript).

* Basic knowledge of how to use VS 2022.

<br>

## Software Requirements

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

### dllmain.c
- This is a file made by VS for any DLL project which contains the DLL entry point. xNVSE does not normally use this entry point for loading plugins, so this file can be ignored.

### exports.def
- A file that provides information to the linker, used in the example plugin to export the query and load symbols (covered in the next section). There are no extra symbols to export or any necessary data to add for the purpose of this guide, so this file can be ignored.

### main.cpp
- The main file where all of the code involved with loading the plugin and registering script commands is located.

    ### The Includes
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
    | `"fn_intro_to_script_functions.h"` | A header specific to the example plugin that provides examples of script commands. |
    | `"fn_typed_functions.h"` | A header specific to the example plugin that provides more sophisticated examples of script commands that return certain values when called. |

    <br>

    ### `#if RUNTIME` and `#if EDITOR`
    xNVSE and the example plugin both have preprocessor directives in certain places that check if `RUNTIME` or `EDITOR` macros are defined. By default, the plugin compiles in runtime mode where the `RUNTIME` macro is defined, but `EDITOR` isn't. You can compile it under editor mode for the opposite effect by switching the build configuration to one that ends in GECK. The plugin can still load and have script commands registered in the GECK when compiled in a runtime configuration, so don't change to a GECK configuration unless you're explicitly planning to make changes to the editor.

    <br>

    ### Global Variables
    Pointers to some interfaces that you may need have been defined in the main file. Keep in mind that, due to the way the project is set up, these variables are in scope for the `"fn_intro_to_script_functions.h"` and `"fn_typed_functions.h"` headers.

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
    | Type | Name | Purpose | Type Declaration Header |
    |-|-|-|-|
    | `IDebugLog` | gLog | Creates an instance of xNVSE's logger. It can be interfaced through functions like `_MESSAGE` or `_ERROR`. The full list can be found in `./common/IDebugLog.h` | `./common/IDebugLog.h` (included through `./common/IPrefix.h`, which itself is automatically included in all example plugin .cpp files due to the `Forced Include File` project setting in VS) |
    | `PluginHandle` | g_pluginHandle | A unique ID for your plugin obtained from the xNVSE interface. The plugin handle cannot be retrieved until the load function, so it's initialized as an invalid handle for now. | `./nvse/PluginAPI.h` |
    | `NVSEMessagingInterface*` | g_messagingInterface | A pointer the messaging interface class, used to create messages and manage message handlers for communication between plugins. | `./nvse/PluginAPI.h` |
    | `NVSEInterface*` | g_nvseInterface | The NVSE interface  is used for a variety of purposes, such as getting the current game/NVSE version, getting pointers to interfaces, and registering script commands. | `./nvse/PluginAPI.h` |
    | `NVSECommandTableInterface*` | g_cmdTableInterface | The command table interface is used for iterating through and searching for commands and obtaining data on them. Unused in the example plugin (TODO: double check cause I'm not actually sure). | `./nvse/PluginAPI.h` |
    | `NVSEScriptInterface*` | g_script | The script interface can be used to create obscript scripts/expressions dynamically, and can be used to call or get information on [UDF](https://geckwiki.com/index.php/User_Defined_Function)s. | `./nvse/PluginAPI.h` |
    | `NVSEStringVarInterface*` | g_stringInterface | The string interface is used to manipulate [String Variables](https://geckwiki.com/index.php?title=String_Variable) that NVSE introduced to Obscript. | `./nvse/PluginAPI.h` |
    | `NVSEArrayVarInterface*` | g_arrayInterface | The array interface is used to manipulate [Array Variables](https://geckwiki.com/index.php?title=Array_Variable) that NVSE introduced to Obscript. | `./nvse/PluginAPI.h` |
    | `NVSEDataInterface*` | g_dataInterface | The data interface gives you access to some NVSE singletons and functions at runtime, such as `ArrayVarMap` and `StringVarMap`, which manage all array and string variables respectively. | `./nvse/PluginAPI.h` |
    | `NVSESerializationInterface*` | g_serializationInterface | The serialization interface allows you to read/write data from NVSE cosaves, which are files made alongside normal game saves, allowing you to store persistent data related to your plugin. | `./nvse/PluginAPI.h` |
    | `NVSEConsoleInterface*` | g_consoleInterface | The console interface allows you to run console commands through your plugin. | `./nvse/PluginAPI.h` |
    | `NVSEEventManagerInterface*` | g_eventInterface | The event interface is used for managing [script events](https://geckwiki.com/index.php?title=NVSE_Event_Handling), calling Obscript UDFs and/or C++ code from a plugin when they're dispatched. | `./nvse/PluginAPI.h` |
    | `bool (*ExtractArgsEx)(COMMAND_ARGS_EX, ...)` | ExtractArgsEx | A pointer to a function that is retrieved at runtime. This is usually called in script command functions to obtain arguments for the command from the script calling it. | None. The function itself is declared in `nvse/GameAPI.h`. |

    <br>

    ### Plugin Query Function
    `NVSEPlugin_Query` is a function called right after the plugin .dll is loaded, used for version checks and giving xNVSE data on the plugin (by filling out a `PluginInfo` struct). This function is exported when building the .dll so that xNVSE can find and call it at runtime just by using its name. The query function should usually only be used for version checks and filling out `PluginInfo`, as most of the data related to the game and other plugins hasn't been loaded in yet.

    ```cpp
    bool NVSEPlugin_Query(const NVSEInterface* nvse, PluginInfo* info)
    ```
    `NVSEPlugin_Query` must have this definition (returns a `bool`, takes pointers to `NVSEInterface` and `PluginInfo` data structures. xNVSE will call this function and supply the arguments). If the function returns `true`, xNVSE will call `NVSEPlugin_Load` later on, after all other NVSE functions have been queried. The `NVSEInterface` pointer should not be stored yet, and should only be used to obtain version numbers and similar data (`isNoGore`, `isEditor`).

    ```cpp
    _MESSAGE("query");
    ```
    An example of one of the logging functions being used, which will result in the word "query" being logged to `nvse_plugin_example.log` in the game's root folder. (TODO: Might make a section on logging stuff, so I'll have to update this to forward people to it if I do)

    ```cpp
	info->infoVersion = PluginInfo::kInfoVersion;
	info->name = "MyFirstPlugin";
	info->version = 2;
    ```
    Here, xNVSE sends a pointer to a structure that has to be filled with data on the plugin. `name` and `version` can be replaced with any desired value, but `infoVersion` should be left alone.

    ```cpp
	if (nvse->nvseVersion < PACKED_NVSE_VERSION)
	{
		_ERROR("NVSE version too old (got %08X expected at least %08X)", nvse->nvseVersion, PACKED_NVSE_VERSION);
		return false;
	}
    ```
    This version check makes sure that the current version of xNVSE being used meets or exceeds the minimum version required for the plugin. If it does not, it will return `false`, which will stop xNVSE from running the load function, and will unload your plugin (TODO: check if it does cause I'm not actually sure). To change the minimum version required, `NVSE_VERSION_INTEGER`, `NVSE_VERSION_INTEGER_MINOR`, `NVSE_VERSION_INTEGER_BETA`, and `NVSE_VERSION_VERSTRING` in `nvse/nvse_version.h` (included automatically in the example plugin through `common/IPrefix.h`). It's recommended to leave these values alone and inform the user that they have to have the latest xNVSE version as of when your plugin is released.

    ```cpp
    if (!nvse->isEditor)
	{
		if (nvse->runtimeVersion < RUNTIME_VERSION_1_4_0_525)
		{
			_ERROR("incorrect runtime version (got %08X need at least %08X)", nvse->runtimeVersion, RUNTIME_VERSION_1_4_0_525);
			return false;
		}

		if (nvse->isNogore)
		{
			_ERROR("NoGore is not supported");
			return false;
		}
	}
    ```
    This next branch is taken when the game is launched with xNVSE. First, it checks to make sure that the game's executable version matches the latest version supported by xNVSE. The next check looks at if the NoGore German version of FNV is being used. xNVSE itself will load if a NoGore copy is used (TODO: check this as well), but it's easier if your plugin fails the check if NoGore is detected. This is because there are some changes in the .exe that might not make things work as intended, so unless you're willing to test if the NoGore version works with your plugin and make possible adjustments if it doesn't, it's easier to leave it unsupported.

    ```cpp
	else
	{
		if (nvse->editorVersion < CS_VERSION_1_4_0_518)
		{
			_ERROR("incorrect editor version (got %08X need at least %08X)", nvse->editorVersion, CS_VERSION_1_4_0_518);
			return false;
		}
	}
    ```
    This alternate branch is taken when the GECK is launched with xNVSE. It's similar to the runtime version check above, but checks the GECK executable and uses a different version value.

    ```cpp
    return true;
    ```
    If the query function reaches this point and returns `true`, the query is considered successful and `NVSEPlugin_Load` will be called later on.

    <br>

    ### Plugin Load Function
    If the plugin query is successful, xNVSE will call `xNVSEPlugin_Load` after all other plugins have been queried. The data you have access to here is still somewhat limited as the game's data hasn't fully loaded in yet, but registering script commands can be done at this point.

    ```cpp
    bool NVSEPlugin_Load(NVSEInterface* nvse)
    ```
    Like the `NVSEPlugin_Query`, `NVSEPlugin_Load` must have this definition (returns a `bool`, takes pointers to the `NVSEInterface` structure). This function is also exported when building the .dll. If `false` is returned, the load will be considered a failure and your plugin will be unloaded. This does not revert any changes to the game's data that may have been made earlier in the load function, so make sure you return `false` before registering script commands (TODO: maybe with script commands it might unregister them or do a bit of cleanup idk. Check just in case) or making changes to the game's code.

    ```cpp
    g_pluginHandle = nvse->GetPluginHandle();
    ```
    The query and load functions are the only time that a valid handle for your plugin can be returned from `GetPluginHandle`, so call this function now and save the value in case it's needed later. The value may not be valid if called twice or from outside of these functions.

    ```cpp
    g_nvseInterface = nvse;
    ```
    The pointer to the NVSE interface can and should now be saved for later use.

    ```cpp
    g_messagingInterface = static_cast<NVSEMessagingInterface*>(nvse->QueryInterface(kInterface_Messaging));
	g_messagingInterface->RegisterListener(g_pluginHandle, "NVSE", MessageHandler);
    ```
    The first line of this code retrieves a pointer to the `NVSEMessagingInterface` structure. It does this by calling the `QueryInterface` function from `NVSEInterface`, which takes an enum prefixed with `kInterface` (found in `nvse/PluginAPI.h`) and returns the corresponding interface. `QueryInterface` returns a `void*` to the interface, so it must be cast to the appropriate interface pointer. The second line registers the function `MessageHandler`, which was defined earlier, and sets it to listen to messages that xNVSE itself sends out (see the "Message Handling" section below for more information).

    ```cpp
	if (!nvse->isEditor)
	{
    #if RUNTIME
		g_script = static_cast<NVSEScriptInterface*>(nvse->QueryInterface(kInterface_Script));
		g_stringInterface = static_cast<NVSEStringVarInterface*>(nvse->QueryInterface(kInterface_StringVar));
		g_arrayInterface = static_cast<NVSEArrayVarInterface*>(nvse->QueryInterface(kInterface_ArrayVar));
		g_dataInterface = static_cast<NVSEDataInterface*>(nvse->QueryInterface(kInterface_Data));
		g_eventInterface = static_cast<NVSEEventManagerInterface*>(nvse->QueryInterface(kInterface_EventManager));
		g_serializationInterface = static_cast<NVSESerializationInterface*>(nvse->QueryInterface(kInterface_Serialization));
		g_consoleInterface = static_cast<NVSEConsoleInterface*>(nvse->QueryInterface(kInterface_Console));
		ExtractArgsEx = g_script->ExtractArgsEx;
    #endif
	}
    ```
    Initializes all runtime specific global variables defined earlier by querying their interfaces, as seen with the messaging interface.

    ```cpp
	UInt32 const examplePluginOpcodeBase = 0x2000;
	nvse->SetOpcodeBase(examplePluginOpcodeBase);

	RegisterScriptCommand(ExamplePlugin_PluginTest);
	REG_CMD(ExamplePlugin_CrashScript);
	REG_CMD(ExamplePlugin_IsNPCFemale);
	REG_CMD(ExamplePlugin_FunctionWithAnAlias);
	REG_TYPED_CMD(ExamplePlugin_ReturnForm, Form);
	REG_TYPED_CMD(ExamplePlugin_ReturnString, String);
	REG_TYPED_CMD(ExamplePlugin_ReturnArray, Array);
    ```
    Sets a base for the opcode range that your script commands will be registered under and then registers said commands. See the "Registering Commands" section below for more detail.

    <br>

    ### Message Handling
    xNVSE features a system that allows it to notify plugins whenever certain game events happen, like if a user is saving or quitting the game.
    
    ```cpp
    void MessageHandler(NVSEMessagingInterface::Message* msg)
    ```
    In order for your plugin to receive messages, a message handler is needed. A message handler function returns nothing and takes a pointer to an xNVSE message. xNVSE will call this function with a pointer to a `Message` structure containing information about an event whenever certain game events occur.

    ```cpp
	switch (msg->type)
	{
	case NVSEMessagingInterface::kMessage_PostLoad: break;
	case NVSEMessagingInterface::kMessage_ExitGame: break;
	case NVSEMessagingInterface::kMessage_ExitToMainMenu: break;
	case NVSEMessagingInterface::kMessage_LoadGame: break;
	case NVSEMessagingInterface::kMessage_SaveGame: break;
    #if EDITOR
	case NVSEMessagingInterface::kMessage_ScriptEditorPrecompile: break;
    #endif
	case NVSEMessagingInterface::kMessage_PreLoadGame: break;
	case NVSEMessagingInterface::kMessage_ExitGame_Console: break;
	case NVSEMessagingInterface::kMessage_PostLoadGame: break;
	case NVSEMessagingInterface::kMessage_PostPostLoad: break;
	case NVSEMessagingInterface::kMessage_RuntimeScriptError: break;
	case NVSEMessagingInterface::kMessage_DeleteGame: break;
	case NVSEMessagingInterface::kMessage_RenameGame: break;
	case NVSEMessagingInterface::kMessage_RenameNewGame: break;
	case NVSEMessagingInterface::kMessage_NewGame: break;
	case NVSEMessagingInterface::kMessage_DeleteGameName: break;
	case NVSEMessagingInterface::kMessage_RenameGameName: break;
	case NVSEMessagingInterface::kMessage_RenameNewGameName: break;
	case NVSEMessagingInterface::kMessage_DeferredInit: break;
	case NVSEMessagingInterface::kMessage_ClearScriptDataCache: break;
	case NVSEMessagingInterface::kMessage_MainGameLoop: break;
	case NVSEMessagingInterface::kMessage_ScriptCompile: break;
	case NVSEMessagingInterface::kMessage_EventListDestroyed: break;
	case NVSEMessagingInterface::kMessage_PostQueryPlugins: break;
	default: break;
	}
    ```
    The `Message` struct has a `type` field, which changes based on what kind of message that xNVSE sends out. A switch statement is used in the example plugin to handle all of the different types of messages that xNVSE can send. Obviously, not all cases must be handled and unused ones can be removed from the switch statement. The different values of `type` are enums starting with `kMessage` that can be found under the `NVSEMessagingInterface` class.

    <br>

<br>