cmake_minimum_required (VERSION 3.6.3)

project (MyTest) 

SET_PROPERTY(GLOBAL PROPERTY USE_FOLDERS ON) 

set(CMAKE_CONFIGURATION_TYPES "Debug;Release;RelWithDebInfo" CACHE STRING "Configuration types" FORCE)

SET(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/CMake")

file(GLOB SRC_FILES
    ${CMAKE_SOURCE_DIR}/src/*.h
    ${CMAKE_SOURCE_DIR}/src/*.cpp
)

file(GLOB EXT_FILES )

include_directories(
  ${CMAKE_SOURCE_DIR}/src/
)

#######################################################################################

#set (USE_MSVC_RUNTIME_LIBRARY_DLL true)
#IF (NOT USE_MSVC_RUNTIME_LIBRARY_DLL)
#    #We statically link to reduce dependancies
#    FOREACH(flag_var CMAKE_CXX_FLAGS CMAKE_CXX_FLAGS_DEBUG CMAKE_CXX_FLAGS_RELEASE CMAKE_CXX_FLAGS_MINSIZEREL CMAKE_CXX_FLAGS_RELWITHDEBINFO CMAKE_CXX_FLAGS_RETAIL)
#        IF(${flag_var} MATCHES "/MD")
#            STRING(REGEX REPLACE "/MD" "/MT" ${flag_var} "${${flag_var}}")
#        ENDIF(${flag_var} MATCHES "/MD")
#        IF(${flag_var} MATCHES "/MDd")
#            STRING(REGEX REPLACE "/MDd" "/MTd" ${flag_var} "${${flag_var}}")
#        ENDIF(${flag_var} MATCHES "/MDd")
#    ENDFOREACH(flag_var)
#ENDIF (NOT USE_MSVC_RUNTIME_LIBRARY_DLL)

#######################################################################################

foreach(f ${SRC_FILES})
    # Get the path of the file relative to ${DIRECTORY},
    # then alter it (not compulsory)
    file(RELATIVE_PATH SRCGR ${CMAKE_SOURCE_DIR} ${f})
    set(SRCGR "Sources/${SRCGR}")

    # Extract the folder, ie remove the filename part
    string(REGEX REPLACE "(.*)(/[^/]*)$" "\\1" SRCGR ${SRCGR})

    # Source_group expects \\ (double antislash), not / (slash)
    string(REPLACE / \\ SRCGR ${SRCGR})
    source_group("${SRCGR}" FILES ${f})
endforeach()

## Here Find needed package
#find_package(OpenGL)

find_package(OpenMP)
if(OPENMP_FOUND)
if(MSVC)
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /openmp")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /openmp")
	#set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}")
endif()
endif()

if(MSVC)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /MP")
endif()

## add c++17 complile flag on vs
if (MSVC_VERSION GREATER_EQUAL "1900")
    include(CheckCXXCompilerFlag)
    CHECK_CXX_COMPILER_FLAG("/std:c++17" _cpp_17_flag_supported)
    if (_cpp_17_flag_supported)
		message(STATUS "C++ 17 is suppoted in VS")
        add_compile_options("/std:c++17")
    endif()
endif()

SET(LINK_OPTIONS " ")

#--------------------------------------------------------------------
# preproc macro define
#--------------------------------------------------------------------
add_definitions(-D_CRT_SECURE_NO_WARNINGS)
add_definitions(-D__STDC_FORMAT_MACROS)
add_definitions(-DBX_CONFIG_ENABLE_MSVC_LEVEL4_WARNINGS=1)
add_definitions(-D__STDC_LIMIT_MACROS)
add_definitions(-D__STDC_CONSTANT_MACROS)
add_definitions(-DIMGUI_DISABLE_OBSOLETE_FUNCTIONS)
add_definitions(-DUSE_DL_PREFIX)


MESSAGE(STATUS "Console is visible")

#--------------------------------------------------------------------
# target 
#--------------------------------------------------------------------
SET(EXE_NAME "PPM")
ADD_EXECUTABLE(${EXE_NAME} ${SRC_FILES} ${EXT_FILES})

#--------------------------------------------------------------------
# set as vs startup project 
#--------------------------------------------------------------------
SET_PROPERTY (DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT ${EXE_NAME}) 
GET_PROPERTY (STARTUP_SET DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT SET)
MESSAGE(STATUS "VS_STARTUP_PROJECT is set ? " ${STARTUP_SET})

#--------------------------------------------------------------------
# output dirs, properties of target
#--------------------------------------------------------------------
set_target_properties(${EXE_NAME} PROPERTIES RUNTIME_OUTPUT_DIRECTORY_DEBUG ${CMAKE_SOURCE_DIR}/bin )
set_target_properties(${EXE_NAME} PROPERTIES RUNTIME_OUTPUT_DIRECTORY_RELEASE ${CMAKE_SOURCE_DIR}/bin )
set_target_properties(${EXE_NAME} PROPERTIES RUNTIME_OUTPUT_DIRECTORY_RELWITHDEBINFO ${CMAKE_SOURCE_DIR}/bin )
set_target_properties(${EXE_NAME} PROPERTIES DEBUG_POSTFIX "_d")
set_target_properties(${EXE_NAME} PROPERTIES RELWITHDEBINFO_POSTFIX "RelWithDebInfo")
set_target_properties(${EXE_NAME} PROPERTIES VS_DEBUGGER_WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}/bin")


#######################################################################################
##For reference
##https://cmake.org/cmake/help/latest/manual/cmake-properties.7.html#target-properties
####################################################################################### 
