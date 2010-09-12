cmake_minimum_required(VERSION 2.8)

project(libtiff)

include_directories(BEFORE ${PROJECT_SOURCE_DIR})
include_directories(BEFORE ${PROJECT_BINARY_DIR})

include(CTest)
include(CPack)

set (EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin CACHE PATH "Single output directory for building all executables.")
set (LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin CACHE PATH "Single output directory for building all libraries.")
mark_as_advanced(LIBRARY_OUTPUT_PATH EXECUTABLE_OUTPUT_PATH)

# library object files common to compression and decompression
set(common_SRCS
  tif_aux.c
  tif_close.c
  tif_codec.c
  tif_compress.c
  tif_color.c
  tif_dir.c
  tif_dirinfo.c
  tif_dirread.c
  tif_dirwrite.c
  tif_dumpmode.c
  tif_error.c
  tif_extension.c
  tif_fax3.c
  tif_flush.c
  tif_getimage.c
  tif_luv.c
  tif_lzw.c
  tif_next.c
  tif_open.c
  tif_packbits.c
  tif_predict.c
  tif_print.c
  tif_read.c
  tif_strip.c
  tif_swab.c
  tif_tile.c
  tif_version.c
  tif_warning.c
  tif_write.c
  tif_zip.c
  tif_jpeg.c
  )

add_definitions(-DJPEG_SUPPORT -DZIP_SUPPORT -D_HPUX_SOURCE)

if(WIN32)
  if(BUILD_SHARED_LIBS)
    set(TIFFDLL 1)
  else(BUILD_SHARED_LIBS)
    set(TIFFSTATIC 1)
  endif(BUILD_SHARED_LIBS)
endif(WIN32)

if(NOT UNIX)
  add_definitions( -DTIF_PLATFORM_CONSOLE )
  set(common_SRCS ${common_SRCS} tif_win32.c tif_fax3sm.c)
  if(NOT MINGW)
    set(common_SRCS ${common_SRCS} libtiff.def)
  endif(NOT MINGW)
else (NOT UNIX)
  set(TIFF_RIGHT_VERSION 1)

  set(common_SRCS ${common_SRCS} tif_unix.c tif_fax3sm.c)
  set_source_files_properties(tif_fax3sm.c GENERATED)
  add_executable(mkg3states mkg3states.c)
  if(CYGWIN)
    set(EXE_EXTENSION .exe)
  endiF(CYGWIN)
  add_custom_command(
    OUTPUT ${PROJECT_BINARY_DIR}/tif_fax3sm.c
    COMMAND ${EXECUTABLE_OUTPUT_PATH}/${CMAKE_CFG_INTDIR}/mkg3states${EXE_EXTENSION}
    ARGS -c const ${PROJECT_BINARY_DIR}/tif_fax3sm.c
    DEPENDS mkg3states
    )
endif (NOT UNIX)

include (${CMAKE_ROOT}/Modules/CheckIncludeFiles.cmake)
include(${CMAKE_ROOT}/Modules/CheckSymbolExists.cmake)
include(${CMAKE_ROOT}/Modules/CheckFunctionExists.cmake)

# Check if header file exists and add it to the list.
CHECK_INCLUDE_FILES("assert.h" HAVE_ASSERT_H)
CHECK_INCLUDE_FILES("dlfcn.h" HAVE_DLFCN_H)
CHECK_INCLUDE_FILES("fcntl.h" HAVE_FCNTL_H)
CHECK_INCLUDE_FILES("inttypes.h" HAVE_INTTYPES_H)
CHECK_INCLUDE_FILES("limits.h" HAVE_LIMITS_H)
CHECK_INCLUDE_FILES("malloc.h" HAVE_MALLOC_H)
CHECK_INCLUDE_FILES("memory.h" HAVE_MEMORY_H)
CHECK_INCLUDE_FILES("stdint.h" HAVE_STDINT_H)
CHECK_INCLUDE_FILES("stdlib.h" HAVE_STDLIB_H)
CHECK_INCLUDE_FILES("string.h" HAVE_STRING_H)
CHECK_INCLUDE_FILES("strings.h" HAVE_STRINGS_H)
CHECK_INCLUDE_FILES("sys/stat.h" HAVE_SYS_STAT_H)
CHECK_INCLUDE_FILES("sys/time.h" HAVE_SYS_TIME_H)
CHECK_INCLUDE_FILES("sys/types.h" HAVE_SYS_TYPES_H)
CHECK_INCLUDE_FILES("unistd.h" HAVE_UNISTD_H)
CHECK_INCLUDE_FILES("windows.h" HAVE_WINDOWS_H)
CHECK_INCLUDE_FILES("strings.h" HAVE_STRINGS_H)
CHECK_INCLUDE_FILES("ieeefp.h" HAVE_IEEEFP_H)

CHECK_FUNCTION_EXISTS(floor HAVE_FLOOR)
CHECK_FUNCTION_EXISTS(getopt HAVE_GETOPT)
CHECK_FUNCTION_EXISTS(isascii HAVE_ISASCII)
CHECK_FUNCTION_EXISTS(memmove HAVE_MEMMOVE)
CHECK_FUNCTION_EXISTS(memset HAVE_MEMSET)
CHECK_FUNCTION_EXISTS(mmap HAVE_MMAP)
CHECK_FUNCTION_EXISTS(pow HAVE_POW)
CHECK_FUNCTION_EXISTS(sqrt HAVE_SQRT)
CHECK_FUNCTION_EXISTS(strcasecmp HAVE_STRCASECMP)
CHECK_FUNCTION_EXISTS(strchr HAVE_STRCHR)
CHECK_FUNCTION_EXISTS(strrchr HAVE_STRRCHR)
CHECK_FUNCTION_EXISTS(strstr HAVE_STRSTR)
CHECK_FUNCTION_EXISTS(strtol HAVE_STRTOL)
CHECK_FUNCTION_EXISTS(areroul HAVE_STRTOUL)


include(CheckTypeSize)

CHECK_TYPE_SIZE("signed int" SIZEOF_SIGNED_INT)
CHECK_TYPE_SIZE("unsigned int" SIZEOF_UNSIGNED_INT)
CHECK_TYPE_SIZE("signed short" SIZEOF_SIGNED_SHORT)
CHECK_TYPE_SIZE("unsigned short" SIZEOF_UNSIGNED_SHORT)
CHECK_TYPE_SIZE("signed long" SIZEOF_SIGNED_LONG)
CHECK_TYPE_SIZE("unsigned long" SIZEOF_UNSIGNED_LONG)
CHECK_TYPE_SIZE("signed long long" SIZEOF_SIGNED_LONG_LONG)
CHECK_TYPE_SIZE("unsigned long long" SIZEOF_UNSIGNED_LONG_LONG)
CHECK_TYPE_SIZE("unsigned char *" SIZEOF_UNSIGNED_CHAR_P)


configure_file(${PROJECT_SOURCE_DIR}/tiffDllConfig.h.in
  ${PROJECT_BINARY_DIR}/tiffDllConfig.h)
configure_file(${PROJECT_SOURCE_DIR}/tif_config.h.in
  ${PROJECT_BINARY_DIR}/tif_config.h)

add_library(libtiff ${common_SRCS})
target_link_libraries(libtiff zlib jpeg)

if(UNIX)
  target_link_libraries(libtiff -lm)
endif(UNIX)

set(LIBTIFF_INSTALL_LIB_DIR "/lib/libtiff")

install(TARGETS libtiff
  RUNTIME DESTINATION ${LIBTIFF_INSTALL_LIB_DIR} COMPONENT RuntimeLibraries
  LIBRARY DESTINATION ${LIBTIFF_INSTALL_LIB_DIR} COMPONENT RuntimeLibraries
  ARCHIVE DESTINATION ${LIBTIFF_INSTALL_LIB_DIR} COMPONENT Development)

install(FILES
  ${PROJECT_BINARY_DIR}/tiffDllConfig.h
  ${PROJECT_SOURCE_DIR}/tiffvers.h
  ${PROJECT_SOURCE_DIR}/itk_tiff_mangle.h
  ${PROJECT_SOURCE_DIR}/tiff.h
  ${PROJECT_SOURCE_DIR}/tiffio.h
  ${PROJECT_SOURCE_DIR}/tconf.h
  DESTINATION ${LIBTIFF_INSTALL_LIB_DIR}
  COMPONENT Development)

if(CMAKE_COMPILER_IS_GNUCXX)
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fno-strict-aliasing")
endif(CMAKE_COMPILER_IS_GNUCXX)
