cmake_minimum_required(VERSION 3.0.0)
project(qnanopainter VERSION 0.1.0)

set(CMAKE_VERBOSE_MAKEFILE ON CACHE BOOL "ON" FORCE)

option(QNANO_DEBUG "Enable this to get drawing debug information" OFF)
option(QNANO_QT_GL_INCLUDE "Enable this to let Qt include OpenGL headers" ON)
option(QNANO_ENABLE_GLES3 "This will enable GLES3 (disable to force GLES2)" ON)
option(QNANO_ENABLE_TOUCH_SIGNALS "This will enable signalling touch events
                                   Can be useful when using view/widget classes directly" OFF)
option(QNANO_ENABLE_PAINT_SIGNALS "This will enable signalling paint events
                                   Can be useful when using view/widget classes directly" OFF)
option(QNANO_USE_RENDERNODE "Enable this to use QRenderNode (available since Qt 5.8.0) instead of QQuickFramebufferObject" OFF)


option(QNANO_BUILD_GLES_BACKENDS "When building for embedded devices you can define manually which backends are supported" OFF)
option(QNANO_BUILD_GL_BACKENDS "When building for embedded devices you can define manually which backends are supported" OFF)

# Qt6 from msys2 need extra lib
option(QNANO_MSYS2 "Enable this to build in MSYS2" OFF)

set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)

find_package(QT NAMES Qt6 Qt5 REQUIRED COMPONENTS Core)
    message(STATUS "QNANOPAINTER QT VERSION=${QT_VERSION_MAJOR}") 
if(QT_VERSION_MAJOR EQUAL  6)
    find_package(Qt6 REQUIRED COMPONENTS Widgets OpenGL Quick OpenGLWidgets)
    set(QNANOPAINTER_QT_LIBS  Qt6::Widgets Qt6::OpenGL Qt6::Quick Qt6::OpenGLWidgets)
else()
    find_package(Qt5 REQUIRED COMPONENTS Widgets OpenGL Quick )
    set(QNANOPAINTER_QT_LIBS  Qt5::Widgets Qt5::OpenGL Qt5::Quick)
endif() 


########################################### 
if(QT_VERSION_MAJOR EQUAL  5)
    set(IS_QUICK_IN_QT ON)
    set(IS_WIDGETS_IN_QT ON)
    # ????? set(QNANO_BUILD_GLES_BACKENDS ON)
    set(QNANO_BUILD_GL_BACKENDS ON)
else()
    if(QT_FEATURE_quick_canvas) # QT_FEATURE_quick not exists ??
        set(IS_QUICK_IN_QT ON)
    endif()    

    if(QT_FEATURE_widgets)
        set(IS_WIDGETS_IN_QT ON)
    endif()    
    
    if(QT_FEATURE_opengles2 OR QT_FEATURE_opengles3)
        set(QNANO_BUILD_GLES_BACKENDS ON)
    else()   
        set(QNANO_BUILD_GL_BACKENDS ON)
    endif()    
endif()


set(SOURCES 
    qnanopainter.cpp 
    qnanocolor.cpp 
    qnanolineargradient.cpp 
    qnanoimagepattern.cpp 
    qnanoimage.cpp 
    qnanofont.cpp 
    qnanoradialgradient.cpp 
    qnanoboxgradient.cpp 
    qnanowindow.cpp 
    private/qnanodebug.cpp
)

set(HEADERS 
    private/qnanobrush.h 
    qnanopainter.h 
    qnanocolor.h 
    qnanolineargradient.h 
    qnanoimagepattern.h 
    qnanoimage.h 
    qnanofont.h 
    qnanoradialgradient.h 
    qnanoboxgradient.h 
    private/qnanodataelement.h 
    private/qnanobackend.h 
    private/qnanobackendfactory.h 
    qnanowindow.h 
    private/qnanodebug.h
)


if(IS_QUICK_IN_QT)
    set(SOURCES ${SOURCES}
        qnanoquickitem.cpp 
        qnanoquickitempainter.cpp        
    )
    set(HEADERS ${HEADERS}
        qnanoquickitem.h 
        qnanoquickitempainter.h
    )
endif()

if(IS_WIDGETS_IN_QT)
    set(SOURCES ${SOURCES}
        qnanowidget.cpp      
    )
    set(HEADERS ${HEADERS}
        qnanowidget.h
    )
endif()

if(QNANO_BUILD_GLES_BACKENDS)
    set(SOURCES ${SOURCES}
        private/qnanobackendgles2.h 
        private/qnanobackendgles3.h
    )
    set(HEADERS ${HEADERS}
        private/qnanobackendgles2.cpp 
        private/qnanobackendgles3.cpp
    )    
endif()

if(QNANO_BUILD_GL_BACKENDS)
    set(SOURCES ${SOURCES}
        private/qnanobackendgl2.h 
        private/qnanobackendgl3.h
    )
    set(HEADERS ${HEADERS}
        private/qnanobackendgl2.cpp 
        private/qnanobackendgl3.cpp
    )    
endif()



add_library(qnanopainter 
    ${SOURCES} 
    ${HEADERS}
    libqnanopainterdata.qrc
    nanovg/nanovg.c
    nanovg/nanovg.h
)


if (WIN32)
    if(QNANO_MSYS2)
        message(STATUS "BUILD MINGW add:GLESv2")
        set(QNANOPAINTER_QT_LIBS ${QNANOPAINTER_QT_LIBS} GLESv2)
    endif()
endif()

if(MSVC)
    # Suppress fontstash warnings about fopen & strncpy usage
    add_definitions(-D_CRT_SECURE_NO_WARNINGS)
endif()

# add QT Libs 
target_link_libraries(qnanopainter ${QNANOPAINTER_QT_LIBS} )

# add to INCLUDE PATH 
target_include_directories(qnanopainter PUBLIC .)  

# cmake variable ==> c++ #define
if(QNANO_QT_GL_INCLUDE)
    target_compile_definitions(qnanopainter PUBLIC QNANO_QT_GL_INCLUDE)
    message(STATUS "QNANO_QT_GL_INCLUDE=${QNANO_QT_GL_INCLUDE}")
endif() 
if(QNANO_DEBUG)
    target_compile_definitions(qnanopainter PUBLIC QNANO_DEBUG)
    message(STATUS "QNANO_DEBUG=${QNANO_DEBUG}")
endif()
if(QNANO_BUILD_GL_BACKENDS)
    target_compile_definitions(qnanopainter PUBLIC QNANO_BUILD_GL_BACKENDS)
    message(STATUS "QNANO_BUILD_GL_BACKENDS=${QNANO_BUILD_GL_BACKENDS}")
endif()    
if(QNANO_BUILD_GLES_BACKENDS)
    target_compile_definitions(qnanopainter PUBLIC QNANO_BUILD_GLES_BACKENDS)
    message(STATUS "QNANO_BUILD_GLES_BACKENDS=${QNANO_BUILD_GLES_BACKENDS}")
endif() 
if(QNANO_ENABLE_GLES3)    
    target_compile_definitions(qnanopainter PUBLIC QNANO_ENABLE_GLES3)
    message(STATUS "QNANO_ENABLE_GLES3=${QNANO_ENABLE_GLES3}")
endif()    
