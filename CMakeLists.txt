# Copyright (c) Facebook, Inc. and its affiliates.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
cmake_minimum_required(VERSION 3.14)

# the policy allows us to change options without caching
cmake_policy(SET CMP0077 NEW)
set(CMAKE_POLICY_DEFAULT_CMP0077 NEW)

# set the project name
project(velox VERSION 0.0.1)

option(VELOX_ENABLE_DUCKDB "Build duckDB to enable differential testing." ON)
option(VELOX_ENABLE_EXEC "Build exec." ON)
option(VELOX_ENABLE_AGGREGATES "Build aggregates." ON)
option(VELOX_ENABLE_HIVE_CONNECTOR "Build Hive connector." ON)
option(VELOX_ENABLE_TPCH_CONNECTOR "Build TPC-H connector." ON)
option(VELOX_ENABLE_PRESTO_FUNCTIONS "Build Presto SQL functions." ON)
option(VELOX_ENABLE_SPARK_FUNCTIONS "Build Spark SQL functions." ON)
option(VELOX_ENABLE_EXPRESSION "Build expression." ON)
option(VELOX_ENABLE_PARSE "Build parser used for unit tests." ON)
option(VELOX_ENABLE_EXAMPLES
       "Build examples. This will enable VELOX_ENABLE_EXPRESSION automatically."
       OFF)
option(VELOX_ENABLE_SUBSTRAIT "Buid Substrait-to-Velox converter." OFF)
option(VELOX_ENABLE_BENCHMARKS "Enable Velox top level benchmarks." OFF)
option(VELOX_ENABLE_BENCHMARKS_BASIC "Enable Velox basic benchmarks." OFF)
option(VELOX_ENABLE_S3 "Build S3 Connector" OFF)
option(VELOX_ENABLE_HDFS "Build Hdfs Connector" OFF)
option(VELOX_ENABLE_PARQUET "Enable Parquet support" OFF)
option(VELOX_ENABLE_ARROW "Enable Arrow support" OFF)

option(VELOX_BUILD_TEST_UTILS "Builds Velox test utilities" OFF)
option(VELOX_BUILD_PYTHON_PACKAGE "Builds Velox Python bindings" OFF)
option(VELOX_BUILD_BENCHMARKS "Builds Velox benchmarks" OFF)

if(TRUE)
  # Enable and disable components for velox base build
  set(VELOX_BUILD_TESTING OFF)
  set(VELOX_ENABLE_PRESTO_FUNCTIONS ON)
  set(VELOX_ENABLE_DUCKDB OFF)
  set(VELOX_ENABLE_EXPRESSION ON)
  set(VELOX_ENABLE_PARSE OFF)
  set(VELOX_ENABLE_EXEC OFF)
  set(VELOX_ENABLE_AGGREGATES OFF)
  set(VELOX_ENABLE_HIVE_CONNECTOR OFF)
  set(VELOX_ENABLE_TPCH_CONNECTOR OFF)
  set(VELOX_ENABLE_SPARK_FUNCTIONS OFF)
  set(VELOX_ENABLE_EXAMPLES OFF)
  set(VELOX_ENABLE_S3 OFF)
  set(VELOX_ENABLE_SUBSTRAIT OFF)
  set(VELOX_CODEGEN_SUPPORT OFF)
endif()

# Under Ninja, we are able to designate certain targets large enough to require
# restricted parallelism.
if("${MAX_HIGH_MEM_JOBS}")
  set_property(GLOBAL PROPERTY JOB_POOLS
                               "high_memory_pool=${MAX_HIGH_MEM_JOBS}")
else()
  set_property(GLOBAL PROPERTY JOB_POOLS high_memory_pool=1000)
endif()

if("${MAX_LINK_JOBS}")
  set_property(GLOBAL APPEND PROPERTY JOB_POOLS
                                      "link_job_pool=${MAX_LINK_JOBS}")
  set(CMAKE_JOB_POOL_LINK link_job_pool)
endif()

if("${TREAT_WARNINGS_AS_ERRORS}")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Werror")
endif()

set(BOOST_INCLUDE_LIBRARIES
    atomic
    context
    date_time
    filesystem
    program_options
    regex
    system
    thread)

find_package(Boost 1.66.0 COMPONENTS ${BOOST_INCLUDE_LIBRARIES})
find_package(
  gflags
  COMPONENTS shared
  REQUIRED)
add_library(gflags::gflags ALIAS gflags_shared)
find_package(glog REQUIRED)
find_package(folly REQUIRED)
add_library(folly::folly ALIAS Folly::folly)
find_package(fmt)
find_library(EVENT event)
find_library(DOUBLE_CONVERSION double-conversion)
find_library(RE2 re2 REQUIRED)
find_package(BISON 3.0.4 REQUIRED)
find_package(FLEX 2.5.13 REQUIRED)

include(CTest) # include after project() but before add_subdirectory()

add_library(velox INTERFACE)
target_compile_features(velox INTERFACE cxx_std_17)
set_target_properties(velox PROPERTIES POSITION_INDEPENDENT_CODE ON)
# TODO use target_include_dirs
include_directories(SYSTEM ${PROJECT_SOURCE_DIR})
include_directories(SYSTEM ${PROJECT_SOURCE_DIR}/velox)
include_directories(SYSTEM ${PROJECT_SOURCE_DIR}/velox/external)

add_library(velox_minimal INTERFACE)
target_link_libraries(velox_minimal INTERFACE velox_functions_prestosql
                                              velox_expression)
target_link_libraries(velox INTERFACE velox_minimal)

# TODO make target based
if("${ENABLE_ALL_WARNINGS}")
  if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    add_compile_options(-Wno-range-loop-analysis - Wno-mismatched-tags
                        -Wno-nullability-completeness)
  elseif(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    add_compile_options(
      -Wno-implicit-fallthrough
      -Wno-empty-body
      -Wno-class-memaccess
      -Wno-comment
      -Wno-int-in-bool-context
      -Wno-redundant-move
      -Wno-array-bounds
      -Wno-maybe-uninitialized
      -Wno-unused-result
      -Wno-format-overflow
      -Wno-strict-aliasing
      -Wno-type-limits)
  endif()

  add_compile_options(-Wno-unused -Wno-unused-parameter -Wno-sign-compare
                      -Wno-ignored-qualifiers -Wall -Wextra)

endif()

# TODO make os dependent etc
add_compile_options(-mavx2 -mfma -mavx -mf16c -mlzcnt -mbmi2)

message("FINAL CMAKE_CXX_FLAGS=${CMAKE_CXX_FLAGS}")

set_target_properties(velox PROPERTIES EXPORT_COMPILE_COMMANDS ON)

add_subdirectory(velox)
add_subdirectory(third_party)
add_library(xsimd::xsimd ALIAS xsimd)
