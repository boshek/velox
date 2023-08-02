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

add_subdirectory(utils)

add_executable(
  velox_dwio_common_test
  BitConcatenationTest.cpp
  BitPackDecoderTest.cpp
  ChainedBufferTests.cpp
  ColumnSelectorTests.cpp
  DataBufferTests.cpp
  DecoderUtilTest.cpp
  LocalFileSinkTest.cpp
  LoggedExceptionTest.cpp
  RangeTests.cpp
  ReadFileInputStreamTests.cpp
  RetryTests.cpp
  TestBufferedInput.cpp
  TypeTests.cpp)
add_test(velox_dwio_common_test velox_dwio_common_test)
target_link_libraries(
  velox_dwio_common_test
  velox_dwio_common_test_utils
  velox_temp_path
  Boost::regex
  velox_link_libs
  Folly::folly
  ${TEST_LINK_LIBS}
  gflags::gflags
  gtest
  gtest_main
  gmock
  glog::glog
  fmt::fmt)

add_executable(velox_dwio_common_data_buffer_benchmark DataBufferBenchmark.cpp)

target_link_libraries(
  velox_dwio_common_data_buffer_benchmark velox_dwio_common velox_memory
  velox_dwio_common_exception Folly::folly ${FOLLY_BENCHMARK})

add_executable(velox_dwio_common_int_decoder_benchmark IntDecoderBenchmark.cpp)
target_link_libraries(
  velox_dwio_common_int_decoder_benchmark velox_dwio_common_exception
  velox_exception velox_dwio_dwrf_common Folly::folly ${FOLLY_BENCHMARK})

add_library(velox_e2e_filter_test_base E2EFilterTestBase.cpp)

target_link_libraries(
  velox_e2e_filter_test_base
  PUBLIC velox_dwio_common
         velox_dwio_common_test_utils
         velox_link_libs
         velox_time
         velox_type
         velox_type_fbhive
         velox_vector
         ${TEST_LINK_LIBS}
         gtest
  PRIVATE velox_expression velox_functions_prestosql velox_parse_expression
          velox_parse_parser velox_parse_utils velox_vector_test_lib)

if(VELOX_ENABLE_ARROW)
  add_executable(velox_dwio_common_bitpack_decoder_benchmark
                 BitPackDecoderBenchmark.cpp)

  # The duckdb include on BitPackDecoderTest.cpp triggers this error in gcc.
  if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    target_compile_options(velox_dwio_common_bitpack_decoder_benchmark
                           PRIVATE -Wno-deprecated-declarations)
  endif()

  target_link_libraries(
    velox_dwio_common_bitpack_decoder_benchmark velox_dwio_common arrow duckdb
    Folly::folly ${FOLLY_BENCHMARK})
endif()
