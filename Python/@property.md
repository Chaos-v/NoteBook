# @property的介绍与使用

@property作为python的一种装饰器，用来修饰方法。

可以使用@property装饰器创建**只读属性**，@property装饰器会将方法转化为相同名称的只读属性，可以与定义的属性搭配使用，防止属性被修改。

## 使用

### 1、像属性一样访问

```python
class DataSet(object):
  @property
  def method_with_property(self):  # 含有@property
      return 15
  def method_without_property(self):  #不含@property
      return 15


l = DataSet()

# 使用@property可以直接用调用属性的形式来调用方法
print(l.method_with_property) 

# 不使用@property则必须使用通常的调用方法进行调用
print(l.method_without_property())  
```

### 2、与所定义的属性配合使用，防止属性被修改

Python在属性定义时无法设置私有属性，通过@property的方法进行设置可以隐藏属性名称，使用户无法随意修改。

```python
class DataSet(object):
    def __init__(self):
        self._images = 1
        self._labels = 2 # 定义属性的名称

    @property
    def images(self): 
    """相当属性，只可以使用，用户无法法随意修改。"""
        return self._images

    @property
    def labels(self):
        return self._labels


l = DataSet()

# 用户在不知道_images名称的条件下直接调用images即可获得_images值
# 因此用户无法更改属性，从而保护了类的属性。
print(l.images)

```

