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

add_library(
  velox_hive_connector OBJECT
  FileHandle.cpp
  HiveConfig.cpp
  HiveConnector.cpp
  HiveDataSink.cpp
  HiveDataSource.cpp
  HivePartitionUtil.cpp
  PartitionIdGenerator.cpp
  TableHandle.cpp)

target_link_libraries(
  velox_hive_connector
  PUBLIC velox_caching
         velox_common_compression
         velox_connector
         velox_core
         velox_dwio_common
         velox_exec
         velox_expression
         velox_file
         velox_type
         velox_vector
         Folly::folly
  PRIVATE velox_common_base
          velox_config
          velox_dwio_catalog_fbhive
          velox_dwio_dwrf_reader
          velox_dwio_dwrf_writer
          velox_dwio_parquet_reader
          velox_dwio_parquet_writer
          velox_gcs
          velox_hdfs
          velox_hive_partition_function
          velox_s3fs
          velox_time
          Boost::headers)

add_library(velox_hive_partition_function HivePartitionFunction.cpp)

target_link_libraries(velox_hive_partition_function PUBLIC velox_core
                                                           velox_vector)

add_subdirectory(storage_adapters)

if(${VELOX_BUILD_TESTING})
  add_subdirectory(tests)
endif()

if(${VELOX_ENABLE_BENCHMARKS})
  add_subdirectory(benchmarks)
endif()
