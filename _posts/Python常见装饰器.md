# Python常见装饰器

## @property

property的作用主要是将一个函数变为属性进行调用

例如

```
class Person(object):
    def __init__(self,name):
        self.name=name

    def get_name(self):
        return self.name
p=Person("TT")
print(p.get_name())

输出:
TT
```

该类下的属性只有name一个：

![装饰器1](C:\Users\wangyu\Desktop\博客\装饰器1.png)

使用property后

```
class Person(object):

    def __init__(self,name):
        self.name=name
    @property
    def get_name(self):
        return self.name

p=Person("TT")

print(p.get_name)

输出:

TT
```

使用property后，该类下添加了一个get_name属性：

![装饰器2](C:\Users\wangyu\Desktop\博客\装饰器2.png)





## @lru_cache(None)

functools模块中的lru_cache(maxsize,typed)

加速递归函数

算法题目中可以使用

相当于记忆化搜索，保存已递归的数值