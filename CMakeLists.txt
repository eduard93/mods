cmake_minimum_required(VERSION 2.8)

PROJECT(MODS)

add_definitions(-DA64)
add_definitions(-std=c++11)

SET(WITH_CAFFE FALSE)

message("Looking for OpenCV")
SET (OpenCV_DIR "/lib/opencv/build-release/")
#set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "/windows/mods/mods-video/binary_desc/brisk/")
FIND_PACKAGE( OpenCV  )
if (OPENCV_FOUND)
  message("Found!")
  set(OpenCV_CXX_FLAGS "-L/lib/opencv/build-release/lib/")
  include_directories(/lib/opencv/build-release/include)
  set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenCV_CXX_FLAGS}")
  link_directories(/lib/opencv/build-release/lib)
  
  #SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenCV_CXX_FLAGS}")
  set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenCV_CXX_FLAGS}")
  #include_directories(${OpenCV_INDLUDE_DIRS})
  message("${OpenCV_CXX_FLAGS}")
else(OPENCV_FOUND)
  message("Not found!")
  set(OpenCV_CXX_FLAGS "-L/lib/opencv/build-release/lib/")
include_directories(/lib/opencv/build-release/include)
  set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenCV_CXX_FLAGS}")
  link_directories(/lib/opencv/build-release/lib)
endif (OPENCV_FOUND)
#FIND_PACKAGE(CUDA REQUIRED)

enable_language (Fortran)
FIND_PACKAGE(LAPACK REQUIRED)
if (LAPACK_FOUND)
  SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${LAPACK_CXX_FLAGS}")
endif (LAPACK_FOUND)

FIND_PACKAGE(OpenMP)
if (OPENMP_FOUND)
  SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
  set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_CXX_FLAGS}")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
endif (OPENMP_FOUND)

SET(CMAKE_BUILD_TYPE "RELEASE")

IF(CMAKE_COMPILER_IS_GNUCXX)
  SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wno-strict-aliasing")
  SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wextra -Wno-write-strings")
  SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-deprecated -ansi")
  SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3 -ftree-vectorize -funroll-loops")
ENDIF(CMAKE_COMPILER_IS_GNUCXX)


if(WITH_CAFFE)
add_definitions(-DCPU_ONLY) #caffe cpu_only
include_directories(/home/old-ufo/caffe-clear/include)
include_directories(/home/old-ufo/caffe-clear/build/src/)
link_directories(/home/old-ufo/caffe-clear/build/lib)
endif(WITH_CAFFE)

link_directories(${CMAKE_SOURCE_DIR}/vlfeat/bin/glnxa64)

add_subdirectory( ./libdaisy z)
include_directories(
.
./libdaisy/include
./libdaisy/include/daisy
)

add_subdirectory( ./inih)
add_subdirectory( ./matutls)
add_subdirectory( ./detectors)
add_subdirectory( ./detectors/affinedetectors)
add_subdirectory( ./detectors/mser )
add_subdirectory( ./opensurf )
add_subdirectory( ./libMatch)
add_subdirectory( ./libNumerics)
add_subdirectory( ./akaze)
add_subdirectory( ./mrogh)
add_subdirectory( ./TILDE/c++)

####### DEGENSAC
include_directories(
.
./degensac)
set(degensac_srcs
        degensac/DegUtils.c
        degensac/exp_ranF.c
        degensac/exp_ranH.c
        degensac/Ftools.c
        degensac/hash.c
        degensac/Htools.c
        degensac/ranF.c
        degensac/ranH2el.c
        degensac/ranH.c
        degensac/rtools.c
        degensac/utools.c
        degensac/lapwrap.c
)
add_library(degensac ${degensac_srcs})
TARGET_LINK_LIBRARIES(degensac matutls lapack)
##### End of DEGENSAC

#Add vlfeat
include_directories(./vlfeat
./vlfeat/vl)

###################Matching
include_directories(.
./matching
 )

set(matching_srcs
        matching/siftdesc.cpp
        matching/matching.cpp
        orsa.cpp
)
add_library(matching ${matching_srcs})
TARGET_LINK_LIBRARIES(matching ${OpenCV_LIBS} degensac lapack Match Numerics)

################## View synthesis

set(synth-detection_srcs
        synth-detection.cpp)

include_directories(.)

add_library(synth-detection ${synth-detection_srcs})

if(WITH_CAFFE)
TARGET_LINK_LIBRARIES(synth-detection matching opensurf
mser AKAZE affinedetectors ${OpenCV_LIBS} helpers vl caffe glog protobuf lmdb blas)
else(WITH_CAFFE)
TARGET_LINK_LIBRARIES(synth-detection matching opensurf
mser affinedetectors AKAZE helpers vl ${OpenCV_LIBS})
endif(WITH_CAFFE)

#include_directories(.
#./dali)
#./smsld)


add_executable(mods mods.cpp correspondencebank.cpp imagerepresentation.cpp
io_mods.cpp  ssdesc-cpp-1.1.1/ssdesc.cc)#smsld/wzhlib.cpp smsld/descriptor.cpp smsld/Match.cpp smsld/Image.cpp)
if(WITH_CAFFE)
TARGET_LINK_LIBRARIES(mods inih matching synth-detection mser affinedetectors 
AKAZE vl mrogh TILDE caffe glog protobuf leveldb openblas  ${OpenCV_LIBS} vl m daisy z)
else(WITH_CAFFE)
TARGET_LINK_LIBRARIES(mods inih matching synth-detection mser affinedetectors  AKAZE vl mrogh TILDE 
${OpenCV_LIBS} vl m daisy z)
endif(WITH_CAFFE)

add_executable(extract_features extract_features.cpp correspondencebank.cpp imagerepresentation.cpp
io_mods.cpp  ssdesc-cpp-1.1.1/ssdesc.cc) # smsld/wzhlib.cpp smsld/descriptor.cpp smsld/Match.cpp smsld/Image.cpp)
if(WITH_CAFFE)
TARGET_LINK_LIBRARIES(extract_features inih matching synth-detection mser affinedetectors 
AKAZE vl mrogh TILDE  caffe glog protobuf leveldb openblas ${OpenCV_LIBS} ceigs fftw3 arpack  m cxsparse umfpack daisy z)
else(WITH_CAFFE)
TARGET_LINK_LIBRARIES(extract_features inih matching synth-detection mser affinedetectors  AKAZE vl mrogh TILDE 
${OpenCV_LIBS} m daisy z)
endif(WITH_CAFFE)

add_executable(extract_regions_for_benchmark extract_regions_for_benchmark correspondencebank.cpp
imagerepresentation.cpp io_mods.cpp  ssdesc-cpp-1.1.1/ssdesc.cc) # smsld/wzhlib.cpp smsld/descriptor.cpp smsld/Match.cpp smsld/Image.cpp)
if(WITH_CAFFE)
TARGET_LINK_LIBRARIES(extract_regions_for_benchmark inih matching synth-detection mser affinedetectors 
AKAZE vl mrogh TILDE  caffe glog protobuf leveldb openblas ${OpenCV_LIBS} ceigs fftw3 arpack  m cxsparse umfpack daisy z)
else(WITH_CAFFE)
TARGET_LINK_LIBRARIES(extract_regions_for_benchmark inih matching synth-detection mser affinedetectors  AKAZE vl mrogh TILDE 
${OpenCV_LIBS} m daisy z)
endif(WITH_CAFFE)

add_executable(export_descriptors export_descriptors correspondencebank.cpp
imagerepresentation.cpp io_mods.cpp  ssdesc-cpp-1.1.1/ssdesc.cc) # smsld/wzhlib.cpp smsld/descriptor.cpp smsld/Match.cpp smsld/Image.cpp)
if(WITH_CAFFE)
TARGET_LINK_LIBRARIES(export_descriptors inih matching synth-detection mser affinedetectors 
AKAZE vl mrogh TILDE caffe glog protobuf leveldb openblas  ${OpenCV_LIBS} ceigs fftw3 arpack  m cxsparse umfpack daisy z)
else(WITH_CAFFE )
TARGET_LINK_LIBRARIES(export_descriptors inih matching synth-detection mser affinedetectors  AKAZE vl mrogh TILDE 
 opencv_core m daisy z)
endif(WITH_CAFFE)
#${OpenCV_LIBS}
add_executable(mods_multi mods_multi.cpp correspondencebank.cpp
imagerepresentation.cpp io_mods.cpp  ssdesc-cpp-1.1.1/ssdesc.cc) # smsld/wzhlib.cpp smsld/descriptor.cpp smsld/Match.cpp smsld/Image.cpp)
if(WITH_CAFFE)
TARGET_LINK_LIBRARIES(mods_multi inih matching synth-detection mser affinedetectors 
AKAZE vl mrogh TILDE caffe glog protobuf leveldb openblas  ${OpenCV_LIBS} ceigs fftw3 arpack  m cxsparse umfpack daisy z)
else(WITH_CAFFE)
TARGET_LINK_LIBRARIES(mods_multi inih matching synth-detection mser affinedetectors  AKAZE vl mrogh TILDE 
${OpenCV_LIBS} m daisy z)
endif(WITH_CAFFE)

