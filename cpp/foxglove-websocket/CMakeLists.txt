cmake_minimum_required(VERSION 3.15)
project(FoxgloveWebSocket CXX)

find_package(nlohmann_json REQUIRED)
find_package(websocketpp REQUIRED)

add_library(foxglove_websocket src/base64.cpp src/parameter.cpp src/serialization.cpp src/server_factory.cpp)
# target_include_directories(foxglove_websocket
#     PUBLIC include
#     SYSTEM
#     ${nlohmann_json_INCLUDE_DIRS}
#     ${websocketpp_INCLUDE_DIRS}
# )

target_include_directories(foxglove_websocket
  PUBLIC
    $<INSTALL_INTERFACE:include/foxglove>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
)

target_link_libraries(foxglove_websocket PRIVATE nlohmann_json::nlohmann_json websocketpp::websocketpp)
set_target_properties(foxglove_websocket PROPERTIES CXX_STANDARD 17 CXX_STANDARD_REQUIRED ON)

if (MSVC)
  target_compile_options(foxglove_websocket PRIVATE /W4)
else()
  target_compile_options(foxglove_websocket PRIVATE -Wall -Wextra -Wpedantic -Wold-style-cast -Wfloat-equal)
endif()

include(GNUInstallDirs)
install(TARGETS foxglove_websocket 
  EXPORT foxglove_websocketTargets
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
  )

install(EXPORT foxglove_websocketTargets
  FILE foxglove_websocketTargets.cmake
  NAMESPACE foxglove_websocket::
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/foxglove_websocket
)

# handles the inclusion of the find_dependency and the checking of the targets 
include(CMakePackageConfigHelpers)
# within this template file that gets used to help create the find_package config file 
# the targets get included and the above deps also get included to ensure that when find_package
# is called all deps are findable
configure_package_config_file(cmake/foxglove_websocketConfig.cmake.in
  "${CMAKE_CURRENT_BINARY_DIR}/foxglove_websocketConfig.cmake"
  INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/foxglove_websocket
)

install(FILES
  "${CMAKE_CURRENT_BINARY_DIR}/foxglove_websocketConfig.cmake"
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/foxglove_websocket
)

install(DIRECTORY include/
        DESTINATION ${CMAKE_INSTALL_PREFIX}/include
        FILES_MATCHING PATTERN "*.hpp")

install(FILES ${CMAKE_SOURCE_DIR}/LICENSE DESTINATION ${CMAKE_INSTALL_PREFIX}/licenses/)
