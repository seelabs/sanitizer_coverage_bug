This project is a minimal test case for a possible bug in clang's sanitize-coverage.

If a c file defines more than one function with external linkage, then the program will
either crash (if the lld linker is used) or fail to compile (if it is not used).

The project builds two executables: one_c_fun and two_c_funs. The first builds ands links a c file that defines one external function, the second
builds and links a c file with two external functions. There are settings in the CMakeFile to control if `sanitize-coverage` is used and if the lld linker is used. The observed behavior is as follows:

 Observed behavior table:

| use_lld | use_coverage | behavior |
| ------- | ------------ | -------- |
|   Off   |   Off        |  No Issues |
|   Off   |   On         | Link Error for two_c_funs (see (1)) |
|   On    |   Off        | No Issues |
|   On    |   On         | Segmentation fault for two_c_funs |

 Conclusion:
 1) -sanitize-coverage cannot link in a c file with more than one function with external linkage
 2) The lld linker does not detect the problem and compiles the program (tho the program will segfault)

 (1) Link error is:

 [1/2] Linking CXX executable two_c_funs
 FAILED: two_c_funs 
 : && /home/swd/apps/clang-6.0/bin/clang++    -fsanitize=fuzzer -fsanitize-coverage=trace-pc-guard  -rdynamic CMakeFiles/two_c_funs.dir/test.cpp.o CMakeFiles/two_c_funs.dir/two_c_funs.c.o  -o two_c_funs   && :
 `.text.sancov.module_ctor.6' referenced in section `.init_array.2[sancov.module_ctor.6]' of CMakeFiles/two_c_funs.dir/two_c_funs.c.o: defined in discarded section `.text.sancov.module_ctor.6[sancov.module_ctor]' of CMakeFiles/two_c_funs.dir/two_c_funs.c.o
 clang-6.0: error: linker command failed with exit code 1 (use -v to see invocation)
 [2/2] Linking CXX executable one_c_fun
 ninja: build stopped: subcommand failed.
