find_package(
  Qt5
  CONFIG
  REQUIRED
  Core
  Gui
  Widgets
  Network
  Svg
  DBus
  LinguistTools)

set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)

add_executable(flameshot)

add_executable(Flameshot::flameshot ALIAS flameshot)

if(WIN32)
  set_property(TARGET flameshot PROPERTY WIN32_EXECUTABLE true)
endif()

add_subdirectory(cli)
add_subdirectory(config)
add_subdirectory(core)
add_subdirectory(utils)
add_subdirectory(widgets)
add_subdirectory(tools)

set(FLAMESHOT_TS_FILES
    ${CMAKE_SOURCE_DIR}/data/translations/Internationalization_ca.ts
    ${CMAKE_SOURCE_DIR}/data/translations/Internationalization_cs.ts
    ${CMAKE_SOURCE_DIR}/data/translations/Internationalization_de_DE.ts
    ${CMAKE_SOURCE_DIR}/data/translations/Internationalization_es.ts
    ${CMAKE_SOURCE_DIR}/data/translations/Internationalization_eu.ts
    ${CMAKE_SOURCE_DIR}/data/translations/Internationalization_fr.ts
    ${CMAKE_SOURCE_DIR}/data/translations/Internationalization_hu.ts
    ${CMAKE_SOURCE_DIR}/data/translations/Internationalization_it_IT.ts
    ${CMAKE_SOURCE_DIR}/data/translations/Internationalization_ja.ts
    ${CMAKE_SOURCE_DIR}/data/translations/Internationalization_ka.ts
    ${CMAKE_SOURCE_DIR}/data/translations/Internationalization_ko.ts
    ${CMAKE_SOURCE_DIR}/data/translations/Internationalization_nl.ts
    ${CMAKE_SOURCE_DIR}/data/translations/Internationalization_nl_NL.ts
    ${CMAKE_SOURCE_DIR}/data/translations/Internationalization_pl.ts
    ${CMAKE_SOURCE_DIR}/data/translations/Internationalization_pt_BR.ts
    ${CMAKE_SOURCE_DIR}/data/translations/Internationalization_ru.ts
    ${CMAKE_SOURCE_DIR}/data/translations/Internationalization_sk.ts
    ${CMAKE_SOURCE_DIR}/data/translations/Internationalization_sr_SP.ts
    ${CMAKE_SOURCE_DIR}/data/translations/Internationalization_sv_SE.ts
    ${CMAKE_SOURCE_DIR}/data/translations/Internationalization_tr.ts
    ${CMAKE_SOURCE_DIR}/data/translations/Internationalization_uk.ts
    ${CMAKE_SOURCE_DIR}/data/translations/Internationalization_zh_CN.ts
    ${CMAKE_SOURCE_DIR}/data/translations/Internationalization_zh_HK.ts
    ${CMAKE_SOURCE_DIR}/data/translations/Internationalization_zh_TW.ts)

if(GENERATE_TS)
  qt5_create_translation(QM_FILES ${CMAKE_SOURCE_DIR} ${FLAMESHOT_TS_FILES})
else()
  qt5_add_translation(QM_FILES ${FLAMESHOT_TS_FILES})
endif()

target_sources(
  flameshot
  PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/../external/singleapplication/singleapplication.cpp
          ${CMAKE_CURRENT_SOURCE_DIR}/../external/Qt-Color-Widgets/src/color_utils.cpp
          ${CMAKE_CURRENT_SOURCE_DIR}/../external/Qt-Color-Widgets/src/color_wheel.cpp
          ${CMAKE_CURRENT_SOURCE_DIR}/../external/Qt-Color-Widgets/include/color_wheel.hpp
          ${CMAKE_CURRENT_SOURCE_DIR}/../data/graphics.qrc
          ${CMAKE_CURRENT_SOURCE_DIR}/../data/flameshot.rc # windows binary icon resource file
          ${QM_FILES}
          main.cpp)

target_include_directories(
  flameshot
  PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../>
         $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/>
         $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../external/singleapplication/>
         $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../external/Qt-Color-Widgets/include>
         $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../dbus/>
         $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/cli>
         $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/config>
         $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/core>
         $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/tools>
         $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/tools/arrow>
         $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/tools/pixelate>
         $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/tools/circle>
         $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/tools/circlecount>
         $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/tools/copy>
         $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/tools/exit>
         $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/tools/imgur>
         $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/tools/launcher>
         $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/tools/line>
         $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/tools/marker>
         $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/tools/move>
         $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/tools/pencil>
         $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/tools/pin>
         $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/tools/rectangle>
         $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/tools/redo>
         $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/tools/save>
         $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/tools/selection>
         $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/tools/sizeindicator>
         $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/tools/text>
         $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/tools/undo>
         $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/utils>
         $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/widgets>
         $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/widgets/panel>
         $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/widgets/capture>
         $<INSTALL_INTERFACE:include/mylib>)

target_link_libraries(
  flameshot
  project_warnings
  project_options
  Qt5::Svg
  Qt5::DBus
  Qt5::Network
  Qt5::Widgets)

set(USE_OPENSSL FALSE)
if(ENABLE_OPENSSL)
  find_package(OpenSSL)
  if (OPENSSL_FOUND)
    message(STATUS "OpenSSL support enabled.")
    set(USE_OPENSSL TRUE)
  endif()
else()
  mark_as_advanced(CLEAR OPENSSL_LIBRARIES OPENSSL_INCLUDE_DIR)
endif()

if(NOT USE_OPENSSL)
  message(WARNING "OpenSSL is required to upload screenshot to imgur")
endif()

target_compile_definitions(flameshot PRIVATE APP_PREFIX="${CMAKE_INSTALL_PREFIX}")
target_compile_definitions(flameshot PRIVATE APP_VERSION="v${PROJECT_VERSION}")
target_compile_definitions(flameshot PRIVATE IMGUR_CLIENT_ID="313baf0c7b4d3ff")
target_compile_definitions(flameshot PRIVATE QAPPLICATION_CLASS=QApplication)

foreach(FILE ${QM_FILES})
  get_filename_component(F_NAME ${FILE} NAME)
  add_custom_command(
    TARGET flameshot
    POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_BINARY_DIR}/${F_NAME}
            ${CMAKE_CURRENT_BINARY_DIR}/translations/${F_NAME})
endforeach()

# ######################################################################################################################
# Installation instructions

include(GNUInstallDirs)
set(INSTALL_CONFIGDIR ${CMAKE_INSTALL_LIBDIR}/cmake/Flameshot)

# Install binary
install(
  TARGETS flameshot
  EXPORT flameshot-targets
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

# Install desktop files, completion and dbus files
configure_file(${CMAKE_SOURCE_DIR}/data/desktopEntry/package/org.flameshot.Flameshot.desktop
               ${CMAKE_CURRENT_BINARY_DIR}/share/applications/org.flameshot.Flameshot.desktop COPYONLY)

configure_file(${CMAKE_SOURCE_DIR}/data/appdata/org.flameshot.Flameshot.metainfo.xml
               ${CMAKE_CURRENT_BINARY_DIR}/share/metainfo/org.flameshot.Flameshot.metainfo.xml COPYONLY)

configure_file(${CMAKE_SOURCE_DIR}/data/shell-completion/flameshot
               ${CMAKE_CURRENT_BINARY_DIR}/share/bash-completion/completions/flameshot COPYONLY)

configure_file(${CMAKE_SOURCE_DIR}/data/shell-completion/flameshot
               ${CMAKE_CURRENT_BINARY_DIR}/share/zsh/site-functions/_flameshot COPYONLY)

configure_file(${CMAKE_SOURCE_DIR}/data/dbus/org.flameshot.Flameshot.xml
               ${CMAKE_CURRENT_BINARY_DIR}/share/dbus-1/interfaces/org.flameshot.Flameshot.xml COPYONLY)

configure_file(${CMAKE_SOURCE_DIR}/data/dbus/org.flameshot.Flameshot.service.in
               ${CMAKE_CURRENT_BINARY_DIR}/share/dbus-1/services/org.flameshot.Flameshot.service)

# Install Icons
configure_file(${CMAKE_SOURCE_DIR}/data/img/hicolor/48x48/apps/org.flameshot.Flameshot.png
               ${CMAKE_CURRENT_BINARY_DIR}/share/icons/hicolor/48x48/apps/org.flameshot.Flameshot.png COPYONLY)

configure_file(${CMAKE_SOURCE_DIR}/data/img/hicolor/128x128/apps/org.flameshot.Flameshot.png
               ${CMAKE_CURRENT_BINARY_DIR}/share/icons/hicolor/128x128/apps/org.flameshot.Flameshot.png COPYONLY)

configure_file(${CMAKE_SOURCE_DIR}/data/img/hicolor/scalable/apps/org.flameshot.Flameshot.svg
               ${CMAKE_CURRENT_BINARY_DIR}/share/icons/hicolor/scalable/apps/org.flameshot.Flameshot.svg COPYONLY)

## Install icon with both names
configure_file(${CMAKE_SOURCE_DIR}/data/img/hicolor/48x48/apps/flameshot.png
               ${CMAKE_CURRENT_BINARY_DIR}/share/icons/hicolor/48x48/apps/flameshot.png COPYONLY)

configure_file(${CMAKE_SOURCE_DIR}/data/img/hicolor/128x128/apps/flameshot.png
               ${CMAKE_CURRENT_BINARY_DIR}/share/icons/hicolor/128x128/apps/flameshot.png COPYONLY)

configure_file(${CMAKE_SOURCE_DIR}/data/img/hicolor/scalable/apps/flameshot.svg
               ${CMAKE_CURRENT_BINARY_DIR}/share/icons/hicolor/scalable/apps/flameshot.svg COPYONLY)


# Install assets
install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/share/ DESTINATION ${CMAKE_INSTALL_DATAROOTDIR})

# Install Translations
install(FILES ${QM_FILES} DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/flameshot/translations)


# windeployqt
if(WIN32)
  if(EXISTS $ENV{QTDIR}/bin/windeployqt.exe)
    if(CMAKE_BUILD_TYPE MATCHES Release)
      set(BINARIES_TYPE --release)
    else()
      set(BINARIES_TYPE --debug)
    endif()

    add_custom_command(
      TARGET flameshot POST_BUILD
      COMMAND ${CMAKE_COMMAND} -E remove_directory ${CMAKE_BINARY_DIR}/windeployqt_stuff
      COMMAND $ENV{QTDIR}/bin/windeployqt.exe ${BINARIES_TYPE} --no-translations  --compiler-runtime --no-system-d3d-compiler --no-angle --no-webkit2 --no-quick-import --dir ${CMAKE_BINARY_DIR}/windeployqt_stuff $<TARGET_FILE:flameshot>
      # copy translations manually
      # QM_FILES
      COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_BINARY_DIR}/windeployqt_stuff/translations
      COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_BINARY_DIR}/src/translations ${CMAKE_BINARY_DIR}/windeployqt_stuff/translations
    )

    install(
      DIRECTORY ${CMAKE_BINARY_DIR}/windeployqt_stuff/
      DESTINATION bin
    )

    if (ENABLE_OPENSSL)
      if (EXISTS $ENV{OPENSSL_ROOT_DIR}/bin)
        install(DIRECTORY $ENV{OPENSSL_ROOT_DIR}/bin/
                DESTINATION bin
                FILES_MATCHING PATTERN "*.dll"
                )
      else()
        message(WARNING "Unable to find OpenSSL dlls.")
      endif()
    endif()

  else()
    message("Unable to find executable QTDIR/bin/windeployqt.")
  endif()
endif()
