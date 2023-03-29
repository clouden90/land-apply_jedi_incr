# unit testing
Unit Testing Framework for ufs-landDA System. We implemented ctest and pFUnit (unit testing for fortran). Currently land-offline_workflow supports 9 tests as shown below. Users who plan to `design/add` a new test should refer to `Adding test` for details on how to do so. At a minimum, these users will need to add the new test case to the `./test/CMakeLists.txt`, and add corresponding files in the `test` folder.

```
$ ctest -N
Test project /work2/noaa/epic-ps/ycteng/land_DA/20230307/build
  Test #1: test_jediincr_module
  Test #2: test_python_compare_ecbuild
  Test #3: test_python_compare_ctest
  Test #4: test_vector2tile
  Test #5: test_create_ens
  Test #6: test_letkfoi_snowda
  Test #7: test_apply_jediincr
  Test #8: test_tile2vector
  Test #9: test_land_driver

Total Tests: 9
```

## Building and running the existing unit tests
Before you write your own unit tests, you should make sure that you can build and run the existing tests.

The following is an example of how to compile/build and run unit testing, on the Hera/Orion HPC.

```bash
# download demo source code
git clone -b unit_test_demo --recurse-submodules https://github.com/yichengt90/land-apply_jedi_incr.git 

# load the machine specific modules (now only support Hera/Orion, here we use Orion as example)
module use land-apply_jedi_incr/modulefiles
module load landda_orion.intel

# git clone pFUint and build
export FC=mpiifort
export CC=mpiicc
export CXX=mpiicpc
git clone https://github.com/Goddard-Fortran-Ecosystem/pFUnit.git
mkdir -p pFUnit/build; cd pFUnit/build; ecbuild ..; make -j2; make install

# compiling
cd ../land-apply_jedi_incr
mkdir build
cd build
ecbuild .. -DCMAKE_PREFIX_PATH={YOUR ROOT PATH}/pFUnit/build/installed
make -j2

# grab a compute node and run ctests
salloc --ntasks 6 --exclusive --qos=windfall --time=00:05:00
#
module use ../modulefiles
module load landda_orion.intel
#
ctest --stop-on-failure

Screen output:

Test project /lustre/land/land-apply_jedi_incr/build
    Start 1: test_jediincr_module
1/3 Test #1: test_jediincr_module .............   Passed    0.02 sec
    Start 2: test_python_compare
2/3 Test #2: test_python_compare ..............   Passed    0.13 sec
    Start 3: test_apply_jediincr
3/3 Test #3: test_apply_jediincr ..............   Passed    1.76 sec

100% tests passed, 0 tests failed out of 3

Total Test time (real) =   1.92 sec
```
## Add new test
We build the unit tests using a build system called CMake. There are a few steps needed to get your new unit tests to build alongside the others:

1. Add the new production module to the build system
2. Tell CMake about your new unit test directory
3. Add/Mod CMakeLists.txt file in your new unit test directory

## General guidelines for writing unit tests
Good unit tests test a single, well-defined condition. This generally means that you make a single call to the function / subroutine that you're testing, with a single set of inputs.Good unit tests are "FIRST":
* Fast (order milliseconds or less)
  * This means that, generally, they should not do any file i/o. Also, if you are testing a complex function, test it with a simple set of inputs - not a 10,000-element array that will require a few seconds of runtime to process.
* Independent
  * This means that test Y shouldn't depend on some global variable that was created by test X. Dependencies like this cause problems if the tests run in a different order, if one test is dropped, etc.
* Repeatable
  * This means, for example, that you shouldn't generate random numbers in your tests.
* Self-verifying
  * This means that you shouldn't write a test that writes out its answers for manual comparison. Tests should generate an automatic pass/fail result.
* Timely
  * This means that the tests should be written before the production code (Test Driven Development), or immediately afterwards - not six months later when it's time to finally merge your changes onto the trunk, and have forgotten the details of what you have written. Much of the benefit of unit tests comes from developing them alongside the production code.

## References
Below are some useful refernces:

* [Creating and running tests with CTest](https://coderefinery.github.io/cmake-workshop/testing/) for detailed usages of ctest.
* [cesm_unit_test_tutorial](https://github.com/NCAR/cesm_unit_test_tutorial#add-the-new-production-module-to-the-build-system) for detailed usages of pFUnit.
