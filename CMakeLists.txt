cmake_minimum_required(VERSION 3.12)

set(PATCH_VERSION "1" CACHE INTERNAL "Patch version")
set(PROJECT_VESRION 0.0.${PATCH_VERSION})

project(mapreduce VERSION ${PROJECT_VESRION})

configure_file(version.h.in version.h)

find_package(Boost COMPONENTS program_options REQUIRED)

set(HEADERS
    src/thread_pool.h
    src/controller.h
    src/mapper.h
    src/reducer.h
    src/splitter.h)

set(SOURCES
    src/thread_pool.cxx
    src/controller.cxx
    src/mapper.cxx
    src/reducer.cxx
    src/splitter.cxx)

add_executable(${PROJECT_NAME}
  src/main.cxx
  ${SOURCES}
  ${HEADERS})

add_executable(tests
  src/tests.cxx
  ${SOURCES}
  ${HEADERS})

include(FetchContent)
FetchContent_Declare(
  googletest
  GIT_REPOSITORY https://github.com/google/googletest.git
  GIT_TAG release-1.12.1
)

set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(googletest)

include(GoogleTest)
gtest_discover_tests(tests)

file(COPY data DESTINATION "${CMAKE_BINARY_DIR}")

set_target_properties(${PROJECT_NAME} tests PROPERTIES
    CXX_STANDARD 17
    CXX_STANDARD_REQUIRED ON
    CXX_EXTENSIONS ON
)

target_include_directories(${PROJECT_NAME}
  PRIVATE "${CMAKE_BINARY_DIR}")

target_link_libraries(${PROJECT_NAME}
  PRIVATE
  pthread
  Boost::program_options)

target_link_libraries(tests
  PRIVATE
  GTest::gtest_main
  pthread)

if(MSVC)
    target_compile_options(${PROJECT_NAME} PRIVATE
        /W4
    )
    target_compile_options(tests PRIVATE
        /W4
    )
else()
    target_compile_options(${PROJECT_NAME} PRIVATE
        -Werror -Wall -Wimplicit-fallthrough -pedantic-errors
    )
    target_compile_options(tests PRIVATE
        -Werror -Wall -Wimplicit-fallthrough -pedantic-errors
    )
endif()


install(TARGETS ${PROJECT_NAME} RUNTIME DESTINATION bin)

set(CPACK_GENERATOR DEB)

set(CPACK_PACKAGE_VERSION_MAJOR "${PROJECT_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${PROJECT_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${PROJECT_VERSION_PATCH}")

set(CPACK_PACKAGE_CONTACT example@example.com)

include(CPack)


enable_testing()
add_test(tests tests)
