set(INCROOT ${PROJECT_SOURCE_DIR}/include/SFML/Graphics)
set(SRCROOT ${PROJECT_SOURCE_DIR}/src/SFML/Graphics)

# all source files
set(SRC
    ${SRCROOT}/BlendMode.cpp
    ${INCROOT}/BlendMode.hpp
    ${INCROOT}/Color.hpp
    ${INCROOT}/Color.inl
    ${INCROOT}/CoordinateType.hpp
    ${INCROOT}/Export.hpp
    ${SRCROOT}/Font.cpp
    ${INCROOT}/Font.hpp
    ${SRCROOT}/Glsl.cpp
    ${INCROOT}/Glsl.hpp
    ${INCROOT}/Glsl.inl
    ${INCROOT}/Glyph.hpp
    ${SRCROOT}/GLCheck.cpp
    ${SRCROOT}/GLCheck.hpp
    ${SRCROOT}/GLExtensions.hpp
    ${SRCROOT}/GLExtensions.cpp
    ${SRCROOT}/Image.cpp
    ${INCROOT}/Image.hpp
    ${INCROOT}/PrimitiveType.hpp
    ${INCROOT}/Rect.hpp
    ${INCROOT}/Rect.inl
    ${SRCROOT}/RenderStates.cpp
    ${INCROOT}/RenderStates.hpp
    ${SRCROOT}/RenderTexture.cpp
    ${INCROOT}/RenderTexture.hpp
    ${SRCROOT}/RenderTarget.cpp
    ${INCROOT}/RenderTarget.hpp
    ${SRCROOT}/RenderWindow.cpp
    ${INCROOT}/RenderWindow.hpp
    ${SRCROOT}/Shader.cpp
    ${INCROOT}/Shader.hpp
    ${SRCROOT}/StencilMode.cpp
    ${INCROOT}/StencilMode.hpp
    ${SRCROOT}/Texture.cpp
    ${INCROOT}/Texture.hpp
    ${SRCROOT}/TextureSaver.cpp
    ${SRCROOT}/TextureSaver.hpp
    ${SRCROOT}/Transform.cpp
    ${INCROOT}/Transform.hpp
    ${INCROOT}/Transform.inl
    ${SRCROOT}/Transformable.cpp
    ${INCROOT}/Transformable.hpp
    ${SRCROOT}/View.cpp
    ${INCROOT}/View.hpp
    ${INCROOT}/Vertex.hpp
)
source_group("" FILES ${SRC})

# drawables sources
set(DRAWABLES_SRC
    ${INCROOT}/Drawable.hpp
    ${SRCROOT}/Shape.cpp
    ${INCROOT}/Shape.hpp
    ${SRCROOT}/CircleShape.cpp
    ${INCROOT}/CircleShape.hpp
    ${SRCROOT}/RectangleShape.cpp
    ${INCROOT}/RectangleShape.hpp
    ${SRCROOT}/ConvexShape.cpp
    ${INCROOT}/ConvexShape.hpp
    ${SRCROOT}/Sprite.cpp
    ${INCROOT}/Sprite.hpp
    ${SRCROOT}/Text.cpp
    ${INCROOT}/Text.hpp
    ${SRCROOT}/VertexArray.cpp
    ${INCROOT}/VertexArray.hpp
    ${SRCROOT}/VertexBuffer.cpp
    ${INCROOT}/VertexBuffer.hpp
)
source_group("drawables" FILES ${DRAWABLES_SRC})

# render-texture sources
set(RENDER_TEXTURE_SRC
    ${SRCROOT}/RenderTextureImpl.hpp
    ${SRCROOT}/RenderTextureImplFBO.cpp
    ${SRCROOT}/RenderTextureImplFBO.hpp
    ${SRCROOT}/RenderTextureImplDefault.cpp
    ${SRCROOT}/RenderTextureImplDefault.hpp
)
source_group("render texture" FILES ${RENDER_TEXTURE_SRC})


# define the sfml-graphics target
sfml_add_library(Graphics
                 SOURCES ${SRC} ${DRAWABLES_SRC} ${RENDER_TEXTURE_SRC})

# setup dependencies
target_link_libraries(sfml-graphics PUBLIC SFML::Window)

# stb_image sources
target_include_directories(sfml-graphics SYSTEM PRIVATE "${PROJECT_SOURCE_DIR}/extlibs/headers/stb_image")

# glad sources
target_include_directories(sfml-graphics SYSTEM PRIVATE "${PROJECT_SOURCE_DIR}/extlibs/headers/glad/include")

# let CMake know about our additional graphics libraries paths
if(SFML_OS_WINDOWS)
    list(APPEND CMAKE_INCLUDE_PATH "${PROJECT_SOURCE_DIR}/extlibs/headers/freetype2")
elseif(SFML_OS_MACOS)
    list(APPEND CMAKE_INCLUDE_PATH "${PROJECT_SOURCE_DIR}/extlibs/headers/freetype2")
    list(APPEND CMAKE_LIBRARY_PATH "${PROJECT_SOURCE_DIR}/extlibs/libs-macos/Frameworks")
elseif(SFML_OS_IOS)
    list(APPEND CMAKE_INCLUDE_PATH "${PROJECT_SOURCE_DIR}/extlibs/headers/freetype2")
elseif(SFML_OS_ANDROID)
    list(APPEND CMAKE_INCLUDE_PATH "${PROJECT_SOURCE_DIR}/extlibs/headers/freetype2")
endif()

# find external libraries
if(SFML_OS_ANDROID)
    target_link_libraries(sfml-graphics PRIVATE z)
elseif(SFML_OS_IOS)
    target_link_libraries(sfml-graphics PRIVATE z bz2)
endif()

find_package(Freetype REQUIRED)
target_link_libraries(sfml-graphics PRIVATE Freetype::Freetype)

# on some platforms (e.g. Raspberry Pi 3 armhf), GCC requires linking libatomic to use <atomic> features
# that aren't supported by native CPU instructions (64-bit atomic operations on 32-bit architecture)
if(SFML_COMPILER_GCC)
    include(CheckCXXSourceCompiles)
    check_cxx_source_compiles("#include <atomic>
        int main(){std::atomic_ullong x(1); return x.fetch_add(1);}" ATOMIC_TEST)
    if(NOT ATOMIC_TEST)
        target_link_libraries(sfml-graphics PRIVATE atomic)
    endif()
endif()

# add preprocessor symbols
target_compile_definitions(sfml-graphics PRIVATE "STBI_FAILURE_USERMSG")

# Image.cpp must be compiled with the -fno-strict-aliasing
# when gcc is used; otherwise saving PNGs may crash in stb_image_write
if(SFML_COMPILER_GCC)
    set_source_files_properties(${SRCROOT}/Image.cpp PROPERTIES COMPILE_FLAGS -fno-strict-aliasing)
endif()
