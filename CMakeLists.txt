cmake_minimum_required(VERSION 4.2)
project(InstantMeshes)

# Default to Release build
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  message(STATUS "Setting build type to 'Release' as none was specified.")
  set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build." FORCE)
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

# Enable C++11
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS ON)

option(INSTANT_MESHES_DEV_MODE "Instant meshes developer mode" OFF)

# Check submodules
if(NOT IS_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/ext/nanogui/ext/glfw")
  message(FATAL_ERROR "The Instant Meshes dependency repositories (NanoGUI, GLFW, etc.) are missing! "
    "You probably did not clone the project with --recursive. It is possible to recover "
    "by calling \"git submodule update --init --recursive\"")
endif()

# Enable folders for projects in Visual Studio
set_property(GLOBAL PROPERTY USE_FOLDERS ON)

# Build NanoGUI
set(NANOGUI_BUILD_EXAMPLE OFF CACHE BOOL " " FORCE)
set(NANOGUI_BUILD_SHARED  OFF CACHE BOOL " " FORCE)
set(NANOGUI_BUILD_PYTHON  OFF CACHE BOOL " " FORCE)
add_subdirectory(ext/nanogui ext_build/nanogui)

# Build TBB
set(TBB_BUILD_STATIC ON CACHE BOOL " " FORCE)
set(TBB_BUILD_SHARED OFF CACHE BOOL " " FORCE)
set(TBB_BUILD_TBBMALLOC OFF CACHE BOOL " " FORCE)
set(TBB_BUILD_TBBMALLOC_PROXY OFF CACHE BOOL " " FORCE)
set(TBB_BUILD_TESTS OFF CACHE BOOL " " FORCE)
add_subdirectory(ext/tbb ext_build/tbb)

# Global compiler settings
if (MSVC)
  add_definitions(/D "_CRT_SECURE_NO_WARNINGS" /D "__TBB_NO_IMPLICIT_LINKAGE")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /MP")
  if (NOT CMAKE_SIZEOF_VOID_P EQUAL 8)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /arch:SSE2 /DEIGEN_DONT_ALIGN")
  endif()
  # Use static CRT for static build
  foreach(flag_var
    CMAKE_CXX_FLAGS CMAKE_CXX_FLAGS_DEBUG CMAKE_CXX_FLAGS_RELEASE
    CMAKE_CXX_FLAGS_MINSIZEREL CMAKE_CXX_FLAGS_RELWITHDEBINFO
    CMAKE_C_FLAGS CMAKE_C_FLAGS_DEBUG CMAKE_C_FLAGS_RELEASE
    CMAKE_C_FLAGS CMAKE_C_FLAGS_MINSIZEREL CMAKE_C_FLAGS_RELWITHDEBINFO)
    string(REPLACE "/MD" "/MT" ${flag_var} "${${flag_var}}")
  endforeach()
endif()

# Resource generation
set(PNG_FILES
  "${CMAKE_CURRENT_SOURCE_DIR}/resources/comb.png"
  "${CMAKE_CURRENT_SOURCE_DIR}/resources/delete_stroke.png"
  "${CMAKE_CURRENT_SOURCE_DIR}/resources/loadmesh.png"
  "${CMAKE_CURRENT_SOURCE_DIR}/resources/scare.png"
  "${CMAKE_CURRENT_SOURCE_DIR}/resources/sing_dir.png"
  "${CMAKE_CURRENT_SOURCE_DIR}/resources/sing_pos.png"
  "${CMAKE_CURRENT_SOURCE_DIR}/resources/instantmeshes.png")
file(GLOB_RECURSE RESOURCE_FILES
  "${CMAKE_CURRENT_SOURCE_DIR}/resources/*.frag"
  "${CMAKE_CURRENT_SOURCE_DIR}/resources/*.vert"
  "${CMAKE_CURRENT_SOURCE_DIR}/resources/*.geo")
list(APPEND RESOURCE_FILES ${PNG_FILES})

string (REPLACE ";" "," RESOURCE_FILES_STRING "${RESOURCE_FILES}")

add_custom_command(
  OUTPUT resources.cpp resources.h
  COMMAND ${CMAKE_COMMAND}
          -DOUTPUT_C=resources.cpp
          -DOUTPUT_H=resources.h
          "-DINPUT_FILES=${RESOURCE_FILES_STRING}"
          -P "${CMAKE_CURRENT_SOURCE_DIR}/resources/bin2c.cmake"
  DEPENDS ${RESOURCE_FILES}
  COMMENT "Generating resources from PNGs and shaders"
  VERBATIM)

# Create the main executable
add_executable(InstantMeshes MACOSX_BUNDLE)

target_sources(InstantMeshes PRIVATE
  resources.cpp
  src/glutil.h src/glutil.cpp
  src/widgets.h src/widgets.cpp
  src/aabb.h
  src/main.cpp src/common.h
  src/meshio.h src/meshio.cpp
  src/normal.h src/normal.cpp
  src/adjacency.h src/adjacency.cpp
  src/meshstats.h src/meshstats.cpp
  src/hierarchy.h src/hierarchy.cpp
  src/extract.h src/extract.cpp
  src/viewer.h src/viewer.cpp
  src/field.h src/field.cpp
  src/bvh.h src/bvh.cpp
  src/subdivide.h src/subdivide.cpp
  src/reorder.h src/reorder.cpp
  src/serializer.h src/serializer.cpp
  src/batch.h src/batch.cpp
  src/smoothcurve.h src/smoothcurve.cpp
  src/cleanup.h src/cleanup.cpp
  src/dedge.h src/dedge.cpp
  ext/rply/rply.c
)

if (APPLE)
  target_sources(InstantMeshes PRIVATE resources/im.icns)
  set_source_files_properties(resources/im.icns PROPERTIES MACOSX_PACKAGE_LOCATION "Resources")
elseif(WIN32)
  target_sources(InstantMeshes PRIVATE resources/im.rc)
endif()

target_include_directories(InstantMeshes PRIVATE
  src
  ext/nanogui/include
  ${NANOGUI_EXTRA_INCS}
  ext/tbb/include
  ext/dset
  ext/pss
  ext/pcg32
  ext/rply
  ext/half
  ${CMAKE_CURRENT_BINARY_DIR}
)

target_compile_definitions(InstantMeshes PRIVATE ${NANOGUI_EXTRA_DEFS})
if (INSTANT_MESHES_DEV_MODE)
  target_compile_definitions(InstantMeshes PRIVATE DEV_MODE)
endif()

target_link_libraries(InstantMeshes PRIVATE
  nanogui
  tbb_static
  ${NANOGUI_EXTRA_LIBS}
)

set_target_properties(InstantMeshes PROPERTIES
  OUTPUT_NAME "Instant Meshes"
  MACOSX_BUNDLE_BUNDLE_NAME "Instant Meshes"
  MACOSX_BUNDLE_BUNDLE_GUI_IDENTIFIER "com.im.InstantMeshes"
  MACOSX_BUNDLE_ICON_FILE im.icns
  MACOSX_BUNDLE_INFO_PLIST ${CMAKE_CURRENT_SOURCE_DIR}/resources/MacOSXBundleInfo.plist.in
)

# Compiler warnings
if(MSVC)
  target_compile_options(InstantMeshes PRIVATE /W4)
  set_source_files_properties(ext/rply/rply.c PROPERTIES COMPILE_FLAGS "/wd4127")
else()
  target_compile_options(InstantMeshes PRIVATE -Wall -Wextra -Wno-unused-parameter)
  if (CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    target_compile_options(InstantMeshes PRIVATE -Wno-gnu-anonymous-struct -Wno-c99-extensions -Wno-nested-anon-types)
  endif()
endif()

if (NOT APPLE AND NOT WIN32)
  set_target_properties(InstantMeshes PROPERTIES LINK_FLAGS "-static-libgcc")
endif()

if (UNIX AND NOT APPLE AND NOT CMAKE_BUILD_TYPE MATCHES Debug)
  add_custom_command(TARGET InstantMeshes POST_BUILD COMMAND strip $<TARGET_FILE:InstantMeshes>)
endif()
