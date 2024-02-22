cmake_minimum_required(VERSION 3.16)

# Ignored on other non OS X platforms
# set(CMAKE_OSX_ARCHITECTURES "x86_64;arm64" CACHE INTERNAL "" FORCE)
set(CMAKE_OSX_ARCHITECTURES "arm64" CACHE INTERNAL "" FORCE)

project(your-project-name-goes-here LANGUAGES C) # Change the name to your project here

set(CMAKE_C_STANDARD 99)

if(UNIX AND NOT APPLE) # Assume Linux
# Setup GLAD
# TODO: Switch this to VK
add_library(glad libs/glad/glad.c)
target_include_directories(glad PUBLIC libs)
elseif(APPLE)
# Use metal
# add_library(glad libs/glad/glad.c)
# target_include_directories(glad PUBLIC libs)
add_library(metalcpp libs/glad/glad.c)
target_include_directories(metalcpp PUBLIC libs)
else()
# Setup GLAD
add_library(glad libs/glad/glad.c)
target_include_directories(glad PUBLIC libs)
endif()
# Setup rayfork
add_subdirectory(libs/rayfork)

# Setup project
add_executable(${CMAKE_PROJECT_NAME} src/platform.c src/game.c)
if (NOT APPLE)
    target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE glad rayfork)
else()
    target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE rayfork)
endif()
target_include_directories(${CMAKE_PROJECT_NAME} PRIVATE libs/rayfork libs/sokol)
target_compile_definitions(${CMAKE_PROJECT_NAME} PUBLIC ASSETS_PATH="${CMAKE_CURRENT_SOURCE_DIR}/assets/") # This is useful to get an ASSETS_PATH in your IDE during development but you should comment this if you compile a release version and uncomment the next line
#target_compile_definitions(${CMAKE_PROJECT_NAME} PUBLIC ASSETS_PATH="my-path-to-the-assets-directory") # Uncomment this line to setup the ASSETS_PATH macro to the final assets directory when you share the game

# Enable MacOS specific flags and link against it's specific libraries
if (APPLE)
    target_link_libraries(${CMAKE_PROJECT_NAME} 
    PUBLIC 
        # $<IF:$<CONFIG:Debug>,SDL2d,SDL2>
        # $<IF:$<CONFIG:Debug>,SDL2maind,SDL2main>
        "-framework Cocoa"
        "-framework QuartzCore"
        "-framework Metal"
        "-framework MetalKit"
        # "-framework AudioToolbox"
        # "-framework AVFoundation"
        # "-framework Carbon"
        # "-framework Cocoa"
        # "-framework CoreAudio"
        # "-framework CoreGraphics"
        # "-framework CoreHaptics"
        # "-framework CoreMotion"
        # "-framework Foundation"
        # "-framework ForceFeedback"
        # "-framework GameController"
        # "-framework ImageIO"
        # "-framework IOKit"
    )
    target_compile_options(${CMAKE_PROJECT_NAME} PUBLIC "-fobjc-arc" "-ObjC")
       
        # "-framework QuartzCore"
        
endif()

# Enable Linux specific flags and link against it's specific libraries
# TODO: switch this to Wayland instead of X11
if (UNIX AND NOT APPLE)
    target_link_libraries(${CMAKE_PROJECT_NAME} PUBLIC m dl pthread X11 Xi Xcursor)
endif()
