project (Tapkee  LANGUAGES CXX)

cmake_minimum_required (VERSION 3.3)

# cmake version > 3.1
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# set paths
set (TAPKEE_INCLUDE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/include")
set (TAPKEE_SRC_DIR "${CMAKE_CURRENT_SOURCE_DIR}/src")
set (TAPKEE_TESTS_DIR "${CMAKE_CURRENT_SOURCE_DIR}/test/unit")
set (CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/src/cmake")

set (CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/lib")
set (CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/lib")
set (CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/bin")

# Eigen3 detection
find_package(Eigen3 REQUIRED)
include_directories(SYSTEM "${EIGEN3_INCLUDE_DIR}")

# ARPACK detection
find_package(Arpack)
if (ARPACK_FOUND)
	link_directories("${ARPACK_PATH}")
endif()

# OpenMP detection
find_package(OpenMP)
if (OPENMP_FOUND)
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
	set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}")
endif()

# ViennaCL detection
find_package(OpenCL)
if (OPENCL_FOUND)
	find_package(ViennaCL)
	if (VIENNACL_FOUND)
		set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OPENCL_C_FLAGS}")
		set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OPENCL_CXX_FLAGS}")
		set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OPENCL_EXE_LINKER_FLAGS}")
		include_directories("${VIENNACL_INCLUDE_DIRS}")
	endif()
endif()

include_directories("${TAPKEE_INCLUDE_DIR}")
add_library(tapkee_library INTERFACE)
target_include_directories(tapkee_library INTERFACE "${TAPKEE_INCLUDE_DIR}" "${EIGEN3_INCLUDE_DIR}")

if (ARPACK_FOUND)
	include_directories(${OPENGL_INCLUDE_DIR} ${GLEW_INCLUDE_DIRS})
	target_link_libraries(tapkee_library INTERFACE arpack)
	add_definitions(-DTAPKEE_WITH_ARPACK)
endif()

if (VIENNACL_FOUND)
	target_link_libraries(tapkee_library INTERFACE OpenCL)
	add_definitions(-DTAPKEE_WITH_VIENNACL)
endif()



if (CMAKE_COMPILER_IS_GNUCXX)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -pedantic -Wno-long-long -Wshadow")
endif()

if (CMAKE_BUILD_TYPE MATCHES Debug)
	add_definitions(-DTAPKEE_DEBUG)
endif()

option(PRECOMPUTED "Precompute kernel/distance matrices before reducing dimensionality" OFF)

if (PRECOMPUTED)
	add_definitions(-DUSE_PRECOMPUTED)
endif()

option(GPL_FREE "Build without GPL-licensed components" OFF)

if (NOT GPL_FREE)
	add_definitions(-DTAPKEE_USE_LGPL_COVERTREE)
endif()



