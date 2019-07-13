# cpp工程目录结构(私人用)

文档名称：C++工程目录规范

| 序号 | 更改原因 | 版本 | 作者 | 更改日期 | 备注 |
| ---- | -------- | ---- | ---- | -------- | ---- |
|   1   | 创建文档 | v0.1 | Hatton | 2019.7.10 | 无 |
|      |          |      |      |          |      |
|      |          |      |      |          |      |

# 前言

由于组内项目代码几乎没有规范的目录结构，几经迭代开发之后，整个项目代码看起来已经惨不忍睹。于是，笔者针对C++工程目录的规范进行了调研。不过，项目目录的规范本身就是一个仁者见仁智者见智的问题，就好比笔者和组长在针对头文件的存放问题上就存在了分歧。不过，这种规范的事情哪有对错，干脆组内项目就按照组长的意思来，自己做项目的话还是按照自己的想法舒服一点，于是乎这个 `repo` 就产生了。这个目录结构参考了大量的网络资料，经过笔者的理解总结而成，不过笔者也没有什么丰富的工程经验，只是按照现有的理解感觉这么分配比较合理。当然，有规范总比没有好，先按此规范执行，若发现不合适的地方再进行调整。

# 顶层目录结构

```shell
project_name
├── deploy
├── build
├── doc
├── 3rdparty
├── include
│   └── project_name
├── project_name
├── tools
├── scripts
├── platforms
├── test
├── LICENSE
├── CMakeLists.txt
├── build.sh
├── toolchain.cmake
├── .gitignore
├── readme.md
└── sample
```

- **deploy :** 用于存放部署、交付的文件，其包含子目录bin、lib、include分别存放本项目最总生成的可执行文件、库文件以及对外所提供的头文件。
- **build :** 用于存放build时cmake产生的中间文件，其包含子目录release和debug。
- **doc :** 用于存放项目的相关文档。
- **3rdparty :** 用于存放第三方库，每个第三库以单独目录的形式组织在3rdparty目录下。其中每个第三方目录下又有 `include` 和 `lib` 分别存放第三方库的头文件和库文件。
- **include/project_name :** 用于存放每个模块以及整个工程对外的头文件。具体格式如下文。
- **project_name :** 存放源码文件，以及内部头文件。具体格式如下文。
- **tools :** 包含一些支持项目构建的工具，如编译器等，一般情况下使用软链接。
- **scripts :** 包含一些脚本文件，如使用Jenkins进行自动化构建时所需要的脚本文件，以及一些用于预处理的脚本文件。
- **platforms :** 用于一些交叉编译时所需要的工具链等文件，按照平台进行划分来组织子目录。每个子目录下存放 `toolchain.cmake` 等用于指定平台的文件。
- **test :** 分模块存放测试代码。
- **LICENSE :** 版权信息说明。
- **CMakeLists.txt :** cmake文件。
- **build.sh :** build脚本文件。
- **.gitignore :** 指明git忽略规则。
- **readme.md :** 存放工程说明文件。
- **sample :** 存放示例代码。

# 源文件目录结构说明

结构示例：

```shell
# example modules tree
project_name
├── module_1
│   ├── dir_1
│   │   ├── something.cc
│   │   └── something.h
│   ├── dir_2
│   ├── module_1.cc
│   ├── CMakeLists.txt
│   └── README.md
├── module_2
│   ├── dir_1
│   ├── dir_2
│   ├── module_2.cc
│   ├── CMakeLists.txt
│   └── README.md
├── module_3
├── main.cc
└── CMakeLists.txt
```

1. 总源码文件目录以 `project_name` 命名，即与项目同名，存放在项目根目录下。
2. 源码文件分模块进行组织，分别以各个模块进行命名存放在 `project_name` 目录下，如示例中的 `module_1` 、`module_2`。
3. 在每个子模块目录下，只包含源文件以及该模块内部所调用的头文件。
4. 每个子模块的根目录下存放该模块的主要功能逻辑代码，如 `module_1.cc`。另外，可按照功能再划分子目录进行源码组织，但不可以出现模块嵌套的情况。
5. 若要包含内部头文件时，包含路径要从 `project_name` 开始路径要完整，如`#include "project_name/module1/dir_1/somthing.h"`，以防止头文件名称冲突的情况，同时遵循了[Google C++编码规范](https://zh-google-styleguide.readthedocs.io/en/latest/google-cpp-styleguide/contents/)。

# 头文件目录结构说明

```shell
# example include tree
include
└── project_name
    ├── module_1
    │   ├── module_1_header_1.h
    │   └── module_1_header_2.h
    ├── module_2
    │   └── module_2.h
    └── project_name.h
```

1. （公共）头文件目录以 `include/project_name` 命名，即文件目录为两级，存放在项目根目录下。该目录只包含所有对外的头文件。
2. 头文件同样分模块进行组织，分别以各个模块进行命名存放在 `include/project_name` 目录下，如示例中的 `module_1` 、`module_1`。
3. `include/project_name` 目录下最多只包含一级子目录，即最多按照模块再划分一级，模块内的功能头文件不再以功能进行划分。
4. 若要包含外部头文件时，包含路径同样要从 `project_name` 开始路径要完整，如`#include "project_name/module_2/module_2.h"`。

# 其他

1. 针对头文件的包含，顶层 `CMakeLists.txt` 只指定 `${CMAKE_SOURCE_DIR}\include` 和 `${CMAKE_SOURCE_DIR}`，以保证所有的包含规则都是从工程根目录开始包含。
2. 添加 `include` 目录使得公共头文件和对内部文件可以分离开，使多个模块之间合作开发时项目内部结构更加清晰。
5. （暂时）在 `3rdparty` 下存放的工程中用到的第三方库和第三方源码。第三方库尽量不要直接把静态连接库直接放到git仓库中，应该另外提供链接以便下载，或者提供文档说明库的名称和版本自行安装下载，或者提供git仓库自行编译。第三方源码一般为开源的，只提供git链接。

# TODO

1. 添加cmake示例。
2. 添加简单工程示例。

# 参考

大型项目CMakeLIsts.txt的编写规范
https://blog.csdn.net/dongfang1984/article/details/55105537

CmakeLists.txt书写规范
https://blog.csdn.net/csdnhuaong/article/details/80895679

cmake简介与编写规范
http://675816156.github.io/2016/06/26/cmake%E7%AE%80%E4%BB%8B%E4%B8%8E%E7%BC%96%E5%86%99%E8%A7%84%E8%8C%83/

C++工程目录架构推荐
https://www.cnblogs.com/kuliuheng/p/5729559.html

A Simple C++ Project Structure
https://hiltmon.com/blog/2013/07/03/a-simple-c-plus-plus-project-structure/

Directory Structure for a C++ Project
https://mariuszbartosik.com/directory-structure-for-a-c-project/

C++ application development ( Part 1 — Project structure )
https://medium.com/heuristics/c-application-development-part-1-project-structure-454b00f9eddc

AakashMallik/C++ project structure （与上个参考重）
https://gist.github.com/AakashMallik/f7451b09a0807ed7203fb97cd125bff5#file-c-project-structure

Filesystem Hierarchy Standard
http://www.pathname.com/fhs/pub/fhs-2.3.html

How to structure your project
https://cliutils.gitlab.io/modern-cmake/chapters/basics/structure.html

kigster/cmake-project-template
https://github.com/kigster/cmake-project-template

opencv/opencv
https://github.com/opencv/opencv
