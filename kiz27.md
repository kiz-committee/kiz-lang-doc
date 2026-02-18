# kiz2027标准文档

### 就地修改

```kiz
+= -= /= *=
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

### 模块定义(更强大的模块系统)
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

### 包含类型注释的函数签名
类型仅做注释，不进行检查
```
fn function_name(param1: ty1, param2: ty2) -> ty3
    statement
end
```

### 常量定义
```
immut name = expression
```

### 管道运算符
```
expression |> func1 |> func2
```

### 组合定义（类似Rust风格枚举）
```
group Name
    member1
    member2
end

n = Name.member1(expression) # 可带值
# 通过with符号来匹配携带的值
when n
    .member1 with expression => 
        statements
    .member2 with expression => 
        statements
```

### 复合赋值/调用方法
```
in obj
    .name = expression
    .method(arg1, arg2)
end
```

### 链式比较
```
a > b > c
c < b < a
a or b or c
```

### 对象的\_\_call\_\_、\_\_init\_\_与Object的更多职责

在原先的 kiz 中，我们通过规定\_\_call\_\_方法，实现对象的新建，但是这就导致了，创建出来的对象仍然可以调用以创建新的对象，所以，当我们希望我们创建的对象的\_\_call\_\_方法按指定方式运行，如：

```kiz
// A 的定义……
adder = A()
print(adder(2, 3)) // 我们希望输出 5
```

此时我们必须在\_\_call\_\_中覆写创建出的对象的\_\_call\_\_方法，如：

```kiz
A = create()
A.__call__ = fn (self)
    obj = create(self)
    obj.__call__ = fn (self, a, b)
        return a + b
    end
    return obj
end
```

但是这样过于笨重，因此我们希望Object承担更多职责：

```kiz
// Object 的 kiz 伪代码：
Object = create() => (
    fn __call__(self, *args, **kwargs)
        obj = create(Object)
        del obj.__call__
        if (ischild(obj, Object))
            obj.__init__(*args, **kwargs)
        end
        return obj
    end

    fn __init__(self, *args, **kwargs)
        // 对新建对象的初始化，这里是一种做法，即不接受任何参数
        if len(args) or len(kwargs):
            throw Error("TypeError", "A() takes no arguments")
    end
    
    fn __eq__(self, other)
        return self is other
    end
)
```

这样前面的例子就可以写成：

```kiz
A = create()
A.__init__ = fn (self)
    self.__call__ = fn (self, a, b)
        return a + b
    end
end
```

减少了样板代码。

### 判断时赋值
```
if name = expression
    statements
end

if name = expression; expression
    statements
end
```

### GC
基于分代式的垃圾回收器