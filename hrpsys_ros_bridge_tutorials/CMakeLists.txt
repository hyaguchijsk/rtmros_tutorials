if(NOT USE_ROSBUILD)
  include(catkin.cmake)
  return()
endif()


cmake_minimum_required(VERSION 2.4.6)
include($ENV{ROS_ROOT}/core/rosbuild/rosbuild.cmake)
rosbuild_find_ros_package(hrpsys_ros_bridge)
include(${hrpsys_ros_bridge_PACKAGE_PATH}/cmake/compile_robot_model.cmake)

rosbuild_init()

# Set the build type.  Options are:
#  Coverage       : w/ debug symbols, w/o optimization, w/ code-coverage
#  Debug          : w/ debug symbols, w/o optimization
#  Release        : w/o debug symbols, w/ optimization
#  RelWithDebInfo : w/ debug symbols, w/ optimization
#  MinSizeRel     : w/o debug symbols, w/ optimization, stripped binaries
#set(ROS_BUILD_TYPE RelWithDebInfo)

#set the default path for built executables to the "bin" directory
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)
#set the default path for built libraries to the "lib" directory
set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)

#uncomment if you have defined services
#rosbuild_gensrv()
#rtmbuild_genidl()

#common commands for building c++ executables and libraries
#rosbuild_add_library(${PROJECT_NAME} src/example.cpp)
#target_link_libraries(${PROJECT_NAME} another_library)
#rosbuild_add_boost_directories()
#rosbuild_link_boost(${PROJECT_NAME} thread)
#rosbuild_add_executable(example examples/example.cpp)
#target_link_libraries(example ${PROJECT_NAME})

##
## wrl,dae models
##
set(wrl_robots)
rosbuild_find_ros_package(openhrp3)
rosbuild_find_ros_package(hrpsys)
rosbuild_find_ros_package(collada_robots)
rosbuild_find_ros_package(choreonoid)
rosbuild_find_ros_package(webots_simulator)

###
### convert model for disclosed models
###
if(EXISTS ${hrpsys_PACKAGE_PATH}/share/hrpsys/samples/HRP4C/HRP4Cmain.wrl)
  compile_openhrp_model(
    ${hrpsys_PACKAGE_PATH}/share/hrpsys/samples/HRP4C/HRP4Cmain.wrl
    HRP4C
    -a rightarm_torso,BODY,R_WRIST_R_LINK,0,0,0,0.707,0,0.707,0 -a leftarm_torso,BODY,L_WRIST_R_LINK,0,0,0,0.707,0,0.707,0 -a rightarm,BODY,CHEST_Y_LINK,0,0,0,0.707,0,0.707,0 -a leftarm,CHEST_Y_LINK,L_WRIST_R_LINK,0,0,0,0.707,0,0.707,0
    --conf-file-option "virtual_force_sensor: vlhsensor, CHEST_Y, L_HAND_J0, 0,0,0, 0,0,1,0, vrhsensor, CHEST_Y, R_HAND_J0, 0,0,0, 0,0,1,0"
    --conf-file-option "abc_leg_offset: 0.0, 0.06845, 0.0"
  --robothardware-conf-file-option "pdgains.file_name: ${PROJECT_SOURCE_DIR}/models/PDgains.sav"
)
endif(EXISTS ${hrpsys_PACKAGE_PATH}/share/hrpsys/samples/HRP4C/HRP4Cmain.wrl)

compile_openhrp_model(
  ${openhrp3_PACKAGE_PATH}/share/OpenHRP-3.1/sample/model/PA10/pa10.main.wrl)
compile_openhrp_model(
  ${openhrp3_PACKAGE_PATH}/share/OpenHRP-3.1/sample/model/sample1.wrl SampleRobot
  --conf-file-option "abc_leg_offset: 0,0.09,0"
  --conf-file-option "abc_stride_parameter: 0.15,0.05,10"
  --conf-file-option "end_effectors: lleg,LLEG_ANKLE_R,WAIST,0.0,0.0,-0.07,0.0,0.0,0.0,0.0, rleg,RLEG_ANKLE_R,WAIST,0.0,0.0,-0.07,0.0,0.0,0.0,0.0, larm,LARM_WRIST_P,CHEST,0.0,0,-0.12,0,1.0,0.0,1.5708, rarm,RARM_WRIST_P,CHEST,0.0,0,-0.12,0,1.0,0.0,1.5708,"
  --robothardware-conf-file-option "pdgains.file_name: ${PROJECT_SOURCE_DIR}/models/PDgains.sav"
)

compile_collada_model(
  ${collada_robots_PACKAGE_PATH}/data/robots/kawada-hironx.dae
  --proj-file-root-option "0,0,0.5,0,0,1,0"
  )
compile_collada_model(${collada_robots_PACKAGE_PATH}/data/robots/willowgarage-pr2.dae)

# generate webots model
add_custom_command(OUTPUT ${webots_simulator_PACKAGE_PATH}/build/webots/resources/projects/robots/darwin-op/protos/DARwInOP.proto
  COMMAND make -C ${webots_simulator_PACKAGE_PATH})
add_custom_command(OUTPUT ${PROJECT_SOURCE_DIR}/models/DARWIN.wrl
  COMMAND rosrun roseus roseus ${PROJECT_SOURCE_DIR}/euslisp/webotsvrml2openhrp3vrml.l
  "\\(progn \\(convert-webots-vrml-\\>openhrp3-vrml-for-darwin \\\"${PROJECT_SOURCE_DIR}/models/DARWIN.wrl\\\"\\) \\(exit\\)\\)"
  DEPENDS ${webots_simulator_PACKAGE_PATH}/build/webots/resources/projects/robots/darwin-op/protos/DARwInOP.proto)
if (EXISTS ${webots_simulator_PACKAGE_PATH}/build/webots/resources/projects/robots/darwin-op/protos/DARwInOP.proto)
  compile_openhrp_model(${PROJECT_SOURCE_DIR}/models/DARWIN.wrl)
else()
  message("\n\n\n\n${webots_simulator_PACKAGE_PATH}/build/webots/resources/projects/robots/darwin-op/protos/DARwInOP.proto is not found..\n\n\n\n")
endif()

# yaskawa model
rosbuild_download_data(http://openhrp-aist-grx.googlecode.com/files/YaskawaSmartPal5.zip build/YaskawaSmartPal5.zip 3db916d5dc5ef4259c7dfc1ac85dd380)
add_custom_command(
  OUTPUT ${PROJECT_SOURCE_DIR}/models/YaskawaSmartPal5.wrl
  COMMAND unzip -o ${PROJECT_SOURCE_DIR}/build/YaskawaSmartPal5.zip
  COMMAND cp    -rf ${PROJECT_SOURCE_DIR}/build/YaskawaSmartPal5/* ${PROJECT_SOURCE_DIR}/models/
  DEPENDS ${PROJECT_SOURCE_DIR}/build/YaskawaSmartPal5.zip)
compile_openhrp_model(${PROJECT_SOURCE_DIR}/models/YaskawaSmartPal5.wrl
  YaskawaSmartPal5
  -a leftarm,WAIST_LINK2,LARM_LINK7,0,0,0,0.707,0,0.707,0,LARM_JOINTG,-1 -a leftarm_waist,WAIST_LINK0,LARM_LINK7,0,0,0,0.707,0,0.707,0,LARM_JOINTG,-1 -a rightarm,WAIST_LINK2,RARM_LINK7,0,0,0,0.707,0,0.707,0,RARM_JOINTG,-1 -a rightarm_waist,WAIST_LINK0,RARM_LINK7,0,0,0,0.707,0,0.707,0,RARM_JOINTG,-1)

# choreonoid model
if (EXISTS ${choreonoid_PACKAGE_PATH}/share/choreonoid-1.3/model/GR001/GR001.wrl)
  compile_openhrp_model(${choreonoid_PACKAGE_PATH}/share/choreonoid-1.3/model/GR001/GR001.wrl)
else()
  message("\n\n\n\n ${choreonoid_PACKAGE_PATH}/share/choreonoid-1.3/model/GR001/GR001.wrl is not found..\n\n\n\n")
endif()

###
### convert model for closed models
###
macro(compile_model_for_closed_robots _robot_wrl_file _OpenHRP2_robot_name)
  if(EXISTS ${_robot_wrl_file})
    compile_openhrp_model(
      ${_robot_wrl_file}
      ${_OpenHRP2_robot_name}
      ${ARGN})
  else()
    message("\n\n\n\n ${_robot_wrl_file} is not found..\n\n\n\n")
  endif()
endmacro()
macro(compile_openhrp_model_for_closed_robots _OpenHRP2_robot_vrml_name _OpenHRP2_robot_dir _OpenHRP2_robot_name)
  compile_model_for_closed_robots(
    $ENV{CVSDIR}/OpenHRP/etc/${_OpenHRP2_robot_dir}/${_OpenHRP2_robot_vrml_name}main.wrl
    ${_OpenHRP2_robot_name}
    ${ARGN})
endmacro()
macro(gen_minmax_table_for_closed_robots _OpenHRP2_robot_vrml_name _OpenHRP2_robot_dir _OpenHRP2_robot_name)
  if (EXISTS $ENV{CVSDIR}/OpenHRP/etc/${_OpenHRP2_robot_dir}/${_OpenHRP2_robot_vrml_name}main.wrl)
    string(TOLOWER ${_OpenHRP2_robot_name} _sname)
    set(_workdir ${PROJECT_SOURCE_DIR}/models)
    set(_gen_jointmm_command_arg "\"\\(write-min-max-table \\(${_sname}\\) \\\"${_workdir}/${_sname}.l\\\" :margin 1.0\\)\"")
    add_custom_command(OUTPUT ${PROJECT_SOURCE_DIR}/build/${_sname}_joint_minmax_done
      COMMAND rosrun roseus roseus ${PROJECT_SOURCE_DIR}/euslisp/make-joint-min-max-table.l ${_workdir}/${_sname}.l "\"${_gen_jointmm_command_arg}\"" "\"(exit)\"" && touch ${PROJECT_SOURCE_DIR}/build/${_sname}_joint_minmax_done
      DEPENDS ${_workdir}/${_sname}.l)
    add_custom_target(${_sname}_${PROJECT_NAME}_compile2 ALL DEPENDS ${PROJECT_SOURCE_DIR}/build/${_sname}_joint_minmax_done ${_sname}_${PROJECT_NAME}_compile)
  endif()
endmacro()

# old HRP2xx.wrl files should be coverted.
compile_openhrp_model_for_closed_robots(HRP2JSK HRP2JSK_for_OpenHRP3 HRP2JSK
  --conf-file-option "abc_leg_offset: 0.0,0.105,0.0"
  --conf-file-option "end_effectors: rleg,RLEG_JOINT5,WAIST,0.0,-0.01,-0.105,0.0,0.0,0.0,0.0, lleg,LLEG_JOINT5,WAIST,0.0,0.01,-0.105,0.0,0.0,0.0,0.0, rarm,RARM_JOINT6,CHEST_JOINT1,0.0,0.0169,-0.174,0.0,1.0,0.0,1.5708, larm,LARM_JOINT6,CHEST_JOINT1,0.0,-0.0169,-0.174,0.0,1.0,0.0,1.5708,"
  --robothardware-conf-file-option "pdgains.file_name: ${PROJECT_SOURCE_DIR}/models/PDgains.sav"
  )
gen_minmax_table_for_closed_robots(HRP2JSK HRP2JSK_for_OpenHRP3 HRP2JSK)
compile_openhrp_model_for_closed_robots(HRP2JSKNT HRP2JSKNT_for_OpenHRP3 HRP2JSKNT
  --conf-file-option "abc_leg_offset: 0.0,0.105,0.0"
  --conf-file-option "end_effectors: rleg,RLEG_JOINT5,WAIST,0.035589,-0.01,-0.105,0.0,0.0,0.0,0.0, lleg,LLEG_JOINT5,WAIST,0.035589,0.01,-0.105,0.0,0.0,0.0,0.0, rarm,RARM_JOINT6,CHEST_JOINT1,-0.0042,0.0392,-0.1245,0.0,1.0,0.0,1.5708, larm,LARM_JOINT6,CHEST_JOINT1,-0.0042,-0.0392,-0.1245,0.0,1.0,0.0,1.5708,"
  --robothardware-conf-file-option "pdgains.file_name: ${PROJECT_SOURCE_DIR}/models/PDgains.sav"
  )
gen_minmax_table_for_closed_robots(HRP2JSKNT HRP2JSKNT_for_OpenHRP3 HRP2JSKNT)
compile_openhrp_model_for_closed_robots(HRP2JSKNTS HRP2JSKNTS_for_OpenHRP3 HRP2JSKNTS
  --conf-file-option "abc_leg_offset: 0.0,0.105,0.0"
  --conf-file-option "end_effectors: rleg,RLEG_JOINT5,WAIST,0.035589,-0.01,-0.105,0.0,0.0,0.0,0.0, lleg,LLEG_JOINT5,WAIST,0.035589,0.01,-0.105,0.0,0.0,0.0,0.0, rarm,RARM_JOINT6,CHEST_JOINT1,-0.0042,0.0392,-0.1245,0.0,1.0,0.0,1.5708, larm,LARM_JOINT6,CHEST_JOINT1,-0.0042,-0.0392,-0.1245,0.0,1.0,0.0,1.5708,"
  --robothardware-conf-file-option "pdgains.file_name: ${PROJECT_SOURCE_DIR}/models/PDgains.sav"
  )
gen_minmax_table_for_closed_robots(HRP2JSKNTS HRP2JSKNTS_for_OpenHRP3 HRP2JSKNTS)
compile_openhrp_model_for_closed_robots(HRP2W HRP2W_for_OpenHRP3 HRP2W
  --conf-file-option "end_effectors: rarm,RARM_JOINT6,CHEST_JOINT1,-0.0042,0.0392,-0.1245,0.0,1.0,0.0,1.5708, larm,LARM_JOINT6,CHEST_JOINT1,-0.0042,-0.0392,-0.1245,0.0,1.0,0.0,1.5708,"
  --robothardware-conf-file-option "pdgains.file_name: ${PROJECT_SOURCE_DIR}/models/PDgains.sav"
  )
compile_openhrp_model_for_closed_robots(HRP2JSKNT HRP2JSKNT_WITH_3HAND HRP2JSKNT_WITH_3HAND
 -a leftarm,CHEST_LINK1,LARM_LINK6,-0.0042,-0.0392,-0.1245,-3.373247e-18,1.0,9.813081e-18,1.5708,L_THUMBCM_Y,0,L_THUMBCM_P,1,L_INDEXMP_R,0,L_INDEXMP_P,0,L_INDEXPIP_R,-1,L_MIDDLEPIP_R,-1
 -a leftarm_torso,BODY,LARM_LINK6,-0.0042,-0.0392,-0.1245,-3.373247e-18,1.0,9.813081e-18,1.5708,L_THUMBCM_Y,0,L_THUMBCM_P,1,L_INDEXMP_R,0,L_INDEXMP_P,0,L_INDEXPIP_R,-1,L_MIDDLEPIP_R,-1
 -a leftarm_grasp,CHEST_LINK1,LARM_LINK6,0.0,-0.03,-0.17,1.0,0.0,0.0,2.0944,L_THUMBCM_Y,0,L_THUMBCM_P,1,L_INDEXMP_R,0,L_INDEXMP_P,0,L_INDEXPIP_R,-1,L_MIDDLEPIP_R,-1
 -a rightarm,CHEST_LINK1,RARM_LINK6,-0.0042,0.0392,-0.1245,3.373247e-18,1.0,-9.813081e-18,1.5708,R_THUMBCM_Y,0,R_THUMBCM_P,1,R_INDEXMP_R,0,R_INDEXMP_P,0,R_INDEXPIP_R,1,R_MIDDLEPIP_R,1
 -a rightarm_torso,BODY,RARM_LINK6,-0.0042,0.0392,-0.1245,3.373247e-18,1.0,-9.813081e-18,1.5708,R_THUMBCM_Y,0,R_THUMBCM_P,1,R_INDEXMP_R,0,R_INDEXMP_P,0,R_INDEXPIP_R,1,R_MIDDLEPIP_R,1
 -a rightarm_grasp,CHEST_LINK1,RARM_LINK6,0.0,0.03,-0.17,-1.0,0.0,0.0,2.0944,R_THUMBCM_Y,0,R_THUMBCM_P,1,R_INDEXMP_R,0,R_INDEXMP_P,0,R_INDEXPIP_R,1,R_MIDDLEPIP_R,1
  )
compile_openhrp_model_for_closed_robots(HRP2JSKNTS HRP2JSKNTS_WITH_3HAND HRP2JSKNTS_WITH_3HAND
  -a leftarm,CHEST_LINK1,LARM_LINK6,-0.0042,-0.0392,-0.1245,-3.373247e-18,1.0,9.813081e-18,1.5708,L_THUMBCM_Y,0,L_THUMBCM_P,1,L_INDEXMP_R,0,L_INDEXMP_P,0,L_INDEXPIP_R,-1,L_MIDDLEPIP_R,-1
 -a leftarm_torso,BODY,LARM_LINK6,-0.0042,-0.0392,-0.1245,-3.373247e-18,1.0,9.813081e-18,1.5708,L_THUMBCM_Y,0,L_THUMBCM_P,1,L_INDEXMP_R,0,L_INDEXMP_P,0,L_INDEXPIP_R,-1,L_MIDDLEPIP_R,-1
 -a leftarm_grasp,CHEST_LINK1,LARM_LINK6,0.0,-0.03,-0.17,1.0,0.0,0.0,2.0944,L_THUMBCM_Y,0,L_THUMBCM_P,1,L_INDEXMP_R,0,L_INDEXMP_P,0,L_INDEXPIP_R,-1,L_MIDDLEPIP_R,-1
 -a rightarm,CHEST_LINK1,RARM_LINK6,-0.0042,0.0392,-0.1245,3.373247e-18,1.0,-9.813081e-18,1.5708,R_THUMBCM_Y,0,R_THUMBCM_P,1,R_INDEXMP_R,0,R_INDEXMP_P,0,R_INDEXPIP_R,1,R_MIDDLEPIP_R,1
 -a rightarm_torso,BODY,RARM_LINK6,-0.0042,0.0392,-0.1245,3.373247e-18,1.0,-9.813081e-18,1.5708,R_THUMBCM_Y,0,R_THUMBCM_P,1,R_INDEXMP_R,0,R_INDEXMP_P,0,R_INDEXPIP_R,1,R_MIDDLEPIP_R,1
 -a rightarm_grasp,CHEST_LINK1,RARM_LINK6,0.0,0.03,-0.17,-1.0,0.0,0.0,2.0944,R_THUMBCM_Y,0,R_THUMBCM_P,1,R_INDEXMP_R,0,R_INDEXMP_P,0,R_INDEXPIP_R,1,R_MIDDLEPIP_R,1
  )
compile_openhrp_model_for_closed_robots(HRP4R HRP4R HRP4R
  -a leftarm,L_SHOULDER_P_LINK,L_WRIST_R_LINK,0,0,0,0,0,0,1,L_HAND_J0,-1,L_HAND_J1,-1
  -a rightarm,R_SHOULDER_P_LINK,R_WRIST_R_LINK,0,0,0,0,0,0,1,R_HAND_J0,1,R_HAND_J1,1
  --conf-file-option "virtual_force_sensor: vlhsensor, CHEST_Y, L_WRIST_R, 0,0,0, 0,0,1,0, vrhsensor, CHEST_Y, R_WRIST_R, 0,0,0, 0,0,1,0, vlfsensor, WAIST, L_ANKLE_R, 0,0,0, 0,0,1,0, vrfsensor, WAIST, R_ANKLE_R, 0,0,0, 0,0,1,0"
  --conf-file-option "abc_leg_offset: 0.0, 0.079919, 0.0"
  --conf-file-option "end_effectors: rarm,R_WRIST_R,CHEST_Y,0.0,0.0,-0.1,-1.471962e-17,1.0,-1.471962e-17,1.5708, larm,L_WRIST_R,CHEST_Y,0.0,0.0,-0.1,1.471962e-17,1.0,1.471962e-17,1.5708, rleg,R_ANKLE_R,WAIST,0.0,0.0,-0.091849,0.0,0.0,0.0,0.0, lleg,L_ANKLE_R,WAIST,0.0,0.0,-0.091849,0.0,0.0,0.0,0.0,"
  )
compile_model_for_closed_robots($ENV{CVSDIR}/euslib/rbrain/urataleg/URATALEGmain.wrl URATALEG
  --robothardware-conf-file-option "pdgains.file_name: ${PROJECT_SOURCE_DIR}/models/PDgains.sav"
  --conf-file-option "abc_leg_offset: 0.0, 0.08, 0.0"
  --conf-file-option "end_effectors: rleg,RLEG_JOINT5,WAIST,0.0,0.0,-0.096,0.0,0.0,0.0,0.0, lleg,LLEG_JOINT5,WAIST,0.0,0.0,-0.096,0.0,0.0,0.0,0.0,"
  --conf-dt-option "0.002"
  --simulation-timestep-option "0.002"
#  --conf-file-option "collision_pair: WAIST:LLEG_JOINT1 WAIST:LLEG_JOINT2 WAIST:LLEG_JOINT3 WAIST:LLEG_JOINT4 WAIST:LLEG_JOINT5 WAIST:RLEG_JOINT1 WAIST:RLEG_JOINT2 WAIST:RLEG_JOINT3 WAIST:RLEG_JOINT4 WAIST:RLEG_JOINT5 LLEG_JOINT0:RLEG_JOINT0 LLEG_JOINT0:LLEG_JOINT2 LLEG_JOINT0:LLEG_JOINT3 LLEG_JOINT0:LLEG_JOINT4 LLEG_JOINT0:LLEG_JOINT5 LLEG_JOINT0:RLEG_JOINT1 LLEG_JOINT0:RLEG_JOINT2 LLEG_JOINT0:RLEG_JOINT3 LLEG_JOINT0:RLEG_JOINT4 LLEG_JOINT0:RLEG_JOINT5 RLEG_JOINT0:LLEG_JOINT1 RLEG_JOINT0:LLEG_JOINT2 RLEG_JOINT0:LLEG_JOINT3 RLEG_JOINT0:LLEG_JOINT4 RLEG_JOINT0:LLEG_JOINT5 RLEG_JOINT0:RLEG_JOINT2 RLEG_JOINT0:RLEG_JOINT3 RLEG_JOINT0:RLEG_JOINT4 RLEG_JOINT0:RLEG_JOINT5 LLEG_JOINT1:LLEG_JOINT3 LLEG_JOINT1:LLEG_JOINT4 LLEG_JOINT1:LLEG_JOINT5 LLEG_JOINT1:RLEG_JOINT1 LLEG_JOINT1:RLEG_JOINT2 LLEG_JOINT1:RLEG_JOINT3 LLEG_JOINT1:RLEG_JOINT4 LLEG_JOINT1:RLEG_JOINT5 LLEG_JOINT2:LLEG_JOINT4 LLEG_JOINT2:LLEG_JOINT5 LLEG_JOINT2:RLEG_JOINT1 LLEG_JOINT2:RLEG_JOINT2 LLEG_JOINT2:RLEG_JOINT3 LLEG_JOINT2:RLEG_JOINT4 LLEG_JOINT2:RLEG_JOINT5 LLEG_JOINT3:LLEG_JOINT5 LLEG_JOINT3:RLEG_JOINT1 LLEG_JOINT3:RLEG_JOINT2 LLEG_JOINT3:RLEG_JOINT3 LLEG_JOINT3:RLEG_JOINT4 LLEG_JOINT3:RLEG_JOINT5 LLEG_JOINT4:RLEG_JOINT1 LLEG_JOINT4:RLEG_JOINT2 LLEG_JOINT4:RLEG_JOINT3 LLEG_JOINT4:RLEG_JOINT4 LLEG_JOINT4:RLEG_JOINT5 LLEG_JOINT5:RLEG_JOINT1 LLEG_JOINT5:RLEG_JOINT2 LLEG_JOINT5:RLEG_JOINT3 LLEG_JOINT5:RLEG_JOINT4 LLEG_JOINT5:RLEG_JOINT5 RLEG_JOINT1:RLEG_JOINT3 RLEG_JOINT1:RLEG_JOINT4 RLEG_JOINT1:RLEG_JOINT5 RLEG_JOINT2:RLEG_JOINT4 RLEG_JOINT2:RLEG_JOINT5 RLEG_JOINT3:RLEG_JOINT5"
  --conf-file-option "collision_pair: WAIST:LLEG_JOINT1 WAIST:LLEG_JOINT3 WAIST:LLEG_JOINT4 WAIST:RLEG_JOINT1 WAIST:RLEG_JOINT2 WAIST:RLEG_JOINT3 WAIST:RLEG_JOINT4 WAIST:RLEG_JOINT5 LLEG_JOINT0:RLEG_JOINT0 LLEG_JOINT0:LLEG_JOINT3 LLEG_JOINT0:LLEG_JOINT4 LLEG_JOINT0:LLEG_JOINT5 LLEG_JOINT0:RLEG_JOINT1 LLEG_JOINT0:RLEG_JOINT2 LLEG_JOINT0:RLEG_JOINT3 LLEG_JOINT0:RLEG_JOINT4 LLEG_JOINT0:RLEG_JOINT5 RLEG_JOINT0:LLEG_JOINT1 RLEG_JOINT0:LLEG_JOINT2 RLEG_JOINT0:LLEG_JOINT3 RLEG_JOINT0:LLEG_JOINT4 RLEG_JOINT0:LLEG_JOINT5 RLEG_JOINT0:RLEG_JOINT3 RLEG_JOINT0:RLEG_JOINT4 RLEG_JOINT0:RLEG_JOINT5 LLEG_JOINT1:LLEG_JOINT3 LLEG_JOINT1:LLEG_JOINT4 LLEG_JOINT1:LLEG_JOINT5 LLEG_JOINT1:RLEG_JOINT1 LLEG_JOINT1:RLEG_JOINT2 LLEG_JOINT1:RLEG_JOINT3 LLEG_JOINT1:RLEG_JOINT4 LLEG_JOINT1:RLEG_JOINT5 LLEG_JOINT2:LLEG_JOINT4 LLEG_JOINT2:LLEG_JOINT5 LLEG_JOINT2:RLEG_JOINT1 LLEG_JOINT2:RLEG_JOINT2 LLEG_JOINT2:RLEG_JOINT3 LLEG_JOINT2:RLEG_JOINT4 LLEG_JOINT2:RLEG_JOINT5 LLEG_JOINT3:RLEG_JOINT1 LLEG_JOINT3:RLEG_JOINT2 LLEG_JOINT3:RLEG_JOINT3 LLEG_JOINT3:RLEG_JOINT4 LLEG_JOINT3:RLEG_JOINT5 LLEG_JOINT4:RLEG_JOINT1 LLEG_JOINT4:RLEG_JOINT2 LLEG_JOINT4:RLEG_JOINT3 LLEG_JOINT4:RLEG_JOINT4 LLEG_JOINT4:RLEG_JOINT5 LLEG_JOINT5:RLEG_JOINT1 LLEG_JOINT5:RLEG_JOINT2 LLEG_JOINT5:RLEG_JOINT3 LLEG_JOINT5:RLEG_JOINT4 LLEG_JOINT5:RLEG_JOINT5 RLEG_JOINT1:RLEG_JOINT3 RLEG_JOINT1:RLEG_JOINT4 RLEG_JOINT1:RLEG_JOINT5 RLEG_JOINT2:RLEG_JOINT4 RLEG_JOINT2:RLEG_JOINT5"
  )

compile_model_for_closed_robots($ENV{CVSDIR}/euslib/rbrain/staro/STAROmain.wrl STARO
  --robothardware-conf-file-option "pdgains.file_name: ${PROJECT_SOURCE_DIR}/models/PDgains.sav"
  --conf-file-option "abc_leg_offset: 0.0, 0.1, 0.0"
  --conf-file-option "end_effectors: rarm,RARM_JOINT7,CHEST_JOINT1,0.0,-0.15701,0.0,0.57735,-0.57735,-0.57735,2.0944, larm,LARM_JOINT7,CHEST_JOINT1,-5.684342e-17,0.15701,-1.136868e-16,-0.57735,-0.57735,0.57735,2.0944, rleg,RLEG_JOINT5,WAIST,0.0,0.0,-0.096,0.0,0.0,0.0,0.0, lleg,LLEG_JOINT5,WAIST,0.0,0.0,-0.096,0.0,0.0,0.0,0.0,"
  --conf-file-option "torque_offset: 0, 0, 0, 0, 0, 0, 0, 0,  0, 0, 0, 0, 0, 0, 0, 0,  0, 0, 0, 0, 0, 0,   0, 0, 0, 0, 0, 0,  0, 0, 0, 0, 0, 0"
  --conf-file-option "torque_filter_params: 2, 1.0, 1.88903, -0.89487, 0.0014603, 0.0029206, 0.0014603"
  --conf-file-option "error_to_torque_gain: 0, 0, 0, 0, 0, 0, 0, 0,  0, 0, 0, 0, 0, 0, 0, 0,  0, 0, 0, 0, 0, 0,   0, 0, 0, 0, 0, 0,  0, 0, 0, 0, 0, 0"
  --conf-file-option "error_dead_zone: 0, 0, 0, 0, 0, 0, 0, 0,  0, 0, 0, 0, 0, 0, 0, 0,  0, 0, 0, 0, 0, 0,   0, 0, 0, 0, 0, 0,  0, 0, 0, 0, 0, 0"
  --conf-file-option "torque_gain: 0.0001"
  --conf-file-option "collision_pair: WAIST:RARM_JOINT2 WAIST:LARM_JOINT2 RLEG_JOINT2:RLEG_JOINT5 LLEG_JOINT2:LLEG_JOINT5 RARM_JOINT7:LARM_JOINT7 RARM_JOINT6:LARM_JOINT7 RARM_JOINT7:LARM_JOINT6 LARM_JOINT7:RLEG_JOINT5 RARM_JOINT7:RLEG_JOINT5 RARM_JOINT5:LARM_JOINT7 RARM_JOINT7:LARM_JOINT5 RARM_JOINT7:LLEG_JOINT5 LARM_JOINT7:LLEG_JOINT5 RARM_JOINT6:LARM_JOINT6 RARM_JOINT7:RLEG_JOINT4 LARM_JOINT7:RLEG_JOINT4 RARM_JOINT7:LLEG_JOINT4 LARM_JOINT6:LLEG_JOINT5 RARM_JOINT6:LARM_JOINT5 LARM_JOINT6:RLEG_JOINT5 RARM_JOINT4:LARM_JOINT7 RARM_JOINT7:LARM_JOINT4 RARM_JOINT6:LLEG_JOINT5 LARM_JOINT7:LLEG_JOINT4 RARM_JOINT5:LARM_JOINT6 RARM_JOINT6:RLEG_JOINT5 RARM_JOINT4:LARM_JOINT6 RARM_JOINT7:RLEG_JOINT3 LARM_JOINT7:LLEG_JOINT3 RARM_JOINT7:LLEG_JOINT3 RARM_JOINT7:LARM_JOINT3 RARM_JOINT5:LARM_JOINT5 LARM_JOINT7:RLEG_JOINT3 RARM_JOINT6:LLEG_JOINT4 LARM_JOINT6:LLEG_JOINT4 RARM_JOINT6:RLEG_JOINT4 RLEG_JOINT5:LLEG_JOINT5 LARM_JOINT5:RLEG_JOINT5 RARM_JOINT5:RLEG_JOINT5 RARM_JOINT5:LLEG_JOINT5 RARM_JOINT3:LARM_JOINT7 LARM_JOINT5:LLEG_JOINT5 RARM_JOINT6:LARM_JOINT4 LARM_JOINT6:LLEG_JOINT3 RLEG_JOINT5:LLEG_JOINT4 LARM_JOINT6:RLEG_JOINT3 LARM_JOINT5:RLEG_JOINT4 RARM_JOINT6:LLEG_JOINT3 RARM_JOINT5:RLEG_JOINT4 RARM_JOINT2:LARM_JOINT7 RARM_JOINT5:LARM_JOINT4 LARM_JOINT4:LLEG_JOINT5 RARM_JOINT6:RLEG_JOINT3 RARM_JOINT7:LARM_JOINT2 LARM_JOINT4:RLEG_JOINT5 RARM_JOINT5:LLEG_JOINT4 RLEG_JOINT4:LLEG_JOINT5 RARM_JOINT4:RLEG_JOINT5 RARM_JOINT7:RLEG_JOINT2 RARM_JOINT6:LARM_JOINT3 LARM_JOINT7:RLEG_JOINT2 LARM_JOINT5:LLEG_JOINT4 RARM_JOINT4:LLEG_JOINT5 RARM_JOINT4:LARM_JOINT5 RARM_JOINT3:LARM_JOINT6 LARM_JOINT7:LLEG_JOINT2 RARM_JOINT7:LLEG_JOINT2 RARM_JOINT3:LLEG_JOINT5 RARM_JOINT3:LARM_JOINT5 RARM_JOINT6:RLEG_JOINT2 RARM_JOINT5:LLEG_JOINT3 RARM_JOINT3:RLEG_JOINT5 RARM_JOINT5:RLEG_JOINT3 RARM_JOINT6:LLEG_JOINT2 LARM_JOINT5:RLEG_JOINT3 RLEG_JOINT5:LLEG_JOINT3 LARM_JOINT4:LLEG_JOINT4 LARM_JOINT4:RLEG_JOINT4 RLEG_JOINT4:LLEG_JOINT4 LARM_JOINT3:LLEG_JOINT5 LARM_JOINT5:LLEG_JOINT3 LARM_JOINT3:RLEG_JOINT5 RLEG_JOINT3:LLEG_JOINT5 RARM_JOINT5:LARM_JOINT3 LARM_JOINT6:RLEG_JOINT2 LARM_JOINT6:LLEG_JOINT2 RARM_JOINT1:LARM_JOINT7 RARM_JOINT7:LARM_JOINT1 RARM_JOINT5:RLEG_JOINT2 LARM_JOINT5:LLEG_JOINT2 RARM_JOINT4:LLEG_JOINT3 LARM_JOINT5:RLEG_JOINT2 LARM_JOINT4:LLEG_JOINT3 LARM_JOINT4:RLEG_JOINT3 LARM_JOINT3:LLEG_JOINT4 LARM_JOINT3:RLEG_JOINT4 LARM_JOINT2:LLEG_JOINT5 LARM_JOINT2:RLEG_JOINT5 RARM_JOINT5:LLEG_JOINT2 LARM_JOINT7:RLEG_JOINT0 RARM_JOINT7:LLEG_JOINT0 RARM_JOINT7:RLEG_JOINT0 RARM_JOINT2:RLEG_JOINT5 RARM_JOINT2:LLEG_JOINT5 RLEG_JOINT2:LLEG_JOINT5 RLEG_JOINT3:LLEG_JOINT4 RARM_JOINT3:RLEG_JOINT4 LARM_JOINT7:LLEG_JOINT0 RARM_JOINT3:LLEG_JOINT4 RLEG_JOINT4:LLEG_JOINT3 RARM_JOINT4:RLEG_JOINT3 RLEG_JOINT5:LLEG_JOINT2 RARM_JOINT1:LLEG_JOINT5 WAIST:RARM_JOINT7 RLEG_JOINT1:LLEG_JOINT5 RARM_JOINT1:RLEG_JOINT5 RLEG_JOINT2:LLEG_JOINT4 LARM_JOINT1:RLEG_JOINT5 RLEG_JOINT5:LLEG_JOINT1 LARM_JOINT6:LLEG_JOINT0 LARM_JOINT1:LLEG_JOINT5 RLEG_JOINT3:LLEG_JOINT3 LARM_JOINT4:LLEG_JOINT2 WAIST:LARM_JOINT7 RARM_JOINT4:LLEG_JOINT2 LARM_JOINT4:RLEG_JOINT2 LARM_JOINT3:RLEG_JOINT3 RLEG_JOINT4:LLEG_JOINT2 LARM_JOINT3:LLEG_JOINT3 RARM_JOINT6:RLEG_JOINT0 RARM_JOINT4:RLEG_JOINT2 RARM_JOINT3:RLEG_JOINT3 RARM_JOINT2:RLEG_JOINT4 RARM_JOINT6:LLEG_JOINT0 RARM_JOINT3:LLEG_JOINT3 RARM_JOINT0:RARM_JOINT7 LARM_JOINT2:RLEG_JOINT3 LARM_JOINT0:LLEG_JOINT5 RLEG_JOINT3:LLEG_JOINT2 LARM_JOINT3:RLEG_JOINT2 RARM_JOINT0:RLEG_JOINT5 RARM_JOINT5:RLEG_JOINT0 LARM_JOINT3:LLEG_JOINT2 LARM_JOINT2:LLEG_JOINT3 RARM_JOINT3:RLEG_JOINT2 RARM_JOINT3:LLEG_JOINT2 RARM_JOINT5:LLEG_JOINT0 RLEG_JOINT4:LLEG_JOINT1 RLEG_JOINT0:LLEG_JOINT5 WAIST:RARM_JOINT6 LARM_JOINT5:RLEG_JOINT0 RARM_JOINT1:RLEG_JOINT4 WAIST:LARM_JOINT6 RLEG_JOINT2:LLEG_JOINT3 LARM_JOINT0:LARM_JOINT7 RARM_JOINT2:RLEG_JOINT3 LARM_JOINT5:LLEG_JOINT0 RLEG_JOINT5:LLEG_JOINT0 RARM_JOINT0:RARM_JOINT6 WAIST:RARM_JOINT5 WAIST:RLEG_JOINT5 WAIST:LARM_JOINT5 WAIST:LLEG_JOINT5 LARM_JOINT4:LLEG_JOINT0 RARM_JOINT4:RLEG_JOINT0 RLEG_JOINT0:LLEG_JOINT4 RLEG_JOINT1:LLEG_JOINT3 RARM_JOINT1:RLEG_JOINT3 RLEG_JOINT2:LLEG_JOINT2 LARM_JOINT0:LARM_JOINT6 RARM_JOINT1:RARM_JOINT7 RLEG_JOINT4:LLEG_JOINT0 LARM_JOINT2:LLEG_JOINT2 LARM_JOINT2:RLEG_JOINT2 RARM_JOINT0:RLEG_JOINT4 LARM_JOINT1:LLEG_JOINT3 LARM_JOINT1:RLEG_JOINT3 RLEG_JOINT3:LLEG_JOINT1 LARM_JOINT1:LARM_JOINT7 RLEG_JOINT2:LLEG_JOINT1 RLEG_JOINT1:LLEG_JOINT2 RLEG_JOINT3:LLEG_JOINT0 RLEG_JOINT0:LLEG_JOINT3 RLEG_JOINT0:RLEG_JOINT5 LLEG_JOINT0:LLEG_JOINT5 RARM_JOINT0:RLEG_JOINT3 WAIST:LLEG_JOINT4 LARM_JOINT0:LARM_JOINT5 RARM_JOINT3:RLEG_JOINT0 RARM_JOINT0:RARM_JOINT5 LARM_JOINT3:LLEG_JOINT0 RARM_JOINT0:LLEG_JOINT3 RARM_JOINT2:RARM_JOINT7 RARM_JOINT1:RARM_JOINT6 RARM_JOINT1:RLEG_JOINT2 LARM_JOINT1:LLEG_JOINT2 WAIST:RARM_JOINT4 LARM_JOINT2:LARM_JOINT7 LARM_JOINT0:RLEG_JOINT3 WAIST:LARM_JOINT4 LARM_JOINT1:RLEG_JOINT2 LARM_JOINT1:LARM_JOINT6 WAIST:RLEG_JOINT4 LARM_JOINT0:LLEG_JOINT3 WAIST:RLEG_JOINT3 RARM_JOINT0:RLEG_JOINT2 WAIST:LARM_JOINT3 RLEG_JOINT1:RLEG_JOINT5 RARM_JOINT1:RARM_JOINT5 WAIST:RARM_JOINT3 WAIST:LLEG_JOINT3 LARM_JOINT3:LARM_JOINT7 LARM_JOINT1:LARM_JOINT5 LARM_JOINT0:LLEG_JOINT2 LARM_JOINT0:RLEG_JOINT2 RARM_JOINT3:RARM_JOINT7 RLEG_JOINT0:LLEG_JOINT2 LLEG_JOINT1:LLEG_JOINT5 RLEG_JOINT2:LLEG_JOINT0 WAIST:RARM_JOINT2 LLEG_JOINT2:LLEG_JOINT5 RLEG_JOINT2:RLEG_JOINT5 WAIST:LARM_JOINT2"
  --conf-dt-option "0.002"
  --simulation-timestep-option "0.002"
  )

if(EXISTS $ENV{CVSDIR}/OpenHRP/etc/HRP3HAND_L/HRP3HAND_Lmain.wrl)
  compile_openhrp_model(
    $ENV{CVSDIR}/OpenHRP/etc/HRP3HAND_L/HRP3HAND_Lmain.wrl
    HRP3HAND_L)
endif()

if(EXISTS $ENV{CVSDIR}/OpenHRP/etc/HRP3HAND_R/HRP3HAND_Rmain.wrl)
  compile_openhrp_model(
    $ENV{CVSDIR}/OpenHRP/etc/HRP3HAND_R/HRP3HAND_Rmain.wrl
    HRP3HAND_R)
endif()

# KOJIRO model generation ;; gen kojiro.dae and kojiro.l from kojiro.wrl and kojiro.yaml
if(EXISTS ${PROJECT_SOURCE_DIR}/models/kojiro.wrl)
  compile_openhrp_model(${PROJECT_SOURCE_DIR}/models/kojiro.wrl)
endif(EXISTS ${PROJECT_SOURCE_DIR}/models/kojiro.wrl)

if(EXISTS ${PROJECT_SOURCE_DIR}/models/TESTMDOFARM.wrl)
  compile_openhrp_model(
    ${PROJECT_SOURCE_DIR}/models/TESTMDOFARM.wrl
    TESTMDOFARM)
endif()

macro (generate_default_launch_eusinterface_files_for_jsk_closed_openhrp_robots ROBOT_DIR ROBOT_NAME)
  set(_arg_list ${ARGV})
  # remove arguments of this macro
  list(REMOVE_AT _arg_list 0 1)
  if(EXISTS $ENV{CVSDIR}/OpenHRP/etc/${ROBOT_DIR}/${ROBOT_NAME}main.wrl)
    generate_default_launch_eusinterface_files("$(env CVSDIR)/OpenHRP/etc/${ROBOT_DIR}/${ROBOT_NAME}main.wrl" hrpsys_ros_bridge_tutorials ${ROBOT_NAME} ${_arg_list})
  endif()
endmacro ()
macro (generate_default_launch_eusinterface_files_for_jsk_closed_rbrain_robots ROBOT_DIR ROBOT_NAME)
  set(_arg_list ${ARGV})
  # remove arguments of this macro
  list(REMOVE_AT _arg_list 0 1)
  if(EXISTS $ENV{CVSDIR}/euslib/rbrain/${ROBOT_DIR}/${ROBOT_NAME}main.wrl)
    generate_default_launch_eusinterface_files("$(env CVSDIR)/euslib/rbrain/${ROBOT_DIR}/${ROBOT_NAME}main.wrl" hrpsys_ros_bridge_tutorials ${ROBOT_NAME} ${_arg_list})
  endif()
endmacro ()

generate_default_launch_eusinterface_files_for_jsk_closed_openhrp_robots(HRP2JSK_for_OpenHRP3 HRP2JSK "--use-unstable-hrpsys-config")
generate_default_launch_eusinterface_files_for_jsk_closed_openhrp_robots(HRP2JSKNT_for_OpenHRP3 HRP2JSKNT "--use-unstable-hrpsys-config")
generate_default_launch_eusinterface_files_for_jsk_closed_openhrp_robots(HRP2JSKNTS_for_OpenHRP3 HRP2JSKNTS "--use-unstable-hrpsys-config")
generate_default_launch_eusinterface_files_for_jsk_closed_openhrp_robots(HRP2W_for_OpenHRP3 HRP2W "--use-unstable-hrpsys-config")
generate_default_launch_eusinterface_files_for_jsk_closed_openhrp_robots(HRP4R HRP4R "--use-unstable-hrpsys-config")
generate_default_launch_eusinterface_files_for_jsk_closed_rbrain_robots(staro STARO "--use-unstable-hrpsys-config")
generate_default_launch_eusinterface_files_for_jsk_closed_rbrain_robots(urataleg URATALEG "--use-unstable-hrpsys-config")
generate_default_launch_eusinterface_files(
  "$(find hrpsys_ros_bridge_tutorials)/models/TESTMDOFARM.wrl"
  hrpsys_ros_bridge_tutorials
  TESTMDOFARM
  "--no-euslisp")
generate_default_launch_eusinterface_files("$(find openhrp3)/share/OpenHRP-3.1/sample/model/sample1.wrl" hrpsys_ros_bridge_tutorials SampleRobot "--use-unstable-hrpsys-config")

macro (generate_hand_attached_hrp2_model _robot_name)
  set(_model_dir "${PROJECT_SOURCE_DIR}/models/")
  set(_in_urdf_file "${_model_dir}/${_robot_name}.urdf")
  set(_out_urdf_file "${_model_dir}/${_robot_name}_WH.urdf")
  string(TOLOWER ${_robot_name} _srobot_name)
  set(_launch_file "${PROJECT_SOURCE_DIR}/launch/${_srobot_name}_ros_bridge.launch")
  set(_script_file "${PROJECT_SOURCE_DIR}/models/gen_hand_attached_hrp2_model.sh")
  message("generate hand_attached_hrp2_model for ${_robot_name}")
  add_custom_command(OUTPUT ${_out_urdf_file}
      COMMAND ${_script_file} ${_robot_name} ${_in_urdf_file} ${_out_urdf_file} ${_launch_file} ${PROJECT_SOURCE_DIR}/..
      DEPENDS ${_in_urdf_file} ${_launch_file} "${_model_dir}/HRP3HAND_L.urdf" "${_model_dir}/HRP3HAND_R.urdf")
  add_custom_target(${_robot_name}_model_generate DEPENDS ${_out_urdf_file})
  list(APPEND compile_urdf_robots ${_robot_name}_model_generate)
endmacro()

if(EXISTS $ENV{CVSDIR}/OpenHRP/etc/HRP3HAND_R/HRP3HAND_Rmain.wrl)
  generate_hand_attached_hrp2_model(HRP2JSKNT)
  generate_hand_attached_hrp2_model(HRP2JSKNTS)
  add_custom_target(all_robots_model_generate ALL DEPENDS ${compile_urdf_robots})
endif()

##
## test
##

rosbuild_check_for_display(disp)
if(disp)
  file(MAKE_DIRECTORY ${PROJECT_SOURCE_DIR}/build/images)
  rosbuild_find_ros_package("hrpsys")
  if(EXISTS ${hrpsys_PACKAGE_PATH}/share/hrpsys/samples/HRP4C/HRP4Cmain.wrl)
    rosbuild_add_rostest(launch/hrp4c.launch)
    rosbuild_add_rostest(launch/hrp4c_nosim.launch)
  endif()
  # rosbuild_add_rostest(launch/pa10.launch) #segfault?
  rosbuild_add_rostest(launch/darwin.launch)
  rosbuild_add_rostest(launch/samplerobot.launch)
  rosbuild_add_rostest(launch/hironx.launch)
  rosbuild_add_rostest(launch/smartpal5.launch)
  rosbuild_add_rostest(launch/pr2.launch)
  rosbuild_add_rostest(launch/gr001.launch)
endif(disp)

##
## documents
##
rosbuild_find_ros_package("jsk_tools")
execute_process(COMMAND cmake -E chdir ${PROJECT_SOURCE_DIR} python ${jsk_tools_PACKAGE_PATH}/bin/launchdoc-generator.py ${PROJECT_NAME} --output_dir=./build/ --nomakefile RESULT_VARIABLE _make_failed)




