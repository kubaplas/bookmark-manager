 # project name
    project (com.github.bartzaalberg.bookmark-manager)

    # the oldest stable cmake version we support
    cmake_minimum_required (VERSION 2.6)

    # tell cmake where its modules can be found in our project directory
    list (APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake)
    list (APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake/src)

    # where we install data directory (if we have any)
    set (DATADIR "${CMAKE_INSTALL_PREFIX}/share")

    # what to call that directory where we install data too
    set (PKGDATADIR "${DATADIR}/com.github.bartzaalberg.bookmark-manager")

    set (EXEC_NAME "com.github.bartzaalberg.bookmark-manager")
    set (EXEC_NAME2 "com.github.bartzaalberg.bookmark-manager-quicklist")
    set (EXEC_NAME3 "com.github.bartzaalberg.bookmark-manager-quicklist-polkit")
    set (VERSION "1.4.2")
    set (VERSION_INFO "whats up world")

    set (DEPS_PACKAGES
        gtk+-3.0
        granite
        vte-2.91
    )

    find_package (PkgConfig)
    pkg_check_modules (DEPS REQUIRED ${DEPS_PACKAGES})


    add_definitions(${DEPS_CFLAGS})
    link_libraries(${DEPS_LIBRARIES})
    link_directories(${DEPS_LIBRARY_DIRS})

    # make sure we have vala
    find_package(Vala REQUIRED)
    # make sure we use vala
    include(ValaVersion)
    # make sure it's the desired version of vala
    ensure_vala_version("0.16" MINIMUM)

    #------------------------------------------------------------------------------------------

    # Translation files
    set (GETTEXT_PACKAGE "${CMAKE_PROJECT_NAME}")
    add_definitions (-DGETTEXT_PACKAGE=\"${GETTEXT_PACKAGE}\")
    add_subdirectory (po)

    #------------------------------------------------------------------------------------------

    # files we want to compile
    include(ValaPrecompile)
    vala_precompile(VALA_C ${EXEC_NAME2}
        src/DesktopFileManager/DesktopFileApplication.vala
        src/DesktopFileManager/DesktopFileManager.vala
        src/ConfigFileReader.vala
        src/Bookmark.vala

    # tell what libraries to use when compiling
    PACKAGES
        gtk+-3.0
    )

    # tell cmake what to call the executable we just made
    add_executable(${EXEC_NAME2} ${VALA_C})

    #------------------------------------------------------------------------------------------
    # files we want to compile
    include(ValaPrecompile)
    vala_precompile(VALA_C ${EXEC_NAME}
        src/Application.vala
        src/MainWindow.vala
        src/ListBoxRow.vala
        src/ListBox.vala
        src/ConfigFileReader.vala
        src/Constants.vala
        src/Bookmark.vala
        src/StackManager.vala
        src/Dialogs/Preferences.vala
        src/Dialogs/DeleteConfirm.vala
        src/Dialogs/Alert.vala
        src/Dialogs/Cheatsheet.vala
        src/Views/AddBookmark.vala
        src/Views/EditBookmark.vala
        src/Views/ListBookmarks.vala
        src/Views/EmptyView.vala
        src/Views/NotFoundView.vala
        src/Components/HeaderBar.vala
        src/Components/HeaderLabel.vala
        src/Components/BookmarkFormLabel.vala
        src/Components/BookmarkFormEntry.vala
        src/Components/BookmarkFormCheckButton.vala
        src/Components/BookmarkForm.vala
        src/Components/Terminal.vala

    # tell what libraries to use when compiling
    PACKAGES
        gtk+-3.0
        granite
        vte-2.91
    )

    include (GResource)
    glib_compile_resources (GLIB_RESOURCES_ICONS SOURCE /data/icons/icons.gresource.xml)

    # tell cmake what to call the executable we just made and install
    add_executable(${EXEC_NAME} ${VALA_C} ${GLIB_RESOURCES_ICONS})

    #------------------------------------------------------------------------------------------

    # files we want to compile
    include(ValaPrecompile)
    vala_precompile(VALA_C ${EXEC_NAME3}
        src/Polkit/Application.vala

    # tell what libraries to use when compiling
    PACKAGES
        gtk+-3.0
        gio-2.0
    )

    # tell cmake what to call the executable we just made
    add_executable(${EXEC_NAME3} ${VALA_C})
    target_compile_options (${EXEC_NAME3} PUBLIC -fPIE)

    #------------------------------------------------------------------------------------------

    # install the binaries we just made
    install (TARGETS ${EXEC_NAME} RUNTIME DESTINATION bin)
    install (TARGETS ${EXEC_NAME2} RUNTIME DESTINATION bin)
    install (TARGETS ${EXEC_NAME3} RUNTIME DESTINATION bin)
    #add the data directory and activated the cmakelists in that folder
    add_subdirectory (data)

    # add settings schema to pc and recomple all schemas
    include (GSettings)
    add_schema (data/com.github.bartzaalberg.bookmark-manager.gschema.xml)

    # install the polkit action file
    install (FILES ${CMAKE_CURRENT_SOURCE_DIR}/data/com.github.bartzaalberg.bookmark-manager.pkexec.policy DESTINATION /usr/share/polkit-1/actions/)
