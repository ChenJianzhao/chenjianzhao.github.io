## Mac下同时安装多个版本的JDK

## 下载 JDK

[JDK1.6（含源码）](https://download.developer.apple.com/Developer_Tools/java_for_os_x_2013005_developer_package/java_for_os_x_2013005_dp__11m4609.dmg) 需要登录 AppleID

[JDK1.6（不含源码）](https://support.apple.com/kb/DL1572?viewlocale=zh_CN&locale=zh_CN)

[JDK1.7（含源码）](https://download.developer.apple.com/Developer_Tools/java_for_mac_os_x_10.6_update_17_developer_package/java_for_mac_os_x_10.6_update_17_dp__10m4609.dmg)需要登录 AppleID

[JDK1.7（不含源码）](https://support.apple.com/kb/DL1573?viewlocale=zh_CN&locale=zh_CN)

[JDK1.8（目前最新版）](https://www.java.com/zh_CN/download/mac_download.jsp)



## 配置 JAVA_HOME

在用户目录下的 bash 配置文件.bash_profile 中配置 JAVA_HOME的路径：

```
export JAVA_6_HOME=/Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Home
export JAVA_7_HOME=/Library/Java/JavaVirtualMachines/jdk1.7.0.jdk/Contents/Home
export JAVA_8_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0.jdk/Contents/Home

export JAVA_HOME=$JAVA_8_HOME
```



创建alias命令动态切换`JAVA_HOME`的配置

```
alias jdk8='export JAVA_HOME=$JAVA_8_HOME'
alias jdk7='export JAVA_HOME=$JAVA_7_HOME'
alias jdk6='export JAVA_HOME=$JAVA_6_HOME'
```



切换版本验证

```
## 使文件生效
source .bash_profile 

## 查看当前版本
java -version
java version "1.8.0_131"
Java(TM) SE Runtime Environment (build 1.8.0_131-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.131-b11, mixed mode)

## 切换版本
jdk6

## 查看切换后的版本
java -version
java version "1.6.0_65"
Java(TM) SE Runtime Environment (build 1.6.0_65-b14-468)
Java HotSpot(TM) 64-Bit Server VM (build 20.65-b04-468, mixed mode)
```