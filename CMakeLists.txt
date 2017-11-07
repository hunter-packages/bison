cmake_minimum_required(VERSION 3.1)

project(bison LANGUAGES C CXX)

include(CheckCCompilerFlag)
include(CheckCSourceCompiles)
include(CheckCXXCompilerFlag)
include(CheckFunctionExists)
include(CheckIncludeFile)

function(append_flag_if_supported flags flag lang)
  string(MAKE_C_IDENTIFIER "supports${flag}" flag_supported_var)
  if(lang STREQUAL cxx)
    check_cxx_compiler_flag(${flag} ${flag_supported_var})
  else()
    check_c_compiler_flag(${flag} ${flag_supported_var})
  endif()
  if(${flag_supported_var})
    list(APPEND flags "${flag}")
  endif()
endfunction()

set(CMAKE_REQUIRED_FLAGS "-Wunknown-pragmas -Werror")
check_c_source_compiles("
  #pragma GCC diagnostic push
  #pragma GCC diagnostic pop
  int main() { return 0; }"
  gcc_pragma_push_works)
set(CMAKE_REQUIRED_FLAGS)

set(bison_gcc_warnings_help
    "Turn on lots of GCC warnings (not recommended).")
set(bison_gcc_warnings_help
    "${bison_gcc_warnings} Also, issue synclines from the examples/ to ")
set(bison_gcc_warnings_help
    "${bison_gcc_warnings_help} the corresponding source in the Texinfo doc.")
option(bison_gcc_warnings OFF)

if(bison_gcc_warnings)
  foreach(warning -Wall -Wextra -Wno-sign-compare -Wcast-align -Wdocumentation
                  -Wformat -Wpointer-arith -Wwrite-strings)
    append_flag_if_supported(warn_common ${warning} c)
  endforeach()

  foreach(warning -Wbad-function-cast -Wshadow -Wstrict-prototypes)
    append_flag_if_supported(warn_c ${warning} c)
  endforeach()

  foreach(warning -Wnoexcept)
    append_flag_if_supported(warn_cxx ${warning} cxx)
  endforeach()

  foreach(warning -Wundef -pedantic -Wsign-compare -fno-color-diagnostics)
    append_flag_if_supported(warn_tests ${warning} cxx)
  endforeach()

  foreach(flag -Wno-null-conversion -Wno-sign-compare
               -Wno-zero-as-null-pointer-constant)
    append_flag_if_supported(flex_cxx_flags ${flag} cxx)
  endforeach()
endif()

check_c_source_compiles("
  #include <limits.h>
  int test_array[CHAR_BIT]; int main() { return 0; }"
  c_works)
if(NOT c_works)
  message(FATAL_ERROR "C compiler does not work")
endif()

check_cxx_source_compiles("
  #include <cstdlib>
  #include <iostream>
  #include <map>
  #include <string>
  using namespace std;
  int main()
  {
      std::cerr << \"\";
      cout << \"\";
      typedef std::pair<unsigned int, int> uipair;
      std::map<unsigned int, int> m;
      std::map<unsigned int, int>::iterator i;
      m.insert (uipair (4, -4));
      for (i = m.begin (); i != m.end (); ++i)
          if (i->first != 4)
              return 1;
  }"
  cxx_works)
if(NOT cxx_works)
  message(FATAL_ERROR "C++ compiler does not work")
endif()

option(bison_build_yacc "build a yacc command and an -ly library" ON)

check_include_file("locale.h" have_locale_h)

foreach(inline_keyword inline __inline __inline__)
  check_c_source_compiles(
    "${inline_keyword} void foo() {} int main() { return 0; }"
    have_${inline_keyword})
  if(have_${inline_keyword})
    set(inline ${inline_keyword})
    break()
  endif()
endforeach()

check_function_exists(setlocale have_setlocale)
