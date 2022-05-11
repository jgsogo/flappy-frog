cmake_minimum_required(VERSION 3.15)

find_package(SDL2 REQUIRED CONFIG)
find_package(box2d REQUIRED CONFIG)
# FIXME: sdl_image conan package still not adding properties
find_package(sdl_image REQUIRED CONFIG)
find_package(CLI11 REQUIRED CONFIG)
find_package(fmt REQUIRED CONFIG)
find_package(tinkerforge-bindings REQUIRED CONFIG)

add_executable(mygame main.cpp
                      Frog.cpp
                      Pipe.cpp
                      Game.cpp
                      World.cpp
                      Frog.hpp
                      Pipe.hpp
                      Game.hpp
                      World.hpp)
target_include_directories(mygame PRIVATE include)

# FIXME: sdl_image conan package still not adding properties
target_link_libraries(mygame CLI11::CLI11 fmt::fmt box2d::box2d SDL2::SDL2main sdl_image::sdl_image tinkerforge::bindings)

install(TARGETS mygame
        PUBLIC_HEADER DESTINATION include
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib)

install(FILES
#        ${MAGNUM_EMSCRIPTENAPPLICATION_JS}
#        ${CMAKE_CURRENT_SOURCE_DIR}/resources/favicon.ico
        $<TARGET_FILE_DIR:mygame>/mygame.js
        $<TARGET_FILE_DIR:mygame>/mygame.wasm
    DESTINATION ${CMAKE_INSTALL_PREFIX}/bin)

file(GLOB ASSETS "${CMAKE_SOURCE_DIR}/assets/*.png")

include(${CMAKE_SOURCE_DIR}/cmake/emscripten.cmake)
foreach(asset ${ASSETS})
        file(RELATIVE_PATH assetRelativePath ${CMAKE_SOURCE_DIR}/assets/ ${asset})
        add_custom_command(TARGET mygame POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy_if_different
                ${asset}
                $<TARGET_FILE_DIR:mygame>

                COMMAND ${CMAKE_COMMAND} -E copy_if_different
                ${asset}
                ${CMAKE_INSTALL_PREFIX}/bin/${assetRelativePath})

        emscripten_embed_file(mygame ${asset} ${assetRelativePath})
endforeach(asset)
