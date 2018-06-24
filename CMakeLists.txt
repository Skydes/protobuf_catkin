cmake_minimum_required(VERSION 2.8.3)
project(protobuf_catkin)

find_package(catkin_simple REQUIRED)
catkin_simple()

include(ExternalProject)

file(MAKE_DIRECTORY ${CATKIN_DEVEL_PREFIX}/include)

#ExternalProject_Add(protobuf_src
  #URL https://github.com/google/protobuf/releases/download/v2.6.1/protobuf-2.6.1.tar.gz
  #UPDATE_COMMAND ""
  #PATCH_COMMAND patch -p0 < ${CMAKE_SOURCE_DIR}/unused_parameter_warnings.patch &&
                #patch -p0 < ${CMAKE_SOURCE_DIR}/unused_parameter_warnings_wire_format.patch
  #CONFIGURE_COMMAND cd ../protobuf_src && ./autogen.sh && ./configure --with-pic --prefix=${CATKIN_DEVEL_PREFIX}
  #BUILD_COMMAND cd ../protobuf_src && make -j8
  #INSTALL_COMMAND cd ../protobuf_src && make install -j8
#)

set(PROTOBUF_INCLUDE_DIRS ${CMAKE_CURRENT_BINARY_DIR}/protobuf/src/protobuf/src)
set(PROTOBUF_URL https://github.com/google/protobuf.git)
set(PROTOBUF_TAG v3.6.0)
ExternalProject_Add(protobuf_src
    PREFIX protobuf
    GIT_REPOSITORY ${PROTOBUF_URL}
    GIT_TAG ${PROTOBUF_TAG}
    BUILD_IN_SOURCE 1
    SOURCE_DIR ${CMAKE_CURRENT_BINARY_DIR}/protobuf/src/protobuf
    CONFIGURE_COMMAND ${CMAKE_COMMAND} cmake/
        -DCMAKE_BUILD_TYPE:STRING=Release
        -Dprotobuf_BUILD_TESTS:BOOL=OFF
        -DCMAKE_INSTALL_PREFIX=${CATKIN_DEVEL_PREFIX}
        -DCMAKE_INSTALL_LIBDIR=lib
        -DBUILD_SHARED_LIBS=ON
    CMAKE_CACHE_ARGS
        -DCMAKE_BUILD_TYPE:STRING=Release
        -Dprotobuf_BUILD_TESTS:BOOL=OFF
        -DCMAKE_INSTALL_PREFIX=${CATKIN_DEVEL_PREFIX}
        -DCMAKE_INSTALL_LIBDIR=lib
        -DBUILD_SHARED_LIBS=ON
)

cs_add_library(${PROJECT_NAME} src/dependency_tracker.cc)
add_dependencies(${PROJECT_NAME} protobuf_src)
target_link_libraries(${PROJECT_NAME}
  ${CATKIN_DEVEL_PREFIX}/lib/libprotobuf-lite${CMAKE_SHARED_LIBRARY_SUFFIX}
  ${CATKIN_DEVEL_PREFIX}/lib/libprotoc${CMAKE_SHARED_LIBRARY_SUFFIX}
  ${CATKIN_DEVEL_PREFIX}/lib/libprotobuf${CMAKE_SHARED_LIBRARY_SUFFIX})

cs_install()

cs_export(
  INCLUDE_DIRS ${CATKIN_DEVEL_PREFIX}/include
  CFG_EXTRAS protobuf-generate-cpp.cmake protobuf-extras.cmake)
