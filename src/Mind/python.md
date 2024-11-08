# Python 教程

## Python 解释器

- 解释器启动
  解释器的操作方式类似 Unix Shell：用与 tty 设备关联的标准输入调用时，可以交互式地读取和执行命令；以文件名参数，或标准输入文件调用时，则读取并执行文件中的 脚本。
  命令行终端 使用 python3 启动 python 解释器
- 传入参数
  使用 sys 模块的 argv 接收变量，import sys 导入模块。
- 交互模式
  这种模式中会显示主提示符，提示输入下一个指令，主提示符通常用 >>> 表示，输入连续行时，显示次要提示符 ...
- 源文件字符编码
  Python 源码文件的编码是 UTF-8。
  声明其他编码 # -*- coding: cp1252 -*-

## Python 速览

- 注释 # 开头
- 计算器
  \+ \- \* / // () % **
- 文本（str 类）定义使用 单引号、双引号、三引号（"""...""",'''...'''）定义（如果字符串内有对应的引号需要转义）,跨行字符串使用三引号定义。
  - 字符串拼接使用 +
  - 字符串支持索引（下标访问）切片操作
  - 字符串长度 len()
  - 字符串不可变
- 列表:可将不同值组合在一起
  - 定义 b = [1, 's', 1.2]
  - 列表内元素可变
  - 添加元素 append() 函数
  - 修改元素 b[0] = 
  - 清空列表 b[:] = []

## Python 控制流

- if 条件分支
  - 格式
    if x < 0:
        x = 0
        print('Negative changed to zero')
    elif x == 0:
        print('Zero')
    elif x == 1:
        print('Single')
    else:
        print('More')
- while
  - 格式
    while 循环条件
        循环体
  - 循环条件 Python 和 C 一样，任何非零整数都为真，
  - 循环体 需要缩进的 可以是空格或 tab
- for 作用：在列表或字符串等任意序列的元素上迭代
  - 格式
    for w in words:
        print(w, len(w))
- range() 函数
  - 格式 range(start, stop[, step])
  range 构造器的参数必须为整数（可以是内置的 int 或任何实现了 \__index__() 特殊方法的对象）。 如果省略 step 参数，则默认为 1。 如果省略 start 参数，则默认为 0。 如果 step 为零，则会引发 ValueError。
- enumerate() 函数
  - 格式 enumerate(iterable, start=0)
  - 返回一个枚举对象。iterable 必须是一个序列，或 iterator，或其他支持迭代的对象。 enumerate() 返回的迭代器的 __next__() 方法返回一个元组，里面包含一个计数值（从 start 开始，默认为 0）和通过迭代 iterable 获得的值。
  - 例子
    seasons = ['Spring', 'Summer', 'Fall', 'Winter']
    list(enumerate(seasons))
    [(0, 'Spring'), (1, 'Summer'), (2, 'Fall'), (3, 'Winter')]
- break 和 continue
  - break 跳出最近的一次 for 或 while 循环，后续内容不在执行
  - continue 将继续执行循环的下一次迭代
- else
  在 for 循环中，else 子句会在循环结束其他最后一次迭代之后，即未执行 break 的情况下被执行。
  在 while 循环中，它会在循环条件变为假值后执行。
  在这两类循环中，当在循环被 break 终结时 else 子句 不会 被执行。 当然，其他提前结束循环的方式，如 return 或是引发异常，也会跳过 else 子句的执行。
- pass
  pass 语句不执行任何动作。语法上需要一个语句，但程序毋需执行任何动作时，可以使用该语句。
  - 格式
    while True:
        pass  # 无限等待键盘中断 (Ctrl+C)
- match
  类似 rust 中的模式匹配
  - 格式
    match status:
        case 400:
            return "Bad request"
        case 404:
            return "Not found"
        case 418:
            return "I'm a teapot"
        case _:
            return "Something's wrong with the internet"
- 定义函数
  定义 函数使用关键字 def，后跟函数名与括号内的形参列表。函数语句从下一行开始，并且必须缩进。
  格式
  def fib2(n):  # 返回斐波那契数组直到 n
    """Return a list containing the Fibonacci series up to n."""
    result = []
    a, b = 0, 1
    while a < n:
        result.append(a)    # 见下
        a, b = b, a+b
    return result
- 函数详解
  - 默认值参数：参数设置默认值
    格式
    def ask_ok(prompt, retries=4, reminder='Please try again!'):
    while True:
        pass
    默认值只计算一次。
    def f(a, L=[]):
        L.append(a)
        return L

    print(f(1))
    print(f(2))
    print(f(3))
    结果
    [1]
    [1, 2]
    [1, 2, 3]
  - 关键字参数
    kwarg=value 形式的 关键字参数 也可以用于调用函数。
    例子
    def parrot(voltage, state='a stiff', action='voom', type='Norwegian Blue'):
        print("-- This parrot wouldn't", action, end=' ')
        print("if you put", voltage, "volts through it.")
        print("-- Lovely plumage, the", type)
        print("-- It's", state, "!")
    函数调用
    parrot(1000)                                          # 1 个位置参数
    parrot(voltage=1000)                                  # 1 个关键字参数
    parrot(voltage=1000000, action='VOOOOOM')             # 2 个关键字参数
  - 特殊参数
    默认情况下，参数可以按位置或显式关键字传递给 Python 函数。为了让代码易读、高效，最好限制参数的传递方式，这样，开发者只需查看函数定义，即可确定参数项是仅按位置、按位置或关键字，还是仅按关键字传递。
    定义
    def f(pos1, pos2, /, pos_or_kwd, *, kwd1, kwd2):
      -----------    ----------     ----------
        |             |                  |
        |        位置或关键字   |
        |                                - 仅限关键字
         -- 仅限位置
  - 任意实参列表
    调用函数时，使用任意数量的实参是最少见的选项。这些实参包含在元组中。
    例子
    def concat(*args, sep="/"):
        return sep.join(args)

    concat("earth", "mars", "venus")
    结果 'earth/mars/venus'
  - 解包实参列表
    函数调用要求独立的位置参数，但实参在列表或元组里时，要执行相反的操作。
    例子
    args = [3, 6]
    list(range(*args)) 
    字典可以用 ** 操作符传递关键字参数
  - lambda 表达式 lambda 关键字用于创建小巧的匿名函数。
    例子
    def make_incrementor(n):
        return lambda x: x + n
  - 文档字符串
    第一行应为对象用途的简短摘要。为保持简洁，不要在这里显式说明对象名或类型，因为可通过其他方式获取这些信息（除非该名称碰巧是描述函数操作的动词）。这一行应以大写字母开头，以句点结尾。
    文档字符串为多行时，第二行应为空白行，在视觉上将摘要与其余描述分开。后面的行可包含若干段落，描述对象的调用约定、副作用等。
    例子
    def my_function():
        """Do nothing, but document it.

        No, really, it doesn't do anything.
        """
        pass
  - 函数注解 是可选的用户自定义函数类型的元数据完整信息
    标注 以字典的形式存放在函数的 \__annotations__ 属性中而对函数的其他部分没有影响。
- 编码风格 [PEP8](https://peps.python.org/pep-0008/)
  - 缩进，用 4 个空格，不要用制表符。
  - 4 个空格是小缩进（更深嵌套）和大缩进（更易阅读）之间的折中方案。制表符会引起混乱，最好别用。
  - 换行，一行不超过 79 个字符。
  - 这样换行的小屏阅读体验更好，还便于在大屏显示器上并排阅读多个代码文件。
  - 用空行分隔函数和类，及函数内较大的代码块。
  - 最好把注释放到单独一行。
  - 使用文档字符串。
  - 运算符前后、逗号后要用空格，但不要直接在括号内使用： a = f(1, 2) + g(3, 4)。
  - 类和函数的命名要一致；按惯例，命名类用 UpperCamelCase，命名函数与方法用 lowercase_with_underscores。命名方法中第一个参数总是用 self (类和方法详见 初探类)。
  - 编写用于国际多语环境的代码时，不要用生僻的编码。Python 默认的 UTF-8 或纯 ASCII 可以胜任各种情况。
  - 同理，就算多语阅读、维护代码的可能再小，也不要在标识符中使用非 ASCII 字符。

## 数据结构

## 模块

## 输入与输出

## 错误和异常

## 类

## 标准库操作

## 虚拟环境和包

## 交互式编辑

## 浮点数运算