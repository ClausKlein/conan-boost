project(test_package)
cmake_minimum_required(VERSION 3.1.2)
enable_testing()

set(Boost_DEBUG 1)
include(${CMAKE_BINARY_DIR}/conanbuildinfo.cmake)

if(CONAN_SETTINGS_OS STREQUAL "Android")
    set(CONAN_LIBCXX "") # NDK fails when specified: https://github.com/android-ndk/ndk/issues/541
endif()

conan_basic_setup()

if(NOT HEADER_ONLY)
    if(WITH_PYTHON)
        find_package(Boost COMPONENTS regex python REQUIRED)
    else()
        find_package(Boost COMPONENTS regex REQUIRED)
    endif()

    include_directories(${Boost_INCLUDE_DIRS})
    add_executable(lambda lambda.cpp)
    add_executable(regex_exe regex.cpp)
    add_executable(my_test test.cpp)
    target_link_libraries(regex_exe ${Boost_LIBRARIES})

    message("LIBS=> ${CONAN_LIBS}")
    target_link_libraries(my_test ${CONAN_LIBS})
    # FIXME! conan_target_link_libraries(my_test)

    if(WITH_PYTHON)
        add_library(hello_ext SHARED python.cpp)
        if(WIN32)
            set_target_properties(hello_ext PROPERTIES SUFFIX ".pyd")
            target_include_directories(hello_ext PRIVATE C:/Python27/include)
            target_link_libraries(hello_ext C:/Python27/libs/python27.lib ${CONAN_LIBS})
        endif()
    endif()
    add_test(NAME TestRegex
             COMMAND ${CMAKE_CROSSCOMPILING_EMULATOR} "$<TARGET_FILE:regex_exe>")

else()
    add_executable(lambda lambda.cpp)
    target_include_directories(lambda PRIVATE ${CONAN_INCLUDE_DIRS_BOOST})
endif()

add_test(NAME TestLambda
         COMMAND ${CMAKE_CROSSCOMPILING_EMULATOR} "$<TARGET_FILE:lambda>" ${TEST_ARGS})

if(NOT HEADER_ONLY)
    # Test a different exe linking with the CONAN_LIBS to actually test the package_info
    add_executable(newregex regex.cpp)
    target_link_libraries(newregex ${CONAN_LIBS})
    # FIXME! conan_target_link_libraries(newregex)
    add_test(NAME TestRegexNew
             COMMAND ${CMAKE_CROSSCOMPILING_EMULATOR} "$<TARGET_FILE:newregex>" ${TEST_ARGS})
endif()
