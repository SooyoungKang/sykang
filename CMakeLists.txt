cmake_minimum_required(VERSION 2.8)
project(WPEBackend)

set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c99 -D_POSIX_SOURCE")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

find_package(EGL REQUIRED)

add_definitions(${EGL_DEFINITIONS})

if(WPE_BACKEND)
  add_definitions(-DWPE_BACKEND=\"${WPE_BACKEND}\")
endif()

set(WPE_INCLUDE_DIRECTORIES
    "include"
    "src"
    ${EGL_INCLUDE_DIRS}
)

if (UNIX)
    set(WPE_LIBRARIES
        dl
    )
endif ()

set(WPE_SOURCES
    src/input.c
    src/loader.c
    src/pasteboard.c
    src/pasteboard-generic.cpp
    src/pasteboard-noop.cpp
    src/renderer-backend-egl.c
    src/renderer-host.c
    src/view-backend.c
)

set(WPE_PUBLIC_HEADERS
  include/wpe/input.h
  include/wpe/loader.h
  include/wpe/pasteboard.h
  include/wpe/renderer-backend-egl.h
  include/wpe/renderer-host.h
  include/wpe/view-backend.h
  include/wpe/wpebackend-export-macros.h
)

add_library(WPEBackend SHARED ${WPE_SOURCES})
target_include_directories(WPEBackend PRIVATE ${WPE_INCLUDE_DIRECTORIES})
target_link_libraries(WPEBackend ${WPE_LIBRARIES})

set(WPE_VERSION_MAJOR 0)
set(WPE_VERSION_MINOR 1)
set(WPE_VERSION ${WPE_VERSION_MAJOR}.${WPE_VERSION_MINOR})

set_target_properties(WPEBackend PROPERTIES VERSION ${WPE_VERSION} SOVERSION ${WPE_VERSION_MAJOR})

install(TARGETS WPEBackend
  EXPORT WPETargets
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  INCLUDES DESTINATION include
)
install(
  FILES
    ${WPE_PUBLIC_HEADERS}
    DESTINATION
    include/wpe
)

configure_file(wpe.pc.cmake wpe.pc @ONLY)
install(
  FILES
    "${CMAKE_CURRENT_BINARY_DIR}/wpe.pc"
    DESTINATION
    lib/pkgconfig
)
