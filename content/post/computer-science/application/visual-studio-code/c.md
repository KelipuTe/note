---
draft: false
date: 2022-05-09 08:00:00 +0800
lastmod: 2023-04-10 08:00:00 +0800
title: "在 Visual Studio Code 中，配置 C 语言开发环境"
summary: "在 Visual Studio Code 中，配置 C 语言开发环境"
toc: true

categories:
- application(应用)

tags:
- computer-science(计算机科学)
- application(应用)
- ccpp
---
## 前言

实践的环境：

- CPU AMD64(x86_64)
- Windows 11 家庭版
- x86_64-pc-cygwin
- gcc 版本 11.2.0 (GCC)
- Visual Studio Code 1.77.1

公共的操作放在这篇里面：[在 Windows 11 中，安装和配置 Visual Studio Code](/post/computer-science/application/visual-studio-code/windows)

## 正文

### 安装 C/C++ 编译器

先把 C 语言开发环境装好：[在 Windows 11 中配置 C 语言开发环境](/post/computer-science/programming-language/c/windows)

### 安装 VSCode 扩展

- （可选）安装 Chinese (Simplified) (简体中文) Language Pack for Visual Studio Code 组件（简体中文语言包）。
- 安装 C/C++ 组件。
- 安装 Code Runner 组件。
- （可选）安装 CMake 组件。
- （可选）安装 CMake Tools 组件。
- （可选）安装 Clang-Format 组件（代码格式化）。
- （可选）安装 Git Graph 组件（查看 Git 树，执行 Git 操作）。
- （可选）安装 Git History 组件（查看 Git 历史）。
- （可选）安装 GitLens 组件（查看代码作者）。
- （可选）安装 Markdown All in One 组件。

### 配置 C/C++ 组件

调出命令面板。在命令面板的输入框中，输入 "C/C++"。然后，在下面的列表中，找到并点击 "C/C++: 编辑配置(UI)"（没安装简体中文语言包的话，应该是 "C/C++: Edit Configurations(UI)"），进入 "IntelliSense 配置" 标签页。

配置 "编译器路径"，我这里是 "C:\cygwin64\bin\gcc.exe"。配置 "IntelliSense 模式"，我这里选的是 "windows-gcc-x64"。

配置完成后，会自动在项目根目录的 ".vscode/" 目录（没有会自动创建）下，创建一个 c_cpp_properties.json 文件，文件的内容如下。这文件一般不需要改。

```
{
    "configurations": [
        {
            "name": "Win32",
            "includePath": [
                "${workspaceFolder}/**"
            ],
            "defines": [
                "_DEBUG",
                "UNICODE",
                "_UNICODE"
            ],
            "compilerPath": "C:\\cygwin64\\bin\\gcc.exe",
            "cStandard": "c17",
            "cppStandard": "gnu++17",
            "intelliSenseMode": "windows-gcc-x64",
            "configurationProvider": "ms-vscode.cmake-tools"
        }
    ],
    "version": 4
}
```

### 配置 C/C++ 任务文件

先编辑一个 C 源码文件。比如，我在根目录下，写了一个 hello_world.c。然后，在这个文件的编辑页面里。注意，这里一定要在 C 源码文件的编辑页面里。要不然下面的操作，一些选项出不来。

在命令面板的输入框中，输入 "Tasks"。然后，在下面的列表中，找到并点击 "任务: 配置默认生成任务"（没安装简体中文语言包的话，应该是 "Tasks: Configure Default Build Task"）。

然后，在下面的列表中，找到并点击 "C/C++: gcc.exe 生成活动文件"（没安装简体中文语言包的话，应该是 "C/C++ gcc.exe build active file"）。

或者，在最上面的菜单栏，点击 "终端"，在弹出的菜单中，点击 "配置任务"。然后，在弹出的列表中，找到并点击 "C/C++: gcc.exe 生成活动文件"。

这两个操作的效果是一样的。会自动在项目根目录的 ".vscode/" 目录（没有会自动创建）下，创建一个 tasks.json 文件，文件的内容如下。鼠标移动到键名上会显示备注。

```
{
	"version": "2.0.0",
	"tasks": [
		{
			"type": "cppbuild",
			"label": "C/C++: gcc.exe 生成活动文件",
			"command": "C:\\cygwin64\\bin\\gcc.exe",
			"args": [
				"-fdiagnostics-color=always",
				"-g",
				"${file}",
				"-o",
				"${fileDirname}\\${fileBasenameNoExtension}.exe"
			],
			"options": {
				"cwd": "${fileDirname}"
			},
			"problemMatcher": [
				"$gcc"
			],
			"group": "build",
			"detail": "编译器: C:\\cygwin64\\bin\\gcc.exe"
		}
	]
}
```

- label，任务名称。可以改的，launch.json 里面要用。
- command，执行编译的编译器或脚本的路径。改成 gcc 编译器的路径。
- args，其他要传递给编译器或编译脚本的参数。就是传给 gcc 命令的参数。

在最上面的菜单栏，点击 "终端"，在弹出的菜单中，点击 "运行任务"。然后，在弹出的列表中，找到并点击 "C/C++ gcc.exe build active file"。

然后，就会执行 tasks.json 里面的内容。并在 hello_world.c 文件所在的目录下，生成一个 hello_world.exe 可执行文件。在最下面的 "终端" 子窗口里面，会输出以下内容。

```
正在执行任务: C/C++: gcc.exe 生成活动文件 

正在启动生成...
C:\cygwin64\bin\gcc.exe -fdiagnostics-color=always -g D:\workspace\demo-c\hello_world.c -o D:\workspace\demo-c\hello_world.exe

生成已成功完成。
 *  终端将被任务重用，按任意键关闭。
```

这里执行的命令，就对应上面 tasks.json 里面的内容。

- `${file}` 表示当前选中的文件名。对应 "D:\workspace\demo-c\hello_world.c"。
- `${fileDirname}` 表示当前选中的文件所在的目录。对应 "D:\workspace\demo-c"。
- `${fileBasenameNoExtension}` 表示当前选中的文件去掉文件后缀。对应 "hello_world"。
- 其他的也可以对应上，这里就不多说了。

然后，在最下面的 "终端" 子窗口里面，使用 `.\hello_world` 命令（Windows 里面是右斜线），就可以在运行刚才生成的 hello_world.exe 可执行文件。

### 断点调试

找一个源码文件，打上断点。

在最上面的菜单栏，点击 "运行"，在弹出的菜单中，点击 "添加配置"。然后，在弹出的列表中，找到并点击 "C++ GDB/LLDB"。

这个操作会自动在项目根目录的 ".vscode/" 目录（没有会自动创建）下，创建一个 launch.json 文件，文件的内容如下。

```
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": []
}
```

然后，在最上面的菜单栏，点击 "运行"，在弹出的菜单中，点击 "添加配置"。然后，在弹出的列表中，找到并点击 "C/C++: (gdb) 启动"。

这个操作会自动在 launch.json 文件里添加配置代码。鼠标移动到键名上会显示备注。

```
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(gdb) 启动",
            "type": "cppdbg",
            "request": "launch",
            "program": "输入程序名称，例如 ${workspaceFolder}/a.exe",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${fileDirname}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "miDebuggerPath": "/path/to/gdb",
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                },
                {
                    "description": "将反汇编风格设置为 Intel",
                    "text": "-gdb-set disassembly-flavor intel",
                    "ignoreFailures": true
                }
            ]
        }
    ]
}
```

需要根据自己的配置，对配置代码做一些修改。

```
{
    "configurations": [
        {
            "name": "C/C++: gcc.exe 调试活动文件",
            "type": "cppdbg",
            "request": "launch",
            "program": "${fileDirname}\\${fileBasenameNoExtension}.exe",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${fileDirname}",
            "environment": [],
            "externalConsole": true,
            "MIMode": "gdb",
            "miDebuggerPath": "C:\\cygwin64\\bin\\gdb.exe",
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                },
                {
                    "description": "将反汇编风格设置为 Intel",
                    "text": "-gdb-set disassembly-flavor intel",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "C/C++: gcc.exe 生成活动文件"
        }
    ]
}
```

- name，配置名称。可以改的。
- program，程序可执行文件的完整路径。这里改成和 tasks.json 里的 args 一样。
- miDebuggerPath，MI 调试程序(如 gdb)的路径。改成 gdb 的路径。
- 添加 preLaunchTask 配置，调试会话开始前要运行的任务。改成 tasks.json 里对应任务的 label。

### 多文件编译

上面配置的 tasks.json 只能编译单个文件。想编译多个文件可以通过修改 tasks.json 的 args 实现。直接把要编译的文件写进去，就和手写 gcc 命令差不多。

```json
"args": [
	"-g",
	"${fileDirname}\\hello1.c",
	"${fileDirname}\\hello2.c",
	"-o",
	"${fileDirname}\\hello.exe"
],
```

- 直接写 makefile。
- 用 CMake 工具。VSCode 的扩展里有 CMake 插件和 CMake Tools 插件。

### 代码格式化

#### LLVM

先安装 LLVM。去 [官网](https://llvm.org/) 页面左边 Download 那里，点击 "All Releases"，进入 [发布页面] (https://releases.llvm.org/)。

发布页面往下滑，就直接点最新的版本后面的 download 链接，这里是 16.0.0 版本的 [链接](https://github.com/llvm/llvm-project/releases/tag/llvmorg-16.0.0)。它会跳到 github 的发布页面。找对应的版本就好了，我这里是 LLVM-16.0.0-win64.exe。

安装结束后，需要配置环境变量。这里和安装 GCC 的时候一样，就不重复了。

- 在环境变量窗口下面的 "系统变量" 里找到 path，然后点击 "编辑，打开 "编辑环境变量" 窗口"。
- 添加 "LLVM 的安装目录下的 bin 目录的路径"。我这里，这个目录是 "C:\Program Files\LLVM\bin"。

#### 配置 VSCode

打开设置标签页。设置 Editor: Default Formatter 为 Clang-Format。

然后在扩展工具栏中，选择 Clang-Format。在弹出的 Clang-Format 组件的描述标签页，会介绍怎么配置，主要是下面这个部分。

> Specifying the location of clang-format</br>
> This extension will attempt to find clang-format on your PATH. Alternatively, the clang-format executable can be specified in your vscode settings.json file:</br>

```
{
    "clang-format.executable": "/absolute/path/to/clang-format"
}
```

我这里就是在 settings.json 里，把这个配置改成 "C:\Program Files\LLVM\bin\clang-format.exe。

打开 C/C++ 组件的设置标签页。下面这两个方式都可以。

- 在扩展工具栏中，选择 C/C++。在弹出的 C/C++ 组件的描述标签页，点击的标题下面的设置图标，在弹出的下拉菜单中，选择扩展设置。
- 在设置标签页上面的输入框中，输入 "@ext:ms-vscode.cpptools"。

在 C/C++ 组件的设置标签页中。

- 检查一下 C_Cpp: Formatting 设置项是不是 default 或者 clangFormat，这个是默认配置一般不会变。
- 检查一下 C_Cpp: Clang_format_style 设置项是不是 file，设置为 file 表示格式化的时候会先寻找 Clang-Format 组件的格式化配置文件。

最后，我的 settings.json 是下面这样的。

```
{
    "editor.formatOnSave": true,
    "editor.defaultFormatter": "xaver.clang-format",
    "clang-format.executable": "C:\\Program Files\\LLVM\\bin\\clang-format.exe"
}
```

接着在项目根目录新建一个 ".clang-format" 文件，这个文件就是上面说的格式化配置文件。详细的配置方式可以去 Clang-Format 的文档里去看，Clang-Format 组件的描述标签页就可以找到链接。

```
#每行字符的限制，0表示没有限制
ColumnLimit: 0
#缩进宽度
IndentWidth: 2
#连续空行的最大数量
MaxEmptyLinesToKeep: 1
#缩进case标签
IndentCaseLabels: true
```

配置完成就可以格式化代码了，默认快捷键是 "Shift+Alt+F"。也可以在需要格式化的文件的编辑窗口里，点击鼠标右键，在弹出的菜单里，选择 "格式化文档"。

### 使用 Code Runner 运行代码

安装 Code Runner 组件后，右上角的 "运行按钮" 那里，会多一个 Run Code 的选项。

选中 hello_world.c 文件，然后，点击右上角的运行按钮，选择 Run Code 选项。这个操作，会在下面的 "终端" 子窗口里面，输出以下内容，相当于编译了 hello_world.c 文件，然后执行了生成的 hello_world.exe 可执行文件。

```
[Running] cd "d:\workspace\demo-c\" && gcc hello_world.c -o hello_world && "d:\workspace\demo-c\"hello_world
hello, world
```

## 参考

- [Vscode配置C/C++环境](https://blog.csdn.net/Hudiscount/article/details/120209994)
- [vscode调试c++断点失效解决方法](https://blog.csdn.net/changyana/article/details/122708559)
