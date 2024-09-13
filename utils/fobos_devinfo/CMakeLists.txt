cmake_minimum_required(VERSION 3.13)
project(fobos_devinfo)

# Get source files
file(GLOB_RECURSE SRC "src/*.c")

# Create executable
add_executable(${PROJECT_NAME} ${SRC})

# Link to librfnm
target_link_libraries(${PROJECT_NAME} PRIVATE fobos)

# Create install directive
install(TARGETS ${PROJECT_NAME} DESTINATION ${CMAKE_INSTALL_BINDIR})