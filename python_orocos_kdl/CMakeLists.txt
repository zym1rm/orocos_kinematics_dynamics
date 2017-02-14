cmake_minimum_required(VERSION 2.4.6)

project(python_orocos_kdl)

find_package(orocos_kdl)
include_directories(${orocos_kdl_INCLUDE_DIRS})
link_directories(${orocos_kdl_LIBRARY_DIRS})

SET(PYTHON_VERSION 2 CACHE STRING "Python Version")
find_package(PythonInterp ${PYTHON_VERSION} REQUIRED)
find_package(PythonLibs ${PYTHON_VERSION_MAJOR}.${PYTHON_VERSION_MINOR} REQUIRED)
execute_process(COMMAND ${PYTHON_EXECUTABLE} -c "from distutils.sysconfig import get_python_lib; print(get_python_lib(plat_specific=True, prefix=''))" OUTPUT_VARIABLE PYTHON_SITE_PACKAGES OUTPUT_STRIP_TRAILING_WHITESPACE)
list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)
find_package(SIP REQUIRED)
include(SIPMacros)

include_directories(${SIP_INCLUDE_DIR} ${PYTHON_INCLUDE_DIRS})

file(GLOB SIP_FILES "${CMAKE_CURRENT_SOURCE_DIR}/*.sip")
set(SIP_INCLUDES ${SIP_FILES})
set(SIP_EXTRA_OPTIONS "-o")
set(PYTHON_SITE_PACKAGES_INSTALL_DIR ${CMAKE_INSTALL_PREFIX}/${PYTHON_SITE_PACKAGES})

#### Settings for rpath
IF(${CMAKE_MINIMUM_REQUIRED_VERSION} VERSION_GREATER "2.8.12")
    MESSAGE(AUTHOR_WARNING "CMAKE_MINIMUM_REQUIRED_VERSION is now ${CMAKE_MINIMUM_REQUIRED_VERSION}. This check can be removed.")
ENDIF()
IF(NOT (CMAKE_VERSION VERSION_LESS 2.8.12))
    IF(NOT MSVC)
        #add the option to disable RPATH
        OPTION(OROCOSKDL_ENABLE_RPATH "Enable RPATH during installation" TRUE)
        MARK_AS_ADVANCED(OROCOSKDL_ENABLE_RPATH)
    ENDIF(NOT MSVC)

    IF(OROCOSKDL_ENABLE_RPATH)
        #Configure RPATH
        SET(CMAKE_MACOSX_RPATH TRUE) #enable RPATH on OSX. This also suppress warnings on CMake >= 3.0
        # when building, don't use the install RPATH already
        # (but later on when installing)
        SET(CMAKE_BUILD_WITH_INSTALL_RPATH FALSE)
        #build directory by default is built with RPATH
        SET(CMAKE_SKIP_BUILD_RPATH  FALSE)

        #This is relative RPATH for libraries built in the same project
        #I assume that the directory is
        # - install_dir/something for binaries
        # - install_dir/lib for libraries
        LIST(FIND CMAKE_PLATFORM_IMPLICIT_LINK_DIRECTORIES "${CMAKE_INSTALL_PREFIX}/lib" isSystemDir)
        IF("${isSystemDir}" STREQUAL "-1")
            FILE(RELATIVE_PATH _rel_path "${PYTHON_SITE_PACKAGES_INSTALL_DIR}" "${CMAKE_INSTALL_PREFIX}/lib" )
            IF (${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
                SET(CMAKE_INSTALL_RPATH "@loader_path/${_rel_path}")
            ELSE()
                SET(CMAKE_INSTALL_RPATH "\$ORIGIN/${_rel_path}")
            ENDIF()
        ENDIF("${isSystemDir}" STREQUAL "-1")
        # add the automatically determined parts of the RPATH
        # which point to directories outside the build tree to the install RPATH
        SET(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE) #very important!
    ENDIF()
ENDIF()
#####end RPATH




add_sip_python_module(PyKDL PyKDL/PyKDL.sip ${orocos_kdl_LIBRARIES})

install(FILES package.xml DESTINATION share/python_orocos_kdl)
