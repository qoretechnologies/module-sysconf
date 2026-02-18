cmake_minimum_required(VERSION 2.8.12)

project(qore-sysconf-module)

set (VERSION_MAJOR 1)
set (VERSION_MINOR 1)
set (VERSION_PATCH 0)

# where to look first for cmake modules, before ${CMAKE_ROOT}/Modules/ is checked
set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake )

FIND_PACKAGE (Qore REQUIRED)

# detect sysconf function
INCLUDE (CheckFunctionExists)
CHECK_FUNCTION_EXISTS(sysconf HAVE_SYSCONF)
if (HAVE_SYSCONF)
    add_definitions(-DHAVE_SYSCONF)
endif(HAVE_SYSCONF)
CHECK_FUNCTION_EXISTS(confstr HAVE_CONFSTR)
if (HAVE_CONFSTR)
    add_definitions(-DHAVE_CONFSTR)
endif (HAVE_CONFSTR)
CHECK_FUNCTION_EXISTS(pathconf HAVE_PATHCONF)
if (HAVE_PATHCONF)
    add_definitions(-DHAVE_PATHCONF)
endif(HAVE_PATHCONF)

if (${CMAKE_SYSTEM_NAME} MATCHES SunOS)
    add_definitions(-D__EXTENSIONS__)
endif()

# Check for C++11.
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
if(COMPILER_SUPPORTS_CXX11)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
else()
    message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()

include_directories( ${CMAKE_SOURCE_DIR}/src )

set(CPP_SRC
)
set(QPP_SRC
    src/sysconf.qpp
    src/constants_sysconf.qpp
    src/constants_confstr.qpp
    src/constants_pathconf.qpp
)

qore_wrap_qpp_value(QPP_SOURCES METALIST QPP_META_SRC ${QPP_SRC})

SET (module_name "sysconf")

add_library(${module_name} MODULE ${CPP_SRC} ${QPP_SOURCES})

if (DEFINED ENV{DOXYGEN_EXECUTABLE})
    set(DOXYGEN_EXECUTABLE $ENV{DOXYGEN_EXECUTABLE})
endif()

qore_external_binary_module(${module_name} "${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH}")

qore_dist("${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH}")

# install qpp metadata files
if (COMMAND qore_install_qpp_metadata)
    qore_install_qpp_metadata(${module_name} ${QPP_META_SRC})
else()
    install(FILES ${QPP_META_SRC}
            DESTINATION ${CMAKE_INSTALL_DATADIR}/qore/metadata)
endif()

qore_config_info()

if (DOXYGEN_FOUND)
  qore_wrap_dox(QORE_DOX_SRC ${QORE_DOX_TMPL_SRC})
  add_custom_target(QORE_MOD_DOX_FILES DEPENDS ${QORE_DOX_SRC})
  add_dependencies(docs-module QORE_MOD_DOX_FILES)
endif()
