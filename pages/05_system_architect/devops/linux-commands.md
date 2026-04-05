# Linux常用命令 - 架构师学习笔记

## Linux常用命令

Linux作为服务器操作系统，掌握其常用命令是每个架构师的基本技能。这些命令可以帮助我们进行系统管理、故障排查、性能监控等工作。

熟练掌握Linux命令不仅能提高工作效率，还能帮助我们快速定位和解决系统问题。

## 系统信息命令

### uname - 系统信息

```bash
# 查看系统信息
uname -a

# 查看内核版本
uname -r

# 查看系统架构
uname -m
```

显示系统基本信息，包括内核版本、硬件架构等。

### top/htop - 进程监控

```bash
# 查看系统进程和资源使用情况
top

# 更友好的进程监控工具
htop
```

实时监控系统进程和资源使用情况，包括CPU、内存等。

### df - 磁盘空间

```bash
# 查看磁盘空间使用情况
df -h

# 查看inode使用情况
df -i
```

查看文件系统磁盘空间使用情况，帮助判断是否需要清理空间。

## 文件操作命令

### ls - 列出文件

```bash
# 列出当前目录文件
ls -la

# 按时间排序
ls -lt

# 递归列出子目录
ls -R
```

列出目录中的文件和子目录，支持多种显示选项。

### find - 查找文件

```bash
# 按文件名查找
find /path -name "filename"

# 按修改时间查找
find /path -mtime -7

# 按文件大小查找
find /path -size +100M
```

在指定目录中查找文件，支持按名称、时间、大小等条件查找。

### grep - 文本搜索

```bash
# 在文件中搜索文本
grep "pattern" file.txt

# 递归搜索目录
grep -r "pattern" /path

# 忽略大小写
grep -i "pattern" file.txt

# 显示行号
grep -n "pattern" file.txt
```

在文件中搜索指定的文本模式，是日志分析的重要工具。

## 网络诊断命令

### netstat/ss - 网络连接

```bash
# 查看网络连接状态
netstat -tuln

# 查看监听端口
ss -tuln

# 查看进程占用端口
netstat -tulnp
```

查看网络连接状态、监听端口以及进程占用端口情况。

### ping/curl - 网络连通性

```bash
# 测试网络连通性
ping google.com

# 发送HTTP请求
curl -I https://www.example.com

# 测试端口连通性
telnet hostname port
```

测试网络连通性和服务可用性。

### tcpdump - 网络抓包

```bash
# 抓取指定接口的数据包
tcpdump -i eth0

# 抓取指定主机的数据包
tcpdump host hostname

# 抓取HTTP数据包
tcpdump -s 0 -A 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)>2)) != 0)'
```

捕获和分析网络数据包，用于网络故障排查。

## 系统性能监控

### vmstat - 虚拟内存统计

```bash
# 查看系统整体性能
vmstat 1 5

# 查看磁盘I/O统计
vmstat -d
```

报告虚拟内存统计信息，包括进程、内存、分页、块IO、系统和CPU活动。

### iostat - I/O统计

```bash
# 查看CPU和I/O统计
iostat -x 1 5

# 查看指定设备的统计信息
iostat -p sda
```

监控系统设备的I/O负载，帮助识别I/O瓶颈。

### sar - 系统活动报告

```bash
# 查看CPU使用率
sar -u

# 查看内存使用情况
sar -r

# 查看网络统计
sar -n DEV
```

收集、报告和保存系统活动信息，是性能分析的重要工具。

## Linux命令使用技巧

- 使用管道(|)组合命令，实现复杂功能
- 使用重定向(>, >>, <)进行输入输出控制
- 使用通配符(*)匹配多个文件
- 使用别名(alias)简化常用命令
- 使用history查看命令历史
