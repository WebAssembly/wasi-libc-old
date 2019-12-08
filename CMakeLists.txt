
cmake_minimum_required(VERSION 3.15)

project(wasi-libc
  LANGUAGES ASM C)

option(ENABLE_THREADS "enable threads" OFF)

add_compile_options(-target wasm32-unknown-none)

include_directories(SYSTEM basics/include)
if(ENABLE_THREADS)
  add_compile_options(-mthread-model posix)
  add_compile_definitions($<$<COMPILE_LANGUAGE:C>:WASM_THREAD_MODEL_POSIX>)
else()
  add_compile_options(-mthread-model single)
  add_compile_definitions($<$<COMPILE_LANGUAGE:C>:WASM_THREAD_MODEL_SINGLE>)
endif()

add_library(startup OBJECT
  basics/libc/crt1.s)

add_library(basics STATIC
  basics/libc/string.c)
set_target_properties(basics PROPERTIES
  OUTPUT_NAME libc-basics)

add_library(dlmalloc
  dlmalloc/src/wrapper.c)
set_target_properties(dlmalloc PROPERTIES
  OUTPUT_NAME libc-dlmalloc)
target_include_directories(dlmalloc PRIVATE
  dlmalloc/include)

install(DIRECTORY basics/include
  DESTINATION .)
install(FILES $<TARGET_OBJECTS:startup>
  DESTINATION lib)
install(TARGETS basics dlmalloc
  DESTINATION lib)
