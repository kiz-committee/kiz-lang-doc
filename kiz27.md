# kiz2027标准文档

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
fn function_name(param1: tu1, param2: ty2) -> ty3
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
with obj
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

### GC
基于分代式的垃圾回收器