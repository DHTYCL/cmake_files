#LandmarkDetector library
include_directories(../../local/LandmarkDetector/include)

#Utilities library
include_directories(../../local/Utilities/include)


SET(SOURCE
    src/Face_utils.cpp
	src/FaceAnalyser.cpp
	src/FaceAnalyserParameters.cpp
	src/SVM_dynamic_lin.cpp
	src/SVM_static_lin.cpp
	src/SVR_dynamic_lin_regressors.cpp
	src/SVR_static_lin_regressors.cpp
)

SET(HEADERS
    include/Face_utils.h	
	include/FaceAnalyser.h
	include/FaceAnalyserParameters.h
	include/SVM_dynamic_lin.h
	include/SVM_static_lin.h
	include/SVR_dynamic_lin_regressors.h
	include/SVR_static_lin_regressors.h
)

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

add_library( FaceAnalyser SHARED ${SOURCE} ${HEADERS})
add_library( OpenFace::FaceAnalyser ALIAS FaceAnalyser)
target_include_directories(FaceAnalyser PUBLIC ${FACEANALYSER_SOURCE_DIR}/include)
target_include_directories(FaceAnalyser PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:$<INSTALL_PREFIX>/include/OpenFace>)


target_include_directories(FaceAnalyser PUBLIC ${TBB_ROOT_DIR}/include)
target_include_directories(FaceAnalyser PUBLIC ${Boost_INCLUDE_DIRS})
target_include_directories(FaceAnalyser PUBLIC ${Boost_INCLUDE_DIRS}/boost)
target_include_directories(FaceAnalyser PUBLIC ${OpenCV_INCLUDE_DIRS})

target_link_libraries(FaceAnalyser PUBLIC ${OpenCV_LIBS} ${Boost_LIBRARIES} ${TBB_LIBRARIES} ${OpenBLAS_LIB})
target_link_libraries(FaceAnalyser PUBLIC dlib::dlib)

target_include_directories(FaceAnalyser PUBLIC ${OpenBLAS_INCLUDE_DIR})

install (TARGETS FaceAnalyser EXPORT OpenFaceTargets LIBRARY DESTINATION lib ARCHIVE DESTINATION lib)
install (FILES ${HEADERS} DESTINATION include/OpenFace)
