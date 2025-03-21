cmake_minimum_required(VERSION 3.15)
project(FoxgloveWebSocket VERSION 1.4.0 LANGUAGES CXX)

include(CMakePackageConfigHelpers)
include(GNUInstallDirs)
include(FetchContent)

# Fetch asio (header-only, C++11)
FetchContent_Declare(
  asio
  GIT_REPOSITORY https://github.com/chriskohlhoff/asio.git
  GIT_TAG 30a170c0e84a798a1752652439bd93f900062eda
)
FetchContent_MakeAvailable(asio)
add_library(asio_lib INTERFACE)
target_include_directories(asio_lib INTERFACE
  "${asio_SOURCE_DIR}/asio/include")
target_compile_features(asio_lib INTERFACE cxx_std_11)
add_library(asio::asio ALIAS asio_lib)

# Fetch websocketpp (header-only, C++11)
FetchContent_Declare(
  websocketpp
  GIT_REPOSITORY https://github.com/zaphoyd/websocketpp.git
  GIT_TAG develop
)
FetchContent_MakeAvailable(websocketpp)

add_library(websocketpp_lib INTERFACE)
target_include_directories(websocketpp_lib INTERFACE
  "${websocketpp_SOURCE_DIR}"
)
target_compile_features(websocketpp_lib INTERFACE cxx_std_11)
add_library(websocketpp::websocketpp ALIAS websocketpp_lib)
target_link_libraries(websocketpp_lib INTERFACE asio::asio)

# Find JSON dependency
find_package(nlohmann_json REQUIRED)

# Main library (C++17, privately depends on asio_lib and websocketpp_lib)
add_library(foxglove_websocket
  src/base64.cpp
  src/parameter.cpp
  src/serialization.cpp
  src/server_factory.cpp
)
set_target_properties(foxglove_websocket PROPERTIES
  CXX_STANDARD 17
  CXX_STANDARD_REQUIRED ON
)
target_include_directories(foxglove_websocket
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<BUILD_INTERFACE:${websocketpp_SOURCE_DIR}>
    $<BUILD_INTERFACE:${asio_SOURCE_DIR}/asio/include>
    $<INSTALL_INTERFACE:include>
)
target_link_libraries(foxglove_websocket
  PRIVATE
    websocketpp::websocketpp
    nlohmann_json::nlohmann_json
)

install(TARGETS foxglove_websocket
  EXPORT foxglove_websocket-targets
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)
install(DIRECTORY ${websocketpp_SOURCE_DIR}/websocketpp
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)
install(DIRECTORY ${asio_SOURCE_DIR}/asio/include/
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

install(EXPORT foxglove_websocket-targets
  FILE foxglove_websocket-targets.cmake
  NAMESPACE foxglove::
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/foxglove_websocket
)

configure_package_config_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/foxglove_websocket-config.cmake.in"
  "${CMAKE_CURRENT_BINARY_DIR}/foxglove_websocket-config.cmake"
  INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/foxglove_websocket
)

write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/foxglove_websocket-config-version.cmake"
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY SameMajorVersion
)

install(
  FILES
    "${CMAKE_CURRENT_BINARY_DIR}/foxglove_websocket-config.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/foxglove_websocket-config-version.cmake"
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/foxglove_websocket
)
