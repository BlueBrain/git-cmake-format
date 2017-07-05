project(git-clang-format-cmake)
cmake_minimum_required(VERSION 2.8)

list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR})
find_package(ClangFormat REQUIRED)

set(GCF_CLANGFORMAT_STYLE "file" CACHE STRING
  "Parameter pass to clang-format -style=<here>")
set(GCF_IGNORE_LIST "" CACHE STRING
  "Semi colon separated list of directories to ignore")

option(CLANG_FORMAT_PRECOMMIT
       "Ensure git staging area is formatted according to clang-format"
       FALSE)

if(CLANG_FORMAT_PRECOMMIT)
  find_package(Git REQUIRED)
  find_package(PythonInterp REQUIRED)

  set(GCF_SCRIPT ${CMAKE_CURRENT_SOURCE_DIR}/git-cmake-format.py)

  execute_process(COMMAND ${GIT_EXECUTABLE} rev-parse --show-toplevel
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    OUTPUT_VARIABLE GCF_GIT_ROOT
    OUTPUT_STRIP_TRAILING_WHITESPACE)
  if(NOT GCF_GIT_ROOT)
    message(WARNING "Not in a git repository")
  else(NOT GCF_GIT_ROOT)
    configure_file(
      ${CMAKE_CURRENT_SOURCE_DIR}/git-pre-commit-hook
      ${GCF_GIT_ROOT}/.git/hooks/pre-commit
      @ONLY)
    unset(GCF_GIT_ROOT)

    add_custom_target(clang-format
      ${PYTHON_EXECUTABLE} ${GCF_SCRIPT}
      --cmake ${GIT_EXECUTABLE}
      ${CLANG_FORMAT_EXECUTABLE} -style=${GCF_CLANGFORMAT_STYLE} -ignore=${GCF_IGNORE_LIST}
  	  WORKING_DIRECTORY ${CMAKE_SOURCE_DIR})
  endif(NOT GCF_GIT_ROOT)
  unset(GCF_SCRIPT)
endif()