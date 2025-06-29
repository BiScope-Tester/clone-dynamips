# dynamips - build system
# Control variables:
#  - DYNAMIPS_RENAME : rename executable to dynamips (auto;stable;unstable;<empty>)
#  - DYNAMIPS_CODE   : set default code (stable;unstable;both)
#  - DYNAMIPS_ARCH   : build target architecture (amd64;x86;nojit)
#  - ANY_COMPILER    : skip compiler check (assumes gcc)
#  - CMAKE_INSTALL_PREFIX : where to install

cmake_minimum_required ( VERSION 3.5 )
foreach ( _dir
   "/usr/sfw" # search in "Sun FreeWare" (SunOS)
   "/opt/csw" # search in "OpenCSW" (SunOS)
   )
   if ( IS_ABSOLUTE "${_dir}" AND IS_DIRECTORY "${_dir}" )
      list ( APPEND CMAKE_PREFIX_PATH "${_dir}" )
   endif ()
endforeach ()
if ( CMAKE_PREFIX_PATH )
   message ( STATUS "CMAKE_PREFIX_PATH=${CMAKE_PREFIX_PATH}" )
endif ( CMAKE_PREFIX_PATH )
list ( APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake" )
message ( STATUS "CMAKE_MODULE_PATH=${CMAKE_MODULE_PATH}" )

project ( dynamips 
   VERSION 0.2.23
   LANGUAGES C
)
set ( DYNAMIPS_VERSION_TRAIN "${PROJECT_VERSION}" )
set ( DYNAMIPS_VERSION_SUB )

include ( utils )
include ( dependencies )
include ( configure )

if( APPLE OR (CMAKE_SYSTEM_NAME MATCHES "FreeBSD") )
    include_directories(include /usr/local/include/)
    include_directories(include /opt/local/include/)
    include_directories(include /opt/homebrew/include/)
endif()

# Enable testing
include(CTest)

add_subdirectory ( man )
add_subdirectory ( common )
add_subdirectory ( stable )
add_subdirectory ( unstable )

install_docs (
   "ChangeLog"
   "COPYING"
   "MAINTAINERS"
   "README.md"
   "README.hypervisor"
   "RELEASE-NOTES"
   "TODO"
   )

# uninstall target - leaves behind any directory created during install
set ( _templatefile "${CMAKE_SOURCE_DIR}/cmake/cmake_uninstall.cmake.in" )
set ( _scriptfile "${CMAKE_BINARY_DIR}/cmake_uninstall.cmake" )
configure_file ( "${_templatefile}" "${_scriptfile}" IMMEDIATE @ONLY )
add_custom_target (
   uninstall
   COMMAND ${CMAKE_COMMAND} -P "${_scriptfile}"
   COMMENT "Uninstall files listed in install_manifest.txt"
   )

print_summary ()

# Enable packaging
include(CPack)
