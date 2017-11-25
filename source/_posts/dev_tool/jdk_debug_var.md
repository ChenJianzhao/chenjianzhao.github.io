---
categories: 开发工具
date: 2017-11-24 14:00
title: Windows 下编译 jdk 实现 dubug 查看本地变量
---

待翻译

Generally speaking, to be able to watch the variables while stepping through JDK source code, you need the class files to be compiled with debug information i.e. compile using `javac -g`.

According to [this post](http://www.javalobby.org/java/forums/t19866.html) (please note that I haven't tried it) you don't need to compile all sources, only the ones you need. Putting your newly compiled classes in the `$jdk/jre/lib/`~~ext/~~`endorsed`directory, the new classes would be used instead the ones in the original `rt.jar`.

1. Create your working folder. I chose `d:\` root folder
2. Inside your working folder create the source folder i.e. `jdk7_src` and output folder `jdk_debug`
3. From your `JDK_HOME` folder get the `src.zip` file and unzip it inside `jdk7_src`
4. Select what you will compile and delete the rest. For all of them you might need additional steps. I have chosen the folders:
   - `java`
   - `javax`
   - `org`
5. From your `JDK_HOME\jre\lib` get the file `rt.jar` and put in the work folder (this is only for convenience to not specify too large file names in the command line).
6. Execute the command: `dir /B /S /X jdk7_src\*.java > filelist.txt` to create a file named `filelist.txt` with the list of all java files that will be compiled. This will be given as input to `javac`
7. Execute `javac` using the command:
   `javac -J-Xms16m -J-Xmx1024m -sourcepath d:\jdk7_src -cp d:\rt.jar -d d:\jdk_debug -g @filelist.txt >> log.txt 2>&1` This will compile all the files in the `jdk_debug` folder and will generate a `log.txt` file in your working folder. Check the log contents. You should get a bunch of warnings but no error.
8. Go inside the `jdk_debug` folder and run the command: `jar cf0 rt_debug.jar *`. This will generate your new runtime library with degug information.
9. Copy that new jar to the folder `JDK_HOME\jre\lib\endorsed`. If the `endorsed` folder does not exist, create it.

[stackoverflow - debug jdk source can't watch variable what it is](https://stackoverflow.com/questions/18255474/debug-jdk-source-cant-watch-variable-what-it-is)