<!--
 * @Author: wolf-li
 * @Date: 2024-10-20 21:34:27
 * @LastEditTime: 2024-10-22 17:41:16
 * @LastEditors: wolf-li
 * @Description: 
 * @FilePath: /note/src/Mind/LinuxMind.md
 * talk is cheep show me your code.
-->
# Linux 系统管理

## Linux 发行版

- Debin 分支
- fedora  分支 （开源版 redhat 2003年停止维护）
- arch 分支 （包含 GNU 非自由的模块）未首到 GNU 计划的官方支持
- andriod (linux 发行版，不是 GNU/Linux) 不使用 GNU的一系列工具和库，大幅修改了 Linux 内核以精简运行时开销，适配移动设备 ASOP 项目 只是用了 GPL 许可的 Linux kernel 而在 Kernel 之上的 ART (Android Runtime)、Bionic C 库、驱动透明化的 HAL (Hardware Abstraction Layer) 则作为用户态存在，避免 Android 系统框架、Google 移动应用服务框架（GMS）和各厂商的驱动程序被 GPL 感染而开源。
- 构成
  - linux kernel
  - GNU Utilities
  - Addition Software
  - Package manager

- SHELL
  - 常用快捷键
  - 通配符
  - 元字符
  - 特殊字符
    - 单引号  所见即所得，强引用，单引号内容原样输出
    - 双引号  解析特殊字符
    - 反引号 常用于引用命令结果等同于 $()
  - 命令
    - 历史命令 history
    - 命令查找
      - 查找命令的绝对路径 which
      - 查找命令的程序、源代码等相关文件 whereis 
  
## 文件管理

- 概念
  - 在 Linux 或 Unix 操作系统中，所有的文件和目录都被组织成以一根节点开始的倒置的树状结构。
  - 文件系统的最顶层是由根目录开始的，系统使用 / 来表示根目录。在目录之下的既可以是目录，也可以是文件，而每一个目录中又可以包含子录文件。如此反复就可以构成一个庞大的文件系统。
  - 特殊目录  ./  当前目录  ../ 上级目录 隐藏目录或文件 开头为 .
  - 文件系统作用：描述和组织系统的存储资源
  - 文件系统构成
    - 名字空间：为对象命名并按照层次化方式对其进行组织的方法
    - API：一组用于导航并处理对象的系统调用
    - 安全模型：用于保护、隐藏和共享对象的方案
    - 实现：将逻辑模型与硬件联系在一起
- 目录结构
  - 系统启动必须
    - /boot  存放启动 linux 时使用的核心文件，包括连接文件和镜文件
    - /sys  sysfs文件系统集成了下面3种文件系统的信息：①针对进程息的proc文  系统②针对设备的devfs文件系统③针对伪终端的devpts件系统。该文件系统是内核  备树的一个直观反映。当一个内核对象被建的时候，对应的文件和目录也在内核对象  系统中
    - /etc. etcetera 等等的缩写，该目录下存放的是所有系统管理所要的配置文  和子目录
    - /lib  /lib64  存放基本代码库（比如c++库），其作用类似Windows里的DL  文件。几乎所有的应用程序都需要用到这些共享库。
  - 命令
    - /bin  bin是 binary 二进制缩写，目录存放常用命令
    - /sbin 只有系统管理可以使用的命令
  - 外部文件管理
    - /dev  存放 linux 的外部设备
    - /media   类windows的其他设备，例如U盘、光驱等等，识别linux会把设备放到这个目录下。
    - /mnt  临时挂载别的文件系统的，我们可以将光驱挂载在/mnt/上然后进入该目录就可以查看光驱里的内容了。
  - 临时文件
    - /run  系统启动以来的信息，系统重启时，这个目录下的文件应该清除 /varrun指向这个目录
    - /tmp  用来存放一些临时文件
    - /lost+found  一般情况为空，系统异常关机后，存放文件
  - 账户
    - /usr 用户的很多应用程序和文件都在这个目录
    - /home  用户的主目录，每个普通用户都有一个目录名是以自己的账名命名的
    - /usr/bin  系统用户使用的应用程序与命令
    - /usr/sbin  超级用户使用的比较高级的管理程序和守护程序
    - /usr/src 内核源代码
    - /root  超级用户的家目录
  - 运行过程使用
    - /var 存放经常修改的数据
    - /proc  管理内存空间，虚拟目录，系统内核内存数据的映射
  - 运行过程用
    - /opt  默认是空的，安装额外的软件可以放在这里
    - /srv  存放服务启动后需要拉取的数据
  - 重要的文件
    - /etc/hostname. 主机名
    - /etc/resolve.conf  DNS服务配置
    - /etc/hosts 手动域名
    - /etc/*-release 操作系统信息
- 文件属性
  - 文件权限位（ rwx rwx rwx ）
    文件类型  所属用户权限 所属组权限 其他用户权限
    数字表示 r-4 w-2 x-1  - 0 
    \- 没有权限
    s 特殊功能
  - setuid、setgid
    - 八进制 4000 和 2000 的位分别对应 setuid 和 setgid 位，如果可执行文件设置了这两位，程序便可以访问原本没有权限访问的文件和进程
  - 文件默认权限。644
  - 目录默认权限   755
  - 文件类型
    - ◆- 普通文件
    - ◆d 目录文件
    - ◆b 块设备
    - ◆c 字符设备
    - ◆l 符号链接文件 （软连接）rm 删除
    - ◆p 管道文件pipe 进程间通信使用，也称为 FIFO（先进先出） rm 删除
    - ◆s 套接字文件socket， 进程之间的连接，允许其安全、妥善的进行通信。 rm 删除
  - 文件的3种时间
    - atime：Access Time 最后一次访问文件（读取或执行）或目录的间；
    - mtime：Modofy Time 最后一次修改文件内容（数据）或目录内（目录内文      - 件列表）的时间；
    - ctime：Change Time 最后一次改变文件属性（元数据）或目录属（元数据）的时间；
    - 查看方式
      - atime: ls -lu filename
      - mcime: ls -l filename
      - ctime: ls -lc filename
  - 访问控制列表（ACL
    - 控制文件访问相比于传统9位长度的访问系统更强大，更复杂的方法
    - 类型
      - POSIX版 20世纪90年代中期制定的规范（getfacl， setfack）
      - NFSv4 ACL 适配 linux 与 windows
- 文件元数据
  - indoe 
    inode是“index node”的缩写，中文译名为索引节点或i节点。它是个数据结构，用于跟踪Linux或基于UNIX的文件系统中的所有文件和录。每个文件或目录在文件系统中都会被分配一个唯一的inode号码，且该inode号码在文件的整个生命周期内都是不变的。
  - 作用 存储文件的元数据信息：inode记录了文件的许多重要属性，如件的大小、拥有者、权限、创建时间、修改时间和访问时间等，以及文件链接数和磁盘块的指针等。这些元数据使得文件系统能够快速地检索和管文件。
  - 提供文件系统的性能优化：由于inode中记录了文件的元数据信息，系可以通过读取inode来获取文件的属性，而无需读取整个文件。这样可以著提高文件系统的性能，特别是对于大量小文件的读取和管理。
  - 实现硬链接：inode中的链接数属性可以用来记录有多少个文件名指向一个inode。硬链接是指在文件系统中创建一个新的文件名，该文件名与始文件名指向同一个inode，共享相同的数据块。这样可以节省存储空间并且对于不同的文件名可以使用不同的权限和属性。
  - 管理文件的数据块：inode中还包含了指向存储文件实际数据的数据块指针。通过这些指针，操作系统可以快速定位文件的数据块并进行读取或入操作。
  - inode 限制与问题
    - inode耗尽问题：如果文件系统中的inode数量耗尽，即使磁盘上还可用的存储空间，也无法再创建新的文件或目录。因为每个新文件或目都需要一个唯一的inode来标识。
    - inode数量监控：定期检查inode使用情况非常重要，以确保其不会到配置的限制。可以使用上述的df -i命令来监控inode的使用情况。
  - 命令
    - 查看 inode 
      ls -i
      stat 文件名
  - 检查 inode 使用情况
    - df -i
    - find -inum 可以根据 inode 编号查找文件
- 常见命令
  - 目录
    - 创建目录 mkdir
    - 删除目录 rmdir
    - 查看某个目录下文件信息 ls
    - 切换目录 cd
    - 显示当前目录 pwd
    - 树状显示目录的内容 tree （默认没有这个命令）
  - 文件查找
    - 制定目录下查找文件 find
    - 查找文件或目录 locate （默认没有）比 find -name 快，   体搜索目录，而是搜索一个数据库 /var/lib/locatedb
  - 文件内容添加
    - xargs -n 3 < filename
    - cat >> filename << EOF
      context
      EOF
    - echo "test" > filename
  - 查看返回路径中文件名字 basename
  - 文件创建
    - touch
    - echo "" > file1
  - 文件创建连接
    - 软连接 ln -s
    - 硬连接 ln 
  - 文件复制
    - cp
    - rsync
  - 文件移动
      mv
  - 文件删除
      rm
  - 文件查看
    - 文件内容查看 cat
    - 文件内容逆序查看 tac
    - 分屏查看  more  less
    - 部分查看。head  tail
  - 文本统计
    - wc 
  - 文件编辑
    - 命令
      - vi/vim
      - nano
  - 文件信息
    - 展示文件或文件系统信息 stat
    - 文件类型查看 file
  - 文件分割
    - split
  - 文件/目录扩展属性
    - 属性选项
      a：只能追加内容，不能删除或修改。
      i：文件不能被删除、重命名、修改或链接。
      b：不更新文件或目录的最后存取时间。
      c：自动压缩文件，读取时解压缩，写入时压缩。
      u：删除文件时，文件内容会被保存，便于恢复。
      e：文件会被完全从磁盘上删除。
      s：文件被删除时，其内容会被完全覆盖，以提高安全性。
      S：文件写入时，数据会同步写入磁盘，以确保数据完整性。
      d：文件或目录被删除时，不会放入回收站，而是直接删除。
    - 改变文件或目录的扩展属性 chattr
      - 文件不可修改 chattr +i  filename
      - 允许文件追加内容 chattr +a  filename
    - 显示文件或目录的扩展属性 lschattr
  - 文件管理
    - 设置文件或目录权限 chmod
    - 权限掩码设置 umask
      /etc/profile、~/.bash_profile 或 ~/.profile 设置
    - 设置文件或目录拥有者或所属组  chown
    - 改变目录或文件所属组 chgrp
    - acl
      - 查看文件acl getfacl
      - setfacl
  - 压缩解压缩
    - tar  
      - tar -zcvf 压缩文件名 .tar.gz 被压缩文件名
      - tar -zxvf 压缩文件名.tar.gz
    - zip
      - zip -q -r   html.zip /home/Blinux/html
      - unzip 解压
    - gzip
      - gzip –c filename > filename.gz
      - gzip -d filename.gz

## 用户管理

- linux 系统是一个多用户多任务的操作系统，任何一个要使用系统资源的用户，都必须向系统管理员申请一个账号，用这个账号进入系统。
- 账户机制
  - 一个用户其实就是一个数字，UID用户ID，32位无符号整数，有关账户管理的一切几乎是围绕这个数字展开的。
- 用户管理
  - 用户添加 useradd
  - 用户删除 userdel
  - 用户修改 usermod
  - 用户锁定  usermod -L 用户名
  - 用户解锁 usermod -U 用户名
- 密码管理：
  - 修改密码 passwd
  - 账户的密码策略。chage
  - 批量修改密码。 chpasswd 命令
    - 用户切换 su  权限升级 sudo
    - 用户查询 /etc/passwd  用户信息都在这个文件（密码信息存放在 Linux /etc/shadow， Unix /etc/master.passwd）
        文件格式：用户名：密码（linux 显示为 x， unix 显示为 *）：用户ID：用户组 ID：注释：用户目录：登陆：shell
    - 用户密码加密：使用加盐SHA-512加密算法
    - 用户密码文件 /etc/shadow 格式
      - 登陆名：加密密码：最后一次更改密码的日期：密码更改的最小间隔天数：提前多少天通知用户密码将要过期：密码过期后多少天禁用该用户：账户过期日期：备用保留字段（目前为空）
    - 修改用户密码加密算法（修改加密算法并不会更改已有密码，需要手动用户手动更新，可以通过设置密码失效强制更新）
      - ubuntu/debin 在 /etc/pam.d/common-passwd 中找到
      - redhat/centos 在 /etc/login.defs 文件使用 authconfig 命令设置
    - 用户密码失效
      - chage -d 0 username
  - 批量添加用户
    - 方式一 可以使用 bash 脚本
    - 方式二
      编辑一个用户文件，每一列按照 /etc/passwd 密码文件的格式书写，注意每个用户名、UUID、宿主目录可以不相同，密码可以留空用 root 身份执行命令 /usr/sbin/newusers 从刚创建的文件 user.txt 导入数据，创建用户命令执行 /usr/sbin/pwunconv 编辑每个用户的密码对照文件 格式：用户名：密码以 root 身份执行命令 /usr/sbin/chpasswd执行命令 /usr/sbin/pwconv 将密码编码为 shadow passwd，并将结果吸入 /etc/shadow
- 组管理
  - 组ID也是一个 32位二进制整数无符号（uint）
    - GID 为 0 保留给 root 、system 或 wheel组
    - 系统也保留一些用于管理的预定义组，不同厂商定义不同。CentOS中 bin 组GID 为  1， Ubuntu 和 Debin 为 2
  - 用户组创建 groupadd
  - 删除用户组 groupdel
  - 修改用户组信息 groupmod
  - 一个用户有多个组权限可以进行组切换  newgrp groupname
  - 用户组信息都在 /etc/group文件中。格式 组名：口令：组织标号：组内用户列表
- PAM
  - 可插入式认证模块（系统认证集中化管理）
- 集中式账户管理
  - LDAP
- SSO（应用程序级别的单一登陆系统）


## 定时任务

- crontab
  linux 通过 crond 服务支持 crontab
  检查 crond 服务 systemctl list-unit-files｜ grep crond
  - crontab  -u user  用来设定某个用户的 crontab 服务
  - crontab -e。编辑某个用户的 crontab 文件内容，如果不指定用户，显示当前  的         crontab 文件
  - crontab -l。显示某个用户的 crontab 文件内容，如果不指用户，显示当前  的         crontab 文件
  - crontab -r。从 /var/spool/cron 目录中删除某个用户的 crontab 文件，  不指定用        户，默认删除当前用户的 crontab 文件
  - crontab -i  在删除用户的 crontab 文件时给提示
  - crontab 文件
  /etc/cron.allow 相当于白名单，优先级比 deny 高
  /etc/cron.deny. 相当于黑名单
  /etc/crontab.  系统级别的定时任务
  /etc/cron.d/ 这个文件夹下的配置同。/etc/crontab
  /var/spool/cron/ 存放每个用户的 crontab 所有权是特定的用户
  - crontab 文件。/etc/crontab
  格式 *  *  * * *
  取值范围。0-59 分钟
  取值范围。0-23 小时
  取值范围。 1-31 几号
  取值范围    1-12。月份
  取值范围。 0-7。星期几
- at
  - 一次性定时任务 at
    命令 at 从文件或标准输入中读取命令并在将来的一个时间执行，只执行一次。at 在正常执行时需要 atd 守护进程
  - anacron
    如果 linux 服务器在关机时间之内有系统定时任务（cron）需要执行，那么这些     任务是不会执行的，anacron 工具会使用一天，七天，一个月作为检测周期来判断    有定时任务在关机之后没有执行，anacron 工具会在特定的时间内重新执行这些任  centos6 中 anacron工具不再是单独的服务，而变成了系统命令。
    anacron 文件 /var/spool/anacron/目录存在 /etc/cron.{dailyweekly      monthly} 保存 anacron 工具上次执行的时间，
    配置文件 /etc/anacrontab 文件
		配置文件 /etc/anacrontab
天数。 强制延迟（分）。 工作名称。  实际执行命令
1   5   cron.daily.    nice run-parts /etc/cron.daily
	- systemd.timer

## Linux 文本三剑客

- 正则表达式
  - 常规正则
^ 尖角号，用于模式最左侧，匹配以 xx 开头
$ dollar 符，结尾
^$ 组合符，表示空格
.  匹配任意一个字符，不匹配空行
\\ 转义字符，特殊含义的字符显出原形
\* 匹配前一个字符，
.\* 组合匹配所有
^.\* 组合符，匹配任意多个字符开头内容
.*$ 匹配任意多个字符结尾的内容
[abc] 匹配[] 集合内任意字符
[^abc] 匹配除了 ^后面的字符
  - 扩展正则
- sed 文本替换
  文本替换 sed 's/book/books/' file
-n 选项或 -p 命令，打印发生替换的行。 sed -n 's/test/TEST/p' file
文件内容替换  sed -i 's/book/books/g' file
删除空白行  sed '/^$/d' filename
删除文件的第二行  sed '2d' filename
删除文件的第二行到末尾所有行 sed '2,$d' file
删除文件最后一行  sed '$d' file
删除文件中所有开头是 test 行  sed '/^test/d' file

- grep  文本过滤
	多级目录对文件递归检索。  grep "class" . -R -n
忽略匹配样式中的字符大小写. grep -i "hello"
匹配多个模式。 grep -e "class" -e "virtural"  file
只在目录中所有的 .php 和 .html 文件中递归搜索字符 main() grep "main()" . -r. --include *.{php,html}
在搜索结果中排出所有 README 文件。grep "main()"  . -r --exclude "README"

- awk 格式化文本
  - awk 语法
awk 参数 模式 动作 文件/数据
默认使用 空格 作为分隔符 多个空格视为一个分隔符
$0 代表一整行
$1  第一列
$NF 最后一列，列数
awk '\{print $1,$2\}' file
自定义输出 外部是单引号，内部双引号
显示指定行。awk 'NR==5' file
  - awk 内置变量
FS 输入字段分隔符默认空白符
OFS 输出字段分隔符默认空白符
RS 输入记录分隔符（输入换行符）
ORS 输出记录分隔符
NF 当前字段的个数
NR 行号
FNR 各文件分别计数的行号
指定分隔符 awk -F":" '{print $1}' filename. awk -v FS=':' '{print $1}' filename

## 软件管理

- RPM 软件包管理工具， rpm 是 redhat linux 发行版用来转门管理 Linux 各项套件的程序，由于遵循 GPL 规则且功能强大
  - 常用命令
rpm -qa 列出所有安装过的包
rpm -e 包名。 卸载软件包
rpm -ivh 包名  安装 rpm 包
  - yum 是 Fedora 和 Redhat 以及 SUSE 中基于 rpm 的包管理器工具，它可以使系统管理人员交互和自动化地管理 RPM 软件包，能够从指定的服务器自动下载 RPM 包并且安装，可以自动处理依赖性关系，并且一次性安装所有依赖的软件包
    - 自动搜索最快镜像插件。yum instal yum-fastestmirror
    - 安装 图形化窗口插件。yum install yumex
    - 查看可能批量安装的列表。yum grouplist
    - 安装
      yum install
      yum groupinstal group1  安装程序组
    - 更新和升级
      yum update。全部更新
      yum update package1。更新指定包
      yum check-update 检查可以更新的程序
      yum upgrade package1。升级指定程序
      yum groupdate group1。升级指定程序组
    - 查找
      yum info package1。显示安装包信息
      yum list。显示所有安装的软件包
      yum list package1。显示指定程序安装包情况
      yum groupinfo group1。显示程序组 group1 信息
      yum search string 根据关键字搜索软件包
    - 删除程序
      yum remove package 
      yum groupremove group1。删除程序组 group1
      yum deplist package1。 查看程序 package1 依赖情况
    - 清除缓存
      yum clean packages。 清除缓存目录下的软件包
      yum clean headers。 清除缓存目录下的 headers
      yum clean oldheaders。清除缓存目录下旧的 headers
- apt 是 debin linux 发行版中的 deb包管理工具，所有基于 debin 的发行版都使用这个包管理系统
  - 更新
    apt-get update
  - 安装
    apt-get install packagename
  - 卸载(保留配置文件)
    apt-get remove packagename
  - 卸载（删除配置文件）
    apt-get -purge remove package
  - 缓存清空
    apt-get clean
  - 更新所有安装的软件包
   apt-get upgrade

## 磁盘管理

- 磁盘介绍
  - 机械硬盘物理特性；
盘片（Platters）：盘片是机械硬盘的核心部件
扇区：最小的存储单位，主要有 512bytes 和 4k两种模式

- 硬盘识别（linux 根据设备类型完成识别）
  - IDE 存储设备在计算机中被识别为 hda ，hdb...
  - SATA、USB或 SCSI 设备会被识别为 sda, sdb ...
  - KVM 虚拟机的 vitio 硬盘设备，系统会识别为 vda, vdb
  - NVME 设备会被识别为 nvme0n1, nvme0n2....
- 分区(根据功能进行划分方便管理)
  - 硬盘分区方式
    - MBR（msdos 最多可以分4个主分区，单个分区容量有限，无法创建容量大于 2TB 的分区）
      - 传统分区方式中，如果需要更多分区，可以在扩展分区中创建逻辑分区
      - 具体操作步骤
        - fdisk -l 查看硬盘分区表（fdisk 是一个交互式命令）
        - fdisk /dev/sda 为 sda 硬盘进行分区
          - m 帮助
          - n 新建分区
            - p 创建主分区
            - e 创建扩展分区
            - 1 分区编号
          - d 删除分区
          - p 查看分区情况
          - w 保存退出
        - partprobe /dev//sda 立即读取分区表，无需重启即可识别新建分区
    - GPT 不受限制，还能提供分区表的冗余信息以实现分区表的设备与安全使用工具 parted
      - parted [选项] [硬盘 [命令]]
      - 分区操作
        - 修改分区类型 parted /dev/sda mklabel gpt
        - 查看修改效果 parted /dev/sdc print
        - 创建新的 ext4类型 分区 parted /dev/sdc mkpart primary ext4 1 2G
        - 创建新的 xfs 类型 分区 parted /dev/sdc mkpart primary xfs 2G 4G
        - 创建没有文件系统类型分区 parted /dev/sdc mkpart primary 6G 8G
        - 查看分区信息 parted /dev/sdc print
        - 删除新建分区 parted /dev/sdc rm 4
- 格式化
  - 将硬盘格式化为 ext4 类型 mkfs.ext4 /dev/sdc1
  - 交换分区格式化 mkswap /dev/sdc3
- 挂载文件系统
  - 手动挂载文件系统（临时方案）
    - mount /dev/sdc1 /data1 #挂载 /dev/sdc1 到 /data1
    - umount /dev/sdc1 # 取消挂载
  - 修改系统配置文件（永久方案）
    - /etc/fstab 系统配置文件
      - 文件内容：设备名/设备标签  挂载目录名称  文件系统类型  挂载属性  文件系统是否使用 dump 备份（0不备份 1每天备份）  文件系统在开机后使用 fsck 程序进行硬盘检测程序（1 表示最先检测， 2 表示第二个检测 0 表示不需要检测）
- swap 分区
  - 
- buffer cache
- RAID 独立冗余硬盘阵列
  - 原理：多块硬盘按照不同的方式组成成为一块逻辑硬盘
  - 分类：
    - 硬 RAID（有独立的控制芯片或控制卡，不需要消耗 CPU 内存等资源）
    - 软 RAID （mdadm 命令 Rocky linux9 目前支持的 RAID 0，1，4，5，6，10）
  - RAID 级别
    - RAID 0
    - RAID 1
    - RAID 4 
    - RAID 5
    - RAID 10
  - RAID 性能测试
    - 写入模拟 time dd if=/dev/zero of=/txt bs=1M count=1000
- LVM（动态管理存储）
  LVM 逻辑卷管理器，基于内核的 device mapper 功能。相比于直接在创建分区表后使用分区，LVM 提供了更灵活的管理方式：
  \* LVM 可以管理多个硬盘（物理卷）上的存储空间
  \* LVM 中的逻辑卷可以跨多个物理卷，文件系统不需要关心物理卷的位置
  \* LVM 的逻辑卷可以动态调整大小，而不需要移动分区的位置

  - 基本概念：
    - 物理卷（PV） 通常是一块硬盘
    - 卷组（VG）由多个物理卷组成
    - 物理扩展单元大小（PE）：将物理卷组合为卷组后所划分的最小存储单位，逻辑意义上硬盘的最小存储单元 LVM 默认 4MB
    - 逻辑卷（LV）在卷组里分配的逻辑存储空间，称之为【逻辑】，可以跨多个物理卷，也不一定是连续的
  - 命令
    - 普通分区或硬盘转换为物理卷 pvcreate /dev/sdc4 /dev/sde
    - 创建卷组 test_vg1 vgcreate test_vg1 /dev/sdb5 /dev/sdb6
    - 创建卷组 test_vg2 PE为 16M vgcreate test_vg1 /dev/sdb5 -s 16M /dev/sdb6
    - 从卷组中提取存储空间，创建逻辑卷
      - 从 test_vg1 中提取 2G 容量，创建名为 test_lv1 的逻辑卷 lvcreate -L 2G -n test_lv1 test_vg1
      - 使用 200 个PE 创建逻辑卷 test_lv2  lvcreate -l 200 -n test_lv2 test_vg2
    - 查看物理卷 pvdisplay
    - 查看卷组 vgdisplay
    - 查看逻辑卷 lvdisplay
    - 扩展 LVM 容量
      - 卷组有多余空间
        - 扩充逻辑卷 lvextend -L +120G /dev/test_vg/test_data
        - 调整文件系统大小 xfs_growfs /dev/test_vg/test_data
      - 卷组没有多余空间，有多余物理卷
        - 扩充卷组 vgextend test_vg /dev/sdb5
        - 扩充逻辑卷 lvextend -L +120G /dev/test_vg/test_data
        - 调整文件系统大小 xfs_growfs /dev/test_vg/test_data
      - 卷组没有多余空间，没有多余物理卷
        - 创建物理卷 pvcreate
        - 扩充卷组或创建一个卷组 vgextend test_vg /dev/sdb5
        - 扩充逻辑卷 lvextend -L +120G /dev/test_vg/test_data
        - 调整文件系统大小 xfs_growfs /dev/test_vg/test_data
    - 删除 LVM 分区
      - 卸载文件系统（记得更新 /etc/fstab 文件） umount /dev/test_vg/test_data
      - 删除逻辑卷 lvremove /dev/test_vg/test_data
      - 删除卷组 vgremove test_vg
      - 删除物理卷 pvremove /dev/sdc{2,3}
	备份
	术语
		文件系统：是一种用于磁盘上的数据结构，它提供了文件和目录的概念，以及对文件的读写、删除等操作。一个分区上只能有一个文件系统，但一个文件系统可以跨多个分区
		分区：通过在磁盘上划分出互不重叠的区域，可以将磁盘的容量划分为多个小块，用于不同的用途

## 进程管理

- 进程：运行程序的一组抽象，通过它可以管理和监控内存、处理器时间、I/O资源
  - 进程的组成（一个地址空间以及一组内核数据结构）
    - 地址空间：一系列内存页，由内核标记为进程所有
    - 内核数据结构
      - 进程空间地址映射
      - 进程当前状态（睡眠、停止、可运行等）
      - 进程的执行优先级
      - 进程占用的资源信息（CPU、内存等）
      - 进程打开的文件和网络端口信息
      - 进程的信号验码
      - 进程的属主
  - 线程：是进程中的执行上下文，进程至少有一个线程，而有些进程可以有多个线程
  - PID 进程的ID号
    - 内核会为每个进程分配一个唯一的ID号。大多数处理进程的命令和系统调用都需要指定 PID 来标识操作目标，PID是按照进程的创建顺序分配的
    - Linux 定义了“名字空间”的概念，可以进一步限制进程所能够感知到的资源以及相互影响的能力
  - PPID 父PID
    - Unix 和 Linux 中没有任何系统调用能够直接创建春运行特定程序的心进程。想要实现这一点，需要分两步走。首先已有的进程必须自我复制，创建一个新进程，这个复制体然后再把自己的内容替换成另一个程序。
    - 进程复制完成后，，原先的进程叫做负进程，复制出的副本叫做子进程。进程的 PPID 属性就是其父进程的 PID。
  - UID：是创建该进程的用户身份编号，就是其父进程的 PID 通常只允许创建者（也就是属主）和超级用户对进程进行操作。
  - EUID：有效的 用户ID，这是另一个能够决定进程在特定时刻有权限访问哪些资源和文件的 UID，大多数进程的 UID和 EUID 都是相同的。例外就是射者 setuid 的程序
  - GID：进程的组标识编号。EGID 与 GID与 UID和 EUID类似，GID属性差不多已经作废了，出于访问权限的目的，一个进程可以同时多个进程的成员。
  - 友善度：进程的调度优先级决定了能够获得 CPU 时间。
  - 控制终端（tty）：大多数守护进程都有与之关联的控制终端，控制终端决定了默认的标准输入，标准输出和标准错误输出。它还能够向进程发送信号，以响应键盘事件（ctrl-c ctrl-v）。真正控制终端智能在博物馆中见到了，现在都是伪终端，当启动一个 shell 命令时，终端窗口通常就会变成该进程的控制终端。
- 命令
  - ps 报告当前系统进程状态
    - 不加参数，显示当前用户所在终端的进程 ps（PID TTY TIME CMD）
    - ps -ef (e 和 -A 相同，显示现行终端机下的所有进程，包括其他用户进程， f 显示 uid, pid, parent pid, recent CPU usage, process start time, controlling tty, elapsed CPU usage, and the associated command)
    - 查找特定进程 ps -ef ｜ grep ssh
    - 使用 BSD 语法显示进程信息 ps -aux
      - USER：该进程属于的⽤户
      - PID：该进程号
      - %CPU：进程占用 CPU资源比率
      - %MEM：物理内存占用百分比
      - VSZ：进程使⽤的虚拟内存，单位Kbytes
      - RSS: 该进程占⽤固定的内存量，单位Kbytes
      - TTY: 该进程运⾏的终端位置
      - STAT: 终端目前状态
        - R 运行中
          S 睡眠中，可以被唤醒
          D 不可中断睡眠
          T 正在检测或者停止了
          Z 已停止，僵尸进程
          \+ 前台进程
          l 多线程进程
          N 低优先级进程
          < 高优先级进程
          s 进程领导者
          L 锁定到内存中
      - START 进程启动时间
      - TIME CPU 运行时间
      - COMMAND 进程命令
    - 显示指定用户进程 ps -u root
    - 显示进程树 ps -eH
    - 自定义格式： ps -eo pid，args，psr
  - 显示进程树 pstree
  - 显示进程数量 pstree -p
  - 根据进程名字查看信息 pgrep 
  - 发送信号到指定进程 kill
    - 常用信号
      - 1 挂起，终端掉线或者退出
      - 2 中断，通常是 ctrl + c
      - 3 退出，通常是 ctrl + \
      - 9 杀死进程
      - 15 软件终止
      - 20 暂停进程
    - 特殊信号 0 代表不发任何信号给 pid，但是会检测 pid 是否存在
  - 通过名字杀死进程 killall
    - killall nginx
    - 杀死所有用 test 用户启动的 nginx 进程 killall -u test nginx
  - 通过名字终止进程 pkill 比 killall 可能要执行多次，pkill 可以杀死进程及子进程
    - 通过进程名杀死 pkill nginx
    - 通过终端名杀死 pkill -t pst/1
    - 通过用户杀死 pkill -u nginx
  - top 实时监视系统进程状态资源使用情况
    - 常用参数
      -d num 设置时间间隔
      -u username 指定用户名
      -p pid 指定进程
      -n num 循环显示次数 
    - 交互式命令
      z：打开，关闭颜⾊
      Z: 全局显示颜⾊修改 
      h：显示帮助画⾯，给出⼀些简短的命令总结说明； 
      k：终⽌⼀个进程； 
      i：忽略闲置和僵死进程，这是⼀个开关式命令； 
      q：退出程序； 
      r：重新安排⼀个进程的优先级别； 
      S：切换到累计模式； 
      s：改变两次刷新之间的延迟时间（单位为s），如果有⼩数，就换算成ms。输⼊ f或者
      F：从当前显示中添加或者删除项⽬； 
      o或者O：改变显示项⽬的顺序； 
      l：切换显示平均负载和启动时间信息； 
      m：切换显示内存信息； 
      t：切换显示进程和CPU状态信息； 
      c：切换显示命令名称和完整命令⾏； 
      M：根据驻留内存⼤⼩进⾏排序； 
      P：根据CPU使⽤百分⽐⼤⼩进⾏排序； 
      T：根据时间/累计时间进⾏排序； 
      w：将当前设置写⼊~/.toprc⽂件中。 
      B：全局字体加粗 数字1：⽤于多核监控CPU，监控每个逻辑CPU的情况 
      b：打开，关闭加粗 x，⾼亮的形式排序对应的列
      < > ：移动选择排序的列
    - 显示进程信息
      - 第一行系统信息：
        
  - 后台运行 nohup
  - bg
  - fg
  - init （ /etc/inittab 根据这个配置文件创建 Linux 进程 ）系统初始化工具，是所有 linux 进程的父进程，进程 ID 是1
    - 0 停机
    - 1 单用户模式
    - 2 多用户模式，没有 NFS
    - 3 完全多用户模式（标准的运行级）
    - 4 没有用到
    - 5 x11
    - 6 重新启动
  - runlevel 显示当前运行级别

## 网络管理

- 网卡信息
  - ifconfig （net-tools 包）
    - 启动网卡  ifconfig ens33 up
    - 关闭网卡  ifconfig ens33 down
    - 设置ip  ifconfig ens33 192.168.1.1
    - 设置ip  ifconfig ens33 192.168.1.1 netmask 255.255.255.0
    - 设置ip  ifconfig ens33 192.168.1.1/24 up
    - 修改 MAC 地址 ifconfig ens33 hw ether 00:0C:29:12:10:CF
  - 修改网卡配置文件 redhat 系列 （/etc/sysconfig/network-scripts/）
  - ifup ens33 启动网卡
  - ifdown ens33 关闭网卡
  - route
    - route 查看路由表
    - route -n 不解析 DNS
    - 删除默认网关 route del default
    - 添加路由条目 route add default gw 192.168.178.2
  - arp 显示所有缓存条目
  - ip 配置（临时配置）
    - 配置网卡接口地址(可以为同一个网卡配置多个地址) ip add 172.16.0.16/16 dev ens160
    - 查看网卡接口信息 ip addr show ens160
    - 查看网卡的物理链路信息 ip link show ens160
    - 删除网卡的IP地址配置 ip addr del 172.16.0.17/16 dev ens160
    - 关闭 ens160 网卡接口 ip link set ens160 down
    - 激活 ens160 网卡接口 ip link set ens160 up
    - 添加默认路由 ip route add default via 172.16.0.254
    - 添加自定义路由表信息 ip route add 172.17.0.0/24 via 172.17.0.1
    - 删除路由表信息 ip route del 172.16.5.0/24 via 172.16.5.1
  - 网络参数配置 nmcli 命令（永久配置）
    - 查看网卡物理设备信息 nmcli device show
    - 查看网络连接信息（UUID 对应 网卡名） nmcli connection showw
    - 删除网络连接（不会删除网卡）nmcli connection delete ens160
    - 配置网卡
      - 创建网络连接
        - nmcli connection add ifname ens160 con-name ens160 type ethernet
      - 配置网络连接的 IP地址、子网掩码、网关、DNS等
        - nmcli connection modify ens160 ipv4.method auto autoconnect yes\
          ipv4.method manual
          ipv4.address 172.16.0.160/16
          ipv4.gateway 172.16.0.254
          ipv4.dns 8.8.8.8
          autoconnect yes
      - 配置静态路由 nmcli connection modify ens160 ipv4.routes "172.16.8.0/24 172.16.8.1, 172.16.9.0/24 172.16.9.1"
      - 激活网卡 nmcli connection down ens160 && nmcli connection up ens160 up
    - 修改系统配置文件参数 Rocky9 /etc/NetworkManager/system-connections/ 目录中
    - netstat 显示网络连接、接口状态、伪连接、网络链路信息和组播成员
      - 显示当前路由表 netstat -rn
      - 显示网络接口情况 netstat -i
    - ss （iproute包）CentOS7 替代 netstat 命令，用于查看网络连接状态，
      - 显示所有 socket 连接 ss -an
      - 格式化输出 ss -an ｜ column -t
      - 显示正在监听的 TCP 和 UDP 连接 ss -tupnl ｜ column -t
      - 统计服务器连接数 ss -s
    - telnet 远程登录，一般用来测试 TCP 接口
    - wget 
    - curl
    - nslookup 域名查询（bind-utils）
      - nslookup www.baidu.com
    - nmap 嗅探命令（一般不要使用，企业内部有探测）
- 网络排障
  - 原则：
    - 一次只修改一个地方，每次改动都要测试，如果没有达到预期，恢复操作
    - 记录下你之前参与的情况，将之后所做的每一步都记录
    - 从系统或网络的一端开始，逐步排查关键部件，直到找出问题所在
    - 利用 OSI 参考模型从低向上，或自顶向下进行排查
  - 按照参考模型自下向上进行排查时，要问自己
    - 有没有物理线路或链路指示灯
    - 接口是否配置正确
    - ARP 表中有没有显示其他主机
    - 本地机器上有没有安装防火墙
    - 你当前所在位置和目的地之间是否有防火墙
    - 如果有防火墙，则是否能够响应 ICMP ping
    - 本地地址能否 ping 通
    - 能否 ping 其他本地主机 IP
    - DNS 工作是否正常
    - 能否 ping 通其他主机名
    - WEB 或 SSH 等高层应用是否可以访问
    - 检查过防火墙吗？
    - 使用命令 ping、traceroute、tcpdump、ss 、nslookup
- 网络监控
  - SmokePing
- 网络性能测试
  - iperf
- 双网卡绑定

## 安全

- 防火墙
  - firewalld
  - iptables
- selinux （redhat 系列用）
		工作模式：
enforce： SElinux 根据安全策略，积极阻止有潜在风险的操作
permission：仅记录会被阻止的操作，
disable：selinux 禁用，日志也不记录
	AppArmor （debin 系列用）
- OpenVPN
- WireGuard

## 日志
- 日志管理
  - 从各种源头收集日志
  - 为消息的查询、分析、过滤、监视提供结构化接口
  - 管理消息的保留和日期，以便消息在发挥作用或合法期间可以尽可能久地保留，遵循相关规范。
- systemd journal
- syslog

## 驱动程序和内核


## 打印

## 词汇

- DMI（Desktop Management Interface，桌面管理接口）是一种用于管理和跟踪计算机系统硬件信息的标准规范。
- 守护进程