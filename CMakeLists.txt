cmake_minimum_required(VERSION 3.16)
project(LaTeX)

# Build options
option(QT "Compile using Qt instead of Win32/Gtk" OFF)
option(SKIA "Compile using Qt and Skia instead of Win32/Gtk" OFF)
option(BUILD_SAMPLES "Build sample applications" ON)
option(HAVE_LOG "Enable logging" ON)
option(GRAPHICS_DEBUG "Enable graphics debugging" ON)
option(MEM_CHECK "Compile for memory checking only" OFF)

add_library(LaTeX "")

# MSVC Compat

if (MSVC)
    set(CMAKE_CXX_STANDARD 17)
    set(CMAKE_CXX_STANDARD_REQUIRED ON)
    add_compile_options("/utf-8")
    target_compile_features(LaTeX PUBLIC cxx_std_17)
else ()

    # check if compiler has c++11/c++17 support

    include(CheckCXXCompilerFlag)
    CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
    CHECK_CXX_COMPILER_FLAG("-std=c++17" COMPILER_SUPPORTS_CXX17)
    CHECK_CXX_COMPILER_FLAG("-std=c++20" COMPILER_SUPPORTS_CXX20)

    # check gcc version
    set(CMAKE_CXX_STANDARD 20)
    set(CMAKE_CXX_STANDARD_REQUIRED ON)

    if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
        if ("${CMAKE_CXX_COMPILER_VERSION}" VERSION_LESS 9)
            # needs extra lib to use std::filesystem
            target_link_libraries(LaTeX PUBLIC "stdc++fs")
        endif ()
        if ("${CMAKE_CXX_COMPILER_VERSION}" VERSION_LESS 8)
            # dose not have full c++17 features
            set(COMPILER_SUPPORTS_CXX17 OFF)
        endif ()
    endif ()

    if (COMPILER_SUPPORTS_CXX20)
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++20")
    elseif (COMPILER_SUPPORTS_CXX17)
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17")
    elseif (COMPILER_SUPPORTS_CXX11)
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
    else ()
        message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no c++11 support. Please use a different one that supports c++11.")
    endif ()
endif ()
# copy res dir

file(COPY res DESTINATION .)
if (MSVC)
    find_package(tinyxml2 CONFIG REQUIRED)
    target_link_libraries(LaTeX PRIVATE tinyxml2::tinyxml2)
else ()
    find_package(PkgConfig REQUIRED)
    pkg_check_modules(tinyxml2 REQUIRED IMPORTED_TARGET tinyxml2)
    target_link_libraries(LaTeX PRIVATE tinyxml2)
endif ()

# source files
target_sources(LaTeX PRIVATE
        # atom folder
        src/atom/atom_basic.cpp
        src/atom/atom_char.cpp
        src/atom/atom_impl.cpp
        src/atom/atom_matrix.cpp
        src/atom/atom_row.cpp
        src/atom/atom_space.cpp
        src/atom/colors_def.cpp
        src/atom/unit_conversion.cpp
        # box folder
        src/box/box.cpp
        src/box/box_factory.cpp
        src/box/box_group.cpp
        src/box/box_single.cpp
        # core folder
        src/core/core.cpp
        src/core/formula.cpp
        src/core/formula_def.cpp
        src/core/glue.cpp
        src/core/localized_num.cpp
        src/core/macro.cpp
        src/core/macro_def.cpp
        src/core/macro_impl.cpp
        src/core/parser.cpp
        # fonts folder
        src/fonts/alphabet.cpp
        src/fonts/font_basic.cpp
        src/fonts/font_info.cpp
        src/fonts/fonts.cpp
        # utils folder
        src/utils/string_utils.cpp
        src/utils/utf.cpp
        src/utils/utils.cpp
        # res folder
        src/res/builtin/formula_mappings.res.cpp
        src/res/builtin/symbol_mapping.res.cpp
        src/res/builtin/tex_param.res.cpp
        src/res/builtin/tex_symbols.res.cpp
        src/res/font/bi10.def.cpp
        src/res/font/bx10.def.cpp
        src/res/font/cmbsy10.def.cpp
        src/res/font/cmbx10.def.cpp
        src/res/font/cmbxti10.def.cpp
        src/res/font/cmex10.def.cpp
        src/res/font/cmmi10.def.cpp
        src/res/font/cmmi10_unchanged.def.cpp
        src/res/font/cmmib10.def.cpp
        src/res/font/cmmib10_unchanged.def.cpp
        src/res/font/cmr10.def.cpp
        src/res/font/cmss10.def.cpp
        src/res/font/cmssbx10.def.cpp
        src/res/font/cmssi10.def.cpp
        src/res/font/cmsy10.def.cpp
        src/res/font/cmti10.def.cpp
        src/res/font/cmti10_unchanged.def.cpp
        src/res/font/cmtt10.def.cpp
        src/res/font/dsrom10.def.cpp
        src/res/font/eufb10.def.cpp
        src/res/font/eufm10.def.cpp
        src/res/font/i10.def.cpp
        src/res/font/moustache.def.cpp
        src/res/font/msam10.def.cpp
        src/res/font/msbm10.def.cpp
        src/res/font/r10.def.cpp
        src/res/font/r10_unchanged.def.cpp
        src/res/font/rsfs10.def.cpp
        src/res/font/sb10.def.cpp
        src/res/font/sbi10.def.cpp
        src/res/font/si10.def.cpp
        src/res/font/special.def.cpp
        src/res/font/ss10.def.cpp
        src/res/font/stmary10.def.cpp
        src/res/font/tt10.def.cpp
        src/res/parser/font_parser.cpp
        src/res/parser/formula_parser.cpp
        src/res/reg/builtin_font_reg.cpp
        src/res/reg/builtin_syms_reg.cpp
        src/res/sym/amsfonts.def.cpp
        src/res/sym/amssymb.def.cpp
        src/res/sym/base.def.cpp
        src/res/sym/stmaryrd.def.cpp
        src/res/sym/symspecial.def.cpp

        src/latex.cpp
        src/render.cpp
        )
target_include_directories(LaTeX PUBLIC src)

# check operating system

if (QT)
    message(STATUS, "Cross platform build using Qt")
    target_compile_definitions(LaTeX PUBLIC -DBUILD_QT)
    find_package(QT NAMES Qt6 Qt5 COMPONENTS Gui REQUIRED)
    find_package(Qt${QT_VERSION_MAJOR} COMPONENTS Gui REQUIRED)
    target_sources(LaTeX PRIVATE
            src/platform/qt/graphic_qt.cpp
            )
    target_link_libraries(LaTeX PRIVATE
            Qt${QT_VERSION_MAJOR}::Gui
            )
elseif (SKIA)
    message(STATUS, "Cross platform build using Qt and Skia for rendering")
    target_compile_definitions(LaTeX PUBLIC -DBUILD_SKIA -DSK_GL)
    target_include_directories(LaTeX PUBLIC src)
    if (MSVC)
        find_package(skia REQUIRED)
        target_link_libraries(LaTeX INTERFACE skia skia::skia)
    else ()
        include_directories(../skia ../skia/include)
        link_directories(../skia)
    endif ()
    find_package(QT NAMES Qt6 Qt5 COMPONENTS Core REQUIRED)
    find_package(Qt${QT_VERSION_MAJOR} COMPONENTS Core REQUIRED)
    target_sources(LaTeX PRIVATE
            src/platform/skia/graphic_skia.cpp
            )
    target_link_libraries(LaTeX PRIVATE
            Qt${QT_VERSION_MAJOR}::Core
            )
elseif (WIN32)
    message(STATUS "We are working on Windows")
    target_compile_definitions(LaTeX PUBLIC -DBUILD_WIN32 -D_HAS_STD_BYTE=0)
    target_sources(LaTeX PRIVATE
            src/platform/gdi_win/graphic_win32.cpp
            )
    target_link_libraries(LaTeX PRIVATE gdiplus)
elseif (UNIX)
    message(STATUS "Building the Cairo/Pango backend for GTK 4")
    target_compile_definitions(LaTeX PUBLIC -DBUILD_GTK)
    target_compile_features(LaTeX PUBLIC cxx_std_17)
    find_package(Fontconfig REQUIRED)
    pkg_check_modules(PangoMM REQUIRED IMPORTED_TARGET pangomm-2.48)
    pkg_check_modules(CairoMM REQUIRED IMPORTED_TARGET cairomm-1.16)
    target_sources(LaTeX PRIVATE
            src/platform/cairo/graphic_cairo.cpp
            )
    target_link_libraries(LaTeX
            PUBLIC PkgConfig::PangoMM PkgConfig::CairoMM
            PRIVATE Fontconfig::Fontconfig
            )
else ()
    message(STATUS "We are working on a unknown platform")
    # other platforms...
endif ()

# compile definitions

if (HAVE_LOG)
    add_definitions(-DHAVE_LOG)
endif ()

if (GRAPHICS_DEBUG)
    add_definitions(-DGRAPHICS_DEBUG)
endif ()

if (MEM_CHECK)
    add_definitions(-DMEM_CHECK)
endif ()

if (BUILD_SAMPLES)
    add_subdirectory(src/samples)
endif ()
