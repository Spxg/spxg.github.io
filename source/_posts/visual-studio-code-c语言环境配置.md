---
title: Visual Studio Code C/C++环境配置
tags: []
id: '589'
categories:
  - - 技术相关
date: 2019-09-20 22:04:34
---

Visual Studio Code 与 Visual Studio IDE 的区别

根据[官方的说法](https://code.visualstudio.com/docs/supporting/faq#_what-is-the-difference-between-visual-studio-code-and-visual-studio-ide)有

## What is the difference between Visual Studio Code and Visual Studio IDE?[#](https://code.visualstudio.com/docs/supporting/faq#_what-is-the-difference-between-visual-studio-code-and-visual-studio-ide)

Visual Studio Code is a streamlined code editor with support for development operations like debugging, task running, and version control. It aims to provide just the tools a developer needs for a quick code-build-debug cycle and leaves more complex workflows to fuller featured IDEs, such as [Visual Studio IDE](https://visualstudio.microsoft.com/). 按照我的理解就是前者是个轻量级代码编辑器， 手动装好拓展和相关的编译器，就可以编写调试相应的程序，主要针对文件而不是项目，是Sublime或Atom on Electron的竞争对手；而Visual Studio IDE一看“IDE”就感觉不简单， IDE即Integrated Development Environment， 是个集成开发环境，微软旨在让其成为世界上最好的IDE，其功能强大的同时程序也不小。相比之下，Visual Studio 才是我们初选者的选择。

安装Visual Studio Code

Visual Studio Code 支持的平台有自家的Windows， MacOS，以及Linux，[点我跳官网下载](https://code.visualstudio.com/)，但是Visual Studio Code 在Windows上配置相对麻烦（在MacOS直接编译C程序时，其系统会自动弹出安装GCC的选项；在Linux应该装个GCC就好了（这个我还真没试过））,直接全选了，毕竟还挺实用的，然后安装即可。 ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/09/QQ截图20190920202803.png)

配置Visual Studio Code

先创建一个写代码的地方吧，比如Code，然后随意在其中创建一个C文件，会弹出下载C拓展的选项，点击安装（Install）即可，如图 ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/09/QQ截图20190920203656.png) 拓展是有了，但是编译器还没，根据[官方文档](https://code.visualstudio.com/docs/cpp/config-mingw)，使用的是[Mingw](http://mingw-w64.org/doku.php/start)，[下载地址](http://sourceforge.net/projects/mingw-w64/files/Toolchains%20targetting%20Win32/Personal%20Builds/mingw-builds/installer/mingw-w64-install.exe/download)，打开安装程序，如下图选择（应该没人用32位了吧![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/07/5b6603441f7e1552.png)） ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/09/QQ截图20191127191914.png) 这里以我的安装目录为演示，以下步骤都以这个目录为示范，目录不同请自行修改下面的配置文件 ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/09/QQ截图20191127192158.png) 然后在终端打入path

setx path "%path%;c:\\Software\\mingw64\\bin"

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/09/QQ截图20190920205756.png) 接下来生成配置文件，进入Code文件夹，按下Ctrl+Shift+P， 如图选择 ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/09/QQ截图20190920210340.png) 然后我们输入path地址及选择编译器 ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/09/QQ截图20190920210455.png) 我修改了标准参数（c11->c99) ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/09/QQ截图20190920210536.png) 接下来创建task.json, 官方有

## Create a build task[#](https://code.visualstudio.com/docs/cpp/config-mingw#_create-a-build-task)

Next, create a `tasks.json` file to tell VS Code how to build (compile) the program. This task will invoke the g++ compiler to create an executable file based on the source code.

1.  From the main menu, choose **View > Command Palette** and then type "task" and choose **Tasks: Configure Default Build Task**. In the dropdown, select **Create tasks.json file from template**, then choose **Others**. VS Code creates a minimal `tasks.json` file and opens it in the editor. （可以直接在.vscode中创建个task.json,然后copy进去保存即可）
2.  Go ahead and replace the entire file contents with the following code snippet （下面是我的配置）:
    
    {
        "tasks": \[
            {
                "type": "shell",
                "label": "Build",
                "command": "D:\\\\Software\\\\mingw64\\\\bin\\\\gcc.exe",
                "args": \[
                    "-g",
                    "${file}",
                    "-o",
                    "${fileDirname}\\\\${fileBasenameNoExtension}.exe",
                    "-std=gnu99",
                    "-fexec-charset=GBK",
                    "-finput-charset=UTF-8"
                \],
                "options": {
                    "cwd": "D:\\\\Software\\\\mingw64\\\\bin"
                },
                "group": {
                    "kind": "build",
                    "isDefault": true
                },
                "presentation": {
                    "reveal": "silent"
                }
            }
        \],
        "version": "2.0.0"
    }
    
     

官方对于launch.json有

## Configure debug settings

Next, we'll configure VS Code to launch the GCC debugger (gdb.exe) when you press F5.

1.  From the Command Palette, type "launch" and then choose **Debug: Open launch.json**. Next, choose the **GDB/LLDB** environment.
2.  For `program`, use the program name `helloworld.exe` (which matches what you specified in `tasks.json`). You will need to adjust your `miDebuggerPath` value to match the path to your Mingw-w64 installation.
3.  By default, the C++ extension adds a breakpoint to the first line of `main`. The `stopAtEntry` value is set to `true` to cause the debugger to stop on that breakpoint. You can set this to `false` if you prefer to ignore it.
4.  Optionally, set `externalConsole` to `true` to run the program in an external console.

Your complete `launch.json` file should look something like this （已修改，可以直接在.vscode中创建个launch.json,然后copy进去保存即可）:

{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": \[
        
        {
            "name": "gcc.exe build and debug active file",
            "type": "cppdbg",
            "request": "launch",
            "program": "${fileDirname}\\\\${fileBasenameNoExtension}.exe",
            "args": \[\],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": \[\],
            "externalConsole": false,
            "MIMode": "gdb",
            "miDebuggerPath": "D:\\\\Software\\\\mingw64\\\\bin\\\\gdb.exe",
            "setupCommands": \[
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            \],
            "preLaunchTask": "Build"
        }
    \]
}

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/09/QQ截图20190920212812.png) 全部创建完成后，即可在该目录下写C了，换目录时记得把.vscode文件复制过去

Visual Studio Code 的功能

编译运行当然没有问题

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/09/QQ截图20190920213458.png)

调试当然正常

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/09/QQ截图20190920213608.png)

代码补全当然好用

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/09/QQ截图20190920213655.png)

还有很好的规范学习工具——格式化功能

![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/09/QQ截图20190920213747.png)

安装中文拓展

进入拓展，搜索安装并重启软件即可 ![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/09/QQ截图20190920214456.png)

对于一些功能的思考

1.  代码补全对于我这种新手来说有利也有弊，利的方面自然是写的快了，但可能会因此忘掉语法的写法，这是致命的（上机考试的软件可没有代码补全），所以我这种新手觉得还是多自己写写，而代码补全可以作为学习的工具来研究语法该怎样写
2.  格式化代码是个好东西，它可以让我们的代码更规范，但是我们不能因为这功能而写出杂乱的代码，我们应该学习如何写，格式化的代码值得研究

相关文件（供网络不通![](https://wordpress-1253676827.file.myqcloud.com/wp-content/uploads/2019/07/5b6603441f7e1552.png)的朋友下载）及懒人直接配置文件（copy大法好）

[一键轻松](https://spxg-my.sharepoint.com/:f:/g/personal/spxg_spxg_me/Ev-LK1V5bStGk4b6_8hSw9ABnV2_X9prihAWeJY19tVBlw?e=CoXdwY)

 

引用文章

1.  [Using Mingw-w64 in VS Code](https://code.visualstudio.com/docs/cpp/config-mingw)
2.  [Visual Studio 2017和Visual Studio Code的区别！](https://blog.csdn.net/weibo1230123/article/details/89259510)