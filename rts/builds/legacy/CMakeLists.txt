# Place executables and shared libs under "build-dir/",
# instead of under "build-dir/rts/"
# This way, we have the build-dir structure more like the install-dir one,
# which makes testing spring in the builddir easier, eg. like this:
# cd build-dir
# SPRING_DATADIR=$(pwd) ./spring
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_LIBRARY_OUTPUT_DIRECTORY}")


### Assemble libraries
find_package(SDL2 REQUIRED)
set(engineIncludes ${SDL2_INCLUDE_DIR})
set(engineLibraries ${SDL2_LIBRARY})
if("${SDL2_VERSION_STRING}" VERSION_LESS "2")
	message(FATAL_ERROR "Found SDL v${SDL2_VERSION_STRING} while 2 is required!")
endif()

set(OpenGL_GL_PREFERENCE LEGACY)
find_package_static(OpenGL 3.0 REQUIRED)
find_package_static(GLEW 2.2.0 REQUIRED)
list(APPEND engineLibraries OpenGL::GL OpenGL::GLU GLEW::GLEW)

find_fontconfig_hack()
find_package_static(Fontconfig 2.11 REQUIRED)
list(APPEND engineLibraries Fontconfig::Fontconfig)
add_definitions(-DUSE_FONTCONFIG)

if (NOT WIN32)
	find_package_static(EXPAT 2.2.5 REQUIRED)
	list(APPEND engineLibraries EXPAT::EXPAT)
endif()

# !!! If you change freetype resolution here, change it also in rts/lib
find_freetype_hack() # hack to find different named freetype.dll
find_package_static(Freetype 2.8.1 REQUIRED)
list(APPEND engineLibraries Freetype::Freetype)

if    (UNIX)
	find_package(X11 REQUIRED)
	list(APPEND engineLibraries ${X11_Xcursor_LIB} ${X11_X11_LIB})
endif (UNIX)

if (APPLE)
	find_library(COREFOUNDATION_LIBRARY Foundation)
	list(APPEND engineLibraries ${COREFOUNDATION_LIBRARY})
endif (APPLE)

list(APPEND engineLibraries squish rgetc1)
list(APPEND engineLibraries ${sound-impl})
list(APPEND engineLibraries engineSystemNet)
list(APPEND engineLibraries ${engineCommonLibraries})
#list(APPEND engineLibraries engineaGui)
list(APPEND engineLibraries ${SPRING_SIM_LIBRARIES})
list(APPEND engineLibraries engineSim)
list(APPEND engineLibraries pr-downloader)
list(APPEND engineLibraries RmlUi::Core)
list(APPEND engineLibraries RmlUi::Debugger)
list(APPEND engineLibraries lunasvg)

### Assemble external incude dirs
list(APPEND engineIncludes ${OPENAL_INCLUDE_DIR})
list(APPEND engineIncludes ${ENGINE_SRC_ROOT_DIR}/lib/asio/include)
list(APPEND engineIncludes ${ENGINE_SRC_ROOT_DIR}/lib/slimsig/include)
list(APPEND engineIncludes ${ENGINE_SRC_ROOT_DIR}/lib/cereal/include)

include_directories(${engineIncludes})

### Build the executable
add_executable(engine-legacy ${EXE_FLAGS} ${engineSources} ${ENGINE_ICON} ${engineHeaders})
target_link_libraries(engine-legacy ${engineLibraries})


### Install the executable
install(TARGETS engine-legacy DESTINATION ${BINDIR})

create_engine_build_and_install_target(legacy)
