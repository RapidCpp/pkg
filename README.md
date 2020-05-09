# CMake 

## Overview

[CMake](https://cmake.sh) helps you manage C and C++ libraries on Windows, Linux, MacOS, Raspberry Pi, iOS and android, etc. This tool and ecosystem are constantly evolving; your involvement are vital to its success!

Currently this project depends on [vcpkg](https://github.com/microsoft/vcpkg). For short description of available commands, run vcpkg help.

* Github: [https://github.com/microsoft/vcpkg](https://github.com/microsoft/vcpkg)
* Slack: [https://cppalliance.org/slack/](https://cppalliance.org/slack/), the #vcpkg channel
* Docs: [Documentation](docs/index.md)

## Possibilities

* [milestone1](https://github.com/suhao/cmake/issues/1#issue-607267924)


## Quick Start

Prerequisites:
- Windows 10, 8.1, 7, Linux, or MacOS
- Visual Studio 2015 Update 3 or newer (on Windows)
- Git
- gcc >= 7 or equivalent clang (on Linux)
- *Optional:* CMake 3.12.4

To get started:
```
> git clone https://github.com/Microsoft/vcpkg.git
> cd vcpkg

PS> .\bootstrap-vcpkg.bat
Linux:~/$ ./bootstrap-vcpkg.sh
```

Then, to hook up user-wide [integration](docs/users/integration.md), run (note: requires admin on first use)
```
PS> .\vcpkg integrate install
Linux:~/$ ./vcpkg integrate install
```

Install any packages with
```
PS> .\vcpkg install sdl2 curl
Linux:~/$ ./vcpkg install sdl2 curl
```

The best way to use installed libraries with CMake is via the toolchain file `scripts\buildsystems\vcpkg.cmake`. To use this file, you simply need to add it onto your CMake command line as `-DCMAKE_TOOLCHAIN_FILE=[vcpkg root]\scripts\buildsystems\vcpkg.cmake`.

In Visual Studio, you can create a New Project (or open an existing one). All installed libraries are immediately ready to be `#include`'d and used in your project without additional configuration.

For more information, see our [using a package](docs/examples/installing-and-using-packages.md) example for the specifics. If your library is not present in vcpkg catalog, you can open an [issue on the GitHub repo](https://github.com/microsoft/vcpkg/issues) where the dev team and the community can see it and potentially create the port file for this library.

Additional notes on macOS and Linux support can be found in the [official announcement](https://blogs.msdn.microsoft.com/vcblog/2018/04/24/announcing-a-single-c-library-manager-for-linux-macos-and-windows-vcpkg/).

## Tab-Completion / Auto-Completion
`vcpkg` supports auto-completion of commands, package names, options etc in Powershell and bash. To enable tab-completion, use one of the following:
```
PS> .\vcpkg integrate powershell
Linux:~/$ ./vcpkg integrate bash
```
and restart your console.


## Examples
See the [documentation](docs/index.md) for specific walkthroughs, including [installing and using a package](docs/examples/installing-and-using-packages.md), [adding a new package from a zipfile](docs/examples/packaging-zipfiles.md), and [adding a new package from a GitHub repo](docs/examples/packaging-github-repos.md).

Our docs are now also available online at ReadTheDocs: <https://vcpkg.readthedocs.io/>!

See a 4 minute [video demo](https://www.youtube.com/watch?v=y41WFKbQFTw).


## Contributing
Vcpkg is built with your contributions. Here are some ways you can contribute:

* [Submit Issues](https://github.com/suuhao/cmake/issues) in vcpkg or existing packages
* [Submit Fixes and New Packages](https://github.com/suhao/cmake/pulls)

Please refer to our [Contribution guidelines](CONTRIBUTING.md) for more details.