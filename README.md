# Environments

## Introduction

'Environments' is a developer/build tool designed to expose a preset-based management of, and control over, environment configuration variables consumed by runtime applications, on a per-project/repo scope. Intended usage resembles a middle-ground between [NVM (Node version manager)](https://github.com/nvm-sh/nvm), the [CMakePresets.json API](https://cmake.org/cmake/help/latest/manual/cmake-presets.7.html), and [NodeJS '.env' files](https://github.com/motdotla/dotenv).

Web, native, and embedded developers will be familiar with the usage of '.env' files to provide different configurations to your runtimes, based on particular choices (i.e., a 'debug'-mode environment might connect your codebase to a local server for testing, whereas the corresponding 'release'-mode connects to the online production server).

While these '.env' files are emminently useful and powerful development tools, they only provide a simple allocation of 'key = value'; there is no concept of type inference - is a version number parsed as a string of chars, or an array of integers? There is also no opportunity to provide descriptive information, or contain multiple possible result variables, or perhaps types, inside the definition.

To this end, 'Environments' is an experimental attempt at using Javascript's object notation (JSON) to increase the available utility of environment variables during development, and provide more interactive ways of sourcing, tracking, and understanding the environment that are building for, or perhaps even deploying to. 

Furthermore, these environment variable presets can easily be built into compiled binary files using simple and highly familiar command-line instructions, which can then be linked into your executable applications for controlling runtime behaviours and logic. Supported binary output formats include system-native executables, static and dynamic C/C++ libraries, and NodeJS native addon modules. Environment variables can also be pulled from an existing runtime environment and recorded into our customary JSON-formatted files, allowing further inspection, control, and experimentation. This aspect of the tool also aims to provide a level of runtime safety and security via binary encryption into a wide range of deployable formats. 

The bulk of the project is predominently centred on the 'env.schema.json' file which you can call from the '$schema' key in a new .JSON file, like so;

```
// test.env.json
{
    "$schema": "https://raw.githubusercontent.com/cmodules/environments/main/env.schema.json#"
}
```

The remainder of this file will now have annotated entries for all it's API (which is currently under development, but already working pretty well in it's current state enough to share).

Once your environment is defined, you can easily access it as a Js module from an 'index.js' file;

```
#!/usr/bin/env node

import * as environment from "./env.json";

const env = environment;

export { env };
```

*The above file is executable directly from the command line; The NodeJS binary found in the current environment scope (think "PATH") will execute the javascript commands. You could call 'node ./index.js' followed by a 'toString()' function call on the command line to report various definitions to the console...

You might also use tools such as Node-Gyp, dotconfig, CMakeRC, and the Node Addon API to greatly extend the fucntionality of your new-object-based environment (please see the development branch for current status of all of that...)



When developing and building software and technologies, it becomes fairly common that we accrue multiple tools that each cover very similar/the same tasks, and even might have similar/indentical user interfaces. An example of this is in NodeJS development, where pre-existing codebases require particular version numbers of node and it's toolset in order to run correctly, creating 'version-lock' requirement for development tools that differs from project to project. Another example is CMake for C/C++ development, where the possibility of targeting system environments that differ wildly from the development environment is a common and long-standing concern that CMake itself was built to address, as an extension of GNU Make's concept of a 'build environment'. In fact, we all know that environment variables interact with our system and provide us with some level of control; but, since most such variables exist only during a given runtime and are not typically exposed to the developer, it can be difficult to ensure that, for example, that build step we're running which takes 30 minutes to complete, is calling the 64-bit compiler and not the 32-bit, under the hood. There also does not currently appear to be a singular way to manage the variables across differeng tools, platforms, and concerns.

'Environments' takes its' inspirations directly from experience with all of the above, and attempts to bridge a percieved gap between common developer toolings by providing a cache-like 'single source of truth' of environment variables, in turn attempting to enhance the overall flexibility and possibilities of these variables for further downstream processing.

Some small examples; first, defining a "base" preset with variables that work across most/all platforms and toolsets, then 'inheriting' this preset's variables to further define distinct presets for "win32" and "unix" - style platforms (a rough example but if you've read this far, then I'm sure this will make sense):
```
// env.json
{
  "$schema": "https://raw.githubusercontent.com/cmodules/environments/main/env.schema.json#",
  "version": 2,
  "environmentPresets": [
    {
      "name": "base",
      "commands": {
        // Wrap commands with flags (if flags are undefined, they will just be ignored)...
        "CP_COMMAND": "${CP} ${CPFLAGS}",
        "RM_COMMAND": "${RM} ${RMFLAGS}"
      },
      "flags": {
        "ALL": "--all",
        "HELP": "--help",
        "IGNORE": "--ignore",
        "INTERACTIVE": "--interactive",
        "LIST": "--list",
        "OUTPUT": "--output",
        "PRESET": "--preset",
        "PROFILE": "--profile=${USER}",
        "QUIET": "--quiet",
        "RECURSE": "--recurse",
        "RECURSIVE": "--recursive",
        "SYNC": "--sync",
        "VERBOSE": "--verbose",
        "VERSION": "--version"
      },
      "variables": {},
      "utilities": {
        "EOL": "lf",
        "PATH_SEPERATOR": ":"
      },
      "paths": {
        "INSTALL_BINDIR": "/usr/local/bin",
        "INSTALL_LIBDIR": "/usr/local/lib",
        "PATH": [
          "/usr/local/bin",
          "/usr/bin",
          "/bin",
          "/usr/bin/site_perl",
          "/usr/bin/vendor_perl",
          "/usr/bin/core_perl"
        ]
      }
    }
  ]
}
```

A recommended "best practice" approach would be to define most of your required variables in a "base"-level preset, and then inherit this and override *just* the few changes that you need;

```
// win32.env.json
{
  "$schema": "https://raw.githubusercontent.com/cmodules/environments/main/env.schema.json#",
  "version": 2,
  "include": "env.json",
  "environmentPresets": [
    {
      "name": "win32",
      "inherits": ["base"],
      "binaries": {
        "CMD": "cmd",
        "PWSH": "pwsh",
        "MSVC": "msvc"
      },
      "commands": {
        // Common commands
        "RM": "Remove-Item",
        "CP": "Copy-Item",
        "PROMPT": "$binaries{CMD}"
      },
      "definitions": {
        "WIN32": true
      },
      "utilities": {
        "EOL": "crlf",
        "PATH_SEPERATOR": ";"
      },
      "paths": {
        "INSTALL_BINDIR": "C:/'Program Files'/MyApplication/bin",
        "INSTALL_LIBDIR": "C:/'Program Files'/MyApplication/lib",
        "PATH": [
          "C:/Windows/System32",
          "C:/Windows",
          "C:/Windows/System32/Wbem",
          "C:/Windows/System32/WindowsPowerShell/v1.0/",
        ]
      }
    }
  ]
}
```

Because of inheritance, you might only need to alter a small part of a larger definition, such as amending the flags from the previously-constructed "CP_COMMAND" and "RM_COMMAND" vars when in certain Unix environments:

```
// unix.env.json
{
  "$schema": "https://raw.githubusercontent.com/cmodules/environments/main/env.schema.json#",
  "version": 2,
  "include": "env.json",
  "environmentPresets": [
    {
      "name": "unix",
      "inherits": ["base"],
      "binaries":
        "LN": "ln",
        "LS": "ls",
        // Flags
        "CPFLAGS": "${INTERACTIVE}",
        "LSFLAGS": "${LIST} ${ALL}",
        "RMFLAGS": "-f ${RECURSE} ${VERBOSE}"
      },
      "variables": {}
    }
  ]
}
```

You might also have binaries - or even vars - that only exist on a single platform, such as with Apple:

```
// apple.env.json
{
  "$schema": "https://raw.githubusercontent.com/cmodules/environments/main/env.schema.json#",
  "version": 2,
  "include": "unix.env.json",
  "environmentPresets": [
    {
      "name": "apple",
      "inherits": ["unix"],
      "binaries": {
        // Clang compiler by default...
        "CC": "AppleClang++"
        "CXX": "AppleClang++",
        // Apple-only, no need to have defined this one at a universal "base" scope...
        "OSX_DEPLOYMENT_TARGET": 10.13
      },
      "variables": {}
    }
  ]
}
```

Once your environment is fully defined, you should (in theory) have some highly portable variable "recipes" for easy script and shell command syntaxes. For example, to run your Object file linker to create a dynamically-linked library ('.dll' file), you could define the following command from a script or command line;

```
{
  // Make a wrapped command that adapts to it's associated environment variables:
  "CLEAN_LIBRARY": "${RM_COMMAND} ${LIB_DIR}/{SOURCE_CODE}.dll && ${ECHO} 'cleaning...'",
  "BUILD_LIBRARY": "${LD} ${SOURCE_CODE}.cpp ${LDLIBS} ${LDFLAGS} ${OUTPUT_FLAG} ${LIB_DIR} && ${ECHO} 'building...'",
  "REBUILD_LIBRARY": "${CLEAN_LIBRARY} && ${BUILD_LIBRARY}"
}
```

Then simply call your wrapped command (in theory)...

```
C:/my/project/> "${REBUILD_LIBRARY}"
// "cleaning..."
// "building..."
```

A nice utility could be to add 'verbose' flags/args to your 'debug' environment, for further technical readouts of the processes being called. Perhaps even better, you could add a 'VERBOSE' flag to the command at the "base"-preset level, and simply leave it undefined (and thus resolves to an empty char) in environments that verbosity is not required.

```
{
  {
    "environmentPreset": [
      "debug": {
        "VERBOSE": true
      },
      "release": {
        // "VERBOSE": null
      }
    ]
  }
  // Make a wrapped command that contains a '--verbose' flag at a global level:
  "CLEAN_LIBRARY_VERB": "${RM_COMMAND} {VERBOSE} ${LIB_DIR}/{SOURCE_CODE}.dll && ${ECHO} 'cleaning...'",
  "REBUILD_LIBRARY": "${CLEAN_LIBRARY_VERB} && ${BUILD_LIBRARY}"
}
```

These variables should resolve against the in-use environment preset, meaning that you might switch to a different environment preset but call the exact same above command, resolving to an executable command appropriate to the chosen preset environment. The variable '${LD}' - here, pointing at a library linker executable - might resolve to a different program, when switching from an 'x64 64bit arch' preset to an 'x86 32bit arch' environment preset, or perhaps you might have the resulting file placed inside a differing '${LIB_DIR}' depending if operating in a 'debug'- or 'release'-mode environment preset.

This API is under development, but it's likely that users of NodeJS and tools like Make and CMake will understand the mechanisms enough to experiment with the existing codebase - and perhaps offer some useage suggestions etc to help reach maturity of functionality even further :)

Thanks for reading!

https://gcc.gnu.org/onlinedocs/gcc/Environment-Variables.html
https://www.gnu.org/software/make/manual/make.html
https://earthly.dev/blog/makefile-variables/
https://llvm.org/docs/GettingStarted.html#local-llvm-configuration
https://clang.llvm.org/docs/UsersManual.html
https://vector-of-bool.github.io/2017/01/21/cmrc.html
https://blog.conan.io/2022/10/13/Different-flavors-Clang-compiler-Windows.html
