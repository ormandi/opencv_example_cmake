cmake_minimum_required(VERSION 3.20)
project(opencv_example CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# opencv is static here, so pkg-config has to report Libs.private too. Use the
# plain pkg_check_modules form: IMPORTED_TARGET drops the -L dirs the codecs
# live under. find_package(OpenCV) does not work, as the static export names
# Eigen3::Eigen without importing it.
set(PKG_CONFIG_USE_STATIC_LIBS ON)
find_package(PkgConfig REQUIRED)
pkg_check_modules(OpenCV REQUIRED opencv4)

# OpenCV's generated opencv4.pc leaks cmake target names and a gflags alias
# that does not exist as a library. Both come from the sfm module, which this
# demo does not call into, so drop them.
set(demo_libs ${OpenCV_STATIC_LIBRARIES})
list(FILTER demo_libs EXCLUDE REGEX "::|^gflags_shared$|^opencv\\.sfm\\.")

add_executable(demo main.cc)
target_include_directories(demo PRIVATE ${OpenCV_STATIC_INCLUDE_DIRS})
target_link_directories(demo PRIVATE ${OpenCV_STATIC_LIBRARY_DIRS})
target_link_libraries(demo PRIVATE ${demo_libs})

# imgcodecs calls into libtiff, which opencv4.pc never lists.
target_link_libraries(demo PRIVATE tiff)

# SHELL: keeps each "-framework X" pair intact; cmake would otherwise split it
# and pass a bare -lCocoa.
if(APPLE)
  target_link_options(demo PRIVATE "SHELL:-framework Cocoa"
                                   "SHELL:-framework AppKit")
endif()
