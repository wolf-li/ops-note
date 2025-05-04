<!--
 * @Author: wolf-li
 * @Date: 2025-04-01 09:52:03
 * @LastEditTime: 2025-04-01 10:41:13
 * @LastEditors: wolf-li
 * @Description: 
 * @FilePath: /note/src/Mind/perl.md
 * talk is cheep show me your code.
-->
# Perl

- 概述
  - 20世纪80年代起，在系统管理和文本处理等领域展露头角。Perl 全称 （Practical Extraction and Report Language）自 1987年 由 Larry Wall 创立以来，一直是系统管理员简化日常任务的首选工具。强大的文本处理能力，内置的正则表达式，处理文本数据的利器。
  - linux 环境查看是否安装 which perl, 查看 perl 版本 perl -v
- 基本语法
  - 标量变量： 数据类型的变量可以是数字，字符串，浮点数，不作严格的区分。在使用时在变量的名字前面加上一个 $，表示是标量。`$age = 25;`
  - 数组：数组变量以字符 @ 开头，索引从 0 开始，如：`@arr=(1,2,3)。@name = ("google","runoob", "taobao");`
  - 哈希: 是一个无序的 key/value 对集合。可以使用键作为下标获取值。哈希变量以字符 % 开头。 `%data = ('google', 45);`
  - 