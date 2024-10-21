<!--
 * @Author: wolf-li
 * @Date: 2024-10-20 20:02:02
 * @LastEditTime: 2024-10-21 14:27:44
 * @LastEditors: wolf-li
 * @Description: 
 * @FilePath: /note/src/Nginx/linux内核参数调优.md
 * talk is cheep show me your code.
-->
内核设置

Linux 操作系统修改内核参数有3种方式：

修改 /etc/sysctl.conf 文件，加入配置选项，格式为 key = value ，修改保存后调用 sysctl -p 加载新配置
使用 sysctl 命令临时修改，如： sysctl -w net.ipv4.tcp_mem="379008 505344 758016"
直接修改 /proc/sys/ 目录中的文件，如： echo "379008 505344 758016" > /proc/sys/net/ipv4/tcp_mem
第一种方式在操作系统重启后会自动生效，第二和第三种方法重启后失效