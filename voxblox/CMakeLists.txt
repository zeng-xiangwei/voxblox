cmake_minimum_required(VERSION 3.0.2)
project(voxblox)

set(CMAKE_MACOSX_RPATH 0)
add_definitions(-std=c++14 -Wall -Wextra)

find_package(minkindr REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(Protobuf REQUIRED)
find_package(glog REQUIRED)

find_package(catkin REQUIRED)
catkin_package()

############
# PROTOBUF #
############
# General idea: first check if we have protobuf catkin, then use that.
# Otherwise use system protobuf.
include(${PROJECT_SOURCE_DIR}/cmake/functions.cmake)
set(PROTO_DEFNS proto/voxblox/Block.proto
                proto/voxblox/Layer.proto)
set(BASE_PATH "proto")
set(PROTOBUF_COMPILE_PYTHON true)

PROTOBUF_CATKIN_GENERATE_CPP2(${BASE_PATH} PROTO_SRCS PROTO_HDRS ${PROTO_DEFNS})

####################
# SET SOURCE FILES #
####################

set("${PROJECT_NAME}_SRCS"
  src/alignment/icp.cc
  src/core/block.cc
  src/core/esdf_map.cc
  src/core/tsdf_map.cc
  src/integrator/esdf_integrator.cc
  src/integrator/esdf_occ_integrator.cc
  src/integrator/integrator_utils.cc
  src/integrator/intensity_integrator.cc
  src/integrator/tsdf_integrator.cc
  src/io/mesh_ply.cc
  src/io/sdf_ply.cc
  src/mesh/marching_cubes.cc
  src/simulation/objects.cc
  src/simulation/simulation_world.cc
  src/utils/camera_model.cc
  src/utils/evaluation_utils.cc
  src/utils/layer_utils.cc
  src/utils/neighbor_tools.cc
  src/utils/protobuf_utils.cc
  src/utils/timing.cc
  src/utils/voxel_utils.cc
)

#############
# LIBRARIES #
#############
add_library(${PROJECT_NAME}_proto
  ${PROTO_SRCS}
)
target_include_directories(${PROJECT_NAME}_proto SYSTEM PUBLIC
  ${PROTOBUF_INCLUDE_DIR})
target_link_libraries(${PROJECT_NAME}_proto 
  ${PROTOBUF_LIBRARIES}
)

add_library(${PROJECT_NAME}
  ${${PROJECT_NAME}_SRCS}
)
target_include_directories(${PROJECT_NAME} PUBLIC 
  ${catkin_INCLUDE_DIRS}
  ${EIGEN3_INCLUDE_DIR}
  ${minkindr_INCLUDE_DIRS}
  ${PROJECT_SOURCE_DIR}/include
)
target_link_libraries(${PROJECT_NAME} 
  ${PROJECT_NAME}_proto
  ${catkin_LIBRARIES}
  ${minkindr_LIBRARIES}
  glog::glog
)

############
# BINARIES #
############

add_executable(tsdf_to_esdf
  test/tsdf_to_esdf.cc
)
target_link_libraries(tsdf_to_esdf ${PROJECT_NAME})

add_executable(test_load_esdf
  test/test_load_esdf.cc
)
target_link_libraries(test_load_esdf ${PROJECT_NAME})

#########
# TESTS #
#########
add_custom_target(test_data)
add_custom_command(TARGET test_data
                  COMMAND rm -rf test_data
                  COMMAND mkdir -p test_data
                  COMMAND cp -r ${CMAKE_SOURCE_DIR}/test/test_data/*
                                test_data/ || :)

#add_definitions(-DVISUALIZE_UNIT_TEST_RESULTS)

catkin_add_gtest(test_approx_hash_array
  test/test_approx_hash_array.cc
)
target_link_libraries(test_approx_hash_array ${PROJECT_NAME})

catkin_add_gtest(test_tsdf_map
  test/test_tsdf_map.cc
)
target_link_libraries(test_tsdf_map ${PROJECT_NAME})

catkin_add_gtest(test_protobuf
  test/test_protobuf.cc
)
target_link_libraries(test_protobuf ${PROJECT_NAME})

catkin_add_gtest(test_tsdf_interpolator
  test/test_tsdf_interpolator.cc
)
target_link_libraries(test_tsdf_interpolator ${PROJECT_NAME})

catkin_add_gtest(test_layer
  test/test_layer.cc
)
target_link_libraries(test_layer ${PROJECT_NAME})

catkin_add_gtest(test_merge_integration
  test/test_merge_integration.cc
)
target_link_libraries(test_merge_integration ${PROJECT_NAME})

catkin_add_gtest(test_layer_utils
  test/test_layer_utils.cc
)
target_link_libraries(test_layer_utils ${PROJECT_NAME})

catkin_add_gtest(test_sdf_integrators
  test/test_sdf_integrators.cc
)
target_link_libraries(test_sdf_integrators ${PROJECT_NAME})

catkin_add_gtest(test_bucket_queue
  test/test_bucket_queue.cc
)
target_link_libraries(test_bucket_queue ${PROJECT_NAME})

catkin_add_gtest(test_clear_spheres
  test/test_clear_spheres.cc
)
target_link_libraries(test_clear_spheres ${PROJECT_NAME})

##########
# EXPORT #
##########
# Library
set(${PROJECT_NAME}_TARGETS)
list(APPEND ${PROJECT_NAME}_TARGETS ${PROJECT_NAME})
list(APPEND ${PROJECT_NAME}_TARGETS ${PROJECT_NAME}_proto)
install(TARGETS ${${PROJECT_NAME}_TARGETS}
    ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
    LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
    RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

# Headers
set(${PROJECT_NAME}_LOCAL_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/include)
install(DIRECTORY ${PROJECT_NAME}_LOCAL_INCLUDE_DIR
    DESTINATION ${CATKIN_GLOBAL_INCLUDE_DESTINATION}
    FILES_MATCHING PATTERN "*.h" PATTERN "*.hpp"
    PATTERN ".svn" EXCLUDE
)

# Dependencies
set(${PROJECT_NAME}_RUN_DEPENDS)
list(APPEND ${PROJECT_NAME}_RUN_DEPENDS minkindr)

set(${PROJECT_NAME}_LIBRARIES)
list(APPEND ${PROJECT_NAME}_LIBRARIES ${PROJECT_NAME} ${PROJECT_NAME}_proto)
catkin_package(
    INCLUDE_DIRS ${${PROJECT_NAME}_LOCAL_INCLUDE_DIR}
    LIBRARIES ${${PROJECT_NAME}_LIBRARIES}
    CATKIN_DEPENDS ${${PROJECT_NAME}_RUN_DEPENDS}
)
