#BUILDING A PLUGIN FROM SCRATCH USING CMAKE

#TODO 0: Set CMAKE_VERBOSE on for more helpful command line info while compiling
set( CMAKE_VERBOSE_MAKEFILE on )

#TODO1 : Set CMAKE version:
cmake_minimum_required(VERSION 3.12.0) #Version Selected Using Hendrik's Implementation

#TODO 2: Set Project Name and Version
project(Skeleton VERSION 1.0) #project version set to 1.0

#TODO 3: Set IMPORTANT PATHS:
#############################
#############################

# 3.1 SDK Path because a lot of the directories that are to be included
# are present in the SDK dir
set(AE_SDK_PATH "C:/Users/arini/Desktop/Trioscope/PluginDev/AfterEffectsSDK") #Fix spaces/Don't do spaces
## How to set this up so that it doesn't depend on my own unique path?

# #3.2 Set After Effects File Path:
# set(AE_PATH "C:/Program Files/Adobe/Adobe After Effects 2024")

# #3.3 After Effects Binary path?
# set(AE_BINARY_PATH "${AE_PATH}/Support Files/AfterFX.exe")

# #3.4 After Effects .aex plugin output directory
# set(AE_PLUGIN_OUTPUT_DIR "C:/Program Files/Adobe/Common/Plug-ins/7.0/MediaCore/trioBuild")

#TODO 4: GLOB RECURSE SRC FILES

#Motivation behind selecting these files:
#These were the files that were included in Skeleton.h
file(GLOB_RECURSE SOURCE_FILES ${AE_SDK_PATH}/Examples/Util/*.cpp ./Skeleton_Strings.cpp ./Skeleton.cpp)

#TODO 5: Include the necessary directories
include_directories(
    ${AE_SDK_PATH}/Examples/Headers
    ${AE_SDK_PATH}/Examples/Headers/SP
    # ${AE_SDK_PATH}/Examples/Headers/Win
    ${AE_SDK_PATH}/Examples/Resources
    ${AE_SDK_PATH}/Examples/Util
)

#TODO 6: Actually make our Library!
#Double Check Module/Shared
add_library(${PROJECT_NAME} MODULE ${SOURCE_FILES})
#This step produces a dll, verified

#Use add_custom_command to figure out how to compile the weird resources files separately
add_custom_command(
			OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.rr
			COMMAND cl /I "${AE_SDK_PATH}/Examples/Headers" /EP \"${CMAKE_CURRENT_SOURCE_DIR}/SkeletonPiPL.r\" > ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.rr
	)

    add_custom_command(
		DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.rr
		OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.rrc
		COMMAND "${AE_SDK_PATH}/Examples/Resources/PiPLtool.exe" ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.rr ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.rrc
	)

    add_custom_command(
		DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.rrc
		OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.rc
		COMMAND cl /D "MSWindows" /EP ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.rrc > ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.rc
	)

    target_sources(
		${PROJECT_NAME} PRIVATE
		${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.rc
	)

    set_target_properties(${PROJECT_NAME} PROPERTIES SUFFIX ".aex")