cmake_minimum_required(VERSION 3.10 FATAL_ERROR)

# Set up the Vita SDK toolchain
if(NOT DEFINED CMAKE_TOOLCHAIN_FILE)
  if(DEFINED ENV{VITASDK})
    set(VITASDK $ENV{VITASDK})
    set(CMAKE_TOOLCHAIN_FILE "${VITASDK}/share/vita.toolchain.cmake" CACHE PATH "Toolchain file")
  else()
    message(FATAL_ERROR "Please define VITASDK to point to your SDK path!")
  endif()
endif()

# Include toolchain and Vita-specific CMake macros
include("${CMAKE_TOOLCHAIN_FILE}")
include("${VITASDK}/share/vita.cmake" REQUIRED)

project(kkr C CXX)

# App metadata
set(VITA_APP_NAME "katzenklein-redux")
set(VITA_TITLEID  "KKRBROCRO")
set(VITA_MKSFOEX_FLAGS "-d ATTRIBUTE2=12")

# Common compiler flags for C
set(COMMON_C_FLAGS
    -g
    #-O0 #debug to amke it close to src and comment -O3
    -D_GNU_SOURCE
    -Wall
    -O3
    -fdiagnostics-color=always
    -fno-optimize-sibling-calls
    -mfloat-abi=softfp
)


# Linker flags for symbol wrapping
set(WRAP_LINK_FLAGS
    -Wl,-q
    #
    #-Wl,--wrap,memcpy
    #-Wl,--wrap,memset
    #-Wl,--wrap,malloc
    #-Wl,--wrap,memalign
    #-Wl,--wrap,free
    #-Wl,--wrap,calloc
    #-Wl,--wrap,realloc
    -Wl,--allow-multiple-definition
)

# Build loader as a static library (exclude main.c)
file(GLOB LOADER_SOURCES "${CMAKE_SOURCE_DIR}/loader/*.c")
list(REMOVE_ITEM LOADER_SOURCES "${CMAKE_SOURCE_DIR}/loader/main.c")
add_library(loader STATIC ${LOADER_SOURCES})
target_compile_options(loader PRIVATE ${COMMON_C_FLAGS})

# Main executable: compile only main.c, then link loader + all other libraries
add_executable(kkr loader/main.c)
target_compile_options(kkr PRIVATE ${COMMON_C_FLAGS})
target_compile_options(kkr PRIVATE -std=c++11 -Wno-write-strings -fpermissive -fno-rtti)
target_link_options(kkr PRIVATE ${WRAP_LINK_FLAGS})
target_link_libraries(kkr PRIVATE
    loader
    pthread
    SDL2_mixer
    xmp
    SDL2_image
    SDL2_ttf
    SDL2_net
    freetype
    soloud_static
    SDL2
    png
    webp
    jpeg
    mikmod
    z
    m
    sndfile
    opusfile
    opus
    modplug
    vorbisfile
    vorbis
    vorbisenc
    ogg
    FLAC
    openal
    imgui
    vitaGL
    stdc++
    vitashark
    SceShaccCgExt
    mathneon
    mpg123
    taihen_stub
    kubridge_stub
    SceNet_stub
    SceNetCtl_stub
    SceAppMgr_stub
    SceAppUtil_stub
    SceAudio_stub
    SceAudioIn_stub
    SceCtrl_stub
    SceCommonDialog_stub
    SceDisplay_stub
    SceFios2_stub
    SceGxm_stub
    SceIme_stub
    SceLibcBridge_stub
    SceShaccCg_stub
    SceSysmodule_stub
    ScePower_stub
    SceTouch_stub
    SceMotion_stub
    SceHid_stub
    SceVshBridge_stub
    SceKernelDmacMgr_stub
)

# Post-build: copy raw ELF for parsing
add_custom_command(
    TARGET kkr
    POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:kkr> ${CMAKE_BINARY_DIR}/kkr.elf
    COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:kkr> /home/mint/Desktop/vita-parse-core/kkr.elf
    COMMENT "Copying raw ELF for parsing output"
)

# Package into SELF and VPK
vita_create_self(eboot.bin kkr UNSAFE)
vita_create_vpk(kkr.vpk ${VITA_TITLEID} eboot.bin
    VERSION ${VITA_VERSION}
    NAME ${VITA_APP_NAME}
    FILE
        ${CMAKE_SOURCE_DIR}/livearea/icon0.png sce_sys/icon0.png
        ${CMAKE_SOURCE_DIR}/livearea/pic0.png sce_sys/pic0.png
        ${CMAKE_SOURCE_DIR}/livearea/bg0.png sce_sys/livearea/contents/bg0.png
        ${CMAKE_SOURCE_DIR}/livearea/startup.png sce_sys/livearea/contents/startup.png
        ${CMAKE_SOURCE_DIR}/livearea/template.xml sce_sys/livearea/contents/template.xml
)
