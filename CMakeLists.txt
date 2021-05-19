cmake_minimum_required (VERSION 3.12)

project (libpng C)

option (LIBPNG_BUILD_STATIC "Build the static version of libpng" ON)
option (LIBPNG_BUILD_SHARED "Build the shared library version of libpng" ON)

set (SRCS "")

macro (add_sources)
    file (RELATIVE_PATH _relPath "${PROJECT_SOURCE_DIR}" "${CMAKE_CURRENT_SOURCE_DIR}")
    foreach (_src ${ARGN})
        if (_relPath)
            list (APPEND SRCS "${_relPath}/${_src}")
        else()
            list (APPEND SRCS "${_src}")
        endif()
    endforeach()
endmacro()

# checkout dependencies that we need
include(ExternalProject)

set(ZLIB_PREFIX "${CMAKE_CURRENT_BINARY_DIR}/zlib")
set(ZLIB_INSTALL_DIR "${CMAKE_CURRENT_BINARY_DIR}/zlib")
set(ZLIB_CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${ZLIB_INSTALL_DIR} -DCMAKE_TOOLCHAIN_FILE=${CMAKE_TOOLCHAIN_FILE})

FetchContent_Declare(
  zlib
  GIT_REPOSITORY https://github.com/umbrella-c/zlib.git
  GIT_TAG master
)
FetchContent_MakeAvailable(zlib)

add_sources(
    src/png.c
    src/pngerror.c
    src/pngget.c
    src/pngmem.c
    src/pngpread.c
    src/pngread.c
    src/pngrio.c
    src/pngrtran.c
    src/pngrutil.c
    src/pngset.c
    src/pngtrans.c
    src/pngwio.c
    src/pngwrite.c
    src/pngwtran.c
    src/pngwutil.c
)

include_directories (
    ${ZLIB_INSTALL_DIR}/include
    include
)

if (LIBPNG_BUILD_STATIC)
    add_library(png_static STATIC ${SRCS})

    install(TARGETS png_static
        ARCHIVE DESTINATION lib
        LIBRARY DESTINATION lib
    )
endif ()

if (LIBPNG_BUILD_SHARED)
    set (SHARED_SRCS ${SRCS})
    if (MOLLENOS)
        set (SHARED_SRCS ${SHARED_SRCS} main.c)
    endif ()

    if (MSVC OR MOLLENOS)
        set (CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} /def:libpng.def")
    endif()

    add_library(png SHARED ${SHARED_SRCS})
    add_dependencies(png zlib_dependency)
    target_link_libraries(png zlib)

    if (WIN32 OR MOLLENOS)
        target_compile_definitions(png PUBLIC -DPNG_BUILD_DLL)
    endif ()

    install(TARGETS png
        ARCHIVE DESTINATION lib
        LIBRARY DESTINATION lib
        RUNTIME DESTINATION bin
    )
endif ()

install(FILES include/png.h include/pngconf.h include/pnglibconf.h DESTINATION include)
