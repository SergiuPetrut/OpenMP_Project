CMAKE_MINIMUM_REQUIRED(VERSION 3.9)
project(Satellites)

add_executable(parallel parallel.c)


# Here is an example syntax how to add compiler options to your build process
# See the project work document on compiler flag syntax on Linux and Windows

# target_compile_options(parallel PRIVATE "add-your-option-here")
# target_compile_options(parallel PRIVATE "add-your-second-option-here")


# Added vectorization info for GCC here ; Question 6.1.3

target_compile_options(parallel PRIVATE "-fopt-info-vec")

#Question 4 : Added the following flags : 

#Fastest optimization 
target_compile_options(parallel PRIVATE "-Ofast")
#target_compile_options(parallel PRIVATE "-O3")

#This automatically enables appropriate SIMD instruction sets, so you might not need to explicitly specify -mavx, -mavx2, etc.,
#unless you are compiling for a different target than the host


#SIMD FLAGS
#the march for the processor basically does the same
#as mavx512f
target_compile_options(parallel PRIVATE "-march=znver4")

#target_compile_options(parallel PRIVATE "-mavx512f")

target_compile_options(parallel PRIVATE "-ftree-vectorize")
target_compile_options(parallel PRIVATE "-funroll-loops")


#This flag tells the compiler to tune the performance of the generated code for the host CPU without changing the architecture.
#It adjusts scheduling and other low-level details to make the most out of the specific CPU model.

target_compile_options(parallel PRIVATE "-mtune=znver4")

#This flag allows the use of both the x87 and SSE units for floating-point math. 
#It can improve performance in some cases by allowing better instruction scheduling.

#target_compile_options(parallel PRIVATE "-mfpmath=both")


target_compile_options(parallel PRIVATE "-ftree-parallelize-loops=16")

#FP FLAGS
target_compile_options(parallel PRIVATE "-ffast-math")
target_compile_options(parallel PRIVATE "-ffp-contract=fast")
target_compile_options(parallel PRIVATE "-fno-math-errno")

target_compile_options(parallel PRIVATE "-flto")



# UNCOMMENT THESE TO ENABLE OPENMP
#
 find_package(OpenMP REQUIRED)
 target_link_libraries(parallel OpenMP::OpenMP_C)


# UNCOMMENT THESE TO ENABLE OPENCL
# This will also copy the kernel source file parallel.cl (created by you) to the build directory
# The copying command is unfortunately not perfect, as it doesn't redo the copy if you only edit
# the parallel.cl, but leave the parallel.c untouched.
# Because of this, you might need to force 'Rebuild All' to ensure kernel code updates propagate
# to the build directory. If you know enough CMake magic, feel free to fix this and let me know.
#
# find_package(OpenCL REQUIRED)
# target_include_directories(parallel PRIVATE ${OpenCL_INCLUDE_DIRS})
# target_link_libraries(parallel ${OpenCL_LIBRARIES})
# add_custom_command(
#     TARGET parallel POST_BUILD
#     COMMAND ${CMAKE_COMMAND} -E copy_if_different
#     "${CMAKE_SOURCE_DIR}/parallel.cl"
#     $<TARGET_FILE_DIR:parallel>
#     VERBATIM)


# Find and link SDL2
if (WIN32)
    set(SDL2_DIR "${CMAKE_SOURCE_DIR}/SDL2-2.30.7/cmake")
endif()
find_package(SDL2 REQUIRED)
target_include_directories(parallel PRIVATE ${SDL2_INCLUDE_DIR})
target_link_libraries(parallel ${SDL2_LIBRARIES})

if (WIN32)
# copy the sdl2 .dll file to the same folder as the executable
add_custom_command(
    TARGET parallel POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
    $<TARGET_FILE:SDL2::SDL2>
    $<TARGET_FILE_DIR:parallel>
    VERBATIM)
else()
    # Math library shouldn't be linked on Windows, but must be linked on Linux
    target_link_libraries(parallel m)
endif()
