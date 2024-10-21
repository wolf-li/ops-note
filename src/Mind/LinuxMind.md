<!--
 * @Author: wolf-li
 * @Date: 2024-10-20 21:34:27
 * @LastEditTime: 2024-10-21 14:03:56
 * @LastEditors: wolf-li
 * @Description: 
 * @FilePath: /note/src/ThinkMind.md
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
- 文件权限
  - rwx rwx rwx
    文件类型  所属用户权限 所属组权限 其他用户权限
    数字表示 r-4 w-2 x-1  - 0 
    \- 没有权限
    s 特殊功能
  - 文件默认权限。644
  - 目录默认权限   755
  - 文件类型
    - ◆- 普通文件
    - ◆d 目录文件
    - ◆b 块设备
    - ◆c 字符设备
    - ◆l 符号链接文件
    - ◆p 管道文件pipe
    - ◆s 套接字文件socket
  - 文件的3种时间
    - atime：Access Time 最后一次访问文件（读取或执行）或目录的间；
    - mtime：Modofy Time 最后一次修改文件内容（数据）或目录内（目录内文      - 件列表）的时间；
    - ctime：Change Time 最后一次改变文件属性（元数据）或目录属（元数据）的时间；
    - 查看方式
      - atime: ls -lu filename
      - mcime: ls -l filename
      - ctime: ls -lc filename
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
    - 用户查询 /etc/passwd  用户信息都在这个文件
        文件格式：用户名：密码：用户ID：用户组 ID：注释：用户目录：登陆：shell
  - 批量添加用户 
    - 方式一 可以使用 bash 脚本
    - 方式二 
      编辑一个用户文件，每一列按照 /etc/passwd 密码文件的格式书写，注意每个用户名、UUID、宿主目录可以不相同，密码可以留空用 root 身份执行命令 /usr/sbin/newusers 从刚创建的文件 user.txt 导入数据，创建用户命令执行 /usr/sbin/pwunconv 编辑每个用户的密码对照文件 格式：用户名：密码以 root 身份执行命令 /usr/sbin/chpasswd执行命令 /usr/sbin/pwconv 将密码编码为 shadow passwd，并将结果吸入 /etc/shadow
- 组管理
  - 用户组创建 groupadd
  - 删除用户组 groupdel
  - 修改用户组信息 groupmod
  - 一个用户有多个组权限可以进行组切换  newgrp groupname
  - 用户组信息都在 /etc/group文件中。格式 组名：口令：组织标号：组内用户列表

## 定时任务
      
- crontab 
  linux 通过 crond 服务支持 crontab
  检查 crond 服务 systemctl list-unit-files｜ grep crond
  		crontab  -u user  用来设定某个用户的 crontab 服务
  crontab -e。编辑某个用户的 crontab 文件内容，如果不指定用户，显示当前  的         crontab 文件
   crontab -l。显示某个用户的 crontab 文件内容，如果不指用户，显示当前  的         crontab 文件
  crontab -r。从 /var/spool/cron 目录中删除某个用户的 crontab 文件，  不指定用        户，默认删除当前用户的 crontab 文件
  crontab -i  在删除用户的 crontab 文件时给提示
  		crontab 文件
  /etc/cron.allow 相当于白名单，优先级比 deny 高
  /etc/cron.deny. 相当于黑名单
  /etc/crontab.  系统级别的定时任务
  /etc/cron.d/ 这个文件夹下的配置同。/etc/crontab
  /var/spool/cron/ 存放每个用户的 crontab 所有权是特定的用户
  		crontab 文件。/etc/crontab
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