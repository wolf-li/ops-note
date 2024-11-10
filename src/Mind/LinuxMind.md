<!--
 * @Author: wolf-li
 * @Date: 2024-10-20 21:34:27
 * @LastEditTime: 2024-11-10 17:08:04
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

- perl

## 引导程序

- 引导（“启动计算机”的标准说法，bootstrapping 一词的缩写，使用这个词是计算机必须自己动起来）
- 引导过程
  - 加电
  - 从 NVRAM 中装载 BIOS/UEFI
  - 探测硬件
  - 选择引导设备（磁盘、网络...）
  - 识别 EFI 系统分区
  - 载入引导装载程序 GRUB
  - 确定引导那个内核
  - 装载内核
  - 实例化内核数据结构
  - 启动 init/systemd(PID为1)
  - 执行启动脚本
  - 运行系统
- 系统固件：机器加电后，CPU会执行存储在 ROM 中的引导代码。在虚拟系统中，这个 ROM 未必真实存在，但概念还是一样的。
  - 系统固件通常知晓主板上安装的所有设备，比如 STAT控制器，网络接口卡，USB控制器以及电源和温度传感器。
    对于物理硬件（相对于虚拟硬件），多数固件都提供了用户界面。不过这个界面一般都比较粗糙，而且不太容易访问到
    需要在系统加电后迅速按下特定的按键。具体看厂商（control 、F6、F8）。在正常的引导过程中，系统固件会侦测硬件和
    磁盘，执行一些简单的健康检查，然后查找下一阶段的引导代码。你可以使用固件的用户界面指定引导设备，一般都是通过设置
    优先级来实现的。
  - 传统 BIOS
    - 传统 BIOS 认为引导设备是以主引导记录（MBR）为起始。MBR包括第一阶段的引导装载程序（也叫做“引导块”）和一个原始的磁盘分区表。可供引导程序装载程序
      使用的空间非常有限（也叫做“引导块”）和一个原始的磁盘分区表。可用引导程序使用的空间非常有限（不足 512字节）所以除了载入运行第二阶段的引导装载程序
      之外，不能进行其他操作。
    - 引导块和 BIOS 都没有复杂到能够读取任意类型的标准文件系统，所以第二阶段的引导装载程序必须放在一个容易找到的地方。在典型的场景中，引导块从 MBR 中
      读取分区信息，识别出标注为“活动”的磁盘分区。然后读取并执行位于该分区起始位置上的第二阶段引导装载程序。分区上的这个区域也被成为卷引导记录。
  - UEFI（Unified Extensible Firmware Interface 统一可扩展固件接口）
    - UEFI 规范中包含了一个叫做 GUID 分区表（GUID Partition Table，GPT）的现代磁盘存储方案。UEFI能够理解文件分配表（FAT）文件系统，这种源自 MS-DOS
      的文件系统即简单又有效。这些特性结合在一起形成了一个新的概念：EFI系统分区。在引导期间固件会通过查询 GPT 来识别 ESP，然后从 ESP中的文件直接读取并执行
      配置好的目标应用程序。ESP只是一个普通的 FAT 文件系统，它可以有任何操作系统挂载、读取、写入以及维护。磁盘上不再需要有任何隐匿的引导块。
    - 显示引导选项的汇总信息 efibootmgr -v
    - 尝试从系统硬盘引导，然后在尝试网络，同时忽略其他引导选项，可以使用 sudo efibootmgr -o 0004，0002 （0004， 0002 从上一个命令的结果获取的 bootOrder）
- 引导装载程序:主要任务是识别并载入相应的操作系统内核。大多数引导装载程序会在引导期间提供一个界面，让用户在多个内核或操作系统中选择
  - GRUB（GRand Unified Boot loader， GNU开发是大多数 linux 发行版默认的引导装载程序）
    - GRUB 有两个分支：传统的 GRUB（GRUB Legacy），另一个全新的版本叫做 GRUB2
    - GRUB的配置文件叫做 grub.cfg 通常位于 /boot/grub（redhat 为 /boot/grub2）
    - 不要手动修改 grub.cfg 文件，通常都是利用工具 grub-mkconfig 生成。实际上大多数发行版都会在内核升级之后自动重新生成 grub.cfg，可以修改模版文件 /etc/default/grub
      - /etc/default/grub 配置参数，修改后运行 update-grub 或 grub2-mkconfig
        GRUB_BACKGROUND 背景图片
        GRUB_COMMANDLINE_LINUX 添加到菜单表项中的 Linux 参数
        GRUB_DEFAULT 默认菜单项的编号或标题
        GRUB_DISABLE_RECOVERY 不生成恢复模式的菜单选项
        GRUB_PRELOAD_MODULES 需要尽早装载 GRUB 模块列表
  - FreeBSD引导过程（了解）
- 系统管理守护进程
  - 一旦内核被加载并完成了初始化过程，它就会在用户空间创建“自发”进程。称为字符进程的原因在于是由内核自主启动的，而在正常情况下，仅在现有进程出发请求时才会创建新进程
  - init 首要任务是确保系统在任何时刻都运行着正确的服务和守护进程，init 定义了系统操作模式的概念：
    - 单用户模式：该模式中只挂载少数量的文件系统，不运行任何服务，在控制台中启动 root shell
    - 多用户模式：在该模式中挂载所有定制的文件系统，启动所有配置好的网络服务以及窗口系统和控制台的图形化登录管理器
    - 服务器模式：类似于多用户模式，除了不启动图形化用户界面。
    - 从引导到进入多用户模式的过程中，init 通常有一堆活要做：设置计算机名、设置时区、使用 fsck 检查磁盘、挂载文件系统等
  - init 实现
    - SysV init：系统模式叫做“运行级”。大数据运行级都使用单个字母或数组表示
      - 问题：init 自身并没有强大到满足处理现代操作系统的需求。init 的大部分系统实际上都有一套标准化的、固定不变的 init 配置。该配置指向另一层 shell 脚本
        由其负责改变运行级、修改配置。而第二层脚本中又包括第三层针对特定守护进程和系统的脚本，这些脚本交叉连接到特定的运行级的目录，目录中明确了需要在某些运行级
        下运行的服务。init 缺少一个能够描述服务之间依赖关系的通用模型，因而所有的启动脚本和卸载脚本只能以串行方式运行，有系统管理员负责维护。后面的操作系统必须
        等到之前的操作全部结束后才能执行，所以根本无法实现并行操作，系统需要花费大量的时间用来改变状态。
    - systemd（现在主流）：吸收了之前通过胶带、脚本、管理员实现的 init 特性，正式形成了一套有关如何配置、访问、管理服务的统一理论
- 关机
  - halt 命令会执行关闭系统所有必须的操作，该命令会将关机事件写入日志中、杀死不必要的进程，将已缓存的文件系统块写会磁盘，然后挂起内核。在大多数系统中 halt -p 会最后切断系统电源
  - reboot 基本上和 halt 一样，只不过它做的是重新引导。
  - shutdown 命令建立在 halt 和 reboot 之上，它能够实现定时关机以及想登陆用户发送警告
  - 云主机关机要注意有的云厂商 stop 和 reboot 操作和你想的不一样，需要查看文档。
- 系统无法引导的应对策略
  - 不调试，将系统恢复到已知的良好状态（利用快照，重新挂载磁盘）
  - 是系统足以运行一个 shell，然后进行交互式调试
  - 引导另一系统镜像，挂载故障系统的文件系统并以此开展检查

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
  格式 \*  \*  \*  \*  \*
  取值范围。0-59 分钟
  取值范围。0-23 小时
  取值范围。 1-31 几号
  取值范围    1-12。月份
  取值范围。 0-7。星期几
- at
  - 一次性定时任务 at
    命令 at 从文件或标准输入中读取命令并在将来的一个时间执行，只执行一次。at 在正常执行时需要 atd 守护进程
- anacron
    如果 linux 服务器在关机时间之内有系统定时任务（cron）需要执行，那么这些任务是不会执行的，anacron 工具会使用一天，七天，一个月作为检测周期来判断有定时任务在关机之后没有执行，anacron 工具会在特定的时间内重新执行这些任  centos6 中 anacron工具不再是单独的服务，而变成了系统命令。
    anacron 是一个程序不是一个服务，当操作系统进入 crontab 的任务列表同时 anacorn 会每小时被主动执行一次。
    anacron 文件 /var/spool/anacron/目录存在 /etc/cron.{dailyweeklymonthly} 保存 anacron 工具上次执行的时间，
    配置文件 /etc/anacrontab 文件
    配置文件 /etc/anacrontab
天数。 强制延迟（分）。 工作名称。  实际执行命令
1   5   cron.daily.    nice run-parts /etc/cron.daily
  - 命令语法
    - anacron -u [job]
- systemd.timer

## Linux 文本三剑客

- 正则表达式
  - 基本正则
    - ^ 尖角号，用于模式最左侧，匹配以 xx 开头
    - $ dollar 符，结尾
    - ^$ 组合符，表示空格
    - .  匹配任意一个字符，不匹配空行
    - \\ 转义字符，特殊含义的字符显出原形
    - \* 匹配前一个字符，
    - .\* 组合匹配所有
    - ^.\* 组合符，匹配任意多个字符开头内容
    - .*$ 匹配任意多个字符结尾的内容
    - \{n,m\} 匹配前一个字符，重复 n 到 m 次
    - \{n,\} 匹配前一个字符，重复至少 n 次
    - \{n\} 匹配前一个字符，重复 n 次
    - \(\) 将 () 之间的内容存储在保留空间，最多存储 9 个
    - [x-z] 匹配连续字符范围
    - [abc] 匹配[] 集合内任意字符
    - [^abc] 匹配除了 ^后面的字符
  - 扩展正则 grep -E 或 egrep
    - {n,m} 等同于基本正则 {n,m}
    - \+ 匹配 \+ 前面的字符，出现一次或多次
    - ? 匹配 ? 前面的字符，出现 0 次或一次
    - | 匹配逻辑 或
    - () 匹配正则集合
  - POSIX 规范
    - 字符集 含义
      [:alpha:] 字母字符
      [:alnum:] 数字字符
      [:cntrl:] 控制字符
      [:digit:] 数字字符
      [:xdigit:] 十六进制数字字符
      [:punct:] 标点字符
      [:graph:] 非空格字符
      [:print:] 任何可以显示的字符
      [:space:] 任何产生空白的字符
      [:blank:] 空格与 tab 健字符
      [:lower:] 小写字符
      [:upper:] 大写字符
  - Perl 正则 grep -P
    - \B 与 \b 为反义词，\Bthe\B 不匹配单词 the, 仅会匹配中间含有 the 的单词
    - \d 可以匹配数字
    - \b 边界字符，匹配单词的开始或结尾
    - \D 匹配所有非数字
    - \s 匹配所有空白符号（比如空格、Tab缩紧、制表符）
    - \S 匹配所有非空白符号
    - \w 可以匹配字母、数字、下划线
    - \W 可以匹配除字母、数字、下划线以外的符号
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

- 打包系统（packaging system）：简化了软件管理工作。软件包内运行软件所有需要的全部文件，包括预编译好的二进制，依赖信息，可由管理员定制的模版等
- RPM 软件包管理工具， rpm 是 redhat linux 发行版用来转门管理 Linux 各项套件的程序，由于遵循 GPL 规则且功能强大
  - 常用命令
    rpm -qa 列出所有安装过的包
    rpm -e 包名。 卸载软件包
    rpm -ivh 包名  安装 rpm 包
    rpm -q --whatrequires \<packageName> 查看某个软件包的依赖
    rpm -U \<packageName> 更新某个软件包
- yum 是 Fedora 和 Redhat 以及 SUSE 中基于 rpm 的包管理器工具，它可以使系统管理人员交互和自动化地管理 RPM 软件包，能够从指定的服务器自动下载 RPM 包并且安装，可以自动处理依赖性关系，并且一次性安装所有依赖的软件包
  - 仓库配置文件 /etc/yum.repos.d/
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
  - 自定义软件源（dnf/yum都可以）
    - 适用场景
      - 当收集的软件越来越多时，有必要将这些软件汇总并创建自己的软件源
      - 离线部署
    - 操作步骤
      - 安装 createrepo
      - 所有的软件包保存到某个目录 比如：/soft
      - 运行 createrepo /soft 自动给 /soft 中的软件生成配套的数据库信息
- dnf 新一代的 Yum v4，采用了 DNF 架构，DNF在性能和资源占用上都比老版本的 yum 有了很多优化，为了和旧版本兼容，新版操作系统依然可以使用 yum 命令
  - 常用命令
    - 查看当前系统已经配置的软件源
      dnf repolist
    - 查看软件源的详细信息
      dnf repolist -v
      dnf repoinfo
    - 清空缓存
      dnf clean all
    - 软件包安装
      dnf install -y \<packageName>
    - 软件源更新
      dnf update
    - 卸载软件包
      dnf remove \<packageName>
    - 搜索软件包
      dnf search \<packageName>
    - 查看 dnf 安装和卸载历史
      dnf history
    - 下载软件包到当前目录
      dnf download \<packageName>
    - 查看那个软件包可以提供 xx 命令
      dnf provides xx
- apt 是 debin linux 发行版中的 deb包管理工具，所有基于 debin 的发行版都使用这个包管理系统
  - 仓库配置文件 /etc/apt/sources.list
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
- dpkg 管理 .deb 包
  - 查看某个软件是否安装
    dpkg -l \<packagename>
  - 安装某个软件包
    dpkg --install \<packagename>
- 常见问题
  - 软件包依赖问题：使用 rpm 安装时，如果系统提示某个软件包依赖其他软件包，但该系统没有这个依赖，可以使用 --nodeps 选项忽略依赖关系
  - RPM 数据库损坏：rpm 软件包到相关数据库存放在 /var/lib/rpm 目录中，如果数据库出现损坏，可以使用 rpm --rebuilddb 修复数据库资料
  - 软件安装的时间问题：软件安装时系统提示，“warning： clock skew detected”说明系统时间严重出错，可以通过 date -s "2023-01-12 14:00" 修改系统时间
  - DNF 繁忙的问题：可能有其他人使用，或者安装卡死。找到进程杀死dnf 相关进程即可。
  - GCC 编译器问题：依赖问题，当前操作系统内没有 gcc ，需要手动安装。

## 磁盘管理

- 磁盘介绍
  - 机械硬盘物物理组成（圆形碟片、机械手臂、磁头、主轴马达）
    - 数据读取过程：主轴马达让碟片转动，然后机械手臂可伸展让磁头在碟片上进行读写操作
    - 碟片上的数据存放概念
      - 盘⽚在设计时就是在 盘⽚的同⼼圆上，切出⼀个个的⼩区块，这些⼩区块整合 成⼀个圆环，让机器⼿臂上的磁头去存取。
      - 扇区：小区块就是扇区（磁盘最小的数据存储单位 512B ，硬盘容量增大最近都为 4KB）
      - 磁道：同一个同心圆内的扇区组合成的圆就是磁道
      - 柱面：所有碟片上面的同一个磁道可以组合成一个柱面
    - 机械硬盘传输接口
      - SATA接口：SATA是Serial ATA的缩写，即串⾏ATA。它是⼀种电脑总线，主要功能是 ⽤作主板和⼤量存储设备(如硬盘及光盘驱动器)之间的数据传输之⽤。
        - SATA信息 （传统物理硬盘传输极限速度在 150～200MB/s 之间）
        版本      带宽（Gbit/s）   速度（MB/s）
        SATA1.0  1.5             150
        SATA2.0  3               300
        SATA3.0  6               600
      - SAS接口：SAS(Serial Attached SCSI)即串⾏连接SCSI，是新⼀代的SCSI技术，和 现在流⾏的Serial ATA(SATA)硬盘相同，都是采⽤串⾏技术以获得更⾼的 传输速度，并通过缩短连结线改善内部空间等。
        - SAS是 并⾏SCSI接⼝之后 开发出的全新接⼝
        - 此接⼝的设计是为了 改善存储系统的效能、可⽤性和扩充性 ，并且提供与SATA硬盘的兼容性。
        - SAS 信息
          版本    带宽（Gbit/s）   速度（MB/s）
          SAS1    3              300
          SAS2    6              600
          SAS3    12             1200
      - SAS和SATA磁盘在价格上差别很⼤，SATA磁盘价格低廉，SAS有着更⾼ 的存储附加属性，很多企业在数据中⼼还是使⽤的SAS磁盘，并且企业级 磁盘阵列卡的连接插槽也是由SAS接⼝开发的
- 硬盘识别（linux 根据设备类型完成识别）
  - IDE 存储设备在计算机中被识别为 hda ，hdb...
  - SATA、USB或 SCSI 设备会被识别为 sda, sdb ...
  - KVM 虚拟机的 vitio 硬盘设备，系统会识别为 vda, vdb
  - NVME 设备会被识别为 nvme0n1, nvme0n2....
- 分区(根据功能进行划分方便管理)
  - 硬盘分区方式
    - MBR（msdos 最多可以分4个主分区，单个分区容量有限，无法创建容量大于 2TB 的分区）
      - MBR组成(数据大小512B)
        - 主引导记录（MBR）：位于MBR(0-446)B，负责启动操作系统的加载，当计算机启动时，BIOS会读取MBR中的引导程序，并将控制权传递给它，常见的引导程序（GRUB Linux， NTLDR windows）
        - 分区表（partition table）：记录整块硬盘分区信息，(447-510)64B
          - 分区表包含四个分区记录每个记录 16字节，用于描述硬盘上的分区信息
          - 每个分区记录包含以下信息
            - 分区状态：活动/非活动
            - 分区起始磁头、磁道、扇区
            - 分区类型：FAT32、NTFS、EXT4等
            - 分区结束磁头、磁道、扇区
            - 分区起始逻辑块地址（LBA）
            - 分区大小
        - 魔法数字（Magic Number）
          - 位于MBR的最后两个字节，值为 0xAA55
          - BIOS通过检查这个魔术数字来验证 MBR 的有效性
        - MBR 信息查看（Linux）
          - sudo fdisk -l /dev/sdX
          - sudo dd if=/dev/sdX bs=512 count=1 | hexdump -C
          - sudo gdisk -l /dev/sdX
          - sudo dd if=/dev/sdX of=mbr.bin bs=512 count=1
        - MBR 限制
          - 分区数量限制：最多只能有四个主分区或者三个主分区加厚收纳柜一个扩展分区
          - 磁盘大小限制：由于MBR使用 32位来寻址逻辑块，最大支持容量为 2TB实际由于 CHS寻址方式限制，单个分区最大容量为 2TB
          - 兼容性问题：MBR与BIOS兼容，但不支持 UEFI启动
      - 传统分区方式中，如果需要更多分区，可以在扩展分区中创建逻辑分区
        - 磁盘默认的分区表仅能写入四组分区信息
        - 四组分区信息称为主要（primary）或扩展（Extended）分区
        - 分区的最小单位通通常为柱面
        - 当系统要写入磁盘时，一定会参考分区表，才能针对某个分区进行数据的处理
        - 分区的最小单位通常为柱面
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
    - GPT（GUID partion table）磁盘分区表。相对于 MBR 不受限制，还能提供分区表的冗余信息以实现分区表的设备与安全使用工具 parted、gdisk
      - GPT结构
        - 为了兼容所有硬盘，在扇区定义上，大多数会使用逻辑区块地址（logical block address LBA，默认 512B）处理。GPT 使用 34 个 LBA区块来记录分区信息。整个磁盘最后 34 个 LBA也拿来作为另一个备份。
        - LBA0（兼容 MBR 区块）保护MBR：
          - 位于硬盘的起始位置，占一个扇区
          - 包含一个类型为 0xEE 的分区，覆盖整个硬盘，可以防止 MBR 工具错误地格式化 GPT 硬盘
          - 不用于启动，仅用于向后兼容
        - LBA1（GPT 表头记录）：包含分区表的位置和大小、磁盘的唯一标识符（GUID）、分区表和校验和
        - LBA2-33（实际记录分区信息处）：从LBA2开始，每个 LBA 都可以记录4组分区记录，默认可以有 128 组分区记录。
          - 每个分区表项描述一个分区，包括分区类型GUID、唯一标识符、起始和结束的 LBA 以及属性标志
      - parted [选项] [硬盘 [命令]]
      - 分区操作
        - 修改分区类型 parted /dev/sda mklabel gpt
        - 查看修改效果 parted /dev/sdc print
        - 创建新的 ext4类型 分区 parted /dev/sdc mkpart primary ext4 1 2G
        - 创建新的 xfs 类型 分区 parted /dev/sdc mkpart primary xfs 2G 4G
        - 创建没有文件系统类型分区 parted /dev/sdc mkpart primary 6G 8G
        - 查看分区信息 parted /dev/sdc print
        - 删除新建分区 parted /dev/sdc rm 4
  - 为什么要分区
    - 数据的安全性
      - 每个分区数据是分开的。你需要将某个分区数据进行整理是，可以移动数据到另一分区备份
    - 系统的性能考虑（机械硬盘）
      - 由于分区将数据集中在某个柱面段中，数据集中，将有助于数据读取到速度与性能
- 格式化
  - 将硬盘格式化为 ext4 类型 mkfs.ext4 /dev/sdc1
  - 交换分区格式化 mkswap /dev/sdc3
- 挂载文件系统 (文件系统：一种用于磁盘上的数据结构，提供了文件和目录的概念，以及对文件读写、删除等操作，一个分区上只能有一个文件系统，一个文件系统可以有多个分区)
  - 手动挂载文件系统（临时方案）
    - mount /dev/sdc1 /data1 #挂载 /dev/sdc1 到 /data1
    - umount /dev/sdc1 # 取消挂载
  - 修改系统配置文件（永久方案）
    - /etc/fstab 系统配置文件
      - 文件内容：设备名/设备标签  挂载目录名称  文件系统类型  挂载属性  文件系统是否使用 dump 备份（0不备份 1每天备份）  文件系统在开机后使用 fsck 程序进行硬盘检测程序（1 表示最先检测， 2 表示第二个检测 0 表示不需要检测）
- swap 分区
  - swap space 是磁盘上的一块区域，可以是一个分区，也可以是一个文件，或者组合方式出现，当系统物理内存吃紧时，Linux 会讲内存中不常访问的数据放入 swap 上，这样系统有更多物理内存为其他进程服务，而当系统要访问 swap 上存储的内容时，系统会将 swap 上的数据加载到内存中。
  -
- buffer cache
- RAID 独立冗余硬盘阵列
  - 原理：多块硬盘按照不同的方式组成成为一块逻辑硬盘
  - 分类：
    - 硬 RAID（有独立的控制芯片或控制卡，不需要消耗 CPU 内存等资源）
    - 软 RAID （mdadm 命令 Rocky linux9 目前支持的 RAID 0，1，4，5，6，10）
  - RAID 级别
    - RAID 0 也成条带化，将数据分块存储在多个硬盘上，可以充分利用所有容量，获得叠加的顺序读写性能（但随机读写一般），没有冗余，任何一块磁盘损坏都会导致整个阵列数据丢失，适合需要高性能但不需要数据安全的场景
    - RAID 1 也称镜像，将数据完全复制到多个盘上，提供了绝对冗余，整个阵列中只要有一块磁盘存储数据久不会丢失。空间利用率低，适合需要高可靠的场景，同时由于每块盘上的数据完全一致，RAID 1 的读取性能可以叠加，但写入性能不会提升
    - RAID 4
    - RAID 5 将数据和一份校验信息存储在多个盘上，可以允许阵列中一块磁盘损坏，重建期间性能会严重下降。
    - RAID 10 = RAID 1 + RAID 0
  - RAID 性能测试(dd 只能测试顺序写，无法测随机读写，dd 使用非常低的 I/O 队列深度无法重复测试设备的并发尤其是固态，dd 命令使用的特殊设备 /dev/urandom/dev/random本身性能有限，对磁盘进行更全面的性能测试需要使用 fio)
    - 写入测试  dd if=/dev/zero of=test.img bs=1M count=1000 oflag=direct
    - 读取测试  dd if=/dev/sda1 of=/dev/null bs=1M count=1000 iflag=direct
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
- 硬盘健康状态
  - 机械硬盘健康状态检测 可以使用 SMART（self-monitioring analysis and Reporting Technology system）服务查看默认支持 SAS、SCSI 接口类型硬盘，检测硬盘需要支持 SMART 协议。
    - 自我检测 smartctl -t short /dev/sda
    - 显示磁盘整体信息 smartctl -a /dev/sda

## 备份和恢复

- 备份因素考虑
  - 备份哪些文件：哪些数据或文件对于用户来说是重要的。
    - 基本设置信息、服务的内容数据
    - linux 操作系统需要备份
      /etc/整个目录
      /home/ 整个目录
      /var/spool/mail/
      /var/spoll/{at,cron}
      /boot/
      /root/
    - 服务数据
      - WWW服务 /var/www
      - MariaDB /var/lib/mysql 整个目录
  - 选择什么备份媒介（考虑点 成本、存储时间）
    - 磁带（消磁、发霉）
    - 异地备份系统
    - CD、DVD、NAS等
  - 备份的方式
    - 完整备份（Full backup）
      - 累计备份
        - 系统在进行完第一次完整备份后，经过一度时间点运行，比较系统与备份之间的差异，仅备份有差异的文件。而第二次累计备份则与第一次累计备份的数据比较，也是进备份有差异的数据。
        - 数据恢复：比较麻烦，需要从第一次备份依次还原
        - 例子
          完整备份 xfsdump -l 0 -L 'full' -M 'full' -f /backup/dump.home /home
          第一次累计备份 xfsdump -l 0 -L 'full-1' -M 'full-1' -f /backup/dump.home1 /home
      - 差异备份
        - 系统进行第一完整备份后，每次的备份都是与原始的完整备份比较的结果
        - 可以使用 rsync
    - 部分备份
  - 备份的频率
  - 备份使用的工具
- 灾难恢复
  - 备份是方式系统掉数据丢失，如果系统挂掉如何恢复
  - 硬件问题
    - 定位损坏硬件，更换相应硬件
    - 直接将完整的系统恢复即可
  - 软件问题（发生信息安全事件）
    - 断网，最好将系统完整备份到其他媒介上，已备未来查验
    - 开始查看日志文件，尝试找出各种可能的问题
    - 开始安装新系统（最好找最新的发型版）
    - 进行系统的升级，与防火墙规则定制
    - 根据 2 的错误，在安装完新的系统后，修复 BUG
    - 进行各项服务与相关数据恢复
    - 测试服务是否正常，服务上线

## 进程管理

- 进程：运行程序的一组抽象，通过它可以管理和监控内存、处理器时间、I/O资源
  - 进程的组成（一个地址空间以及一组内核数据结构）
    - 地址空间：一系列内存页，由内核标记为进程所有
    - 内核数据结构
      - 进程空间地址映射
      - 进程当前状态（睡眠、停止、可运行等）
      - 进程的执行优先级，实时优先级：0-99 （高优先级）普通优先级：100-139（低优先级）默认低优先级为 120
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
    - 优先级查看 ps -eo pid,comm,pri,nice
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
      -c 显示进程完整路径
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
      - 第一行信息：
        top - 09:03:03 up 11:38,  2 users,  load average: 0.00, 0.00, 0.00
              时间        运行时间  登陆用户    系统负载，任务队列的平均长度（一段时间内处于可运行状态和不可中断状态的进程平均数量。）
      - 第二行信息：
        Tasks: 204 total,   1 running, 203 sleeping,   0 stopped,   0 zombie
              总进程数       正在运行的       休眠的          停止的       僵尸进程数
      - 第三行信息：
        %Cpu(s):  0.2 us,  0.2 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
              用户空间占用  内核空间占用  nice值  空闲 CPU  等待输入输出  
      - 第四行信息
        MiB Mem :   3911.4 total,   2664.4 free,    248.9 used,    998.1 buff/cache
                   物理内存使用          空闲          已使用        buffer/cache 使用
      - 第五行信息
        MiB Swap:      0.0 total,      0.0 free,      0.0 used.   3470.9 avail Mem
                      swap 总大小       swap 剩余      swap 使用的   swap 可用
      - 动态进程字段
         PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
         进程id  用户   优先级 nice 虚拟内存  进程使用未被  共享内存 进程状态 cpu使用率 内存使用率  启动时间 执行命令
         nice 不是 priority 但是可以影响优先级（范围 -20-19），nice值越低，说明进程越不nice，抢占cpu的能力就越强，优先级就越高。静态优先级
  - 进程优先级调整 renice
    - 基本语法： renice priority [-p] pid
    - 改变一个进程优先级 renice 10 -p 124
    - 改变一个进程组 renice -n 10  -g 4
  - 后台运行 nohup （运行程序的信息不输出到终端，输出内容会写入当前目录的 nohup.out）
    - nohup \<command> & 命令执行放入后台
    - nohup \<command> > nhup.out 不显示执行结果，重定向
  - bg 将作业放到后台运行
  - fg jobid 将后台作业放到前台展示
  - job 显示当前终端后台运行程序
  - init （ /etc/inittab 根据这个配置文件创建 Linux 进程 ）系统初始化工具，是所有 linux 进程的父进程，进程 ID 是1
    - 0 停机
    - 1 单用户模式
    - 2 多用户模式，没有 NFS
    - 3 完全多用户模式（标准的运行级）
    - 4 没有用到
    - 5 x11
    - 6 重新启动
  - runlevel 显示当前运行级别

## 内存管理

- 系统内核为每个进程定义了地址空间，创建了一种假象；进程拥有一块无限的连续的内存区域。实际上，不同进程的内存页面全部都是胡乱堆放在系统的物理内存中。只有内核的记账（bookkeeping）和内存保护机制才能分清楚它们。

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
- netstat 显示网络连接、接口状态、伪接、网络链路信息和组播成员
  - 显示当前路由表 netstat -rn
  - 显示网络接口情况 netstat -i
- ss （iproute包）CentOS7 替代netstat 命令，用于查看网络连接状态，
  - 显示所有 socket 连接 ss -an
  - 格式化输出 ss -an ｜ column -t
  - 显示正在监听的 TCP 和 UDP 连接 ss-tupnl ｜ column -t
  - 统计服务器连接数 ss -s
- telnet 远程登录，一般用来测试 TCP 口
- wget
- curl
- nslookup 域名查询（bind-utils）
  - nslookup www\.baidu\.com
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

## BASH

- SHELL ：用户和内核对话界面。该环境只有一个命令提示符，让用户输从键盘入指令，也称为命令行环境（CLI command line interface）
- SHELL种类
  - sh
  - csh
  - Bourne Again shell（bash）
  - Z Shell（zsh）macos 默认
  - 查看当前默认的 SHELL echo $SHELL
  - 查看当前系统安装的所有 SHELL cat /etc/shells
  - 找到 某个 SHELL 可执行文件位置 which fish
  - 修改默认 SHELL chsh -s /usr/bin/fish
- 基本语法
  - echo 命令的作用是在屏幕输出一行文本，可以将该命令的参数原样输出。
    - -n 参数echo输出的文本末尾会有一个回车符。-n参数可以取消末尾的回车符，使得下一个提示符紧跟在输出内容的后面。
    - -e 参数会解释引号（双引号和单引号）里面的特殊字符（比如换行符\n）。如果不使用-e参数，即默认情况下，引号会让特殊字符变成普通字符，echo不解释它们，原样输出。
  - 命令格式 command [ arg1 ... [ argN ] ]
  - 命令参数使用 空格或tab 进行区分
  - 分号是命令的结束符，使得一行可以放置多个命令，上一个命令执行结束后，再执行第二个命令。
  - 命令组合符号 || （command1 || command2 第一个命令失败也运行第二个）和 &&（command1 && command2 第一个命令成功运行第二个）
  - type 判断命令来源
    - -a 查看一个命令的所有定义
    - -t 可以返回一个命令的类型：别名（alias），关键词（keyword），函数（function），内置命令（builtin）和文件（file）
  - 快捷键
    Ctrl + L：清除屏幕并将当前行移到页面顶部。
    Ctrl + C：中止当前正在执行的命令。
    Shift + PageUp：向上滚动。
    Shift + PageDown：向下滚动。
    Ctrl + U：从光标位置删除到行首。
    Ctrl + K：从光标位置删除到行尾。
    Ctrl + W：删除光标位置前一个单词。
    Ctrl + D：关闭 Shell 会话。
    ↑，↓：浏览已执行命令的历史记录。
- 模式扩展
  - Shell 接收到用户输入的命令以后，会根据空格将用户的输入，拆分成一个个词元（token）。然后，Shell 会扩展词元里面的特殊字符，扩展完成后才会调用相应的命令。这种特殊字符的扩展，称为模式扩展（globbing）。其中有些用到通配符，又称为通配符扩展（wildcard expansion）。Bash 一共提供八种扩展。
  - 关闭扩展  set -o noglob 或者 set -f
  - 开启扩展  set +o noglob 或者 set +f
  - 波浪线扩展：
    - 波浪线~会自动扩展成当前用户的主目录。
    - 扩展当前目录 echo ~+
  - ？字符扩展: ?字符代表文件路径里面的任意单个字符，不包括空字符。
    - 匹配单个字符 ls ?.txt
    - 匹配多个字符 ls ??.txt
  - \* 字符扩展:*字符代表文件路径里面的任意数量的任意字符，包括零个字符。
    - 显示所有隐藏文件 echo .*
    - 匹配子目录的文件 ls */*.txt
  - 方括号扩展：[...]只有文件确实存在才会扩展。
    - 括号之中的任意一个字符。比如，[aeiou]可以匹配五个元音字母中的任意一个。
    - [^...]和[!...] 它们表示匹配不在方括号里面的字符
    - 如果需要匹配 [  可以放在方括号内
    - 如果需要匹配 - （连字符）只能在方括号的开始或结尾 比如[-aeiou]或[aeiou-]。
  - [start-end] 扩展: 表示匹配一个连续的范围。比如，[a-c]等同于[abc]，[0-9]匹配[0123456789]。
    - [a-z]：所有小写字母。
    - [a-zA-Z]：所有小写字母与大写字母。
    - [a-zA-Z0-9]：所有小写字母、大写字母与数字。
    - [abc]*：所有以a、b、c字符之一开头的文件名。
    - program.[co]：文件program.c与文件program.o。
    - BACKUP.\[0-9]\[0-9][0-9]：所有以BACKUP.开头，后面是三个数字的文件名。
  - 大括号扩展: 大括号扩展{...}表示分别扩展成大括号里面的所有值，各个值之间使用逗号分隔。比如，{1,2,3}扩展成1 2 3。
  - {start..end} 扩展: 大括号扩展有一个简写形式{start..end}，表示扩展成一个连续序列。比如，{a..z}可以扩展成26个小写英文字母。
    - 正序 echo d{a..d}g // dag dbg dcg ddg
    - 逆序 echo {c..a}  // c b a
  - 变量扩展:将 $ 作为开头的词视为变量
    - echo $SHELL / echo ${SHELL}
    - 返回匹配所有给定字符串的变量名 echo ${!s*}
  - 子命令扩展 $(...)可以扩展成另一个命令的运行结果，该命令的所有输出都会作为返回值。
    - echo $(date)
    - echo `date`
    - $(...)可以嵌套，比如$(ls $(pwd))。
  - 算术扩展: $((...))可以扩展成整数运算的结果
    - echo $(( a + b ))
  - 字符类:POSIX正则 [[:class:]]表示一个字符类，扩展成某一类特定字符之中的一个。常用的字符类如下。
    [[:alnum:]]：匹配任意英文字母与数字
    [[:alpha:]]：匹配任意英文字母
    [[:blank:]]：空格和 Tab 键。
    [[:cntrl:]]：ASCII 码 0-31 的不可打印字符。
    [[:digit:]]：匹配任意数字 0-9。
    [[:graph:]]：A-Z、a-z、0-9 和标点符号。
    [[:lower:]]：匹配任意小写字母 a-z。
    [[:print:]]：ASCII 码 32-127 的可打印字符。
    [[:punct:]]：标点符号（除了 A-Z、a-z、0-9 的可打印字符）。
    [[:space:]]：空格、Tab、LF（10）、VT（11）、FF（12）、CR（13）。
    [[:upper:]]：匹配任意大写字母 A-Z。
    [[:xdigit:]]：16进制字符（A-F、a-f、0-9）。
  - 使用注意
    - 通配符先解释后执行。 ls a*.txt
    - 文件名扩展在不匹配时，会原样输出。
    - 文件名可以使用通配符。 touch 'fo*'
  - 量词语法：量词语法用来控制模式匹配的次数。它只有在 Bash 的extglob参数打开的情况下才能使用，不过一般是默认打开的。下面的命令可以查询。
    - Bash 的extglob参数是否打开 shopt extglob
    - 开启某个参数 shopt -s extglob
    - 关闭某个参数 shopt -u extglob
    - 量词语法
      ?(pattern-list)：模式匹配零次或一次。
      *(pattern-list)：模式匹配零次或多次。
      +(pattern-list)：模式匹配一次或多次。  ls abc+(.txt) // 例子显示 abc.txt abc.txt.txt
      @(pattern-list)：只匹配一次模式。 ls abc@(.txt|.php) // 例子显示 abc.php abc.txt
      !(pattern-list)：匹配给定模式以外的任何内容。
  - shopt 命令：命令可以调整 Bash 的行为。它有好几个参数跟通配符扩展有关。
    - dotglob 参数可以让扩展结果包括隐藏文件（即点开头的文件） ls *
    - nullglob 参数可以让通配符不匹配任何文件名时，返回空字符。
    - failglob 参数使得通配符不匹配任何文件名时，Bash 会直接报错，而不是让各个命令去处理。
    - extglob 参数 参数使得 Bash 支持 ksh 的一些扩展语法。
    - nocaseglob 参数 可以让通配符扩展不区分大小写。
- 引号和转义
  - bash 只有一种数据类型就是字符串，不管用户输入什么数据都将视为字符串。
  - 转义：某些字符在 bash 中有特殊含义（$ & *）需要添加 \ 进行转义变成普通字符
    - 需要转义的字符 ! " # & ' ( ) , ; < > [ | \ ] ^ { } ` $ * ?
    - 反斜杠除了转义还可以表示一些不可打印的字符
      \a 响铃
      \b 退格
      \n 换行
      \r 回车
      \t 制表符
  - 单引号：允许字符串放入单引号或双引号中使用，单引号用于保留字符串的字面含义，各种特殊字符在单引号中，都会变成普通字符，比如 *、$等。单引号中要想使用单引号使用反斜缸转义还需要在单引号前增加 $。比如 echo $'it\'s'。更合理的方式是使用双引号。
  - 双引号：双引号比单引号宽松，大部分特殊字符在双引号里面，都会失去特殊含义，变成普通字符。三个特殊字符除外：美元符号（$）、反引号（`）和反斜杠（\）。这三个字符在双引号之中，依然有特殊含义，会被 Bash 自动扩展。美元符号用来引用变量，反引号则是执行子命令。反斜杠用来转义
  - Here 文档：一种输入多行字符串的方式格式如下
    << token
    text1
    text2
    token
    它的格式分成开始标记（<< token）和结束标记（token）。开始标记是两个小于号 + Here 文档的名称，名称可以随意取，后面必须是一个换行符；结束标记是单独一行顶格写的 Here 文档名称，如果不是顶格，结束标记不起作用。两者之间就是多行字符串的内容。
  - Here 字符串：Here 文档的一个变体，使用三个小于号（<<<）表示。
    $ cat <<< 'hi there'
    \# 等同于
    $ echo 'hi there' | cat
- 变量 分成环境变量和自定义变量两类
  - 环境变量：是 Bash 环境自带的变量，进入 Shell 时已经定义好了，可以直接使用。它们通常是系统定义好的，也可以由用户从父 Shell 传入子 Shell。是只读的，可以视为常量。由于它们的变量名全部都是大写，所以传统上，如果用户要自己定义一个常量，也会使用全部大写的变量名。
    - 显示所有环境变量 env 或 printenv
    - 常见环境变量
      BASHPID：Bash 进程的进程 ID。
      BASHOPTS：当前 Shell 的参数，可以用shopt命令修改。
      DISPLAY：图形环境的显示器名字，通常是:0，表示 X Server 的第一个显示器。
      EDITOR：默认的文本编辑器。
      HOME：用户的主目录。
      HOST：当前主机的名称。
      IFS：词与词之间的分隔符，默认为空格。
      LANG：字符集以及语言编码，比如zh_CN.UTF-8。
      PATH：由冒号分开的目录列表，当输入可执行程序名后，会搜索这个目录列表。
      PS1：Shell 提示符。
      PS2： 输入多行命令时，次要的 Shell 提示符。
      PWD：当前工作目录。
      RANDOM：返回一个0到32767之间的随机数。
      SHELL：Shell 的名字。
      SHELLOPTS：启动当前 Shell 的set命令的参数，参见《set 命令》一章。
      TERM：终端类型名，即终端仿真器所用的协议。
      UID：当前用户的 ID 编号。
      USER：当前用户的用户名。
  - 自定义变量：是用户在当前 Shell 里面自己定义的变量，仅在当前 Shell 可用。一旦退出当前 Shell，该变量就不存在了。
    - set 显示所有变量
    - 创建变量
      - 变量命名规则
        字母、数字和下划线字符组成。
        第一个字符必须是一个字母或一个下划线，不能是数字。
        不允许出现空格和标点符号。
      - 变量声明： variable=value 如果变量的值包含空格，则必须将值放在引号中。myvar="hello world"
    - 读取变量
      - 直接读取 变量前加 $ 或 ${变量名}。 echo $SHELL ${PWD}
      - 变量本身也是变量读取 ${!变量名}
    - 删除变量：unset命令用来删除一个变量，变量设置为空字符串。不存在的变量一律等于空字符串
    - 输出变量：用户创建的变量仅可用于当前 Shell，子 Shell 默认读取不到父 Shell 定义的变量。为了把变量传递给子 Shell，需要使用export命令。这样输出的变量，对于子 Shell 来说就是环境变量。 NAME=foo export NAME
    - 特殊变量：bash 提供一些特殊变量，用户不能进行赋值。
      $? 返回上一个命令的返回值
      $$ 显示当前 SHELL pid 等价于 $BASHPID
      $_ 为上一个命令的最后一个参数。
      $! 为最近一个后台执行的异步命令的进程 ID。
      $0 为当前 Shell 的名称（在命令行直接执行时）或者脚本名（在脚本中执行时）。
      $- 为当前 Shell 的启动参数。
      $# 表示脚本的参数数量，
      $@ 表示脚本的参数值。
    - 变量默认值：提供四个特殊语法，跟变量的默认值有关，目的是保证变量不为空。
      - ${varname:-word} 如果变量varname存在且不为空，则返回它的值，否则返回word。
      - ${varname:=word} 如果变量varname存在且不为空，则返回它的值，否则将它设为word，并且返回word。
      - ${varname:+word} 如果变量名存在且不为空，则返回word，否则返回空值。
      - ${varname:?message} 如果变量varname存在且不为空，则返回它的值，否则打印出varname: message
    - declare 命令 可以声明一些特殊类型的变量，为变量设置一些限制，比如声明只读类型的变量和整数类型的变量。
      形式 declare OPTION VARIABLE=value
      declare命令如果用在函数中，声明的变量只在函数内部有效，等同于local命令。
      - 主要参数
        -a：声明数组变量。
        -f：输出所有函数定义。 参数输出当前环境的所有函数，包括它的定义
        -F：输出所有函数名。
        -i：声明整数变量。可以直接进行数学运算
        -l：声明变量为小写字母。可以自动把变量值转成小写字母。
        -p：查看变量信息。
        -r：声明只读变量。 可以声明只读变量，无法改变变量值，也不能unset变量。
        -x：该变量输出为环境变量。 -x参数等同于export命令
        -u：声明变量为大写字母。 变量为大写字母，可以自动把变量值转成大写字母。
    - readonly 等同于 declare -r 用来声明只读变量，不能改变其值，也不能 unset
    - let 声明变量时，可以直接执行算术表达式
      let foo=1+2 echo $foo // 结果3
- 字符串操作
  - 获取字符串的长度
    ${#变量名} $#变量名
  - 子字符串,字符串提取子串的语法如下。
    - ${FOO:3} 截取FOO的值的子字符串, 从3到结束
    - ${FOO:0:3} 子串 (位置，长度)
    - ${FOO:(-3):3} 从右边开始的子串, bash 4.2之后才支持负数索引
  - 搜索和替换
    - 字符串头部模式匹配
      \${FOO#word} 从头开始扫描word，将匹配word正则表达的字符过滤掉,#为最短匹配
      \${FOO##word} 从头开始扫描word，将匹配word正则表达的字符过滤掉,##为最长匹配
    - 字符串尾部模式匹配
      \${FOO%word} 从尾开始扫描word，将匹配word正则表达式的字符过滤掉,%为最短匹配
      \${FOO%%word} 从尾开始扫描word，将匹配word正则表达式的字符过滤掉,%%为最长匹配
    - 任意位置匹配
      \${FOO/pattern/string} 替换第一个匹配项
      \${FOO//pattern/string} 替换全部匹配项
      \${FOO/%pattern/string} 替换后缀
      \${FOO/#pattern/string} 替换前缀
  - 改变大小写
    - 转为大写 ${varname^^}
    - 转为小写 ${varname,,}
- 算数运算
  - 算数表达式
    ((...))语法可以进行**整数**的算术运算。
    支持的算数运算符
    +：加法
    -：减法
    *：乘法
    /：除法（整除）
    %：余数
    **：指数
    ++：自增运算（前缀或后缀）
    --：自减运算（前缀或后缀）
  - 数值的进制:BASH 默认十进制，可以使用其他进制
    number：没有任何特殊表示法的数字是十进制数（以10为底）。
    0number：八进制数。
    0xnumber：十六进制数。
    base#number：base进制的数。
    echo $((0xff))
  - 位运算
    \$((...))支持以下的二进制位运算符。
    <<：位左移运算，把一个数字的所有位向左移动指定的位。
    >>：位右移运算，把一个数字的所有位向右移动指定的位。
    &：位的“与”运算，对两个数字的所有位执行一个AND操作。
    |：位的“或”运算，对两个数字的所有位执行一个OR操作。
    ~：位的“否”运算，对一个数字的所有位取反。
    ^：位的异或运算（exclusive or），对两个数字的所有位执行一个异或操作。
  - 逻辑运算
    \$((...))支持以下的逻辑运算符。
    <：小于
    \>：大于
    <=：小于或相等
    \>=：大于或相等
    ==：相等
    !=：不相等
    &&：逻辑与
    ||：逻辑或
    !：逻辑否
    expr1?expr2:expr3：三元条件运算符。若表达式expr1的计算结果为非零值（算术真），则执行表达式expr2，否则执行表达式expr3。
  - 赋值运算
    \$((...))支持的赋值运算符，有以下这些。
    parameter = value：简单赋值。
    parameter += value：等价于parameter = parameter + value。
    parameter -= value：等价于parameter = parameter – value。
    parameter *= value：等价于parameter = parameter \* value。
    parameter /= value：等价于parameter = parameter / value。
    parameter \%= value：等价于parameter = parameter \% value。
    parameter <<= value：等价于parameter = parameter << value。
    parameter >>= value：等价于parameter = parameter >> value。
    parameter &= value：等价于parameter = parameter & value。
    parameter |= value：等价于parameter = parameter | value。
    parameter ^= value：等价于parameter = parameter ^ value。
  - 求值运算：在$((...))内部是求值运算符，执行前后两个表达式，并返回后一个表达式的值。
    echo $((foo = 1 + 2, 3 * 4))
  - expr 命令 expr命令支持整数的算术运算，可以不使用((...))语法。
    expr $foo + 2
  - let 命令 let命令用于将算术运算的结果，赋予一个变量。 let x=2+3  echo $x
- 操作历史
  Bash 会保留用户的操作历史，即用户输入的每一条命令都会记录，默认是保存最近的500条命令。有了操作历史以后，就可以使用方向键的↑和↓，快速浏览上一条和下一条命令。
  退出当前 Shell 的时候，Bash 会将用户在当前 Shell 的操作历史写入~/.bash_history文件，该文件默认储存500个操作。
  历史命令记录文件 echo $HISTFILE
  - history 命令
    history 默认展示后10条命令
    history -c 清空历史操作命令
  - 环境变量
    HISTTIMEFORMAT 历史记录格式
    HISTSIZE  设置保存历史操作的数量
    HISTIGNORE 配置那些命令不记录到历史中
  - ctrl + r 搜索历史命令
  - ! + 行号，样式 !8 执行历史命令第几行的命令
  - ！- 行号, 样式 !-3 执行历史命令倒数第几行的命令
  - !! 命令返回上一条命令
  - ！+ 搜索词 可以快速匹配命令
  - !? + 搜索词 可以搜索命令的任意部分，包括参数部分。它跟! + 搜索词的主要区别是，后者是从行首开始匹配。
  - !$ 上一个命令的最后一个参数, !* 上一个命令的所有参数
  - !:p 执行输出上一条命令不想执行
  - ^string1^string2 ^string1^string2用来执行最近一条包含string1的命令，将其替换成string2。
  - histverify 参数 如果希望增加一个确认步骤，先输出是什么命令，让用户确认后再执行，可以打开 Shell 的histverify选项。shopt -s histverify
  - 快捷键
    Ctrl + p：显示上一个命令，与向上箭头效果相同（previous）。
    Ctrl + n：显示下一个命令，与向下箭头效果相同（next）。
    Alt + <：显示第一个命令。
    Alt + >：显示最后一个命令，即当前的命令。
    Ctrl + o：执行历史文件里面的当前条目，并自动显示下一条命令。这对重复执行某个序列的命令很有帮助。
- 行操作
  Bash 内置了 Readline 库，具有这个库提供的很多“行操作”功能，比如命令的自动补全，可以大大加快操作速度。
  这个库默认采用 Emacs 快捷键，也可以改成 Vi 快捷键。
  修改位 vi 快捷键 set -o vi
  - 光标移动
    Ctrl + a：移到行首。
    Ctrl + b：向行首移动一个字符，与左箭头作用相同。
    Ctrl + e：移到行尾。
    Ctrl + f：向行尾移动一个字符，与右箭头作用相同。
    Alt + f：移动到当前单词的词尾。
    Alt + b：移动到当前单词的词首。
    Ctrl + l 清空屏幕
  - 编辑操作
    Ctrl + d：删除光标位置的字符（delete）。 如果没有任何字符会退出当前shell
    Ctrl + w：删除光标前面的单词。
    Ctrl + t：光标位置的字符与它前面一位的字符交换位置（transpose）。
    Alt + t：光标位置的词与它前面一位的词交换位置（transpose）。
    Alt + l：将光标位置至词尾转为小写（lowercase）。
    Alt + u：将光标位置至词尾转为大写（uppercase）。
  - 自动补全
    命令输入到一半的时候，可以按一下 Tab 键，Readline 会自动补全命令或路径。alt 键可以使用 esc 键替换
    Tab：完成自动补全。
    Alt + ?：列出可能的补全，与连按两次 Tab 键作用相同。
    Alt + /：尝试文件路径补全。
    Ctrl + x /：先按Ctrl + x，再按/，等同于Alt + ?，列出可能的文件路径补全。
    Alt + !：命令补全。
    Ctrl + x !：先按Ctrl + x，再按!，等同于Alt + !，命令补全。
    Alt + ~：用户名补全。
    Ctrl + x ~：先按Ctrl + x，再按~，等同于Alt + ~，用户名补全。
    Alt + $：变量名补全。
    Ctrl + x $：先按Ctrl + x，再按$，等同于Alt + $，变量名补全。
    Alt + @：主机名补全。
    Ctrl + x @：先按Ctrl + x，再按@，等同于Alt + @，主机名补全。
    Alt + *：在命令行一次性插入所有可能的补全。
    Alt + Tab：尝试用.bash_history里面以前执行命令，进行补全。
  - 其他快捷键
    Ctrl + j：等同于回车键（LINEFEED）。
    Ctrl + m：等同于回车键（CARRIAGE RETURN）。
    Ctrl + o：等同于回车键，并展示操作历史的下一个命令。
    Ctrl + v：将下一个输入的特殊字符变成字面量，比如回车变成^M。
    Ctrl + [：等同于 ESC。
    Alt + .：插入上一个命令的最后一个词。
    Alt + _：等同于Alt + .。
- 目录堆栈
  方便用户在不同目录之间进行切换
  - cd - 返回前一次记录的目录
  - pushd，popd 记忆多重目录
    第一次使用pushd命令时，会将当前目录先放入堆栈，然后将所要进入的目录也放入堆栈，位置在前一个记录的上方。以后每次使用pushd命令，都会将所要进入的目录，放在堆栈的顶部。
  - dirs 命令
    dirs命令可以显示目录堆栈的内容，一般用来查看pushd和popd操作后的结果。
    参数
    -c：清空目录栈。
    -l：用户主目录不显示波浪号前缀，而打印完整的目录。
    -p：每行一个条目打印目录栈，默认是打印在一行。
    -v：每行一个条目，每个条目之前显示位置编号（从0开始）。
    +N：N为整数，表示显示堆顶算起的第 N 个目录，从零开始。
    -N：N为整数，表示显示堆底算起的第 N 个目录，从零开始。
- 脚本入门
  脚本（script）就是包含一系列命令的一个文本文件。Shell 读取这个文件，依次执行里面的所有命令，就好像这些命令直接输入到命令行一样。所有能够在命令行完成的任务，都能够用脚本完成。
  脚本的好处是可以重复使用，也可以指定在特定场合自动调用，比如系统启动或关闭时自动执行脚本。
  - Shebang 行
    脚本的第一行通常用于指定解释器，即这个脚本必须通过什么解释器执行。这一行以#!字符开头，这个字符称为 Shebang，所以这一行就叫做 Shebang 行。
    #!/bin/sh 或 #!/bin/bash
    #!/usr/bin/env bash 通用写法
  - 执行权限和路径
    只要指定了 Shebang 行的脚本，可以直接执行。这有一个前提条件，就是脚本需要有执行权限。可以使用下面的命令，赋予脚本执行权限。
    所有用户添加执行权限 chmod +x script.sh chmod 755 script.sh
  - env 命令
    env命令总是指向/usr/bin/env文件，或者说，这个二进制文件总是在目录/usr/bin。
  - 注释
  - 脚本参数
  - shift 命令
  - getopts 命令
  - 配置项参数终止符 -- 
  - exit 命令
  - 命令执行结果
  - source
  - 别名，alias 别名



## 安全

- 安全要素
- 基本安全措施
- 强力安全工具
- 密码学入门
- 防火墙
  - firewalld
    通过将网络划分为不同区域，制定不同区域之间的访问控制策略
    zero区域       策略
    drop（丢弃）    任何接收的⽹络数据包都被丢弃，没有任何回复。 仅能有发送出去的⽹络连接。
    block（限制）   任何接收的⽹络连接都被 IPv4 的 icmp-host-prohibited 信息和 IPv6 的 icmp6-adm-prohibited 信 息所拒绝。
    public（公共）  在公共区域内使⽤，不能相信⽹络内的其他计算机 不会对您的计算机造成危害，只能接收经过选取的 连接。
    externel（外部）特别是为路由器启⽤了伪装功能的外部⽹。您不能信任来⾃⽹络的其他计算，不能相信它们不会对您 的计算机造成危害，只能接收经过选择的连接。
    dmz（非军事区） ⽤于您的⾮军事区内的电脑，此区域内可公开访 问，可以有限地进⼊您的内部⽹络，仅仅接收经过 选择的连接。
    work（工作区）  ⽤于⼯作区。您可以基本相信⽹络内的其他电脑不 会危害您的电脑。仅仅接收经过选择的连接。
    home（家庭）    ⽤于家庭⽹络。您可以基本信任⽹络内的其他计算 机不会危害您的计算机。仅仅接收经过选择的连 接。
    internel（内部）⽤于内部⽹络。同于home
    trusted（信任） 可接受所有的⽹络连接。
  - iptable
    linux 上应用程序，用于管理 Netfilter 。Netfilter 是 Linux 内核空间的程序代码，在 Linux 内核中实现了防火墙
    iptables会从上⾄下的读取防⽕墙规则，找到匹配的规则后，就结束匹配 ⼯作，并且执⾏对应的动作。 如果读取所有的防⽕墙规则都没有符合的，就执⾏默认的规则。 规则⼀般两种：允许、拒绝。
    - 使用条件
      sudo 用户权限
      拥有终端
    - Netfilter 数据包传输内置三个过滤器链
      - INPUT
      - OUTPUT
      - FORWARD
    - iptable 组成（五链四表）
      - tables 表是将类似规则分组的文件。一个表格由几个规则链组成。
        四个默认表管理不同的 rule chains
        - Filter 默认数据包过滤表。它充当门禁，决定哪些数据包进入和离开网络。
        - NAT 包含将数据包路由到远程网络的NAT规则。它用于需要更改的数据包。
        - Mangle 调整IP数据包内容
        - RAW 用于处理异常，包括的规则链有：prerouting，output。一般使用不到。
      - chains 链条是一串规则。当收到数据包时，iptables会找到适当的表，并通过规则链进行过滤，直到找到匹配项。
        - INPUT
          处理目的地为本地应用程序或服务的传入数据包。
        - OUTPUT
          管理本地应用程序或服务上生成的传出数据包。
        - FORWARD
          适用于通过系统从一个网络接口到另一个网络接口的数据包。
        - PREROUTING
          在路由之前更改数据包。更改发生在路由决定之前。
        - POSTROUTING
          路由后更改数据包。更改发生在路由决定之后。
      - tables 对应的 chains
        - rules 规则
          定义匹配数据包的条件，捕获数据包后发送到 targets
        - targets 目标
          对数据包采取的动作（接收、丢弃、驳回）
          - ACCEPT 允许数据包通过防火墙
          - DROP 在通知发送者的情况下丢弃数据包
          - REJECT 通知发送者错误下丢弃数据包
          - LOG 记录数据包信息到日志文件中
          - SNAT 源地址转换
          - DNAT 目标地址转换
          - MASQUERADE 为动态分配的IP更改数据包的源地址。
      - 安装
        - deb 包管理
          apt install iptables
          apt install iptables-persistent
          systemctl enable netfilter-persistent
        - rpm 包管理
          yum install iptables
          yum install iptables-services
         systemctl enable iptables
      - 常用命令
        - 查看规则
        iptables -L
        - 允许换回地址通信
        iptables -A INPUT -i lo -j ACCEPT
        - 允许访问 http 服务
        iptables -A INPUT -p tcp --dport 80 -j ACCEPT
        - 允许某个 IP 访问
        iptables -A INPUT -s [IP-address] -j ACCEPT
        - 记录
        iptables -A INPUT -j LOG --log-prefix "Dropped: "
        - 删除某个规则
        iptables -L --line-numbers
        - 移除所有规则链
        iptables -F
        - 服务器拒绝 icmp 流量
        iptable -A INPUT -p icmp --icmp-type 8 -s 0/0 -j REJECT
        - 禁止访问本机 80 端口
        iptable -A INPUT -p tcp --drop 80 -j REJECT
        - 保存配置
          deb
          netfilter-persistent save
          rpm
          service iptables save
- selinux （redhat 系列用）
    工作模式：
enforce： SElinux 根据安全策略，积极阻止有潜在风险的操作
permission：仅记录会被阻止的操作，
disable：selinux 禁用，日志也不记录
	AppArmor （debin 系列用）
- VPN
  - OpenVPN
  - WireGuard
- 专业认证与标准
- 安全信息来源
- 安全问题处理

## 日志管理

- 日志管理流程
  - 从各种源头收集日志
  - 为消息的查询、分析、过滤、监视提供结构化接口
  - 管理消息的保留和日期，以便消息在发挥作用或合法期间可以尽可能久地保留，遵循相关规范。
- 常见日志
  - apache2/*  Apache httpd 的日志
  - /var/log/apt*  APT 软件包安装日志
  - /var/log/auth.log sudo 授权日志
  - /var/log/boot.log 系统启动脚本输出
  - /var/log/cloud-init.log cloud-init 初始化脚本
  - /var/log/cron,cron/log  cron的执行情况和出现的错误
  - /var/log/daemon.log 所有的守护进程信息
  - /var/log/debug* 调试过程输出
  - /var/log/dmesg 内核消息缓冲区
  - /var/log/dpkg.log 软件包管理日志
  - /var/log/faillog 失败的登陆尝试
  - /var/log/kern.log 所有的内核功能消息
  - /var/log/lastlog login 每个用户最后一次登陆的时间
  - /var/log/mail* mail相关 
  - /var/log/messages 主要的系统日志文件
  - /var/log/samba* Samba服务相关
  - /var/log/secure sshd 私有授权消息
  - /var/log/syslog* 主要的系统日志文件 
  - /var/log/wtmp 登陆记录
  - /var/log/xen/* xen 相关日志
  - /var/log/Xorg.n.log X window 服务错误
  - /var/log/yum.log 软件包管理日志
- 日志文件会增长，最好单独挂载，进行日志切割。
- systemd journal 日志
  - systemd 包含了一个叫做 systemd-journald 的日志记录守护进程。它复制了 syslog 的大部分功能，但仍能与 syslog 和平共处（取决于如何配置）
  - 文件格式：二进制
  - 信息是收集位置
    - 通过套接字 /dev/log (软连接实际 /run/systemd/journal/dev-log)
    - 通过设备文件 /dev/kmsg (字符设备) 收集有 linux kernel 产生的日志文件
    - UNIX 套接字 /run/systemd/stdout 为通过标准输出写入日志消息的软件服务
    - UNIX 套接字 /run/systemd/socket 为通过 systemd journal API 提交日志消息的软件服务
    - 内核守护进程 auditd 的审计消息
  - 有的管理员使用 systemd-journal-remote 实用工具 （systemd-journal-upload system-journal-gateway）将序列化的日志消息通过网络发送到远端的 journal
  - 配置 systemd journal
    systemd journal 默认配置文件是 /etc/systemd/journald.conf 该文件并不能直接编辑，需要将定制好的配置放入目录 /etc/systemd/journald.conf.d/ 所有以 .conf 为结尾的文件都会被自动加载，默认的 journal.conf 中包含的所有选项都处于注释状态，而且都已经设置好了默认值，因此你一眼就可以看出哪些选项可以使用。其中包括日志的最大容量、消息的留存期以及各种各种速率限制设置。
    Storage 选项控制着是否将内存保存到磁盘。
    - volatile 日志保存在内存中
    - persistent 将日志保存到 /var/log/journal 如果该目录不存在则自动创建。
    - auto 将日志保存在 /var/log/journal 但不会自动创建目录（默认选项）
    - none 丢弃所有的日志数据
  - 常用命令
    - 日志查看 journalctl -u ssh
    - 实施读取日志文件 journalctl -f \<service>
    - 显示日志所占用的磁盘空间 journalctl --disk-usage
    - 显示一个带有数字标识符的系统引导顺序表。 journalctl --list-boots
    - 显示自昨日午夜开始，直到现在的所有信息 journalctl --since=yesterday --until=now
    - 显示特定程序最近的 100 条日志 journalctl -n 100 /usr/sbin/sshd
- syslog
  - 是一套全面的日志记录系统，也是 IETF 标准的日志记录协议。它有两个重要的功能：将程序员从各种写入日志文件的枯燥方法中解脱出来，管理员控制日志记录的权利。在 syslog 出现之前，每个程序都可以拥有自己的日志记录策略。系统管理员无法做到统一控制该保存哪些信息或保存到哪里。syslog 用法灵活。允许管理员按照源（功能）和重要性（安全级别）对消息进行分类并将其引向不同的目的地：日志文件、用户终端，甚至是其他机器。linux 系统中最初是 syslogd 守护进程现已更新为 rsyslogd。rsyslog 扩展了原来的功能同时保持了 API 向后兼容性。
  - rsyslog 架构
    - 事件流和处理事件流的引擎
    - 日志记录文件 /var/log/syslog
    - 配置文件 /etc/rsyslog.conf
    - rsyslogd 进程通常从引导时就开始运行。知晓 syslog 的程序日志将会记录日志记录写入特殊文件 /dev/log，这是一个 Unix 套接字。在没有使用 systemd 系统的常用配置中，rsyslogd 直接从该套接字读取信息，查询配置文件。
  - rsyslog 版本(redhat 7 使用 rsyslogd 7， debin ubuntu 使用rsyslogd 8)
  - rsyslog 配置
    - rsyslog 配置文件中的各行按照出现的先后顺序，自上而下的处理。配置文件的顶部是用于配置守护进程自身的全局属性。指定了要载入哪些输入模块、默认的消息格式、文件的所有权和权限，用于维护 rsyslog 状态的工作目录等设置。
    - 大多数发行版使用遗留指令 $IncludeConfig 以包含来自配置目录中的额外文件（通常是 /etc/rsyslog.d/*.conf）,考虑到顺序的重要性，发行版通过在文件名前放置数字来组织文件。默认 ubuntu 包含如下文件 20-ufw.conf  21-cloudinit.conf
    - rsyslog 3种配置语法
      - 原始 syslog 配置文件格式的行。是以内核日志记录守护进程 sysklogd 命名的。其形式简单有效，但存在一些限制。可用于构建简单的过滤器
      - 遗留的 rsyslog 指令，这些指令均以 $ 开头，其语法源自 rsyslog 的古老版本。
      - RaiinerScript 脚本语法
    - 模块：rsyslog 模块拓展了核心处理引擎的能力，所有的输入（源）和输出（目标）都可以通过模块配置，模块甚至可以解析并修改消息。可以自己使用 c 编写模块
      - 常见输入输出模块
        imjournal 模块是和 systemd journal 集中在一起的
        imuxsock 模块从 UNIX 域套接字读取消息。如果 systemd 不存在，该模块为默认
        imklog 模块知道如何在 linux 和 BSD系统中读取内核消息
        imfile 模块可以将纯文本文件转换成 syslog 消息格式，在导入不具备原生 syslog 支持的软件产生的日志文件
        imtcp 和 imudp 模块可以分别在网络上接收 TCP 和 UDP 形式的日志消息。可以利用这两个模块实现网络的集中式日志记录。
        immark 模块存在，rsyslog 会定期生成时间戳消息。
        omfile 模块可以将消息写入文件。
        omfwd 模块可以通过 TCP 或 UDP 连接日志消息转发给远程 syslog 服务器。
        omkafka 模块是 Apache Kafka 数据流引擎的生产者实现
        omelasticsearch 模块可以直接向 elasticsearch 集群写入日志
        ommysql 模块可以将日志写入 mysql
    - sysklogd 语法
      - 格式  selector  action，facility.level  action
      - syslog 设施名称
        \*  除 mark 的所有设施
        auth 与安全和授权相关的命令
        authpriv 敏感/私有的授权信息
        cron   cron 守护进程
        daemon 系统守护进程
        ftp  ftp守护进程（已废弃）
        kern 内核
        local0-7 8种本地消息
        lpr 行式打印机
        mark 定期产生的时间戳
        mail sendmail、postfix 以及其他和邮件相关的软件
      - syslog 严重级别
        emerg  恐慌（panic）状况，系统不可用
        alert  紧急状况，应立即采取行动
        crit 临界状况
        err  其他错误
        warning 告警信息
        notice  应该调查的事项
        info 信息类消息
        debug 调试
- logrotate 跨平台日志管理与轮转
  - logrotate 包含了一些理针对被管理的日志文件组的规范。出现在日志文件规范之外的选项应用于后续规范。
  - logrotate.conf 选项
    compress 压缩日志
    daily、weekly、monthly 一句特定的调度计划轮替日志文件
    delaycompress 压缩除当前和最近版本之外的其他所有版本
    endscript prerotate 或 postrotate 脚本的结束标记符
    errors emailaddr 将错误提醒发送至指定的 emailaddr
    missingok 如果日志文件不存在，那么不发出抱怨信息
    notifempty 如果日志文件不为空，那么不执行轮替操作
    olddir dir 指定要被放入 dir 中的旧版本日志文件
    postrotate 引入在日志文件被轮替后的脚本
    size logsize 如果日志文件 > logsize 则进行轮替
- 日志记录策略
  - 制定日志策略需要考虑的内容
    - 涉及多少系统和应用
    - 可用的存储基础设施是什么类型
    - 日志要被留存多长时间
    - 那种日志类型的事件重要
  - 大部分应用都需要考虑以下信息
    - 用户名或用户ID
    - 事件成功或失败
    - 网络事件的源地址
    - 日期和时间（来自权威信息 ntp）
    - 添加、修改或删除的敏感数据
    - 事件详情

## 驱动程序和内核

- 内核相关的日常事务
  - 内核提供的接口包括
    - 硬件设备管理与抽象
    - 进程和线程（以及之间通信）
    - 内存管理（虚拟内存和内存空间保护）
    - I/O 设施（文件系统、网络功能、串口等）
    - 内务管理（housekeeping）功能（启动、关机、定时器、多任务等）
  - 内核版本（遵循语义化版本管理）
    - 主版本
    - 次版本
    - 补丁号
- 设备及驱动程序
  - 设备驱动程序是一个抽象层，负责管理系统与特定类型硬件之间的交互，这使得内核的其余部分无需了解具体的细节。设备所理解的硬件命令和内核定义（及使用）的固定编程接口之间的翻译由驱动完成。驱动程序有助于大部分内核保持独立性。
  - 购买硬件时要注意厂商是否提供相应操作系统驱动
  - 设备文件与设备编号
    - 大多数情况下，设备驱动属于内核的一部分，并非用户进程。通过 /dev 无论是 内核空间还是用户空间都可以访问驱动程序，内核会讲文件操作应设为相应的驱动程序代码调用即可。
    - 设备文件管理较难：当系统只支持少数类型的设备时，手动维护设备文件还管的过来。随着设备数量的增长，/dev目录下设备文件的数量越来越多，其中有些文件与当前系统还毫不相干
    - 现代操作系统都会自动管理设备文件。也可以手动管理 mknod 创建设备文件 mknod filename[要创建的设备文件] type[设备文件类型块还是字符] major[主设备号] minor[次设备号]
    - 现代设备文件管理
      - Linux 和 FreeBSD 都实现了设备文件的自动管理。如果检测到新的设备，那么两个系统都会自动创建相应的设备文件。当设备消失时（例如拔掉 U盘），其设备文件也会被删除。由于架构方面的原因，Linux 和 FreeBSD 在创建设备文件的方法各不相同。FreeBSD 中，内核使用一种可以挂载到 /dev 的专用文件系统（devfs）来创建设备。在 Linux 中有一个运行在用户空间的守护进程 udev 负责此项工作。两种系统侦听内核产生的底层事件流，了解设备的出现及消失。
      - 自动化设备管理概要
      组成部分             Linux                FreeBSD
      /dev 文件系统        udev/devtmpfs        devfs
      /dev文件系统配置文件  -                    /etc/devfs.conf /etc/devfs.rules
      设备管理程序守护进程   udevd                devd
      守护进程配置文件      /etc/udev/udev.conf  /etc/devd.conf
      文件系统自动挂载      udevd                devd
  - Linux 设备管理
    - sysfs 技术：Linux 2.6 引入的，由内核实现的一个虚拟的文件系统，提供系统中可用设备及其配置文件和状态的详细信息，内核空间和用户空间都可以访问。sysfs 通常挂载到 /sys 目录，其中的内容包括设备使用的 IRQ 以及排队等待写入磁盘控制器的数据块数。
    - /sys 子目录
      block  有关块设备的信息
      bus    内核所知的各种总线 PCI-E、SCSI、USB等
      class  按照设备功能类型组织成的目录
      dev    按照字符设备和块设备划分的设备信息
      devices 所有以发现设备的分层次表达模型
      fireware 特定平台子系统（ACPI）的接口
      fs     内核所知的部分文件系统
      kernel 内核的内部信息，例如缓冲区和虚拟内存状态
      module 内核加载动态模块
      power 系统电源状态信息
    - 设备配置信息之前都在 /proc 文件系统中。
    - udevadm 可以查询设备信息、触发事件、控制 udevd 守护进程、监视 udev 和内核事件。
      - 常用命令
        - 首个参数 info、trigger、settle、control、monitor、test
        - 显示设备的所有 udev 属性 udevadm info -a -n <设备名字>
      - 配置文件默认规则目录 /lib/udev/rules.d 本地配置目录 /etc/udev/rules.d/ ，修改默认规则，在本地目录创建新的文件就可以忽略或覆盖默认规则文件。配置文件名字格式：nn-description.rules （后缀不能省）配置文件规则采用 match_clause  assign_clause
      - udevd 使用的匹配键
        ACTION  匹配事件类型，例如 add 或 remove
        ATTR{filename}  匹配设备的 sysfs 值
        DEVPATH   匹配特定的设备路径
        DRIVER   匹配设备使用的驱动程序
        ENV{key}  匹配环境变量值
        KERNEL   匹配设备的内核名称
        PROGRAM  执行外部命令；如果返回码为 0，则匹配
        RESULT  匹配上一个 PROGRAM 调用的输出
        SUBSYSTEM  匹配特定的子系统
        TEST{omask}  测试文件是否存在
    - 添加 Linux 驱动设备
      - linux 系统中，设备驱动程序通常以 3 种形式发布
        - 针对特定呢呵版本的补丁
        - 可装载的内核模块
        - 安装脚本或着可安装驱动程序的软件包
      - cd kernel_src_dir； patch -p1 < patch_file
- Linux 内核配置
  - 内核模块文件存放位置 /lib/modules/`uname -r`/  
  - 修改可调整的（动态）内核配置参数
    - 通过 /proc/sys 中的特殊文件在运行期间查看和设置内核选项,并不是所有文件都可以修改，可以查看内核源代码树文档
      - 一些可以调整在内核参数在 /proc/sys 中所对应的文件
      文件               功能
      cdrom/autoclose    挂载时自动关闭 CD-ROM
      cdrom/autoeject    卸载时自动弹出 CD-ROM
      fs/file-max        设置能够打开的文件最大数量
      kernel/ctrl-alt-del 按下 control-alt-delete 时重启系统，对于不安全的终端，也许可以增强安全性
      kernel/panic       如果发生内核恐慌，在重新引导之前需要等待的秒数，如果设置为 0 ，表示不自动重新引导系统
      kernel/printk_ratelimit    设置内核消息之间最少间隔的秒数
      kernel/printk_ratelimit_burst  在 printk_ratelimit 到达之前，可以连续发送的消息数量
      kernel/shmmax      设置共享内存的最大数量
      net/ip*/conf/default/rp_filter  允许 IP 源路由验证
      net/ip*/icmp_echo_ignore_all 如果设置为1，则忽略 ICMP ping
      net/ip*/ip_forward  如果设置为1，则允许 IP 转发
      net/ip*/ip_local_port_range  设置建立连接时使用本地端口范围
      net/ip*/tcp_syncookies  抵御 SYN 洪范攻击
      tcp_fin_timeout   设置等待最后一个 TCP FIN 分组的秒数
      vm/overcommit_memory  控制内存超频行为，如果物理内存无法满足虚拟内存分配请求，内核该如何应对
      vm/overcommit_ratio  如果出现超频可以使用多少物理内存
    - 临时修改 sysctl net.ipv4.ip_forward=0 && sysctl -p
    - 永久修改可以修改文件 /etc/sysctl.conf
  - 重新构建内核（编译源代码）
    - 设置合适的 .conf 文件
    - 将目录切换到 内核源代码目录顶层
    - 执行 make config、make gconfig 或 make menuconfig
    - 执行 make clean
    - 执行 make
    - 执行 make modules_install
    - 执行 make install
  - 将新的驱动程序和模块动态加载到现有内核中（临时）
    - modprobe ip_vs 动态加载 ip_vs 模块
    - lsmod | grep ip_vs  查看模块是否加载成功
    - modprobe -r ip_vs 动态卸载 ip_vs 模块
    - modinfo ip_vs 查看模块信息
  - 将内核模块开机自动加载
    - 通过 modporobe 命令加载的模块仅在当前有效，计算机重启后并不会被加载，如果需要开机自动加载内核模块，需要将 modprobe 命令吸入 /etc/rc.d/rc.local 文件中 rc.local 文件时开机加载, 也可以通过修改 systemd 配置文件
    - 在 /etc/modprobe.d/ 目录下添加配置文件 xxx.conf 添加 option xxx
- 在云中引导其他内核
  - 云实例的引导不同于传统硬件。大多数云供应商都避开了 GRUB，要么用的是修改过的开源引导程序，要么采用某种完全不会使用引导装载程序的方案。在 AWS 中，你需要从一个以 PV-GRUB 作为引导装载程序的基础 Amazon 镜像开始。
- 内核错误
  - 内核崩溃（又叫做内核恐慌）是一件哪怕在正确配置的系统中也会发生的事。可能出现的问题（用户输入错误、硬件故障、内核BUG、设备驱动程序）
  - 软死锁：如果系统处于内核模式超过数秒，就会发生死锁，这使得用户级任务无分运行，该间隔可以配置，通常为 10s 左右。对于进程而言，这段时间已经长到足以错过多个CPU周期，在软死锁期间，只有内核在运行，但它任能处理中断（例如网络接口和键盘中断）数据依旧能够退出系统
  - 硬死锁：和软死锁一样，但是复杂的地方在于大多数处理器中断都不会被处理。硬死锁的症状很明显，检测速度相对较快。但即使是配置正确的系统在某些极端情况下（CPU高负载），也会出现软死锁
  - 恐慌
  - Linux oops

## 打印

- 打印功能依赖于以下几部分
  - 一个负责收集和调度作业的打印“假脱机程序” spool。
  - 能够与假脱机程序交互的用户层面的实用工具（命令行或GUI）这些工具能够将打印作业发送给假脱机程序、查询作业状态（挂起完成）、删除或重新调度作业
    以及配置打印系统其他组成部分。
  - 与打印设备本身交互的后端
  - 一种能让假脱机程序通信并传输打印作业的网络协议
- CUPS 打印
  - CUPS 服务端也是web服务器
  - CUPS客户端可以是 web，也可以是命令行
  - CUPS 服务端口 631
  - 命令
    - lpr foo.pdf 默认打印机打印文件
- CUPS 打印服务器管理
- 故障排查
  - 重启打印守护进程 （cupsd）
  - 日志文件 CUPS 维护了3种日志文件：页面日志、访问日志、错误日志
  - 直接打印连接 直接运行打印机后端，测试打印机的物理连接 /usr/lib/cups/backend/usb

## 常见服务

- 文件共享
  - NFS
  - Samba
  - vsftpd
  - proFTPD
- 版本控制
  - SVN
  - git
- 网络存储
  - iSCSI
- 基础服务
  - 电子邮件
    - SMTP
    - POP3
    - IMAP
  - DNS
  - SSH
  - NTP

## 配置管理

- 配置管理：在网络上实现操作系统管理自动化。管理员编写规范，描述如何配置服务器，然后由配置管理软件依次实现。
  - 传统实现方法是编写一系列脚本，辅以脚本失效时的临时补救方案。随着时间推移，以此管理的系统往往会退化成一对混乱的残骸，充斥着无法可靠再现的软件包版本和配置。
  - 以代码的形式获取所需的状态。在版本系统中跟踪之后的改动和更新，以创建审计线索和参考点。这些代码还可以作为网络的非正式文档。管理员或开发人员可以阅读代码，确定系统配置方式。
  - 大多数配置系统代码的写法都是声明式，而非过程式。只需要描述你想要达到什么样的状态就行了。配置管理系统会按照自己的逻辑对目标系统作出必要的调整。
  - CM系统的工作是将一系列配置规范（即操作）应用于各个机器。操作的颗粒度各异，但通常比较粗糙，足以对应到可能出现在系统管理员待办事项列表中的事项。
- 配置管理的危险
  - 主流开源的 CM 系统在描述模型时使用的词汇并不相同。不同系统之间缺乏一致性和标准性。不同 CM 使用经验无法直接迁移
  - 随着站点的增长，用于支持配置管理系统的基础设施也必须有相应增长，CM 升级也是一件不小的工程
  - 一个站点想要完全实现配置管理，一定程度的运维成熟性和严格性是比不可少的。主机一旦处于 CM 系统控制之下，就绝不能在动手修改，否则就立即退回雪花系统的状态了。
- 配置管理要素
  - 操作和参数
    - 操作：小规模的行为或检查。操作看起来像命令，通常都是 CM 系统本身的实现语言编写的，利用了系统的标准工具和库。
    - 参数：类似命令的参数（执行不同的细分操作）
    - 操作和命令的不同
      - 大多数操作在重复操作后不造成任何问题，幂等性
      - 操作知道自己何时改变了系统的实际状态
      - 操作知道何时需要改变系统状态。如果档期配合符合规范，操作退出，不进行任何改动
      - 操作会将结果报告给 CM 系统。
      - 操作力求跨平台
  - 变量
    - 变量是命名过的纸，会影响到配置如何应用于单个机器。变量通常用于设置参数值并填充配置模版的空白。
    - 注意点
      - 变量通常可在配置库的多个不同位置和上下文中定义
      - 每个定义都有其可见性的作用域。作用域类型因 CM 系统不同而异，有的涵盖单个机器、一组机器、或是特定的一组操作
      - 在特定的上下文中，可以有多个作用域处于活跃状态。作用域能够嵌套，不过更长久的只是共同有效
      - 因为多个作用与都能够为相同的变量赋值，必须要有某种形式冲突解决方案。
  - 事实数据
    - CM 系统检查每个客户端可以确定可描述的事实数据（fact），例如主机网络接口的 IP 地址，操作系统类型等。
  - 变更处理程序
    - 在大多数系统中，只要指定的一组或多组操作报告它已修改了目标系统，处理程序就会运行。处理程序并不知道有关此次改动的具体性质，但由于操作于其他处理程序之间的关联非常具体，所以并不需要其他信息。
  - 绑定
    - 通过将特定的操作集合与特定主机或主机组相关联，完善了基本配置模型
  - 操作集与操作集仓库
    - 操作集：完成特定功能（例如 安装、配置及运行 Web 服务器）的一系列操作
    - CM 厂商维护着公共仓库，其中包含由官方支持以及用户自发贡献的操作集。
  - 环境
    - 把处于配置管理下的客户端隔离成多个世界（传统分类：开发、测试、生产）
  - 客户端清单与注册
    - 托管主机的清单（inventory）可以存在于平面文件或者何时的关系数据库中，在某些情况下，甚至完全可以是动态的。
  - CM 系统所采用的术语
    上文中的术语             Ansible       Salt      Puppet                       Chef
    operation（操作）       task（任务）    state     resource                     resource
    op type（操作类型）      module        function  resource type/provider       provider
    op list（操作列表）      tasks         states    class/manifest               recipe
    parameter（参数）       parameter     parameter  property/attribute          attribute
    binding（绑定）         playbook      top file   classification/declaration  run list
    master host（控制主机）  control       master     master                      server
    client host（客户机）    host          minion     agent、node                 node
    client group           group         nodegroup  role
    variable               variable      variable   parameter/variable          attribute
    fact                   fact          grain      fact                        automatic/attribute
    notification           notification  requisite  notify                      notifies
    handler                handler       state      subscribe                   subscribe
    bundle                 role          formula    module                      cookbook
    bundle repo            galaxy        github     forge                       supermarket
- Ansible
- Salt

## 集群及高可用



## 虚拟化

- 虚拟化
- Linux 虚拟化
- packer
- vagrant

## 持续集成与交付

- CI/CD基础
- 流水线
- Jenkins

## 监控

- 监控概览
  - 监控：在每个系统上线前都将其添加到监控平台，定期执行成套的检查和调教。主动评估指标和趋势，在问题对用户产生影响或事对数据造成威胁前将起找出。
  - 监控目的：保证整个 IT 基础设施按照预期运行，并且可以访问且易于处理的形式编制有助于管理和规划的数据。
  - 监控系统
    - 从感兴趣的系统或设备中收集原始数据
    - 监控平台检查数据，决定何时的操作方法，通常是管理员设置的规则实现的
    - 原始数据有监控系统决定的操作进入后端，完成实际的处理
  - 监控系统做到
    - 仪表化
    - 数据类型
    - 摄入与处理
    - 通知
    - 仪表盘与 UI
- 监控文化
  - 如果有人关心或依赖某个系统或服务，那就必须将起纳入监控。这样服务或用户所依赖的环境中的一切都要处于监控中
  - 如果生产设备、系统或是服务表现出可监控的属性，那么这些属性就应该被监控
- 监控平台
- 数据收集
- 网络监控
- 系统监控
- 应用监控
- 安全监控
- SNMP
- 监控技巧

## 性能分析



## 词汇

- DMI（Desktop Management Interface，桌面管理接口）是一种用于管理和跟踪计算机系统硬件信息的标准规范。
- 守护进程


---
性能分析 《图解性能分析》
