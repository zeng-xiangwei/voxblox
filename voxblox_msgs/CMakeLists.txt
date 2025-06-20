cmake_minimum_required(VERSION 3.0.2)
project(voxblox_msgs)

set(PACKAGE_DEPENDENCIES
  nav_msgs
  std_msgs
)

find_package(catkin REQUIRED COMPONENTS message_generation ${PACKAGE_DEPENDENCIES})

add_message_files(
  DIRECTORY msg
  FILES
    Block.msg
    Layer.msg
    LayerWithTrajectory.msg
    Mesh.msg
    MeshBlock.msg
    MultiMesh.msg
    VoxelEvaluationDetails.msg
)

add_service_files(
  DIRECTORY srv
  FILES
    FilePath.srv
)

generate_messages(
  DEPENDENCIES
    ${PACKAGE_DEPENDENCIES}
)

catkin_package(
  CATKIN_DEPENDS
    ${PACKAGE_DEPENDENCIES}
    message_runtime
)
