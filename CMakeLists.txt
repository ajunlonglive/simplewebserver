cmake_minimum_required (VERSION 2.8.8)
project (Simple-Web-Server)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -Wall -Wextra")


add_library(simple-web-server INTERFACE)

target_include_directories(simple-web-server INTERFACE ${CMAKE_CURRENT_SOURCE_DIR} .)


target_link_libraries(simple-web-server INTERFACE ${CMAKE_THREAD_LIBS_INIT})

find_package(Threads REQUIRED)

#set(BOOST_COMPONENTS system thread filesystem date_time)
# Late 2017 TODO: remove the following checks and always use std::regex
#if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
#    if (CMAKE_CXX_COMPILER_VERSION VERSION_LESS 4.9)
#        set(BOOST_COMPONENTS ${BOOST_COMPONENTS} regex)
#        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DUSE_BOOST_REGEX")
#    endif()
#endif()
#find_package(Boost 1.70.0 COMPONENTS ${BOOST_COMPONENTS} REQUIRED)
include_directories($ENV{CONDA_PREFIX}/include)
link_directories($ENV{CONDA_PREFIX}/include)
if(APPLE)
    set(OPENSSL_ROOT_DIR "/usr/local/opt/openssl")
endif()

add_executable(http_examples http_examples.cpp)
target_link_libraries(http_examples boost_system boost_thread boost_filesystem boost_date_time)
target_link_libraries(http_examples ${CMAKE_THREAD_LIBS_INIT})

#TODO: add requirement for version 1.0.1g (can it be done in one line?)

if(OPENSSL_FOUND)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DHAVE_OPENSSL")
    target_link_libraries(http_examples ssl)
    include_directories($ENV{CONDA_PREFIX}/include)
    link_directories($ENV{CONDA_PREFIX}/include)
    add_executable(https_examples https_examples.cpp)
    target_link_libraries(https_examples boost_system boost_thread boost_filesystem boost_date_time)
    target_link_libraries(https_examples ssl)
    target_link_libraries(https_examples Threads)
endif()

if(MSYS) #TODO: Is MSYS true when MSVC is true?
    target_link_libraries(http_examples ws2_32 wsock32)
    if(OPENSSL_FOUND)
        target_link_libraries(https_examples ws2_32 wsock32)
    endif()
endif()

enable_testing()
add_subdirectory(tests)

install(FILES server_http.hpp client_http.hpp server_https.hpp client_https.hpp DESTINATION include/simple-web-server)
