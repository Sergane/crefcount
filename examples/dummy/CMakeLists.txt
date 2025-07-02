cmake_minimum_required(VERSION 3.10)
project(MultiFileProject)

set(BUILD_SHARED_LIBS OFF)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Create the static library from c.c
add_library(C STATIC c.c)

# Build b.c and link it with the C static library
add_library(B STATIC b.c)
target_link_libraries(B PRIVATE C)

# Build the executable using main.c, a.c, and link to B
add_executable(main main.c a.c)
target_link_libraries(main PRIVATE B -static)
