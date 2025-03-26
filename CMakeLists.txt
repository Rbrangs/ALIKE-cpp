cmake_minimum_required(VERSION 3.0)
project(ALIKE_cpp LANGUAGES CXX CUDA)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_BUILD_TYPE "Release")

### CUDA 설정
find_package(CUDAToolkit REQUIRED PATHS /usr/local/cuda)
include_directories(${CUDAToolkit_INCLUDE_DIRS})

# 최신 GPU 아키텍처 설정
# CUDA 아키텍처를 명시적으로 지정하여 'compute_35' 아키텍처를 사용하지 않도록 설정
set(CMAKE_CUDA_ARCHITECTURES 75 80 86)  # RTX 20xx, 30xx, 40xx GPU 아키텍처
set(CUDA_NVCC_FLAGS "${CUDA_NVCC_FLAGS} -arch=sm_75 -Wno-deprecated-gpu-targets")

### OpenMP
find_package(OpenMP)
include_directories(${OpenMP_INCLUDE_DIRS})

### Torchlib
find_package(Torch REQUIRED PATHS ${PROJECT_SOURCE_DIR}/libtorch)
message(STATUS "Found Torch include is ${TORCH_INCLUDE_DIRS}")
include_directories(${TORCH_INCLUDE_DIRS})
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${TORCH_CXX_FLAGS}")

### OpenCV
find_package(OpenCV REQUIRED)
message(STATUS "Found OpenCV version is ${OpenCV_VERSION}")
message(STATUS "Found OpenCV include is ${OpenCV_INCLUDE_DIRS}")
include_directories(${OpenCV_INCLUDE_DIRS})

# CUDA 라이브러리 빌드
cuda_add_library(get_patches_cuda SHARED src/get_patches.cu)
set_target_properties(get_patches_cuda PROPERTIES CUDA_SEPARABLE_COMPILATION ON)

add_library(get_patches SHARED src/get_patches.cpp)
target_link_libraries(get_patches get_patches_cuda)

# 실행 파일 빌드
include_directories(include)
add_executable(demo
        src/main.cpp
        src/utils.cpp)
target_link_libraries(demo ${TORCH_LIBRARIES} ${OpenCV_LIBS} get_patches)
