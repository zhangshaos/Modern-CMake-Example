# Modern-CMake-Example
Config a shared library in Windows (DLL) and import it.

1. `pkg` is a shared library. Build it and install it.
2. `main` is an exe. Build it, install it (just copy dependent DLLs) and run it.

# Usage
The `pkg` is dependent on fmt. Build them like:  

```cmake
cmake ... -DCMAKE_TOOLCHAIN_FILE=.../vcpkg.cmake -DCMAKE_INSTALL_PREFIX=.../
```
