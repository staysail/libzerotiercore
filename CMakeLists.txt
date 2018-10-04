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

project(zerotiercore LANGUAGES CXX)
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

# For tracing zerotier:
#
# Add a fprintf in node/Trace.cpp at ZT_LOCAL_TRACE
# e.g.: fprintf(stderr, "ZT_TRACE: %s\n", traceMsgBuf);
#
# Uncomment the line below:
#target_compile_options(${PROJECT_NAME} PRIVATE -DZT_TRACE)

# Because we are going to expose this as a C library, we need to depend on
# the C++ runtime.
target_link_libraries(${PROJECT_NAME} PRIVATE
	${CMAKE_CXX_IMPLICIT_LINK_LIBRARIES})

install(TARGETS ${PROJECT_NAME}
	EXPORT ${PROJECT_NAME}-target
	INCLUDES DESTINATION include
	ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
	LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

install(FILES ZeroTierOne/include/ZeroTierOne.h
	DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/zerotiercore
)

install(EXPORT ${PROJECT_NAME}-target
	DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
	FILE ${PROJECT_NAME}-config.cmake
	NAMESPACE zerotiercore::
)
