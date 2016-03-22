<!--
    作者：华校专
    email: huaxz1986@163.com
**  本文档可用于个人学习目的，不得用于商业目的  **
-->
# 类的高级主题
1.通过在自定义类中嵌入内置类型，类似委托；这样自定义类就可以实现内置类型的接口。  
这些接口在内部通过操作嵌入的内置类型来实现。
>这是一种扩展内置类型的方式

2.通过子类扩展内置类型：所有内置类型都可以直接创建子类，如`list`、`str`、`dict`、`tuple`、`set`等

* 内置类型的子类实例可以用于内置类型对象能够出现的任何地方

3.在Python3中所有的类都是新式类。
> Python2.2之前的类不是新式类

* 所有的类都是从`object`内置类派生而来
* `type(obj)`返回对象实例所属的类对象 
	>对于实例对象的`type(obj)`返回值也是`obj.__class__`
* `type(classname)`返回`"type"`，因为所有`class`对象都是`type`的实例
* 由于所有`class`均直接或者间接地派生自`object`类，因此每个实例对象都是`object`类的实例
* `object`是`type`类的实例，但是同时`type`又派生自`object`

  ![新式类](../imgs/python_28_1.JPG)

4.Python3中的类有一个`.__slots__`属性，它是一个字符串列表。这个列表限定了类的实例对象的合法属性名。如果给实例赋了一个`.__slots__`列表之外的属性名会引发异常

* 虽然有了`.__slots__`列表，但是实例对象的属性还是必须先赋值才存在

  ![.__slots__属性](../imgs/python_28_2.JPG)

* 当有`.__slots__`列表存在时，默认会删除`.__dict__`属性，而`getattr()`，`setattr()`
  以及`dir()`等函数均使用`.__slots__`属性，因此仍旧可以正常工作
	* 可以在`.__slots__`列表中添加`.__dict__`字符串，
	  因此对于使用`.__dict__`的地方均能正常工作

  ![.__slots__和 .__dict__](../imgs/python_28_3.JPG)

* `.__slots__`属性的使用可以优化内存和读取速度
* 在继承中:
	* 若子类继承自一个没有`.__slots__`的超类，则超类的`.__dict__`属性可用，则子类中的`.__slots__`
	  没有意义。因为子类继承了超类的`.__dict__`属性

		![.__slots__和 .__dict__](../imgs/python_28_4.JPG)

	* 若子类有`.__slots__`，超类也有`.__slots__`，子类的合法属性名为父类和子类的`.__slots__`列表的并集

		![超类子类都有`.__slots__`](../imgs/python_28_5.JPG)

	* 若超类有`.__slots__`，子类未定义`.__slots__`，则子类将会有一个`.__dict__`属性

		![超类有`.__slots__`子类没有](../imgs/python_28_6.JPG)

5.Python3的`property`机制：  
`property`是一个对象，通过它给类变量名赋值。

```
  class A:
	age=property(getMethod,setMethod,delMethod,docMethod)
	                                 # 或者直接指定docstring
	def getMethod(self):
		pass
	def setMethod(self,val):
		pass
	def delMethod(self):
		pass
	def docMethod(self):
		...
		# return a string
```

* `property`优点：代码简单，运行速度更快;缺点：当类编写时可能还无法确定`property`名字，因此无法提供动态的接口
* 如果`property`的`docstring`或者`docMethod`为`None`，则Python使用`getMethod`的`docstring`。

  ![property](../imgs/python_28_7.JPG)

一个添加了语法糖的方案为：

```
  class A:
    	def __init__(self):
        	self._x = None
    	@property #定义了一个property get函数，必选
    	def x(self): # property name 就是 get函数的函数名
        	"""I'm the 'x' property."""
        	return self._x
    	@x.setter #定义了一个property set函数，可选
    	def x(self, value):
        	self._x = value
    	@x.deleter #定义了一个property del函数，可选
    	def x(self):
        	del self._x
```

6.Python类中有两种特殊的方法：`staticmethod`方法和`classmethod`方法

* `staticmethod`方法：当以实例对象调用`staticmethod`方法时，Python并不会将实例对象传入作为参数；而普通的实例方法，通过实例对象调用时，Python将实例对象作为第一个参数传入
	* 定义`staticmethod`方法：

		```
		class A:
			@staticmethod #定义一个staticmethod
			def func(*args,**kwargs)
				pass		
		```
* `classmethod`方法：当以实例对象或者类对象调用`classmethod`方法时，Python将类对象（如果是实例对象调用，则提取该实例所属的类对象）传入函数的第一个参数`cls`中
	* 定义`classmethod`方法：

		```
		class A:
			@classmethod #classmethod
			def func(cls,*args,**kwargs)
				pass		
		```

总结一下，类中可以定义四种方法：

* 普通方法：方法就是类对象的一个属性，执行常规函数调用语义`classname.method(args)`
* 实例方法：传入一个实例作为方法的第一个实参。调用时可以：
	* `obj.method(args)`:通过实例调用
	* `classname.method(obj,args)`：通过类调用
* `staticmethod`方法：* `obj.method(args)`通过实例调用时，执行的是`classname.method(args)`语义
* `classmethod`方法：* `obj.method(args)`执行的是`classname.method(classname,args)`语义
	
  ![Python类中四种方法](../imgs/python_28_8.JPG)

7.类的实例方法中，用哪个实例调用的该方法，`self`就是指向那个实例对象  
类的`classmethod`方法中，用哪个类调用该方法，`cls`就指向那个类对象

  ![实例方法和类方法中的self、cls](../imgs/python_28_9.JPG)

8.类对象与实例对象都是可变对象，可以给类属性、实例属性进行赋值，这就是原地修改。这种行为会影响对它的多处引用  
  任何在类层次所作的修改都会反映到所有实例对象中

  ![类对象与实例对象都是可变对象](../imgs/python_28_10.JPG)

9.若类的某个属性是可变对象（如列表、字典），则对它的修改会立即影响所有的实例对象

  ![类属性为可变对象](../imgs/python_28_11.JPG)

10.多重继承中，超类在`class`语句首行内的顺序很重要。Python搜索继承树时总是根据超类的顺序，从左到右搜索超类。

  ![多重继承超类定义顺序](../imgs/python_28_12.JPG)

	