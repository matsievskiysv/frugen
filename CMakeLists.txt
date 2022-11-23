cmake_minimum_required(VERSION 3.0)
project(frugen C)
if(COMMAND cmake_policy)
	cmake_policy(SET CMP0003 NEW)
endif(COMMAND cmake_policy)

execute_process(COMMAND
	git
	describe
	--always
	--long
	--dirty
	WORKING_DIRECTORY
	"${CMAKE_CURRENT_SOURCE_DIR}"
	RESULT_VARIABLE
	res
	OUTPUT_VARIABLE
	gitver
	ERROR_QUIET
	OUTPUT_STRIP_TRAILING_WHITESPACE)

if(NOT res EQUAL 0)
	set(gitver "UNKNOWN")
endif()

string(REPLACE "-" "." gitver "${gitver}")

option(BINARY_32BIT "compile 32bit version" OFF)
option(DEBUG_OUTPUT "show extra debug output" OFF)
option(DEBUG_SYMBOLS "compile with debug symbols" OFF)
option(DEBUG_NO_OPTIMIZE "do not optimize the code" OFF)

if (NOT BUILD_SHARED_LIBS)
  set(CMAKE_FIND_LIBRARY_SUFFIXES .a)
  set(CMAKE_LINK_SEARCH_START_STATIC ON)
  set(CMAKE_LINK_SEARCH_END_STATIC ON)
endif (NOT BUILD_SHARED_LIBS)

find_library(JSON_LIB json-c)
if (JSON_LIB)
	message (STATUS "Using JSON Library found at " ${JSON_LIB})
else (JSON_LIB)
	message (WARNING "JSON library *NOT* found. JSON support *disabled*!")
endif (JSON_LIB)

# {{{ fru
add_library(fru fru.c)
target_compile_options(fru PRIVATE -Wall -Wextra -Wunreachable-code)
target_compile_definitions(fru PRIVATE VERSION="${gitver}")
if (BINARY_32BIT)
  target_compile_options(fru PUBLIC -m32)
  target_link_options(fru PUBLIC -m32)
endif (BINARY_32BIT)
if (DEBUG_OUTPUT)
  target_compile_definitions(fru PRIVATE DEBUG)
endif(DEBUG_OUTPUT)
if (DEBUG_SYMBOLS)
  target_compile_options(fru -g3)
endif (DEBUG_SYMBOLS)
if (DEBUG_NO_OPTIMIZE)
  target_compile_options(fru PRIVATE -O0)
endif (DEBUG_NO_OPTIMIZE)
set_target_properties(fru PROPERTIES PUBLIC_HEADER fru.h)
install(TARGETS fru)
# }}}

# {{{ frugen
add_executable(frugen frugen.c)
if (NOT BUILD_SHARED_LIBS)
  target_link_options(frugen PUBLIC -static)
endif (NOT BUILD_SHARED_LIBS)
target_link_libraries(frugen fru)
target_compile_options(frugen PRIVATE -Wall -Wextra -Wunreachable-code)
target_compile_definitions(frugen PRIVATE VERSION="${gitver}")
if (JSON_LIB)
  target_compile_definitions(frugen PRIVATE __HAS_JSON__)
  target_link_libraries(frugen ${JSON_LIB})
endif (JSON_LIB)
if (BINARY_32BIT)
  target_compile_options(frugen PUBLIC -m32)
  target_link_options(frugen PUBLIC -m32)
endif (BINARY_32BIT)
if (DEBUG_OUTPUT)
  target_compile_definitions(frugen PRIVATE DEBUG)
endif(DEBUG_OUTPUT)
if (DEBUG_SYMBOLS)
  target_compile_options(frugen -g3)
endif (DEBUG_SYMBOLS)
if (DEBUG_NO_OPTIMIZE)
  target_compile_options(frugen PRIVATE -O0)
endif (DEBUG_NO_OPTIMIZE)
install(TARGETS frugen)
# }}}
