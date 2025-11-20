<div align="right">
  <p>
    <strong>English</strong> | <a href="https://github.com/POSIdev-community/AI.DslEditor.Plugin.VSCode/blob/main/README.ru.md">Русский</a>
  </p>
</div>

## Overview

JSA DSL is a VSCode plugin that helps to create custom [DSL rules](https://github.com/POSIdev-community/AI.DslEditor.Plugin.VSCode/blob/main/media/DSL_syntax.md). It provides advanced highlighting of JSA DSL syntax, smart autocomplete, and comprehensive code diagnostics. The plugin also supports quick bug fixes, code navigation, and automatic formatting when editing `.jsa.dsl` files.

## Key features

**Syntax highlighting**

Support of colored highlighting for all JSA DSL constructs.

**Smart autocomplete**

Real-time autocompletion of language constructs that considers the context and syntax rules.

**Advanced code diagnostics**

The plugin provides a comprehensive three-level check that includes:
* Syntax analysis:
   * Detection of errors in the code structure (unmatched brackets, missing semicolons, unfinished blocks)
   * Name validation for namespaces, classes, and functions
   * Validation of arguments of the `Detect` and `filter` functions
* Semantic check:
   * Detection of duplicate declarations of classes and functions
   * Checking for unknown PVO Grammar IDs and vulnerability types
   * Analysis of data type matches and usage of allowed references
   * Detection of multiple `return` operators
   * Checking for duplicate names of function parameters and base classes
   * Disabling of nested class and function declarations
* Code quality analysis:
   * Detection of unnecessary `pass` operators
   * Detection of unused parameters in functions

**Context tooltips**

Hovering over a code element shows information about the syntax and semantics.

**Quick Fixes**

Deletion of unnecessary `pass` operators and automatic renaming or deletion of unused parameters

**Code navigation**

Support of the following commands to speed up code navigation and refactoring:
* **Go to Definition**: go to the source code of functions and classes
* **Find References**: find all references to the selected element in the project

**Real-time formatting**

Automatic stylistic unification of code as you enter it (active by default for `.jsa.dsl` files)

## Installation and use

When you install the plugin, the language server is downloaded and started automatically. Server files are saved to the following locations:
* In Windows: `%LOCALAPPDATA%\jsa-dsl\language-server\{server-version}`
* In Linux and macOS: `~/.config/jsa-dsl/language-server/{server-version}`

![Automatic download of the language server](https://github.com/POSIdev-community/AI.DslEditor.Plugin.VSCode/raw/main/media/download_lsp.gif)

To use the plugin, just open the `.jsa.dsl` file.

## Installing the language server manually

The plugin requires the JSA DSL language server for correct operation. The server can be installed automatically or manually (by downloading it from the links provided below).

To install the language server manually:

1. Download the language server archive using one of the links:

   * For Windows: [download](https://update.ptsecurity.com/api/v6/products/JSA.DSL.LSP-Server.Windows/0.1.0.62/download/jsa.dsl.lsp.server-0.1.0.62.zip)

   * For Linux: [download](https://update.ptsecurity.com/api/v6/products/JSA.DSL.LSP-Server.Linux/0.1.0.62/download/jsa.dsl.lsp.server-0.1.0.62.tar.gz)

   * For macOS (Intel): [download](https://update.ptsecurity.com/api/v6/products/JSA.DSL.LSP-Server.MacOS/0.1.0.62/download/jsa.dsl.lsp.server-0.1.0.62.tar.gz)

   * For macOS (ARM): [download](https://update.ptsecurity.com/api/v6/products/JSA.DSL.LSP-Server.MacOS_arm64/0.1.0.62/download/jsa.dsl.lsp.server-0.1.0.62.tar.gz)

2. Unpack the archive to one of the following locations:

   * In Windows: `%LOCALAPPDATA%\jsa-dsl\language-server\0.1.0.62\`

   * In Linux: `~/.config/jsa-dsl/language-server/0.1.0.62/`

   * In macOS: `~/.config/jsa-dsl/language-server/0.1.0.62/`

   > **Note.** You can also extract the server to any other directory and specify the path in the plugin settings using the `jsa.dsl.languageServer.customPath` parameter (see the [Plugin settings](#plugin-settings) section).

3. In macOS, run the commands provided below to delete the `com.apple.quarantine` attribute, sign the binary file, and set execution permissions:

   ```bash
   xattr -cr <path_to_server_folder>/JSA.DSL.LSP
   codesign --sign - --force --deep <path_to_server_folder>/JSA.DSL.LSP
   ```

4. In Linux/macOS, set execute permissions for the executable file:

   ```bash
   chmod +x <path_to_server_folder>/JSA.DSL.LSP
   ```

5. Restart the plugin by running the `JSA DSL: Restart plugin` command.

After restart, the plugin should detect the installed language server and start using it. If the plugin does not detect the server, make sure that:
* The files were extracted to the right directory.
* Access permissions are set correctly (Linux/macOS).
* The logs in **View** → **Output** → **JSA DSL** don't contain any errors.

## Supported file extensions

The plugin is automatically associated with files with the following extensions:
* `.dsl`, the generic extension for DSL files (** enabled by default**)
* `.jsa.dsl`, the main recommended extension for JSA DSL scripts

This allows you to work with JSA DSL scripts regardless of the file extension used.

**Possible conflicts with other plugins**

The `.dsl` extension is also used by other tools and plugins (such as Xtext DSL, Jenkins Job DSL, or Gradle DSL). If you have other plugins for `.dsl` files installed and experience conflicts, you can disable automatic association.

To disable automatic association:

1. Open the [plugin settings](#plugin-settings).

2. Set the `jsa.dsl.associateAllDslFiles` option to `false`.

3. Restart the plugin by running the `JSA DSL: Restart plugin` command.

   The plugin will then work only with `.jsa.dsl` files.

## Plugin settings and commands

### Plugin settings

You can go to the plugin settings by selecting **File** → **Preferences** → **Settings** in the main menu or by pressing <kbd>Ctrl</kbd>+<kbd>,</kbd>.

The plugin configuration page contains the following settings:
* `jsa.dsl.languageServer.customPath`, a custom path to the language server. By default, there's no custom path and the standard path is used (see the [Installation and use](#installation-and-use) section).

* `jsa.dsl.associateAllDslFiles` enables the processing of all `.dsl` files along with `.jsa.dsl` files. The default value is`true`. If changed, plugin restart is required.
   >***Note.** If you encounter conflicts with other plugins that process `.dsl` files, disable this option.*

* `jsa.dsl.diagnostics.enableUnusedParameter` enables warnings for unused function parameters. The default value is `false`.

* `jsa.dsl.diagnostics.enableUnnecessaryPassWarning` enables warnings for unnecessary `pass` statements. The default value is`true`.

* `jsa.dsl.diagnostics.enableTypeCheckWarning` enables warnings for return type mismatches. The default value is`true`.

## Plugin commands

To work with the plugin, you can enter the following commands into the command palette:
* `JSA DSL: Install language server`. Installs the language server manually. Available if the server is not installed.
* `JSA DSL: Restart plugin`. Restarts the plugin and language server. May be needed when updating settings or fixing issues.

## Support and feedback

If you have any questions about using the plugin, detect an issue, or want to suggest an improvement, leave a message in the project repository.
