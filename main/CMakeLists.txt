cmake_minimum_required(VERSION 3.16)
project(main VERSION 0.0.1)

set(hello_ROOT "E:\\CMAKE_INSTALL_PREFIX\\cmk_pkg")

set(CMAKE_CXX_STANDARD 17)
add_compile_options("$<$<C_COMPILER_ID:MSVC>:/utf-8>")
add_compile_options("$<$<CXX_COMPILER_ID:MSVC>:/utf-8>")

find_package(hello CONFIG REQUIRED)

add_executable(${PROJECT_NAME} main.cpp)
target_link_libraries(${PROJECT_NAME} PUBLIC hello::hello)

# Install the DLLs

function(get_dep_directories target dirs)
    get_target_property(link_libs ${target} INTERFACE_LINK_LIBRARIES)
    if(${link_libs} MATCHES "NOTFOUND$")
        return()
    endif()
    set(link_dirs)
    foreach(lib IN LISTS link_libs)
        if(NOT ${lib} MATCHES "NOTFOUND$")
            # Found {lib}'s location.
            get_target_property(loc ${lib} LOCATION)
            get_filename_component(loc ${loc} DIRECTORY)
            list(APPEND link_dirs ${loc})
            # Found {lib}'s dependents' location.
            get_dep_directories(${lib} sub_dirs)
            list(APPEND link_dirs ${sub_dirs})
        endif()
    endforeach()
    set(${dirs} ${link_dirs} PARENT_SCOPE)
endfunction()

get_dep_directories(${PROJECT_NAME} link_dirs)
message(STATUS "Target ${PROJECT_NAME}'s linked dll dirs are ${link_dirs}.")
message(STATUS "Using `cmake install command` to install those dll.")

install(CODE "set(DLLs_PATH ${link_dirs})")
install(CODE [[

    message(STATUS "DLLs PATH is ${DLLs_PATH}.")
    file(GET_RUNTIME_DEPENDENCIES
        RESOLVED_DEPENDENCIES_VAR deps
        UNRESOLVED_DEPENDENCIES_VAR failed_deps
        EXECUTABLES $<TARGET_FILE:main>
        DIRECTORIES ${DLLs_PATH}
        POST_EXCLUDE_REGEXES "C:[/\\]WINDOWS")
    message(STATUS "deps are ${deps}.")
    message(STATUS "failed deps are ${failed_deps}.")
    file(COPY ${deps} DESTINATION $<TARGET_FILE_DIR:main>)

]])

