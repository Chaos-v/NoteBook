# @staticmethod和@classmethod方法的用法和区别

## staticmethod

1、装饰器 @staticmethod 修饰的方法为静态方法，一般使用形式：

● **类名.方法名（）**
● **实例化**一个类对象，再通过类对象**调用**静态方法


```python
class Web(object):
    @staticmethod
    def foo_staticmethod():
        """静态方法"""
        pass

if __name__ == '__main__':
    # （1）直接使用类名+方法名调用
    Web.foo_staticmethod()

    # （2）实例化一个对象
    instance = Web()
    # 使用实例对象去调用静态方法（不建议）
    instance.foo_staticmethod() 
```

2、类中定义的静态变量，使用 **className.variableName** 进行访问。

3、静态方法内部使用其他的静态方法、类方法，使用 **className.methodName()** 进行访问。

```python
class Web(object):
    # 静态变量（类变量）
    name = "Python_Web"

    # 构造方法
    def __init__(self):
        self.desc = "实例属性，不共享"

    # 类方法
    @classmethod
    def foo_classmethod_other(cls):
        print('类方法被调用！')

    # 另外一个静态方法
    @staticmethod
    def foo_staticmethod_other():
        print('另外一个静态方法被调用！')

    @staticmethod
    def foo_staticmethod():
        """静态方法"""
        # 引用静态变量（类变量）
        print(Web.name) 

        # 调用其他静态方法
        print(Web.foo_staticmethod_other()) 

        # 调用类方法
        print(Web.foo_classmethod_other())
```

4、静态方法内部调用普通方法或者访问实例属性，必须**通过实例对象去引用**，不能直接使用类名去访问。

```python
    @staticmethod
    def foo_staticmethod():
        """静态方法内部调用普通方法或者访问实例属性"""
        instance = Web()

        # 获取实例属性
        print(instance.desc) # desc在3中程序已经定义

        # 调用普通方法
        instance.norm_method()
```

5、在子类中调用父类定义好的静态方法，只需要将类名替换为子名。相当于子类在继承时候直接继承了父类的静态方法。

```python
class Web(object):
    """父类中定义静态方法"""
    @staticmethod
    def foo_staticmethod(arg1, arg2):
        pass

class Django(Web):
    """子类继承于父类"""
    pass

if __name__ == '__main__':
    # （1）使用类名（父类）去调用静态方法
    Web.foo_staticmethod("Hello", ",AirPython")

    # （2）使用类名（子类）去调用静态方法
    Django.foo_staticmethod("Hello", ",AirPython")
```

***

## classmethod

1、装饰器 @classmethod 修饰的方法为类方法，使用时会将类本身作为第一个参数 cls 传递给类方法。

cls 代表外层类本身，可以实例化，也可以直接调用静态方法、类方法、静态变量。

一般使用方法：

● **类名.方法名（）**
● **实例化**一个类对象，再通过类对象**调用**静态方法

```python
class Web(object):
    # 静态变量（类变量）
    name = "Python_Web"

    def __init__(self):
        """构造方法"""
        self.desc = "实例属性，不共享"

    def norm_method(self):
        """普通方法"""
        print('普通方法被调用！')

    @classmethod
    def use_classmethod(cls):
        """类方法，第一个参数为cls，代表类本身"""
        pass

    @staticmethod
    def foo_staticmethod():
        """静态方法"""
        print('静态方法被调用！')

    @classmethod
    def foo_classmethod_other(cls):
        """其他类方法"""
        print('另外一个类方法被调用！')


if __name__ == '__main__':
    # 使用类名去调用类方法
    Web.use_classmethod()
```

2、静态方法**内部引用静态变量**通常有2种等效的方式，分别是：

● className.variableName
● cls.variableName

```python
    # 类方法，第一个参数为cls，代表类本身
    @classmethod
    def foo_classmethod(cls):
        # 调用静态变量方式一
        print(cls.name)

        # 调用静态变量方式二
        print(Web.name)
```

3、类方法 classmethod **内部调用其他类方法**或者**内部调用静态方法**可以通过以下两种形式去调用方法

 ● className.classMethodName()
 ● className.staticMethodName()

```python
    # 类方法，第一个参数为cls，代表类本身
    @classmethod
    def foo_classmethod(cls):
        # 调用其他类方法
        cls.foo_classmethod_other()

        # 调用静态方法
        cls.foo_staticmethod()
```
 
 4、类方法在方法**内部调用类中普通方法**或者**内部访问类中实例属性**时候，需要**通过 cls 变量实例化类对象，然后通过这个对象去调用**普通方法和实例属性。

```python
    # 类方法，第一个参数为cls，代表类本身
    @classmethod
    def foo_classmethod(cls):
        # 如果要调用实例属性，必须使用cls实例化一个对象，然后再去引用
        print(cls().desc)

        # 如果要调用普通方法，必须使用cls实例化一个对象，然后再去引用
        cls().norm_method()
```

5、在子类中调用父类定义好的类方法时，和静态方法 @staticmethod 一样，只需要将类名替换为子类名即可。

## 区别
下面总结一下普通方法、静态方法、类方法的区别：

**普通方法：** 第一个参数 self 代表实例对象本身，可以使用 self 直接引用定义的实例属性和普通方法；如果需要调用静态方法和类方法，通过「 类名.方法名() 」调用即可；

**静态方法** 使用「 类名.静态变量 」引用静态变量，利用「 类名.方法名() 」调用其他静态方法和类方法；如果需要调用普通方法，需要先实例化一个对象，然后利用对象去调用普通方法；

**类方法** 第一个参数 cls 代表类本身（等价），通过「 cls.静态变量 」或「 类名.静态变量 」引用静态变量，利用「 cls.方法名() 」或「 类名.方法名() 」去调用静态方法和类方法；如果需要调用普通方法，需要先实例化一个对象，然后利用对象去调用普通方法；

静态方法和类方法是针对类定义的，除了可以使用类名去调用，也可以使用实例对象去调用，但是不建议。

## 总结
一般来说，如果方法内部涉及到实例对象属性的操作，建议用普通方法；如果方法内部没有操作实例属性的操作，仅仅包含一些工具性的操作，建议使用静态方法；而如果需要对类属性，即静态变量进行限制性操作，则建议使用类方法。
