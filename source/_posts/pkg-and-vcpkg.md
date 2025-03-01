---
title: 包管理与vcpkg入门
date: 2025-02-19 21:24:32
tags: [C++]

description: 讲解代码协作、包管理与vcpkg入门教程
---


> 我们将会在Windows上进行实际操作。在GNU/Linux上，本文所述的很多操作往往更加容易实现，因此不再赘述，如有问题请善用搜索引擎和AI。
> 
> 本文中的许多操作都需要良好的网络环境。如果你在某些步骤卡住，然后发现网络错误，或许可以考虑相关问题。

你需要安装这些软件：

- `Git`
- `CMake`
- `Ninja`
- `LLVM`
- `Visual Studio Community 2022`或[`Visual Studio 2022 Build Tools`](https://aka.ms/vs/17/release/vs_BuildTools.exe)

> 如果你嫌VS2022太大，并且用不到这个IDE，那么可以装Build Tools。

除VS2022以外的软件都可以通过`winget`安装：

```bash
winget install Git.Git Kitware.CMake LLVM.LLVM Ninja-build.Ninja
```

你需要保证这些东西都在环境变量`PATH`中。

## 项目构建

考虑这样一份代码：

```cxx
// main.cpp
#include <iostream>
using std::cout;
using std::endl;

void foo1()
{
    cout << "foo1" << endl;
}

int main()
{
    foo1();
    return 0;
}
```

你输入`clang++ main.cpp -o main.exe && main.exe`，果然得到了`foo1`的输出，这很好。

有一天，你发现你`foo1`写得太好了，经常会用到，或者你发现随着`main.cpp`越来越复杂，编译时间也越来越长，
总之你想要把`foo1`拆开：

```cxx
// foo2.cpp
#include <iostream>

using std::cout;
using std::endl;

void foo2()
{
    cout << "foo2" << endl;
}
```

```cxx
// foo2.h
void foo2();
```

```cxx
// main.cpp
#include "foo2.h"

int main()
{
    foo2();
    return 0;
}
```

你输入`clang++ main.cpp foo2.cpp -o main.exe && main.exe`，果然得到了`foo2`的输出，这仍然很好。

随着你的项目越来越复杂，你可能有越来越多的文件需要编译、链接，这个命令可能会变成这样：

```bash
clang -c foo1.cpp -o foo1.obj
clang -c foo2.cpp -o foo2.obj
clang -c foo3.cpp -o foo3.obj
...

clang main.cpp foo1.obj foo2.obj foo3.obj -o main.exe

```

此外，有时为了缩减编译时间，没有改动的内容我们不想再编译一次，但这就需要我们手工找出哪些文件改动了，哪些没有。

这很麻烦，不太好。

这个时候我们考虑引入`Makefile`：

```makefile
# 定义编译器
CC = clang

# 定义目标可执行文件
TARGET = main.exe

# 定义源文件和目标文件
SRCS = main.cpp foo1.cpp foo2.cpp foo3.cpp
OBJS = foo1.obj foo2.obj foo3.obj

# 默认目标
all: $(TARGET)

# 生成可执行文件
$(TARGET): main.cpp $(OBJS)
	$(CC) main.cpp $(OBJS) -o $(TARGET)

# 编译每个源文件为目标文件
%.obj: %.cpp
	$(CC) -c $< -o $@

# 清理生成的文件
clean:
	rm -f $(OBJS) $(TARGET)

.PHONY: all clean
```

这样每次你只需要敲一下`make`，所有的工作就会由`make`帮你完成了。

看起来很不错。

有人问：主播主播，你的Makefile确实很强，但还是太吃操作了，有没有更加简单强势的构建系统可以推荐一下吗？有的兄弟，有的。

我们希望通过一些**更高阶、更抽象**的构建描述，直接指定类似“某可执行文件依赖于某些源文件”、“某可执行文件在编译时需要到某些地方搜索头文件”和“某些文件需要被链接到某些库上”之类的指令，然后这个高阶构建系统可以**直接生成Makefile或者类似物**。

很幸运的是，`CMake`就是这样一个高阶构建系统。

> 这部分不会有极其详细的基础介绍，如果是非常新的新手，建议参考[CMake官方教程](https://cmake.org/cmake/help/latest/guide/tutorial/index.html)

考虑这样一个目录结构：

```raw
project/
├── include/
|   ├── foo1.h
|   ├── foo2.h
|   └── foo3.h
└── src/
    ├── foo1.cpp
    ├── foo2.cpp
    ├── foo3.cpp
    └── main.cpp
```

思考一下我们需要描述清楚哪些事情：

- 源文件有哪些
- 头文件去哪里找

那么写出来的`CMakeLists.txt`是这样的：

```cmake
# 指定 CMake 最低版本要求
cmake_minimum_required(VERSION 3.14)

# 设置项目名称和语言标准
project(MyProject LANGUAGES CXX)

# 设置 C++ 标准
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# 添加可执行文件目标，并包含所有源文件
# 这里我们定义了一个“目标”（Target）
add_executable(main
    src/main.cpp
    src/foo1.cpp
    src/foo2.cpp
    src/foo3.cpp
)

# 为可执行文件目标指定头文件目录
target_include_directories(main PRIVATE "include")
```

让`CMake`以`Makefile`为生成器来配置项目：

```bash
cmake -S . -B ./build -G "Unix Makefiles"
```

让`CMake`利用`Makefile`构建项目（编译链接）：

```bash
cmake --build ./build
```

> 上面这一步需要你有可以运行的`make`命令。
>
> 如有需要可以用`winget install ezwinports.make`来安装。
> 
> 这不是必须的，因为我们可以使用`Ninja`来替代`make`。

> `Ninja`是以速度见长的构建系统，它在设计上并不是让人手写的，而是利用`CMake`之类的工具来生成的。

让`CMake`以`Ninja`为生成器来配置项目：

```bash
cmake -S . -B ./build -G "Ninja"
```

让`CMake`利用`Ninja`构建项目（编译链接）：

```bash
cmake --build ./build
```

## 代码协作

终于有一天，你的同事/同学/小伙伴发现你的`foo`写得实在是太好了，他们也想用一用。
又或者你自己也需要使用其他人写好的代码（调库），这时候怎么办？

当然可以手动维护很多份拷贝，然后写一个特别复杂的构建脚本来代替你手动编译链接，
最后把产出的文件搬来搬去。

但这很麻烦，麻烦到了近乎不可能完成的地步。

上面我们利用`CMake`实现了构建的步骤。我们可以思考一下，如果我们需要让这些`foo`的结构一目了然，怎么办？

我们可以这样安排目录结构：

```raw
project/
├── CMakeLists.txt
├── include/
│   └── foo/
│       ├── foo1.h
│       ├── foo2.h
│       └── foo3.h
└── src/
    ├── foo/
    │   ├── foo1.cpp
    │   ├── foo2.cpp
    │   └── foo3.cpp
    └── main.cpp
```

然后把foo作为一个单独的库目标来构建：

```cmake
# 指定 CMake 最低版本要求
cmake_minimum_required(VERSION 3.14)

# 设置项目名称和语言标准
project(MyProject LANGUAGES CXX)

# 设置 C++ 标准
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# 添加库目标：libfoo
add_library(foo STATIC
    src/foo/foo1.cpp
    src/foo/foo2.cpp
    src/foo/foo3.cpp
)

# 为库目标指定头文件目录
target_include_directories(foo PUBLIC include)

# 添加可执行文件目标
add_executable(main src/main.cpp)

# 将库链接到可执行文件
target_link_libraries(main PRIVATE foo)
```

这样文件目录结构看起来就比较清晰了。

那么更进一步，有没有办法让别人也能用这个库呢？

<!-- TODO: 基本的CMakeLists.txt，CMake先配置再生成这个过程 -->
TODO: CMake怎么导出一个库，怎么用一个库，git clone，include，在没有包管理的前提下

TODO: 这太麻烦了，有没有像Python那样pip install就可以的办法？有的兄弟有的
TODO: 系统包管理（apt等）
TODO: vcpkg教学：安装、使用，例程
