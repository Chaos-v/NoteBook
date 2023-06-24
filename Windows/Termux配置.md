*@DATE: 2023/04/21 19:37:17*

---

## Termux 安装

- 从 GitHub 或者 F-droid 下载并安装 Termux 

**注意：** 上述两个安装包需要从同一个安装源安装，安装后，可根据需要授予相应的访问权限。

---

## Termux 换源

以清华源为例，还原后使用下列命令行更新所有包

```bash
apt update && apt upgrade
```

### 图形界面（TUI）替换

在较新版的 Termux 中，官方提供了图形界面（TUI）来半自动替换镜像，推荐使用该种方式以规避其他风险。 在 Termux 中执行如下命令

```bash
termux-change-repo
```

在图形界面引导下，使用自带方向键可上下移动。
第一步使用空格选择需要更换的仓库，之后在第二步选择 TUNA/BFSU 镜像源。确认无误后回车，镜像源会自动完成更换。

### 命令行替换

使用如下命令行替换官方源为 TUNA 镜像源

```bash
sed -i 's@^\(deb.*stable main\)$@#\1\ndeb https://mirrors.tuna.tsinghua.edu.cn/termux/apt/termux-main stable main@' $PREFIX/etc/apt/sources.list
apt update && apt upgrade
```

### 手动修改

编辑 $PREFIX/etc/apt/sources.list 修改为如下内容

```bash
# The termux repository mirror from TUNA:
deb https://mirrors.tuna.tsinghua.edu.cn/termux/apt/termux-main stable main
```

请使用内置或安装在 Termux 里的文本编辑器，例如 vi / vim / nano 等，不要使用 RE 管理器等其他具有 ROOT 权限的外部 APP 来修改 Termux 的文件。

---

## 获取手机内部存储

获取共享空间命令行：

```bash
termux-setup-storage
```

---

## 调用安卓原生能力

1. 安装 Termux:API 
2. 在 Termux 终端中安装对应的包后才可以与手机底层硬件进行交互。

    ```bash
    pkg install termux-api

    # 测试命令
    termux-battery-status

    # output
    {
    "health": "GOOD",
    "percentage": 100,
    "plugged": "UNPLUGGED",
    "status": "DISCHARGING",
    "temperature": 24.600000381469727
    }
    ```

---

## 安装 Linux

1. 安装基础组件 proot-distro
   
    ```bash
    pkg install proot-distro 
    ```

    查看 proot-distro 的使用帮助为

    ```bash
    proot-distro help 
    ```

2. 查看可安装的Linux系统

    ```bash
    proot-distro list
    ```

3. 使用对应命令安装系统

    ```bash
    proot-distro install <alias> 

    # 比如要安装 Ubuntu
    proot-distro install ubuntu
    ```

4. 安装完成后，进入 Linux发行版环境

    ```bash
    proot-distro login ubuntu
    ```

5. 退出

    ```bash
    exit
    ```

#### 安装图形界面

6. 下载并安装 AnLinux，在仪表板上选择你想要安装的Linux发行版，然后选择复制指令，然后在Termux的终端窗口，粘贴这段指令等待安装即可，安装完成后使用下面命令进入完整版 linux 系统

    ```bash
    ./start-ubuntu.sh
    ```

7. 进入完整版linux系统，后为解决不能访问手机自身存储文件系统的问题，使用 termux 自带的 nano 修改 start-ubuntu.sh 文件，取消注释即可

    ```bash
    #command+=“ -b /sdcard”
    ```

8. 安装桌面，在 AnLinux 中，左上角选择桌面，第一步，选择对应的Linux版本，第二步选择建议的 Xfce4 桌面，然后复制提示的指令，在 termux 中进入安装的系统后，粘贴并运行指令。

9. 运行完成后，在安装 termux 中进入安装的 Linux 系统后，开启 VNC 服务。第一次开启时还需要设置密码。
    
    ```bash
    vncserver-start
    ```
10. 开启后基础主机名和端口号，比如:localhost:1，开启VNC Viewer，然后设置，点击 Connect 即可连接

11. 使用完毕后，记得关闭 VNC

    ```bash
    vncserver-stop
    ```

#### Linux 安装后续
```bash
apt update && apt upgrade

apt install gcc
apt install g++
apt install gfortran
apt install cmake

# 安装 java
apt search openjdk
apt install default-jdk

# 安装 Python
apt install python3


```

创建一个用户newuser，并交互式的设置密码

```bash
adduser newuser
```

设置sudo权限

为用户添加sudo权限，可以使用修改sudoers和adduser两种方法，推荐使用第二种： etc/sudoers文件就是与sudo组有关的文件，在里面添加一行

```bash
adduser newuser sudo
```
