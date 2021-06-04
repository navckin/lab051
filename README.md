[![Build Status](https://travis-ci.com/navckin/lab051.svg?branch=main)](https://travis-ci.com/navckin/lab051)
[![Coverage Status](https://coveralls.io/repos/github/navckin/lab051/badge.svg?branch=homework)](https://coveralls.io/github/navckin/lab051?branch=homework)
## Laboratory work V

Данная лабораторная работа посвещена изучению фреймворков для тестирования на примере **GTest**

```sh
$ open https://github.com/google/googletest
```
Gtests - это библиотека для модульного тестирования (англ. unit testing) на языке С++. (по идее)
Общие понятия: Ключевым понятием в Google test framework является понятие утверждения (assert). Утверждение представляет собой выражение, результатом выполнения которого может быть успех (success), некритический отказ (nonfatal failure) и критический отказ (fatal failure). Критический отказ вызывает завершение выполнения теста, в остальных случаях тест продолжается. Сам тест представляет собой набор утверждений.


## Tasks

- [+] 1. Создать публичный репозиторий с названием **lab05** на сервисе **GitHub**
- [+] 2. Выполнить инструкцию учебного материала
- [+] 3. Ознакомиться со ссылками учебного материала
- [+] 4. Составить отчет и отправить ссылку личным сообщением в **Slack**


## Homework

### Задание
1. Создайте `CMakeList.txt` для библиотеки *banking*.

```sh
$ cat > banking/CMakeLists.txt <<EOF
project(banking_lib)

if (NOT TARGET libbanking)
    add_library(libbanking STATIC
        ${CMAKE_CURRENT_LIST_DIR}/Account.cpp
        ${CMAKE_CURRENT_LIST_DIR}/Transaction.cpp
    )

    install(TARGETS libbanking
        ARCHIVE DESTINATION lib
        LIBRARY DESTINATION lib
    )
endif(NOT TARGET libbanking)

include_directories(${CMAKE_CURRENT_LIST_DIR})
EOF
```



3. Создайте модульные тесты на классы `Transaction` и `Account`.
    * Используйте mock-объекты.
    * Покрытие кода должно составлять 100%.
    * 
Тест для `Transaction`
```sh

#include "Account.h"
#include "Transaction.h"

#include <gtest/gtest.h>
#include <gmock/gmock.h>

using testing::_;

class TransactionMock : public Transaction {
public:
    MOCK_METHOD3(Make, bool(Account& from, Account& to, int sum));
};

TEST(Transaction, Mock) {
    TransactionMock tr;
    Account acc1(1, 100);
    Account acc2(2, 300);

    EXPECT_CALL(tr, Make(_, _, _)).Times(5);
    tr.set_fee(200);
    tr.Make(acc1, acc2, 200);
    tr.Make(acc2, acc1, 300);
    // throws
    tr.Make(acc2, acc1, 50);
    tr.Make(acc2, acc1, 0);
    tr.Make(acc1, acc2, -8);
}

TEST(Transaction, TestTransaction) {
    Transaction tr;
    Account a1(1, 100), a2(2, 300);
    tr.set_fee(10);
    EXPECT_EQ(tr.fee(), 10);
    EXPECT_THROW(tr.Make(a1, a2, 90), std::logic_error);
    EXPECT_THROW(tr.Make(a1, a2, -1), std::invalid_argument);
    EXPECT_THROW(tr.Make(a1, a1, 100), std::logic_error);
    EXPECT_FALSE(tr.Make(a1, a2, 400));
    EXPECT_FALSE(tr.Make(a2, a1, 300));
    EXPECT_TRUE(tr.Make(a2, a1, 246));
}
```
Тест для `Account`
```sh
#include "Account.h"
#include "Transaction.h"

#include <gtest/gtest.h>
#include <gmock/gmock.h>

using testing::_;

class AccountMock : public Account {
public:
    explicit AccountMock(int id, int balance) : Account(id, balance) {}
    MOCK_CONST_METHOD0(GetBalance, int());
    MOCK_METHOD1(ChangeBalance, void(int diff));
    MOCK_METHOD0(Lock, void());
    MOCK_METHOD0(Unlock, void());
};

TEST(Account, Mock) {
    AccountMock acc(1, 666);
    EXPECT_CALL(acc, GetBalance()).Times(1);
    EXPECT_CALL(acc, ChangeBalance(_)).Times(2);
    EXPECT_CALL(acc, Lock()).Times(2);
    EXPECT_CALL(acc, Unlock()).Times(1);
    acc.GetBalance();
    acc.ChangeBalance(100);
    acc.Lock();
    acc.ChangeBalance(100);
    acc.Lock();
    acc.Unlock();
}

TEST(Account, TestAccount) {
    Account acc(1, 666);
    EXPECT_EQ(acc.id(), 1);
    EXPECT_EQ(acc.GetBalance(), 666);
    EXPECT_THROW(acc.ChangeBalance(200), std::runtime_error);
    EXPECT_NO_THROW(acc.Lock());
    acc.ChangeBalance(200);
    EXPECT_EQ(acc.GetBalance(), 866);
    EXPECT_THROW(acc.Lock(), std::runtime_error);
    EXPECT_NO_THROW(acc.Unlock());
}
```

4. Настройте сборочную процедуру на **TravisCI**. CMakeLists.txt для сборки с тестами:

```sh
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
```
файл travis.yml:
```sh
language: cpp
os:
  - linux
addons:
  apt:
    sources:
    - george-edison55-precise-backports
    packages:
    - cmake
    - cmake-data
before_install:
- pip install --user cpp-coveralls
script:
- cmake -H. -B_build -DBUILD_TESTS=ON
- cmake --build _build
- cmake --build _build --target test -- ARGS=--verbose
after_success:
- coveralls --root . -E ".*gtest.*" -E ".*CMakeFiles.*"
```

6. Настройте [Coveralls.io](https://coveralls.io/).

Файл coveralls.yml:

 ```sh
service_name: travis-pro
repo_token: токен, который висит на сайте Coveralls.io
```

## Links

- [C++ CI: Travis, CMake, GTest, Coveralls & Appveyor](http://david-grs.github.io/cpp-clang-travis-cmake-gtest-coveralls-appveyor/)
- [Boost.Tests](http://www.boost.org/doc/libs/1_63_0/libs/test/doc/html/)
- [Catch](https://github.com/catchorg/Catch2)

```
Copyright (c) 2015-2021 The ISC Authors
```
