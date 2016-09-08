ADD_SUBDIRECTORY(external)

SET(PLATFORM_TO_USE "PLATFORM_DESKTOP" CACHE STRING "Platform to compile for")
SET_PROPERTY(CACHE PLATFORM_TO_USE PROPERTY STRINGS PLATFORM_DESKTOP PLATFORM_RPI PLATFORM_WEB)

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

ADD_DEPENDENCIES(raylib glfw3)

INSTALL(TARGETS raylib DESTINATION lib)
INSTALL(FILES raylib.h DESTINATION include)

IF(CMAKE_COMPILER_IS_GNUCC)
    TARGET_COMPILE_OPTIONS(raylib PRIVATE -O1 -Wall -std=gnu99 -fgnu89-inline
        -Wno-missing-braces)
ELSEIF(MSVC)
    # TODO: Set these to something that corresponds to the GCC flags?
ENDIF()

IF(${PLATFORM_TO_USE} MATCHES "PLATFORM_DESKTOP")

     TARGET_COMPILE_DEFINITIONS(raylib PUBLIC -DPLATFORM_DESKTOP -DGRAPHICS_API_OPENGL_33)
     TARGET_INCLUDE_DIRECTORIES(raylib PUBLIC
        ${CMAKE_CURRENT_SOURCE_DIR}
        ${CMAKE_CURRENT_SOURCE_DIR}/external/openal_soft/include
        ${CMAKE_INSTALL_PREFIX}/include)

ENDIF()

IF(${PLATFORM_TO_USE} MATCHES "PLATFORM_RPI")

     TARGET_COMPILE_DEFINITIONS(raylib PUBLIC -DPLATFORM_RPI -GRAPHICS_API_OPENGL_ES2)
     TARGET_INCLUDE_DIRECTORIES(
        ${CMAKE_CURRENT_SOURCE_DIR}
        ${CMAKE_CURRENT_SOURCE_DIR}/external
        "/opt/vc/include"
        "/opt/vc/include/interface/vmcs_host/linux"
        "/opt/vc/include/interface/vcos/pthreads")

ENDIF()

IF(${PLATFORM_TO_USE} MATCHES "PLATFORM_WEB")

     TARGET_COMPILE_DEFINITIONS(raylib PUBLIC -DPLATFORM_WEB -GRAPHICS_API_OPENGL_ES2)
     TARGET_INCLUDE_DIRECTORIES(
        ${CMAKE_CURRENT_SOURCE_DIR}
        ${CMAKE_CURRENT_SOURCE_DIR}/external
        ${CMAKE_CURRENT_SOURCE_DIR}/external/openal_soft/include
        ${CMAKE_INSTALL_PREFIX}/include)

ENDIF()
