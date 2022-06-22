# some tip

## 特殊方法


类别 | 方法名
-- | -- 
字符串/字节序列的表示形式 | repr str format bytes
数值转换 | abs bool complex int float hash index
集合模拟 | len getitem setitem delitem contains
迭代枚举 | iter reversed next
可调用模拟 | call
上下文管理 | enter exit
实例创建和销毁 | new init del
属性管理 | getattr getattribute setattr delattr dir
属性描述符 | get set delete
跟类相关的服务 | prepare instancecheck subclasscheck
-- | --
一元运算符 | neg -、 post + 、abs abs()
比较运算符 | lt <、le <=、 eq ==、 ne != 、gt > 、ge >=
算术运算符 | add + 、sub - 、mul * 、 truediv / 、 floordiv // 、mod % 、 divmod divmod()、 pow **或pow()、 round round()
反向算术运算符 | radd rsub rmul rtruediv rfloordiv rmod rdivmod rpow 
增量运算符 | iadd isub imul itruediv ifloordiv imod ipow
位运算符 | invert ~、 lshift <<、 rshift >>、 and &、 or \|、 xor ^
反向位运算符 | rlshift rrshift rand ror rxor
增量赋值运算符 | ilshift irshift iand ixor ior

使用生成器表达式代替列表推导
使用filter和map

```python
>>> symbols = '##@$#@$'
>>> ascii = [ord(s) for s in symbols if ord(s) < 127]
>>> ascii
[35, 35, 64, 36, 35, 64, 36]

>>> ascii = list(filter(lambda c: c < 127, map(ord, symbols)))
>>> ascii
[35, 35, 64, 36, 35, 64, 36]
```

同理可以使用生成器表达式初始化元组和数组

```python
>>> tuple(ord(s) for s in symbols if ord(s) < 127)
(35, 35, 64, 36, 35, 64, 36)
>>> import array
>>> array.array('I', (ord(s) for s in symbols if ord(s) < 127))
array('I', [35, 35, 64, 36, 35, 64, 36])
```

元组不仅仅是不可变的列表，元组其实是对数据的记录：元组中的每个元素都存放了记录中一个字段的数据，外加这个字段的位置，元组的拆包时对于没用的元素，可以使用“_”(注意：gettext.gettext函数的别名)当占位符，用*来处理剩下的元素。

具名元组：collections.namedtuple是一个工厂函数，可以用来构建一个带有字段名的元组和一个带有名字的类(帮助调试程序)


