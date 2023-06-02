# Environments

## Introduction

'Environments' is a developer/build tool designed to expose a preset-based management of, and control over, environment configuration variables consumed by runtime applications, on a per-project/repo scope. Intended usage resembles a middle-ground between the CMakePresets API and NodeJS '.env' files.

Furthermore, these environment variable presets can easily be built into compiled binary files using simple and highly familiar command-line instructions, which can then be linked into your executable applications for controlling runtime behaviours and logic. Supported binary output formats include system-native executables, static and dynamic C/C++ libraries, and NodeJS native addon modules. Environment variables can also be pulled from an existing runtime environment and recorded into our customary JSON-formatted files, allowing further inspection, control, and experimentation.

This tool aims to provide a level of runtime safety and security via binary encryption. Web, native, and embedded developers will be familiar with the usage of '.env' files to provide different configurations to your runtimes, based on particular choices (i.e., a 'debug'-mode environment might connect your codebase to a local server for testing, whereas the corresponding 'release'-mode connects to the online production server).

While these '.env' files are emminently useful and powerful development tools, they only provide a simple allocation of 'key = value'; there is no concept of type inference - is a version number parsed as a string of chars, or an array of integers? There is also no opportunity to provide descriptive information, or contain multiple possible result variables, or perhaps types, inside the definition.

To this end, 'Environments' is an experimental attempt at using Javascript's object notation (JSON) to increase the available utility of environment variables during development, and provide more interactive ways of sourcing, tracking, and understanding the environment that are building for, or perhaps even deploying to.

The bulk of the project is predominently the 'env.schema.json' file which you can call from the '$schema' key in a new .JSON file, like so;

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

You might also use tools such as Node-Gyp, dotconfig, CMakeRC, and the Node Addon API to greatly extend the fucntionality of your new-object-based environment (please see the development branch for current status of all of that...)

Some small examples; first, defining a "base" preset with variables that work across most/all platforms and toolsets, then 'inheriting' this preset's variables to further define distinct presets for "win32" and "unix" - style platforms (a rough example but if you've read this far, then I'm sure this will make sense):
```
// env.json
{
  "$schema": "https://raw.githubusercontent.com/cmodules/environments/main/env.schema.json#",
  "version": 2,
  "environmentPresets": [
    {
      "name": "base",
      "flags": {
        "ALL": "--all",
        "HELP": "--help",
        "IGNORE": "--ignore",
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
        "PROMPT": "$binaries{CMD}",
        "RM": "Remove-Item",
        "CP": "Copy-Item"
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
      "binaries": {
        // Common commands
        "CP": "cp",
        "LN": "ln",
        "LS": "ls",
        "RM": "rm"
      },
      "variables": {}
    }
  ]
}
```

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
        "CXX": "AppleClang++"
      },
      "variables": {}
    }
  ]
}
```

Once your environment is fully defined, you should have some highly portable variable "recipes" for easy script and shell command syntaxes. For example, to run your Object file linker to create a dynamically-linked library ('.dll' file), you could define the following command from a script or command line;

```
C:/my/project/> ${LD} ${SOURCE_CODE}.cpp ${LDLIBS} ${LDFLAGS} ${OUTPUT_FLAG} ${LIB_DIR}
```

These variables should resolve against the in-use environment preset, meaning that you might switch to a different environment preset but call the exact same above command, resolving to an executable command appropriate to the chosen preset environment. The variable '${LD}' - here, pointing at a library linker executable - might resolve to a different program, when switching from an 'x64 64bit arch' preset to an 'x86 32bit arch' environment preset, or perhaps you might have the resulting file placed inside a differing '${LIB_DIR}' depending if building for debug or release modes.

This API is under development, but it's likely that users of NodeJS and tools like Make and CMake will understand the mechanisms enough to experiment with the existing codebase - and perhaps offer some useage suggestions etc to help reach maturity of functionality even further :)

Thanks for reading!
