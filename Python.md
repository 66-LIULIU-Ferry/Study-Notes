### 基础数据类型与语法
##### 1. 列表（List）和元组（Tuple）的区别
- **列表**：可变对象，可修改/增删元素，占用内存大
- **元组**：不可变对象，创建后不可修改，占用内存小
##### 2. 深拷贝（deepcopy）和浅拷贝（copy）的区别
- **浅拷贝**：只拷贝父对象，不拷贝内部嵌套的子对象
- **深拷贝**：递归拷贝所有曾记得对象，完全独立，修改互不影响
##### 3. is 和 == 的区别
- is：比较的是内存地址，及id()是否相同
- == ：比较的是值内容是否相同
##### 4. 变量查找顺序（LEGB）
- **L（Local）**：局部作用域（函数内）
- **E (Enclosing)**：嵌套父级函数的局部作用域（闭包外层）
- **G (Global)**：全局作用域（模块级别）
- **B (Built-in)**：内置作用域（如 `len`, `int` 等）

## 高级特征
##### 1. 什么是装饰器（Decorator）
在不修改原函数代码的情况下，给函数“加功能”的工具
``` python
def log(func):  
def wrapper():  
print("函数开始执行")  
func()  
print("函数执行结束")  
return wrapper  
  
@log  
def hello():  
print("Hello!")  
  
hello()
```
输出：
```
函数开始执行  
Hello!  
函数执行结束
```
##### 2. 迭代器（Iterator）和生成器（Generator）的区别
- **迭代器**：可以被遍历的对象，满足两个条件：`__iter__()`和`__next__()`
``` python
nums = [1, 2, 3]  
it = iter(nums)  # 获取迭代器

# 取下一个值
print(next(it)) # 1  
print(next(it)) # 2
```
- **生成器**：创建迭代器的方法
```python
def gen():
    print("开始")
    yield 1
    print("继续")
    yield 2

# `yield` 是一个关键字，用来暂停函数执行，并返回一个值，下次从暂停的位置继续执行
```
调用：
```python
g = gen()

print(next(g))
print(next(g))
```
输出：
```
开始
1
继续
2
```
##### 3. 闭包
必须满足三个条件：
1. 有嵌套函数
2. 内部函数引用了外部函数的变量。
3. 外部函数返回了内部函数的引用
```python
def counter():
    count = 0

    def add():
        nonlocal count  # 一定要加nonlocal
        count += 1
        return count

    return add

c = counter()

print(c())  # 1
print(c())  # 2
print(c())  # 3
```
##### 4. args 和 kwargs
- `args`：接收不定数量的**位置参数**，打包成一个**元组** (tuple)。
- `kwargs`：接收不定数量的**关键字参数**，打包成一个**字典** (dict)

## 面向对象（OOP）
##### 1. `__init__` 和 `__new__` 的区别？
- `__init__`：负责初始化已经创建好的实例属性
- `__new__`：负责创建并返回类的实例对象
```
当执行`obj = A()`的时候，实际发生的是：
1. 调用 A.__new__(cls) → 创建对象
2. 返回实例对象 obj
3. 调用 A.__init__(self) → 初始化对象
```
##### 2. **类方法 (`@classmethod`) 和静态方法 (`@staticmethod`) 的区别**
- `@classmethod`：可以访问类
- `@staticmethod`：不能访问类或实例
```python
class A:
    count = 10

    @classmethod
    def cls_method(cls):
        print(cls.count)

    @staticmethod
    def static_method():
        print("hello")
```
##### 3. **Python 多重继承中的 MRO（方法解析顺序）是什么？**
先看自己 → 再看左边父类 → 再看右边父类 → 再往上找
```python
class A:
    def f(self):
        print("A")

class B(A):
    pass

class C(A):
    def f(self):
        print("C")

class D(B, C):
    pass

d = D()
d.f()
```
查找顺序：
```
D → B → C → A → object
```
输出：
```
C
```
## 内存管理与并发
##### 1. **请简述 Python 的垃圾回收机制 (GC)**
**Python回收垃圾 = 引用计数为主 + 标记清除 + 分代回收**
- **引用计数（为主）**：对象有一个引用计数器，有变量指向它则+1，失去则-1，变成0时立刻收回内存
- **标记清除**：专门用来解决循环引用（比如A包含B，B包含A），导致计数不为0的问题
- **分代回收**：用空间换时间，将对象按照存活时间分为三代，存活越久的对象检查频次越低，提高垃圾回收效率
##### 2. 什么是GIL（全局解释器锁）？它有什么影响？
- **概念**：解释器持有的一把互斥锁，保证同一时刻只有一个线程在执行Python字节码
- **影响**：多线程无法利用多核CPU
- **对策**：
	- **计算密集型任务**：用多进程，绕过GIL
	- **I/O密集型任务**：用多线程，因为遇到I/O阻塞时线程会主动释放GIL，多线程依然能提升效率
##### 3. 进程、线程、协程的区别？
- **进程（Process）**：操作系统资源分配的最小单元，开销最大，数据隔离
- **线程（Thread）**：CPU调度的最小单位，共享进程内存，容易产生数据竞态
- **协程**：用户态轻量级线程
