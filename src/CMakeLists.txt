# find all dependency libraries:
find_package(OpenMP)
if (NOT OPENMP_FOUND)
	message(WARNING "OpenMP is not found. Will be slow.")
else()
	set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
endif()


find_package(JPEG)
if(JPEG_FOUND)
	include_directories(${JPEG_INCLUDE_DIR})
endif()

find_package(Eigen3 REQUIRED)
include_directories(${CMAKE_CURRENT_SOURCE_DIR})
add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/third-party/)

# ---------------------------------------------------------------------------------------------

# find sources
file(GLOB SOURCES feature/*.cc stitch/*.cc lib/*.cc)

# ---------------------------------------------------------------------------------------------

add_library(openpano ${SOURCES})	# TODO shared?
target_link_libraries(openpano ${JPEG_LIBRARY} lodepng Eigen3::Eigen ${OpenCV_LIBS})

add_executable(image-stitching main.cc)
target_link_libraries(image-stitching openpano ${OpenCV_LIBS})

# ---------------------------------------------------------------------------------------------
# tests

add_executable(test_lib_geometry tests/test_lib_geometry.cc)
target_link_libraries(test_lib_geometry openpano)

add_executable(test_feature_gaussian tests/test_feature_gaussian.cc)
target_link_libraries(test_feature_gaussian openpano)

add_executable(test_stitch_multiband tests/test_stitch_multiband.cc)
target_link_libraries(test_stitch_multiband openpano)

# ---------------------------------------------------------------------------------------------

if (NOT JPEG_FOUND)
	target_compile_definitions(openpano PUBLIC DISABLE_JPEG)
	target_compile_definitions(image-stitching PUBLIC DISABLE_JPEG)
endif()

find_package(PythonLibs)
if(PYTHONLIBS_FOUND AND BUILD_SHARED_LIBS)
	add_library(pyopenpano SHARED python/pybind.cc)
	target_link_libraries(pyopenpano openpano pybind11)
	target_include_directories(pyopenpano SYSTEM
		PRIVATE ${PYTHON_INCLUDE_DIRS}
	)
endif()
