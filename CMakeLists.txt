cmake_minimum_required(VERSION 3.10)
project(heatshrink C)

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O3 -g -Wall -Wextra -pedantic -Wmissing-prototypes -Wstrict-prototypes -Wmissing-declarations")

# Optional: libtheft integration
# set(THEFT_PATH "/usr/local")
# include_directories(${THEFT_PATH}/include)
# link_directories(${THEFT_PATH}/lib)
# add_definitions(-DHEATSHRINK_HAS_THEFT)

# Source files
set(SOURCES
    src/heatshrink_encoder.c
    src/heatshrink_decoder.c
)
set(HEADERS
    heatshrink_config.h
    inc/heatshrink_common.h
    inc/heatshrink_encoder.h
    inc/heatshrink_decoder.h)


# Static and dynamic versions
add_library(heatshrink_static STATIC ${SOURCES})
target_compile_definitions(heatshrink_static 
    PRIVATE 
    HEATSHRINK_DYNAMIC_ALLOC=0)

add_library(heatshrink_dynamic STATIC ${SOURCES})
target_compile_definitions(heatshrink_dynamic 
    PRIVATE 
    HEATSHRINK_DYNAMIC_ALLOC=1)

include_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}/inc
    ${CMAKE_CURRENT_SOURCE_DIR}/
)
# Executables
add_executable(heatshrink heatshrink.c)
# target_include_directories(heatshrink PRIVATE 
#     ${CMAKE_CURRENT_SOURCE_DIR}/inc
#     ${CMAKE_CURRENT_SOURCE_DIR}/
# )
target_link_libraries(heatshrink heatshrink_dynamic)
target_compile_definitions(heatshrink PRIVATE HEATSHRINK_DYNAMIC_ALLOC=1)

# add_executable(test_heatshrink_static test/test_heatshrink_static.c)
# target_link_libraries(test_heatshrink_static heatshrink_static)
# target_compile_definitions(test_heatshrink_static PRIVATE HEATSHRINK_DYNAMIC_ALLOC=0)

# add_executable(test_heatshrink_dynamic
#     test/test_heatshrink_dynamic.c
#     test/test_heatshrink_dynamic_theft.c
#     )
# target_link_libraries(test_heatshrink_dynamic heatshrink_dynamic)
# target_compile_definitions(test_heatshrink_dynamic PRIVATE HEATSHRINK_DYNAMIC_ALLOC=1)

# # Custom target for running tests
# add_custom_target(test
#     COMMAND test_heatshrink_static
#     COMMAND test_heatshrink_dynamic
#     DEPENDS test_heatshrink_static test_heatshrink_dynamic
# )

# Diagrams (requires Graphviz)
find_program(DOT_EXECUTABLE dot)
if(DOT_EXECUTABLE)
    add_custom_command(
        OUTPUT dec_sm.png
        COMMAND ${DOT_EXECUTABLE} -o dec_sm.png -Tpng ${CMAKE_SOURCE_DIR}/docs/dec_sm.dot
        DEPENDS ${CMAKE_SOURCE_DIR}/docs/dec_sm.dot
    )
    add_custom_command(
        OUTPUT enc_sm.png
        COMMAND ${DOT_EXECUTABLE} -o enc_sm.png -Tpng ${CMAKE_SOURCE_DIR}/docs/enc_sm.dot
        DEPENDS ${CMAKE_SOURCE_DIR}/docs/enc_sm.dot
    )
    add_custom_target(diagrams ALL DEPENDS dec_sm.png enc_sm.png)
endif()

# Installation
install(TARGETS heatshrink heatshrink_static heatshrink_dynamic DESTINATION bin)
install(FILES ${HEADERS} DESTINATION include)

# Uninstall support
configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/cmake_uninstall.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
    IMMEDIATE @ONLY
)
add_custom_target(uninstall
    COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake
)

# Clean extras
set_directory_properties(PROPERTIES ADDITIONAL_MAKE_CLEAN_FILES "*.core;*.png;benchmark_out")

