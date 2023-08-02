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
if(${VELOX_BUILD_TESTING})
  add_subdirectory(tests)
endif()

add_subdirectory(tz)
add_subdirectory(fbhive)

add_library(
  velox_type
  DecimalUtil.cpp
  DoubleUtil.cpp
  Filter.cpp
  HugeInt.cpp
  StringView.h
  Subfield.cpp
  Timestamp.cpp
  TimestampConversion.cpp
  Tokenizer.cpp
  Type.cpp
  Variant.cpp)

target_link_libraries(
  velox_type
  PUBLIC velox_common_base velox_exception velox_serialization Boost::headers Folly::folly fmt::fmt
  PRIVATE velox_encode velox_external_date velox_type_tz re2::re2)
