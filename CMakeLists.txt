cmake_minimum_required(VERSION 3.16)
project(PaintEditor LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_AUTOMOC ON) # Автоматически запускает MOC для Q_OBJECT
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

find_package(Qt6 REQUIRED COMPONENTS Widgets PrintSupport)

set(SOURCES
    main.cpp
    MainWindow.cpp
    Canvas.cpp
    RibbonButton.cpp
    FileMenu.cpp
    Dialogs.cpp
    Icons.cpp
    BrushTool.cpp
    PencilTool.cpp
    FillTool.cpp
    LineTool.cpp
    ShapeTool.cpp
    EraserTool.cpp
)

if(WIN32)
    # WIN32 — приложение без консольного окна; app.rc подключает иконку .exe
    add_executable(PaintEditor WIN32 ${SOURCES} app.rc)
    # M_PI в <cmath>, без макросов min/max из windows.h
    target_compile_definitions(PaintEditor PRIVATE _USE_MATH_DEFINES NOMINMAX)
    if(MSVC)
        # русские строки в исходниках в UTF-8
        target_compile_options(PaintEditor PRIVATE /utf-8)
    endif()
elseif(APPLE)
    set(APP_ICON ${CMAKE_CURRENT_SOURCE_DIR}/icon.icns)
    set_source_files_properties(${APP_ICON} PROPERTIES MACOSX_PACKAGE_LOCATION "Resources")
    add_executable(PaintEditor MACOSX_BUNDLE ${SOURCES} ${APP_ICON})
    set_target_properties(PaintEditor PROPERTIES
        MACOSX_BUNDLE_ICON_FILE icon.icns
        MACOSX_BUNDLE_BUNDLE_NAME "Растровый редактор"
        MACOSX_BUNDLE_GUI_IDENTIFIER com.example.painteditor
    )
else()
    add_executable(PaintEditor ${SOURCES})
endif()

# стандартные заголовки, которые MSVC сам не подтягивает через Qt
target_precompile_headers(PaintEditor PRIVATE
    <algorithm> <cmath> <functional> <memory> <vector>)

target_link_libraries(PaintEditor PRIVATE Qt6::Widgets Qt6::PrintSupport)

# Windows: после сборки копируем Qt DLL и плагины рядом с .exe
if(WIN32)
    find_program(WINDEPLOYQT windeployqt HINTS "${Qt6_DIR}/../../../bin")
    if(WINDEPLOYQT)
        add_custom_command(TARGET PaintEditor POST_BUILD
            COMMAND "${WINDEPLOYQT}" --no-translations --no-system-d3d-compiler
                    "$<TARGET_FILE:PaintEditor>"
            COMMENT "windeployqt: копирование библиотек Qt")
    endif()
endif()
