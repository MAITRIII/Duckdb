cmake_minimum_required(VERSION 3.12)

# Set project name
project(RMI-DuckDB)

# Set C++ Standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Suppress deprecated warnings for modern compilers
if(CMAKE_CXX_COMPILER_ID MATCHES "Clang|GNU")
    add_compile_options(-Wno-deprecated-declarations)
endif()

# Include directories
include_directories(src/include)
include_directories(src/PGM-index/include) # Include for ALEX extension

# Find OpenSSL
find_package(OpenSSL REQUIRED)

# Find OpenMP (optional)
find_package(OpenMP)
if(OpenMP_FOUND)
    message(STATUS "OpenMP found")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
    add_definitions(-DUSE_OPENMP)
else()
    message(WARNING "OpenMP not found. Continuing without OpenMP support.")
endif()

# ALEX Extension
set(ALEX_TARGET_NAME alex)
set(ALEX_EXTENSION_SOURCES src/alex_extension.cpp)

# Add static and loadable libraries for ALEX
build_static_extension(${ALEX_TARGET_NAME} ${ALEX_EXTENSION_SOURCES})
build_loadable_extension(${ALEX_TARGET_NAME} "" ${ALEX_EXTENSION_SOURCES})

# Link OpenSSL for ALEX
target_link_libraries(${ALEX_TARGET_NAME}_extension OpenSSL::SSL OpenSSL::Crypto)
target_link_libraries(${ALEX_TARGET_NAME}_loadable_extension OpenSSL::SSL OpenSSL::Crypto)

# Install extensions
install(
  TARGETS ${ALEX_TARGET_NAME}_extension
  EXPORT "${DUCKDB_EXPORT_SET}"
  LIBRARY DESTINATION "${INSTALL_LIB_DIR}"
  ARCHIVE DESTINATION "${INSTALL_LIB_DIR}"
)

# Suppress deprecation warnings for DuckDB compatibility
add_compile_definitions(_LIBCPP_ENABLE_CXX17_REMOVED_FEATURES)

# Build configuration summary
message(STATUS "Build configuration complete. Extensions will be placed in ${CMAKE_BINARY_DIR}/extensions.")
