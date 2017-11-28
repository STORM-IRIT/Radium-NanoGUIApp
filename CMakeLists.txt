set(app_target testGLFW)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

find_package(glfw REQUIRED)
find_package(nanogui REQUIRED)
find_package(Radium REQUIRED)


# use Radium's common compilation options
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${RADIUM_ROOT_DIR}/cmake)
include (CompileFlags)

if ( NOT CMAKE_PREFIX_PATH )
  set( CMAKE_PREFIX_PATH ${CMAKE_CURRENT_SOURCE_DIR} )
endif()

# Win32 stuff
if (MSVC OR MSVC_IDE)
    # Copy libs / targets in the correct directories
    if ("${CMAKE_GENERATOR}" STREQUAL "NMake Makefiles")
        set(PDB_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin)
    else()
        foreach(OUTPUTCONFIG ${CMAKE_CONFIGURATION_TYPES})
            string(TOUPPER ${OUTPUTCONFIG} OUTPUTCONFIG)
            set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_${OUTPUTCONFIG} ${RADIUM_PLUGIN_OUTPUT_PATH})
            set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_${OUTPUTCONFIG} ${RADIUM_PLUGIN_OUTPUT_PATH})
            set(CMAKE_LIBRARY_OUTPUT_DIRECTORY_${OUTPUTCONFIG} ${RADIUM_PLUGIN_OUTPUT_PATH})
        endforeach(OUTPUTCONFIG ${CMAKE_CONFIGURATION_TYPES})
    endif()
endif(MSVC OR MSVC_IDE)

# Access to Radium headers and declarations/defintions
include_directories(
    .
    ${RADIUM_INCLUDE_DIRS}
    ${GLFW_INCLUDE_DIR}
    ${NANOGUI_INCLUDE_DIR}
    )

# Get files
file( GLOB file_sources *.cpp *.c )
file( GLOB file_headers *.hpp *.h )

# Generate an executable
add_executable( ${app_target} ${file_sources} ${file_headers} )

add_dependencies( ${app_target} radiumEngine radiumCore radium_assets)
target_compile_options(${app_target} PRIVATE -fexceptions)


if(UNIX AND NOT APPLE)
    find_package(X11 REQUIRED)
    set (ADDITIONAL_LIBS dl X11 Xrandr Xinerama Xxf86vm Xcursor pthread)
else()
    set(ADDITIONAL_LIBS "")
endif()

# Link libraries
target_link_libraries( ${app_target} # target
    ${RADIUM_LIBRARIES} # Radium libs
    ${GLBINDING_LIBRARIES}           # Radium dep
    ${NANOGUI_LIBRARY}
    ${GLFW_LIBRARY}
    ${ADDITIONAL_LIBS}
)

if (MSVC)
    set_property( TARGET ${app_target} PROPERTY IMPORTED_LOCATION "${RADIUM_SUBMODULES_INSTALL_DIRECTORY}/bin" )
endif(MSVC)
