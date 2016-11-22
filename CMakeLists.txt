cmake_minimum_required(VERSION 3.3)

project (get-configured NONE)

include (ExternalProject)

# need to find our Find* modules.
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

if (NOT CMAKE_BUILD_TYPE)
	set (CMAKE_BUILD_TYPE Release CACHE INTERNAL "Choose the type of build. Release, Debug etc." FORCE)
endif ()

# Very basic multi-configuration generator detection.
if (CMAKE_GENERATOR MATCHES "Visual Studio .*")
	message(STATUS "Multi-Configuration generator detected.")
	set (MULTI_CONFIG_COMMAND_EXT --config ${CMAKE_BUILD_TYPE})
endif ()

# which of the GET projects and dependencies to be configured
# default is to build all
option (CONFIGURE_CSIM "Configure and build CSim" OFF)
option (CONFIGURE_C2BB "Configure and build CellML-2-BeatBox" OFF)
option (CONFIGURE_GET_SIMULATOR "Configure and build GET Simulator" OFF)
option (CONFIGURE_ZLIB "Configure and build zlib dependency" ON)
option (CONFIGURE_LIBXML2 "Configure and build libxml2 dependency" ON)
option (CONFIGURE_LIBCELLML "Configure and build CellML API dependency" OFF)
option (CONFIGURE_LLVM "Configure and build LLVM dependency" OFF)
option (CONFIGURE_CLANG "Configure and build Clang dependency" OFF)
option (CONFIGURE_LIBSBML "Configure and build libSBML dependency" ON)
option (CONFIGURE_LIBNUML "Configure and build libNuML dependency" ON)
option (CONFIGURE_LIBSEDML "Configure and build libSEDML dependency" OFF)
option (CONFIGURE_SUNDIALS "Configure and build Sundials dependency" ON)


# Common settings for dependency external projects
set (DEPENDENCIES_PREFIX "${CMAKE_CURRENT_BINARY_DIR}/Dependencies")
set (DEPENDENCIES_INSTALL_DIR "${CMAKE_INSTALL_PREFIX}")
set (DEPENDENCIES_CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX=${DEPENDENCIES_INSTALL_DIR}
    -DPACKAGE_CONFIG_DIR=cmake
    -DCMAKE_PREFIX_PATH=${DEPENDENCIES_INSTALL_DIR}/cmake
    -DCMAKE_MODULE_PATH=${CMAKE_MODULE_PATH}
    -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
    -DCMAKE_NO_SYSTEM_FROM_IMPORTED=YES
    -DCMAKE_POSITION_INDEPENDENT_CODE=YES # -fPIC flag - always enable
    )

# Common settings for the GET projects
set (GET_PREFIX "${CMAKE_CURRENT_BINARY_DIR}/GET")
set (GET_INSTALL_DIR "${CMAKE_INSTALL_PREFIX}")
set (GET_CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX=${GET_INSTALL_DIR}
    -DPACKAGE_CONFIG_DIR=cmake
    -DCMAKE_PREFIX_PATH=${DEPENDENCIES_INSTALL_DIR}/cmake
    -DCMAKE_MODULE_PATH=${CMAKE_MODULE_PATH}
    -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
    -DCMAKE_NO_SYSTEM_FROM_IMPORTED=YES
    -DCMAKE_POSITION_INDEPENDENT_CODE=YES # -fPIC flag - always enable
    )

# make it easy to put all the dependencies in the same place
#
set_property (DIRECTORY PROPERTY EP_BASE ${DEPENDENCIES_PREFIX})

if (CONFIGURE_ZLIB)
    ExternalProject_Add (ep_zlib
        GIT_REPOSITORY https://github.com/nickerso/opencmiss-zlib
        GIT_TAG devel
        UPDATE_COMMAND ""
        PATCH_COMMAND ""
        BUILD_COMMAND ${CMAKE_COMMAND} --build . ${MULTI_CONFIG_COMMAND_EXT}
        TEST_COMMAND ""
        CMAKE_ARGS ${DEPENDENCIES_CMAKE_ARGS} -DBUILD_TESTS=OFF
    )
endif ()

if (CONFIGURE_LIBXML2)
    ExternalProject_Add (ep_libxml2
        DEPENDS ep_zlib
        GIT_REPOSITORY https://github.com/nickerso/opencmiss-libxml2
        GIT_TAG standalone-csim-build
        UPDATE_COMMAND ""
        PATCH_COMMAND ""
        BUILD_COMMAND ${CMAKE_COMMAND} --build . ${MULTI_CONFIG_COMMAND_EXT}
        TEST_COMMAND ""
        CMAKE_ARGS ${DEPENDENCIES_CMAKE_ARGS} -DBUILD_TESTS=OFF
    )
endif ()

if (CONFIGURE_LIBCELLML)
    # Despite the name, this is still the CellML API
    ExternalProject_Add (ep_libcellml
        DEPENDS ep_libxml2 ep_zlib
        GIT_REPOSITORY https://github.com/nickerso/opencmiss-libcellml
        GIT_TAG standalone-csim-building
        UPDATE_COMMAND ""
        PATCH_COMMAND ""
        BUILD_COMMAND ${CMAKE_COMMAND} --build . ${MULTI_CONFIG_COMMAND_EXT}
        TEST_COMMAND ""
        CMAKE_ARGS ${DEPENDENCIES_CMAKE_ARGS} -DBUILD_TESTING=OFF
            -DENABLE_ANNOTOOLS=ON
            -DENABLE_CCGS=ON
            -DENABLE_CELEDS=OFF
            -DENABLE_CELEDS_EXPORTER=OFF
            -DENABLE_CIS=OFF
            -DENABLE_CUSES=ON
            -DENABLE_CEVAS=ON
            -DENABLE_CONTEXT=OFF
            -DENABLE_EXAMPLES=OFF
            -DENABLE_GSL_INTEGRATORS=OFF
            -DENABLE_MALAES=ON
            -DENABLE_MOFS=OFF
            -DENABLE_RDF=OFF
            -DENABLE_SPROS=OFF
            -DENABLE_SRUS=OFF
            -DENABLE_TELICEMS=OFF
            -DENABLE_VACSS=OFF
            -DENABLE_CGRS=OFF
    )
endif ()

if (CONFIGURE_LIBSBML)
    ExternalProject_Add (ep_libsbml
        DEPENDS ep_libxml2 ep_zlib
        GIT_REPOSITORY https://github.com/nickerso/get-libsbml/
        GIT_TAG libsbml-5.13.0
        UPDATE_COMMAND ""
        PATCH_COMMAND ""
        BUILD_COMMAND ${CMAKE_COMMAND} --build . ${MULTI_CONFIG_COMMAND_EXT}
        TEST_COMMAND ""
        CMAKE_ARGS ${DEPENDENCIES_CMAKE_ARGS}
            -DWITH_CPP_NAMESPACE=ON
            -DWITH_SWIG=OFF
            -DWITH_BZIP2=OFF
            -DLIBSBML_SKIP_SHARED_LIBRARY=TRUE
    )
endif ()

if (CONFIGURE_LIBNUML)
    ExternalProject_Add (ep_libnuml
        DEPENDS ep_libsbml ep_libxml2 ep_zlib
        GIT_REPOSITORY https://github.com/nickerso/NuML
        GIT_TAG get-configured
        UPDATE_COMMAND ""
        PATCH_COMMAND ""
        BUILD_COMMAND ${CMAKE_COMMAND} --build . ${MULTI_CONFIG_COMMAND_EXT}
        TEST_COMMAND ""
        CMAKE_ARGS ${DEPENDENCIES_CMAKE_ARGS}
            -DWITH_CPP_NAMESPACE=OFF
            -DWITH_SWIG=OFF
            -DWITH_BZIP2=OFF
            -DLIBNUML_SKIP_SHARED_LIBRARY=TRUE
    )
endif ()

if (CONFIGURE_LIBSEDML)
    ExternalProject_Add (ep_libsedml
        DEPENDS ep_libsbml ep_libnuml ep_libxml2 ep_zlib
        GIT_REPOSITORY https://github.com/nickerso/libSEDML
        GIT_TAG get-configured
        UPDATE_COMMAND ""
        PATCH_COMMAND ""
        BUILD_COMMAND ${CMAKE_COMMAND} --build . ${MULTI_CONFIG_COMMAND_EXT}
        TEST_COMMAND ""
        CMAKE_ARGS ${DEPENDENCIES_CMAKE_ARGS}
            -DWITH_EXAMPLES=ON
            -DWITH_CPP_NAMESPACE=ON
            -DWITH_SWIG=OFF
            -DWITH_BZIP2=OFF
            -DLIBSEDML_SKIP_SHARED_LIBRARY=TRUE
    )
endif ()

if (CONFIGURE_SUNDIALS)
    ExternalProject_Add (ep_sundials
        GIT_REPOSITORY https://github.com/nickerso/get-sundials
        GIT_TAG v2.7.0
        UPDATE_COMMAND ""
        PATCH_COMMAND ""
        BUILD_COMMAND ${CMAKE_COMMAND} --build . ${MULTI_CONFIG_COMMAND_EXT}
        TEST_COMMAND ""
        CMAKE_ARGS ${DEPENDENCIES_CMAKE_ARGS}
            -DBUILD_SHARED_LIBS=OFF
            -DMPI_ENABLE=OFF
            -DLAPACK_ENABLE=OFF
            -DOPENMP_ENABLE=OFF
            -DPETSC_ENABLE=OFF
            -DSUPERLUMT_ENABLE=OFF
            -DEXAMPLES_INSTALL=ON
            -DEXAMPLES_INSTALL_PATH=${DEPENDENCIES_INSTALL_DIR}/share/sundials/examples
    )
endif ()

if (CONFIGURE_LLVM)
    ExternalProject_Add (ep_llvm
        DEPENDS ep_libxml2 ep_libcellml
        GIT_REPOSITORY https://github.com/nickerso/opencmiss-llvm
        GIT_TAG devel
        UPDATE_COMMAND ""
        PATCH_COMMAND ""
        BUILD_COMMAND ${CMAKE_COMMAND} --build . ${MULTI_CONFIG_COMMAND_EXT}
        TEST_COMMAND ""
        CMAKE_ARGS ${DEPENDENCIES_CMAKE_ARGS}
    )
endif ()

if (CONFIGURE_CLANG)
    ExternalProject_Add (ep_clang
        DEPENDS ep_llvm ep_libxml2 ep_zlib
        GIT_REPOSITORY https://github.com/nickerso/opencmiss-clang
        GIT_TAG backport-3.8-libxml-patch
        UPDATE_COMMAND ""
        PATCH_COMMAND ""
        BUILD_COMMAND ${CMAKE_COMMAND} --build . ${MULTI_CONFIG_COMMAND_EXT}
        TEST_COMMAND ""
        CMAKE_ARGS ${DEPENDENCIES_CMAKE_ARGS}
    )
endif ()

if (CONFIGURE_CSIM)
    ExternalProject_Add (csim
        DEPENDS ep_clang ep_libcellml
        GIT_REPOSITORY https://github.com/nickerso/csim
        GIT_TAG get-configured
        PREFIX ${GET_PREFIX}
        STAMP_DIR ${GET_PREFIX}/stamp/csim
        DOWNLOAD_DIR ${GET_PREFIX}/download/csim
        SOURCE_DIR ${GET_PREFIX}/csim
        BINARY_DIR ${GET_PREFIX}/build/${CMAKE_BUILD_TYPE}/csim
        UPDATE_COMMAND ""
        PATCH_COMMAND ""
        BUILD_COMMAND ${CMAKE_COMMAND} --build . ${MULTI_CONFIG_COMMAND_EXT}
        TEST_COMMAND "ctest"
        CMAKE_ARGS ${GET_CMAKE_ARGS} -DBUILD_TESTING=ON
    )
endif ()

if (CONFIGURE_GET_SIMULATOR)
    ExternalProject_Add (get-simulator
        DEPENDS csim ep_libsedml ep_sundials
        GIT_REPOSITORY https://github.com/nickerso/get-simulator
        GIT_TAG master
        PREFIX ${GET_PREFIX}
        STAMP_DIR ${GET_PREFIX}/stamp/get-simulator
        DOWNLOAD_DIR ${GET_PREFIX}/download/get-simulator
        SOURCE_DIR ${GET_PREFIX}/get-simulator
        BINARY_DIR ${GET_PREFIX}/build/${CMAKE_BUILD_TYPE}/get-simulator
        UPDATE_COMMAND ""
        PATCH_COMMAND ""
        BUILD_COMMAND ${CMAKE_COMMAND} --build . ${MULTI_CONFIG_COMMAND_EXT}
        TEST_COMMAND "ctest"
        CMAKE_ARGS ${GET_CMAKE_ARGS} -DBUILD_TESTING=ON
    )
endif ()

if (CONFIGURE_C2BB)
    ExternalProject_Add (c2bb
        DEPENDS csim
        GIT_REPOSITORY https://github.com/nickerso/c2bb
        GIT_TAG master
        PREFIX ${GET_PREFIX}
        STAMP_DIR ${GET_PREFIX}/stamp/c2bb
        DOWNLOAD_DIR ${GET_PREFIX}/download/c2bb
        SOURCE_DIR ${GET_PREFIX}/c2bb
        BINARY_DIR ${GET_PREFIX}/build/${CMAKE_BUILD_TYPE}/c2bb
        UPDATE_COMMAND ""
        PATCH_COMMAND ""
        BUILD_COMMAND ${CMAKE_COMMAND} --build . ${MULTI_CONFIG_COMMAND_EXT}
        TEST_COMMAND ""
        CMAKE_ARGS ${GET_CMAKE_ARGS}
    )
endif ()
