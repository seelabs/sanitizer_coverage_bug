cmake_minimum_required(VERSION 3.1.0)

# Observed behavior table:
# use_lld | use_coverage | behavior
#   Off   |   Off        |  No Issues
#   Off   |   On         | Link Error for two_c_funs (see (1))
#   On    |   Off        | No Issues
#   On    |   On         | Segmentation fault for two_c_funs
#
# Conclusion:
# 1) -sanitize-coverage cannot link in a c file with more than one function with external linkage
# 2) The lld linker does not detect the problem and compiles the program (tho the program will segfault)
#
# (1) Link error is:
#
# [1/2] Linking CXX executable two_c_funs
# FAILED: two_c_funs 
# : && /home/swd/apps/clang-6.0/bin/clang++    -fsanitize=fuzzer -fsanitize-coverage=trace-pc-guard  -rdynamic CMakeFiles/two_c_funs.dir/test.cpp.o CMakeFiles/two_c_funs.dir/two_c_funs.c.o  -o two_c_funs   && :
# `.text.sancov.module_ctor.6' referenced in section `.init_array.2[sancov.module_ctor.6]' of CMakeFiles/two_c_funs.dir/two_c_funs.c.o: defined in discarded section `.text.sancov.module_ctor.6[sancov.module_ctor]' of CMakeFiles/two_c_funs.dir/two_c_funs.c.o
# clang-6.0: error: linker command failed with exit code 1 (use -v to see invocation)
# [2/2] Linking CXX executable one_c_fun
# ninja: build stopped: subcommand failed.



set(use_lld Off)
set(use_coverage On)

if (use_lld)
  set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -fuse-ld=lld")
endif()

set(OneCFunFuzzerSrc
  ${CMAKE_SOURCE_DIR}/test.cpp
  ${CMAKE_SOURCE_DIR}/one_c_fun.c
  )

set(TwoCFunFuzzerSrc
  ${CMAKE_SOURCE_DIR}/test.cpp
  ${CMAKE_SOURCE_DIR}/two_c_funs.c
  )

add_compile_options(-fsanitize=fuzzer -fno-omit-frame-pointer)
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -fsanitize=fuzzer")
if (use_coverage)
  add_compile_options(-fsanitize-coverage=trace-pc-guard)
  set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -fsanitize-coverage=trace-pc-guard")
endif()

add_executable(one_c_fun  ${OneCFunFuzzerSrc})
add_executable(two_c_funs  ${TwoCFunFuzzerSrc})

