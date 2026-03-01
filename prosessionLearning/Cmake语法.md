# Cmake语法

详情参见：[CMake 教程 | 菜鸟教程](https://www.runoob.com/cmake/cmake-tutorial.html)

一个工业级C++项目的CMake结构如下：

```cmake
项目根目录/
├── CMakeLists.txt              # 根目录CMake
├── cmake/                      # 自定义CMake模块
│   ├── FindDependencies.cmake
│   ├── CompilerWarnings.cmake
│   ├── CodeCoverage.cmake
│   └── ...
├── src/                       # 主源码
│   ├── CMakeLists.txt
│   └── ...
├── include/                   # 公共头文件
│   └── 项目名/
│       └── ...
├── tests/                     # 单元测试
│   ├── CMakeLists.txt
│   └── ...
├── examples/                  # 示例代码
│   ├── CMakeLists.txt
│   └── ...
├── docs/                      # 文档
├── third_party/              # 第三方依赖
└── scripts/                  # 构建脚本
```



## Cmake核心内置指令

### 项目定义与基础配置部分

```cmake
#版本与项目。  
cmake_minimum_required(VERSION x.x)  #必须放在第一行，指定cmake最低版本
project(<PROJECT_NAME>[VERSION x.x] [LAGUAGES CXX]) #定义项目名称、版本和语言

#管理策略
cmake_policy(SET CMPXXXX NEW)         #控制特定CMake策略行为，保证兼容

#标准与全局变量
set(CMAKE_CXX_STANDARD 17)            #设置C++标准 （11,14,17,20）
set(CMAKE_CXX_STANDARD_REQUIRED ON)   #强制要求编译器支持指定标准。
set(CMAKE_CXX_EXTENSION OFF)          #禁用编译器特有扩展，保证可移植性

```



















## 根目录的CMakeList.txt

```cmake
cmake_minimum_required(VERSION 3.20) #支持现在CMake特性
project(ProjectName,
	VERSION 1.0.0
	DESCRIPTION "项目描述"
	LANGUAGE CXX c
)

cmake_policy(SET CMP0077 NEW)
cmake_policy(SET CMP0091 NEW)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSION OFF)
```

