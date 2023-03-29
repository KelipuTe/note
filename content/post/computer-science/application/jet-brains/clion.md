---
draft: false
date: 2021-11-17 08:00:00 +0800
lastmod: 2021-11-17 08:00:00 +0800
title: "Clion 的使用"
summary: "编译和运行单个源码文件；"
toc: true

categories:
- application(应用)

tags:
- computer-science(计算机科学)
- application(应用)
- jet-brains
- clion
---
## 正文

### 编译和运行单个 C 源码文件

假设。有一个项目。在项目根目录下，有一个文件名为 hello_world.c 的源码文件。

打开源码文件。在代码编辑窗口中，右击鼠标。在弹出的菜单栏中，选择 "Add executable for signal c/cpp file"。这会在 "{项目目录}/CMakeLists.txt" 文件中添加一行 `add_executable(hello_world hello_world.c)`。

在 Clion 的 Project 工具窗口（一般在左侧）内，选中项目文件夹，右击鼠标。在弹出的菜单栏中，选择 "Reload Cmake Project"。这个时候，会在项目目录下生成 cmake-build-debug 文件夹。

在 Clion 的菜单栏下面一行的工具栏里面找到 "Select Run/Debug configuration"。点击 "Select Run/Debug configuration"。在弹出的菜单栏中，应该可以看到 hello_world。选中 hello_world。

"Select Run/Debug configuration" 右边有两个按钮 Run 和 Debug。在前面选中 hello_world 之后，这两个按钮会变成 "Run 'hello_world'" 和 "Debug 'hello_world'"。

点击 Run 按钮可以直接编译并运行 hello_world.c 源码文件。在 hello_world.c 源码文件打了断点之后，点击 Debug 按钮，可以以调试的方式启动。
