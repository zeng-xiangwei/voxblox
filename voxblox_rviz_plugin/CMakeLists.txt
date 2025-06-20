cmake_minimum_required(VERSION 2.8.3)
project(voxblox_rviz_plugin)

set(CMAKE_MACOSX_RPATH 0)
add_definitions(-std=c++11 -Wall)

## This setting causes Qt's "MOC" generation to happen automatically.
set(CMAKE_AUTOMOC ON)

find_package(catkin REQUIRED COMPONENTS
  rviz
  voxblox
  voxblox_msgs
)
catkin_package()

## This plugin includes Qt widgets, so we must include Qt.
## We'll use the version that rviz used so they are compatible.
if(rviz_QT_VERSION VERSION_LESS "5")
  message(STATUS "Using Qt4 based on the rviz_QT_VERSION: ${rviz_QT_VERSION}")
  find_package(Qt4 ${rviz_QT_VERSION} EXACT REQUIRED QtCore QtGui)
  ## pull in all required include dirs, define QT_LIBRARIES, etc.
  include(${QT_USE_FILE})
  qt4_wrap_cpp(MOC_FILES
    ${INCLUDE_FILES_QT}
  )
else()
  message(STATUS "Using Qt5 based on the rviz_QT_VERSION: ${rviz_QT_VERSION}")
  find_package(Qt5 ${rviz_QT_VERSION} EXACT REQUIRED Core Widgets)
  ## make target_link_libraries(${QT_LIBRARIES}) pull in all required dependencies
  set(QT_LIBRARIES Qt5::Widgets)
  QT5_WRAP_CPP(MOC_FILES
    ${INCLUDE_FILES_QT}
  )
endif()

## Avoid Qt signals and slots defining "emit", "slots", etc.
add_definitions(-DQT_NO_KEYWORDS)

set(HEADER_FILES
        include/voxblox_rviz_plugin/voxblox_mesh_display.h
        include/voxblox_rviz_plugin/voxblox_multi_mesh_display.h
        include/voxblox_rviz_plugin/voxblox_mesh_visual.h
        include/voxblox_rviz_plugin/material_loader.h
        )

set(SRC_FILES
        src/voxblox_mesh_display.cc
        src/voxblox_multi_mesh_display.cc
        src/voxblox_mesh_visual.cc
        src/material_loader.cc
        )

add_library(${PROJECT_NAME}
  ${SRC_FILES}
  ${HEADER_FILES}
  ${MOC_FILES}
)

target_include_directories(${PROJECT_NAME} PUBLIC
    ${catkin_INCLUDE_DIRS}
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)
target_link_libraries(${PROJECT_NAME}
  ${QT_LIBRARIES}
  ${catkin_LIBRARIES}
)


##########
# EXPORT #
##########

install(
  FILES plugin_description.xml
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION})

install(
  DIRECTORY icons/
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/icons)

set(RUNTIME_DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})
install(TARGETS ${PROJECT_NAME}
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${RUNTIME_DESTINATION}
)
