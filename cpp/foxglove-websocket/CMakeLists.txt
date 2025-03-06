cmake_minimum_required(VERSION 3.15)
project(FoxgloveWebSocket VERSION 1.4.0 LANGUAGES CXX)

include(CMakePackageConfigHelpers)
include(GNUInstallDirs)

include(FetchContent)
FetchContent_Declare(
  asio
  GIT_REPOSITORY "https://github.com/chriskohlhoff/asio.git"
  GIT_TAG "30a170c0e84a798a1752652439bd93f900062eda"
)
FetchContent_GetProperties(asio)
if(NOT asio_POPULATED)
  FetchContent_Populate(asio)
endif()
message(${asio_SOURCE_DIR})

find_package(nlohmann_json REQUIRED)
find_package(websocketpp REQUIRED)

add_library(foxglove_websocket
  src/base64.cpp
  src/parameter.cpp
  src/serialization.cpp
  src/server_factory.cpp
)

target_include_directories(foxglove_websocket
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
        $<BUILD_INTERFACE:${asio_SOURCE_DIR}/asio/include/>
    PRIVATE
        ${nlohmann_json_INCLUDE_DIRS}
        ${websocketpp_INCLUDE_DIRS}
)

target_link_libraries(foxglove_websocket
    PRIVATE
        nlohmann_json::nlohmann_json
        websocketpp::websocketpp
)

set_target_properties(foxglove_websocket PROPERTIES CXX_STANDARD 20 CXX_STANDARD_REQUIRED ON)

if(MSVC)
  target_compile_options(foxglove_websocket PRIVATE /W4)
else()
  target_compile_options(foxglove_websocket PRIVATE -Wall -Wextra -Wpedantic -Wold-style-cast -Wfloat-equal)
endif()

add_library(foxglove::foxglove_websocket ALIAS foxglove_websocket)

install(TARGETS foxglove_websocket
        EXPORT foxglove_websocket-targets
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

install(DIRECTORY ${asio_SOURCE_DIR}/asio/include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
install(FILES ${CMAKE_SOURCE_DIR}/LICENSE DESTINATION ${CMAKE_INSTALL_PREFIX}/licenses/)

install(EXPORT foxglove_websocket-targets
        FILE foxglove_websocket-targets.cmake
        NAMESPACE foxglove::
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/foxglove_websocket)

configure_package_config_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/foxglove_websocket-config.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/foxglove_websocket-config.cmake
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/foxglove_websocket
)

write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/foxglove_websocket-config-version.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
)

install(FILES 
    ${CMAKE_CURRENT_BINARY_DIR}/foxglove_websocket-config.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/foxglove_websocket-config-version.cmake
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/foxglove_websocket
)

