cmake_minimum_required(VERSION 3.6)

project(fcitx-m17n VERSION 5.1.4)

set(REQUIRED_FCITX_VERSION 5.1.13)
find_package(ECM REQUIRED 1.0.0)
set(CMAKE_MODULE_PATH ${ECM_MODULE_PATH} "${CMAKE_CURRENT_SOURCE_DIR}/cmake" ${CMAKE_MODULE_PATH})
include(FeatureSummary)
include(GNUInstallDirs)
include(ECMUninstallTarget)

option(ENABLE_TEST "Build Test" On)
option(ENABLE_COVERAGE "Build the project with gcov support (Need ENABLE_TEST=On)" Off)
set(GCOV_TOOL "gcov" CACHE STRING "Path to gcov tool used by coverage.")

find_package(Fcitx5Core ${REQUIRED_FCITX_VERSION} REQUIRED)
find_package(Fcitx5Module REQUIRED COMPONENTS TestFrontend)
find_package(Gettext REQUIRED)
find_package(PkgConfig REQUIRED)

if (NOT DEFINED M17N_TARGET)
    pkg_check_modules(M17NGui IMPORTED_TARGET "m17n-gui>=1.6.3" REQUIRED)
    # Required for data and testing
    pkg_check_modules(M17NDB "m17n-db" REQUIRED)
    set(M17N_TARGET PkgConfig::M17NGui)
endif()

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")

include("${FCITX_INSTALL_CMAKECONFIG_DIR}/Fcitx5Utils/Fcitx5CompilerSettings.cmake")
add_definitions(-DFCITX_GETTEXT_DOMAIN=\"fcitx5-m17n\" -D_GNU_SOURCE)
fcitx5_add_i18n_definition()

add_subdirectory(po)
add_subdirectory(im)

if (ENABLE_TEST)
    enable_testing()
    add_subdirectory(test)

    if (ENABLE_COVERAGE)
        add_custom_target(coverage
                        COMMAND "${CMAKE_CTEST_COMMAND}"
                        COMMAND lcov --gcov-tool "${GCOV_TOOL}" --no-external --capture --directory ./ -b "${CMAKE_CURRENT_SOURCE_DIR}" --output-file coverage.info
                        COMMAND genhtml coverage.info --output-directory "coverage_pages"
                        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})
    endif()
endif ()

fcitx5_translate_desktop_file(org.fcitx.Fcitx5.Addon.M17N.metainfo.xml.in
    org.fcitx.Fcitx5.Addon.M17N.metainfo.xml XML)

install(FILES "${CMAKE_CURRENT_BINARY_DIR}/org.fcitx.Fcitx5.Addon.M17N.metainfo.xml" DESTINATION ${CMAKE_INSTALL_DATADIR}/metainfo)

feature_summary(WHAT ALL FATAL_ON_MISSING_REQUIRED_PACKAGES)
