# This is the main CMakeLists.txt file for your project.
# It defines how your project is built, where to find dependencies,
# and how to link them.

# Specify the minimum required CMake version.
cmake_minimum_required(VERSION 3.26)

# Set the name of your main project and the executable.
# It's good practice to keep them consistent.
set(MAIN_PROJECT_NAME "oop")
set(MAIN_EXECUTABLE_NAME "${MAIN_PROJECT_NAME}")

# Declare your project.
project(${MAIN_PROJECT_NAME} CXX)

# Set the C++ standard to C++23. This is crucial for ABI compatibility.
# If you built SFML from source, ensure it was compiled with the same C++ standard.
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Include custom CMake modules. Ensure these files exist in your 'cmake/' directory.
include(cmake/Options.cmake)
include(cmake/CompilerFlags.cmake)
include(cmake/CopyHelper.cmake)

###############################################################################
# SFML Configuration
# This section finds the SFML library on your system.
# We are requesting SFML version 2.5 or higher.
# The COMPONENTS specify which parts of SFML your project uses (graphics, window, system).
find_package(SFML 2.5 COMPONENTS graphics window system REQUIRED)

###############################################################################
# Executable Definition
# This command MUST come before any target_... commands that modify this executable.
# It defines your main executable and lists all its source files.
add_executable(${MAIN_EXECUTABLE_NAME}
    main.cpp
    enemy.cpp
    entity.cpp
    exception.cpp
    functions.cpp
    game.cpp
    visitor.cpp
    goblin.cpp
    item.cpp
    orc.cpp
    player.cpp
    player_builder.cpp
    potion.cpp
    skeleton.cpp
    user_interface.cpp
    wolf.cpp
    crow.cpp
    # Include any generated source files if applicable
    generated/src/Helper.cpp
)

###############################################################################
# Linking Libraries
# Link your executable against the SFML libraries.
# We first try to use SFML's modern imported targets (SFML::Graphics etc.).
# If those are not found (e.g., with older SFML installations), we fall back
# to linking directly against the library names (sfml-graphics etc.).
if(TARGET SFML::Graphics AND TARGET SFML::Window AND TARGET SFML::System)
    set(SFML_LIBRARIES SFML::Graphics SFML::Window SFML::System)
else()
    # Fallback for older SFML installations or if imported targets are not exposed.
    set(SFML_LIBRARIES sfml-graphics sfml-window sfml-system)
    # Ensure SFML include directories are added if not handled by find_package's imported targets
    # This might not be strictly necessary if SFML_INCLUDE_DIR is correctly set by find_package
    target_include_directories(${MAIN_EXECUTABLE_NAME} PRIVATE ${SFML_INCLUDE_DIR})
endif()

# Apply the SFML libraries to your executable.
target_link_libraries(${MAIN_EXECUTABLE_NAME} PRIVATE ${SFML_LIBRARIES})

###############################################################################
# Compiler Flags and Include Directories
# Apply compiler flags (e.g., sanitizers) to your executable.
# The 'set_compiler_flags' function is defined in 'cmake/CompilerFlags.cmake'.
set_compiler_flags(RUN_SANITIZERS TRUE TARGET_NAMES ${MAIN_EXECUTABLE_NAME})

# Specify additional include directories for your project.
# 'SYSTEM' tells tools like cppcheck and clang-tidy to ignore warnings from these directories.
target_include_directories(${MAIN_EXECUTABLE_NAME} SYSTEM PRIVATE generated/include)


###############################################################################
# External Dependencies (FetchContent - commented out by default)
# This section shows an example of how to fetch and include external dependencies
# directly from source using CMake's FetchContent module.
#
# include(FetchContent)
#
# set(FETCHCONTENT_QUIET OFF)
# set(FETCHCONTENT_UPDATES_DISCONNECTED ON)
#
# FetchContent_Declare(
#         SomeLib
#         GIT_REPOSITORY https://github.com/<SomeUser>/<SomeLib>.git
#         GIT_TAG        <some_git_hash> # <which tag/branch @ <some_date>>
#         GIT_SHALLOW    1  # works only with branches or tags, not with arbitrary commit hashes
# )
#
# FetchContent_MakeAvailable(SomeLib)

###############################################################################
# External Dependencies (find_package - commented out by default)
# This section shows an example of finding other system-installed libraries.
#
# find_package(Threads REQUIRED)


###############################################################################
# Installation and File Copying
# Define rules for installing your executable and copying necessary files.

# Install the main executable to a 'bin' folder (DESTINATION_DIR is set in Options.cmake).
install(TARGETS ${MAIN_EXECUTABLE_NAME} DESTINATION ${DESTINATION_DIR})

# For Apple platforms, install a launcher script if it exists.
if(APPLE)
    install(FILES launcher.command DESTINATION ${DESTINATION_DIR})
endif()

# Copy data files (like 'tastatura.txt') to the build/installation directory.
# The 'copy_files' function is defined in 'cmake/CopyHelper.cmake'.
copy_files(FILES tastatura.txt COPY_TO_DESTINATION TARGET_NAME ${MAIN_EXECUTABLE_NAME})
# Examples for copying multiple files or directories:
# copy_files(FILES tastatura.txt config.json DIRECTORY images sounds COPY_TO_DESTINATION TARGET_NAME ${MAIN_EXECUTABLE_NAME})
# copy_files(DIRECTORY images sounds COPY_TO_DESTINATION TARGET_NAME ${MAIN_EXECUTABLE_NAME})
