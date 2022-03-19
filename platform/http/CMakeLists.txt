if(TARGET mbgl-http)
    return()
endif()

add_library(
        mbgl-http STATIC
        ${PROJECT_SOURCE_DIR}/platform/http/http_file_source.hpp
        ${PROJECT_SOURCE_DIR}/include/mbgl/storage/file_source.hpp
        ${PROJECT_SOURCE_DIR}/include/mbgl/storage/resource.hpp
        ${PROJECT_SOURCE_DIR}/include/mbgl/storage/response.hpp
        ${PROJECT_SOURCE_DIR}/include/mbgl/util/chrono.hpp
        ${PROJECT_SOURCE_DIR}/include/mbgl/platform/time.hpp
        ${PROJECT_SOURCE_DIR}/include/mbgl/util/optional.hpp
        ${PROJECT_SOURCE_DIR}/vendor/mapbox-base/deps/optional/optional.hpp
        ${PROJECT_SOURCE_DIR}/include/mbgl/util/bitmask_operations.hpp
        ${PROJECT_SOURCE_DIR}/include/mbgl/util/traits.hpp
        ${PROJECT_SOURCE_DIR}/include/mbgl/util/util.hpp
        ${PROJECT_SOURCE_DIR}/include/mbgl/util/font_stack.hpp
        ${PROJECT_SOURCE_DIR}/include/mbgl/util/immutable.hpp
        ${PROJECT_SOURCE_DIR}/include/mbgl/style/layer.hpp
        ${PROJECT_SOURCE_DIR}/include/mbgl/style/conversion.hpp
        ${PROJECT_SOURCE_DIR}/include/mbgl/style/style_property.hpp
        ${PROJECT_SOURCE_DIR}/include/mbgl/util/feature.hpp
        ${PROJECT_SOURCE_DIR}/include/mbgl/style/types.hpp
        ${PROJECT_SOURCE_DIR}/include/mbgl/util/tileset.hpp
        ${PROJECT_SOURCE_DIR}/include/mbgl/util/range.hpp
        ${PROJECT_SOURCE_DIR}/include/mbgl/util/constants.hpp
        ${PROJECT_SOURCE_DIR}/include/mbgl/util/unitbezier.hpp
        ${PROJECT_SOURCE_DIR}/include/mbgl/util/geo.hpp
)


target_include_directories(
        mbgl-http
        PUBLIC
        ${PROJECT_SOURCE_DIR}/platform/http
        PRIVATE
        ${PROJECT_SOURCE_DIR}/platform/default
        ${PROJECT_SOURCE_DIR}/vendor/mapbox-base/include
        ${PROJECT_SOURCE_DIR}/vendor/mapbox-base/include/mapbox/compatibility
        ${PROJECT_SOURCE_DIR}/vendor/mapbox-base/include/mapbox/util
        ${PROJECT_SOURCE_DIR}/vendor/mapbox-base/include/mapbox/std
        ${PROJECT_SOURCE_DIR}/vendor/mapbox-base/deps/geometry.hpp/include
        ${PROJECT_SOURCE_DIR}/vendor/mapbox-base/deps/variant/include
        ${PROJECT_SOURCE_DIR}/vendor/mapbox-base/deps/optional
        ${PROJECT_SOURCE_DIR}/include/mbgl/util
)

macro(setup_apple_platform)
    set(CMAKE_OSX_DEPLOYMENT_TARGET "10.11")
    set(
            mbgl_HTTP_LIB_PLATFORM_BACKEND_SOURCES
            $<$<BOOL:${MBGL_PUBLIC_BUILD}>:${PROJECT_SOURCE_DIR}/platform/darwin/src/http_file_source.mm>
    )
    include(${PROJECT_SOURCE_DIR}/vendor/icu.cmake)
    target_link_libraries(
            mbgl-http
            PRIVATE
            mbgl-vendor-icu
            "-framework Foundation"
    )
    target_include_directories(
            mbgl-http
            PUBLIC
            ${PROJECT_SOURCE_DIR}/platform/darwin/include
            ${PROJECT_SOURCE_DIR}/include/
            ${PROJECT_SOURCE_DIR}/src
            ${PROJECT_SOURCE_DIR}/platform/darwin/include
            ${PROJECT_SOURCE_DIR}/platform/darwin/src
            ${PROJECT_SOURCE_DIR}/platform/macos/src

    )
    set_target_properties(mbgl-http PROPERTIES XCODE_ATTRIBUTE_IPHONEOS_DEPLOYMENT_TARGET "${IOS_DEPLOYMENT_TARGET}")
    set_target_properties(mbgl-http PROPERTIES XCODE_ATTRIBUTE_ENABLE_BITCODE "YES")
    set_target_properties(mbgl-http PROPERTIES XCODE_ATTRIBUTE_BITCODE_GENERATION_MODE bitcode)
    set_target_properties(mbgl-http PROPERTIES XCODE_ATTRIBUTE_ONLY_ACTIVE_ARCH $<$<CONFIG:Debug>:YES>)
endmacro()

if (MBGL_WITH_QT)
    file(GLOB mbgl_HTTP_LIB_PLATFORM_BACKEND_SOURCES
            $<$<BOOL:${MBGL_PUBLIC_BUILD}>:${PROJECT_SOURCE_DIR}/platform/qt/src/http_file_source.cpp>
            $<$<BOOL:${MBGL_PUBLIC_BUILD}>:${PROJECT_SOURCE_DIR}/platform/qt/src/http_file_source.hpp>
            $<$<BOOL:${MBGL_PUBLIC_BUILD}>:${PROJECT_SOURCE_DIR}/platform/qt/src/http_request.cpp>
            $<$<BOOL:${MBGL_PUBLIC_BUILD}>:${PROJECT_SOURCE_DIR}/platform/qt/src/http_request.hpp>
            )
elseif (CMAKE_SYSTEM_NAME STREQUAL Android)
    file(GLOB mbgl_HTTP_LIB_PLATFORM_BACKEND_SOURCES
            $<$<BOOL:${MBGL_PUBLIC_BUILD}>:${PROJECT_SOURCE_DIR}/platform/default/src/mbgl/storage/http_file_source.cpp>
            )
elseif (CMAKE_SYSTEM_NAME STREQUAL iOS)
    setup_apple_platform()
elseif (CMAKE_SYSTEM_NAME STREQUAL Linux)
    find_package(CURL REQUIRED)
    file(GLOB mbgl_HTTP_LIB_PLATFORM_BACKEND_SOURCES
            $<$<BOOL:${MBGL_PUBLIC_BUILD}>:${PROJECT_SOURCE_DIR}/platform/default/src/mbgl/storage/http_file_source.cpp>
            )

    target_include_directories(
            mbgl-http
            PRIVATE
            ${PROJECT_SOURCE_DIR}/platform/default/include
            ${CURL_INCLUDE_DIRS}
    )
    target_link_libraries(
            mbgl-http
            PRIVATE
            ${CURL_LIBRARIES}
    )
    target_compile_options(mbgl-http PUBLIC -std=c++11 -fpermissive -w -Wall)
    set_target_properties(mbgl-http PROPERTIES LINKER_LANGUAGE CXX)
elseif (CMAKE_SYSTEM_NAME STREQUAL Darwin)
    setup_apple_platform()
else ()
    message(FATAL_ERROR "Unsupported target platform: " ${CMAKE_SYSTEM_NAME})
endif ()

target_sources(
        mbgl-http
        PUBLIC
        ${PROJECT_SOURCE_DIR}/platform/http/http_file_source.hpp
        PRIVATE
        ${mbgl_HTTP_LIB_PLATFORM_BACKEND_SOURCES}
)

if (CMAKE_SYSTEM_NAME STREQUAL Darwin OR CMAKE_SYSTEM_NAME STREQUAL iOS)
    set_target_properties(
            mbgl-http
            PROPERTIES XCODE_ATTRIBUTE_CLANG_ENABLE_OBJC_ARC YES
    )
    target_compile_options(mbgl-http PUBLIC -fobjc-arc)
endif ()

target_link_libraries(
        mbgl-http
        PRIVATE
        mbgl-compiler-options
)

set_target_properties(
        mbgl-http
        PROPERTIES
        INTERFACE_MAPBOX_NAME "Mapbox GL Http"
        INTERFACE_MAPBOX_URL "https://github.com/mapbox/mapbox-gl-native"
        INTERFACE_MAPBOX_AUTHOR "Mapbox candidate Kair"
        INTERFACE_MAPBOX_LICENSE ${PROJECT_SOURCE_DIR}/LICENSE.md
        ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}
)