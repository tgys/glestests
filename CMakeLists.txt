cmake_minimum_required(VERSION 3.0)
project(GLES_X11)

set_property(GLOBAL PROPERTY USE_FOLDERS ON)

set(CMAKE_FLAGS "${CMAKE_FLAGS} -Wall -Wl --start-group")

# Set directory paths
set(SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src)
set(BIN_DIR ${CMAKE_CURRENT_SOURCE_DIR}/bin)
set(LIBS_DIR ${CMAKE_CURRENT_SOURCE_DIR}/libs)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${BIN_DIR})

function(update_submodule module)

    find_package(Git REQUIRED)

    if(NOT EXISTS ${LIBS_DIR}/${module})
        execute_process(COMMAND ${GIT_EXECUTABLE} submodule update --init --recursive -- ${LIBS_DIR}/${module}
                WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
                COMMAND_ERROR_IS_FATAL ANY)
    endif()

    #string(COMPARE EQUAL ${module} "glad" isglad)
    #if(isglad)
    #    execute_process(COMMAND ${GIT_EXECUTABLE} checkout glad2
    #            WORKING_DIRECTORY ${LIBS_DIR}/glad
    #            COMMAND_ERROR_IS_FATAL ANY)
    #endif()

    if(EXISTS ${LIBS_DIR}/${module}/CMakeLists.txt)
        add_subdirectory(${LIBS_DIR}/${module})
    endif()

endfunction(update_submodule)

update_submodule(glfw)
#update_submodule(glad)
update_submodule(glm)
update_submodule(stb)
update_submodule(spdlog)

set(COMMON_SRCS
        ${SOURCE_DIR}/common/main_egl.c
        ${SOURCE_DIR}/common/bmp.c
        ${SOURCE_DIR}/common/cmdline.c
        ${SOURCE_DIR}/common/common.c
        ${SOURCE_DIR}/common/maths.c
        ${SOURCE_DIR}/common/vertexbuffergenerator.cpp
)

configure_file(${SOURCE_DIR}/include/conf/root_directory.h.in ${SOURCE_DIR}/include/root_directory.h)

find_package(OpenGL REQUIRED)
find_package(X11 REQUIRED)
if(NOT ${X11_FOUND})
    message(FATAL_ERROR "X11 libraries could not be found. Please try setting: -DCMAKE_PREFIX_PATH pointing towards your X11 libraries")
endif()

add_definitions(${OPENGL_DEFINITIONS})
set(TEST_NAMES
        "eglsampleNSDK"
        "helloapi"
        "transform_feedback_1_stream_1_buffer"
        "transform_feedback_1_stream_4_buffer"
        "cssampling"
        "point4"
        "depthvarying"
        "pvrtc_smalltextures"
        "colormask"
        "large_uniform_buffer"
        "fbo_render_mipmap"
        "blending_min_max"
        "map_buffer"
        "rgb8_rgba8"
        "image_load_store_compute"
        "int_2_10_10_10_rev_vertex"
        "frag_depth"
        "shadow_comparison"
        "multiple_render_targets"
        "row_major_matrices"
        "max_vertex_textures"
        "explicit_attrib_location"
        "instanced_arrays"
        "interpolate"
        "uniform_buffers"
        "texture_arrays"
        "msaaresolve"
        "dynamic_constants"
        "dynamic_objects"
        "zloadstore"
        "zloadstore_msaa"
        "large_instanced_arrays"
        "sbo_instruction"
        "two_sided_stencil"
        "primitiverestart"
        "depthaccuracy"
        "zlsalphatestintersectingtris"
        "texture_stress"
        "mixed_fbo"
        "oes_vertex_half_float"
        "oes_vertex_array_object"
        "milestone0"
        "milestone1"
        "milestone2"
        "milestone3"
        "mrt_mixed_types"
        "mrt_many_fbo"
        "fbo_mipgen"
        "uniquePixels"
        "scratchh"
)


find_library(astc_lib astc ${CMAKE_CURRENT_SOURCE_DIR}/SDKs/ASTC/lib64)
find_library(egl EGL ${CMAKE_CURRENT_SOURCE_DIR}/SDKs/Native_SDK/lib/Linux_x86_64/)
find_library(pvrtexlib PVRTexLib ${CMAKE_CURRENT_SOURCE_DIR}/SDKs/PVRTexNew/lib64/)
#find_library(spdlog )

#add_subdirectory("${LIBS_DIR}/glad/cmake" glad_cmake)
#glad_add_library(glad_egl REPRODUCIBLE LOADER API egl=1.5 gles2=2.0)

set(INCLUDE_DIRS
        ${LIBS_DIR}/glm/glm
       # ${LIBS_DIR}/glfw/include
       # ${LIBS_DIR}/stb/
        ${SOURCE_DIR}/include
        ${CMAKE_CURRENT_SOURCE_DIR}/SDKs/ASTC/include
        ${CMAKE_CURRENT_SOURCE_DIR}/SDKs/Native_SDK/include
        ${CMAKE_CURRENT_SOURCE_DIR}/SDKs/Native_SDK/include/EGL
        ${CMAKE_CURRENT_SOURCE_DIR}/SDKs/Native_SDK/include/GLES
        ${CMAKE_CURRENT_SOURCE_DIR}/SDKs/Native_SDK/include/GLES2
        ${CMAKE_CURRENT_SOURCE_DIR}/SDKs/Native_SDK/include/GLES3
        ${CMAKE_CURRENT_SOURCE_DIR}/SDKs/PVRTexNew/include
       # ${X11_INCLUDE_DIR}
)

set(LIBS ${CMAKE_DL_LIBS})
list(APPEND LIBS
        ${glm}
        ${glfw}
      #  glad_egl
        spdlog
        ${astc_lib}
        ${egl}
        ${gles}
        ${gles2}
        ${gles3}
        ${pvrtexlib}
        ${X11_LIBRARIES}
)

foreach(X IN LISTS TEST_NAMES)
    message(STATUS "building ${X}")
    message(STATUS "${SOURCE_DIR}/${X}/${X}.cpp")
    file(GLOB_RECURSE ${X}_SOURCE ${SOURCE_DIR}/${X}/*.cpp)

    string(COMPARE EQUAL ${X} "eglsampleNSDK" is_sample)
    string(COMPARE EQUAL ${X} "helloapi" is_helloapi)
    if(is_sample OR is_helloapi)
        add_executable(${X} ${${X}_SOURCE})
    else()
        add_executable(${X} ${COMMON_SRCS} ${${X}_SOURCE})
    endif()

    target_compile_definitions(${X} PUBLIC -DTEST_NAME="${X}")

####    target_include_directories(${X} PUBLIC ${LIBS_DIR}/glad/include)
####    target_link_libraries(${X} PUBLIC glad)
####
####    target_include_directories(${X} PUBLIC ${LIBS_DIR}/assimp/include)
####    target_link_libraries(${X} PUBLIC assimp)

    #target_include_directories(${X} PUBLIC ${SOURCE_DIR}/pvrtc)
    #link_directories(${SOURCE_DIR}/pvrtc/lib64)
    #target_link_libraries(${X} LINK_PUBLIC PVRTC)

    target_include_directories(${X} PUBLIC ${INCLUDE_DIRS})
    target_link_libraries(${X} LINK_PUBLIC ${LIBS})

    if (OpenGL_FOUND)
        target_include_directories(${X} PRIVATE ${OPENGL_INCLUDE_DIRS})
        target_link_libraries(${X} PRIVATE ${OPENGL_LIBRARIES})
    else()
        message(FATAL_ERROR "OpenGL was not found on the system")
    endif()

    set_target_properties(${X} PROPERTIES
            CXX_STANDARD 20
            CXX_STANDARD_REQUIRED YES
            CXX_EXTENSIONS NO)

    # Set project folders
    set_target_properties(${X} PROPERTIES FOLDER ${SOURCE_DIR}/${X})

endforeach()

