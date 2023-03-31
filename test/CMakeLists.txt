# test fortran jedi incr module
add_pfunit_ctest(test_jediincr_module
  TEST_SOURCES test_jediincr.pf
  LINK_LIBRARIES jediincr
  )

# test for adding jedi increment
add_test(NAME test_apply_jediincr
         COMMAND ${PROJECT_SOURCE_DIR}/test/apply_jedi_incr.sh ${PROJECT_BINARY_DIR} ${PROJECT_SOURCE_DIR}
         WORKING_DIRECTORY ${PROJECT_BINARY_DIR}/test)
set_tests_properties(test_apply_jediincr
    PROPERTIES
    DEPENDS "test_jediincr_module; test_python_compare"
    ENVIRONMENT "LANDDA_INPUTS=$ENV{LANDDA_INPUTS};
                 PYTHONPATH=${PROJECT_SOURCE_DIR}/test:$ENV{PYTHONPATH}; 
                 TOL=$ENV{TOL}")    
