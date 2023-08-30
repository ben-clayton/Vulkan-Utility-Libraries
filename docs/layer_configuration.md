<!--
Copyright 2023 The Khronos Group Inc.
Copyright 2023 Valve Corporation
Copyright 2023 LunarG, Inc.

SPDX-License-Identifier: Apache-2.0
-->

[![Khronos Vulkan][1]][2]

[1]: https://vulkan.lunarg.com/img/Vulkan_100px_Dec16.png "https://www.khronos.org/vulkan/"
[2]: https://www.khronos.org/vulkan/

# Layers Configuration

[![Creative Commons][3]][4]

[3]: https://i.creativecommons.org/l/by-nd/4.0/88x31.png "Creative Commons License"
[4]: https://creativecommons.org/licenses/by-nd/4.0/

Vulkan supports intercepting or hooking API entry points via a layer framework. A layer can intercept all or any subset of Vulkan API entry points. Multiple layers can be chained together to cascade their functionality in the appearance of a single, larger layer.

Vulkan layers give Vulkan application developers the ability to add additional functionality to applications without modifying the application itself, e.g., validating API usages, dumping API entry points or generating screenshots of specified frames.

Vulkan layers can be configured using three different methods to match specific Vulkan developers workflows:
- Using the Vulkan API: `VkCreateInstance()` and `VK_EXT_layer_settings`.
- Using a GUI interface called *[Vulkan Configurator](https://vulkan.lunarg.com/doc/sdk/latest/windows/vkconfig.html)*.
- Using environment variables.

These three methods are implemented by the **Vulkan Layer Settings** library part of the *[Vulkan-Utility-Libraries](https://github.com/KhronosGroup/Vulkan-Utility-Libraries)* repository.
Any layer project including this library will provide these three methods to control layer settings bringing consistency and easy of use of layers to the Vulkan community.

Configuring layers means multiple tasks: Enabling layers; Ordering layers; Configuring the layers capabilities. These three aspects are described with each method to configure layers.

## Configuring Vulkan Layers using *Vulkan Configurator*

Developers can configure layers through a graphical user interface. *Vulkan Configurator* allows full user control of Vulkan layers, including enabling or disabling specific layers, controlling layer order, changing layer settings, etc.
*Vulkan Configurator* configures the layers by applying a global system configuration of the Vulkan loader and creating a vk_layer_settings.txt file that will be find by any layer.

*Vulkan Configurator* can be used using command line to setup the system environment. Use the commannd `vkconfig --help` for more information in this case.

We recommand the using *Vulkan Configurator* GUI approach for Vulkan application developers, it's the most effectively approach to switch between multiple layers configurations and quickly iterate during the development process.
Additionally, *Vulkan Configurator* present to the Vulkan application developers the layers found on the system and the settings of each settings, allowing Vulkan application developers to discover new fonctionnalities from the GUI without having to dig into each layer documentation.

### Layer Settings File

To control the layer settings, *Vulkan Configurator* is generating the `vk_layer_settings.txt` file which set values for each setting of each layer the Vulkan developer wants to configure.

By default, the **Vulkan Layer Settings** library requires the settings file to be named `vk_layer_settings.txt` and it will search it in the working directory of the targeted application. Hence, if a file is found in the working directory of the targeted application, the **Vulkan Layer Settings** library will by pass the layer settings created by *Vulkan Configurator*.
If `VK_LAYER_SETTINGS_PATH` is set and is a directory, then the settings file must be a file called `vk_layer_settings.txt` in the directory given by `VK_LAYER_SETTINGS_PATH`.
If `VK_LAYER_SETTINGS_PATH` is set and is not a directory, then it must point to a file (with any name) which is the layer settings file.

The settings file consists of comment lines and settings lines.  Comment lines begin with the `#` character.  Settings lines have the following format:

   `<`*`LayerName`*`>.<`*`setting_name`*`> = <`*`setting_value`*`>`

The settings and values available for each layer are listed in the layer manifest and the documentation of each layer using **Vulkan Layer Settings** library to implement layer settings.

## Configuring Vulkan Layers using `VkCreateInstance()`
Applications may programmatically activate layers via the `vkCreateInstance()` entry point. This
is done by setting `enabledLayerCount` and `ppEnabledLayerNames` in the `VkInstanceCreateInfo`
structure.

### Code example

```cpp
    const char* profile_file_data = JSON_TEST_FILES_PATH "VP_LUNARG_desktop_baseline_2023.json";
    const char* profile_name_data = "VP_LUNARG_desktop_baseline_2023";
    const VkBool32 emulate_portability_data = VK_TRUE;
    const char* simulate_capabilities[] = {"SIMULATE_FEATURES_BIT", "SIMULATE_PROPERTIES_BIT", "SIMULATE_EXTENSIONS_BIT"};

    const VkLayerSettingEXT settings[] = {
        {kLayerName, kLayerSettingsProfileFile, VK_LAYER_SETTING_TYPE_STRING_EXT, 1, &profile_file_data},
        {kLayerName, kLayerSettingsProfileName, VK_LAYER_SETTING_TYPE_STRING_EXT, 1, &profile_name_data},
        {kLayerName, kLayerSettingsEmulatePortability, VK_LAYER_SETTING_TYPE_BOOL32_EXT, 1, &emulate_portability_data},
        {kLayerName, kLayerSettingsSimulateCapabilities, VK_LAYER_SETTING_TYPE_STRING_EXT, static_cast<uint32_t>(std::size(simulate_capabilities)), simulate_capabilities}};

    const VkLayerSettingsCreateInfoEXT layer_settings_create_info = {
        VK_STRUCTURE_TYPE_LAYER_SETTINGS_CREATE_INFO_EXT, nullptr,
        static_cast<uint32_t>(std::size(settings)), settings};

    const VkApplicationInfo app_info = {
        VK_STRUCTURE_TYPE_APPLICATION_INFO, nullptr,
        "profiles_tests", VK_MAKE_API_VERSION(0, 1, 0, 0),
        "profiles_tests", VK_MAKE_API_VERSION(0, 1, 0, 0),
        VK_API_VERSION_1_3
    };

    const char* layers[] = {
        "VK_LAYER_KHRONOS_validation", "VK_LAYER_KHRONOS_profiles"
    };

    const VkInstanceCreateInfo inst_create_info = {
        VK_STRUCTURE_TYPE_INSTANCE_CREATE_INFO, &layer_settings_create_info, 0,
        &app_info,
        static_cast<uint32_t>(std::size(layers)), layers,
        0, nullptr
    };

    VkInstance instance = VK_NULL_HANDLE;
    VkResult result = vkCreateInstance(&inst_create_info, nullptr, &instance);
```

## Configuring Vulkan Layers using Environment Variables

### Enabling and ordering Vulkan Layers

In order to enable a Vulkan layer from the command-line, you must first make sure:

  1. The layer's Manifest JSON file is found by the Vulkan Desktop Loader because it is in:
      * One of the standard operating system install paths
      * It was added using one of the layer path environment variables (`VK_LAYER_PATH` or `VK_ADD_LAYER_PATH`).
      * See the `Layer Discovery` section of the Vulkan Loader's [Layer Interface doc](https://github.com/KhronosGroup/Vulkan-Loader/blob/main/docs/LoaderLayerInterface.md).
  2. The layer's library file is able to be loaded by the Vulkan Desktop Loader because it is in:
      * A standard library path for the operating system
      * The library path has been updated using an operating system-specific mechanism such as:
          * Linux: adding the path to the layer's library .so with `LD_LIBRARY_PATH`
          * MacOS: adding the path to the layer's library .dylib with `DYLD_LIBRARY_PATH` 
  3. The layer's library file is compiled for the same target and bitdepth (32 vs 64) as the application

### Activating Specific SDK Layers

To activate layers located in a particular SDK installation, or layers built locally from source, specify the layer JSON
manifest file directory using either `VK_LAYER_PATH` or `VK_ADD_LAYER_PATH`.
The difference between `VK_LAYER_PATH` and `VK_ADD_LAYER_PATH` is that `VK_LAYER_PATH` overrides the system layer paths
so that no system layers are loaded by default unless their path is added to the environment variable.
`VK_ADD_LAYER_PATH` on the otherhand, causes the loader to search the additional layer paths listed in the
environment variable first, and then the standard system paths will be searched.

#### Example Usage On Windows:

For example, if a Vulkan SDK is installed in `C:\VulkanSDK\1.2.198.0`, execute the following in a Command Window:

```
C:\> set VK_LAYER_PATH=C:\VulkanSDK\1.2.198.0\Bin
```

#### Example Usage on Linux

For Linux, if Vulkan SDK 1.2.198.0 was locally installed in `/sdk` and `VULKAN_SDK=/sdk/1.2.198.0/x86_64`:

```
$ export VK_LAYER_PATH=$VULKAN_SDK/lib/vulkan/layers
$ export LD_LIBRARY_PATH=$VULKAN_SDK/lib:$VULKAN_SDK/lib/vulkan/layers
```

#### Example Usage on MacOS

For macOS, if Vulkan SDK 1.2.198.0 was locally installed in `/sdk` and `VULKAN_SDK=/sdk/1.2.198/macOS`:

```
$ export VK_LAYER_PATH=$VULKAN_SDK/share/vulkan/explicit_layers.d
$ export DYLD_LIBRARY_PATH=$VULKAN_SDK/lib
```

### Enabling Layers

Originally, the Vulkan Desktop Loader provided `VK_INSTANCE_LAYERS` to enable layers from the command-line.
However, starting with the Vulkan Loader built against the 1.3.234 Vulkan headers, the `VK_LOADER_LAYERS_ENABLE` environment
variable was added to allow for more easily enabling Vulkan layers.
The newer Loaders will continue to accept the original `VK_INSTANCE_LAYERS` environment variable for some time, but it is
considered deprecated.

#### Vulkan 1.3.234 Loader and Newer

The easiest way to enable a layer with a more recent drop of the Vulkan Loader is using the `VK_LOADER_LAYERS_ENABLE
environment variable.
This environment variable accepts a case-insensitive, comma-delimited list of globs which can be used to define
the layers to load.

For example, previously if you wanted to enable the Api Dump layer and the Validation layer, you would have to set
`VK_INSTANCE_LAYERS` equal to the full name of each layer:

```
VK_INSTANCE_LAYERS=VK_LAYER_LUNARG_api_dump;VK_LAYER_KHRONOS_validation
```

Now, with `VK_LOADER_LAYERS_ENABLE`, you simply can use stars where you don't want to fill in the full name:

```
VK_LOADER_LAYERS_ENABLE=*api_dump,*validation
```

##### Example Usage On Windows:

```
C:\> set VK_LOADER_LAYERS_ENABLE=*api_dump,*validation
```

##### Example Usage On Linux/macOS:

```
$ export VK_LOADER_LAYERS_ENABLE=*api_dump,*validation
```

More info about the new layer filtering environment variables can be found in the `Layer Filtering` section of the
of the [Loader Layer Documentation](https://github.com/KhronosGroup/Vulkan-Loader/blob/main/docs/LoaderLayerInterface.md).

#### Older Loaders

Older Vulkan Desktop loaders will not accept the filtering environment variable, and so must continue using the original
`VK_INSTANCE_LAYERS` environment variable.


##### Example Usage On Windows:

The variable should include a semicolon-separated list of layer names to activate.
Note that order is relevant, with the initial layer being the closest to the application, and the final layer being closest to the driver.

```
C:\> set VK_INSTANCE_LAYERS=VK_LAYER_LUNARG_api_dump;VK_LAYER_KHRONOS_validation
```

In this example, the api_dump layer will be called _before_ the Khronos validation layer.
`VK_INSTANCE_LAYERS` may also be set in the system environment variables.

##### Example Usage On Linux/macOS:

The variable should include a colon-separated list of layer names to activate.
Note that order is relevant, with the initial layer being the closest to the application, and the final layer being closest to the driver.

```
$ export VK_INSTANCE_LAYERS=VK_LAYER_LUNARG_api_dump:VK_LAYER_KHRONOS_validation
```

In this example, the api_dump layer will be called _before_ the Khronos validation layer.

## Layer Settings Environment Variables

Some settings from the settings file can also be set using environment variables. The settings that can be set using environment variables are 
listed in the documentation for each supported layer.  If an environment variable is set, its value takes precedence over the value in the settings file.
