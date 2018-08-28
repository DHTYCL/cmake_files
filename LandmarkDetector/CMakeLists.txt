
#Utilities library
include_directories(../../local/Utilities/include)
include_directories(/home/aitian/OpenFace/lib/3rdParty/OpenBLAS/include)
set(CMAKE_CONFIG_DIR etc/OpenFace)
MESSAGE("aitian find CMAKE_CONFIG_DIR:  ${CMAKE_CONFIG_DIR}")

SET(CMAKE_CXX_FLAGS -fPIC)
SET(MAKE_C_FLAGS -fPIC)
SET(CMAKE_EXE_LINKER_FLAGS -fPIC)
SET(SOURCE
    src/CCNF_patch_expert.cpp
	src/CEN_patch_expert.cpp
	src/CNN_utils.cpp
	src/FaceDetectorMTCNN.cpp
	src/LandmarkDetectionValidator.cpp
    src/LandmarkDetectorFunc.cpp
	src/LandmarkDetectorModel.cpp
    src/LandmarkDetectorUtils.cpp
	src/LandmarkDetectorParameters.cpp
	src/Patch_experts.cpp
	src/PAW.cpp
    src/PDM.cpp
	src/SVR_patch_expert.cpp
	src/stdafx.cpp
)

SET(HEADERS
    include/CCNF_patch_expert.h	
	include/CEN_patch_expert.h
    include/CNN_utils.h
	include/FaceDetectorMTCNN.h
    include/LandmarkCoreIncludes.h
	include/LandmarkDetectionValidator.h
    include/LandmarkDetectorFunc.h
	include/LandmarkDetectorModel.h
	include/LandmarkDetectorParameters.h
	include/LandmarkDetectorUtils.h
	include/Patch_experts.h	
    include/PAW.h
	include/PDM.h
	include/SVR_patch_expert.h		
	include/stdafx.h
)


find_package( Boost 1.5.9 REQUIRED COMPONENTS filesystem system)
if(${Boost_FOUND})
	MESSAGE("Boost information:") 
	MESSAGE("  Boost_VERSION: ${Boost_VERSION}")
	MESSAGE("  Boost_INCLUDE_DIRS: ${Boost_INCLUDE_DIRS}") 
	MESSAGE("  Boost_LIBRARIES: ${Boost_LIBRARIES}") 
	MESSAGE("  Boost_LIBRARY_DIRS: ${Boost_LIBRARY_DIRS}") 
else()
    MESSAGE(FATAL_ERROR "Boost not found in the system.")
endif()

if (${CMAKE_CXX_COMPILER_ID} STREQUAL "GNU")
    execute_process(COMMAND ${CMAKE_CXX_COMPILER} -dumpversion OUTPUT_VARIABLE GCC_VERSION)
    if (GCC_VERSION VERSION_LESS 4.7)
        set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x -msse -msse2 -msse3")
    else ()
        set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -msse -msse2 -msse3")
    endif ()
else ()
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -msse -msse2 -msse3")
endif ()

# dlib
find_package(dlib 19.13)
if(${dlib_FOUND})
    message("dlib information:")
    message("  dlib version: ${dlib_VERSION}")

    if (NOT TARGET dlib)
        add_library(dlib INTERFACE IMPORTED GLOBAL)
    endif()
else()
    message(FATAL_ERROR "dlib not found in the system, please install dlib")
endif()

# suppress auto_ptr deprecation warnings
if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang" OR "${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
    add_compile_options("-Wno-deprecated-declarations")
endif()

add_library( LandmarkDetector ${SOURCE} ${HEADERS} )
add_library( OpenFace::LandmarkDetector ALIAS LandmarkDetector)

target_include_directories(LandmarkDetector PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:$<INSTALL_PREFIX>/include/OpenFace>)

target_include_directories(LandmarkDetector PRIVATE ${LandmarkDetector_SOURCE_DIR}/include)
target_include_directories(LandmarkDetector PUBLIC ${TBB_ROOT_DIR}/include)
target_include_directories(LandmarkDetector PUBLIC ${Boost_INCLUDE_DIRS})
target_include_directories(LandmarkDetector PUBLIC ${Boost_INCLUDE_DIRS}/boost)
target_include_directories(LandmarkDetector PUBLIC ${OpenCV_INCLUDE_DIRS})

target_link_libraries(LandmarkDetector PUBLIC ${OpenCV_LIBS} ${Boost_LIBRARIES} ${TBB_LIBRARIES} ${OpenBLAS_LIB})
target_link_libraries(LandmarkDetector PUBLIC dlib::dlib)

target_include_directories(LandmarkDetector PRIVATE ${OpenBLAS_INCLUDE_DIR})

install (TARGETS LandmarkDetector EXPORT OpenFaceTargets LIBRARY DESTINATION lib ARCHIVE DESTINATION lib)
install (FILES ${HEADERS} DESTINATION include/OpenFace)
