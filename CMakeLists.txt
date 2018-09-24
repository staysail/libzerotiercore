#
# Copyright 2018 Staysail Systems, Inc. <info@staysail.tech>
#
# Use under the terms of the GPLv3, see the file LICENSE.GPLv3 for
# more details.   Contact info@staysail.tech to make other
# arrangements for altenrative licensing.
#
# Note that the above license *only* pertains to these CMake recipes.
# The actual ZeroTier code is provided by ZeroTier under a GPLv3
# license (at this writing) -- please check with them for availability
# of other license terms.  (We use the same license terms here to avoid
# confusion.  If more liberal licensing of the CMake recipes provided
# here is needed for any reason, please contact us and we will be happy
# to accommodate you.)
#
# This CMakeFile drives production of the libzerotiercore project
# from sources originating from the ZeroTierOne project.  Please see
# the ZeroTierOne project (github.com/zerotier/ZeroTierOne) for a
# complete and more general implementation of the ZeroTier stack.
#
cmake_minimum_required(VERSION 3.1)
include (GNUInstallDirs)

project(zerotiercore)
set(CMAKE_CXX_STANDARD 11)

# Git submodule -- we use that to get ZeroTierOne
find_package(Git QUIET)
if(GIT_FOUND AND EXISTS "${PROJECT_SOURCE_DIR}/.git")
	# Update submodules as needed
	option(GIT_SUBMODULE "Check submodules during build" ON)
	if (GIT_SUBMODULE)
		message(STATUS "Submodule update")
		execute_process(COMMAND ${GIT_EXECUTABLE} submodule update --init --recursive --depth 1
			WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
			RESULT_VARIABLE GIT_SUBMOD_RESULT
		)
		if (NOT GIT_SUBMOD_RESULT EQUAL "0")
			message(FATAL_ERROR "git submodule update --init failed with ${GIT_SUBMOD_RESULT}, please checkout submodules")
		endif()
	endif()
endif()

if(NOT EXISTS "${PROJECT_SOURCE_DIR}/ZeroTierOne/CMakeLists.txt")
	message(FATAL_ERROR "The submodules were not downloaded! GIT_SUBMODULE was turned off or failed. Please update submodules and try again.")
endif()


file(GLOB ztsrcs ZeroTierOne/node/*.cpp)
add_library(${PROJECT_NAME} ${ztsrcs})

set_target_properties (${PROJECT_NAME}
        PROPERTIES LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR})
set_target_properties (${PROJECT_NAME}
        PROPERTIES ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR})

install(TARGETS ${PROJECT_NAME}
	EXPORT ${PROJECT_NAME}-target
	FRAMEWORK DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT library
	ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT library
	LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT library
)

install(FILES ZeroTierOne/include/ZeroTierOne.h
	DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/zerotiercore
	COMPONENT headers
)

install(EXPORT ${PROJECT_NAME}-target
	FILE ${PROJECT_NAME}-config.cmake
	NAMESPACE zerotiercore::
	DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
)

