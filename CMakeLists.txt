cmake_minimum_required(VERSION 3.16)
project(spotify-qt-quick LANGUAGES CXX VERSION 2.0)

# Qt
set(CMAKE_AUTOMOC ON)

# C++17
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Find Qt
find_package(Qt6 COMPONENTS Core Gui Quick QuickControls2 REQUIRED)

# Create main executable
qt_add_executable(${PROJECT_NAME} WIN32 MACOSX_BUNDLE MANUAL_FINALIZATION)

# Collect source files
add_subdirectory(src)

# Resources
include(QmlModule.cmake)

# Headers are in src/
target_include_directories(${PROJECT_NAME} PRIVATE
	"${CMAKE_CURRENT_SOURCE_DIR}/src"
	# QML requires headers in root
	"${CMAKE_CURRENT_SOURCE_DIR}/src/page"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/qml")

# Get version from Git
find_package(Git QUIET)
if (GIT_FOUND AND EXISTS "${PROJECT_SOURCE_DIR}/.git")
	execute_process(COMMAND ${GIT_EXECUTABLE} rev-parse --short HEAD
		WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
		OUTPUT_VARIABLE GIT_COMMIT
		OUTPUT_STRIP_TRAILING_WHITESPACE)
	target_compile_definitions(${PROJECT_NAME} PRIVATE GIT_COMMIT="${GIT_COMMIT}")
endif ()

# App related definitions
target_compile_definitions(${PROJECT_NAME} PRIVATE APP_VERSION="v${PROJECT_VERSION}")
target_compile_definitions(${PROJECT_NAME} PRIVATE APP_NAME="${PROJECT_NAME}")
target_compile_definitions(${PROJECT_NAME} PRIVATE APP_ICON="${PROJECT_NAME}")
target_compile_definitions(${PROJECT_NAME} PRIVATE ORG_NAME="kraxarn")
target_compile_definitions(${PROJECT_NAME} PRIVATE PKG_NAME="com.kraxarn")
target_compile_definitions(${PROJECT_NAME} PRIVATE BUILD_TYPE="${CMAKE_BUILD_TYPE}")

# Enable QML debugging in debug
if (CMAKE_BUILD_TYPE STREQUAL "Debug")
	target_compile_definitions(${PROJECT_NAME} PRIVATE QT_QML_DEBUG)
endif ()

# Install icons and desktop shortcut on unix-like
# Currently, it uses the same icon as spotify-qt
if (UNIX)
	install(FILES core/res/logo/spotify-qt.svg DESTINATION share/icons/hicolor/scalable/apps)
	install(FILES res/app/spotify-qt-quick.desktop DESTINATION share/applications)
endif ()

# Don't show console window under windows
if (WIN32)
	target_link_options(${PROJECT_NAME} PRIVATE -mwindows)
endif ()

# Android config
if (ANDROID)
	# https://bugreports.qt.io/browse/QTBUG-110326
	set_target_properties(${PROJECT_NAME} PROPERTIES QT_ANDROID_TARGET_SDK_VERSION 32)
	qt_android_generate_deployment_settings(${PROJECT_NAME})
endif ()

# Link Qt
target_link_libraries(${PROJECT_NAME} PRIVATE Qt6::Core Qt6::Gui Qt6::Quick Qt6::QuickControls2)

# spotify-qt-lib
set(QT_VERSION_MAJOR 6)
set(LIB_QT_IMPL ON)
add_subdirectory(core/lib)
target_link_libraries(${PROJECT_NAME} PRIVATE spotify-qt-lib)

qt_finalize_target(${PROJECT_NAME})