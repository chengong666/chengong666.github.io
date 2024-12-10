---
title: lua基础篇
date: 2024-12-10 22:04:04
tags:
---
# LUA学习笔记

## 基本语法

### 提示符

以字母获取下划线开头，但是最好不要使用下划线加大写字母，因为Lua的保留字也是这样的。

Lua 不允许使用特殊字符如 **@**, **$**, 和 **%** 来定义标示符。 Lua 是一个区分大小写的编程语言。因此在 Lua 中 Runoob 与 runoob 是两个不同的标示符。

### 关键字

![image-20240326225000507](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240326225000507.png)

### 全局变量

lua中默认就是全局变量，没有初始化的变量自动赋值为nil,如果想要删除一个全局变量，只需要将变量赋值为nil

这样变量b就好像从没被使用过一样。换句话说, 当且仅当一个变量不等于nil时，这个变量即存在

## lua数据类型

![image-20240326230625618](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240326230625618.png)

> nil可以用于删除数据，将nil赋值给对应元素即可，和null类似

对于全局变量和 table，nil 还有一个"删除"作用，给全局变量或者 table 表里的变量赋一个 nil 值，等同于把它们删掉，执行下面代码就知：

```lua
tab1 = { key1 = "val1", key2 = "val2", "val3" }
for k, v in pairs(tab1) do
    print(k .. " - " .. v)
end
 
tab1.key1 = nil
for k, v in pairs(tab1) do
    print(k .. " - " .. v)
end
```

nil 作比较时应该加上双引号 **"**：

```lua
> type(X)
nil
> type(X)==nil
false
> type(X)=="nil"
true
>
```

**boolean：**在LUA中0表示true，boolean 类型只有两个可选值：true（真） 和 false（假），Lua 把 false 和 nil 看作是 false，其他的都为 true，数字 0 也是 true

**number**：在LUA中只有一种数据类型，也就是doubole

**string：**string的表示方式有三种，可以使用单引号，双引号，还有一种是在其他大的语言中很少使用的，通过方括号

```lua
html = [[
<html>
<head></head>
<body>
    <a href="http://www.runoob.com/">菜鸟教程</a>
</body>
</html>
]]
print(html)
```

string类型在与数字进行运算的时候会自动进行类型的转换。

字符串的连接通过..进行连接

**列如：**

```lua
print("a" .. 'b')
ab
print(157 .. 428)
157428
```

**table**

在 Lua 里，table 的创建是通过"构造表达式"来完成，最简单构造表达式是{}，用来创建一个空表。也可以在表里添加一些数据，直接初始化表,table在lua的应用很广泛，可以用作数组，较为重要:

```lua
-- 创建一个空的 table
local tbl1 = {}
 
-- 直接初始表
local tbl2 = {"apple", "pear", "orange", "grape"}
```

lua中的table是一个关联数组，有点想其他语法中的字典，他的索引可以使数字也可以还是字符串。

```lua
-- table_test.lua 脚本文件
a = {}
a["key"] = "value"
key = 10
a[key] = 22
a[key] = a[key] + 11
for k, v in pairs(a) do
    print(k .. " : " .. v)
end
```

> lua中的索引是从1开始的，和其他的于洋不一样，其他的语言索引从0开始，但是在lua中也可以输出索引0的数字，但是得到的结果为nil，他不会越界，但是值为nil

```lua
-- table_test2.lua 脚本文件
local tbl = {"apple", "pear", "orange", "grape"}
for key, val in pairs(tbl) do
    print("Key", key)
end
```

```lua
$ lua table_test2.lua 
Key    1
Key    2
Key    3
Key    4
```

lua中的table的长度不固定，能够实现动态扩展

```lua
-- table_test3.lua 脚本文件
a3 = {}
for i = 1, 10 do
    a3[i] = i
end
a3["key"] = "val"
print(a3["key"])
print(a3["none"])
$ lua table_test3.lua 
val
nil

```

**function**

基本使用

```lua
-- function_test.lua 脚本文件
function factorial1(n)
    if n == 0 then
        return 1
    else
        return n * factorial1(n - 1)
    end
end
print(factorial1(5))
factorial2 = factorial1
print(factorial2(5))
```

函数可以作为参数进行传递，可以摘参数列表中传入匿名函数

```lua
-- function_test2.lua 脚本文件
function testFun(tab,fun)
        for k ,v in pairs(tab) do
                print(fun(k,v));
        end
end


tab={key1="val1",key2="val2"};
testFun(tab,
function(key,val)--匿名函数
        return key.."="..val;
end
);
```

**thread（线程）**

在 Lua 里，最主要的线程是协同程序（coroutine）。它跟线程（thread）差不多，拥有自己独立的栈、局部变量和指令指针，可以跟其他协同程序共享全局变量和其他大部分东西。

**线程跟协程的区别：**线程可以同时多个运行，而协程任意时刻只能运行一个，并且处于运行状态的协程只有被挂起（suspend）时才会暂停。

**对于为什么一个时刻只能运行一个协程的解释：**

在Lua中，协程是一种轻量级的线程，它们可以在同一个线程中并发运行。但是，每个时刻只能有一个协程处于运行状态，因为Lua的协程是基于协作式多任务处理的。

与操作系统中的线程不同，Lua中的协程不是由操作系统内核调度的，而是由Lua虚拟机自己的调度器来调度的。在Lua中，协程可以通过yield函数主动让出执行权，让其他协程运行。因此，每个时刻只有一个协程在运行，其他协程处于挂起状态，等待被唤醒。

当一个协程执行到yield函数时，它会将自己的状态保存下来，并让出执行权。调度器会选择另一个处于挂起状态的协程来运行，恢复其状态并继续执行。这个过程可以一直循环下去，直到所有协程都执行完毕。

因此，尽管Lua中的协程可以在同一个线程中并发运行，但每个时刻只能有一个协程处于运行状态，其他协程处于挂起状态。这是因为Lua的协程是**基于协作式多任务处理的**，**需要协程自己主动让出执行权才能切换到其他协程运行。**

**基于协作式多任务处理**：

基于协作式多任务处理是一种多任务处理的方式，其中任务之间通过协作来共享处理器时间。在基于协作式的多任务处理中，每个任务（例如线程、协程或进程）在运行时可以自主决定何时让出处理器时间，而不是由系统强制性地进行调度。

在基于协作式的多任务处理中，任务之间通常通过显式地让出执行权来协调，这通常是通过调用特定的函数（如yield）来实现的。当一个任务决定让出处理器时间时，它会通知调度器或其他任务，以便其他任务有机会执行。这种方式要求任务之间相互信任，以确保它们按照协议合作，否则可能会导致死锁或其他问题。

相对于基于抢占式的多任务处理，基于协作式的多任务处理通常更简单，因为任务不需要考虑被系统强制性地中断的情况。然而，基于协作式的多任务处理也可能存在一些问题，如某个任务长时间不让出处理器时间可能会导致其他任务无法执行，从而降低系统的响应性。

总的来说，基于协作式多任务处理是一种多任务处理的方式，其中任务通过协作来共享处理器时间，每个任务自主决定何时让出执行权，以实现多任务之间的协同工作。

**Lua中的协程和Unity中的协程都是基于协作式多任务处理的，它们都可以让开发者方便地实现异步编程和协程调度。**

相同之处：

- 都是**基于协作式多任务处理的**，任务之间通过**协作来共享处理器时间**。
- 都可以通过**yield函数让出执行权**，让其他协程或任务运行。
- 都可以方便地实现异步编程和协程调度，使得代码更加简洁易懂。

不同之处：

- Lua中的协程是**轻量级的线程**，可以在同一个线程中并发运行，而Unity中的协程是基于MonoBehaviour组件的，**只能在Unity主线程中运行。**

- Lua中的协程可以通过coroutine库中的函数来创建和管理，而Unity中的协程可以通过StartCoroutine和StopCoroutine等方法来创建和管理。

- 在Lua中，协程的调度由Lua虚拟机自己的调度器来管理，而在Unity中，协程的调度由Unity引擎的**协程调度器**来管理。

- 在Lua中，协程可以通过**resume**函数恢复执行，而在Unity中，协程由于是基于MonoBehaviour组件的，只能通过yield return语句来让出执行权和恢复执行。

  总的来说，Lua中的协程和Unity中的协程都是基于协作式多任务处理的，它们都可以让开发者方便地实现异步编程和协程调度。它们的主要区别在于实现方式和使用方式上的差异。 

  **至于为什么已经有了线程还要去实现协程的原因所给出的解释**

  虽然线程和协程都可以用于并发处理，但它们之间存在一些重要的区别，这些区别导致了在某些情况下使用协程会更加合适。

  1. **轻量级：**协程通常比线程更轻量级。线程的创建和管理可能需要更多的系统资源，而协程通常由程序员在应用层面进行管理，因此可以更加灵活地控制和调度。

  2. **调度控制：**在协程中，调度是由程序员显式控制的，而线程的调度通常由操作系统或者运行时环境来管理。这意味着在协程中，程序员可以更加精细地控制任务的执行顺序和时机。

  3. **适用场景：**协程通常更适合于处理大量的、需要频繁切换的任务，例如游戏中的动画更新、用户输入响应等。而线程更适合于处理长时间运行的、需要独立执行的任务，例如网络请求、文件操作等。

  在Unity和Lua中，协程的实现更适合于处理游戏中常见的异步操作、动画更新等任务。通过协程，开发者可以更加灵活地控制任务的执行顺序和时机，从而提高代码的可读性和可维护性。

  因此，尽管线程和协程在某些功能上有重叠，但它们在实际应用中有着不同的使用场景和优势，开发者可以根据具体的需求选择合适的并发处理方式。

------

**userdata:**在lua中表示自定义数据，通常用于存储c或者c++中的结构体数据类型

## lua变量

变量在使用前，需要在代码中进行声明，即创建该变量。

编译程序执行代码之前编译器需要知道如何给语句变量开辟存储区，用于存储变量的值。

Lua 变量有三种类型：全局变量、局部变量、表中的域。

Lua 中的变量全是全局变量，哪怕是语句块或是函数里，除非用 local 显式声明为局部变量。

局部变量的作用域为从声明位置开始到所在语句块结束。

变量的默认值均为 nil。

**在能够使用局部变量的时候要尽量使用局部变量，以为局部变量的访问速度快于全局变量，并且过多的使用全局变量，会导致全局变量污染问题。**

**赋值语句**

```lua
a = "hello" .. "world"
t.n = t.n + 1
a, b = 10, 2*x       <-->       a=10; b=2*x
x, y = y, x                     -- swap 'x' for 'y'
a[i], a[j] = a[j], a[i]         -- swap 'a[i]' for 'a[j]'

```

注意事项：赋值一定要一一对应，否则没有获取的值，将会自动赋值为nil

```lua
a, b, c = 0, 1
print(a,b,c)             --> 0   1   nil
 
a, b = a+1, b+1, b+2     -- value of b+2 is ignored
print(a,b)               --> 1   2
 
a, b, c = 0
print(a,b,c)             --> 0   nil   nil
```

变量可以用于接收函数的返回值，lua中的函数可以设定多个返回值

```lua
a, b = f()
```

**索引**

```lua
> site = {}
> site["key"] = "www.runoob.com"
> print(site["key"])
www.runoob.com
> print(site.key)
www.runoob.com
```

## lua循环

**循环类型**

| while循环     | 在条件为 true 时，让程序重复地执行某些语句。执行语句前会先检查条件是否为 true。 |
| ------------- | ------------------------------------------------------------ |
| for循环       | 重复执行指定语句，重复次数可在 for 语句中控制。              |
| repeat..until | 重复执行循环，直到 指定的条件为真时为止                      |
| 循环嵌套      | 可以在循环内嵌套一个或多个循环语句（while do ... end;for ... do ... end;repeat ... until;） |

**while循环：**

```lua
while(condition)
do
   statements
end
基本语法
```

**for循环**

类型：

| 数值for循环 |
| ----------- |
| 泛型for循环 |

数值for循环

```lua
for i=1,f(x) do
    print(i)
end
 
for i=10,1,-1 do
    print(i)
end
```

泛型for循环

```lua
--打印数组a的所有值  
a = {"one", "two", "three"}
for i, v in ipairs(a) do 
    print(i, v)
end
```

**思考：**

**ipairs的原理是迭代器，迭代器的本质是指针**

答：不完全正确。在Lua中，迭代器是一种用于遍历集合中元素的机制，它并不一定是指针。迭代器是一种抽象的概念，它可以用于访问集合中的元素，而不必暴露集合的内部结构。

在Lua中，ipairs函数是用于迭代数组元素的迭代器函数。它返回一个迭代器函数和数组本身，迭代器函数在每次调用时返回下一个索引和对应的值，直到遍历完整个数组。

迭代器的本质是一个可以记住遍历位置的函数。它可以是一个闭包，也可以是一个带有状态的对象。在每次调用迭代器函数时，它都会返回下一个元素，直到遍历完集合。

因此，迭代器并不一定是指针，它可以是任何一种能够记录遍历状态并返回下一个元素的机制。

**在c++中迭代器是指针实现的吗**

答：在C++中，迭代器不一定是指针实现的，尽管在许多情况下，迭代器确实被实现为指针。C++标准库中的迭代器是一种抽象的概念，它提供了一种统一的访问容器元素的方式，而不必暴露容器的内部结构。

C++的迭代器被设计为类对象，它们可以被用于遍历容器中的元素。迭代器提供了一组操作符重载，使得它们的行为类似于指针，例如解引用操作符 `*` 和成员访问操作符 `->`。这使得在使用迭代器时，可以像使用指针一样进行操作。

C++标准库中的迭代器分为几种类型，包括输入迭代器、输出迭代器、前向迭代器、双向迭代器和随机访问迭代器。这些迭代器类型提供了不同的功能和性能保证，以满足不同的需求。

虽然在许多情况下，迭代器被实现为指针，但这并不是强制的。实际上，C++标准库中的迭代器可以是各种不同的实现，只要它们满足迭代器的要求，即提供适当的操作符重载和行为。因此，虽然迭代器通常表现得像指针，但它们并不一定是指针实现的。

**那么什么是迭代器？**

答：**（核心是访问容器元素，而无法得知其内部结构，对于实现方式并没有限制）**

迭代器是一种设计模式或者抽象概念，用于提供一种统一的方式来访问容器（如数组、列表、集合等）中的元素，而不必暴露容器的内部结构。迭代器允许客户端代码按顺序访问容器中的元素，而不需要了解容器的具体实现方式。

在编程语言和标准库中，迭代器通常是一个对象或者函数，它提供了一组操作符重载或方法，允许对容器中的元素进行遍历和访问。通过迭代器，可以实现对容器的遍历、查找、过滤、映射等操作，从而提高代码的灵活性和可复用性。

迭代器通常提供以下基本功能：

1. 访问容器中的元素：迭代器允许按顺序访问容器中的元素，包括读取和修改元素的值。

2. 移动：迭代器可以向前移动到下一个元素，也可以向后移动到上一个元素，以实现遍历容器的功能。

3. 定位：迭代器可以定位到容器中的特定位置，允许直接访问该位置的元素。

4. 结束检测：迭代器通常提供方法来检测是否已经到达容器的末尾，以便终止遍历。

迭代器在许多编程语言和标准库中都被广泛应用，例如C++ STL、Java的迭代器接口、Python的迭代器和生成器等。通过使用迭代器，可以将容器的遍历和操作与具体的数据结构分离，提高代码的可读性、可维护性和灵活性。

**repeat..until循环**（类似于do..while循环）

无论是否满足条件都必须执行一次代码

```lua
--[ 变量定义 --]
a = 10
--[ 执行循环 --]
repeat
   print("a的值为:", a)
   a = a + 1
until( a > 15 )
```

**循环嵌套：**

既可以是相同类型的循环进行嵌套，也可以是不同类型的函数进行嵌套

```lua
j =2
for i=2,10 do
   for j=2,(i/j) , 2 do
      if(not(i%j)) 
      then
         break 
      end
      if(j > (i/j))then
         print("i 的值为：",i)
      end
   end
end
```

break语句

**和其他语言中的一样**

```lua
--[ 定义变量 --]
a = 10

--[ while 循环 --]
while( a < 20 )
do
   print("a 的值为:", a)
   a=a+1
   if( a > 15)
   then
      --[ 使用 break 语句终止循环 --]
      break
   end
end

```

**goto语句**

相当于跳转到对应标签的代码

```lua
local a = 1
::label:: print("--- goto label ---")

a = a+1
if a < 3 then
    goto label   -- a 小于 3 的时候跳转到标签 label
end
```

通过goto标签可以实现continue

```lua
for i=1, 3 do
    if i <= 2 then
        print(i, "yes continue")
        goto continue
    end
    print(i, " no continue")
    ::continue::
    print([[i'm end]])
end
```

## LUA流程控制

控制结构语句类型

![image-20240402160332064](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20240402160332064.png)

**if**

```lua
--[ 定义变量 --]
a = 10;

--[ 使用 if 语句 --]
if( a < 20 )
then
   --[ if 条件为 true 时打印以下信息 --]
   print("a 小于 20" );
end
print("a 的值为:", a);
```

if...else

```lua
--[ 定义变量 --]
a = 100;
--[ 检查条件 --]
if( a < 20 )
then
   --[ if 条件为 true 时执行该语句块 --]
   print("a 小于 20" )
else
   --[ if 条件为 false 时执行该语句块 --]
   print("a 大于 20" )
end
print("a 的值为 :", a)
```

if嵌套

```lua
--[ 定义变量 --]
a = 100;
b = 200;

--[ 检查条件 --]
if( a == 100 )
then
   --[ if 条件为 true 时执行以下 if 条件判断 --]
   if( b == 200 )
   then
      --[ if 条件为 true 时执行该语句块 --]
      print("a 的值为 100 b 的值为 200" );
   end
end
print("a 的值为 :", a );
print("b 的值为 :", b );
```

## LUA函数

**格式**

```lua
optional_function_scope function function_name( argument1, argument2, argument3..., argumentn)
    function_body
    return result_params_comma_separated
end
```

- **optional_function_scope:** 该参数是可选的指定函数是全局函数还是局部函数，未设置该参数默认为全局函数，如果你需要设置函数为局部函数需要使用关键字 **local**。
- **function_name:** 指定函数名称。
- **argument1, argument2, argument3..., argumentn:** 函数参数，多个参数以逗号隔开，函数也可以不带参数。
- **function_body:** 函数体，函数中需要执行的代码语句块。
- **result_params_comma_separated:** 函数返回值，Lua语言函数可以返回多个值，每个值以逗号隔开。

**案列**

```lua
--[[ 函数返回两个值的最大值 --]]
function max(num1, num2)

   if (num1 > num2) then
      result = num1;
   else
      result = num2;
   end

   return result; 
end
-- 调用函数
print("两值比较最大值为 ",max(10,4))
print("两值比较最大值为 ",max(5,6))
```

**函数也可以作为一种参数传入**

```lua
myprint = function(param)
   print("这是打印函数 -   ##",param,"##")
end

function add(num1,num2,functionPrint)
   result = num1 + num2
   -- 调用传递的函数参数
   functionPrint(result)
end
myprint(10)
-- myprint 函数作为参数传递
add(2,5,myprint)
```

同时函数可以设置多个返回值。

**可变参数的使用**

```lua
function add(...)  
local s = 0  
  for i, v in ipairs{...} do   --> {...} 表示一个由所有变长参数构成的数组  
    s = s + v  
  end  
  return s  
end  
print(add(3,4,5,6,7))  --->25
```

可以用一个变量接收可变参数

```lua
function average(...)
   result = 0
   local arg={...}    --> arg 为一个表，局部变量
   for i,v in ipairs(arg) do
      result = result + v
   end
   print("总共传入 " .. #arg .. " 个数")
   return result/#arg
end

print("平均值为",average(10,5,3,4,5,6))
```

如果我们需要使用固定参数，那么固定参数就必须放在可变参数的前面，否则无效

```lua
function fwrite(fmt, ...)  ---> 固定的参数fmt
    return io.write(string.format(fmt, ...))     
end

fwrite("runoob\n")       --->fmt = "runoob", 没有变长参数。  
fwrite("%d%d\n", 1, 2)   --->fmt = "%d%d", 变长参数为 1 和 2
```

1. **获取可变参数的长度**

2. 通常在遍历变长参数的时候只需要使用 **{…}**，然而变长参数可能会包含一些 **nil**，那么就可以用 **select** 函数来访问变长参数了：**select('#', …)** 或者 **select(n, …)**

3. **select('#', …)** 返回可变参数的长度。

4. **select(n, …)** 用于返回从起点 **n** 开始到结束位置的所有参数列表。

5. 调用 select 时，必须传入一个固定实参 selector(选择开关) 和一系列变长参数。如果 selector 为数字 n，那么 select 返回参数列表中从索引 **n** 开始到结束位置的所有参数列表，否则只能为字符串 **#**，这样 select 返回变长参数的总数。

```lua
function f(...)
    a = select(3,...)  -->从第三个位置开始，变量 a 对应右边变量列表的第一个参数
    print (a)
    print (select(3,...)) -->打印所有列表参数
end

f(0,1,2,3,4,5)
```

## LUA运算符

运算符是一个特殊的符号，用于告诉解释器执行特定的数学或逻辑运算。Lua提供了以下几种运算符类型：

- 算术运算符
- 关系运算符
- 逻辑运算符
- 其他运算符

**算术运算符**

| 操作符 | 描述                 | 实例                |
| :----- | :------------------- | :------------------ |
| +      | 加法                 | A + B 输出结果 30   |
| -      | 减法                 | A - B 输出结果 -10  |
| *      | 乘法                 | A * B 输出结果 200  |
| /      | 除法                 | B / A 输出结果 2    |
| %      | 取余                 | B % A 输出结果 0    |
| ^      | 乘幂                 | A^2 输出结果 100    |
| -      | 负号                 | -A 输出结果 -10     |
| //     | 整除运算符(>=lua5.3) | **5//2** 输出结果 2 |

**关系运算符**

下表列出了 Lua 语言中的常用关系运算符，设定 A 的值为10，B 的值为 20：

| 操作符 | 描述                                                         | 实例                  |
| :----- | :----------------------------------------------------------- | :-------------------- |
| ==     | 等于，检测两个值是否相等，相等返回 true，否则返回 false      | (A == B) 为 false。   |
| ~=     | 不等于，检测两个值是否相等，不相等返回 true，否则返回 false  | (A ~= B) 为 true。    |
| >      | 大于，如果左边的值大于右边的值，返回 true，否则返回 false    | (A > B) 为 false。    |
| <      | 小于，如果左边的值大于右边的值，返回 false，否则返回 true    | (A < B) 为 true。     |
| >=     | 大于等于，如果左边的值大于等于右边的值，返回 true，否则返回 false | (A >= B) 返回 false。 |
| <=     | 小于等于， 如果左边的值小于等于右边的值，返回 true，否则返回 false | (A <= B) 返回 true。  |

**逻辑运算符**

下表列出了 Lua 语言中的常用逻辑运算符，设定 A 的值为 true，B 的值为 false：

| 操作符 | 描述                                                         | 实例                   |
| :----- | :----------------------------------------------------------- | :--------------------- |
| and    | 逻辑与操作符。 若 A 为 false，则返回 A，否则返回 B。         | (A and B) 为 false。   |
| or     | 逻辑或操作符。 若 A 为 true，则返回 A，否则返回 B。          | (A or B) 为 true。     |
| not    | 逻辑非操作符。与逻辑运算结果相反，如果条件为 true，逻辑非为 false。 | not(A and B) 为 true。 |

**其他运算符**

下表列出了 Lua 语言中的连接运算符与计算表或字符串长度的运算符：

| 操作符 | 描述                               | 实例                                                         |
| :----- | :--------------------------------- | :----------------------------------------------------------- |
| ..     | 连接两个字符串                     | a..b ，其中 a 为 "Hello " ， b 为 "World", 输出结果为 "Hello World"。 |
| #      | 一元运算符，返回字符串或表的长度。 | #"Hello" 返回 5                                              |

运算符优先级

```
^
not    - (unary)
*      /       %
+      -
..
<      >      <=     >=     ~=     ==
and
or

```

**除了 ^ 和 .. 外所有的二元运算符都是左连接的。**

```
a+i < b/2+1          <-->       (a+i) < ((b/2)+1)
5+x^2*8              <-->       5+((x^2)*8)
a < y and y <= z     <-->       (a < y) and (y <= z)
-x^2                 <-->       -(x^2)
x^y^z                <-->       x^(y^z)

```

## LUA字符串

**字符串长度计算**

在 Lua 中，要计算字符串的长度（即字符串中字符的个数），你可以使用 **string.len**函数或 **utf8.len** 函数，包含中文的一般用 **utf8.len**，**string.len** 函数用于计算只包含 ASCII 字符串的长度。

**实例**



```lua
local myString = "Hello, RUNOOB!"

-- 计算字符串的长度（字符个数）
local length = string.len(myString)

print(length) -- 输出 14

以上实例的 myString 字符串只包含 ASCII 字符，因此 string.len函数可以准确地返回字符串的长度。


```

**包含中文的字符串使用 utf8.len函数：**

**实例**

```lua
**local** myString = "Hello, 世界!"

*-- 计算字符串的长度（字符个数）*
**local** length1 = utf8.len(myString)
print(length1) *-- 输出 9*

*-- string.len 函数会导致结果不准确*
**local** length2 = string.len(myString)
print(length2) *-- 输出 14*
```

**转义字符**

| 转义字符 | 意义                                | ASCII码值（十进制） |
| -------- | ----------------------------------- | ------------------- |
| \a       | 响铃(BEL)                           | 007                 |
| \b       | 退格(BS) ，将当前位置移到前一列     | 008                 |
| \f       | 换页(FF)，将当前位置移到下页开头    | 012                 |
| \n       | 换行(LF) ，将当前位置移到下一行开头 | 010                 |
| \r       | 回车(CR) ，将当前位置移到本行开头   | 013                 |
| \t       | 水平制表(HT) （跳到下一个TAB位置）  | 009                 |
| \v       | 垂直制表(VT)                        | 011                 |
| \\       | 代表一个反斜线字符''\'              | 092                 |
| \'       | 代表一个单引号（撇号）字符          | 039                 |
| \"       | 代表一个双引号字符                  | 034                 |
| \0       | 空字符(NULL)                        | 000                 |
| \ddd     | 1到3位八进制数所代表的任意字符      | 三位八进制          |
| \xhh     | 1到2位十六进制所代表的任意字符      | 二位十六进制        |

**字符串操作**

| 序号 | 方法 & 用途                                                  |
| :--- | :----------------------------------------------------------- |
| 1    | **string.upper(argument):** 字符串全部转为大写字母。         |
| 2    | **string.lower(argument):** 字符串全部转为小写字母。         |
| 3    | **string.gsub(mainString,findString,replaceString,num)**在字符串中替换。mainString 为要操作的字符串， findString 为被替换的字符，replaceString 要替换的字符，num 替换次数（可以忽略，则全部替换），如：`> string.gsub("aaaa","a","z",3); zzza  3` |
| 4    | **string.find (str, substr, [init, [plain]])** 在一个指定的目标字符串 str 中搜索指定的内容 substr，如果找到了一个匹配的子串，就会返回这个子串的起始索引和结束索引，不存在则返回 nil。**init** 指定了搜索的起始位置，默认为 1，可以一个负数，表示从后往前数的字符个数。**plain** 表示是否使用简单模式，默认为 false，true 只做简单的查找子串的操作，false 表示使用使用正则模式匹配。以下实例查找字符串 "Lua" 的起始索引和结束索引位置：`> string.find("Hello Lua user", "Lua", 1)  7  9` |
| 5    | **string.reverse(arg)** 字符串反转`> string.reverse("Lua") auL` |
| 6    | **string.format(...)** 返回一个类似printf的格式化字符串`> string.format("the value is:%d",4) the value is:4` |
| 7    | **string.char(arg) 和 string.byte(arg[,int])** char 将整型数字转成字符并连接， byte 转换字符为整数值(可以指定某个字符，默认第一个字符)。`> string.char(97,98,99,100) abcd > string.byte("ABCD",4) 68 > string.byte("ABCD") 65 >` |
| 8    | **string.len(arg)** 计算字符串长度。`string.len("abc") 3`    |
| 9    | **string.rep(string, n)** 返回字符串string的n个拷贝`> string.rep("abcd",2) abcdabcd` |
| 10   | **..** 链接两个字符串`> print("www.runoob.".."com") www.runoob.com` |
| 11   | **string.gmatch(str, pattern)** 返回一个迭代器函数，每一次调用这个函数，返回一个在字符串 str 找到的下一个符合 pattern 描述的子串。如果参数 pattern 描述的字符串没有找到，迭代函数返回nil。`> for word in string.gmatch("Hello Lua user", "%a+") do print(word) end Hello Lua user` |
| 12   | **string.match(str, pattern, init)** string.match()只寻找源字串str中的第一个配对. 参数init可选, 指定搜寻过程的起点, 默认为1。 在成功配对时, 函数将返回配对表达式中的所有捕获结果; 如果没有设置捕获标记, 则返回整个配对字符串. 当没有成功的配对时, 返回nil。`> = string.match("I have 2 questions for you.", "%d+ %a+") 2 questions > = string.format("%d, %q", string.match("I have 2 questions for you.", "(%d+) (%a+)")) 2, "questions"` |

**字符串截取**

字符串截取使用 sub() 方法。

string.sub() 用于截取字符串，原型为：

```
string.sub(s, i [, j])
```

参数说明：

- s：要截取的字符串。
- i：截取开始位置。
- j：截取结束位置，默认为 -1，最后一个字符。

**案例**

```lua
-- 字符串
local sourcestr = "prefix--runoobgoogletaobao--suffix"
print("\n原始字符串", string.format("%q", sourcestr))

-- 截取部分，第4个到第15个
local first_sub = string.sub(sourcestr, 4, 15)
print("\n第一次截取", string.format("%q", first_sub))

-- 取字符串前缀，第1个到第8个
local second_sub = string.sub(sourcestr, 1, 8)
print("\n第二次截取", string.format("%q", second_sub))

-- 截取最后10个
local third_sub = string.sub(sourcestr, -10)
print("\n第三次截取", string.format("%q", third_sub))

-- 索引越界，输出原始字符串
local fourth_sub = string.sub(sourcestr, -100)
print("\n第四次截取", string.format("%q", fourth_sub))
```

**字符串格式化**

Lua 提供了 **string.format()** 函数来生成具有特定格式的字符串, 函数的第一个参数是格式 , 之后是对应格式中每个代号的各种数据。

由于格式字符串的存在, 使得产生的长字符串可读性大大提高了。这个函数的格式很像 C 语言中的 printf()。

以下实例演示了如何对字符串进行格式化操作：

格式字符串可能包含以下的转义码:

- %c - 接受一个数字, 并将其转化为ASCII码表中对应的字符
- %d, %i - 接受一个数字并将其转化为有符号的整数格式
- %o - 接受一个数字并将其转化为八进制数格式
- %u - 接受一个数字并将其转化为无符号整数格式
- %x - 接受一个数字并将其转化为十六进制数格式, 使用小写字母
- %X - 接受一个数字并将其转化为十六进制数格式, 使用大写字母
- %e - 接受一个数字并将其转化为科学记数法格式, 使用小写字母e
- %E - 接受一个数字并将其转化为科学记数法格式, 使用大写字母E
- %f - 接受一个数字并将其转化为浮点数格式
- %g(%G) - 接受一个数字并将其转化为%e(%E, 对应%G)及%f中较短的一种格式
- %q - 接受一个字符串并将其转化为可安全被Lua编译器读入的格式
- %s - 接受一个字符串并按照给定的参数格式化该字符串

为进一步细化格式, 可以在%号后添加参数. 参数将以如下的顺序读入:

- (1) 符号: 一个+号表示其后的数字转义符将让正数显示正号. 默认情况下只有负数显示符号.
- (2) 占位符: 一个0, 在后面指定了字串宽度时占位用. 不填时的默认占位符是空格.
- (3) 对齐标识: 在指定了字串宽度时, 默认为右对齐, 增加-号可以改为左对齐.
- (4) 宽度数值
- (5) 小数位数/字串裁切: 在宽度数值后增加的小数部分n, 若后接f(浮点数转义符, 如%6.3f)则设定该浮点数的小数只保留n位, 若后接s(字符串转义符, 如%5.3s)则设定该字符串只显示前n位.

**实例**

```lua
string1 = "Lua"
string2 = "Tutorial"
number1 = 10
number2 = 20
*-- 基本字符串格式化*
print(string.format("基本格式化 %s %s",string1,string2))
*-- 日期格式化*
date = 2; month = 1; year = 2014
print(string.format("日期格式化 %02d/%02d/%04d", date, month, year))
*-- 十进制格式化*
print(string.format("%.4f",1/3))
```

**以上代码执行结果为：**

```
基本格式化 Lua Tutorial
日期格式化 02/01/2014
0.3333
```

**字符和数字之间的转换**

```lua
-- 字符转换
-- 转换第一个字符
print(string.byte("Lua"))
-- 转换第三个字符
print(string.byte("Lua",3))
-- 转换末尾第一个字符
print(string.byte("Lua",-1))
-- 第二个字符
print(string.byte("Lua",2))
-- 转换末尾第二个字符
print(string.byte("Lua",-2))
-- 整数 ASCII 码转换为字符
print(string.char(97))
```

其他函数

```lua
string1 = "www."
string2 = "runoob"
string3 = ".com"
-- 使用 .. 进行字符串连接
print("连接字符串",string1..string2..string3)

-- 字符串长度
print("字符串长度 ",string.len(string2))

-- 字符串复制 2 次
repeatedString = string.rep(string2,2)
print(repeatedString)
```

**匹配模式**

具体使用再记录

## 数组

数组，就是相同数据类型的元素按一定顺序排列的集合，可以是一维数组和多维数组。

在 Lua 中，数组不是一种特定的数据类型，而是一种用来存储一组值的数据结构。

实际上，Lua 中并没有专门的数组类型，而是使用一种被称为 **"table"** 的数据结构来实现数组的功能。

Lua 数组的索引键值可以使用整数表示，数组的大小不是固定的。

**在 Lua 索引值是以 1 为起始，但你也可以指定 0 开始**。

**一维数组：逻辑结构为线性表**

```lua
-- 创建一个数组
local myArray = {10, 20, 30, 40, 50}

-- 访问数组元素
print(myArray[1])  -- 输出 10
print(myArray[3])  -- 输出 30
```

**计算数组的长度**

```lua
local myArray = {10, 20, 30, 40, 50}

-- 计算数组长度
local length = #myArray

print(length) -- 输出 5
```

**为数组添加元素**

```lua
-- 创建一个数组
local myArray = {10, 20, 30, 40, 50}

-- 添加新元素到数组末尾
myArray[#myArray + 1] = 60


-- 循环遍历数组
for i = 1, #myArray do
    print(myArray[i])
end
```

多维数组

```lua
-- 初始化数组
array = {}
for i=1,3 do
   array[i] = {}
      for j=1,3 do
         array[i][j] = i*j
      end
end

-- 访问数组
for i=1,3 do
   for j=1,3 do
      print(array[i][j])
   end
end
```

以一维索引的方式定义二维数组

```lua
-- 初始化数组
array = {}
maxRows = 3
maxColumns = 3
for row=1,maxRows do
   for col=1,maxColumns do
      array[row*maxColumns +col] = row*col
   end
end

-- 访问数组
for row=1,maxRows do
   for col=1,maxColumns do
      print(array[row*maxColumns +col])
   end
end
```



## Lua 迭代器

迭代器（iterator）是一种**对象**，它能够用来遍历标准模板库容器中的部分或全部元素，每个迭代器对象代表容器中的确定的地址。

在 Lua 中迭代器是一种**支持指针类型的结构**，它可以遍历集合的每一个元素。

------

**泛型 for 迭代器**

泛型 for 在自己内部保存迭代函数，实际上它保存三个值：**迭代函数、状态常量、控制变量。**

泛型 for 迭代器提供了集合的 key/value 对，语法格式如下：

```lua
for k, v in pairs(t) do
    print(k, v)
end
```

上面代码中，k, v为变量列表；pairs(t)为表达式列表。

原理

```lua
--泛型遍历的原理
function iter (a, i)
    i = i + 1
    local v = a[i]
    if v then
        return i, v
    end
end
--迭代函数，会返回k，和来自于列表a
--a:状态常量，也就是要遍历的列表
--0：控制变量，也就是索引
--pairs是一种无状态的迭代器
```

查看以下实例:

**实例**

```lua
array = {"Google", "Runoob"}
for key,value in ipairs(array)
do
  print(key, value)
end
```



以上代码执行输出结果为：

```
1  Google
2  Runoob
```

以上实例中我们使用了 Lua 默认提供的迭代函数 ipairs。

下面我们看看泛型 for 的执行过程：

- 首先，初始化，计算 in 后面表达式的值，表达式应该返回泛型 for 需要的三个值：迭代函数、状态常量、控制变量；与多值赋值一样，如果表达式返回的结果个数不足三个会自动用 nil 补足，多出部分会被忽略。
- 第二，将状态常量和控制变量作为参数调用迭代函数（注意：对于 for 结构来说，状态常量没有用处，仅仅在初始化时获取他的值并传递给迭代函数）。
- 第三，将迭代函数返回的值赋给变量列表。
- 第四，如果返回的第一个值为nil循环结束，否则执行循环体。
- 第五，回到第二步再次调用迭代函数

在Lua中我们常常使用函数来描述迭代器，每次调用该函数就返回集合的下一个元素。Lua 的迭代器包含以下两种类型：

- 无状态的迭代器
- 多状态的迭代器

------

**无状态的迭代器**

无状态的迭代器是指不保留任何状态的迭代器，因此在循环中我们可以利用无状态迭代器避免创建闭包花费额外的代价。

每一次迭代，迭代函数都是用两个变量（状态常量和控制变量）的值作为参数被调用，一个无状态的迭代器只利用这两个值可以获取下一个元素。

这种无状态迭代器的典型的简单的例子是 ipairs，它遍历数组的每一个元素，元素的索引需要是数值。

以下实例我们使用了一个简单的函数来实现迭代器，实现 数字 n 的平方：

**实例**

```lua
function square(iteratorMaxCount,currentNumber)
  if currentNumber<iteratorMaxCount
  then
   currentNumber = currentNumber+1
  return currentNumber, currentNumber*currentNumber
  end
end

for i,n in square,3,0
do
  print(i,n)
end
```

以上实例输出结果为：

```
1    1
2    4
3    9
```

迭代的状态包括被遍历的表（循环过程中不会改变的状态常量）和当前的索引下标（控制变量），ipairs 和迭代函数都很简单，我们在 Lua 中可以这样实现：

**实例**

```lua
function iter (a, i)
  i = i + 1
  local v = a[i]
  if v then
    return i, v
  end
end

function ipairs (a)
  return iter, a, 0
end
```



当 Lua 调用 ipairs(a) 开始循环时，他获取三个值：迭代函数 iter、状态常量 a、控制变量初始值 0；然后 Lua 调用 iter(a,0) 返回 1, a[1]（除非 a[1]=nil）；第二次迭代调用 iter(a,1) 返回 2, a[2]……直到第一个 nil 元素。

------

**多状态的迭代器**

很多情况下，迭代器需要保存多个状态信息而不是简单的状态常量和控制变量，最简单的方法是使用闭包，还有一种方法就是将所有的状态信息封装到 table 内，将 table 作为迭代器的状态常量，因为这种情况下可以将所有的信息存放在 table 内，所以迭代函数通常不需要第二个参数。



以下实例我们创建了自己的迭代器：

**实例**



```lua
array = {"Google", "Runoob"}

function elementIterator (collection)
  local index = 0
  local count = #collection
  -- 闭包函数*
  return function ()
   index = index + 1
   if index <= count
   then
     --  返回迭代器的当前元素*
     return collection[index]
   end
  end
end

for element in elementIterator(array)
do
  print(element)
end
```

以上实例输出结果为：

```
Google
Runoob
```

以上实例中我们可以看到，elementIterator 内使用了闭包函数，实现计算集合大小并输出各个元素。

## Lua table(表)

table 是 Lua 的一种数据结构用来帮助我们创建不同的数据类型，如：数组、字典等。

Lua table 使用关联型数组，你可以用任意类型的值来作数组的索引，但这个值不能是 nil。

Lua table 是不固定大小的，你可以根据自己需要进行扩容。

Lua也是通过table来解决模块（module）、包（package）和对象（Object）的。 例如string.format表示使用"format"来索引table string。

------

**table(表)的构造**

构造器是创建和初始化表的表达式。表是Lua特有的功能强大的东西。最简单的构造函数是{}，用来创建一个空表。可以直接初始化数组:

*-- 初始化表*
mytable = {}

*-- 指定值*
mytable[1]= "Lua"

*-- 移除引用*
mytable = nil
*-- lua 垃圾回收会释放内存*

当我们为 table a 并设置元素，然后将 a 赋值给 b，则 a 与 b 都指向同一个内存。如果 a 设置为 nil ，则 b 同样能访问 table 的元素。如果没有指定的变量指向a，Lua的垃圾回收机制会清理相对应的内存。

以下实例演示了以上的描述情况：

**实例**

*

```lua
-- 简单的 table*
mytable = {}
print("mytable 的类型是 ",type(mytable))

mytable[1]= "Lua"
mytable["wow"] = "修改前"
print("mytable 索引为 1 的元素是 ", mytable[1])
print("mytable 索引为 wow 的元素是 ", mytable["wow"])

*-- alternatetable和mytable的是指同一个 table*
alternatetable = mytable

print("alternatetable 索引为 1 的元素是 ", alternatetable[1])
print("alternatetable 索引为 wow 的元素是 ", alternatetable["wow"])

alternatetable["wow"] = "修改后"

print("mytable 索引为 wow 的元素是 ", mytable["wow"])

*-- 释放变量*
alternatetable = nil
print("alternatetable 是 ", alternatetable)

*-- mytable 仍然可以访问*
print("mytable 索引为 wow 的元素是 ", mytable["wow"])

mytable = nil
print("mytable 是 ", mytable)
```

以上代码执行结果为：

```
mytable 的类型是        table
mytable 索引为 1 的元素是       Lua
mytable 索引为 wow 的元素是     修改前
alternatetable 索引为 1 的元素是        Lua
alternatetable 索引为 wow 的元素是      修改前
mytable 索引为 wow 的元素是     修改后
alternatetable 是       nil
mytable 索引为 wow 的元素是     修改后
mytable 是      nil
```

------

**Table 操作**

以下列出了 Table 操作常用的方法：

| 序号 | 方法 & 用途                                                  |
| :--- | :----------------------------------------------------------- |
| 1    | **table.concat (table [, sep [, start [, end]]]):**concat是concatenate(连锁, 连接)的缩写. table.concat()函数列出参数中指定table的数组部分从start位置到end位置的所有元素, 元素间以指定的分隔符(sep)隔开。 |
| 2    | **table.insert (table, [pos,] value):**在table的数组部分指定位置(pos)插入值为value的一个元素. pos参数可选, 默认为数组部分末尾. |
| 3    | **table.maxn (table)**指定table中所有正数key值中最大的key值. 如果不存在key值为正数的元素, 则返回0。(**Lua5.2之后该方法已经不存在了,本文使用了自定义函数实现**) |
| 4    | **table.remove (table [, pos])**返回table数组部分位于pos位置的元素. 其后的元素会被前移. pos参数可选, 默认为table长度, 即从最后一个元素删起。 |
| 5    | **table.sort (table [, comp])**对给定的table进行升序排序。   |

接下来我们来看下这几个方法的实例。

**Table 连接**

我们可以使用 concat() 输出一个列表中元素连接成的字符串:

**实例**

```lua
fruits = {"banana","orange","apple"}
*-- 返回 table 连接后的字符串*
print("连接后的字符串 ",table.concat(fruits))

*-- 指定连接字符*
print("连接后的字符串 ",table.concat(fruits,", "))

*-- 指定索引来连接 table*
print("连接后的字符串 ",table.concat(fruits,", ", 2,3))

执行以上代码输出结果为：
```

```
连接后的字符串     bananaorangeapple
连接后的字符串     banana, orange, apple
连接后的字符串     orange, apple
```

**插入和移除**

以下实例演示了 table 的插入和移除操作:

**实例**

fruits = {"banana","orange","apple"}

```lua
*-- 在末尾插入*
table.insert(fruits,"mango")
print("索引为 4 的元素为 ",fruits[4])

*-- 在索引为 2 的键处插入*
table.insert(fruits,2,"grapes")
print("索引为 2 的元素为 ",fruits[2])

print("最后一个元素为 ",fruits[5])
table.remove(fruits)
print("移除后最后一个元素为 ",fruits[5])

执行以上代码输出结果为：
```



```
索引为 4 的元素为     mango
索引为 2 的元素为     grapes
最后一个元素为     mango
移除后最后一个元素为     nil
```

**Table 排序**

以下实例演示了 sort() 方法的使用，用于对 Table 进行排序：

**实例**

```lua
fruits = {"banana","orange","apple","grapes"}
print("排序前")
for k,v in ipairs(fruits) do
    print(k,v)
end

table.sort(fruits)
print("排序后")
for k,v in ipairs(fruits) do
    print(k,v)
end
```



执行以上代码输出结果为：

```
排序前
1    banana
2    orange
3    apple
4    grapes
排序后
1    apple
2    banana
3    grapes
4    orange
```

**Table 最大值**

table.maxn 在 Lua5.2 之后该方法已经不存在了，我们定义了 table_maxn 方法来实现。

以下实例演示了如何获取 table 中的最大值：

**实例**

```lua
function table_maxn(t)
 local mn=nil;
 for k, v in pairs(t) do
  if(mn==nil) then
   mn=v
  end
  if mn < v **then**
   mn = v
  end
 end
 return mn
end
tbl = {[1] = 2, [2] = 6, [3] = 34, [26] =5}
print("tbl 最大值：", table_maxn(tbl))
print("tbl 长度 ", #tbl)
```

执行以上代码输出结果为：

```
tbl 最大值：    34
tbl 长度     3
```

> **注意：**
>
> 当我们获取 table 的长度的时候无论是使用 **#** 还是 **table.getn** 其都会在索引中断的地方停止计数，而导致无法正确取得 table 的长度。
>
> 可以使用以下方法来代替：
>
> ```
> function table_leng(t)
> local leng=0
> for k, v in pairs(t) do
>  leng=leng+1
> end
> return leng;
> end
> ```

## Lua 模块与包

模块类似于一个封装库，从 Lua 5.1 开始，Lua 加入了标准的模块管理机制，可以把一些公用的代码放在一个文件里，以 API 接口的形式在其他地方调用，有利于代码的重用和降低代码耦合度。

Lua 的模块是由变量、函数等已知元素组成的 table，因此创建一个模块很简单，就是创建一个 table，然后把需要导出的常量、函数放入其中，最后返回这个 table 就行。以下为创建自定义模块 module.lua，文件代码格式如下：

*

```lua
-- 文件名为 module.lua
-- 定义一个名为 module 的模块
local module = {}

-- 定义一个常量
module.constant = "这是一个常量"

-- 定义一个函数
function module.func1()
  io.write("这是一个公有函数！\n")
end

local function func2()
  print("这是一个私有函数！")
end

function module.func3()
  func2()
end

return module
```

由上可知，模块的结构就是一个 table 的结构，因此可以像操作调用 table 里的元素那样来操作调用模块里的常量或函数。

上面的 func2 声明为程序块的局部变量，即表示一个私有函数，因此是不能从外部访问模块里的这个私有函数，必须通过模块里的公有函数来调用.

------

## require 函数

Lua提供了一个名为require的函数用来加载模块。要加载一个模块，只需要简单地调用就可以了。例如：

```
require("<模块名>")
```

或者

```
require "<模块名>"
```

执行 require 后会返回一个由模块常量或函数组成的 table，并且还会定义一个包含该 table 的全局变量。

### test_module.lua 文件

```lua
*-- test_module.lua 文件*
*-- module 模块为上文提到到 module.lua*
require("module")
 
print(module.constant)
module.func3()
```

以上代码执行结果为：

```
这是一个常量
这是一个私有函数！
```

或者给加载的模块定义一个别名变量，方便调用：

### test_module2.lua 文件

*-- test_module2.lua 文件*
*-- module 模块为上文提到到 module.lua*
*-- 别名变量 m*
**local** m = require("module")

print(m.constant)

m.func3()

以上代码执行结果为：

```
这是一个常量
这是一个私有函数！
```

### 加载机制

对于自定义的模块，模块文件不是放在哪个文件目录都行，函数 require 有它自己的文件路径加载策略，它会尝试从 Lua 文件或 C 程序库中加载模块。

require 用于搜索 Lua 文件的路径是存放在全局变量 package.path 中，当 Lua 启动后，会以环境变量 LUA_PATH 的值来初始这个环境变量。如果没有找到该环境变量，则使用一个编译时定义的默认路径来初始化。

当然，如果没有 LUA_PATH 这个环境变量，也可以自定义设置，在当前用户根目录下打开 .profile 文件（没有则创建，打开 .bashrc 文件也可以），例如把 "~/lua/" 路径加入 LUA_PATH 环境变量里：

```
#LUA_PATH
export LUA_PATH="~/lua/?.lua;;"
```

文件路径以 ";" 号分隔，最后的 2 个 ";;" 表示新加的路径后面加上原来的默认路径。

接着，更新环境变量参数，使之立即生效。

```
source ~/.profile
```

这时假设 package.path 的值是：

```
/Users/dengjoe/lua/?.lua;./?.lua;/usr/local/share/lua/5.1/?.lua;/usr/local/share/lua/5.1/?/init.lua;/usr/local/lib/lua/5.1/?.lua;/usr/local/lib/lua/5.1/?/init.lua
```

那么调用 require("module") 时就会尝试打开以下文件目录去搜索目标。

```
/Users/dengjoe/lua/module.lua;
./module.lua
/usr/local/share/lua/5.1/module.lua
/usr/local/share/lua/5.1/module/init.lua
/usr/local/lib/lua/5.1/module.lua
/usr/local/lib/lua/5.1/module/init.lua
```

如果找过目标文件，则会调用 package.loadfile 来加载模块。否则，就会去找 C 程序库。

搜索的文件路径是从全局变量 package.cpath 获取，而这个变量则是通过环境变量 LUA_CPATH 来初始。

搜索的策略跟上面的一样，只不过现在换成搜索的是 so 或 dll 类型的文件。如果找得到，那么 require 就会通过 package.loadlib 来加载它。

------

### C 包

Lua和C是很容易结合的，使用 C 为 Lua 写包。

与Lua中写包不同，C包在使用以前必须首先加载并连接，在大多数系统中最容易的实现方式是通过动态连接库机制。

Lua在一个叫loadlib的函数内提供了所有的动态连接的功能。这个函数有两个参数:库的绝对路径和初始化函数。所以典型的调用的例子如下:

**local** path = "/usr/local/lua/lib/libluasocket.so"
**local** f = loadlib(path, "luaopen_socket")

loadlib 函数加载指定的库并且连接到 Lua，然而它并不打开库（也就是说没有调用初始化函数），反之他返回初始化函数作为 Lua 的一个函数，这样我们就可以直接在Lua中调用他。

如果加载动态库或者查找初始化函数时出错，loadlib 将返回 nil 和错误信息。我们可以修改前面一段代码，使其检测错误然后调用初始化函数：

**local** path = "/usr/local/lua/lib/libluasocket.so"
*-- 或者 path = "C:\\windows\\luasocket.dll"，这是 Window 平台下*
**local** f = assert(loadlib(path, "luaopen_socket"))
f() *-- 真正打开库*

一般情况下我们期望二进制的发布库包含一个与前面代码段相似的 stub 文件，安装二进制库的时候可以随便放在某个目录，只需要修改 stub 文件对应二进制库的实际路径即可。

将 stub 文件所在的目录加入到 LUA_PATH，这样设定后就可以使用 require 函数加载 C 库了。

# Lua 元表(Metatable)

在 Lua table 中我们可以访问对应的 key 来得到 value 值，但是却无法对两个 table 进行操作(比如相加)。

因此 Lua 提供了元表(Metatable)，允许我们改变 table 的行为，每个行为关联了对应的元方法。

例如，使用元表我们可以定义 Lua 如何计算两个 table 的相加操作 a+b。

当 Lua 试图对两个表进行相加时，先检查两者之一是否有元表，之后检查是否有一个叫 **__add** 的字段，若找到，则调用对应的值。 **__add** 等即时字段，其对应的值（往往是一个函数或是 table）就是"元方法"。

有两个很重要的函数来处理元表：

- **setmetatable(table,metatable):** 对指定 table 设置元表(metatable)，如果元表(metatable)中存在 __metatable 键值，setmetatable 会失败。
- **getmetatable(table):** 返回对象的元表(metatable)。

以下实例演示了如何对指定的表设置元表：

mytable = {}              *-- 普通表*
mymetatable = {}            *-- 元表*
setmetatable(mytable,mymetatable)   *-- 把 mymetatable 设为 mytable 的元表*

以上代码也可以直接写成一行：

```
mytable = setmetatable({},{})
```

以下为返回对象元表：

```
getmetatable(mytable)                 -- 这会返回 mymetatable
```

------

## __index 元方法

这是 metatable 最常用的键。

当你通过键来访问 table 的时候，如果这个键没有值，那么Lua就会寻找该table的metatable（假定有metatable）中的__index 键。如果__index包含一个表格，Lua会在表格中查找相应的键。

我们可以在使用 lua 命令进入交互模式查看：

$ lua
Lua 5.3.0  Copyright (C) 1994-2015 Lua.org, PUC-Rio
\> other = { foo = 3 }
\> t = setmetatable({}, { __index = other })
\> t.foo
3
\> t.bar
nil

如果__index包含一个函数的话，Lua就会调用那个函数，table和键会作为参数传递给函数。

__index 元方法查看表中元素是否存在，如果不存在，返回结果为 nil；如果存在则由 __index 返回结果。

## 实例

mytable = setmetatable({key1 = "value1"}, {
 __index = **function**(mytable, key)
  **if** key == "key2" **then**
   **return** "metatablevalue"
  **else**
   **return** nil
  **end**
 **end**
})

print(mytable.key1,mytable.key2)

实例输出结果为：

```
value1    metatablevalue
```

实例解析：

- mytable 表赋值为 **{key1 = "value1"}**。

- mytable 设置了元表，元方法为 __index。

- 在mytable表中查找 key1，如果找到，返回该元素，找不到则继续。

- 在mytable表中查找 key2，如果找到，返回 metatablevalue，找不到则继续。

- 判断元表有没有__index方法，如果__index方法是一个函数，则调用该函数。

- 元方法中查看是否传入 "key2" 键的参数（mytable.key2已设置），如果传入 "key2" 参数返回 "metatablevalue"，否则返回 mytable 对应的键值。

  

我们可以将以上代码简单写成：

mytable = setmetatable({key1 = "value1"}, { __index = { key2 = "metatablevalue" } })
print(mytable.key1,mytable.key2)

> ### 总结
>
> Lua 查找一个表元素时的规则，其实就是如下 3 个步骤:
>
> - 1.在表中查找，如果找到，返回该元素，找不到则继续
> - 2.判断该表是否有元表，如果没有元表，返回 nil，有元表则继续。
> - 3.判断元表有没有 __index 方法，如果 __index 方法为 nil，则返回 nil；如果 __index 方法是一个表，则重复 1、2、3；如果 __index 方法是一个函数，则返回该函数的返回值。
>
> 该部分内容来自作者寰子：https://blog.csdn.net/xocoder/article/details/9028347

------

## __newindex 元方法

__newindex 元方法用来对表更新，__index则用来对表访问 。

当你给表的一个缺少的索引赋值，解释器就会查找__newindex 元方法：如果存在则调用这个函数而不进行赋值操作。

以下实例演示了 __newindex 元方法的应用：

## 实例

mymetatable = {}
mytable = setmetatable({key1 = "value1"}, { __newindex = mymetatable })

print(mytable.key1)

mytable.newkey = "新值2"
print(mytable.newkey,mymetatable.newkey)

mytable.key1 = "新值1"
print(mytable.key1,mymetatable.key1)

以上实例执行输出结果为：

```
value1
nil    新值2
新值1    nil
```

以上实例中表设置了元方法 __newindex，在对新索引键（newkey）赋值时（mytable.newkey = "新值2"），会调用元方法，而不进行赋值。而如果对已存在的索引键（key1），则会进行赋值，而不调用元方法 __newindex。

以下实例使用了 rawset 函数来更新表：

## 实例

mytable = setmetatable({key1 = "value1"}, {
  __newindex = **function**(mytable, key, value)
    rawset(mytable, key, "**\"**"..value.."**\"**")
  **end**
})

mytable.key1 = "new value"
mytable.key2 = 4

print(mytable.key1,mytable.key2)

以上实例执行输出结果为：

```
new value    "4"
```

------

## 为表添加操作符

以下实例演示了两表相加操作：

## 实例

*-- 计算表中最大值，table.maxn在Lua5.2以上版本中已无法使用*
*-- 自定义计算表中最大键值函数 table_maxn，即返回表最大键值*
**function** table_maxn(t)
  **local** mn = 0
  **for** k, v **in** pairs(t) **do**
    **if** mn < k **then**
      mn = k
    **end**
  **end**
  **return** mn
**end**

*-- 两表相加操作*
mytable = setmetatable({ 1, 2, 3 }, {
 __add = **function**(mytable, newtable)
  **for** i = 1, table_maxn(newtable) **do**
   table.insert(mytable, table_maxn(mytable)+1,newtable[i])
  **end**
  **return** mytable
 **end**
})

secondtable = {4,5,6}

mytable = mytable + secondtable
    **for** k,v **in** ipairs(mytable) **do**
print(k,v)
**end**

以上实例执行输出结果为：

```
1    1
2    2
3    3
4    4
5    5
6    6
```

__add 键包含在元表中，并进行相加操作。 表中对应的操作列表如下：(**注意：****__**是两个下划线)

| 模式     | 描述               |
| :------- | :----------------- |
| __add    | 对应的运算符 '+'.  |
| __sub    | 对应的运算符 '-'.  |
| __mul    | 对应的运算符 '*'.  |
| __div    | 对应的运算符 '/'.  |
| __mod    | 对应的运算符 '%'.  |
| __unm    | 对应的运算符 '-'.  |
| __concat | 对应的运算符 '..'. |
| __eq     | 对应的运算符 '=='. |
| __lt     | 对应的运算符 '<'.  |
| __le     | 对应的运算符 '<='. |

------

## __call 元方法

__call 元方法在 Lua 调用一个值时调用。以下实例演示了计算表中元素的和：

## 实例

*-- 计算表中最大值，table.maxn在Lua5.2以上版本中已无法使用*
*-- 自定义计算表中最大键值函数 table_maxn，即计算表的元素个数*
**function** table_maxn(t)
  **local** mn = 0
  **for** k, v **in** pairs(t) **do**
    **if** mn < k **then**
      mn = k
    **end**
  **end**
  **return** mn
**end**

*-- 定义元方法__call*
mytable = setmetatable({10}, {
 __call = **function**(mytable, newtable)
    sum = 0
    **for** i = 1, table_maxn(mytable) **do**
        sum = sum + mytable[i]
    **end**
  **for** i = 1, table_maxn(newtable) **do**
        sum = sum + newtable[i]
    **end**
    **return** sum
 **end**
})
newtable = {10,20,30}
print(mytable(newtable))

以上实例执行输出结果为：

```
70
```

------

## __tostring 元方法

__tostring 元方法用于修改表的输出行为。以下实例我们自定义了表的输出内容：

## 实例

mytable = setmetatable({ 10, 20, 30 }, {
 __tostring = **function**(mytable)
  sum = 0
  **for** k, v **in** pairs(mytable) **do**
        sum = sum + v
    **end**
  **return** "表所有元素的和为 " .. sum
 **end**
})
print(mytable)

以上实例执行输出结果为：

```
表所有元素的和为 60
```

从本文中我们可以知道元表可以很好的简化我们的代码功能，所以了解 Lua 的元表，可以让我们写出更加简单优秀的 Lua 代码。