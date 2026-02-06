# kiz2027标准文档

### 延时资源释放

核心语法
在作用域结束或当前作用域抛出异常时, ensue后的表达式会被执行
```
ensure expression
```
示例
```
f = open("foo.txt")
ensure f.close()

print("do something")
throw RuntimeError() # 此时ensure才被执行

```

### 错误集合定义
核心语法
```
error name
    ty = subname1 | subname2
end
```
示例
```
error IOError
    ty = FileNoFoundError | FileUnwriteError 
end
```

### 模块定义
在module块内设置要导出的对象
核心语法
```
module name
    sub1, sub2
end
```
示例
```
module math
    sin, cos, tan, e, pi
end
```
其他文件
```
# 兼容两种写法
import module math
import math
```

### 模式匹配
核心语法
```
when name
    value => 
        statement
    value =>
        statement
    _ =>
        statement
end

# 表达式形态
when name(
value => expression
value => expression
_ => expression
)
```

### GC
基于分代式的垃圾回收器

### VmFrame优化
合并操作数栈和常量池为`locals_with_stack`变量(类似Python), 
作为局部变量表和操作数栈的唯一基本动态数组

以便优化速度