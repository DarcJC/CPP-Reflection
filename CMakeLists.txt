
add_subdirectory(Source/Runtime MetaRuntime)
add_subdirectory(Source/Parser MetaParser)

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/Source/CMake")

include(MetaParser)

include_directories(${META_RUNTIME_INCLUDE_DIRS})

set_property(GLOBAL PROPERTY USE_FOLDERS ON)
set_property(GLOBAL PROPERTY PREDEFINED_TARGETS_FOLDER "CMake")

set(REFLECTION_ENABLED_TARGETS Rendering)

function (generate_reflection)
    foreach (target IN ITEMS ${REFLECTION_ENABLED_TARGETS})
        message(STATUS "Generating ${target}'s reflection information...")
        get_target_property(TEMP_SOURCE_ROOT ${target} SOURCE_ROOT)
        get_target_property(TEMP_HEADER_FILES ${target} HEADER_FILES)
        get_target_property(TEMP_MODULE_HEADER ${target} MODULE_HEADER)
        get_target_property(TEMP_REFLECT_HEADER ${target} REFLECT_HEADER)
        set(GENERATED_DIR "${CMAKE_CURRENT_BINARY_DIR}/Generated")
        meta_parser_prebuild(
            TARGET ${target}
            GENERATED_DIR ${GENERATED_DIR}
            SOURCE_ROOT ${TEMP_SOURCE_ROOT}
            HEADER_FILES ${TEMP_HEADER_FILES}
            MODULE_HEADER ${TEMP_MODULE_HEADER}
            OUT_MODULE_SOURCE META_MODULE_SOURCE
            OUT_GENERATED_FILES META_GENERATED_FILES
            OUT_INC TEMP_META_GENERATED_HEADERS
            OUT_SRC TEMP_META_GENERATED_SOURCES
        )
        target_sources(${target} PUBLIC ${META_GENERATED_FILES})
        target_link_libraries(${target} PUBLIC MetaRuntime)
        meta_parser_build(
            TARGET ${target}
            SOURCE_ROOT ${TEMP_SOURCE_ROOT}
            GENERATED_DIR ${GENERATED_DIR}
            GENERATED_FILES ${META_GENERATED_FILES}
            SOURCE_FILE ${TEMP_REFLECT_HEADER}
            MODULE_HEADER ${TEMP_MODULE_HEADER}
            MODULE_SOURCE_FILE ${META_MODULE_SOURCE}
            HEADER_FILES ${TEMP_HEADER_FILES}
            PARSER_EXECUTABLE "$<TARGET_FILE:MetaParser>"
        )
    endforeach ()
endfunction()
