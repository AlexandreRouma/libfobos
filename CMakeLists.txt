cmake_minimum_required(VERSION 3.13)
project(libfobos VERSION 0.2.0)

# Options
option(BUILD_FOBOS_UTILS "Build FobosSDR utilities such as fobos_devinfo" ON)
option(INSTALL_UDEV_RULES "Install UDEV rules (Linux only)" ON)

# List all source files
file(GLOB_RECURSE SRC "src/*.c")

# On windows, all symbols must be exported
if (MSVC)
    set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
endif ()

# Create dynamic libs
add_library(fobos SHARED ${SRC})

# Find libusb
find_package(PkgConfig REQUIRED)
pkg_check_modules(libusb REQUIRED IMPORTED_TARGET libusb-1.0)

# Set optimisation flags
if (${CMAKE_BUILD_TYPE} MATCHES "Debug")
    # Debug Flags
    if (MSVC)
        target_compile_options(fobos PRIVATE /EHsc)
    else ()
        target_compile_options(fobos PRIVATE -g -Og)
    endif ()
else()
    # Normal Flags
    if (MSVC)
        target_compile_options(fobos PRIVATE /O2 /Ob2 /EHsc)
    else ()
        target_compile_options(fobos PRIVATE -O3)
    endif ()
endif()

# Include the include folder
target_include_directories(fobos PUBLIC "include/")

# Link to libusb
target_link_libraries(fobos PRIVATE PkgConfig::libusb)

# TODO: Have it default instead of override
if (MSVC)
    set(CMAKE_INSTALL_PREFIX "C:/Program Files/RigExpert/Fobos/")
    set(CMAKE_INSTALL_BINDIR "bin")
    set(CMAKE_INSTALL_LIBDIR "lib")
    set(CMAKE_INSTALL_INCLUDEDIR "include")
else ()
    include(GNUInstallDirs)
endif ()

if (NOT MSVC)
    # Configure pkgconfig file
    configure_file(${CMAKE_SOURCE_DIR}/libfobos.pc.in ${CMAKE_CURRENT_BINARY_DIR}/libfobos.pc @ONLY)

    # Install pkgconfig file
    install(FILES ${CMAKE_CURRENT_BINARY_DIR}/libfobos.pc DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)
endif ()

# Install the library
if (MSVC)
    install(TARGETS fobos)
else ()
    install(TARGETS fobos DESTINATION ${CMAKE_INSTALL_LIBDIR})
endif ()

# Install the headers
install(DIRECTORY ${CMAKE_SOURCE_DIR}/include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

# On Windows, install dependencies
if (MSVC)
    install(FILES $<TARGET_FILE_DIR:fobos>/libusb-1.0.dll DESTINATION ${CMAKE_INSTALL_BINDIR})
endif ()

# Install UDEV rules
if (LINUX AND INSTALL_UDEV_RULES)
    install(FILES ${CMAKE_SOURCE_DIR}/fobos-sdr.rules DESTINATION ${CMAKE_INSTALL_LIBDIR}/udev/rules.d)
endif ()

# Build utils if enabled
if (BUILD_FOBOS_UTILS)
    add_subdirectory("utils/fobos_devinfo")
    add_subdirectory("utils/fobos_recorder")
endif ()

# Create uninstall target
configure_file(${CMAKE_SOURCE_DIR}/cmake/cmake_uninstall.cmake ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake @ONLY)
add_custom_target(uninstall ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)