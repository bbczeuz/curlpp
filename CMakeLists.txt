project(curlpp)
cmake_minimum_required(VERSION 2.8)

if (${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION} VERSION_GREATER 2.8)
	#FIXME: Better use 'target_compile_features' instead of CXX_STANDARD
	message(STATUS "Cmake version 2.9 or newer")
	set(CMAKE_CXX_STANDARD 11) # C++11...
	set(CMAKE_CXX_STANDARD_REQUIRED ON) #...is required...
	set(CMAKE_CXX_EXTENSIONS OFF) #...without compiler extensions like gnu++11

	#Debian8: probably needed 
	add_definitions(-std=c++11)
elseif (${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION} VERSION_GREATER 2.7)
	message(STATUS "Cmake version 2.8 or older")
	#XXX: This could work for even older cmake versions, too, but is not tested
	add_definitions(-std=c++11)
	add_definitions(-g)
else()
	#We shouldn't arrive here due to cmake_minimum_required
	message(FATAL_ERROR "Cmake version not supported")
endif()

set(CURLPP_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR})
set(CURLPP_BINARY_DIR ${CMAKE_RUNTIME_OUTPUT_DIRECTORY})



#########################################################################################
# Root folders
set(ProjectRootDir ${${PROJECT_NAME}_SOURCE_DIR}/)
message(STATUS "\tUsing Sources From: ${ProjectRootDir}")

set(ProjectBinaryDir ${${PROJECT_NAME}_BINARY_DIR}/)
message(STATUS "\tPutting all major output in: ${ProjectBinaryDir}")

# Source Subfolders
set(SrcFolder ${ProjectRootDir}src/)
message(STATUS "\tInputs of type .cpp at: ${SrcFolder}")

set(IncludeFolder ${ProjectRootDir}include/)
message(STATUS "\tInputs of type .h and .hpp at: ${IncludeFolder}")

set(ExamplesFolder ${ProjectRootDir}examples/)
message(STATUS "\tInputs to be built as standalone examples are at: ${ExamplesFolder}")

#########################################################################################
# Install folders
#Paths are relative to CMAKE_INSTALL_PREFIX
if (NOT LIB_INSTALL_DIR)
	set(LIB_INSTALL_DIR lib)
endif ()

if (NOT BIN_INSTALL_DIR)
	set(BIN_INSTALL_DIR bin)
endif ()

#add_subdirectory(extras)

#########################################################################################
# For more verbose displays
function(DisplayList Header ListToShow)
	message(STATUS "\t${Header}")
	foreach(ListItem ${ListToShow})
		message(STATUS "\t\t${ListItem}")
	endforeach(ListItem ${ListToShow})
endfunction(DisplayList)

#########################################################################################
# File Lists
file(GLOB_RECURSE HeaderFileList "${IncludeFolder}*")
install(DIRECTORY include/utilspp/ DESTINATION "include/utilspp")
install(DIRECTORY include/curlpp/ DESTINATION "include/curlpp")
DisplayList("Using Header Files:" "${HeaderFileList}")

file(GLOB_RECURSE SourceFileList "${SrcFolder}*")
DisplayList("Using Source Files:" "${SourceFileList}")



#########################################################################################
# Look for dependencies

# Documented at https://cmake.org/cmake/help/v3.0/module/FindCURL.html?highlight=curlpp
# Seems simple.

message(STATUS "Looking for CURL")
include(FindCURL)
cmake_minimum_required(VERSION 2.8)
find_package(CURL REQUIRED)


if(${CURL_FOUND})
	message(STATUS "Found CURL version: ${CURL_VERSION_STRING}")
	message(STATUS "Using CURL include dir(s): ${CURL_INCLUDE_DIRS}")
	message(STATUS "Using CURL lib(s): ${CURL_LIBRARIES}")
else(${CURL_FOUND})
	message(FATAL_ERROR "Could not find CURL")
endif(${CURL_FOUND})

# All following targets should search these directories for headers
include_directories(
	${IncludeFolder}
	${CURL_INCLUDE_DIRS}
)

#########################################################################################
# Connect all the found parts

# Define main library
message(STATUS
	"Defining the shared library for ${PROJECT_NAME} including "
	"all source and header files not in examples."
)

add_library(${PROJECT_NAME} SHARED ${HeaderFileList} ${SourceFileList})
target_link_libraries(${PROJECT_NAME} ${CURL_LIBRARIES})

add_library(${PROJECT_NAME}_static STATIC ${HeaderFileList} ${SourceFileList})
target_link_libraries(${PROJECT_NAME}_static ${CURL_LIBRARIES})

install(TARGETS ${PROJECT_NAME} ${PROJECT_NAME}_static
	RUNTIME DESTINATION ${BIN_INSTALL_DIR} 
	LIBRARY DESTINATION ${LIB_INSTALL_DIR}
	ARCHIVE DESTINATION ${LIB_INSTALL_DIR}/static
)

