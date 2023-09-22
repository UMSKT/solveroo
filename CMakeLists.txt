cmake_minimum_required(VERSION 3.12)
project(SOLVEROO LANGUAGES CXX CUDA)

# Set the C++ standard to C++11
set(CMAKE_CXX_STANDARD 11)

# Include CUDA
find_package(CUDA REQUIRED)

# Add source files
set(SOURCES
    SECPK1/IntGroup.cpp
    main.cpp
    SECPK1/Random.cpp
    Timer.cpp
    SECPK1/Int.cpp
    SECPK1/IntMod.cpp
    SECPK1/Point.cpp
    SECPK1/SECP256K1.cpp
    Kangaroo.cpp
    HashTable.cpp
    Thread.cpp
    Check.cpp
    Backup.cpp
    Network.cpp
    Merge.cpp
    PartMerge.cpp
)

# Include GPU source files if the "gpu" option is enabled
if (GPU)
    CUDA_ADD_LIBRARY(GPUEngine GPU/GPUEngine.cu)
    set(SOURCES ${SOURCES} GPU/GPUEngine.cu)
endif()

# Create an executable
add_executable(kangaroo ${SOURCES})

# Include directories
include_directories(${CUDA_INCLUDE_DIRS})

# Link libraries
if (NOT MSVC)
    target_link_libraries(kangaroo ${CUDA_LIBRARIES} pthread)
else()
    set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")
    set(CMAKE_CXX_FLAGS_RELEASE "/MT")
    set(CMAKE_CXX_FLAGS_DEBUG "/MTd")
    set(DEBUG ON)
endif()

# Compiler flags
if (GPU)
    if (DEBUG)
        target_compile_options(GPUEngine PRIVATE -G -maxrregcount=0 --ptxas-options=-v --compiler-options -fPIC -ccbin ${CXX} -m64 -g -v -I${CUDA_INCLUDE_DIRS} -gencode=arch=compute_${CCAP},code=sm_${CCAP})
    else()
        target_compile_options(GPUEngine PRIVATE -maxrregcount=0 --ptxas-options=-v --compiler-options -fPIC -ccbin ${CXX} -m64 -O3 -I${CUDA_INCLUDE_DIRS} -gencode=arch=compute_${CCAP},code=sm_${CCAP})
    endif()
endif()

if (MSVC)
    if (DEBUG)
        target_compile_options(kangaroo PRIVATE /W3 /Zi /EHsc)
        target_link_libraries(kangaroo ws2_32)
        set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} /DEBUG")
    else()
        target_compile_options(kangaroo PRIVATE /O3 /EHsc)
    endif()
else()
    if (DEBUG)
        target_compile_options(kangaroo PRIVATE -m64 -mssse3 -Wno-unused-result -Wno-write-strings -g -I${CUDA_INCLUDE_DIRS})
    else()
        target_compile_options(kangaroo PRIVATE -m64 -mssse3 -Wno-unused-result -Wno-write-strings -O2 -I${CUDA_INCLUDE_DIRS})
    endif()
endif()

# Create directories for object files
file(MAKE_DIRECTORY obj)
file(MAKE_DIRECTORY obj/SECPK1)
file(MAKE_DIRECTORY obj/GPU)
