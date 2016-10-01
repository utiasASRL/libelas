### HEADER
cmake_minimum_required (VERSION 2.8)
project (libelas)

# use sse3 instruction set
SET(CMAKE_CXX_FLAGS "-msse3")

### CATKIN
set(CATKIN_ENABLED "TRUE" CACHE BOOL "Catkin will export package information (for use with ROS)")
if (CATKIN_ENABLED)
  find_package(catkin REQUIRED)
  catkin_package()
endif()

include_directories(src)

### TARGETS
add_library(${PROJECT_NAME}
  src/elas.cpp
  src/descriptor.cpp
  src/filter.cpp
  src/matrix.cpp
  src/triangle.cpp
)
add_dependencies(${PROJECT_NAME} ${catkin_EXPORTED_TARGETS})

add_executable(${PROJECT_NAME}__demo
  src/main.cpp
)
target_link_libraries(${PROJECT_NAME}__demo
  ${PROJECT_NAME}
)
add_dependencies(${PROJECT_NAME}__demo ${catkin_EXPORTED_TARGETS})

######################################################################################
### Set up install
######################################################################################

#setup installation locations
set(INSTALL_INCLUDE_DIR include/${PROJECT_NAME} CACHE PATH "Installation directory for header files")
set(INSTALL_BIN_DIR lib/${PROJECT_NAME} CACHE PATH "Installation directory for compiled binaries")
set(INSTALL_CATKIN_DIR share/${PROJECT_NAME} CACHE PATH "Installation directory for catkin package.xml")
if(WIN32 AND NOT CYGWIN)
  set(DEF_INSTALL_CMAKE_DIR CMake)
else()
  set(DEF_INSTALL_CMAKE_DIR lib/cmake/${PROJECT_NAME})
endif()
set(INSTALL_CMAKE_DIR ${DEF_INSTALL_CMAKE_DIR} CACHE PATH "Installation directory for CMake files")

# Make relative paths absolute (needed later on)
foreach(p INCLUDE BIN CMAKE CATKIN)
    set(var INSTALL_${p}_DIR)
    if(NOT IS_ABSOLUTE "${${var}}")
        set(${var} "${CMAKE_INSTALL_PREFIX}/${${var}}")
    endif()
endforeach()

install(
  FILES
    src/elas.h
    src/descriptor.h
    src/filter.h
    src/matrix.h
    src/triangle.h
    src/image.h
    src/timer.h
  DESTINATION ${INSTALL_INCLUDE_DIR}
)

install(
  TARGETS
    ${PROJECT_NAME}
    ${PROJECT_NAME}__demo
  DESTINATION
    ${INSTALL_BIN_DIR}
)

######################################################################################
### Enable find_package()
######################################################################################

### Create the <package>Config.cmake and <package>ConfigVersion files

# Export the package for use from the build-tree
export(PACKAGE ${PROJECT_NAME})

# Create variable for the local build tree
get_property(CONF_INCLUDE_DIRS DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY INCLUDE_DIRECTORIES)

# Change the include location for the case of an install location
set(CONF_INCLUDE_DIRS ${CONF_INCLUDE_DIRS} ${INSTALL_INCLUDE_DIR})

# Create the <name>Config.cmake for the installation tree
message(STATUS "### ${INSTALL_INCLUDE_DIR}")
message(STATUS "### ${INSTALL_BIN_DIR}")
configure_file(${PROJECT_NAME}Config.cmake.in
  "${PROJECT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/${PROJECT_NAME}Config.cmake" @ONLY)
configure_file(${PROJECT_NAME}Config.cmake.in
  "${CATKIN_DEVEL_PREFIX}/share/${PROJECT_NAME}/cmake/${PROJECT_NAME}Config.cmake" @ONLY)

# Install the <package>Config.cmake
install(FILES
  "${PROJECT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/${PROJECT_NAME}Config.cmake"
  DESTINATION "${INSTALL_CMAKE_DIR}" COMPONENT dev)

add_custom_target(uninstall
  COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)

# Install Catkin package.xml
install(FILES package.xml DESTINATION ${INSTALL_CATKIN_DIR})