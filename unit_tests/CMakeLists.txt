cmake_minimum_required(VERSION 2.8.11 FATAL_ERROR)

add_definitions(-std=c++11 -pthread -v)

enable_testing()

if (CMAKE_VERSION VERSION_LESS 3.2)
    set(UPDATE_DISCONNECTED_IF_AVAILABLE "")
else()
    set(UPDATE_DISCONNECTED_IF_AVAILABLE "UPDATE_DISCONNECTED 1")
endif()

include(${CMAKE_MODULE_PATH}/DownloadProject.cmake)
download_project(PROJ                googletest
                 GIT_REPOSITORY      https://github.com/google/googletest.git
                 GIT_TAG             master
                 ${UPDATE_DISCONNECTED_IF_AVAILABLE}
)

# Prevent GoogleTest from overriding our compiler/linker options
# when building with Visual Studio
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)

add_subdirectory(${googletest_SOURCE_DIR} ${googletest_BINARY_DIR})

# When using CMake 2.8.11 or later, header path dependencies
# are automatically added to the gtest and gmock targets.
# For earlier CMake versions, we have to explicitly add the
# required directories to the header search path ourselves.
if (CMAKE_VERSION VERSION_LESS 2.8.11)
    include_directories("${gtest_SOURCE_DIR}/include"
                        "${gmock_SOURCE_DIR}/include")
endif()


include_directories("../KafkaPluginApp/src/" ".")

include_directories("/opt/epics/modules/areaDetector/ADCore/include" "/opt/epics/base-3.15.5/include" "/opt/epics/base-3.15.5/include/compiler/clang/" "/opt/epics/base-3.15.5/include/os/Darwin/" "/opt/epics/modules/asyn/include/")

link_directories("/opt/local/lib" "/usr/local/lib" "/opt/epics/modules/areaDetector/ADCore/lib/darwin-x86" "/opt/epics/modules/asyn/lib/darwin-x86" "/opt/epics/base-3.15.5/lib/darwin-x86")

add_executable(unit_tests "RunTests.cpp" "KafkaProducerTest.cpp" "../KafkaPluginApp/src/jsoncpp.cpp" "../KafkaPluginApp/src/KafkaProducer.cpp" "../KafkaPluginApp/src/ParamUtility.cpp")

find_library (LIBSTDCXX NAMES stdc++)
add_compile_options(-stdlib=libstdc++)


# set(needed_features cxx_strong_enums cxx_constexpr)
# target_compile_features(LogTests PRIVATE ${needed_features})

target_link_libraries(unit_tests gtest gmock_main NDPlugin ADBase asyn Com rdkafka++ rdkafka pthread z crypto ${LIBSTDCXX} c++)

add_test(TestAll unit_tests)