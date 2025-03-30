# 1.    设置工程名称，linux下随便设置
project(main)

# 2.    设置cmake最低版本
cmake_minimum_required(VERSION 2.8.12)

# 3.    设置编译参数
# 3.1   指定C++版本为14
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)# 这是用来给neovim自动补全开启的


# 3.2   调试或者发布版本（Debug/Release）
set(CMAKE_BUILD_TYPE "Debug")

# 4.    设置源文件目录，保存为变量DIR_SRC
aux_source_directory(${CMAKE_SOURCE_DIR}/src SRC_FILES)

# 5.    设置头文件目录
include_directories(${CMAKE_SOURCE_DIR}/include)

# 6.    设置第三方库的头文件目录
# 6.1 root
list(APPEND CMAKE_PREFIX_PATH "~/sofeware/root/root-install/root/") # 自己设置root的安装路径
find_package(ROOT REQUIRED)
include_directories(${ROOT_INCLUDE_DIRS})

# 6.2 geant4
option(WITH_GEANT4_UIVIS "Build example with Geant4 UI and Vis drivers" ON)
if(WITH_GEANT4_UIVIS)
  find_package(Geant4 REQUIRED ui_all vis_all)
else()
  find_package(Geant4 REQUIRED)
endif()

# 只是为了测试
message(STATUS "ROOT_INCLUDE_DIRS = ${ROOT_INCLUDE_DIRS}") # 查看第三方库的头文件是否正确
message(STATUS "ROOT_LIBRARIES = ${ROOT_LIBRARIES}") # 查看第三方库的动态库是否正确
message(STATUS "SRC_FILES = ${SRC_FILES}") # 查看源文件是否正确
message(STATUS "DIR_INCLUDE = ${CMAKE_SOURCE_DIR}/include") # 查看头文件是否正确

# 7.    编译为可执行文件
add_executable(main main.cpp ${SRC_FILES}) # main为可执行文件名
target_link_libraries(main ${ROOT_LIBRARIES}  ${Geant4_LIBRARIES}) # 将动态库和可执行文件链接 
# main.cpp单独且放在src同级

# 8. 构建每个类为动态库
file(GLOB CLASS_SOURCES ${CMAKE_SOURCE_DIR}/src/*.cpp)
foreach(CLASS_SOURCE ${CLASS_SOURCES})
    # 获取类的名称
    get_filename_component(CLASS_NAME ${CLASS_SOURCE} NAME_WE)
    
    # 构建为动态库
    add_library(${CLASS_NAME} SHARED ${CLASS_SOURCE})
    
    # 设置动态库输出路径为bin/
    set_target_properties(${CLASS_NAME} PROPERTIES LIBRARY_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/bin)
    
    # 将动态库链接到主程序
    target_link_libraries(main ${CLASS_NAME})
endforeach()


# 自动复制 compile_commands.json
# 9. 添加自定义命令和目标用于复制 compile_commands.json
add_custom_command(
    OUTPUT ${PROJECT_SOURCE_DIR}/compile_commands.json
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
            ${PROJECT_BINARY_DIR}/compile_commands.json
            ${PROJECT_SOURCE_DIR}/compile_commands.json
    DEPENDS ${PROJECT_BINARY_DIR}/compile_commands.json
)

add_custom_target(copy_compile_commands ALL DEPENDS ${PROJECT_SOURCE_DIR}/compile_commands.json)
