cmake_minimum_required(VERSION 3.12.0)

project(SingleApplication LANGUAGES CXX DESCRIPTION "Replacement for QtSingleApplication")

set(CMAKE_AUTOMOC ON)

add_library(${PROJECT_NAME} STATIC
    singleapplication.cpp
    singleapplication_p.cpp
)
add_library(${PROJECT_NAME}::${PROJECT_NAME} ALIAS ${PROJECT_NAME})

#FIXED: Qt 默认版本为 6, SingleApplication 继承自 QApplication
if(NOT QT_DEFAULT_MAJOR_VERSION)
    set(QT_DEFAULT_MAJOR_VERSION 6 CACHE STRING "Qt version to use (5 or 6), defaults to 6")
endif()
set(QAPPLICATION_CLASS "QApplication")
#* ↑↑↑↑↑↑↑

# Find dependencies
set(QT_COMPONENTS Core Network)
set(QT_LIBRARIES Qt${QT_DEFAULT_MAJOR_VERSION}::Core Qt${QT_DEFAULT_MAJOR_VERSION}::Network)

if(QAPPLICATION_CLASS STREQUAL QApplication)
    list(APPEND QT_COMPONENTS Widgets)
    list(APPEND QT_LIBRARIES Qt${QT_DEFAULT_MAJOR_VERSION}::Widgets)
elseif(QAPPLICATION_CLASS STREQUAL QGuiApplication)
    list(APPEND QT_COMPONENTS Gui)
    list(APPEND QT_LIBRARIES Qt${QT_DEFAULT_MAJOR_VERSION}::Gui)
else()
    set(QAPPLICATION_CLASS QCoreApplication)
endif()

find_package(Qt${QT_DEFAULT_MAJOR_VERSION} COMPONENTS ${QT_COMPONENTS} REQUIRED)

option(SINGLEAPPLICATION_DOCUMENTATION "Generate Doxygen documentation" OFF)
if(SINGLEAPPLICATION_DOCUMENTATION)
    find_package(Doxygen)
endif()

target_link_libraries(${PROJECT_NAME} PUBLIC ${QT_LIBRARIES})

if(WIN32)
    target_link_libraries(${PROJECT_NAME} PRIVATE advapi32)
endif()

target_compile_definitions(${PROJECT_NAME} PUBLIC QAPPLICATION_CLASS=${QAPPLICATION_CLASS})
target_include_directories(${PROJECT_NAME} PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
target_compile_definitions(${PROJECT_NAME} PRIVATE
    QT_NO_CAST_TO_ASCII
    QT_NO_CAST_FROM_ASCII
    QT_NO_URL_CAST_FROM_STRING
    QT_NO_CAST_FROM_BYTEARRAY
    QT_USE_QSTRINGBUILDER
    QT_NO_NARROWING_CONVERSIONS_IN_CONNECT
    QT_NO_KEYWORDS
    QT_NO_FOREACH
)
