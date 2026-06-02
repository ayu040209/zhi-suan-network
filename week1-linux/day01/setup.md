# Day 1 - Python 环境安装记录

## 系统环境
- OS: CentOS 7.7.1908 (Core)
- 虚拟机: VMware Workstation 17.5
- 网络模式: NAT
- 用户名: ayu (普通用户) / root

## 安装过程

### 1. 检查 Python3
```bash
python3 --version
结果: bash: python3: command not found...
说明系统未预装 Python3。
CentOS 7 默认官方源无法连接，报错 Could not resolve host: mirrorlist.centos.org
执行
sudo mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
sudo curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
sudo yum clean all && sudo yum makecache
结果: Metadata Cache Created，阿里云源换源成功。
sudo yum install -y python3 python3-pip
结果: Complete!，安装成功。
python3 --version   # Python 3.6.8
pip3 --version      # pip 9.0.3
python3 -c "print('Hello World')"
结果: 输出 Hello World，环境正常。

| 问题          | 原因                          | 解决                   |
| :---------- | :-------------------------- | :------------------- |
| yum 无法连接    | CentOS 官方源停止维护/网络不通         | 更换阿里云镜像源             |
| git push 失败 | GitHub 需要 Token 认证，且虚拟机网络受限 | 改为在 Windows 宿主机网页端上传 |
