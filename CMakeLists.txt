cmake_minimum_required(VERSION 3.15)

# Set default install prefix if not specified
if(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
  set(CMAKE_INSTALL_PREFIX "/usr/local" CACHE PATH "Install path prefix" FORCE)
endif()

set(ENABLE_TESTS ON CACHE BOOL "Enable tests")

set(CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS} -Wall -Wextra")

set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -O0 -ggdb -Wall ")

project(famfs_unit)


include_directories("${PROJECT_SOURCE_DIR}/src")
include_directories("${PROJECT_SOURCE_DIR}/testlib")


##
### Source definitions ###
##

message(STATUS "PROJECT_SOURCE_DIR=${PROJECT_SOURCE_DIR}")
include_directories("${PROJECT_SOURCE_DIR}")

set(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake-modules)
if (CMAKE_BUILD_TYPE STREQUAL "Debug")
  if(CMAKE_COMPILER_IS_GNUCXX AND FAMFS_TEST_COVERAGE)
    message("Compiling for code coverage tests")
    include(CodeCoverage)
    append_coverage_compiler_flags()
  endif()
endif()

add_subdirectory(testlib)

add_library(libfamfs src/famfs_lib.c  )

target_include_directories(libfamfs PUBLIC ${PROJECT_SOURCE_DIR}/linux_include)

add_library(libpcq src/pcq_lib.c  )

target_include_directories(libpcq PUBLIC ${PROJECT_SOURCE_DIR}/linux_include)

add_executable(famfs src/famfs_cli.c )
add_executable(mkfs.famfs src/mkfs.famfs.c )
add_executable(pcq src/pcq.c )

target_link_libraries(famfs libfamfs famfstest uuid z)
target_link_libraries(mkfs.famfs libfamfs uuid z)
target_link_libraries(libfamfs  uuid z)
target_link_libraries(pcq libpcq libfamfs uuid z famfstest)


#
## Test definitions ###
#

if(ENABLE_TESTS)
    configure_file(CMakeLists.txt.in
            googletest-download/CMakeLists.txt)
    execute_process(COMMAND ${CMAKE_COMMAND} -G "${CMAKE_GENERATOR}" .
            WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/googletest-download )
    execute_process(COMMAND ${CMAKE_COMMAND} --build .
            WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/googletest-download )

    add_subdirectory(${CMAKE_BINARY_DIR}/googletest-src
            ${CMAKE_BINARY_DIR}/googletest-build)

    enable_testing()

    add_subdirectory(test)

    include(ExternalProject)

    ExternalProject_Add(multichase
    PREFIX "${CMAKE_CURRENT_BINARY_DIR}"
    GIT_REPOSITORY "https://github.com/jagalactic/multichase.git"
    GIT_TAG dax
    UPDATE_COMMAND ""
    BUILD_IN_SOURCE 1
    CONFIGURE_COMMAND   echo "Empty configure step for jtrace ${CMAKE_CURRENT_BINARY_DIR}"
    BUILD_COMMAND make clean all
    INSTALL_COMMAND ""
    )

    message("multichase: ${CMAKE_CURRENT_BINARY_DIR}")

endif()

#
# Installation
#
install(TARGETS famfs DESTINATION bin)
install(TARGETS mkfs.famfs DESTINATION bin)
install(TARGETS pcq DESTINATION bin)

#Install library and header files? Maybe later...

#install(TARGETS libfamfs DESTINATION lib)
#install(FILES famfs_lib.h DESTINATION include)

#Install man pages? Heck yeah, as soon as we have some...

# You can also install documentation or man pages
#install(FILES README.md DESTINATION share/doc/my_project)
#install(FILES famfs.1 DESTINATION share/man/man1)
