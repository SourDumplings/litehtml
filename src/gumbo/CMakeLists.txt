cmake_minimum_required(VERSION 3.5)

project(gumbo C)

# Soname
# MAJOR is incremented when symbols are removed or changed in an incompatible way
# MINOR is incremented when new symbols are added
set(PROJECT_MAJOR 0)
set(PROJECT_MINOR 0)

set(SOURCE_GUMBO
    ${PROJECT_SOURCE_DIR}/attribute.c
    ${PROJECT_SOURCE_DIR}/char_ref.c
    ${PROJECT_SOURCE_DIR}/error.c
    ${PROJECT_SOURCE_DIR}/parser.c
    ${PROJECT_SOURCE_DIR}/string_buffer.c
    ${PROJECT_SOURCE_DIR}/string_piece.c
    ${PROJECT_SOURCE_DIR}/tag.c
    ${PROJECT_SOURCE_DIR}/tokenizer.c
    ${PROJECT_SOURCE_DIR}/utf8.c
    ${PROJECT_SOURCE_DIR}/util.c
    ${PROJECT_SOURCE_DIR}/vector.c
)

set(HEADER_GUMBO
    ${PROJECT_SOURCE_DIR}/include/gumbo/attribute.h
    ${PROJECT_SOURCE_DIR}/include/gumbo/char_ref.h
    ${PROJECT_SOURCE_DIR}/include/gumbo/error.h
    ${PROJECT_SOURCE_DIR}/include/gumbo.h
    ${PROJECT_SOURCE_DIR}/include/gumbo/insertion_mode.h
    ${PROJECT_SOURCE_DIR}/include/gumbo/parser.h
    ${PROJECT_SOURCE_DIR}/include/gumbo/string_buffer.h
    ${PROJECT_SOURCE_DIR}/include/gumbo/string_piece.h
    ${PROJECT_SOURCE_DIR}/include/gumbo/tag_enum.h
    ${PROJECT_SOURCE_DIR}/include/gumbo/tag_gperf.h
    ${PROJECT_SOURCE_DIR}/include/gumbo/tag_sizes.h
    ${PROJECT_SOURCE_DIR}/include/gumbo/tag_strings.h
    ${PROJECT_SOURCE_DIR}/include/gumbo/token_type.h
    ${PROJECT_SOURCE_DIR}/include/gumbo/tokenizer.h
    ${PROJECT_SOURCE_DIR}/include/gumbo/tokenizer_states.h
    ${PROJECT_SOURCE_DIR}/include/gumbo/utf8.h
    ${PROJECT_SOURCE_DIR}/include/gumbo/util.h
    ${PROJECT_SOURCE_DIR}/include/gumbo/vector.h
)

set(PROJECT_LIB_VERSION ${PROJECT_MAJOR}.${PROJECT_MINOR}.${PROJECT_PATCH})
set(PROJECT_SO_VERSION ${PROJECT_MAJOR})

if (MSVC)
    add_compile_options(/wd4244 /wd4267)
    add_library(${PROJECT_NAME} STATIC ${SOURCE_GUMBO})
else()
    add_library(${PROJECT_NAME} SHARED ${SOURCE_GUMBO})
endif()

set_target_properties(${PROJECT_NAME} PROPERTIES VERSION ${PROJECT_LIB_VERSION} SOVERSION ${PROJECT_SO_VERSION})

set_target_properties(${PROJECT_NAME} PROPERTIES
    C_STANDARD 99
    PUBLIC_HEADER "${HEADER_GUMBO}"
    ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib
    LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib
    RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin
)

if (MSVC)
    target_include_directories(${PROJECT_NAME} PRIVATE visualc/include)
endif()

# Export gumbo includes.
target_include_directories(${PROJECT_NAME} PUBLIC
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>)
target_include_directories(${PROJECT_NAME} PRIVATE include/gumbo)

# install and export
install(TARGETS ${PROJECT_NAME}
    EXPORT gumbo
    RUNTIME DESTINATION bin COMPONENT libraries
    ARCHIVE DESTINATION lib${LIB_SUFFIX} COMPONENT libraries
    LIBRARY DESTINATION lib${LIB_SUFFIX} COMPONENT libraries
    PUBLIC_HEADER DESTINATION include/gumbo
)
install(EXPORT gumbo FILE gumboConfig.cmake DESTINATION lib${LIB_SUFFIX}/cmake/gumbo)

if (MSVC AND ${CMAKE_BUILD_TYPE} MATCHES "Debug")
	# Install .pdb files if using MSVC
	message("Installing pdb file...")
    install(FILES ${CMAKE_BINARY_DIR}/lib/${PROJECT_NAME}.pdb DESTINATION lib${LIB_SUFFIX} CONFIGURATIONS Debug)
endif()