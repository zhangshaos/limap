# cmake version to be used
cmake_minimum_required( VERSION 3.17 )

# project name
project(LIMAP)

if(MSVC)
    add_compile_options(
            /utf-8
            /bigobj
            /MP)
    add_compile_definitions(
            _USE_MATH_DEFINES
            _CRT_SECURE_NO_WARNINGS)
endif()
set(CMAKE_CXX_STANDARD 17)
set(CMAKK_CXX_STANDARD_REQUIRED ON) #表示编译时c++标准不会在无法满足时向下取，而是会直接报错
set(BUILD_SHARED_LIBS OFF) #只允许静态库
set(CMAKE_POSITION_INDEPENDENT_CODE FALSE) #fix https://discourse.cmake.org/t/windows-clang-platform-adds-fpic-for-shared-dlls/1784
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/lib) # 静态库 or .lib
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/lib) # .so
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/bin) # .exe or .dll

################################################################################
# Include CMake dependencies
################################################################################
if(CMAKE_CXX_COMPILER_VERSION VERSION_LESS 9.1)
    message(FATAL_ERROR "GCC version needs to be at least 9.1")
endif()
set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

# Include helper macros and commands, and allow the included file to override
# the CMake policies in this file
include(${CMAKE_CURRENT_SOURCE_DIR}/cmake/CMakeHelper.cmake NO_POLICY_SCOPE)

################################################################################
# Options
################################################################################
option(OPENMP_ENABLED "Whether to enable OpenMP parallelization" ON)
option(INTERPOLATION_ENABLED "Whether to enable interpolation-based pixel-perfect optimization" OFF)

################################################################################
# Compiler specific configuration
################################################################################

if(OPENMP_ENABLED)
  find_package(OpenMP)
  if(OPENMP_FOUND)
    message(STATUS "Enabling OpenMP support")
    add_definitions("-DOPENMP_ENABLED")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
  endif()
endif()

if(INTERPOLATION_ENABLED)
    message(STATUS "Enabling pixelwise optimization with ceres interpolation. This should be disabled for clang.")
    add_definitions("-DINTERPOLATION_ENABLED")
else()
    message(STATUS "Disabling pixelwise optimization with ceres interpolation.")
endif()

################################################################################
# Find packages
################################################################################
find_package(Eigen3 3.3 REQUIRED)
find_package(COLMAP REQUIRED)
if(${CERES_VERSION} VERSION_LESS "2.2.0")
    # ceres 2.2.0 changes the interface of local parameterization
    add_definitions("-DCERES_PARAMETERIZATION_ENABLED")
endif()
add_subdirectory(vendor)

################################################################################
# Add sources
################################################################################

set(LIMAP_INCLUDE_DIRS
  ${HDF5_INCLUDE_DIRS}
  ${EIGEN3_INCLUDE_DIR}
  ${PROJECT_SOURCE_DIR}
  ${FREEIMAGE_INCLUDE_DIRS}
  ${COLMAP_INCLUDE_DIRS}
)

set(LIMAP_LINK_DIRS
  ${COLMAP_LINK_DIRS}
)

set(LIMAP_EXTERNAL_LIBRARIES
  ${CERES_LIBRARIES}
  ${FREEIMAGE_LIBRARIES}
  ${COLMAP_LIBRARIES}
  ${HDF5_C_LIBRARIES}
  ${Boost_LIBRARIES}
  PoseLib::PoseLib
)

if(OPENMP_FOUND)
    list(APPEND LIMAP_EXTERNAL_LIBRARIES ${OpenMP_libomp_LIBRARY})
endif()

set(LIMAP_INTERNAL_LIBRARIES
  HighFive
  pybind11::module
  JLinkage
  igl::core
)

include_directories(
    third-party
    vendor
    limap
    ${LIMAP_INCLUDE_DIRS}
)

link_directories(${LIMAP_LINK_DIRS})

include_directories(${JLINKAGE_INCLUDE_DIRS})
include_directories(${POSELIB_INCLUDE_DIRS})
include_directories(${RANSACLIB_INCLUDE_DIRS})
add_subdirectory(limap)

################################################################################
# Generate source groups for Visual Studio, XCode, etc.
################################################################################
LIMAP_ADD_SOURCE_DIR(limap/base BASE_SRCS *.h *.cc)
LIMAP_ADD_SOURCE_DIR(limap/ceresbase BASE_SRCS *.h *.cc)
LIMAP_ADD_SOURCE_DIR(limap/solvers BASE_SRCS *.h *.cc)
LIMAP_ADD_SOURCE_DIR(limap/vplib BASE_SRCS *.h *.cc)
LIMAP_ADD_SOURCE_DIR(limap/pointsfm BASE_SRCS *.h *.cc)
LIMAP_ADD_SOURCE_DIR(limap/undistortion BASE_SRCS *.h *.cc)
LIMAP_ADD_SOURCE_DIR(limap/fitting BASE_SRCS *.h *.cc)
LIMAP_ADD_SOURCE_DIR(limap/estimators BASE_SRCS *.h *.cc)
LIMAP_ADD_SOURCE_DIR(limap/triangulation BASE_SRCS *.h *.cc)
LIMAP_ADD_SOURCE_DIR(limap/merging BASE_SRCS *.h *.cc)
LIMAP_ADD_SOURCE_DIR(limap/features BASE_SRCS *.h *.cc)
LIMAP_ADD_SOURCE_DIR(limap/evaluation BASE_SRCS *.h *.cc)
LIMAP_ADD_SOURCE_DIR(limap/structures BASE_SRCS *.h *.cc)
LIMAP_ADD_SOURCE_DIR(limap/optimize BASE_SRCS *.h *.cc)
LIMAP_ADD_SOURCE_DIR(limap/optimize/line_refinement BASE_SRCS *.h *.cc)
LIMAP_ADD_SOURCE_DIR(limap/optimize/hybrid_bundle_adjustment BASE_SRCS *.h *.cc)
LIMAP_ADD_SOURCE_DIR(limap/optimize/global_pl_association BASE_SRCS *.h *.cc)
LIMAP_ADD_SOURCE_DIR(limap/optimize/line_localization BASE_SRCS *.h *.cc)

