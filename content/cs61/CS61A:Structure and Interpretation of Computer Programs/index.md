+++
date = '2025-08-10T16:28:19+08:00'
draft = false
title = 'CS61A:Structure and Interpretation of Computer Programs'

+++

# CS61A：Structure and Interpretation of Computer Programs

> ​	相当于简单学习一下py，再打一打编程的基础，因为课程lab的质量很高。
>
> **坚持下去，哪怕它没有学分。**
>
> ​	这也是计算机的一大圣经(计算机程序的构造和解释)了，但是我还没有好好理解过。
>
> ​	笔者做这个课程的时候已经是大二结束了，所以不会重复很多基础的东西，比较跳跃。
>
> ​	资源来自于中文课本：https://composingprograms.netlify.app/
>
> ​	会摘录其中我觉得对我有用的内容(未报备)。我认为有些部分翻译的很抽象.如果你英语还可以,可以对照着看.

## 1.1第一章 函数构建抽象

### 1.1.1高阶函数

#### 1.1.1.1**函数作为参数传递：**

```py
 # 把函数作为参数传递
 def summation (n, term):
     sum , k = 0 , 1
     while k <= n:
         sum += term(k)
         k += 1
     return sum
 
 def pi_term(x):
     return 8 / ((4*x-3) * (4*x-1))
 
 def pi_sum(n):
     return summation(n, pi_term)
 
 res = pi_sum(1e6)
 print(res)
```

#### 1.1.1.2**嵌套函数的定义：**

继承环境 帧链

​	`sqrt_update` 函数体中的返回表达式可以通过遵循这一帧链来解析 `a` 的值。查找名称会找到当前环境中绑定到该名称的第一个值。Python 首先在 `sqrt_update` 帧中进行检查 --> 不存在 `a` ，然后又到 `sqrt_update` 的父帧 `f1` 中进行检查，发现 `a` 被绑定到了 256。

> 环境的继承，在父帧中找。

```python
 1 def average(x, y):
 2       return (x + y)/2
 3   
 4   def improve(update, close, guess=1):
 5       while not close(guess):
 6           guess = update(guess)
 7       return guess
 8   
 9   def approx_eq(x, y, tolerance=1e-3):
 10      return abs(x - y) < tolerance
 11  
 12  def sqrt(a):
 13      def sqrt_update(x):
 14          return average(x, a/x)
 15      def sqrt_close(x):
 16          return approx_eq(x * x, a)
 17      return improve(sqrt_update, sqrt_close)
 18  
 19  result = sqrt(256)
```

#### 1.1.1.3**Python 中词法作用域的两个关键优势：**

- 局部函数的名称**不会影响定义它的函数的外部名称**，因为**局部函数的名称将绑定在定义它的当前局部环境**中，而**不是全局环境**中。
- **局部函数可以访问外层函数的环境**，这是因为**局部函数的函数体的求值环境会继承定义它的求值环境**。

​	这里的 `sqrt_update` 函数自带了一些数据：`a` 在定义它的环境中引用的值，因为它以这种方式“封装”信息，所以**局部定义的函数通常被称为闭包**（closures）。

#### 1.1.1.4**把函数作为返回值**

```py
 >>> def compose1(f, g):
         def h(x):
             return f(g(x))
         return h
```

#### 1.1.1.5**柯里化（Curring）**

> ​	我认为这里的便捷性是，有一个函数有两个参数，你在调用的时候可以分两次调用分别传入两个参数。

```python
 >>> def curried_pow(x):
         def h(y):
             return pow(x, y)
         return h
 >>> curried_pow(2)(3)
 8
```

这个参数传递**稍微有点抽象**：

```py
 >>> def map_to_range(start, end, f):
         while start < end:
             print(f(start))
             start = start + 1
 >>> map_to_range(0, 10, curried_pow(2))
 1
 2
 4
 8
 16
 32
 64
 128
 256
 512
```

curried_pow(2)本身就可以作为一个可以传入一个参数的函数。

更复杂：curring和uncurring

```python
 >>> def curry2(f):
         """返回给定的双参数函数的柯里化版本"""
         def g(x):
             def h(y):
                 return f(x, y)
             return h
         return g
 >>> def uncurry2(g):
         """返回给定的柯里化函数的双参数版本"""
         def f(x, y):
             return g(x)(y)
         return f
 >>> pow_curried = curry2(pow)
 >>> pow_curried(2)(5)
 32
 >>> map_to_range(0, 10, pow_curried(2))
 1
 2
 4
 8
 16
 32
 64
 128
 256
 512
```

​	`curry2` 函数接受一个**双参数函数** `f` 并返回一个单参数函数 `g`。当 `g` 应用于参数 `x` 时，它返回一个单参数函数 `h`。当 `h` 应用于参数 `y` 时，它调用 `f(x, y)`。因此，`curry2(f)(x)(y)` **等价于** `f(x, y)` 。`uncurry2` **函数反转了柯里化变换**，因此 `uncurry2(curry2(f))` 等价于 `f`。

#### 1.1.1.6**Lambda表达式**

一个 **lambda** 表达式的计算结果是一个函数，它仅有一个**返回表达式**作为主体。不允许使用**赋值和控制**语句。

```py
 >>> def compose1(f, g):
         return lambda x: f(g(x))
```

**匿名函数**：

```py
>>> s = lambda x: x * x
>>> s
<function <lambda> at 0xf3f490>
>>> s(12)
144
```

**嵌套lambda**，有点难以辨认：

```py
>>> compose1 = lambda f,g: lambda x: f(g(x))
```

这么理解：

```python
lambda              x         :              f(g(x))
"A function that    takes x   and returns    f(g(x))"
```

#### 1.1.1.7**函数装饰器**

```py
>>> def trace(fn):
        def wrapped(x):
            print('-> ', fn, '(', x, ')')
            return fn(x)
        return wrapped

>>> @trace
    def triple(x):
        return 3 * x

>>> triple(12)
->  <function triple at 0x102a39848> ( 12 )
36
```

​	在这个例子中，定义了一个高阶函数 `trace`，它返回一个函数，该函数在调用其参数前先输出一个打印语句来显示该参数。`triple` 的 `def` 语句有一个注解（annotation） `@trace`，它会影响 `def` 执行的规则。和往常一样，函数 `triple` 被创建了。**但是，名称 triple 不会绑定到这个函数上。相反，这个名称会被绑定到在新定义的 `triple` 函数调用 `trace` 后返回的函数值上**。代码中，这个装饰器等价于：

> 相当于函数是把自己作为参数，调用上面的函数来作为返回值的结果，有什么作用，还没理解？

```py
>>> def triple(x):
        return 3 * x
>>> triple = trace(triple)
```

​	装饰器符号 `@` 也可以后跟一个调用表达式。跟在 `@` 后面的表达式会先被解析（就像上面的 'trace' 名称一样），然后是 `def` 语句，最后将装饰器表达式的运算结果应用到新定义的函数上，并将其结果绑定到 `def` 语句中的名称上。

> Hog写完了，比较简单也比较有趣，整体就是关于函数的抽象这里的一些内容，如果没有理解可能会容易绕进去：2025.5.23 14:28.

#### 1.1.1.8递归函数

就是递归（？）,主要就是递归的思想，比较基础。

这里的hw03就是一些普通的递归函数。

## 1.2第二章 数据构建抽象

> 有效使用内置数据类型和用户定义的数据类型是数据处理型应用（data processing applications）的基础。
>
> 面向对象，对象就是数据，抽象或者具体的。

### 1.2.1原始数据类型

三种

```py
>>> type(1 + 2j)
<class 'complex'>
>>> type(1)
<class 'int'>
>>> type(1.1)
<class 'float'>
```

> ​	非数值类型（Non-numeric types）：值可以表示许多其他类型的数据，比如**声音、图像、位置、网址、网络连接**等等。它们**中间的少数可以用原始数据类型**表示，例如用于值 `True` 和 `False` 的 `bool` 类，其他大多数值的类型必须由程序员使用我们将在本章中学习到的组合和抽象方法来定义。

### 1.2.2数据抽象

程序---》操作抽象数据  + 定义具体表示

list 列表

**抽象屏障**：上层的抽象利用下层的抽象，不可以越级来调用。

> 特定表示的函数越少，程序越好得以维护。

### 1.2.3序列

> ​	序列（**sequence**）是一组**有顺序的值的集合**，是计算机科学中的一个强大且基本的抽象概念。序列**并不是特定内置类型或抽象数据表示的实例**，而是一个包含不同类型数据间共享行为的集合。也就是说，序列有很多种类，但它们都具有共同的行为。

#### 1.2.3.1list 列表

```py
>>> digits = [1, 8, 2, 8]
>>> len(digits)
4
>>> digits[3]
8
```

**序列计算：**

> 我觉得py比较奇妙的地方。

```py
>>> [2, 7] + digits * 2
[2, 7, 1, 8, 2, 8, 1, 8, 2, 8]
```

**for循环的执行过程：**

一个 `for` 循环语句由如下格式的单个子句组成：

```python
for <name> in <expression>:
    <suite>
```

`for` 循环语句按以下过程执行：

1. 执行头部（header）中的 `<expression>`，它必须产生一个可迭代（iterable）的值（译者注：可迭代的详细概念可见 *4.2 隐式序列*）
2. 对该可迭代值中的每个元素，按顺序： 
   1. 将当前帧的 `<name>` 绑定到该元素值
   2. 执行 `<suite>`

**序列解包**

**range (start, end(unincluded), step)**

```py
>>> list(range(5, 8))
[5, 6, 7]
```

范围通常出现在 `for` 循环头部中的表达式，以指定 `<suite>` 应执行的次数。一个惯用的使用方式是：如果 `<name>` 没有在 `<suite>` 中被使用到，则用下划线字符 "_" 作为 `<name>`。

```py
>>> for _ in range(3):
        print('Go Bears!')

Go Bears!
Go Bears!
Go Bears!
```

对解释器而言，这个下划线只是环境中的另一个名称，但对程序员具有约定俗成的含义，表示该名称不会出现在任何未来的表达式中。

#### 1.2.3.2序列的处理

> 序列是复合数据的一种常见形式，常见到整个程序都可能围绕着这个单一的抽象来组织。
>
> 比如malloc函数在堆内存上的分配。

**列表推导式（List Comprehensions）**

```py
>>> odds = [1, 3, 5, 7, 9]
>>> [x+1 for x in odds]
[2, 4, 6, 8, 10]
>>> [x for x in odds if 25 % x == 0]
[1, 5]
```

一般推导的形式：

```py
[<map expression> for <name> in <sequence expression> if <filter expression>]
```

#### 1.2.3.3序列的抽象

成员资格：

```py
>>> digits
[1, 8, 2, 8]
>>> 2 in digits
True
>>> 1828 not in digits
True
```

slicing 切片：选取特定的范围，也可以选取step

```py
>>> digits[0:2]
[1, 8]
>>> digits[1:]
[8, 2, 8]
```

> 这里看完可以先做lab03理解一下。
>
> 整个lab03是比较简单的，都是最简单的递归函数。

#### 1.2.3.4Tree

> ​	使用列表包含其他列表，闭包（Closure Property）属性。
>
> ​	「数学中，若对某个集合的成员进行一种**运算**，生成的仍然是这个集合的成员，则该集合被称为**在这个运算下闭合**。」

就是tree，没什么说的，这个很好理解。

遍历处理Tree上的所有数字，注意使用`isinstsnce`来判断是否是某个类型对应的实例。

```py
def deep_map(f, s):
    """Replace all non-list elements x with f(x) in the nested list s.

    >>> six = [1, 2, [3, [4], 5], 6]
    >>> deep_map(lambda x: x * x, six)
    >>> six
    [1, 4, [9, [16], 25], 36]
    >>> # Check that you're not making new lists
    >>> s = [3, [1, [4, [1]]]]
    >>> s1 = s[1]
    >>> s2 = s1[1]
    >>> s3 = s2[1]
    >>> deep_map(lambda x: x + 1, s)
    >>> s
    [4, [2, [5, [2]]]]
    >>> s1 is s[1]
    True
    >>> s2 is s1[1]
    True
    >>> s3 is s2[1]
    True
    """
    "*** YOUR CODE HERE ***"
    for index in range(len(s)):
        if not isinstance(s[index], list):
            s[index] = f(s[index])
        else:
            deep_map(f, s[index])
```

> 这里要读一读官网的手册，要不然会看不懂在干什么。
>
> 比较像简单的leetcode问题。

#### 1.2.3.5LinkedList

> 我们再次来理解一下什么是链表。

```python
four = [1, [2, [3, [4, 'empty']]]]
```

我们把这样的一个嵌套的序列理解成一个链表。

其实和tree的构成是类似的。

那么这个链表的方法就应该这样定义：

```py
>>> empty = 'empty'
>>> def is_link(s):
        """判断 s 是否为链表"""
        return s == empty or (len(s) == 2 and is_link(s[1]))

>>> def link(first, rest):
        """用 first 和 rest 构建一个链表"""
        assert is_link(rest), " rest 必须是一个链表"
        return [first, rest]

>>> def first(s):
        """返回链表 s 的第一个元素"""
        assert is_link(s), " first 只能用于链表"
        assert s != empty, "空链表没有第一个元素"
        return s[0]

>>> def rest(s):
        """返回 s 的剩余元素"""
        assert is_link(s), " rest 只能用于链表"
        assert s != empty, "空链表没有剩余元素"
        return s[1]
```

### 1.2.4可变数据

1.对象OOP的基本概念。

2.可变对象---》列表

比较典型的概念，检查是alias还是copy。

```python
>>> suits is nest[0]
True
>>> suits is ['heart', 'diamond', 'spade', 'club']
False
>>> suits == ['heart', 'diamond', 'spade', 'club']
True
```

tuple元组：

**不可变**对象。

```python
>>> 1, 2 + 3
(1, 5)
>>> ("the", 1, ("and", "only"))
('the', 1, ('and', 'only'))
>>> type( (10, 20) )
<class 'tuple'>
```

类似的语法：

```python
>>> code = ("up", "up", "down", "down") + ("left", "right") * 2
>>> len(code)
8
>>> code[3]
'down'
>>> code.count("down")
2
>>> code.index("left")
4
```

但是不能和list一样对于序列进行自由的操作。

#### 1.2.4.1字典（Dictionary）

其实就是**HashTable**？

```py
>>> numerals = {'I': 1.0, 'V': 5, 'X': 10}
>>> numerals['X']
10
```

字典本身是乱序的。

> ​	Python 3.7 及以上版本的字典顺序会确保为插入顺序，此行为是自 3.6 版开始的 CPython 实现细节，字典会保留插入时的顺序，对键的更新也不会影响顺序，删除后再次添加的键将被插入到末尾

字典类型也有一些限制：

- 字典的 key **不可以是可变数据，也不能包含可变数据**。

- 一个 key 只能对应一个 value。

  get方法：

```py
>>> numerals.get('A', 0)
0
>>> numerals.get('V', 0)
5
```

推导式创建dic的语法：

```py
>>> {x: x*x for x in range(3,6)}
{3: 9, 4: 16, 5: 25}
```

> 先来做lab04,我们再继续来研究理论内容。

例子：嵌套构建一个dic。

```py
def divide(quotients, divisors):
    """Return a dictonary in which each quotient q is a key for the list of
    divisors that it divides evenly.

    >>> divide([3, 4, 5], [8, 9, 10, 11, 12])
    {3: [9, 12], 4: [8, 12], 5: [10]}
    >>> divide(range(1, 5), range(20, 25))
    {1: [20, 21, 22, 23, 24], 2: [20, 22, 24], 3: [21, 24], 4: [20, 24]}
    """
    return {x: [y for y in divisors if y % x == 0] for x in quotients}
```

下一个纯自己实现确实有点复杂。

> 使用**nonlocal**引用闭包之外的变量。

```py
def buy(fruits_to_buy, prices, total_amount):
    """Print ways to buy some of each fruit so that the sum of prices is amount.

    >>> prices = {'oranges': 4, 'apples': 3, 'bananas': 2, 'kiwis': 9}
    >>> buy(['apples', 'oranges', 'bananas'], prices, 12)  # We can only buy apple, orange, and banana, but not kiwi
    [2 apples][1 orange][1 banana]
    >>> buy(['apples', 'oranges', 'bananas'], prices, 16)
    [2 apples][1 orange][3 bananas]
    [2 apples][2 oranges][1 banana]
    >>> buy(['apples', 'kiwis'], prices, 36)
    [3 apples][3 kiwis]
    [6 apples][2 kiwis]
    [9 apples][1 kiwi]
    """
    
    ans_count = 0

    def add(fruits, amount, cart):
        nonlocal ans_count

        if not fruits:
            if amount == 0:
                for fruit, count in cart.items():
                    if count == 0:
                        return
                
                ans_count += 1
                if ans_count > 1:
                    print()
                for fruit, count in cart.items():
                    if count == 1:
                        s = fruit[:-1]
                    else:
                        s = fruit[:]
                    print("[" + str(count) + " " + s + "]", end = "")    
            return 

        fruit = fruits[0]
        price = prices[fruit]
        count = amount // price

        for index in range(count + 1):
            cart[fruit] = index
            add(fruits[1:], amount - price * index, cart)
            cart[fruit] = 0
    
    cart = {fruit : 0 for fruit in fruits_to_buy}
    add(fruits_to_buy, total_amount, cart)
```

相对来说比较简单，能理解字典的工作原理即可。

> 接着来hw04，稍微有点难度，考察对于Tree的理解。

一个交错洗牌的函数，**zip**就是把两个list对应位置的元素合成一个tuple元组。



```py
def shuffle(s):
    """Return a shuffled list that interleaves the two halves of s.

    >>> shuffle(range(6))
    [0, 3, 1, 4, 2, 5]
    >>> letters = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h']
    >>> shuffle(letters)
    ['a', 'e', 'b', 'f', 'c', 'g', 'd', 'h']
    >>> shuffle(shuffle(letters))
    ['a', 'c', 'e', 'g', 'b', 'd', 'f', 'h']
    >>> letters  # Original list should not be modified
    ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h']
    """
    assert len(s) % 2 == 0, 'len(seq) must be even'
    "*** YOUR CODE HERE ***"
    # 交错洗牌
    mid = len(s) // 2
    list1 = s[:mid]
    list2 = s[mid:]
    res = []
    for num1, num2 in zip(list1, list2):
        res.extend((num1, num2))
    return res
```

> 接下来我们会接触大量概念性的东西。

#### 1.2.4.2局部状态 

​	列表和字典拥有局部状态（local state），即它们可以在程序执行过程中的某个时间点修改自身的值。状态（state）就意味着当前的值有可能发生变化。

​	函数也有状态，会改变状态的就不叫纯函数，很多coding中令人疑惑的错误就是没有理解函数状态的改变造成的，比如**iterator**迭代器。

> 就是不要在迭代的过程中修改原来的序列。

> 注意这里的nonlocal,一旦是非局部的，我们不会把这个balance的值和局部帧绑定起来。
>
> Python 中 **nonlocal** 声明的效果：当前执行帧之外的变量可以通过赋值语句更改。

```py
>>> def make_withdraw(balance):
        """返回一个每次调用都会减少 balance 的 withdraw 函数"""
        def withdraw(amount):
            nonlocal balance                 # 声明 balance 是非局部的
            if amount > balance:
                return '余额不足'
            balance = balance - amount       # 重新绑定
            return balance
        return withdraw
```

​	通过引入非局部语句，我们为赋值语句创建了双重作用。他们可以更改局部绑定 (local bindings)，也可以更改非局部绑定  (nonlocal  bindings)。事实上，赋值语句已经有了很多作用：它们可以创建新的变量，也可以为现有变量重新赋值。赋值也可以改变列表和字典的内容。Python 中赋值语句的多种作用可能会使执行赋值语句时的效果变得不太明显。作为程序员，我们有责任清楚地记录代码，以便其他人可以理解赋值的效果。

> 那么有的时候，赋值语句高不清楚会让人很迷惑。

​	**Python 特质 (Python Particulars)**。这种非局部赋值模式是具有高阶函数和词法作用域的编程语言的普遍特征。大多数其他语言根本不需要非局部语句。相反，非局部赋值通常是赋值语句的默认行为。

​	**Python** 在变量名称查找方面也有一个不常见的限制：在一个函数体内，**多次出现的同一个变量名必须处于同一个运行帧**内。因此，**Python**  无法在非局部帧中查找某个变量名对应的值，然后在局部帧中为同样名称的变量赋值，因为同名变量会在同一函数的两个不同帧中被访问。此限制允许  Python 在执行函数体之前预先计算哪个帧包含哪个名称。当代码违反了这个限制时，程序会产生令人困惑的错误消息。

​	正确理解包含 **nonlocal** 声明的代码的关键是记住：只有函数调用才能引入新帧。赋值语句只能更改现有帧中的绑定关系。

​	**相同与变化 (Sameness and change)**。这些微妙之处的出现是因为，通过引入改变非局部环境的非纯函数，我们改变了表达式的性质。仅包含纯函数调用的表达式是引用透明 (referentially transparent) 的；即如果在函数中，用一个等于子表达式的值来替换子表达式，它的值不会改变。

​	重新绑定操作违反了引用透明的条件，因为它们不仅仅是返回一个值；他们还会在执行过程中改变运行环境。当我们引入任意的重新绑定时，我们遇到了一个棘手的认识论问题：两个值相同意味着什么。在我们的计算环境模型中，两个单独定义的函数是不同的，因为对一个函数的更改可能不会反映在另一个函数中。

> 关于这样的操作，我们不能随便绑定和引入别名。

### 1.2.5列表和字典实现 

> 状态就意味着对象的存在。
>
> 带状态的函数就是一个对象。

```py
>>> def mutable_link():
        """返回一个可变链表的函数"""
        contents = empty
        def dispatch(message, value=None):
            nonlocal contents
            if message == 'len':
                return len_link(contents)
            elif message == 'getitem':
                return getitem_link(contents, value)
            elif message == 'push_first':
                contents = link(value, contents)
            elif message == 'pop_first':
                f = first(contents)
                contents = rest(contents)
                return f
            elif message == 'str':
                return join_link(contents, ", ")
        return dispatch
```

​	**dispatch** 函数是实现抽象数据消息传递接口的通用方法。为实现消息分发，到目前为止，我们使用条件语句将消息字符串与一组固定的已知消息进行比较。

> 注意下面这样的实现方式，不用ifelse,直接把str和定义的函数名称绑定起来。
>
> 而是使用字典，这就是**调度字典**。---》避免使用了nonlocal定义。

```py
def account(initial_balance):
    def deposit(amount):
        dispatch['balance'] += amount
        return dispatch['balance']
    def withdraw(amount):
        if amount > dispatch['balance']:
            return 'Insufficient funds'
        dispatch['balance'] -= amount
        return dispatch['balance']
    dispatch = {'deposit':   deposit,
                'withdraw':  withdraw,
                'balance':   initial_balance}
    return dispatch

def withdraw(account, amount):
    return account['withdraw'](amount)
def deposit(account, amount):
    return account['deposit'](amount)
def check_balance(account):
    return account['balance']

a = account(20)
deposit(a, 5)
withdraw(a, 17)
check_balance(a)
```

### 1.2.6约束传递 (Propagating Constraints) 

> ​	传统的计算机计算是单向的，但是如果要计算p * v = n * k * t这样的一个对象，我们要怎么处理。

​	比较复杂，但是简单来说还是一种编程方式，一种参数网络，更改其中的参数会对于网络中的其余部分产生影响。

> 我们来做lab05。

关于对象的引用

is	是判断同一个对象。

==	判断内容是否相同。

理解这个过程中在干什么？

```py
>>> s = [3,4,5]
>>> s.extend([s.append(9), s.append(10)])
>>> s
[3, 4, 5, 9, 10, None, None]
```

​	然后是`iter()`函数，关于序列的迭代器的讨论。

​	这个lab要对于上面提到的概念非常熟悉。

> ​	简单的示例代码，我把代码放在这里是为了当我看到的时候，知道该怎么使用和要注意些什么东西。

```py
   grouped = {}
    for x in s:
        key = fn(x)
        if key in grouped:
            grouped[key].append(x)
        else:
            grouped[key] = [x]
    return grouped
```

> 利用强大的切片功能。

```py
def partial_reverse(s, start):
    """Reverse part of a list in-place, starting with start up to the end of
    the list.

    >>> a = [1, 2, 3, 4, 5, 6, 7]
    >>> partial_reverse(a, 2)
    >>> a
    [1, 2, 7, 6, 5, 4, 3]
    >>> partial_reverse(a, 5)
    >>> a
    [1, 2, 7, 6, 5, 3, 4]
    """
    "*** YOUR CODE HERE ***"
    s[start:] = s[start:][::-1]
```

> 接下来做homework05的内容。
>
> 这里是使用关键字**yield**来生成数字：处理任务调度，复杂迭代，超大文件等等。
>
> 你可以直接理解成一个可以自定义行为的迭代器。

```py
def hailstone(n):
    """
    Yields the elements of the hailstone sequence starting at n.
    At the end of the sequence, yield 1 infinitely.

    >>> hail_gen = hailstone(10)
    >>> [next(hail_gen) for _ in range(10)]
    [10, 5, 16, 8, 4, 2, 1, 1, 1, 1]
    >>> next(hail_gen)
    1
    """
    "*** YOUR CODE HERE ***"
    # 返回一个长度为n的数组的迭代器,怎么做到一直返回1？
    # 使用yield关键字进行生成：暂时返回，懒惰生成数据。
    while n != 1:
      yield n
      if n % 2 == 1:
        n = 3 * n + 1
      else:
        n = n // 2
    
    while True:
      yield 1
```

> 虽然是简单的爬楼梯问题，但是还是需要思考。
>
> 主要是要理解生成器迭代时候的行为。

```py
def stair_ways(n):
    """
    Yield all the ways to climb a set of n stairs taking
    1 or 2 steps at a time.

    >>> list(stair_ways(0))
    [[]]
    >>> s_w = stair_ways(4)
    >>> sorted([next(s_w) for _ in range(5)])
    [[1, 1, 1, 1], [1, 1, 2], [1, 2, 1], [2, 1, 1], [2, 2]]
    >>> list(s_w) # Ensure you're not yielding extra
    []
    """
    "*** YOUR CODE HERE ***"
    # 虽然是简单的问题，当涉及到生成器的时候，我们就从生成器的行为上来考虑
    if n == 0:
        yield []
    elif n > 0:
        for way in stair_ways(n - 1):
            yield [1] + way
        for way in stair_ways(n - 2):
            yield [2] + way
```

### 1.2.7面向对象编程 OOP

> ​	关于py的**OOP**部分，我一直关于这个不是很理解，在此之前我已经接触了Java和Cpp的OOP部分，我认为想要学明白OOP,一定要在工程实践中才能理解透彻。再多的理论都很难让人理解具体的用法和为什么要这样使用，**设计模式**这样的东西也是一样，总之我们来看一看。

#### 1.2.7.1对象和类

类是模板，对象是初始化的实例。

对象具有方法和属性。

#### 1.2.7.2类的定义 

```py
class <name>:
    <suite>
```

> 基本概念。

构造函数：

```py
>>> class Account:
        def __init__(self, account_holder):
            self.balance = 0
            self.holder = account_holder
```

​	一般来说，我们使用参数名称 `self` 作为**构造函数的第一个参数**，它**会自动绑定到正在实例化的对象**。几乎所有的 Python 代码都遵守这个规定。这就类似于Java中的this，注意py中的语法。

> 对象的独立性。

```py
>>> a is a
True
>>> a is not b
True
```

我们怎么定义方法：

```py
>>> class Account:
    # 构造函数
        def __init__(self, account_holder):
            self.balance = 0
            self.holder = account_holder
        def deposit(self, amount):
            self.balance = self.balance + amount
            return self.balance
        def withdraw(self, amount):
            if amount > self.balance:
                return 'Insufficient funds'
            self.balance = self.balance - amount
            return self.balance
```

​	为了使用点表达式，我们的方法都要使用**self首参**：每个方法都包含着一个特殊的首参 `self` ，该参数绑定调用该方法的对象。例如，假设在特定的 `Account` 对象上调用 `deposit` 并传递单个参数：存入的金额。对象本身就被绑定到 `self` ，而传入的参数绑定到 `amount` 。所有调用的方法都可以通过 `self` 参数来访问对象，因此它们**都可以访问和操作对象的状态**。

#### 1.2.7.3消息传递和点表达式 

​	**方法和函数**：在对象上调用方法时，该对象将作为第一个参数隐式传递给该方法。也就是说，点左侧的 `<expression>` 值的对象将自动作为第一个参数传递给点表达式右侧命名的方法。因此，对象绑定到参数 `self`。

​	为了实现自动 `self` 绑定，Python 区分了我们从文本开头就一直在创建的函数和绑定方法，它们将函数和将调用该方法的对象耦合在一起。绑定方法值已与其第一个参数（调用它的实例）相关联，在调用该方法时将命名为 `self`。

​	作为一个类的属性，这是一个函数，但是作为一个实例的属性，这是一个方法（自动绑定的）。

```py
>>> type(Account.deposit)
<class 'Function'>
>>> type(spock_account.deposit)
<class 'method'>
```

​	这两个结果的区别仅在于第一个是参数为 `self` 和 `amount` 的标准双参数函数。第二种是单参数方法，调用方法时，名称 `self` 将自动绑定到名为 `spock_account` 的对象，而参数 `amount` 将绑定到传递给方法的参数。这两个值（无论是函数值还是绑定方法值）都与相同的 `deposit` 函数体相关联。

​	那么我们就可以这样使用方法。

```py
>>> Account.deposit(spock_account, 1001)    # 函数 deposit 接受两个参数
1011
>>> spock_account.deposit(1000)             # 方法 deposit 接受一个参数
2011
```

> ​	**命名约定**：类名通常使用 CapWords 约定（也称为 CamelCase，因为名称中间的大写字母看起来像驼峰）编写。方法名称遵循使用下划线分隔的小写单词命名函数的标准约定。
>
> ​	在某些情况下，有一些实例变量和方法与对象的维护和一致性相关，我们不希望对象的用户看到或使用。它们不是类定义的抽象的一部分，而是实现的一部分。Python 的约定规定，如果属性名称以下划线开头，则只能在类本身的方法中访问它，而不是用户访问。（这就是私有的方法）

#### 1.2.7.4类属性

就是静态变量，可以认为是共有的。

```py
>>> class Account:
        interest = 0.02            # 类属性
        def __init__(self, account_holder):
            self.balance = 0
            self.holder = account_holder
        # 在这里定义更多的方法
```

如果同名，我们优先考虑实例的属性，接着再考虑类属性：

1. 点表达式左侧的 `<expression>` ，生成点表达式的对象。
2. `<name>` 与该对象的实例属性匹配；如果存在具有该名称的属性，则返回属性值。
3. 如果实例属性中没有 `<name>` ，则在类中查找 `<name>`，生成类属性。
4. 除非它是函数，否则返回属性值。如果是函数，则返回该名称绑定的方法。

#### 1.2.7.5继承 

​	在面向对象编程范式中，我们经常会发现不同类型之间存在关联，尤其是在类的专业化程度上。即使两个类具有相似的属性，它们的特殊性也可能不同，为了实现逻辑上相同的类之间的差异性。

​	`CheckingAccount` 是 `Account` 的特化。在 OOP 术语中，通用帐户将用作 `CheckingAccount` 的基类，而 `CheckingAccount` 将用作 `Account` 的子类。术语基类（base class）也常叫父类（parent class）和超类（superclass），而子类（subclass）也叫孩子类（child class）。

我们怎么从基类做继承的操作：

```py
>>> class Account:
        """一个余额非零的账户。"""
        interest = 0.02
        def __init__(self, account_holder):
            self.balance = 0
            self.holder = account_holder
        def deposit(self, amount):
            """存入账户 amount，并返回变化后的余额"""
            self.balance = self.balance + amount
            return self.balance
        def withdraw(self, amount):
            """从账号中取出 amount，并返回变化后的余额"""
            if amount > self.balance:
                return 'Insufficient funds'
            self.balance = self.balance - amount
            return self.balance
```



```py
>>> class CheckingAccount(Account):
        """从账号取钱会扣出手续费的账号"""
        withdraw_charge = 1
        interest = 0.01
        def withdraw(self, amount):
            return Account.withdraw(self, amount + self.withdraw_charge)
```

对于基类的方法进行重写。

关于多重继承：

```py
>>> class SavingsAccount(Account):
        deposit_charge = 2
        def deposit(self, amount):
            return Account.deposit(self, amount - self.deposit_charge)
```

同时继承两个基类，当我们使用这个类的方法的时候，它自动就会使用两个基类中重写的方法。

```py
>>> class AsSeenOnTVAccount(CheckingAccount, SavingsAccount):
        def __init__(self, account_holder):
            self.holder = account_holder
            self.balance = 1           # 赠送的 1 $!
```

那么重名？有具体的方法解析的顺序。

#### 1.2.7.6对象的作用 

> ​	多范式语言，如 **Python**，允许程序员将组织范式与适当的问题相匹配。学会识别何时引入新类，而不是新函数，以简化或模块化程序，是软件工程中一项重要的设计技能，值得认真关注。
>
> ​	接下来是lab06,关于OOP的基本概念。

```py
class Mint:
    """A mint creates coins by stamping on years.

    The update method sets the mint's stamp to Mint.present_year.

    >>> mint = Mint()
    >>> mint.year
    2024
    >>> dime = mint.create(Dime)
    >>> dime.year
    2024
    >>> Mint.present_year = 2104  # Time passes
    >>> nickel = mint.create(Nickel)
    >>> nickel.year     # The mint has not updated its stamp yet
    2024
    >>> nickel.worth()  # 5 cents + (80 - 50 years)
    35
    >>> mint.update()   # The mint's year is updated to 2104
    >>> Mint.present_year = 2179     # More time passes
    >>> mint.create(Dime).worth()    # 10 cents + (75 - 50 years)
    35
    >>> Mint().create(Dime).worth()  # A new mint has the current year
    10
    >>> dime.worth()     # 10 cents + (155 - 50 years)
    115
    >>> Dime.cents = 20  # Upgrade all dimes!
    >>> dime.worth()     # 20 cents + (155 - 50 years)
    125
    """
    present_year = 2024

    def __init__(self):
        self.update()

    def create(self, coin):
        "*** YOUR CODE HERE ***"
        return coin(self.year)


    def update(self):
        "*** YOUR CODE HERE ***"
        self.year = Mint.present_year

class Coin:
    cents = None # will be provided by subclasses, but not by Coin itself

    def __init__(self, year):
        self.year = year

    def worth(self):
        "*** YOUR CODE HERE ***"
        if Mint.present_year - self.year - 50 > 0:
            return self.cents + Mint.present_year - self.year - 50
        else:
            return self.cents    
class Nickel(Coin):
    cents = 5

class Dime(Coin):
    cents = 10
```

​	比较简单的关于继承的逻辑，只有几行代码，但是一定要搞清楚在干嘛。

> 接下来我们做pro中的cats部分。
>
> 有点意思，测试打字速度并且有自动纠错的功能。
>
> 还要实现多个玩家的功能。

```OK
Point breakdown
    Problem 1: 1.0/1
    Problem 2: 1.0/1
    Problem 3: 2.0/2
    Problem 4: 1.0/1
    Problem 5: 2.0/2
    Problem 6: 3.0/3
    Problem 7: 3.0/3
    Problem 8: 2.0/2
    Problem 9: 1.0/1
    Problem 10: 3.0/3

Score:
    Total: 19.0
```

> ​	ok,那么也是拿下了，整体也不难，主要就是有一个编辑距离的问题和熟悉基本的语法。
>
> 不做附加问题了，主要就是装饰器，memo优化的内容，来不及了。
>
> ​	接下来我们先把第二章的内容看完。再去做作业之类的。

### 1.2.8实现类和对象

​	用字典可以实现,并且函数可以实现一切.

​	有点抽象,总之就是使用调度字段来实现功能.

### 1.2.9对象抽象

#### 1.2.9.1字符串转换

抽象数据描述--->泛型函数

py对于一个原始值的字符串表示:

```py
>>> 12e12
12000000000000.0
>>> print(repr(12e12))
12000000000000.0
```

如果没有这个原始值,就是一个尖括号的字符串描述:

```py
>>> repr(min)
'<built-in function min>'
```

str构造器不相同的地方:

```py
>>> from datetime import date
>>> tues = date(2011, 9, 12)
>>> repr(tues)
'datetime.date(2011, 9, 12)'
>>> str(tues)
'2011-09-12'
```

​	定义 `repr` 函数带来了一个新的挑战：我们想要它正确地应用于所有的数据类型，即使是那些实现 `repr` 时还不存在的类型。我们希望它是一个**通用的或者多态（polymorphic）的函数**，可以被应用于数据的多种（多）不同形式（态）。

​	在这情况下，对象系统提供了一种优雅的解决方案：`repr` 函数总是在其参数值上调用一个名为 `__repr__` 的方法。

```py
>>> tues.__repr__()
'datetime.date(2011, 9, 12)'
```

​	通过在用户定义类中实现这个相同的方法，我们可以将 `repr` 函数的适用范围扩展到将来我们创建的任何类。这个例子突出了点表达式的另一个优势，那就是它们提供了一种机制，可以把现有的函数的作用域扩展到新的对象类型。

#### 1.2.9.2专用方法

> ​	在 Python 中，某些特殊名称会在特殊情况下被 Python 解释器调用。例如，类的 `__init__` 方法会在对象被创建时自动调用。`__str__` 方法会在打印时自动调用，`__repr__` 方法会在交互式环境显示其值的时候自动调用。

​	这就是"专用".

​	一般任务一个对象是True,但是可以定义__bool__,当我们想按照自己的逻辑判断真假的时候,就会自动调用这个函数.

```py
>>> Account.__bool__ = lambda self: self.balance != 0
>>> bool(Account('Jack'))
False
>>> if not Account('Jack'):
        print('Jack has nothing')
Jack has nothing
```

比如len:

```py
>>> len('Go Bears!')
9
```

实际上是因为所有序列的内置类型都设置了__len__的方法:

```py
>>> 'Go Bears!'.__len__()
9
```

`__getitem__` 方法由元素选择操作符调用，但也可以直接调用它。

```py
>>> 'Go Bears!'[3]
'B'
>>> 'Go Bears!'.__getitem__(3)
'B'
```

考虑这个高阶函数:

```py
>>> def make_adder(n):
        def adder(k):
            return n + k
        return adder

>>> add_three = make_adder(3)
>>> add_three(4)
7
```

可以定义一个类,看起来就像是一个高阶函数,利用__call__方法:

```py
>>> class Adder(object):
        def __init__(self, n):
            self.n = n
        def __call__(self, k):
            return self.n + k
>>> add_three_obj = Adder(3)	# 创建一个对象的时候会调用__call__方法
>>> add_three_obj(4)
7
```

> ​	算术运算。特定的方法也可以定义应用在用户定义的对象上的内置操作符的行为。为了提供这种通用性，Python 遵循特定的协议来应用每个操作符。例如，为了计算包含 `+` 操作符的表达式，Python 会检查操作符左右两侧的运算对象上是否有特定的方法。首先 Python 在左侧运算对象上检查其是否有 `__add__` 方法，然后在右侧运算对象上检查其是否有 `__radd__` 方法。如果找到了其中一个方法，就会将另一个运算对象的值作为它的参数来调用这个方法。

#### 1.2.9.3多重表示

​	对于一个相同的数据对象,在不同的场景下我们可能想使用不同的表示方法,比如复数.

> ​	实现的逻辑是自顶向下的.

比如实现复数系统,这是一个超类:

```py
>>> class Number:
        def __add__(self, other):
            return self.add(other)
        def __mul__(self, other):
            return self.mul(other)
```

这是复数,具体来实现add和mul的逻辑:

```py
>>> class Complex(Number):
        def add(self, other):
            return ComplexRI(self.real + other.real, self.imag + other.imag)
        def mul(self, other):
            magnitude = self.magnitude * other.magnitude
            return ComplexMA(magnitude, self.angle + other.angle)
```

这个实现假定存在两个表示复数的类，分别对应他们的两种自然表示形式。

- `ComplexRI` 使用实部和虚部构建一个复数。
- `ComplexMA` 使用幅度和角度构建一个复数。

​	我们怎么保证多重属性之间是共通的,比如实部和虚部发生变化的时候,对应的弧度和幅值也会发生相应的变化.

> ​	Python 有一种简单的计算属性的特性，可以通过零参数函数实时的计算属性。`@property` 修饰符允许函数在没有调用表达式语法（表达式后跟随圆括号）的情况下被调用。`Complex` 类存储了 `real` 和 `imag` 属性并在需要时计算 `magnitude` 和 `angle` 属性。

实际上这是一种计算出来的属性:

```py
>>> from math import atan2
>>> class ComplexRI(Complex):
        def __init__(self, real, imag):
            self.real = real
            self.imag = imag
        @property
        def magnitude(self):
            return (self.real ** 2 + self.imag ** 2) ** 0.5
        @property
        def angle(self):
            return atan2(self.imag, self.real)
        def __repr__(self):
            return 'ComplexRI({0:g}, {1:g})'.format(self.real, self.imag)
```

调用的时候:

```py
>>> ri = ComplexRI(5, 12)
>>> ri.real
5
>>> ri.magnitude
13.0
>>> ri.real = 9
>>> ri.real
9
>>> ri.magnitude
15.0
```

​	@property是用本身的属性在调用的时候可以进行零参数的实时计算,他并没有改变什么具有状态的值.

> ​	实际上我认为这并没有达到我想象中的效果,因为**本质上还是持有不同的类**.

> ​	使用接口来编码多重表示具有十分吸引人的特点。**每一种表示形式的类都可以被单独开发**；它们只需要就它们共享的属性名称和和对于这些属性的行为条件达成一致。接口还具有可添加性。如果程序员想要添加一个复数的第三方表现形式到同一个程序中，他们只需要创建一个拥有相同属性名称的类即可。

但是一定要约定相同的转换方式,但是如果有三个或者多个表示形式的时候,这样的表示方式还合适么?

#### 1.2.9.4泛型函数

​	我们上面的Complex.add的方法就是一个泛型的函数,因为可以接受两个类类型的参数.

> ​	使用接口和消息传递只是多种可以被用于实现泛型函数的方法中的一种。在本节我们将会考虑另外两个方法：**类型派发和类型强制转换**。

比如我们加上一个Rational类来实现Number:

```py
>>> from fractions import gcd
>>> class Rational(Number):
        def __init__(self, numer, denom):
            g = gcd(numer, denom)
            self.numer = numer // g
            self.denom = denom // g
        def __repr__(self):
            return 'Rational({0}, {1})'.format(self.numer, self.denom)
        def add(self, other):
            nx, dx = self.numer, self.denom
            ny, dy = other.numer, other.denom
            return Rational(nx * dy + ny * dx, dx * dy)
        def mul(self, other):
            numer = self.numer * other.numer
            denom = self.denom * other.denom
            return Rational(numer, denom)
```

这实现了分数的add和mul.

那么可以实现的是:

```py
>>> Rational(2, 5) + Rational(1, 10)
Rational(1, 2)
>>> Rational(1, 4) * Rational(2, 3)
Rational(1, 6)
```

但是我们想让分数和复数进行加减,虽然数学上有定义,但是现在暂时还是不支持.

> ​	**类型派发**。一种实现跨类型操作的方式是选择**基于函数或方法的参数类型来选择相应的行为**。类型派发的思想是写一个**能够检查它所收到的参数的类型的函数**，然后**根据参数类型执行恰当的代码**。

根据不同类型采取不同的行为.

​	内置的函数 `isinstance` 接受一个对象或一个类。如果对象的类是所给的类或者继承自所给的类，它会返回一个真值。--->判断是否是对象的一个实例.

```py
>>> c = ComplexRI(1, 1)
>>> isinstance(c, ComplexRI)
True
>>> isinstance(c, Complex)
True
>>> isinstance(c, ComplexMA)
False
```

用isinstance就能针对不同的类型采取不同的行为:

```py
>>> def is_real(c):
        """Return whether c is a real number with no imaginary part."""
        if isinstance(c, ComplexRI):
            return c.imag == 0
        elif isinstance(c, ComplexMA):
            return c.angle % pi == 0

>>> is_real(ComplexRI(1, 1))
False
>>> is_real(ComplexMA(2, pi))
True
```

我们还可以使用一种属性来确定是否相同:

```py
>>> def is_real(c):
        """Return whether c is a real number with no imaginary part."""
        if isinstance(c, ComplexRI):
            return c.imag == 0
        elif isinstance(c, ComplexMA):
            return c.angle % pi == 0

>>> is_real(ComplexRI(1, 1))
False
>>> is_real(ComplexMA(2, pi))
True
```

那就可以写这样一个函数:

```py
>>> def add_complex_and_rational(c, r):
        return ComplexRI(c.real + r.numer/r.denom, c.imag)
```

然后:

```py
>>> def mul_complex_and_rational(c, r):
        r_magnitude, r_angle = r.numer/r.denom, 0
        if r_magnitude < 0:
            r_magnitude, r_angle = -r_magnitude, pi
        return ComplexMA(c.magnitude * r_magnitude, c.angle + r_angle)
```

add和mul基本定义,然后交换顺序:

```py
>>> def add_rational_and_complex(r, c):
        return add_complex_and_rational(c, r)
>>> def mul_rational_and_complex(r, c):
        return mul_complex_and_rational(c, r)
```

这样看起来是否很臃肿,接下来我们实现类型派发:(更改的是超类Number)

```py
>>> class Number:
        def __add__(self, other):
            if self.type_tag == other.type_tag:
                return self.add(other)
            elif (self.type_tag, other.type_tag) in self.adders:
                return self.cross_apply(other, self.adders)
        def __mul__(self, other):
            if self.type_tag == other.type_tag:
                return self.mul(other)
            elif (self.type_tag, other.type_tag) in self.multipliers:
                return self.cross_apply(other, self.multipliers)
        def cross_apply(self, other, cross_fns):
            cross_fn = cross_fns[(self.type_tag, other.type_tag)]
            return cross_fn(self, other)
        adders = {("com", "rat"): add_complex_and_rational,
                    ("rat", "com"): add_rational_and_complex}
        multipliers = {("com", "rat"): mul_complex_and_rational,
                        ("rat", "com"): mul_rational_and_complex}
```

那么如果有新的子类,就可以在字典里面再加入一些值.

那么就可以实现功能:

```py
>>> ComplexRI(1.5, 0) + Rational(3, 2)
ComplexRI(3, 0)
>>> Rational(-1, 2) * ComplexMA(4, pi/2)
ComplexMA(2, 1.5 * pi)
```

那么强制类型转换其实也很简单了:

```py
>>> def rational_to_complex(r):
        return ComplexRI(r.numer/r.denom, 0)
```

在可以转换的情况下,可以直接返回一个实部是分数值,虚部是0的一个虚数.

> ​	这一节比较长,但是不难理解.

### 1.2.10效率

记忆化	时空复杂度的问题

### 1.2.11递归对象

链表,树和集合

> ​	后面两篇没咋看,直接做lab,先做ants(https://insideempire.github.io/CS61A-Website-Archive/proj/ants/index.html).主要就是OOP的思想实现一个类似于植物大战僵尸的小游戏,感觉也比较有意思,我们来体验一下OOP.

​	这个unlock部分就好像是在做什么任务型阅读一样.

​	游戏确实还是比较有意思的,基本就是读逻辑,然后去实现,比PVZ简单很多.

> ​	要注意更改list的浅拷贝遍历的问题,以及不要打破抽象屏障才是规范的.
>
> ​	技巧就是在浅拷贝上面遍历,在原来要修改的地方进行修改.

```py
def reduce_health(self, amount):
        """Reduce health by AMOUNT, and remove the FireAnt from its place if it
        has no health remaining.

        Make sure to reduce the health of each bee in the current place, and apply
        the additional damage if the fire ant dies.
        """
        # BEGIN Problem 5
        "*** YOUR CODE HERE ***"
        # 尽力不打破抽象屏障
        damage_to_bees = amount
        bees_with_fire = self.place.bees

        Ant.reduce_health(self, amount)
        if self.health <= 0:
            damage_to_bees += self.damage
        # 使用浅拷贝,注意操作的对象还是一样的,两个数组之间的对象不是独立的,所以才叫"浅"拷贝
        for bee in bees_with_fire[:]:
            Insect.reduce_health(bee, damage_to_bees)
            if bee.health <= 0:
                Insect.remove_from(bee, bee.place)
        # END Problem 5
```

> ​	注意我们在调用方法和参数的时候,self,以及类名对应.
>
> ​	比如在没有更改的情况下,self.damage的值和class持有的damage的值是相同的,但是一旦发生变化,那么就各自持有各自的值.

```py
#Problem 6
class HungryAnt(Ant):
    name = 'Hungry'
    implemented = True
    damage = 0
    food_cost = 4
    chew_cooldown = 3

    def __init__(self, health=1, cooldown=0):
        super().__init__(health)
        self.cooldown = cooldown

    def action(self, gamestate):
        if self.cooldown == 0:
            unlucky_bee = random_bee(self.place.bees)
            if unlucky_bee != None:
                Insect.reduce_health(unlucky_bee, unlucky_bee.health)
                self.cooldown = HungryAnt.chew_cooldown
        else:
            self.cooldown -= 1
```

> ​	OK,Ants结束,很好的体验了一下简单的OOP,附加就暂时不写了,时间比较紧张.

之后做lab08,hw05已经做过了,我们就结束了第二章.

简单力扣水平吧.



## 1.3计算机程序的解释

### 1.3.1引言

解释器	设计语言而不是仅仅是使用者,这是程序员的视角

> 用py写一个解释Scheme语言的解释器.



​	我们研究怎样设计一个解释器,核心是两个互递归函数,第一个求解环境中的表达式,第二个把函数应用于参数

Lab10:

REPL---读取,求值,打印循环

1.读取:

词法分析:把输入的字符串分解为token

语法分析:把这些token组织成数据结构,用来处理---pair

2.求值:

eval:评估表达式,当是调用函数的时候,用apply获取结果.

apply:评估之后的运算符用于参数,还会调用eval.

这是类似于那种前缀表达式之类的计算

> ​	这相当于是一个小的预习.整体的感觉就像下面一样,eval进行语法分析,并且调用具体的计算,apply根据不同的operand确定不同的计算方法,其中的部分还是会使用调用eval.

```py
def calc_eval(exp):
    # 这是进行语法分析
    if isinstance(exp, Pair):
        operator =  exp.first# UPDATE THIS FOR Q2, e.g (+ 1 2), + is the operator
        operands = exp.rest # UPDATE THIS FOR Q2, e.g (+ 1 2), 1 and 2 are operands
        if operator == 'and': # and expressions
            return eval_and(operands)
        elif operator == 'define': # define expressions
            return eval_define(operands)
        else: # Call expressions
            return calc_apply(calc_eval(operator), operands.map(calc_eval)) # UPDATE THIS FOR Q2, what is type(operator)?
    elif exp in OPERATORS:   # Looking up procedures
        return OPERATORS[exp]
    elif isinstance(exp, int) or isinstance(exp, bool):   # Numbers and booleans
        return exp
        # 用bindings实现变量的定义的问题
    elif exp in bindings: # CHANGE THIS CONDITION FOR Q4 where are variables stored?
        return bindings[exp] # UPDATE THIS FOR Q4, how do you access a variable?

def calc_apply(op, args):
    return op(args)

def floor_div(args):
    # 实现除法的逻辑,apply把'//'绑定到我们的这个方法上面去
    # 类似与链表,用循环处理
    curr = args
    res = curr.first
    while curr.rest != nil:
        res //= curr.rest.first
        curr = curr.rest
    return res


scheme_t = True   # Scheme's #t
scheme_f = False  # Scheme's #f

def eval_and(expressions):
    # 实现一个and表达式,注意所有表达式为True,返回最后一个表达式的值
    if expressions is nil:
        return scheme_t
    
    curr = expressions
    while curr.rest is not nil:
        value = calc_eval(curr.first)
        if value is scheme_f:
            return scheme_f
        curr = curr.rest
        # 理解pair是怎么工作的
    return calc_eval(curr.first)
        
bindings = {}

def eval_define(expressions):
    # 直接进行符号绑定,要先进行求值的操作
    # 注意要绑定的是一个值表达式,所以是exp.rest.first!!!
    variable = expressions.first
    value_exp = expressions.rest.first
    value = calc_eval(value_exp)
    bindings[variable] = value
    return variable
```







### 1.3.2函数式编程

​	Scheme 是 [Lisp](http://en.wikipedia.org/wiki/Lisp_(programming_language)) 的一个变种，而 Lisp 是继 [Fortran](http://en.wikipedia.org/wiki/Fortran) 之后仍然广受欢迎的第二古老的编程语言。Lisp 程序员社区几十年来持续蓬勃发展，[Clojure](http://en.wikipedia.org/wiki/Clojure) 等 Lisp 的新方言拥有现代编程语言中增长最快的开发人员社区。如果你想亲手试试本文中的例子，可以下载一个 [Scheme 的解释器](http://inst.eecs.berkeley.edu/~scheme/) 来操作。

#### 1.3.2.1表达式

前缀表达式:

```scheme
(+ (* 3 5) (- 10 6))
```

可以写在多行上面:

```scheme
(+ (* 3
      (+ (* 2 4)
         (+ 3 5)))
   (+ (- 10 7)
      6))
```

if表达式:

```scheme
(if <predicate> <consequent> <alternative>)
```

比较方法:

```py
(>= 2 1)
```

- `(and <e1> ... <en>)` 解释器会从左到右依次检查 `<e>` 表达式。一旦有一个 `<e>` 的结果是假，整个 `and` 表达式就直接返回假，并且剩下的 `<e>` 表达式不再检查。只有当所有 `<e>` 都是真时，`and` 表达式的返回值才是最后一个表达式的结果。
- `(or <e1> ... <en>)` 解释器会从左到右依次检查 `<e>` 表达式。一旦有一个 `<e>` 的结果是真，`or` 表达式就直接返回那个真值，并且剩下的 `<e>` 表达式不再检查。只有当所有 `<e>` 都是假时，`or` 表达式才返回假。
- `(not <e>)` 当 `<e>` 表达式的结果是假时，`not` 表达式就返回真，否则返回假。

#### 1.3.2.2定义

define可以定义一个新的变量.

类似于:

```scheme
(define (<name> <formal parameters>) <body>)

(define (square x) (* x x))

(square 21)

(square (+ 2 5))

(square (square 3))
```

嵌套定义 递归 局部定义:

```py
(define (sqrt x)
  (define (good-enough? guess)
    (< (abs (- (square guess) x)) 0.001))
  (define (improve guess)
    (average guess (/ x guess)))
  (define (sqrt-iter guess)
    (if (good-enough? guess)
        guess
        (sqrt-iter (improve guess))))
  (sqrt-iter 1.0))
(sqrt 9)
```

创建lambda表达式:(只是不用设置函数名)

```scheme
((lambda (x y z) (+ x y (square z))) 1 2 3)
```

#### 1.3.2.3复合类型

pair内置数据结构.

你可以这样访问,注意表示的方式.

```scheme
(define x (cons 1 2))
x
(1 . 2)
(car x)
1
(cdr x)
2
```

#### 1.3.2.4符号数据

引用符号a b而不是他们的值,怎么上符号.

```scheme
scm> (define a 1)
a
scm> (define b 2)
b
scm> (list a b)
(1 2)
scm> (list 'a b)
(a 2)
scm> (list 'a 'b)
(a b)
```

任何不被求值的表达式都是引用.

### 1.3.3异常 Exceptions

异常是一个对象实例,继承自某个BaseException类,常见用法就是构造实例然后抛出.

```py
>>> raise Exception(' An error occurred')
Traceback (most recent call last):
	File "<stdin>", line 1, in <module>
Exception: an error occurred
```

"<stdin>"表示这个异常是在交互会话中触发的,而并非来自于某个文件.

handling exceptions:

用try来处理:

```py
try:
	<try suite>
except <exception class> as <name>:
	<except suite>
```

​	在执行 try 语句时，`<try suite>` 总是立即执行。只有在执行 `<try suite>` 过程中发生异常时，except 子句的内容才会执行。每个 except 子句指定了要处理的特定异常类。例如，如果 `<exception class>` 是 `AssertionError`，那么在执行 `<try suite>` 过程中引发的任何继承自 `AssertionError` 类的实例都将由随后的 `<except suite>` 处理。在 `<except suite>` 内部，标识符 `<name>` 绑定到被引发的异常对象，但此绑定不会在 `<except suite>` 之外存在。

比如:当出现某种异常的时候,我们会有对应的处理方法.

```py
>>> try:
		x = 1 / 0:
	except ZeroDivisionError as e:
		print('handling a', type(e))
		x = 0
handling a <class 'ZeroDivisionError'>
>>> x
0
```

异常对象.

自定义继承于Exception即可,可以具有属性.

> ​	简单来讲,异常处理使得程序更有健壮性.

### 1.3.4组合语言的解释器

解释器就是一种函数.

#### 1.3.4.1基于Scheme语法的计算器

基本运算:

```scheme
scm> (*)
1
scm> (+ 1 2 3 4)
10
scm> (* 1 1 4 5 1 4)
80
```

​	减法（）具有两种行为。只传入一个参数时，它会对该值取相反数。传入至少两个参数时，它会用第一个参数减去之后的所有参数。除法（）也有类似的两种行为：计算单个参数的倒数，或用第一个参数除以之后的所有参数.

先对于子表达式求值,再得到最后的结果:

```scheme
scm> (- 100 (* 7 (+ 8 (/ -12 -3))))
16.0
```

#### 1.3.4.2表达式树

​	Scheme对,列表一定是嵌套对.

​	实现的时候,所有的计算器语言表达式都是py的Pair列表,我们读入嵌套的列表并且转换为pair实例的表达式树.

```py
>>> expr = Pair('+', Pair(Pair('*', Pair(3, Pair(4, nil))), Pair(5, nil)))
>>> print(expr)
(+ (* 3 4) 5)
>>> print(expr.second.first)
(* 3 4)
>>> expr.second.first.second.first
3
```

#### 1.3.4.3解析表达式



原始文本--->表达式树 的过程.

解析器:

1.词法分析器 lexical analyzer

​	输入字符串--->划分成token

2.语法分析器 syntactic analyzer

​	根据生成的token生成表达式树 token序列会被语法分析器消耗

##### 1.3.4.3.1词法分析

tokenizer

对于格式良好的表达式进行分词的操作:

```py
>>> tokenize_line('(+ 1 (* 2.3 45))')
['(', '+', 1, '(', '*', 2.3, 45, ')', ')']
```

可以单独作用于每一行

##### 1.3.4.3.2语法分析

树递归.

```scheme
>>> lines = ['(+ 1', '   (* 2.3 45))']
>>> expression = scheme_read(Buffer(tokenize_lines(lines)))
>>> expression
Pair('+', Pair(1, Pair(Pair('*', Pair(2.3, Pair(45, nil))), nil)))
>>> print(expression)
(+ 1 (* 2.3 45))
```

​	信息丰富的语法错误的反馈,可以极大地提高解释器的可用性(也就是异常)。由此引发的 `SyntaxError` 异常包含对所遇问题的描述。

> ​	总之就是读取token,然后生成Pair.

#### 1.3.4.4计算器语言求值

求值器,表达式作为一个参数并且返回一个值.

### 1.3.5抽象语言的解释器

> ​	我们的大部分lab都将在这一章结束.
>
> ​	HW 07、HW 08、HW 09、Lab 10&Lab11、Scheme、Scheme Challenge、Scheme Contest.
>
> ​	加油!

怎么定义新的运算符以及为我们的值进行命名.

#### 1.3.5.1结构

1.正确解析点列表和引号

```py
>>> read_line("(car '(1 . 2))")
Pair('car', Pair(Pair('quote', Pair(Pair(1, 2), nil)), nil))
```

2.求值---一种简化的形式

```py
>>> def scheme_eval(expr, env):
        """Evaluate Scheme expression expr in environment env."""
        if scheme_symbolp(expr):
            return env[expr]
        elif scheme_atomp(expr):
            return expr
        first, rest = expr.first, expr.second
        if first == "lambda":
            return do_lambda_form(rest, env)
        elif first == "define":
            do_define_form(rest, env)
            return None
        else:
            procedure = scheme_eval(first, env)
            args = rest.map(lambda operand: scheme_eval(operand, env))
            return scheme_apply(procedure, args, env)
```

基本流程还是lab10中我们写过的.

#### 1.3.5.2环境ENV

​	类似于py,现在我们已经描述了 Scheme 解释器的结构，接下来我们来实现构成环境的 `Frame` 类。每个 `Frame` 实例代表一个环境，在这个环境中，符号与值绑定。一个帧有一个保存绑定（`bindings`）的字典，以及一个父（`parent`）帧。对于全局帧而言，父帧为 `None`。这就是环境的实现方法.

​	绑定不能直接访问，而是通过两种 `Frame` 方法：`lookup` 和 `define`。第一个方法实现了第一章中描述的计算环境模型的查找流程。符号与当前帧的绑定相匹配。如果找到它，则返回它绑定到的值。如果没有找到，则继续在父帧中查找。另一方面，`define` 方法用来将符号绑定到当前帧中的值。

lookup逐渐向上进行查找,define用来和frame进行绑定.

```scheme
(define (factorial n)
  (if (= n 0) 1 (* n (factorial (- n 1)))))

(factorial 5)
120
```

do_define_form来进行求值的操作.

来分析上面的两个输入.

第一个输入表达式是一个 `define` 形式，将由 Python 函数 `do_define_form` 求值。定义一个函数有如下步骤：

1. 检查表达式的格式，确保它是一个格式良好的 Scheme 列表，在关键字 `define` 后面至少有两个元素。
2. 分析第一个元素（这里是一个 `Pair`），找出函数名称 `factorial` 和形式参数表 `(n)`。
3. 使用提供的形式参数、函数主体和父环境创建 `LambdaProcedure`。
4. 在当前环境的第一帧中，将 `factorial` 符号与此函数绑定。在示例中，环境只包括全局帧。

第二个输入是调用表达式。传递给 `scheme_apply` 的 `procedure` 是刚刚创建并绑定到符号 `factorial` 的 `LambdaProcedure`。传入的 `args` 是一个单元素 Scheme 列表 `(5)`。为了应用该函数，我们将创建一个新帧来扩展全局帧（`factorial` 函数的父环境）。在这帧中，符号 `n` 被绑定为数值 5。然后，我们将在该环境中对 `factorial` 函数主体进行求值，并返回其值。

> ​	这里有点复杂,还是具体做的时候再看.

#### 1.3.5.3数据即程序

程序是对于抽象机器的描述.

用户的程序就是解释器的数据.

在执行过程中对于构建的表达式进行求值.

#### schemeLab简单记录

##### 熟悉scheme

> ​	逆天超级小括号语言.

scheme其实是一种诞生于这门课程的著名的教学语言.

会写简单的scheme代码,主要是理解前缀表达式.

比如实现幂函数:

```scheme
(define (pow base exp) 
  (if (or (= base 1) (= exp 0)) 
    1
    (* base (pow base (- exp 1))))
)
```

let语句进行局部变量的绑定:

```scheme
(define (repeatedly-cube n x)
  (if (zero? n)
      x
      (let ((y (* x x x)))
        (repeatedly-cube (- n 1) y))))
```

**car**:获取list的第一个元素.

**cdr**:获取list中除第一个元素的剩余部分.

```scheme
(define (cadr s) 
  (car (cdr s))
)

(define (caddr s) 
  (car (cdr (cdr s)))
)
```

那么这样的简单实现就是返回list中的第几个元素.

怎么使用条件?

```scheme
(define (ascending? s) 
    (or (null? s)
        (null? (cdr s))
        (and (<= (car s) (car (cdr s))) (ascending? (cdr s)))
    )
)
```



返回满足条件的list中的元素,list的本质还是pair对:

cond就是类似于switch语句,最后的else就是default.

cons就是给定两个参数,构造一个pair的数据对象.

```scheme
(define (my-filter pred s) 
    (cond ((null? s) '())
            ((pred (car s)) (cons (car s) (my-filter pred (cdr s))))
            (else (my-filter pred (cdr s)))
    )
)
```

交替取两个数组的元素:

```scheme
(define (interleave lst1 lst2) 
   (cond    ((null? lst1)  lst2)
            ((null? lst2)  lst1)
            (else (cons (car lst1)
                        (cons (car lst2) (interleave (cdr lst1) (cdr lst2)))))))
```

怎么用lambda函数:

```scheme
(define (no-repeats s) 
    (if (null? s)
        '()
        (cons (car s) 
            (no-repeats 
                (filter (lambda (x) (not (= x (car s)))) 
                        (cdr s))))))
```

`做代码构造,有点模板的味道,还有  ,  表示先不要引用,先进行动态的求值.

代码生成,动态构造lambda表达式.

```scheme
(define (curry-cook formals body) 
  (if (null? (cdr formals))
    `(lambda (,(car formals)) ,body)
    `(lambda (,(car formals)) ,(curry-cook (cdr formals) body))))
```

curry展开的操作:

```scheme
(define (curry-consume curry args)
  (if (null? args)
      curry
      (let ((result (curry (car args))))
      (curry-consume result (cdr args)))))
```

这几行代码真是抽象过头了:

把switch语句转换成cond语句,`做代码生成,map把lambda应用于list上面去.

```scheme
(define (switch-to-cond switch-expr)
  (cons `cond
        (map (lambda (option)
               (cons (list `equal? (car (cdr switch-expr))(car option))  (cdr option)))
             (car (cdr (cdr switch-expr))) ) ))
```

比如实现欧几里得算法:

```scheme
(define (gcd a b)
  (if (= b 0)
      a
      (gcd b (modulo a b))))
```

插入:

```scheme
(define (pow-expr base exp) 
  (cond ((= exp 0) 1)
        ((= exp 1) `(* ,base 1))
        ((even? exp) `(square ,(pow-expr base (/ exp 2))))
        (else `(* ,base ,(pow-expr base (- exp 1))))))
```

宏函数和begin的用法:

宏定义.begin--->按照顺序执行一系列的子句(...)(...)

```scheme
(define-macro (repeat n expr)
  `(repeated-call ,n (lambda() ,expr)))

; Call zero-argument procedure f n times and return the final result.
(define (repeated-call n f)
  (if (= n 1)
      (f)
      (begin (f) (repeated-call (- n 1) f))))
```

##### 解释器的实现

我们要更改四个文件:

- `scheme_eval_apply.py`
- `scheme_forms.py`
- `scheme_classes.py`
- `questions.scm`

###### PARTI:Evaluator

求值器的作用.

符号定义和查找.

表达式求值.

调用已经实现的方法.



环境,frame的问题,比如lookup的实现:

我们遍历字典的键值对.

```py
# 从我当前的这个frame开始往parent的frame进行查找,找到并且返回这个value
    def lookup(self, symbol):
        """Return the value bound to SYMBOL. Errors if SYMBOL is not found."""
        # BEGIN PROBLEM 1
        curr = self
        while curr != None:
            for key, value in curr.bindings.items():
                if key == symbol:
                    return curr.bindings[key]
            curr = curr.parent
        # END PROBLEM 1
        raise SchemeError('unknown identifier: {0}'.format(symbol))
```

###### PARTII:Procedures

lambda函数,自定义函数,动态作用域

理解我们自己创建的函数是怎么被调用的,以及frame的作用原理和规则



###### PARTIII:Special Forms

这是关于如何实现let函数,主要是还是理解expr的Pair形式是怎样的.

```py
def make_let_frame(bindings, env):
    """Create a child frame of Frame ENV that contains the definitions given in
    BINDINGS. The Scheme list BINDINGS must have the form of a proper bindings
    list in a let expression: each item must be a list containing a symbol
    and a Scheme expression."""
    if not scheme_listp(bindings):
        raise SchemeError('bad bindings list in let form')
    names = vals = nil
    # BEGIN PROBLEM 14
    # 传进来的bindings应该是一个pair对象
    # 我还要倒着构造两个pair串
    curr = bindings
    while curr != nil:
        validate_form(curr.first, 2, 2)
        binding = curr.first
        names = Pair(binding.first, names)
        vals = Pair(binding.rest.first, vals)
        curr = curr.rest
    validate_formals(names)

    # 然后还要对于vals表达式进行求值
    vals = nil
    curr = bindings
    while curr != nil:
        val = scheme_eval(curr.first.rest.first, env)
        vals = Pair(val, vals)
        curr = curr.rest

    # END PROBLEM 14
    return env.make_child_frame(names, vals)
```

> ​	整体实现难度不高,都在我的仓库里面.

## 1.4数据处理

> 程序被组织成对于顺序数据流操作的管道.
>
> 有效的处理和操作连续的数据流.
>
> 无限序列.

### 1.4.1隐式序列

不是把所有元素显式的存放在内存中.而是在访问某个元素的时候,才去计算它的值.

```scheme
>>> r = range(10000,1000000000)
>>> r[45006230]
45016230
```



显然range肯定不会存放这么多数字,而是调用的时候直接加上一个值得到value.

那么这就是惰性计算(**Lazy Computation**),这是本篇的重点.

#### 1.4.1.1iterator

```py
>>> primes = [2, 3, 5, 7]
>>> type(primes)
<class 'list'>
>>> iterator = iter(primes)
>>> type(iterator)
<class 'list-iterator'>
>>> next(iterator)
2
>>> next(iterator)
3
>>> next(iterator)
5
```

获取迭代器,和基本的遍历操作.

没有更多,引发StopIteration异常:

```py
>>> next(iterator)
7
>>> next(iterator)
Traceback (most recent call las):
  File "<stdin>", line 1, in <module>
StopIteration

>>> try:
        next(iterator)
    except StopIteration:
        print('No more values')
No more values
```

两个迭代器是独立的:

```py
>>> r = range(3, 13)
>>> s = iter(r)  # r 的第一个迭代器
>>> next(s)
3
>>> next(s)
4
>>> t = iter(r)  # r 的第二个迭代器
>>> next(t)
3
>>> next(t)
4
>>> u = t        # u 绑定到 r 的第二个迭代器
>>> next(u)
5
>>> next(u)
6
```

#### 1.4.1.2可迭代性



可迭代对象:序列 + 容器

```py
>>> d = {'one': 1, 'two': 2, 'three': 3}
>>> d
{'one': 1, 'three': 3, 'two': 2}
>>> k = iter(d)
>>> next(k)
'one'
>>> next(k)
'three'
>>> v = iter(d.values())
>>> next(v)
1
>>> next(v)
3
```

py3.6+--->字典是有序的,按照插入的顺序

#### 1.4.1.3内置迭代器

```py
>>> def double_and_print(x):
        print('***', x, '=>', 2*x, '***')
        return 2*x
>>> s = range(3, 7)
>>> doubled = map(double_and_print, s)  # double_and_print 未被调用
>>> next(doubled)                       # double_and_print 调用一次
*** 3 => 6 ***
6
>>> next(doubled)                       # double_and_print 再次调用
*** 4 => 8 ***
8
>>> list(doubled)                       # double_and_print 再次调用兩次
*** 5 => 10 ***                         # list() 会把剩余的值都计算出来并生成一个列表
*** 6 => 12 ***
[10, 12]
```

例如map函数,只有iterator被next调用的时候,才会产生计算.

#### 1.4.1.4For语句

for是对于迭代对象进行操作.

```py
>>> items = counts.__iter__()
>>> try:
        while True:
             item = items.__next__()
             print(item)
    except StopIteration:
        pass
1
2
3
```

#### 1.4.1.5生成器和yield语句

比如:

```py
>>> def letters_generator():
        current = 'a'
        while current <= 'd':
            yield current
            current = chr(ord(current) + 1)

>>> for letter in letters_generator():
        print(letter)
a
b
c
d
```

每次调用next的时候,每次执行到一个yield语句就直接返回.

我们可以手动调用__next__来遍历这个生成器:

```py
>>> letters = letters_generator()
>>> type(letters)
<class 'generator'>
>>> letters.__next__()
'a'
>>> letters.__next__()
'b'
>>> letters.__next__()
'c'
>>> letters.__next__()
'd'
>>> letters.__next__()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```

> ​	结语,第四章的内容整体比较抽象,都是一些概览性的intro的内容,比如数据库,并行计算,多线程,开发,网络等,是很好的科普内容.
