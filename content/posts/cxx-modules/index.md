+++
date = '2025-04-26T15:27:42+07:00'
title = 'cxx-modules.md'
+++

As everyone who clicked on this post should know by now, C++ modules have been a
thing since C++20. Oh, 2020, it's been a while, huh. Back then, C++ modules to
me were nothing but a distant dream. We need to wait for major compilers to
adopt this new thing, CMake to support it, and LSPs like `clangd` to handle them
properly. And looking at how some people are still writing C++ like 'C with
classes', certainly this won't come soon, right?

Well, it's 2025 now and C++ modules is still in its infancy. People still
trashtalked on this feature until this very day. Wait, using C++ modules mean
that you don't need to separate headers and source files any more? Screw this,
I'm diving head first into this thing.

C and C++'s header-source separation is a major pain in the ass. You have to
declare and define a function, and when that function's signature changes, you
will have to fix both the declaration and the definition. C++ modules solve
this, kind of. It replaces `.h` and `.cpp` file by a `.cppm` file, which can
serves as a header file and a source file at the same time. If you want to hide
implementation details (which you will probably need to because of C++ modules'
weirdness), you can treat that `.cppm` file as a header and add some trusty-old
`.cpp` source files to that library.

This blog post will be a _status-update_, kind of, of C++ modules. It is still
in its infancy, so there are quite some hoops needed to jump through.
Personally, I use it for personal projects that have no uses other than to force
me to learn something (e.g. [my Ray Tracing in One Weekend
implementation](https://github.com/btmxh/rt-weekend-vk)) or some tool I only
use (and when it's finished C++ modules will _hopefully_ be stablized so
everyone can run that too).

## CMake

First of all, use the Ninja generator. It's fast, cross-platform, supports
`compile_commands.json`. Use it.

CMake supports C++20 modules but not the fancy C++23 `import std;`. To use that,
you need to add these two lines:

```cmake
set(CMAKE_EXPERIMENTAL_CXX_IMPORT_STD "a9e1cf81-9932-4810-974b-6eccaf14e457")
set(CMAKE_CXX_MODULE_STD 1)
```

**before** the `project()` call.

The UUID thing can be taken from
[here](https://github.com/Kitware/CMake/blob/master/Help/dev/experimental.rst#c-import-std-support).
I would strongly advise you go there instead of using the one above (which may
be outdated).

Also, try to set the minimum version of CMake to something at least 3.31. Or
4.0.

To define a module target (e.g. a library), you need to use the shiny CMake 3.11
`target_sources` to provide source files. Note that file sets are CMake 3.23, and
`CXX_MODULE` file sets are CMake 3.28. Well, this should be relevant if you set
the `cmake_minimum_required` to something recent.

```cmake
add_library(lib)
target_sources(lib
    # C++ module file set, providing the public interface
    PUBLIC FILE_SET CXX_MODULES
    FILES lib.cppm

    # C++ implementation details
    PRIVATE lib-internal.cpp
)
# put some C++23 for good measure
target_compile_features(vkb_wrapper PUBLIC cxx_std_23)
```

**NOTE**:
Make sure that the modules you are using have compatible compile features. For
example, if you have a module using a C++20 standard library and your main
executable using C++23, it might break! (it did for me, at least).

## Compilers

If you are on Windows, use MSVC.

Otherwise, use clang. Chances are, you are also using `clangd`, so just use the
same tool as your compiler to avoid some headache. Trust me, you will have some.

As for the time of writing, GCC 15 just dropped, which people have said to [have
pretty good support for
modules](https://www.reddit.com/r/cpp/comments/1j7vplc/gcc_support_std_module_with_cmake_40_now/).
Though, I will care about it once the Arch reposistory has it.

Here is my `clang-toolchain.cmake` file to force CMake to use clang instead of
gcc. The `stdlib=libc++` flag makes clang use libc++ instead of the system
libstdc++ (provided by GCC).

```cmake
set(CMAKE_C_COMPILER clang CACHE STRING "C compiler" FORCE)
set(CMAKE_CXX_COMPILER clang++ CACHE STRING "C++ compiler" FORCE)
set(CMAKE_CXX_FLAGS "-stdlib=libc++")
```

If you use `clangd`, just use the `compile_commands.json` and don't set any
extra options.

## Packages

[Here](https://arewemodulesyet.org/) is a list of packages with module support
status. So if you use something like `Vulkan-Hpp`, you are in luck[^1].

Otherwise, you will have to do some work. If you are lucky, you can just
`#include` the header file. This works for C headers, as far as I'm aware.

For a `.cpp` file, you do

```cpp
import std;
#include <some/lib/that/doesnt/support/module.h>
```

But for a `.cppm`, it's somewhat more complicated:

```cpp
module;

#include <some/lib/that/doesnt/support/module.h>

export module mod;

// do it here, not above!
import std;

// actual code
```

because you don't want the included code to be part of the module, apparently.

Here are some examples of this in action, taken from my RTIOW code:

```cpp
module;

#define STB_IMAGE_WRITE_IMPLEMENTATION
#include <stb/stb_image_write.h>

export module stbi;

export namespace stbi {
int write_png(const char *filename, int width, int height, int comp,
              const unsigned char *pixels, int stride) {
  return stbi_write_png(filename, width, height, comp, pixels, stride);
}

int write_hdr(const char *filename, int width, int height, int comp,
              float *pixels) {
  return stbi_write_hdr(filename, width, height, comp, pixels);
}
} // namespace stbi
```

```cpp
module;

// this imported std
#include <VkBootstrap.h>

export module vkb;

// so don't use std here, or else clangd will die

// if there's a way to export everything from namespace vkb, please let me know
export namespace vkb {
using vkb::Instance;
using vkb::InstanceBuilder;
using vkb::PhysicalDeviceSelector;
using vkb::PhysicalDevice;
using vkb::DeviceBuilder;
using vkb::Device;
using vkb::QueueType;
} // namespace vkb
```

## `clangd` hackeries

`clangd` still gives a lot of false-positive errors. My `vk_mem_alloc.h` wrapper
file has:

```cpp
module;

#define VMA_IMPLEMENTATION
#include <vk_mem_alloc.h>

export module vma;

import std;
import vulkan_hpp;
```

which compiled fine with CMake/Ninja/clang, but `clangd` spits out a bunch of
redefinition errors, which indicates that it's trying to include some STL header
twice. I fixed this by moving the implementation of `vk_mem_alloc.h` into a
`.cpp` file, but I have no idea what to do if the public API of `vk_mem_alloc.h`
uses the STL.

[^1]: Not that `Vulkan-Hpp` bloated compile time to the point that you're forced to
use modules.
