---
title: Windows Docker Image with Cygwin
date: 2019-09-02
categories: Docker
---

我们通常会用Ansible做一下自动化配置的工作，Ansible是基于SSH协议进行通信的，当要统一为Linux和Windows的机器提供支持的时候，也需要Windows的主机能支持SSH。Ansible支持通过WinRM协议对Windows主机进行管理，详细配置可以参考这里：[https://docs.ansible.com/ansible/latest/user_guide/windows_winrm.html](https://docs.ansible.com/ansible/latest/user_guide/windows_winrm.html)。本文重点在如何通过Cygwin在Windows上支持SSH，并构建相应的Docker镜像。

构建该Windows Docker镜像的步骤:

1. 安装Cygwin；
2. 在Windows上创建对应的账户并加入Administrator用户组；
3. 创建对应的用户文件夹和.ssh文件夹，将authorized_keys拷贝到.ssh文件夹并设置对应的权限；
4. 将Cygwin用户和Windows用户关联起来，并设置相应的权限；
5. 在entrypoint文件中启动配置Cgywin的程序和sshd服务；

### sshd_config配置文件

```
Port 2222
AuthorizedKeysFile %h/.ssh/authorized_keys
PasswordAuthentication yes
RsaAuthentication yes
StrictModes no
Subsystem       sftp    /usr/sbin/sftp-server
```

* `Port`指sshd服务的端口，这里用2222代替常用的22端口；
* `AuthorizedKeysFile`指无密码访问时候authorized_keys文件存放的位置；
* `PasswordAuthentication`指是否允许通过密码登录，这里设置成允许；
* `RsaAuthentication`指无密码登录时采用的认证方式，这个设置是必须的；
* `StrictModes`设置为no，防止`ssh permission bad ownership issue`；

### Powershell安装脚本

#### install.ps1

```powershell
Write-Host "Create root user accounts..."
$pw = ConvertTo-SecureString -String $env:PASSWORD  -AsPlainText -Force
New-LocalUser -Name root -Password $pw -PasswordNeverExpires -AccountNeverExpires
Add-LocalGroupMember -Group Administrators -Member root

Write-Host "Install Cygwin..."
Invoke-WebRequest https://cygwin.com/setup-x86_64.exe -OutFile C:\setup-x86_64.exe
Start-Process C:\setup-x86_64.exe -Wait -NoNewWindow -ArgumentList "-q -n -l C:\cygwin64\packages -s http://mirrors.kernel.org/sourceware/cygwin/ -R C:\cygwin64 -P python-devel,openssh,cygrunsrv,wget,tar,qawk,bzip2,subversion,vim,make,gcc-fortran,gcc-g++,gcc-core,make,openssl,openssl-devel,libffi-devel,libyaml-devel,zip,unzip,gdb,libsasl2,gettext,git"
Remove-Item C:\setup-x86_64.exe

Write-Host "Create home folder, copy authorized_keys and set permissions..."
New-Item C:\cygwin64\home\root -type directory -force
New-Item C:\cygwin64\home\root\.ssh -type directory -force
Copy-Item C:\authorized_keys -Destination C:\cygwin64\home\root\.ssh
C:\cygwin64\bin\chown -R root /cygdrive/c/cygwin64/home/root
C:\cygwin64\bin\chmod 700 /cygdrive/c/cygwin64/home/root/.ssh
C:\cygwin64\bin\chmod 640 /cygdrive/c/cygwin64/home/root/.ssh/authorized_keys

Write-Host "Map Cygwin user with Windows User..."
C:\cygwin64\bin\mkpasswd -l -u root > /cygwin64/etc/passwd
C:\cygwin64\bin\mkgroup --local > /cygwin64/etc/group

Write-Host "Set related permissions..."
C:\cygwin64\bin\chmod +r /etc/passwd
C:\cygwin64\bin\chmod u+w /etc/passwd
C:\cygwin64\bin\chmod +r /etc/group
C:\cygwin64\bin\chmod u+w /etc/group
C:\cygwin64\bin\chmod 755 /var
C:\cygwin64\bin\touch /var/log/sshd.log
C:\cygwin64\bin\chmod 664 /var/log/sshd.log

Write-Host "Set path..."
$newPath = 'C:\cygwin64\bin;\cygwin\bin;' + [Environment]::GetEnvironmentVariable("PATH", [EnvironmentVariableTarget]::Machine)
[Environment]::SetEnvironmentVariable("PATH", $newPath, [EnvironmentVariableTarget]::Machine)
```

### Docker启动配置脚本

#### Entrypoint.ps1

```powershell
Start-Process C:\cygwin64\bin\bash.exe -Wait -NoNewWindow -ArgumentList "C:\cygwin64\bin\ssh-host-config --yes -c '' -u root -w $env:PASSWORD"

$FileName = "C:\cygwin64\etc\sshd_config"
if (Test-Path $FileName) {
  Remove-Item $FileName
}
Move-Item C:\sshd_config C:\cygwin64\etc\sshd_config -Force

Start-Process powershell -Verb runAs -ArgumentList "net start sshd"

Get-Content -Path "C:\License.txt" -Wait | Out-Null
```

### Dockerfile

```dockerfile
FROM microsoft/windowsservercore:latest

ARG PASSWORD
ENV PASSWORD ${PASSWORD:-root_password}
COPY install.ps1 C:\\install.ps1
COPY sshd_config C:\\sshd_config
COPY entrypoint.ps1 C:\\entrypoint.ps1
COPY authorized_keys C:\\authorized_keys

RUN powershell C:\\install.ps1; \
    powershell -Command "Remove-Item C:\\install.ps1 -Force"

EXPOSE 2222/tcp 5985/tcp

ENTRYPOINT ["powershell.exe", "C:\\entrypoint.ps1"]
```

### References

* [https://github.com/MicrosoftDocs/Virtualization-Documentation/issues/358](https://github.com/MicrosoftDocs/Virtualization-Documentation/issues/358)
* [https://superuser.com/questions/1041500/simplest-way-to-add-multiple-users-in-cygwin](https://superuser.com/questions/1041500/simplest-way-to-add-multiple-users-in-cygwin)
* [https://www.daveperrett.com/articles/2010/09/14/ssh-authentication-refused](https://www.daveperrett.com/articles/2010/09/14/ssh-authentication-refused)
* [https://cygwin.com/ml/cygwin/2016-03/msg00097.html](https://cygwin.com/ml/cygwin/2016-03/msg00097.html)
* [https://solidlinux.wordpress.com/2010/06/28/configure-passwordless-ssh-login-in-linuxcygwin](https://solidlinux.wordpress.com/2010/06/28/configure-passwordless-ssh-login-in-linuxcygwin)

