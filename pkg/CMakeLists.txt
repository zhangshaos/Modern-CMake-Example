cmake_minimum_required(VERSION 3.16)
project(hello VERSION 0.0.1)

set(CMAKE_CXX_STANDARD 17)
add_compile_options("$<$<C_COMPILER_ID:MSVC>:/utf-8>")
add_compile_options("$<$<CXX_COMPILER_ID:MSVC>:/utf-8>")

if(MSVC)
    set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS TRUE)
endif()

find_package(fmt CONFIG REQUIRED)

add_library(${PROJECT_NAME} SHARED include/library.h library.cpp)

target_link_libraries(${PROJECT_NAME} PUBLIC fmt::fmt)

include(GNUInstallDirs)

target_include_directories(${PROJECT_NAME} PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/include>)
# Add target property about target's include dir after install.
# See GNUInstallDirs, CMAKE_INSTALL_INCLUDEDIR == include (Relative to CMAKE_INSTALL_PREFIX)

# (1) Install (Following relative paths are relative to CMAKE_INSTALL_PREFIX)

install(TARGETS ${PROJECT_NAME} EXPORT ${PROJECT_NAME}_Export
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

install(EXPORT ${PROJECT_NAME}_Export
        DESTINATION cmake
        NAMESPACE hello::)

install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
        FILES_MATCHING
        PATTERN *.h
        PATTERN *.inl
        PATTERN *.hpp)

# (2) Config

include(CMakePackageConfigHelpers)

set(${PROJECT_NAME}_CONFIG_DIR cmake) # Config those variable for fun.
set(${PROJECT_NAME}_INCLUDE_DIR include)
set(${PROJECT_NAME}_LIBRARIES lib)
set(${PROJECT_NAME}_BINS bin)

configure_package_config_file(
        "${CMAKE_CURRENT_SOURCE_DIR}/Config.cmake.in"
        "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
        INSTALL_DESTINATION cmake  # Same as install(EXPORT), relative path.
        PATH_VARS
        ${PROJECT_NAME}_CONFIG_DIR ${PROJECT_NAME}_INCLUDE_DIR ${PROJECT_NAME}_LIBRARIES ${PROJECT_NAME}_BINS)  # Config those variable for fun.

write_basic_package_version_file(
        ${PROJECT_NAME}ConfigVersion.cmake
        VERSION ${PROJECT_VERSION}
        COMPATIBILITY SameMajorVersion)

install(FILES
        ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
        ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
        DESTINATION cmake)  # Same as install(EXPORT), relative path.
