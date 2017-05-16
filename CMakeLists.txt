### HEADER
cmake_minimum_required (VERSION 2.8)
project (libelas)

# build tool
find_package(catkin_optional REQUIRED)
co_build_type() # cache the build type

### COMPILER SETUP

# check architecture
execute_process(COMMAND
  dpkg-architecture
    -qDEB_HOST_ARCH
  OUTPUT_VARIABLE
    CMAKE_DEB_HOST_ARCH # Store in CMAKE_DEB_HOST_ARCH
  OUTPUT_STRIP_TRAILING_WHITESPACE
)

# x86 specific flags
# only build if we're on x86
if(${CMAKE_DEB_HOST_ARCH} MATCHES "i386")
    SET(CMAKE_CXX_FLAGS "-msse3")


    ### ENABLE FIND_PACKAGE

    # export the package (for package config)
    co_export(
      INCLUDE_DIRS include
      LIBRARIES ${PROJECT_NAME}
      EXECUTABLES ${PROJECT_NAME}__demo)


    ### TARGETS
    include_directories(include/libelas)
    add_library(${PROJECT_NAME}
      src/elas.cpp
      src/descriptor.cpp
      src/filter.cpp
      src/matrix.cpp
      src/triangle.cpp
    )

    add_executable(${PROJECT_NAME}__demo src/main.cpp)
    target_link_libraries(${PROJECT_NAME}__demo ${PROJECT_NAME})

    ### Set up install
    co_install()

else()
 set(libelas_LIBRARIES "")
 set(libelas_FOUND FALSE)
endif()


