*@DATE: 2023/04/21 17:06:25*

---

## OpenSSH Server

### 使用 PowerShell 安装 OpenSSH

若要使用 PowerShell 安装 OpenSSH，请先以管理员身份运行 PowerShell。 为了确保 OpenSSH 可用，请运行以下 cmdlet

```PowerShell
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'
```

根据需要安装服务器或客户端组件：

```PowerShell
# Install the OpenSSH Client
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0

# Install the OpenSSH Server
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

将默认 shell 设置为 powershell.exe

```PowerShell
New-ItemProperty -Path "HKLM:\SOFTWARE\OpenSSH" -Name DefaultShell -Value "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" -PropertyType String -Force
```

### 启动并配置 OpenSSH 服务器

若要启动并配置 OpenSSH 服务器来开启使用，请以管理员身份打开 PowerShell，然后运行以下命令来启动 sshd service：

```PowerShell
# 开启 SSHD 服务
Start-Service sshd

# 设置服务的自动启动(OPTIONAL but recommended):
Set-Service -Name sshd -StartupType 'Automatic'

# Confirm the Firewall rule is configured. It should be created automatically by setup. Run the following to verify
if (!(Get-NetFirewallRule -Name "OpenSSH-Server-In-TCP" -ErrorAction SilentlyContinue | Select-Object Name, Enabled)) {
    Write-Output "Firewall Rule 'OpenSSH-Server-In-TCP' does not exist, creating it..."
    New-NetFirewallRule -Name 'OpenSSH-Server-In-TCP' -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
} else {
    Write-Output "Firewall rule 'OpenSSH-Server-In-TCP' has been created and exists."
}
```

### 查看本地 SSH 服务

输入如下命令后回车会要求输入密码，密码是本机密码

```PowerShell
ssh localhost
```

---

## OpenSSH Client

### 连接到 OpenSSH 服务器

安装后，可从使用 PowerShell 安装了 OpenSSH 客户端的 Windows 10 或 Windows Server 2019 设备连接到 OpenSSH 服务器，如下所示。 请务必以管理员身份运行 PowerShell：

```PowerShell
ssh username@servername
```

连接后，会收到如下所示的消息：

```PowerShell
The authenticity of host 'servername (10.00.00.001)' can't be established.
ECDSA key fingerprint is SHA256:(<a large string>).
Are you sure you want to continue connecting (yes/no)?
```

选择“是”后，该服务器会添加到包含 Windows 客户端上的已知 SSH 主机的列表中。

系统此时会提示你输入密码。 作为安全预防措施，密码在键入的过程中不会显示。

连接后，你将看到 Windows 命令行界面提示符：

```PowerShell
domain\username@SERVERNAME C:\Users\username>
```
