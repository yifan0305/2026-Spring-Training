# Linux 速查手册

------

## 目录

- [基础入门](#基础入门)
- [文件与目录](#文件与目录)
- [链接](#链接)
- [用户与权限](#用户与权限)
- [查找文件](#查找文件)
- [软件仓库](#软件仓库)
- [文本操作](#文本操作)
- [重定向与管道](#重定向与管道)
- [进程管理](#进程管理)
- [文件压缩](#文件压缩)
- [网络](#网络)
- [Vim 编辑器](#vim-编辑器)

------

## 基础入门

### 命令行解读

```bash
[root@iZm5e8dsxce9ufaic7hi3uZ ~]# pwd
/root
```

| 部分                      | 说明                                                         |
| ------------------------- | ------------------------------------------------------------ |
| `root`                    | 当前用户名                                                   |
| `iZm5e8dsxce9ufaic7hi3uZ` | 主机名                                                       |
| `~`                       | 当前目录（家目录）；root 的家目录为 `/root`，普通用户在 `/home/用户名` |
| `#` / `$`                 | `#` 表示 root 权限，`$` 表示普通用户                         |

```bash
whoami      # 查看当前用户名
hostname    # 查看当前主机名
```

### 键盘快捷键

| 快捷键             | 功能                |
| ------------------ | ------------------- |
| `↑` / `↓`          | 调取历史命令        |
| `Tab`              | 命令/路径补全       |
| `Ctrl + R`         | 搜索历史命令        |
| `Ctrl + C`         | 中止当前命令        |
| `Ctrl + L`         | 清屏                |
| `Ctrl + A` / `E`   | 光标跳到行首 / 行尾 |
| `Ctrl + U` / `K`   | 剪切到行首 / 行尾   |
| `Ctrl + W`         | 剪切左侧一个单词    |
| `Ctrl + Y`         | 粘贴剪切内容        |
| `Ctrl + D`         | 关闭 Shell 会话     |
| `Ctrl + Shift + =` | 放大终端字体        |
| `Ctrl + -`         | 缩小终端字体        |

> `history` 列出所有历史命令；`!编号`（如 `!2`）直接执行对应命令。

------

## 文件与目录

```bash
ls              # 列出当前目录文件
ls -a           # 显示隐藏文件
ls -l           # 详细信息
ls -t           # 按修改时间排序

pwd             # 显示当前路径

cd /path        # 切换目录
cd ~            # 回到家目录
cd ..           # 返回上级目录
cd -            # 返回上一次目录

mkdir dir           # 创建目录
mkdir -p a/b/c      # 递归创建

touch file.txt      # 创建文件 / 更新时间戳

cp file1 file2      # 复制文件
cp -r dir1 dir2     # 复制目录

mv file1 file2      # 移动 / 重命名

rm file             # 删除文件
rm -r dir           # 删除目录
rm -rf dir          # 强制递归删除（慎用）

cat file            # 查看文件内容
less file           # 分页查看（q 退出）
head -n 5 file      # 查看前 5 行
tail -n 5 file      # 查看后 5 行
tail -f file        # 实时追踪文件末尾（常用于日志）
```

------

## 链接

Linux 文件由三部分组成：**文件名**、**文件内容**、**权限**，文件名通过 inode 绑定到内容。

### 硬链接

两个文件共享同一 inode（同一内容），删除任意一方不影响内容，直到两者都被删除。
 **不支持目录链接。**

```bash
ln file1 file2      # 创建 file2 为 file1 的硬链接
```

### 软链接（符号链接）

类似 Windows 快捷方式，有独立 inode，指向源文件路径。
 删除源文件后，软链接变为**死链接**。

```bash
ln -s file1 file2   # 创建软链接

ls -l
# lrwxrwxrwx 1 root root 5 Jan 14 06:42 file2 -> file1
```

------

## 用户与权限

### 用户管理

```bash
sudo command            # 以 root 身份执行命令
useradd lion            # 添加用户（家目录在 /home/lion）
passwd lion             # 修改密码
userdel lion            # 删除用户（保留 /home 目录）
userdel lion -r         # 同时删除 /home 目录
su lion                 # 切换到 lion 用户
sudo su                 # 切换为 root
```

### 群组管理

```bash
groupadd friends                    # 创建群组
groupdel foo                        # 删除群组
groups lion                         # 查看用户所在群组

usermod -g friends lion             # 修改主群组
usermod -G friends,foo lion         # 添加到多个群组
usermod -a -G bar lion              # 追加群组（不离开原群组）
usermod -l newname lion             # 重命名用户（家目录需手动改）

chgrp bar file.txt                  # 修改文件群组
chown lion file.txt                 # 修改文件所有者
chown lion:bar file.txt             # 同时修改所有者和群组
chown -R lion:lion /home/frank      # 递归修改
```

### 文件权限

```
drwxr-xr-x
│└──┬──┘└──┬──┘└──┬──┘
│  所有者   群组   其他用户
└─ d=目录  -=文件  l=链接
```

权限字符：`r`=读(4)，`w`=写(2)，`x`=执行(1)，`-`=无权限(0)

```bash
chmod 740 file.txt
# 7(rwx) 4(r--) 0(---)  →  所有者读写执行，群组只读，其他无权限

chmod -R 777 /home/lion     # 递归修改

# 字母方式
chmod u+rx file             # 所有者增加读和执行
chmod g+r,o-r file          # 群组加读，其他去读
chmod u=rwx,g=r,o=- file    # 精确分配
```

------

## 查找文件

### locate — 数据库查找（快）

```bash
locate file.txt         # 查找包含关键字的文件
locate fil*.txt         # 支持通配符
updatedb                # 更新数据库（新建文件需先执行）
```

### find — 实时遍历硬盘（精准）

```bash
# 按名称
find / -name "syslog"
find /var/log -name "syslog*"

# 按大小
find /var -size +10M        # 大于 10M
find /var -size -50k        # 小于 50k

# 按访问时间
find -name "*.txt" -atime -7    # 近 7 天访问过

# 按类型
find . -name "file" -type f     # 仅文件
find . -name "file" -type d     # 仅目录

# 对结果操作
find -name "*.jpg" -delete                  # 批量删除
find -name "*.c" -exec chmod 600 {} \;      # 批量执行命令
find -name "*.c" -ok chmod 600 {} \;        # 带确认提示
```

------

## 软件仓库

```bash
yum update              # 更新所有软件包
yum search xxx          # 搜索软件包
yum install xxx         # 安装
yum remove xxx          # 卸载
```

**切换阿里云源（CentOS 7）：**

```bash
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
yum makecache
```

**查看手册：**

```bash
man ls              # 查看命令手册
man 3 rand          # 查看第 3 节（库函数）
ls --help           # 快速帮助
```

------

## 文本操作

```bash
# grep — 搜索关键字
grep path /etc/profile          # 查找含 path 的行
grep -i path /etc/profile       # 忽略大小写
grep -n path /etc/profile       # 显示行号
grep -v path /etc/profile       # 显示不含关键字的行
grep -r hello /etc              # 递归查找目录
grep -E ^path /etc/profile      # 正则：以 path 开头

# sort — 排序
sort name.txt                   # 默认字典序
sort -r name.txt                # 倒序
sort -n nums.txt                # 数字排序
sort -R name.txt                # 随机排序
sort -o sorted.txt name.txt     # 输出到文件

# wc — 统计
wc name.txt             # 行数 单词数 字节数
wc -l name.txt          # 只统计行数
wc -w name.txt          # 只统计单词数
wc -c name.txt          # 只统计字节数
wc -m name.txt          # 只统计字符数

# uniq — 去重（仅处理连续重复行）
uniq name.txt               # 去重输出
uniq -c name.txt            # 统计重复次数
uniq -d name.txt            # 只显示重复行

# cut — 剪切
cut -c 2-4 name.txt                 # 提取每行第 2-4 个字符
cut -d , -f 1 notes.csv             # 按逗号分隔，取第 1 列
```

------

## 重定向与管道

```
键盘 → stdin(0) → 命令 → stdout(1) → 终端
                       → stderr(2) → 终端
```

| 符号    | 说明                               |
| ------- | ---------------------------------- |
| `>`     | 标准输出重定向（覆盖）             |
| `>>`    | 标准输出重定向（追加）             |
| `2>`    | 标准错误重定向                     |
| `2>>`   | 标准错误追加                       |
| `2>&1`  | 错误合并到标准输出                 |
| `<`     | 指定命令输入来源                   |
| `<<END` | 键盘输入直到 END 结束              |
| `|`     | 管道：前一命令输出作为后一命令输入 |

```bash
cut -d , -f 1 notes.csv > name.csv          # 提取第一列写入文件
cat not_exist.csv > res.txt 2> errors.log   # 分别重定向输出和错误
cat not_exist.csv >> res.txt 2>&1           # 合并追加

# 管道组合
cut -d , -f 1 name.csv | sort > sorted.txt
du | sort -nr | head
grep log -Ir /var/log | cut -d : -f 1 | sort | uniq
```

------

## 进程管理

### 查看进程

```bash
w                       # 查看登录用户及其操作
ps                      # 当前终端进程快照
ps -ef                  # 所有进程
ps -aux                 # 含 CPU/内存信息
top                     # 动态进程列表（q 退出）
```

### 控制进程

```bash
kill 956                # 结束进程
kill -9 956             # 强制结束
```

### 前后台切换

```bash
command &               # 启动后台进程
nohup command &         # 忽略挂断信号运行于后台
Ctrl + Z                # 暂停当前前台进程
bg %1                   # 将暂停的进程转为后台运行
fg %1                   # 将后台进程转为前台
jobs                    # 查看后台进程列表
```

> **进程状态码**：`R` 运行 · `S` 休眠 · `D` 不可中断 · `Z` 僵死 · `T` 停止

### systemd 守护进程

```bash
systemctl start nginx       # 启动服务
systemctl stop nginx        # 停止
systemctl restart nginx     # 重启
systemctl status nginx      # 查看状态
systemctl enable nginx      # 开机自启
systemctl disable nginx     # 取消自启
```

------

## 文件压缩

```bash
# tar 归档
tar -cvf archive.tar file1 file2    # 创建归档
tar -tf archive.tar                  # 查看内容（不解压）
tar -xvf archive.tar                 # 解压
tar -rvf archive.tar file.txt        # 追加文件

# tar + gzip（推荐）
tar -czvf archive.tar.gz dir/       # 压缩
tar -xzvf archive.tar.gz            # 解压

# tar + bzip2
tar -cjvf archive.tar.bz2 dir/
tar -xjvf archive.tar.bz2

# zip / unzip
zip -r sort.zip sort/
unzip archive.zip
unzip -l archive.zip                # 只查看不解压

# 查看压缩文件内容（不解压）
zcat file.tar.gz
zless file.tar.gz
```

------

## 网络

```bash
ifconfig                        # 查看网络接口信息
host github.com                 # 域名解析
host 13.229.188.59              # IP 反查主机名

# SSH 连接
ssh root@172.20.10.1            # 默认 22 端口

# 免密登录
ssh-keygen                      # 生成密钥对
ssh-copy-id root@172.x.x.x     # 上传公钥到服务器

# SSH 别名配置（~/.ssh/config）
Host lion
    HostName 172.x.x.x
    Port     22
    User     root
# 配置后直接：ssh lion

# 文件下载
wget http://example.com/file.zip
wget -c http://...              # 断点续传

# 文件传输
scp file.txt root@192.x.x.x:/path/         # 本地 → 远程
scp root@192.x.x.x:/path/file.txt .        # 远程 → 本地

# 增量同步
rsync -arv src/ dst/                        # 本地同步
rsync -arv src/ root@192.x.x.x:dst/        # 同步到远程
rsync -arv --delete src/ dst/              # 同步并删除目标多余文件
```

------

## Vim 编辑器

### 模式切换

```
正常模式（默认）
   ↓ i/a/o              ↑ Esc
插入模式

正常模式 → : → 命令模式
```

### 打开与退出

```
vim file.txt        打开文件
:q                  退出（未修改）
:wq  或  :x         保存并退出
:q!                 不保存强制退出
```

### 光标移动

```
h / j / k / l       左 / 下 / 上 / 右
w                   跳到下一个单词
0 / $               行首 / 行尾
gg / G              文件首 / 文件尾
数字 + gg           跳到第 N 行
```

### 编辑操作

```
i / a / o           进入插入模式（光标前 / 后 / 新行）
x                   删除当前字符
dd                  删除当前行
dw                  删除当前单词
d0 / d$             删除到行首 / 行尾
yy                  复制当前行
yw                  复制当前单词
p                   粘贴
r + 字符            替换当前字符
u                   撤销
Ctrl + R            重做
```

### 查找与替换

```
/关键字             向后查找，n 下一个，N 上一个
?关键字             向前查找

:s/old/new          替换当前行第一个
:s/old/new/g        替换当前行所有
:2,4s/old/new/g     替换第 2-4 行所有
:%s/old/new/g       全文替换
```

### 其他功能

```
:set nu / nonu      显示 / 隐藏行号
:r filename         在光标处插入另一文件内容
:!ls                执行外部命令
:sp file            水平分屏
:vsp file           垂直分屏
Ctrl+W + 方向键     切换分屏视口
```

### 可视模式

```
v                           字符选择
V                           行选择
Ctrl + v                    块选择（列编辑）
Ctrl + v → 选中 → I → 输入 → Esc Esc    多行同时插入
d                           删除选中
u / U                       转小写 / 大写
```

------

> **永久配置 Vim**：编辑 `~/.vimrc`，常用配置如 `set nu`（显示行号）、`syntax on`（语法高亮）。