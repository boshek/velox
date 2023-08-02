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

add_library(velox_temp_path TempFilePath.cpp TempDirectoryPath.cpp)

target_link_libraries(
  velox_temp_path
  PUBLIC velox_exception
  PRIVATE Boost::filesystem)

add_library(
  velox_exec_test_lib
  AssertQueryBuilder.cpp
  Cursor.cpp
  HiveConnectorTestBase.cpp
  LocalExchangeSource.cpp
  OperatorTestBase.cpp
  PlanBuilder.cpp
  QueryAssertions.cpp
  SumNonPODAggregate.cpp
  TpchQueryBuilder.cpp
  VectorTestUtil.cpp)

target_link_libraries(
  velox_exec_test_lib
  PUBLIC velox_caching
         velox_core
         velox_dwio_common
         velox_dwio_dwrf_common
         velox_exec
         velox_hive_connector
         velox_memory
         velox_parse_parser
         velox_temp_path
         velox_tpch_connector
         velox_type
         velox_vector
         velox_vector_fuzzer
         velox_vector_test_lib
         Folly::folly
         gtest
  PRIVATE velox_aggregates
          velox_common_base
          velox_duckdb_conversion
          velox_duckdb_parser
          velox_dwio_common_test_utils
          velox_dwio_dwrf_reader
          velox_dwio_dwrf_writer
          velox_expression
          velox_expression_functions
          velox_file
          velox_functions_prestosql
          velox_parse_expression
          velox_parse_utils
          velox_presto_serializer
          velox_test_util
          velox_tpch_gen)
