cmake_minimum_required(VERSION 3.16.3)
#project(ALIKE_cpp LANGUAGES CXX CUDA)
set(CMAKE_CUDA_STANDARD 14)  # CUDA 14 사용 (CUDA 12.1과 호환)
set(CMAKE_CUDA_STANDARD_REQUIRED ON)

set(CMAKE_BUILD_TYPE "Release")
set(CMAKE_CUDA_ARCHITECTURES 75)

### CUDA
find_package(CUDA PATHS /usr/local/cuda)
include_directories(${CUDA_INCLUDE_DIRS})
find_package(Threads REQUIRED)
###

### OpenMP
find_package(OpenMP)
include_directories(${OpenMP_INCLUDE_DIRS})
###

### Torchlib
find_package(Torch REQUIRED PATHS ${PROJECT_SOURCE_DIR}/libtorch)
message(STATUS "Found Torch include is ${TORCH_INCLUDE_DIRS}")
include_directories(${TORCH_INCLUDE_DIRS})
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${TORCH_CXX_FLAGS}")
set(CUDA_NVCC_FLAGS "${CUDA_NVCC_FLAGS} -Wno-deprecated-gpu-targets")
###

### OPENCV
find_package(OpenCV REQUIRED)
message(STATUS "Found OpenCV version is ${OpenCV_VERSION}")
message(STATUS "Found OpenCV include is ${OpenCV_INCLUDE_DIRS}")
include_directories(${OpenCV_INCLUDE_DIRS})
###

# the library that only has the cuda code
cuda_add_library(get_patches_cuda SHARED src/get_patches.cu)
set_target_properties(get_patches_cuda PROPERTIES CUDA_SEPARABLE_COMPILATION ON)

add_library(get_patches SHARED src/get_patches.cpp)
target_link_libraries(get_patches get_patches_cuda)

include_directories(include)
add_executable(demo
        src/main.cpp
        src/utils.cpp)
target_link_libraries(demo PUBLIC Threads::Threads ${TORCH_LIBRARIES} ${OpenCV_LIBS} get_patches)
