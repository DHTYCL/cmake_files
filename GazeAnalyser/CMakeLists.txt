#LandmarkDetector library
include_directories(../../local/LandmarkDetector/include)

#Utilities library
include_directories(../../local/Utilities/include)

SET(SOURCE
    src/GazeEstimation.cpp
)

SET(HEADERS
    include/GazeEstimation.h
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

add_library( GazeAnalyser  SHARED ${SOURCE} ${HEADERS})
add_library( OpenFace::GazeAnalyser ALIAS GazeAnalyser)

target_include_directories(GazeAnalyser PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:$<INSTALL_PREFIX>/include/OpenFace>)

target_include_directories(GazeAnalyser PRIVATE ${GAZEANALYSER_SOURCE_DIR}/include)
target_include_directories(GazeAnalyser PUBLIC ${TBB_ROOT_DIR}/include)
target_include_directories(GazeAnalyser PUBLIC ${Boost_INCLUDE_DIRS})
target_include_directories(GazeAnalyser PUBLIC ${Boost_INCLUDE_DIRS}/boost)
target_include_directories(GazeAnalyser PUBLIC ${OpenCV_INCLUDE_DIRS})

target_link_libraries(GazeAnalyser PUBLIC ${OpenCV_LIBS} ${Boost_LIBRARIES} ${TBB_LIBRARIES} ${OpenBLAS_LIB})
target_link_libraries(GazeAnalyser PUBLIC dlib::dlib)

target_include_directories(GazeAnalyser PRIVATE ${OpenBLAS_INCLUDE_DIR})

install (TARGETS GazeAnalyser EXPORT OpenFaceTargets LIBRARY DESTINATION lib ARCHIVE DESTINATION lib)
install (FILES ${HEADERS} DESTINATION include/OpenFace)
