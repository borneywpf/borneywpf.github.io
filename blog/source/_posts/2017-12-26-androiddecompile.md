---
title: Android 反编译工具使用汇总
date: 2017-12-26 19:06:31
categories: android
tags:
  - android
  - 反编译
  - 工具

---

## 使用场景

作为Android开发，我们经常会反编译查看一些东西(比如查看其他优秀软件的源码)，最终都是为了将android的apk处理成能查看的java代码，本文主要讲述了一些工具的安装、作用和使用方式

## 工具篇

### apktool

  [官方地址](https://ibotpeaches.github.io/Apktool/),这个工具主要有以下特性：  
  - 将资源(dex,resources.arsc,.9.png,XMLS)等解码成原始形式
  - 可以将解码后的资源重新编译成APK/JAR
  - 可以设置依赖的framework资源

  下载地址: :https://ibotpeaches.github.io/Apktool/install/  
  安装方式: 将[脚本](https://raw.githubusercontent.com/iBotPeaches/Apktool/master/scripts/linux/apktool)保存在你的PATH环境制定的目录下，将[最新jar](https://bitbucket.org/iBotPeaches/apktool/downloads/)放在和上述脚本的目录下，修改上述脚本中 jarfile=apktool.jar 中的 apktool.jar 为你下载的jar包的名称或修改下载jar包名称为apktool.jar  

  使用方式：

  ```
  Apktool v2.3.0 - a tool for reengineering Android apk files
  with smali v2.2.1 and baksmali v2.2.1
  Copyright 2014 Ryszard Wiśniewski <brut.alll@gmail.com>
  Updated by Connor Tumbleson <connor.tumbleson@gmail.com>

  usage: apktool
  -advance,--advanced   prints advance information.
  -version,--version    prints the version then exits
  usage: apktool if|install-framework [options] <framework.apk>
  -p,--frame-path <dir>   Stores framework files into <dir>.
  -t,--tag <tag>          Tag frameworks using <tag>.
  usage: apktool d[ecode] [options] <file_apk>
  -f,--force              Force delete destination directory.
  -o,--output <dir>       The name of folder that gets written. Default is apk.out
  -p,--frame-path <dir>   Uses framework files located in <dir>.
  -r,--no-res             Do not decode resources.
  -s,--no-src             Do not decode sources.
  -t,--frame-tag <tag>    Uses framework files tagged by <tag>.
  usage: apktool b[uild] [options] <app_path>
  -f,--force-all          Skip changes detection and build all files.
  -o,--output <dir>       The name of apk that gets written. Default is dist/name.apk
  -p,--frame-path <dir>   Uses framework files located in <dir>.

  For additional info, see: http://ibotpeaches.github.io/Apktool/
  For smali/baksmali info, see: https://github.com/JesusFreke/smali
  ```

  使用示例：

  如果以来平台和设备需要设置framework环境  

  ```
  $apktool if -p <输出目录> framework.apk
  ```

  反编译  

  ```
  $apktool d -p <上述的framework的安装目录> -o <输出目录> xxx.apk
  ```

  重新编译打包

  ```
  $apktool b -p <上述的framework的安装目录> -o xxx.apk <输入目录>
  ```

  其他参数请大家自己实验.

### smalidea

  上述使用apktool反编译生成的是smali文件，这个文件我个人理解主要为为了方便开发理解dex二进制文件而设计的，比如用javap可以查看class文件一个道理.

  [smalidea](https://github.com/JesusFreke/smali/wiki/smalidea)是一个 IntelliJ IDEA插件，自然也是可以在Android Studio上安装，这样就方便我们查看饭编译生成的smali文件，并且可以debug.

### baksmali/smali

  在Android6.0之后，为了提升android应用程序的运行性能，google在art之上引入了oat的过程，应用程序安装会做一次odex，把dex文件进一步优化成一个odex文件.

  项目地址: https://github.com/JesusFreke/smali/  
  下载地址: https://bitbucket.org/JesusFreke/smali/downloads/  
  安装方式: 下载jar包，copy一份上述的apktool脚本创建smali和baksmali两个脚本，分别修改其中的 jarfile，并加入环境变量PATH中.

  baksmali的作用就是能将odex文件饭编译成smali文件

  使用方式:

  ```
  usage: baksmali [--version] [--help] [<command [<args>]]

  Options:
    --help,-h,-? - Show usage information
    --version,-v - Print the version of baksmali and then exit

  Commands:
    deodex(de,x) - Deodexes an odex/oat file
    disassemble(dis,d) - Disassembles a dex file.
    dump(du) - Prints an annotated hex dump for the given dex file
    help(h) - Shows usage information
    list(l) - Lists various objects in a dex file.

  See baksmali help <command> for more information about a specific command

  ```

  使用示例:

  如果依赖平台的需要先pull出framework

  ```
  $adb pull /system/framework framework
  $baksmali deodex app.odex -b framework/arm/boot.oat -o <输出目录>
  ```

  smali的作用是将smali文件重新打包成dex文件

  使用方式:

  ```
  usage: smali [-v] [-h] [<command [<args>]]

  Options:
    -h,-?,--help - Show usage information
    -v,--version - Print the version of baksmali and then exit

  Commands:
    assemble(ass,as,a) - Assembles smali files into a dex file.
    help(h) - Shows usage information

  See smali help <command> for more information about a specific command
  ```

  使用示例:

  ```
  smali assemble <输入目录> -o xxx.dex
  ```

### jadx

  [jadx](https://github.com/skylot/jadx)是一个将Android apk或dex文件通过命令行生产java源码，或者用GUI工具查看.

  下载地址:https://github.com/skylot/jadx/releases

  使用方式:

  ```
  usage: jadx [options] <input file> (.dex, .apk, .jar or .class)
  options:
   -d, --output-dir           - output directory
   -j, --threads-count        - processing threads count
   -r, --no-res               - do not decode resources
   -s, --no-src               - do not decompile source code
   -e, --export-gradle        - save as android gradle project
       --show-bad-code        - show inconsistent code (incorrectly decompiled)
       --no-replace-consts    - don't replace constant value with matching constant field
       --escape-unicode       - escape non latin characters in strings (with \u)
       --deobf                - activate deobfuscation
       --deobf-min            - min length of name
       --deobf-max            - max length of name
       --deobf-rewrite-cfg    - force to save deobfuscation map
       --deobf-use-sourcename - use source file name as class name alias
       --cfg                  - save methods control flow graph to dot file
       --raw-cfg              - save methods control flow graph (use raw instructions)
   -f, --fallback             - make simple dump (using goto instead of 'if', 'for', etc)
   -v, --verbose              - verbose output
   -h, --help                 - print this help
  Example:
   jadx -d out classes.dex
  ```

  GUI工具效果如下:
  ![jadx](https://camo.githubusercontent.com/bd3c0ea851c23c4535e43590a86c940a0786faa6/687474703a2f2f736b796c6f742e6769746875622e696f2f6a6164782f6a6164782d6775692e706e67)  

### dex2jar

  [dex2jar](https://github.com/pxb1988/dex2jar)可以将dex转换成jar包，也可以将dex转换成smali或这将smali转换成dex，不过最后这两个我没用过，大家就自行研究吧.

  下载地址:https://github.com/pxb1988/dex2jar/releases  

  使用方式:

  ```
  d2j-dex2jar -- convert dex to jar
  usage: d2j-dex2jar [options] <file0> [file1 ... fileN]
  options:
   -d,--debug-info              translate debug info
   -e,--exception-file <file>   detail exception file, default is $current_dir/[fi
                                le-name]-error.zip
   -f,--force                   force overwrite
   -h,--help                    Print this help message
   -n,--not-handle-exception    not handle any exception throwed by dex2jar
   -nc,--no-code
   -o,--output <out-jar-file>   output .jar file, default is $current_dir/[file-na
                                me]-dex2jar.jar
   -os,--optmize-synchronized   optmize-synchronized
   -p,--print-ir                print ir to Syste.out
   -r,--reuse-reg               reuse regiter while generate java .class file
   -s                           same with --topological-sort/-ts
   -ts,--topological-sort       sort block by topological, that will generate more
                                 readable code, default enabled
  version: reader-2.0, translator-2.0, ir-2.0

  ```

  使用示例:

  ```
  sh d2j-dex2jar.sh -f xxx.apk -o 输出.jar
  ```

### jd-gui

  [jd-gui](http://jd.benow.ca/)是一个独立的图形工具，显示".class"文件的Java源代码。可以使用JD-GUI浏览重建的源代码，以便即时访问方法和字段;这个有不同系统的版本，可以自行安装体验

  效果如下:  
  ![jd-gui](http://jd.benow.ca/img/screenshot17.png)

### getdebug脚本
  在我的[script](https://github.com/borneywpf/script)工程中，我自己写了一个[getdebug.py](https://github.com/borneywpf/script/blob/master/android/getdebug.py)脚本,这个脚本的作用就是通过以上的部分工具将手机中指定的包名或apk(非debug版本)处理成一个debug版本的apk

  使用方式:

  ```
  Usage: getdebug.py -a <apksigner file> --pk8=<pk8 file> --pem=<pem file> arg1 arg2...

  Generate debug apk by packageName or apk file.

  Options:
    -h, --help            show this help message and exit
    --apksigner=<file>    the apksigner file(签名工具)
    --pk8=<file>          the pk8 sign file(pk8签名文件)
    --pem=<file>          the pem sign file(pem签名文件)
    --arm=<32|64>         the arm type(arm类型，主要用于andorid6.0之后的odex的反编译)
    -a <android version code>
                          the android version code(android版本号)
    -d                    debug this script
  ```

## 总结
  至此，本文就结束了，主要讲述了Android开发过程中，常用的反编译工具，以及我自己写的一个自动化脚本.谢谢大家!!!
