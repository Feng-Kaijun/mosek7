if(BUILD_TESTING)
  find_package(Threads MODULE REQUIRED)

  add_library(mosek SHARED IMPORTED)
  set_target_properties(mosek PROPERTIES
    INTERFACE_LINK_LIBRARIES
      "${MOSEK_PLATFORM_DIR}/bin/${IOMP5_LIBRARY_FILE};${LIBM_LIBRARY};Threads::Threads"
    IMPORTED_LOCATION "${MOSEK_PLATFORM_DIR}/bin/${MOSEK_LIBRARY_FILE}"
    INTERFACE_INCLUDE_DIRECTORIES "${MOSEK_PLATFORM_DIR}/h"
  )

  set(MOSEK_EXAMPLES_DIR
    "${MOSEK_ARCHIVE_DIR}/${PROJECT_VERSION_MAJOR}/tools/examples"
  )

  function(mosek_c_add_test name)
    add_executable(${name}_c "${MOSEK_EXAMPLES_DIR}/c/${name}.c")
    target_link_libraries(${name}_c PRIVATE mosek)
    add_test(NAME ${name}_c COMMAND ${name}_c ${ARGN})
    set_tests_properties(${name}_c PROPERTIES
      LABELS "c;small"
      REQUIRED_FILES "$ENV{HOME}/mosek/mosek.lic"
      TIMEOUT 60
    )
  endfunction()

  mosek_c_add_test(callback intpnt "${MOSEK_EXAMPLES_DIR}/data/25fv47.mps")

  if(CMAKE_C_COMPILER_ID MATCHES Clang OR CMAKE_C_COMPILER_ID STREQUAL GNU)
    target_compile_options(callback_c PRIVATE -Wno-switch)
  endif()

  mosek_c_add_test(concurrent1 "${MOSEK_EXAMPLES_DIR}/data/25fv47.mps")
  mosek_c_add_test(concurrent2 "${MOSEK_EXAMPLES_DIR}/data/25fv47.mps")
  mosek_c_add_test(cqo1)
  mosek_c_add_test(feasrepairex1 "${MOSEK_EXAMPLES_DIR}/data/feasrepair.lp")
  mosek_c_add_test(lo1)
  mosek_c_add_test(lo2)
  mosek_c_add_test(milo1)
  mosek_c_add_test(mioinitsol)
  mosek_c_add_test(production)
  mosek_c_add_test(qcqo1)
  mosek_c_add_test(qo1)
  mosek_c_add_test(sensitivity)
  mosek_c_add_test(sdo1)
  mosek_c_add_test(solutionquality)
  mosek_c_add_test(solvebasis)
  mosek_c_add_test(solvelinear)

  add_library(scopt_c STATIC
    "${MOSEK_EXAMPLES_DIR}/c/scopt-ext.c"
    "${MOSEK_EXAMPLES_DIR}/c/scopt-ext.h"
  )
  target_link_libraries(scopt_c PUBLIC mosek)

  add_library(expopt_c STATIC
    "${MOSEK_EXAMPLES_DIR}/c/dgopt.c"
    "${MOSEK_EXAMPLES_DIR}/c/dgopt.h"
    "${MOSEK_EXAMPLES_DIR}/c/expopt.c"
    "${MOSEK_EXAMPLES_DIR}/c/expopt.h"
  )
  target_link_libraries(expopt_c PUBLIC mosek scopt_c)

  mosek_c_add_test(tstexpopt)
  target_link_libraries(tstexpopt_c PRIVATE expopt_c)

  mosek_c_add_test(tstscopt)
  target_link_libraries(tstscopt_c PRIVATE scopt_c)

  if(Java_FOUND)
    set(MOSEK_CLASSPATH
      "${CMAKE_CURRENT_BINARY_DIR}:${MOSEK_PLATFORM_DIR}/bin/mosek.jar"
    )

    function(mosek_java_add_test name)
      add_custom_command(
        OUTPUT com/mosek/example/${name}.class
        COMMAND
          "${Java_JAVAC_EXECUTABLE}"
          -d "${CMAKE_CURRENT_BINARY_DIR}"
          -classpath "${MOSEK_CLASSPATH}"
          "${MOSEK_EXAMPLES_DIR}/java/${name}.java"
        DEPENDS "${MOSEK_EXAMPLES_DIR}/java/${name}.java"
      )
      add_custom_target(${name}_java ALL
        DEPENDS com/mosek/example/${name}.class
      )
      add_test(NAME ${name}_java
        COMMAND
          "${Java_JAVA_EXECUTABLE}"
          -classpath "${MOSEK_CLASSPATH}"
          com.mosek.example.${name}
          ${ARGN}
      )
      set_tests_properties(${name}_java PROPERTIES
        LABELS "java;small"
        REQUIRED_FILES "$ENV{HOME}/mosek/mosek.lic"
        TIMEOUT 60
      )
    endfunction()

    mosek_java_add_test(concurrent1 "${MOSEK_EXAMPLES_DIR}/data/25fv47.mps")
    mosek_java_add_test(cqo1)
    mosek_java_add_test(feasrepairex1
      "${MOSEK_EXAMPLES_DIR}/data/feasrepair.lp"
    )
    mosek_java_add_test(lo1)
    mosek_java_add_test(lo2)
    mosek_java_add_test(milo1)
    mosek_java_add_test(mioinitsol)
    mosek_java_add_test(production)
    mosek_java_add_test(qcqo1)
    mosek_java_add_test(qo1)
    mosek_java_add_test(sdo1)
    mosek_java_add_test(sensitivity)
    mosek_java_add_test(simple "${MOSEK_EXAMPLES_DIR}/data/25fv47.mps")
    mosek_java_add_test(solvebasis)

    set(MOSEK_JAVA_ADDITIONAL_MAKE_CLEAN_FILES
      com/mosek/example/concurrent1$1.class
      com/mosek/example/cqo1$1.class
      com/mosek/example/cqo1$2.class
      com/mosek/example/feasrepairex1$1.class
      com/mosek/example/lo1$1.class
      com/mosek/example/lo1$2.class
      com/mosek/example/lo2$1.class
      com/mosek/example/lo2$2.class
      com/mosek/example/milo1$1.class
      com/mosek/example/milo1$2.class
      com/mosek/example/qcqo1$1.class
      com/mosek/example/qcqo1$2.class
      com/mosek/example/qo1$1.class
      com/mosek/example/qo1$2.class
      com/mosek/example/sdo1$1.class
      com/mosek/example/sdo1$2.class
      com/mosek/example/sensitivity$1.class
      com/mosek/example/simple$1.class
    )

    set_directory_properties(PROPERTIES ADDITIONAL_MAKE_CLEAN_FILES
      "${MOSEK_JAVA_ADDITIONAL_MAKE_CLEAN_FILES}"
    )
  endif()

  if(Matlab_FOUND)
    set(MOSEK_MATLAB_DIR
      "${MOSEK_ARCHIVE_DIR}/${PROJECT_VERSION_MAJOR}/toolbox")

    function(mosek_matlab_add_test name)
      matlab_add_unit_test(NAME ${name}_matlab
        UNITTEST_FILE "${MOSEK_MATLAB_DIR}/examples/${name}.m"
        TIMEOUT -1
        ADDITIONAL_PATH "${MOSEK_MATLAB_DIR}/${MOSEK_MATLAB_RELEASE}"
        MATLAB_ADDITIONAL_STARTUP_OPTIONS -nojvm
      )
      set_tests_properties(${name}_matlab PROPERTIES
        LABELS "matlab;small"
        REQUIRED_FILES "$ENV{HOME}/mosek/mosek.lic"
        TIMEOUT 60
      )
    endfunction()

    mosek_matlab_add_test(lo1)
    mosek_matlab_add_test(lo2)
    mosek_matlab_add_test(milo1)
    mosek_matlab_add_test(qco)
    mosek_matlab_add_test(qo1)
    mosek_matlab_add_test(sdo1)
    mosek_matlab_add_test(sensitivity)
  endif()
endif()
