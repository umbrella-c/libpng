cmake_minimum_required (VERSION 3.12)

project (libpng C)

option (LIBPNG_BUILD_STATIC "Build the static version of libpng" ON)
option (LIBPNG_BUILD_SHARED "Build the shared library version of libpng" ON)

find_package(ZLIB REQUIRED)

set (SRCS 
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

if (LIBPNG_BUILD_STATIC)
    add_library(png_static STATIC ${SRCS})
    target_include_directories(png_static PRIVATE ${ZLIB_INCLUDE_DIRS})
    target_include_directories(png_static PUBLIC include)

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
    target_include_directories(png PRIVATE ${ZLIB_INCLUDE_DIRS})
    target_include_directories(png PUBLIC include)
    target_link_libraries(png ${ZLIB_LIBRARIES})

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
