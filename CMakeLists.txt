cmake_minimum_required(VERSION 3.10)
SET(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake)
project(guiproc VERSION 0.0.1 LANGUAGES CXX C)

# Default to C99
if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 99)
endif()

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 17)
endif()

message(STATUS "System : ${CMAKE_SYSTEM_NAME}")
message(STATUS "Target system processor: ${CMAKE_SYSTEM_PROCESSOR}")
message(STATUS "Host architecture: ${CMAKE_HOST_SYSTEM_PROCESSOR}")
message(STATUS "Install prefix: ${CMAKE_INSTALL_PREFIX}")

# 执行git命令，并把结果重定向到自定义变量（此处是我的变量<GIT_VERSION>）。
execute_process(
	COMMAND	git log -1 --format=%H 		
	WORKING_DIRECTORY ${PROJECT_SOURCE_DIR} 
	OUTPUT_VARIABLE GIT_VERSION
)
execute_process(
	COMMAND git remote -v 			
	WORKING_DIRECTORY ${PROJECT_SOURCE_DIR} 
	OUTPUT_VARIABLE GIT_REMOTE_PATH
)

# 对得到的结果进行处理，尤其注意 \n,\t,\r之类的特殊字符，在cmake时没问题，但是生成的.cmake文件有问题，导致make出错。
string (REGEX REPLACE ".*\t| .*" "" GIT_PATH ${GIT_REMOTE_PATH})
string (REGEX REPLACE "[\n\t\r]" "" GIT_VERSION ${GIT_VERSION})

# 增加编译选项，把宏导入源码
add_definitions( -DGIT_VERSION=\"${GIT_VERSION}\")
add_definitions( -DGIT_PATH=\"${GIT_PATH}\")

# 获取工程路径
add_definitions(-DPROJECT_PATH="${CMAKE_INSTALL_PREFIX}")

set(COMMON_LIBRARIES
    pthread
    m
)

if (${CMAKE_SYSTEM_NAME} MATCHES "Linux")
if (${CMAKE_SYSTEM_PROCESSOR} STREQUAL ${CMAKE_HOST_SYSTEM_PROCESSOR})
  # 直接编译
  message(STATUS "host compile")
  add_definitions(-DUSE_FBDEV=0 -DUSE_EVDEV=0 -DUSE_SDL=1 -DUSE_MOUSE=1 -D_GNU_SOURCE)
  find_package(SDL2 REQUIRED)
  list(APPEND COMMON_LIBRARIES ${SDL2_LIBRARIES})
else()
  # 交叉编译fb应用
  message(STATUS "cross compile")
  add_definitions(-DUSE_FBDEV=1 -DUSE_EVDEV=1 -DUSE_SDL=0 -DUSE_MOUSE=0 -D_GNU_SOURCE)
endif()
  list(APPEND COMMON_LIBRARIES stdc++fs atomic)
elseif(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
set(CMAKE_CXX_STANDARD 17)
if (${MACHINES} MATCHES "arm64")
  message(STATUS "apple slicon compile")
else()
  message(STATUS "intel compile")
endif()
  add_definitions(-DLV_BUILD_TEST=0 -DSIMULATOR -D_GNU_SOURCE -DCONFIG_ELOOP_KQUEUE)
  find_package(SDL2 REQUIRED)
  list(APPEND COMMON_LIBRARIES 
    ${SDL2_LIBRARIES}
  )
  include_directories(${SDL2_INCLUDE_DIRS})
  set(CMAKE_EXE_LINKER_FLAGS "-framework CoreFoundation -framework Security")
else()
  # windows not support
  message(STATUS "System : ${CMAKE_SYSTEM_NAME} not support")
endif()

include_directories(
  .
  lvgl
)

file(GLOB main_srcs 
    src/*.c
    *.c
)

add_subdirectory(lvgl)
add_subdirectory(lv_drivers)

set(EXENAME ${PROJECT_NAME}-v${PROJECT_VERSION}-${GIT_VERSION})
add_executable(${EXENAME} ${main_srcs})
target_link_libraries(${EXENAME} PRIVATE
  lvgl
  lvgl::examples
  lvgl::demos
  lvgl::drivers
  ${COMMON_LIBRARIES}
)

add_custom_target(create_symlink ALL
    COMMAND ${CMAKE_COMMAND} -E create_symlink ${EXENAME} ${PROJECT_NAME}
    COMMENT "Creating alias symlink for my_program"
)

add_dependencies(create_symlink ${EXENAME})

install(TARGETS ${EXENAME}
    RUNTIME DESTINATION ${PROJECT_NAME})

# 安装软链接到目标目录
install(
    FILES ${CMAKE_BINARY_DIR}/${PROJECT_NAME}
    DESTINATION ${PROJECT_NAME}
)
