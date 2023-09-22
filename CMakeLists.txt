cmake_minimum_required(VERSION 3.12)
project(SOLVEROO LANGUAGES CXX CUDA)

# Set the C++ standard to C++11
set(CMAKE_CXX_STANDARD 11)

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
    set(CMAKE_CUDA_ARCHITECTURES ${CCAP})
    add_library(GPUEngine GPU/GPUEngine.cu)
    set_property(TARGET GPUEngine PROPERTY CUDA_RESOLVE_DEVICE_SYMBOLS ON)
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
    set(DEBUG ON)
endif()

# Compiler flags
if (GPU)
    target_link_libraries(kangaroo GPUEngine)
    target_compile_definitions(kangaroo PRIVATE WITHGPU)
    if (DEBUG)
        target_compile_options(GPUEngine PRIVATE -G -maxrregcount=0 --ptxas-options=-v -m64 -g)
    else()
        target_compile_options(GPUEngine PRIVATE -maxrregcount=0 --ptxas-options=-v -m64 -O3)
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
        target_compile_options(kangaroo PRIVATE -m64 -mssse3 -Wno-unused-result -Wno-write-strings -g)
    else()
        target_compile_options(kangaroo PRIVATE -m64 -mssse3 -Wno-unused-result -Wno-write-strings -O3)
    endif()
endif()
