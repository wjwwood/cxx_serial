cmake_minimum_required(VERSION 3.1)
project(serial)

if(APPLE)
    find_library(IOKIT_LIBRARY IOKit)
    find_library(FOUNDATION_LIBRARY Foundation)
endif()

## Sources
set(serial_SRCS
    src/serial.cc
)
set(serial_HDRS
    ${CMAKE_CURRENT_LIST_DIR}/include/serial/serial.h
    ${CMAKE_CURRENT_LIST_DIR}/include/serial/v8stdint.h
)
if(APPLE)
    # If OSX
    list(APPEND serial_SRCS src/impl/unix.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_osx.cc)
elseif(UNIX)
    # If unix
    list(APPEND serial_SRCS src/impl/unix.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_linux.cc)
else()
    # If windows
    list(APPEND serial_SRCS src/impl/win.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_win.cc)
endif()

## Add serial library
add_library(${PROJECT_NAME})
target_sources(${PROJECT_NAME}
    PRIVATE ${serial_SRCS}
    PUBLIC ${serial_HDRS}
)
if(APPLE)
    target_link_libraries(${PROJECT_NAME} ${FOUNDATION_LIBRARY} ${IOKIT_LIBRARY})
elseif(UNIX)
    target_link_libraries(${PROJECT_NAME} rt pthread)
else()
    target_link_libraries(${PROJECT_NAME} setupapi)
endif()
# Make the headers available to any targets linking the library (e.g., serial_example, test, via FetchContent).
target_include_directories(${PROJECT_NAME} PUBLIC ${CMAKE_CURRENT_LIST_DIR}/include)

## Uncomment for example
add_executable(serial_example examples/serial_example.cc)
add_dependencies(serial_example ${PROJECT_NAME})
target_link_libraries(serial_example ${PROJECT_NAME})

## Include headers
include_directories(include)

## Install executable
install(TARGETS ${PROJECT_NAME}
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
    RUNTIME DESTINATION bin
)

## Install headers
install(FILES ${serial_HDRS}
    DESTINATION include/serial)

## Install CMake config
install(FILES cmake/serialConfig.cmake
    DESTINATION share/serial/cmake)


## Install package.xml
install(FILES package.xml
    DESTINATION share/serial)

## Tests
include(CTest)
if(BUILD_TESTING)
    add_subdirectory(tests)
endif()
