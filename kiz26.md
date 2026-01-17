# Kiz-2026.1
版本号含义：2026年第一个正式版本

## 语言核心定位
kiz-lang 是一门 **面向对象(原型链模型）、强类型+动态类型** 的轻量化脚本语言，**使用C++开发**，采用「半编译半解析」架构，内置**栈式虚拟机**(VM）与基于引用计数(reference count）的对象模型。

**核心设计亮点**：
- 通过对象的 `__parent__` 属性绑定上级对象，实现原型链继承
- 支持运算符重载与魔术方法
- int 类型为无限精度整数，小数为自动无限精度精准小数

## 目录
1.  [快速使用指令](#快速使用指令)
2.  [核心语法](#核心语法)

    2.1  注释

    2.2  变量与表达式

    2.3  对象

    2.4  控制流

    2.5  函数

    2.6  错误处理

    2.7  模块系统

3.  [基本运算符](#基本运算符)
4.  [内置对象](#内置对象)
5.  [对象魔术名](#对象魔术名)
6.  [Kiz-2027版本功能规划](#Kiz-2027版本功能规划)

## 快速使用指令
通过 `kiz.exe` 可执行文件操作，支持以下命令：
```bash
.\kiz.exe          # 启动 REPL 交互式环境
.\kiz.exe repl     # 同上，显式指定 REPL 模式
.\kiz.exe <path>   # 运行指定路径的 .kiz 脚本文件
.\kiz.exe run <path> # 同上，显式指定运行模式
.\kiz.exe version  # 查看当前版本号
.\kiz.exe help     # 查看帮助文档
```

**Hello World**

直接运行kiz.exe，在弹出的repl窗口输入
```
>>> print("Hello World")
```

---
## 核心语法

### 注释
目前只支持单行注释
```
# text
```

### 变量与表达式
变量声明/赋值都不需要关键字
```
var_name = expression
```

基本字面量
```
x = [1,2,"7"]
x[0]
x.contains("7")

x = 0
x = 0.1
x = "Hello World"
x = True
x = False
x = Nil

x = |a| a+1 # 单行lambda
x = fn (a)  # 多行lambda
    return a
end
x(1)

x = {"a"=0, "b"=1, 0=2} # 任意可哈希的值都可以作为键
x["a"]

a = [1,2,3]
b = a # List是可变对象, kiz自动选择拷贝

a = 0
b = a # Int是不可变对象, kiz自动选择引用

0.1 + 0.2 == 0.3 # True, kiz中的小数是Decimal类型而不是传统浮点数
```

kiz是动态类型的，意味着你可以为一个变量赋值任何类型的对象。

kiz是强类型的，意味着你不可以 `1 + "1"`。
### 对象
Kiz中，一切皆是对象
任意参数数量大于一的函数都可以通过设置属性语句绑定为对象方法(函数的第一个实际参数指向调用源对象、类似于Python的self/cls)
基本创建对象语法
```
obj = create()
obj.v = 0
obj.v  # 0
```

复合创建对象语法
```
object Name
    attr_name = expression
    fn method(this, param1, param2)
        statements
    end
end
```

```
object Dog
    name = ""
    # 重载对象的__call__实现类似Python __init__的特性
    fn __call__(this, n)
        # 创建一个__parent__属性指向Dog的对象并设置其name属性为参数n
        o = create(this, {name=n})
        return o
    end
    # 函数第一个参数指向调用源对象，建议命名为this
    fn bark (this)
        print(this.name, "is barking")
    end
end

my_dog = Dog("Tom")
my_dog.bark()
```

### 控制流
分支语句
注意if的判断会通过对象的 `__bool__` 方法来判断
```
if expression 
    statements
else if expression 
    statements
else if expression
    statements 
else
    statements 
end
```

循环
```
while expression 
    statements
end

# obj必须是拥有__next__的对象
for var_name : obj
    statements
end
```

```
break
```

类似于其他语言的continue语句
```
next
```

### 函数
kiz的函数支持递归

实名函数
```
fn function_name(param1, param2)
    statements
end
```

```
# 在没有任何形式参数时，fn foo()的签名可以简写为fn foo
fn foo(m)
    # 函数是对象，意味着函数可以拥有属性，利用函数的属性可以实现其他语言函数静态变量类似的功能
    foo.k = 0
    a = 1 + m
    # 可以不写return, return 不一定要写在函数最后一行，默认return Nil
    return a
end


foo(6)
foo.k # 0
```

匿名函数
```
fn (param1, pararm2)
    statements
end

# 单行匿名函数简写形式
| param1, param2 | expression
```

```
foo = fn ()
    a = 1
    return a
end

m = [1,2,3].map( |a| a>0 ) # 单行匿名函数可以简写
```

跨作用域处理(与Python的global, nonlocal语句略有语法上的差异)

获取上层作用域的变量
```
x # 直接获取，与获取局部变量语法上无区别
```

设置全局变量
```
global name = expression
```

```
global x = 0
```

设置上层作用域的变量(向上层作用域查找直到调用栈顶部）
```
nonlocal name = expression
```

```
nonlocal x = 0
```

### 错误处理
抛出错误


自定义错误对象的创建建议返回Error对象, 

因为Error对象会储存错误代码的位置信息的调用栈快照,

如果你抛出Error对象, 那么会打印TraceBack, 错误名和错误信息,

如果你抛出其他对象则只会打印错误名和错误信息。
```
object MyError
    fn __call__(this, name, msg)
        return Error(name, msg)
    end
end
```

```
throw expression
```

注意: catch不会重复捕获

```
try
    # do something

catch var_name: ErrorBasedObject
    # do something
	
catch var_name: Error2BasedObject
    # do something

finally
    # do something
end
```

当try中有break continue时, 如果会跳转到try-catch-finally的, 会先执行finally块

当try中有return时, 会先执行finally块

### 模块系统
从标准模块表中导入模块
```
import std_lib_name
```

当前标准模块表
```
builtins
math
runtime
```

从指定路径导入模块
```
import "other.kiz"
```

路径搜索优先级
```
1. 指定路径
2. ../ 指定路径
3. Kiz可执行文件(kiz.exe/kiz.elf)所在目录 / libs / 指定路径
```

```
# 这是other.kiz
# 设置模块名
__name__ = "other"

x = 100

fn a()
    print("Hello from a")
end

fn foo ()
    # 支持模块内引用其他数据
    a()
    
    print("hello from other.kiz")
end
```

使用模块成员
```
other.x
other.x = 0
other.foo() # 调用模块子函数不会把module作为函数的第一个实际参数
```
### 基本运算符

| 运算符        | 功能描述                                                                                                  |
| ---------- | ----------------------------------------------------------------------------------------------------- |
| `a+b`      | 加法运算，支持Int、Decimal、Str类型；Str类型为拼接，数值类型为算术加，调用`__add__`魔术方法                                            |
| `a-b`      | 减法运算，仅支持Int、Decimal类型，调用`__sub__`魔术方法                                                                 |
| `a*b`      | 乘法运算，支持Int、Decimal、Str类型；Str类型为重复拼接，数值类型为算术乘，调用`__mul__`魔术方法                                          |
| `a/b`      | 除法运算，仅支持Int、Decimal类型，返回无限精度结果，调用`__div__`魔术方法                                                        |
| `a^b`      | 乘方运算，仅支持Int、Decimal类型，调用`__pow__`魔术方法                                                                 |
| `a%b`      | 取模运算，仅支持Int、Decimal类型，返回除法余数，调用`__mod__`魔术方法                                                          |
| `a==b`     | 相等性判断，调用对象`__eq__`魔术方法，原型链查找匹配逻辑                                                                      |
| `a>=b`     | 大于等于比较，仅支持数值类型，调用`__ge__`魔术方法                                                                         |
| `a<=b`     | 小于等于比较，仅支持数值类型，调用`__le__`魔术方法                                                                         |
| `a!=b`     | 不等性判断，基于`__eq__`结果取反                                                                                  |
| `a and b`  | 短路逻辑与，先判断a的`__bool__`，为True时再判断b，否则直接返回a                                                              |
| `a or b`   | 短路逻辑或，先判断a的`__bool__`，为False时再判断b，否则直接返回a                                                             |
| `not a`    | 逻辑非，对a的`__bool__`结果取反                                                                                 |
| `a[b]`     | 下标访问，调用对象`__getitem__`魔术方法                                                                            |
| `a[b] = c` | 下标赋值，调用对象`__setitem__`魔术方法                                                                            |
| `a(b)`     | 函数/方法调用，调用对象`__call__`魔术方法；方法调用时自动绑定调用源对象为第一个参数                                                       |
| `a.b`      | 对象访问(kiz的属性查找按照优先在该对象查找，如果找不到再往父对象找的原则）                                                               |
| `a.b = c`  | 对象属性赋值，直接修改当前对象的属性表，不触发原型链查找                                                                          |
| `a.b(c)`   | 方法调用(注意kiz对于方法的查找遵循该原则：1. 如果是普通方法则优先在该对象查找，如果找不到再往父对象找；2. 如果是魔术方法，则忽略该对象的属性表，往直接父对象中查找，如果找不到再往间接父对象找） |


### 内置对象

| 对象                                            | 类型   | 功能描述                                                                                                                     | 重载的运算符                                                 |
| --------------------------------------------- | ---- | ------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------ |
| `print(...)`                                  | 函数   | 打印任意数量参数到标准输出，参数间用空格分隔，末尾换行                                                                                              | 无                                                      |
| `input(prompt="")`                            | 函数   | 输出prompt提示文本，读取用户输入的一行字符串并返回                                                                                             | 无                                                      |
| `get_refc(obj)`                               | 函数   | 获取对象的引用计数值，用于调试引用计数机制                                                                                                    | 无                                                      |
| `breakpoint()`                                | 函数   | 触发虚拟机断点调试，暂停执行并等待调试指令                                                                                                    | 无                                                      |
| `now()`                                       | 函数   | 返回当前系统时间戳(毫秒级），类型为Int                                                                                                    | 无                                                         |
| `getattr(current_only=False, obj, attr_name, default_val)`        | 函数   | 获取对象attr_name属性；属性不存在时返回default_val，未传default_val则抛错                                                                     | 无                                                      |
| `setattr(obj, attr_name, val)`                | 函数   | 设置对象attr_name属性值为val，直接修改当前对象属性表                                                                                         | 无                                                      |
| `delattr(obj, attr_name)`                     | 函数   | 删除对象attr_name属性，属性不存在则抛错                                                                                                 | 无                                                      |
| `hasattr(obj, attr_name, current_only=False)` | 函数   | 判断对象是否存在attr_name属性(含原型链查找, 通过设置 `current_only`属性为 `True` 取消按原型链查找），返回Bool类型                                             | 无                                                      |
| `ischild(obj, parent_obj)`                    | 函数   | 判断obj的原型链中是否包含parent_obj，返回Bool类型                                                                                        | 无                                                      |
| `help(key="")`                                | 函数   | 无参时返回总帮助文档；传入key时返回对应内置对象/语法的帮助信息                                                                                        | 无                                                      |
| `range(start=0, end, step=1)`                 | 函数   | 生成整数序列列表，包含start，不包含end，步长为step                                                                              | 无                                                      |
| `cmd(inst_name, inst_args={})`                | 函数   | 执行shell指令，inst_name为指令名，inst_args为指令参数 (List/Dict)                                                                                  | 无                                                      |
| `create(obj=parent)`                          | 函数   | 创建一个 `__parent__`属性为obj的空对象                                                                                              | 无                                                      |
| `hash(obj)`                                   | 函数   | 获取对象的哈希值                                                                                                                   | 无                                                        |
| `Int`                                         | 基本类型 | 无限精度整数类型，支持任意大小整数运算                                                                                                      | `+ - * / ^ % == > < Int(other_type_obj)`               |
| `Decimal`                                     | 基本类型 | 无限精度小数类型，避免浮点数精度丢失问题                                                                                                     | `+ - * / ^ % == > < Decimal(other_type_obj)`           |
| `Str`                                         | 基本类型 | 字符串类型(除魔术方法外的其他方法<br>`startswith` `endswith` `isnum` `isalpha` `find` `map` `count` `filter` )                           | `+ * == Str[idx] Str(other_type_obj)`                  |
| `Nil`                                         | 基本类型 | 空值类型，唯一实例为`Nil`，表示无有效数据                                                                                                  | 无                                                      |
| `Bool`                                        | 基本类型 | 布尔类型，仅有`True`和`False`两个实例，支持逻辑运算                                                                                         | `and or not ==`                                        |
| `List`                                        | 基本类型 | 有序可变序列(动态数组），支持下标访问、增删元素；除魔术方法外的其他方法`foreach` `reverse` `extend` `pop` `insert` `find` `map` `count` `filter` `__next__` | `+ * == List[idx] List[idx]=item List(other_type_obj)` |
| `Dict`                                        | 基本类型 | 键值对集合，键支持任意可哈希对象，值支持任意对象；支持键的增删改查                                                                                        | `+ == Dict[key] Dict[key]=value Dict()`                |
| `Object`                                      | 基本类型 | 所有对象的根原型，提供基础属性查找与原型链机制                                                                                                  | `== Object[attr_name] Object[attr_name]=value`         |
| `Func`                                        | 基本类型 | 用户定义的函数对象，支持属性绑定、递归调用                                                                                                    | 无                                                      |
| `NFunc`                                       | 基本类型 | 内置函数(使用C++实现的函数），性能优于用户定义函数                                                                                              | 无                                                      |
| `Module`                                      | 基本类型 | 模块对象，存储模块内的变量、函数等成员，支持属性访问                                                                                               | 无                                                      |
| `Error`                                       | 基本类型 | 错误基本对象，包含错误信息字符串与调用栈信息，所有错误类型的父对象                                                                                        | 无                                                      |


### 对象魔术名

| 魔术方法名              | 类型   | 功能描述                        |
| ---------------------- | ------ | ------------------------------- |
| `__parent__`           | 任意对象 | 原型链核心属性，指向父对象       |
| `__add__`              | 函数   | 重载 `+` 运算符                  |
| `__sub__`              | 函数   | 重载 `-` 运算符                  |
| `__mul__`              | 函数   | 重载 `*` 运算符                  |
| `__div__`              | 函数   | 重载 `/` 运算符                  |
| `__pow__`              | 函数   | 重载 `^` 运算符(乘方）          |
| `__mod__`              | 函数   | 重载 `%` 运算符 (取模）         |
| `__eq__`               | 函数   | 重载 `==` 运算符                 |
| `__gt__`               | 函数   | 重载 `>` 运算符                  |
| `__lt__`               | 函数   | 重载 `<` 运算符                  |
| `__ge__`               | 函数   | 重载 `>=` 运算符                 |
| `__le__`               | 函数   | 重载 `<=` 运算符                 |
| `__owner_module__`     | 模块   | 标注对象所属模块id(用于模块系统查找） |
| `__call__`             | 函数   | 支持对象直接调用(`obj()`）      |
| `__bool__`             | 函数   | 支持布尔判断(`if obj`）         |
| `__str__`              | 函数   | 转换为字符串(`str(obj)`）       |
| `__dstr__`             | 函数   | 返回调试字符串(类似 python 的 `repr`） |
| `__getitem__`          | 函数   | 重载下标访问(`obj[idx]`）       |
| `__setitem__`          | 函数   | 重载下标赋值(`obj[idx] = val`） |
| `__next__`             | 函数   | 迭代器方法(支持 `for` 循环）    |
| `__mutable__`         | 函数   | 判断对象可变性，用于决定引用/拷贝对象 |
| `__hash__`            | 函数   | 获取对象的哈希值                    |
| `__name__`             | 字符串  | 设置模块名                       |


---

## Kiz-2027版本功能规划
- 管道运算符
- 小整数优化
- 模板字符串
- `when .. => .. end` 模式匹配语句
- `fn obj.a(this) ... end` 直接设置方法语句
- `..obj` 解包语句

---

文档编撰:  *azhz1107cat*
