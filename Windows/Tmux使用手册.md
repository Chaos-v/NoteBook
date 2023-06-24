*@DATE: 2023/06/01 12:40:12*

---

## 简介

Tmux 是一个终端复用器（terminal multiplexer），允许在单个窗口中，同时访问多个会话。Tmux中每个程序都有自己的终端。

tmux的主要用途是：

- 通过在tmux中运行远程服务器上运行的程序，保护它们免受连接中断的影响。

- 允许从多台不同的本地计算机访问在远程服务器上运行的程序。

- 在一个终端中同时使用多个程序和shell，类似窗口管理器。

---

## 基本用法

### 安装

```bash
# Ubuntu/Debian 
sudo apt-get install tmux

# termux
apt install tmux
```

### 启动与退出

安装完成后，键入 tmux 命令即可进入 Tmux 窗口。按下 `Ctrl+d` 或者显式输入 `exit` 命令，就可以退出 Tmux 窗口。

```bash
# 启动命令
tmux

# 退出
exit
```

### 新建会话

启动 tmux 后，第一个启动的 Tmux 窗口，编号是`0`，第二个窗口的编号是`1`，以此类推。这些窗口对应的会话，就是 0 号会话、1 号会话。

使用编号区分会话，不太直观，更好的方法是为会话起名。

```bash
# 新建指定命名的会话
tmux new -s <session-name>
```

### 分离会话

在 Tmux 窗口中，按下`Ctrl+b d`或者输入`tmux detach`命令，就会将当前会话与窗口分离。

```bash
tmux detach
```

上面命令执行后，就会退出当前 Tmux 窗口，但是会话和里面的进程仍然在后台运行。

`tmux ls`命令可以查看当前所有的 Tmux 会话。

```bash
tmux ls
# or
tmux list-session
```

### 接入会话

`tmux attach`命令用于重新接入某个已存在的会话。

```bash
# 使用会话编号
tmux attach -t 0
 
# 使用会话名称
tmux attach -t <session-name>
```

### 杀死会话

`tmux kill-session`命令用于杀死某个会话。

```bash
# 方式1，使用会话编号
tmux kill-session -t 0

# 方式2，使用会话名称
tmux kill-session -t <session-name>
```

### 切换会话

`tmux switch`命令用于切换会话。

```bash
# 使用会话编号
tmux switch -t 0

# 使用会话名称
tmux switch -t <session-name>
```

### 重命名会话

`tmux rename-session`命令用于重命名会话。

```bash
tmux rename-session -t 0 <new-name>
```

上面命令将0号会话重命名。

### 其他命令

```bash
# 列出所有快捷键，及其对应的 Tmux 命令
tmux list-keys

# 列出所有 Tmux 命令及其参数
tmux list-commands

# 列出当前所有 Tmux 会话的信息
tmux info

# 重新加载当前的 Tmux 配置
tmux source-file ~/.tmux.conf
```

其他命令根据需要补充
