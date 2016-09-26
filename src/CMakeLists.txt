IF(BOOTSTRAP)
    ADD_SUBDIRECTORY(external)
    RETURN()
ENDIF()

SET(PLATFORM_TO_USE "PLATFORM_DESKTOP" CACHE STRING "Platform to compile for")
SET_PROPERTY(CACHE PLATFORM_TO_USE PROPERTY STRINGS PLATFORM_DESKTOP PLATFORM_RPI PLATFORM_WEB)

SET(CMAKE_INCLUDE_CURRENT_DIR ON)

SET(SOURCES
    audio.c
    camera.c
    core.c
    external/stb_vorbis.c
    external/tinfl.c
    gestures.c
    models.c
    rlgl.c
    shapes.c
    text.c
    textures.c
    utils.c
)

ADD_LIBRARY(raylib STATIC ${SOURCES})

FIND_PACKAGE(glfw3 REQUIRED)
FIND_LIBRARY(openal NAMES openal OpenAL32)

TARGET_LINK_LIBRARIES(raylib PUBLIC glfw ${openal})

INSTALL(TARGETS raylib EXPORT raylibTargets
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    RUNTIME DESTINATION bin
    INCLUDES DESTINATION include
)

INSTALL(FILES raylib.h raymath.h DESTINATION include/raylib)

IF(CMAKE_COMPILER_IS_GNUCC)
    TARGET_COMPILE_OPTIONS(raylib PRIVATE -O1 -Wall -std=gnu99 -fgnu89-inline
        -Wno-missing-braces)
ELSEIF(MSVC)
    # TODO: Set these to something that corresponds to the GCC flags?
ENDIF()

IF(${PLATFORM_TO_USE} MATCHES "PLATFORM_DESKTOP")

     TARGET_COMPILE_DEFINITIONS(raylib PUBLIC -DPLATFORM_DESKTOP -DGRAPHICS_API_OPENGL_33)
     TARGET_INCLUDE_DIRECTORIES(raylib PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/external/openal_soft/include
        ${CMAKE_INSTALL_PREFIX}/include)

     IF(CMAKE_SYSTEM_NAME STREQUAL "Linux")
         TARGET_LINK_LIBRARIES(raylib PUBLIC m dl pthread X11 Xrandr Xinerama
            Xi Xxf86vm Xcursor)
     ELSEIF(CMAKE_SYSTEM_NAME STREQUAL "Windows")
         TARGET_LINK_LIBRARIES(raylib PUBLIC gdi32)
     ENDIF()

ENDIF()

IF(${PLATFORM_TO_USE} MATCHES "PLATFORM_RPI")

     TARGET_COMPILE_DEFINITIONS(raylib PUBLIC -DPLATFORM_RPI -GRAPHICS_API_OPENGL_ES2)
     TARGET_INCLUDE_DIRECTORIES(raylib PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/external
        "/opt/vc/include"
        "/opt/vc/include/interface/vmcs_host/linux"
        "/opt/vc/include/interface/vcos/pthreads")

ENDIF()

IF(${PLATFORM_TO_USE} MATCHES "PLATFORM_WEB")

     TARGET_COMPILE_DEFINITIONS(raylib PUBLIC -DPLATFORM_WEB -GRAPHICS_API_OPENGL_ES2)
     TARGET_INCLUDE_DIRECTORIES(raylib PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/external
        ${CMAKE_CURRENT_SOURCE_DIR}/external/openal_soft/include
        ${CMAKE_INSTALL_PREFIX}/include)

ENDIF()

INCLUDE(CMakePackageConfigHelpers)
CONFIGURE_PACKAGE_CONFIG_FILE(raylibConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/raylibConfig.cmake
    INSTALL_DESTINATION lib/cmake/raylib
)

INSTALL(EXPORT raylibTargets
    DESTINATION lib/cmake/raylib
)

INSTALL(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/raylibConfig.cmake
    DESTINATION lib/cmake/raylib
)
