project(banking)

cmake_minimum_required(VERSION 3.4)

set(CMAKE_CXX_STANDART 11)
set(CMAKE_CXX_STANDART_REQUIRED ON)

option(BUILD_TESTS "Build tests" OFF)

option(COVERAGE ON)

include(${CMAKE_CURRENT_SOURCE_DIR}/banking/CMakeLists.txt)

if (BUILD_TESTS)
    enable_testing()
    add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/third-party/gtest)
    if (TARGET libbanking)
        add_executable(check_account ${CMAKE_CURRENT_SOURCE_DIR}/banking/tests/test_account.cpp)
        target_link_libraries(check_account libbanking gtest_main gmock_main)
        add_test(NAME account COMMAND check_account)

        add_executable(check_transaction ${CMAKE_CURRENT_SOURCE_DIR}/banking/tests/test_transaction.cpp)
        target_link_libraries(check_transaction libbanking gtest_main gmock_main)
        add_test(NAME transaction COMMAND check_transaction)

        if (COVERAGE)
            target_compile_options(check_account PRIVATE --coverage)
            target_compile_options(check_transaction PRIVATE --coverage)

            target_link_libraries(check_account --coverage)
            target_link_libraries(check_transaction --coverage)
        endif(COVERAGE)
    endif(TARGET libbanking)
endif(BUILD_TESTS)

before_install:
- pip install --user cpp-coveralls

after_success:
- coveralls --root . -E ".*gtest.*" -E ".*CMakeFiles.*"
