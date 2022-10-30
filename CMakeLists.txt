## Copyright 2021 Intel Corporation
## SPDX-License-Identifier: Apache-2.0

cmake_minimum_required(VERSION 3.12)

if(NOT CMAKE_INSTALL_PREFIX)
  set(CMAKE_INSTALL_PREFIX
      "${CMAKE_BINARY_DIR}/install"
      CACHE STRING "Final install location." FORCE)
endif()

project( STD_A5_RenderKit_superbuild LANGUAGES CXX C   VERSION  "8.8.8.8"  )





#set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
if(MSVC)
    set(CMAKE_DEBUG_POSTFIX "d")
endif()
#link_directories(${CMAKE_BINARY_DIR} )

message( "CMAKE_C_COMPILER = ${CMAKE_C_COMPILER}"  )
message( "CMAKE_C_COMPILER_VERSION = ${CMAKE_C_COMPILER_VERSION}"  )
message( "CMAKE_CXX_PLATFORM_ID = ${CMAKE_CXX_PLATFORM_ID}"  )
message( "CMAKE_CXX_COMPILER_TARGET = ${CMAKE_CXX_COMPILER_TARGET}"  )
message( "CMAKE_CXX_COMPILER_AR = ${CMAKE_CXX_COMPILER_AR}"  )
message( "CMAKE_C_COMPILER_ABI = ${CMAKE_C_COMPILER_ABI}"  )
message( "CMAKE_C_COMPILER_TARGET = ${CMAKE_C_COMPILER_TARGET}"  )
message( "CMAKE_HOST_SYSTEM = ${CMAKE_HOST_SYSTEM}"  )
message( "CMAKE_HOST_SYSTEM_NAME = ${CMAKE_HOST_SYSTEM_NAME}"  )
message( "CMAKE_VS_PLATFORM_NAME = ${CMAKE_VS_PLATFORM_NAME}"  )
message( "CMAKE_VS_PLATFORM_TOOLSET = ${CMAKE_VS_PLATFORM_TOOLSET}"  )
message( "CMAKE_CXX_COMPILER_VERSION = ${CMAKE_CXX_COMPILER_VERSION}"  )
#message(  "CMAKE_CXX_COMPILER_TARGET= ${CMAKE_CXX_COMPILER_TARGET}"   )
message(  "CMAKE_CXX_COMPILER_ABI= ${CMAKE_CXX_COMPILER_ABI}"   )
message(  "CMAKE_CXX_COMPILER_EXTERNAL_TOOLCHAIN= ${CMAKE_CXX_COMPILER_EXTERNAL_TOOLCHAIN}"   )
message(  "CMAKE_CXX_COMPILER_ID= ${CMAKE_CXX_COMPILER_ID}"   )
message(  "CMAKE_CXX_COMPILER_ARCHITECTURE_ID= ${CMAKE_CXX_COMPILER_ARCHITECTURE_ID}"   )
#message(  "CMAKE_HOST_SYSTEM_NAME= ${CMAKE_HOST_SYSTEM_NAME}"   )
message(  "CMAKE_Fortran_FORMAT= ${CMAKE_Fortran_FORMAT}"   )
message(  "CMAKE_Fortran_COMPILER= ${CMAKE_Fortran_COMPILER}"   )
message(  "CMAKE_Fortran_COMPILER_VERSION= ${CMAKE_Fortran_COMPILER_VERSION}"   )
message(  "CMAKE_Fortran_COMPILER_ID= ${CMAKE_Fortran_COMPILER_ID}"   )



# 改变安装路径可以使用下面的语句，Unix: "/usr/local", Windows: "c:/Program Files/${PROJECT_NAME}" 为默认值，所有的DESTINATION均以此目录为根目录
# set(CMAKE_INSTALL_PREFIX ${CMAKE_BINARY_DIR}/mymath)
#set(CMAKE_INSTALL_PREFIX ${CMAKE_BINARY_DIR}/../${PROJECT_NAME}_${PROJECT_VERSION}_${CMAKE_HOST_SYSTEM}_${CMAKE_CXX_COMPILER_ID}_${CMAKE_CXX_COMPILER_VERSION}_install)
#set(CMAKE_INSTALL_PREFIX ${CMAKE_BINARY_DIR}/../${PROJECT_NAME}_${PROJECT_VERSION}_${CMAKE_CXX_COMPILER_ID}_${CMAKE_CXX_COMPILER_VERSION}_${CMAKE_Fortran_COMPILER_ID}_${CMAKE_Fortran_COMPILER_VERSION}_install)







set ( PreferredToolArchitecture  x64 )
# -T host=x64 来指定默认的cpp编译器为64位的。
#cmake -G "Visual Studio 14 2015" -A x64 -T host=x64 '<cmake源码目录>',



include(CMakeDependentOption)
include(ExternalProject)
include(GNUInstallDirs)
include(ProcessorCount)

include(macros.cmake)

# Options ##

ProcessorCount(PROCESSOR_COUNT)
set(NUM_BUILD_JOBS ${PROCESSOR_COUNT} CACHE STRING "Number of build jobs '-j <n>'")

set(DEFAULT_BUILD_COMMAND cmake --build . --config release -j ${NUM_BUILD_JOBS})

get_filename_component(INSTALL_DIR_ABSOLUTE
  ${CMAKE_INSTALL_PREFIX} ABSOLUTE BASE_DIR ${CMAKE_CURRENT_BINARY_DIR})

cmake_dependent_option(
  OSPRAY_ENABLE_MODULE_MPI
  "Enable OSPRay's MPI module"
  ON
  "NOT APPLE"
  OFF
)

## Build projects ##

include(get_tbb.cmake)
include(get_ispc.cmake)
include(get_glfw.cmake)

if(OSPRAY_ENABLE_MODULE_MPI)
  include(get_snappy.cmake)
endif()

set(_RKCOMMON_VERSION 1.10.0)
set(_EMBREE_VERSION 3.13.4)
set(_OPENVKL_VERSION 1.3.0)
set(_OIDN_VERSION 1.4.3)
set(_OSPRAY_VERSION 2.10.0)
set(_OSPRAY_STUDIO_VERSION 0.11.1)

build_subproject(
  NAME rkcommon
  URL "https://github.com/ospray/rkcommon/archive/refs/tags/v${_RKCOMMON_VERSION}.zip"
  BUILD_ARGS
    -DTBB_ROOT=${TBB_PATH}
    -DINSTALL_DEPS=OFF
    -DBUILD_TESTING=OFF
#    -DCMAKE_DEBUG_POSTFIX=d
  DEPENDS_ON tbb ispc
)


build_subproject(
  NAME embree
  URL "https://github.com/embree/embree/archive/refs/tags/v${_EMBREE_VERSION}.zip"
  BUILD_ARGS
    -DTBB_ROOT=${TBB_PATH}
    -DISPC_EXECUTABLE=${ISPC_PATH}
    -DEMBREE_TUTORIALS=OFF
    -DBUILD_TESTING=OFF
#     -DCMAKE_DEBUG_POSTFIX=d
  DEPENDS_ON rkcommon
)

build_subproject(
  NAME openvkl
  URL "https://github.com/openvkl/openvkl/archive/refs/tags/v${_OPENVKL_VERSION}.zip"
  BUILD_ARGS
    -DTBB_ROOT=${TBB_PATH}
    -DISPC_EXECUTABLE=${ISPC_PATH}
    -DBUILD_BENCHMARKS=OFF
    -DBUILD_EXAMPLES=OFF
    -DBUILD_TESTING=OFF
#     -DCMAKE_DEBUG_POSTFIX=d
  DEPENDS_ON rkcommon embree
)

build_subproject(
  NAME oidn
  URL "https://github.com/OpenImageDenoise/oidn/releases/download/v${_OIDN_VERSION}/oidn-${_OIDN_VERSION}.src.zip"
  BUILD_ARGS
    -DTBB_ROOT=${TBB_PATH}
    -DISPC_EXECUTABLE=${ISPC_PATH}
    -DOIDN_APPS=ON
#     -DCMAKE_DEBUG_POSTFIX=d
  DEPENDS_ON rkcommon
)

build_subproject(
  NAME ospray
  URL "https://github.com/ospray/ospray/archive/refs/tags/v${_OSPRAY_VERSION}.zip"
  BUILD_ARGS
    -DTBB_ROOT=${TBB_PATH}
    -DISPC_EXECUTABLE=${ISPC_PATH}
    -DOSPRAY_BUILD_ISA=ALL
    -DOSPRAY_MODULE_BILINEAR_PATCH=OFF
    -DOSPRAY_MODULE_DENOISER=ON
    -DOSPRAY_MODULE_MPI=${OSPRAY_ENABLE_MODULE_MPI}
    -DOSPRAY_ENABLE_APPS=OFF
    -DOSPRAY_STRICT_BUILD=OFF
    -DOSPRAY_INSTALL_DEPENDENCIES=OFF
#     -DCMAKE_DEBUG_POSTFIX=d
     -DA=x64 -Dhost=x64   -DPreferredToolArchitecture=x64
  DEPENDS_ON rkcommon openvkl embree oidn glfw $<$<BOOL:${OSPRAY_ENABLE_MODULE_MPI}>:snappy>
)

build_subproject(
  NAME ospray_studio
  URL "https://github.com/ospray/ospray_studio/archive/refs/tags/v${_OSPRAY_STUDIO_VERSION}.zip"
  BUILD_ARGS
    -DTBB_ROOT=${TBB_PATH}
    -DBUILD_APPS=ON
    -DBUILD_PLUGINS=ON
    -DENABLE_OPENIMAGEIO=OFF  # These can be enabled if the dependency is pre-installed
    -DENABLE_EXR=OFF
    -DENABLE_OPENVDB=OFF
    -DUSE_PYSG=ON
#     -DCMAKE_DEBUG_POSTFIX=d
     -DA=x64 -Dhost=x64   -DPreferredToolArchitecture=x64
  DEPENDS_ON ospray rkcommon openvkl embree oidn glfw
)

if( 0 )
endif()
#
add_executable( ${PROJECT_NAME}  main.cpp   )
#install(TARGETS  ${PROJECT_NAME}
#    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR})
install(TARGETS  ${PROJECT_NAME}
    RUNTIME DESTINATION ${CMAKE_INSTALL_LIBDIR})



# 改变安装路径可以使用下面的语句，Unix: "/usr/local", Windows: "c:/Program Files/${PROJECT_NAME}" 为默认值，所有的DESTINATION均以此目录为根目录
# set(CMAKE_INSTALL_PREFIX ${CMAKE_BINARY_DIR}/mymath)
#set(CMAKE_INSTALL_PREFIX ${CMAKE_BINARY_DIR}/../${PROJECT_NAME}_${PROJECT_VERSION}_${CMAKE_HOST_SYSTEM}_${CMAKE_CXX_COMPILER_ID}_${CMAKE_CXX_COMPILER_VERSION}_install)
set(CMAKE_INSTALL_PREFIX ${CMAKE_BINARY_DIR}/../${PROJECT_NAME}_${PROJECT_VERSION}_${CMAKE_CXX_COMPILER_ID}_${CMAKE_CXX_COMPILER_VERSION}_${CMAKE_Fortran_COMPILER_ID}_${CMAKE_Fortran_COMPILER_VERSION}_install)

