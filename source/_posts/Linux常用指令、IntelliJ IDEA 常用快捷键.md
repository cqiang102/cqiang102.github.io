---
title: Linux 常用指令、IntelliJ IDEA 常用快捷键
date: 2019-11-17 19:19:27
categories: 笔记
tags: [Linux,IntelliJ IDEA]
type: posts
description: Linux 常用指令、IntelliJ IDEA 常用快捷键,缓慢持续更新ing...
---
# Linux 篇
- 查询占用此端口的程序 。
	- `lsof -i:{port}`  。 例：`lsof -i:3306` 。
	- `netstat -tunlp|grep {port}` 。 例：`netstat -tunlp|grep 8080` 。
- 查看端口使用情况 `netstat -anp` 。
- 查看本机资源使用情况 `top` 或安装（apt-get install htop）`htop`可实时变化 。
- 杀死进程 `kill {pid}`  。例：`kill 7821` 。
- 查进程 `ps aux | grep ***` 。
- 查日志 `tail -f catalina.out` 。
- 打 tar 包 `tar zcvf FileName.tar.gz DirName` 。
- 解压 tar 包 `tar zxvf FileName.tar.gz` 。
- 修改 hostname `hostnamectl set-hostname you-hostname`
- 查看网关地址 `route -n`
- 配置 hosts `sudo vim /etc/hosts`
- ubuntu18.04 取消 dhcp ,修改 ip 地址
```

vi /etc/netplan/50-cloud-init.yaml

network:
    ethernets:
        ens33:
          addresses: [192.168.80.160/24]
          gateway4: 192.168.80.2
          nameservers:
            addresses: [114.114.114.114,8.8.8.8]
    version: 2

# 使配置生效
netplan apply
```
- 查日志常用 `less` 的一些操作

```shell
# `G`   到日志末尾
# `?`   往上面查找
# `/`   往下面查找
# `n`   下一个匹配项
# `N`   上一个匹配项
# `j/k` 上一行/ 下一行
```

- 修改系统环境变量 `sudo vim /etc/proflie`

- root 账户下使用其他用户运行程序 `sudo -u username /usr/bin/appname`

- 添加应用图标

  ```shell
  sudo vim /usr/share/applications
  # 复制一个原来的图标，对照修改
  # 例：system-config-printer.desktop
  
  [Desktop Entry]
  Name=Printers # 名字
  GenericName=Printers
  X-GNOME-FullName=Printers
  Comment=Configure printers
  Exec=system-config-printer # 执行的程序
  Terminal=false
  Type=Application
  Icon=printer # 图标
  StartupNotify=true
  NotShowIn=KDE;GNOME;
  X-Ubuntu-Gettext-Domain=system-config-printer
  Categories=GNOME;GTK;Settings;HardwareSettings;X-GNOME-Settings-Panel;X-Unity-Settings-Panel;System;Printing; # 分类
  X-GNOME-Settings-Panel=printing
  X-Unity-Settings-Panel=printing
  Keywords=Printer;Queue;Print;Paper;Ink;Toner;
  X-Desktop-File-Install-Version=0.24
  
  ```

- Maven,Flutter,Java 等系统环境变量配置

  ```shell
  vim /etc/profile
  
  export JAVA_HOME=/usr/local/java/jdk1.8.0_171
  export JRE_HOME=/usr/local/java/jdk1.8.0_171/jre
  export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib
  export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH:$HOME/bin
  
  export M2_HOME=/usr/local/apache-maven-3.6.3
  export PATH=${M2_HOME}/bin:$PATH
  
  export PUB_HOSTED_URL=https://pub.flutter-io.cn
  export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
  export FLUTTER_HOME=/usr/local/flutter
  export PATH=${PATH}:${FLUTTER_HOME}/bin
  ```


- 查看当前目录已经占用了多大空间，需要进入目录后输入命令 `du -sh`

# IntelliJ IDEA 篇

- 万能快捷键 `Alt`+`Enter` 。
- XML 或者 HTML 中选中一个节点 `Ctrl`+`W` 。
- 删除光标所在行 `Ctrl`+`Y` 。
- 双击 `Shift` 搜索 。
- 重命名 `Ctrl`+`Shift`+`R` 。
- 返回上次查看代码的位置, `Alt`+`←`、 `Alt`+`→` 。 
- 折叠/ 展开代码块的快捷键 。
	- `Ctrl` + `+/-` , 当前方法展开、折叠  。
	- `Ctrl` + `Shift` + `+/-` , 全部展开、折叠 。
- IDEA 内切换窗口 `Ctrl` + `Alt` + `[/]` 
- 关闭当前打开的文件选项卡 `Ctrl` + `F4`
- 向下复制一行 `Ctrl` + `D`
