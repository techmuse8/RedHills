<img alt="banner redhills" src="https://github.com/user-attachments/assets/698372b7-7dc7-47f2-a675-49cf7ea3c0f0" />
<div align="center">
  <img alt="wiiu" height="56" src="https://github.com/user-attachments/assets/c1576bbd-fcc0-4ca9-982f-a6dfe5a8545d">
  <a href="https://go.nsmbu.net/discord">
    <img alt="discord" height="56" src="https://github.com/user-attachments/assets/785798a3-2702-42be-b960-584b1df86075">
  </a>
  <a href="https://zenith.nsmbu.net/wiki/Red_Hills">
    <img alt="docs" height="56" src="https://github.com/user-attachments/assets/109c6469-e10d-4407-940d-554203452499">
  </a>
</div>

## Overview
**Red Hills** is a specialized fork of Clang 19.1.7 designed to achieve 100% ABI compatibility with proprietary Green Hills PowerPC Wii U binaries. We've murdered the C++98 restrictions of old and delivered a modern, open toolchain for the Wii U ecosystem. It is explicitly designed for use with the [Tachyon](https://github.com/Zenith-Team/Tachyon) Wii U code injection toolkit, in which it comes bundled with.

## Goals
- Full [ABI parity](https://zenith.nsmbu.net/wiki/Green_Hills_C%2B%2B_ABI) with GHS such that binaries built with either compiler may call upon each other with no behaviour concerns.
## Non-goals
- It is not a goal to have full codegen parity with GHS, for uses such as matching decompilations.
- It is not a goal to support language features which are impossible to use in conjunction such as C++ exceptions.
- It is not a goal to produce a drop-in replacement for GHS, thus the language frontend and driver invocation interface will not recieve parity.
- It is not a goal to maintain any form of support for general non-GHS targets, please use upstream Clang or alternatives for such use cases.

## Key Changes
A comprehensive guide to the ABI can be found at the documentation [here](https://zenith.nsmbu.net/wiki/Green_Hills_C%2B%2B_ABI).
### Parity
- **Pointer To Member Function** (`ItaniumCXXABI`): Modified structure to mirror GHS emission and calling behaviour.
- **Virtual Table** (`CGVTables`): Patched generation of table data and address-point to allow interface layout parity.
- **Class Layout** (`RecordLayoutBuilder`): Never use tail padding, and place the virtual table pointer at the end of the struct.
- **Multiple Inheritance** (`RecordLayoutBuilder`): Only inherit the virtual table pointer from the first base in the tree.
- **Allocating Constructors** (`CGClass`): Null `this` causes heap-allocation of the object.
- **Unified Destructors** (`ItaniumCXXABI`): Removed base and deleting destructors, the complete destructor handles both using a flag.
- **Stack Returns**: When returning a value in the stack using a pointer parameter `sret`, the `this` parameter takes priority and `sret` is placed after.
### Extensions
- **__builtin_mangle**: Introducing a method of converting an identifier into a string literal of its mangled form at compile-time.
  - Signature: `const char* __builtin_mangle(Decl, OptionalSignatureForOverloads)`
- **PreserveAll**: Added support for this calling convention on PowerPC, allowing safe hooking of functions without clobbering processor registers.
  - Decorate a function with the `[[clang::preserve_all]]` attribute to use this feature.
### Driver Defaults
- **Triple**: `powerpc-eabi`
- **Exceptions**: Disabled (currently incompatible)
- **RTTI**: Disabled (currently incompatible)
- **Thread-safe Statics**: Disabled
- **Aligned Allocation**: Disabled

## Limitations
See the [issue tracker](https://github.com/Zenith-Team/RedHills/issues) to see a list of known parity issues or missing features.

## Usage
Use it like standard Clang, however deviating from the default flags too much may result in breaking the ABI parity or codegen corruption.
- Note that the default behaviour is to compile only without linking, as `ld` is expected to be invoked manually to produce the resulting RPL. See [Tachyon](https://github.com/Zenith-Team/Tachyon) for more details.

## Contributing
Contributions are welcome! We appreciate external help for ensuring parity and support, please utilize the [issue tracker](https://github.com/Zenith-Team/RedHills/issues) to file or find bugs or enhancements to work on.

## Building
1. Clone the repository
2. Install the dependencies:
   - Clang
   - CMake
   - Ninja
3. Run the corresponding setup script for your system (`setup[OS].bat/sh`)
4. Initiate the compile with `ninja -C build clang`
5. The built files will be in `build/bin/`

Consult the [Getting Started with LLVM](https://llvm.org/docs/GettingStarted.html#getting-the-source-code-and-building-llvm) page for additional information on building and running Red Hills.

## Credits
- [Luminyx](https://github.com/Luminyx1) - ABI patches, language/backend extensions
- [jhmaster](https://github.com/jhmaster2000) - Testing and verification
### Special Thanks
- [Oliver Hunt](https://waffles.dog)
- [The LLVM Project](https://llvm.org)

## License
All code in the Red Hills repository has been made available under the [Apache License v2.0 with LLVM Exceptions](https://llvm.org/LICENSE.txt).
