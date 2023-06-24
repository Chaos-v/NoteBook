# Android 调试桥 (adb)

Android 调试桥 (adb) 是一种功能多样的命令行工具，可让您与设备进行通信。adb 命令可用于执行各种设备操作（例如安装和调试应用），并提供对 Unix shell（可用来在设备上运行各种命令）的访问权限。它是一种客户端-服务器程序，包括以下三个组件：

**客户端**：用于发送命令。客户端在开发机器上运行。您可以通过发出 adb 命令从命令行终端调用客户端。
**守护程序 (adbd)**：用于在设备上运行命令。守护程序在每个设备上作为后台进程运行。
**服务器**：用于管理客户端与守护程序之间的通信。服务器在开发机器上作为后台进程运行。

## adb 的工作原理

当您启动某个 adb 客户端时，该客户端会先检查是否有 adb 服务器进程正在运行。如果没有，它会启动服务器进程。服务器在启动后会与本地 TCP 端口 5037 绑定，并监听 adb 客户端发出的命令 - 所有 adb 客户端均通过端口 5037 与 adb 服务器通信。

然后，服务器会与所有正在运行的设备建立连接。它通过扫描 5555 到 5585 之间（该范围供前 16 个模拟器使用）的奇数号端口查找模拟器。服务器一旦发现 adb 守护程序 (adbd)，便会与相应的端口建立连接。请注意，每个模拟器都使用一对按顺序排列的端口 - 用于控制台连接的偶数号端口和用于 adb 连接的奇数号端口。例如：

> 模拟器 1，控制台：5554
> 模拟器 1，adb：5555
> 模拟器 2，控制台：5556
> 模拟器 2，adb：5557
> 依此类推

如上所示，在端口 5555 处与 adb 连接的模拟器与控制台监听端口为 5554 的模拟器是同一个。

服务器与所有设备均建立连接后，您便可以使用 adb 命令访问这些设备。由于服务器管理与设备的连接，并处理来自多个 adb 客户端的命令，因此您可以从任意客户端（或从某个脚本）控制任意设备。

## 在设备上启用 adb 调试

在通过 USB 连接的设备上使用 adb，必须在设备的系统设置开发者选项下启用 USB 调试。

## 通过 Wi-Fi 连接到设备（Android 10 及更低版本）

一般情况下，adb 通过 USB 与设备进行通信，也可以通过 Wi-Fi 使用 adb。如要连接到搭载 Android 10 或更低版本的设备，必须通过 USB 执行一些初始步骤，如下所述：

1、将 Android 设备和 adb 主机连接到这两者都可以访问的同一 Wi-Fi 网络。请注意，并非所有接入点都适用；您可能需要使用防火墙已正确配置为支持 adb 的接入点。

2、如果您要连接到 Wear OS 设备，请关闭手机上与该设备配对的蓝牙。

3、使用 USB 线将设备连接到主机。

4、设置目标设备以监听端口 5555 上的 TCP/IP 连接。

```
    adb tcpip 5555
```

5、拔掉连接目标设备的 USB 线。

6、找到 Android 设备的 IP 地址。例如，对于 Nexus 设备，您可以在设置 > 关于平板电脑（或关于手机）> 状态 > IP 地址下找到 IP 地址。或者，对于 Wear OS 设备，您可以在设置 > WLAN 设置 > 高级 > IP 地址下找到 IP 地址。

7、通过 IP 地址连接到设备。

```
    adb connect device_ip_address:5555
```

8、确认主机已连接到目标设备：

```
    $ adb devices
    List of devices attached
    device_ip_address:5555 device
```

9、如果 adb 连接断开。确保主机仍与 Android 设备连接到同一个 Wi-Fi 网络。通过再次执行 ```adb connect``` 步骤重新连接。如果上述操作未解决问题，使用如下命令重置 adb 主机：

```
    adb kill-server
```

然后，从头开始操作。

## 查询设备

在发出 adb 命令之前，为了解哪些设备实例已连接到 adb 服务器，可以使用 ```devices``` 命令生成已连接设备的列表。

```
    adb devices -l
```

作为回应，adb 会针对每个设备输出以下状态信息：

&emsp; ● **序列号**：由 adb 创建的字符串，用于通过端口号唯一标识设备。 下面是一个序列号示例：```emulator-5554```

&emsp; ● **状态**：设备的连接状态可以是以下几项之一：

&emsp; &emsp; ● ```offline```：设备未连接到 adb 或没有响应。

&emsp; &emsp; ● ```device```：设备现已连接到 adb 服务器。请注意，此状态并不表示 Android 系统已完全启动并可正常运行，因为在设备连接到 adb 时系统仍在启动。不过，在启动后，这将是设备的正常运行状态。

&emsp; &emsp; ● ```no device```：未连接任何设备。

&emsp; ● **说明**：如果您包含 ```-l``` 选项，```devices``` 命令会告知您设备是什么。当您连接了多个设备时，此信息很有用，可帮助您将它们区分开来。

以下示例展示了 ```devices``` 命令及其输出。有三个设备正在运行。列表中的前两行表示模拟器，第三行表示连接到计算机的硬件设备。

```
    $ adb devices
    List of devices attached
    emulator-5556 device product:sdk_google_phone_x86_64 model:Android_SDK_built_for_x86_64 device:generic_x86_64
    emulator-5554 device product:sdk_google_phone_x86 model:Android_SDK_built_for_x86 device:generic_x86
    0a388e93      device usb:1-1 product:razor model:Nexus_7 device:flo
```

### 模拟器未列出
```adb devices``` 命令的极端命令序列会导致正在运行的模拟器不显示在 ```adb devices``` 输出中（即使在您的桌面上可以看到相应模拟器）。当满足以下所有条件时，就会发生这种情况：

> &emsp; 1、adb 服务器未在运行，
> &emsp; 2、您在使用 ```emulator``` 命令时，将 ```-port``` 或 ```-ports``` 选项的端口值设为 5554 到 5584 之间的奇数，
> &emsp; 3、您选择的奇数号端口处于空闲状态，因此可以与指定端口号的端口建立连接，或者该端口处于忙碌状态，模拟器切换到了符合第 2 条中要求的另一个端口，以及
> &emsp; 4、启动模拟器后才启动 adb 服务器。

避免出现这种情况的一种方法是让模拟器自行选择端口，并且每次运行的模拟器数量不要超过 16 个。另一种方法是始终先启动 adb 服务器，然后再使用 ```emulator``` 命令，如下例所示。

**示例 1**：在下面的命令序列中，```adb devices``` 命令启动了 adb 服务器，但是设备列表未显示。

停止 adb 服务器，然后按照所示顺序输入以下命令。对于 avd 名称，请提供系统中有效的 avd 名称。如需获取 avd 名称列表，请输入 ```emulator -list-avds```。 ```emulator``` 命令位于 ```*android_sdk/tools*``` 目录下。

```
    $ adb kill-server
    $ emulator -avd Nexus_6_API_25 -port 5555
    $ adb devices

    List of devices attached
    * daemon not running. starting it now on port 5037 *
    * daemon started successfully *
```

**示例 2**：在下面的命令序列中，```adb devices``` 显示了设备列表，因为先启动了 adb 服务器。

如果想在 ```adb devices``` 输出中看到模拟器，请停止 adb 服务器，然后在使用 ```emulator``` 命令之后、使用 ```adb devices``` 命令之前，重新启动该服务器，如下所示：

```
    $ adb kill-server
    $ emulator -avd Nexus_6_API_25 -port 5557
    $ adb start-server
    $ adb devices

    List of devices attached
    emulator-5557 device
```

## 将命令发送至特定设备

如果有多个设备在运行，发出 adb 命令时必须指定目标设备。

使用 **devices** 命令获取目标设备的序列号。获得序列号后，结合使用 **-s** 选项与 adb 命令来指定序列号。

如果要发出很多 adb 命令，可以将 ```\$ANDROID_SERIAL``` 环境变量设为包含序列号。如果您同时使用 ```-s``` 和 ```\$ANDROID_SERIAL```，```-s``` 会替换 ```\$ANDROID_SERIAL```。

如果有多个可用设备，但只有一个是模拟器，请使用 ```-e``` 选项将命令发送至该模拟器。同样，如果有多个设备，但只连接了一个硬件设备，请使用 ```-d``` 选项将命令发送至该硬件设备。

## 安装应用

使用 adb 的 ```install``` 命令在模拟器或连接的设备上安装 APK

```
    adb install path_to_apk
```

安装测试 APK 时，必须在 ```install``` 命令中使用 ```-t``` 选项。

## 设置端口转发

使用 ```forward``` 命令设置任意端口转发，将特定主机端口上的请求转发到设备上的其他端口。以下示例设置了主机端口 6100 到设备端口 7100 的转发：

```
adb forward tcp:6100 tcp:7100
```

## 将文件复制到设备/从设备复制文件

使用 ```pull``` 和 ```push``` 命令将文件复制到设备或从设备复制文件。与 ```install``` 命令（仅将 APK 文件复制到特定位置）不同，使用 ```pull``` 和 ```push``` 命令可将任意目录和文件复制到设备中的任何位置。

```
    # 需从设备中复制某个文件或目录（及其子目录），请使用以下命令：
    adb pull remote local

    # 需将某个文件或目录（及其子目录）复制到设备，请使用以下命令：
    adb push local remote
```

## 停止 adb 服务器

停止 adb 服务器，使用 ```adb kill-server``` 命令。然后，您可以通过发出其他任何 adb 命令来重启服务器。

## 发出 shell 命令

使用 ```shell``` 命令通过 adb 发出设备命令，也可以启动交互式 ```shell```。如需发出单个命令，请使用 shell 命令，如下所示：

```
    adb [-d |-e | -s serial_number] shell shell_command
```

在设备上启动交互式 shell，请使用 shell 命令，如下所示：

```
    adb [-d | -e | -s serial_number] shell
```

退出交互式 shell，请按 Ctrl + D 键或输入 ```exit```

Android 提供了大多数常见的 Unix 命令行工具。如需查看可用工具的列表，请使用以下命令：

```
    adb shell ls /system/bin
```

对于大多数命令，都可通过 ```--help``` 参数获得命令帮助。

### 调用 activity 管理器 (```am```)

在 adb shell 中，您可以使用 activity 管理器 (```am```) 工具发出命令以执行各种系统操作，如启动 activity、强行停止进程、广播 intent、修改设备屏幕属性，等等。在 shell 中，相应的语法为：

```
    am command
```

您也可以直接从 adb 发出 activity 管理器命令，无需进入远程 shell。例如：

```
    adb shell am start -a android.intent.action.VIEW
```

**表 2.** 可用的 activity 管理器命令

| 命令	| 说明
| ----------- | ----------- |
| ```start [options] intent```	| 启动由 ```intent``` 指定的 Activity。<br> 请参阅 intent 参数的规范。<br>具体选项包括：<br>● ```-D```：启用调试功能。<br>● ```-W```：等待启动完成。<br>● ```--start-profiler file```：启动性能分析器并将结果发送至 file。<br>● ```-P file```：类似于 ```--start-profiler```，但当应用进入空闲状态时剖析停止。<br>● ```-R count```：重复启动 activity ```count``` 次。在每次重复前，将完成顶层 activity。<br>● ```-S```：在启动 activity 前，强行停止目标应用。<br>● ```--opengl-trace```：启用 OpenGL 函数的跟踪。<br>● ```--user user_id | current```：指定要作为哪个用户运行；如果未指定，则作为当前用户运行。<br> |
| ```startservice [options] intent```	| 启动由 ```intent``` 指定的 Service。<br>请参阅 intent 参数的规范。<br>具体选项包括：<br>```--user user_id | current```：指定要作为哪个用户运行；如果未指定，则作为当前用户运行。 |
| ```force-stop package```	| 强行停止与 ```package```（应用的软件包名称）关联的所有进程。 |
| ```kill [options] package```	| 终止与 ```package```（应用的软件包名称）关联的所有进程。此命令仅终止可安全终止且不会影响用户体验的进程。<br>具体选项包括：<br>```--user user_id | all \| current```：指定要终止哪个用户的进程；如果未指定，则终止所有用户的进程。<br>| 
| ```kill-all```	| 终止所有后台进程。| 
| ```broadcast [options] intent```	| 发出广播 intent。<br>请参阅 intent 参数的规范。<br>具体选项包括：<br>```[--user user_id \| all | current]```：指定要发送给哪个用户；如果未指定，则发送给所有用户。<br> | 
| instrument [options] ```component```	|使用 Instrumentation 实例启动监控。通常情况下，目标 component 采用 ```test_package/runner_class``` 格式。<br>具体选项包括：<br>```-r```：输出原始结果（否则，对 ```report_key_streamresult``` 进行解码）。与 ```[-e perf true]``` 结合使用可生成性能测量的原始输出。<br>```-e name value```：将参数 name 设为 value。 对于测试运行程序，通用格式为 ```-e testrunner_flag value[,value...]```。<br>```-p file```：将剖析数据写入 file。<br>```-w```：等待插桩完成后再返回。测试运行程序需要使用此选项。<br>```--no-window-animation```：运行时关闭窗口动画。<br>```--user user_id | current```：指定以哪个用户身份运行插桩；如果未指定，则以当前用户身份运行。<br> |
| ```profile start process file```	| 启动 process 的性能分析器，将结果写入 file。|
| ```profile stop process```	| 停止 process 的性能分析器。|
| ```dumpheap [options] process file```	| 转储 process 的堆，写入 file。<br>具体选项包括：<br>```--user [user_id | current]```：提供进程名称时，指定要转储的进程的用户；如果未指定，则使用当前用户。<br>-n：转储原生堆，而非托管堆。<br> |
| ```set-debug-app [options] package```	| 设置要调试的应用 package。<br>具体选项包括：<br> -w：应用启动时等待调试程序。<br> --persistent：保留此值。<br> |
| ```clear-debug-app```	| 清除之前使用 set-debug-app 设置的待调试软件包。|
|```monitor [options]```	|开始监控崩溃或 ANR。<br>具体选项包括：<br>```--gdb```：在崩溃/ANR 时，在给定的端口上启动 gdbserv。
|```screen-compat {on \| off} package```	|控制 package 的屏幕兼容性模式。|
| ```display-size [reset \| widthxheight```]	|替换设备显示尺寸。此命令支持使用大屏设备模仿小屏幕分辨率（反之亦然），对于在不同尺寸的屏幕上测试应用非常有用。<br>示例：<br>```am display-size 1280x800```|
| ```display-density dpi```	| 替换设备显示密度。此命令支持使用低密度屏幕在高密度屏幕环境上进行测试（反之亦然），对于在不同密度的屏幕上测试应用非常有用。<br>示例：<br>```am display-density 480```| 
|```to-uri intent```	|以 URI 的形式输出给定的 intent 规范。<br>请参阅 intent 参数的规范。|
|```to-intent-uri intent```	|以 intent: URI 的形式输出给定的 intent 规范。<br>请参阅 intent 参数的规范。|

#### intent 参数的规范

对于采用 intent 参数的 activity 管理器命令，您可以使用以下选项指定 intent：

> ```-a action``` 
> &emsp;指定 intent 操作，如 android.intent.action.VIEW。只能声明一次。
> ```-d data_uri```
> &emsp;指定 intent 数据 URI，如 content://contacts/people/1。只能声明一次。
> ```-t mime_type```
> &emsp;指定 intent MIME 类型，如 image/png。只能声明一次。
> ```-c category```
> &emsp;指定 intent 类别，如 android.intent.category.APP_CONTACTS。
> ```-n component```
> &emsp;指定带有软件包名称前缀的组件名称以创建显式 intent，如 com.example.app/.ExampleActivity。
> ```-f flags```
> &emsp;向 setFlags() 支持的 intent 添加标记。
> ```--esn extra_key```
> &emsp;添加一个空 extra。URI intent 不支持此选项。
> ```-e | --es extra_key extra_string_value```
> &emsp;以键值对的形式添加字符串数据。
> ```--ez extra_key extra_boolean_value```
> &emsp;以键值对的形式添加布尔值数据。
> ```--ei extra_key extra_int_value```
> &emsp;以键值对的形式添加整数型数据。
> ```--el extra_key extra_long_value```
> &emsp;以键值对的形式添加长整型数据。
> ```--ef extra_key extra_float_value```
> &emsp;以键值对的形式添加浮点型数据。
> ```--eu extra_key extra_uri_value```
> &emsp;以键值对的形式添加 URI 数据。
> ```--ecn extra_key extra_component_name_value```
> &emsp;添加组件名称，该名称作为 ComponentName 对象进行转换和传递。
> ```--eia extra_key extra_int_value[,extra_int_value...]```
> &emsp;添加整数数组。
> ```--ela extra_key extra_long_value[,extra_long_value...]```
> &emsp;添加长整数数组。
> ```--efa extra_key extra_float_value[,extra_float_value...]```
> &emsp;添加浮点数数组。
> ```--grant-read-uri-permission```
> &emsp;添加 FLAG_GRANT_READ_URI_PERMISSION 标记。
> ```--grant-write-uri-permission```
> &emsp;添加 FLAG_GRANT_WRITE_URI_PERMISSION 标记。
> ```--debug-log-resolution```
> &emsp;添加 FLAG_DEBUG_LOG_RESOLUTION 标记。
> ```--exclude-stopped-packages```
> &emsp;添加 FLAG_EXCLUDE_STOPPED_PACKAGES 标记。
> ```--include-stopped-packages```
> &emsp;添加 FLAG_INCLUDE_STOPPED_PACKAGES 标记。
> ```--activity-brought-to-front```
> &emsp;添加 FLAG_ACTIVITY_BROUGHT_TO_FRONT 标记。
> ```--activity-clear-top```
> &emsp;添加 FLAG_ACTIVITY_CLEAR_TOP 标记。
> ```--activity-clear-when-task-reset```
> &emsp;添加 FLAG_ACTIVITY_CLEAR_WHEN_TASK_RESET 标记。
> ```--activity-exclude-from-recents```
> &emsp;添加 FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS 标记。
> ```--activity-launched-from-history```
> &emsp;添加 FLAG_ACTIVITY_LAUNCHED_FROM_HISTORY 标记。
> ```--activity-multiple-task```
> &emsp;添加 FLAG_ACTIVITY_MULTIPLE_TASK 标记。
> ```--activity-no-animation```
> &emsp;添加 FLAG_ACTIVITY_NO_ANIMATION 标记。
> ```--activity-no-history```
> &emsp;添加 FLAG_ACTIVITY_NO_HISTORY 标记。
> ```--activity-no-user-action```
> &emsp;添加 FLAG_ACTIVITY_NO_USER_ACTION 标记。
> ```--activity-previous-is-top```
> &emsp;添加 FLAG_ACTIVITY_PREVIOUS_IS_TOP 标记。
> ```--activity-reorder-to-front```
> &emsp;添加 FLAG_ACTIVITY_REORDER_TO_FRONT 标记。
> ```--activity-reset-task-if-needed```
> &emsp;添加 FLAG_ACTIVITY_RESET_TASK_IF_NEEDED 标记。
> ```--activity-single-top```
> &emsp;添加 FLAG_ACTIVITY_SINGLE_TOP 标记。
> ```--activity-clear-task```
> &emsp;添加 FLAG_ACTIVITY_CLEAR_TASK 标记。
> ```--activity-task-on-home```
> &emsp;添加 FLAG_ACTIVITY_TASK_ON_HOME 标记。
> ```--receiver-registered-only```
> &emsp;添加 FLAG_RECEIVER_REGISTERED_ONLY 标记。
> ```--receiver-replace-pending```
> &emsp;添加 FLAG_RECEIVER_REPLACE_PENDING 标记。
> ```--selector```
> &emsp;需要使用 -d 和 -t 选项设置 intent 数据和类型。
> ```URI component package```
> &emsp;如果不受上述任一选项的限制，您可以直接指定 URI、软件包名称和组件名称。当某个参数不受限制时，如果该参数包含“:”（英文冒号），那么该工具会假定参数是 URI；如果该参数包含“/”（正斜线），那么该工具会假定参数是组件名称；如果并非上述两种情况，那么该工具会假定参数是软件包名称。

### 调用软件包管理器 (pm)

在 adb shell 中，您可以使用软件包管理器 (pm) 工具发出命令，以对设备上安装的应用软件包执行操作和查询。在 shell 中，相应的语法为：

```
    pm command
```

您也可以直接从 adb 发出软件包管理器命令，无需进入远程 shell。例如：

```
    adb shell pm uninstall com.example.MyApp
```

**表 3**. 可用的软件包管理器命令。

|命令	|说明|
|---------|---------|
|```list packages [options] filter```	|输出所有软件包，或者，仅输出软件包名称包含 filter 中的文本的软件包。<br>具体选项：<br>● ```-f```：查看它们的关联文件。<br>● ```-d```：进行过滤以仅显示已停用的软件包。<br>● ```-e```：进行过滤以仅显示已启用的软件包。<br>● ```-s```：进行过滤以仅显示系统软件包。<br>● ```-3```：进行过滤以仅显示第三方软件包。<br>● ```-i```：查看软件包的安装程序。<br>● ```-u```：也包括已卸载的软件包。<br>● ```--user user_id```：要查询的用户空间。|
|```list permission-groups```	|输出所有已知的权限组。|
|```list permissions [options] group```	|输出所有已知的权限，或者，仅输出 group 中的权限。<br>具体选项：<br>● ```-g```：按组进行整理。<br>● ```-f```：输出所有信息。<br>● ```-s```：简短摘要。<br>● ```-d```：仅列出危险权限。<br>● ```-u```：仅列出用户将看到的权限。<br>
|```list instrumentation [options]```	|列出所有测试软件包。<br>具体选项：<br>```-f```：列出测试软件包的 APK 文件。<br>```target_package```：仅列出此应用的测试软件包。|
|```list features```	|输出系统的所有功能。|
|```list libraries```	|输出当前设备支持的所有库。|
|```list users```	|输出系统中的所有用户。|
|```path package```	|输出给定 package 的 APK 的路径。
|```install [options] path```	|将软件包（通过 path 指定）安装到系统。<br>具体选项：<br>● ```-r```：重新安装现有应用，并保留其数据。<br>● ```-t```：允许安装测试 APK。仅当您运行或调试了应用或者使用了 Android Studio 的 Build > Build APK 命令时，Gradle 才会生成测试 APK。如果是使用开发者预览版 SDK（如果 targetSdkVersion 是字母，而非数字）构建的 APK，那么安装测试 APK 时必须在 install 命令中包含 -t 选项。<br>● ```-i installer_package_name```：指定安装程序软件包名称。<br>● ```--install-location location```：使用以下某个值设置安装位置：<br>&emsp; ● ```0```：使用默认安装位置。<br>&emsp; ● ```1```：在内部设备存储上安装。<br>&emsp; ● ```2```：在外部介质上安装。<br>● ```-f```：在内部系统内存上安装软件包。<br>● ```-d```：允许版本代码降级。<br>● ```-g```：授予应用清单中列出的所有权限。<br>● ```--fastdeploy```：通过仅更新已更改的 APK 部分来快速更新安装的软件包。<br>● ```--incremental```：仅安装 APK 中启动应用所需的部分，同时在后台流式传输剩余数据。如要使用此功能，您必须为 APK 签名，创建一个 APK 签名方案 v4 文件，并将此文件放在 APK 所在的目录中。只有部分设备支持此功能。此选项会强制 adb 使用该功能，如果该功能不受支持，则会失败（并提供有关失败原因的详细信息）。附加 --wait 选项，可等到 APK 完全安装完毕后再授予对 APK 的访问权限。<br>● ```--no-incremental``` 可阻止 adb 使用此功能。|
|```uninstall [options] package```	|从系统中移除软件包。<br>具体选项：<br>```-k```：移除软件包后保留数据和缓存目录。|
|```clear package```	|删除与软件包关联的所有数据。|
|```enable package_or_component```	|启用给定的软件包或组件（写为“package/class”）。|
|```disable package_or_component```	|停用给定的软件包或组件（写为“package/class”）。|
|```disable-user [options] package_or_component```	|具体选项：<br>```--user user_id```：要停用的用户。|
|```grant package_name permission```	|向应用授予权限。在搭载 Android 6.0（API 级别 23）及更高版本的设备上，该权限可以是应用清单中声明的任何权限。在搭载 Android 5.1（API 级别 22）及更低版本的设备上，该权限必须是应用定义的可选权限。|
|```revoke package_name permission```	|从应用撤消权限。在搭载 Android 6.0（API 级别 23）及更高版本的设备上，该权限可以是应用清单中声明的任何权限。在搭载 Android 5.1（API 级别 22）及更低版本的设备上，该权限必须是应用定义的可选权限。|
|```set-install-location location```	|更改默认安装位置。位置值如下：<br>```0```：自动：让系统决定最合适的位置。<br>```1```：内部：在内部设备存储上安装。<br>```2```：外部：在外部介质上安装。<br> <br>注意：此命令仅用于调试目的；使用此命令可能会导致应用中断和其他意外行为。|
|```get-install-location```	|返回当前安装位置。返回值如下：<br>```0 [auto]```：让系统决定最合适的位置<br>```1 [internal]```：在内部设备存储上安装<br>```2 [external]```：在外部介质上安装 | 
|```set-permission-enforced permission [true | false]```	|指定是否应强制执行指定权限。|
|```trim-caches desired_free_space```	|减少缓存文件以达到给定的可用空间。|
|```create-user user_name```	|创建具有给定 ```user_name``` 的新用户，从而输出该用户的新用户标识符。|
|```remove-user user_id```	|移除具有给定 ```user_id``` 的用户，从而删除与该用户关联的所有数据。|
|```get-max-users```	|输出设备支持的最大用户数。|
|```get-app-links [options] [package]	```|输出给定 package 的域名验证状态，如果未指定软件包，则输出所有软件包的域名验证状态。状态代码的定义如下：<br>```none```：没有为此域名记录任何内容<br>```verified```：域名已成功通过验证<br>```approved```：强行批准了域名，通常是通过执行 shell 命令来实现的<br>```denied```：强行拒绝了域名，通常是通过执行 shell 命令来实现的<br>```migrated```：从旧响应流程中保留的验证状态<br>```restored```：从用户数据恢复流程中保留的验证状态<br>```legacy_failure```：旧版验证程序拒绝了域名，原因未知<br>```system_configured```：设备配置自动批准了域名<br>```>= 1024```：设备验证程序特定的自定义错误代码<br>选项如下：<br>```--user user_id```：包括用户选择的域名（涵盖所有域名，而不仅仅是执行 autoVerify 的域名）。<br> |
|```reset-app-links [options] [package]	```|重置给定软件包的域名验证状态，如果未指定任何软件包，则重置所有软件包的域名验证状态。<br>package：要重置的软件包，如果使用“all”，则重置所有软件包<br>选项如下：<br>```--user user_id```：包括用户选择的域名（涵盖所有域名，而不仅仅是执行 autoVerify 的域名）。<br>|
|```verify-app-links [--re-verify] [package]	```|广播给定 package 的域名验证请求，如果未指定软件包，则发送所有软件包的域名验证请求。仅当软件包之前未记录响应时发送该请求。<br>```--re-verify```：即使软件包已记录响应也发送|
|```set-app-links [--package package] state domains	```|手动设置软件包的域名状态。仅当软件包将域名声明为 autoVerify 时，此命令才能正常运行。此命令不会针对无法应用的域名报告失败。<br>● ```--package package```：要设置的软件包，如果使用“all”，则设置所有软件包<br>● ```state```：要为域名设置的代码，有效值为：<br>&emsp; ● ```STATE_NO_RESPONSE (0)```：按未记录过任何响应的情况进行重置。<br>&emsp; ● ```STATE_SUCCESS (1)```：将域名视为已成功通过域名验证代理的验证。请注意，域名验证代理可以覆盖此设置。<br>&emsp; ● ```STATE_APPROVED (2)```：将域名视为一律批准，防止域名验证代理更改状态。<br>&emsp; ● ```STATE_DENIED (3)```：将域名视为一律拒绝，防止域名验证代理更改状态。<br>&emsp; ● ```domains```：要更改的域名的列表（以空格分隔），如果使用“all”，则更改所有域名。|
|```set-app-links-user-selection --user user_id [--package package] enabled domains	```|手动设置主机用户针对软件包选择的域名的状态。仅当软件包声明相应域名时，此命令才能正常运行。此命令不会针对无法应用的域名报告失败。<br>```--user user_id```：要为其更改域名选择的用户<br>```--package package/code>: the package to set<``` <br>```enabled: whether or not to approve the domain```<br>```domains: space separated list of domains to change, or "all" to change every domain.```|
|```set-app-links-user-selection --user user_id [--package package] enabled domains```	|手动设置主机用户针对软件包选择的域名的状态。仅当软件包声明相应域名时，此命令才能正常运行。此命令不会针对无法应用的域名报告失败。<br>```--user user_id```：要为其更改域名选择的用户<br>```--package package```：要设置的软件包<br>```enabled```：是否批准域名<br>```domains```：要更改的域名的列表（以空格分隔），如果使用“all”，则更改所有域名。|
|```set-app-links-allowed --user user_id [--package package] allowed```	|切换软件包的自动验证链接处理设置。<br>```--user user_id```：要为其更改域名选择的用户<br>```--package package```：要设置的软件包，如果使用“all”，则设置所有软件包；如果未指定任何软件包，则重置软件包。<br>```allowed```：值为 true 时，表示允许软件包打开自动验证链接；值为 false 时，表示不允许这么做|
|```get-app-link-owners --user user_id [--package package] domains```	|为给定用户输出特定域名的所有者（按优先级从低到高的顺序排列）。<br>```--user user_id```：要为其执行查询的用户<br>```--package package```：（可选）同时针对软件包声明的所有域名输出结果；如果使用“```all```”，则针对所有软件包声明的所有域名输出结果<br>```domains```：要对其执行查询的域名列表（以空格分隔）|

### 调用设备政策管理器 (```dpm```)

为便于您开发和测试设备管理应用（或其他企业应用），您可以向设备政策管理器 (```dpm```) 工具发出命令。使用该工具可控制活动管理应用，或更改设备上的政策状态数据。在 shell 中，语法如下：

```
    dpm command
```

您也可以直接从 adb 发出设备政策管理器命令，无需进入远程 shell：

```
    adb shell dpm command
```

**表 4**. 可用的设备政策管理器命令

|命令	|说明|
|---------|---------|
|```set-active-admin [options] component```	|将 component 设为活动管理。<br>具体选项包括：<br>● ```--user user_id```：指定目标用户。您也可以传递 ````--user current```` 以选择当前用户。|
|```set-profile-owner [options] component```	|将 component 设为活动管理，并将其软件包设为现有用户的个人资料所有者。<br>具体选项包括：<br>● ```--user user_id```：指定目标用户。您也可以传递 ```--user current``` 以选择当前用户。<br>● ```--name name```：指定简单易懂的组织名称。|
|```set-device-owner [options] component```	|将 component 设为活动管理，并将其软件包设为设备所有者。<br>具体选项包括：<br>● ```--user user_id```：指定目标用户。您也可以传递 ```--user current``` 以选择当前用户。<br>● ```--name name```：指定简单易懂的组织名称。|
|```remove-active-admin [options] component```	|停用活动管理。应用必须在清单中声明 android:testOnly。此命令还会移除设备所有者和个人资料所有者。<br>具体选项包括：<br>```--user user_id```：指定目标用户。您也可以传递``` --user current``` 以选择当前用户。|
|```clear-freeze-period-record```	|清除设备之前设置的系统 OTA 更新冻结期记录。在开发管理冻结期的应用时，这有助于避免设备存在调度方面的限制。请参阅管理系统更新。<br>在搭载 Android 9.0（API 级别 28）及更高版本的设备上受支持。|
|```force-network-logs```	|强制系统让任何现有网络日志随时可供 DPC 检索。如果有可用的连接或 DNS 日志，DPC 会收到 ```onNetworkLogsAvailable()``` 回调。请参阅网络活动日志。<br>此命令有调用频率限制。在搭载 Android 9.0（API 级别 28）及更高版本的设备上受支持。|
|```force-security-logs```	|强制系统向 DPC 提供任何现有安全日志。如果有可用的日志，DPC 会收到 ```onSecurityLogsAvailable()``` 回调。请参阅记录企业设备活动。<br>此命令有调用频率限制。在搭载 Android 9.0（API 级别 28）及更高版本的设备上受支持。|

### 截取屏幕截图

```screencap``` 命令是一个用于对设备显示屏截取屏幕截图的 shell 实用程序。在 shell 中，语法如下：

```
    screencap filename
```

如需从命令行使用 screencap，请输入以下命令：

```
    adb shell screencap /sdcard/screen.png
```

以下屏幕截图会话示例展示了如何使用 adb shell 截取屏幕截图，以及如何使用 pull 命令从设备下载屏幕截图文件：

```
    $ adb shell
    shell@ $ screencap /sdcard/screen.png
    shell@ $ exit
    $ adb pull /sdcard/screen.png
```

### 录制视频

```screenrecord``` 命令是一个用于录制设备（搭载 Android 4.4（API 级别 19）及更高版本）显示屏的 shell 实用程序。该实用程序将屏幕 activity 录制为 MPEG-4 文件。您可以使用此文件创建宣传视频或培训视频，或将其用于调试或测试。

在 shell 中，使用以下语法：

```
    screenrecord [options] filename
```

如需从命令行使用 ```screenrecord```，请输入以下命令：

```
    adb shell screenrecord /sdcard/demo.mp4
```

按 Ctrl + C 键（在 Mac 上，按 Command + C 键）可停止屏幕录制；如果不手动停止，到三分钟或 ```--time-limit``` 设置的时间限制时，录制将会自动停止。

如需开始录制设备屏幕，请运行 ``screenrecord`` 命令以录制视频。然后，运行 ```pull``` 命令以将视频从设备下载到主机。下面是一个录制会话示例：

```
    $ adb shell
    shell@ $ screenrecord --verbose /sdcard/demo.mp4
    (press Control + C to stop)
    shell@ $ exit
    $ adb pull /sdcard/demo.mp4
```

```screenrecord``` 实用程序能以您要求的任何支持的分辨率和比特率进行录制，同时保持设备显示屏的宽高比。默认情况下，该实用程序以本机显示分辨率和屏幕方向进行录制，时长不超过三分钟。

```screenrecord``` 实用程序的局限性：

> &emsp; ● 音频不与视频文件一起录制。
> &emsp; ● 无法在搭载 Wear OS 的设备上录制视频。
> &emsp; ● 某些设备可能无法以它们的本机显示分辨率进行录制。如果在录制屏幕时出现问题，请尝试使用较低的屏幕分辨率。
> &emsp; ● 不支持在录制时旋转屏幕。如果在录制期间屏幕发生了旋转，则部分屏幕内容在录制时将被切断。

表 5. ```screenrecord``` 选项

|选项	|说明|
|---------|---------|
|```--help```	|显示命令语法和选项|
|```--size widthxheight```	|设置视频大小：1280x720。默认值为设备的本机显示屏分辨率（如果支持）；如果不支持，则为 1280x720。为获得最佳效果，请使用设备的 Advanced Video Coding (AVC) 编码器支持的大小。|
|```--bit-rate rate```	|设置视频的视频比特率（以 MB/秒为单位）。默认值为 4Mbps。您可以增加比特率以提升视频品质，但这样做会导致视频文件变大。下面的示例将录制比特率设为 6Mbps：<br>```screenrecord --bit-rate 6000000 /sdcard/demo.mp4```|
|```--time-limit time```	|设置最大录制时长（以秒为单位）。默认值和最大值均为 180（3 分钟）。|
|```--rotate```	|将输出旋转 90 度。此功能处于实验阶段。|
|```--verbose```	|在命令行屏幕显示日志信息。如果您不设置此选项，则该实用程序在运行时不会显示任何信息。|

### 读取应用的 ART 配置文件

从 Android 7.0（API 级别 24）开始，Android Runtime (ART) 会收集已安装应用的执行配置文件，这些配置文件用于优化应用性能。您可能需要检查收集的配置文件，以了解在应用启动期间，系统频繁执行了哪些方法和使用了哪些类。

要生成文本格式的配置文件信息，请使用以下命令：

```
adb shell cmd package dump-profiles package
```

要检索生成的文件，请使用：

```
adb pull /data/misc/profman/package.txt
```

### 重置测试设备

如果您在多个测试设备上测试应用，则在两次测试之间重置设备可能很有用，例如，可以移除用户数据并重置测试环境。您可以使用 ```testharness``` adb shell 命令对搭载 Android 10（API 级别 29）或更高版本的测试设备执行恢复出厂设置，如下所示。

```
adb shell cmd testharness enable
```

使用 ```testharness``` 恢复设备时，设备会自动将允许通过当前工作站调试设备的 RSA 密钥备份在一个持久性位置。也就是说，在重置设备后，工作站可以继续调试设备并向设备发出 adb 命令，而无需手动注册新密钥。

此外，为了帮助您更轻松且更安全地继续测试您的应用，使用 ```testharness``` 恢复设备还会更改以下设备设置：

> ● 设备会设置某些系统设置，以便不会出现初始设备设置向导。也就是说，设备会进入一种状态，供您快速安装、调试和测试您的应用。
> ● 设置：
> &emsp; ● 停用锁定屏幕
> &emsp; ● 停用紧急提醒
> &emsp; ● 停用帐户自动同步
> &emsp; ● 停用自动系统更新
> ● 其他：
> &emsp; ● 停用预装的安全应用

如果您的应用需要检测并适应 ```testharness``` 命令的默认设置，您可以使用 ```ActivityManager.isRunningInUserTestHarness()```。

### sqlite

```sqlite3``` 可启动用于检查 sqlite 数据库的 sqlite 命令行程序。它包含用于输出表格内容的 ```.dump``` 以及用于输出现有表格的 ```SQL CREATE``` 语句的 ```.schema``` 等命令。您也可以从命令行执行 SQLite 命令，如下所示。

```
    $ adb -s emulator-5554 shell
    $ sqlite3 /data/data/com.example.app/databases/rssitems.db
    SQLite version 3.3.12
    Enter ".help" for instructions
```