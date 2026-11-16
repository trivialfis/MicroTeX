# Keep sample executables beside the copied runtime resources.
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")

if (QT)
    find_package(Qt${QT_VERSION_MAJOR} COMPONENTS Widgets PrintSupport REQUIRED)
    add_executable(LaTeXQtSample
            qt_texwidget.cpp
            qt_mainwindow.cpp
            qt_main.cpp
            )
    target_link_libraries(LaTeXQtSample PRIVATE
            Qt${QT_VERSION_MAJOR}::Widgets
            Qt${QT_VERSION_MAJOR}::PrintSupport
            LaTeX)
    set_target_properties(LaTeXQtSample PROPERTIES
            OUTPUT_NAME LaTeX
            AUTOMOC ON)

    add_executable(latex2png latex2png.cpp)
    target_link_libraries(latex2png PRIVATE
            LaTeX
            Qt${QT_VERSION_MAJOR}::Core
            Qt${QT_VERSION_MAJOR}::Gui)
elseif (SKIA)
    find_package(Qt${QT_VERSION_MAJOR} COMPONENTS Widgets REQUIRED)
    add_executable(LaTeXQtSkiaSample
            qt_skiatexwidget.cpp
            qt_mainwindow.cpp
            qt_main.cpp
            )
    target_link_libraries(LaTeXQtSkiaSample PRIVATE
            Qt${QT_VERSION_MAJOR}::Widgets LaTeX)
    set_target_properties(LaTeXQtSkiaSample PROPERTIES
            OUTPUT_NAME LaTeX
            AUTOMOC ON)
elseif (WIN32)
    add_executable(LaTeXWin32Sample WIN32 win32_main.cpp)
    target_link_libraries(LaTeXWin32Sample PRIVATE LaTeX)
    set_target_properties(LaTeXWin32Sample PROPERTIES OUTPUT_NAME LaTeX)
elseif (UNIX)
    pkg_check_modules(GTKMM REQUIRED IMPORTED_TARGET gtkmm-4.0>=4.10)
    add_executable(LaTeXGtkSample gtkmm_main.cpp)
    target_link_libraries(LaTeXGtkSample PRIVATE PkgConfig::GTKMM LaTeX)
    set_target_properties(LaTeXGtkSample PROPERTIES OUTPUT_NAME LaTeX)
endif ()

