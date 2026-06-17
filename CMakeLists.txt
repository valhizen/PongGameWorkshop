cmake_minimum_required(VERSION 3.10)

if(POLICY CMP0141)
  cmake_policy(SET CMP0141 NEW)
  set(CMAKE_MSVC_DEBUG_INFORMATION_FORMAT "$<IF:$<AND:$<C_COMPILER_ID:MSVC>,$<CXX_COMPILER_ID:MSVC>>,$<$<CONFIG:Debug,RelWithDebInfo>:EditAndContinue>,$<$<CONFIG:Debug,RelWithDebInfo>:ProgramDatabase>>")
endif()

project("PongGameWorkshop")

find_package(OpenGL REQUIRED)

if(WIN32)
    set(SDL3_ROOT "${CMAKE_SOURCE_DIR}/external/sdl")

    add_library(SDL3::SDL3 SHARED IMPORTED)
    set_target_properties(SDL3::SDL3 PROPERTIES
        IMPORTED_IMPLIB               "${SDL3_ROOT}/lib/x64/SDL3.lib"
        IMPORTED_LOCATION             "${SDL3_ROOT}/lib/x64/SDL3.dll"
        INTERFACE_INCLUDE_DIRECTORIES "${SDL3_ROOT}/include"
    )

    function(copy_sdl3_dll target)
        add_custom_command(TARGET ${target} POST_BUILD
            COMMAND ${CMAKE_COMMAND} -E copy_if_different
                "${SDL3_ROOT}/lib/x64/SDL3.dll"
                "$<TARGET_FILE_DIR:${target}>/SDL3.dll"
        )
    endfunction()

else()
    find_package(SDL3 QUIET)

    if(NOT SDL3_FOUND)
        find_package(PkgConfig REQUIRED)
        pkg_check_modules(SDL3 REQUIRED sdl3)

        add_library(SDL3::SDL3 INTERFACE IMPORTED)
        set_target_properties(SDL3::SDL3 PROPERTIES
            INTERFACE_INCLUDE_DIRECTORIES "${SDL3_INCLUDE_DIRS}"
            INTERFACE_LINK_LIBRARIES      "${SDL3_LIBRARIES}"
            INTERFACE_COMPILE_OPTIONS     "${SDL3_CFLAGS_OTHER}"
        )
        target_link_directories(SDL3::SDL3 INTERFACE ${SDL3_LIBRARY_DIRS})
    endif()

    function(copy_sdl3_dll target)
    endfunction()
endif()

set(GLAD_DIR    "${CMAKE_SOURCE_DIR}/external/glad")
set(GLAD_SOURCE "${GLAD_DIR}/src/glad.c")
set(GLM_DIR     "${CMAKE_SOURCE_DIR}/external/glm")
set(IMGUI_DIR   "${CMAKE_SOURCE_DIR}/external/imgui")
set(IMGUI_SOURCES
    ${IMGUI_DIR}/imgui.cpp
    ${IMGUI_DIR}/imgui_demo.cpp
    ${IMGUI_DIR}/imgui_draw.cpp
    ${IMGUI_DIR}/imgui_tables.cpp
    ${IMGUI_DIR}/imgui_widgets.cpp
    ${IMGUI_DIR}/backends/imgui_impl_opengl3.cpp
    ${IMGUI_DIR}/backends/imgui_impl_sdl3.cpp
)

set(SHADER_HPP "src/Shader.hpp")

add_library(CommonSettings INTERFACE)
target_include_directories(CommonSettings INTERFACE
    "src"
    ${IMGUI_DIR}
    ${IMGUI_DIR}/backends
    ${GLAD_DIR}/include
    ${GLM_DIR}
)
target_link_libraries(CommonSettings INTERFACE OpenGL::GL SDL3::SDL3)

if(UNIX AND NOT APPLE)
    target_link_libraries(CommonSettings INTERFACE dl pthread)
endif()

add_executable(Window     "src/Window/main.cpp")
target_link_libraries(Window PRIVATE CommonSettings)
copy_sdl3_dll(Window)

add_executable(Input      "src/Input/main.cpp")
target_link_libraries(Input PRIVATE CommonSettings)
copy_sdl3_dll(Input)

add_executable(OpenGlInit "src/OpenGlInit/main.cpp" ${GLAD_SOURCE})
target_link_libraries(OpenGlInit PRIVATE CommonSettings)
copy_sdl3_dll(OpenGlInit)

add_executable(ImGui      "src/ImGui/main.cpp" ${GLAD_SOURCE} ${IMGUI_SOURCES})
target_link_libraries(ImGui PRIVATE CommonSettings)
copy_sdl3_dll(ImGui)

add_executable(Triangle   "src/Triangle/main.cpp" ${SHADER_HPP} ${GLAD_SOURCE} ${IMGUI_SOURCES})
target_link_libraries(Triangle PRIVATE CommonSettings)
copy_sdl3_dll(Triangle)

add_executable(Circle     "src/Circle/main.cpp" ${SHADER_HPP} ${GLAD_SOURCE} ${IMGUI_SOURCES})
target_link_libraries(Circle PRIVATE CommonSettings)
copy_sdl3_dll(Circle)

add_executable(MainLoop   "src/MainLoop/main.cpp" ${GLAD_SOURCE} ${IMGUI_SOURCES})
target_link_libraries(MainLoop PRIVATE CommonSettings)
copy_sdl3_dll(MainLoop)

add_executable(Pong       "src/Pong/main.cpp" ${SHADER_HPP} ${GLAD_SOURCE} ${IMGUI_SOURCES})
target_link_libraries(Pong PRIVATE CommonSettings)
copy_sdl3_dll(Pong)