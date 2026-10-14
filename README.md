# opencv_example_cmake

A minimal OpenCV demo built with CMake. It opens the image given on the command
line and shows it in a window until a key is pressed.

## Requirements

This needs the dev shell from https://github.com/ormandi/nix_shell_env, which
provides clang 19, cmake, pkg-config and a static OpenCV 4.13.0. Nothing here
builds outside it, since the headers and libraries are only on the search path
inside the shell.

## Building

CMake creates the build directory itself, so there is no `mkdir` step.

```console
$ nix develop github:ormandi/nix_shell_env
$ cmake -S . -B build
-- Checking for module 'opencv4'
--   Found opencv4, version 4.13.0
-- Configuring done (1.0s)
-- Generating done (0.0s)
-- Build files have been written to: .../opencv_example_cmake/build

$ cmake --build build
[ 50%] Building CXX object CMakeFiles/demo.dir/main.cc.o
[100%] Linking CXX executable demo
[100%] Built target demo

$ ./build/demo test.png
test.png: 64x64, 3 channel(s)
```

## What is going on

OpenCV is linked statically, so the `libopencv_*.a` archives end up inside the
binary; `otool -L build/demo` (or `ldd` on Linux) lists no OpenCV libraries at
all. The lookup goes through pkg-config rather than `find_package(OpenCV)`,
because the static CMake export names `Eigen3::Eigen` in its link interface
without importing it and fails outright. `PKG_CONFIG_USE_STATIC_LIBS` is what
pulls in the image codecs, which stay dynamic; the `list(FILTER ...)` and the
explicit `tiff` exist because OpenCV's generated `opencv4.pc` names some
libraries that are not real and omits libtiff, which imgcodecs needs. Note that
the macOS window has no close button, so a keypress is the only way to exit.
