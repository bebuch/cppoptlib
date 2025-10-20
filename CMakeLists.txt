cmake_minimum_required(VERSION 3.10)
project(cppoptlib VERSION 1.0.3 LANGUAGES CXX)

# Set the C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# Add options for building examples and tests
option(BUILD_TESTING "Build tests" ON)
option(BUILD_EXAMPLES "Build examples" ON)
option(cppoptlib_BUILD_TESTING "Build tests even when used as a sub-project" ON)
option(cppoptlib_BUILD_EXAMPLES "Build examples even when used as a sub-project" ON)

# Find Eigen3
find_package(Eigen3 REQUIRED)
if (TARGET Eigen3::Eigen)
    set(EIGEN_TARGET Eigen3::Eigen)
else()
    set(EIGEN_TARGET Eigen3::Eigen3)
endif()

# GNU standard installation directories
include(GNUInstallDirs)

# Create the header-only library target
add_library(cppoptlib INTERFACE)
target_include_directories(cppoptlib INTERFACE
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>)
target_link_libraries(cppoptlib INTERFACE ${EIGEN_TARGET})

# Add the example
if((CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME OR cppoptlib_BUILD_EXAMPLES) AND BUILD_EXAMPLES)
    add_executable(simple "src/examples/simple.cc")
    target_link_libraries(simple cppoptlib ${EIGEN_TARGET})
endif()

# Add the test
if((CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME OR cppoptlib_BUILD_TESTING) AND BUILD_TESTING)
    enable_testing()
    find_package(GTest REQUIRED)
    add_executable(verify "src/tests/verify.cc")
    target_link_libraries(verify cppoptlib GTest::GTest ${EIGEN_TARGET})
    add_test(NAME verify COMMAND verify)
endif()

# Export the target
install(TARGETS cppoptlib EXPORT cppoptlibTargets)

# Create the cppoptlibConfig.cmake file content
file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/cppoptlibConfig.cmake"
[=[include(CMakeFindDependencyMacro)
find_dependency(Eigen3)
include("${CMAKE_CURRENT_LIST_DIR}/cppoptlibTargets.cmake")
]=])

# Install the header files
install(DIRECTORY "include/" DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}")

# Create and install CMake config files
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/cppoptlibConfigVersion.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY AnyNewerVersion
)

install(EXPORT cppoptlibTargets
    FILE "cppoptlibTargets.cmake"
    NAMESPACE "cppoptlib::"
    DESTINATION "lib/cmake/cppoptlib"
)

install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/cppoptlibConfig.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/cppoptlibConfigVersion.cmake"
    DESTINATION "lib/cmake/cppoptlib"
)

