cmake_minimum_required(VERSION 3.10)
project(PoseLib VERSION 2.0.0)

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
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/lib) # 静态库 or .lib
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/lib) # .so
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/bin) # .exe or .dll

# Set variables
set(LIBRARY_NAME   ${PROJECT_NAME})
set(LIBRARY_FOLDER ${PROJECT_NAME})
include(${PROJECT_SOURCE_DIR}/cmake/SetEnv.cmake)

# Eigen
find_package(Eigen3)

# Library sources
add_subdirectory(${LIBRARY_FOLDER})

# Benchmark
option(WITH_BENCHMARK "Build benchmark example." OFF)
if(WITH_BENCHMARK)
	add_subdirectory(benchmark)
endif()

# python bindings
option(PYTHON_PACKAGE "Build python package." OFF)
if(PYTHON_PACKAGE)
	add_subdirectory(pybind)
endif()

# Compilation options
if(MSVC)
	target_compile_options(${LIBRARY_NAME} PRIVATE /bigobj /fp:fast)
else()
	target_compile_options(${LIBRARY_NAME} PRIVATE
		-march=native -ffast-math -fno-unsafe-math-optimizations
		-funroll-loops -fprefetch-loop-arrays -funswitch-loops
	 	-Wall -Werror -fPIC -Wno-ignored-optimization-argument)
	if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
		target_compile_options(${LIBRARY_NAME} PRIVATE
				-Wno-maybe-uninitialized)
	endif()
endif()
