# NOTE: Ensure that the CUDA Toolkit v11.1 or later is installed and in your PATH.

if (NOT CMAKE_SYSTEM_NAME STREQUAL "Linux")
	message(STATUS "skipping dvplowlatencydemo (requires Linux)")
	return()
endif()

find_package(CUDA 11.1)
if (NOT CUDA_FOUND)
	message(STATUS "skipping cudalowlatencydemo (CUDA not found)")
	return()
endif()
set(OpenGL_GL_PREFERENCE LEGACY)
find_package(OpenGL)
if (NOT OPENGL_FOUND)
	message(STATUS "skipping cudalowlatencydemo (OpenGL not found)")
	return()
endif()
find_package(GLEW)
if (NOT GLEW_FOUND)
	message(STATUS "skipping cudalowlatencydemo (GLEW not found)")
	return()
endif()
find_package(X11)
if (NOT X11_FOUND)
	message(STATUS "skipping cudalowlatencydemo (X11 not found)")
	return()
endif()

if (NOT CMAKE_CUDA_COMPILER)
	set(CMAKE_CUDA_COMPILER ${CUDA_NVCC_EXECUTABLE})
endif()

project(dvplowlatencydemo LANGUAGES CUDA CXX)

set(TARGET_INCLUDE_DIRS
	${CMAKE_CURRENT_SOURCE_DIR}/../
	${CMAKE_CURRENT_SOURCE_DIR}/../common
	${CMAKE_CURRENT_SOURCE_DIR}/../../../thirdparty/nvidia/include
	${CMAKE_CURRENT_SOURCE_DIR}/../../../thirdparty/nvidia/gpudirect/include
	${CMAKE_CURRENT_SOURCE_DIR}/../../../ajantv2
	${CMAKE_CURRENT_SOURCE_DIR}/../../../ajantv2/includes
	${CUDA_INCLUDE_DIRS}
)

set(DVPLOWLATENCYDEMO_BUILD_FILES
	Makefile)
set(DVPLOWLATENCYDEMO_HEADERS
	AJA_Logo_Small.h)
set(DVPLOWLATENCYDEMO_SOURCES
	oglapp.cpp)
set(NVIDIA_COMMON_SOURCES
	${CMAKE_CURRENT_SOURCE_DIR}/../common/ntv2errorlist.cpp
	${CMAKE_CURRENT_SOURCE_DIR}/../common/ntv2errorlist.h
	${CMAKE_CURRENT_SOURCE_DIR}/../common/ntv2glTextureTransfer.h
	${CMAKE_CURRENT_SOURCE_DIR}/../common/ntv2glTextureTransferNV.cpp
	${CMAKE_CURRENT_SOURCE_DIR}/../common/ntv2glTextureTransferNV.h
	${CMAKE_CURRENT_SOURCE_DIR}/../common/ntv2gpucircularbuffer.cpp
	${CMAKE_CURRENT_SOURCE_DIR}/../common/ntv2gpucircularbuffer.h
	${CMAKE_CURRENT_SOURCE_DIR}/../common/ntv2rendertotexture.cpp
	${CMAKE_CURRENT_SOURCE_DIR}/../common/ntv2rendertotexture.h
	${CMAKE_CURRENT_SOURCE_DIR}/../common/ntv2texture.cpp
	${CMAKE_CURRENT_SOURCE_DIR}/../common/ntv2texture.h
	${CMAKE_CURRENT_SOURCE_DIR}/../common/oglview.cpp
	${CMAKE_CURRENT_SOURCE_DIR}/../common/oglview.h
	${CMAKE_CURRENT_SOURCE_DIR}/../common/opengl.h
	${CMAKE_CURRENT_SOURCE_DIR}/../common/simplegpuvio.cpp
	${CMAKE_CURRENT_SOURCE_DIR}/../common/simplegpuvio.h)

if (CMAKE_SYSTEM_NAME STREQUAL "Windows")
elseif (CMAKE_SYSTEM_NAME STREQUAL "Darwin")
    find_library(FOUNDATION_FRAMEWORK Foundation)
    set(TARGET_LINK_LIBS ${FOUNDATION_FRAMEWORK})
elseif (CMAKE_SYSTEM_NAME STREQUAL "Linux")
	set(TARGET_LINK_LIBS X11 OpenGL::GL OpenGL::GLU GLEW::GLEW dvp dl pthread rt)
	set(TARGET_LINK_DIRS
		${TARGET_LINK_DIRS}
		${CMAKE_CURRENT_SOURCE_DIR}/../../../thirdparty/nvidia/gpudirect/lib/linux/lib64)
endif()

set(TARGET_SOURCES
	${DVPLOWLATENCYDEMO_SOURCES}
	${NVIDIA_COMMON_SOURCES})

add_executable(dvplowlatencydemo ${TARGET_SOURCES})
add_dependencies(dvplowlatencydemo ajantv2)
target_include_directories(dvplowlatencydemo PRIVATE ${TARGET_INCLUDE_DIRS})
target_link_directories(dvplowlatencydemo PRIVATE ${TARGET_LINK_DIRS})
target_link_libraries(dvplowlatencydemo PRIVATE ajantv2 ${TARGET_LINK_LIBS})

install(TARGETS dvplowlatencydemo
	ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
	LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
	RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
	FRAMEWORK DESTINATION ${CMAKE_INSTALL_LIBDIR})
if (AJA_INSTALL_SOURCES)
	install(FILES ${DVPLOWLATENCYDEMO_HEADERS} 		DESTINATION ${CMAKE_INSTALL_PREFIX}/demos/NVIDIA/dvplowlatencydemo)
	install(FILES ${DVPLOWLATENCYDEMO_SOURCES} 		DESTINATION ${CMAKE_INSTALL_PREFIX}/demos/NVIDIA/dvplowlatencydemo)
	install(FILES ${DVPLOWLATENCYDEMO_BUILD_FILES} 	DESTINATION ${CMAKE_INSTALL_PREFIX}/demos/NVIDIA/dvplowlatencydemo)
	install(FILES ${NVIDIA_COMMON_SOURCES} 			DESTINATION ${CMAKE_INSTALL_PREFIX}/demos/NVIDIA/common)
	install(FILES ${NVIDIA_COMMONCUDA_SOURCES} 		DESTINATION ${CMAKE_INSTALL_PREFIX}/demos/NVIDIA/commonCUDA)
endif()
if (AJA_INSTALL_CMAKE)
	install(FILES CMakeLists.txt DESTINATION ${CMAKE_INSTALL_PREFIX}/demos/NVIDIA/dvplowlatencydemo)
endif()
if (AJA_INSTALL_MISC)
	install(FILES AJA_Logo_Small.png DESTINATION ${CMAKE_INSTALL_PREFIX}/demos/NVIDIA/dvplowlatencydemo)
endif()

if (AJA_CODE_SIGN)
    aja_code_sign(dvplowlatencydemo)
endif()
